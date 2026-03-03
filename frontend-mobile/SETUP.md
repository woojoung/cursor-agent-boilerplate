# 프론트엔드(모바일) 기본 레포 세팅

> 이 포지션은 **Flutter / Dart** 기준입니다.  
> **에이전트 대화창**: 이 폴더(frontend-mobile)를 Cursor로 연 뒤 채팅에서 **`/setup-repo`** 를 실행하거나 "모바일 새 레포 세팅해줘"라고 요청하면, 프로젝트명만 입력해 자동으로 생성·복사됩니다.  
> 수동 절차·검증 체크리스트: **[docs/SETUP_REPOS.md](../docs/SETUP_REPOS.md)**.

## 1. 프로젝트 생성

```bash
flutter create my_mobile_app
cd my_mobile_app
```

## 2. .cursor + AGENTS.md 적용

이 보일러플레이트의 **frontend-mobile/** 폴더에서 아래를 **새 레포 루트**로 복사합니다.

- `.cursor/` (rules, commands, skills, agents 전부)
- `AGENTS.md`
- (선택) `.cursor-plugin/`

## 3. Cursor에서 열기

새 레포 폴더를 Cursor에서 Open 하면 Flutter용 규칙·명령이 적용됩니다.
