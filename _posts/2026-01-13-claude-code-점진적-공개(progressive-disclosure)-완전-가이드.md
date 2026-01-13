---
title: "Claude Code 점진적 공개(Progressive Disclosure) 완전 가이드"
date: 2026-01-13 19:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  progressive-disclosure,  MCP,  claude-skills,  Claude.write]
---


## 들어가며: AI 에이전트 개발의 패러다임 전환

AI 에이전트 기술이 급속도로 발전하면서, 개발자들은 점점 더 복잡하고 강력한 자동화 시스템을 구축하고자 합니다. 하지만 이러한 야심찬 목표를 달성하는 과정에서 한 가지 근본적인 문제에 직면하게 되었습니다. 바로 컨텍스트 윈도우의 한계입니다. 전통적인 AI 에이전트 설계 방식은 에이전트가 사용할 수 있는 모든 도구와 문서, 참조 자료를 처음부터 시스템 프롬프트에 포함시키는 것이었습니다. 이는 마치 누군가에게 백과사전 전체를 한 번에 읽게 한 뒤 질문에 답하라고 하는 것과 같았습니다.

Anthropic, Cloudflare, Cursor와 같은 선도적인 AI 기업들은 각자 독립적으로 연구를 진행하던 중 놀라운 사실을 발견했습니다. 그들은 모두 동일한 결론에 도달했는데, 그것은 바로 "필요한 정보를 필요한 순간에만 제공하는 것"이 훨씬 더 효율적이라는 것이었습니다. 이것이 바로 점진적 공개(Progressive Disclosure)의 핵심 개념입니다.

Claude Code는 이러한 인사이트를 실제 개발 워크플로우에 통합한 선구적인 사례입니다. 이 매뉴얼에서는 점진적 공개가 무엇인지, 왜 필요한지, 그리고 어떻게 구현하고 활용할 수 있는지에 대해 심도 있게 다룰 것입니다.

## 1. 점진적 공개의 본질: 문제 인식과 해결 방향

### 1.1 전통적 접근법의 한계

과거 AI 에이전트를 설계할 때, 개발자들은 직관적으로 "에이전트가 뭐든지 할 수 있게 하려면 모든 도구를 미리 알려줘야 한다"고 생각했습니다. 이는 표면적으로 합리적인 접근처럼 보였습니다. 예를 들어, 파일 작업, 데이터베이스 쿼리, API 호출, 코드 분석 등 50개의 도구가 있다면, 이 50개 도구의 스키마와 사용법을 모두 시스템 프롬프트에 포함시키는 것입니다.

하지만 이 방식은 여러 심각한 문제를 야기했습니다. 첫째, 컨텍스트 오버플로우 문제입니다. AI 모델의 컨텍스트 윈도우가 아무리 크다고 해도(현재 Claude 3.5는 200K 토큰까지 지원), 수백 개의 도구 정의와 문서를 담다 보면 정작 사용자의 질문이나 현재 작업 맥락이 묻혀버립니다. 모델이 주의를 기울여야 할 핵심 정보가 수많은 잡음 속에서 희석되는 것입니다.

둘째, 성능 저하 문제입니다. 연구 결과에 따르면, 컨텍스트가 지나치게 크면 모델의 정확도가 실제로 감소합니다. 이는 "lost in the middle" 현상으로 알려져 있는데, 모델이 컨텍스트의 중간 부분에 있는 정보를 잘 활용하지 못하는 경향을 말합니다. 또한 추론 시간도 길어져 응답 속도가 느려집니다.

셋째, 경제적 비효율성입니다. AI API는 일반적으로 입력 토큰 수에 비례하여 비용이 청구됩니다. 매 요청마다 수만 토큰의 도구 정의를 반복적으로 전송한다면, 실제 작업에 필요한 토큰은 얼마 안 되는데 비용은 기하급수적으로 증가하게 됩니다.

### 1.2 점진적 공개: 새로운 설계 철학

점진적 공개는 이러한 문제를 근본적으로 다른 관점에서 접근합니다. 핵심 아이디어는 간단합니다. "에이전트에게 모든 도구를 미리 주는 대신, 도구를 찾는 방법을 가르쳐라"는 것입니다. 이는 마치 도서관에서 필요한 책을 찾을 때, 모든 책을 한 번에 읽는 것이 아니라 카탈로그 시스템을 이용하여 필요한 책만 찾아 읽는 것과 같습니다.

구체적으로, 점진적 공개는 다음과 같은 원칙을 따릅니다. 에이전트는 처음에는 매우 제한된 컨텍스트로 시작합니다. 기본적인 파일 시스템 접근 능력(ls, cat, grep 등)과 "어디에 무엇이 있는지 찾는 방법"만 알고 있습니다. 실제 작업을 수행하다가 특정 기능이 필요해지면, 에이전트는 스스로 해당 기능을 찾아내고, 필요한 만큼만 컨텍스트에 로드하여 사용합니다. 작업이 끝나면 해당 정보는 다시 컨텍스트에서 제거될 수 있습니다.

이러한 접근법의 장점은 명확합니다. 컨텍스트는 항상 현재 작업에 필요한 최소한의 정보만 담고 있어 모델이 집중력을 유지할 수 있습니다. 토큰 사용량이 극적으로 감소하여 비용 효율성이 크게 향상됩니다. 그리고 에이전트가 수백, 수천 개의 도구를 가지고 있어도 컨텍스트가 폭발하지 않으므로 확장성이 뛰어납니다.

### 1.3 실증적 효과: 수치로 보는 개선

Claude Code에서 점진적 공개를 적용한 결과는 놀라웠습니다. 토큰 사용량은 기존 대비 85%에서 최대 98%까지 감소했습니다. 예를 들어, 기존에 100개의 도구를 모두 로드할 때 약 50,000 토큰이 소비되었다면, 점진적 공개를 사용하면 실제로 사용하는 5-10개 도구만 로드되어 1,000-7,500 토큰만 사용하게 됩니다.

응답 속도도 크게 개선되었습니다. 컨텍스트가 작아지면 모델이 처리해야 할 정보량이 줄어들어 첫 토큰 생성 시간(Time To First Token)이 단축됩니다. 실제 벤치마크에서 평균 응답 시간이 30-50% 감소한 것으로 나타났습니다.

