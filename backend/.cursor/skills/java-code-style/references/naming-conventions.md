# 네이밍 규칙

## 클래스 네이밍

### UseCase 인터페이스

**패턴**: `{동사}{도메인}Usecase`

**규칙**:

- 동사는 명확한 행동 표현 사용
- 도메인은 단수형
- Usecase는 대문자 U로 시작

**예시**:

```java
// 생성 (Create)
public interface CreateExampleUsecase {
  ExampleResult execute(CreateExampleCommand command);
}

// 조회 단건 (Get)
public interface GetExampleUsecase {
  ExampleResult execute(GetExampleQuery query);
}

// 조회 목록 (Get + 복수형)
public interface GetExamplesUsecase {
  List<ExampleResult> execute(GetExamplesQuery query);
}

// 수정 (Update)
public interface UpdateExampleUsecase {
  ExampleResult execute(UpdateExampleCommand command);
}

// 삭제 (Delete)
public interface DeleteExampleUsecase {
  void execute(DeleteExampleCommand command);
}

// 업서트 (Upsert)
public interface UpsertStepsUseCase {
  List<StepResult> execute(UpsertStepsCommand command);
}
```

**일반적인 동사**:

- `Create`: 생성
- `Get`: 단건 조회
- `Get{복수형}`: 목록 조회
- `Update`: 수정
- `Delete`: 삭제
- `Upsert`: 생성 또는 수정
- `Sync`: 동기화
- `Process`: 처리
- `Send`: 전송
- `Publish`: 발행

### Adapter (UseCase 구현체)

**패턴**: `{Feature}CoreService`

**규칙**:

- Feature는 기능 영역 (예: Example, Step, Message)
- `CoreService` 접미사 필수
- **package-private** (class 앞에 public 없음)
- 여러 UseCase 인터페이스 implements 가능

**예시**:

```java

@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ExampleCoreService implements
  CreateExampleUsecase,
  GetExampleUsecase,
  GetExamplesUsecase,
  UpdateExampleUsecase {
  // 구현
}
```

**복잡한 경우 역할별 분리**:

```java
// 기본 CRUD
class ApplicantProgressCoreService {
}

// Worker 전용 (외부 API 호출)
class ApplicantProgressWorkerCoreService {
}

// 다운로드/내보내기
class ApplicantProgressDownloadCoreService {
}
```

### Command (생성/수정/삭제 입력)

**패턴**: `{동사}{도메인}Command`

**규칙**:

- 동사는 UseCase와 동일
- Command 접미사 필수
- **Java 21 record** 사용
- **public** 접근자

**예시**:

```java
/**
 * Example 생성 Command
 */
public record CreateExampleCommand(
    String name,
    String description
  ) {
  public CreateExampleCommand {
    // Compact Constructor로 검증
    if (name == null || name.isBlank()) {
      throw new IllegalArgumentException("Name cannot be empty");
    }
  }
}

/**
 * Example 수정 Command
 */
public record UpdateExampleCommand(
  String id,
  String name,
  String description
) {
  public UpdateExampleCommand {
    if (id == null || id.isBlank()) {
      throw new IllegalArgumentException("ID cannot be empty");
    }
  }
}

/**
 * Example 삭제 Command
 */
public record DeleteExampleCommand(String id) {
  public DeleteExampleCommand {
    if (id == null || id.isBlank()) {
      throw new IllegalArgumentException("ID cannot be empty");
    }
  }
}
```

### Query (조회 입력)

**패턴**: `Get{도메인}Query` 또는 `Get{도메인}sQuery`

**규칙**:

- Get으로 시작
- 단건 조회는 단수형 Query
- 목록 조회는 복수형 Query
- **Java 21 record** 사용
- **public** 접근자

**예시**:

