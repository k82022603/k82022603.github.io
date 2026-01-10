---
title: "MCP, Skill, Agent 완전 가이드"
date: 2026-01-10 08:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  MCP,  agent-skills,  sub-agents,  agentic-engineers,  Claude.write]
---

## 누가 결정하는가: 신뢰할 수 있는 AI 시스템 설계

---

## 서문: 복잡함 속의 단순한 진실

AI 도구를 실무에 도입하려고 할 때, 개발자들은 종종 혼란스러운 용어의 홍수에 빠집니다. MCP, Skill, Agent, Workflow, Tool, Function Calling... 이 모든 것들이 어떻게 연결되는지, 언제 무엇을 써야 하는지 명확하지 않습니다.

하지만 이 복잡함 속에는 놀랍도록 단순한 구조가 숨어 있습니다. MCP, Skill, Agent를 한 줄로 세우면 전체 그림이 보입니다. 아래에서 위로, 연결에서 기능으로, 기능에서 자율 실행으로 올라가는 계층 구조입니다. MCP는 외부 세계와의 표준 연결 규격이고, Skill은 그 연결을 활용해 특정 업무를 완수하는 정해진 레시피이며, Agent는 목표만 주어지면 스스로 계획을 세우고 도구를 선택하며 반복을 통해 문제를 해결하는 자율 실행자입니다.

이 가이드는 이 세 가지 개념을 실무 관점에서 완전히 이해하고, 언제 무엇을 사용해야 하는지 명확하게 판단할 수 있도록 돕습니다. 우리는 "데모용 똑똑함"이 아니라 "운영에서 살아남는 신뢰"를 만드는 방법에 집중할 것입니다.

---

## 1부: 핵심 개념의 본질

### 누가 결정하는가

MCP, Skill, Agent의 핵심 차이를 이해하려면, 단 하나의 질문에 답하면 됩니다. "누가 결정하는가?" 이 질문은 시스템의 제어권이 어디에 있는지, 예측 가능성이 어느 정도인지, 그리고 위험 수준이 얼마나 되는지를 정확히 보여줍니다.

Skill을 사용할 때, 결정권은 완전히 사람에게 있습니다. 개발자나 설계자가 모든 흐름을 미리 정의합니다. 웹 검색을 하고, 결과를 요약하고, 정해진 포맷으로 보고서를 만들고, 특정 채널에 알림을 보내는 모든 단계를 사람이 설계합니다. AI 모델은 그저 그 길을 "잘" 걷는 역할만 합니다. 검색 쿼리를 잘 만들고, 요약을 잘하고, 보고서를 읽기 좋게 작성하는 것은 모델의 몫이지만, 어떤 순서로 무엇을 할지는 이미 정해져 있습니다.

이것이 Skill의 가장 큰 장점입니다. 예측 가능하고, 감사 가능하며, 운영에서 강합니다. 실무에서 필요한 것은 대개 "똑똑함"보다 "신뢰"입니다. 매일 아침 정확히 같은 방식으로 보고서가 생성되고, 실패했을 때 정확히 어느 단계에서 왜 실패했는지 알 수 있으며, 다른 팀원이 와도 같은 결과를 재현할 수 있다는 것, 이것이 운영 환경에서 가장 중요한 가치입니다.

반면 Agent를 사용할 때는, 사람이 목표와 제약만 줍니다. "이번 주 발생한 장애의 원인을 찾고 재발 방지안을 마련하라"처럼 추상적인 목표를 제시하면, Agent가 스스로 계획을 세웁니다. 로그를 어떤 순서로 확인할지, 어떤 메트릭을 먼저 볼지, 코드의 어느 부분을 집중적으로 분석할지를 AI가 결정합니다. 그리고 중간 결과를 보면서 계획을 수정합니다. 처음에는 네트워크 이슈를 의심했다가, 로그를 보니 데이터베이스 락 문제로 판단이 바뀌고, 다시 코드를 보니 트랜잭션 설계 문제였다는 식으로 추론이 진화합니다.

Agent는 더 강력합니다. 예측하지 못한 상황에서도 스스로 대응하고, 복잡한 문제를 여러 단계로 분해하며, 실패하면 다른 접근을 시도합니다. 하지만 그만큼 통제, 비용, 안정성을 설계하지 않으면 위험해집니다. Agent가 예상치 못한 API를 수십 번 호출해서 요금 폭탄을 맞을 수도 있고, 중요한 데이터를 삭제하는 도구를 잘못 사용할 수도 있으며, 무한 루프에 빠져서 리소스를 고갈시킬 수도 있습니다. 강한 만큼, 경계도 더 필요합니다.

MCP는 이 모든 것의 기반입니다. Skill이든 Agent든, 외부 도구를 사용하려면 표준화된 방법이 필요합니다. MCP는 바로 그 표준 연결 규격입니다. 데이터베이스를 쿼리하든, Slack에 메시지를 보내든, GitHub에서 이슈를 조회하든, 모두 MCP 프로토콜을 통해 일관된 방식으로 호출됩니다. 이것이 운영 가능성의 바닥입니다.

### 계층 구조의 실체

이제 이 세 가지를 계층으로 시각화해봅시다. 가장 아래에 MCP 레이어가 있습니다. 이것은 연결 계층입니다. 수십 개의 외부 도구와 서비스가 MCP 서버로 표준화되어 있습니다. Slack MCP 서버, GitHub MCP 서버, PostgreSQL MCP 서버, Google Drive MCP 서버 등이 모두 같은 프로토콜로 통신합니다.

그 위에 Skill 레이어가 있습니다. 이것은 기능 계층입니다. MCP 도구들을 조합해서 의미 있는 업무를 완성합니다. "주간 보고서 생성" Skill은 Jira MCP를 통해 이슈를 가져오고, Slack MCP를 통해 대화 히스토리를 분석하며, Google Docs MCP를 통해 문서를 생성합니다. 모든 단계가 명시적으로 정의되어 있습니다.

맨 위에 Agent 레이어가 있습니다. 이것은 자율 실행 계층입니다. Agent는 아래의 Skill들을 "고수준 도구"처럼 사용하거나, MCP 도구를 직접 호출하면서 목표를 달성합니다. "이번 분기 매출이 왜 하락했는지 분석하라"는 목표를 받으면, Agent가 스스로 "먼저 매출 데이터를 조회하고, 그다음 주요 고객사별로 분해하고, 고객사 담당자와의 대화 기록을 확인하고, 경쟁사 동향을 웹에서 검색하고..."처럼 계획을 세워 실행합니다.

현실에서는 대개 이 세 가지를 조합합니다. 선택의 문제가 아니라 조합의 문제입니다. 먼저 MCP로 필요한 모든 도구를 표준화해서 연결합니다. 그 위에 Skill로 반복되는 안정적인 업무를 자동화합니다. 매일 아침 리포트, 배포 전 체크리스트, 장애 발생 시 초기 대응 같은 것들입니다. 그리고 예외가 많고 탐색이 필요한 영역, 정해진 절차로는 해결할 수 없는 복잡한 문제만 Agent에게 맡깁니다.

---

## 2부: MCP - 운영 가능성의 기반

### MCP가 해결하는 근본 문제

AI 모델이 외부 세계와 상호작용하려면 도구가 필요합니다. 파일을 읽고, API를 호출하고, 데이터베이스를 쿼리하고, 메시지를 보내야 합니다. 초기에는 각 도구마다 다른 방식으로 통합되었습니다. OpenAI Function Calling은 JSON 스키마를 쓰고, LangChain Tools는 Python 클래스를 쓰고, 각 프레임워크마다 고유한 방식이 있었습니다.

이것은 엄청난 비효율을 만들었습니다. Slack 통합을 한 번 만들었는데, Claude에서 쓰려면 다시 만들어야 하고, GPT에서 쓰려면 또 다시 만들어야 했습니다. 더 심각한 문제는 표준이 없으니 보안, 권한, 감사, 에러 처리를 각자 다르게 구현한다는 것이었습니다. 한 도구는 API 키를 환경 변수로 받고, 다른 도구는 설정 파일로 받고, 또 다른 도구는 코드에 하드코딩되어 있었습니다.

MCP(Model Context Protocol)는 이 문제를 해결하기 위한 표준입니다. Anthropic이 만들었지만, 오픈 프로토콜이라 누구나 구현할 수 있습니다. 핵심 아이디어는 간단합니다. 모든 도구를 "MCP 서버"로 패키징하고, 모든 AI 시스템(Claude, GPT, Gemini 등)이 "MCP 클라이언트"로 이 서버들과 통신하는 것입니다.

### MCP의 구조

