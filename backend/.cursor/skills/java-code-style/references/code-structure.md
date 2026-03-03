# 코드 구조 및 패턴

## 접근 제어

### public 사용 규칙

**public으로 선언해야 하는 것**:
- UseCase 인터페이스
- Command record
- Query record
- Result record
- Event record
- Mapper 인터페이스
- Policy 인터페이스

**예시**:
```java
// public 인터페이스
public interface CreateExampleUsecase {
    ExampleResult execute(CreateExampleCommand command);
}

// public record
public record CreateExampleCommand(String name) {}
public record ExampleResult(String id, String name) {}
```

### package-private 사용 규칙

**package-private으로 선언해야 하는 것**:
- Adapter (UseCase 구현체)
- Policy 구현체 (대부분)

**예시**:
```java
// package-private Adapter
@Service
@RequiredArgsConstructor
class ExampleCoreService implements CreateExampleUsecase {
    // 구현
}

// package-private Policy 구현체
@Component
class DefaultExampleCreatePolicy implements ExampleCreatePolicy {
    // 구현
}
```

### 접근 제어 원칙

- **최소 권한 원칙**: 필요한 경우에만 public
- **인터페이스와 구현체 분리**: 인터페이스는 public, 구현체는 package-private
- **데이터 클래스는 public**: API 경계를 넘나드는 데이터는 public

## Java 21 Features

### record 사용

**사용 대상**:
- Command
- Query
- Result
- Event
- Domain Model (불변 객체)

**기본 패턴**:
```java
public record CreateExampleCommand(String name, String description) {
    // Compact Constructor
    public CreateExampleCommand {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name required");
        }
    }
}
```

**@Builder와 함께 사용**:
```java
@Builder
public record ExampleResult(
    String id,
    String name,
    String description,
    Instant createdAt
) {}
```

**toBuilder = true 패턴**:
```java
@Builder(toBuilder = true)
public record StepResult(
    String id,
    String name,
    // 보강 정보
    String jobPostingId,
    Boolean hasProgress
) {}

// 사용
StepResult enriched = baseResult.toBuilder()
    .jobPostingId("12345")
    .hasProgress(true)
    .build();
```

### text block 사용

**SQL 쿼리**:
```java
String sql = """
    SELECT e.id, e.name, e.description
    FROM example e
    WHERE e.deleted_at IS NULL
    ORDER BY e.created_at DESC
    """;
```

**JSON**:
```java
String json = """
    {
        "name": "%s",
        "description": "%s",
        "metadata": {
            "version": "1.0"
        }
    }
    """.formatted(name, description);
```

**HTML/템플릿**:
```java
String html = """
    <!DOCTYPE html>
    <html>
    <body>
        <h1>%s</h1>
        <p>%s</p>
    </body>
    </html>
    """.formatted(title, content);
```

### pattern matching switch

**플랫폼별 분기**:
```java
CompletableFuture<Response> future = switch (command.platform()) {
    case JOBKOREA -> atsClient.getApplicants(params);
    case ALBAMON -> jobHubClient.getApplicants(params);
    case SARAMIN -> saraminClient.getApplicants(params);
};
```

**타입별 처리**:
```java
String message = switch (errorCode) {
    case BAD_REQUEST -> "잘못된 요청입니다";
    case NOT_FOUND -> "리소스를 찾을 수 없습니다";
    case FORBIDDEN -> "권한이 없습니다";
    default -> "오류가 발생했습니다";
};
```

**sealed 클래스와 함께** (가능한 경우):
```java
sealed interface Shape permits Circle, Rectangle, Triangle {}

double area = switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
    case Triangle t -> 0.5 * t.base() * t.height();
};
```

## Adapter 구조 패턴

### 기본 Adapter 패턴

