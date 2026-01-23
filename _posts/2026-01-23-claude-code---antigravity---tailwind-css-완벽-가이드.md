---
title: "Claude Code + Antigravity + Tailwind CSS 완벽 가이드"
date: 2026-01-23 06:20:00 +0900
categories: [AI,  Guide]
mermaid: [True]
tags: [AI,  Guide,  Antigravity,  claude-code,  Tailwind-CSS,  Claude.write]
---


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

**초보 개발자**: 코딩을 처음 배우는 분들도 AI의 도움으로 전문가 수준의 웹 애플리케이션을 만들 수 있습니다. 더 이상 수백 시간의 학습이 필요하지 않습니다.

**경험 많은 개발자**: AI를 활용하여 생산성을 10배 이상 높이고, 반복적인 작업에서 벗어나 창의적인 문제 해결에 집중할 수 있습니다.

**디자이너 및 기획자**: 코딩 지식 없이도 자신의 아이디어를 실제 작동하는 프로토타입으로 만들 수 있습니다.

**스타트업 창업자**: MVP를 빠르게 개발하고 시장 검증을 받을 수 있습니다. 몇 주가 걸리던 작업이 몇 시간으로 단축됩니다.

### 이 가이드에서 배울 내용

이 가이드를 따라가면 다음과 같은 능력을 갖추게 됩니다.

**Claude Code 마스터하기**: 터미널에서 Claude와 대화하며 코드를 작성하고, 파일을 편집하고, Git을 관리하는 방법을 배웁니다. Claude Code는 단순한 코드 작성 도구가 아니라 전체 개발 워크플로우를 자동화하는 에이전트입니다.

**Antigravity 활용하기**: Google의 AI 코딩 도구인 Antigravity를 사용하여 Gemini 3 Pro의 강력한 추론 능력을 활용하는 방법을 익힙니다. Stitch MCP와 통합하여 디자인에서 개발까지 전체 파이프라인을 구축합니다.

**Tailwind CSS 완전 정복**: 유틸리티 우선(Utility-First) CSS 프레임워크인 Tailwind CSS를 사용하여 빠르고 반응형이며 아름다운 UI를 만드는 방법을 배웁니다. Alexandru.so의 Snippets를 활용하여 JavaScript 없이도 매력적인 인터랙션을 구현합니다.

**실전 프로젝트 완성**: 이론만이 아닌 실제로 작동하는 랜딩 페이지, 대시보드, SaaS 애플리케이션을 처음부터 끝까지 만들어봅니다. 각 단계마다 상세한 설명과 함께 진행합니다.

### 학습 시간과 필요한 사전 지식

**예상 학습 시간**: 
- 기초 섹션 (1-4장): 2-3시간
- 실전 프로젝트 (5-6장): 3-4시간
- 고급 섹션 (7-9장): 2-3시간
- 총 7-10시간

**사전 지식**:
- 기본적인 컴퓨터 사용 능력
- HTML/CSS의 기초적인 이해 (선택사항 - 없어도 따라갈 수 있습니다)
- 터미널 명령어 기초 (가이드에서 모두 설명합니다)

**필요한 도구**:
- 컴퓨터 (macOS, Linux, 또는 Windows)
- 인터넷 연결
- 텍스트 에디터 (VS Code 권장)
- 웹 브라우저 (Chrome 또는 Firefox 권장)

### 2026년 AI 코딩의 현재 상황

2026년 1월 현재, AI 코딩 도구들은 폭발적으로 성장하고 있습니다. 특히 주목할 만한 트렌드는 다음과 같습니다.

**"Vibe Coding"의 등장**: 전통적인 방식처럼 코드를 한 줄씩 작성하는 대신, 자연어로 원하는 것을 설명하면 AI가 전체 애플리케이션을 생성합니다. Anthropic의 엔지니어 Felix Rieseberg는 "우리가 원하는 것을 설명하고, Claude가 구현을 처리하고, 우리가 방향을 조정하는" 방식으로 Claude Cowork를 만들었다고 밝혔습니다.

**에이전트(Agent) 시대**: AI가 단순히 코드를 제안하는 것을 넘어, 스스로 에러를 수정하고, 테스트를 실행하고, PR을 제출하는 등 전체 워크플로우를 자율적으로 수행합니다. Ethan Mollick 교수는 "지난 한 달간 AI가 갑작스럽게 능력이 도약했다"고 평가했습니다.

**개발자 커뮤니티의 열광**: 시애틀에서 열린 Claude Code 밋업에는 150명 이상의 엔지니어가 참석했고, AI 전설 Andrej Karpathy는 "프로그래머로서 이렇게 뒤처진 느낌을 받은 적이 없다"고 트윗했습니다. Google의 시니어 엔지니어 Jaana Dogan은 "Claude Code에 문제 설명을 주었더니 작년에 우리가 만든 것을 1시간 만에 생성했다"고 밝혔습니다.