MCP 아키텍처는 세 개의 레이어로 구성됩니다. 가장 아래에 MCP Server가 있습니다. 이것은 실제 도구의 기능을 제공합니다. Slack MCP Server는 Slack API를 래핑하고, GitHub MCP Server는 GitHub API를 래핑합니다. 각 서버는 표준 MCP 프로토콜로 통신하지만, 내부적으로는 각자의 방식으로 외부 서비스와 연결됩니다.

중간에 MCP Host가 있습니다. 이것은 여러 MCP 서버들을 관리하는 오케스트레이터입니다. Claude Desktop, Cursor, Cline 같은 애플리케이션이 MCP Host 역할을 합니다. Host는 사용자가 설정한 서버 목록을 읽어서, 각 서버를 시작하고, 서버가 제공하는 도구 목록을 가져오고, AI 모델에게 "이런 도구들을 사용할 수 있어"라고 알려줍니다.

맨 위에 AI Model이 있습니다. Claude, GPT, Gemini 등이 여기에 해당합니다. 모델은 Host를 통해 "어떤 도구들이 있어?"라고 물어보고, "Slack에 메시지 보내줘"처럼 도구 호출을 요청합니다. Host는 이 요청을 적절한 MCP Server로 라우팅하고, 서버의 응답을 다시 모델에게 전달합니다.

실제 통신은 JSON-RPC 2.0 프로토콜로 이루어집니다. 매우 간단한 예를 보겠습니다. AI 모델이 Slack에 메시지를 보내고 싶다면, Host에게 이렇게 요청합니다:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "slack_send_message",
    "arguments": {
      "channel": "#dev-team",
      "text": "Deploy completed successfully!"
    }
  },
  "id": 1
}
```

Host는 이것을 Slack MCP Server로 전달하고, 서버는 실제 Slack API를 호출한 후 결과를 반환합니다:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Message sent to #dev-team at 2026-01-10T10:30:00Z"
      }
    ]
  },
  "id": 1
}
```

이 표준화 덕분에 한 번 만든 MCP Server는 모든 AI 시스템에서 재사용됩니다. Slack MCP Server를 만들면, Claude에서도 쓰고, GPT에서도 쓰고, 자체 제작한 AI 시스템에서도 씁니다. 통합 비용이 O(N×M)에서 O(N+M)으로 줄어듭니다. N개의 도구와 M개의 AI 시스템을 연결하는데, 예전에는 N×M개의 통합을 만들어야 했지만, 이제는 N개의 MCP Server와 M개의 MCP Client만 만들면 됩니다.

### MCP의 세 가지 핵심 기능

MCP는 세 가지 주요 기능을 제공합니다. 첫째는 Tools입니다. 이것이 가장 기본적인 기능입니다. 외부 작업을 수행하는 함수들입니다. `slack_send_message`, `github_create_issue`, `postgres_query` 같은 것들입니다. 각 도구는 명확한 입력 스키마와 출력 스키마를 가집니다.

둘째는 Resources입니다. 이것은 AI가 읽을 수 있는 데이터 소스입니다. 파일 내용, 데이터베이스 레코드, API 응답 등을 표준화된 형태로 제공합니다. 예를 들어, GitHub MCP Server는 `github://repo/issues/123` 같은 URI로 특정 이슈를 리소스로 노출합니다. AI는 이 리소스를 읽어서 컨텍스트를 확보할 수 있습니다.

셋째는 Prompts입니다. 이것은 재사용 가능한 프롬프트 템플릿입니다. "PR 리뷰 프롬프트", "버그 분석 프롬프트" 같은 것을 MCP Server에 내장할 수 있습니다. 도메인 전문가가 만든 고품질 프롬프트를 모든 사용자가 쉽게 활용하게 만듭니다.

### 실전 MCP 서버 구현

간단한 MCP Server를 직접 만들어보면 개념이 명확해집니다. Python으로 "할 일 관리" MCP Server를 만들어봅시다. 이 서버는 SQLite 데이터베이스에 할 일을 저장하고, 추가/조회/완료 표시 기능을 제공합니다.

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import sqlite3
from datetime import datetime

# MCP 서버 초기화
app = Server("todo-manager")

# 데이터베이스 초기화
def init_db():
    conn = sqlite3.connect("todos.db")
    conn.execute("""
        CREATE TABLE IF NOT EXISTS todos (
            id INTEGER PRIMARY KEY,
            title TEXT NOT NULL,
            description TEXT,
            completed BOOLEAN DEFAULT 0,
            created_at TEXT,
            completed_at TEXT
        )
    """)
    conn.commit()
    conn.close()

init_db()

# Tool 1: 할 일 추가
async def add_todo(title: str, description: str = "") -> str:
    """새로운 할 일을 추가합니다."""
    conn = sqlite3.connect("todos.db")
    cursor = conn.cursor()
    
    cursor.execute(
        "INSERT INTO todos (title, description, created_at) VALUES (?, ?, ?)",
        (title, description, datetime.now().isoformat())
    )
    
    todo_id = cursor.lastrowid
    conn.commit()
    conn.close()
    
    return f"할 일이 추가되었습니다 (ID: {todo_id})"

# Tool 2: 할 일 목록 조회
async def list_todos(show_completed: bool = False) -> str:
    """할 일 목록을 조회합니다."""
    conn = sqlite3.connect("todos.db")
    cursor = conn.cursor()
    
    if show_completed:
        cursor.execute("SELECT * FROM todos ORDER BY created_at DESC")
    else:
        cursor.execute(
            "SELECT * FROM todos WHERE completed = 0 ORDER BY created_at DESC"
        )
    
    todos = cursor.fetchall()
    conn.close()
    
    if not todos:
        return "할 일이 없습니다."
    
    result = []
    for todo in todos:
        status = "✅" if todo[3] else "⬜"
        result.append(
            f"{status} [{todo[0]}] {todo[1]}\n"
            f"   {todo[2] if todo[2] else '설명 없음'}\n"
            f"   생성: {todo[4]}"
        )
    
    return "\n\n".join(result)

# Tool 3: 할 일 완료 표시
async def complete_todo(todo_id: int) -> str:
    """할 일을 완료 상태로 변경합니다."""
    conn = sqlite3.connect("todos.db")
    cursor = conn.cursor()
    
    cursor.execute(
        "UPDATE todos SET completed = 1, completed_at = ? WHERE id = ?",
        (datetime.now().isoformat(), todo_id)
    )
    
    if cursor.rowcount == 0:
        conn.close()
        return f"ID {todo_id}인 할 일을 찾을 수 없습니다."
    
    conn.commit()
    conn.close()
    
    return f"할 일 {todo_id}을(를) 완료했습니다."

# 서버 실행
if __name__ == "__main__":
    app.run()
```

이 서버는 세 개의 도구를 제공합니다. `add_todo`는 새 할 일을 추가하고, `list_todos`는 목록을 조회하며, `complete_todo`는 완료 표시를 합니다. 각 함수의 시그니처(매개변수 타입, 반환 타입, docstring)가 자동으로 도구 스키마로 변환됩니다.

이제 이 서버를 Claude Desktop에 등록하려면, `~/Library/Application Support/Claude/claude_desktop_config.json` 파일(macOS 기준)에 다음을 추가합니다:

```json
{
  "mcpServers": {
    "todo-manager": {
      "command": "python",
      "args": ["/path/to/todo_server.py"]
    }
  }
}
```

Claude를 재시작하면, 이제 자연어로 할 일을 관리할 수 있습니다:

```
사용자: "내일 회의 준비하기를 할 일로 추가해줘"
Claude: [add_todo 도구 호출]
       할 일이 추가되었습니다 (ID: 1)

사용자: "내 할 일 목록 보여줘"
Claude: [list_todos 도구 호출]
       ⬜ [1] 내일 회의 준비하기
          설명 없음
          생성: 2026-01-10T10:30:00

사용자: "1번 완료했어"
Claude: [complete_todo 도구 호출]
       할 일 1을(를) 완료했습니다.
