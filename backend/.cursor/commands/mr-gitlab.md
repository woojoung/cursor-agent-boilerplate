# MR GitLab

Git 상태를 확인하고, 커밋·푸시·GitLab MR 생성을 진행한다. 리뷰 결과를 MR 설명에 포함한다. **`git add -A` 사용 금지. 이미 staged 된 파일만 커밋한다.**

## 실행 절차

1. **Commit & Push**: `/commit-push`와 동일하게 진행 (Git 상태 확인 → 필요 시 피처 브랜치 생성 → Stage된 것만 Commit → context-updater로 AGENTS.md 반영 후 커밋 → Push).
2. **Code Review**: 변경 파일 대상으로 코드 리뷰 실행 (아키텍처·코드 품질·테스트·보안). 리뷰 결과를 요약한다.
3. **MR 생성**: GitLab MCP 또는 사용자 안내로 MR 생성. MR 설명에 리뷰 요약을 포함한다.
4. **복귀**: 작업 후 루트 브랜치(예: develop)로 checkout 한다.

## Git 상태별 분기

- 변경사항 없음 → 중단.
- 변경사항 있음 → push 완료 후 code review → MR 생성.

## MR 템플릿

`.gitlab/merge_request_templates/default.md`가 있으면 그 템플릿을 기반으로 작성. 없으면 기본 템플릿 사용.

## 주의사항

- git add -A 사용 금지. AGENTS.md는 별도 스테이징 후 커밋. 루트 브랜치에서 실행 시 새 피처 브랜치 생성. 민감 파일 제외. MR 생성 후 루트 브랜치로 복귀.
