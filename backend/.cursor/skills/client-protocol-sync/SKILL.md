---
name: client-protocol-sync
description: Swagger/OpenAPI를 분석하여 *-client, *-protocol 모듈에 HttpExchange 기반 코드를 자동 생성합니다.
---

# Client Protocol Sync Skill

Swagger/OpenAPI 명세를 기반으로 `*-client`, `*-protocol` 모듈에 HttpExchange 스택 코드를 자동 생성합니다.

## 사용법

```
/client-protocol-sync
```

실행 시 Swagger URL을 입력받습니다:
- **전체 URL**: 루트 swagger-ui URL 제공 시 모든 API 생성
- **부분 URL**: 특정 태그 또는 단일 API 링크 제공 시 해당 부분만 생성

## 워크플로우

```
┌──────────────────────────────────────────────────────────────┐
│                  /client-protocol-sync                        │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │  Swagger URL 입력 요청 │
                └───────────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   OpenAPI 분석        │
                │   - 엔드포인트 수 확인  │
                │   - 스키마 추출        │
                └───────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
    ┌───────────────┐               ┌───────────────┐
    │ API >= 10개    │               │ API < 10개    │
    │ → 확인 절차    │               │ → 바로 진행   │
    └───────────────┘               └───────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   대상 모듈 탐색       │
                │   - *-client yml 검색 │
                │   - baseUrl 매칭      │
                └───────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
    ┌───────────────┐               ┌───────────────┐
    │ 모듈 발견      │               │ 모듈 미발견   │
    │ → 코드 생성   │               │ → 확인 절차   │
    └───────────────┘               └───────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   코드 생성           │
                │   - openapi-generator │
                │   - 패턴 적용         │
                │   - 불필요 파일 제거   │
                └───────────────────────┘
                            │
                            ▼
                ┌───────────────────────┐
                │   Upsert 완료         │
                └───────────────────────┘
```

## 확인 절차

### API 개수 확인 (전체 URL 제공 시)

```
전체 swagger를 분석했습니다.
생성할 API 개수: 15개

진행하시겠습니까?
- [Y] 전체 생성
- [N] 특정 태그/API만 선택
- [C] 취소
```

### 대상 모듈 미발견 시

```
Swagger baseUrl: https://api.example.com
매칭되는 *-client 모듈을 찾지 못했습니다.

어떤 모듈에 생성할까요?
- 기존 모듈 목록: [point-client, order-client, ...]
- 새 모듈 생성: {모듈명} 입력
```

## 코드 생성 규칙

### Client 모듈 (*-client)

#### 인터페이스 패턴

```java
public interface {Domain}ApiClient {

    CompletableFuture<{Response}> {methodName}({Request} request);
}
```

#### 구현체 패턴

```java
@Slf4j
@RequiredArgsConstructor
public class {Domain}ApiClientImpl implements {Domain}ApiClient {

    private final Raw{Domain}ApiClient client;

    @Override
    @Retry(name = "{domain}-client")
    @TimeLimiter(name = "{domain}-client")
    public CompletableFuture<{Response}> {methodName}(@Valid {Request} request) {
        return client.{methodName}(request);
    }

    public interface Raw{Domain}ApiClient {

        @{HttpMethod}Exchange("{path}")
        CompletableFuture<{Response}> {methodName}(@RequestBody {Request} request);
    }
}
```

### Protocol 모듈 (*-protocol)

#### Request 패턴

```java
@Schema(description = "{설명}")
@Builder(toBuilder = true)
public record {Name}Req(
        @NotBlank
        @Schema(description = "{필드 설명}")
        String field1,

        @NotNull
        @Schema(description = "{필드 설명}")
        Long field2
) {
}
```

#### Response 패턴

```java
@Schema(description = "{설명}")
@Builder(toBuilder = true)
public record {Name}Res(
        @Schema(description = "{필드 설명}")
        Long id,

        @Schema(description = "{필드 설명}")
        String name
) {
}
```

#### Enum 패턴

