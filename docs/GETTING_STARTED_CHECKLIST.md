# 바로 해보기 전 체크리스트 — 완성도 & 포지션별 연동·세팅

> **목적**: 보일러플레이트가 **어느 정도 완성됐는지**, **바로 시도해도 되는지**, 그리고 **Slack·Cursor·Figma·Stitch 등 포지션별로 어떤 연동·세팅**이 필요한지 한 문서에서 점검할 수 있게 정리합니다.

---

## 1. 어느 정도 완성된 거야?

| 구분 | 상태 | 설명 |
|------|------|------|
| **역할·규칙·프로토콜** | ✅ 완료 | pm, design, backend, frontend-web, frontend-mobile, qa, infra — 역할 정의, rules/commands/skills/agents, AGENTS.md, 메시지 프로토콜, 핸드오프·템플릿 모두 정의됨. |
| **실전 A (Cursor만)** | ✅ 바로 가능 | 이 레포를 플러그인 또는 복사로 실무 레포에 적용 → Cursor에서 해당 포지션으로 작업. 추가 인프라 없이 가능. |
| **실전 B Phase 2a (Slack + Cursor)** | ✅ 바로 가능 | Cursor Dashboard에서 Slack 연동만 설정하면, 슬랙에서 @Cursor 호출로 Cloud Agent 사용 가능. 별도 봇 코드 없음. |
| **실전 B Phase 2b~3** | ⚠️ 구축 필요 | 오케스트레이터·에이전트 런타임·메모리·GitHub/CI는 별도 서비스로 구현 필요. 이 레포는 “계약·프롬프트 소스”만 제공. |

**한 줄**: **지금 단계에서 “바로 해보기” = 실전 A 또는 Phase 2a까지.** Cursor(+ 선택적으로 Slack)만 세팅하면 되고, Figma/Stitch 등은 **쓰고 싶은 포지션**에서만 추가로 연동하면 됩니다.

---

## 2. 이제 바로 해보면 돼?

**네.** 아래 **공통 세팅**과 **포지션별 연동·세팅**만 진행하면 됩니다.

- **공통**: Cursor 설치·실무 레포에 보일러플레이트 적용(플러그인 또는 복사).
- **Slack 쓸 때**: Cursor Dashboard → Integrations → Slack 연결.
- **포지션별**: 사용할 포지션에 맞춰 Figma, Stitch, MCP 등은 **선택**으로 추가.

---

## 2.1 뭐부터 세팅하면 돼? (권장 순서)

| 순서 | 세팅 | 왜 먼저? | 하고 나면 |
|------|------|----------|-----------|
| **1** | **Cursor** | 에이전트·규칙·명령이 전부 Cursor 안에서 돌아감. 이걸 안 하면 아무것도 못 씀. | Cursor에서 실무 레포 열고 `/start`, `/plan-from-spec` 등 사용 가능. |
| **2** | **GitHub (또는 GitLab)** | 실무 코드가 이미 올라가 있는 곳. Cursor가 레포 열고, Cloud Agent는 클론·푸시할 때 필요. | Cursor에서 레포 연결, (선택) Cloud Agent가 해당 레포 기준으로 작업. |
| **3** | **Slack** | Cursor만으로도 가능하지만, 슬랙에서 @Cursor 쓰려면 연동 필요. | 슬랙 채널에서 @Cursor 호출 → Cloud Agent 실행 → 결과 슬랙 수신. |
| **4~** | **Figma, Stitch** | design / frontend-web 포지션에서 디자인→코드 쓸 때만 필요. | 디자인 링크로 규격 정리·컴포넌트 생성 요청 가능. |
| **4~** | **기타 MCP** | 포지션별로 필요할 때만 (이슈 트래커, DB, E2E 등). | 해당 도구와 Cursor 연동. |

**한 줄**: **1 → Cursor, 2 → GitHub(레포), 3 → Slack.** 그 다음에 Figma·Stitch·기타는 쓰는 포지션에서만.

### 최소 경로 (오늘 당장 해보기)

1. **Cursor** 설치·로그인.
2. **실무 레포**를 GitHub/GitLab에서 클론(이미 있으면 생략) → Cursor에서 **Open**.
3. 이 보일러플레이트의 **해당 포지션 폴더**를 **플러그인**으로 추가하거나, `.cursor/` + `AGENTS.md` **복사**해서 실무 레포에 넣기.
4. Cursor 채팅에서 `/start` 또는 "스펙 기준으로 계획 세워줘" 해보기.

→ 여기까지면 **Slack·Figma 없이** Cursor만으로 실전 A 가능.  
Slack 쓰고 싶으면 **Cursor Dashboard → Integrations → Slack** 연결 후 슬랙에서 @Cursor 호출.

---

## 3. 공통 세팅 (모든 포지션)

