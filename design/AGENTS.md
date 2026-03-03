# AGENTS.md — 디자인/브랜딩

> 이 포지션에서 Cursor 등 AI 에이전트가 참고하는 공통 맥락입니다.

## 핵심

- 항상 **한국어**로 응답합니다.
- 디자인 시스템, 컴포넌트·토큰 명명, 가이드라인을 기준으로 합니다. UI 컴포넌트 규격, 브랜드 가이드 반영, 접근성·반응형 체크를 사용합니다.
- **AGENTS.md**, **spec.md**, **plan.md**, **task.md**를 참고하며, 문서화·히스토리를 유지합니다.

## 규칙·명령·스킬·에이전트

- **Rules**: `.cursor/rules/` (main, design-system, tokens, a11y, agents, skills) — 디자인 시스템·토큰·접근성.
- **Commands**: `/start`(역질문), `/component-spec`, `/design-review`.
- **Skills**: token-naming, component-spec, brand-guide.
- **Agents**: design-reviewer, token-validator, component-spec-writer.

### 역질문·템플릿 (포지션을 모르는 사용자용)

- **PROMPT_TEMPLATES.md**: 자주 쓰는 요청과 에이전트가 **역으로 질문**할 때 제시할 옵션(① 컴포넌트 규격 ② 디자인 리뷰 ③ 토큰 검증 ④ Figma/핸드오프 안내).
- 사용자가 `/start` 또는 "도와줘", "뭐 할 수 있어?" 등 **모호한 요청**을 하면: 옵션을 제시하고, 선택에 따라 추가 질문 후 해당 명령을 실행·안내한다.

### 완료 시 슬랙/오케스트레이터에 전달할 산출물

- **디자인 스펙/와이어프레임 완료 시**: `design_completed` 이벤트 — design_spec_link, figma_link(선택), tokens, variants. [docs/AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §2.2 참고.

실무에 맞게 토큰·브랜드 가이드를 보완해 사용하세요.

## 확장 (도구·MCP)

- **TOOLS_AND_MCP.md**: Figma MCP, Stitch 등 **디자인→프론트엔드** 연동 도구·MCP 목록·연동 방법. 디자이너 에이전트가 만든 UI를 Figma에 두고 프론트엔드에서 MCP로 사용하는 흐름을 이 파일에 정리할 수 있습니다.
- 전체 확장 가이드: [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md).
