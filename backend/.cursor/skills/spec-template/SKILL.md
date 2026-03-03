---
name: spec-template
description: spec.md 템플릿 및 작성 가이드. Spec-Driven Development의 시작점입니다.
---

# Spec Template Skill

Spec-Driven Development를 위한 spec.md 템플릿과 작성 가이드를 제공합니다.

## 목적

- 요구사항을 명확하게 문서화
- 개발 범위 정의
- plan 생성의 기반 제공

## spec.md 템플릿

```markdown
# Feature: {기능 이름}

> 작성일: {YYYY-MM-DD}
> 작성자: {이름}
> 상태: 초안 | 검토중 | 승인됨

---

## 1. 목표

{이 기능이 해결하려는 문제나 달성하려는 목표를 1-3문장으로 작성}

예시:
- 사용자가 JWT 토큰 기반으로 안전하게 로그인할 수 있도록 한다.
- 기존 세션 기반 인증을 토큰 기반으로 전환하여 확장성을 확보한다.

---

## 2. 요구사항

### 2.1 기능 요구사항

- [ ] {요구사항 1}
- [ ] {요구사항 2}
- [ ] {요구사항 3}

예시:
- [ ] POST /api/auth/login - 이메일/비밀번호로 로그인
- [ ] POST /api/auth/logout - 로그아웃 (토큰 무효화)
- [ ] POST /api/auth/refresh - 토큰 갱신
- [ ] GET /api/auth/me - 현재 사용자 정보 조회

### 2.2 비기능 요구사항

- [ ] 테스트 커버리지 80% 이상
- [ ] 응답 시간 200ms 이내
- [ ] 기존 기능 regression 없음

---

## 3. 제약사항

{기술적 제약이나 비즈니스 제약을 명시}

예시:
- 기존 User 테이블 스키마 변경 불가
- Redis 사용 불가 (토큰 저장소로 DB 사용)
- Spring Security 6.x 버전 사용

---

## 4. 참고사항

{관련 문서, 링크, 기존 코드 참조}

예시:
- 기존 인증 코드: src/main/java/com/example/auth/
- API 문서: https://wiki.company.com/auth-api
- 관련 이슈: JIRA-1234

---

## 5. 용어 정의 (선택)

| 용어 | 정의 |
|------|------|
| Access Token | 인증에 사용되는 단기 토큰 (1시간) |
| Refresh Token | 토큰 갱신에 사용되는 장기 토큰 (7일) |

---

> 작성 완료 후 `/plan-from-spec` 명령어로 구현 계획을 생성하세요.
```

## 작성 가이드라인

### 1. 간결하게 작성 (30줄 이내 권장)

```
❌ 너무 상세한 spec:
- 모든 API 필드 정의
- 데이터베이스 스키마
- 상세 시퀀스 다이어그램

✅ 적절한 spec:
- 핵심 요구사항 목록
- 주요 제약사항
- 참고 링크
```

### 2. "무엇을" 에 집중

```
❌ 구현 방법 명시:
"UserService 클래스에 login() 메서드를 추가하고
BCryptPasswordEncoder를 사용하여 비밀번호를 검증한다"

✅ 요구사항 명시:
"사용자가 이메일과 비밀번호로 로그인할 수 있어야 한다"
```

### 3. 측정 가능한 요구사항

```
❌ 모호한 요구사항:
"빠른 응답 속도"
"안전한 인증"

✅ 명확한 요구사항:
"응답 시간 200ms 이내"
"비밀번호는 BCrypt로 해시화"
```

### 4. 제약사항 명시

```
반드시 포함:
- 사용 불가한 기술/라이브러리
- 변경 불가한 기존 코드
- 호환성 요구사항
- 성능 요구사항
```

## 예시 Spec 문서들

### 예시 1: 사용자 인증

```markdown
# Feature: JWT 기반 사용자 인증

> 작성일: 2026-01-25
> 상태: 초안

## 목표
세션 기반 인증을 JWT 토큰 기반으로 전환하여 수평 확장이 가능한 구조로 개선한다.

## 요구사항

### 기능 요구사항
- [ ] POST /api/auth/login - 로그인 (JWT 발급)
- [ ] POST /api/auth/refresh - 토큰 갱신
- [ ] 토큰 만료 시간: Access 1시간, Refresh 7일

### 비기능 요구사항
- [ ] 커버리지 80% 이상
- [ ] 기존 API 하위 호환성 유지

## 제약사항
- Redis 사용 불가 (Refresh Token은 DB 저장)
- 기존 User 엔티티 수정 최소화

## 참고사항
- 기존 코드: src/main/java/com/example/security/
```

### 예시 2: 테스트 커버리지 향상

```markdown
# Feature: Auth 모듈 테스트 커버리지 향상

> 작성일: 2026-01-25
> 상태: 초안

## 목표
src/main/java/com/example/auth 폴더의 테스트 커버리지를 현재 45%에서 80% 이상으로 향상시킨다.

## 요구사항

### 기능 요구사항
- [ ] AuthService 테스트 추가 (현재 30% → 80%)
- [ ] AuthController 테스트 추가 (현재 50% → 80%)
- [ ] TokenProvider 테스트 추가 (현재 60% → 90%)

### 비기능 요구사항
- [ ] 기존 테스트 깨지지 않음
- [ ] 테스트 실행 시간 30초 이내

## 제약사항
- 프로덕션 코드 변경 최소화
- Mock 기반 단위 테스트 우선

## 참고사항
- 현재 커버리지 리포트: build/reports/jacoco/
```

### 예시 3: API 엔드포인트 추가

```markdown
# Feature: 주문 취소 API

> 작성일: 2026-01-25
> 상태: 초안

## 목표
사용자가 주문을 취소할 수 있는 API를 제공한다.

## 요구사항

### 기능 요구사항
- [ ] POST /api/orders/{id}/cancel - 주문 취소
- [ ] 취소 가능 조건: 배송 시작 전
- [ ] 취소 시 재고 복구
- [ ] 취소 시 결제 환불 요청

### 비기능 요구사항
- [ ] 트랜잭션 일관성 보장
- [ ] 취소 이력 저장

## 제약사항
- 결제 환불은 외부 API 호출 (PG사)
- 환불 실패 시 수동 처리 필요

## 참고사항
- 주문 도메인: src/main/java/com/example/order/
- PG 연동: src/main/java/com/example/payment/
```

## 상태 관리

```
초안 (Draft)
    ↓ 검토 요청
검토중 (In Review)
    ↓ 승인
승인됨 (Approved)
    ↓ /plan-from-spec
구현중 (In Progress)
    ↓ 완료
완료됨 (Done)
```

## 연관 명령어

- `/spec-template`: 이 스킬을 사용하여 spec.md 생성
- `/plan-from-spec`: spec.md를 기반으로 plan 생성