```java
/**
 * Example 단건 조회 Query
 */
public record GetExampleQuery(String id) {
  public GetExampleQuery {
    if (id == null || id.isBlank()) {
      throw new IllegalArgumentException("ID cannot be empty");
    }
  }
}

/**
 * Example 목록 조회 Query
 */
public record GetExamplesQuery() {
  // 필터 조건 없는 경우 빈 record
}

/**
 * Example 목록 조회 Query (필터 포함)
 */
public record GetExamplesQuery(
  String name,
  LocalDate createdAfter,
  LocalDate createdBefore
) {
  // 필터 조건이 있는 경우
}

/**
 * Step 목록 조회 Query
 */
public record GetStepsQuery(String jobPostingId) {
  public GetStepsQuery {
    if (jobPostingId == null || jobPostingId.isBlank()) {
      throw new IllegalArgumentException("Job posting ID required");
    }
  }
}
```

### Result (출력)

**패턴**: `{도메인}Result`

**규칙**:

- Result 접미사 필수
- **Java 21 record** 사용
- **@Builder** 어노테이션 적용
- **public** 접근자
- Domain Model과 1:1 매핑 (필요한 필드만)

**예시**:

```java
/**
 * Example UseCase 실행 결과
 */
@Builder
public record ExampleResult(
    String id,
    String name,
    String description,
    Instant createdAt,
    Instant updatedAt
  ) {
}

/**
 * Step Result (보강된 정보 포함)
 */
@Builder(toBuilder = true)
public record StepResult(
  String id,
  String jobPostingMappingId,
  String name,
  Integer ordering,
  // 보강 정보
  String jobPostingId,
  String workspaceId,
  Boolean hasApplicantProgress
) {
}
```

**toBuilder = true 활용**:

```java

@Builder(toBuilder = true)
public record ExampleResult(...) {
}

// 사용
ExampleResult enriched = baseResult.toBuilder()
  .additionalInfo("extra")
  .build();
```

### Mapper (변환 계층)

**패턴**: `{Feature}CoreMapper`

**규칙**:

- Feature는 기능 영역
- `CoreMapper` 접미사 필수
- **@Mapper** 어노테이션 (MapStruct)
- 인터페이스로 정의

**예시**:

```java

@Mapper
public interface ExampleCoreMapper {

  // 기본 변환 메서드
  Example toDomainModel(CreateExampleCommand command);

  ExampleResult toResult(Example model);

  List<ExampleResult> toResults(List<Example> models);

  // default 메서드로 도메인 모델 생성
  default Example toNewExample(String name, String description) {
    return Example.builder()
      .name(name.trim())
      .description(description)
      .build();
  }

  // Result 보강
  default ExampleResult toEnrichedResult(
    ExampleResult baseResult,
    String additionalInfo) {
    return baseResult.toBuilder()
      .additionalInfo(additionalInfo)
      .build();
  }
}
```

**Mapper 네이밍 컨벤션**:

- `toDomainModel()`: Command/Query → Domain Model
- `toResult()`: Domain Model → Result (단건)
- `toResults()`: Domain Model List → Result List
- `toNew{Entity}()`: 새 도메인 모델 생성
- `toUpdated{Entity}()`: 업데이트용 복사본 생성
- `toEnriched{Result}()`: Result 보강

### Policy (비즈니스 정책)

**패턴**:

- 인터페이스: `{도메인}{동작}Policy`
- 구현체: `Default{도메인}{동작}Policy`

**규칙**:

- Policy 접미사 필수
- 인터페이스는 public
- 구현체는 package-private 또는 public
- `@Component` 어노테이션 적용

**예시**:

```java
/**
 * Example 생성 정책 인터페이스
 */
public interface ExampleCreatePolicy {
  void validate(CreateExampleCommand command);
}

/**
 * Example 생성 정책 구현체
 */
@Slf4j
@Component
@RequiredArgsConstructor
class DefaultExampleCreatePolicy implements ExampleCreatePolicy {

  @Override
  public void validate(CreateExampleCommand command) {
    log.debug("Validating create example command: {}", command);

    if (command.name().length() > 126) {
      throw new JobkoreaBusinessException(
        CommonErrorCode.BAD_REQUEST,
        "Name cannot exceed 126 characters"
      );
    }
  }
}
```

**Policy 네이밍 패턴**:

- `{도메인}CreatePolicy`: 생성 정책
- `{도메인}UpdatePolicy`: 수정 정책
- `{도메인}DeletePolicy`: 삭제 정책
- `{도메인}UpsertPolicy`: 업서트 정책

### Event (도메인 이벤트)

**패턴**: `{도메인}{과거형동사}EventV{버전}`

**규칙**:

- 과거형 동사 사용 (Created, Updated, Deleted 등)
- EventV{버전} 접미사 (V1, V2, ...)
- **Java 21 record** 사용
- **@Builder(toBuilder = true)** 적용
- **public** 접근자

**예시**:

```java
/**
 * Example 생성 이벤트 V1
 */
@Builder(toBuilder = true)
public record ExampleCreatedEventV1(
    @JsonProperty("_context") ExampleEventContext _context,
    @JsonIgnore String source
  ) implements ExampleDomainEvent {

  public static final String EVENT_TYPE = "bizcenter.applicant.ExampleCreated.v1";

  @Override
  public ExampleEventContext get$Context() {
    return _context;
  }

  @Override
  public String getSource() {
    return source;
  }
}

/**
 * Example 수정 이벤트 V1
 */
@Builder(toBuilder = true)
public record ExampleUpdatedEventV1(
  @JsonProperty("_context") ExampleEventContext _context,
  @JsonIgnore String source
) implements ExampleDomainEvent {

  public static final String EVENT_TYPE = "bizcenter.applicant.ExampleUpdated.v1";
}
```

**일반적인 이벤트 동사**:

- `Created`: 생성됨
- `Updated`: 수정됨
- `Deleted`: 삭제됨
- `Synced`: 동기화됨
- `Sent`: 전송됨
- `Published`: 발행됨

## 메서드 네이밍

### UseCase 메서드

**패턴**: `execute(Command/Query)`

**규칙**:

- 모든 UseCase는 `execute()` 메서드 사용
- 파라미터는 Command 또는 Query
- 반환 타입은 Result 또는 void

**예시**:

```java
public interface CreateExampleUsecase {
  ExampleResult execute(CreateExampleCommand command);
}

public interface GetExampleUsecase {
  ExampleResult execute(GetExampleQuery query);
}

public interface DeleteExampleUsecase {
  void execute(DeleteExampleCommand command);
}
```

### Policy 메서드

**패턴**: `validate(Command/Query)` 또는 `validate(ValidationInput)`

**규칙**:

- `validate()` 메서드 사용
- 파라미터는 Command/Query 또는 ValidationInput
- 반환 타입은 void
- 정책 위반 시 예외 발생

**예시**:

```java
public interface ExampleCreatePolicy {
  void validate(CreateExampleCommand command);
}

public interface StepUpsertPolicy {
  void validate(StepUpsertValidationInput input);
}
```

### Mapper 메서드

**기본 변환**:

- `toDomainModel(Command/Query)`: Command/Query → Domain Model
- `toResult(Model)`: Domain Model → Result
- `toResults(List<Model>)`: List → List

**도메인 모델 생성**:

- `toNew{Entity}()`: 새 엔티티 생성
- `toUpdated{Entity}()`: 업데이트용 엔티티

**Result 보강**:

- `toEnriched{Result}()`: 기본 Result에 정보 추가

**예시**:

```java

@Mapper
public interface StepCoreMapper {

  // 기본 변환
  Step toDomainModel(CreateStepCommand command);

  StepResult toResult(Step model);

  List<StepResult> toResults(List<Step> models);

  // 도메인 모델 생성
  default Step toNewStep(String name, Integer ordering) {
    return Step.builder()
      .name(name.trim())
      .ordering(ordering)
      .build();
  }

  // Result 보강
  default StepResult toEnrichedResult(
    StepResult baseResult,
    String jobPostingId,
    String workspaceId,
    Boolean hasApplicantProgress) {
    return baseResult.toBuilder()
      .jobPostingId(jobPostingId)
      .workspaceId(workspaceId)
      .hasApplicantProgress(hasApplicantProgress)
      .build();
  }
}
```

## 패키지 네이밍

### Core Layer 패키지 구조

