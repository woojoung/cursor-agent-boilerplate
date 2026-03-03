# AGENTS.md — 기획/PM

> 이 포지션에서 Cursor 등 AI 에이전트가 참고하는 공통 맥락입니다.

## 핵심

- 항상 **한국어**로 응답합니다.
- 요구정의·스펙 문서 형식, 이슈·태스크 템플릿을 기준으로 합니다. 스펙 초안, 태스크 분해, 리스크·의존성 체크, KPI/지표 정리를 사용합니다.
- **AGENTS.md**, **spec.md**, **plan.md**, **task.md**를 참고하며, 문서화·히스토리를 유지합니다.

## 규칙·명령·스킬·에이전트

- **Rules**: `.cursor/rules/` (main, spec-format, issue-template, agents, skills) — 스펙·이슈·태스크·리스크.
- **Commands**: `/start`(역질문), `/spec-template`, `/task-breakdown`, `/risk-check`.
- **Skills**: spec-draft, task-breakdown, risk-dependency.
- **Agents**: spec-writer, task-planner, risk-analyzer.

### 역질문·템플릿 (포지션을 모르는 사용자용)

- **PROMPT_TEMPLATES.md**: 이 포지션에서 자주 쓰는 요청 목록과, 에이전트가 **역으로 질문**할 때 제시할 옵션.
- 사용자가 `/start`를 실행하거나 "도와줘", "뭐 할 수 있어?" 등 **작업 내용이 불명확**할 때: PROMPT_TEMPLATES.md의 옵션(① 스펙 초안 ② 태스크 분해 ③ 리스크 점검 ④ 이슈 템플릿)을 보여 주고, 선택에 따라 추가 질문 후 해당 명령을 실행하거나 안내한다.

### 완료 시 슬랙/오케스트레이터에 전달할 산출물

- **스펙 완료 시**: `spec_completed` 이벤트 — spec 링크(또는 이슈 URL), 목표 한 줄, AC 목록, 우선순위(P0/P1/P2). [docs/AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §2.1 참고.
- **태스크 분해·할당 완료 시**: `task_ready_for_impl` — task_id, spec_ref, ac_ref, branch_name.

실무에 맞게 스펙·이슈 템플릿을 보완해 사용하세요.