```

### MCP의 실무 활용 패턴

실무에서 MCP를 효과적으로 활용하려면 몇 가지 패턴을 이해해야 합니다. 첫째, 도메인별로 서버를 분리하는 것입니다. 하나의 거대한 MCP Server보다는, `github-mcp`, `slack-mcp`, `database-mcp`처럼 관심사별로 나누는 것이 유지보수에 좋습니다. 각 팀이 자신의 도메인 서버를 관리하고, 다른 팀은 그것을 가져다 쓰는 구조입니다.

둘째, 권한 관리를 명시적으로 설계하는 것입니다. MCP Server 내부에서 사용자 인증과 권한 체크를 수행해야 합니다. "이 사용자는 읽기만 가능", "이 사용자는 특정 채널에만 메시지 전송 가능" 같은 정책을 코드로 구현합니다. AI가 도구를 호출한다고 해서, 모든 권한을 무제한으로 주는 것은 위험합니다.

셋째, 에러 처리를 세밀하게 하는 것입니다. 외부 API는 언제든 실패할 수 있습니다. 네트워크 타임아웃, Rate Limit 초과, 인증 만료 등 다양한 에러가 발생합니다. MCP Server는 이런 에러를 명확한 메시지로 변환해서 AI에게 전달해야 합니다. "Slack API rate limit exceeded. Retry after 60 seconds." 같은 구체적인 정보를 주면, AI가 적절히 대응할 수 있습니다.

넷째, 로깅과 감사를 기본으로 하는 것입니다. 모든 도구 호출을 로그에 남기고, 누가 언제 어떤 도구를 어떤 인자로 호출했는지 추적합니다. 특히 중요한 작업(데이터 삭제, 권한 변경, 금액 이동 등)은 별도의 감사 로그를 남기고, 필요하면 사람의 승인을 받도록 설계합니다.

---

## 3부: Skill - 신뢰를 만드는 레시피

### Skill의 본질

Skill은 정해진 흐름을 따라 특정 업무를 완수하는 워크플로우입니다. 요리 레시피처럼, 단계가 명확하고, 순서가 정해져 있으며, 같은 재료로 같은 과정을 거치면 같은 결과가 나옵니다. AI는 각 단계를 "잘" 수행하는 역할을 맡지만, 전체 흐름은 사람이 설계합니다.

Skill이 강력한 이유는 예측 가능성입니다. 매일 아침 9시에 "일일 보고서 생성" Skill이 실행되면, 정확히 같은 방식으로 데이터를 수집하고, 같은 포맷으로 정리하며, 같은 채널에 보고합니다. 팀원이 바뀌어도, 모델 버전이 바뀌어도, 결과는 일관됩니다. 이것이 운영 환경에서 가장 중요한 가치입니다.

Skill의 또 다른 장점은 감사 가능성입니다. 문제가 생기면 정확히 어느 단계에서 왜 실패했는지 알 수 있습니다. "3단계 데이터 수집에서 API 타임아웃 발생"처럼 명확합니다. Agent가 자율적으로 실행하다가 실패하면, 디버깅이 어렵습니다. 모델이 어떤 생각의 흐름으로 그런 결정을 내렸는지 추적하기 힘듭니다. 하지만 Skill은 흐름이 명시적이라 문제 지점을 바로 찾을 수 있습니다.

### Skill의 구조

Skill은 대개 마크다운이나 YAML로 작성됩니다. Anthropic의 Claude Desktop에서는 `.claude/skills/` 디렉토리에 마크다운 파일로 Skill을 정의합니다. 간단한 예를 봅시다. "GitHub 이슈 요약 보고서" Skill입니다:

~~~markdown
---
description: GitHub 이슈를 주간 단위로 요약하여 보고서를 생성하고 Slack에 전송
---

# GitHub 이슈 주간 보고서 Skill

## Step 1: 기간 설정

지난 7일간의 이슈를 조회합니다.
- 시작일: 오늘 - 7일
- 종료일: 오늘

## Step 2: 이슈 조회

GitHub MCP를 사용하여 이슈 목록을 가져옵니다.

```
도구: github_list_issues
매개변수:
  - repository: "company/main-product"
  - state: "all"
  - since: [Step 1의 시작일]
  - until: [Step 1의 종료일]
```

## Step 3: 이슈 분류

조회된 이슈들을 다음 기준으로 분류합니다:
- 새로 생성된 이슈
- 진행 중인 이슈
- 완료된 이슈
- 우선순위별 (P0, P1, P2)
- 담당자별

## Step 4: 통계 계산

다음 통계를 계산합니다:
- 총 이슈 수
- 완료율 (완료된 이슈 / 전체 이슈)
- 평균 처리 시간
- 담당자별 처리 건수

## Step 5: 보고서 작성

다음 형식으로 보고서를 작성합니다:

```
# GitHub 이슈 주간 보고서 (YYYY-MM-DD ~ YYYY-MM-DD)

## 요약
- 총 이슈: [수]
- 완료: [수] ([완료율]%)
- 평균 처리 시간: [일]일

## 주요 이슈
[P0/P1 이슈 목록, 각 이슈마다 제목, 담당자, 상태]

## 담당자별 현황
[각 담당자별 처리 건수와 주요 이슈]

## 주간 인사이트
[특이사항, 트렌드, 개선이 필요한 부분]
```

## Step 6: Slack 전송

작성된 보고서를 Slack의 #dev-reports 채널에 전송합니다.

```
도구: slack_send_message
매개변수:
  - channel: "#dev-reports"
  - text: [Step 5에서 작성한 보고서]
```

## Step 7: 완료 확인

다음 정보를 반환합니다:
- 보고서 생성 시간
- 조회된 이슈 수
- Slack 전송 성공 여부
~~~

이 Skill은 7개의 명확한 단계로 구성되어 있습니다. 각 단계는 입력이 무엇이고 출력이 무엇인지 명시합니다. AI는 이 흐름을 따라가며, 각 단계의 작업을 수행합니다. 예를 들어 Step 5에서 보고서를 작성할 때, AI는 주어진 형식에 맞춰 실제 데이터를 채워 넣습니다. 하지만 "보고서를 작성한다"는 것 자체는 이미 정해진 단계입니다.

### Skill과 Workflow의 차이

여기서 혼란스러울 수 있는 개념이 Workflow입니다. Skill과 Workflow는 무엇이 다를까요? 실무적으로는 거의 같은 개념으로 쓰입니다. 다만 강조점이 약간 다릅니다.

Workflow는 "흐름"에 초점을 맞춥니다. A를 하고, 그다음 B를 하고, 그다음 C를 한다는 순서와 의존성을 강조합니다. 주로 자동화 도구(Zapier, n8n, Airflow)에서 쓰는 용어입니다.

Skill은 "능력"에 초점을 맞춥니다. 이 시스템이 "할 수 있는 것"이 무엇인지를 강조합니다. "주간 보고서 생성 스킬", "고객 문의 자동 분류 스킬"처럼 특정 업무를 완수하는 능력을 캡슐화한 것입니다. 주로 AI 도구(Claude, GPT)에서 쓰는 용어입니다.

하지만 본질은 같습니다. 정해진 단계를 따라 업무를 수행하는 것입니다. 이 가이드에서는 "Skill"로 통일하겠습니다.

### Skill 설계의 5가지 원칙

좋은 Skill을 설계하려면 다섯 가지 원칙을 따라야 합니다.

첫째, 단일 책임 원칙입니다. 하나의 Skill은 하나의 명확한 목적만 가져야 합니다. "GitHub 이슈 보고서 + 코드 리뷰 + 배포 체크리스트"를 하나의 Skill에 넣으면 안 됩니다. 각각을 독립적인 Skill로 만들고, 필요하면 "마스터 Skill"이 여러 Skill을 순차적으로 호출하도록 설계합니다.

둘째, 명확한 입력과 출력입니다. Skill이 무엇을 받아서 무엇을 만드는지 명시해야 합니다. "일일 판매 리포트 Skill"이라면, 입력은 "날짜 범위"이고 출력은 "마크다운 형식의 리포트 + Slack 전송 확인"입니다. 이것이 명확하지 않으면, 재사용하기 어렵고 테스트하기도 힘듭니다.

셋째, 원자적 단계입니다. 각 단계는 더 이상 나눌 수 없을 정도로 작고 명확해야 합니다. "데이터 수집 및 분석"은 너무 큽니다. "데이터 수집", "데이터 정제", "통계 계산", "그래프 생성"처럼 나눠야 합니다. 원자적 단계는 디버깅을 쉽게 만들고, 재사용성을 높입니다.

넷째, 에러 처리입니다. 각 단계에서 실패 가능성을 고려하고, 실패했을 때 어떻게 할지 명시해야 합니다. "API 호출 실패 시 3회 재시도, 그래도 실패하면 관리자에게 알림"처럼 구체적으로 정의합니다. Skill이 중간에 멈추면 안 됩니다. 실패를 우아하게 처리하거나, 적어도 실패 상태를 명확히 보고해야 합니다.

다섯째, 테스트 가능성입니다. Skill을 실제 환경에서 실행하기 전에, 안전하게 테스트할 수 있어야 합니다. Mock 데이터를 사용하거나, Dry-run 모드를 지원하거나, 테스트 전용 환경을 만들거나 해서, "이 Skill이 정말 예상대로 작동하는가"를 검증할 수 있어야 합니다.

### 실전 Skill: 장애 대응 자동화

실무에서 자주 쓰이는 "장애 초기 대응" Skill을 설계해봅시다. 프로덕션 환경에서 에러 알림이 오면, 이 Skill이 자동으로 기본 정보를 수집하고 초기 분석을 수행합니다.

~~~markdown
---
description: 프로덕션 장애 발생 시 자동으로 초기 정보를 수집하고 분석하는 Skill
trigger: Slack 알림 (#alerts 채널에 "CRITICAL" 포함)
---

# 장애 초기 대응 Skill

## Step 1: 알림 파싱

Slack 알림 메시지에서 핵심 정보를 추출합니다:
- 서비스 이름
- 에러 타입
- 발생 시간
- 영향 범위 (사용자 수, 지역 등)

## Step 2: 최근 로그 수집

Elasticsearch MCP를 사용하여 최근 10분간의 에러 로그를 수집합니다.

```
도구: elasticsearch_query
매개변수:
  - index: "production-logs-*"
  - query:
      range:
        timestamp: 
          gte: "now-10m"
      match:
        level: "ERROR"
  - size: 100
  - sort: timestamp desc
