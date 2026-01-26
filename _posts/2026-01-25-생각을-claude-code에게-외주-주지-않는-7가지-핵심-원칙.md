---
title: "생각을 Claude Code에게 외주 주지 않는 7가지 핵심 원칙"
date: 2026-01-25 13:10:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  coding-agents,  claude-code,  TDD,  Claude.write]
---

## 2026년 프로덕션 레벨 AI 코딩을 위한 실전 가이드

---

## 들어가며: Claude Code가 바꾼 개발의 패러다임

2025년 2월, Anthropic이 Claude Code를 출시했을 때 많은 개발자들은 이것이 단순히 또 하나의 AI 코딩 도구라고 생각했습니다. 하지만 불과 1년도 안 되어 상황은 완전히 달라졌습니다. 2026년 1월 현재, Claude Code는 매주 1억 9천 5백만 줄 이상의 코드를 처리하고 있으며, 115,000명 이상의 활성 개발자가 실제 프로덕션 소프트웨어를 만드는 데 사용하고 있습니다.

하지만 수치보다 중요한 것은 이 도구가 개발자들의 일하는 방식 자체를 근본적으로 바꾸고 있다는 사실입니다. 한 개발자는 이를 "당신의 컴퓨터에 사는 작은 영혼 또는 유령(a little spirit/ghost that lives on your computer)"이라고 표현했습니다. Claude Code는 더 이상 코드를 복사-붙여넣기하는 보조 도구가 아닙니다. 터미널에서 직접 동작하며, 전체 코드베이스를 이해하고, 파일을 읽고 쓰고, 테스트를 실행하고, Git을 조작하는 진정한 **에이전트**입니다.

그러나 이러한 강력함은 양날의 검입니다. Claude Code를 단순히 "문제를 해결해줘"라고 요청하고 결과를 기다리는 도구로 사용한다면, 당신은 생산성 향상은커녕 기술 부채와 버그의 산을 쌓게 될 것입니다. 반대로 올바른 원칙을 이해하고 적용한다면, 일주일 분량의 작업을 하루 만에 완료하면서도 전문가 리뷰를 통과하는 코드 품질을 유지할 수 있습니다.

이 문서는 Anthropic의 공식 베스트 프랙티스 가이드, 수개월간의 실전 경험, 그리고 Claude Code 커뮤니티에서 검증된 패턴들을 종합하여 "생각을 AI에게 외주 주지 않는" 7가지 핵심 원칙을 제시합니다. 이는 단순한 팁이 아니라, 프로덕션 레벨의 소프트웨어를 만들기 위한 시스템적 접근법입니다.

---

## 원칙 1: CLAUDE.md로 프로젝트의 영구적 두뇌를 만들어라

### 매번 반복하는 고통에서 벗어나기

당신은 Claude Code에게 새 세션을 시작할 때마다 같은 말을 반복하고 있지 않습니까?

"TypeScript strict mode를 사용해줘."
"아니, CommonJS 말고 ES modules로 써줘."
"Lodash 쓰지 말고 네이티브 JavaScript로..."
"그거 아니야, 우리는 Repository 패턴을 사용한다니까!"

이것은 비효율적일 뿐만 아니라 위험합니다. 중요한 규칙을 잊어버리는 순간, Claude Code는 프로젝트 표준에 맞지 않는 코드를 생성하고, 당신은 그것을 수동으로 고치느라 시간을 낭비하게 됩니다.

### CLAUDE.md: 프로젝트의 영구적 기억

CLAUDE.md는 Claude Code가 자동으로 읽는 특별한 파일입니다. 이것은 당신의 프로젝트에 대한 **영구적인 두뇌**입니다. 한 번 작성하면, Claude Code는 매 세션마다 이 파일을 읽고 그 안의 규칙을 따릅니다.

Anthropic의 공식 가이드는 이렇게 설명합니다: "CLAUDE.md는 당신이 매번 반복해서 말할 필요 없이 Claude가 알아야 할 모든 것을 담는 곳입니다."

### 무엇을 포함시킬 것인가

**1. Bash 명령어**
```markdown
# Bash Commands
- `npm run build`: 프로젝트 빌드
- `npm run test`: 테스트 실행 (항상 코드 변경 후 실행할 것)
- `npm run typecheck`: TypeScript 타입 체크
- `npm run lint:fix`: ESLint 자동 수정
```

이것은 단순해 보이지만 강력합니다. Claude Code는 코드를 작성한 후 자동으로 `npm run typecheck`를 실행하여 타입 에러가 없는지 확인합니다.

**2. 코딩 스타일 가이드라인**
```markdown
# Code Style
- ES modules (import/export) 사용, CommonJS (require) 금지
- 가능한 경우 import를 destructure (예: `import { foo } from 'bar'`)
- 함수는 최대 50줄 이하로 유지
- async/await 사용, Promise.then() 체인 금지
- any 타입 절대 금지 (unknown 사용)
```

**3. 아키텍처 패턴과 제약사항**
```markdown
# Architecture
- Layered Architecture: Controller → Service → Repository
- 비즈니스 로직은 Service 계층에만 작성
- 데이터베이스 접근은 Repository 계층에서만
- DTO에 class-validator로 검증 로직 추가

# Dependencies
- 새로운 의존성 추가 전 반드시 정당한 이유 명시
- 이미 프로젝트에 있는 라이브러리로 해결 가능하면 새로 추가 금지
- 번들 크기 고려 (lodash 대신 lodash-es)
```

**4. 워크플로우와 품질 기준**
```markdown
# Workflow
- 코드 변경 완료 후 반드시 typecheck 실행
- 단일 테스트만 실행 (전체 테스트 스위트는 성능상 금지)
- 작은 diff 선호 - 요청하지 않은 리팩토링 금지

# Definition of Done
- 모든 테스트 통과
- 테스트 커버리지가 낮아지지 않음
- Lighthouse 성능 점수 ≥ 90 (변경된 페이지)
- PR에는 요약, 근거, 스크린샷, 테스트 노트 포함
```

**5. 해결된 버그와 알려진 문제**
```markdown
# Known Issues & Solutions

## Database Connection Pool Exhaustion
**문제**: 높은 부하 시 연결 풀 고갈
**해결책**: `max_connections=50, timeout=30` 설정
**관련 파일**: `config/database.ts`
**참조**: PR #456

## Race Condition in Cart Update
**문제**: 동시에 장바구니 업데이트 시 데이터 손실
**해결책**: Optimistic locking 패턴 사용
**구현**: `services/cart.service.ts:updateQuantity()`
**참조**: Issue #123
```

