# 코드 예시 모음

## UseCase 인터페이스

### 생성 (Create)

```java
package bizcenter.core.example.usecase;

import bizcenter.core.example.model.command.CreateExampleCommand;
import bizcenter.core.example.model.result.ExampleResult;

/**
 * Example 생성 UseCase
 */
public interface CreateExampleUsecase {
    /**
     * Example 생성
     *
     * @param command 생성 명령
     * @return 생성된 Example 결과
     */
    ExampleResult execute(CreateExampleCommand command);
}
```

### 조회 (Get)

```java
package bizcenter.core.example.usecase;

import bizcenter.core.example.model.query.GetExampleQuery;
import bizcenter.core.example.model.result.ExampleResult;

/**
 * Example 단건 조회 UseCase
 */
public interface GetExampleUsecase {
    /**
     * Example 조회
     *
     * @param query 조회 쿼리
     * @return Example 결과
     */
    ExampleResult execute(GetExampleQuery query);
}
```

### 목록 조회 (Get List)

```java
package bizcenter.core.example.usecase;

import bizcenter.core.example.model.query.GetExamplesQuery;
import bizcenter.core.example.model.result.ExampleResult;
import java.util.List;

/**
 * Example 목록 조회 UseCase
 */
public interface GetExamplesUsecase {
    /**
     * Example 목록 조회
     *
     * @param query 조회 쿼리
     * @return Example 결과 목록
     */
    List<ExampleResult> execute(GetExamplesQuery query);
}
```

### 수정 (Update)

```java
package bizcenter.core.example.usecase;

import bizcenter.core.example.model.command.UpdateExampleCommand;
import bizcenter.core.example.model.result.ExampleResult;

/**
 * Example 수정 UseCase
 */
public interface UpdateExampleUsecase {
    /**
     * Example 수정
     *
     * @param command 수정 명령
     * @return 수정된 Example 결과
     */
    ExampleResult execute(UpdateExampleCommand command);
}
```

### 삭제 (Delete)

```java
package bizcenter.core.example.usecase;

import bizcenter.core.example.model.command.DeleteExampleCommand;

/**
 * Example 삭제 UseCase
 */
public interface DeleteExampleUsecase {
    /**
     * Example 삭제
     *
     * @param command 삭제 명령
     */
    void execute(DeleteExampleCommand command);
}
```

## Command/Query/Result

### Command (별도 파일)

```java
package bizcenter.core.example.model.command;

/**
 * Example 생성 Command
 */
public record CreateExampleCommand(
    String name,
    String description
) {
    /**
     * Compact Constructor로 검증
     */
    public CreateExampleCommand {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (description == null || description.isBlank()) {
            throw new IllegalArgumentException("Description cannot be empty");
        }
        if (name.length() > 126) {
            throw new IllegalArgumentException("Name cannot exceed 126 characters");
        }
    }
}
```

### Query (별도 파일)

```java
package bizcenter.core.example.model.query;

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
public record GetExamplesQuery(
    String name,  // 이름 필터 (선택)
    Boolean includeDeleted  // 삭제된 항목 포함 여부 (선택)
) {
    public GetExamplesQuery {
        // 검증 로직 (선택적 필드는 null 허용)
    }
}
```

### Result (별도 파일)

```java
package bizcenter.core.example.model.result;

import lombok.Builder;
import java.time.Instant;

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
) {}
```

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

        /**
         * Step 입력 정보
         */
        public record StepInput(
            String id,           // 기존 ID (업데이트 시)
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
     *
     * @param command 업서트 명령
     * @return 업서트된 Step 결과 목록
     */
    List<Result> execute(Command command);
}
```

## Adapter (UseCase 구현체)

### 기본 Adapter

```java
package bizcenter.core.example.adapter;

