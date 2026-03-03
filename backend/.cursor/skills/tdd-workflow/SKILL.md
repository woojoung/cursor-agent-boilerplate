---
name: tdd-workflow
description: TDD(Test-Driven Development) 방법론 상세 가이드. RED-GREEN-REFACTOR 사이클과 테스트 작성 패턴을 제공합니다.
---

# TDD Workflow Skill

TDD(Test-Driven Development) 방법론의 상세 가이드입니다.

## TDD 원칙

### RED-GREEN-REFACTOR 사이클

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│    🔴 RED          🟢 GREEN         🔵 REFACTOR      │
│    ─────────────────────────────────────────────     │
│    실패하는         테스트 통과하는    코드 품질       │
│    테스트 작성      최소 코드 작성     개선            │
│                                                      │
│         │              │               │             │
│         ▼              ▼               ▼             │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐        │
│    │  Test   │───▶│  Code   │───▶│ Refactor│───┐    │
│    │  First  │    │  Pass   │    │  Clean  │   │    │
│    └─────────┘    └─────────┘    └─────────┘   │    │
│         ▲                                      │    │
│         └──────────────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 핵심 규칙

1. **테스트 먼저**: 프로덕션 코드 전에 테스트 작성
2. **최소 구현**: 테스트 통과하는 가장 간단한 코드
3. **지속적 리팩토링**: 매 사이클마다 코드 정리
4. **작은 단위**: 한 번에 하나의 기능만

## RED Phase: 테스트 작성

### Given-When-Then 패턴

```java
@Test
@DisplayName("유효한 인증 정보로 로그인 시 토큰을 반환한다")
void loginWithValidCredentials_ReturnsToken() {
    // Given - 사전 조건 설정
    var command = new LoginCommand("user@email.com", "password123");
    var user = User.create("user@email.com", "encodedPassword");

    when(userRepository.findByEmail("user@email.com"))
        .thenReturn(Optional.of(user));
    when(passwordEncoder.matches("password123", "encodedPassword"))
        .thenReturn(true);
    when(tokenProvider.createToken(any()))
        .thenReturn("jwt-token");

    // When - 테스트 대상 실행
    var result = authUseCase.login(command);

    // Then - 결과 검증
    assertThat(result).isNotNull();
    assertThat(result.accessToken()).isEqualTo("jwt-token");
    assertThat(result.expiresIn()).isPositive();
}
```

### 테스트 케이스 설계

```java
// 정상 케이스
@Test
void validInput_Success() { }

// 경계값 케이스
@Test
void emptyInput_ThrowsException() { }

@Test
void nullInput_ThrowsException() { }

@Test
void maxLengthInput_Success() { }

// 예외 케이스
@Test
void invalidEmail_ThrowsInvalidEmailException() { }

@Test
void wrongPassword_ThrowsAuthenticationException() { }

@Test
void nonExistentUser_ThrowsUserNotFoundException() { }
```

### 테스트 명명 규칙

```java
// 패턴 1: method_condition_expectedResult
@Test
void login_ValidCredentials_ReturnsToken() { }

@Test
void login_InvalidPassword_ThrowsException() { }

// 패턴 2: should_expectedBehavior_when_condition
@Test
void shouldReturnToken_whenCredentialsAreValid() { }

@Test
void shouldThrowException_whenPasswordIsInvalid() { }

// 패턴 3: DisplayName 활용 (한글 가능)
@Test
@DisplayName("유효한 인증 정보로 로그인하면 토큰을 반환한다")
void loginWithValidCredentials() { }
```

## GREEN Phase: 최소 구현

### 단계적 구현

```java
// Step 1: 컴파일만 되게 (껍데기)
public class AuthUseCase {
    public LoginResponse login(LoginCommand command) {
        return null;  // 일단 null 반환
    }
}

// Step 2: 테스트 통과하는 최소 구현
public class AuthUseCase {
    public LoginResponse login(LoginCommand command) {
        return new LoginResponse("jwt-token", 3600);  // 하드코딩
    }
}

// Step 3: 실제 로직 구현
public class AuthUseCase {
    public LoginResponse login(LoginCommand command) {
        User user = userRepository.findByEmail(command.email())
            .orElseThrow(() -> new UserNotFoundException(command.email()));

        if (!passwordEncoder.matches(command.password(), user.getPassword())) {
            throw new InvalidPasswordException();
        }

        String token = tokenProvider.createToken(user);
        return new LoginResponse(token, 3600);
    }
}
```

### Fake It Till You Make It

```java
// 처음에는 가짜 구현
@Test
void add_TwoNumbers_ReturnsSum() {
    Calculator calc = new Calculator();
    assertThat(calc.add(2, 3)).isEqualTo(5);
}

// Step 1: 하드코딩으로 통과
public int add(int a, int b) {
    return 5;  // Fake it!
}

// Step 2: 테스트 추가
@Test
void add_DifferentNumbers_ReturnsSum() {
    assertThat(calc.add(3, 4)).isEqualTo(7);
}

// Step 3: 실제 구현
public int add(int a, int b) {
    return a + b;  // Make it!
}
```

## REFACTOR Phase: 코드 개선

### 리팩토링 체크리스트

```
□ 중복 코드 제거
□ 메서드 추출 (긴 메서드 분리)
□ 변수명/메서드명 개선
□ 매직 넘버 상수화
□ 조건문 단순화
□ 불필요한 코드 제거
```

### 리팩토링 예시

