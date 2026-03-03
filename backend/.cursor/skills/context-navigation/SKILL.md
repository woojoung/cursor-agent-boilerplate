---
name: context-navigation
description: AGENTS.md 기반 코드 맥락 네비게이션. 폴더별 문서를 활용하여 코드 구조를 빠르게 파악합니다.
---

# Context Navigation Skill

AGENTS.md 문서를 활용하여 코드베이스의 구조와 맥락을 빠르게 파악하는 스킬입니다.

## 목적

- 새로운 코드베이스 빠른 이해
- 관련 코드 위치 신속 파악
- 의존성 관계 시각화
- 작업 영향 범위 분석

## AGENTS.md 구조

### 폴더별 AGENTS.md 배치

```
src/main/java/com/example/
├── AGENTS.md                    ← 프로젝트 전체 개요
├── auth/
│   ├── AGENTS.md                ← 인증 모듈 개요
│   ├── domain/
│   │   └── AGENTS.md            ← 도메인 레이어
│   ├── application/
│   │   └── AGENTS.md            ← 애플리케이션 레이어
│   └── adapter/
│       └── AGENTS.md            ← 어댑터 레이어
├── order/
│   └── AGENTS.md                ← 주문 모듈 개요
└── common/
    └── AGENTS.md                ← 공통 모듈 개요
```

### AGENTS.md 표준 구조

```markdown
# {폴더명}

> 최종 업데이트: {date}

## 개요

{이 폴더/모듈의 목적과 역할}

## 구조

```
{폴더명}/
├── {파일1} - {설명}
├── {파일2} - {설명}
└── {하위폴더}/
```

## 주요 클래스

### {ClassName}
- **역할**: {역할}
- **의존성**: {의존 클래스}
- **사용처**: {사용하는 곳}

## 의존성 관계

```
{diagram}
```

## 관련 문서

- [{관련 모듈}](path/to/AGENTS.md)
```

## 네비게이션 전략

### 1. Top-Down 탐색

프로젝트 전체에서 시작하여 세부 모듈로 이동

```
1. src/main/java/com/example/AGENTS.md    ← 전체 구조 파악
   │
   ▼
2. src/main/java/com/example/auth/AGENTS.md    ← 관련 모듈 이동
   │
   ▼
3. src/main/java/com/example/auth/domain/AGENTS.md    ← 레이어별 상세
```

### 2. Bottom-Up 탐색

특정 파일에서 시작하여 상위 문맥 파악

```
1. AuthService.java 수정 필요
   │
   ▼
2. auth/application/AGENTS.md    ← 현재 레이어 이해
   │
   ▼
3. auth/AGENTS.md    ← 모듈 전체 이해
   │
   ▼
4. AGENTS.md    ← 프로젝트 구조 이해
```

### 3. 관련 모듈 탐색

의존성 관계를 따라 이동

```
AuthService에서 UserRepository 사용
   │
   ▼
auth/AGENTS.md의 "의존성 관계" 섹션 확인
   │
   ▼
user/AGENTS.md로 이동
```

## 활용 시나리오

### 시나리오 1: 새 기능 추가 위치 파악

```
질문: "결제 기능을 어디에 추가해야 하나요?"

탐색 순서:
1. AGENTS.md → 모듈 목록 확인
2. payment/ 모듈 존재 확인 (있으면 이동)
3. 없으면 유사 모듈(order/) 구조 참고
4. 새 모듈 위치 결정
```

### 시나리오 2: 버그 위치 추적

```
문제: "로그인 시 토큰이 발급되지 않음"

탐색 순서:
1. auth/AGENTS.md → 인증 흐름 파악
2. auth/application/AGENTS.md → UseCase 확인
3. AuthUseCase 클래스 → login 메서드 찾기
4. 관련 테스트 확인
```

### 시나리오 3: 리팩토링 영향 범위 분석

```
작업: "UserRepository 인터페이스 변경"

분석 순서:
1. user/AGENTS.md → "사용처" 확인
2. 의존하는 모듈 목록 수집
   - auth/ (AuthUseCase)
   - order/ (OrderUseCase)
3. 각 모듈의 AGENTS.md에서 상세 사용처 확인
4. 영향받는 클래스 목록 작성
```

## 빠른 참조 패턴

### 클래스 찾기

```
"{클래스명}이 어디 있나요?"

1. 프로젝트 AGENTS.md에서 모듈 확인
2. 관련 모듈의 AGENTS.md에서 "주요 클래스" 섹션 검색
3. 파일 경로 확인
```

### 의존성 확인

```
"{클래스}가 무엇에 의존하나요?"

1. 해당 클래스가 속한 폴더의 AGENTS.md 열기
2. "주요 클래스" 섹션에서 해당 클래스 찾기
3. "의존성" 항목 확인
```

### 사용처 확인

```
"{클래스}를 사용하는 곳은?"

1. 해당 클래스의 AGENTS.md에서 "사용처" 확인
2. 또는 상위 모듈 AGENTS.md의 "의존성 관계" 확인
```

## 의존성 다이어그램

### 모듈 간 의존성

```
┌─────────────────────────────────────────────────────┐
│                     adapter/in/web                  │
│                    (Controllers)                    │
└───────────────────────┬─────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│                    application                      │
│                    (UseCases)                       │
└───────────────────────┬─────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌───────────────┐ ┌───────────┐ ┌───────────────────┐
│    domain     │ │   port    │ │   adapter/out     │
│  (Entities)   │ │   (I/F)   │ │  (Repositories)   │
└───────────────┘ └───────────┘ └───────────────────┘
```

### 모듈 의존성

```
        ┌──────────┐
        │  common  │
        └──────────┘
             ▲
    ┌────────┼────────┐
    │        │        │
┌───────┐ ┌───────┐ ┌───────┐
│ auth  │ │ user  │ │ order │
└───────┘ └───────┘ └───────┘
    │        ▲        │
    └────────┘        │
    (UserRepository)  │
             ┌────────┘
             ▼
        ┌──────────┐
        │ payment  │
        └──────────┘
```

## 연관 에이전트

- **context-updater**: AGENTS.md 자동 업데이트
- **planner**: 네비게이션 결과로 plan 수립

## 팁

1. 새 프로젝트 시작 시 최상위 AGENTS.md부터 읽기
2. 코드 변경 시 관련 AGENTS.md 확인 습관화
3. AGENTS.md가 없는 폴더 발견 시 context-updater로 생성
4. 의존성 다이어그램을 시각적으로 그려보기
