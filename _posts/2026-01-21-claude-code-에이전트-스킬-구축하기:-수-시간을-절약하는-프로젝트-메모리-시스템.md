---
title: "Claude Code 에이전트 스킬 구축하기: 수 시간을 절약하는 프로젝트 메모리 시스템"
date: 2026-01-21 20:30:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,  claude-code,  claude-skills,  agent-skills,  Medium,  Claude.write]
---


## 원문

[**Build Your First Claude Code Agent Skill: A Simple Project Memory System That Saves Hours**](https://pub.spillwave.com/build-your-first-claude-code-skill-a-simple-project-memory-system-that-saves-hours-1d13f21aff9e)

## 들어가며: AI 기억 상실의 대가

화요일 밤 11시, 당신은 낯익은 에러 메시지를 응시하고 있습니다. "Connection refused on port 5432". 이 문제를 전에 본 적이 있다는 확신이 듭니다. 분명히 해결했던 문제인데, 어디서였을까요? 언제였을까요? 커밋 메시지는 단지 "fixed db connection"이라고만 적혀 있습니다. Stack Overflow는 12개의 서로 다른 답변을 제시하고, AI 어시스턴트는 이미 시도해본 해결책들을 다시 제안합니다.

이것이 바로 AI 기억 상실의 실체이며, 당신이 인식하는 것보다 훨씬 많은 비용을 치르게 만듭니다.

모든 AI 코딩 어시스턴트는 동일한 한계를 공유합니다. 세션 간의 모든 것을 잊어버린다는 것입니다. 새로운 채팅을 시작하면 Claude는 어제 당신이 스테이징 환경에서 포트 5432가 아닌 5433을 사용한다는 것을 발견하는 데 45분을 소비했다는 사실을 기억하지 못합니다. 팀의 기존 전문성 때문에 MongoDB가 아닌 PostgreSQL을 선택했다는 것도 기억하지 못합니다. "connection refused" 에러가 항상 VPN 연결이 끊어졌음을 의미한다는 것도 모릅니다.

힘들게 얻은 모든 지식이 사라집니다. 매번. 계속해서.

저는 YouTube 영상을 보다가 간단한 해결책을 발견했습니다. 영상 제작자는 작은 마크다운 파일들을 CLAUDE.md에서 링크하여 의사결정, 버그, 핵심 사실들을 추적했습니다. 그 순간 한 가지 아이디어가 떠올랐습니다. 이것을 자동으로 관리하는 스킬을 만들 수 있다면 어떨까?

그 아이디어가 제가 Claude Code를 위해 작성한 첫 번째 스킬 중 하나인 project-memory가 되었습니다. 놀랍도록 단순함에도 불구하고(300줄 미만), 이 스킬은 제게 수많은 시간을 절약해주었습니다. 이 튜토리얼은 여러분만의 프로젝트 메모리 스킬을 구축하는 방법과, 더 중요하게는 실제 문제를 해결하는 스킬을 만드는 사고방식을 보여줍니다.

## AI 기억 상실의 진짜 비용

익숙한 상황을 그려보겠습니다.

**1개월차: 발견의 시간**

에러가 발생합니다: "CORS policy blocked request from localhost:3000". 2시간 동안 프록시 설정, 헤더 변경, nginx 재작성을 시도하며 디버깅합니다. 마침내 해결책을 찾습니다. package.json에 프록시 설정을 추가하는 것이었습니다.

**6개월차: 데자뷔의 순간**

똑같은 에러가 다시 발생합니다: "CORS policy blocked request from localhost:3000". 개발자는 "이거 낯익은데... 우리가 어떻게 고쳤더라?"라고 생각합니다. 오래된 커밋을 검색하고, Stack Overflow를 다시 확인하고, 메모리가 없는 Claude에게 묻습니다. 정확히 같은 프록시 설정 해결책을 재발견하는 데 1시간이 걸립니다.

익숙하게 들리나요? 불편한 진실은 AI 코드 어시스턴트가 실제로 이 문제를 더 악화시킨다는 것입니다. 메모리가 없으면 다음과 같은 일이 발생합니다.

- 각 새로운 채팅 세션이 지식 제로에서 시작합니다
- 모든 버그가 처음 해결하는 것처럼 느껴집니다
- 해결책이 반복적으로 "재발견"됩니다 (저는 6개월 동안 같은 CORS 문제를 네 번 해결했습니다)
- 시간이 지나도 학습이 축적되지 않습니다

숨겨진 비용은 엄청납니다. 매주 이미 해결한 문제를 다시 해결하는 데 30분만 소비해도, 연간 26시간입니다. 시간당 100달러로 계산하면, 개발자 한 명당 연간 2,600달러의 낭비된 시간입니다.

하지만 AI 어시스턴트가 기억할 수 있다면 어떨까요?

## Claude Code 에이전트 스킬이란?

project-memory에 대해 깊이 들어가기 전에, 스킬이 실제로 무엇인지 이해해봅시다. Claude에게 특정 워크플로우를 가르치거나 도메인 지식을 제공하고 싶었던 적이 있다면, 스킬이 바로 그 해답입니다.

에이전트 스킬은 단순히 SKILL.md 파일과 선택적 지원 리소스를 포함하는 폴더입니다. SKILL.md 파일은 두 부분으로 구성됩니다.

첫째, YAML 프론트매터입니다. 이는 Claude에게 스킬을 언제 활성화할지 알려주는 메타데이터입니다. 둘째, 마크다운 본문입니다. 스킬이 활성화되었을 때 Claude가 따르는 지침입니다.

스킬을 설치 가능한 재사용 가능한 "전문가 모드"로 생각하면 됩니다. "프로젝트 메모리 설정"이라고 말하면, Claude는 무작정 추측하지 않습니다. 정확히 그 작업을 위해 테스트된 특정 지침을 로드합니다.

### 스킬 파일 구조

SKILL.md 파일은 다음과 같은 구조를 가집니다.

파일 상단에는 YAML 프론트매터가 있습니다. 여기에는 스킬의 이름과 설명이 포함됩니다. 설명은 Claude가 이 스킬을 언제 사용해야 할지 이해하는 데 매우 중요합니다.

그 아래에는 마크다운 본문이 있습니다. 이 섹션들은 개요, 사용 시기, 핵심 기능, 예제, 성공 기준 등을 포함합니다. 이 구조화된 접근 방식은 Claude가 스킬을 정확하게 이해하고 실행할 수 있게 해줍니다.

### 스킬 저장 위치

스킬은 일반적으로 두 위치에 저장될 수 있습니다.

첫째, 사용자 홈 디렉토리의 `~/.claude/skills/`입니다. 이곳에 저장된 스킬들은 모든 프로젝트에서 전역적으로 사용 가능합니다. 코드 리뷰, 문서화와 같은 범용 스킬에 적합합니다.

둘째, 프로젝트별 스킬을 위한 `.claude/skills/`입니다. 이는 해당 프로젝트에서만 로컬로 사용됩니다. 프로젝트별 워크플로우나 팀 규칙에 사용됩니다.

실제로는 엔터프라이즈 스킬, 모노레포 수준에서 공유되는 스킬 등도 있을 수 있으며, 사용하는 제공자에 따라 달라집니다.

### 스킬 활성화 작동 방식

요청을 하면 Claude Code는 에이전트 스킬 작성을 위한 점진적 공개 패턴을 따릅니다. 필요할 때만 필요한 것만 로드합니다. 이는 강력한 기능에 접근할 수 있으면서도 상호작용을 빠르게 유지합니다.

프로세스는 세 단계로 진행됩니다.

첫 번째는 발견 단계입니다. Claude는 디렉토리를 스캔하고 메타데이터만 로드합니다. 두 번째는 매칭 단계입니다. 요청이 스킬과 일치하는지 확인합니다. 세 번째는 실행 단계입니다. SKILL.md를 로드하고, 지침을 따르고, 작업을 완료합니다.

핵심 통찰은 다음과 같습니다. Claude는 스킬 디렉토리를 스캔하고 일치하는 항목을 찾을 때까지 메타데이터(이름과 설명)만 읽습니다. 그때서야 전체 지침을 로드합니다. 즉, 일반적인 상호작용을 늦추지 않으면서 수십 개의 스킬을 설치할 수 있습니다.

## 프로젝트 메모리 에이전트 스킬

project-memory 스킬은 `docs/project_notes/`에 네 개의 전문화된 파일로 구조화된 지식 시스템을 만듭니다. 각 파일은 프로젝트 지식을 보존하고 회상하는 데 있어 뚜렷한 목적을 제공합니다.

### 메모리 파일 구조

프로젝트 메모리 시스템은 `docs/project_notes/` 폴더에 네 개의 핵심 파일을 생성합니다.

`bugs.md`는 검색 가능한 버그 데이터베이스 역할을 합니다. "BUG-018: Pulumi state drift — run pulumi refresh — yes"와 같은 항목은 문제와 해결책을 모두 문서화하여, 같은 문제가 다시 나타날 때 시간 낭비를 방지합니다. 저는 이를 프로젝트의 재발하는 문제를 추적하는 데 사용합니다.

`decisions.md` 파일은 ADR(Architectural Decision Records)을 통해 아키텍처 선택을 추적합니다. "ADR-012: 모든 차트에 D3.js 사용(팀 전문성)"과 같은 항목은 일관성을 보장하고 향후 상충되는 기술 결정을 방지합니다.

`key_facts.md`는 필수 프로젝트 세부정보에 대한 빠른 참조 역할을 합니다. "스테이징 API: https://api.staging.example.com:8443"과 같은 정보는 포트와 URL에 대한 추측을 제거합니다.

`issues.md`는 시간순 작업 로그를 유지합니다. "TICKET-456: 사용자 인증 구현 — 2024-01-15"와 같은 항목은 완료된 작업을 특정 티켓 및 날짜와 연결하는 감사 추적을 만듭니다.

이 파일들은 CLAUDE.md와 AGENTS.md 설정 파일에 링크되어 프로젝트 메모리를 구성합니다.

### 스킬의 각 부분 이해하기

#### 파트 1: 에이전트 스킬의 프론트매터 (스킬의 "언제")

```yaml
---
name: project-memory
description: Set up and maintain a structured project memory system in docs/project_notes/
  that tracks bugs with solutions, architectural decisions, key project facts, and work
  history. Use this skill when asked to "set up project memory", "track our decisions",
  "log a bug fix", "update project memory", or "initialize memory system".
---
```

설명 필드가 중요합니다. 이것은 스킬의 가장 중요한 부분이라고 할 수 있습니다. Claude에게 언제 활성화할지 알려줍니다. 설명에 포함된 특정 트리거 문구들을 주목하세요: "set up project memory", "track our decisions", "log a bug fix". 이 중 하나를 말하면 Claude는 일치를 인식하고 전체 지침을 로드합니다.

자신만의 스킬 설명을 작성할 때는 사용자가 자연스럽게 말할 수 있는 3-5개의 특정 문구를 포함하세요. 다양한 방법으로 활성화를 테스트해보세요. "의사결정 추적을 도와줘" vs "의사결정 추적 설정해줘" vs "아키텍처 선택을 로그하고 싶어" 등입니다.

#### 파트 2: 이 에이전트 스킬을 사용할 때 (상황적 트리거)

```markdown
## When to Use This Skill

Invoke this skill when:

- Starting a new project that will accumulate knowledge over time
- The project already has recurring bugs or decisions that should be documented
- The user asks to "set up project memory" or "track our decisions"
- Encountering a problem that feels familiar ("didn't we solve this before?")
- Before proposing an architectural change (check existing decisions first)
```

이 섹션은 트리거를 강화하고 Claude가 메모리가 유용한 더 넓은 맥락을 이해하도록 돕습니다. "이전에 이 문제를 해결하지 않았나?"라는 문구가 특히 강력합니다. 메모리가 도움이 될 좌절의 정확한 순간을 포착합니다.

#### 파트 3: 초기 설정 지침 (에이전트 스킬의 "무엇")

```markdown
## Core Capabilities

### 1. Initial Setup - Create Memory Infrastructure

When invoked for the first time in a project, create the following structure:

docs/
└── project_notes/
    ├── bugs.md         # Bug log with solutions
    ├── decisions.md    # Architectural Decision Records
    ├── key_facts.md    # Project configuration and constants
    └── issues.md       # Work log with ticket references

**Directory naming rationale:** Using `docs/project_notes/` instead of `memory/`
makes it look like standard engineering organization, not AI-specific tooling.
```

이 설계 결정은 보이는 것보다 중요합니다. 메모리를 일반 문서처럼 보이게 만드는 것입니다. `ai-memory/`나 `claude-context/`에 파일을 만들면 개발자들은 이것을 AI 도구로 보고 유지 관리하지 않을 것입니다. `docs/project_notes/`에서는 AI 지원을 사용하든 안 하든 누구나 업데이트할 수 있는 표준 엔지니어링 문서처럼 보입니다.

#### 파트 4: 메모리 항목 형식 (구조화된 지식)

스킬은 각 유형의 항목에 대해 일관된 형식을 정의합니다. 일관성은 중요합니다. Claude가 몇 달 후에도 구조화된 형식을 안정적으로 파싱할 수 있기 때문입니다.

버그 항목은 Issue(문제), Root Cause(근본 원인), Solution(해결책), Prevention(예방) 형식을 따릅니다. 의사결정 항목은 ADR 형식을 사용합니다. Context(맥락), Decision(결정), Alternatives(대안), Consequences(결과)입니다.

버그 항목의 Prevention 필드가 특히 가치 있습니다. 버그를 "우리가 고친 문제"에서 "우리가 배운 교훈"으로 변환합니다. 6개월 후 Claude는 같은 문제에 부딪히기 전에 경고할 수 있습니다.

#### 파트 5: CLAUDE.md를 위한 메모리 프로토콜 (마법)

여기서 진짜 힘이 나타납니다. 스킬은 Claude가 기본적으로 메모리를 인식하도록 CLAUDE.md를 구성합니다.

```markdown
### 2. Configure CLAUDE.md - Memory-Aware Behavior

Add or update the following section in the project's `CLAUDE.md` file:

## Project Memory System

### Memory-Aware Protocols

**Before proposing architectural changes:**
- Check `docs/project_notes/decisions.md` for existing decisions
- Verify the proposed approach doesn't conflict with past choices

**When encountering errors or bugs:**
- Search `docs/project_notes/bugs.md` for similar issues
- Apply known solutions if found
- Document new bugs and solutions when resolved

**When looking up project configuration:**
- Check `docs/project_notes/key_facts.md` for credentials, ports, URLs
- Prefer documented facts over assumptions
```

여기서 마법이 일어납니다. 이러한 프로토콜이 CLAUDE.md에 있으면 Claude는 결정이나 제안을 하기 전에 자동으로 메모리 파일을 확인합니다. "버그 로그를 확인해"라고 기억할 필요가 없습니다. 기본 동작이 됩니다.

## 실제 영향: 지식 복리

메모리가 일상적인 개발 워크플로우를 어떻게 변화시키는지 구체적인 예를 공유하겠습니다.

### 예제 1: 버그 해결 속도 (실제 수치)

**시나리오:**

10월 20일, 배포 중 Pulumi 상태 드리프트 에러가 발생했습니다. 에러 메시지는 모호했습니다: "error: update failed". 45분 동안 IAM 권한을 확인하고, 최근 변경사항을 검토하고, 다른 배포 전략을 시도한 후, 해결책을 발견했습니다. 상태 파일을 동기화하기 위해 `pulumi refresh --yes`를 실행하는 것이었습니다.

BUG-018로 문서화했습니다:

```markdown
### 2024-10-20 - BUG-018: Pulumi State Drift During Deploy

**Issue**: Deploy failed with cryptic "update failed" error after manual 
GCP console changes

**Root Cause**: Pulumi state file out of sync with actual infrastructure 
after teammate made console changes

**Solution**: Run `pulumi refresh --yes` before deploy to sync state

**Prevention**: Add `pulumi refresh --yes` to CI/CD pipeline before deploys; 
document that manual console changes require refresh
```

이틀 후인 10월 22일, 같은 에러가 나타났습니다. 이번에는:

- Claude에게 에러에 대해 말했습니다
- Claude가 bugs.md를 확인하고 BUG-018을 찾았습니다
- Claude가 `pulumi refresh --yes`를 제안했습니다
- 5분 만에 문제가 해결되었습니다

결과: 45분이 5분으로 단축되었습니다. 89%의 해결 시간 감소입니다.

### 예제 2: 아키텍처 일관성 (번들 비대화 방지)

**메모리 없이:**

사용자: "대시보드에 막대 차트를 추가해줘"
Claude: "막대 차트를 위해 Chart.js를 추가하겠습니다. 설치할게요..."
결과: 번들 크기 +85KB, 중복 차트 의존성, 일관되지 않은 스타일링

**메모리와 함께:**

사용자: "대시보드에 막대 차트를 추가해줘"
Claude: decisions.md를 확인 중...
발견: ADR-012 - 모든 차트에 D3.js 사용 (팀이 D3 전문성 보유, 이미 번들에 포함)
Claude: "모든 차트에 D3.js를 사용하기로 결정한 것을 확인했습니다. 기존 차트와 일치하도록 D3를 사용해 막대 차트를 만들겠습니다."
결과: 기존 D3.js 사용, 일관된 스타일링, 번들 증가 없음

이러한 일관성은 복리로 쌓입니다. 하나의 차트 라이브러리 결정이 수십 건의 향후 의존성 충돌을 방지합니다. 이것들은 저에게 실제 세계 시나리오입니다.

### 복리 효과

지식 캡처는 복리로 보상됩니다. 진행 과정을 보면 첫 번째 만남(2시간 디버깅)에서 문서화로, 두 번째 만남(5분), 세 번째(2분), 그리고 예방적 조언으로 이어집니다.

진행 과정을 주목하세요. 네 번째 만남에서 Claude는 단지 버그를 고치지 않습니다. 예방합니다. "이전에 상태 드리프트 문제를 일으킨 변경을 하려는 것으로 보입니다. 먼저 pulumi refresh를 실행할까요?"

프로젝트가 시간이 지나면서 더 어려워지는 것이 아니라 더 쉬워집니다. 그것이 문서화된 지식의 복리 효과입니다.

## 크로스 플랫폼: 에이전트 스킬 표준

강력한 것이 있습니다. project-memory는 Claude Code만을 위한 것이 아닙니다. Agent Skill Standard(agentskills.io)를 따르며, 이는 동일한 스킬이 14개 이상의 AI 코딩 어시스턴트에서 작동한다는 의미입니다. 스킬에는 AGENTS.md를 최신 상태로 유지하는 지침도 포함되어 있습니다.

왜 이것이 중요할까요? 팀은 종종 다른 AI 도구를 사용합니다. 한 개발자는 Claude Code를 선호할 수 있고, 다른 개발자는 Cursor를 사용하고, 세 번째는 GitHub Copilot을 좋아합니다. 표준화된 스킬을 사용하면 모두가 같은 프로젝트 메모리를 공유합니다.

### 크로스 플랫폼 설치

에이전트 스킬은 크로스 플랫폼이 되었으므로 몇 가지 범용 스킬 설치 프로그램이 있습니다. GitHub와 SkillzWave 에이전트 마켓플레이스에서 스킬을 찾고, 오픈 소스 skilz CLI를 에이전트 스킬 설치 프로그램으로 사용하여 Claude Code, Codex, Gemini, Cursor, GitHub Copilot 등에 배포할 수 있습니다.

skilz CLI로 에이전트 스킬 설치:

```bash
# skilz 설치
pip install skilz

# Claude Code용으로 전역 설치
skilz install -g https://github.com/SpillwaveSolutions/project-memory

# 특정 프로젝트에만 설치
skilz install -g https://github.com/SpillwaveSolutions/project-memory --project

# 다른 에이전트용 설치
skilz install -g https://github.com/SpillwaveSolutions/project-memory --agent codex
skilz install -g https://github.com/SpillwaveSolutions/project-memory --agent gemini
skilz install -g https://github.com/SpillwaveSolutions/project-memory --agent cursor
skilz install -g https://github.com/SpillwaveSolutions/project-memory --agent opencode
```

같은 지식을 사용하는 어떤 AI 코딩 도구에서든 접근할 수 있습니다. Claude Code에서 버그 수정을 문서화하면, Cursor나 Gemini 또는 Codex를 사용하는 팀원이 다음 날 그 지식으로부터 혜택을 받을 수 있습니다.

## 보안: 절대 저장해서는 안 되는 것

이 섹션은 매우 중요합니다. 프로젝트 메모리를 사용하기 전에 주의 깊게 읽으세요.

key_facts.md 파일은 프로젝트 설정의 편리한 조회를 위해 설계되었습니다. 그러나 조심하지 않으면 편의성이 위험을 만듭니다. 메모리 파일에 비밀을 저장하지 마세요. 이것들은 저장소 접근 권한이 있는 누구나 읽을 수 있는 버전 관리된 마크다운입니다. 스킬은 이 주의사항을 명시합니다.

### 위험 영역

key_facts.md(또는 어떤 메모리 파일)에 절대 저장해서는 안 되는 것들:

- **인증 자격 증명**: 암호, API 키, 액세스 토큰, 또는 모든 형태의 인증 비밀
- **서비스 계정 키**: GCP/AWS JSON 키 파일, 개인 키, 또는 서비스 계정 자격 증명
- **OAuth 비밀**: 클라이언트 비밀, 리프레시 토큰, 또는 OAuth 구성 비밀
- **데이터베이스 자격 증명**: 암호를 포함하는 연결 문자열 또는 모든 데이터베이스 인증 세부 정보
- **인프라 비밀**: SSH 개인 키, VPN 자격 증명, 또는 네트워크 액세스 자격 증명

### 안전 영역

key_facts.md에 저장해도 안전한 것들:

- **호스트명과 URL**: api.staging.example.com 또는 https://api.example.com/v1과 같은 공개 도메인 이름
- **포트 번호**: PostgreSQL: 5432 또는 Redis: 6379와 같은 표준 포트
- **프로젝트 식별자**: GCP 프로젝트 ID, AWS 계정 ID, 또는 유사한 민감하지 않은 식별자
- **서비스 계정 이메일 주소**: service-account@project.iam.gserviceaccount.com과 같은 신원 정보
- **환경 이름**: staging, production, 또는 dev와 같은 환경 레이블
- **공개 엔드포인트**: 공개적으로 접근 가능한 URL 또는 API 엔드포인트
- **구성 값**: 타임아웃 값, 재시도 횟수, 또는 기능 플래그와 같은 민감하지 않은 설정

### 비밀이 속한 곳

비밀은 다음과 같은 곳에 저장되어야 합니다:

- **.env 파일(gitignored)**: `DATABASE_PASSWORD=secret123`과 같은 로컬 개발 비밀에 가장 적합
- **클라우드 비밀 관리자**: 프로덕션 시스템은 GCP Secret Manager, AWS Secrets Manager, 또는 Azure Key Vault를 사용해야 함
- **CI/CD 변수**: 자동화된 파이프라인은 GitHub Actions 비밀, GitLab CI/CD 변수, 또는 유사한 것을 사용해야 함
- **암호 관리자**: 1Password, LastPass, Bitwarden, 또는 유사한 도구를 통한 팀 자격 증명 공유
- **환경 변수**: 배포 시 주입되는 런타임 비밀, 버전 제어에 절대 커밋하지 않음
- **Kubernetes 비밀**: Kubernetes Secret 객체를 사용하는 컨테이너화된 애플리케이션용
- **하드웨어 보안 모듈(HSM)**: 엔터프라이즈 환경에서 매우 민감한 암호화 키용

지금 바로 기존 key_facts.md(있다면)에서 password, secret, key, token과 같은 패턴을 검색하세요. 비밀을 발견하면 즉시 이동하고 손상된 자격 증명을 교체하세요.

## 일반적인 함정 (그리고 피하는 방법)

수십 개의 프로젝트에서 project-memory를 사용하고 다른 사람들이 채택하도록 도우면서, 같은 실수를 반복적으로 보았습니다. 좌절을 피하기 위해 이것들로부터 배우세요.

### 함정 1: 문서 유령 도시

**문제**: 열정적으로 프로젝트 메모리를 설정하고, 세 개의 항목을 추가한 다음, 다시는 건드리지 않습니다. 6개월 후, 파일들은 오래되고 쓸모없게 됩니다.

**해결책**: 메모리 업데이트를 할 일 목록이 아닌 워크플로우에 추가하세요. 모든 버그 수정 후 Claude에게 "이것을 bugs.md에 로그해줘"라고 요청하세요. 모든 아키텍처 논의 후 "이 결정에 대한 ADR을 추가해줘"라고 말하세요. 의도적이 아니라 반사적으로 만드세요.

저는 또한 워크플로우를 아키텍트 에이전트, 코드 에이전트, 그리고 중간의 인간으로 나누는 또 다른 스킬을 작성했습니다. 이 스킬은 Claude를 위한 훅 또는 OpenCode를 위한 플러그인을 설치하여 핵심 결정과 방향 전환의 로그 파일을 자동으로 추적할 수 있는 능력이 있으며, 이것을 실제로 수행했습니다. 이렇게 하면 잊어버리더라도 백그라운드 에이전트가 로그를 스캔하고 놓쳤을 수 있는 핵심 결정, 버그, key_facts를 찾을 수 있습니다.

### 함정 2: 소설 길이의 항목

**문제**: 버그 항목이 여러 페이지 에세이가 됩니다. 의사결정 기록에 전체 논의 기록이 포함됩니다. 너무 길어서 아무도 읽지 않습니다.

**해결책**: 각 항목은 30초 안에 스캔 가능해야 합니다. 구조화된 형식(Issue, Root Cause, Solution, Prevention)을 사용하세요. 더 많은 세부 사항이 필요하면 별도의 문서에 링크하세요.

### 함정 3: 모든 것 파일

**문제**: 모든 정보가 key_facts.md로 들어가고, 이것이 조직화 없이 500줄로 증가합니다.

**해결책**: 네 개의 파일을 모두 의도된 목적으로 사용하세요:

- 버그는 bugs.md로
- 의사결정은 decisions.md로
- 구성은 key_facts.md로
- 작업 기록은 issues.md로

key_facts.md가 100줄을 초과하면, 아마도 사실로 위장한 의사결정이 포함되어 있을 것입니다.

### 데자뷰 함정

**문제**: 좌절스러운 순간을 경험합니다. "이 버그를 전에 본 것 같은데. 우리가 고쳤던 것 같은데. 해결 방법이 있었던 것 같은데." 하지만 세부 사항을 기억할 수 없어서 해결책을 재발견하는 데 또 30분을 소비합니다.

**해결책**: 이것이 바로 프로젝트 메모리가 제거하는 것입니다. 그 데자뷰 느낌이 올 때, 첫 번째 반응은 "bugs.md 파일을 확인해"여야 합니다. 그 재발하는 CORS 문제? 문서화되어 있습니다. 화요일마다 실패하는 그 불안정한 테스트? 해결 방법이 기록되어 있습니다. 그 인증 엣지 케이스? 이미 해결되고 로그되었습니다.

아름다움은 프로젝트 메모리가 팀이 종종 "부족 지식"이라고 부르는 조직적 지식을 만들지만, 누군가의 머리에 갇혀 있는 대신 접근 가능하게 만든다는 것입니다. 팀이 모든 차트에 D3.js를 사용하기로 결정하면, 그 결정이 기록됩니다. 6개월 후, 새로운 사람이 합류하여 "어떤 차트 라이브러리를 사용해야 하나요?"라고 물으면, Claude는 추측하지 않습니다. decisions.md를 확인하고 올바른 답변을 제공합니다.

실제 영향: 같은 문제를 두 번 해결하지 않습니다. 일관되지 않은 기술 선택이 없습니다. "이전에 이것을 고쳤던 것 같은데"라는 좌절이 없습니다. 메모리가 지속되기 때문에 재발하는 것들이 끝납니다.

### 함정 4: 비밀 은닉처

**문제**: 개발자들이 "편의를 위해" key_facts.md에 API 키를 저장합니다. 키가 영원히 git 히스토리에 남게 됩니다. .env를 사용하고 .env를 .gitignore에 추가하세요. 우리는 문명화되었고 사회에는 규칙이 있습니다. :)