```

## Step 3: 에러 패턴 분석

수집된 로그에서 패턴을 찾습니다:
- 가장 많이 발생한 에러 메시지 (Top 5)
- 에러 발생 빈도 (분당 몇 건)
- 특정 엔드포인트에 집중되었는지
- 특정 사용자 그룹에만 발생했는지

## Step 4: 인프라 상태 확인

DataDog MCP를 사용하여 인프라 메트릭을 확인합니다.

```
도구: datadog_get_metrics
매개변수:
  - metrics:
      - "system.cpu.user"
      - "system.mem.used"
      - "postgresql.connections"
      - "redis.connected_clients"
  - timeframe: "last_10_minutes"
```

이상 징후를 찾습니다:
- CPU 사용률 90% 이상
- 메모리 사용률 85% 이상
- DB 커넥션 풀 고갈
- Redis 응답 지연

## Step 5: 최근 배포 확인

GitHub MCP를 사용하여 최근 24시간 내 배포를 확인합니다.

```
도구: github_list_deployments
매개변수:
  - repository: "company/main-product"
  - environment: "production"
  - since: "now-24h"
```

만약 최근 배포가 있다면:
- 배포 시간과 장애 발생 시간 비교
- 배포된 커밋의 변경사항 확인
- 롤백 가능 여부 판단

## Step 6: 초기 분석 리포트 생성

다음 형식으로 리포트를 작성합니다:

```
# 🚨 장애 초기 분석 리포트

**발생 시간**: [시간]
**서비스**: [이름]
**심각도**: CRITICAL

## 증상
- 에러 타입: [타입]
- 에러 빈도: [분당 N건]
- 영향 범위: [사용자 수 또는 지역]

## 주요 에러 (Top 3)
1. [에러 메시지 1] (N건)
2. [에러 메시지 2] (M건)
3. [에러 메시지 3] (K건)

## 인프라 상태
- CPU: [%]
- Memory: [%]
- DB Connections: [N/Max]
- 이상 징후: [있음/없음]

## 최근 변경사항
- 마지막 배포: [시간] ([커밋 해시])
- 주요 변경: [요약]

## 초기 가설
[로그와 메트릭을 바탕으로 가능한 원인 추정]

## 권장 조치
1. [즉시 해야 할 일]
2. [다음 단계]
3. [롤백 고려 여부]
```

## Step 7: 보고 및 알림

작성된 리포트를 여러 채널에 전송합니다:

1. Slack #incidents 채널에 전체 리포트 전송
2. PagerDuty에 인시던트 생성
3. 담당 엔지니어에게 SMS 발송 (심각도 CRITICAL인 경우)

```
도구 1: slack_send_message
매개변수:
  - channel: "#incidents"
  - text: [리포트]

도구 2: pagerduty_create_incident
매개변수:
  - title: [서비스명] 장애
  - severity: "critical"
  - description: [리포트 요약]
  - assigned_to: [온콜 엔지니어]
```

## Step 8: 모니터링 대시보드 생성

Grafana MCP를 사용하여 임시 대시보드를 생성합니다.
해당 서비스의 핵심 메트릭을 실시간으로 보여주는 대시보드를 만들고,
URL을 Slack에 공유합니다.

## 에러 처리

- Step 2 (로그 수집) 실패: "로그 시스템 접근 불가. 수동 확인 필요" 알림
- Step 4 (메트릭 수집) 실패: 해당 섹션을 "N/A"로 표시하고 계속 진행
- Step 7 (알림 전송) 실패: 로컬에 리포트 저장 후 관리자 직접 연락

## 성공 기준

- 알림 수신부터 리포트 생성까지 3분 이내
- 리포트에 최소 3개 이상의 액션 아이템 포함
- 담당자에게 알림 전달 성공
~~~

이 Skill은 실제 장애 대응의 초기 단계를 자동화합니다. 사람이 직접 하면 15-20분 걸리는 작업을 3분 안에 완료합니다. 더 중요한 것은, 빠짐없이 모든 정보를 수집한다는 것입니다. 사람은 스트레스 상황에서 실수하지만, Skill은 항상 같은 체크리스트를 따릅니다.

---

## 4부: Agent - 자율 실행의 힘과 위험

### Agent의 본질

Agent는 목표만 주어지면 스스로 계획을 세우고, 도구를 선택하며, 결과를 보고 계획을 수정하는 자율 실행 시스템입니다. Skill과의 핵심 차이는 "누가 결정하는가"입니다. Skill은 사람이 모든 단계를 정의하지만, Agent는 AI가 단계를 결정합니다.

간단한 예를 들어봅시다. "지난 분기 매출 하락 원인을 분석하고 보고서를 작성하라"는 목표를 줬다고 합시다. Skill이라면 이렇게 작동합니다: "1단계 매출 데이터 조회, 2단계 고객사별 분해, 3단계 제품별 분해, 4단계 지역별 분해, 5단계 트렌드 분석, 6단계 보고서 작성." 모든 단계가 미리 정해져 있습니다.

Agent라면 이렇게 작동합니다: "먼저 전체 매출 데이터를 봐야겠다. [매출 조회] 음, 3월에 급락했네. 3월에 무슨 일이 있었을까? [고객사별 조회] A사의 주문이 사라졌구나. A사에 무슨 일이 있었는지 알아봐야겠다. [CRM 조회] 담당자가 퇴사했고 후임이 없었구나. 그럼 다른 고객사는? [B사 조회] B사는 정상이네. 그럼 A사 문제가 주요 원인이구나. 근데 왜 후임을 안 뽑았지? [HR 시스템 조회] 채용 동결이었구나. 그럼 이건 영업팀 문제가 아니라 회사 정책 문제네..."

Agent는 중간 결과를 보면서 다음 행동을 결정합니다. 계획이 동적으로 바뀝니다. 처음에는 예상하지 못한 패턴을 발견하면, 새로운 방향으로 탐색합니다. 이것이 Agent의 강력함입니다. 복잡하고 예측 불가능한 문제를 풀 수 있습니다.

하지만 그만큼 위험합니다. Agent가 잘못된 방향으로 탐색하면, 무한 루프에 빠지거나 엉뚱한 데이터를 망치거나 비용을 폭발시킬 수 있습니다. "매출 하락 원인 찾기"가 "전 직원 이메일 읽기"로 확장되고, "고객사 계약서 전부 다운로드"로 이어지고, "경쟁사 웹사이트 크롤링"까지 가면서 API 요금이 수천 달러에 도달할 수 있습니다.

### Agent의 핵심 구조: ReAct 패턴

대부분의 Agent는 ReAct(Reasoning + Acting) 패턴을 따릅니다. 이것은 "생각하고, 행동하고, 관찰하고, 다시 생각하고..."를 반복하는 루프입니다. 각 사이클마다 세 단계가 있습니다:

첫째, Thought(생각)입니다. Agent가 현재 상황을 분석하고 다음에 무엇을 할지 결정합니다. "매출 데이터를 봤는데 3월에 급락했다. 그 시기에 무슨 일이 있었는지 확인해야겠다."

둘째, Action(행동)입니다. Agent가 도구를 선택하고 실행합니다. "CRM에서 3월의 주요 이벤트를 조회하자."

셋째, Observation(관찰)입니다. 도구 실행 결과를 받아서 분석합니다. "A사 담당자가 퇴사했고 후임이 3개월간 없었구나."

그리고 다시 Thought로 돌아갑니다. 이 루프는 목표를 달성할 때까지, 또는 최대 반복 횟수에 도달할 때까지 계속됩니다.

코드로 보면 더 명확합니다. LangChain 기반의 간단한 Agent 구현입니다:

```python
from langchain.agents import initialize_agent, AgentType
from langchain.chat_models import ChatAnthropic
from langchain.tools import tool

# 도구 정의
def get_sales_data(period: str) -> str:
    """지정된 기간의 매출 데이터를 조회합니다."""
    # 실제로는 데이터베이스 쿼리
    return f"{period} 매출: 120M (전년 대비 -15%)"

