---
title: "Claude Code 슬래시 커맨드와 스킬 통합: 완벽 가이드"
date: 2026-01-31 20:00:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,  claude-code,  Skills,  slash-commands,  frontmatter,  SKILL.md,  Claude.write]
---


## 들어가며

Claude Code 버전 2.1.3에서 일어난 가장 중요한 변화 중 하나는 슬래시 커맨드(Slash Commands)와 스킬(Skills) 시스템의 통합입니다. 이 변화는 겉보기에는 작은 업데이트처럼 보일 수 있지만, 실제로는 개발 워크플로우를 근본적으로 개선하는 중요한 전환점입니다. 

Anthropic은 이 통합을 진행하면서도 기존에 구축된 모든 것을 그대로 유지하는 완벽한 하위 호환성을 제공했습니다. 여러분이 이미 만들어둔 슬래시 커맨드와 스킬은 아무런 수정 없이 그대로 작동합니다. 이는 단순히 두 개의 비슷한 기능을 하나로 합친 것이 아니라, 개발자들이 수개월 동안 겪어온 혼란을 해소하고 더 직관적이고 강력한 시스템을 만들어낸 것입니다.

## 통합 이전: 두 개의 유사한 시스템

Claude Code를 사용하던 개발자들은 커스텀 워크플로우를 만들기 위해 두 가지 방법 중 하나를 선택해야 했습니다. 이 두 가지 방법은 표면적으로는 비슷해 보였지만, 미묘한 차이점들이 있어 많은 혼란을 야기했습니다.

### 슬래시 커맨드의 작동 방식

슬래시 커맨드는 `.claude/commands/` 디렉토리에 단일 마크다운 파일로 존재했습니다. 예를 들어 `/review`라는 커맨드를 만들고 싶다면, `.claude/commands/review.md` 파일을 생성하면 되었습니다. Claude Code는 이 파일을 자동으로 인식하고 `/review`를 커맨드로 등록했습니다.

이러한 슬래시 커맨드는 주로 사용자가 직접 호출하는 방식으로 설계되었습니다. 개발자가 명시적으로 `/review`를 입력하면 해당 커맨드가 실행되는 구조였습니다. 파일 하나로 완결되는 단순한 구조 덕분에 빠르게 만들고 사용할 수 있다는 장점이 있었습니다.

### 스킬의 작동 방식

반면 스킬은 `.claude/skills/` 디렉토리에 전체 디렉토리 구조로 존재했습니다. 하나의 스킬은 `SKILL.md` 파일을 중심으로 여러 지원 파일들을 포함할 수 있는 폴더였습니다. 예를 들어 리뷰 스킬을 만든다면 `.claude/skills/review/SKILL.md`와 같은 구조를 갖게 됩니다.

스킬은 자동 발견(auto-discovery)을 염두에 두고 설계되었습니다. Claude가 대화 컨텍스트를 분석해서 스킬의 설명과 매칭되면 자동으로 해당 스킬을 로드하는 방식이었습니다. 사용자가 명시적으로 호출하지 않아도 Claude가 상황에 맞게 적절한 스킬을 불러올 수 있었습니다.

### 혼란의 원인

문제는 이 두 시스템이 실제로는 그렇게 다르지 않다는 점이었습니다. 둘 다 `/` 접두사를 사용하는 커스텀 워크플로우를 만들었고, 둘 다 Claude에 의해 자동으로 호출될 수 있었으며, 둘 다 대화에 지시사항을 로드하는 역할을 했습니다.

개발자들은 무언가를 만들 때마다 "이것을 커맨드로 만들어야 할까, 스킬로 만들어야 할까?"라는 질문에 직면했습니다. 비교표를 읽고, 차이점을 분석하고, 신중하게 선택했지만, 결국 잘못된 선택을 하는 경우가 많았습니다. 이러한 불필요한 복잡성은 개발 경험을 저해하는 요소였습니다.

## 통합 이후: 하나로 합쳐진 시스템

버전 2.1.3의 통합은 이러한 혼란을 완전히 해소했습니다. 이제 `.claude/commands/review.md` 파일과 `.claude/skills/review/SKILL.md` 파일은 완전히 동일하게 작동합니다. 둘 다 `/review` 커맨드를 생성하고, 시스템은 이 둘을 구별하지 않습니다.

### 주요 변화점

통합은 어느 한 접근 방식을 제거한 것이 아니라, 두 방식을 하나의 메커니즘으로 통일한 것입니다. `.claude/commands/`에 있는 파일들은 이전과 완전히 동일하게 작동하며, 마이그레이션이 필요하지 않습니다.

스킬은 이제 `/skill-name` 형식으로 명시적으로 호출할 수 있는 능력을 얻었습니다. 이는 커맨드가 항상 가지고 있던 특성입니다. 반대로 커맨드는 Claude에 의해 자동으로 발견될 수 있는 능력을 얻었습니다. 이는 스킬이 항상 가지고 있던 특성입니다.

