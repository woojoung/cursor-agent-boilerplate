# 시작 (역질문 모드) — QA

사용자가 **`/start`** 또는 "도와줘", "QA 쪽 뭐 해줘?" 등 **모호한 요청**을 하면, **PROMPT_TEMPLATES.md**의 옵션(① 스펙→시나리오 ② 테스트 계획 ③ 버그 리포트 템플릿)을 제시하고, 선택에 따라 추가 질문 후 해당 명령(`/scenario-from-spec`, `/test-plan`, `/bug-template`)을 실행하거나 안내한다.
