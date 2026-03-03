---
name: java-code-style
description: Java/Spring Boot 코드 스타일 가이드. Clean Architecture, CQRS, 네이밍 컨벤션을 정의합니다.
---

# Java Code Style Skill

Java/Spring Boot 프로젝트를 위한 코드 스타일 가이드입니다.
세부 지침은 [references](references) 폴더를 참고하세요.

## 아키텍처 패턴

### Clean Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frameworks                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                      Adapters                         │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │                  Application                    │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │                 Domain                    │  │  │  │
│  │  │  │         (Entities, Business Rules)        │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  │            (Use Cases, Ports)                   │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  │        (Controllers, Repositories, Gateways)          │  │
│  └───────────────────────────────────────────────────────┘  │
│              (Spring, JPA, Web, External APIs)              │
└─────────────────────────────────────────────────────────────┘
```

### Entity vs Model 분리

```
Entity (JPA 영속화):
├── @Entity, @Table 어노테이션
├── DB 테이블과 1:1 매핑
├── equals/hashCode: 식별자(id) 기반
└── Repository에서 반환

Model (순수 도메인):
├── 불변 객체 (final 필드, Record 권장)
├── 비즈니스 로직 포함 가능
├── Service 레이어에서 반환
└── Entity 내부 필드 직접 노출 금지
```

```java
// Entity - JPA 영속화 전용
@Entity
@Table(schema = "app", name = "users")
public class UserEntity {
    @Id
    private Long id;
    private String email;
    private String passwordHash;  // 민감정보
}

// Model - 도메인 로직용 (Record 권장)
public record User(
    Long id,
    String email,
    String name,
    LocalDateTime createdAt
) {
    public static User from(UserEntity entity) {
        return new User(entity.getId(), entity.getEmail(), ...);
    }
}
```

### Policy 패턴

비즈니스 제약 조건은 Policy 클래스로 분리합니다.

```java
// Policy - 비즈니스 규칙 검증
@Component
@RequiredArgsConstructor
public class WorkspaceOwnershipPolicy {

    private final WorkspaceRepository workspaceRepository;

    public void validate(String workspaceId, String userId) {
        if (!workspaceRepository.existsByIdAndOwnerId(workspaceId, userId)) {
            throw new AccessDeniedException("워크스페이스 접근 권한 없음");
        }
    }
}

// Service에서 Policy 사용
@Service
@RequiredArgsConstructor
public class JobPostingService {

    private final WorkspaceOwnershipPolicy ownershipPolicy;

    public void updateJobPosting(String workspaceId, String userId, ...) {
        ownershipPolicy.validate(workspaceId, userId);  // 정책 검증
        // 비즈니스 로직
    }
}
```

### 패키지 구조

```
com.example.{module}/
├── domain/                    # 도메인 레이어
│   ├── {Entity}.java         # 도메인 엔티티
│   ├── {ValueObject}.java    # 값 객체
│   └── {DomainService}.java  # 도메인 서비스
│
├── application/               # 애플리케이션 레이어
│   ├── port/
│   │   ├── in/               # 인바운드 포트 (UseCase 인터페이스)
│   │   └── out/              # 아웃바운드 포트 (Repository 인터페이스)
│   ├── service/              # UseCase 구현체
│   └── dto/                  # Command, Query, Response
│
├── adapter/                   # 어댑터 레이어
│   ├── in/
│   │   └── web/              # 컨트롤러
│   └── out/
│       ├── persistence/      # JPA Repository 구현
│       └── external/         # 외부 API 클라이언트
│
└── config/                    # 설정
```

### CQRS 패턴

```java
// Command - 상태 변경 (Create, Update, Delete)
public record CreateUserCommand(
    String email,
    String password,
    String name
) {}

// Query - 조회만 (Read)
public record GetUserQuery(
    Long userId
) {}

// Command Handler
public interface CreateUserUseCase {
    Long execute(CreateUserCommand command);  // ID 반환
}

// Query Handler
public interface GetUserUseCase {
    UserResponse execute(GetUserQuery query);  // DTO 반환
}
```

## 네이밍 컨벤션

### 클래스 네이밍

| 유형       | 패턴                      | 예시                                             |
| ---------- | ------------------------- | ------------------------------------------------ |
| UseCase    | `{Action}{Domain}UseCase` | `CreateUserUseCase`, `UpdateOrderUseCase`        |
| Command    | `{Action}{Domain}Command` | `CreateUserCommand`, `LoginCommand`              |
| Query      | `{Action}{Domain}Query`   | `GetUserQuery`, `SearchOrdersQuery`              |
| Response   | `{Domain}Response`        | `UserResponse`, `OrderDetailResponse`            |
| Controller | `{Domain}Controller`      | `UserController`, `OrderController`              |
| Repository | `{Domain}Repository`      | `UserRepository`, `OrderRepository`              |
| Adapter    | `{Domain}{Type}Adapter`   | `UserPersistenceAdapter`, `PaymentApiAdapter`    |
| Entity     | `{Domain}Entity`          | `UserEntity`, `OrderEntity`                      |
| Exception  | `{Domain}{Type}Exception` | `UserNotFoundException`, `InvalidTokenException` |

### 메서드 네이밍

```java
// 조회
User findById(Long id);           // 단건 조회 (없으면 null)
Optional<User> findByEmail(String email);  // Optional 반환
List<User> findAll();             // 전체 조회
List<User> searchByName(String name);     // 검색
UserResponse getUser(Long id);    // UseCase에서 사용

// 생성
User create(CreateUserCommand cmd);
Long register(RegisterUserCommand cmd);

// 수정
void update(UpdateUserCommand cmd);
void modify(Long id, ModifyRequest request);