**해결책**: 위의 보안 섹션을 참조하세요. 이미 비밀을 커밋했다면 즉시 교체하세요. git 히스토리에서 제거하는 것만으로는 충분하지 않습니다. 비밀이 이미 다른 곳에 복제되었을 수 있기 때문입니다.

### 함정 5: 고립된 메모리

**문제**: 한 개발자가 프로젝트 메모리를 종교적으로 유지 관리합니다. 다른 사람들은 그것이 존재하는지 모릅니다. 지식 사일로가 형성됩니다.

**해결책**: README 또는 온보딩 문서에 프로젝트 메모리에 대한 섹션을 추가하세요. PR 리뷰에서 메모리 파일을 언급하세요: "이 버그 수정은 bugs.md에 문서화되어야 합니다."

### 함정 6: 오래된 의사결정

**문제**: ADR-003은 "모든 API에 REST 사용"이라고 말하지만 팀은 2년 전에 GraphQL로 마이그레이션했습니다. 메모리 파일에 이제 오해의 소지가 있는 정보가 포함되어 있습니다.

**해결책**: 분기별로 의사결정 기록을 검토하세요. 오래된 의사결정을 superseded로 표시하세요: **상태**: ADR-027로 대체됨. 삭제하지 마세요. 향후 개발자들이 왜 오래된 코드가 REST를 사용하는지 궁금해할 수 있습니다.

