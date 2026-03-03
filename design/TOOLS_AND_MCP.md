# 디자인 — 추천 도구·MCP

> 이 포지션에서 **실무 레포**나 Cursor에 추가할 수 있는 **도구·MCP** 예시입니다.  
> 디자이너 에이전트가 만든 UI를 **Figma**에 두고, **Figma MCP·Stitch**로 프론트엔드 개발에 전달하는 흐름을 확장할 수 있습니다. (전체 가이드: [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md))

---

## 0. 보안·개인정보 (토큰/키는 레포에 올리지 않기)

**Figma Personal Access Token, Stitch API 키 등 개인·팀 자격정보는 GitHub에 커밋하지 마세요.**

| 레포에 넣는 것 | 로컬에서만 쓰는 것 (절대 커밋 금지) |
|----------------|-------------------------------------|
| **예시 파일** (`.env.example`, 설정 구조만 있는 예시) | **실제 토큰·API 키** (`.env`, Cursor MCP 설정의 비밀값) |
| **안내 문서** (어디서 토큰 발급받는지, 어디에 넣는지) | Figma PAT, Stitch 키 등 |

- **실제 값**은 `.env`에 두고, `.env`는 이미 `.gitignore`에 포함되어 있음.
- **예시**: 이 폴더의 `.env.example`을 복사해 `.env`로 만들고, placeholder를 본인/팀 토큰으로 **로컬에서만** 채워 사용하세요.
- Cursor MCP 설정에서 토큰을 넣을 때도 **환경 변수**로 참조하도록 하면, 설정 파일 자체에는 비밀값이 들어가지 않습니다.

---

## 1. Figma MCP

**용도**: Figma 디자인(레이어·변수·스타일·오토 레이아웃)을 Cursor에 전달. **design** 포지션에서는 디자인 검토·토큰 정리, **frontend-web** 포지션에서는 동일 MCP로 디자인→코드 생성에 사용.

**연동 요약**:

1. Figma에서 [Personal Access Token 발급](https://www.figma.com/developers/api#access-tokens) (Figma 설정 → Account → Personal access tokens). **이 토큰은 레포에 올리지 말고** 로컬 `.env`에만 넣으세요.
2. Cursor **MCP 설정**에 Figma MCP 서버 추가. 토큰은 환경 변수(예: `FIGMA_PERSONAL_ACCESS_TOKEN`)로 참조하도록 설정.
3. 디자인 파일/노드 링크를 Cursor에 붙여 “이 디자인 기준으로 컴포넌트 규격 정리해줘” 또는 (프론트엔드에서) “이 디자인으로 코드 생성해줘” 요청.

**Figma 권장**: Variables(타이포·간격·색·radius), Auto Layout 전반 사용 시 MCP가 구조를 잘 인식합니다.

**참고**: [Figma to Cursor MCP](https://cursorideguide.com/use-cases/figma-to-cursor-with-mcp) · [Figma MCP Server](https://builder.io/blog/figma-mcp-server)

---

## 2. Stitch (디자인 → 코드)

**용도**: Figma와 연동해 디자인을 코드로 변환. **디자이너 에이전트**가 정리한 UI를 Figma에 반영한 뒤, Stitch로 내보내거나 Figma MCP와 함께 **frontend-web**에서 동일 소스를 사용.

**워크플로우**:

- design: Figma에서 UI·디자인 토큰 정리 → Stitch/내보내기 설정.
- frontend-web: Figma MCP 또는 Stitch 연동으로 Cursor에서 해당 디자인 참조 후 컴포넌트 구현.

팀에서 Stitch를 쓰면 이 파일에 **설치·환경·Cursor 연동** 단계를 추가해 두면 됩니다. Stitch API 키 등이 필요하면 `.env.example`에 항목만 추가하고, **실제 키는 .env에만 넣고 커밋하지 마세요.**

---

## 3. 디자인 토큰·브랜드 가이드

- 디자인 시스템 **토큰**(색·간격·타이포)을 Variables로 관리하고, MCP·내보내기로 개발에 전달.
- **design** 포지션의 `.cursor/rules`(tokens.mdc, design-system.mdc)와 이 파일을 함께 두면, 새 도구·MCP가 나와도 “토큰·컴포넌트 규격” 기준은 유지하면서 연동만 확장할 수 있습니다.

---

## 4. 확장 시

- 새 **MCP**(예: 디자인 시스템 전용 MCP)나 **도구**를 쓰면 이 파일에 **이름·용도·연동 한 줄·링크** 추가.
- design ↔ frontend-web 간 **핸드오프 절차**가 바뀌면 [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md)와 frontend-web의 **TOOLS_AND_MCP.md**를 함께 갱신.
