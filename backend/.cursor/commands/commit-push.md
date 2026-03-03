# Commit & Push

Stage된 파일만 커밋하고 원격에 푸시한다. **`git add -A` 사용 금지. 이미 staged 된 파일만 커밋한다.**

## 실행 절차

1. Git 상태 확인: 현재 브랜치, staged 파일, 미푸시 커밋.
2. 루트 브랜치(예: develop)이면 피처 브랜치 생성 여부 확인 → 생성 시 `feature/{gitlab_account}/{description}` 형식.
3. Commit: staged 파일이 있으면 커밋 메시지(인자 또는 자동 생성)로 commit. 없으면 변경사항 없음이면 중단.
4. Context updater: 변경된 AGENTS.md가 있으면 `git add "**/AGENTS.md"` 후 별도 커밋.
5. Push: `git push -u origin {브랜치명}`.

## 사용 시나리오

- MR 생성 전: `/mr-gitlab` 내부에서 호출.
- MR 생성 후 추가 수정: 리뷰 피드백 반영 후 직접 호출.
- 단순 커밋+푸시만 필요할 때.

## 커밋 메시지 컨벤션

`{type}: {subject}` — feat, fix, refactor, test, docs, chore.

## 주의

- 루트 브랜치에 직접 커밋하지 않는다. 민감 파일(.env, credentials.json 등)은 커밋하지 않는다.