이것은 매우 중요합니다. Claude Code가 같은 실수를 반복하지 않도록 방지합니다.

### 파일 위치와 우선순위

CLAUDE.md는 다음 위치에 둘 수 있습니다:

1. **프로젝트 루트** (`/CLAUDE.md`): 전체 프로젝트 규칙
2. **모노레포의 각 패키지** (`/packages/api/CLAUDE.md`): 패키지별 규칙
3. **로컬 설정** (`/CLAUDE.local.md`): 개인 설정 (`.gitignore`에 추가)

Claude Code는 이 파일들을 계층적으로 읽습니다. 예를 들어, `/packages/api`에서 실행하면:
1. `/CLAUDE.md` (전체 프로젝트 규칙)
2. `/packages/api/CLAUDE.md` (패키지별 규칙)

### 실제 사례: Command Stick의 경험

ClaudeLog의 InventorBlack은 Command Stick 프레임워크 개발 중 CLAUDE.md를 "작업 컨텍스트, 규칙, 번호가 매겨진 단계, 예제의 모듈"로 구조화했습니다. 초기에는 Claude Code가 반복적으로 실패했지만, CLAUDE.md를 정제한 후 "마법처럼 작동했다"고 보고합니다.

이것이 핵심입니다. CLAUDE.md는 단순한 문서가 아니라 **프로젝트의 DNA**입니다. 이것을 올바르게 설정하면, Claude Code는 당신의 팀원처럼 일관된 품질의 코드를 생성합니다.

---

## 원칙 2: Plan-then-Execute로 설계를 먼저 확정하라

### 왜 바로 코딩하면 안 되는가

Claude Code에게 "사용자 인증 시스템을 구현해줘"라고 요청하고 즉시 코딩을 시작하게 하는 것은 건축가 없이 건물을 짓는 것과 같습니다. AI는 빠르게 코드를 생성할 수 있지만, **전체 그림을 보지 못하면** 다음과 같은 문제가 발생합니다:

- 일관성 없는 아키텍처
- 엣지 케이스 처리 누락
- 보안 취약점
- 나중에 변경하기 어려운 설계 결정

한 개발자는 이를 "10명의 개발자가 서로 대화 없이 작업한 것 같은 코드"라고 표현했습니다.

### Plan-then-Execute: 4단계 워크플로우

Anthropic 공식 베스트 프랙티스와 실무자들의 경험을 종합하면, 가장 효과적인 워크플로우는 다음과 같습니다:

**Step 1: 계획 요청하기**

```
프롬프트:
"사용자 인증 시스템을 구현하기 위한 상세한 계획을 작성해줘.
다음을 포함해야 해:
- 데이터 모델 (User, Session, Token)
- API 엔드포인트 목록과 스펙
- 보안 고려사항 (비밀번호 해싱, JWT, CSRF)
- 단계별 구현 순서
- 각 단계의 테스트 전략

아직 코드를 작성하지 마. 계획만 제시해줘."
```

"Think hard"나 "think deeply" 같은 구문을 추가하면 Claude Code가 더 많은 추론 토큰을 할당받습니다. 실제로 Claude Code 소스코드를 분석한 결과, "think hard"는 10,000 토큰의 추론 예산을, "ultrathink"는 31,999 토큰을 할당하는 것으로 밝혀졌습니다.

**Step 2: 명확한 중단 지시**

```
"잠깐, 코드 작성하지 마. 이 계획을 먼저 검토하고 싶어."
```

이것이 중요한 이유는 Claude Code의 기본 성향이 **행동 지향적(action-oriented)**이기 때문입니다. 명시적으로 멈추라고 하지 않으면, 계획을 제시하자마자 코딩을 시작할 수 있습니다.

**Step 3: 계획 검토 및 정제**

Claude Code가 제시한 계획을 읽어보세요. 다음을 확인합니다:

- **빠진 것은 없는가?** "비밀번호 재설정 기능이 빠졌네. 추가해줘."
- **잘못된 가정은 없는가?** "아니, 우리는 OAuth를 사용하지 않아. JWT만 사용해."
- **보안 문제는 없는가?** "비밀번호를 평문으로 저장하면 안 돼. bcrypt로 해싱해야 해."
- **성능 고려사항은?** "세션을 데이터베이스에 저장하면 느려. Redis를 사용하자."

대화를 통해 계획을 반복적으로 개선합니다:

```
나: "Step 2에서 JWT를 생성한다고 했는데, refresh token도 필요해."
Claude: "알겠습니다. 계획을 수정했습니다:
  2a. Access token (15분 유효)
  2b. Refresh token (7일 유효, httpOnly cookie로 저장)
  2c. Refresh token rotation 구현"
```

**Step 4: 승인 후 구현**

계획이 완벽하다고 판단되면:

```
"좋아, 이 계획대로 진행하자. 
Step 1부터 시작해줘: User 모델과 마이그레이션만 먼저 작성해."
```

### 왜 이것이 "느려 보이지만" 실제로는 더 빠른가

처음에는 이 과정이 시간 낭비처럼 느껴질 수 있습니다. "계획 없이 바로 코딩하면 더 빠르지 않나?"

하지만 실제로는 정반대입니다:

**계획 없이:**
- AI가 코드 작성 (10분)
- 당신이 리뷰하며 설계 문제 발견 (5분)
- "아니야, 이렇게 하면 안 돼" → 처음부터 다시 (10분)
- 또 다른 문제 발견 → 다시 수정 (10분)
- **총 35분 + 좌절감**

**계획 후:**
- 계획 수립 및 검토 (10분)
- 올바른 방향으로 코드 작성 (10분)
- 리뷰 및 마이너 수정 (5분)
- **총 25분 + 자신감**

더 중요한 것은 **코드 품질**입니다. 계획 기반 접근법은 일관된 아키텍처, 적절한 에러 처리, 보안 고려사항을 보장합니다.

### Spec-based 워크플로우: 다음 레벨

2025년 말 Claude Code 커뮤니티에서 인기를 얻은 "Spec-based" 접근법은 이를 한 단계 더 발전시킵니다:

