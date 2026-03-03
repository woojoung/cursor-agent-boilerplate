# Code Review (frontend-mobile)

Flutter 코드 리뷰. 위젯·ViewModel·Repository·테스트 관점.

## 체크리스트

| 항목 | 검사 내용 |
|------|----------|
| 레이어 | UI/Data 분리, 로직이 위젯에 없는지 |
| ViewModel | 단일 책임, Repository 주입, 테스트 가능 |
| 위젯 | 분할·재사용, build 안 로직 최소 |
| 테스트 | ViewModel·위젯 테스트 존재·품질 |

출력: 항목별 Pass/이슈, 이슈별 파일:줄·제안.
