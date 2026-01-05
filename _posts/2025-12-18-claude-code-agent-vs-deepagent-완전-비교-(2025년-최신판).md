---
title: "Claude Code Agent vs DeepAgent 완전 비교 (2025년 최신판)"
date: 2025-12-18 14:00:00 +0900
categories: [AI,  Platform & Framework]
mermaid: [True]
tags: [AI,  engineering-platform,   development-framework,  coding-agents,  claude-code-agent,  deep-agents,  claude-code,  LangChain,  LangGraph,  LangSmith,  agent-builder,  Claude.write]
---


## 서론

AI 에이전트 개발 생태계는 2025년 들어 급격한 진화를 겪고 있습니다. 두 가지 주목할 만한 접근 방식이 등장했는데, 하나는 Anthropic이 직접 개발한 상용 제품인 Claude Code이고, 다른 하나는 LangChain 팀이 만든 오픈소스 프레임워크인 DeepAgents입니다. 흥미로운 점은 DeepAgents가 Claude Code의 성공적인 아키텍처 패턴을 일반화하여 만들어졌다는 것입니다. 

이 문서는 2025년 12월 현재 최신 정보를 바탕으로 두 시스템을 심층적으로 비교하며, LangChain 1.1, LangGraph 1.0, LangSmith Agent Builder 같은 최신 발전사항을 포함하여 각각의 장단점과 적합한 사용 사례를 명확히 제시합니다.

