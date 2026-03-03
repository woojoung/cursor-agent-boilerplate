---
name: widget-design
description: 위젯 분할·ViewModel·재사용·테스트 가능 설계.
---

# Widget Design Skill (frontend-mobile)

## 분할

- 한 화면 = 하나의 Screen 위젯 + ViewModel. 화면 내 영역은 작은 위젯으로 분리(screens/{feature}/widgets/).
- 공통 UI는 presentation/widgets 또는 core/widgets.

## ViewModel

- 입력(사용자·라우트)·Repository 호출·상태 변경. View는 ViewModel listen만.
- 테스트: Repository를 Mock/Fake로 주입해 단위 테스트.

## 재사용

- 파라미터는 생성자로. 콜백(onXxx)으로 이벤트 상위 전달. 공통 테마·텍스트 스타일은 Design 시스템 토큰 사용.

## 테스트

- ViewModel 테스트로 비즈니스 로직 검증. 위젯 테스트는 Fake ViewModel로 UI·탭·입력 검증.
