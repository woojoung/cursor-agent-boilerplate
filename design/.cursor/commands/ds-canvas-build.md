---
description: 로컬 HTML(또는 팀 표준)로 DS·컴포넌트 웹/모바일 시각 캔버스를 생성·갱신한다.
---

## 입력 (사용자가 제공하거나 에이전트가 질문)

- 토큰 소스: (파일 경로 또는 CSS 변수 생성 방식)
- 컴포넌트 범위: (예: Button, Input, Card, Modal…)
- 필수 상태: (예: default, hover, disabled, error, loading)
- 뷰포트: 웹 기준 폭, 모바일 기준 폭 (기본 1280 / 390)
- 출력 경로: 기본 `canvas/design-system-canvas.html` (레포 루트 또는 `design/canvas/` 등 팀 규칙에 맞게 조정)

## 할 일

1. `design-system-visual-canvas` 스킬 규칙에 맞춰 단일 HTML(또는 팀 표준) 작성.
2. 토큰 스와치 + 컴포넌트 매트릭스 + 웹/모바일 프레임 포함.
3. 생성 후 열어볼 경로를 사용자에게 알림.

## 완료 조건

- 파일이 생성/갱신되었고, 섹션(토큰·매트릭스·레이아웃 스모크·다크 옵션)이 비어 있지 않음.

## 관련 스킬

- `.cursor/skills/design-system-visual-canvas/SKILL.md`
