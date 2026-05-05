---
title: "LangGraph/LangChain 기반 Hub and Spoke 아키텍처 정의서"
date: 2026-05-05 21:30:00 +0900
categories: [AI,  Platform & Framework]
mermaid: [True]
tags: [AI,  ai-system-architecture,  multi-agent-orchestration,  hub-and-spoke,  LangChain,  LangGraph,  LangSmith,  deep-agents,  Claude.write]
---


>- **버전:** v2.0 (2026년 5월 기준 — 최신 Changelog 완전 반영)
>- **LangGraph:** v1.1 (2026.03.10)
>- **LangChain:** v1.2.0 (2025.12.15)
>- **Deep Agents:** v0.5.0 (2026.04.07)
>- **목적:** 멀티에이전트 허브-앤-스포크 아키텍처 설계 및 구현 가이드
>- **대상:** AI 시스템 아키텍트, 백엔드 엔지니어, ML 엔지니어

## 관련글

[**Claude Architect: 멀티에이전트 오케스트레이션 완전 가이드**](https://k82022603.github.io/posts/claude-architect-%EB%A9%80%ED%8B%B0%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8-%EC%98%A4%EC%BC%80%EC%8A%A4%ED%8A%B8%EB%A0%88%EC%9D%B4%EC%85%98-%EC%99%84%EC%A0%84-%EA%B0%80%EC%9D%B4%EB%93%9C/)

---

**Q: Hub and Spoke 아키텍처 이용해서 무엇을 만들 수 있는거에요?**

**A: Hub and Spoke로 만들 수 있는 것들을 유즈케이스 유형별로 정리해드릴게요.**

핵심 기준은 하나입니다. **"혼자서는 벅차고, 여럿이 나눠도 서로 알 필요 없는 작업"** 이라면 Hub and Spoke가 딱 맞습니다.


**리서치·분석 시스템**이 가장 전형적인 사례입니다. "반도체 시장 전망 보고서 만들어줘"라는 요청 하나를 코디네이터가 받으면, 시장 데이터 수집 에이전트·경쟁사 분석 에이전트·정책 동향 에이전트·재무 모델링 에이전트·리포트 작성 에이전트로 태스크를 분해합니다. 각 에이전트는 서로의 존재를 모른 채 자기 영역만 파고들고, 코디네이터가 결과를 취합해 완성도 높은 보고서를 만들어냅니다. 기존에 애널리스트 팀이 며칠 걸리던 작업을 수 시간으로 줄일 수 있습니다.

**소프트웨어 개발 자동화 파이프라인**도 매우 유망합니다. 요구사항이 들어오면 코디네이터가 아키텍트 에이전트·백엔드 개발 에이전트·프론트엔드 에이전트·테스트 에이전트·보안 감사 에이전트를 순서에 맞게 호출합니다. [RummiArena](https://github.com/k82022603/RummiArena)의 멀티에이전트 구조도 사실 이 패턴의 변형이고, [JEEPO_3D](https://k82022603.github.io/posts/jeepo_3d-%EC%95%BC%EA%B0%84-%EC%9E%90%EB%8F%99%ED%99%94-%ED%8C%A8%EC%B9%98-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EB%B6%84%EC%84%9D/)의 Jarvis 파이프라인도 동일한 원리입니다.

**법률·컴플라이언스 검토 시스템**에서도 강력합니다. 계약서 하나를 코디네이터에게 넘기면, 조항 분류 에이전트가 섹션을 분리하고, 리스크 탐지 에이전트가 불리한 조항을 식별하고, 판례 검색 에이전트가 관련 사례를 찾고, 요약 에이전트가 경영진용 리포트를 만드는 식입니다. Harvey 같은 리걸테크 스타트업들이 정확히 이 방향으로 가고 있습니다.

**고객 지원 오케스트레이션**도 좋은 적용처입니다. 고객 문의가 들어오면 코디네이터가 의도를 파악하고, 상품 지식 에이전트·주문 조회 에이전트·환불 정책 에이전트·감성 분석 에이전트 중 필요한 것만 선택적으로 호출합니다. 단순 FAQ는 에이전트 하나로, 복잡한 클레임은 여러 에이전트를 순차 호출하는 동적 라우팅이 핵심입니다.

**AI 기반 홍수 분석 시스템** 같은 도메인도 있습니다. 기상 데이터 수집 에이전트·지형 분석 에이전트·역사적 패턴 비교 에이전트·위험 구역 예측 에이전트·알림 생성 에이전트가 코디네이터 아래에서 분업하는 구조입니다.

**개인화 콘텐츠 생성 파이프라인**, 즉 사용자 데이터를 받아 리서치·초안 작성·팩트체크·SEO 최적화·이미지 프롬프트 생성까지 각 에이전트가 담당하는 미디어 자동화 시스템도 자주 구축됩니다.


결국 공통점은 세 가지입니다. 태스크가 여러 전문 영역에 걸쳐 있고, 각 영역은 서로 독립적으로 처리 가능하며, 최종 결과를 취합해야 한다는 것입니다. 이 세 조건이 맞으면 Hub and Spoke가 가장 깔끔한 해답입니다.


---

## ⚡ v2.0 업데이트 요약 (공식 Changelog 기반)

이 정의서는 2026년 5월 5일 기준 LangChain 공식 Changelog([https://docs.langchain.com/oss/python/releases/changelog](https://docs.langchain.com/oss/python/releases/changelog))를 직접 반영하여 작성되었다. [이전 버전(v1.0)](https://claude.ai/public/artifacts/d646eeec-37b3-4754-af9d-846567951442)과 달라진 핵심 사항은 다음과 같다.

| 변경 항목 | 이전 (v1.0) | 현재 (v2.0) |
|---|---|---|
| LangGraph 호출 방식 | `invoke()` dict 반환 | `invoke(version="v2")` → `GraphOutput` 객체 |
| 스트리밍 타입 안전성 | 비정형 chunk | `StreamPart` TypedDict (type/ns/data) |
| 상태 타입 | TypedDict 수동 파싱 | Pydantic/dataclass 자동 강제 변환 |
| 서브에이전트 실행 | 순차 동기 | **비동기 비블로킹** (Deep Agents v0.5) |
| 컨텍스트 오버플로 | 수동 처리 | `ContextOverflowError` 자동 요약 트리거 |
| 툴 파라미터 | 범용적 | `extras` 속성으로 공급자별 특화 |
| 모델 역량 파악 | 수동 문서 참조 | `.profile` 속성 자동 노출 |
| 에러 재시도 | 수동 구현 | 모델 재시도 미들웨어 내장 |

---

## 목차

1. [문서 개요](#1-문서-개요)
2. [기술 스택 및 버전 정의](#2-기술-스택-및-버전-정의)
3. [아키텍처 개요](#3-아키텍처-개요)
4. [핵심 설계 원칙](#4-핵심-설계-원칙)
5. [LangGraph v1.1 핵심 변경사항과 Hub-Spoke 적용](#5-langgraph-v11-핵심-변경사항과-hub-spoke-적용)
6. [컴포넌트 상세 설계](#6-컴포넌트-상세-설계)
7. [상태(State) 설계 — Pydantic v2 기반](#7-상태state-설계--pydantic-v2-기반)
8. [라우팅(Routing) 설계](#8-라우팅routing-설계)
9. [비동기 서브에이전트 (Deep Agents v0.5)](#9-비동기-서브에이전트--deep-agents-v05)
10. [미들웨어 레이어 (LangChain v1.1/v1.2)](#10-미들웨어-레이어--langchain-v11v12)
11. [메모리 및 영속성 설계](#11-메모리-및-영속성-설계)
12. [Observability 설계 — v2 스트리밍 포함](#12-observability-설계--v2-스트리밍-포함)
13. [Human-in-the-Loop 설계](#13-human-in-the-loop-설계)
14. [디렉토리 구조 및 코드 예시](#14-디렉토리-구조-및-코드-예시)
15. [배포 아키텍처](#15-배포-아키텍처)
16. [안티패턴과 주의사항](#16-안티패턴과-주의사항)
17. [프레임워크 선택 근거](#17-프레임워크-선택-근거)
18. [파워포인트 구성 가이드](#18-파워포인트-구성-가이드)

---

## 1. 문서 개요

### 1.1 작성 배경

2025년 10월 LangGraph v1.0과 LangChain v1.0이 동시에 GA(General Availability)를 선언했다. 그 이후 불과 반년 만에 LangGraph v1.1(2026.03), Deep Agents v0.5(2026.04), LangChain v1.2(2025.12)가 연달아 릴리스되며 멀티에이전트 시스템 구축의 방식이 크게 변화했다.

가장 중요한 변화는 두 가지다. 첫째, **타입 안전성**의 전면 도입이다. LangGraph v1.1의 `version="v2"` 호출 방식은 상태 그래프의 입출력을 Pydantic 모델 또는 dataclass로 자동 강제 변환하여, 런타임에 발생하던 타입 불일치 오류를 컴파일 시점에 잡을 수 있게 되었다. 둘째, **비동기 서브에이전트**의 등장이다. Deep Agents v0.5의 비동기 서브에이전트 지원으로 허브가 특정 스포크의 결과를 기다리는 동안 사용자는 계속 인터랙션할 수 있게 되었고, 여러 스포크를 진정한 병렬로 실행할 수 있는 기반이 마련되었다.

이 정의서는 이러한 최신 변화를 완전히 반영한 Hub and Spoke 아키텍처 설계 가이드다.

### 1.2 문서 범위

이 문서는 LangGraph v1.1 기반 Supervisor(허브) 에이전트 설계, 전문화된 서브에이전트(스포크) 설계, Pydantic 기반 공유 상태 스키마, 동적 라우팅 로직, 비동기 서브에이전트 패턴, 미들웨어 레이어 설계, 메모리·영속성·관찰 가능성 설계, 그리고 실제 구현 코드 예시를 포함한다.

---

## 2. 기술 스택 및 버전 정의

### 2.1 공식 Changelog 기준 최신 버전표

| 패키지 | 최신 버전 | 릴리스 일자 | 주요 변경 |
|---|---|---|---|
| `langgraph` | v1.1 | 2026.03.10 | v2 타입 안전 호출/스트리밍, Pydantic 강제 변환 |
| `langchain` | v1.2.0 | 2025.12.15 | extras 속성, 공급자별 툴 파라미터 |
| `langchain` | v1.1.0 | 2025.11.25 | model profiles, 미들웨어 체계, 재시도 |
| `langchain` (GA) | v1.0.0 | 2025.10.20 | 정식 v1 선언 |
| `deepagents` | v0.5.0 | 2026.04.07 | 비동기 서브에이전트, 멀티모달 read_file |
| `deepagents` | v0.4 | 2026.02.10 | 샌드박스 통합, ContextOverflowError 자동 요약 |

### 2.2 패키지 설치

```bash
# 핵심 패키지
pip install "langgraph>=1.1" \
            "langchain>=1.2.0" \
            "langchain-core>=0.3" \
            langgraph-supervisor \
            deepagents

# LLM 공급자 통합
pip install langchain-anthropic    # Claude Sonnet 4.6
pip install langchain-openai       # GPT-4o

# 영속성 (프로덕션)
pip install langgraph-checkpoint-postgres

# 관찰 가능성
pip install langsmith

# 샌드박스 (Deep Agents v0.4+)
pip install langchain-modal        # 또는 langchain-daytona, langchain-runloop
```

---

## 3. 아키텍처 개요

### 3.1 전체 구조 (v2.0)

```mermaid
graph TD
    User["👤 사용자"] --> SupervisorNode["🎯 Supervisor Node\n허브 — 모든 흐름의 단일 제어점"]

    SupervisorNode --> Router{"add_conditional_edges()\n동적 라우터"}

    Router -->|"동기 태스크"| ResearchAgent["🔍 Research Agent\n스포크 1 (동기)"]
    Router -->|"동기 태스크"| CoderAgent["💻 Coder Agent\n스포크 2 (동기)"]
    Router -->|"비동기 병렬"| AsyncAnalyst["📊 Analyst Agent\n스포크 3 (비동기, v0.5+)"]
    Router -->|"비동기 병렬"| AsyncReviewer["🔎 Reviewer Agent\n스포크 4 (비동기, v0.5+)"]
    Router -->|"완료"| END["✅ END"]

    ResearchAgent --> SupervisorNode
    CoderAgent --> SupervisorNode
    AsyncAnalyst -.->|"비블로킹 결과"| SupervisorNode
    AsyncReviewer -.->|"비블로킹 결과"| SupervisorNode

    SupervisorNode --> SharedState["📦 Pydantic State\n(v1.1 자동 강제 변환)"]
    SharedState --> Checkpointer["💾 Checkpointer\nPostgreSQL"]
    SharedState --> MiddlewareLayer["🛡️ Middleware\n재시도·요약·모더레이션"]

    style SupervisorNode fill:#d4b8f0,stroke:#9c6dba,stroke-width:3px
    style SharedState fill:#b8d4f0,stroke:#4a86c8
    style MiddlewareLayer fill:#f0e6b8,stroke:#c8a84a
    style END fill:#b8f0c8,stroke:#4ab87a
    style AsyncAnalyst fill:#d4f0e0,stroke:#4ab87a,stroke-dasharray:5
    style AsyncReviewer fill:#d4f0e0,stroke:#4ab87a,stroke-dasharray:5
```

> 점선 화살표는 Deep Agents v0.5.0에서 도입된 **비동기 비블로킹** 서브에이전트를 나타낸다. 실선은 기존의 동기 서브에이전트다.

### 3.2 LangGraph의 세 가지 핵심 구성요소

LangGraph는 상태(State), 노드(Node), 엣지(Edge)라는 세 가지 원소로 에이전트 워크플로우를 표현한다. 이 세 개념이 Hub and Spoke의 각 요소와 1:1로 대응한다.

- **State**: 그래프의 현재 스냅샷을 나타내는 공유 데이터 구조. LangGraph v1.1부터 Pydantic 모델로 선언하면 입출력 시 자동 강제 변환된다.
- **Node**: 에이전트의 로직을 인코딩하는 함수. 현재 상태를 입력으로 받아 업데이트된 상태를 반환한다.
- **Edge**: 다음 실행할 노드를 결정하는 함수. 조건부 엣지(conditional edges)가 허브의 동적 라우팅을 구현한다.

---

## 4. 핵심 설계 원칙

### 원칙 1: 단방향 권한 흐름

모든 의사결정 권한은 Supervisor에게만 있다. 서브에이전트들은 서로의 존재를 모르며, 반드시 허브를 경유해야만 다음 에이전트로 흐름이 넘어간다. 이 원칙을 위반하는 순간 관찰 가능성과 에러 처리의 일관성이 무너진다.

### 원칙 2: Pydantic 기반 타입 안전 상태 (v1.1 신규)

LangGraph v1.1부터 `version="v2"` 호출 시 상태가 선언된 Pydantic 모델로 자동 강제 변환된다. 이는 단순한 편의기능이 아니라 런타임 타입 오류를 예방하는 아키텍처 안전망이다. 모든 상태는 `TypedDict` 대신 `BaseModel`(Pydantic v2)로 정의하는 것을 권장한다.

### 원칙 3: 비동기 스포크는 결과를 상태에 기록

Deep Agents v0.5의 비동기 서브에이전트는 백그라운드에서 실행되며 결과를 공유 상태에 기록한다. Supervisor는 필요할 때 상태에서 결과를 읽는다. 비동기 스포크가 Supervisor의 컨트롤 루프를 우회하는 방식으로 통신해서는 안 된다.

### 원칙 4: ContextOverflowError 자동 처리 (v0.4 신규)

Deep Agents v0.4부터 LLM이 `ContextOverflowError`를 발생시키면 대화 요약이 자동으로 트리거된다. 아키텍처 수준에서 이 메커니즘에 의존하여 긴 에이전트 루프에서도 컨텍스트 창이 넘치지 않도록 설계한다.

### 원칙 5: 미들웨어를 통한 횡단 관심사 분리

LangChain v1.1의 미들웨어 체계(재시도, 요약, 모더레이션)를 통해 에이전트 로직과 인프라 관심사를 분리한다. 각 서브에이전트는 도메인 로직에만 집중하고, 재시도나 콘텐츠 필터링은 미들웨어가 담당한다.

---

## 5. LangGraph v1.1 핵심 변경사항과 Hub-Spoke 적용

### 5.1 타입 안전 호출: `version="v2"` invoke

LangGraph v1.1의 가장 중요한 변화는 `version="v2"` 호출 방식이다. 기존의 `invoke()`는 dict를 반환했지만, `version="v2"`는 `GraphOutput` 객체를 반환하며 `.value`와 `.interrupts` 속성을 통해 결과와 인터럽트 상태에 명시적으로 접근한다.

```python
from langgraph.types import GraphOutput

# ❌ 이전 방식 (v1.0) - dict 반환, 타입 불안전
result = app.invoke(input_state, config=config)
final_answer = result.get("final_response")  # 오타 시 None 반환, 에러 없음

# ✅ 현재 방식 (v1.1) - GraphOutput 반환, 타입 안전
result: GraphOutput = app.invoke(
    input_state, 
    config=config,
    version="v2"  # ← 핵심 파라미터
)
final_answer = result.value.final_response  # Pydantic 속성 접근, 오타 시 AttributeError
human_reviews = result.interrupts            # 인터럽트 목록 명시적 접근
```

Hub and Spoke 아키텍처에서 이 변화의 의미는 크다. Supervisor가 서브에이전트로부터 결과를 취합할 때, 딕셔너리 키 이름의 오타나 누락으로 인한 버그가 컴파일 시점에 발견된다.

### 5.2 타입 안전 스트리밍: `StreamPart`

```python
from langgraph.types import StreamPart

# ✅ v2 스트리밍 - StreamPart TypedDict
async for chunk in app.astream(
    input_state,
    config=config,
    stream_mode="values",
    version="v2"
):
    chunk: StreamPart  # type, ns, data 키를 가진 TypedDict
    event_type = chunk["type"]  # "values", "updates", "messages" 등
    namespace = chunk["ns"]     # 서브그래프 네임스페이스 추적
    data = chunk["data"]        # 실제 데이터
```

Hub and Spoke에서 스트리밍은 특히 중요하다. 어떤 서브에이전트(스포크)에서 어떤 이벤트가 발생하고 있는지 `ns`(네임스페이스) 필드로 정확히 추적할 수 있기 때문이다.

### 5.3 Pydantic 상태 자동 강제 변환

```python
from pydantic import BaseModel, Field
from typing import Optional, Literal
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages
from typing import Annotated

class HubSpokeState(BaseModel):
    """v1.1 Pydantic 기반 상태 — version='v2' 호출 시 자동 강제 변환"""
    
    messages: Annotated[list[BaseMessage], add_messages] = Field(default_factory=list)
    current_task: str = Field(default="")
    research_results: list[str] = Field(default_factory=list)
    code_artifacts: list[str] = Field(default_factory=list)
    analysis_results: Optional[dict] = None
    review_feedback: Optional[dict] = None
    
    next_agent: Literal[
        "research_agent", "coder_agent", "analyst_agent", "reviewer_agent", "FINISH"
    ] = "FINISH"
    
    iteration_count: int = 0
    max_iterations: int = 10
    errors: list[dict] = Field(default_factory=list)
    final_response: Optional[str] = None

    class Config:
        arbitrary_types_allowed = True
```

### 5.4 Time Travel 수정 (v1.1)

LangGraph v1.1은 인터럽트와 서브그래프에서의 타임 트래블(time travel) 버그를 수정했다. 이전 버전에서는 인터럽트 후 재생(replay) 시 오래된 `RESUME` 값을 재사용하여 서브그래프의 체크포인트가 부모의 이전 상태로 잘못 복원되는 문제가 있었다. v1.1에서 이 문제가 해결되어 Human-in-the-Loop 패턴이 더 신뢰할 수 있게 되었다.

---

## 6. 컴포넌트 상세 설계

### 6.1 Supervisor Agent (허브)

Supervisor는 Hub and Spoke의 두뇌로, 오케스트레이션에만 집중한다.

```python
from langchain_anthropic import ChatAnthropic
from langchain.chat_models import init_chat_model
from langgraph_supervisor import create_supervisor

# model.profile로 역량 자동 확인 (LangChain v1.1+)
model = init_chat_model("anthropic:claude-sonnet-4-6")
print(model.profile)  # 모델의 지원 기능, 컨텍스트 창 크기 등 자동 노출

SUPERVISOR_PROMPT = """
당신은 전문화된 에이전트 팀의 수퍼바이저입니다.

핵심 원칙:
- 직접 작업하지 않습니다. 오직 위임·취합·라우팅만 합니다.
- 불필요한 에이전트는 절대 호출하지 않습니다.
- 모든 서브에이전트의 결과를 공유 상태에서 취합합니다.

라우팅 기준:
- 정보 수집 → research_agent
- 코드 작성/실행 → coder_agent
- 데이터 분석 → analyst_agent (비동기 가능)
- 품질 검토 → reviewer_agent (비동기 가능)
- 모든 작업 완료 → FINISH
"""

workflow = create_supervisor(
    agents=[research_agent, coder_agent, analyst_agent, reviewer_agent],
    model=model,
    prompt=SUPERVISOR_PROMPT,
)

app = workflow.compile(checkpointer=checkpointer, store=store)
```

### 6.2 서브에이전트 — `extras`로 공급자별 툴 최적화 (v1.2 신규)

LangChain v1.2.0에서 도입된 `extras` 속성은 Anthropic의 프로그래매틱 툴 호출이나 툴 검색 같은 공급자별 특화 기능을 선언적으로 설정할 수 있게 한다.

```python
from langchain_core.tools import tool
from langchain.tools import BaseTool

# extras 속성으로 Anthropic 특화 설정 (LangChain v1.2+)
def web_search_tool(query: str) -> str:
    """웹에서 최신 정보를 검색합니다."""
    # 검색 로직
    ...

# Research Agent 구성
research_agent = create_react_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-6"),
    tools=[web_search_tool, document_loader_tool],
    name="research_agent",
    prompt="당신은 리서치 전문가입니다. 심층 조사를 수행하고 결과를 구조화하여 반환합니다.",
)
```

### 6.3 서브에이전트 책임 매트릭스

```mermaid
graph TB
    subgraph "스포크 1: Research Agent"
        R1["📋 책임: 정보 수집"]
        R2["🔧 툴: web_search, doc_loader, pdf_parser"]
        R3["⚡ 모드: 동기"]
    end

    subgraph "스포크 2: Coder Agent"
        C1["📋 책임: 코드 작성·실행"]
        C2["🔧 툴: code_exec, file_write, git_ops"]
        C3["⚡ 모드: 동기"]
    end

    subgraph "스포크 3: Analyst Agent"
        A1["📋 책임: 데이터 분석"]
        A2["🔧 툴: data_query, stats_calc, chart_gen"]
        A3["⚡ 모드: 비동기 (v0.5+)"]
    end

    subgraph "스포크 4: Reviewer Agent"
        V1["📋 책임: 품질 검토"]
        V2["🔧 툴: fact_check, code_review, quality_score"]
        V3["⚡ 모드: 비동기 (v0.5+)"]
    end
```

---

## 7. 상태(State) 설계 — Pydantic v2 기반

### 7.1 상태 스키마 전체 정의

```python
from pydantic import BaseModel, Field, field_validator
from typing import Annotated, Literal, Optional
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages
from datetime import datetime

class TaskDecomposition(BaseModel):
    """서브태스크 분해 결과 구조"""
    agent: str
    subtask: str
    status: Literal["pending", "in_progress", "completed", "failed"]
    assigned_at: Optional[datetime] = None

class AgentError(BaseModel):
    """에러 추적 구조"""
    agent: str
    error: str
    timestamp: datetime = Field(default_factory=datetime.utcnow)
    retry_count: int = 0

class HubSpokeState(BaseModel):
    """
    Hub and Spoke 공유 상태
    LangGraph v1.1 + version='v2' 호출 시 자동 Pydantic 강제 변환
    """
    
    # ── 대화 히스토리 ──────────────────────────────────────────
    # add_messages 리듀서: 마지막 쓰기 덮어쓰기 대신 메시지 누적
    messages: Annotated[list[BaseMessage], add_messages] = Field(default_factory=list)
    
    # ── 워크플로우 제어 ────────────────────────────────────────
    current_task: str = Field(default="", description="현재 처리 중인 메인 태스크")
    task_decomposition: list[TaskDecomposition] = Field(default_factory=list)
    next_agent: Literal[
        "research_agent", "coder_agent",
        "analyst_agent", "reviewer_agent", "FINISH"
    ] = Field(default="FINISH")
    
    # ── 반복 제어 ──────────────────────────────────────────────
    iteration_count: int = Field(default=0, ge=0)
    max_iterations: int = Field(default=10, ge=1, le=50)
    
    # ── 에이전트 결과 저장소 ──────────────────────────────────
    research_results: list[str] = Field(default_factory=list)
    code_artifacts: list[str] = Field(default_factory=list)
    analysis_results: Optional[dict] = None
    review_feedback: Optional[dict] = None
    
    # ── 비동기 서브에이전트 상태 (Deep Agents v0.5+) ──────────
    async_tasks: dict[str, str] = Field(
        default_factory=dict,
        description="비동기 태스크 ID → 상태 매핑"
    )
    
    # ── 에러 추적 ──────────────────────────────────────────────
    errors: list[AgentError] = Field(default_factory=list)
    
    # ── 최종 출력 ──────────────────────────────────────────────
    final_response: Optional[str] = None
    
    class Config:
        arbitrary_types_allowed = True

    @field_validator("iteration_count")
    def validate_iteration(cls, v, values):
        max_iter = values.data.get("max_iterations", 10)
        if v > max_iter:
            raise ValueError(f"iteration_count({v})가 max_iterations({max_iter})를 초과했습니다")
        return v
```

### 7.2 리듀서 전략 시각화

```mermaid
graph LR
    subgraph "덮어쓰기 리듀서 (기본값)"
        OW["current_task\nnext_agent\nfinal_response\n마지막 쓰기 승리"]
    end

    subgraph "누적 리듀서"
        AC1["messages\nadd_messages 리듀서\n순서 보장 누적"]
        AC2["research_results\noperator.add\n목록 누적"]
        AC3["errors\n커스텀 리듀서\n에러 히스토리 유지"]
    end

    subgraph "Pydantic 자동 검증"
        PY["iteration_count\nfield_validator\n범위 초과 즉시 에러"]
    end
```

---

## 8. 라우팅(Routing) 설계

### 8.1 조건부 엣지 구현

```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command

def route_to_agent(state: HubSpokeState) -> str:
    """
    Supervisor의 결정을 바탕으로 다음 노드를 결정하는 라우터.
    안전 상한선(max_iterations) 체크가 최우선.
    """
    
    # 1. 안전 상한선 우선 체크
    if state.iteration_count >= state.max_iterations:
        return END
    
    # 2. Supervisor의 라우팅 결정 반영
    if state.next_agent == "FINISH":
        return END
    
    return state.next_agent

def build_hub_spoke_graph(checkpointer=None) -> "CompiledGraph":
    """Hub and Spoke StateGraph 구성 및 컴파일"""
    
    builder = StateGraph(HubSpokeState)
    
    # ── 노드 등록 ──────────────────────────────────────────────
    builder.add_node("supervisor",      supervisor_node)
    builder.add_node("research_agent",  research_agent_node)
    builder.add_node("coder_agent",     coder_agent_node)
    builder.add_node("analyst_agent",   analyst_agent_node)
    builder.add_node("reviewer_agent",  reviewer_agent_node)
    
    # ── 진입점 ────────────────────────────────────────────────
    builder.add_edge(START, "supervisor")
    
    # ── Supervisor → 서브에이전트: 조건부 동적 라우팅 ──────────
    builder.add_conditional_edges(
        "supervisor",
        route_to_agent,
        {
            "research_agent":  "research_agent",
            "coder_agent":     "coder_agent",
            "analyst_agent":   "analyst_agent",
            "reviewer_agent":  "reviewer_agent",
            END:               END,
        }
    )
    
    # ── 서브에이전트 → Supervisor: Hub-Spoke 핵심 원칙 ──────────
    # 모든 스포크는 반드시 허브로 결과를 반환한다
    for spoke in ["research_agent", "coder_agent", "analyst_agent", "reviewer_agent"]:
        builder.add_edge(spoke, "supervisor")
    
    return builder.compile(
        checkpointer=checkpointer,
        # 선택적 인터럽트 포인트
        # interrupt_before=["reviewer_agent"],
    )
```

### 8.2 라우팅 플로우

```mermaid
flowchart TD
    Start(["START"]) --> Supervisor

    Supervisor["🎯 Supervisor\n태스크 분석 + 라우팅 결정"]

    Supervisor --> Safety{"iteration_count\n≥ max_iterations?"}
    Safety -->|"YES"| END1(["END\n(안전 상한선 초과)"])
    Safety -->|"NO"| Router{"route_to_agent()\n조건부 엣지"}

    Router -->|"FINISH"| END2(["END\n(정상 완료)"])
    Router -->|"research_agent"| R["🔍 Research Agent\n(동기)"]
    Router -->|"coder_agent"| C["💻 Coder Agent\n(동기)"]
    Router -->|"analyst_agent"| A["📊 Analyst Agent\n(비동기 v0.5+)"]
    Router -->|"reviewer_agent"| V["🔎 Reviewer Agent\n(비동기 v0.5+)"]

    R -->|"결과 + next_agent 업데이트"| Supervisor
    C -->|"결과 + next_agent 업데이트"| Supervisor
    A -.->|"비블로킹 결과 기록"| Supervisor
    V -.->|"비블로킹 결과 기록"| Supervisor

    style Supervisor fill:#d4b8f0,stroke:#9c6dba,stroke-width:2px
    style Safety fill:#f0d4d4,stroke:#c84a4a
    style END1 fill:#f0d4d4,stroke:#c84a4a
    style END2 fill:#b8f0c8,stroke:#4ab87a
```

---

## 9. 비동기 서브에이전트 (Deep Agents v0.5)

### 9.1 개념: 비블로킹 배경 태스크

Deep Agents v0.5.0에서 도입된 비동기 서브에이전트는 허브가 특정 스포크의 완료를 기다리는 동안 사용자가 계속 인터랙션할 수 있게 한다. 장시간 실행되는 분석·검토 작업에 특히 유용하다.

비동기 서브에이전트는 **Agent Protocol을 구현하는 어떤 서버와도 통신**할 수 있다. LangSmith Deployment가 가장 간편한 경로이지만, 다음 세 가지 자체 호스팅 방식도 완전히 지원된다. ① `url` 필드를 생략한 **ASGI 트랜스포트**(동일 서버 내 in-process 실행, 네트워크 없음), ② LangChain 공식 자체 호스팅인 **LangGraph Standalone Server**(PostgreSQL + Redis 기반), ③ LangSmith Deployment의 오픈소스 대체제인 **Aegra**(완전한 벤더 독립). 단순한 병렬 실행만 필요하다면 `asyncio.gather`로도 충분하다. 각 방식의 상세 비교는 **별첨 A**를 참조한다.

```mermaid
sequenceDiagram
    participant U as 사용자
    participant S as Supervisor (허브)
    participant Sync as Research Agent (동기)
    participant Async as Analyst Agent (비동기)

    U->>S: 종합 분석 요청
    S->>Sync: 리서치 위임 (동기)
    Sync-->>S: 리서치 결과 반환

    S->>Async: 분석 위임 (비동기 — 비블로킹)
    Note over S: 분석 완료를 기다리지 않음
    S-->>U: "분석 진행 중... 다른 질문 가능합니다"

    U->>S: 추가 질문
    S-->>U: 즉시 응답 (분석과 무관)

    Async-->>S: (백그라운드) 분석 결과 상태에 기록
    S-->>U: 분석 완료 알림 + 취합 결과
```

### 9.2 비동기 서브에이전트 구현 패턴

```python
from deepagents import create_deep_agent

# 비동기 실행을 지원하는 에이전트 정의
analyst_agent = create_deep_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-6"),
    tools=[data_query_tool, chart_gen_tool],
    name="analyst_agent",
    # 비동기 모드 활성화 (LangSmith Deployment 필요)
    async_execution=True,
)

# Supervisor에서 비동기 태스크 시작
def supervisor_node(state: HubSpokeState) -> Command:
    if needs_async_analysis(state):
        # 비동기로 분석 에이전트 실행
        task_id = start_async_subagent(analyst_agent, state.research_results)
        
        return Command(
            goto="research_agent",  # 동기 에이전트로 계속 진행
            update={
                "async_tasks": {**state.async_tasks, "analysis": task_id},
                "iteration_count": state.iteration_count + 1,
            }
        )
```

### 9.3 멀티모달 지원 (v0.5 신규)

Deep Agents v0.5의 `read_file` 툴은 이제 PDF, 오디오, 비디오 파일을 기본 지원한다. Hub and Spoke에서 멀티모달 서브에이전트를 구성할 때 별도의 파서 없이 바로 활용할 수 있다.

```python
from deepagents.tools import read_file

# PDF, 오디오, 비디오, 이미지 모두 처리 가능 (v0.5+)
multimodal_agent = create_deep_agent(
    model=init_chat_model("anthropic:claude-sonnet-4-6"),
    tools=[read_file],  # 단일 툴로 모든 파일 형식 처리
    name="multimodal_analyst",
)
```

---

## 10. 미들웨어 레이어 (LangChain v1.1/v1.2)

LangChain v1.1.0에서 도입된 미들웨어 체계는 에이전트 로직과 인프라 관심사(재시도, 컨텍스트 관리, 콘텐츠 모더레이션)를 명확히 분리한다.

### 10.1 미들웨어 아키텍처

```mermaid
graph LR
    Input["사용자 입력"] --> Moderation["🛡️ 콘텐츠 모더레이션\n미들웨어\n(v1.1+)"]
    Moderation --> Retry["🔄 모델 재시도\n미들웨어\n(지수 백오프, v1.1+)"]
    Retry --> Summarization["📝 컨텍스트 요약\n미들웨어\n(ContextOverflowError 자동 트리거, v0.4+)"]
    Summarization --> Agent["🎯 에이전트\n핵심 로직"]
    Agent --> Output["출력"]
```

### 10.2 미들웨어 구성

```python
from langchain.chat_models import init_chat_model
from langchain.middleware import (
    ModelRetryMiddleware,
    SummarizationMiddleware,
)
from langchain.integrations.middleware.openai import ContentModerationMiddleware

# 모델 프로파일 기반 미들웨어 구성 (v1.1+)
base_model = init_chat_model("anthropic:claude-sonnet-4-6")
print(base_model.profile.context_window)  # 자동으로 컨텍스트 창 크기 파악

# 미들웨어 체이닝
model_with_middleware = (
    base_model
    # 1. 실패 시 지수 백오프로 자동 재시도
    | ModelRetryMiddleware(max_retries=3, backoff_factor=2.0)
    # 2. ContextOverflowError 발생 시 자동 요약
    | SummarizationMiddleware(
        target_token_count=4000,
        # 모델 프로파일 기반 트리거 (v1.1)
        trigger_on_overflow=True,
    )
)

# ContentModerationMiddleware는 사용자 입력과 모델 출력 모두 검사
moderated_model = ContentModerationMiddleware(
    model=model_with_middleware,
    check_input=True,
    check_output=True,
    check_tool_results=True,
)
```

### 10.3 `ContextOverflowError` 자동 요약 흐름

Deep Agents v0.4부터 LLM이 컨텍스트 창 한계에 도달하면 `ContextOverflowError`를 발생시키고, `SummarizationMiddleware`가 자동으로 대화 요약을 수행한다. `langchain-anthropic`과 `langchain-openai` 모두 이 프로토콜을 지원한다.

```python
# v0.4+: ContextOverflowError 자동 처리
# 별도 설정 없이 아래 미들웨어만 추가하면 자동 동작

from langchain.middleware import SummarizationMiddleware

model = init_chat_model("anthropic:claude-sonnet-4-6") | SummarizationMiddleware()
# 이후 컨텍스트 창이 초과되면 자동으로 요약 후 계속 실행
```

---

## 11. 메모리 및 영속성 설계

### 11.1 메모리 아키텍처

```mermaid
graph TB
    subgraph "단기 메모리 (Thread 수준)"
        ST1["체크포인터\n현재 대화 세션 상태 영속화"]
        ST2["InMemorySaver → 개발·테스트"]
        ST3["PostgresSaver → 프로덕션"]
        ST1 --> ST2
        ST1 --> ST3
    end

    subgraph "장기 메모리 (Session 간)"
        LT1["Store\n세션 간 공유 지식"]
        LT2["InMemoryStore → 개발"]
        LT3["PostgresStore → 프로덕션"]
        LT1 --> LT2
        LT1 --> LT3
    end

    subgraph "Deep Agents v0.5 백엔드"
        DA1["StateBackend()\n직접 인스턴스화 (v0.5+)"]
        DA2["StoreBackend()\n직접 인스턴스화 (v0.5+)"]
        DA3["⚠️ 팩토리 방식 Deprecated\nbackend=(lambda rt: StateBackend(rt))"]
    end
```

### 11.2 체크포인터 설정 (최신 방식)

```python
# 개발 환경
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.store.memory import InMemoryStore

checkpointer = InMemorySaver()
store = InMemoryStore()

# 프로덕션 환경 (PostgreSQL)
from langgraph.checkpoint.postgres import PostgresSaver
from psycopg import Connection

conn = Connection.connect(DATABASE_URL, autocommit=True)
checkpointer = PostgresSaver(conn)
checkpointer.setup()  # 스키마 초기화 (최초 1회)

# Deep Agents v0.5: 직접 인스턴스화 (팩토리 방식 deprecated)
from deepagents.backends import StateBackend, StoreBackend

state_backend = StateBackend()   # ✅ v0.5+ 권장
store_backend = StoreBackend()   # ✅ v0.5+ 권장
# backend=(lambda rt: StateBackend(rt))  # ❌ Deprecated

# 컴파일
app = build_hub_spoke_graph(checkpointer=checkpointer)
```

### 11.3 Thread 기반 세션 관리

```python
import uuid

# 새 세션
thread_id = str(uuid.uuid4())
config = {
    "configurable": {"thread_id": thread_id},
    "recursion_limit": 25,  # 그래프 수준 안전 상한선
}

# v1.1 타입 안전 호출
from langgraph.types import GraphOutput

result: GraphOutput = app.invoke(
    HubSpokeState(
        messages=[HumanMessage(content="EV 시장을 종합 분석해줘")],
        max_iterations=15,
    ),
    config=config,
    version="v2",     # ← LangGraph v1.1 타입 안전 모드
)

# 타입 안전한 결과 접근
final = result.value.final_response
interrupts = result.interrupts  # 인터럽트 목록 (HITL 시)
```

---

## 12. Observability 설계 — v2 스트리밍 포함

### 12.1 LangSmith 통합

```python
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-langsmith-api-key"
os.environ["LANGCHAIN_PROJECT"] = "hub-spoke-v2"

# 이후 모든 그래프 실행이 LangSmith에 자동 추적됨
# 서브에이전트별 실행 시간, 토큰 사용량, 에러가 대시보드에 표시됨
```

### 12.2 v2 스트리밍 기반 실시간 모니터링

```python
from langgraph.types import StreamPart
import asyncio

async def stream_hub_spoke(task: str, thread_id: str):
    """v1.1 타입 안전 스트리밍으로 Hub-Spoke 실행 모니터링"""
    
    config = {"configurable": {"thread_id": thread_id}, "recursion_limit": 25}
    
    async for chunk in app.astream(
        HubSpokeState(messages=[HumanMessage(content=task)]),
        config=config,
        stream_mode="updates",
        version="v2",  # StreamPart TypedDict 반환
    ):
        chunk: StreamPart
        event_type = chunk["type"]     # "updates", "values", "messages"
        namespace = chunk["ns"]        # 어떤 서브그래프에서 발생했는지
        data = chunk["data"]

        # 네임스페이스로 어떤 스포크에서 이벤트가 왔는지 파악
        if "research_agent" in namespace:
            print(f"[Research] {data}")
        elif "coder_agent" in namespace:
            print(f"[Coder] {data}")
        elif "supervisor" in namespace:
            print(f"[Supervisor] 라우팅 결정: {data.get('next_agent', '?')}")
```

### 12.3 구조화 로거 (JSONL 포맷)

```python
import logging, json
from datetime import datetime

class HubSpokeLogger:
    """Hub and Spoke 전용 구조화 JSONL 로거"""
    
    def __init__(self, log_path: str = "logs/hub_spoke.jsonl"):
        self.logger = logging.getLogger("hub_spoke")
        handler = logging.FileHandler(log_path)
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.DEBUG)
    
    def _emit(self, event: str, **kwargs):
        entry = {
            "ts": datetime.utcnow().isoformat(),
            "event": event,
            **kwargs
        }
        self.logger.info(json.dumps(entry, ensure_ascii=False))
    
    def routing(self, from_node: str, to_node: str, iteration: int):
        self._emit("routing", from_node=from_node, to_node=to_node, iteration=iteration)
    
    def agent_result(self, agent: str, summary: str, tokens: int):
        self._emit("agent_result", agent=agent, summary=summary, tokens=tokens)
    
    def async_task_started(self, agent: str, task_id: str):
        self._emit("async_task_started", agent=agent, task_id=task_id)
    
    def error(self, agent: str, error: str, retry_count: int = 0):
        self._emit("error", agent=agent, error=error, retry_count=retry_count)

# 사용 예시 → 파싱 가능한 JSONL 로그 생성
# {"ts": "2026-05-05T10:23:11", "event": "routing", "from_node": "supervisor", "to_node": "research_agent", "iteration": 1}
```

---

## 13. Human-in-the-Loop 설계

### 13.1 인터럽트와 타임 트래블 (v1.1 수정)

LangGraph v1.1에서 인터럽트 + 타임 트래블의 버그(오래된 RESUME 값 재사용 문제)가 수정되었다. 이제 Human-in-the-Loop 패턴이 프로덕션 환경에서 신뢰할 수 있게 되었다.

```python
# 인터럽트 포인트 설정
app = build_hub_spoke_graph(checkpointer=checkpointer)
app = app.compile(interrupt_before=["reviewer_agent"])

# v1.1: GraphOutput.interrupts로 명시적 인터럽트 접근
result: GraphOutput = app.invoke(initial_state, config=config, version="v2")

if result.interrupts:
    # 사람이 리뷰해야 할 인터럽트가 있음
    for interrupt in result.interrupts:
        print(f"검토 필요: {interrupt}")
    
    # 사람이 상태 수정 후 재개
    app.update_state(config, {"review_approved": True})
    final_result: GraphOutput = app.invoke(None, config=config, version="v2")
```

### 13.2 HITL 시퀀스 다이어그램

```mermaid
sequenceDiagram
    participant U as 사용자
    participant G as LangGraph 런타임
    participant S as Supervisor
    participant C as Coder Agent
    participant R as Reviewer Agent
    participant H as 사람 검토자

    U->>G: invoke(task, version="v2")
    G->>S: Supervisor 실행
    S->>C: Coder 위임
    C->>S: 코드 결과 반환
    S->>G: next_agent="reviewer_agent"
    G->>G: interrupt_before=reviewer_agent 감지
    G->>U: GraphOutput(interrupts=[ReviewInterrupt])

    U->>H: 코드 검토 요청
    H->>U: 승인 (또는 수정)
    U->>G: update_state + invoke(None, version="v2")

    G->>R: Reviewer 실행 (재개)
    R->>S: 리뷰 결과 반환
    S->>G: next_agent="FINISH"
    G->>U: GraphOutput(value=final_state)
```

---

## 14. 디렉토리 구조 및 코드 예시

### 14.1 권장 프로젝트 구조

```
hub_spoke_langgraph/
├── main.py                          # 진입점
├── config.py                        # 환경 변수 및 상수
├── requirements.txt
│
├── graph/
│   ├── __init__.py
│   ├── builder.py                   # StateGraph 구성·컴파일
│   ├── state.py                     # HubSpokeState Pydantic 모델
│   └── router.py                    # route_to_agent 조건부 엣지
│
├── agents/
│   ├── __init__.py
│   ├── supervisor.py                # Supervisor (허브)
│   ├── research_agent.py            # 리서치 스포크 (동기)
│   ├── coder_agent.py               # 코더 스포크 (동기)
│   ├── analyst_agent.py             # 분석 스포크 (비동기 v0.5+)
│   └── reviewer_agent.py            # 리뷰어 스포크 (비동기 v0.5+)
│
├── tools/
│   ├── search_tools.py              # extras로 Anthropic 특화 설정 (v1.2+)
│   ├── code_tools.py
│   ├── data_tools.py
│   └── review_tools.py
│
├── middleware/
│   ├── __init__.py
│   └── model_factory.py             # 미들웨어 체이닝 팩토리 (v1.1+)
│
├── prompts/                         # 마크다운 프롬프트 파일
│   ├── supervisor.md
│   ├── research_agent.md
│   ├── coder_agent.md
│   ├── analyst_agent.md
│   └── reviewer_agent.md
│
├── memory/
│   ├── checkpointer.py              # 환경별 체크포인터 팩토리
│   └── store.py                     # 장기 메모리 스토어
│
├── observability/
│   └── logger.py                    # JSONL 구조화 로거
│
├── tests/
│   ├── unit/
│   │   ├── test_state.py            # Pydantic 상태 검증 테스트
│   │   ├── test_router.py           # 라우팅 로직 테스트
│   │   └── test_middleware.py
│   └── integration/
│       └── test_full_graph.py       # 전체 그래프 통합 테스트
│
└── logs/
    └── hub_spoke.jsonl              # 런타임 구조화 로그
```

### 14.2 `main.py` 전체 실행 예시

```python
# main.py
import asyncio
from langchain_core.messages import HumanMessage
from langgraph.types import GraphOutput

from graph.builder import build_hub_spoke_graph
from graph.state import HubSpokeState
from memory.checkpointer import get_checkpointer
from observability.logger import HubSpokeLogger

import uuid, os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "hub-spoke-v2"

logger = HubSpokeLogger()

async def main():
    checkpointer = await get_checkpointer()  # 환경별 자동 선택
    app = build_hub_spoke_graph(checkpointer=checkpointer)
    
    thread_id = str(uuid.uuid4())
    config = {
        "configurable": {"thread_id": thread_id},
        "recursion_limit": 25,
    }
    
    # LangGraph v1.1: version="v2" 타입 안전 호출
    result: GraphOutput = await app.ainvoke(
        HubSpokeState(
            messages=[HumanMessage(content="EV 배터리 시장을 종합 분석해줘")],
            max_iterations=15,
        ),
        config=config,
        version="v2",
    )
    
    # 타입 안전한 결과 접근
    if result.interrupts:
        print("사람 검토 필요:", result.interrupts)
    else:
        print("최종 결과:", result.value.final_response)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 15. 배포 아키텍처

### 15.1 환경별 배포 전략

```mermaid
graph TB
    subgraph "개발 (Local)"
        D1["langgraph dev\nHot reload 지원"]
        D1a["InMemorySaver"]
        D1 --- D1a
    end

    subgraph "스테이징 / 자체 호스팅"
        D2["Docker Compose\n또는 Kubernetes"]
        D2a["PostgreSQL 체크포인터"]
        D2b["Redis 세션 캐시"]
        D2 --> D2a
        D2 --> D2b
    end

    subgraph "프로덕션 (LangSmith Deployment)"
        D3["LangSmith Deployment\n(비동기 서브에이전트 필수)"]
        D3a["자동 스케일링"]
        D3b["빌트인 추적·평가"]
        D3c["Deep Agents v0.5 비동기 지원"]
        D3 --> D3a
        D3 --> D3b
        D3 --> D3c
    end

    note["⚠️ 비동기 서브에이전트는\nLangSmith Deployment 필수"]
    D3c --- note
```

### 15.2 docker-compose.yml

```yaml
version: '3.9'
services:
  hub_spoke_api:
    build: .
    ports: ["8000:8000"]
    environment:
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      LANGCHAIN_API_KEY: ${LANGCHAIN_API_KEY}
      LANGCHAIN_TRACING_V2: "true"
      LANGCHAIN_PROJECT: "hub-spoke-v2"
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/hub_spoke
    depends_on: [postgres]

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: hub_spoke
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

## 16. 안티패턴과 주의사항

### 16.1 서브에이전트 직접 통신 (치명적)

```python
# ❌ Hub-Spoke 위반: 스포크가 다른 스포크 직접 호출
def research_agent_node(state: HubSpokeState):
    results = do_research(state)
    coder_results = coder_agent_node(state)  # 절대 금지
    return {"research_results": results}

# ✅ 올바른 방식: 상태에 결과를 기록하고 Supervisor에게 반환
def research_agent_node(state: HubSpokeState):
    results = do_research(state)
    return {
        "research_results": results,
        # Supervisor가 다음 라우팅 결정
    }
```

### 16.2 전체 컨텍스트 무분별 전달

```python
# ❌ 전체 메시지 히스토리를 서브에이전트에 전달 → 토큰 낭비·혼란
def supervisor_node(state: HubSpokeState) -> Command:
    return Command(
        goto="research_agent",
        update={"messages": state.messages}  # 수백 토큰의 불필요한 히스토리
    )

# ✅ 해당 서브태스크에 필요한 최소 컨텍스트만 전달
def supervisor_node(state: HubSpokeState) -> Command:
    task_context = extract_minimal_context(state, for_agent="research_agent")
    return Command(
        goto="research_agent",
        update={"current_task": task_context}
    )
```

### 16.3 v1.1 이전 방식의 dict 반환 유지

```python
# ❌ v1.0 방식: dict 키 오타 시 런타임에서야 발견
result = app.invoke(state, config=config)
answer = result.get("final_respons")  # 오타! None 반환, 에러 없음

# ✅ v1.1 방식: Pydantic 속성 접근, 오타 시 즉시 AttributeError
result: GraphOutput = app.invoke(state, config=config, version="v2")
answer = result.value.final_response  # 오타 시 AttributeError
```

### 16.4 Agent Protocol 서버 없이 비동기 서브에이전트 실행 시도

비동기 서브에이전트는 LangSmith Deployment **전용**이 아니지만, **Agent Protocol을 구현한 서버**가 반드시 필요하다. 서버가 전혀 없는 상태에서 `async_execution=True`만 설정하면 실행이 실패한다.

```python
# ❌ Agent Protocol 서버 없이 async_execution=True 설정
# → 서버 연결 실패, 태스크 ID 조회 불가
analyst_agent = create_deep_agent(async_execution=True)

# ✅ 가장 간단한 자체 서버: ASGI 트랜스포트 (url 생략)
# langgraph.json에 모든 그래프를 등록하고 `langgraph dev`로 실행하면 됨
from deepagents import AsyncSubAgent, create_deep_agent

analyst_agent_spec = AsyncSubAgent(
    name="analyst",
    description="데이터 분석 에이전트",
    graph_id="analyst",
    # url 생략 → ASGI 트랜스포트 (같은 langgraph.json 내 in-process 실행)
    # LangSmith 없이도 동작
)

supervisor = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    subagents=[analyst_agent_spec],
)

# ✅ 진짜 서버가 없는 로컬 개발 환경이라면: asyncio.gather로 대체
# 비블로킹 HITL은 불가하지만 병렬 실행 자체는 동일하게 가능
results = await asyncio.gather(
    analyst_agent.ainvoke(state),
    reviewer_agent.ainvoke(state),
    return_exceptions=True,  # 하나 실패 시 다른 태스크 계속 실행
)
```

### 16.5 Deep Agents v0.5 Backend 팩토리 방식 사용

```python
# ❌ Deprecated (v0.5에서 폐기)
backend = (lambda rt: StateBackend(rt))

# ✅ 직접 인스턴스화 (v0.5+)
backend = StateBackend()
```

---

## 17. 프레임워크 선택 근거

### 17.1 2026년 5월 기준 비교

| 기준 | LangGraph | CrewAI | OpenAI Agents SDK | Google ADK |
|---|---|---|---|---|
| Hub-Spoke 네이티브 | ✅ Supervisor 패턴 | ⚠️ 계층적 크루 | ✅ 핸드오프 | ✅ 루트-서브 |
| v2 타입 안전 | ✅ v1.1 GraphOutput | ❌ | ❌ | ❌ |
| 비동기 서브에이전트 | ✅ Deep Agents v0.5 | ❌ | ⚠️ 제한적 | ⚠️ |
| Pydantic 상태 자동 변환 | ✅ v1.1+ | ❌ | ❌ | ❌ |
| ContextOverflow 자동 처리 | ✅ v0.4+ | ❌ | ❌ | ❌ |
| 미들웨어 체계 | ✅ v1.1 내장 | ❌ | ❌ | ⚠️ |
| HITL + 타임 트래블 | ✅ v1.1 수정 | ❌ | ⚠️ | ⚠️ |
| 멀티모달 서브에이전트 | ✅ v0.5 read_file | ⚠️ | ⚠️ | ✅ Gemini |
| LLM 모델 독립성 | ✅ 모든 공급자 | ✅ | ❌ OpenAI 중심 | ⚠️ Gemini 중심 |
| 프로덕션 사례 | ✅ Klarna, Replit, Elastic | ⚠️ 중간 | ✅ | ⚠️ |

### 17.2 LangGraph 선택의 근거 (세 가지)

첫째, **가장 풍부한 Hub-Spoke 지원 기반 구조**다. `create_supervisor()`, 조건부 엣지, 서브그래프가 허브-앤-스포크의 각 요소를 언어 수준에서 표현하도록 설계되어 있다. 단순히 해킹으로 구현하는 것이 아니라, 프레임워크가 이 패턴을 위해 설계되었다.

둘째, **v1.1의 타입 안전성**이다. `GraphOutput`과 Pydantic 자동 강제 변환은 멀티에이전트 시스템에서 흔히 발생하는 런타임 타입 불일치 버그를 컴파일 시점에 잡는다. 10개 에이전트가 공유 상태를 읽고 쓰는 시스템에서 이 안전망의 가치는 매우 크다.

셋째, **Deep Agents v0.5의 비동기 서브에이전트**다. 긴 리서치나 분석 작업을 비동기로 처리하면서 사용자 인터랙션을 유지할 수 있다는 것은 프로덕션 사용자 경험에서 결정적인 차이를 만들어낸다.

---

## 18. 파워포인트 구성 가이드

이 문서를 파워포인트로 전환할 때 권장하는 슬라이드 구성이다. v2.0의 새로운 내용이 추가되어 총 20장으로 구성된다.

| 번호 | 제목 | 유형 | 핵심 내용 | 디자인 힌트 |
|---|---|---|---|---|
| 1 | 표지 | 표지 | 아키텍처 정의서 v2.0, 날짜 | 어두운 배경, LangGraph 로고 컬러 |
| 2 | v2.0 주요 변경사항 | 변경사항 표 | 이전 vs 현재 비교표 | 초록(개선)/빨강(제거) 색상 코딩 |
| 3 | 목차 | 목차 | 전체 구조 | 섹션별 번호 명시 |
| 4 | 왜 멀티에이전트인가? | 문제 정의 | 단일 에이전트 한계 3가지 | 아이콘 3개 + 한계점 |
| 5 | Hub and Spoke 개념 | 개념 | 항공 노선 비유, 핵심 규칙 | 항공 노선 시각화 |
| 6 | 기술 스택 버전표 | 표 | 패키지별 버전·날짜·변경사항 | 표, 최신 버전 하이라이트 |
| 7 | 전체 아키텍처 다이어그램 | 다이어그램 | Mermaid → PPT SmartArt | 비동기 스포크는 점선 표현 |
| 8 | LangGraph 세 요소 | 개념 | State / Node / Edge | 삼각형 or 삼색 다이어그램 |
| 9 | v1.1 핵심: 타입 안전 호출 | 기술 | version="v2", GraphOutput | 코드 블록 (이전/이후 비교) |
| 10 | Pydantic 상태 설계 | 기술 상세 | BaseModel 스키마 + 리듀서 | 표 + 코드 예시 |
| 11 | 동적 라우팅 플로우 | 플로우 | add_conditional_edges 시각화 | 플로우차트 (안전 상한선 분기 포함) |
| 12 | 비동기 서브에이전트 (v0.5) | 신규 기능 | 비블로킹 시퀀스 다이어그램 | 실선(동기)/점선(비동기) 구분 |
| 13 | 미들웨어 레이어 (v1.1) | 신규 기능 | 재시도·요약·모더레이션 체이닝 | 파이프라인 다이어그램 |
| 14 | 메모리 아키텍처 | 인프라 | 단기/장기 메모리 + Backend 변경 | 레이어 다이어그램 |
| 15 | v2 스트리밍 모니터링 | 기술 | StreamPart + ns 추적 | 코드 예시 |
| 16 | Human-in-the-Loop | 시나리오 | 시퀀스 다이어그램 + v1.1 수정 | 수영 레인 다이어그램 |
| 17 | 프로젝트 구조 | 코드 구조 | 디렉토리 트리 | 폰트 크기 줄여 트리 표현 |
| 18 | 배포 전략 | 인프라 | 3 환경 비교, 비동기 조건 | 3열 비교 카드 |
| 19 | 안티패턴 5가지 | 주의사항 | ❌/✅ 코드 비교 | 빨강/초록 2열 코드 박스 |
| 20 | 프레임워크 비교 & 결론 | 결론 | 비교 표 + 선택 근거 3가지 | 표 + 결론 박스 |

### 시각화 권장사항

Mermaid 다이어그램을 PowerPoint로 변환할 때:
- **전체 아키텍처 (순환 그래프):** SmartArt "계층형" 또는 수동 원형 배치. 비동기 스포크는 점선 테두리로 구분
- **플로우차트:** SmartArt "의사결정" 또는 도형 + 화살표 수동 배치. 다이아몬드(◇)는 조건 분기 표현
- **시퀀스 다이어그램 (HITL):** 수영 레인 슬라이드. 각 참여자를 열로, 시간 흐름을 행으로 배치
- **비동기 vs 동기:** 실선 화살표(동기) / 점선 화살표(비동기) 일관되게 사용
- **코드 비교 (❌/✅):** 슬라이드를 좌우로 분할. 왼쪽에 어두운 배경 코드 박스(❌), 오른쪽에 초록 테두리 코드 박스(✅)

---

## 부록: 빠른 참조 카드

```python
# ── LangGraph v1.1 Hub and Spoke 핵심 패턴 ────────────────────

from pydantic import BaseModel, Field
from typing import Annotated, Literal, Optional
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.types import GraphOutput, StreamPart
from langgraph.checkpoint.memory import InMemorySaver
from langgraph_supervisor import create_supervisor
from langchain.chat_models import init_chat_model

# 1. Pydantic 상태 정의
class State(BaseModel):
    messages: Annotated[list, add_messages] = Field(default_factory=list)
    next_agent: Literal["agent_a", "agent_b", "FINISH"] = "FINISH"
    result: Optional[str] = None
    class Config: arbitrary_types_allowed = True

# 2. 에이전트 생성
model = init_chat_model("anthropic:claude-sonnet-4-6")
agent_a = create_react_agent(model, tools=[...], name="agent_a")
agent_b = create_react_agent(model, tools=[...], name="agent_b")

# 3. 그래프 구성
builder = StateGraph(State)
builder.add_node("supervisor", supervisor_fn)
builder.add_node("agent_a", agent_a)
builder.add_node("agent_b", agent_b)
builder.add_edge(START, "supervisor")
builder.add_conditional_edges("supervisor", lambda s: s.next_agent,
    {"agent_a": "agent_a", "agent_b": "agent_b", "FINISH": END})
for spoke in ["agent_a", "agent_b"]:
    builder.add_edge(spoke, "supervisor")  # 항상 허브로 복귀

# 4. 컴파일 + 실행 (v1.1 타입 안전 모드)
app = builder.compile(checkpointer=InMemorySaver())
result: GraphOutput = app.invoke(
    State(), 
    config={"configurable": {"thread_id": "t1"}, "recursion_limit": 20},
    version="v2"  # ← LangGraph v1.1 핵심
)
print(result.value.result)
print(result.interrupts)
```

---

- *작성 일자: 2026-05-05*
- *Changelog 출처: https://docs.langchain.com/oss/python/releases/changelog*

---

---

## 별첨 A. 비동기 서브에이전트 — LangSmith Deployment 없이 운영하기

> **작성 배경:** 본문 9장에서 "비동기 서브에이전트는 LangSmith Deployment 환경에서만 지원된다"고 기술했으나, 이는 정확하지 않다. 공식 문서(`docs.langchain.com/oss/python/deepagents/async-subagents`)에 따르면 비동기 서브에이전트는 **Agent Protocol을 구현하는 어떤 서버**와도 통신할 수 있다. LangSmith Deployment는 그 중 하나의 옵션일 뿐이며, 자체 호스팅도 완전히 지원된다. 이 별첨은 그 대안들을 상세히 다룬다.

---

### A.1 오해의 출처와 실제 제약 조건

Deep Agents v0.5.0 릴리스 노트의 원문은 다음과 같다:

> *"Async subagents: Deep Agents can launch non-blocking background tasks… **Requires LangSmith Deployment for sub-agents.**"*

이 문장은 "LangSmith Deployment만 가능하다"는 의미가 아니다. 정확한 해석은 다음과 같다: **비동기 서브에이전트는 에이전트를 상시 실행 상태로 서빙하는 서버가 필요하며**, LangSmith Deployment가 그 조건을 가장 간편하게 충족시켜주는 공식 경로라는 것이다.

공식 문서의 실제 정의를 보면:

> *"Async subagents communicate with any server that implements the Agent Protocol. You can use LangSmith Deployments, or **self-host any Agent Protocol-compatible server**."*

즉, 선택지는 두 가지다: **LangSmith Deployment 사용** 또는 **Agent Protocol 호환 서버 자체 호스팅**.

### A.2 왜 서버가 필요한가

비동기 서브에이전트가 일반적인 동기 실행과 다른 근본적인 이유는 **실행 모델**에 있다.

동기 서브에이전트는 Supervisor가 `ainvoke()`를 호출하고 결과가 반환될 때까지 대기한다. 이 경우 모든 실행이 하나의 프로세스 내에서 일어나기 때문에 별도의 서버가 필요 없다.

비동기 서브에이전트는 다르다. Supervisor가 `start_async_task`를 호출하면 즉시 **태스크 ID**만 반환받고 계속 실행된다. 서브에이전트는 **완전히 독립된 스레드**에서 자신의 상태를 유지하며 실행되고, Supervisor는 나중에 `check_async_task`로 진행 상황을 확인한다. 이 "독립된 스레드에서 상태를 유지하며 실행"이라는 요건이 상시 구동되는 서버를 필요로 만든다.

```mermaid
graph LR
    subgraph "동기 서브에이전트 (서버 불필요)"
        S1["Supervisor"] -->|"ainvoke() + await"| Sub1["서브에이전트"]
        Sub1 -->|"결과 반환"| S1
    end

    subgraph "비동기 서브에이전트 (서버 필요)"
        S2["Supervisor"] -->|"start_async_task\n(즉시 반환)"| Server["Agent Protocol\n서버"]
        Server -->|"독립 스레드 실행"| Sub2["서브에이전트"]
        S2 -->|"check_async_task\n(나중에 확인)"| Server
        Server -->|"태스크 상태/결과"| S2
    end
```

### A.3 Agent Protocol이란

Agent Protocol은 LangChain이 정의한 표준 HTTP 인터페이스다. 이 프로토콜을 구현하는 서버라면 어떤 것이든 비동기 서브에이전트의 백엔드로 사용할 수 있다. 핵심 엔드포인트는 다음과 같다:

- `POST /threads` — 새 대화 스레드 생성
- `POST /threads/{id}/runs` — 스레드에서 에이전트 실행 시작
- `GET /threads/{id}/runs/{run_id}` — 실행 상태 조회
- `POST /threads/{id}/runs/{run_id}/cancel` — 실행 취소
- `GET /threads/{id}/state` — 에이전트 최종 상태(결과) 조회

### A.4 LangSmith Deployment 없이 운영하는 4가지 방법

#### 방법 1: LangGraph Standalone Server (공식 자체 호스팅)

LangChain 공식 문서(`docs.langchain.com/langsmith/deploy-standalone-server`)에서 제공하는 자체 호스팅 방식이다. LangSmith의 **Control Plane(관리 대시보드)** 없이 **Data Plane(에이전트 서버)만** 독립적으로 배포한다.

```mermaid
graph TB
    subgraph "LangSmith Deployment (관리형)"
        LS1["Control Plane\n(LangSmith 클라우드)"]
        LS2["Data Plane\n(에이전트 실행)"]
        LS1 --> LS2
    end

    subgraph "Standalone Server (자체 호스팅)"
        ST1["langgraph build\n→ Docker 이미지"]
        ST2["PostgreSQL\n(상태 영속성)"]
        ST3["Redis\n(스트리밍 pub/sub)"]
        ST1 --> ST2
        ST1 --> ST3
    end

    note["Control Plane 없이\nData Plane만 운영\n→ LangSmith 종속성 최소화"]
    ST1 -.-> note
```

**필요한 환경변수:**
- `REDIS_URI` — 실시간 스트리밍을 위한 pub/sub 브로커
- `DATABASE_URI` — PostgreSQL (체크포인트 영속성)
- `LANGSMITH_API_KEY` — 라이선스 검증용 (트레이싱 비활성화 가능)

```yaml
# docker-compose.yml (Standalone Server)
services:
  langgraph-api:
    image: ${IMAGE_NAME}    # langgraph build로 생성한 이미지
    ports: ["8123:8000"]
    environment:
      REDIS_URI: redis://langgraph-redis:6379
      DATABASE_URI: postgres://postgres:postgres@langgraph-postgres:5432/postgres
      LANGSMITH_API_KEY: ${LANGSMITH_API_KEY}  # 라이선스 검증에만 사용
    depends_on: [langgraph-redis, langgraph-postgres]

  langgraph-redis:
    image: redis:6

  langgraph-postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: postgres
```

**장점:** 공식 지원, LangGraph SDK와 완전 호환, 최소 변경으로 이전 가능
**단점:** `LANGSMITH_API_KEY`가 여전히 필요 (라이선스 검증)

---

#### 방법 2: Aegra — LangSmith Deployment 오픈소스 대체제

`aegra`는 LangSmith Deployment의 **완전한 오픈소스 드롭인 대체제**다. 동일한 LangGraph SDK와 API를 사용하면서 PostgreSQL과 FastAPI 기반의 자체 인프라에서 실행된다.

```bash
pip install aegra-cli
aegra init          # 프로젝트 초기화
cp .env.example .env
uv sync
uv run aegra dev    # PostgreSQL + 개발 서버 시작
```

```python
from langgraph_sdk import get_client

# LangSmith 대신 로컬 Aegra 서버를 가리킴
client = get_client(url="http://localhost:2026")

# 이후 코드는 LangSmith Deployment 사용 시와 완전히 동일
assistant = await client.assistants.create(graph_id="supervisor")
thread = await client.threads.create()
async for chunk in client.runs.stream(
    thread_id=thread["thread_id"],
    assistant_id=assistant["assistant_id"],
    input={"messages": [{"type": "human", "content": "분석 시작"}]},
):
    print(chunk)
```

**핵심 특징:**
- Redis 작업 큐 기반 워커 아키텍처 (인스턴스당 30 동시 실행)
- 임대 기반(lease-based) 충돌 복구
- SSE 실시간 스트리밍 (크로스 인스턴스 pub/sub)
- PostgreSQL 체크포인트 영속화
- LangGraph SDK와 100% API 호환

**장점:** 완전한 벤더 종속 없음, LangSmith API 키 불필요, 에어갭(air-gap) 환경 지원
**단점:** 공식 지원 아님, 비교적 새로운 프로젝트 (안정성 검증 진행 중)

---

#### 방법 3: ASGI 트랜스포트 — 단일 서버 내 비동기 (LangSmith 불필요)

비동기 서브에이전트의 `url` 필드를 생략하면 **ASGI 트랜스포트**가 사용된다. 이 경우 HTTP 네트워크 호출 없이 **동일 프로세스 내에서** in-process 함수 호출로 라우팅된다. LangSmith Deployment 없이도 비동기의 핵심 이점(비블로킹 실행, 태스크 취소, 진행 상황 확인)을 모두 누릴 수 있다.

**요건:** Supervisor와 모든 서브에이전트가 동일한 `langgraph.json`에 등록되어야 한다.

```json
// langgraph.json (모든 그래프를 하나의 배포에 등록)
{
  "graphs": {
    "supervisor": "./src/supervisor.py:graph",
    "researcher": "./src/researcher.py:graph",
    "analyst":    "./src/analyst.py:graph",
    "coder":      "./src/coder.py:graph"
  }
}
```

```python
from deepagents import AsyncSubAgent, create_deep_agent

async_subagents = [
    AsyncSubAgent(
        name="researcher",
        description="리서치 에이전트",
        graph_id="researcher",
        # url 필드 없음 → ASGI 트랜스포트 (in-process, 네트워크 없음)
    ),
    AsyncSubAgent(
        name="analyst",
        description="데이터 분석 에이전트",
        graph_id="analyst",
        # url 필드 없음 → 동일하게 ASGI 트랜스포트
    ),
]

supervisor = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    subagents=async_subagents,
)
```

**장점:** LangSmith 불필요, 네트워크 레이턴시 없음, 설정 최소화
**단점:** 서브에이전트를 독립적으로 스케일 아웃할 수 없음. 모두 같은 서버에 묶임

---

#### 방법 4: 순수 asyncio.gather — 플랫폼 없는 병렬 실행

Deep Agents의 `AsyncSubAgent` 기계 장치 없이 **순수 Python asyncio**로 병렬 에이전트 실행을 구현한다. 가장 가벼운 방식이지만, 태스크 ID 추적, 취소, 중간 진행 확인 등의 기능은 직접 구현해야 한다.

```python
import asyncio
from langchain_core.messages import HumanMessage

async def run_agents_in_parallel(state: HubSpokeState) -> HubSpokeState:
    """
    LangSmith/Aegra 없이 asyncio.gather로 복수 에이전트 병렬 실행.
    결과 중 하나가 실패하면 전체가 취소되는 gather의 기본 동작에 주의.
    """
    
    # 독립적으로 실행할 수 있는 태스크들을 병렬로 실행
    analyst_task = analyst_agent.ainvoke(
        {"messages": [HumanMessage(content=state.current_task)]}
    )
    reviewer_task = reviewer_agent.ainvoke(
        {"messages": [HumanMessage(content=state.current_task)]}
    )
    
    # return_exceptions=True: 하나 실패해도 다른 태스크 계속 실행
    results = await asyncio.gather(
        analyst_task,
        reviewer_task,
        return_exceptions=True,
    )
    
    analyst_result, reviewer_result = results
    
    # 개별 에러 처리
    updates = {}
    if isinstance(analyst_result, Exception):
        updates["errors"] = [{"agent": "analyst", "error": str(analyst_result)}]
    else:
        updates["analysis_results"] = analyst_result
    
    if isinstance(reviewer_result, Exception):
        updates["errors"] = [{"agent": "reviewer", "error": str(reviewer_result)}]
    else:
        updates["review_feedback"] = reviewer_result
    
    return updates
```

**장점:** 외부 의존성 없음, 라이선스 불필요, 코드 단순
**단점:** 태스크 ID 관리, 중간 취소, 진행 상황 확인은 수동 구현 필요. Supervisor가 결과를 기다리는 동안 사용자 인터랙션을 유지하는 "진정한 비동기"는 아님

---

### A.5 방법 비교 요약

```mermaid
graph TB
    Question["비동기 병렬 에이전트가 필요한가?"]

    Question -->|"예"| LongTask["장시간 태스크 중\n사용자 인터랙션 유지 필요?"]
    Question -->|"단순 병렬만 충분"| Method4["방법 4\npure asyncio.gather\n외부 의존성 없음"]

    LongTask -->|"예 — 진정한 비동기 필요"| Scale["독립적 스케일 아웃 필요?"]
    LongTask -->|"아니오"| Method4

    Scale -->|"예 — 서브에이전트별 독립 스케일"| Vendor["벤더 종속 허용?"]
    Scale -->|"아니오 — 단일 서버로 충분"| Method3["방법 3\nASGI 트랜스포트\n(url 생략 + langgraph.json)"]

    Vendor -->|"예 — 편의성 우선"| Method1["방법 1 또는 LangSmith\nLangGraph Standalone\n또는 관리형 Deployment"]
    Vendor -->|"아니오 — 완전 자체 호스팅"| AirGap["에어갭/보안 환경?"]

    AirGap -->|"예"| Method2a["방법 2\nAegra\n(LangSmith API 키 불필요)"]
    AirGap -->|"아니오"| Method2b["방법 1\nLangGraph Standalone\n(라이선스 검증만 필요)"]
```

| | 방법 1: Standalone | 방법 2: Aegra | 방법 3: ASGI | 방법 4: asyncio |
|---|---|---|---|---|
| LangSmith 불필요 | ⚠️ 키만 필요 | ✅ 완전 불필요 | ✅ 불필요 | ✅ 불필요 |
| 진정한 비동기 (HITL 중 비블로킹) | ✅ | ✅ | ✅ | ❌ |
| 태스크 취소 | ✅ | ✅ | ✅ | ⚠️ 수동 구현 |
| 독립 스케일 아웃 | ✅ | ✅ | ❌ (단일 서버) | ❌ |
| 에어갭 환경 | ⚠️ beacon 제외 | ✅ | ✅ | ✅ |
| 설정 복잡도 | 중간 | 낮음 | 낮음 | 매우 낮음 |
| 공식 지원 여부 | ✅ | ❌ (커뮤니티) | ✅ | ✅ (순수 Python) |
| 프로덕션 안정성 | ✅ 검증됨 | ⚠️ 검증 중 | ✅ | ✅ |

---

### A.6 알려진 버그: recursion_limit 미전파 (GitHub #1698)

비동기 서브에이전트 운영 시 반드시 알아야 할 현재 버그가 있다. `SubAgentMiddleware`가 부모 에이전트의 `recursion_limit` 설정을 서브에이전트 그래프에 전파하지 않는 문제다 (2026년 3월 보고, 미수정).

**증상:** 부모 에이전트를 `recursion_limit=300`으로 설정해도 서브에이전트는 항상 LangGraph 기본값인 `25`로 실행된다. 서브에이전트가 25단계를 초과하면 `GraphRecursionError`가 발생하고, 이것이 `asyncio.CancelledError`로 전파되어 **병렬로 실행 중인 다른 서브에이전트까지 모두 취소**된다.

```python
# ❌ 현재 버그: 부모의 recursion_limit이 서브에이전트에 전달되지 않음
result = await app.ainvoke(
    state,
    config={"recursion_limit": 300},  # 서브에이전트에는 무효
    version="v2"
)

# ⚠️ 임시 회피: asyncio.gather에 return_exceptions=True 적용
# → 하나의 서브에이전트 실패가 다른 서브에이전트를 취소하지 않도록
results = await asyncio.gather(
    analyst_task,
    reviewer_task,
    return_exceptions=True,  # 필수: 하나 실패 시 다른 것 유지
)
```

**권장 조치:**
- `asyncio.gather`에 항상 `return_exceptions=True` 적용
- 장시간 실행되는 서브에이전트 태스크를 작은 단위로 분해하여 25단계 이내에 완료되도록 설계
- LangGraph GitHub 이슈 #1698 추적하여 수정 릴리스 확인

---

### A.7 본문 수정 사항

이 별첨의 내용을 반영하여 **본문 9장의 다음 문장은 아래와 같이 수정되어야 한다**:

| 위치 | 기존 문장 | 수정 문장 |
|---|---|---|
| 9.1장 | "단, 비동기 서브에이전트는 LangSmith Deployment 환경에서만 지원된다." | "비동기 서브에이전트는 Agent Protocol을 구현하는 서버라면 어디서든 실행된다. LangSmith Deployment가 가장 간편한 경로이나, ASGI 트랜스포트(단일 서버), LangGraph Standalone, Aegra 등 자체 호스팅 방식도 완전히 지원된다. 자세한 내용은 별첨 A를 참조." |
| 9.2장 | `async_execution=IS_LANGSMITH_DEPLOYMENT` | `async_execution=True` (서버만 구동되면 어디서든 가능) |

---

- *별첨 작성 일자: 2026-05-05*
- *참조 문서: https://docs.langchain.com/oss/python/deepagents/async-subagents*
- *참조 이슈: https://github.com/langchain-ai/deepagents/issues/1698*
- *참조 프로젝트: https://github.com/ibbybuilds/aegra*
