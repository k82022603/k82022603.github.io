---
title: "Claude Code 실무 적용 가이드: 보리스의 워크플로우를 프로젝트에 도입하기"
date: 2026-01-10 07:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  Developer,  Claude.write,  Gemini.write]
---


보리스가 제시한 Claude Code 활용법을 실제 프로젝트에 적용하려면 단순히 개념을 이해하는 것을 넘어, 구체적인 설정과 파일 구조, 그리고 팀 내 프로세스를 체계화해야 합니다. 이 문서에서는 즉시 실행 가능한 구체적인 전략들을 단계별로 살펴보겠습니다.


## claude.md 파일: 프로젝트의 DNA 설계하기

claude.md 파일은 Claude가 프로젝트를 처음 접했을 때 읽는 '온보딩 문서'입니다. 이 파일이 없으면 Claude는 매번 프로젝트의 맥락을 새로 파악해야 하며, 같은 실수를 반복할 가능성이 높아집니다. 효과적인 claude.md는 단순한 README가 아니라, Claude의 행동을 규정하는 명확한 규칙서여야 합니다.

### 필수 구성 요소와 작성 원칙

claude.md는 크게 네 가지 섹션으로 구성됩니다. 첫째는 프로젝트 개요로, 이 프로젝트가 무엇을 하는지, 핵심 목표가 무엇인지를 한두 문장으로 명확히 정의합니다. 예를 들어 "이 프로젝트는 Next.js 기반의 B2B SaaS 대시보드이며, 실시간 데이터 시각화와 다중 테넌트 권한 관리가 핵심 기능입니다"와 같은 식입니다.

둘째는 기술 스택과 아키텍처 설명입니다. 여기서는 단순히 "React, TypeScript, PostgreSQL 사용"이라고 나열하는 것이 아니라, 각 기술이 어떻게 조직되어 있는지를 설명해야 합니다. "프론트엔드는 src/components 디렉토리에 Atomic Design 패턴으로 구성되어 있으며, API 호출은 src/services에서 Axios 인스턴스를 통해 중앙 집중식으로 관리됩니다. 상태 관리는 Zustand를 사용하며, 글로벌 상태는 src/store에 정의되어 있습니다"와 같이 구체적으로 적습니다.

셋째는 코딩 컨벤션과 스타일 가이드입니다. ESLint나 Prettier 설정이 있더라도, Claude가 특히 주의해야 할 규칙을 명시적으로 적어야 합니다. 예를 들어 "모든 컴포넌트는 반드시 TypeScript로 작성하며, any 타입 사용 금지. Props는 interface로 정의하고 파일 상단에 배치. 비동기 함수는 try-catch로 감싸고 에러는 반드시 로깅 서비스로 전송"과 같은 구체적인 지침을 포함합니다.

넷째, 그리고 가장 중요한 것이 **"하지 말아야 할 것들(DON'Ts)"** 섹션입니다. 보리스가 강조했듯이, Claude가 반복적으로 저지르는 실수를 여기에 기록하는 것이 핵심입니다. "절대 axios를 직접 import하지 말 것. 반드시 src/services/api.ts의 apiClient를 사용할 것", "환경 변수는 process.env가 아니라 config/env.ts를 통해 접근할 것", "데이터베이스 마이그레이션 파일을 수동으로 수정하지 말 것. 항상 새 마이그레이션을 생성할 것"과 같은 금지 사항들이 들어갑니다.

### 실제 claude.md 예시

실제 프로젝트에 적용할 수 있는 claude.md의 구체적인 예시를 살펴보겠습니다. 이 예시는 웹 애플리케이션 프로젝트를 가정합니다.

~~~markdown
# 프로젝트 개요
이 프로젝트는 실시간 협업 문서 편집 플랫폼입니다. Next.js 14 App Router 기반이며, WebSocket을 통한 실시간 동기화가 핵심 기능입니다.

# 기술 스택
- Frontend: Next.js 14, React 18, TypeScript 5.x
- Styling: Tailwind CSS, shadcn/ui
- State Management: Zustand (글로벌), React Query (서버 상태)
- Backend: Next.js API Routes, Prisma ORM
- Database: PostgreSQL 15
- Real-time: Socket.io
- Testing: Vitest, React Testing Library

# 디렉토리 구조
```
src/
├── app/              # Next.js App Router 페이지
├── components/       # React 컴포넌트 (Atomic Design)
│   ├── atoms/
│   ├── molecules/
│   ├── organisms/
│   └── templates/
├── lib/             # 유틸리티 함수
├── services/        # API 클라이언트 및 비즈니스 로직
├── store/           # Zustand 스토어
├── types/           # TypeScript 타입 정의
└── hooks/           # 커스텀 React Hook
```

