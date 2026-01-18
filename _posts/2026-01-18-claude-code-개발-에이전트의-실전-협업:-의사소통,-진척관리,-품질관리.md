---
title: "Claude Code 개발 에이전트의 실전 협업: 의사소통, 진척관리, 품질관리"
date: 2026-01-18 11:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,   ai-agent,  ralph-playbook,  software-engineering,  Claude.write]
---


## 목차
1. [서론: 단일 에이전트에서 에이전트 팀으로](#서론-단일-에이전트에서-에이전트-팀으로)
2. [에이전트 간 의사소통 (Communication)](#에이전트-간-의사소통-communication)
3. [진척 관리 (Progress Management)](#진척-관리-progress-management)
4. [품질 관리 (Quality Management)](#품질-관리-quality-management)
5. [Jira & Slack 통합: 실무 도구와의 연결](#jira--slack-통합-실무-도구와의-연결)
6. [Claude Code에서 Ralph 방법론 실행하기](#claude-code에서-ralph-방법론-실행하기)
7. [Multi-Agent 오케스트레이션: 대규모 협업](#multi-agent-오케스트레이션-대규모-협업)
8. [실전 워크플로우: 단계별 구현](#실전-워크플로우-단계별-구현)
9. [비용 관리와 최적화](#비용-관리와-최적화)
10. [결론: 에이전트 팀 운영의 현실](#결론-에이전트-팀-운영의-현실)

---

## 서론: 단일 에이전트에서 에이전트 팀으로

### 진화의 단계

Steve Yegge가 정의한 "프로그래머의 진화" 7단계:

```
Stage 1-2: 수동 사용
└─ Claude Code를 IDE처럼 사용

Stage 3-4: 싱글 에이전트 최적화
├─ CLAUDE.md로 컨텍스트 관리
├─ 스킬과 서브에이전트 활용
└─ 효율적인 프롬프팅

Stage 5: CLI, Multi-agent, YOLO
├─ 3-5개 병렬 인스턴스
├─ 매우 빠른 개발 속도
└─ Git worktrees 활용

Stage 6-7: 수작업 관리의 한계
├─ 10개 이상의 에이전트
├─ 수작업 조율의 어려움
└─ 오케스트레이션 필요성 인식

Stage 8: 오케스트레이터 구축
└─ Gas Town, Claude-Flow 등 사용
```

대부분의 개발자는 Stage 3-5에 있으며, 이 문서는 **Stage 5-7에서 효과적으로 작동하는 실전 패턴**에 초점을 맞춥니다.

### 핵심 질문들

이 문서가 답하는 질문들:

1. **의사소통**: 여러 Claude Code 인스턴스가 어떻게 서로 정보를 공유하는가?
2. **진척 관리**: 20개의 병렬 작업이 진행 중일 때 어떻게 추적하는가?
3. **품질 관리**: 자동화된 코드가 품질 기준을 어떻게 충족하는가?
4. **도구 통합**: Jira, Slack 같은 실무 도구와 어떻게 연결하는가?
5. **실전 적용**: Ralph 방법론을 Claude Code에서 구체적으로 어떻게 구현하는가?

---

## 에이전트 간 의사소통 (Communication)

### MCP (Model Context Protocol): 표준 통신 프로토콜

2026년 현재, 에이전트 간 의사소통의 표준은 **MCP (Model Context Protocol)**입니다. Anthropic가 개발한 이 프로토콜은 "AI를 위한 USB-C"로 비유됩니다.

#### MCP의 작동 원리

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP 통신 아키텍처                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Agent A (Claude Code #1)                                    │
│  ┌──────────────┐                                           │
│  │ MCP Client   │                                           │
│  └──────┬───────┘                                           │
│         │ JSON-RPC 2.0                                       │
│         ▼                                                    │
│  ┌──────────────┐         ┌──────────────┐                 │
│  │ MCP Server A │◄────────┤ Shared State │                 │
│  └──────┬───────┘         │  (Git/Files) │                 │
│         │                 └──────┬───────┘                 │
│         │                        │                          │
│         └────────────────────────┼─────────────┐            │
│                                  │             │            │
│  Agent B (Claude Code #2)        │             │            │
│  ┌──────────────┐                │             │            │
│  │ MCP Client   │                │             │            │
│  └──────┬───────┘                │             │            │
│         │ JSON-RPC 2.0            │             │            │
│         ▼                         ▼             ▼            │
│  ┌──────────────┐         ┌──────────────┐ ┌──────────┐   │
│  │ MCP Server B │◄────────┤ Message Bus  │ │  Tools   │   │
│  └──────────────┘         │  (Beads/Mail)│ │ (Jira 등)│   │
│                           └──────────────┘ └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### MCP 서버 설정 (Claude Code)

**1. 기본 설정 파일 (`~/.mcp.json` 또는 `.mcp.json`)**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    },
    "jira": {
      "command": "node",
      "args": ["/path/to/jira-mcp-server/dist/index.js"],
      "env": {
        "JIRA_HOST": "https://your-company.atlassian.net",
        "JIRA_EMAIL": "your-email@company.com",
        "JIRA_API_TOKEN": "your_jira_token"
      }
    },
    "agent-mail": {
      "command": "npx",
      "args": ["-y", "mcp-agent-mail"],
      "env": {
        "MAIL_DIR": "/path/to/shared/mail"
      }
    }
  }
}
```

**2. 프로젝트별 설정**

각 프로젝트 루트에 `.mcp.json` 생성:

```json
{
  "mcpServers": {
    "local-db": {
      "command": "node",
      "args": ["./mcp-servers/db-server.js"],
      "env": {
        "DB_PATH": "./project.db"
      }
    },
    "project-mail": {
      "command": "npx",
      "args": ["-y", "mcp-agent-mail"],
      "env": {
        "MAIL_DIR": "./.agent-mail"
      }
    }
  }
}
```

**3. MCP 서버 관리 명령어**

```bash
# MCP 서버 목록 확인
claude mcp list

# 특정 서버 상세 정보
claude mcp get github

# 서버 추가
claude mcp add my-server -- node server.js

# 서버 제거
claude mcp remove my-server

# Claude Code 내에서 상태 확인
/mcp
```

### Agent Mail & Beads: 에이전트 간 메시징

Jeffrey Emanuel이 개발한 **MCP Agent Mail**은 에이전트 간 비동기 메시징을 가능하게 합니다.

#### Agent Mail 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                  Agent Mail 시스템                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Agent A                    Mail Directory                   │
│  ┌──────────┐              ┌────────────────┐               │
│  │ Inbox    │◄─────────────│ /shared/mail/  │               │
│  │ Outbox   │─────────────►│ agents/        │               │
│  └──────────┘              │  ├─ agent-a/   │               │
│                            │  ├─ agent-b/   │               │
│  Agent B                    │  └─ agent-c/   │               │
│  ┌──────────┐              └────────────────┘               │
│  │ Inbox    │◄──────────┐                                   │
│  │ Outbox   │───────────┘                                   │
│  └──────────┘                                               │
│                                                              │
│  메시지 형식 (JSON):                                          │
│  {                                                           │
│    "from": "agent-a",                                        │
│    "to": "agent-b",                                          │
│    "subject": "Code review completed",                       │
│    "body": "Reviewed PR #123. Found 2 issues...",           │
│    "timestamp": "2026-01-18T10:00:00Z",                      │
│    "metadata": {                                             │
│      "pr_number": 123,                                       │
│      "priority": "high"                                      │
│    }                                                         │
│  }                                                           │
└─────────────────────────────────────────────────────────────┘
```

#### Agent Mail 실전 사용

**설치 및 설정**:

```bash
# Agent Mail MCP 서버 설치
npm install -g mcp-agent-mail

# 메일 디렉토리 생성
mkdir -p ~/agent-workspace/.agent-mail

# .mcp.json에 추가
cat >> .mcp.json << 'EOF'
{
  "mcpServers": {
    "agent-mail": {
      "command": "npx",
      "args": ["-y", "mcp-agent-mail"],
      "env": {
        "MAIL_DIR": "/absolute/path/to/.agent-mail",
        "AGENT_NAME": "main-agent"
      }
    }
  }
}
EOF
```

**사용 예시 (Claude Code 내)**:

```
개발자: "다른 에이전트에게 백엔드 API 작업이 완료되었다고 메시지를 보내줘"

Agent A: 
[agent-mail MCP 도구 사용]
메시지를 frontend-agent에게 발송했습니다:
- 제목: "Backend API endpoints ready"
- 내용: "POST /api/users 및 GET /api/users/:id 구현 완료. 
         API 문서는 docs/api.md 참조. 테스트 100% 통과."
```

```
# 다른 터미널에서 Frontend Agent 시작
cd frontend-workspace
claude --resume

[Agent Mail을 통해 자동으로 메시지 확인]
Frontend Agent:
새 메시지가 있습니다 (from: backend-agent):
"Backend API endpoints ready..."

이 정보를 바탕으로 프론트엔드 통합 작업을 시작하겠습니다.
```

### Beads: Git 기반 구조화 데이터 공유

Steve Yegge의 **Beads**는 Git + SQLite를 사용한 에이전트 메모리 시스템입니다.

#### Beads 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                    Beads 시스템                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  로컬 작업 (빠른 쿼리)                                         │
│  ┌──────────────────┐                                       │
│  │ SQLite Database  │                                       │
│  │ (.beads/beads.db)│                                       │
│  │                  │                                       │
│  │ - Issues         │                                       │
│  │ - Dependencies   │                                       │
│  │ - Comments       │                                       │
│  │ - Events         │                                       │
│  └────────┬─────────┘                                       │
│           │ auto-sync (5초 디바운스)                         │
│           ▼                                                  │
│  ┌──────────────────┐                                       │
│  │ JSONL 파일        │                                       │
│  │(.beads/issues.jsonl)                                     │
│  │                  │                                       │
│  │ {"id":1,"title":"Auth","status":"in-progress"...}       │
│  │ {"id":2,"title":"DB","status":"todo"...}                │
│  └────────┬─────────┘                                       │
│           │ git push/pull                                    │
│           ▼                                                  │
│  ┌──────────────────┐                                       │
│  │  Git Repository  │◄──────► 모든 에이전트가 공유           │
│  └──────────────────┘                                       │
└─────────────────────────────────────────────────────────────┘
```

#### Beads 실전 사용

**설치**:

```bash
# Beads CLI 설치
go install github.com/steveyegge/beads/cmd/bd@latest

# 프로젝트에서 Beads 초기화
cd your-project
bd init

# .gitignore 확인 (.beads/beads.db는 무시됨)
cat .gitignore | grep beads
# .beads/beads.db
```

**작업 생성 및 관리**:

```bash
# 이슈 생성
bd create "Implement user authentication" \
  --priority high \
  --labels backend,security

# 이슈 목록
bd list --status todo

# 이슈 할당
bd assign 123 --agent backend-agent

# 이슈 상태 업데이트
bd update 123 --status in-progress

# 의존성 추가
bd depend 124 --on 123  # 124는 123에 의존

# JSON 출력 (프로그래밍 방식)
bd list --json

# Git에 동기화
git add .beads/issues.jsonl
git commit -m "Update issue tracking"
git push
```

**Claude Code에서 Beads 사용**:

```
개발자: "Beads에서 내가 할당된 작업 목록을 보여줘"

Claude Code:
[bash_tool: bd list --assignee current-agent --json]

할당된 작업 3개:
1. [#123] Implement JWT authentication (in-progress)
2. [#145] Add password reset endpoint (todo)
3. [#156] Rate limiting middleware (todo)

#123부터 시작할까요?
```

### Git Worktrees: 물리적 격리

여러 에이전트가 동일한 저장소에서 작업할 때 **Git worktrees**로 격리합니다.

#### Git Worktrees 구조

```
프로젝트 구조:
my-project/
├── main/                    # 메인 작업 디렉토리
│   ├── src/
│   ├── tests/
│   └── .git/               # 실제 Git 저장소
│
├── worktrees/
│   ├── agent-a-feat-auth/  # Agent A 전용 worktree
│   │   ├── src/
│   │   └── .git            # 심볼릭 링크
│   │
│   ├── agent-b-feat-ui/    # Agent B 전용 worktree
│   │   ├── src/
│   │   └── .git            # 심볼릭 링크
│   │
│   └── agent-c-bugfix/     # Agent C 전용 worktree
│       ├── src/
│       └── .git            # 심볼릭 링크
```

#### Worktrees 실전 설정

**초기 설정**:

```bash
# 메인 프로젝트
cd ~/projects/my-app
git clone https://github.com/company/my-app.git
cd my-app

# Agent A를 위한 worktree (feature/auth 브랜치)
git worktree add ../worktrees/agent-a-auth -b feature/auth

# Agent B를 위한 worktree (feature/ui 브랜치)
git worktree add ../worktrees/agent-b-ui -b feature/ui

# Agent C를 위한 worktree (bugfix/login 브랜치)
git worktree add ../worktrees/agent-c-bugfix -b bugfix/login

# 현재 worktrees 확인
git worktree list
# /Users/dev/projects/my-app                    abc123 [main]
# /Users/dev/projects/worktrees/agent-a-auth    def456 [feature/auth]
# /Users/dev/projects/worktrees/agent-b-ui      ghi789 [feature/ui]
# /Users/dev/projects/worktrees/agent-c-bugfix  jkl012 [bugfix/login]
```

**각 에이전트 시작**:

```bash
# Terminal 1: Agent A
cd ~/projects/worktrees/agent-a-auth
claude --resume

# Terminal 2: Agent B  
cd ~/projects/worktrees/agent-b-ui
claude --resume

# Terminal 3: Agent C
cd ~/projects/worktrees/agent-c-bugfix
claude --resume
```

**tmux를 사용한 관리**:

```bash
# tmux 세션 생성
tmux new -s dev-agents

# 첫 번째 창: Agent A
cd ~/projects/worktrees/agent-a-auth
claude --resume

# 새 창 생성: Ctrl+b c
# 두 번째 창: Agent B
cd ~/projects/worktrees/agent-b-ui
claude --resume

# 새 창 생성: Ctrl+b c
# 세 번째 창: Agent C
cd ~/projects/worktrees/agent-c-bugfix
claude --resume

# 창 전환: Ctrl+b n (다음), Ctrl+b p (이전)
# 창 목록: Ctrl+b w
```

### Claude Code as MCP Server: 에이전트가 에이전트를 호출

Claude Code는 MCP 서버로도 작동하여 다른 에이전트가 호출할 수 있습니다.

#### 설정 방법

**Claude Code를 MCP 서버로 실행**:

```bash
# 한 번만 실행 (권한 수락)
claude --dangerously-skip-permissions

# MCP 서버 모드로 시작
claude mcp serve
```

**다른 Claude Code/Cursor에서 사용**:

```json
// ~/.mcp.json
{
  "mcpServers": {
    "claude-code-worker": {
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {
        "CLAUDE_SKIP_PERMISSIONS": "1"
      }
    }
  }
}
```

**사용 예시**:

```
Master Agent (Cursor):
"이 복잡한 리팩토링을 Claude Code 워커에게 위임해줘"

[claude-code-worker MCP 도구 사용]

Claude Code Worker가 리팩토링을 완료했습니다:
- 15개 파일 수정
- 모든 테스트 통과
- 커밋: "refactor: modernize authentication flow"
```

---

## 진척 관리 (Progress Management)

### 계층적 진척 추적 시스템

```
┌─────────────────────────────────────────────────────────────┐
│                진척 관리 계층                                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Level 1: 전략적 (일주일-월 단위)                             │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Jira / Linear                                    │       │
│  │ - Epic, User Stories                             │       │
│  │ - Sprint Planning                                │       │
│  │ - 비즈니스 가시성                                 │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼ 양방향 동기화                                       │
│  Level 2: 전술적 (일 단위)                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Beads / .beads/issues.jsonl                      │       │
│  │ - 세부 작업 분해                                  │       │
│  │ - 에이전트 할당                                   │       │
│  │ - 의존성 관리                                     │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼ 작업 배분                                          │
│  Level 3: 실행적 (시간 단위)                                  │
│  ┌──────────────────────────────────────────────────┐       │
│  │ progress.md / IMPLEMENTATION_PLAN.md             │       │
│  │ - 현재 작업 상태                                  │       │
│  │ - 다음 단계                                       │       │
│  │ - 블로커                                          │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼ 실시간 업데이트                                     │
│  Level 4: 검증적 (분 단위)                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │ Git Commits + CI/CD                              │       │
│  │ - 실제 코드 변경                                  │       │
│  │ - 테스트 결과                                     │       │
│  │ - 배포 상태                                       │       │
│  └──────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### Level 1: Jira 통합 (전략적 관리)

#### Jira MCP 서버 설정

**방법 1: Composio를 통한 설정 (가장 쉬움)**

```bash
# Composio Jira MCP 설치
npx @composio/mcp@latest setup \
  "https://mcp.composio.dev/partner/composio/jira/mcp?customerId=YOUR_ID" \
  "jira-integration" --client

# .mcp.json에 자동 추가됨
```

**방법 2: 직접 설정 (더 많은 제어)**

```bash
# Jira API 토큰 생성
# 1. https://id.atlassian.com/manage-profile/security 방문
# 2. "Create API token" 클릭
# 3. 토큰 복사

# MCP 서버 설치
npm install -g @modelcontextprotocol/server-jira

# .mcp.json 설정
cat >> ~/.mcp.json << 'EOF'
{
  "mcpServers": {
    "jira": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-jira"],
      "env": {
        "JIRA_HOST": "your-company.atlassian.net",
        "JIRA_EMAIL": "your-email@company.com",
        "JIRA_API_TOKEN": "your_api_token_here",
        "JIRA_DEFAULT_PROJECT": "PROJ"
      }
    }
  }
}
EOF
```

#### 실전 Jira 워크플로우

**시나리오: 새로운 기능 개발**

```
개발자 → Claude Code:
"Jira에서 ENG-4521 이슈를 가져와서 구현해줘"

Claude Code:
[jira MCP 도구 사용: get-issue]

ENG-4521: "사용자 프로필 이미지 업로드 기능 추가"

수용 기준:
- 이미지 크기 제한 5MB
- 지원 형식: JPEG, PNG, WebP
- S3에 업로드
- 썸네일 자동 생성 (200x200)

작업을 다음과 같이 분해합니다:

1. 파일 업로드 엔드포인트 생성
2. 이미지 검증 미들웨어
3. S3 업로드 서비스
4. 썸네일 생성 로직
5. 프론트엔드 업로드 UI

[구현 시작...]

[작업 완료 후]
[jira MCP 도구 사용: update-issue]
ENG-4521 상태를 "In Review"로 업데이트했습니다.
코멘트 추가: "구현 완료. PR #456 생성. 모든 테스트 통과."
```

**자동 진척 업데이트**:

```
# CLAUDE.md에 추가
## Jira Integration

When you complete a task:
1. Update the Jira issue status
2. Add a comment with:
   - PR link
   - Test results
   - Any blockers or notes

When you start a task:
1. Update status to "In Progress"
2. Add yourself as assignee
3. Log time if significant
```

### Level 2: Beads 시스템 (전술적 관리)

#### Beads 작업 분해 패턴

**Jira Epic → Beads 작업으로 변환**:

```bash
# Jira Epic: "User Authentication System"
# Beads로 세분화

bd create "Setup PostgreSQL user table" \
  --priority high \
  --labels database,backend \
  --epic "User Auth" \
  --estimate 2h

bd create "Implement bcrypt password hashing" \
  --priority high \
  --labels security,backend \
  --epic "User Auth" \
  --estimate 1h \
  --depends-on 1

bd create "Create JWT token generation" \
  --priority high \
  --labels auth,backend \
  --epic "User Auth" \
  --estimate 2h \
  --depends-on 2

bd create "Add login endpoint" \
  --priority high \
  --labels api,backend \
  --epic "User Auth" \
  --estimate 3h \
  --depends-on 3

bd create "Write integration tests" \
  --priority medium \
  --labels testing,backend \
  --epic "User Auth" \
  --estimate 2h \
  --depends-on 4

# 의존성 그래프 시각화
bd graph --epic "User Auth"
```

#### Gas Town Convoy 패턴

Gas Town의 **Convoy** 개념: 관련 작업들의 묶음

```bash
# Gas Town 사용 예시
gt convoy create "User Auth System" \
  issue-1 issue-2 issue-3 issue-4 issue-5 \
  --notify

# Convoy 상태 확인
gt convoy list
# CONVOY: User Auth System
# Status: In Progress (3/5 complete)
# - ✓ issue-1: Database setup (agent-a)
# - ✓ issue-2: Password hashing (agent-a)
# - ✓ issue-3: JWT generation (agent-b)
# - ⧗ issue-4: Login endpoint (agent-c)
# - ⭘ issue-5: Integration tests (unassigned)

# 특정 이슈를 에이전트에 할당
gt sling issue-5 agent-d
```

### Level 3: progress.md (실행적 관리)

#### progress.md 템플릿

각 에이전트 작업 디렉토리에:

```markdown
# Progress Tracking

**Agent**: backend-agent-1
**Task**: Implement user authentication endpoints
**Started**: 2026-01-18 10:00
**Last Updated**: 2026-01-18 14:30

## Current Status

Working on: Login endpoint implementation
Progress: 70% complete

## Completed Today

- [x] Created User model with Prisma schema
- [x] Implemented password hashing with bcrypt
- [x] Added JWT token generation utility
- [x] Created POST /api/auth/register endpoint
  - Email validation
  - Password strength check
  - Duplicate email prevention
- [x] 70% - Implementing POST /api/auth/login endpoint
  - Credential validation ✓
  - Token generation ✓
  - Response formatting (in progress)

## Next Steps

1. Complete login endpoint error handling
2. Add rate limiting middleware
3. Write unit tests for auth endpoints
4. Integration tests for full auth flow
5. Update API documentation

## Blockers

None currently

## Learnings

- bcrypt cost factor 10 is standard for production
- JWT expiration should match session requirements (30d in our case)
- Need to handle JWT_SECRET missing gracefully

## Metrics

- Files changed: 7
- Lines added: 342
- Lines removed: 18
- Tests written: 12
- Tests passing: 12/12 (100%)

## Communication

- Sent message to frontend-agent about API endpoints ready
- Waiting for UI design from design-agent for error states

## For Next Session

- Remember to add rate limiting (5 attempts per 15min)
- Don't forget to update Postman collection
- Check if password reset endpoint is in this sprint
```

#### 자동 progress.md 업데이트

```
# CLAUDE.md에 추가
## Progress Tracking

After every significant milestone (test passing, feature complete, etc.):
1. Update progress.md with what you accomplished
2. Note any new learnings or gotchas
3. Update the metrics section
4. List next steps

Format:
- Be concise but specific
- Include file names and line counts
- Note any architectural decisions
- Flag blockers immediately
```

### Level 4: Git Commits (검증적 관리)

#### 자동화된 커밋 패턴

```bash
# Git hooks로 자동 진척 업데이트
# .git/hooks/post-commit

#!/bin/bash

# 커밋 정보 추출
COMMIT_MSG=$(git log -1 --pretty=%B)
COMMIT_HASH=$(git log -1 --pretty=%h)
FILES_CHANGED=$(git diff --name-only HEAD~1 HEAD | wc -l)

# Beads 업데이트
if [[ $COMMIT_MSG =~ ^feat.*\#([0-9]+) ]]; then
  ISSUE_NUM="${BASH_REMATCH[1]}"
  bd update $ISSUE_NUM \
    --add-comment "Commit $COMMIT_HASH: $COMMIT_MSG" \
    --progress +10
fi

# progress.md 업데이트
echo "- [$(date +%H:%M)] Committed: $COMMIT_MSG ($FILES_CHANGED files)" \
  >> progress.md
```

#### 대시보드 생성

```bash
# 전체 진척 상황 대시보드
#!/bin/bash
# dashboard.sh

echo "=== Development Dashboard ==="
echo "Date: $(date)"
echo ""

echo "=== Jira Status ==="
# Jira 이슈 상태
claude --prompt "Show me the status of all issues in current sprint from Jira"

echo ""
echo "=== Beads Issues ==="
bd list --format table

echo ""
echo "=== Git Activity (Last 24h) ==="
git log --since="24 hours ago" --oneline --all

echo ""
echo "=== Active Agents ==="
# tmux 세션 확인
tmux list-sessions 2>/dev/null || echo "No active tmux sessions"

echo ""
echo "=== Recent Commits by Agent ==="
git log --since="24 hours ago" --format="%an: %s" | sort | uniq -c
```

**Cron으로 자동 실행**:

```bash
# 매 시간마다 대시보드 업데이트
0 * * * * /path/to/dashboard.sh > /path/to/project/DASHBOARD.md
```

---

## 품질 관리 (Quality Management)

### Multi-Layer Quality Gates

```
┌─────────────────────────────────────────────────────────────┐
│                    품질 관리 계층                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: 즉각적 피드백 (초 단위)                              │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • 컴파일 에러                                     │       │
│  │ • 타입 체크 (TypeScript, Python mypy 등)          │       │
│  │ • Linter (ESLint, Ruff, 등)                      │       │
│  │ • Formatter (Prettier, Black, 등)                │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼                                                    │
│  Layer 2: 자동 테스트 (분 단위)                               │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • Unit Tests                                     │       │
│  │ • Integration Tests                              │       │
│  │ • Property-Based Tests                           │       │
│  │ • Snapshot Tests                                 │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼                                                    │
│  Layer 3: 코드 리뷰 에이전트 (분-시간 단위)                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • AI 코드 리뷰어                                  │       │
│  │ • 보안 스캔                                       │       │
│  │ • 성능 분석                                       │       │
│  │ • 문서화 체크                                     │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼                                                    │
│  Layer 4: LLM-as-Judge (분-시간 단위)                         │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • 코드 품질 (가독성, 유지보수성)                   │       │
│  │ • 아키텍처 일관성                                 │       │
│  │ • UX/UI 품질                                      │       │
│  │ • 문서 품질                                       │       │
│  └──────────────────────────────────────────────────┘       │
│         ▼                                                    │
│  Layer 5: 인간 리뷰 (일 단위)                                 │
│  ┌──────────────────────────────────────────────────┐       │
│  │ • PR 최종 승인                                    │       │
│  │ • 아키텍처 결정 검토                              │       │
│  │ • 비즈니스 로직 검증                              │       │
│  └──────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### Layer 1-2: 자동화된 검증

#### AGENTS.md에 품질 기준 정의

~~~markdown
# AGENTS.md

## Quality Standards

### Mandatory Checks (Must Pass)

**Linting**:
```bash
npm run lint
# Must exit 0
```

**Type Checking**:
```bash
npm run type-check
# Must exit 0
```

**Unit Tests**:
```bash
npm test
# Must achieve 80%+ coverage
# All tests must pass
```

**Integration Tests**:
```bash
npm run test:integration
# All critical paths must pass
```

### Code Style

- Use TypeScript for all new files
- Prefer functional components (React)
- Use async/await over promises
- Maximum function length: 50 lines
- Maximum file length: 300 lines

### Testing Requirements

Every new feature must include:
1. Unit tests for all functions
2. Integration tests for API endpoints
3. E2E tests for user flows (if UI)
4. Error case testing

### Documentation

All functions must have:
- JSDoc comments
- Parameter descriptions
- Return type documentation
- Example usage (for public APIs)
```

#### Pre-commit Hooks

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-commit checks..."

# Lint
echo "  Linting..."
npm run lint || exit 1

# Type check
echo "  Type checking..."
npm run type-check || exit 1

# Run tests
echo "  Running tests..."
npm test || exit 1

# Check coverage
echo "  Checking coverage..."
COVERAGE=$(npm run test:coverage --silent | grep "Statements" | awk '{print $3}' | sed 's/%//')
if (( $(echo "$COVERAGE < 80" | bc -l) )); then
  echo "❌ Coverage below 80% ($COVERAGE%)"
  exit 1
fi

echo "✅ All checks passed!"
```
~~~

### Layer 3: AI 코드 리뷰어

#### 전용 리뷰 에이전트 설정

~~~yaml
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Specialized code review agent
tools:
  - Read
  - Grep
  - Bash
disallowedTools:
  - Write
  - Edit
permissionMode: bypassPermissions
---

You are a meticulous code reviewer. Your job is to:

1. Review code for:
   - Logic errors
   - Security vulnerabilities
   - Performance issues
   - Code style violations
   - Missing error handling
   - Unclear variable names

2. Check against project standards:
   - Read CLAUDE.md for project conventions
   - Verify tests are comprehensive
   - Ensure documentation is updated

3. Provide actionable feedback:
   - Specific line numbers
   - Suggested fixes
   - Severity levels (critical, major, minor)

4. Output format:
```markdown
## Code Review Report

### Summary
[Overall assessment]

### Critical Issues (Must Fix)
- [File:Line] Description
  Suggestion: [specific fix]

### Major Issues (Should Fix)
- [File:Line] Description

### Minor Issues (Nice to Have)
- [File:Line] Description

### Positive Notes
- [What was done well]
```
~~~

#### 리뷰 워크플로우

```bash
# 리뷰 스크립트
#!/bin/bash
# review.sh

BRANCH=$(git branch --show-current)
BASE_BRANCH=${1:-main}

echo "🔍 Starting code review for $BRANCH..."

# 변경된 파일 목록
CHANGED_FILES=$(git diff --name-only $BASE_BRANCH...$BRANCH)

# 리뷰 에이전트 호출
claude --agent code-reviewer --prompt "
Review the following changes from $BASE_BRANCH to $BRANCH:

Changed files:
$CHANGED_FILES

Please conduct a thorough code review focusing on:
1. Security vulnerabilities
2. Logic errors
3. Performance issues
4. Test coverage
5. Documentation completeness

Provide specific, actionable feedback.
"

# 결과를 파일로 저장
claude --agent code-reviewer --prompt "..." > .review/review-$(date +%Y%m%d-%H%M%S).md
```

#### GitHub Actions 통합

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Claude Code
        run: |
          npm install -g @anthropic-ai/claude-code
          
      - name: Run AI Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude --agent code-reviewer --prompt "
          Review PR #${{ github.event.pull_request.number }}
          Base: ${{ github.base_ref }}
          Head: ${{ github.head_ref }}
          
          Changed files:
          $(git diff --name-only origin/${{ github.base_ref }}...origin/${{ github.head_ref }})
          " > review-output.md

      - name: Post Review as Comment
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const reviewContent = fs.readFileSync('review-output.md', 'utf8');
            
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `## 🤖 AI Code Review\n\n${reviewContent}`
            });
```

### Layer 4: LLM-as-Judge

#### 주관적 품질 평가

```typescript
// src/lib/llm-judge.ts

import Anthropic from '@anthropic-ai/sdk';

interface JudgeCriteria {
  aspect: string;
  weight: number;
  passThreshold: number;
}

interface JudgmentResult {
  aspect: string;
  score: number; // 0-100
  passes: boolean;
  feedback: string;
}

export async function judgeCodeQuality(
  code: string,
  criteria: JudgeCriteria[]
): Promise<JudgmentResult[]> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const results: JudgmentResult[] = [];

  for (const criterion of criteria) {
    const response = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1000,
      messages: [{
        role: 'user',
        content: `
Evaluate the following code for ${criterion.aspect}.

Code:
\`\`\`
${code}
\`\`\`

Provide a score from 0-100 and specific feedback.

Respond ONLY with JSON:
{
  "score": <number 0-100>,
  "feedback": "<specific feedback>"
}
`
      }]
    });

    const content = response.content[0];
    if (content.type !== 'text') continue;

    const result = JSON.parse(content.text);
    results.push({
      aspect: criterion.aspect,
      score: result.score,
      passes: result.score >= criterion.passThreshold,
      feedback: result.feedback
    });
  }

  return results;
}

// 사용 예시
const criteria: JudgeCriteria[] = [
  {
    aspect: 'Readability',
    weight: 1.0,
    passThreshold: 70
  },
  {
    aspect: 'Maintainability',
    weight: 1.0,
    passThreshold: 70
  },
  {
    aspect: 'Performance Optimization',
    weight: 0.8,
    passThreshold: 60
  },
  {
    aspect: 'Error Handling',
    weight: 1.2,
    passThreshold: 80
  }
];

const code = await readFile('src/auth/login.ts', 'utf-8');
const judgments = await judgeCodeQuality(code, criteria);

const failed = judgments.filter(j => !j.passes);
if (failed.length > 0) {
  console.error('Quality check failed:');
  failed.forEach(j => {
    console.error(`- ${j.aspect}: ${j.score}/100`);
    console.error(`  ${j.feedback}`);
  });
  process.exit(1);
}
```

#### 테스트 통합

```typescript
// src/__tests__/quality-gate.test.ts

import { judgeCodeQuality } from '../lib/llm-judge';
import { readFile } from 'fs/promises';

describe('Quality Gates', () => {
  it('should pass readability check for auth module', async () => {
    const code = await readFile('src/auth/login.ts', 'utf-8');
    
    const results = await judgeCodeQuality(code, [{
      aspect: 'Readability',
      weight: 1.0,
      passThreshold: 70
    }]);

    expect(results[0].passes).toBe(true);
  }, 30000); // 30초 타임아웃

  it('should pass error handling check for API routes', async () => {
    const code = await readFile('src/routes/users.ts', 'utf-8');
    
    const results = await judgeCodeQuality(code, [{
      aspect: 'Error Handling',
      weight: 1.2,
      passThreshold: 80
    }]);

    expect(results[0].score).toBeGreaterThanOrEqual(80);
  }, 30000);
});
```

### 품질 대시보드

```typescript
// scripts/quality-dashboard.ts

import { execSync } from 'child_process';
import { judgeCodeQuality } from '../src/lib/llm-judge';
import { readdir, readFile } from 'fs/promises';
import { join } from 'path';

interface QualityMetrics {
  coverage: number;
  lintErrors: number;
  typeErrors: number;
  testsPassing: number;
  testsTotal: number;
  llmQualityScore: number;
}

async function generateQualityDashboard(): Promise<QualityMetrics> {
  // Coverage
  const coverageOutput = execSync('npm run test:coverage --silent').toString();
  const coverage = parseFloat(
    coverageOutput.match(/Statements.*?([\d.]+)%/)?.[1] || '0'
  );

  // Lint
  let lintErrors = 0;
  try {
    execSync('npm run lint --silent');
  } catch (e: any) {
    const output = e.stdout?.toString() || '';
    lintErrors = (output.match(/✖/g) || []).length;
  }

  // Type errors
  let typeErrors = 0;
  try {
    execSync('npm run type-check --silent');
  } catch (e: any) {
    const output = e.stdout?.toString() || '';
    typeErrors = (output.match(/error TS/g) || []).length;
  }

  // Tests
  const testOutput = execSync('npm test --silent').toString();
  const testsMatch = testOutput.match(/Tests:.*?(\d+) passed, (\d+) total/);
  const testsPassing = parseInt(testsMatch?.[1] || '0');
  const testsTotal = parseInt(testsMatch?.[2] || '0');

  // LLM Quality Score (샘플 파일들 평가)
  const srcFiles = await readdir('src', { recursive: true });
  const tsFiles = srcFiles.filter(f => f.endsWith('.ts'));
  
  let totalScore = 0;
  let fileCount = 0;

  for (const file of tsFiles.slice(0, 10)) { // 최대 10개 파일
    const code = await readFile(join('src', file), 'utf-8');
    const results = await judgeCodeQuality(code, [
      { aspect: 'Overall Quality', weight: 1.0, passThreshold: 70 }
    ]);
    totalScore += results[0].score;
    fileCount++;
  }

  return {
    coverage,
    lintErrors,
    typeErrors,
    testsPassing,
    testsTotal,
    llmQualityScore: fileCount > 0 ? totalScore / fileCount : 0
  };
}

// 실행
generateQualityDashboard().then(metrics => {
  console.log('📊 Quality Dashboard');
  console.log('===================');
  console.log(`Coverage: ${metrics.coverage}%`);
  console.log(`Lint Errors: ${metrics.lintErrors}`);
  console.log(`Type Errors: ${metrics.typeErrors}`);
  console.log(`Tests: ${metrics.testsPassing}/${metrics.testsTotal} passing`);
  console.log(`LLM Quality Score: ${metrics.llmQualityScore.toFixed(1)}/100`);
  
  // 실패 조건
  if (metrics.coverage < 80 ||
      metrics.lintErrors > 0 ||
      metrics.typeErrors > 0 ||
      metrics.testsPassing < metrics.testsTotal ||
      metrics.llmQualityScore < 70) {
    console.error('\n❌ Quality gates not met');
    process.exit(1);
  }
  
  console.log('\n✅ All quality gates passed');
});
```

---

## Jira & Slack 통합: 실무 도구와의 연결

### Jira 실전 통합

#### 완전한 워크플로우

**1. 이슈 생성 → Claude Code 작업 → PR 생성 → Jira 업데이트**

```
# Terminal 1: Main Developer
cd ~/projects/my-app

# Jira에서 이슈 확인
claude --prompt "Jira의 PROJ 프로젝트에서 우선순위가 높은 이슈 5개를 보여줘"

출력:
1. PROJ-123: 사용자 로그인 API 구현 (High, Backend)
2. PROJ-124: 프로필 이미지 업로드 (High, Backend)
3. PROJ-125: 비밀번호 재설정 기능 (High, Backend)
4. PROJ-126: 대시보드 UI 개선 (Medium, Frontend)
5. PROJ-127: 알림 시스템 구축 (Medium, Fullstack)

# 이슈 할당 및 작업 시작
claude --prompt "PROJ-123을 가져와서 구현해줘"

[Jira MCP를 통해 이슈 상세 정보 조회]
[이슈를 자신에게 할당]
[상태를 "In Progress"로 변경]

[구현 진행...]

[완료 후]
[GitHub PR 생성]
[Jira 이슈에 PR 링크 추가]
[상태를 "In Review"로 변경]
```

**2. CLAUDE.md에 Jira 워크플로우 정의**

~~~markdown
# CLAUDE.md

## Jira Integration Workflow

### Starting Work

When assigned a Jira issue:
1. Fetch issue details: "Get Jira issue [ISSUE-ID]"
2. Understand acceptance criteria
3. Break down into subtasks in progress.md
4. Update Jira status to "In Progress"
5. Add yourself as assignee

### During Work

- Log significant progress in Jira comments
- Update time tracking if enabled
- Flag blockers immediately in Jira

### Completing Work

1. Ensure all tests pass
2. Create PR with conventional commit format
3. Link PR in Jira issue
4. Update Jira status to "In Review"
5. Add summary comment:
   - What was implemented
   - Test coverage
   - Any architectural decisions
   - Migration steps (if any)

### Jira Comment Format

```
✅ Implementation Complete

**Changes**:
- Added POST /api/auth/login endpoint
- Implemented JWT token generation
- Added rate limiting (5 attempts/15min)

**Testing**:
- Unit tests: 15/15 passing
- Integration tests: 5/5 passing
- Coverage: 94%

**PR**: #456
**Commits**: 7 commits, 342 lines added

**Notes**:
- JWT secret must be configured in .env
- Database migration required for user table
```

### Jira Issue Transitions

Valid transitions:
- Todo → In Progress
- In Progress → In Review
- In Review → Done
- In Review → Todo (if rejected)

Use: `claude jira transition ISSUE-123 --to "In Review"`
~~~

#### 자동화 스크립트

```bash
# scripts/jira-workflow.sh

#!/bin/bash

ISSUE_ID=$1
ACTION=$2

case $ACTION in
  start)
    # 이슈를 In Progress로
    claude --prompt "Update Jira issue $ISSUE_ID status to 'In Progress' and assign to me"
    
    # Beads에 작업 생성
    bd create "Work on $ISSUE_ID" \
      --jira-id $ISSUE_ID \
      --priority high
    
    # Git 브랜치 생성
    git checkout -b feature/$ISSUE_ID
    ;;

  complete)
    # PR 생성
    PR_URL=$(gh pr create --title "feat: $ISSUE_ID implementation" --body "Closes $ISSUE_ID")
    
    # Jira 업데이트
    claude --prompt "
    Update Jira issue $ISSUE_ID:
    - Status: In Review
    - Add comment: Implementation complete. PR: $PR_URL
    - Link PR: $PR_URL
    "
    
    # Beads 업데이트
    bd update $(bd list --jira-id $ISSUE_ID --json | jq -r '.id') \
      --status done \
      --add-comment "PR created: $PR_URL"
    ;;

  *)
    echo "Usage: $0 ISSUE_ID {start|complete}"
    exit 1
    ;;
esac
```

### Slack 통합 (알림 및 협업)

#### Slack MCP 서버 설정

```bash
# Slack App 생성 (https://api.slack.com/apps)
# Bot Token Scopes:
# - chat:write
# - files:write
# - channels:read
# - users:read

# Slack MCP 설치
npm install -g @modelcontextprotocol/server-slack

# .mcp.json 설정
cat >> .mcp.json << 'EOF'
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-bot-token",
        "SLACK_DEFAULT_CHANNEL": "#dev-team"
      }
    }
  }
}
EOF
```

#### 자동 진척 알림

```typescript
// scripts/slack-notifier.ts

