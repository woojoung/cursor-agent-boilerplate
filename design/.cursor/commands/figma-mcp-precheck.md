---
description: Figma MCP 연결·전제 점검 (채널, 문서, 스타일/컴포넌트 스냅샷, 레포 경로와 갭 요약)
---

## 목표

Talk to Figma 세션을 쓰기 전에 연결과 Figma 측 메타를 점검하고, 레포의 토큰·컴포넌트 진실 공급원과 비교 가능한 요약을 남긴다.

## 사용자 입력 (필수)

1. **채널**: Figma 플러그인에 표시된 채널 문자열을 그대로 붙여넣기 (매 연결마다 바뀔 수 있음). **AI는 채널을 자동 조회할 수 없다.**
2. **레포 경로**: 디자인 토큰·UI 컴포넌트 루트 또는 매핑 문서 경로 (@ 파일로 지정).

## 에이전트 절차

1. `join_channel` 호출 (사용자가 준 채널). 실패 시 플러그인 실행·네트워크·채널 복사 여부 안내.
2. `get_document_info`로 문서 연결 확인.
3. `get_styles`, `get_local_components` 호출해 이름 목록·요약 수집.
4. (선택) 사용자가 프레임을 선택한 상태면 `read_my_design` 또는 `get_selection`으로 샘플 위계 확인.
5. 사용자가 지정한 레포 파일을 읽고, Figma 이름과 **교집합 / Figma만 있음 / 코드만 있음** 표로 갭 요약.
6. **MCP로 불가**: 레이어 **이름 변경**은 Talk to Figma MCP에 일반적으로 없음 → 필요 시 Figma에서 수동 정리 안내.

## 산출물

- 연결 성공 여부
- Figma 스타일·로컬 컴포넌트 요약
- 코드↔Figma 이름 갭 표 (가능한 범위)
- 다음 액션 (매핑 문서 추가, 리네이밍 백로그 등)

## 관련 문서·스킬

- [docs/process/figma-mcp-handoff.md](../../../docs/process/figma-mcp-handoff.md)
- 스킬: `.cursor/skills/figma-to-code-workflow/SKILL.md`
