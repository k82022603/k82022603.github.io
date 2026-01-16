---
title: "Google Antigravity Agent Skills vs. Claude Code Skills"
date: 2026-01-16 04:00:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,  Material,  Antigravity,  claude-code,  agent-skills,  Claude.write]
---

## 동일한 표준, 다른 구현 - 완벽 비교 가이드

> **핵심 발견**: 두 시스템 모두 **동일한 오픈 스탠다드**를 사용합니다!  
> **표준**: Agent Skills Specification (agentskills.io)  
> **제안자**: Anthropic (2025년 10월)  
> **채택자**: Claude Code, Google Antigravity, OpenAI Codex, Cursor, 기타

---

## 📊 한눈에 비교

| 항목 | Claude Code Skills | Google Antigravity Skills |
|------|-------------------|--------------------------|
| **발표 시기** | 2025년 10월 16일 | 2026년 1월 14일 |
| **제안자** | Anthropic (원조) | Google (채택) |
| **파일 형식** | SKILL.md (동일) | SKILL.md (동일) |
| **호환성** | ✅ 완전 호환 | ✅ 완전 호환 |
| **설치 위치** | `~/.claude/skills/` | `~/.gemini/antigravity/skills/` |
| **프로젝트 스킬** | `.claude/skills/` | `.agent/skills/` |
| **자동 발견** | ✅ Yes | ✅ Yes |
| **플랫폼** | claude.ai, Claude Code, API | Antigravity IDE |
| **지원 모델** | Claude Sonnet/Opus | Gemini 3, Claude, GPT-OSS |
| **공식 스킬** | 60+ (GitHub) | 제한적 (초기) |
| **마켓플레이스** | SkillsMP.com | 개발 중 |

---

## 🎯 동일한 핵심 (The Same Core)

### 1. 완전히 동일한 SKILL.md 형식

```markdown
---
name: skill-name
description: What this skill does and when to use it
---

# Skill Title

Instructions for the AI agent...
```

**중요**: 한 플랫폼에서 작성한 스킬을 **다른 플랫폼에서 그대로 사용 가능**합니다!

```bash
# Claude Code 스킬을 Antigravity에서 사용
cp -r ~/.claude/skills/my-skill ~/.gemini/antigravity/skills/

# Antigravity 스킬을 Claude Code에서 사용
cp -r .agent/skills/my-skill ~/.claude/skills/
```

### 2. 동일한 구조

```
skill-name/
├── SKILL.md          # [필수] 메타데이터 + 지침
├── scripts/          # [선택] 실행 스크립트
├── references/       # [선택] 참고 문서
├── examples/         # [선택] Few-shot 예제
└── assets/           # [선택] 템플릿, 리소스
```

### 3. 동일한 Progressive Disclosure 원리

```
1. 시작 시: 메타데이터만 로드 (name + description)
2. 매칭 시: 전체 SKILL.md 로드
3. 필요 시: references/, scripts/ 접근
```

### 4. 동일한 보안 원칙

- ✅ 스크립트는 Context에 로드되지 않음 (출력만)
- ✅ References는 필요할 때만 읽음
- ✅ 신뢰할 수 있는 소스만 사용 권장

---

## 🔀 차이점 (The Differences)

### 1. 플랫폼 & 생태계

#### Claude Code Skills

**플랫폼**:
- 🌐 **claude.ai**: 웹 인터페이스
- 💻 **Claude Code**: VS Code 포크 IDE
- 🔌 **Claude API**: 프로그래밍 방식
- 🛠️ **Claude Agent SDK**: 커스텀 에이전트

**생태계**:
- 📦 **60+ 공식 스킬** (anthropics/skills GitHub)
- 🏪 **SkillsMP.com**: 커뮤니티 마켓플레이스
- 🔧 **skill-creator**: 스킬 생성 도우미 스킬
- 📚 **Skills Cookbook**: 상세 가이드

**장점**:
- ✅ 성숙한 생태계
- ✅ 풍부한 예제 및 문서
- ✅ 활발한 커뮤니티
- ✅ 다중 플랫폼 지원

#### Google Antigravity Skills

**플랫폼**:
- 🖥️ **Antigravity IDE**: 단독 에이전트 중심 IDE

**생태계**:
- 📦 **제한적 공식 스킬** (초기 단계)
- 🏗️ **마켓플레이스 개발 중**
- 📖 **Google Codelabs**: 튜토리얼

