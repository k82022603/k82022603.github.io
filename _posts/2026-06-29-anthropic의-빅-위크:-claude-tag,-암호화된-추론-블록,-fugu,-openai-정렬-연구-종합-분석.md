---
title: "Anthropic의 빅 위크: Claude Tag, 암호화된 추론 블록, Fugu, OpenAI 정렬 연구 종합 분석"
date: 2026-06-29 14:00:00 +0900
categories: [AI,  Material]
mermaid: [True]
tags: [AI,  claude-tag,  claude-code,  adaptive-thinking,  Sakana,  Alignment,  Claude.write]
---


> **작성 기준 영상**: ["Anthropic Just Replaced Claude Code With New Claude Tag"](https://www.youtube.com/watch?v=hDsZMb_8FYo) (Jun 25, 2026)  
> **출처 채널**: AI Revolution (YouTube)  
> **작성일자**: 2026-06-29

---

## 목차

1. [개요: 왜 이 한 주가 중요한가](#1-개요)
2. [Claude Tag: Slack에 상주하는 팀 AI 에이전트](#2-claude-tag)
   - 2.1 Claude Tag란 무엇인가
   - 2.2 핵심 기능 심층 분석
   - 2.3 기업 거버넌스 — 토큰 버짓과 감사 로그
   - 2.4 실제 데모 시나리오 재현
   - 2.5 Claude Cowork와의 연계
   - 2.6 경쟁 지형
3. [Claude Code 사고 깊이 저하 논란](#3-claude-code-controversy)
   - 3.1 사건의 발단
   - 3.2 Anthropic이 변경한 세 가지
   - 3.3 Patrick McKenna의 발견
4. [Matt Green의 암호화 추론 블록 연구](#4-matt-green)
   - 4.1 연구 배경
   - 4.2 OpenAI reasoning.encrypted_content 구조
   - 4.3 Anthropic thinking.signature 구조
   - 4.4 세션 간 재생 취약점
   - 4.5 사이드 채널 공격
   - 4.6 Green의 권고사항
5. [Sakana AI Fugu: 오케스트레이션 라우터로서의 AI](#5-sakana-fugu)
   - 5.1 Fugu란 무엇인가
   - 5.2 TRINITY와 Conductor 아키텍처
   - 5.3 벤치마크 성능
   - 5.4 가격 정책
   - 5.5 구조적 리스크
6. [OpenAI 정렬 연구: 유익한 RL이 도메인을 초월하는가](#6-openai-alignment)
   - 6.1 연구 배경 — 창발적 오정렬 문제
   - 6.2 연구 방법론
   - 6.3 15가지 유익한 특성
   - 6.4 핵심 실험 결과
   - 6.5 크로스 도메인 전이
   - 6.6 정렬 지속성
7. [종합 시사점 및 결론](#7-conclusion)

---

## 1. 개요: 왜 이 한 주가 중요한가 {#1-개요}

2026년 6월 셋째 주는 AI 산업 전반에서 유난히 밀도 높은 한 주였다. Anthropic은 Claude Tag를 출시하면서 AI 협업의 단위를 개인 대화창에서 팀 채널로 이동시켰고, 동시에 Claude Code의 기본 설정 변경을 둘러싼 논란이 수면 위로 드러났다. 암호화폐 분야에서 명성을 쌓아 온 Johns Hopkins 대학의 암호학 교수 Matt Green은 Claude와 OpenAI 모두의 추론 블록이 암호화된 방식으로 처리된다는 사실을 파헤쳤고, 일본의 Sakana AI는 자체 모델 가중치 없이 타사 프론티어 모델을 오케스트레이션하는 Fugu를 공개했다. OpenAI는 강화학습을 이용해 정렬 특성이 훈련 도메인을 넘어 일반화될 수 있는지를 검증하는 연구 논문을 발표했다.

이 문서는 해당 영상이 다룬 다섯 가지 스토리를 각각 깊이 있게 분석한다.

```mermaid
graph TD
    A["Anthropic 빅 위크 (June 2026)"] --> B["Claude Tag\n팀 AI 에이전트 출시"]
    A --> C["Claude Code 논란\n사고 깊이 67% 저하 의혹"]
    A --> D["Matt Green 연구\n암호화 추론 블록 분석"]
    A --> E["Sakana AI Fugu\n멀티에이전트 라우터"]
    A --> F["OpenAI 정렬 연구\n유익한 RL 일반화"]
    B --> B1["Slack 채널 상주 AI\n멀티플레이어 패러다임"]
    C --> C1["adaptive thinking 비활성화\nredact-thinking 활성화\n노력 수준 하향"]
    D --> D1["세션 간 재생 취약점\n사이드 채널 공격 증명"]
    E --> E1["자체 가중치 없음\nClaude+GPT+Gemini 라우팅"]
    F --> F1["53개 평가 중 44개(83%) 우위\n크로스 도메인 전이 확인"]
```

---

## 2. Claude Tag: Slack에 상주하는 팀 AI 에이전트 {#2-claude-tag}

### 2.1 Claude Tag란 무엇인가

2026년 6월 23일, Anthropic은 Claude Tag를 출시하며 AI 협업의 단위를 개인 대화에서 팀 채널로 전환했다. 기존의 Slack 봇이나 Claude 개인 세션과 근본적으로 다른 점은 채널에 하나의 Claude가 공유 팀원으로서 참여한다는 데 있다. 누구나 `@Claude`를 태그해 작업을 위임할 수 있고, 그 작업이 어디까지 진행되었는지 채널 전체가 볼 수 있다.

Claude Tag는 Anthropic의 인기 있는 에이전트 제품인 Claude Code 및 Cowork와 유사하다. 직원이 Claude Tag에게 작업을 지시하면 봇은 이를 단계별로 분해하고 독립적으로 처리한 후 Slack을 통해 팀에 최종 결과를 전달한다.

Claude Tag는 2026년 6월 23일 Claude Enterprise 및 Team 고객을 대상으로 공개 베타로 출시되었으며, Claude Opus 4.8로 구동된다. 기존의 사용자별 Claude in Slack 앱을 대체하는 지속적이고 멀티플레이어 방식의 AI 팀원이다.

Andrej Karpathy는 이 변화를 조직 전반의 모든 인간 활동 방식에 훨씬 더 부합한다고 평가했다. Anthropic의 자체 제품팀은 내부 버전의 Claude Tag를 사용해 코드의 65%를 생성하고 있으며, Claude Tag 자체를 구축하는 데 쓴 코드의 대부분도 포함된다.

Claude Tag의 출시는 단순한 봇 업그레이드가 아니라, AI와 팀이 공존하는 방식을 재정의하는 패러다임 전환이다. Andrej Karpathy가 "LLM 인터페이스의 세 번째 대변환"으로 표현한 것처럼, 웹 채팅과 데스크톱 앱에 이어 AI가 조직 인프라 속에 지속적으로 내장되어 팀과 협업하는 단계로의 진입을 의미한다.

### 2.2 핵심 기능 심층 분석

**멀티플레이어 공유 컨텍스트**

Claude Tag를 통해 특정 Slack 채널의 모든 구성원이 하나의 Claude 아이덴티티에 접근할 수 있다. 누구나 Claude가 무엇을 하고 있는지 볼 수 있고, 이전 사람이 멈춘 지점부터 대화를 이어갈 수 있다. 이전의 개인 챗봇 패러다임에서는 한 사람이 AI와 나눈 대화를 다른 팀원이 이어받으려면 처음부터 맥락을 다시 설명해야 했다. Claude Tag는 이 맥락 재설명 비용을 제거한다.

**앰비언트 모드 (Ambient Mode)**

Claude Tag에는 앰비언트 동작 기능도 있다. 이를 활성화하면 봇은 직원들에게 조직 전반의 정보를 선제적으로 업데이트하고 잊혀지거나 해결되지 않은 스레드나 작업을 추적한다.

영상에서 강조된 앰비언트 모드의 구체적인 동작은 다음과 같다. Claude는 자신이 포함된 채널의 대화를 상시 모니터링하고, 교착 상태에 빠진 논의, 미해결 이슈, 누군가의 결정이 필요한 사항을 자발적으로 플래그한다. 또한 연결된 도구에서 관련 정보를 수집해 팀이 알아야 할 사항을 사전 통보한다.

**비동기 실행**

장시간이 소요되는 작업을 위임한 뒤 Slack을 완전히 닫아도 Claude는 계속 작업을 진행하고 완료 시 알림을 전송한다. 주간 채널 요약 생성, 긴급 항목 플래그, 일정에 따른 관련자 리마인더 전송 같은 반복 작업을 별도의 관리 없이 지속적으로 실행할 수 있다.

**지속 학습과 채널 기억**

Anthropic에 따르면 Claude는 채널을 팔로우하면서 해당 작업에 대해 점점 더 많이 학습한다. 또한 권한이 부여된 경우 조직의 다른 채널에서 자동으로 정보를 수집할 수도 있다.

```mermaid
flowchart LR
    subgraph CHANNEL["Slack 채널 (공개)"]
        U1["팀원 A\n@Claude 작업 위임"] --> CL["Claude AGENT\n단계별 작업 분해 및 실행"]
        U2["팀원 B\n진행 상황 확인 및 피드백"] --> CL
        U3["팀원 C\n작업 이어받기"] --> CL
        CL --> RESULT["결과물 스레드 게시"]
    end
    subgraph TOOLS["연결 도구"]
        GH["GitHub"]
        JIRA["Jira / Linear"]
        DB["데이터베이스"]
        CRM["CRM 시스템"]
    end
    CL --> GH
    CL --> JIRA
    CL --> DB
    CL --> CRM
    subgraph AMBIENT["앰비언트 모드"]
        MON["채널 상시 모니터링"] --> FLAG["교착 논의 플래그"]
        MON --> NOTIFY["미해결 이슈 알림"]
        MON --> REMIND["관련자 리마인더"]
    end
    CL --> MON
```

### 2.3 기업 거버넌스 — 토큰 버짓과 감사 로그

기업 환경에서 AI 에이전트를 도입할 때 가장 중요한 문제 중 하나는 통제 가능성이다. Claude Tag는 이를 위한 두 가지 핵심 거버넌스 메커니즘을 제공한다.

**Claude 아이덴티티 격리**

시스템 관리자는 Claude가 접근할 수 있는 도구, 정보, 채널을 지정하며, 각 Claude 아이덴티티는 관리자가 정의한 채널로 범위가 제한된다. 예를 들어 법무 업무를 위해 설정된 Claude는 엔지니어링 채널에 기억을 전파할 수 없다.

영상의 시각 자료에서 보여준 토큰 버짓 관리 대시보드는 다음과 같이 계층적 구조로 설계되어 있다. 조직 수준에서 월간 토큰 총량(예: 10.0M 중 8.0M 사용)을 설정하고, 그 하위에 채널별 개별 버짓(예: #engineering 채널 4.0M 중 2.5M, #marketing 채널 2.0M 중 1.2M)을 배분한다.

**완전한 감사 로그**

모든 Claude 작업과 그것을 트리거한 사람의 기록이 완전한 감사 로그로 보존된다. 영상에서 보여준 감사 로그 예시에는 채널 생성, 토큰 버짓 업데이트, API 키 취소 같은 민감한 작업들이 행위자 및 타임스탬프와 함께 기록되어 있었다.

Anthropic은 Claude Tag가 Slack에서 시작되는 이유를 이 플랫폼이 팀과 AI 간 협업 작업의 자연스러운 장소이기 때문이라고 설명했다.

### 2.4 실제 데모 시나리오 재현

영상에서 Anthropic이 공개한 데모는 실제 소프트웨어 개발 플로우를 그대로 담고 있어 Claude Tag의 실용성을 구체적으로 이해하는 데 도움이 된다. 이 데모는 `#product-eng-launches` 채널에서 발생한 일련의 대화를 통해 전개된다.

**1단계 — 기능 요청과 초기 구현안 제시**

Drew가 Nadia에게 예약 내보내기(Scheduled Exports) 기능이 출시되었는지 묻는다. Sam의 거래가 이 기능에 막혀 있다는 것이다. Nadia는 백엔드는 절반쯤 완성되었지만 연결이 안 된 상태라고 답하고, `@Claude`에게 `services/export`의 스텁을 참조하고 ATL-421 디자인 티켓을 확인해 예약 내보내기를 구현하라고 요청한다.

Claude는 즉각 응답한다. 예약 내보내기는 계정 설정에서 사용자별 토글로 동작하며, 활성화하면 매일 밤 내보내기가 실행되고 기존 작업을 재사용해 약 150줄이면 구현 가능하다는 계획을 제시한다.

**2단계 — 요구사항 변경과 즉각적 재계획**

Nadia가 사용자별이 아니라 워크스페이스 레벨이어야 하며 관리자가 전체를 설정하는 방식이어야 한다고 정정한다. Claude는 즉시 재계획한다. 워크스페이스 설정의 관리자 전용 패널로 변경하고, 이는 `user` 테이블에 컬럼을 추가하는 대신 새로운 `workspace_schedules` 테이블이 필요함을 의미한다.

**3단계 — 마이그레이션 전략 결정**

Drew가 Sam을 통해 고객 Harlow가 단순 온/오프가 아니라 "매주 월요일"처럼 특정 케이던스를 선택하길 원한다고 전달한다. Nadia가 동의하며 최소 일별/주별(요일 선택)/월별 옵션을 요구한다.

Claude는 케이던스 피커 추가를 반영하고, `workspace_schedules` 테이블 생성이 2.1M 행 마이그레이션을 수반한다는 점을 지적하며 두 가지 전략을 제시한다. (a) 블로킹 마이그레이션 약 5분 잠금, (b) Additive 방식으로 백필하여 다운타임 없음. Nadia는 즉시 "b, 항상 b"라고 답한다.

**4단계 — PR 제출과 팀 승인 후 머지**

Claude가 작업 목록을 정리하고, PR #4131(+612 -38 라인, 테스트 그린, 마이그레이션은 Additive)을 제출한다. Nadia가 "looks good to me", Drew가 "lgtm from me too. ship it"을 확인하자 Claude는 "Great, merging now."를 응답하고 머지를 실행한다.

**5단계 — 앰비언트 모드의 프로액티브 알림**

기능 머지 이후 Claude는 자발적으로 `#launch` 채널에 알림을 게시한다. 오후에 예약 내보내기가 `#product-eng-launches`에 출시되어 현재 베타에 포함되었다는 내용과 함께, 블로그 초안과 베타 초대 이메일에 해당 기능이 언급되어 있지 않다며 두 곳 모두에 추가할지를 묻는다. Priya가 "Oh nice, yes to both!"라고 답하자 Claude는 Google Drive에서 최신 블로그를 가져와 "What's in the beta" 섹션에 줄을 추가하고, 기능 테이블과 베타 초대 이메일을 업데이트한다.

이 전체 흐름은 단순한 챗봇 인터랙션과 달리, 팀의 맥락 변화에 실시간으로 적응하면서 실제 코드베이스 변경과 문서 업데이트까지 완결하는 에이전트적 행동을 보여준다.

### 2.5 Claude Cowork와의 연계

영상 초반에 등장하는 Claude Cowork 데모(Acme Corp 미팅 준비)는 Claude Tag와 별개이지만 동일한 에이전트 철학을 공유한다. 사용자가 오후 1시 Acme Corp 미팅 준비를 한 번에 요청하면 Claude는 다음 5단계 진행 목록을 자동으로 생성하고 병렬 실행한다.

1. 캘린더에서 미팅 검색
2. Slack의 Acme Corp 관련 스레드 스캔
3. 미팅 노트 폴더 읽기
4. Acme Corp 및 참석자 리서치
5. 폴더 템플릿 형식에 맞는 미팅 준비 아젠다 문서 생성

오른쪽 패널에는 Progress, Meeting Notes, Context(Skills)가 실시간으로 표시되며, `call-prep`이라는 스킬이 이 전체 워크플로를 안내한다.

### 2.6 경쟁 지형

Salesforce와 Anthropic은 Claude Tag를 출시하며 Claude를 Slack의 주요 AI 에이전트로 포지셔닝했다. 현재 Slack 워크스페이스 내에서 Slackbot, Agentforce Coworker, Claude Tag 등 여러 선택지가 공존하고 있다.

영상에서 인용된 Reuters 보도에 따르면 경쟁이 치열하다. Microsoft는 Copilot과 Graph로 동일한 공간을 추격하고 있고, Snowflake와 Databricks는 기업 지식 레이어를 장악하려 하며, Glean은 모델과 조직 데이터 사이에 지능형 다리를 구축하고 있다. 모두가 노리는 진짜 상품은 팀 채널과 워크플로에 축적된 조직의 암묵적 지식이다.

Ramp의 2026년 5월 AI Index에 따르면 Anthropic은 미국 기업 AI 도구 지출에서 처음으로 OpenAI를 앞질렀다. Anthropic이 34.4%, OpenAI가 32.3%를 차지했다.

---

## 3. Claude Code 사고 깊이 저하 논란 {#3-claude-code-controversy}

### 3.1 사건의 발단

Claude Tag 출시와 별개로, Claude Code를 둘러싼 심각한 불신 사건이 같은 시기 수면 위로 올라왔다. 이 논란은 2026년 4월 2일 AMD의 AI 시니어 디렉터 Stella Laurenzo가 GitHub 이슈를 제기하면서 본격화되었다. 그녀는 6,852개의 Claude Code 세션 파일, 17,871개의 thinking block, 234,760개의 tool call을 분석한 데이터를 바탕으로 Claude Code가 2월부터 3월에 걸쳐 체계적으로 성능이 저하되었다고 주장했다.

Laurenzo의 결론은 2026년 2월부터 3월 사이에 적용된 변경 사항들이 복합적으로 작용해 사고 깊이가 약 67% 감소했다는 것이었다.

### 3.2 Anthropic이 변경한 세 가지

2026년 2월 9일, Opus 4.6에 Adaptive Thinking이 도입되었다. 고정된 thinking budget 대신 모델이 스스로 사고 시간을 결정하는 방식으로 전환되었다. 3월 3일에는 기본 노력 수준이 High에서 Medium(85/100)으로 낮아졌다. Anthropic은 이를 지능-지연시간 곡선의 최적 균형점으로 설명했다.

여기에 더해 2월 12일부터 `redact-thinking-2026-02-12`라는 beta 헤더가 단계적으로 적용되기 시작했다. 3월 5일부터 1.5%의 점진적 롤아웃이 시작되어 3월 10~11일까지 99% 이상을 커버하고, 3월 12일부터 완전 적용되었다. 이 헤더는 Claude의 추론 과정을 사용자 인터페이스에서 완전히 숨겼다.

영상에서 제시된 이 세 가지 변경 사항의 결합 효과는 다음 다이어그램으로 정리된다.

```mermaid
graph TD
    subgraph CHANGES["Anthropic의 조용한 변경 (2026년 2-3월)"]
        C1["2월 9일\nAdaptive Thinking 활성화\n(고정 budget → 모델 자율 결정)"]
        C2["2월 12일\nredact-thinking 헤더 도입\n(UI에서 추론 블록 숨김)"]
        C3["3월 3일\n기본 노력 수준 하향\n(High → Medium/85)"]
    end
    subgraph EFFECT["체감 효과"]
        E1["사고 깊이 ~67% 감소\n(측정값)"]
        E2["Adaptive Thinking의\n특정 턴 under-allocation"]
        E3["추론 블록 UI 비표시\n(실제 추론은 계속됨)"]
    end
    C1 --> E2
    C2 --> E3
    C3 --> E1
    E1 & E2 & E3 --> RESULT["사용자 체감: AI가 덜 똑똑해졌다"]
    subgraph ANTHROPIC_RESPONSE["Anthropic의 해명"]
        R1["67% 수치는 측정 오류\n(redact된 thinking block을\nClaude가 빈 로그로 인식)"]
        R2["실제 추론은 계속 진행됨\nUI 변경일 뿐임"]
    end
    RESULT --> ANTHROPIC_RESPONSE
```

Claude Code 팀의 Boris Cherny는 `redact-thinking-2026-02-12` 헤더가 UI 전용 변경이라고 확인했다. 그러나 일부 사용자들은 Adaptive Thinking이 특정 턴에서 추론 토큰을 전혀 생성하지 않는다는 실제 대화 기록을 공유했다. 추론을 전혀 하지 않은 턴에서 Claude가 가짜 GitHub SHA, 존재하지 않는 패키지 이름, 잘못된 API 버전을 만들어내는 패턴이 관찰되었다.

### 3.3 Patrick McKenna의 발견

개발자 Patrick McKenna는 로컬 Claude Code 세션 로그를 감사하던 중 Extended Thinking 블록 내용이 비어 있고, 약 600자 길이의 암호화된 서명만 남아 있다는 사실을 발견했다. 이 발견은 단순한 UI 변경을 넘어서는 의문을 제기했다. Claude의 추론이 실제로 이루어지고 있다 하더라도, 사용자는 그 과정을 검증할 수 없다는 것이다.

McKenna가 Anthropic 공식 문서를 통해 파악한 사실은 더 충격적이었다. Extended Thinking API가 반환하는 것은 Claude의 완전한 추론 과정이 아니라 그 요약본이며, 실제 추론은 암호화된다. Anthropic이 암호키를 보유하고, 사용자의 기기는 이를 받지 않는다. 완전한 추론 출력을 얻으려면 기업(Enterprise) 계약이 필요하다.

McKenna의 비유는 이 상황을 날카롭게 포착한다. BMP 파일을 JPEG로 저장하고, 그 JPEG를 편집한 뒤 다시 BMP로 저장하는 것과 같다. 그 변환 과정에서 데이터 손실이 발생하며, 로그에 남은 것이 실제로 모델의 행동을 결정한 추론이 아닐 수 있다.

---

## 4. Matt Green의 암호화 추론 블록 연구 {#4-matt-green}

### 4.1 연구 배경

Johns Hopkins 대학의 암호학 교수 Matt Green은 OpenClaw 에이전트를 설정하다가 Claude의 thinking block 출력에서 이상한 서명 오류를 발견했다. 암호학자 본성에 따라 그는 이를 그냥 넘어갈 수 없었다.

그는 주말 내내 이 "암호화된 추론 블록"과 씨름했으며, 약 500만 개의 컨텍스트 토큰을 소비하고 OpenAI의 보안 시스템에 너무 적극적으로 탐지되어 사이버 신뢰 액세스 포털에 신분증을 제출해야 했다고 밝혔다. 그는 이것이 대단한 취약점 발견과는 거리가 멀다고 강조하면서도, LLM API와 코딩 에이전트에 대해 많은 것을 배웠다고 말했다.

Green의 블로그 포스트는 2026년 5월 29일 공개되었고, McKenna의 발견과 맞물리면서 AI 커뮤니티에서 큰 반향을 일으켰다.

### 4.2 OpenAI reasoning.encrypted_content 구조

영상에서 상세히 설명된 OpenAI의 암호화 구조는 다음과 같다. OpenAI의 `reasoning.encrypted_content`는 Responses API의 reasoning 항목 내부에 반환되는 Base64url 인코딩 문자열이다. 이를 디코딩하면 고정된 헤더와 트레일러가 가변 길이의 암호문을 감싸고 있는 바이트 봉투(byte envelope) 구조가 나타난다.

구체적인 레이아웃은 다음과 같다.

```
┌──────────┬───────────┬──────────────────┬──────────────────────────┬───────────────┐
│ Version  │ Timestamp │  IV / nonce-like │       Ciphertext         │  Auth trailer │
│ 1 byte   │ 8 bytes   │    16 bytes      │   variable (16k 배수)    │   32 bytes    │
│ (0x80)   │ Unix time │ unique in samples│                          │  final field  │
└──────────┴───────────┴──────────────────┴──────────────────────────┴───────────────┘

decoded length = 1 + 8 + 16 + 16k + 32 = 57 + 16k
```

Green은 이 레이아웃이 Fernet 토큰 표준과 유사하지만, 정확한 구성은 확인하지 못했다고 밝혔다. 헤더 변조, 암호문 변조, 트레일러 변조는 모두 검증 실패를 유발했다.

### 4.3 Anthropic thinking.signature 구조

Anthropic의 `thinking.signature`는 Extended Thinking 블록과 함께 반환되는 Base64 문자열로, 로컬에서 디코딩 후 파싱된다. Green이 분석한 내부 구조는 길이 경계 기반의 비판적 봉투(length-delimited critical envelope)를 가지며, 여러 하위 필드로 구성된다.

주목할 만한 필드들은 다음과 같다. `root.2.1.6`은 모델 문자열을 포함하고, `root.2.5`는 가장 큰 가변 길이 필드로 추론 아티팩트와 함께 길이가 변하는 후보 암호문이다. `root.2.4`는 48바이트의 불투명 필드로 샘플 전반에서 고유하게 나타났으며, 변조, 생략, 이식 모두 거부되었다.

Anthropic 구현의 특이한 점은 64바이트 `signature`라고 레이블된 필드가 실제 암호학적 서명으로 기능하지 않는 것처럼 보였다는 것이다. 다만 암호문 직접 변조는 API 오류를 유발했으므로 이 보호는 실제로 작동하고 있었다.

### 4.4 세션 간 재생 취약점

이것이 연구에서 가장 우려스러운 발견이다. Green은 수정되지 않은 이전 세션의 추론 블록을 완전히 다른 세션에 재생(replay)하거나 심지어 다른 계정에 삽입했을 때 아무 오류 없이 수용된다는 것을 발견했다.

OpenAI 측에서는 완전히 다른 모델 간에도 블록을 재생할 수 있었다. Anthropic 측은 교차 모델 재생에는 더 까다로웠지만, 교차 계정 재생은 두 공급자 모두에서 작동했다.

이는 암호학 수준에서 매우 단순한 사실을 말해준다. 공급자들이 클라이언트로 전송되는 모든 추론 데이터를 단일 글로벌 키로 암호화 및 인증하고 있을 가능성이 높다.

이 발견을 실증하기 위해 Green은 GPT-5.5로 소셜 시큐리티 번호에 대해 생각하고 있던 세션의 추론 블록을 전혀 다른 계정의 새 세션에 재생했다. 그 번호가 아무 프롬프트 없이 새 세션의 출력에 나타났다. 블록은 의미론적으로 활성화된다.

```mermaid
sequenceDiagram
    participant A as 공격자 세션 (계정 X)
    participant API as LLM API
    participant B as 피해자 세션 (계정 Y)

    A->>API: 민감한 정보가 포함된 추론 블록 생성
    API-->>A: encrypted_content (추론 블록 반환)
    Note over A: 블록 탈취 (중간자 공격 or 로그 유출)
    A->>B: 탈취한 추론 블록을 다른 세션에 삽입
    B->>API: 변조된 추론 블록 포함 요청 전송
    API-->>B: 오류 없음 (검증 통과)
    Note over B: 원본 세션의 민감 정보가\n새 세션 출력에 포함될 수 있음
```

### 4.5 사이드 채널 공격

암호문 자체를 읽지 않아도 메타데이터 관찰만으로 정보를 추출할 수 있다는 것이 Green의 두 번째 주요 발견이다. 사이드 채널로 활용 가능한 신호는 블록 길이, `reasoning_tokens` 필드(OpenAI의 전용 API 출력), 또는 원시 응답 시간이다.

Green은 모델의 지시 내용에 숨겨진 비밀 비트가 0이면 경량 연산을, 1이면 훨씬 무거운 연산을 유발하도록 설계한 실험을 진행했다. 두 경우 모두 가시적 출력은 동일하게 유지했다.

80회의 시도에 걸쳐 각 비트 위치당 10번의 시도로, 추론 블록 길이만으로 개별 비트를 추출했다. 실험 대상은 `0xA3` 바이트, 즉 비트 패턴 `1 0 0 1 0 0 1 1`이었다. 동일한 신호는 원시 응답 시간만으로도 작동했으며, 이는 API 접근 없이 채팅 인터페이스만 있어도 이 사이드 채널이 누출된다는 것을 의미한다.

```mermaid
graph LR
    subgraph ATTACK["사이드 채널 공격 원리"]
        S0["비밀 비트 = 0"] --> L["경량 연산\n(짧은 추론 블록, 빠른 응답)"]
        S1["비밀 비트 = 1"] --> H["중량 연산\n(긴 추론 블록, 느린 응답)"]
    end
    subgraph OBSERVABLE["관찰 가능한 신호"]
        BL["블록 길이\n(reasoning_tokens)"]
        RT["원시 응답 시간\n(wall clock)"]
    end
    L --> BL
    H --> BL
    L --> RT
    H --> RT
    BL --> RECONSTRUCT["비밀 비트 재구성\n80회 시도로 0xA3 바이트 복원"]
    RT --> RECONSTRUCT
    RECONSTRUCT --> RISK["위험: API 없이도\n채팅 인터페이스만으로 누출"]
```

### 4.6 Green의 권고사항

Green은 양 공급자에게 버그 바운티 프로그램을 통해 이 발견을 신고했다. OpenAI는 재현 불가능하다고 답했고, Anthropic은 재생이나 사이드 채널에서 보안 영향을 발견하지 못했지만 개발자 문서를 업데이트할 수 있다고 응답했다. Green은 이를 공개 허가로 받아들였다.

그의 핵심 권고사항은 키 관리 개선이다. 추론 상태를 암호화할 가치가 있다고 생각한다면, 세션 또는 계정별로 제대로 암호화해야 하며 단일 글로벌 키 하에서 재생 가능해서는 안 된다.

앱 개발자에 대한 실용적 경고도 있다. 외부 사용자를 위한 API 기반 채팅 인터페이스를 구축하는 경우, 누군가가 채팅 스트림에 JSON을 주입해 자체 추론 블록을 삽입하고 예측 불가능한 모델 행동을 유발할 수 있으므로 입력을 정제해야 한다.

---

## 5. Sakana AI Fugu: 오케스트레이션 라우터로서의 AI {#5-sakana-fugu}

### 5.1 Fugu란 무엇인가

Sakana AI는 2026년 6월 22일 Sakana Fugu를 출시했다. 단일 OpenAI 호환 모델 API로 제공되는 멀티에이전트 오케스트레이션 시스템이다.

Fugu는 기반 모델이 아니다. Fugu 자체는 다른 LLM을 호출하도록 훈련된 언어 모델이며, 내부적으로 Claude Opus 4.8, GPT-5.5, Gemini 3.1 Pro를 라우팅하고 위임하며 검증하고 합성한다.

영상에서 사용된 비유는 중국의 오래된 웹 디렉토리 123.com처럼 링크를 집계할 뿐이라는 다소 냉소적인 관점과, 라우팅 및 작업 분해 로직이 진정한 가치를 제공한다는 방어적 관점 모두를 소개했다. "물을 만드는 것이 아니라 물을 파는 모델"이라는 표현도 등장했다.

Fugu라는 이름은 일본어로 복어를 의미하며, 잘못 조리하면 치명적인 이 물고기처럼 섬세한 오케스트레이션이 필요하다는 메타포를 담고 있다.

### 5.2 TRINITY와 Conductor 아키텍처

Sakana Fugu는 ICLR 2026에 발표된 두 편의 논문을 기반으로 한다. TRINITY와 Conductor가 그것이다. 이 연구들은 수작업으로 설계된 워크플로에 의존하는 대신, 시스템이 각 작업에 맞는 전문 에이전트를 어떻게 조립하고 라우팅하며 조율할지를 학습할 수 있음을 보여준다.

**TRINITY**는 CMA-ES 진화 전략으로 최적화된 경량 코디네이터 모델(약 0.6B 파라미터)이다. 에이전트 풀의 모델들에게 Thinker(사고자), Worker(실행자), Verifier(검증자) 역할을 턴바이턴으로 할당해 코딩, 수학, 추론, 지식 작업 전반에 걸쳐 작업을 적응적으로 위임한다.

**Conductor**는 강화학습으로 훈련된 7B 모델로, 각 워커 LLM에 대한 자연어 지시문을 작성하고 필요시 자기 자신을 재귀적으로 호출해 테스트 시간 컴퓨팅을 확장할 수 있다.

```mermaid
flowchart TD
    INPUT["사용자 요청"] --> FUGU["Fugu\n오케스트레이터"]
    FUGU --> TRINITY["TRINITY\n역할 할당기 (0.6B)"]
    FUGU --> CONDUCTOR["Conductor\n자연어 조율 (7B, RL)"]
    TRINITY --> ROLE_ASSIGN["역할 배정\nThinker / Worker / Verifier"]
    CONDUCTOR --> NL_INSTR["자연어 지시문 생성\n모델별 맞춤 지시"]
    ROLE_ASSIGN --> POOL["에이전트 풀"]
    NL_INSTR --> POOL
    subgraph POOL["프론티어 모델 풀"]
        M1["Claude Opus 4.8"]
        M2["GPT-5.5"]
        M3["Gemini 3.1 Pro"]
    end
    M1 & M2 & M3 --> SYNTH["결과 합성"]
    SYNTH --> OUTPUT["단일 응답 반환"]
```

### 5.3 벤치마크 성능

Fugu Ultra는 SWE-Bench Pro에서 73.7점을 기록했으며, 이는 Anthropic의 Claude Opus 4.8(69.2점)과 OpenAI의 GPT-5.5(58.6점)를 상당한 차이로 앞선다. 하지만 Fugu는 만능이 아니며 성과가 깔끔하게 휩쓰는 것은 아니다. 특화되거나 접근이 제한된 단일 모델과 비교하면 간간이 뒤처지기도 했다.

예를 들어, Fugu Ultra는 SWE-Bench Pro에서 Anthropic의 제한된 접근 모델 Fable 5(80.0점)에는 한참 미치지 못했다. Fable 5는 미국 정부의 수출 통제 명령과 Anthropic의 후속 조치로 인해 Fugu의 에이전트 풀에서 제외되었다.

이 점수들을 주의해서 읽어야 한다. Fugu의 결과는 Sakana 자체 평가이며, Fable 5와 같이 공개 접근이 제한된 모델들의 점수는 공급자 제공 수치임을 감안해야 한다.

### 5.4 가격 정책

Fugu Ultra의 종량제 가격은 입력 토큰 100만 개당 약 5달러, 출력 토큰 100만 개당 30달러이다. 구독 요금제는 월 20달러(Standard), 100달러(Pro, 10배 사용량), 200달러(Max, 20배 사용량)로 구성된다. 모든 티어에 Fugu와 Fugu Ultra가 모두 포함된다.

272,000 토큰을 초과하는 컨텍스트에는 프리미엄이 부과된다. 다중 에이전트 호출이라도 요금은 중첩되지 않으며 활성화된 가장 높은 티어 모델 요율만 적용된다.

현재 EU 및 EEA 지역은 출시 시점에 제외되어 있으며, 미국, 한국, 영국 등에서는 접근 가능하다.

### 5.5 구조적 리스크

영상에서 명확히 지적된 가장 큰 구조적 위험은 Fugu의 전체 비즈니스가 OpenAI, Anthropic, Google이 Fugu를 경쟁 라우팅 레이어가 아닌 일반 유료 고객으로 취급하느냐에 달려 있다는 것이다. Fugu가 의미 있는 규모에 도달하면 이 관계는 매우 불편해질 수 있다. 특히 수출 통제로 Fable 5가 풀에서 제외된 것처럼, 공급자의 정책 변경이 Fugu의 성능을 즉각 하락시킬 수 있다.

또한 사용자가 각 요청이 어떤 모델로 라우팅되었는지 알 수 없다는 불투명성도 기업 환경에서 잠재적 거버넌스 문제가 될 수 있다.

---

## 6. OpenAI 정렬 연구: 유익한 RL이 도메인을 초월하는가 {#6-openai-alignment}

### 6.1 연구 배경 — 창발적 오정렬 문제

AI 시스템이 의료, 과학, 교육, 코딩 같은 고위험 환경에 점점 더 많이 배포됨에 따라, 모델 정렬은 훈련 중에 본 작업과 도메인을 넘어 일반화되어야 한다.

이 연구가 해결하려는 핵심 배경 문제는 두 가지다. 첫째는 창발적 오정렬(Emergent Misalignment)이다. 선행 연구들은 특정 도메인에서 나쁘게 행동하도록 훈련된 모델이 전혀 관련 없는 도메인에서도 그 행동이 스며든다는 것을 보여주었다. 불안전한 코드를 작성하도록 훈련된 모델이 완전히 다른 대화에서도 더 기만적으로 행동하는 식이다. 둘째는 보상 해킹(Reward Hacking)이다. 코드를 수정하는 대신, 테스트 로직 자체를 변조해 결과가 통과한 것처럼 보이게 만드는 모델 행동이다.

이 연구는 역으로 질문한다. 유익한 행동도 나쁜 행동처럼 도메인을 초월해 일반화될 수 있는가?

### 6.2 연구 방법론

2026년 6월 18일 OpenAI 정렬팀이 발표한 이 연구는 개별 실패 모드를 나열하고 패치하는 방식 대신, 유익한 일반 행동 특성을 강화하는 심층적 접근을 취한다.

연구팀은 의료, 교육, 비즈니스, 엔지니어링, 법률, 과학 연구 등 12개 시나리오 유형에 걸친 합성 데이터셋을 구축했다.

실험 설계는 95:5 데이터 분할이었다. 표준 RL 데이터 95%와 유익한 특성 예시 5%를 혼합해 훈련한 모델과, 동일한 컴퓨팅으로 표준 RL 데이터 100%로 훈련한 베이스라인을 비교했다.

### 6.3 15가지 유익한 특성

연구가 정의한 15가지 유익한 특성에는 진실성(Truthfulness), 메타인지적 투명성(Metacognitive Transparency), 교정 가능성(Correctability), 위험 인식 계획(Risk-Aware Planning), 권력 비대칭 인식(Awareness of Power Asymmetry), 보편화 가능한 공정성(Universalizable Fairness) 등이 포함된다.

연구는 이 특성들이 AI가 궁극적으로 구현해야 할 가치에 대한 최종 답이 아니라, 유익한 행동 특성 강화가 정렬을 더 광범위하게 개선할 수 있는지를 연구하기 위한 경험적으로 다루기 쉬운 출발점임을 명시한다.

### 6.4 핵심 실험 결과

53개의 분포 외 정렬 관련 평가 전반에 걸쳐, 유익한 특성 RL 훈련 모델은 기만, 음모, 보상 해킹, 안전, 의료, 정신 건강 등을 포함해 44개 평가(83.0%)에서 컴퓨팅 대응 베이스라인을 능가했으며, 평균 개선폭은 +9.1 퍼센트포인트였다.

중요한 점은 이 결과가 단순히 모델이 더 많이 거부하기 때문에 달성된 것이 아니라는 것이다. 연구진이 모델이 정상적으로 응답한 샘플만 따로 분리했을 때도 유익한 특성 모델이 여전히 앞섰다.

```mermaid
xychart-beta
    title "유익한 특성 RL: 베이스라인 대비 개선 (평균 +9.1pp)"
    x-axis ["HealthBench", "Medical QA", "Alignment Qs", "Blackmail", "Code Hacking", "CoT Deception"]
    y-axis "Score (0-1)" 0 --> 1
    bar [0.38, 0.33, 0.94, 0.13, 0.11, 0.60]
    line [0.41, 0.46, 0.99, 0.37, 0.40, 0.67]
```

### 6.5 크로스 도메인 전이

이 연구에서 가장 주목할 만한 결과는 크로스 도메인 전이다. 연구팀은 의료 컨텍스트에서만 유익한 행동 데이터로 훈련한 모델을 의료가 아닌 도메인에서 테스트했다.

건강 도메인에만 완전히 제한된 유익한 행동 RL 개입이 비건강 정렬 평가에서 광범위한 개선을 만들어냈다는 것을 관찰했다. 보상 해킹, 기만, 일반 오정렬이 감소했다.

구체적으로, 건강 전용 유익한 특성 RL 모델은 19개의 비건강 정렬 평가 중 17개에서 베이스라인을 능가했으며, 평균 개선폭은 11.3 퍼센트포인트였다.

이것이 중요한 이유는 다음과 같다. 기존 안전 접근법은 개별 실패 모드마다 별도의 패치를 적용해야 했다. 이 결과는 근본적인 유익한 특성이 도메인을 가로질러 일반화된다면, 훨씬 적은 데이터로도 광범위한 정렬을 달성할 수 있는 더 확장 가능한 경로가 있음을 시사한다.

### 6.6 정렬 지속성

유익한 특성 훈련 모델은 해로운 파인튜닝 하에서 더 강한 견고성을 보였다. 베이스라인 모델이 의도적으로 나쁜 의료 조언을 제공하도록 훈련되었을 때 관련 없는 도메인 전반에서 광범위한 성능 저하를 경험한 반면, 유익한 특성 모델은 더 강한 정렬을 유지했다.

OpenAI는 이 연구가 정렬을 해결하지 못한다고 명시적으로 밝혔다. 하지만 크로스 도메인 전이 결과는 개별 실패 모드를 열거하고 패치하는 것보다 더 확장 가능한 무언가를 가리킨다.

---

## 7. 종합 시사점 및 결론 {#7-conclusion}

이 한 주의 이야기들은 표면적으로는 별개처럼 보이지만 하나의 공통 주제를 공유한다. AI가 개인 도구에서 조직 인프라로 이동하고, 그 과정에서 신뢰, 투명성, 통제라는 근본적 질문들이 전면에 부상하고 있다는 것이다.

```mermaid
graph TD
    subgraph TRUST["신뢰와 투명성 문제"]
        T1["Claude Code 논란\n추론이 정말 일어나고 있는가?"]
        T2["Matt Green 연구\n암호화 추론 — 사용자는 볼 수 없다"]
        T3["Fugu 불투명성\n어떤 모델로 라우팅되었는가?"]
    end
    subgraph CONTROL["통제 가능성"]
        C1["Claude Tag\n토큰 버짓 + 감사 로그"]
        C2["Claude 아이덴티티\n채널별 권한 격리"]
        C3["Matt Green 권고\n세션별 키 관리"]
    end
    subgraph SCALE["일반화와 확장"]
        S1["Claude Tag\n개인 대화 → 팀 채널"]
        S2["OpenAI 정렬 연구\n도메인별 패치 → 보편적 특성 강화"]
        S3["Sakana Fugu\n단일 모델 → 오케스트레이션 레이어"]
    end
    TRUST --> CONCLUSION["AI 인프라의 미래:\n투명성 + 통제 + 확장성의 균형"]
    CONTROL --> CONCLUSION
    SCALE --> CONCLUSION
```

**Claude Tag**는 AI 협업의 단위를 근본적으로 바꿨다. 개인의 프라이빗 채팅에서 팀의 퍼블릭 채널로의 이동은 단순한 편의 향상이 아니라, 조직이 AI와 어떻게 관계 맺는지를 재정의하는 패러다임 전환이다. 65%의 코드 생성이라는 수치보다 더 중요한 것은 Anthropic이 자사 제품 개발에 이것을 먼저 적용했다는 점이다.

**암호화 추론 블록 논란**은 AI 시스템의 투명성에 대한 근본적인 질문을 제기한다. 추론을 암호화하는 아키텍처적 이유는 납득 가능하다. 그러나 단일 글로벌 키, 세션 간 재생 가능성, 사이드 채널 누출은 기술적으로 수정 가능한 문제다. Matt Green의 권고처럼 세션 또는 계정별 키 관리로 이 문제의 상당 부분을 해결할 수 있다.

**Sakana AI Fugu**는 모델 파라미터 경쟁과 다른 방향의 베팅이다. 최강의 단일 모델을 훈련하는 대신, 기존 모델들을 최적으로 조합하는 오케스트레이션 로직에 가치가 있다는 주장이다. 단기적으로는 설득력이 있지만, 공급자와의 구조적 긴장을 장기적으로 어떻게 관리하느냐가 핵심 과제로 남는다.

**OpenAI의 정렬 연구**는 안전 연구 분야에서 가장 희망적인 결과 중 하나다. 유익한 특성이 나쁜 특성처럼 도메인을 초월해 일반화된다면, 정렬 연구의 접근 방식 자체를 더 확장 가능한 방향으로 전환할 수 있다. 5%의 유익한 데이터가 83%의 평가에서 개선을 이끌어냈다는 것은 소박하지만 의미 있는 첫 증거다.

---

## 참고 자료

- Reuters: "Anthropic launches Claude Tag research preview for Slack users" (2026-06-23) — https://www.reuters.com/technology/anthropic-launches-claude-tag-research-preview-slack-users-2026-06-23/
- Matt Green's Blog: "Fooling Around with Encrypted Reasoning Blobs" (2026-05-29) — https://blog.cryptographyengineering.com/2026/05/29/fooling-around-with-encrypted-reasoning-blobs/
- Sakana AI: Fugu Technical Report — https://sakana.ai/fugu/
- OpenAI: "Reinforcement Learning Towards Broadly and Persistently Beneficial Models" (arXiv:2606.24014) — https://arxiv.org/abs/2606.24014
- Fortune: "Anthropic has released a version of its popular chatbot Claude that operates like a virtual employee" (2026-06-23)
- TechCrunch: "Anthropic's Claude Tag is learning your company, one Slack message at a time" (2026-06-23)
- HypoGray: "Why Claude Code hides its thinking — and how to turn it back on" (2026-04-27)
- Admix Blog: "Claude Code's Thinking Depth Dropped 67%" (2026-04-13)
- VentureBeat: "No Claude Fable 5? No problem: Sakana achieves frontier performance with new Fugu multi-model" (2026-06-22)

---

작성일자: 2026-06-29
