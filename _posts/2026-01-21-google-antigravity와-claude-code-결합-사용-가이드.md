---
title: "Google Antigravity와 Claude Code 결합 사용 가이드"
date: 2026-01-21 21:30:00 +0900
categories: [AI,  Claude Code & Google Antigravity]
mermaid: [True]
tags: [AI,   claude-code,  Antigravity,  Claude.write]
---


## 서론: 왜 두 도구를 함께 사용하는가?

2025년 11월 18일 Google이 Gemini 3와 함께 Antigravity를 출시하면서 AI 코딩 환경은 새로운 전환점을 맞이했습니다. 동시에 Anthropic의 Claude Code는 2025년 2월 출시 이후 지속적인 업데이트를 통해 터미널 기반 AI 코딩 도구의 표준으로 자리잡았습니다. 이 두 도구는 각각 독특한 강점을 가지고 있으며, 이를 전략적으로 결합하면 개발 생산성을 극대적으로 높일 수 있습니다.

Google Antigravity는 '에이전트 우선(Agent-First)' 플랫폼으로, AI 에이전트가 에디터, 터미널, 브라우저에 직접 접근하여 복잡한 소프트웨어 작업을 자율적으로 계획하고 실행할 수 있습니다. Gemini 3 Pro를 핵심 추론 엔진으로 사용하며, Claude Sonnet 4.5와 GPT-OSS도 지원합니다. 특히 브라우저 통합과 시각화 작업에 강점을 보입니다.

Claude Code는 터미널 기반의 에이전틱 코딩 도구로, 개발자의 워크플로우에 자연스럽게 통합됩니다. 코드베이스 전체를 이해하고, 여러 파일에 걸친 복잡한 편집을 수행하며, 단계별 개발 작업을 실행할 수 있습니다. Claude Opus 4.5 및 Sonnet 4.5 모델을 사용하여 정교한 코드 생성과 디버깅에서 탁월한 성능을 보입니다.

두 도구를 결합하면 각각의 강점을 활용할 수 있습니다. Gemini 3 Pro는 계획 수립과 오케스트레이션을 담당하고, Claude는 실제 코드 실행과 생성을 처리합니다. 이는 마치 프로젝트 매니저와 시니어 개발자가 함께 작업하는 것과 같은 효과를 냅니다.

## 1부: 기본 설정과 도구 이해

### Google Antigravity 설치 및 설정

Google Antigravity는 현재 무료 퍼블릭 프리뷰로 제공되며, Gemini 3 Pro 사용에 대한 '관대한 사용 한도(generous rate limits)'가 적용됩니다. 설치는 간단하지만 몇 가지 시스템 요구사항을 확인해야 합니다.

**시스템 요구사항**
- macOS: Monterey (버전 12) 이상
- Windows: Windows 10 이상
- Linux: 특정 배포판 지원
- 최소 RAM: 8GB (권장: 16GB)
- 개인 Gmail 계정 필요 (현재 프리뷰 단계)

**설치 과정**

공식 다운로드 페이지(antigravity.google/download)를 방문하여 운영체제에 맞는 설치 파일을 다운로드합니다. 제3자 미러 사이트는 피하고 반드시 공식 소스를 사용해야 합니다.

설치 파일을 실행하면 초기 설정 화면이 나타납니다. 여러 질문에 답하면서 진행하는데, 각 단계에서 'Next' 버튼을 클릭합니다. 먼저 설정 흐름을 선택하는 화면이 나타나는데, 기존 VS Code나 Cursor 설정을 가져올 수 있지만 새로 시작하는 것을 권장합니다. 'Start fresh' 옵션을 선택하고 진행합니다.

다음으로 에디터 테마를 선택합니다. 다크 모드나 라이트 모드 중 선호하는 것을 선택하면 되는데, 언제든지 설정에서 변경할 수 있습니다.

가장 중요한 단계는 에이전트 매니저 구성입니다. 화면 왼쪽에 다양한 개발 모드가 표시되는데, 이는 기본적으로 "누가 운전대를 잡을 것인가?"를 묻는 것입니다.

**개발 모드 선택**
- Agent-driven development (에이전트 주도): 완전 자동 조종 모드입니다. AI에게 무엇을 만들지 지시하면 자동으로 코드를 작성하고, 파일을 생성하며, 명령을 실행합니다.
- Review-driven development (검토 주도): AI가 거의 모든 작업 전에 승인을 요청합니다. 가장 안전하지만 느립니다.
- Agent-assisted development (에이전트 보조): 권장 옵션입니다. 사용자가 통제권을 유지하면서 AI가 안전한 자동화를 돕습니다.

오른쪽의 Terminal Policy는 'Auto'로 설정되어 있어 AI가 표준 명령을 자동으로 실행할 수 있으며, 'Agent Decides'는 에이전트가 명령 실행 시점을 결정하게 합니다. 권장 설정인 Agent-assisted development를 선택합니다.

명령줄 도구 설치 옵션이 나타나면 'agy' 명령으로 Antigravity를 열 수 있도록 설정합니다. 이는 터미널에서 빠르게 Antigravity를 실행하는 데 유용합니다.

이제 Google 계정으로 로그인합니다. 개인 Gmail 계정으로 로그인해야 하며, 브라우저가 열려 인증을 진행합니다. 성공적으로 인증되면 다시 Antigravity 애플리케이션으로 돌아옵니다.

마지막으로 이용약관에 동의하고 데이터 수집 옵션을 선택합니다. 설정이 완료되면 Antigravity가 협업할 준비를 마치고 대기합니다.

**인터페이스 이해**

Antigravity의 인터페이스는 Visual Studio Code를 기반으로 하지만 에이전트 관리를 우선시하도록 구조가 변경되었습니다. 다섯 가지 주요 영역으로 구성됩니다.

중앙의 에디터 패널은 가장 큰 영역으로, VS Code 환경이 그대로 유지됩니다. 에이전트가 생성한 코드가 여기에 나타나며, 수동으로 편집할 수도 있습니다. 일반적인 코드 편집 작업은 모두 이 영역에서 이루어집니다.

