---
title: "Google Stitch MCP × Antigravity 연동 실전 가이드"
date: 2026-01-23 06:10:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Guide,  Google,  google-cloud,  Stitch,  MCP,  Antigravity,  Claude.write]
---


## 목차
1. [개요](#개요)
2. [사전 준비사항](#사전-준비사항)
3. [단계별 설치 가이드](#단계별-설치-가이드)
4. [실전 사용 예제](#실전-사용-예제)
5. [고급 활용법](#고급-활용법)
6. [문제 해결](#문제-해결)
7. [실전 팁과 베스트 프랙티스](#실전-팁과-베스트-프랙티스)

---

**📝 가이드 사용 안내**

이 가이드에서 대문자로 표기된 값들(예: `YOUR_PROJECT_ID`, `YOUR_EMAIL@gmail.com`, `{username}`)은 **placeholder**입니다. 실제 명령어를 실행할 때는 여러분의 실제 값으로 교체해야 합니다.

예시:
- `YOUR_PROJECT_ID` → `stitch-mcp-485109` (실제 프로젝트 ID)
- `YOUR_EMAIL@gmail.com` → `john@gmail.com` (본인의 이메일)
- `/Users/{username}/` → `/Users/john/` (실제 사용자 경로)

---

## 개요

Google Stitch MCP(Model Context Protocol) Server를 Antigravity와 연동하면, 자연어 명령만으로 전문가 수준의 UI 디자인을 생성하고 즉시 HTML/CSS 코드로 변환할 수 있습니다. 이 가이드는 실제 설치부터 프로덕션급 대시보드 생성까지 전 과정을 다룹니다.

### 왜 Stitch MCP + Antigravity인가?

Antigravity는 Google의 AI 코딩 도구로, Gemini 3 Pro와 긴밀하게 통합되어 있습니다. Stitch MCP를 연동하면 다음과 같은 워크플로우가 가능해집니다.

**전통적인 방식**:
1. 디자이너가 Figma에서 디자인 (1-2주)
2. 개발자가 HTML/CSS 코딩 (1-2주)
3. 백엔드 연동 및 테스트 (1주)
4. 총 4-5주 소요

**Stitch MCP + Antigravity 방식**:
1. 자연어로 디자인 요청 (1분)
2. Stitch가 디자인 생성 (1-2분)
3. HTML/CSS 코드 자동 생성 (즉시)
4. Gemini가 백엔드 연동 코드 작성 (10-30분)
5. 총 1-2시간 소요

시간이 95% 이상 단축되는 혁신적인 변화입니다.

## 사전 준비사항

### 필수 요구사항

**1. Google Cloud 계정**
- 무료 계정으로도 충분하지만, 결제 정보 등록이 필요할 수 있습니다
- Stitch API 자체는 현재 무료입니다
- 프로젝트를 생성할 권한이 있어야 합니다

**2. Antigravity 설치**
- Antigravity는 Google의 AI 코딩 도구입니다
- 다운로드: [https://antigravity.dev](https://antigravity.dev) (실제 URL은 Google 공식 문서 참조)
- macOS, Linux, Windows를 지원합니다

**3. gcloud CLI (자동 설치됨)**
- davideast/stitch-mcp가 자동으로 처리합니다
- 수동 설치가 필요한 경우: [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)
- 버전 553.0.0 이상 권장

**4. Node.js 및 npm**
- Node.js 18.x 이상 (최신 LTS 버전 권장)
- npm은 Node.js와 함께 자동 설치됩니다
- 확인: `node --version` 및 `npm --version`

**5. 인터넷 연결**
- 안정적인 인터넷 연결이 필요합니다
- OAuth 인증을 위해 브라우저 접근이 필요합니다
- VPN 사용 시 Google Cloud 접근이 차단되지 않는지 확인하세요

### 권장 사양

- RAM: 8GB 이상
- 저장공간: 5GB 이상 여유 공간
- 운영체제: macOS 12+, Ubuntu 20.04+, Windows 10+

## 단계별 설치 가이드

### 0단계: 공식 문서 확인 (중요!)

설치를 시작하기 전에 반드시 공식 문서를 읽어보세요. API 스펙이 변경될 수 있으므로 최신 정보를 확인하는 것이 중요합니다.

```bash
# 공식 설치 가이드
https://stitch.withgoogle.com/docs/mcp/setup
```

이 가이드는 2026년 1월 22일 기준으로 작성되었으며, 최신 변경사항은 공식 문서를 참조하세요.

### 1단계: Stitch MCP 초기화

터미널(또는 명령 프롬프트)을 열고 다음 명령을 실행합니다.

```bash
npx @_davideast/stitch-mcp init
```

이 명령은 대화형 설정 마법사를 시작합니다. 다음과 같은 단계들이 진행됩니다.

#### 1.1 MCP 클라이언트 선택

```
? Which MCP client are you using?
  Claude Code
❯ Antigravity
  Cursor
  Custom
```

화살표 키로 `Antigravity`를 선택하고 Enter를 누르세요.

#### 1.2 Google Cloud CLI 확인

```
✔ Google Cloud CLI ready (bundled): v553.0.0
  Path: /Users/{username}/google-cloud-sdk/bin/gcloud
```

gcloud CLI가 자동으로 감지되거나 번들된 버전이 사용됩니다. 문제가 없으면 자동으로 다음 단계로 진행됩니다.

#### 1.3 사용자 인증 (gcloud auth login)

```
Authenticate with Google Cloud
CLOUDSDK_CONFIG="~/.stitch-mcp/config" gcloud auth login
(copied to clipboard)

✔ Press Enter when complete
```

명령어가 자동으로 클립보드에 복사됩니다. 다음 단계를 따르세요.

1. 터미널에 명령어를 붙여넣고 실행합니다
2. 브라우저가 자동으로 열립니다
3. Google 계정으로 로그인합니다
4. "Google Cloud SDK가 Google 계정에 액세스하도록 허용하시겠습니까?" 메시지가 나타나면 **허용**을 클릭합니다
5. 인증이 완료되면 브라우저에 "You are now authenticated with the gcloud CLI" 메시지가 표시됩니다
6. 터미널로 돌아가서 Enter를 누릅니다

**예상 출력**:
```
✔ Logged in as: YOUR_EMAIL@gmail.com
```

#### 1.4 Application Default Credentials (ADC) 설정

```
Authorize Application Default Credentials
CLOUDSDK_CONFIG="~/.stitch-mcp/config" gcloud auth application-default login
(copied to clipboard)

✔ Press Enter when complete
```

이전 단계와 유사하게 진행합니다.

1. 클립보드에 복사된 명령어를 터미널에 붙여넣고 실행합니다
2. 브라우저가 다시 열립니다
3. Google 계정으로 로그인합니다 (이미 로그인되어 있을 수 있음)
4. "Google Auth Library가 Google 계정에 액세스하도록 허용하시겠습니까?" 메시지에서 **허용**을 클릭합니다
5. 완료 메시지를 확인하고 터미널로 돌아가 Enter를 누릅니다

**예상 출력**:
```
✔ ADC configured
```

**중요**: ADC는 애플리케이션(이 경우 Stitch MCP Server)이 여러분의 Google Cloud 계정에 접근할 수 있도록 하는 인증 방식입니다. 사용자 인증과는 별도로 필요합니다.

#### 1.5 Google Cloud 프로젝트 선택

```
✔ Select a project:
❯ my-stitch-project (YOUR_PROJECT_ID)
  another-project (project-123456)
  [Create new project]
```

기존 프로젝트를 선택하거나 새 프로젝트를 생성할 수 있습니다.

**새 프로젝트 생성 시**:
```
? Enter project name: my-stitch-project
? Enter project ID: YOUR_PROJECT_ID

Creating project...
✔ Project created: YOUR_PROJECT_ID
```

프로젝트 ID는 전역적으로 고유해야 하므로, 이미 사용 중인 ID는 사용할 수 없습니다. 일반적으로 `stitch-mcp-{random-numbers}` 형식으로 생성됩니다.

**중요**: 이 가이드에서 `YOUR_PROJECT_ID`, `YOUR_EMAIL@gmail.com` 같은 대문자 표기는 실제 값으로 교체해야 하는 placeholder입니다. 예를 들어:
- `YOUR_PROJECT_ID` → 실제로 생성된 프로젝트 ID (예: `stitch-mcp-485109`)
- `YOUR_EMAIL@gmail.com` → 여러분의 실제 이메일 주소
- `{username}` → 여러분의 실제 사용자 이름

#### 1.6 IAM 권한 설정

```
✔ Required IAM role is already configured.
```

필요한 권한(`roles/serviceusage.serviceUsageConsumer`)이 자동으로 확인되고 설정됩니다. 권한이 없는 경우, 다음과 같은 명령이 자동으로 실행됩니다.

```bash
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="user:YOUR_EMAIL@gmail.com" \
  --role="roles/serviceusage.serviceUsageConsumer"
```

#### 1.7 Stitch API 활성화

```
Enabling Stitch API...
✔ Stitch API enabled
```

다음 명령이 자동으로 실행됩니다.

```bash
gcloud beta services mcp enable stitch.googleapis.com --project=YOUR_PROJECT_ID
```

#### 1.8 MCP 설정 파일 생성

```
✔ Configuration generated

Setup Complete!

Your Antigravity MCP config has been saved to:
~/.gemini/antigravity/mcp_config.json

To activate:
1. Restart Antigravity
2. Check "Manage MCP servers" to verify connection
```

설정이 완료되었습니다! 생성된 설정 파일의 내용은 다음과 같습니다.

```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_PROJECT_ID": "YOUR_PROJECT_ID"
      }
    }
  }
}
```

### 2단계: 설치 확인 (Doctor 명령)

설치가 올바르게 완료되었는지 확인하기 위해 진단 도구를 실행합니다.

```bash
npx @_davideast/stitch-mcp doctor
```

**예상 출력**:

```
Stitch Doctor

✔ Installed (system): v553.0.0
  Path: /Users/{username}/google-cloud-sdk/bin/gcloud
✔ Authenticated: YOUR_EMAIL@gmail.com
✔ Present
✔ Set: stich-mcp-485109
✔ Healthy (200)

────────────────────────────────────────────────────────────────

Health Check Summary

✔ Google Cloud CLI: Installed (system): v553.0.0
  Path: /Users/{username}/google-cloud-sdk/bin/gcloud
✔ User Authentication: Authenticated: YOUR_EMAIL@gmail.com
✔ Application Credentials: Present
✔ Active Project: Set: stich-mcp-485109
✔ Stitch API: Healthy (200)

All checks passed!
```

모든 항목에 ✔ 표시가 있어야 합니다. 하나라도 문제가 있으면 [문제 해결](#문제-해결) 섹션을 참조하세요.

**Doctor 명령이 하는 일**:
1. gcloud CLI 설치 여부 및 버전 확인
2. 사용자 인증 상태 확인
3. Application Default Credentials 존재 확인
4. 활성 프로젝트 설정 확인
5. Stitch API가 정상적으로 응답하는지 확인

이 명령은 문제가 발생했을 때 진단 용도로도 사용할 수 있습니다.

### 3단계: Antigravity에서 MCP 서버 연결 확인

Antigravity를 실행(또는 재시작)하고 MCP 서버가 올바르게 연결되었는지 확인합니다.

#### 3.1 Antigravity 재시작

Antigravity를 완전히 종료하고 다시 시작합니다. macOS의 경우:

```bash
# Antigravity 프로세스 종료
killall Antigravity

# Antigravity 재시작
open -a Antigravity
```

Linux의 경우 애플리케이션 메뉴에서 Antigravity를 종료하고 다시 시작하거나, 터미널에서 `antigravity` 명령을 실행합니다.

#### 3.2 MCP 서버 관리 화면 열기

Antigravity 인터페이스에서 다음과 같이 진행합니다.

1. Antigravity 창의 우측 상단 또는 설정 메뉴에서 **"Manage MCP servers"**를 클릭합니다
2. MCP 서버 목록이 표시됩니다

#### 3.3 Stitch 서버 확인

**예상 화면**:

```
Manage MCP servers                          6 / 100 tools    View raw config    Refresh

stitch         6 / 6    stitch    Configure                                    Enabled  [ON]

1. create_project
   Creates a new Stitch project. A project is a container for UI designs and frontend code.

2. get_project
   Retrieves the details of a specific Stitch project using its project name.

3. list_projects
   Lists all Stitch projects accessible to the user.

4. list_screens
   Lists all screens within a given Stitch project.

5. get_screen
   Retrieves the details of a specific screen within a project.

6. generate_screen_from_text
   [텍스트 프롬프트에서 화면 생성]
```

**확인 포인트**:
- `stitch` 서버가 목록에 있어야 합니다
- `6 / 6` 표시: 6개의 도구가 모두 활성화되었음을 의미합니다
- `Enabled` 스위치가 켜져 있어야 합니다 (파란색)
- 각 도구(create_project, get_project, list_projects 등)가 토글 가능한 상태여야 합니다

모든 도구를 확장하면 다음과 같은 6개 도구가 표시됩니다.

**1. create_project**
- 새 Stitch 프로젝트를 생성합니다
- 프로젝트는 UI 디자인과 프론트엔드 코드를 담는 컨테이너입니다

**2. get_project**
- 특정 Stitch 프로젝트의 세부 정보를 가져옵니다
- 프로젝트 이름을 사용하여 조회합니다
- 형식: `projects/{project_id}`

**3. list_projects**
- 사용자가 접근 가능한 모든 Stitch 프로젝트를 나열합니다
- 기본적으로 사용자가 소유한 프로젝트를 나열합니다
- 필터 옵션: `view=owned` (소유한 프로젝트) 또는 `view=shared` (공유된 프로젝트)

**4. list_screens**
- 특정 Stitch 프로젝트 내의 모든 화면을 나열합니다
- 프로젝트 ID가 필요합니다
- 형식: `projects/{project_id}`

**5. get_screen**
- 프로젝트 내 특정 화면의 세부 정보를 가져옵니다
- 프로젝트 ID와 화면 ID가 필요합니다
- 예: `3780309359108792857` (프로젝트 ID), `88805318abe84d16add098fae3add91e` (화면 ID)

**6. generate_screen_from_text**
- 텍스트 프롬프트에서 화면을 생성합니다
- 가장 자주 사용되는 핵심 도구입니다

모든 도구가 정상적으로 표시되고 활성화되어 있다면 설치가 성공적으로 완료된 것입니다!

#### 3.4 설정 파일 직접 확인 (선택사항)

설정이 올바르게 생성되었는지 직접 확인하고 싶다면 다음 명령을 사용하세요.

```bash
cat ~/.gemini/antigravity/mcp_config.json
```

**예상 출력**:
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": [
        "@_davideast/stitch-mcp",
        "proxy"
      ],
      "env": {
        "STITCH_PROJECT_ID": "YOUR_PROJECT_ID"
      }
    }
  }
}
```

## 실전 사용 예제

이제 실제로 Stitch를 사용해서 전문가 수준의 UI를 생성해봅시다. 여기서는 AWS LAMP 스택 서버 모니터링 대시보드를 만드는 실제 사례를 다룹니다.

### 예제 1: 세련된 다크모드 서버 모니터링 대시보드

#### 4.1 Antigravity에서 프롬프트 작성

Antigravity의 채팅 창에 다음과 같이 입력합니다.

```
스티치로 세련된 다크 모드 대시보드 화면 생성해줘
서버는 AWS LAMP 스택이야.
마이크로 인터랙션으로 재미를 강조해줘
```

이것은 간단한 예시이지만, 실전에서는 더 상세한 프롬프트를 사용하는 것이 좋습니다. 다음은 실제로 성공적인 결과를 낳은 상세한 프롬프트입니다.

**상세 프롬프트 예시**:

```
스티치로 세련된 다크 모드 대시보드 화면을 생성해줘.

서버 환경: AWS LAMP 스택 (Linux, Apache, MySQL, PHP)

디자인 요구사항:
- 프리미엄하고 최첨단 느낌의 UI
- 깊은 블랙과 차콜 그레이를 베이스로 사용
- 시안(cyan), 퍼플(purple), 그린(green)의 생동감 넘치는 네온 악센트
- 데이터 시각화를 위한 빛나는 네온 차트

주요 기능:
1. 실시간 서버 메트릭 (CPU 부하, RAM 사용량, Disk I/O) - 빛나는 네온 차트로 표시
2. LAMP 스택 서비스 상태 (Linux, Apache, MySQL, PHP) - 인터랙티브 상태 인디케이터
3. 마이크로 인터랙션: 
   - 호버 시 미묘하게 기울어지는 카드
   - 활성 서비스에 빛나는 펄스 효과
   - 데이터 업데이트 시 부드러운 트랜지션
4. 사이드바와 카드에 글래스모피즘 효과로 깊이감 추가
5. 높은 가독성을 위한 모던 타이포그래피 (Inter 또는 Roboto)

전체적으로 WOW-worthy한 디자인이어야 하며, 전문적인 모니터링과 하이엔드 미적 감각을 모두 강조하면서 재미있는 마이크로 애니메이션을 포함해야 해.
```

#### 4.2 AI의 처리 과정

Antigravity가 프롬프트를 받으면 다음과 같이 처리합니다.

**1단계: 프롬프트 분석**
```
Analyzing request...
- Design type: Dashboard
- Theme: Dark mode
- Stack: AWS LAMP
- Key features: Real-time metrics, service status, micro-interactions
- Visual style: Premium, neon accents, glassmorphism
```

**2단계: MCP Tool 호출**
```
MCP Tool: stitch / generate_screen_from_text

Hide Details
run with these arguments:

json
{
  "deviceType": "DESKTOP",
  "modelId": "GEMINI_3_PRO",
  "projectId": "YOUR_GENERATED_PROJECT_ID",
  "prompt": "A sophisticated dark mode dashboard for an AWS LAMP stack server monitoring system. The UI should feel premium and state-of-the-art, using deep blacks and charcoal grays with vibrant neon accents (cyan, purple, and green) for data visualization. \n\nKey features to include:\n1. Real-time server metrics (CPU load, RAM usage, Disk I/O) using glowing neon charts.\n2. LAMP stack service status (Linux, Apache, MySQL, PHP) with interactive status indicators.\n3. Micro-interactions: Cards that subtly tilt on hover, glowing pulses for active services, and smooth transitions for data updates.\n4. Glassmorphism effect for sidebar and cards to add depth.\n5. Modern typography (Inter or Roboto) for high readability.\n\nThe design should be WOW-worthy, emphasizing both professional monitoring and high-end aesthetic appeal with fun micro-animations."
}
```

**주요 파라미터 설명**:
- `deviceType`: "DESKTOP" - 데스크톱 화면 크기로 생성
- `modelId`: "GEMINI_3_PRO" - 최신 Gemini 3 Pro 모델 사용
- `projectId`: 자동 생성된 프로젝트 ID
- `prompt`: 영어로 번역되고 최적화된 프롬프트

**3단계: Stitch API 호출**

Gemini 3 Pro가 프롬프트를 분석하고 디자인을 생성합니다. 이 과정은 약 30초에서 2분 정도 소요됩니다.

```
Generating design with Gemini 3 Pro...
⏳ Estimated time: 90 seconds
```

**4단계: 결과 반환**

생성이 완료되면 Antigravity에 결과가 표시됩니다.

#### 4.3 생성된 디자인 확인

**미리보기 화면**:

Antigravity에는 다음과 같은 요소가 표시됩니다.

**좌측 패널: 디자인 설명**
```
Cyber-Command Server Status

A sophisticated dark mode dashboard for an AWS LAMP stack server monitoring system. The UI should feel premium and state-of-the-art, using deep blacks and charcoal grays with vibrant neon accents (cyan, purple, and green) for data visualization.

Key features to include:

- Real-time server metrics (CPU load, RAM usage, Disk I/O) using glowing neon charts.
- LAMP stack service status (Linux, Apache, MySQL, PHP) with interactive status indicators.
- Micro-interactions: Cards that subtly tilt on hover, glowing pulses for active services, and smooth transitions for data updates.
- Glassmorphism effect for sidebar and cards to add depth.
- Modern typography (Inter or Roboto) for high readability.

The design should be WOW-worthy, emphasizing both professional monitoring and high-end aesthetic appeal with fun micro-animations.
```

**우측 패널: 생성된 화면 미리보기**

두 가지 버전의 대시보드가 표시됩니다.

**버전 1: AWS LAMP Stack Monitoring Dashboard**
- 왼쪽 사이드바: Dashboard, Instances, Logs, Alerts, Settings 메뉴
- 상단: Instance 정보 (instance I-0x83f2a, IP: 172.31.42.19)
- 메트릭 카드: CPU LOAD (42%), MEMORY (12.4 / 16 GB), DISK I/O (840 IOPS), NET TRAFFIC (1.2 MB/s)
- 중앙: Performance Metrics 차트 (24H, 6H, 24H 탭)
- 우측: Service Health (Ubuntu 22.04, Apache HTTP, MySQL 8.0, PHP-FPM 8.1 - 모두 정상)
- 하단: Console Output 로그

**버전 2: CYBER-COMMAND NOC Dashboard (v2.4)**
- 더욱 세련된 사이버펑크 스타일
- 상단: 로케이션 표시 (us-east-1, N. Virginia), LIVE 배지, 알림 아이콘
- 좌측: Stack Status (Linux, Apache, MySQL, PHP - 모두 초록색 인디케이터)
- Cluster Health: Good
- 메인 섹션:
  - TOTAL REQUESTS: 2.4M (차트)
  - AVG LATENCY: 42ms (체크 아이콘)
  - ERROR RATE: 0.01% (체크 아이콘)
  - SYSTEM HEALTH: 100% (대형 원형 게이지, OPTIMAL PERFORMANCE 표시)
  - ACTIVE THREADS: 124 / 200
- 우측: RESOURCE USAGE
  - vCPU Usage: 0% (Core 1: 52%, Core 2: 38%)
  - Memory (RAM): 481MB / 945MB (프로그레스 바)
  - NVMe Storage: 340GB Free (프로그레스 바)
  - NETWORK I/O: Eth0, 바 차트 (RX: 245 Mbps, TX: 112 Mbps)
- 하단: SYSTEM LOGS (시간순 로그 메시지)

두 버전 모두 다크 모드, 네온 악센트, 실시간 메트릭을 특징으로 하지만, 버전 2가 더 세련되고 사이버펑크 느낌이 강합니다.

#### 4.4 인터랙션 확인

Stitch에서 생성된 디자인에는 다음과 같은 인터랙션이 포함되어 있습니다 (HTML/CSS로 구현됨).

**마이크로 인터랙션**:
1. **카드 호버 효과**: 마우스를 올리면 카드가 미묘하게 위로 떠오르고 그림자가 강해집니다
2. **펄스 애니메이션**: 활성 서비스 인디케이터가 부드럽게 깜빡입니다
3. **차트 애니메이션**: 데이터가 업데이트될 때 부드러운 트랜지션으로 변경됩니다
4. **글로우 효과**: 네온 악센트 요소들이 미묘하게 빛납니다

이 모든 효과는 CSS animation과 transition으로 구현되며, JavaScript가 필요하지 않습니다.

### 5단계: 생성된 HTML 확인 및 다운로드

#### 5.1 HTML 코드 보기

Antigravity에서 생성된 디자인의 HTML 코드를 볼 수 있습니다.

1. 미리보기 화면 하단 또는 옆에 **"View Code"** 또는 **"Show HTML"** 버튼이 있습니다
2. 클릭하면 전체 HTML/CSS 코드가 표시됩니다

**코드 구조**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CYBER-COMMAND NOC DASHBOARD</title>
    <style>
        /* 글로벌 스타일 */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #0a0a0a 0%, #1a1a2e 100%);
            color: #e0e0e0;
            min-height: 100vh;
        }
        
        /* 글래스모피즘 효과 */
        .glass {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 12px;
        }
        
        /* 네온 글로우 효과 */
        .neon-glow {
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }
        
        /* 카드 호버 효과 */
        .card {
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        
        .card:hover {
            transform: translateY(-5px) rotateX(2deg);
            box-shadow: 0 10px 40px rgba(0, 255, 255, 0.3);
        }
        
        /* 더 많은 스타일... */
    </style>
</head>
<body>
    <!-- 대시보드 HTML 구조 -->
    <div class="dashboard">
        <aside class="sidebar glass">
            <!-- 사이드바 내용 -->
        </aside>
        <main class="main-content">
            <!-- 메인 컨텐츠 -->
        </main>
    </div>
</body>
</html>
```

#### 5.2 HTML 다운로드 및 테스트

**방법 1: Antigravity에서 직접 다운로드**

1. "Download HTML" 또는 "Export" 버튼 클릭
2. 파일 이름을 지정하고 저장 (예: `cyber-dashboard.html`)
3. 브라우저에서 파일을 열어 확인

**방법 2: Claude에게 요청**

```
이 HTML 코드를 파일로 저장해줘. 파일명은 cyber-dashboard.html로 해줘.
```

Claude가 자동으로 파일을 생성하고 다운로드 링크를 제공합니다.

**방법 3: 수동 복사**

1. HTML 코드 전체를 선택하고 복사
2. 텍스트 에디터(VS Code, Sublime Text 등)에서 새 파일 생성
3. 코드 붙여넣기
4. `cyber-dashboard.html`로 저장

#### 5.3 로컬에서 테스트

저장한 HTML 파일을 브라우저에서 열어봅니다.

```bash
# macOS
open cyber-dashboard.html

# Linux
xdg-open cyber-dashboard.html

# Windows
start cyber-dashboard.html
```

또는 파일을 더블클릭하여 기본 브라우저에서 엽니다.

**확인 사항**:
- 다크모드 배경이 올바르게 표시되는가?
- 네온 악센트 색상(시안, 퍼플, 그린)이 보이는가?
- 카드에 마우스를 올리면 호버 효과가 작동하는가?
- 활성 서비스 인디케이터가 깜빡이는가?
- 레이아웃이 깨지지 않고 잘 정렬되어 있는가?

모든 것이 정상적으로 작동한다면 성공입니다!

#### 5.4 실제 데이터와 연결

생성된 HTML은 정적인 목업(mockup)입니다. 실제 서버 데이터와 연결하려면 다음 단계가 필요합니다.

**1단계: 백엔드 API 설정**

AWS LAMP 서버에서 실시간 메트릭을 제공하는 API를 만듭니다.

```php
<?php
// /api/metrics.php

header('Content-Type: application/json');
header('Access-Control-Allow-Origin: *');

// CPU 사용률 가져오기
$cpu_load = sys_getloadavg()[0] * 100 / 4; // 4코어 기준

// 메모리 사용량 가져오기
$free = shell_exec('free -b');
preg_match_all('/\d+/', $free, $matches);
$mem_total = $matches[0][7];
$mem_used = $matches[0][8];

// 디스크 I/O (간단한 예시)
$disk_stats = file_get_contents('/proc/diskstats');
// ... 파싱 로직 ...

// 서비스 상태 확인
$services = [
    'apache' => shell_exec('systemctl is-active apache2'),
    'mysql' => shell_exec('systemctl is-active mysql'),
    'php-fpm' => shell_exec('systemctl is-active php8.1-fpm')
];

$response = [
    'cpu_load' => round($cpu_load, 1),
    'memory' => [
        'used' => $mem_used,
        'total' => $mem_total,
        'percentage' => round(($mem_used / $mem_total) * 100, 1)
    ],
    'disk_io' => 840, // 예시 값
    'services' => [
        'apache' => trim($services['apache']) === 'active',
        'mysql' => trim($services['mysql']) === 'active',
        'php-fpm' => trim($services['php-fpm']) === 'active'
    ],
    'timestamp' => time()
];

echo json_encode($response);
```

**2단계: JavaScript로 API 연결**

Claude에게 요청합니다.

```
이 HTML에 JavaScript를 추가해서 /api/metrics.php에서 실시간 데이터를 가져와 
차트와 메트릭을 업데이트하도록 만들어줘. 5초마다 자동으로 갱신되어야 해.
```

Claude가 다음과 같은 코드를 추가합니다.

```javascript
<script>
// API에서 메트릭 가져오기
async function fetchMetrics() {
    try {
        const response = await fetch('/api/metrics.php');
        const data = await response.json();
        
        // CPU 업데이트
        document.querySelector('.cpu-load').textContent = data.cpu_load + '%';
        
        // 메모리 업데이트
        const memUsed = (data.memory.used / (1024**3)).toFixed(1);
        const memTotal = (data.memory.total / (1024**3)).toFixed(1);
        document.querySelector('.memory-value').textContent = `${memUsed} / ${memTotal} GB`;
        document.querySelector('.memory-bar').style.width = data.memory.percentage + '%';
        
        // 서비스 상태 업데이트
        updateServiceStatus('apache', data.services.apache);
        updateServiceStatus('mysql', data.services.mysql);
        updateServiceStatus('php-fpm', data.services['php-fpm']);
        
        // 차트 업데이트 (Chart.js 사용 예시)
        if (window.cpuChart) {
            window.cpuChart.data.datasets[0].data.push(data.cpu_load);
            window.cpuChart.data.labels.push(new Date().toLocaleTimeString());
            // 최근 20개 데이터만 유지
            if (window.cpuChart.data.labels.length > 20) {
                window.cpuChart.data.datasets[0].data.shift();
                window.cpuChart.data.labels.shift();
            }
            window.cpuChart.update();
        }
    } catch (error) {
        console.error('Failed to fetch metrics:', error);
    }
}

function updateServiceStatus(service, isActive) {
    const indicator = document.querySelector(`.service-${service} .status-indicator`);
    if (isActive) {
        indicator.classList.add('active');
        indicator.classList.remove('inactive');
    } else {
        indicator.classList.add('inactive');
        indicator.classList.remove('active');
    }
}

// 초기 로드
fetchMetrics();

// 5초마다 갱신
setInterval(fetchMetrics, 5000);
</script>
```

**3단계: 배포**

완성된 파일을 AWS 서버에 업로드합니다.

```bash
# SCP를 사용한 업로드 예시
scp cyber-dashboard.html user@your-server:/var/www/html/dashboard.html
scp api/metrics.php user@your-server:/var/www/html/api/metrics.php
```

이제 `http://your-server/dashboard.html`에 접속하면 실시간으로 업데이트되는 대시보드를 볼 수 있습니다!

### 예제 2: 이커머스 상품 페이지

다른 사용 사례로, 전자상거래 상품 페이지를 만들어봅시다.

```
스티치로 미니멀한 전자상거래 상품 상세 페이지를 만들어줘.

상품: 프리미엄 무선 헤드폰
색상: 화이트와 라이트 그레이 베이스, 블랙 악센트

레이아웃:
1. 왼쪽: 큰 상품 이미지 (갤러리, 5장)
2. 오른쪽: 상품 정보
   - 제품명과 브랜드
   - 별점 및 리뷰 개수
   - 가격 (원가 취소선, 할인가 강조)
   - 색상 선택 (화이트, 블랙, 실버)
   - 수량 선택
   - "장바구니 담기"와 "바로 구매" 버튼
3. 하단: 탭 메뉴 (상세 설명, 스펙, 리뷰, Q&A)

디자인 스타일:
- 깔끔하고 모던한 미니멀리즘
- 넓은 여백으로 시원한 느낌
- 제품 이미지가 돋보이도록
- 부드러운 그림자와 둥근 모서리
- 호버 시 버튼에 부드러운 스케일 효과
```

Gemini가 프롬프트를 분석하고 세련된 상품 페이지를 생성합니다. 결과물은 다음과 같은 특징을 가집니다.

- 좌측 이미지 갤러리: 메인 이미지와 썸네일 5개
- 우측 정보 패널: 계층적 타이포그래피로 정보 전달
- 색상 선택: 클릭 가능한 색상 원들
- CTA 버튼: "장바구니 담기"(아웃라인), "바로 구매"(솔리드) 구분
- 탭 메뉴: 밑줄 애니메이션으로 활성 탭 표시
- 반응형 디자인: 모바일에서는 세로 레이아웃으로 자동 전환

### 예제 3: SaaS 랜딩 페이지

SaaS 제품의 랜딩 페이지도 빠르게 생성할 수 있습니다.

```
스티치로 AI 코딩 도구의 랜딩 페이지를 만들어줘.

제품: CodeGenius AI - AI 기반 코드 자동 완성 도구
타겟: 개발자

섹션:
1. Hero: 
   - 강력한 헤드라인: "10배 빠른 코딩"
   - 서브헤드: "AI가 당신의 생각을 코드로 변환합니다"
   - CTA: "무료로 시작하기"
   - 우측: 제품 스크린샷 (코드 에디터 화면)

2. Features (3컬럼):
   - 실시간 AI 자동완성
   - 버그 자동 감지 및 수정
   - 모든 언어 지원

3. Demo Video: 
   - 가운데 정렬된 비디오 플레이어
   - 재생 버튼 오버레이

4. Testimonials (캐러셀):
   - 고객 사진, 이름, 직책, 후기

5. Pricing (3티어):
   - Free, Pro, Enterprise
   - 가격, 기능 비교

6. CTA:
   - "오늘 바로 시작하세요"
   - 이메일 입력 + "시작하기" 버튼

7. Footer:
   - 링크, 소셜 미디어, 저작권

디자인:
- 대담하고 현대적
- 그라데이션 (보라색 → 파란색)
- 밝은 배경, 다크 텍스트
- 큰 타이포그래피로 임팩트
- 마이크로 인터랙션 (버튼 호버, 스크롤 애니메이션)
```

결과물은 풀스택 랜딩 페이지로, 모든 섹션이 조화롭게 배치되고 스크롤에 따라 부드럽게 나타나는 애니메이션이 포함됩니다.

## 고급 활용법

기본 사용법을 익혔다면, 이제 더 고급 기능들을 활용해봅시다.

### 디자인 반복 및 개선

첫 번째 생성 결과가 완벽하지 않을 수 있습니다. 다음과 같이 반복적으로 개선할 수 있습니다.

**초기 생성**:
```
스티치로 블로그 포스트 레이아웃을 만들어줘.
```

**결과 확인 후 개선 요청**:
```
좋은데, 다음 사항을 수정해줘:
1. 제목 폰트를 더 크게 (48px)
2. 읽기 시간 추가 (5 min read)
3. 소셜 공유 버튼을 좌측 고정 사이드바로 이동
4. 본문 폰트를 Georgia로 변경 (세리프 폰트가 읽기에 더 좋음)
5. 이미지 캡션 스타일 추가 (이탤릭, 작은 폰트)
```

Antigravity는 기존 디자인을 기반으로 수정사항을 적용합니다.

### 디자인 시스템 추출 및 재사용

여러 화면을 만들 때 일관성을 유지하려면 디자인 시스템을 추출하세요.

```
이 대시보드의 디자인 시스템을 추출해줘. 
색상 팔레트, 타이포그래피, 간격, 컴포넌트 스타일을 모두 포함해서.
```

Claude가 다음과 같은 디자인 토큰을 추출합니다.

```css
/* 색상 팔레트 */
:root {
  --color-bg-primary: #0a0a0a;
  --color-bg-secondary: #1a1a2e;
  --color-text-primary: #e0e0e0;
  --color-text-secondary: #a0a0a0;
  --color-accent-cyan: #00ffff;
  --color-accent-purple: #b19cd9;
  --color-accent-green: #00ff88;
  --color-border: rgba(255, 255, 255, 0.1);
}

/* 타이포그래피 */
:root {
  --font-family-primary: 'Inter', sans-serif;
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.5rem;     /* 24px */
  --font-size-2xl: 2rem;      /* 32px */
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-bold: 700;
}

/* 간격 */
:root {
  --spacing-xs: 0.25rem;   /* 4px */
  --spacing-sm: 0.5rem;    /* 8px */
  --spacing-md: 1rem;      /* 16px */
  --spacing-lg: 1.5rem;    /* 24px */
  --spacing-xl: 2rem;      /* 32px */
  --spacing-2xl: 3rem;     /* 48px */
}

/* 그림자 */
:root {
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 8px rgba(0, 0, 0, 0.2);
  --shadow-lg: 0 10px 40px rgba(0, 255, 255, 0.3);
}

/* 둥근 모서리 */
:root {
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
}
```

이제 새로운 화면을 만들 때 이 디자인 시스템을 참조하도록 요청할 수 있습니다.

```
위에서 추출한 디자인 시스템을 사용해서 설정 페이지를 만들어줘.
동일한 색상, 폰트, 간격을 사용해야 해.
```

### React 컴포넌트로 변환

HTML을 React 컴포넌트로 변환하면 재사용성과 유지보수성이 높아집니다.

```
이 대시보드를 React 컴포넌트로 변환해줘.
- 함수형 컴포넌트 사용
- TypeScript로 작성
- styled-components로 스타일링
- 컴포넌트를 적절히 분리 (Header, Sidebar, MetricCard, Chart 등)
- Props 인터페이스 정의
```

Claude가 다음과 같은 구조로 변환합니다.

```typescript
// types.ts
export interface Metric {
  label: string;
  value: number;
  unit: string;
  trend?: 'up' | 'down';
}

export interface ServiceStatus {
  name: string;
  isActive: boolean;
  uptime: string;
}

// MetricCard.tsx
import styled from 'styled-components';

interface MetricCardProps {
  metric: Metric;
}

const MetricCard: React.FC<MetricCardProps> = ({ metric }) => {
  return (
    <Card>
      <Label>{metric.label}</Label>
      <Value>{metric.value}{metric.unit}</Value>
      {metric.trend && <Trend trend={metric.trend} />}
    </Card>
  );
};

const Card = styled.div`
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1.5rem;
  transition: transform 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
  }
`;

// ... 더 많은 스타일 ...

export default MetricCard;
```

전체 폴더 구조:
```
src/
├── components/
│   ├── Dashboard/
│   │   ├── Dashboard.tsx
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   ├── MetricCard.tsx
│   │   ├── Chart.tsx
│   │   └── ServiceStatus.tsx
│   └── ...
├── types/
│   └── index.ts
├── hooks/
│   └── useMetrics.ts
└── App.tsx
```

### Tailwind CSS로 리팩토링

많은 프로젝트가 Tailwind CSS를 사용합니다. Stitch가 생성한 CSS를 Tailwind 클래스로 변환할 수 있습니다.

```
이 HTML의 CSS를 Tailwind CSS 유틸리티 클래스로 변환해줘.
모든 인라인 스타일과 <style> 태그를 Tailwind 클래스로 대체해야 해.
```

**변환 전**:
```html
<div style="background: rgba(255, 255, 255, 0.05); border-radius: 12px; padding: 1.5rem;">
  <h3 style="font-size: 1.5rem; font-weight: 700; color: #00ffff;">CPU Load</h3>
  <p style="font-size: 2rem; font-weight: 700;">42%</p>
</div>
```

**변환 후**:
```html
<div class="bg-white/5 rounded-xl p-6 backdrop-blur-md border border-white/10">
  <h3 class="text-2xl font-bold text-cyan-400">CPU Load</h3>
  <p class="text-4xl font-bold">42%</p>
</div>
```

훨씬 깔끔하고 유지보수하기 쉬워집니다.

### 접근성 강화

웹 접근성(Accessibility)은 모든 사용자가 콘텐츠를 이용할 수 있도록 하는 것입니다.

```
이 대시보드의 접근성을 WCAG 2.1 AA 수준으로 개선해줘:
1. 모든 대화형 요소에 적절한 ARIA 레이블 추가
2. 키보드 네비게이션 지원 (Tab 순서 최적화)
3. 포커스 스타일 명확하게 표시
4. 색상 대비 비율 4.5:1 이상 확보
5. 스크린 리더를 위한 시맨틱 HTML 사용
6. 이미지에 alt 텍스트 추가
```

Claude가 다음과 같은 개선사항을 적용합니다.

```html
