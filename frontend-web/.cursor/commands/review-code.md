# Code Review (frontend-web)

코드 리뷰를 실행한다. 컴포넌트·아키텍처·접근성·테스트 관점으로 검토한다.

## 리뷰 체크리스트

| 항목 | 검사 내용 |
|------|----------|
| 아키텍처 | App Router 사용, Server/Client 구분, 폴더 구조 |
| 컴포넌트 | 단일 책임, props·타입, 재사용성 |
| 접근성 | 시맨틱, alt, label, 포커스·aria |
| 테스트 | 핵심 동작·접근성 검증 여부 |
| 성능 | 불필요한 "use client", 번들·이미지 고려 |

## 실행 모드

- **경로 지정**: 예) `app/dashboard`, `components/feature/user` → 해당 경로 리뷰.
- **출력**: 항목별 Pass/이슈 개수, 이슈별 파일:줄·내용·개선 제안.
