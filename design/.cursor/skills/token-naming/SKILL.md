---
name: token-naming
description: 디자인 토큰 계층·시맨틱 명명·카테고리 규칙.
---

# Token Naming Skill (design)

## 계층

- **글로벌**: color.*, space.*, font.*, radius.*, shadow.*. 원시 값만.
- **시맨틱**: color.primary, color.text.default, surface.card, border.focus. 역할 기반. 테마 교체 시 이 레이어만 변경.
- **컴포넌트**: button.primary.bg, input.border.error. 컴포넌트 스코프.

## 명명 규칙

- 카테고리 → 역할 → 변형. 일관된 구분자(dot 또는 dash).
- 시맨틱 이름은 용도만 (primary, success, error). 리터럴(blue-600)은 글로벌만.
- 대비·접근성 필요 색은 시맨틱 쌍으로 (text.on.primary 등).