**코드를 넘어선 활용**: "Claude Code"라는 이름이 오해를 불러일으킨다는 지적이 있습니다. 이 도구는 코딩뿐만 아니라 세금 신고 준비, 극장 티켓 예약, 스프레드시트 분석 등 컴퓨터로 할 수 있는 거의 모든 작업을 수행할 수 있습니다. The Transformer팀은 "비코더이지만 Claude Code로 은행 명세서를 정리하고 세금 초안을 준비했는데 모두 정확했다"고 보고했습니다.

이러한 변화는 단순한 도구의 발전이 아니라, 소프트웨어 개발의 패러다임 전환을 의미합니다. 이제 "아이디어에서 구현까지"의 시간이 몇 주에서 몇 시간으로 단축되고 있습니다.

---

## Claude Code 시작하기

### Claude Code란 무엇인가?

Claude Code는 Anthropic이 2026년 초에 출시한 에이전틱(agentic) 코딩 도구입니다. 가장 큰 특징은 **터미널에서 직접 작동한다**는 점입니다. 웹 인터페이스나 IDE 플러그인이 아닌, 여러분이 이미 사용하고 있는 터미널 환경에서 Claude와 대화하며 코드를 작성합니다.

#### 왜 Claude Code가 특별한가?

**1. 에이전트로서의 자율성**

Claude Code는 단순히 코드를 제안하는 것을 넘어, 실제로 행동을 취합니다. 파일을 직접 편집하고, 명령을 실행하고, Git 커밋을 만들고, 테스트를 실행하고, 에러를 스스로 수정합니다. 여러분은 "무엇을 원하는지"만 말하면 되고, Claude가 "어떻게 할지"를 알아서 처리합니다.

**2. 전체 코드베이스 이해**

Claude Code는 프로젝트의 전체 구조와 의존성을 몇 초 만에 파악합니다. 여러분이 수동으로 컨텍스트 파일을 선택할 필요가 없습니다. "에이전틱 검색(agentic search)"을 통해 관련 파일들을 자동으로 찾고 이해합니다.

**3. MCP (Model Context Protocol) 통합**

MCP를 통해 Claude Code는 Google Drive의 디자인 문서를 읽거나, Jira의 티켓을 업데이트하거나, 여러분의 커스텀 개발 도구를 사용할 수 있습니다. 이전 가이드에서 다룬 Stitch MCP도 여기에 해당합니다.

**4. Unix 철학에 충실**

Claude Code는 조합 가능하고 스크립트화할 수 있습니다. 예를 들어:
```bash
tail -f app.log | claude -p "로그 스트림에서 이상 징후가 보이면 Slack으로 알려줘"
```
이런 명령이 실제로 작동합니다. CI/CD 파이프라인에 통합하는 것도 가능합니다.

**5. 엔터프라이즈 지원**

Claude API를 사용하거나 AWS 또는 GCP에서 호스팅할 수 있습니다. 기업 환경에서도 안전하게 사용할 수 있도록 설계되었습니다.

### 설치 방법

Claude Code를 설치하는 방법은 운영체제에 따라 다릅니다.

#### macOS 설치

**방법 1: 공식 설치 프로그램 (권장)**

1. https://code.claude.com/download 에서 macOS 설치 프로그램을 다운로드합니다.
2. 다운로드한 `.dmg` 파일을 열고 Claude Code를 Applications 폴더로 드래그합니다.
3. 터미널을 열고 다음 명령을 실행하여 PATH에 추가합니다:
   ```bash
   # Claude Code를 PATH에 추가
   export PATH="/Applications/Claude Code.app/Contents/MacOS:$PATH"
   
   # 영구적으로 설정하려면 .zshrc나 .bashrc에 추가
   echo 'export PATH="/Applications/Claude Code.app/Contents/MacOS:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

**방법 2: Homebrew 사용**

Homebrew를 사용하면 설치와 업데이트가 간편합니다.

```bash
# Claude Code 설치
brew install --cask claude-code

# 설치 확인
claude --version
```

**참고**: Homebrew 설치는 자동 업데이트되지 않습니다. 주기적으로 업데이트하세요:
```bash
brew upgrade claude-code
```

#### Linux 설치

**Ubuntu/Debian 계열**:

```bash
# 공식 리포지토리 추가
curl -fsSL https://code.claude.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/claude-code.gpg
echo "deb [signed-by=/usr/share/keyrings/claude-code.gpg] https://code.claude.com/linux/ubuntu stable main" | sudo tee /etc/apt/sources.list.d/claude-code.list

