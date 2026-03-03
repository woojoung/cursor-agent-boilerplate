---
name: pipeline-design
description: CI/CD 단계·검증·배포·승인 게이트 설계.
---

# Pipeline Design Skill (infra)

## 단계

- **Build**: 소스 체크아웃, 의존성, 아티팩트 생성. Docker 이미지 빌드·푸시.
- **Test**: 단위·통합 테스트. 정적 분석(코드·IaC).
- **Deploy**: 환경별 apply 또는 이미지 배포. 승인 게이트(수동/자동) 정책에 따름.

## 보안

- 비밀은 파이프라인 시크릿. OIDC 등 단기 토큰 권장. 최소 권한 역할.
- IaC 변경 시 plan·tfsec 결과 리뷰 후 apply.

## 롤백

- 이전 아티팩트·이미지 태그 보존. 롤백 시 동일 파이프라인으로 이전 버전 배포 가능하게.