import bizcenter.core.example.event.ExampleCreatedEventV1;
import bizcenter.core.example.event.ExampleUpdatedEventV1;
import bizcenter.core.example.event.base.DefaultExampleEventContext;
import bizcenter.core.example.mapper.ExampleCoreMapper;
import bizcenter.core.example.model.command.CreateExampleCommand;
import bizcenter.core.example.model.command.UpdateExampleCommand;
import bizcenter.core.example.model.query.GetExampleQuery;
import bizcenter.core.example.model.query.GetExamplesQuery;
import bizcenter.core.example.model.result.ExampleResult;
import bizcenter.core.example.policy.ExampleCreatePolicy;
import bizcenter.core.example.policy.ExampleUpdatePolicy;
import bizcenter.core.example.usecase.CreateExampleUsecase;
import bizcenter.core.example.usecase.GetExampleUsecase;
import bizcenter.core.example.usecase.GetExamplesUsecase;
import bizcenter.core.example.usecase.UpdateExampleUsecase;
import bizcenter.domain.example.model.Example;
import bizcenter.domain.example.service.ExampleDomainService;
import java.util.List;
import com.example.common.event.client.publisher.DomainEventPublisher;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * Example Core Adapter
 * <p>
 * 모든 Example UseCase를 구현하며, 트랜잭션 경계를 관리합니다.
 */
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ExampleCoreService implements
        CreateExampleUsecase,
        GetExampleUsecase,
        GetExamplesUsecase,
        UpdateExampleUsecase {

    private static final String SOURCE = "bizcenter-ats";

    private final ExampleDomainService exampleDomainService;
    private final ExampleCreatePolicy exampleCreatePolicy;
    private final ExampleUpdatePolicy exampleUpdatePolicy;
    private final ExampleCoreMapper coreMapper;
    private final DomainEventPublisher domainEventPublisher;

    /**
     * Example 생성
     */
    @Override
    @Transactional
    public ExampleResult execute(CreateExampleCommand command) {
        log.info("Creating example with command: {}", command);

        // 1. 정책 검증
        exampleCreatePolicy.validate(command);

        // 2. Command -> Domain Model 변환
        Example domainModel = coreMapper.toDomainModel(command);

        // 3. Domain Service 호출
        Example created = exampleDomainService.create(domainModel);

        // 4. 도메인 이벤트 발행
        publishExampleCreatedEvent(created);

        // 5. Domain Model -> Result 변환
        ExampleResult result = coreMapper.toResult(created);

        log.info("Example created successfully: id={}", result.id());
        return result;
    }

    /**
     * Example 단건 조회
     */
    @Override
    public ExampleResult execute(GetExampleQuery query) {
        log.debug("Getting example with query: {}", query);

        Example example = exampleDomainService.findById(query.id());
        return coreMapper.toResult(example);
    }

    /**
     * Example 목록 조회
     */
    @Override
    public List<ExampleResult> execute(GetExamplesQuery query) {
        log.debug("Getting examples with query: {}", query);

        List<Example> examples = exampleDomainService.findAll();
        return coreMapper.toResults(examples);
    }

    /**
     * Example 수정
     */
    @Override
    @Transactional
    public ExampleResult execute(UpdateExampleCommand command) {
        log.info("Updating example with command: {}", command);

        // 1. 정책 검증
        exampleUpdatePolicy.validate(command);

        // 2. 기존 Example 조회
        Example existing = exampleDomainService.findById(command.id());

        // 3. Command + 기존 Model -> 업데이트 Model
        Example updated = coreMapper.toUpdatedModel(existing, command);

        // 4. Domain Service 호출
        Example saved = exampleDomainService.update(updated);

        // 5. 도메인 이벤트 발행
        publishExampleUpdatedEvent(saved);

        // 6. Domain Model -> Result 변환
        ExampleResult result = coreMapper.toResult(saved);

        log.info("Example updated successfully: id={}", result.id());
        return result;
    }

    /**
     * Example 생성 이벤트 발행
     */
    private void publishExampleCreatedEvent(Example example) {
        var context = DefaultExampleEventContext.of(
                example.id(),
                example.name(),
                example.description()
        );

        var event = ExampleCreatedEventV1.builder()
                ._context(context)
                .source(SOURCE)
                .build();

        domainEventPublisher.publish(event);
        log.debug("Published ExampleCreatedEvent: id={}", example.id());
    }

    /**
     * Example 수정 이벤트 발행
     */
    private void publishExampleUpdatedEvent(Example example) {
        var context = DefaultExampleEventContext.of(
                example.id(),
                example.name(),
                example.description()
        );

        var event = ExampleUpdatedEventV1.builder()
                ._context(context)
                .source(SOURCE)
                .build();

        domainEventPublisher.publish(event);
        log.debug("Published ExampleUpdatedEvent: id={}", example.id());
    }
}
```

### N+1 방지 패턴 Adapter

```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class StepCoreService implements GetStepsUseCase {

    private final StepDomainService stepDomainService;
    private final JobPostingMappingDomainService jobPostingMappingDomainService;
    private final ApplicantProgressDomainService applicantProgressDomainService;
    private final StepCoreMapper stepCoreMapper;

    @Override
    public List<StepResult> execute(GetStepsQuery query) {
        log.debug("Getting steps by jobPostingId: {}", query.jobPostingId());

        // 1. JobPostingMapping 조회
        var jobPostingMapping = jobPostingMappingDomainService
                .getByJobPostingId(query.jobPostingId());
        if (jobPostingMapping == null) {
            return List.of();
        }

        // 2. Step 목록 조회
        var steps = stepDomainService.getAllByJobPostingMappingId(jobPostingMapping.id());
        var stepResults = stepCoreMapper.toResults(steps);

        // 3. N+1 방지: 한 번에 ApplicantProgress 조회
        var stepIds = stepResults.stream().map(StepResult::id).toList();
        var stepIdsHaveApplicantProgress = applicantProgressDomainService
                .getStepIdsHaveApplicantProgress(stepIds);
        var stepIdsWithProgress = Set.copyOf(stepIdsHaveApplicantProgress);

        // 4. Result 보강
        return stepResults.stream()
                .map(stepResult -> stepCoreMapper.toEnrichedResult(
                        stepResult,
                        jobPostingMapping.jobPostingId(),
                        jobPostingMapping.workspaceId(),
                        stepIdsWithProgress.contains(stepResult.id())
                ))
                .toList();
    }
}
```

## Mapper

### 기본 Mapper

```java
package bizcenter.core.example.mapper;