import { WebClient } from '@slack/web-api';

const slack = new WebClient(process.env.SLACK_BOT_TOKEN);

interface ProgressUpdate {
  agent: string;
  task: string;
  status: 'started' | 'completed' | 'blocked';
  details?: string;
}

export async function notifyProgress(update: ProgressUpdate) {
  const emoji = {
    started: '🚀',
    completed: '✅',
    blocked: '🚨'
  }[update.status];

  let message = `${emoji} **${update.agent}**: ${update.status} work on ${update.task}`;
  
  if (update.details) {
    message += `\n> ${update.details}`;
  }

  await slack.chat.postMessage({
    channel: '#dev-team',
    text: message,
    mrkdwn: true
  });
}

// Git hook에서 사용
// .git/hooks/post-commit
const commitMsg = execSync('git log -1 --pretty=%B').toString();
const agent = execSync('whoami').toString().trim();

if (commitMsg.includes('feat:') || commitMsg.includes('fix:')) {
  notifyProgress({
    agent,
    task: commitMsg.split(':')[1].trim(),
    status: 'completed',
    details: `Commit: ${execSync('git log -1 --pretty=%h').toString().trim()}`
  });
}
```

#### Claude Code에서 Slack 사용

```
개발자: "팀에게 백엔드 API가 준비되었다고 Slack으로 알려줘"