# 코딩 규칙
1. 모든 컴포넌트는 함수형 컴포넌트로 작성
2. Props는 interface로 정의하고 컴포넌트 위에 배치
3. 파일명은 PascalCase (컴포넌트), camelCase (유틸리티)
4. 한 파일당 하나의 컴포넌트만 export
5. 비동기 작업은 React Query 사용 권장
6. 에러 처리는 Error Boundary 활용

# 절대 하지 말 것 (CRITICAL)
1. ❌ fetch() 직접 사용 금지 → services/api.ts의 apiClient 사용
2. ❌ 인라인 스타일 작성 금지 → Tailwind 유틸리티 클래스 사용
3. ❌ any 타입 사용 금지 → unknown 사용 후 타입 가드 구현
4. ❌ console.log 배포 코드에 남기지 말 것
5. ❌ Prisma 스키마 직접 수정 금지 → 마이그레이션 생성
6. ❌ 환경 변수 process.env 직접 접근 금지 → lib/env.ts 사용
7. ❌ setTimeout/setInterval 직접 사용 금지 → hooks/useTimer.ts 사용

# 테스트 작성 규칙
1. 모든 컴포넌트는 최소 렌더링 테스트 필수
2. API 호출 함수는 모킹된 응답으로 테스트
3. 테스트 파일명: ComponentName.test.tsx
4. 각 테스트는 describe로 그룹화
5. 테스트 실행: npm run test

# Claude가 자주 실수한 내역 (계속 업데이트)
- 2025-01-08: API 라우트에서 CORS 헤더 누락 → middleware.ts 확인 필수
- 2025-01-09: Socket.io 이벤트 리스너 중복 등록 → useEffect cleanup 필수
- 2025-01-10: Prisma 쿼리에서 relation 누락 → include 명시적 지정
~~~

이 예시에서 주목할 점은 단순한 설명이 아니라 **Claude가 실행 가능한 명령**으로 작성되어 있다는 것입니다. "Tailwind를 사용합니다"가 아니라 "인라인 스타일 금지, Tailwind 유틸리티 클래스 사용"처럼 행동을 지시하는 방식입니다.

### 팀 단위 claude.md 관리 전략

보리스의 팀이 매주 여러 번 claude.md를 업데이트한다는 점은 이 파일이 정적인 문서가 아니라 살아있는 지식 베이스라는 것을 의미합니다. 이를 팀 차원에서 관리하려면 몇 가지 프로세스가 필요합니다.

첫째, 코드 리뷰 시 Claude가 만든 코드에서 발견된 패턴적 실수는 즉시 claude.md에 추가합니다. 예를 들어 Pull Request 리뷰에서 "또 API 호출에 에러 핸들링이 빠졌네요"라는 피드백이 반복된다면, "모든 API 호출은 try-catch로 감싸고 toast.error()로 사용자에게 알릴 것"이라는 규칙을 추가합니다.

둘째, 주간 회고에서 "Claude가 이번 주에 가장 많이 실수한 것"을 공유하고, 이를 문서화합니다. 이는 단순히 불평하는 것이 아니라, 팀 전체가 Claude를 더 효과적으로 사용하기 위한 학습 과정입니다.

셋째, claude.md 자체를 버전 관리합니다. Git에 커밋하면서 "Added: API rate limiting 주의사항", "Updated: 테스트 커버리지 기준 70% → 80%"와 같이 변경 내용을 커밋 메시지에 명확히 기록합니다.

## 테스트 주도 워크플로우: Claude에게 자가 검증 능력 부여하기

보리스가 강조한 "검증 루프" 구축은 Claude의 생산성을 극대화하는 핵심 전략입니다. Claude가 코드를 짜고 나서 사람이 확인하는 것이 아니라, Claude 스스로 테스트를 실행하고 통과할 때까지 반복하게 만드는 것입니다.

### 테스트 우선 프롬프팅 기법

일반적으로 우리는 "사용자 인증 기능을 구현해줘"라고 요청합니다. 하지만 보리스의 방식은 다릅니다. "사용자 인증 기능을 구현하되, 먼저 다음과 같은 테스트 케이스를 작성하고, 이 테스트가 모두 통과할 때까지 코드를 반복 수정해줘"라고 지시합니다.

구체적인 프롬프트 예시를 보겠습니다.

```
나는 사용자 로그인 API를 만들려고 해. 다음 순서로 작업해줘:

1. 먼저 tests/api/auth.test.ts 파일을 만들어서 다음 테스트 케이스를 작성:
   - 올바른 이메일/비밀번호로 로그인 시 JWT 토큰 반환
   - 잘못된 비밀번호로 로그인 시 401 에러 반환
   - 존재하지 않는 이메일로 로그인 시 404 에러 반환
   - 이메일 형식이 잘못된 경우 400 에러 반환
   - 비밀번호가 5자 미만인 경우 400 에러 반환

2. 그 다음 실제 API 라우트를 구현해줘 (src/app/api/auth/login/route.ts)

3. 테스트를 실행해서 모두 통과하는지 확인하고, 실패하면 원인을 분석해서 코드를 수정해줘

4. 모든 테스트가 통과하면 결과를 요약해서 보고해줘
```

