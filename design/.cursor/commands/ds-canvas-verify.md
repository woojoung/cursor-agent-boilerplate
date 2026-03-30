---
description: DS 캔버스(또는 스토리북)와 기준 이미지를 비교해 시각 차이를 목록화하고, 제한된 횟수로 수정 루프를 돌린다.
---

## 입력

- 기준 이미지: Figma export 또는 사용자 스크린샷
- 대상: `canvas/design-system-canvas.html` 또는 스토리북 URL
- 최대 루프: 기본 3
- 객관적 검증: (예: `pnpm lint`, `pnpm build` — 프로젝트에 맞게)

## 할 일

1. 기준 이미지와 현재 구현을 비교해 **차이 목록** 작성.
2. 한 라운드에 **논리적 수정 단위**로만 패치.
3. 객관적 검증 실행(가능하면).
4. 종료 조건: 차이 없음 / 사용자 동의한 잔여 차이만 남음 / 최대 루프 도달 / 2회 연속 미세 변화 시 사용자 승인 요청.

## 금지

- 종료 조건·상한 없는 무한 루프.

## 관련 스킬

- `.cursor/skills/design-system-visual-canvas/SKILL.md`
- `.cursor/skills/figma-to-code-workflow/SKILL.md` (자동 검증 루프 규칙과 정합)