def get_customer_data(customer_id: str) -> str:
    """특정 고객사의 상세 정보를 조회합니다."""
    # 실제로는 CRM API 호출
    return f"고객사 {customer_id}: 담당자 퇴사(2026-03-01), 주문 중단"

def search_internal_docs(query: str) -> str:
    """내부 문서를 검색합니다."""
    # 실제로는 문서 검색 시스템
    return f"'{query}' 관련 문서: 채용 동결 공지(2026-02-15)"

# Agent 초기화
llm = ChatAnthropic(model="claude-sonnet-4-20250514", temperature=0)
tools = [get_sales_data, get_customer_data, search_internal_docs]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    max_iterations=10,  # 최대 10번 루프
    max_execution_time=300,  # 최대 5분
)

# 실행
result = agent.run(
    "지난 분기 매출이 왜 하락했는지 원인을 찾아서 "
    "보고서 형식으로 정리해줘. 주요 원인을 3가지 이상 찾아야 해."
)

print(result)
```

실행 로그는 이렇게 나옵니다:

```
> Entering new AgentExecutor chain...

Thought: 먼저 전체 매출 추이를 확인해야겠다.
Action: get_sales_data
Action Input: {"period": "2026-Q1"}

Observation: 2026-Q1 매출: 120M (전년 대비 -15%)

Thought: 15% 하락은 심각하다. 고객사별로 분해해서 어디서 문제가 생겼는지 봐야겠다.
Action: get_customer_data
Action Input: {"customer_id": "A사"}

Observation: 고객사 A사: 담당자 퇴사(2026-03-01), 주문 중단

Thought: A사 담당자 퇴사가 매출 하락의 주요 원인일 수 있다. 왜 후임을 안 뽑았을까?
Action: search_internal_docs
Action Input: {"query": "채용 동결"}

Observation: '채용 동결' 관련 문서: 채용 동결 공지(2026-02-15)

Thought: 이제 패턴이 보인다. 원인을 정리하자.

Final Answer:
# 2026 Q1 매출 하락 원인 분석 보고서

## 주요 원인

1. **A사 주문 중단** (-10%p 영향)
   - 담당 영업사원 퇴사 (2026-03-01)
   - 채용 동결로 후임 미배치
   - 3개월간 고객 관리 공백

2. **회사 차원 채용 동결** (간접 영향)
   - 2026-02-15부터 시행
   - 영업팀 인력 부족 심화
   - 신규 고객 확보 어려움

3. **영업 프로세스 취약성** (구조적 문제)
   - 단일 담당자 의존 체계
   - 인수인계 프로세스 부재
   - 위기 대응 매뉴얼 미비

## 권장 조치

1. 즉시: A사 관계 복구 (임원 차원 접촉)
2. 단기: 팀 간 백업 체계 구축
3. 장기: 고객 관리 시스템 자동화

> Finished chain.
```

Agent가 스스로 3단계를 거쳐 문제를 해결했습니다. 사람이 "데이터 조회 → 고객 조회 → 문서 검색"이라고 지시하지 않았습니다. Agent가 중간 결과를 보면서 "다음은 이걸 봐야겠다"고 판단했습니다.

### Agent의 4가지 위험 요소

Agent를 운영 환경에 도입하려면, 네 가지 위험을 반드시 관리해야 합니다.

첫째, 무한 루프입니다. Agent가 같은 행동을 반복하면서 빠져나오지 못하는 상황입니다. "고객 데이터 조회 → 분석 → 다시 조회 → 다시 분석..."을 무한히 반복하면, 시간과 비용이 낭비됩니다. 해결책은 명확한 종료 조건과 최대 반복 횟수입니다. 위 코드의 `max_iterations=10`이 바로 이것입니다.

둘째, 비용 폭발입니다. Agent가 API를 수백 번 호출하면서 비용이 급증할 수 있습니다. 특히 LLM API는 호출당 비용이 높습니다. GPT-4를 100번 호출하면 수십 달러가 나갑니다. 해결책은 비용 예산을 설정하고, 예산에 도달하면 강제 종료하는 것입니다.

```python
class CostLimitExceeded(Exception):
    pass

class BudgetTracker:
    def __init__(self, max_cost_usd=10.0):
        self.max_cost = max_cost_usd
        self.current_cost = 0.0
    
    def add_cost(self, cost):
        self.current_cost += cost
        if self.current_cost > self.max_cost:
            raise CostLimitExceeded(
                f"예산 초과: ${self.current_cost:.2f} / ${self.max_cost:.2f}"
            )

budget = BudgetTracker(max_cost_usd=5.0)

# Agent 실행 시 비용 추적
# 각 LLM 호출 후 budget.add_cost() 호출
```

셋째, 잘못된 도구 사용입니다. Agent가 삭제 도구를 잘못 호출하거나, 민감한 데이터를 잘못된 곳에 전송할 수 있습니다. 해결책은 도구에 권한 체크를 내장하고, 위험한 도구는 "확인 필요" 플래그를 설정하는 것입니다.

```python
def delete_customer_data(customer_id: str) -> str:
    """고객 데이터를 삭제합니다. 주의: 복구 불가능!"""
    # 실행 전 사람의 승인 필요
    return f"고객 {customer_id} 데이터 삭제 완료"

# Agent가 이 도구를 호출하려고 하면:
# "⚠️ delete_customer_data를 실행하려고 합니다. 승인하시겠습니까? (Y/N)"
```

넷째, 환각(Hallucination)입니다. Agent가 없는 도구를 호출하려 하거나, 잘못된 매개변수를 전달하거나, 결과를 오해석할 수 있습니다. 해결책은 도구 스키마를 엄격히 정의하고, 결과 검증 로직을 추가하는 것입니다.

### Agent vs Skill: 언제 무엇을 쓸 것인가

실무적으로 가장 중요한 질문은 "이 작업을 Agent로 할 것인가, Skill로 할 것인가"입니다. 명확한 결정 기준이 있습니다.

다음 조건에 해당하면 Skill을 사용합니다:

1. 작업 흐름이 명확하고 반복 가능하다. 매일 같은 방식으로 실행되어야 한다.
2. 결과 포맷이 고정되어 있다. "항상 이 형식으로 보고서를 만들어야 한다."
3. 실패 시 영향이 크다. 고객 데이터를 다루거나, 금융 거래를 하거나, 법적 문서를 생성한다.
4. 감사가 필요하다. 누가 언제 무엇을 했는지 명확히 추적해야 한다.
5. 여러 사람이 실행한다. 팀원들이 같은 결과를 재현할 수 있어야 한다.

예시: 일일 보고서, 배포 체크리스트, 고객 문의 분류, 송장 생성, 코드 린트

다음 조건에 해당하면 Agent를 사용합니다:

1. 예외가 많다. 표준 절차로 해결할 수 없는 케이스가 많다.
2. 탐색이 필요하다. 어디서부터 봐야 할지 모르는 문제를 푼다.
3. 상황에 따라 접근이 달라진다. A면 이렇게, B면 저렇게 해야 한다.
4. 창의성이 필요하다. 새로운 해결책을 찾아야 한다.
5. 사람이 직접 하기엔 시간이 오래 걸린다. 수십 개의 소스를 조사해야 한다.

예시: 장애 원인 분석, 경쟁사 리서치, 복잡한 고객 문의 대응, 코드 리팩토링 제안, 신규 시장 조사

중간 지대도 있습니다. "반은 정해져 있고 반은 자율적"인 작업입니다. 이 경우 하이브리드 접근을 씁니다. Skill로 기본 흐름을 만들고, 특정 단계에서 Agent를 호출합니다.

예를 들어, "고객 문의 대응" 작업은 이렇게 설계할 수 있습니다:

```
Skill: 고객 문의 대응 워크플로우

Step 1: 문의 분류 (Skill)
  - 유형: 기술 지원 / 결제 / 계정 / 기타
  - 우선순위: 긴급 / 보통 / 낮음

Step 2: 자동 대응 가능 여부 확인 (Skill)
  - FAQ 검색
  - 일치하면 → 템플릿 답변 전송 후 종료
  - 불일치하면 → Step 3으로

Step 3: 맞춤 답변 생성 (Agent)
  - 목표: 고객 질문에 정확하고 친절하게 답변
  - 도구: 내부 문서 검색, 과거 티켓 조회, 제품 스펙 조회
  - 제약: 500자 이내, 전문 용어 최소화
  → Agent가 자율적으로 정보를 수집하고 답변 작성

Step 4: 답변 검토 (Skill)
  - 금지어 체크 (회사 정책)
  - 톤 체크 (친절함, 전문성)
  - 통과하면 전송, 실패하면 → Step 5

Step 5: 사람 검토 요청 (Skill)
  - 담당자에게 알림
  - 초안과 원본 질문 전달
