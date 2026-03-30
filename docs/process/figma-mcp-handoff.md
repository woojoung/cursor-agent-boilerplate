# Figma MCP 핸드오프 프로세스

디자인 시안을 코드로 옮길 때 **Talk to Figma MCP**와 레포 **진실 공급원(토큰·컴포넌트)**을 맞추는 절차입니다.

참고: [Figmapedia — 피그마&클로드 실무 치트키들](https://figmapedia.com/entry/327fdea8-0034-8084-992e-ced9b4e29ae1)

## 1. 역할

- **디자인**: 컴포넌트·변수(토큰) 네이밍을 코드/매핑 문서와 정렬. 반복 UI는 **컴포넌트화**해 구현 단위와 맞춘다.
- **개발**: 레포 토큰·공용 컴포넌트를 진실 공급원으로 유지한다.

## 2. 매 세션 연결 (채널)

1. Figma에서 Talk to Figma(또는 동일) **플러그인 실행**.
2. 표시된 **채널** 문자열을 복사 → Cursor 채팅에 붙여넣기.
3. **중요**: 채널은 연결마다 갱신될 수 있으며, **AI가 MCP만으로 채널을 자동 조회하는 것은 일반적으로 불가**하다. 사용자 입력이 전제다.
4. 에이전트가 `join_channel` 후 문서를 읽거나, **`/figma-mcp-precheck`** 명령(design 포지션)으로 점검한다.

## 3. 전제 점검 (MCP + 레포)

| 단계 | 도구·행동 |
|------|-----------|
| 연결 | `join_channel` (사용자 제공 채널) |
| 문서 | `get_document_info` |
| Figma 메타 | `get_styles`, `get_local_components` |
| 레포 | 토큰·컴포넌트·매핑 문서 경로를 읽고 이름 **갭 표** 작성 |

**한계**

- **노드/레이어 이름 일괄 변경**은 Talk to Figma MCP 도구에 보통 없음 → Figma에서 수동 처리.
- **전 노드 네이밍 완벽**은 비현실적일 수 있음 → **섹션 프레임·컴포넌트·인스턴스** 이름과 **annotations**를 우선한다.

## 4. 구현 이후

- 스킬 **figma-to-code-workflow**: Phase 1(읽기: 노드+스크린샷) → Phase 2(구현) → Phase 3(검증).
- 선택: **에이전트 자동 검증 루프** — 최대 반복 횟수·종료 조건 필수.
- 디자인 시스템을 **로컬 HTML 등으로 시각 확인**하려면 **design-system-visual-canvas** 스킬 및 `/ds-canvas-build`, `/ds-canvas-verify`.

## 5. 보일러플레이트 내 위치

| 종류 | 경로 |
|------|------|
| Figma→코드 스킬 | `design/.cursor/skills/figma-to-code-workflow/SKILL.md` |
| DS 시각 캔버스 스킬 | `design/.cursor/skills/design-system-visual-canvas/SKILL.md` |
| 명령: MCP 전제 점검 | `design/.cursor/commands/figma-mcp-precheck.md` |
| 명령: 캔버스 생성 | `design/.cursor/commands/ds-canvas-build.md` |
| 명령: 캔버스 검증 | `design/.cursor/commands/ds-canvas-verify.md` |
| 도구·MCP 개요 | `design/TOOLS_AND_MCP.md` |

## 6. 프론트엔드 포지션과의 관계

- **frontend-web** / **frontend-mobile**에서도 동일 MCP로 구현할 수 있다. 이 프로세스 문서는 **공통 핸드오프 기준**으로 참고하고, 구현 스택에 맞게 빌드·검증 명령만 바꾼다.
