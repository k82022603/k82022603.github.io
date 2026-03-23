---
title: "Everything Claude Code (ECC) 완전 분석 가이드"
date: 2026-03-23 20:20:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  Harness,  sub-agents,  agent-skills,  slash-commands,  async-hooks,  Rules,  MCP,  agent-shield,  Claude.write]
---


> **작성 기준일**: 2026년 3월 23일  
> **대상 버전**: v1.9.0 이상  
> **GitHub**: https://github.com/affaan-m/everything-claude-code  
> **별점**: 72,700+ ⭐ (포크 9,100+)  
> **라이선스**: MIT (무료 오픈소스)

---

## 목차

1. [프로젝트 개요 — 이게 뭔가요?](#1-프로젝트-개요--이게-뭔가요)
2. [탄생 배경 — Anthropic 해커톤 우승작](#2-탄생-배경--anthropic-해커톤-우승작)
3. [핵심 구성요소 전체 해설](#3-핵심-구성요소-전체-해설)
   - 3.1 [에이전트 28개](#31-에이전트-28개)
   - 3.2 [스킬 116개](#32-스킬-116개)
   - 3.3 [슬래시 커맨드 59개](#33-슬래시-커맨드-59개)
   - 3.4 [훅 시스템](#34-훅-시스템)
   - 3.5 [규칙 시스템 (Rules)](#35-규칙-시스템-rules)
   - 3.6 [MCP 서버 설정 14개](#36-mcp-서버-설정-14개)
4. [AgentShield — AI 에이전트 보안 스캐너](#4-agentshield--ai-에이전트-보안-스캐너)
5. [크로스 플랫폼 지원](#5-크로스-플랫폼-지원)
6. [자동 학습 시스템 — Instinct & Skill 진화](#6-자동-학습-시스템--instinct--skill-진화)
7. [설치 방법](#7-설치-방법)
8. [버전 히스토리 — 무엇이 얼마나 바뀌었나](#8-버전-히스토리--무엇이-얼마나-바뀌었나)
9. [Claude Code 최신 기능과의 호환성 평가](#9-claude-code-최신-기능과의-호환성-평가)
10. [토큰 비용 최적화 전략](#10-토큰-비용-최적화-전략)
11. [주의사항 및 한계](#11-주의사항-및-한계)
12. [어떤 사람에게 권하는가?](#12-어떤-사람에게-권하는가)
13. [결론 — 10개월 후에도 현역인가?](#13-결론--10개월-후에도-현역인가)

---

## 1. 프로젝트 개요 — 이게 뭔가요?

Everything Claude Code, 줄여서 **ECC**는 Claude Code를 비롯한 AI 코딩 도구를 훨씬 더 강력하고 체계적으로 사용할 수 있게 해주는 확장팩이다. 공식 명칭은 스스로를 **"에이전트 하네스 성능 최적화 시스템(Agent Harness Performance Optimization System)"** 이라고 정의한다.

비유를 들어보자. Claude Code 자체가 텅 빈 사무실이라면, ECC는 그 사무실에 다음을 한꺼번에 세팅해준다.

- 각자 전문 분야가 다른 **팀원 28명 (에이전트)**
- 업무 매뉴얼 **116권 (스킬)**
- 자주 쓰는 **단축 명령어 59개 (커맨드)**
- 주요 시점마다 자동으로 실행되는 **자동화 스크립트 (훅)**
- 언어별 **코딩 규칙집 (Rules)**
- 외부 도구 연동 **설정 파일 14개 (MCP 설정)**
- 전체 설정을 보안 관점에서 스캔하는 **보안 감사 도구 (AgentShield)**

모든 것이 무료이고 MIT 라이선스다. "기능 구현 계획 세워줘"라고 길게 프롬프트를 쓸 필요 없이 `/plan "사용자 인증 추가"` 한 줄만 치면 planner 에이전트가 알아서 작업 계획을 세운다. 코드 리뷰도 `/review` 한 방이면 된다.

2026년 3월 기준으로 GitHub 별점이 72,700개를 넘어섰으며 포크가 9,100개 이상이다. 커뮤니티 기여자만 30명 이상, 내부 테스트는 992개를 넘어선다. 단순한 설정 모음집이 아니라 실전에서 갈고닦은 종합 시스템으로 진화했다.

---

## 2. 탄생 배경 — Anthropic 해커톤 우승작

ECC는 **Affaan Mustafa**와 그의 파트너 **DRodriguezFX**가 **2025년 9월 Anthropic × Forum Ventures 해커톤**에서 `zenith.chat`이라는 제품을 Claude Code만으로 8시간 만에 만들어 우승하면서 공개된 설정 파일 모음에서 출발했다.

해커톤 직후 공개된 초기 버전은 Claude Code를 잘 쓰는 사람의 CLAUDE.md 템플릿과 설정값을 모아둔 수준이었다. 그러나 Claude Code 자체가 매달 굵직한 기능을 추가하면서, ECC도 함께 빠르게 성장했다. 10개월 이상의 집약적인 일상 사용을 통해 실제 제품을 만들며 실전 검증된 설정들이 계속 추가되었다.

현재는 단순한 설정 파일 묶음을 훨씬 넘어섰다. 두 명의 작성자가 쓴 가이드 문서가 크게 두 편 있는데, **Shorthand Guide**는 설정과 철학을 다루는 입문편이고, **Longform Guide**는 토큰 최적화, 메모리 지속성, 평가(eval), 병렬화를 다루는 고급편이다. 이 두 가이드의 조회수를 합산하면 추적 기준으로 300만 뷰, 플랫폼 전반으로는 1,000만 뷰 이상으로 추정된다.

---

## 3. 핵심 구성요소 전체 해설

ECC 레포지토리의 디렉토리 구조는 각 컴포넌트별로 명확하게 분리되어 있다.

```
everything-claude-code/
├── agents/          # 28개 전문 에이전트
├── skills/          # 116개 워크플로 스킬
├── commands/        # 59개 슬래시 커맨드
├── hooks/           # 라이프사이클 훅
├── rules/           # 언어별 규칙 (공통 + 언어별)
├── scripts/         # 크로스 플랫폼 Node.js 유틸리티
├── mcp-configs/     # 14개 MCP 서버 설정
└── tests/           # 테스트 스위트 (992개+)
```

### 3.1 에이전트 28개

Claude Code는 **서브에이전트(Subagent)** 기능을 지원한다. 메인 Claude에게 일을 시키면, Claude가 "이 작업은 전문가에게 맡기는 게 낫겠다"고 판단해서 별도의 AI 인스턴스를 띄워 위임하는 방식이다. 위임받은 에이전트는 자신만의 격리된 컨텍스트 공간에서 작업하고, 결과물만 메인 에이전트에게 돌려준다.

ECC는 이 서브에이전트를 28개 미리 정의해두었다. 각 에이전트는 마크다운 파일에 YAML 프론트매터 형식으로 이름, 설명, 사용 가능한 도구, 권한 수준, 모델이 명시된다. 아래는 주요 에이전트들이다.

**개발 계획 및 설계 에이전트**

- **planner**: 기능 구현 계획을 세운다. 어떤 작업을 먼저 하고 어떤 것을 나중에 할지 순서를 결정하며, 의존성을 분석해 작업을 체계적으로 분해한다.
- **architect**: 시스템 전체의 설계를 검토하고 판단한다. "이 구조가 지금 요구사항에 맞는가?", "확장성 측면에서 문제는 없는가?"를 평가한다.
- **tdd-guide**: 테스트를 먼저 작성하고, 코드를 나중에 작성하는 TDD(테스트 주도 개발) 방법론을 안내하고 집행한다.

**코드 품질 에이전트**

- **code-reviewer**: 코드 품질, 가독성, 유지보수성, 보안을 종합적으로 리뷰한다.
- **typescript-reviewer**: TypeScript 전용 코드 리뷰어로, 타입 안전성, 인터페이스 설계, 제네릭 활용 등 TypeScript에 특화된 관점으로 검토한다.
- **security-reviewer**: 보안 취약점 전용 리뷰어다. OWASP Top 10, 인증/인가 결함, 민감 데이터 노출 등을 점검한다.

**특수 목적 에이전트**

- **pytorch-build-resolver**: 딥러닝 환경에서 자주 발생하는 PyTorch 빌드 충돌과 의존성 문제를 전문적으로 진단하고 해결한다.
- **market-researcher**: 시장 조사 보고서 작성을 돕는 에이전트로, 경쟁사 분석, 산업 트렌드, 사용자 세그먼트 분석을 수행한다.
- **instinct-creator**: ECC의 자동 학습 시스템과 연동되어, 반복되는 패턴으로부터 새로운 "본능(Instinct)"을 생성한다.

각 에이전트는 사용할 수 있는 도구(Read, Write, Bash, Grep, Glob 등), 활용할 모델(Sonnet, Opus 등), 파일 접근 권한 범위가 각기 다르게 제한되어 있다. 이는 보안과 성능 양측에서 모두 중요한 설계다. 일반 코드 리뷰 에이전트가 프로덕션 데이터베이스에 직접 쓰기 접근을 하거나, 시장 조사 에이전트가 파일 시스템을 마음대로 수정하는 상황을 방지한다.

### 3.2 스킬 116개

스킬(Skill)은 "이 작업은 이렇게 수행하라"는 구조화된 매뉴얼이다. 마크다운 파일 하나에 언제 사용할지(When to Use), 어떻게 동작하는지(How It Works), 단계별 절차(Steps), 주의사항(Cautions), 예시(Examples)가 담겨 있다. Claude Code는 관련 작업을 감지하면 자동으로 해당 스킬을 로드해서 따른다.

ECC에 포함된 116개의 스킬을 카테고리별로 살펴보면 다음과 같다.

**코딩 워크플로 스킬**

- **search-first**: 새 코드를 작성하기 전에 항상 기존 코드베이스를 먼저 검색하도록 강제한다. 중복 구현을 줄이고 기존 패턴을 재활용하는 습관을 만든다.
- **tdd-workflow**: TDD 사이클(Red → Green → Refactor)을 단계별로 안내한다. 실패하는 테스트를 먼저 작성하고, 최소한의 코드로 통과시킨 뒤, 리팩토링하는 흐름을 체계화한다.
- **strategic-compact**: Claude Code의 컨텍스트 윈도우가 95%에 도달하면 자동 압축이 일어나는데, 그 전에 논리적 체크포인트에서 `/compact`를 수동 실행하도록 제안한다. 이 스킬은 언제 압축하는 것이 최적인지를 판단하는 의사결정 가이드다.
- **skill-stocktake**: 기존에 설치된 스킬들을 전체 점검하고, 중복되거나 노후화된 스킬을 정리하는 절차를 안내한다.
- **continuous-agent-loop**: 자율 실행 루프를 설계하고, 에이전트가 스스로 작업을 반복하며 목표를 달성하도록 오케스트레이션하는 방법을 제공한다.

**언어 및 프레임워크별 스킬**

- **swift-actor-persistence**: Swift에서 Actor 모델과 데이터 지속성을 함께 구현할 때의 패턴을 안내한다.
- **swift-protocol-di-testing**: Swift의 프로토콜 기반 의존성 주입과 테스트 전략을 다룬다.
- **regex-vs-llm-structured-text**: 정규식으로 처리할 것인지, LLM으로 처리할 것인지를 선택하는 기준을 제시한다. 비용, 정확도, 유지보수성을 기준으로 상황별 최적 선택지를 안내한다.
- **content-hash-cache-pattern**: 콘텐츠 해시 기반 캐싱 패턴을 구현할 때의 베스트 프랙티스를 담고 있다.
- **cost-aware-llm-pipeline**: 토큰 비용을 의식하면서 LLM 파이프라인을 설계하는 방법론이다. 어느 단계에 어떤 모델을 쓸지, 캐싱은 어디서 할지, 배치 처리는 언제 유리한지를 안내한다.

**비즈니스 및 콘텐츠 스킬 (v1.7.0에서 추가)**

- **market-research**: 시장 조사 보고서를 체계적으로 작성하는 절차를 담고 있다. 경쟁사 분석 프레임워크, 사용자 인터뷰 가이드, 보고서 형식까지 포함한다.
- **investor-materials**: 투자자용 자료(피치덱, 재무 모델, IR 자료)를 작성할 때의 규칙과 형식을 안내한다.
- **article-writing**: 기술 아티클을 작성하는 구조와 스타일 가이드를 제공한다.
- **content-engine**: 콘텐츠 마케팅 파이프라인을 설계하고, 반복 가능한 콘텐츠 생산 워크플로를 구축하는 방법을 안내한다.
- **frontend-slides**: 외부 의존성 없이 순수 HTML과 CSS만으로 프레젠테이션 슬라이드를 만드는 스킬이다. Reveal.js나 별도 도구 없이도 폴리시 있는 발표 자료를 만들 수 있다.

**학습 및 시스템 스킬**

- **analyze-repo**: 레포지토리 전체를 분석해서 코드베이스의 구조, 패턴, 기술 부채, 개선 포인트를 요약 보고한다.
- **workflow-discovery**: 현재 프로젝트에서 반복적으로 수행되는 작업 패턴을 탐지하고, 자동화하거나 스킬화할 수 있는 후보를 찾아낸다.
- **pattern-detection**: 코드베이스 전반에서 안티패턴이나 일관성 없는 구현을 발견한다.
- **learning-dna**: 개발자의 개인적인 코딩 패턴과 선호를 학습하고, 그에 맞는 제안을 개인화한다.

### 3.3 슬래시 커맨드 59개

커맨드는 Claude Code 세션 안에서 `/커맨드명` 형태로 즉시 실행할 수 있는 단축 명령어들이다. 에이전트와 스킬을 조합해서 복잡한 작업을 단 한 줄 입력으로 시작할 수 있게 해준다. 자주 쓰이는 것들을 나열해보면 다음과 같다.

- **/plan**: 구현할 기능의 계획을 세운다. `planner` 에이전트가 실행된다.
- **/tdd**: TDD 방식으로 개발을 시작한다. 테스트를 먼저 짜는 흐름을 안내받는다.
- **/review**: 현재 코드를 종합 리뷰한다. 품질, 보안, TypeScript 규칙 등을 점검한다.
- **/security-scan**: AgentShield를 통해 현재 설정 파일들의 보안 취약점을 스캔한다.
- **/harness-audit**: ECC 하네스 전체의 신뢰도와 위험도를 점수화해서 기준선을 파악한다.
- **/quality-gate**: 지정된 경로를 엄격 모드로 품질 검사한다. 기준을 통과하지 못하면 진행이 차단된다.
- **/compact**: 컨텍스트 윈도우를 전략적으로 압축한다.
- **/instinct-import**: 다른 프로젝트나 팀원으로부터 학습된 Instinct를 가져온다.
- **/evolve**: 누적된 Instinct 3개 이상이 모이면 이를 재사용 가능한 스킬로 진화시킨다.
- **/codex-setup**: OpenAI Codex CLI 호환을 위한 `codex.md` 설정 파일을 자동 생성한다.

### 3.4 훅 시스템

훅(Hook)은 Claude Code의 라이프사이클 이벤트에 특정 동작을 결합하는 장치다. ECC 이전까지는 이를 취약한 bash 인라인 명령어로 처리했으나, v1.8.0부터 **크로스 플랫폼 Node.js 스크립트**로 교체되어 Windows, macOS, Linux 모두에서 안정적으로 동작한다.

ECC가 활용하는 라이프사이클 훅은 다음과 같다.

- **PreToolUse**: 도구를 실행하기 전에 실행된다. 위험한 명령어를 사전에 차단하거나, 품질 게이트를 통과시키는 데 사용된다.
- **PostToolUse**: 도구 실행 후에 작동한다. 결과를 검증하거나 후처리하는 데 활용된다.
- **SessionStart**: 세션이 시작될 때 실행된다. 환경 변수 설정, 배너 출력, 프로젝트 컨텍스트 로드에 사용된다.
- **SessionEnd / Stop**: 세션이 종료될 때 실행된다. 학습된 패턴을 저장하고, 세션 요약본을 생성하며, 상태를 보존한다.
- **PreCompact**: 컨텍스트가 압축되기 직전에 실행된다. 중요한 상태 정보를 압축 이전에 따로 저장해 손실을 방지한다.

훅의 엄격도는 환경변수 하나로 조절 가능하다.

```bash
export ECC_HOOK_PROFILE=minimal   # 낮은 오버헤드, 기본 기능만
export ECC_HOOK_PROFILE=standard  # 대부분의 팀에게 적합한 기본 설정
export ECC_HOOK_PROFILE=strict    # 최대 품질 게이트, 엄격한 검사

# 특정 훅만 비활성화하고 싶을 때
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder"
```

### 3.5 규칙 시스템 (Rules)

규칙(Rules)은 Claude가 항상 따라야 하는 코딩 가이드라인이다. ECC는 이를 두 레이어로 나눠 관리한다.

첫 번째는 **공통 규칙**이다. 언어와 무관하게 모든 프로젝트에 적용된다. 비밀 키를 절대 하드코딩하지 않는다, 이뮤터블(불변) 패턴을 항상 따른다, 파일 하나의 길이는 800줄을 넘지 않는다(권장 200~400줄), 에러 처리를 모든 레이어에서 수행한다, 기능 및 도메인별로 파일을 조직한다 등이 여기에 해당한다.

두 번째는 **언어별 규칙**이다. 12개 프로그래밍 언어를 지원하며, 각 언어의 관용구(idiom)와 베스트 프랙티스를 반영한 규칙이 분리되어 있다. 지원 언어는 **TypeScript, Python, Go, Swift, Java, Kotlin, PHP, Perl, C++, Rust, JavaScript, Ruby**다.

규칙 파일들은 Claude Code 플러그인 시스템이 자동으로 배포하지 못하는 컴포넌트이므로, 별도 설치가 필요하다.

### 3.6 MCP 서버 설정 14개

MCP(Model Context Protocol)는 AI 에이전트가 외부 도구 및 서비스와 통신하는 표준 프로토콜이다. ECC는 자주 사용되는 14개의 MCP 서버 설정을 미리 준비해두었다. GitHub, Slack, Google Drive, Notion, PostgreSQL, Redis, Stripe, Linear 등과의 연동 설정이 여기에 포함된다.

단, MCP 서버를 너무 많이 동시에 활성화하면 심각한 문제가 발생할 수 있다. Claude Code의 컨텍스트 윈도우는 20만 토큰인데, MCP 서버를 과도하게 활성화하면 도구 설명 텍스트만으로 이 공간이 7만 토큰 수준까지 줄어들 수 있다. ECC의 권장 설정은 다음과 같다.

- 설정 파일에 최대 20~30개의 MCP를 등록해두되,
- 프로젝트별로 실제 활성화하는 MCP는 10개 이내로 유지하고,
- 활성 도구 수를 80개 미만으로 관리한다.

---

## 4. AgentShield — AI 에이전트 보안 스캐너

ECC에서 가장 주목할 만한 독자적 기능이다. AI 코딩 도구를 사용하다 보면 설정 파일이 점점 늘어난다. CLAUDE.md, settings.json, MCP 서버 설정, 훅 스크립트, 에이전트 정의 파일 등이 프로젝트마다 쌓이는데, 이 파일들 안에 보안 허점이 존재하면 실제 위협으로 이어질 수 있다.

**AgentShield**는 이 문제를 해결하기 위해 만들어진 AI 에이전트 설정 전용 보안 스캐너다. 단순한 패턴 매칭이 아니라 Opus 4.6 모델 세 개를 활용한 **적대적 추론(Adversarial Reasoning)** 방식을 채택하고 있다. 총 **102개의 보안 규칙**, **1,282개의 테스트**를 보유한다.

### 스캔 대상 및 5개 카테고리

AgentShield가 검사하는 파일과 카테고리를 구체적으로 살펴보면 다음과 같다.

1. **시크릿 감지 (Secrets Detection)**: API 키, 데이터베이스 비밀번호, JWT 시크릿, AWS 자격증명 등 14가지 패턴의 민감 정보 노출을 탐지한다.

2. **권한 감사 (Permission Auditing)**: 에이전트에게 부여된 권한이 필요 이상으로 넓은지 확인한다. 최소 권한 원칙(Principle of Least Privilege)을 기준으로 평가한다.

3. **훅 인젝션 분석 (Hook Injection Analysis)**: 훅 스크립트에 프롬프트 인젝션 공격이 가능한 취약점이 있는지 검사한다. 외부 콘텐츠가 훅의 실행 흐름을 변경할 수 있는 경우를 잡아낸다.

4. **MCP 서버 리스크 프로파일링 (MCP Server Risk Profiling)**: 연동된 MCP 서버들의 신뢰도와 접근 권한 범위를 평가한다. 서드파티 MCP 서버를 여러 개 연결해서 사용하는 환경에서 특히 중요하다.

5. **에이전트 설정 리뷰 (Agent Config Review)**: 각 에이전트 정의 파일에 잘못된 권한 설정, 과도한 도구 접근, 안전하지 않은 지시 내용이 있는지 점검한다.

### 사용 방법

```bash
# 기본 스캔 (설치 없이 npx로 즉시 실행)
npx ecc-agentshield scan

# 특정 경로 스캔
npx ecc-agentshield scan --path ~/.claude/

# 안전한 문제는 자동 수정
npx ecc-agentshield scan --fix

# JSON 형식 출력 (CI 파이프라인 통합용)
npx ecc-agentshield scan --format json

# 마크다운 리포트 생성
npx ecc-agentshield scan --format markdown

# Opus 4.6 3-에이전트 심층 분석 (스트리밍 출력)
npx ecc-agentshield scan --opus --stream

# 초기 보안 설정 파일 생성
npx ecc-agentshield init
```

### `--opus` 플래그: 레드팀/블루팀/감사 파이프라인

`--opus` 플래그를 사용하면 단순한 규칙 매칭을 넘어서는 심층 분석이 진행된다. Claude Opus 4.6 에이전트 세 개가 동시에 투입되어 각각의 역할을 수행한다.

- **공격자 에이전트 (Attacker)**: 악의적인 공격자의 관점에서 취약점 체인을 찾는다. 프롬프트 인젝션, 권한 에스컬레이션, 설정 파일 조작 등의 실제 공격 경로를 시뮬레이션한다.
- **방어자 에이전트 (Defender)**: 기존 보안 장치들이 공격자의 발견 사항에 대해 충분한 방어력을 제공하는지 평가한다. 구체적인 수정 방안을 제시한다.
- **감사관 에이전트 (Auditor)**: 공격자와 방어자 양측의 주장을 종합해서 최종 보안 등급(A~F 등급)을 산출하고, 우선순위가 매겨진 개선 권장사항 목록을 작성한다.

결과물은 터미널 색상 코드, JSON, 마크다운, HTML 형식 중 하나로 출력할 수 있어 CI/CD 파이프라인에 통합하기도 용이하다.

Claude Code 설정 파일뿐 아니라 `.cursorrules`, `agents.json` 등 다른 AI 도구의 설정 파일도 스캔 가능하다.

---

## 5. 크로스 플랫폼 지원

ECC의 또 다른 강점은 하나의 설정 파일이 여러 AI 코딩 도구에서 동시에 작동한다는 점이다. 지원하는 도구와 해당 설정 디렉토리는 다음과 같다.

| 도구 | 제작사 | 설정 경로 | 비고 |
|------|--------|-----------|------|
| **Claude Code** | Anthropic | `.claude/` | 가장 깊이 통합된 메인 플랫폼 |
| **Codex CLI** | OpenAI | `.codex/` | `/codex-setup` 커맨드로 `codex.md` 자동 생성 |
| **Cursor** | Cursor Inc. | `.cursorrules` | 설치 시 자동 감지 후 `.cursorrules` 생성 |
| **OpenCode** | OpenCode | `.opencode/` | 20개 이상의 추가 이벤트 훅 지원 |
| **Cowork** | Anthropic | - | 최근 추가된 데스크탑 자동화 도구 |

이 수준의 크로스 플랫폼 지원은 ECC 외에서는 찾기 어렵다. 회사나 팀에서 도구를 혼용하는 상황(예: 개인은 Claude Code, CI는 Codex, 팀은 Cursor를 쓰는 경우)에서 동일한 설정 기준을 유지할 수 있다는 의미다.

또한 **GitHub Marketplace**에 **ECC Tools**라는 이름의 GitHub 앱으로도 등록되어 있으며, 무료/프로/엔터프라이즈 티어를 지원한다. 공개 레포지토리는 무료 티어로 PR 스캐닝, 커뮤니티 스킬 사용이 가능하고, 비공개 레포지토리와 고급 AgentShield 스캐닝, 팀 설정 공유는 유료 플랜에서 제공된다.

---

## 6. 자동 학습 시스템 — Instinct & Skill 진화

ECC가 단순한 정적 설정 파일 묶음을 넘어서는 결정적인 이유가 여기에 있다. ECC는 사용할수록 스스로 발전하는 **크로스 세션 지식 축적 시스템**을 갖추고 있다.

### 1세대: 스킬 기반 학습 (v1)

세션이 종료되는 Stop 훅이 실행될 때, 해당 세션에서 반복된 코딩 패턴을 자동으로 추출하여 `~/.claude/skills/learned/` 경로에 저장한다. 다음 세션이 시작되면 이 학습된 스킬들이 로드되어 이전 세션의 패턴이 이어진다. 1세대 방식은 학습 가능한 패턴의 약 50~80%를 커버한다.

### 2세대: Instinct 기반 학습 (v2)

1세대보다 훨씬 정교한 방식이다. PreToolUse와 PostToolUse 훅을 통해 도구 실행 전후의 모든 상호작용을 100% 관찰한다. 각각의 학습 단위는 **Instinct(본능)** 이라고 부른다. 각 Instinct에는 0.3~0.9 범위의 **신뢰도 점수**가 부여된다.

Instinct가 3개 이상 관련된 패턴으로 축적되면, `/evolve` 커맨드를 실행해서 이 Instinct들을 하나의 재사용 가능한 **스킬 모듈**로 통합할 수 있다. 성공한 패턴은 신뢰도 점수가 올라가고, 실패하거나 비효율적인 패턴은 가중치가 낮아진다.

팀 단위에서는 `Instinct 라이브러리`를 서로 공유하고 가져올 수 있다. 한 개발자가 오랜 시간 쌓아온 노하우를 팀 전체로 이전할 수 있다는 뜻이다. `/instinct-import` 커맨드로 다른 팀원의 Instinct 라이브러리를 가져오면, 그 개발자의 숙련도가 즉시 팀에 복제된다.

이 학습 시스템은 AI 코딩 어시스턴트 설정 공간에서 진정으로 새로운 기여다. 대부분의 설정 시스템은 한 번 세팅하면 수동으로 유지보수하는 정적 방식인 반면, ECC의 학습 레이어는 동적으로 자기 개선하는 구조다.

---

## 7. 설치 방법

### 방법 1: 플러그인으로 설치 (가장 빠름)

Claude Code 세션 안에서 아래 두 줄을 순서대로 입력한다.

```bash
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

또는 `~/.claude.json`에 직접 설정을 추가할 수도 있다.

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

이 방법으로는 에이전트, 스킬, 커맨드, 훅이 한 번에 설치된다. 단, **Claude Code 플러그인 시스템은 Rules(코딩 규칙)를 자동으로 배포하지 못한다**는 제약이 있다. 규칙은 별도 설치가 필요하다.

### 방법 2: npm/bun으로 선택적 설치 (권장)

v1.9.0에서 도입된 **선택적 설치 아키텍처**를 활용한다. `install-plan.js`와 `install-apply.js` 스크립트로 원하는 컴포넌트만 골라서 설치할 수 있다.

```bash
# 저장소 클론
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code
npm install

# 언어 선택해서 설치 (TypeScript 예시)
./install.sh typescript

# Windows 환경
.\install.ps1 typescript

# 여러 언어 동시 설치
./install.sh typescript python golang

# 프로필별 설치 (core, developer, security, full)
./install.sh --profile developer
```

**4가지 설치 프로필**은 다음과 같다.

- **core**: 가장 핵심적인 스킬, 에이전트, 훅만 설치한다. 처음 시작하는 사람에게 적합하다.
- **developer**: 개발 워크플로 전반을 커버하는 구성이다. 대부분의 개인 개발자에게 권장된다.
- **security**: AgentShield와 보안 관련 스킬 및 에이전트를 포함한다. 프로덕션에 AI 에이전트를 배포하는 팀에게 필수다.
- **full**: 모든 컴포넌트를 설치한다. 컨텍스트 윈도우 소모가 크므로 관리에 주의가 필요하다.

### 방법 3: 수동 선택 설치

레포지토리를 클론한 후 필요한 파일만 골라서 프로젝트의 `.claude/` 디렉토리에 복사하는 방법이다. 116개 스킬 중 10개만, 28개 에이전트 중 5개만 원하는 경우 이 방법이 가장 깔끔하다.

```bash
git clone https://github.com/affaan-m/everything-claude-code.git

# 에이전트 복사
cp everything-claude-code/agents/planner.md ~/.claude/agents/
cp everything-claude-code/agents/code-reviewer.md ~/.claude/agents/

# 스킬 복사
cp -r everything-claude-code/skills/tdd-workflow/ ~/.claude/skills/
cp -r everything-claude-code/skills/search-first/ ~/.claude/skills/

# 커맨드 복사
cp everything-claude-code/commands/plan.md ~/.claude/commands/
```

### 방법 4: GitHub Extension으로 설치

```bash
gh ext install affaan-m/everything-claude-code
```

---

## 8. 버전 히스토리 — 무엇이 얼마나 바뀌었나

ECC가 "설정 모음"에서 "시스템"으로 어떻게 진화했는지를 버전별로 살펴보면 다음과 같다.

**초기 버전 (2025년 9월 해커톤 직후)**  
Anthropic × Forum Ventures 해커톤 우승 후 공개된 초기 버전. CLAUDE.md 템플릿과 좋은 설정값을 모아둔 수준이었다.

**v1.7.0**  
비즈니스 스킬 5개 추가 — 아티클 작성, 콘텐츠 엔진, 투자자 자료, 시장 리서치 등. 코딩 도구에서 지식 작업 도구로의 확장을 시도했다.

**v1.8.0**  
이름 자체가 달라졌다. 이전까지는 "설정 번들"이었다면 이 버전부터 "에이전트 하네스 성능 최적화 시스템"을 공식 명칭으로 채택했다. 주요 변화는 세 가지다. 첫째, 취약한 bash 인라인 훅 명령어를 안전한 크로스 플랫폼 Node.js 스크립트로 교체했다. 둘째, Claude Code, Cursor, OpenCode, Codex(맥 앱 + CLI)가 훅, 커맨드, 스킬 실행 방식에서 더 긴밀하게 통합되었다. 셋째, 세션 시작 시 root 경로 해석 오류와 세션 요약이 비어있는 버그가 수정되었다.

**v1.9.0**  
가장 최신 버전으로 여러 의미에서 도약했다. 내부 테스트 992개로 확장되었고, Codex CLI 지원이 강화되었다. 7개의 새 스킬(search-first, swift-actor-persistence, swift-protocol-di-testing, regex-vs-llm-structured-text, content-hash-cache-pattern, cost-aware-llm-pipeline, skill-stocktake)이 추가됐다. AgentShield가 공식 통합되어 1,282개의 테스트와 102개의 보안 규칙을 갖추었으며, GitHub Marketplace에 ECC Tools 앱이 등록되어 무료/프로/엔터프라이즈 티어를 제공한다. 30명 이상의 커뮤니티 기여자로부터 30개 이상의 PR이 병합되었다.

커밋 수는 731개 이상. 활발한 업데이트가 지속되고 있다.

---

## 9. Claude Code 최신 기능과의 호환성 평가

Claude Code 자체가 매달 굵직한 기능을 추가하고 있는 상황에서, ECC가 얼마나 잘 따라가고 있는지 솔직하게 평가해보면 다음과 같다.

### 잘 따라가고 있는 부분

**플러그인 마켓플레이스 완전 지원**  
Claude Code가 공식 플러그인 생태계를 열자마자 올라탔다. 공식 마켓플레이스에 등록되어 있으며, GitHub Marketplace 앱도 운영 중이다.

**라이프사이클 훅의 심도 있는 활용**  
PreToolUse, PostToolUse, SessionStart, SessionEnd, PreCompact, Stop의 모든 주요 이벤트에 동작이 걸려 있다. 훅이 Node.js 스크립트로 교체되어 Windows, macOS, Linux에서 안정적으로 작동한다. 환경변수 하나로 엄격도를 조절하는 설계는 실전에서 매우 유용하다.

**서브에이전트 28개의 역할 분리**  
Claude Code의 서브에이전트 기능을 가장 체계적으로 활용하는 오픈소스 프로젝트 중 하나다. 각 에이전트가 고유한 도구 세트, 권한 수준, 모델을 가지고 있어 "전문가 집단"처럼 운용된다.

**크로스 플랫폼의 독보적 커버리지**  
Claude Code, Codex, Cursor, OpenCode, Cowork 다섯 도구에서 동일한 설정이 작동한다는 점은 ECC만의 강점이다.

**스킬 시스템과의 완전한 정렬**  
Claude Code 공식 스킬 표준(Agent Skills Open Standard)을 따르며, 커스텀 커맨드가 스킬로 통합된 변화에도 대응되어 있다.

### 아직 반영되지 않은 부분

**Agent Teams(다중 에이전트 협업)**  
Claude Code가 실험적 기능으로 도입한 Agent Teams는 여러 AI 에이전트가 팀을 이루어 서로 직접 대화하면서 협업하는 방식이다. ECC의 28개 에이전트는 모두 "메인 에이전트에게 보고하는" 수직적 구조다. 에이전트 A가 발견한 것을 에이전트 B에게 직접 전달하는 수평적 팀 협업은 아직 구현되지 않았다. 다만, Agent Teams 자체가 아직 실험 단계인 만큼 안정화를 기다리는 전략일 수 있다.

**Worktree 격리의 네이티브 통합**  
Claude Code의 `--worktree` 플래그는 여러 작업을 파일 충돌 없이 병렬로 처리할 수 있게 해준다. ECC의 가이드 문서에서는 worktree 병렬화를 언급하지만, 실제 설정이나 훅에 네이티브하게 통합된 형태는 아직 보이지 않는다.

**Claude Code Skills 2.0 자동 테스트**  
Anthropic이 claude.ai 설정 화면 측에서 제공하는 "스킬 자동 테스트 + 블라인드 비교" 기능과 ECC는 서로 다른 레이어에서 작동한다. ECC는 파일 시스템 기반의 `/harness-audit`와 `/quality-gate`로 유사한 문제를 해결하는 자체 접근법을 갖고 있어, 직접적인 충돌은 없지만 완전히 통합되지도 않은 상태다.

---

## 10. 토큰 비용 최적화 전략

ECC는 Claude Code의 비용 문제를 매우 진지하게 다룬다. Claude Code를 무분별하게 사용하면 토큰 비용이 급격히 높아질 수 있기 때문이다. ECC에 내장된 비용 최적화 전략들을 살펴보면 다음과 같다.

**모델 선택 설정**  
기본 설정으로 메인 모델은 Sonnet, 소형 작업은 Haiku를 사용하도록 설정되어 있다.

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

`MAX_THINKING_TOKENS`를 10,000으로 제한하면 확장 추론에 쓰이는 토큰이 통제된다. `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`를 50으로 설정하면 기본 95%가 아닌 50% 시점에 자동 압축이 이루어져 컨텍스트 공간을 효율적으로 유지한다.

**strategic-compact 스킬**  
단순히 자동 압축에 의존하지 않고, 논리적으로 의미 있는 작업 완료 시점에 수동으로 `/compact`를 실행하도록 안내한다. 자동 압축은 코드 중간에서 발생할 수 있지만, 전략적 압축은 기능 단위나 파일 단위 완료 후에 실행된다.

**MCP 과도 활성화 방지**  
MCP 서버를 너무 많이 동시에 활성화하면 도구 설명 텍스트만으로 20만 토큰 윈도우가 7만 토큰으로 줄어든다는 사실을 문서 곳곳에서 경고한다.

**search-first 스킬**  
코드를 새로 작성하기 전에 기존 코드를 먼저 검색하도록 강제한다. 이미 있는 코드를 다시 생성하는 낭비를 줄인다.

**cost-aware-llm-pipeline 스킬**  
LLM 파이프라인에서 어떤 단계에 어떤 모델을 쓸지 의식적으로 선택하도록 안내한다. 단순한 분류 작업에 Opus를 쓰거나, 보일러플레이트 코드 생성에 비싼 모델을 투입하는 실수를 방지한다.

---

## 11. 주의사항 및 한계

아무리 훌륭한 도구라도 한계와 주의사항이 있다.

**컨텍스트 윈도우 소모**  
116개의 스킬, 28개의 에이전트, 59개의 커맨드 정의가 모두 Claude의 컨텍스트에 로드되면 대화에 쓸 수 있는 공간이 눈에 띄게 줄어들 수 있다. v1.9.0에서 선택적 설치 아키텍처를 도입한 것도 이 문제를 의식한 결과다. 처음부터 모든 것을 설치하기보다는 core 프로필에서 시작해서 필요한 것만 추가하는 방식을 권장한다.

**Claude Code 업데이트와의 시차**  
Claude Code 자체가 매주 수준으로 빠르게 바뀌고 있다. ECC도 활발하게 업데이트되지만, "방금 나온 Claude Code 신기능"을 ECC가 즉시 지원하기를 기대하기는 어렵다. 시차가 발생하는 건 불가피하다.

**Rules 별도 설치 필요**  
Claude Code 플러그인 시스템의 현재 제약으로 인해 코딩 규칙(Rules) 파일은 플러그인 설치만으로 자동 배포되지 않는다. 별도 설치 단계를 거쳐야 한다. 초보자에게는 다소 불편할 수 있다.

**Windows 설정 시 주의**  
Linux/macOS와 Windows 환경에서 경로 처리 방식이 다르기 때문에 일부 훅 스크립트에서 경고가 발생할 수 있다. v1.8.0에서 Windows 경로 처리가 개선되었으나, 모든 엣지케이스가 해결된 것은 아니다.

**설정 파일 유지보수**  
ECC를 설치하면 관리해야 할 설정 파일이 크게 늘어난다. 처음에는 편리하지만, 장기적으로는 자신의 워크플로에 맞지 않는 스킬이나 에이전트가 노이즈가 될 수도 있다. 정기적으로 `/harness-audit`와 `skill-stocktake` 스킬로 점검하는 습관이 필요하다.

---

## 12. 어떤 사람에게 권하는가?

**강력히 권하는 경우**

Claude Code를 매일 주력 도구로 사용하고 있는데, 매번 유사한 프롬프트를 반복 작성하고 있다면 ECC의 커맨드 시스템이 그 반복을 없애준다. `/plan`, `/tdd`, `/review` 하나로 복잡한 워크플로가 시작된다.

여러 프로그래밍 언어로 프로젝트를 동시에 관리하고 있다면, 12개 언어별 코딩 규칙이 미리 정리되어 있으므로 언어별로 일관된 기준을 유지하기 편하다.

MCP 서버를 여러 개 연결해서 복잡한 에이전트 워크플로를 구성하고 있다면, AgentShield로 보안 취약점을 한번 스캔해보는 것만으로도 충분한 가치가 있다. 의외의 허점이 발견되는 경우가 많다.

Claude Code와 Cursor, 또는 Claude Code와 Codex를 동시에 사용하고 있다면, ECC의 크로스 플랫폼 지원 덕분에 동일한 설정을 여러 도구에 일관되게 적용할 수 있다.

팀으로 AI 코딩 도구를 사용하고 있다면, Instinct 라이브러리 공유 기능을 통해 숙련된 팀원의 노하우를 팀 전체로 이전할 수 있다.

**굳이 권하지 않는 경우**

Claude Code를 주 1~2회 가볍게 사용하는 수준이라면 오버스펙이다. 단순한 한두 가지 기능이 목적이라면, ECC를 전부 설치하는 것보다 필요한 스킬 파일 몇 개만 선택적으로 가져다 쓰는 것이 더 현명하다.

---

## 13. 결론 — 10개월 후에도 현역인가?

답은 **예스**다.

초기 버전이 "Claude Code 파워 유저의 설정 모음집"이었다면, 현재의 ECC v1.9.0은 아래 네 가지를 갖춘 종합 시스템이다.

1. **규모**: 에이전트 28개, 스킬 116개, 커맨드 59개, 992개의 내부 테스트
2. **깊이**: 라이프사이클 훅의 전 단계 통합, 크로스 세션 학습 시스템
3. **범위**: 5개 AI 코딩 도구, 12개 프로그래밍 언어, 비즈니스 스킬까지
4. **보안**: 102개 규칙, 1,282개 테스트의 전용 보안 스캐너

Claude Code 자체가 Agent Teams, Worktree 격리 등 새로운 기능을 계속 쏟아내고 있어 ECC가 모든 것을 즉각 따라가지는 못하지만, 커밋 수 731개, 30명 이상의 커뮤니티 기여자, 72,700개의 GitHub 별점이 이야기하는 것은 명확하다. 이 프로젝트는 살아있고, 계속 발전하고 있으며, 실전에서 쓰이고 있다.

Claude Code를 진지하게 쓰는 개발자라면, 한 번은 들여다볼 가치가 있다.

---

*이 문서는 2026년 3월 23일 기준 최신 정보를 토대로 작성되었습니다.*  
*출처: https://github.com/affaan-m/everything-claude-code | https://ecc.tools*