![claude_code_deepagents_architecture_comparison](https://raw.githubusercontent.com/k82022603/k82022603.github.io/ea799a40660db9bf57418cde1661157708d21c85/assets/img/claude_code_deepagents_architecture_comparison.svg){: style="width:1200px; height:auto;" }


## 2025년 주요 변화사항

### LangChain 생태계의 1.0 릴리스

2025년 9월, LangChain과 LangGraph는 알파 버전을 거쳐 10월에 정식 1.0 릴리스를 발표했습니다. 이는 프로덕션 환경에서 안정적인 에이전트 시스템을 구축할 수 있는 중요한 이정표가 되었습니다.

**LangChain 1.1의 핵심 개선사항:**
- 에이전트 개발의 신뢰성, 구조화, 컨텍스트 인식 강화
- 표준화된 메시지 콘텐츠 블록: 추론(reasoning), 인용(citations), 서버 사이드 도구 호출 등 최신 LLM 기능을 제공업체에 관계없이 표준화
- 모델 프로파일: 각 채팅 모델이 `.profile` 속성을 통해 지원 기능과 역량을 노출
- 향상된 `create_agent` 구현: LangGraph 위에 구축되어 내장 에이전트 런타임 활용
- 레거시 지원: 기존 체인과 에이전트는 `langchain-legacy` 패키지로 계속 사용 가능

**LangGraph 1.0의 핵심 기능:**
- 내장 에이전트 런타임: 영구 실행(durable execution), 단기 메모리, Human-in-the-Loop 패턴, 스트리밍
- 무중단 업그레이드: 기존 사용자에게 호환성 유지
- 프로덕션 검증: Uber, LinkedIn, Klarna 등 대기업에서 이미 사용 중

**LangGraph 2025년 주요 업데이트:**
- 노드 레벨 캐싱 (6월): 개별 노드의 결과를 캐시하여 중복 계산 방지 및 실행 속도 향상
- 지연 노드(Deferred Nodes): 모든 업스트림 경로가 완료될 때까지 실행을 연기하는 노드, 맵-리듀스, 합의, 에이전트 협업 워크플로우에 이상적
- Pre/Post 모델 훅: 모델 호출 전후에 커스텀 로직 추가 가능
- 향상된 JavaScript 지원: 완전한 타입 안전성, `.stream()` 메서드의 streamMode에 따른 타입 추론
- React 통합: 단일 훅으로 LangGraph를 React 애플리케이션에 통합, 스레드 및 상태 관리, 메시지 스트리밍, 브랜칭 내장
- LangMem SDK (2월): 에이전트가 장기 메모리를 통해 학습하고 개선하도록 돕는 경량 Python 라이브러리
- LangGraph Supervisor (2월): 계층적 멀티 에이전트 시스템 구축을 단순화하는 경량 Python 라이브러리

### LangSmith Agent Builder의 등장

2025년 10월 비공개 프리뷰로 시작하여 12월 2일 공개 베타로 전환된 LangSmith Agent Builder는 코드 없이 에이전트를 구축할 수 있는 노코드 플랫폼입니다. 이는 에이전트 개발의 민주화를 대표하는 중요한 발전입니다.

**주요 특징:**
- 대화형 설정 플로우: 자연어로 원하는 것을 설명하면 시스템이 자동으로 프롬프트 생성, 도구 연결, 트리거 설정
- 진짜 에이전트, 워크플로우 아님: 고정된 경로를 따르는 비주얼 워크플로우 빌더와 달리, 동적으로 추론하고 적응하는 실제 에이전트 생성
- 내장 메모리: 에이전트가 수정사항과 업데이트를 시간이 지나도 기억, 한 번 수정하면 향후 상호작용에서 자동 반영
- MCP 도구 통합: Model Context Protocol을 통해 외부 서비스 연결, Gmail, Slack, LinkedIn, Linear 및 커스텀 MCP 서버 지원
- 멀티 모델 지원: OpenAI와 Anthropic 모델 중 작업에 따라 선택 가능
- 워크스페이스 에이전트: 팀 전체에서 에이전트 탐색, 복사, 커스터마이징 가능
- 프로그래밍 방식 호출: API를 통해 에이전트 호출 가능, 기존 워크플로우 및 시스템에 원활한 통합
- DeepAgents 기반: LangSmith Agent Builder는 DeepAgents 아키텍처 위에 구축됨

### Model Context Protocol (MCP)의 확산

2024년 11월 Anthropic이 발표한 MCP는 2025년 들어 AI 에이전트 생태계의 핵심 표준으로 자리잡았습니다.

**2025년 MCP 주요 발전:**
- OpenAI 공식 채택 (3월): ChatGPT 데스크톱 앱, OpenAI Agents SDK, Responses API에 통합
- Google 지원 발표 (4월): Google DeepMind CEO Demis Hassabis가 Gemini 모델과 인프라에서 MCP 지원 확인
- Linux Foundation 기부 (12월): Anthropic이 MCP를 Agentic AI Foundation (AAIF)에 기부, OpenAI, Google, Microsoft, AWS, Cloudflare, Bloomberg 등이 지원
- Google의 관리형 MCP 서버 출시 (12월): Maps, BigQuery, Compute Engine, Kubernetes Engine을 위한 완전 관리형 원격 MCP 서버 제공
- DeepAgents의 MCP 통합: `langchain-mcp-adapters`를 통해 MCP 도구를 쉽게 사용할 수 있도록 지원

**보안 고려사항:**
2025년 4월, 보안 연구자들이 MCP의 여러 보안 이슈를 발견했습니다:
- 프롬프트 인젝션 취약점
- 도구 조합을 통한 파일 유출 가능성
- 유사 도구가 신뢰할 수 있는 도구를 조용히 대체할 수 있는 문제

이러한 문제들은 현재 커뮤니티에서 적극적으로 다루어지고 있으며, 프로덕션 배포 시 반드시 고려해야 합니다.

### DeepAgents의 진화

**DeepAgents 0.2 (10월 2025):**
- 플러그인 방식의 Backend 시스템 도입
- Remote Sandbox 지원: 복잡한 에이전트 코드를 안전하고 격리된 재현 가능한 환경에서 실행

**DeepAgents 0.3 (12월 2025 - 현재 개발 중):**
- MCP 통합 강화
- JavaScript 지원 추가
- 더 많은 미들웨어 옵션

**LangChain Sandbox (6월 2025):**
- DeepAgents와 함께 사용할 수 있는 안전한 Python 실행 환경
- Pyodide (WebAssembly의 Python) 기반
- AI 에이전트에서 신뢰할 수 없는 Python 코드를 안전하게 실행

## 핵심 철학의 차이

### Claude Code: 단순성과 사용자 경험

Claude Code의 설계 철학은 "architectural simplicity at every juncture"로 요약됩니다. 복잡한 멀티 에이전트 시스템이 유행임에도 불구하고, Claude Code는 의도적으로 단일 메인 스레드를 유지합니다. 이는 디버깅을 용이하게 하고, 시스템을 예측 가능하게 만들며, 사용자에게 통제감을 제공합니다.

Anthropic 엔지니어들은 Claude Code를 개발하면서 "프로그래머가 매일 사용하는 도구를 Claude에게도 제공해야 한다"는 원칙을 따랐습니다. 즉, Claude가 코드를 작성하려면 파일을 찾고, 작성하고, 편집하고, 린트하고, 실행하고, 디버깅하고, 때로는 성공할 때까지 이러한 작업을 반복적으로 수행할 수 있어야 합니다. 이를 위해 Claude에게 터미널을 통한 컴퓨터 접근 권한을 제공했고, 이것이 Claude Code를 코딩 도구뿐만 아니라 범용 에이전트로 만드는 핵심이 되었습니다.

Claude Code의 또 다른 중요한 특징은 사용자 경험에 대한 집착입니다. 단순히 기능적으로 작동하는 것을 넘어서, 사용자가 "happy"하고 "delightful"하게 느끼도록 만드는 것을 목표로 합니다. 충분한 자율성을 제공하면서도 통제권을 잃는 불안감을 주지 않는 절묘한 균형을 추구합니다.

### DeepAgents: 일반화와 확장성

DeepAgents는 Claude Code, Deep Research, Manus 같은 성공적인 에이전트들의 공통 패턴을 추출하여 일반화한 프레임워크입니다. LangChain 팀은 이러한 애플리케이션들이 "얕은(shallow)" 에이전트의 한계를 극복하기 위해 네 가지 핵심 요소를 구현했다는 것을 발견했습니다: 계획 도구(planning tool), 서브에이전트(sub-agents), 파일시스템 접근(file system access), 그리고 상세한 프롬프트(detailed prompts)입니다.

DeepAgents의 목표는 이러한 요소들을 범용적으로 구현하여, 개발자가 자신의 도메인에 맞는 Deep Agent를 쉽게 만들 수 있도록 하는 것입니다. 초기에는 Claude Code를 얼마나 범용적으로 만들 수 있는지에 대한 실험으로 시작했지만, 이제는 독립적인 프레임워크로 발전했습니다.

DeepAgents는 처음부터 오픈소스로 설계되었으며, 커뮤니티가 기여하고 확장할 수 있도록 모듈화되어 있습니다. 플러그인 방식의 Backend 시스템, 미들웨어 스택, 다양한 LLM 제공업체 지원 등은 모두 확장성을 염두에 둔 설계입니다.

**2025년 주요 통계:**
연구에 따르면 에이전트 작업 길이가 7개월마다 두 배로 증가하고 있습니다. 이러한 장기 작업은 수십 번의 도구 호출을 포함하여 비용과 신뢰성 문제를 야기합니다. DeepAgents는 이러한 문제를 해결하기 위해 설계된 에이전트 하네스입니다.

## 아키텍처 비교

### Claude Code의 아키텍처

Claude Code는 놀라울 정도로 단순한 아키텍처를 가지고 있습니다. 핵심은 하나의 메인 루프와 하나의 메시지 히스토리입니다. 멀티 에이전트 시스템 대신, Claude Code는 "최대 하나의 브랜치" 접근 방식을 사용합니다. 메인 에이전트가 복잡한 하위 작업을 만나면, 서브에이전트를 생성할 수 없는 자기 자신의 클론을 최대 하나만 생성합니다. 이 서브에이전트의 결과는 메인 메시지 히스토리에 도구 응답으로 추가됩니다.

이러한 단순한 구조는 의도적인 선택입니다. 문제가 충분히 간단하면 메인 루프가 반복적인 도구 호출을 통해 처리합니다. 하지만 하나 이상의 복잡한 작업이 있으면 메인 에이전트가 자신의 클론을 만듭니다. TODO 리스트와 최대 하나의 브랜치 조합은 에이전트가 문제를 하위 문제로 분해할 수 있으면서도 최종 목표를 계속 주시하도록 보장합니다.

Claude Code는 시스템 프롬프트에만 약 2,800 토큰을, 도구 설명에 9,400 토큰을 사용합니다. 휴리스틱, 예제, 그리고 "IMPORTANT" 리마인더로 가득 차 있는 매우 상세한 프롬프트를 사용합니다. 사용자 프롬프트에는 항상 CLAUDE.md 파일이 포함되며, 이는 일반적으로 1,000-2,000 토큰을 차지합니다.

흥미로운 점은 Claude Code가 모든 LLM 호출의 50% 이상을 Claude 3.5 Haiku(더 작은 모델)로 수행한다는 것입니다. 큰 파일 읽기, 웹 페이지 파싱, Git 히스토리 처리, 긴 대화 요약 등에 사용됩니다. 심지어 키 입력마다 표시되는 한 단어 처리 레이블도 Haiku로 생성됩니다. 작은 모델은 표준 모델(Sonnet 4, GPT-4.1)보다 70-80% 저렴하므로, Claude Code는 이를 자유롭게 활용합니다.

### DeepAgents의 아키텍처

DeepAgents는 LangGraph 위에 구축된 더 구조화된 아키텍처를 가지고 있습니다. 핵심 함수인 `create_deep_agent()`는 여러 구성 요소를 조립하여 컴파일된 LangGraph StateGraph를 반환합니다.

**기본 미들웨어 스택:**
1. FilesystemMiddleware: 파일 시스템 도구(ls, read_file, write_file, edit_file) 제공
2. SubagentsMiddleware: 서브에이전트 생성 및 관리
3. TodoMiddleware: TODO 리스트 계획 도구
4. 커스텀 미들웨어: 사용자가 추가한 미들웨어가 기본 미들웨어 뒤에 추가됨

**Backend 시스템:**
DeepAgents 0.2에서 도입된 핵심 혁신입니다. 네 가지 구현을 제공합니다:
- **StateBackend** (기본값): 임시 메모리에 파일 저장, 셸 실행 불가
- **FilesystemBackend**: 실제 디스크 사용, 셸 실행 가능
- **StoreBackend**: LangGraph Store를 통해 세션 간 영구 저장
- **CompositeBackend**: 경로별로 다른 Backend 사용 가능, 하이브리드 접근 가능

DeepAgents는 기본적으로 Claude Sonnet 4.5 모델을 사용하지만, 어떤 LangChain 모델 객체도 사용할 수 있습니다. 이는 OpenAI, Google, Anthropic 등 다양한 제공업체 간 전환을 쉽게 만듭니다.

**모노레포 구조:**
- **deepagents** (Core): 프로그래밍 방식 에이전트 생성
- **deepagents-cli**: 터미널 상호작용 도구
- **deepagents-harbor**: 벤치마크 평가 통합

## 주요 기능 비교

### 계획 및 작업 분해

**Claude Code**는 TODO 리스트를 핵심 계획 메커니즘으로 사용합니다. 에이전트는 작업을 시작하기 전에 TODO 항목을 작성하고, 진행하면서 업데이트하며, 완료되면 체크합니다. 이는 사용자에게 진행 상황을 시각적으로 보여주고, 에이전트가 최종 목표를 잃지 않도록 도와줍니다. TODO 리스트는 단순하지만 효과적이며, 사용자가 에이전트의 계획을 쉽게 이해할 수 있게 합니다.

**DeepAgents**도 동일한 write_todos 도구를 제공합니다. 이는 복잡한 작업을 개별 단계로 분해하고, 진행 상황을 추적하며, 새로운 정보가 나타나면 계획을 조정할 수 있게 합니다. DeepAgents의 TODO 시스템은 Claude Code에서 영감을 받았지만, 더 일반화되어 다양한 도메인에서 사용할 수 있습니다.

### 파일시스템 접근

**Claude Code**는 사용자의 로컬 파일시스템에 직접 접근합니다. 터미널을 통해 bash 명령을 실행할 수 있으므로, 파일 읽기/쓰기뿐만 아니라 grep, tail, find 같은 유닉스 도구를 활용할 수 있습니다. 큰 파일을 다룰 때는 전체를 컨텍스트에 로드하지 않고, 필요한 부분만 추출하는 스크립트를 작성합니다.

**DeepAgents**는 플러그인 방식의 Backend 추상화를 통해 더 유연한 파일시스템 접근을 제공합니다. StateBackend는 임시 메모리에 파일을 저장하고(셸 실행 불가), FilesystemBackend는 실제 디스크를 사용하며(셸 실행 가능), StoreBackend는 LangGraph Store를 통해 세션 간 영구 저장을 제공합니다. CompositeBackend를 사용하면 경로별로 다른 Backend를 사용할 수 있어, 작업 파일은 로컬에, 메모리는 영구 저장소에 보관하는 하이브리드 접근이 가능합니다.

DeepAgents의 파일시스템 도구는 자동 컨텍스트 관리를 수행하여, 큰 파일을 만나면 자동으로 파일시스템에 오프로드하여 컨텍스트 윈도우 오버플로우를 방지합니다.

### 서브에이전트

**Claude Code**의 서브에이전트는 매우 제한적입니다. 메인 에이전트만 서브에이전트를 생성할 수 있으며, 서브에이전트는 다시 다른 서브에이전트를 생성할 수 없습니다. 즉, 최대 깊이가 2입니다. 서브에이전트는 메인 에이전트의 클론이지만, 서브에이전트 생성 능력이 제거됩니다. 결과는 메인 메시지 히스토리에 도구 응답으로 추가됩니다.

이러한 제약은 의도적입니다. 복잡도를 제한하여 디버깅을 쉽게 하고, 무한 재귀를 방지하며, 사용자가 무슨 일이 일어나고 있는지 이해할 수 있게 합니다.

**DeepAgents**는 더 유연한 서브에이전트 시스템을 제공합니다. 서브에이전트는 고유한 이름, 설명, 프롬프트, 도구, 그리고 심지어 다른 모델을 가질 수 있습니다. 예를 들어, 메인 에이전트는 Claude를 사용하고, 특정 서브에이전트는 GPT-4o를 사용하도록 설정할 수 있습니다.

서브에이전트는 두 가지 방식으로 정의할 수 있습니다:
1. **간단한 딕셔너리**: name, description, prompt, tools, model을 지정
2. **CompiledSubAgent**: 사전에 구축된 LangGraph 그래프를 전달

DeepAgents는 task 도구를 통해 서브에이전트를 호출합니다. 이는 작업 격리와 전문화를 가능하게 하여, 복잡한 작업을 여러 전문가에게 위임할 수 있습니다.

### 프롬프트 및 지시사항

**Claude Code**는 매우 상세하고 규범적인 시스템 프롬프트를 사용합니다. 2,800 토큰의 시스템 프롬프트와 9,400 토큰의 도구 설명은 에이전트가 어떻게 행동해야 하는지에 대한 명확한 지침을 제공합니다. 또한 CLAUDE.md 파일을 통해 프로젝트별 컨텍스트와 사용자 선호도를 유지합니다.

CLAUDE.md는 협업 문서로 기능합니다. Claude는 프로젝트를 이해하면서 이 파일을 업데이트하고, 사용자도 선호도를 추가할 수 있습니다. 이는 대화 간 메모리 역할을 하여, Claude가 프로젝트의 컨텍스트를 잃지 않도록 합니다.

**DeepAgents**는 Claude Code에서 영감을 받은 BASE_AGENT_PROMPT를 제공합니다. 기본 프롬프트는 "In order to complete the objective that the user asks of you, you have access to a number of standard tools"로 시작하여, 내장 도구 사용법을 설명합니다.

개발자는 system_prompt 파라미터를 통해 커스텀 지시사항을 추가할 수 있으며, 이는 기본 프롬프트에 삽입됩니다. DeepAgents는 프롬프트를 더 범용적으로 만들기 위해 Claude Code의 프롬프트를 수정했지만, 핵심 구조는 유지합니다.

### Agent Skills

**Claude Code**는 2025년 10월에 Agent Skills를 도입했습니다. Skills는 조직화된 폴더로, 지시사항, 스크립트, 리소스를 포함하며, 에이전트가 동적으로 발견하고 로드하여 특정 작업을 더 잘 수행할 수 있게 합니다.

Skills의 핵심 설계 원칙은 "progressive disclosure"입니다. 잘 조직된 매뉴얼처럼 목차로 시작하여, 필요할 때만 특정 챕터와 상세한 부록을 로드합니다. 에이전트는 SKILL.md 메타데이터를 먼저 읽고, 필요하면 추가 파일을 로드합니다. 이는 스킬에 포함될 수 있는 컨텍스트 양이 사실상 무제한임을 의미합니다.

**DeepAgents**는 명시적인 Skills 시스템을 내장하고 있지 않지만, 동일한 패턴을 구현할 수 있습니다. FilesystemMiddleware를 통해 파일을 읽을 수 있고, 커스텀 미들웨어를 추가하여 스킬 발견 및 로드 로직을 구현할 수 있습니다. DeepAgents-cli 패키지는 스킬 관리 기능을 제공합니다.

### Model Context Protocol (MCP) 통합

**Claude Code**는 Anthropic의 제품이므로 MCP와 자연스럽게 통합됩니다. Claude Desktop 앱은 로컬 MCP 서버를 지원하며, 사용자는 다양한 MCP 서버를 연결하여 에이전트의 기능을 확장할 수 있습니다.

**DeepAgents**는 `langchain-mcp-adapters` 패키지를 통해 MCP를 지원합니다:

```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from deepagents import create_deep_agent

async def main():
    mcp_client = MultiServerMCPClient(...)
    mcp_tools = await mcp_client.get_tools()
    
    agent = create_deep_agent(tools=mcp_tools)
    
    async for chunk in agent.astream({
        "messages": [{"role": "user", "content": "..."}]
    }):
        chunk["messages"][-1].pretty_print()
```

이를 통해 DeepAgents는 MCP 서버가 제공하는 모든 도구에 접근할 수 있으며, Google Drive, Slack, GitHub, Postgres 등 다양한 외부 시스템과 통합할 수 있습니다.

**2025년 12월 업데이트:**
Google이 Maps, BigQuery, Compute Engine, Kubernetes Engine을 위한 완전 관리형 MCP 서버를 출시하면서, DeepAgents 사용자는 이러한 서비스를 URL 하나로 쉽게 통합할 수 있게 되었습니다. 이는 개발자가 일주일 이상 걸리던 커넥터 설정을 몇 분으로 단축시켰습니다.

## 사용성 및 개발자 경험

### Claude Code

Claude Code는 상용 제품으로서 매끄러운 사용자 경험을 제공합니다. CLI 인터페이스는 직관적이며, 에이전트의 진행 상황을 실시간으로 보여주는 멋진 UI 요소들이 있습니다. 키 입력마다 업데이트되는 처리 레이블(예: "thinking...", "editing...", "running...")은 Haiku로 생성되며, 사용자에게 피드백을 제공합니다.

설치는 간단합니다. Anthropic 계정이 있으면 Claude Code를 다운로드하고 즉시 사용할 수 있습니다. 설정이 거의 필요 없으며, 대부분의 경우 바로 작동합니다.

하지만 Claude Code는 폐쇄 소스입니다. 내부 작동 방식을 수정할 수 없으며, Anthropic의 API와 모델에 종속됩니다. 비용은 사용량에 따라 다르지만, Claude API 가격 정책을 따릅니다.

### DeepAgents

DeepAgents는 오픈소스 프레임워크로서 완전한 투명성과 커스터마이징을 제공합니다. 소스 코드를 읽고, 수정하고, 기여할 수 있습니다. GitHub에서 이슈를 제출하고, PR을 보내고, 커뮤니티와 협력할 수 있습니다.

설치는 표준 Python 패키지 관리자를 통해 이루어집니다:
```bash
pip install deepagents
```

하지만 프레임워크이기 때문에 더 많은 설정이 필요합니다. 에이전트를 생성하고, 도구를 정의하고, Backend를 선택하고, 프롬프트를 작성해야 합니다. 즉시 사용 가능한 제품이 아니라 빌딩 블록입니다.

DeepAgents-cli 패키지는 더 나은 개발자 경험을 제공합니다. 터미널에서 대화형으로 에이전트와 상호작용할 수 있으며, 스킬 관리와 메모리 관리 기능을 제공합니다.

**2025년 최신 개발자 도구:**
- **LangGraph Studio**: 에이전트를 시각적으로 디버그하고 정제할 수 있는 IDE
- **LangSmith 통합**: LangGraph Studio에서 LLM 실행을 LangSmith Playground에서 직접 열 수 있음
- **비용 추적**: LangSmith가 에이전트 애플리케이션의 리소스 사용량을 동적으로 추적
- **멀티턴 평가**: 전체 엔드투엔드 상호작용에서 에이전트 성능을 측정

## LangSmith Agent Builder vs DeepAgents vs Claude Code

2025년 12월 현재, 세 가지 주요 옵션이 있습니다:

### LangSmith Agent Builder
**적합한 경우:**
- 비개발자 또는 비즈니스 사용자
- 빠른 프로토타이핑이 필요한 개발자
- 내부 생산성 에이전트 (이메일, Slack, Salesforce 어시스턴트)
- 노코드로 에이전트 구축하고 즉시 배포

**장점:**
- 코드 작성 없이 대화형으로 에이전트 생성
- 내장 메모리와 Human-in-the-Loop
- MCP 통합 및 워크스페이스 협업
- DeepAgents 아키텍처의 장점 활용

**단점:**
- 저수준 제어 제한적
- 커스텀 로직 구현에 제약
- 현재 베타 단계

### DeepAgents
**적합한 경우:**
- 프로덕션 에이전트 시스템 구축하는 개발자
- 완전한 커스터마이징이 필요한 경우
- 특정 도메인 에이전트 (금융, 의료, 법률 등)
- 오픈소스와 벤더 중립성이 중요한 경우

**장점:**
- 완전한 코드 제어와 투명성
- 다양한 LLM과 Backend 옵션
- LangGraph의 모든 기능 활용
- 활발한 커뮤니티와 빠른 발전

**단점:**
- 더 많은 설정과 코드 작성 필요
- 학습 곡선이 있음
- 직접 유지보수 필요

### Claude Code
**적합한 경우:**
- 코딩 작업이 주요 사용 사례
- 프리미엄 모델 (Claude 4)이 필요한 경우
- 즉시 사용 가능한 솔루션 원하는 개발자
- 최고 수준의 UX가 중요한 경우

**장점:**
- 설치 후 바로 사용 가능
- 최적화된 코딩 경험
- 최신 Claude 모델 사용
- 매끄러운 UI/UX

**단점:**
- 폐쇄 소스, 커스터마이징 제한적
- Anthropic API에 종속
- 코딩 외 작업에는 덜 적합

## 성능 및 효율성

### 토큰 사용

**Claude Code**는 토큰 효율성에 매우 신경을 씁니다. 작은 모델을 자유롭게 사용하여 비용을 절감하고, 큰 파일을 다룰 때는 전체를 로드하지 않고 필요한 부분만 추출합니다. Git 히스토리 요약, 메시지 히스토리 압축 등의 주기적인 최적화도 수행합니다.

프롬프트 캐싱을 활용하여 반복되는 시스템 프롬프트와 도구 설명을 재사용합니다. 이는 특히 긴 대화에서 토큰 사용을 크게 줄입니다.

**DeepAgents**도 비슷한 최적화를 제공합니다. 컨텍스트 요약과 파일시스템 오프로드를 통해 컨텍스트 윈도우 오버플로우를 방지합니다. LangChain 1.1의 모델 프로파일을 활용하여 컨텍스트 인식 요약의 트리거 포인트를 유연하게 설정할 수 있습니다.

**2025년 최적화 기능:**
- **노드 레벨 캐싱** (LangGraph): 개별 노드의 결과를 캐시하여 중복 계산 방지
- **지연 노드**: 병렬 브랜치가 모두 완료될 때까지 실행 연기, 효율적인 맵-리듀스 가능
- **LangChain Sandbox**: WebAssembly를 사용하여 안전하게 Python 코드 실행, 오버헤드 최소화

### 실행 속도

**Claude Code**는 최적화된 상용 제품으로서 빠른 응답 시간을 제공합니다. Haiku를 많이 사용하여 간단한 작업을 빠르게 처리하고, 복잡한 작업만 Sonnet이나 Opus로 처리합니다.

**DeepAgents**의 속도는 사용하는 모델과 Backend에 따라 다릅니다. StateBackend는 가장 빠르고, FilesystemBackend는 디스크 I/O로 인해 약간 느릴 수 있으며, StoreBackend는 네트워크 지연이 있을 수 있습니다. 하지만 노드 레벨 캐싱과 적절한 설정으로 Claude Code와 비슷한 성능을 얻을 수 있습니다.

**LangSmith Agent Builder**는 DeepAgents 위에 구축되어 있으므로 기본적으로 유사한 성능을 제공하지만, 노코드 인터페이스의 추가 레이어로 인한 약간의 오버헤드가 있을 수 있습니다.

## 적합한 사용 사례

### Claude Code가 적합한 경우

1. **빠른 시작이 필요한 경우**: 설정 없이 바로 사용하고 싶다면 Claude Code가 최선입니다. 다운로드하고 몇 분 안에 코딩을 시작할 수 있습니다.

2. **코딩이 주요 작업인 경우**: Claude Code는 코딩에 최적화되어 있습니다. 코드 마이그레이션, 버그 수정, 리팩토링 등의 작업에 탁월합니다.

3. **프리미엄 모델이 필요한 경우**: Claude Code는 최신 Claude 모델(Sonnet 4.5, Opus 4.5)을 사용합니다. 이는 다른 모델보다 코딩과 추론에서 우수한 성능을 보입니다.

4. **사용자 경험이 중요한 경우**: 매끄러운 UI/UX, 실시간 피드백, 직관적인 인터페이스가 중요하다면 Claude Code가 낫습니다.

5. **내부 수정이 필요 없는 경우**: 에이전트의 내부 작동 방식을 변경할 필요가 없고, 기본 동작으로 충분하다면 Claude Code로 충분합니다.

### DeepAgents가 적합한 경우

1. **커스터마이징이 필요한 경우**: 에이전트의 모든 측면을 제어하고 싶다면 DeepAgents를 사용하세요. 프롬프트, 도구, Backend, 미들웨어 등을 자유롭게 수정할 수 있습니다.

2. **특정 도메인 에이전트를 구축하는 경우**: 금융, 의료, 법률 등 특정 도메인에 특화된 에이전트를 만들 때 DeepAgents의 유연성이 빛을 발합니다.

3. **벤더 종속을 피하고 싶은 경우**: 여러 LLM 제공업체를 사용하거나 전환할 수 있는 유연성이 필요하면 DeepAgents가 적합합니다.

4. **오픈소스가 중요한 경우**: 투명성, 커뮤니티 기여, 코드 검증이 중요한 프로젝트라면 오픈소스인 DeepAgents를 선택하세요.

5. **복잡한 장기 작업이 필요한 경우**: 수십 개의 도구 호출, 여러 서브에이전트, 복잡한 워크플로우가 필요한 경우 DeepAgents의 구조화된 접근이 유리합니다.

6. **비용 최적화가 중요한 경우**: 더 저렴한 모델을 사용하거나, 자체 호스팅 모델을 사용하거나, Backend를 최적화하여 비용을 절감할 수 있습니다.

7. **연구 및 실험이 목적인 경우**: 에이전트 아키텍처를 연구하거나 새로운 패턴을 실험하려면 오픈소스인 DeepAgents가 훨씬 유리합니다.

8. **MCP 생태계 활용이 중요한 경우**: 2025년 들어 MCP가 산업 표준으로 자리잡으면서, DeepAgents의 MCP 통합은 Google, OpenAI, Microsoft 등 주요 제공업체의 서비스와 원활한 통합을 가능하게 합니다.

### LangSmith Agent Builder가 적합한 경우

1. **비개발자가 에이전트를 만들어야 하는 경우**: 코드를 작성하지 않고도 대화형으로 에이전트를 구축할 수 있습니다.

2. **빠른 프로토타이핑**: 개발자도 아이디어를 빠르게 검증하기 위해 Agent Builder로 프로토타입을 만들고, 나중에 DeepAgents나 LangGraph로 마이그레이션할 수 있습니다.

3. **내부 생산성 도구**: 이메일 트리지, Slack 어시스턴트, Salesforce 헬퍼 등 내부 팀 생산성 향상 에이전트에 이상적입니다.

4. **팀 협업이 중요한 경우**: 워크스페이스 기능을 통해 팀원들이 에이전트를 공유하고 복사하여 커스터마이징할 수 있습니다.

5. **승인 워크플로우가 필요한 경우**: 내장된 Human-in-the-Loop로 중요한 작업은 승인을 받을 수 있습니다.

## 통합 및 확장성

### Claude Code

Claude Code는 Anthropic 생태계와 긴밀하게 통합되어 있습니다. Claude API, LangSmith(모니터링), Claude.ai(웹 인터페이스) 등과 자연스럽게 연동됩니다.

확장성은 Agent Skills를 통해 제공됩니다. 커뮤니티가 만든 수십 개의 스킬이 있으며, 자신만의 스킬을 만들 수 있습니다. 하지만 핵심 에이전트 로직은 수정할 수 없습니다.

### DeepAgents

DeepAgents는 LangChain 생태계의 일부로서 광범위한 통합을 제공합니다:
- LangChain의 수천 개 통합(OpenAI, Google, Anthropic, Hugging Face 등)
- LangGraph의 저수준 제어와 워크플로우
- LangSmith의 트레이싱, 평가, 배포
- MCP (Model Context Protocol) 서버 통합
- 다양한 벡터 스토어, 데이터베이스, API

**2025년 새로운 통합:**
- **Google의 관리형 MCP 서버**: Maps, BigQuery, Compute Engine, Kubernetes Engine
- **LangGraph Supervisor**: 계층적 멀티 에이전트 시스템
- **LangMem SDK**: 장기 메모리 통합
- **React 훅**: 단일 훅으로 React 앱에 통합

확장성은 모듈화된 설계를 통해 제공됩니다. 커스텀 미들웨어, Backend, 서브에이전트, 도구를 추가할 수 있으며, 심지어 create_deep_agent 함수 자체도 수정할 수 있습니다.

### LangSmith Agent Builder

LangSmith Agent Builder는 DeepAgents 위에 구축되어 있으므로 동일한 통합 옵션을 제공하지만, 노코드 인터페이스를 통해 접근합니다:
- MCP 서버를 통한 외부 서비스 연결
- Gmail, Slack, LinkedIn, Linear 등 사전 구성된 통합
- 프로그래밍 방식 API 호출
- 워크스페이스 레벨 도구 거버넌스

## 비용 고려사항

### Claude Code

Claude Code는 Anthropic API 사용량에 따라 비용이 청구됩니다:
- **Sonnet 4.5**: 입력 1M 토큰당 $3, 출력 1M 토큰당 $15
- **Opus 4.5**: 더 비싸지만 더 강력

프롬프트 캐싱을 사용하면 비용을 크게 절감할 수 있습니다. 또한 Haiku를 많이 사용하여 간단한 작업의 비용을 줄입니다.

정확한 비용은 사용 패턴에 따라 다르지만, 일반적으로 시간당 $0.50 ~ $5 범위입니다.

### DeepAgents

DeepAgents는 오픈소스이므로 프레임워크 자체는 무료입니다. 하지만 사용하는 LLM에 따라 비용이 발생합니다:
- Claude API 사용 시: Claude Code와 동일
- OpenAI API 사용 시: GPT-4 가격 정책
- 로컬 모델 사용 시: 인프라 비용만 (Ollama 등)

Backend 선택도 비용에 영향을 줍니다:
- StateBackend: 추가 비용 없음
- FilesystemBackend: 스토리지 비용
- StoreBackend: 데이터베이스 비용

일반적으로 DeepAgents는 더 저렴한 모델을 선택하거나 로컬 모델을 사용할 수 있어 비용 면에서 더 유연합니다.

### LangSmith Agent Builder

LangSmith Agent Builder는 현재 베타 단계로, 기존 LangSmith 사용자에게 무료로 제공됩니다. 정식 출시 후 가격 정책은 아직 발표되지 않았습니다. 에이전트가 사용하는 LLM과 도구에 대한 비용은 별도로 발생합니다.

**비용 추적 기능 (2025년 6월):**
LangSmith는 이제 에이전트 애플리케이션의 비용 추적을 제공합니다. 에이전트가 동적으로 결정하는 리소스 사용량을 추적하므로, 예상치 못한 비용을 방지할 수 있습니다.

## 커뮤니티 및 지원

### Claude Code

Claude Code는 Anthropic의 공식 제품으로 전문적인 지원을 받습니다. 문서는 잘 작성되어 있고, 공식 포럼과 디스코드가 있으며, 버그는 빠르게 수정됩니다.

커뮤니티는 성장하고 있으며, 많은 개발자가 Agent Skills를 공유하고 있습니다. GitHub에는 Claude Code를 위한 다양한 프레임워크와 도구가 있습니다(SuperClaude, BMAD, Claude Flow, Awesome Claude 등).

하지만 코드가 폐쇄 소스이므로 커뮤니티가 내부 로직에 기여할 수 없습니다.

### DeepAgents

DeepAgents는 활발한 오픈소스 커뮤니티를 가지고 있습니다. GitHub에서 이슈를 제출하고, PR을 보내고, 토론에 참여할 수 있습니다. LangChain 팀이 적극적으로 유지 관리하며, 커뮤니티의 피드백을 빠르게 반영합니다.

Python과 JavaScript 버전이 모두 있으며, 세 가지 패키지(Core, CLI, Harbor)가 서로 다른 요구사항을 충족합니다.

LangChain Discord와 포럼에서 도움을 받을 수 있으며, 수많은 튜토리얼과 예제가 커뮤니티에서 공유됩니다.

**2025년 커뮤니티 발전:**
- 수많은 커뮤니티 빌드 MCP 서버
- DeepMCPAgent 같은 서드파티 확장
- AI Alliance의 Deep Research Agent for Finance 같은 오픈소스 프로젝트
- AgentLabUI와 Gofannon 같은 프랙티셔너 워크벤치

### LangSmith Agent Builder

LangSmith Agent Builder는 아직 베타 단계이지만, LangChain의 강력한 커뮤니티와 지원을 받습니다. 공식 문서, 튜토리얼, DataCamp 같은 교육 플랫폼의 가이드가 있습니다.

비공개 프리뷰 기간 동안 수천 명의 사용자가 수천 개의 에이전트를 구축했으며, 이러한 피드백이 제품 개선에 반영되고 있습니다.

## 미래 전망

### Claude Code

Anthropic은 Claude Code를 "Claude Agent SDK"로 리브랜딩하여 범위를 확장하고 있습니다. 코딩뿐만 아니라 딥 리서치, 비디오 생성, 노트 작성 등 다양한 용도로 사용됩니다. Anthropic 내부에서는 거의 모든 주요 에이전트 루프를 Claude Code가 지원합니다.

**예상되는 향후 업데이트:**
- 더 많은 Agent Skills
- 더 나은 멀티모달 지원
- 더 강력한 서브에이전트 시스템
- MCP 생태계와의 더 깊은 통합

### DeepAgents

DeepAgents는 빠르게 발전하고 있습니다:
- **2025년 8월**: 첫 출시
- **2025년 10월**: 0.2 (Backend 시스템, Remote Sandbox)
- **2025년 12월**: 0.3 (MCP 통합 강화, JavaScript 지원)

**향후 로드맵:**
- 더 많은 미들웨어 옵션
- 완전한 시스템 프롬프트 커스터마이징
- 더 나은 평가 도구
- 더 강력한 CLI
- 더 많은 커뮤니티 기여

LangChain 팀은 DeepAgents를 "agent harness"의 표준으로 만들려는 야심찬 계획을 가지고 있습니다.

![ecosystem_3tier_architecture](https://raw.githubusercontent.com/k82022603/k82022603.github.io/ea799a40660db9bf57418cde1661157708d21c85/assets/img/ecosystem_3tier_architecture.svg){: style="width:1200px; height:auto;" }


### LangSmith Agent Builder

LangSmith Agent Builder는 2025년 12월 공개 베타로 전환되었으며, 향후 몇 달 내 정식 출시가 예상됩니다.

**예상되는 발전:**
- 더 많은 사전 구성 통합
- 향상된 워크스페이스 협업 기능
- 더 많은 모델 선택지
- 엔터프라이즈 기능 (거버넌스, 감사, 규정 준수)
- 개발자를 위한 더 나은 API

LangChain 팀의 비전은 비개발자도 에이전트를 쉽게 만들 수 있게 하면서, 기술 팀은 도구 거버넌스와 보안을 관리하는 것입니다.

### MCP 생태계

MCP는 2025년 들어 AI 에이전트의 사실상 표준이 되었습니다:
- OpenAI, Google, Microsoft, Anthropic 모두 지원
- Linux Foundation의 Agentic AI Foundation으로 이전
- 수백 개의 MCP 서버가 커뮤니티에서 개발됨
- Google의 Apigee를 통한 엔터프라이즈 통합

Claude Code와 DeepAgents 모두 MCP를 지원하므로, 사용자는 동일한 MCP 서버를 양쪽에서 사용할 수 있습니다. 이는 두 시스템 간의 상호 운용성을 높이고, 에이전트 생태계의 성숙도를 보여줍니다.

## 실전 비교 시나리오

### 시나리오 1: 코드베이스 리팩토링

**요구사항**: 레거시 Python 프로젝트를 현대적인 구조로 리팩토링

**Claude Code 접근:**
1. 프로젝트 디렉토리에서 `claude-code` 실행
2. "Refactor this codebase to use modern Python practices" 요청
3. Claude가 코드베이스를 인덱싱하고 TODO 리스트 생성
4. 자동으로 파일을 읽고, 수정하고, 린트하고, 테스트 실행
5. 사용자에게 실시간 진행 상황 표시
6. 완료 시 변경사항 요약 제공

**장점**: 빠른 시작, 자동화된 워크플로우, 실시간 피드백
**단점**: 커스터마이징 제한, Claude API에 종속

**DeepAgents 접근:**
1. 리팩토링 에이전트 생성 (커스텀 도구 포함)
2. FilesystemBackend로 실제 파일 접근
3. 특화된 서브에이전트 사용 (코드 분석, 테스트 작성 등)
4. LangSmith로 각 단계 트레이싱
5. 커스텀 평가자로 품질 검증
6. 결과를 영구 저장소에 보관

**장점**: 완전한 제어, 커스텀 워크플로우, 다양한 모델 사용 가능
**단점**: 더 많은 설정 필요, 직접 구현해야 할 부분 많음

**LangSmith Agent Builder 접근:**
1. Agent Builder에서 "코드 리팩토링 에이전트" 생성
2. MCP를 통해 GitHub 또는 로컬 파일 시스템 연결
3. 대화형으로 리팩토링 규칙과 우선순위 설정
4. 에이전트가 변경사항 제안, 사용자 승인 후 적용
5. Google Sheets에 변경사항 로그 자동 기록

**장점**: 코드 없이 빠른 설정, 승인 워크플로우, 팀 공유 가능
**단점**: 복잡한 리팩토링에는 제한적, 베타 단계

### 시나리오 2: 연구 보고서 작성

**요구사항**: 특정 주제에 대한 심층 연구 보고서 작성

**Claude Code 접근:**
1. 연구 스킬 로드 (있는 경우)
2. 웹 검색 도구 사용하여 정보 수집
3. 파일에 노트 작성
4. Markdown 보고서 생성
5. 참고문헌 자동 포맷팅

**장점**: Skills를 통한 확장, 깔끔한 워크플로우
**단점**: 긴 연구에는 컨텍스트 윈도우 제한 가능

**DeepAgents 접근:**
1. 연구 서브에이전트 생성 (Tavily 검색 도구 포함)
2. StoreBackend로 연구 노트 영구 저장
3. 여러 세션에 걸쳐 작업 지속
4. 커스텀 평가자로 보고서 품질 검증
5. 자동 인용 생성 도구 추가
6. MCP를 통해 Google Scholar, ArXiv 등 학술 데이터베이스 연결

**장점**: 세션 간 지속성, 무제한 메모리, 커스텀 평가, MCP 통합
**단점**: 초기 설정 복잡, 더 많은 코드 작성 필요

**LangSmith Agent Builder 접근:**
1. "연구 에이전트" 템플릿 선택
2. 웹 검색, MCP 도구 (Google Scholar 등) 연결
3. 대화형으로 연구 주제와 깊이 설정
4. 내장 메모리로 여러 세션에 걸쳐 연구 진행
5. Google Docs에 자동으로 보고서 작성
6. 중요한 인용 추가 시 승인 요청

**장점**: 노코드, 빠른 설정, 팀원과 공유 가능, 승인 워크플로우
**단점**: 복잡한 학술 형식에는 제한적

### 시나리오 3: 고객 지원 에이전트

**요구사항**: 티켓을 분류하고, 데이터를 조회하며, 사용자에게 응답하는 에이전트

**Claude Code 접근:**
- Claude Code는 대화형 지원 에이전트보다는 터미널 기반 작업에 최적화되어 있어 이 시나리오에는 덜 적합합니다.
- 하지만 Anthropic의 Claude API를 직접 사용하면 훌륭한 고객 지원 에이전트를 만들 수 있습니다.

**DeepAgents 접근:**
1. 티켓 분류 서브에이전트 생성
2. MCP를 통해 CRM, 지식베이스, 데이터베이스 연결
3. StoreBackend로 사용자 컨텍스트 저장
4. Human-in-the-Loop로 에스컬레이션
5. LangSmith로 성능 모니터링
6. 온라인 평가로 품질 추적

**장점**: 완전한 제어, 프로덕션 기능, 세션 간 메모리, MCP 통합
**단점**: 처음부터 구축해야 함

**LangSmith Agent Builder 접근:**
1. "고객 지원 에이전트" 템플릿 선택
2. Slack, Zendesk, Salesforce MCP 서버 연결
3. 대화형으로 응답 스타일과 에스컬레이션 규칙 설정
4. 내장 메모리로 고객 히스토리 추적
5. 중요한 응답 전송 전 승인 요청
6. 워크스페이스에서 팀원과 공유

**장점**: 빠른 배포, 노코드, 팀 협업, 승인 워크플로우
**단점**: 복잡한 비즈니스 로직에는 제한적

### 시나리오 4: 이메일 트리지 어시스턴트 (2025년 신규)

**요구사항**: 새 이메일을 자동으로 분류하고, 답변 초안을 작성하며, Google Sheets에 로그

**LangSmith Agent Builder 접근 (최적):**
1. Agent Builder에서 "이메일 트리지 어시스턴트" 생성
2. Gmail MCP 서버 연결 및 인증
3. 분류 기준 설정: 즉시 답변, 일정 잡기, 위임, 보관
4. 낯선 발신자에 대해 웹 검색으로 컨텍스트 추가
5. Google Sheets MCP 서버 연결
6. 문서 생성/업데이트 전 승인 요청 설정
7. 배포 후 Slack으로 알림 받기

**장점**: 완벽한 MCP 통합, 승인 워크플로우, 노코드, 즉시 배포
**사용 사례**: DataCamp 튜토리얼에서 상세히 소개됨 (2025년 12월)

**DeepAgents 접근:**
1. 이메일 처리 에이전트 생성
2. `langchain-mcp-adapters`로 Gmail, Google Sheets MCP 서버 연결
3. 분류 로직과 웹 검색 통합
4. StoreBackend로 이메일 히스토리 저장
5. 커스텀 미들웨어로 승인 워크플로우 구현

**장점**: 완전한 커스터마이징, 복잡한 비즈니스 로직 가능
**단점**: 더 많은 개발 시간 필요

## 생태계 비교: 대안 및 경쟁자

2025년 현재, AI 에이전트 플랫폼 생태계는 빠르게 발전하고 있습니다. Claude Code와 DeepAgents 외에도 여러 대안이 있습니다:

### 주요 경쟁 플랫폼

**Google Agent Development Kit (ADK):**
- Google 내부 제품(Agentspace, Customer Engagement Suite)에서 사용
- 멀티 에이전트 시스템에 중점
- Agent2Agent (A2A) 프로토콜 주도
- Vertex AI Agent Builder와 통합

**Microsoft Copilot Studio:**
- Azure AI Foundry와 통합
- A2A 프로토콜 지원 예정
- 엔터프라이즈 중심

**Amazon Bedrock Multi-Agent Systems (MAS):**
- 오픈소스 프레임워크 통합
- AWS 서비스와 긴밀한 통합

**기타 오픈소스 대안:**
- **Atomic Agents**: 경량 에이전트 프레임워크
- **MCP-Agent**: MCP 중심 에이전트 빌더
- **DeepMCPAgent**: DeepAgents와 MCP 통합에 특화

### 프로토콜 표준화

2025년의 중요한 발전은 프로토콜 표준화입니다:

**MCP (Model Context Protocol):**
- Anthropic 주도, 모델-컨텍스트 통합에 중점
- OpenAI, Google, Microsoft 모두 채택
- Linux Foundation으로 이전

**A2A (Agent2Agent):**
- Google 주도, 에이전트 간 통신에 중점
- JSON-RPC 2.0 over HTTP(S), SSE 스트리밍
- Microsoft도 Azure AI Foundry와 Copilot Studio에 통합 예정

**업계 관계:**
"에이전트 애플리케이션은 A2A와 MCP 둘 다 필요합니다. MCP는 도구를 위해, A2A는 에이전트를 위해."

이는 Claude Code와 DeepAgents가 단순히 독립적인 솔루션이 아니라, 더 큰 에이전트 생태계의 일부임을 보여줍니다. 두 시스템 모두 MCP를 지원하며, 향후 A2A 통합도 예상됩니다.

## 결론

Claude Code와 DeepAgents는 서로 다른 목적을 가진 도구입니다. Claude Code는 즉시 사용 가능한 프리미엄 제품으로, 특히 코딩 작업에서 탁월한 사용자 경험을 제공합니다. DeepAgents는 유연한 오픈소스 프레임워크로, 커스터마이징과 확장성이 중요한 경우에 적합합니다. 그리고 2025년 12월 새로 등장한 LangSmith Agent Builder는 노코드 에이전트 구축의 새로운 가능성을 열었습니다.

흥미로운 점은 이들이 상호 배타적이지 않다는 것입니다. DeepAgents는 Claude Code의 패턴을 일반화했으므로, Claude Code에서 배운 교훈을 DeepAgents에 적용할 수 있습니다. LangSmith Agent Builder는 DeepAgents 위에 구축되어 있어, 노코드로 시작하고 필요시 코드로 전환할 수 있습니다. 또한 세 시스템 모두 MCP를 지원하므로, 동일한 도구와 통합을 공유할 수 있습니다.

### 최종 권장사항 (2025년 12월 버전)

**빠른 프로토타이핑 또는 비개발자:**
→ **LangSmith Agent Builder** (베타)
- 대화형으로 에이전트 생성
- 내장 메모리와 승인 워크플로우
- MCP 통합으로 다양한 서비스 연결
- 팀 협업 및 공유 용이

**코딩 생산성 도구 또는 즉시 사용:**
→ **Claude Code**
- 설정 없이 바로 시작
- 최신 Claude 모델 사용
- 매끄러운 UX
- Agent Skills로 확장

**프로덕션 에이전트 시스템 또는 완전한 제어:**
→ **DeepAgents**
- 모든 측면 커스터마이징 가능
- 오픈소스, 벤더 중립
- LangGraph의 모든 기능 활용
- MCP 생태계 완전 통합

**권장 워크플로우:**
1. **아이디어 검증**: LangSmith Agent Builder로 빠른 프로토타입
2. **개발 가속**: Claude Code로 코드 작성 및 디버깅
3. **프로덕션 배포**: DeepAgents로 커스터마이징 및 확장
4. **모니터링**: LangSmith로 트레이싱 및 평가

어떤 도구를 선택하든, 2025년의 AI 에이전트 생태계는 그 어느 때보다 성숙하고 표준화되었습니다. MCP와 A2A 같은 표준 프로토콜, LangChain 1.1과 LangGraph 1.0 같은 안정적인 프레임워크, 그리고 LangSmith Agent Builder 같은 노코드 도구의 등장으로, 이제 누구나 프로덕션급 AI 에이전트를 만들 수 있는 시대가 되었습니다.

단순히 코드를 완성하거나 질문에 답하는 것을 넘어, 복잡한 작업을 자율적으로 수행하고, 계획하고, 적응하는 진정한 에이전트의 시대가 도래했습니다. Claude Code, DeepAgents, LangSmith Agent Builder는 각각 이 새로운 시대의 서로 다른 측면을 대표하며, 함께 사용할 때 가장 강력합니다.

---

**문서 작성일: 2025-12-18**

**주요 참고 자료:**
- Anthropic Engineering Blog: "Building agents with the Claude Agent SDK" (2025-09-29)
- Anthropic Engineering Blog: "Equipping agents for the real world with Agent Skills" (2025-10-16)
- LangChain Blog: "Deep Agents" (2025-08-04)
- LangChain Blog: "Doubling down on Deepagents" (2025-10-28)
- LangChain Blog: "LangChain & LangGraph 1.0 alpha releases" (2025-09-02)
- LangChain Blog: "Standard message content" (2025-09-03)
- LangChain Blog: "LangGraph Release Week Recap" (2025-06-09)
- LangChain Blog: "LangSmith Agent Builder now in Public Beta" (2025-12-02)
- LangChain Blog: "Introducing LangSmith's No Code Agent Builder" (2025-10-29)
- LangChain Changelog: https://changelog.langchain.com/
- TechCrunch: "Google launches managed MCP servers" (2025-12-10)
- Wikipedia: "Model Context Protocol" (Updated 2025-12-16)
- DataCamp: "LangSmith Agent Builder Tutorial" (2025-12-04)
- Minusx Blog: "What makes Claude Code so damn good" (2025-08-21)
- Han Lee: "Claude Agent Skills: A First Principles Deep Dive" (2025-10-26)
- GitHub: langchain-ai/deepagents
- GitHub: langchain-ai/langgraph
