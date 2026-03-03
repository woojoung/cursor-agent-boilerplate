# AGENTS.md — 인프라/DevOps

> 이 포지션에서 Cursor 등 AI 에이전트가 참고하는 공통 맥락입니다.

## 핵심

- 항상 **한국어**로 응답합니다.
- IaC·컨테이너·배포 규칙, 보안·모니터링 관례를 기준으로 합니다. 파이프라인·배포 설계, 보안 점검, 장애·롤백 절차를 사용합니다.
- **AGENTS.md**, **spec.md**, **plan.md**, **task.md**를 참고하며, 문서화·히스토리를 유지합니다.

## 규칙·명령·스킬·에이전트

- **Rules**: `.cursor/rules/` (main, iac, security, agents, skills) — IaC·Terraform·Docker·보안·런북.
- **Commands**: `/start`(역질문), `/plan-from-spec`, `/review-infra`, `/runbook-draft`.
- **Skills**: terraform-patterns, pipeline-design, runbook-template.
- **Agents**: planner, infra-reviewer, runbook-writer.

### 역질문·템플릿 (포지션을 모르는 사용자용)

- **PROMPT_TEMPLATES.md**: 자주 쓰는 요청과 역질문 옵션(① 인프라 계획 ② 인프라 리뷰 ③ 런북 초안).
- 사용자가 `/start` 또는 "도와줘", "뭐 할 수 있어?" 등 **모호한 요청**을 하면: 옵션을 제시하고, 선택에 따라 해당 명령을 실행·안내한다.

### 완료 시 슬랙/오케스트레이터에 전달할 산출물

- **배포 완료/실패 시**: `deploy_done` 또는 `deploy_failed` 이벤트 — env, success, rollback_done, message(선택). [docs/AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §2.6 참고.

실무에 맞게 tfsec·파이프라인·런북 형식을 보완해 사용하세요.
