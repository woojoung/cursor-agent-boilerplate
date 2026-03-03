# 모델 배정 원칙 — 효과적 동작과 비용 최소화

> **목적**: "판단엔 Opus, 실행엔 Sonnet, 탐색엔 Haiku" 같은 **역할별 모델 배정** 원칙을 정리하고, Cursor에서 포지션·서브에이전트별로 어떻게 적용할 수 있는지 안내합니다.  
> Cursor는 [서브에이전트](https://cursor.com/ko/docs/context/subagents)에서 `model` 필드로 `fast`, `inherit` 또는 특정 모델 ID를 지정할 수 있습니다.

---

## 1. 전문가들이 말하는 모델 배정 원칙

| 용도 | 권장 모델 | 이유 |
|------|-----------|------|
| **판단·의사결정** (계획 수립, 리스크 분석, 아키텍처 결정, 최종 검토) | Opus(또는 최상위 모델) | 복잡한 추론·트레이드오프·일관성 필요. 잘못된 판단 비용이 큼. |
| **실행** (구현, 리뷰, 테스트 작성·실행, MR 초안) | Sonnet(또는 중간 모델) | 품질과 속도·비용 균형. 반복 작업이 많아 비용 민감. |
| **탐색·경량 작업** (코드베이스 검색, 맥락 문서 갱신, 포맷·정리) | Haiku(또는 빠른/경량 모델) | 단순·반복 작업. 토큰 사용 많고 결과는 요약만 필요. |

이 원칙대로 쓰면 **효과는 유지하면서 비용을 줄일 수 있습니다**.

---

## 2. Cursor에서 쓸 수 있는 옵션

### 2.1 메인 에이전트(대화창) 모델

- 사용자가 **Cursor 모델 선택기**에서 직접 선택합니다.  
- Opus, Sonnet, Haiku(및 Cursor/Composer 제공 모델) 등이 있으면, **판단이 많이 필요한 대화**에서는 상위 모델(Opus), **일상 구현·리뷰**에서는 Sonnet을 쓰는 식으로 조정할 수 있습니다.  
- Cursor 제품에 따라 제공 모델 이름·등급이 다를 수 있으므로, 설정 화면 기준으로 "가장 강한/중간/가장 빠른"에 위 표를 대응시키면 됩니다.

### 2.2 서브에이전트(Subagent) `model` 필드

[Cursor 서브에이전트 문서](https://cursor.com/ko/docs/context/subagents)에 따르면, 각 서브에이전트 마크다운 파일의 YAML 프론트매터에서:

| 값 | 의미 |
|----|------|
| `model: inherit` | **메인 에이전트와 동일한 모델** 사용. 기본값. 판단·실행 모두 메인과 동일 품질이 필요할 때. |
| `model: fast` | **더 빠르고 저렴한 모델** 사용. 탐색·경량 작업·검증에 적합. |
| (특정 모델 ID) | 해당 모델 사용. 제품에서 지원하고 Task 도구가 허용하면 지정 가능. (일부 환경에서는 `fast`만 유효하다는 버그 리포트 있음.) |

**주의**:  
- **레거시 요청 기준 요금제**에서는 Max Mode를 켜야 서브에이전트 `model` 설정이 적용됩니다.  
- **사용량 기준 요금제**에서는 설정한 모델이 기본 적용됩니다.

---

## 3. 포지션·역할별 모델 배정 가이드

아래는 **원칙**을 Cursor 서브에이전트 `model` 값으로 옮긴 권장안입니다.  
실제로는 `inherit` / `fast` 두 가지가 안정적으로 동작하므로, "판단형 = inherit", "탐색/경량 = fast"로 나눕니다.

| 포지션 | 서브에이전트(예) | 권장 model | 이유 |
|--------|------------------|------------|------|
| **공통** | planner, spec-writer, risk-analyzer, design-reviewer | `inherit` | 판단·계획·검토 → 메인과 동일 수준 모델. |
| **공통** | code-reviewer, security-reviewer, test-runner | `inherit` | 실행 품질 중요. 메인 선택(Opus/Sonnet) 따름. |
| **공통** | context-updater, (탐색·문서 갱신 전용) | `fast` | 탐색·정리 위주. 비용 절감. |
| **공통** | verifier (완료 검증만) | `fast` | Cursor 문서 예시처럼 검증·테스트 실행 위주면 fast로 충분. |
| **백엔드** | planner, code-reviewer, security-reviewer | `inherit` | 설계·보안 판단 중요. |
| **백엔드** | build-fixer, test-writer | `inherit` | 구현·테스트 품질. |
| **백엔드** | context-updater | `fast` | 이미 `fast` 사용 중. 유지. |
| **PM** | spec-writer, task-planner, risk-analyzer | `inherit` | 스펙·태스크·리스크 판단. |
| **QA** | scenario-writer, test-planner, bug-analyzer | `inherit` | 시나리오·회귀 범위 판단. |
| **디자인** | design-reviewer, component-spec-writer, token-validator | `inherit` | 일관성·가이드라인 판단. |
| **인프라** | planner, infra-reviewer, runbook-writer | `inherit` | 설계·보안·운영 판단. |

---

## 4. 이 레포에서 이미 적용된 부분

- **context-updater** (backend): `model: fast` — 맥락 문서 갱신은 경량 작업.  
- **그 외 대부분 서브에이전트**: `model: inherit` — 판단·실행 품질 유지.

원칙에 맞춰 **검증 전용(verifier)** 이나 **순수 탐색/정리** 역할을 추가할 때는 `model: fast`를 두고, **계획·리뷰·보안·스펙** 관련은 `inherit`을 유지하면 효과와 비용 균형이 좋습니다.

---

## 5. 비용 최소화 요약

1. **메인 에이전트**: 할 일이 "계획·의사결정·복잡 리뷰"면 상위 모델(Opus), "일상 구현·작은 수정"이면 중간 모델(Sonnet) 선택.  
2. **서브에이전트**:  
   - 판단·실행 → `model: inherit`  
   - 탐색·문서 갱신·단순 검증 → `model: fast`  
3. **Cursor에 Opus/Sonnet/Haiku가 다 있다면**: 메인을 용도에 맞게 바꾸고, 서브에이전트는 위처럼 inherit/fast로 나누면 동일 원칙을 지키면서 비용을 줄일 수 있습니다.