Claude Code:
[slack MCP 도구 사용]

#dev-team 채널에 메시지를 발송했습니다:

"🎉 Backend API Ready

Authentication endpoints are now available:
- POST /api/auth/register
- POST /api/auth/login
- POST /api/auth/logout

API documentation updated in docs/api.md
All tests passing (28/28)

Frontend team can start integration!"
```

#### 일일 스탠드업 자동화

```bash
# scripts/daily-standup.sh

#!/bin/bash

# 어제 완료된 작업
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
COMPLETED=$(git log --since="$YESTERDAY 00:00" --until="$YESTERDAY 23:59" \
  --format="%s" | grep -E "^(feat|fix):" | sed 's/^/- /')

# 오늘 계획된 작업
TODAY_TASKS=$(bd list --status todo --assignee me --format markdown)

# Slack에 포스트
claude --prompt "
Post to #standup channel:

**Daily Standup - $(date +%Y-%m-%d)**

**✅ Yesterday**:
$COMPLETED

**📋 Today**:
$TODAY_TASKS

**🚨 Blockers**: None
"

# Cron: 매일 아침 9시
# 0 9 * * * /path/to/daily-standup.sh
```

---

## Claude Code에서 Ralph 방법론 실행하기

### 구체적 구현 패턴

#### 1. 프로젝트 구조 설정

```
my-project/
├── .claude/
│   ├── agents/                    # 서브에이전트 정의
│   │   ├── planner.md
│   │   ├── builder.md
│   │   ├── reviewer.md
│   │   └── documenter.md
│   └── skills/                    # 커스텀 스킬
│       ├── ralph-loop/
│       └── quality-gate/
│
├── .mcp.json                      # MCP 서버 설정
│
├── specs/                         # 요구사항 명세
│   ├── authentication.md
│   ├── user-profile.md
│   └── file-upload.md
│
├── .agent-mail/                   # 에이전트 간 메시징
│   ├── main-agent/
│   ├── backend-agent/
│   └── frontend-agent/
│
├── .beads/                        # 작업 추적
│   ├── issues.jsonl
│   └── beads.db (gitignored)
│
├── progress.md                    # 현재 진척상황
├── IMPLEMENTATION_PLAN.md         # 실행 계획
├── AGENTS.md                      # 도구 및 규칙
├── CLAUDE.md                      # 프로젝트 컨텍스트
│
└── worktrees/                     # 병렬 작업공간
    ├── agent-a-auth/
    ├── agent-b-ui/
    └── agent-c-testing/