```java
@Schema(description = "{설명}")
@Getter
@FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)
@RequiredArgsConstructor
public enum {EnumName} {

    VALUE1("value1"),
    VALUE2("value2");

    String text;
}
```

## OpenAPI Generator 설정

### 사용할 Generator

```bash
# httpexchange 기반 생성
openapi-generator-cli generate \
  -i {swagger-url}/v3/api-docs \
  -g java \
  --library webclient \
  -o ./generated \
  --additional-properties=\
    useJakartaEe=true,\
    dateLibrary=java8,\
    openApiNullable=false,\
    useBeanValidation=true
```

### 생성 후 처리

```
생성 파일 중 유지:
├── *Api.java        → Raw*ApiClient 참조용
├── *Req.java        → protocol 모듈로 이동
├── *Res.java        → protocol 모듈로 이동
└── *Enum.java       → protocol 모듈로 이동

삭제 대상:
├── ApiClient.java
├── Configuration.java
├── auth/
├── model/ (변환 후)
└── 기타 설정 파일들
```

## 파일 구조

### Client 모듈

```
client/{name}-client/
├── src/main/java/{package}/
│   ├── {Name}ClientModule.java
│   ├── autoconfigure/
│   │   └── {Name}ClientAutoConfig.java
│   ├── client/
│   │   └── {domain}/
│   │       ├── {Domain}ApiClient.java
│   │       └── {Domain}ApiClientImpl.java
│   └── config/
│       ├── {Name}ClientConfig.java
│       └── {Name}ClientProperties.java
└── src/main/resources/
    └── config/
        └── application-client-{name}.yml
```

### Protocol 모듈

```
client/{name}-protocol/
└── src/main/java/{package}/protocol/
    ├── {domain}/
    │   ├── request/
    │   │   └── {Name}Req.java
    │   └── response/
    │       └── {Name}Res.java
    └── common/
        └── enums/
            └── {EnumName}.java
```

## Upsert 규칙

### 기존 파일 존재 시

1. **인터페이스**: 새 메서드만 추가 (기존 유지)
2. **구현체**: 새 메서드 추가 + Raw 인터페이스에 메서드 추가
3. **DTO**: 필드 비교 후 새 필드 추가 (기존 유지)
4. **Enum**: 새 값 추가 (기존 유지)

### 충돌 처리

```
기존 메서드와 시그니처가 다른 경우:
→ 기존 유지 + 새 버전 주석으로 표시

// TODO: OpenAPI 변경사항 확인 필요
// New signature: CompletableFuture<NewRes> methodName(NewReq request);
```

## 모듈 탐색 로직

### baseUrl 매칭

```yaml
# application-client-{name}.yml 검색
# 또는 application-client-{name}-httpexchange.yml

# Properties 클래스에서 baseUrl 확인
@ConfigurationProperties("client.{name}")
public class {Name}ClientProperties {
    private String baseUrl;  # 이 값과 Swagger host 매칭
}
```

### 탐색 순서

1. `client/*/src/main/resources/config/*.yml` 검색
2. yml 내 `base-url` 또는 관련 설정 확인
3. Swagger host와 비교하여 매칭 모듈 찾기

## 예시

### 입력

```
/client-protocol-sync

> 스웨거 URL을 입력해주세요:
https://point-api.example.com/swagger-ui/index.html#/Transaction%20API/issue
```

### 출력

```
✅ OpenAPI 분석 완료
- 대상 API: POST /v1/transactions/issue
- Request: PointIssueReq
- Response: PointIssueRes

✅ 대상 모듈 확인
- Client: client/point-client
- Protocol: client/point-protocol

✅ 코드 생성 완료
- [U] PointTransactionApiClient.java (메서드 추가)
- [U] PointTransactionApiClientImpl.java (메서드 추가)
- [C] PointIssueReq.java (신규 생성)
- [C] PointIssueRes.java (신규 생성)

[U] = Updated, [C] = Created
```

## 연관 스킬

- `java-code-style`: 생성된 코드의 스타일 검증
- `verification-loop`: 빌드 검증
