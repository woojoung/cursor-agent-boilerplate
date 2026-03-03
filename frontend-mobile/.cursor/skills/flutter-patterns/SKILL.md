---
name: flutter-patterns
description: Flutter 레이어·상태관리·네비게이션·플랫폼 채널 실무 패턴.
---

# Flutter Patterns Skill

## 레이어

- **Presentation**: Screen + ViewModel. ViewModel은 ChangeNotifier/Riverpod/Bloc 등으로 상태 보유, Repository 호출.
- **Data**: Repository 구현체, Remote/Local DataSource. Model ↔ Entity 변환.
- **Domain**(선택): 복잡한 경우 UseCase/Entity.

## 상태 관리

- 프로젝트 채택안(Provider, Riverpod, Bloc) 준수. setState는 로컬 UI 상태만.
- 비동기: async/await, FutureBuilder/StreamBuilder 적절히. 로딩·에러 상태 처리.

## 네비게이션

- GoRouter, Navigator 2.0 등 일관된 라우팅. 딥링크·백스택 정책 명확히.

## 플랫폼

- MethodChannel/EventChannel 필요 시 platform 별 코드 분리. 테스트 시 Mock 채널 사용.
