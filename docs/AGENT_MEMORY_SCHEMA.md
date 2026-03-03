# 에이전트 메모리 스키마 (초안)

> 슬랙 에이전트 런타임에서 **역할별 컨텍스트**를 저장할 때 참고하는 메모리 스키마·용도 초안입니다.  
> 벡터 DB 또는 키-값 저장소 구현 시 아래 필드를 기준으로 확장할 수 있습니다.

---

## 1. 저장 단위

- **스레드 단위**: Slack 스레드(또는 작업 단위)별로 한 레코드. `thread_id` 또는 `task_id`를 키로 사용.
- **역할별**: 동일 스레드라도 역할(po, design, frontend-web, backend, qa)별로 별도 컨텍스트를 둘 수 있음(선택).

---

## 2. 역할별 저장 권장 필드 (초안)

| 필드 | 용도 | 예시 |
|------|------|------|
| **thread_id** | 슬랙 스레드(또는 대화) 식별자 | `C01234:1234567890.123456` |
| **role** | 포지션 | `pm`, `design`, `frontend-web`, `backend`, `qa` |
| **recent_messages** | 최근 N개 메시지 요약 또는 원문(토큰 제한 내) | 사용자·에이전트 주고받은 내용 요약 |
| **spec_id** | 현재 스레드에서 다루는 스펙/이슈 ID | `SPEC-001`, `#42` |
| **spec_link** | 스펙 문서 또는 이슈 URL | `https://...` |
| **task_id** | 구현 태스크/이슈 ID (FE/BE/QA) | `TASK-01` |
| **pr_url** | 리뷰 대상 PR URL (QA) | `https://github.com/.../pull/10` |
| **last_event** | 마지막 발생 이벤트 타입 | `spec_completed`, `pr_opened` |
| **updated_at** | 마지막 갱신 시각 (ISO 8601) | `2025-03-03T12:00:00Z` |

---

## 3. 용도

- **같은 스레드 내 후속 질문**: `recent_messages`로 맥락 유지.  
- **다음 역할로 넘길 때**: `spec_id`, `spec_link`, `task_id`, `pr_url`을 메시지 프로토콜 페이로드와 함께 전달.  
- **오케스트레이터 상태와 연동**: `last_event`, `task_id`로 상태 전이(TODO/IN_PROGRESS/DONE)와 일치시킬 수 있음.

---

## 4. 참고

- 상세 이벤트·페이로드: [AGENT_MESSAGE_PROTOCOL.md](AGENT_MESSAGE_PROTOCOL.md).  
- 비전·아키텍처: [SILO_AGENT_VISION.md](SILO_AGENT_VISION.md) §4.2.
