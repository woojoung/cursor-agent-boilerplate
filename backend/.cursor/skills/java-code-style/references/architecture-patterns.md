# 아키텍처 패턴

## Clean Architecture 개요

BizCenter 프로젝트는 Clean Architecture를 기반으로 레이어를 분리합니다.

### 레이어 구조

```
API Layer → Core Layer → Domain Layer
    ↓           ↓            ↓
Controller  UseCase      Domain Service
            Adapter      Repository
```

### 의존성 규칙

- **단방향 의존**: API → Core → Domain (역방향 의존 금지)
- **순환 의존 금지**: 모듈 간 순환 참조 절대 불가
- **인터페이스 분리**: 각 레이어는 인터페이스로 통신

## CQRS 패턴

Command와 Query를 명확히 분리하여 책임을 구분합니다.

### Command (CUD - Create, Update, Delete)

**특징**:
- 상태를 변경하는 작업
- 트랜잭션 필수 (`@Transactional`)
- 정책 검증 수행 (Policy)
- 도메인 이벤트 발행 (필요시)
- Result 객체 반환

**예시**:
```java
@Transactional
public ExampleResult execute(CreateExampleCommand command) {
    // 1. 정책 검증
    exampleCreatePolicy.validate(command);

    // 2. Command → Domain Model 변환
    Example domainModel = mapper.toDomainModel(command);

    // 3. Domain Service 호출
    Example created = domainService.create(domainModel);

    // 4. 도메인 이벤트 발행
    publishExampleCreatedEvent(created);

    // 5. Domain Model → Result 변환
    return mapper.toResult(created);
}
```

### Query (Read)

**특징**:
- 상태를 조회만 하는 작업
- 읽기 전용 트랜잭션 (`@Transactional(readOnly = true)`)
- 정책 검증 불필요 (대부분)
- 이벤트 발행 불필요
- Result 또는 List<Result> 반환

**예시**:
```java
@Override
public ExampleResult execute(GetExampleQuery query) {
    Example example = domainService.findById(query.id());
    return mapper.toResult(example);
}

@Override
public List<ExampleResult> execute(GetExamplesQuery query) {
    List<Example> examples = domainService.findAll();
    return mapper.toResults(examples);
}
```

**Query 중 부수 효과가 있는 경우**:
- 읽음 처리, 통계 업데이트 등은 `@Transactional` 명시
- 기본 조회는 `@Transactional(readOnly = true)` 유지

```java
@Override
@Transactional // readOnly = false (쓰기 작업 포함)
public MessageResult execute(GetMessageQuery query) {
    Message message = messageService.findById(query.id());
    messageService.markAsRead(query.id()); // 읽음 처리
    return mapper.toResult(message);
}
```

## 레이어별 책임

### API Layer

**역할**:
- HTTP 요청/응답 처리
- 인증/인가
- Request DTO → Command/Query 변환
- Result → Response DTO 변환

**금지사항**:
- ❌ Entity 직접 노출 (DTO만 사용)
- ❌ 비즈니스 로직 포함
- ❌ 트랜잭션 관리 (Core에 위임)
- ❌ Domain Service 직접 호출 (UseCase를 통해서만)

**예시**:
```java
@RestController
@RequestMapping("/api/v1/examples")
@RequiredArgsConstructor
public class ExampleController {

    private final CreateExampleUsecase createExampleUsecase;
    private final GetExampleUsecase getExampleUsecase;

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
```

### Core Layer (UseCase Layer)

**역할**:
- 비즈니스 유스케이스 진입점
- 트랜잭션 경계 관리
- Command/Query ↔ Domain Model 변환
- 정책 검증 조율
- 도메인 이벤트 발행 조율

**구성 요소**:
- **UseCase**: 인터페이스, public
- **Adapter**: UseCase 구현체, package-private
- **Command/Query**: 입력 파라미터, public
- **Result**: 출력 결과, public
- **Mapper**: 변환 계층
- **Policy**: 비즈니스 정책 검증

**처리 흐름** (표준 5단계):

1. **정책 검증**: Policy.validate()
2. **변환**: Command → Domain Model (Mapper)
3. **도메인 로직**: Domain Service 호출
4. **이벤트 발행**: DomainEventPublisher.publish() (필요시)
5. **결과 변환**: Domain Model → Result (Mapper)

### Domain Layer

**역할**:
- 순수 비즈니스 로직
- CRUD 작업
- 도메인 규칙 적용

**구성 요소**:
- **Domain Service**: 비즈니스 로직
- **Domain Model**: 불변 객체 (record 또는 class)
- **Entity**: JPA 엔티티
- **Repository**: 데이터 접근

**금지사항**:
- ❌ 외부 의존성 (순수 비즈니스 로직만)
- ❌ 트랜잭션 생성 (참여만 가능)
- ❌ 도메인 이벤트 발행 (Core Layer에서 수행)

## 트랜잭션 경계

### 트랜잭션 관리 원칙

- **Core Layer (Adapter)가 트랜잭션 시작점**
- Domain Layer는 트랜잭션에 참여만
- API Layer는 트랜잭션 관리 안 함

### Adapter 트랜잭션 패턴

