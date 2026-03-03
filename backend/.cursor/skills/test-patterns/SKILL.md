---
name: test-patterns
description: Unit Test와 Integration Test 작성 패턴. Mock 활용, @DataJpaTest, 테스트 데이터 관리 가이드를 제공합니다.
---

# Test Patterns Skill

Unit Test와 Integration Test 작성을 위한 패턴과 가이드입니다.

## 테스트 네이밍 규칙

### SUT (System Under Test) 변수명

테스트 대상 객체는 `sut` 변수명을 사용합니다.

```java
class UserServiceTest {

    private UserService sut;  // System Under Test

    @BeforeEach
    void setUp() {
        sut = new UserService(userRepository, passwordEncoder);
    }

    @Test
    void createUser_ValidInput_Success() {
        // When
        var result = sut.createUser(command);  // sut 사용

        // Then
        assertThat(result).isNotNull();
    }
}
```

## SoftAssertions 활용 패턴

### 기본 사용법

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
        softly.assertThat(result.name()).isEqualTo("테스트");
    });
}
```

### .as() 메서드로 검증 의도 명시

```java
@Test
void findExpiredJobPostings_Success() {
    // When
    var result = sut.findExpiredJobPostings(siteType, now);

    // Then
    assertSoftly(softly -> {
        softly.assertThat(result)
            .as("만료된 채용공고가 조회되어야 함")
            .isNotEmpty();
        softly.assertThat(result.get(0).jobPostingId())
            .as("조회된 채용공고 ID가 일치해야 함")
            .isEqualTo(expectedId);
    });
}
```

### 컬렉션 검증 패턴

```java
@Test
void findAllUsers_Success() {
    // When
    var result = sut.findAllUsers(query);

    // Then
    assertSoftly(softly -> {
        softly.assertThat(result).hasSize(2);
        softly.assertThat(result)
            .extracting(User::id)
            .containsExactlyInAnyOrder("user-1", "user-2");
        softly.assertThat(result)
            .extracting(User::email)
            .containsExactlyInAnyOrder("user1@test.com", "user2@test.com");
    });
}
```

### 단일 검증 시 Expression Lambda

```java
// 단일 검증 - Expression Lambda
assertSoftly(softly -> softly.assertThat(result).isEmpty());

// 다건 검증 - Statement Lambda
assertSoftly(softly -> {
    softly.assertThat(result).hasSize(2);
    softly.assertThat(result.get(0).name()).isEqualTo("Alice");
});
```

## 테스트 분류

### 테스트 피라미드

```
                    ┌───────┐
                   /  E2E   \          ← 최소 (수동/선택적)
                  /───────────\
                 /  Integration \       ← 중간
                /─────────────────\
               /    Unit Tests     \    ← 가장 많이
              /─────────────────────\
```

### 테스트 범위

| 유형 | 대상 | Mock 범위 | 속도 |
|------|------|----------|------|
| Unit | 단일 클래스/메서드 | 모든 외부 의존성 | 빠름 |
| Integration | 여러 컴포넌트 | 외부 시스템만 | 중간 |
| E2E | 전체 시스템 | 없음 | 느림 |

## Unit Test 패턴

### 기본 구조

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

    @Test
    @DisplayName("유효한 인증 정보로 로그인 시 토큰을 반환한다")
    void login_ValidCredentials_ReturnsToken() {
        // Given
        var command = new LoginCommand("user@email.com", "password");
        var user = createTestUser();

        when(userRepository.findByEmail("user@email.com"))
            .thenReturn(Optional.of(user));
        when(passwordEncoder.matches("password", user.getPassword()))
            .thenReturn(true);
        when(tokenProvider.createToken(user))
            .thenReturn("jwt-token");

        // When
        var result = authUseCase.login(command);

        // Then
        assertThat(result.accessToken()).isEqualTo("jwt-token");
        verify(userRepository).findByEmail("user@email.com");
    }

    private User createTestUser() {
        return User.builder()
            .id(1L)
            .email("user@email.com")
            .password("encoded-password")
            .build();
    }
}
```

### 예외 테스트

