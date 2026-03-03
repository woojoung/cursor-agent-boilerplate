# 새 레포 세팅 (프론트엔드 모바일)

사용자가 **대화창에서 요청**하면, Flutter 기준으로 새 프로젝트를 만들고 이 포지션의 `.cursor`·`AGENTS.md`를 복사한다. (쉘 스크립트 없이 에이전트가 터미널 명령을 실행.)

## 호출 방법

- 슬래시: `/setup-repo`
- 또는 대화: "모바일 새 레포 세팅해줘", "Flutter 프로젝트 my_mobile 만들어줘"

## 실행 절차

1. **입력 확인**
   - 프로젝트 이름(폴더명)을 사용자에게 요청. 예: `my_mobile`.
   - (선택) 대상 경로. 미지정 시 **현재 워크스페이스의 상위 디렉터리**에 생성 (예: `../my_mobile`).

2. **복사 소스 결정**
   - 현재 워크스페이스 루트에 `frontend-mobile/` 폴더가 있으면 → 복사 소스는 `frontend-mobile/`.
   - 없으면(이미 frontend-mobile 폴더를 연 경우) → 복사 소스는 `.`(현재 디렉터리).

3. **터미널로 프로젝트 생성**
   - `flutter`가 PATH에 있는지 확인. 없으면 "Flutter CLI를 설치한 뒤 다시 요청해 주세요"라고 안내.
   - `PROJECT_NAME`, `TARGET_DIR`은 사용자 입력으로 치환. `TARGET_DIR` 기본값: `..`.
   ```bash
   cd <워크스페이스_루트>
   mkdir -p TARGET_DIR
   cd TARGET_DIR
   flutter create PROJECT_NAME --org com.example
   ```

4. **.cursor·AGENTS.md 복사**
   - `SOURCE` = 2에서 정한 경로(frontend-mobile 또는 .).
   ```bash
   cp -R SOURCE/.cursor TARGET_DIR/PROJECT_NAME/
   cp SOURCE/AGENTS.md TARGET_DIR/PROJECT_NAME/
   [ -d SOURCE/.cursor-plugin ] && cp -R SOURCE/.cursor-plugin TARGET_DIR/PROJECT_NAME/
   [ -f SOURCE/SETUP.md ] && cp SOURCE/SETUP.md TARGET_DIR/PROJECT_NAME/
   ```

5. **완료 안내**
   - "세팅이 완료되었습니다. **Cursor에서 File → Open Folder**로 아래 폴더를 열어주세요: `TARGET_DIR/PROJECT_NAME` (절대 경로로 안내)."

## 주의

- `PROJECT_NAME`에 공백 없이 사용 (Flutter는 스네이크 케이스 권장, 예: `my_app`).
- Flutter 미설치 시 3단계 전에 안내 후 중단.