이제 구분은 기능적인 것이 아니라 구조적인 것입니다. 간단한 커맨드는 `.claude/commands/`에 단일 마크다운 파일로 유지하면 됩니다. 복잡한 스킬은 디렉토리 구조를 활용해 여러 파일, 번들된 스크립트, 템플릿을 조직적으로 관리할 수 있습니다.

### 결정의 단순화

이제 개발자가 내려야 할 결정은 매우 명확합니다. 기능이나 능력의 차이가 아니라 조직화 방식의 차이입니다. 

간단한 지시사항 하나만 필요하다면 `.claude/commands/`에 파일 하나를 만들면 됩니다. 여러 지원 파일이 필요하거나, 템플릿을 포함하거나, 스크립트를 번들링해야 한다면 `.claude/skills/`에 디렉토리 구조로 만들면 됩니다.

둘 다 동일한 방식으로 호출되고, 둘 다 자동 발견될 수 있습니다. 선택은 순전히 프로젝트의 복잡도와 조직화 필요성에 따라 결정하면 됩니다.

### 하위 호환성의 완벽함

통합의 가장 인상적인 부분은 완벽한 하위 호환성입니다. 여러분이 작성한 모든 커맨드는 계속 작동합니다. 구축한 모든 스킬도 기능적으로 아무런 변화 없이 동작합니다.

기존 워크플로우를 마이그레이션하거나 수정할 필요가 전혀 없습니다. 업데이트를 적용하는 순간부터 모든 것이 그냥 작동합니다. 이는 Anthropic이 얼마나 신중하게 이 변화를 설계하고 구현했는지를 보여주는 증거입니다.

## Agent Skills 표준과 이식성

Claude Code의 스킬은 Agent Skills라는 오픈 스탠다드를 따릅니다. 이것이 의미하는 바는 매우 중요합니다. 여러분이 작성한 스킬은 Claude Code에서만 작동하는 것이 아니라, 여러 AI 도구에서 사용될 수 있다는 뜻입니다.

웹 인터페이스인 claude.ai도 스킬을 지원하며, Claude Desktop도 동일한 스킬 시스템을 사용합니다. 이들은 모두 같은 구조, 같은 프론트매터 필드, 같은 호출 모델을 공유합니다.

이러한 이식성은 스킬을 더욱 가치 있게 만듭니다. 한 번 작성하면 여러 곳에서 사용할 수 있기 때문입니다. 통합으로 인해 이러한 이점은 단순한 커맨드에도 확장되었습니다. `.claude/commands/`에 있는 모든 것은 필요할 때 이식 가능한 스킬로 변환될 수 있습니다.

## 커맨드 생성: 슬래시 커맨드 접근법

가장 간단한 방식부터 살펴보겠습니다. 슬래시 커맨드는 `.claude/commands/`에 단일 파일 구조로 생성됩니다. 각 커맨드는 독립적인 마크다운 파일입니다.

### 기본 커맨드 만들기

코드 리뷰를 위한 커맨드를 만들어보겠습니다. `.claude/commands/review.md` 파일을 다음과 같이 작성합니다:

```markdown
---
description: Review code for best practices and potential issues
---
Review the current code following these guidelines:
1. Check for obvious bugs or logic errors
2. Verify proper error handling
3. Look for security vulnerabilities
4. Suggest performance improvements
5. Verify code follows project conventions

Provide specific line numbers and concrete suggestions for improvements.
```

파일을 저장하면 `/review` 커맨드가 즉시 Claude Code에서 사용 가능해집니다. `/review`를 입력하고 엔터를 누르면, Claude가 이 지시사항을 실행합니다.

프론트매터의 `description` 필드는 선택사항이지만 매우 중요합니다. 이 설명을 통해 Claude는 언제 이 커맨드를 자동으로 로드해야 할지 판단할 수 있습니다. 명확하고 구체적인 설명을 작성하면, 대화 중에 자연스럽게 커맨드가 활성화됩니다.

### 커맨드의 즉시성

슬래시 커맨드 접근법의 가장 큰 장점은 즉시성입니다. 복잡한 디렉토리 구조를 만들 필요 없이, 단일 파일로 즉시 기능을 추가할 수 있습니다. 프로토타이핑이나 빠른 워크플로우 구축에 이상적입니다.

버전 2.1.0부터는 핫 리로드(hot reload)가 지원됩니다. 파일을 생성하거나 수정하면 Claude Code 세션을 재시작할 필요 없이 즉시 변경사항이 반영됩니다. 시스템이 `.claude/commands/` 디렉토리를 감시하고 있다가 변화를 실시간으로 적용합니다.

## 커맨드 생성: 스킬 접근법

스킬은 디렉토리 구조를 사용합니다. 각 스킬은 `SKILL.md` 파일과 선택적인 지원 파일들을 포함하는 폴더입니다.

### 기본 스킬 만들기

동일한 리뷰 기능을 스킬로 만들어보겠습니다. `.claude/skills/review/SKILL.md` 파일을 생성합니다:

```markdown
---
name: review
description: Review code for best practices and potential issues
---
Review the current code following these guidelines:
1. Check for obvious bugs or logic errors
2. Verify proper error handling
3. Look for security vulnerabilities
4. Suggest performance improvements
5. Verify code follows project conventions

Provide specific line numbers and concrete suggestions for improvements.
```

이것은 슬래시 커맨드 버전과 동일한 `/review` 커맨드를 생성합니다. 지시사항과 동작은 완전히 같습니다.

프론트매터에 `name` 필드가 있다는 점이 차이입니다. 이 필드가 실제 커맨드 이름을 정의합니다. 디렉토리 이름과 커맨드 이름이 다를 수 있다는 뜻입니다.

### 네이밍 컨벤션

스킬의 네이밍은 유연합니다. `.claude/skills/review/SKILL.md`에서 `name: review`로 설정하면 `/review`가 됩니다. 하지만 `.claude/skills/code-review/SKILL.md`에서도 `name: review`로 설정할 수 있고, 여전히 `/review` 커맨드가 생성됩니다.

디렉토리 이름은 조직화를 위한 것이고, `name` 필드가 실제 커맨드를 제어합니다. 이를 통해 논리적으로 스킬을 조직하면서도 커맨드 이름은 깔끔하게 유지할 수 있습니다.

커맨드는 이러한 유연성이 없습니다. 파일 이름이 곧 커맨드 이름입니다.

## 스킬만의 고급 기능

스킬이 단순히 다른 디렉토리에 있는 커맨드가 아닌 이유는 디렉토리 구조가 가능하게 하는 기능들 때문입니다.

### 지원 파일과 번들 리소스

가장 명확한 장점은 여러 파일을 포함할 수 있다는 것입니다. 스킬은 템플릿, 설정, 스크립트, 참조 문서를 모두 하나의 패키지로 포함할 수 있습니다.

실제 예를 들어보겠습니다. 배포 스킬을 번들 리소스와 함께 만들어봅니다:

```
.claude/skills/deploy/
├── SKILL.md
├── deployment-checklist.md
├── rollback-procedure.md
└── scripts/
    ├── health-check.sh
    └── notify-team.sh
```

`SKILL.md`는 `!command` 문법을 사용해 이 파일들을 참조합니다:

```markdown
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---
Before deploying, verify this checklist:
!cat .claude/skills/deploy/deployment-checklist.md

Run the deployment:
1. Build the application
2. Run health checks: !.claude/skills/deploy/scripts/health-check.sh
3. Deploy to production
4. Verify deployment succeeded
5. Notify team: !.claude/skills/deploy/scripts/notify-team.sh

If issues occur, follow rollback procedure:
!cat .claude/skills/deploy/rollback-procedure.md
```

`/deploy`를 호출하면, Claude는 모든 파일 내용과 스크립트 출력이 인라인으로 주입된 완전한 지시사항을 받게 됩니다. 스킬은 필요한 모든 것을 하나의 일관된 패키지로 번들링합니다.

단순한 커맨드는 이것을 할 수 없습니다. 그들은 지원 리소스가 없는 단일 파일입니다.

### 프론트매터 설정의 강력함

스킬은 동작을 제어하는 상세한 프론트매터 설정을 지원합니다:

```markdown
---
name: api-docs
description: Generate API documentation from code
disable-model-invocation: false
allowed-tools: Read, Grep, Bash
context: fork
---
```

각 필드는 특정 목적을 갖습니다:

**name** — 슬래시 커맨드를 정의합니다 (`/api-docs`)

**description** — Claude가 언제 이 스킬을 자동으로 호출할지 결정하는 데 도움을 줍니다. 명확하고 구체적인 설명이 중요합니다.

**disable-model-invocation** — `true`로 설정하면 Claude가 이 스킬을 자동으로 호출할 수 없습니다. 오직 수동 `/command` 호출만 가능합니다. `false`이거나 생략되면, Claude가 설명과 대화가 매칭될 때 자동으로 로드할 수 있습니다.

**allowed-tools** — 이 스킬이 실행될 때 Claude가 사용할 수 있는 도구를 제한합니다. 이 예에서는 Read, Grep, Bash 도구만 사용 가능하며, 파일 편집이나 git 작업 등은 불가능합니다.

**context** — `fork`로 설정하면, 스킬을 메인 대화가 아닌 격리된 서브에이전트 컨텍스트에서 실행합니다. 장시간 실행되거나 방해가 될 수 있는 작업에 유용합니다.

`.claude/commands/`의 커맨드도 이제 같은 프론트매터 필드를 지원합니다. 통합으로 인해 이 기능이 단순 커맨드에도 제공되었습니다. 하지만 스킬은 처음부터 이를 염두에 두고 설계되었습니다.

### 자동 발견과 지능적 호출

스킬은 자동 발견을 위해 만들어졌습니다. Claude는 스킬 설명을 읽고 대화 컨텍스트를 기반으로 관련 스킬을 자동으로 로드합니다.

`.claude/skills/git-commit/SKILL.md`를 만들어봅니다:

```markdown
---
name: git-commit
description: Create semantic commit messages following conventional commits spec. Use when committing code changes.
---
Generate a commit message following conventional commits:
Format: <type>(<scope>): <description>

Types: feat, fix, docs, style, refactor, test, chore

Review the staged changes and create an appropriate commit message.
```

이제 "commit these changes"라고 말하면, Claude는 설명이 매칭된다는 것을 인식하고 자동으로 git-commit 스킬을 로드합니다.

명시적으로 `/git-commit`을 호출하지 않았지만, Claude가 관련성을 판단했습니다.

이러한 자동 발견이 스킬이 설계된 목적입니다. 커맨드도 이제 이것을 할 수 있지만, 스킬 모델이 이를 더 의도적으로 만듭니다.

여기서 `description` 필드가 매우 중요합니다. 스킬이 언제 사용되어야 하는지를 명확하게 서술하는 설명을 작성해야 합니다. 작업에 대해 자연스럽게 말하는 방식과 매칭되는 트리거 문구를 포함시키세요.

### $ARGUMENTS 변수로 파라미터화

스킬과 커맨드 모두 인수를 지원하지만, 스킬이 이를 더 명시적으로 만듭니다:

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
---
Fix GitHub issue $ARGUMENTS following our coding standards:
1. Read the issue description
2. Understand the requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

`/fix-issue 123`을 실행하면, Claude는 "Fix GitHub issue 123 following our coding standards..."를 받게 됩니다.

`$ARGUMENTS` 플레이스홀더가 커맨드 뒤에 입력한 내용으로 대체됩니다. 이를 통해 스킬을 파라미터화할 수 있습니다. 하나의 스킬 정의로 무한한 변형을 처리할 수 있습니다.

스킬에 `$ARGUMENTS`를 포함하지 않았지만 인수와 함께 호출하면, Claude Code가 자동으로 스킬 내용 끝에 `ARGUMENTS: <your input>`을 추가합니다.

### Fork를 통한 서브에이전트 컨텍스트

`context: fork` 프론트매터 옵션은 스킬을 격리된 서브에이전트에서 실행합니다:

```markdown
---
name: analyze-codebase
description: Perform deep analysis of the entire codebase
context: fork
allowed-tools: Read, Grep, Bash
---
Analyze the codebase structure:
1. Identify all entry points
2. Map dependencies
3. Find potential bottlenecks
4. Generate architecture diagram
```

서브에이전트는 자체 대화 기록, 자체 토큰 예산, 자체 도구 제한을 갖습니다. 작업이 완료되면 결과가 메인 세션으로 돌아옵니다.

이러한 격리는 메인 대화를 어지럽히거나 토큰 한도를 초과할 수 있는 작업에 유용합니다. 서브에이전트가 무거운 작업을 수행하면서 주요 컨텍스트를 오염시키지 않습니다.

커맨드도 이제 `context: fork`를 사용할 수 있지만, 이 패턴은 복잡한 다단계 워크플로우를 위해 설계된 스킬과 자연스럽게 어울립니다.

### 가시성과 사용자 호출 가능성

`.claude/skills/`의 스킬은 기본적으로 슬래시 커맨드 메뉴에 표시됩니다. `/`를 입력하면 자동완성에서 볼 수 있습니다.

프론트매터로 메뉴에서 스킬을 숨길 수 있습니다:

```markdown
---
name: internal-helper
description: Internal helper skill not meant for direct invocation
user-invocable: false
---
```

이 스킬은 여전히 존재하고, Claude가 설명과 매칭되면 자동으로 호출할 수 있습니다. 하지만 `/` 자동완성 메뉴에는 나타나지 않습니다. 다른 워크플로우를 지원하지만 독립적인 커맨드가 아닌 헬퍼 스킬에 유용합니다.

커맨드는 항상 사용자 호출 가능합니다. 수동으로 호출되도록 특별히 존재하기 때문입니다.

## 실전 구현과 테스트

이론은 충분합니다. 이제 실제로 무언가를 만들어봅시다. 간단한 커맨드와 완전한 스킬을 모두 만들고, 통합된 시스템이 어떻게 작동하는지 테스트해보겠습니다.

### 첫 번째 커맨드 만들기

일반적인 문제를 확인하는 코드 리뷰 커맨드를 만들어봅니다.

`.claude/commands/quick-review.md`를 생성합니다:

```markdown
---
description: Quick code review focusing on obvious issues
---
Review the current file for:
- Syntax errors or typos
- Missing error handling
- Hardcoded values that should be config
- Console logs left in production code
- Obvious performance issues

Keep it concise - just flag real problems.
```

파일을 저장합니다.

Claude Code에서 `/quick-review`를 입력하고 엔터를 누릅니다. Claude가 현재 컨텍스트에서 이 지시사항을 실행합니다.

이것이 가장 단순한 구현입니다. 파일 하나로 즉시 기능이 추가되었습니다.

### 완전한 스킬로 변환하기

이제 이 커맨드를 지원 리소스가 있는 스킬로 변환해봅시다.

