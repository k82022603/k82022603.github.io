---
title: "Code-Simplifier 개발자 가이드: Claude Code 팀의 리팩토링 비밀 무기"
date: 2026-01-09 22:10:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,   sub-agents,  code-refactoring,  Developers,  Claude.write]
---


## 들어가며: Claude Code 창시자가 매일 사용하는 도구

2026년 1월 초, Claude Code의 창시자이자 Anthropic의 Staff Engineer인 Boris Cherny가 자신의 개발 워크플로우를 공개했습니다. 이 공개는 개발자 커뮤니티에 충격을 주었습니다. "Claude Code 베스트 프랙티스를 창시자로부터 직접 배우지 않는다면 프로그래머로서 뒤처지는 것"이라는 평가가 나올 정도였습니다.

그의 워크플로우에서 가장 주목받은 요소 중 하나가 바로 code-simplifier 서브에이전트입니다. Boris는 "메인 작업이 끝난 후 아키텍처를 정리하기 위해" 이 에이전트를 사용한다고 밝혔습니다. 그리고 놀랍게도 Anthropic은 이 도구를 오픈소스로 공개했습니다. Claude Code 팀이 내부적으로 사용하던 실전 도구를 이제 누구나 사용할 수 있게 된 것입니다.

code-simplifier는 단순한 코드 포맷터가 아닙니다. 이것은 리팩토링 전문가(refactoring specialist)로 특화된 AI 에이전트입니다. 핵심 원칙은 명확합니다: **외부에서 관찰 가능한 동작이나 공개 API를 변경하지 않고 코드 품질을 개선합니다**. 기능을 유지하면서 가독성, 유지보수성, 그리고 코드의 "즐거움"을 높이는 것이 목표입니다.

왜 이것이 혁명적일까요? AI 코딩 도구들이 코드를 빠르게 생성하는 시대에, 진짜 병목은 "생성 속도"가 아니라 "생성된 코드의 품질"입니다. AI가 만든 코드는 작동하지만, 종종 다음과 같은 문제를 가집니다:

- **AI 특유의 "냄새"**: 과도하게 장황한 주석, 불필요한 방어 코드, 인간이 쓰지 않을 추상화
- **중복과 복잡성**: 같은 로직의 반복, 과도한 중첩, 일관성 없는 패턴
- **가독성 저하**: 변수명과 함수명의 불일치, 복잡한 조건문, 숨겨진 의도

code-simplifier는 이런 문제들을 체계적으로 해결합니다. 그리고 그것을 "슬롭(slop)"이라는 재미있는 용어로 부릅니다. AI가 생성한 불필요한 부산물들을 제거하고, 인간 개발자가 쓴 것처럼 깔끔한 코드로 만듭니다.

이 가이드는 code-simplifier의 작동 원리, 설치 및 설정, 실전 활용법, Boris Cherny의 워크플로우 통합 방법, 그리고 베스트 프랙티스를 종합적으로 다룹니다. 단순히 도구를 사용하는 방법뿐만 아니라, 왜 이 도구가 현대 AI 기반 개발에 필수적인지, 그리고 어떻게 팀 전체의 코드 품질을 향상시킬 수 있는지 설명합니다.

---

## 1부: Code-Simplifier 이해하기

### 1.1 Code-Simplifier란 무엇인가

code-simplifier는 Claude Code의 서브에이전트로, 코드 리팩토링에 특화된 전문 AI입니다. 일반적인 Claude Code 세션이 기능 구현에 집중한다면, code-simplifier는 **그 결과물을 더 나은 코드로 변환하는 것**에만 집중합니다.

기술적으로 code-simplifier는 커스텀 시스템 프롬프트를 가진 에이전트입니다. 이 프롬프트는 에이전트의 페르소나, 목표, 제약사항, 그리고 작동 원칙을 정의합니다. 핵심 페르소나는 "리팩토링 전문가(refactoring specialist)"이며, 코드를 더 명확하고, 간결하며, 유지보수하기 쉽게 만드는 것이 유일한 목적입니다.

**핵심 원칙: 동작 보존**  
code-simplifier의 가장 중요한 제약은 "외부에서 관찰 가능한 동작을 변경하지 않는다"는 것입니다. 이것은 다음을 의미합니다:

- **공개 API 유지**: 함수 시그니처, 클래스 인터페이스, 모듈 export가 동일하게 유지됩니다
- **기능 동등성**: 모든 입력에 대해 동일한 출력을 생성합니다
- **부작용 보존**: 파일 쓰기, 데이터베이스 업데이트 같은 부작용의 순서와 결과가 유지됩니다
- **에러 동작 일치**: 같은 조건에서 같은 예외를 발생시킵니다

명시적으로 승인받지 않는 한, code-simplifier는 절대 기능을 변경하지 않습니다. 이것이 안전하게 프로덕션 코드에 적용할 수 있는 이유입니다.

### 1.2 무엇을 "단순화"하는가

code-simplifier가 타깃하는 것은 명확합니다:

**1. AI가 생성한 "슬롭(Slop)" 제거**  
AI 특유의 패턴을 식별하고 제거합니다:
- 과도하게 장황한 주석 (인간이 쓰지 않을 스타일)
- 불필요한 방어 코드 (이미 검증된 경로에서의 추가 체크)
- 과도한 try/catch 블록 (코드베이스의 다른 부분과 불일치)
- AI 특유의 설명적 변수명 (너무 길고 자명함)

**2. 코드 중복 제거 (DRY 원칙)**  
같은 로직이 여러 곳에 반복되면 함수로 추출합니다. 유사한 패턴을 통합하고 공통 추상화를 찾습니다.

**3. 복잡성 감소**  
- 중첩된 조건문을 early return으로 평평하게
- 복잡한 boolean 표현식을 의미 있는 변수로 분해
- 긴 함수를 작은 함수로 분해
- 복잡한 상태 머신을 더 명확한 구조로

**4. 명명 개선**  
- 일관되지 않은 변수/함수명을 통일
- 모호한 이름을 명확하게 (data → userData, handle → processPayment)
- 약어를 풀어쓰기 (btn → button, ctx → context)
- 코드베이스의 명명 규칙에 맞추기