오른쪽의 에이전트 매니저는 Antigravity의 핵심 혁신입니다. 여기서 여러 AI 에이전트를 배포하고 관리할 수 있습니다. 각 에이전트는 자체 작업 공간을 가지며, 계획 수립, 코드 작성, 테스트를 자율적으로 수행합니다. Plan 모드는 실행 전에 상세한 계획을 생성하며 복잡한 작업에 이상적입니다. Fast 모드는 지시사항을 즉시 실행하여 빠른 수정에 적합합니다.

에이전트 매니저 하단에는 작업 히스토리가 표시됩니다. 활성 및 과거 에이전트 작업들이 설명, 계획, 진행 상황, 상태 표시와 함께 나타납니다.

아티팩트 섹션은 에이전트가 생성한 로그, 패치, 추론 단계, 중간 출력물을 저장합니다. 에이전트가 어떻게 문제를 해결했는지 이해하는 데 유용합니다.

브라우저 패널은 라이브 앱 미리보기를 제공합니다. 에이전트가 자동 테스트를 위해 브라우저와 상호작용할 수 있으며, 수동으로도 사용할 수 있습니다. 이는 웹 개발에서 특히 강력한 기능입니다.

하단의 터미널은 패키지 설치나 명령 실행에 사용됩니다. 에이전트가 자동으로 사용할 수도 있고 수동으로 입력할 수도 있습니다.

### Claude Code 설치 및 설정

Claude Code는 터미널 기반 도구로, Anthropic의 강력한 모델들과 직접 통신합니다. 설치는 명령줄을 통해 이루어지며, Pro, Max, Team, Enterprise 플랜 또는 Console 계정이 필요합니다.

**설치 방법**

macOS나 Linux에서는 터미널을 열고 다음 명령을 실행합니다:
```bash
curl -fsSL https://claude.ai/install.sh | sh
```

Windows에서는 PowerShell을 관리자 권한으로 열고 공식 웹사이트에서 제공하는 설치 명령을 복사하여 실행합니다. PowerShell이 자동으로 Claude Code를 시스템에 다운로드합니다.

설치가 완료되면 터미널에서 'claude'를 입력하고 엔터를 눌러 실행합니다. 처음 실행하면 로그인을 요청하는데, Claude 계정 자격증명으로 로그인합니다.

**주요 기능 이해**

Claude Code는 단순한 코드 완성 도구가 아닙니다. 전체 저장소를 이해하고, 여러 파일에 걸친 강력한 편집을 수행하며, 에이전틱 검색을 통해 전체 코드베이스를 파악합니다. 수동으로 컨텍스트를 선택할 필요가 없습니다.

체크포인트 기능은 각 변경 전에 코드 상태를 자동으로 저장합니다. 문제가 발생하면 Esc를 두 번 누르거나 `/rewind` 명령으로 즉시 되돌릴 수 있습니다.

서브에이전트 기능을 통해 특수 작업을 별도 에이전트에게 위임할 수 있습니다. 한 에이전트는 백엔드를 만들고 다른 에이전트는 프론트엔드를 동시에 작업할 수 있습니다.

Skills는 재사용 가능한 워크플로우입니다. 프로젝트 초기화, 테스트 생성, 배포 스크립트와 같은 반복적인 작업을 자동화할 수 있습니다.

Hooks는 에이전트, Skills, 슬래시 명령에 대한 세밀한 제어를 제공합니다. PreToolUse, PostToolUse, Stop 로직을 범위 지정하여 상태 관리, 도구 제약, 감사 로깅을 처리할 수 있습니다.

MCP(Model Context Protocol) 통합은 브라우저 자동화, API 접근 등 추가 기능을 위해 외부 서버에 연결할 수 있게 합니다.

**필수 명령어**

Claude Code를 효과적으로 사용하기 위한 핵심 명령어들이 있습니다.

`/model` 명령은 사용 중인 모델을 확인하거나 변경합니다. Opus 4.5, Sonnet 4.5, Haiku 4.5 중에서 선택할 수 있습니다.

`/rewind` 명령은 이전 체크포인트로 되돌립니다. 실수를 빠르게 복구할 수 있습니다.

`/task` 명령은 서브에이전트를 시작합니다. 예를 들어 "/task 사용자 인증 시스템 구현"과 같이 사용합니다.

`/skills` 명령은 사용 가능한 Skills를 나열합니다. Skills는 ~/.claude/skills 디렉토리에 저장됩니다.

`/teleport`와 `/remote-env` 명령은 세션 텔레포테이션을 가능하게 합니다. claude.ai/code에서 원격 세션을 재개하고 구성할 수 있어 장치 간 작업 전환이나 협업자와의 세션 공유에 이상적입니다.

Ctrl+B는 에이전트와 셸 명령을 동시에 백그라운드로 보냅니다. 단일 키 입력으로 장기 실행 작업을 백그라운드로 보내 진행 상황을 잃지 않고 다른 작업을 수행할 수 있습니다.

## 2부: 두 도구를 결합하는 실전 워크플로우

### 방법 1: Antigravity 터미널에서 Claude Code 실행

가장 직접적인 통합 방법은 Antigravity의 통합 터미널에서 Claude Code를 실행하는 것입니다. 이 방법은 두 도구의 강점을 하나의 워크스페이스에서 활용할 수 있게 합니다.

**설정 단계**

Antigravity를 실행하고 프로젝트 폴더를 엽니다. 예를 들어 'my-hybrid-project'라는 새 폴더를 생성합니다. 하단의 터미널 패널에서 프로젝트 폴더로 이동한 후 'claude' 명령을 입력하여 Claude Code를 시작합니다.

처음 실행 시 Claude Code가 이 폴더에서 작업할 것인지 묻습니다. 'yes'를 클릭하면 Claude Code가 Antigravity 내부에서 로드됩니다. 상태 확인을 통해 버전 세부정보, 세션 ID, 사용 중인 모델(일반적으로 Claude Sonnet 4.5 또는 Opus 4.5)을 확인할 수 있습니다.

