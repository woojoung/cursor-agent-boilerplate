# Code Review

코드 리뷰를 실행한다. code-reviewer + security-reviewer 관점으로 순차 검토한다.

## 리뷰 체크리스트

| 항목 | 검사 내용 |
|------|----------|
| 아키텍처 | Clean Architecture, CQRS 준수 |
| 코드 품질 | 네이밍, 메서드 길이, 복잡도 |
| 테스트 | 존재 여부, 커버리지, 품질 |
| 보안 | OWASP Top 10, Spring Security |

## 실행 모드

- **폴더 경로**: 예) `src/main/java/com/example/auth` → 해당 경로 전체 리뷰.
- **MR 번호**: 예) MR-123 → 해당 MR 변경분 리뷰 (GitLab 연동 시).

## 출력 형식

- 항목별 Pass/이슈 개수 표로 정리.
- 이슈가 있으면 파일:줄, 내용, 개선 제안(한 줄)으로 정리한다.

예: "아키텍처 Pass, 코드 품질 2개 이슈, 테스트 Pass, 보안 Pass" 및 이슈 상세 목록.