**5. 구조 개선**  
- 관련 로직을 함께 그룹화
- 책임 분리 (Single Responsibility Principle)
- 의존성 방향 정리
- 모듈 경계 명확화

### 1.3 무엇을 "유지"하는가

code-simplifier가 변경하지 **않는** 것도 중요합니다:

**1. 의도적인 설계 결정**  
- 성능 최적화 (캐싱, 메모이제이션, 특정 알고리즘 선택)
- 보안 체크 (입력 검증, 권한 확인)
- 에러 핸들링 전략 (재시도 로직, 폴백)
- 특정 프레임워크나 라이브러리 패턴

**2. 공개 인터페이스**  
- 함수 시그니처
- 클래스 생성자
- 모듈 export
- API 엔드포인트 형태

**3. 테스트 가능성**  
리팩토링 후에도 기존 테스트가 통과해야 합니다. 테스트가 깨진다면 동작이 변경된 것이므로 되돌립니다.

**4. 유용한 주석**  
모든 주석을 제거하는 것이 아닙니다. 다음은 유지됩니다:
- 비직관적 로직의 "왜"를 설명하는 주석
- 외부 제약사항 설명 (API 제한, 브라우저 호환성)
- 복잡한 알고리즘의 개요
- TODO와 FIXME (컨텍스트 제공)

### 1.4 Boris Cherny의 워크플로우에서의 역할

Boris의 워크플로우를 보면 code-simplifier의 위치가 명확합니다:

```
1. Plan Mode로 시작 → 구현 계획 수립
2. 구현 단계 → Claude가 기능 코드 작성
3. 검증 단계 → verify-app 에이전트로 테스트
4. 정리 단계 → code-simplifier로 리팩토링  ← 여기!
5. 커밋 → /commit-push-pr 명령으로 PR 생성
```

code-simplifier는 **4단계에서** 활성화됩니다. 메인 작업이 끝나고, 테스트가 통과한 후, PR을 만들기 전에 실행됩니다. 이것은 의도적인 위치입니다:

- **기능 구현 중에는 사용하지 않음**: 빠른 반복이 중요한 시점에 리팩토링은 방해됩니다
- **테스트 후에 사용**: 동작이 검증된 후에만 구조를 변경합니다
- **PR 전에 사용**: 코드 리뷰어가 깨끗한 코드를 받습니다

Boris는 이것을 "아키텍처 정리(clean up architecture)"라고 표현합니다. 기능은 완성되었지만, 코드가 아직 리뷰 준비가 안 된 상태에서 code-simplifier가 마지막 다듬질을 합니다.

---

## 2부: 설치 및 설정

### 2.1 공식 플러그인 설치

code-simplifier는 두 가지 방법으로 제공됩니다:

**방법 1: 독립 플러그인 (권장)**  
```bash
# 공식 플러그인 마켓플레이스 추가
claude plugin marketplace add anthropics/claude-code-plugins

# code-simplifier 설치
claude plugin install code-simplifier
```

**방법 2: PR 리뷰 툴킷의 일부**  
code-simplifier는 PR 리뷰 툴킷에도 포함되어 있습니다:
```bash
claude plugin install pr-review-toolkit
```

이 툴킷은 6개의 전문 에이전트를 제공하며, code-simplifier는 그 중 하나입니다. 다른 에이전트들:
- comment-analyzer: 주석 품질 검토
- pr-test-analyzer: 테스트 커버리지 분석
- silent-failure-hunter: 숨겨진 에러 발견
- type-design-analyzer: 타입 설계 검토
- code-reviewer: 일반 코드 리뷰

**방법 3: 세션 내에서 설치**  
Claude Code 세션 내에서도 설치 가능합니다:
```bash
# 세션 시작
claude

# 마켓플레이스 업데이트
/plugin marketplace update claude-plugins-official

# 설치
/plugin install code-simplifier
```

### 2.2 설치 확인

설치가 완료되면 code-simplifier 에이전트를 사용할 수 있습니다:

```bash
# 설치된 플러그인 확인
/plugin list

# code-simplifier 정보 확인
/plugin info code-simplifier
```

### 2.3 커뮤니티 버전

공식 버전 외에도 개발자들이 만든 여러 변형이 있습니다:

**Nick Nisi의 Essentials 플러그인**  
```bash
claude plugin marketplace add nicknisi/claude-plugins
claude plugin install essentials
```

Nick의 버전은 "de-slopify" 명령을 포함하며, AI가 생성한 패턴을 더 공격적으로 제거합니다.

**fcakyon의 general-dev 플러그인**  
```bash
claude plugin marketplace add fcakyon/claude-codex-settings
claude plugin install general-dev@claude-settings
```

이 버전은 다양한 유틸리티와 함께 code-simplifier를 제공합니다.

### 2.4 로컬 커스터마이징

code-simplifier를 프로젝트에 맞게 커스터마이즈하고 싶다면 로컬에서 설정할 수 있습니다:

**1단계: 에이전트 파일 생성**  
`.claude/agents/code-simplifier.md`:

```markdown
# Code Simplifier - Custom

You are Code Simplifier, specialized in refactoring code for [Your Project].

## Core Principles
- Maintain behavior
- Improve readability
- Follow [Your Project]'s conventions

## Project-Specific Rules
- Use our error handling pattern: try/catch only at API boundaries
- Prefer composition over inheritance
- Max function length: 50 lines
- Max file length: 300 lines

## Naming Conventions
- React components: PascalCase
- Hooks: useCamelCase
- Utilities: camelCase
- Constants: UPPER_SNAKE_CASE

## What NOT to simplify
- Performance-critical sections (marked with // PERF:)
- Third-party integrations (marked with // EXTERNAL:)
- Backward compatibility shims (marked with // COMPAT:)
```

**2단계: 사용**  
```bash
claude -a code-simplifier "Simplify the code in @src/components/UserProfile.tsx"
```

### 2.5 팀 전체 배포

팀에서 code-simplifier를 공유하려면:

**옵션 1: Git으로 설정 공유**  
`.claude/` 디렉토리를 Git에 체크인:
```bash
git add .claude/agents/code-simplifier.md
git commit -m "Add team code-simplifier configuration"
```