# 패키지 업데이트 및 설치
sudo apt update
sudo apt install claude-code

# 설치 확인
claude --version
```

**Fedora/RHEL 계열**:

```bash
# RPM 리포지토리 추가
sudo dnf config-manager --add-repo https://code.claude.com/linux/fedora/claude-code.repo

# 설치
sudo dnf install claude-code

# 설치 확인
claude --version
```

#### Windows 설치

**방법 1: WinGet 사용 (Windows 11 권장)**

```powershell
# Claude Code 설치
winget install Anthropic.ClaudeCode

# 설치 확인
claude --version
```

**참고**: WinGet 설치는 자동 업데이트되지 않습니다. 주기적으로 업데이트하세요:
```powershell
winget upgrade Anthropic.ClaudeCode
```

**방법 2: 공식 설치 프로그램**

1. https://code.claude.com/download 에서 Windows 설치 프로그램을 다운로드합니다.
2. 다운로드한 `.exe` 파일을 실행하고 설치 마법사를 따릅니다.
3. 설치 완료 후 명령 프롬프트나 PowerShell을 **재시작**해야 `claude` 명령을 사용할 수 있습니다.

### 계정 설정 및 로그인

Claude Code를 처음 실행하면 로그인이 필요합니다.

```bash
# Claude Code 시작
claude
```

처음 실행하면 다음과 같은 프롬프트가 나타납니다:

```
Welcome to Claude Code!

To get started, please log in with your Claude account.

? How would you like to authenticate?
❯ Claude.ai account (Pro, Max, Teams, or Enterprise)
  Claude Console (API access)
  Cancel
```

**옵션 1: Claude.ai 계정 (권장)**

대부분의 사용자에게 권장되는 방법입니다. Claude Pro, Max, Teams, 또는 Enterprise 구독이 필요합니다.

1. "Claude.ai account" 선택
2. 브라우저가 자동으로 열리고 Claude.ai 로그인 페이지가 나타납니다
3. 계정으로 로그인하고 Claude Code 접근을 승인합니다
4. "Authentication successful!" 메시지가 표시되면 브라우저를 닫고 터미널로 돌아갑니다

**옵션 2: Claude Console (API 접근)**

API를 직접 사용하고 싶거나 프로그래매틱 접근이 필요한 경우 이 방법을 선택합니다.

1. "Claude Console" 선택
2. https://console.anthropic.com 에서 API 키를 생성합니다
3. API 키를 입력하라는 프롬프트가 나타나면 붙여넣습니다
4. API 키는 안전하게 저장되며, 매번 입력할 필요가 없습니다

**로그인 확인**:

```bash
# 현재 로그인 상태 확인
claude auth status

# 예상 출력:
# ✓ Authenticated as: your-email@example.com
# Subscription: Claude Pro
# Model access: Opus 4.5, Sonnet 4.5, Haiku 4.5
```

**로그아웃**:

필요시 다음 명령으로 로그아웃할 수 있습니다:
```bash
claude auth logout
```

### 기본 사용법

Claude Code는 크게 두 가지 모드로 사용할 수 있습니다: **대화형 모드**와 **일회성 명령 모드**입니다.

#### 대화형 모드 (Chat Mode)

대화형 모드는 Claude와 지속적으로 대화하며 작업하는 방식입니다.

```bash
# 대화형 모드 시작
claude

# 또는 특정 프로젝트 디렉토리에서
cd /path/to/your/project
claude
```

터미널에 Claude Code의 프롬프트가 나타납니다:

```
Claude Code v2.4.1 | Opus 4.5
Ready. What can I help you build?

You:
```

이제 자연어로 원하는 것을 입력하면 됩니다. 예를 들어:

```
You: 이 프로젝트의 구조를 설명해줘

Claude: 프로젝트 구조를 분석하고 있습니다...

이 프로젝트는 Express.js 기반의 웹 애플리케이션입니다. 구조는 다음과 같습니다:

project/
├── src/
│   ├── routes/        # API 라우트 정의
│   ├── controllers/   # 비즈니스 로직
│   ├── models/        # 데이터 모델 (MongoDB)
│   └── middleware/    # 인증, 에러 처리 등
├── public/            # 정적 파일 (CSS, JS, 이미지)
├── views/             # EJS 템플릿
├── tests/             # Jest 테스트
└── package.json       # 의존성 및 스크립트

주요 의존성:
- express: 웹 프레임워크
- mongoose: MongoDB ORM
- jsonwebtoken: JWT 인증
- bcrypt: 비밀번호 해싱

