---
name: design-system-visual-canvas
description: >-
  디자인 시스템·컴포넌트를 로컬 HTML 캔버스(웹·모바일 뷰포트)로 시각화하고, 토큰 적용·상태 매트릭스·검증 루프를 수행한다.
  Use when building or reviewing DS previews, component states, responsive checks, or Figma-handoff visual parity without a full app run.
---

# 디자인 시스템 시각 캔버스

## 목표

- **토큰(색·타이포·간격·반경)**이 코드(또는 CSS 변수)와 일치하는지 **한 화면**에서 확인한다.
- **컴포넌트**를 **상태 × 뷰포트(웹/모바일)** 매트릭스로 보여 **누락·깨짐**을 빨리 찾는다.
- Figma 핸드오프와 병행할 때 **“보이는 기준”**을 팀이 공유한다.

## 산출물 규칙

- 기본 출력: 프로젝트 루트 또는 `design/` 기준 `canvas/design-system-canvas.html` (단일 파일 우선; 팀 표준 경로가 있으면 그곳).
- **외부 빌드 도구 필수 아님**: 순수 HTML + CSS (`:root` 변수) + 최소 JS로도 가능. 팀이 React/Vite 스토리북을 쓰면 그 URL을 “캔버스”로 대체해도 됨.
- **웹·모바일 동시 표시**: 두 개의 프레임(예: 1280px / 390px) 또는 CSS `@media`로 동일 페이지 내 비교.

## 캔버스에 반드시 포함할 섹션

1. **토큰 스와치**: color, radius, spacing 스텝, font scale (실제 값 + 변수명).
2. **컴포넌트 매트릭스**: 각 컴포넌트별 `default / hover / disabled / error / loading` 등 팀이 정한 상태.
3. **레이아웃 스모크**: flex/grid 깨짐, 줄바꿈, 최소 너비에서 overflow.
4. **다크 모드**(사용 시): 토글 또는 별도 블록.

## 데이터 원칙

- **진실 공급원**: 레포의 토큰 정의 파일(또는 CSS 변수 생성 결과). 캔버스는 **import/빌드 산출물과 동기화**되도록 유지(수동이면 “마지막 동기 일자” 주석).
- Figma와 1:1이 아닐 수 있음 → 차이는 **의도적/토큰 한도**로 라벨링.

## `/ds-canvas-build` 호출 시 에이전트 행동

1. 사용자에게 **토큰 소스 경로**, **포함할 컴포넌트 목록**, **필수 상태 목록**을 확인(없으면 질문).
2. `canvas/design-system-canvas.html` 생성 또는 갱신 (경로는 사용자·레포 규칙에 맞게 조정).
3. 로컬에서 열어 깨짐 여부를 확인할 수 있게 **파일 경로**를 응답에 명시.

## `/ds-canvas-verify` 호출 시 에이전트 행동

1. **기준 이미지** 확보: Figma `export_node_as_image` 또는 사용자 제공 스크린샷.
2. 캔버스(또는 스토리북 URL)와 **나란히 비교**해 차이를 **글머리 목록**으로 작성.
3. **최대 반복 횟수**(기본 3)와 **객관적 검증**(린트/빌드 등)을 사용자와 합의 후 루프.
4. 2회 연속 차이가 거의 없으면 사용자 승인 요청 후 종료.

## 금지

- 토큰을 캔버스에만 하드코딩하고 레포와 영구적으로 엇갈리게 두기.
- 컴포넌트 상태를 임의로 줄여 “완료” 처리하기.

## 관련 명령

- `/ds-canvas-build`
- `/ds-canvas-verify`
