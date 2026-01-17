---
title: "Claude Agent SDK 완전 가이드"
date: 2025-10-10 21:00:00 +0900
categories: [AI,  AI Agent]
mermaid: [True]
tags: [AI,  claude-agent-sdk,  claude-code,  sub-agents,  Claude.write]
---


멀티 에이전트 개인 비서 만들기

---

## 📋 목차

1. [소개](#1-소개)
2. [Claude Agent SDK란?](#2-claude-agent-sdk란)
3. [프로젝트 설정](#3-프로젝트-설정)
4. [핵심 개념](#4-핵심-개념)
5. [Claude Code 쿼리하기](#5-claude-code-쿼리하기)
6. [메시지 수신 및 처리](#6-메시지-수신-및-처리)
7. [커스텀 도구 구현](#7-커스텀-도구-구현)
8. [Claude Agent Options 심화](#8-claude-agent-options-심화)
9. [대화 루프 구현](#9-대화-루프-구현)
10. [MCP 서버 통합](#10-mcp-서버-통합)
11. [서브에이전트(Subagents)](#11-서브에이전트subagents)
12. [소스코드 구조 분석](#12-소스코드-구조-분석)
13. [실전 예제: 개인 비서 구축](#13-실전-예제-개인-비서-구축)
14. [모범 사례 및 팁](#14-모범-사례-및-팁)

---

## 1. 소개

### 1.1 개요

Claude Agent SDK는 Claude Code를 기반으로 구축된 프로덕션 레벨의 AI 에이전트 개발 도구입니다. 원래 Claude Code SDK로 시작했지만, 코딩 작업 외에도 광범위한 활용 가능성을 인정받아 Claude Agent SDK로 재명명되었습니다.

### 1.2 주요 기능

Claude Agent SDK는 다음과 같은 핵심 기능을 제공합니다:

- **자동 컨텍스트 관리**: 컨텍스트 압축 및 관리를 통해 에이전트가 컨텍스트 한계를 초과하지 않도록 보장
- **풍부한 도구 생태계**: 파일 작업, 코드 실행, 웹 검색, MCP 확장성
- **고급 권한 시스템**: 에이전트 기능에 대한 세밀한 제어
- **프로덕션 필수 요소**: 내장 오류 처리, 세션 관리, 모니터링
- **최적화된 Claude 통합**: 자동 프롬프트 캐싱 및 성능 최적화

### 1.3 활용 사례

Claude Agent SDK로 구축 가능한 에이전트 유형:

- **코딩 에이전트**: 프로덕션 문제 진단 및 수정, 코드 리뷰, 보안 감사
- **법률 어시스턴트**: 계약서 검토 및 규정 준수 검사
- **금융 어드바이저**: 보고서 및 예측 분석
- **고객 지원 에이전트**: 기술 문제 해결
- **콘텐츠 제작 어시스턴트**: 마케팅 팀을 위한 콘텐츠 생성
- **딥 리서치 에이전트**: 대규모 문서 컬렉션 분석

---

## 2. Claude Agent SDK란?

### 2.1 SDK의 정의

Claude Agent SDK는 Python과 TypeScript로 제공되는 라이브러리로, Claude에게 로컬 컴퓨터 접근 권한을 제공합니다. 단순히 텍스트를 생성하는 것이 아니라, 에이전트가 도구를 사용하여 환경과 상호작용할 수 있게 합니다.

### 2.2 핵심 아키텍처

Claude Agent SDK는 Claude Code를 구동하는 동일한 에이전트 하네스를 노출합니다. 이는 개발자가 모든 도구 호출을 직접 재구현할 필요 없이 간단한 에이전트를 구축할 수 있게 해줍니다.

**에이전트 루프 구조:**
```
사용자 쿼리 → 컨텍스트 수집 → 계획 수립 → 도구 호출 → 도구 응답 → 작업 검증 → 반복
```

### 2.3 기존 프레임워크와의 차이점

**Claude Agent SDK의 장점:**
- ✅ 높은 수준의 추상화로 에이전트 루프를 직접 구현할 필요 없음
- ✅ Anthropic 전문가들이 최적화한 에이전트 루프 활용
- ✅ Claude Code의 강력한 기능 바로 사용 가능
- ✅ 병렬 처리를 통한 빠른 실행 속도

**트레이드오프:**
- ⚠️ 에이전트 루프에 대한 세밀한 제어권 제한
- ⚠️ 높은 수준의 추상화로 인한 커스터마이징 제약
- ⚠️ 추적(tracing) 및 내부 시스템 프롬프트 가시성 부족

### 2.4 언제 사용해야 하는가?

**적합한 상황:**
- 빠르게 강력한 에이전트를 구축하고 싶을 때
- 처음부터 에이전트 루프를 구현하는 시간을 절약하고 싶을 때
- Claude Code의 검증된 아키텍처를 활용하고 싶을 때
- 비즈니스 로직에 집중하고 싶을 때

**부적합한 상황:**
- 에이전트 루프에 대한 완전한 제어가 필요할 때
- 복잡한 커스텀 에이전트 조정이 필요할 때
- 상세한 추적 및 디버깅이 중요할 때

---

## 3. 프로젝트 설정

### 3.1 사전 요구사항

**필수 설치 항목:**
- Python 3.10 이상
- Node.js 18 이상
- npm 패키지 매니저
- Google Chrome (Playwright MCP용)

### 3.2 설치 가이드

#### 3.2.1 UV 설치 (Python 프로젝트 관리 도구)

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

#### 3.2.2 Claude Code CLI 설치

```bash
npm install -g @anthropic-ai/claude-code
```

설치 후 PATH에 추가:
```bash
# Windows
# C:\Users\<user>\.local\bin을 시스템 PATH에 추가

# macOS/Linux
# 일반적으로 자동으로 PATH에 추가됨
```

#### 3.2.3 Claude Code 인증

```bash
claude auth login
```

프롬프트에 따라 Anthropic 계정으로 로그인합니다.

**참고:** Claude Pro 또는 Max 플랜이 있는 경우, "Log in with your subscription account"를 선택하면 사용량이 플랜에서 차감됩니다.

#### 3.2.4 Python SDK 설치

```bash
# 저장소 클론
git clone https://github.com/kenneth-liao/claude-agent-sdk-intro
cd claude-agent-sdk-intro

# 가상 환경 설정 및 의존성 설치
uv sync
```

### 3.3 API 키 설정 (선택사항)

Claude Code CLI로 인증한 경우 API 키 설정이 필요하지 않습니다. API 키를 직접 사용하려면:

```bash
# .env 파일 생성
echo "ANTHROPIC_API_KEY=your_api_key_here" > .env
```

### 3.4 프로젝트 구조 개요

```
claude-agent-sdk-intro/
├── .claude/                      # Claude Code 설정
│   ├── agents/                   # 서브에이전트 정의
│   │   └── document-creator.md
│   └── settings.json             # 프로젝트 설정
├── output-styles/                # 커스텀 출력 스타일
│   └── personal-assistant.md
├── hooks/                        # 이벤트 후크
│   └── log_agent_actions.py
├── logs/                         # 에이전트 실행 로그
├── db/                          # 데모용 데이터
│   └── products.json
├── cli_tools.py                 # CLI 유틸리티
├── 01_query.py                  # 기본 쿼리 예제
├── 02_messages.py               # 메시지 처리 예제
├── 03_tools.py                  # 커스텀 도구 예제
├── 04_options.py                # 옵션 설정 예제
├── 05_conversation_loop.py      # 대화 루프 예제
├── 06_mcp.py                    # MCP 서버 예제
├── 07_subagents.py              # 서브에이전트 예제
└── main.py                      # 완전한 개인 비서
```

---

## 4. 핵심 개념

### 4.1 에이전트 루프

Claude Agent SDK의 핵심은 자동화된 에이전트 루프입니다:

```
┌─────────────────────────────────────────┐
│  1. 사용자 쿼리 수신                     │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  2. 컨텍스트 수집 (파일 읽기, 검색 등)   │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  3. 작업 계획 수립                       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  4. 도구 호출 (병렬 처리 가능)           │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  5. 도구 응답 수신                       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  6. 작업 검증                            │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  7. 사용자 응답 또는 반복                │
└─────────────────────────────────────────┘
```

**이 모든 과정이 SDK에 의해 자동으로 처리됩니다!**

### 4.2 메시지 타입

Claude Agent SDK는 다양한 메시지 타입을 반환합니다:

1. **SystemMessage**: 에이전트의 전체 구성 정보
   - 사용 가능한 도구
   - 연결된 MCP 서버
   - 모델 정보
   - 권한 모드
   - 서브에이전트 목록

2. **UserMessage**: 사용자가 보낸 메시지

3. **AssistantMessage**: Claude의 응답
   - TextBlock: 일반 텍스트 응답
   - ToolUseBlock: 도구 호출
   - ThinkingBlock: 사고 과정 (extended thinking 사용 시)

4. **ToolMessage**: 도구 실행 결과

5. **ResultMessage**: 실행 완료 후 요약
   - 세션 ID
   - 토큰 사용량
   - 총 비용

### 4.3 컨텍스트 관리

Claude Agent SDK는 자동 컨텍스트 압축 및 관리를 제공하여 에이전트가 컨텍스트 한계를 초과하지 않도록 보장합니다.

**컨텍스트 최적화 기능:**
- 자동 프롬프트 캐싱: 반복되는 입력 토큰을 캐시하여 비용 절감
- 컨텍스트 압축: 긴 대화 내역을 효율적으로 압축
- 세션 관리: 여러 쿼리에 걸친 대화 지속성 유지

---

## 5. Claude Code 쿼리하기

### 5.1 두 가지 상호작용 방법

Claude Agent SDK는 두 가지 주요 방법을 제공합니다:

#### 5.1.1 query() 메서드

**특징:**
- 비상태(stateless) 쿼리
- 이전 대화 내역 유지 안 됨
- 일회성 질문에 적합
- 간단하고 빠름

**사용 예제:**

```python
from claude_agent_sdk import query, ClaudeAgentOptions

# 모델 설정
MODEL = "claude-sonnet-4-20250514"

# 에이전트 옵션 구성
options = ClaudeAgentOptions(
    model=MODEL
)

# 쿼리 실행
async def simple_query():
    async for message in query(
        prompt="안녕하세요",
        options=options
    ):
        print(message)
```

#### 5.1.2 ClaudeSDKClient

**특징:**
- 상태 유지(stateful) 세션
- 여러 쿼리에 걸친 대화 지속성
- 컨텍스트 공유 및 캐싱 활용
- 대부분의 사용 사례에 권장

**사용 예제:**

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

options = ClaudeAgentOptions(
    model=MODEL
)

async def client_example():
    # 컨텍스트 관리자로 클라이언트 설정
    async with ClaudeSDKClient(options=options) as client:
        # 첫 번째 쿼리
        await client.query("안녕하세요")
        async for message in client.receive_response():
            print(message)
        
        # 두 번째 쿼리 (같은 세션)
        await client.query("제 이름을 기억하시나요?")
        async for message in client.receive_response():
            print(message)
```

### 5.2 응답 메시지 구조

#### 5.2.1 SystemMessage

```python
# 첫 번째로 받는 메시지
SystemMessage(
    data={
        "tools": [...],           # 사용 가능한 도구 목록
        "mcp_servers": [...],     # 연결된 MCP 서버
        "model": "claude-sonnet-4-20250514",
        "permission_mode": "accept_edits",
        "agents": {...}           # 서브에이전트 정의
    }
)
```

#### 5.2.2 ResultMessage

```python
# 마지막에 받는 메시지
ResultMessage(
    session_id="abc123",
    usage={
        "input_tokens": 1000,
        "output_tokens": 500,
        "cache_creation_input_tokens": 800,
        "cache_read_input_tokens": 200
    },
    cost=0.00123  # USD
)
```

### 5.3 실행 예제

```bash
# 기본 쿼리 예제 실행
uv run 01_query.py

# 클라이언트 예제 실행
uv run 01_query.py --use-client
```

**출력 예시:**
```
=== System Message ===
Tools: bash, view, str_replace, create_file, ...
Model: claude-sonnet-4-20250514
Permission Mode: accept_edits

=== Assistant Response ===
안녕하세요! 무엇을 도와드릴까요?

=== Result ===
Session ID: abc123
Cost: $0.00123
```

---

## 6. 메시지 수신 및 처리

### 6.1 메시지 파싱 유틸리티

복잡한 메시지 객체를 읽기 쉽게 표시하기 위한 유틸리티를 구현합니다.

#### 6.1.1 CLI Tools 구조

**cli_tools.py:**

```python
from rich.console import Console
from rich.panel import Panel
from rich import print as rprint
from claude_agent_sdk import (
    SystemMessage,
    AssistantMessage,
    TextBlock,
    ToolUseBlock,
    ThinkingBlock
)

console = Console()

# 메시지 타입별 스타일 정의
MESSAGE_STYLES = {
    "system": "bold cyan",
    "assistant": "bold green",
    "tool": "bold yellow",
    "thinking": "bold magenta",
    "user": "bold blue"
}

def print_message(message):
    """메시지 타입에 따라 적절하게 포맷하여 출력"""
    
    if isinstance(message, SystemMessage):
        print_system_message(message)
    
    elif isinstance(message, AssistantMessage):
        print_assistant_message(message)
    
    # ... 기타 메시지 타입 처리

def print_system_message(message):
    """시스템 메시지를 패널로 표시"""
    tools = message.data.get("tools", [])
    mcp_servers = message.data.get("mcp_servers", {})
    
    content = f"""
📋 사용 가능한 도구: {len(tools)}
🔌 MCP 서버: {len(mcp_servers)}
🤖 모델: {message.data.get('model')}
🔐 권한 모드: {message.data.get('permission_mode')}
    """
    
    console.print(Panel(
        content,
        title="시스템 구성",
        style=MESSAGE_STYLES["system"]
    ))

def print_assistant_message(message):
    """어시스턴트 메시지 처리"""
    for block in message.content:
        if isinstance(block, TextBlock):
            rprint(f"[{MESSAGE_STYLES['assistant']}]🤖 Claude:[/] {block.text}")
        
        elif isinstance(block, ToolUseBlock):
            console.print(Panel(
                f"도구: {block.name}\n입력: {block.input}",
                title="🔧 도구 호출",
                style=MESSAGE_STYLES["tool"]
            ))
        
        elif isinstance(block, ThinkingBlock):
            console.print(Panel(
                block.thinking,
                title="💭 사고 과정",
                style=MESSAGE_STYLES["thinking"]
            ))
```

#### 6.1.2 CLI 인수 파싱

```python
import argparse

def parse_args():
    """명령줄 인수 파싱"""
    parser = argparse.ArgumentParser(description="Claude Agent SDK 예제")
    
    parser.add_argument(
        "--model",
        type=str,
        default="claude-sonnet-4-20250514",
        help="사용할 모델"
    )
    
    parser.add_argument(
        "--raw",
        action="store_true",
        help="원시 메시지 출력"
    )
    
    parser.add_argument(
        "--debug",
        action="store_true",
        help="디버그 모드 활성화"
    )
    
    return parser.parse_args()
```

### 6.2 메시지 처리 예제

**02_messages.py:**

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from cli_tools import print_message, parse_args

async def main():
    args = parse_args()
    
    options = ClaudeAgentOptions(
        model=args.model
    )
    
    async with ClaudeSDKClient(options=options) as client:
        await client.query("안녕하세요! 간단한 Python 함수를 작성해주세요.")
        
        async for message in client.receive_response():
            if args.raw:
                # 원시 메시지 출력
                print(message)
            else:
                # 포맷된 메시지 출력
                print_message(message)

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 6.3 실행 및 출력

```bash
# 포맷된 출력
uv run 02_messages.py

# 원시 메시지 출력
uv run 02_messages.py --raw

# 다른 모델 사용
uv run 02_messages.py --model claude-opus-4-20250514
```

**포맷된 출력 예시:**

```
┌─────────────────────────────────────┐
│         시스템 구성                  │
├─────────────────────────────────────┤
│ 📋 사용 가능한 도구: 12             │
│ 🔌 MCP 서버: 0                      │
│ 🤖 모델: claude-sonnet-4-20250514   │
│ 🔐 권한 모드: accept_edits          │
└─────────────────────────────────────┘

🤖 Claude: 물론이죠! 간단한 Python 함수를 작성해드리겠습니다.

┌─────────────────────────────────────┐
│         🔧 도구 호출                │
├─────────────────────────────────────┤
│ 도구: create_file                   │
│ 입력: {                             │
│   "path": "example.py",             │
│   "content": "def greet(name)..."   │
│ }                                   │
└─────────────────────────────────────┘

🤖 Claude: 파일을 생성했습니다!
```

---

## 7. 커스텀 도구 구현

### 7.1 도구 정의의 기본

Claude Agent SDK에서 커스텀 도구를 사용하려면 in-process MCP 서버로 구현해야 합니다. 이는 별도의 프로세스 없이 Python 애플리케이션 내에서 직접 실행됩니다.

#### 7.1.1 도구 데코레이터 사용

```python
from claude_agent_sdk import tool

    name="search_products",        # 도구 이름
    description="제품 데이터베이스에서 제품 검색",  # 도구 설명
    input_schema={                 # 입력 매개변수 스키마
        "query": str
    }
)
async def search_products(args: dict) -> dict:
    """
    제품 검색 도구
    
    Args:
        args: {"query": "검색어"} 형태의 딕셔너리
    
    Returns:
        {"content": [...]} 형태의 응답
    """
    query = args["query"]
    
    # 데이터베이스에서 검색 (예: JSON 파일)
    with open("db/products.json", "r") as f:
        products = json.load(f)
    
    results = [
        product for product in products
        if query.lower() in product["name"].lower()
        or query.lower() in product["description"].lower()
    ]
    
    return {
        "content": [
            {
                "type": "text",
                "text": json.dumps(results, indent=2, ensure_ascii=False)
            }
        ]
    }
```

**중요 규칙:**
- ✅ 함수는 반드시 `async`여야 함 (병렬 처리를 위해)
- ✅ 매개변수는 항상 `args: dict` 형태
- ✅ 반환값은 `{"content": [...]}` 형태의 딕셔너리
- ✅ `@tool` 데코레이터에서 입력 스키마 정의

### 7.2 SDK MCP 서버 생성

```python
from claude_agent_sdk import create_sdk_mcp_server

# 여러 도구를 하나의 서버로 묶기
server = create_sdk_mcp_server(
    name="product-tools",      # 서버 이름
    version="1.0.0",           # 버전
    tools=[                    # 도구 목록
        search_products,
        # create_product,
        # update_product,
        # delete_product
    ]
)
```

### 7.3 에이전트 옵션에 통합

```python
from claude_agent_sdk import ClaudeAgentOptions

options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514",
    
    # MCP 서버 추가
    mcp_servers={
        "products": server
    },
    
    # 도구 명시적으로 허용
    allowed_tools=[
        "mcp__products__search_products",
        # "mcp__products__create_product",
    ]
)
```

**도구 이름 규칙:**
```
mcp__{서버_이름}__{도구_이름}
```

예시:
- `mcp__products__search_products`
- `mcp__products__create_product`
- `mcp__email__send_message`

### 7.4 완전한 예제

**03_tools.py:**

```python
import json
from claude_agent_sdk import (
    tool,
    create_sdk_mcp_server,
    ClaudeSDKClient,
    ClaudeAgentOptions
)
from cli_tools import print_message, parse_args

# 1. 도구 정의
    name="search-products",
    description="우주 관련 장난감 데이터베이스에서 제품 검색",
    input_schema={"query": str}
)
async def search_products(args: dict) -> dict:
    query = args["query"]
    
    # 데이터 로드
    with open("db/products.json", "r", encoding="utf-8") as f:
        products = json.load(f)
    
    # 검색
    results = [
        p for p in products
        if query.lower() in p["name"].lower()
        or query.lower() in p["description"].lower()
    ]
    
    return {
        "content": [{
            "type": "text",
            "text": f"검색 결과 ({len(results)}개):\n\n" + 
                    json.dumps(results, indent=2, ensure_ascii=False)
        }]
    }

# 2. MCP 서버 생성
server = create_sdk_mcp_server(
    name="product-search",
    version="1.0.0",
    tools=[search_products]
)

# 3. 에이전트 실행
async def main():
    args = parse_args()
    
    options = ClaudeAgentOptions(
        model=args.model,
        mcp_servers={"products": server},
        allowed_tools=["mcp__products__search-products"]
    )
    
    async with ClaudeSDKClient(options=options) as client:
        await client.query(
            "아이들을 위한 망원경을 찾아주세요. "
            "가격과 추천 연령대를 알려주세요."
        )
        
        async for message in client.receive_response():
            print_message(message)

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

**db/products.json:**

```json
[
  {
    "id": 1,
    "name": "어린이용 천체 망원경",
    "description": "초보자를 위한 70mm 굴절 망원경",
    "price": 89.99,
    "age_range": "8-14세",
    "category": "astronomy"
  },
  {
    "id": 2,
    "name": "태양계 모형 키트",
    "description": "움직이는 행성들이 있는 교육용 모형",
    "price": 34.99,
    "age_range": "6-12세",
    "category": "educational"
  },
  {
    "id": 3,
    "name": "우주비행사 역할놀이 세트",
    "description": "헬멧과 배낭이 포함된 완전한 세트",
    "price": 49.99,
    "age_range": "4-10세",
    "category": "pretend-play"
  }
]
```

### 7.5 실행 결과

```bash
uv run 03_tools.py
```

**출력:**

```
┌─────────────────────────────────────┐
│         🔧 도구 호출                │
├─────────────────────────────────────┤
│ 도구: search-products               │
│ 입력: {"query": "망원경"}           │
└─────────────────────────────────────┘

🤖 Claude: 검색 결과를 찾았습니다. 
아이들을 위한 망원경을 추천해드리겠습니다:

**어린이용 천체 망원경**
- 가격: $89.99
- 추천 연령: 8-14세
- 설명: 초보자를 위한 70mm 굴절 망원경

이 망원경은 천문학에 관심 있는 어린이들에게 
완벽한 선택입니다!
```

---

## 8. Claude Agent Options 심화

### 8.1 ClaudeAgentOptions 개요

`ClaudeAgentOptions`는 에이전트의 동작을 세밀하게 제어하는 핵심 구성 클래스입니다.

### 8.2 주요 설정 옵션

#### 8.2.1 모델 선택

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514"  # 기본 추천
    # model="claude-opus-4-20250514"   # 더 강력한 모델
    # model="claude-haiku-4-20250514"  # 더 빠르고 저렴한 모델
)
```

#### 8.2.2 도구 권한 설정

```python
options = ClaudeAgentOptions(
    # 명시적으로 허용할 도구
    allowed_tools=[
        "bash",
        "view",
        "create_file",
        "str_replace",
        "mcp__products__search_products"
    ],
    
    # 차단할 도구
    disallowed_tools=[
        "web_search",      # 웹 검색 차단
        "web_fetch"        # 웹 페치 차단
    ]
)
```

**중요:** `allowed_tools`와 `disallowed_tools`는 권한 모드 설정보다 우선합니다.

#### 8.2.3 권한 모드

```python
options = ClaudeAgentOptions(
    permission_mode="accept_edits"  # 권장 설정
)
```

**권한 모드 종류:**

| 모드 | 설명 | 사용 시나리오 |
|------|------|--------------|
| `default` | 파일 수정 시 승인 요청, 읽기는 자동 승인 | 일반적인 개발 작업 |
| `accept_edits` | 편집 자동 승인 | 신뢰할 수 있는 작업 자동화 |
| `plan` | 도구 실행 안 함, 계획만 수립 | 안전한 테스트 환경 |
| `permission_bypass` | 모든 도구 자동 승인 (YOLO 모드) | 완전 자동화 (주의 필요) |

#### 8.2.4 설정 소스

```python
options = ClaudeAgentOptions(
    setting_sources=["project"]  # 또는 ["user"]
)
```

**설정 로드 위치:**

**`project`:**
- 프로젝트 루트의 `CLAUDE.md`
- `.claude/settings.json`
- `.claude/agents/` (서브에이전트)
- `.claude/commands/` (슬래시 명령)

**`user`:**
- `~/.claude/CLAUDE.md`
- `~/.claude/settings.json`

#### 8.2.5 디렉토리 접근 제어

```python
options = ClaudeAgentOptions(
    add_directories=[
        "/path/to/other/project",
        "/home/user/documents"
    ]
)
```

기본적으로 에이전트는 현재 작업 디렉토리에만 접근 가능합니다. `add_directories`로 추가 경로를 허용할 수 있습니다.

### 8.3 시스템 프롬프트 수정

#### 8.3.1 네 가지 방법

Claude Agent SDK는 시스템 프롬프트를 수정하는 여러 방법을 제공합니다:

**1. CLAUDE.md 파일 (프로젝트 수준)**

```markdown

# 프로젝트 컨텍스트

이 프로젝트는 전자상거래 플랫폼입니다.

## 코딩 스타일
- Python: PEP 8 준수
- 타입 힌트 사용
- Docstring 필수

## 중요 파일
- `src/models/`: 데이터 모델
- `src/api/`: API 엔드포인트
```

**2. 추가 시스템 프롬프트 (Append 모드)**

```python
options = ClaudeAgentOptions(
    system_prompt="항상 한국어로 응답하세요. 친절하고 전문적인 톤을 유지하세요.",
    # 기존 시스템 프롬프트에 추가됨
)
```

**3. 커스텀 시스템 프롬프트 (Replace 모드)**

```python
options = ClaudeAgentOptions(
    custom_system_prompt="""
    당신은 해적입니다. 해적처럼 말하세요.
    모든 응답에 "아호이!"를 포함하세요.
    """
    # 기존 시스템 프롬프트를 완전히 대체
    # ⚠️ 주의: 도구 사용 지침도 사라짐!
)
```

**4. 출력 스타일 (Output Style)**

```python
# settings.json 또는 코드에서 설정
options = ClaudeAgentOptions(
    settings=json.dumps({
        "outputStyle": "personal-assistant"
    })
)
```

**output-styles/personal-assistant.md:**

```markdown
# 개인 비서 - Kaya

당신은 Kaya라는 이름의 개인 AI 비서입니다.

## 정체성
- 효율적이고 프로액티브한 비서
- 사용자의 생산성을 최우선으로 생각
- 전문적이지만 친근한 톤 유지

## 커뮤니케이션 스타일
- 간결하고 명확한 응답
- 중요한 정보는 **굵게** 강조
- 필요시 단계별 설명 제공

## 하위 에이전트 사용
다음 하위 에이전트를 활용하세요:
- **youtube-analyst**: YouTube 분석 및 댓글 검토
- **researcher**: 웹 리서치 및 보고서 작성
- **document-creator**: 문서 작성 및 포맷팅
```

#### 8.3.2 출력 스타일의 장점

출력 스타일은 다음을 유지하면서 역할을 커스터마이징할 수 있습니다:
- ✅ 모든 도구 사용 지침
- ✅ 환경 상호작용 방법
- ✅ Claude Code의 핵심 기능
- ❌ 역할과 톤만 변경

**권장:** 대부분의 경우 출력 스타일 사용을 권장합니다.

### 8.4 후크(Hooks) 설정

#### 8.4.1 후크란?

후크는 에이전트 실행 중 특정 이벤트에서 자동으로 실행되는 콜백입니다.

**사용 가능한 이벤트:**
- `beforeToolUse`: 도구 호출 전
- `afterToolUse`: 도구 호출 후
- `onNotification`: 알림 발생 시
- `onStop`: 에이전트 중지 시

#### 8.4.2 설정 방법

**settings.json:**

```json
{
  "outputStyle": "personal-assistant",
  "hooks": [
    {
      "event": "onStop",
      "command": "afplay /System/Library/Sounds/Glass.aiff"
    },
    {
      "event": "onStop",
      "runCommand": "uv run hooks/log_agent_actions.py"
    }
  ]
}
```

#### 8.4.3 로깅 후크 예제

**hooks/log_agent_actions.py:**

```python
import json
import sys
from datetime import datetime
from pathlib import Path

def parse_messages():
    """stdin에서 메시지를 읽어 파싱"""
    messages = []
    for line in sys.stdin:
        try:
            msg = json.loads(line)
            messages.append(msg)
        except json.JSONDecodeError:
            continue
    return messages

def create_log(messages):
    """에이전트 행동 로그 생성"""
    log = []
    
    for msg in messages:
        if msg.get("type") == "tool_use":
            log.append({
                "timestamp": datetime.now().isoformat(),
                "action": "tool_call",
                "tool": msg.get("name"),
                "input": msg.get("input")
            })
        
        elif msg.get("type") == "tool_result":
            log.append({
                "timestamp": datetime.now().isoformat(),
                "action": "tool_result",
                "status": "success" if not msg.get("is_error") else "error"
            })
    
    return log

def main():
    messages = parse_messages()
    log = create_log(messages)
    
    # 로그 저장
    log_dir = Path("logs")
    log_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    log_file = log_dir / f"agent_actions_{timestamp}.json"
    
    with open(log_file, "w") as f:
        json.dump(log, f, indent=2)
    
    print(f"로그 저장됨: {log_file}")

if __name__ == "__main__":
    main()
```

### 8.5 완전한 옵션 예제

**04_options.py:**

```python
import json
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

async def main():
    options = ClaudeAgentOptions(
        # 모델 설정
        model="claude-sonnet-4-20250514",
        
        # 권한 설정
        permission_mode="accept_edits",
        allowed_tools=[
            "bash",
            "view",
            "create_file",
            "str_replace",
        ],
        disallowed_tools=[
            "web_search",
            "web_fetch"
        ],
        
        # 설정 로드
        setting_sources=["project"],
        
        # 추가 디렉토리 접근
        add_directories=[
            str(Path.home() / "Documents")
        ],
        
        # 또는 코드에서 직접 설정
        # settings=json.dumps({
        #     "outputStyle": "personal-assistant",
        #     "hooks": [...]
        # })
    )
    
    async with ClaudeSDKClient(options=options) as client:
        await client.query("프로젝트 구조를 분석해주세요.")
        
        async for message in client.receive_response():
            print_message(message)

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

---

## 9. 대화 루프 구현

### 9.1 기본 대화 루프

이전 예제들은 단일 쿼리만 처리했습니다. 실제 비서를 만들려면 지속적인 대화가 필요합니다.

**05_conversation_loop.py:**

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from cli_tools import print_message, parse_args

async def main():
    args = parse_args()
    
    options = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        setting_sources=["project"]
    )
    
    async with ClaudeSDKClient(options=options) as client:
        print("🤖 개인 비서 Kaya입니다. 종료하려면 'exit' 또는 'quit'를 입력하세요.\n")
        
        # 무한 대화 루프
        while True:
            # 사용자 입력 받기
            user_input = input("👤 당신: ").strip()
            
            # 종료 조건
            if user_input.lower() in ["exit", "quit", "종료"]:
                print("👋 안녕히 가세요!")
                break
            
            # 빈 입력 무시
            if not user_input:
                continue
            
            # 쿼리 전송
            await client.query(user_input)
            
            # 응답 수신 및 출력
            async for message in client.receive_response():
                if not args.raw:
                    print_message(message)
                else:
                    print(message)
            
            print()  # 가독성을 위한 빈 줄

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 9.2 대화 컨텍스트 유지

`ClaudeSDKClient`를 사용하면 자동으로 대화 컨텍스트가 유지됩니다:

```python
async with ClaudeSDKClient(options=options) as client:
    # 첫 번째 쿼리
    await client.query("제 이름은 김철수입니다.")
    async for msg in client.receive_response():
        pass
    
    # 두 번째 쿼리 - 이전 컨텍스트 기억
    await client.query("제 이름이 뭐죠?")
    async for msg in client.receive_response():
        print_message(msg)
    # 출력: "당신의 이름은 김철수입니다."
```

### 9.3 향상된 대화 루프

실전에서는 더 많은 기능이 필요합니다:

```python
import asyncio
from pathlib import Path
from datetime import datetime

class ConversationManager:
    """대화 관리 클래스"""
    
    def __init__(self, client: ClaudeSDKClient):
        self.client = client
        self.history = []
        self.session_start = datetime.now()
    
    async def send_query(self, user_input: str):
        """쿼리 전송 및 히스토리 저장"""
        self.history.append({
            "role": "user",
            "content": user_input,
            "timestamp": datetime.now()
        })
        
        await self.client.query(user_input)
    
    async def receive_and_display(self):
        """응답 수신 및 표시"""
        assistant_response = []
        
        async for message in self.client.receive_response():
            print_message(message)
            
            # 어시스턴트 응답 저장
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        assistant_response.append(block.text)
        
        # 히스토리에 추가
        if assistant_response:
            self.history.append({
                "role": "assistant",
                "content": "\n".join(assistant_response),
                "timestamp": datetime.now()
            })
    
    def save_history(self):
        """대화 히스토리 저장"""
        log_dir = Path("logs/conversations")
        log_dir.mkdir(parents=True, exist_ok=True)
        
        filename = f"conversation_{self.session_start.strftime('%Y%m%d_%H%M%S')}.json"
        filepath = log_dir / filename
        
        with open(filepath, "w", encoding="utf-8") as f:
            json.dump(self.history, f, indent=2, ensure_ascii=False, default=str)
        
        print(f"\n💾 대화 저장됨: {filepath}")

async def main():
    args = parse_args()
    
    options = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        setting_sources=["project"]
    )
    
    async with ClaudeSDKClient(options=options) as client:
        manager = ConversationManager(client)
        
        print("🤖 개인 비서 Kaya입니다.")
        print("명령어: 'exit' (종료), 'save' (대화 저장), 'clear' (화면 지우기)\n")
        
        try:
            while True:
                user_input = input("👤 당신: ").strip()
                
                # 명령어 처리
                if user_input.lower() in ["exit", "quit"]:
                    manager.save_history()
                    print("👋 안녕히 가세요!")
                    break
                
                elif user_input.lower() == "save":
                    manager.save_history()
                    continue
                
                elif user_input.lower() == "clear":
                    import os
                    os.system('clear' if os.name != 'nt' else 'cls')
                    continue
                
                elif not user_input:
                    continue
                
                # 일반 쿼리 처리
                await manager.send_query(user_input)
                await manager.receive_and_display()
                print()
        
        except KeyboardInterrupt:
            print("\n\n⚠️ 중단됨. 대화를 저장합니다...")
            manager.save_history()

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 9.4 실행 예제

```bash
uv run 05_conversation_loop.py
```

**대화 예시:**

```
🤖 개인 비서 Kaya입니다.
명령어: 'exit' (종료), 'save' (대화 저장), 'clear' (화면 지우기)

👤 당신: 안녕하세요! 오늘 날씨 어때요?

🤖 Claude: 안녕하세요! 죄송하지만 저는 실시간 날씨 정보에 
접근할 수 없습니다. 웹 검색 기능이 비활성화되어 있습니다.
대신 다른 도움을 드릴 수 있을까요?

👤 당신: 그럼 Python으로 간단한 계산기를 만들어주세요.

┌─────────────────────────────────────┐
│         🔧 도구 호출                │
├─────────────────────────────────────┤
│ 도구: create_file                   │
│ 입력: {                             │
│   "path": "calculator.py",          │
│   "content": "def add(a, b)..."     │
│ }                                   │
└─────────────────────────────────────┘

🤖 Claude: 간단한 계산기를 만들었습니다! 
calculator.py 파일을 확인해보세요.

👤 당신: save
💾 대화 저장됨: logs/conversations/conversation_20250110_143022.json

👤 당신: exit
💾 대화 저장됨: logs/conversations/conversation_20250110_143022.json
👋 안녕히 가세요!
```

---

## 10. MCP 서버 통합

### 10.1 MCP란?

Model Context Protocol (MCP)은 외부 서비스와의 표준화된 통합을 제공하며, 인증 및 API 호출을 자동으로 처리합니다.

### 10.2 외부 MCP 서버 사용

#### 10.2.1 Playwright MCP 서버 추가

Playwright MCP는 브라우저 자동화 기능을 제공합니다.

**MCP 서버 구성:**

```python
from claude_agent_sdk import ClaudeAgentOptions

# Playwright MCP 서버 설정
playwright_mcp = {
    "command": "npx",
    "args": [
        "-y",
        "@modelcontextprotocol/server-playwright"
    ]
}

options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514",
    permission_mode="accept_edits",
    
    # MCP 서버 추가
    mcp_servers={
        "playwright": playwright_mcp
    },
    
    # 도구 권한 - 중요!
    allowed_tools=[
        "bash",
        "view",
        # Playwright 도구는 아직 허용 안 함
    ]
)
```

#### 10.2.2 MCP 도구 권한 부여

MCP 서버를 추가해도 도구를 자동으로 사용할 수 없습니다. **명시적으로 허용해야 합니다**.

**도구 이름 확인:**

1. 에이전트 실행
2. SystemMessage에서 사용 가능한 도구 확인
3. 원하는 도구의 정확한 이름 복사

**예제:**

```python
options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514",
    permission_mode="accept_edits",
    mcp_servers={"playwright": playwright_mcp},
    
    # Playwright 도구 허용
    allowed_tools=[
        "bash",
        "view",
        "mcp__playwright__playwright_navigate",
        "mcp__playwright__playwright_screenshot",
        "mcp__playwright__playwright_click",
        "mcp__playwright__playwright_fill",
        "mcp__playwright__playwright_evaluate"
    ]
)
```

### 10.3 MCP 예제

**06_mcp.py:**

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from cli_tools import print_message, parse_args

# Playwright MCP 서버 설정
playwright_mcp = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-playwright"]
}

async def main():
    args = parse_args()
    
    # 1단계: 도구 확인 (권한 없이)
    print("=== 1단계: 사용 가능한 도구 확인 ===\n")
    
    options_check = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        mcp_servers={"playwright": playwright_mcp},
        allowed_tools=["bash", "view"]  # MCP 도구 허용 안 함
    )
    
    async with ClaudeSDKClient(options=options_check) as client:
        await client.query("Playwright를 사용해 YouTube를 열어주세요.")
        
        async for message in client.receive_response():
            if isinstance(message, SystemMessage):
                tools = [t for t in message.data["tools"] 
                        if "playwright" in t.get("name", "")]
                print("Playwright 도구:")
                for tool in tools:
                    print(f"  - {tool['name']}")
                print()
            
            elif isinstance(message, AssistantMessage):
                print_message(message)
    
    print("\n" + "="*50 + "\n")
    
    # 2단계: 도구 권한 부여 후 실행
    print("=== 2단계: 권한 부여 후 실행 ===\n")
    
    options_allowed = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        mcp_servers={"playwright": playwright_mcp},
        allowed_tools=[
            "bash",
            "view",
            "mcp__playwright__playwright_navigate",
            "mcp__playwright__playwright_screenshot"
        ]
    )
    
    async with ClaudeSDKClient(options=options_allowed) as client:
        await client.query(
            "Playwright를 사용해 YouTube를 열고 스크린샷을 찍어주세요."
        )
        
        async for message in client.receive_response():
            print_message(message)

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 10.4 실행 결과

```bash
uv run 06_mcp.py
```

**출력:**

```
=== 1단계: 사용 가능한 도구 확인 ===

Playwright 도구:
  - mcp__playwright__playwright_navigate
  - mcp__playwright__playwright_screenshot
  - mcp__playwright__playwright_click
  - mcp__playwright__playwright_fill
  - mcp__playwright__playwright_select
  - mcp__playwright__playwright_hover
  - mcp__playwright__playwright_evaluate

🤖 Claude: Playwright 도구가 있는 것을 확인했지만, 
사용 권한이 없습니다. 도구를 허용해주세요.

==================================================

=== 2단계: 권한 부여 후 실행 ===

┌─────────────────────────────────────┐
│         🔧 도구 호출                │
├─────────────────────────────────────┤
│ 도구: playwright_navigate           │
│ 입력: {"url": "https://youtube.com"}│
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│         🔧 도구 호출                │
├─────────────────────────────────────┤
│ 도구: playwright_screenshot         │
│ 입력: {"name": "youtube_home"}      │
└─────────────────────────────────────┘

🤖 Claude: YouTube를 열고 스크린샷을 찍었습니다!
```

### 10.5 다른 MCP 서버들

**인기 있는 MCP 서버들:**

```python
# Google Drive MCP
gdrive_mcp = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-gdrive"]
}

# GitHub MCP
github_mcp = {
    "command": "npx",
    "args": [
        "-y",
        "@modelcontextprotocol/server-github"
    ],
    "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
    }
}

# Slack MCP
slack_mcp = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-slack"],
    "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-token",
        "SLACK_TEAM_ID": "T01234567"
    }
}

# 모두 추가
options = ClaudeAgentOptions(
    mcp_servers={
        "playwright": playwright_mcp,
        "gdrive": gdrive_mcp,
        "github": github_mcp,
        "slack": slack_mcp
    },
    allowed_tools=[
        # 각 MCP의 필요한 도구들...
    ]
)
```

---

## 11. 서브에이전트(Subagents)

### 11.1 서브에이전트의 개념

서브에이전트는 특정 작업에 특화된 전문 에이전트입니다. 메인 에이전트가 여러 서브에이전트를 생성하여 병렬로 작업을 수행할 수 있어, 생산성이 기하급수적으로 증가합니다.

**장점:**
- 🎯 **컨텍스트 격리**: 각 서브에이전트는 독립적인 컨텍스트
- 🔒 **도구 권한 분리**: 각자 다른 도구 세트 사용
- ⚡ **병렬 처리**: 여러 작업을 동시에 수행
- 📊 **전문화**: 각 에이전트가 특정 역할에 최적화

### 11.2 서브에이전트 정의 방법

#### 11.2.1 코드에서 정의

```python
from claude_agent_sdk import ClaudeAgentOptions, AgentDefinition

# 서브에이전트 정의
youtube_analyst = AgentDefinition(
    description="YouTube 채널 성과를 분석하는 전문가",
    prompt="""
당신은 YouTube 분석 전문가입니다.

## 역할
- YouTube Analytics 데이터 해석
- 동영상 성과 메트릭 분석
- 댓글 감성 분석
- 구독자 성장 추세 파악

## 도구
- Playwright를 사용해 YouTube Studio 접근
- 데이터를 구조화된 형식으로 정리
- 인사이트와 권장사항 제공

## 출력 형식
마크다운 보고서 형식으로 결과를 제공하세요.
    """,
    model="claude-sonnet-4-20250514",
    tools=[
        "mcp__playwright__playwright_navigate",
        "mcp__playwright__playwright_click",
        "mcp__playwright__playwright_screenshot",
        "view",
        "create_file"
    ]
)

# 리서처 에이전트
researcher = AgentDefinition(
    description="웹 리서치를 수행하고 종합 보고서를 작성하는 전문가",
    prompt="""
당신은 전문 리서처입니다.

## 역할
- 여러 소스에서 정보 수집
- 신뢰할 수 있는 출처 우선
- 데이터 교차 검증
- 종합 보고서 작성

## 리서치 프로세스
1. 키워드 식별 및 검색 쿼리 최적화
2. 여러 소스에서 정보 수집 (최소 5개)
3. 정보의 신뢰성 평가
4. 핵심 인사이트 추출
5. 구조화된 보고서 작성

## 출력
- 마크다운 형식
- 목차 포함
- 모든 주장에 출처 표시
- 실행 가능한 인사이트 제공
    """,
    model="claude-sonnet-4-20250514",
    tools=[
        "web_search",
        "web_fetch",
        "create_file",
        "str_replace"
    ]
)

# 옵션에 추가
options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514",
    permission_mode="accept_edits",
    setting_sources=["project"],
    
    # 서브에이전트 정의
    agents={
        "youtube-analyst": youtube_analyst,
        "researcher": researcher
    },
    
    # MCP 서버
    mcp_servers={"playwright": playwright_mcp},
    
    # 메인 에이전트와 서브에이전트의 모든 도구 허용
    allowed_tools=[
        "bash",
        "view",
        "create_file",
        "str_replace",
        "web_search",
        "web_fetch",
        "mcp__playwright__playwright_navigate",
        "mcp__playwright__playwright_click",
        "mcp__playwright__playwright_screenshot"
    ]
)
```

**중요:** 서브에이전트의 도구는 메인 에이전트의 `allowed_tools`에도 포함되어야 합니다!

#### 11.2.2 파일에서 정의

**.claude/agents/document-creator.md:**

```markdown
---
description: 전문적인 문서를 작성하는 전문가
model: claude-sonnet-4-20250514
tools:
  - view
  - create_file
  - str_replace
  - bash
---

# 문서 작성 전문가

당신은 전문적인 문서를 작성하는 전문가입니다.

## 역할
- 마크다운, DOCX, PDF 등 다양한 형식의 문서 작성
- 명확하고 구조화된 콘텐츠
- 전문적인 포맷팅
- 시각적 요소 포함 (표, 다이어그램 등)

## 문서 작성 가이드라인

### 구조
1. **제목과 목차**: 명확한 계층 구조
2. **서론**: 문서의 목적과 범위
3. **본문**: 논리적 흐름으로 구성
4. **결론**: 핵심 요점 요약
5. **참고자료**: 출처 및 추가 자료

### 스타일
- 간결하고 명확한 문장
- 전문 용어는 처음 사용 시 설명
- 적절한 헤딩 사용 (H1-H4)
- 중요 정보는 **굵게** 또는 *기울임* 강조

### 포맷
- 마크다운: 웹 문서, README
- DOCX: 공식 보고서, 제안서
- PDF: 최종 배포본

## 도구 사용
- `view`: 참고 문서 읽기
- `create_file`: 새 문서 생성
- `str_replace`: 문서 수정
- `bash`: 문서 변환 (pandoc 등)
```

설정 파일에서 자동으로 로드됩니다:

```python
options = ClaudeAgentOptions(
    setting_sources=["project"],  # .claude/ 폴더에서 로드
    # agents={}는 지정하지 않아도 됨
)
```

### 11.3 서브에이전트 활용 예제

**07_subagents.py:**

```python
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AgentDefinition
)
from cli_tools import print_message, parse_args

# Playwright MCP
playwright_mcp = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-playwright"]
}