**기본 구조**:

```
bizcenter.core.{feature}/
├── usecase/              # UseCase 인터페이스
├── adapter/              # UseCase 구현체 (Adapter)
├── mapper/               # Mapper 인터페이스
├── policy/               # Policy 인터페이스 및 구현체
└── event/                # 도메인 이벤트
```

**model 패키지 사용하는 경우** (표준 구조):

```
bizcenter.core.{feature}/
├── usecase/
├── adapter/
├── model/
│   ├── command/          # Command 클래스
│   ├── query/            # Query 클래스
│   └── result/           # Result 클래스
├── mapper/
├── policy/
└── event/
```

**내부 클래스 패턴 사용하는 경우** (간소화 구조):

```
bizcenter.core.{feature}/
├── usecase/              # UseCase + Command/Query/Result 내부 클래스
├── adapter/
├── mapper/
└── policy/
```

**예시: Step feature**:

```
bizcenter.core.step/
├── usecase/
│   ├── CreateStepsForJobPostingUsecase.java  # record Command, Result 포함
│   ├── GetStepUseCase.java                   # record Query, Result 포함
│   └── UpsertStepsUseCase.java
├── adapter/
│   └── StepCoreService.java
├── mapper/
│   └── StepCoreMapper.java
└── policy/
    ├── StepUpsertPolicy.java
    └── StepUpsertPolicyImpl.java
```

### Domain Layer 패키지 구조

```
bizcenter.domain.{domain}/
├── model/                # Domain Model
├── entity/               # JPA Entity
├── repository/           # Repository 인터페이스
└── service/              # Domain Service
```

### 패키지 네이밍 규칙

- 소문자만 사용
- 복합어는 camelCase 아닌 붙여쓰기 (예: `applicantprogress`)
- 명확한 의미 전달
- 너무 길지 않게 (2-3 단어 이내)

## 변수 네이밍

### 일반 규칙

- camelCase 사용
- 의미 있는 이름 사용
- 축약어 지양 (명확한 경우만 허용)

**좋은 예**:

```java
String jobPostingId;
List<ApplicantProgress> applicantProgresses;
Map<String, Step> stepMap;
ExampleResult result;
```

**나쁜 예**:

```java
String jpId;           // 축약어
List<ApplicantProgress> list;  // 모호함
Map<String, Step> m;   // 의미 없음
ExampleResult r;       // 축약어
```

### 컬렉션 변수

- List: 복수형 사용 (`examples`, `steps`)
- Map: `{key}To{value}Map` 또는 `{value}Map` (`stepMap`, `idToStepMap`)
- Set: 복수형 사용 (`stepIds`, `processedIds`)

**예시**:

```java
List<Example> examples;
Map<String, Step> stepMap;
Set<String> stepIds;
```

## 상수 네이밍

### 클래스 상수

- UPPER_SNAKE_CASE 사용
- static final로 선언

**예시**:

```java
private static final String SOURCE = "bizcenter-ats";
private static final int MAX_NAME_LENGTH = 126;
public static final String EVENT_TYPE = "bizcenter.applicant.ExampleCreated.v1";
```

### 객체 생성,변환등의 메소드명

- 가능한 아래 3가지 네이밍 규칙을 따릅니다:

  - from: 1~2개 파라미터로 → 다른 타입 변환 (ActivityContext.from(progress, memberId))
    - 의미: "~로부터" - 하나의 파라미터를 받아 변환
  - of: 여러 파라미터로 → 객체 생성 (ExampleEventContext.of(...))
    - 의미: "~을 가진" - 여러 파라미터를 조합해서 인스턴스 생성
  - to{Type}: 다른 타입으로 변환 (mapper.toDomainModel(command))
    - 의미: "~로 변환" - 현재 객체를 다른 타입으로 변환 (주로 인스턴스 메서드)
  - build: 빌더 패턴 사용 시 (ExampleResult.builder().id(...).build())
    - 의미: "구축하다" - 빌더 패턴에서 최종 객체 생성

- create, valueOf, toInstance 등은 지양 합니다. 
