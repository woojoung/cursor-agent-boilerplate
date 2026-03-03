# Spec 템플릿 생성

spec.md 템플릿을 생성하여 Spec-Driven Development를 시작합니다.

## 실행 절차

1. 사용자에게 기능 이름, 목표, 요구사항을 질문한다.
2. spec.md 위치를 결정한다 (예: `docs/prompt/{username}/{YY-MM-DD-feature}/spec.md`).
3. `.cursor/skills/spec-template/SKILL.md`의 템플릿을 참고하여 spec.md를 생성한다.
4. 완료 후 다음 단계(`/plan-from-spec`)를 안내한다.

## 사용 예시

- 인자 없음: 기능 이름·목표를 질문한 뒤 spec 초안 작성
- 인자 있음: 예) "사용자 인증 기능" → 해당 기능에 맞는 spec.md 초안 생성

완료 시: "spec.md를 열어 요구사항을 상세히 작성한 뒤(30줄 이내 권장), `/plan-from-spec`을 실행하세요"라고 안내한다.
