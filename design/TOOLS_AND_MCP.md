# 디자인 — 추천 도구·MCP

> 이 포지션에서 **실무 레포**나 Cursor에 추가할 수 있는 **도구·MCP** 예시입니다.  
> 디자이너 에이전트가 만든 UI를 **Figma**에 두고, **Figma MCP·Stitch**로 프론트엔드 개발에 전달하는 흐름을 확장할 수 있습니다. (전체 가이드: [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md))

---

## 1. Figma MCP

**용도**: Figma 디자인(레이어·변수·스타일·오토 레이아웃)을 Cursor에 전달. **design** 포지션에서는 디자인 검토·토큰 정리, **frontend-web** 포지션에서는 동일 MCP로 디자인→코드 생성에 사용.

**연동 요약**:

- Figma **Personal Access Token** 발급.
- Cursor **MCP 설정**에 Figma MCP 서버 추가.
- 디자인 파일/노드 링크를 Cursor에 붙여 “이 디자인 기준으로 컴포넌트 규격 정리해줘” 또는 (프론트엔드에서) “이 디자인으로 코드 생성해줘” 요청.

**Figma 권장**: Variables(타이포·간격·색·radius), Auto Layout 전반 사용 시 MCP가 구조를 잘 인식합니다.

**참고**: [Figma to Cursor MCP](https://cursorideguide.com/use-cases/figma-to-cursor-with-mcp) · [Figma MCP Server](https://builder.io/blog/figma-mcp-server)

---

## 2. Stitch (디자인 → 코드)

**용도**: Figma와 연동해 디자인을 코드로 변환. **디자이너 에이전트**가 정리한 UI를 Figma에 반영한 뒤, Stitch로 내보내거나 Figma MCP와 함께 **frontend-web**에서 동일 소스를 사용.

**워크플로우**:

- design: Figma에서 UI·디자인 토큰 정리 → Stitch/내보내기 설정.
- frontend-web: Figma MCP 또는 Stitch 연동으로 Cursor에서 해당 디자인 참조 후 컴포넌트 구현.

팀에서 Stitch를 쓰면 이 파일에 **설치·환경·Cursor 연동** 단계를 추가해 두면 됩니다.

---

## 3. 디자인 토큰·브랜드 가이드

- 디자인 시스템 **토큰**(색·간격·타이포)을 Variables로 관리하고, MCP·내보내기로 개발에 전달.
- **design** 포지션의 `.cursor/rules`(tokens.mdc, design-system.mdc)와 이 파일을 함께 두면, 새 도구·MCP가 나와도 “토큰·컴포넌트 규격” 기준은 유지하면서 연동만 확장할 수 있습니다.

---

## 4. 확장 시

- 새 **MCP**(예: 디자인 시스템 전용 MCP)나 **도구**를 쓰면 이 파일에 **이름·용도·연동 한 줄·링크** 추가.
- design ↔ frontend-web 간 **핸드오프 절차**가 바뀌면 [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md)와 frontend-web의 **TOOLS_AND_MCP.md**를 함께 갱신.