가장 중요한 것은 정확도 향상입니다. 컨텍스트가 집중되어 있으면 모델이 관련 정보를 더 잘 파악하고 활용할 수 있습니다. 복잡한 다단계 작업에서의 성공률이 기존 대비 20-40% 향상되었다는 보고가 있습니다.

## 2. 핵심 메커니즘: 점진적 공개의 작동 원리

### 2.1 도구 검색 도구(Tool Discovery Tool)

점진적 공개의 가장 핵심적인 메커니즘은 "도구를 찾는 도구"입니다. 이는 메타 도구라고 할 수 있는데, 다른 도구들을 검색하고 발견하는 역할을 합니다. Claude Code에서 이는 주로 파일 시스템 탐색 능력으로 구현됩니다.

에이전트는 skills 디렉토리가 존재한다는 것을 알고 있습니다. 이 디렉토리 안에는 여러 서브디렉토리와 skill 파일들이 체계적으로 정리되어 있습니다. 각 skill 파일은 특정 기능이나 작업을 수행하는 방법을 담고 있습니다. 예를 들어, `database/postgres_query.md`는 PostgreSQL 데이터베이스 쿼리 방법을, `api/rest_client.md`는 REST API 호출 방법을 설명합니다.

에이전트가 데이터베이스 작업이 필요한 상황에 직면하면, 다음과 같은 과정을 거칩니다. 먼저 "데이터베이스 관련 기능이 필요하다"는 것을 인식합니다. 그런 다음 `ls skills/` 명령어로 skills 디렉토리의 구조를 파악합니다. `grep -r "database" skills/` 또는 `grep -r "postgres" skills/`로 관련 파일을 검색합니다. 발견된 파일의 front matter(메타데이터 섹션)를 읽어 정확히 원하는 기능인지 확인합니다. 적절하다고 판단되면 전체 파일을 읽어들여 해당 기능을 사용합니다.

이 과정은 매우 효율적입니다. 수백 개의 skill 파일이 있어도, 에이전트는 필요한 것만 선택적으로 로드하므로 컨텍스트가 항상 관리 가능한 수준으로 유지됩니다.

### 2.2 Front Matter 기반 메타데이터 시스템

각 skill 파일은 YAML 형식의 front matter로 시작합니다. 이는 파일의 전체 내용을 읽지 않고도 해당 skill이 무엇을 하는지 빠르게 파악할 수 있게 해줍니다. 일반적인 front matter는 다음과 같은 정보를 포함합니다.

```yaml
---
name: postgres_query
description: PostgreSQL 데이터베이스에 쿼리를 실행하고 결과를 처리합니다
category: database
tags: [sql, postgres, query, data]
dependencies: [psycopg2, connection_config]
complexity: medium
estimated_tokens: 2500
---
```

이러한 메타데이터는 에이전트가 효율적으로 의사결정을 내리는 데 도움을 줍니다. description을 통해 기능의 목적을 빠르게 이해할 수 있고, tags를 통해 관련 키워드로 검색할 수 있습니다. dependencies를 보고 선행 요구사항이 있는지 확인할 수 있으며, estimated_tokens로 이 skill을 로드했을 때 컨텍스트에 미칠 영향을 예측할 수 있습니다.

에이전트는 일반적으로 다음과 같은 전략을 사용합니다. 먼저 `find skills/ -name "*.md"` 명령으로 모든 skill 파일 목록을 얻습니다. 그런 다음 각 파일의 처음 몇 줄(front matter 부분)만 읽어 빠르게 스캔합니다. 현재 작업과 관련성이 높은 것으로 판단되는 skill만 전체를 로드합니다.

이 방식의 핵심은 "완전한 정보보다 충분한 정보"를 우선한다는 것입니다. 에이전트는 처음부터 완벽한 이해를 추구하지 않고, 빠르게 탐색하고 필요할 때 깊이 들어가는 방식을 취합니다.

### 2.3 Bash를 활용한 동적 컨텍스트 관리

Claude Code에서 Bash의 역할은 단순히 명령을 실행하는 것을 넘어섭니다. Bash는 에이전트가 자신의 컨텍스트를 능동적으로 관리할 수 있게 해주는 핵심 인터페이스입니다.

grep, find, ls, cat과 같은 기본 유닉스 명령어들은 에이전트에게 강력한 탐색 능력을 제공합니다. 예를 들어, 에이전트가 "API 호출"과 관련된 기능을 찾고 싶다면 `grep -r "API\|REST\|HTTP" skills/ --include="*.md" | head -20` 같은 명령을 실행할 수 있습니다. 이는 skills 디렉토리 전체를 검색하되, 처음 20개 결과만 가져옴으로써 컨텍스트 오버플로우를 방지합니다.

또한 에이전트는 파일의 일부만 선택적으로 읽을 수 있습니다. `head -n 50 skills/complex_skill.md`로 파일의 처음 50줄만 읽어 개요를 파악하거나, `sed -n '/^## Installation/,/^## Usage/p' skills/tool.md`로 특정 섹션만 추출할 수 있습니다.

이러한 Bash 기반 접근법의 장점은 유연성입니다. 에이전트는 하드코딩된 도구 발견 로직에 의존하지 않고, 상황에 맞게 적절한 검색 전략을 즉석에서 구성할 수 있습니다. 또한 새로운 skill이 추가되거나 기존 skill이 수정되어도, 에이전트는 자동으로 이를 발견하고 활용할 수 있습니다.

### 2.4 Model Context Protocol (MCP)의 진화

Model Context Protocol은 AI 에이전트가 외부 시스템과 상호작용하는 표준화된 방식을 정의합니다. 전통적으로 MCP를 사용할 때는 모든 도구의 스키마를 처음부터 등록하고, 이를 시스템 프롬프트에 포함시켰습니다.