이제 Antigravity와 Claude Code가 동일한 프로젝트에서 동시에 실행되고 있습니다. Antigravity의 레이아웃은 VS Code를 기반으로 하기 때문에 터미널 경험과 컨트롤이 동일하게 느껴집니다.

**역할 분담 전략**

효과적인 협업을 위해서는 각 도구의 강점에 맞게 작업을 분배해야 합니다.

**Google Antigravity(Gemini 3 Pro)를 사용할 작업**
- 프로젝트 계획 및 아키텍처 설계
- 백엔드 API 개발 및 서버 로직
- 데이터베이스 스키마 설계 및 통합
- 복잡한 비즈니스 로직 구현
- 브라우저 기반 테스트 및 UI 검증
- 이미지 생성 및 시각화 (Nano Banana Pro 사용)

**Claude Code(Sonnet 4.5 또는 Opus 4.5)를 사용할 작업**
- 프론트엔드 UI 컴포넌트 개발
- 정교한 코드 리팩토링
- 버그 수정 및 디버깅
- 테스트 코드 작성
- Git 워크플로우 관리 (커밋, PR 생성)
- 다중 파일 편집 작업

**실전 예제: 도서 추천 앱 구축**

구체적인 예를 통해 협업 프로세스를 살펴보겠습니다. Gemini API를 사용하는 도서 추천 앱을 만들어봅니다.

먼저 프로젝트 구조를 설정합니다. Antigravity에서 메인 프로젝트 폴더 'book-recommendation-app'을 생성합니다. 그 안에 'backend'와 'frontend' 두 개의 하위 폴더를 만듭니다.

**백엔드 개발 (Antigravity/Gemini 사용)**

Antigravity의 에이전트 매니저에서 새 대화를 시작합니다. Gemini 3 Pro 모델을 선택하고 Plan 모드로 설정합니다. 복잡한 작업에는 계획이 먼저 필요하기 때문입니다.

프롬프트를 작성합니다: "backend 폴더를 사용하여 Gemini를 활용한 도서 추천 Node.js Express API를 구축하세요. POST /api/recommendations 엔드포인트 하나를 만들어 사용자 선호도를 입력받고 세 권의 책 추천을 JSON으로 반환하세요. CORS를 활성화하고 포트 3000에서 실행하세요."

Gemini는 계획 아티팩트를 생성합니다. 프로젝트 설정, Express 서버 생성, Gemini API 통합, 추천 엔드포인트 추가, CORS 활성화 등의 작업 목록이 체크리스트로 표시됩니다. 계획을 검토하고 승인하면 Gemini가 실행을 시작합니다.

체크리스트가 실시간으로 업데이트됩니다. 각 파일(index.js, package.json, .env)이 생성되고, Node.js 모듈이 설치됩니다. 각 단계를 확인하고 수락하면서 진행합니다. 몇 분 내에 백엔드 구현이 완료되며, Express 서버가 구성되고, Gemini API 통합이 준비되고, 적절한 오류 처리와 함께 엔드포인트가 구현됩니다.

**프론트엔드 개발 (Claude Code 사용)**

Antigravity 터미널에서 Claude Code가 이미 실행 중입니다. Claude Code에 프롬프트를 전달합니다: "frontend 폴더를 사용하여 HTML, 바닐라 JavaScript, CSS로 간단한 도서 추천 페이지를 만드세요. 사용자 선호도를 입력할 텍스트 필드, 추천 받기 버튼, 제목/저자/이유를 보여주는 세 권의 책 표시 영역을 포함한 하나의 HTML 파일을 만드세요. 버튼 클릭 시 http://localhost:3000/api/recommendations로 POST 요청을 보내고 결과를 표시하세요."

Claude Code는 즉시 작업을 시작합니다. 터미널에서 프로세스를 명확하게 볼 수 있습니다. HTML 구조를 구축하고, JavaScript 로직을 추가하고, CSS로 스타일링합니다. 몇 분 내에 index.html 파일이 완성됩니다. 모든 것이 하나의 파일에 깔끔하게 포함되어 있습니다.

**테스트 및 디버깅**

이제 양쪽 서버를 시작할 시간입니다. Claude Code에 "프론트엔드 테스트를 위해 라이브 서버를 실행할 수 있나요?"라고 요청합니다. Claude Code가 즉시 명령을 준비하고 실행합니다. 포트 충돌이 발생하면 자동으로 감지하고 수정합니다. 라이브 서버가 열리고 앱 미리보기가 로드됩니다.

Antigravity에서 "백엔드 서버를 실행하여 테스트할 수 있나요?"라고 요청합니다. Antigravity는 서버를 시작하기 전에 백엔드 파일을 분석합니다. 종속성을 확인하고 구성을 검증한 다음 Express 서버를 실행합니다. 초기 테스트에서 API 키 문제가 발견될 수 있습니다. .env 파일의 Gemini API 키를 실제 키로 업데이트해야 합니다.

Google AI Studio에서 API 키를 생성하고 .env 파일을 업데이트한 후 다시 테스트를 요청합니다. Antigravity가 여러 Gemini 모델(Gemini Pro, Gemini 2.0 Flash)을 자동으로 테스트하여 사용 가능한 모델을 찾습니다. 몇 번의 시도 후 성공적으로 연결됩니다.

앱 미리보기에서 "AI engineering"을 입력 필드에 입력하고 "Get Recommendations" 버튼을 클릭합니다. 초기에 fetch 요청이 실패할 수 있습니다. 이때 Claude Code의 디버깅 강점이 드러납니다.

브라우저 콘솔의 오류 스크린샷을 찍어 Claude Code에 드래그하고 "이 함수에서 fetch가 실패합니다. 백엔드 문제인가요, 프론트엔드 문제인가요?"라고 메시지를 보냅니다. Claude Code가 스크린샷을 즉시 분석하여 오류에 대한 상세한 분석을 제공하고 fetch 로직의 정확한 문제를 식별합니다.

"이것을 수정하고 업데이트하세요"라고 지시하면 Claude Code가 몇 초 만에 파일을 업데이트합니다. 문제, 원인, 수정 방법을 설명하는 명확한 메시지를 제공합니다. 몇 번의 반복 후 완벽하게 수정되고 백엔드 서버도 재시작됩니다.

