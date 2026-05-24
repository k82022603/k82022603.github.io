---
title: "Hermes Agent 실전 가이드: 설치부터 Docker 운영까지"
date: 2026-05-24 20:10:00 +0900
categories: [AI,  Hermes Agent]
mermaid: [True]
tags: [AI,  ai-agent,  hermes-agent,  Guide,  Claude.write]
---


> Nous Research 공식 문서를 바탕으로 작성한 Hermes Agent 설치, CLI 사용법, Docker 배포 완전 해설입니다.  
> 처음 설치하는 사용자부터 팀·서버 운영자까지 모두를 대상으로 합니다.

---

## 목차

1. [빠른 시작 가이드 (Quickstart)](#1-빠른-시작-가이드-quickstart)
2. [설치 (Installation)](#2-설치-installation)
3. [CLI 인터페이스 완전 해설](#3-cli-인터페이스-완전-해설)
4. [Docker로 운영하기](#4-docker로-운영하기)

---

## 1. 빠른 시작 가이드 (Quickstart)

### 1-1. 이 가이드가 필요한 사람

빠른 시작 가이드는 다음과 같은 상황에 있는 사람을 위해 작성되어 있습니다.

- 처음 설치하고 최단 경로로 작동하는 환경을 원하는 사용자
- 프로바이더를 전환하면서 설정 실수로 시간을 낭비하고 싶지 않은 사용자
- 팀용 봇이나 상시 동작 워크플로우를 구성하려는 사용자
- "설치는 됐는데 아무것도 안 된다"는 상황에서 벗어나고 싶은 사용자

### 1-2. 목표별 최단 경로

| 목표 | 첫 번째 할 일 | 그 다음 할 일 |
|---|---|---|
| 내 기기에서 Hermes를 일단 작동시키고 싶다 | `hermes setup` 실행 | 실제 채팅으로 응답 확인 |
| 사용할 프로바이더를 이미 알고 있다 | `hermes model` 실행 | 설정 저장 후 채팅 시작 |
| 봇 또는 상시 동작 환경을 원한다 | CLI 작동 확인 후 `hermes gateway setup` | Telegram, Discord, Slack 등 연결 |
| 로컬 또는 자체 호스팅 모델을 쓰고 싶다 | `hermes model` → 커스텀 엔드포인트 설정 | 엔드포인트, 모델명, 컨텍스트 길이 검증 |
| 멀티 프로바이더 폴백을 쓰고 싶다 | `hermes model` 먼저 | 기본 채팅이 작동한 후에만 라우팅·폴백 추가 |

> **핵심 원칙**: Hermes가 정상적인 채팅 한 번을 완료하지 못한다면 기능을 더 추가하지 마세요. 먼저 깨끗하게 대화 한 번이 성공해야 합니다. 그 다음에 게이트웨이, 크론, 스킬, 음성, 라우팅을 하나씩 추가합니다.

---

### 1-3. 1단계: Hermes Agent 설치

**방법 A — pip (가장 간단)**

```bash
pip install hermes-agent
hermes postinstall     # 선택: Node.js, 브라우저, ripgrep, ffmpeg 설치 + 설정 실행
```

PyPI 릴리즈는 tagged 버전(메이저/마이너 릴리즈)을 추적합니다. `main` 브랜치의 최신 커밋을 원한다면 방법 B를 사용하세요.

**방법 B — git 인스톨러 (main 브랜치 추적)**

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

설치 완료 후 셸을 다시 로드합니다.

```bash
source ~/.bashrc   # 또는 source ~/.zshrc
```

---

### 1-4. 2단계: 프로바이더 선택

설정 중 가장 중요한 단계입니다. `hermes model` 명령으로 대화형으로 선택할 수 있습니다.

```bash
hermes model
```

**가장 쉬운 경로: Nous Portal**

Nous Portal 구독 하나로 300개 이상의 모델과 Tool Gateway(웹 검색, 이미지 생성, TTS, 클라우드 브라우저)를 모두 사용할 수 있습니다. 신규 설치라면 다음 한 줄로 로그인, 프로바이더 설정, Tool Gateway 활성화를 한꺼번에 처리합니다.

```bash
hermes setup --portal
```

**주요 프로바이더 목록**

| 프로바이더 | 설명 | 설정 방법 |
|---|---|---|
| Nous Portal | 구독 기반, 설정 불필요 | `hermes model`에서 OAuth 로그인 |
| OpenAI Codex | ChatGPT OAuth, Codex 모델 사용 | `hermes model`에서 device code 인증 |
| Anthropic | Claude 모델 직접 연결 — Max 플랜 + 추가 크레딧(OAuth) 또는 API 키(종량제) | `hermes model` → OAuth 로그인, 또는 Anthropic API 키 |
| OpenRouter | 여러 프로바이더 라우팅 | API 키 입력 |
| DeepSeek | DeepSeek API 직접 접근 | `DEEPSEEK_API_KEY` 환경변수 설정 |
| GitHub Copilot | GitHub Copilot 구독 (GPT-5.x, Claude, Gemini 등) | `hermes model`에서 OAuth 또는 토큰 설정 |
| AWS Bedrock | Claude, Nova, Llama, DeepSeek via Converse API | IAM 역할 또는 `aws configure` |
| Hugging Face | 20개 이상 오픈 모델 (Qwen, DeepSeek, Kimi 등) | `HF_TOKEN` 환경변수 설정 |
| Custom Endpoint | vLLM, SGLang, Ollama, OpenAI 호환 API | base URL + API 키 설정 |
| Alibaba Cloud | Qwen 모델 (DashScope) | `DASHSCOPE_API_KEY` 설정 |
| NVIDIA NIM | Nemotron 모델 | `NVIDIA_API_KEY` 설정 |
| Kimi / Moonshot | Moonshot 코딩 및 채팅 모델 | `KIMI_API_KEY` 설정 |

> **최소 컨텍스트 요구사항**: Hermes Agent는 최소 **64,000 토큰** 컨텍스트를 지원하는 모델을 요구합니다. 더 작은 컨텍스트 윈도우를 가진 모델은 시작 시 거부됩니다. 로컬 모델을 사용한다면 컨텍스트 크기를 최소 64K로 설정하세요 (llama.cpp: `--ctx-size 65536`, Ollama: `-c 65536`).

**설정 저장 방식**

Hermes는 시크릿과 일반 설정을 분리해서 저장합니다.

- 시크릿과 토큰 → `~/.hermes/.env`
- 일반 설정 → `~/.hermes/config.yaml`

CLI를 통해 설정하면 올바른 파일에 자동으로 저장됩니다.

```bash
hermes config set model anthropic/claude-opus-4.6
hermes config set terminal.backend docker
hermes config set OPENROUTER_API_KEY sk-or-...
```

---

### 1-5. 3단계: 첫 번째 채팅 실행

```bash
hermes            # 클래식 CLI
hermes --tui      # 모던 TUI (권장)
```

시작하면 현재 모델, 사용 가능한 도구, 설치된 스킬을 보여주는 웰컴 배너가 표시됩니다. 검증하기 쉬운 구체적인 프롬프트로 테스트합니다.

```
이 저장소를 5개 항목으로 요약하고 메인 진입점이 무엇인지 알려줘.
현재 디렉토리를 확인하고 메인 프로젝트 파일이 무엇인지 알려줘.
```

**성공 기준**

- 배너에 선택한 모델/프로바이더가 표시됨
- 오류 없이 응답이 돌아옴
- 필요 시 도구(터미널, 파일 읽기, 웹 검색)를 사용할 수 있음
- 두 번 이상 연속 대화가 정상적으로 진행됨

---

### 1-6. 4단계: 세션 저장 확인

```bash
hermes --continue    # 가장 최근 세션 재개
hermes -c            # 단축형
```

이전 세션으로 돌아올 수 있어야 합니다. 안 된다면 같은 프로파일에 있는지, 세션이 실제로 저장됐는지 확인합니다. 여러 설정이나 여러 기기를 다룰 때 이 기능이 중요합니다.

---

### 1-7. 5단계: 주요 기능 체험

**터미널 사용**

```
디스크 사용량을 알려줘. 가장 큰 디렉토리 상위 5개를 보여줘.
```

에이전트가 터미널 명령을 대신 실행하고 결과를 보여줍니다.

**슬래시 명령어**

`/`를 입력하면 자동완성 드롭다운이 표시됩니다.

| 명령어 | 설명 |
|---|---|
| `/help` | 사용 가능한 모든 명령어 표시 |
| `/tools` | 현재 사용 가능한 도구 목록 |
| `/model` | 대화형으로 모델 전환 |
| `/personality pirate` | 성격 변경 체험 |
| `/save` | 대화 저장 |

---

### 1-8. 자주 발생하는 문제와 해결법

| 증상 | 가능한 원인 | 해결 방법 |
|---|---|---|
| Hermes는 열리지만 빈 응답 또는 오류 발생 | 프로바이더 인증 또는 모델 선택 문제 | `hermes model` 다시 실행하여 프로바이더·모델·인증 확인 |
| 커스텀 엔드포인트가 "작동"하지만 응답이 이상함 | 잘못된 base URL, 모델명, 또는 OpenAI 비호환 API | 별도 클라이언트로 엔드포인트를 먼저 검증 |
| 게이트웨이가 시작됐지만 메시지를 받지 못함 | 봇 토큰, 허용 목록, 플랫폼 설정 미완료 | `hermes gateway setup` 재실행 및 `hermes gateway status` 확인 |
| `hermes --continue`로 이전 세션을 찾지 못함 | 프로파일이 다르거나 세션이 저장 안 됨 | `hermes sessions list`로 세션 확인, 올바른 프로파일 여부 체크 |
| `hermes doctor`가 설정 문제 감지 | 설정 값 누락 또는 오래된 값 | 설정 수정 후 기본 채팅 재테스트 |

**복구 순서**

무언가 잘못됐을 때 이 순서대로 실행하면 대부분 해결됩니다.

```bash
hermes doctor
hermes model
hermes setup
hermes sessions list
hermes --continue
hermes gateway status
```

---

## 2. 설치 (Installation)

### 2-1. 플랫폼별 설치 방법

#### Linux / macOS / WSL2

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

#### Windows (네이티브, PowerShell) — 얼리 베타

> **주의**: 네이티브 Windows 지원은 얼리 베타 단계입니다. 공통 경로는 작동하지만 POSIX 인스톨러만큼 광범위하게 테스트되지 않았습니다. Windows에서 가장 안정적인 방법은 WSL2 내에서 위의 Linux 명령을 실행하는 것입니다.

```powershell
iex (irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1)
```

Windows 인스톨러가 처리하는 항목들입니다.

- **uv** (빠른 Python 패키지 매니저)
- **Python 3.11**
- **Node.js 22**
- **PortableGit** (자체 포함 Git-for-Windows 배포본 — bash.exe와 POSIX 툴체인 포함)
- 코드 클론 위치: `%LOCALAPPDATA%\hermes\hermes-agent`
- 가상 환경 생성 및 `hermes`를 User PATH에 추가

설치 완료 후 새 PowerShell 창을 열어야 PATH가 적용됩니다.

Git 처리 방식에 대해 알아두면 좋은 사항이 있습니다. 시스템에 git이 이미 있으면 기존 것을 사용하고, 없으면 PortableGit(~50MB)을 `%LOCALAPPDATA%\hermes\git`에 다운로드합니다. 관리자 권한 없이도 설치 가능하며 기존 시스템 Git에 영향을 주지 않습니다.

#### Android / Termux

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

인스톨러가 Termux 환경을 자동 감지해 Android 전용 설치 흐름으로 전환합니다. Termux `pkg`로 시스템 의존성을 설치하고, 브라우저/WhatsApp 부트스트랩은 기본적으로 건너뜁니다.

---

### 2-2. 설치 레이아웃

설치 방식에 따라 파일이 저장되는 위치가 다릅니다.

| 인스톨러 | 코드 위치 | hermes 바이너리 | 데이터 디렉토리 |
|---|---|---|---|
| pip install | Python site-packages | `~/.local/bin/hermes` | `~/.hermes/` |
| 일반 사용자 (git 인스톨러) | `~/.hermes/hermes-agent/` | `~/.local/bin/hermes` (심링크) | `~/.hermes/` |
| 루트 모드 (`sudo curl … \| sudo bash`) | `/usr/local/lib/hermes-agent/` | `/usr/local/bin/hermes` | `/root/.hermes/` |

루트 모드 FHS 레이아웃은 공유 머신 배포에 적합합니다. 한 번의 시스템 설치로 모든 사용자가 Hermes를 사용할 수 있고, 각 사용자의 설정은 각자의 `~/.hermes/`에 저장됩니다.

---

### 2-3. Windows 기능 지원 범위 (얼리 베타)

| 기능 | 지원 여부 |
|---|---|
| CLI (hermes chat, setup, gateway 등) | 네이티브 지원 |
| 게이트웨이 (Telegram, Discord, Slack 등) | 네이티브, PowerShell 백그라운드 프로세스로 실행 |
| 크론 스케줄러 | 네이티브 지원 |
| 브라우저 도구 (Playwright/Chromium) | 네이티브 지원 |
| MCP 서버 (stdio 및 HTTP) | 네이티브 지원 |
| 대시보드 `/chat` 터미널 탭 | **WSL2 전용** (POSIX PTY 필요) |

대시보드의 세션·잡·메트릭 부분은 네이티브로 작동하지만, 내장 PTY 터미널 탭만 WSL2가 필요합니다.

---

### 2-4. 인스톨러가 자동으로 처리하는 것들

인스톨러 한 번 실행으로 다음이 모두 자동 처리됩니다.

- Python 3.11 설치 (uv를 통해, sudo 불필요)
- Node.js v22 (브라우저 자동화 및 WhatsApp 브릿지용)
- ripgrep (빠른 파일 검색)
- ffmpeg (TTS용 오디오 형식 변환)
- 가상 환경 생성
- 전역 `hermes` 명령어 설정
- LLM 프로바이더 설정

> Git만 미리 설치되어 있으면 나머지는 모두 인스톨러가 알아서 처리합니다.

---

### 2-5. sudo 없는 설치 / 시스템 서비스 사용자 설치

권한이 없는 전용 사용자(예: `hermes` systemd 서비스 계정)로 실행하는 방법입니다. 유일하게 루트가 필요한 것은 Playwright의 `--with-deps` 단계로, Chromium이 사용하는 공유 라이브러리를 `apt`로 설치합니다.

**권장 방법 (Debian/Ubuntu)**

1단계: 관리자 권한으로 한 번만 실행합니다.

```bash
sudo npx playwright install-deps chromium
```

2단계: 권한 없는 서비스 사용자로 일반 인스톨러를 실행합니다. 인스톨러가 sudo 없음을 감지하고 `--with-deps` 없이 Chromium을 사용자 로컬 Playwright 캐시에 설치합니다.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

브라우저 자동화가 필요 없다면 `--skip-browser` 플래그로 Playwright 단계를 완전히 건너뛸 수 있습니다.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash -s -- --skip-browser
```

---

### 2-6. 문제 해결

| 문제 | 해결 방법 |
|---|---|
| `hermes: command not found` | 셸 재로드 (`source ~/.bashrc`) 또는 PATH 확인 |
| API 키 미설정 | `hermes model` 실행하거나 `hermes config set OPENROUTER_API_KEY your_key` |
| 업데이트 후 설정 누락 | `hermes config check` 후 `hermes config migrate` |

더 자세한 진단은 `hermes doctor` 명령을 실행하면 무엇이 빠졌고 어떻게 고치면 되는지 알려줍니다.

---

## 3. CLI 인터페이스 완전 해설

### 3-1. CLI 실행 방법

기본 사용법부터 다양한 옵션까지 알아봅니다.

```bash
# 인터랙티브 세션 시작 (기본)
hermes

# 단일 쿼리 모드 (비인터랙티브)
hermes chat -q "Hello"

# 특정 모델 지정
hermes chat --model "anthropic/claude-sonnet-4"

# 특정 프로바이더 지정
hermes chat --provider nous        # Nous Portal 사용
hermes chat --provider openrouter  # OpenRouter 강제

# 특정 툴셋 지정
hermes chat --toolsets "web,terminal,skills"

# 하나 이상의 스킬을 미리 로드하고 시작
hermes -s hermes-agent-dev,github-auth
hermes chat -s github-pr-workflow -q "draft PR 열어줘"

# 이전 세션 재개
hermes --continue             # 가장 최근 CLI 세션 재개 (-c)
hermes --resume <session_id>  # ID로 특정 세션 재개 (-r)

# 상세 모드 (디버그 출력)
hermes chat --verbose

# 격리된 git worktree (여러 에이전트 병렬 실행)
hermes -w                        # worktree에서 인터랙티브 모드
hermes -w -q "이슈 #123 고쳐줘"  # worktree에서 단일 쿼리
```

---

### 3-2. 인터페이스 구성

Hermes CLI는 두 가지 터미널 인터페이스를 제공합니다. 클래식 `prompt_toolkit` CLI와 모달 오버레이, 마우스 선택, 논블로킹 입력을 갖춘 최신 TUI가 있습니다. 두 인터페이스는 동일한 세션, 슬래시 명령어, 설정을 공유합니다.

**웰컴 배너**에는 현재 모델, 터미널 백엔드, 작업 디렉토리, 사용 가능한 도구, 설치된 스킬이 한눈에 표시됩니다.

---

### 3-3. 상태 바 (Status Bar)

입력 영역 위에 실시간으로 업데이트되는 상태 바가 표시됩니다.

```
 ⚕ claude-sonnet-4-20250514 │ 12.4K/200K │ [██████░░░░] 6% │ $0.06 │ 15m
```

| 요소 | 설명 |
|---|---|
| 모델명 | 현재 모델 (26자 초과 시 잘림) |
| 토큰 수 | 사용된 컨텍스트 토큰 / 최대 컨텍스트 윈도우 |
| 컨텍스트 바 | 색상으로 구분된 시각적 채움 표시 |
| 비용 | 예상 세션 비용 |
| 🗜️ N | 컨텍스트 압축 횟수 — 세션이 자동 압축된 횟수 |
| ▶ N | 활성 백그라운드 작업 수 |
| 경과 시간 | 세션 시작 후 지난 시간 |
| ⚠ YOLO | YOLO 모드 경고 — 자동 승인 모드가 켜진 경우 표시 |

**컨텍스트 색상 코드**

| 색상 | 임계값 | 의미 |
|---|---|---|
| 녹색 | 50% 미만 | 여유 충분 |
| 노란색 | 50–80% | 점점 채워짐 |
| 주황색 | 80–95% | 한계에 접근 중 |
| 빨간색 | 95% 이상 | 거의 가득 참 — `/compress` 고려 |

상세 비용 내역(입력/출력 토큰별)을 보려면 `/usage`를 입력합니다.

---

### 3-4. 키 바인딩

| 키 | 동작 |
|---|---|
| `Enter` | 메시지 전송 |
| `Alt+Enter`, `Ctrl+J`, `Shift+Enter` | 새 줄 (멀티라인 입력) |
| `Alt+V` | 클립보드에서 이미지 붙여넣기 |
| `Ctrl+V` | 텍스트 붙여넣기 및 클립보드 이미지 첨부 |
| `Ctrl+B` | 음성 모드에서 녹음 시작/중지 |
| `Ctrl+G` | 현재 입력 버퍼를 `$EDITOR`에서 편집 (저장 후 종료하면 전송) |
| `Ctrl+X Ctrl+E` | 외부 편집기 단축키 (이맥스 스타일, `Ctrl+G`와 동일 동작) |
| `Ctrl+C` | 에이전트 중단 (2초 내 두 번 누르면 강제 종료) |
| `Ctrl+D` | 종료 |
| `Ctrl+Z` | Hermes를 백그라운드로 전환 (Unix만 해당). `fg`로 복귀 |
| `Tab` | 자동 제안(고스트 텍스트) 수락 또는 슬래시 명령어 자동완성 |

**멀티라인 붙여넣기 미리보기**: 여러 줄을 붙여넣으면 전체를 스크롤에 쏟아붓는 대신 간결한 한 줄 미리보기(`[pasted: 47 lines, 1,842 chars — press Enter to send]`)를 표시합니다.

**Shift+Enter 호환성**: 대부분의 터미널은 기본적으로 Enter와 Shift+Enter를 동일하게 전송합니다. Kitty keyboard protocol을 지원하는 터미널(Kitty, foot, WezTerm, Ghostty)에서만 구분됩니다. 모든 터미널에서 `Alt+Enter`와 `Ctrl+J`는 항상 작동합니다.

---

### 3-5. 슬래시 명령어

`/`를 입력하면 자동완성 드롭다운이 표시됩니다. 명령어는 대소문자를 구분하지 않습니다(`/HELP`와 `/help`는 동일).

| 명령어 | 설명 |
|---|---|
| `/help` | 명령어 도움말 표시 |
| `/model` | 현재 모델 표시 또는 변경 |
| `/tools` | 현재 사용 가능한 도구 목록 |
| `/skills browse` | 스킬 허브 및 공식 선택적 스킬 탐색 |
| `/background <prompt>` | 별도 백그라운드 세션에서 프롬프트 실행 |
| `/skin` | 활성 CLI 스킨 표시 또는 변경 |
| `/voice on` | CLI 음성 모드 활성화 (녹음: Ctrl+B) |
| `/voice tts` | Hermes 응답의 음성 재생 토글 |
| `/reasoning high` | 추론 노력 증가 |
| `/title My Session` | 현재 세션 이름 지정 |
| `/status` | 세션 정보 표시 (모델/프로파일/토큰/경과 시간 및 최근 활동 요약) |
| `/sessions` | 인터랙티브 세션 선택기 열기 |
| `/compress` | 컨텍스트 수동 압축 |
| `/busy queue` | 에이전트 작동 중 입력 처리 방식 변경 |
| `/yolo` | 자동 승인 모드 토글 |

**설치된 스킬은 자동으로 슬래시 명령어가 됩니다.**

```bash
/gif-search funny cats
/axolotl help me fine-tune Llama 3 on my dataset
/github-pr-workflow create a PR for the auth refactor
```

---

### 3-6. 빠른 명령어 (Quick Commands)

LLM을 호출하지 않고 셸 명령어를 즉시 실행하는 커스텀 명령어를 정의할 수 있습니다. CLI와 메시징 플랫폼 모두에서 동작합니다.

```yaml
# ~/.hermes/config.yaml
quick_commands:
  status:
    type: exec
    command: systemctl status hermes-agent
  gpu:
    type: exec
    command: nvidia-smi --query-gpu=utilization.gpu,memory.used --format=csv,noheader
  restart:
    type: alias
    target: /gateway restart
```

이후 채팅에서 `/status`, `/gpu`, `/restart`를 입력하면 즉시 실행됩니다.

---

### 3-7. 성격(Personality) 설정

에이전트의 어조를 바꾸는 미리 정의된 성격을 설정할 수 있습니다.

```bash
/personality pirate
/personality kawaii
/personality concise
```

내장 성격: `helpful`, `concise`, `technical`, `creative`, `teacher`, `kawaii`, `catgirl`, `pirate`, `shakespeare`, `surfer`, `noir`, `uwu`, `philosopher`, `hype`

`~/.hermes/config.yaml`에서 커스텀 성격을 정의할 수도 있습니다.

```yaml
personalities:
  helpful: "You are a helpful, friendly AI assistant."
  kawaii: "You are a kawaii assistant! Use cute expressions..."
  pirate: "Arrr! Ye be talkin' to Captain Hermes..."
```

---

### 3-8. 에이전트 중단 방법

에이전트가 너무 오래 걸릴 때 중단하는 방법은 두 가지입니다.

- 에이전트가 작동 중인 동안 새 메시지를 입력하고 Enter를 누르면 현재 작업을 중단하고 새 지시로 전환합니다.
- `Ctrl+C`를 눌러도 현재 작업을 중단합니다 (2초 내 두 번 누르면 강제 종료).

진행 중인 터미널 명령은 즉시 종료됩니다 (SIGTERM 후 1초 뒤 SIGKILL).

**Busy Input Mode**

에이전트가 작동 중일 때 Enter를 누르면 어떻게 동작할지 설정할 수 있습니다.

| 모드 | 동작 |
|---|---|
| `interrupt` (기본값) | 현재 작업을 중단하고 즉시 처리 |
| `queue` | 메시지를 조용히 대기열에 넣고 에이전트가 끝난 후 다음 차례로 전송 |
| `steer` | 다음 도구 호출 후 현재 실행에 메시지를 주입 — 중단 없이 방향 전환 |

```yaml
# ~/.hermes/config.yaml
display:
  busy_input_mode: "steer"
```

CLI 내에서도 변경 가능합니다.

```bash
/busy queue
/busy steer
/busy interrupt
```

---

### 3-9. 도구 진행 상황 표시

에이전트가 작동 중일 때 CLI는 애니메이션 피드백을 보여줍니다.

**생각 중 애니메이션:**

```
  ◜ (｡•́︿•̀｡) pondering... (1.2s)
  ◠ (⊙_⊙) contemplating... (2.4s)
  ✧٩(ˊᗜˋ*)و✧ got it! (3.1s)
```

**도구 실행 피드:**

```
  ┊ 💻 terminal `ls -la` (0.3s)
  ┊ 🔍 web_search (1.2s)
  ┊ 📄 web_extract (2.1s)
```

`/verbose`로 표시 모드를 순환할 수 있습니다: `off` → `new` → `all` → `verbose`

---

### 3-10. 세션 관리

**세션 재개**

세션을 종료하면 재개 명령이 출력됩니다.

```
Resume this session with:
  hermes --resume 20260225_143052_a1b2c3

Session:    20260225_143052_a1b2c3
Duration:   12m 34s
Messages:   28 (5 user, 18 tool calls)
```

재개 방법은 여러 가지입니다.

```bash
hermes --continue                          # 가장 최근 CLI 세션 재개
hermes -c                                  # 단축형
hermes -c "my project"                     # 이름으로 세션 재개 (계보 내 최신)
hermes --resume 20260225_143052_a1b2c3     # ID로 특정 세션 재개
hermes --resume "refactoring auth"         # 제목으로 재개
```

재개하면 모든 이전 메시지, 도구 호출, 응답이 SQLite에서 복원되어 마치 그 자리를 떠난 적 없는 것처럼 에이전트가 전체 히스토리를 볼 수 있습니다.

**세션 저장소**

CLI 세션은 `~/.hermes/state.db` SQLite 데이터베이스에 저장됩니다. 데이터베이스는 세션 메타데이터, 메시지 히스토리, 압축/재개 전반의 계보(lineage), 전문 검색 인덱스를 유지합니다.

---

### 3-11. 컨텍스트 압축

대화가 길어지면 컨텍스트 한계에 접근할 때 자동으로 중간 턴들이 요약됩니다.

```yaml
# ~/.hermes/config.yaml
compression:
  enabled: true
  threshold: 0.50    # 컨텍스트 한계의 50%에서 압축

auxiliary:
  compression:
    model: ""  # 비워두면 기본 채팅 모델 사용. 또는 저렴한 빠른 모델 지정 가능
               # 예: "google/gemini-3-flash-preview"
```

압축이 실행되면 처음 3개 턴과 마지막 20개 턴은 항상 보존하고, 중간 턴들이 요약됩니다.

---

### 3-12. 백그라운드 세션

현재 CLI를 계속 사용하면서 별도 백그라운드 세션에서 프롬프트를 실행할 수 있습니다.

```bash
/background /var/log의 로그를 분석하고 오늘의 오류를 요약해줘
```

즉시 확인 메시지와 함께 프롬프트가 돌아옵니다.

```
🔄 Background task #1 started: "Analyze the logs in /var/log and summarize..."
   Task ID: bg_143022_a1b2c3
```

백그라운드 에이전트는 현재 세션 히스토리를 모르는 완전히 독립된 세션으로 실행됩니다. 현재 세션의 모델, 프로바이더, 툴셋, 추론 설정은 상속받습니다. 여러 백그라운드 작업을 동시에 실행할 수 있으며, 각각 번호가 붙은 ID를 받습니다.

작업이 완료되면 결과가 터미널에 패널로 표시됩니다.

```
╭─ ⚕ Hermes (background #1) ──────────────────────────────────╮
│ 오늘 syslog에서 3개의 오류를 찾았습니다:                     │
│ 1. 03:22에 OOM killer 실행 — nginx 프로세스 종료             │
│ 2. 07:15에 /dev/sda1 디스크 I/O 오류                        │
│ 3. 14:30에 192.168.1.50으로부터 SSH 로그인 시도 실패         │
╰──────────────────────────────────────────────────────────────╯
```

---

## 4. Docker로 운영하기

### 4-1. Docker와 Hermes의 두 가지 관계

Docker는 Hermes와 두 가지 방식으로 교차합니다.

1. **Hermes를 Docker 안에서 실행** — 에이전트 자체가 컨테이너 안에서 동작합니다 (이 섹션의 주요 내용).
2. **Docker를 터미널 백엔드로 사용** — 에이전트는 호스트에서 실행하지만, 모든 터미널 명령을 단일한 영구 Docker 샌드박스 컨테이너 안에서 실행합니다.

이 섹션은 방법 1에 집중합니다.

---

### 4-2. 빠른 시작

처음이라면 대화형으로 컨테이너를 실행해 설정 마법사를 완료합니다.

```bash
mkdir -p ~/.hermes
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup
```

이 명령으로 설정 마법사가 실행되어 API 키를 물어보고 `~/.hermes/.env`에 저장합니다. 이 과정은 최초 한 번만 필요합니다.

---

### 4-3. 게이트웨이 모드로 실행

설정이 완료된 후 백그라운드에서 영구 게이트웨이로 실행합니다.

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run
```

포트 8642는 게이트웨이의 OpenAI 호환 API 서버와 헬스 엔드포인트를 노출합니다. 텔레그램, 디스코드 같은 채팅 플랫폼만 사용한다면 포트 노출은 선택 사항입니다. 대시보드나 외부 도구에서 게이트웨이에 접근하려면 필요합니다.

**API 서버를 외부에 노출하려면** 추가 환경 변수가 필요합니다.

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  -e API_SERVER_ENABLED=true \
  -e API_SERVER_HOST=0.0.0.0 \
  -e API_SERVER_KEY="$(openssl rand -hex 32)" \
  -e API_SERVER_CORS_ORIGINS='*' \
  nousresearch/hermes-agent gateway run
```

> **보안 경고**: 인터넷에 연결된 기기에서 포트를 여는 것은 보안 위험입니다. 위험을 충분히 이해한 경우에만 하세요.

---

### 4-4. 대시보드 실행

내장 웹 대시보드는 게이트웨이와 같은 컨테이너 안에서 선택적 사이드 프로세스로 실행됩니다. `HERMES_DASHBOARD=1`을 설정하면 컨테이너 루프백(127.0.0.1)에서 대시보드가 실행됩니다.

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  -e HERMES_DASHBOARD=1 \
  nousresearch/hermes-agent gateway run
```

대시보드 관련 환경 변수 목록입니다.

| 환경 변수 | 설명 | 기본값 |
|---|---|---|
| `HERMES_DASHBOARD` | `1`로 설정 시 대시보드를 메인 명령과 함께 실행 | 미설정 (대시보드 미시작) |
| `HERMES_DASHBOARD_HOST` | 대시보드 HTTP 서버 바인드 주소 | `127.0.0.1` |
| `HERMES_DASHBOARD_PORT` | 대시보드 HTTP 서버 포트 | `9119` |
| `HERMES_DASHBOARD_TUI` | `1`로 설정 시 브라우저 내 Chat 탭 노출 | 미설정 |

> 기본적으로 대시보드는 루프백에 머물러 인증되지 않은 웹 서페이스가 네트워크에 노출되지 않습니다.

---

### 4-5. 인터랙티브 CLI 채팅

```bash
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent
```

이미 실행 중인 컨테이너에 터미널을 연 경우에는 다음으로 직접 실행합니다.

```bash
/opt/hermes/.venv/bin/hermes
```

---

### 4-6. 영구 볼륨 구성

`/opt/data` 볼륨은 모든 Hermes 상태의 단일 진실의 원천(source of truth)입니다. 호스트의 `~/.hermes/` 디렉토리에 매핑됩니다.

| 경로 | 내용 |
|---|---|
| `.env` | API 키와 시크릿 |
| `config.yaml` | Hermes 전체 설정 |
| `SOUL.md` | 에이전트 성격/정체성 |
| `sessions/` | 대화 기록 |
| `memories/` | 영구 메모리 저장소 |
| `skills/` | 설치된 스킬 |
| `cron/` | 예약 작업 정의 |
| `hooks/` | 이벤트 훅 |
| `logs/` | 런타임 로그 |
| `skins/` | 커스텀 CLI 스킨 |

> **경고**: 같은 데이터 디렉토리에 두 개의 Hermes 게이트웨이 컨테이너를 동시에 실행하지 마세요. 세션 파일과 메모리 저장소는 동시 쓰기를 지원하지 않습니다.

---

### 4-7. 멀티 프로파일 운영

Docker 환경에서는 Hermes 내장 멀티 프로파일 기능보다 **프로파일당 컨테이너 하나**를 권장합니다.

```bash
# 업무 프로파일
docker run -d \
  --name hermes-work \
  --restart unless-stopped \
  -v ~/.hermes-work:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run

# 개인 프로파일
docker run -d \
  --name hermes-personal \
  --restart unless-stopped \
  -v ~/.hermes-personal:/opt/data \
  -p 8643:8642 \
  nousresearch/hermes-agent gateway run
```

컨테이너 분리의 이점은 이렇습니다. 각 컨테이너는 자체 파일 시스템, 프로세스 테이블, 리소스 한계를 가져 완전한 격리가 이루어집니다. 한 프로파일의 충돌이나 폭주 세션이 다른 프로파일에 영향을 주지 않습니다. 각 에이전트를 독립적으로 업그레이드·재시작·일시 중지·롤백할 수 있습니다.

**Docker Compose로 멀티 프로파일 구성:**

```yaml
services:
  hermes-work:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-work
    restart: unless-stopped
    command: gateway run
    ports:
      - "8642:8642"
    volumes:
      - ~/.hermes-work:/opt/data

  hermes-personal:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-personal
    restart: unless-stopped
    command: gateway run
    ports:
      - "8643:8642"
    volumes:
      - ~/.hermes-personal:/opt/data
```

---

### 4-8. Docker Compose 전체 예시

게이트웨이와 대시보드를 함께 실행하는 완전한 구성입니다.

```yaml
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    command: gateway run
    ports:
      - "8642:8642"   # 게이트웨이 API
      - "9119:9119"   # 대시보드 (HERMES_DASHBOARD=1일 때만)
    volumes:
      - ~/.hermes:/opt/data
    environment:
      - HERMES_DASHBOARD=1
      # 아래 주석 해제로 .env 파일 대신 환경 변수로 API 키 전달 가능:
      # - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      # - OPENAI_API_KEY=${OPENAI_API_KEY}
      # - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: "2.0"
```

실행 및 로그 확인:

```bash
docker compose up -d
docker compose logs -f
```

대시보드 출력은 `[dashboard]` 접두사로 구분됩니다.

---

### 4-9. 리소스 권장 사양

| 리소스 | 최소 | 권장 |
|---|---|---|
| 메모리 | 1 GB | 2–4 GB |
| CPU | 1 코어 | 2 코어 |
| 디스크 (데이터 볼륨) | 500 MB | 2+ GB (세션/스킬이 쌓일수록 증가) |

브라우저 자동화(Playwright/Chromium)가 가장 메모리를 많이 사용합니다. 브라우저 도구가 필요 없다면 1 GB면 충분합니다.

```bash
docker run -d \
  --name hermes \
  --restart unless-stopped \
  --memory=4g --cpus=2 \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run
```

---

### 4-10. 로컬 추론 서버(vLLM, Ollama 등) 연결

Hermes Docker와 추론 서버를 함께 사용할 때 네트워킹에 주의가 필요합니다.

**Docker Compose 방식 (권장)**

두 서비스를 같은 Docker 네트워크에 놓는 것이 가장 안정적입니다.

```yaml
services:
  vllm:
    image: vllm/vllm-openai:latest
    container_name: vllm
    command: >
      --model Qwen/Qwen2.5-7B-Instruct
      --served-model-name my-model
      --host 0.0.0.0
      --port 8000
    ports:
      - "8000:8000"
    networks:
      - hermes-net

  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    command: gateway run
    volumes:
      - ~/.hermes:/opt/data
    networks:
      - hermes-net

networks:
  hermes-net:
    driver: bridge
```

`~/.hermes/config.yaml`에서 컨테이너 이름을 호스트명으로 사용합니다.

```yaml
model:
  provider: custom
  model: my-model
  base_url: http://vllm:8000/v1
  api_key: "none"
```

> `localhost`나 `127.0.0.1`이 아니라 컨테이너 이름(`vllm`)을 호스트명으로 사용합니다. `base_url` 끝에 슬래시(`/`)를 붙이지 마세요.

**Standalone Docker run 방식**

추론 서버가 호스트에서 실행 중이라면 플랫폼별로 다르게 설정합니다.

- macOS / Windows: `host.docker.internal` 사용
- Linux: `--network host` 옵션 사용

**연결 확인**

```bash
docker exec hermes curl -s http://vllm:8000/v1/models
```

JSON 응답이 돌아오면 정상입니다.

---

### 4-11. Docker 이미지 구성

공식 이미지는 `debian:13.4` 기반이며 다음이 포함되어 있습니다.

- Python 3 및 모든 Hermes 의존성 (`uv pip install -e ".[all]"`)
- Node.js + npm (브라우저 자동화 및 WhatsApp 브릿지)
- Playwright + Chromium
- ripgrep, ffmpeg, git, tini
- **docker-cli** — 컨테이너 안에서 호스트의 Docker 데몬을 제어할 수 있음 (`/var/run/docker.sock` 마운트 필요)
- **openssh-client** — 컨테이너화된 설치에서 SSH 터미널 백엔드 활성화
- WhatsApp 브릿지

엔트리포인트(`docker/entrypoint.sh`)는 최초 실행 시 데이터 볼륨을 초기화합니다. 디렉토리 구조 생성, `.env.example` → `.env` 복사, 기본 `config.yaml`과 `SOUL.md` 복사, 번들 스킬 동기화 등을 처리하고, `HERMES_DASHBOARD=1`이면 대시보드를 백그라운드 사이드 프로세스로 실행한 뒤 hermes를 시작합니다.

---

### 4-12. 업그레이드

```bash
docker pull nousresearch/hermes-agent:latest
docker rm -f hermes
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run
```

또는 Docker Compose를 사용하는 경우:

```bash
docker compose pull
docker compose up -d
```

데이터 디렉토리는 그대로 유지됩니다.

---

### 4-13. Docker 문제 해결

| 문제 | 원인 및 해결 방법 |
|---|---|
| 컨테이너가 즉시 종료됨 | `docker logs hermes`로 확인. 주로 .env 파일 없음/잘못됨 또는 포트 충돌 |
| "Permission denied" 오류 | 컨테이너가 UID 10000의 non-root `hermes` 사용자로 실행됨. `chmod -R 755 ~/.hermes` 실행 |
| 브라우저 도구가 작동 안 함 | Playwright는 공유 메모리가 필요함. `--shm-size=1g` 추가 |
| 게이트웨이가 네트워크 오류 후 재연결 안 됨 | `docker restart hermes`로 컨테이너 재시작. `--restart unless-stopped` 플래그가 대부분의 일시적 오류를 처리함 |

**컨테이너 상태 확인**

```bash
docker logs --tail 50 hermes          # 최근 로그
docker run -it --rm nousresearch/hermes-agent:latest version  # 버전 확인
docker stats hermes                    # 리소스 사용량
```

---

## 빠른 참조 명령어

| 명령어 | 설명 |
|---|---|
| `hermes` | 채팅 시작 |
| `hermes --tui` | 모던 TUI로 채팅 시작 |
| `hermes model` | LLM 프로바이더 및 모델 선택 |
| `hermes tools` | 플랫폼별 활성화 도구 설정 |
| `hermes setup` | 전체 설정 마법사 |
| `hermes doctor` | 문제 진단 |
| `hermes update` | 최신 버전으로 업데이트 |
| `hermes gateway` | 메시징 게이트웨이 시작 |
| `hermes --continue` | 마지막 세션 재개 |
| `hermes sessions list` | 세션 목록 보기 |

---

*작성일: 2026년 5월 24일*  
*참고: Nous Research 공식 Hermes Agent 문서 (hermes-agent.nousresearch.com) — Quickstart, Installation, CLI Interface, Docker*

- [Quickstart](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart)
- [Installation](https://hermes-agent.nousresearch.com/docs/getting-started/installation)
- [CLI Interface](https://hermes-agent.nousresearch.com/docs/user-guide/cli)
- [Hermes Agent — Docker](https://hermes-agent.nousresearch.com/docs/user-guide/docker)