**장점**:
- ✅ 에이전트 중심 IDE 통합
- ✅ 다중 모델 지원 (Gemini, Claude, GPT)
- ✅ Manager View (병렬 에이전트)
- ✅ Browser Control 내장

---

### 2. 스킬 설치 위치

#### Claude Code

```bash
# Personal (Global) Skills
~/.claude/skills/
  ├── my-skill-1/
  └── my-skill-2/

# Project Skills
<project-root>/.claude/skills/
  ├── project-skill-1/
  └── project-skill-2/

# Plugin Skills
~/.claude/plugins/my-plugin/skills/
```

#### Google Antigravity

```bash
# Global Skills
~/.gemini/antigravity/skills/
  ├── my-skill-1/
  └── my-skill-2/

# Workspace (Project) Skills
<project-root>/.agent/skills/
  ├── project-skill-1/
  └── project-skill-2/
```

**우선순위**:
- **Claude Code**: Managed > Personal > Project > Plugin
- **Antigravity**: Workspace > Global

---

### 3. 설정 파일 차이

#### Claude Code: CLAUDE.md

```markdown
# CLAUDE.md (프로젝트 루트)

## Project Overview
React TypeScript application...

## Code Style
- Use functional components
- Prettier formatting
- ESLint rules

## Testing
- Jest + React Testing Library
- Minimum 80% coverage

## Workflows
- Feature branches from main
- Conventional commits
```

**목적**: 프로젝트 전반적인 컨텍스트 제공

#### Antigravity: Rules + Workflows + Skills

```markdown
# Rules (항상 적용)
~/.gemini/antigravity/rules.md

# Workflows (수동 트리거: /)
~/.gemini/antigravity/workflows/

# Skills (자동 로드)
~/.gemini/antigravity/skills/
```

**목적**: 3단계 계층 구조 (Rules < Workflows < Skills)

**비교**:

| 기능 | Claude Code | Antigravity |
|------|-------------|-------------|
| 프로젝트 컨텍스트 | CLAUDE.md | Rules |
| 저장된 프롬프트 | Slash Commands | Workflows |
| 자동 실행 능력 | Skills | Skills |

---

### 4. Subagents vs Manager View

#### Claude Code: Subagents

```markdown
# .claude/agents/code-reviewer.md
---
name: code-reviewer
description: Review code for quality
skills: pr-review, security-check
---
```

**특징**:
- Fork된 별도 컨텍스트
- 명시적 스킬 로드
- 독립 실행

#### Antigravity: Manager View

```
Manager View (중앙 제어)
  ├── Agent 1: 백엔드 개발 (병렬)
  ├── Agent 2: 프론트엔드 개발 (병렬)
  ├── Agent 3: 테스트 작성 (병렬)
  └── Browser Agent: UI 테스트 (자동)
```

**특징**:
- 중앙 오케스트레이션
- 자동 병렬 실행
- UI에서 진행 상황 모니터링

---

### 5. 브라우저 제어

#### Claude Code

- ❌ 기본적으로 브라우저 제어 없음
- ⚠️ MCP Server로 확장 가능

#### Antigravity

- ✅ **Browser Agent** 내장
- ✅ 자동 UI 테스트
- ✅ 스크린샷 캡처
- ✅ 플로우 녹화

**예시**:
```
Antigravity: "로그인 플로우 테스트하고 스크린샷 찍어줘"
→ Browser Agent 자동 실행
→ 실제 브라우저 조작
→ 결과 검증 + 스크린샷
```

---

### 6. 문서 생성 능력

#### Claude Code

**Pre-built Skills**:
- 📄 PDF (읽기 + 폼 작성)
- 📊 Excel (수식 포함)
- 📝 Word (문서 생성/편집)
- 🎨 PowerPoint (프레젠테이션)

**구현**: 각 문서 타입별 전문 스킬

#### Antigravity

**문서 생성**:
- 📄 PDF (기본 지원)
- 📊 Spreadsheets (기본 지원)
- 📝 Documents (기본 지원)

**구현**: IDE 레벨 통합

---

## 🔄 상호 호환성 (Cross-Platform Compatibility)

### 스킬 공유 가능성

**✅ 100% 호환**:
```bash
# 1. Claude에서 만든 스킬
~/.claude/skills/conventional-commits/SKILL.md

# 2. Antigravity로 복사
cp -r ~/.claude/skills/conventional-commits \
      ~/.gemini/antigravity/skills/

# 3. 즉시 사용 가능!
```