팀원들은 리포지토리를 클론하면 자동으로 같은 설정을 사용합니다.

**옵션 2: 팀 플러그인 마켓플레이스**  
팀 전용 플러그인 마켓플레이스를 만들 수 있습니다:
```bash
# 팀 마켓플레이스 추가
claude plugin marketplace add your-org/claude-plugins

# 팀원들이 설치
claude plugin install code-simplifier@your-org
```

---

## 3부: 실전 활용법

### 3.1 기본 사용 패턴

**패턴 1: 단일 파일 단순화**  
가장 간단한 사용법입니다:

```bash
claude -a code-simplifier "Simplify @src/utils/dataProcessor.js"
```

code-simplifier가 파일을 분석하고, 개선 사항을 식별하고, 리팩토링을 적용합니다. 완료되면 변경 사항을 요약합니다:

```
✓ Simplified 3 nested conditionals to early returns
✓ Extracted 2 duplicate blocks into helper function
✓ Renamed 4 variables for clarity (data → userMetrics, etc.)
✓ Removed 7 redundant comments
✓ Combined 3 similar functions into one parameterized function
```

**패턴 2: 디렉토리 전체 정리**  
모듈 전체를 개선할 때:

```bash
claude -a code-simplifier "Review and simplify all files in @src/api/"
```

code-simplifier는 디렉토리의 모든 파일을 순회하며 개선합니다. 이것은 리팩토링 세션의 끝에 유용합니다.

**패턴 3: 특정 함수 타겟팅**  
복잡한 함수만 집중적으로 개선:

```bash
claude -a code-simplifier "This function has too many nested ifs. 
Simplify the handleUserAuthentication function in @src/auth/handler.ts"
```

code-simplifier는 특정 함수에만 집중하고, 파일의 나머지는 건드리지 않습니다.

**패턴 4: 스타일 통일**  
명명 규칙이나 코딩 스타일을 통일할 때:

```bash
claude -a code-simplifier "The naming in @src/models/ is inconsistent. 
Standardize all variable and function names to match our style guide."
```

### 3.2 Boris Cherny의 워크플로우 통합

Boris는 code-simplifier를 특정 타이밍에 사용합니다. 그의 패턴을 따르려면:

**1단계: 기능 구현**  
일반 Claude Code 세션으로 기능을 구현합니다:
```bash
claude --mode plan "Implement user profile editing feature"
```

Plan mode로 계획을 세우고, 승인 후 구현합니다.

**2단계: 검증**  
verify-app 에이전트로 테스트합니다:
```bash
claude -a verify-app "Test the user profile editing feature"
```

이것은 end-to-end 테스트를 실행하고 UI를 확인합니다.

**3단계: 정리 (code-simplifier)**  
테스트가 통과하면 코드를 정리합니다:
```bash
claude -a code-simplifier "Clean up the code for user profile feature. 
Focus on @src/components/UserProfile/ and @src/api/userProfile.ts"
```

**4단계: 커밋 및 PR**  
```bash
claude /commit-push-pr
```

이 순서를 따르면 PR에 항상 깨끗한 코드가 들어갑니다.

### 3.3 슬래시 커맨드로 자동화

반복적으로 사용한다면 슬래시 커맨드로 만들 수 있습니다.

`.claude/commands/simplify.md`:

```markdown
# Simplify Code

Use the code-simplifier agent to clean up recent changes.

```bash
# Get list of modified files
MODIFIED_FILES=$(git diff --name-only HEAD)

# If no files modified, check staged files
if [ -z "$MODIFIED_FILES" ]; then
  MODIFIED_FILES=$(git diff --cached --name-only)
fi

# Show files
echo "Files to simplify:"
echo "$MODIFIED_FILES"
```

Use code-simplifier agent to simplify these files:
$MODIFIED_FILES

Focus on:
- Removing AI-generated verbosity
- Eliminating duplication
- Improving naming consistency
- Simplifying complex conditionals
```

사용:
```bash
claude /simplify
```

자동으로 변경된 파일들을 찾아 code-simplifier를 적용합니다.

### 3.4 PostToolUse Hook 통합

Boris는 PostToolUse hook을 사용해서 Claude가 코드를 작성할 때마다 자동 포맷팅합니다. code-simplifier도 비슷하게 통합할 수 있습니다.

`.claude/hooks.json`:

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": "bun run format || true"
        }
      ]
    }
  ]
}
```

더 공격적으로 code-simplifier를 자동 실행하려면:

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "agent",
          "agent": "code-simplifier",
          "condition": "if file size > 200 lines"
        }
      ]
    }
  ]
}
```

**주의**: 이것은 모든 편집 후 code-simplifier를 실행하므로 느릴 수 있습니다. 신중하게 사용하세요.

### 3.5 CLAUDE.md와의 시너지

Boris의 핵심 전략 중 하나는 CLAUDE.md 파일입니다. code-simplifier와 결합하면 강력합니다.

`CLAUDE.md`:

```markdown
# Code Simplification Rules

When using code-simplifier, enforce these project-specific patterns:

## Preferred Patterns
- Early returns over nested ifs
- Named exports (no default exports)
- Explicit error types (no generic Error)
- Pure functions where possible

## Avoid
- Classes (prefer functions and composition)
- Getters/setters (use plain properties)
- Abbreviations (write full words)
- Magic numbers (use named constants)

## Naming
- Events: onUserClick (not handleUserClick)
- Handlers: handleUserClick (not onUserClick)
- Async functions: fetchUser (not getUser)
- Boolean props: isActive (not active)
```

code-simplifier가 이 파일을 컨텍스트로 가지면 프로젝트 규칙에 맞게 리팩토링합니다:

```bash
claude -a code-simplifier "Simplify according to @CLAUDE.md rules"
```

### 3.6 Git 워크플로우 통합

**Pre-commit Hook**  
커밋 전에 자동으로 코드를 정리하도록 설정할 수 있습니다.

`.git/hooks/pre-commit`:

```bash
#!/bin/bash

# Get staged files
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|ts|jsx|tsx)$')

if [ -z "$STAGED_FILES" ]; then
  exit 0
fi

echo "Running code-simplifier on staged files..."

# Run code-simplifier
claude -a code-simplifier "Simplify these files: $STAGED_FILES" --auto-accept

# Re-stage modified files
git add $STAGED_FILES

exit 0
```

**주의**: pre-commit hook은 커밋을 느리게 만들 수 있습니다. 선택적으로 사용하세요.

**PR 리뷰 워크플로우**  
GitHub Actions로 PR마다 code-simplifier를 실행할 수 있습니다:

`.github/workflows/simplify.yml`:

```yaml
name: Code Simplification Suggestions

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  simplify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run code-simplifier
        run: |
          # Install Claude Code
          npm install -g @anthropic/claude-code
          
          # Get changed files
          CHANGED_FILES=$(git diff --name-only origin/main...HEAD)
          
          # Run code-simplifier and save suggestions
          claude -a code-simplifier "Analyze these files and provide simplification suggestions (don't modify): $CHANGED_FILES" > suggestions.md
      
      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const suggestions = fs.readFileSync('suggestions.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Code Simplification Suggestions\n\n${suggestions}`
            });
```

이것은 자동으로 코드를 수정하지 않고, 제안만 PR 코멘트로 남깁니다.

---

## 4부: 고급 패턴과 최적화

### 4.1 선택적 단순화

모든 코드를 단순화하고 싶지 않을 때가 있습니다. 특정 섹션을 보호하려면:

**주석 마커 사용**  
```javascript
// SIMPLIFIER:SKIP-START
// This is a performance-critical hot path
// Optimized manually, do not simplify
function fastPathProcessor(data) {
  // Complex but optimized logic
}
// SIMPLIFIER:SKIP-END
```

code-simplifier 프롬프트에 규칙 추가:
```bash
claude -a code-simplifier "Simplify @src/, but skip sections marked with SIMPLIFIER:SKIP"
```

**파일 레벨 제외**  
`.claude/agents/code-simplifier.md`에 제외 패턴 추가:

```markdown
## Exclusions
- Do not modify files in `src/vendor/` (third-party code)
- Do not modify files ending with `.generated.ts` (generated code)
- Do not modify test fixtures in `tests/fixtures/`
```

### 4.2 점진적 리팩토링

대형 코드베이스를 한 번에 리팩토링하는 것은 위험합니다. 점진적 접근법:

**Phase 1: 분석만**  
```bash
claude -a code-simplifier "Analyze @src/ and identify the top 10 files 
that would benefit most from simplification. Don't modify anything, just report."
```

code-simplifier가 우선순위 목록을 제공합니다.

**Phase 2: 하나씩 리팩토링**  
```bash
# 가장 문제가 많은 파일부터 시작
claude -a code-simplifier "Simplify @src/api/legacyHandler.ts"

# 테스트
npm test

# 다음 파일
claude -a code-simplifier "Simplify @src/utils/dataTransform.js"

# 테스트
npm test
```

각 리팩토링 후 테스트를 실행하고 커밋합니다. 문제가 생기면 쉽게 되돌릴 수 있습니다.

**Phase 3: 모듈 단위 통합**  
개별 파일들이 개선되면 모듈 레벨 일관성을 높입니다:
```bash
claude -a code-simplifier "Review @src/api/ module. 
All files have been simplified individually. 
Now ensure consistency across the module: naming, patterns, structure."
```

### 4.3 측정 및 추적

리팩토링의 영향을 측정하려면:

**복잡도 메트릭 추적**  
```bash
# 리팩토링 전
npx complexity-report src/ > before.json

# code-simplifier 실행
claude -a code-simplifier "Simplify @src/"

# 리팩토링 후
npx complexity-report src/ > after.json

# 비교
diff before.json after.json
```

일반적인 개선:
- Cyclomatic complexity 20-30% 감소
- 함수 길이 15-25% 감소
- 중복 코드 30-40% 감소

**커밋 메시지에 메트릭 포함**  
```bash
claude /commit-push-pr "refactor: Simplify user authentication module

Applied code-simplifier to src/auth/

Metrics:
- Cyclomatic complexity: 42 → 28 (-33%)
- Average function length: 45 → 32 lines (-29%)
- Duplicate blocks: 8 → 2 (-75%)
"
```

### 4.4 AI 슬롭 탐지 및 제거

Nick Nisi가 제안한 "de-slopify" 개념을 적용하려면:

**슬롭 패턴 정의**  
`.claude/agents/de-slop.md`:

```markdown
# De-Slop Agent

Your job is to identify and remove AI-generated patterns that don't fit human coding style.

## AI Slop Indicators

### Verbose Comments
```javascript
// BAD: AI slop
// This function takes a user object and returns the user's full name
// by concatenating the first name and last name with a space in between
function getUserFullName(user) {
  return `${user.firstName} ${user.lastName}`;
}

// GOOD: Human style
function getUserFullName(user) {
  return `${user.firstName} ${user.lastName}`;
}
```

### Unnecessary Abstractions
```javascript
// BAD: AI slop
function createStringConcatenator() {
  return (a, b) => a + b;
}
const concat = createStringConcatenator();
const result = concat("hello", "world");

// GOOD: Human style
const result = "hello" + "world";
```

### Defensive Programming Overkill
```javascript
// BAD: AI slop (in trusted code path)
function processValidatedData(data) {
  if (!data) throw new Error("Data is required");
  if (typeof data !== 'object') throw new Error("Data must be object");
  // data is already validated by caller
  // ...
}

// GOOD: Human style
function processValidatedData(data) {
  // caller has already validated
  // ...
}
```

Remove these patterns aggressively.
```

**사용**  
```bash
claude -a de-slop "Remove AI slop from @src/components/"
```

### 4.5 팀 코드 리뷰 통합

code-simplifier를 코드 리뷰 프로세스에 통합하면:

**패턴: 리뷰어 어시스턴트**  
리뷰어가 PR을 받으면:

```bash
# PR 브랜치 체크아웃
git checkout pr-branch

# code-simplifier로 개선 제안
claude -a code-simplifier "Review this PR and suggest simplifications. 
Don't modify code, just provide suggestions in markdown format."
```

code-simplifier가 리뷰 코멘트를 생성합니다:

```markdown
## Simplification Suggestions