이제 Claude Code가 두 서버(프론트엔드와 백엔드)를 모두 관리합니다. 에디터 하단에서 두 개의 백그라운드 작업 메시지를 볼 수 있습니다. 앱 미리보기로 돌아가 다시 테스트하면 세 권의 책 추천이 성공적으로 나타납니다.

### 방법 2: antigravity-claude-proxy를 통한 무료 통합

더 고급 사용자를 위한 방법으로, antigravity-claude-proxy라는 오픈소스 프로젝트를 사용하면 Antigravity의 무료 모델 액세스를 Claude Code에서 활용할 수 있습니다. 이는 별도의 Claude Code 플랜 비용 없이 강력한 모델을 사용할 수 있게 합니다.

**antigravity-claude-proxy란?**

Badri Narayanan S가 개발한 이 경량 프록시 서버는 Google Antigravity의 Cloud Code가 제공하는 AI 모델을 Anthropic 호환 API로 노출합니다. Antigravity의 Claude와 Gemini 모델(thinking 모드 포함)을 Claude Code CLI 같은 도구에서 사용할 수 있게 합니다. Antigravity 퍼블릭 프리뷰의 일부로 기술적으로 무료입니다.

작동 방식은 다음과 같습니다. Claude Code 요청을 받아 Anthropic 메시징 API 형식으로 수신합니다. 이를 Google Generative AI API 형식으로 변환합니다. OAuth 토큰으로 Antigravity의 Cloud Code를 통해 전송합니다. 응답을 스트림 지원과 함께 다시 Anthropic 형식으로 변환합니다. API 불일치를 직접 처리할 필요 없이 종단 간 Claude 또는 Gemini 결과를 얻습니다.

**설치 및 설정**

먼저 프록시를 설치합니다. 터미널에서 다음 명령을 실행합니다:
```bash
npm install -g antigravity-claude-proxy
```

설치가 완료되면 Google 계정으로 로그인해야 합니다. Antigravity IDE 내에서 Google 계정에 로그인되어 있는지 확인하거나, 다음 명령으로 OAuth를 통해 하나 이상의 Google 계정을 추가할 수 있습니다:
```bash
antigravity-claude-proxy auth add
```

이 명령은 Google OAuth를 위한 브라우저 탭을 엽니다. 로그인하고 액세스를 승인합니다. 여러 계정에 대해 반복할 수 있습니다.

계정을 관리하는 유용한 명령들:
```bash
# 모든 계정 나열
antigravity-claude-proxy accounts list

# 계정 작동 확인
antigravity-claude-proxy accounts verify

# 대화형 계정 관리
antigravity-claude-proxy accounts
```

이 방법은 Google One 구독이 없는 사용자에게 유용합니다. 일일 할당량에 도달하면 다른 계정으로 전환할 수 있습니다.

다음으로 프록시를 시작합니다:
```bash
antigravity-claude-proxy start
```

프록시가 실행되면 Claude Code가 localhost의 특정 포트를 통해 요청을 보낼 수 있습니다. 프록시는 이를 Antigravity로 라우팅하고 응답을 다시 전달합니다.

**Claude Code에서 사용**

Claude Code에서 `/model` 명령을 입력하여 모델 관리 화면을 엽니다. 여기서 다양한 Claude 모델 간에 전환할 수 있습니다. 프록시가 올바르게 설정되었다면 Antigravity의 무료 할당량을 사용하여 이러한 모델들에 접근할 수 있습니다.

이 방법의 장점은 명확합니다. Claude Code의 워크플로우 깊이를 유지하면서 Antigravity의 무료 모델 액세스를 활용할 수 있습니다. 실험, 사이드 프로젝트, 또는 비용 부담 없이 AI 지원 코딩을 탐색하려는 개발자에게 이상적입니다.

### 방법 3: Skills 공유를 통한 워크플로우 통합

2026년 1월, Google Antigravity 버전 1.14.2에서 획기적인 업데이트가 이루어졌습니다. Antigravity가 Claude Code Skills와 호환되는 에이전트 Skills를 추가한 것입니다. 이는 AI 도구 간 워크플로우 재사용성을 실질적으로 가능하게 만들었습니다.

**Skills란 무엇인가?**

Skills는 재사용 가능한 자동화 워크플로우입니다. 프로젝트 초기화, 코드 생성, 문서 작성, 배포와 같은 반복적인 작업을 자동화합니다. Claude Code에서 Skills는 ~/.claude/skills 디렉토리에 저장되며, Antigravity는 동일한 표준을 채택했습니다.

Anthropic이 도입한 개방형 표준을 Google이 채택했다는 것은 중요한 의미를 갖습니다. Claude Code를 위해 작성한 Skill을 수정 없이 Antigravity에서 사용할 수 있습니다. Skill을 한 번 작성하면 어디서나 사용할 수 있다는 뜻입니다.

**Skills 작성 및 공유**

Skills는 마크다운 파일로 작성되며 YAML frontmatter와 실행 지시사항을 포함합니다. 기본 구조는 다음과 같습니다:

```markdown
---
name: deploy-to-production
description: Deploy application to production environment
user-invocable: true
context: fork
---

# Deploy to Production

This skill handles the complete deployment process.

## Steps
1. Run tests to ensure code quality
2. Build the production bundle
3. Deploy to hosting service
4. Verify deployment
5. Update deployment log

## Instructions for AI
Execute the following steps in order. Ask for confirmation before deploying.
[구체적인 명령어와 검증 단계]
```

frontmatter의 `user-invocable: true`는 이 Skill을 슬래시 명령 메뉴에서 볼 수 있게 합니다. `context: fork`는 격리된 컨텍스트에서 실행되어 메인 에이전트 상태를 오염시키지 않습니다.

**Antigravity에서 Claude Code Skills 사용**

Claude Code에서 이미 유용한 Skills를 작성했다면 Antigravity에서도 바로 사용할 수 있습니다. ~/.claude/skills 디렉토리의 Skills는 Antigravity가 자동으로 인식합니다.

