---
name: nextjs-patterns
description: Next.js App Router, Server/Client Component, 데이터 fetch·캐싱 실무 패턴.
---

# Next.js Patterns Skill

## App Router

- **app/** 사용. page.tsx, layout.tsx, loading.tsx, error.tsx, not-found.tsx로 라우트·로딩·에러 처리.
- **Server Component 기본**: async 컴포넌트에서 직접 fetch. 클라이언트 번들 감소.
- **"use client"**: 이벤트, useState/useEffect, 브라우저 API 필요 시에만. 가능하면 leaf 노드에 제한.

## 데이터 fetch

- Server Component: fetch(..., { next: { revalidate: 60 } }) 등으로 캐싱. 동적은 cache: 'no-store'.
- 클라이언트: SWR/React Query 등으로 클라이언트 캐시·재검증.

## 라우팅·레이아웃

- (그룹): URL에 반영되지 않는 라우트 그룹. layout 공유.
- 병렬/인터셉팅 라우트: @folder, (.)(..) 규칙으로 모달·로딩 UI 구성 시 활용.

## 성능·접근성

- 동적 import로 클라이언트 컴포넌트 지연 로드. next/link로 클라이언트 네비게이션. 이미지는 next/image로 최적화.
- eslint-plugin-jsx-a11y, 시맨틱 HTML, 포커스·aria 유지.
