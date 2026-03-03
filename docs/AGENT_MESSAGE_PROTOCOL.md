# 에이전트 간 메시지 프로토콜

> 이 문서는 **AI 사일로 조직**에서 역할별 에이전트·오케스트레이터·슬랙 봇이 주고받는 **구조화 메시지(이벤트)** 의 계약입니다.  
> 오케스트레이터·에이전트 런타임 구현 시 이 문서를 단일 소스로 참조합니다.  
> 핸드오프 포맷은 [INTEGRATION.md §4.2](INTEGRATION.md)를 함께 참고하세요.

---

## 1. 이벤트 타입 및 역할 매핑

| 이벤트 | 발신 역할 | 수신 역할 | 용도 |
|--------|-----------|-----------|------|
| `spec_completed` | PO (pm) | Designer, FE, BE, QA | 스펙 확정 시 다음 포지션에 전달 |
| `design_completed` | Designer (design) | FE (frontend-web, frontend-mobile) | 디자인 스펙/와이어프레임 완료 시 FE에 전달 |
| `task_ready_for_impl` | PO (pm) | FE, BE | 구현용 태스크 할당·브랜치 정보 전달 |
| `pr_opened` | FE, BE | QA (qa) | PR 생성 시 QA 리뷰·테스트 요청 |
| `qa_approved` | QA (qa) | FE, BE, Orchestrator | 테스트·리뷰 통과, 머지 허용 |
| `qa_blocked` | QA (qa) | FE, BE, Orchestrator | 테스트 실패 또는 리뷰 미통과, 머지 차단 |
| `deploy_done` | CI/CD | Orchestrator, Slack | 배포 성공 알림 |
| `deploy_failed` | CI/CD | Orchestrator, Slack | 배포 실패·롤백 알림 |
| `blocked_escalation` | any | PO (pm) | 블로커 발생 시 PO 에스컬레이션 |

---

## 2. 페이로드 JSON 스키마 (예시)

구현 시 아래 필드는 **필수(required)** 로 두고, 확장 필드는 팀 정책에 따라 추가합니다.

### 2.1 spec_completed

**발신**: PO → Designer, FE, BE, QA

```json
{
  "event": "spec_completed",
  "spec_id": "string (이슈 ID 또는 문서 식별자)",
  "spec_link": "string (URL: spec.md 또는 이슈 링크)",
  "goal_one_liner": "string (목표 한 줄 요약)",
  "ac_list": ["string (수용 조건 1)", "string (수용 조건 2)", "..."],
  "priority": "P0 | P1 | P2",
  "created_at": "string (ISO 8601)"
}
```

### 2.2 design_completed

**발신**: Designer → FE

```json
{
  "event": "design_completed",
  "design_spec_link": "string (규격 문서 URL)",
  "figma_link": "string (선택, Figma URL)",
  "tokens": "string 또는 object (디자인 토큰 요약/경로)",
  "variants": ["string (변형 목록)"],
  "created_at": "string (ISO 8601)"
}
```

### 2.3 task_ready_for_impl

**발신**: PO → FE, BE

```json
{
  "event": "task_ready_for_impl",
  "task_id": "string (이슈/태스크 ID)",
  "spec_ref": "string (스펙 링크 또는 spec_id)",
  "ac_ref": ["string (관련 AC)"],
  "branch_name": "string (생성된 브랜치명 또는 권장 브랜치명)",
  "created_at": "string (ISO 8601)"
}
```

### 2.4 pr_opened

**발신**: FE 또는 BE → QA

```json
{
  "event": "pr_opened",
  "pr_url": "string (PR URL)",
  "branch": "string (브랜치명)",
  "change_summary": "string (변경 요약)",
  "regression_hint": "string (회귀 테스트 시 확인할 영역)",
  "author_role": "frontend-web | frontend-mobile | backend",
  "created_at": "string (ISO 8601)"
}
```

### 2.5 qa_approved / qa_blocked

**발신**: QA → FE, BE, Orchestrator

```json
{
  "event": "qa_approved",
  "pr_url": "string (PR URL)",
  "approved": true,
  "reason": "string (선택, 통과 사유 요약)",
  "created_at": "string (ISO 8601)"
}
```

```json
{
  "event": "qa_blocked",
  "pr_url": "string (PR URL)",
  "approved": false,
  "reason": "string (실패 사유: 테스트 실패, 리뷰 코멘트 등)",
  "created_at": "string (ISO 8601)"
}
```

### 2.6 deploy_done / deploy_failed

**발신**: CI/CD → Orchestrator, Slack

```json
{
  "event": "deploy_done",
  "env": "string (예: staging, production)",
  "success": true,
  "rollback_done": false,
  "created_at": "string (ISO 8601)"
}
```

```json
{
  "event": "deploy_failed",
  "env": "string",
  "success": false,
  "rollback_done": "boolean (롤백 수행 여부)",
  "message": "string (선택, 에러 요약)",
  "created_at": "string (ISO 8601)"
}
```

### 2.7 blocked_escalation

**발신**: any → PO

