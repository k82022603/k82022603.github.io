---
title: "GSD (Get Shit Done) 프레임워크 개발자 가이드"
date: 2026-01-11 12:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  Guide,  Claude,  claude-code,  context-rot,  context-engineering,  GSD,  Framework,  claude-code-plugin,  Claude.write]
---

npx get-shit-done-cc --global
# 또는
npx get-shit-done-cc -g

# 로컬 설치
npx get-shit-done-cc --local
# 또는
npx get-shit-done-cc -l
```

#### 방법 3: GitHub에서 직접 클론 (개발자용)

GSD의 소스 코드를 수정하거나 기여하고 싶다면 GitHub에서 직접 클론할 수 있습니다:

```bash
git clone https://github.com/glittercowboy/get-shit-done.git
cd get-shit-done
node bin/install.js --local
```

이 방법은 `./.claude/` 디렉토리에 설치되며, 기여하기 전에 수정 사항을 테스트하는 데 유용합니다.

### 설치 확인

설치가 완료되었는지 확인하려면 Claude Code를 실행하고 다음 명령어를 입력합니다:

```bash
/gsd:help
```

GSD의 모든 명령어 목록과 사용법이 표시되면 설치가 성공적으로 완료된 것입니다.

### 권한 설정 (중요)

GSD는 원활한 자동화를 위해 설계되었습니다. Git 커밋, 날짜 확인 등의 명령을 반복적으로 승인하는 것은 GSD의 목적에 어긋납니다. 따라서 다음 두 가지 방법 중 하나를 선택하여 권한을 설정하는 것을 강력히 권장합니다.

#### 옵션 1: 전역 권한 스킵 (가장 간단)

Claude Code를 실행할 때 다음 플래그를 사용합니다:

```bash
claude --dangerously-skip-permissions
```

이 방법은 모든 명령어를 자동으로 승인하므로 GSD가 중단 없이 작동합니다.

#### 옵션 2: 프로젝트별 권한 설정

전역 권한 스킵이 부담스럽다면, 프로젝트의 `.claude/settings.json` 파일에 다음 설정을 추가합니다:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(sort:*)",
      "Bash(grep:*)",
      "Bash(tr:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git tag:*)"
    ]
  }
}
```

이 설정은 GSD가 사용하는 모든 명령어를 자동으로 승인하면서도, 다른 위험한 명령어는 여전히 수동 승인을 요구합니다.

### 최신 버전으로 업데이트

GSD를 최신 버전으로 업데이트하려면 다음 명령어를 실행합니다:

```bash
npx get-shit-done-cc@latest
```

기존 설치를 덮어쓰고 최신 기능과 버그 수정 사항을 적용합니다.

---

## 핵심 개념과 작동 원리

GSD의 효과를 제대로 이해하려면 그 작동 원리와 설계 철학을 이해해야 합니다. 이 섹션에서는 GSD를 특별하게 만드는 핵심 메커니즘을 상세히 설명합니다.

### 컨텍스트 부패(Context Rot) 문제

대규모 언어 모델(LLM)은 대화가 길어질수록 성능이 저하됩니다. 이를 "컨텍스트 부패"라고 합니다. Claude가 대화 중에 "컨텍스트 제한으로 인해 더 간결하게 작성하겠습니다"라고 말하는 것을 본 적이 있다면, 바로 이것입니다. 그 "간결함"은 실제로는 품질 저하를 의미합니다.

컨텍스트 창이 채워질수록:
- AI는 이전 지시사항을 잊거나 무시하기 시작합니다
- 일관성이 떨어지고 모순된 결정을 내립니다
- 코드 품질이 저하되고 "코너를 깎기" 시작합니다
- 검증 단계를 건너뛰거나 단순화합니다

### GSD의 해결책: 서브 에이전트 아키텍처

GSD는 각 작업을 별도의 서브 에이전트에서 실행함으로써 이 문제를 근본적으로 해결합니다. 각 서브 에이전트는:

- **완전히 새로운 컨텍스트 창**: 약 20만 토큰의 깨끗한 상태로 시작
- **축적된 쓰레기 제로**: 이전 작업의 잔여물이 전혀 없음
- **최대 품질 유지**: 처음부터 끝까지 일관된 성능
- **독립적 실행**: 각 작업이 다른 작업의 영향을 받지 않음

이는 마치 매번 새로운 전문가를 고용하는 것과 같습니다. 피곤하거나 산만한 개발자가 아니라, 항상 집중력이 최고조인 상태의 전문가입니다.

### 계층적 작업 구조

GSD는 프로젝트를 다음과 같은 계층 구조로 세분화합니다:
```
Milestone (마일스톤)
└── Phase (단계)
    └── Plan (계획) (2-3개의 원자적 작업)
        └── Atomic Task (원자적 작업) (독립적으로 실행 가능한 최소 단위)
```

예를 들어, "사용자 인증 시스템 구축"이라는 마일스톤이 있다면:

**Milestone**: 사용자 인증 시스템 v1.0
- **Phase 1**: 데이터베이스 설계
  - **Plan 1-1**: 사용자 테이블 생성
    - Task: users 테이블 마이그레이션 작성
    - Task: 인덱스 및 제약 조건 추가
    - Task: 시드 데이터 생성
- **Phase 2**: 인증 API 구현
  - **Plan 2-1**: 회원가입 엔드포인트
    - Task: 이메일 유효성 검증 로직
    - Task: 비밀번호 해싱 구현
    - Task: 사용자 생성 API 엔드포인트
  - **Plan 2-2**: 로그인 엔드포인트
    - Task: JWT 토큰 생성 유틸리티
    - Task: 자격 증명 검증 로직
    - Task: 로그인 API 엔드포인트

각 원자적 작업은 독립적으로 실행 가능하며, 명확한 입력과 출력, 그리고 검증 기준을 갖습니다.

### XML 기반 프롬프트 포맷팅

GSD는 모든 작업을 구조화된 XML 형식으로 변환합니다. 이는 Claude가 가장 잘 이해하고 실행할 수 있는 형식입니다:

```xml
<task type="auto">
  <n>로그인 엔드포인트 생성</n>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    JWT를 위해 jose 라이브러리 사용 (jsonwebtoken은 CommonJS 문제 있음).
    users 테이블에 대해 자격 증명 검증.
    성공 시 httpOnly 쿠키 반환.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login이 200과 Set-Cookie 반환</verify>
  <done>유효한 자격 증명은 쿠키 반환, 무효한 자격 증명은 401 반환</done>
</task>
```

이 구조는:
- **명확한 지시사항**: 추측이 필요 없음
- **내장된 검증**: 작업 완료를 확인하는 방법이 명시됨
- **일관된 형식**: Claude가 항상 동일한 구조를 기대할 수 있음
- **추적 가능성**: 각 작업이 문서화되고 버전 관리됨

### 핵심 문서 시스템

