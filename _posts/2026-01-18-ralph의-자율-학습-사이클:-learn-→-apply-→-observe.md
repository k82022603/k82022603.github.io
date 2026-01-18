---
title: "Ralph의 자율 학습 사이클: Learn → Apply → Observe"
date: 2026-01-18 10:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,   ai-agent,  ralph-playbook,  software-engineering,  Claude.write]
---


## 목차
1. [개요](#개요)
2. [Ralph의 Learn-Apply-Observe 사이클](#ralph의-learn-apply-observe-사이클)
3. [사이클의 상세 분석](#사이클의-상세-분석)
4. [메타 학습: 반복 간 학습](#메타-학습-반복-간-학습)
5. [자율성의 핵심 메커니즘](#자율성의-핵심-메커니즘)
6. [인간 vs Ralph의 학습 방식 비교](#인간-vs-ralph의-학습-방식-비교)
7. [실전 예시: 실제 사이클 동작](#실전-예시-실제-사이클-동작)
8. [자율 학습의 한계와 극복 방법](#자율-학습의-한계와-극복-방법)
9. [결론](#결론)

---

## 개요

**핵심 질문**: AI Agent가 learn → apply → observe 사이클을 스스로 돈다?

**답**: 네, 맞습니다! Ralph는 정확히 **learn → apply → observe 사이클을 자율적으로 반복**합니다. 

하지만 이것이 작동하는 방식에는 중요한 뉘앙스가 있습니다. Ralph의 학습은 전통적인 머신러닝 학습(가중치 업데이트)이 아니라, **파일 시스템과 Git을 통한 외부 메모리 기반 학습**입니다.

---

## Ralph의 Learn-Apply-Observe 사이클

### 단일 반복(Iteration) 내부의 사이클

```
┌─────────────────────────────────────────────────────────┐
│               하나의 Ralph 반복                          │
│                                                         │
│  1. LEARN (파일에서 학습)                                │
│     ├─ specs/ 읽기 (무엇을 만들어야 하는가?)             │
│     ├─ IMPLEMENTATION_PLAN.md 읽기 (다음 할 일)         │
│     ├─ 기존 코드 패턴 파악 (어떻게 해야 하는가?)         │
│     ├─ AGENTS.md 읽기 (어떤 도구를 사용하는가?)         │
│     └─ progress.md 읽기 (이전에 무엇을 했는가?)         │
│                                                         │
│  2. APPLY (적용)                                        │
│     ├─ 계획에서 최우선 작업 선택                         │
│     ├─ 기존 패턴을 따라 코드 작성/수정                   │
│     ├─ 새로운 파일 생성 또는 기존 파일 수정              │
│     └─ 구현 완료                                        │
│                                                         │
│  3. OBSERVE (관찰 & 피드백)                             │
│     ├─ 테스트 실행                                      │
│     ├─ 빌드 체크                                        │
│     ├─ 린트/타입 체크                                   │
│     └─ 결과 분석                                        │
│                                                         │
│     실패? ──┐                                           │
│       │     │                                           │
│      No    Yes                                         │
│       │     │                                           │
│       │     └──→ 4. RE-LEARN (에러에서 학습)            │
│       │              ├─ 에러 로그 분석                   │
│       │              ├─ 실패 원인 파악                   │
│       │              ├─ 수정 전략 수립                   │
│       │              └─ 2번(APPLY)으로 돌아감            │
│       │                                                 │
│       ▼                                                 │
│  5. COMMIT & RECORD (학습 내용 기록)                    │
│     ├─ Git 커밋 (변경 사항 영구 저장)                    │
│     ├─ progress.md 업데이트 (무엇을 배웠는가)            │
│     ├─ AGENTS.md 업데이트 (새로운 패턴 발견)            │
│     └─ 컨텍스트 클리어 (다음 반복 준비)                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
              │
              ▼
        다음 반복 시작 (완전히 새로운 컨텍스트)
```

### 여러 반복에 걸친 메타 사이클

```
반복 1              반복 2              반복 3              반복 N
┌─────────┐        ┌─────────┐        ┌─────────┐        ┌─────────┐
│ Learn   │        │ Learn   │        │ Learn   │        │ Learn   │
│ (파일)  │        │ (파일+  │        │ (파일+  │        │ (파일+  │
│         │        │  Git)   │        │  Git+   │        │  Git+   │
│         │        │         │        │ 학습)   │        │ 학습)   │
└────┬────┘        └────┬────┘        └────┬────┘        └────┬────┘
     │                  │                  │                  │
     ▼                  ▼                  ▼                  ▼
┌─────────┐        ┌─────────┐        ┌─────────┐        ┌─────────┐
│ Apply   │        │ Apply   │        │ Apply   │        │ Apply   │
│ (코드   │        │ (개선된 │        │ (더 나은│        │ (최적화 │
│  작성)  │        │  패턴)  │        │  방법)  │        │  완료)  │
└────┬────┘        └────┬────┘        └────┬────┘        └────┬────┘
     │                  │                  │                  │
     ▼                  ▼                  ▼                  ▼
┌─────────┐        ┌─────────┐        ┌─────────┐        ┌─────────┐
│Observe  │        │Observe  │        │Observe  │        │Observe  │
│실패 3번 │        │실패 1번 │        │1번에    │        │모든     │
│→ 학습   │        │→ 학습   │        │성공     │        │테스트   │
│         │        │         │        │         │        │통과 ✓   │
└────┬────┘        └────┬────┘        └────┬────┘        └────┬────┘
     │                  │                  │                  │
     ▼                  ▼                  ▼                  ▼
  Git 커밋           Git 커밋           Git 커밋           Git 커밋
  progress           progress           progress           완료!
  업데이트           업데이트           업데이트           
```

---

## 사이클의 상세 분석

### 1. LEARN (학습 단계)

Ralph가 매 반복에서 학습하는 소스:

#### 명시적 지식 (Explicit Knowledge)

**specs/ (요구사항)**
```markdown
# specs/authentication.md

## Success Criteria
- [ ] Users can log in with email/password
- [ ] Sessions persist for 30 days
- [ ] Password reset works
```
- **학습 내용**: "무엇을" 만들어야 하는가
- **용도**: 목표 이해

**IMPLEMENTATION_PLAN.md (작업 계획)**
```markdown
## Next Tasks (Priority Order)

1. [ ] Create user login endpoint
   - POST /api/auth/login
   - Validate credentials
   - Generate JWT token

2. [ ] Implement session management
   - Store sessions in Redis
   - 30-day expiration
```
- **학습 내용**: "다음에" 무엇을 할 것인가
- **용도**: 작업 선택

**AGENTS.md (도구 및 규칙)**
```markdown
# Testing
Run tests: `npm test`
All tests must pass before committing.

# Code Patterns
- Use bcrypt for password hashing (cost: 10)
- JWT tokens in Authorization header
- Error responses follow RFC 7807 format
```
- **학습 내용**: "어떻게" 해야 하는가, "어떤 도구"를 사용하는가
- **용도**: 구현 가이드

**progress.md (진행 상황)**
```markdown
## Completed
- [x] Set up PostgreSQL
- [x] Created User model

## Learnings
- Database connection pooling set to max 20
- User emails must be unique (unique constraint)

## Next
- Implement login endpoint
```
- **학습 내용**: "이미" 무엇을 했는가, "무엇을" 배웠는가
- **용도**: 중복 작업 방지, 컨텍스트 이해

#### 암묵적 지식 (Implicit Knowledge)

**기존 코드 패턴 (Code Patterns)**
```typescript
// src/routes/register.ts (기존 코드)
import bcrypt from 'bcrypt';

export async function registerUser(email: string, password: string) {
  const hashedPassword = await bcrypt.hash(password, 10);
  // ...
}
```
- **학습 내용**: 프로젝트의 코딩 스타일, 아키텍처 패턴
- **용도**: 일관된 코드 작성

**Git 히스토리 (Git History)**
```bash
commit abc123: "Add user registration with bcrypt hashing"
commit def456: "Fix: Increase bcrypt cost to 10 for security"
```
- **학습 내용**: 이전 결정의 맥락, 진화 과정
- **용도**: 왜 이렇게 했는지 이해

### 2. APPLY (적용 단계)

학습한 내용을 바탕으로 코드를 작성:

```typescript
// Ralph가 학습 기반으로 작성한 코드
// src/routes/login.ts

import bcrypt from 'bcrypt'; // AGENTS.md에서 학습
import jwt from 'jsonwebtoken';
import { findUserByEmail } from '../models/user'; // 기존 패턴 참고

export async function loginUser(email: string, password: string) {
  // 기존 register.ts 패턴 따름
  const user = await findUserByEmail(email);
  if (!user) {
    // RFC 7807 에러 형식 (AGENTS.md에서 학습)
    throw new ProblemDetail({
      type: 'authentication-failed',
      title: 'Invalid credentials',
      status: 401
    });
  }

  // bcrypt 사용 (AGENTS.md + 기존 코드에서 학습)
  const isValid = await bcrypt.compare(password, user.hashedPassword);
  if (!isValid) {
    throw new ProblemDetail({
      type: 'authentication-failed',
      title: 'Invalid credentials',
      status: 401
    });
  }

  // JWT 생성 (specs에서 30일 요구사항 학습)
  const token = jwt.sign(
    { userId: user.id },
    process.env.JWT_SECRET,
    { expiresIn: '30d' } // specs/authentication.md 요구사항 반영
  );

  return { token };
}
```

**학습이 적용된 지점**:
1. bcrypt 사용 (AGENTS.md)
2. RFC 7807 에러 형식 (AGENTS.md)
3. 기존 코드 구조 따름 (코드 패턴)
4. 30일 세션 (specs)

### 3. OBSERVE (관찰 단계)

#### 자동화된 피드백 (Backpressure)

**테스트 실행**
```bash
$ npm test

FAIL src/routes/login.test.ts
  ✕ should return token for valid credentials
    Expected 200, received 500
    Error: JWT_SECRET is not defined
```

**Ralph의 관찰**:
- ❌ 테스트 실패
- 🔍 에러: JWT_SECRET 환경변수 누락
- 💡 학습: 환경변수 설정 필요

#### 피드백 루프

```
관찰 → 분석 → 재학습 → 재적용
   ↑                      ↓
   └──────────────────────┘
```

**재학습 (Re-Learn)**:
```
에러 로그 분석:
- JWT_SECRET이 정의되지 않음
- 환경변수가 필요함

기존 코드 검색:
- 다른 파일들은 어떻게 했나?
- .env.example 파일 발견
```

**재적용 (Re-Apply)**:
```typescript
// 수정된 코드
if (!process.env.JWT_SECRET) {
  throw new Error('JWT_SECRET environment variable is required');
}

const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '30d' }
);
```

**재관찰 (Re-Observe)**:
```bash
$ npm test

PASS src/routes/login.test.ts
  ✓ should return token for valid credentials
  ✓ should reject invalid password
  ✓ should reject non-existent user

Test Suites: 1 passed, 1 total
Tests: 3 passed, 3 total
```

### 4. COMMIT & RECORD (기록 단계)

**Git 커밋**:
```bash
git add src/routes/login.ts
git commit -m "feat: implement user login with JWT authentication

- Add POST /api/auth/login endpoint
- Validate credentials with bcrypt
- Generate JWT tokens with 30-day expiration
- Add error handling for invalid credentials
- Tests: all authentication tests passing"
```

**progress.md 업데이트**:
```markdown
## Completed
- [x] Set up PostgreSQL
- [x] Created User model
- [x] Implemented user login endpoint ← 새로 추가

## Learnings
- Database connection pooling set to max 20
- User emails must be unique (unique constraint)
- JWT_SECRET must be set in environment variables ← 새로운 학습
- Using 30-day token expiration as per specs

## Next
- Implement password reset functionality
- Add rate limiting to auth endpoints
```

**AGENTS.md 업데이트** (선택적):
```markdown
# Environment Variables Required
- JWT_SECRET: Secret key for JWT signing (required for auth)
- DATABASE_URL: PostgreSQL connection string
- REDIS_URL: Redis connection for sessions

# Authentication Patterns
- JWT tokens expire in 30 days (specs requirement)
- Always validate JWT_SECRET exists before signing
- Use bcrypt.compare() for password verification
```

---

## 메타 학습: 반복 간 학습

Ralph의 강력함은 **반복마다 조금씩 더 똑똑해진다**는 점입니다.

### 첫 번째 반복: 시행착오

```
LEARN:    specs만 읽음, 패턴 모름
APPLY:    초기 구현 시도
OBSERVE:  테스트 3개 실패
          ├─ JWT_SECRET 누락
          ├─ 에러 형식 잘못됨
          └─ 비밀번호 검증 로직 오류
COMMIT:   없음 (테스트 실패)
```

### 두 번째 반복: 개선

```
LEARN:    specs + 첫 번째 실패에서 학습
          + AGENTS.md 참조
APPLY:    개선된 구현
OBSERVE:  테스트 1개 실패
          └─ 에러 메시지 형식 개선 필요
COMMIT:   없음 (아직 실패)
```

### 세 번째 반복: 성공

```
LEARN:    specs + 이전 모든 실패 + Git 히스토리
          + 기존 코드 패턴 완전 이해
APPLY:    모든 요구사항 충족하는 코드
OBSERVE:  모든 테스트 통과 ✓
COMMIT:   성공! Git에 기록, progress 업데이트
```

### 네 번째+ 반복: 축적된 지식 활용

```
LEARN:    specs + 전체 Git 히스토리
          + progress.md의 모든 학습 내용
          + 확립된 패턴들
APPLY:    첫 시도에 거의 완벽한 코드
OBSERVE:  1번에 통과 (또는 미세 조정만 필요)
COMMIT:   빠른 완료
```

---

## 자율성의 핵심 메커니즘

### 1. 상태 없는 반복 (Stateless Iterations)

**각 반복은 완전히 새로운 시작**:
```
반복 N의 대화 내역 → 삭제
반복 N의 시행착오 → 삭제
반복 N의 중간 생각 → 삭제

유지되는 것:
✓ Git 커밋 (성공한 코드만)
✓ 파일 변경사항
✓ progress.md (핵심 학습 내용)
✓ AGENTS.md 업데이트 (새로운 패턴)
```

**이점**:
- 컨텍스트 오염 방지
- 항상 "맑은 정신"으로 시작
- 실패한 시도가 다음 시도를 방해하지 않음

### 2. 압력 기반 학습 (Backpressure-Driven Learning)

**전통적 방식** (작동 안 함):
```
프롬프트: "버그 없는 코드를 작성해주세요"
AI: "네, 알겠습니다!" 
결과: 여전히 버그 있음
```

**Ralph 방식** (작동함):
```
시스템: 테스트 실행 → 실패 → 앞으로 못 감
AI: 에러 분석 → 학습 → 수정
시스템: 테스트 실행 → 성공 → 진행 허용
```

**압력의 계층**:

1. **즉각적 압력**:
   - 컴파일 에러: 코드가 실행조차 안 됨
   - 타입 에러: 타입 불일치

2. **중간 압력**:
   - 유닛 테스트 실패
   - 린트 경고
   - 통합 테스트 실패

3. **최종 압력**:
   - E2E 테스트
   - 성능 벤치마크
   - LLM-as-Judge (주관적 품질)

### 3. 발견 기반 학습 (Discovery-Based Learning)

Ralph는 명시적으로 가르쳐주지 않은 것도 **코드를 탐색하며 학습**합니다.

**예시: Phase 0c (탐색 단계)**

```
Ralph: "src/lib 디렉토리를 탐색해보자"

발견 1: src/lib/validation.ts
→ 학습: "이 프로젝트는 validator.js를 사용하는구나"
→ 적용: 새 코드에서 같은 라이브러리 사용

발견 2: src/lib/llm-review.test.ts
→ 학습: "주관적 품질은 LLM으로 판단하는구나"
→ 적용: 새 기능에도 같은 패턴 적용

발견 3: src/lib/db.ts
→ 학습: "connection pool 설정이 여기 있구나"
→ 적용: 새 DB 쿼리 시 같은 pool 사용
```

**AGENTS.md 업데이트 불필요**:
- 코드 예시가 문서 역할
- Ralph가 탐색하며 자연스럽게 학습
- "Documentation as Code"

### 4. 경험 축적 (Experience Accumulation)

**Git = 장기 기억 (Long-term Memory)**

```bash
$ git log --oneline
abc123 feat: add user login
def456 fix: handle JWT_SECRET validation
ghi789 refactor: extract auth middleware
jkl012 feat: add password reset
mno345 fix: rate limiting on auth endpoints
```

Ralph는 매 반복마다 전체 Git 히스토리를 볼 수 있습니다:
- 어떤 패턴이 성공했는가
- 어떤 결정이 나중에 수정되었는가 (왜?)
- 프로젝트가 어떻게 진화했는가

**progress.md = 단기 기억 (Short-term Memory)**

현재 스프린트/작업에 대한 메모:
```markdown
## Critical Learnings
- Auth endpoints MUST have rate limiting (learned from security audit)
- JWT refresh tokens need separate table (learned from session bug)
- Always validate email format BEFORE database query (performance)
```

---

## 인간 vs Ralph의 학습 방식 비교

| 측면 | 인간 개발자 | Ralph |
|------|------------|-------|
| **학습 소스** | 경험, 문서, 동료 | 파일, Git, 테스트 피드백 |
| **메모리** | 뇌 (휘발성 + 장기) | 파일 + Git (영구) |
| **학습 속도** | 시간에 걸쳐 점진적 | 매 반복마다 즉각적 |
| **컨텍스트 유지** | 하루 종일 유지 | 매 작업마다 리셋 |
| **피드백 루프** | 수동 (테스트 실행) | 자동 (강제) |
| **일관성** | 피로, 주의력 변동 | 항상 동일 (프롬프트 기반) |
| **창의성** | 높음 | 제한적 (패턴 내) |
| **규모** | 1명 = 1 작업 | 1 루프 = N 작업 |

### 시너지: 인간 + Ralph

```
인간의 역할:
├─ 아키텍처 설계
├─ 비즈니스 로직 정의
├─ 명세서 작성 (specs/)
├─ 압력 메커니즘 설계 (테스트, 린트)
└─ 최종 검토 및 승인

Ralph의 역할:
├─ 패턴 따르기
├─ 반복 작업 수행
├─ 자가 검증
├─ 문서화
└─ 24/7 작업 가능
```

---

## 실전 예시: 실제 사이클 동작

### 시나리오: REST API 인증 구현

**초기 설정**:

```markdown
# specs/authentication.md

## JTBD
As a user, I need to securely authenticate to access protected resources.

## Success Criteria
- [ ] POST /api/auth/register creates new users
- [ ] POST /api/auth/login returns JWT token
- [ ] Protected routes verify JWT
- [ ] Invalid tokens return 401
- [ ] All auth endpoints have rate limiting
- [ ] Passwords are hashed with bcrypt
```

```markdown
# AGENTS.md

# Testing
Run tests: `npm test`
Run specific test: `npm test -- auth`

# Code Quality
Lint: `npm run lint`
Type check: `npm run type-check`

# Dependencies
- bcrypt for password hashing
- jsonwebtoken for JWT
- express-rate-limit for rate limiting
```

### 반복 1: 등록 엔드포인트

**LEARN**:
```
- specs/authentication.md 읽음
- AGENTS.md 도구 확인
- 기존 코드 없음 (첫 번째 기능)
```

**APPLY**:
```typescript
// src/routes/auth.ts (첫 시도)
import express from 'express';
import bcrypt from 'bcrypt';

const router = express.Router();

router.post('/register', async (req, res) => {
  const { email, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  // TODO: Save to database
  res.json({ success: true });
});

export default router;
```

**OBSERVE**:
```bash
$ npm test

FAIL src/routes/auth.test.ts
  ✕ should reject weak passwords
  ✕ should validate email format
  ✕ should return 409 for duplicate email

3 failed, 0 passed
```

**RE-LEARN** (실패에서 학습):
```
에러 분석:
1. 비밀번호 강도 검증 없음
2. 이메일 형식 검증 없음
3. 중복 이메일 체크 없음
4. 데이터베이스 연결 없음
```

**RE-APPLY**:
```typescript
// src/routes/auth.ts (개선)
import express from 'express';
import bcrypt from 'bcrypt';
import { z } from 'zod';
import { db } from '../lib/db';

const router = express.Router();

const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/),
});

router.post('/register', async (req, res) => {
  try {
    const { email, password } = registerSchema.parse(req.body);
    
    // Check duplicate
    const existing = await db.user.findUnique({ where: { email } });
    if (existing) {
      return res.status(409).json({ error: 'Email already exists' });
    }

    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await db.user.create({
      data: { email, hashedPassword }
    });

    res.status(201).json({ id: user.id, email: user.email });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors });
    }
    throw error;
  }
});

export default router;
```

**RE-OBSERVE**:
```bash
$ npm test

PASS src/routes/auth.test.ts
  ✓ should create user with valid data
  ✓ should reject weak passwords
  ✓ should validate email format
  ✓ should return 409 for duplicate email

4 passed, 0 failed
```

**COMMIT**:
```bash
git add src/routes/auth.ts
git commit -m "feat: implement user registration

- Add POST /api/auth/register endpoint
- Validate email format and password strength
- Check for duplicate emails (409 conflict)
- Hash passwords with bcrypt (cost: 10)
- Tests: all registration tests passing"
```

**progress.md 업데이트**:
```markdown
## Completed
- [x] POST /api/auth/register creates new users

## Learnings
- Using Zod for request validation
- Password must: min 8 chars, 1 uppercase, 1 number
- bcrypt cost factor: 10 (from AGENTS.md)
- Return 409 for duplicate emails (REST standard)

## Next
- Implement login endpoint
- Add JWT token generation
```

### 반복 2: 로그인 엔드포인트

**LEARN**:
```
- specs/authentication.md (JWT 요구사항)
- AGENTS.md (JWT 도구)
- progress.md (이전 학습 내용)
- 기존 코드 (register.ts의 패턴)
  → Zod 사용
  → 에러 처리 패턴
  → 응답 형식
```

**APPLY** (첫 시도부터 더 나음):
```typescript
// src/routes/auth.ts (로그인 추가)

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string(),
});

router.post('/login', async (req, res) => {
  try {
    const { email, password } = loginSchema.parse(req.body);
    
    const user = await db.user.findUnique({ where: { email } });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const isValid = await bcrypt.compare(password, user.hashedPassword);
    if (!isValid) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const token = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET!,
      { expiresIn: '30d' } // specs 요구사항
    );

    res.json({ token });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors });
    }
    throw error;
  }
});
```

**OBSERVE**:
```bash
$ npm test

PASS src/routes/auth.test.ts
  ✓ should return token for valid credentials
  ✓ should reject invalid password
  ✓ should reject non-existent user

7 passed, 0 failed
```

**학습의 진화 확인**:
- 첫 시도에 거의 완벽
- 이전 패턴(Zod, 에러 처리) 자동 적용
- specs 요구사항(30일) 정확히 반영

### 반복 3: Rate Limiting 추가

**LEARN**:
```
- specs: "rate limiting" 요구
- AGENTS.md: express-rate-limit 사용
- progress.md: 완료된 엔드포인트 2개
- Git 히스토리: 기존 미들웨어 패턴
```

**APPLY**:
```typescript
// src/middleware/rate-limit.ts (새 파일)
import rateLimit from 'express-rate-limit';

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 requests per window
  message: 'Too many authentication attempts, please try again later',
});

// src/routes/auth.ts (업데이트)
import { authLimiter } from '../middleware/rate-limit';

router.post('/register', authLimiter, async (req, res) => { ... });
router.post('/login', authLimiter, async (req, res) => { ... });
```

**OBSERVE**:
```bash
$ npm test

PASS src/routes/auth.test.ts
  ✓ should rate limit after 5 attempts

8 passed, 0 failed
```

**COMMIT & 최종 업데이트**:
```markdown
# progress.md

## Completed
- [x] POST /api/auth/register creates new users
- [x] POST /api/auth/login returns JWT token
- [x] All auth endpoints have rate limiting

## Learnings
- Zod for request validation (consistent pattern)
- bcrypt cost: 10
- JWT expiration: 30 days (specs requirement)
- Rate limiting: 5 attempts per 15 minutes
- Always return same error message for invalid login (security)

## Architecture Decisions
- Separated rate limiting into middleware
- JWT_SECRET from environment variables
- User-facing errors use generic messages (security)

## Next
- Implement protected route middleware
- Add JWT verification
```

---

## 자율 학습의 한계와 극복 방법

### 한계 1: 창의성 부족

**문제**: Ralph는 패턴을 따르지만, 새로운 패턴을 발명하지는 못합니다.

**극복 방법**:
- **인간의 역할**: 초기 패턴 설정
- **예시 코드**: 한 번 보여주면 따라 함
- **AGENTS.md**: 명시적 가이드 제공

### 한계 2: 컨텍스트 제한

**문제**: 매 반복마다 컨텍스트 리셋으로 대규모 코드베이스 이해 어려움

**극복 방법**:
- **progress.md**: 핵심 정보 요약
- **AGENTS.md**: 중요 패턴 문서화
- **작업 분해**: 큰 작업을 작은 단위로

### 한계 3: 주관적 판단

**문제**: "좋은 UX"같은 주관적 기준 판단 어려움

**극복 방법**:
- **LLM-as-Judge**: 주관적 품질도 테스트로
- **명확한 기준**: specs에 구체적 기준 명시
- **인간 검토**: 최종 승인은 인간이

### 한계 4: 아키텍처 결정

**문제**: 마이크로서비스 vs 모놀리스 같은 전략적 결정 불가

**극복 방법**:
- **인간이 결정**: 아키텍처는 인간의 영역
- **Ralph는 실행**: 결정된 아키텍처 내에서 구현
- **명확한 경계**: 무엇을 자동화할지/말지 구분

---

## 결론

### Ralph의 자율 학습 사이클: 핵심 요약

Ralph는 **learn → apply → observe 사이클을 완전히 자율적으로 수행**합니다:

1. **LEARN**: 파일 시스템(specs, AGENTS.md, 코드, Git)에서 학습
2. **APPLY**: 학습한 패턴을 새 코드에 적용
3. **OBSERVE**: 자동화된 테스트로 결과 검증
4. **반복**: 성공할 때까지 계속, 성공 시 커밋하고 다음 작업

### 왜 작동하는가?

**전통적 AI 학습**:
```
데이터 → 모델 훈련 → 가중치 업데이트 → 추론
(느림, 비쌈, 데이터 필요)
```

**Ralph의 학습**:
```
파일 읽기 → 패턴 인식 → 적용 → 피드백 → 수정
(빠름, 저렴, 즉각적)
```

### 진짜 마법은...

Ralph의 진짜 힘은 개별 반복이 아니라 **수십, 수백 번의 반복을 통한 점진적 개선**입니다:

```
반복 1: 완전히 틀림 → 3번 실패
반복 2: 거의 맞음 → 1번 실패
반복 3: 성공 → 커밋
반복 4: 학습 적용 → 1번에 성공
반복 5: 더 빠름
...
반복 100: 거의 완벽
```

### 인간 + Ralph = 최강

```
┌─────────────────────────────────────────┐
│         인간 (루프 위에서)               │
│  - 아키텍처 설계                         │
│  - 비즈니스 로직 정의                    │
│  - 압력 메커니즘 설계                    │
│  - 최종 검토                            │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│         Ralph (루프 안에서)              │
│  Learn → Apply → Observe → Repeat       │
│  - 24/7 작업                            │
│  - 패턴 따르기                          │
│  - 자가 검증                            │
│  - 무한 반복 가능                       │
└─────────────────────────────────────────┘
```

### 미래: 진화하는 학습

Geoffrey Huntley의 비전인 **"Loom"** (진화적 소프트웨어):
- Ralph가 단순히 사람의 명세를 따르는 것을 넘어
- 스스로 최적화하고
- 수익을 극대화하고
- 제품을 진화시키는

하지만 그때도 **인간 엔지니어는 여전히 중요**합니다:
- 시스템 설계
- 압력 메커니즘 구축
- 윤리적 가이드라인 설정
- 최종 책임

**"Let Ralph Ralph, but keep engineers on the loop."**

---

## 참고 자료

- [The Ralph Playbook - Clayton Farr](https://claytonfarr.github.io/ralph-playbook/)
- [The Ralph Wiggum Playbook - paddo.dev](https://paddo.dev/blog/ralph-wiggum-playbook/)
- [Inventing the Ralph Wiggum Loop - Dev Interrupted](https://linearb.io/dev-interrupted/podcast/inventing-the-ralph-wiggum-loop)
- [everything is a ralph loop - Geoffrey Huntley](https://ghuntley.com/loop/)

---

**작성 일자**: 2026-01-18