## 자신만의 에이전트 스킬 구축하기

project-memory 스킬은 모든 스킬을 만들 때 적용할 수 있는 핵심 원칙을 보여줍니다. 이 가이드라인은 수십 개의 스킬을 구축하고 어떤 것이 사용되고 어떤 것이 버려지는지 확인한 경험에서 나옵니다.

### 1. 실제로 겪고 있는 진짜 문제를 해결하기

다음과 같은 경우 스킬을 구축하세요:

- Claude(또는 팀원)에게 같은 설명을 반복하고 있을 때
- 같은 정보를 반복적으로 찾고 있을 때
- 같은 다단계 프로세스를 수동으로 따르고 있을 때
- Claude가 프로젝트에 대해 "그냥 알았으면 좋겠다"고 바랄 때

안티패턴: 가상의 문제를 위한 스킬을 구축하지 마세요. 고통을 경험하지 않았다면 올바른 해결책을 설계하지 못할 것입니다.

### 2. 집중하기 (하나의 스킬 = 하나의 목적)

모든 것을 하는 "슈퍼 스킬"을 만들고 싶은 충동에 저항하세요. 구성 가능하고 집중된 스킬이 더 재사용 가능하고 유지 관리하기 쉽습니다.

좋음: project-memory는 메모리를 처리합니다. code-review는 리뷰를 처리합니다.
나쁨: project-everything은 메모리, 리뷰, 배포, 그리고 커피 주문까지 관리하려고 합니다.