다음에 무엇을 도와드릴까요?
```

**대화형 모드의 장점**:
- 프로젝트 컨텍스트를 유지하며 연속적인 작업이 가능합니다
- 이전 대화를 참조할 수 있습니다
- 복잡한 다단계 작업에 적합합니다

**대화형 모드 종료**:
```
You: exit
# 또는 Ctrl+D
```

#### 일회성 명령 모드 (One-shot Mode)

간단한 작업이나 스크립트에서 사용하기 좋은 방식입니다.

```bash
# -p 또는 --prompt 옵션으로 명령 전달
claude -p "README.md 파일을 만들어줘. 프로젝트 설명과 설치 방법을 포함해야 해"

# 여러 줄 명령
claude -p "
이 프로젝트에 Docker 설정을 추가해줘:
1. Dockerfile 생성
2. docker-compose.yml 생성
3. .dockerignore 생성
4. README에 Docker 사용법 추가
"
```

**파이프와 결합**:

Claude Code는 Unix 철학을 따르므로 파이프와 결합할 수 있습니다.

```bash
# 로그 파일 모니터링 및 분석
tail -f app.log | claude -p "에러가 보이면 원인을 분석하고 Slack으로 알려줘"

# Git diff 리뷰
git diff | claude -p "이 변경사항을 리뷰하고 개선점을 제안해줘"

# 테스트 결과 요약
npm test | claude -p "테스트 결과를 요약하고 실패한 테스트를 자세히 설명해줘"
```

### Claude Code의 핵심 기능

#### 1. 파일 편집

Claude Code는 파일을 직접 생성하고 편집할 수 있습니다.

```bash
claude

You: src/utils 디렉토리에 date-formatter.js 파일을 만들어줘.
     날짜를 다양한 형식으로 포맷하는 유틸리티 함수들을 포함해야 해.

Claude: 파일을 생성하고 있습니다...

✓ Created src/utils/date-formatter.js

파일 내용:
[코드 미리보기 표시]

다음 작업을 제안합니다:
1. 이 유틸리티의 테스트 작성
2. JSDoc 문서 추가
3. README에 사용 예제 추가

어떤 것부터 진행할까요?
```

**파일 편집 승인**:

Claude는 파일을 변경하기 전에 항상 승인을 요청합니다 (기본 설정).

```
? Claude would like to create src/utils/date-formatter.js. Allow? (y/n)
```

자동 승인 모드를 원하면:
```bash
claude --auto-approve
```

하지만 보안을 위해 신중하게 사용해야 합니다.

#### 2. 명령 실행

Claude는 터미널 명령을 실행할 수 있습니다.

```
You: 프로젝트의 모든 의존성을 최신 버전으로 업데이트해줘

Claude: 다음 명령을 실행하겠습니다:
npm outdated
npm update
npm audit fix

? Execute these commands? (y/n)
```

승인하면 Claude가 명령을 실행하고 결과를 해석합니다.

```
✓ Executed: npm update
✓ Executed: npm audit fix

업데이트 완료:
- express: 4.17.1 → 4.18.2
- mongoose: 6.0.12 → 7.2.1
- jsonwebtoken: 8.5.1 → 9.0.0

2개의 보안 취약점이 수정되었습니다.
```

#### 3. Git 워크플로우

Claude Code는 Git을 완벽하게 이해하고 통합합니다.

```
You: 현재 변경사항을 커밋하고 feature/user-auth 브랜치에 푸시해줘

Claude: 다음 작업을 수행하겠습니다:
1. 변경된 파일 스테이징
2. 커밋 메시지 작성
3. feature/user-auth 브랜치로 푸시

변경된 파일:
- src/auth/login.js (modified)
- src/auth/signup.js (new)
- tests/auth.test.js (new)

제안하는 커밋 메시지:
feat: Add user authentication system

- Implement login and signup endpoints
- Add JWT token generation
- Include comprehensive test coverage

? Proceed with commit and push? (y/n)
```

#### 4. 프로젝트 이해 및 탐색

Claude Code의 강력한 기능 중 하나는 프로젝트 전체를 빠르게 이해하는 능력입니다.

```
You: 이 프로젝트에서 인증이 어떻게 구현되어 있는지 설명해줘

Claude: 프로젝트를 분석하고 있습니다...

인증 시스템 개요:

1. 전략: JWT (JSON Web Token) 기반 인증
   위치: src/auth/jwt.js

2. 주요 컴포넌트:
   - 로그인: src/auth/login.js
     → 이메일/비밀번호 검증
     → JWT 토큰 생성 및 반환
   
   - 회원가입: src/auth/signup.js
     → 사용자 생성
     → 비밀번호 bcrypt 해싱
   
   - 미들웨어: src/middleware/auth.js
     → JWT 검증
     → 보호된 라우트 접근 제어