```java
// Before
public LoginResponse login(LoginCommand command) {
    User user = userRepository.findByEmail(command.email())
        .orElseThrow(() -> new UserNotFoundException(command.email()));

    if (!passwordEncoder.matches(command.password(), user.getPassword())) {
        throw new InvalidPasswordException();
    }

    if (!user.isActive()) {
        throw new InactiveUserException();
    }

    if (user.isLocked()) {
        throw new LockedUserException();
    }

    String token = tokenProvider.createToken(user);
    return new LoginResponse(token, 3600);
}

// After (메서드 추출)
public LoginResponse login(LoginCommand command) {
    User user = findUserByEmail(command.email());
    validatePassword(command.password(), user);
    validateUserStatus(user);

    return createLoginResponse(user);
}

private User findUserByEmail(String email) {
    return userRepository.findByEmail(email)
        .orElseThrow(() -> new UserNotFoundException(email));
}

private void validatePassword(String rawPassword, User user) {
    if (!passwordEncoder.matches(rawPassword, user.getPassword())) {
        throw new InvalidPasswordException();
    }
}

private void validateUserStatus(User user) {
    if (!user.isActive()) {
        throw new InactiveUserException();
    }
    if (user.isLocked()) {
        throw new LockedUserException();
    }
}

private LoginResponse createLoginResponse(User user) {
    String token = tokenProvider.createToken(user);
    return new LoginResponse(token, TOKEN_EXPIRY_SECONDS);
}
```

## Mockito 활용 패턴

### Mock 기본 설정

```java
@ExtendWith(MockitoExtension.class)
class AuthUseCaseTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @Mock
    private TokenProvider tokenProvider;

    @InjectMocks
    private AuthUseCase authUseCase;

    @BeforeEach
    void setUp() {
        // 공통 Mock 설정
    }
}
```

### Stubbing 패턴

```java
// 기본 반환값 설정
when(userRepository.findById(1L)).thenReturn(Optional.of(user));

// any() 매처 사용
when(userRepository.findById(any())).thenReturn(Optional.of(user));

// 예외 발생
when(userRepository.findById(999L))
    .thenThrow(new UserNotFoundException(999L));

// 연속 호출
when(tokenProvider.createToken(any()))
    .thenReturn("token1")
    .thenReturn("token2");

// void 메서드 stubbing
doNothing().when(emailSender).send(any());
doThrow(new EmailException()).when(emailSender).send(any());
```

### Verification 패턴

```java
// 호출 횟수 검증
verify(userRepository, times(1)).findById(1L);
verify(userRepository, never()).delete(any());
verify(emailSender, atLeastOnce()).send(any());

// 호출 순서 검증
InOrder inOrder = inOrder(userRepository, emailSender);
inOrder.verify(userRepository).save(any());
inOrder.verify(emailSender).send(any());

// 인자 캡처
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(userRepository).save(captor.capture());
User savedUser = captor.getValue();
assertThat(savedUser.getEmail()).isEqualTo("test@email.com");
```

## AssertJ 활용 패턴

```java
// 기본 검증
assertThat(result).isNotNull();
assertThat(result).isEqualTo(expected);
assertThat(result).isNotEqualTo(other);

// 문자열 검증
assertThat(name).startsWith("John");
assertThat(email).contains("@");
assertThat(message).isBlank();

// 숫자 검증
assertThat(age).isPositive();
assertThat(count).isBetween(1, 10);
assertThat(price).isGreaterThan(0);

// 컬렉션 검증
assertThat(users).hasSize(3);
assertThat(users).contains(user1, user2);
assertThat(users).extracting("name").containsExactly("Alice", "Bob");

// 예외 검증
assertThatThrownBy(() -> service.process(null))
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessageContaining("null");

assertThatCode(() -> service.process(validInput))
    .doesNotThrowAnyException();
```

## SoftAssertions 활용

다건 검증 시 SoftAssertions를 사용하여 모든 실패를 한 번에 확인합니다.

```java
import static org.assertj.core.api.SoftAssertions.assertSoftly;

@Test
void createUser_Success() {
    // When
    var result = sut.createUser(command);

    // Then - 여러 검증을 그룹화
    assertSoftly(softly -> {
        softly.assertThat(result).isNotNull();
        softly.assertThat(result.id()).isNotNull();
        softly.assertThat(result.email()).isEqualTo("test@email.com");
    });
}

// .as() 메서드로 검증 의도 명시 권장
assertSoftly(softly -> {
    softly.assertThat(result)
        .as("결과가 비어있지 않아야 함")
        .isNotEmpty();
    softly.assertThat(result.get(0).id())
        .as("첫 번째 결과 ID가 일치해야 함")
        .isEqualTo(expectedId);
});
```

## 테스트 더블 선택 가이드

```
┌─────────────────────────────────────────────────────────┐
│                    Test Double 선택                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Dummy    : 파라미터 채우기용, 실제 사용 안 함             │
│  Stub     : 미리 정해진 값 반환                          │
│  Spy      : 실제 객체 + 일부 stubbing                    │
│  Mock     : 호출 검증 가능, 행위 검증                     │
│  Fake     : 동작하는 간단한 구현 (In-memory DB 등)        │
│                                                         │
└─────────────────────────────────────────────────────────┘

권장:
- 외부 의존성 (DB, API) → Mock
- 복잡한 로직의 협력 객체 → Stub
- 상태 검증이 필요한 경우 → Spy
- 테스트용 간단한 구현 → Fake
```

## 연관 에이전트

- **tdd-guide**: 이 스킬을 기반으로 TDD 사이클 강제
- **planner**: TDD 단계를 plan에 포함
- **code-reviewer**: 테스트 품질 리뷰