이런 식으로 지시하면 Claude는 다음과 같은 순서로 작업합니다. 먼저 테스트 코드를 작성하고, 실제 구현을 하고, `npm test`를 실행합니다. 테스트가 실패하면 에러 메시지를 읽고 코드를 수정하고 다시 테스트를 실행합니다. 이 과정을 테스트가 통과할 때까지 반복합니다. 사용자는 중간 과정에 개입할 필요 없이, 최종 결과만 확인하면 됩니다.

### 실전 테스트 설정: Vitest와 React Testing Library

보리스가 선호하는 테스트 환경은 빠르고 설정이 간단한 도구들입니다. Vitest는 Vite 기반 프로젝트에서 Jest보다 훨씬 빠르게 동작하며, 설정도 최소화되어 있습니다.

프로젝트에 테스트 환경을 구축하는 구체적인 단계는 다음과 같습니다. 먼저 필요한 패키지를 설치합니다.

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

그 다음 vitest.config.ts 파일을 생성합니다.

```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
    globals: true,
    coverage: {
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

tests/setup.ts 파일을 만들어서 테스트 환경을 초기화합니다.

```typescript
import '@testing-library/jest-dom'
import { expect, afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'

// 각 테스트 후 자동으로 cleanup
afterEach(() => {
  cleanup()
})
```

이제 package.json에 테스트 스크립트를 추가합니다.

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

이 설정이 완료되면 Claude에게 "새로운 컴포넌트를 만들 때 항상 .test.tsx 파일도 함께 만들고, 렌더링 테스트와 주요 인터랙션 테스트를 포함시켜줘. 그리고 npm test를 실행해서 통과하는지 확인해줘"라고 지시할 수 있습니다.

### UI 컴포넌트 자동 검증: Browser 도구 활용

백엔드 로직은 테스트 코드로 검증할 수 있지만, UI는 눈으로 확인해야 합니다. 보리스는 Claude의 브라우저 도구를 활용해서 이 과정도 자동화합니다.

Claude Code에 브라우저 확장 기능이 설치되어 있다면, 다음과 같이 지시할 수 있습니다.

```
로그인 페이지를 만들어줘. 다음 요구사항을 충족해야 해:
- 이메일과 비밀번호 입력 필드
- "로그인" 버튼 (클릭 시 API 호출)
- 로딩 중에는 버튼 비활성화 및 스피너 표시
- 에러 발생 시 빨간색 메시지 표시

구현이 끝나면:
1. npm run dev로 개발 서버 실행
2. 브라우저로 http://localhost:3000/login 열기
3. 스크린샷 찍어서 보여주기
4. 실제로 폼에 데이터 입력하고 제출 버튼 클릭해보기
5. 동작이 의도대로 되는지 확인하고 보고
```

Claude는 터미널에서 `npm run dev`를 실행하고, 브라우저를 열어서 페이지를 로드하고, 스크린샷을 찍어서 보여줍니다. 심지어 폼에 테스트 데이터를 입력하고 버튼을 클릭하는 것까지 자동으로 수행할 수 있습니다. 만약 레이아웃이 깨지거나 버튼이 동작하지 않으면, Claude가 이를 인지하고 코드를 수정한 뒤 다시 확인하는 과정을 반복합니다.

## 명령어 및 워크플로우 자동화: 반복 작업 제거하기

보리스는 자주 반복되는 작업을 슬래시 명령어로 만들어서 팀과 공유한다고 했습니다. Claude Code는 프로젝트 폴더에 `.claude/commands` 디렉토리를 만들어서 커스텀 명령어를 정의할 수 있습니다.

### 커스텀 명령어 만들기

예를 들어 새로운 React 컴포넌트를 만들 때마다 동일한 보일러플레이트가 필요하다면, 다음과 같은 명령어를 만들 수 있습니다.

`.claude/commands/new-component.md` 파일을 생성합니다.

~~~markdown
# 새 React 컴포넌트 생성

다음 단계로 새 컴포넌트를 생성해줘:

1. 사용자에게 컴포넌트 이름을 입력받아 (예: UserProfile)
2. src/components/[category]/[ComponentName].tsx 생성
   - Props interface 정의
   - TypeScript 함수형 컴포넌트 작성
   - 기본 props 값 설정
3. src/components/[category]/[ComponentName].test.tsx 생성
   - 기본 렌더링 테스트
   - Props 전달 테스트
4. src/components/[category]/index.ts에 export 추가
5. 테스트 실행해서 통과 확인

컴포넌트 템플릿:
```typescript
interface [ComponentName]Props {
  // Props 정의
}

export function [ComponentName]({ }: [ComponentName]Props) {
  return (
    <div>
      {/* 구현 */}
    </div>
  )
}
```
~~~

이제 Claude Code에서 `/new-component`를 입력하면 이 워크플로우가 자동으로 실행됩니다.

또 다른 유용한 명령어는 API 엔드포인트 생성입니다.

`.claude/commands/new-api.md` 파일:

```markdown
# 새 API 엔드포인트 생성

다음 순서로 진행:

1. 엔드포인트 경로와 메서드를 사용자에게 확인 (예: POST /api/users)
2. src/app/api/[path]/route.ts 생성
   - 요청 body validation (Zod 사용)
   - Prisma를 통한 DB 작업
   - 적절한 HTTP 상태 코드 반환
   - 에러 핸들링 포함
3. tests/api/[path].test.ts 생성
   - 성공 케이스 테스트
   - validation 실패 케이스 테스트
   - 에러 핸들링 테스트
4. Postman 컬렉션 업데이트 (postman/collection.json)
5. 테스트 실행 및 결과 보고

모든 API는 다음 형식을 따를 것:
- 인증 필요 시 미들웨어 사용
- 에러는 { error: string, details?: any } 형식
- 성공 응답은 { data: any } 형식
```

### Settings.json: 보안과 효율의 균형

보리스는 settings.json을 통해 Claude가 실행할 수 있는 터미널 명령어를 제어한다고 했습니다. 이는 보안상 중요한 설정입니다.

`.claude/settings.json` 파일을 생성해서 다음과 같이 설정할 수 있습니다.

```json
{
  "allowedCommands": [
    "npm test",
    "npm run build",
    "npm run dev",
    "git status",
    "git diff",
    "npx prisma migrate dev",
    "npx prisma generate"
  ],
  "blockedCommands": [
    "rm -rf",
    "sudo",
    "npm publish",
    "git push --force",
    "dropdb"
  ],
  "requireConfirmation": [
    "git push",
    "npm install",
    "prisma migrate deploy"
  ],
  "autoApprove": [
    "npm test",
    "git status"
  ]
}
```

이 설정은 Claude가 테스트나 빌드는 자유롭게 실행할 수 있지만, 위험한 명령어는 차단하고, 중요한 명령어는 사용자 확인을 받도록 합니다.

## 멀티 세션 및 병렬 작업 전략

보리스가 5개의 터미널 탭에서 병렬로 Claude를 실행한다는 것은 단순히 여러 개를 동시에 켜놓는 것 이상의 의미가 있습니다. 각 세션에 명확한 역할을 부여하고, 서로 간섭하지 않도록 설계하는 것이 핵심입니다.

### 세션 분리 전략

실무에서 다음과 같이 역할을 나눌 수 있습니다. 

**세션 1 - 새 기능 개발**: 이 세션에서는 Product Manager나 디자이너가 요청한 새로운 기능을 구현합니다. "대시보드에 실시간 알림 기능 추가"와 같은 작업입니다.

**세션 2 - 버그 수정**: 이슈 트래커나 Sentry에서 발견된 버그를 수정하는 전용 세션입니다. "로그인 후 리다이렉트가 작동하지 않는 문제 해결"과 같은 작업입니다.

**세션 3 - 테스트 및 리팩토링**: 기존 코드의 품질을 개선하는 세션입니다. "UserService의 테스트 커버리지를 80%로 올리기", "레거시 클래스 컴포넌트를 함수형으로 변환"과 같은 작업입니다.

**세션 4 - 문서화**: API 문서, README, 주석 업데이트 등을 담당합니다. "새로 추가된 인증 API를 Swagger 문서에 반영"과 같은 작업입니다.

**세션 5 - 모니터링 및 자동화**: 로그 분석, 성능 측정, CI/CD 스크립트 작성 등을 처리합니다.

각 세션은 독립적인 Git 브랜치에서 작업하며, claude.md는 공유하지만 작업 컨텍스트는 분리됩니다. 이렇게 하면 한 세션에서 복잡한 작업을 진행하는 동안 다른 세션에서 간단한 버그를 빠르게 수정할 수 있습니다.

### 비동기 작업 패턴

보리스의 모바일 워크플로우는 "지시하고 떠나기" 패턴입니다. 예를 들어 출근길에 스마트폰으로 Claude에게 다음과 같이 지시합니다.

```
오늘 아침 Sentry에 보고된 에러들을 분석해줘:
1. 지난 24시간 에러 로그를 Sentry API로 가져오기
2. 각 에러의 빈도와 영향받은 사용자 수 정리
3. 재현 가능한 에러는 재현 단계 작성
4. 수정 가능한 것은 바로 수정하고 테스트
5. 수정 못한 것은 GitHub Issue로 등록
6. 결과를 summary.md 파일로 저장

내가 사무실 도착할 때쯤 끝나있을거야. 중간에 질문 있으면 슬랙으로 보내줘.
```

Claude는 이 작업을 백그라운드에서 수행하고, 사용자가 컴퓨터 앞에 앉았을 때는 이미 분석 보고서와 수정된 코드, 그리고 생성된 이슈 티켓이 준비되어 있습니다.

## 실제 프로젝트 적용 시나리오

이론을 실제로 적용하는 예시를 살펴보겠습니다. 새로운 웹 프로젝트를 시작한다고 가정합니다.

### Day 1: 프로젝트 초기 설정

먼저 Claude에게 다음과 같이 지시합니다.

```
Next.js 14 프로젝트를 생성하고 다음 설정을 해줘:
1. TypeScript, Tailwind CSS, ESLint, Prettier 설정
2. Vitest와 React Testing Library 설치 및 설정
3. Zustand, React Query 설치
4. 기본 디렉토리 구조 생성 (components, lib, services, hooks)
5. claude.md 파일 생성 (프로젝트 개요, 기술 스택, 규칙 포함)
6. .claude/commands 디렉토리에 new-component, new-api 명령어 추가
7. Git 초기화 및 .gitignore 설정
8. README.md 작성

모든 설정이 끝나면 npm run dev로 서버 실행해서 정상 작동 확인.
```

Claude는 약 5분 안에 모든 설정을 완료하고, 개발 서버가 정상적으로 실행되는 것까지 확인해줍니다.

### Day 2-5: 핵심 기능 개발

이제 실제 기능을 개발합니다. 사용자 인증 시스템을 만든다고 가정하겠습니다.

```
사용자 인증 시스템을 구축해줘. TDD 방식으로 진행:

Phase 1 - 백엔드
1. Prisma 스키마에 User 모델 추가 (email, password, name)
2. POST /api/auth/register 엔드포인트 테스트 작성
3. 엔드포인트 구현 (비밀번호 해싱 포함)
4. POST /api/auth/login 테스트 작성 및 구현
5. JWT 기반 인증 미들웨어 구현
6. 모든 테스트 통과 확인

Phase 2 - 프론트엔드
1. 로그인 페이지 컴포넌트 테스트 작성
2. 컴포넌트 구현 (폼 validation 포함)
3. 회원가입 페이지 동일하게 진행
4. 인증 상태 관리 Zustand 스토어 생성
5. 보호된 라우트 구현 (미들웨어 사용)
6. 브라우저로 실제 동작 테스트

Phase 3 - 통합 테스트
1. E2E 테스트 작성 (회원가입 → 로그인 → 보호된 페이지 접근)
2. 모든 테스트 실행 및 결과 보고

각 Phase가 끝날 때마다 중간 보고해줘.
```

Claude는 각 단계를 순차적으로 진행하면서, 테스트가 실패하면 스스로 수정하고, 성공하면 다음 단계로 넘어갑니다. 사용자는 각 Phase 종료 시점에만 결과를 확인하고 승인하면 됩니다.

### Week 2: 팀 협업 패턴 정립

이제 팀원들이 합류했다고 가정합니다. claude.md에 팀 협업 규칙을 추가합니다.

~~~markdown
# 팀 협업 규칙

## Git 워크플로우
- main 브랜치는 보호됨. 직접 push 금지
- 새 기능은 feature/기능명 브랜치에서 작업
- PR 생성 시 자동으로 테스트와 린트 실행
- 최소 1명의 리뷰어 승인 필요

## 커밋 메시지
- feat: 새 기능 추가
- fix: 버그 수정
- refactor: 코드 개선
- test: 테스트 추가/수정
- docs: 문서 업데이트

## PR 설명 템플릿
Claude가 PR을 생성할 때 다음 형식 사용:
```
## 변경 사항
- [변경 내용 요약]

## 테스트
- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 수동 테스트 완료

## 스크린샷 (UI 변경 시)
[브라우저 스크린샷]
```
~~~

이제 팀원들은 각자의 Claude 세션에서 동일한 규칙을 따르며 작업하고, claude.md는 Git을 통해 동기화됩니다.

## 고급 활용: 외부 도구 통합

보리스는 Claude를 Slack, BigQuery, Sentry와 연결해서 사용한다고 했습니다. 이를 구현하는 방법을 살펴보겠습니다.

### Slack 통합

Claude가 작업 결과를 Slack으로 보고하게 하려면, Slack Webhook URL을 환경 변수로 설정하고, 커스텀 명령어를 만듭니다.

`.claude/commands/report-to-slack.md`:

~~~markdown
# Slack 작업 보고

다음 정보를 Slack에 보고:
1. 수행한 작업 요약
2. 변경된 파일 목록
3. 테스트 결과
4. 발견된 문제점 (있다면)

curl을 사용해서 $SLACK_WEBHOOK_URL로 POST 요청:
```bash
curl -X POST $SLACK_WEBHOOK_URL \
  -H 'Content-Type: application/json' \
  -d '{
    "text": "🤖 Claude 작업 완료",
    "blocks": [...]
  }'
