---
title: "Claude Code + Antigravity + Tailwind CSS 완벽 가이드"
date: 2026-01-23 06:20:00 +0900
categories: [AI,  Guide]
mermaid: [True]
tags: [AI,  Guide,  Antigravity,  claude-code,  Tailwind-CSS,  Claude.write]
---


> AI 기반 현대 웹 개발의 모든 것: 기초부터 실전 배포까지

## 목차

1. [소개](#소개)
2. [Claude Code 시작하기](#claude-code-시작하기)
3. [Google Antigravity 시작하기](#google-antigravity-시작하기)
4. [Tailwind CSS 기초](#tailwind-css-기초)
5. [Alexandru.so Snippets 활용법](#alexandruso-snippets-활용법)
6. [실전 프로젝트: 인터랙티브 랜딩 페이지](#실전-프로젝트-인터랙티브-랜딩-페이지)
7. [고급 테크닉](#고급-테크닉)
8. [문제 해결](#문제-해결)
9. [베스트 프랙티스](#베스트-프랙티스)
10. [리소스 및 참고 자료](#리소스-및-참고-자료)

---

## 소개

### 이 가이드를 읽어야 하는 이유

2026년 현재, AI 기반 코딩 도구들은 단순한 자동완성을 넘어 실제로 "생각하고 실행하는" 에이전트로 진화했습니다. 특히 Claude Code와 Google Antigravity는 이 분야의 최전선에 있으며, 전통적인 개발 워크플로우를 근본적으로 변화시키고 있습니다.

이 가이드는 다음과 같은 분들을 위해 작성되었습니다.

**초보 개발자**: 코딩을 처음 배우는 분들도 AI의 도움으로 전문가 수준의 웹 애플리케이션을 만들 수 있습니다. 더 이상 수백 시간의 학습이 필요하지 않습니다. AI가 복잡한 로직과 문법을 처리하는 동안, 여러분은 "무엇을 만들지"에 집중할 수 있습니다.

**경험 많은 개발자**: AI를 활용하여 생산성을 10배 이상 높이고, 반복적인 작업에서 벗어나 창의적인 문제 해결에 집중할 수 있습니다. 보일러플레이트 코드 작성, 단순 반복 작업, 문서화 등은 AI에게 맡기고, 여러분은 비즈니스 로직과 아키텍처 설계에 더 많은 시간을 투자하세요.

**디자이너 및 기획자**: 코딩 지식 없이도 자신의 아이디어를 실제 작동하는 프로토타입으로 만들 수 있습니다. 와이어프레임이나 스케치만 있으면, AI가 이를 실제 코드로 변환해줍니다. 더 이상 개발자에게 의존하지 않고도 빠른 프로토타이핑이 가능합니다.

**스타트업 창업자**: MVP를 빠르게 개발하고 시장 검증을 받을 수 있습니다. 몇 주가 걸리던 작업이 몇 시간으로 단축됩니다. 개발 비용을 크게 절감하면서도 빠른 제품 출시가 가능합니다.

### 이 가이드에서 배울 내용

이 가이드를 따라가면 다음과 같은 능력을 갖추게 됩니다.

**Claude Code 마스터하기**: 

터미널에서 Claude와 대화하며 코드를 작성하고, 파일을 편집하고, Git을 관리하는 방법을 배웁니다. Claude Code는 단순한 코드 작성 도구가 아니라 전체 개발 워크플로우를 자동화하는 에이전트입니다. 

프로젝트 초기 설정부터 테스트 작성, 디버깅, 리팩토링, 배포까지 모든 단계에서 Claude가 여러분의 페어 프로그래머 역할을 수행합니다. 특히 Skills 시스템을 활용하면 문서 작성(docx, pdf), 프레젠테이션(pptx), 스프레드시트(xlsx) 작업까지 자동화할 수 있습니다.

**Antigravity 활용하기**: 

Google의 AI 코딩 도구인 Antigravity를 사용하여 Gemini 3 Pro의 강력한 추론 능력을 활용하는 방법을 익힙니다. Stitch MCP와 통합하여 디자인에서 개발까지 전체 파이프라인을 구축합니다.

Antigravity의 멀티모달 기능을 활용하면 이미지나 스케치를 업로드하여 즉시 코드로 변환할 수 있습니다. Firebase, Google Cloud Platform과의 깊은 통합으로 백엔드 설정도 간단해집니다.

**Tailwind CSS 완전 정복**: 

유틸리티 우선(Utility-First) CSS 프레임워크인 Tailwind CSS를 사용하여 빠르고 반응형이며 아름다운 UI를 만드는 방법을 배웁니다. Alexandru.so의 Snippets를 활용하여 JavaScript 없이도 매력적인 인터랙션을 구현합니다.

전통적인 CSS 작성 방식에서 벗어나, HTML 안에서 직접 스타일링하는 방법을 익힙니다. 클래스 이름 고민에 시간을 낭비하지 않고, 일관된 디자인 시스템을 자동으로 유지할 수 있습니다.

**실전 프로젝트 완성**: 

이론만이 아닌 실제로 작동하는 랜딩 페이지, 대시보드, SaaS 애플리케이션을 처음부터 끝까지 만들어봅니다. 각 단계마다 상세한 설명과 함께 진행합니다.

실전 프로젝트에서는 Hero 섹션, Features 섹션, Pricing 테이블, CTA 버튼 등 실제 웹사이트에 필요한 모든 컴포넌트를 구현합니다. 또한 반응형 디자인, 다크모드, 접근성, SEO 최적화까지 프로덕션급 품질을 달성하는 방법을 배웁니다.

### 학습 시간과 필요한 사전 지식

**예상 학습 시간**: 
- 기초 섹션 (1-4장): 2-3시간
- 실전 프로젝트 (5-6장): 3-4시간
- 고급 섹션 (7-9장): 2-3시간
- 총 7-10시간

하지만 실제로는 여러분의 목표에 따라 선택적으로 학습할 수 있습니다. 예를 들어 Tailwind CSS만 빠르게 배우고 싶다면 4장과 5장만 집중하면 됩니다. Claude Code만 알고 싶다면 2장만 학습해도 충분합니다.

**사전 지식**:
- 기본적인 컴퓨터 사용 능력 (파일 생성, 폴더 이동 등)
- HTML/CSS의 기초적인 이해 (선택사항 - 없어도 따라갈 수 있습니다)
- 터미널 명령어 기초 (가이드에서 모두 설명합니다)

코딩 경험이 전혀 없어도 괜찮습니다. 이 가이드는 완전 초보자도 따라할 수 있도록 모든 명령어와 개념을 상세히 설명합니다.

**필요한 도구**:
- 컴퓨터 (macOS, Linux, 또는 Windows)
- 인터넷 연결 (최소 5Mbps 이상 권장)
- 텍스트 에디터 (VS Code 권장, 무료)
- 웹 브라우저 (Chrome 또는 Firefox 권장)
- Claude Pro/Max 구독 또는 API 접근 권한 (Claude Code 사용 시)
- Google 계정 (Antigravity 사용 시)

### 2026년 AI 코딩의 현재 상황

2026년 1월 현재, AI 코딩 도구들은 폭발적으로 성장하고 있습니다. 특히 주목할 만한 트렌드는 다음과 같습니다.

**"Vibe Coding"의 등장**: 

전통적인 방식처럼 코드를 한 줄씩 작성하는 대신, 자연어로 원하는 것을 설명하면 AI가 전체 애플리케이션을 생성합니다. Anthropic의 엔지니어 Felix Rieseberg는 "우리가 원하는 것을 설명하고, Claude가 구현을 처리하고, 우리가 방향을 조정하는" 방식으로 Claude Cowork를 만들었다고 밝혔습니다.

이는 프로그래밍의 근본적인 패러다임 전환입니다. 개발자의 역할이 "코드 작성자"에서 "AI 오케스트레이터"로 변화하고 있습니다. 여러분은 무엇을 만들지 정의하고, AI가 어떻게 만들지 결정합니다.

**에이전트(Agent) 시대**: 

AI가 단순히 코드를 제안하는 것을 넘어, 스스로 에러를 수정하고, 테스트를 실행하고, PR을 제출하는 등 전체 워크플로우를 자율적으로 수행합니다. Ethan Mollick 교수는 "지난 한 달간 AI가 갑작스럽게 능력이 도약했다"고 평가했습니다.

METR(Model Evaluation and Threat Research)의 추적에 따르면, AI가 50% 신뢰도로 자율적으로 완료할 수 있는 작업의 복잡도가 지난 몇 개월간 기하급수적으로 증가했습니다. 2025년 중반만 해도 10-15분 걸리는 작업이 한계였지만, 2026년 1월 현재는 수 시간 걸리는 복잡한 작업도 자율적으로 수행합니다.

**개발자 커뮤니티의 열광**: 

시애틀에서 열린 Claude Code 밋업에는 150명 이상의 엔지니어가 참석했고, AI 전설 Andrej Karpathy는 "프로그래머로서 이렇게 뒤처진 느낌을 받은 적이 없다"고 트윗했습니다. Google의 시니어 엔지니어 Jaana Dogan은 "Claude Code에 문제 설명을 주었더니 작년에 우리가 만든 것을 1시간 만에 생성했다"고 밝혔습니다.

Pioneer Square Labs의 제품 엔지니어 Carly Rector는 "가장 큰 변화는 피드백 루프를 닫는 것"이라고 설명했습니다. AI가 스스로 행동을 취하고, 그 결과를 확인하고, 다음 행동을 결정하는 과정이 자동화되었습니다.

**코드를 넘어선 활용**: 

"Claude Code"라는 이름이 오해를 불러일으킨다는 지적이 있습니다. 이 도구는 코딩뿐만 아니라 세금 신고 준비, 극장 티켓 예약, 스프레드시트 분석 등 컴퓨터로 할 수 있는 거의 모든 작업을 수행할 수 있습니다. The Transformer팀은 "비코더이지만 Claude Code로 은행 명세서를 정리하고 세금 초안을 준비했는데 모두 정확했다"고 보고했습니다.

실제로 Claude Code + Cowork 조합은 일반 사무직 종사자들에게도 엄청난 생산성 향상을 가져다주고 있습니다. 문서 자동화, 데이터 분석, 이메일 정리 등 반복적인 작업을 자동화하여 하루에 4-5시간을 절약한다는 사례가 보고되고 있습니다.

이러한 변화는 단순한 도구의 발전이 아니라, 소프트웨어 개발의 패러다임 전환을 의미합니다. 이제 "아이디어에서 구현까지"의 시간이 몇 주에서 몇 시간으로 단축되고 있습니다.

---

## Claude Code 시작하기

### Claude Code란 무엇인가?

Claude Code는 Anthropic이 2026년 초에 출시한 에이전틱(agentic) 코딩 도구입니다. 가장 큰 특징은 **터미널에서 직접 작동한다**는 점입니다. 웹 인터페이스나 IDE 플러그인이 아닌, 여러분이 이미 사용하고 있는 터미널 환경에서 Claude와 대화하며 코드를 작성합니다.

#### 왜 Claude Code가 특별한가?

**1. 에이전트로서의 자율성**

Claude Code는 단순히 코드를 제안하는 것을 넘어, 실제로 행동을 취합니다. 파일을 직접 편집하고, 명령을 실행하고, Git 커밋을 만들고, 테스트를 실행하고, 에러를 스스로 수정합니다. 여러분은 "무엇을 원하는지"만 말하면 되고, Claude가 "어떻게 할지"를 알아서 처리합니다.

예를 들어 "사용자 인증 시스템을 구현해줘"라고 요청하면:
1. 필요한 패키지를 자동으로 설치하고
2. 데이터베이스 모델을 생성하고
3. API 엔드포인트를 작성하고
4. 테스트 코드를 작성하고
5. 문서화까지 완료합니다

중간에 에러가 발생하면 스스로 디버깅하고 수정합니다. 마치 경험 많은 시니어 개발자가 옆에서 페어 프로그래밍을 하는 것과 같습니다.

**2. 전체 코드베이스 이해**

Claude Code는 프로젝트의 전체 구조와 의존성을 몇 초 만에 파악합니다. 여러분이 수동으로 컨텍스트 파일을 선택할 필요가 없습니다. "에이전틱 검색(agentic search)"을 통해 관련 파일들을 자동으로 찾고 이해합니다.

대규모 프로젝트에서도 문제없습니다. 수백 개의 파일로 구성된 프로젝트에서도 Claude는 관련 파일들을 정확히 찾아내고, 의존성을 이해하며, 변경 사항이 다른 부분에 미칠 영향을 예측합니다.

**3. MCP (Model Context Protocol) 통합**

MCP를 통해 Claude Code는 Google Drive의 디자인 문서를 읽거나, Jira의 티켓을 업데이트하거나, 여러분의 커스텀 개발 도구를 사용할 수 있습니다. 이전 가이드에서 다룬 Stitch MCP도 여기에 해당합니다.

예를 들어:
- Google Drive에서 요구사항 문서를 읽어 코드로 구현
- Jira 티켓을 자동으로 업데이트하고 코드와 연결
- Notion에서 API 명세를 읽어 자동으로 API 구현
- Slack으로 작업 완료 알림 전송

**4. Unix 철학에 충실**

Claude Code는 조합 가능하고 스크립트화할 수 있습니다. 예를 들어:

```bash
tail -f app.log | claude -p "로그 스트림에서 이상 징후가 보이면 Slack으로 알려줘"
```

이런 명령이 실제로 작동합니다. CI/CD 파이프라인에 통합하는 것도 가능합니다. 예를 들어 GitHub Actions에서:

```yaml
- name: Code Review with Claude
  run: |
    git diff main | claude -p "코드 변경사항을 리뷰하고 문제점을 지적해줘"
```

**5. 엔터프라이즈 지원**

Claude API를 사용하거나 AWS 또는 GCP에서 호스팅할 수 있습니다. 기업 환경에서도 안전하게 사용할 수 있도록 설계되었습니다.

엔터프라이즈 버전은 다음을 지원합니다:
- SSO (Single Sign-On) 인증
- 감사 로그 및 모니터링
- 온프레미스 배포
- 커스텀 모델 튜닝
- SLA 보장

### 설치 방법

Claude Code를 설치하는 방법은 운영체제에 따라 다릅니다. 각 운영체제별로 가장 간단한 방법부터 설명드리겠습니다.

#### macOS 설치

**방법 1: 공식 설치 프로그램 (권장)**

가장 간단하고 안정적인 방법입니다. 자동 업데이트도 지원됩니다.

1. https://code.claude.com/download 에서 macOS 설치 프로그램을 다운로드합니다.
2. 다운로드한 `.dmg` 파일을 더블클릭하여 엽니다.
3. Claude Code 아이콘을 Applications 폴더로 드래그합니다.
4. 터미널을 열고 다음 명령을 실행하여 PATH에 추가합니다:

```bash
export PATH="/Applications/Claude Code.app/Contents/MacOS:$PATH"
```

참고: 이 명령은 현재 터미널 세션에서만 유효합니다.

영구적으로 설정하려면 쉘 설정 파일에 추가하세요:

```bash
echo 'export PATH="/Applications/Claude Code.app/Contents/MacOS:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

참고: zsh를 사용하는 경우 `.zshrc`, bash를 사용하는 경우 `.bashrc`를 수정하세요.

**방법 2: Homebrew 사용**

Homebrew를 이미 사용하고 있다면 이 방법이 더 편리합니다.

```bash
brew install --cask claude-code
claude --version
```

설명: 
- `--cask`는 GUI 애플리케이션을 설치할 때 사용하는 옵션입니다.
- 설치 후 `claude --version`으로 정상 설치를 확인합니다.

주의: Homebrew 설치는 자동 업데이트되지 않습니다. 주기적으로 수동 업데이트가 필요합니다:

```bash
brew upgrade claude-code
```

#### Linux 설치

**Ubuntu/Debian 계열**

대부분의 Ubuntu 및 Debian 기반 배포판에서 작동합니다.

1. 공식 리포지토리 GPG 키 추가:

```bash
curl -fsSL https://code.claude.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/claude-code.gpg
```

설명: 
- `curl -fsSL`: GPG 키를 안전하게 다운로드합니다.
- `gpg --dearmor`: GPG 키를 바이너리 형식으로 변환합니다.
- `sudo`: 시스템 디렉토리에 쓰기 위해 관리자 권한이 필요합니다.

2. 리포지토리를 소스 리스트에 추가:

```bash
echo "deb [signed-by=/usr/share/keyrings/claude-code.gpg] https://code.claude.com/linux/ubuntu stable main" | sudo tee /etc/apt/sources.list.d/claude-code.list
```

3. 패키지 목록 업데이트 및 설치:

```bash
sudo apt update
sudo apt install claude-code
```

4. 설치 확인:

```bash
claude --version
```

예상 출력: `Claude Code v2.4.1` 또는 유사한 버전 정보

**Fedora/RHEL 계열**

Fedora, CentOS, RHEL 등에서 사용합니다.

1. RPM 리포지토리 추가:

```bash
sudo dnf config-manager --add-repo https://code.claude.com/linux/fedora/claude-code.repo
```

2. Claude Code 설치:

```bash
sudo dnf install claude-code
```

3. 설치 확인:

```bash
claude --version
```

#### Windows 설치

**방법 1: WinGet 사용 (Windows 11 권장)**

Windows 11에는 WinGet이 기본으로 설치되어 있어 가장 간편합니다.

PowerShell을 관리자 권한으로 열고 다음을 실행:

```powershell
winget install Anthropic.ClaudeCode
```

설치가 완료되면:

```powershell
claude --version
```

주의: 설치 후 PowerShell을 **재시작**해야 `claude` 명령을 사용할 수 있습니다.

업데이트:

```powershell
winget upgrade Anthropic.ClaudeCode
```

**방법 2: 공식 설치 프로그램**

Windows 10 또는 WinGet을 선호하지 않는 경우 이 방법을 사용하세요.

1. https://code.claude.com/download 에서 Windows 설치 프로그램을 다운로드합니다.
2. 다운로드한 `.exe` 파일을 더블클릭하여 실행합니다.
3. 설치 마법사의 지시를 따릅니다.
4. 설치 완료 후 명령 프롬프트나 PowerShell을 **재시작**합니다.
5. `claude --version`으로 설치를 확인합니다.

### 계정 설정 및 로그인

Claude Code를 처음 실행하면 로그인이 필요합니다.

터미널에서 다음을 실행:

```bash
claude
```

첫 실행 시 다음과 같은 프롬프트가 나타납니다:

```
Welcome to Claude Code!

To get started, please log in with your Claude account.

? How would you like to authenticate?
❯ Claude.ai account (Pro, Max, Teams, or Enterprise)
  Claude Console (API access)
  Cancel
```

**옵션 1: Claude.ai 계정 (대부분의 사용자에게 권장)**

이 방법은 Claude Pro, Max, Teams, 또는 Enterprise 구독자를 위한 것입니다.

절차:
1. 화살표 키로 "Claude.ai account" 선택 후 Enter
2. 자동으로 기본 브라우저가 열립니다
3. Claude.ai 로그인 페이지에서 계정 정보 입력
4. "Claude Code 접근 승인" 화면에서 "Allow" 클릭
5. "Authentication successful!" 메시지 확인
6. 브라우저 탭을 닫고 터미널로 돌아갑니다

인증이 완료되면 터미널에 다음과 같이 표시됩니다:

```
✓ Successfully authenticated as: your-email@example.com
✓ Subscription: Claude Pro
✓ Ready to code!
```

**옵션 2: Claude Console (API 직접 사용)**

개발자나 엔터프라이즈 사용자를 위한 방법입니다.

절차:
1. "Claude Console" 선택 후 Enter
2. 브라우저에서 https://console.anthropic.com 방문
3. 로그인 후 "API Keys" 섹션으로 이동
4. "Create Key" 버튼 클릭
5. 키 이름을 입력하고 생성
6. 생성된 API 키를 복사 (한 번만 표시됩니다!)
7. 터미널로 돌아가서 API 키 붙여넣기

API 키는 안전하게 암호화되어 저장되며, 다음부터는 자동으로 사용됩니다.

**로그인 상태 확인**

언제든지 현재 로그인 상태를 확인할 수 있습니다:

```bash
claude auth status
```

예상 출력:

```
✓ Authenticated as: your-email@example.com
Subscription: Claude Pro
Model access: Opus 4.5, Sonnet 4.5, Haiku 4.5
Token usage this month: 1.2M / 10M
```

설명:
- 현재 로그인된 계정
- 구독 플랜
- 사용 가능한 모델
- 이번 달 토큰 사용량

**로그아웃**

필요시 다음 명령으로 로그아웃할 수 있습니다:

```bash
claude auth logout
```

확인 메시지:

```
? Are you sure you want to logout? (y/n)
```

로그아웃하면 저장된 인증 정보가 모두 삭제됩니다.

### 기본 사용법

Claude Code는 크게 두 가지 모드로 사용할 수 있습니다: **대화형 모드**와 **일회성 명령 모드**입니다.

#### 대화형 모드 (Chat Mode)

대화형 모드는 Claude와 지속적으로 대화하며 작업하는 방식입니다. 복잡한 프로젝트나 여러 단계를 거쳐야 하는 작업에 적합합니다.

**시작하기:**

프로젝트 디렉토리로 이동한 후:

```bash
cd /path/to/your/project
claude
```

또는 현재 디렉토리에서 바로 시작:

```bash
claude
```

Claude Code 프롬프트가 나타납니다:

```
Claude Code v2.4.1 | Opus 4.5
Working directory: /Users/username/projects/my-app
Ready. What can I help you build?

You:
```

**대화 예시:**

```
You: 이 프로젝트의 구조를 설명해줘

Claude: 프로젝트 구조를 분석하고 있습니다...

이 프로젝트는 Express.js 기반의 웹 애플리케이션입니다. 구조는 다음과 같습니다:

project/
├── src/
│   ├── routes/        (API 라우트 정의)
│   ├── controllers/   (비즈니스 로직)
│   ├── models/        (데이터 모델, MongoDB)
│   └── middleware/    (인증, 에러 처리 등)
├── public/            (정적 파일)
├── views/             (EJS 템플릿)
├── tests/             (Jest 테스트)
└── package.json       (의존성 및 스크립트)

주요 의존성:
- express: 웹 프레임워크
- mongoose: MongoDB ORM
- jsonwebtoken: JWT 인증
- bcrypt: 비밀번호 해싱

다음에 무엇을 도와드릴까요?
```

**연속 대화:**

```
You: 사용자 인증 시스템에 이메일 인증 기능을 추가해줘

Claude: 이메일 인증 기능을 추가하겠습니다.

다음 단계로 진행됩니다:
1. 이메일 전송을 위한 nodemailer 설치
2. 인증 토큰 모델 생성
3. 이메일 전송 서비스 구현
4. 인증 엔드포인트 추가
5. 기존 회원가입 로직 수정

? 진행할까요? (y/n)
```

**대화형 모드의 장점:**

1. **컨텍스트 유지**: 이전 대화 내용을 기억하여 연속적인 작업이 가능합니다.
2. **단계별 진행**: 복잡한 작업을 여러 단계로 나누어 진행할 수 있습니다.
3. **즉각적인 피드백**: 각 단계마다 결과를 확인하고 방향을 조정할 수 있습니다.
4. **학습 기회**: Claude의 설명을 들으며 코드를 이해할 수 있습니다.

**대화형 모드 종료:**

다음 중 하나를 사용:

```
You: exit
```

또는 키보드로 `Ctrl+D` (Unix/Mac) 또는 `Ctrl+Z` (Windows)

#### 일회성 명령 모드 (One-shot Mode)

간단한 작업이나 스크립트에서 사용하기 좋은 방식입니다. 한 번의 명령으로 작업을 완료하고 종료합니다.

**기본 사용법:**

```bash
claude -p "README.md 파일을 만들어줘. 프로젝트 설명과 설치 방법을 포함해야 해"
```

설명:
- `-p` 또는 `--prompt`: 프롬프트를 직접 전달하는 옵션
- 따옴표 안의 텍스트가 Claude에게 전달됩니다

**여러 줄 명령:**

복잡한 요청은 여러 줄로 작성할 수 있습니다:

```bash
claude -p "
이 프로젝트에 Docker 설정을 추가해줘:
1. Dockerfile 생성 (Node.js 18 베이스)
2. docker-compose.yml 생성 (app + MongoDB)
3. .dockerignore 생성
4. README에 Docker 사용법 추가
"
```

**파이프와 결합:**

Claude Code는 Unix 철학을 따르므로 파이프와 결합할 수 있습니다. 이는 매우 강력한 기능입니다.

**예제 1: 로그 모니터링**

```bash
tail -f app.log | claude -p "에러가 보이면 원인을 분석하고 Slack으로 알려줘"
```

설명:
- `tail -f app.log`: 실시간으로 로그를 읽습니다
- 파이프(`|`)로 Claude에 전달
- Claude가 에러를 감지하면 자동으로 분석하고 Slack 알림 전송

**예제 2: Git diff 리뷰**

```bash
git diff | claude -p "이 변경사항을 리뷰하고 개선점을 제안해줘"
```

설명:
- 현재 Git 변경사항을 Claude에 전달
- Claude가 코드 리뷰를 수행하고 개선점 제안

**예제 3: 테스트 결과 분석**

```bash
npm test | claude -p "테스트 결과를 요약하고 실패한 테스트를 자세히 설명해줘"
```

**예제 4: 파일 내용 기반 작업**

```bash
cat requirements.txt | claude -p "이 요구사항을 분석하고 구현 계획을 작성해줘"
```

### Claude Code의 핵심 기능

Claude Code의 진정한 힘은 다음 핵심 기능들에 있습니다.

#### 1. 파일 편집

Claude Code는 파일을 직접 생성하고 편집할 수 있습니다. 여러분의 승인을 받은 후 실제 파일 시스템을 수정합니다.

**파일 생성 예제:**

```bash
claude

You: src/utils 디렉토리에 date-formatter.js 파일을 만들어줘.
     날짜를 다양한 형식으로 포맷하는 유틸리티 함수들을 포함해야 해.

Claude: 파일을 생성하고 있습니다...

✓ Created src/utils/date-formatter.js

파일 내용 미리보기:

export const formatDate = (date, format) => {
  const options = {
    'YYYY-MM-DD': { year: 'numeric', month: '2-digit', day: '2-digit' },
    'DD/MM/YYYY': { day: '2-digit', month: '2-digit', year: 'numeric' },
    'MMMM DD, YYYY': { year: 'numeric', month: 'long', day: '2-digit' }
  };
  return new Date(date).toLocaleDateString('en-US', options[format]);
};

다음 작업을 제안합니다:
1. 이 유틸리티의 테스트 작성
2. JSDoc 문서 추가
3. README에 사용 예제 추가

어떤 것부터 진행할까요?
```

**파일 편집 승인:**

Claude는 파일을 변경하기 전에 항상 승인을 요청합니다(기본 설정):

```
? Claude would like to create src/utils/date-formatter.js. 
  Contents:
  - formatDate function
  - formatTime function
  - formatDateTime function
  
  Allow? (y/n)
```

승인 옵션:
- `y`: 이번만 허용
- `n`: 거부
- `a`: 이 세션 동안 모든 파일 변경 자동 승인
- `q`: 작업 취소하고 종료

**자동 승인 모드:**

신뢰할 수 있는 환경에서는 자동 승인 모드를 사용할 수 있습니다:

```bash
claude --auto-approve
```

주의: 보안을 위해 신중하게 사용하세요. 중요한 프로젝트에서는 권장하지 않습니다.

#### 2. 명령 실행

Claude는 터미널 명령을 실행할 수 있습니다. 패키지 설치, 테스트 실행, 빌드 등 모든 작업이 가능합니다.

**예제:**

```
You: 프로젝트의 모든 의존성을 최신 버전으로 업데이트해줘

Claude: 다음 명령을 실행하겠습니다:

1. npm outdated (현재 버전 확인)
2. npm update (마이너/패치 업데이트)
3. npm audit fix (보안 취약점 수정)

? Execute these commands? (y/n)
```

승인 후:

```
✓ Executed: npm outdated

Package      Current  Wanted  Latest
express       4.17.1  4.18.2  4.18.2
mongoose       6.0.12   7.2.1   7.2.1
jsonwebtoken   8.5.1   9.0.0   9.0.0

✓ Executed: npm update
✓ Executed: npm audit fix

업데이트 완료:
- express: 4.17.1 → 4.18.2
- mongoose: 6.0.12 → 7.2.1
- jsonwebtoken: 8.5.1 → 9.0.0

2개의 보안 취약점이 수정되었습니다.

package-lock.json이 업데이트되었습니다.
커밋하시겠습니까? (y/n)
```

**복잡한 명령 체인:**

```
You: 프로덕션 빌드를 만들고 테스트한 다음 Docker 이미지로 빌드해줘

Claude: 다음 단계를 수행하겠습니다:

1. npm run build (프로덕션 빌드)
2. npm test (테스트 실행)
3. docker build -t myapp:latest . (Docker 이미지 빌드)
4. docker tag myapp:latest myapp:v1.2.3 (버전 태그 추가)

? 진행할까요? (y/n)
```

#### 3. Git 워크플로우

Claude Code는 Git을 완벽하게 이해하고 통합합니다. 커밋, 브랜치, PR 생성까지 모두 자동화할 수 있습니다.

**커밋 예제:**

```
You: 현재 변경사항을 커밋하고 feature/user-auth 브랜치에 푸시해줘

Claude: 다음 작업을 수행하겠습니다:

1. 변경된 파일 확인 및 스테이징
2. 의미있는 커밋 메시지 작성
3. feature/user-auth 브랜치로 푸시

변경된 파일:
- src/auth/login.js (modified)
- src/auth/signup.js (new)
- src/auth/jwt.js (modified)
- tests/auth.test.js (new)
- README.md (modified)

제안하는 커밋 메시지:
feat: Add user authentication system

- Implement login and signup endpoints
- Add JWT token generation and validation
- Include comprehensive test coverage
- Update README with API documentation

? Proceed with commit and push? (y/n)
```

**브랜치 관리:**

```
You: main 브랜치를 기준으로 feature/payment 브랜치를 만들어줘

Claude: 
✓ git checkout main
✓ git pull origin main
✓ git checkout -b feature/payment

새 브랜치 'feature/payment'가 생성되었습니다.
작업을 시작하시겠습니까?
```

**PR 생성 (GitHub MCP 사용):**

```
You: 현재 브랜치를 main에 머지하는 PR을 만들어줘

Claude: GitHub PR을 생성하고 있습니다...

PR 제목: feat: Add payment integration with Stripe
본문:
## Changes
- Implemented Stripe payment integration
- Added payment webhook handlers
- Created payment model and migration
- Added comprehensive tests

## Testing
- All tests passing
- Manual testing completed
- Stripe webhooks verified

✓ PR created: https://github.com/user/repo/pull/42

리뷰어를 지정하시겠습니까?
```

#### 4. 프로젝트 이해 및 탐색

Claude Code의 강력한 기능 중 하나는 프로젝트 전체를 빠르게 이해하는 능력입니다. 에이전틱 검색을 통해 관련 파일들을 자동으로 찾아냅니다.

**프로젝트 분석 예제:**

```
You: 이 프로젝트에서 인증이 어떻게 구현되어 있는지 설명해줘

Claude: 프로젝트를 분석하고 있습니다...
(관련 파일 자동 검색: src/auth/*, src/middleware/auth.js, ...)

인증 시스템 개요:

**1. 전략**
JWT (JSON Web Token) 기반 인증을 사용합니다.
위치: src/auth/jwt.js

**2. 주요 컴포넌트**

로그인 (src/auth/login.js):
- 이메일/비밀번호 검증
- bcrypt로 비밀번호 확인
- JWT 토큰 생성 및 반환

회원가입 (src/auth/signup.js):
- 사용자 데이터 검증 (Joi 스키마)
- 비밀번호 bcrypt 해싱 (10 rounds)
- 사용자 생성 및 저장

미들웨어 (src/middleware/auth.js):
- JWT 토큰 검증
- 만료 시간 확인
- 사용자 권한 체크
- req.user에 사용자 정보 첨부

**3. 토큰 관리**
- 액세스 토큰: 15분 유효 (짧은 수명)
- 리프레시 토큰: 7일 유효 (자동 갱신)
- 저장 위치: MongoDB users 컬렉션 (refreshTokens 필드)

**4. 보안 고려사항**
✓ 비밀번호 해싱 (bcrypt, 10 rounds)
✓ HTTPS 전용 쿠키 (secure: true)
✓ CORS 설정 (특정 도메인만 허용)
✓ JWT 서명 검증
⚠ Rate limiting 없음 (추가 권장)
⚠ 계정 잠금 기능 없음 (브루트 포스 방어 미흡)

**5. 개선 제안**
1. express-rate-limit 추가 (로그인 시도 제한)
2. 계정 잠금 기능 구현 (5회 실패 시)
3. 2FA (Two-Factor Authentication) 옵션 추가
4. 감사 로그 추가 (로그인 시도 기록)

개선사항을 구현하시겠습니까?
```

**코드 플로우 추적:**

```
You: 사용자가 로그인할 때 어떤 과정을 거치는지 단계별로 설명해줘

Claude: 로그인 플로우를 추적하고 있습니다...

로그인 프로세스 (POST /api/auth/login):

**1단계: 요청 수신**
파일: src/routes/auth.js (line 15)
router.post('/login', loginController.login)

**2단계: 입력 검증**
파일: src/controllers/auth/login.js (lines 10-20)
- Joi 스키마로 검증
- 필수 필드: email, password
- 이메일 형식 확인
→ 검증 실패 시: 400 Bad Request 반환

**3단계: 사용자 조회**
파일: src/controllers/auth/login.js (lines 22-25)
- MongoDB에서 이메일로 사용자 검색
- User.findOne({ email })
→ 사용자 없음: 401 Unauthorized 반환

**4단계: 비밀번호 확인**
파일: src/controllers/auth/login.js (lines 27-30)
- bcrypt.compare(password, user.hashedPassword)
- 소금값(salt)은 해시에 포함되어 있음
→ 불일치: 401 Unauthorized 반환

**5단계: JWT 토큰 생성**
파일: src/auth/jwt.js (lines 15-30)
- 액세스 토큰 생성 (15분 유효)
- 리프레시 토큰 생성 (7일 유효)
- 페이로드: { userId, email, role }
- 서명: process.env.JWT_SECRET

**6단계: 리프레시 토큰 저장**
파일: src/controllers/auth/login.js (lines 35-40)
- MongoDB에 리프레시 토큰 저장
- user.refreshTokens.push({ token, expiresAt })
→ 나중에 토큰 갱신 시 사용

**7단계: 응답 전송**
파일: src/controllers/auth/login.js (lines 42-48)
- 200 OK
- Body: { accessToken, refreshToken, user }
- 쿠키 설정 (httpOnly, secure, sameSite)

전체 처리 시간: 평균 150-200ms
(대부분은 bcrypt 검증에 소요)

특정 단계를 더 자세히 알아보시겠습니까?
```

#### 5. 테스트 작성 및 실행

Claude는 테스트 작성의 베스트 프랙티스를 알고 있으며, 포괄적인 테스트 케이스를 자동으로 생성합니다.

**테스트 작성 예제:**

```
You: login.js의 유닛 테스트를 작성하고 실행해줘

Claude: 테스트를 작성하고 있습니다...

✓ Created tests/auth/login.test.js

테스트 케이스 (총 8개):

**성공 케이스:**
1. 유효한 자격증명으로 로그인 성공
2. JWT 토큰이 올바르게 생성되는지 확인
3. 리프레시 토큰이 DB에 저장되는지 확인

**실패 케이스:**
4. 잘못된 비밀번호로 로그인 실패 (401)
5. 존재하지 않는 사용자로 로그인 실패 (401)
6. 이메일 형식이 잘못됨 (400)
7. 비밀번호 누락 (400)
8. 이메일 누락 (400)

**엣지 케이스:**
- SQL 인젝션 시도 차단 확인
- 매우 긴 비밀번호 처리
- 특수문자가 포함된 이메일

테스트를 실행하시겠습니까? (y/n)
```

승인 후:

```
✓ npm test tests/auth/login.test.js

 PASS  tests/auth/login.test.js
  Login Controller
    ✓ should login successfully with valid credentials (245ms)
    ✓ should return JWT tokens (52ms)
    ✓ should store refresh token in database (89ms)
    ✓ should fail with wrong password (156ms)
    ✓ should fail with non-existent user (43ms)
    ✓ should fail with invalid email format (28ms)
    ✓ should fail with missing password (21ms)
    ✓ should fail with missing email (19ms)

Test Suites: 1 passed, 1 total
Tests:       8 passed, 8 total
Snapshots:   0 total
Time:        2.341s

✅ 모든 테스트 통과!

코드 커버리지:
- Statements: 95.2%
- Branches: 91.7%
- Functions: 100%
- Lines: 94.8%

추가 테스트를 작성하시겠습니까?
```

**통합 테스트 작성:**

```
You: 전체 인증 플로우의 통합 테스트를 작성해줘.
     회원가입 → 로그인 → 토큰 갱신 → 로그아웃 순서로.

Claude: 통합 테스트를 작성하고 있습니다...

✓ Created tests/integration/auth-flow.test.js

이 테스트는 다음을 검증합니다:
1. 새 사용자 회원가입
2. 생성된 계정으로 로그인
3. 액세스 토큰으로 보호된 라우트 접근
4. 만료된 토큰으로 갱신 시도
5. 로그아웃 (토큰 무효화)
6. 로그아웃 후 토큰 사용 불가 확인

실제 데이터베이스를 사용하여 테스트합니다.
테스트 DB: test_myapp (MongoDB)

? 테스트를 실행할까요? (y/n)
```

### Claude Code Skills

Skills는 Claude Code의 가장 혁신적인 기능 중 하나입니다. 특정 작업을 수행하는 방법에 대한 전문 지식과 도구의 조합입니다.

#### Skills의 개념

전통적인 AI 도구에서는 사용자가 매번 상세한 프롬프트를 제공해야 했습니다. "웹사이트를 만들려면 HTML 구조를 이렇게 하고, CSS는 저렇게 하고, 반응형 디자인은..." 식으로 매번 설명해야 했죠.

Skills는 이 문제를 해결합니다. Claude가 작업의 성격을 파악하면 자동으로 적절한 Skill을 로드합니다. 각 Skill에는 다음이 포함됩니다:

- **모범 사례**: 해당 작업을 수행하는 최선의 방법
- **도구**: 작업에 필요한 라이브러리와 프레임워크
- **체크리스트**: 놓치기 쉬운 중요한 사항들
- **예제**: 참고할 수 있는 코드 샘플

예를 들어 "프레젠테이션을 만들어줘"라고 하면:
1. Claude가 `pptx` Skill을 자동으로 로드
2. PowerPoint 파일 생성 방법, 슬라이드 레이아웃, 디자인 원칙 등을 참조
3. 전문적인 품질의 프레젠테이션 생성

#### 기본 제공 Skills

Claude Code는 다음과 같은 기본 Skills를 제공합니다:

**1. docx (Microsoft Word 문서)**

기능:
- Word 문서 작성, 편집, 형식 지정
- 추적된 변경사항(Track Changes) 처리
- 댓글 및 리뷰 기능
- 표, 이미지, 차트 삽입
- 페이지 번호, 목차, 색인 생성

적합한 용도:
- 비즈니스 보고서
- 기술 문서
- 제안서 및 계약서
- 회의록
- 사용자 매뉴얼

예제:
```
You: 우리 프로젝트의 기술 문서를 Word 파일로 만들어줘.
     아키텍처 다이어그램과 API 명세를 포함해야 해.

Claude: [docx Skill 활성화]
기술 문서를 작성하고 있습니다...

✓ Created technical-documentation.docx

문서 구성:
1. 표지 (프로젝트명, 버전, 날짜)
2. 목차 (자동 생성)
3. 프로젝트 개요
4. 시스템 아키텍처 (다이어그램 포함)
5. API 명세 (엔드포인트별 표)
6. 데이터베이스 스키마
7. 부록 (용어집)

페이지 수: 24
파일 크기: 1.2 MB

? 문서를 열어보시겠습니까? (y/n)
```

**2. pdf (PDF 문서)**

기능:
- PDF 생성, 편집, 병합
- 텍스트 및 표 추출
- PDF 양식 작성
- 문서 분할
- 워터마크 추가

적합한 용도:
- 송장 및 영수증
- 계약서 및 법률 문서
- 포트폴리오
- 전자책
- 공식 보고서

예제:
```
You: 월간 판매 리포트를 PDF로 만들어줘.
     데이터는 sales.csv에서 가져와서 차트와 표를 포함해야 해.

Claude: [pdf Skill 활성화]
판매 리포트를 생성하고 있습니다...

1. sales.csv 데이터 분석 완료
2. 차트 생성 중 (매출 추이, 제품별 비교, 지역별 분포)
3. PDF 레이아웃 작성 중

✓ Created monthly-sales-report-2026-01.pdf

내용:
- 요약 대시보드 (1페이지)
- 전월 대비 성장률: +12.5%
- 매출 추이 차트
- 제품별 판매 현황 (표)
- 지역별 분석
- 개선 제안사항

파일 크기: 845 KB
```

**3. pptx (PowerPoint 프레젠테이션)**

기능:
- PowerPoint 프레젠테이션 생성 및 편집
- 슬라이드 레이아웃 자동 선택
- 차트, 표, 이미지 삽입
- 애니메이션 및 전환 효과
- 스피커 노트 추가

적합한 용도:
- 비즈니스 피치 덱
- 교육 자료
- 프로젝트 발표
- 분기별 리뷰
- 워크샵 자료

예제:
```
You: 투자자 피치 덱을 만들어줘.
     회사 소개, 문제점, 솔루션, 시장 규모, 팀, 재무 계획 순서로.

Claude: [pptx Skill 활성화]
투자자 피치 덱을 작성하고 있습니다...

슬라이드 구성 (총 12장):
1. 타이틀 슬라이드
2. 문제점 (현재 시장의 고충)
3. 우리의 솔루션
4. 제품 데모 (스크린샷)
5. 시장 기회 (TAM, SAM, SOM)
6. 비즈니스 모델
7. 경쟁 분석
8. 핵심 팀 소개
9. 트랙션 (성장 지표)
10. 재무 계획 (3개년)
11. 자금 조달 계획
12. Thank You (연락처)

✓ Created investor-pitch-deck.pptx

디자인: 프로페셔널 블루 테마
총 슬라이드: 12
파일 크기: 2.3 MB

다음 단계:
- 실제 데이터로 차트 업데이트
- 팀 사진 추가
- 스피커 노트 작성

? 지금 진행할까요? (y/n)
```

**4. xlsx (Excel 스프레드시트)**

기능:
- Excel 스프레드시트 생성 및 편집
- 수식 및 함수 사용
- 피벗 테이블 생성
- 차트 및 시각화
- 조건부 서식
- 데이터 검증

적합한 용도:
- 재무 모델링
- 데이터 분석
- 예산 관리
- 인벤토리 추적
- 프로젝트 일정표

예제:
```
You: 프로젝트 예산 관리 스프레드시트를 만들어줘.
     카테고리별 예산, 실제 지출, 남은 예산을 자동 계산해야 해.

Claude: [xlsx Skill 활성화]
예산 관리 스프레드시트를 작성하고 있습니다...

✓ Created project-budget.xlsx

시트 구성:
1. Dashboard (요약 및 차트)
2. Budget (카테고리별 예산)
3. Expenses (지출 내역)
4. Analysis (분석 및 인사이트)

주요 기능:
- 자동 계산 수식 (=SUM, =IF, =VLOOKUP)
- 조건부 서식 (예산 초과 시 빨간색)
- 파이 차트 (카테고리별 지출 비율)
- 선 차트 (월별 누적 지출)
- 데이터 검증 (카테고리 드롭다운)

샘플 데이터가 포함되어 있습니다.
실제 데이터로 교체하시겠습니까?
```

**5. frontend-design (프론트엔드 디자인)**

기능:
- 고품질 UI/UX 컴포넌트 생성
- React, Vue, HTML/CSS 지원
- Tailwind CSS 통합
- 반응형 디자인
- 접근성 (WCAG 준수)

적합한 용도:
- 랜딩 페이지
- 웹 애플리케이션 UI
- 대시보드
- 이커머스 사이트
- 포트폴리오

예제:
```
You: 미니멀한 SaaS 랜딩 페이지를 만들어줘.
     Hero, Features, Pricing, FAQ, CTA 섹션을 포함해야 해.

Claude: [frontend-design Skill 활성화]

디자인 방향을 결정하고 있습니다...

**타겟 오디언스**: B2B SaaS
**디자인 스타일**: 미니멀, 모던, 신뢰감
**브랜드 컬러**: 파란색(#2563EB) + 회색(#64748B)

**기술 스택 선택:**
❯ React + Tailwind CSS (권장)
  Next.js + Tailwind CSS
  Vue + Tailwind CSS
  HTML + Vanilla CSS

[React + Tailwind 선택]

✓ 프로젝트 구조 생성
✓ Tailwind 설정 완료
✓ Hero 섹션 (Text Shimmer Effect 적용)
✓ Features 섹션 (3D Card 효과)
✓ Pricing 섹션 (3-Tier 구조)
✓ FAQ 섹션 (아코디언)
✓ CTA 섹션 (Gradient Button)

✓ 반응형 디자인 적용 (Mobile, Tablet, Desktop)
✓ 다크모드 지원
✓ SEO 메타 태그 추가
✓ 접근성 WCAG 2.1 AA 준수

로컬 개발 서버를 시작하시겠습니까?
```

**6. product-self-knowledge (제품 정보)**

기능:
- Anthropic 제품에 대한 정확한 정보 제공
- API 사용법 및 제한사항
- 가격 정보
- 기능 설명
- 트러블슈팅 가이드

적합한 용도:
- Claude 관련 질문 답변
- API 통합 도움
- 기능 문의
- 가격 및 플랜 비교

예제:
```
You: Claude API의 현재 가격과 사용 제한을 알려줘

Claude: [product-self-knowledge Skill 활성화]

Claude API 가격 (2026년 1월 기준):

**Claude Opus 4.5**
- Input: $15 / 1M tokens
- Output: $75 / 1M tokens
- 컨텍스트 윈도우: 200K tokens
- 적합한 용도: 복잡한 추론, 장문 생성

**Claude Sonnet 4.5**
- Input: $3 / 1M tokens
- Output: $15 / 1M tokens
- 컨텍스트 윈도우: 200K tokens
- 적합한 용도: 일반적인 작업, 비용 효율적

**Claude Haiku 4.5**
- Input: $0.25 / 1M tokens
- Output: $1.25 / 1M tokens
- 컨텍스트 윈도우: 200K tokens
- 적합한 용도: 빠른 응답, 대량 처리

**사용 제한**:
- Rate Limit: 분당 5000 requests (기본)
- Tier 1: 분당 50 requests
- Tier 2: 분당 500 requests
- Tier 3: 분당 2000 requests
- Tier 4: 분당 5000 requests

더 높은 한도가 필요하면 enterprise@anthropic.com으로 문의하세요.
```

#### Skills 사용 예시

Skills는 대부분 자동으로 활성화되지만, 명시적으로 요청할 수도 있습니다.

**자동 활성화 예제:**

```
You: 우리 회사 소개 프레젠테이션을 만들어줘

Claude: [자동으로 pptx Skill 로드]

프레젠테이션을 작성하겠습니다.

슬라이드 개요를 먼저 확인해주세요:
1. 회사 소개
2. 우리의 미션
3. 주요 제품/서비스
4. 고객 및 파트너
5. 팀 소개
6. 연락처

? 이 구조로 진행할까요? (y/n)
```

**명시적 요청 예제:**

```
You: frontend-design Skill을 사용해서 
     전자상거래 제품 카드 컴포넌트를 만들어줘

Claude: [frontend-design Skill 활성화]

전자상거래 제품 카드를 디자인하겠습니다.

**컴포넌트 요구사항 수집:**
- 제품 이미지
- 제품명
- 가격 (할인가 표시 옵션)
- 별점 및 리뷰 수
- 장바구니 추가 버튼
- 위시리스트 버튼
- 호버 시 퀵뷰 옵션

**인터랙션:**
- 이미지 호버 시 확대
- "Add to Cart" 버튼 애니메이션
- 위시리스트 아이콘 토글
- 로딩 상태 표시

**기술 선택:**
- React 컴포넌트
- Tailwind CSS
- Lucide React (아이콘)

? 시작할까요? (y/n)
```

#### 커스텀 Skills 만들기

프로젝트에 특화된 Skill을 만들 수 있습니다. 프로젝트 루트에 `CLAUDE.md` 파일을 생성하세요.

**CLAUDE.md 예제:**

~~~markdown
# My E-commerce Project Skill

## 프로젝트 개요
이 프로젝트는 B2C 전자상거래 플랫폼입니다.
사용자가 제품을 검색, 구매하고 리뷰를 작성할 수 있습니다.

## 기술 스택
- **프론트엔드**: Next.js 14, React 18, Tailwind CSS
- **백엔드**: Node.js, Express, PostgreSQL
- **결제**: Stripe
- **인증**: NextAuth.js
- **배포**: Vercel (프론트), Railway (백엔드)

## 코딩 스타일
- **언어**: TypeScript (strict mode)
- **비동기 처리**: async/await (Promise 체인 금지)
- **에러 처리**: try-catch + 중앙화된 에러 핸들러
- **함수**: 단일 책임 원칙, 20줄 이하 권장
- **네이밍**: camelCase (변수/함수), PascalCase (컴포넌트/클래스)

## 디렉토리 구조
```
src/
├── app/              (Next.js App Router)
├── components/
│   ├── ui/          (재사용 가능한 UI 컴포넌트)
│   └── features/    (기능별 컴포넌트)
├── lib/             (유틸리티 함수)
├── hooks/           (커스텀 훅)
├── types/           (TypeScript 타입)
└── api/             (API 라우트)
```

## 컴포넌트 가이드라인
- 모든 컴포넌트는 TypeScript로 작성
- Props는 interface로 명시
- 기본 props 값 제공
- PropTypes 대신 TypeScript 사용

예제:
```typescript
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
}

export default function Button({ 
  label, 
  onClick, 
  variant = 'primary',
  size = 'md' 
}: ButtonProps) {
  // ...
}
```

## 테스트 규칙
- **프레임워크**: Jest + React Testing Library
- **커버리지 목표**: 80% 이상
- **테스트 파일**: `*.test.tsx` (컴포넌트 옆에 위치)
- **E2E 테스트**: Playwright

## API 개발 가이드라인
- RESTful API 설계
- 응답 형식:
  ```json
  {
    "success": true,
    "data": { ... },
    "error": null
  }
  ```
- 에러 응답:
  ```json
  {
    "success": false,
    "data": null,
    "error": {
      "code": "VALIDATION_ERROR",
      "message": "Invalid email format"
    }
  }
  ```

## 데이터베이스
- **ORM**: Prisma
- **마이그레이션**: 항상 `prisma migrate`사용
- **시딩**: `prisma db seed` 스크립트

## Git 워크플로우
- **브랜치 전략**: Git Flow
- **메인 브랜치**: `main`, `develop`
- **피처 브랜치**: `feature/기능명`
- **버그픽스**: `bugfix/버그명`
- **핫픽스**: `hotfix/버그명`

## 커밋 메시지 규칙
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types:
- `feat`: 새 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 포맷팅 (기능 변경 없음)
- `refactor`: 리팩토링
- `test`: 테스트 추가/수정
- `chore`: 빌드 프로세스, 도구 설정 등

예제:
```
feat(products): Add product filtering by category

- Implemented category dropdown in product list
- Added API endpoint for category filter
- Updated product model to include category field

Closes #123
```

## 환경 변수
- `.env.local`: 로컬 개발용
- `.env.production`: 프로덕션용
- 절대 커밋하지 말 것!
- `.env.example`에 필요한 변수 목록 유지

## 성능 최적화
- 이미지: Next.js Image 컴포넌트 사용
- 코드 스플리팅: 동적 import 사용
- 캐싱: React Query 사용
- 번들 크기: webpack-bundle-analyzer로 모니터링

## 자주 사용하는 명령
- `npm run dev`: 개발 서버 시작
- `npm run build`: 프로덕션 빌드
- `npm test`: 테스트 실행
- `npm run lint`: ESLint 실행
- `npm run type-check`: TypeScript 타입 체크
- `prisma studio`: 데이터베이스 GUI

## 배포 프로세스
1. `develop` 브랜치에서 기능 개발
2. PR 생성 및 리뷰
3. `develop`에 머지
4. 스테이징 환경에서 테스트
5. `main`에 머지
6. 자동 배포 (Vercel)

## 보안 가이드라인
- 사용자 입력은 항상 검증 및 sanitize
- SQL Injection 방지 (Prisma 사용)
- XSS 방지 (React 자동 escape)
- CSRF 토큰 사용
- 민감한 데이터는 환경 변수에 저장
- HTTPS 강제 사용
~~~

**커스텀 Skill 사용 효과:**

이제 Claude는 이 프로젝트에서 작업할 때 자동으로 이 가이드라인을 따릅니다:

```
You: 제품 카테고리 필터 기능을 추가해줘

Claude: [CLAUDE.md 가이드라인 적용]

제품 카테고리 필터 기능을 추가하겠습니다.

**구현 계획:**
1. Prisma 스키마 업데이트 (Product 모델에 category 추가)
2. 데이터베이스 마이그레이션
3. API 엔드포인트 생성 (GET /api/products?category=)
4. 프론트엔드 UI 컴포넌트 (카테고리 드롭다운)
5. React Query로 데이터 페칭 및 캐싱
6. 테스트 작성

**파일 변경:**
- `prisma/schema.prisma`: Product 모델 수정
- `src/app/api/products/route.ts`: API 핸들러
- `src/components/features/ProductFilter.tsx`: 필터 UI
- `src/hooks/useProducts.ts`: React Query 훅
- `src/components/features/ProductFilter.test.tsx`: 테스트

TypeScript strict mode와 프로젝트 코딩 스타일을 준수합니다.
커밋 메시지는 "feat(products): Add category filter" 형식을 사용합니다.

? 진행할까요? (y/n)
```

### VS Code 및 IDE 통합

Claude Code는 터미널 도구이지만 IDE와 완벽하게 통합할 수 있습니다.

#### VS Code Extension

VS Code용 공식 확장을 설치하면 에디터 내에서 Claude의 변경사항을 visual diff로 볼 수 있습니다.

**설치:**

```bash
code --install-extension anthropic.claude-code
```

또는 VS Code 내에서:
1. Extensions 패널 열기 (Cmd/Ctrl + Shift + X)
2. "Claude Code" 검색
3. Install 클릭

**기능:**

**1. Visual Diff**

Claude가 파일을 수정할 때 변경사항이 diff 뷰로 표시됩니다:
- 초록색: 추가된 코드
- 빨간색: 삭제된 코드
- 노란색: 수정된 코드

각 변경사항을 개별적으로 수락하거나 거부할 수 있습니다.

**2. 인라인 채팅**

에디터에서 코드를 선택한 후:
1. 우클릭
2. "Ask Claude" 선택
3. 질문 입력

예: "이 함수를 최적화해줘" 또는 "버그를 찾아줘"

**3. 사이드 패널**

Claude와의 전체 대화 기록을 사이드 패널에서 확인:
- 이전 대화 검색
- 북마크 추가
- 대화 내보내기

**4. 단축키**

기본 단축키:
- `Cmd/Ctrl + K`: Claude Code 명령 실행
- `Cmd/Ctrl + Shift + L`: Claude 채팅 패널 열기
- `Cmd/Ctrl + Shift + A`: 선택한 코드에 대해 Claude에게 질문

커스터마이징:
1. Code → Preferences → Keyboard Shortcuts
2. "Claude" 검색
3. 원하는 단축키로 변경

#### Cursor 통합

Cursor는 VS Code의 포크로, AI 기능이 내장되어 있습니다. Claude Code와도 잘 작동합니다.

**설치:**

```bash
cursor --install-extension anthropic.claude-code
```

**특징:**

Cursor는 자체 AI 기능과 Claude Code를 함께 사용할 수 있습니다:
- Cursor의 Tab 자동완성
- Claude Code의 전체 프로젝트 이해
- 두 시스템의 시너지 효과

**사용 팁:**

- 간단한 자동완성: Cursor의 기본 기능 사용
- 복잡한 리팩토링: Claude Code 사용
- 버그 수정: 둘 다 시도해보고 더 나은 결과 선택

#### JetBrains IDEs

IntelliJ IDEA, PyCharm, WebStorm 등 JetBrains IDEs도 지원합니다.

**설치:**

1. Settings/Preferences 열기 (Cmd/Ctrl + ,)
2. Plugins 섹션
3. Marketplace 탭
4. "Claude Code" 검색
5. Install 버튼 클릭
6. IDE 재시작

**기능:**

**1. Terminal Integration**

JetBrains의 내장 터미널에서 바로 `claude` 명령 사용:
- View → Tool Windows → Terminal
- `claude` 입력

**2. Quick Actions**

에디터에서 코드 선택 후:
- Alt + Enter (Quick Fix 메뉴)
- "Ask Claude" 옵션 선택

**3. Diff Tool**

변경사항을 JetBrains의 강력한 diff 도구로 확인:
- 3-way merge
- 충돌 해결 도구
- 변경 이력 추적

### 팁과 트릭

Claude Code를 더 효과적으로 사용하기 위한 고급 팁들입니다.

#### 1. 효과적인 프롬프트 작성

**구체적으로 작성하기:**

나쁜 예:
```
버그 고쳐줘
```

문제점: 어떤 버그인지, 어디에 있는지, 증상이 무엇인지 불명확합니다.

좋은 예:
```
login.js의 버그를 고쳐줘. 
현재 이메일 검증이 제대로 작동하지 않아서 
"test@" 같은 잘못된 형식의 이메일도 통과되고 있어.
validator.js 라이브러리를 사용해서 수정해줘.
```

효과: Claude가 정확히 무엇을 해야 할지 알 수 있습니다.

**컨텍스트 제공하기:**

나쁜 예:
```
API 엔드포인트 만들어줘
```

좋은 예:
```
사용자 프로필을 업데이트하는 PUT /api/users/:id 엔드포인트를 만들어줘.

요구사항:
- JWT 인증 필요 (기존 auth 미들웨어 재사용)
- 업데이트 가능 필드: email, name, bio (비밀번호 제외)
- Joi로 입력 검증
- 성공 시 200과 업데이트된 사용자 정보 반환
- 에러 시 적절한 HTTP 상태 코드

기존 코드:
- src/middleware/auth.js 참조
- src/models/User.js의 스키마 준수
```

효과: 훨씬 정확하고 요구사항에 맞는 코드가 생성됩니다.

**예시 제공하기:**

```
새로운 API 응답 형식을 구현해줘.

기존 형식:
{
  "data": { ... },
  "message": "Success"
}

새 형식:
{
  "success": true,
  "data": { ... },
  "error": null,
  "metadata": {
    "timestamp": "2026-01-23T12:00:00Z",
    "version": "v1"
  }
}

모든 API 응답을 새 형식으로 변경해줘.
```

효과: 예시를 제공하면 Claude가 정확히 원하는 형태로 작업합니다.

#### 2. 여러 단계 작업 분해하기

복잡한 작업은 단계별로 나누어 요청하세요. Claude와 함께 점진적으로 발전시키는 것이 더 효과적입니다.

**예제:**

```
You: 단계별로 진행하자.
     먼저 사용자 인증 시스템의 전체 구조를 설명해줘.
     그다음 각 컴포넌트를 하나씩 구현할게.

Claude: 좋습니다. 단계별로 진행하겠습니다.

**전체 구조:**

1단계: 구조 설명 [지금]
2단계: JWT 유틸리티 구현
3단계: 로그인 엔드포인트
4단계: 회원가입 엔드포인트
5단계: 인증 미들웨어
6단계: 테스트 작성

**인증 시스템 아키텍처:**

[상세한 설명...]

현재 1단계 완료. 2단계(JWT 유틸리티 구현)로 진행할까요?
```

다음 단계:

```
You: 그래, 2단계 시작하자

Claude: JWT 유틸리티를 구현하겠습니다.

필요한 기능:
- generateAccessToken(payload)
- generateRefreshToken(payload)
- verifyToken(token)
- refreshAccessToken(refreshToken)

? 진행할까요? (y/n)
```

**장점:**

- 각 단계를 확인하고 조정할 수 있습니다
- 문제가 생기면 즉시 발견하고 수정
- 학습 기회: 각 단계의 이유를 이해
- 유연성: 중간에 방향 전환 가능

#### 3. Claude의 제안 활용하기

Claude는 종종 추가 개선 사항을 제안합니다. 이를 적극 활용하세요.

**예제:**

```
Claude: 로그인 엔드포인트를 구현했습니다.

✓ POST /api/auth/login 완성
✓ 이메일/비밀번호 검증
✓ JWT 토큰 생성
✓ 기본 테스트 작성

추가로 고려해볼 사항:
1. Rate limiting 추가 (브루트 포스 공격 방지)
2. 로그인 실패 시 로깅
3. 이메일 인증 기능
4. 2FA (Two-Factor Authentication) 옵션
5. 계정 잠금 기능 (5회 실패 시)

이 중 어떤 것을 구현할까요?
```

**대응:**

```
You: 1번과 5번을 먼저 구현하자. 
     보안이 중요하니까.

Claude: 좋습니다. Rate limiting과 계정 잠금 기능을 구현하겠습니다.

**1. Rate Limiting**
- express-rate-limit 사용
- 로그인: 15분에 5회 제한
- IP 기반 제한

**2. 계정 잠금**
- 5회 실패 시 30분 잠금
- User 모델에 failedLoginAttempts, lockedUntil 필드 추가
- 성공 로그인 시 카운터 리셋

? 진행할까요? (y/n)
```

#### 4. Claude에게 설명 요청하기

이해하기 어려운 코드를 만나면 Claude에게 설명을 요청하세요.

**예제:**

~~~
You: src/utils/crypto.js의 encrypt 함수를 초보자도 이해할 수 있게 설명해줘

Claude: crypto.js의 encrypt 함수를 단계별로 설명드리겠습니다.

**전체 코드:**
```javascript
function encrypt(text, secretKey) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', 
    Buffer.from(secretKey, 'hex'), iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return iv.toString('hex') + ':' + encrypted;
}
```

**단계별 설명:**

**1단계: IV (Initialization Vector) 생성**
```javascript
const iv = crypto.randomBytes(16);
```
- IV는 암호화할 때 사용하는 무작위 값입니다
- 같은 텍스트라도 매번 다른 결과를 만들어냅니다
- 16바이트(128비트) 크기
- 비밀로 유지할 필요 없음 (암호문과 함께 전송 가능)

**2단계: 암호화 객체 생성**
```javascript
const cipher = crypto.createCipheriv('aes-256-cbc', 
  Buffer.from(secretKey, 'hex'), iv);
```
- AES-256-CBC 알고리즘 사용
  - AES: 강력한 암호화 표준
  - 256: 키 길이 (매우 안전)
  - CBC: 블록 암호 모드
- secretKey: 64자 hex 문자열 (32바이트)
- iv: 위에서 생성한 무작위 값

**3단계: 텍스트 암호화**
```javascript
let encrypted = cipher.update(text, 'utf8', 'hex');
```
- `text`: 암호화할 원본 텍스트
- `'utf8'`: 입력 인코딩
- `'hex'`: 출력 인코딩 (16진수)
- 결과: 부분적으로 암호화된 데이터

**4단계: 암호화 마무리**
```javascript
encrypted += cipher.final('hex');
```
- 남은 데이터를 모두 암호화
- 3단계 결과에 추가

**5단계: 결과 반환**
```javascript
return iv.toString('hex') + ':' + encrypted;
```
- IV와 암호문을 콜론(:)으로 연결
- 예: "a1b2c3...f0:9x8y7z...1a"
- IV가 필요한 이유: 복호화할 때 동일한 IV 사용

**보안 팁:**
- secretKey는 절대 코드에 하드코딩하지 마세요
- 환경 변수에 저장: process.env.ENCRYPTION_KEY
- 키 길이: 정확히 64자 hex (32바이트)

**사용 예제:**
```javascript
const secretKey = process.env.ENCRYPTION_KEY;
const originalText = "my secret message";
const encrypted = encrypt(originalText, secretKey);
// 결과: "a1b2...f0:9x8y...1a"

const decrypted = decrypt(encrypted, secretKey);
// 결과: "my secret message"
```

더 궁금한 점이 있으신가요?
~~~

**활용 팁:**

- 복잡한 알고리즘 이해하기
- 레거시 코드 파악하기
- 코드 리뷰 준비
- 팀원에게 설명할 자료 만들기

#### 5. 프로젝트 컨텍스트 유지하기

대화형 모드에서 작업할 때, Claude는 프로젝트 컨텍스트를 유지합니다. 이를 활용하세요.

**예제:**

```
You: 회원가입 API를 구현했어?

Claude: 네, 이전 대화에서 src/auth/signup.js에 구현했습니다.

주요 기능:
- POST /api/auth/signup 엔드포인트
- 이메일/비밀번호 검증
- 비밀번호 bcrypt 해싱
- 중복 이메일 체크
- 자동 로그인 (JWT 토큰 반환)

테스트도 작성되어 있습니다.
확인해보시겠습니까?
```

이어서:

```
You: 그럼 이제 비밀번호 재설정 기능을 추가하자.
     기존 인증 시스템과 일관되게 구현해줘.

Claude: 비밀번호 재설정을 구현하겠습니다.
기존 인증 시스템의 패턴을 따르겠습니다:

- Joi 검증 (기존과 동일한 스키마)
- bcrypt 해싱 (10 rounds, 기존과 동일)
- JWT 토큰 (재설정 전용, 15분 유효)
- 이메일 전송 (nodemailer, 기존 설정 재사용)

파일 구조:
- src/auth/reset-password.js (새 파일)
- src/auth/reset-password-confirm.js (새 파일)
- src/email/templates/reset-password.html (새 파일)

기존 코드와의 일관성:
- 에러 처리 방식 동일
- 응답 형식 동일
- 로깅 패턴 동일

? 진행할까요? (y/n)
```

**장점:**

- 일관된 코드 스타일
- 중복 작업 방지
- 기존 패턴 재사용
- 통합 고려

---

## Google Antigravity 시작하기

### Antigravity란 무엇인가?

Google Antigravity는 Google이 2025년 하반기에 출시한 AI 기반 코딩 도구입니다. Gemini 3 Pro 모델을 활용하며, Claude Code와 유사하게 에이전트로 작동하여 개발 워크플로우를 자동화합니다.

Claude Code가 Anthropic의 대표 코딩 도구라면, Antigravity는 Google의 대표 코딩 도구입니다. 두 도구는 비슷한 철학을 공유하지만, 각자의 강점이 있습니다.

#### Antigravity의 특징

**1. Gemini 3 Pro 통합**

Antigravity는 Google의 최신 AI 모델인 Gemini 3 Pro를 사용합니다. Gemini 3 Pro는 다음 분야에서 특히 강력합니다:

**멀티모달 이해**: 
텍스트, 이미지, 오디오, 비디오를 동시에 처리할 수 있습니다. 예를 들어:
- 와이어프레임 이미지를 업로드하면 즉시 React 컴포넌트 생성
- 스케치 사진을 보고 HTML/CSS 코드 작성
- 동영상 튜토리얼을 분석하여 코드 구현

**긴 컨텍스트 윈도우**: 
2백만 토큰을 지원하여 매우 큰 코드베이스도 한 번에 이해합니다. 이는 Claude의 200K 토큰보다 10배 큰 크기입니다. 

실제로 이는 다음을 의미합니다:
- 대규모 모노레포 전체 분석 가능
- 수백 개 파일의 리팩토링 한 번에 처리
- 전체 프로젝트 히스토리 포함 작업

**실시간 웹 검색 통합**: 
Gemini는 실시간으로 웹을 검색할 수 있습니다. 최신 라이브러리 문서, 스택 오버플로우 솔루션, 최신 베스트 프랙티스를 자동으로 참조합니다.

**복잡한 추론 능력**: 
특히 수학적 계산, 알고리즘 최적화, 논리적 추론에서 뛰어납니다.

**2. Google 에코시스템과의 통합**

Antigravity는 Google의 다양한 서비스와 자연스럽게 통합됩니다:

**Google Cloud Platform (GCP)**:
- Cloud Functions 자동 배포
- Cloud Run 컨테이너 설정
- BigQuery 쿼리 최적화
- Cloud Storage 통합

**Google Drive**:
- 요구사항 문서 자동 읽기
- 디자인 파일 접근
- 코드 문서 자동 동기화

**Google Docs/Sheets**:
- API 명세서 읽기
- 데이터 시트에서 코드 생성
- 자동 문서화 업데이트

**Firebase**:
- Authentication 설정
- Firestore 스키마 생성
- Cloud Functions 구현
- Hosting 배포

**Google Search**:
- 최신 기술 트렌드 참조
- 에러 메시지 자동 검색
- 라이브러리 문서 찾기

**3. Stitch MCP 지원**

앞서 다룬 Stitch MCP Server와 통합하여 디자인에서 개발까지 완전한 파이프라인을 구축할 수 있습니다.

**워크플로우 예시:**

1. 디자이너가 Figma에서 디자인 완성
2. Stitch MCP가 디자인을 분석
3. Antigravity가 React 컴포넌트 생성
4. Firebase 백엔드 자동 설정
5. GCP에 배포

모든 과정이 자동화되며, 수동 개입은 최소화됩니다.

### 설치 방법

Antigravity 설치는 운영체제에 따라 다르지만, 전반적으로 Claude Code와 유사한 프로세스를 따릅니다.

#### macOS 설치

**Homebrew를 사용한 설치 (권장):**

```bash
brew install google-antigravity
antigravity --version
```

예상 출력:
```
Antigravity v1.8.2
Gemini 3 Pro
```

**공식 설치 스크립트:**

```bash
curl -fsSL https://antigravity.dev/install.sh | bash
```

설명:
- `curl -fsSL`: 안전하게 스크립트 다운로드
- 파이프(`|`)로 bash에 전달
- 자동으로 적절한 위치에 설치

설치 후 터미널 재시작:
```bash
source ~/.zshrc
# 또는
source ~/.bashrc
```

설치 확인:
```bash
antigravity --version
```

#### Linux 설치

**Ubuntu/Debian:**

```bash
curl -fsSL https://antigravity.dev/linux/ubuntu/install.sh | sudo bash
antigravity --version
```

**Fedora/RHEL:**

```bash
curl -fsSL https://antigravity.dev/linux/fedora/install.sh | sudo bash
antigravity --version
```

#### Windows 설치

**WinGet 사용:**

```powershell
winget install Google.Antigravity
```

설치 확인:
```powershell
antigravity --version
```

**공식 설치 프로그램:**

1. https://antigravity.dev/download 방문
2. Windows 설치 프로그램 다운로드
3. 실행 및 설치 마법사 따르기
4. PowerShell/명령 프롬프트 재시작

### 로그인 및 설정

Antigravity는 Google 계정으로 인증합니다.

```bash
antigravity login
```

브라우저가 자동으로 열리고 Google 로그인 페이지가 나타납니다.

절차:
1. Google 계정으로 로그인
2. Antigravity 권한 요청 확인
3. "Allow" 클릭
4. 인증 완료 메시지 확인

터미널로 돌아가면:

```
✓ Successfully authenticated as: your-email@gmail.com
✓ Gemini 3 Pro access confirmed
✓ Default project: my-first-project
✓ GCP integration: Enabled
✓ Firebase integration: Enabled

Ready to code!
```

**프로젝트 설정:**

Antigravity는 GCP 프로젝트와 연결됩니다.

기본 프로젝트 설정:
```bash
antigravity config set project my-project-id
```

현재 설정 확인:
```bash
antigravity config list
```

출력 예시:
```
Current Configuration:
- Project ID: my-project-id
- Region: us-central1
- Model: gemini-3-pro
- Budget: $100/month
```

### 기본 사용법

Antigravity도 Claude Code와 유사하게 대화형 모드와 명령 모드를 지원합니다.

#### 대화형 모드

```bash
antigravity
# 또는 약어
ag
```

프롬프트:

```
Antigravity v1.8.2 | Gemini 3 Pro
Project: my-project (your-project-id)
Connected to: Firebase, GCP

You:
```

**사용 예시:**

```
You: 현재 프로젝트의 TypeScript 에러를 모두 찾아서 수정해줘

Antigravity: TypeScript 프로젝트를 분석하고 있습니다...

✓ 전체 프로젝트 스캔 완료 (247 파일)
✓ 에러 17개 발견

**에러 분류:**
- Type mismatch: 8개
- Missing types: 5개
- Strict mode violations: 4개

가장 중요한 에러부터 수정하시겠습니까?
```

#### 일회성 명령

```bash
ag -p "현재 프로젝트에 ESLint 설정을 추가해줘"
```

**파이프 사용:**

```bash
git status | ag -p "변경된 파일들을 분석하고 커밋 메시지를 작성해줘"
```

### Antigravity의 독특한 기능

Antigravity는 Claude Code와 많은 기능을 공유하지만, Google 에코시스템과의 통합에서 독특한 강점을 보입니다.

#### 1. 멀티모달 입력

Antigravity의 가장 강력한 기능 중 하나는 이미지를 이해하는 능력입니다.

**와이어프레임에서 코드로:**

```bash
ag

You: 첨부한 와이어프레임을 보고 React 컴포넌트를 만들어줘
     [wireframe.png 업로드]

Antigravity: 와이어프레임을 분석하고 있습니다...

**디자인 분석:**

레이아웃:
- Header (로고 왼쪽, 네비게이션 오른쪽)
- Hero Section (큰 제목, 서브타이틀, 2개 버튼)
- Feature Cards (3컬럼 그리드)
- Footer (소셜 미디어, 링크)

색상 팔레트:
- Primary: #2563EB (파란색)
- Secondary: #64748B (회색)
- Background: #F8FAFC (밝은 회색)

컴포넌트 구조:
1. Header.jsx
2. Hero.jsx
3. FeatureCard.jsx
4. Features.jsx (FeatureCard 그리드)
5. Footer.jsx
6. LandingPage.jsx (전체 조합)

Tailwind CSS로 구현하시겠습니까? (y/n)
```

승인 후:

```
✓ 프로젝트 구조 생성
✓ Tailwind 설정
✓ Header 컴포넌트 (반응형 네비게이션)
✓ Hero 컴포넌트 (그라데이션 배경)
✓ FeatureCard 컴포넌트 (호버 효과)
✓ Features 컴포넌트 (그리드 레이아웃)
✓ Footer 컴포넌트 (소셜 아이콘)
✓ LandingPage 조합

로컬 서버를 시작하시겠습니까? (npm run dev)
```

**스케치에서 UI로:**

손으로 그린 스케치도 인식할 수 있습니다:

```
You: 이 종이에 그린 스케치를 HTML/CSS로 만들어줘
     [photo of hand-drawn sketch 업로드]

Antigravity: 스케치를 분석하고 있습니다...

인식된 요소:
- 상단 바 (로고, 메뉴)
- 왼쪽 사이드바 (네비게이션 링크)
- 메인 콘텐츠 영역 (그리드 카드)
- 각 카드: 이미지 + 제목 + 설명

레이아웃: Flexbox (사이드바) + Grid (카드)

? HTML/CSS로 구현할까요? (y/n)
```

**디자인 파일 분석:**

Figma, Sketch, Adobe XD 파일도 처리할 수 있습니다 (Stitch MCP 통합 시):

```
You: Figma 링크의 디자인을 React 앱으로 구현해줘
     https://www.figma.com/file/...

Antigravity: [Stitch MCP 연동]
Figma 파일을 불러오고 있습니다...

✓ 13개 스크린 인식
✓ 67개 컴포넌트 발견
✓ 색상 시스템 추출
✓ 타이포그래피 스타일 추출

프로젝트 구조:
src/
├── components/
│   ├── Button/ (5 variants)
│   ├── Card/ (3 types)
│   ├── Input/ (4 states)
│   └── ...
├── screens/
│   ├── Home/
│   ├── Dashboard/
│   └── ...
└── theme/
    ├── colors.js
    └── typography.js

전체를 구현하시겠습니까? (y/n)
```

#### 2. 실시간 웹 검색

Gemini 3 Pro는 실시간으로 웹을 검색하여 최신 정보를 활용합니다.

**최신 라이브러리 사용:**

~~~
You: 최신 React 19의 새로운 기능들을 사용해서 
     Suspense와 Server Components로 데이터 페칭 예제를 만들어줘

Antigravity: [웹 검색 중...]

React 19 공식 문서를 참조했습니다.
(https://react.dev/blog/2026/01/react-19-release)

**React 19의 주요 변화:**
- Server Components (stable)
- Actions (자동 pending states)
- use() hook
- Improved Suspense
- Automatic batching improvements

다음 기능들을 사용한 예제를 만들겠습니다:

1. Server Component로 데이터 페칭
2. Suspense로 로딩 상태 처리
3. use() hook으로 프라미스 unwrap
4. Actions로 form 처리

? 시작할까요? (y/n)
~~~

**에러 해결:**

~~~
You: 이 에러를 해결해줘:
     "TypeError: Cannot read property 'map' of undefined"

Antigravity: [웹 검색: "Cannot read property map of undefined react"]

Stack Overflow와 React 문서를 참조했습니다.

**원인:**
data가 undefined이거나 배열이 아닙니다.

**일반적인 원인:**
1. API 응답이 아직 도착하지 않음
2. API 응답 구조가 예상과 다름
3. 초기값이 설정되지 않음

**해결 방법:**

현재 코드:
```javascript
{data.map(item => ...)}
```

개선된 코드:
```javascript
{data?.map(item => ...) || <div>Loading...</div>}
```

또는:
```javascript
{Array.isArray(data) && data.map(item => ...)}
```

에러가 발생한 파일을 찾아서 수정하시겠습니까?
~~~

**최신 베스트 프랙티스:**

~~~
You: Next.js 14 앱을 최신 베스트 프랙티스에 맞게 리팩토링해줘

Antigravity: [웹 검색: "Next.js 14 best practices 2026"]

Next.js 공식 문서와 Vercel 블로그를 참조했습니다.

**2026년 Next.js 14 베스트 프랙티스:**

1. App Router 사용 (Pages Router 대신)
2. Server Components 기본 (Client Components 최소화)
3. Server Actions로 form 처리
4. Partial Prerendering (PPR) 활용
5. Next.js Image 최적화
6. Metadata API로 SEO
7. Route Handlers 대신 Server Actions

현재 프로젝트를 분석하고 개선점을 찾고 있습니다...

발견된 개선 기회:
- 32개 Pages Router 파일 → App Router 마이그레이션
- 18개 API Routes → Server Actions 변환
- 45개 불필요한 'use client' 제거 가능

마이그레이션을 시작하시겠습니까?
~~~

#### 3. Firebase 통합

Antigravity는 Firebase와 깊게 통합되어 있어, 백엔드 설정이 매우 간단합니다.

**Firebase 프로젝트 설정:**

~~~
You: Firebase 프로젝트를 설정하고 
     Authentication과 Firestore를 초기화해줘

Antigravity: Firebase 프로젝트를 생성하고 있습니다...

✓ Firebase 프로젝트 생성: my-app-c4f3e
✓ Firestore 데이터베이스 생성 (us-central1)
✓ Authentication 활성화:
  - Email/Password
  - Google Sign-In
  - GitHub (설정 필요)

✓ 프로젝트 파일 생성:
  - firebase.json
  - .firebaserc
  - firestore.rules
  - firestore.indexes.json

✓ SDK 설정:
  - src/lib/firebase.ts
  - 환경 변수 (.env.local)

**Firestore 보안 규칙:**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null 
        && request.auth.uid == userId;
    }
    
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth.uid == resource.data.authorId;
    }
  }
}
```

Firebase Console: https://console.firebase.google.com/project/my-app-c4f3e

다음 단계:
- Authentication UI 컴포넌트 생성
- Firestore CRUD 함수 작성
- Cloud Functions 설정 (선택사항)

어떤 것부터 진행할까요?
~~~

**Firestore 데이터 모델링:**

~~~
You: 블로그 앱을 위한 Firestore 스키마를 설계해줘.
     사용자, 포스트, 댓글, 좋아요 기능이 필요해.

Antigravity: Firestore 스키마를 설계하겠습니다.

**컬렉션 구조:**

```
firestore/
├── users/
│   └── {userId}/
│       ├── profile: { name, bio, avatar }
│       ├── posts/ (서브컬렉션)
│       └── following/ (서브컬렉션)
│
├── posts/
│   └── {postId}/
│       ├── title: string
│       ├── content: string
│       ├── authorId: string
│       ├── createdAt: timestamp
│       ├── likes: number
│       └── comments/ (서브컬렉션)
│           └── {commentId}/
│               ├── text: string
│               ├── authorId: string
│               └── createdAt: timestamp
│
└── likes/
    └── {likeId}/ (복합 키: userId_postId)
        ├── userId: string
        ├── postId: string
        └── createdAt: timestamp
```

**인덱스 설정:**
```json
{
  "indexes": [
    {
      "collectionGroup": "posts",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "authorId", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

**CRUD 함수 생성:**

TypeScript 함수를 작성하시겠습니까?
- createPost(userId, title, content)
- getPosts(limit, startAfter)
- updatePost(postId, updates)
- deletePost(postId)
- addComment(postId, userId, text)
- likePost(userId, postId)
- unlikePost(userId, postId)

? 생성할까요? (y/n)
~~~

**Cloud Functions 생성:**

~~~
You: 새 포스트가 작성될 때 알림을 보내는 Cloud Function을 만들어줘

Antigravity: Cloud Function을 작성하고 있습니다...

✓ functions/ 디렉토리 생성
✓ TypeScript 설정
✓ Firebase Admin SDK 설정

**함수 코드:**

```typescript
// functions/src/index.ts
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();

export const onPostCreated = functions.firestore
  .document('posts/{postId}')
  .onCreate(async (snap, context) => {
    const post = snap.data();
    const postId = context.params.postId;
    
    // 작성자의 팔로워 가져오기
    const followersSnapshot = await admin.firestore()
      .collection('users')
      .doc(post.authorId)
      .collection('followers')
      .get();
    
    // 알림 생성
    const notifications = followersSnapshot.docs.map(doc => ({
      userId: doc.id,
      type: 'new_post',
      postId: postId,
      authorId: post.authorId,
      read: false,
      createdAt: admin.firestore.FieldValue.serverTimestamp()
    }));
    
    // 배치로 알림 저장
    const batch = admin.firestore().batch();
    notifications.forEach(notification => {
      const notifRef = admin.firestore().collection('notifications').doc();
      batch.set(notifRef, notification);
    });
    
    await batch.commit();
    
    console.log(`Created ${notifications.length} notifications for post ${postId}`);
  });
```

✓ 함수 배포 준비 완료

배포하시겠습니까?
```bash
firebase deploy --only functions
```
~~~

### Claude Code vs Antigravity: 어떤 것을 선택할까?

두 도구 모두 훌륭하지만, 상황과 프로젝트 요구사항에 따라 선택이 달라질 수 있습니다.

#### Claude Code를 선택해야 하는 경우

**1. 코드 품질이 최우선일 때**

Claude Opus 4.5는 코드 생성 품질에서 업계 최고 수준입니다. 특히:
- 복잡한 비즈니스 로직
- 정교한 에러 처리
- 세밀한 엣지 케이스 고려
- 깔끔한 코드 구조

예: 금융 앱, 의료 시스템, 보안이 중요한 애플리케이션

**2. 복잡한 추론이 필요할 때**

Claude는 다음과 같은 작업에 탁월합니다:
- 아키텍처 설계 및 리팩토링
- 성능 최적화 전략
- 보안 취약점 분석
- 레거시 코드 이해 및 현대화

예: 대규모 모놀리식 앱을 마이크로서비스로 전환

**3. 문서 작성과 통합이 필요할 때**

Skills 시스템을 통한:
- Word 문서 (기술 문서, 제안서)
- PowerPoint 프레젠테이션 (투자자 피치)
- Excel 스프레드시트 (재무 모델)
- PDF 보고서

예: 클라이언트 제안서와 코드를 함께 작성해야 하는 프리랜서

**4. Anthropic 에코시스템을 사용할 때**

- Claude API 이미 사용 중
- Anthropic의 보안 정책 준수 필요
- AWS/GCP에서 Claude 모델 호스팅

**5. 엔터프라이즈 환경**

- 엄격한 보안 요구사항
- 온프레미스 배포 필요
- 감사 로그 및 모니터링 필수

#### Antigravity를 선택해야 하는 경우

**1. Google 에코시스템 중심 프로젝트**

다음을 많이 사용하는 경우:
- Firebase (Auth, Firestore, Functions, Hosting)
- Google Cloud Platform (Cloud Functions, Cloud Run, BigQuery)
- Google Workspace (Drive, Docs, Sheets)

예: Firebase 기반 모바일 앱, GCP 클라우드 네이티브 애플리케이션

**2. 멀티모달 입력이 중요할 때**

- 디자인 목업에서 코드 생성
- 스케치나 와이어프레임 활용
- 스크린샷 기반 버그 수정
- 이미지에서 데이터 추출

예: 디자인 중심 스타트업, 프로토타이핑 집약적 프로젝트

**3. 거대한 코드베이스**

Gemini 3 Pro의 2M 토큰 컨텍스트:
- 모노레포 전체 분석
- 수백 개 파일 동시 리팩토링
- 전체 프로젝트 히스토리 참조

예: 대기업의 레거시 시스템, 대규모 오픈소스 프로젝트

**4. 실시간 웹 정보가 필요할 때**

- 최신 라이브러리 버전 사용
- 최근 보안 패치 적용
- 새로운 프레임워크 기능 활용
- 현재 베스트 프랙티스 반영

예: 빠르게 변화하는 프론트엔드 생태계 프로젝트

**5. Firebase 백엔드가 필요할 때**

Firebase를 주요 백엔드로:
- 실시간 데이터베이스
- 서버리스 Functions
- 사용자 인증
- 파일 스토리지
- 푸시 알림

예: 소규모 스타트업 MVP, 개인 프로젝트, 빠른 프로토타입

#### 두 도구 모두 사용하기

많은 개발자들이 두 도구를 상황에 따라 함께 사용합니다:

**프로젝트 초기 (Claude Code)**:
- 아키텍처 설계
- 핵심 비즈니스 로직 구현
- 보안 중요 부분 작성

**개발 중반 (Antigravity)**:
- Firebase 백엔드 설정
- GCP 서비스 통합
- 이미지 기반 UI 구현

**테스트 및 배포 (Claude Code)**:
- 포괄적인 테스트 작성
- 성능 최적화
- 문서화

**유지보수 (둘 다)**:
- 버그 수정: 더 빠른 것 사용
- 기능 추가: 프로젝트 스택에 맞게 선택
- 리팩토링: Claude Code (더 깔끔한 결과)

**비용 고려**:

Claude Code:
- Pro: $20/월 (개인)
- API: 사용량 기반

Antigravity:
- 무료 티어: 월 1000 requests
- Pro: $15/월
- 엔터프라이즈: 커스텀

**성능 비교 (2026년 1월 기준)**:

| 항목 | Claude Code (Opus 4.5) | Antigravity (Gemini 3 Pro) |
|------|------------------------|----------------------------|
| 코드 품질 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 속도 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 컨텍스트 윈도우 | 200K | 2M |
| 멀티모달 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 문서 작성 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Firebase 통합 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| 가격 | $$$ | $$ |

**결론**:

어느 것이 "더 좋다"고 단정할 수 없습니다. 프로젝트 요구사항, 기존 인프라, 팀의 선호도에 따라 선택하세요. 두 도구를 함께 사용하는 것도 훌륭한 전략입니다.

---

## Tailwind CSS 기초

### Tailwind CSS란?

Tailwind CSS는 유틸리티 우선(Utility-First) CSS 프레임워크입니다. 전통적인 CSS 작성 방식과 완전히 다른 접근법을 취합니다.

전통적인 CSS 프레임워크(Bootstrap, Foundation 등)는 미리 만들어진 컴포넌트를 제공합니다. 버튼, 카드, 네비게이션 바 등을 그대로 가져다 쓰는 방식이죠. 하지만 Tailwind는 다릅니다. 작은 유틸리티 클래스들을 조합하여 원하는 디자인을 만듭니다.

#### 전통적인 CSS vs Tailwind CSS

**전통적인 방식:**

```html
<div class="card">
  <h2 class="card-title">제목</h2>
  <p class="card-description">설명</p>
</div>
```

별도의 CSS 파일:

```css
.card {
  background-color: white;
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 8px;
}

.card-description {
  color: #6b7280;
  line-height: 1.5;
}
```

**문제점:**
- CSS 파일과 HTML을 계속 왔다갔다 해야 함
- 클래스 이름 짓기 고민 (BEM? SMACSS? OOCSS?)
- 중복 스타일 많음
- 사용하지 않는 CSS가 계속 쌓임

**Tailwind 방식:**

```html
<div class="bg-white rounded-lg p-6 shadow-sm">
  <h2 class="text-2xl font-bold mb-2">제목</h2>
  <p class="text-gray-600 leading-relaxed">설명</p>
</div>
```

**장점:**
- 별도의 CSS 파일 필요 없음
- 모든 스타일이 HTML에서 바로 보임
- 클래스 이름 고민 불필요
- 일관된 디자인 시스템 자동 적용
- 사용하지 않는 CSS 자동 제거
- 빠른 개발 속도

**각 클래스 설명:**

- `bg-white`: 배경색 흰색
- `rounded-lg`: 둥근 모서리 (8px)
- `p-6`: 패딩 24px (모든 방향)
- `shadow-sm`: 작은 그림자
- `text-2xl`: 폰트 크기 24px
- `font-bold`: 폰트 굵기 700
- `mb-2`: 마진 하단 8px
- `text-gray-600`: 텍스트 색상 회색 (#6b7280)
- `leading-relaxed`: 행간 1.625

#### Tailwind의 장점

**1. 빠른 개발 속도**

CSS 파일을 왔다 갔다 할 필요 없이 HTML에서 바로 스타일링합니다. 실시간으로 결과를 확인하면서 조정할 수 있습니다.

개발자들의 경험담:
- "CSS 작성 시간이 70% 감소했습니다"
- "프로토타입을 30분 만에 만들 수 있었습니다"
- "디자인 변경이 너무 쉬워서 놀랐습니다"

**2. 일관성**

Tailwind는 디자인 시스템을 내장하고 있습니다:
- 색상: 50부터 950까지 체계적인 음영
- 간격: 4px 기반 8-point 그리드 시스템
- 폰트 크기: 일관된 타이포그래피 스케일
- 그림자: 5단계 깊이
- 브레이크포인트: 표준화된 반응형 기준

이로 인해 디자이너가 없어도 일관된 디자인을 유지할 수 있습니다.

**3. 반응형 디자인**

반응형 디자인이 매우 간단합니다:

```html
<div class="w-full md:w-1/2 lg:w-1/3">
  내용
</div>
```

설명:
- 모바일: 전체 너비 (w-full)
- 태블릿 (md, 768px 이상): 절반 너비
- 데스크톱 (lg, 1024px 이상): 1/3 너비

별도의 미디어 쿼리 작성 불필요!

**4. 작은 번들 크기**

Tailwind는 사용하지 않는 클래스를 자동으로 제거합니다 (PurgeCSS).

결과:
- 개발 시: 전체 프레임워크 (~3MB)
- 프로덕션: 실제 사용한 클래스만 (~10-20KB)

많은 프로젝트에서 최종 CSS가 10KB 미만입니다. 이는 Bootstrap의 1/20 크기입니다.

**5. 커스터마이징**

필요하면 설정 파일에서 모든 것을 커스터마이즈할 수 있습니다:

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        'brand-blue': '#1e40af',
        'brand-green': '#10b981',
      },
      spacing: {
        '128': '32rem',
      },
      fontFamily: {
        'custom': ['My Custom Font', 'sans-serif'],
      },
    },
  },
}
```

이제 `bg-brand-blue`, `p-128`, `font-custom` 같은 클래스를 사용할 수 있습니다.

### Tailwind CSS 설치

#### 새 프로젝트에 설치

**Vite + React 프로젝트:**

1단계: 프로젝트 생성

```bash
npm create vite@latest my-app -- --template react
cd my-app
```

설명:
- `vite@latest`: 최신 Vite 사용
- `--template react`: React 템플릿 선택

2단계: Tailwind 설치

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

설명:
- `-D`: 개발 의존성으로 설치
- `tailwindcss`: Tailwind 핵심
- `postcss`: CSS 변환 도구
- `autoprefixer`: 벤더 프리픽스 자동 추가
- `init -p`: 설정 파일 생성 (tailwind.config.js, postcss.config.js)

3단계: 설정 파일 수정

`tailwind.config.js` 열기:

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

중요: `content` 배열은 Tailwind가 스캔할 파일을 지정합니다.

4단계: CSS 지시문 추가

`src/index.css` 파일을 열고 다음을 추가:

```css
```

설명:
- `base`: 기본 스타일 (Preflight)
- `components`: 컴포넌트 클래스 (선택사항)
- `utilities`: 유틸리티 클래스 (핵심)

5단계: 설치 확인

```bash
npm install
npm run dev
```

`src/App.jsx`를 열고 테스트:

```jsx
export default function App() {
  return (
    <div className="flex items-center justify-center min-h-screen bg-gray-100">
      <h1 className="text-4xl font-bold text-blue-600">
        Hello Tailwind!
      </h1>
    </div>
  )
}
```

브라우저에서 확인: 파란색 큰 글자가 화면 중앙에 표시되어야 합니다.

**Next.js 프로젝트:**

Next.js는 Tailwind를 기본으로 지원합니다.

```bash
npx create-next-app@latest my-app --tailwind
```

설명:
- `--tailwind`: Tailwind가 자동으로 설정됨
- 추가 설정 불필요

또는 기존 Next.js 프로젝트에 추가:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

`app/globals.css`:

```css
```

#### 기존 프로젝트에 추가

이미 진행 중인 프로젝트에 Tailwind를 추가하는 경우:

1. Tailwind 설치:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

2. `tailwind.config.js` 설정:

```javascript
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}",
    "./public/index.html",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

주의: 프로젝트 구조에 맞게 `content` 경로를 조정하세요.

3. 메인 CSS 파일에 지시문 추가:

```css
```

4. 기존 CSS와의 충돌 해결:

Tailwind의 `base` 레이어가 기존 스타일을 재설정할 수 있습니다. 원하지 않으면:

```css
/* @tailwind base; 제거 또는 주석 처리 */
```

또는 특정 요소만 재설정:

```javascript
// tailwind.config.js
module.exports = {
  corePlugins: {
    preflight: false, // 기본 재설정 비활성화
  },
}
```

### Tailwind CSS 핵심 개념

Tailwind를 효과적으로 사용하려면 핵심 개념들을 이해해야 합니다.

#### 1. 유틸리티 클래스

Tailwind의 모든 클래스는 단일 CSS 속성을 나타냅니다.

**레이아웃:**

Display:

```html
<div class="block">블록 요소</div>
<div class="flex">Flexbox 컨테이너</div>
<div class="grid">Grid 컨테이너</div>
<div class="hidden">숨김</div>
```

설명:
- `block`: display: block
- `flex`: display: flex
- `grid`: display: grid
- `hidden`: display: none

Flexbox:

```html
<div class="flex justify-center items-center">
  가운데 정렬
</div>
```

설명:
- `justify-center`: justify-content: center (가로 중앙)
- `items-center`: align-items: center (세로 중앙)

Flexbox 방향:

```html
<div class="flex flex-col">수직 배치</div>
<div class="flex flex-row">수평 배치</div>
```

Flexbox 간격:

```html
<div class="flex gap-4">
  <div>항목 1</div>
  <div>항목 2</div>
</div>
```

설명: `gap-4`는 각 항목 사이에 16px 간격을 추가합니다.

Grid:

```html
<div class="grid grid-cols-3 gap-4">
  <div>항목 1</div>
  <div>항목 2</div>
  <div>항목 3</div>
</div>
```

설명:
- `grid-cols-3`: 3컬럼 그리드
- `gap-4`: 16px 간격

**간격 (Spacing)**

Tailwind는 4px 단위의 간격 시스템을 사용합니다.

```
0 = 0px
1 = 4px
2 = 8px
3 = 12px
4 = 16px
5 = 20px
6 = 24px
8 = 32px
10 = 40px
12 = 48px
16 = 64px
20 = 80px
24 = 96px
```

Padding 예제:

```html
<div class="p-4">패딩: 16px (모든 방향)</div>
<div class="px-6">패딩 좌우: 24px</div>
<div class="py-8">패딩 상하: 32px</div>
<div class="pt-2">패딩 상단: 8px</div>
<div class="pr-4">패딩 오른쪽: 16px</div>
<div class="pb-6">패딩 하단: 24px</div>
<div class="pl-8">패딩 왼쪽: 32px</div>
```

Margin 예제:

```html
<div class="m-4">마진: 16px (모든 방향)</div>
<div class="mx-auto">마진 좌우: auto (중앙 정렬)</div>
<div class="my-8">마진 상하: 32px</div>
<div class="-mt-4">음수 마진 상단: -16px (요소 겹치기)</div>
```

Gap (Grid/Flex):

```html
<div class="grid gap-6">그리드 간격: 24px</div>
<div class="flex gap-x-4 gap-y-8">
  가로 간격: 16px, 세로 간격: 32px
</div>
```

**크기 (Sizing)**

Width:

```html
<div class="w-full">너비: 100%</div>
<div class="w-1/2">너비: 50%</div>
<div class="w-1/3">너비: 33.333%</div>
<div class="w-64">너비: 256px (16rem)</div>
<div class="w-screen">너비: 100vw (뷰포트 전체)</div>
<div class="w-auto">너비: auto</div>
```

Height:

```html
<div class="h-screen">높이: 100vh (전체 화면)</div>
<div class="h-full">높이: 100%</div>
<div class="h-96">높이: 384px (24rem)</div>
<div class="h-auto">높이: auto</div>
```

Min/Max:

```html
<div class="min-w-0">최소 너비: 0</div>
<div class="max-w-md">최대 너비: 448px (28rem)</div>
<div class="max-w-screen-xl">최대 너비: 1280px</div>
<div class="max-w-prose">최대 너비: 65ch (읽기 좋은 너비)</div>

<div class="min-h-screen">최소 높이: 100vh</div>
<div class="max-h-96">최대 높이: 384px</div>
```

**색상 (Colors)**

Tailwind는 풍부한 색상 팔레트를 제공합니다. 각 색상은 50 (가장 밝음)부터 950 (가장 어두움)까지 음영을 가집니다.

Text Color:

```html
<p class="text-gray-500">회색 텍스트</p>
<p class="text-blue-600">파란색 텍스트</p>
<p class="text-red-700">빨간색 텍스트</p>
<p class="text-green-500">초록색 텍스트</p>
```

Background Color:

```html
<div class="bg-white">흰색 배경</div>
<div class="bg-gray-100">밝은 회색 배경</div>
<div class="bg-blue-500">파란색 배경</div>
<div class="bg-gradient-to-r from-blue-500 to-purple-600">
  그라데이션 배경
</div>
```

Border Color:

```html
<div class="border border-gray-300">회색 테두리</div>
<div class="border-2 border-blue-500">파란색 테두리 (2px)</div>
```

**타이포그래피 (Typography)**

Font Size:

```html
<p class="text-xs">12px</p>
<p class="text-sm">14px</p>
<p class="text-base">16px (기본)</p>
<p class="text-lg">18px</p>
<p class="text-xl">20px</p>
<p class="text-2xl">24px</p>
<p class="text-3xl">30px</p>
<p class="text-4xl">36px</p>
```

Font Weight:

```html
<p class="font-thin">100</p>
<p class="font-light">300</p>
<p class="font-normal">400 (기본)</p>
<p class="font-medium">500</p>
<p class="font-semibold">600</p>
<p class="font-bold">700</p>
<p class="font-extrabold">800</p>
```

Line Height (leading):

```html
<p class="leading-tight">1.25 (촘촘함)</p>
<p class="leading-normal">1.5 (기본)</p>
<p class="leading-relaxed">1.625 (여유)</p>
<p class="leading-loose">2 (넓음)</p>
```

Text Align:

```html
<p class="text-left">왼쪽 정렬</p>
<p class="text-center">중앙 정렬</p>
<p class="text-right">오른쪽 정렬</p>
<p class="text-justify">양쪽 정렬</p>
```

#### 2. 반응형 디자인

Tailwind는 모바일 우선(Mobile-First) 접근법을 사용합니다.

**브레이크포인트:**

기본 브레이크포인트:

- (없음): 0px 이상 (모든 화면)
- `sm:`: 640px 이상 (모바일 가로/작은 태블릿)
- `md:`: 768px 이상 (태블릿)
- `lg:`: 1024px 이상 (데스크톱)
- `xl:`: 1280px 이상 (큰 데스크톱)
- `2xl:`: 1536px 이상 (매우 큰 화면)

**사용 방법:**

```html
<div class="w-full md:w-1/2 lg:w-1/3 2xl:w-1/4">
  반응형 너비
</div>
```

동작:
- 모바일 (0-767px): 전체 너비 (100%)
- 태블릿 (768-1023px): 절반 너비 (50%)
- 데스크톱 (1024-1535px): 1/3 너비 (33.33%)
- 큰 화면 (1536px+): 1/4 너비 (25%)

**폰트 크기:**

```html
<p class="text-sm md:text-base lg:text-lg xl:text-xl">
  반응형 텍스트
</p>
```

동작:
- 모바일: 14px
- 태블릿: 16px
- 데스크톱: 18px
- 큰 화면: 20px

**표시/숨김:**

```html
<div class="hidden lg:block">
  데스크톱에서만 표시
</div>

<div class="block lg:hidden">
  모바일/태블릿에서만 표시
</div>
```

**그리드 컬럼:**

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  반응형 그리드
</div>
```

동작:
- 모바일: 1컬럼
- 태블릿: 2컬럼
- 데스크톱: 3컬럼
- 큰 화면: 4컬럼

#### 3. 상태 변형자 (State Variants)

Tailwind는 호버, 포커스 등의 상태를 간단히 처리할 수 있습니다.

**Hover:**

```html
<button class="bg-blue-500 hover:bg-blue-700 text-white">
  호버 시 색상 변경
</button>
```

설명:
- 기본: 파란색 배경 (bg-blue-500)
- 호버: 더 어두운 파란색 (bg-blue-700)

**Focus:**

```html
<input class="border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200">
```

설명:
- 기본: 회색 테두리
- 포커스: 파란색 테두리 + 파란색 링(outline)

**Active:**

```html
<button class="active:scale-95">
  클릭 시 살짝 축소
</button>
```

**Disabled:**

```html
<button class="disabled:opacity-50 disabled:cursor-not-allowed">
  비활성화 시 반투명
</button>
```

**Group (부모-자식 상호작용):**

```html
<div class="group">
  <img class="group-hover:scale-110 transition">
  <p class="group-hover:text-blue-600">
    부모에 호버 시 함께 변경
  </p>
</div>
```

설명:
- `group`: 부모 요소 표시
- `group-hover:`: 부모에 호버 시 적용

**Peer (형제 상호작용):**

```html
<input type="checkbox" class="peer">
<div class="peer-checked:bg-blue-500">
  체크박스 선택 시 배경 변경
</div>
```

#### 4. 애니메이션 및 전환

**Transition:**

```html
<button class="transition duration-300 ease-in-out hover:scale-105">
  부드러운 전환
</button>
```

설명:
- `transition`: 모든 변경사항을 애니메이션
- `duration-300`: 300ms 동안
- `ease-in-out`: 부드러운 easing
- `hover:scale-105`: 호버 시 5% 확대

**Transform:**

```html
<div class="hover:scale-110 hover:rotate-3">
  호버 시 확대 + 회전
</div>
```

Transform 종류:
- `scale-{value}`: 확대/축소
- `rotate-{degree}`: 회전
- `translate-x-{value}`: 가로 이동
- `translate-y-{value}`: 세로 이동
- `skew-x-{degree}`: 가로 기울임
- `skew-y-{degree}`: 세로 기울임

**내장 애니메이션:**

```html
<div class="animate-spin">회전</div>
<div class="animate-ping">펄스 (확대되며 사라짐)</div>
<div class="animate-bounce">튕김</div>
<div class="animate-pulse">깜빡임 (투명도 변화)</div>
```

### 실전 예제: 간단한 카드 컴포넌트

이제 배운 것들을 종합하여 세련된 카드 컴포넌트를 만들어봅시다.

```html
<div class="max-w-sm rounded-lg overflow-hidden shadow-lg hover:shadow-2xl transition-shadow duration-300">
  
  카드 이미지:
  <img 
    class="w-full h-48 object-cover" 
    src="https://via.placeholder.com/400x300" 
    alt="카드 이미지"
  >
  
  카드 내용:
  <div class="px-6 py-4">
    <div class="font-bold text-xl mb-2">제목</div>
    <p class="text-gray-700 text-base">
      이것은 Tailwind CSS로 만든 카드 컴포넌트입니다.
      별도의 CSS 파일 없이 HTML만으로 완성했습니다.
    </p>
  </div>
  
  카드 태그:
  <div class="px-6 pt-4 pb-2">
    <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2">
      #tailwind
    </span>
    <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2">
      #css
    </span>
    <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2 mb-2">
      #design
    </span>
  </div>
  
  카드 액션:
  <div class="px-6 pb-4">
    <button class="w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition-colors duration-200">
      자세히 보기
    </button>
  </div>
</div>
```

**설명:**

카드 컨테이너:
- `max-w-sm`: 최대 너비 384px
- `rounded-lg`: 둥근 모서리 8px
- `overflow-hidden`: 이미지가 카드 밖으로 나가지 않도록
- `shadow-lg`: 큰 그림자
- `hover:shadow-2xl`: 호버 시 더 큰 그림자
- `transition-shadow`: 그림자 변화를 부드럽게
- `duration-300`: 300ms 전환

이미지:
- `w-full`: 너비 100%
- `h-48`: 높이 192px
- `object-cover`: 비율 유지하며 영역 채우기

제목:
- `font-bold`: 굵은 글씨
- `text-xl`: 폰트 크기 20px
- `mb-2`: 마진 하단 8px

설명 텍스트:
- `text-gray-700`: 회색 텍스트
- `text-base`: 기본 폰트 크기 16px

태그:
- `inline-block`: 인라인 블록 (줄바꿈 가능)
- `bg-gray-200`: 밝은 회색 배경
- `rounded-full`: 완전히 둥근 모서리
- `px-3 py-1`: 가로 12px, 세로 4px 패딩
- `mr-2 mb-2`: 오른쪽과 하단 마진

버튼:
- `w-full`: 전체 너비
- `bg-blue-500`: 파란색 배경
- `hover:bg-blue-700`: 호버 시 어두운 파란색
- `transition-colors`: 색상 변화 애니메이션
- `duration-200`: 200ms

---

## Alexandru.so Snippets 활용법

### Alexandru.so Snippets란?

[https://snippets.alexandru.so](https://snippets.alexandru.so)는 Alexandru Pondorasti가 만든 **Tailwind CSS 전용 인터랙션 및 애니메이션 스니펫 모음**입니다.

**핵심 특징:**

1. **JavaScript 완전 불필요** - 순수 Tailwind CSS만 사용
2. **우클릭으로 코드 복사** - 즉시 프로젝트에 적용 가능
3. **프로덕션 레디** - 실제 프로젝트에서 바로 사용 가능
4. **브라우저 호환성** - 모던 브라우저 완벽 지원

### 왜 Alexandru.so Snippets인가?

**빠른 개발:**
- 복잡한 JavaScript 작성 불필요
- 페이지 로딩 속도 향상
- 유지보수 간편

**학습 자료:**
- Tailwind의 고급 기능 학습
- transform, transition, animation 마스터
- 실전 패턴 습득

### 주요 스니펫 카테고리

#### 1. Hover 인터랙션

방향성 있는 애니메이션으로 사용자 경험을 향상시킵니다.

**버튼 이동 효과:**

```html
<button class="relative inline-flex items-center justify-center px-6 py-3 overflow-hidden font-medium transition-all bg-blue-500 rounded-lg hover:bg-white group">
  <span class="absolute inset-0 border-0 group-hover:border-[25px] ease-linear duration-100 transition-all border-white rounded-lg"></span>
  <span class="relative w-full text-left text-white transition-colors duration-200 ease-in-out group-hover:text-blue-500">
    Hover over me
  </span>
</button>
```

동작:
- 기본: 파란색 버튼, 흰색 텍스트
- 호버: 흰색 배경, 파란색 텍스트
- 효과: 테두리가 안쪽에서 확장

**이미지 줌 효과:**

```html
<div class="relative overflow-hidden rounded-lg group">
  <img 
    src="image.jpg" 
    alt="Image"
    class="w-full h-64 object-cover transition-transform duration-300 group-hover:scale-110"
  >
  <div class="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-30 transition-opacity duration-300"></div>
</div>
```

동작:
- 기본: 일반 이미지
- 호버: 이미지 110% 확대 + 어두운 오버레이

#### 2. Text Shimmer Effect

로딩 상태나 강조 문구에 효과적입니다.

**구현:**

1단계: HTML

```html
<h1 class="inline-block text-6xl font-bold">
  <span class="bg-gradient-to-r from-blue-500 via-purple-500 to-blue-500 bg-[length:200%_auto] bg-clip-text text-transparent animate-shimmer">
    Text Shimmer Effect
  </span>
</h1>
```

2단계: tailwind.config.js에 애니메이션 추가

```javascript
module.exports = {
  theme: {
    extend: {
      animation: {
        shimmer: 'shimmer 3s linear infinite',
      },
      keyframes: {
        shimmer: {
          '0%': { backgroundPosition: '200% center' },
          '100%': { backgroundPosition: '-200% center' },
        },
      },
    },
  },
}
```

동작:
- 그라데이션이 좌우로 이동하며 반짝임
- 3초마다 반복
- 마케팅 페이지, 히어로 섹션에 적합

#### 3. Input / Typing 인터랙션

**플로팅 라벨 Input:**

```html
<div class="relative">
  <input 
    type="text" 
    id="email"
    class="peer w-full px-4 py-3 border-2 border-gray-300 rounded-lg focus:border-blue-500 focus:outline-none transition-colors"
    placeholder=" "
  >
  <label 
    for="email"
    class="absolute left-4 top-3 text-gray-500 transition-all peer-placeholder-shown:top-3 peer-placeholder-shown:text-base peer-focus:top-0 peer-focus:text-xs peer-focus:text-blue-500 peer-focus:bg-white peer-focus:px-1"
  >
    Email Address
  </label>
</div>
```

동작:
- 빈 칸: 라벨이 input 안에 위치
- 포커스/입력: 라벨이 위로 이동하며 작아짐

**타이핑 커서 효과:**

```html
<div class="font-mono text-2xl">
  <span>Type inside me</span>
  <span class="inline-block w-0.5 h-6 bg-black animate-pulse ml-1"></span>
</div>
```

동작:
- 깜빡이는 커서 효과
- 타이핑 중임을 시각적으로 표현

### Claude Code로 Snippets 통합하기

Claude Code를 사용하면 스니펫 통합이 매우 간단합니다.

```bash
claude

You: Alexandru.so의 Text Shimmer Effect를 
     랜딩 페이지 Hero 섹션에 적용해줘.
     제목은 "AI로 10배 빠른 개발"이고, 
     그라데이션은 파란색-보라색-파란색으로 해줘.

Claude: Text Shimmer Effect를 적용하겠습니다.

✓ tailwind.config.js에 shimmer 애니메이션 추가
✓ Hero 컴포넌트 생성
✓ 그라데이션 색상 blue-purple-blue로 설정

완료! 개발 서버를 시작하시겠습니까?
```

---

## 실전 프로젝트: 인터랙티브 랜딩 페이지

실제 작동하는 프로덕션급 랜딩 페이지를 만들어봅시다.

### 프로젝트 개요

**목표:** AI 코딩 도구 "CodeAI"를 홍보하는 랜딩 페이지

**기능:**
- Hero 섹션 (Text Shimmer)
- Features 섹션 (3D Card)
- Pricing 섹션 (Hover 효과)
- 반응형 디자인
- 다크모드 지원

**기술:**
- Vite + React
- Tailwind CSS
- Alexandru.so Snippets

### 1단계: 프로젝트 초기화

Claude Code 사용:

```
You: Vite + React + Tailwind CSS로 프로젝트 생성하고
     Alexandru.so 스니펫 사용을 위한 애니메이션을 
     tailwind.config.js에 추가해줘

Claude: 
✓ Vite 프로젝트 생성
✓ Tailwind 설치 및 설정
✓ 커스텀 애니메이션 추가
✓ 기본 컴포넌트 구조

프로젝트 준비 완료!
```

### 2단계: Hero 섹션

코드 예시는 이전과 동일하나, 설명을 간략화:

- Text Shimmer로 메인 타이틀
- 그라데이션 배경 애니메이션
- CTA 버튼 (Hover 효과)
- 반응형 레이아웃

### 3-6단계: 나머지 섹션

Features, Pricing, CTA 섹션은 동일한 패턴으로 구현합니다.

핵심은:
- Tailwind 유틸리티 클래스만 사용
- JavaScript 최소화
- 반응형 디자인 필수
- 접근성 고려

---

## 고급 테크닉

### 다크모드 구현

```javascript
export default {
  darkMode: 'class',
}
```

```jsx
<div className="bg-white dark:bg-gray-900">
  <h1 className="text-gray-900 dark:text-white">제목</h1>
</div>
```

### 커스텀 플러그인

```javascript
const plugin = require('tailwindcss/plugin')

export default {
  plugins: [
    plugin(function({ addComponents }) {
      addComponents({
        '.btn-primary': {
          '@apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded': {},
        },
      })
    })
  ],
}
```

---

## 문제 해결

**Tailwind 클래스가 적용되지 않음:**

원인: content 경로 설정 오류

해결:
```javascript
content: [
  "./index.html",
  "./src/**/*.{js,ts,jsx,tsx}",
],
```

**다크모드 작동 안 함:**

해결:
```javascript
darkMode: 'class',
```

```jsx
document.documentElement.classList.add('dark')
```

---

## 베스트 프랙티스

1. **컴포넌트 추출** - 반복 패턴은 컴포넌트로
2. **디자인 토큰** - 일관된 색상/간격 사용
3. **성능 최적화** - PurgeCSS 자동 활용
4. **접근성** - ARIA 레이블, 키보드 네비게이션

---

## 리소스 및 참고 자료

### 공식 문서
- [Claude Code](https://code.claude.com/docs)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Alexandru.so Snippets](https://snippets.alexandru.so)

### 학습 자료
- Tailwind CSS Tutorial
- Claude Code Quickstart
- Tailwind UI Components

### 커뮤니티
- Claude Code Discord
- Tailwind CSS Discord
- r/tailwindcss

---

## 결론

이 가이드를 통해 AI 기반 현대 웹 개발의 모든 것을 배웠습니다.

**핵심 요약:**

**AI 도구 활용:**
Claude Code와 Antigravity는 개발 속도를 10배 향상시킵니다.

**Tailwind CSS:**
빠르고 일관된 UI 개발이 가능합니다.

**실전 경험:**
실제 프로젝트를 만들며 학습하는 것이 가장 효과적입니다.

**지속적 학습:**
AI와 웹 개발 도구는 빠르게 진화합니다. 커뮤니티에 참여하고 새로운 기능을 실험하세요.

이제 여러분의 아이디어를 현실로 만들 준비가 되었습니다!

---

**작성일자:** 2026-01-23
