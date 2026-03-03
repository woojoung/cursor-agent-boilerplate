# 특정 레포지토리에서 cursor-agent-boilerplate 플러그인 추가하기

> Cursor에서 **작업할 레포(실무 레포)** 를 연 상태에서, **pm / design / frontend-mobile** 등 포지션 플러그인을 쓰기 위해 추가하는 방법을 단계별로 정리합니다.

---

## 준비

- **대상 레포**를 Cursor에서 **File → Open Folder**로 연 상태에서 진행합니다.
- **cursor-agent-boilerplate**는 이미 GitHub에 있고, 로컬에 클론해 두었다고 가정합니다.  
  예: `.../cursor-agent-boilerplate`

---

## 방법 1. Cursor Settings → Plugins에서 추가 (URL 또는 로컬)

### 1단계: Plugins 화면 열기

1. **Cursor** 실행 후 **대상 레포** 폴더 열기.
2. 메뉴에서 **Cursor → Settings** (또는 `Cmd + ,`).
3. 왼쪽 사이드바에서 **Plugins** 클릭.  
   → 지금 보이는 화면이 "Plugins" 설정입니다.

### 2단계: 플러그인 소스 추가할 수 있는 곳 찾기

현재 화면에는 **Browse Marketplace** 버튼과 Suggested 플러그인 목록만 보일 수 있습니다. 아래를 순서대로 확인해 보세요.

- **Plugins 화면 안**
  - 상단이나 목록 위/아래에 **"+ Add"**, **"Add plugin"**, **"Install from URL"**, **"Add from path"** 같은 버튼/링크가 있으면 클릭.
  - 나오는 입력창에:
    - **URL로 넣는 경우**: `https://github.com/woojoung/cursor-agent-boilerplate`
    - **로컬 경로로 넣는 경우**: 아래 "로컬 경로 예시" 참고.
- **Browse Marketplace 클릭 후**
  - 마켓플레이스 화면에 **"Add from GitHub"**, **"Install from repository"**, **URL 입력란**이 있으면,  
    `https://github.com/woojoung/cursor-agent-boilerplate` 입력 후 설치.
- **왼쪽 사이드바의 "Marketplace"**
  - **Marketplace** 항목을 클릭해 보세요.  
    거기에 검색창·URL 입력·로컬 경로 추가가 있을 수 있습니다.

### 3단계: 로컬 경로로 넣는 경우 (폴더 단위)

Cursor가 **로컬 폴더 경로**를 플러그인으로 받는다면, **포지션당 하나씩** 아래처럼 추가합니다. 필요한 포지션만 골라 쓰면 됩니다.

| 추가할 플러그인 | 넣을 경로 (로컬 클론 기준) |
|-----------------|----------------------------|
| **pm**          | `.../cursor-agent-boilerplate/pm` |
| **design**      | `.../cursor-agent-boilerplate/design` |
| **frontend-mobile** | `.../cursor-agent-boilerplate/frontend-mobile` |
| **frontend-web** | `.../cursor-agent-boilerplate/frontend-web` |
| **backend**     | `.../cursor-agent-boilerplate/backend` |
| **qa**          | `.../cursor-agent-boilerplate/qa` |
| **infra**       | `.../cursor-agent-boilerplate/infra` |

**예시 (맥 기준):**

```
/Users/you/cursor-agent-boilerplate/pm
/Users/you/cursor-agent-boilerplate/design
/Users/you/cursor-agent-boilerplate/frontend-mobile
```

각 경로는 **해당 포지션 폴더 전체**(그 안에 `.cursor-plugin/plugin.json`이 있는 폴더)를 가리키면 됩니다.

### 4단계: 적용 확인

- **대상 레포** 프로젝트에서 채팅 창에 **`/`** 입력.
- **pm** 플러그인: `/spec-template`, `/task-breakdown` 등이 보이면 됩니다.
- **design** 플러그인: `/design-review` 등이 보이면 됩니다.
- **frontend-mobile** 플러그인: `/plan-from-spec`, `/review-code` 등이 보이면 됩니다.
- (다른 포지션도 해당 폴더 추가 시 해당 명령·스킬이 나타납니다.)

---

## 방법 2. 복사로 쓰기 (플러그인 UI가 없을 때)

Cursor에서 "URL/로컬로 플러그인 추가"가 안 보이거나, Private 레포라 마켓플레이스에 안 올렸다면, **포지션 폴더 내용을 대상 레포에 복사**해서 같은 효과를 낼 수 있습니다.

### 1단계: 복사할 소스

- **cursor-agent-boilerplate**를 클론한 위치에서, 쓸 포지션만 골라:
  - `pm/` → `.cursor/` (rules, commands, skills, agents), `AGENTS.md`
  - `design/` → 위와 동일
  - `frontend-mobile/` → 위와 동일
  - (필요 시 frontend-web, backend, qa, infra 동일)

### 2단계: 대상 레포에 넣는 방식

- **대상 레포**에 이미 **.cursor/rules/** 가 있으면 **rules는 덮어쓰지 말고** 유지합니다. (프로젝트 전용 규칙 보존.)
- **commands, skills, agents**는 대상 레포에 없을 수 있으므로, 쓸 포지션만 골라서:
  - `cursor-agent-boilerplate/pm/.cursor/commands` → `{대상 레포}/.cursor/commands/` (기존 commands에 합치거나 pm 하위로)
  - `cursor-agent-boilerplate/pm/.cursor/skills` → `{대상 레포}/.cursor/skills/`
  - `cursor-agent-boilerplate/pm/.cursor/agents` → `{대상 레포}/.cursor/agents/`
  - design, frontend-mobile 등도 같은 식으로 **폴더 단위로** 복사해 넣습니다.
- **AGENTS.md**는 대상 레포 루트에 이미 있으면, 보일러플레이트의 AGENTS.md 내용 중 **문서 진입점·역할별 참조**만 참고해 루트 AGENTS.md를 보완해도 됩니다.

이렇게 하면 Cursor는 **대상 레포/.cursor** 아래의 rules/commands/skills/agents를 읽어서, 플러그인을 "설치"하지 않아도 비슷하게 동작합니다.

---

## 방법 3. 마켓플레이스에 등록된 경우

나중에 [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)에서 **cursor-agent-boilerplate**를 제출·승인받아 마켓플레이스에 올라가면:

1. Cursor **Settings → Plugins** (또는 **Marketplace**)에서 **"Browse Marketplace"** 클릭.
2. **"cursor-agent-boilerplate"** 또는 레포 소유자 이름 검색.
3. 필요한 **포지션** 플러그인(pm, design, frontend-mobile 등)을 각각 **Install** 하면 됩니다.

---

## 요약

| 방법 | 할 일 |
|------|--------|
| **1. Plugins에서 URL/로컬 추가** | Settings → Plugins에서 "Add plugin" / URL·경로 입력 가능한 곳에 레포 URL 또는 `.../cursor-agent-boilerplate/{포지션 폴더}` 경로 추가. |
| **2. 복사** | 쓸 포지션의 `.cursor/`(commands, skills, agents)를 대상 레포 `.cursor/`에 합쳐 넣기. rules는 기존 대상 레포 것 유지. |
| **3. 마켓플레이스** | 마켓플레이스에 등록된 후 Browse Marketplace에서 검색해 설치. |

**대상 레포**에 맞는 포지션(pm, design, frontend-mobile, frontend-web, backend 등)만 골라 위 세 가지 중 하나를 적용하면 됩니다.