```

이 하이브리드 방식은 안정성과 유연성을 동시에 확보합니다. 정형화된 부분은 Skill로 보장하고, 창의성이 필요한 부분만 Agent에게 맡깁니다.

---

## 5부: 실전 통합 전략

### 3-Layer 아키텍처

실무에서 MCP, Skill, Agent를 통합할 때는 명확한 아키텍처가 필요합니다. 가장 효과적인 패턴은 3-Layer 아키텍처입니다.

**Layer 1: MCP 도구 계층**

가장 아래에 모든 외부 연결을 MCP로 표준화합니다. 데이터베이스, API, 파일 시스템, 메시징 등 모든 것이 MCP Server로 패키징됩니다. 이 계층은 "어떻게"에 집중합니다. "어떻게 Slack에 메시지를 보내는가", "어떻게 GitHub 이슈를 생성하는가" 같은 기술적 구현입니다.

팀 구조: 각 도메인팀이 자신의 MCP Server를 관리합니다. DevOps 팀은 인프라 MCP Server, Data 팀은 분석 MCP Server, Product 팀은 제품 정보 MCP Server를 만듭니다.

**Layer 2: Skill 기능 계층**

중간에 비즈니스 로직을 Skill로 구현합니다. MCP 도구들을 조합해서 의미 있는 업무를 완성합니다. 이 계층은 "무엇을"에 집중합니다. "무엇을 달성해야 하는가", "어떤 결과물을 만들어야 하는가" 같은 비즈니스 목표입니다.

팀 구조: 비즈니스 팀과 개발팀이 협업합니다. 비즈니스 팀이 "이런 보고서가 필요해"라고 요구사항을 정의하면, 개발팀이 Skill로 구현합니다. 그리고 비즈니스 팀이 직접 실행하고 피드백합니다.

**Layer 3: Agent 자율 계층**

맨 위에 복잡하고 예측 불가능한 작업을 Agent가 처리합니다. Skill을 "고수준 도구"처럼 호출하거나, MCP 도구를 직접 조합하면서 목표를 달성합니다. 이 계층은 "왜"에 집중합니다. "왜 이런 결과가 나왔는가", "왜 다른 접근이 필요한가" 같은 추론과 판단입니다.

팀 구조: 전문가와 AI가 협업합니다. 도메인 전문가가 복잡한 문제를 정의하면, Agent가 자율적으로 해결하고, 전문가가 결과를 검증합니다.

### 실전 사례: 전사 고객 지원 시스템

구체적인 예로 "전사 고객 지원 시스템"을 3-Layer로 설계해봅시다.

**Layer 1: MCP Servers (8개)**

```
1. zendesk-mcp: 티켓 조회/생성/업데이트
2. slack-mcp: 메시지 전송, 채널 관리
3. docs-mcp: 내부 문서 검색
4. crm-mcp: 고객 정보 조회
5. product-mcp: 제품 스펙, FAQ 조회
6. analytics-mcp: 고객 행동 데이터 조회
7. billing-mcp: 결제 정보 조회
8. email-mcp: 이메일 전송
```

각 서버는 독립적으로 개발되고 테스트됩니다. Zendesk 팀이 zendesk-mcp를 관리하고, IT 팀이 docs-mcp를 관리합니다.

**Layer 2: Skills (12개)**

```
1. classify-ticket: 티켓 자동 분류
   - zendesk-mcp로 티켓 읽기
   - product-mcp로 제품 정보 확인
   - 카테고리 + 우선순위 자동 설정

2. auto-respond-faq: FAQ 자동 답변
   - docs-mcp로 FAQ 검색
   - 일치하면 템플릿 답변 전송
   - zendesk-mcp로 티켓 해결

3. escalate-to-expert: 전문가 에스컬레이션
   - crm-mcp로 고객 등급 확인
   - slack-mcp로 담당 전문가에게 알림
   - zendesk-mcp로 티켓 할당

4. usage-analysis: 사용 패턴 분석
   - analytics-mcp로 고객 행동 조회
   - 문제 발생 전 행동 분석
   - 패턴 리포트 생성

5. billing-check: 결제 상태 확인
   - billing-mcp로 결제 이력 조회
   - 미납/연체 여부 확인
   - 필요시 결제팀 알림

... (나머지 7개 Skill)
```

각 Skill은 명확한 입력과 출력이 있습니다. `classify-ticket`은 티켓 ID를 받아서 카테고리와 우선순위를 반환합니다. 테스트 가능하고 재사용 가능합니다.

**Layer 3: Agents (3개)**

```
1. complex-inquiry-agent
   목표: 복잡한 고객 문의에 맞춤 답변
   도구: 모든 MCP Servers + Skills
   제약: 답변 500자 이내, 검토 필수
   
   실행 흐름 (자율):
   - 티켓 내용 분석
   - 관련 문서/FAQ/과거 티켓 검색
   - 고객 사용 이력 확인
   - 맞춤 답변 작성
   - 초안을 사람에게 전달

2. root-cause-agent
   목표: 반복 문의의 근본 원인 찾기
   도구: analytics-mcp, product-mcp, docs-mcp
   제약: 최대 20분 실행, 비용 $2 이내
   
   실행 흐름 (자율):
   - 유사 티켓 패턴 분석
   - 공통 사용자 행동 추출
   - 제품 변경 이력 확인
   - UI/UX 문제 가설 생성
   - 제품팀에 리포트 전달

3. customer-journey-agent
   목표: 특정 고객의 전체 여정 재구성
   도구: crm-mcp, analytics-mcp, zendesk-mcp
   제약: VIP 고객만, 주 1회 제한
   
   실행 흐름 (자율):
   - 가입부터 현재까지 타임라인 생성
   - 주요 터치포인트 식별
   - 만족도 변화 추적
   - 이탈 위험 평가
   - 맞춤 유지 전략 제안
```

각 Agent는 넓은 자율성을 가지지만, 명확한 목표와 제약이 있습니다. 비용, 시간, 권한이 제한되어 있어서 폭주하지 않습니다.

### 통합 워크플로우: 티켓 처리 파이프라인

이제 이 3개 Layer가 어떻게 함께 작동하는지 실제 티켓 처리 과정을 따라가봅시다.

**티켓 도착**

```
고객: "결제가 안 돼요. 카드는 정상인데 계속 에러가 나요."
→ Zendesk에 티켓 생성 (자동)
```

**Step 1: 자동 분류 (Skill)**

```
classify-ticket Skill 실행:
1. zendesk-mcp로 티켓 읽기
2. 내용 분석: "결제", "에러" 키워드 발견
3. product-mcp로 결제 시스템 FAQ 확인
4. 분류 결과:
   - 카테고리: Billing
   - 우선순위: High (결제 차단)
   - 담당팀: Payments Team
5. zendesk-mcp로 티켓 업데이트
```

**Step 2: FAQ 확인 (Skill)**

```
auto-respond-faq Skill 실행:
1. docs-mcp로 "결제 에러" FAQ 검색
2. 10개 FAQ 발견, 하지만 정확히 일치하는 건 없음
   (고객은 "카드는 정상"이라고 했는데, FAQ는 "카드 확인" 권장)
3. 판단: 자동 답변 불가
4. 다음 단계로 넘김
```

**Step 3: 상세 정보 수집 (Skill)**

```
billing-check Skill 실행:
1. billing-mcp로 고객 결제 이력 조회
2. 발견:
   - 마지막 성공 결제: 2026-01-03
   - 실패한 시도: 2026-01-09 (오늘) 3회
   - 에러 코드: "insufficient_funds"
3. 모순 발견: 고객은 "카드 정상"인데 에러는 "잔액 부족"
```

**Step 4: Agent 호출 (복잡한 상황)**

```
complex-inquiry-agent 실행:

Thought 1: 고객은 카드가 정상이라고 하는데, 시스템은 잔액 부족 에러를 내고 있다.
           실제 결제 로그를 봐야겠다.

Action 1: analytics-mcp.get_payment_logs(customer_id, date="2026-01-09")
Observation 1: 
  - 시도 1 (10:30): Amount $299, Error "insufficient_funds"
  - 시도 2 (10:35): Amount $299, Error "insufficient_funds"  
  - 시도 3 (10:40): Amount $299, Error "insufficient_funds"

Thought 2: 같은 금액을 3번 시도했다. 고객 카드 한도를 확인해볼까?
           아니다, 우리는 카드 한도 정보가 없다.
           혹시 우리 측 문제일까? 다른 고객도 같은 에러가 있는지 봐야겠다.

Action 2: zendesk-mcp.search_tickets(
            query="insufficient_funds",
            date_range="today"
          )
Observation 2: 
  - 총 47건의 유사 티켓 발견!
  - 모두 오늘 오전 10시 이후 발생
  - 모두 같은 에러 코드