**표준 구조**:
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

    private final ExampleDomainService exampleDomainService;
    private final ExampleCreatePolicy exampleCreatePolicy;
    private final ExampleUpdatePolicy exampleUpdatePolicy;
    private final ExampleCoreMapper coreMapper;
    private final DomainEventPublisher domainEventPublisher;

    @Override
    @Transactional
    public ExampleResult execute(CreateExampleCommand command) {
        log.info("Creating example: {}", command);

        // 1. 정책 검증
        exampleCreatePolicy.validate(command);

        // 2. Command → Domain Model 변환
        Example model = coreMapper.toDomainModel(command);

        // 3. Domain Service 호출
        Example created = exampleDomainService.create(model);

        // 4. 도메인 이벤트 발행
        publishExampleCreatedEvent(created);

        // 5. Domain Model → Result 변환
        ExampleResult result = coreMapper.toResult(created);

        log.info("Example created: id={}", result.id());
        return result;
    }

    @Override
    public ExampleResult execute(GetExampleQuery query) {
        log.debug("Getting example: id={}", query.id());

        Example example = exampleDomainService.findById(query.id());
        return coreMapper.toResult(example);
    }

    @Override
    public List<ExampleResult> execute(GetExamplesQuery query) {
        log.debug("Getting all examples");

        List<Example> examples = exampleDomainService.findAll();
        return coreMapper.toResults(examples);
    }

    @Override
    @Transactional
    public ExampleResult execute(UpdateExampleCommand command) {
        log.info("Updating example: id={}", command.id());

        exampleUpdatePolicy.validate(command);

        Example existing = exampleDomainService.findById(command.id());
        Example updated = coreMapper.toUpdatedModel(existing, command);
        Example saved = exampleDomainService.update(updated);

        publishExampleUpdatedEvent(saved);

        return coreMapper.toResult(saved);
    }

    private void publishExampleCreatedEvent(Example example) {
        var context = DefaultExampleEventContext.of(
            example.id(),
            example.name()
        );

        var event = ExampleCreatedEventV1.builder()
            ._context(context)
            .source("bizcenter-ats")
            .build();

        domainEventPublisher.publish(event);
    }

    private void publishExampleUpdatedEvent(Example example) {
        // 이벤트 발행 로직
    }
}
```

### 역할별 Adapter 분리 패턴

**기본 CRUD Adapter**:
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ApplicantProgressCoreService implements
        CreateApplicantProgressUsecase,
        GetApplicantProgressUsecase,
        UpdateApplicantProgressUsecase {

    private final ApplicantProgressDomainService domainService;
    private final ApplicantProgressCoreMapper mapper;
    private final ApplicantProgressPolicy policy;

    // 기본 CRUD 구현
}
```

**Worker 전용 Adapter**:
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ApplicantProgressWorkerCoreService implements
        SyncApplicantProgressFromExternalUsecase,
        ProcessApplicantProgressEventUsecase {

    private final AtsClient atsClient;
    private final JobHubClient jobHubClient;
    private final ApplicantProgressDomainService domainService;

    @Transactional
    public void execute(SyncCommand command) {
        // 외부 API 호출
        CompletableFuture<Response> future = switch (command.platform()) {
            case JOBKOREA -> atsClient.getApplicants(command.params());
            case ALBAMON -> jobHubClient.getApplicants(command.params());
        };

        Response response = future.join();
        domainService.syncApplicants(response.data());
    }
}
```

**Download/Export 전용 Adapter**:
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ApplicantProgressDownloadCoreService implements
        DownloadApplicantProgressUsecase,
        ExportApplicantProgressToExcelUsecase {

    private final ApplicantProgressDomainService domainService;
    private final ExcelExportService excelService;

    @Override
    public byte[] execute(DownloadCommand command) {
        // 대용량 조회 및 파일 생성
        List<ApplicantProgress> progresses = domainService.findAllByFilter(command.filter());
        return excelService.createExcel(progresses);
    }
}
```

## Command/Query/Result 내부 클래스 패턴

간단한 feature는 UseCase 인터페이스에 record를 내부 클래스로 정의:

### 내부 클래스 패턴

```java
package bizcenter.core.step.usecase;

import java.util.List;

/**
 * Step 업서트 UseCase
 */
public interface UpsertStepsUseCase {

    /**
     * Step 업서트 Command
     */
    record Command(
        String jobPostingMappingId,
        List<StepInput> steps
    ) {
        public Command {
            if (jobPostingMappingId == null || jobPostingMappingId.isBlank()) {
                throw new IllegalArgumentException("Job posting mapping ID required");
            }
            if (steps == null || steps.isEmpty()) {
                throw new IllegalArgumentException("Steps required");
            }
        }

        public record StepInput(
            String id,
            String name,
            Integer ordering,
            String stepType
        ) {}
    }

    /**
     * Step 업서트 Result
     */
    record Result(
        String id,
        String name,
        Integer ordering,
        String stepType,
        String jobPostingMappingId
    ) {}

    /**
     * Step 업서트 실행
     */
    List<Result> execute(Command command);
}
```

