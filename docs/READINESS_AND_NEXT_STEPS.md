# 목표 비전 대비 현황 · 실전 진입 전 점검

> **목적**: "토스식 AI 사일로" 비전에 **지금 얼마나 가까운지**, **바로 실전에 들어갈 수 있는지**, **남은 세팅**이 무엇인지 한눈에 파악할 수 있도록 정리합니다.

---

## 1. 비전 대비 "얼마나 가까운가?"

### 1.1 이미 갖춰진 것 (이 레포에서 완료)

| 영역 | 완료 내용 | 비고 |
|------|-----------|------|
| **역할 정의** | pm, design, backend, frontend-web, frontend-mobile, qa, infra — FRIDAY·Black Widow·Spider-Man·Hulk·Hawkeye 매핑 | [SILO_AGENT_VISION.md §2](SILO_AGENT_VISION.md) |
| **워크플로우·규칙** | spec → plan → task → 구현 → 리뷰 → MR → 배포, 포지션별 rules/commands/skills/agents | 각 포지션 `.cursor/`, AGENTS.md |
| **역할별 산출물·핸드오프** | 완료 시 "누가 누구에게 무엇을" 전달할지 AGENTS.md에 명시, 이벤트 타입·페이로드 스키마 문서화 | [AGENT_MESSAGE_PROTOCOL.md](AGENT_MESSAGE_PROTOCOL.md) |
| **역할–채널 매핑** | Slack 채널 ↔ 포지션/역할 예시 표 | AGENT_MESSAGE_PROTOCOL.md §4 |
| **상태 전이 규칙** | TODO → IN_PROGRESS → BLOCKED → DONE, 이벤트 연계 | AGENT_MESSAGE_PROTOCOL.md §5 |
| **슬랙 메시지 템플릿** | spec_completed, pr_opened 등 이벤트별 메시지 예시 | AGENT_MESSAGE_PROTOCOL.md §6 |
| **에이전트 메모리 스키마** | 역할별 저장 필드 초안(thread_id, spec_id, pr_url 등) | [AGENT_MEMORY_SCHEMA.md](AGENT_MEMORY_SCHEMA.md) |
| **슬랙 에이전트용 프롬프트** | pm, design, backend, frontend-web, qa — `slack-agent-prompt.md` | 각 포지션 폴더 |
| **역질문·템플릿** | PROMPT_TEMPLATES.md, `/start` 명령으로 포지션별 옵션 제시 | 포지션별 |
| **모델 배정 가이드** | 판단(Opus)·실행(Sonnet)·탐색(Haiku), 서브에이전트 inherit/fast | [MODEL_ASSIGNMENT.md](MODEL_ASSIGNMENT.md) |

**요약**: **역할·프로토콜·프롬프트·문서** 관점에서는 비전에 필요한 **"단일 소스(계약)"** 가 이 레포에 갖춰져 있습니다.  
즉, **"에이전트가 누구이고, 무엇을 하고, 무엇을 주고받는지"** 는 정의된 상태입니다.

### 1.2 아직 없는 것 (이 레포 밖에서 구축할 것)

| 영역 | 필요한 것 | 비고 |
|------|-----------|------|
| **Slack 앱·봇** | 스레드 기반 대화, 멘션, 이벤트 수신·발송 | Phase 2a부터 사용 |
| **오케스트레이터 서비스** | 작업·상태(TODO/IN_PROGRESS/BLOCKED/DONE) 관리, 이벤트에 따른 전이·알림 | Phase 2b–3 |
| **에이전트 런타임** | Cursor 밖에서 LLM 호출 + 이 레포 프롬프트 로드 → Slack/산출물 전송 | Phase 3a–3b |
| **에이전트 메모리 저장소** | 스레드/태스크별 컨텍스트 저장 (벡터 DB 또는 KV) | Phase 3a–3b |
| **GitHub 연동** | 브랜치·PR 생성, CI 상태 구독, 머지 정책(QA 승인 시만) | Phase 3c |
| **CI/CD·롤백** | 배포 실패 시 자동 롤백 + Slack 알림 | Phase 3c |

**요약**: **슬랙 위에서 에이전트가 스스로 대화·리뷰·에스컬레이션** 하려면, 위 **런타임·인프라**를 별도 서비스로 구현해야 합니다. 이 레포는 그때 **역할 정의·프로토콜·프롬프트의 소스**로만 쓰입니다.

---

## 2. "실전"의 두 가지 의미

### 실전 A: 인간이 Cursor로 역할별 에이전트 사용 (지금 가능)

- **의미**: 팀원이 **Cursor를 열고**, 이 보일러플레이트(플러그인 또는 복사)를 적용한 **실무 레포**에서, 포지션별 rules/commands/agents로 작업. Slack은 "할 일 공유·알림" 정도만 사용.
- **준비도**: ✅ **바로 실전 가능.**  
  - 이 레포 클론 → 실무 레포에 플러그인 또는 `.cursor/`·AGENTS.md 복사 → Cursor에서 해당 레포 열고 작업.
- **남은 세팅**: 아래 §3 "실전 A 체크리스트"만 점검하면 됨.