# 서브에이전트 정의
youtube_analyst = AgentDefinition(
    description="YouTube 채널 성과를 분석하는 전문가",
    prompt="""당신은 YouTube 분석 전문가입니다.
    
Playwright를 사용해 YouTube 분석 데이터를 수집하고,
성과 메트릭, 댓글 감성, 성장 추세를 분석하여
마크다운 보고서로 작성하세요.""",
    model="claude-sonnet-4-20250514",
    tools=[
        "mcp__playwright__playwright_navigate",
        "mcp__playwright__playwright_screenshot",
        "view",
        "create_file"
    ]
)

researcher = AgentDefinition(
    description="웹 리서치를 수행하고 종합 보고서를 작성하는 전문가",
    prompt="""당신은 전문 리서처입니다.
    
여러 신뢰할 수 있는 소스에서 정보를 수집하고,
교차 검증하여 종합 보고서를 작성하세요.
모든 주장에는 출처를 명시하세요.""",
    model="claude-sonnet-4-20250514",
    tools=["web_search", "web_fetch", "create_file"]
)

async def main():
    args = parse_args()
    
    options = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        setting_sources=["project"],  # document-creator도 로드됨
        
        agents={
            "youtube-analyst": youtube_analyst,
            "researcher": researcher
        },
        
        mcp_servers={"playwright": playwright_mcp},
        
        allowed_tools=[
            "bash",
            "view",
            "create_file",
            "str_replace",
            "web_search",
            "web_fetch",
            "mcp__playwright__playwright_navigate",
            "mcp__playwright__playwright_screenshot"
        ]
    )
    
    # 복잡한 멀티 에이전트 작업
    complex_query = """
안녕, Kaya! 다음 작업들을 병렬로 처리해줘:

1. YouTube 분석: 내 최신 동영상의 성과를 분석하고 댓글도 확인해줘.
2. 리서치: 콘텐츠 크리에이터를 위한 최고의 AI 도구를 조사해줘.
3. 문서 작성: Claude Agent SDK에서 후크를 구현하는 방법에 대한 
   실용적인 가이드를 작성해줘. 공식 문서를 참고해.
    """
    
    async with ClaudeSDKClient(options=options) as client:
        print("🚀 멀티 에이전트 작업 시작...\n")
        
        await client.query(complex_query)
        
        async for message in client.receive_response():
            print_message(message)

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 11.4 실행 결과

