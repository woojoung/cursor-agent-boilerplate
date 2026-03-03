# 새 레포 세팅 (프론트엔드 웹)

사용자가 **대화창에서 요청**하면, Next.js 기준으로 새 프로젝트를 만들고 이 포지션의 `.cursor`·`AGENTS.md`를 복사한다. (쉘 스크립트 없이 에이전트가 터미널 명령을 실행.)

## 호출 방법

- 슬래시: `/setup-repo`
- 또는 대화: "프론트 웹 새 레포 세팅해줘", "Next.js 프로젝트 my-app으로 만들어줘"

## 실행 절차

1. **입력 확인**
   - 프로젝트 이름(폴더명)을 사용자에게 요청. 예: `my-app`.
   - (선택) 대상 경로. 미지정 시 **현재 워크스페이스의 상위 디렉터리**에 생성 (예: `../my-app`).

2. **복사 소스 결정**
   - 현재 워크스페이스 루트에 `frontend-web/` 폴더가 있으면 → 복사 소스는 `frontend-web/`.
   - 없으면(이미 frontend-web 폴더를 연 경우) → 복사 소스는 `.`(현재 디렉터리).

3. **터미널로 프로젝트 생성**
   - `PROJECT_NAME`, `TARGET_DIR`은 사용자 입력으로 치환. `TARGET_DIR` 기본값: `..`.
   ```bash
   cd <워크스페이스_루트>
   mkdir -p TARGET_DIR
   cd TARGET_DIR
   npx --yes create-next-app@latest PROJECT_NAME --ts --eslint --app --src-dir=false --tailwind --no-turbopack --import-alias "@/*"
   ```
   - (실패 시: 폴더만 만들고 4단계로 진행해 .cursor만 복사하고, "Next.js는 수동으로 `npx create-next-app@latest PROJECT_NAME` 실행 후 같은 폴더에 .cursor가 복사된 상태입니다"라고 안내.)

4. **.cursor·AGENTS.md 복사**
   - `SOURCE` = 2에서 정한 경로(frontend-web 또는 .).
   ```bash
   cp -R SOURCE/.cursor TARGET_DIR/PROJECT_NAME/
   cp SOURCE/AGENTS.md TARGET_DIR/PROJECT_NAME/
   [ -d SOURCE/.cursor-plugin ] && cp -R SOURCE/.cursor-plugin TARGET_DIR/PROJECT_NAME/
   [ -f SOURCE/SETUP.md ] && cp SOURCE/SETUP.md TARGET_DIR/PROJECT_NAME/
   ```

5. **완료 안내**
   - "세팅이 완료되었습니다. **Cursor에서 File → Open Folder**로 아래 폴더를 열어주세요: `TARGET_DIR/PROJECT_NAME` (절대 경로로 안내)."

## 주의

- `PROJECT_NAME`에 공백·특수문자 없이 사용.
- `npx create-next-app`이 대화형으로 바뀌면, 사용자에게 수동 생성 후 "같은 이름 폴더에 .cursor만 복사해줘"라고 요청할 수 있음.