```

#### 2. CLAUDE.md 작성 (Ralph 최적화)

```markdown
# Project: Authentication System

## Overview
Enterprise-grade authentication system with JWT, OAuth, and MFA support.

## Tech Stack
- Backend: Node.js, Express, TypeScript
- Database: PostgreSQL with Prisma ORM
- Auth: JWT, bcrypt, OAuth 2.0
- Testing: Jest, Supertest

## Ralph Loop Configuration

### Phase 1: Requirements (specs/)
All features must have a spec in specs/ directory.
Each spec follows JTBD (Jobs to Be Done) format.

### Phase 2: Planning (IMPLEMENTATION_PLAN.md)
Before implementing, always:
1. Read the relevant spec
2. Check current code state
3. Identify gaps
4. Update IMPLEMENTATION_PLAN.md with next steps
5. Prioritize tasks by dependencies

### Phase 3: Building (The Loop)
For each task:
1. Implement the feature
2. Run tests: `npm test`
3. Run linter: `npm run lint`
4. Run type check: `npm run type-check`
5. If any check fails, analyze and fix
6. Repeat until all pass
7. Commit with conventional commits
8. Update progress.md

### Backpressure Mechanisms
**Required Checks** (must pass before commit):
- `npm run lint` → 0 errors
- `npm run type-check` → 0 errors
- `npm test` → 100% pass, 80%+ coverage
- `npm run test:integration` → all pass

