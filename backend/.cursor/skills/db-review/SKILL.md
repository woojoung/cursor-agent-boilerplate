---
name: db-review
description:
  Postgres Best Practices 기반 도메인 모듈 DB 리뷰.
---

# DB Review

Postgres Best Practices 기반의 도메인 모듈 DB 리뷰 스킬입니다.
**(중요) 개선이 필요한 부분들만 상세히 리뷰결과를 냅니다. 양호한 부분(잘된점)들은 요약만 작성합니다**
리뷰를 마친후 md 파일로 결과를 저장합니다.

# 사용 시점

(1) *-domain 모듈의 특정 도메인 DB 리뷰 요청 시
(2) Entity/Repository 코드 리뷰 시
(3) DB 성능 최적화 검토 시
(4) /db-review 명령어 실행 시.

## Workflow

```
1. 도메인 경로 확인
   └─ 경로 미제공 시 → AskUserQuestion으로 확인

2. 파일 수집 및 분석
   ├─ Entity 파일 분석
   ├─ Repository 파일 분석
   └─ Flyway SQL 마이그레이션 분석

3. postgres-best-practices 스킬 활용 리뷰
    └─ `postgres-best-practices:supabase-postgres-best-practices` 스킬 호출

4. 리뷰 결과 저장
   └─ {도메인경로}/REVIEW_DB.md 파일 생성
```

## Step 1: 도메인 경로 확인

도메인 경로가 제공되지 않은 경우 AskUserQuestion으로 확인.

**보일러플레이트 예시** (실제 구조에 맞게 프로젝트에서 조정):

```
예시 경로 (일반적인 Java/Spring 레이아웃):
- backend/src/main/java/com/example/domain/{도메인명}
- backend/modules/{모듈명}/src/main/java/.../domain/{도메인명}

패턴 (참고용):
- {프로젝트루트}/src/main/java/{패키지}/domain/{도메인명}
- 또는 팀에서 사용하는 도메인 모듈 기준 경로

※ 프로젝트의 AGENTS.md, 모듈 구조(spec/plan)를 참고해 실제 도메인 루트를 확인한다.
```

## Step 2: 파일 수집

### 2-1. 도메인 파일 수집

```bash
# Entity 파일
{도메인경로}/entity/*.java

# Repository 파일
{도메인경로}/repository/*.java

# Service 파일 (트랜잭션 분석용)
{도메인경로}/service/*.java
```

### 2-2. Flyway 마이그레이션 수집

flyway/migration/postgresql/{도메인} 경로내 SQL 수집후 시간순으로 분석하여, 최종 스키마 도출해야함.

## Step 3: postgres-best-practices 스킬 활용

`postgres-best-practices:supabase-postgres-best-practices` 스킬을 호출하여 리뷰 진행.

### 최소 체크 항목

**(중요)아래 체크리스트는 최소 점검 항목이며, 실제 리뷰 시 더 다양한 관점에서 리뷰할것**

**Query Performance:**

- WHERE/JOIN 컬럼 인덱스 존재 여부
- Partial Index 적용 가능성 (Soft Delete 패턴)
- 복합 인덱스 최적화

**Data Access Patterns:**

- Batch Insert/Update 사용 여부
- N+1 쿼리 방지 (IN 절 사용)
- Bulk 연산 활용

**Concurrency & Locking:**

- `@Transactional(readOnly = true)` 기본 적용
- 쓰기 트랜잭션 범위 최소화
- 긴 트랜잭션 방지

**Schema Design:**

- FK 인덱스 존재 여부
- UNIQUE 제약 조건 (NULL 처리)
- 적절한 데이터 타입

## Step 4: 리뷰 결과 저장

리뷰 완료 후 `{도메인경로}/REVIEW_DB.md` 파일 생성.

### 결과 파일 구조

```markdown
# Postgres Best Practices 리뷰 결과

> 리뷰 일시: YYYY-MM-DD
> 대상: {도메인경로}
> 기준: Supabase Postgres Best Practices

## Query Performance

## Data Access Patterns

## Concurrency & Locking

## Schema Design

## ...추가 리뷰 카테고리...

## 5. 종합 평가

| 카테고리 | 평가 | 점수 |
|----------|------|------|

## 권장 개선 마이그레이션 스크립트

```sql
-- 개선 SQL
```

## 7. 요약

### 잘 된 점

### 개선 필요

```

## 의존성

이 스킬은 다음 스킬을 활용합니다:
- `postgres-best-practices:supabase-postgres-best-practices`

## 연관 에이전트

- **code-reviewer**: 코드 리뷰 시 DB 관련 이슈 탐지
- **planner**: DB 스키마 설계 계획 수립
