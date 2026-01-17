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