| 항목 | 할 일 | 참고 |
|------|--------|------|
| **Cursor** | 설치·로그인. (선택) Max Mode·모델 선택. | [Cursor](https://cursor.com) |
| **보일러플레이트 적용** | 실무 레포에 **플러그인**으로 해당 포지션 폴더 추가, 또는 **복사**로 `.cursor/` + `AGENTS.md` 반영. | [INTEGRATION.md](INTEGRATION.md) §2 |
| **실무 레포** | 작업할 포지션에 맞는 레포(백엔드/프론트 웹/모바일 등)를 Cursor에서 Open. | [SETUP_REPOS.md](SETUP_REPOS.md), `/setup-repo` |
| **스펙 플로우** | spec → plan → task 순서 사용할지 팀에 공유. (선택) plan.md·task.md 위치 정함. | 각 포지션 AGENTS.md |

---

## 4. 포지션별 연동·세팅 (필수 아님, 쓰는 것만)

아래는 **해당 포지션으로 일할 때** 쓰고 싶은 도구만 골라 세팅하면 됩니다.

| 포지션 | 연동/도구 | 세팅 요약 | 참고 |
|--------|-----------|-----------|------|
| **공통 (Slack 사용 시)** | **Slack + Cursor** | Cursor Dashboard → Integrations → Slack 연결. 슬랙에서 @Cursor + 지시문으로 Cloud Agent 실행. | [PHASE2A_CURSOR_SLACK.md](PHASE2A_CURSOR_SLACK.md) |
| **design** | **Figma** | Figma Personal Access Token 발급. Cursor MCP에 Figma MCP 서버 추가. 디자인 링크로 “이 디자인 기준으로 규격 정리해줘” 등 요청. | [design/TOOLS_AND_MCP.md](../design/TOOLS_AND_MCP.md), [EXTENSIBILITY.md](EXTENSIBILITY.md) §4 |
| **design** | **Stitch** | Figma와 연동해 디자인→코드. design에서 Figma 정리 후 Stitch/내보내기, frontend-web에서 동일 소스 사용. 팀 사용 시 TOOLS_AND_MCP에 설치·연동 단계 추가. | [design/TOOLS_AND_MCP.md](../design/TOOLS_AND_MCP.md) §2 |
| **frontend-web** | **Figma MCP** | 위와 동일 Figma MCP. “이 Figma 디자인으로 컴포넌트/페이지 구현해줘” 요청. | [frontend-web/TOOLS_AND_MCP.md](../frontend-web/TOOLS_AND_MCP.md), [EXTENSIBILITY.md](EXTENSIBILITY.md) §4 |
| **frontend-web** | **Stitch** | Figma → 코드 변환. Figma MCP와 함께 사용. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §4 |
| **frontend-web** | **Agentation** (선택) | 시각 피드백 도구. `npm install agentation`, 개발 환경에만 포함. “여기 패딩 늘려줘” 등 영역 지정 피드백. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §3 |
| **backend** | DB·API MCP (선택) | 팀에서 쓰는 DB/API MCP가 있으면 backend/TOOLS_AND_MCP.md에 추가 후 Cursor MCP 설정. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §2 |
| **qa** | 테스트·E2E MCP (선택) | 테스트 러너·E2E·버그 트래킹 MCP가 있으면 qa/TOOLS_AND_MCP.md에 추가. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §2 |
| **pm** | 이슈 트래커·노션 MCP (선택) | Jira, Linear, Notion 등 MCP 사용 시 pm/TOOLS_AND_MCP.md에 추가. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §2 |
| **infra** | Terraform·클라우드 MCP (선택) | IaC·모니터링 MCP 사용 시 infra/TOOLS_AND_MCP.md에 추가. | [EXTENSIBILITY.md](EXTENSIBILITY.md) §2 |

정리하면:

- **Slack·Cursor**: Phase 2a까지 쓸 때만 Slack 연동 설정.
- **Figma·Stitch**: **design**, **frontend-web** 포지션에서 디자인→개발 파이프라인 쓸 때만 세팅.
- 그 외 MCP·도구: 해당 포지션의 **TOOLS_AND_MCP.md**에 적어 두고, 쓸 것만 Cursor(또는 팀 환경)에 설정.

---

## 5. 진행 순서 제안

1. **1단계**: Cursor 설치 → 실무 레포(GitHub/GitLab) 열기 → 보일러플레이트 해당 포지션 **플러그인 또는 복사** 적용.
2. **2단계**: Cursor에서 `/start`, `/plan-from-spec`, `/review-code` 등으로 **실전 A** 시작.
3. **3단계 (선택)**: Cursor Dashboard → Slack 연동 → 슬랙 채널에서 @Cursor로 동일 작업 실행.
4. **4단계 (선택, design/frontend-web)**: Figma MCP(·Stitch) 설정 → 디자인 링크로 규격 정리·코드 생성 요청.  
   **뭐부터?** → 위 **§2.1** 표 참고 (1 Cursor → 2 GitHub → 3 Slack → 4~ Figma·기타).

이 순서대로 하면 “바로 해보기”와 “포지션별 연동”을 필요한 만큼만 단계적으로 진행할 수 있습니다.