### 3. 명확하고 자연스러운 트리거 문구 사용하기

설명에는 사용자가 자연스럽게 말할 수 있는 특정 문구가 포함되어야 합니다:

- "프로젝트 메모리 설정"
- "이 버그 로그"
- "핵심 사실 업데이트"
- "우리의 의사결정 추적"

트리거를 테스트하려면 "내가 실제로 이렇게 말할까?"라고 물어보세요. "문서 하위 시스템 초기화"와 같은 전문 용어가 많은 트리거는 피하세요.

### 4. 표준 문서처럼 보이게 만들기

`docs/`의 파일들은 유지 관리됩니다. `ai-stuff/`의 파일들은 무시됩니다.

팀이 이미 따르는 규칙을 사용하여 파일과 디렉토리의 이름을 지정하세요. 의사결정이 일반적으로 `adr/` 폴더에 들어간다면 그곳에 넣으세요. 팀이 Notion을 사용한다면 Notion 형식으로 내보내는 스킬을 만드는 것을 고려하세요.

### 5. 템플릿과 예제 포함하기

원하는 정확한 형식을 보여주세요. Claude는 추상적인 지침보다 예제를 더 안정적으로 따릅니다.

덜 효과적:

```
관련 정보가 있는 버그 항목을 만드세요
```

