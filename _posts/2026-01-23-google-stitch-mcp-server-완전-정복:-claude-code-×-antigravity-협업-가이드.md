---
title: "Google Stitch MCP Server 완전 정복: Claude Code × Antigravity 협업 가이드"
date: 2026-01-23 07:00:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Guide,  Google,  google-cloud,  Stitch,  MCP,  Antigravity,  claude-code,  Claude.write]
---


> AI 도구 협업으로 디자인부터 프로덕션까지 완전 자동화하기

## 목차

1. [개요](#개요)
2. [Stitch MCP란 무엇인가](#stitch-mcp란-무엇인가)
3. [왜 두 도구를 함께 쓰는가](#왜-두-도구를-함께-쓰는가)
4. [환경 설정](#환경-설정)
5. [단독 사용 가이드](#단독-사용-가이드)
6. [협업 워크플로우](#협업-워크플로우)
7. [실전 프로젝트 예제](#실전-프로젝트-예제)
8. [고급 활용 패턴](#고급-활용-패턴)
9. [문제 해결](#문제-해결)
10. [베스트 프랙티스](#베스트-프랙티스)

---

## 개요

### 무엇을 배울 수 있나요?

이 가이드를 마치면 다음을 할 수 있습니다:

- ✅ **Stitch MCP를 Claude Code와 Antigravity에 설정**
- ✅ **각 도구의 강점을 이해하고 적재적소에 활용**
- ✅ **두 AI 도구를 협업시켜 생산성 극대화**
- ✅ **디자인부터 배포까지 완전 자동화 워크플로우 구축**
- ✅ **실전 프로젝트에 바로 적용 가능한 패턴 습득**

### 필요한 사전 지식

**최소 요구사항:**
- 기본적인 터미널 사용법
- HTML/CSS 기초 (읽을 수 있는 수준)
- Git 기본 명령어

**권장 사항:**
- React/Vue 같은 프레임워크 경험
- 디자인 도구(Figma 등) 사용 경험
- API 개념 이해

하지만 걱정 마세요! 모든 단계를 상세히 설명합니다.

---

## Stitch MCP란 무엇인가

### MCP (Model Context Protocol)

MCP는 AI 모델이 외부 도구와 통신하는 표준 프로토콜입니다.

**전통적인 방식:**
```
사용자 → AI → 응답
```

AI는 학습된 지식만 사용할 수 있습니다.

**MCP 방식:**
```
사용자 → AI ↔ MCP Server (Stitch, Drive, Slack 등) → 응답
```

AI가 실시간으로 외부 도구를 사용할 수 있습니다.

### Google Stitch MCP Server

Stitch는 **UI 디자인 생성 전문 MCP 서버**입니다.

**핵심 기능:**

1. **자연어 → UI 디자인**
   - "대시보드 만들어줘" → 완성된 UI 디자인
   
2. **디자인 → 코드 변환**
   - UI 디자인 → HTML/CSS/React 코드
   
3. **반복 개선**
   - 피드백 → 디자인 수정 → 재생성

4. **다양한 출력 형식**
   - HTML/CSS (정적 사이트)
   - React (SPA)
   - Vue (SPA)
   - Tailwind CSS

**Stitch의 강점:**

🎨 **전문성** - UI/UX 디자인에 특화
⚡ **속도** - 몇 초 만에 프로페셔널 디자인
🔄 **반복** - 무한 수정 가능
📱 **반응형** - 모든 화면 크기 지원
♿ **접근성** - WCAG 표준 준수

---

## 왜 두 도구를 함께 쓰는가

### Claude Code vs Antigravity vs 협업

각 도구의 강점을 이해하고 조합하면 시너지 효과가 극대화됩니다.

#### Claude Code의 강점

**1. 코드 품질**
- 깔끔하고 유지보수 가능한 코드
- 베스트 프랙티스 자동 적용
- 에러 처리 완벽

**2. 복잡한 로직**
- 비즈니스 로직 구현
- 알고리즘 최적화
- 아키텍처 설계

**3. 문서화**
- 자동 주석 생성
- README 작성
- API 문서화

**4. 테스트**
- 유닛 테스트 작성
- 통합 테스트
- E2E 테스트

**5. 리팩토링**
- 코드 개선
- 성능 최적화
- 기술 부채 해소

#### Antigravity의 강점

**1. Google 에코시스템**
- Firebase 통합
- GCP 서비스 활용
- Google Drive 연동

**2. 멀티모달**
- 이미지 → 코드
- 스케치 → UI
- 디자인 파일 분석

**3. 실시간 웹 검색**
- 최신 라이브러리 정보
- 베스트 프랙티스 업데이트
- 에러 해결 검색

**4. 긴 컨텍스트**
- 2M 토큰 (Claude의 10배)
- 대규모 코드베이스 분석
- 전체 프로젝트 이해

**5. Firebase 자동화**
- 즉시 백엔드 설정
- 인증 시스템 구축
- 데이터베이스 스키마

### 협업 시나리오

두 도구를 함께 쓰면 각자의 약점을 보완합니다.

**시나리오 1: 프론트엔드 중심 프로젝트**

```
Antigravity (Stitch) → UI 디자인 생성
↓
Claude Code → 코드 품질 개선, 테스트 작성
↓
Antigravity → Firebase 백엔드 연동
↓
Claude Code → 최종 리팩토링, 문서화
```

**시나리오 2: 풀스택 프로젝트**

```
Claude Code → 아키텍처 설계, API 명세
↓
Antigravity (Stitch) → UI 디자인 + 코드 생성
↓
Claude Code → 백엔드 로직 구현
↓
Antigravity → Firebase/GCP 배포
↓
Claude Code → 테스트, 문서화
```

**시나리오 3: 빠른 프로토타입**

```
Antigravity (Stitch) → 즉시 UI 생성
↓
Antigravity → Firebase 백엔드 설정
↓
Claude Code → 핵심 로직만 정교화
```

### 도구 선택 가이드

**Stitch + Claude Code를 선택하세요:**
- 코드 품질이 최우선
- 복잡한 비즈니스 로직
- 테스트 커버리지 중요
- 장기 유지보수 고려

**Stitch + Antigravity를 선택하세요:**
- Firebase/GCP 사용
- 빠른 프로토타입 필요
- 이미지/디자인 파일 활용
- Google 에코시스템 중심

**세 도구 모두 사용하세요:**
- 대규모 프로젝트
- 최고 품질 요구
- 팀 협업
- 프로덕션 배포

---

## 환경 설정

### 사전 준비

**필수 설치:**

1. Node.js 18+ 및 npm
2. Google Cloud 계정
3. Claude Code 또는/그리고 Antigravity

**확인 명령어:**

```bash
node --version
npm --version
```

예상 출력:
```
v20.11.0
v10.2.4
```

### Claude Code에 Stitch MCP 설정

#### 1단계: Stitch MCP 초기화

```bash
npx -y davideast/stitch-mcp init
```

과정:
1. Google Cloud 프로젝트 생성
2. API 활성화
3. OAuth 인증
4. 자격증명 저장

**상세 과정:**

프로젝트 ID 입력 프롬프트:
```
? Enter a unique project ID: stitch-claude-project-2026
```

팁:
- 전역적으로 고유해야 함
- 6-30자, 소문자/숫자/하이픈만
- 의미 있는 이름 권장

OAuth 인증:
```
브라우저가 열리고 Google 로그인
→ 권한 승인
→ 인증 완료
```

#### 2단계: Claude Code 설정 파일 편집

**설정 파일 위치:**

macOS/Linux:
```bash
~/.claude-code/config.json
```

Windows:
```powershell
%USERPROFILE%\.claude-code\config.json
```

**파일 열기:**

```bash
code ~/.claude-code/config.json
```

또는:

```bash
nano ~/.claude-code/config.json
```

**추가할 내용:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": [
        "-y",
        "davideast/stitch-mcp"
      ],
      "env": {}
    }
  }
}
```

설명:
- `mcpServers`: MCP 서버 목록
- `stitch`: 서버 식별자 (원하는 이름 가능)
- `command`: 실행 명령어
- `args`: 명령 인자 배열
- `env`: 환경 변수 (선택사항)

**기존 설정이 있는 경우:**

기존:
```json
{
  "mcpServers": {
    "google-drive": {
      "command": "node",
      "args": ["/path/to/drive-mcp"]
    }
  }
}
```

수정 후 (쉼표 주의):
```json
{
  "mcpServers": {
    "google-drive": {
      "command": "node",
      "args": ["/path/to/drive-mcp"]
    },
    "stitch": {
      "command": "npx",
      "args": ["-y", "davideast/stitch-mcp"]
    }
  }
}
```

#### 3단계: Claude Code 재시작

```bash
claude quit
claude
```

**연결 확인:**

```
Claude Code v2.4.1

🔌 MCP Servers:
  ✓ stitch (connected)
  ✓ google-drive (connected)

Ready!
```

### Antigravity에 Stitch MCP 설정

#### 1단계: Stitch MCP 초기화 (동일)

Claude Code와 동일한 과정:

```bash
npx -y davideast/stitch-mcp init
```

참고: 이미 Claude Code를 위해 초기화했다면 건너뛰어도 됩니다.

#### 2단계: Antigravity 설정 파일 편집

**설정 파일 위치:**

macOS/Linux:
```bash
~/.antigravity/config.json
```

Windows:
```powershell
%USERPROFILE%\.antigravity\config.json
```

**파일 열기:**

```bash
code ~/.antigravity/config.json
```

**추가할 내용:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": [
        "-y",
        "davideast/stitch-mcp"
      ]
    }
  }
}
```

#### 3단계: Antigravity 재시작

```bash
antigravity quit
antigravity
```

**연결 확인:**

```
Antigravity v1.8.2 | Gemini 3 Pro

🔌 MCP Servers connected:
  ✓ stitch (davideast/stitch-mcp)

Ready!
```

### 두 도구 모두에 설정하기

**권장 방법:**

1. 먼저 Claude Code에 설정
2. 테스트 후 Antigravity에도 설정
3. 각 도구에서 간단한 테스트 수행

**테스트 명령:**

Claude Code:
```bash
claude

You: Stitch MCP가 연결되어 있는지 확인해줘

Claude: ✓ Stitch MCP 서버가 정상적으로 연결되어 있습니다.
```

Antigravity:
```bash
antigravity

You: Stitch를 사용할 수 있나요?

Antigravity: 네, Stitch MCP 서버가 연결되어 있습니다. 
             UI 디자인을 생성할 수 있습니다.
```

---

## 단독 사용 가이드

각 도구에서 Stitch를 단독으로 사용하는 방법입니다.

### Claude Code에서 Stitch 사용

Claude Code는 Stitch를 자동으로 감지하고 적절한 상황에서 사용합니다.

#### 기본 사용법

```bash
claude

You: 관리자 대시보드 UI를 디자인해줘.
     왼쪽에 사이드바, 상단에 헤더, 
     메인 영역에 통계 카드들을 배치해.
```

**Claude의 응답:**

```
Stitch MCP를 사용하여 디자인을 생성하겠습니다.

[Stitch 호출 중...]

✓ 디자인 생성 완료

생성된 컴포넌트:
- Sidebar (240px width, dark theme)
- Header (64px height, user info)
- Stats Grid (4 cards, 2x2)
- Main content area

? HTML/CSS 코드를 생성할까요? (Y/n)
```

#### 코드 생성 및 개선

```
You: 네, 코드를 생성하고 
     React 컴포넌트로 분리해줘.
```

**Claude의 작업:**

1. Stitch로 디자인 → HTML/CSS 생성
2. Claude가 React로 변환
3. 컴포넌트 분리
4. PropTypes 추가
5. 파일 구조 생성

```
✓ src/components/Sidebar.jsx
✓ src/components/Header.jsx
✓ src/components/StatsCard.jsx
✓ src/components/Dashboard.jsx
✓ src/App.jsx

각 컴포넌트에 다음을 추가했습니다:
- PropTypes 검증
- 기본 props
- JSDoc 주석
- 에러 바운더리

다음 단계:
1. npm install prop-types
2. npm start

테스트를 작성할까요?
```

#### 반복 개선

```
You: 좋아. 근데 사이드바가 너무 좁은 것 같아.
     280px로 늘리고, 아이콘도 추가해줘.
```

**Claude의 작업:**

```
디자인을 업데이트하겠습니다.

[Stitch 재호출...]

✓ 사이드바 너비 280px로 변경
✓ 아이콘 추가 (react-icons 사용)

변경된 파일:
- src/components/Sidebar.jsx

추가 설치 필요:
npm install react-icons

? 설치할까요? (Y/n)
```

### Antigravity에서 Stitch 사용

Antigravity는 Gemini의 강력한 멀티모달 능력과 Stitch를 결합합니다.

#### 기본 사용법

```bash
antigravity

You: 이커머스 제품 카드를 디자인해줘.
     최신 디자인 트렌드를 검색해서 반영해.
```

**Antigravity의 응답:**

```
[웹 검색: "ecommerce product card design trends 2026"]

최신 트렌드 분석:
- 큰 이미지 비율 (1:1 또는 4:5)
- 미니멀한 정보 표시
- 마이크로 인터랙션
- 그라데이션 배경

[Stitch MCP 호출...]

✓ 트렌디한 제품 카드 디자인 생성

특징:
- 4:5 이미지 비율
- Glassmorphism 효과
- 호버 시 3D 틸트
- 부드러운 애니메이션

미리보기: [URL]

? 코드를 생성할까요?
```

#### 이미지 기반 디자인

Antigravity의 멀티모달 능력 활용:

```
You: [스케치 이미지 업로드]
     이 스케치를 참고해서 랜딩 페이지를 만들어줘.
```

**Antigravity의 작업:**

```
[이미지 분석 중...]

스케치에서 인식한 요소:
- Hero 섹션 (큰 제목 + CTA)
- 3컬럼 Feature 섹션
- Testimonial 카드들
- Footer (소셜 링크)

[Stitch로 디자인 생성...]

✓ 스케치 기반 랜딩 페이지 생성

추가 개선:
- 반응형 레이아웃
- 스크롤 애니메이션
- 최적화된 색상 팔레트

? Firebase Hosting으로 배포할까요?
```

#### Firebase 통합

Antigravity만의 강점:

```
You: 네, 배포하고 
     Contact 폼에 Firestore 연동도 해줘.
```

**Antigravity의 작업:**

```
1. Firebase 프로젝트 초기화
✓ 프로젝트: landing-page-a3b2c

2. Firestore 설정
✓ 컬렉션: contacts
✓ 보안 규칙: 작성 전용

3. Contact 폼 연동
✓ 실시간 검증
✓ Firestore 저장
✓ 성공 메시지 표시

4. Firebase Hosting 배포
✓ 빌드 완료
✓ 배포 완료

🎉 배포 URL: https://landing-page-a3b2c.web.app

다음 단계:
- Analytics 설정
- 커스텀 도메인
- Performance 모니터링
```

---

## 협업 워크플로우

두 AI 도구를 협업시키는 실전 워크플로우입니다.

### 워크플로우 1: 순차적 협업

**개념:** 한 도구가 끝나면 다음 도구로 전달

**장점:**
- 각 도구의 강점 극대화
- 명확한 책임 분리
- 결과물 품질 최상

**사용 시기:**
- 품질이 최우선
- 시간 여유 있음
- 복잡한 프로젝트

#### 단계별 가이드

**1단계: Antigravity로 디자인 + 백엔드**

```bash
antigravity

You: SaaS 앱의 대시보드를 만들어줘.
     
     기능:
     - 사용자 통계 (실시간)
     - 최근 활동 목록
     - 월별 매출 차트
     
     Firebase로 백엔드 설정도 해줘.
```

**Antigravity 작업:**

```
Phase 1: 디자인 생성 (Stitch)
✓ 대시보드 레이아웃
✓ 통계 카드 컴포넌트
✓ 활동 목록
✓ 차트 영역

Phase 2: Firebase 설정
✓ Firestore 데이터베이스
✓ 컬렉션 구조:
  - users/ (사용자 정보)
  - activities/ (활동 로그)
  - revenue/ (매출 데이터)

Phase 3: 코드 생성 (React)
✓ src/ 폴더 구조
✓ Firebase 연동 코드
✓ 실시간 업데이트 로직

파일이 준비되었습니다.
? Claude Code로 품질 개선을 할까요?
```

**2단계: Claude Code로 품질 개선**

작업 디렉토리로 이동:

```bash
cd dashboard-app
claude
```

**Claude Code에 작업 요청:**

```
You: Antigravity가 생성한 이 프로젝트를 분석하고
     다음을 개선해줘:
     
     1. 코드 품질 향상
     2. 에러 처리 추가
     3. 테스트 작성
     4. 성능 최적화
     5. 문서화
```

**Claude Code 작업:**

```
프로젝트 분석 중...

발견된 개선 사항:
1. PropTypes 누락 (7개 컴포넌트)
2. 에러 바운더리 없음
3. 메모이제이션 기회 (5곳)
4. 테스트 커버리지 0%

개선 작업 시작...

✓ PropTypes 추가 (모든 컴포넌트)
✓ ErrorBoundary 컴포넌트 생성 및 적용
✓ React.memo() 적용 (성능 최적화)
✓ useMemo/useCallback 적용
✓ Jest 테스트 작성 (커버리지 85%)
✓ README.md 작성 (설치, 사용법, API)
✓ JSDoc 주석 추가

개선 완료!

변경 사항 요약:
- 15개 파일 수정
- 23개 파일 추가 (테스트)
- 1개 파일 추가 (문서)

? Git 커밋을 만들까요?
```

**3단계: Git 관리 및 배포**

```
You: 네, 의미있는 커밋들로 나눠서 만들어줘.
```

**Claude Code 작업:**

```
Git 저장소 분석 중...

커밋 계획:
1. feat: Add PropTypes to all components
2. feat: Implement error boundaries
3. perf: Add memoization for performance
4. test: Add Jest test suite (85% coverage)
5. docs: Add comprehensive README

? 이 계획대로 커밋할까요? (Y/n)

[Y 선택]

✓ 5개 커밋 생성
✓ 각 커밋은 의미있는 단위
✓ Conventional Commits 형식

현재 브랜치: main
? Vercel에 배포할까요?
```

**결과:**

- Antigravity: 빠른 프로토타입 + Firebase 백엔드
- Claude Code: 프로덕션 품질 코드 + 테스트 + 문서
- 최종: 배포 가능한 완성품

### 워크플로우 2: 병렬 협업

**개념:** 각 도구가 다른 부분을 동시에 작업

**장점:**
- 시간 절약
- 역할 분담 명확
- 빠른 프로토타입

**사용 시기:**
- 빠른 결과 필요
- 팀 프로젝트
- 모듈화된 구조

#### 단계별 가이드

**준비: 작업 분할**

프로젝트를 분석하고 역할 분담:

```
프로젝트: 블로그 플랫폼

Antigravity 담당:
- UI 디자인 및 생성 (Stitch)
- Firebase 백엔드
- 인증 시스템
- 배포

Claude Code 담당:
- 비즈니스 로직
- 에디터 기능 (rich text)
- 검색 기능
- 테스트
```

**실행: 동시 작업**

**터미널 1 - Antigravity:**

```bash
antigravity

You: 블로그 플랫폼의 UI를 디자인하고 
     Firebase 백엔드를 설정해줘.
     
     페이지:
     - 홈 (포스트 목록)
     - 포스트 상세
     - 에디터 (나중에 통합할 것)
     - 프로필
     
     Firebase:
     - Authentication (Email/Google)
     - Firestore (posts, users, comments)
     - Storage (이미지)
```

**터미널 2 - Claude Code (동시에):**

```bash
claude

You: 블로그 에디터 기능을 구현해줘.
     
     요구사항:
     - Rich text 편집 (제목, 본문, 서식)
     - 마크다운 지원
     - 이미지 업로드
     - 자동 저장
     - 실시간 미리보기
     
     라이브러리: Slate.js 또는 TipTap 사용
     테스트 포함
```

**통합 단계:**

두 작업이 완료되면 통합:

```bash
터미널 1 (Antigravity):

You: 에디터 컴포넌트를 통합할 준비가 됐어.
     Claude Code가 만든 Editor.jsx를 가져와서
     에디터 페이지에 넣어줘.
```

```bash
터미널 2 (Claude Code):

You: Antigravity가 만든 UI 구조를 분석하고
     우리 에디터를 자연스럽게 통합해줘.
     스타일도 일치시켜.
```

**최종 배포:**

```bash
antigravity

You: 모든 것이 통합됐으니 배포해줘.
     
     체크리스트:
     - 프로덕션 빌드
     - 환경 변수 설정
     - Firebase Hosting
     - 커스텀 도메인
```

### 워크플로우 3: 반복적 협업

**개념:** 도구들이 번갈아가며 개선

**장점:**
- 최고 품질
- 지속적 개선
- 각 도구 장점 극대화

**사용 시기:**
- 중요한 프로젝트
- 지속적 개선 필요
- 높은 품질 기준

#### 반복 사이클

**Iteration 1: 기본 구조**

```
Antigravity (Stitch):
→ 초기 디자인 생성
→ 기본 Firebase 설정

Claude Code:
→ 코드 구조 개선
→ TypeScript 적용
→ 기본 테스트
```

**Iteration 2: 기능 추가**

```
Antigravity:
→ 새 디자인 컴포넌트
→ Firebase Functions 추가
→ 실시간 기능

Claude Code:
→ 복잡한 로직 구현
→ 상태 관리 (Redux/Zustand)
→ 통합 테스트
```

**Iteration 3: 최적화**

```
Claude Code:
→ 성능 프로파일링
→ 최적화 (memo, lazy loading)
→ 번들 크기 감소

Antigravity:
→ Firebase 최적화
→ CDN 설정
→ 모니터링
```

**Iteration 4: 완성도**

```
Claude Code:
→ 엣지 케이스 처리
→ 접근성 개선
→ 문서화 완성

Antigravity:
→ SEO 최적화
→ Analytics 통합
→ 최종 배포
```

#### 실전 예제

**프로젝트: 실시간 채팅 앱**

**Iteration 1 (1일차):**

Antigravity:
```
✓ 채팅 UI 디자인 (Stitch)
✓ Firebase Authentication
✓ Firestore 실시간 DB
✓ 기본 메시지 전송
```

Claude Code:
```
✓ TypeScript 전환
✓ 메시지 타입 정의
✓ 유효성 검증 로직
✓ 유닛 테스트 (50%)
```

**Iteration 2 (2일차):**

Antigravity:
```
✓ 파일 업로드 (Storage)
✓ 읽음 표시 기능
✓ 타이핑 인디케이터
✓ 푸시 알림 (FCM)
```

Claude Code:
```
✓ WebSocket 최적화
✓ 메시지 캐싱
✓ 무한 스크롤
✓ 통합 테스트 추가
```

**Iteration 3 (3일차):**

Claude Code:
```
✓ 성능 최적화
✓ 메모리 누수 수정
✓ 번들 크기 40% 감소
✓ E2E 테스트 (Playwright)
```

Antigravity:
```
✓ Firebase 규칙 최적화
✓ Cloud Functions 추가
✓ 프로덕션 배포
✓ Analytics 설정
```

**최종 결과:**

- 개발 기간: 3일
- 테스트 커버리지: 85%
- 성능 점수: 95+
- 프로덕션 배포 완료

---

## 실전 프로젝트 예제

완전한 프로젝트를 처음부터 끝까지 진행해봅시다.

### 프로젝트: SaaS 랜딩 페이지 + 대시보드

**목표:** 완전히 작동하는 SaaS 플랫폼

**기능:**
- 마케팅 랜딩 페이지
- 사용자 인증
- 프로필 대시보드
- 결제 연동 (Stripe)
- 관리자 대시보드

**기술 스택:**
- Frontend: React + Tailwind
- Backend: Firebase
- Payments: Stripe
- Hosting: Vercel

#### Phase 1: 랜딩 페이지 (Antigravity + Stitch)

**Day 1 - Morning:**

```bash
antigravity

You: SaaS 제품의 랜딩 페이지를 만들어줘.
     
     제품: "TaskMaster" - AI 기반 작업 관리 도구
     
     섹션:
     1. Hero (강렬한 헤드라인 + CTA)
     2. Features (3개 주요 기능, 아이콘 포함)
     3. How It Works (3단계 프로세스)
     4. Pricing (3 티어: Free, Pro, Enterprise)
     5. Testimonials (고객 후기 3개)
     6. CTA (무료 시작하기)
     7. Footer
     
     디자인: 모던, 미니멀, 파란색 계열
     반응형 필수
```

**Antigravity 작업:**

```
[Stitch MCP 호출...]

✓ TaskMaster 랜딩 페이지 디자인 완료

컴포넌트:
- Hero (gradient background, animated CTA)
- FeatureCard (icon + title + description)
- ProcessStep (numbered steps, icons)
- PricingCard (3 tiers, highlight Pro)
- TestimonialCard (avatar, quote, name, role)
- Footer (links, social icons)

색상 팔레트:
- Primary: #3B82F6 (Blue)
- Secondary: #8B5CF6 (Purple)
- Accent: #06B6D4 (Cyan)

타이포그래피:
- Heading: Inter Bold
- Body: Inter Regular

? React + Tailwind로 코드 생성할까요?

[Y 선택]

✓ 코드 생성 완료
✓ 파일 저장: ./taskmaster-landing/

구조:
src/
├── components/
│   ├── Hero.jsx
│   ├── Features.jsx
│   ├── HowItWorks.jsx
│   ├── Pricing.jsx
│   ├── Testimonials.jsx
│   └── Footer.jsx
├── App.jsx
└── index.css

? 로컬 서버를 시작할까요?

[Y 선택]

✓ npm install 완료
✓ 개발 서버 시작: http://localhost:3000

브라우저에서 확인하세요!
```

**Day 1 - Afternoon: Claude Code로 개선:**

```bash
cd taskmaster-landing
claude

You: Antigravity가 만든 랜딩 페이지를 분석하고
     다음을 개선해줘:
     
     1. SEO 최적화 (메타 태그, schema.org)
     2. 접근성 (WCAG 2.1 AA)
     3. 성능 (lazy loading, 이미지 최적화)
     4. 애니메이션 추가 (스크롤 애니메이션)
     5. Analytics 통합 준비
```

**Claude Code 작업:**

```
프로젝트 분석 완료.

개선 계획:

1. SEO 최적화
✓ index.html에 메타 태그 추가
✓ Open Graph 태그
✓ Twitter Card 태그
✓ Schema.org JSON-LD (SoftwareApplication)
✓ robots.txt, sitemap.xml 생성

2. 접근성
✓ ARIA 레이블 추가 (모든 interactive 요소)
✓ 키보드 네비게이션 개선
✓ Skip to content 링크
✓ 색상 대비 비율 체크 및 조정
✓ alt 텍스트 개선

3. 성능
✓ React.lazy() 적용 (code splitting)
✓ 이미지 lazy loading
✓ Intersection Observer 활용
✓ CSS 최적화 (purge unused)

4. 애니메이션
✓ Framer Motion 설치 및 설정
✓ Fade-in on scroll
✓ Slide-in animations
✓ Stagger effects

5. Analytics
✓ Google Analytics 4 설정 파일
✓ 이벤트 트래킹 헬퍼 함수
✓ 페이지뷰 자동 추적

? 적용할까요? (Y/n)

[Y 선택]

작업 중...

✓ 21개 파일 수정
✓ 8개 파일 추가
✓ package.json 업데이트 (framer-motion 추가)

Lighthouse 점수:
- Performance: 65 → 95
- Accessibility: 78 → 100
- Best Practices: 83 → 100
- SEO: 70 → 100

? Git 커밋을 만들까요?
```

#### Phase 2: 인증 시스템 (Antigravity)

**Day 2 - Morning:**

```bash
antigravity

You: 로그인/회원가입 페이지를 추가해줘.
     Firebase Authentication 사용.
     
     기능:
     - 이메일/비밀번호
     - Google 로그인
     - 비밀번호 재설정
     - 이메일 인증
     
     UI는 랜딩 페이지와 일관된 디자인으로.
```

**Antigravity 작업:**

```
Phase 1: Firebase 설정
✓ Firebase 프로젝트: taskmaster-prod-a2b3c
✓ Authentication 활성화:
  - Email/Password
  - Google Sign-In

Phase 2: UI 디자인 (Stitch)
✓ 로그인 페이지
✓ 회원가입 페이지
✓ 비밀번호 재설정 페이지
✓ 이메일 인증 확인 페이지

디자인 일관성:
✓ 동일한 색상 팔레트
✓ 동일한 타이포그래피
✓ 일관된 버튼 스타일

Phase 3: 코드 생성
✓ Firebase 설정 파일
✓ Auth 컨텍스트 (React Context)
✓ 로그인/회원가입 폼
✓ Protected Route 컴포넌트

파일 구조:
src/
├── contexts/
│   └── AuthContext.jsx
├── pages/
│   ├── Login.jsx
│   ├── Signup.jsx
│   └── ResetPassword.jsx
├── components/
│   └── ProtectedRoute.jsx
└── config/
    └── firebase.js

? 적용하고 테스트할까요?

[Y 선택]

✓ 코드 통합 완료
✓ Firebase 환경 변수 설정
✓ 로컬 테스트 성공

테스트 계정:
- Email: test@example.com
- Password: Test123!

? Claude Code로 보안 강화할까요?
```

**Day 2 - Afternoon: Claude Code로 보안 강화:**

```bash
claude

You: 인증 시스템의 보안을 강화해줘.
     
     체크리스트:
     - 입력 검증 (이메일, 비밀번호 강도)
     - XSS 방어
     - CSRF 방어
     - Rate limiting
     - 에러 메시지 개선 (정보 누출 방지)
     - 세션 관리
     - 테스트 작성
```

**Claude Code 작업:**

```
보안 분석 중...

발견된 취약점:
1. 비밀번호 강도 검증 없음
2. 에러 메시지가 너무 상세 (정보 누출)
3. Rate limiting 없음
4. 토큰 갱신 로직 없음

보안 강화 작업:

1. 입력 검증
✓ Zod 스키마 적용
✓ 비밀번호 강도: 8자 이상, 대소문자+숫자+특수문자
✓ 이메일 형식 검증
✓ 실시간 유효성 피드백

2. XSS/CSRF 방어
✓ DOMPurify 적용
✓ CSP 헤더 설정
✓ CSRF 토큰 (Firebase 자동 처리)

3. Rate Limiting
✓ 로그인 시도: 5회/15분
✓ 비밀번호 재설정: 3회/시간
✓ 회원가입: 10회/시간

4. 에러 처리
✓ 일반화된 에러 메시지
✓ 로그는 서버에만 기록
✓ 사용자 친화적 메시지

5. 세션 관리
✓ 토큰 자동 갱신
✓ 30분 비활동 시 로그아웃
✓ 다중 디바이스 세션 관리

6. 테스트
✓ Auth 로직 유닛 테스트
✓ 통합 테스트
✓ 보안 테스트 (OWASP Top 10)

? 적용할까요?

[Y 선택]

✓ 보안 강화 완료
✓ 테스트 커버리지: 90%

보안 체크리스트:
✅ OWASP Top 10 대응
✅ 입력 검증
✅ 출력 인코딩
✅ 인증/인가
✅ 세션 관리
✅ 암호화
✅ 에러 처리
✅ 로깅/모니터링
```

#### Phase 3: 사용자 대시보드 (협업)

**Day 3: 병렬 작업:**

**터미널 1 - Antigravity:**

```bash
antigravity

You: 사용자 대시보드를 만들어줘.
     
     기능:
     - 개인 정보 편집
     - 구독 관리
     - 사용량 통계
     - 최근 활동
     
     Firebase Firestore에 데이터 저장.
```

**터미널 2 - Claude Code (동시에):**

```bash
claude

You: 대시보드의 통계 차트를 구현해줘.
     
     라이브러리: Recharts
     차트 종류:
     - 월별 작업 완료 추이 (선 그래프)
     - 카테고리별 분포 (파이 차트)
     - 주간 생산성 (막대 그래프)
     
     실시간 업데이트 지원.
```

**통합:**

```bash
antigravity

You: Claude Code가 만든 차트 컴포넌트들을
     대시보드에 통합해줘.
```

#### Phase 4: 결제 연동 (Claude Code)

**Day 4:**

```bash
claude

You: Stripe 결제를 연동해줘.
     
     플랜:
     - Free: $0
     - Pro: $19/월
     - Enterprise: 커스텀
     
     기능:
     - 결제 페이지
     - 구독 관리
     - 영수증 발행
     - Webhook 처리
     
     테스트 모드로 구현.
```

**Claude Code 작업:**

```
Stripe 연동 계획:

1. 서버 사이드 (Firebase Functions)
✓ 결제 세션 생성 API
✓ Webhook 핸들러
✓ 구독 상태 동기화

2. 클라이언트
✓ Stripe.js 통합
✓ 결제 폼
✓ 구독 관리 UI

3. 보안
✓ API 키 환경 변수화
✓ Webhook 서명 검증
✓ 민감 정보 암호화

구현 중...

✓ functions/src/stripe.ts 생성
✓ 결제 세션 API
✓ Webhook 엔드포인트
✓ 클라이언트 결제 페이지
✓ 구독 관리 대시보드

테스트 카드:
- 성공: 4242 4242 4242 4242
- 실패: 4000 0000 0000 0002

? Firebase Functions 배포할까요?
```

#### Phase 5: 최종 배포 (Antigravity)

**Day 5:**

```bash
antigravity

You: 전체 앱을 프로덕션에 배포해줘.
     
     체크리스트:
     - 환경 변수 설정
     - 프로덕션 빌드
     - Firebase Hosting
     - 커스텀 도메인 (taskmaster.com)
     - SSL 인증서
     - CDN 설정
```

**Antigravity 작업:**

```
배포 체크리스트 실행:

1. 환경 변수
✓ .env.production 생성
✓ Firebase 프로덕션 키
✓ Stripe 프로덕션 키
✓ Analytics ID

2. 빌드 최적화
✓ 프로덕션 빌드: npm run build
✓ 번들 크기 분석
✓ 트리 쉐이킹 확인
✓ 최종 크기: 145KB (gzip)

3. Firebase 배포
✓ firebase deploy --only hosting
✓ 배포 완료: https://taskmaster-prod-a2b3c.web.app

4. 커스텀 도메인
✓ DNS 설정 가이드 제공
✓ taskmaster.com → Firebase
✓ SSL 자동 프로비저닝

5. CDN
✓ Firebase Hosting CDN 활성화
✓ 전세계 엣지 로케이션
✓ 캐싱 설정 최적화

6. 모니터링
✓ Firebase Analytics
✓ Performance Monitoring
✓ Crashlytics
✓ 대시보드: [URL]

🎉 배포 완료!

프로덕션 URL: https://taskmaster.com
대시보드: https://console.firebase.google.com/project/taskmaster-prod-a2b3c

다음 단계:
- 사용자 피드백 수집
- A/B 테스트 설정
- SEO 모니터링
```

### 프로젝트 결과

**개발 기간:** 5일

**팀 구성:** 1명 (+ 2 AI 도구)

**결과물:**
- ✅ 마케팅 랜딩 페이지
- ✅ 완전한 인증 시스템
- ✅ 사용자 대시보드
- ✅ Stripe 결제 연동
- ✅ 프로덕션 배포
- ✅ 테스트 커버리지 85%
- ✅ Lighthouse 95+ 점수

**비용:**
- Claude Code: $20/월
- Antigravity: $15/월
- Firebase: 무료 티어
- Stripe: 거래당 2.9% + $0.30
- 총: $35/월

**전통적 개발과 비교:**

| 항목 | 전통적 방식 | AI 협업 방식 |
|------|------------|-------------|
| 기간 | 6-8주 | 5일 |
| 팀 | 3명 (FE, BE, 디자이너) | 1명 + AI |
| 비용 | $30,000+ | $35/월 |
| 품질 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 고급 활용 패턴

### 패턴 1: 디자인 시스템 우선 접근

**개념:** 먼저 디자인 시스템을 정의하고 일관성 유지

**워크플로우:**

1. **Claude Code로 디자인 시스템 정의**

```bash
claude

You: 우리 회사의 디자인 시스템을 정의해줘.
     
     브랜드:
     - 이름: TechFlow
     - 산업: B2B SaaS
     - 분위기: 프로페셔널, 신뢰감
     
     다음을 포함해줘:
     - 색상 팔레트 (primary, secondary, neutral)
     - 타이포그래피 (폰트, 크기, 행간)
     - 간격 시스템
     - 그림자/Elevation
     - Border radius
     - 애니메이션 타이밍
```

**Claude Code 작업:**

```
디자인 시스템 생성 중...

✓ design-system.json 생성

{
  "colors": {
    "primary": {
      "50": "#eff6ff",
      "100": "#dbeafe",
      ...
      "900": "#1e3a8a"
    },
    "secondary": {...},
    "neutral": {...}
  },
  "typography": {
    "fontFamily": {
      "sans": "Inter, sans-serif",
      "mono": "Fira Code, monospace"
    },
    "fontSize": {
      "xs": "0.75rem",
      ...
    }
  },
  "spacing": {
    "0": "0",
    "1": "0.25rem",
    ...
  },
  ...
}

✓ Tailwind config 생성
✓ CSS 변수 생성
✓ Storybook 설정
✓ 컴포넌트 예제

? Figma 플러그인으로 내보낼까요?
```

2. **Stitch에서 디자인 시스템 활용**

```bash
antigravity

You: design-system.json을 참조해서
     대시보드를 디자인해줘.
     
     모든 색상, 폰트, 간격은
     디자인 시스템에서만 사용.
```

**결과:** 완벽하게 일관된 디자인

### 패턴 2: 컴포넌트 라이브러리 구축

**워크플로우:**

1. **Antigravity로 기본 컴포넌트 생성**

```bash
antigravity

You: 재사용 가능한 컴포넌트 라이브러리를 만들어줘.
     
     컴포넌트:
     - Button (variants: primary, secondary, outline)
     - Input (text, email, password)
     - Card
     - Modal
     - Dropdown
     - Table
     
     각 컴포넌트는 Stitch로 디자인하고
     React로 코드 생성.
```

2. **Claude Code로 품질 향상**

```bash
claude

You: 생성된 컴포넌트 라이브러리를 개선해줘.
     
     - TypeScript 변환
     - Storybook 스토리 작성
     - 테스트 작성
     - 접근성 개선
     - 문서화
```

3. **npm 패키지로 배포**

```bash
claude

You: 이 컴포넌트 라이브러리를
     npm 패키지로 배포 준비해줘.
     
     - package.json 설정
     - 빌드 설정 (Rollup)
     - README 작성
     - 배포 스크립트
```

### 패턴 3: 템플릿 기반 빠른 시작

**개념:** 자주 쓰는 패턴을 템플릿화

**템플릿 생성:**

```bash
claude

You: 다음 템플릿들을 만들어줘:
     
     1. SaaS 랜딩 페이지 템플릿
     2. 관리자 대시보드 템플릿
     3. 블로그 템플릿
     4. 이커머스 템플릿
     
     각 템플릿:
     - 폴더 구조
     - 기본 컴포넌트
     - 라우팅 설정
     - 설정 파일
```

**템플릿 사용:**

```bash
antigravity

You: SaaS 랜딩 페이지 템플릿을 사용해서
     우리 제품 페이지를 만들어줘.
     
     제품: DataViz Pro
     내용: [...]
```

**속도:** 템플릿 없이 2시간 → 템플릿으로 15분

---

## 문제 해결

### Stitch MCP 관련 문제

**문제 1: "Stitch 서버 연결 실패"**

증상:
```
✗ stitch (connection failed)
```

해결 단계:

1. MCP 서버 수동 실행:

```bash
npx -y davideast/stitch-mcp
```

에러 메시지 확인.

2. 자격증명 재설정:

```bash
rm ~/.config/stitch-mcp/credentials.json
npx -y davideast/stitch-mcp init
```

3. config.json 확인:

```bash
cat ~/.claude-code/config.json
```

JSON 문법 오류 체크.

4. Node.js 버전 확인:

```bash
node --version
```

18 이상이어야 함.

**문제 2: "디자인 생성 실패"**

증상:
```
✗ Design generation failed
```

해결:

1. 프롬프트 명확히:

나쁜 예:
```
대시보드 만들어
```

좋은 예:
```
관리자 대시보드를 만들어줘.
레이아웃: 왼쪽 사이드바 (240px), 상단 헤더 (64px)
메인: 4개의 통계 카드 (2x2 그리드)
색상: 파란색 (#3B82F6) 계열
```

2. 복잡도 낮추기:

한 번에 하나씩:
```
첫 번째: 사이드바만
두 번째: 헤더 추가
세 번째: 통계 카드
```

**문제 3: "할당량 초과"**

증상:
```
Quota exceeded
```

해결:

1. Google Cloud Console에서 할당량 확인
2. 다음 날까지 대기
3. 유료 플랜 고려

### Claude Code × Antigravity 협업 문제

**문제: 파일 충돌**

증상:
```
같은 파일을 두 도구가 동시에 수정
```

해결:

1. 작업 영역 분리:

```
프로젝트/
├── ui/          (Antigravity)
├── logic/       (Claude Code)
└── shared/      (협업)
```

2. Git 브랜치 사용:

```bash
Antigravity → feature/ui 브랜치
Claude Code → feature/logic 브랜치
나중에 머지
```

3. 순차 작업:

```
1. Antigravity 완료
2. 커밋
3. Claude Code 시작
```

**문제: 스타일 불일치**

증상:
```
각 도구가 다른 스타일 생성
```

해결:

1. 디자인 시스템 먼저:

```bash
claude

You: design-system.json 생성
```

2. 모든 요청에 참조:

```
design-system.json을 참고해서...
```

3. Linter 설정:

```bash
claude

You: ESLint + Prettier 설정해줘.
     일관된 코드 스타일 강제.
```

---

## 베스트 프랙티스

### 효과적인 프롬프트 작성

**1. SMART 원칙**

- **S**pecific (구체적): "버튼" → "파란색 primary 버튼, 16px 패딩"
- **M**easurable (측정 가능): "큰 텍스트" → "text-2xl (24px)"
- **A**chievable (달성 가능): 한 번에 너무 많이 요구 X
- **R**elevant (관련성): 현재 작업에 집중
- **T**ime-bound (시간 명시): "빠르게" → "5분 안에 프로토타입"

**2. 컨텍스트 제공**

나쁜 예:
```
웹사이트 만들어줘
```

좋은 예:
```
컨텍스트: B2B SaaS 스타트업
목적: 투자자 피칭용 프로토타입
기간: 오늘 안에 완성
타겟: 기업 의사결정자
톤: 프로페셔널, 신뢰감
레퍼런스: Slack, Notion 스타일

웹사이트 만들어줘.
```

**3. 점진적 상세화**

1차: 개요
```
SaaS 랜딩 페이지
```

2차: 구조
```
섹션: Hero, Features, Pricing, CTA
```

3차: 세부사항
```
Hero: 헤드라인, 서브헤드라인, 2개 CTA 버튼
Features: 3컬럼 그리드, 아이콘 포함
...
```

### 협업 워크플로우 최적화

**1. 역할 분담 명확히**

```yaml
Antigravity:
  - UI 디자인 (Stitch)
  - Firebase 설정
  - 초기 코드 생성
  - 배포

Claude Code:
  - 코드 품질
  - 복잡한 로직
  - 테스트
  - 문서화
```

**2. 동기화 포인트 설정**

```
Day 1 EOD: Antigravity 작업 완료 → 커밋
Day 2 Start: Claude Code 시작
Day 2 EOD: Claude Code 작업 완료 → 머지
Day 3: 통합 테스트
```

**3. 통신 프로토콜**

파일 헤더에 메타데이터:

```javascript
/**
 * 생성: Antigravity (2026-01-23)
 * 수정: Claude Code (2026-01-24)
 * 상태: Ready for review
 * 다음: 성능 테스트 필요
 */
```

### 품질 보증

**체크리스트:**

디자인:
- [ ] 디자인 시스템 준수
- [ ] 반응형 (모바일/태블릿/데스크톱)
- [ ] 접근성 (WCAG 2.1 AA)
- [ ] 브랜드 일관성

코드:
- [ ] TypeScript (타입 안전성)
- [ ] ESLint 통과
- [ ] Prettier 포맷팅
- [ ] 주석 및 문서화

성능:
- [ ] Lighthouse 90+
- [ ] 번들 크기 < 200KB
- [ ] Lazy loading
- [ ] 이미지 최적화

테스트:
- [ ] 유닛 테스트 80%+
- [ ] 통합 테스트
- [ ] E2E 주요 플로우
- [ ] 보안 테스트

배포:
- [ ] 환경 변수 분리
- [ ] 에러 트래킹
- [ ] Analytics
- [ ] 모니터링

---

## 결론

### 핵심 요약

**Stitch MCP의 가치:**
- UI 디자인 자동화
- 몇 초 만에 프로페셔널 결과
- 무한 반복 개선

**Claude Code의 강점:**
- 최고 품질 코드
- 복잡한 로직 구현
- 테스트와 문서화

**Antigravity의 강점:**
- Google 에코시스템
- 멀티모달 입력
- Firebase 자동화

**협업의 시너지:**
- 개발 속도 10배
- 품질 향상
- 비용 절감 95%

### 실천 가이드

**1주차: 기초 익히기**
- 각 도구 개별 사용
- 간단한 프로젝트 (랜딩 페이지)
- 프롬프트 작성 연습

**2주차: 협업 연습**
- 순차적 협업 워크플로우
- 소규모 프로젝트
- Git 워크플로우 확립

**3주차: 고급 활용**
- 병렬 협업
- 디자인 시스템 구축
- 템플릿 생성

**4주차: 프로덕션**
- 완전한 프로젝트
- 배포 자동화
- 모니터링 설정

### 다음 단계

**학습 리소스:**
- 공식 문서 정기적 확인
- 커뮤니티 참여
- 실전 프로젝트 진행

**실험:**
- 다양한 워크플로우 시도
- 자신만의 패턴 개발
- 팀과 공유

**기여:**
- 템플릿 공유
- 베스트 프랙티스 문서화
- 커뮤니티 피드백

이제 여러분은 AI 도구 협업의 달인입니다! 🚀

---

**작성일자:** 2026-01-23

**버전:** 1.0

**변경이력:**
- 2026-01-23: 초안 작성

**저자:** AI 개발 가이드 시리즈

**라이선스:** 자유롭게 공유하고 수정 가능 (출처 명시)

**기여자:**
- 피드백과 개선 제안을 환영합니다
- Issues: [GitHub 링크]
- Pull Requests: [GitHub 링크]

**관련 문서:**
- [Claude Code 완전 가이드](./claude-code-guide.md)
- [Antigravity 시작 가이드](./antigravity-guide.md)
- [Tailwind CSS 통합 가이드](./tailwind-guide.md)