import bizcenter.core.example.model.command.CreateExampleCommand;
import bizcenter.core.example.model.command.UpdateExampleCommand;
import bizcenter.core.example.model.result.ExampleResult;
import bizcenter.domain.example.model.Example;
import org.mapstruct.Mapper;
import org.mapstruct.Mapping;
import java.util.List;

/**
 * Example Core Mapper
 */
@Mapper
public interface ExampleCoreMapper {

    /**
     * CreateCommand -> Domain Model
     */
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "createdBy", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "updatedBy", ignore = true)
    @Mapping(target = "deletedAt", ignore = true)
    Example toDomainModel(CreateExampleCommand command);

    /**
     * Domain Model -> Result
     */
    ExampleResult toResult(Example model);

    /**
     * Domain Model List -> Result List
     */
    List<ExampleResult> toResults(List<Example> models);

    /**
     * 새 Example 생성 (builder 응집)
     */
    default Example toNewExample(String name, String description) {
        return Example.builder()
                .name(name.trim())
                .description(description)
                .build();
    }

    /**
     * 기존 Example 업데이트
     */
    default Example toUpdatedModel(Example existing, UpdateExampleCommand command) {
        return Example.builder()
                .id(existing.id())
                .name(command.name().trim())
                .description(command.description())
                .createdAt(existing.createdAt())
                .createdBy(existing.createdBy())
                .updatedAt(java.time.Instant.now())
                .updatedBy(getCurrentUser())
                .deletedAt(null)
                .build();
    }

    /**
     * 현재 사용자 조회
     */
    default String getCurrentUser() {
        // SecurityContext에서 사용자 정보 가져오기
        return "system"; // 예시
    }
}
```

### Result 보강 패턴

```java
@Mapper
public interface StepCoreMapper {

    StepResult toResult(Step model);
    List<StepResult> toResults(List<Step> models);

