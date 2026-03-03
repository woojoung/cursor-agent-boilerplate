# 새 레포 세팅 (백엔드)

사용자가 **대화창에서 요청**하면, Java/Spring Boot 기준으로 새 프로젝트를 만들고 이 포지션의 `.cursor`·`AGENTS.md`를 복사한다. (쉘 스크립트 없이 에이전트가 터미널 명령을 실행.)

## 호출 방법

- 슬래시: `/setup-repo`
- 또는 대화: "백엔드 새 레포 세팅해줘", "Spring Boot 프로젝트 my-api로 만들어줘"

## 실행 절차

1. **입력 확인**
   - 프로젝트 이름(폴더명)을 사용자에게 요청. 예: `my-api`.
   - (선택) 대상 경로. 미지정 시 **현재 워크스페이스의 상위 디렉터리**에 생성 (예: `../my-api`).

2. **복사 소스 결정**
   - 현재 워크스페이스 루트에 `backend/` 폴더가 있으면 → 복사 소스는 `backend/`.
   - 없으면(이미 backend 폴더를 연 경우) → 복사 소스는 `.`(현재 디렉터리).

3. **터미널로 프로젝트 생성**
   - 아래를 **한 번에 실행**하거나, 순서대로 실행한다. `PROJECT_NAME`, `TARGET_DIR`은 사용자 입력으로 치환.
   - `TARGET_DIR` 기본값: `..` (워크스페이스 루트의 부모).
   ```bash
   cd <워크스페이스_루트>
   curl -sL -o /tmp/sb-setup.zip "https://start.spring.io/starter.zip?type=gradle-project&language=java&javaVersion=21&bootVersion=3.3.0&baseDir=PROJECT_NAME&groupId=com.example&artifactId=PROJECT_NAME&name=PROJECT_NAME&dependencies=web,data-jpa,validation"
   mkdir -p TARGET_DIR
   unzip -q -o /tmp/sb-setup.zip -d TARGET_DIR
   rm -f /tmp/sb-setup.zip
   ```

4. **.cursor·AGENTS.md 복사**
   - `SOURCE` = 2에서 정한 경로(backend 또는 .).
   ```bash
   cp -R SOURCE/.cursor TARGET_DIR/PROJECT_NAME/
   cp SOURCE/AGENTS.md TARGET_DIR/PROJECT_NAME/
   [ -d SOURCE/.cursor-plugin ] && cp -R SOURCE/.cursor-plugin TARGET_DIR/PROJECT_NAME/
   [ -f SOURCE/SETUP.md ] && cp SOURCE/SETUP.md TARGET_DIR/PROJECT_NAME/
   ```

5. **완료 안내**
   - "세팅이 완료되었습니다. **Cursor에서 File → Open Folder**로 아래 폴더를 열어주세요: `TARGET_DIR/PROJECT_NAME` (절대 경로로 안내)."

## 주의

- `PROJECT_NAME`에 공백·특수문자 없이 사용 (폴더명).
- 대상 경로는 절대 경로 또는 워크스페이스 기준 상대 경로로 실행.