### 실전 B: 슬랙 에이전트가 스스로 협업 (추가 구축 필요)

- **의미**: PO·디자이너·FE·BE·QA 에이전트가 **Slack 스레드에서 직접** 메시지 주고받고, PR 생성·리뷰·머지 차단까지 (반)자동.
- **준비도**: ⚠️ **아직 불가.**  
  - 이 레포에는 "무엇을 할지·무엇을 주고받을지"만 정의되어 있고, **Slack 봇 + 오케스트레이터 + 에이전트 런타임**이 없음.
- **남은 세팅**: §4 "실전 B를 위한 남은 구축" 참고.

---

## 3. 실전 A: "지금 바로" 들어가기 전 체크리스트

인간이 Cursor로 포지션별 에이전트를 쓰는 **실전 A**만 쓸 때, 아래만 확인하면 됩니다.

- [ ] **역할 정의 레포**: 이 보일러플레이트(또는 포크)를 팀이 접근 가능한 곳에 두었는가?
- [ ] **실무 레포 적용**:  
  - [ ] 사용할 **포지션 폴더**를 플러그인으로 추가했는가, 또는 `.cursor/`·AGENTS.md를 복사했는가?  
  - [ ] 복사한 경우, 팀 규칙(브랜치 전략, 리뷰 기준 등)에 맞게 rules/AGENTS.md를 조정했는가?
- [ ] **스펙·플로우**: spec → plan → task 순서를 팀이 따르기로 했는가? (필요 시 plan.md·task.md 템플릿 위치 공유.)
- [ ] **Slack(선택)**: "할 일 + 프롬프트 템플릿"만 전달하는 봇을 쓸 경우, 채널·멘션 규칙만 정하면 됨.  
  - [ ] INTEGRATION.md §3·§4의 역할–채널 매핑을 참고해 채널 구조를 정했는가?

위가 끝나면 **실전 A는 바로 시작 가능**합니다. 추가로 "에이전트가 슬랙에서 직접 행동"하는 기능은 실전 B 단계에서 구축하면 됩니다.

---

## 4. 실전 B: 슬랙 에이전트 협업을 위해 남은 구축

실전 B(슬랙 위 자동 협업)를 목표로 할 때, **이 레포 다음으로** 할 일을 단계별로 정리했습니다.

| 단계 | 할 일 | 이 레포에서 참고할 문서 |
|------|--------|---------------------------|
| **Phase 2a** | **Cursor Slack 연동** 설정 후, 슬랙 대화창에서 **@Cursor** + 지시문으로 Cloud Agent 실행. "할 일"은 사람이 적거나 [AGENT_MESSAGE_PROTOCOL](AGENT_MESSAGE_PROTOCOL.md) §6 템플릿 복사. 별도 봇 코드 없음. | [PHASE2A_CURSOR_SLACK.md](PHASE2A_CURSOR_SLACK.md), AGENT_MESSAGE_PROTOCOL.md §6 |
| **Phase 2b** | 오케스트레이터 서비스: 상태(TODO/IN_PROGRESS/DONE) 저장, Slack 알림, (선택) GitHub 이슈 연동 | AGENT_MESSAGE_PROTOCOL.md §4·§5 |
| **Phase 3a** | **한 역할만** 에이전트 런타임(예: PO): 슬랙 메시지 수신 → pm/ 프롬프트 로드 → LLM → 스레드 답변 | pm/slack-agent-prompt.md, AGENT_MESSAGE_PROTOCOL.md |
| **Phase 3b** | 역할별 에이전트 런타임 확장, 메시지 프로토콜 구현, (선택) PR 반자동 생성 | 각 포지션 slack-agent-prompt.md, AGENT_MEMORY_SCHEMA.md |
| **Phase 3c** | GitHub API(브랜치·PR·CI), QA 머지 차단 로직, 배포 실패 시 롤백 + Slack 알림 | qa/ rules·AGENTS.md, AGENT_MESSAGE_PROTOCOL.md |

상세 로드맵은 [SILO_AGENT_VISION.md §6](SILO_AGENT_VISION.md)에 있습니다.

---

## 5. 한 줄 요약

| 질문 | 답 |
|------|----|
| **목표 비전에 어느 정도 가까운가?** | **역할·프로토콜·프롬프트·문서**는 비전에 맞게 갖춰져 있음. **실행 인프라**(슬랙 봇, 오케스트레이터, 에이전트 런타임, GitHub/CI)는 아직 없음. |
| **바로 실전에 들어갈 수 있는가?** | **실전 A**(인간이 Cursor로 역할별 에이전트 사용)는 **가능**. §3 체크리스트만 하면 됨. **실전 B**(슬랙 에이전트가 스스로 협업)는 **불가** — Phase 2a부터 순서대로 구축 필요. |
| **남은 세팅이 있는가?** | **실전 A**: 플러그인/복사 적용, 스펙 플로우·(선택) Slack 채널만 점검. **실전 B**: Slack 앱 → 오케스트레이터 → 에이전트 런타임 → 메모리 → GitHub/CI 순으로 구축. |