디렉토리 구조를 만듭니다:

```
.claude/skills/code-review/
├── SKILL.md
├── review-checklist.md
└── severity-guide.md
```

`.claude/skills/code-review/SKILL.md`를 생성합니다:

```markdown
---
name: code-review
description: Comprehensive code review with severity classification. Use when reviewing code changes or pull requests.
allowed-tools: Read, Grep, Glob
---

Perform a comprehensive code review using our standards:
!cat .claude/skills/code-review/review-checklist.md

Classify issues by severity:
!cat .claude/skills/code-review/severity-guide.md

Provide specific line numbers and actionable suggestions for each issue found.
```

`.claude/skills/code-review/review-checklist.md`를 생성합니다:

```markdown
## Code Review Checklist

### Correctness
- Logic errors or edge cases
- Null pointer risks
- Off-by-one errors
- Race conditions

### Security
- Input validation
- SQL injection risks
- XSS vulnerabilities
- Authentication/authorization issues

### Performance
- N+1 queries
- Unnecessary loops
- Memory leaks
- Inefficient algorithms

### Maintainability
- Code duplication
- Complex conditional logic
- Missing comments for non-obvious code
- Inconsistent naming
```

`.claude/skills/code-review/severity-guide.md`를 생성합니다:

```markdown
## Issue Severity Levels

**Critical**: Security vulnerabilities, data loss risks, crashes
**High**: Logic errors, performance problems, broken functionality
**Medium**: Code quality issues, minor bugs, edge cases
**Low**: Style inconsistencies, missing comments, suggestions
```

이제 동일한 `/code-review` 커맨드를 갖게 되었지만, 구조화된 지원 콘텐츠가 함께 제공됩니다. 스킬이 모든 것을 하나로 번들링합니다.

### 자동 발견 테스트하기

스킬의 설명은 "Comprehensive code review with severity classification. Use when reviewing code changes or pull requests."입니다.

명시적으로 커맨드를 호출하지 않고 Claude Code에서 대화를 시작합니다:

> "I need to review this pull request before merging"

Claude는 "review"와 "pull request"가 스킬 설명과 매칭된다는 것을 인식하고 자동으로 로드해야 합니다.

대화에서 스킬이 활성화되는 것을 볼 수 있습니다.

이를 단순 커맨드 접근법과 비교해보세요. `.claude/commands/quick-review.md`도 자동 발견될 수 있지만, 설명을 수동으로 추가해야 합니다.

### 파라미터화 추가하기

스킬이 인수를 받도록 만들어봅시다. `.claude/skills/code-review/SKILL.md`를 업데이트합니다:

```markdown
---
name: code-review
description: Comprehensive code review with severity classification
allowed-tools: Read, Grep, Glob
---
Review target: $ARGUMENTS

Perform a comprehensive code review using our standards:
!cat .claude/skills/code-review/review-checklist.md

Classify issues by severity:
!cat .claude/skills/code-review/severity-guide.md

Focus the review on: $ARGUMENTS

Provide specific line numbers and actionable suggestions.
```

이제 다음과 같이 호출합니다:

```
/code-review src/api/users.ts
```

Claude는 `$ARGUMENTS`가 `src/api/users.ts`로 대체된 스킬 지시사항을 받습니다. 스킬이 컨텍스트를 인식하게 됩니다.

### Fork된 스킬 만들기

일부 작업은 격리되어 실행되어야 합니다. 서브에이전트에서 실행되는 코드베이스 분석 스킬을 만들어봅시다:

`.claude/skills/analyze/SKILL.md`를 생성합니다:

```markdown
---
name: analyze
description: Deep analysis of codebase structure and dependencies
context: fork
allowed-tools: Read, Grep, Bash
---

Analyze the codebase comprehensively:
1. Map all entry points (main files, exports)
2. Identify key dependencies
3. Find circular dependencies
4. List dead code (unused files)
5. Measure code complexity hotspots

Use grep, find, and file reading to gather data.
Provide a structured report with specific findings.
```

이 스킬은 격리된 서브에이전트 컨텍스트에서 실행됩니다.

`/analyze`를 호출하면, Claude가 서브에이전트를 생성하고 분석을 수행한 후 결과를 메인 세션으로 반환합니다.

`allowed-tools: Read, Grep, Bash` 제한은 서브에이전트가 파일을 수정하지 못하도록 합니다. 읽기와 분석만 가능합니다.

### 도구 제한 테스트하기

읽기 전용 문서화 스킬을 만듭니다:

```markdown
---
name: explain-code
description: Explain how code works with diagrams and analogies
allowed-tools: Read, Grep
---

When explaining code, include:
1. High-level overview in plain language
2. ASCII diagram showing flow or structure
3. Step-by-step walkthrough
4. Common gotchas or misconceptions
```

`allowed-tools: Read, Grep` 제한은 Claude가 파일을 읽고 검색할 수 있지만, 이 스킬이 활성화된 동안 쓰기, 편집, bash 커맨드 실행을 할 수 없다는 의미입니다.