3. 토큰 관리:
   - 액세스 토큰: 15분 유효
   - 리프레시 토큰: 7일 유효
   - 저장 위치: MongoDB users 컬렉션

4. 보안 고려사항:
   ✓ 비밀번호 해싱 (bcrypt, 10 rounds)
   ✓ HTTPS 전용 쿠키
   ✓ CORS 설정 있음
   ⚠ Rate limiting 없음 (추가 권장)

개선 제안을 원하시나요?
```

#### 5. 테스트 작성 및 실행

```
You: login.js의 유닛 테스트를 작성하고 실행해줘

Claude: 테스트를 작성하고 있습니다...

✓ Created tests/auth/login.test.js

테스트 케이스:
1. 유효한 자격증명으로 로그인 성공
2. 잘못된 비밀번호로 로그인 실패
3. 존재하지 않는 사용자로 로그인 실패
4. 입력 검증 (이메일 형식, 필수 필드)
5. JWT 토큰 생성 확인

테스트를 실행하시겠습니까? (y/n)

[Yes 선택 시]

✓ npm test tests/auth/login.test.js

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Snapshots:   0 total
Time:        2.341s

✅ 모든 테스트 통과!
```

### Claude Code Skills

Skills는 Claude Code의 가장 혁신적인 기능 중 하나입니다. Skill은 특정 작업을 수행하는 방법에 대한 지시사항과 도구의 조합입니다.

#### Skills의 개념

전통적인 AI 도구에서는 사용자가 매번 상세한 프롬프트를 제공해야 했습니다. "웹사이트를 만들려면 HTML 구조를 이렇게 하고, CSS는 저렇게 하고..." 식으로 말이죠. 

Skills는 이 문제를 해결합니다. Claude가 "웹사이트를 만들어야겠구나"하고 판단하면, 자동으로 Website Creator Skill을 로드합니다. 이 Skill에는 웹사이트를 만드는 모범 사례, 사용해야 할 도구, 체크리스트 등이 포함되어 있습니다.

#### 기본 제공 Skills

Claude Code는 다음과 같은 기본 Skills를 제공합니다:

**1. docx (문서 생성)**
- Microsoft Word 문서 작성, 편집, 형식 지정
- 추적된 변경사항, 댓글, 표 지원
- 용도: 보고서, 제안서, 문서 자동화

**2. pdf (PDF 처리)**
- PDF 생성, 편집, 텍스트 추출
- 양식 작성, 문서 병합/분할
- 용도: 계약서, 송장, 자동화된 보고서

**3. pptx (프레젠테이션)**
- PowerPoint 프레젠테이션 생성 및 편집
- 슬라이드 레이아웃, 차트, 애니메이션
- 용도: 비즈니스 프레젠테이션, 피치 덱

**4. xlsx (스프레드시트)**
- Excel 스프레드시트 작업
- 수식, 차트, 피벗 테이블
- 용도: 데이터 분석, 재무 모델링

**5. frontend-design (프론트엔드 디자인)**
- 고품질 UI/UX 생성
- React, HTML/CSS, Tailwind 지원
- 용도: 랜딩 페이지, 웹 애플리케이션

**6. product-self-knowledge (제품 지식)**
- Anthropic 제품 정보 참조
- API 사용법, 제한사항, 가격 정보
- 용도: Claude 관련 질문 답변

#### Skills 사용 예시

Skills는 자동으로 활성화되지만, 명시적으로 요청할 수도 있습니다.

```
You: 프론트엔드 디자인 스킬을 사용해서 
     미니멀한 랜딩 페이지를 만들어줘

Claude: [frontend-design Skill 활성화]

디자인 요구사항을 수집하고 있습니다...

1. 타겟 오디언스: B2B SaaS
2. 브랜드 컬러: 프로페셔널한 블루/화이트
3. 섹션: Hero, Features, Pricing, CTA
4. 스타일: 미니멀, 모던, 신뢰감

프레임워크 선택:
❯ React + Tailwind CSS (권장)
  HTML + Vanilla CSS
  Vue + Tailwind CSS

[React + Tailwind 선택 시]

✓ 프로젝트 구조 생성
✓ Tailwind 설정 완료
✓ Hero 섹션 컴포넌트 생성
✓ Features 섹션 컴포넌트 생성
...

랜딩 페이지가 완성되었습니다!
로컬 서버를 시작하시겠습니까? (npm run dev)
```

#### 커스텀 Skills 만들기

프로젝트에 특화된 Skill을 만들 수 있습니다. `CLAUDE.md` 파일을 프로젝트 루트에 생성하세요.

```markdown
# My Project Skill

