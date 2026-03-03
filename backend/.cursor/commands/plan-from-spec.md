# Plan from Spec

spec.md를 분석하여 구현 계획(plan.md)을 생성한다. TDD/검증/커버리지/리뷰 파이프라인이 포함된다.

## 실행 절차

1. spec.md 탐색 (`./spec.md`, `./docs/spec.md`, `./*.spec.md`).
2. spec.md 분석: 목표, 요구사항, 제약사항 추출.
3. 코드베이스 분석: 유사 기능, 영향 범위, 의존성.
4. plan.md 생성 (TDD/Verify/Coverage/Review 파이프라인 포함).
5. 사용자 승인을 기다린다.

## plan.md 포함 항목

- Phase 1: 테스트 작성 (RED)
- Phase 2: 구현 (GREEN)
- Phase 3: 리팩토링 (REFACTOR)
- 파이프라인: 빌드 검증, 커버리지 80%, 코드 리뷰

## 모드

- **인자 없음**: 현재 디렉터리의 spec.md를 분석하여 plan.md 생성.
- **인자 있음**: 예) "사용자 인증 기능 추가" → 대화형으로 요구사항 정리 후 spec.md 작성 유도, 이어서 plan.md 생성.

완료 시: "plan.md가 생성되었습니다. TDD·빌드/테스트 검증·커버리지 80%·코드 리뷰가 포함되어 있습니다. plan.md를 검토하고 승인해 주세요."라고 안내한다.