이 스킬이 실행되는 동안 Claude에게 코드를 수정하라고 해보세요. 불가능합니다. 도구 제한이 읽기 전용 모드를 강제합니다.

## 실전 활용 팁과 모범 사례

통합된 시스템을 효과적으로 사용하기 위한 몇 가지 실전 팁을 정리했습니다.

### 언제 커맨드를 사용할까

단순한 일회성 작업이나 빠른 프로토타이핑에는 `.claude/commands/`를 사용하세요. 몇 줄의 지시사항만 필요하고, 추가 리소스가 없으며, 빠르게 만들어 테스트하고 싶을 때 이상적입니다.

예를 들어 빠른 포맷 체크, 간단한 코드 생성 템플릿, 기본적인 분석 작업 같은 것들입니다. 파일 하나로 충분하다면 커맨드가 답입니다.

### 언제 스킬을 사용할까

복잡한 워크플로우, 여러 단계가 있는 프로세스, 지원 파일이 필요한 경우에는 `.claude/skills/`를 사용하세요. 체크리스트, 템플릿, 설정 파일, 스크립트를 번들링해야 한다면 스킬 구조가 필수입니다.

또한 팀 간에 공유하거나, 여러 프로젝트에서 재사용하거나, 다른 AI 도구로 이식할 계획이 있다면 스킬로 만드는 것이 좋습니다. Agent Skills 표준을 따르기 때문에 이식성이 보장됩니다.

### 설명 작성의 중요성

`description` 필드는 자동 발견의 핵심입니다. 명확하고 구체적인 설명을 작성하세요. 사용자가 자연스럽게 말하는 방식의 트리거 문구를 포함하세요.

나쁜 예:
```markdown
description: Reviews code
```

좋은 예:
```markdown
description: Comprehensive code review with security analysis. Use when reviewing pull requests, checking for vulnerabilities, or analyzing code quality before merge.
```

좋은 설명은 "when", "for", "use when" 같은 문구를 포함해 명확한 트리거 조건을 제시합니다.

### 도구 제한 활용하기

`allowed-tools`를 사용해 스킬이 할 수 있는 것을 제한하세요. 분석만 하는 스킬이라면 `Read, Grep, Bash`만 허용하세요. 문서를 생성하는 스킬이라면 `Read, Write`를 허용하세요.

이는 안전성을 높이고, 스킬의 목적을 명확하게 하며, 예상치 못한 부작용을 방지합니다.

### Fork를 언제 사용할까

`context: fork`는 다음과 같은 경우에 사용하세요:

- 전체 코드베이스 분석처럼 토큰을 많이 소비하는 작업
- 여러 파일을 읽고 처리하는 장시간 작업
- 메인 대화의 흐름을 방해할 수 있는 작업
- 격리된 환경이 필요한 실험적인 작업

서브에이전트는 작업을 완료한 후 요약된 결과만 메인 세션으로 반환하므로, 메인 컨텍스트가 깔끔하게 유지됩니다.

### 핫 리로드 활용하기

버전 2.1.0 이후 핫 리로드가 지원되므로, 커맨드나 스킬을 수정하면 즉시 반영됩니다. 이를 활용해 반복적으로 개선하세요.

스킬을 만들고, 테스트하고, 수정하고, 다시 테스트하는 사이클을 빠르게 반복할 수 있습니다. Claude Code 세션을 유지한 채로 실시간으로 개선 작업을 진행하세요.

### 버전 관리

스킬과 커맨드를 Git으로 관리하세요. `.claude/` 디렉토리 전체를 리포지토리에 포함시키면 팀원들과 공유하고, 변경 이력을 추적하고, 프로젝트 간에 이식할 수 있습니다.

특히 복잡한 스킬은 버전 관리가 중요합니다. 시간이 지나면서 개선하고, 버그를 수정하고, 새로운 기능을 추가하게 되는데, 이력을 추적하는 것이 매우 유용합니다.

## 마이그레이션 전략

기존에 슬래시 커맨드나 스킬을 사용하고 있었다면, 통합 이후 마이그레이션이 필요할까요? 답은 "필요 없다"입니다.

### 기존 커맨드는 그대로

`.claude/commands/`에 있는 모든 커맨드는 업데이트 없이 계속 작동합니다. 아무것도 수정할 필요가 없습니다. 단순히 계속 사용하면 됩니다.

원한다면 점진적으로 복잡한 커맨드를 스킬로 변환할 수 있습니다. 하지만 강제사항은 아닙니다. 단순한 커맨드는 단순하게 유지하는 것도 완벽히 타당한 선택입니다.

### 기존 스킬도 그대로

`.claude/skills/`의 모든 스킬도 동일하게 작동합니다. 프론트매터 변경이나 구조 수정이 필요하지 않습니다.

스킬은 이제 명시적 호출도 가능하다는 추가 이점을 얻었습니다. 자동 발견만 가능했던 스킬을 이제 `/skill-name`으로 직접 호출할 수도 있습니다.

### 점진적 개선 전략

