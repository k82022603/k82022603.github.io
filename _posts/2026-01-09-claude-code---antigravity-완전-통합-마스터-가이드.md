---
title: "Claude Code + Antigravity 완전 통합 마스터 가이드"
date: 2026-01-09 23:30:00 +0900
categories: [AI,  Vibe Coding]
mermaid: [True]
tags: [AI,  claude-code,  google-antigravity,  vibe-coding,  coding-agents,  Developer,  Claude.write]
---


**2026년 최고의 AI 개발 환경 구축 완전판**

---

## 목차

1. [환경 선택 가이드](#1부-환경-선택-가이드)
2. [설치 및 초기 설정](#2부-설치-및-초기-설정)
3. [Extension 통합](#3부-extension-통합)
4. [Rules & Workflows 설정](#4부-rules--workflows-설정)
5. [프로젝트 구조 구축](#5부-프로젝트-구조-구축)
6. [통합 워크플로우](#6부-통합-워크플로우)
7. [실전 예제](#7부-실전-예제)
8. [문제 해결](#8부-문제-해결)

---

# 1부: 환경 선택 가이드

## 핵심 요약

```
Claude Code (CLI 도구)
→ WSL ⭐⭐⭐⭐⭐ (강력 권장!)
   - 네이티브 Linux 성능
   - 개발 도구 완벽 통합
   - 1.5~2배 빠른 속도

Antigravity (GUI IDE)
→ Windows ⭐⭐⭐⭐⭐ (강력 권장!)
   - 안정적인 GUI
   - Browser Agent 완벽
   - WSL 프로젝트 접근 가능

최적 조합: Claude Code (WSL) + Antigravity (Windows)
```

## 성능 비교 (실제 벤치마크)

### TypeScript 빌드

| 환경 | 시간 | 차이 |
|------|------|------|
| WSL (`~/projects`) | 3.2초 | **최고** ⭐ |
| Windows | 5.1초 | 1.6배 느림 |
| WSL (`/mnt/c/`) | 12.5초 | 3.9배 느림 |

### npm install

| 환경 | 시간 | 차이 |
|------|------|------|
| WSL (`~/projects`) | 8.3초 | **최고** ⭐ |
| Windows | 15.7초 | 1.9배 느림 |
| WSL (`/mnt/c/`) | 45.2초 | 5.4배 느림 |

### Docker build

| 환경 | 시간 | 차이 |
|------|------|------|
| WSL + Docker Desktop | 22초 | **최고** ⭐ |
| Windows + Docker Desktop | 28초 | 1.3배 느림 |

## 권장 설정 다이어그램

```
┌─────────────────────────────────────────────────────┐
│          Your Development Environment               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  [WSL Ubuntu]                  [Windows]           │
│  ┌─────────────────┐          ┌──────────────────┐ │
│  │ Claude Code CLI │◄────────►│  Antigravity GUI │ │
│  │  - 터미널 작업   │          │  - UI 작업        │ │
│  │  - API 구현      │          │  - 컴포넌트 생성   │ │
│  │  - 테스트        │          │  - Browser 테스트 │ │
│  └─────────────────┘          └──────────────────┘ │
│         ↓                             ↓            │
│  ~/projects/my-app  ◄═══════►  \\wsl$\Ubuntu\...   │
│  (동일한 프로젝트!)                                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

# 2부: 설치 및 초기 설정

## Step 1: WSL 설치 (Windows 사용자)

### WSL 활성화

```powershell
# PowerShell (관리자 권한)
wsl --install

# 재부팅 후
wsl --set-default-version 2
```

### Ubuntu 설치

```powershell
# Microsoft Store에서 "Ubuntu" 검색 후 설치
# 또는 명령어로
wsl --install -d Ubuntu
```

### 초기 설정

```bash
# WSL Ubuntu 첫 실행 시
# 사용자명: claude (또는 원하는 이름)
# 비밀번호 설정

# 시스템 업데이트
sudo apt update
sudo apt upgrade -y
```

## Step 2: Claude Code 설치 (WSL)

### Node.js 설치

```bash
# Node.js 20 설치
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 확인
node --version  # v20.x.x
npm --version   # 10.x.x
```

### Claude Code 설치

```bash
# npm으로 설치
npm install -g @anthropic/claude-code

# 또는 공식 설치 스크립트
curl -fsSL https://claude.ai/install.sh | sh

# 확인
claude --version
```

### 로그인

```bash
# 로그인 시작
claude auth login

# 브라우저가 열리면 Claude.ai 계정으로 로그인
# (Pro 계정 필요: $20/월)

# 로그인 확인
claude auth whoami
```

## Step 3: Antigravity 설치 (Windows)

### 다운로드

1. Windows 브라우저에서 https://antigravity.google/download 접속
2. "Download for Windows" 클릭 (약 300MB)
3. `antigravity-setup.exe` 다운로드

### 설치

```
1. antigravity-setup.exe 실행
2. 설치 경로 선택 (기본값 권장)
3. "Install" 클릭
4. 설치 완료 (3-5분 소요)
```

### 로그인

```
1. Antigravity 실행
2. "Sign in with Google" 클릭
3. 개인 Gmail 계정으로 로그인 (필수)
   ⚠️ Google Workspace 계정 안 됨
4. 권한 승인
5. Terminal 권한 설정: "Always ask" 선택 (권장)
6. 모델 선택: Gemini 3 Pro (기본값)
```

## Step 4: WSL ↔ Windows 연결 확인

### Windows에서 WSL 폴더 접근

```
Windows 탐색기 열기
주소창에 입력: \\wsl$\Ubuntu
→ home > claude > projects 확인
```

### WSL에서 Windows 탐색기 열기

```bash
cd ~/projects
explorer.exe .
# → Windows 탐색기가 WSL 폴더를 열음
```

## Step 5: 프로젝트 디렉토리 생성

```bash
# WSL에서
mkdir -p ~/projects
cd ~/projects

# 테스트 프로젝트
mkdir test-app
cd test-app
npm init -y

# Git 초기화
git init
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

---

# 3부: Extension 통합

## Antigravity에 Claude Code Extension 설치

### 왜 Extension이 필요한가?

```
Extension 없이:
- Antigravity 내장 Claude (무료, 제한적)
- Claude Code CLI (WSL, 유료)

Extension 설치 시:
✅ Antigravity 안에서 Claude 직접 사용
✅ GUI Chat 인터페이스
✅ / 슬래시 명령어
✅ 상단 Claude 아이콘
```

### 설치 방법

**Step 1: Extensions 패널 열기**

```
Antigravity 실행
→ 왼쪽 사이드바 "Extensions" 아이콘 클릭
   (또는 Ctrl+Shift+X)
```

**Step 2: 검색 및 설치**

```
1. 검색창에 "Claude" 입력
2. "Claude Code for VS Code" 찾기
   Publisher: Anthropic
   
3. "Install" 버튼 클릭
4. 설치 진행 (220MB+, 시간 소요)
5. "Reload Required" → 클릭
```

**Step 3: 확인**

```
✅ 상단 도구 모음에 Claude 아이콘 (φ)
✅ 왼쪽 사이드바에 Claude 패널
✅ 클릭 시 Chat 패널 열림
```

### Extension 로그인

```
1. Claude 아이콘 클릭
2. Chat 패널에 입력: /login
3. 브라우저 열림 → Claude.ai 로그인
4. "Authorize" 클릭
5. Antigravity로 복귀
```

### 슬래시 명령어

```
/login        - 로그인
/logout       - 로그아웃
/model        - 모델 변경 (sonnet/opus/haiku)
/init         - 프로젝트 분석 및 CLAUDE.md 생성
/clear        - 대화 기록 지우기
/settings     - 설정
/help         - 도움말
```

### Extension vs CLI vs 내장 비교

| 기능 | Extension | CLI (WSL) | 내장 |
|------|-----------|-----------|------|
| **위치** | Antigravity 내부 | 터미널 | Antigravity |
| **비용** | $20/월 | $20/월 | 무료 (제한) |
| **UI** | GUI Chat | CLI | Manager View |
| **성능** | 중간 | **최고** | 중간 |
| **MCP** | 많음 | 많음 | 제한적 |
| **할당량** | 높음 | 높음 | 낮음 |

**권장 조합**:
1. Claude Code CLI (WSL) - 메인 작업 ⭐
2. Antigravity 내장 - 무료 작업
3. Extension - 편의성 (선택)

---

# 4부: Rules & Workflows 설정

## Rules와 Workflows의 차이

```
Rules (규칙)
├─ 위치: .antigravity/rules.md (프로젝트)
│         ~/.gemini/GEMINI.md (전역)
├─ 역할: AI의 "헌법"
├─ 적용: 항상 활성 (자동)
└─ 용도: 코딩 스타일, 아키텍처, 보안

Workflows (워크플로우)
├─ 위치: .agent/workflows/*.md
├─ 역할: AI의 "작업 매뉴얼"
├─ 적용: / 명령으로 호출 (수동)
└─ 용도: 반복 작업 자동화
```

## 프로젝트 구조

```
your-project/
├── CLAUDE.md                    # Claude Code 규칙
├── .antigravity/
│   └── rules.md                 # Antigravity 규칙
├── .claude/                     # Claude Code 설정
│   ├── settings.json
│   ├── commands/                # 슬래시 커맨드
│   │   ├── sync-rules.md
│   │   ├── simplify.md
│   │   └── commit-push-pr.md
│   └── agents/                  # 서브 에이전트
│       ├── code-simplifier.md
│       └── verify-app.md
├── .agent/                      # Antigravity 워크플로우
│   └── workflows/
│       ├── create-component.md
│       ├── create-endpoint.md
│       ├── generate-tests.md
│       └── deploy.md
├── scripts/
│   └── sync-rules.sh            # 규칙 동기화
└── src/
```

## 공통 규칙 마스터 파일

**`_shared_rules.md`** (프로젝트 루트):

```markdown
# 프로젝트 개발 규칙 (공통)

## 코드 스타일
- 언어: TypeScript 필수
- 명명 규칙:
  - 변수/함수: camelCase
  - 컴포넌트/클래스: PascalCase
  - 상수: UPPER_SNAKE_CASE
  - 파일명: kebab-case.ts
- 들여쓰기: 스페이스 2칸
- 최대 길이:
  - 함수: 50줄
  - 파일: 300줄
  - 라인: 100자

## 아키텍처
- Clean Architecture 준수
- 레이어 구조:
  Domain (비즈니스) → Application (유스케이스) 
  → Infrastructure (외부) → Presentation (UI)
- Dependency Inversion Principle (DIP)
- Circular Dependency 절대 금지

## 테스트
- 프레임워크: Jest
- 커버리지: 80% 이상
- 구조: Given-When-Then
- 필수:
  - 비즈니스 로직: 유닛 테스트
  - API: 통합 테스트
  - 핵심 플로우: E2E 테스트

## Git 워크플로우
- 브랜치: feature/*, bugfix/*, hotfix/*
- 커밋: Conventional Commits
  - feat: 새 기능
  - fix: 버그 수정
  - refactor: 리팩토링
  - test: 테스트
  - docs: 문서
- PR: 300줄 이하 권장

## 보안
- 환경 변수 사용 (.env)
- 입력 검증 및 sanitize
- SQL Injection 방지 (ORM/Prepared)
- XSS 방지 (이스케이핑)
- CSRF 토큰
- Rate Limiting
- HTTPS 강제

## 성능
- 불필요한 연산 제거
- 적절한 캐싱
- DB 쿼리 최적화 (N+1 회피)
- 이미지 최적화, Lazy Loading
- Code Splitting
- 번들 크기 모니터링

## 금지 사항
- any 타입 금지
- console.log 프로덕션 금지
- 긴 함수/파일 금지
- God Object 금지
- Magic Number 금지
```

## 동기화 스크립트

**`scripts/sync-rules.sh`**:

```bash
#!/bin/bash

GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}=== Rules Synchronization ===${NC}\n"

# CLAUDE.md 생성
echo -e "${BLUE}Generating CLAUDE.md...${NC}"
cat > CLAUDE.md << 'EOF'
# 프로젝트 개발 규칙

**이 파일은 자동 생성됩니다.**
**수정: _shared_rules.md 편집 → ./scripts/sync-rules.sh 실행**

---

EOF

cat _shared_rules.md >> CLAUDE.md

cat >> CLAUDE.md << 'EOF'

---

## Claude Code 전용 설정

### 사용 모델
- 복잡한 작업: Opus 4.5
- 일반 작업: Sonnet 4.5
- 간단한 작업: Haiku 4.5

### 권장 워크플로우
1. 요구사항 분석
2. /plan 모드로 계획
3. Plan 검토 및 승인
4. 구현
5. code-simplifier로 정리
6. /commit-push-pr로 PR

### MCP 서버
- Playwright: 브라우저 자동화
- Filesystem: 파일 작업
- Git: 버전 관리
EOF

echo -e "${GREEN}✓ CLAUDE.md generated${NC}"

# .antigravity/rules.md 생성
echo -e "${BLUE}Generating .antigravity/rules.md...${NC}"
mkdir -p .antigravity

cat > .antigravity/rules.md << 'EOF'
# 프로젝트 개발 규칙

**이 파일은 자동 생성됩니다.**
**수정: _shared_rules.md 편집 → ./scripts/sync-rules.sh 실행**

---

EOF

cat _shared_rules.md >> .antigravity/rules.md

cat >> .antigravity/rules.md << 'EOF'

---

## Antigravity 전용 설정

### 사용 모델
- 연구/계획: Gemini 3 Pro
- 구현: Claude Opus 4.5
- 빠른 작업: Gemini 3 Flash

### 권장 워크플로우
1. Manager View에서 분석
2. Task List 생성
3. Implementation Plan 검토
4. 구현 진행
5. Browser Agent로 테스트
6. Walkthrough 확인
7. 변경사항 커밋

### Workflows 활용
- /create-component: 컴포넌트 생성
- /create-endpoint: API 생성
- /generate-tests: 테스트 생성
- /deploy: 배포 준비
EOF

echo -e "${GREEN}✓ .antigravity/rules.md generated${NC}"

echo -e "\n${GREEN}=== Synchronization Complete ===${NC}"
echo -e "Files updated:"
echo -e "  - CLAUDE.md"
echo -e "  - .antigravity/rules.md"
```

**권한 설정 및 실행**:

```bash
chmod +x scripts/sync-rules.sh
./scripts/sync-rules.sh
```

## Git Hook 자동화

**`.git/hooks/pre-commit`**:

```bash
#!/bin/bash

if git diff --cached --name-only | grep -q "_shared_rules.md"; then
    echo "Detected changes in _shared_rules.md"
    echo "Running sync-rules.sh..."
    ./scripts/sync-rules.sh
    git add CLAUDE.md .antigravity/rules.md
    echo "✓ Rules synchronized and staged"
fi

exit 0
```

```bash
chmod +x .git/hooks/pre-commit
```

## Antigravity Workflows

### 1. React 컴포넌트 생성

**`.agent/workflows/create-component.md`**:

~~~markdown
---
description: Create a new React component with tests and styles
---

1. Ask the user for the component name (e.g., "UserProfile")

2. Create directory structure:
```
src/components/[ComponentName]/
├── index.tsx
├── [ComponentName].module.css
├── [ComponentName].test.tsx
└── [ComponentName].stories.tsx
```

3. Generate `index.tsx`:
```typescript
import React from 'react';
import styles from './[ComponentName].module.css';

interface [ComponentName]Props {
  className?: string;
  children?: React.ReactNode;
}

export const [ComponentName]: React.FC<[ComponentName]Props> = ({
  className,
  children
}) => {
  return (
    <div className={`${styles.container} ${className || ''}`}>
      {children}
    </div>
  );
};
```

4. Generate CSS Module with responsive design

5. Generate comprehensive tests (unit + integration)

6. Generate Storybook story

7. Update exports in `src/components/index.ts`

8. Display success message with next steps
~~~

### 2. API 엔드포인트 생성

**`.agent/workflows/create-endpoint.md`**:

```markdown
---
description: Create a new API endpoint with validation and tests
---

1. Ask user:
   - Resource name
   - HTTP method
   - Route path

2. Create files:
   - src/routes/[resource].route.ts
   - src/controllers/[resource].controller.ts
   - src/services/[resource].service.ts
   - src/schemas/[resource].schema.ts
   - src/routes/[resource].route.test.ts

3. Implement:
   - Route with authentication & validation
   - Controller with error handling
   - Service with business logic
   - Joi validation schema
   - Integration tests (200, 401, 400 cases)

4. Update main app.ts

5. Display success message
```

### 3. 테스트 생성

**`.agent/workflows/generate-tests.md`**:

```markdown
---
description: Generate comprehensive tests for existing code
---

1. Ask: "Which file needs tests?"

2. Analyze file:
   - Exported functions/classes
   - Input/output types
   - Edge cases
   - Dependencies to mock

3. Generate test structure:
   - Setup/teardown
   - Happy path tests
   - Edge cases (empty, null, undefined)
   - Error cases
   - Boundary conditions

4. Target: 80%+ coverage

5. Display report:
   - Test cases created
   - Coverage estimate
   - Run command
```

### 4. 배포

**`.agent/workflows/deploy.md`**:

```markdown
---
description: Prepare and deploy application
---

1. Ask: "Which environment? (dev/staging/prod)"

2. Pre-deployment checks:
// turbo
3. Run: npm run typecheck

// turbo
4. Run: npm run lint

// turbo
5. Run: npm test

// turbo
6. Run: npm run build:[env]

7. If prod: Require "CONFIRM" input

8. Create git tag: v[version]-[env]

9. Deploy based on environment:
   - dev: docker-compose
   - staging: kubectl staging
   - prod: kubectl prod

10. Health check

11. Smoke tests

12. Display report + rollback command
```

## Claude Code Commands

### 1. 규칙 동기화

**`.claude/commands/sync-rules.md`**:

~~~markdown
# Sync Rules

Synchronizes shared rules to both files.

```bash
./scripts/sync-rules.sh
```
~~~

### 2. 코드 정리

**`.claude/commands/simplify.md`**:

~~~markdown
# Simplify Code

Runs code-simplifier on changed files.

```bash
CHANGED_FILES=$(git diff --name-only HEAD)
if [ -n "$CHANGED_FILES" ]; then
  claude -a code-simplifier "Simplify: $CHANGED_FILES"
fi
```
~~~

### 3. Commit & PR

**`.claude/commands/commit-push-pr.md`**:

~~~markdown
# Commit, Push, and Create PR

Complete workflow.

1. Ask: "What is this change about?"

2. Stage all changes
```bash
git add .
```

3. Commit with conventional format
```bash
git commit -m "[type]: [description]"
```

4. Push
```bash
git push origin HEAD
```

5. Create PR
```bash
gh pr create --title "[type]: [description]" \
  --body "## Changes
- [Auto-generated]

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass

## Checklist
- [ ] Code follows guidelines
- [ ] Documentation updated"
```
~~~

## Claude Code Agents

### Code Simplifier

**`.claude/agents/code-simplifier.md`**:

~~~markdown
---
hooks:
  PostToolUse:
    - matcher: "Write|Edit"
      hooks:
        - type: "command"
          command: "npm test -- --bail"
---

# Code Simplifier Agent

Refactor code while preserving behavior.

## Principles

**PRESERVE**:
- Public APIs
- Function signatures
- Return types
- Side effects
- Error behavior

**SIMPLIFY**:
- Remove verbose comments
- Extract repeated logic
- Flatten nested conditionals
- Use early returns
- Rename unclear variables
- Remove dead code

**KEEP**:
- Performance optimizations (mark with `// PERF:`)
- Security checks
- Edge case handling

## Process

1. Read target files
2. Analyze complexity
3. Identify opportunities
4. Apply refactoring
5. Run tests
6. Report changes

Always verify with tests!
~~~

### App Verifier

**`.claude/agents/verify-app.md`**:

~~~markdown
# Verify App Agent

Comprehensive application verification.

## Steps

1. Type Check: `npm run typecheck`
2. Lint: `npm run lint`
3. Unit Tests: `npm test`
4. Build: `npm run build`
5. Integration Tests: `npm run test:integration`
6. Start App: `npm run dev`
7. Health Check: `curl localhost:3000/health`
8. Cleanup

## Report Format

```
✓ Type Check: Passed
✓ Lint Check: Passed
✓ Unit Tests: 145 passed, 0 failed
✓ Build: Success (245KB gzipped)
✓ Integration Tests: 23 passed
✓ App: Started successfully
✓ Health Check: OK

All verifications passed!
```
~~~

---

# 5부: 프로젝트 구조 구축

## 완전한 프로젝트 초기화

```bash
# WSL에서 실행

# 1. 프로젝트 디렉토리 생성
mkdir -p ~/projects/my-app
cd ~/projects/my-app

# 2. 전체 구조 생성
mkdir -p .claude/commands .claude/agents
mkdir -p .antigravity
mkdir -p .agent/workflows
mkdir -p scripts src

# 3. 기본 파일 생성
touch _shared_rules.md
touch scripts/sync-rules.sh
touch .claude/settings.json

# 4. Git 초기화
git init
cat > .gitignore << EOF
node_modules/
.env
.env.local
dist/
build/
*.log
EOF

# 5. Package.json
npm init -y

# 6. TypeScript 설정
npm install -D typescript @types/node
npx tsc --init

# 7. 규칙 파일 복사
# (_shared_rules.md 내용 작성)

# 8. 동기화 스크립트 복사 및 실행
chmod +x scripts/sync-rules.sh
./scripts/sync-rules.sh

# 9. Git 초기 커밋
git add .
git commit -m "chore: Initialize project structure"
```

## Claude Code Settings

**`.claude/settings.json`**:

```json
{
  "model": "claude-sonnet-4-5",
  "responseLanguage": "ko",
  "useCache": true,
  "streamResponses": true,
  
  "permissions": {
    "Bash": [
      "npm *",
      "git add *",
      "git commit *",
      "git push",
      "git pull",
      "git status",
      "docker-compose *",
      "python *.py",
      "node *.js"
    ],
    "Task": [
      "code-simplifier",
      "verify-app"
    ]
  },
  
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run format || true"
          }
        ]
      }
    ]
  },
  
  "checkpointing": {
    "enabled": true,
    "autoSave": true,
    "maxCheckpoints": 10
  }
}
```

## 프로젝트 템플릿 저장

```bash
# 완성된 구조를 템플릿으로 저장
cd ~/projects
tar -czf project-template.tar.gz \
  my-app/.claude \
  my-app/.antigravity \
  my-app/.agent \
  my-app/scripts \
  my-app/_shared_rules.md \
  my-app/.gitignore

# 새 프로젝트 시작 시
mkdir new-project
cd new-project
tar -xzf ~/projects/project-template.tar.gz --strip-components=1
npm init -y
./scripts/sync-rules.sh
```

---

# 6부: 통합 워크플로우

## 일일 개발 워크플로우

### 아침 (Planning)

```bash
# [Windows] Antigravity 실행
# Manager View에서 Gemini 3 Pro 선택

"Review today's tasks and create implementation plan:
1. User authentication refactoring
2. Password reset feature
3. Email verification bug fix"

# Gemini가 분석 및 상세 계획 수립
# → artifacts/plan.md 생성
```

### 오전 (Feature 개발 - Backend)

```bash
# [WSL] Claude Code로 백엔드 구현
cd ~/projects/my-app

# Plan 모드로 시작
claude --mode plan "Implement password reset feature following @CLAUDE.md"

# Plan 검토 후 승인
# Opus 4.5가 구현 시작

# 진행 상황 확인
claude "Show progress"

# 완료 후 테스트
npm test
```

### 오전 (Feature 개발 - Frontend)

```
# [Windows] Antigravity에서 프론트엔드

# WSL 프로젝트 열기
File > Open Folder
→ \\wsl$\Ubuntu\home\claude\projects\my-app

# Workflow 실행
/create-component PasswordResetForm

# 생성 확인
# → src/components/PasswordResetForm/ 생성됨
```

### 점심 후 (Testing)

```
# [Windows] Antigravity Browser Agent

Manager View에서:
"Test the password reset flow:
1. Navigate to /reset-password
2. Enter email: test@example.com
3. Check console for API call
4. Verify email sent (mock)
5. Follow reset link
6. Enter new password
7. Login with new password

Capture screenshots at each step."

# Agent가 자동으로:
# - 브라우저 실행
# - 각 단계 수행
# - 스크린샷 캡처
# - 결과 artifacts 생성
```

### 오후 (Refinement)

```bash
# [WSL] Claude Code로 정리

# 변경된 파일들 정리
claude -a code-simplifier "Simplify all changed files"

# 테스트 추가
claude -s generate-tests "Add tests for PasswordResetService"

# 검증
claude -a verify-app "Verify the application"
```

### 마무리 (Review & Commit)

```
# [Windows] Antigravity 최종 검토

Manager View에서:
"Review all changes made today:
- Code quality check
- Test coverage (should be 80%+)
- Documentation completeness
- Security review
- Performance considerations"

# artifacts/review.md 생성됨
```

```bash
# [WSL] PR 생성

claude /commit-push-pr

# 타입 선택: feat
# 설명 입력: Add password reset feature

# 자동으로:
# - git add .
# - git commit
# - git push
# - gh pr create
# → PR URL 출력
```

## Feature 개발 전체 프로세스

### Phase 1: Research (Antigravity)

```
[Antigravity] Gemini 3 Pro (거대한 컨텍스트)
↓
요구사항 분석
- 사용자 스토리 검토
- 유사 구현 사례 조사 (web search)
- 기술 스택 비교
- 보안 고려사항
↓
artifacts/research.md
```

### Phase 2: Planning (Claude Code)

```bash
[Claude Code] Opus 4.5 (최고 정확도)
↓
claude --mode plan "@artifacts/research.md Create implementation plan"
↓
Plan 생성:
- 파일 구조
- 단계별 구현 순서
- 테스트 전략
- 예상 시간
↓
plan/implementation-plan.md
```

### Phase 3: Implementation (병행)

```bash
[Backend - WSL Claude Code]
claude "Implement authentication API following the plan"
↓
- routes/auth.route.ts
- controllers/auth.controller.ts
- services/auth.service.ts
- schemas/auth.schema.ts

[Frontend - Windows Antigravity]
/create-component LoginForm
/create-component RegisterForm
↓
- components/LoginForm/
- components/RegisterForm/
```

### Phase 4: Integration (Claude Code)

```bash
[Claude Code]
claude "Connect frontend components to backend API"
↓
- API client setup
- Error handling
- Loading states
- Form validation
```

### Phase 5: Testing (Antigravity)

```
[Antigravity] Browser Agent
↓
자동 테스트 수행:
1. 회원가입 플로우
2. 로그인 플로우
3. 에러 케이스
4. 성능 측정
↓
artifacts/test-results.md
+ 스크린샷 캡처
```

### Phase 6: Refinement (Claude Code)

```bash
[Claude Code]
# 코드 정리
claude -a code-simplifier "Clean up all files"

# 테스트 추가
claude -s generate-tests "Complete test coverage"

# 최종 검증
claude -a verify-app "Full verification"
```

### Phase 7: Deployment (Antigravity)

```
[Antigravity]
/deploy
↓
Environment: staging
↓
자동 실행:
- Type check ✓
- Lint ✓
- Tests ✓
- Build ✓
- Deploy to staging ✓
- Health check ✓
- Smoke tests ✓
↓
artifacts/deployment-report.md
```

## 버그 수정 워크플로우

```bash
# 1. [WSL] Claude Code로 분석
claude "Analyze the authentication bug. 
Error: 'Token expired' appearing prematurely."

# 2. Opus 4.5가 전체 코드베이스 분석
# → 문제 위치 특정
# → 근본 원인 파악

# 3. Checkpoint 생성 (안전장치)
claude /checkpoint create "Before bug fix"

# 4. 수정 진행
claude "Fix the token expiration logic:
- Use 1 hour expiration
- Implement refresh token
- Add proper error handling"

# 5. [Windows] Antigravity로 테스트
"Test authentication with different scenarios:
- Normal login (should work)
- Token near expiration (should refresh)
- Token expired (should prompt re-login)
- Refresh token expired (should re-auth)"

# 6. 문제 발생 시 롤백
claude /checkpoint restore 1

# 7. 다시 시도 (다른 접근)
claude "Fix using different approach: JWT refresh strategy"

# 8. 성공 시 정리
claude -a code-simplifier "Clean up the fix"

# 9. 회귀 테스트 추가
claude -s generate-tests "Add regression tests for token expiration"

# 10. PR
claude /commit-push-pr
# Type: fix
# Description: Fix premature token expiration
```

## 리팩토링 워크플로우

```
# 1. [Antigravity] 현황 분석
"Analyze the authentication module for refactoring:
- Identify code smells
- Measure complexity
- Find duplication
- Review test coverage
- Check performance bottlenecks"

# 2. artifacts/refactor-analysis.md 생성

# 3. [Claude Code] 리팩토링 계획
claude --mode plan "@artifacts/refactor-analysis.md 
Create phased refactoring plan"

# 4. Checkpoint (안전장치)
claude /checkpoint create "Before refactoring"

# 5. 단계적 리팩토링
claude "Phase 1: Extract repeated login logic into helper"
claude -a verify-app
claude /checkpoint create "Phase 1 complete"

claude "Phase 2: Simplify nested auth checks with guard clauses"
claude -a verify-app
claude /checkpoint create "Phase 2 complete"

claude "Phase 3: Optimize token validation performance"
claude -a verify-app

# 6. 최종 정리
claude -a code-simplifier "Final cleanup"

# 7. [Antigravity] 회귀 테스트
/generate-tests

"Run comprehensive regression tests:
- All existing tests should pass
- Performance should improve
- No breaking changes"

# 8. [Claude Code] 전체 검증
claude -a verify-app "Full verification with load testing"
```

---

# 7부: 실전 예제

## 예제 1: E-Commerce 상품 관리

### Requirements

```
사용자 스토리:
- 관리자가 상품을 CRUD할 수 있다
- 상품은 이름, 설명, 가격, 재고, 이미지를 가진다
- 고객은 상품 목록을 보고 상세 정보를 확인할 수 있다
- 장바구니에 추가할 수 있다
```

### Step 1: Research (Antigravity)

```
Manager View:
"Research best practices for e-commerce product management:
- Database schema design
- Image handling (upload, storage, CDN)
- Inventory management
- Pricing strategies
- Security considerations"

→ artifacts/ecommerce-research.md
```

### Step 2: Planning (Claude Code)

```bash
claude --mode plan "Create implementation plan for product CRUD:
- Follow @CLAUDE.md guidelines
- Use Clean Architecture
- Include comprehensive tests
- Consider scalability"

→ plan/product-crud-plan.md
```

### Step 3: Database Schema (Claude Code)

```bash
claude "Create Prisma schema for products"
```

**Generated: `prisma/schema.prisma`**

```prisma
model Product {
  id          String   @id @default(uuid())
  name        String
  slug        String   @unique
  description String?
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  images      String[]
  category    Category @relation(fields: [categoryId], references: [id])
  categoryId  String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  cartItems   CartItem[]
  
  @@index([categoryId])
  @@index([createdAt])
}
```

### Step 4: Backend API (Claude Code)

```bash
claude "Implement product CRUD API endpoints following the plan"
```

**Generated files:**
- `src/routes/products.route.ts`
- `src/controllers/products.controller.ts`
- `src/services/products.service.ts`
- `src/schemas/products.schema.ts`
- `src/routes/products.route.test.ts`

### Step 5: Admin UI (Antigravity)

```
/create-component ProductList
/create-component ProductForm
/create-component ImageUploader
```

**Generated:**
- `src/components/ProductList/`
- `src/components/ProductForm/`
- `src/components/ImageUploader/`

### Step 6: Customer UI (Antigravity)

```
/create-component ProductCatalog
/create-component ProductDetail
/create-component AddToCartButton
```

### Step 7: Integration Testing (Antigravity)

```
"Test complete product management flow:

Admin Flow:
1. Login as admin
2. Navigate to /admin/products
3. Click 'Add Product'
4. Fill form: name, description, price, stock
5. Upload images
6. Submit
7. Verify product appears in list
8. Edit product
9. Delete product

Customer Flow:
1. Navigate to /products
2. View product catalog
3. Click product
4. View detail page
5. Click 'Add to Cart'
6. Verify cart updated
7. Check inventory decreased

Capture screenshots and measure performance."
```

### Step 8: Refinement (Claude Code)

```bash
# Code quality
claude -a code-simplifier "Simplify product management code"

# Tests
claude -s generate-tests "Complete test coverage for products"

# Verification
claude -a verify-app "Verify product module"
```

### Step 9: Deployment (Antigravity)

```
/deploy

Environment: staging

→ All checks pass
→ Deployed to staging
→ Health check: ✓
→ Smoke tests: ✓
```

## 예제 2: SaaS Dashboard with Analytics

### Requirements

```
사용자 스토리:
- 사용자가 대시보드에서 핵심 지표를 본다
- 실시간으로 데이터 업데이트된다
- 날짜 범위 필터링
- 차트: 라인, 바, 파이
- 데이터 테이블
- CSV 내보내기
```

### Implementation

```bash
# [Claude Code] Backend - 데이터 API
claude "Create analytics API:
- GET /api/analytics/overview (KPIs)
- GET /api/analytics/trends (time series)
- GET /api/analytics/distribution (pie chart data)
- GET /api/analytics/export (CSV)
- Support date range filtering
- Implement caching (Redis)"

# [Antigravity] Frontend - Dashboard Agent
"Create comprehensive dashboard using Claude Code extension:

Requirements:
- 4 KPI cards (Users, Revenue, Conversions, Growth)
- Line chart for trends (Recharts)
- Bar chart for comparisons
- Pie chart for distribution
- Data table with pagination
- Date range picker
- CSV export button
- Auto-refresh every 30s
- Loading skeletons
- Error handling

Use Tailwind for styling.
Dark mode support."

# Generated: src/components/Dashboard/

# [Antigravity] Test
"Test dashboard:
1. Load with default date range
2. Verify all charts render
3. Change date range → data updates
4. Test auto-refresh
5. Export CSV
6. Test error states
7. Measure performance

Capture screenshots of all states."

# [Claude Code] Optimize
claude "Optimize dashboard performance:
- React.memo for chart components
- useMemo for expensive calculations
- Debounce date picker
- Virtual scrolling for table
- Lazy load charts"
```

## 예제 3: Real-time Chat Application

### Backend (Claude Code in WSL)

```bash
claude "Create WebSocket chat server:
- Socket.io integration
- User authentication
- Rooms support
- Message persistence (MongoDB)
- Typing indicators
- Online users list
- File upload support
- Rate limiting"
```

### Frontend (Antigravity)

```
/create-component ChatRoom
/create-component MessageList
/create-component MessageInput
/create-component UserList
/create-component FileUploader
```

### Real-time Testing (Antigravity)

```
"Test real-time chat with multiple browser windows:

Window 1 (User A):
1. Login as Alice
2. Join room 'general'
3. Type message 'Hello'
4. Upload image

Window 2 (User B):
1. Login as Bob
2. Join room 'general'
3. Should see Alice's message in real-time
4. Type reply 'Hi Alice'
5. Send

Window 1:
1. Should see Bob's reply instantly
2. Check typing indicator works

Test edge cases:
- Network disconnect/reconnect
- Rate limiting
- Large file upload
- Many concurrent users

Capture video of real-time interactions."
```

---

# 8부: 문제 해결

## WSL 관련 문제

### 문제 1: /mnt/c/에서 작업이 매우 느림

**증상**:
```bash
cd /mnt/c/Users/KTDS/projects
npm install  # 매우 느림 (분 단위)
```

**원인**: Windows 파일시스템을 거쳐서 접근

**해결**:
```bash
# ❌ 느림
cd /mnt/c/Users/...

# ✅ 빠름 (WSL 파일시스템)
cd ~/projects

# 프로젝트 이동
mv /mnt/c/Users/KTDS/projects/my-app ~/projects/
cd ~/projects/my-app
npm install  # 훨씬 빠름!
```

### 문제 2: Git 줄바꿈 충돌

**증상**:
```bash
# Git이 모든 파일을 modified로 표시
git status
# modified: (almost all files)
```

**원인**: Windows (CRLF) vs Linux (LF) 차이

**해결**:
```bash
# WSL에서
git config --global core.autocrlf input
git config --global core.eol lf

# 프로젝트 루트에 .gitattributes
echo "* text=auto eol=lf" > .gitattributes
git add .gitattributes
git commit -m "chore: Fix line endings"

# 재적용
git rm --cached -r .
git reset --hard
```

### 문제 3: Windows에서 WSL 프로젝트 접근 안 됨

**증상**:
```
Windows 탐색기에서
\\wsl$\Ubuntu를 입력했지만
"네트워크 경로를 찾을 수 없습니다"
```

**해결**:
```bash
# WSL이 실행 중인지 확인
# PowerShell에서
wsl --list --running

# 실행 중이 아니면
wsl

# WSL 터미널이 열린 상태에서
# Windows 탐색기에 \\wsl$\Ubuntu 입력
```

### 문제 4: Docker 명령어 안 됨

**증상**:
```bash
docker ps
# Cannot connect to Docker daemon
```

**해결**:
```
1. Docker Desktop for Windows 설치
2. Docker Desktop 실행
3. Settings > Resources > WSL Integration
4. "Ubuntu" 체크박스 활성화
5. "Apply & Restart"

# WSL에서 확인
docker --version
docker ps
```

## Antigravity 문제

### 문제 5: Extension 아이콘이 안 보임

**증상**:
```
Claude Code Extension 설치했지만
상단 아이콘 없음
사이드바 아이콘 없음
```

**원인**: Extension 충돌 (알려진 버그)

**해결**:
```
방법 1: 재활성화
1. Extensions 패널 (Ctrl+Shift+X)
2. "Claude Code" 검색
3. "Disable" 클릭
4. "Enable" 클릭
5. Antigravity 재시작

방법 2: 다른 Extensions 비활성화
1. 모든 Extension 비활성화
2. Claude Code만 활성화
3. Antigravity 재시작
4. 아이콘 확인
5. 필요한 다른 Extension 하나씩 활성화
```

### 문제 6: 로그인 브라우저가 안 열림

**증상**:
```
Extension에서 /login 입력
아무 일도 안 일어남
```

**해결**:
```bash
# WSL에서 기본 브라우저 설정
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"

# 영구 설정
echo 'export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"' >> ~/.bashrc
source ~/.bashrc

# Antigravity 재시작 후 /login 다시 시도
```

### 문제 7: Antigravity 높은 CPU 사용

**증상**:
```
Antigravity 종료 후에도
CPU 사용률 40-60%
systemd-coredump 프로세스
```

**원인**: 충돌 시 core dump 생성

**해결 (Linux/WSL)**:
```bash
# Core dump 비활성화
ulimit -c 0

# 영구 설정
echo 'ulimit -c 0' >> ~/.bashrc

# 런치 스크립트 생성
cat > ~/launch-antigravity.sh << 'EOF'
#!/bin/bash
ulimit -c 0
antigravity "$@"
EOF

chmod +x ~/launch-antigravity.sh

# 사용
~/launch-antigravity.sh
```

## Claude Code 문제

### 문제 8: API Rate Limit

**증상**:
```
"Rate limit exceeded"
"Too many requests"
```

**해결**:
```
1. Pro 계정 확인 ($20/월)
2. 할당량 확인: https://claude.ai/settings
3. 잠시 대기 (1시간)
4. Antigravity 무료 할당량 사용
5. 업그레이드 고려 (Team/Enterprise)
```

### 문제 9: Claude Code 명령어 안 됨

**증상**:
```bash
claude
# command not found
```

**해결**:
```bash
# 설치 확인
which claude

# 없으면 재설치
npm install -g @anthropic/claude-code

# PATH 확인
echo $PATH
# npm global bin이 포함되어 있어야 함

# npm global bin 경로 확인
npm config get prefix
# /home/claude/.npm-global

# PATH에 추가
echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# 확인
claude --version
```

### 문제 10: Checkpoint 복원 안 됨

**증상**:
```bash
claude /checkpoint restore 1
# Checkpoint not found
```

**해결**:
```bash
# Checkpoint 목록 확인
claude /checkpoint list

# Checkpoint 디렉토리 확인
ls -la .claude/checkpoints/

# 없으면 checkpointing 활성화 확인
cat .claude/settings.json
# "checkpointing": {"enabled": true}

# 새 checkpoint 생성
claude /checkpoint create "test"

# 확인
claude /checkpoint list
```

## 규칙 동기화 문제

### 문제 11: sync-rules.sh 실행 안 됨

**증상**:
```bash
./scripts/sync-rules.sh
# Permission denied
```

**해결**:
```bash
# 권한 부여
chmod +x scripts/sync-rules.sh

# 실행
./scripts/sync-rules.sh
```

### 문제 12: CLAUDE.md와 rules.md가 다름

**증상**:
```bash
diff CLAUDE.md .antigravity/rules.md
# 차이 발견
```

**해결**:
```bash
# 재동기화
./scripts/sync-rules.sh

# 확인
diff CLAUDE.md .antigravity/rules.md
# 차이 없어야 함

# Git 커밋
git add CLAUDE.md .antigravity/rules.md
git commit -m "chore: Sync rules"
```

## 성능 최적화

### 문제 13: 규칙 파일이 너무 큼

**증상**:
```bash
wc -l CLAUDE.md
# 2000+ lines
```

**해결**:
```markdown
# CLAUDE.md를 간결하게
# 핵심 규칙만 유지 (100-200줄)

핵심 원칙 10개
금지사항 5개
간단한 예시

상세 내용:
"자세한 내용은 docs/CODING_GUIDELINES.md 참조"
```

### 문제 14: Workflow가 너무 느림

**증상**:
```
/create-component Button
# 5분 이상 소요
```

**해결**:
```markdown
# Workflow 최적화

❌ 느린 방법:
"Generate complete component with:
- TypeScript
- CSS Module
- 10+ tests
- Storybook
- Documentation
- Accessibility
- Performance optimization
- ..."

✅ 빠른 방법:
"Generate component:
1. TypeScript interface
2. Basic implementation
3. 3 essential tests
4. Storybook story

User can extend later."
```

## 디버깅 팁

### 일반 디버깅 체크리스트

```
[ ] WSL 실행 중인가?
    wsl --list --running

[ ] 프로젝트가 WSL 홈에 있는가?
    pwd
    # /home/claude/projects/... ✓

[ ] Node.js 버전 확인
    node --version  # v20+

[ ] npm 글로벌 경로 확인
    echo $PATH
    # npm global bin 포함?

[ ] Claude Code 로그인 상태
    claude auth whoami

[ ] Antigravity 로그인 상태
    # Antigravity 우측 상단에 Gmail 주소?

[ ] Git 설정 확인
    git config --list

[ ] Docker 실행 중
    docker ps

[ ] 규칙 파일 동기화
    diff CLAUDE.md .antigravity/rules.md
```

### 로그 확인

```bash
# Claude Code 로그
cat ~/.claude/logs/claude-code.log

# Antigravity 로그 (Windows)
# %APPDATA%\Antigravity\logs\

# WSL 시스템 로그
journalctl -xe
```

---

# 부록: 빠른 참조

## 필수 명령어

### WSL

```bash
# WSL 시작/종료
wsl
exit

# Windows 탐색기 열기
explorer.exe .

# 프로젝트 생성
mkdir -p ~/projects/my-app
cd ~/projects/my-app
npm init -y
git init
```

### Claude Code

```bash
# 기본 사용
claude "프롬프트"

# Plan 모드
claude --mode plan "프롬프트"

# Agent 실행
claude -a agent-name "프롬프트"

# Slash Command
claude -s command-name "인자"

# Checkpoint
claude /checkpoint create "설명"
claude /checkpoint list
claude /checkpoint restore N

# 로그인/로그아웃
claude auth login
claude auth logout
claude auth whoami
```

### Antigravity

```
# Manager View
- 모델 선택 (Gemini 3 Pro / Claude Opus 4.5)
- Task List 생성
- Implementation Plan 검토

# Workflows
/create-component [Name]
/create-endpoint [Resource]
/generate-tests
/deploy

# Browser Agent
"Test [feature] flow:
1. Step 1
2. Step 2
3. Capture screenshots"

# Extension
/login
/logout
/init
/model [sonnet|opus|haiku]
```

### Git

```bash
# 규칙 동기화 후 커밋
./scripts/sync-rules.sh
git add CLAUDE.md .antigravity/rules.md _shared_rules.md
git commit -m "chore: Update rules"

# 일반 워크플로우
git add .
git commit -m "feat: Add feature"
git push origin HEAD
gh pr create
```

## 프로젝트 체크리스트

### 신규 프로젝트

```
[ ] WSL 디렉토리 생성
    mkdir -p ~/projects/my-app

[ ] 프로젝트 구조 생성
    .claude/, .antigravity/, .agent/, scripts/

[ ] 규칙 파일 작성
    _shared_rules.md

[ ] 동기화 스크립트
    scripts/sync-rules.sh
    chmod +x

[ ] 초기 동기화
    ./scripts/sync-rules.sh

[ ] Git 초기화
    git init
    .gitignore 작성

[ ] Package 초기화
    npm init -y
    npm install -D typescript

[ ] Claude Code 설정
    .claude/settings.json

[ ] Workflows 작성
    .agent/workflows/

[ ] Windows에서 확인
    \\wsl$\Ubuntu\home\claude\projects\my-app

[ ] Antigravity에서 열기
    File > Open Folder

[ ] 초기 커밋
    git add .
    git commit -m "chore: Initialize project"
```

### Feature 개발

```
[ ] Research (Antigravity Gemini)
    Manager View → 연구 및 분석

[ ] Planning (Claude Code Plan)
    claude --mode plan

[ ] Backend (Claude Code)
    API 구현 + 테스트

[ ] Frontend (Antigravity)
    컴포넌트 생성 + 스타일

[ ] Integration (Claude Code)
    연결 + 에러 처리

[ ] Testing (Antigravity)
    Browser Agent 자동 테스트

[ ] Refinement (Claude Code)
    code-simplifier + 테스트 보강

[ ] Review (Antigravity)
    전체 변경사항 검토

[ ] Deployment
    /deploy staging
```

## 권장 설정 요약

### 최적 환경

```
운영체제: Windows 11
WSL: WSL 2 + Ubuntu 22.04+
Claude Code: CLI in WSL
Antigravity: Windows 네이티브 앱
프로젝트: WSL 홈 (~/projects/)
```

### 도구 역할 분담

```
Claude Code (WSL):
- 터미널 작업
- API 구현
- 복잡한 로직
- 테스트 작성
- 코드 정리

Antigravity (Windows):
- GUI 작업
- 컴포넌트 생성
- Browser 테스트
- 계획 및 연구
- 시각적 확인
```

### 비용

```
필수 ($20/월):
- Claude Code Pro 또는
- Claude Code Extension Pro

무료:
- Antigravity (Public Preview)
- Gemini 3 Pro (제한적)
- Claude Sonnet/Opus 4.5 (제한적)
```

---

## 마치며

이 가이드를 따르면 **2026년 최고의 AI 개발 환경**을 구축할 수 있습니다:

✅ **최고 성능**: WSL에서 Claude Code (1.5~2배 빠름)  
✅ **완벽한 GUI**: Windows에서 Antigravity  
✅ **통합 워크플로우**: Rules + Workflows  
✅ **자동화**: 동기화 스크립트, Git Hooks  
✅ **팀 협업**: Git으로 설정 공유  

**시작하기**:

```bash
# 1. WSL 설치
wsl --install

# 2. Claude Code 설치
npm install -g @anthropic/claude-code

# 3. Antigravity 다운로드
# https://antigravity.google/download

# 4. 첫 프로젝트
mkdir -p ~/projects/my-app
cd ~/projects/my-app
# 이 가이드 따라하기!
```

**더 알아보기**:
- Claude Code 문서: https://docs.claude.ai/code
- Antigravity 문서: https://antigravity.google/docs
- MCP 서버: https://modelcontextprotocol.io

행운을 빕니다! 🚀

**작성 일자: 2026-01-09**
