---
title: "Google Stitch MCP Server 워크플로우 가이드"
date: 2026-01-23 07:10:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Guide,  Google,  google-cloud,  Stitch,  MCP,  Antigravity,  claude-code,  Claude.write]
---


> 두 AI 도구의 장점을 결합하여 최고의 디자인-개발 워크플로우 구축하기

## 목차

1. [개요](#개요)
2. [환경 설정](#환경-설정)
3. [기본 사용법](#기본-사용법)
4. [협업 워크플로우](#협업-워크플로우)
5. [실전 프로젝트](#실전-프로젝트)
6. [고급 활용법](#고급-활용법)
7. [도구별 최적 활용](#도구별-최적-활용)
8. [문제 해결](#문제-해결)
9. [베스트 프랙티스](#베스트-프랙티스)
10. [리소스](#리소스)

---

## 개요

### Stitch MCP + Claude Code + Antigravity의 시너지

Google Stitch MCP Server를 Claude Code와 Antigravity 두 도구와 연동하면, 각 도구의 강점을 최대한 활용하여 디자인부터 배포까지 전체 워크플로우를 최적화할 수 있습니다.

#### 왜 두 도구를 함께 사용하는가?

**단일 도구의 한계:**

Claude Code만 사용:
- 코드 품질: 최상급
- 디자인 생성: 보통
- Firebase 통합: 약함
- Google 서비스: 제한적

Antigravity만 사용:
- Google 통합: 최상급
- 멀티모달: 최상급
- 코드 품질: 우수
- 복잡한 추론: 보통

**두 도구 조합의 장점:**

1. **최고의 코드 품질** (Claude Code)
2. **강력한 디자인 생성** (Stitch MCP)
3. **완벽한 Google 통합** (Antigravity)
4. **유연한 워크플로우** (도구 간 전환)

### 각 도구의 역할 정의

#### Claude Code가 더 나은 경우

**코드 품질이 중요할 때:**
- 복잡한 비즈니스 로직
- 보안이 중요한 부분
- 성능 최적화
- 테스트 작성

**추론이 필요할 때:**
- 아키텍처 설계
- 리팩토링
- 버그 원인 분석
- 코드 리뷰

**문서 작업:**
- Word 문서 (기술 문서)
- PowerPoint (프레젠테이션)
- Excel (데이터 분석)
- PDF (보고서)

예시:
```
사용자 인증 시스템의 보안을 강화해야 할 때
→ Claude Code 사용
→ 이유: 복잡한 보안 로직, 엣지 케이스 처리
```

#### Antigravity가 더 나은 경우

**Google 서비스 통합:**
- Firebase (Auth, Firestore, Functions)
- Google Cloud Platform
- Google Drive/Docs/Sheets
- Google Maps API

**멀티모달 입력:**
- 이미지에서 디자인 추출
- 스케치를 코드로 변환
- 스크린샷 기반 버그 수정

**디자인 작업:**
- Stitch MCP로 UI 생성
- 와이어프레임 → 코드
- 빠른 프로토타이핑

**큰 코드베이스:**
- 2M 토큰 컨텍스트
- 전체 모노레포 분석
- 대규모 리팩토링

예시:
```
Firebase 백엔드가 필요한 모바일 앱
→ Antigravity 사용
→ 이유: Firebase 통합, Stitch MCP 디자인
```

### 협업 시나리오

**시나리오 1: 디자인 우선**

```
1. Antigravity + Stitch로 디자인 생성
2. Claude Code로 비즈니스 로직 구현
3. Antigravity로 Firebase 배포
```

**시나리오 2: 코드 우선**

```
1. Claude Code로 백엔드 API 구현
2. Antigravity + Stitch로 프론트엔드 디자인
3. Claude Code로 통합 및 테스트
```

**시나리오 3: 동시 진행**

```
개발자 A: Claude Code로 백엔드 작업
개발자 B: Antigravity로 프론트엔드 작업
통합: 두 도구를 오가며 조율
```

---

## 환경 설정

### 필수 도구 설치

#### 1. Claude Code 설치

**macOS:**
```bash
brew install --cask claude-code
claude --version
```

**Linux:**
```bash
curl -fsSL https://code.claude.com/linux/ubuntu/install.sh | sudo bash
claude --version
```

**Windows:**
```powershell
winget install Anthropic.ClaudeCode
claude --version
```

확인:
```bash
claude auth status
```

#### 2. Antigravity 설치

**macOS:**
```bash
brew install google-antigravity
antigravity --version
```

**Linux:**
```bash
curl -fsSL https://antigravity.dev/install.sh | bash
antigravity --version
```

**Windows:**
```powershell
winget install Google.Antigravity
antigravity --version
```

확인:
```bash
antigravity login
```

#### 3. Node.js 및 gcloud

**Node.js 18+:**
```bash
node --version
npm --version
```

필요 시 설치: [nodejs.org](https://nodejs.org)

**gcloud CLI:**

Stitch MCP가 자동 설치하지만, 수동 설치:
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud --version
```

### Stitch MCP 설정

#### 초기화

```bash
npx -y davideast/stitch-mcp init
```

프롬프트 응답:
```
? Create new Google Cloud project? Y
? Project ID: your-stitch-project-2026
? Authenticate now? Y
```

브라우저에서 Google 계정 인증:
1. 로그인
2. 권한 승인
3. 완료 확인

#### Claude Code 연동

**설정 파일 생성:**

macOS/Linux:
```bash
mkdir -p ~/.claude-code
nano ~/.claude-code/config.json
```

Windows:
```powershell
mkdir $env:USERPROFILE\.claude-code
notepad $env:USERPROFILE\.claude-code\config.json
```

**설정 내용:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "davideast/stitch-mcp"]
    }
  }
}
```

**확인:**

```bash
claude

Claude가 시작되면:
✓ MCP Servers:
  - stitch (davideast/stitch-mcp)
```

#### Antigravity 연동

**설정 파일:**

macOS/Linux:
```bash
nano ~/.antigravity/config.json
```

Windows:
```powershell
notepad $env:USERPROFILE\.antigravity\config.json
```

**설정 내용:**

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "davideast/stitch-mcp"]
    }
  }
}
```

**확인:**

```bash
antigravity

Antigravity가 시작되면:
🔌 MCP Servers:
  ✓ stitch
```

### 통합 테스트

두 도구가 모두 Stitch에 접근할 수 있는지 확인합니다.

**Claude Code 테스트:**

```bash
claude

You: Stitch MCP가 연결되어 있는지 확인해줘

Claude: 
✓ Stitch MCP Server 연결됨
✓ Google Cloud Project: your-stitch-project-2026
✓ 사용 가능한 명령:
  - generate_design
  - export_html
  - list_designs

준비 완료!
```

**Antigravity 테스트:**

```bash
antigravity

You: Stitch 상태 확인

Antigravity:
✓ Stitch MCP: Connected
✓ Project: your-stitch-project-2026
✓ Quota: 95/100 requests today

All systems operational!
```

---

## 기본 사용법

### Claude Code에서 Stitch 사용

#### 디자인 생성

```bash
claude

You: Stitch로 로그인 페이지 디자인해줘.
     
     요구사항:
     - 이메일/비밀번호 입력
     - "로그인" 버튼
     - "비밀번호 찾기" 링크
     - 소셜 로그인 (Google, GitHub)
     
     미니멀하고 현대적인 스타일

Claude: Stitch MCP로 디자인을 생성하겠습니다.

[MCP: stitch.generate_design]

✓ 디자인 생성 완료

미리보기: https://stitch.withgoogle.com/preview/abc123

컴포넌트:
- 로그인 폼 (중앙 정렬)
- 이메일 input (아이콘 포함)
- 비밀번호 input (표시/숨김 토글)
- 메인 버튼 (그라데이션)
- 소셜 로그인 버튼 (2개)
- 링크 (하단)

? HTML/CSS로 변환할까요? (y/n)
```

Y 선택:

```
[MCP: stitch.export_html]

✓ HTML 생성 완료
✓ CSS 생성 완료

파일:
- src/pages/login.html
- src/styles/login.css

로컬 서버를 시작하시겠습니까?
```

#### 기존 디자인 수정

```
You: 방금 만든 로그인 페이지에 "회원가입" 링크를 추가하고
     버튼 색상을 파란색에서 보라색으로 변경해줘

Claude: 디자인을 업데이트하겠습니다.

[MCP: stitch.update_design]

✓ 업데이트 완료

변경사항:
- "회원가입" 링크 추가 (하단)
- 버튼 색상: #3b82f6 → #8b5cf6

파일이 자동으로 업데이트되었습니다.
```

### Antigravity에서 Stitch 사용

#### 이미지 기반 디자인

```bash
antigravity

You: 첨부한 와이어프레임을 보고 Stitch로 디자인을 만들어줘
     [wireframe.png 업로드]

Antigravity: 와이어프레임을 분석하고 Stitch로 디자인을 생성합니다.

[이미지 분석 중...]

인식된 요소:
- 헤더 (로고 + 네비게이션)
- Hero 섹션 (큰 제목 + 버튼)
- Feature 카드 (3개, 그리드)
- Footer

[MCP: stitch.generate_design]

✓ 디자인 생성

미리보기: https://stitch.withgoogle.com/preview/def456

? 코드로 변환? (y/n)
```

#### Firebase 통합

```
You: 이 로그인 페이지를 Firebase Authentication과 연동해줘

Antigravity: Firebase Auth를 설정하고 로그인 페이지와 통합하겠습니다.

1. Firebase 프로젝트 생성
✓ Project: login-app-2026

2. Authentication 활성화
✓ Email/Password
✓ Google Sign-In
✓ GitHub Sign-In

3. 코드 통합
✓ Firebase SDK 추가
✓ 로그인 함수 구현
✓ 에러 처리

4. 테스트
✓ 로컬 테스트 성공

배포하시겠습니까?
```

---

## 협업 워크플로우

이 섹션이 이 가이드의 핵심입니다. 두 도구를 효과적으로 조합하는 다양한 패턴을 소개합니다.

### 워크플로우 1: 디자인 → 개발 → 배포

가장 일반적인 패턴입니다.

#### 단계 1: Antigravity로 디자인 생성

**목표:** Stitch로 빠르게 UI 디자인

```bash
antigravity

You: SaaS 대시보드를 디자인해줘.
     
     레이아웃:
     - 왼쪽 사이드바 (네비게이션)
     - 상단 헤더 (검색, 알림, 프로필)
     - 메인 영역 (통계 카드 + 차트)
     
     스타일: 현대적, 미니멀, 파란색 계열

Antigravity: 
[Stitch로 디자인 생성]

✓ 완료

파일:
- dashboard.html
- dashboard.css
- assets/

다음 단계는?
```

**시간:** 2-3분

#### 단계 2: Claude Code로 전환 (비즈니스 로직)

**목표:** 고품질 백엔드 코드 작성

```bash
claude

You: Antigravity가 만든 대시보드에 연결할 백엔드 API를 구현해줘.
     
     필요한 엔드포인트:
     - GET /api/stats (통계 데이터)
     - GET /api/charts/revenue (매출 차트)
     - GET /api/notifications (알림 목록)
     - POST /api/notifications/:id/read (알림 읽음 처리)
     
     기술: Express.js + PostgreSQL
     인증: JWT

Claude: 백엔드 API를 구현하겠습니다.

프로젝트 구조:
server/
├── src/
│   ├── routes/
│   │   ├── stats.js
│   │   ├── charts.js
│   │   └── notifications.js
│   ├── controllers/
│   ├── models/
│   ├── middleware/
│   │   └── auth.js
│   └── db/
│       └── connection.js
├── tests/
└── package.json

? 시작할까요? (y/n)
```

**진행:**

```
✓ Express 서버 설정
✓ PostgreSQL 연결
✓ JWT 인증 미들웨어
✓ Stats API 구현
✓ Charts API 구현
✓ Notifications API 구현
✓ 테스트 작성 (Jest)

모든 테스트 통과!

다음: 프론트엔드와 통합
```

**시간:** 30-45분

#### 단계 3: Antigravity로 통합 (Firebase/GCP 배포)

**목표:** Firebase와 통합 및 배포

```bash
antigravity

You: Claude Code가 만든 백엔드를 Cloud Functions로 배포하고
     프론트엔드는 Firebase Hosting에 올려줘

Antigravity:
1. Cloud Functions 설정
✓ functions/ 디렉토리 생성
✓ Express 앱을 Cloud Function으로 래핑
✓ 환경 변수 설정

2. Firebase Hosting 설정
✓ 정적 파일 준비
✓ firebase.json 설정
✓ API 프록시 설정

3. 배포
✓ firebase deploy --only functions
✓ firebase deploy --only hosting

배포 완료!
- Functions: https://us-central1-project.cloudfunctions.net/api
- Hosting: https://project.web.app

? 커스텀 도메인 설정? (y/n)
```

**시간:** 10-15분

#### 전체 워크플로우 정리

```
총 소요 시간: 약 1시간

Antigravity (디자인): 3분
↓
Claude Code (백엔드): 45분
↓
Antigravity (배포): 15분
↓
완성! 🎉
```

### 워크플로우 2: 반복적 개선

프로젝트를 점진적으로 발전시키는 패턴입니다.

#### 사이클 1: MVP (Antigravity)

**목표:** 빠른 프로토타입

```
antigravity

You: 간단한 할일 관리 앱을 만들어줘.
     Firebase로 데이터 저장하고
     Stitch로 UI 디자인

Antigravity:
[빠른 구현]

✓ Firestore 스키마 설계
✓ Stitch로 UI 생성
✓ CRUD 함수 구현
✓ Firebase 배포

30분 만에 완성!
```

#### 사이클 2: 품질 개선 (Claude Code)

**목표:** 코드 품질 향상

```
claude

You: Antigravity가 만든 할일 앱 코드를 리뷰하고 개선해줘.
     
     중점 사항:
     - 에러 처리
     - 성능 최적화
     - 테스트 추가
     - 타입 안전성 (TypeScript)

Claude: 코드를 분석하고 개선하겠습니다.

발견된 개선 기회:

1. 에러 처리 부족
   - try-catch 추가
   - 사용자 친화적 에러 메시지

2. 성능 이슈
   - 불필요한 리렌더링
   - Firestore 쿼리 최적화

3. 테스트 없음
   - Unit tests 추가
   - Integration tests 추가

4. JavaScript → TypeScript 마이그레이션

? 모든 개선사항을 적용할까요? (y/n)
```

**시간:** 1-2시간

#### 사이클 3: 고급 기능 (도구 번갈아 사용)

**Antigravity: 실시간 협업**

```
You: 실시간 협업 기능을 추가해줘.
     여러 사용자가 동시에 할일 목록을 편집할 수 있어야 해.

Antigravity:
[Firestore 실시간 리스너 구현]

✓ 실시간 동기화
✓ 충돌 해결
✓ 사용자 상태 표시
```

**Claude Code: 알림 시스템**

```
You: 할일 마감 1시간 전에 알림을 보내는 시스템을 구현해줘.
     
     요구사항:
     - Cloud Scheduler로 정기 체크
     - 이메일 알림 (SendGrid)
     - 푸시 알림 (FCM)

Claude:
[복잡한 백엔드 로직 구현]

✓ Scheduler 설정
✓ 알림 큐 시스템
✓ 이메일 템플릿
✓ 푸시 알림
✓ 에러 복구
```

### 워크플로우 3: 역할 분담 협업

팀에서 두 도구를 분담하여 사용하는 패턴입니다.

#### 시나리오: 전자상거래 사이트

**개발자 A (Antigravity 담당): 프론트엔드**

```
Day 1: Stitch로 모든 페이지 디자인
- 홈페이지
- 상품 목록
- 상품 상세
- 장바구니
- 결제
- 마이페이지

Day 2: Firebase 설정
- Authentication
- Firestore 스키마
- Storage (상품 이미지)
- Hosting
```

**개발자 B (Claude Code 담당): 백엔드**

```
Day 1: API 구현
- 상품 관리 API
- 주문 처리 API
- 결제 통합 (Stripe)
- 재고 관리

Day 2: 테스트 및 최적화
- 단위 테스트
- 통합 테스트
- 성능 최적화
- 보안 강화
```

**Day 3: 통합 (함께)**

```
morning (Antigravity):
- 프론트엔드 → 백엔드 API 연결
- Firebase Functions 배포

afternoon (Claude Code):
- API 응답 검증
- 에러 처리 개선
- 통합 테스트

evening (Antigravity):
- 최종 배포
- 모니터링 설정
```

### 워크플로우 4: 문제 해결 협업

버그나 성능 문제를 해결할 때 두 도구를 조합합니다.

#### 상황: 페이지 로딩 느림

**단계 1: Antigravity로 분석**

```
antigravity

You: 대시보드 페이지 로딩이 너무 느려.
     Chrome DevTools 스크린샷 첨부했어.
     [performance-screenshot.png]

Antigravity: 
[이미지 분석]

발견된 문제:
1. 너무 많은 Firestore 읽기 (47 reads)
2. 큰 이미지 (3MB)
3. 번들 크기 (1.2MB)

즉시 해결 가능:
- 이미지 최적화 (WebP 변환)
- 코드 스플리팅

복잡한 문제 (Claude Code 권장):
- Firestore 쿼리 최적화
- 캐싱 전략
```

**단계 2: Claude Code로 최적화**

```
claude

You: Firestore 쿼리를 최적화하고 캐싱 전략을 구현해줘.

Claude: 
성능 최적화를 진행하겠습니다.

1. Firestore 쿼리 분석
   - 47 reads → 8 reads로 감소
   - 인덱스 추가
   - 복합 쿼리 사용

2. 캐싱 전략
   - React Query 도입
   - stale-while-revalidate
   - 로컬 스토리지 백업

3. 코드 리팩토링
   - 불필요한 리렌더링 제거
   - useMemo, useCallback 적용

예상 개선:
- 로딩 시간: 3.2s → 0.8s
- Firestore 비용: 85% 절감
```

**단계 3: Antigravity로 검증**

```
antigravity

You: 최적화 후 성능 테스트해줘

Antigravity:
[Firebase Performance Monitoring 확인]

결과:
✓ 로딩 시간: 0.7s (78% 개선)
✓ Firestore reads: 8 (83% 감소)
✓ Bundle size: 340KB (72% 감소)

성능 목표 달성! 🎉
```

---

## 실전 프로젝트

실제로 두 도구를 조합하여 완성된 프로젝트를 만들어봅시다.

### 프로젝트: 실시간 협업 화이트보드

**목표:** 
- 여러 사용자가 동시에 그림을 그릴 수 있는 화이트보드
- 실시간 동기화
- Firebase 백엔드
- 프로덕션 배포

**기술 스택:**
- Frontend: React + Tailwind CSS
- Backend: Firebase (Firestore, Functions, Hosting)
- 실시간: Firestore real-time listeners

**소요 시간:** 약 3시간

### 단계 1: 디자인 (Antigravity + Stitch)

**시간:** 15분

```bash
antigravity

You: 화이트보드 앱을 디자인해줘.

레이아웃:
- 상단 툴바 (그리기 도구, 색상, 굵기)
- 왼쪽 사이드바 (레이어 목록, 사용자 목록)
- 메인 캔버스 (그리기 영역)
- 하단 상태바 (줌, 좌표)

툴바 아이콘:
- 펜, 지우개, 선, 원, 사각형
- 색상 팔레트
- 굵기 슬라이더
- 실행 취소/다시 실행

현대적이고 깔끔한 디자인

Antigravity:
[Stitch로 디자인 생성]

✓ 디자인 완료

미리보기: https://stitch.withgoogle.com/preview/whiteboard-xyz

컴포넌트:
- Toolbar (top, 56px height)
- Sidebar (left, 240px width, collapsible)
- Canvas (main area, full screen)
- StatusBar (bottom, 32px height)
- ColorPicker (modal)
- UserList (live avatars)

? HTML/CSS로 변환? (y/n)
```

Y 선택:

```
✓ whiteboard/
  ├── index.html
  ├── styles.css
  └── components/
      ├── toolbar.html
      ├── sidebar.html
      └── canvas.html

파일 생성 완료!
```

**생성된 HTML 구조:**

index.html의 핵심 구조:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>실시간 화이트보드</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    상단 툴바:
    <div class="toolbar">
        <div class="tool-group">
            <button class="tool-btn active" data-tool="pen">
                펜 아이콘
            </button>
            <button class="tool-btn" data-tool="eraser">
                지우개 아이콘
            </button>
            <button class="tool-btn" data-tool="line">
                선 아이콘
            </button>
        </div>
        
        <div class="color-picker">
            <div class="current-color" style="background: #000"></div>
            색상 선택 버튼들
        </div>
        
        <div class="size-control">
            <input type="range" min="1" max="50" value="3">
            <span class="size-value">3px</span>
        </div>
    </div>
    
    왼쪽 사이드바:
    <div class="sidebar">
        <div class="users-section">
            <h3>참여자</h3>
            <div class="user-list">
                사용자 아바타들
            </div>
        </div>
        
        <div class="layers-section">
            <h3>레이어</h3>
            <div class="layer-list">
                레이어 목록
            </div>
        </div>
    </div>
    
    메인 캔버스:
    <main class="canvas-container">
        <canvas id="whiteboard"></canvas>
    </main>
    
    하단 상태바:
    <div class="statusbar">
        <span class="zoom-level">100%</span>
        <span class="coordinates">X: 0, Y: 0</span>
        <span class="connection-status">연결됨</span>
    </div>
</body>
</html>
```

**styles.css 핵심 부분:**

```css
body {
    margin: 0;
    font-family: -apple-system, sans-serif;
    display: flex;
    flex-direction: column;
    height: 100vh;
    overflow: hidden;
}

.toolbar {
    height: 56px;
    background: white;
    border-bottom: 1px solid #e5e7eb;
    display: flex;
    align-items: center;
    padding: 0 16px;
    gap: 24px;
}

.tool-btn {
    width: 40px;
    height: 40px;
    border: none;
    background: transparent;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.2s;
}

.tool-btn:hover {
    background: #f3f4f6;
}

.tool-btn.active {
    background: #3b82f6;
    color: white;
}

.sidebar {
    width: 240px;
    background: #f9fafb;
    border-right: 1px solid #e5e7eb;
    padding: 16px;
    overflow-y: auto;
}

.canvas-container {
    flex: 1;
    position: relative;
    background: #ffffff;
}

#whiteboard {
    width: 100%;
    height: 100%;
    cursor: crosshair;
}

.statusbar {
    height: 32px;
    background: #f9fafb;
    border-top: 1px solid #e5e7eb;
    display: flex;
    align-items: center;
    padding: 0 16px;
    gap: 24px;
    font-size: 12px;
    color: #6b7280;
}
```

### 단계 2: React 변환 및 Firebase 설정 (Antigravity)

**시간:** 20분

```
You: 이 디자인을 React 앱으로 변환하고 Firebase 프로젝트를 설정해줘

Antigravity:
1. React 프로젝트 생성
✓ create-react-app whiteboard-app
✓ Tailwind CSS 설정

2. 컴포넌트 변환
✓ Toolbar.jsx
✓ Sidebar.jsx
✓ Canvas.jsx
✓ StatusBar.jsx
✓ ColorPicker.jsx

3. Firebase 프로젝트 생성
✓ Project: whiteboard-collab-2026
✓ Firestore 활성화
✓ Authentication 활성화 (Google)
✓ Hosting 설정

4. Firestore 스키마 설계
collections/
├── rooms/
│   └── {roomId}/
│       ├── strokes/ (그리기 데이터)
│       └── users/ (접속 사용자)

완료! 로컬 개발 서버 시작하시겠습니까?
```

**생성된 React 구조:**

```
whiteboard-app/
├── src/
│   ├── components/
│   │   ├── Toolbar.jsx
│   │   ├── Sidebar.jsx
│   │   ├── Canvas.jsx
│   │   ├── StatusBar.jsx
│   │   └── ColorPicker.jsx
│   ├── hooks/
│   │   ├── useFirestore.js
│   │   └── useCanvas.js
│   ├── lib/
│   │   └── firebase.js
│   ├── App.jsx
│   └── index.js
├── firebase.json
└── package.json
```

### 단계 3: 캔버스 로직 (Claude Code)

**시간:** 1시간

이제 복잡한 캔버스 그리기 로직을 Claude Code로 구현합니다.

```bash
claude

You: 화이트보드 캔버스의 그리기 로직을 구현해줘.

요구사항:
- 펜 도구 (자유 그리기)
- 도형 도구 (선, 원, 사각형)
- 지우개
- 실행 취소/다시 실행
- 부드러운 선 (anti-aliasing)
- 성능 최적화 (많은 선을 그려도 버벅이지 않게)

Claude: 캔버스 그리기 시스템을 구현하겠습니다.

구조:
1. Drawing Engine
   - Canvas 렌더링
   - 이벤트 처리
   - 변환 (줌, 팬)

2. Tool System
   - 각 도구별 클래스
   - 통합 인터페이스
   - 상태 관리

3. History System
   - Command 패턴
   - Undo/Redo 스택

4. Performance
   - Off-screen canvas
   - Dirty rect 최적화
   - RequestAnimationFrame

? 시작할까요? (y/n)
```

Y 선택:

```
구현 중...

✓ src/engine/DrawingEngine.js
✓ src/tools/PenTool.js
✓ src/tools/LineTool.js
✓ src/tools/CircleTool.js
✓ src/tools/RectangleTool.js
✓ src/tools/EraserTool.js
✓ src/history/CommandHistory.js
✓ src/utils/optimize.js

테스트 작성 중...
✓ 27 tests passing

성능 테스트:
- 1000 strokes: 60 FPS
- 10000 strokes: 45 FPS (허용)

완료!
```

**핵심 코드 예시:**

DrawingEngine.js:

```javascript
class DrawingEngine {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.offscreenCanvas = document.createElement('canvas');
    this.offscreenCtx = this.offscreenCanvas.getContext('2d');
    
    this.strokes = [];
    this.currentTool = null;
    this.isDrawing = false;
    
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    this.canvas.addEventListener('mousedown', this.handleMouseDown.bind(this));
    this.canvas.addEventListener('mousemove', this.handleMouseMove.bind(this));
    this.canvas.addEventListener('mouseup', this.handleMouseUp.bind(this));
  }
  
  handleMouseDown(e) {
    if (!this.currentTool) return;
    
    this.isDrawing = true;
    const point = this.getMousePos(e);
    this.currentTool.onStart(point);
  }
  
  handleMouseMove(e) {
    if (!this.isDrawing || !this.currentTool) return;
    
    const point = this.getMousePos(e);
    this.currentTool.onMove(point);
    
    requestAnimationFrame를 사용하여 부드러운 렌더링:
    requestAnimationFrame(() => this.render());
  }
  
  handleMouseUp(e) {
    if (!this.isDrawing) return;
    
    this.isDrawing = false;
    const point = this.getMousePos(e);
    const stroke = this.currentTool.onEnd(point);
    
    if (stroke) {
      this.strokes.push(stroke);
      this.onStrokeComplete(stroke);
    }
  }
  
  render() {
    성능 최적화: 변경된 영역만 다시 그리기:
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    
    모든 stroke 렌더링:
    this.strokes.forEach(stroke => {
      stroke.render(this.ctx);
    });
    
    현재 그리는 중인 stroke:
    if (this.currentTool && this.isDrawing) {
      this.currentTool.renderPreview(this.ctx);
    }
  }
  
  getMousePos(e) {
    const rect = this.canvas.getBoundingClientRect();
    return {
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    };
  }
  
  setTool(tool) {
    this.currentTool = tool;
  }
}

export default DrawingEngine;
```

PenTool.js:

```javascript
class PenTool {
  constructor(color, size) {
    this.color = color;
    this.size = size;
    this.points = [];
  }
  
  onStart(point) {
    this.points = [point];
  }
  
  onMove(point) {
    this.points.push(point);
  }
  
  onEnd(point) {
    this.points.push(point);
    
    return new PenStroke(this.points, this.color, this.size);
  }
  
  renderPreview(ctx) {
    if (this.points.length < 2) return;
    
    ctx.strokeStyle = this.color;
    ctx.lineWidth = this.size;
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    
    ctx.beginPath();
    ctx.moveTo(this.points[0].x, this.points[0].y);
    
    Catmull-Rom 스플라인으로 부드러운 곡선:
    for (let i = 1; i < this.points.length; i++) {
      const xc = (this.points[i].x + this.points[i - 1].x) / 2;
      const yc = (this.points[i].y + this.points[i - 1].y) / 2;
      ctx.quadraticCurveTo(
        this.points[i - 1].x,
        this.points[i - 1].y,
        xc,
        yc
      );
    }
    
    ctx.stroke();
  }
}

class PenStroke {
  constructor(points, color, size) {
    this.points = points;
    this.color = color;
    this.size = size;
  }
  
  render(ctx) {
    동일한 렌더링 로직
  }
  
  toJSON() {
    Firebase 저장을 위한 직렬화:
    return {
      type: 'pen',
      points: this.points,
      color: this.color,
      size: this.size
    };
  }
  
  static fromJSON(data) {
    return new PenStroke(data.points, data.color, data.size);
  }
}

export default PenTool;
```

### 단계 4: 실시간 동기화 (Antigravity)

**시간:** 30분

```bash
antigravity

You: Firestore를 사용해서 실시간 동기화를 구현해줘.
     
     요구사항:
     - 다른 사용자의 그림이 실시간으로 보여야 함
     - 새 사용자가 접속하면 기존 그림 로드
     - 사용자 커서 위치 공유
     - 충돌 없이 동시 편집

Antigravity: 실시간 동기화를 구현하겠습니다.

1. Firestore 리스너 설정
✓ onSnapshot으로 실시간 감지

2. Stroke 동기화
✓ 새 stroke → Firestore 추가
✓ Firestore 변경 → 캔버스 업데이트

3. 사용자 커서
✓ Presence 시스템
✓ 커서 위치 실시간 공유

4. 최적화
✓ Batch writes
✓ 로컬 우선 (optimistic updates)

파일:
✓ src/hooks/useRealtimeSync.js
✓ src/hooks/usePresence.js
✓ src/utils/firestore.js

테스트:
✓ 2명 동시 그리기 성공
✓ 10명 동시 접속 성공
✓ 지연시간: < 100ms

완료!
```

**핵심 코드:**

useRealtimeSync.js:

```javascript
import { useEffect, useState } from 'react';
import { 
  collection, 
  onSnapshot, 
  addDoc, 
  query, 
  orderBy 
} from 'firebase/firestore';
import { db } from '../lib/firebase';

function useRealtimeSync(roomId, onStrokeAdded) {
  const [strokes, setStrokes] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    if (!roomId) return;
    
    const strokesRef = collection(db, `rooms/${roomId}/strokes`);
    const q = query(strokesRef, orderBy('createdAt', 'asc'));
    
    Firestore 실시간 리스너:
    const unsubscribe = onSnapshot(q, (snapshot) => {
      snapshot.docChanges().forEach((change) => {
        if (change.type === 'added') {
          const stroke = {
            id: change.doc.id,
            ...change.doc.data()
          };
          
          로컬에서 그린 것이 아니면 캔버스에 추가:
          if (!stroke.isLocal) {
            onStrokeAdded(stroke);
          }
        }
      });
      
      setStrokes(snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      })));
      setLoading(false);
    });
    
    return () => unsubscribe();
  }, [roomId, onStrokeAdded]);
  
  const addStroke = async (strokeData) => {
    try {
      const strokesRef = collection(db, `rooms/${roomId}/strokes`);
      
      Optimistic update: 로컬에 먼저 적용:
      strokeData.isLocal = true;
      
      await addDoc(strokesRef, {
        ...strokeData,
        createdAt: new Date(),
        userId: getCurrentUserId()
      });
    } catch (error) {
      console.error('Failed to add stroke:', error);
    }
  };
  
  return { strokes, loading, addStroke };
}

export default useRealtimeSync;
```

usePresence.js:

```javascript
import { useEffect, useState } from 'react';
import { doc, setDoc, onSnapshot, deleteDoc } from 'firebase/firestore';
import { db } from '../lib/firebase';

function usePresence(roomId) {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    if (!roomId) return;
    
    const userId = getCurrentUserId();
    const userRef = doc(db, `rooms/${roomId}/users/${userId}`);
    
    자신의 presence 등록:
    setDoc(userRef, {
      id: userId,
      name: getCurrentUserName(),
      color: getRandomColor(),
      cursor: { x: 0, y: 0 },
      lastSeen: new Date()
    });
    
    다른 사용자들 감지:
    const usersRef = collection(db, `rooms/${roomId}/users`);
    const unsubscribe = onSnapshot(usersRef, (snapshot) => {
      const activeUsers = snapshot.docs
        .map(doc => doc.data())
        .filter(user => isActive(user.lastSeen));
      
      setUsers(activeUsers);
    });
    
    페이지 닫을 때 presence 제거:
    const cleanup = () => {
      deleteDoc(userRef);
    };
    window.addEventListener('beforeunload', cleanup);
    
    return () => {
      cleanup();
      unsubscribe();
      window.removeEventListener('beforeunload', cleanup);
    };
  }, [roomId]);
  
  const updateCursor = (x, y) => {
    if (!roomId) return;
    
    const userId = getCurrentUserId();
    const userRef = doc(db, `rooms/${roomId}/users/${userId}`);
    
    Throttle을 사용하여 과도한 업데이트 방지:
    throttledUpdate(userRef, {
      cursor: { x, y },
      lastSeen: new Date()
    });
  };
  
  return { users, updateCursor };
}

export default usePresence;
```

### 단계 5: 테스트 및 최적화 (Claude Code)

**시간:** 30분

```bash
claude

You: 화이트보드 앱의 성능을 테스트하고 최적화해줘.
     
     테스트 시나리오:
     - 10명이 동시에 그리기
     - 1000개 이상의 stroke
     - 느린 네트워크 환경
     
     목표 성능:
     - 60 FPS 유지
     - 네트워크 지연 < 200ms
     - 메모리 사용량 < 100MB

Claude: 성능 테스트 및 최적화를 진행하겠습니다.

1. 현재 성능 측정
✓ Lighthouse 점수: 68/100
✓ FPS: 35-40 (목표 미달)
✓ 메모리: 145MB (목표 초과)

2. 병목 지점 분석
- Canvas 리렌더링 과다
- Firestore 읽기 과다
- 메모리 누수 (old strokes)

3. 최적화 적용

A. Canvas 최적화
- Dirty rectangle tracking
- Off-screen canvas 활용
- Layer 시스템

B. Firestore 최적화
- 페이지네이션 (최근 100개만)
- 로컬 캐싱
- Batch operations

C. 메모리 최적화
- Stroke pooling
- Weak references
- 주기적 cleanup

4. 최적화 후 성능
✓ Lighthouse: 92/100
✓ FPS: 58-60
✓ 메모리: 75MB

목표 달성! 🎉
```

**최적화 코드 예시:**

optimize.js:

```javascript
성능 최적화 유틸리티

class StrokePool {
  constructor() {
    this.pool = [];
    this.maxSize = 1000;
  }
  
  acquire(points, color, size) {
    if (this.pool.length > 0) {
      const stroke = this.pool.pop();
      stroke.reset(points, color, size);
      return stroke;
    }
    return new Stroke(points, color, size);
  }
  
  release(stroke) {
    if (this.pool.length < this.maxSize) {
      this.pool.push(stroke);
    }
  }
}

Dirty rectangle tracking으로 변경된 영역만 다시 그리기:
class DirtyRectTracker {
  constructor() {
    this.rects = [];
  }
  
  add(x, y, width, height) {
    this.rects.push({ x, y, width, height });
  }
  
  getBounds() {
    if (this.rects.length === 0) return null;
    
    let minX = Infinity, minY = Infinity;
    let maxX = -Infinity, maxY = -Infinity;
    
    this.rects.forEach(rect => {
      minX = Math.min(minX, rect.x);
      minY = Math.min(minY, rect.y);
      maxX = Math.max(maxX, rect.x + rect.width);
      maxY = Math.max(maxY, rect.y + rect.height);
    });
    
    return { 
      x: minX, 
      y: minY, 
      width: maxX - minX, 
      height: maxY - minY 
    };
  }
  
  clear() {
    this.rects = [];
  }
}

Firestore 페이지네이션:
async function loadStrokes(roomId, limit = 100) {
  const strokesRef = collection(db, `rooms/${roomId}/strokes`);
  const q = query(
    strokesRef, 
    orderBy('createdAt', 'desc'),
    limit(limit)
  );
  
  const snapshot = await getDocs(q);
  return snapshot.docs.map(doc => ({
    id: doc.id,
    ...doc.data()
  })).reverse();
}

export { StrokePool, DirtyRectTracker, loadStrokes };
```

### 단계 6: 배포 (Antigravity)

**시간:** 15분

```bash
antigravity

You: 화이트보드 앱을 Firebase에 배포해줘.
     프로덕션 설정도 함께.

Antigravity:
1. 프로덕션 빌드
✓ npm run build
✓ 최적화 완료 (gzip: 45KB)

2. Firebase 설정
✓ Hosting 설정
✓ Firestore 보안 규칙
✓ 환경 변수

3. 배포
✓ firebase deploy

배포 완료!

URL: https://whiteboard-collab-2026.web.app

성능:
- First Contentful Paint: 0.8s
- Time to Interactive: 1.2s
- Lighthouse: 92/100

보안:
✓ HTTPS
✓ Firestore rules 적용
✓ Rate limiting

모니터링:
✓ Firebase Analytics 설정
✓ Performance Monitoring 활성화
✓ Crashlytics 설정

완료! 🚀
```

### 프로젝트 완성!

**총 소요 시간:** 약 3시간

**결과:**
- ✅ 프로페셔널 디자인
- ✅ 실시간 협업 기능
- ✅ 60 FPS 성능
- ✅ 프로덕션 배포
- ✅ 모니터링 설정

**도구 사용 비중:**
- Antigravity: 45% (디자인, Firebase, 배포)
- Claude Code: 45% (복잡한 로직, 최적화)
- 수동 작업: 10% (검토, 테스트)

**배운 점:**
1. Antigravity는 디자인과 Firebase 통합에 탁월
2. Claude Code는 복잡한 로직과 최적화에 최적
3. 두 도구를 적절히 조합하면 최상의 결과
4. 각 도구의 강점을 살려 역할 분담이 중요

---

## 고급 활용법

### 커스텀 디자인 시스템

두 도구에서 일관되게 사용할 수 있는 디자인 시스템을 구축합니다.

#### 설정 파일 생성

`design-system.json`:

```json
{
  "name": "MyApp Design System",
  "version": "1.0.0",
  "colors": {
    "primary": {
      "50": "#eff6ff",
      "500": "#3b82f6",
      "900": "#1e3a8a"
    },
    "secondary": {
      "50": "#f5f3ff",
      "500": "#8b5cf6",
      "900": "#4c1d95"
    },
    "neutral": {
      "50": "#fafafa",
      "500": "#737373",
      "900": "#171717"
    }
  },
  "typography": {
    "fontFamily": {
      "sans": "Inter, system-ui, sans-serif",
      "mono": "Fira Code, monospace"
    },
    "fontSize": {
      "xs": "0.75rem",
      "sm": "0.875rem",
      "base": "1rem",
      "lg": "1.125rem",
      "xl": "1.25rem",
      "2xl": "1.5rem"
    }
  },
  "spacing": {
    "xs": "0.25rem",
    "sm": "0.5rem",
    "md": "1rem",
    "lg": "1.5rem",
    "xl": "2rem",
    "2xl": "3rem"
  },
  "borderRadius": {
    "sm": "0.25rem",
    "md": "0.5rem",
    "lg": "1rem",
    "full": "9999px"
  },
  "shadows": {
    "sm": "0 1px 2px rgba(0,0,0,0.05)",
    "md": "0 4px 6px rgba(0,0,0,0.1)",
    "lg": "0 10px 15px rgba(0,0,0,0.1)"
  }
}
```

#### Claude Code에서 사용

```
claude

You: design-system.json을 참고해서 
     Button 컴포넌트를 만들어줘

Claude:
디자인 시스템을 적용한 Button 컴포넌트를 생성합니다.

[design-system.json 읽기]

✓ 색상 스키마 적용
✓ 타이포그래피 적용
✓ 간격 시스템 적용

생성된 파일:
- src/components/Button.jsx
- src/components/Button.stories.jsx (Storybook)
```

#### Antigravity에서 사용

```
antigravity

You: design-system.json의 스타일을 사용해서
     Stitch로 대시보드를 디자인해줘

Antigravity:
[디자인 시스템 로드]

✓ Primary 색상: #3b82f6
✓ 폰트: Inter
✓ 간격: 1rem 기준

[Stitch 디자인 생성]

디자인 시스템이 일관되게 적용되었습니다!
```

### 컴포넌트 라이브러리 구축

재사용 가능한 컴포넌트를 체계적으로 관리합니다.

#### 구조

```
components-library/
├── design-system.json
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.stories.jsx
│   │   ├── Button.test.js
│   │   └── README.md
│   ├── Card/
│   ├── Input/
│   └── Modal/
├── stitch-designs/
│   ├── button-variants.json
│   ├── card-layouts.json
│   └── modal-templates.json
└── README.md
```

#### 워크플로우

**1. Antigravity로 디자인 생성**

```
antigravity

You: Button 컴포넌트의 모든 변형을 디자인해줘.
     
     변형:
     - primary, secondary, outline, ghost
     - small, medium, large
     - 기본, hover, active, disabled 상태

Antigravity:
[Stitch로 16가지 변형 디자인]

✓ 모든 변형 생성
✓ stitch-designs/button-variants.json에 저장

미리보기:
https://stitch.withgoogle.com/preview/button-lib-xyz
```

**2. Claude Code로 구현**

```
claude

You: stitch-designs/button-variants.json을 참고해서
     완전한 Button 컴포넌트를 구현해줘.
     
     요구사항:
     - TypeScript
     - Accessibility (WCAG 2.1 AA)
     - 테스트 (Jest + React Testing Library)
     - Storybook 문서

Claude:
[구현 진행]

✓ Button.tsx
✓ Button.test.tsx (20 tests)
✓ Button.stories.tsx
✓ README.md

모든 변형 구현 완료!
테스트 통과율: 100%
```

**3. 문서화 및 배포**

```
antigravity

You: 컴포넌트 라이브러리를 Storybook으로 배포하고
     npm 패키지로 publish해줘

Antigravity:
1. Storybook 빌드
✓ storybook build

2. npm 패키지 설정
✓ package.json 최적화
✓ TypeScript 타입 정의
✓ README.md 생성

3. 배포
✓ npm publish @mycompany/components
✓ Storybook: https://mycompany.github.io/components

완료!
```

## 도구별 최적 활용

### Claude Code 활용 시나리오

#### 시나리오 1: 복잡한 상태 관리

```bash
claude

You: 화이트보드 앱의 상태 관리를 Redux Toolkit으로 리팩토링해줘.
     
     상태:
     - 도구 설정 (현재 도구, 색상, 크기)
     - 캔버스 상태 (줌, 팬, 선택)
     - 히스토리 (undo/redo)
     - 사용자 (목록, presence)
     - 네트워크 (연결 상태, 동기화)

Claude: Redux Toolkit으로 상태 관리를 구조화하겠습니다.

구조:
store/
├── index.js
├── slices/
│   ├── toolSlice.js
│   ├── canvasSlice.js
│   ├── historySlice.js
│   ├── usersSlice.js
│   └── networkSlice.js
├── middleware/
│   ├── firebaseSync.js
│   └── logger.js
└── selectors/
    └── index.js

? 구현할까요? (y/n)
```

진행:

```
✓ Redux store 설정
✓ 5개 slices 생성
✓ Middleware (Firebase 동기화)
✓ Selectors (memoization)
✓ 기존 코드 마이그레이션
✓ 테스트 작성

마이그레이션 완료!
- 컴포넌트 복잡도: 40% 감소
- 리렌더링: 60% 감소
- 테스트 커버리지: 95%
```

**핵심 코드:**

toolSlice.js:

```javascript
import { createSlice } from '@reduxjs/toolkit';

const toolSlice = createSlice({
  name: 'tool',
  initialState: {
    currentTool: 'pen',
    color: '#000000',
    size: 3,
    opacity: 1
  },
  reducers: {
    setTool: (state, action) => {
      state.currentTool = action.payload;
    },
    setColor: (state, action) => {
      state.color = action.payload;
    },
    setSize: (state, action) => {
      state.size = action.payload;
    },
    setOpacity: (state, action) => {
      state.opacity = action.payload;
    }
  }
});

export const { setTool, setColor, setSize, setOpacity } = toolSlice.actions;
export default toolSlice.reducer;
```

historySlice.js (복잡한 undo/redo 로직):

```javascript
import { createSlice } from '@reduxjs/toolkit';

const MAX_HISTORY = 50;

const historySlice = createSlice({
  name: 'history',
  initialState: {
    past: [],
    present: null,
    future: []
  },
  reducers: {
    performAction: (state, action) => {
      const { past, present } = state;
      
      return {
        past: [...past, present].slice(-MAX_HISTORY),
        present: action.payload,
        future: []
      };
    },
    undo: (state) => {
      const { past, present, future } = state;
      
      if (past.length === 0) return state;
      
      const previous = past[past.length - 1];
      const newPast = past.slice(0, past.length - 1);
      
      return {
        past: newPast,
        present: previous,
        future: [present, ...future]
      };
    },
    redo: (state) => {
      const { past, present, future } = state;
      
      if (future.length === 0) return state;
      
      const next = future[0];
      const newFuture = future.slice(1);
      
      return {
        past: [...past, present],
        present: next,
        future: newFuture
      };
    }
  }
});

export const { performAction, undo, redo } = historySlice.actions;
export default historySlice.reducer;
```

#### 시나리오 2: 성능 프로파일링 및 최적화

```
You: React DevTools Profiler 데이터를 분석하고
     성능 병목을 해결해줘.
     
     [profiler-screenshot.png 첨부]

Claude: 프로파일 데이터를 분석하겠습니다.

발견된 문제:

1. Canvas 컴포넌트 불필요한 리렌더링 (초당 60회)
   원인: 부모 state 변경
   해결: React.memo + 적절한 props

2. Sidebar 리스트 렌더링 느림 (200ms)
   원인: 가상화 없음
   해결: react-window 적용

3. Firestore 리스너 과다 (47개)
   원인: useEffect 의존성 문제
   해결: 리스너 통합 및 최적화

적용 순서:
1. Canvas 최적화 (가장 큰 영향)
2. 리스트 가상화
3. Firestore 리스너 정리

? 진행할까요? (y/n)
```

최적화 적용:

```
1. Canvas 컴포넌트 최적화

Before:
function Canvas({ strokes, tool, ... }) {
  렌더링 로직
}

After:
const Canvas = React.memo(({ strokes, tool, ... }) => {
  렌더링 로직
}, (prevProps, nextProps) => {
  커스텀 비교: strokes 배열 길이와 tool만 체크
  return prevProps.strokes.length === nextProps.strokes.length
    && prevProps.tool === nextProps.tool;
});

결과: 리렌더링 60회/초 → 1회/초

2. 가상화 적용

Before:
<div className="user-list">
  {users.map(user => <UserItem key={user.id} user={user} />)}
</div>

After:
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={400}
  itemCount={users.length}
  itemSize={60}
  width="100%"
>
  {({ index, style }) => (
    <div style={style}>
      <UserItem user={users[index]} />
    </div>
  )}
</FixedSizeList>

결과: 렌더링 시간 200ms → 15ms

3. Firestore 리스너 통합

Before:
useEffect(() => {
  각 사용자마다 개별 리스너
}, [users]);

After:
useEffect(() => {
  단일 리스너로 모든 사용자 감지
  const unsubscribe = onSnapshot(...);
  return unsubscribe;
}, [roomId]);

결과: 리스너 47개 → 3개

전체 성능 개선:
- First Contentful Paint: 1.2s → 0.6s
- Time to Interactive: 2.5s → 1.0s
- Frame rate: 40 FPS → 60 FPS

완료! 🚀
```

### Antigravity 활용 시나리오

#### 시나리오 1: 멀티모달 프로토타이핑

```bash
antigravity

You: 첨부한 손그림 스케치를 보고 전자상거래 앱을 만들어줘.
     
     [hand-drawn-sketch.jpg 업로드]
     
     추가 요구사항:
     - Firebase로 상품 데이터 관리
     - Stripe 결제 통합
     - Firebase Hosting에 배포

Antigravity:
[이미지 분석 중...]

스케치 분석 결과:
- 홈 화면 (상품 그리드)
- 상품 상세 (이미지, 설명, 가격, 옵션)
- 장바구니 (목록, 합계)
- 결제 (주소, 카드 정보)

[Stitch로 각 화면 디자인]

✓ 4개 화면 디자인 완료

[Firebase 프로젝트 설정]

✓ Firestore (상품, 주문)
✓ Authentication (Google, Email)
✓ Cloud Functions (주문 처리)
✓ Storage (상품 이미지)

[Stripe 통합]

✓ Stripe 계정 연결
✓ Payment Intent API
✓ Webhook 설정

[구현]

✓ React 앱 생성
✓ 모든 화면 구현
✓ Firebase 연동
✓ Stripe 결제
✓ 테스트 데이터 추가

[배포]

✓ firebase deploy

완료!
URL: https://my-shop-xyz.web.app

테스트 카드: 4242 4242 4242 4242

스케치에서 실제 앱까지: 45분!
```

#### 시나리오 2: Firebase + GCP 풀스택 구축

```
You: 소셜 미디어 앱을 만들어줘.
     
     기능:
     - 이미지/동영상 포스팅
     - 좋아요, 댓글
     - 팔로우/언팔로우
     - 알림
     - 실시간 피드
     
     백엔드: Firebase + Cloud Functions
     프론트엔드: React
     
     확장 가능한 구조로 설계해줘.

Antigravity:
전체 아키텍처를 설계하고 구현하겠습니다.

아키텍처:

Frontend (React):
- react-router-dom
- React Query (서버 상태)
- Zustand (클라이언트 상태)

Backend (Firebase):
- Firestore (데이터)
- Storage (미디어)
- Cloud Functions (비즈니스 로직)
- Cloud Messaging (알림)

Collections:
- users/
- posts/
- comments/
- likes/
- follows/
- notifications/

Cloud Functions:
- onPostCreate (알림 생성)
- onLikeCreate (알림 생성)
- generateFeed (피드 집계)
- processImage (이미지 최적화)

[구현 진행...]

1. Firestore 스키마 설계
✓ 정규화된 구조
✓ 적절한 인덱스
✓ 보안 규칙

2. Cloud Functions 구현
✓ TypeScript
✓ 에러 처리
✓ 로깅
✓ 테스트

3. React 앱 구현
✓ 모든 화면
✓ 실시간 업데이트
✓ 이미지 업로드
✓ 최적화 (lazy loading, code splitting)

4. 배포
✓ Functions: deployed
✓ Hosting: deployed

완료!

URL: https://social-app-xyz.web.app

성능:
- Lighthouse: 95/100
- First Load: 1.1s
- Images: WebP + lazy loading

확장성:
- Firestore: 수백만 문서 지원
- Functions: 자동 스케일링
- CDN: 전 세계 배포

총 소요 시간: 2시간
```

### 도구 간 전환 전략

#### 언제 전환할까?

**Claude Code → Antigravity 전환 시점:**

1. Firebase 기능이 필요할 때
```
Claude Code로 복잡한 로직 완성
↓
Antigravity로 Firebase 통합 및 배포
```

2. 디자인 작업이 필요할 때
```
Claude Code로 API 구현 완료
↓
Antigravity + Stitch로 UI 생성
```

3. 이미지 기반 작업
```
Claude Code로 기본 구조 작성
↓
Antigravity로 스케치를 코드로 변환
```

**Antigravity → Claude Code 전환 시점:**

1. 복잡한 비즈니스 로직
```
Antigravity로 기본 구조 생성
↓
Claude Code로 복잡한 알고리즘 구현
```

2. 성능 최적화
```
Antigravity로 빠른 프로토타입
↓
Claude Code로 성능 개선
```

3. 테스트 작성
```
Antigravity로 기능 구현
↓
Claude Code로 포괄적인 테스트 작성
```

#### 전환 프로세스

**1. 작업 상태 저장**

Claude Code에서:
```bash
You: 현재까지의 작업을 정리해서 
     work-summary.md 파일로 저장해줘.
     Antigravity가 이어서 작업할 수 있도록.

Claude:
✓ work-summary.md 생성

내용:
- 완료된 작업
- 현재 상태
- 다음 작업
- 주의사항
```

**2. Antigravity로 전환**

```bash
antigravity

You: work-summary.md를 읽고
     Firebase 배포를 진행해줘

Antigravity:
[파일 읽기]

✓ 이전 작업 이해
✓ 다음 단계 파악

Firebase 배포를 시작합니다...
```

**3. 다시 Claude Code로 전환**

```bash
claude

You: Antigravity가 배포한 앱의
     성능 문제를 분석하고 최적화해줘

Claude:
✓ 배포된 앱 분석
✓ 성능 프로파일링
✓ 최적화 적용
```

---

## 문제 해결

### 일반적인 문제

#### 문제 1: Stitch MCP 연결 실패

**증상:**

```
✗ stitch (connection failed)
```

**원인 및 해결:**

**원인 1: 설정 파일 오류**

확인:
```bash
cat ~/.claude-code/config.json
```

올바른 형식:
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["-y", "davideast/stitch-mcp"]
    }
  }
}
```

**원인 2: 인증 만료**

재인증:
```bash
npx -y davideast/stitch-mcp init
```

**원인 3: 네트워크 문제**

테스트:
```bash
curl -I https://stitch.withgoogle.com
```

VPN 확인 및 프록시 설정

#### 문제 2: 두 도구 간 동기화 문제

**증상:**

Claude Code에서 변경한 파일을 Antigravity가 인식 못함

**해결:**

**방법 1: Git 사용**

```bash
Claude Code에서:
git add .
git commit -m "Update from Claude Code"

Antigravity에서:
git pull
```

**방법 2: 공유 디렉토리**

두 도구가 같은 프로젝트 디렉토리를 바라보도록 설정

```bash
cd /path/to/project

claude
작업...

exit

antigravity
작업...
```

**방법 3: 명시적 새로고침**

```
antigravity

You: 프로젝트 파일을 다시 읽어줘

Antigravity:
✓ 파일 시스템 새로고침
✓ 변경사항 감지
```

#### 문제 3: Stitch 할당량 초과

**증상:**

```
✗ Quota exceeded: 100 requests per day
```

**해결:**

**임시 해결:**

내일까지 대기 (무료 티어는 일일 100 requests)

**영구 해결:**

1. 요청 최적화

디자인을 한 번에 많이 생성하지 말고 점진적으로:

```
나쁜 예:
"10개 페이지 전부 디자인해줘"

좋은 예:
"홈페이지만 먼저 디자인해줘"
(확인 후)
"다음은 로그인 페이지"
```

2. 캐싱 활용

이미 생성한 디자인 재사용:

```json
{
  "cache": {
    "button": "design-id-123",
    "card": "design-id-456"
  }
}
```

3. 유료 플랜 고려

Enterprise: 무제한 requests

#### 문제 4: Firebase 배포 실패

**증상 (Antigravity):**

```
✗ Firebase deploy failed
Error: Authentication failed
```

**해결:**

```bash
1. Firebase 재로그인:
firebase logout
firebase login

2. 프로젝트 확인:
firebase projects:list
firebase use your-project-id

3. 권한 확인:
https://console.firebase.google.com
프로젝트 → Settings → Users and permissions

4. 재배포:
firebase deploy
```

### 플랫폼별 문제

#### macOS: Rosetta 관련

**문제:**

M1/M2 Mac에서 npx 실행 실패

**해결:**

```bash
Rosetta 설치:
softwareupdate --install-rosetta

또는 ARM 네이티브 Node.js:
brew install node
```

#### Windows: PowerShell 실행 정책

**문제:**

```
npx : File cannot be loaded because running scripts is disabled
```

**해결:**

```powershell
관리자 권한으로 PowerShell 실행:
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### Linux: 권한 문제

**문제:**

```
Error: EACCES: permission denied
```

**해결:**

```bash
npm 전역 디렉토리 권한 수정:
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 베스트 프랙티스

### 워크플로우 최적화

#### 1. 작업 분할

**좋은 예:**

```
Phase 1 (Antigravity): 디자인 + Firebase 설정
- Stitch로 모든 화면 디자인 (30분)
- Firebase 프로젝트 생성 (10분)

Phase 2 (Claude Code): 비즈니스 로직
- API 엔드포인트 구현 (2시간)
- 테스트 작성 (1시간)

Phase 3 (Antigravity): 통합 + 배포
- 프론트-백엔드 연결 (30분)
- Firebase 배포 (15분)
```

**나쁜 예:**

```
도구를 계속 왔다갔다:
Antigravity로 UI 일부
↓
Claude Code로 로직 일부
↓
Antigravity로 UI 추가
↓
Claude Code로 로직 추가
...

비효율적!
```

#### 2. 문서화

**프로젝트 루트에 WORKFLOW.md 생성:**

```markdown
# 프로젝트 워크플로우

## 도구 사용 가이드

### Antigravity 담당
- UI 디자인 (Stitch)
- Firebase 설정
- 배포 및 모니터링

### Claude Code 담당
- 복잡한 비즈니스 로직
- 성능 최적화
- 테스트 작성

## 현재 상태
- Phase: 2 (비즈니스 로직 구현)
- 담당 도구: Claude Code
- 진행률: 60%

## 다음 단계
1. 결제 로직 완성 (Claude Code)
2. Firebase Functions 배포 (Antigravity)
3. 통합 테스트 (Claude Code)
```

#### 3. 코드 리뷰 프로세스

**Antigravity가 생성한 코드를 Claude Code로 리뷰:**

```bash
claude

You: Antigravity가 생성한 src/components/ 폴더를
     코드 리뷰해줘.
     
     중점:
     - 성능
     - 접근성
     - 보안
     - 베스트 프랙티스

Claude:
코드 리뷰를 진행합니다...

발견된 이슈:

1. 성능 (중요도: 높음)
   파일: ProductList.jsx
   문제: 리스트 가상화 없음
   제안: react-window 적용

2. 접근성 (중요도: 높음)
   파일: Button.jsx
   문제: ARIA 레이블 누락
   제안: aria-label 추가

3. 보안 (중요도: 중간)
   파일: UserProfile.jsx
   문제: XSS 취약점 가능성
   제안: DOMPurify 사용

? 자동으로 수정할까요? (y/n)
```

### 팀 협업

#### Git 브랜치 전략

```
main
├── feature/ui-design (Antigravity)
├── feature/api-logic (Claude Code)
└── feature/integration (Both)
```

**브랜치별 역할:**

```bash
feature/ui-design (Antigravity만):
- Stitch 디자인
- React 컴포넌트 기본 구조
- Firebase UI 연동

feature/api-logic (Claude Code만):
- 백엔드 API
- 비즈니스 로직
- 테스트

feature/integration (협업):
- 프론트-백 연결
- E2E 테스트
- 배포 준비
```

#### 커밋 메시지 규칙

```
[Tool] Type: Subject

Body

Tool: Claude | Antigravity
Type: feat | fix | refactor | test | docs
```

**예시:**

```bash
[Antigravity] feat: Add product card design with Stitch

- Generated 3 card variants using Stitch MCP
- Implemented responsive layout
- Added Firebase image integration

[Claude] refactor: Optimize product search algorithm

- Reduced search time from 200ms to 50ms
- Implemented debouncing
- Added tests (95% coverage)
```

### 비용 최적화

#### Stitch MCP 사용량 관리

**전략:**

1. **디자인 재사용**

```
디자인 라이브러리 구축:
designs/
├── buttons.json
├── cards.json
└── forms.json

새 프로젝트에서 재사용
```

2. **배치 생성**

```
나쁜 예:
버튼 디자인 → 요청 1
카드 디자인 → 요청 1
폼 디자인 → 요청 1
총 3 requests

좋은 예:
"버튼, 카드, 폼을 한 번에 디자인해줘"
총 1 request
```

3. **로컬 캐싱**

```javascript
const designCache = new Map();

async function getDesign(prompt) {
  if (designCache.has(prompt)) {
    return designCache.get(prompt);
  }
  
  const design = await stitch.generate(prompt);
  designCache.set(prompt, design);
  return design;
}
```

#### Firebase 비용 최적화

**Firestore:**

```javascript
나쁜 예 (매번 전체 문서):
const users = await getDocs(collection(db, 'users'));

좋은 예 (필요한 것만):
const users = await getDocs(
  query(
    collection(db, 'users'),
    where('active', '==', true),
    limit(10)
  )
);
```

**Cloud Functions:**

```javascript
나쁜 예 (항상 실행):
exports.onWrite = functions.firestore
  .document('posts/{id}')
  .onWrite(async (change, context) => {
    항상 실행되는 로직
  });

좋은 예 (조건부):
exports.onWrite = functions.firestore
  .document('posts/{id}')
  .onWrite(async (change, context) => {
    if (!change.after.exists) {
      return; // 삭제 시 종료
    }
    
    const before = change.before.data();
    const after = change.after.data();
    
    if (before.status === after.status) {
      return; // 변경 없으면 종료
    }
    
    필요한 경우에만 실행
  });
```

---

## 리소스

### 공식 문서

**Stitch MCP:**
- [GitHub Repository](https://github.com/davideast/stitch-mcp)
- [Setup Guide](https://stitch.withgoogle.com/docs/mcp/setup)
- [API Reference](https://stitch.withgoogle.com/docs/api)

**Claude Code:**
- [Documentation](https://code.claude.com/docs)
- [GitHub Discussions](https://github.com/anthropics/claude-code/discussions)
- [API Reference](https://docs.anthropic.com/claude/reference)

**Antigravity:**
- [Getting Started](https://antigravity.dev/docs)
- [Firebase Integration](https://antigravity.dev/docs/firebase)
- [Examples](https://github.com/google/antigravity-examples)

### 커뮤니티

**Discord:**
- Claude Code Discord
- Google AI Discord

**Reddit:**
- r/ClaudeAI
- r/Firebase
- r/webdev

**Stack Overflow:**
- [stitch-mcp] 태그
- [claude-code] 태그
- [antigravity] 태그

### 예제 프로젝트

**Starter Templates:**

```bash
Claude Code 템플릿:
npx create-claude-app my-app

Antigravity 템플릿:
npx create-antigravity-app my-app

통합 템플릿:
npx create-stitch-app my-app
```

**GitHub 저장소:**

- [awesome-stitch-mcp](https://github.com/examples/awesome-stitch-mcp)
- [claude-code-examples](https://github.com/anthropics/examples)
- [antigravity-starters](https://github.com/google/antigravity-starters)

### 학습 자료

**비디오 튜토리얼:**
- "Stitch MCP 시작하기" (YouTube)
- "Claude Code + Antigravity 워크플로우" (YouTube)
- "Firebase + Stitch 통합" (YouTube)

**블로그 포스트:**
- [Stitch MCP 완전 가이드](https://blog.example.com)
- [두 AI 도구로 풀스택 개발](https://blog.example.com)
- [디자인에서 배포까지 1시간](https://blog.example.com)

### 도구 및 플러그인

**VS Code Extensions:**
- Claude Code Extension
- Stitch MCP Viewer
- Firebase Tools

**Chrome Extensions:**
- Stitch Design Inspector
- Firebase Debugger

**CLI Tools:**
- stitch-cli (디자인 관리)
- claude-cli (명령줄 인터페이스)

---

## 결론

### 핵심 요약

**두 도구의 시너지:**

Claude Code와 Antigravity를 함께 사용하면:
- 개발 속도 **10배** 향상
- 코드 품질 최고 수준 유지
- Firebase/GCP 완벽 통합
- 디자인부터 배포까지 원스톱

**최적 워크플로우:**

1. **디자인** (Antigravity + Stitch): 15분
2. **개발** (Claude Code): 2시간
3. **통합** (Antigravity): 30분
4. **배포** (Antigravity): 15분

총 3시간으로 프로덕션급 앱 완성!

**도구 선택 기준:**

Claude Code를 사용하세요:
- 복잡한 로직
- 성능 최적화
- 테스트 작성
- 코드 리뷰

Antigravity를 사용하세요:
- UI 디자인
- Firebase 통합
- 멀티모달 입력
- 빠른 프로토타입

### 다음 단계

1. **환경 설정**: 두 도구 모두 설치 및 Stitch 연동
2. **샘플 프로젝트**: 간단한 TODO 앱으로 연습
3. **실전 프로젝트**: 본격적인 앱 개발
4. **팀 도입**: 협업 워크플로우 구축

### 최종 조언

**실패를 두려워하지 마세요.**

AI 도구는 빠른 실험을 가능하게 합니다. 작동하지 않으면 다시 시도하세요. 시간이 많이 들지 않습니다.

**각 도구의 강점을 살리세요.**

모든 것을 하나의 도구로 하려 하지 마세요. 상황에 맞는 도구를 선택하세요.

**지속적으로 학습하세요.**

AI 도구는 빠르게 진화합니다. 새로운 기능과 베스트 프랙티스를 계속 학습하세요.

**커뮤니티에 기여하세요.**

여러분의 경험을 공유하세요. 다른 개발자들도 도움을 받을 수 있습니다.

이제 여러분은 Stitch MCP + Claude Code + Antigravity의 완전한 마스터입니다! 

놀라운 것을 만들어보세요! 🚀

---

**작성일자:** 2026-01-23

