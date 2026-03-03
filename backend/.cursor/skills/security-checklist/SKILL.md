---
name: security-checklist
description: OWASP Top 10 기반 보안 체크리스트. Spring Security 설정, 데이터 보호 가이드를 제공합니다.
---

# Security Checklist Skill

OWASP Top 10 기반의 보안 체크리스트와 Spring Security 설정 가이드입니다.

## OWASP Top 10 체크리스트

### A01: Broken Access Control

```
□ 모든 API 엔드포인트에 인증 검증
□ 리소스 접근 시 권한 검증 (@PreAuthorize)
□ URL 기반 접근 제어 설정
□ 직접 객체 참조(IDOR) 방지
□ CORS 설정 적절성
□ HTTP 메서드 제한
```

**안전한 코드 예시:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    public UserResponse getUser(@PathVariable Long id) {
        return userUseCase.getUser(id);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(@PathVariable Long id) {
        userUseCase.deleteUser(id);
    }
}
```

### A02: Cryptographic Failures

```
□ 비밀번호 안전한 해시 사용 (BCrypt, Argon2)
□ 민감 데이터 전송 시 TLS/HTTPS
□ 하드코딩된 암호화 키 없음
□ 적절한 암호화 알고리즘 사용
□ 민감 데이터 암호화 저장
□ 암호화 키 안전한 관리
```

**안전한 코드 예시:**

```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // strength 12
    }
}

@Service
public class UserService {

    private final PasswordEncoder passwordEncoder;

    public void createUser(CreateUserCommand command) {
        String encodedPassword = passwordEncoder.encode(command.password());
        // 해시된 비밀번호 저장
    }
}
```

### A03: Injection

```
□ SQL 파라미터 바인딩 사용
□ JPA/Hibernate 사용 시 JPQL 파라미터
□ Native Query 사용 시 검증
□ 외부 명령 실행 금지
□ XSS 방지 (출력 인코딩)
□ LDAP Injection 방지
```

**안전한 코드 예시:**

```java
// ✅ 안전: 파라미터 바인딩
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);

// ✅ 안전: JPA Criteria
public List<User> searchUsers(String name) {
    return queryFactory
        .selectFrom(user)
        .where(user.name.containsIgnoreCase(name))
        .fetch();
}

// ❌ 위험: 문자열 연결
String query = "SELECT * FROM users WHERE email = '" + email + "'";
```

### A04: Insecure Design

```
□ 입력값 검증 (Bean Validation)
□ 비즈니스 로직 보안 (금액 검증 등)
□ Rate Limiting 적용
□ 적절한 에러 처리
□ 보안 로깅
□ Fail-safe 기본값
```

**안전한 코드 예시:**

```java
public record TransferCommand(
    @NotNull Long fromAccountId,
    @NotNull Long toAccountId,
    @Positive @DecimalMax("10000000") BigDecimal amount
) {}

@Service
public class TransferService {

    @RateLimiter(name = "transfer", fallbackMethod = "transferFallback")
    public void transfer(TransferCommand command) {
        Account from = accountRepository.findById(command.fromAccountId())
            .orElseThrow(() -> new AccountNotFoundException(command.fromAccountId()));

        // 잔액 검증
        if (from.getBalance().compareTo(command.amount()) < 0) {
            throw new InsufficientBalanceException();
        }

        // 이체 실행
    }
}
```

### A05: Security Misconfiguration

```
□ 기본 계정/비밀번호 변경
□ 불필요한 기능 비활성화
□ 에러 메시지에 스택 트레이스 노출 금지
□ 보안 헤더 설정
□ 디버그 모드 비활성화 (프로덕션)
□ 액추에이터 엔드포인트 제한
```

**안전한 설정 예시:**

```yaml
# application-prod.yml
spring:
  jpa:
    show-sql: false
  devtools:
    restart:
      enabled: false

management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus
  endpoint:
    health:
      show-details: never

server:
  error:
    include-stacktrace: never
    include-message: never
```

### A06: Vulnerable Components

```
□ 의존성 버전 최신화
□ 알려진 취약점 검사
□ 불필요한 의존성 제거
□ SBOM(Software Bill of Materials) 관리
```

**검사 도구:**

```groovy
// build.gradle
plugins {
    id 'org.owasp.dependencycheck' version '8.4.0'
}