```
1단계: 최소한의 스펙으로 시작
"사용자 인증 시스템이 필요해"

2단계: AskUserQuestionTool로 인터뷰
Claude가 질문:
- "어떤 인증 방법을 사용하나요? (이메일/비밀번호, OAuth, SSO?)"
- "세션 관리는 어떻게 하나요?"
- "비밀번호 정책은?"
- "2FA가 필요한가요?"

3단계: 상세 스펙 문서 생성
모든 요구사항이 명확해진 후, Claude가 완전한 사양 문서 작성

4단계: 새 세션에서 스펙 실행
사양 문서를 새 세션에 제공하고 구현 시작
```

이 접근법의 장점은 **모호성을 완전히 제거**한다는 것입니다. Claude Code가 가정하는 대신 명확히 묻고, 당신의 답변을 바탕으로 정확한 사양을 만듭니다.

---

## 원칙 3: Small Diffs로 점진적으로 반복하라

### "작은 변경, 큰 성공"의 철학

CLAUDE.md에 포함시켜야 할 가장 중요한 규칙 중 하나:

```markdown
# Coding Rules
- **작은 diff를 선호한다 - 요청하지 않은 리팩토링은 금지**
- 변경된 로직에 대한 테스트 추가/수정
- 한 번에 하나의 이슈만 다루는 PR
```

이것이 왜 중요할까요?

### 큰 변경의 문제점

Claude Code에게 "완전한 기능을 한 번에 구현해줘"라고 요청하면:

**문제 1: 리뷰 불가능**
- 300줄 변경 → 어디서 문제가 시작되었는지 알 수 없음
- 여러 파일 동시 수정 → 의존성 파악 어려움
- 큰 diff → 미묘한 버그 놓치기 쉬움

**문제 2: 롤백 어려움**
- 기능의 80%는 좋은데 20%가 문제
- 전체를 버릴 것인가, 수동으로 고칠 것인가?

**문제 3: 컨텍스트 한계**
- AI가 큰 변경을 추적하다가 일관성 잃음
- "10명의 개발자" 증후군 재발

### Small Diffs 전략

**1단계별 기능 추가**

```
나쁜 예:
"전자상거래 장바구니 시스템 전체를 구현해줘"
→ 500줄 코드, 10개 파일 변경, 리뷰 불가능

좋은 예:
"Step 1: 장바구니에 상품을 추가하는 API만 구현해줘
  - POST /api/cart/items
  - 요청: { productId, quantity }
  - 응답: { cartId, items }
  
  테스트를 포함해서 작성하고, 실행해서 통과하는지 확인해줘."
```

결과: 50줄 코드, 1-2개 파일, 명확한 리뷰

**2. Git 커밋 전략**

```bash
# 각 작은 변경마다 커밋
git commit -m "feat: Add cart item API endpoint"
# 테스트 실행 및 확인
npm test

git commit -m "feat: Add remove cart item endpoint"
# 테스트 실행 및 확인
npm test

git commit -m "feat: Add update quantity endpoint"
# 테스트 실행 및 확인
npm test
```

이렇게 하면:
- 각 커밋이 독립적인 체크포인트
- 문제 발생 시 정확한 커밋으로 롤백 가능
- 리뷰어가 변경사항을 단계별로 이해

**3. 요청하지 않은 리팩토링 방지**

Claude Code는 종종 "좋은 의도"로 요청하지 않은 개선을 합니다:

```
당신의 요청:
"login() 함수에 로깅 추가해줘"

Claude Code가 할 수 있는 것:
✓ login() 함수에 로깅 추가
✗ 파일 전체를 ES6로 변환
✗ 모든 함수에 JSDoc 추가
✗ 변수명 "개선"
✗ 코드 포맷팅 변경
```

CLAUDE.md에 명시적으로 금지하세요:
```markdown
- 요청하지 않은 리팩토링 금지
- 변경은 최소한으로 유지
- "개선"하고 싶으면 먼저 물어볼 것
```

### 실전 예시: 인증 시스템 구현

**전체 계획:**
1. User 모델 및 마이그레이션
2. 비밀번호 해싱 서비스
3. 회원가입 API
4. 로그인 API
5. JWT 토큰 생성/검증
6. 인증 미들웨어
7. 비밀번호 재설정

**실행:**
```
세션 1: "Step 1만 구현해줘"
→ User 모델, Prisma 마이그레이션
→ 테스트: npm run prisma:migrate
→ 커밋: "feat: Add User model"

세션 2: "Step 2 구현해줘"
→ PasswordService (bcrypt)
→ 테스트: 단위 테스트 작성 및 실행
→ 커밋: "feat: Add password hashing service"

세션 3: "Step 3 구현해줘"
→ POST /api/auth/signup
→ 테스트: API 통합 테스트
→ 커밋: "feat: Add signup endpoint"

... (계속)
```

각 단계는:
- 30분 이내 완료
- 명확한 성공 기준
- 독립적으로 테스트 가능
- 이해하기 쉬운 diff

---

## 원칙 4: 서브 에이전트로 컨텍스트를 격리하라

### 컨텍스트 오염의 문제

하나의 Claude Code 세션에서 여러 작업을 동시에 진행하면 어떤 일이 발생할까요?

```
당신: "버그를 수정해줘"
Claude: [버그 수정 시작]

당신: "아, 그런데 이 코드 보안 취약점도 검토해줘"
Claude: [보안 검토 시작, 버그 수정 컨텍스트 혼란]

당신: "문서도 업데이트해야 하는데..."
Claude: [이제 무엇을 하고 있는지 헷갈림]
```

이것이 **컨텍스트 오염(Context Pollution)**입니다. 하나의 대화에 너무 많은 다른 작업이 섞이면, AI는 집중력을 잃고 품질이 떨어집니다.

### 서브 에이전트: 전문가 팀 구성하기

2025년 중반 Claude Code에 도입된 서브 에이전트(Sub-Agents)는 이 문제를 혁신적으로 해결합니다. 서브 에이전트는 **특정 작업을 전담하는 전문화된 AI 인스턴스**입니다.

각 서브 에이전트는:
- **독립적인 컨텍스트 윈도우**: 다른 작업에 방해받지 않음
- **전문화된 시스템 프롬프트**: 특정 도메인에 최적화
- **제한된 도구 접근**: 필요한 도구만 사용
- **독립적인 권한**: 보안과 격리 보장

### 서브 에이전트 정의하기

`.claude/agents/` 디렉토리에 Markdown 파일로 정의합니다:

**보안 리뷰 에이전트** (`.claude/agents/security-reviewer.md`):
```markdown
---
name: security-reviewer
description: 보안 취약점 검사 전문가. 코드에서 SQL 인젝션, XSS, CSRF 등을 찾아냅니다.
tools: Read, Grep, Bash
---

당신은 애플리케이션 보안 전문가입니다.

## 검사 항목
1. SQL 인젝션: 사용자 입력이 쿼리에 직접 삽입되는가?
2. XSS: 사용자 입력이 이스케이프 없이 렌더링되는가?
3. CSRF: 상태 변경 엔드포인트에 토큰 검증이 있는가?
4. 인증: 민감한 엔드포인트가 보호되는가?
5. 비밀 정보: 하드코딩된 credential이 있는가?

## 보고 형식
각 발견사항에 대해:
- **심각도**: Critical/High/Medium/Low
- **위치**: 파일명:줄번호
- **설명**: 무엇이 문제인가
- **영향**: 공격자가 무엇을 할 수 있는가
- **수정 방법**: 구체적인 해결책

**Edit 도구를 사용하지 마세요.** 보고만 하고 수정은 하지 마세요.
```

**디버깅 에이전트** (`.claude/agents/debugger.md`):
```markdown
---
name: debugger
description: 에러, 테스트 실패, 예상치 못한 동작을 디버깅하는 전문가
tools: Read, Edit, Bash, Grep, Glob
---

당신은 근본 원인 분석 전문 디버거입니다.

## 디버깅 프로세스
1. 에러 메시지와 스택 트레이스 캡처
2. 재현 단계 확인
3. 실패 위치 격리
4. 최소한의 수정 구현
5. 솔루션 작동 검증

## 분석 접근법
- 에러 메시지와 로그 분석
- 최근 코드 변경 확인
- 가설 수립 및 테스트
- 전략적 디버그 로깅 추가
- 변수 상태 검사

## 각 이슈에 대한 보고
- 근본 원인 설명
- 진단을 뒷받침하는 증거
- 구체적인 코드 수정
- 테스트 접근법
- 재발 방지 권장사항
```

### 서브 에이전트 사용하기

**방법 1: 명시적 위임**
~~~
나: "security-reviewer 에이전트를 사용해서 
     auth.service.ts 파일의 보안 취약점을 검사해줘"

Claude: [security-reviewer 서브 에이전트 실행]
security-reviewer: 
"## 보안 검사 보고서

**Critical: SQL Injection**
- 위치: auth.service.ts:45
- 설명: 사용자 입력이 직접 쿼리에 삽입됨
  ```typescript
  const query = `SELECT * FROM users WHERE email = '${email}'`;
  ```
- 수정: Parameterized query 사용
  ```typescript
  const query = 'SELECT * FROM users WHERE email = ?';
  const result = await db.query(query, [email]);
  ```
"
~~~

**방법 2: 자동 위임**
Claude Code는 description을 보고 적절한 서브 에이전트를 자동으로 선택합니다:

```
나: "이 PR에 보안 문제가 있는지 검토해줘"

Claude: [자동으로 security-reviewer 서브 에이전트 선택 및 실행]
```

### 병렬 서브 에이전트: 생산성의 폭발

2025년 12월 업데이트로 도입된 **비동기 서브 에이전트**는 게임 체인저입니다. 이제 여러 서브 에이전트를 **동시에** 실행할 수 있습니다:

```
나: "4개의 병렬 태스크로 코드베이스를 탐색해줘. 
     각 에이전트는 다른 디렉토리를 탐색해."

Claude: 
Task(백엔드 구조 탐색) - 진행 중
Task(프론트엔드 컴포넌트 분석) - 진행 중
Task(데이터베이스 스키마 검토) - 진행 중
Task(API 엔드포인트 문서화) - 진행 중

[모든 태스크 완료 후 결과 통합]
```

**최대 병렬 처리**: 현재 Claude Code는 최대 10개의 서브 에이전트를 동시에 실행할 수 있습니다. 10개 이상 요청하면 큐에 대기하며, 하나가 완료되면 다음이 시작됩니다.

### 실전 사례: Linear 티켓 생성 워크플로우

한 개발자는 다음과 같은 멀티 에이전트 워크플로우를 구축했습니다:

**커스텀 명령어** (`.claude/commands/create-ticket.md`):
```markdown
---
name: create-ticket
description: 새 기능에 대한 Linear 티켓을 생성
---

다음 서브 에이전트들을 병렬로 실행하여 티켓을 작성하세요:

1. **product-manager**: 비즈니스 요구사항 정의
2. **ux-designer**: UI/UX 고려사항 작성
3. **senior-software-engineer**: 기술 구현 계획 수립

각 에이전트의 출력을 통합하여 완전한 Linear 티켓 생성.
```

**사용:**
```
나: "/create-ticket 사용자 프로필 페이지"

Claude:
Task(product-manager) - 비즈니스 요구사항 작성 중
Task(ux-designer) - UI/UX 스펙 작성 중  
Task(senior-software-engineer) - 기술 설계 작성 중

[몇 분 후 완성된 Linear 티켓 생성]
```

결과: 원래 몇 시간 걸리던 작업이 몇 분으로 단축.

### 서브 에이전트 전략

**언제 서브 에이전트를 사용할까?**

✅ **사용해야 할 때:**
- 작업이 상세한 출력을 생성하지만 메인 컨텍스트에는 요약만 필요
- 특정 도구 제한이나 권한을 강제하고 싶을 때
- 작업이 자기 완결적이고 요약을 반환할 수 있을 때
- 병렬 처리로 속도를 높이고 싶을 때

❌ **사용하지 말아야 할 때:**
- 단순한 반복 작업 (Skills 사용)
- 메인 컨텍스트에 모든 세부사항이 필요
- 레이턴시가 중요 (서브 에이전트는 새로 시작해서 컨텍스트 수집 시간 필요)

---

## 원칙 5: "Think Hard"로 깊은 추론을 유도하라

### 숨겨진 기능: 추론 예산 조절

2025년 4월, 한 개발자가 Claude Code의 번들된 JavaScript를 역공학하여 흥미로운 발견을 했습니다. "think hard" 같은 특정 구문이 Claude에게 **더 많은 추론 토큰**을 할당한다는 것입니다:

```javascript
// Claude Code 내부 코드
let prompt = message.content.toLowerCase();

if (prompt.includes("think harder") || 
    prompt.includes("think intensely") || 
    prompt.includes("think longer") || 
    prompt.includes("think really hard") || 
    prompt.includes("think super hard") || 
    prompt.includes("think very hard") || 
    prompt.includes("ultrathink")) {
    return 31999; // "메가싱크" 모드
}

if (prompt.includes("think about it") || 
    prompt.includes("think a lot") || 
    prompt.includes("think deeply") || 
    prompt.includes("think hard") || 
    prompt.includes("think more") || 
    prompt.includes("megathink")) {
    return 10000; // "딥싱크" 모드
}
```