```bash
uv run 07_subagents.py
```

**출력 예시:**

```
🚀 멀티 에이전트 작업 시작...

┌─────────────────────────────────────┐
│      💼 서브에이전트 호출            │
├─────────────────────────────────────┤
│ 에이전트: youtube-analyst           │
│ 작업: YouTube 분석 수행             │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│      💼 서브에이전트 호출            │
├─────────────────────────────────────┤
│ 에이전트: researcher                │
│ 작업: AI 도구 리서치                │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│      💼 서브에이전트 호출            │
├─────────────────────────────────────┤
│ 에이전트: document-creator          │
│ 작업: 후크 가이드 작성              │
└─────────────────────────────────────┘

[병렬 처리 중...]

🤖 Kaya: 세 가지 작업이 모두 완료되었습니다!

📊 **YouTube 분석 보고서**
- 파일: youtube_analysis_20250110.md
- 최신 동영상 조회수: 1,741
- 평균 시청 시간: 18분 32초
- 긍정 댓글: 92%

🔍 **AI 도구 리서치**
- 파일: ai_tools_for_creators.md
- 조사한 도구: 15개
- 카테고리: 영상 편집, 썸네일 생성, 스크립트 작성

📖 **후크 구현 가이드**
- 파일: claude_agent_hooks_guide.md
- 4가지 후크 타입 설명
- 실용적인 예제 코드 포함
```