Antigravity에서 Skills 메뉴를 열면(슬래시 명령 또는 에이전트 매니저에서) Claude Code Skills가 나타납니다. 예를 들어 테스트 자동화를 위한 'run-e2e-tests' Skill이나 문서 생성을 위한 'generate-api-docs' Skill 등을 공유할 수 있습니다.

핫 리로드 기능 덕분에 ~/.claude/skills 또는 .claude/skills에 새 Skill이나 업데이트된 Skill을 추가하면 세션을 재시작할 필요 없이 즉시 사용할 수 있습니다. 이는 반복 작업이 매우 빠르게 이루어진다는 의미입니다.

**실용적인 Skills 예제**

프론트엔드 개발자라면 다음과 같은 Skills가 유용할 것입니다:

'create-react-component' Skill은 React 컴포넌트 보일러플레이트를 생성하고, TypeScript 타입을 추가하며, 테스트 파일을 만들고, Storybook 스토리를 생성합니다.

'optimize-images' Skill은 프로젝트의 모든 이미지를 찾아 최적화하고, WebP 버전을 생성하며, 지연 로딩 속성을 추가합니다.

백엔드 개발자에게는 다음 Skills가 도움이 됩니다:

'setup-api-endpoint' Skill은 Express 라우터를 생성하고, 유효성 검사 미들웨어를 추가하며, 컨트롤러 로직을 작성하고, API 문서를 업데이트합니다.

'database-migration' Skill은 데이터베이스 마이그레이션을 생성하고, 롤백 스크립트를 포함하며, 타임스탬프를 추가하고, 마이그레이션 로그를 업데이트합니다.

두 도구 모두에서 사용할 수 있는 범용 Skills도 있습니다:

'git-commit-smart' Skill은 변경사항을 분석하고, 의미 있는 커밋 메시지를 생성하며, 적절한 파일을 스테이징하고, 커밋을 생성합니다.

'refactor-for-performance' Skill은 성능 병목 지점을 식별하고, 최적화를 제안하며, 변경사항을 적용하고, 벤치마크를 실행하여 개선을 검증합니다.

## 3부: 고급 패턴과 최적화

### 하이브리드 워크플로우 패턴

두 도구를 효과적으로 결합하면 단순한 병렬 작업을 넘어서는 시너지 효과를 얻을 수 있습니다. 실전에서 검증된 몇 가지 패턴을 소개합니다.

**계획-실행-검증 사이클**

가장 효과적인 패턴은 Gemini의 계획 능력과 Claude의 실행 정확성, Antigravity의 브라우저 검증 기능을 결합하는 것입니다.

먼저 Antigravity에서 Gemini 3 Pro를 Plan 모드로 사용하여 기능을 계획합니다. "사용자 인증 시스템을 OAuth 2.0과 JWT를 사용하여 구현하는 계획을 세워주세요. 보안 모범 사례를 포함하세요"와 같이 요청합니다. Gemini가 상세한 계획을 생성하면 검토하고 승인합니다.

다음으로 Claude Code에 계획을 전달하고 실행을 요청합니다. "다음 계획에 따라 OAuth 인증을 구현하세요: [계획 붙여넣기]" Claude는 코드를 작성하고 필요한 패키지를 설치하며 설정 파일을 생성합니다.

마지막으로 Antigravity의 브라우저 패널을 사용하여 구현을 테스트합니다. 에이전트가 실제 브라우저에서 로그인 플로우를 자동으로 테스트하고 결과를 보고합니다.

이 사이클은 각 도구의 강점을 활용합니다. Gemini의 고수준 추론, Claude의 정확한 코드 생성, Antigravity의 자동화된 UI 테스트가 결합됩니다.

**병렬 개발 전략**

복잡한 기능의 경우 여러 에이전트를 동시에 실행하여 개발 시간을 대폭 단축할 수 있습니다.

Antigravity에서 백엔드 에이전트를 시작합니다. "API 엔드포인트와 데이터베이스 스키마를 구현하세요" 에이전트가 백그라운드에서 작업하도록 설정합니다.

동시에 Claude Code에서 프론트엔드 작업을 시작합니다. "로그인 페이지와 대시보드 컴포넌트를 만드세요" Claude Code는 별도 터미널에서 실행되므로 간섭이 없습니다.

필요하다면 Claude Code에서 서브에이전트도 시작할 수 있습니다. "/task 테스트 스위트 작성"을 실행하면 세 번째 에이전트가 테스트 코드를 작성합니다.

이제 세 개의 에이전트가 동시에 작업합니다. 백엔드 API(Gemini), 프론트엔드 UI(Claude), 테스트 코드(Claude 서브에이전트). 각각 자신의 영역에서 독립적으로 작업하면서 전체 프로젝트를 빠르게 완성합니다.

**컨텍스트 관리 최적화**

두 도구 모두 CLAUDE.md 파일을 지원하여 프로젝트별 컨텍스트를 제공합니다. 이를 효과적으로 활용하면 에이전트 성능이 크게 향상됩니다.

프로젝트 루트에 CLAUDE.md 파일을 생성합니다:

```markdown
# Project Context

## Tech Stack
- Frontend: React 18 with TypeScript
- Backend: Node.js with Express
- Database: PostgreSQL
- Auth: OAuth 2.0 with JWT

## Coding Standards
- Use functional components with hooks
- Implement proper error handling
- Follow ESLint configuration
- Write unit tests for all utilities

## File Organization
- Components in /src/components
- API routes in /api/routes
- Database models in /models
- Tests alongside source files with .test.ts extension
```

이 파일은 300줄 이하로 유지하고 보편적으로 적용 가능한 정보에 집중해야 합니다. 두 도구 모두 이 파일을 읽고 컨텍스트로 사용하여 더 일관된 코드를 생성합니다.

### 비용 최적화 전략

AI 도구를 집약적으로 사용하면 비용이 빠르게 증가할 수 있습니다. 효과적인 최적화 전략이 필요합니다.

**무료 할당량 최대 활용**