### src/api/userService.ts

**Line 45-60: Complex nested conditionals**
Current code has 4 levels of nesting. Suggest using early returns.

**Line 78: Duplicate logic**
Same validation logic appears in line 145. Extract to `validateUserInput()`.

**Line 92: Unclear naming**
Variable `data` is too generic. Rename to `userProfileData`.

### src/components/UserForm.tsx

**Line 120-135: Long function**
`handleSubmit` is 50 lines. Consider extracting:
- Validation → `validateForm()`
- API call → `submitUserData()`
- UI update → `updateFormState()`
```

리뷰어는 이 제안을 PR 코멘트로 남기고, 원작자가 수용 여부를 결정합니다.

---

## 5부: Boris Cherny의 전체 워크플로우

code-simplifier는 Boris의 복잡한 시스템의 한 부분입니다. 전체 그림을 이해하면 더 효과적으로 사용할 수 있습니다.

### 5.1 Boris의 13가지 핵심 원칙

Boris가 공개한 워크플로우의 핵심:

**1. 병렬 처리**: 5개 터미널 + 5-10개 브라우저 세션  
**2. Opus 4.5 전용**: "가장 좋은 코딩 모델"  
**3. Plan Mode 우선**: 모든 중요한 작업은 계획부터  
**4. CLAUDE.md**: 팀 전체가 공유하는 지식 베이스  
**5. PR에서 학습**: @claude 태그로 실수를 문서화  
**6. 슬래시 커맨드**: 반복 워크플로우 자동화  
**7. 서브에이전트**: code-simplifier, verify-app 등  
**8. PostToolUse Hook**: 자동 포맷팅  
**9. 권한 관리**: --dangerously-skip 대신 /permissions  
**10. 검증 루프**: AI가 자기 작업을 검증 (2-3배 품질 향상)  
**11. 시스템 알림**: 작업 완료 시 알림  
**12. Teleport**: 로컬 ↔ 웹 세션 이동  
**13. 모바일 시작**: 아침에 폰으로 세션 시작, 나중에 체크

### 5.2 code-simplifier의 위치

Boris의 워크플로우에서 code-simplifier의 구체적 타이밍:

```
[작업 시작]
  ↓
[1] Plan Mode 진입 (Shift+Tab 2회)
  ↓
[2] 계획 반복 및 개선
  ↓
[3] 계획 승인 → 자동 수락 모드로 전환
  ↓
[4] Claude가 구현 (여러 파일, 많은 편집)
  ↓
[5] PostToolUse Hook → 자동 포맷 (bun run format)
  ↓
[6] verify-app 에이전트 → End-to-end 테스트
  ↓
[7] code-simplifier 에이전트 → 아키텍처 정리  ← 여기!
  ↓
[8] 최종 검토
  ↓
[9] /commit-push-pr → PR 생성
  ↓
[10] PR에서 @claude 태그 → CLAUDE.md 업데이트
  ↓
[작업 완료]
```

**핵심**: code-simplifier는 구현과 PR 사이의 다리입니다. 작동하는 코드를 리뷰 가능한 코드로 변환합니다.

### 5.3 검증 루프의 중요성

Boris가 강조하는 가장 중요한 원칙: **AI에게 작업을 검증할 방법을 제공하세요**.

code-simplifier도 검증이 필요합니다:

**방법 1: 테스트 자동 실행**  
code-simplifier를 실행한 후 항상 테스트:
```bash
claude -a code-simplifier "Simplify @src/api/" && npm test
```

테스트가 실패하면 리팩토링을 되돌립니다.

**방법 2: 리팩토링 전후 비교**  
```bash
# 현재 상태 백업
git stash

# 리팩토링
claude -a code-simplifier "Simplify @src/api/"

# 동작 비교 (integration tests)
npm run test:integration

# 문제가 있으면
git checkout .
git stash pop
```

**방법 3: 점진적 검증**  
```bash
claude -a code-simplifier "Simplify @src/api/userService.ts. 
After each change, run the relevant tests. If any test fails, revert that change."
```

code-simplifier가 변경 후마다 자체 검증합니다.

### 5.4 CLAUDE.md 활용 극대화

Boris 팀의 CLAUDE.md는 2000 토큰 정도로 간결합니다. code-simplifier 관련 섹션 예시:

```markdown
# Development Workflow

## Code Style
- **Always use `bun`, not `npm`**
- Prefer early returns over nested ifs
- Use named exports (no default exports)
- Max function length: 50 lines
- Max file length: 300 lines

## Simplification Rules (for code-simplifier)
- Remove redundant comments (keep only "why", not "what")
- Extract functions when logic repeats 2+ times
- Rename generic variables (data, item, thing → specific names)
- Flatten nested conditionals to early returns
- Combine similar functions with parameters