### 11.5 서브에이전트 모범 사례

**1. 명확한 역할 정의:**
```markdown
✅ 좋은 예: "YouTube 채널 성과를 분석하는 전문가"
❌ 나쁜 예: "데이터를 분석하는 에이전트"
```

**2. 적절한 도구 할당:**
```python
# YouTube 분석가는 Playwright만 필요
youtube_analyst.tools = [
    "mcp__playwright__playwright_navigate",
    "mcp__playwright__playwright_screenshot",
    "view",
    "create_file"
]

# 리서처는 웹 검색만 필요
researcher.tools = [
    "web_search",
    "web_fetch",
    "create_file"
]
```

**3. 상세한 프롬프트:**
```python
prompt="""
당신은 YouTube 분석 전문가입니다.

## 역할
- YouTube Analytics 데이터 해석
- ...

## 프로세스
1. Playwright로 YouTube Studio 접근
2. 최신 동영상 메트릭 수집
3. ...

## 출력 형식
마크다운 보고서:
- 요약
- 상세 메트릭
- 인사이트
- 권장사항
"""
```

**4. 메인 에이전트의 역할:**

메인 에이전트(개인 비서)는 **오케스트레이터** 역할:
- 사용자 의도 파악
- 적절한 서브에이전트 선택
- 작업 분배 및 조정
- 결과 통합 및 요약