Thought 3: 이건 시스템 장애다! 결제 게이트웨이에 문제가 생긴 것 같다.
           고객 문제가 아니라 우리 문제다.
           즉시 기술팀에 알려야 한다.

Action 3: slack-mcp.send_urgent_message(
            channel="#payments-oncall",
            text="🚨 긴급: 결제 게이트웨이 장애 의심. "
                 "10시 이후 47건의 insufficient_funds 에러 발생. "
                 "고객 카드는 정상. 즉시 확인 필요."
          )

Thought 4: 이제 고객에게 답변을 만들어야 한다.
           "당신 카드 문제"라고 하면 안 되고,
           "우리 시스템 문제, 곧 해결됨"이라고 해야 한다.

Final Answer (초안):
---
안녕하세요,

문의 주신 결제 문제를 확인했습니다. 
말씀하신 대로 고객님의 카드에는 문제가 없으며, 
현재 저희 결제 시스템에 일시적인 기술적 문제가 발생한 것으로 확인되었습니다.

기술팀에서 긴급 조치 중이며, 약 30분 내로 해결될 예정입니다.
해결 즉시 별도로 안내드리겠습니다.

불편을 드려 죄송합니다.
---

[이 초안을 사람 검토자에게 전달]
```

**Step 5: 사람 검토 (Skill)**

```
human-review Skill 실행:
1. 담당자 John에게 Slack DM 전송
2. 초안, 원본 티켓, Agent 분석 과정 모두 전달
3. John 승인 → 고객에게 전송
4. John 거부 → 수정 후 재전송
```

이 전체 과정이 3분 안에 완료됩니다. Layer 1(MCP)은 모든 시스템에 접근하고, Layer 2(Skill)는 정형화된 단계를 수행하며, Layer 3(Agent)는 예외 상황을 판단하고 해결합니다.

---

## 6부: 운영과 거버넌스

### 신뢰를 만드는 것은 통제다

Agent가 아무리 똑똑해도, 통제되지 않으면 운영 환경에 들어갈 수 없습니다. 실무에서 AI 시스템을 도입할 때 가장 중요한 것은 "신뢰"입니다. 그리고 신뢰는 통제에서 나옵니다.

통제의 첫 번째 축은 **관찰 가능성(Observability)** 입니다. 시스템이 무엇을 하고 있는지, 왜 그런 결정을 내렸는지, 언제 실패했는지를 명확히 볼 수 있어야 합니다.

모든 도구 호출을 로그로 남깁니다. 누가, 언제, 어떤 도구를, 어떤 인자로 호출했는지, 결과는 무엇이었는지를 기록합니다. 이것은 MCP 레이어에서 자동으로 처리됩니다:

```python
import logging
from datetime import datetime

class MCPServerWithLogging:
    def __init__(self, name):
        self.name = name
        self.logger = logging.getLogger(f"mcp.{name}")
    
    def call_tool(self, tool_name, args, user_id):
        start_time = datetime.now()
        
        # 호출 로그
        self.logger.info(
            f"Tool called: {tool_name}",
            extra={
                "user_id": user_id,
                "tool": tool_name,
                "args": args,
                "timestamp": start_time.isoformat()
            }
        )
        
        try:
            result = self._execute_tool(tool_name, args)
            
            # 성공 로그
            duration = (datetime.now() - start_time).total_seconds()
            self.logger.info(
                f"Tool succeeded: {tool_name}",
                extra={
                    "user_id": user_id,
                    "tool": tool_name,
                    "duration_seconds": duration,
                    "result_size": len(str(result))
                }
            )
            
            return result
            
        except Exception as e:
            # 실패 로그
            self.logger.error(
                f"Tool failed: {tool_name}",
                extra={
                    "user_id": user_id,
                    "tool": tool_name,
                    "error_type": type(e).__name__,
                    "error_message": str(e)
                },
                exc_info=True
            )
            raise
```

이 로그들은 중앙 로그 시스템(ELK, Datadog 등)으로 전송되어, 대시보드에서 실시간 모니터링됩니다. "지난 1시간 동안 가장 많이 호출된 도구는?", "에러율이 높은 도구는?", "특정 사용자의 활동 타임라인은?" 같은 질문에 즉시 답할 수 있습니다.

Skill과 Agent의 실행 과정도 추적합니다. 각 단계별로 "시작", "완료", "실패" 이벤트를 기록합니다. Agent의 경우, 각 Thought-Action-Observation 사이클을 모두 저장합니다. 나중에 "왜 Agent가 이런 결정을 내렸는가"를 재구성할 수 있습니다.

통제의 두 번째 축은 **권한 관리(Authorization) **입니다. 모든 사용자가 모든 도구를 쓸 수 있으면 안 됩니다. Role-Based Access Control(RBAC)을 구현합니다:

```python
class Permission:
    def __init__(self, tool_name, allowed_roles):
        self.tool_name = tool_name
        self.allowed_roles = set(allowed_roles)
    
    def check(self, user_role):
        if user_role not in self.allowed_roles:
            raise PermissionError(
                f"Role '{user_role}' is not allowed to use tool '{self.tool_name}'"
            )

# 권한 정의
permissions = {
    "database_query": Permission("database_query", ["engineer", "analyst", "admin"]),
    "database_delete": Permission("database_delete", ["admin"]),
    "slack_send_message": Permission("slack_send_message", ["everyone"]),
    "billing_refund": Permission("billing_refund", ["finance", "admin"]),
}

# 실행 전 권한 체크
def call_tool_with_auth(tool_name, args, user):
    if tool_name in permissions:
        permissions[tool_name].check(user.role)
    
    return execute_tool(tool_name, args)
```

중요한 도구는 추가 승인이 필요합니다. "데이터 삭제", "금액 이동", "권한 변경" 같은 작업은 Agent가 자동으로 실행하면 안 됩니다. 사람의 명시적 승인을 받아야 합니다:

```python
class ApprovalRequired(Exception):
    def __init__(self, tool_name, args, reason):
        self.tool_name = tool_name
        self.args = args
        self.reason = reason

def delete_customer_data(customer_id: str) -> str:
    """고객 데이터를 영구 삭제합니다."""
    raise ApprovalRequired(
        tool_name="delete_customer_data",
        args={"customer_id": customer_id},
        reason="데이터 삭제는 복구 불가능합니다. 승인이 필요합니다."
    )

# Agent가 이 도구를 호출하려 하면:
# 1. ApprovalRequired 예외 발생
# 2. 담당자에게 Slack/Email 알림
# 3. 담당자가 승인/거부 버튼 클릭
# 4. 승인되면 실제 실행, 거부되면 Agent에게 "거부됨" 반환
```

통제의 세 번째 축은 **비용 관리(Cost Management)** 입니다. Agent가 LLM API를 무제한 호출하면 비용이 폭발합니다. 예산을 설정하고 추적합니다:

```python
class CostTracker:
    # LLM 비용 (예시)
    COSTS = {
        "claude-sonnet-4": {"input": 0.003, "output": 0.015},  # per 1K tokens
        "gpt-4": {"input": 0.03, "output": 0.06},
    }
    
    def __init__(self, budget_usd):
        self.budget = budget_usd
        self.spent = 0.0
        self.calls = []
    
    def add_llm_call(self, model, input_tokens, output_tokens):
        cost = (
            (input_tokens / 1000) * self.COSTS[model]["input"] +
            (output_tokens / 1000) * self.COSTS[model]["output"]
        )
        
        self.spent += cost
        self.calls.append({
            "model": model,
            "input_tokens": input_tokens,
            "output_tokens": output_tokens,
            "cost_usd": cost,
            "timestamp": datetime.now()
        })
        
        if self.spent > self.budget:
            raise BudgetExceeded(
                f"예산 초과: ${self.spent:.2f} / ${self.budget:.2f}"
            )
        
        # 예산 80% 도달 시 경고
        if self.spent > self.budget * 0.8:
            logging.warning(
                f"Budget warning: {(self.spent/self.budget)*100:.1f}% used"
            )
    
    def report(self):
        return {
            "budget_usd": self.budget,
            "spent_usd": self.spent,
            "remaining_usd": self.budget - self.spent,
            "total_calls": len(self.calls),
            "breakdown": self._breakdown_by_model()
        }

# Agent 실행 시 예산 설정
tracker = CostTracker(budget_usd=10.0)
agent = Agent(tools=tools, cost_tracker=tracker)
```

매일/매주 비용 리포트를 생성하고, 예상치 못한 비용 증가가 있으면 알림을 보냅니다.

통제의 네 번째 축은 **안전 장치(Safety Rails) **입니다. Agent가 위험한 결정을 내리지 못하도록 제약을 겁니다:

```python
class SafetyRails:
    def __init__(self):
        self.rules = []
    
    def add_rule(self, rule_fn, error_message):
        """안전 규칙 추가"""
        self.rules.append((rule_fn, error_message))
    
    def check(self, tool_name, args):
        """모든 규칙 검사"""
        for rule_fn, error_message in self.rules:
            if not rule_fn(tool_name, args):
                raise SafetyViolation(error_message)

