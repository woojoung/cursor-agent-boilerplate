---
name: terraform-patterns
description: Terraform 모듈·상태·변수·보안(tfsec) 실무 패턴.
---

# Terraform Patterns Skill (infra)

## 모듈

- 환경(dev/stage/prod)별 tfvars. 공통은 모듈로. output으로 연동.
- 리소스 네이밍: {env}-{project}-{resource}. 태그 일관 적용.

## 상태

- 원격 백엔드 필수. 락 활성화. 상태 접근 권한 최소화.
- sensitive output 마킹. 상태 파일 공유·로컬 저장 금지.

## 보안

- tfsec 실행: unencrypted storage, public access, 과도한 권한, 하드코딩 비밀 검사.
- CI에서 terraform plan + tfsec. plan 결과 리뷰 후 apply.

## 변수

- type·description 명시. default는 선택. 비밀은 변수로 받고 환경/볼트에서 주입.