---

## 12. 소스코드 구조 분석

### 12.1 프로젝트 디렉토리 구조

```
claude-agent-sdk-intro/
│
├── .claude/                          # Claude Code 설정 디렉토리
│   ├── agents/                       # 서브에이전트 정의
│   │   └── document-creator.md      # 문서 작성 에이전트
│   ├── commands/                     # 슬래시 명령 (옵션)
│   └── settings.json                 # 프로젝트 설정
│
├── output-styles/                    # 출력 스타일 정의
│   └── personal-assistant.md         # 개인 비서 스타일
│
├── hooks/                            # 이벤트 후크 스크립트
│   └── log_agent_actions.py         # 에이전트 행동 로깅
│
├── logs/                             # 실행 로그
│   ├── conversations/                # 대화 히스토리
│   └── agent_actions_*.json         # 에이전트 행동 로그
│
├── db/                              # 데모 데이터
│   └── products.json                # 제품 데이터베이스
│
├── outputs/                         # 생성된 출력물
│   ├── youtube_analysis.md
│   ├── ai_tools_research.md
│   └── hooks_guide.md
│
├── cli_tools.py                     # CLI 유틸리티
├── 01_query.py                      # 기본 쿼리 예제
├── 02_messages.py                   # 메시지 처리
├── 03_tools.py                      # 커스텀 도구
├── 04_options.py                    # 옵션 설정
├── 05_conversation_loop.py          # 대화 루프
├── 06_mcp.py                        # MCP 통합
├── 07_subagents.py                  # 서브에이전트
├── main.py                          # 완전한 개인 비서
│
├── CLAUDE.md                        # 프로젝트 컨텍스트
├── pyproject.toml                   # Python 프로젝트 설정
├── uv.lock                          # 의존성 잠금 파일
└── README.md                        # 프로젝트 문서
```

### 12.2 핵심 파일 분석

#### 12.2.1 CLAUDE.md

프로젝트 수준의 컨텍스트 파일:

```markdown
# Claude Agent SDK 튜토리얼 프로젝트

이 프로젝트는 Claude Agent SDK를 사용하여 개인 AI 비서를 
만드는 방법을 단계별로 보여줍니다.

## 프로젝트 구조
- `01_query.py` ~ `07_subagents.py`: 단계별 예제
- `main.py`: 완전한 개인 비서 구현
- `.claude/`: Claude Code 설정
- `hooks/`: 이벤트 후크
- `db/`: 데모 데이터

## 코딩 규칙
- Python 3.10+ 사용
- 타입 힌트 권장
- async/await 패턴 사용
- 명확한 함수 docstring

## 중요 사항
- 모든 MCP 도구는 명시적으로 허용 필요
- 서브에이전트의 도구는 메인 에이전트에서도 허용
- 후크는 settings.json에서 설정
```

#### 12.2.2 .claude/settings.json

```json
{
  "outputStyle": "personal-assistant",
  "hooks": [
    {
      "event": "onStop",
      "command": "afplay /System/Library/Sounds/Glass.aiff"
    },
    {
      "event": "onStop",
      "runCommand": "uv run hooks/log_agent_actions.py"
    }
  ],
  "customInstructions": "항상 한국어로 응답하세요."
}
```

#### 12.2.3 output-styles/personal-assistant.md

```markdown
# 개인 비서 - Kaya

당신은 Kaya라는 이름의 효율적인 개인 AI 비서입니다.

## 정체성
- 프로액티브하고 솔루션 지향적
- 사용자의 시간을 존중
- 전문적이지만 친근한 톤

## 커뮤니케이션
- 간결하고 명확한 응답
- 불필요한 서론 없이 바로 본론
- 중요 정보는 **굵게** 강조
- 필요시 단계별 가이드 제공

## 하위 에이전트 활용
다음 전문가들을 활용하세요:

- **youtube-analyst**: YouTube 채널 분석
  - 동영상 성과 메트릭
  - 댓글 감성 분석
  - 성장 추세 파악

- **researcher**: 웹 리서치 및 보고서
  - 여러 소스에서 정보 수집
  - 신뢰할 수 있는 출처 우선
  - 종합 보고서 작성

- **document-creator**: 문서 작성
  - 마크다운, DOCX, PDF
  - 전문적인 포맷팅
  - 구조화된 콘텐츠

## 작업 처리
- 복잡한 요청은 하위 에이전트에 위임
- 여러 작업은 병렬로 처리
- 완료 후 명확한 요약 제공
```

#### 12.2.4 cli_tools.py

