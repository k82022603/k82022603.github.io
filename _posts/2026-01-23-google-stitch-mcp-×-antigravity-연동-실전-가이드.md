---
title: "Google Stitch MCP × Antigravity 연동 실전 가이드"
date: 2026-01-23 06:10:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Guide,  Google,  google-cloud,  Stitch,  MCP,  Antigravity,  Claude.write]
---


> 자연어로 프로페셔널 UI 디자인하고 즉시 코드로 변환하기

## 목차

1. [개요](#개요)
2. [사전 준비사항](#사전-준비사항)
3. [단계별 설치 가이드](#단계별-설치-가이드)
4. [실전 사용 예제](#실전-사용-예제)
5. [고급 활용법](#고급-활용법)
6. [문제 해결](#문제-해결)
7. [실전 팁과 베스트 프랙티스](#실전-팁과-베스트-프랙티스)

---

## 📝 가이드 사용 안내

이 가이드에서 대문자로 표기된 값들은 **placeholder**입니다. 실제 명령어 실행 시 여러분의 값으로 교체하세요.

**Placeholder 예시:**
- `YOUR_PROJECT_ID` → `stitch-mcp-485109`
- `YOUR_EMAIL@gmail.com` → `john@gmail.com`
- `/Users/{username}/` → `/Users/john/`
- `{path}` → 실제 경로

**중요:** 모든 명령어를 복사-붙여넣기 하기 전에 반드시 실제 값으로 수정하세요!

---

## 개요

### Stitch MCP + Antigravity란?

Google Stitch MCP(Model Context Protocol) Server를 Antigravity와 연동하면, 자연어 명령만으로 전문가 수준의 UI 디자인을 생성하고 즉시 HTML/CSS 코드로 변환할 수 있습니다.

**핵심 가치:**

1. **디자인 자동화** - "대시보드 디자인해줘" → 완성된 UI
2. **즉시 코드 변환** - 디자인을 HTML/CSS로 자동 생성
3. **프로덕션 품질** - 실제 프로젝트에 바로 사용 가능
4. **시간 절약** - 95% 이상의 개발 시간 단축

### 왜 Stitch MCP + Antigravity인가?

Antigravity는 Google의 AI 코딩 도구로, Gemini 3 Pro와 긴밀하게 통합되어 있습니다. Stitch MCP를 연동하면 혁신적인 워크플로우가 가능해집니다.

**전통적인 방식:**

1. 디자이너가 Figma에서 디자인 (1-2주)
2. 개발자가 HTML/CSS 코딩 (1-2주)
3. 백엔드 연동 및 테스트 (1주)
4. **총 4-5주 소요**

**Stitch MCP + Antigravity 방식:**

1. 자연어로 디자인 요청 (1분)
2. Stitch가 디자인 생성 (1-2분)
3. HTML/CSS 코드 자동 생성 (즉시)
4. Gemini가 백엔드 연동 코드 작성 (10-30분)
5. **총 1-2시간 소요**

**결과:** 시간이 95% 이상 단축되는 혁신적인 변화입니다.

### 실제 활용 사례

**스타트업 MVP 개발:**
- 1주일 → 1일로 단축
- 디자이너 없이도 전문적인 UI
- 빠른 시장 검증

**프로토타이핑:**
- 아이디어를 즉시 시각화
- 투자자 피칭 자료
- A/B 테스트 빠른 준비

**개인 프로젝트:**
- 디자인 스킬 불필요
- 프로페셔널한 결과물
- 학습 곡선 최소화

---

## 사전 준비사항

### 필수 요구사항

**1. Google Cloud 계정**

계정 요구사항:
- Google 계정 (Gmail 또는 Workspace)
- Google Cloud 프로젝트 생성 권한
- 결제 정보 등록 (무료 크레딧 사용 가능)

참고:
- Stitch API는 현재 무료입니다
- 무료 계정으로도 충분히 사용 가능
- 과금 걱정 없이 시작할 수 있습니다

**2. Antigravity 설치**

Antigravity는 Google의 AI 코딩 도구입니다.

설치 방법:
```bash
macOS (Homebrew):
brew install google-antigravity

Linux (스크립트):
curl -fsSL https://antigravity.dev/install.sh | bash

Windows (WinGet):
winget install Google.Antigravity
```

확인:
```bash
antigravity --version
```

예상 출력: `Antigravity v1.8.2`

**3. gcloud CLI**

davideast/stitch-mcp가 자동으로 처리하지만, 문제 발생 시 수동 설치:

다운로드:
- [Google Cloud SDK 설치 페이지](https://cloud.google.com/sdk/docs/install)

권장 버전:
- 553.0.0 이상

확인:
```bash
gcloud --version
```

**4. Node.js 및 npm**

Node.js 18 이상 필요:

설치 확인:
```bash
node --version
npm --version
```

필요한 버전:
- Node.js: v18.0.0 이상 (LTS 권장)
- npm: v9.0.0 이상

설치 필요 시:
- [Node.js 공식 사이트](https://nodejs.org)에서 LTS 버전 다운로드

**5. 인터넷 연결**

요구사항:
- 안정적인 인터넷 연결
- OAuth 인증을 위한 브라우저 접근
- Google Cloud 접근 가능 (VPN 차단 확인)

### 권장 사양

**하드웨어:**
- RAM: 8GB 이상 (16GB 권장)
- 저장공간: 5GB 이상 여유 공간
- CPU: 듀얼 코어 이상

**운영체제:**
- macOS 12 (Monterey) 이상
- Ubuntu 20.04 LTS 이상
- Windows 10 이상 (Windows 11 권장)

---

## 단계별 설치 가이드

### 0단계: 공식 문서 확인 (중요!)

설치를 시작하기 전에 반드시 공식 문서를 읽어보세요.

**공식 문서:**
- [Stitch MCP Setup Guide](https://stitch.withgoogle.com/docs/mcp/setup)
- [davideast/stitch-mcp GitHub](https://github.com/davideast/stitch-mcp)

**주의사항:**
- 이 가이드는 2026년 1월 23일 기준입니다
- API 스펙이 변경될 수 있으므로 최신 정보 확인 필수
- 공식 문서가 항상 최신 정보입니다

### 1단계: Stitch MCP 초기화

터미널을 열고 다음 명령을 실행합니다.

**명령어:**

```bash
npx -y davideast/stitch-mcp init
```

설명:
- `npx -y`: npm 패키지를 설치 없이 바로 실행 (자동 승인)
- `davideast/stitch-mcp`: GitHub의 Stitch MCP 패키지
- `init`: 초기 설정 시작

**예상 출력:**

```
🎨 Stitch MCP Setup
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📦 Checking dependencies...
✓ Node.js found (v20.11.0)
✓ npm found (v10.2.4)

🔍 Checking for gcloud...
✗ gcloud not found

📥 Installing Google Cloud SDK...
```

자동 진행 사항:
1. 의존성 확인 (Node.js, npm)
2. gcloud CLI 존재 여부 확인
3. 없으면 자동 설치 시작
4. 설치 완료 후 다음 단계로

**시간:** 약 2-5분 소요 (인터넷 속도에 따라 다름)

### 2단계: Google Cloud 프로젝트 생성

초기화가 완료되면 자동으로 프로젝트 생성 프롬프트가 나타납니다.

**프롬프트:**

```
? Would you like to create a new Google Cloud project? (Y/n)
```

**Y 선택 후:**

```
? Enter a unique project ID (e.g., stitch-mcp-12345):
```

**프로젝트 ID 입력 팁:**

좋은 예시:
- `stitch-mcp-485109`
- `my-stitch-project-2026`
- `company-stitch-dev`

나쁜 예시:
- `test` (너무 짧음, 중복 가능성 높음)
- `my project` (공백 불가)
- `프로젝트` (영문만 가능)

규칙:
- 6-30자
- 소문자, 숫자, 하이픈만 가능
- 글로벌 고유값 (다른 사람이 이미 사용 중이면 불가)
- 시작은 소문자

**자동 작업:**

```
Creating Google Cloud project...
✓ Project created: stitch-mcp-485109
✓ Billing enabled
✓ APIs enabled:
  - Stitch API
  - Cloud Resource Manager API
✓ Service account created
```

프로젝트 생성 시:
1. 프로젝트 ID로 GCP 프로젝트 생성
2. 필요한 API 자동 활성화
3. 서비스 계정 자동 생성
4. 권한 자동 설정

**시간:** 약 30초-1분

### 3단계: OAuth 인증

브라우저가 자동으로 열립니다.

**브라우저 화면:**

1. **Google 로그인 페이지**

```
Sign in with Google
━━━━━━━━━━━━━━━━━━

Email or phone
[_______________]

Next
```

Google 계정으로 로그인하세요.

2. **권한 승인 화면**

```
Stitch MCP wants to access your Google Account

This will allow Stitch MCP to:
✓ View and manage data across Google Cloud services
✓ Generate and manage designs
✓ Access project resources

Cancel   Allow
```

**Allow 클릭**

설명:
- Stitch가 디자인을 생성하고 관리하기 위해 필요
- 프로젝트 리소스 접근 권한
- 안전한 OAuth 2.0 인증

3. **인증 완료**

```
✓ Authentication successful!

You can close this window and return to the terminal.
```

브라우저를 닫고 터미널로 돌아갑니다.

**터미널 출력:**

```
✓ Authentication successful!
✓ Credentials saved

🎉 Setup complete!
```

**저장 위치:**
- macOS/Linux: `~/.config/stitch-mcp/credentials.json`
- Windows: `%APPDATA%\stitch-mcp\credentials.json`

참고: 이 파일은 절대 공유하지 마세요!

### 4단계: Antigravity 설정

Stitch MCP를 Antigravity에 연결합니다.

**설정 파일 위치:**

macOS/Linux:
```bash
~/.antigravity/config.json
```

Windows:
```powershell
%USERPROFILE%\.antigravity\config.json
```

**설정 파일 편집:**

```bash
nano ~/.antigravity/config.json
```

또는 VS Code:
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

설명:
- `mcpServers`: MCP 서버 목록
- `stitch`: 서버 이름 (원하는 이름 사용 가능)
- `command`: 실행 명령어 (npx)
- `args`: 명령어 인자 배열

**기존 설정이 있는 경우:**

기존:
```json
{
  "mcpServers": {
    "other-server": {
      "command": "some-command"
    }
  }
}
```

수정 후:
```json
{
  "mcpServers": {
    "other-server": {
      "command": "some-command"
    },
    "stitch": {
      "command": "npx",
      "args": ["-y", "davideast/stitch-mcp"]
    }
  }
}
```

주의: JSON 형식을 정확히 지켜야 합니다 (쉼표, 중괄호 등).

**파일 저장:**

nano 사용 시:
- `Ctrl + O` (저장)
- `Enter` (확인)
- `Ctrl + X` (종료)

VS Code 사용 시:
- `Cmd/Ctrl + S` (저장)

### 5단계: Antigravity 재시작

설정을 적용하기 위해 Antigravity를 재시작합니다.

**현재 실행 중이면 종료:**

```bash
antigravity quit
```

또는 프로세스 강제 종료:

macOS/Linux:
```bash
pkill -f antigravity
```

Windows:
```powershell
taskkill /F /IM antigravity.exe
```

**재시작:**

```bash
antigravity
```

**연결 확인:**

Antigravity를 시작하면:

```
Antigravity v1.8.2 | Gemini 3 Pro

🔌 MCP Servers connected:
  ✓ stitch (davideast/stitch-mcp)

Ready!
```

`stitch` 서버가 목록에 표시되면 성공입니다!

**연결 실패 시:**

```
✗ stitch (connection failed)
```

문제 해결 섹션을 참조하세요.

---

## 실전 사용 예제

### 예제 1: 간단한 대시보드 생성

가장 기본적인 사용 예제입니다.

**1단계: Antigravity 시작**

```bash
antigravity
```

**2단계: 디자인 요청**

Antigravity 프롬프트에서:

```
You: 관리자 대시보드를 디자인해줘. 
     왼쪽에 사이드바, 오른쪽에 통계 카드들이 있어야 해.
```

**3단계: Stitch 작동**

Antigravity가 자동으로 Stitch MCP를 호출합니다:

```
Using MCP: stitch

Generating design...

✓ Design generated
✓ Preview available
```

**4단계: 미리보기**

생성된 디자인 URL:

```
🎨 Design Preview:
https://stitch.withgoogle.com/preview/abc123def456

Components:
- Sidebar (left, 240px width)
- Header (top, 64px height)
- Stats cards (4x grid)
- Chart area (main content)

? Generate HTML/CSS code? (Y/n)
```

**5단계: 코드 생성**

Y를 선택하면:

```
Generating HTML/CSS...

✓ index.html created
✓ styles.css created
✓ Files saved to: ./dashboard/

? Open in browser? (Y/n)
```

**6단계: 결과 확인**

생성된 파일 구조:

```
dashboard/
├── index.html
├── styles.css
└── assets/
    └── (아이콘 등)
```

브라우저에서 확인:

```bash
cd dashboard
open index.html
```

또는:

```bash
python -m http.server 8000
```

그리고 브라우저에서 `http://localhost:8000` 접속

**생성된 코드 예시:**

index.html (구조):

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>관리자 대시보드</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="dashboard">
        사이드바:
        <aside class="sidebar">
            <div class="logo">Dashboard</div>
            <nav>
                <a href="#" class="nav-item active">홈</a>
                <a href="#" class="nav-item">통계</a>
                <a href="#" class="nav-item">사용자</a>
                <a href="#" class="nav-item">설정</a>
            </nav>
        </aside>
        
        메인 컨텐츠:
        <main class="main-content">
            <header class="header">
                <h1>대시보드</h1>
                <div class="user-info">관리자</div>
            </header>
            
            <div class="stats-grid">
                <div class="stat-card">
                    <div class="stat-value">1,234</div>
                    <div class="stat-label">총 사용자</div>
                </div>
                <div class="stat-card">
                    <div class="stat-value">567</div>
                    <div class="stat-label">활성 사용자</div>
                </div>
                <div class="stat-card">
                    <div class="stat-value">89</div>
                    <div class="stat-label">신규 가입</div>
                </div>
                <div class="stat-card">
                    <div class="stat-value">$12,345</div>
                    <div class="stat-label">매출</div>
                </div>
            </div>
            
            <div class="chart-container">
                차트 영역 (차트 라이브러리 추가 필요)
            </div>
        </main>
    </div>
</body>
</html>
```

styles.css (스타일):

```css
/* 기본 리셋 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: #f5f5f5;
}

/* 대시보드 레이아웃 */
.dashboard {
    display: flex;
    min-height: 100vh;
}

/* 사이드바 */
.sidebar {
    width: 240px;
    background: #1e293b;
    color: white;
    padding: 20px;
}

.logo {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 40px;
}

.nav-item {
    display: block;
    padding: 12px 16px;
    color: #94a3b8;
    text-decoration: none;
    border-radius: 8px;
    margin-bottom: 8px;
    transition: all 0.2s;
}

.nav-item:hover {
    background: #334155;
    color: white;
}

.nav-item.active {
    background: #3b82f6;
    color: white;
}

/* 메인 컨텐츠 */
.main-content {
    flex: 1;
    padding: 24px;
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 32px;
}

.header h1 {
    font-size: 28px;
    color: #1e293b;
}

/* 통계 카드 그리드 */
.stats-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 24px;
    margin-bottom: 32px;
}

.stat-card {
    background: white;
    padding: 24px;
    border-radius: 12px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.stat-value {
    font-size: 32px;
    font-weight: bold;
    color: #1e293b;
    margin-bottom: 8px;
}

.stat-label {
    font-size: 14px;
    color: #64748b;
}

/* 차트 컨테이너 */
.chart-container {
    background: white;
    padding: 24px;
    border-radius: 12px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    height: 400px;
}

/* 반응형 */
    .stats-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}
```

### 예제 2: 이커머스 제품 카드

더 구체적인 요청을 해봅시다.

**요청:**

```
You: 이커머스 제품 카드를 디자인해줘.
     
     요구사항:
     - 제품 이미지 (정사각형)
     - 제품명 (2줄 제한)
     - 가격 (원래 가격 + 할인가)
     - 별점 (5점 만점)
     - "장바구니 추가" 버튼
     - 호버 시 그림자 효과
     - 위시리스트 하트 아이콘 (우측 상단)
     
     현대적이고 미니멀한 디자인으로 해줘.
```

**생성 결과:**

```
✓ Product card design generated

Components:
- Image container (aspect-ratio: 1:1)
- Product info section
- Price display (strikethrough + discount)
- Star rating (5 stars)
- Add to cart button
- Wishlist heart icon

Interactions:
- Hover: shadow elevation
- Heart: toggle favorite (static)
- Button: ripple effect

? Generate code? (Y/n)
```

**생성된 HTML:**

```html
<div class="product-card">
    <div class="image-container">
        <img src="product.jpg" alt="제품명">
        <button class="wishlist-btn">
            <svg width="24" height="24" viewBox="0 0 24 24">
                하트 아이콘 SVG
            </svg>
        </button>
    </div>
    
    <div class="product-info">
        <h3 class="product-name">무선 블루투스 이어폰 프리미엄 에디션</h3>
        
        <div class="price-section">
            <span class="original-price">₩159,000</span>
            <span class="discount-price">₩99,000</span>
            <span class="discount-badge">38% OFF</span>
        </div>
        
        <div class="rating">
            <div class="stars">★★★★☆</div>
            <span class="review-count">(124)</span>
        </div>
        
        <button class="add-to-cart">장바구니 추가</button>
    </div>
</div>
```

**생성된 CSS:**

```css
.product-card {
    width: 280px;
    background: white;
    border-radius: 12px;
    overflow: hidden;
    transition: box-shadow 0.3s ease;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.product-card:hover {
    box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}

.image-container {
    position: relative;
    aspect-ratio: 1 / 1;
    overflow: hidden;
    background: #f5f5f5;
}

.image-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.wishlist-btn {
    position: absolute;
    top: 12px;
    right: 12px;
    width: 40px;
    height: 40px;
    border: none;
    background: white;
    border-radius: 50%;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.product-info {
    padding: 16px;
}

.product-name {
    font-size: 16px;
    font-weight: 600;
    line-height: 1.4;
    margin-bottom: 12px;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

.price-section {
    margin-bottom: 8px;
}

.original-price {
    font-size: 14px;
    color: #999;
    text-decoration: line-through;
    margin-right: 8px;
}

.discount-price {
    font-size: 20px;
    font-weight: bold;
    color: #e74c3c;
}

.discount-badge {
    font-size: 12px;
    color: white;
    background: #e74c3c;
    padding: 2px 8px;
    border-radius: 4px;
    margin-left: 8px;
}

.rating {
    display: flex;
    align-items: center;
    margin-bottom: 16px;
}

.stars {
    color: #ffc107;
    margin-right: 8px;
}

.review-count {
    font-size: 14px;
    color: #666;
}

.add-to-cart {
    width: 100%;
    padding: 12px;
    background: #3b82f6;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.2s;
}

.add-to-cart:hover {
    background: #2563eb;
}
```

### 예제 3: 복잡한 다단계 요청

실제 프로젝트처럼 단계적으로 발전시켜봅시다.

**1단계: 초기 디자인**

```
You: SaaS 앱의 가격 페이지를 디자인해줘.
     3개 플랜: Free, Pro, Enterprise
```

**2단계: 피드백 및 수정**

```
You: 좋은데, Pro 플랜을 강조해줘.
     "Most Popular" 배지를 추가하고 살짝 크게 만들어줘.
```

**3단계: 세부 조정**

```
You: 각 플랜마다 기능 목록을 추가해줘.
     체크마크 아이콘을 사용하고,
     Enterprise는 "Contact Us" 버튼으로 변경해줘.
```

**4단계: 최종 다듬기**

```
You: 다크모드 버전도 만들어줘.
     toggle 스위치를 추가해서 전환할 수 있게.
```

**최종 결과:**

Stitch가 각 단계마다 디자인을 업데이트하고, 최종적으로 완성된 가격 페이지 코드를 생성합니다.

---

## 고급 활용법

### 디자인 시스템 정의하기

일관된 디자인을 위해 디자인 시스템을 먼저 정의할 수 있습니다.

**방법 1: 프롬프트에 포함**

```
You: 우리 회사 디자인 시스템:
     
     색상:
     - Primary: #3b82f6
     - Secondary: #8b5cf6
     - Success: #10b981
     - Danger: #ef4444
     
     폰트:
     - 제목: Inter Bold
     - 본문: Inter Regular
     
     간격:
     - 작음: 8px
     - 중간: 16px
     - 큼: 24px
     
     이 시스템을 사용해서 랜딩 페이지를 디자인해줘.
```

**방법 2: 설정 파일 사용**

`design-system.json` 생성:

```json
{
  "colors": {
    "primary": "#3b82f6",
    "secondary": "#8b5cf6",
    "success": "#10b981",
    "danger": "#ef4444"
  },
  "fonts": {
    "heading": "Inter Bold",
    "body": "Inter Regular"
  },
  "spacing": {
    "sm": "8px",
    "md": "16px",
    "lg": "24px"
  }
}
```

프롬프트에서 참조:

```
You: design-system.json 파일을 참고해서
     랜딩 페이지를 디자인해줘.
```

### Figma/Sketch 파일 가져오기

기존 디자인을 기반으로 변형할 수 있습니다.

```
You: 첨부한 Figma 링크를 참고해서
     비슷한 스타일로 로그인 페이지를 만들어줘.
     
     [Figma 링크: https://www.figma.com/file/...]
```

Stitch가 Figma 파일을 분석하고 유사한 디자인을 생성합니다.

### 반응형 디자인 지정

```
You: 모바일, 태블릿, 데스크톱 각각의 레이아웃을 디자인해줘.
     
     모바일 (< 768px):
     - 1컬럼 레이아웃
     - 햄버거 메뉴
     
     태블릿 (768-1024px):
     - 2컬럼 레이아웃
     - 축소된 사이드바
     
     데스크톱 (> 1024px):
     - 3컬럼 레이아웃
     - 전체 사이드바
```

### 인터랙션 정의

```
You: 버튼에 다음 인터랙션을 추가해줘:
     
     - Hover: 배경색 진하게
     - Active: 살짝 축소 (scale 0.95)
     - Focus: 파란색 외곽선
     - Disabled: 회색 + 투명도 50%
     
     모든 전환은 0.2초 ease-in-out
```

---

## 문제 해결

### 일반적인 문제

**1. "stitch 서버가 연결되지 않습니다"**

증상:
```
✗ stitch (connection failed)
```

해결방법:

단계 1: config.json 확인

```bash
cat ~/.antigravity/config.json
```

올바른 형식인지 확인 (JSON 문법 오류 체크)

단계 2: Stitch MCP 수동 실행

```bash
npx -y davideast/stitch-mcp
```

에러 메시지를 확인하고 문제 파악

단계 3: 인증 재설정

```bash
npx -y davideast/stitch-mcp init
```

처음부터 다시 설정

**2. "인증 실패" 에러**

증상:
```
✗ Authentication failed
```

해결방법:

단계 1: 자격증명 파일 삭제

```bash
rm ~/.config/stitch-mcp/credentials.json
```

단계 2: 재인증

```bash
npx -y davideast/stitch-mcp init
```

새로운 OAuth 플로우 시작

단계 3: 브라우저 확인

인증 시 팝업 차단이 활성화되어 있지 않은지 확인

**3. "API 할당량 초과"**

증상:
```
✗ Quota exceeded
```

해결방법:

Google Cloud Console에서 할당량 확인:
1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 프로젝트 선택
3. "APIs & Services" → "Quotas"
4. Stitch API 할당량 확인

무료 티어 제한:
- 하루 100 requests
- 프로젝트당 10 designs

해결책:
- 내일까지 대기
- 유료 플랜 고려
- 새 프로젝트 생성 (비권장)

**4. "디자인 생성 실패"**

증상:
```
✗ Design generation failed
```

해결방법:

단계 1: 프롬프트 명확히 하기

나쁜 예:
```
대시보드 만들어줘
```

좋은 예:
```
관리자 대시보드를 만들어줘.
왼쪽에 사이드바 (240px), 
상단에 헤더 (64px),
메인 영역에 통계 카드 4개를 2x2 그리드로.
```

단계 2: 복잡도 줄이기

한 번에 너무 많은 것을 요청하지 마세요.
단계별로 나누어 요청하세요.

단계 3: 로그 확인

```bash
tail -f ~/.antigravity/logs/stitch.log
```

에러 메시지 확인

### 프로젝트별 문제

**macOS Catalina 이하**

문제: gcloud 설치 실패

해결:
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
```

수동으로 Google Cloud SDK 설치

**Windows 권한 문제**

문제: PowerShell 실행 정책

해결:
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**Linux: Node.js 버전 낮음**

문제: Node.js < 18

해결:
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

---

## 실전 팁과 베스트 프랙티스

### 프롬프트 작성 팁

**1. 구체적으로 작성하기**

나쁜 예:
```
웹사이트 만들어줘
```

좋은 예:
```
전자상거래 제품 리스트 페이지를 만들어줘.

레이아웃:
- 왼쪽: 필터 사이드바 (카테고리, 가격대, 브랜드)
- 오른쪽: 제품 그리드 (3컬럼, 반응형)

각 제품 카드:
- 정사각형 이미지
- 제품명 (2줄 제한)
- 가격 + 할인율
- 별점
- "장바구니 추가" 버튼
```

**2. 레퍼런스 제공하기**

```
You: Airbnb 스타일의 숙소 카드를 디자인해줘.
     
     참고: https://www.airbnb.com
     
     포함할 요소:
     - 여러 이미지 (스와이프 가능)
     - 위치 + 별점
     - 날짜 + 가격
     - 하트 아이콘 (위시리스트)
```

**3. 우선순위 표시하기**

```
You: 블로그 포스트 카드

필수:
- 제목
- 발행일
- 요약 (3줄)

선택:
- 저자 정보
- 태그
- 읽기 시간

생략 가능:
- 댓글 수
- 공유 버튼
```

### 디자인 반복 프로세스

**1단계: 빠른 프로토타입**

```
간단한 버전으로 시작
핵심 기능만 포함
빠르게 확인
```

**2단계: 피드백**

```
생성된 디자인 검토
부족한 부분 파악
우선순위 재조정
```

**3단계: 점진적 개선**

```
한 번에 하나씩 개선
각 변경사항 확인
필요하면 되돌리기
```

**4단계: 최종 다듬기**

```
세부 사항 조정
일관성 확인
접근성 체크
```

### 성능 최적화

**생성된 코드 최적화:**

단계 1: HTML 구조 확인

불필요한 div 제거
시맨틱 태그 사용 (header, nav, main, footer)

단계 2: CSS 정리

중복 스타일 제거
CSS 변수 활용
미사용 스타일 삭제

단계 3: 이미지 최적화

적절한 이미지 형식 사용 (WebP, AVIF)
lazy loading 추가
responsive images

예시:
```html
<img 
  src="product.webp" 
  alt="제품" 
  loading="lazy"
  srcset="product-small.webp 400w, product-large.webp 800w"
  sizes="(max-width: 600px) 400px, 800px"
>
```

### 팀 협업

**디자인 시스템 공유:**

1. 설정 파일 저장소에 커밋
2. 팀원들과 공유
3. 일관된 디자인 유지

```bash
git add design-system.json
git commit -m "Add design system config"
git push
```

**컴포넌트 라이브러리 구축:**

각 디자인을 재사용 가능한 컴포넌트로:

```
components/
├── Button.html
├── Card.html
├── Header.html
└── Footer.html
```

**문서화:**

README.md에 사용법 기록:

```markdown
# 디자인 가이드

## 버튼 컴포넌트

사용법:
\`\`\`html
<button class="btn btn-primary">클릭</button>
\`\`\`

변형:
- btn-primary (파란색)
- btn-secondary (회색)
- btn-success (초록색)
```

---

## 결론

Stitch MCP × Antigravity 연동으로 디자인부터 코드까지 전체 프로세스가 자동화됩니다.

### 핵심 요약

**시간 절약:**
- 전통적 방식: 4-5주
- Stitch + Antigravity: 1-2시간
- **95% 이상 단축**

**품질 향상:**
- 전문가 수준의 디자인
- 일관된 코드 스타일
- 프로덕션 레디

**접근성:**
- 디자인 스킬 불필요
- 코딩 지식 최소화
- 누구나 사용 가능

### 다음 단계

1. **실습하기** - 간단한 대시보드부터 시작
2. **실험하기** - 다양한 프롬프트 테스트
3. **최적화하기** - 생성된 코드 개선
4. **공유하기** - 팀과 베스트 프랙티스 공유

### 추가 리소스

**공식 문서:**
- [Stitch Documentation](https://stitch.withgoogle.com/docs)
- [Antigravity Guide](https://antigravity.dev/docs)
- [davideast/stitch-mcp GitHub](https://github.com/davideast/stitch-mcp)

**커뮤니티:**
- Stitch Discord
- Antigravity Forum
- r/StitchMCP

**학습 자료:**
- 공식 튜토리얼
- 예제 프로젝트
- 비디오 가이드

이제 여러분의 아이디어를 실현할 준비가 되었습니다. 행운을 빕니다! 🚀

---

**작성일자:** 2026-01-23

