# 백엔드 기본 레포 세팅

> 이 포지션은 **Java / Spring Boot** 기준입니다.  
> **에이전트 대화창**: 이 폴더(backend)를 Cursor로 연 뒤 채팅에서 **`/setup-repo`** 를 실행하거나 "백엔드 새 레포 세팅해줘"라고 요청하면, 프로젝트명만 입력해 자동으로 생성·복사됩니다.  
> 수동 절차·검증 체크리스트: **[docs/SETUP_REPOS.md](../docs/SETUP_REPOS.md)**.

## 1. 프로젝트 생성

- [Spring Initializr](https://start.spring.io/): Java 21, Spring Boot 3.x, Gradle, Web·JPA·Validation 등 선택 후 다운로드·압축 해제.
- 또는:  
  `curl https://start.spring.io/starter.zip -d type=gradle-project -d language=java -d javaVersion=21 -d bootVersion=3.3 -d name=my-backend -o backend.zip && unzip backend.zip -d my-backend && cd my-backend`

## 2. .cursor + AGENTS.md 적용

이 보일러플레이트의 **backend/** 폴더에서 아래를 **새 레포 루트**로 복사합니다.

- `.cursor/` (rules, commands, skills, agents 전부)
- `AGENTS.md`
- (선택) `.cursor-plugin/`

## 3. Cursor에서 열기

새 레포 폴더를 Cursor에서 Open 하면 Java/Spring Boot용 규칙·명령이 적용됩니다.