```python
"""CLI 유틸리티 모듈"""

import argparse
from rich.console import Console
from rich.panel import Panel
from rich.syntax import Syntax
from rich import print as rprint

from claude_agent_sdk import (
    SystemMessage,
    AssistantMessage,
    UserMessage,
    ToolMessage,
    ResultMessage,
    TextBlock,
    ToolUseBlock,
    ThinkingBlock
)

console = Console()

# 메시지 타입별 색상
MESSAGE_STYLES = {
    "system": "bold cyan",
    "assistant": "bold green",
    "tool": "bold yellow",
    "thinking": "bold magenta",
    "user": "bold blue",
    "error": "bold red"
}

def parse_args():
    """명령줄 인수 파싱"""
    parser = argparse.ArgumentParser(
        description="Claude Agent SDK 예제"
    )
    
    parser.add_argument(
        "--model",
        type=str,
        default="claude-sonnet-4-20250514",
        help="사용할 모델 (기본: Sonnet 4.5)"
    )
    
    parser.add_argument(
        "--raw",
        action="store_true",
        help="원시 메시지 출력 (디버깅용)"
    )
    
    parser.add_argument(
        "--debug",
        action="store_true",
        help="디버그 모드 활성화"
    )
    
    parser.add_argument(
        "--no-hooks",
        action="store_true",
        help="후크 비활성화"
    )
    
    return parser.parse_args()

def print_message(message):
    """메시지 타입에 따라 적절하게 출력"""
    
    if isinstance(message, SystemMessage):
        print_system_message(message)
    
    elif isinstance(message, AssistantMessage):
        print_assistant_message(message)
    
    elif isinstance(message, UserMessage):
        rprint(f"[{MESSAGE_STYLES['user']}]👤 사용자:[/] {message.content}")
    
    elif isinstance(message, ToolMessage):
        print_tool_message(message)
    
    elif isinstance(message, ResultMessage):
        print_result_message(message)

def print_system_message(message):
    """시스템 메시지 출력"""
    data = message.data
    
    tools = data.get("tools", [])
    mcp_servers = data.get("mcp_servers", {})
    agents = data.get("agents", {})
    
    content = f"""
📋 도구: {len(tools)}개
🔌 MCP 서버: {len(mcp_servers)}개
👥 서브에이전트: {len(agents)}개
🤖 모델: {data.get('model')}
🔐 권한: {data.get('permission_mode')}
    """.strip()
    
    console.print(Panel(
        content,
        title="⚙️  시스템 구성",
        style=MESSAGE_STYLES["system"],
        border_style="cyan"
    ))

def print_assistant_message(message):
    """어시스턴트 메시지 출력"""
    for block in message.content:
        if isinstance(block, TextBlock):
            rprint(f"[{MESSAGE_STYLES['assistant']}]🤖 Claude:[/] {block.text}")
        
        elif isinstance(block, ToolUseBlock):
            tool_name = block.name
            tool_input = block.input
            
            # 입력이 긴 경우 축약
            if len(str(tool_input)) > 100:
                tool_input_str = str(tool_input)[:100] + "..."
            else:
                tool_input_str = str(tool_input)
            
            console.print(Panel(
                f"**도구:** {tool_name}\n**입력:** {tool_input_str}",
                title="🔧 도구 호출",
                style=MESSAGE_STYLES["tool"],
                border_style="yellow"
            ))
        
        elif isinstance(block, ThinkingBlock):
            # 사고 과정은 축약해서 표시
            thinking = block.thinking
            if len(thinking) > 200:
                thinking = thinking[:200] + "..."
            
            console.print(Panel(
                thinking,
                title="💭 사고 과정",
                style=MESSAGE_STYLES["thinking"],
                border_style="magenta"
            ))

def print_tool_message(message):
    """도구 메시지 출력"""
    is_error = message.is_error
    content = message.content
    
    # 결과가 긴 경우 축약
    if len(str(content)) > 200:
        content_str = str(content)[:200] + "..."
    else:
        content_str = str(content)
    
    style = MESSAGE_STYLES["error"] if is_error else MESSAGE_STYLES["tool"]
    title = "❌ 도구 오류" if is_error else "✅ 도구 결과"
    
    console.print(Panel(
        content_str,
        title=title,
        style=style,
        border_style="red" if is_error else "green"
    ))

def print_result_message(message):
    """결과 메시지 출력"""
    usage = message.usage
    cost = message.cost
    session_id = message.session_id
    
    content = f"""
🆔 세션: {session_id}
💰 비용: ${cost:.6f}
📊 입력 토큰: {usage.get('input_tokens', 0):,}
📊 출력 토큰: {usage.get('output_tokens', 0):,}
💾 캐시 생성: {usage.get('cache_creation_input_tokens', 0):,}
⚡ 캐시 읽기: {usage.get('cache_read_input_tokens', 0):,}
    """.strip()
    
    console.print(Panel(
        content,
        title="📈 실행 결과",
        style="bold blue",
        border_style="blue"
    ))

def print_code(code: str, language: str = "python"):
    """코드를 하이라이팅해서 출력"""
    syntax = Syntax(code, language, theme="monokai", line_numbers=True)
    console.print(syntax)
```

#### 12.2.5 main.py (완전한 개인 비서)

```python
"""
완전한 개인 비서 구현
"""

import json
from pathlib import Path
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AgentDefinition
)
from cli_tools import print_message, parse_args

# Playwright MCP 설정
playwright_mcp = {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-playwright"]
}

# YouTube 분석가
youtube_analyst = AgentDefinition(
    description="YouTube 채널 성과를 분석하는 전문가. "
                "동영상 메트릭, 댓글 감성, 성장 추세를 분석합니다.",
    prompt=Path("prompts/youtube_analyst.md").read_text(encoding="utf-8"),
    model="claude-sonnet-4-20250514",
    tools=[
        "mcp__playwright__playwright_navigate",
        "mcp__playwright__playwright_click",
        "mcp__playwright__playwright_screenshot",
        "mcp__playwright__playwright_fill",
        "view",
        "create_file",
        "str_replace"
    ]
)

# 리서처
researcher = AgentDefinition(
    description="웹 리서치를 수행하고 종합 보고서를 작성하는 전문가. "
                "여러 신뢰할 수 있는 소스에서 정보를 수집합니다.",
    prompt=Path("prompts/researcher.md").read_text(encoding="utf-8"),
    model="claude-sonnet-4-20250514",
    tools=[
        "web_search",
        "web_fetch",
        "view",
        "create_file",
        "str_replace",
        "bash"
    ]
)

async def main():
    """메인 함수"""
    args = parse_args()
    
    # 옵션 설정
    options = ClaudeAgentOptions(
        model=args.model,
        permission_mode="accept_edits",
        setting_sources=["project"],
        
        # 서브에이전트
        agents={
            "youtube-analyst": youtube_analyst,
            "researcher": researcher
            # document-creator는 .claude/agents/에서 로드
        },
        
        # MCP 서버
        mcp_servers={"playwright": playwright_mcp},
        
        # 허용 도구
        allowed_tools=[
            "bash",
            "view",
            "create_file",
            "str_replace",
            "web_search",
            "web_fetch",
            "mcp__playwright__playwright_navigate",
            "mcp__playwright__playwright_click",
            "mcp__playwright__playwright_screenshot",
            "mcp__playwright__playwright_fill",
            "mcp__playwright__playwright_evaluate"
        ]
    )
    
    # 클라이언트 시작
    async with ClaudeSDKClient(options=options) as client:
        print("="*60)
        print("🤖 개인 비서 Kaya입니다.")
        print("="*60)
        print("\n명령어:")
        print("  • 'exit' 또는 'quit': 종료")
        print("  • 'help': 도움말")
        print("  • 'demo': 데모 실행\n")
        
        while True:
            try:
                user_input = input("\n👤 당신: ").strip()
                
                if not user_input:
                    continue
                
                # 명령어 처리
                if user_input.lower() in ["exit", "quit", "종료"]:
                    print("\n👋 안녕히 가세요!")
                    break
                
                elif user_input.lower() == "help":
                    print_help()
                    continue
                
                elif user_input.lower() == "demo":
                    user_input = get_demo_query()
                    print(f"\n📝 데모 쿼리:\n{user_input}\n")
                
                # 쿼리 전송
                await client.query(user_input)
                
                # 응답 수신
                print()
                async for message in client.receive_response():
                    if args.raw:
                        print(message)
                    else:
                        print_message(message)
            
            except KeyboardInterrupt:
                print("\n\n⚠️  중단됨")
                break
            
            except Exception as e:
                print(f"\n❌ 오류: {e}")

def print_help():
    """도움말 출력"""
    help_text = """
📚 사용 가능한 기능:

1️⃣  YouTube 분석
   예: "내 최신 YouTube 동영상 분석해줘. 댓글도 확인해."

2️⃣  웹 리서치
   예: "2025년 AI 트렌드를 조사해줘."

3️⃣  문서 작성
   예: "Python 가상환경 사용법에 대한 가이드를 작성해줘."

4️⃣  복합 작업
   예: demo 명령 실행

💡 팁:
• 복잡한 작업은 여러 하위 에이전트가 병렬로 처리합니다.
• 작업 완료 후 outputs/ 폴더를 확인하세요.
• 로그는 logs/ 폴더에 저장됩니다.
    """
    print(help_text)

def get_demo_query():
    """데모 쿼리 반환"""
    return """
안녕, Kaya! 다음 세 가지 작업을 병렬로 처리해줘:

1. YouTube 분석: 내 최신 동영상의 성과를 분석하고, 
   조회수, 시청 시간, 구독자 증가를 파악해줘. 
   댓글도 확인해서 시청자 반응을 알려줘.

2. 리서치: 콘텐츠 크리에이터를 위한 최고의 AI 도구를 조사해줘.
   특히 동영상 편집, 썸네일 생성, 스크립트 작성 도구에 집중해.

3. 문서 작성: Claude Agent SDK에서 후크(hooks)를 구현하는 방법에 대한
   실용적인 가이드를 작성해줘. 공식 문서를 참고하고 코드 예제를 포함해.

각 작업은 별도의 마크다운 파일로 저장해줘.
    """

if __name__ == "__main__":
    import anyio
    anyio.run(main)
```

### 12.3 의존성 관리

#### 12.3.1 pyproject.toml

```toml
[project]
name = "claude-agent-sdk-intro"
version = "1.0.0"
description = "Claude Agent SDK 튜토리얼"
requires-python = ">=3.10"

dependencies = [
    "claude-agent-sdk>=1.0.0",
    "rich>=13.0.0",
    "python-dotenv>=1.0.0"
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
    "pytest>=7.0.0",
    "black>=23.0.0",
    "mypy>=1.0.0"
]
```

### 12.4 실행 흐름 다이어그램

```
사용자 입력
    │
    ▼
main.py 실행
    │
    ├─→ 인수 파싱 (cli_tools.parse_args)
    │
    ├─→ 옵션 설정 (ClaudeAgentOptions)
    │   ├─ 모델 선택
    │   ├─ 권한 설정
    │   ├─ 서브에이전트 정의
    │   ├─ MCP 서버 추가
    │   └─ 설정 로드 (.claude/)
    │
    ├─→ 클라이언트 생성 (ClaudeSDKClient)
    │   │
    │   └─→ 대화 루프 시작
    │       │
    │       ├─→ 사용자 입력 수신
    │       │
    │       ├─→ 쿼리 전송 (client.query)
    │       │   │
    │       │   └─→ 에이전트 루프 (SDK 내부)
    │       │       ├─ 컨텍스트 수집
    │       │       ├─ 계획 수립
    │       │       ├─ 서브에이전트 호출 (병렬)
    │       │       ├─ 도구 실행
    │       │       └─ 응답 생성
    │       │
    │       ├─→ 응답 수신 (client.receive_response)
    │       │   │
    │       │   └─→ 메시지 파싱 및 출력
    │       │       ├─ SystemMessage
    │       │       ├─ AssistantMessage
    │       │       ├─ ToolMessage
    │       │       └─ ResultMessage
    │       │
    │       └─→ 후크 실행 (onStop)
    │           ├─ 사운드 재생
    │           └─ 로그 저장
    │
    └─→ 종료
```

---

## 13. 실전 예제: 개인 비서 구축

### 13.1 프로젝트 목표

**완전한 멀티 에이전트 개인 비서 "Kaya" 구축:**
- YouTube 채널 분석 및 관리
- 웹 리서치 및 보고서 작성
- 전문 문서 생성
- 병렬 작업 처리

### 13.2 단계별 구현

#### 단계 1: 기본 설정

```bash
# 프로젝트 생성 및 설정
mkdir my-personal-assistant
cd my-personal-assistant

# 가상 환경 초기화
uv init
uv add claude-agent-sdk rich python-dotenv

# 디렉토리 구조 생성
mkdir -p .claude/agents output-styles hooks logs prompts outputs db
```