**Simon Willison의 증언** (2025년 10월):
> "You can grab a skills folder right now, point Codex CLI or 
> Gemini CLI at it and say 'read pdf/SKILL.md' and it will work, 
> despite those tools having no baked in knowledge of the skills system."

### 표준 준수

**agentskills.io 표준**:
```
✅ Anthropic Claude Code
✅ Google Antigravity
✅ OpenAI Codex CLI
✅ Cursor IDE
✅ 기타 에이전트 도구들
```

---

## 📈 생태계 성숙도

### Claude Code Skills (성숙)

**공식 스킬 (anthropics/skills)**:
```
document/
├── pdf/           # PDF 조작
├── docx/          # Word 문서
├── xlsx/          # Excel 스프레드시트
└── pptx/          # PowerPoint

development/
├── mcp-server/    # MCP 서버 생성
├── skill-creator/ # 스킬 제작 도구
└── websim/        # 웹 시뮬레이션

communication/
├── slack-gif/     # Slack GIF 생성
└── email-draft/   # 이메일 초안

creative/
├── art/           # 예술 작업
├── music/         # 음악 생성
└── design/        # 디자인 작업
```

**커뮤니티**:
- 🏪 **SkillsMP.com**: 수백 개 커뮤니티 스킬
- 🔍 AI 시맨틱 검색
- ⭐ 인기도 정렬
- 🏷️ 카테고리별 분류

### Google Antigravity Skills (초기)

**현황** (2026년 1월):
- 📦 제한적 공식 스킬
- 🏗️ 생태계 구축 중
- 📖 Google Codelabs 튜토리얼
- 🚀 빠른 성장 예상

---

## 💻 실전 사용 시나리오 비교

### 시나리오 1: 코드 리뷰

#### Claude Code
```bash
# 1. 스킬 설치
git clone https://github.com/anthropics/skills
cp -r skills/skills/code-review ~/.claude/skills/

# 2. 사용
[Claude Code에서]
"PR #123 리뷰해줘"

→ code-review 스킬 자동 로드
→ 보안, 스타일, 퍼포먼스 체크
→ 리뷰 리포트 생성
```

#### Antigravity
```bash
# 1. 스킬 설치
mkdir -p .agent/skills/code-review
# SKILL.md 작성

# 2. 사용 (동일)
[Antigravity에서]
"PR #123 리뷰해줘"

→ code-review 스킬 자동 로드
→ 보안, 스타일, 퍼포먼스 체크
→ 리뷰 리포트 생성
```

**결과**: 동일한 스킬, 동일한 결과

---

### 시나리오 2: 복잡한 기능 개발

#### Claude Code
```
[Synchronous Approach]

1. "User authentication 구현해줘"
   → Claude 구현
   
2. "이제 OAuth 추가해줘"
   → Claude 추가 구현
   
3. "테스트 작성해줘"
   → Claude 테스트 작성

[순차적 실행]
```

#### Antigravity
```
[Asynchronous Approach - Manager View]

"User authentication with OAuth 구현하고 테스트 작성해줘"

→ Manager View 활성화
  ├── Agent 1: 인증 로직 구현 (병렬)
  ├── Agent 2: OAuth 통합 (병렬)
  └── Agent 3: 테스트 작성 (병렬)

[병렬 실행, 더 빠름]
```

**차이점**: 
- Claude Code: 순차적, 대화형
- Antigravity: 병렬, 오케스트레이션

---

### 시나리오 3: UI 검증

#### Claude Code
```
[Manual Approach]

1. "React 컴포넌트 만들어줘"
   → 컴포넌트 생성
   
2. [개발자가 직접 브라우저에서 테스트]

3. "버그 있어, 수정해줘"
   → 수정
```

#### Antigravity
```
[Automated Approach]

"React 컴포넌트 만들고 브라우저에서 테스트해줘"

→ 컴포넌트 생성
→ Browser Agent 자동 실행
  - 브라우저 실행
  - 컴포넌트 렌더링
  - 인터랙션 테스트
  - 스크린샷 캡처
→ 결과 리포트 + 스크린샷
```

**차이점**: Antigravity는 브라우저 자동화 내장

---

## 🎨 스킬 작성 예제 (공통)

### 동일한 스킬, 양쪽 모두에서 작동

~~~markdown
---
name: api-endpoint-generator
description: |
  Generates production-ready REST API endpoints with validation,
  error handling, and tests. Use when creating new API routes.
tags: [api, backend, nodejs, express]
---

# API Endpoint Generator

## Instructions

When the user requests a new API endpoint:

1. **Ask for requirements**:
   - HTTP method (GET/POST/PUT/DELETE)
   - Route path
   - Request/Response schema
   - Authentication needed?

2. **Generate files**:
   - `routes/<endpoint>.js` - Express route
   - `controllers/<endpoint>.controller.js` - Business logic
   - `validators/<endpoint>.validator.js` - Joi validation
   - `tests/<endpoint>.test.js` - Jest tests

3. **Include**:
   - Input validation
   - Error handling (try-catch)
   - HTTP status codes
   - API documentation (JSDoc)

## Example

For: "Create POST /api/users endpoint"

Generate:
```javascript
// routes/users.js
const express = require('express');
const { createUser } = require('../controllers/users.controller');
const { validateCreateUser } = require('../validators/users.validator');

const router = express.Router();

router.post('/users', validateCreateUser, createUser);

module.exports = router;
```
~~~

**사용처**:
```bash
# Claude Code에서 사용
~/.claude/skills/api-endpoint-generator/

# Antigravity에서 사용 (동일한 파일)
~/.gemini/antigravity/skills/api-endpoint-generator/

# 둘 다 작동!
```

---

## 🏆 언제 어느 것을 선택할까?

### Claude Code를 선택하는 경우

**✅ 추천 상황**:
1. **웹 기반 작업** (claude.ai 사용)
2. **API 통합** (프로그래밍 방식 제어)
3. **문서 작업 중심** (PDF, Excel, Word, PPT)
4. **성숙한 생태계** 필요 (60+ 스킬)
5. **다중 플랫폼** (claude.ai + Claude Code + API)
6. **커뮤니티 스킬** 활용

**강점**:
- 📚 풍부한 문서 및 예제
- 🏪 활발한 커뮤니티 마켓플레이스
- 🔌 다양한 플랫폼 옵션
- 📄 강력한 문서 생성 능력

**약점**:
- ⚠️ 브라우저 자동화 제한적
- ⚠️ 순차적 실행 (병렬 부족)
- ⚠️ 단일 모델 (Claude만)

---

### Google Antigravity를 선택하는 경우

**✅ 추천 상황**:
1. **복잡한 프로젝트** (다단계 작업)
2. **병렬 실행** 필요 (시간 절약)
3. **UI 테스트 자동화**
4. **다중 모델 활용** (Gemini + Claude + GPT)
5. **에이전트 중심 워크플로우**
6. **대규모 리팩토링**

**강점**:
- 🚀 병렬 에이전트 오케스트레이션
- 🌐 Browser Agent (자동 UI 테스트)
- 🤖 다중 모델 지원
- 📊 Manager View (시각적 모니터링)
- 🎯 에이전트 중심 설계

**약점**:
- ⚠️ 초기 생태계 (스킬 적음)
- ⚠️ 단일 플랫폼 (IDE만)
- ⚠️ 커뮤니티 성숙도 낮음

---

## 🔮 미래 전망

### 수렴 가능성

**공통 표준 사용** → **생태계 통합 가능성**:

```
현재:
Claude Skills ─────┐
                   ├─ Agent Skills Standard
Antigravity Skills ┘

미래 예상:
Universal Skills Marketplace
├── Claude Code
├── Antigravity
├── Cursor
├── VS Code Copilot
└── 기타 도구들

→ "Write once, use everywhere"
```

### 예상 발전 방향

#### Claude Code
- 🔮 **Subagent 개선**: 더 강력한 병렬 실행
- 🔮 **MCP 통합**: 더 많은 외부 도구 연결
- 🔮 **Enterprise 기능**: 조직 전체 스킬 배포

#### Antigravity
- 🔮 **생태계 확장**: 더 많은 공식 스킬
- 🔮 **마켓플레이스**: 커뮤니티 스킬 공유
- 🔮 **API 제공**: 프로그래밍 방식 접근

---

## 💡 하이브리드 전략 (Best of Both Worlds)

### 전략: 두 도구 모두 활용

```
프로젝트 루트:
├── .claude/
│   └── skills/         # Claude Code 전용
│       └── document-gen/
├── .agent/
│   └── skills/         # Antigravity 전용
│       └── deployment/
└── shared-skills/      # 공통 스킬
    ├── code-review/
    └── testing/
```

**심볼릭 링크로 공유**:
```bash
# 공통 스킬을 양쪽에서 사용
ln -s ~/shared-skills/code-review ~/.claude/skills/
ln -s ~/shared-skills/code-review ~/.gemini/antigravity/skills/
```

### 역할 분담

