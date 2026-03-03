# runtime — 실전 B(Phase 2~3) 실행용 코드

이 폴더는 **실전 B**(슬랙 에이전트 협업)를 위해 이 레포 밖에서 돌리는 **런타임·오케스트레이터**용 코드를 둘 때 사용합니다.

- **Phase 2a**: 별도 봇 코드 없이 **Slack + Cursor 연동**으로 진행. 슬랙 대화창에서 @Cursor 호출 + (필요 시) [AGENT_MESSAGE_PROTOCOL.md](../docs/AGENT_MESSAGE_PROTOCOL.md) §6 템플릿 복사. → [docs/PHASE2A_CURSOR_SLACK.md](../docs/PHASE2A_CURSOR_SLACK.md)
- **Phase 2b 이후**: 오케스트레이터·에이전트 런타임·메모리·GitHub/CI 연동을 별도 서비스로 구축할 때, 이 레포의 `docs/`·포지션 폴더를 참조합니다.