GSD는 프로젝트 전반에 걸쳐 일관성을 유지하기 위해 여러 핵심 문서를 사용합니다:

#### PROJECT.md
프로젝트의 비전과 요구사항을 담은 단일 진실 공급원(Single Source of Truth)입니다. 모든 작업에 자동으로 로드되어 AI가 항상 프로젝트의 목표와 제약 사항을 인식하도록 합니다.

**내용 예시:**
- 프로젝트 개요 및 목표
- 타겟 사용자
- 핵심 기능 목록
- 기술 스택 및 제약 사항
- 보안 요구사항
- 성능 기준

**크기 제한**: 일반적으로 300-500줄 이내로 유지하여 컨텍스트 부패 방지

#### ROADMAP.md
프로젝트를 단계별로 분해한 전술적 문서입니다. 어디로 가는지, 무엇이 완료되었는지를 보여줍니다.

**내용 예시:**
- 마일스톤 목록
- 각 마일스톤의 단계(Phase) 목록
- 단계별 목표 및 산출물
- 의존성 관계
- 예상 완료 시간

**특징**: 프로젝트가 진행됨에 따라 동적으로 업데이트됨

#### STATE.md
현재 프로젝트의 상태를 추적하는 살아있는 메모리입니다. 세션 간에 지속되며, Claude가 중단된 곳에서 정확히 다시 시작할 수 있도록 합니다.

**내용 예시:**
- 현재 진행 중인 단계 및 작업
- 완료된 작업 목록
- 결정 사항 및 근거
- 발견된 문제점 및 해결책
- 성능 지표 (예: 커밋 수, 테스트 통과율)
- 다음 단계 계획

**중요성**: 며칠 후 프로젝트로 돌아와도 정확히 어디에 있었는지 알 수 있음

#### PLAN.md
각 단계의 원자적 작업 계획을 담은 문서입니다. XML 형식으로 구조화되어 있으며, 서브 에이전트가 실행할 정확한 지시사항을 포함합니다.

**내용**: 2-3개의 원자적 작업, 각각 파일 경로, 수행할 작업, 검증 방법, 완료 기준 포함

#### SUMMARY.md
각 계획 실행 후 생성되는 요약 문서입니다. 무엇이 변경되었는지, 어떤 문제가 있었는지, 어떻게 해결했는지를 기록하여 Git 히스토리에 커밋됩니다.

**활용**: 향후 작업에서 참고 자료로 사용되며, 프로젝트 전체의 진화 과정을 추적할 수 있음

#### ISSUES.md
나중에 처리할 개선 사항, 알려진 버그, 기술 부채를 추적하는 문서입니다. 세션 간에 지속되며, `/gsd:consider-issues` 명령으로 검토할 수 있습니다.

**장점**: 현재 작업에 집중하면서도 향후 개선 사항을 잃어버리지 않음

### 크기 제한 및 품질 관리

GSD는 Claude의 품질이 저하되는 지점을 기반으로 문서 크기를 제한합니다:

- **PROJECT.md**: 최대 500줄
- **ROADMAP.md**: 전체 로드맵을 포괄하되, 단계당 간결하게
- **PLAN.md**: 최대 3개의 작업
- **STATE.md**: 핵심 정보만 유지, 주기적으로 정리

이러한 제한 아래에서 유지하면 일관되게 우수한 품질을 얻을 수 있습니다.

### 원자적 Git 커밋

각 작업은 완료 즉시 자동으로 커밋됩니다. 하나의 계획(2-3개 작업)은 일반적으로 2-4개의 커밋을 생성합니다:

```bash
abc123f docs(08-02): 사용자 등록 계획 완료
def456g feat(08-02): 이메일 확인 흐름 추가
hij789k feat(08-02): 비밀번호 해싱 구현
lmn012o feat(08-02): 등록 엔드포인트 생성
```

**이점:**
- `git bisect`로 정확한 실패 지점 찾기
- 각 작업을 독립적으로 되돌리기 가능
- Claude가 향후 세션에서 명확한 히스토리 참조 가능
- AI 자동화 워크플로우에서 더 나은 관찰 가능성

모든 커밋은 외과적이고, 추적 가능하며, 의미 있습니다.

### 모듈식 설계

GSD는 유연성을 위해 설계되었습니다:

- **단계 추가**: 현재 마일스톤에 새로운 기능 추가
- **단계 삽입**: 긴급 작업을 단계 사이에 끼워넣기
- **마일스톤 완료**: 버전 1.0을 출시하고 2.0 준비
- **계획 조정**: 모든 것을 재구축하지 않고 수정

프로젝트는 결코 고정되지 않습니다. 시스템이 적응합니다.

---

## 신규 프로젝트 시작하기

완전히 새로운 프로젝트를 시작하는 경우(그린필드 개발), GSD는 아이디어 단계부터 배포까지 전체 프로세스를 안내합니다. 이 섹션에서는 첫 번째 프로젝트를 단계별로 진행하는 방법을 설명합니다.

### 1단계: 프로젝트 디렉토리 준비

먼저 새 프로젝트를 위한 디렉토리를 생성하고 Git 저장소로 초기화합니다:

```bash
# 프로젝트 디렉토리 생성
mkdir my-awesome-app
cd my-awesome-app

# Git 저장소 초기화 (GSD의 자동 커밋 기능에 필수)
git init

# 선택 사항: 원격 저장소 연결
git remote add origin https://github.com/yourusername/my-awesome-app.git
```

Git 초기화는 필수입니다. GSD는 각 작업을 자동으로 커밋하므로 Git 저장소가 없으면 작동하지 않습니다.

### 2단계: 프로젝트 초기화

Claude Code를 실행하고 GSD 프로젝트를 초기화합니다:

```bash
# Claude Code 실행 (권한 자동 승인 권장)
claude --dangerously-skip-permissions

# Claude Code 내에서 GSD 프로젝트 초기화
/gsd:new-project
```

`/gsd:new-project` 명령을 실행하면 GSD가 대화형으로 프로젝트에 대한 질문을 시작합니다. 이 단계에서 시스템은 프로젝트의 모든 측면을 이해하려고 노력합니다.

### 3단계: 아이디어 추출 과정

GSD는 단순히 "무엇을 만들 것인가?"라고 묻지 않습니다. 체계적으로 정보를 추출합니다:

**초기 질문 예시:**
- "어떤 애플리케이션을 만들고 싶으신가요?"
- "주요 타겟 사용자는 누구인가요?"
- "핵심 기능 3가지는 무엇인가요?"

**기술적 질문:**
- "선호하는 프로그래밍 언어나 프레임워크가 있나요?"
- "데이터베이스는 어떤 것을 사용하고 싶으신가요?"
- "배포 환경은 어디인가요? (Vercel, AWS, 자체 서버 등)"