#### 단계 2: 프로젝트 컨텍스트 작성

**CLAUDE.md:**

```markdown
# 개인 비서 프로젝트

이 프로젝트는 나의 일상 업무를 돕는 AI 비서입니다.

## 주요 업무
- YouTube 채널 관리 및 분석
- 콘텐츠 아이디어 리서치
- 문서 작성 자동화

## 코딩 규칙
- Python 3.10+
- async/await 패턴
- 타입 힌트 사용
- 명확한 docstring

## 파일 구조
- `main.py`: 메인 비서
- `.claude/agents/`: 서브에이전트
- `prompts/`: 에이전트 프롬프트
- `outputs/`: 생성된 결과물
```

#### 단계 3: 출력 스타일 정의

**output-styles/personal-assistant.md:**

```markdown
# 개인 비서 - Kaya

당신은 효율적이고 프로액티브한 개인 비서 Kaya입니다.

## 정체성
- 사용자의 시간을 최우선으로 생각
- 명확하고 실행 가능한 정보 제공
- 전문적이지만 친근한 톤

## 작업 처리 원칙
1. **이해**: 사용자의 의도를 정확히 파악
2. **계획**: 최적의 접근 방법 결정
3. **실행**: 적절한 에이전트 활용
4. **보고**: 명확한 결과 요약

## 하위 에이전트
- **youtube-analyst**: YouTube 분석
- **researcher**: 웹 리서치
- **document-creator**: 문서 작성

## 커뮤니케이션
- 간결하고 명확
- 중요 정보 강조
- 필요시 단계별 가이드
```

#### 단계 4: 서브에이전트 프롬프트 작성

**prompts/youtube_analyst.md:**

```markdown
# YouTube 분석 전문가

당신은 YouTube 채널 성과를 분석하는 전문가입니다.

## 전문 분야
- YouTube Analytics 데이터 해석
- 동영상 성과 메트릭 분석
- 댓글 감성 분석 및 인사이트 추출
- 구독자 성장 추세 및 패턴 파악
- 경쟁사 분석

## 분석 프로세스

### 1. 데이터 수집
- Playwright를 사용해 YouTube Studio 접근
- 최신 동영상 메트릭 수집:
  - 조회수, 시청 시간, CTR
  - 좋아요/싫어요 비율
  - 구독자 증감
  - 트래픽 소스
- 상위 댓글 (최소 20개) 수집

### 2. 메트릭 분석
- 핵심 지표 추세 파악
- 이전 동영상과 비교
- 채널 평균 대비 성과
- 이상치 및 패턴 식별

### 3. 댓글 분석
- 감성 분석 (긍정/중립/부정)
- 주요 주제 및 키워드 추출
- 시청자 질문 및 요청 파악
- 개선 제안 식별

### 4. 인사이트 도출
- 성공 요인 분석
- 개선 기회 식별
- 다음 콘텐츠 아이디어
- 실행 가능한 권장사항

## 출력 형식

마크다운 보고서를 다음 구조로 작성하세요:

```markdown
# YouTube 분석 보고서
날짜: [YYYY-MM-DD]

## 📊 요약
- 주요 메트릭 하이라이트
- 핵심 인사이트 (3-5개)

## 📈 성과 메트릭
### 최신 동영상: [제목]
- **조회수**: X,XXX (채널 평균 대비 ±X%)
- **시청 시간**: XX분 (평균 시청 지속 시간: XX%)
- **좋아요 비율**: X.X%
- **댓글**: XXX개
- **구독자 증감**: +XXX

### 트래픽 소스
- YouTube 검색: XX%
- 추천: XX%
- 외부: XX%

## 💬 댓글 분석
### 감성 분석
- 긍정: XX%
- 중립: XX%
- 부정: XX%

### 주요 주제
1. [주제 1]: [설명]
2. [주제 2]: [설명]
...

### 시청자 질문/요청
- [질문/요청 1]
- [질문/요청 2]
...

## 💡 인사이트 및 권장사항
1. **[인사이트 제목]**
   - 관찰: [내용]
   - 권장사항: [실행 가능한 조언]

2. ...

## 🎯 다음 콘텐츠 아이디어
1. [아이디어 1]: [근거]
2. [아이디어 2]: [근거]
...
```

## 중요 사항
- 모든 주장은 데이터로 뒷받침
- 실행 가능한 권장사항 제시
- 명확하고 구조화된 형식
- 긍정적이고 건설적인 톤 유지
```

**prompts/researcher.md:**

```markdown
# 웹 리서치 전문가

당신은 종합적인 웹 리서치를 수행하는 전문가입니다.

## 전문 분야
- 다양한 소스에서 정보 수집
- 신뢰할 수 있는 출처 식별
- 데이터 교차 검증
- 종합 보고서 작성

## 리서치 방법론

### 1. 주제 분석
- 핵심 키워드 식별
- 리서치 범위 정의
- 검색 쿼리 최적화

### 2. 정보 수집
- 최소 5-10개의 다양한 소스 검색
- 각 소스의 신뢰도 평가:
  - 공식 문서 (우선)
  - 학술 논문 및 연구
  - 업계 리더 블로그
  - 뉴스 기사
  - 커뮤니티 포럼 (참고용)

### 3. 정보 검증
- 여러 소스 간 교차 확인
- 최신성 확인 (날짜 확인)
- 편향성 식별
- 사실과 의견 구분

### 4. 종합 및 분석
- 핵심 인사이트 추출
- 패턴 및 추세 파악
- 상반되는 정보 조정
- 실행 가능한 결론 도출

## 출력 형식

마크다운 보고서를 다음 구조로 작성하세요:

```markdown
# 리서치 보고서: [주제]
날짜: [YYYY-MM-DD]

## 🎯 목차
1. 요약
2. [주요 섹션 1]
3. [주요 섹션 2]
...
8. 결론 및 권장사항
9. 참고문헌

## 📝 요약 (Executive Summary)
- 리서치 목적
- 핵심 발견 (3-5개)
- 주요 권장사항

## [주요 섹션 1]
### 개요
[섹션 설명]

### 주요 발견
1. **[발견 1]** [1]
   - [상세 설명]
   - [데이터/증거]

2. **[발견 2]** [2]
   - [상세 설명]

...

## [추가 섹션들...]

## 💡 결론 및 권장사항
### 핵심 인사이트
1. [인사이트 1]
2. [인사이트 2]

### 권장사항
1. **[권장사항 1]**
   - 이유: [설명]
   - 실행 방법: [단계]

2. ...

## 📚 참고문헌
[1] [저자/출처]. [제목]. [URL] (날짜)
[2] ...
```

## 품질 기준
- **정확성**: 모든 정보 검증
- **신뢰성**: 권위 있는 출처 우선
- **포괄성**: 다양한 관점 포함
- **명확성**: 이해하기 쉬운 언어
- **실용성**: 실행 가능한 인사이트

## 인용 규칙
- 모든 주장에 출처 표시 [숫자]
- URL 및 접근 날짜 포함
- 직접 인용 시 따옴표 사용
```

#### 단계 5: 문서 작성 에이전트

**.claude/agents/document-creator.md:**

```markdown
---
description: 전문적인 문서를 작성하는 전문가
model: claude-sonnet-4-20250514
tools:
  - view
  - create_file
  - str_replace
  - bash
---

# 문서 작성 전문가

당신은 다양한 형식의 전문 문서를 작성하는 전문가입니다.

## 전문 분야
- 기술 문서 및 가이드
- 튜토리얼 및 하우투
- 보고서 및 백서
- 프레젠테이션 자료

## 지원 형식
- **마크다운** (.md): 웹 문서, README, 블로그
- **Word** (.docx): 공식 보고서, 제안서
- **PDF**: 최종 배포본
- **PowerPoint** (.pptx): 프레젠테이션

## 문서 작성 프로세스

### 1. 요구사항 분석
- 문서 목적 및 대상 독자 파악
- 필요한 정보 및 리소스 확인
- 적절한 형식 및 구조 결정

### 2. 개요 작성
- 논리적 흐름 설계
- 주요 섹션 및 하위 섹션 정의
- 필요한 시각 자료 식별

### 3. 콘텐츠 작성
- 명확하고 간결한 문장
- 적절한 기술 수준 유지
- 예제 및 사례 포함
- 시각 자료 통합

### 4. 검토 및 개선
- 논리적 일관성 확인
- 문법 및 맞춤법 검사
- 포맷팅 최적화

## 문서 구조 템플릿

### 기술 가이드
```markdown
# [제목]

## 📋 개요
- 이 가이드의 목적
- 대상 독자
- 사전 요구사항

## 🎯 목차
[자동 생성]

## 소개
[배경 설명]

## [주요 섹션 1]
### [하위 섹션 1.1]
[내용]

### [하위 섹션 1.2]
[내용]

## [주요 섹션 2]
...

## 💡 모범 사례
1. [사례 1]
2. [사례 2]

## ⚠️ 일반적인 문제
### [문제 1]
**증상**: [설명]
**해결책**: [단계별 해결 방법]

## 🔗 참고 자료
- [링크 1]
- [링크 2]

## 결론
[요약 및 다음 단계]
```

## 스타일 가이드

### 제목 계층
- H1 (#): 문서 제목
- H2 (##): 주요 섹션
- H3 (###): 하위 섹션
- H4 (####): 세부 항목

### 강조
- **굵게**: 중요 용어, 핵심 개념
- *기울임*: 강조, 새로운 용어
- `코드`: 파일명, 명령어, 코드 조각

### 목록
- 순서 있는 목록: 단계, 순위
- 순서 없는 목록: 항목, 기능

### 코드 블록
```언어
코드 내용
```

### 표
명확한 헤더와 정렬 사용

## 품질 기준
- ✅ 명확하고 이해하기 쉬움
- ✅ 구조화되고 논리적
- ✅ 시각적으로 매력적
- ✅ 실행 가능한 정보 제공
- ✅ 오류 없음 (문법, 코드 등)
```

#### 단계 6: 설정 파일

**.claude/settings.json:**

```json
{
  "outputStyle": "personal-assistant",
  "hooks": [
    {
      "event": "onStop",
      "command": "afplay /System/Library/Sounds/Glass.aiff"
    },
    {
      "event": "onStop",
      "runCommand": "uv run hooks/log_agent_actions.py"
    }
  ]
}
```

#### 단계 7: 로깅 후크

**hooks/log_agent_actions.py:**

```python
"""에이전트 행동을 로그 파일로 저장"""

import json
import sys
from datetime import datetime
from pathlib import Path

def parse_stdin_messages():
    """stdin에서 메시지 파싱"""
    messages = []
    for line in sys.stdin:
        line = line.strip()
        if not line:
            continue
        try:
            msg = json.loads(line)
            messages.append(msg)
        except json.JSONDecodeError:
            pass
    return messages

def extract_actions(messages):
    """메시지에서 에이전트 행동 추출"""
    actions = []
    
    for msg in messages:
        msg_type = msg.get("type")
        
        if msg_type == "tool_use":
            actions.append({
                "timestamp": datetime.now().isoformat(),
                "type": "tool_call",
                "tool": msg.get("name"),
                "input": msg.get("input")
            })
        
        elif msg_type == "tool_result":
            actions.append({
                "timestamp": datetime.now().isoformat(),
                "type": "tool_result",
                "status": "error" if msg.get("is_error") else "success",
                "content": str(msg.get("content", ""))[:200]  # 축약
            })
        
        elif msg_type == "text":
            # 어시스턴트 응답 요약
            text = msg.get("text", "")
            if text and len(text) > 50:
                actions.append({
                    "timestamp": datetime.now().isoformat(),
                    "type": "assistant_response",
                    "preview": text[:100] + "..."
                })
    
    return actions

def save_log(actions):
    """로그 파일 저장"""
    log_dir = Path("logs")
    log_dir.mkdir(exist_ok=True)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    log_file = log_dir / f"agent_actions_{timestamp}.json"
    
    log_data = {
        "timestamp": datetime.now().isoformat(),
        "action_count": len(actions),
        "actions": actions
    }
    
    with open(log_file, "w", encoding="utf-8") as f:
        json.dump(log_data, f, indent=2, ensure_ascii=False)
    
    print(f"✅ 로그 저장: {log_file}", file=sys.stderr)

def main():
    messages = parse_stdin_messages()
    actions = extract_actions(messages)
    save_log(actions)

if __name__ == "__main__":
    main()
```