기존 워크플로우를 유지하면서 점진적으로 개선하는 전략을 권장합니다:

1. 현재 커맨드와 스킬을 그대로 사용하세요
2. 새로운 기능이 필요할 때, 단순하면 커맨드로, 복잡하면 스킬로 만드세요
3. 시간이 지나면서 자연스럽게 복잡해진 커맨드를 스킬로 전환하세요
4. 자동 발견이 필요한 커맨드에 `description`을 추가하세요
5. 도구 제한이나 fork가 필요한 경우 프론트매터를 추가하세요

이러한 접근 방식은 중단 없이 점진적으로 시스템을 개선할 수 있게 합니다.

## 고급 사용 사례

통합된 시스템의 진정한 힘은 고급 워크플로우에서 드러납니다.

### 계층적 스킬 구조

스킬 내에서 다른 스킬을 호출할 수 있습니다. 이를 통해 계층적 워크플로우를 구축할 수 있습니다.

예를 들어 메인 배포 스킬이 테스트 스킬, 빌드 스킬, 알림 스킬을 순차적으로 호출하는 구조를 만들 수 있습니다.

### 조건부 스킬 로딩

설명을 신중하게 작성하면 컨텍스트에 따라 다른 스킬이 로드되도록 할 수 있습니다. 프론트엔드 작업을 할 때는 UI 관련 스킬이, 백엔드 작업을 할 때는 API 관련 스킬이 자동으로 활성화되도록 만들 수 있습니다.

### 팀 워크플로우 표준화

스킬을 사용해 팀 전체의 워크플로우를 표준화할 수 있습니다. 코드 리뷰 기준, 커밋 메시지 형식, 문서화 템플릿 등을 스킬로 만들어 공유하면, 모든 팀원이 동일한 표준을 따르게 됩니다.

`.claude/` 디렉토리를 Git으로 관리하면 이러한 표준이 프로젝트의 일부가 되고, 새로운 팀원도 즉시 활용할 수 있습니다.

### 프로젝트별 커스터마이징

각 프로젝트마다 고유한 요구사항이 있습니다. 프로젝트 루트의 `.claude/` 디렉토리에 프로젝트 특화 스킬을 만들어 두면, 해당 프로젝트에서만 사용 가능한 커스텀 워크플로우를 구축할 수 있습니다.

### 외부 도구 통합

`!command` 문법을 사용하면 외부 도구와 통합할 수 있습니다. 린터, 테스트 러너, 빌드 도구의 출력을 스킬에 주입하여 Claude가 이를 분석하고 조치를 취하도록 만들 수 있습니다.

## 성능과 최적화

스킬과 커맨드를 효과적으로 사용하려면 성능 고려사항도 이해해야 합니다.

### 토큰 사용 최적화

스킬이 많은 파일을 `!cat`으로 주입한다면 토큰 사용량이 급증할 수 있습니다. 필요한 정보만 포함하도록 지원 파일을 간결하게 유지하세요.

긴 참조 문서가 필요하다면, 전체를 주입하는 대신 관련 섹션만 추출하는 스크립트를 사용하는 것을 고려하세요.

### 자동 발견 균형 잡기

너무 많은 스킬이 자동 발견되도록 설정하면 컨텍스트가 혼잡해질 수 있습니다. 정말 자동으로 로드되어야 하는 스킬만 `disable-model-invocation: false`로 설정하세요.

자주 사용하지만 자동 발견이 필요하지 않은 스킬은 `disable-model-invocation: true`로 설정하고 명시적으로 호출하세요.

### Fork 사용의 균형

Fork는 격리를 제공하지만 오버헤드도 있습니다. 서브에이전트 생성과 결과 통합에 시간이 걸립니다. 정말 격리가 필요한 경우에만 사용하세요.

간단한 작업은 메인 컨텍스트에서 실행하는 것이 더 효율적입니다.

## 문제 해결과 디버깅

스킬과 커맨드를 사용하다 보면 예상치 못한 문제를 만날 수 있습니다.

### 커맨드가 인식되지 않을 때

파일 이름과 위치를 확인하세요. `.claude/commands/` 또는 `.claude/skills/skill-name/` 경로가 정확한지 확인합니다.

SKILL.md의 `name` 필드가 올바른지 확인하세요. 오타가 있으면 커맨드가 예상과 다른 이름으로 등록됩니다.

핫 리로드가 작동하지 않으면 Claude Code 세션을 재시작해보세요.

### 자동 발견이 작동하지 않을 때

`description` 필드가 충분히 구체적이고 명확한지 확인하세요. 모호한 설명은 자동 발견을 어렵게 만듭니다.

`disable-model-invocation`이 `true`로 설정되어 있지 않은지 확인하세요. 이 설정이 있으면 자동 호출이 불가능합니다.

설명에 실제로 사용하는 용어와 문구가 포함되어 있는지 확인하세요. "deploy"라고 말하는데 설명에 "deployment"만 있으면 매칭이 어려울 수 있습니다.

### !command 문법이 작동하지 않을 때