## 프로젝트 개요
이 프로젝트는 전자상거래 플랫폼입니다.
Node.js + Express + MongoDB를 사용합니다.

## 코딩 스타일
- ES6+ 문법 사용
- async/await 선호 (Promise 체인 대신)
- 함수는 한 가지 일만 하도록 작성
- 주석은 "왜"를 설명 ("무엇"이 아니라)

## 테스트 규칙
- 모든 API 엔드포인트는 테스트가 있어야 함
- Jest + Supertest 사용
- 테스트 파일 이름: *.test.js

## 커밋 메시지 규칙
- feat: 새 기능
- fix: 버그 수정
- docs: 문서 변경
- test: 테스트 추가/수정
- refactor: 리팩토링

## 자주 사용하는 명령
- `npm run dev`: 개발 서버 시작
- `npm test`: 테스트 실행
- `npm run lint`: ESLint 실행
```

이제 Claude는 이 프로젝트에서 작업할 때 자동으로 이 가이드라인을 따릅니다.

### VS Code 및 IDE 통합

Claude Code는 터미널 도구이지만 IDE와 통합할 수 있습니다.

#### VS Code Extension

VS Code용 공식 확장을 설치하면 에디터 내에서 Claude의 변경사항을 visual diff로 볼 수 있습니다.

```bash
# VS Code Extension 설치
code --install-extension anthropic.claude-code
```

**사용 방법**:

1. VS Code에서 프로젝트 열기
2. 터미널 패널에서 `claude` 실행
3. Claude가 파일을 수정하면 에디터에 diff가 표시됩니다
4. 변경사항을 검토하고 수락하거나 거절할 수 있습니다

**단축키**:
- `Cmd/Ctrl + K`: Claude Code 명령 실행
- `Cmd/Ctrl + Shift + L`: Claude와 채팅 패널 열기

#### Cursor 통합

Cursor (VS Code 포크)도 지원됩니다.

```bash
# Cursor에서 Extension 설치
cursor --install-extension anthropic.claude-code
```

사용법은 VS Code와 동일합니다.

#### JetBrains IDEs

IntelliJ IDEA, PyCharm, WebStorm 등 JetBrains IDEs도 지원합니다.

1. Settings/Preferences → Plugins
2. "Claude Code" 검색 및 설치
3. 재시작 후 하단 터미널에서 `claude` 사용

### 팁과 트릭

#### 1. 효과적인 프롬프트 작성

**구체적으로 작성하기**:
```
❌ 나쁜 예: "버그 고쳐줘"
✅ 좋은 예: "login.js의 버그를 고쳐줘. 현재 이메일 검증이 제대로 작동하지 않아서 
   잘못된 형식의 이메일도 통과되고 있어."
```

**컨텍스트 제공하기**:
```
❌ 나쁜 예: "API 엔드포인트 만들어줘"
✅ 좋은 예: "사용자 프로필을 업데이트하는 PUT /api/users/:id 엔드포인트를 만들어줘.
   JWT 인증이 필요하고, 이메일과 이름만 업데이트 가능해야 해.
   기존의 auth 미들웨어를 재사용하고, Joi로 입력 검증을 해줘."
```

#### 2. 여러 단계 작업 분해하기

복잡한 작업은 단계별로 나누어 요청하세요.

```
You: 단계별로 진행하자.
     먼저 사용자 인증 시스템의 구조를 설명해줘.
     그다음 각 컴포넌트를 하나씩 구현할게.

Claude: 좋습니다. 단계별로 진행하겠습니다.

1단계: 구조 설명 [완료]
2단계: JWT 유틸리티 구현 [예정]
3단계: 로그인 엔드포인트 [예정]
4단계: 회원가입 엔드포인트 [예정]
5단계: 인증 미들웨어 [예정]
6단계: 테스트 작성 [예정]

현재 1단계 완료. 2단계로 진행할까요?
```

#### 3. Claude의 제안 활용하기

Claude는 종종 추가 작업을 제안합니다. 이를 적극 활용하세요.

```
Claude: 로그인 엔드포인트를 구현했습니다.

추가로 고려해볼 사항:
1. Rate limiting 추가 (브루트 포스 공격 방지)
2. 로그인 실패 로깅
3. 이메일 인증 기능
4. 2FA (Two-Factor Authentication) 옵션

이 중 어떤 것을 구현할까요?
```

#### 4. Claude에게 설명 요청하기

코드를 이해하기 어려울 때 Claude에게 설명을 요청하세요.

```
You: src/utils/crypto.js의 encrypt 함수를 초보자도 이해할 수 있게 설명해줘

Claude: crypto.js의 encrypt 함수를 단계별로 설명드리겠습니다.