#### 단계 8: 메인 비서 구현

이미 위의 12.2.5에서 제공했습니다.

### 13.3 실행 및 테스트

```bash
# 데모 실행
uv run main.py

# 프롬프트에서
👤 당신: demo

# 또는 직접 쿼리
👤 당신: 내 YouTube 채널을 분석하고, 
         최근 AI 트렌드를 리서치한 후,
         발견한 내용을 바탕으로 다음 동영상 
         아이디어 10개를 문서로 만들어줘.
```

### 13.4 결과물

작업 완료 후 다음 파일들이 생성됩니다:

```
outputs/
├── youtube_analysis_20250110.md      # YouTube 분석 보고서
├── ai_trends_research_20250110.md    # AI 트렌드 리서치
└── video_ideas_20250110.md           # 동영상 아이디어 문서

logs/
├── conversations/
│   └── conversation_20250110_143000.json
└── agent_actions_20250110_143000.json
```

---

## 14. 모범 사례 및 팁

### 14.1 에이전트 설계 원칙

#### 14.1.1 단일 책임 원칙

각 서브에이전트는 하나의 명확한 역할만 담당해야 합니다:

✅ **좋은 예:**
```python
youtube_analyst = AgentDefinition(
    description="YouTube 채널 성과 분석 전문가",
    # 하나의 명확한 역할
)

researcher = AgentDefinition(
    description="웹 리서치 및 보고서 작성 전문가",
    # 다른 명확한 역할
)
```

❌ **나쁜 예:**
```python
swiss_army_knife = AgentDefinition(
    description="YouTube 분석, 리서치, 문서 작성, 코딩 등 모든 것을 하는 에이전트",
    # 너무 많은 책임
)
```

#### 14.1.2 적절한 도구 할당

에이전트에게 필요한 도구만 할당:

```python
# YouTube 분석가는 브라우저만 필요
youtube_analyst.tools = [
    "mcp__playwright__playwright_navigate",
    "mcp__playwright__playwright_screenshot",
    "view",
    "create_file"
]

# 웹 검색이나 코드 실행은 필요 없음
```

#### 14.1.3 명확한 프롬프트

프롬프트는 구체적이고 구조화되어야 합니다:

```markdown
# 좋은 프롬프트
## 역할
당신은 YouTube 분석 전문가입니다.

## 전문 분야
- YouTube Analytics 데이터 해석
- 동영상 성과 메트릭 분석
- ...

## 작업 프로세스
1. 데이터 수집
2. 분석
3. 인사이트 도출
4. 보고서 작성

## 출력 형식
[구체적인 템플릿]
```

### 14.2 성능 최적화

#### 14.2.1 프롬프트 캐싱 활용

Claude Agent SDK는 자동 프롬프트 캐싱을 제공합니다.

```python
# ClaudeSDKClient 사용 시 자동 캐싱
async with ClaudeSDKClient(options=options) as client:
    # 첫 쿼리: 전체 비용
    await client.query("첫 번째 질문")
    
    # 두 번째 쿼리: 캐시된 입력 사용 → 약 90% 비용 절감
    await client.query("두 번째 질문")
```

#### 14.2.2 병렬 처리 극대화

여러 독립적인 작업은 병렬로 처리:

```python
# 좋은 쿼리: 병렬 처리 가능
query = """
1. YouTube 분석해줘
2. 최근 AI 트렌드 리서치해줘
3. 문서 작성해줘
"""

# 세 가지 작업이 동시에 실행됨
```

#### 14.2.3 적절한 모델 선택

```python
# 간단한 작업: Haiku (빠르고 저렴)
simple_options = ClaudeAgentOptions(
    model="claude-haiku-4-20250514"
)

# 복잡한 작업: Sonnet (균형)
complex_options = ClaudeAgentOptions(
    model="claude-sonnet-4-20250514"  # 권장
)

# 매우 복잡한 작업: Opus (가장 강력)
critical_options = ClaudeAgentOptions(
    model="claude-opus-4-20250514"
)
```

### 14.3 디버깅 및 모니터링

#### 14.3.1 로깅 활용

```python
# hooks/log_agent_actions.py 사용
# settings.json에서 설정

# 로그 파일 확인
# logs/agent_actions_*.json
```

#### 14.3.2 원시 메시지 출력

```bash
# 디버깅 모드
uv run main.py --raw --debug
```

#### 14.3.3 SystemMessage 확인

```python
async for message in client.receive_response():
    if isinstance(message, SystemMessage):
        print("사용 가능한 도구:")
        for tool in message.data["tools"]:
            print(f"  - {tool['name']}")
        
        print("\nMCP 서버:")
        for name, config in message.data["mcp_servers"].items():
            print(f"  - {name}")
        
        print("\n서브에이전트:")
        for name, agent in message.data["agents"].items():
            print(f"  - {name}: {agent.description}")
```

### 14.4 비용 관리

#### 14.4.1 ResultMessage 모니터링

```python
if isinstance(message, ResultMessage):
    print(f"세션 비용: ${message.cost:.6f}")
    print(f"입력 토큰: {message.usage['input_tokens']:,}")
    print(f"출력 토큰: {message.usage['output_tokens']:,}")
    print(f"캐시 사용: {message.usage.get('cache_read_input_tokens', 0):,}")
```

#### 14.4.2 비용 절감 팁

1. **ClaudeSDKClient 사용**: 캐싱으로 최대 90% 절감
2. **적절한 모델 선택**: Haiku → Sonnet → Opus
3. **간결한 프롬프트**: 불필요한 토큰 제거
4. **도구 최소화**: 필요한 도구만 허용

### 14.5 보안 및 안전

#### 14.5.1 권한 관리

```python
options = ClaudeAgentOptions(
    # 프로덕션: accept_edits
    permission_mode="accept_edits",
    
    # 개발/테스트: default (수동 승인)
    # permission_mode="default",
    
    # 민감한 작업: 특정 도구 차단
    disallowed_tools=[
        "bash",  # 쉘 명령 실행 금지
        "web_fetch"  # 외부 URL 접근 금지
    ]
)
```

#### 14.5.2 디렉토리 접근 제한

```python
options = ClaudeAgentOptions(
    # 현재 프로젝트만 접근 가능 (기본)
    # add_directories를 지정하지 않음
    
    # 또는 특정 디렉토리만 허용
    add_directories=[
        "/path/to/safe/directory"
    ]
)
```

#### 14.5.3 API 키 보안

```bash
# .env 파일 사용
ANTHROPIC_API_KEY=sk-ant-...

# .gitignore에 추가
echo ".env" >> .gitignore
```

### 14.6 일반적인 문제 해결

#### 문제 1: MCP 도구가 실행되지 않음

**원인**: 도구를 허용하지 않음

**해결:**
```python
options = ClaudeAgentOptions(
    allowed_tools=[
        "mcp__서버명__도구명"  # 명시적 허용 필요
    ]
)
```

#### 문제 2: 서브에이전트가 도구 사용 못함

**원인**: 서브에이전트의 도구가 메인 에이전트에서 허용되지 않음

**해결:**
```python
options = ClaudeAgentOptions(
    agents={"sub": sub_agent},
    
    # 서브에이전트의 모든 도구를 메인에서도 허용
    allowed_tools=[
        *sub_agent.tools,  # 서브에이전트 도구
        "bash",            # 메인 에이전트 도구
        "view"
    ]
)
```

#### 문제 3: 설정이 로드되지 않음

**원인**: `setting_sources` 미지정

**해결:**
```python
options = ClaudeAgentOptions(
    setting_sources=["project"]  # 명시적으로 지정
)
```

#### 문제 4: 높은 비용

**원인**: 캐싱 미활용, 비효율적인 프롬프트

**해결:**
```python
# query() 대신 ClaudeSDKClient 사용
async with ClaudeSDKClient(options=options) as client:
    # 여러 쿼리에서 캐싱 활용
    ...

# 간결한 프롬프트 작성
# 불필요한 상세 설명 제거
```

### 14.7 추가 리소스

**공식 문서:**
- [Claude Agent SDK 개요](https://docs.claude.com/en/api/agent-sdk/overview)
- [Python SDK GitHub](https://github.com/anthropics/claude-agent-sdk-python)
- [MCP 문서](https://docs.claude.com/en/docs/claude-code/mcp)

**커뮤니티:**
- [Awesome Claude Code Subagents](https://github.com/ckreiling/awesome-claude-code-subagents)
- Claude Discord 커뮤니티
- Reddit: r/ClaudeAI

**튜토리얼:**
- [DataCamp: Claude Agent SDK Tutorial](https://www.datacamp.com/tutorial/how-to-use-claude-agent-sdk)
- [Anthropic 블로그: Building Agents](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk)

---

## 마무리

이 가이드에서 Claude Agent SDK를 사용하여 멀티 에이전트 개인 비서를 구축하는 방법을 단계별로 살펴보았습니다. 핵심 내용을 요약하면:

### 주요 학습 내용

1. **Claude Agent SDK 기초**
   - Claude Code 기반의 프로덕션 레벨 에이전트 프레임워크
   - 자동 컨텍스트 관리 및 최적화
   - 풍부한 도구 생태계

2. **에이전트 구성**
   - `ClaudeAgentOptions`로 세밀한 제어
   - 권한 관리 및 도구 허용
   - 출력 스타일 커스터마이징

3. **커스텀 도구**
   - `@tool` 데코레이터로 쉬운 정의
   - In-process MCP 서버로 구현
   - 명시적 권한 부여 필요

4. **서브에이전트**
   - 전문화된 역할 분담
   - 병렬 처리로 효율 극대화
   - 독립적인 컨텍스트 및 도구 세트

5. **실전 구현**
   - 프로젝트 구조 설계
   - 프롬프트 엔지니어링
   - 모니터링 및 디버깅

### 다음 단계

이제 여러분만의 AI 에이전트를 구축할 준비가 되었습니다:

1. **간단하게 시작**: 기본 쿼리부터 시작
2. **점진적 확장**: 도구와 서브에이전트 추가
3. **최적화**: 성능과 비용 모니터링
4. **커스터마이징**: 자신의 워크플로우에 맞게 조정

### 기억할 점

- 🎯 **명확한 역할**: 각 에이전트는 하나의 명확한 책임
- 🔧 **적절한 도구**: 필요한 도구만 할당
- ⚡ **병렬 처리**: 독립적인 작업은 동시에 수행
- 💰 **비용 관리**: 캐싱 활용 및 적절한 모델 선택
- 🔐 **보안**: 권한 관리 및 접근 제한

Claude Agent SDK로 여러분의 생산성을 새로운 수준으로 끌어올리세요! 🚀



