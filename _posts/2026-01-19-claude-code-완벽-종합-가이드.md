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

```markdown