[상세한 설명과 예시가 이어집니다...]
```

---

## Google Antigravity 시작하기

### Antigravity란 무엇인가?

Google Antigravity는 Google이 2025년 하반기에 출시한 AI 기반 코딩 도구입니다. Gemini 3 Pro 모델을 활용하며, Claude Code와 유사하게 에이전트로 작동하여 개발 워크플로우를 자동화합니다.

#### Antigravity의 특징

**1. Gemini 3 Pro 통합**

Antigravity는 Google의 최신 AI 모델인 Gemini 3 Pro를 사용합니다. Gemini 3 Pro는 특히 다음 분야에서 강력합니다:
- 멀티모달 이해 (텍스트, 이미지, 코드를 동시에 처리)
- 긴 컨텍스트 윈도우 (2백만 토큰)
- 실시간 웹 검색 통합
- 복잡한 추론 능력

**2. Google 에코시스템과의 통합**

Antigravity는 Google의 다양한 서비스와 자연스럽게 통합됩니다:
- Google Cloud Platform
- Google Drive
- Google Docs/Sheets
- Firebase
- Google Search

**3. Stitch MCP 지원**

앞서 다룬 Stitch MCP Server와 통합하여 디자인에서 개발까지 완전한 파이프라인을 구축할 수 있습니다.

### 설치 방법

Antigravity 설치는 운영체제에 따라 다릅니다.

#### macOS 설치

```bash
# Homebrew를 사용한 설치 (권장)
brew install google-antigravity

# 또는 공식 설치 스크립트
curl -fsSL https://antigravity.dev/install.sh | bash

# 설치 확인
antigravity --version
```

#### Linux 설치

```bash
# Ubuntu/Debian
curl -fsSL https://antigravity.dev/linux/ubuntu/install.sh | sudo bash

# Fedora/RHEL
curl -fsSL https://antigravity.dev/linux/fedora/install.sh | sudo bash

# 설치 확인
antigravity --version
```

#### Windows 설치

```powershell
# WinGet 사용
winget install Google.Antigravity

# 또는 설치 프로그램 다운로드
# https://antigravity.dev/download
```

### 로그인 및 설정

```bash
# Antigravity 로그인
antigravity login
```

브라우저가 열리고 Google 계정으로 로그인합니다.

```
✓ Successfully authenticated as your-email@gmail.com
✓ Gemini 3 Pro access confirmed
✓ Default project: my-first-project
```

### 기본 사용법

Antigravity도 Claude Code와 유사하게 대화형 모드와 명령 모드를 지원합니다.

#### 대화형 모드

```bash
# Antigravity 시작
antigravity

# 또는 약어
ag
```

프롬프트가 나타납니다:

```
Antigravity v1.8.2 | Gemini 3 Pro
Connected to: my-first-project

You:
```

#### 일회성 명령

```bash
# -p 또는 --prompt 옵션
ag -p "현재 프로젝트의 TypeScript 에러를 모두 찾아서 수정해줘"

# 파이프 사용
git status | ag -p "변경된 파일들을 분석하고 커밋 메시지를 작성해줘"
```

### Antigravity의 독특한 기능

#### 1. 멀티모달 입력

Antigravity는 이미지를 이해할 수 있습니다.

```bash
ag

You: 첨부한 와이어프레임을 보고 React 컴포넌트를 만들어줘
     [wireframe.png 업로드]

Antigravity: 와이어프레임을 분석하고 있습니다...

이 디자인은 다음 컴포넌트로 구성됩니다:
1. Header (로고, 네비게이션)
2. Hero Section (큰 제목, CTA 버튼)
3. Feature Cards (3컬럼 그리드)
4. Footer

Tailwind CSS로 구현하시겠습니까? (y/n)
```

#### 2. 실시간 웹 검색

Gemini 3 Pro는 실시간으로 웹을 검색할 수 있습니다.

```
You: 최신 React 18의 새로운 기능들을 사용해서 
     Suspense와 Concurrent Rendering을 적용한 예제를 만들어줘

Antigravity: [웹 검색 중...]

React 18의 최신 문서를 참조했습니다.
다음 기능들을 사용한 예제를 만들겠습니다:

1. Suspense for Data Fetching
2. Concurrent Rendering
3. Automatic Batching
4. startTransition API

[구현이 이어집니다...]
```

#### 3. Firebase 통합

Antigravity는 Firebase와 깊게 통합되어 있습니다.

```
You: Firebase 프로젝트를 설정하고 
     Authentication과 Firestore를 초기화해줘

Antigravity: Firebase CLI를 사용하여 설정하겠습니다.

✓ Firebase 프로젝트 생성: my-app-firebase
✓ Authentication 활성화 (이메일/비밀번호, Google)
✓ Firestore 데이터베이스 생성
✓ firebase.json 설정 완료
✓ src/config/firebase.js 생성