Antigravity는 현재 퍼블릭 프리뷰로 Gemini 3 Pro에 대한 관대한 무료 할당량을 제공합니다. 5시간마다 새로고침되는 사용 제한이 있습니다. 이를 전략적으로 활용합니다.

계획 수립과 아키텍처 결정에는 Antigravity의 무료 Gemini 3 Pro를 사용합니다. 실제 코드 생성과 디버깅에는 Claude Code를 사용합니다. antigravity-claude-proxy를 설정했다면 Claude Code에서도 Antigravity의 무료 할당량을 활용할 수 있습니다.

Google One 구독자는 Claude 모델에 대해 더 관대한 사용 제한을 받습니다. 학생들은 무료 Google One 구독을 신청할 수 있으며, 인도의 Jio 사용자는 무료 Google AI Pro 구독을 받습니다.

**모델 선택 최적화**

모든 작업에 가장 강력한 모델이 필요한 것은 아닙니다. 작업의 복잡도에 따라 모델을 선택합니다.

간단한 코드 완성이나 수정에는 Claude Haiku 4.5나 Gemini Flash를 사용합니다. 이들은 빠르고 비용 효율적입니다.

복잡한 리팩토링이나 아키텍처 결정에는 Claude Sonnet 4.5나 Gemini 3 Pro를 사용합니다. 중간 수준의 작업에 적합합니다.

매우 복잡한 다단계 작업이나 전체 기능 구현에만 Claude Opus 4.5를 사용합니다. 가장 비싸지만 가장 강력합니다.

Claude Code에서 `/model` 명령으로 모델을 쉽게 전환할 수 있습니다. Antigravity에서도 에이전트 매니저에서 모델을 선택할 수 있습니다.

**배치 작업과 백그라운드 실행**

관련 작업을 배치로 묶어 처리하면 컨텍스트 전환을 줄이고 효율성을 높일 수 있습니다. "다음 세 가지 버그를 수정하세요: 1) 로그인 타임아웃, 2) 이미지 로딩 오류, 3) 폼 검증 문제"와 같이 요청합니다.

백그라운드 작업을 활용하여 대기 시간을 줄입니다. Claude Code에서 Ctrl+B로 작업을 백그라운드로 보내고 다른 작업을 계속할 수 있습니다. Antigravity의 에이전트도 백그라운드에서 실행되므로 여러 작업을 동시에 진행할 수 있습니다.

### 디버깅 및 문제 해결

두 도구를 함께 사용할 때 발생할 수 있는 일반적인 문제와 해결 방법입니다.

**포트 충돌**

Antigravity와 Claude Code가 동시에 개발 서버를 실행하면 포트 충돌이 발생할 수 있습니다. Claude Code는 일반적으로 이를 자동으로 감지하고 다른 포트로 전환하지만, 수동으로 포트를 지정해야 할 때도 있습니다.

"포트 3001에서 개발 서버를 시작하세요"와 같이 명시적으로 포트를 지정합니다. 또는 각 서비스에 대해 고정 포트 설정을 프로젝트 설정에 추가합니다.

**컨텍스트 불일치**

두 도구가 다른 버전의 코드를 보고 있으면 혼란이 발생할 수 있습니다. 정기적으로 Git 커밋을 하여 두 도구가 동일한 코드 상태를 공유하도록 합니다.

각 주요 단계 후에 커밋합니다. "이 변경사항을 커밋하고 다음 단계로 진행하세요"라고 요청합니다. Claude Code는 Git 통합이 뛰어나므로 커밋과 푸시를 자연스럽게 처리합니다.

**API 키 및 환경 변수**

두 도구가 같은 .env 파일을 공유하지만 때로는 환경 변수를 다르게 읽을 수 있습니다. .env 파일이 프로젝트 루트에 있고 올바른 형식인지 확인합니다.

민감한 API 키는 .env에만 저장하고 절대 Git에 커밋하지 않습니다. .gitignore 파일에 .env가 포함되어 있는지 확인합니다.

**성능 저하**

여러 에이전트가 동시에 실행되면 시스템 리소스가 부족할 수 있습니다. 최소 16GB RAM을 권장하며, 복잡한 프로젝트의 경우 32GB가 이상적입니다.

필요 없는 백그라운드 작업을 종료합니다. Claude Code에서 `/tasks` 명령으로 활성 작업을 확인하고 필요 없는 것을 중지합니다. Antigravity의 에이전트 매니저에서도 진행 중인 에이전트를 검토하고 관리합니다.

## 4부: 실전 프로젝트 템플릿

### 풀스택 웹 애플리케이션

완전한 풀스택 애플리케이션을 처음부터 구축하는 과정을 단계별로 살펴보겠습니다.

**단계 1: 프로젝트 초기화 (Antigravity)**

Antigravity를 열고 새 프로젝트 폴더를 생성합니다. 에이전트 매니저에서 Gemini 3 Pro를 Plan 모드로 선택합니다.

프롬프트: "프로젝트 이름은 TaskMaster입니다. React + TypeScript 프론트엔드, Node.js + Express 백엔드, PostgreSQL 데이터베이스를 사용하는 작업 관리 애플리케이션의 프로젝트 구조를 계획하세요. 사용자 인증, 작업 CRUD, 태그 기능을 포함합니다."

Gemini가 상세한 폴더 구조, 기술 스택 선택, 데이터베이스 스키마, API 엔드포인트 목록을 포함한 계획을 생성합니다. 계획을 검토하고 승인합니다.

**단계 2: 백엔드 개발 (Antigravity)**

Gemini에게 백엔드 구현을 시작하도록 요청합니다: "승인된 계획에 따라 백엔드 API를 구현하세요. Express 서버, PostgreSQL 연결, 인증 미들웨어, 작업 관련 엔드포인트를 생성하세요."

Antigravity가 파일을 생성하고, 패키지를 설치하며, 서버 로직을 구현합니다. 데이터베이스 마이그레이션도 자동으로 생성됩니다.

**단계 3: 프론트엔드 개발 (Claude Code)**