하지만 점진적 공개 패러다임에서 MCP의 활용 방식이 변화하고 있습니다. 이제는 "MCP 도구 탐색기(MCP Tool Explorer)"라는 메타 도구를 통해 사용 가능한 MCP 도구들을 동적으로 발견합니다. 에이전트는 필요한 순간에만 특정 MCP 도구의 스키마를 요청하고 로드합니다. 도구 사용이 끝나면 해당 스키마는 컨텍스트에서 제거될 수 있습니다.

예를 들어, GitHub MCP 서버를 사용하는 경우를 생각해봅시다. 기존 방식이라면 GitHub의 모든 API 엔드포인트(리포지토리 조회, 이슈 생성, PR 관리, Actions 실행 등)를 처음부터 모두 로드했을 것입니다. 점진적 공개 방식에서는 에이전트가 "GitHub 이슈 생성이 필요하다"고 판단했을 때만 `mcp_discover("github", "create_issue")` 같은 명령으로 해당 도구만 로드합니다.

이는 MCP 생태계의 확장성을 크게 향상시킵니다. 수십 개의 MCP 서버를 통합하더라도 컨텍스트 문제가 발생하지 않기 때문입니다.

## 3. Skills 시스템: Claude Code의 구현 세부사항

### 3.1 Skills 디렉토리 구조

Claude Code의 skills 디렉토리는 체계적으로 조직된 지식 베이스입니다. 일반적인 구조는 다음과 같습니다.

```
skills/
├── README.md                    # Skills 시스템 전체 개요
├── public/                      # Anthropic이 제공하는 공식 skills
│   ├── web/
│   │   ├── scraping.md
│   │   └── api_integration.md
│   ├── data/
│   │   ├── csv_processing.md
│   │   └── json_manipulation.md
│   └── code/
│       ├── refactoring.md
│       └── testing.md
├── private/                     # 사용자가 추가한 비공개 skills
│   └── company_specific/
│       └── internal_api.md
└── examples/                    # 참고용 예제 skills
    └── skill_template.md
```

이 구조는 몇 가지 중요한 원칙을 반영합니다. 첫째, 공개/비공개 분리입니다. public skills는 모든 사용자가 공유할 수 있는 범용 기능이고, private skills는 개인이나 조직 특화 기능입니다. 둘째, 카테고리화입니다. 관련 skills를 서브디렉토리로 그룹화하여 탐색을 용이하게 합니다. 셋째, 템플릿 제공입니다. 사용자가 자신만의 skill을 쉽게 만들 수 있도록 예제와 템플릿을 제공합니다.

### 3.2 Skill 파일 작성 규칙

효과적인 skill 파일을 작성하려면 몇 가지 규칙을 따라야 합니다. 가장 중요한 것은 간결성입니다. 각 skill 파일은 300줄을 넘지 않는 것이 권장됩니다. 이는 토큰 사용량을 관리 가능한 수준으로 유지하고, 에이전트가 빠르게 이해할 수 있게 합니다.

보편성도 중요합니다. Skill은 특정 프로젝트나 작업에 종속되지 않고, 다양한 상황에서 재사용 가능해야 합니다. 예를 들어, "프로젝트 X의 Y 기능 구현하기"보다는 "REST API 클라이언트 만들기"처럼 일반화된 지식을 담아야 합니다.

구조적 일관성도 필수입니다. 모든 skill은 동일한 섹션 구조를 따라야 에이전트가 예측 가능하게 정보를 찾을 수 있습니다. 일반적인 구조는 다음과 같습니다.

```markdown
---
name: skill_name
description: 간단한 한 줄 설명
category: 카테고리명
tags: [키워드1, 키워드2]
---

# Skill 이름

## 개요
이 skill이 해결하는 문제와 언제 사용해야 하는지 설명

## 사전 요구사항
필요한 도구, 라이브러리, 설정 등

## 사용 방법
단계별 지침과 코드 예제

## 예제
실제 사용 사례

## 주의사항
흔한 실수나 제약사항

## 참고자료
관련 문서 링크
```

### 3.3 Progressive Context Loading의 실제 동작

에이전트가 실제로 어떻게 점진적으로 컨텍스트를 로드하는지 구체적인 시나리오를 통해 살펴보겠습니다.

사용자가 "웹사이트에서 데이터를 스크래핑하고 CSV로 저장해줘"라고 요청했다고 가정합니다. 에이전트는 다음과 같은 사고 과정을 거칩니다.

첫 번째 단계에서 에이전트는 작업을 분석합니다. "웹 스크래핑"과 "CSV 저장" 두 가지 주요 작업이 필요하다는 것을 파악합니다. 

두 번째 단계에서 관련 skills를 탐색합니다. `ls skills/public/web/` 명령으로 웹 관련 skills가 무엇이 있는지 확인합니다. `grep -l "scrap" skills/public/web/*.md` 명령으로 스크래핑 관련 파일을 찾습니다.

세 번째 단계에서 `skills/public/web/scraping.md`의 front matter를 읽어 이것이 필요한 skill인지 확인합니다. 적합하다고 판단되면 전체 파일을 읽어 컨텍스트에 로드합니다.

네 번째 단계에서 스크래핑 작업을 수행합니다. 이때 컨텍스트에는 현재 작업에 필요한 스크래핑 skill만 있습니다.

다섯 번째 단계에서 이제 CSV 저장이 필요합니다. `find skills/ -name "*csv*"` 명령으로 CSV 관련 skill을 찾습니다. `skills/public/data/csv_processing.md`를 로드합니다.

여섯 번째 단계에서 CSV로 저장하는 작업을 완료합니다. 이제 컨텍스트에는 스크래핑과 CSV 처리 두 개의 skills만 로드되어 있습니다.

이 과정에서 에이전트는 수백 개의 사용 가능한 skills 중 실제로 필요한 두 개만 선택적으로 사용했습니다. 만약 모든 skills를 처음부터 로드했다면 수만 토큰이 필요했을 것이지만, 점진적 공개 방식으로는 5,000-7,000 토큰 정도만 사용했을 것입니다.

### 3.4 Skill 간 의존성 관리

일부 skills는 다른 skills에 의존할 수 있습니다. 예를 들어, "데이터베이스 마이그레이션 실행" skill은 "데이터베이스 연결 설정" skill을 필요로 할 수 있습니다. 점진적 공개 시스템에서는 이러한 의존성을 어떻게 관리할까요?

