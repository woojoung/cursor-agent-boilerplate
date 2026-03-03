# docs — 문서 목차

> 이 폴더는 cursor-agent-boilerplate 레포의 **문서**를 모아 둔 곳입니다. 루트 README와 포지션별 AGENTS.md는 상위에 있습니다.

## 하위 구조

| 문서 | 용도 |
|------|------|
| **[GETTING_STARTED_CHECKLIST.md](GETTING_STARTED_CHECKLIST.md)** | **바로 해보기 전** — 완성도 요약, 공통·포지션별 연동·세팅(Slack, Cursor, Figma, Stitch 등) 체크리스트. |
| **[IS_THIS_AN_AGENT.md](IS_THIS_AN_AGENT.md)** | **에이전트라고 부를 수 있나?** — 서브에이전트·역할 정의·에이전트 보일러플레이트 의미 정리. |
| **[PUBLISH_GITHUB_AND_PLUGIN.md](PUBLISH_GITHUB_AND_PLUGIN.md)** | **GitHub 업로드·Cursor 플러그인 등록** — chae3229@gmail.com 계정으로 올리고 마켓플레이스 제출 절차. |
| **[PROMPTS.md](PROMPTS.md)** | Cursor 대화창에서 복사해 쓸 수 있는 **프롬프트 모음**. 포지션별·Slack·역할 정의 레포 참고 맥락 포함. |
| **[INTEGRATION.md](INTEGRATION.md)** | **실무 레포 적용**(플러그인 vs 복사)·**Slack 봇 연동**·팀 워크플로우(기획→개발→리뷰→QA→배포) 가이드. |
| **[EXTENSIBILITY.md](EXTENSIBILITY.md)** | **확장성** — 포지션별 도구·MCP·기술 스택·Slack 추가. Agentation, Figma/Stitch MCP 예시. |
| **[SETUP_REPOS.md](SETUP_REPOS.md)** | **포지션별 기본 레포 세팅** — 백엔드(Java/Spring), 프론트 웹(Next.js), 모바일(Flutter) 생성 절차. .cursor 기술 스택 검증 요약. |
| **[AGENTIC_ENGINEERING.md](AGENTIC_ENGINEERING.md)** | **에이전틱 엔지니어링** — 에이전트가 똑똑·효율적으로 작업하기 위한 9가지 원칙(분해·컨텍스트·DoD·실패 복구·관찰 가능성·메모리·병렬·추상화·감각). GeekNews 기사 반영. |
| **[MODEL_ASSIGNMENT.md](MODEL_ASSIGNMENT.md)** | **모델 배정 원칙** — 판단(Opus)·실행(Sonnet)·탐색(Haiku) 역할별 모델 권장, Cursor 서브에이전트 `model: inherit`/`fast` 적용 가이드, 비용 최소화. |
| **[SILO_AGENT_VISION.md](SILO_AGENT_VISION.md)** | **AI 사일로 조직 비전** — 토스식 사일로를 AI 에이전트로 재현하는 시스템의 실행 가능성, 현재 레포와의 매핑, 아키텍처·메시지 프로토콜·단계별 로드맵. |
| **[READINESS_AND_NEXT_STEPS.md](READINESS_AND_NEXT_STEPS.md)** | **비전 대비 현황·실전 진입** — 얼마나 가까운지, 실전 A(인간+Cursor) vs 실전 B(슬랙 에이전트), 남은 세팅·체크리스트. |
| **[WHY_ORCHESTRATOR_IS_SEPARATE.md](WHY_ORCHESTRATOR_IS_SEPARATE.md)** | **Phase 2b~3가 "별도 서비스"인 이유** — "계약·프롬프트 소스만 제공" 의미, 이 레포 vs 구축할 서비스, 비유·도식·FAQ. |
| **[PHASE2A_CURSOR_SLACK.md](PHASE2A_CURSOR_SLACK.md)** | **Phase 2a: Slack + Cursor** — 슬랙 대화창에서 @Cursor 호출로 Cloud Agent 실행. 별도 봇 없이 §6 템플릿 복사·@Cursor 만 사용. |
| **[AGENT_MESSAGE_PROTOCOL.md](AGENT_MESSAGE_PROTOCOL.md)** | **에이전트 간 메시지 프로토콜** — 이벤트 타입·페이로드 스키마·역할–채널 매핑·상태 전이 규칙·슬랙 메시지 템플릿. 오케스트레이터·런타임 구현 시 계약 문서. |
| **[AGENT_MEMORY_SCHEMA.md](AGENT_MEMORY_SCHEMA.md)** | **에이전트 메모리 스키마** — 역할별 컨텍스트 저장 필드(thread_id, spec_id, pr_url 등) 초안. 런타임 메모리 구현 시 참고. |

## 참고

- 루트 **README.md**: 설치·사용·폴더 구조·실무 적용 요약.
- 각 포지션 폴더의 **AGENTS.md**: 해당 포지션 규칙·명령·스킬·에이전트 요약.
