# AGENTS.md — 프론트엔드(모바일·Flutter)

> 이 포지션에서 Cursor 등 AI 에이전트가 참고하는 공통 맥락입니다.

## 핵심

- 항상 **한국어**로 응답합니다.
- **Flutter/Dart** 기반 모바일 개발 시: 위젯·상태·플랫폼 채널, 위젯 설계·테스트·디자인 시스템 연동·스크린 플로우를 기준으로 작업합니다.
- **AGENTS.md**, **spec.md**, **plan.md**, **task.md**를 참고하며, 문서화·히스토리를 유지합니다.

## 규칙·명령·스킬·에이전트

- **Rules**: `.cursor/rules/` (main, coding-style, testing, agents, skills) — Flutter 레이어·위젯·ViewModel·테스트.
- **Commands**: `/start`(역질문), `/spec-template`, `/plan-from-spec`, `/review-code`, `/mr-gitlab`, `/setup-repo`.
- **Skills**: context-navigation, spec-template, flutter-patterns, widget-design.
- **Agents**: planner, code-reviewer, widget-designer, test-runner.

### 역질문·템플릿 (포지션을 모르는 사용자용)

- **PROMPT_TEMPLATES.md**: 자주 쓰는 요청과 역질문 옵션(① 스펙→계획→태스크 ② 코드 리뷰 ③ MR 초안 ④ 위젯·단위 테스트 ⑤ 새 레포 세팅).
- 사용자가 `/start` 또는 "도와줘", "뭐 할 수 있어?" 등 **모호한 요청**을 하면: 옵션을 제시하고, 선택에 따라 해당 명령을 실행·안내한다.

### 완료 시 슬랙/오케스트레이터에 전달할 산출물

- **PR 생성 시**: `pr_opened` 이벤트 — pr_url, branch, change_summary, regression_hint, author_role(frontend-mobile). [docs/AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §2.4 참고.

실무에 맞게 규칙·스킬 내용을 보완해 사용하세요.