Front matter의 dependencies 필드가 이를 해결합니다. 에이전트가 특정 skill을 로드할 때, dependencies를 확인하고 필요한 경우 의존하는 skills도 함께 로드합니다. 이는 재귀적으로 작동하여 전체 의존성 트리를 해결합니다.

```yaml
---
name: database_migration
dependencies: [database_connection, schema_validator]
---
```

에이전트가 database_migration skill을 로드하면, 자동으로 database_connection과 schema_validator도 로드됩니다. 하지만 이것도 필요한 순간에만 이루어지므로, 사용자가 마이그레이션 작업을 요청하기 전까지는 이 어떤 skill도 컨텍스트에 없습니다.

## 4. 메모리 관리: 단순함의 힘

### 4.1 복잡한 RAG 시스템을 버리다

많은 AI 에이전트 시스템은 Retrieval-Augmented Generation (RAG)을 메모리 관리에 사용합니다. 텍스트를 벡터로 임베딩하고, 벡터 데이터베이스에 저장하며, 유사도 검색으로 관련 정보를 찾는 방식입니다. 이는 이론적으로 강력해 보이지만, 실제로는 여러 문제가 있습니다.

벡터 검색은 의미적 유사성을 기반으로 하므로, 때로는 관련 없는 정보를 가져오거나 정작 필요한 정보를 놓칠 수 있습니다. 임베딩 모델의 성능에 따라 검색 품질이 크게 달라집니다. 또한 벡터 데이터베이스를 관리하고 유지하는 것은 추가적인 복잡성과 비용을 발생시킵니다.

Claude Code는 이와 다른 접근법을 취합니다. 바로 단순한 Markdown 파일을 사용하는 것입니다. 에이전트는 memory.md 같은 파일에 자신의 작업 내역, 배운 점, 중요한 컨텍스트를 직접 기록합니다. 필요할 때는 이 파일을 읽어 과거 정보를 상기합니다.

### 4.2 Self-Documenting Agent

이 방식의 핵심은 에이전트가 스스로 자신의 메모리를 관리한다는 것입니다. 에이전트는 작업을 수행하면서 중요한 정보를 자동으로 memory.md에 추가합니다. 예를 들어:

```markdown
# Memory

## 프로젝트 정보
- 프로젝트명: OrderManagementSystem
- 메인 언어: Python 3.9
- 프레임워크: FastAPI
- 데이터베이스: PostgreSQL 14

## 최근 작업
### 2026-01-10: API 엔드포인트 리팩토링
- /api/orders 엔드포인트를 async/await 패턴으로 변경
- 응답 시간 30% 개선
- 다음 단계: 테스트 케이스 작성 필요

## 발견한 이슈
- 데이터베이스 커넥션 풀 크기가 부족함
- config.py에서 max_connections를 20으로 증가시킴

## 중요한 결정사항
- 인증 방식은 JWT 사용하기로 결정
- 토큰 만료 시간: 1시간
```

이러한 메모리는 구조화되어 있지만 동시에 매우 유연합니다. 에이전트는 자유롭게 섹션을 추가하고, 내용을 업데이트하며, 더 이상 필요 없는 정보는 삭제할 수 있습니다.

### 4.3 컨텍스트 압축 전략

시간이 지나면서 memory.md가 커질 수 있습니다. 이는 다시 컨텍스트 문제를 야기할 수 있습니다. 점진적 공개는 여기에도 적용됩니다.

에이전트는 주기적으로 메모리를 압축합니다. 오래된 작업 내역은 요약하고, 더 이상 관련 없는 정보는 아카이브 파일로 이동시킵니다. 예를 들어, 6개월 전의 작업 상세 내역은 memory_archive_2025.md로 옮기고, 핵심 요약만 현재 메모리에 남깁니다.

또한 에이전트는 메모리를 주제별로 분할할 수 있습니다. memory_api.md, memory_database.md, memory_ui.md 처럼 도메인별로 파일을 나누면, 특정 작업에는 관련된 메모리 파일만 로드하면 됩니다.

### 4.4 Grep을 이용한 빠른 메모리 검색

벡터 검색 대신, 에이전트는 grep 같은 전통적인 텍스트 검색 도구를 사용합니다. 이는 놀랍도록 효과적입니다.

"API 인증 관련해서 뭔가 했던 것 같은데..." 같은 상황에서, 에이전트는 `grep -i "auth\|jwt\|token" memory.md`로 빠르게 관련 내역을 찾을 수 있습니다. 정확한 키워드를 기억한다면, 벡터 검색보다 grep이 훨씬 더 정확하고 빠릅니다.

또한 정규표현식을 활용하면 매우 정교한 검색이 가능합니다. `grep -E "database.*connection.*pool" memory.md`는 "database", "connection", "pool"이 순서대로 나타나는 줄을 찾아줍니다.

## 5. 구현 가이드: 점진적 공개 시스템 구축하기

### 5.1 기본 설정

자신의 프로젝트에 점진적 공개를 적용하려면 먼저 디렉토리 구조를 설정해야 합니다. 프로젝트 루트에 skills 디렉토리를 만들고, 적절한 서브디렉토리를 구성합니다.

```bash
mkdir -p skills/public/{web,data,code,api}
mkdir -p skills/private
mkdir -p skills/examples
```

그런 다음 기본 README를 작성합니다.

```markdown
# Skills Directory

이 디렉토리는 AI 에이전트가 사용할 수 있는 skills를 담고 있습니다.

## 구조
- public/: 범용 skills
- private/: 프로젝트 특화 skills
- examples/: 참고용 예제

## Skill 추가하기
1. 적절한 카테고리 디렉토리 선택
2. skill_template.md를 복사
3. Front matter와 내용 작성
4. 300줄 이하로 유지
```

### 5.2 첫 번째 Skill 만들기

실제로 skill을 하나 만들어보겠습니다. 간단한 웹 스크래핑 skill을 예로 들겠습니다.

~~~markdown
---
name: web_scraping_basic
description: BeautifulSoup을 사용한 기본 웹 스크래핑
category: web
tags: [scraping, beautifulsoup, html, parsing]
dependencies: []
complexity: easy
estimated_tokens: 1500
---