dependencyCheck {
    failBuildOnCVSS = 7
    suppressionFile = 'config/dependency-check-suppression.xml'
}
```

### A07: Authentication Failures

```
□ 강력한 비밀번호 정책
□ 계정 잠금 정책
□ 세션 관리
□ 토큰 만료 설정
□ 로그아웃 처리
□ MFA 지원 (해당 시)
```

**안전한 코드 예시:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .logout(logout -> logout
                .logoutUrl("/api/auth/logout")
                .logoutSuccessHandler(customLogoutSuccessHandler())
            );
        return http.build();
    }
}

// 계정 잠금
@Service
public class LoginAttemptService {

    private static final int MAX_ATTEMPTS = 5;
    private final Cache<String, Integer> attemptsCache;

    public void loginFailed(String email) {
        int attempts = attemptsCache.get(email, () -> 0) + 1;
        attemptsCache.put(email, attempts);

        if (attempts >= MAX_ATTEMPTS) {
            throw new AccountLockedException("Account locked due to too many failed attempts");
        }
    }
}
```

### A08: Software Integrity Failures

```
□ 의존성 무결성 검증
□ CI/CD 파이프라인 보안
□ 서명된 빌드
□ 안전한 배포 프로세스
```

### A09: Security Logging Failures

```
□ 인증 실패 로깅
□ 권한 거부 로깅
□ 민감정보 로깅 금지
□ 로그 무결성 보장
□ 로그 접근 제어
```

**안전한 코드 예시:**

```java
@Slf4j
@Aspect
@Component
public class SecurityAuditAspect {

    @AfterThrowing(
        pointcut = "execution(* com.example..*.login(..))",
        throwing = "ex"
    )
    public void logAuthenticationFailure(JoinPoint jp, AuthenticationException ex) {
        // ✅ 안전: 비밀번호 제외
        log.warn("Authentication failed for user: {}, reason: {}",
            getEmail(jp), ex.getMessage());
    }

    // ❌ 위험: 비밀번호 포함
    // log.warn("Login failed: email={}, password={}", email, password);
}
```

### A10: SSRF (Server-Side Request Forgery)

```
□ 외부 URL 요청 시 화이트리스트 검증
□ 내부 네트워크 접근 방지
□ URL 리다이렉트 검증
□ DNS Rebinding 방지
```

**안전한 코드 예시:**

```java
@Service
public class UrlFetchService {

    private static final Set<String> ALLOWED_HOSTS = Set.of(
        "api.example.com",
        "cdn.example.com"
    );

    public String fetch(String url) {
        URI uri = URI.create(url);

        if (!ALLOWED_HOSTS.contains(uri.getHost())) {
            throw new SecurityException("URL not in whitelist: " + uri.getHost());
        }

        if (isInternalAddress(uri.getHost())) {
            throw new SecurityException("Internal addresses not allowed");
        }

        return restTemplate.getForObject(url, String.class);
    }

    private boolean isInternalAddress(String host) {
        // 127.0.0.1, localhost, 10.x.x.x, 192.168.x.x 등 체크
    }
}
```

## Spring Security 설정

### 기본 보안 설정

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // CSRF
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                // REST API인 경우 비활성화
                // .disable()
            )

            // 보안 헤더
            .headers(headers -> headers
                .frameOptions(frame -> frame.deny())
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives("default-src 'self'"))
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000))
            )

            // 인증
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )

            // 세션
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        return http.build();
    }
}
```

## 빠른 체크리스트

### 코드 리뷰 시

```
□ SQL 쿼리에 파라미터 바인딩 사용?
□ 사용자 입력값 검증?
□ 권한 검증 존재?
□ 민감정보 로깅 없음?
□ 하드코딩된 시크릿 없음?
□ 예외 처리 시 정보 노출 없음?
```

### 배포 전

```
□ 디버그 모드 비활성화?
□ 프로덕션 프로파일 설정?
□ 시크릿 외부화?
□ HTTPS 강제?
□ 보안 헤더 설정?
□ 의존성 취약점 검사?
```

## 연관 에이전트

- **security-reviewer**: 이 체크리스트로 보안 리뷰
- **code-reviewer**: 일반 리뷰에 보안 항목 포함