**Optional Checks** (run but don't block):
- `npm run test:e2e` → manual verification
- Security audit (automated weekly)

## Code Standards
- TypeScript strict mode
- ESLint with Airbnb config
- Prettier for formatting
- Maximum function length: 50 lines
- Maximum file length: 300 lines
- Prefer pure functions
- Document all public APIs

## Testing Requirements
- Unit tests for all business logic
- Integration tests for all API endpoints
- E2E tests for critical user flows
- Minimum 80% coverage

## Git Workflow
- Branch naming: `feature/JIRA-ID-short-description`
- Commit format: `type(scope): message`
  - Types: feat, fix, docs, style, refactor, test, chore
- PR required for merge to main
- Squash commits on merge

## Agent Coordination
- Use Agent Mail for async communication
- Update Beads for task tracking
- Update progress.md after significant milestones
- Notify team on Slack for major completions

## Context Management
When context gets too full:
1. Commit current work
2. Update progress.md with:
   - What was completed
   - What's next
   - Any blockers
3. Use /clear to reset
4. Resume with fresh context
```

#### 3. 서브에이전트 정의

**Planner Agent (`.claude/agents/planner.md`)**:

~~~markdown
---
name: planner
description: Gap analysis and planning specialist
tools:
  - Read
  - Grep
  - Glob
  - Bash
disallowedTools:
  - Write
  - Edit
---

You are a planning specialist. Your job is to:

1. Read specs/ directory to understand requirements
2. Analyze current codebase state
3. Identify gaps between spec and implementation
4. Create prioritized task list in IMPLEMENTATION_PLAN.md
5. Identify dependencies between tasks

Process:
1. Read the spec file provided
2. Explore codebase to see what exists
3. List what's missing or needs change
4. Order tasks by dependencies (what must come first)
5. Estimate complexity for each task

Output Format (IMPLEMENTATION_PLAN.md):
```markdown
# Implementation Plan

**Spec**: [spec name]
**Generated**: [timestamp]

## Gap Analysis
- [x] Existing: User model with basic fields
- [ ] Missing: Password hashing
- [ ] Missing: JWT token generation
- [ ] Missing: Login endpoint
- [ ] Missing: Registration endpoint

## Tasks (Prioritized by Dependencies)

### High Priority
1. [ ] Implement password hashing utility
   - Location: src/utils/password.ts
   - Dependencies: None
   - Estimate: 1h

2. [ ] Implement JWT token utilities
   - Location: src/utils/jwt.ts
   - Dependencies: None
   - Estimate: 1h

### Medium Priority
3. [ ] Create user registration endpoint
   - Location: src/routes/auth.ts
   - Dependencies: Task 1
   - Estimate: 2h

4. [ ] Create login endpoint
   - Location: src/routes/auth.ts
   - Dependencies: Task 1, 2
   - Estimate: 2h

### Tests
5. [ ] Unit tests for password hashing
6. [ ] Unit tests for JWT
7. [ ] Integration tests for registration
8. [ ] Integration tests for login
```
~~~

**Builder Agent (`.claude/agents/builder.md`)**:

~~~markdown
---
name: builder
description: Implementation specialist
permissionMode: bypassPermissions
---

You are an implementation specialist. Your job is to:

1. Read IMPLEMENTATION_PLAN.md
2. Pick the highest priority unchecked task
3. Implement it following project standards (AGENTS.md)
4. Run all checks (tests, lint, type-check)
5. Fix any failures
6. Commit when all checks pass
7. Update progress.md

Process:
1. Check IMPLEMENTATION_PLAN.md for next task
2. If no plan exists, ask for /agent planner to run
3. Implement the task
4. Run: `npm run lint && npm run type-check && npm test`
5. If failures, analyze errors and fix
6. Repeat step 4-5 until all pass
7. Commit with message: "feat(scope): description"
8. Mark task as complete in IMPLEMENTATION_PLAN.md
9. Update progress.md

Never skip tests. Never commit without all checks passing.
~~~

**Reviewer Agent (`.claude/agents/reviewer.md`)**:

~~~markdown
---
name: reviewer
description: Code review specialist
tools:
  - Read
  - Grep
  - Bash
disallowedTools:
  - Write
  - Edit
---

You are a code review specialist. Your job is to:

1. Review code changes for:
   - Logic errors
   - Security vulnerabilities
   - Performance issues
   - Style violations
   - Missing tests
   - Missing documentation

2. Check against standards:
   - CLAUDE.md conventions
   - AGENTS.md rules
   - Test coverage requirements

3. Provide specific, actionable feedback

Process:
1. Get list of changed files: `git diff --name-only main...current-branch`
2. For each file:
   - Read the file
   - Check for issues
   - Verify tests exist
   - Check documentation
3. Run quality checks:
   - `npm run lint`
   - `npm run type-check`
   - `npm test`
4. Generate review report

Output Format:
```markdown
## Code Review

### Summary
[Overall assessment]

### Critical Issues 🚨
- [File:Line] [Description]
  Fix: [Specific suggestion]

### Major Issues ⚠️
- [File:Line] [Description]

### Minor Issues 💡
- [File:Line] [Description]

### Positive Notes ✅
- [What was done well]

### Recommendations
1. [Actionable suggestion]
2. [Actionable suggestion]
```
~~~

#### 4. Ralph Loop 스크립트

```bash
#!/bin/bash
# ralph-loop.sh

set -e

MAX_ITERATIONS=${1:-10}
SPEC_FILE=${2:-""}

echo "🔄 Starting Ralph Loop (max $MAX_ITERATIONS iterations)"

if [ -n "$SPEC_FILE" ]; then
  echo "📋 Spec: $SPEC_FILE"
fi

for i in $(seq 1 $MAX_ITERATIONS); do
  echo ""
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🔁 Iteration $i/$MAX_ITERATIONS"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  
  # Phase 1: Planning (if needed)
  if [ ! -f "IMPLEMENTATION_PLAN.md" ] || [ $i -eq 1 ]; then
    echo "📝 Phase 1: Planning"
    claude --agent planner --prompt "
    Create implementation plan for ${SPEC_FILE:-all specs}.
    Analyze current code state and identify gaps.
    "
  fi
  
  # Phase 2: Building
  echo "🔨 Phase 2: Building"
  claude --agent builder --prompt "
  Continue implementation from IMPLEMENTATION_PLAN.md.
  Pick next unchecked task and implement it.
  Run all checks and fix any failures.
  Commit when done.
  "
  
  # Check if all tasks complete
  REMAINING=$(grep -c "\[ \]" IMPLEMENTATION_PLAN.md || echo "0")
  
  if [ "$REMAINING" -eq "0" ]; then
    echo "✅ All tasks complete!"
    
    # Phase 3: Review
    echo "🔍 Phase 3: Review"
    claude --agent reviewer --prompt "
    Review all changes in this branch.
    Provide detailed feedback.
    "
    
    break
  fi
  
  echo "📊 Remaining tasks: $REMAINING"
  
  # Cooldown (to avoid rate limits)
  if [ $i -lt $MAX_ITERATIONS ]; then
    echo "⏳ Cooldown 10s..."
    sleep 10
  fi
done

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "🏁 Ralph Loop Complete"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

**사용 예시**:

```bash
# 특정 스펙으로 시작
./ralph-loop.sh 20 specs/authentication.md

# 모든 미완료 작업 처리
./ralph-loop.sh 50

# 단일 반복만
./ralph-loop.sh 1
```

---

## Multi-Agent 오케스트레이션: 대규모 협업

### Gas Town 아키텍처

Steve Yegge의 Gas Town은 20-30개의 에이전트를 조율하는 시스템입니다.

#### Gas Town 핵심 개념

```
┌─────────────────────────────────────────────────────────────┐
│                  Gas Town 구조                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  🏙️ The Town (~/:gt)                                         │
│  └─ 모든 프로젝트의 HQ                                         │
│                                                              │
│  🏗️ Rigs (프로젝트 컨테이너)                                  │
│  ├─ ~/gt/my-app/                                            │
│  ├─ ~/gt/another-project/                                    │
│  └─ Each rig = Git repo + agents + state                    │
│                                                              │
│  👔 The Mayor (조율자)                                        │
│  └─ 전체 워크스페이스를 이해하는 메인 에이전트                   │
│     "데이터베이스 마이그레이션을 처리해줘"                       │
│     → Convoy 생성, 작업 분배, 진행 모니터링                     │
│                                                              │
│  🚚 Convoys (작업 묶음)                                       │
│  └─ 관련된 이슈들의 그룹                                       │
│     Convoy: "Auth System"                                    │
│     ├─ issue-1: DB setup                                    │
│     ├─ issue-2: Password hashing                            │
│     └─ issue-3: JWT tokens                                  │
│                                                              │
│  🔨 Polecats (워커 에이전트)                                  │
│  └─ 병렬로 실행되는 작업 에이전트들                             │
│     각자 Git worktree, Beads, 독립 실행                       │
│                                                              │
│  👁️ Witness & Deacon (모니터링)                              │
│  ├─ Witness: 시스템 상태 감시                                 │
│  └─ Deacon: 문제 발생 시 복구                                │
│                                                              │
│  ⚙️ Refinery (병합 관리자)                                    │
│  └─ PR 검토 및 자동 병합                                       │
│     테스트 통과 시 자동으로 main에 병합                         │
│                                                              │
│  🪝 Hooks (에이전트별 상태)                                   │
│  └─ 각 에이전트의 영구 저장소                                  │
│     작업 히스토리, 메모, 워크플로우                             │
│                                                              │
│  💎 Beads (작업 데이터)                                       │
│  └─ Git-tracked JSONL                                        │
│     모든 이슈, 이벤트, 메일 저장                               │
└─────────────────────────────────────────────────────────────┘
```

#### Gas Town 설치 및 시작

```bash
# Gas Town 설치
go install github.com/steveyegge/gastown/cmd/gt@latest

# PATH 추가
echo 'export PATH="$PATH:$HOME/go/bin"' >> ~/.zshrc
source ~/.zshrc

# Town 초기화
gt install ~/gt --git
cd ~/gt

# 첫 번째 프로젝트 추가
gt rig add my-app https://github.com/company/my-app.git

# Crew 생성 (개인 작업공간)
gt crew add your-name --rig my-app
cd my-app/crew/your-name

# Mayor 시작 (메인 인터페이스)
gt mayor attach
```

#### Mayor와 대화하기

```
# Mayor 세션 안에서

Mayor> 안녕, 사용자 인증 시스템을 구축하고 싶어

Mayor: 알겠습니다. Auth 시스템을 위한 Convoy를 만들겠습니다.

[Convoy 생성: "User Authentication"]
[Beads 이슈 생성]
  - #1: Database schema for users
  - #2: Password hashing utility
  - #3: JWT token generation
  - #4: Registration endpoint
  - #5: Login endpoint
  - #6: Integration tests

[작업 배분]
  - Polecat-1 → #1 (DB schema)
  - Polecat-2 → #2 (Password hashing)
  - Polecat-3 → #3 (JWT)

작업이 시작되었습니다. 진행 상황을 확인하려면:
  gt convoy list
```

```bash
# 다른 터미널에서 진행 상황 확인
gt convoy list

CONVOY: User Authentication
Status: In Progress (2/6 complete)
─────────────────────────────────────────
✅ #1: Database schema (polecat-1) - Done
✅ #2: Password hashing (polecat-2) - Done  
⧗  #3: JWT generation (polecat-3) - In Progress
⭘  #4: Registration endpoint (unassigned)
⭘  #5: Login endpoint (unassigned)
⭘  #6: Integration tests (unassigned)
```

### Claude-Flow: 엔터프라이즈 오케스트레이션

Claude-Flow v3는 175+ 도구를 제공하는 엔터프라이즈급 오케스트레이터입니다.

#### Claude-Flow 특징

```
┌─────────────────────────────────────────────────────────────┐
│                Claude-Flow v3 아키텍처                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  🐝 Coordinated Agent Teams (Hive Mind)                      │
│  ├─ Hierarchical: Queen → Workers                           │
│  ├─ Mesh: Peer-to-peer collaboration                        │
│  └─ Automatic spawn & coordination                          │
│                                                              │
│  🧠 Self-Learning System                                     │
│  ├─ Successful patterns stored                              │
│  ├─ Routing optimization over time                          │
│  └─ Catastrophic forgetting prevention                      │
│                                                              │
│  🔌 Multi-LLM Support                                        │
│  ├─ Claude, GPT, Gemini, Cohere                             │
│  ├─ Local models (Llama)                                    │
│  ├─ Automatic failover                                      │
│  └─ Cost optimization routing                               │
│                                                              │
│  ⚡ MCP Native Integration                                   │
│  └─ Plug into Claude Code directly                          │
│                                                              │
│  🔒 Production Security                                      │
│  ├─ Prompt injection protection                             │
│  ├─ Input validation                                        │
│  ├─ Path traversal prevention                               │
│  └─ Safe credential handling                                │
└─────────────────────────────────────────────────────────────┘
```

#### Claude-Flow 설치 및 사용

```bash
# 설치
npm install -g claude-flow@v3alpha

# 초기화
npx claude-flow@v3alpha init

# MCP 서버로 시작
npx claude-flow@v3alpha mcp start

# Claude Code와 통합
claude mcp add claude-flow -- npx -y claude-flow@v3alpha

# Hive Mind 초기화
npx claude-flow hive-mind init

# 작업 스폰
npx claude-flow hive-mind spawn "Build authentication API"

# 상태 확인
npx claude-flow hive-mind status
npx claude-flow hive-mind metrics
```

#### Claude-Flow with Claude Code

```
개발자 → Claude Code:
"claude-flow를 사용해서 전체 인증 시스템을 구축해줘"

Claude Code:
[claude-flow MCP 도구 사용]

Hive Mind 생성 중...

Queen Agent 초기화:
- 작업 분석
- 아키텍처 설계
- 워커 할당 계획

Worker Agents 스폰 (5개):
- Worker-1: Database schema design
- Worker-2: Backend API implementation
- Worker-3: Frontend integration
- Worker-4: Test suite creation
- Worker-5: Documentation

작업 진행 중... (실시간 모니터링)

[30분 후]

✅ 모든 작업 완료
- 15개 파일 생성
- 342개 테스트 통과
- 문서화 완료
- PR #789 생성

비용: $24.50 (멀티 모델 최적화)
```

### DAKB: Distributed Agent Knowledge Base

여러 기계, 여러 LLM에 걸쳐 에이전트들이 지식을 공유하는 시스템입니다.

#### DAKB 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│           DAKB (Distributed Agent Knowledge Base)            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Clients (Any Machine, Any LLM)                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │Claude    │ │   GPT    │ │  Gemini  │ │Local LLM │       │
│  │Code #1   │ │  Agent   │ │  Agent   │ │  Agent   │       │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘       │
│       │            │            │            │              │
│       └────────────┴────────────┴────────────┘              │
│                           │                                 │
│                           ▼                                 │
│               ┌───────────────────────┐                     │
│               │   MCP / REST / SDK    │                     │
│               └───────────┬───────────┘                     │
│                           │                                 │
│                           ▼                                 │
│               ┌───────────────────────┐                     │
│               │   Gateway (FastAPI)   │                     │
│               └───────────┬───────────┘                     │
│                           │                                 │
│        ┌──────────────────┼──────────────────┐              │
│        ▼                  ▼                  ▼              │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐         │
│  │ MongoDB  │      │ Embedding│      │ Message  │         │
│  │Knowledge │      │ Service  │      │   Bus    │         │
│  └──────────┘      └──────────┘      └──────────┘         │
└─────────────────────────────────────────────────────────────┘
```

#### DAKB 설치 및 사용

```bash
# 설치
pip install dakb-server dakb-client

# 초기화 (secrets 생성)
dakb-server init

# 서비스 시작
dakb-server start

# 상태 확인
curl http://localhost:3100/health

# Claude Code에 MCP 서버 추가
# .mcp.json
{
  "mcpServers": {
    "dakb": {
      "command": "python",
      "args": ["-m", "dakb.mcp"],
      "env": {
        "DAKB_AUTH_TOKEN": "your-generated-token",
        "DAKB_GATEWAY_URL": "http://localhost:3100",
        "DAKB_PROFILE": "standard"
      }
    }
  }
}
```

#### DAKB 사용 시나리오

**시나리오 1: 크로스 플랫폼 지식 공유**

```
# Machine 1: Claude Code
개발자: "인증 API 구현 중 bcrypt cost factor를 10으로 결정했어. 
       이 정보를 knowledge base에 저장해줘"

Claude Code:
[DAKB MCP 도구 사용: store-knowledge]
저장 완료:
- Topic: "bcrypt configuration"
- Content: "Cost factor: 10 for production use"
- Tags: ["security", "authentication", "best-practice"]
- Visibility: Team-wide
```

```
# Machine 2: GPT Agent (다른 개발자의 컴퓨터)
개발자: "우리 프로젝트에서 bcrypt 설정이 어떻게 되어있지?"

GPT Agent:
[DAKB API 쿼리]
검색 결과:
- bcrypt cost factor: 10 (production standard)
- Stored by: claude-code-agent
- Date: 2026-01-18
- Source: Authentication implementation

이 설정을 적용하시겠습니까?
```

**시나리오 2: 연구 발견 공유**

```
Claude Code Agent A:
"API rate limiting 연구 중 발견:
express-rate-limit보다 redis-rate-limit가 
분산 환경에서 30% 더 효율적"

[DAKB에 저장: "Rate Limiting Research"]

... 다음 날 ...

Claude Code Agent B:
"rate limiting 구현이 필요한데 추천사항이 있나?"

[DAKB 검색 결과]
"어제 Agent A가 발견: redis-rate-limit 사용 권장
분산 환경에서 30% 효율 향상..."
```

---

## 실전 워크플로우: 단계별 구현

### Workflow 1: 소규모 팀 (3-5명, 에이전트 5-10개)

#### 구성

```
팀 구성:
- 개발자 3명
- 각 개발자당 2-3개 Claude Code 인스턴스

도구 스택:
- Git worktrees (병렬 작업)
- Agent Mail (에이전트 간 통신)
- Beads (작업 추적)
- Jira (외부 가시성)
- Slack (팀 협업)
```

#### 일일 워크플로우

**아침 (9:00-9:30)**:

```bash
# 1. Daily standup (자동화)
./scripts/daily-standup.sh
# → Slack #standup에 포스트

# 2. Jira 동기화
claude --prompt "오늘 할 Jira 이슈들을 Beads로 가져와줘"

# 3. 작업 할당
bd list --status todo | while read issue; do
  # 의존성 체크
  DEPS=$(bd show $issue --json | jq -r '.dependencies[]')
  if [ -z "$DEPS" ]; then
    # 병렬 작업 가능
    gt sling $issue agent-$(( RANDOM % 3 + 1 ))
  fi
done
```

**오전 작업 (9:30-12:00)**:

```bash
# tmux 세션 시작
tmux new -s dev-morning

# 창 1: Main Agent (본인)
cd ~/projects/main
claude --resume

# 창 2: Agent A (Backend)
cd ~/projects/worktrees/backend-auth
claude --resume --prompt "Implement authentication endpoints"

# 창 3: Agent B (Frontend)
cd ~/projects/worktrees/frontend-ui
claude --resume --prompt "Create login UI components"

# 창 4: Monitoring
watch -n 30 'bd list --format table && echo "---" && git log --oneline --since="today"'
```

**점심 전 체크 (11:50)**:

```bash
# 진행 상황 확인
./scripts/progress-check.sh

# 출력:
# ✅ Backend Auth: 2/3 tasks complete
# ⧗  Frontend UI: 1/2 tasks in progress
# ⭘  Integration: 0/1 tasks pending (blocked by Backend)
#
# Estimated completion: Today 15:00
```

**오후 작업 (13:00-17:00)**:

```bash
# 병합 및 통합
# Agent C: Integration Testing
cd ~/projects/worktrees/integration-tests
claude --prompt "
Backend auth API 완성됨.
Frontend UI 완성됨.
통합 테스트 작성하고 실행해줘.
"

# 자동 PR 생성 (테스트 통과 시)
# .github/workflows/auto-pr.yml 이 처리
```

**하루 마무리 (17:00-17:30)**:

```bash
# 1. 모든 에이전트 커밋 상태 확인
git worktree list | while read worktree; do
  cd $worktree
  if ! git diff-index --quiet HEAD --; then
    echo "⚠️  Uncommitted changes in $worktree"
  fi
done

# 2. 진행 상황 Jira 업데이트
./scripts/sync-to-jira.sh

# 3. 내일 계획 생성
claude --prompt "
오늘 완료된 작업을 바탕으로
내일의 IMPLEMENTATION_PLAN.md 업데이트해줘
"

# 4. 팀 공유
./scripts/daily-summary.sh | slack-cli post -c #team-updates
```

### Workflow 2: 중규모 팀 (10-20명, 에이전트 30-50개)

#### 구성

```
팀 구성:
- Backend 팀: 5명 → 15개 에이전트
- Frontend 팀: 5명 → 15개 에이전트  
- QA 팀: 3명 → 10개 에이전트
- DevOps: 2명 → 5개 에이전트

오케스트레이션:
- Gas Town (작업 조율)
- DAKB (지식 공유)
- Jira (프로젝트 관리)
- Slack (실시간 협업)
- GitHub Actions (CI/CD)
```

#### 주간 워크플로우

**월요일: Sprint Planning**

```bash
# 1. Jira Epic을 Gas Town Convoy로 변환
claude --prompt "
Jira Sprint 23의 모든 Epic을 가져와서
각 Epic을 Gas Town Convoy로 생성해줘
"

# Mayor에게 지시
gt mayor attach

Mayor> Sprint 23 시작. 5개 Epic을 처리해줘

Mayor:
[Convoy 생성]
- "User Management" (8 issues)
- "Payment Integration" (12 issues)
- "Analytics Dashboard" (6 issues)
- "Performance Optimization" (10 issues)
- "Security Hardening" (5 issues)

[팀별 할당]
- Backend team → Payment + Security
- Frontend team → User Management + Dashboard
- QA team → All (테스트 작성)
- DevOps → Performance

[Polecats 스폰: 35개]

작업이 시작되었습니다.
```

**화-목: 개발 진행**

```bash
# 실시간 모니터링 대시보드
gt monitor

# 출력 (실시간 업데이트):
┌────────────────────────────────────────────────────┐
│ Gas Town Dashboard - Sprint 23                      │
├────────────────────────────────────────────────────┤
│ Active Convoys: 5                                  │
│ Active Polecats: 28/35                             │
│ Completed Tasks: 23/41 (56%)                       │
│                                                     │
│ Convoy Status:                                     │
│ ✅ User Management: 8/8 (Complete)                 │
│ ⧗  Payment Integration: 7/12 (In Progress)        │
│ ⧗  Analytics Dashboard: 3/6 (In Progress)         │
│ ⧗  Performance Optimization: 4/10 (In Progress)   │
│ ⭘  Security Hardening: 1/5 (Blocked - dep)        │
│                                                     │
│ Recent Activity:                                   │
│ [14:23] polecat-12: Completed payment-webhook     │
│ [14:19] polecat-07: Tests passing (dashboard-api) │
│ [14:15] polecat-23: PR #456 merged                │
│                                                     │
│ Issues:                                            │
│ ⚠️  polecat-18: High memory usage (investigating) │
│ 🚨 polecat-31: Test failures (auth integration)   │
└────────────────────────────────────────────────────┘
```

**자동 이슈 대응**:

```
# Deacon (모니터링 에이전트)가 감지
[14:15] polecat-31 test failures detected

# Witness (복구 에이전트) 활성화
Witness:
1. Test 실패 분석
2. 관련 에이전트에 메일 발송
3. 필요 시 작업 재할당

[Agent Mail to polecat-31]
From: witness
Subject: Test failures in auth integration
Body: 
Integration tests failing:
- test_oauth_flow: timeout
- test_jwt_refresh: 401 error

Root cause analysis:
- JWT_SECRET env var not set in test environment

Suggested fix:
- Update .env.test with JWT_SECRET

작업 재시작할까요? (auto-retry in 2 min)
```

**금요일: Sprint Review & Retrospective**

```bash
# 주간 리포트 자동 생성
./scripts/weekly-report.sh

출력:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Sprint 23 Summary (2026-01-13 ~ 2026-01-17)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Completed: 38/41 tasks (93%)
⧗  In Progress: 3 tasks
⭘  Blocked: 0 tasks

Velocity: 38 points (target: 35) 📈

By Team:
- Backend: 15/15 ✓
- Frontend: 12/13 (92%)
- QA: 10/10 ✓
- DevOps: 1/3 (33%) ⚠️

Code Metrics:
- Commits: 247
- PRs merged: 68
- Tests added: 156
- Code coverage: 87% (↑ 3%)

Top Contributors (agents):
1. polecat-07 (backend): 14 tasks
2. polecat-12 (frontend): 12 tasks
3. polecat-23 (qa): 10 tasks

Issues:
- DevOps velocity lower than expected
  Action: Review polecat-30 config

Cost:
- API calls: $487 (budget: $500) ✓
- Avg cost/task: $12.82

Next Sprint:
- Carry over 3 tasks
- New capacity: 38 points
```

---

## 비용 관리와 최적화

### 비용 구조 이해

```
┌─────────────────────────────────────────────────────────────┐
│                  AI 개발 비용 분해                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Claude Pro Max: $100-200/월 (무제한이 아님!)                 │
│  ├─ 주간 사용 한도 존재                                        │
│  ├─ 한도 초과 시 추가 구독 필요                                │
│  └─ 3개 계정 = $600/월 가능                                   │
│                                                              │
│  Claude API (사용량 기반):                                     │
│  ├─ Opus 4.5: $15 / 1M input, $75 / 1M output               │
│  ├─ Sonnet 4.5: $3 / 1M input, $15 / 1M output              │
│  └─ Haiku 4.5: $0.25 / 1M input, $1.25 / 1M output          │
│                                                              │
│  Gas Town 실제 비용 (사례):                                    │
│  ├─ 1시간 작업: $100                                          │
│  ├─ 하루 작업: $200-300                                       │
│  └─ 주간 작업: $1000-2000                                     │
│                                                              │
│  절약 전략:                                                   │
│  ├─ Prompt Caching (90% 비용 절감)                           │
│  ├─ 모델 믹스 (Opus/Sonnet/Haiku)                            │
│  ├─ 배치 작업                                                 │
│  └─ 작업 범위 제한                                            │
└─────────────────────────────────────────────────────────────┘
```

### 비용 최적화 전략

#### 1. Prompt Caching 활용

```typescript
// 캐시 활용 예시
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  max_tokens: 1024,
  system: [
    {
      type: 'text',
      text: 'You are a code review expert...',
      cache_control: { type: 'ephemeral' } // 캐시됨!
    },
    {
      type: 'text',
      text: readFileSync('CLAUDE.md', 'utf-8'),
      cache_control: { type: 'ephemeral' } // 이것도 캐시!
    }
  ],
  messages: [{ role: 'user', content: prompt }]
});

