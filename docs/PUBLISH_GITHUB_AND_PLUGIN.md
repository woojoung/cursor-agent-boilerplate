# GitHub에 올리고 Cursor 플러그인으로 등록하기

> **목적**: 이 보일러플레이트 레포를 **GitHub(chae3229@gmail.com 계정)** 에 올리고, **Cursor 플러그인(마켓플레이스)** 으로 등록하는 절차를 정리합니다.

---

## 1. GitHub에 레포 올리기

### 1.1 GitHub에서 새 레포 만들기

1. [GitHub](https://github.com) 로그인 (chae3229@gmail.com 계정).
2. **New repository** 클릭.
3. **Repository name**: 예) `cursor-agent-boilerplate` (원하는 이름으로).
4. **Public** 선택. (Cursor 플러그인은 오픈소스여야 마켓플레이스 등록 가능.)
5. **Create repository** 클릭.  
   → 생성 후 나오는 **레포 URL**을 복사 (예: `https://github.com/chae3229/cursor-agent-boilerplate`).

### 1.2 로컬에서 push

보일러플레이트 폴더가 있는 위치에서:

```bash
cd /path/to/cursor-agent-boilerplate

# 아직 git이 아니면
git init
git add .
git commit -m "Initial: cursor-agent-boilerplate (rules, commands, skills, agents, docs)"

# GitHub 레포 연결 (위에서 만든 URL로 변경)
git remote add origin https://github.com/chae3229/cursor-agent-boilerplate.git

# 기본 브랜치 이름이 main이 아니면
git branch -M main

# 올리기
git push -u origin main
```

이미 다른 remote(예: wooree)가 있으면:

```bash
git remote -v
# origin을 새 GitHub 레포로 바꾸거나, 새 이름으로 추가
git remote set-url origin https://github.com/chae3229/cursor-agent-boilerplate.git
# 또는
git remote add github https://github.com/chae3229/cursor-agent-boilerplate.git
git push -u github main
```

---

## 2. 플러그인 manifest에 repository·author 반영

GitHub 레포 URL이 정해지면, 각 포지션의 **plugin.json** 에 `repository` 와 `author` 를 넣어 두는 것이 좋습니다.

| 파일 | 수정 예시 |
|------|-----------|
| `backend/.cursor-plugin/plugin.json` | `"repository": "https://github.com/chae3229/cursor-agent-boilerplate", "author": { "name": "chae3229", "email": "chae3229@gmail.com" }` |
| `frontend-web/.cursor-plugin/plugin.json` | 동일 repository, author |
| `pm/`, `design/`, `qa/`, `infra/`, `frontend-mobile/` | 각각 동일 |

레포 루트에 단일 plugin이 있는 구조가 아니므로, **포지션 폴더마다** `.cursor-plugin/plugin.json` 이 있고, **마켓플레이스에는 포지션별로 플러그인을 따로 등록**하는 방식이 됩니다 (아래 §3 참고).

---

## 3. Cursor 플러그인(마켓플레이스) 등록

- Cursor 플러그인은 **수동 심사**로 등록됩니다.  
  [Cursor Marketplace - Publish](https://cursor.com/marketplace/publish) 에서 제출합니다.
- 이 레포는 **포지션별로 플러그인이 여러 개**입니다.  
  `backend/`, `frontend-web/`, `frontend-mobile/`, `qa/`, `infra/`, `pm/`, `design/` 각각에 `.cursor-plugin/plugin.json` 이 있으므로, **플러그인 7개**로 볼 수 있습니다.

### 3.1 등록 방식 선택

| 방식 | 설명 |
|------|------|
| **한 레포, 여러 플러그인** | GitHub에는 레포 하나만 올리고, 마켓플레이스 제출 시 "이 레포의 backend 폴더가 백엔드용 플러그인, frontend-web 폴더가 프론트 웹용 플러그인"처럼 **경로별로** 여러 플러그인을 설명해서 제출. (제출 폼이 경로를 받는지에 따라 가능 여부 다름.) |
| **포지션별로 따로 제출** | backend만 쓰는 플러그인, frontend-web만 쓰는 플러그인 … 식으로 **7번 제출**. 각 제출 시 "소스는 같은 GitHub 레포의 backend(또는 해당 폴더) 경로"라고 안내. |

Cursor 마켓플레이스 제출 페이지에서 **플러그인 소스로 GitHub 레포 URL** 또는 **레포 + 하위 경로**를 넣는 방식이면, 한 레포로 7개 플러그인을 등록할 수 있습니다.  
제출 화면이 **한 URL = 한 플러그인**만 받는다면, 포지션별로 7번 제출하고 각각 해당 폴더 경로(또는 그 폴더만 있는 별도 레포)를 안내하면 됩니다.

### 3.2 제출 전 확인

- [Building Plugins \| Cursor Docs](https://cursor.com/docs/plugins/building) 에서 **plugin.json** 필수 항목 확인.
- **name**: 소문자, kebab-case (예: `cursor-agent-boilerplate-backend`).
- **오픈소스**: 마켓플레이스 플러그인은 오픈소스여야 합니다. GitHub 레포를 Public으로 두면 됩니다.

### 3.3 제출 절차

1. [https://cursor.com/marketplace/publish](https://cursor.com/marketplace/publish) 접속.
2. 로그인 후 제출 폼에 다음을 채움:
   - **플러그인 소스**: GitHub 레포 URL (및 필요 시 하위 경로, 예: `.../cursor-agent-boilerplate/tree/main/backend`).
   - **이름·설명**: 해당 포지션의 `plugin.json` 의 name, description 과 맞춤.
   - **저자·연락처**: chae3229@gmail.com 등.
3. 제출 후 Cursor 팀 **수동 심사**를 거쳐 마켓플레이스에 노출됩니다.

---

## 4. 요약

| 단계 | 할 일 |
|------|--------|
| 1 | GitHub에 새 레포 생성 (Public), URL 복사. |
| 2 | 로컬에서 `git remote add` / `git push` 로 코드 푸시. |
| 3 | 각 `.cursor-plugin/plugin.json` 에 `repository`, `author` (chae3229@gmail.com) 반영 후 필요 시 커밋·푸시. |
| 4 | [Cursor Marketplace - Publish](https://cursor.com/marketplace/publish) 에서 플러그인 제출 (포지션별 1개씩 또는 레포+경로로 여러 개). |

**에이전트로 부를 수 있는지**에 대한 설명은 [IS_THIS_AN_AGENT.md](IS_THIS_AN_AGENT.md) 에 정리해 두었습니다.