Firebase Console: https://console.firebase.google.com/project/my-app-firebase

다음 단계: 
- Authentication UI 컴포넌트 생성
- Firestore CRUD 함수 작성
어떤 것부터 진행할까요?
```

### Claude Code vs Antigravity: 어떤 것을 선택할까?

두 도구 모두 훌륭하지만, 상황에 따라 선택이 달라질 수 있습니다.

**Claude Code를 선택해야 하는 경우**:
- Anthropic Claude 모델 (Opus 4.5)의 우수한 코드 생성 품질을 원할 때
- 복잡한 추론과 설명이 필요한 작업
- Stitch MCP를 통한 UI 디자인 생성
- Unix 철학에 맞는 파이프라인 구축
- 엔터프라이즈 환경에서 AWS/GCP 호스팅

**Antigravity를 선택해야 하는 경우**:
- Google 에코시스템 (Firebase, GCP, Drive 등)과 통합이 필요할 때
- 이미지/멀티모달 입력이 중요한 프로젝트
- 실시간 웹 검색이 필요한 작업
- Gemini 3 Pro의 긴 컨텍스트 윈도우가 필요할 때
- Google Search 통합이 중요한 경우

**두 도구 모두 사용하기**:

많은 개발자들이 두 도구를 함께 사용합니다. 예를 들어:
- 프로젝트 초기 설계와 복잡한 로직: Claude Code
- Firebase 백엔드 설정과 Google 서비스 통합: Antigravity
- UI 디자인 생성: Claude Code + Stitch MCP
- 이미지 기반 프로토타이핑: Antigravity

---

## Tailwind CSS 기초

### Tailwind CSS란?

Tailwind CSS는 유틸리티 우선(Utility-First) CSS 프레임워크입니다. 전통적인 CSS 작성 방식과 완전히 다른 접근법을 취합니다.

#### 전통적인 CSS vs Tailwind CSS

**전통적인 방식**:

```html
<div class="card">
  <h2 class="card-title">제목</h2>
  <p class="card-description">설명</p>
</div>
```

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

**Tailwind 방식**:

```html
<div class="bg-white rounded-lg p-6 shadow-sm">
  <h2 class="text-2xl font-bold mb-2">제목</h2>
  <p class="text-gray-600 leading-relaxed">설명</p>
</div>
```

별도의 CSS 파일이 필요 없습니다! 모든 스타일이 HTML에 유틸리티 클래스로 표현됩니다.

#### Tailwind의 장점

**1. 빠른 개발 속도**

CSS 파일을 왔다 갔다 할 필요 없이 HTML에서 바로 스타일링합니다. 클래스 이름을 고민할 필요도 없습니다.

**2. 일관성**

Tailwind는 디자인 시스템을 내장하고 있습니다. 색상, 간격, 폰트 크기 등이 모두 사전 정의되어 있어 일관된 디자인을 자연스럽게 유지할 수 있습니다.

**3. 반응형 디자인**

반응형 디자인이 매우 간단합니다:

```html
<div class="w-full md:w-1/2 lg:w-1/3">
  <!-- 모바일: 전체 너비 -->
  <!-- 태블릿: 절반 너비 -->
  <!-- 데스크톱: 1/3 너비 -->
</div>
```

**4. 작은 번들 크기**

사용하지 않는 클래스는 자동으로 제거됩니다 (PurgeCSS/JIT 모드).

**5. 커스터마이징**

필요하면 설정 파일에서 모든 것을 커스터마이즈할 수 있습니다.

### Tailwind CSS 설치

#### 새 프로젝트에 설치

**Vite + React 프로젝트**:

```bash
# 프로젝트 생성
npm create vite@latest my-app -- --template react
cd my-app

# Tailwind 설치
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# 설치 확인
npm install
```

`tailwind.config.js` 파일 수정:

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

`src/index.css`에 Tailwind 지시문 추가:

```css
```

**Next.js 프로젝트**:

```bash
# Next.js 프로젝트 생성 (Tailwind 자동 포함)
npx create-next-app@latest my-app --tailwind

# 또는 기존 프로젝트에 추가
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

#### 기존 프로젝트에 추가

```bash
# Tailwind 설치
npm install -D tailwindcss postcss autoprefixer

# 설정 파일 생성
npx tailwindcss init -p
```

`tailwind.config.js` 설정 후 CSS 파일에 지시문을 추가하면 됩니다.

### Tailwind CSS 핵심 개념

#### 1. 유틸리티 클래스

Tailwind의 모든 클래스는 단일 CSS 속성을 나타냅니다.

**레이아웃**:
```html