더 효과적:

```markdown
### YYYY-MM-DD - 버그 제목

**Issue**: 무슨 일이 일어났는지
**Root Cause**: 왜 일어났는지
**Solution**: 어떻게 고쳤는지
**Prevention**: 어떻게 피할 수 있는지
```

## 직접 시도해보기

다음 5분 안에 워크플로우에 프로젝트 메모리를 추가할 준비가 되셨나요? 시작하는 방법은 다음과 같습니다.

### 빠른 시작

```bash
# skilz CLI 설치
pip install skilz

# project-memory 에이전트 스킬 전역 설치 (모든 프로젝트에서 작동)
skilz install https://github.com/SpillwaveSolutions/project-memory
```

그런 다음 모든 프로젝트에서 Claude Code를 엽니다:

```
"이 프로젝트를 위한 프로젝트 메모리를 설정해줘"
```

Claude가 메모리 인프라를 만들고, CLAUDE.md를 구성하면 준비가 완료됩니다.

작동하지 않으면 다음 중 하나를 시도할 수 있습니다.

```
"프로젝트 메모리 스킬을 사용하여 이 프로젝트를 위한 프로젝트 메모리를 설정해줘"
```

또는...

```
"/project-memory-skill 이 프로젝트를 위한 프로젝트 메모리를 설정해줘"
```

스킬은 또한 처음부터 AGENTS.md와 함께 작동합니다. 저는 Claude Code와 OpenCode를 모두 자주 사용하기 때문에 둘 다 작동하도록 작성했습니다. 다시 말하지만, 이것은 제가 작성한 첫 번째 스킬 중 하나이며 개선의 여지가 있습니다.

### 첫 주 챌린지

한 번에 모든 것을 문서화하려고 하지 마세요. 대신:

- **1일차**: 프로젝트 메모리 설정
- **2일차**: key_facts.md에 하나의 항목 추가 (항상 잊어버리는 URL이나 포트)
- **3일차**: 버그를 고칠 때 bugs.md에 문서화
- **4일차**: decisions.md에 하나의 아키텍처 결정 기록
- **5일차**: 며칠 전에 문서화한 것에 대해 Claude에게 물어보기

