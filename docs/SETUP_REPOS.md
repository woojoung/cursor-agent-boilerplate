# 포지션별 기본 레포 세팅 · 기술 스택 검증

> 이 문서의 목적: (1) 각 포지션의 **.cursor 하위 내용이 해당 기술 스택 기반으로 작성되어 있는지** 검증 결과를 정리하고, (2) 작업자가 **필요 시 기본 레포를 세팅**할 수 있도록 **백엔드(Java/Spring)**, **프론트 웹(Next.js)**, **프론트 모바일(Flutter)** 순서로 절차를 안내합니다.
>
> **자동 세팅**: 해당 포지션 폴더(backend, frontend-web, frontend-mobile)를 Cursor에서 연 뒤, 채팅에서 **`/setup-repo`** 를 실행하거나 "백엔드(또는 프론트 웹/모바일) 새 레포 세팅해줘"라고 요청하면, 에이전트가 프로젝트명을 묻고 터미널로 프로젝트 생성 + .cursor·AGENTS.md 복사를 진행합니다.

---

## 1. 기술 스택 검증 — .cursor 내용이 해당 스택 기반인지

각 포지션의 **.cursor/rules**, **.cursor/skills**, **.cursor/commands**, **.cursor/agents**는 아래 기술 스택에 맞게 작성되어 있습니다.

### 1.1 백엔드 (backend/) — Java / Spring Boot

| 항목 | 기술 스택 반영 여부 | 내용 요약 |
|------|---------------------|-----------|
| **rules** | ✅ | coding-style: Java/Spring Boot, Clean Architecture, CQRS, JPA Entity vs Model. testing: `**/*Test.java`, Mockito, @DataJpaTest, @WebMvcTest, @SpringBootTest. security: OWASP, Spring Security. |
| **skills** | ✅ | java-code-style, spec-template, test-patterns(Spring 테스트), security-checklist(Spring Security), gitlab-workflow, verification-loop(gradlew). |
| **commands** | ✅ | spec-template, plan-from-spec, review-code(Spring·Clean Architecture), mr-gitlab, commit-push. |
| **agents** | ✅ | planner, code-reviewer, security-reviewer(OWASP·Spring Security), build-fixer, tdd-guide, test-runner, test-writer 등. |

**결론**: 백엔드 .cursor 전반이 **Java 21, Spring Boot, Clean Architecture, JPA, TDD(JUnit/Mockito)** 기준으로 일관되게 작성됨.

---

### 1.2 프론트엔드 웹 (frontend-web/) — Next.js / React / TypeScript

| 항목 | 기술 스택 반영 여부 | 내용 요약 |
|------|---------------------|-----------|
| **rules** | ✅ | coding-style: Next.js App Router, app/, Server/Client Components, .tsx/.ts. testing: Vitest/Jest, React Testing Library, Playwright/Cypress. a11y: 시맨틱·WCAG·jsx-a11y. |
| **skills** | ✅ | context-navigation(Next.js app/components/lib), spec-template, nextjs-patterns(App Router, fetch, 라우팅), component-design. |
| **commands** | ✅ | spec-template, plan-from-spec, review-code(App Router·접근성), mr-gitlab. |
| **agents** | ✅ | planner(Next.js 기준 plan), code-reviewer(Next.js/React), component-designer, test-runner. |

**결론**: 프론트 웹 .cursor 전반이 **Next.js App Router, React, TypeScript, Vitest/React Testing Library** 기준으로 일관되게 작성됨.

---

### 1.3 프론트엔드 모바일 (frontend-mobile/) — Flutter / Dart

| 항목 | 기술 스택 반영 여부 | 내용 요약 |
|------|---------------------|-----------|
| **rules** | ✅ | coding-style: Flutter/Dart, lib/ 구조, presentation/data/domain, ViewModel·Repository, .dart. testing: `*_test.dart`, mocktail, widget_test, integration_test. |
| **skills** | ✅ | context-navigation(lib 구조), spec-template, flutter-patterns(ViewModel·Repository·상태관리), widget-design. |
| **commands** | ✅ | spec-template, plan-from-spec, review-code(위젯·ViewModel), mr-gitlab. |
| **agents** | ✅ | planner(Flutter 기준 plan), code-reviewer(Flutter), widget-designer, test-runner(flutter test). |

**결론**: 프론트 모바일 .cursor 전반이 **Flutter, Dart, ViewModel/Repository 레이어, widget_test** 기준으로 일관되게 작성됨.

