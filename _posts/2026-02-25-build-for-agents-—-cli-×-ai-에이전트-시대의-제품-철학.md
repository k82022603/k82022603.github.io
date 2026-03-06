---
title: "\"Build for Agents\" — CLI × AI 에이전트 시대의 제품 철학"
date: 2026-02-25 22:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  AndrejKarpathy,  claude-code-cli,  github-cli,  Claude.write]
---


> 작성일: 2026-02-25  
> 원본 출처: Andrej Karpathy, X(구 트위터) [@karpathy](https://x.com/karpathy/status/2026360908398862478)

> CLIs are super exciting precisely because they are a "legacy" technology, which means AI agents can natively and easily use them, combine them, interact with them via the entire terminal toolkit.
>
>E.g ask your Claude/Codex agent to install this new Polymarket CLI and ask for any arbitrary dashboards or interfaces or logic. The agents will build it for you. Install the Github CLI too and you can ask them to navigate the repo, see issues, PRs, discussions, even the code itself.
>
>Example: Claude built this terminal dashboard in ~3 minutes, of the highest volume polymarkets and the 24hr change. Or you can make it a web app or whatever you want. Even more powerful when you use it as a module of bigger pipelines.
>
>If you have any kind of product or service think: can agents access and use them?
>
>- are your legacy docs (for humans) at least exportable in markdown?
>- have you written Skills for your product?
>- can your product/service be usable via CLI? Or MCP?
>- ...
>
>It's 2026. Build. For. Agents.
>
---

## 1. 이 글의 배경 — Karpathy가 던진 선언

2026년 2월 24일, AI 분야의 최고 권위자 중 한 명인 Andrej Karpathy(전 Tesla AI 디렉터, OpenAI 공동 창업자)가 X에 짧지만 매우 도전적인 메시지를 남겼다.

> *"It's 2026. Build. For. Agents."*

그가 던진 핵심 주장은 단순하다. CLI(Command Line Interface)는 수십 년 된 레거시 기술이지만, 바로 그 이유 때문에 AI 에이전트가 가장 자연스럽게 사용할 수 있는 인터페이스라는 것이다. 새로운 UI를 발명할 필요가 없다. 이미 세상에 존재하는 CLI 도구들이, 에이전트 시대의 가장 강력한 실행 레이어가 된다.

그는 이를 직접 실증해 보였다. Claude 에이전트에게 Polymarket CLI를 설치하고, 거래량 상위 예측 시장의 현황과 24시간 변동폭을 보여주는 터미널 대시보드를 만들어달라고 요청했고, Claude는 **약 3분 만에** 이를 완성했다.

---

## 2. 왜 CLI인가 — "레거시가 곧 장점"이라는 역설

### 에이전트는 텍스트로 세상과 상호작용한다

LLM 기반 에이전트의 본질은 텍스트 입출력이다. 마우스 클릭, 드래그, 시각적 버튼 탐색은 에이전트에게 어색하고 오류가 잦다. 반면 터미널 명령어는 에이전트가 정확하게 생성하고 실행할 수 있는 형태다. CLI와 에이전트는 인터페이스 레벨에서 근본적으로 호환된다.

### LLM의 훈련 데이터에 CLI가 넘친다

GitHub, Stack Overflow, 공식 문서, 기술 블로그에는 수억 개의 터미널 명령어와 사용 예제가 존재한다. LLM은 이미 CLI 사용법을 깊이 학습한 상태다. 새로운 CLI 도구가 나와도, 에이전트는 README 몇 줄만 읽으면 즉시 활용법을 파악하고 실행에 옮길 수 있다.

### Unix 철학과 에이전트의 사고가 일치한다

Unix의 핵심 철학은 "하나의 도구는 하나의 일을 잘 하고, 도구들은 파이프로 연결한다"는 것이다. 이는 에이전트가 복잡한 문제를 분해하고, 여러 도구를 조합해 해결하는 방식과 완벽하게 맞아떨어진다.

```bash
# 에이전트가 자연스럽게 구성하는 파이프라인 예시
polymarket -o json markets list --active true --order volume_num \
  | jq '.[] | select(.volume > 100000000)' \
  | sort -k2 -rn \
  | head -10
```

### 설치와 확장이 한 줄이다

```bash
brew install polymarket
gh extension install github/gh-copilot
cargo install ripgrep
```

에이전트는 새로운 능력이 필요할 때 패키지 관리자 한 줄로 도구를 설치하고 즉시 사용할 수 있다. GUI 앱을 에이전트가 자율적으로 설치하고 조작하는 것과는 차원이 다른 단순함이다.

---

## 3. Polymarket CLI — 에이전트를 위해 설계된 예측 시장 도구

### Polymarket이란

Polymarket은 세계 최대의 탈중앙화 예측 시장 플랫폼이다. 정치, 경제, 스포츠, 과학 등 다양한 주제에 대해 실제 자금으로 확률에 베팅하는 구조를 갖추고 있다. 시장 참여자들의 집단 지성으로 형성된 가격(확률)은 단순 여론조사보다 훨씬 정확한 예측력을 보여주는 것으로 알려져 있으며, 주요 마켓의 총 거래량은 수억 달러에 달한다.

### Polymarket CLI의 특징

Polymarket CLI는 Rust로 작성된 공식 커맨드라인 도구로, Polymarket 개발자 Suhail Kakar가 "AI 에이전트가 예측 시장에 접근하는 가장 빠른 방법"으로 발표했다.

**에이전트 친화적 설계의 핵심은 JSON 출력이다.** 모든 명령어에 `--output json` 옵션이 지원되어 에이전트가 파싱하고 다음 단계로 넘기기에 최적화되어 있다.

```bash
# 지갑 없이 즉시 사용 가능 — 마켓 데이터 조회
polymarket markets list --active true --order volume_num --limit 10

# 특정 주제 검색
polymarket markets search "NBA champion"
polymarket events list --tag politics

# 에이전트/파이프라인용 JSON 출력
polymarket -o json markets list --limit 20

# 특정 마켓 상세 조회
polymarket markets get will-trump-win-the-2024-election

# 오더북 및 가격 확인
polymarket clob price <token_id>
```

지갑이 없어도 마켓 탐색, 가격 조회, 데이터 분석은 즉시 가능하다. 지갑 연결 시에는 주문 생성·취소, 잔고 확인, 실제 거래까지 에이전트가 자율적으로 수행할 수 있다.

---

## 4. GitHub CLI와의 시너지 — 파이프라인의 시작

Karpathy가 Polymarket CLI와 함께 GitHub CLI를 언급한 이유는 명확하다. 두 도구를 조합하면 에이전트가 데이터 수집에서 코드 실행, 이슈 관리까지 이어지는 완전한 자율 파이프라인을 구성할 수 있기 때문이다.

```bash
# GitHub CLI로 저장소 탐색
gh repo view Polymarket/polymarket-cli       # 저장소 개요
gh issue list --repo owner/repo              # 오픈 이슈 목록
gh pr list --state open                      # 진행 중인 PR
gh api repos/owner/repo/contents/src         # 소스코드 직접 탐색

# 에이전트가 구성할 수 있는 조합 파이프라인 예시
# 1. Polymarket에서 특정 이벤트 확률 데이터 수집
# 2. 확률 변화가 임계치를 넘으면 GitHub 이슈 자동 생성
# 3. 관련 코드 수정안 작성 후 PR 자동 제출
```

이 수준의 자동화는 불과 1~2년 전만 해도 복잡한 오케스트레이션 시스템이 필요했다. 지금은 에이전트에게 두 개의 CLI를 쥐여주면 된다.

---

## 5. Claude가 3분 만에 대시보드를 만든 이유

Karpathy가 Claude에게 대시보드 작성을 요청했을 때, 실제로 일어난 일은 단순한 코드 생성이 아니었다. Claude는 다음 워크플로우를 스스로 설계하고 실행했다.

1. Polymarket CLI 설치 방법을 파악하고 자동으로 설치
2. `polymarket -o json markets list` 실행 후 데이터 스키마 분석
3. 거래량 기준 정렬 및 24시간 변동폭 필터링 로직 설계
4. 터미널에 테이블 형태로 렌더링하는 스크립트 작성
5. 실행 권한 설정 후 `~/bin/polymarket-dashboard`로 배포

코드를 작성한 것이 아니라 **전체 인프라를 자율적으로 설계하고 구현**한 것이다. 3분이라는 시간이 보여주는 것은 속도가 아니라, 에이전트 시대에 "개발"의 의미가 근본적으로 달라졌다는 사실이다.

---

## 6. "Build for Agents" — 제품과 서비스를 위한 체크리스트

Karpathy의 글에서 가장 실용적인 부분은 마지막 질문들이다. 그는 제품이나 서비스를 가진 사람이라면 다음을 자문해보라고 한다.

### ① 레거시 문서를 마크다운으로 내보낼 수 있는가?

PDF, Word, HWP, 독점 포맷에 갇힌 문서는 에이전트가 파싱하기 어렵다. 마크다운은 에이전트가 읽고 처리하기 가장 이상적인 텍스트 포맷이다. 제품 매뉴얼, API 문서, 내부 정책서가 마크다운으로 제공되면 에이전트가 즉시 내용을 학습하고 활용할 수 있다.

### ② 제품에 Skills(에이전트용 사용 가이드)를 작성했는가?

Claude의 Skills, Anthropic의 MCP Skills처럼 에이전트에게 "이 제품을 어떻게 써야 하는가"를 설명하는 구조화된 문서가 필요하다. 이것은 사람이 아닌 에이전트를 위한 온보딩 문서다. 잘 쓰인 `SKILL.md` 하나가 에이전트의 제품 활용 수준을 결정한다.

### ③ CLI 또는 MCP로 접근 가능한가?

웹 앱에만 기능이 있는 서비스는 에이전트가 자율적으로 사용하기 어렵다. REST API가 있으면 연결은 가능하지만, CLI나 MCP 서버로 노출되어 있으면 설정 없이 바로 연결된다. Model Context Protocol(MCP)은 에이전트와 서비스 사이의 표준 연결 규격으로 빠르게 자리잡고 있다.

### 실천 우선순위 정리

| 준비 항목 | 핵심 내용 | 우선순위 |
|-----------|-----------|----------|
| 문서 마크다운화 | 모든 문서를 `.md`로 내보내기 가능하게 | 🔴 즉시 |
| JSON 출력 지원 | 모든 API/CLI에 `--output json` 옵션 추가 | 🔴 즉시 |
| CLI 인터페이스 구축 | 핵심 기능을 명령어로 노출, 패키지 배포 | 🔴 높음 |
| MCP 서버 구현 | Claude/Cursor 등 에이전트와 직접 연결 | 🟡 중간 |
| Skills 문서 작성 | 에이전트 전용 `SKILL.md` 작성 및 배포 | 🟡 중간 |
| 레거시 문서 정리 | PDF/HWP → 마크다운 변환 파이프라인 구축 | 🟡 중간 |

---

## 7. 예측 시장이 AI 에이전트 생태계와 갖는 특별한 시너지

Polymarket 같은 예측 시장은 단순한 투자 도구를 넘어, AI 에이전트 시대의 핵심 인프라로 기능할 가능성이 크다. 이유는 세 가지다.

**에이전트가 정보 수집 후 자율적으로 포지션을 취할 수 있다.** 뉴스를 읽고, 분석하고, 시장 확률과 자신의 판단 사이에 괴리가 있다고 판단하면 CLI를 통해 직접 베팅을 실행한다.

**예측 시장 데이터 자체가 에이전트의 의사결정 신호가 된다.** 집단 지성이 형성한 확률 데이터는 에이전트의 전략 실행 조건으로 활용될 수 있다. "연준 금리 동결 확률이 70%를 넘으면 포트폴리오를 재조정하라"는 식의 자동화가 가능하다.

**에이전트 간 정보 시장으로 발전할 수 있다.** 장기적으로 에이전트들이 서로의 예측을 시장에서 검증하고, 신뢰도 높은 에이전트의 판단이 자연스럽게 가격에 반영되는 메커니즘이 형성될 수 있다.

---

## 8. 한국 기업/개발자를 위한 시사점

이 흐름은 국내 기업에도 동일하게 적용된다. 한국 시장의 맥락에서 특히 주목해야 할 부분이 있다.

**레거시 문서 문제가 더 심각하다.** 국내 기업들은 HWP, 독점 그룹웨어, 폐쇄형 인트라넷에 방대한 지식이 갇혀 있는 경우가 많다. 이 문서들이 마크다운으로 변환되지 않으면, 에이전트가 기업의 핵심 지식에 접근할 수 없다.

**CLI 문화가 상대적으로 약하다.** B2B SaaS 제품 대부분이 웹 대시보드 중심으로 설계되어 있다. CLI 인터페이스를 추가하는 것이 당장 사용자 경험에 큰 변화를 주지 않더라도, 에이전트 통합 측면에서는 결정적 차이를 만든다.

**MCP 생태계 참여가 선택이 아닌 필수가 된다.** Anthropic의 Model Context Protocol을 기준으로 SaaS, ERP, 협업 도구들이 에이전트 연결 레이어를 구축하는 흐름이 가속화되고 있다. 국내 서비스들도 이 흐름에 올라타야 한다.

---

## 9. 결론 — 에이전트를 위해 설계하는 것이 곧 미래를 위한 설계다

Karpathy의 메시지는 근본적으로 설계 철학의 전환을 요구한다. 지금까지 제품 설계의 기준은 "사람이 쓰기 편한가?"였다. 앞으로는 여기에 "에이전트가 사용할 수 있는가?"가 반드시 추가되어야 한다.

에이전트가 사용할 수 있는 제품은 인간 사용자와 에이전트 사용자를 동시에 확보한다. 에이전트가 사용할 수 없는 제품은, 자동화와 통합의 흐름에서 점차 뒤처진다.

CLI는 그 시작점으로 가장 낮은 진입장벽을 제공한다. 이미 존재하는 레거시 기술을, 이미 학습된 에이전트와 연결하기만 하면 된다.

*"It's 2026. Build. For. Agents."*

---

## 참고 자료

- [Andrej Karpathy X 게시글 원문](https://x.com/karpathy/status/2026360908398862478)
- [Polymarket CLI GitHub 저장소](https://github.com/Polymarket/polymarket-cli)
- [Polymarket 공식 개발자 문서](https://docs.polymarket.com/)
- [PANews — Polymarket CLI 출시 보도](https://www.panewslab.com/en/articles/019c9059-0116-767a-83b8-25e2dbe7fa84)
- [Anthropic Model Context Protocol 공식 문서](https://docs.anthropic.com/en/build-with-claude/mcp)

---

*작성일: 2026-02-25*
