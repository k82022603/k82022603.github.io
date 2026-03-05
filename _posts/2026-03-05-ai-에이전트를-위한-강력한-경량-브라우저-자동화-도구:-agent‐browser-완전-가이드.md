---
title: "AI 에이전트를 위한 강력한 경량 브라우저 자동화 도구: agent-browser 완전 가이드"
date: 2026-03-05 21:00:00 +0900
categories: [AI,  AI Agent]
mermaid: [True]
tags: [AI,  Vercel,  agent-browser,  CLI,  Playwright,  playwright-mcp,  Skills,  Slack,  Electron,  Claude.write]
---


> **Vercel Labs**가 공개한 AI 코딩 에이전트 전용 CLI 브라우저 자동화 도구의 모든 것  
> 작성 기준: 2026-03-05

## 관련글
[Agent Browser: Vercel이 공개한, AI 에이전트를 위한 경량화된 명령줄 브라우저 자동화 도구](https://discuss.pytorch.kr/t/agent-browser-vercel-ai/9125)

---

## 목차

1. [등장 배경과 문제 의식](#1-등장-배경과-문제-의식)
2. [agent-browser란 무엇인가](#2-agent-browser란-무엇인가)
3. [3계층 아키텍처: 성능과 안정성의 설계 원리](#3-3계층-아키텍처-성능과-안정성의-설계-원리)
4. [핵심 혁신: 스냅샷 기반 참조 시스템](#4-핵심-혁신-스냅샷-기반-참조-시스템)
5. [기존 Playwright MCP와의 비교](#5-기존-playwright-mcp와의-비교)
6. [설치 및 초기 설정](#6-설치-및-초기-설정)
7. [AI 코딩 어시스턴트 연동 (Skills 시스템)](#7-ai-코딩-어시스턴트-연동-skills-시스템)
8. [전체 명령어 레퍼런스](#8-전체-명령어-레퍼런스)
9. [고급 기능: 네트워크, 세션, 디버깅](#9-고급-기능-네트워크-세션-디버깅)
10. [확장 스킬: Electron 앱과 Slack 자동화](#10-확장-스킬-electron-앱과-slack-자동화)
11. [클라우드 브라우저 인프라 연동](#11-클라우드-브라우저-인프라-연동)
12. [실전 활용 패턴과 워크플로우 예시](#12-실전-활용-패턴과-워크플로우-예시)
13. [보안 기능과 엔터프라이즈 고려사항](#13-보안-기능과-엔터프라이즈-고려사항)
14. [라이선스 및 생태계 현황](#14-라이선스-및-생태계-현황)

---

## 1. 등장 배경과 문제 의식

AI 코딩 에이전트의 역할이 단순한 코드 생성을 넘어 프론트엔드 개발, 테스트, 품질 검증까지 확장되면서, 한 가지 본질적인 문제가 수면 위로 떠올랐습니다. AI가 스스로 작성한 코드가 실제 브라우저 환경에서 의도한 대로 동작하는지를 누가 검증할 것인가 하는 문제입니다. 종전까지는 AI가 코드를 완성한 뒤 "작업이 완료되었습니다"라는 메시지를 출력하면, 개발자가 직접 브라우저를 열고 클릭하며 동작을 확인해야 했습니다. 이 검증 단계는 자동화의 공백으로 남아 있었고, 에이전트 기반 개발의 핵심 병목 지점이 되었습니다.

이 문제를 해소하기 위한 기존의 접근법은 주로 Model Context Protocol(MCP) 기반의 Playwright 도구를 활용하는 것이었습니다. 그러나 이 방식에는 근본적인 한계가 있었습니다. AI 모델이 브라우저의 전체 접근성 트리(Accessibility Tree)나 복잡한 DOM 구조를 통째로 읽어들이면, 단 한 번의 화면 탐색만으로도 수만 개의 토큰이 소모됩니다. 복잡한 현대 웹 애플리케이션의 경우 하나의 페이지에서 수천 개의 DOM 노드가 존재하고, 이를 모두 AI에게 전달하면 컨텍스트 윈도우 한계에 순식간에 부딪히는 것은 물론, 정작 중요한 의사결정에 써야 할 토큰을 낭비하는 악순환이 벌어졌습니다.

Vercel의 내부 연구 결과도 이 문제를 정량적으로 확인해 주었습니다. Vercel이 자체 개발한 텍스트-to-SQL 에이전트 D0의 사례를 보면, 17개의 전문화된 도구를 사용했을 때 성공률이 80%에 그쳤고, 평균 처리 시간은 274.8초, 평균 토큰 소비는 약 102,000개에 달했습니다. 최악의 경우 724초가 소요되고 145,463개의 토큰을 소진하고도 작업이 실패하는 상황이 발생했습니다. 이를 `ExecuteCommand`와 `ExecuteSQL` 두 개의 단순한 도구로 재구성했더니 성공률이 100%로 올라갔고, 평균 처리 시간은 77.4초(3.5배 단축), 평균 토큰 사용량은 약 61,000개(37% 감소)로 극적으로 개선되었습니다. **도구의 수를 줄이고 단순화하면 AI 성능이 오히려 향상된다**는 역설적 원칙이 agent-browser 설계의 근간입니다.

---

## 2. agent-browser란 무엇인가

**agent-browser**는 Vercel Labs가 개발하고 오픈소스로 공개한, AI 에이전트 전용 헤드리스 브라우저 자동화 CLI(Command-Line Interface) 도구입니다. 프로젝트는 Apache License 2.0 하에 GitHub(`vercel-labs/agent-browser`)에서 관리되고 있습니다.

이 도구의 핵심 가치는 "AI가 브라우저를 제어하는 방식을 근본적으로 바꾸는 것"에 있습니다. 복잡한 CSS 선택자나 XPath 없이, 간결한 참조 태그(@e1, @e2 등)만으로 화면 요소를 정밀하게 식별하고 제어할 수 있도록 설계되었습니다. 결과적으로 AI 모델이 소비하는 토큰을 기존 방식 대비 최대 90~93%까지 절감할 수 있다는 것이 내부 연구 및 사용자 피드백을 통해 확인되었습니다.

agent-browser는 Claude Code, Cursor, GitHub Copilot, Gemini CLI, Codex, Goose, OpenCode, Windsurf 등 터미널 bash 명령어를 지원하는 모든 AI 코딩 어시스턴트와 즉각 호환됩니다. 별도의 서버 설정이나 웹소켓 프로토콜 구성 없이, 단 하나의 전역 설치 명령으로 강력한 브라우저 자동화 환경을 구축할 수 있다는 점에서 기존 도구들과 뚜렷하게 차별화됩니다.

---

## 3. 3계층 아키텍처: 성능과 안정성의 설계 원리

agent-browser의 내부 구조는 빠른 명령 실행과 안정적인 브라우저 제어를 동시에 달성하기 위해 정교한 3계층으로 설계되어 있습니다.

### 1계층: Rust 기반 CLI (최전방 인터페이스)

사용자의 명령을 가장 먼저 처리하는 최전방에는 Rust 언어로 작성된 고성능 명령줄 인터페이스가 위치합니다. Rust는 메모리 안전성과 제로-코스트 추상화로 유명한 시스템 프로그래밍 언어로, 밀리초 미만(sub-millisecond)의 극도로 짧은 지연 시간 내에 명령어를 파싱하고 백그라운드 데몬 프로세스와 통신할 수 있습니다. `npm install -g agent-browser`로 전역 설치했을 때 네이티브 Rust 바이너리가 활성화되어 최고 성능을 발휘합니다. 반면 `npx agent-browser`로 실행하면 Node.js 경유로 처리되므로 속도가 다소 느려집니다. 정기적으로 사용할 목적이라면 전역 설치가 권장됩니다.

### 2계층: Node.js 데몬 (브라우저 생명주기 관리자)

Rust CLI 뒤에서 상주하며 실질적인 브라우저 세션을 관리하는 것은 Node.js 기반의 데몬 프로세스입니다. 이 데몬은 Playwright 브라우저 인스턴스의 생명주기를 완전히 관장하며, 명령이 들어올 때마다 새 브라우저를 실행하는 대신 이미 열려 있는 세션을 재사용함으로써 연속 명령 실행 속도를 극적으로 높입니다. 데몬은 첫 번째 명령이 실행될 때 자동으로 기동되어 백그라운드에서 계속 실행 상태를 유지하므로, 개발자는 데몬의 존재를 의식하지 않아도 됩니다.

### 3계층: Node.js 폴백 메커니즘

네이티브 Rust 바이너리 실행이 불가능한 특수 환경(일부 CI/CD 파이프라인, 특정 Linux 배포판 등)을 대비하여, Node.js 프로세스로 자동 전환하는 폴백 메커니즘이 구현되어 있습니다. 이 덕분에 운영 환경의 제약과 관계없이 동일한 명령어로 일관된 동작을 보장받을 수 있습니다.

이 3계층 구조의 핵심 이점은 **"무거운 브라우저를 매번 새로 띄우지 않고도 격리된 세션을 안정적으로 유지하며 빠른 연속 작업을 수행"**할 수 있다는 점입니다. AI 에이전트가 수십, 수백 개의 명령을 연속으로 실행하는 장시간 자율 세션에서 이 아키텍처의 강점이 더욱 두드러집니다.

---

## 4. 핵심 혁신: 스냅샷 기반 참조 시스템

agent-browser에서 가장 혁신적인 기술 요소는 단연 **스냅샷(snapshot) 기반의 고유 요소 참조 시스템**입니다. 이 시스템이 agent-browser를 단순한 Playwright 래퍼와 구분하는 결정적 차별점입니다.

### 작동 원리

AI 에이전트가 특정 URL로 이동한 뒤 `agent-browser snapshot` 명령을 실행하면, 도구는 페이지의 DOM 전체를 그대로 전달하는 대신 다음 과정을 거쳐 정제된 정보를 반환합니다.

첫째, 페이지에 존재하는 수천 개의 DOM 노드 중 사용자가 실제로 상호작용할 수 있는 의미 있는 요소만을 선별합니다. 버튼, 텍스트 입력창, 링크, 드롭다운, 체크박스 등 기능적으로 중요한 요소들이 여기에 해당합니다.

둘째, 선별된 각 요소에 `@e1`, `@e2`, `@e3`와 같이 짧고 고유한 참조 식별자를 순차적으로 부여합니다. 이 식별자는 복잡한 CSS 선택자나 XPath 경로 없이도 특정 요소를 정확히 가리키는 간결한 포인터 역할을 합니다.

셋째, AI 모델에게는 이 압축된 참조 목록만이 전달됩니다. AI는 장황한 원본 DOM 문서를 읽는 대신 이 간결한 목록을 분석하여 즉각적으로 다음 행동을 결정합니다.

```bash
# 핵심 워크플로우 예시
agent-browser open https://example.com/login

agent-browser snapshot -i
# 출력 예시:
# @e1 [input type="email"] placeholder="이메일 주소"
# @e2 [input type="password"] placeholder="비밀번호"
# @e3 [button] "로그인"
# @e4 [a] "비밀번호 찾기"

agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "my_password"
agent-browser click @e3
agent-browser wait --load networkidle
```

### 왜 이것이 중요한가

전통적인 브라우저 자동화에서 AI는 복잡한 CSS 선택자(`#app > div.container > form > input[type="email"]`)나 XPath(`/html/body/div[1]/form/input[2]`)를 추론해야 했습니다. 이 과정에서 오류가 빈번하게 발생하고, 페이지 구조가 조금만 바뀌어도 선택자가 깨지는 취약성이 내재되어 있었습니다. 참조 식별자 방식은 이러한 취약성을 원천적으로 제거하고, AI가 오류 없이 결정론적으로 요소를 선택할 수 있도록 보장합니다.

`-i` 플래그를 사용하면 상호작용 가능한 요소만 필터링하여 출력 크기를 더욱 줄일 수 있고, `-C` 플래그를 추가하면 onclick 속성이 있는 div나 cursor:pointer 스타일의 커서 상호작용 요소까지 포함합니다. `-d` 옵션으로 트리 탐색 깊이를 제한하거나, `-s` 옵션으로 특정 CSS 선택자 범위 내로 스냅샷을 한정할 수도 있어, 대규모 단일 페이지 애플리케이션에서도 필요한 정보만 정밀하게 추출할 수 있습니다.

---

## 5. 기존 Playwright MCP와의 비교

agent-browser가 해결하고자 했던 문제를 더 명확히 이해하기 위해, 기존 MCP 기반 Playwright 도구와의 차이를 구체적으로 살펴보겠습니다.

### 도구 복잡성

기존 Playwright MCP는 26개에 달하는 도구 메서드를 AI에게 노출합니다. 도구 정의 자체의 스키마 오버헤드가 컨텍스트를 잡아먹고, AI는 어떤 도구를 선택해야 할지 판단하는 데만도 상당한 추론 비용을 소모합니다. agent-browser는 단순한 CLI 명령어 체계로 이 복잡성을 해소합니다. AI 에이전트는 MCP 서버 스키마를 로드하지 않고 bash 명령어를 직접 실행하므로, 유휴 상태에서도 도구 정의로 인한 토큰 소비가 없습니다.

### 컨텍스트 토큰 소비

복잡한 웹 페이지의 경우 전체 접근성 트리를 그대로 전송하면 수천 개의 노드가 포함되어 컨텍스트 창이 빠르게 소진됩니다. agent-browser의 스냅샷 참조 시스템은 이를 최대 90~93%까지 절감하는 것으로 내부 연구에서 확인되었습니다. 장시간 자율 에이전트 세션에서 이 차이는 작업 완료 가능 여부를 좌우하는 결정적 요소가 됩니다.

### 설정 복잡도

Playwright MCP는 별도의 서버 프로세스를 기동하고 AI 도구와 웹소켓 또는 stdio 프로토콜로 연동하는 설정이 필요합니다. agent-browser는 전역 설치 후 바로 사용 가능한 단순한 CLI 구조를 택하여 진입 장벽을 대폭 낮췄습니다.

### 각 도구의 적합 사용 시나리오

두 도구는 경쟁 관계라기보다 상호보완적입니다. agent-browser는 컨텍스트 예산이 중요한 장시간 자율 세션, 기본적인 탐색 및 검증 작업, MCP 설정을 생략하고 싶은 환경에 적합합니다. Playwright MCP는 정교한 네트워크 인터셉트, PDF 생성, 복잡한 멀티탭 워크플로우, 기존 Playwright 테스트 스위트와의 연동이 필요한 경우에 여전히 강력한 선택지입니다.

---

## 6. 설치 및 초기 설정

### 전역 설치 (권장)

```bash
# npm을 통한 전역 설치 (Rust 네이티브 바이너리 포함)
npm install -g agent-browser

# Chromium 브라우저 다운로드 (최초 1회)
agent-browser install

# Linux 환경에서 시스템 의존성 포함 설치
agent-browser install --with-deps
```

전역 설치 시 네이티브 Rust 바이너리가 활성화되어 밀리초 미만의 명령 파싱 성능을 발휘합니다. 이후의 모든 명령은 항상 실행 중인 데몬 프로세스에 즉각 전달됩니다.

### npx를 통한 임시 실행

설치 없이 바로 테스트해보고 싶다면 npx를 사용할 수 있습니다. 단, Node.js를 경유하므로 전역 설치 대비 속도가 느립니다.

```bash
npx agent-browser install   # 최초 1회 Chromium 다운로드
npx agent-browser open example.com
```

### 소스 빌드 (개발자용)

```bash
git clone https://github.com/vercel-labs/agent-browser
cd agent-browser
pnpm install
pnpm build
pnpm build:native   # Rust 빌드 (rustup 필요)
pnpm link --global
agent-browser install
```

---

## 7. AI 코딩 어시스턴트 연동 (Skills 시스템)

agent-browser는 단독 CLI로도 동작하지만, AI 코딩 어시스턴트에 "스킬(Skill)"을 등록하면 에이전트가 자동으로 적절한 상황에서 agent-browser를 활용하도록 유도할 수 있습니다. 스킬은 AI가 참조하는 구조화된 지침 파일로, 올바른 워크플로우, 명령어 사용법, 세션 관리 전략 등이 포함되어 있습니다.

### Claude Code에 스킬 등록

```bash
# 공식 agent-browser 스킬 설치
npx skills add vercel-labs/agent-browser --skill agent-browser

# 설치 후 .claude/skills/agent-browser/SKILL.md 파일이 생성됨
```

스킬 파일은 저장소에서 자동으로 최신 상태를 유지하므로, node_modules에서 수동 복사하면 최신 버전을 놓칠 수 있습니다. 더욱 일관된 동작을 위해 프로젝트 또는 전역 지침 파일에 다음 내용을 추가하는 것이 권장됩니다.

```markdown
## Browser Automation
Use `agent-browser` for web automation.
Run `agent-browser --help` for all commands.
Core workflow:
1. `agent-browser open <url>` - Navigate to page
2. `agent-browser snapshot -i` - Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` - Interact using refs
```

### 추가 스킬 설치

```bash
# Dogfood 스킬 (탐색적 테스트)
npx skills add vercel-labs/agent-browser --skill dogfood

# Electron 앱 제어 스킬
npx skills add vercel-labs/agent-browser --skill electron

# Slack 자동화 스킬
npx skills add vercel-labs/agent-browser --skill slack
```

스킬이 등록되면 Cursor, Claude Code 등 AI 도구는 사용자의 자연어 요청을 분석하여 자동으로 적합한 스킬을 선택하고 실행합니다. "이 로그인 폼을 테스트해줘", "Slack에서 새 알림 확인해줘"와 같은 요청만으로도 에이전트가 스스로 agent-browser를 활용하여 작업을 수행합니다.

---

## 8. 전체 명령어 레퍼런스

### 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `agent-browser open <url>` | URL로 이동 (별칭: goto, navigate) |
| `agent-browser click <sel>` | 요소 클릭 (`--new-tab` 옵션으로 새 탭) |
| `agent-browser dblclick <sel>` | 더블 클릭 |
| `agent-browser fill <sel> <text>` | 기존 내용 지우고 텍스트 입력 |
| `agent-browser type <sel> <text>` | 기존 내용 유지하며 텍스트 추가 |
| `agent-browser press <key>` | 특정 키 입력 (Enter, Tab, Control+a 등) |
| `agent-browser keyboard type <text>` | 현재 포커스 위치에 실제 키 입력 |
| `agent-browser keyboard inserttext <text>` | 키 이벤트 없이 텍스트 삽입 |
| `agent-browser hover <sel>` | 마우스 커서 올리기 |
| `agent-browser select <sel> <val>` | 드롭다운 옵션 선택 |
| `agent-browser check <sel>` | 체크박스 체크 |
| `agent-browser uncheck <sel>` | 체크박스 해제 |
| `agent-browser scroll <dir> [px]` | 방향(up/down/left/right) 스크롤 |
| `agent-browser drag <src> <tgt>` | 드래그 앤 드롭 |
| `agent-browser upload <sel> <files>` | 파일 업로드 |
| `agent-browser screenshot [path]` | 스크린샷 (--full: 전체 페이지) |
| `agent-browser screenshot --annotate` | 요소 번호 표시 스크린샷 |
| `agent-browser pdf <path>` | 페이지를 PDF로 저장 |
| `agent-browser snapshot` | AI 최적화 접근성 트리 생성 |
| `agent-browser eval <js>` | JavaScript 코드 실행 |
| `agent-browser close` | 브라우저 종료 (별칭: quit, exit) |

### 스냅샷 옵션 상세

```bash
agent-browser snapshot            # 전체 접근성 트리
agent-browser snapshot -i         # 상호작용 가능 요소만
agent-browser snapshot -i -C      # 커서 상호작용 요소(onclick div 등) 포함
agent-browser snapshot -c         # 압축 모드 (빈 구조 요소 제거)
agent-browser snapshot -d 3       # 트리 깊이 3단계 제한
agent-browser snapshot -s "#main" # 특정 CSS 선택자 범위 한정
agent-browser snapshot -i -c -d 5 # 옵션 조합 사용
```

### 정보 추출 명령어

```bash
agent-browser get text <sel>      # 텍스트 내용 추출
agent-browser get html <sel>      # innerHTML 추출
agent-browser get value <sel>     # 입력 필드 값
agent-browser get attr <sel> <attr> # 특정 속성 값
agent-browser get title           # 페이지 제목
agent-browser get url             # 현재 URL
agent-browser get count <sel>     # 일치 요소 개수
agent-browser get box <sel>       # 요소의 Bounding Box 좌표
agent-browser get styles <sel>    # 계산된 스타일
```

### 상태 확인 명령어

```bash
agent-browser is visible <sel>    # 요소 가시성 확인
agent-browser is enabled <sel>    # 요소 활성화 여부
agent-browser is checked <sel>    # 체크 상태 확인
```

### 의미론적 요소 탐색

```bash
agent-browser find role <role> <action> [value]     # ARIA 역할로 탐색
agent-browser find text <text> <action>              # 텍스트 내용으로 탐색
agent-browser find label <label> <action> [value]    # 라벨 텍스트로 탐색
agent-browser find placeholder <ph> <action> [value] # 플레이스홀더로 탐색
agent-browser find alt <text> <action>               # alt 텍스트로 탐색
agent-browser find testid <id> <action> [value]      # data-testid로 탐색
agent-browser find first <sel> <action> [value]      # 첫 번째 일치 요소
agent-browser find last <sel> <action> [value]       # 마지막 일치 요소
agent-browser find nth <n> <sel> <action> [value]    # n번째 일치 요소
```

사용 가능한 동작(action): `click`, `fill`, `type`, `hover`, `focus`, `check`, `uncheck`, `text`

### 대기 명령어

```bash
agent-browser wait <selector>              # 요소가 나타날 때까지 대기
agent-browser wait <ms>                    # 지정 밀리초 대기
agent-browser wait --text "Welcome"        # 특정 텍스트 출현 대기
agent-browser wait --url "**/dashboard"    # URL 패턴 일치 대기
agent-browser wait --load networkidle      # 페이지 완전 로드 대기
agent-browser wait --fn "window.ready === true"  # JS 조건 충족 대기
```

### 탐색 명령어

```bash
agent-browser back      # 뒤로 가기
agent-browser forward   # 앞으로 가기
agent-browser reload    # 페이지 새로고침
```

### 마우스 세밀 제어

```bash
agent-browser mouse move <x> <y>     # 마우스 이동
agent-browser mouse down [button]    # 마우스 버튼 누르기
agent-browser mouse up [button]      # 마우스 버튼 떼기
agent-browser mouse wheel <dy> [dx]  # 마우스 휠 스크롤
```

### 탭 및 창 관리

```bash
agent-browser tab            # 탭 목록 확인
agent-browser tab new [url]  # 새 탭 열기
agent-browser tab <n>        # n번째 탭으로 전환
agent-browser tab close [n]  # 탭 닫기
agent-browser window new     # 새 창 열기
```

### 프레임 및 대화상자

```bash
agent-browser frame <sel>       # iframe 내부로 컨텍스트 전환
agent-browser frame main        # 메인 프레임으로 복귀
agent-browser dialog accept [text]  # 확인 대화상자 수락
agent-browser dialog dismiss    # 대화상자 취소
```

---

## 9. 고급 기능: 네트워크, 세션, 디버깅

### 브라우저 환경 설정

```bash
agent-browser set viewport <w> <h>      # 뷰포트 크기 설정
agent-browser set device "iPhone 14"   # 기기 에뮬레이션
agent-browser set geo <lat> <lng>       # GPS 위치 설정
agent-browser set offline on            # 오프라인 모드 활성화
agent-browser set headers <json>        # HTTP 헤더 추가
agent-browser set credentials <u> <p>  # HTTP 기본 인증
agent-browser set media dark            # 다크 모드 에뮬레이션
```

모바일 기기 에뮬레이션은 iPhone 14를 비롯한 다양한 기기 프로필을 지원하여, 반응형 웹 애플리케이션의 모바일 환경을 정확히 재현할 수 있습니다. GPS 좌표 설정을 통해 위치 기반 서비스의 동작도 검증할 수 있습니다.

### 쿠키와 로컬 스토리지 제어

```bash
agent-browser cookies                    # 모든 쿠키 확인
agent-browser cookies set <name> <val>  # 쿠키 설정
agent-browser cookies clear             # 쿠키 삭제

agent-browser storage local             # 모든 로컬 스토리지 확인
agent-browser storage local <key>       # 특정 키 값 확인
agent-browser storage local set <k> <v> # 값 설정
agent-browser storage local clear       # 로컬 스토리지 초기화
agent-browser storage session           # 세션 스토리지 (동일 방식)
```

로그인 세션 쿠키를 저장하고 불러오는 기능을 활용하면, 매번 인증 절차를 반복하지 않고도 보안 구역 내의 자동화 작업을 이어갈 수 있습니다. 이는 회원가입이 필요한 SaaS 서비스 테스트 자동화에서 특히 유용합니다.

### 네트워크 인터셉트

```bash
agent-browser network route <url>                # 요청 가로채기
agent-browser network route <url> --abort        # 요청 차단 (오프라인 시뮬레이션)
agent-browser network route <url> --body <json>  # 응답 데이터 모킹
agent-browser network unroute [url]              # 라우팅 해제
agent-browser network requests                   # 추적된 요청 목록
agent-browser network requests --filter api     # API 요청만 필터링
```

네트워크 인터셉트 기능은 외부 API 서버 없이도 다양한 응답 시나리오를 시뮬레이션할 수 있게 해줍니다. 오류 응답, 빈 데이터, 예외 케이스 등 실제 운영 환경에서 재현하기 어려운 상황을 자유롭게 테스트할 수 있습니다.

### 스냅샷 비교와 시각적 회귀 테스트

```bash
agent-browser diff snapshot                        # 현재와 마지막 스냅샷 비교
agent-browser diff snapshot --baseline before.txt  # 저장된 스냅샷 파일과 비교
agent-browser diff screenshot --baseline before.png # 픽셀 단위 시각적 차이 비교
agent-browser diff url https://v1.com https://v2.com # 두 URL 스냅샷 비교
```

코드 변경 전후의 화면을 픽셀 단위로 비교하거나 접근성 트리 구조를 대조하는 시각적 회귀 테스트는 UI 컴포넌트의 의도치 않은 변화를 즉각 감지하는 데 핵심적인 기능입니다.

### 디버깅 도구

```bash
agent-browser trace start [path]   # 트레이스 기록 시작
agent-browser trace stop [path]    # 트레이스 기록 중지 및 저장
agent-browser profiler start       # Chrome DevTools 수준 프로파일링 시작
agent-browser console              # 콘솔 메시지(log, error 등) 확인
agent-browser errors               # 페이지 내 JavaScript 예외 오류 확인
agent-browser highlight <sel>      # 화면에서 요소 강조 표시

agent-browser state save <path>    # 인증 상태 저장
agent-browser state load <path>    # 저장된 인증 상태 불러오기
```

브라우저에서 발생하는 JavaScript 예외와 콘솔 로그를 직접 추적하고, Chrome DevTools 수준의 성능 프로파일링까지 수행할 수 있어 프론트엔드 디버깅 워크플로우를 CLI 환경에서 완결할 수 있습니다.

---

## 10. 확장 스킬: Electron 앱과 Slack 자동화

### Electron 기반 데스크톱 앱 제어

agent-browser의 활용 범위는 웹 브라우저를 넘어 Chromium 기반으로 제작된 데스크톱 애플리케이션으로까지 확장됩니다. Visual Studio Code, Slack, Discord, Figma, Notion 등 대부분의 인기 있는 데스크톱 앱들이 Electron 프레임워크로 제작되어 있으며, 내부적으로 Chrome DevTools Protocol(CDP) 포트를 외부에 노출할 수 있습니다.

```bash
# Electron 스킬 설치
npx skills add vercel-labs/agent-browser --skill electron

# CDP 포트를 통해 실행 중인 Electron 앱에 연결
agent-browser connect <port>
```

agent-browser는 이 원격 디버깅 포트를 통해 CDP에 직접 연결하여 데스크톱 앱의 내부 DOM 구조를 파악하고, 웹 자동화와 완전히 동일한 스냅샷 및 상호작용 워크플로우를 데스크톱 화면에 적용합니다. 단순히 화면 좌표를 클릭하는 불안정한 기존 방식을 탈피하여, 내부 구조를 정확히 파악한 견고한 자동화가 가능합니다.

### Slack 메신저 자동화

Slack 자동화 스킬은 복잡한 봇 API 토큰 발급이나 OAuth 설정 없이도 Slack을 자동화할 수 있는 혁신적인 접근법을 제공합니다.

```bash
# Slack 스킬 설치
npx skills add vercel-labs/agent-browser --skill slack
```

에이전트는 이미 로그인되어 있는 Slack 데스크톱 앱 세션에 직접 연결하거나, 새 브라우저 탭에서 웹 버전 Slack을 열어 사용자 인터페이스를 탐색합니다. 스냅샷 참조 시스템을 통해 읽지 않은 메시지 확인, 특정 채널 이동, 대화 내용 검색, 메시지 전송, 스크린샷 캡처 등 모든 작업을 수행합니다.

이 방식은 엄격한 보안 정책으로 외부 API 접근이 제한된 사내 환경에서 특히 강력한 우회로를 제공합니다. 공식 봇 연동이 불가능한 폐쇄적인 엔터프라이즈 환경에서도 화면 제어 기반으로 반복 업무를 자동화할 수 있습니다.

---

## 11. 클라우드 브라우저 인프라 연동

로컬 브라우저 실행이 어려운 서버리스 환경이나 CI/CD 파이프라인에서는 클라우드 브라우저 인프라와 연동하여 agent-browser를 사용할 수 있습니다.

### Browserbase 연동

```bash
export BROWSERBASE_API_KEY="your-api-key"
export BROWSERBASE_PROJECT_ID="your-project-id"

# -p 플래그로 Browserbase 프로바이더 지정
agent-browser -p browserbase open https://example.com

# 또는 환경 변수로 설정
export AGENT_BROWSER_PROVIDER=browserbase
agent-browser open https://example.com
```

Browserbase는 원격 브라우저 인프라를 제공하여, 로컬 환경에서 브라우저를 실행하기 어려운 경우에도 동일한 agent-browser 명령어를 그대로 사용할 수 있게 해줍니다.

### 라이브 스트리밍 (페어 브라우징)

AI 에이전트가 브라우저를 제어하는 과정을 실시간으로 관찰하거나, 사람이 함께 상호작용하는 "페어 브라우징" 모드를 지원합니다.

```bash
# WebSocket 스트리밍 포트 설정
AGENT_BROWSER_STREAM_PORT=9223 agent-browser open example.com
```

이 명령은 지정된 포트에 WebSocket 서버를 기동하여 브라우저 뷰포트를 스트리밍합니다. `ws://localhost:9223`에 연결하면 실시간 프레임을 수신하고 마우스/키보드 입력 이벤트를 전송할 수 있습니다.

---

## 12. 실전 활용 패턴과 워크플로우 예시

### 로그인 자동화

```bash
agent-browser open https://app.example.com/login
agent-browser snapshot -i
agent-browser fill @e1 "user@example.com"
agent-browser fill @e2 "secure_password"
agent-browser click @e3
agent-browser wait --url "**/dashboard"
agent-browser state save auth.json   # 세션 저장 (재사용 가능)
```

### 폼 데이터 입력 자동화

```bash
agent-browser open https://app.example.com/signup
agent-browser snapshot -i
agent-browser fill @e1 "Jane Doe"
agent-browser fill @e2 "jane@example.com"
agent-browser select @e3 "California"
agent-browser check @e4              # 약관 동의 체크박스
agent-browser click @e5              # 제출 버튼
agent-browser wait --load networkidle
agent-browser screenshot result.png
```

### E2E 테스트 자동화 (자기 검증 루프)

```bash
# 코드 빌드 후 자동 브라우저 검증
agent-browser open http://localhost:3000
agent-browser wait --load networkidle
agent-browser snapshot -i -c         # 압축된 인터랙티브 스냅샷
agent-browser screenshot before.png

# 특정 기능 동작 확인
agent-browser click @e2
agent-browser wait --text "Success"
agent-browser snapshot -i
agent-browser diff screenshot --baseline before.png
agent-browser errors                 # JS 오류 확인
```

### 명령어 체이닝

```bash
# && 연산자로 여러 명령을 순차 실행 (출력 확인 불필요 시)
agent-browser open https://example.com && \
agent-browser wait --load networkidle && \
agent-browser screenshot page.png

# 스냅샷 확인 후 상호작용 (순차 실행)
agent-browser fill @e1 "user@example.com" && \
agent-browser fill @e2 "password123" && \
agent-browser click @e3
```

중간 출력을 읽지 않아도 될 때(페이지 오픈 + 대기 + 스크린샷 등)는 `&&`로 체이닝하고, 스냅샷으로 참조 번호를 확인한 후 상호작용하는 경우처럼 이전 명령의 출력을 분석해야 할 때는 명령을 분리하여 실행하는 것이 올바른 패턴입니다.

---

## 13. 보안 기능과 엔터프라이즈 고려사항

agent-browser는 자율 에이전트 환경에서의 보안 리스크를 고려한 여러 방어 메커니즘을 내장하고 있습니다.

### 콘텐츠 경계 마커

```bash
# LLM이 도구 출력과 신뢰할 수 없는 콘텐츠를 구분할 수 있도록 구분자 추가
agent-browser --content-boundaries open https://example.com
```

페이지 출력을 구분자로 감싸서 AI 모델이 도구의 출력과 페이지에서 읽어온 신뢰할 수 없는 콘텐츠를 혼동하지 않도록 방지합니다.

### 도메인 허용 목록

```bash
# 특정 도메인으로만 탐색 제한
agent-browser --allowed-domains "example.com,*.example.com" open https://example.com
```

탐색 허용 도메인을 명시적으로 지정하여 에이전트가 의도치 않은 외부 사이트로 이동하는 것을 방지합니다. 서브도메인 와일드카드(`*.example.com`)도 지원하며, CDN 도메인도 필요에 따라 추가할 수 있습니다.

### 액션 정책 파일

```bash
# 정적 정책 파일로 파괴적 액션 제한
agent-browser --action-policy ./policy.json open https://example.com
```

### 민감한 액션 확인 요청

```bash
# eval, download 등 위험한 액션 실행 전 명시적 승인 요구
agent-browser --confirm-actions eval,download open https://example.com
```

### 출력 길이 제한

```bash
# 컨텍스트 과부하 방지를 위한 최대 출력 길이 설정
agent-browser --max-output 50000 open https://example.com
```

---

## 14. 라이선스 및 생태계 현황

agent-browser 프로젝트는 **Apache License 2.0** 라이선스로 공개 및 배포되어 있습니다. 상업적 사용, 수정, 배포, 특허 사용이 허용되며, 라이선스 및 저작권 고지 의무가 부과됩니다.

프로젝트 출시 이후 AI 코딩 에이전트 생태계에서 빠르게 주목을 받고 있으며, Claude Code, Cursor, GitHub Copilot, Gemini CLI, Codex, Goose, OpenCode, Windsurf 등 주요 AI 개발 도구들과의 공식 호환성이 확인되었습니다. Bash 명령어를 지원하는 환경이라면 운영체제와 플랫폼에 관계없이 사용할 수 있는 범용성이 생태계 확장의 핵심 동력입니다.

공식 리소스:
- GitHub: `https://github.com/vercel-labs/agent-browser`


---

*이 문서는 공개된 기술 문서, GitHub 공식 저장소, 및 사용자 커뮤니티 자료를 바탕으로 작성되었습니다.*  
*작성 일자: 2026-03-05*