**제약 사항 및 요구사항:**
- "특별히 피해야 할 라이브러리나 패턴이 있나요?"
- "성능 요구사항이 있나요? (예: 응답 시간, 동시 사용자 수)"
- "보안이나 규정 준수 요구사항이 있나요?"

**예외 상황 및 엣지 케이스:**
- "사용자 입력 검증은 어떻게 처리할까요?"
- "오류 발생 시 어떻게 해야 하나요?"
- "오프라인 상태는 어떻게 처리할까요?"

### 실전 예시: 할 일 관리 앱

실제 대화 예시를 보겠습니다:

```
GSD: 어떤 애플리케이션을 만들고 싶으신가요?

사용자: Next.js 15와 Tailwind CSS를 사용한 개인용 할 일 관리 앱을 만들고 싶어요. 
데이터베이스는 Supabase를 사용하고, 사용자는 Google 로그인을 할 수 있어야 해요. 
핵심 기능은 '우선순위 기반 태스크 정렬'과 '매일 정해진 시간에 리마인더 발송'이에요.

GSD: 좋습니다. 몇 가지 추가 질문이 있습니다:

1. 리마인더는 어떤 방식으로 발송되나요? (이메일, 푸시 알림, 브라우저 알림?)
2. 태스크에 첨부 파일이나 이미지를 추가할 수 있어야 하나요?
3. 태스크를 다른 사람과 공유하거나 협업할 수 있어야 하나요?
4. 모바일 반응형 디자인이 중요한가요?
5. 다크 모드를 지원해야 하나요?

사용자: 1. 브라우저 알림과 이메일 둘 다 지원하면 좋겠어요.
2. 아니요, 텍스트만으로 충분해요.
3. 처음에는 개인용으로만 사용하고, 나중에 공유 기능을 추가할 수 있으면 좋겠어요.
4. 네, 모바일에서도 잘 작동해야 해요.
5. 네, 다크 모드는 필수예요.

GSD: 완벽합니다. Supabase 인증 설정이나 SMTP 서버 같은 외부 서비스 설정이 
이미 되어 있나요, 아니면 프로젝트 내에서 설정해야 하나요?

사용자: Supabase 프로젝트는 이미 생성했고, Google OAuth 앱도 등록했어요. 
이메일은 Resend를 사용하려고 하는데, 아직 설정은 안 했어요.

GSD: 알겠습니다. 환경 변수로 관리할 민감한 정보가 있을 텐데요:
- Supabase URL, Anon Key
- Google OAuth Client ID, Secret
- Resend API Key
이것들을 .env.local에서 관리하는 것으로 진행할까요?

사용자: 네, 맞아요.

GSD: 마지막으로, 특별히 피해야 할 패턴이나 라이브러리가 있나요? 
예를 들어, "Redux는 사용하지 않기", "타입스크립트 strict 모드 사용" 같은 것들이요.

사용자: TypeScript strict 모드를 사용하고, 상태 관리는 Zustand를 쓰고 싶어요. 
그리고 모든 API는 Next.js의 App Router와 Server Actions를 사용했으면 좋겠어요.
```

이런 식으로 대화가 진행되면서 GSD는 프로젝트에 대한 완전한 그림을 그립니다.

### 4단계: PROJECT.md 생성

모든 질문이 끝나면 GSD는 수집한 정보를 바탕으로 `PROJECT.md` 파일을 생성합니다:

```markdown
# 할 일 관리 앱 - 프로젝트 사양

## 개요
개인용 할 일 관리 애플리케이션. 우선순위 기반 정렬과 시간 기반 리마인더를 
핵심 기능으로 하는 현대적인 웹 앱.

## 기술 스택
- **프론트엔드**: Next.js 15 (App Router), React, TypeScript (strict mode)
- **스타일링**: Tailwind CSS, 다크 모드 지원
- **상태 관리**: Zustand
- **백엔드**: Next.js Server Actions, Supabase (PostgreSQL)
- **인증**: Supabase Auth + Google OAuth
- **알림**: 브라우저 Notification API + Resend (이메일)

## 핵심 기능
1. 우선순위 기반 태스크 정렬 (높음/중간/낮음)
2. 시간 기반 리마인더 (브라우저 알림 + 이메일)
3. Google 소셜 로그인
4. 완전한 모바일 반응형
5. 다크/라이트 모드 전환

## 제약 사항
- Redux 사용 금지
- 모든 API는 Server Actions로 구현
- 환경 변수는 .env.local에서 관리
- TypeScript strict 모드 필수

## 향후 확장 계획
- 태스크 공유 및 협업 기능 (Phase 2)
```

이 문서는 프로젝트의 북극성 역할을 합니다. 모든 작업에서 참조되어 AI가 항상 올바른 방향으로 나아가도록 합니다.

### 5단계: 로드맵 생성

`PROJECT.md`가 준비되면 다음 단계로 로드맵을 생성합니다:

```
/gsd:create-roadmap
```

GSD는 프로젝트를 여러 단계(Phase)로 분해합니다. 할 일 관리 앱 예시의 로드맵은 다음과 같을 수 있습니다:

```markdown
# 로드맵: 할 일 관리 앱 v1.0

## Phase 1: 프로젝트 설정 및 인증
목표: Next.js 앱 스캐폴딩, Supabase 연결, Google OAuth 구현

## Phase 2: 데이터베이스 스키마
목표: Supabase에 tasks 테이블 생성, 관계 설정

## Phase 3: 태스크 CRUD
목표: 태스크 생성/읽기/수정/삭제 기능 구현

## Phase 4: 우선순위 정렬
목표: 태스크를 우선순위별로 정렬하는 UI 및 로직

## Phase 5: 리마인더 시스템
목표: 브라우저 알림 및 이메일 리마인더 구현

## Phase 6: UI/UX 마무리
목표: 다크 모드, 모바일 최적화, 애니메이션

## Phase 7: 테스트 및 배포
목표: E2E 테스트, Vercel 배포 설정
```

동시에 `STATE.md` 파일도 생성되어 현재 진행 상황을 추적합니다.

### 6단계: 단계별 계획 및 실행

이제 본격적인 개발이 시작됩니다. 각 단계를 하나씩 진행합니다:

```
# Phase 1 계획 생성
/gsd:plan-phase 1
```

GSD는 Phase 1을 2-3개의 원자적 작업으로 분해한 `PLAN.md` 파일을 생성합니다:

```xml
<plan phase="1" title="프로젝트 설정 및 인증">
  <task type="auto">
    <n>Next.js 프로젝트 초기 설정</n>
    <files>
      package.json
      next.config.js
      tsconfig.json
      tailwind.config.ts
    </files>
    <action>
      create-next-app으로 프로젝트 생성 (App Router 사용).
      TypeScript strict 모드 활성화.
      Tailwind CSS 설정.
      .env.local.example 파일 생성 (Supabase, Google OAuth 키 템플릿).
    </action>
    <verify>npm run dev가 정상 실행되고 localhost:3000에 Next.js 기본 페이지 표시</verify>
    <done>프로젝트가 실행되고 TypeScript 및 Tailwind가 정상 작동</done>
  </task>

  <task type="auto">
    <n>Supabase 클라이언트 설정</n>
    <files>
      lib/supabase.ts
      .env.local
    </files>
    <action>
      @supabase/supabase-js 설치.
      createClient로 Supabase 클라이언트 생성.
      환경 변수에서 URL과 Anon Key 로드.
    </action>
    <verify>Supabase 클라이언트가 정상적으로 초기화되는지 간단한 쿼리로 테스트</verify>
    <done>Supabase 연결 확인 완료</done>
  </task>

  <task type="auto">
    <n>Google OAuth 인증 구현</n>
    <files>
      app/api/auth/callback/route.ts
      app/login/page.tsx
      middleware.ts
    </files>
    <action>
      Supabase Auth의 Google Provider 설정.
      로그인 페이지에 Google 로그인 버튼 추가.
      인증 콜백 처리 라우트 생성.
      middleware로 인증되지 않은 사용자 리디렉션.
    </action>
    <verify>Google로 로그인 시 대시보드로 리디렉션되는지 확인</verify>
    <done>Google 로그인 완전히 작동</done>
  </task>
</plan>
```

계획이 승인되면 실행합니다:

```
/gsd:execute-plan
```

GSD는 각 작업을 별도의 서브 에이전트에서 실행합니다. 이 과정에서:
- 각 작업이 완료되면 자동으로 Git 커밋
- 검증 단계를 통과하는지 확인
- 문제가 발생하면 자동으로 재시도하거나 사용자에게 알림
- `SUMMARY.md`에 실행 결과 기록

### 7단계: 진행 상황 확인 및 다음 단계

작업 중간에 현재 상태를 확인하려면:

```
/gsd:progress
```

이 명령은 다음 정보를 제공합니다:
- 현재 진행 중인 단계
- 완료된 단계 목록
- 다음에 수행할 작업
- 발견된 이슈나 경고

Phase 1이 완료되면 Phase 2로 진행합니다:

```
/gsd:plan-phase 2
/gsd:execute-plan
```

이런 식으로 모든 단계를 순차적으로 진행합니다.

### 8단계: 마일스톤 완료 및 출시

모든 단계가 완료되면 마일스톤을 완료하고 v1.0을 출시합니다:

```
/gsd:complete-milestone
```

이 명령은:
- 현재 마일스톤의 모든 문서를 아카이브
- 최종 테스트 및 검증 수행
- 배포 준비 (예: 프로덕션 빌드, 환경 변수 확인)
- Git 태그 생성 (v1.0.0)

축하합니다! 첫 번째 애플리케이션이 완성되었습니다.

### 9단계: 다음 버전 계획

v2.0에서 새로운 기능(예: 태스크 공유)을 추가하고 싶다면:

```
/gsd:discuss-milestone
```

GSD와 대화를 통해 v2.0의 목표와 새로운 기능을 논의합니다. 그런 다음:

```
/gsd:new-milestone "v2.0 - 협업 기능"
```

새로운 마일스톤이 생성되고, 위 과정을 반복합니다.

---

## 기존 프로젝트에 적용하기

이미 작업 중인 프로젝트가 있거나 레거시 코드베이스를 개선하고 싶다면(브라운필드 개발), GSD는 코드베이스를 먼저 분석하여 이해한 다음 작업을 진행합니다. 이 섹션에서는 기존 프로젝트에 GSD를 적용하는 방법을 설명합니다.

### 1단계: 코드베이스 매핑

기존 프로젝트 디렉토리로 이동한 후 Claude Code를 실행하고 코드베이스 매핑을 시작합니다:

```bash
cd existing-project
claude --dangerously-skip-permissions

# Claude Code 내에서
/gsd:map-codebase
```

이 명령은 여러 병렬 에이전트를 생성하여 코드베이스를 분석합니다. 분석 과정은 다음과 같습니다:

**분석 대상:**
- 디렉토리 구조 및 파일 조직
- 사용 중인 언어, 프레임워크, 라이브러리
- 아키텍처 패턴 (MVC, MVVM, Clean Architecture 등)
- 데이터 흐름 및 레이어 분리
- 코딩 컨벤션 및 스타일 가이드
- 테스트 프레임워크 및 커버리지
- 외부 서비스 통합 (API, 데이터베이스, 서드파티 SDK)
- 알려진 문제점 및 기술 부채

### 2단계: 코드베이스 문서 생성

매핑이 완료되면 `.planning/codebase/` 디렉토리에 7개의 문서가 생성됩니다:

#### STACK.md
프로젝트에서 사용하는 모든 기술을 나열합니다.

```markdown
# 기술 스택

## 언어
- TypeScript 5.0 (strict mode)
- Node.js 18.x

## 프론트엔드
- React 18.2
- Next.js 13 (Pages Router)
- Styled Components 5.3
- React Query 4.0

## 백엔드
- Express.js 4.18
- Prisma ORM 5.0
- PostgreSQL 15

## 테스팅
- Jest 29
- React Testing Library 13
- Playwright E2E

## 인프라
- Docker
- Nginx
- AWS (EC2, RDS, S3)
```

#### ARCHITECTURE.md
시스템의 구조와 패턴을 설명합니다.

```markdown
# 아키텍처

## 전체 구조
- 모놀리식 Next.js 애플리케이션
- API는 `/pages/api` 라우트로 구현
- 프론트엔드와 백엔드가 동일 프로세스에서 실행

## 레이어 분리
1. **Presentation**: React 컴포넌트 (pages/, components/)
2. **Business Logic**: hooks/, services/
3. **Data Access**: lib/prisma.ts, lib/api.ts
4. **Database**: PostgreSQL (Prisma 스키마)

## 데이터 흐름
Client → React Query → API Route → Service → Prisma → Database

## 주요 패턴
- Custom Hooks로 로직 재사용
- React Context API로 전역 상태 관리 (사용자 인증만)
- Styled Components의 ThemeProvider로 테마 관리
```

#### STRUCTURE.md
디렉토리 레이아웃과 각 폴더의 역할을 설명합니다.

~~~markdown
# 프로젝트 구조

```
project/
├── pages/              # Next.js 페이지 및 API 라우트
│   ├── api/           # API 엔드포인트
│   ├── index.tsx      # 홈 페이지
│   └── dashboard/     # 대시보드 페이지들
├── components/         # 재사용 가능한 React 컴포넌트
│   ├── common/        # 공통 UI 컴포넌트 (Button, Input 등)
│   └── features/      # 기능별 컴포넌트 (TaskList, UserProfile 등)
├── hooks/             # 커스텀 React Hooks
├── services/          # 비즈니스 로직 (API 호출, 데이터 변환)
├── lib/               # 유틸리티 및 설정
│   ├── prisma.ts     # Prisma 클라이언트
│   └── auth.ts       # 인증 헬퍼
├── prisma/            # Prisma 스키마 및 마이그레이션
├── public/            # 정적 파일
└── styles/            # 글로벌 스타일 및 테마
```
~~~