터미널에서 Claude Code를 시작합니다. 프롬프트: "React + TypeScript로 TaskMaster 프론트엔드를 구현하세요. 로그인 페이지, 대시보드, 작업 목록, 작업 상세 페이지를 만드세요. Tailwind CSS로 스타일링하고 백엔드 API와 통합하세요."

Claude Code가 React 컴포넌트를 생성하고, 라우팅을 설정하며, API 호출을 구현합니다. TypeScript 타입도 자동으로 정의됩니다.

**단계 4: 통합 테스트 (Antigravity + Claude Code)**

Antigravity의 브라우저 패널을 사용하여 전체 애플리케이션을 테스트합니다. 에이전트가 회원가입, 로그인, 작업 생성, 작업 수정, 작업 삭제 플로우를 자동으로 테스트합니다.

동시에 Claude Code에서 서브에이전트를 시작하여 단위 테스트와 통합 테스트를 작성하도록 합니다. "/task Jest로 API 엔드포인트 테스트 작성"

**단계 5: 배포 준비 (Claude Code)**

Claude Code에 배포 설정을 요청합니다: "Docker 컨테이너화를 설정하고, CI/CD 파이프라인을 위한 GitHub Actions 워크플로우를 생성하세요. 환경 변수 관리와 프로덕션 빌드 설정을 포함하세요."

Claude Code가 Dockerfile, docker-compose.yml, GitHub Actions 워크플로우를 생성합니다. 배포 문서도 자동으로 작성됩니다.

### REST API 서비스

백엔드 API에 집중하는 프로젝트의 경우 다른 전략이 효과적입니다.

**단계 1: API 설계 (Antigravity)**

Gemini 3 Pro Deep Think 모드를 사용하여 API 아키텍처를 설계합니다: "e-커머스 플랫폼을 위한 RESTful API를 설계하세요. 제품, 주문, 사용자, 결제를 관리합니다. OpenAPI 3.0 명세를 생성하세요."

Gemini가 전체 API 명세를 작성하고, 엔드포인트, 요청/응답 스키마, 인증 플로우, 오류 처리를 정의합니다.

**단계 2: 코어 구현 (Claude Code)**

Claude Code에 백엔드 구현을 요청합니다. OpenAPI 명세를 제공하면 Claude가 이를 기반으로 Express 라우터, 컨트롤러, 서비스 레이어, 데이터베이스 모델을 생성합니다.

Claude의 강점은 다중 파일 편집입니다. 관련된 모든 파일을 한 번에 업데이트하여 일관성을 유지합니다.

**단계 3: 문서화 (Antigravity + Claude Code)**

Antigravity의 Nano Banana Pro를 사용하여 API 다이어그램과 시각 자료를 생성합니다. Claude Code는 API 문서를 Markdown으로 작성하고 예제 코드를 추가합니다.

**단계 4: 테스트 자동화 (Claude Code)**

Claude Code 서브에이전트를 사용하여 포괄적인 테스트 스위트를 작성합니다. 단위 테스트, 통합 테스트, 엔드투엔드 테스트를 모두 포함합니다.

### 모바일 반응형 랜딩 페이지

프론트엔드에 집중하는 프로젝트에서는 시각적 피드백이 중요합니다.

**단계 1: 디자인 컨셉 (Antigravity)**

Gemini에게 랜딩 페이지 디자인 전략을 요청합니다: "AI 자동화 커뮤니티를 위한 전환 최적화된 랜딩 페이지를 디자인하세요. 현대적인 디자인 원칙, 명확한 CTA, 커뮤니티 혜택 강조를 포함하세요."

**단계 2: HTML/CSS 구현 (Claude Code)**

Claude Code에 구현을 요청합니다: "Gemini가 제안한 디자인에 따라 HTML, CSS, JavaScript로 랜딩 페이지를 만드세요. Tailwind CSS를 사용하고, 애니메이션을 추가하며, 모바일 반응형으로 만드세요."

**단계 3: 실시간 미리보기 (Antigravity)**

Antigravity의 브라우저 패널에서 실시간으로 페이지를 미리보고 테스트합니다. 다양한 화면 크기에서 레이아웃을 확인하고 필요한 조정을 요청합니다.

**단계 4: 이미지 최적화 (Claude Code + Antigravity)**

Antigravity의 Nano Banana Pro로 커스텀 이미지와 아이콘을 생성합니다. Claude Code는 이미지 최적화 스크립트를 작성하여 WebP 변환과 지연 로딩을 구현합니다.

## 5부: 모범 사례와 팁

### 효과적인 프롬프트 작성

두 도구 모두에서 좋은 결과를 얻으려면 명확하고 구체적인 프롬프트가 필요합니다.

**Antigravity용 프롬프트 패턴**

Antigravity는 에이전틱하므로 상세한 지시사항이 필요합니다. "매우 유능한 주니어 개발자에게 주는 상세한 지시사항"처럼 작성합니다.

고수준 목표와 분위기를 설명합니다: "glassmorphism 효과를 사용한 프리미엄하고 차분한 느낌의 물 섭취 추적 앱을 만드세요."

상세한 요구사항을 bullet point로 작성합니다: 기능, UX, 디자인, 성능, 접근성 등 에이전트가 첫 시도에서 품질을 제공하는 데 필요한 모든 것을 포함합니다.

계획 우선 안전: "명령/코드 실행 전에: 상세한 계획 아티팩트(폴더 트리, 종속성, 단계)를 생성하고 승인을 요청하세요"를 항상 포함합니다.

**Claude Code용 프롬프트 패턴**

Claude Code는 터미널 기반이므로 더 직접적이고 실행 중심적인 프롬프트가 효과적입니다.

명확한 작업 정의: "user_auth.py 파일에서 패스워드 해싱 로직을 bcrypt로 변경하세요"

컨텍스트 제공: "@파일명 이 컴포넌트에 로딩 상태를 추가하세요"와 같이 특정 파일을 참조합니다.

예상 결과 명시: "리팩토링 후 모든 기존 테스트가 통과해야 합니다"

### 보안 고려사항

두 도구를 사용할 때 보안을 최우선으로 고려해야 합니다.

