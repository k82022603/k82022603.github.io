---
title: "개발자를 위한 Claude Code & Skills 완벽 가이드"
date: 2026-01-07 23:05:00 +0900
categories: [AI,  Claude Skills]
mermaid: [True]
tags: [AI,  claude-code,  claude-skills,  agent-skills,  Claude.write]
---

## Claude Code & Skills 실전 활용 매뉴얼

---
## 목차

1. [환경 설정](#1-환경-설정)
2. [CLAUDE.md 작성 가이드](#2-claudemd-작성-가이드)
3. [Skills 개발](#3-skills-개발)
4. [슬래시 명령어 구축](#4-슬래시-명령어-구축)
5. [서브에이전트 설계](#5-서브에이전트-설계)
6. [워크플로우 자동화](#6-워크플로우-자동화)
7. [병렬 실행 전략](#7-병렬-실행-전략)
8. [검증 및 테스팅](#8-검증-및-테스팅)
9. [팀 협업 전략](#9-팀-협업-전략)
10. [프로젝트별 설정](#10-프로젝트별-설정)

---

## 1. 환경 설정

### Claude Code 설치

```bash
# npm을 통한 설치
npm install -g @anthropic-ai/claude-code

# 또는 Homebrew (macOS)
brew install claude-code

# 설치 확인
claude-code --version
```

### 프로젝트 구조 초기화

```bash
# 프로젝트 루트에서
mkdir -p .claude/{skills,commands,rules,subagents,hooks}
touch .claude/CLAUDE.md

# .gitignore에 추가
echo ".claude/sessions/" >> .gitignore
echo ".claude/cache/" >> .gitignore
```

**표준 디렉토리 구조:**
```
your-project/
├── .claude/
│   ├── CLAUDE.md              # 메인 가이드라인
│   ├── skills/                # 커스텀 Skills
│   │   ├── frontend-design/
│   │   └── api-generator/
│   ├── commands/              # 슬래시 명령어
│   │   ├── commit-push-pr.md
│   │   └── fix-issue.md
│   ├── rules/                 # 조건부 규칙
│   │   ├── api-rules.md
│   │   └── test-rules.md
│   ├── subagents/            # 서브에이전트
│   │   └── security-check.md
│   └── hooks/                # Pre/Post 훅
│       └── post-tool-use.sh
├── src/
└── tests/
```

### iTerm2 알림 설정 (macOS)

```bash
# iTerm2 > Preferences > Profiles > Terminal
# "Notifications" 섹션:
# ✓ "Notify on next mark"
# ✓ "Send notifications when tab title changes"

# Claude Code에서 자동으로 알림 트리거
```

### 권한 사전 승인

```bash
# Claude Code 실행 후
/permissions

# 안전한 명령어 사전 승인:
# - git status
# - git diff
# - npm test
# - bun run typecheck
# - docker ps
# - kubectl get pods
```

---

## 2. CLAUDE.md 작성 가이드

### 기본 템플릿

~~~markdown
# Claude Development Guide

## 프로젝트 개요

**프로젝트명**: MyApp
**기술 스택**: React + TypeScript + Node.js + PostgreSQL
**패키지 매니저**: bun (npm 절대 사용 금지)

## 디렉토리 구조

```
src/
├── api/          # REST API 엔드포인트
├── components/   # React 컴포넌트
├── utils/        # 유틸리티 함수
├── types/        # TypeScript 타입 정의
└── tests/        # 테스트 파일
```

## 개발 워크플로우

### 새 기능 개발
1. `git checkout -b feature/feature-name`
2. 코드 작성
3. `bun run typecheck` (타입 체크)
4. `bun test` (테스트 실행)
5. `git commit -m "feat: description"`
6. `/commit-push-pr` (자동 PR 생성)

### 테스트 명령어
```bash
# 전체 테스트
bun test

# 특정 파일
bun test -- path/to/file.test.ts

# Watch 모드
bun test --watch
```

## 코딩 규칙

### TypeScript
- **필수**: 모든 함수에 타입 힌트
- **금지**: `any` 타입 (unknown 사용)
- **금지**: `enum` (string literal union 사용)
- **선호**: `type` over `interface`

### React
- **필수**: 함수형 컴포넌트만 사용
- **필수**: Props에 타입 정의
- **금지**: Class 컴포넌트
- **선호**: 커스텀 훅으로 로직 분리

### API 설계
- **필수**: 모든 엔드포인트에 입력 검증
- **필수**: 표준 에러 응답 형식
```typescript
{
  error: {
    code: "VALIDATION_ERROR",
    message: "Invalid input",
    details: []
  }
}
```

### 데이터베이스
- **필수**: 모든 마이그레이션은 `migrations/` 폴더에
- **금지**: 프로덕션 DB에 직접 쿼리
- **선호**: Prisma ORM 사용

## 파일 경계

### 편집 가능
- `/src/`
- `/tests/`
- `/docs/`
- `/migrations/`

### 절대 건드리지 말 것
- `/node_modules/`
- `/dist/`
- `/.next/`
- `/coverage/`

## Git 규칙

### 커밋 메시지 형식
```
<type>: <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: 새 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `refactor`: 리팩토링
- `test`: 테스트 추가/수정
- `chore`: 빌드/설정 변경

### 브랜치 네이밍
- `feature/` - 새 기능
- `fix/` - 버그 수정
- `refactor/` - 리팩토링
- `docs/` - 문서

## 자주 발생한 실수

### 1. Enum 사용
❌ **절대 하지 마세요:**
```typescript
enum Status {
  Active = "active",
  Inactive = "inactive"
}
```

✅ **대신 이렇게:**
```typescript
type Status = "active" | "inactive";
```

**이유**: enum은 런타임 코드를 생성하고 타입 안정성이 떨어집니다.

### 2. npm 사용
❌ `npm install package-name`
✅ `bun add package-name`

**이유**: 프로젝트는 bun.lock을 사용하며 npm은 package-lock.json을 생성합니다.

### 3. 직접 console.log 남기기
❌ `console.log("debug info")`
✅ `logger.debug("debug info")` (utils/logger.ts 사용)

**이유**: 프로덕션에 console.log가 남으면 안 됩니다.

## 참조 문서

- API 문서: `/docs/api.md`
- 데이터베이스 스키마: `/docs/schema.md`
- 배포 가이드: `/docs/deployment.md`
~~~

### WHAT, WHY, HOW 프레임워크

**WHAT (무엇을)**: 프로젝트 구조, 기술 스택
**WHY (왜)**: 각 결정의 이유, 프로젝트 목표
**HOW (어떻게)**: 명령어, 워크플로우, 테스트 방법

### 조건부 규칙

**`.claude/rules/api-rules.md`:**
~~~markdown
---
paths: src/api/**/*.ts
---

# API 개발 규칙

이 규칙은 src/api/ 폴더의 TypeScript 파일에만 적용됩니다.

## 엔드포인트 구조
```typescript
// 표준 템플릿
export async function handler(
  req: Request,
  res: Response
): Promise<Response> {
  // 1. 입력 검증
  const input = validateInput(req.body);
  
  // 2. 비즈니스 로직
  const result = await service.process(input);
  
  // 3. 응답
  return res.json({ data: result });
}
```

## 에러 핸들링
- 모든 에러는 ApiError로 래핑
- HTTP 상태 코드를 명확히 설정
- 클라이언트에게 유용한 메시지 제공

## 인증/인가
- 모든 보호된 엔드포인트에 `@authenticated` 데코레이터
- 권한 체크는 미들웨어에서
~~~

---

## 3. Skills 개발

### Skill 기본 구조

```
my-skill/
├── skill.md                 # 메인 지침
├── details/
│   └── advanced.md         # 상세 가이드
├── examples/
│   └── sample.ts           # 예제 코드
└── scripts/
    └── helper.py           # 유틸리티 스크립트
```

### 프론트엔드 디자인 Skill

**`.claude/skills/frontend-design/skill.md`:**

~~~markdown
---
name: Frontend Design
description: 현대적이고 전문적인 UI/UX를 생성하는 Skill
---

# 프론트엔드 디자인 Skill

## 사용 시점
- 새로운 컴포넌트나 페이지를 만들 때
- UI/UX 개선이 필요할 때
- 랜딩 페이지나 마케팅 페이지 제작 시

## 디자인 원칙

### 색상 시스템
```typescript
const colors = {
  // 절대 사용 금지: 파란-보라 그라데이션, 네온 색상
  primary: {
    dark: '#1A1A1A',
    light: '#F5F5F5',
    accent: '#2D2D2D', // 프로젝트 맥락에 따라 조정
  },
  semantic: {
    success: '#10B981',
    error: '#EF4444',
    warning: '#F59E0B',
    info: '#3B82F6',
  },
  neutral: {
    50: '#F9FAFB',
    100: '#F3F4F6',
    // ... 900까지
  }
};
```

### 타이포그래피
```css
/* 제목 */
font-family: 'Inter', sans-serif;
font-weight: 700;
line-height: 1.2;

/* 본문 */
font-family: 'Inter', sans-serif;
font-weight: 400;
line-height: 1.6;

/* 코드 */
font-family: 'JetBrains Mono', monospace;
```

### 간격 시스템
8px 기반 시스템: 8, 16, 24, 32, 48, 64, 96

### 반응형 브레이크포인트
```typescript
const breakpoints = {
  sm: '640px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px',
};
```

## 컴포넌트 스타일 가이드

### 버튼
```typescript
// Primary Button
<button className="
  px-4 py-2 
  bg-primary-dark text-white 
  rounded-lg 
  hover:bg-primary-dark/90 
  transition-colors duration-200
  font-medium
">
  Action
</button>

// Secondary Button
<button className="
  px-4 py-2 
  border border-neutral-300 
  rounded-lg 
  hover:bg-neutral-50 
  transition-colors duration-200
">
  Cancel
</button>
```

### 입력 필드
```typescript
<input 
  className="
    w-full px-4 py-2
    border border-neutral-300 rounded-lg
    focus:ring-2 focus:ring-primary-accent focus:border-transparent
    transition-all duration-200
  "
  placeholder="Enter text..."
/>
```

### 카드
```typescript
<div className="
  p-6 
  bg-white 
  rounded-xl 
  shadow-sm 
  hover:shadow-md 
  transition-shadow duration-200
">
  Content
</div>
```

## 애니메이션 규칙

### 전환 시간
- **빠름**: 150ms (hover, focus)
- **보통**: 200-300ms (슬라이드, 페이드)
- **느림**: 400ms+ (복잡한 애니메이션)

### Easing
```css
/* 기본 */
transition-timing-function: ease-in-out;

/* 진입 */
transition-timing-function: ease-out;

/* 퇴장 */
transition-timing-function: ease-in;
```

### 과도한 애니메이션 금지
- 페이지 로드 시 너무 많은 애니메이션 지양
- 필수 요소만 강조
- 접근성 고려 (prefers-reduced-motion)

## 접근성 체크리스트

- [ ] 색상 대비비 최소 4.5:1 (WCAG AA)
- [ ] 키보드 네비게이션 지원
- [ ] 적절한 ARIA 레이블
- [ ] 포커스 인디케이터 명확히
- [ ] 대체 텍스트 (이미지)

## 예제

자세한 예제는 `examples/` 폴더 참조:
- 로그인 페이지: `examples/login-page.tsx`
- 대시보드: `examples/dashboard.tsx`
- 랜딩 페이지: `examples/landing.tsx`
~~~

### API 생성기 Skill

**`.claude/skills/api-generator/skill.md`:**

~~~markdown
---
name: API Generator
description: RESTful API 엔드포인트를 표준 형식으로 생성
---

# API 생성기 Skill

## 사용 시점
"새로운 API 엔드포인트를 만들어줘" 같은 요청을 받았을 때

## 생성 프로세스

### 1단계: 요구사항 파악
다음 질문에 답을 얻어야 합니다:
- 엔드포인트 경로? (예: `/api/users`)
- HTTP 메서드? (GET, POST, PUT, DELETE)
- 입력 파라미터?
- 응답 형식?
- 인증 필요 여부?

### 2단계: 파일 생성

**라우터 파일**: `src/api/routes/<resource>.ts`
**컨트롤러**: `src/api/controllers/<resource>.controller.ts`
**서비스**: `src/api/services/<resource>.service.ts`
**타입**: `src/types/<resource>.types.ts`
**테스트**: `tests/api/<resource>.test.ts`

### 3단계: 표준 템플릿 적용

#### 타입 정의
```typescript
// src/types/user.types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserInput {
  email: string;
  name: string;
  password: string;
}

export interface UpdateUserInput {
  name?: string;
  email?: string;
}
```

#### 서비스 레이어
```typescript
// src/api/services/user.service.ts
import { db } from '@/lib/database';
import type { User, CreateUserInput } from '@/types/user.types';

export class UserService {
  async findById(id: string): Promise<User | null> {
    return db.user.findUnique({ where: { id } });
  }

  async create(input: CreateUserInput): Promise<User> {
    // 비즈니스 로직
    const hashedPassword = await hash(input.password);
    return db.user.create({
      data: {
        ...input,
        password: hashedPassword,
      },
    });
  }

  async update(id: string, input: UpdateUserInput): Promise<User> {
    return db.user.update({
      where: { id },
      data: input,
    });
  }

  async delete(id: string): Promise<void> {
    await db.user.delete({ where: { id } });
  }
}
```

#### 컨트롤러
```typescript
// src/api/controllers/user.controller.ts
import { Request, Response } from 'express';
import { UserService } from '@/services/user.service';
import { validateInput } from '@/utils/validation';
import { createUserSchema } from '@/schemas/user.schema';

export class UserController {
  private service = new UserService();

  async create(req: Request, res: Response) {
    try {
      // 1. 입력 검증
      const input = validateInput(createUserSchema, req.body);
      
      // 2. 서비스 호출
      const user = await this.service.create(input);
      
      // 3. 응답 (비밀번호 제외)
      const { password, ...safeUser } = user;
      return res.status(201).json({ data: safeUser });
      
    } catch (error) {
      if (error instanceof ValidationError) {
        return res.status(400).json({
          error: {
            code: 'VALIDATION_ERROR',
            message: error.message,
            details: error.details,
          },
        });
      }
      throw error; // 글로벌 에러 핸들러로
    }
  }

  async findById(req: Request, res: Response) {
    const user = await this.service.findById(req.params.id);
    
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'NOT_FOUND',
          message: 'User not found',
        },
      });
    }
    
    const { password, ...safeUser } = user;
    return res.json({ data: safeUser });
  }
}
```

#### 라우터
```typescript
// src/api/routes/users.ts
import { Router } from 'express';
import { UserController } from '@/controllers/user.controller';
import { authenticated } from '@/middleware/auth';

const router = Router();
const controller = new UserController();

router.post('/', controller.create);
router.get('/:id', authenticated, controller.findById);
router.put('/:id', authenticated, controller.update);
router.delete('/:id', authenticated, controller.delete);

export default router;
```

#### 테스트
```typescript
// tests/api/user.test.ts
import { describe, it, expect, beforeEach } from 'bun:test';
import request from 'supertest';
import { app } from '@/app';
import { db } from '@/lib/database';

describe('User API', () => {
  beforeEach(async () => {
    await db.user.deleteMany();
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const input = {
        email: 'test@example.com',
        name: 'Test User',
        password: 'securePassword123',
      };

      const response = await request(app)
        .post('/api/users')
        .send(input)
        .expect(201);

      expect(response.body.data).toMatchObject({
        email: input.email,
        name: input.name,
      });
      expect(response.body.data.password).toBeUndefined();
    });

    it('should return validation error for invalid email', async () => {
      const input = {
        email: 'invalid-email',
        name: 'Test',
        password: 'pass',
      };

      const response = await request(app)
        .post('/api/users')
        .send(input)
        .expect(400);

      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });
  });
});
```

### 4단계: 문서 생성

API 문서를 자동 업데이트:

```markdown
## POST /api/users

사용자를 생성합니다.

**Request:**
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "password": "securePassword"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe",
    "createdAt": "2026-01-07T..."
  }
}
```

**Errors:**
- 400: Validation error
- 409: Email already exists
```

### 5단계: 통합

메인 라우터에 등록:
```typescript
// src/api/index.ts
import userRoutes from './routes/users';

app.use('/api/users', userRoutes);
```

## 체크리스트

생성 완료 후 확인:
- [ ] 타입 정의 완료
- [ ] 서비스 레이어 구현
- [ ] 컨트롤러 구현
- [ ] 라우터 등록
- [ ] 테스트 작성 (최소 happy path + error case)
- [ ] API 문서 업데이트
- [ ] 입력 검증 스키마 작성
- [ ] 에러 핸들링 구현
~~~

---

## 4. 슬래시 명령어 구축

### Commit-Push-PR 명령어

**`.claude/commands/commit-push-pr.md`:**

~~~markdown
# Commit, Push, and Create PR

작업을 커밋하고 푸시한 후 PR을 자동 생성합니다.

## 실행 단계

### 1. Git 상태 확인
```bash
git status
```

변경된 파일이 있는지 확인합니다.

### 2. 변경사항 스테이징
```bash
git add .
```

### 3. 커밋 메시지 생성

변경 내용을 분석하여 Conventional Commits 형식으로 작성:

```bash
git diff --cached
```

**형식:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**예시:**
```
feat(api): add user authentication endpoint

- Implement JWT token generation
- Add password hashing with bcrypt
- Create login/logout endpoints

Closes #123
```

### 4. 커밋 실행
```bash
git commit -m "<generated message>"
```

### 5. 원격 푸시
```bash
# 현재 브랜치 이름 가져오기
BRANCH=$(git branch --show-current)

# 푸시
git push origin $BRANCH

# 만약 새 브랜치라면
git push --set-upstream origin $BRANCH
```

### 6. Pull Request 생성

GitHub CLI 사용:
```bash
gh pr create \
  --title "<PR title from commit message>" \
  --body "<PR description from commit body>" \
  --base main
```

**PR 설명 템플릿:**
```markdown
## 변경 사항
<commit body>

## 테스트
- [ ] 단위 테스트 통과
- [ ] 타입 체크 통과
- [ ] 린트 통과

## 스크린샷 (UI 변경 시)
<if applicable>

## 관련 이슈
Closes #<issue number>
```

### 7. PR URL 출력
```bash
# PR이 생성되면 URL 출력
echo "✅ PR created: <URL>"
```

## 에러 처리

### 변경사항이 없는 경우
```bash
if [ -z "$(git status --porcelain)" ]; then
  echo "⚠️  No changes to commit"
  exit 0
fi
```

### 충돌이 있는 경우
```bash
if ! git push; then
  echo "❌ Push failed. Resolve conflicts and try again."
  exit 1
fi
```

### GitHub CLI가 없는 경우
```bash
if ! command -v gh &> /dev/null; then
  echo "⚠️  GitHub CLI not installed. PR creation skipped."
  echo "🔗 Create manually: https://github.com/<repo>/compare/$BRANCH"
  exit 0
fi
```
~~~

### GitHub 이슈 수정 명령어

**`.claude/commands/fix-github-issue.md`:**

~~~markdown
# Fix GitHub Issue

GitHub 이슈를 읽고 자동으로 수정합니다.

**Usage:** `/fix-github-issue <issue-number>`

## 실행 단계

### 1. 이슈 정보 가져오기
```bash
ISSUE_NUM=$ARGUMENTS

# GitHub CLI로 이슈 상세 정보 가져오기
gh issue view $ISSUE_NUM --json title,body,labels,assignees
```

### 2. 이슈 분석

이슈 제목과 본문을 읽고 다음을 파악:
- 무엇이 문제인가?
- 어떤 파일이 관련되어 있는가?
- 예상 원인은?
- 재현 단계는?

### 3. 코드베이스 검색

관련 파일 찾기:
```bash
# 키워드로 파일 검색
rg "<keyword from issue>" --type typescript

# 또는 특정 함수/클래스 검색
rg "function <functionName>" --type typescript
```

### 4. 문제 재현 (가능한 경우)

이슈에 재현 단계가 있다면:
1. 해당 테스트 케이스 작성
2. 실패하는 것 확인
3. 수정 후 통과하는지 검증

### 5. 수정 구현

이슈 설명에 기반하여 최소한의 변경으로 문제 해결:
- 근본 원인 해결
- 엣지 케이스 고려
- 기존 기능 영향 최소화

### 6. 테스트 작성/업데이트

```typescript
describe('Issue #${ISSUE_NUM}: <issue title>', () => {
  it('should <expected behavior>', () => {
    // 테스트 코드
  });
});
```

### 7. 검증

```bash
# 타입 체크
bun run typecheck

# 테스트 실행
bun test

# 린트
bun run lint
```

### 8. 커밋 및 PR

```bash
# 커밋 메시지에 이슈 번호 포함
git commit -m "fix: resolve issue #${ISSUE_NUM}

<commit body>

Closes #${ISSUE_NUM}"

# PR 생성
gh pr create \
  --title "Fix: <issue title>" \
  --body "Fixes #${ISSUE_NUM}" \
  --base main
```

### 9. 이슈에 코멘트

```bash
gh issue comment $ISSUE_NUM \
  --body "Fixed in PR #<PR number>. The issue was caused by <explanation>."
```

## Plan Mode 사용 권장

복잡한 이슈의 경우 먼저 Plan Mode(Shift+Tab x2)로:
1. 문제 분석
2. 해결 전략 수립
3. 영향 범위 파악
4. 승인 후 실행
~~~

---

## 5. 서브에이전트 설계

### 코드 간소화 에이전트

**`.claude/subagents/code-simplifier.md`:**

~~~markdown
# Code Simplifier Subagent

코드를 더 간결하고 읽기 쉽게 리팩토링합니다.

## 목적
편집 후 코드의 복잡도를 줄이고 가독성을 향상시킵니다.

## 호출 시점
- 기능 구현 완료 후
- 코드 리뷰 전
- PR 생성 전

## 간소화 원칙

### 1. 중복 제거 (DRY)

**Before:**
```typescript
function calculateTotal(items: Item[]) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  return total;
}

function calculateTax(items: Item[]) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  return total * 0.1;
}
```

**After:**
```typescript
function calculateSubtotal(items: Item[]) {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function calculateTotal(items: Item[]) {
  return calculateSubtotal(items);
}

function calculateTax(items: Item[]) {
  return calculateSubtotal(items) * 0.1;
}
```

### 2. 복잡한 조건문 단순화

**Before:**
```typescript
if (user.isActive && user.emailVerified && (user.role === 'admin' || user.role === 'moderator') && !user.isBanned) {
  // ...
}
```

**After:**
```typescript
function canAccessAdminPanel(user: User) {
  return (
    user.isActive &&
    user.emailVerified &&
    ['admin', 'moderator'].includes(user.role) &&
    !user.isBanned
  );
}

if (canAccessAdminPanel(user)) {
  // ...
}
```

### 3. Early Return 패턴

**Before:**
```typescript
function processOrder(order: Order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.isPaid) {
        // 실제 로직
        return result;
      } else {
        throw new Error('Order not paid');
      }
    } else {
      throw new Error('No items');
    }
  } else {
    throw new Error('Order not found');
  }
}
```

**After:**
```typescript
function processOrder(order: Order) {
  if (!order) throw new Error('Order not found');
  if (order.items.length === 0) throw new Error('No items');
  if (!order.isPaid) throw new Error('Order not paid');
  
  // 실제 로직
  return result;
}
```

### 4. 매직 넘버/스트링 제거

**Before:**
```typescript
if (user.status === 1) {
  // ...
} else if (user.status === 2) {
  // ...
}
```

**After:**
```typescript
const UserStatus = {
  ACTIVE: 1,
  INACTIVE: 2,
  SUSPENDED: 3,
} as const;

if (user.status === UserStatus.ACTIVE) {
  // ...
} else if (user.status === UserStatus.INACTIVE) {
  // ...
}
```

### 5. 함수 분해

**Before:**
```typescript
async function createUser(data: CreateUserInput) {
  // 검증 (20 lines)
  // 비밀번호 해싱 (5 lines)
  // DB 저장 (10 lines)
  // 이메일 발송 (15 lines)
  // 로깅 (5 lines)
  // 55 lines total
}
```

**After:**
```typescript
async function createUser(data: CreateUserInput) {
  const validatedData = validateUserInput(data);
  const hashedPassword = await hashPassword(validatedData.password);
  const user = await saveUserToDatabase({ ...validatedData, password: hashedPassword });
  await sendWelcomeEmail(user);
  logUserCreation(user);
  return user;
}
// 각 함수는 5-10 lines
```

## 복잡도 메트릭

다음 기준으로 복잡도 평가:
- **함수 길이**: 50줄 이하 권장
- **중첩 깊이**: 3단계 이하
- **매개변수 수**: 3개 이하
- **순환 복잡도**: 10 이하

## 실행 프로세스

1. 변경된 파일 스캔
2. 복잡도 높은 함수 식별
3. 리팩토링 제안
4. 테스트 통과 확인
5. 커밋

## 주의사항

- 기능 변경 금지 (순수 리팩토링만)
- 테스트 깨뜨리지 않기
- 성능 저하 방지
~~~

### 보안 체크 에이전트

**`.claude/subagents/security-checker.md`:**

~~~markdown
# Security Checker Subagent

코드의 보안 취약점을 스캔합니다.

## 검사 항목

### 1. 하드코딩된 비밀

```typescript
// ❌ 절대 하지 마세요
const API_KEY = "sk_live_abc123def456";
const DB_PASSWORD = "mySecretPassword123";

// ✅ 환경 변수 사용
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = process.env.DB_PASSWORD;
```

**검색 패턴:**
```bash
rg -i "(api[_-]?key|password|secret|token)\s*=\s*['\"]" \
  --type typescript \
  --type javascript
```

### 2. SQL 인젝션

```typescript
// ❌ 위험
const query = `SELECT * FROM users WHERE email = '${userInput}'`;

// ✅ 안전 (파라미터화된 쿼리)
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [userInput]);

// ✅ ORM 사용
const user = await db.user.findUnique({ where: { email: userInput } });
```

### 3. XSS (Cross-Site Scripting)

```typescript
// ❌ 위험
element.innerHTML = userInput;

// ✅ 안전
element.textContent = userInput;

// React에서
// ❌ 위험
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ 안전
<div>{userInput}</div>
```

### 4. 안전하지 않은 의존성

```bash
# package.json 스캔
bun audit

# 또는 npm
npm audit

# 자동 수정 (주의해서 사용)
bun audit --fix
```

### 5. 민감한 데이터 로깅

```typescript
// ❌ 위험
console.log('User logged in:', { email, password });

// ✅ 안전
console.log('User logged in:', { email });

// Logger 사용 시
logger.info('User logged in', { userId: user.id }); // 민감 정보 제외
```

### 6. 불안전한 파일 업로드

```typescript
// ❌ 위험
app.post('/upload', (req, res) => {
  const fileName = req.body.fileName;
  fs.writeFileSync(`./uploads/${fileName}`, req.body.content);
});

// ✅ 안전
import path from 'path';
import { sanitizeFileName } from '@/utils/security';

app.post('/upload', (req, res) => {
  const fileName = sanitizeFileName(req.body.fileName);
  const safeFileName = path.basename(fileName); // 경로 조작 방지
  
  // MIME 타입 검증
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
  if (!allowedTypes.includes(req.body.mimeType)) {
    return res.status(400).json({ error: 'Invalid file type' });
  }
  
  fs.writeFileSync(`./uploads/${safeFileName}`, req.body.content);
});
```

### 7. CSRF 토큰 누락

```typescript
// Express + CSRF 미들웨어
import csrf from 'csurf';

app.use(csrf({ cookie: true }));

app.post('/api/action', (req, res) => {
  // CSRF 토큰은 자동으로 검증됨
});
```

### 8. 인증/인가 누락

```typescript
// ❌ 위험 - 인증 없이 민감한 작업
app.delete('/api/users/:id', async (req, res) => {
  await db.user.delete({ where: { id: req.params.id } });
});

// ✅ 안전
import { authenticated, authorize } from '@/middleware/auth';

app.delete(
  '/api/users/:id', 
  authenticated, 
  authorize('admin'),
  async (req, res) => {
    await db.user.delete({ where: { id: req.params.id } });
  }
);
```

## 실행

```bash
# 수동 실행
/subagent security-checker

# CI/CD에 통합
# .github/workflows/security.yml
- name: Security Check
  run: |
    bun run security-check
    bun audit
```

## 자동 수정

가능한 경우 자동으로 수정:
- 환경 변수로 이동
- 안전한 함수로 교체
- 누락된 검증 추가

## 보고서 생성

```markdown
# Security Scan Report

## Critical Issues (2)
1. Hardcoded API key in src/api/client.ts:15
2. SQL injection vulnerability in src/db/queries.ts:42

## Warnings (5)
1. Deprecated dependency: lodash@4.17.20
2. Missing CSRF token in POST /api/action
...

## Recommendations
- Rotate exposed API keys immediately
- Update dependencies
- Add input validation
```
~~~

---

## 6. 워크플로우 자동화

### PostToolUse 훅

**`.claude/hooks/post-tool-use.sh`:**

```bash
#!/bin/bash

# PostToolUse hook - 도구 실행 후 자동 작업

TOOL_NAME=$1
FILE_PATH=$2

case $TOOL_NAME in
  "write_file")
    # 파일 작성 후 자동 포매팅
    if [[ $FILE_PATH == *.ts || $FILE_PATH == *.tsx ]]; then
      prettier --write "$FILE_PATH"
      echo "✅ Formatted: $FILE_PATH"
    fi
    
    if [[ $FILE_PATH == *.py ]]; then
      black "$FILE_PATH"
      echo "✅ Formatted: $FILE_PATH"
    fi
    
    # 특정 디렉토리는 자동 린트
    if [[ $FILE_PATH == src/api/* ]]; then
      eslint --fix "$FILE_PATH"
      echo "✅ Linted: $FILE_PATH"
    fi
    ;;
    
  "edit_file")
    # 파일 수정 후 타입 체크
    if [[ $FILE_PATH == *.ts || $FILE_PATH == *.tsx ]]; then
      bun run typecheck
    fi
    ;;
    
  "run_command")
    # 테스트 실행 후 커버리지 체크
    if [[ $FILE_PATH == *"test"* ]]; then
      if [ -f coverage/coverage-summary.json ]; then
        COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
        echo "📊 Test coverage: ${COVERAGE}%"
        
        if (( $(echo "$COVERAGE < 80" | bc -l) )); then
          echo "⚠️  Coverage below 80%"
        fi
      fi
    fi
    ;;
esac

exit 0
```

### PreToolUse 훅

**`.claude/hooks/pre-tool-use.sh`:**

```bash
#!/bin/bash

# PreToolUse hook - 도구 실행 전 검증

TOOL_NAME=$1
ARGS=$2

case $TOOL_NAME in
  "write_file")
    FILE_PATH=$(echo $ARGS | jq -r '.path')
    
    # 금지된 디렉토리 체크
    FORBIDDEN_DIRS=("node_modules" "dist" ".next" "coverage")
    for dir in "${FORBIDDEN_DIRS[@]}"; do
      if [[ $FILE_PATH == $dir/* ]]; then
        echo "❌ Cannot write to $dir directory"
        exit 1
      fi
    done
    ;;
    
  "run_command")
    COMMAND=$(echo $ARGS | jq -r '.command')
    
    # 위험한 명령어 차단
    DANGEROUS_COMMANDS=("rm -rf /" "rm -rf ~" "rm -rf *" "format c:")
    for cmd in "${DANGEROUS_COMMANDS[@]}"; do
      if [[ $COMMAND == *"$cmd"* ]]; then
        echo "❌ Dangerous command blocked: $cmd"
        exit 1
      fi
    done
    ;;
esac

exit 0
```

---

## 7. 병렬 실행 전략

### 터미널 설정

**iTerm2 프로필 생성:**

```bash
# ~/.zshrc 또는 ~/.bashrc

# Claude 세션 별칭
alias c1="cd ~/projects/myapp-1 && claude-code"
alias c2="cd ~/projects/myapp-2 && claude-code"
alias c3="cd ~/projects/myapp-3 && claude-code"
alias c4="cd ~/projects/myapp-4 && claude-code"
alias c5="cd ~/projects/myapp-5 && claude-code"

# 한 번에 모든 세션 시작
alias claude-fleet="
  open -a iTerm2 -n --args -e 'c1' &
  open -a iTerm2 -n --args -e 'c2' &
  open -a iTerm2 -n --args -e 'c3' &
  open -a ITerm2 -n --args -e 'c4' &
  open -a iTerm2 -n --args -e 'c5'
"
```

### 작업 분배 전략

```markdown
# 병렬 작업 분배 예시

**세션 1: 새 기능 개발**
- 브랜치: feature/user-dashboard
- 작업: 새로운 대시보드 컴포넌트 구현

**세션 2: 버그 수정**
- 브랜치: fix/login-error
- 작업: 로그인 에러 디버깅 및 수정

**세션 3: 테스트 스위트**
- 브랜치: test/api-coverage
- 작업: API 테스트 커버리지 향상

**세션 4: 리팩토링**
- 브랜치: refactor/database-layer
- 작업: 데이터베이스 레이어 정리

**세션 5: 문서화**
- 브랜치: docs/api-reference
- 작업: API 레퍼런스 문서 작성
```

### 웹-로컬 텔레포트

```bash
# 웹 세션을 로컬로 이동
claude-code --teleport <session-id>

# 로컬 세션을 웹으로 이동
claude-code --teleport-to-web

# 사용 사례:
# - 계획 단계: 웹 (깔끔한 UI)
# - 실행 단계: 터미널 (강력한 제어)
```

---

## 8. 검증 및 테스팅

### 자동 검증 루프

**`.claude/commands/verify-changes.md`:**

~~~markdown
# Verify Changes

변경사항을 자동으로 검증합니다.

## 검증 단계

### 1. 타입 체크
```bash
bun run typecheck
```

실패 시 자동으로 타입 오류 수정 시도.

### 2. 린팅
```bash
bun run lint --fix
```

### 3. 테스트 실행
```bash
# 변경된 파일 관련 테스트만
git diff --name-only | grep -E '\.(ts|tsx)$' | while read file; do
  test_file="${file/src/tests}"
  test_file="${test_file/.ts/.test.ts}"
  
  if [ -f "$test_file" ]; then
    bun test "$test_file"
  fi
done

# 전체 테스트
bun test
```

### 4. 빌드 확인
```bash
bun run build
```

### 5. E2E 테스트 (해당하는 경우)
```bash
# Playwright
bun test:e2e

# Cypress
bun cypress run
```

### 6. 브라우저 테스트 (UI 변경 시)

Chrome 확장 프로그램 사용:
1. 개발 서버 시작
2. 브라우저 열기
3. 수동 테스트 시나리오 실행
4. 스크린샷 캡처
5. 예상 동작과 비교

### 7. 성능 체크
```bash
# 번들 사이즈 체크
# 100KB 이상인 경우 경고
find dist -type f -size +100k -exec ls -lh {} \;
```

## 검증 보고서

```markdown
# Verification Report

## ✅ Passed
- Type check
- Lint
- Unit tests (125/125)
- Build

## ⚠️ Warnings
- Bundle size increased by 15KB
- Test coverage decreased from 85% to 83%

## ❌ Failed
None

## Recommendations
- Review bundle size increase
- Add tests for new utility functions
```
~~~

### CI/CD 통합

**`.github/workflows/claude-verify.yml`:**

```yaml
name: Claude Verification

on: [push, pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
        
      - name: Install dependencies
        run: bun install
        
      - name: Type check
        run: bun run typecheck
        
      - name: Lint
        run: bun run lint
        
      - name: Test
        run: bun test
        
      - name: Build
        run: bun run build
        
      - name: Security audit
        run: bun audit
```

---

## 9. 팀 협업 전략

### 팀 Git 워크플로우

**브랜치 전략**
- **main**: 프로덕션 코드
- **develop**: 개발 통합 브랜치
- **feature/***: 새 기능
- **fix/***: 버그 수정
- **release/***: 릴리스 준비

**규칙**
1. **main과 develop는 보호됨** - 직접 푸시 불가
2. **모든 변경은 PR을 통해** - 최소 1명의 리뷰 필요
3. **PR에는 Claude 검증 통과 필수**
4. **CLAUDE.md 업데이트는 별도 커밋**

**PR 템플릿**
`.github/pull_request_template.md`:
```markdown
## 변경 사항