### 사용 시점

**내부 클래스 패턴 사용**:
- 간단한 CRUD 작업
- Command/Query/Result가 짧은 경우 (각 5개 필드 이하)
- Policy가 없거나 간단한 경우

**별도 파일 패턴 사용**:
- 복잡한 비즈니스 로직
- Command/Query/Result가 긴 경우 (5개 필드 초과)
- Policy가 복잡한 경우
- Event 발행이 필요한 경우

## Lombok 어노테이션

### Adapter/Service 클래스

```java
@Slf4j                    // 로깅
@Service                  // Spring Bean
@RequiredArgsConstructor  // 생성자 주입
@Transactional(readOnly = true)  // 기본 읽기 전용
class ExampleCoreService {
    // final 필드는 생성자 주입됨
}
```

### record와 @Builder

```java
@Builder
public record ExampleResult(
    String id,
    String name
) {}

// 사용
ExampleResult result = ExampleResult.builder()
    .id("123")
    .name("Test")
    .build();
```

### @Builder(toBuilder = true)

```java
@Builder(toBuilder = true)
public record StepResult(
    String id,
    String name,
    String jobPostingId  // 보강 필드
) {}

// 사용
StepResult base = StepResult.builder()
    .id("123")
    .name("Step 1")
    .build();

StepResult enriched = base.toBuilder()
    .jobPostingId("job-456")
    .build();
```

## MapStruct 패턴

### 기본 Mapper 인터페이스

```java
@Mapper
public interface ExampleCoreMapper {

    // 기본 변환 (MapStruct 자동 생성)
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    Example toDomainModel(CreateExampleCommand command);

    ExampleResult toResult(Example model);

    List<ExampleResult> toResults(List<Example> models);

    // default 메서드로 수동 변환
    default Example toNewExample(String name, String description) {
        return Example.builder()
                .name(name.trim())
                .description(description)
                .build();
    }
}
```

### Mapper에서 도메인 모델 생성 응집

Lombok builder를 사용하는 코드는 Mapper 내부로 응집:

```java
@Mapper
public interface StepCoreMapper {

    // 기본 변환
    StepResult toResult(Step model);
    List<StepResult> toResults(List<Step> models);

    // 도메인 모델 생성 로직 응집
    default Step toNewStep(
            String name,
            Integer ordering,
            String stepType,
            String jobPostingMappingId) {
        return Step.builder()
                .name(name.trim())
                .ordering(ordering)
                .stepType(stepType)
                .jobPostingMappingId(jobPostingMappingId)
                .build();
    }

    default Step toUpdatedStep(Step existing, String newName) {
        return Step.builder()
                .id(existing.id())
                .name(newName.trim())
                .ordering(existing.ordering())
                .stepType(existing.stepType())
                .jobPostingMappingId(existing.jobPostingMappingId())
                .createdAt(existing.createdAt())
                .createdBy(existing.createdBy())
                .updatedAt(Instant.now())
                .updatedBy(getCurrentUser())
                .build();
    }

    // Result 보강
    default StepResult toEnrichedResult(
            StepResult baseResult,
            String jobPostingId,
            String workspaceId,
            Boolean hasProgress) {
        return baseResult.toBuilder()
                .jobPostingId(jobPostingId)
                .workspaceId(workspaceId)
                .hasApplicantProgress(hasProgress)
                .build();
    }
}
```

### MapStruct와 Lombok 함께 사용

`build.gradle.kts`:
```kotlin
dependencies {
    // MapStruct
    implementation("org.mapstruct:mapstruct")
    annotationProcessor("org.mapstruct:mapstruct-processor")

    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok-mapstruct-binding:0.2.0")
}
```

**중요**: `lombok-mapstruct-binding` 필수!

## Policy 패턴

### 단순 검증 Policy

```java
public interface ExampleCreatePolicy {
    void validate(CreateExampleCommand command);
}

@Slf4j
@Component
@RequiredArgsConstructor
class DefaultExampleCreatePolicy implements ExampleCreatePolicy {

    @Override
    public void validate(CreateExampleCommand command) {
        if (command.name().length() > 126) {
            throw new JobkoreaBusinessException(
                CommonErrorCode.BAD_REQUEST,
                "Name cannot exceed 126 characters"
            );
        }
    }
}
```