**API 키 관리**

절대 API 키를 프롬프트나 코드에 직접 입력하지 않습니다. 항상 .env 파일을 사용하고 .gitignore에 추가합니다.

Claude Code와 Antigravity 모두 환경 변수를 안전하게 읽을 수 있습니다. "GEMINI_API_KEY 환경 변수를 사용하세요"와 같이 명시합니다.

**코드 검토**

에이전트가 생성한 코드를 항상 검토합니다. 특히 보안과 관련된 부분(인증, 데이터베이스 쿼리, 파일 업로드)은 세심하게 확인합니다.

민감한 작업의 경우 에이전트에게 보안 체크리스트를 따르도록 요청합니다: "OWASP Top 10 보안 모범 사례에 따라 이 로그인 기능을 구현하세요."

**데이터 프라이버시**

Claude Code는 로컬에서 실행되며 백엔드 서버 없이 모델 API와 직접 통신합니다. 코드는 사용자의 컴퓨터에 남습니다.

Antigravity도 로컬에 설치되지만 Google 계정과 연동됩니다. 민감한 프로젝트의 경우 어떤 데이터가 Google 서버로 전송되는지 이해해야 합니다.

### 팀 협업

개인 사용을 넘어 팀에서 이러한 도구를 사용할 때의 고려사항입니다.

**공유 Skills 라이브러리**

팀의 모든 구성원이 동일한 Skills를 사용하도록 프로젝트 저장소에 `.claude/skills` 폴더를 포함시킵니다. 팀 특정 워크플로우를 Skills로 문서화하면 일관성이 유지됩니다.

**CLAUDE.md 표준화**

팀 전체가 따르는 표준 CLAUDE.md 템플릿을 만듭니다. 코딩 스타일, 파일 구조, 명명 규칙, 테스트 요구사항 등을 포함합니다.

**버전 관리**

에이전트가 만든 커밋에 일관된 커밋 메시지 형식을 사용합니다. "feat:", "fix:", "refactor:" 같은 접두사를 사용하는 Conventional Commits 형식을 권장합니다.

### 학습 리소스

두 도구를 마스터하기 위한 추가 학습 자료입니다.

**공식 문서**
- Google Antigravity: codelabs.developers.google.com에서 시작 가이드
- Claude Code: claude.com/product/claude-code에서 공식 문서
- Anthropic 프롬프트 엔지니어링: docs.claude.com에서 모범 사례

**커뮤니티 리소스**
- Reddit의 r/ClaudeAI와 r/GoogleAI 커뮤니티
- Discord 서버에서 다른 개발자들과 교류
- YouTube에서 Joe Njenga, Nathan Lambert 같은 크리에이터의 튜토리얼

**실험과 반복**

가장 중요한 학습은 실제 프로젝트에서 나옵니다. 작은 프로젝트로 시작하여 두 도구의 강점과 약점을 파악합니다. 무엇이 효과적인지 기록하고, 실패에서 배우며, 점차 더 복잡한 작업으로 확장합니다.

## 결론: AI 코딩의 미래

Google Antigravity와 Claude Code의 결합은 단순히 두 도구를 함께 사용하는 것 이상의 의미를 갖습니다. 이는 AI 지원 개발의 새로운 패러다임을 보여줍니다. 개발자가 타이피스트에서 아키텍트로, 코드 작성자에서 시스템 설계자로 역할이 전환되고 있습니다.

이러한 도구들은 개발자를 대체하는 것이 아니라 증폭시킵니다. Gemini의 계획 능력과 Claude의 실행 정확성을 결합하면 개발 시간을 65-95% 단축하고 토큰 사용을 50-90% 절감할 수 있다는 보고가 있습니다. 하지만 진정한 가치는 속도가 아니라 품질에 있습니다. 개발자는 이제 반복적인 코딩 대신 제품 비전, 사용자 경험, 시스템 아키텍처에 더 많은 시간을 할애할 수 있습니다.

2026년 현재, 우리는 AI 코딩 혁명의 초기 단계에 있습니다. 도구들은 빠르게 발전하고 있으며, 새로운 기능이 거의 매주 추가됩니다. Antigravity와 Claude Code의 상호운용성(특히 공유 Skills 표준)은 더 개방적이고 협력적인 AI 개발 생태계의 시작을 알립니다.

이 가이드에서 제시한 워크플로우와 패턴은 시작점일 뿐입니다. 각 개발자와 팀은 자신의 필요와 선호에 맞는 고유한 접근 방식을 발견할 것입니다. 중요한 것은 실험하고, 배우고, 적응하는 것입니다. AI 도구는 계속 진화하며, 이를 효과적으로 활용하는 개발자가 다음 세대의 소프트웨어를 만들어갈 것입니다.

이제 여러분의 차례입니다. Google Antigravity를 다운로드하고, Claude Code를 설치하고, 첫 번째 하이브리드 프로젝트를 시작해보세요. 무엇을 만들지는 상상력에 달려 있습니다. 도구는 준비되어 있습니다.

---

**문서 정보**
- 작성 기준: 2026년 1월 최신 정보
- Google Antigravity 버전: 1.14.2 (2026년 1월)
- Claude Code 버전: 2.1.0 (2026년 1월)
- 참고 자료: 공식 문서, 커뮤니티 가이드, 실제 사용 사례

**부록: 빠른 참조**

**Antigravity 단축키**
- Cmd/Ctrl + , : 설정
- Cmd/Ctrl + B : 사이드바 토글
- Cmd/Ctrl + ` : 터미널 토글

**Claude Code 필수 명령어**
- /model : 모델 선택/변경
- /task : 서브에이전트 시작
- /skills : Skills 목록
- /rewind : 이전 상태로 되돌리기
- Ctrl+B : 백그라운드 전환

**문제 해결 체크리스트**
1. 두 도구 모두 최신 버전인가?
2. API 키가 올바르게 설정되었는가?
3. Git 커밋이 동기화되어 있는가?
4. CLAUDE.md 파일이 최신 상태인가?
5. 충분한 시스템 리소스가 있는가?