# 기본 웹 스크래핑

## 개요

이 skill은 Python의 BeautifulSoup 라이브러리를 사용하여 웹페이지에서 데이터를 추출하는 방법을 설명합니다. 정적 HTML 페이지에 적합하며, JavaScript로 렌더링되는 동적 콘텐츠에는 Selenium이나 Playwright를 사용해야 합니다.

## 사전 요구사항

다음 패키지가 설치되어 있어야 합니다:
```bash
pip install requests beautifulsoup4 lxml
```

## 기본 사용법

웹 스크래핑의 기본 흐름은 다음과 같습니다. 먼저 requests를 사용하여 웹페이지의 HTML을 가져옵니다. 그런 다음 BeautifulSoup으로 HTML을 파싱합니다. CSS 선택자나 태그 이름으로 원하는 요소를 찾습니다. 마지막으로 텍스트나 속성 값을 추출합니다.

```python
import requests
from bs4 import BeautifulSoup

# 웹페이지 가져오기
response = requests.get('https://example.com')
response.raise_for_status()  # 에러 확인

# HTML 파싱
soup = BeautifulSoup(response.text, 'lxml')

# 제목 추출
title = soup.find('h1').text.strip()

# 모든 링크 추출
links = [a['href'] for a in soup.find_all('a', href=True)]

# CSS 선택자 사용
articles = soup.select('.article-content p')
paragraphs = [p.text.strip() for p in articles]
```

## 고급 기법

여러 페이지를 순회해야 하는 경우, 페이지네이션을 처리해야 합니다. 다음 페이지 링크를 찾아 재귀적으로 방문할 수 있습니다.

```python
def scrape_all_pages(base_url):
    results = []
    current_url = base_url
    
    while current_url:
        response = requests.get(current_url)
        soup = BeautifulSoup(response.text, 'lxml')
        
        # 현재 페이지 데이터 추출
        items = soup.find_all('div', class_='item')
        results.extend([item.text for item in items])
        
        # 다음 페이지 찾기
        next_link = soup.find('a', class_='next')
        current_url = next_link['href'] if next_link else None
    
    return results
```

## 주의사항

웹 스크래핑 시 반드시 고려해야 할 사항들이 있습니다. robots.txt 파일을 확인하여 스크래핑이 허용되는지 확인하세요. 요청 간 적절한 지연을 두어 서버에 부담을 주지 않도록 합니다. User-Agent 헤더를 설정하여 봇임을 명시하거나 실제 브라우저처럼 보이게 할 수 있습니다.

```python
headers = {
    'User-Agent': 'MyBot/1.0 (contact@example.com)'
}
response = requests.get(url, headers=headers)
```

웹사이트의 이용약관을 확인하고 준수하세요. 일부 사이트는 스크래핑을 명시적으로 금지합니다.

## 참고자료

- BeautifulSoup 공식 문서: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
- Requests 라이브러리: https://docs.python-requests.org/
~~~

이제 에이전트가 `grep -r "scraping" skills/`를 실행하면 이 파일을 발견할 수 있고, front matter를 읽어 웹 스크래핑 기능임을 파악한 후, 필요하다면 전체 내용을 로드하여 사용할 수 있습니다.

### 5.3 에이전트에게 Skills 사용법 가르치기

에이전트가 skills 시스템을 효과적으로 활용하도록 하려면, 시스템 프롬프트에 다음과 같은 지침을 포함해야 합니다.

```
당신은 skills 디렉토리에 접근할 수 있습니다. 이 디렉토리에는 다양한 작업을 수행하는 방법을 담은 문서들이 있습니다.

작업을 수행하기 전에, 관련 skill이 있는지 확인하세요:
1. `ls skills/public/` 또는 해당 카테고리 디렉토리를 확인
2. `grep -r "키워드" skills/`로 관련 파일 검색
3. 발견된 파일의 front matter (---로 둘러싸인 부분)를 읽어 내용 파악
4. 적절한 skill이면 전체 파일을 읽고 지침을 따름

모든 도구를 처음부터 로드하려 하지 마세요. 필요한 것만 찾아서 사용하세요.
```

### 5.4 점진적 공개를 위한 코드 패턴

실제 코드에서 점진적 공개를 구현하려면 다음과 같은 패턴을 사용할 수 있습니다.

```python
class ProgressiveDisclosureAgent:
    def __init__(self, skills_dir="./skills"):
        self.skills_dir = skills_dir
        self.loaded_skills = {}
        self.skill_metadata = self._index_skills()
    
    def _index_skills(self):
        """모든 skill의 메타데이터만 먼저 인덱싱"""
        import os
        import yaml
        
        metadata = {}
        for root, dirs, files in os.walk(self.skills_dir):
            for file in files:
                if file.endswith('.md'):
                    filepath = os.path.join(root, file)
                    with open(filepath, 'r') as f:
                        content = f.read()
                        # Front matter 추출
                        if content.startswith('---'):
                            end = content.find('---', 3)
                            if end != -1:
                                front_matter = content[3:end]
                                meta = yaml.safe_load(front_matter)
                                meta['filepath'] = filepath
                                metadata[meta['name']] = meta
        return metadata
    
    def find_skills(self, query):
        """쿼리와 관련된 skills 찾기"""
        relevant = []
        query_lower = query.lower()
        
        for name, meta in self.skill_metadata.items():
            # 이름, 설명, 태그에서 검색
            if (query_lower in meta['name'].lower() or
                query_lower in meta['description'].lower() or
                any(query_lower in tag.lower() for tag in meta.get('tags', []))):
                relevant.append(meta)
        
        return sorted(relevant, key=lambda x: x.get('complexity', 'medium'))
    
    def load_skill(self, skill_name):
        """필요한 skill만 로드"""
        if skill_name in self.loaded_skills:
            return self.loaded_skills[skill_name]
        
        if skill_name not in self.skill_metadata:
            raise ValueError(f"Skill not found: {skill_name}")
        
        filepath = self.skill_metadata[skill_name]['filepath']
        with open(filepath, 'r') as f:
            content = f.read()
        
        self.loaded_skills[skill_name] = content
        return content
    
    def execute_task(self, user_request):
        """사용자 요청 처리"""
        # 1. 관련 skills 찾기
        relevant_skills = self.find_skills(user_request)
        
        # 2. 가장 적합한 skill 선택 (예: 첫 번째)
        if relevant_skills:
            skill_name = relevant_skills[0]['name']
            skill_content = self.load_skill(skill_name)
            
            # 3. skill 내용과 함께 LLM에 요청
            prompt = f"""
            다음 skill을 참고하여 작업을 수행하세요:
            
            {skill_content}
            
            사용자 요청: {user_request}
            """
            
            # LLM 호출 (실제 구현에서는 API 호출)
            response = self.call_llm(prompt)
            return response
        else:
            # skill이 없으면 일반적인 방법으로 처리
            return self.call_llm(user_request)
    
    def call_llm(self, prompt):
        # 실제 LLM API 호출
        pass
```

