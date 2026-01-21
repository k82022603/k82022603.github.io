---
title: "Antigravity Agent Skills 완벽 가이드"
date: 2026-01-21 06:00:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,   Antigravity,   Developer,  Guide,  Claude.write]
---


## 목차
1. [개요](#개요)
2. [Agent Skills란 무엇인가](#agent-skills란-무엇인가)
3. [왜 Agent Skills를 사용해야 하는가](#왜-agent-skills를-사용해야-하는가)
4. [Skills vs MCP 서버](#skills-vs-mcp-서버)
5. [오픈소스 스킬팩 활용하기](#오픈소스-스킬팩-활용하기)
6. [프로젝트에 스킬 통합하기](#프로젝트에-스킬-통합하기)
7. [필수 메타 스킬 설정](#필수-메타-스킬-설정)
8. [커스텀 스킬 만들기](#커스텀-스킬-만들기)
9. [스킬 패턴 5가지](#스킬-패턴-5가지)
10. [실전 활용 예시](#실전-활용-예시)
11. [고급 활용 전략](#고급-활용-전략)
12. [문제 해결 및 팁](#문제-해결-및-팁)

---

## 개요

Google Antigravity는 2025년 12월 출시된 AI 에이전트 중심 개발 플랫폼입니다. 기존의 코드 어시스턴트들이 단순히 자동완성이나 제안을 제공했다면, Antigravity는 완전히 자율적인 AI 에이전트가 계획을 세우고, 코드를 작성하며, 테스트하고, 검증까지 수행하는 패러다임을 제시합니다.

2026년 1월, Antigravity는 **Agent Skills**라는 획기적인 기능을 정식 도입했습니다. 이것은 단순한 기능 추가가 아니라, AI 개발 워크플로우의 근본적인 전환을 의미합니다. Skills는 Anthropic이 처음 제안한 오픈 스탠다드를 Google이 공식 채택한 것으로, Claude Code, Cursor, OpenCode, Gemini CLI 등 주요 AI 개발 도구들이 모두 지원하는 범용 형식입니다.

이 문서는 Agent Skills의 개념부터 실전 활용법까지, 개발 생산성을 극대화하기 위한 모든 것을 다룹니다.

---

## Agent Skills란 무엇인가

### 핵심 개념

Agent Skills는 AI 에이전트가 특정 작업을 수행할 때 참조할 수 있는 **재사용 가능한 지식 패키지**입니다. 마치 인간 전문가가 특정 분야의 매뉴얼이나 체크리스트를 참조하듯이, AI 에이전트도 Skills를 통해 전문화된 지식과 절차를 습득합니다.

### 기술적 정의

공식 문서에 따르면, Skill은 다음과 같이 정의됩니다:

> "Skills are an open standard for extending agent capabilities. A skill is a folder containing a SKILL.md file with instructions that the agent can follow when working on specific tasks."

즉, 스킬은 단순히 폴더 하나와 `SKILL.md` 파일로 구성된 매우 간단한 구조입니다. 하지만 이 단순함 속에 엄청난 힘이 숨어 있습니다.

### 기본 구조

```
my-skill/
├── SKILL.md              # 필수: 스킬 정의 및 지침
├── scripts/              # 선택: 실행 가능한 스크립트
├── references/           # 선택: 추가 참조 문서
├── examples/             # 선택: 사용 예시
└── assets/              # 선택: 템플릿, 데이터 등
```

### SKILL.md 구조

모든 스킬의 핵심은 `SKILL.md` 파일입니다. 이 파일은 YAML frontmatter와 마크다운 본문으로 구성됩니다:

```markdown
---
name: my-awesome-skill
description: 이 스킬이 무엇을 하는지, 언제 사용되는지 명확하게 설명합니다. 트리거 키워드를 포함해야 합니다.
---

# My Awesome Skill

## Purpose
이 스킬의 목적과 해결하는 문제

## When to Use
어떤 상황에서 이 스킬을 사용해야 하는지

## Key Information
실제 가이드라인, 패턴, 예시 등
```

### Progressive Disclosure 원칙

Skills의 가장 중요한 특징은 **점진적 노출(Progressive Disclosure)** 원칙입니다. 

기존 방식에서는 모든 규칙과 컨텍스트를 항상 프롬프트에 포함해야 했습니다. 이는 몇 가지 문제를 야기했습니다:

1. **토큰 낭비**: 관련 없는 정보까지 모두 컨텍스트 윈도우에 로드
2. **비용 증가**: 불필요한 토큰 사용으로 API 비용 상승
3. **성능 저하**: 너무 많은 정보로 인한 에이전트 혼란
4. **컨텍스트 제한**: 윈도우 크기 초과로 중요한 정보 누락

Skills는 이 문제를 근본적으로 해결합니다:

```
기존 방식:
[모든 규칙 + 모든 가이드 + 모든 템플릿] → 에이전트

Skills 방식:
1. 에이전트가 작업 분석
2. 필요한 스킬만 메타데이터에서 검색
3. 해당 스킬의 전체 내용만 로드
4. 작업 완료 후 언로드
```

이렇게 하면 에이전트는 시작 시 스킬의 간단한 메타데이터(이름과 설명)만 읽고, 실제로 필요할 때만 전체 내용을 로드합니다. 스크립트 코드는 아예 컨텍스트 윈도우에 들어가지 않고 직접 실행만 됩니다.

---

## 왜 Agent Skills를 사용해야 하는가

### 1. 반복 작업 제거

매번 프롬프트를 새로 작성하는 것은 비효율적입니다.

**Before (Skills 없이):**
```
사용자: "이 코드 리뷰해줘. 버그 찾고, 성능 문제 체크하고, 
       스타일 가이드 준수 확인하고, 보안 취약점도 봐줘."

[다음 프로젝트에서]
사용자: "또 이 코드 리뷰해줘. 버그 찾고, 성능 문제 체크하고..."
```

**After (Skills 사용):**
```
사용자: "코드 리뷰해줘"
에이전트: [code-review 스킬 자동 로드]
         → 버그 검사
         → 성능 분석
         → 스타일 가이드 검증
         → 보안 스캔
         → 개선 제안
         [완료]
```

### 2. 팀 표준 자동 적용

코드 리뷰, 커밋 메시지 형식, 아키텍처 패턴 등 팀 표준을 스킬로 정의하면, 모든 팀원의 AI 에이전트가 동일한 기준을 자동으로 따릅니다.

```
프로젝트/.agent/skills/
├── team-code-review/
├── conventional-commits/
├── react-architecture/
└── api-design-standards/
```

이렇게 설정하면:
- 신입 개발자도 시니어 수준의 코드 작성
- 코드 리뷰 시간 70% 감소
- 일관된 코드베이스 유지
- 온보딩 시간 대폭 단축

### 3. 컨텍스트 윈도우 최적화

일반적인 프로젝트에서는 수십 개의 규칙과 가이드라인이 있습니다. 모든 것을 매번 로드하면:

```
토큰 사용량 (예시):
- 코딩 스타일 가이드: 2,000 토큰
- API 디자인 패턴: 1,500 토큰
- 테스트 전략: 1,800 토큰
- 배포 절차: 2,200 토큰
- 보안 체크리스트: 1,900 토큰
총: 9,400 토큰 (매 요청마다!)

실제 필요한 것: 특정 작업에 맞는 1-2개 스킬만 (2,000-3,000 토큰)
절약: 약 70% 토큰 절약
```

### 4. 전문가 수준의 출력 품질

스킬을 통해 에이전트는 일반주의자(Generalist)에서 전문가(Specialist)로 변신합니다.

**논문 증거**: arXiv:2601.04748v1에 따르면, 적절한 스킬을 갖춘 단일 에이전트가 여러 일반 에이전트보다 70% 더 효율적이며, 작업 완료율도 40% 향상되었습니다.

### 5. 검증 가능한 워크플로우

스킬은 단순한 지침이 아니라 실행 가능한 스크립트를 포함할 수 있습니다:

```python
# database-migration 스킬의 scripts/verify.py
def verify_migration():
    # 1. 백업 확인
    # 2. 마이그레이션 파일 검증
    # 3. 롤백 계획 확인
    # 4. 테스트 DB에서 시뮬레이션
    # 5. 성공 시에만 진행 승인
```

에이전트는 이 스크립트를 자동으로 실행하여 작업을 검증합니다. 코드가 컨텍스트에 들어가지 않으므로 토큰도 절약됩니다.

### 6. 크로스 플랫폼 호환성

한 번 만든 스킬은 여러 플랫폼에서 재사용 가능합니다:

- Google Antigravity
- Claude Code (Anthropic)
- Cursor
- OpenCode
- Gemini CLI
- GitHub Copilot (곧 지원 예정)

동일한 `.agent/skills/` 폴더를 여러 프로젝트에서 공유하거나, 글로벌 스킬로 설정할 수 있습니다.

---

## Skills vs MCP 서버

많은 개발자들이 Skills와 MCP(Model Context Protocol) 서버를 혼동합니다. 두 기술은 서로 다른 목적을 가지고 있으며, 상호 보완적입니다.

### MCP 서버란?

MCP는 **지속적인 외부 시스템 연결**을 위한 프로토콜입니다.

**특징:**
- 클라이언트-서버 아키텍처
- 지속적인 프로세스로 실행
- 인증 상태 유지
- 동적 도구 및 리소스 노출
- PostgreSQL, GitHub, Slack 등과 같은 외부 시스템 연결

**예시:**
```
MCP 서버: PostgreSQL 연결
- 데이터베이스 연결 유지
- 쿼리 실행
- 트랜잭션 관리
- 실시간 데이터 접근
```

### Skills란?

Skills는 **경량의 임시 작업 정의**입니다.

**특징:**
- 서버리스, 파일 기반
- 필요할 때만 로드
- 상태 비저장 (Stateless)
- 버전 관리 가능 (Git과 함께)
- 로컬 또는 원격 실행

**예시:**
```
Skill: 체인지로그 생성
- Git 히스토리 분석
- 컨벤셔널 커밋 파싱
- 마크다운 포맷팅
- 완료 후 언로드
```

### 비교표

| 측면 | Skills | MCP 서버 |
|------|--------|----------|
| **목적** | 작업 절차 정의 | 외부 시스템 연결 |
| **아키텍처** | 파일 기반 | 클라이언트-서버 |
| **수명** | 임시적 (작업 중에만) | 지속적 (세션 내내) |
| **상태** | Stateless | Stateful |
| **인프라** | 필요 없음 | 서버 프로세스 필요 |
| **복잡도** | 매우 낮음 | 중간~높음 |
| **버전 관리** | Git으로 쉽게 관리 | 서버 배포 관리 필요 |
| **사용 사례** | 코드 리뷰, 테스트, 포맷팅 | DB 접근, API 호출, 외부 통합 |
| **토큰 효율** | 매우 높음 (Progressive Disclosure) | 보통 (필요한 도구만 노출) |

### 언제 무엇을 사용할까?

**Skills 사용:**
- 코드 리뷰 자동화
- 테스트 생성
- 커밋 메시지 포맷팅
- 코드 스타일 검사
- 프로젝트 템플릿 적용
- 배포 전 체크리스트

**MCP 서버 사용:**
- 데이터베이스 쿼리
- GitHub PR 생성/관리
- Slack 메시지 전송
- Jira 이슈 업데이트
- 외부 API 지속적 호출

**함께 사용:**
```
시나리오: 프로덕션 배포

1. Skills: deployment-checklist
   - 테스트 통과 확인
   - 버전 번호 검증
   - 체인지로그 생성

2. MCP 서버: GitHub
   - PR 상태 확인
   - 승인 여부 검증

3. MCP 서버: Slack
   - 배포 알림 전송

4. Skills: rollback-plan
   - 롤백 절차 문서화
   - 롤백 스크립트 준비
```

---

## 오픈소스 스킬팩 활용하기

처음부터 모든 스킬을 직접 만들 필요는 없습니다. 커뮤니티가 이미 수백 개의 검증된 스킬을 만들어 놓았습니다.

### Antigravity Awesome Skills

가장 포괄적인 스킬 컬렉션은 **sickn33/antigravity-awesome-skills** 레포지토리입니다.

**통계 (2026년 1월 기준):**
- 224개의 검증된 스킬
- GitHub 스타 800+
- 포크 150+
- Anthropic, Vercel, OpenAI, Google 공식 스킬 포함

**주요 카테고리:**

1. **문서 작성 (Document Skills)**
   - docx: Word 문서 생성/편집
   - pdf: PDF 생성/양식 작성
   - pptx: 프레젠테이션 생성
   - xlsx: 스프레드시트 작업

2. **개발 워크플로우**
   - git-commit-formatter: Conventional Commits 자동 적용
   - code-review: 포괄적 코드 리뷰
   - tdd: 테스트 주도 개발
   - brainstorming: 기획 단계 브레인스토밍

3. **프레임워크 특화**
   - react-best-practices: React 컴포넌트 패턴
   - nextjs-clean-architecture: Next.js 프로젝트 구조
   - vue-composition-api: Vue 3 Composition API
   - angular-standalone: Angular Standalone 컴포넌트

4. **백엔드 개발**
   - backend-express-typescript: Express/TypeScript 마이크로서비스
   - aws-serverless: AWS Lambda 패턴
   - azure-functions: Azure Functions 오케스트레이션
   - graphql-schema-design: GraphQL 스키마 설계

5. **데이터베이스**
   - prisma-patterns: Prisma ORM 베스트 프랙티스
   - mongodb-schema: MongoDB 스키마 디자인
   - postgres-optimization: PostgreSQL 성능 최적화

6. **보안**
   - api-security-testing: API 보안 테스트
   - aws-pentesting: AWS 보안 평가
   - active-directory-attacks: AD 보안 테스트

7. **마케팅 & 비즈니스**
   - copywriting: 마케팅 카피 작성
   - seo-audit: SEO 감사
   - email-sequence: 이메일 시퀀스 설계
   - pricing-strategy: 가격 전략

8. **메타 스킬 (Meta Skills)**
   - using-superpowers: 스킬 검색 및 활용 가이드
   - verification-before-completion: 완료 전 자동 검증
   - skill-developer: 새로운 스킬 개발 가이드

### 설치 방법

#### 전역 설치 (모든 프로젝트에서 사용)

```bash
# Antigravity 전역 스킬 디렉토리
git clone https://github.com/sickn33/antigravity-awesome-skills.git \
  ~/.gemini/antigravity/skills

# Claude Code 전역 스킬 디렉토리  
git clone https://github.com/sickn33/antigravity-awesome-skills.git \
  ~/.claude/skills

# Gemini CLI 전역 스킬 디렉토리
git clone https://github.com/sickn33/antigravity-awesome-skills.git \
  ~/.gemini/skills

# Cursor 전역 스킬 디렉토리
git clone https://github.com/sickn33/antigravity-awesome-skills.git \
  ~/.cursor/skills
```

#### 프로젝트별 설치 (특정 프로젝트만)

```bash
# 프로젝트 루트에서 실행
git clone https://github.com/sickn33/antigravity-awesome-skills.git \
  .agent/skills
```

### 스킬 업데이트

```bash
# 스킬 디렉토리로 이동
cd ~/.gemini/antigravity/skills  # 또는 프로젝트의 .agent/skills

# 최신 스킬 가져오기
git pull origin main
```

### 필수 스킬 추천

모든 224개 스킬이 항상 필요한 것은 아닙니다. 프로젝트 성격에 따라 선택적으로 활성화할 수 있습니다.

**웹 개발 프로젝트:**
```
.agent/skills/
├── brainstorming/
├── frontend-design/
├── react-best-practices/
├── verification-before-completion/
└── using-superpowers/
```

**백엔드 API 프로젝트:**
```
.agent/skills/
├── backend-express-typescript/
├── api-security-testing/
├── prisma-patterns/
├── verification-before-completion/
└── using-superpowers/
```

**풀스택 SaaS:**
```
.agent/skills/
├── brainstorming/
├── frontend-design/
├── backend-express-typescript/
├── stripe-integration/
├── copywriting/
├── verification-before-completion/
└── using-superpowers/
```

---

## 프로젝트에 스킬 통합하기

### 디렉토리 구조 생성

```bash
# 프로젝트 루트에서
mkdir -p .agent/skills
```

### 선택적 스킬 복사

전체 레포지토리를 클론하는 대신, 필요한 스킬만 선택적으로 복사할 수도 있습니다:

```bash
# 1. 임시 디렉토리에 전체 레포 클론
git clone https://github.com/sickn33/antigravity-awesome-skills.git /tmp/skills

# 2. 필요한 스킬만 복사
cp -r /tmp/skills/skills/brainstorming .agent/skills/
cp -r /tmp/skills/skills/frontend-design .agent/skills/
cp -r /tmp/skills/skills/verification-before-completion .agent/skills/
cp -r /tmp/skills/skills/using-superpowers .agent/skills/

# 3. 임시 디렉토리 삭제
rm -rf /tmp/skills
```

### 스킬 확인

```bash
# 설치된 스킬 목록 확인
ls -la .agent/skills/

# 특정 스킬의 내용 확인
cat .agent/skills/brainstorming/SKILL.md
```

### Git 설정

`.gitignore`에 추가:

```gitignore
# 전역 스킬은 추적하지 않음
.agent/skills/

# 프로젝트 특화 커스텀 스킬만 추적
!.agent/skills/korean-saas-landing/
!.agent/skills/company-code-review/
```

또는 팀 전체가 동일한 스킬을 사용하게 하려면:

```gitignore
# 모든 스킬을 Git으로 관리
# .gitignore에 .agent/ 관련 항목 없음
```

---

## 필수 메타 스킬 설정

### using-superpowers 스킬

이 스킬은 에이전트가 **다른 스킬을 효과적으로 찾고 사용**하도록 안내합니다.

**핵심 기능:**
1. 작업 시작 전 스킬 목록 스캔
2. 가장 적합한 스킬 선택
3. 해당 스킬의 지침 따름
4. 필요시 여러 스킬 조합

**왜 중요한가:**

에이전트가 스킬의 존재를 알아도, 언제 어떤 스킬을 사용할지 모를 수 있습니다. `using-superpowers`는 메타인지(meta-cognition) 역할을 합니다.

**Before:**
```
사용자: "React 컴포넌트 만들어줘"
에이전트: [일반적인 React 코드 생성]
```

**After (using-superpowers 적용):**
```
사용자: "React 컴포넌트 만들어줘"
에이전트: 
  1. 스킬 목록 스캔
  2. "react-best-practices" 발견
  3. 해당 스킬 로드
  4. 스킬에 정의된 패턴 따름
  5. Tailwind, TypeScript, 접근성 고려한 프로덕션급 컴포넌트 생성
```

**설정 방법:**

`.agent/rules.md` 또는 프로젝트 루트에 `CLAUDE.md` (Claude Code의 경우) 생성:

```markdown
# 프로젝트 규칙

## 스킬 사용

작업을 시작하기 전에 ALWAYS:
1. @using-superpowers 스킬을 먼저 참조
2. 사용 가능한 스킬 목록 확인
3. 현재 작업에 적합한 스킬 선택
4. 선택한 스킬의 지침을 따름
```

### verification-before-completion 스킬

이 스킬은 에이전트가 "완료했습니다"라고 말하기 전에 **실제로 검증**하도록 강제합니다.

**핵심 원칙:**

일반 에이전트의 문제:
```
에이전트: "코드를 작성했습니다. 완료!"
개발자: (테스트 실행) → 에러 발생
개발자: "버그 있어요..."
에이전트: "죄송합니다. 수정하겠습니다."
```

`verification-before-completion` 적용:
```
에이전트: 
  1. 코드 작성
  2. 자동으로 테스트 실행
  3. 빌드 검증
  4. 린트 체크
  5. 모든 검증 통과 확인
  6. "완료했습니다" (검증 결과 포함)
```

**설정 방법:**

```markdown
# .agent/rules.md

## 완료 전 검증

작업 완료를 선언하기 전에 ALWAYS:
1. @verification-before-completion 스킬 참조
2. 적절한 검증 명령 실행:
   - 테스트: `npm test` 또는 `pytest`
   - 빌드: `npm run build`
   - 린트: `npm run lint`
   - 타입 체크: `tsc --noEmit`
3. 모든 검증이 통과한 경우에만 완료 선언
4. 검증 결과를 사용자에게 보고
```

**실제 스킬 내용 예시:**

~~~markdown
---
name: verification-before-completion
description: 완료 전 자동 검증 실행. 테스트, 빌드, 린트 등을 통과해야만 완료 선언.
---

# Verification Before Completion

## Purpose
에이전트의 "완료" 선언에 객관적 증거를 요구합니다.

## Verification Steps

### 1. 테스트 실행
```bash
npm test         # JavaScript/TypeScript
pytest           # Python
go test ./...    # Go
cargo test       # Rust
```

### 2. 빌드 검증
```bash
npm run build    # 빌드 성공 확인
tsc --noEmit     # 타입 에러 없음 확인
```

### 3. 코드 품질
```bash
npm run lint     # ESLint
black --check .  # Python 포맷팅
```

### 4. 커밋 메시지 (해당 시)
- Conventional Commits 형식 준수
- 의미 있는 변경사항 설명

## Output Format
```
✅ Verification Results:
- Tests: 42 passed, 0 failed
- Build: Success (0 errors, 0 warnings)
- Lint: No issues
- Type Check: Passed

Task completed successfully with verification.
```
~~~

---

## 커스텀 스킬 만들기

오픈소스 스킬만으로 충분하지 않을 때, 자신만의 스킬을 만들 수 있습니다.

### 사례 1: korean-saas-landing (한국 SaaS 랜딩 페이지)

한국 B2B SaaS의 랜딩 페이지는 미국과 다른 구조와 카피 패턴을 가집니다.

~~~markdown
---
name: korean-saas-landing
description: 한국 B2B SaaS 랜딩 페이지 구조 및 카피라이팅 패턴. "랜딩", "landing", "B2B", "SaaS", "한국 시장" 등의 키워드에 반응.
---

# Korean B2B SaaS Landing Page

## Purpose
한국 B2B 시장에 최적화된 랜딩 페이지 구조와 카피를 제공

## When to Use
- 한국 B2B SaaS 랜딩 페이지 제작
- 기존 랜딩 페이지의 한국 시장 현지화
- 전환율 최적화가 필요한 경우

## 구조 패턴

### 1. Hero Section
- **헤드라인 공식**: "[업무 고충] 해결하고 [정량적 결과] 달성하세요"
  - 예: "계약서 검토에 3일 걸리셨나요? 이제 30분이면 됩니다"
- **서브헤드**: 구체적 수치와 신뢰 요소
  - 예: "법무법인 1,200곳이 선택한 AI 계약 검토 서비스"
- **CTA**: 명확하고 위험 부담 없는 행동
  - 좋은 예: "무료로 시작하기", "데모 신청"
  - 나쁜 예: "지금 구매", "가입하기"

### 2. Social Proof (신뢰 구축)
- 한국 기업들은 레퍼런스를 매우 중요시함
- 우선순위:
  1. 로고 벽: 대기업/공공기관 먼저
  2. 구체적 숫자: "OO 기업 포함 1,200개 조직"
  3. 고객 인터뷰: 직급과 회사명 명시
  
### 3. 기능 설명
- **문제-해결 구조**:
  - AS-IS (현재 문제): "수작업으로 2시간 소요"
  - TO-BE (해결 후): "AI가 3분 만에 완료"
- 스크린샷은 한글 UI 필수
- 각 기능마다 구체적 ROI 제시

### 4. 가격 표시
- 한국 B2B는 투명한 가격을 선호하지 않는 경향
- 권장 접근:
  - "맞춤 견적" 또는 "도입 문의"
  - 단, Freemium 모델은 명확히 표시
  
### 5. FAQ
- 필수 항목:
  - 보안/개인정보 처리
  - 기존 시스템 연동 가능 여부
  - 도입 기간
  - 기술 지원
  
### 6. 마지막 CTA
- 2단계 CTA 제공:
  - Primary: "도입 문의하기"
  - Secondary: "자료 다운로드"

## 카피라이팅 원칙

1. **존댓말 사용**: "~하세요", "~입니다"
2. **구체적 숫자**: "빠릅니다" → "3배 빠릅니다"
3. **권위 활용**: "OO대학교 연구진 개발", "특허 출원"
4. **무료 체험 강조**: 한국 시장은 체험 후 결정 선호

## 기술 스택 권장

```typescript
// Next.js + Tailwind 기본 구조
export default function KoreanLanding() {
  return (
    <>
      <HeroSection />
      <SocialProof />
      <ProblemSolution />
      <Features />
      <Pricing />
      <FAQ />
      <FinalCTA />
    </>
  )
}
```

## 참고 자료

- 성공 사례: 잔디, 슬랙 한국 버전, 노션 한국 시장 진입 케이스
- 벤치마크: 국내 주요 B2B SaaS 랜딩 분석
~~~

**스킬 저장:**

```bash
mkdir -p .agent/skills/korean-saas-landing
# 위 내용을 .agent/skills/korean-saas-landing/SKILL.md에 저장
```

### 사례 2: nextjs-clean-architecture

프로젝트 아키텍처를 일관되게 유지하는 스킬:

~~~markdown
---
name: nextjs-clean-architecture
description: Next.js App Router 프로젝트의 Clean Architecture 패턴. 폴더 구조, 네이밍, 계층 분리 기준 정의.
---

# Next.js Clean Architecture

## 폴더 구조

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Route Groups
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   └── api/               # API Routes
│
├── components/            # React 컴포넌트
│   ├── ui/               # 재사용 가능한 UI 컴포넌트
│   │   ├── Button.tsx
│   │   └── Input.tsx
│   ├── features/         # 기능별 컴포넌트
│   │   ├── auth/
│   │   └── dashboard/
│   └── layouts/          # 레이아웃 컴포넌트
│
├── lib/                  # 비즈니스 로직 (도메인 계층)
│   ├── services/         # 서비스 레이어
│   │   ├── authService.ts
│   │   └── userService.ts
│   ├── repositories/     # 데이터 접근 레이어
│   │   └── userRepository.ts
│   ├── entities/         # 도메인 엔티티
│   │   └── User.ts
│   └── useCases/         # Use Case (비즈니스 규칙)
│       └── createUser.ts
│
├── hooks/                # Custom React Hooks
│   ├── useAuth.ts
│   └── useUser.ts
│
├── types/                # TypeScript 타입 정의
│   ├── api.ts
│   └── models.ts
│
└── utils/                # 유틸리티 함수
    ├── validation.ts
    └── formatting.ts
```

## 계층 분리 원칙

### 1. Presentation Layer (app/, components/)
- UI 렌더링만 담당
- 비즈니스 로직 포함 금지
- hooks를 통해 데이터 접근

```tsx
// ❌ Bad: 컴포넌트에 비즈니스 로직
export default function UserProfile() {
  const handleUpdate = async (data) => {
    const validated = validateUser(data)
    await fetch('/api/user', { method: 'PUT', body: validated })
  }
}

// ✅ Good: hook으로 분리
export default function UserProfile() {
  const { updateUser } = useUser()
  const handleUpdate = (data) => updateUser(data)
}
```

### 2. Domain Layer (lib/services, lib/useCases)
- 비즈니스 규칙 구현
- 외부 의존성 없음 (프레임워크, DB 독립적)

```typescript
// lib/useCases/createUser.ts
export async function createUser(
  data: CreateUserInput,
  repository: UserRepository
): Promise<User> {
  // 비즈니스 규칙 검증
  if (data.age < 18) {
    throw new Error('Must be 18 or older')
  }
  
  // 도메인 엔티티 생성
  const user = new User(data)
  
  // 영속화
  return repository.save(user)
}
```

### 3. Data Layer (lib/repositories)
- 데이터베이스, API 등 외부 시스템과 통신
- 도메인 계층이 정의한 인터페이스 구현

```typescript
// lib/repositories/userRepository.ts
export interface UserRepository {
  save(user: User): Promise<User>
  findById(id: string): Promise<User | null>
}

export class PrismaUserRepository implements UserRepository {
  async save(user: User): Promise<User> {
    return prisma.user.create({ data: user })
  }
  
  async findById(id: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { id } })
  }
}
```

## 네이밍 규칙

### 파일명
- 컴포넌트: PascalCase (UserProfile.tsx)
- 훅: camelCase (useAuth.ts)
- 유틸: camelCase (formatDate.ts)
- 상수: UPPER_SNAKE_CASE (API_CONFIG.ts)

### 함수명
- 컴포넌트: PascalCase
- 훅: use + PascalCase
- 서비스: 명사 + 동사 (userService.create)
- Use Case: 동사 + 명사 (createUser)

## 의존성 규칙

```
app/ → hooks → lib/services → lib/repositories
     ↓                      ↓
components/              lib/entities/
```

- 하위 계층은 상위 계층을 알지 못함
- 의존성은 항상 안쪽(도메인)을 향함

## API Route 패턴

```typescript
// app/api/users/route.ts
import { createUser } from '@/lib/useCases/createUser'
import { PrismaUserRepository } from '@/lib/repositories/userRepository'

export async function POST(request: Request) {
  const data = await request.json()
  const repository = new PrismaUserRepository()
  
  try {
    const user = await createUser(data, repository)
    return Response.json(user, { status: 201 })
  } catch (error) {
    return Response.json({ error: error.message }, { status: 400 })
  }
}
```

## 테스트 전략

```typescript
// __tests__/lib/useCases/createUser.test.ts
describe('createUser', () => {
  it('should create user when valid data', async () => {
    const mockRepository = {
      save: jest.fn().mockResolvedValue(mockUser)
    }
    
    const result = await createUser(validData, mockRepository)
    
    expect(result).toEqual(mockUser)
    expect(mockRepository.save).toHaveBeenCalled()
  })
})
```

## 환경 설정

```typescript
// lib/config/env.ts
const config = {
  database: {
    url: process.env.DATABASE_URL!,
  },
  auth: {
    secret: process.env.AUTH_SECRET!,
  },
} as const

export default config
```

---

## 사용 예시

프로젝트에서 이 스킬을 사용하는 방법:

**개발자:**
```
"새로운 블로그 기능을 추가해줘. Clean Architecture 패턴 따라서."
```

**에이전트:**
```

1. 도메인 엔티티 생성
   - lib/entities/Post.ts

2. Repository 인터페이스 및 구현
   - lib/repositories/postRepository.ts

3. Use Cases
   - lib/useCases/createPost.ts
   - lib/useCases/getPost.ts

4. Service Layer
   - lib/services/postService.ts

5. API Routes
   - app/api/posts/route.ts
   - app/api/posts/[id]/route.ts

6. Hooks
   - hooks/usePost.ts

7. Components
   - components/features/blog/PostList.tsx
   - components/features/blog/PostDetail.tsx

완료! 계층 분리 및 의존성 규칙 준수 확인됨.
```
~~~

---

## 스킬 패턴 5가지

스킬은 복잡도와 기능에 따라 5가지 패턴으로 분류됩니다.

### 1. Basic Router Pattern (기본 라우터)

**특징:**
- `SKILL.md`만 존재
- 순수한 프롬프트 엔지니어링
- 추가 파일 없음

**사용 시나리오:**
간단한 규칙이나 포맷팅 작업

**예시: Git Commit Formatter**

~~~markdown
---
name: git-commit-formatter
description: Git 커밋 메시지를 Conventional Commits 형식으로 자동 변환
---

# Git Commit Formatter

## Conventional Commits Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

## Types
- feat: 새로운 기능
- fix: 버그 수정
- docs: 문서 변경
- style: 코드 포맷팅 (세미콜론, 공백 등)
- refactor: 리팩토링
- test: 테스트 추가/수정
- chore: 빌드 설정 등 기타 변경

## Rules
1. type은 소문자만 사용
2. description은 명령형 현재 시제
3. 마침표(.) 금지
4. 50자 이내 권장

## Examples

❌ Bad:
```
Updated the login page.
Fixed a bug in the user service
```

✅ Good:
```
feat(auth): add social login with Google
fix(user): resolve duplicate email validation error
docs(readme): update installation instructions
```
~~~

**디렉토리 구조:**
```
git-commit-formatter/
└── SKILL.md
```

### 2. Reference Pattern (참조 패턴)

**특징:**
- `SKILL.md` + `references/` 폴더
- 추가 문서나 예제 참조
- 500라인 제한 우회

**사용 시나리오:**
복잡한 가이드라인이나 API 문서 참조

**예시: API Design Standards**

```
api-design-standards/
├── SKILL.md
└── references/
    ├── rest-conventions.md
    ├── error-handling.md
    └── authentication.md
```

**SKILL.md:**
```markdown
---
name: api-design-standards
description: RESTful API 설계 표준 및 베스트 프랙티스
---

# API Design Standards

## Overview
이 스킬은 일관된 API 설계를 위한 표준을 정의합니다.

## Quick Reference

### Endpoint Naming
- 복수형 명사 사용: `/users`, `/posts`
- 계층 구조: `/users/{id}/posts`
- 동사 금지: `/getUser` ❌ → `/users/{id}` ✅

### HTTP Methods
- GET: 조회 (멱등성)
- POST: 생성 (비멱등성)
- PUT: 전체 업데이트 (멱등성)
- PATCH: 부분 업데이트 (멱등성)
- DELETE: 삭제 (멱등성)

## Detailed Guidelines

자세한 내용은 다음 참조 문서를 확인:

- [REST Conventions](references/rest-conventions.md)
- [Error Handling](references/error-handling.md)
- [Authentication](references/authentication.md)
```

**references/error-handling.md:**
~~~markdown
# Error Handling Standards

## Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "timestamp": "2026-01-21T06:00:00Z",
    "requestId": "req_123456"
  }
}
```

## HTTP Status Codes

- 200 OK: 성공
- 201 Created: 리소스 생성 성공
- 204 No Content: 성공 (응답 바디 없음)
- 400 Bad Request: 클라이언트 요청 오류
- 401 Unauthorized: 인증 필요
- 403 Forbidden: 권한 없음
- 404 Not Found: 리소스 없음
- 409 Conflict: 충돌 (중복 등)
- 422 Unprocessable Entity: 검증 실패
- 500 Internal Server Error: 서버 오류
- 503 Service Unavailable: 일시적 장애

## Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| VALIDATION_ERROR | 입력 검증 실패 | 422 |
| AUTHENTICATION_REQUIRED | 인증 필요 | 401 |
| INSUFFICIENT_PERMISSIONS | 권한 부족 | 403 |
| RESOURCE_NOT_FOUND | 리소스 없음 | 404 |
| DUPLICATE_RESOURCE | 중복 리소스 | 409 |
| RATE_LIMIT_EXCEEDED | 요청 한도 초과 | 429 |
| INTERNAL_ERROR | 내부 서버 오류 | 500 |
~~~

### 3. Few-shot Pattern (예시 패턴)

**특징:**
- `SKILL.md` + `examples/` 폴더
- 구체적인 코드 예시 제공
- 학습 효과 극대화

**사용 시나리오:**
특정 코딩 패턴이나 스타일 학습

**예시: React Component Patterns**

```
react-patterns/
├── SKILL.md
└── examples/
    ├── button-component.tsx
    ├── form-with-validation.tsx
    └── data-fetching-hook.tsx
```

**examples/button-component.tsx:**
```typescript
// ✅ Correct Pattern: Compound Component with Variants

import { cva, type VariantProps } from 'class-variance-authority'
import { forwardRef } from 'react'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    return (
      <button
        className={buttonVariants({ variant, size, className })}
        ref={ref}
        {...props}
      />
    )
  }
)

Button.displayName = 'Button'

export { Button, buttonVariants }

// Usage:
// <Button variant="default" size="lg">Click me</Button>
// <Button variant="destructive" size="sm">Delete</Button>
```

### 4. Tool Use Pattern (도구 사용 패턴)

**특징:**
- `SKILL.md` + `scripts/` 폴더
- 실행 가능한 스크립트 포함
- 자동화 및 검증

**사용 시나리오:**
테스트, 린팅, 빌드, 배포 등 자동화

**예시: Database Migration Validator**

```
db-migration-validator/
├── SKILL.md
└── scripts/
    ├── validate.py
    ├── backup.sh
    └── rollback.sh
```

**SKILL.md:**
~~~markdown
---
name: db-migration-validator
description: 데이터베이스 마이그레이션 전 자동 검증 및 백업
---

# Database Migration Validator

## Purpose
프로덕션 DB 마이그레이션 시 안전성 보장

## Workflow

1. **백업 생성**
   ```bash
   ./scripts/backup.sh
   ```

2. **마이그레이션 검증**
   ```bash
   python scripts/validate.py
   ```

3. **마이그레이션 실행**
   (검증 통과 시에만)

4. **롤백 계획 준비**
   ```bash
   ./scripts/rollback.sh --dry-run
   ```

## Validation Checks

- SQL 구문 오류 없음
- 인덱스 누락 확인
- 외래키 제약조건 검증
- 데이터 무결성 체크
- 성능 임팩트 분석

## Usage

```bash
# 에이전트가 자동으로 실행:
python scripts/validate.py --migration-file=20260121_add_user_roles.sql
```
~~~

**scripts/validate.py:**
```python
#!/usr/bin/env python3
import argparse
import sys
import re

def validate_migration(file_path: str) -> bool:
    """마이그레이션 파일 검증"""
    errors = []
    
    with open(file_path, 'r') as f:
        content = f.read()
    
    # 1. DROP TABLE 금지
    if re.search(r'DROP\s+TABLE', content, re.IGNORECASE):
        errors.append("❌ DROP TABLE detected. Use ALTER instead.")
    
    # 2. 인덱스 체크
    if 'ADD COLUMN' in content.upper():
        if 'CREATE INDEX' not in content.upper():
            errors.append("⚠️  New columns should have indexes.")
    
    # 3. NOT NULL without DEFAULT
    if re.search(r'ADD\s+COLUMN.*NOT\s+NULL(?!.*DEFAULT)', content, re.IGNORECASE):
        errors.append("❌ NOT NULL column must have DEFAULT value.")
    
    # 4. Foreign Key 검증
    if 'FOREIGN KEY' in content.upper():
        if 'ON DELETE' not in content.upper():
            errors.append("⚠️  Foreign key should specify ON DELETE behavior.")
    
    if errors:
        print("\n🚨 Migration Validation Failed:\n")
        for error in errors:
            print(f"  {error}")
        return False
    
    print("\n✅ Migration Validation Passed!")
    return True

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--migration-file', required=True)
    args = parser.parse_args()
    
    success = validate_migration(args.migration_file)
    sys.exit(0 if success else 1)
```

### 5. All-in-One Pattern (통합 패턴)

**특징:**
- `SKILL.md` + `references/` + `examples/` + `scripts/`
- 모든 요소 포함
- 가장 강력하고 완전한 스킬

**사용 시나리오:**
복잡한 도메인이나 엔터프라이즈 표준

**예시: Full-Stack Feature Builder**

```
fullstack-feature/
├── SKILL.md
├── references/
│   ├── backend-api-design.md
│   ├── frontend-component-structure.md
│   └── database-schema-patterns.md
├── examples/
│   ├── user-management/
│   │   ├── backend/
│   │   │   ├── controller.ts
│   │   │   ├── service.ts
│   │   │   └── repository.ts
│   │   └── frontend/
│   │       ├── UserList.tsx
│   │       └── useUsers.ts
│   └── product-catalog/
└── scripts/
    ├── generate-api-client.sh
    ├── run-integration-tests.sh
    └── check-dependencies.py
```

---

## 실전 활용 예시

### 시나리오 1: 새로운 기능 개발

**개발자 요청:**
```
"사용자 프로필 편집 기능을 추가해줘. 
 - 프로필 사진 업로드
 - 이름, 이메일, 전화번호 수정
 - 비밀번호 변경
 React + Express + Prisma 스택으로."
```

**에이전트 워크플로우 (using-superpowers 적용):**

```
1. 스킬 스캔
   @using-superpowers 로드
   → 사용 가능한 스킬 목록 확인:
     - brainstorming
     - nextjs-clean-architecture
     - backend-express-typescript
     - prisma-patterns
     - react-best-practices
     - frontend-design
     - verification-before-completion

2. 브레인스토밍 단계
   @brainstorming 로드
   → 요구사항 명확화:
     Q: "프로필 사진 크기 제한은?"
     Q: "이메일 변경 시 인증 필요한가요?"
     Q: "비밀번호 변경 시 현재 비밀번호 확인?"
   
   사용자 답변 수집 후 계획 수립

3. 아키텍처 적용
   @nextjs-clean-architecture 로드
   
   폴더 구조 생성:
   src/
   ├── lib/
   │   ├── entities/UserProfile.ts
   │   ├── useCases/updateProfile.ts
   │   ├── repositories/userRepository.ts
   │   └── services/profileService.ts
   ├── app/api/profile/
   │   └── route.ts
   ├── components/features/profile/
   │   ├── ProfileForm.tsx
   │   └── AvatarUpload.tsx
   └── hooks/
       └── useProfile.ts

4. 백엔드 구현
   @backend-express-typescript 로드
   @prisma-patterns 로드
   
   - Prisma 스키마 업데이트
   - Repository 구현
   - Service 레이어 작성
   - API 엔드포인트 생성
   - 검증 로직 추가

5. 프론트엔드 구현
   @react-best-practices 로드
   @frontend-design 로드
   
   - 타입 안전한 Form 컴포넌트
   - 파일 업로드 처리
   - 에러 핸들링
   - 낙관적 업데이트

6. 검증
   @verification-before-completion 로드
   
   자동 실행:
   ✓ TypeScript 타입 체크
   ✓ ESLint 검사
   ✓ 단위 테스트 실행
   ✓ Prisma 마이그레이션 검증
   ✓ API 엔드포인트 테스트

7. 완료 보고
   "✅ 사용자 프로필 편집 기능 완료
   
   구현 내용:
   - 백엔드 API: PUT /api/profile
   - 프론트엔드: ProfileForm 컴포넌트
   - 데이터베이스: User 테이블 업데이트
   
   검증 결과:
   - 타입 체크: 통과
   - Lint: 0 issues
   - 테스트: 18/18 passed
   - 빌드: 성공
   
   다음 단계:
   - 통합 테스트 실행 권장
   - 스테이징 환경 배포 가능"
```

### 시나리오 2: 레거시 코드 리팩토링

**개발자 요청:**
```
"이 오래된 jQuery 코드를 React로 리팩토링해줘"
```

**에이전트 워크플로우:**

```
1. @using-superpowers
   → code-review 스킬 발견
   
2. @code-review
   기존 코드 분석:
   - 기능 파악
   - 상태 관리 패턴 식별
   - 의존성 확인
   - 잠재적 버그 발견

3. @react-best-practices
   리팩토링 계획 수립:
   - 함수형 컴포넌트 + Hooks
   - 상태는 useState/useReducer
   - 사이드 이펙트는 useEffect
   - 최적화: useMemo, useCallback

4. 단계별 구현
   - HTML → JSX 변환
   - jQuery 셀렉터 → ref
   - AJAX → fetch/axios
   - 이벤트 핸들러 → React 방식

5. @verification-before-completion
   - 기존 기능 100% 동작 확인
   - 성능 비교 (Lighthouse)
   - 접근성 검사
   - 번들 사이즈 체크

6. 완료
```

### 시나리오 3: 한국 시장 진출 랜딩 페이지

**개발자 요청:**
```
"미국 SaaS 제품의 한국 시장 랜딩 페이지 만들어줘"
```

**에이전트 워크플로우:**

```
1. @korean-saas-landing (커스텀 스킬)
   
   구조 생성:
   - Hero: 문제-해결 헤드라인
   - Social Proof: 한국 기업 로고 배치
   - 기능 설명: AS-IS/TO-BE 구조
   - 가격: "맞춤 견적" 접근
   - FAQ: 보안/연동 중심

2. @frontend-design
   
   디자인 시스템 적용:
   - 한글 웹폰트 최적화
   - 모바일 최우선 (한국 80% 모바일)
   - Tailwind 유틸리티

3. @copywriting (마케팅 스킬)
   
   카피 작성:
   - 존댓말 일관성
   - 구체적 수치 강조
   - 신뢰 요소 강화

4. @verification-before-completion
   
   체크리스트:
   ✓ 한글 폰트 로딩 최적화
   ✓ SEO 메타태그 (한글)
   ✓ Open Graph 이미지
   ✓ 모바일 반응형 확인
   ✓ Lighthouse 성능 90+
```

---

## 고급 활용 전략

### 1. 스킬 조합 (Skill Composition)

여러 스킬을 조합하여 더 강력한 워크플로우를 만들 수 있습니다.

**예: E-commerce 제품 페이지 구축**

```markdown
# .agent/rules.md

## E-commerce Product Page Workflow

작업 순서:

1. @brainstorming
   - 제품 카테고리 파악
   - 주요 기능 정의
   - 전환 포인트 식별

2. @nextjs-clean-architecture
   - 폴더 구조 설정
   - 계층 분리

3. @react-best-practices
   - 컴포넌트 구현

4. @frontend-design
   - UI/UX 디자인

5. @copywriting
   - 제품 설명 작성
   - CTA 문구 최적화

6. @seo-audit
   - 메타태그 최적화
   - 스키마 마크업

7. @verification-before-completion
   - 성능 검증
   - 접근성 확인
```

### 2. 조건부 스킬 활성화

`.agent/rules.md`에서 조건부 로직:

```markdown
## Conditional Skill Activation

### Production Code
프로덕션 코드 작업 시:
- ALWAYS use @verification-before-completion
- ALWAYS use @code-review
- 테스트 커버리지 80% 이상 필수

### Prototype/POC
프로토타입 작업 시:
- @brainstorming만 사용
- 빠른 구현 우선
- 검증 단계 생략 가능

### Documentation
문서 작업 시:
- @technical-writing
- 한글 문서는 @korean-technical-writing
```

### 3. 팀 표준 강제

```markdown
# .agent/rules.md (팀 공유)

## Non-Negotiable Standards

다음 스킬은 무조건 따를 것:

1. @conventional-commits
   - 모든 커밋 메시지
   - 예외 없음

2. @team-code-review
   - PR 생성 전 자동 실행
   - 모든 체크 통과 필수

3. @api-design-standards
   - 새로운 API 엔드포인트
   - 팀 승인 없이 변경 금지

4. @security-checklist
   - 인증/권한 관련 코드
   - 보안팀 검토 필수
```

### 4. 스킬 버전 관리

```
.agent/skills/
├── v1/
│   └── old-api-design/
└── v2/
    └── new-api-design/

# .agent/rules.md
사용 버전: v2
레거시 유지보수 시에만 v1 참조
```

### 5. 동적 스킬 로딩

```markdown
## Dynamic Skill Loading

파일 확장자별 자동 스킬:
- *.test.ts → @tdd
- *.stories.tsx → @storybook-patterns
- *.spec.ts → @integration-testing
- migration_*.sql → @db-migration-validator
- Dockerfile → @docker-best-practices
```

---

## 문제 해결 및 팁

### 문제 1: 스킬이 로드되지 않음

**증상:**
```
에이전트가 스킬을 무시하고 일반적인 응답만 함
```

**해결:**

1. **스킬 위치 확인**
   ```bash
   # 프로젝트 스킬
   ls -la .agent/skills/
   
   # 전역 스킬
   ls -la ~/.gemini/antigravity/skills/
   ```

2. **SKILL.md frontmatter 검증**
   ```yaml
   ---
   name: my-skill        # 필수
   description: "..."    # 필수, 트리거 키워드 포함
   ---
   ```

3. **Antigravity 재시작**
   ```
   1. Antigravity 종료
   2. 캐시 삭제: rm -rf ~/.gemini/cache
   3. Antigravity 재실행
   ```

4. **명시적 스킬 호출**
   ```
   사용자: "@my-skill을 사용해서 이 작업 해줘"
   ```

### 문제 2: 스킬이 너무 많아서 느림

**증상:**
```
에이전트 응답 시간이 길어짐
컨텍스트 윈도우 부족 에러
```

**해결:**

1. **선택적 스킬만 유지**
   ```bash
   # 불필요한 스킬 제거
   rm -rf .agent/skills/unused-skill
   ```

2. **글로벌 vs 프로젝트 스킬 분리**
   ```
   전역: 범용 스킬 (git-commit, code-review)
   프로젝트: 특화 스킬 (nextjs-architecture, korean-landing)
   ```

3. **Progressive Disclosure 활용**
   - 큰 스킬은 references로 분리
   - SKILL.md는 500줄 이하 유지

### 문제 3: 스킬 충돌

**증상:**
```
두 스킬이 서로 모순된 지침 제공
```

**해결:**

1. **우선순위 명시 (.agent/rules.md)**
   ```markdown
   ## Skill Priority
   
   충돌 시 우선순위:
   1. 프로젝트 특화 스킬
   2. 팀 표준 스킬
   3. 글로벌 스킬
   ```

2. **스킬 통합**
   ```
   기존: react-patterns + nextjs-patterns (충돌)
   → 통합: nextjs-react-patterns (단일 스킬)
   ```

### 문제 4: 스킬 업데이트 반영 안됨

**해결:**

```bash
# 스킬 레포 업데이트
cd .agent/skills
git pull origin main

# 또는 특정 스킬만
cd .agent/skills/my-skill
git pull
```

**팁: Git Submodule 사용**

```bash
# 초기 설정
git submodule add https://github.com/sickn33/antigravity-awesome-skills.git .agent/skills

# 업데이트
git submodule update --remote --merge
```

### 팁 1: 스킬 효과 측정

**Before/After 비교:**

```python
# scripts/measure-skill-impact.py
import time
import json

metrics = {
    "without_skills": {
        "time": 45,  # minutes
        "iterations": 7,
        "errors": 3
    },
    "with_skills": {
        "time": 15,  # minutes
        "iterations": 2,
        "errors": 0
    }
}

improvement = (1 - metrics["with_skills"]["time"] / metrics["without_skills"]["time"]) * 100
print(f"시간 절감: {improvement}%")
```

**실제 사례:**
- 랜딩 페이지 제작: 2시간 → 30분 (75% 감소)
- API 개발: 3시간 → 1시간 (66% 감소)
- 코드 리뷰: 30분 → 5분 (83% 감소)

### 팁 2: 팀 스킬 라이브러리 구축

```
company/.agent/skills/
├── core/                    # 핵심 표준
│   ├── code-review/
│   ├── git-workflow/
│   └── security-baseline/
├── platform/                # 플랫폼별
│   ├── web/
│   ├── mobile/
│   └── backend/
└── domain/                  # 도메인별
    ├── fintech/
    ├── healthcare/
    └── e-commerce/
```

### 팁 3: 스킬 문서화

~~~markdown
# .agent/skills/README.md

# 프로젝트 스킬 가이드

## 설치된 스킬

| 스킬 | 목적 | 트리거 키워드 |
|------|------|--------------|
| nextjs-architecture | 프로젝트 구조 | "구조", "폴더", "아키텍처" |
| korean-landing | 랜딩 페이지 | "랜딩", "landing", "한국" |
| verification | 검증 자동화 | 작업 완료 시 자동 |

## 사용 예시

### 새 기능 개발
```
```

### 마케팅 페이지
```
```
~~~

### 팁 4: 지속적 개선

**스킬 성능 추적:**

```javascript
// .agent/skills/metrics.json
{
  "skill_usage": {
    "brainstorming": { "count": 45, "avg_time": 2.3 },
    "verification-before-completion": { "count": 112, "success_rate": 0.94 }
  },
  "improvements": [
    {
      "date": "2026-01-15",
      "skill": "korean-landing",
      "change": "Added pricing strategy section",
      "impact": "15% faster landing page creation"
    }
  ]
}
```

### 팁 5: 에이전트 피드백 루프

~~~markdown
## Skill Improvement Workflow

1. 에이전트가 스킬 사용
2. 결과 평가
3. 개선점 발견 시:
   ```bash
   .agent/skills/my-skill/improvements.md에 기록
   ```
4. 주간 리뷰에서 스킬 업데이트
5. 팀 공유
~~~

---

## 결론

Google Antigravity의 Agent Skills는 단순한 기능이 아니라, AI 개발 워크플로우의 패러다임 전환입니다. 

**핵심 가치:**

1. **효율성**: 반복 작업 70-80% 감소
2. **일관성**: 팀 표준 자동 적용
3. **품질**: 전문가 수준의 출력
4. **확장성**: 오픈 스탠다드로 크로스 플랫폼
5. **유지보수성**: Git으로 버전 관리

**시작 체크리스트:**

- [ ] Antigravity Awesome Skills 레포 클론
- [ ] `using-superpowers` 스킬 전역 활성화
- [ ] `verification-before-completion` 프로젝트 적용
- [ ] 첫 커스텀 스킬 만들기 (팀 코딩 표준)
- [ ] `.agent/rules.md`에 스킬 워크플로우 정의
- [ ] 팀과 스킬 라이브러리 공유

**다음 단계:**

- 더 많은 스킬 탐색: https://github.com/sickn33/antigravity-awesome-skills
- Google Antigravity 공식 문서: https://antigravity.google/docs/skills
- Anthropic Agent Skills 표준: https://anthropic.com/agent-skills

프롬프트 소설 쓰는 시대는 끝났습니다. 이제는 스킬을 만들고 조합하는 시대입니다. 🚀

---

**작성일자: 2026-01-21**

---

## 관련글 

```
안티그래비티 Agent Skills 제대로 안 쓰면
매번 프롬프트 소설 쓰다가 하루 다 감 ㄹㅇ

요약해서 이렇게 씀.

1) 오픈소스 스킬팩 하나는 무조건 깔아둬
- GitHub에 Antigravity Awesome Skills 레포 있음.
- 안티그래비티/Claude Code/Cursor 같이 쓰는 130+ 스킬 모음임.
- 보안 점검, 린트, 테스트 생성, 성능 분석까지 다 들어있음.

내가 기본으로 켜두는 스킬:
- using-superpowers 
 → 에이전트가 "지금 어떤 스킬 쓸지 먼저 고르고 실행"하게 강제함.
- verification-before-completion 
 → "끝남"이라고 말하기 전에 테스트/빌드부터 돌리게 만듦.

2) 프로젝트에 붙이는 방법
- 레포에 /.agent/skills/ 폴더 만들어서 스킬들 복사해둬.
- 전역 룰에 "항상 using-superpowers 먼저 호출해"라고 적어둬.
- 그러면 답하기 전에 스킬부터 뒤져보고 실행함.

3) 직접 만드는 스킬 예시
- korean-saas-landing 
 → 한국 B2B 랜딩 구조/카피 패턴을 SKILL.md에 정리해둬.
- nextjs-clean-architecture 
 → 폴더 구조, 네이밍, 훅/서비스 분리 기준 다 정의해둬.

스킬은 한 번 만들어두면
그 프로젝트에서 하는 대화마다 계속 재사용됨.
여기서 효율 차이 벌어짐.

새로운게 나오면 맛은 봐야지￼

https://www.threads.com/@akitect_cat/post/DTvVAAykggl

```