### 언제 "Think Hard"를 사용할까?

**복잡한 아키텍처 결정:**
```
나: "Think deeply about this: 
     마이크로서비스 아키텍처와 모놀리식 아키텍처 중 
     우리 e-커머스 플랫폼에 더 적합한 것은?
     
     고려사항:
     - 팀 크기: 5명
     - 예상 트래픽: 일 10만 요청
     - 배포 빈도: 주 2-3회
     - 기술 스택: Node.js, PostgreSQL
     
     각 옵션의 장단점을 깊이 분석하고 권장사항을 제시해줘."
```

Claude는 더 많은 토큰을 할당받아 깊이 있는 분석을 수행합니다.

**어려운 버그 디버깅:**
```
나: "Think really hard about this bug:
     
     [에러 로그 붙여넣기]
     
     이것은 간헐적으로 발생하며 재현이 어렵습니다.
     가능한 모든 원인을 고려하고, 각 가설을 검증하는 방법을 제시해줘."
```

**성능 최적화:**
```
na: "Ultrathink: 
     이 쿼리가 왜 느린지 분석해줘.
     
     [느린 SQL 쿼리]
     
     실행 계획을 분석하고, 인덱스 전략, 쿼리 재작성, 
     캐싱 등 모든 최적화 옵션을 고려해줘."
```

### 일반 프롬프트 vs "Think Hard" 비교

**일반 프롬프트:**
```
나: "이 코드를 최적화해줘"

Claude: [빠르게 몇 가지 일반적인 최적화 적용]
```

**Think Hard 프롬프트:**
```
나: "Think deeply: 이 코드의 성능 병목을 심층 분석하고 최적화해줘"

Claude: [추론 과정]
"먼저 코드 흐름을 분석하겠습니다...
루프 내부의 데이터베이스 쿼리가 N+1 문제를 유발하고 있습니다...
배열 연산의 시간 복잡도를 계산하면...
메모리 프로파일링 결과...
종합하면, 다음 세 가지 최적화가 가장 효과적입니다..."

[구체적이고 근거 있는 최적화 제시]
```

### 주의사항

**1. 비용 고려**
더 많은 추론 토큰 = 더 높은 비용. 간단한 작업에는 불필요합니다.

**2. 항상 필요한 것은 아니다**
단순한 CRUD API나 boilerplate 코드에는 "think hard" 없이도 충분합니다.

**3. 플랜 단계에서 사용**
특히 효과적인 시점: **계획 수립 단계**
```
"Think deeply: 사용자 인증 시스템 구현 계획을 작성해줘"
```

---

## 원칙 6: 테스트로 AI의 눈을 제공하라

### AI는 자신이 작성한 코드가 작동하는지 모른다

이것은 많은 개발자들이 간과하는 중요한 사실입니다. Claude Code는 코드를 생성할 수 있지만, **실행하지 않으면 그것이 제대로 작동하는지 알 수 없습니다**.

테스트 없이:
```
Claude: "장바구니 기능을 구현했습니다."
당신: "고마워"
[나중에 발견: 수량 업데이트가 작동 안 함, 음수 수량 허용, 재고 확인 누락...]
```

테스트와 함께:
```
Claude: "장바구니 기능을 구현하고 테스트를 실행했습니다."
  ✓ 상품 추가
  ✓ 상품 제거  
  ✓ 수량 업데이트
  ✗ 음수 수량 테스트 실패
  
Claude: "음수 수량 검증을 추가했습니다. 다시 테스트..."
  ✓ 모든 테스트 통과
```

### TDD (Test-Driven Development)의 힘

Claude Code와 TDD는 완벽한 조합입니다:

**Step 1: 테스트 먼저 작성**
```
나: "먼저 테스트를 작성해줘. 
     장바구니 기능의 사양:
     - 상품을 추가할 수 있다
     - 같은 상품을 다시 추가하면 수량이 증가한다
     - 수량은 1 이상이어야 한다
     - 재고보다 많이 추가할 수 없다
     
     이 사양을 검증하는 테스트를 작성해줘."
```

**Step 2: 테스트 실행 (당연히 실패)**
```
Claude: "테스트를 작성했습니다. 실행해보겠습니다."
  npm test
  
  ✗ 상품을 추가할 수 있다 - FAIL (함수가 없음)
  ✗ 같은 상품을 다시 추가하면 수량이 증가한다 - FAIL
  ✗ 수량은 1 이상이어야 한다 - FAIL
  ✗ 재고보다 많이 추가할 수 없다 - FAIL
```

**Step 3: 테스트를 통과하는 코드 구현**
```
나: "이제 이 테스트들을 통과하는 코드를 구현해줘."

Claude: [코드 작성]
  npm test
  
  ✓ 상품을 추가할 수 있다
  ✓ 같은 상품을 다시 추가하면 수량이 증가한다
  ✓ 수량은 1 이상이어야 한다
  ✓ 재고보다 많이 추가할 수 없다
  
  모든 테스트 통과!
```

### CLAUDE.md에 테스트 규칙 명시

```markdown
# Testing Requirements

## 단위 테스트
- 모든 Service 함수는 단위 테스트 필수
- 테스트 커버리지 80% 이상 유지
- 각 테스트는 독립적이어야 함 (순서 무관)

## 통합 테스트
- 모든 API 엔드포인트는 통합 테스트 필수
- 성공 케이스와 실패 케이스 모두 테스트
- 인증, 권한, 에러 처리 검증

## 워크플로우
1. 새 기능 구현 시: 테스트 먼저 작성 (TDD)
2. 코드 작성 후: 즉시 테스트 실행
3. 테스트 실패 시: 디버깅하고 수정
4. 모든 테스트 통과할 때까지 반복

## Claude에게
- 코드를 작성한 후 **자동으로** 테스트를 실행하세요
- 테스트 실패 시 즉시 수정하세요
- "모든 테스트 통과" 확인 없이 작업 완료라고 하지 마세요
```

### 테스트를 통한 반복 개선

Claude Code의 강력한 기능 중 하나는 **테스트 결과를 보고 스스로 개선**할 수 있다는 것입니다:

```
나: "장바구니 수량 업데이트 기능을 구현하고 테스트해줘"

Claude: [코드 작성]
  npm test
  
  ✗ 수량 업데이트 테스트 - FAIL
  Expected: 5, Received: undefined

Claude: "실패 원인을 파악했습니다. return 문이 빠졌네요."
  [코드 수정]
  npm test
  
  ✓ 수량 업데이트 테스트 - PASS
```

이것이 바로 "AI가 자신의 눈을 가지게 된다"는 의미입니다. 테스트는 AI에게 피드백 루프를 제공하여 스스로 검증하고 개선할 수 있게 합니다.

### 회귀 테스트: 과거 실수 방지

```markdown
# Known Issues & Solutions

## 이슈 #123: 장바구니 동시 업데이트 시 데이터 손실
**문제**: 두 사용자가 동시에 수량을 업데이트하면 한 쪽 변경사항 손실
**해결**: Optimistic locking 패턴 적용
**테스트**: `cart.service.spec.ts:concurrentUpdateTest()`
**확인 방법**: 이 테스트가 항상 통과해야 함
```

새로운 변경사항이 있을 때마다 이 테스트가 실행되어 같은 버그가 재발하지 않도록 보장합니다.

---

## 원칙 7: 코드 리뷰를 절대 건너뛰지 마라

### AI 코드를 신뢰하지만 검증하라

"Trust but verify" - 이것이 AI 생성 코드를 다루는 황금률입니다.

Claude Code는 놀라울 정도로 좋은 코드를 생성할 수 있습니다. 하지만 **100% 완벽하지 않습니다**. 그리고 당신이 그 코드의 최종 책임자입니다.

### 리뷰 체크리스트

**1. 기능 정확성**
```
✓ 요청한 기능이 정확히 구현되었는가?
✓ 엣지 케이스가 처리되는가?
✓ 에러 처리가 적절한가?
```

**2. 코드 품질**
```
✓ 프로젝트 코딩 표준을 따르는가?
✓ 명명이 명확하고 일관적인가?
✓ 함수가 단일 책임을 가지는가?
✓ 불필요한 복잡성이 없는가?
```

**3. 보안**
```
✓ 사용자 입력이 검증되는가?
✓ SQL 인젝션 가능성은?
✓ XSS 취약점은?
✓ 민감한 정보가 노출되지 않는가?
```

**4. 성능**
```
✓ N+1 쿼리 문제는 없는가?
✓ 불필요한 루프나 재계산은?
✓ 메모리 누수 가능성은?
```

**5. 테스트**
```
✓ 적절한 테스트가 있는가?
✓ 테스트가 실제로 통과하는가?
✓ 커버리지가 충분한가?
```

### "설명해줘" 기법

**가장 강력한 리뷰 도구는 이것입니다:**

```
나: "방금 작성한 코드의 15-30줄을 설명해줘.
     특히 왜 Map 대신 Set을 사용했는지 설명해."

Claude: "이 부분은 중복을 제거하기 위해 Set을 사용했습니다..."
```

만약 Claude의 설명이 이해되지 않거나, 당신이 더 나은 방법을 알고 있다면:

```
나: "아니, Set보다 Map을 사용하는 게 나아. 
     각 항목의 개수를 추적해야 하거든.
     Map으로 바꿔줘."
```

**절대 규칙: 스스로 설명할 수 없는 코드는 절대 머지하지 마세요.**

### 크로스 모델 리뷰: 다중 관점

한 단계 더 나아가려면, 다른 AI 모델로 리뷰하세요:

```
워크플로우:
1. Claude Code로 코드 작성
2. 코드를 복사해서 GPT-4o에게:
   "이 코드의 보안, 성능, 유지보수성을 리뷰해줘"
3. GPT-4o의 피드백을 Claude Code에게:
   "위 리뷰 의견을 반영해서 개선해줘"
4. 개선된 코드를 Gemini에게 최종 검토 요청
```

각 모델은 다른 강점을 가지고 있어, 서로의 맹점을 보완합니다.

### 자동화된 리뷰 도구

CLAUDE.md에 추가:
~~~markdown
# Code Review Workflow

## 자동화 도구 (필수)
- ESLint: 코드 스타일 및 일반적인 문제 감지
- Prettier: 코드 포맷팅 일관성
- TypeScript: 타입 체크
- SonarQube: 코드 품질 및 보안 검사

## 리뷰 프로세스
1. 코드 작성 완료
2. 자동화 도구 실행:
   ```
   npm run lint
   npm run typecheck
   npm run test
   npm run security-scan
   ```
3. 모든 체크 통과 확인
4. 수동 코드 리뷰
5. 승인 후 머지
~~~

### 리뷰 부채 누적 방지

**안티패턴:**
```
AI에게 10개 기능을 한 번에 구현하게 함
→ 2000줄 코드 생성
→ 리뷰하려니 압도적
→ "나중에 리뷰하지 뭐" (영원히 안 함)
→ 버그 폭탄 누적
```

**베스트 프랙티스:**
```
각 작은 기능마다:
1. 구현 (50-100줄)
2. 즉시 리뷰 (5-10분)
3. 수정 필요시 바로 수정
4. 승인 후 다음 기능
```

리뷰는 누적되지 않고, 각 단계에서 품질이 보장됩니다.

---

## 실전 적용: 종합 워크플로우

지금까지의 7가지 원칙을 실제 프로젝트에 어떻게 적용할까요? 완전한 워크플로우를 단계별로 살펴보겠습니다.

### 프로젝트 초기 설정 (1회)

**1. CLAUDE.md 작성**
```bash
touch CLAUDE.md
```

내용 (프로젝트에 맞게 조정):
```markdown
# 프로젝트: E-Commerce Platform

## 기술 스택
- Backend: Node.js 20, TypeScript 5.3, Express
- Database: PostgreSQL 16, Prisma ORM
- Frontend: React 18, Next.js 14
- Testing: Vitest, Playwright

## Bash Commands
- `npm run dev`: 개발 서버 실행
- `npm run build`: 프로덕션 빌드
- `npm run test`: 단위 테스트
- `npm run test:e2e`: E2E 테스트
- `npm run typecheck`: TypeScript 검사
- `npm run lint:fix`: ESLint 자동 수정

## 코딩 표준
[원칙 1에서 설명한 내용]

## 아키텍처
[원칙 1에서 설명한 내용]

## 워크플로우
- 계획 먼저, 코딩은 나중 (Plan-then-Execute)
- 작은 diff로 점진적 개발
- TDD: 테스트 먼저, 구현은 나중
- 코드 작성 후 즉시 테스트 실행
```

