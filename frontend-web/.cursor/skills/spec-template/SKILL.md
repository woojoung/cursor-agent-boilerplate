---
name: spec-template
description: spec.md 템플릿 및 작성 가이드. Spec-Driven Development 시작점.
---

# Spec Template Skill (frontend-web)

프론트엔드(웹) 기능 spec 작성 시 목표·요구사항·제약을 구조화합니다.

## spec.md 템플릿 요약

```markdown
# Feature: {기능 이름}
## 1. 목표
{해결할 문제·달성 목표 1–3문장}
## 2. 요구사항
### 기능 요구사항
- [ ] 페이지/라우트, 컴포넌트, 상호작용
- [ ] API 연동, 상태, 에러 처리
### 비기능 (성능·접근성·SEO)
- [ ] LCP/CLS 목표, a11y 기준, 메타/OG
## 3. 제약사항
{Next 버전, 디자인 시스템, 브라우저 지원 등}
## 4. 참고
{디자인 시안, API 문서, 관련 이슈}
```

## 작성 원칙

- 사용자 시나리오·화면 단위로 요구사항 나열. 한 항목당 검증 가능하게.
- 접근성·반응형·에러/로딩 상태를 비기능 요구에 포함.