// 비용 확인
console.log('Cache stats:');
console.log('- Read from cache:', response.usage.cache_read_input_tokens);
console.log('- New tokens:', response.usage.input_tokens);
console.log('- Savings:', 
  (response.usage.cache_read_input_tokens / 
   (response.usage.cache_read_input_tokens + response.usage.input_tokens) * 100
  ).toFixed(1) + '%'
);
```

**실제 절감 사례**:

```
케이스 1: 코드 리뷰 (CLAUDE.md 20KB)
- 캐시 없음: $0.15/요청
- 캐시 사용: $0.015/요청
- 절감: 90%

케이스 2: 반복 작업 (specs/ 50KB)
- 10번 반복
- 캐시 없음: $1.50
- 캐시 사용: $0.24
- 절감: 84%
```

#### 2. 모델 믹스 전략

```python
# 작업 종류별 모델 선택
MODEL_STRATEGY = {
    'planning': 'opus-4.5',      # 복잡한 추론 필요
    'implementation': 'sonnet-4.5', # 밸런스
    'testing': 'haiku-4.5',      # 반복 작업
    'documentation': 'haiku-4.5', # 단순 작업
    'review': 'sonnet-4.5',      # 품질 중요
}

def select_model(task_type: str) -> str:
    return MODEL_STRATEGY.get(task_type, 'sonnet-4.5')