**2. 서브 에이전트 설정**
```bash
mkdir -p .claude/agents
```

필요한 에이전트 생성:
- `security-reviewer.md`
- `debugger.md`
- `test-writer.md`
- `code-reviewer.md`

### 새 기능 개발 (매번)

**Phase 1: 계획 (10-15분)**

```
나: "Think deeply: 
     '상품 리뷰' 기능을 구현하려고 해.
     
     요구사항:
     - 사용자는 구매한 상품에 대해 리뷰를 작성할 수 있다
     - 별점 (1-5) + 텍스트 리뷰
     - 이미지 첨부 가능 (최대 3개)
     - 리뷰는 수정/삭제 가능 (본인만)
     - 상품 페이지에서 리뷰 목록 조회
     
     상세한 구현 계획을 작성해줘. 
     아직 코드는 작성하지 마."

Claude: [계획 수립]
"## 상품 리뷰 기능 구현 계획

### 1단계: 데이터 모델
- Review 모델 (id, userId, productId, rating, comment, createdAt, updatedAt)
- ReviewImage 모델 (id, reviewId, imageUrl)
- Migration 작성

### 2단계: API 엔드포인트
- POST /api/products/:id/reviews - 리뷰 작성
- GET /api/products/:id/reviews - 리뷰 목록
- PATCH /api/reviews/:id - 리뷰 수정
- DELETE /api/reviews/:id - 리뷰 삭제

### 3단계: 이미지 업로드
- S3 업로드 서비스
- 이미지 리사이징 (최대 1920x1080)

### 4단계: 권한 검증
- 구매 이력 확인
- 소유권 검증 (수정/삭제)

### 5단계: 테스트
- 각 단계별 테스트 작성
..."

나: "좋아, 하지만 3단계를 수정해줘. 
     우리는 이미 ImageUploadService가 있어. 
     그걸 재사용하자."

Claude: [계획 수정]

나: "완벽해. 이 계획을 specs/review-feature.md에 저장해줘."
```

**Phase 2: 단계별 구현 (각 30-45분)**

**Step 1: 데이터 모델**
~~~
나: "Step 1만 구현하자: 데이터 모델과 마이그레이션"

Claude: [Prisma schema 작성, migration 생성]

나: [schema 검토]
    "좋아. migration 실행하고 타입 생성해줘."

Claude: 
```bash
npm run prisma:migrate dev
npm run prisma:generate
```
Done!
```

나:
```bash
git add .
git commit -m "feat: Add Review and ReviewImage models"
```
~~~

**Step 2: API 엔드포인트 - TDD 방식**
~~~
나: "Step 2: 리뷰 작성 API 구현
     
     먼저 테스트를 작성해줘:
     - 성공 케이스: 유효한 리뷰 작성
     - 실패 케이스: 구매하지 않은 상품에 리뷰
     - 실패 케이스: 별점 범위 벗어남 (0, 6)
     - 실패 케이스: 비인증 사용자"

Claude: [테스트 작성]
```typescript
// reviews.test.ts
describe('POST /api/products/:id/reviews', () => {
  it('should create review for purchased product', async () => {
    // ...
  });
  
  it('should fail for non-purchased product', async () => {
    // ...
  });
  // ...
});
```

나: "테스트 실행해봐"

Claude:
```bash
npm test -- reviews.test.ts
```
모두 실패 (아직 구현 안 함)
```

나: "이제 테스트를 통과하는 코드를 구현해줘"

Claude: [API 엔드포인트 구현]
```bash
npm test -- reviews.test.ts
```
✓ 모든 테스트 통과!
```

나:
```bash
git add .
git commit -m "feat: Add create review endpoint with tests"
```
~~~

**Step 3-5: 같은 방식으로 반복**

각 단계마다:
1. 테스트 작성
2. 구현
3. 테스트 실행
4. 리뷰
5. 커밋

### Phase 3: 보안 리뷰 (15분)

```
나: "security-reviewer 에이전트를 사용해서 
     방금 구현한 리뷰 기능의 보안을 검토해줘"

Claude: [security-reviewer 서브 에이전트 실행]

security-reviewer:
"## 보안 검토 보고서

**High: SQL Injection 위험**
- 위치: reviews.controller.ts:25
- 문제: 정렬 파라미터가 직접 쿼리에 삽입됨
- 수정: 허용 목록 방식으로 검증

**Medium: Rate Limiting 부재**
- 위치: POST /api/products/:id/reviews
- 문제: 리뷰 스팸 방지 없음
- 수정: IP별 시간당 10개 제한

...
"

나: "이 문제들을 수정해줘"

Claude: [수정 구현 및 테스트]
```

### Phase 4: 성능 최적화 (선택적, 10분)

```
na: "Think deeply: 
     리뷰 목록 조회 API의 성능을 분석하고 최적화해줘.
     1000개 리뷰가 있는 상품을 가정."

Claude: [분석]
"현재 N+1 쿼리 문제 발견:
- 리뷰마다 개별 사용자 정보 조회
- 리뷰마다 개별 이미지 조회

최적화:
- Prisma의 include로 eager loading
- 페이지네이션 추가 (20개씩)
- 결과 캐싱 (Redis, 5분)

구현하겠습니다..."
```

### Phase 5: 최종 리뷰 및 PR (20분)

```bash
# 전체 테스트 실행
npm test
npm run test:e2e
npm run typecheck
npm run lint

# 모두 통과 확인

# PR 생성
git push origin feature/product-reviews

# Claude에게 PR 설명 작성 요청
na: "다음 내용으로 PR 설명을 작성해줘:
     - 기능 요약
     - 구현한 엔드포인트 목록
     - 보안 고려사항
     - 테스트 커버리지
     - 스크린샷 (있다면)
     - Breaking changes (있다면)"