```
~~~

### Sentry 통합

에러 모니터링을 자동화하려면:

~~~markdown
# Sentry 에러 분석

1. Sentry API로 최근 24시간 에러 가져오기:
```bash
curl "https://sentry.io/api/0/projects/$ORG/$PROJECT/issues/" \
  -H "Authorization: Bearer $SENTRY_TOKEN"
```

2. 각 에러의 스택 트레이스 분석
3. 코드베이스에서 해당 파일 찾기
4. 가능하면 수정, 불가능하면 GitHub Issue 생성
5. 결과를 errors-report.md로 저장
~~~

## 성능 최적화 및 비용 관리

Claude Code를 팀 전체가 사용하면 API 비용이 증가할 수 있습니다. 보리스의 조언처럼 Opus 4.5를 사용하면 품질은 높아지지만 비용도 증가합니다. 이를 관리하는 전략이 필요합니다.

### 모델 선택 전략

claude.md에 다음과 같은 가이드라인을 추가할 수 있습니다.

~~~markdown
# 모델 선택 가이드

## Sonnet 사용 (빠르고 저렴)
- 간단한 코드 수정
- 스타일 변경 (CSS, formatting)
- 문서 작성
- 테스트 코드 작성

## Opus 4.5 사용 (느리지만 정확)
- 복잡한 비즈니스 로직 구현
- 보안 관련 코드
- 아키텍처 설계
- 디버깅 및 문제 해결

## Plan 모드 항상 사용
- 100줄 이상 코드 변경
- 여러 파일에 걸친 작업
- 데이터베이스 스키마 변경
~~~

### 캐싱 활용

자주 사용하는 컨텍스트는 캐싱을 활용해서 비용을 절감할 수 있습니다. claude.md나 공통 유틸리티 파일은 매 요청마다 전송되므로, Claude의 프롬프트 캐싱 기능을 활용하면 효과적입니다.

## 마무리: 점진적 도입 로드맵

이 모든 전략을 한 번에 도입하기는 어렵습니다. 다음과 같은 단계적 접근을 권장합니다.

**1주차: 기본 설정**
- claude.md 파일 생성 및 기본 규칙 작성
- 테스트 환경 구축 (Vitest 설정)
- 하나의 간단한 기능으로 TDD 워크플로우 테스트

**2주차: 워크플로우 자동화**
- 자주 쓰는 작업 2-3개를 커스텀 명령어로 만들기
- settings.json으로 명령어 허용/차단 설정
- 팀과 공유 및 피드백 수집

**3주차: 검증 루프 강화**
- 모든 새 코드에 테스트 우선 요청하기
- UI 작업 시 브라우저 검증 자동화
- claude.md에 발견된 실수 기록 시작

**4주차: 팀 확산**
- 팀 회고에서 Claude 활용 사례 공유
- claude.md 업데이트 프로세스 정립
- 외부 도구 통합 시작 (Slack, Sentry 등)

보리스의 워크플로우는 Claude를 단순한 코드 생성기가 아니라, 규칙을 이해하고 스스로 검증하며 여러 도구를 조율할 수 있는 자율적 에이전트로 활용하는 것입니다. 이를 위해서는 명확한 가이드라인(claude.md), 자가 검증 메커니즘(테스트), 그리고 반복 작업 자동화(커스텀 명령어)가 필수적입니다. 

이러한 시스템을 구축하는 데는 초기 투자가 필요하지만, 일단 자리를 잡으면 팀 전체의 생산성이 비약적으로 향상됩니다. 가장 중요한 것은 Claude를 '도구'가 아니라 '팀원'으로 대하며, 지속적으로 피드백하고 개선해나가는 것입니다.

---

**작성 일자: 2026-01-10**

---
## 참고영상

[**The One Rule Claude Code's Creator Never Breaks**](https://www.youtube.com/watch?v=B-UXpneKw6M) 이 영상은 Anthropic에서 **Claude Code**를 개발한 보리스(Boris)가 공유하는 효율적인 워크플로우와 팁을 담고 있습니다. 보리스가 직접 밝힌 Claude Code 활용법의 핵심 내용을 요약해 드립니다.

### 1. 검증 루프(Verification Loop) 구축 [00:40](http://www.youtube.com/watch?v=B-UXpneKw6M&t=40)

보리스는 Claude가 작업을 100% 완벽하게 끝내지 못하더라도 실망하지 말고, **스스로 작업을 검증할 수 있는 방법**을 제공하라고 조언합니다.

* **테스트 활용:** 파이썬 코드를 짤 때 테스트 코드도 함께 작성하도록 시키면, Claude가 피드백 루프를 통해 결과물의 품질을 즉시 높일 수 있습니다 [00:50](http://www.youtube.com/watch?v=B-UXpneKw6M&t=50).
* **UI 및 브라우저 검증:** Claude의 브라우저 확장 프로그램이나 시뮬레이터(iOS/Android)를 사용하여 UI가 제대로 작동하는지 확인하게 합니다 [01:26](http://www.youtube.com/watch?v=B-UXpneKw6M&t=86).

### 2. `claude.md` 파일의 중요성 [02:05](http://www.youtube.com/watch?v=B-UXpneKw6M&t=125)

각 프로젝트 폴더에 있는 `claude.md` 파일은 Claude 세션에 프로젝트의 핵심 맥락을 전달하는 역할을 합니다.

* **내용 구성:** 기술 스택, 프로젝트 구조, 코드 스타일, 그리고 **특히 '하지 말아야 할 일'** 을 기록합니다 [02:17](http://www.youtube.com/watch?v=B-UXpneKw6M&t=137).
* **지속적 업데이트:** 팀원들이 매주 여러 번 이 파일을 업데이트하며, Claude가 실수한 부분을 기록해 반복되지 않도록 관리합니다 [02:34](http://www.youtube.com/watch?v=B-UXpneKw6M&t=154).

### 3. 효율적인 모드 및 모델 선택 [04:56](http://www.youtube.com/watch?v=B-UXpneKw6M&t=296)

* **Plan 모드 우선:** 실제 구현에 들어가기 전에 항상 `plan` 모드에서 계획을 먼저 세우고, 계획이 타당한지 검증한 후에 작업을 시작합니다 [05:01](http://www.youtube.com/watch?v=B-UXpneKw6M&t=301).
* **Opus 4.5 활용:** 보리스는 속도가 조금 느리더라도 오류 발생률이 훨씬 낮은 **Opus 4.5(Thinking 기능 활성화)** 모델을 선호합니다. 결과적으로 수동으로 수정하는 시간을 줄여주기 때문입니다 [06:16](http://www.youtube.com/watch?v=B-UXpneKw6M&t=376).

### 4. 고급 활용 및 자동화 [04:22](http://www.youtube.com/watch?v=B-UXpneKw6M&t=262)

* **오케스트레이터로 활용:** Claude Code를 단순한 코딩 도구가 아니라 Slack, BigQuery, Sentry 등 다양한 도구를 제어하는 컨트롤러로 사용합니다.
* **병렬 세션 및 원격 작업:** 보리스는 5개의 터미널 탭에서 병렬로 세션을 실행하며 [05:22](http://www.youtube.com/watch?v=B-UXpneKw6M&t=322), 스마트폰에서도 클라우드 세션을 통해 작업을 지시하고 나중에 결과를 리뷰하는 방식으로 활용합니다 [05:50](http://www.youtube.com/watch?v=B-UXpneKw6M&t=350).

### 5. 팀 협업 및 공유 [03:48](http://www.youtube.com/watch?v=B-UXpneKw6M&t=228)

* `settings.json`을 통해 허용되거나 금지된 터미널 명령어를 설정하고 이를 팀과 공유하여 보안과 효율성을 동시에 챙깁니다 [03:54](http://www.youtube.com/watch?v=B-UXpneKw6M&t=234).
* 자주 반복되는 워크플로우는 슬래시 명령어(`/commands`)로 만들어 Git에 커밋해 팀원들과 공유합니다 [08:28](http://www.youtube.com/watch?v=B-UXpneKw6M&t=508).

보리스는 Claude를 단순한 도구가 아닌 **'주니어 개발자'** 처럼 대하며, 명확한 가이드라인과 검증 수단을 제공하는 것이 생산성 극대화의 비결이라고 강조합니다 [06:10](http://www.youtube.com/watch?v=B-UXpneKw6M&t=370)].

Claude Code의 주 개발자인 보리스(Boris)가 제안하는 핵심 전략은 단순히 인공지능에게 코드를 짜달라고 부탁하는 수준을 넘어, **Claude를 팀의 일원인 '주니어 개발자'로 대우하고 그에 맞는 시스템을 구축하는 것** 입니다.


---

**영상의 내용을 바탕으로 더 상세하게 서술해 드립니다.**


### 1. 신뢰하지 말고 검증하라: 테스트 주도 워크플로우

보리스는 Claude가 완벽할 것이라는 기대를 버려야 한다고 강조합니다. 대신, Claude가 스스로 본인의 실수를 바로잡을 수 있는 **'검증 장치'** 를 마련해 주는 것이 핵심입니다.

* **피드백 루프 생성:** 코드를 작성하라고 시키기 전에 "먼저 이 기능에 대한 테스트 코드를 작성하고, 코드가 완성되면 테스트를 통과할 때까지 스스로 수정하라"고 지시합니다. 이렇게 하면 사용자가 개입하지 않아도 Claude가 터미널 오류 메시지를 읽고 코드를 스스로 고쳐나갑니다.
* **시각적 확인:** 백엔드뿐만 아니라 UI 작업에서도 마찬가지입니다. 브라우저 도구나 모바일 시뮬레이터를 Claude가 직접 실행하게 하여, 의도한 대로 화면이 그려지는지 눈으로 확인하게 하는 과정이 포함되어야 합니다.

### 2. 프로젝트의 헌법, `claude.md` 파일

가장 실무적인 팁 중 하나는 프로젝트 루트 디렉토리에 **`claude.md`** 라는 가이드라인 파일을 두는 것입니다. 이는 Claude에게 프로젝트의 문맥(Context)을 주입하는 가장 강력한 방법입니다.

* **명확한 제약 조건:** 단순한 설명보다는 "이 라이브러리는 절대 쓰지 말 것", "변수명은 CamelCase를 유지할 것"과 같은 구체적인 규칙을 적습니다.
* **살아있는 문서:** 보리스의 팀은 매일 작업을 마치고 Claude가 반복적으로 실수했던 부분을 이 파일에 업데이트합니다. 이는 마치 신입 사원에게 "우리 회사는 이런 실수를 자주 하니 주의해라"라고 인수인계하는 것과 같습니다.

### 3. '생각'하는 모델의 가치 (Claude 3.7 및 4.5 Opus)

보리스는 속도보다 **정확도**를 우선시합니다. 많은 사용자가 빠른 응답을 선호하지만, 그는 'Thinking' 기능이 활성화된 고성능 모델(Claude 3.7 Sonnet 또는 4.5 Opus)을 주로 사용한다고 밝힙니다.

* **장기적 효율:** 모델이 1분 동안 고민해서 완벽한 코드를 내놓는 것이, 10초 만에 틀린 코드를 내놓아 사람이 10분 동안 수정하게 만드는 것보다 훨씬 효율적이라는 논리입니다.
* **Plan 모드 활용:** 작업을 시작하기 전 `/plan` 명령어를 통해 Claude가 무엇을 할지 먼저 나열하게 합니다. 사용자가 이 계획을 승인하거나 수정하는 과정을 거치면, 대규모 코드를 수정할 때 발생할 수 있는 엉뚱한 삽질을 사전에 방지할 수 있습니다.

### 4. 생산성을 극대화하는 멀티태스킹과 자동화

그는 Claude Code를 단순한 텍스트 창이 아니라 **강력한 자동화 엔진**으로 사용합니다.

* **병렬 처리:** 한 번에 하나의 채팅만 하는 것이 아니라, 여러 개의 터미널 탭을 열어두고 각각 다른 작업을 시킵니다. 예를 들어 A 탭에서는 버그 수정을, B 탭에서는 새로운 기능 구현을 동시에 진행하는 방식입니다.
* **도구의 연결:** Claude가 터미널 명령어뿐만 아니라 회사의 내부 DB(BigQuery), 오류 로그 시스템(Sentry), 협업 도구(Slack)에 접근할 수 있도록 권한을 설정합니다. 이를 통해 Claude는 "지난밤 발생한 에러 로그를 분석해서 수정하고 Slack으로 보고해"라는 복잡한 임무를 수행할 수 있게 됩니다.

### 5. 원격 근무의 확장 (Mobile Workflow)

흥미로운 점은 보리스가 이동 중에도 Claude Code를 활용한다는 것입니다.

* **SSH를 통한 접근:** 스마트폰이나 태블릿으로 서버에 접속하여 Claude에게 음성이나 텍스트로 복잡한 코딩 지시를 내립니다.
* **비동기 작업:** "이 기능을 구현해두고 테스트까지 끝내놔"라고 지시한 뒤, 나중에 컴퓨터 앞에 앉았을 때 Claude가 만들어 놓은 결과물과 테스트 로그를 리뷰만 하는 방식으로 시간을 아낍니다.

---

**결론적으로** 보리스의 워크플로우는 Claude를 단순한 '도구'가 아니라, **규칙을 이해하고 스스로 검증하며 여러 도구를 다룰 줄 아는 '지능형 에이전트'** 로 활용하는 데 초점이 맞춰져 있습니다.