```java
@Test
@DisplayName("존재하지 않는 사용자로 로그인 시 예외를 발생시킨다")
void login_NonExistentUser_ThrowsException() {
    // Given
    var command = new LoginCommand("unknown@email.com", "password");

    when(userRepository.findByEmail("unknown@email.com"))
        .thenReturn(Optional.empty());

    // When & Then
    assertThatThrownBy(() -> authUseCase.login(command))
        .isInstanceOf(UserNotFoundException.class)
        .hasMessageContaining("unknown@email.com");

    verify(passwordEncoder, never()).matches(any(), any());
}
```

### 파라미터화 테스트

```java
@ParameterizedTest
@DisplayName("잘못된 이메일 형식은 검증에 실패한다")
@ValueSource(strings = {
    "invalid",
    "invalid@",
    "@domain.com",
    "user@.com"
})
void validateEmail_InvalidFormat_ReturnsFalse(String email) {
    // When
    boolean result = validator.isValidEmail(email);

    // Then
    assertThat(result).isFalse();
}

@ParameterizedTest
@DisplayName("비밀번호 강도 검증")
@CsvSource({
    "abc, false",           // 너무 짧음
    "abcdefgh, false",      // 숫자 없음
    "12345678, false",      // 문자 없음
    "Abcd1234, true"        // 유효
})
void validatePassword_VariousInputs_ReturnsExpected(
        String password, boolean expected) {
    assertThat(validator.isValidPassword(password)).isEqualTo(expected);
}
```

### Mock 활용 패턴

```java
// void 메서드 Mock
doNothing().when(emailSender).send(any(Email.class));
doThrow(new EmailException()).when(emailSender).send(any());

// 연속 호출
when(idGenerator.generate())
    .thenReturn(1L)
    .thenReturn(2L)
    .thenReturn(3L);

// Answer를 사용한 동적 응답
when(userRepository.save(any(User.class)))
    .thenAnswer(invocation -> {
        User user = invocation.getArgument(0);
        return user.toBuilder().id(1L).build();
    });

// ArgumentCaptor
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(userRepository).save(captor.capture());

User savedUser = captor.getValue();
assertThat(savedUser.getEmail()).isEqualTo("test@email.com");
assertThat(savedUser.getCreatedAt()).isNotNull();
```

## Integration Test 패턴

### Repository 테스트 (@DataJpaTest)

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)  // 실제 DB 사용
@Import(QueryDslConfig.class)  // 필요한 설정 Import
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @BeforeEach
    void setUp() {
        // 테스트 데이터 초기화
    }

    @Test
    @DisplayName("이메일로 사용자를 조회한다")
    void findByEmail_ExistingUser_ReturnsUser() {
        // Given
        User user = User.create("test@email.com", "password");
        entityManager.persistAndFlush(user);

        // When
        Optional<User> result = userRepository.findByEmail("test@email.com");

        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getEmail()).isEqualTo("test@email.com");
    }

    @Test
    @DisplayName("존재하지 않는 이메일로 조회 시 빈 결과를 반환한다")
    void findByEmail_NonExistentUser_ReturnsEmpty() {
        // When
        Optional<User> result = userRepository.findByEmail("unknown@email.com");

        // Then
        assertThat(result).isEmpty();
    }
}
```

### Controller 테스트 (standaloneSetup 권장)

standaloneSetup 방식은 Spring Context 없이 빠르게 Controller를 테스트합니다.

```java
class AuthControllerTest {

    private MockMvc mockMvc;
    private ObjectMapper objectMapper;
    private AuthUseCase authUseCase;

    @BeforeEach
    void setUp() {
        // Mock 생성
        authUseCase = mock(AuthUseCase.class);

        // ObjectMapper 설정 (Java 8 날짜/시간 모듈)
        objectMapper = new ObjectMapper().findAndRegisterModules();

        // Validator 설정
        var validator = new LocalValidatorFactoryBean();
        validator.afterPropertiesSet();

        // MockMvc 빌드 (standaloneSetup)
        mockMvc = MockMvcBuilders.standaloneSetup(
                new AuthController(authUseCase))
            .setValidator(validator)
            .build();
    }

