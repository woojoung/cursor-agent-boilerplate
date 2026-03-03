---
name: component-design
description: 컴포넌트 분할·재사용·접근성·테스트 가능 설계.
---

# Component Design Skill (frontend-web)

## 분할 원칙

- **단일 책임**: 한 컴포넌트는 한 가지 역할(예: 폼 제출, 리스트 렌더, 헤더).
- **컨테이너/프레젠테이션**: 데이터·로직은 page 또는 컨테이너, UI만 자식 컴포넌트에 props로 전달.
- **재사용**: 공통 UI는 components/ui, 기능 단위는 components/feature.

## Props·API

- 필수/선택 명확히. 타입으로 문서화 (TypeScript interface). 확장 시 rest 전달 고려.
- 이벤트: onAction 네이밍. 상위에서 처리, 하위는 "무슨 일이 일어났는지"만 전달.

## 접근성

- 시맨틱 태그, role·aria-* 적절히. 키보드·포커스 순서. 라벨·에러 연결.

## 테스트 용이

- props로 주입 가능하게 설계하면 Mock·테스트 시 교체 용이. 사용자 행동(클릭, 입력) 기준으로 검증.