| 작업 유형 | 도구 선택 | 이유 |
|----------|---------|------|
| 문서 생성 | Claude Code | 전문 문서 스킬 (PDF, Excel, PPT) |
| 복잡한 개발 | Antigravity | 병렬 에이전트 오케스트레이션 |
| 코드 리뷰 | 둘 다 | 동일한 스킬 사용 가능 |
| UI 테스트 | Antigravity | Browser Agent 내장 |
| API 작업 | Claude Code | API 통합 우수 |
| 대규모 리팩토링 | Antigravity | 병렬 처리 빠름 |

---

## 📚 학습 리소스

### Claude Code Skills

**공식 문서**:
- https://platform.claude.com/docs/agents-and-tools/agent-skills
- https://code.claude.com/docs/skills
- https://github.com/anthropics/skills

**커뮤니티**:
- https://skillsmp.com
- Agent Skills Cookbook
- Anthropic Academy

**예제**:
- anthropics/skills GitHub (60+ 스킬)
- Community Skills Marketplace

### Google Antigravity Skills

**공식 문서**:
- https://antigravity.google/docs/skills
- https://codelabs.developers.google.com/getting-started-google-antigravity

**커뮤니티**:
- Google AI Developers Forum
- Medium: Agent Skills 튜토리얼

**예제**:
- Romin Irani's GitHub tutorials

---

## 🎯 핵심 요약

### 공통점 (80%)

```
✅ 동일한 SKILL.md 형식
✅ 동일한 Progressive Disclosure
✅ 동일한 파일 구조
✅ 동일한 보안 원칙
✅ 100% 상호 호환 가능
```

### 차이점 (20%)

```
❌ 플랫폼 (다중 vs 단일)
❌ 생태계 성숙도 (성숙 vs 초기)
❌ 실행 방식 (순차 vs 병렬)
❌ 브라우저 제어 (제한적 vs 내장)
❌ 설정 구조 (CLAUDE.md vs Rules/Workflows)
```

### 결론

**"같은 언어를 말하는 두 도구"**

- ✅ 스킬은 **양쪽 모두에서 작동**
- ✅ 한 번 작성, **어디서나 사용**
- ✅ 각자의 **강점 활용**
- ✅ 필요에 따라 **선택 또는 병행**

---

## 🚀 시작하기

### 1. 간단한 스킬로 시작

```bash
# 1. 디렉토리 생성
mkdir -p ~/universal-skills/hello-world

# 2. SKILL.md 작성
cat > ~/universal-skills/hello-world/SKILL.md << 'EOF'
---
name: hello-world
description: |
  A simple greeting skill that demonstrates cross-platform compatibility.
  Use when the user asks for a greeting.
---

# Hello World Skill

When the user requests a greeting:
1. Generate a friendly, personalized greeting
2. Include the current date
3. Add a motivational message

Example: "Hello! Today is January 16, 2026. Make it a great day!"
EOF

# 3. 양쪽 플랫폼에 링크
ln -s ~/universal-skills/hello-world ~/.claude/skills/
ln -s ~/universal-skills/hello-world ~/.gemini/antigravity/skills/

# 4. 테스트
# Claude Code: "인사해줘"
# Antigravity: "인사해줘"
# → 동일한 스킬, 동일한 결과!
```

### 2. 커뮤니티 스킬 탐색

```bash
# Claude Skills 마켓플레이스
https://skillsmp.com

# Anthropic 공식 스킬
git clone https://github.com/anthropics/skills
```

### 3. 나만의 스킬 라이브러리 구축

```bash
# 프로젝트별 스킬 관리
~/my-skills/
├── universal/        # 양쪽 모두 사용
│   ├── code-review/
│   └── testing/
├── claude-specific/  # Claude만
│   └── document-gen/
└── antigravity-specific/  # Antigravity만
    └── ui-automation/
```

---

## ✨ 마지막 조언

**"도구가 아니라 스킬에 투자하세요"**

스킬은 **플랫폼 중립적**입니다. Claude Code에서 작성한 스킬은:
- ✅ Antigravity에서 작동
- ✅ 미래의 다른 도구에서도 작동 가능
- ✅ 오픈 스탠다드 → 영구적 가치

**시작하는 가장 좋은 방법**:
1. 간단한 스킬 하나 만들기
2. 양쪽 플랫폼에서 테스트
3. 점진적으로 개선
4. 팀과 공유
5. 커뮤니티 기여

**The Future is Skill-Based** 🚀

---

**작성일**: 2026-01-16  
