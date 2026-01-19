---
title: "Claude Code 완벽 종합 가이드"
date: 2026-01-19 20:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  Guide,  Claude.write]
---



> **Claude Code를 제로부터 마스터까지: 2025년 최신 버전 완전 정복**

## 관련 GitHUB

[claude-code-cheat-sheet](https://github.com/Njengah/claude-code-cheat-sheet)

## 📑 목차

1. [소개 및 개요](#1-소개-및-개요)
2. [설치 및 시작하기](#2-설치-및-시작하기)
3. [기본 명령어 (레벨 1-3)](#3-기본-명령어-레벨-1-3)
4. [중급 명령어 (레벨 4-6)](#4-중급-명령어-레벨-4-6)
5. [고급 명령어 (레벨 7-10)](#5-고급-명령어-레벨-7-10)
6. [Subagents 완벽 가이드](#6-subagents-완벽-가이드)
7. [실전 활용 예제](#7-실전-활용-예제)
8. [고급 기능 (LSP, Hooks, Skills)](#8-고급-기능-lsp-hooks-skills)
9. [트러블슈팅](#9-트러블슈팅)
10. [2025년 최신 업데이트](#10-2025년-최신-업데이트)
11. [참고 자료](#11-참고-자료)

---

## 1. 소개 및 개요

### 1.1 Claude Code란?

Claude Code는 Anthropic이 개발한 AI 기반 코딩 어시스턴트입니다. 단순한 코드 자동완성을 넘어 전체 개발 프로세스를 지원하는 강력한 도구입니다.

**핵심 특징:**
- 터미널 기반 REPL 인터페이스
- VS Code 확장 프로그램 지원 (베타)
- Subagents를 통한 병렬 작업 처리
- MCP (Model Context Protocol) 통합
- 200K 컨텍스트 윈도우 (Opus 4.5)

### 1.2 주요 모델 비교

| 모델 | 용도 | 특징 | 가격 |
|------|------|------|------|
| **Claude Opus 4.5** | 복잡한 작업 | 최고 성능, 200K 컨텍스트 | $15/1M 토큰 (출력) |
| **Claude Sonnet 4.5** | 일상적 작업 | 균형잡힌 성능/속도 | $3/1M 토큰 (출력) |
| **Claude Haiku 4.5** | 빠른 작업 | 고속, 저비용 | $1/1M 토큰 (출력) |

### 1.3 경쟁사 비교

**Claude Code vs Cursor vs Codex:**
- **Claude Code**: Subagents를 통한 병렬 처리, 뛰어난 추론 능력
- **Cursor**: IDE 통합에 강점, UI/UX 우수
- **GitHub Copilot/Codex**: 코드 완성에 특화

**선택 기준:**
- 복잡한 리팩토링/아키텍처 작업 → Claude Code
- 실시간 코드 완성 중심 → Cursor
- GitHub 워크플로우 통합 → Copilot

---

## 2. 설치 및 시작하기

### 2.1 시스템 요구사항

- Node.js 18 이상
- Unix 기반 시스템 (Linux, macOS) 또는 Windows WSL
- 최소 8GB RAM (권장 16GB)

### 2.2 설치 방법

#### npm을 통한 설치 (권장)

```bash
# Claude Code 설치
npm install -g @anthropic-ai/claude-code

# 버전 확인
claude --version

# Windows 사용자
wsl  # WSL 진입
npm install -g @anthropic-ai/claude-code
```

#### curl을 통한 설치

```bash
curl -sL https://install.anthropic.com | sh
```

### 2.3 인증 설정

```bash
# 대화형 REPL 시작
claude

# 초기 실행 시 API 키 입력 요구
# 또는 환경 변수 설정
export ANTHROPIC_API_KEY="your-api-key"
```

### 2.4 첫 실행

```bash
# 기본 실행
claude

# 초기 프롬프트와 함께 시작
claude "이 프로젝트를 요약해줘"

# 프린트 모드 (실행 후 종료)
claude -p "이 함수를 설명해줘"

# 가장 최근 대화 계속하기
claude -c
```

### 2.5 VS Code 확장 설치 (베타)

1. VS Code Extension Marketplace에서 "Claude Code" 검색
2. 설치 후 사이드바에서 Claude 아이콘 클릭
3. API 키로 인증
4. 실시간 코드 제안 및 diff 뷰 사용 가능

---

## 3. 기본 명령어 (레벨 1-3)

### 🟢 레벨 1: 필수 기본 명령어

#### 설치 및 시작

```bash
# Claude Code 설치
npm install -g @anthropic-ai/claude-code

# 대화형 REPL 시작
claude

# 초기 프롬프트와 함께 시작
claude "summarize this project"

# 버전 확인
claude --version

# 업데이트
claude update
```

#### 기본 네비게이션

```bash
/help          # 도움말 및 사용 가능한 명령어 표시
/exit          # REPL 종료
/clear         # 대화 기록 초기화
/config        # 설정 패널 열기
/doctor        # Claude Code 설치 상태 확인
```

#### 기본 파일 작업

```bash
# 프린트 모드 - 실행 후 종료
claude -p "explain this function"

# 파이프를 통한 입력 처리
cat logs.txt | claude -p "explain"

# 최근 대화 계속하기
claude -c

# SDK를 통한 계속
claude -c -p "Check for type errors"
```

#### 세션 관리

```bash
# ID로 세션 재개
claude -r "abc123" "Finish this PR"

# 플래그를 통한 재개
claude --resume abc123 "query"

# 대화 계속
claude --continue
```

#### 키보드 단축키

| 단축키 | 동작 |
|--------|------|
| `Ctrl+C` | 현재 작업 취소 |
| `Ctrl+D` | Claude Code 종료 |
| `Tab` | 자동완성 |
| `↑/↓` | 명령어 히스토리 탐색 |
| `Ctrl+R` | 프롬프트 히스토리 검색 |
| `Alt+P` (Mac: `Option+P`) | 모델 전환 |

### 🟡 레벨 2: 설정 및 모델 관리

#### 모델 설정

```bash
# 모델 전환
claude --model sonnet                    # Sonnet 모델 사용
claude --model opus                      # Opus 모델 사용
claude --model claude-sonnet-4-5-20250929  # 특정 버전 사용

# 대화 중 모델 전환
/model                                   # 모델 선택기 열기
```

#### 디렉토리 관리

```bash
# 추가 작업 디렉토리 지정
claude --add-dir ../apps ../lib

# 경로 검증
claude --add-dir /path/to/project
```

#### 출력 형식

```bash
# 다양한 출력 형식
claude -p "query" --output-format json
claude -p "query" --output-format text
claude -p "query" --output-format stream-json

# 입력 형식
claude -p --input-format stream-json
```

#### 세션 제어

```bash
# 대화 턴 수 제한
claude -p --max-turns 3 "query"

# 상세 로깅
claude --verbose

# 세션 비용 및 시간
/cos                      # 총 비용 및 시간 표시
```

### 🟠 레벨 3: 도구 및 권한 관리

#### 도구 관리

```bash
# 프롬프트 없이 특정 도구 허용
claude --allowedTools "Bash(git log:*)" "Bash(git diff:*)" "Write"

# 특정 도구 차단
claude --disallowedTools "Bash(rm:*)" "Bash(sudo:*)"

# 특정 도구에 대한 권한 프롬프트
claude -p --permission-prompt-tool mcp_auth_tool "query"

# 모든 권한 프롬프트 건너뛰기 (위험)
claude --dangerously-skip-permissions
```

#### Slash 명령어 - 세션 관리

```bash
/compact [instructions]   # 선택적 지침으로 대화 요약
/clear                    # 대화 기록 및 컨텍스트 재설정
/exit                     # REPL 종료
/help                     # 사용 가능한 명령어 표시
/config                   # 설정 패널 열기
```

#### Slash 명령어 - 시스템

```bash
/doctor                   # 설치 상태 확인
/cos                      # 현재 세션의 비용 및 시간 표시
/ide                      # IDE 통합 관리
/context                  # 컨텍스트 윈도우 정보 확인
/stats                    # 세션 통계 확인
```

---

## 4. 중급 명령어 (레벨 4-6)

### 🔴 레벨 4: MCP 및 고급 통합

#### Model Context Protocol (MCP)

```bash
# MCP 서버 설정
claude --mcp

# MCP 관리 (slash 명령어를 통해)
/mcp                      # MCP 기능 접근
```

**MCP 서버 설정 예시:**

```json
// ~/.claude/mcp.json
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token"
      }
    },
    "database": {
      "command": "python",
      "args": ["-m", "mcp_server_sqlite"],
      "env": {
        "DB_PATH": "/path/to/database.db"
      }
    }
  }
}
```

#### 고급 파이핑

```bash
# 복잡한 파이핑 작업
git log --oneline | claude -p "summarize these commits"
cat error.log | claude -p "find the root cause"
ls -la | claude -p "explain this directory structure"
```

#### 프로그래밍 방식 사용

```bash
# 스크립팅을 위한 JSON 출력
claude -p "analyze code" --output-format json

# 실시간 처리를 위한 스트림 JSON
claude -p "large task" --output-format stream-json

# 배치 처리
claude -p --max-turns 1 "quick query"
```

### 🔵 레벨 5: 고급 워크플로우 및 자동화

#### 사용자 정의 Slash 명령어

```bash
# .claude/commands/에 사용자 정의 명령어 생성
# 예: .claude/commands/debug.md
/debug                    # 사용자 정의 debug 명령어 실행
/test                     # 사용자 정의 test 명령어 실행
/deploy                   # 사용자 정의 deploy 명령어 실행
```

**사용자 정의 명령어 예시:**

`.claude/commands/review.md`

```markdown

---
name: review
description: 코드 리뷰 수행
---

이 코드를 다음 관점에서 리뷰해주세요:
1. 보안 취약점
2. 성능 이슈
3. 코드 품질
4. 베스트 프랙티스 준수
```

#### 복잡한 도구 조합

```bash
# 고급 도구 권한
claude --allowedTools "Bash(git:*)" "Write" "Read" \
       --disallowedTools "Bash(rm:*)" "Bash(sudo:*)"

# 다중 디렉토리 접근
claude --add-dir ../frontend ../backend ../shared
```

#### 성능 최적화

```bash
# 성능을 위한 컨텍스트 제한
claude -p --max-turns 5 "focused query"

# 컨텍스트 정기적 초기화
/clear                    # 작업 간 더 나은 성능을 위해 사용

# 대화 압축
/compact "keep only important parts"
```

### 🟣 레벨 6: 마스터 명령어

#### 고급 설정

```bash
# 복잡한 모델 및 도구 설정
claude --model claude-sonnet-4-5-20250929 \
       --add-dir ../apps ../lib ../tools \
       --allowedTools "Bash(git:*)" "Write" "Read" \
       --verbose \
       --output-format json
```

#### 자동화 스크립트

```bash
#!/bin/bash
# 스크립트화된 Claude 상호작용
claude -p "analyze codebase" --output-format json > analysis.json
claude -p "generate tests" --max-turns 3 --output-format text > tests.txt
```

#### 고급 세션 관리

```bash
# 세션 ID 관리
SESSION_ID=$(claude -p "start analysis" --output-format json | jq -r '.session_id')
claude -r "$SESSION_ID" "continue analysis"
```

#### 복잡한 워크플로우

```bash
# 다단계 자동화
claude -p "analyze project structure" | \
claude -p "suggest improvements" | \
claude -p "create implementation plan"
```

---

## 5. 고급 명령어 (레벨 7-10)

### 🟤 레벨 7: 워크플로우 자동화

#### 자동화된 코드 리뷰 워크플로우

```bash
#!/bin/bash
# 자동화된 PR 리뷰 프로세스
git diff HEAD~1 | claude -p "review this PR for security issues" > security_review.md
git diff HEAD~1 | claude -p "check for performance issues" > performance_review.md
git diff HEAD~1 | claude -p "suggest improvements" > improvements.md
```

#### CI/CD 파이프라인 통합

```bash
# CI/CD 파이프라인 통합
claude -p "analyze test coverage" --output-format json | jq '.coverage_percentage'
claude -p "generate release notes from commits" --max-turns 2 > RELEASE_NOTES.md
```

#### 배치 처리 워크플로우

```bash
# 다중 파일 처리
find . -name "*.js" -exec claude -p "analyze this file for bugs: {}" \; > bug_report.txt

# 자동화된 문서 생성
for file in src/*.py; do
    claude -p "generate docstring for $file" --output-format text >> docs.md
done
```

### ⚫ 레벨 8: 통합 및 생태계

#### IDE 통합 명령어

```bash
# VS Code 통합
/ide vscode                # VS Code 통합 설정
/ide configure             # IDE 설정

# 사용자 정의 IDE 명령어
claude --ide-mode "explain selected code"
claude --ide-mode "refactor this function"
```

#### Git 워크플로우 통합

```bash
# Git hooks 통합
claude -p "create pre-commit hook for code quality" > .git/hooks/pre-commit

# 고급 Git 작업
git log --oneline -10 | claude -p "create changelog from these commits"
git diff --name-only | claude -p "explain what changed in this commit"
```

#### 서드파티 도구 연결

```bash
# 데이터베이스 통합
mysql -e "SHOW TABLES" | claude -p "analyze database structure"

# Docker 통합
docker ps | claude -p "analyze running containers"
docker logs container_name | claude -p "find errors in logs"
```

### ⚪ 레벨 9: 성능 및 최적화

#### 메모리 및 리소스 관리

```bash
# 메모리 사용 최적화
claude -p --max-turns 1 "quick analysis"      # 효율성을 위한 단일 턴
claude -p --compact-mode "analyze with minimal context"

# 리소스 모니터링
/cos                       # 현재 세션 비용 확인
/doctor --performance      # 성능 진단
/stats                     # 세션 통계
```

#### 캐싱 및 최적화

```bash
# 효율적인 세션 재사용
claude -c "continue previous analysis"         # 기존 컨텍스트 재사용
claude --cache-results "repetitive task"      # 일반 작업 캐싱

# 병렬 처리
claude -p "task 1" & claude -p "task 2" & wait  # 병렬 실행
```

#### 대규모 처리

```bash
# 대규모 코드베이스를 효율적으로 처리
claude --add-dir . --max-context 50000 "analyze entire project"
claude --stream-output "process large dataset" | head -100
```

### 🔘 레벨 10: 엔터프라이즈 및 프로덕션

#### 팀 협업

```bash
# 공유 팀 설정
claude --config-file team-config.json "standardized analysis"

# 팀 세션 공유
claude -r "team-session-id" "continue team discussion"
```

#### 프로덕션 환경 설정

```bash
# 프로덕션 준비 설정
claude --production-mode \
       --security-enabled \
       --audit-logging \
       --max-turns 10 \
       "production analysis"
```

#### 엔터프라이즈 보안

```bash
# 보안 중심 작업
claude --disallowedTools "Bash(rm:*)" "Bash(sudo:*)" "Bash(chmod:*)" \
       --audit-mode \
       --no-external-calls \
       "secure code review"
```

#### 모니터링 및 컴플라이언스

```bash
# 감사 및 컴플라이언스
claude --audit-log /var/log/claude-audit.log "compliance check"
claude --compliance-mode "analyze for security compliance"
```

---

## 6. Subagents 완벽 가이드

### 6.1 Subagents란?

Subagents는 Claude Code에서 특정 작업 유형을 처리하는 전문화된 AI 어시스턴트입니다. 각 Subagent는:

- **독립된 컨텍스트 윈도우**: 메인 대화와 분리된 작업 공간
- **사용자 정의 시스템 프롬프트**: 특정 역할에 최적화
- **특정 도구 접근 권한**: 보안 및 효율성 강화
- **독립적 권한**: 각 에이전트별 권한 설정

### 6.2 Subagents를 사용하는 이유

**장점:**
1. **컨텍스트 관리**: 메인 대화가 세부사항으로 오염되지 않음
2. **병렬 처리**: 여러 작업을 동시에 수행
3. **전문화**: 각 도메인에 맞춤 설정
4. **재사용성**: 여러 프로젝트에서 공유 가능
5. **보안**: 도구 권한 세밀하게 제어

**사용 시나리오:**
- ✅ 작업이 상세한 출력을 생성하지만 메인 컨텍스트에 필요하지 않을 때
- ✅ 특정 도구 제한이나 권한을 적용하고 싶을 때
- ✅ 작업이 자체 포함되어 있고 요약만 반환할 수 있을 때
- ✅ 병렬로 여러 작업을 수행하고 싶을 때

**사용하지 말아야 할 때:**
- ❌ 지연 시간이 중요한 경우 (새로 시작하며 컨텍스트 수집에 시간 소요)
- ❌ 메인 대화와 긴밀한 상호작용이 필요한 경우
- ❌ 중첩된 위임이 필요한 경우 (Subagent는 다른 Subagent 생성 불가)

### 6.3 내장 Subagents

Claude Code는 여러 내장 Subagents를 제공합니다:

#### Explore Agent
- **목적**: 파일 발견, 코드 검색, 코드베이스 탐색
- **모델**: Haiku 4.5 (빠르고 저렴)
- **특징**: 읽기 전용, 변경 없이 탐색만 수행
- **레벨**: quick, medium, thorough

```bash
# 사용 예시
"파일 구조를 분석해줘"
"이 프로젝트에서 인증 로직을 찾아줘"
```

#### Plan Agent
- **목적**: 계획 모드에서 코드베이스 연구
- **특징**: Explore Agent와 유사하지만 계획 수립에 특화
- **사용**: Plan 모드 활성화 시 자동 사용

```bash
# Plan 모드 진입
Shift+Tab (2번)

# 사용 예시
"이 기능을 구현하기 위한 계획을 세워줘"
```

#### General-Purpose Agent
- **목적**: 복잡한 다단계 작업
- **특징**: 탐색과 수정 모두 가능, 복잡한 추론
- **사용**: 다중 의존 단계가 필요한 작업

```bash
# 사용 예시
"API를 리팩토링하고 테스트를 업데이트해줘"
"데이터베이스 스키마를 변경하고 마이그레이션을 작성해줘"
```

### 6.4 사용자 정의 Subagent 생성

#### 생성 방법 1: GUI를 통한 생성

```bash
# Subagent 메뉴 열기
/agents

# [Create New Agent] 선택
# 프로젝트 레벨 또는 사용자 레벨 선택
```

#### 생성 방법 2: 파일을 통한 생성

**프로젝트 레벨 Subagent:**
```bash
# 위치: .claude/agents/<agent-name>.md
```

**사용자 레벨 Subagent:**
```bash
# 위치: ~/.claude/agents/<agent-name>.md
```

### 6.5 Subagent 설정 파일 구조

```markdown
---
name: code-reviewer
description: "코드 변경 후 PROACTIVELY 사용. 보안 문제, 성능 이슈, 베스트 프랙티스 위반을 찾아 JSON 형식으로 출력."
model: sonnet
tools:
  - Read
  - Bash(git diff:*)
  - Bash(git log:*)
permissionMode: inherit  # 또는 background
skills:
  - security-checklist
---

# Code Reviewer Agent

당신은 전문 코드 리뷰어입니다.

## 역할
코드 변경사항을 분석하여 다음을 확인합니다:
1. 보안 취약점
2. 성능 문제
3. 코드 품질
4. 베스트 프랙티스 준수

## 프로세스
1. git diff를 사용하여 변경사항 확인
2. 각 파일을 체크리스트에 따라 검토
3. 발견된 문제를 심각도별로 분류
4. JSON 형식으로 결과 반환

## 출력 형식
\`\`\`json
{
  "summary": "전체 요약",
  "issues": [
    {
      "severity": "high|medium|low",
      "type": "security|performance|quality",
      "file": "파일 경로",
      "line": 줄 번호,
      "description": "문제 설명",
      "suggestion": "개선 제안"
    }
  ]
}
\`\`\`

## 제약사항
- 파일을 수정하지 마세요
- 읽기 작업만 수행하세요
- 간결하고 실행 가능한 피드백을 제공하세요
```

### 6.6 Subagent 설정 옵션

#### 필수 필드

```yaml
name: agent-name              # 에이전트 이름 (공백 없음)
description: "에이전트 설명"   # Claude가 언제 사용할지 판단
```

#### 선택 필드

```yaml
model: sonnet                 # 사용할 모델 (sonnet, opus, haiku)
tools:                        # 허용된 도구 목록
  - Read
  - Write
  - Bash(git:*)
disallowedTools:              # 차단할 도구
  - Bash(rm:*)
  - Bash(sudo:*)
permissionMode: inherit       # 권한 모드 (inherit, background)
skills:                       # 로드할 스킬
  - skill-name
hooks:                        # 훅 설정
  Start:
    - type: command
      command: "echo Starting agent"
  Stop:
    - type: command
      command: "echo Stopping agent"
```

### 6.7 Subagent 호출 방법

#### 자동 위임

```bash
# Claude가 description을 분석하여 자동 선택
"이 코드를 리뷰해줘"  # code-reviewer 자동 호출

# description에 트리거 키워드 포함 권장
# "PROACTIVELY use", "MUST use"
```

#### 명시적 호출 (권장)

```bash
# @ 기호로 명시적 호출

# 명시적 호출의 장점
# - 예측 가능
# - 디버깅 용이
# - 프로덕션 환경에 적합
```

### 6.8 실전 Subagent 예제

#### 예제 1: Code Reviewer

```markdown
---
name: code-reviewer
description: "코드 변경 후 보안, 성능, 품질을 검토. PROACTIVELY 사용."
model: sonnet
tools:
  - Read
  - Bash(git diff:*)
---

# Code Reviewer

보안 취약점, 성능 이슈, 코드 품질을 검토합니다.

## 체크리스트
- [ ] SQL 인젝션 위험
- [ ] XSS 취약점
- [ ] 인증/인가 문제
- [ ] 성능 병목
- [ ] 메모리 누수
- [ ] 코드 중복
```

#### 예제 2: Test Runner

```markdown
---
name: test-runner
description: "테스트 스위트 실행 및 실패한 테스트 요약."
model: haiku
tools:
  - Bash(npm test:*)
  - Bash(pytest:*)
  - Read
---

# Test Runner

테스트를 실행하고 결과를 요약합니다.

## 프로세스
1. 적절한 테스트 명령어 실행
2. 실패한 테스트 식별
3. 실패 원인 분석
4. JSON 형식으로 요약 반환
```

#### 예제 3: Debugger

```markdown
---
name: debugger
description: "에러 조사 및 디버깅. 로그 분석 및 근본 원인 파악."
model: sonnet
tools:
  - Read
  - Bash(*)
---

# Debugger

에러의 근본 원인을 찾고 해결책을 제안합니다.

## 프로세스
1. 에러 메시지 및 스택 트레이스 분석
2. 관련 코드 검토
3. 로그 파일 확인
4. 근본 원인 파악
5. 최소 수정 제안
```

#### 예제 4: Performance Optimizer

```markdown
---
name: optimizer
description: "성능 병목 찾기 및 최적화 제안."
model: sonnet
tools:
  - Read
  - Bash(npm run benchmark:*)
---

# Performance Optimizer

성능 문제를 식별하고 최적화합니다.

## 분석 영역
- 알고리즘 복잡도
- 데이터베이스 쿼리
- 메모리 사용
- 네트워크 호출
- 캐싱 기회
```

#### 예제 5: Security Auditor

```markdown
---
name: security-auditor
description: "보안 취약점 감사. MUST use for security reviews."
model: opus
tools:
  - Read
  - Bash(npm audit)
---

# Security Auditor

코드와 의존성의 보안 취약점을 감사합니다.

## 검사 항목
- OWASP Top 10
- 의존성 취약점
- 하드코딩된 시크릿
- 안전하지 않은 암호화
- 권한 상승 위험
```

### 6.9 Subagent 실행 모드

#### Foreground (포그라운드) 모드

```yaml
permissionMode: inherit
```

**특징:**
- 완료될 때까지 메인 대화 차단
- 권한 프롬프트가 사용자에게 전달됨
- 명확한 질문 가능 (AskUserQuestion)
- 대화형 작업에 적합

**사용 예시:**
```bash
# 디버거가 작업을 완료할 때까지 대기
```

#### Background (백그라운드) 모드

```yaml
permissionMode: background
```

**특징:**
- 동시에 실행되며 메인 대화 차단 안 함
- 부모의 권한 상속
- 사전 승인되지 않은 작업 자동 거부
- 병렬 작업에 적합
- MCP 도구 사용 불가

**사용 예시:**
```bash
# 백그라운드에서 실행되는 동안 다른 작업 계속
```

### 6.10 Subagent 비활성화

```json
// .claude/settings.json
{
  "permissions": {
    "deny": [
      "Task(subagent-name)"
    ]
  }
}
```

```bash
# CLI 플래그 사용
claude --disallowedTools "Task(code-reviewer)"
```

### 6.11 Subagent 훅 (Hooks)

#### Subagent 내부 훅 (프론트매터)

```yaml
---
name: db-agent
hooks:
  Start:
    - type: command
      command: "./scripts/setup-db.sh"
  Stop:
    - type: command
      command: "./scripts/cleanup-db.sh"
---
```

#### 메인 세션에서 Subagent 훅

```json
// .claude/settings.json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/setup-db-connection.sh"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "db-agent",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/cleanup-db-connection.sh"
          }
        ]
      }
    ]
  }
}
```

### 6.12 Subagent 디자인 패턴

#### 패턴 1: Fork + Gather (분산 수집)

```bash
# 여러 Subagent를 병렬로 실행하고 결과 수집
"@code-reviewer와 @security-auditor를 동시에 실행하여 
이 PR을 검토하고 결과를 통합해줘"
```

#### 패턴 2: Supervisor Loop (감독자 루프)

```bash
# 오케스트레이터가 작업을 할당하고 결과 검토
"계획을 세우고, 각 단계를 적절한 에이전트에게 위임한 후,
결과를 검토하여 최종 보고서를 작성해줘"
```

#### 패턴 3: Sandbox Execution (샌드박스 실행)

```markdown
---
name: deployer
description: "배포 작업 수행"
tools:
  - Bash(kubectl apply:*)
permissionMode: inherit  # 사용자 승인 필요
---

# Deployer

사용자 승인 후에만 배포를 실행합니다.
```

#### 패턴 4: Sequential Chain (순차 체인)

```bash
# Subagent를 순차적으로 체인
# 결과를 받은 후
```

### 6.13 Subagent 베스트 프랙티스

#### 역할을 명확하게

```markdown
❌ 나쁜 예:
description: "코드 작업"

✅ 좋은 예:
description: "보안 취약점에 대한 코드 리뷰. 변경 후 PROACTIVELY 사용."
```

#### 구체적인 프롬프트

```markdown
❌ 나쁜 예:
"코드를 검토하세요"

✅ 좋은 예:
"다음 체크리스트에 따라 검토하세요:
1. SQL 인젝션 위험
2. XSS 취약점
3. ...
JSON 형식으로 결과를 반환하세요."
```

#### 최소 권한 원칙

```yaml
❌ 나쁜 예:
tools:
  - Bash(*)  # 모든 명령어 허용

✅ 좋은 예:
tools:
  - Read
  - Bash(git diff:*)
  - Bash(git log:*)
```

#### 명시적 호출 사용

```bash
❌ 나쁜 예:
"코드 리뷰해줘"  # 자동 위임, 예측 불가

✅ 좋은 예:
"@code-reviewer 최근 변경사항 검토"  # 명시적, 예측 가능
```

#### 출력 형식 표준화

```markdown
## 출력 형식
모든 결과는 다음 JSON 스키마를 따릅니다:
\`\`\`json
{
  "summary": "string",
  "details": [],
  "recommendations": []
}
\`\`\`
```

### 6.14 Subagent 트러블슈팅

#### 문제 1: Subagent가 호출되지 않음

**원인:**
- description이 너무 모호함
- 작업이 description과 일치하지 않음

**해결책:**
```markdown
# description에 트리거 키워드 추가
description: "PROACTIVELY use after code changes for review."

# 또는 명시적 호출 사용
```

#### 문제 2: 권한 프롬프트가 반복됨

**원인:**
- permissionMode가 적절하지 않음
- 필요한 도구가 허용 목록에 없음

**해결책:**
```yaml
# 프론트매터에 도구 사전 승인
tools:
  - Read
  - Bash(git:*)

# 또는 설정 파일에 추가
```

#### 문제 3: 컨텍스트 오버플로우

**원인:**
- 너무 많은 데이터를 메인 세션으로 반환

**해결책:**
```markdown
# 프롬프트에 명시
"간결한 요약만 반환하세요. 세부사항은 파일에 저장하세요."
```

#### 문제 4: 느린 실행 시간

**원인:**
- Opus 모델 사용
- 복잡한 작업

**해결책:**
```yaml
# 빠른 모델로 전환
model: haiku  # 또는 sonnet

# 또는 백그라운드 모드 사용
permissionMode: background
```

### 6.15 Subagent 고급 사용법

#### Agent SDK를 통한 프로그래밍 방식

```javascript
import { query } from '@anthropic-ai/claude-agent-sdk';
import fs from 'fs';

async function runPipeline() {
  const result = query({
    prompt: 'PR 검증: 보안 리뷰 후 유닛 테스트 실행',
    options: {
      agents: {
        'code-reviewer': {
          description: '코드 변경 후 PROACTIVELY 사용; JSON 형식 출력',
          prompt: fs.readFileSync('./.claude/agents/code-reviewer.md', 'utf8'),
          tools: ['Read', 'Bash(git:*)'],
          model: 'sonnet'
        },
        'test-runner': {
          description: '테스트 스위트 실행 및 실패 테스트 요약',
          prompt: `테스트 러너입니다. 
                   테스트를 실행하고 JSON { summary, failing_tests[] } 반환`,
          tools: ['Bash(npm test:*)']
        }
      }
    }
  });

  for await (const message of result) {
    console.log(message);
  }
}

runPipeline().catch(console.error);
```

#### 플러그인으로 Subagent 배포

```bash
# .claude/plugins/team-agents/plugin.json
{
  "name": "team-agents",
  "version": "1.0.0",
  "agents": [
    "agents/code-reviewer.md",
    "agents/test-runner.md"
  ]
}
```

---

## 7. 실전 활용 예제

### 7.1 코드 리뷰 자동화

#### 시나리오: GitHub PR 자동 리뷰

```bash
# GitHub Actions 워크플로우
# .github/workflows/claude-review.yml

name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      
      - name: Run Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          git diff origin/main...HEAD | \
          claude -p "이 PR을 검토하고 다음 JSON 형식으로 결과를 반환하세요:
          {
            \"summary\": \"전체 요약\",
            \"security_issues\": [],
            \"performance_issues\": [],
            \"suggestions\": []
          }" \
          --output-format json > review.json
      
      - name: Post Review Comment
        uses: actions/github-script@v6
        with:
          script: |
            const review = require('./review.json');
            const comment = `## 🤖 Claude Code Review\n\n${review.summary}`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

### 7.2 테스트 자동 생성

#### 시나리오: 유닛 테스트 자동 생성

```bash
# test-generator.sh
#!/bin/bash

# 테스트가 없는 파일 찾기
files_without_tests=$(find src -name "*.js" -type f | while read file; do
  test_file="${file/src/tests}"
  test_file="${test_file/.js/.test.js}"
  if [ ! -f "$test_file" ]; then
    echo "$file"
  fi
done)

# 각 파일에 대한 테스트 생성
echo "$files_without_tests" | while read file; do
  claude -p "이 파일에 대한 포괄적인 유닛 테스트를 생성하세요:
  
  $(cat $file)
  
  다음을 포함하세요:
  1. 정상 케이스
  2. 엣지 케이스
  3. 에러 케이스
  4. 목(mock) 필요시 Jest 사용
  
  파일명: ${file/src/tests/.test}" \
  --max-turns 2
done
```

### 7.3 리팩토링 자동화

#### 시나리오: 레거시 코드 현대화

```bash
# modernize-codebase.sh
#!/bin/bash

# Subagent 정의
cat > .claude/agents/modernizer.md << 'EOF'
---
name: modernizer
description: "레거시 코드를 현대적 패턴으로 리팩토링"
model: sonnet
tools:
  - Read
  - Write
  - Bash(git:*)
---

# Code Modernizer

레거시 코드를 다음과 같이 현대화합니다:
1. ES6+ 문법 사용
2. 비동기 처리 개선 (async/await)
3. 모듈 시스템 현대화
4. 타입 안정성 추가
5. 베스트 프랙티스 적용

## 프로세스
1. 파일 분석
2. 문제점 식별
3. 리팩토링 수행
4. 테스트 확인
5. 변경사항 커밋
EOF

# 실행
claude "@modernizer 이 디렉토리의 모든 .js 파일을 현대화해주세요"
```

### 7.4 문서 자동 생성

#### 시나리오: API 문서 자동 생성

```bash
# generate-api-docs.sh
#!/bin/bash

# 모든 API 라우트 파일 찾기
api_files=$(find src/routes -name "*.js")

# 각 파일에 대한 문서 생성
for file in $api_files; do
  claude -p "이 API 라우트 파일을 분석하여 OpenAPI 3.0 스펙 문서를 생성하세요:
  
  $(cat $file)
  
  다음을 포함하세요:
  1. 엔드포인트 경로
  2. HTTP 메서드
  3. 요청 파라미터
  4. 응답 스키마
  5. 에러 코드
  6. 예제
  
  YAML 형식으로 출력하세요." \
  --output-format text >> docs/api.yaml
done

echo "API 문서가 docs/api.yaml에 생성되었습니다."
```

### 7.5 데이터베이스 마이그레이션

#### 시나리오: 스키마 변경 자동 마이그레이션

```bash
# Subagent 정의
cat > .claude/agents/db-migrator.md << 'EOF'
---
name: db-migrator
description: "데이터베이스 스키마 변경 및 마이그레이션 생성"
model: sonnet
tools:
  - Read
  - Write
  - Bash(npx prisma:*)
  - Bash(npm run migrate:*)
---

# Database Migrator

데이터베이스 마이그레이션을 안전하게 처리합니다.

## 프로세스
1. 스키마 변경 분석
2. 마이그레이션 SQL 생성
3. 롤백 스크립트 생성
4. 데이터 무결성 검증
5. 테스트 실행
EOF

# 실행
claude "@db-migrator User 모델에 email_verified 필드를 추가하고 
마이그레이션을 생성해주세요. 기본값은 false입니다."
```

### 7.6 보안 감사 자동화

#### 시나리오: 정기 보안 스캔

```bash
# security-audit.sh
#!/bin/bash

cat > .claude/agents/security-auditor.md << 'EOF'
---
name: security-auditor
description: "종합 보안 감사 수행"
model: opus
tools:
  - Read
  - Bash(npm audit)
  - Bash(snyk test)
---

# Security Auditor

## 검사 항목
1. OWASP Top 10
2. 의존성 취약점
3. 하드코딩된 시크릿
4. SQL 인젝션 위험
5. XSS 취약점
6. CSRF 보호
7. 인증/인가 문제

## 출력
JSON 형식으로 심각도별 이슈 보고
EOF

# 주간 감사 실행
claude "@security-auditor 전체 코드베이스에 대한 보안 감사를 수행하고 
보고서를 security-report-$(date +%Y%m%d).json에 저장하세요" \
--output-format json
```

### 7.7 성능 최적화

#### 시나리오: 성능 병목 찾기 및 최적화

```bash
# Subagent 정의
cat > .claude/agents/performance-optimizer.md << 'EOF'
---
name: performance-optimizer
description: "성능 병목 분석 및 최적화"
model: sonnet
tools:
  - Read
  - Write
  - Bash(npm run benchmark:*)
  - Bash(node --prof:*)
---

# Performance Optimizer

## 분석 영역
1. 알고리즘 복잡도
2. 데이터베이스 쿼리 (N+1 문제)
3. 메모리 사용
4. 네트워크 호출
5. 캐싱 기회
6. 번들 크기

## 프로세스
1. 프로파일링 실행
2. 병목 식별
3. 최적화 제안
4. 구현 및 벤치마크
5. 개선 효과 측정
EOF

# 실행
claude "@performance-optimizer API 응답 시간이 느립니다. 
분석하고 최적화해주세요."
```

### 7.8 CI/CD 파이프라인 통합

#### 시나리오: 배포 전 자동 검증

```yaml
# .github/workflows/deploy-validation.yml
name: Deploy Validation

on:
  push:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Comprehensive Validation
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # 다중 Subagent 병렬 실행
          claude -p "다음 검증을 병렬로 수행하세요:
          1. @code-reviewer: 코드 품질 검토
          2. @security-auditor: 보안 검사
          3. @test-runner: 전체 테스트 실행
          4. @performance-optimizer: 성능 회귀 검사
          
          모든 결과를 통합하여 배포 가능 여부를 판단하고
          JSON 형식으로 보고서를 생성하세요." \
          --output-format json > validation-report.json
      
      - name: Check Validation Result
        run: |
          if jq -e '.deploy_approved == false' validation-report.json; then
            echo "배포 검증 실패"
            exit 1
          fi
      
      - name: Deploy
        if: success()
        run: npm run deploy
```

### 7.9 멀티 에이전트 협업

#### 시나리오: 복잡한 기능 개발

```bash
# feature-development.sh
#!/bin/bash

# 여러 Subagent가 협업하여 기능 개발
claude "새로운 결제 기능을 구현하려고 합니다. 다음 단계로 진행하세요:

1. @architect: 아키텍처 설계 및 기술 스택 제안
   - 결과를 docs/payment-architecture.md에 저장

2. @backend-dev: API 엔드포인트 구현
   - RESTful API 설계
   - 데이터베이스 스키마 생성
   - 트랜잭션 로직 구현

3. @frontend-dev: UI 컴포넌트 구현
   - 결제 폼
   - 결제 상태 표시
   - 에러 핸들링

4. @test-engineer: 테스트 작성
   - 유닛 테스트
   - 통합 테스트
   - E2E 테스트

5. @security-auditor: 보안 검토
   - PCI DSS 컴플라이언스 검증
   - 취약점 스캔

각 단계가 완료되면 다음 단계로 진행하고,
최종적으로 전체 구현 보고서를 생성하세요."
```

### 7.10 자동화된 버그 수정

#### 시나리오: 에러 로그 분석 및 자동 수정

```bash
# auto-bug-fix.sh
#!/bin/bash

# 최근 에러 로그 분석
tail -n 1000 /var/log/app.log | claude -p "
이 로그에서 반복되는 에러를 식별하고,
각 에러에 대해:
1. 근본 원인 분석
2. 영향 범위 평가
3. 수정 제안
4. 우선순위 설정

결과를 JSON 형식으로 반환하세요.
" --output-format json > error-analysis.json

# 높은 우선순위 버그 자동 수정
jq -r '.high_priority_bugs[] | .id' error-analysis.json | while read bug_id; do
  claude "@debugger bug $bug_id를 조사하고 가능하면 자동으로 수정하세요.
  수정이 불가능하면 상세한 수정 가이드를 제공하세요."
done
```

---

## 8. 고급 기능 (LSP, Hooks, Skills)

### 8.1 Language Server Protocol (LSP) 통합

#### 8.1.1 LSP란?

LSP는 실시간 코드 진단, 정의 찾기, 참조 추적을 제공합니다.

**주요 기능:**
- 실시간 오류 검출
- 코드 완성
- 정의로 이동
- 참조 찾기
- 리팩토링 지원

#### 8.1.2 LSP 설정

```bash
# 버전 확인 (2.0.55 이상 필요)
claude --version

# LSP 서버 자동 시작
claude
```

**설정 파일:**

```json
// .claude/settings.json
{
  "lsp": {
    "enabled": true,
    "servers": {
      "typescript": {
        "command": "typescript-language-server",
        "args": ["--stdio"]
      },
      "python": {
        "command": "pylsp"
      }
    }
  }
}
```

#### 8.1.3 LSP 활용 예제

```bash
# Claude가 자동으로 LSP 정보 활용
"이 함수의 모든 참조를 찾아서 리팩토링해주세요"
"타입 오류를 수정해주세요"
"사용하지 않는 import를 제거해주세요"
```

### 8.2 Hooks (자동화 트리거)

#### 8.2.1 Hooks란?

Hooks는 특정 이벤트 발생 시 자동으로 실행되는 스크립트입니다.

**Hook 이벤트 타입:**
- `ToolUse`: 도구 사용 전
- `ToolResult`: 도구 사용 후
- `SubagentStart`: Subagent 시작 시
- `SubagentStop`: Subagent 종료 시
- `PermissionRequest`: 권한 요청 시

#### 8.2.2 Hook 설정

**프로젝트 레벨 Hooks:**

```json
// .claude/settings.json
{
  "hooks": {
    "ToolUse": [
      {
        "matcher": "Write(*)",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint:fix"
          }
        ]
      }
    ],
    "ToolResult": [
      {
        "matcher": "Bash(git commit:*)",
        "hooks": [
          {
            "type": "command",
            "command": "npm test"
          }
        ]
      }
    ]
  }
}
```

#### 8.2.3 보안 Hooks 예제

```json
{
  "hooks": {
    "ToolUse": [
      {
        "matcher": "Read(*)",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/check-file-access.sh",
            "stdin": true
          }
        ]
      },
      {
        "matcher": "Bash(rm:*)",
        "hooks": [
          {
            "type": "reject",
            "message": "rm 명령어는 보안상의 이유로 차단되었습니다."
          }
        ]
      }
    ]
  }
}
```

**보안 스크립트 예시:**

```bash
#!/bin/bash
# scripts/check-file-access.sh

# stdin으로 전달된 JSON 파싱
input=$(cat)
file_path=$(echo "$input" | jq -r '.path')

# 위험한 파일 접근 차단
if [[ "$file_path" =~ ^/etc/|^/root/|\.env$ ]]; then
  echo "ERROR: 접근이 제한된 파일입니다: $file_path"
  exit 1
fi

# 프로젝트 외부 접근 차단
if [[ ! "$file_path" =~ ^$(pwd) ]]; then
  echo "ERROR: 프로젝트 디렉토리 외부 접근이 차단되었습니다"
  exit 1
fi

exit 0
```

#### 8.2.4 자동화 Hooks 예제

```json
{
  "hooks": {
    "ToolUse": [
      {
        "matcher": "Write(*.js)",
        "hooks": [
          {
            "type": "command",
            "command": "npm run format",
            "description": "코드 포맷팅"
          },
          {
            "type": "command",
            "command": "npm run lint:fix",
            "description": "린트 자동 수정"
          }
        ]
      },
      {
        "matcher": "Write(*.test.js)",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --related",
            "description": "관련 테스트 실행"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '✓ Subagent 작업 완료' >> .claude/activity.log"
          }
        ]
      }
    ]
  }
}
```

#### 8.2.5 PermissionRequest Hook

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "Bash(git push:*)",
        "hooks": [
          {
            "type": "approve",
            "condition": {
              "type": "command",
              "command": "./scripts/check-ci-status.sh"
            }
          }
        ]
      },
      {
        "matcher": "Bash(npm publish:*)",
        "hooks": [
          {
            "type": "deny",
            "message": "npm publish는 수동으로만 실행 가능합니다."
          }
        ]
      }
    ]
  }
}
```

### 8.3 Skills (재사용 가능한 모듈)

#### 8.3.1 Skills란?

Skills는 작업별 재사용 가능한 모듈로, 여러 애플리케이션, API, 팀 워크플로우에서 공유할 수 있습니다.

**Skills의 장점:**
- 재사용성: 여러 프로젝트에서 공유
- 표준화: 팀 전체의 일관된 작업 방식
- 모듈화: 특정 도메인에 특화
- 유지보수: 중앙 관리

#### 8.3.2 Skill 생성

**Skill 파일 구조:**

`.claude/skills/security-checklist.md`

```markdown

---
name: security-checklist
description: "OWASP Top 10 기반 보안 체크리스트"
version: 1.0.0
---

# Security Checklist

## OWASP Top 10 검사 항목

### 1. Injection
- [ ] SQL 쿼리에 파라미터 바인딩 사용
- [ ] 사용자 입력 검증
- [ ] ORM 사용

### 2. Broken Authentication
- [ ] 강력한 비밀번호 정책
- [ ] 다중 인증 (MFA)
- [ ] 세션 관리

### 3. Sensitive Data Exposure
- [ ] 전송 중 암호화 (TLS)
- [ ] 저장 시 암호화
- [ ] 민감 데이터 로깅 금지

### 4. XML External Entities (XXE)
- [ ] XML 파서 보안 설정
- [ ] DTD 처리 비활성화

### 5. Broken Access Control
- [ ] 권한 검증
- [ ] 리소스 접근 제어
- [ ] IDOR 방지

### 6. Security Misconfiguration
- [ ] 기본 계정 제거
- [ ] 불필요한 기능 비활성화
- [ ] 보안 헤더 설정

### 7. Cross-Site Scripting (XSS)
- [ ] 출력 인코딩
- [ ] Content Security Policy
- [ ] 사용자 입력 검증

### 8. Insecure Deserialization
- [ ] 신뢰할 수 없는 데이터 역직렬화 금지
- [ ] 무결성 검증

### 9. Using Components with Known Vulnerabilities
- [ ] 의존성 스캔 (npm audit, Snyk)
- [ ] 정기적 업데이트

### 10. Insufficient Logging & Monitoring
- [ ] 보안 이벤트 로깅
- [ ] 모니터링 설정
- [ ] 알림 구성
```

#### 8.3.3 Skill 사용

**Subagent에서 Skill 사용:**

```markdown
---
name: security-auditor
description: "보안 감사"
skills:
  - security-checklist
  - owasp-testing-guide
---

# Security Auditor

security-checklist 스킬을 사용하여 체계적으로 감사합니다.
```

**명시적 Skill 로드:**

```bash
claude --skills security-checklist,performance-patterns
```

#### 8.3.4 Skill 예제 모음

**1. Git 워크플로우 Skill**

`.claude/skills/git-workflow.md`

~~~markdown

---
name: git-workflow
description: "표준 Git 워크플로우"
---

# Git Workflow Standards

## 브랜치 전략
- `main`: 프로덕션 코드
- `develop`: 개발 코드
- `feature/*`: 기능 개발
- `bugfix/*`: 버그 수정
- `hotfix/*`: 긴급 수정

## 커밋 메시지 형식
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type
- feat: 새 기능
- fix: 버그 수정
- docs: 문서
- style: 포맷팅
- refactor: 리팩토링
- test: 테스트
- chore: 기타

## PR 프로세스
1. feature 브랜치 생성
2. 변경사항 커밋
3. PR 생성
4. 코드 리뷰
5. CI 통과
6. develop 병합
~~~

**2. API 설계 Skill**

`.claude/skills/api-design.md`

~~~markdown

---
name: api-design
description: "RESTful API 설계 가이드"
---

# API Design Guidelines

## RESTful 원칙
1. 리소스 기반 URL
2. HTTP 메서드 사용
3. 상태코드 적절히 사용
4. 버전 관리

## URL 구조
```
GET    /api/v1/users          # 목록 조회
GET    /api/v1/users/:id      # 단일 조회
POST   /api/v1/users          # 생성
PUT    /api/v1/users/:id      # 전체 수정
PATCH  /api/v1/users/:id      # 부분 수정
DELETE /api/v1/users/:id      # 삭제
```

## 응답 형식
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

## 에러 처리
- 400: 잘못된 요청
- 401: 인증 필요
- 403: 권한 없음
- 404: 리소스 없음
- 500: 서버 오류
~~~

**3. 테스트 전략 Skill**

`.claude/skills/testing-strategy.md`

~~~markdown

---
name: testing-strategy
description: "종합 테스트 전략"
---

# Testing Strategy

## 테스트 피라미드
```
       /\
      /E2E\      10%
     /------\
    / Integration\  20%
   /--------------\
  /   Unit Tests   \  70%
 /------------------\
```

## 유닛 테스트
- 각 함수/메서드 테스트
- 순수 함수 우선
- 목(mock) 최소화
- 커버리지 80% 이상

## 통합 테스트
- API 엔드포인트 테스트
- 데이터베이스 연동 테스트
- 외부 서비스 목 사용

## E2E 테스트
- 핵심 사용자 플로우
- 크리티컬 패스
- CI에서 자동 실행

## 테스트 네이밍
```javascript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', () => {});
    it('should throw error with invalid email', () => {});
    it('should hash password before saving', () => {});
  });
});
```
~~~

#### 8.3.5 Skill 배포 및 공유

**플러그인으로 패키징:**

```json
// .claude/plugins/team-skills/plugin.json
{
  "name": "team-skills",
  "version": "1.0.0",
  "description": "팀 공용 스킬 모음",
  "skills": [
    "skills/security-checklist.md",
    "skills/git-workflow.md",
    "skills/api-design.md",
    "skills/testing-strategy.md"
  ],
  "author": "Your Team",
  "repository": "https://github.com/yourteam/claude-skills"
}
```

**설치 및 사용:**

```bash
# 플러그인 설치
claude plugins install ./team-skills

# 또는 Git 저장소에서
claude plugins install github:yourteam/claude-skills

# 사용
claude --skills security-checklist,git-workflow
```

### 8.4 메모리 (CLAUDE.md / CLAUDE.local.md)

#### 8.4.1 메모리 시스템 개요

Claude Code는 두 가지 메모리 유형을 지원합니다:

**1. 프로젝트 메모리 (CLAUDE.md)**
- Git으로 관리
- 팀과 공유
- 프로젝트 규칙, 스타일 가이드

**2. 사용자 메모리 (CLAUDE.local.md)**
- Git에서 제외 (`.gitignore`)
- 개인 설정, 로컬 경로

#### 8.4.2 메모리 파일 구조

```bash
project/
├── CLAUDE.md              # 프로젝트 메모리
├── CLAUDE.local.md        # 사용자 메모리 (.gitignore)
├── src/
│   └── utils/
│       └── CLAUDE.md      # 하위 디렉토리 메모리
└── .gitignore
```

#### 8.4.3 재귀적 메모리 로딩

Claude Code는 현재 디렉토리부터 루트까지 모든 CLAUDE.md 파일을 로드합니다.

```bash
# claude 실행 위치: ./project/src/utils/
# 로드되는 메모리:
# 1. ./project/CLAUDE.md
# 2. ./project/src/CLAUDE.md (있는 경우)
# 3. ./project/src/utils/CLAUDE.md
```

#### 8.4.4 메모리 파일 예시

**프로젝트 메모리 (CLAUDE.md):**

```markdown
# Project Guidelines

## 코딩 스타일
- Prettier + ESLint 사용
- 2 spaces 들여쓰기
- 세미콜론 사용

## 아키텍처
- MVC 패턴
- Service Layer 분리
- Repository Pattern

## Git 규칙
- Conventional Commits
- PR 필수
- CI 통과 후 병합

## 테스트
- 커버리지 80% 유지
- TDD 권장

## 금지 사항
- `any` 타입 사용 금지
- `console.log` 커밋 금지
- 하드코딩된 값 금지
```

**사용자 메모리 (CLAUDE.local.md):**

```markdown
# Personal Settings

## 로컬 환경
- DB: localhost:5432
- Redis: localhost:6379
- API Base: http://localhost:3000

## 개인 선호사항
- 상세한 주석 선호
- 타입 명시 선호

## 자주 사용하는 명령어
- npm run dev:local
- npm test -- --watch
```

#### 8.4.5 메모리 관리 명령어

```bash
# 메모리 보기/편집
/memory

# 메모리 파일 직접 편집
vim CLAUDE.md
vim CLAUDE.local.md
```

---

## 9. 트러블슈팅

### 9.1 설치 문제

#### 문제: npm 설치 실패

```bash
# 증상
npm install -g @anthropic-ai/claude-code
# 권한 오류

# 해결
sudo npm install -g @anthropic-ai/claude-code
# 또는 nvm 사용
nvm use 18
npm install -g @anthropic-ai/claude-code
```

#### 문제: 버전 호환성

```bash
# Node.js 버전 확인
node --version  # 18 이상 필요

# Node.js 업데이트
nvm install 18
nvm use 18
```

### 9.2 인증 문제

#### 문제: API 키 오류

```bash
# 환경 변수 확인
echo $ANTHROPIC_API_KEY

# 환경 변수 설정
export ANTHROPIC_API_KEY="sk-ant-..."

# 영구 설정 (.bashrc 또는 .zshrc)
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc
source ~/.zshrc
```

### 9.3 성능 문제

#### 문제: 느린 응답 속도

**원인 및 해결:**

```bash
# 1. 컨텍스트 오버플로우
/clear                    # 컨텍스트 초기화
/compact                  # 대화 압축

# 2. 무거운 모델 사용
claude --model haiku     # 더 빠른 모델 사용

# 3. 너무 많은 파일 접근
claude --add-dir ./src   # 필요한 디렉토리만 지정
```

#### 문제: 메모리 부족

```bash
# 최대 턴 수 제한
claude -p --max-turns 3 "query"

# 출력 형식 최적화
claude -p --output-format json "query"
```

### 9.4 권한 문제

#### 문제: 도구 권한 거부

```bash
# 권한 확인
/allowed-tools

# 도구 허용
claude --allowedTools "Bash(git:*)" "Write" "Read"

# 설정 파일에 추가
# .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Write",
      "Read"
    ]
  }
}
```

### 9.5 Subagent 문제

#### 문제: Subagent 호출 안 됨

```bash
# 명시적 호출 사용

# description 개선
# BEFORE:
description: "코드 작업"

# AFTER:
description: "코드 변경 후 PROACTIVELY 사용하여 리뷰 수행"
```

### 9.6 MCP 문제

#### 문제: MCP 서버 연결 실패

```bash
# MCP 설정 확인
claude mcp list

# MCP 재설정
claude mcp reset-project-choices

# 로그 확인
tail -f ~/.claude/logs/mcp.log
```

### 9.7 일반적인 오류

#### 오류: "Context window exceeded"

```bash
# 해결책
/clear                    # 컨텍스트 초기화
/compact "keep essentials"  # 압축

# 예방
claude -p --max-turns 5   # 턴 수 제한
```

#### 오류: "Rate limit exceeded"

```bash
# 대기 후 재시도
sleep 60

# 또는 다른 모델 사용
claude --model haiku     # 더 저렴한 모델
```

### 9.8 디버깅 팁

```bash
# 상세 로깅 활성화
claude --verbose

# 통계 확인
/stats

# 비용 확인
/cos

# 설치 상태 확인
/doctor
```

---

## 10. 2025년 최신 업데이트

### 10.1 주요 업데이트 타임라인

#### 2025년 12월
- **Ultrathink Mode**: 고급 추론 기능
- **비동기 Subagents**: 병렬 실행 최적화
- **LSP 통합**: 실시간 진단
- **AutoCloud GUI**: 작업 관리 인터페이스
- **Slack 통합**: 팀 협업 강화

#### 2025년 10월
- **Plan Mode 개선**: 더 자주 질문
- **Haiku 4.5 Pro**: Pro 사용자용 Haiku 4.5
- **Explore Subagent**: Haiku 기반 빠른 탐색

#### 2025년 9월
- **Claude Sonnet 4.5**: 새로운 기본 모델
- **Checkpoints**: 작업 중단점
- **VS Code 확장 (베타)**: IDE 통합
- **Background Tasks**: 백그라운드 프로세스

### 10.2 새로운 기능 상세

#### Ultrathink Mode

```bash
# Opus 4.5에서 자동 활성화
claude --model opus

# 복잡한 문제 해결에 최적
"이 아키텍처의 성능 문제를 심층 분석하고
최적화 전략을 제시해주세요"
```

**특징:**
- 고급 추론 능력
- 복잡한 코딩 과제 해결
- 성능 최적화

#### 비동기 Subagents

```bash
# 여러 Subagent 동시 실행
"@code-reviewer, @security-auditor, @test-runner를 
병렬로 실행하여 이 PR을 검증해주세요"
```

**장점:**
- 멀티태스킹
- 병렬 실행으로 시간 절약
- 대규모 프로젝트에 효율적

#### AutoCloud GUI

```bash
# 브라우저 기반 작업 관리
# https://autocloud.anthropic.com
```

**기능:**
- 직관적인 작업 관리
- 시각적 워크플로우
- 팀 협업 도구

#### Slack 통합

```bash
# Slack에서 Claude Code 사용
/claude "코드 리뷰해줘"
```

**기능:**
- Slack 채널에서 직접 사용
- 팀원과 결과 공유
- 워크플로우 자동화

### 10.3 성능 개선

#### 컨텍스트 윈도우 추적

```bash
# 실시간 진행 표시줄
# 컨텍스트 윈도우 사용량 시각화

/stats                    # 상세 통계 확인
```

#### 동적 모델 선택

```bash
# Haiku 4.5의 스마트 모델 전환
# Plan 모드: Sonnet 자동 사용
# 실행: Haiku 사용 (비용 절감)
```

### 10.4 모바일 및 브라우저 지원

#### Chrome 통합

```bash
/chrome                   # Chrome에서 테스트/디버그
```

#### Android 앱

- 모바일에서 작업 관리
- 이동 중 개발 가능
- 생산성 향상

### 10.5 엔터프라이즈 기능

#### 엔터프라이즈 관리 설정

```bash
# 조직 전체 설정 관리
# Anthropic 팀에 문의
```

#### 엔터프라이즈 MCP

```json
{
  "mcp": {
    "enterprise": {
      "allowlist": ["github", "jira", "slack"],
      "denylist": []
    }
  }
}
```

### 10.6 2025년 통계

**개발 속도:**
- 176개 업데이트 출시 (2025년)
- 베타에서 v2.0으로 진화
- 주간 평균 3-4개 업데이트

**주요 마일스톤:**
1. CLAUDE.md 메모리 시스템
2. Plan 모드
3. Subagents
4. /context 명령어
5. Skills
6. Opus 4.5
7. LSP 통합

### 10.7 2026년 로드맵 (예상)

**예정된 기능:**
- Long-running Tasks: 장기 실행 작업 지원
- Swarming: 다중 에이전트 오케스트레이션
- 향상된 외부 도구 통합
- 더 큰 컨텍스트 윈도우 (400K-1M)
- 향상된 Compaction 품질
- 최신 지식 업데이트

---

## 11. 참고 자료

### 11.1 공식 문서

- [Claude Code 공식 문서](https://code.claude.com/docs)
- [Anthropic API 문서](https://docs.anthropic.com)
- [MCP 문서](https://docs.anthropic.com/en/docs/build-with-claude/mcp)
- [Claude Agent SDK](https://platform.claude.com/docs/agent-sdk)

### 11.2 커뮤니티 리소스

- [Awesome Claude Code](https://github.com/awesome-claude/awesome-claude-code)
- [Claude Code Examples](https://github.com/anthropics/claude-code-examples)
- [Discord Community](https://discord.gg/anthropic)

### 11.3 블로그 및 튜토리얼

- [Anthropic Engineering Blog](https://www.anthropic.com/news)
- [하이퍼리즘 기술 블로그 - Claude Code 가이드](https://tech.hyperithm.com/)
- [ClaudeLog - 변경 로그](https://claudelog.com/)

### 11.4 GitHub 저장소

- [공식 Claude Code GitHub](https://github.com/anthropics/claude-code)
- [Claude Code Cheat Sheet](https://github.com/Njengah/claude-code-cheat-sheet)
- [MCP Registry](https://github.com/mcp)

### 11.5 비디오 자료

- [WorldofAI - Claude Code 업데이트](https://youtube.com/@WorldofAI)
- [Anthropic YouTube Channel](https://youtube.com/@Anthropic)

### 11.6 도서 및 코스

- [Claude Code Mastery (온라인 코스)](https://example.com/course)
- [AI-Powered Development with Claude](https://example.com/book)

---

## 부록 A: 명령어 빠른 참조

### CLI 명령어

| 명령어 | 설명 |
|--------|------|
| `claude` | REPL 시작 |
| `claude "query"` | 프롬프트와 함께 시작 |
| `claude -p "query"` | 프린트 모드 |
| `claude -c` | 최근 대화 계속 |
| `claude -r "id"` | 세션 재개 |
| `claude update` | 업데이트 |
| `claude --version` | 버전 확인 |

### CLI 플래그

| 플래그 | 설명 |
|--------|------|
| `--model` | 모델 지정 |
| `--add-dir` | 디렉토리 추가 |
| `--allowedTools` | 도구 허용 |
| `--disallowedTools` | 도구 차단 |
| `--output-format` | 출력 형식 |
| `--max-turns` | 턴 수 제한 |
| `--verbose` | 상세 로깅 |

### Slash 명령어

| 명령어 | 설명 |
|--------|------|
| `/help` | 도움말 |
| `/exit` | 종료 |
| `/clear` | 초기화 |
| `/config` | 설정 |
| `/doctor` | 상태 확인 |
| `/cos` | 비용/시간 |
| `/mcp` | MCP 관리 |
| `/agents` | Subagent 관리 |
| `/memory` | 메모리 편집 |
| `/stats` | 통계 |
| `/context` | 컨텍스트 정보 |

---

## 부록 B: 설정 파일 템플릿

### .claude/settings.json

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Bash(git:*)",
      "Bash(npm:*)"
    ],
    "deny": [
      "Bash(rm:*)",
      "Bash(sudo:*)"
    ]
  },
  "hooks": {
    "ToolUse": [
      {
        "matcher": "Write(*)",
        "hooks": [
          {
            "type": "command",
            "command": "npm run format"
          }
        ]
      }
    ]
  },
  "mcp": {
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"]
      }
    }
  }
}
```

### CLAUDE.md 템플릿

```markdown
# Project Name

## 프로젝트 개요
[프로젝트 설명]

## 기술 스택
- Backend: Node.js, Express
- Frontend: React, TypeScript
- Database: PostgreSQL

## 코딩 스타일
- Prettier + ESLint
- 2 spaces 들여쓰기
- 세미콜론 사용

## 아키텍처
- MVC 패턴
- Service Layer 분리

## Git 규칙
- Conventional Commits
- PR 필수

## 금지 사항
- `any` 타입 금지
- `console.log` 커밋 금지
```

---

## 부록 C: 용어집

| 용어 | 설명 |
|------|------|
| **REPL** | Read-Eval-Print Loop, 대화형 인터페이스 |
| **Subagent** | 전문화된 AI 어시스턴트 |
| **MCP** | Model Context Protocol |
| **LSP** | Language Server Protocol |
| **Hook** | 자동 실행 스크립트 |
| **Skill** | 재사용 가능한 모듈 |
| **Context Window** | 모델이 한 번에 처리할 수 있는 토큰 수 |
| **Token** | 텍스트의 기본 단위 (~4글자) |
| **Checkpoint** | 작업 저장 지점 |
| **Compaction** | 대화 압축 |

---

## 맺음말

이 가이드는 Claude Code를 제로부터 마스터까지 학습할 수 있도록 설계되었습니다. 

**학습 경로:**
1. 기본 명령어부터 시작 (레벨 1-3)
2. 설정 및 권한 관리 학습 (레벨 4-6)
3. Subagents로 자동화 구현 (레벨 6)
4. 고급 기능 활용 (LSP, Hooks, Skills)
5. 실전 프로젝트 적용

**추가 학습 자료:**
- 공식 문서 정기 확인
- 커뮤니티 참여
- 실제 프로젝트 적용

**피드백 및 기여:**
- GitHub: [Issues](https://github.com/anthropics/claude-code/issues)
- Discord: [Anthropic Community](https://discord.gg/anthropic)

**최신 업데이트 확인:**
```bash
claude --version
claude update
```

---

**작성 일자: 2026-01-19**

**버전: 1.0.0**

**라이센스: MIT**

---

> **참고:** 이 문서는 2026년 1월 기준으로 작성되었으며, Claude Code는 지속적으로 업데이트됩니다. 최신 정보는 [공식 문서](https://code.claude.com/docs)를 참조하세요.