```

---

## 함정 피하기: 일반적인 실수들

### 실수 1: CLAUDE.md 없이 시작

**증상:**
- 매 세션마다 같은 규칙 반복
- 일관성 없는 코드 스타일
- 프로젝트 표준 위반

**해결:**
- 프로젝트 시작 첫날에 CLAUDE.md 작성
- 팀원과 함께 검토
- 지속적으로 업데이트

### 실수 2: 큰 변경을 한 번에 요청

**증상:**
- 리뷰 불가능한 거대한 PR
- 어디서부터 잘못되었는지 파악 불가
- 롤백 어려움

**해결:**
- 기능을 5-10개 작은 단계로 분해
- 각 단계는 독립적으로 테스트 가능
- 단계마다 커밋

### 실수 3: 테스트 없이 개발

**증상:**
- "완성했어요" → 실행하면 에러 투성이
- 회귀 버그 빈발
- 리팩토링 두려움

**해결:**
- TDD 강제: 테스트 먼저, 코드 나중
- CLAUDE.md에 "테스트 없으면 완료 아님" 명시
- CI/CD에서 테스트 자동 실행

### 실수 4: 코드 리뷰 건너뛰기

**증상:**
- 보안 취약점 누적
- 기술 부채 폭증
- 나중에 "이게 왜 이렇게 되어 있지?" 의문

**해결:**
- AI 코드라도 사람 코드만큼 엄격히 리뷰
- "설명해줘" 기법 활용
- 자동화 도구 (ESLint, SonarQube)

### 실수 5: 서브 에이전트 과용

**증상:**
- 간단한 작업에도 서브 에이전트
- 비용 폭증
- 불필요한 복잡성

**해결:**
- 간단한 작업은 메인 세션에서
- 복잡하거나 격리가 필요한 작업만 서브 에이전트
- 병렬 처리가 실제로 도움이 되는지 확인

---

## 2026년 전망: 다음은 무엇인가

### Claude Code의 진화

2025년 12월 업데이트에서 도입된 기능들:

**1. LSP (Language Server Protocol) 지원**
- 실시간 진단
- 더 정확한 코드 완성
- 타입 정보 활용

**2. 비동기 서브 에이전트**
- 진정한 병렬 개발
- 최대 10개 동시 태스크
- 효율적인 큐잉

**3. Ultrathink 모드**
- 복잡한 문제에 더 많은 추론 능력
- 31,999 토큰 추론 예산
- 품질 향상

**4. Mobile & Browser 통합**
- Android 앱 지원
- `/chrome` 명령어로 Chrome에서 테스트
- 어디서나 개발 가능

**5. Agent Skills**
- 재사용 가능한 모듈
- 팀 간 공유
- 표준화된 워크플로우

### 개발자 역할의 변화

AI 코딩 도구가 발전할수록 개발자는:

**덜 중요해지는 것:**
- Boilerplate 코드 작성
- 구문 오류 수정
- 반복적인 CRUD 구현

**더 중요해지는 것:**
- **아키텍처 설계**: AI가 따를 전체 청사진
- **요구사항 이해**: 명확한 사양 작성
- **품질 관리**: 리뷰, 테스트, 보안
- **컨텍스트 엔지니어링**: CLAUDE.md 같은 가이드 작성
- **팀 협업**: AI 워크플로우 표준화

2026년 성공하는 개발자는:
- AI를 언제 신뢰하고 언제 의심할지 안다
- 효과적인 프롬프트를 작성한다
- 시스템적으로 품질을 보장한다
- 팀의 AI 사용을 리드한다

---

## 결론: 생각하는 개발자로 남기

이 문서의 제목은 "생각을 Claude Code에게 외주 주지 않는 7가지 핵심 원칙"입니다. 이것이 핵심 메시지입니다.

Claude Code는 당신의 생산성을 10배 높일 수 있습니다. 하지만 **당신이 생각을 멈추는 순간**, 그것은 재앙이 됩니다.

### 7가지 원칙 요약

1. **CLAUDE.md**: 프로젝트의 영구적 두뇌 만들기
2. **Plan-then-Execute**: 코딩 전 계획 수립
3. **Small Diffs**: 작은 변경으로 반복
4. **서브 에이전트**: 컨텍스트 격리로 품질 향상
5. **Think Hard**: 복잡한 문제에 깊은 추론 유도
6. **테스트**: AI의 눈 제공
7. **코드 리뷰**: 절대 건너뛰지 않기

### 오늘부터 시작하기

**1단계 (오늘):**
```bash
cd your-project
touch CLAUDE.md
# 위에서 제시한 템플릿으로 작성
git add CLAUDE.md
git commit -m "docs: Add CLAUDE.md for AI coding guidelines"
```

**2단계 (이번 주):**
- 다음 기능 구현 시 Plan-then-Execute 적용
- 작은 단위로 쪼개서 개발
- 각 단계마다 커밋

**3단계 (이번 달):**
- 서브 에이전트 설정
- 팀과 베스트 프랙티스 공유
- TDD 워크플로우 확립

### 마지막 조언

Claude Code는 도구입니다. 강력하지만, 도구일 뿐입니다.

진정한 가치는 **당신의 판단, 경험, 그리고 생각**에서 나옵니다.

AI에게 "이것 해줘"라고 하는 것은 쉽습니다.
하지만 "왜", "어떻게", "무엇을 위해"를 생각하는 것은 여전히 당신의 몫입니다.

**생각을 외주 주지 마세요.**
**AI를 파트너로 만드세요.**

그리고 그 파트너십의 품질은 당신이 얼마나 명확하게 생각하고, 계획하고, 검증하느냐에 달려 있습니다.

---

## 참고 자료

### 공식 문서
- Anthropic: Claude Code Best Practices
  https://www.anthropic.com/engineering/claude-code-best-practices
- Claude Code Documentation
  https://code.claude.com/docs/en/best-practices
- Claude Code Sub-Agents Guide
  https://code.claude.com/docs/en/sub-agents

### 커뮤니티 리소스
- ClaudeLog - 실전 경험과 최적화 기법
  https://claudelog.com/
- r/ClaudeAI - 430,000+ Claude 사용자 커뮤니티
- Claude Code GitHub 예제
  https://github.com/wshobson/agents

### 심화 학습
- "Context Engineering for AI Agents" (Manus, 2025)
- "Advanced Claude Code Techniques" (Salwan Mohamed, 2025)
- "Building Agents with the Claude Agent SDK" (Anthropic)

### 도구 및 템플릿
- Claude Code Agent Templates
  https://github.com/zhsama/claude-sub-agent
- CLAUDE.md Examples
  https://www.gend.co/blog/claude-skills-claude-md-guide

---

**작성일자**: 2026-01-25

**이 문서는 살아있는 문서입니다.** Claude Code가 진화함에 따라, 커뮤니티의 베스트 프랙티스도 진화합니다. 새로운 패턴을 발견하면 공유하고, 다른 개발자의 경험에서 배우세요.

Happy Coding with Claude! 🚀