**클래스 레벨 readOnly**:
```java
@Service
@Transactional(readOnly = true) // 기본값
class ExampleCoreService implements
        CreateExampleUsecase,
        GetExampleUsecase {

    // 조회는 기본 readOnly 사용
    @Override
    public ExampleResult execute(GetExampleQuery query) {
        // readOnly = true
    }

    // 쓰기는 명시적으로 @Transactional
    @Override
    @Transactional // readOnly = false
    public ExampleResult execute(CreateExampleCommand command) {
        // 쓰기 작업
    }
}
```

### 트랜잭션 전파

Domain Service는 `@Transactional` 없이 구현:

```java
@Service
@RequiredArgsConstructor
public class ExampleDomainService {

    private final ExampleRepository repository;

    // @Transactional 없음 (Core에서 시작한 트랜잭션에 참여)
    public Example create(Example example) {
        Example entity = toEntity(example);
        Example saved = repository.save(entity);
        return toModel(saved);
    }
}
```

## 데이터 변환 계층

### 변환 규칙

```
API Layer:  Request DTO → Command/Query
Core Layer: Command → Domain Model → Result
Domain Layer: Domain Model → Entity
```

### 각 레이어의 데이터 타입

| 레이어 | 입력 | 처리 | 출력 |
|--------|------|------|------|
| API | Request DTO | - | Response DTO |
| Core (Adapter) | Command/Query | Domain Model | Result |
| Domain Service | Domain Model | Entity | Domain Model |
| Repository | Entity | - | Entity |

### 변환 예시

```java
// API Layer
@PostMapping
public ResponseEntity<ExampleResponse> create(@RequestBody ExampleRequest request) {
    var command = new CreateExampleCommand(request.name(), request.description());
    ExampleResult result = usecase.execute(command);
    return ResponseEntity.ok(ExampleResponse.from(result));
}

// Core Layer (Adapter)
@Transactional
public ExampleResult execute(CreateExampleCommand command) {
    Example model = mapper.toDomainModel(command); // Command → Model
    Example created = service.create(model);        // Model → Entity → Model
    return mapper.toResult(created);                // Model → Result
}
```

## 성능 최적화 패턴

### N+1 쿼리 방지

**문제가 되는 코드**:
```java
List<ApplicantProgress> progresses = service.getAllProgresses();
for (ApplicantProgress progress : progresses) {
    Step step = stepService.get(progress.stepId()); // N+1 발생!
}
```

**해결 방법**:
```java
List<ApplicantProgress> progresses = service.getAllProgresses();

// 1. ID 목록 추출
List<String> stepIds = progresses.stream()
    .map(ApplicantProgress::stepId)
    .distinct()
    .toList();

// 2. 한 번에 조회하여 Map으로 변환
Map<String, Step> stepMap = stepService.getAllByIds(stepIds).stream()
    .collect(Collectors.toMap(Step::id, Function.identity()));

// 3. Map에서 조회
for (ApplicantProgress progress : progresses) {
    Step step = stepMap.get(progress.stepId());
    // 처리
}
```

### 배치 처리

```java
// 개별 저장 (X)
for (Example example : examples) {
    repository.save(example);
}

// 배치 저장 (O)
repository.saveAll(examples);
```

### Join 조회와 Result 보강

**기본 Result 생성 후 보강**:
```java
// 1. 기본 Result 생성
List<StepResult> stepResults = mapper.toResults(steps);

// 2. 추가 정보 조회
JobPostingMapping mapping = mappingService.getByJobPostingId(jobPostingId);
Set<String> stepIdsWithProgress = getStepIdsHaveProgress(stepResults);

// 3. Result 보강
return stepResults.stream()
    .map(result -> mapper.toEnrichedResult(
        result,
        mapping.jobPostingId(),
        mapping.workspaceId(),
        stepIdsWithProgress.contains(result.id())
    ))
    .toList();
```

## Adapter 분리 패턴

복잡한 feature는 역할별로 여러 Adapter로 분리 가능:

### 예시: ApplicantProgress

```
applicantprogress/
├── adapter/
│   ├── ApplicantProgressCoreService.java         # 기본 CRUD
│   ├── ApplicantProgressWorkerCoreService.java   # Worker 전용 (외부 API 호출)
│   └── ApplicantProgressDownloadCoreService.java # 다운로드/내보내기
```

**분리 기준**:
- 기본 CRUD: 일반 Adapter
- 외부 API 호출: Worker 전용 Adapter
- 대용량 처리: Download/Export 전용 Adapter
- 복잡한 비즈니스 로직: Business 전용 Adapter

### Worker 전용 Adapter 패턴

외부 API 클라이언트 의존성 포함 가능:

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class ApplicantProgressWorkerCoreService {

    private final AtsClient atsClient;
    private final JobHubClient jobHubClient;
    private final DomainService domainService;

    @Transactional
    public void syncFromExternalPlatform(SyncCommand command) {
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

## 아키텍처 선택 기준

| 패턴 | 사용 케이스 | 예시 |
|------|------------|------|
| **기본 아키텍처** | Legacy 도메인, 간단한 CRUD | bizcenter-api, bizcenter-domain-legacy |
| **Clean Architecture** | 복잡한 비즈니스 로직, 신규 도메인 | bizcenter-ats-*, 신규 도메인 |

### 기본 아키텍처 흐름

```
API Controller → AppService → Domain Service → Repository
```

### Clean Architecture 흐름

```
API Controller → UseCase → Adapter → Domain Service → Repository
```