# AGENTS.md에 정의
"""
## Model Selection

- Planning & Architecture: Opus 4.5
- Code Implementation: Sonnet 4.5
- Test Writing: Haiku 4.5
- Documentation: Haiku 4.5
- Code Review: Sonnet 4.5
"""
```

#### 3. 배치 작업으로 묶기

```bash
# 개별 실행 (비효율)
for file in src/*.ts; do
  claude --prompt "Review $file"
done
# 비용: $5-10

# 배치 실행 (효율)
FILES=$(find src -name "*.ts")
claude --prompt "
Review all these files at once:
$FILES

Provide a single comprehensive report.
"
# 비용: $2-3
```

#### 4. 비용 모니터링 대시보드

```typescript
// scripts/cost-tracking.ts

interface UsageStats {
  date: string;
  agent: string;
  model: string;
  inputTokens: number;
  outputTokens: number;
  cacheReadTokens: number;
  cost: number;
}

const PRICING = {
  'opus-4.5': { input: 15, output: 75, cacheRead: 1.5 },
  'sonnet-4.5': { input: 3, output: 15, cacheRead: 0.3 },
  'haiku-4.5': { input: 0.25, output: 1.25, cacheRead: 0.025 },
};

function calculateCost(stats: UsageStats): number {
  const pricing = PRICING[stats.model];
  return (
    (stats.inputTokens / 1_000_000 * pricing.input) +
    (stats.outputTokens / 1_000_000 * pricing.output) +
    (stats.cacheReadTokens / 1_000_000 * pricing.cacheRead)
  );
}

// 일일 리포트
async function dailyCostReport() {
  const stats = await getUsageStats(today());
  const totalCost = stats.reduce((sum, s) => sum + calculateCost(s), 0);
  
  console.log('💰 Daily Cost Report');
  console.log('===================');
  console.log(`Total: $${totalCost.toFixed(2)}`);
  console.log('\nBy Agent:');
  
  const byAgent = groupBy(stats, 'agent');
  for (const [agent, agentStats] of Object.entries(byAgent)) {
    const cost = agentStats.reduce((sum, s) => sum + calculateCost(s), 0);
    console.log(`  ${agent}: $${cost.toFixed(2)}`);
  }
  
  console.log('\nBy Model:');
  const byModel = groupBy(stats, 'model');
  for (const [model, modelStats] of Object.entries(byModel)) {
    const cost = modelStats.reduce((sum, s) => sum + calculateCost(s), 0);
    console.log(`  ${model}: $${cost.toFixed(2)}`);
  }
  
  // 경고
  if (totalCost > DAILY_BUDGET) {
    console.log(`\n⚠️  Over budget! ($${DAILY_BUDGET}/day)`);
  }
}
```

---

## 결론: 에이전트 팀 운영의 현실

### 현재 상태 (2026년 1월)

**작동하는 것**:

1. **단일 에이전트 워크플로우** (Stage 3-4)
   - Claude Code + CLAUDE.md
   - 서브에이전트 활용
   - MCP 서버 통합
   - 비용 대비 효과: ⭐⭐⭐⭐⭐

2. **소규모 병렬 (3-5 에이전트)** (Stage 5)
   - Git worktrees
   - Agent Mail
   - Beads 작업 추적
   - 비용 대비 효과: ⭐⭐⭐⭐

3. **도구 통합**
   - Jira MCP (실용적)
   - GitHub MCP (안정적)
   - Slack 알림 (유용)
   - 비용 대비 효과: ⭐⭐⭐⭐

**실험 단계**:

1. **중규모 병렬 (10-20 에이전트)** (Stage 6-7)
   - 수작업 조율 필요
   - 비용 증가 ($500-1000/week)
   - 복잡성 관리 어려움
   - 비용 대비 효과: ⭐⭐⭐

2. **Gas Town** (Stage 8)
   - 2주된 프로젝트
   - "혼돈" 단계
   - $100/시간 비용
   - 대부분의 팀에게 비추천
   - 비용 대비 효과: ⭐⭐ (현재)

### 실용적 권장사항

**Stage 3-4 개발자 (대부분):**

```markdown
추천 스택:
✅ Claude Code (단일 인스턴스)
✅ CLAUDE.md (프로젝트 컨텍스트)
✅ 서브에이전트 (전문 작업)
✅ Jira MCP (선택)
✅ GitHub MCP
❌ Gas Town (너무 이름)
❌ 10+ 에이전트 병렬

비용: $100-200/월
효과: 생산성 2-3x 향상
```

**Stage 5 개발자 (탐험가):**

```markdown
추천 스택:
✅ 3-5 병렬 Claude Code
✅ Git worktrees
✅ Agent Mail (에이전트 간 통신)
✅ Beads (작업 추적)
✅ Jira + Slack 통합
⚠️  Gas Town (관찰만)
❌ 대규모 자동화

비용: $300-500/월
효과: 생산성 3-5x 향상
주의: 관리 오버헤드 증가
```

**Stage 6-7+ 개발자 (개척자):**

```markdown
실험 스택:
✅ 10+ 에이전트
✅ DAKB (지식 공유)
✅ Claude-Flow (오케스트레이션)
⚠️  Gas Town (신중하게)
⚠️  커스텀 도구 개발

비용: $1000-2000/월
효과: 생산성 5-10x 가능
주의: 
- 높은 복잡성
- 예측 불가능한 비용
- 디버깅 어려움
- 아직 "실험" 단계
```

### 마지막 조언

**성공의 핵심은 단순함입니다**:

1. **작게 시작**: 단일 에이전트 + 좋은 CLAUDE.md
2. **점진적 확장**: 필요할 때만 복잡도 추가
3. **비용 추적**: 매주 비용 검토
4. **효과 측정**: 실제 생산성 향상 확인
5. **팀 학습**: 경험 공유 및 베스트 프랙티스 문서화

**Ralph 방법론의 진정한 가치**:

복잡한 오케스트레이션이 아닌:
- 명확한 요구사항 (specs/)
- 자동화된 검증 (tests, linters)
- 압력 기반 품질 (backpressure)
- 상태 관리 (Git, progress.md)

이것들이 실제로 작동하는 부분입니다.

**"Let Ralph Ralph"의 실제 의미**:

에이전트를 믿되, 시스템을 설계하라.
복잡도를 추가하되, 가치를 측정하라.
자동화하되, 제어를 유지하라.

---

**작성 일자**: 2026-01-18
