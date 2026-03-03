# AGENTS.md — QA/테스트

> 이 포지션에서 Cursor 등 AI 에이전트가 참고하는 공통 맥락입니다.

## 핵심

- 항상 **한국어**로 응답합니다.
- 테스트 케이스·시나리오 작성법, 자동화 표준을 기준으로 합니다. 시나리오 도출, 회귀 범위 산정, 버그 리포트·재현 절차 템플릿을 사용합니다.
- **AGENTS.md**, **spec.md**, **plan.md**, **task.md**를 참고하며, 문서화·히스토리를 유지합니다.

## 규칙·명령·스킬·에이전트

- **Rules**: `.cursor/rules/` (main, test-scenario, automation, agents, skills) — 시나리오·자동화·버그 리포트.
- **Commands**: `/start`(역질문), `/scenario-from-spec`, `/test-plan`, `/bug-template`.
- **Skills**: scenario-writing, regression-scope, bug-report-template.
- **Agents**: scenario-writer, test-planner, bug-analyzer.

### 역질문·템플릿 (포지션을 모르는 사용자용)

- **PROMPT_TEMPLATES.md**: 자주 쓰는 요청과 역질문 옵션(① 스펙→시나리오 ② 테스트 계획 ③ 버그 리포트 템플릿).
- 사용자가 `/start` 또는 "도와줘", "뭐 할 수 있어?" 등 **모호한 요청**을 하면: 옵션을 제시하고, 선택에 따라 해당 명령을 실행·안내한다.

### 완료 시 슬랙/오케스트레이터에 전달할 산출물

- **리뷰·테스트 통과 시**: `qa_approved` 이벤트 — pr_url, approved: true, reason(선택). **미통과 시**: `qa_blocked` — pr_url, approved: false, reason. [docs/AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §2.5 참고.

### 머지 승인 조건 (오케스트레이터 정책 참고)

QA가 **머지 승인(qa_approved)** 을 내기 위한 조건. 오케스트레이터는 "QA 승인 시에만 머지" 로직을 적용할 때 아래를 정책으로 참고합니다.

- **필수**: (1) CI 빌드·테스트 파이프라인 통과, (2) 코드 리뷰 완료(규정된 리뷰어 승인), (3) 해당 PR에 대한 회귀 범위 확인 완료(또는 회귀 테스트 통과).
- **차단(qa_blocked)**: 위 중 하나라도 미충족 시 머지 금지. reason에 "테스트 실패", "리뷰 미완료", "회귀 이슈" 등 사유를 명시.
- 실무에서 추가 조건(보안 스캔, 문서 변경 등)이 있으면 팀 규칙으로 이 목록을 확장합니다.

실무에 맞게 시나리오 형식·자동화 도구를 보완해 사용하세요.