5일차까지, 며칠 전에 말한 것을 Claude가 기억할 때 첫 번째 "아하!" 순간을 경험할 것입니다. 그때 가치가 실제가 됩니다.

### 성공의 모습

**일주일 후**, 다음을 가져야 합니다:

- 메모리 파일 전체에 걸쳐 3-5개의 항목
- Claude가 메모리를 사용하여 도움을 준 적어도 한 번의 사례
- 문서화 주변으로 형성되는 습관

**한 달 후**:

- 메모리 파일 전체에 걸쳐 15-25개의 항목
- 메모리 조회를 통해 절약된 시간의 여러 사례
- 팀원들이 "Claude는 어떻게 그걸 알지?"라고 묻기

## 향후 개선 사항: 프로젝트 메모리를 더욱 강력하게 만들기

이 스킬은 AI 기억 상실의 핵심 문제를 해결하지만, 개선의 여지가 많습니다. 다음은 프로젝트 메모리를 다음 수준으로 끌어올릴 수 있는 몇 가지 개선 사항입니다.

### 1. 자동 목차 생성

**필요성**: bugs.md, decisions.md, key_facts.md가 50-100개 이상의 항목을 넘어서면 특정 정보를 찾기가 더 어려워집니다. AI 코딩 어시스턴트는 에이전트 검색을 사용하지만, 구조화된 내비게이션은 인간과 AI 모두에게 도움이 됩니다.

**해결책**: 특정 임계값(예: 20개 항목)을 초과할 때 각 파일 상단에 자동으로 목차를 생성합니다. 이것은 의미론적 검색 기능을 향상시키고 수동 탐색을 훨씬 빠르게 만듭니다. 이것은 부분 발견 아키텍처에 도움이 될 것입니다.

### 2. 자동 파일 아카이빙

**필요성**: 2년 전의 버그 항목은 최근 항목만큼 관련성이 없습니다. 모든 것을 하나의 파일에 보관하면 잡음이 발생하고 검색 품질이 저하됩니다. 이것은 목차에 추가됩니다.

**해결책**: bugs.md와 issues.md를 위한 자동 아카이빙을 구현합니다:

- 6-12개월 이상 된 버그를 bugs-archive-2025.md로 이동
- 메인 파일에 참조 유지: 오래된 항목은 bugs-archive-2025.md 참조
- 의사결정과 핵심 사실은 일반적으로 아카이빙이 필요하지 않습니다. 변경하기로 결정할 때까지 무기한 관련성을 유지합니다.

### 3. 다중 에이전트 지원 (Claude를 넘어서)

**필요성**: 이 스킬이 원래 만들어졌을 때, Claude Code 스킬은 개방형 표준이 아니었습니다. 이제 Agent Skill Standard가 존재하므로 프로젝트 메모리를 진정으로 범용적으로 만들어야 합니다.

**해결책**: 스킬이 여러 AI 코딩 어시스턴트를 감지하고 구성하도록 만듭니다:

- Claude Code를 사용하는 경우, CLAUDE.md 업데이트
- Gemini를 사용하는 경우, GEMINI.md 업데이트
- Codex 또는 다른 도구를 사용하는 경우, AGENT.md 업데이트 또는 구성 파일 위치 프롬프트

스킬은 설정 중에 물어볼 수 있습니다: "어떤 AI 코딩 어시스턴트를 사용하시나요?" 그리고 모든 관련 파일을 자동으로 구성합니다.

### 진화는 계속됩니다

이러한 향후 개선 사항은 AI 코딩 어시스턴트와 에이전트 검색 기능에 대한 더 깊은 이해를 반영합니다. 생태계가 성숙하고 에이전트가 프로젝트 메모리를 어떻게 사용하는지에 대해 더 많이 배우면서 스킬은 계속 진화할 것입니다.

스킬 시스템의 아름다움은 이러한 개선 사항이 기존 기능을 깨지 않고 점진적으로 추가될 수 있다는 것입니다. 기본부터 시작한 다음 필요가 커짐에 따라 정교함을 쌓으세요.

## 결론: 작게 시작하고, 빠르게 복리화하기

project-memory 스킬은 의도적으로 단순합니다. SKILL.md와 몇 개의 템플릿 파일이 있는 폴더로, 총 300줄 미만일 것입니다. 그러나 이 최소한의 투자는 다음을 통해 수십 시간을 절약해주었습니다:

- 반복되는 디버깅 세션 제거 (같은 버그가 두 번째로 45분이 걸리지 않음)
- 아키텍처 일관성 유지 (더 이상 우발적인 프레임워크 확산 없음)
- 세션 간 프로젝트 지식 보존 (컨텍스트가 채팅 재시작에서 살아남음)
- 여러 AI 코딩 도구에서 작동 (팀 전체 이익)

더 중요한 것은, 작고 집중된 스킬이 어떻게 엄청난 영향을 미칠 수 있는지 보여줍니다. 복잡한 자동화가 필요하지 않습니다. 때로는 구조화된 마크다운 파일과 명확한 프로토콜이면 충분합니다.

오늘 project-memory를 설치해보세요. 이번 주에 하나의 버그, 하나의 의사결정, 그리고 하나의 핵심 사실을 문서화하세요. 기억된 지식의 복리를 경험하세요.

그런 다음, 세 번째로 Claude에게 같은 것을 설명하는 자신을 발견하면, 자신만의 스킬을 구축하세요. 작게 시작하세요. 진짜 문제를 해결하세요. 시간 절약이 복리로 쌓이는 것을 지켜보세요.

다섯 번째로 그 CORS 문제를 다시 해결할 필요가 없는 미래의 당신이 감사할 것입니다.

## 리소스

- **GitHub의 Project Memory Agent Skill**: 전체 소스 코드 및 예제
- **SkillzWave 마켓플레이스의 Project Memory Agent Skill**: 설치 지침이 포함된 스킬
- **Agent Skill Standard**: 크로스 플랫폼 스킬 사양
- **SkillzWave 마켓플레이스**: 커뮤니티 스킬 탐색 및 설치
- **skilz CLI GitHub 저장소**: 설치 및 사용 가이드, 소스 코드
- **skilz Agent Skill Installer**: CLI 문서: 매뉴얼, 설치 및 사용 가이드

성가신 문제를 해결한 스킬을 구축하셨나요? 댓글에 공유해주세요. 커뮤니티가 만드는 것을 보고 싶습니다.

## 저자 소개

Rick Hightower는 Fortune 100 금융 서비스 조직에서 광범위한 경험을 가진 기술 임원이자 데이터 엔지니어로, 고객 경험 메트릭을 최적화하기 위한 고급 머신 러닝 및 AI 솔루션 개발을 이끌었습니다. 그의 전문 지식은 이론적 AI 프레임워크와 실용적 엔터프라이즈 구현을 모두 포괄합니다.

