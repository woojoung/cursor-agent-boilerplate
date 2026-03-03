# AGENTS.md

> cursor-agent-boilerplate 루트 레벨 에이전트 공통 맥락 문서입니다.

## 프로젝트 개요

이 레포는 **Cursor IDE용 에이전트 보일러플레이트**입니다. 소스 코드·빌드 시스템·런타임 서비스가 없는 **문서/설정 전용 레포**이며, 7개 포지션(backend, frontend-web, frontend-mobile, qa, infra, pm, design) 폴더에 `.cursor/rules`, `.cursor/commands`, `.cursor/skills`, `.cursor/agents`, `AGENTS.md`가 각각 정의되어 있습니다.

## Cursor Cloud specific instructions

- **의존성 없음**: 이 레포에는 `package.json`, `requirements.txt`, `Makefile`, `Dockerfile` 등 의존성/빌드 파일이 없습니다. 별도의 의존성 설치나 빌드 단계가 필요 없습니다.
- **실행할 서비스 없음**: 런타임 서비스, 데이터베이스, 서버가 없습니다. `runtime/` 폴더는 Phase 2b+ 로드맵용 빈 placeholder입니다.
- **파일 유형**: 전체 파일은 Markdown(`.md`), Cursor 규칙(`.mdc`), JSON(`.json`), SVG, `.gitignore`, `.env.example`로만 구성됩니다.
- **검증 방법**: 보일러플레이트 무결성 검증은 (1) 모든 `.mdc` 파일이 비어있지 않은지, (2) 모든 `plugin.json`/`marketplace.json`이 유효한 JSON인지, (3) 7개 포지션 폴더에 `.cursor/{rules,commands,skills,agents}` 구조가 올바른지 확인하면 됩니다.
- **사용법**: 이 보일러플레이트는 실무 레포에 플러그인 또는 복사 방식으로 적용합니다. 상세: `README.md` §2, `docs/GETTING_STARTED_CHECKLIST.md`.
