# 확장성 가이드 — 도구·MCP·기술 스택·Slack

> 이 문서의 목적: 각 포지션별로 **실무 레포에 넣을 것**, **Slack에 추가할 것**, **기술 스택·라이브러리·MCP** 등을 확장할 수 있는 **위치와 규칙**을 정리합니다. 새 도구(예: [Agentation](https://benji.org/agentation))나 MCP(예: Figma, Stitch)가 나와도 일관된 방식으로 연동·고도화할 수 있게 합니다.

---

## 1. 확장 포인트 요약

| 확장 유형 | 어디에 두나 | 누가 쓸 수 있나 |
|----------|-------------|-----------------|
| **포지션별 도구·MCP·기술 스택** | 해당 포지션 폴더의 **TOOLS_AND_MCP.md** | 해당 포지션 실무 레포·Cursor |
| **추가 규칙·스킬·명령** | 해당 포지션의 **.cursor/rules**, **.cursor/skills**, **.cursor/commands** | Cursor에서 자동/슬래시 |
| **Slack 챗/봇 추가** | 팀 정책 문서 또는 **docs/INTEGRATION.md** 보완·채널 매핑 테이블 | Slack 봇·채널 |
| **실무 레포 예시·템플릿** | 포지션 폴더의 **examples/** 또는 **docs/** (선택) | 복사·참고용 |

- **TOOLS_AND_MCP.md**: 해당 포지션에서 권장하는 **라이브러리**, **MCP 서버**, **개발 도구**를 나열하고, 설치·연동 방법을 한 곳에 적어 둡니다. 새 도구가 나오면 이 파일만 갱신하면 됩니다.
- **실무 레포**: 이 보일러플레이트를 플러그인/복사로 적용한 뒤, 실무 레포 쪽에 **패키지·MCP 설정**만 추가하면 됩니다. (예: `npm install agentation`, Cursor MCP에 Figma 추가.)

---

## 2. 포지션별 TOOLS_AND_MCP.md

각 포지션 폴더에 **TOOLS_AND_MCP.md**를 두면, 그 포지션의 기술 스택 예시·추천 라이브러리·MCP를 한곳에서 관리할 수 있습니다.

- **backend**: DB MCP, API 클라이언트, 모니터링 도구 등.
- **frontend-web**: [Agentation](https://benji.org/agentation)(시각 피드백), Figma MCP, Stitch(디자인→코드) 등. → 아래 3절·4절 참고.
- **frontend-mobile**: Flutter MCP(있을 경우), 디자인 핸드오프 도구.
- **qa**: 테스트 러너·E2E MCP, 버그 트래킹 연동.
- **infra**: Terraform·클라우드 MCP, 모니터링 MCP.
- **pm**: 이슈 트래커 MCP(Jira, Linear 등), 노션 MCP.
- **design**: Figma MCP, Stitch, 디자인 토큰 내보내기. 디자이너 에이전트가 만든 UI를 Figma와 연동해 프론트엔드에서 사용하는 흐름을 문서화.

**규칙**:  
- TOOLS_AND_MCP.md는 **선택** 파일입니다. 없으면 기존 rules/skills/commands만 사용해도 됩니다.  
- 새 도구·MCP를 도입할 때는 이 파일에 **이름·용도·설치/연동 한 줄·공식 링크**를 추가하고, 필요 시 `.cursor/rules` 또는 skills에 “이 도구 사용 시 주의사항”을 한 줄씩 넣으면 됩니다.

---

## 3. 예시: 프론트엔드 웹 — Agentation

[Agentation](https://benji.org/agentation)은 AI 코딩 에이전트용 **시각 피드백 도구**입니다. UI를 클릭·텍스트 선택·영역 선택해 “여기 패딩 늘려줘”, “이 색 더 어둡게” 같은 피드백을 구조화된 마크다운(셀렉터·위치·컨텍스트)으로 뽑아, Cursor 등에 붙여 넣어 반복 수정할 때 유리합니다.

- **확장 방법**:  
  - **frontend-web/TOOLS_AND_MCP.md**에 Agentation 항목을 추가하고,  
  - 실무 레포(Next.js 등)에서는 `npm install agentation` 후 레이아웃에 `<Agentation />`를 개발 환경에서만 포함합니다.  
- **이점**: 에이전트가 생성한 UI를 “가리키면서” 수정 지점을 전달할 수 있어, 반복 개선이 빨라집니다.  
- **참고**: [Agentation 소개](https://benji.org/agentation), [npm: agentation](https://www.npmjs.com/package/agentation).

---

## 4. 예시: Figma + Stitch MCP — 디자이너 에이전트 → 프론트엔드

디자이너 에이전트가 정리한 UI를 **Figma**에 두고, **Figma MCP**와 **Stitch**(디자인→코드) 같은 연동으로 Cursor에서 바로 코드로 가져와 프론트엔드 개발에 쓰는 흐름입니다.

- **Figma MCP**  
  - Figma 디자인 데이터(레이어, 변수, 스타일, 오토 레이아웃 등)를 Cursor에 전달해, 에이전트가 디자인 의도에 맞는 코드를 생성하게 합니다.  
  - Cursor 설정에 Figma MCP 서버를 추가하고, Figma Personal Access Token을 넣으면 됩니다.  
  - 참고: [Figma to Cursor MCP](https://cursorideguide.com/use-cases/figma-to-cursor-with-mcp), [Figma MCP Server (Builder.io)](https://builder.io/blog/figma-mcp-server), [Figma Developer Docs – MCP](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/).

- **Stitch**  
  - Figma와 연동해 디자인을 코드로 변환하는 흐름에서 함께 쓰는 경우가 많습니다.  
  - **design** 포지션: Figma·Stitch를 TOOLS_AND_MCP.md에 넣고, “디자이너 에이전트가 만든 UI → Figma → MCP/Stitch → 프론트엔드 레포” 경로를 문서화합니다.  
  - **frontend-web** 포지션: 같은 MCP·Stitch 설정을 TOOLS_AND_MCP.md에 적어 두고, Cursor에서 Figma 링크를 붙여 넣어 컴포넌트·페이지를 생성·수정하는 방법을 정리합니다.

- **확장 방법**  
  - **design/TOOLS_AND_MCP.md**: Figma MCP, Stitch, 디자인 토큰 내보내기. “디자이너가 Figma에서 정리한 UI를 프론트엔드에서 MCP로 사용” 흐름 요약.  
  - **frontend-web/TOOLS_AND_MCP.md**: Figma MCP, Stitch. “Figma 링크/파일을 Cursor에 넣고 컴포넌트 생성·수정” 절차.  
  - 두 포지션 모두 **.cursor/rules** 또는 skills에 “Figma 변수·오토 레이아웃 사용 시 MCP 품질이 좋아짐” 같은 한 줄 가이드를 넣을 수 있습니다.

---

## 5. Slack·실무 레포에 추가될 것

- **Slack**:  
  - 포지션별·프로젝트별 **채널**, **봇 멘션 규칙**, **알림 포맷**은 팀 정책에 따라 다릅니다.  
  - 확장 시 **docs/INTEGRATION.md**의 “채널 매핑”, “이슈 트래커 연동” 섹션에 “추가할 챗/봇/포맷”을 정리하거나, 팀 전용 문서에 “Slack 추가 항목” 표를 두면 됩니다.  
  - 예: “프론트엔드 MR 시 #design 리뷰 요청 멘션”, “QA 완료 시 BOSS 채널에 한 줄 요약”.

- **실무 레포**:  
  - 각 포지션 **TOOLS_AND_MCP.md**에 적어 둔 **패키지·MCP·환경 변수**를 실무 레포에만 설치·설정하면 됩니다.  
  - 레포 예시(기술 스택 샘플)가 필요하면, 해당 포지션 폴더에 **examples/** 또는 **docs/stack-example.md** 같은 파일을 두고 “예시 dependency 목록·MCP 설정 스니펫”을 넣어 확장할 수 있습니다.

---

## 6. 정리

- **확장성**: 포지션별 **TOOLS_AND_MCP.md**로 도구·MCP·기술 스택을 관리하고, 새 라이브러리·MCP가 나오면 이 파일과 필요 시 rules/skills만 갱신하면 됩니다.  
- **Agentation**: 프론트엔드 웹 개발 시 시각 피드백으로 반복 수정을 줄이려면 frontend-web에 Agentation을 TOOLS_AND_MCP에 추가하고, 실무 레포에 `agentation` 패키지를 넣으면 됩니다.  
- **Figma·Stitch**: 디자이너 에이전트가 만든 UI를 Figma에 두고, Figma MCP + Stitch로 Cursor 프론트엔드 작업에 쓰려면 design·frontend-web 양쪽 TOOLS_AND_MCP.md에 연동 방법을 적고, Cursor MCP 설정만 실무 레포/전역에 추가하면 됩니다.
