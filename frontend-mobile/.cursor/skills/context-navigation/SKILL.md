---
name: context-navigation
description: AGENTS.md·lib 폴더 구조 기반 Flutter 프로젝트 맥락 파악.
---

# Context Navigation Skill (frontend-mobile)

## 목적

- lib/ 하위 presentation / data / domain 구조 파악
- 화면(screens)·ViewModel·Repository 위치 확인
- 의존성 방향: UI → ViewModel → Repository → DataSource

## 탐색

1. 루트 AGENTS.md → 프로젝트·포지션 규칙
2. lib/presentation/screens/{feature} → 화면·ViewModel·widgets
3. lib/data/repositories, datasources → 데이터 계층
4. import 따라가며 의존성 확인