    /**
     * Result 보강 (추가 정보 포함)
     */
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

## Policy

### 단순 검증 Policy

```java
package bizcenter.core.example.policy;

import bizcenter.core.example.model.command.CreateExampleCommand;

/**
 * Example 생성 정책 인터페이스
 */
public interface ExampleCreatePolicy {
    void validate(CreateExampleCommand command);
}
```

```java
package bizcenter.core.example.policy;

import bizcenter.core.example.model.command.CreateExampleCommand;
import com.example.module.exception.BusinessException;
import com.example.module.exception.errorcode.CommonErrorCode;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

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

        // 비즈니스 규칙 검증
        if (command.name().length() > 126) {
            throw new BusinessException(
                    CommonErrorCode.BAD_REQUEST,
                    "Name cannot exceed 126 characters"
            );
        }

        if (command.description().length() > 500) {
            throw new BusinessException(
                    CommonErrorCode.BAD_REQUEST,
                    "Description cannot exceed 500 characters"
            );
        }

        log.debug("Validation passed for command: {}", command);
    }
}
```

### PolicyViolation 활용 Policy

```java
package bizcenter.core.step.policy;

import bizcenter.core.step.policy.input.StepUpsertValidationInput;
import bizcenter.core.step.policy.input.StepUpsertValidationInput.StepInput;
import com.example.module.exception.BusinessException;
import com.example.module.exception.errorcode.CommonErrorCode;
import com.example.module.policy.PolicyViolation;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

/**
 * Step 업서트 정책 구현체
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class StepUpsertPolicyImpl implements StepUpsertPolicy {

    private static final int MAX_STEPS_COUNT = 20;
    private static final int MAX_NAME_LENGTH = 126;

    @Override
    public void validate(StepUpsertValidationInput input) {
        log.debug("Validating step upsert input: jobPostingMappingId={}, steps.size={}",
                input.jobPostingMappingId(), input.steps().size());

        List<PolicyViolation> violations = new ArrayList<>();

        // 1단계: 전체 개수 검증 (빠른 실패)
        validateStepsCount(input.steps(), violations);
        if (!violations.isEmpty()) {
            throwBusinessException(violations);
        }

        // 2단계: 상세 검증
        validateRequiredFields(input.steps(), violations);
        validateNoDuplicates(input.steps(), violations);
        validateStepTypes(input.steps(), violations);
        validateOrderings(input.steps(), violations);

        if (!violations.isEmpty()) {
            throwBusinessException(violations);
        }

        log.debug("Validation passed for input");
    }

    private void validateStepsCount(List<StepInput> steps, List<PolicyViolation> violations) {
        if (steps.size() > MAX_STEPS_COUNT) {
            violations.add(PolicyViolation.of(
                    "Steps cannot exceed %d (current: %d)".formatted(MAX_STEPS_COUNT, steps.size())
            ));
        }
    }

    private void validateRequiredFields(List<StepInput> steps, List<PolicyViolation> violations) {
        for (int i = 0; i < steps.size(); i++) {
            StepInput step = steps.get(i);

            if (step.name() == null || step.name().isBlank()) {
                violations.add(PolicyViolation.of("Step[%d].name is required".formatted(i)));
            } else if (step.name().length() > MAX_NAME_LENGTH) {
                violations.add(PolicyViolation.of(
                        "Step[%d].name cannot exceed %d characters".formatted(i, MAX_NAME_LENGTH)
                ));
            }

            if (step.ordering() == null) {
                violations.add(PolicyViolation.of("Step[%d].ordering is required".formatted(i)));
            }

            if (step.stepType() == null || step.stepType().isBlank()) {
                violations.add(PolicyViolation.of("Step[%d].stepType is required".formatted(i)));
            }
        }
    }

    private void validateNoDuplicates(List<StepInput> steps, List<PolicyViolation> violations) {
        Set<String> names = new HashSet<>();
        Set<Integer> orderings = new HashSet<>();

        for (int i = 0; i < steps.size(); i++) {
            StepInput step = steps.get(i);

            if (step.name() != null && !names.add(step.name())) {
                violations.add(PolicyViolation.of(
                        "Duplicate step name found: '%s' at index %d".formatted(step.name(), i)
                ));
            }

            if (step.ordering() != null && !orderings.add(step.ordering())) {
                violations.add(PolicyViolation.of(
                        "Duplicate ordering found: %d at index %d".formatted(step.ordering(), i)
                ));
            }
        }
    }

    private void validateStepTypes(List<StepInput> steps, List<PolicyViolation> violations) {
        Set<String> validTypes = Set.of("DOCUMENT_SCREENING", "INTERVIEW", "PRACTICAL_TEST", "FINAL");

        for (int i = 0; i < steps.size(); i++) {
            StepInput step = steps.get(i);
            if (step.stepType() != null && !validTypes.contains(step.stepType())) {
                violations.add(PolicyViolation.of(
                        "Step[%d].stepType '%s' is invalid".formatted(i, step.stepType())
                ));
            }
        }
    }

    private void validateOrderings(List<StepInput> steps, List<PolicyViolation> violations) {
        for (int i = 0; i < steps.size(); i++) {
            StepInput step = steps.get(i);
            if (step.ordering() != null && step.ordering() < 1) {
                violations.add(PolicyViolation.of(
                        "Step[%d].ordering must be >= 1 (current: %d)".formatted(i, step.ordering())
                ));
            }
        }
    }

    private void throwBusinessException(List<PolicyViolation> violations) {
        String message = violations.stream()
                .map(PolicyViolation::getErrorMessage)
                .reduce((a, b) -> a + ", " + b)
                .orElse("Validation failed");

        log.warn("Policy validation failed: {}", message);
        throw new BusinessException(CommonErrorCode.BAD_REQUEST, message);
    }
}
```

## Event

### 도메인 이벤트

```java
package bizcenter.core.example.event;

import bizcenter.core.example.event.base.ExampleEventContext;
import bizcenter.core.example.event.base.ExampleDomainEvent;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Builder;

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
```

### 이벤트 Context

```java
package bizcenter.core.example.event.base;

import lombok.Builder;

/**
 * Example Event Context
 */
@Builder
public record DefaultExampleEventContext(
        String id,
        String name,
        String description
) implements ExampleEventContext {

    public static DefaultExampleEventContext of(String id, String name, String description) {
        return DefaultExampleEventContext.builder()
                .id(id)
                .name(name)
                .description(description)
                .build();
    }
}
```

## AutoConfiguration

```java
package bizcenter.core.example.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * Example Core AutoConfiguration
 */
@Configuration
@EnableTransactionManagement
@ComponentScan(basePackages = "bizcenter.core.example")
public class ExampleCoreAutoConfiguration {
    // 필요시 추가 Bean 설정
}
```

## 전체 흐름 예시

### 생성 UseCase 전체 흐름

```java
// 1. API Controller
@RestController
@RequestMapping("/api/v1/examples")
@RequiredArgsConstructor
public class ExampleController {

    private final CreateExampleUsecase createExampleUsecase;

    @PostMapping
    public ResponseEntity<ExampleResponse> create(@RequestBody ExampleRequest request) {
        // Request DTO → Command
        var command = new CreateExampleCommand(request.name(), request.description());

        // UseCase 실행
        ExampleResult result = createExampleUsecase.execute(command);

        // Result → Response DTO
        var response = ExampleResponse.from(result);
        return ResponseEntity.ok(response);
    }
}

// 2. UseCase 인터페이스
public interface CreateExampleUsecase {
    ExampleResult execute(CreateExampleCommand command);
}

// 3. Adapter (구현체)
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ExampleCoreService implements CreateExampleUsecase {

    private final ExampleDomainService exampleDomainService;
    private final ExampleCreatePolicy exampleCreatePolicy;
    private final ExampleCoreMapper coreMapper;
    private final DomainEventPublisher domainEventPublisher;

    @Override
    @Transactional
    public ExampleResult execute(CreateExampleCommand command) {
        // 1. 정책 검증
        exampleCreatePolicy.validate(command);

        // 2. Command → Domain Model
        Example model = coreMapper.toDomainModel(command);

        // 3. Domain Service 호출
        Example created = exampleDomainService.create(model);

        // 4. 이벤트 발행
        publishEvent(created);

        // 5. Domain Model → Result
        return coreMapper.toResult(created);
    }
}

// 4. Domain Service
@Service
@RequiredArgsConstructor
public class ExampleDomainService {

    private final ExampleRepository repository;

    public Example create(Example example) {
        ExampleEntity entity = toEntity(example);
        ExampleEntity saved = repository.save(entity);
        return toModel(saved);
    }
}
```
