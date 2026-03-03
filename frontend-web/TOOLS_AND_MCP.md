# 프론트엔드(웹) — 추천 도구·MCP

> 이 포지션에서 **실무 레포**나 Cursor에 추가할 수 있는 **도구·라이브러리·MCP** 예시입니다.  
> 새 도구가 나오면 이 파일에 항목을 추가해 확장하면 됩니다. (전체 확장 가이드: [docs/EXTENSIBILITY.md](../docs/EXTENSIBILITY.md))

---

## 1. 시각 피드백 — Agentation

**용도**: AI 에이전트가 만든 UI를 **클릭·텍스트 선택·영역 선택**으로 지적해, “여기 패딩 늘려줘”, “이 색 더 어둡게” 같은 피드백을 **구조화된 마크다운**(셀렉터·위치·컨텍스트)으로 뽑아 Cursor에 붙여 넣어 반복 수정할 때 사용.

**설치·연동** (실무 레포):

```bash
npm install agentation
```

```tsx
// layout.tsx 등 (개발 환경에서만)
import { Agentation } from "agentation";

export default function Layout({ children }) {
  return (
    <>
      {children}
      {process.env.NODE_ENV === "development" && <Agentation />}
    </>
  );
}
```

**참고**: [Agentation 소개](https://benji.org/agentation) · npm: `agentation`

---

## 2. 디자인 → 코드 — Figma MCP

**용도**: **Figma** 디자인 데이터(레이어·변수·스타일·오토 레이아웃)를 Cursor에 전달해, 에이전트가 디자인 의도에 맞는 컴포넌트·페이지 코드를 생성·수정.

**연동 요약**:

- Figma **Personal Access Token** 발급 (Figma 설정 > Security).
- Cursor **MCP 설정**에 Figma MCP 서버 추가 (`~/.cursor/mcp.json` 또는 Cursor 설정).
- Cursor 컴포저에 **Figma 파일/노드 링크** 붙여 넣고 에이전트에게 “이 디자인 기준으로 컴포넌트 만들어줘” 요청.

**Figma 측 권장**: Variables(타이포·간격·색·radius), Auto Layout 전반 사용 시 MCP가 구조를 잘 인식합니다.

**참고**: [Figma to Cursor MCP](https://cursorideguide.com/use-cases/figma-to-cursor-with-mcp) · [Figma MCP Server (Builder.io)](https://builder.io/blog/figma-mcp-server) · [Figma Developer Docs – MCP](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/)

---

## 3. 디자인 핸드오프 — Stitch (Figma 연동)

**용도**: Figma와 연동해 **디자인 → 코드** 변환. 디자이너 에이전트가 정리한 UI를 Figma에 두고, Stitch + Figma MCP로 프론트엔드 개발 시 동일한 소스 사용.

**워크플로우**:

- **design** 포지션: Figma에서 UI·토큰 정리 → Stitch/내보내기 설정.
- **frontend-web** 포지션: Figma MCP(또는 Stitch 연동)로 Cursor에서 해당 디자인을 참조해 컴포넌트 구현·수정.

Stitch 사용 시 팀 정책에 맞게 **TOOLS_AND_MCP.md**에 설치·환경 변수·Cursor 연동 단계를 추가하면 됩니다.

---

## 4. 기술 스택 예시 (선택)

실무 레포 **기술 스택 예시**를 이 포지션에 두고 싶다면 아래를 참고해 **examples/** 또는 이 파일 하단에 “추천 dependency 목록”을 넣을 수 있습니다.

- **프레임워크**: Next.js (App Router).
- **스타일**: Tailwind CSS, CSS Modules, 디자인 토큰(선택).
- **테스트**: Vitest, React Testing Library, Playwright.
- **접근성**: eslint-plugin-jsx-a11y, eslint-config-next.

---

## 5. 확장 시

- 새 **MCP**(예: 디자인 시스템 MCP, API 목업 MCP)를 쓰면 이 파일에 **이름·용도·연동 한 줄·링크**를 추가.
- `.cursor/rules` 또는 `.cursor/skills`에 “이 도구 사용 시 주의사항” 한 줄을 넣어 에이전트가 참고하게 할 수 있음.
