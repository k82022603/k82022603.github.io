---
title: "Google Antigravity Agent Skills 완벽 가이드"
date: 2026-01-16 03:00:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,  Antigravity,  agent-skills,  Guide,  Developer,  Claude.write]
---

## AI 에이전트에게 '전문가의 노하우'를 이식하는 방법

> **오픈 스탠다드**: Anthropic 제안 → Google Antigravity, Cursor, Claude 등 채택  
> **공식 문서**: https://antigravity.google/docs/skills

---

## 📖 목차

### PART 1: 개념 편 (Why & What)
1. [Agent Skills란 무엇인가](#agent-skills란-무엇인가)
2. [게임 스킬북 비유로 이해하기](#게임-스킬북-비유로-이해하기)
3. [왜 혁신적인가: Progressive Disclosure](#왜-혁신적인가-progressive-disclosure)
4. [기존 방식의 한계](#기존-방식의-한계)

### PART 2: 실전 편 (How)
5. [SKILL.md 구조 완벽 분석](#skillmd-구조-완벽-분석)
6. [5가지 스킬 패턴](#5가지-스킬-패턴)
7. [실전 예제 5개](#실전-예제-5개)
8. [베스트 프랙티스](#베스트-프랙티스)
9. [일반적인 실수와 해결법](#일반적인-실수와-해결법)
10. [FAQ & 문제 해결](#faq--문제-해결)

---

# PART 1: 개념 편

## Agent Skills란 무엇인가

### 한 줄 정의

**Agent Skills는 AI 에이전트가 특정 작업을 수행하는 방법을 담은 '재사용 가능한 지식 패키지'입니다.**

마치 게임 캐릭터에게 스킬북을 장착하면 새로운 능력을 얻듯이, 에이전트에게 SKILL.md 파일이 담긴 폴더를 제공하면 그 작업의 전문가로 만들 수 있습니다.

### 핵심 구성 요소

```
my-skill/                    # 폴더 = 스킬의 컨테이너
├── SKILL.md                # [필수] 스킬 정의 및 지침
├── scripts/                # [선택] 실행 가능한 스크립트
├── references/             # [선택] 참고 문서
├── examples/               # [선택] Few-shot 예제
└── assets/                 # [선택] 템플릿, 데이터
```

**가장 간단한 형태**:
```
commit-style/
└── SKILL.md    # 이것만 있어도 스킬!
```

### 기존 방식과의 차이

#### ❌ 기존 방식: 매번 설명

```
개발자: "코드 리뷰해줘. 
        보안 취약점 확인하고,
        타입 힌트 있는지 보고,
        SQL 인젝션 체크하고,
        함수명이 컨벤션 따르는지 확인해줘."

[다음 날]
개발자: "이 코드도 리뷰해줘.
        보안 취약점 확인하고..." 
        (똑같은 설명 반복)
```

#### ✅ Agent Skills 방식: 한 번 정의, 영구 사용

```
[1회 설정]
code-review/SKILL.md 생성
→ 보안, 타입, SQL, 네이밍 규칙 정의

[매번 사용]
개발자: "코드 리뷰해줘"
에이전트: [code-review 스킬 자동 로드 & 실행]
```

---

## 게임 스킬북 비유로 이해하기

### 🎮 RPG 게임의 스킬 시스템

```
레벨 1 전사: [기본 공격]
         ↓
스킬북 획득: "불의 검" 스킬북 사용
         ↓
레벨 1 전사: [기본 공격] + [불의 검]
```

**스킬북의 특징**:
- ✅ 한 번 배우면 영구적
- ✅ 다른 캐릭터에게도 적용 가능
- ✅ 레벨업 없이 능력 향상
- ✅ 조합 가능 (검술 + 마법 = 마법검사)

### 🤖 AI 에이전트의 스킬 시스템

```
기본 에이전트: [범용 코딩 능력]
           ↓
SKILL.md 제공: "데이터베이스 마이그레이션" 스킬
           ↓
전문 에이전트: [범용 능력] + [DB 마이그레이션 전문가]
```

**Agent Skills의 특징**:
- ✅ 한 번 작성하면 영구 재사용
- ✅ 팀원 간 공유 가능
- ✅ 추가 학습 없이 즉시 적용
- ✅ 여러 스킬 동시 활성화 가능

### 실제 비유

```
🎮 게임: "화염 마법" 스킬북 장착
   → 캐릭터가 불꽃을 쏠 수 있게 됨

🤖 AI: "Conventional Commits" 스킬 장착
   → 에이전트가 커밋 메시지를 규칙에 맞게 작성

🎮 게임: 여러 스킬북 조합 (검술 + 방어 + 회복)
   → 균형 잡힌 캐릭터

🤖 AI: 여러 스킬 조합 (코드리뷰 + 보안체크 + 테스트생성)
   → 포괄적인 개발 지원
```

---

## 왜 혁신적인가: Progressive Disclosure

### 문제: Context Window Saturation (컨텍스트 포화)

전통적인 방식의 문제:

```
모든 규칙을 한 번에 로드:

시스템 프롬프트 {
  코딩 스타일 가이드 (2,000 토큰)
  + 보안 체크리스트 (1,500 토큰)
  + API 디자인 규칙 (1,800 토큰)
  + 테스트 작성 가이드 (2,200 토큰)
  + 데이터베이스 규칙 (1,700 토큰)
  + 배포 절차 (2,500 토큰)
  = 11,700 토큰 (항상 점유)
}

현재 작업: "함수명 변경" (200 토큰 필요)
→ 11,500 토큰 낭비
→ 느린 응답
→ 높은 비용
→ 도구 혼란 (어느 규칙 적용?)
```

### 해결: Progressive Disclosure (점진적 노출)

```
Skills 방식:

메타데이터만 노출 (100 토큰):
- code-style: "코딩 스타일 검사"
- security: "보안 취약점 검사"
- api-design: "API 엔드포인트 설계"
- test-gen: "테스트 자동 생성"
- db-migration: "데이터베이스 마이그레이션"
- deployment: "프로덕션 배포"

현재 작업: "데이터베이스 테이블 추가"
         ↓
에이전트 판단: "db-migration 스킬 필요"
         ↓
해당 스킬만 로드 (1,700 토큰)
         ↓
효율적 실행

결과:
✅ 100 + 1,700 = 1,800 토큰 (vs 11,700)
✅ 빠른 응답
✅ 낮은 비용
✅ 명확한 컨텍스트
```

### 시각적 비교

```
전통적 방식:
┌─────────────────────────────────┐
│  모든 규칙 (100% 로드)          │ ← 항상 무거움
│  ┌───┬───┬───┬───┬───┬───┐     │
│  │ A │ B │ C │ D │ E │ F │     │
│  └───┴───┴───┴───┴───┴───┘     │
│         ↓                        │
│    단순 작업 처리                │
└─────────────────────────────────┘

Skills 방식:
┌─────────────────────────────────┐
│  메타데이터만 (10% 로드)        │ ← 가벼움
│  ┌───┬───┬───┬───┬───┬───┐     │
│  │ a │ b │ c │ d │ e │ f │     │
│  └─┬─┴───┴───┴───┴───┴───┘     │
│    │                             │
│    ▼ 필요시에만 전체 로드        │
│  ┌─────────┐                    │
│  │ Skill A │                    │
│  │ (100%)  │                    │
│  └─────────┘                    │
└─────────────────────────────────┘
```

---

## 기존 방식의 한계

### 1. 반복적인 프롬프팅

**시나리오**: 매주 코드 리뷰

```
Week 1: "코드 리뷰해줘. 보안 체크, 타입 검사, 네이밍 확인..."
Week 2: "코드 리뷰해줘. 보안 체크, 타입 검사, 네이밍 확인..."
Week 3: "코드 리뷰해줘. 보안..." (지침)
```

**문제**:
- ❌ 매번 동일한 지침 반복
- ❌ 팀원마다 다른 기준
- ❌ 복사-붙여넣기 피로

### 2. 일관성 부족

**시나리오**: 3명의 개발자

```
Developer A: "REST API 만들어줘"
→ 에이전트: [A의 선호도에 맞춰 생성]

Developer B: "REST API 만들어줘"
→ 에이전트: [B의 선호도에 맞춰 생성]

Developer C: "REST API 만들어줘"
→ 에이전트: [C의 선호도에 맞춰 생성]

결과: 3가지 다른 스타일의 API (통일성 없음)
```

### 3. 지식 전파의 어려움

**시나리오**: 시니어 개발자의 노하우

```
시니어: "데이터베이스 마이그레이션은 이렇게 해야 해"
        [30분 설명]
        
주니어: [메모 작성, 일부 누락]

한 달 후...
주니어: "어떻게 하는 거였더라?" (불완전한 기억)
```

**문제**:
- ❌ 구두 전달의 한계
- ❌ 문서화 누락
- ❌ 시간 소모

### 4. 팀 온보딩 비용

**시나리오**: 새로운 팀원 합류

```
Day 1-5: 코딩 스타일 가이드 학습
Day 6-10: 테스트 작성 규칙 학습
Day 11-15: 배포 프로세스 학습
Day 16-20: API 디자인 패턴 학습

= 4주간 생산성 저하
```

---

### Agent Skills가 해결하는 방법

#### ✅ 1. 한 번 정의, 영구 사용

```
code-review/SKILL.md 작성 (1회)
→ 모든 팀원이 동일한 기준으로 리뷰
→ 일관성 100%
```

#### ✅ 2. 지식의 코드화

```
시니어의 노하우를
→ db-migration/SKILL.md로 문서화
→ 주니어도 시니어 수준의 마이그레이션 가능
```

#### ✅ 3. 즉각적인 온보딩

```
Day 1: 프로젝트 clone
     → .agent/skills/ 폴더 포함
     → 모든 팀 규칙 자동 적용
     
= 첫날부터 팀 스탠다드로 작업 가능
```

#### ✅ 4. 팀 간 공유

```
Team A의 "React Component Generator" 스킬
→ Team B가 복사하여 사용
→ Team C가 개선하여 기여
→ 조직 전체의 생산성 향상
```

---

# PART 2: 실전 편

## SKILL.md 구조 완벽 분석

### 최소 구조 (필수 요소만)

```markdown
---
name: skill-name
description: What this skill does and when to use it
---

# Skill Title

Instructions go here...
```

### 완전 구조 (모든 요소)

~~~markdown
---
name: comprehensive-skill
description: Detailed description with trigger keywords
version: 1.0.0
author: team@company.com
tags: [category1, category2]
---

# Skill Title

## Overview
High-level purpose and scope.

## When to Use
Specific scenarios and trigger conditions.

## Prerequisites
- Required tools
- Environment setup
- Permissions needed

## Instructions

### Step 1: Preparation
Detailed first step...

### Step 2: Execution
Detailed second step...

### Step 3: Verification
How to confirm success...

## Examples

### Example 1: Basic Usage
```language
code example
```

### Example 2: Advanced Usage
```language
complex example
```

## Troubleshooting

### Common Issue 1
Problem description and solution.

### Common Issue 2
Problem description and solution.

## Resources
- Link to documentation
- Related skills
- External references
~~~

### YAML Frontmatter 상세 설명

```yaml
---
name: skill-identifier          # [필수] 짧고 명확한 ID (kebab-case)
description: |                  # [필수] 에이전트가 읽는 설명
  When the user asks to "deploy to staging" or
  "test in the staging environment", use this skill
  to safely deploy the current branch.
version: 2.1.0                  # [선택] 버전 관리
author: devops@company.com      # [선택] 유지보수 담당자
tags:                           # [선택] 검색용 태그
  - deployment
  - staging
  - automation
created: 2026-01-15            # [선택] 생성일
updated: 2026-01-16            # [선택] 수정일
---
```

#### Description 작성 황금률

**❌ 나쁜 예**:
```yaml
description: Helps with API stuff
```

**✅ 좋은 예**:
```yaml
description: |
  Generates production-ready REST API endpoint handlers
  in FastAPI following our security and logging conventions.
  Use when the user requests "new API endpoint" or
  "add REST route" or "create API handler".
```

**핵심**:
1. **구체적인 트리거 키워드** 포함
2. **3인칭 시점** 사용 (에이전트가 읽음)
3. **What, When, How** 명확히

---

## 5가지 스킬 패턴

### Pattern 1: Basic Router (SKILL.md만)

**구조**:
```
skill-name/
└── SKILL.md
```

**특징**:
- 가장 간단
- 순수 지침만
- 외부 리소스 없음

**적합한 경우**:
- 간단한 규칙 적용
- 텍스트 변환
- 스타일 가이드

---

### Pattern 2: Reference Pattern (SKILL.md + references/)

**구조**:
```
skill-name/
├── SKILL.md
└── references/
    ├── api-spec.yaml
    ├── naming-conventions.md
    └── security-checklist.txt
```

**특징**:
- 참고 문서 포함
- 에이전트가 필요시 참조
- 대용량 정적 데이터 분리

**사용 예**:
```markdown
# In SKILL.md
When generating API endpoints:
1. Read the OpenAPI specification from references/api-spec.yaml
2. Follow naming conventions in references/naming-conventions.md
3. Verify security requirements using references/security-checklist.txt
```

---

### Pattern 3: Few-Shot Pattern (SKILL.md + examples/)

**구조**:
```
skill-name/
├── SKILL.md
└── examples/
    ├── example1-basic.py
    ├── example2-advanced.py
    └── example3-edge-case.py
```

**특징**:
- 예제로 학습
- Few-shot learning
- 패턴 학습 강화

**사용 예**:
```markdown
# In SKILL.md
Generate components following these examples:
1. examples/example1-basic.py - Simple component
2. examples/example2-advanced.py - With hooks
3. examples/example3-edge-case.py - Error handling
```

---

### Pattern 4: Tool Use Pattern (SKILL.md + scripts/)

**구조**:
```
skill-name/
├── SKILL.md
└── scripts/
    ├── validate.py
    ├── deploy.sh
    └── test-runner.js
```

**특징**:
- 실행 가능한 스크립트
- 복잡한 작업 자동화
- 에이전트가 CLI로 실행

**사용 예**:
```markdown
# In SKILL.md
To deploy to staging:
1. Run `python scripts/validate.py --env staging` to verify
2. Execute `bash scripts/deploy.sh staging` to deploy
3. Use `node scripts/test-runner.js --smoke` to verify
```

---

### Pattern 5: All-in-One (모든 것)

**구조**:
```
skill-name/
├── SKILL.md
├── scripts/
│   ├── setup.sh
│   └── cleanup.py
├── references/
│   └── architecture.md
├── examples/
│   ├── success-case.txt
│   └── failure-case.txt
└── assets/
    ├── template.json
    └── config.yaml
```

**특징**:
- 완전한 패키지
- 복잡한 워크플로우
- 팀 전체 표준

---

## 실전 예제 5개

### 예제 1: Conventional Commits (난이도: ⭐)

**목표**: 커밋 메시지를 Conventional Commits 규칙에 맞게 작성

**디렉토리 구조**:
```
conventional-commits/
└── SKILL.md
```

**SKILL.md**:
~~~markdown
---
name: conventional-commits
description: |
  Enforces Conventional Commits specification for git commit messages.
  Use when the user asks to "commit" or "write commit message" or
  when reviewing commit messages for style compliance.
tags: [git, commits, style]
---

# Conventional Commits Enforcer

## Specification

Commit messages MUST follow this format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types (Required)
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, semicolons, etc.)
- `refactor`: Code change that neither fixes nor adds feature
- `perf`: Performance improvement
- `test`: Adding or updating tests
- `chore`: Changes to build process or auxiliary tools

### Scope (Optional)
Component or module affected: `(api)`, `(ui)`, `(auth)`

### Subject (Required)
- Use imperative mood: "add" not "added" or "adds"
- No capitalized first letter
- No period at the end
- Maximum 50 characters

### Body (Optional)
- Explain WHAT and WHY, not HOW
- Wrap at 72 characters

### Footer (Optional)
- Breaking changes: `BREAKING CHANGE: description`
- Issue references: `Closes #123`

## Examples

### Good ✅
```
feat(auth): add OAuth2 login support

Implement Google and GitHub OAuth2 providers.
Users can now sign in using their existing accounts.

Closes #456
```

### Bad ❌
```
Fixed stuff
```

### Bad ❌
```
Added new feature for authentication with OAuth
```
(Too long subject, capitalized, missing type)

## Validation Process

When the user provides a commit message:

1. Check format: `<type>(<scope>): <subject>`
2. Verify type is one of the allowed types
3. Ensure subject is imperative, lowercase, <50 chars
4. If body exists, check 72 char wrapping
5. Validate footer format if present

If invalid:
- Explain the specific violation
- Provide corrected version
- Explain why the change improves clarity
~~~

**사용 예**:
```
사용자: "커밋 메시지 작성해줘: 로그인 버그 수정했어"

에이전트: [conventional-commits 스킬 로드]
fix(auth): resolve login timeout issue

The login process was timing out due to incorrect
session token expiration handling. Updated the token
refresh logic to properly extend sessions.

Closes #789
```

---

### 예제 2: Code Review with Security Focus (난이도: ⭐⭐)

**목표**: 보안 중심의 체계적인 코드 리뷰

**디렉토리 구조**:
```
code-review-security/
├── SKILL.md
└── references/
    ├── owasp-top-10.md
    ├── secure-coding-guidelines.md
    └── common-vulnerabilities.json
```

**SKILL.md**:
~~~markdown
---
name: code-review-security
description: |
  Performs comprehensive code review with emphasis on security vulnerabilities.
  Checks for OWASP Top 10, secure coding practices, and common attack vectors.
  Use when reviewing pull requests or performing security audits.
tags: [security, code-review, owasp]
version: 2.0.0
---

# Security-Focused Code Review

## Overview
This skill performs a multi-layered code review prioritizing security concerns
while also checking for code quality, performance, and maintainability.

## Review Process

### Phase 1: Security Critical Checks

#### 1.1 Authentication & Authorization
- [ ] Are credentials hardcoded? → **CRITICAL VIOLATION**
- [ ] Is authentication required for sensitive endpoints?
- [ ] Are authorization checks present and correct?
- [ ] Are JWT tokens validated properly?
- [ ] Is password hashing using bcrypt/argon2? (not MD5/SHA1)

#### 1.2 Input Validation
- [ ] All user inputs sanitized?
- [ ] SQL queries use parameterized statements? (not string concat)
- [ ] File uploads have type/size restrictions?
- [ ] Path traversal prevention for file operations?
- [ ] XSS prevention in rendered output?

#### 1.3 Data Protection
- [ ] Sensitive data encrypted at rest?
- [ ] HTTPS enforced for data in transit?
- [ ] API keys in environment variables (not code)?
- [ ] Database credentials properly secured?
- [ ] PII handling compliant with regulations?

### Phase 2: Code Quality

#### 2.1 Type Safety
- [ ] Functions have type hints (Python) / types (TypeScript)?
- [ ] Return types specified?
- [ ] Optional parameters handled?

#### 2.2 Error Handling
- [ ] Try-except blocks present for risky operations?
- [ ] Errors logged appropriately?
- [ ] No sensitive data in error messages?
- [ ] Graceful degradation implemented?

#### 2.3 Code Organization
- [ ] Single Responsibility Principle followed?
- [ ] Functions <50 lines?
- [ ] Cyclomatic complexity <10?
- [ ] Dead code removed?

### Phase 3: Performance

- [ ] Database queries optimized? (N+1 queries avoided)
- [ ] Caching used where appropriate?
- [ ] Large files streamed (not loaded into memory)?
- [ ] Async operations for I/O bound tasks?

## Output Format

```markdown
# Code Review Report

## 🚨 Security Issues (Must Fix)
1. **SQL Injection Risk** (Line 45)
   - Issue: String concatenation in query
   - Fix: Use parameterized query
   - Example: `cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))`

## ⚠️ Security Warnings (Should Fix)
1. **Weak Password Hashing** (Line 78)
   - Issue: Using SHA256 instead of bcrypt
   - Fix: Migrate to bcrypt with salt

## ✅ Code Quality Suggestions
1. **Type Hints Missing** (Line 12)
   - Add return type annotation
   - Example: `def get_user(id: int) -> Optional[User]:`

## 📊 Metrics
- Security Score: 7/10
- Code Quality: 8/10
- Test Coverage: 85%

## Summary
[Overall assessment and priority fixes]
```

## References

For detailed guidance, consult:
- `references/owasp-top-10.md` - Top security risks
- `references/secure-coding-guidelines.md` - Best practices
- `references/common-vulnerabilities.json` - CVE patterns

## Example Reviews

### Scenario: Login Function Review

**Code Submitted**:
```python
def login(username, password):
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    result = db.execute(query)
    return result
```

**Review Output**:
```markdown
## 🚨 CRITICAL SECURITY ISSUES

1. **SQL Injection Vulnerability** (Line 2) - SEVERITY: CRITICAL
   - Current: String concatenation allows injection
   - Attack: username = "admin'--" bypasses password check
   - Fix:
   ```python
   query = "SELECT * FROM users WHERE username=? AND password=?"
   result = db.execute(query, (username, password))
   ```

2. **Password Storage** (Line 2) - SEVERITY: CRITICAL
   - Plaintext password comparison
   - Must hash passwords with bcrypt
   - Fix:
   ```python
   import bcrypt
   
   def login(username, password):
       query = "SELECT * FROM users WHERE username=?"
       user = db.execute(query, (username,)).fetchone()
       if user and bcrypt.checkpw(password.encode(), user['password_hash']):
           return user
       return None
   ```

3. **Missing Type Hints** (Line 1)
   ```python
   def login(username: str, password: str) -> Optional[Dict]:
   ```

## Recommendation
**DO NOT MERGE** - Critical security vulnerabilities must be fixed first.
```
~~~

**사용 예**:
```
사용자: "PR #123 리뷰해줘"

에이전트: [code-review-security 스킬 로드]
         [references/ 문서 참조]
         [체계적 리뷰 수행]
         [보고서 생성]
```

---

### 예제 3: License Header Adder (난이도: ⭐⭐)

**목표**: 새 소스 파일에 자동으로 라이센스 헤더 추가

**디렉토리 구조**:
```
license-header-adder/
├── SKILL.md
└── resources/
    ├── HEADER.txt
    ├── HEADER_PYTHON.txt
    ├── HEADER_JAVA.txt
    └── HEADER_JS.txt
```

**resources/HEADER.txt**:
```
/*
 * Copyright (c) 2026 YourCompany LLC.
 * All rights reserved.
 * 
 * This code is proprietary and confidential.
 * Unauthorized copying, distribution, or use is strictly prohibited.
 */
```

**resources/HEADER_PYTHON.txt**:
```python
# Copyright (c) 2026 YourCompany LLC.
# All rights reserved.
#
# This code is proprietary and confidential.
# Unauthorized copying, distribution, or use is strictly prohibited.
```

**SKILL.md**:
~~~markdown
---
name: license-header-adder
description: |
  Adds the standard corporate license header to new source files.
  Automatically detects file type and uses appropriate comment style.
  Use when creating new files or when user asks to "add license" or
  "add copyright header".
tags: [licensing, copyright, header]
---

# License Header Adder

## Overview
Automatically adds company copyright and license headers to source files.
Headers are stored in the `resources/` folder to avoid bloating the context.

## Supported File Types

| Extension | Header File | Comment Style |
|-----------|-------------|---------------|
| `.py` | HEADER_PYTHON.txt | `#` |
| `.java`, `.c`, `.cpp` | HEADER_JAVA.txt | `/* */` |
| `.js`, `.ts` | HEADER_JS.txt | `/* */` |
| All others | HEADER.txt | Default `/* */` |

## Instructions

### When creating a new file:

1. Determine file extension
2. Read the appropriate header from `resources/HEADER_<TYPE>.txt`
3. Insert header at the top of the file
4. Add one blank line after the header
5. Add the actual code content

### When adding header to existing file:

1. Check if header already exists (search for "Copyright (c)")
2. If exists, skip (avoid duplicates)
3. If not, insert at the very top

## Example Output

### Python File
```python
# Copyright (c) 2026 YourCompany LLC.
# All rights reserved.
#
# This code is proprietary and confidential.

import os
import sys

def main():
    pass
```

### JavaScript File
```javascript
/*
 * Copyright (c) 2026 YourCompany LLC.
 * All rights reserved.
 */

const express = require('express');
const app = express();
```

## Edge Cases

### Already has header
- Action: Skip, notify user "Header already present"

### Custom header requested
- Action: Ask user to update resources/HEADER_<TYPE>.txt first

### Multiple files
- Action: Process each file, report summary:
  ```
  ✅ Added header to: file1.py, file2.js
  ⚠️  Skipped (already has header): file3.py
  ```
~~~

**사용 예**:
```
사용자: "새 Python 파일 만들어줘: user_service.py"

에이전트: [license-header-adder 스킬 로드]
         [resources/HEADER_PYTHON.txt 읽기]
         [헤더 추가된 파일 생성]

결과:
# Copyright (c) 2026 YourCompany LLC.
# All rights reserved.
#
# This code is proprietary and confidential.

class UserService:
    def __init__(self):
        pass
```

---

### 예제 4: Staging Deployment (난이도: ⭐⭐⭐)

**목표**: 안전한 스테이징 환경 배포 자동화

**디렉토리 구조**:
```
deploy-staging/
├── SKILL.md
├── scripts/
│   ├── pre-deploy-check.sh
│   ├── deploy.py
│   └── health-check.js
└── references/
    └── deployment-checklist.md
```

**scripts/pre-deploy-check.sh**:
```bash
#!/bin/bash
# Pre-deployment validation

echo "🔍 Running pre-deployment checks..."

# Check git status
if [[ -n $(git status -s) ]]; then
    echo "❌ Error: Uncommitted changes found"
    exit 1
fi

# Check branch
BRANCH=$(git rev-parse --abbrev-ref HEAD)
if [[ "$BRANCH" == "main" ]] || [[ "$BRANCH" == "production" ]]; then
    echo "❌ Error: Cannot deploy main/production branch to staging"
    exit 1
fi

# Run tests
echo "Running tests..."
npm test || exit 1

# Check environment file
if [[ ! -f .env.staging ]]; then
    echo "❌ Error: .env.staging not found"
    exit 1
fi

echo "✅ All pre-deployment checks passed"
exit 0
```

**scripts/deploy.py**:
```python
#!/usr/bin/env python3
import sys
import subprocess
import requests
import time

def deploy_to_staging():
    """Deploy application to staging environment"""
    
    print("🚀 Starting deployment to staging...")
    
    # Build Docker image
    print("📦 Building Docker image...")
    result = subprocess.run(
        ["docker", "build", "-t", "myapp:staging", "."],
        capture_output=True
    )
    if result.returncode != 0:
        print(f"❌ Build failed: {result.stderr.decode()}")
        sys.exit(1)
    
    # Push to registry
    print("⬆️  Pushing to container registry...")
    subprocess.run([
        "docker", "tag", "myapp:staging", 
        "registry.company.com/myapp:staging"
    ])
    subprocess.run([
        "docker", "push", 
        "registry.company.com/myapp:staging"
    ])
    
    # Deploy to Kubernetes
    print("☸️  Deploying to Kubernetes...")
    subprocess.run([
        "kubectl", "set", "image", 
        "deployment/myapp", 
        "myapp=registry.company.com/myapp:staging",
        "-n", "staging"
    ])
    
    # Wait for rollout
    print("⏳ Waiting for rollout...")
    subprocess.run([
        "kubectl", "rollout", "status", 
        "deployment/myapp", 
        "-n", "staging",
        "--timeout=5m"
    ])
    
    print("✅ Deployment completed!")
    return True

if __name__ == "__main__":
    deploy_to_staging()
```

**scripts/health-check.js**:
```javascript
const axios = require('axios');

async function checkHealth(url, maxRetries = 10) {
    console.log(`🏥 Checking health at ${url}...`);
    
    for (let i = 0; i < maxRetries; i++) {
        try {
            const response = await axios.get(`${url}/health`);
            
            if (response.status === 200 && response.data.status === 'healthy') {
                console.log('✅ Health check passed!');
                console.log(`   Version: ${response.data.version}`);
                console.log(`   Uptime: ${response.data.uptime}s`);
                return true;
            }
        } catch (error) {
            console.log(`⏳ Attempt ${i + 1}/${maxRetries} failed, retrying...`);
            await new Promise(resolve => setTimeout(resolve, 5000));
        }
    }
    
    console.log('❌ Health check failed after all retries');
    return false;
}

const STAGING_URL = process.env.STAGING_URL || 'https://staging.company.com';
checkHealth(STAGING_URL).then(success => {
    process.exit(success ? 0 : 1);
});
```

**SKILL.md**:
~~~markdown
---
name: deploy-staging
description: |
  Deploys the current branch to the staging environment with full validation.
  Includes pre-deployment checks, Docker build, Kubernetes deployment, and
  health verification. Use when user asks to "deploy to staging" or
  "test in staging environment" or "push to staging".
tags: [deployment, staging, devops, kubernetes]
version: 3.1.0
---

# Deploy to Staging

## Overview
Fully automated staging deployment with safety checks, rollout monitoring,
and health verification.

## Prerequisites

- Docker installed and running
- kubectl configured for staging cluster
- .env.staging file present
- All tests passing
- No uncommitted changes

## Deployment Process

### Step 1: Pre-Deployment Validation

Execute the pre-deployment check script:

```bash
bash scripts/pre-deploy-check.sh
```

This will verify:
- ✅ Git status is clean
- ✅ Not on main/production branch
- ✅ All tests pass
- ✅ Environment config exists

If ANY check fails → STOP deployment immediately.

### Step 2: Build & Deploy

Run the deployment script:

```bash
python scripts/deploy.py
```

This performs:
1. Docker image build
2. Push to container registry
3. Kubernetes deployment update
4. Rollout status monitoring

Expected output:
```
🚀 Starting deployment to staging...
📦 Building Docker image...
⬆️  Pushing to container registry...
☸️  Deploying to Kubernetes...
⏳ Waiting for rollout...
✅ Deployment completed!
```

### Step 3: Health Verification

Verify the deployment is healthy:

```bash
node scripts/health-check.js
```

This will:
- Send requests to /health endpoint
- Retry up to 10 times (5 seconds apart)
- Verify response status and version

Success output:
```
🏥 Checking health at https://staging.company.com...
✅ Health check passed!
   Version: v2.3.1
   Uptime: 45s
```

### Step 4: Notification

After successful deployment, provide:

1. **Deployment Summary**:
   ```
   ✅ Staging Deployment Successful
   
   Branch: feature/user-auth
   Commit: a1b2c3d
   Version: v2.3.1
   URL: https://staging.company.com
   
   Health Status: ✅ Healthy
   ```

2. **Test Instructions**:
   ```
   To test:
   1. Visit https://staging.company.com
   2. Verify new feature: [describe feature]
   3. Run smoke tests: npm run test:smoke
   ```

3. **Rollback Instructions** (just in case):
   ```
   To rollback:
   kubectl rollout undo deployment/myapp -n staging
   ```

## Error Handling

### Build Failure
```
❌ Build failed: [error message]
```
→ Fix code issues, then retry

### Deployment Timeout
```
❌ Rollout exceeded timeout (5m)
```
→ Check Kubernetes logs:
```bash
kubectl logs -f deployment/myapp -n staging
```

### Health Check Failure
```
❌ Health check failed after all retries
```
→ Investigate:
1. Check application logs
2. Verify database connectivity
3. Check configuration

**Decision**: Rollback or debug?

## Safety Features

### Automatic Rollback Trigger
If health check fails 3 times in a row → Auto-rollback

### Monitoring
After deployment, monitor for 10 minutes:
- Error rate
- Response time
- CPU/Memory usage

### Gradual Rollout
Deployment uses rolling update:
- Max surge: 1
- Max unavailable: 0
→ Zero downtime

## Reference Documentation

For detailed procedures, see:
- `references/deployment-checklist.md`

## Workflow Summary

```
User Request
    ↓
Pre-Deployment Check
    ↓ (pass)
Build & Deploy
    ↓ (success)
Health Verification
    ↓ (healthy)
✅ DONE + Notify User
```

Any failure → STOP and report error
~~~

**사용 예**:
```
사용자: "스테이징에 배포해줘"

에이전트: [deploy-staging 스킬 로드]

Step 1: bash scripts/pre-deploy-check.sh
  ✅ Git clean
  ✅ Tests passed
  ✅ Config exists

Step 2: python scripts/deploy.py
  📦 Building...
  ⬆️  Pushing...
  ☸️  Deploying...
  ✅ Deployed!

Step 3: node scripts/health-check.js
  ✅ Healthy

에이전트 응답:
"
✅ Staging Deployment Successful

Branch: feature/oauth
Commit: f4e3a21
Version: v2.5.0
URL: https://staging.company.com

Health Status: ✅ Healthy

You can now test the OAuth login feature at the staging URL.
"
```

---

### 예제 5: React Component Generator (난이도: ⭐⭐⭐⭐)

**목표**: 프로덕션급 React 컴포넌트 전체 패키지 생성

**디렉토리 구조**:
```
react-component-generator/
├── SKILL.md
├── scripts/
│   └── generate-component.js
├── examples/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.types.ts
│   │   ├── Button.test.tsx
│   │   └── Button.stories.tsx
│   └── Input/
│       ├── Input.tsx
│       ├── Input.types.ts
│       ├── Input.test.tsx
│       └── Input.stories.tsx
└── assets/
    ├── component.template.tsx
    ├── types.template.ts
    ├── test.template.tsx
    └── stories.template.tsx
```

**SKILL.md**:
~~~markdown
---
name: react-component-generator
description: |
  Generates production-ready React component with TypeScript, unit tests,
  Storybook stories, and complete documentation. Includes proper type definitions,
  accessibility attributes, and best practices. Use when creating new UI components.
tags: [react, typescript, components, testing, storybook]
version: 4.0.0
---

# React Component Generator

## Overview
Creates a complete React component package following our team's standards:
- TypeScript with strict typing
- Jest + React Testing Library tests
- Storybook stories for documentation
- Accessibility (a11y) compliant
- Styled with Tailwind CSS

## Generated Structure

For a component named `ComponentName`:

```
ComponentName/
├── ComponentName.tsx           # Main component
├── ComponentName.types.ts      # TypeScript interfaces
├── ComponentName.test.tsx      # Unit tests
├── ComponentName.stories.tsx   # Storybook documentation
├── index.ts                    # Barrel export
└── README.md                   # Component documentation
```

## Component Template

### ComponentName.tsx
```typescript
import React from 'react';
import { ComponentNameProps } from './ComponentName.types';

/**
 * ComponentName - [Brief description]
 * 
 * @example
 * ```tsx
 * <ComponentName prop1="value" />
 * ```
 */
export const ComponentName: React.FC<ComponentNameProps> = ({
  // Props with defaults
  variant = 'primary',
  size = 'medium',
  disabled = false,
  children,
  onClick,
  ...rest
}) => {
  // Component logic here
  
  return (
    <div
      className={cn(
        'base-styles',
        variantStyles[variant],
        sizeStyles[size],
        disabled && 'opacity-50 cursor-not-allowed'
      )}
      role="button"
      tabIndex={disabled ? -1 : 0}
      onClick={disabled ? undefined : onClick}
      {...rest}
    >
      {children}
    </div>
  );
};

ComponentName.displayName = 'ComponentName';
```

### ComponentName.types.ts
```typescript
import { ReactNode, MouseEvent } from 'react';

export interface ComponentNameProps {
  /** Component variant */
  variant?: 'primary' | 'secondary' | 'ghost';
  
  /** Component size */
  size?: 'small' | 'medium' | 'large';
  
  /** Disabled state */
  disabled?: boolean;
  
  /** Click handler */
  onClick?: (event: MouseEvent<HTMLDivElement>) => void;
  
  /** Child elements */
  children: ReactNode;
  
  /** Additional CSS classes */
  className?: string;
  
  /** Test ID for testing */
  'data-testid'?: string;
}
```

### ComponentName.test.tsx
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  describe('Rendering', () => {
    it('renders with required props', () => {
      render(<ComponentName>Test Content</ComponentName>);
      
      expect(screen.getByText('Test Content')).toBeInTheDocument();
    });
    
    it('applies variant styles correctly', () => {
      const { rerender } = render(
        <ComponentName variant="primary">Primary</ComponentName>
      );
      expect(screen.getByText('Primary')).toHaveClass('variant-primary');
      
      rerender(<ComponentName variant="secondary">Secondary</ComponentName>);
      expect(screen.getByText('Secondary')).toHaveClass('variant-secondary');
    });
  });
  
  describe('Interactions', () => {
    it('handles click events', () => {
      const handleClick = jest.fn();
      render(<ComponentName onClick={handleClick}>Click Me</ComponentName>);
      
      fireEvent.click(screen.getByText('Click Me'));
      expect(handleClick).toHaveBeenCalledTimes(1);
    });
    
    it('does not fire click when disabled', () => {
      const handleClick = jest.fn();
      render(
        <ComponentName onClick={handleClick} disabled>
          Disabled
        </ComponentName>
      );
      
      fireEvent.click(screen.getByText('Disabled'));
      expect(handleClick).not.toHaveBeenCalled();
    });
  });
  
  describe('Accessibility', () => {
    it('has proper ARIA attributes', () => {
      render(<ComponentName>Accessible</ComponentName>);
      
      const component = screen.getByRole('button');
      expect(component).toHaveAttribute('tabIndex', '0');
    });
    
    it('sets tabIndex to -1 when disabled', () => {
      render(<ComponentName disabled>Disabled</ComponentName>);
      
      expect(screen.getByRole('button')).toHaveAttribute('tabIndex', '-1');
    });
  });
});
```

### ComponentName.stories.tsx
```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { ComponentName } from './ComponentName';

const meta: Meta<typeof ComponentName> = {
  title: 'Components/ComponentName',
  component: ComponentName,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'ghost'],
    },
    size: {
      control: 'select',
      options: ['small', 'medium', 'large'],
    },
    disabled: {
      control: 'boolean',
    },
  },
};

export default meta;
type Story = StoryObj<typeof ComponentName>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Interactive: Story = {
  args: {
    children: 'Click Me',
  },
  play: async ({ canvasElement }) => {
    // Interaction tests
  },
};
```

### index.ts (Barrel Export)
```typescript
export { ComponentName } from './ComponentName';
export type { ComponentNameProps } from './ComponentName.types';
```

## Generation Process

### Step 1: Gather Requirements

Ask the user:
1. Component name (PascalCase)
2. Component purpose
3. Required props
4. Variants (if any)
5. Special behaviors

### Step 2: Create Directory Structure

```bash
mkdir -p src/components/ComponentName
cd src/components/ComponentName
```

### Step 3: Generate Files

Use the templates from `assets/` as base:
1. Generate ComponentName.tsx from component.template.tsx
2. Generate ComponentName.types.ts from types.template.ts
3. Generate ComponentName.test.tsx from test.template.tsx
4. Generate ComponentName.stories.tsx from stories.template.tsx
5. Create index.ts
6. Create README.md

### Step 4: Customize Content

Replace placeholders:
- `{{ComponentName}}` → Actual component name
- `{{props}}` → Component props from requirements
- `{{description}}` → Component description
- `{{examples}}` → Usage examples

### Step 5: Validation

Run:
```bash
# Type check
npx tsc --noEmit

# Test
npm test ComponentName

# Lint
npm run lint src/components/ComponentName
```

### Step 6: Documentation

Create README.md:
```markdown
# ComponentName

[Description]

## Usage

```tsx
import { ComponentName } from '@/components/ComponentName';

function App() {
  return <ComponentName variant="primary">Click Me</ComponentName>;
}
```

## Props

See `ComponentName.types.ts` for full prop documentation.

## Examples

View all examples in Storybook:
```
npm run storybook
```
```

## Best Practices Applied

### TypeScript
- ✅ Strict typing enabled
- ✅ Props interface exported
- ✅ No `any` types
- ✅ Proper generics where needed

### Accessibility
- ✅ Semantic HTML
- ✅ ARIA attributes
- ✅ Keyboard navigation
- ✅ Focus management
- ✅ Screen reader support

### Testing
- ✅ Rendering tests
- ✅ Interaction tests
- ✅ Accessibility tests
- ✅ Edge case coverage
- ✅ >80% coverage target

### Performance
- ✅ React.memo if needed
- ✅ useMemo for expensive computations
- ✅ useCallback for event handlers
- ✅ Lazy loading for heavy components

### Code Quality
- ✅ ESLint compliant
- ✅ Prettier formatted
- ✅ JSDoc comments
- ✅ DisplayName set
- ✅ PropTypes (if using JS)

## Example Prompts

### Simple Component
```
"Create a Card component that displays a title, description, and optional image"
```

### Complex Component
```
"Create a DataTable component with sorting, filtering, pagination, 
and row selection. Include export to CSV functionality."
```

### Form Component
```
"Create a SearchBar component with autocomplete suggestions,
debounced input, and keyboard navigation"
```

## Advanced Features

### Compound Components Pattern
```typescript
// Parent
export const Tabs: React.FC<TabsProps> & {
  Tab: typeof Tab;
  Panel: typeof Panel;
} = ({ children, defaultValue }) => {
  // ...
};

// Children
Tabs.Tab = Tab;
Tabs.Panel = Panel;

// Usage
<Tabs defaultValue="tab1">
  <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
  <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
</Tabs>
```

### Render Props Pattern
```typescript
interface RenderProps {
  isOpen: boolean;
  toggle: () => void;
}

<Toggle>
  {({ isOpen, toggle }: RenderProps) => (
    <button onClick={toggle}>
      {isOpen ? 'Close' : 'Open'}
    </button>
  )}
</Toggle>
```

### Custom Hooks Pattern
```typescript
// Extract logic to custom hook
function useComponentName(initialValue: string) {
  const [value, setValue] = useState(initialValue);
  
  const reset = () => setValue(initialValue);
  
  return { value, setValue, reset };
}

// Use in component
const { value, setValue, reset } = useComponentName('default');
```

## References

- Examples: See `examples/Button` and `examples/Input`
- Templates: See `assets/*.template.tsx`
- Team Guidelines: [Link to internal docs]
~~~

**사용 예**:
```
사용자: "검색 기능이 있는 Input 컴포넌트 만들어줘. 
        자동완성 제안 표시하고, debounce 적용하고,
        키보드로 선택 가능하게 해줘."

에이전트: [react-component-generator 스킬 로드]

질문:
1. Component name? → "SearchInput"
2. Debounce delay? → "300ms"
3. Max suggestions? → "5"

생성 중...

✅ Generated:
   SearchInput/
   ├── SearchInput.tsx (자동완성 로직 포함)
   ├── SearchInput.types.ts (props 정의)
   ├── SearchInput.test.tsx (11 tests, 95% coverage)
   ├── SearchInput.stories.tsx (5 stories)
   ├── index.ts
   └── README.md

Features implemented:
- ✅ Debounced input (300ms)
- ✅ Autocomplete dropdown
- ✅ Keyboard navigation (↑↓ Enter Esc)
- ✅ Click outside to close
- ✅ Loading state
- ✅ Accessibility (ARIA)
- ✅ Fully typed
- ✅ Test coverage: 95%

Run Storybook to preview:
  npm run storybook
```

---

## 베스트 프랙티스

### 1. Description은 에이전트가 읽는다

**❌ 사람용 작성**:
```yaml
description: This is a cool skill that helps you
```

**✅ 에이전트용 작성**:
```yaml
description: |
  Generates REST API endpoint handlers when the user requests
  "new API", "create endpoint", or "add route". 
  Includes validation, error handling, and tests.
```

### 2. Single Responsibility

**❌ 만능 스킬**:
```
mega-skill/
└── SKILL.md  # Does everything: code review, deployment, testing...
```

**✅ 전문화된 스킬들**:
```
code-review/
deploy-staging/
generate-tests/
security-audit/
```

### 3. Scripts를 Black Box로 취급

**❌ 스크립트 코드를 SKILL.md에 포함**:
~~~markdown
Run this Python code:
```python
# 100 lines of Python...
```
~~~

**✅ 스크립트 실행만 지시**:
~~~markdown
Execute the validation script:
```bash
python scripts/validate.py --env staging
```

Script will:
- Check database connectivity
- Verify API endpoints
- Test authentication
~~~

### 4. 버전 관리

```yaml
---
name: deploy-production
version: 3.2.1
updated: 2026-01-16
changelog: |
  3.2.1 - Added rollback automation
  3.2.0 - Kubernetes support
  3.1.0 - Health check improvements
---
```

### 5. Conditional Logic을 명시

**❌ 모호한 지침**:
```markdown
Deploy if everything looks good
```

**✅ 명확한 조건문**:
```markdown
## Deployment Decision Tree

IF git status is dirty:
  → STOP: "Uncommitted changes found"

ELSE IF tests fail:
  → STOP: "Tests must pass before deployment"

ELSE IF on main branch:
  → ASK: "Deploy to production? This is irreversible. (yes/no)"
  
ELSE:
  → PROCEED: Deploy to staging
```

### 6. Examples > Explanations

**❌ 추상적 설명**:
```markdown
Follow good commit message practices
```

**✅ 구체적 예제**:
~~~markdown
## Good Commit Message
```
feat(auth): add OAuth2 login support

Implement Google and GitHub OAuth providers.

Closes #123
```

## Bad Commit Message
```
fixed stuff
```
~~~

### 7. Progressive Enhancement

**시작**: 기본 스킬
```
skill-v1/
└── SKILL.md  # 핵심 기능만
```

**발전**: 참고 자료 추가
```
skill-v2/
├── SKILL.md
└── references/
    └── guidelines.md
```

**성숙**: 완전한 패키지
```
skill-v3/
├── SKILL.md
├── scripts/
├── references/
├── examples/
└── assets/
```

---

## 일반적인 실수와 해결법

### 실수 1: Description이 너무 일반적

**❌**:
```yaml
description: Helps with coding
```

**문제**: 에이전트가 언제 사용할지 모름

**✅**:
```yaml
description: |
  Reviews Python code for PEP 8 compliance, type hints,
  and security issues. Use when reviewing pull requests
  or performing code audits.
```

---

### 실수 2: 지침이 너무 모호

**❌**:
```markdown
Make sure the code is good
```

**✅**:
```markdown
Verify:
1. All functions have type hints
2. No SQL string concatenation
3. Error handling present
4. Test coverage >80%
```

---

### 실수 3: 스크립트 경로 오류

**❌**:
```markdown
Run `scripts/deploy.sh`
```
(절대 경로 가정)

**✅**:
~~~markdown
From the skill directory, run:
```bash
bash ./scripts/deploy.sh
```

Or provide absolute path:
```bash
bash ~/.gemini/antigravity/skills/deploy/scripts/deploy.sh
```
~~~

---

### 실수 4: 리소스 파일을 Context에 덤프

**❌**:
```markdown
Use this license header:
[500 lines of license text pasted here]
```

**✅**:
```markdown
Read the license header from:
`resources/LICENSE_HEADER.txt`

Then prepend it to the file.
```

---

### 실수 5: 에러 처리 누락

**❌**:
```markdown
Run the deployment script
```

**✅**:
~~~markdown
Run deployment:
```bash
python scripts/deploy.py
```

IF exit code != 0:
  → Check logs in /tmp/deploy.log
  → Report error to user
  → Ask: "Retry or rollback?"

ELSE:
  → Proceed to health check
~~~

---

### 실수 6: Workspace vs Global 혼동

**잘못된 배치**:
```
Global: project-specific React patterns
Workspace: Universal code review rules
```

**올바른 배치**:
```
Global: Universal code review rules
Workspace: Project-specific React patterns
```

**원칙**:
- **Global**: 모든 프로젝트에 적용 (코딩 스타일, 보안 기본)
- **Workspace**: 프로젝트 특화 (특정 프레임워크, 팀 컨벤션)

---

## FAQ & 문제 해결

### Q1: 스킬이 로드되지 않아요

**증상**: 에이전트가 스킬을 인식하지 못함

**체크리스트**:
1. ✅ 파일명이 정확히 `SKILL.md`인가? (대소문자 구분)
2. ✅ YAML frontmatter가 올바른가? (`---` 로 감싸짐)
3. ✅ name과 description이 있는가?
4. ✅ 경로가 맞는가?
   ```
   Global: ~/.gemini/antigravity/skills/skill-name/SKILL.md
   Workspace: .agent/skills/skill-name/SKILL.md
   ```
5. ✅ Antigravity를 재시작했는가?

**해결**:
```bash
# 경로 확인
ls -la ~/.gemini/antigravity/skills/
ls -la .agent/skills/

# SKILL.md 검증
cat .agent/skills/my-skill/SKILL.md | head -10

# Antigravity 재시작
```

---

### Q2: 에이전트가 다른 스킬을 사용해요

**증상**: "배포해줘" → 코드 리뷰 스킬이 실행됨

**원인**: Description의 키워드 충돌

**예시**:
```yaml
# deploy skill
description: Deploy code to staging  # "code" 키워드

# code-review skill  
description: Review code quality     # "code" 키워드
```

**해결**:
```yaml
# deploy skill
description: |
  Deploy application to staging environment.
  Use when user requests "deploy", "push to staging",
  or "release to staging".

# code-review skill
description: |
  Review pull requests for code quality and security.
  Use when user requests "review PR", "code review",
  or "check code quality".
```

**핵심**: 트리거 키워드를 명확히 분리

---

### Q3: 스크립트가 실행되지 않아요

**증상**: Permission denied

**원인**: 실행 권한 없음

**해결**:
```bash
chmod +x scripts/*.sh
chmod +x scripts/*.py
```

**또는 SKILL.md에서**:
~~~markdown
Run:
```bash
bash scripts/deploy.sh   # bash 명시
python scripts/test.py   # python 명시
```
~~~

---

### Q4: Global vs Workspace 스킬이 충돌해요

**증상**: 같은 이름의 스킬이 2개

**우선순위**:
```
Workspace > Global
```

**예시**:
```
~/.gemini/antigravity/skills/deploy/SKILL.md      (Global)
.agent/skills/deploy/SKILL.md                     (Workspace)

→ Workspace 버전이 사용됨
```

**활용**:
```
Global: 기본 배포 프로세스
Workspace: 프로젝트별 커스터마이징
```

---

### Q5: 스킬이 너무 많아 느려요

**증상**: 에이전트 응답이 느림

**원인**: 스킬 메타데이터가 많음 (>50개)

**해결책 1**: 스킬 정리
```bash
# 사용 안 하는 스킬 삭제
rm -rf ~/.gemini/antigravity/skills/unused-skill

# 또는 비활성화 (이름 변경)
mv skill-name skill-name.disabled
```

**해결책 2**: Workspace 스킬로 분산
```
Global: 10개 (범용)
Project A: 5개 (프로젝트 특화)
Project B: 5개 (프로젝트 특화)
```

**해결책 3**: 스킬 통합
```
Before:
- lint-python
- lint-javascript  
- lint-typescript

After:
- code-linter (모든 언어 지원)
```

---

### Q6: 팀원과 스킬을 공유하고 싶어요

**방법 1**: Git으로 버전 관리
```bash
# 프로젝트 루트에 스킬 포함
.agent/skills/
  ├── team-skill-1/
  └── team-skill-2/

# Git에 커밋
git add .agent/skills/
git commit -m "Add team skills"
git push

# 팀원이 pull하면 자동 적용
git pull
```

**방법 2**: 심볼릭 링크 (Global 공유)
```bash
# 중앙 저장소
~/company-skills/
  ├── code-review/
  └── deployment/

# 각자 심볼릭 링크
ln -s ~/company-skills/* ~/.gemini/antigravity/skills/
```

**방법 3**: 스킬 마켓플레이스 (미래)
```
(Coming soon: Antigravity Skill Marketplace)
```

---

## 다음 단계

### 1. 첫 스킬 만들기

~~~bash
# 1. 디렉토리 생성
mkdir -p .agent/skills/my-first-skill

# 2. SKILL.md 작성
cat > .agent/skills/my-first-skill/SKILL.md << 'EOF'
---
name: commit-message-helper
description: |
  Helps write clear commit messages.
  Use when user asks to "write commit message" or "commit".
---

# Commit Message Helper

Format: `<type>: <subject>`

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation

Example:
```
feat: add user authentication
```
EOF

# 3. Antigravity 재시작 후 테스트
# "커밋 메시지 작성해줘: 로그인 기능 추가"
~~~

### 2. 팀 스킬 라이브러리 구축

```
Week 1: code-review
Week 2: deployment
Week 3: test-generation
Week 4: documentation
```

### 3. 커뮤니티 기여

- GitHub에 스킬 공개
- 베스트 프랙티스 공유
- 피드백 수렴

---

## 추가 리소스

### 공식 문서
- https://antigravity.google/docs/skills
- https://github.com/google/antigravity-skills (예정)

### 예제 저장소
- https://github.com/romin-irani/antigravity-skills-examples

### 커뮤니티
- Google AI Developers Forum
- Discord: Antigravity Users

### 학습 자료
- Google Codelabs: Agent Skills Tutorial
- Medium: Agent Skills 패턴 모음

---

## 결론

**Agent Skills는 단순한 기능이 아니라 패러다임의 전환입니다.**

```
과거: "AI야, 이렇게 해줘" (매번 설명)
     ↓
현재: "AI야, 배포해줘" (스킬이 모든 것을 앎)
     ↓
미래: AI가 스스로 필요한 스킬을 찾아 실행
```

**핵심 가치**:
1. ✅ **재사용성**: 한 번 작성, 영구 사용
2. ✅ **일관성**: 팀 전체가 동일한 기준
3. ✅ **확장성**: 계속 추가하며 진화
4. ✅ **공유성**: 지식의 코드화와 전파

**지금 시작하세요**:
```bash
mkdir -p .agent/skills/my-skill
echo "---
name: my-skill
description: My first Agent Skill
---

# My Skill
Instructions here..." > .agent/skills/my-skill/SKILL.md
```

**게임 캐릭터에게 스킬북을 장착하듯,**  
**AI 에이전트에게 전문가의 노하우를 이식하세요.** 🚀

---

**작성일**: 2026-01-16  

