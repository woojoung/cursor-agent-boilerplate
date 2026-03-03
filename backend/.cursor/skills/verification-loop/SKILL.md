---
name: verification-loop
description: 빌드/테스트 검증 루프. Gradle 빌드, 테스트 실행, JaCoCo 커버리지 검증 단계를 제공합니다.
---

# Verification Loop Skill

빌드, 테스트, 커버리지 검증을 위한 루프 프로세스입니다.

## 검증 루프 개요

```
┌─────────────────────────────────────────────────────────────┐
│                    Verification Loop                        │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  1. Compile   │
                    └───────────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
               ✅ Pass         ❌ Fail
                    │               │
                    │               └── build-fixer 호출
                    ▼
                    ┌───────────────┐
                    │   2. Test     │
                    └───────────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
               ✅ Pass         ❌ Fail
                    │               │
                    │               └── build-fixer 호출
                    ▼
                    ┌───────────────┐
                    │ 3. Coverage   │
                    └───────────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
               ✅ Pass         ❌ Fail
                    │               │
                    │               └── 테스트 추가 필요
                    ▼
                    ┌───────────────┐
                    │ 4. Lint/Check │
                    └───────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   ✅ Done     │
                    └───────────────┘
```

## Step 1: 컴파일 검증

### Gradle 빌드

```bash
# 기본 빌드 (테스트 제외)
./gradlew compileJava compileTestJava

# 전체 빌드
./gradlew build -x test

# 클린 빌드
./gradlew clean build -x test
```

### 성공 기준

```
✅ 컴파일 성공
- BUILD SUCCESSFUL
- 컴파일 에러 0개
- 경고는 허용 (단, deprecated 경고 확인)
```

### 실패 시 대응

```
❌ 컴파일 실패

일반적인 원인:
1. 문법 오류 → 코드 수정
2. 누락된 import → import 추가
3. 타입 불일치 → 타입 수정
4. 의존성 문제 → build.gradle 확인

자동 대응:
→ build-fixer 에이전트 호출
```

## Step 2: 테스트 실행

### 테스트 명령어

```bash
# 전체 테스트 실행
./gradlew test

# 특정 테스트만 실행
./gradlew test --tests "*.AuthServiceTest"

# 실패한 테스트만 재실행
./gradlew test --rerun-tasks

# 테스트 리포트 생성
./gradlew test jacocoTestReport
```

### 성공 기준

```
✅ 테스트 성공
- 모든 테스트 통과
- 실패: 0
- 스킵: 0 (또는 의도적 @Disabled만)
```

### 실패 시 대응

```
❌ 테스트 실패

분석:
1. 테스트 리포트 확인: build/reports/tests/test/index.html
2. 실패 원인 파악:
   - Assertion 실패 → 로직 버그
   - NullPointerException → Mock 설정 문제
   - 타임아웃 → 비동기 처리 문제

자동 대응:
→ build-fixer 에이전트 호출
```

### 테스트 리포트

```
build/reports/tests/test/
├── index.html          ← 메인 리포트
├── classes/
│   └── {TestClass}.html
└── packages/
    └── {package}.html
```

## Step 3: 커버리지 검증

### JaCoCo 설정

```groovy
// build.gradle
plugins {
    id 'jacoco'
}

jacoco {
    toolVersion = "0.8.11"
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
        html.required = true
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.80  // 80%
            }
        }
        rule {
            limit {
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 0.70  // 70%
            }
        }
    }
}

check.dependsOn jacocoTestCoverageVerification
```

### 커버리지 명령어

```bash
# 커버리지 리포트 생성
./gradlew jacocoTestReport

# 커버리지 검증 (임계값 체크)
./gradlew jacocoTestCoverageVerification

# 테스트 + 리포트 + 검증
./gradlew test jacocoTestReport jacocoTestCoverageVerification
```

### 성공 기준

```
✅ 커버리지 통과
- 라인 커버리지: 80% 이상
- 브랜치 커버리지: 70% 이상
- 신규 코드: 100% 권장
```

### 커버리지 리포트

```
build/reports/jacoco/test/html/
├── index.html          ← 전체 요약
├── {package}/
│   ├── index.html      ← 패키지 요약
│   └── {Class}.html    ← 클래스별 상세
```

### 실패 시 대응

```
❌ 커버리지 미달

현재: 72%
목표: 80%

미커버 영역:
- AuthService.validateToken(): 0%
- UserAdapter.update(): 45%

필요 조치:
1. 미커버 메서드 확인
2. 테스트 케이스 추가
3. 재검증
```

## Step 4: 정적 분석 (선택)

### Checkstyle

```groovy
// build.gradle
plugins {
    id 'checkstyle'
}

checkstyle {
    toolVersion = '10.12.4'
    configFile = file("config/checkstyle/checkstyle.xml")
}
```

### SpotBugs

```groovy
plugins {
    id 'com.github.spotbugs' version '5.2.1'
}

spotbugs {
    toolVersion = '4.8.0'
    excludeFilter = file("config/spotbugs/exclude.xml")
}
```

### 실행

```bash
# Checkstyle
./gradlew checkstyleMain checkstyleTest

# SpotBugs
./gradlew spotbugsMain spotbugsTest

# 전체 검사
./gradlew check
```

## 전체 검증 명령어

### 빠른 검증

```bash
# 컴파일 + 테스트만
./gradlew test
```

### 전체 검증

```bash
# 컴파일 + 테스트 + 커버리지 + 정적분석
./gradlew clean check jacocoTestReport
```

### CI/CD용

```bash
# 상세 로그 + 리포트
./gradlew clean check jacocoTestReport --info --stacktrace
```

## 검증 결과 출력

### 성공 시

```
✅ Verification Loop 완료

결과:
├── 컴파일: ✅ Pass
├── 테스트: ✅ 45개 통과 (0 실패)
├── 커버리지: ✅ 87% (목표 80%)
└── 정적분석: ✅ Pass

다음 단계:
→ /review-code 또는 /mr-gitlab 진행 가능
```

### 실패 시

```
❌ Verification Loop 실패

결과:
├── 컴파일: ✅ Pass
├── 테스트: ❌ 2개 실패
├── 커버리지: - (테스트 실패로 스킵)
└── 정적분석: - (스킵)

실패 테스트:
1. AuthServiceTest.loginWithInvalidPassword_ThrowsException
   - Expected: InvalidPasswordException
   - Actual: NullPointerException at AuthService.java:45

2. UserControllerTest.getUser_NotFound_Returns404
   - Expected: 404
   - Actual: 500

수정이 필요합니다. 수정하시겠습니까?
```

## 자동화 통합

### plan-from-spec 파이프라인 내 실행

```
plan.md 승인
    │
    ▼
TDD 사이클 (RED → GREEN → REFACTOR)
    │
    ▼
Verification Loop ← 여기서 자동 실행
    │
    ├── 실패 → 수정 → 재검증
    │
    └── 성공 → Code Review → /mr-gitlab
```

### CI/CD 파이프라인

```yaml
# .gitlab-ci.yml
test:
  stage: test
  script:
    - ./gradlew test jacocoTestReport
  artifacts:
    reports:
      junit: build/test-results/test/*.xml
    paths:
      - build/reports/

coverage:
  stage: test
  script:
    - ./gradlew jacocoTestCoverageVerification
  coverage: '/Total.*?([0-9]{1,3})%/'
```

## 연관 에이전트

- **build-fixer**: 빌드/테스트 실패 시 자동 호출
- **tdd-guide**: TDD 사이클 가이드
- **planner**: 검증 단계 plan에 포함