---

## 2. 포지션별 기본 레포 세팅 절차

작업자가 **새 실무 레포**를 해당 기술 스택으로 만들고, 이 보일러플레이트의 **.cursor + AGENTS.md**를 적용하는 절차입니다.

### 2.1 백엔드 (Java / Spring Boot)

1. **프로젝트 생성** (둘 중 하나)
   - [Spring Initializr](https://start.spring.io/): Java 21, Spring Boot 3.x, Gradle, 필요한 의존성(Web, JPA, Validation 등) 선택 후 다운로드·압축 해제.
   - 또는 CLI:  
     `curl https://start.spring.io/starter.zip -d type=gradle-project -d language=java -d javaVersion=21 -d bootVersion=3.x -d name=my-backend -o backend.zip && unzip backend.zip -d my-backend && cd my-backend`

2. **기본 구조 확인**  
   - `src/main/java`, `src/test/java`, `build.gradle`(또는 pom.xml) 존재.

3. **.cursor + AGENTS.md 적용**  
   - 이 레포의 **cursor-agent-boilerplate/backend/** 에서 아래를 **새 레포 루트**로 복사합니다.  
     - `.cursor/` (rules, commands, skills, agents 통째로)  
     - `AGENTS.md`  
     - (선택) `.cursor-plugin/`

4. **Cursor에서 열기**  
   - 새 레포 폴더를 Cursor에서 Open 하면 슬래시 명령·규칙이 적용됩니다.

---

### 2.2 프론트엔드 웹 (Next.js)

1. **프로젝트 생성**
   ```bash
   npx create-next-app@latest my-frontend --ts --eslint --app --src-dir=false
   cd my-frontend
   ```
   - (선택) Tailwind, import alias 등 프롬프트에서 선택.

2. **기본 구조 확인**  
   - `app/`, `components/`, `package.json` 존재.  
   - `.cursor` 규칙의 app/components/lib 구조와 맞추려면 `lib/`, `hooks/`, `types/` 폴더를 추가해도 됩니다.

3. **.cursor + AGENTS.md 적용**  
   - 이 레포의 **cursor-agent-boilerplate/frontend-web/** 에서 아래를 **새 레포 루트**로 복사합니다.  
     - `.cursor/`  
     - `AGENTS.md`  
     - (선택) `.cursor-plugin/`

4. **Cursor에서 열기**  
   - 새 레포 폴더를 Cursor에서 Open 합니다.

---

### 2.3 프론트엔드 모바일 (Flutter)

1. **프로젝트 생성**
   ```bash
   flutter create my_mobile_app
   cd my_mobile_app
   ```

2. **기본 구조 확인**  
   - `lib/`, `test/`, `pubspec.yaml` 존재.  
   - `.cursor` 규칙의 presentation/data/domain 구조로 가려면 `lib/` 하위에 `core/`, `data/`, `presentation/` 폴더를 만들고, `main.dart`를 그에 맞게 정리하면 됩니다.

3. **.cursor + AGENTS.md 적용**  
   - 이 레포의 **cursor-agent-boilerplate/frontend-mobile/** 에서 아래를 **새 레포 루트**로 복사합니다.  
     - `.cursor/`  
     - `AGENTS.md`  
     - (선택) `.cursor-plugin/`

4. **Cursor에서 열기**  
   - 새 레포 폴더를 Cursor에서 Open 합니다.

---

## 3. 체크리스트 — 세팅 후 확인

- [ ] 새 레포가 해당 기술 스택으로 생성됨 (백엔드: Gradle/Maven + Java 21 + Spring Boot, 프론트 웹: Next.js + TypeScript, 모바일: Flutter).
- [ ] `.cursor/`(rules, commands, skills, agents)와 `AGENTS.md`가 해당 포지션 폴더에서 복사되어 있음.
- [ ] Cursor에서 해당 폴더를 Open 했을 때 `/` 입력 시 포지션에 맞는 명령이 보임 (예: backend → plan-from-spec, review-code).
- [ ] rules의 glob 패턴이 실제 파일 확장자와 맞음 (백엔드: `**/*.java`, 프론트 웹: `**/*.tsx`, `**/*.ts`, 모바일: `**/*.dart`).

위가 모두 만족되면, 해당 기술 스택 기반의 기본 레포 세팅이 완료된 것이며, .cursor 하위 내용도 그 스택에 맞게 적용된 상태입니다.