#### CONVENTIONS.md
코드 스타일, 네이밍 패턴, 프로젝트 규칙을 문서화합니다.

~~~markdown
# 코딩 컨벤션

## 파일 네이밍
- 컴포넌트: PascalCase (UserProfile.tsx)
- Hooks: camelCase with 'use' prefix (useAuth.ts)
- 유틸리티: camelCase (formatDate.ts)

## 컴포넌트 구조
```typescript
// 1. Imports (React, 서드파티, 로컬 순서)
import React from 'react';
import { useQuery } from 'react-query';
import { Button } from '@/components/common';

// 2. Types/Interfaces
interface Props {
  userId: string;
}

// 3. Component
export const UserProfile: React.FC<Props> = ({ userId }) => {
  // 4. Hooks
  const { data, isLoading } = useQuery(...);
  
  // 5. Handlers
  const handleClick = () => {};
  
  // 6. Render
  return <div>...</div>;
};
```

## API 라우트
- RESTful 네이밍 (GET /api/users, POST /api/users/:id)
- 모든 응답은 { data, error } 형식

## 에러 처리
- try-catch로 모든 비동기 함수 래핑
- 사용자에게는 친절한 메시지, 로그에는 상세 에러 기록
~~~

#### TESTING.md
테스트 전략과 패턴을 설명합니다.

~~~markdown
# 테스팅 전략

## 테스트 레이어
- **Unit Tests**: hooks/, services/ (Jest)
- **Integration Tests**: API routes (Jest + Supertest)
- **E2E Tests**: 주요 사용자 플로우 (Playwright)

## 커버리지 목표
- Unit: 80% 이상
- Integration: 주요 API 엔드포인트 100%
- E2E: 핵심 사용자 여정 커버

## 테스트 패턴
```typescript
describe('useAuth hook', () => {
  it('로그인 성공 시 사용자 정보 반환', async () => {
    const { result } = renderHook(() => useAuth());
    await act(() => result.current.login('test@example.com', 'password'));
    expect(result.current.user).toBeDefined();
  });
});
```
~~~

#### INTEGRATIONS.md
외부 서비스 및 API 통합을 나열합니다.

~~~markdown
# 외부 통합

## 인증
- Auth0 (OAuth 2.0, JWT)
- 환경 변수: AUTH0_DOMAIN, AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET

## 결제
- Stripe
- 웹훅: /api/webhooks/stripe
- 환경 변수: STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET

## 이메일
- SendGrid
- 환경 변수: SENDGRID_API_KEY

## 파일 저장소
- AWS S3
- 환경 변수: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, S3_BUCKET_NAME

## 분석
- Google Analytics 4
- Mixpanel
~~~

#### CONCERNS.md
알려진 문제, 기술 부채, 개선이 필요한 영역을 기록합니다.

~~~markdown
# 기술 부채 및 우려 사항

## 높은 우선순위
1. **Pages Router → App Router 마이그레이션**
   - 현재 Next.js 13을 사용하지만 여전히 Pages Router 사용
   - App Router로 전환하면 성능 및 개발자 경험 개선

2. **테스트 커버리지 부족**
   - 현재 커버리지 약 40%
   - 특히 API 라우트와 비즈니스 로직 테스트 부족

3. **중복 코드**
   - components/features/ 내 여러 컴포넌트에서 유사한 로직 반복
   - 공통 패턴을 hooks나 헬퍼로 추출 필요

## 중간 우선순위
1. **스타일 시스템 개선**
   - Styled Components와 인라인 스타일이 혼재
   - 일관된 스타일 시스템으로 통일 필요

2. **에러 처리 표준화**
   - 에러 처리 방식이 파일마다 다름
   - 중앙화된 에러 처리 메커니즘 필요

## 낮은 우선순위
1. **의존성 업데이트**
   - 일부 라이브러리가 구버전 (보안 취약점은 없음)
   - 정기적인 의존성 업데이트 프로세스 수립 필요
~~~

### 3단계: 프로젝트 초기화

코드베이스 문서가 준비되면 일반적인 프로젝트 초기화를 진행합니다:

```
/gsd:new-project
```

하지만 이번에는 GSD가 이미 코드베이스를 이해하고 있으므로 질문이 달라집니다:

```
GSD: 코드베이스 분석을 완료했습니다. Next.js 13 (Pages Router) 프로젝트이며, 
TypeScript, Prisma, Auth0을 사용 중이시네요.

이제 무엇을 추가하거나 변경하고 싶으신가요?

사용자: CONCERNS.md에 나온 것처럼 App Router로 마이그레이션하고 싶어요. 
그리고 테스트 커버리지를 80% 이상으로 올리고 싶고요.

GSD: 좋습니다. 두 가지 목표가 있네요:
1. Pages Router → App Router 마이그레이션
2. 테스트 커버리지 80% 달성

마이그레이션은 점진적으로 진행할까요, 아니면 한 번에 전체를 변환할까요?

사용자: 점진적으로 진행하되, 먼저 새로운 기능은 App Router로 구현하고, 
기존 페이지는 천천히 마이그레이션하는 방식으로 하면 좋겠어요.

GSD: 알겠습니다. 테스트는 어떤 부분에 우선순위를 둘까요?

사용자: API 라우트부터 시작해서 핵심 비즈니스 로직을 먼저 커버하고, 
그다음 E2E 테스트를 추가하는 순서로 진행하면 좋겠어요.
```

### 4단계: 로드맵 생성 및 실행

GSD는 기존 코드베이스의 구조와 컨벤션을 존중하면서 로드맵을 생성합니다:

```
/gsd:create-roadmap
```

생성된 로드맵은 다음과 같을 수 있습니다:

```markdown
# 로드맵: Next.js App Router 마이그레이션 및 테스트 개선

## Phase 1: 마이그레이션 준비
- Next.js 14로 업그레이드
- app/ 디렉토리 생성
- 기존 Pages Router와 공존 설정

## Phase 2: API 라우트 테스트 추가
- /api/users 엔드포인트 테스트 작성
- /api/auth 엔드포인트 테스트 작성
- 테스트 유틸리티 및 목(Mock) 헬퍼 생성

## Phase 3: 새 기능을 App Router로 구현
- 새로운 대시보드 페이지를 app/dashboard/로 생성
- Server Components 활용
- 기존 Pages Router 페이지는 유지

## Phase 4: 핵심 비즈니스 로직 테스트
- services/ 디렉토리 단위 테스트
- hooks/ 디렉토리 테스트
- 커버리지 측정 및 갭 분석

## Phase 5: 기존 페이지 점진적 마이그레이션
- 홈페이지부터 하나씩 App Router로 전환
- 각 페이지 마이그레이션 후 테스트 추가

## Phase 6: E2E 테스트 추가
- 주요 사용자 플로우 (로그인, 대시보드, 설정) E2E 테스트
- CI/CD 파이프라인에 통합
```