### PolicyViolation 활용 패턴

여러 검증 오류를 모아서 한 번에 처리:

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class StepUpsertPolicyImpl implements StepUpsertPolicy {

    @Override
    public void validate(StepUpsertValidationInput input) {
        List<PolicyViolation> violations = new ArrayList<>();

        // 단계별 검증
        validateStepsCount(input.steps(), violations);
        if (!violations.isEmpty()) {
            throwBusinessException(violations);
        }

        validateRequiredFields(input.steps(), violations);
        validateNoDuplicates(input.steps(), violations);
        validateStepTypes(input.steps(), violations);

        if (!violations.isEmpty()) {
            throwBusinessException(violations);
        }
    }

    private void validateStepsCount(List<StepInput> steps, List<PolicyViolation> violations) {
        if (steps.size() > 20) {
            violations.add(PolicyViolation.of("Steps cannot exceed 20"));
        }
    }

    private void validateRequiredFields(List<StepInput> steps, List<PolicyViolation> violations) {
        for (int i = 0; i < steps.size(); i++) {
            StepInput step = steps.get(i);
            if (step.name() == null || step.name().isBlank()) {
                violations.add(PolicyViolation.of("Step[%d].name is required".formatted(i)));
            }
        }
    }

    private void throwBusinessException(List<PolicyViolation> violations) {
        String message = violations.stream()
                .map(PolicyViolation::getErrorMessage)
                .reduce((a, b) -> a + ", " + b)
                .orElse("Validation failed");

        throw new JobkoreaBusinessException(CommonErrorCode.BAD_REQUEST, message);
    }
}
```

## 로깅 패턴

### 로깅 레벨

- `log.debug()`: 개발 중 디버깅 정보
- `log.info()`: 중요한 비즈니스 이벤트 (생성, 수정, 삭제)
- `log.warn()`: 경고 (예상 가능한 오류)
- `log.error()`: 에러 (예상하지 못한 오류)

### Adapter 로깅

```java
@Override
@Transactional
public ExampleResult execute(CreateExampleCommand command) {
    log.info("Creating example: name={}", command.name());

    exampleCreatePolicy.validate(command);
    Example model = coreMapper.toDomainModel(command);
    Example created = exampleDomainService.create(model);
    publishExampleCreatedEvent(created);

    log.info("Example created successfully: id={}", created.id());
    return coreMapper.toResult(created);
}

@Override
public ExampleResult execute(GetExampleQuery query) {
    log.debug("Getting example: id={}", query.id());

    Example example = exampleDomainService.findById(query.id());
    return coreMapper.toResult(example);
}
```

### Policy 로깅

```java
@Override
public void validate(CreateExampleCommand command) {
    log.debug("Validating create example command: {}", command);

    // 검증 로직

    log.debug("Validation passed for command: {}", command);
}
```

## 금지 사항

### 절대 금지

- ❌ `System.out.println()` 사용 (log 사용)
- ❌ Entity를 API Layer에 직접 노출
- ❌ MapperImpl.java 파일 커밋
- ❌ public 접근자 남용 (필요한 경우만)
- ❌ 순환 의존 (API ↔ Core, Core ↔ Domain)

### 지양해야 할 패턴

- ❌ 긴 메서드 (30줄 초과)
- ❌ 깊은 중첩 (3단계 초과)
- ❌ Magic Number (상수로 추출)
- ❌ 주석으로 코드 설명 (자명한 코드 작성)
- ❌ 불필요한 else 절 (early return 활용)

**좋은 예**:
```java
@Override
public ExampleResult execute(GetExampleQuery query) {
    Example example = domainService.findById(query.id());
    if (example == null) {
        throw new NotFoundException("Example not found: " + query.id());
    }
    return mapper.toResult(example);
}
```

**나쁜 예**:
```java
@Override
public ExampleResult execute(GetExampleQuery query) {
    Example example = domainService.findById(query.id());
    if (example != null) {
        return mapper.toResult(example);
    } else {
        throw new NotFoundException("Example not found: " + query.id());
    }
}
```
