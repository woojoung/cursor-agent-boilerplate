# Phase 2a: Slack + Cursor 연동으로 할 일 전달

> **목적**: Phase 2a는 **별도 봇 코드 없이** Slack과 Cursor 연동만으로 진행할 수 있음을 정리합니다.  
> Slack **대화창**에서 **@Cursor** 를 멘션해 지시하면 Cursor **Cloud(Background) Agent**가 실행되고, 결과가 슬랙 스레드로 돌아옵니다.  
> 참고: [GeekNews — Cursor, Slack과 연동하여 백그라운드 에이전트 지원](https://news.hada.io/topic?id=21583), [Cursor Docs — Cloud Agents](https://cursor.com/docs/background-agent)

---

## 1. 왜 별도 JS 봇이 필요 없은가

- **Slack ↔ Cursor 연동**이 되면, **Slack 대화창**에서 `@Cursor` + 지시문만 입력하면 Cursor Cloud Agent가 원격에서 작업하고 결과를 슬랙으로 보냅니다.
- "할 일" 내용은 **사람이 슬랙에 적거나**, 필요 시 [AGENT_MESSAGE_PROTOCOL.md](AGENT_MESSAGE_PROTOCOL.md) §6의 **메시지 템플릿**을 복사해 붙여넣은 뒤 같은 스레드에서 **@Cursor** 를 호출하면 됩니다.
- 따라서 **포맷된 메시지를 자동으로 올려 주는 전용 봇(JS 등)** 은 이 레포에서는 두지 않습니다. Phase 2b에서 오케스트레이터를 만들 때, 그쪽에서 슬랙 포스팅이 필요하면 그때 구현하면 됩니다.

---

## 2. Cursor Slack(Cloud Agent) 연동 요약

- Slack 채널/스레드에서 **@Cursor** 멘션 + 지시문 입력 → Cursor **Cloud Agent** 실행.
- 원격(우분투 기반) 환경에서 GitHub/GitLab 레포 클론·작업 후 결과를 슬랙으로 전송.
- 예시: `@Cursor [branch=dev, model=o3, repo=owner/repo, autopr=false] Fix the login bug`
- 스레드 맥락 인식 가능: `@Cursor fix this` 처럼 짧게만 써도 됨.

### 2.1 설정 방법

1. [Cursor Dashboard](https://www.cursor.com/dashboard) → **Integrations** 에서 Slack 연결.
2. Slack 워크스페이스에 **Cursor 앱** 설치 및 권한 승인.
3. (레거시) Privacy Mode 비활성화 필요할 수 있음 — Cloud Agent는 Cursor 서버에서 코드를 처리합니다.

### 2.2 우리 프로토콜과 함께 쓰는 방법

[AGENT_MESSAGE_PROTOCOL.md](AGENT_MESSAGE_PROTOCOL.md) §6의 **이벤트 → 슬랙 메시지 템플릿**을 참고해, 슬랙에 "할 일"을 적고 같은 스레드에서 **@Cursor** 로 지시하면 됩니다.

- **스펙 완료(spec_completed) 후 디자이너 채널**  
  (필요 시 §6 템플릿을 복사해 슬랙에 붙여넣거나, 요약만 적고)  
  → `@Cursor [repo=org/design-repo, branch=main] 이 스펙 기준으로 유저 플로우·와이어프레임 초안 작성해줘. 스펙: <링크>`

- **PR 리뷰(pr_opened) 후 QA 채널**  
  → `@Cursor [repo=org/backend, branch=feature/xyz] 이 PR 리뷰해줘. 테스트 시나리오 확인하고 머지 승인/차단 의견 남겨줘.`

**정리**: 슬랙 대화창에서 @Cursor 만 잘 쓰면 되고, 별도 봇 코드는 필요 없습니다.