// 삭제
void delete(Long id);
void remove(Long id);

// 검증
boolean isValid(String token);
boolean hasPermission(Long userId, String resource);
void validate(CreateUserCommand cmd);  // void - 예외 발생
```

### 변수 네이밍

```java
// Boolean
boolean isActive;
boolean hasPermission;
boolean canDelete;
boolean shouldNotify;

// 컬렉션
List<User> users;          // 복수형
Set<String> permissions;
Map<Long, Order> orderMap;

// 단일 객체
User user;
Order currentOrder;
String encodedPassword;
```

## Java 21 Features 활용

### Record 사용

```java
// DTO에 Record 활용
public record CreateUserCommand(
    @NotBlank String email,
    @NotBlank @Size(min = 8) String password,
    @NotBlank String name
) {}

public record UserResponse(
    Long id,
    String email,
    String name,
    LocalDateTime createdAt
) {
    // 정적 팩토리 메서드
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId(),
            user.getEmail(),
            user.getName(),
            user.getCreatedAt()
        );
    }
}
```

### Pattern Matching

```java
// instanceof 패턴 매칭
if (exception instanceof UserNotFoundException e) {
    return ResponseEntity.notFound().build();
}

// switch 패턴 매칭
String message = switch (status) {
    case PENDING -> "처리 대기중";
    case APPROVED -> "승인됨";
    case REJECTED -> "거절됨";
    case null -> "상태 없음";
};

// sealed class와 함께
sealed interface PaymentResult permits Success, Failure {}
record Success(String transactionId) implements PaymentResult {}
record Failure(String errorCode, String message) implements PaymentResult {}

String handle(PaymentResult result) {
    return switch (result) {
        case Success s -> "결제 성공: " + s.transactionId();
        case Failure f -> "결제 실패: " + f.message();
    };
}
```

### 기타 Java 21 기능

```java
// Text Blocks
String sql = """
    SELECT u.id, u.name, u.email
    FROM users u
    WHERE u.status = 'ACTIVE'
    ORDER BY u.created_at DESC
    """;

// Virtual Threads (적절한 경우)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> sendEmail(user));
    executor.submit(() -> sendNotification(user));
}
```

## 코드 스타일 규칙

### 접근 제어자

```java
// 기본 원칙: 최소 권한
public class UserService {

    private final UserRepository userRepository;  // private final

    public UserResponse getUser(Long id) {        // public - 외부 노출
        User user = findUserById(id);             // private 메서드 호출
        return UserResponse.from(user);
    }

    private User findUserById(Long id) {          // private - 내부용
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

### 메서드 길이

```java
// ❌ 너무 긴 메서드
public void processOrder(Order order) {
    // 50줄 이상의 코드...
}

// ✅ 적절히 분리된 메서드
public void processOrder(Order order) {
    validateOrder(order);
    calculatePrice(order);
    applyDiscount(order);
    saveOrder(order);
    sendNotification(order);
}

private void validateOrder(Order order) {
    // 검증 로직 (10줄 이내)
}
```

### 예외 처리

```java
// 커스텀 예외 정의
public class UserNotFoundException extends RuntimeException {
    private final Long userId;

    public UserNotFoundException(Long userId) {
        super("User not found: " + userId);
        this.userId = userId;
    }

    public Long getUserId() {
        return userId;
    }
}

// 예외 발생
public User findById(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

// Global Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("USER_NOT_FOUND", e.getMessage()));
    }
}
```

### 의존성 주입

```java
// ✅ 생성자 주입 (권장)
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}

// ❌ 필드 주입 (지양)
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

## MapStruct 컨벤션

### 기본 규칙

```java
@Mapper
public interface UserMapper {

    // Entity -> Model (필요 방향만 정의)
    User toModel(UserEntity entity);

    // Model -> Entity (업데이트 시 @MappingTarget 활용)
    @Mapping(target = "id", ignore = true)
    void updateEntity(@MappingTarget UserEntity entity, User model);

    // 복합 매핑 (여러 소스)
    @Mapping(target = "fullName", expression = "java(firstName + \" \" + lastName)")
    UserResponse toResponse(User user, String firstName, String lastName);
}
```

### 컬렉션 매핑

```java
// Null list → 빈 리스트 반환
@Mapper(nullValueIterableMappingStrategy = NullValueMappingStrategy.RETURN_DEFAULT)
public interface OrderMapper {

    List<OrderResponse> toResponseList(List<Order> orders);
}
```

### EmbeddedId 매핑

```java
@Mapper
public interface CompositeKeyMapper {

    // EmbeddedId 필드 매핑
    @Mapping(target = "id.userId", source = "userId")
    @Mapping(target = "id.productId", source = "productId")
    UserProductEntity toEntity(UserProduct model);

    // 업데이트 시 id 전체 무시
    @Mapping(target = "id", ignore = true)
    void updateEntity(@MappingTarget UserProductEntity entity, UserProduct model);
}
```

## 금지 사항

```java
// ❌ 매직 넘버
if (status == 1) { ... }

// ✅ 상수 또는 Enum 사용
if (status == UserStatus.ACTIVE) { ... }

// ❌ 불필요한 주석
// 사용자를 찾는다
User user = userRepository.findById(id);

// ✅ 자명한 코드는 주석 불필요
User user = userRepository.findById(id);

// ❌ 과도한 null 체크
if (user != null && user.getAddress() != null && user.getAddress().getCity() != null) {
    return user.getAddress().getCity();
}

// ✅ Optional 활용
return Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

## 연관 에이전트

- **code-reviewer**: 이 스킬의 규칙으로 코드 리뷰
- **planner**: 아키텍처 패턴에 따라 plan 생성