# 예시 규칙들
rails = SafetyRails()

# 규칙 1: 한 번에 100개 이상의 레코드 삭제 금지
rails.add_rule(
    lambda tool, args: not (
        tool == "database_delete" and 
        args.get("limit", 1) > 100
    ),
    "한 번에 100개 이상 삭제 불가"
)

# 규칙 2: 프로덕션 DB에 직접 쓰기 금지 (읽기만 허용)
rails.add_rule(
    lambda tool, args: not (
        tool.startswith("database_") and
        args.get("environment") == "production" and
        tool not in ["database_query", "database_read"]
    ),
    "프로덕션 DB 쓰기 금지"
)

# 규칙 3: 외부 URL 호출 시 허용 목록 확인
ALLOWED_DOMAINS = ["api.company.com", "internal.company.com"]
rails.add_rule(
    lambda tool, args: not (
        tool == "http_request" and
        not any(domain in args.get("url", "") for domain in ALLOWED_DOMAINS)
    ),
    f"허용된 도메인만 호출 가능: {ALLOWED_DOMAINS}"
)
```

### 점진적 도입 전략

실무에서 MCP, Skill, Agent를 도입할 때는 점진적 접근이 안전합니다. 한 번에 모든 것을 바꾸려 하면 실패합니다. 4단계 로드맵을 제안합니다.

**Phase 1: MCP 기반 구축 (1-2개월)**

목표: 모든 도구를 MCP로 표준화

먼저 가장 자주 쓰는 도구 5-10개를 MCP Server로 만듭니다. Slack, GitHub, 데이터베이스, 문서 시스템 등입니다. 각 서버는 독립적으로 테스트하고, 문서화하고, 팀에 공유합니다.

이 단계에서는 아직 AI를 본격적으로 도입하지 않습니다. 그냥 "우리 도구들을 표준 인터페이스로 정리"하는 것입니다. 개발자들이 MCP Server를 직접 호출해서 써보고, 피드백하고, 개선합니다.

성공 기준: 팀원 80% 이상이 일상 업무에서 MCP 도구를 사용

**Phase 2: Skill로 반복 작업 자동화 (2-3개월)**

목표: 가장 반복적이고 시간 소모적인 작업 10개를 Skill로 자동화

"매일 하는데 짜증나는 일" 목록을 만듭니다. 일일 보고서, 배포 체크리스트, 코드 리뷰 요약 등입니다. 각각을 Skill로 만들고, 한 명씩 테스트합니다. 잘 되면 팀 전체로 확산합니다.

이 단계에서는 AI의 "똑똑함"보다 "일관성"을 강조합니다. 같은 작업을 항상 같은 방식으로 하는 것이 목표입니다.

성공 기준: 10개 Skill이 매일 안정적으로 실행되고, 팀이 시간 절약을 체감

**Phase 3: Agent로 복잡한 문제 해결 (3-4개월)**

목표: 3-5개의 복잡한 작업을 Agent로 해결

장애 분석, 경쟁사 리서치, 복잡한 고객 문의 같은 것입니다. 각 Agent는 명확한 목표, 제약, 예산을 가지고 시작합니다. 처음에는 사람이 모든 단계를 감독하고, 점차 자율성을 높입니다.

이 단계에서 가장 중요한 것은 "실패 학습"입니다. Agent가 실수하면, 왜 실수했는지 분석하고, 안전 장치를 추가합니다. "이번에는 예산을 초과했으니 다음부터는 예산 제한 추가", "이번에는 잘못된 데이터를 삭제했으니 삭제 전 확인 추가" 같은 식입니다.

성공 기준: 3개 이상의 Agent가 월 20회 이상 성공적으로 실행되고, 실패율 5% 미만

**Phase 4: 전사 확산 및 최적화 (지속)**

목표: 모든 팀이 MCP/Skill/Agent를 사용하고, 지속적으로 개선

성공 사례를 문서화하고, 베스트 프랙티스를 공유하고, 교육 프로그램을 만듭니다. 새로운 팀이 쉽게 시작할 수 있도록 템플릿과 가이드를 제공합니다.

이 단계에서는 "플랫폼 팀"을 만드는 것을 고려합니다. MCP Server 개발, Skill 템플릿 관리, Agent 안전 장치 개선 같은 공통 작업을 전담하는 팀입니다.

---

## 결론: 똑똑함에서 신뢰로

MCP, Skill, Agent를 마스터하면서 깨닫게 되는 진실은 이것입니다. **실무에서 성공하는 것은 가장 똑똑한 시스템이 아니라, 가장 신뢰할 수 있는 시스템입니다.**

데모에서는 "이 Agent가 복잡한 문제를 10분 만에 풀었어요!"가 멋있어 보입니다. 하지만 운영 환경에서는 "이 Skill은 300일 동안 한 번도 실패하지 않았습니다"가 더 가치 있습니다.

똑똑함은 데모에서 빛나지만, 신뢰는 운영에서 살아남습니다.

MCP는 운영 가능성을 만듭니다. 표준화된 도구, 명확한 권한, 추적 가능한 로그. 이것이 없으면 시스템은 블랙박스가 됩니다.

Skill은 신뢰를 만듭니다. 예측 가능한 결과, 일관된 품질, 감사 가능한 흐름. 이것이 없으면 팀은 시스템을 믿지 못합니다.

Agent는 확장을 만듭니다. 복잡한 문제 해결, 창의적 접근, 자율적 학습. 이것이 없으면 시스템은 한계에 부딪힙니다.

세 가지를 현명하게 조합하세요. MCP로 기반을 다지고, Skill로 안정성을 확보하고, Agent로 능력을 확장하세요. 그리고 항상 기억하세요. "누가 결정하는가"가 가장 중요한 질문입니다.

당신이 통제할 수 있는 범위 내에서만 자율성을 허용하세요. 관찰할 수 없는 것은 운영하지 마세요. 신뢰를 쌓는 데는 시간이 걸리지만, 잃는 것은 한순간입니다.

성공적인 AI 시스템 구축을 기원합니다.

**작성 일자: 2026-01-10**

---
## 관련글

```
단상
MCP / Skill / Agent를 한 줄로 세우면, 구조가 의외로 단순해진다.
아래(연결) → 중간(기능) → 위(자율 실행).
 •MCP는 외부 도구를 부르는 표준 연결 규격이고
 •Skill은 특정 업무를 끝내는 정해진 레시피(워크플로우)고
 •Agent는 목표를 주면 스스로 계획·도구·반복을 돌리는 자율 실행자다.
핵심 차이는 결국 하나다.
“누가 결정하냐”
Skill은 사람이 흐름을 미리 짜 둔다.
검색하고, 요약하고, 정해진 포맷으로 보고서까지 만든다.
모델은 그 길을 “잘” 걷는 역할이다.
그래서 예측 가능하고, 감사 가능하고, 운영에서 강하다.
실무에서 필요한 건 대개 “똑똑함”보다 “신뢰”다.
Agent는 사람이 목표와 제약만 준다.
“이번 장애 원인 찾고 재발 방지안까지.”
그러면 에이전트가 계획을 만들고, 툴을 여러 번 돌리고, 중간 결과를 보고 스스로 수정하며 루프를 돈다.
더 강력하다. 대신 통제, 비용, 안정성을 설계하지 않으면 위험해진다.
강한 만큼, 경계도 더 필요하다.
현실은 보통 ‘선택’이 아니라 ‘조합’이다.
먼저 MCP로 도구를 표준화해서 붙인다.
그 위에 Skill로 안정적인 기능을 만든다.
그리고 예외가 많고 탐색이 필요한 영역만 Agent가 올라간다.
즉 에이전트는 대개
 •MCP 툴을 직접 쓰거나
 •Skill을 하나의 “고수준 툴”처럼 호출한다.
그래서 실무 감각도 이렇게 정리된다.
반복 업무이고 결과 포맷이 고정이고 실패하면 큰일이면 Skill.
예외가 많고 조사·탐색·추론이 핵심이면 Agent.
그리고 그 둘의 기반, 운영 가능성의 바닥은 항상 MCP다.
마지막으로 중요한 포인트는 이거다.
 •Skill은 신뢰를 만들고
 •Agent는 확장을 만들고
 •MCP는 운영 가능성을 만든다
똑똑함은 데모에서 빛나지만,
신뢰는 운영에서 살아남는다.

https://www.facebook.com/share/p/1AvwvZbFEj/
```