이후 과정은 신규 프로젝트와 동일합니다:

```
/gsd:plan-phase 1
/gsd:execute-plan
```

### 중요한 차이점

기존 프로젝트에서 GSD를 사용할 때의 주요 차이점:

1. **컨벤션 준수**: GSD는 코드베이스의 기존 패턴과 스타일을 따릅니다. 새로운 파일을 만들 때 CONVENTIONS.md를 참조합니다.

2. **점진적 변경**: 한 번에 모든 것을 바꾸지 않고, 기존 코드와 새 코드가 공존할 수 있도록 합니다.

3. **의존성 인식**: INTEGRATIONS.md와 STACK.md를 참조하여 이미 사용 중인 라이브러리와 서비스를 재사용합니다.

4. **기술 부채 해결**: CONCERNS.md의 문제들을 우선순위에 따라 단계적으로 해결할 수 있습니다.

---

## 실전 사용 가이드

이 섹션에서는 GSD를 효과적으로 사용하기 위한 실전 팁과 베스트 프랙티스를 다룹니다.

### 효과적인 프롬프트 작성

GSD의 품질은 제공하는 정보의 품질에 달려 있습니다. 좋은 프롬프트 예시를 살펴보겠습니다.

#### 좋은 프롬프트 예시

**나쁜 예:**
```
운동 기록 앱 만들어줘.
```

**좋은 예:**
```
Next.js 15와 Tailwind CSS를 사용한 개인용 운동 기록 앱을 만들고 싶어요.

핵심 기능:
1. 사용자가 그날 한 운동을 기록 (운동 종류, 세트 수, 무게, 시간)
2. 주간/월간 진척도를 그래프로 시각화
3. 운동 루틴 템플릿 생성 및 재사용
4. 개인 기록(PR) 자동 추적 및 알림

기술 스택:
- 프론트엔드: Next.js 15 (App Router), React, TypeScript
- 스타일링: Tailwind CSS, shadcn/ui 컴포넌트 라이브러리
- 차트: Recharts
- 데이터베이스: Supabase (PostgreSQL)
- 인증: Supabase Auth (이메일/비밀번호)

제약사항:
- 모바일 우선 디자인 (주로 헬스장에서 사용)
- 오프라인에서도 기록 가능 (나중에 동기화)
- 개인정보보호 중요 (데이터는 사용자만 접근 가능)
```

**왜 좋은가:**
- 구체적인 기능 목록
- 명확한 기술 스택
- 중요한 제약사항 명시
- 사용 컨텍스트 제공 (헬스장에서 사용)

#### 로드맵 수정 시 프롬프트

**나쁜 예:**
```
Phase 2를 더 자세하게 만들어줘.
```

**좋은 예:**
```
Phase 2(데이터베이스 설계)에서 다음 사항을 추가로 반영해 주세요:

1. exercises 테이블에 muscle_groups 컬럼 추가 (운동이 어떤 근육을 사용하는지)
2. workouts와 exercises가 다대다 관계이므로 workout_exercises 조인 테이블 필요
3. 모든 테이블에 created_at, updated_at 타임스탬프 추가
4. Row Level Security (RLS) 정책으로 사용자는 자신의 데이터만 접근 가능하게 설정

또한 Prisma ORM 대신 Supabase의 네이티브 PostgreSQL을 직접 사용하고 싶어요.
```

### 단계별 작업 관리

#### 단계 추가하기

프로젝트 진행 중 새로운 요구사항이 생기면 단계를 추가할 수 있습니다:

```
/gsd:add-phase
```

예시 대화:
```
GSD: 어떤 단계를 추가하고 싶으신가요?

사용자: 사용자들이 운동 사진을 업로드해서 진척도를 시각적으로 비교할 수 있는 
기능을 추가하고 싶어요. Supabase Storage를 사용할 거고요.

GSD: 좋습니다. 이 단계를 현재 로드맵의 어디에 추가할까요? 
현재 Phase 7까지 있습니다.

사용자: Phase 5(그래프 시각화) 다음에 추가해 주세요.

GSD: 알겠습니다. Phase 6으로 "운동 사진 업로드 및 진척도 비교" 단계를 생성하고, 
기존 Phase 6과 7을 각각 Phase 7과 8로 밀어낼게요.
```

#### 긴급 작업 삽입하기

중요한 버그나 긴급 요청이 있을 때는 단계를 삽입합니다:

```
/gsd:insert-phase 3
```

이렇게 하면 Phase 3과 4 사이에 새로운 단계가 삽입됩니다. 예를 들어, Phase 3을 완료한 후 중대한 보안 문제를 발견했다면:

```
/gsd:insert-phase 4

GSD: Phase 4와 5 사이에 새로운 단계를 삽입합니다. 무엇을 해야 하나요?

사용자: Supabase Row Level Security 정책에 문제가 있어요. 
사용자가 다른 사람의 운동 기록을 볼 수 있는 취약점이 발견됐어요. 
이걸 즉시 수정해야 해요.

GSD: 이해했습니다. "보안 취약점 긴급 수정 - RLS 정책 강화"를 새 Phase 4로 생성하고, 
기존 Phase 4 이후는 한 칸씩 밀어낼게요.
```

### 작업 중단 및 재개

며칠 동안 프로젝트를 떠나야 할 때:

```
# 작업 중단 전
/gsd:pause-work
```

이 명령은 현재 상태를 상세히 기록한 핸드오프 파일을 생성합니다:
- 현재 진행 중인 단계와 작업
- 완료된 작업 목록
- 다음에 해야 할 일
- 열린 이슈 또는 결정이 필요한 사항
- 관련 파일 및 코드 위치

나중에 돌아와서:

```
/gsd:resume-work
```

GSD는 핸드오프 파일을 읽고 정확히 중단된 지점부터 다시 시작할 수 있도록 컨텍스트를 복원합니다.

### 이슈 관리

작업 중 "나중에 해야 할 일"이 생기면 즉시 ISSUES.md에 기록됩니다:

```markdown
# 이슈 및 개선 사항

## 높은 우선순위
- [ ] #1: 이메일 알림 전송 실패 시 재시도 로직 추가
- [ ] #2: 사용자 프로필 이미지 크기 제한 (현재 무제한)

## 중간 우선순위
- [ ] #3: 운동 루틴 템플릿을 다른 사용자와 공유하는 기능
- [ ] #4: 다크 모드 추가

## 낮은 우선순위
- [x] #5: 로딩 스피너 애니메이션 개선 (완료)
- [ ] #6: 온보딩 튜토리얼 추가
```

정기적으로 이슈를 검토합니다:

```
/gsd:consider-issues
```