    @Test
    @DisplayName("POST /api/auth/login - 성공")
    void login_ValidRequest_ReturnsToken() throws Exception {
        // Given
        var request = new LoginRequest("user@email.com", "password");
        var response = new LoginResponse("jwt-token", 3600);

        given(authUseCase.login(any(LoginCommand.class)))
            .willReturn(response);

        // When & Then
        mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.accessToken").value("jwt-token"))
            .andExpect(jsonPath("$.expiresIn").value(3600));

        verify(authUseCase).login(any(LoginCommand.class));
    }

    @Test
    @DisplayName("POST /api/auth/login - 잘못된 요청")
    void login_InvalidRequest_ReturnsBadRequest() throws Exception {
        // Given
        var request = new LoginRequest("", "");  // 빈 값

        // When & Then
        mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andDo(print())
            .andExpect(status().isBadRequest());
    }
}
```

### 전체 통합 테스트 (@SpringBootTest)

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class AuthIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @BeforeEach
    void setUp() {
        User user = User.builder()
            .email("test@email.com")
            .password(passwordEncoder.encode("password123"))
            .name("Test User")
            .build();
        userRepository.save(user);
    }

    @Test
    @DisplayName("로그인 전체 흐름 테스트")
    void login_EndToEnd_Success() throws Exception {
        // Given
        var request = Map.of(
            "email", "test@email.com",
            "password", "password123"
        );

        // When & Then
        mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new ObjectMapper().writeValueAsString(request)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.accessToken").exists())
            .andExpect(jsonPath("$.expiresIn").isNumber());
    }
}
```

## 테스트 데이터 관리

### Test Fixtures

```java
public class UserFixtures {

    public static User defaultUser() {
        return User.builder()
            .id(1L)
            .email("user@email.com")
            .password("encoded-password")
            .name("Test User")
            .status(UserStatus.ACTIVE)
            .createdAt(LocalDateTime.now())
            .build();
    }

    public static User inactiveUser() {
        return defaultUser().toBuilder()
            .status(UserStatus.INACTIVE)
            .build();
    }

    public static User adminUser() {
        return defaultUser().toBuilder()
            .email("admin@email.com")
            .roles(Set.of(Role.ADMIN))
            .build();
    }

    public static List<User> userList(int count) {
        return IntStream.range(0, count)
            .mapToObj(i -> defaultUser().toBuilder()
                .id((long) i)
                .email("user" + i + "@email.com")
                .build())
            .toList();
    }
}
```

### Test Builder

```java
public class UserTestBuilder {

    private Long id = 1L;
    private String email = "user@email.com";
    private String password = "encoded-password";
    private String name = "Test User";
    private UserStatus status = UserStatus.ACTIVE;

    public static UserTestBuilder aUser() {
        return new UserTestBuilder();
    }

    public UserTestBuilder withId(Long id) {
        this.id = id;
        return this;
    }

    public UserTestBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    public UserTestBuilder inactive() {
        this.status = UserStatus.INACTIVE;
        return this;
    }

    public User build() {
        return User.builder()
            .id(id)
            .email(email)
            .password(password)
            .name(name)
            .status(status)
            .build();
    }
}

// 사용
User user = aUser()
    .withEmail("custom@email.com")
    .inactive()
    .build();
```

## 테스트 분류 권장사항

### Mock 사용 기준

```
Unit Test (Mock 사용):
├── UseCase/Service 클래스
├── Domain 로직
├── Validator
└── Util 클래스

Integration Test (실제 구현 사용):
├── Repository (@DataJpaTest)
├── Controller (@WebMvcTest)
└── 전체 흐름 (@SpringBootTest)

Mock 하지 않는 것:
├── 테스트 대상 클래스 자체
├── 단순 DTO/VO
└── 상태 없는 유틸리티
```

## 연관 에이전트

- **tdd-guide**: 테스트 작성 가이드
- **code-reviewer**: 테스트 코드 리뷰
- **build-fixer**: 테스트 실패 해결
