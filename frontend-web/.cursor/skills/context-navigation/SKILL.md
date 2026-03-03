---
name: context-navigation
description: AGENTS.md·폴더 구조 기반 Next.js 프로젝트 맥락 파악. app/components/lib 구조 탐색.
---

# Context Navigation Skill (frontend-web)

AGENTS.md와 Next.js 폴더 구조를 활용해 코드베이스 맥락을 빠르게 파악하는 스킬입니다.

## 목적

- app/ 라우트·layout·페이지 구조 이해
- components/ 기능별·UI별 컴포넌트 위치 파악
- lib/, hooks/, types/ 의존성·유틸 위치 확인

## 탐색 전략

1. **루트 AGENTS.md** → 프로젝트 전체·포지션 규칙.
2. **app/** → (routes), layout, page, loading, error 구조.
3. **components/** → ui / feature / layout 구분.
4. **의존성**: import 경로 따라가며 lib·hooks·types 참조 확인.

## 활용

- "이 기능 관련 컴포넌트 어디 있어?" → AGENTS.md 또는 components/feature, app/ 대응 경로.
- "이 훅을 쓰는 곳은?" → grep/참조 검색 후 해당 페이지·컴포넌트 AGENTS.md 또는 파일 주석.