GSD는:
- 해결된 이슈를 자동으로 닫음
- 긴급한 이슈가 있는지 확인
- 우선순위 재조정 제안
- 다음 마일스톤에 포함할 이슈 선택 도움

### 단계별 심층 논의

복잡하거나 불확실한 단계를 계획하기 전에 더 많은 컨텍스트를 수집할 수 있습니다:

```
/gsd:discuss-phase 5
```

예시:
```
GSD: Phase 5 "리마인더 시스템"에 대해 논의하겠습니다. 
현재 로드맵에는 브라우저 알림과 이메일 리마인더를 구현한다고 되어 있는데, 
더 구체적으로 어떻게 작동해야 하나요?

사용자: 사용자가 각 태스크마다 리마인더 시간을 설정할 수 있어야 해요. 
예를 들어 "오후 2시에 운동하기" 같은 식으로요. 
그러면 그 시간에 브라우저 알림이 뜨고, 10분 전에 이메일도 발송되면 좋겠어요.

GSD: 알겠습니다. 몇 가지 기술적 결정이 필요합니다:

1. 리마인더 스케줄링을 어디서 처리할까요?
   a) 클라이언트 측 (브라우저의 Notification API + setTimeout)
   b) 서버 측 (크론 작업 또는 스케줄러)

2. 사용자가 여러 기기를 사용한다면?

3. 사용자가 오프라인이거나 브라우저를 닫았다면?

사용자: 서버 측에서 처리하는 게 더 안정적일 것 같아요. 
Vercel Cron Jobs나 별도의 워커 프로세스를 사용하면 될까요?

GSD: Vercel Cron Jobs는 최소 1분마다만 실행 가능하고, 
정확한 시간 보장이 어렵습니다. 
더 나은 방법은 Supabase의 pg_cron이나 별도의 Redis + Bull Queue를 사용하는 것입니다. 
어느 것을 선호하시나요?

사용자: Supabase pg_cron을 사용하면 추가 인프라가 필요 없으니까 그걸로 해요.
```

이런 식으로 충분히 논의한 후:

```
/gsd:plan-phase 5
```

계획이 훨씬 더 구체적이고 실행 가능해집니다.

### 리서치가 필요한 경우

생소한 기술이나 도메인을 다룰 때는 심층 리서치를 수행할 수 있습니다:

```
/gsd:research-phase 6
```

예시:
```
GSD: Phase 6 "WebRTC 비디오 통화"에 대한 리서치를 시작합니다. 
이 기술은 프로젝트에 처음 도입되는 것으로 보입니다.

리서치 진행 중...

리서치 결과:

## WebRTC 구현 옵션

### 1. 순수 WebRTC (복잡도: 높음)
장점: 완전한 제어, 비용 없음
단점: TURN/STUN 서버 직접 관리, 시그널링 서버 구축 필요

### 2. Agora.io (복잡도: 낮음)
장점: 간단한 SDK, 안정성 높음, 녹화 기능 내장
단점: 유료 (분당 과금)

### 3. Daily.co (복잡도: 낮음)
장점: React 컴포넌트 제공, 관리 필요 없음
단점: 유료, 커스터마이징 제한

### 권장 사항
프로토타입 단계에서는 Daily.co의 무료 티어로 시작하고, 
나중에 트래픽이 증가하면 자체 WebRTC 인프라로 전환하는 것을 추천합니다.

다음 단계:
1. Daily.co 계정 생성 및 API 키 발급
2. @daily-co/daily-react 패키지 설치
3. 간단한 1:1 화상 통화 POC 구현
4. 화면 공유, 녹화 기능 추가

이 방향으로 진행할까요?
```

### 가정 확인하기

GSD가 잘못된 가정을 하고 있다고 느껴질 때:

```
/gsd:list-phase-assumptions 3
```

예시 출력:
```
Phase 3 "사용자 인증"에 대한 GSD의 가정:

1. 사용자는 이메일/비밀번호로만 가입한다.
   → 실제 요구사항: Google, Apple 소셜 로그인도 필요

2. 비밀번호 재설정은 이메일을 통해서만 이루어진다.
   → 확인 필요

3. 세션은 JWT 토큰으로 관리한다.
   → 실제: Supabase Auth 사용 (자체 세션 관리)

4. 2FA(이중 인증)는 필요 없다.
   → 확인 필요

잘못된 가정을 수정하고 싶으신가요?
```

이렇게 하면 계획을 실행하기 전에 잘못된 방향으로 가는 것을 방지할 수 있습니다.

---

## 고급 활용법

GSD를 마스터한 개발자들을 위한 고급 기법을 소개합니다.

### 병렬 워크플로우

Claude Code의 창시자인 Alex Cherny는 동시에 5-10개의 Claude 인스턴스를 실행한다고 밝혔습니다. GSD에서도 이를 구현할 수 있습니다.

**시나리오**: 프론트엔드와 백엔드를 동시에 개발하고 싶은 경우

터미널 1 (프론트엔드):
```bash
cd my-app/frontend
claude --dangerously-skip-permissions

# 프론트엔드 단계만 실행
/gsd:plan-phase 3  # UI 컴포넌트 개발
/gsd:execute-plan
```

터미널 2 (백엔드):
```bash
cd my-app/backend
claude --dangerously-skip-permissions

# 백엔드 단계 실행
/gsd:plan-phase 2  # API 엔드포인트 개발
/gsd:execute-plan
```

**주의사항:**
- 두 작업이 같은 파일을 수정하지 않는지 확인
- STATE.md는 각각 독립적으로 관리
- 완료 후 수동으로 통합 테스트 필요

### CLAUDE.md와의 통합

CLAUDE.md는 프로젝트별 AI 지시사항을 저장하는 파일입니다. GSD와 함께 사용하면 강력합니다:

**CLAUDE.md 예시:**
```markdown
# 프로젝트 규칙

## 절대 하지 말 것
- Redux를 사용하지 마세요 (Zustand만 사용)
- 인라인 스타일을 사용하지 마세요 (Tailwind 클래스만)
- any 타입을 사용하지 마세요 (TypeScript strict mode)

## 항상 할 것
- 모든 컴포넌트에 PropTypes 또는 TypeScript 인터페이스 정의
- 비동기 함수는 try-catch로 감싸기
- 새로운 패키지 설치 전 번들 크기 확인

## 스타일 가이드
- 최대 줄 길이: 80자
- 들여쓰기: 2 스페이스
- 세미콜론 사용

## 코드 리뷰 체크리스트
프랜드 리퀘스트(PR) 전에 다음을 확인하세요:
- [ ] 타입 에러 없음 (`npm run type-check`)
- [ ] 린트 통과 (`npm run lint`)
- [ ] 테스트 통과 (`npm test`)
- [ ] 번들 크기 증가가 10% 미만
```