이 패턴의 핵심은 _index_skills 메서드입니다. 처음에 모든 skill의 메타데이터만 메모리에 로드하고, 실제 내용은 필요할 때만 load_skill로 가져옵니다. 이렇게 하면 수백 개의 skills가 있어도 초기 메모리 사용량은 최소화됩니다.

## 6. 최적화 전략과 베스트 프랙티스

### 6.1 토큰 사용량 모니터링

점진적 공개의 효과를 측정하려면 토큰 사용량을 추적해야 합니다. 각 요청에서 얼마나 많은 토큰이 사용되었는지, 그 중 몇 개가 skills 관련 컨텍스트였는지 기록합니다.

```python
import anthropic

client = anthropic.Anthropic(api_key="your-key")

def track_token_usage(prompt, response):
    input_tokens = response.usage.input_tokens
    output_tokens = response.usage.output_tokens
    
    print(f"입력 토큰: {input_tokens}")
    print(f"출력 토큰: {output_tokens}")
    print(f"총 토큰: {input_tokens + output_tokens}")
    
    # 로그에 기록
    log_metrics({
        'input_tokens': input_tokens,
        'output_tokens': output_tokens,
        'timestamp': datetime.now()
    })
```

시간이 지남에 따라 토큰 사용량의 추세를 분석하여, 점진적 공개가 실제로 효과가 있는지 확인할 수 있습니다. 기존 방식과 비교하여 80% 이상의 토큰 절감이 목표입니다.

### 6.2 Skill 캐싱 전략

자주 사용되는 skills는 캐싱하여 반복적인 파일 읽기를 피할 수 있습니다. 하지만 캐시가 너무 커지면 메모리 문제가 발생할 수 있으므로, LRU (Least Recently Used) 캐시를 사용하는 것이 좋습니다.

```python
from functools import lru_cache

class SkillManager:
    @lru_cache(maxsize=20)
    def load_skill(self, skill_name):
        """최근 20개 skill을 캐시"""
        with open(f"skills/{skill_name}.md", 'r') as f:
            return f.read()
```

이렇게 하면 같은 skill을 여러 번 사용해도 파일을 한 번만 읽습니다. maxsize=20은 대부분의 사용 패턴에 적절한 값입니다.

### 6.3 Skill 버전 관리

Skills는 시간이 지남에 따라 업데이트될 수 있습니다. 버전 관리를 통해 변경사항을 추적하고, 필요시 이전 버전으로 롤백할 수 있습니다.

```yaml
---
name: web_scraping_basic
version: 2.1.0
changelog:
  - 2.1.0: Playwright 예제 추가
  - 2.0.0: 비동기 처리 지원
  - 1.0.0: 초기 버전
---
```

Git을 사용하여 skills 디렉토리를 버전 관리하는 것이 가장 간단한 방법입니다. 팀에서 협업할 때는 각자 작성한 skills를 PR을 통해 리뷰하고 머지할 수 있습니다.

### 6.4 Skill 테스트

Skill이 제대로 작동하는지 확인하려면 테스트가 필요합니다. 각 skill에 대해 간단한 테스트 케이스를 작성할 수 있습니다.

~~~markdown
## 테스트

다음 명령어로 이 skill을 테스트할 수 있습니다:

```bash
python -m pytest tests/test_web_scraping.py
```

테스트 케이스:
1. 정적 HTML 페이지 파싱
2. CSS 선택자로 요소 추출
3. 여러 페이지 순회
4. 에러 처리 (404, timeout 등)
~~~

자동화된 테스트를 통해 skill이 업데이트되어도 기존 기능이 깨지지 않았는지 확인할 수 있습니다.

## 7. 고급 활용: 엔터프라이즈 환경에서의 점진적 공개

### 7.1 대규모 Skill 라이브러리 관리

기업 환경에서는 수백, 수천 개의 skills가 축적될 수 있습니다. 이를 효과적으로 관리하려면 체계적인 조직이 필요합니다.

카테고리 계층 구조를 활용하세요. skills/engineering/backend/database/postgresql/ 같이 여러 레벨의 디렉토리를 사용하여 skills를 세밀하게 분류합니다. 명명 규칙을 표준화하세요. 모든 skill 파일은 동사_명사.md 형식을 따르도록 합니다. 예를 들어, deploy_docker_container.md, query_postgres_database.md 같은 식입니다.

메타데이터를 풍부하게 유지하세요. owner, last_updated, usage_count 같은 필드를 front matter에 추가하여 skill 관리를 용이하게 합니다.

```yaml
---
name: deploy_k8s_service
owner: devops-team@company.com
last_updated: 2026-01-10
usage_count: 234
approval_required: true
---
```

### 7.2 접근 제어와 보안

민감한 정보를 다루는 skills는 접근 제어가 필요합니다. 다음과 같은 방법을 고려할 수 있습니다.

Skills를 public, internal, confidential로 분류합니다. 각 레벨에 따라 누가 접근할 수 있는지 정의합니다. 예를 들어, confidential skills는 특정 팀만 사용할 수 있습니다.

Skill 로딩 시 권한을 확인합니다.