```json
{
  "event": "blocked_escalation",
  "task_id": "string (블로커가 걸린 태스크/이슈 ID)",
  "blocker_reason": "string (블로커 사유)",
  "from_role": "pm | design | frontend-web | frontend-mobile | backend | qa | infra",
  "created_at": "string (ISO 8601)"
}
```

---

## 3. 역할 ↔ 포지션 폴더 매핑

오케스트레이터가 이벤트 수신 시 **어느 포지션 에이전트를 호출할지** 참고용입니다.

| 역할 (비전 명칭) | 포지션 폴더 | 채널 매핑은 §4 참고 |
|------------------|-------------|----------------------|
| FRIDAY (PO) | `pm/` | #silo-*-po |
| Black Widow (Designer) | `design/` | #silo-*-design |
| Spider-Man (FE) | `frontend-web/`, `frontend-mobile/` | #silo-*-fe |
| Hulk (BE) | `backend/` | #silo-*-be |
| Hawkeye (QA) | `qa/` | #silo-*-qa |
| (선택) 인프라 | `infra/` | #silo-*-infra |

---

## 4. 역할–채널 매핑 (예시)

팀/사일로별로 Slack 채널과 포지션을 매핑합니다. 오케스트레이터 설정의 단일 소스로 사용할 수 있습니다.

| Slack 채널 (예시) | 포지션 | 비전 역할 |
|-------------------|--------|-----------|
| `#silo-a-po` | pm | FRIDAY (PO) |
| `#silo-a-design` | design | Black Widow (Designer) |
| `#silo-a-fe` | frontend-web 또는 frontend-mobile | Spider-Man (FE) |
| `#silo-a-be` | backend | Hulk (BE) |
| `#silo-a-qa` | qa | Hawkeye (QA) |
| `#silo-a-infra` | infra | (선택) 인프라 |
| `#silo-a-boss` | — | 인간: 전략·최종 승인 |

다중 사일로 확장 시: `#silo-b-po`, `#silo-b-fe` 등으로 동일 패턴 적용.

---

## 5. 상태 전이 규칙 (오케스트레이터 참조)

작업(또는 이슈) 단위 상태: `TODO` → `IN_PROGRESS` → `BLOCKED`(선택) → `DONE`.  
오케스트레이터는 아래 전이 시 **해당 이벤트**를 발생시키고, §1·§2의 수신 역할/채널에 전파합니다.

| 전이 조건 | 발생 이벤트 | 전파 대상 |
|-----------|-------------|-----------|
| 스펙 확정(PO 완료) | `spec_completed` | Designer, FE, BE, QA 채널 |
| 디자인 스펙/와이어프레임 완료 | `design_completed` | FE 채널 |
| 구현용 태스크·브랜치 할당 | `task_ready_for_impl` | FE, BE 채널 |
| PR 생성됨 | `pr_opened` | QA 채널 |
| QA 테스트·리뷰 통과 | `qa_approved` | FE/BE 채널, Orchestrator (머지 허용) |
| QA 테스트 실패 또는 리뷰 미통과 | `qa_blocked` | FE/BE 채널, Orchestrator (머지 차단) |
| 배포 성공 | `deploy_done` | Orchestrator, Slack 알림 |
| 배포 실패 | `deploy_failed` | Orchestrator, Slack 알림 (롤백 후 동일) |
| 블로커 발생(어떤 역할이든) | `blocked_escalation` | PO 채널 |

---

## 6. 이벤트 → 슬랙 메시지 템플릿 (Phase 2a)

이벤트 발생 시 슬랙에 올릴 **메시지 템플릿** 예시. 봇/오케스트레이터는 아래를 참고해 해당 채널에 포스팅합니다. 추가 예시는 [INTEGRATION.md](INTEGRATION.md) §5.4 참고.

### 6.1 spec_completed → Design / FE / BE / QA 채널

**대상 채널**: #silo-a-design, #silo-a-fe, #silo-a-be, #silo-a-qa (역할–채널 매핑에 따름)

**텍스트 예시**:
```
📋 스펙 확정됨 — 다음 단계 진행해 주세요.

• 목표: {{goal_one_liner}}
• 스펙 링크: {{spec_link}}
• 수용 조건(AC): {{ac_list}}
• 우선순위: {{priority}}

Cursor에서 이 스펙 기준으로 진행할 때 붙여넣기: "이 스펙 기준으로 [역할에 맞는 작업] 해줘. 스펙: {{spec_link}}"
```

**Design 채널용 추가 문구**: "디자인: 유저 플로우·와이어프레임 또는 컴포넌트 규격 초안을 진행해 주세요."  
**FE/BE 채널용 추가 문구**: "구현: plan·task 분해 후 해당 스펙 링크를 기준으로 개발해 주세요."  
**QA 채널용 추가 문구**: "QA: 이 스펙 기준으로 테스트 시나리오 도출해 주세요."

### 6.2 pr_opened → QA 채널

**대상 채널**: #silo-a-qa

**텍스트 예시**:
```
🔀 PR 리뷰 요청

• PR: {{pr_url}}
• 브랜치: {{branch}}
• 변경 요약: {{change_summary}}
• 회귀 시 확인할 영역: {{regression_hint}}

Cursor에서: "이 PR 리뷰해줘. 테스트 시나리오 확인하고 머지 승인/차단 의견 남겨줘."
```