GSD는 각 작업을 실행할 때 자동으로 CLAUDE.md를 컨텍스트에 포함시킵니다. 이를 통해:
- 프로젝트 규칙이 일관되게 적용됨
- 실수를 미리 방지
- 코드 품질이 자동으로 유지됨

### 커스텀 명령어 생성

GSD의 명령어 시스템을 확장할 수 있습니다. `.claude/commands/` 디렉토리에 커스텀 명령어를 추가하면 됩니다.

**예시: 배포 체크리스트 명령어**

`.claude/commands/deploy-check.md`:
```markdown
# /deploy-check 명령어

배포 전 체크리스트를 실행합니다.

## 실행 항목
1. `npm run build` 성공 확인
2. `npm run test` 모든 테스트 통과 확인
3. 환경 변수 설정 확인 (.env.production)
4. 데이터베이스 마이그레이션 실행 확인
5. 번들 크기 분석 (이전 버전 대비)
6. Lighthouse 점수 확인 (90점 이상)

## 출력
- 각 항목의 통과/실패 상태
- 실패 항목에 대한 수정 방법
- 배포 준비 완료 여부
```

이제 `/deploy-check`를 입력하면 자동으로 모든 항목을 확인합니다.

### 멀티 프로젝트 관리

여러 프로젝트를 동시에 관리하는 경우:

**전역 GSD 설정 (~/.claude/gsd-config.json):**
```json
{
  "projects": {
    "main-app": {
      "path": "~/projects/main-app",
      "claude_model": "claude-opus-4-5",
      "auto_commit": true
    },
    "landing-page": {
      "path": "~/projects/landing-page",
      "claude_model": "claude-sonnet-4",
      "auto_commit": false
    },
    "admin-dashboard": {
      "path": "~/projects/admin-dashboard",
      "claude_model": "claude-opus-4-5",
      "auto_commit": true
    }
  }
}
```

**프로젝트 전환:**
```bash
# 프로젝트 간 빠른 전환을 위한 alias 설정
alias gsd-main='cd ~/projects/main-app && claude --dangerously-skip-permissions'
alias gsd-landing='cd ~/projects/landing-page && claude'
alias gsd-admin='cd ~/projects/admin-dashboard && claude --dangerously-skip-permissions'
```

### 템플릿 활용

자주 만드는 프로젝트 타입이 있다면 템플릿을 생성할 수 있습니다:

**Next.js SaaS 템플릿 (.claude/templates/nextjs-saas.json):**
```json
{
  "name": "Next.js SaaS Template",
  "tech_stack": {
    "frontend": "Next.js 15, React, TypeScript, Tailwind CSS",
    "backend": "Next.js API Routes, Server Actions",
    "database": "Supabase (PostgreSQL)",
    "auth": "Supabase Auth + Google OAuth",
    "payments": "Stripe",
    "email": "Resend"
  },
  "standard_phases": [
    {
      "name": "프로젝트 설정",
      "tasks": ["Next.js 초기화", "TypeScript 설정", "Tailwind 설정"]
    },
    {
      "name": "인증 시스템",
      "tasks": ["Supabase 클라이언트", "Google OAuth", "보호된 라우트"]
    },
    {
      "name": "결제 시스템",
      "tasks": ["Stripe 통합", "구독 관리", "웹훅 처리"]
    },
    {
      "name": "대시보드 UI",
      "tasks": ["레이아웃", "내비게이션", "사용자 프로필"]
    }
  ],
  "conventions": {
    "component_structure": "Atomic Design",
    "state_management": "Zustand",
    "form_handling": "React Hook Form + Zod",
    "api_style": "RESTful"
  }
}
```

새 프로젝트 시작 시:
```
/gsd:new-project --template nextjs-saas
```

템플릿의 모든 설정이 자동으로 적용되어 훨씬 빠르게 시작할 수 있습니다.

---

## 문제 해결 및 팁

GSD 사용 중 발생할 수 있는 일반적인 문제와 해결 방법입니다.

### 일반적인 문제

#### 문제 1: "Permission denied" 에러

**증상:**
```
Error: Permission denied: Bash(git commit:*)
```

**해결 방법:**
옵션 A: Claude Code를 권한 스킵 모드로 실행
```bash
claude --dangerously-skip-permissions
```

옵션 B: `.claude/settings.json`에 권한 추가
```json
{
  "permissions": {
    "allow": [
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)"
    ]
  }
}
```

#### 문제 2: 계획 실행이 중간에 멈춤

**증상:**
작업 1은 완료되었는데 작업 2로 넘어가지 않고 멈춤

**원인:**
검증 단계에서 실패했지만 명확하게 보고되지 않음

**해결 방법:**
STATE.md를 확인하여 마지막 작업의 상태를 확인:
```
/gsd:progress
```

수동으로 검증 조건을 확인:
```bash
# PLAN.md의 verify 섹션에 명시된 명령 실행
curl -X POST localhost:3000/api/auth/login
```

문제를 수정한 후:
```
/gsd:execute-plan  # 실패한 작업부터 재시도
```

#### 문제 3: Git 커밋 메시지가 일관성 없음

**증상:**
커밋 메시지가 "WIP", "fix", "update" 같은 일반적인 메시지로 작성됨

**원인:**
GSD의 커밋 메시지 템플릿이 제대로 적용되지 않음

**해결 방법:**
`.claude/gsd-config.json`에 커밋 메시지 규칙 추가:
```json
{
  "git": {
    "commit_template": "{type}({phase}): {description}",
    "commit_types": ["feat", "fix", "docs", "refactor", "test"]
  }
}
```

#### 문제 4: 컨텍스트 창 초과 경고

**증상:**
```
Warning: Context window nearly full (195k / 200k tokens)
```

**원인:**
PROJECT.md 또는 ROADMAP.md가 너무 큼

**해결 방법:**
문서를 간결하게 정리:
- PROJECT.md: 핵심 정보만 유지, 자세한 설명은 별도 문서로
- ROADMAP.md: 완료된 단계는 아카이브로 이동
- STATE.md: 오래된 정보 정리

```
/gsd:cleanup-docs
```

이 명령은 자동으로 불필요한 정보를 정리합니다.

### 성능 최적화

#### 팁 1: 적절한 작업 크기 유지

각 원자적 작업은 다음 기준을 만족해야 합니다:
- **시간**: 15-30분 내에 완료 가능
- **파일**: 1-3개 파일만 수정
- **테스트**: 독립적으로 테스트 가능

너무 큰 작업은 여러 개로 분할하고, 너무 작은 작업은 병합합니다.

#### 팁 2: 효과적인 단계 분할

각 단계(Phase)는:
- **목표**: 명확하고 측정 가능한 산출물
- **작업 수**: 6-9개의 원자적 작업 (2-3개씩 3개 계획)
- **의존성**: 이전 단계의 완료가 필요한지 명확히

#### 팁 3: 서브 에이전트 재사용

같은 유형의 작업을 여러 번 수행하는 경우, 템플릿을 만들어 재사용:

```xml