```python
def load_skill(self, skill_name, user_id):
    meta = self.skill_metadata[skill_name]
    
    if meta.get('confidential') and user_id not in meta.get('authorized_users', []):
        raise PermissionError(f"User {user_id} not authorized for {skill_name}")
    
    return self._read_skill_file(meta['filepath'])
```

민감한 데이터는 환경 변수나 비밀 관리 시스템(예: AWS Secrets Manager)을 통해 주입하고, skill 파일 자체에는 포함하지 않습니다.

### 7.3 Skill 분석과 최적화

어떤 skills가 자주 사용되는지, 어떤 것은 전혀 사용되지 않는지 추적하면 라이브러리를 최적화할 수 있습니다.

사용 통계를 수집합니다.

```python
def load_skill(self, skill_name):
    # 사용 기록
    self.analytics.record_skill_usage(skill_name, datetime.now())
    
    return self._read_skill_file(skill_name)
```

주기적으로 리포트를 생성합니다.

```python
def generate_usage_report():
    top_skills = analytics.get_top_skills(limit=20)
    unused_skills = analytics.get_unused_skills(days=90)
    
    report = f"""
    ## Skill 사용 통계 (최근 30일)
    
    가장 많이 사용된 Skills:
    {format_list(top_skills)}
    
    90일간 사용되지 않은 Skills:
    {format_list(unused_skills)}
    
    권장 사항:
    - 자주 사용되는 skills는 최적화 우선순위
    - 사용되지 않는 skills는 아카이브 검토
    """
    
    return report
```

### 7.4 멀티 에이전트 시스템에서의 점진적 공개

여러 AI 에이전트가 협업하는 시스템에서, 각 에이전트는 자신의 전문 분야에 맞는 skills만 접근하면 됩니다.

예를 들어, 코드 리뷰 에이전트는 code analysis, security checking 관련 skills만 보고, 데이터 분석 에이전트는 data processing, visualization 관련 skills만 봅니다. 이렇게 하면 각 에이전트의 컨텍스트가 더욱 집중되고 효율적입니다.

```python
class SpecializedAgent:
    def __init__(self, role):
        self.role = role
        self.skill_categories = self._get_categories_for_role(role)
    
    def _get_categories_for_role(self, role):
        role_mapping = {
            'code_reviewer': ['code', 'security', 'testing'],
            'data_analyst': ['data', 'analytics', 'visualization'],
            'devops': ['deployment', 'monitoring', 'infrastructure']
        }
        return role_mapping.get(role, [])
    
    def find_skills(self, query):
        # 자신의 카테고리 내에서만 검색
        relevant = []
        for cat in self.skill_categories:
            cat_path = f"skills/{cat}/"
            # grep을 사용하여 해당 카테고리만 검색
            results = subprocess.run(
                ['grep', '-r', query, cat_path],
                capture_output=True, text=True
            )
            relevant.extend(self._parse_grep_results(results.stdout))
        
        return relevant
```

## 8. 문제 해결과 디버깅

### 8.1 흔한 문제들

**문제 1: 에이전트가 관련 skill을 찾지 못함**

원인: 검색 키워드가 skill의 메타데이터나 내용과 매칭되지 않습니다.

해결책: Skill의 tags 필드를 더 포괄적으로 작성하세요. 동의어와 관련 용어를 모두 포함합니다. 예를 들어, 웹 스크래핑 skill이라면 [scraping, crawling, web parsing, data extraction, html parsing] 같이 다양한 표현을 포함합니다.

**문제 2: 컨텍스트가 여전히 너무 큼**

원인: 의존성이 있는 skills를 로드하면서 연쇄적으로 많은 skills가 로드되었습니다.

해결책: 의존성 트리를 검토하고, 불필요한 의존성을 제거하세요. 또한 각 skill의 크기를 점검하여 300줄 제한을 준수하는지 확인합니다.

**문제 3: Skill 내용이 오래됨**

원인: Skill이 업데이트되지 않아 최신 라이브러리나 베스트 프랙티스를 반영하지 못합니다.

해결책: 정기적인 리뷰 프로세스를 수립하세요. 각 skill에 last_reviewed 필드를 추가하고, 6개월마다 리뷰를 예약합니다.

### 8.2 디버깅 도구

점진적 공개 시스템을 디버깅하려면 다음과 같은 도구가 유용합니다.

**로깅**: 어떤 skills가 언제 로드되었는지 상세히 기록합니다.

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

def load_skill(self, skill_name):
    logger.debug(f"Loading skill: {skill_name}")
    start_time = time.time()
    
    content = self._read_skill_file(skill_name)
    
    load_time = time.time() - start_time
    logger.debug(f"Loaded {skill_name} in {load_time:.2f}s, {len(content)} chars")
    
    return content
```

**시각화**: Skill 로딩 패턴을 시각화하여 병목 지점을 파악합니다.

```python
import matplotlib.pyplot as plt

def visualize_skill_usage():
    usage_data = analytics.get_daily_skill_usage()
    
    plt.figure(figsize=(12, 6))
    plt.bar(usage_data.keys(), usage_data.values())
    plt.xlabel('Skill')
    plt.ylabel('Usage Count')
    plt.title('Skill Usage Over Time')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.savefig('skill_usage.png')
```

### 8.3 성능 프로파일링

어디에 시간이 소요되는지 정확히 파악하려면 프로파일링이 필요합니다.

```python
import cProfile
import pstats

def profile_skill_loading():
    profiler = cProfile.Profile()
    profiler.enable()
    
    # 측정할 작업
    agent = ProgressiveDisclosureAgent()
    agent.execute_task("웹사이트에서 데이터 스크래핑")
    
    profiler.disable()
    
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(20)  # 상위 20개 함수 출력
```

이를 통해 파일 I/O, 검색, LLM 호출 등 각 단계에서 소요되는 시간을 정확히 측정할 수 있습니다.

## 9. 미래 전망과 발전 방향

### 9.1 자동 Skill 생성

미래에는 AI가 스스로 새로운 skills를 생성할 수 있을 것입니다. 에이전트가 새로운 작업을 성공적으로 수행하면, 그 과정을 자동으로 문서화하여 새로운 skill로 저장합니다.

```python
def auto_generate_skill(task_description, execution_steps, outcome):
    skill_template = f"""---
name: {generate_skill_name(task_description)}
description: {task_description}
generated: {datetime.now().isoformat()}
verified: false
---

# {task_description}

## 자동 생성된 Skill

이 skill은 AI가 작업을 수행하면서 자동으로 생성되었습니다.

## 실행 단계

{format_steps(execution_steps)}

## 결과

{outcome}

## 검토 필요

이 skill은 아직 인간의 검토를 거치지 않았습니다.
"""
    
    filepath = f"skills/auto_generated/{generate_filename(task_description)}.md"
    with open(filepath, 'w') as f:
        f.write(skill_template)
    
    return filepath