Rick은 Gemini, Claude Code, Codex, OpenCode, Github Copilot CLI, Cursor, Aidr, Qwen Code, Kimi Code 및 약 14개의 다른 코딩 에이전트와 함께 작동하는 skilz 범용 에이전트 스킬 설치 프로그램을 작성했으며 세계 최대 에이전트 스킬 마켓플레이스의 공동 창립자입니다.

엔터프라이즈 AI 구현 및 전략에 대한 통찰력을 얻으려면 LinkedIn 또는 Medium에서 Rick Hightower와 연결하세요.

## 커뮤니티 확장 및 리소스

Claude Code 커뮤니티는 기능을 향상시키는 강력한 확장을 개발했습니다. 다음은 Spillwave Solutions의 몇 가지 유용한 리소스입니다:

### 통합 스킬

- **Notion Uploader/Downloader Agent Skill**: 문서 워크플로우를 위해 Markdown 콘텐츠 및 이미지를 Notion에 원활하게 업로드 및 다운로드
- **Confluence Agent Skill**: 엔터프라이즈 문서를 위해 Markdown 콘텐츠 및 이미지를 Confluence에 업로드 및 다운로드
- **JIRA Integration Agent Skill**: 특수 필수 필드 처리를 포함한 JIRA 티켓 생성 및 읽기

### 고급 개발 에이전트

- **Architect Agent Skill**: Claude Code를 Architect Mode로 전환하여 여러 프로젝트를 관리하고 전문 코드 에이전트로 실행되는 다른 Claude Code 인스턴스에 위임
- **Project Memory Agent Skill**: 소프트웨어 개발 전체에서 중요한 컨텍스트를 유지하기 위해 핵심 결정, 재발하는 버그, 티켓 및 중요한 사실 저장

### 시각화 및 디자인 도구

- **Design Doc Mermaid Agent Skill**: 아키텍처 문서를 위한 전문 Mermaid 다이어그램 생성을 위한 전문 스킬
- **PlantUML Agent Skill**: 소스 코드에서 PlantUML 다이어그램 생성, Markdown에서 다이어그램 추출, 이미지 링크 문서 생성
- **Image Generation Agent Skill**: 문서 및 디자인 작업을 위한 이미지 생성에 Gemini Banana 사용
- **SDD Agent Skill**: GitHub의 Spec-Kit 및 Spec-Driven Development 방법론을 통한 사용자 안내를 위한 포괄적인 Claude Code 스킬
- **PR Reviewer Agent Skill**: Claude Code를 위한 포괄적인 GitHub PR 코드 리뷰 스킬. gh CLI를 통한 데이터 수집 자동화, 업계 표준 기준(보안, 테스트, 유지 관리 가능성)에 대한 분석, 구조화된 리뷰 파일 생성, 승인 워크플로우로 피드백 게시. 인라인 주석, 티켓 추적 및 전문 리뷰 템플릿 포함.

### AI 모델 통합

- **Gemini Agent Skill**: 다중 모델 협업을 위해 Google의 Gemini AI에 특정 작업 위임
- **Image_gen Agent Skill**: 이미지를 생성하기 위해 Gemini Banana를 사용하는 이미지 생성 스킬

최근 저는 안전성, 유용성, 링크 및 PDA를 위한 Agents 스킬을 평가하기 위해 Agent Skill Viewer라는 데스크톱 앱을 작성했습니다.

Spillwave Solutions에서 더 많은 것을 탐색하세요 — 맞춤형 소프트웨어 개발 및 AI 기반 자동화 전문가

---

## 전체 스킬 마크다운 파일

~~~markdown
---
name: project-memory
description: Set up and maintain a structured project memory system in docs/project_notes/ that tracks bugs with solutions, architectural decisions, key project facts, and work history. Use this skill when asked to "set up project memory", "track our decisions", "log a bug fix", "update project memory", or "initialize memory system". Configures both CLAUDE.md and AGENTS.md to maintain memory awareness across different AI coding tools.
---

# Project Memory
 
## Table of Contents