명령어 경로가 정확한지 확인하세요. 상대 경로를 사용할 때는 `.claude/skills/` 디렉토리를 기준으로 합니다.

스크립트에 실행 권한이 있는지 확인하세요. `chmod +x script.sh`로 권한을 부여해야 할 수 있습니다.

명령어가 실제로 실행 가능한지 터미널에서 직접 테스트해보세요.

### 도구 제한 문제

`allowed-tools`가 너무 제한적이면 스킬이 필요한 작업을 수행하지 못할 수 있습니다. 스킬이 실제로 필요로 하는 도구를 모두 포함했는지 확인하세요.

반대로 너무 관대한 제한은 안전성 문제를 야기할 수 있습니다. 최소 권한 원칙을 따르세요.

## 커뮤니티와 생태계

Claude Code 스킬은 오픈 스탠다드를 따르기 때문에 커뮤니티가 형성되고 있습니다.

### 스킬 공유

GitHub에서 Claude Code 스킬을 공유하는 리포지토리들이 생겨나고 있습니다. 다른 개발자들이 만든 스킬을 찾아보고, 여러분의 스킬도 공유할 수 있습니다.

스킬을 공유할 때는 README를 포함해 사용 방법과 요구사항을 명확히 설명하세요.

### 이식성의 가치

Agent Skills 표준 덕분에 한 번 만든 스킬을 여러 도구에서 사용할 수 있습니다. Claude Code, claude.ai, Claude Desktop 모두 같은 스킬을 지원합니다.

앞으로 더 많은 AI 도구들이 이 표준을 채택할 것으로 예상됩니다. 지금 만드는 스킬은 미래에도 유용할 것입니다.

### 발전하는 표준

Agent Skills 표준은 계속 발전하고 있습니다. 새로운 기능과 능력이 추가되면, 여러분의 스킬도 자동으로 이러한 개선사항의 혜택을 받게 됩니다.

커뮤니티의 피드백과 사용 사례가 표준의 발전을 주도하고 있습니다.

## 앞으로의 방향

슬래시 커맨드와 스킬의 통합은 시작에 불과합니다. Anthropic은 계속해서 Claude Code를 개선하고 있습니다.

### 예상되는 발전

더 정교한 자동 발견 메커니즘, 스킬 간 의존성 관리, 향상된 디버깅 도구, 시각적 스킬 편집기 등이 개발될 가능성이 있습니다.

커뮤니티의 피드백을 바탕으로 새로운 프론트매터 옵션과 기능이 추가될 것입니다.

### 학습 리소스

Anthropic은 곧 Claude Code 마스터클래스 코스를 출시할 예정입니다. 이 코스는 복잡한 개발 작업을 위한 다중 에이전트 조정 워크플로우를 다룹니다.

고급 서브에이전트 패턴, 프로덕션 준비 훅 설정, MCP 서버 통합, 팀 협업 전략, 엔터프라이즈 배포 패턴, 실제 사례 연구 등이 포함됩니다.

### 커뮤니티 참여

Claude Code는 빠르게 발전하고 있고, 변화를 따라잡기가 어려울 수 있습니다. 커뮤니티에 참여하고, 경험을 공유하고, 다른 개발자들로부터 배우세요.

여러분의 사용 사례와 스킬은 다른 사람들에게 영감을 줄 수 있고, 전체 생태계를 풍부하게 만듭니다.

## 결론

Claude Code 버전 2.1.3의 슬래시 커맨드와 스킬 통합은 겉보기에는 작은 변화지만, 일상적인 워크플로우를 크게 개선하는 업데이트입니다. 두 개의 유사한 기능 중 하나를 선택해야 하는 정신적 부담이 사라졌습니다. 이제 두 가지 조직화 접근법을 가진 하나의 시스템이 있을 뿐입니다.

완벽한 하위 호환성은 기존의 모든 커맨드와 스킬이 그대로 작동한다는 것을 의미합니다. 마이그레이션 작업 없이 즉시 새로운 기능의 이점을 누릴 수 있습니다.

통합된 시스템을 사용하면서 얻는 주요 이점은 명확성입니다. 무언가를 커맨드로 만들어야 할지 스킬로 만들어야 할지 고민할 필요가 없습니다. 구조가 선택을 지시합니다. 간단하면 `.claude/commands/`에 파일 하나를 만들고, 복잡하면 `.claude/skills/`에 디렉토리 구조를 만들면 됩니다.

Agent Skills 오픈 스탠다드를 따르기 때문에, 한 번 작성한 스킬은 여러 AI 도구에서 사용할 수 있습니다. 이러한 이식성은 여러분의 투자를 더욱 가치 있게 만듭니다.

슬래시 커맨드와 스킬은 Claude Code를 자신의 워크플로우에 맞게 커스터마이징할 수 있는 강력한 도구입니다. 통합으로 인해 이제 더 직관적이고, 더 유연하며, 더 강력해졌습니다. 이 기능을 적극적으로 활용해 자신만의 AI 지원 개발 환경을 구축하세요.

---

**작성 일자: 2026-01-31**