```

물론 자동 생성된 skills는 인간의 검토가 필요합니다. 하지만 초안을 자동으로 만들어주면 문서화 부담이 크게 줄어듭니다.

### 9.2 지능형 Skill 추천

사용 패턴을 학습하여, 에이전트가 사용자의 의도를 더 잘 파악하고 적절한 skill을 추천할 수 있습니다.

```python
class IntelligentSkillRecommender:
    def __init__(self):
        self.usage_history = []
        self.context_embeddings = {}
    
    def recommend_skills(self, current_context):
        # 현재 컨텍스트와 유사한 과거 상황 찾기
        similar_contexts = self.find_similar_contexts(current_context)
        
        # 유사한 상황에서 사용된 skills 추출
        recommended = []
        for ctx in similar_contexts:
            recommended.extend(ctx['skills_used'])
        
        # 빈도 순으로 정렬
        from collections import Counter
        skill_counts = Counter(recommended)
        
        return [skill for skill, count in skill_counts.most_common(5)]
```

### 9.3 크로스 플랫폼 Skill 공유

Skills가 표준화되면, 다른 플랫폼이나 도구 간에도 공유할 수 있습니다. Claude Code에서 작성한 skill을 ChatGPT의 Custom GPT나 다른 AI 에이전트 플랫폼에서도 사용할 수 있게 됩니다.

이를 위해서는 skill 형식의 표준화가 필요합니다. OpenAI, Anthropic, Google 등이 협력하여 공통 skill 형식을 정의할 수 있습니다.

```yaml
---
standard: AI-Skill-Format-1.0
name: web_scraping_basic
platform_compatibility: [claude, gpt, gemini]
---
```

### 9.4 동적 Skill 조합

여러 skills를 조합하여 새로운 기능을 만드는 것도 가능합니다. 예를 들어, "웹 스크래핑" + "데이터 정제" + "CSV 저장" skills를 자동으로 조합하여 "웹에서 데이터 수집 및 저장" 워크플로우를 구성합니다.

```python
def compose_skills(skill_names):
    """여러 skills를 하나의 워크플로우로 조합"""
    workflow = {
        'name': '_'.join(skill_names),
        'steps': []
    }
    
    for skill_name in skill_names:
        skill = load_skill(skill_name)
        workflow['steps'].append({
            'skill': skill_name,
            'content': skill
        })
    
    return workflow
```

## 10. 결론: 점진적 공개가 가져올 변화

점진적 공개는 단순한 최적화 기법을 넘어, AI 에이전트 개발의 패러다임을 바꾸는 철학입니다. 과거에는 "에이전트에게 모든 것을 주자"는 생각이 지배적이었다면, 이제는 "에이전트가 스스로 필요한 것을 찾게 하자"는 방향으로 전환되고 있습니다.

이러한 접근법은 여러 측면에서 혁신적입니다. 기술적으로는 컨텍스트 윈도우의 한계를 극복하고, 토큰 사용량을 극적으로 줄이며, 응답 속도와 정확도를 향상시킵니다. 경제적으로는 AI API 비용을 80-98% 절감하여, 대규모 자동화가 경제적으로 실현 가능해집니다. 확장성 측면에서는 수천 개의 기능을 가진 AI 에이전트를 안정적으로 운영할 수 있게 합니다.

가장 중요한 것은 개발자 경험의 개선입니다. 점진적 공개를 사용하면, 개발자는 복잡한 컨텍스트 관리 로직을 작성할 필요가 없습니다. 그저 잘 작성된 skills를 추가하기만 하면, 에이전트가 알아서 필요할 때 찾아 사용합니다. 이는 마치 좋은 라이브러리를 사용하는 것처럼 직관적이고 편리합니다.

Claude Code가 점진적 공개를 통해 보여준 것은, AI 에이전트의 "지능"이 반드시 많은 정보를 동시에 처리하는 능력에서 나오는 것이 아니라는 점입니다. 오히려 필요한 정보를 적시에 찾고, 불필요한 것은 과감히 무시하는 능력이 더 중요합니다. 이는 인간의 인지 방식과도 유사합니다. 우리는 모든 것을 동시에 기억하고 처리하지 않습니다. 필요할 때 관련 기억을 떠올리고, 현재 작업에 집중합니다.

앞으로 AI 에이전트 개발은 점진적 공개를 기본으로 채택하게 될 것입니다. 이는 더 이상 선택 사항이 아닌 필수가 될 것입니다. 컨텍스트 윈도우가 계속 커진다 해도, 효율성과 정확성을 위해서는 점진적 공개가 여전히 필요할 것입니다.

개발자 여러분께 권장드립니다. 지금 당장 여러분의 프로젝트에 skills 디렉토리를 만들고, 첫 번째 skill을 작성해보세요. 에이전트에게 파일 시스템 접근 권한을 주고, 스스로 필요한 것을 찾도록 격려하세요. 그리고 놀라운 결과를 직접 경험하세요.

점진적 공개는 AI의 미래를 여는 열쇠 중 하나입니다. 이를 마스터하는 것은 차세대 AI 애플리케이션을 구축하는 경쟁력이 될 것입니다.

---

**부록: 추가 자료**

- Anthropic 공식 문서: https://docs.anthropic.com/
- Claude Code GitHub: https://github.com/anthropics/claude-code
- MCP 프로토콜 사양: https://modelcontextprotocol.io/
- Progressive Disclosure in Claude Code (YouTube): [링크](https://www.youtube.com/watch?v=DQHFow2NoQc)