## Do NOT simplify
- Performance-critical paths in `src/performance/`
- Third-party adapter code in `src/adapters/`
- Test fixtures (they're intentionally verbose)
- Generated code (*.generated.ts files)

## Before committing
1. Run `bun run typecheck`
2. Run `bun run test`
3. Run `bun run lint`
4. Use code-simplifier on modified files
5. Use `/commit-push-pr` command
```

이 파일이 리포지토리에 있으면 code-simplifier가 자동으로 참조합니다.

---

## 6부: 베스트 프랙티스와 안티패턴

### 6.1 효과적인 code-simplifier 사용

**DO: 구현 후 사용**  
```bash
# 좋음
claude "Implement feature X"
# ... 구현 완료 ...
claude -a code-simplifier "Clean up the implementation"

# 나쁨
claude "Implement feature X and simplify as you go"
```

구현 중 단순화는 속도를 늦춥니다. 분리하세요.

**DO: 테스트와 함께**  
```bash
# 좋음
claude -a code-simplifier "Simplify @src/api/"
npm test

# 나쁨 - 테스트 없이 커밋
claude -a code-simplifier "Simplify @src/api/"
git add . && git commit -m "refactor"
```

항상 검증하세요.

**DO: 컨텍스트 제공**  
```bash
# 좋음
claude -a code-simplifier "Simplify @src/auth/. 
This module handles OAuth flow. Preserve the state machine logic. 
Focus on reducing duplication and improving naming."

# 나쁨
claude -a code-simplifier "@src/auth/"
```

더 많은 컨텍스트 = 더 나은 결과.

**DO: 점진적 적용**  
```bash
# 좋음 - 파일별로
for file in src/api/*.ts; do
  claude -a code-simplifier "Simplify $file"
  npm test
  git add $file
  git commit -m "refactor: Simplify $(basename $file)"
done

# 나쁨 - 한 번에 전체
claude -a code-simplifier "Simplify everything in src/"
git add . && git commit -m "refactor: Simplify all"
```

### 6.2 안티패턴: 피해야 할 것

**안티패턴 1: 과도한 자동화**  
```bash
# 나쁨 - 모든 편집 후 자동 실행
{
  "PostToolUse": [{
    "matcher": ".*",
    "hooks": [{"type": "agent", "agent": "code-simplifier"}]
  }]
}
```

이것은 세션을 느리게 만들고 불필요한 변경을 유발합니다.

**안티패턴 2: 컨텍스트 무시**  
```bash
# 나쁨
claude -a code-simplifier "Make this simpler" 
# (어떤 파일인지, 왜 복잡한지 설명 없음)
```

code-simplifier는 컨텍스트가 많을수록 좋습니다.

**안티패턴 3: 검증 없는 대규모 리팩토링**  
```bash
# 나쁨
claude -a code-simplifier "Simplify entire codebase"
# (테스트 없이 바로 커밋)
```

대규모 변경은 반드시 단계별로, 검증과 함께.

**안티패턴 4: 맹목적 수용**  
code-simplifier의 모든 제안을 무조건 수용하지 마세요. 특히:
- 성능에 영향을 줄 수 있는 변경
- 복잡한 비즈니스 로직 단순화 (의도가 숨겨질 수 있음)
- 명시적 에러 핸들링 제거

항상 변경 사항을 검토하세요.

**안티패턴 5: 문서화 없는 복잡성**  
```javascript
// 나쁨 - code-simplifier가 이것을 단순화하려 할 것
function complexButNecessary(data) {
  // 복잡하지만 성능상 이유로 필요한 로직
}
```

```javascript
// 좋음 - 주석으로 의도 명시
// PERF: This complex logic is optimized for O(log n) performance
// Simpler approaches tested but were 10x slower on large datasets
// Do not simplify without benchmarking
function complexButNecessary(data) {
  // 복잡하지만 성능상 이유로 필요한 로직
}
```

### 6.3 성능 고려사항

**code-simplifier는 느림**  
code-simplifier는 전체 파일을 분석하고, 개선점을 식별하고, 리팩토링을 적용합니다. 이것은 일반 코드 생성보다 느립니다.

**대략적 시간:**
- 단일 파일 (100-200 lines): 15-30초
- 모듈 (10-20 files): 3-5분
- 대형 코드베이스 (100+ files): 20-40분

**최적화 전략:**

**1. 병렬 처리**  
Boris처럼 여러 세션을 병렬로 실행:
```bash
# 터미널 1
claude -a code-simplifier "Simplify @src/api/"

# 터미널 2
claude -a code-simplifier "Simplify @src/components/"

# 터미널 3
claude -a code-simplifier "Simplify @src/utils/"
```

**2. 선택적 적용**  
모든 파일을 단순화할 필요는 없습니다:
```bash
# 변경된 파일만
CHANGED=$(git diff --name-only HEAD)
claude -a code-simplifier "Simplify only: $CHANGED"
```

**3. 캐싱**  
이미 단순화된 파일은 스킵:
```bash
# .simplified 마커 파일 사용
if [ ! -f .simplified-src-api-user.txt ]; then
  claude -a code-simplifier "Simplify @src/api/user.ts"
  touch .simplified-src-api-user.txt
fi
```

---

## 7부: 커뮤니티와 확장

### 7.1 인기 있는 code-simplifier 변형

**Nick Nisi의 "de-slopify"**  
더 공격적인 AI 슬롭 제거:
```bash
claude plugin install essentials@nicknisi

# 사용
claude "de-slopify @src/"
```

특징:
- AI 특유의 패턴을 더 적극적으로 탐지
- 과도한 추상화 제거
- 불필요한 주석 완전 제거
- 결과: 인간이 쓴 것처럼 보이는 코드

**PR Review Toolkit의 code-simplifier**  
PR 리뷰에 특화된 버전:
```bash
claude plugin install pr-review-toolkit

# 사용
claude /pr-review-toolkit:review-pr --aspects simplify
```

특징:
- PR 컨텍스트를 이해
- 변경된 파일에만 집중
- 리뷰 코멘트 형식으로 제안

### 7.2 커스텀 변형 만들기

자신만의 code-simplifier 변형을 만들려면:

**1단계: 베이스 에이전트 복사**  
```bash
# 공식 버전 다운로드
curl -o .claude/agents/my-simplifier.md \
  https://raw.githubusercontent.com/anthropics/claude-code/main/plugins/code-simplifier/agents/code-simplifier.md
```

**2단계: 프로젝트에 맞게 수정**  
`.claude/agents/my-simplifier.md`:

```markdown
# My Custom Code Simplifier

You are a refactoring specialist for [My Project].

## Base Principles
(공식 버전의 원칙 유지)

## Project-Specific Additions

### [My Project] Patterns
- Use our custom hook pattern: `useFeature()`
- Prefer our utility functions: `pipe()`, `compose()`
- Use our error types: `AppError`, `ValidationError`

### [My Project] Anti-patterns
- Never use `any` type (always find proper type)
- Never use `@ts-ignore` (fix the issue properly)
- Never use `console.log` (use our logger)

### [My Project] Naming
- API functions: `fetchUser` (not `getUser`)
- Event handlers: `onUserClick` (not `handleClick`)
- State: `isLoading` (not `loading`)
```

**3단계: 사용**  
```bash
claude -a my-simplifier "Simplify @src/"
```

### 7.3 code-simplifier 플러그인 생성

팀에서 공유할 플러그인을 만들려면:

**디렉토리 구조**  
```
my-simplifier-plugin/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   └── code-simplifier.md
└── README.md
```

**plugin.json**  
```json
{
  "name": "my-company-simplifier",
  "description": "Code simplifier customized for My Company",
  "version": "1.0.0",
  "author": {
    "name": "My Company Dev Team",
    "email": "dev@mycompany.com"
  },
  "agents": {
    "code-simplifier": {
      "file": "agents/code-simplifier.md",
      "description": "Simplify code following My Company standards"
    }
  }
}
```

**배포**  
```bash
# Git 리포지토리 생성
git init
git add .
git commit -m "Initial commit"
git push origin main

# 팀원들이 설치
claude plugin marketplace add mycompany/claude-plugins
claude plugin install my-company-simplifier
```

### 7.4 커뮤니티 기여

code-simplifier를 개선하고 싶다면:

**공식 리포지토리**  
https://github.com/anthropics/claude-code/tree/main/plugins/code-simplifier

**기여 방법**  
1. 개선 사항 식별 (특정 패턴, 새로운 리팩토링 규칙)
2. Issue 생성 또는 PR 제출
3. 예시 코드와 함께 Before/After 제공
4. 테스트 케이스 추가

**커뮤니티 디스커션**  
- Discord: Anthropic 공식 서버
- GitHub Discussions: claude-code 리포지토리
- X/Twitter: #ClaudeCode 해시태그

---

## 8부: 고급 주제

### 8.1 다른 도구와의 통합

**ESLint와 통합**  
code-simplifier 후 ESLint 자동 수정:
```bash
claude -a code-simplifier "Simplify @src/"
npx eslint --fix src/
```

**Prettier와 조합**  
```bash
claude -a code-simplifier "Simplify @src/"
npx prettier --write src/
```

Boris의 PostToolUse hook처럼 자동화:
```json
{
  "PostToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [
      {"type": "command", "command": "npx prettier --write {file}"},
      {"type": "command", "command": "npx eslint --fix {file}"}
    ]
  }]
}
```

**SonarQube 통합**  
```bash
# code-simplifier 실행
claude -a code-simplifier "Simplify @src/"

# SonarQube 분석
sonar-scanner

# 메트릭 비교
# Technical Debt 감소, Code Smells 감소 확인
```

### 8.2 AI 모델 선택

Boris는 Opus 4.5를 사용하지만, code-simplifier는 다른 모델로도 실행 가능합니다.

**모델 비교**  

| 모델 | 속도 | 품질 | 비용 | 추천 사용 |
|------|------|------|------|-----------|
| Opus 4.5 | 느림 | 최고 | 높음 | 복잡한 리팩토링 |
| Sonnet 4.5 | 중간 | 좋음 | 중간 | 일반 사용 |
| Haiku 4.5 | 빠름 | 괜찮음 | 낮음 | 간단한 정리 |

**모델 지정**  
```bash
# Sonnet 사용
claude --model sonnet-4.5 -a code-simplifier "Simplify @src/"

# Opus 사용 (Boris 스타일)
claude --model opus-4.5 -a code-simplifier "Simplify @src/"
```

**권장 사항**  
- 처음 시도: Sonnet (속도와 품질 균형)
- 복잡한 레거시 코드: Opus (최고 품질)
- 대량 일괄 처리: Haiku (빠른 피드백)

### 8.3 대규모 코드베이스 전략

수십만 라인의 코드베이스를 다룰 때:

**전략 1: 계층적 접근**  
```bash
# 1단계: 모듈 식별
claude -a code-simplifier "Analyze the codebase and identify 
the top 10 modules that would benefit most from simplification"

# 2단계: 모듈별 리팩토링
for module in $(cat priority-modules.txt); do
  claude -a code-simplifier "Simplify module: $module"
  npm test -- $module
  git add $module
  git commit -m "refactor: Simplify $module"
done

# 3단계: 크로스 모듈 일관성
claude -a code-simplifier "Review all simplified modules and 
ensure consistency in naming, patterns, and structure"
```

**전략 2: 점진적 마이그레이션**  
```bash
# 새 코드만 단순화
git diff --name-only main...feature-branch | \
  xargs -I {} claude -a code-simplifier "Simplify {}"
```

**전략 3: 스프린트 기반**  
매 스프린트마다 일정 시간을 리팩토링에 할애:
```
Sprint 1: src/api/ 모듈 (20% 시간)
Sprint 2: src/components/ 모듈 (20% 시간)
Sprint 3: src/utils/ 모듈 (20% 시간)
```

6개월 후 전체 코드베이스가 개선됩니다.

### 8.4 메트릭 및 ROI

code-simplifier의 가치를 정량화하려면:

**측정 지표**  

1. **복잡도 메트릭**
   - Cyclomatic Complexity
   - Cognitive Complexity
   - Halstead Metrics

2. **유지보수성 지표**
   - Maintainability Index
   - Technical Debt Ratio
   - Code Duplication %

3. **가독성 지표**
   - Average Function Length
   - Average File Length
   - Nesting Depth

**ROI 계산 예시**  

가정:
- 개발자 시급: $50
- code-simplifier 실행 시간: 30분
- 절약된 디버깅/유지보수 시간: 4시간/주

```
비용: 30분 × $50 = $25
절약: 4시간/주 × $50 × 52주 = $10,400/년

ROI: ($10,400 - $25) / $25 = 41,500%
```

물론 실제 절약 시간은 측정하기 어렵지만, 코드 품질 향상은 장기적으로 큰 이익을 가져옵니다.

---

## 9부: 문제 해결 및 FAQ

### 9.1 일반적인 문제

**문제 1: code-simplifier가 너무 공격적으로 변경**  

증상: 의도한 것보다 많은 코드가 변경됨  
원인: 충분한 컨텍스트 미제공  
해결책:
```bash
# 나쁨
claude -a code-simplifier "@src/api/"

# 좋음
claude -a code-simplifier "@src/api/
This is a mature, performance-critical module.
- Keep the current architecture
- Focus only on naming and removing duplication
- Don't change the state machine logic in handler.ts"
```

**문제 2: 테스트가 깨짐**  

증상: 리팩토링 후 테스트 실패  
원인: 동작 변경 또는 테스트가 구현 세부사항에 의존  
해결책:
```bash
# 리팩토링 되돌리기
git checkout .

# 더 보수적으로 재시도
claude -a code-simplifier "@src/api/
Only simplify if it maintains exact behavior.
Run tests after each change. If any test fails, revert that change."
```

**문제 3: AI 슬롭이 여전히 남아있음**  

증상: code-simplifier 후에도 장황한 주석이나 불필요한 코드가 남음  
원인: code-simplifier가 "안전하게" 작동  
해결책:
```bash
# de-slopify 사용 (더 공격적)
claude plugin install essentials@nicknisi
claude "de-slopify @src/"

# 또는 명시적 지시
claude -a code-simplifier "@src/
Be aggressive about removing:
- Any comment that just restates the code
- Any defensive check in already-validated code paths
- Any unnecessary abstraction layer"
```

**문제 4: 성능 저하**  

증상: 리팩토링 후 벤치마크가 느려짐  
원인: 성능 최적화 코드가 "단순화"됨  
해결책:
```bash
# 성능 크리티컬 섹션 보호
// PERF:CRITICAL - Do not simplify
// This loop is optimized for cache locality
for (let i = 0; i < array.length; i++) {
  // optimized logic
}
```

그리고 code-simplifier에 지시:
```bash
claude -a code-simplifier "@src/
Skip any section marked with PERF:CRITICAL"
```

### 9.2 자주 묻는 질문

**Q: code-simplifier를 매일 사용해야 하나요?**

A: 꼭 그럴 필요는 없습니다. Boris도 "메인 작업이 끝난 후" 사용합니다. 추천 타이밍:
- PR 생성 전
- 복잡한 기능 구현 후
- 코드 리뷰에서 "너무 복잡하다" 피드백 받았을 때
- 레거시 코드 리팩토링할 때

**Q: code-simplifier가 내 코드 스타일을 존중하나요?**

A: CLAUDE.md에 스타일 가이드를 명시하면 존중합니다. 그렇지 않으면 일반적인 베스트 프랙티스를 적용합니다.

**Q: 다른 리팩토링 도구와 비교하면?**

A:
- ESLint --fix: 문법적 문제만. code-simplifier는 구조적 개선
- Prettier: 포맷만. code-simplifier는 로직 개선
- SonarQube: 분석만. code-simplifier는 자동 수정
- IntelliJ Refactor: 단일 리팩토링. code-simplifier는 전체적 개선

code-simplifier는 이들을 대체하지 않고 보완합니다.

**Q: 프로덕션 코드에 안전한가요?**

A: 동작 보존 원칙을 지키므로 이론적으로 안전합니다. 하지만:
- 항상 테스트 실행
- 변경 사항 검토
- 점진적 적용
- 롤백 준비

**Q: 비용이 얼마나 드나요?**

A: code-simplifier는 일반 Claude Code보다 더 많은 토큰을 소비합니다 (분석 + 생성). Opus 4.5로 큰 파일을 단순화하면 $0.50-$2 정도. 하지만 절약되는 개발 시간을 고려하면 ROI가 높습니다.

**Q: 팀 전체가 같은 설정을 사용하게 하려면?**

A: `.claude/` 디렉토리를 Git에 커밋하세요. 모든 팀원이 자동으로 같은 설정을 사용합니다.

---

## 결론: 코드 품질의 새로운 기준

code-simplifier는 단순한 리팩토링 도구가 아닙니다. 이것은 AI 시대 개발의 새로운 패러다임을 상징합니다. AI가 코드를 빠르게 생성할 수 있는 시대에, 진짜 차별화 요소는 **생성 속도가 아니라 생성된 코드의 품질**입니다.

Boris Cherny와 Claude Code 팀이 code-simplifier를 오픈소스로 공개한 것은 그들의 철학을 보여줍니다: 최고의 도구는 독점이 아니라 공유되어야 합니다. 그들이 매일 사용하는 실전 도구를 커뮤니티와 나눔으로써, 전체 개발 생태계의 코드 품질을 높이고자 합니다.

**핵심 교훈**

1. **생성과 정리를 분리하세요**: 빠른 구현 후 체계적인 정리가 더 효율적입니다

2. **동작을 보존하세요**: 리팩토링은 기능을 변경하지 않습니다. 항상 테스트로 검증하세요

3. **AI 슬롭을 경계하세요**: AI가 만든 코드는 작동하지만 종종 인간이 쓰지 않을 패턴을 가집니다

4. **점진적으로 접근하세요**: 대규모 리팩토링은 한 번에 하지 말고 단계적으로 진행하세요

5. **팀과 공유하세요**: 코드 품질은 개인이 아닌 팀의 책임입니다. 도구와 지식을 공유하세요

**다음 단계**

code-simplifier를 시작하려면:

1. **설치**: `claude plugin install code-simplifier`
2. **첫 실행**: 작은 파일로 시작해서 결과를 관찰하세요
3. **커스터마이즈**: CLAUDE.md에 프로젝트 규칙 추가
4. **통합**: 워크플로우에 code-simplifier 단계 추가
5. **측정**: 코드 메트릭으로 개선 효과 확인
6. **공유**: 팀과 베스트 프랙티스 공유

code-simplifier는 도구입니다. 하지만 제대로 사용하면 코드베이스를 변화시키고, 개발 속도를 높이며, 유지보수 부담을 줄이는 강력한 자산이 됩니다. Boris Cherny가 말했듯이: "AI는 타이핑을 대체하는 것이 아니라 판단을 증폭시키는 것입니다." code-simplifier는 바로 그 증폭기입니다.

깨끗한 코드를 작성하세요. 하지만 먼저 작동하는 코드를 만드세요. 그리고 code-simplifier로 둘 다 가지세요.

---

**참고 자료**
- Boris Cherny의 원본 트윗 스레드: https://x.com/bcherny/status/2007179832300581177
- Claude Code 공식 문서: https://code.claude.com/docs
- code-simplifier 소스: https://github.com/anthropics/claude-code/plugins/code-simplifier
- PR Review Toolkit: https://github.com/anthropics/claude-code/plugins/pr-review-toolkit

**작성 일자: 2026-01-09**