- [Overview](#overview)
- [When to Use This Skill](#when-to-use-this-skill)
- [Core Capabilities](#core-capabilities)
  - [1. Initial Setup - Create Memory Infrastructure](#1-initial-setup---create-memory-infrastructure)
  - [2. Configure CLAUDE.md - Memory-Aware Behavior](#2-configure-claudemd---memory-aware-behavior)
  - [3. Configure AGENTS.md - Multi-Tool Support](#3-configure-agentsmd---multi-tool-support)
  - [4. Searching Memory Files](#4-searching-memory-files)
  - [5. Updating Memory Files](#5-updating-memory-files)
  - [6. Memory File Maintenance](#6-memory-file-maintenance)
- [Templates and References](#templates-and-references)
- [Example Workflows](#example-workflows)
- [Integration with Other Skills](#integration-with-other-skills)
- [Success Criteria](#success-criteria)

## Overview

Maintain institutional knowledge for projects by establishing a structured memory system in `docs/project_notes/`. This skill sets up four key memory files (bugs, decisions, key facts, issues) and configures CLAUDE.md and AGENTS.md to automatically reference and maintain them. The result is a project that remembers past decisions, solutions to problems, and important configuration details across coding sessions and across different AI tools.

## When to Use This Skill

Invoke this skill when:

- Starting a new project that will accumulate knowledge over time
- The project already has recurring bugs or decisions that should be documented
- The user asks to "set up project memory" or "track our decisions"
- The user wants to log a bug fix, architectural decision, or completed work
- Encountering a problem that feels familiar ("didn't we solve this before?")
- Before proposing an architectural change (check existing decisions first)
- Working on projects with multiple developers or AI tools (Claude Code, Cursor, etc.)

## Core Capabilities

### 1. Initial Setup - Create Memory Infrastructure

When invoked for the first time in a project, create the following structure:

```
docs/
└── project_notes/
    ├── bugs.md         # Bug log with solutions
    ├── decisions.md    # Architectural Decision Records
    ├── key_facts.md    # Project configuration and constants
    └── issues.md       # Work log with ticket references
```

**Directory naming rationale:** Using `docs/project_notes/` instead of `memory/` makes it look like standard engineering organization, not AI-specific tooling. This increases adoption and maintenance by human developers.

**Initial file content:** Copy templates from the `references/` directory in this skill:
- Use `references/bugs_template.md` for initial `bugs.md`
- Use `references/decisions_template.md` for initial `decisions.md`
- Use `references/key_facts_template.md` for initial `key_facts.md`
- Use `references/issues_template.md` for initial `issues.md`

Each template includes format examples and usage tips.

### 2. Configure CLAUDE.md - Memory-Aware Behavior

Add or update the following section in the project's `CLAUDE.md` file:

```markdown
## Project Memory System

This project maintains institutional knowledge in `docs/project_notes/` for consistency across sessions.

### Memory Files

- **bugs.md** - Bug log with dates, solutions, and prevention notes
- **decisions.md** - Architectural Decision Records (ADRs) with context and trade-offs
- **key_facts.md** - Project configuration, credentials, ports, important URLs
- **issues.md** - Work log with ticket IDs, descriptions, and URLs

### Memory-Aware Protocols

**Before proposing architectural changes:**
- Check `docs/project_notes/decisions.md` for existing decisions
- Verify the proposed approach doesn't conflict with past choices
- If it does conflict, acknowledge the existing decision and explain why a change is warranted

**When encountering errors or bugs:**
- Search `docs/project_notes/bugs.md` for similar issues
- Apply known solutions if found
- Document new bugs and solutions when resolved

**When looking up project configuration:**
- Check `docs/project_notes/key_facts.md` for credentials, ports, URLs, service accounts
- Prefer documented facts over assumptions

**When completing work on tickets:**
- Log completed work in `docs/project_notes/issues.md`
- Include ticket ID, date, brief description, and URL

**When user requests memory updates:**
- Update the appropriate memory file (bugs, decisions, key_facts, or issues)
- Follow the established format and style (bullet lists, dates, concise entries)

### Style Guidelines for Memory Files

- **Prefer bullet lists over tables** for simplicity and ease of editing
- **Keep entries concise** (1-3 lines for descriptions)
- **Always include dates** for temporal context
- **Include URLs** for tickets, documentation, monitoring dashboards
- **Manual cleanup** of old entries is expected (not automated)
```

### 3. Configure AGENTS.md - Multi-Tool Support

If the project has an `AGENTS.md` file (used for agent workflows or multi-tool projects), add the same memory protocols. This ensures consistency whether using Claude Code, Cursor, GitHub Copilot, or other AI tools.

**If AGENTS.md exists:** Add the same "Project Memory System" section as above.

**If AGENTS.md doesn't exist:** Ask the user if they want to create it. Many projects use multiple AI tools and benefit from shared memory protocols.

### 4. Searching Memory Files

When encountering problems or making decisions, proactively search memory files:

**Search bugs.md:**
```bash
# Look for similar errors
grep -i "connection refused" docs/project_notes/bugs.md

# Find bugs by date range
grep "2025-01" docs/project_notes/bugs.md
```

**Search decisions.md:**
```bash
# Check for decisions about a technology
grep -i "database" docs/project_notes/decisions.md

# Find all ADRs
grep "^### ADR-" docs/project_notes/decisions.md
```

**Search key_facts.md:**
```bash
# Find database connection info
grep -A 5 "Database" docs/project_notes/key_facts.md

# Look up service accounts
grep -i "service account" docs/project_notes/key_facts.md
```

**Use Grep tool for more complex searches:**
- Search across all memory files: `Grep(pattern="oauth", path="docs/project_notes/")`
- Context-aware search: `Grep(pattern="bug", path="docs/project_notes/bugs.md", -A=3, -B=3)`

### 5. Updating Memory Files

When the user requests updates or when documenting resolved issues, update the appropriate memory file:

**Adding a bug entry:**
```markdown
### YYYY-MM-DD - Brief Bug Description
- **Issue**: What went wrong
- **Root Cause**: Why it happened
- **Solution**: How it was fixed
- **Prevention**: How to avoid it in the future
```

**Adding a decision:**
```markdown
### ADR-XXX: Decision Title (YYYY-MM-DD)

**Context:**
- Why the decision was needed
- What problem it solves

**Decision:**
- What was chosen

**Alternatives Considered:**
- Option 1 -> Why rejected
- Option 2 -> Why rejected

**Consequences:**
- Benefits
- Trade-offs
```

**Adding key facts:**
- Organize by category (GCP Project, Database, API, Local Development, etc.)
- Use bullet lists for clarity
- Include both production and development details
- Add URLs for easy navigation
- See `references/key_facts_template.md` for security guidelines on what NOT to store

**Adding work log entry:**
```markdown
### YYYY-MM-DD - TICKET-ID: Brief Description
- **Status**: Completed / In Progress / Blocked
- **Description**: 1-2 line summary
- **URL**: https://jira.company.com/browse/TICKET-ID
- **Notes**: Any important context
```

### 6. Memory File Maintenance

**Periodically clean old entries:**
- User is responsible for manual cleanup (no automation)
- Remove very old bug entries (6+ months) that are no longer relevant
- Archive completed work from issues.md (3+ months old)
- Keep all decisions (they're lightweight and provide historical context)
- Update key_facts.md when project configuration changes

**Conflict resolution:**
- If proposing something that conflicts with decisions.md, explain why revisiting the decision is warranted
- Update the decision entry if the choice changes
- Add date of revision to show evolution

## Templates and References

This skill includes template files in `references/` that demonstrate proper formatting:

- **references/bugs_template.md** - Bug entry format with examples
- **references/decisions_template.md** - ADR format with examples
- **references/key_facts_template.md** - Key facts organization with examples (includes security guidelines)
- **references/issues_template.md** - Work log format with examples

When creating initial memory files, copy these templates to `docs/project_notes/` and customize them for the project.

## Example Workflows

### Scenario 1: Encountering a Familiar Bug

```
User: "I'm getting a 'connection refused' error from the database"
-> Search docs/project_notes/bugs.md for "connection"
-> Find previous solution: "Use AlloyDB Auth Proxy on port 5432"
-> Apply known fix
```

### Scenario 2: Proposing an Architectural Change

```
Internal: "User might benefit from using SQLAlchemy for migrations"
-> Check docs/project_notes/decisions.md
-> Find ADR-002: Already decided to use Alembic
-> Use Alembic instead, maintaining consistency
```

### Scenario 3: User Requests Memory Update

```
User: "Add that CORS fix to our bug log"
-> Read docs/project_notes/bugs.md
-> Add new entry with date, issue, solution, prevention
-> Confirm addition to user
```

### Scenario 4: Looking Up Project Configuration

```
Internal: "Need to connect to database"
-> Check docs/project_notes/key_facts.md
-> Find Database Configuration section
-> Use documented connection string and credentials
```

## Tips for Effective Memory Management

1. **Be proactive**: Check memory files before proposing solutions
2. **Be concise**: Keep entries brief (1-3 lines for descriptions)
3. **Be dated**: Always include dates for temporal context
4. **Be linked**: Include URLs to tickets, docs, monitoring dashboards
5. **Be selective**: Focus on recurring or instructive issues, not every bug

## Integration with Other Skills

The project-memory skill complements other skills:

- **requirements-documenter**: Requirements -> Decisions (ADRs reference requirements)
- **root-cause-debugger**: Bug diagnosis -> Bug log (document solutions after fixes)
- **code-quality-reviewer**: Quality issues -> Decisions (document quality standards)
- **docs-sync-editor**: Code changes -> Key facts (update when config changes)

When using these skills together, consider updating memory files as a follow-up action.

## Success Criteria

This skill is successfully deployed when:

- `docs/project_notes/` directory exists with all four memory files
- CLAUDE.md includes "Project Memory System" section with protocols
- AGENTS.md includes the same protocols (if file exists or user requested)
- Memory files follow template format and style guidelines
- AI assistant checks memory files before proposing changes
- User can easily request memory updates ("add this to bugs.md")
- Memory files look like standard engineering documentation, not AI artifacts
~~~

---

**문서 작성일: 2026-01-21**
