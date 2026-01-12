---
title: "Microsoft GraphRAG와 Neo4j 통합 설계 가이드"
date: 2026-01-12 23:45:00 +0900
categories: [AI,  RAG]
mermaid: [True]
tags: [AI,  RAG,  graph-rag,  Neo4j,  Microsoft-GraphRAG,  hybrid-rag,  PostgreSQL,  Elasticsearch,  BGE-M3,  DeepSeek,  Guide,  Claude.write]
---

## 엔터프라이즈 지식 검색을 위한 실무 중심 아키텍처

---

## 문서 정보
- **작성일**: 2026-01-12
- **버전**: 1.0
- **대상**: 16GB RAM 환경에서 운영 가능한 실무형 시스템
- **목적**: Microsoft GraphRAG의 자동 지식 추출과 Neo4j의 그래프 데이터베이스를 통합한 Hybrid RAG 시스템 구축

---

## 목차

1. [개요 및 배경](#1-개요-및-배경)
2. [Microsoft GraphRAG 이해](#2-microsoft-graphrag-이해)
3. [Neo4j와의 통합 전략](#3-neo4j와의-통합-전략)
4. [비용 최적화: DeepSeek 활용](#4-비용-최적화-deepseek-활용)
5. [데이터 파이프라인 설계](#5-데이터-파이프라인-설계)
6. [16GB RAM 환경 최적화](#6-16gb-ram-환경-최적화)
7. [Elasticsearch와의 3-way 통합](#7-elasticsearch와의-3-way-통합)
8. [실무 구현 가이드](#8-실무-구현-가이드)
9. [성능 최적화 전략](#9-성능-최적화-전략)
10. [운영 및 유지보수](#10-운영-및-유지보수)

---

## 1. 개요 및 배경

### 1.1 왜 GraphRAG와 Neo4j인가?

전통적인 벡터 RAG(Retrieval-Augmented Generation) 시스템은 문서를 청크 단위로 분할하고 임베딩하여 유사도 검색을 수행합니다. 이 방식은 단순하고 효과적이지만, 문서 간의 관계나 지식의 맥락을 충분히 활용하지 못하는 한계가 있습니다.

Microsoft GraphRAG는 이러한 한계를 극복하기 위해 개발되었습니다. 문서에서 자동으로 엔티티(Entity)와 관계(Relationship)를 추출하고, 이를 그래프 구조로 표현합니다. 이를 통해 "누가 어떤 프로젝트에 참여했는가?", "이 기술과 저 기술은 어떤 관계인가?"와 같은 복잡한 질문에 답할 수 있습니다.

Neo4j는 세계에서 가장 널리 사용되는 그래프 데이터베이스로, GraphRAG가 추출한 지식을 영구적으로 저장하고 효율적으로 탐색할 수 있게 해줍니다. Cypher라는 직관적인 그래프 쿼리 언어를 제공하여 복잡한 관계 탐색을 SQL보다 훨씬 쉽게 수행할 수 있습니다.

**이 두 기술을 통합하면**:
- GraphRAG가 문서에서 지식을 자동으로 추출
- Neo4j가 이를 효율적으로 저장하고 관리
- 벡터 검색과 그래프 탐색을 결합한 Hybrid RAG 구현
- 단순 키워드 검색을 넘어선 지능형 지식 검색 실현

### 1.2 전통적 RAG vs Graph RAG 비교

**전통적 Vector RAG의 강점과 한계**:

전통적 RAG는 구현이 단순하고 빠릅니다. 문서를 청크로 나누고, 임베딩 모델로 벡터화하고, 벡터 DB에 저장하면 됩니다. 쿼리가 들어오면 동일한 임베딩 모델로 벡터화하고 코사인 유사도로 관련 청크를 찾아냅니다. 이 방식은 "Django 프로젝트를 시작하는 방법은?"과 같은 단순한 정보 검색에 매우 효과적입니다.

그러나 한계도 명확합니다. "김철수가 참여한 프로젝트 중 React를 사용한 것은?"이라는 질문에는 답하기 어렵습니다. 문서에 "김철수", "React", "프로젝트"라는 단어가 모두 포함되어 있어도, 이들 간의 관계를 명시적으로 이해하지 못하기 때문입니다.

또한 시간적 맥락을 처리하기 어렵습니다. "2023년 당시 유효했던 보안 정책은?"이라는 질문에 답하려면 문서의 작성 시점과 유효 기간을 이해해야 하는데, 단순 벡터 유사도만으로는 이를 정확히 파악할 수 없습니다.

**Microsoft GraphRAG의 차별점**:

GraphRAG는 LLM(Large Language Model)을 활용하여 문서를 깊이 있게 분석합니다. 단순히 텍스트를 벡터로 변환하는 것이 아니라, 문서에서 핵심 엔티티를 식별하고 이들 간의 관계를 추출합니다.

예를 들어, "김철수는 2023년 프로젝트 A에서 React 개발을 담당했다"는 문장에서 다음을 추출합니다:
- **엔티티**: 김철수(Person), 프로젝트 A(Project), React(Technology)
- **관계**: 김철수 -[PARTICIPATED]-> 프로젝트 A, 프로젝트 A -[USED_TECH]-> React
- **시간 정보**: 2023년

이렇게 추출된 정보는 그래프 구조로 저장되어, 복잡한 관계 질문에도 정확하게 답할 수 있습니다. 또한 GraphRAG는 커뮤니티 탐지(Community Detection) 알고리즘을 사용하여 밀접하게 연결된 엔티티 그룹을 자동으로 식별하고, 각 커뮤니티에 대한 요약을 생성합니다. 이를 통해 "프로젝트 A와 관련된 모든 기술과 인물"과 같은 포괄적인 질문에도 답할 수 있습니다.

**Hybrid RAG: 최고의 양쪽 세계**:

Hybrid RAG는 Vector RAG와 Graph RAG의 장점을 결합합니다. 빠른 의미론적 검색에는 벡터 검색을 사용하고, 복잡한 관계 탐색에는 그래프 탐색을 사용합니다. 예를 들어:

1. "보안 취약점 대응 방법"이라는 질문 → 벡터 검색으로 관련 문서 빠르게 필터링
2. 검색된 문서에서 언급된 엔티티 추출 → Neo4j에서 관계 탐색
3. "이 문서를 작성한 사람은 누구인가?", "관련된 다른 프로젝트는?" → 그래프로 추가 맥락 수집
4. 벡터 검색 결과와 그래프 탐색 결과를 통합하여 LLM에 전달 → 포괄적이고 정확한 답변 생성

### 1.3 이 가이드의 목표

이 가이드는 Microsoft GraphRAG와 Neo4j를 실무 환경에 통합하는 구체적인 방법을 제시합니다. 단순한 이론이나 개념 설명을 넘어서, 실제로 작동하는 시스템을 구축할 수 있도록 다음을 제공합니다:

**실용성 중심**:
- 고가의 GPU 없이 일반 개발 워크스테이션(16GB RAM, CPU)에서 운영 가능
- 오픈소스 도구만 사용하여 라이선스 비용 제로
- 단계별 구현 가이드와 실제 코드 예시

**비용 최적화**:
- DeepSeek-V3.2를 활용한 LLM 비용 93% 절감 전략
- 효율적인 메모리 관리로 인프라 비용 최소화
- 캐싱과 배치 처리를 통한 API 호출 최적화

**확장 가능성**:
- 초기에는 작게 시작하여 점진적으로 확장
- 모듈화된 아키텍처로 컴포넌트별 독립 업그레이드
- 사용량 증가에 따른 수평 확장 전략

이 가이드를 따라 구축하면, 수백 개에서 수천 개의 사내 문서를 효과적으로 관리하고, 직원들이 필요한 지식을 빠르게 찾을 수 있는 지능형 검색 시스템을 갖추게 됩니다.

---

## 2. Microsoft GraphRAG 이해

### 2.1 GraphRAG의 핵심 개념

Microsoft GraphRAG는 단순히 텍스트를 그래프로 변환하는 도구가 아닙니다. 문서의 의미를 깊이 이해하고, 숨겨진 지식 구조를 발견하며, 이를 체계적으로 조직하는 지능형 시스템입니다.

**엔티티(Entity) 추출**:

엔티티는 그래프의 노드(Node)에 해당하며, 문서에서 식별 가능한 개념이나 객체를 의미합니다. GraphRAG는 LLM을 사용하여 다양한 유형의 엔티티를 자동으로 추출합니다:

- **인물(Person)**: 김철수, 이영희, John Doe 등
- **조직(Organization)**: 개발팀, 마케팅부, ABC Corp 등
- **프로젝트(Project)**: 프로젝트 A, 신제품 개발, 시스템 리뉴얼 등
- **기술(Technology)**: React, Neo4j, Python, AWS 등
- **개념(Concept)**: 보안, 성능 최적화, 애자일 등
- **위치(Location)**: 서울 본사, 판교 연구소, New York Office 등
- **이벤트(Event)**: 킥오프 미팅, 코드 리뷰, 출시일 등

GraphRAG의 강점은 도메인에 구애받지 않는다는 것입니다. 사전에 엔티티 타입을 정의할 필요 없이, LLM이 문서를 읽고 자동으로 중요한 엔티티를 식별합니다. 물론 특정 도메인에 맞게 엔티티 타입을 커스터마이징할 수도 있습니다.

**관계(Relationship) 매핑**:

관계는 그래프의 엣지(Edge)에 해당하며, 엔티티 간의 연결을 나타냅니다. GraphRAG는 다음과 같은 다양한 관계를 추출합니다:

- **직접적 관계**: "김철수는 프로젝트 A의 리더다" → (김철수) -[LEADS]-> (프로젝트 A)
- **참여 관계**: "이영희가 프로젝트 A에 참여했다" → (이영희) -[PARTICIPATED]-> (프로젝트 A)
- **사용 관계**: "프로젝트 A는 React를 사용한다" → (프로젝트 A) -[USES]-> (React)
- **의존 관계**: "프로젝트 B는 프로젝트 A에 의존한다" → (프로젝트 B) -[DEPENDS_ON]-> (프로젝트 A)
- **시간적 관계**: "프로젝트 A는 2023년에 시작되었다" → (프로젝트 A) -[STARTED_IN]-> (2023)

관계는 방향성과 속성을 가질 수 있습니다. 예를 들어, (김철수) -[PARTICIPATED {role: "개발자", from: "2023-01", to: "2023-12"}]-> (프로젝트 A)와 같이 역할과 기간 정보를 포함할 수 있습니다.

**커뮤니티 탐지(Community Detection)**:

GraphRAG의 독특한 기능 중 하나는 Leiden 알고리즘을 사용한 커뮤니티 탐지입니다. 이는 밀접하게 연결된 엔티티 그룹을 자동으로 식별합니다.

예를 들어, 다음과 같은 엔티티들이 있다고 가정해봅시다:
- 김철수, 이영희, 박민수 (인물)
- 프로젝트 A, 프로젝트 B (프로젝트)
- React, Redux, TypeScript (기술)

커뮤니티 탐지 알고리즘은 이들 사이의 연결 강도를 분석하여 "React 개발 커뮤니티"와 같은 그룹을 형성합니다. 이 커뮤니티에는 React를 사용하는 프로젝트, React 개발 경험이 있는 인물, React 관련 기술들이 포함됩니다.

각 커뮤니티에 대해 GraphRAG는 LLM을 사용하여 요약(Summary)을 자동 생성합니다. 이 요약은 Global Search에서 활용되어, "우리 회사의 React 개발 현황은?"과 같은 포괄적인 질문에 답할 수 있게 합니다.

**계층적 구조(Hierarchical Structure)**:

GraphRAG는 커뮤니티를 계층적으로 조직합니다. 예를 들어:
- Level 0 (최상위): "기술 프로젝트" 커뮤니티
  - Level 1: "프론트엔드 개발" 커뮤니티, "백엔드 개발" 커뮤니티
    - Level 2: "React 프로젝트" 커뮤니티, "Vue 프로젝트" 커뮤니티

이러한 계층 구조는 다양한 추상화 수준에서 지식을 탐색할 수 있게 해줍니다. 상위 레벨 커뮤니티 요약은 전체적인 개요를 제공하고, 하위 레벨로 내려갈수록 구체적인 세부사항을 제공합니다.

### 2.2 GraphRAG의 주요 출력물

Microsoft GraphRAG를 실행하면 여러 Parquet 파일이 생성됩니다. 이 파일들은 추출된 지식을 구조화된 형태로 저장하며, Neo4j로 로드할 데이터의 원천이 됩니다.

**create_final_entities.parquet**:

추출된 모든 엔티티 정보를 담고 있습니다. 주요 컬럼:
- `id`: 엔티티의 고유 식별자 (예: "PERSON_001", "PROJECT_042")
- `title` 또는 `name`: 엔티티의 이름 (예: "김철수", "프로젝트 A")
- `type`: 엔티티 타입 (예: "PERSON", "PROJECT", "TECHNOLOGY")
- `description`: 엔티티에 대한 설명 (LLM이 생성)
- `degree`: 엔티티의 연결 수 (중요도 지표)
- `community`: 속한 커뮤니티 ID
- `text_unit_ids`: 이 엔티티가 언급된 텍스트 청크 ID들

**create_final_relationships.parquet**:

엔티티 간의 관계 정보를 담고 있습니다. 주요 컬럼:
- `id`: 관계의 고유 식별자
- `source`: 출발 엔티티 ID
- `target`: 도착 엔티티 ID
- `description`: 관계에 대한 설명 (예: "김철수는 프로젝트 A의 리더로 참여했다")
- `weight`: 관계의 강도 (0.0 ~ 1.0, LLM이 평가)
- `text_unit_ids`: 이 관계가 추출된 텍스트 청크 ID들
- `attributes`: 추가 속성 (JSON, 예: {"role": "leader", "period": "2023-2024"})

**create_final_communities.parquet**:

커뮤니티 정보와 요약을 담고 있습니다. 주요 컬럼:
- `id`: 커뮤니티 ID
- `title`: 커뮤니티 제목 (LLM이 생성, 예: "React 프론트엔드 개발 팀")
- `level`: 계층 레벨 (0이 최상위)
- `entity_ids`: 이 커뮤니티에 속한 엔티티 ID 목록
- `relationship_ids`: 커뮤니티 내 관계 ID 목록
- `summary`: 커뮤니티 요약 (LLM이 생성, 수백 단어)
- `findings`: 주요 발견 사항 (구조화된 인사이트)

**create_final_text_units.parquet**:

원본 문서의 청크 정보를 담고 있습니다. 주요 컬럼:
- `id`: 텍스트 청크 ID
- `text`: 실제 텍스트 내용
- `document_ids`: 출처 문서 ID
- `entity_ids`: 이 청크에서 추출된 엔티티 ID 목록
- `relationship_ids`: 이 청크에서 추출된 관계 ID 목록
- `n_tokens`: 토큰 수
- `embeddings`: 임베딩 벡터 (옵션)

**create_final_documents.parquet**:

원본 문서 메타데이터를 담고 있습니다. 주요 컬럼:
- `id`: 문서 ID
- `title`: 문서 제목
- `raw_content`: 원본 텍스트 (옵션)
- `text_unit_ids`: 이 문서에 속한 텍스트 청크 ID들

이 파일들은 서로 ID로 연결되어 있어, 문서 → 텍스트 청크 → 엔티티/관계 → 커뮤니티로 이어지는 전체 지식 구조를 재구성할 수 있습니다.

### 2.3 Local Search vs Global Search

GraphRAG는 두 가지 주요 검색 방식을 제공하며, 각각 다른 유형의 질문에 최적화되어 있습니다.

**Local Search (지역 검색)**:

Local Search는 특정 엔티티나 개념에 대한 상세한 정보를 찾는 데 적합합니다. 예를 들어:
- "김철수의 경력은?"
- "프로젝트 A에서 사용된 기술 스택은?"
- "React 개발 베스트 프랙티스는?"

검색 과정:
1. 사용자 질문에서 핵심 엔티티 추출 (예: "김철수")
2. 그래프에서 해당 엔티티 찾기
3. 1-2 hop 이내의 연결된 엔티티와 관계 수집
4. 관련된 텍스트 청크 조회
5. LLM에 전달하여 답변 생성

Local Search의 강점은 정확성과 상세함입니다. 특정 엔티티와 직접 연결된 정보만 수집하므로 노이즈가 적고, 원본 텍스트를 직접 참조하므로 사실 관계가 정확합니다.

**Global Search (전역 검색)**:

Global Search는 포괄적이고 요약적인 답변이 필요한 질문에 적합합니다. 예를 들어:
- "우리 회사의 주요 프로젝트 현황은?"
- "React와 Vue 중 어느 것을 더 많이 사용하나?"
- "지난 1년간 기술 트렌드는?"

검색 과정:
1. 사용자 질문의 주제 파악
2. 관련 커뮤니티들을 상위 레벨부터 탐색
3. 각 커뮤니티의 요약(Summary) 수집
4. 필요시 하위 레벨 커뮤니티로 드릴다운
5. 수집된 요약들을 LLM에 전달하여 종합 답변 생성

Global Search의 강점은 넓은 범위의 정보를 효율적으로 종합한다는 것입니다. 개별 문서를 일일이 검색하는 대신, 사전에 생성된 커뮤니티 요약을 활용하여 빠르고 포괄적인 답변을 제공합니다.

**Hybrid Approach**:

실제 시스템에서는 두 방식을 결합합니다. 질문의 특성을 분석하여:
- 특정 엔티티에 대한 질문 → Local Search
- 트렌드나 전체 현황 → Global Search
- 비교 분석 → Local + Global 결합

예를 들어, "프로젝트 A와 B의 차이점은?"이라는 질문에는:
1. Local Search로 프로젝트 A와 B의 상세 정보 수집
2. Global Search로 두 프로젝트가 속한 커뮤니티의 맥락 파악
3. 두 결과를 통합하여 비교 분석 수행

---

## 3. Neo4j와의 통합 전략

### 3.1 왜 Neo4j인가?

Microsoft GraphRAG는 Parquet 파일 형태로 지식 그래프를 출력합니다. 이는 데이터 교환 형식으로는 적합하지만, 실시간 검색과 복잡한 그래프 탐색에는 한계가 있습니다. Neo4j를 통합하면 다음과 같은 이점을 얻을 수 있습니다.

**실시간 그래프 탐색**:

Parquet 파일을 읽어서 그래프를 재구성하는 것은 느립니다. 특히 수만 개의 노드와 관계가 있는 대규모 그래프에서는 불가능합니다. Neo4j는 그래프 데이터를 메모리와 디스크에 최적화된 형태로 저장하여, 복잡한 탐색 쿼리도 밀리초 단위로 수행할 수 있습니다.

예를 들어, "김철수와 2단계 이내로 연결된 모든 프로젝트"를 찾는 쿼리는 Cypher로 다음과 같이 작성됩니다:

```cypher
MATCH (p:Person {name: "김철수"})-[*1..2]-(proj:Project)
RETURN DISTINCT proj
```

이 쿼리는 Neo4j에서 수 밀리초 내에 실행되지만, Parquet 파일을 읽어서 처리하면 수 초가 걸릴 수 있습니다.

**트랜잭션과 일관성**:

Neo4j는 ACID 트랜잭션을 지원하여 데이터 무결성을 보장합니다. 여러 사용자가 동시에 그래프를 읽고 쓸 때도 일관성을 유지합니다. 이는 협업 환경에서 필수적입니다.

**그래프 알고리즘**:

Neo4j는 다양한 그래프 알고리즘을 제공합니다:
- PageRank: 중요한 엔티티 찾기
- Shortest Path: 엔티티 간 최단 경로
- Community Detection: 커뮤니티 재탐지 및 업데이트
- Centrality: 네트워크에서 중심적인 역할을 하는 엔티티

이러한 알고리즘을 활용하여 "가장 영향력 있는 개발자는?", "프로젝트 A와 B를 연결하는 최단 경로는?" 같은 고급 분석을 수행할 수 있습니다.

**시각화와 탐색**:

Neo4j Browser와 Bloom과 같은 시각화 도구를 사용하여 그래프를 직관적으로 탐색할 수 있습니다. 이는 시스템을 이해하고 디버깅하는 데 매우 유용합니다.

**확장성**:

Neo4j는 수억 개의 노드와 관계를 처리할 수 있으며, 클러스터링을 통해 수평 확장도 가능합니다. 초기에는 단일 인스턴스로 시작하다가, 데이터가 증가하면 클러스터로 전환할 수 있습니다.

### 3.2 Slim Graph 전략

16GB RAM 환경에서 Neo4j를 효율적으로 운영하기 위한 핵심 전략은 **Slim Graph**입니다. 이는 그래프에 최소한의 정보만 저장하고, 상세 데이터는 다른 저장소에 두는 방식입니다.

**전통적 접근 (Fat Graph)**:

일반적으로는 모든 정보를 그래프 노드의 속성으로 저장합니다:

```cypher
CREATE (k:Knowledge {
    knowledge_id: "k123",
    title: "React 개발 가이드",
    content: "React는 ... (긴 텍스트)",
    summary: "React 개발의 기본 개념을 설명 ... (긴 텍스트)",
    tags: ["React", "프론트엔드", "개발"],
    created_at: "2023-05-01",
    updated_at: "2023-12-15",
    contributor: "김철수",
    project: "프로젝트 A",
    views: 152,
    likes: 23
})
```

이 방식은 편리하지만, 메모리를 많이 사용합니다. 특히 `content`와 `summary` 같은 긴 텍스트 필드는 수백 KB를 차지할 수 있습니다. 10,000개의 노드가 있고 각 노드가 평균 100KB를 사용하면 1GB의 메모리가 필요합니다.

**Slim Graph 접근**:

Slim Graph에서는 ID와 관계 정보만 Neo4j에 저장하고, 상세 속성은 PostgreSQL이나 Elasticsearch에 저장합니다:

```cypher
CREATE (k:Knowledge {
    knowledge_id: "k123",
    title: "React 개발 가이드",  // 검색용 최소 정보
    type: "TechnicalGuide"
    // 나머지는 PostgreSQL에서 조회
})
```

이렇게 하면 노드당 메모리 사용량이 수 KB로 줄어들어, 같은 메모리로 10배 이상 많은 노드를 처리할 수 있습니다.

**Slim Graph의 작동 방식**:

1. **그래프 탐색**: Neo4j에서 관련 엔티티 ID들을 빠르게 찾음
```cypher
MATCH (p:Person {name: "김철수"})-[:PARTICIPATED]->(proj:Project)
RETURN proj.knowledge_id
```

2. **상세 정보 조회**: 얻은 ID로 PostgreSQL에서 상세 정보 조회
```python
knowledge_ids = ["k123", "k456", "k789"]
details = postgres.execute(
    "SELECT * FROM knowledge_master WHERE knowledge_id IN %s",
    (tuple(knowledge_ids),)
)
```

3. **결과 통합**: 그래프 관계 정보와 상세 정보를 결합
```python
for detail in details:
    detail['related_persons'] = neo4j.query(
        "MATCH (k:Knowledge {knowledge_id: $id})<-[:CREATED]-(p:Person) RETURN p.name",
        id=detail['knowledge_id']
    )
```

**메모리 절감 효과**:

| 항목 | Fat Graph | Slim Graph | 절감률 |
|------|-----------|------------|--------|
| 노드당 메모리 | ~100KB | ~5KB | 95% |
| 10,000 노드 | 1GB | 50MB | 95% |
| 100,000 노드 | 10GB | 500MB | 95% |

16GB RAM 환경에서 Neo4j에 2GB만 할당해도 40,000개 이상의 노드를 처리할 수 있습니다.

### 3.3 데이터 모델 설계

Neo4j 그래프 스키마는 GraphRAG의 출력 구조를 반영하되, Slim Graph 원칙을 따라 설계합니다.

**노드 타입 (Labels)**:

```cypher
// 1. Entity 노드 (GraphRAG 추출)
CREATE (e:Entity {
    id: "ENTITY_001",          // GraphRAG entity ID
    name: "김철수",             // 검색용
    type: "PERSON"             // PERSON, PROJECT, TECHNOLOGY 등
})

// 2. TextUnit 노드 (청크)
CREATE (t:TextUnit {
    id: "CHUNK_001",
    document_id: "DOC_123",
    chunk_index: 0
    // 실제 텍스트는 Elasticsearch에
})

// 3. Community 노드 (커뮤니티)
CREATE (c:Community {
    id: "COMM_001",
    title: "React 개발팀",
    level: 0
    // 요약은 PostgreSQL에
})

// 4. Knowledge 노드 (문서 마스터)
CREATE (k:Knowledge {
    knowledge_id: "k123",
    title: "React 가이드",
    type: "TechnicalGuide"
    // 상세 정보는 PostgreSQL에
})
```

**관계 타입 (Relationships)**:

```cypher
// 엔티티 간 관계 (GraphRAG 추출)
CREATE (e1:Entity)-[:RELATED_TO {
    description: "함께 참여함",
    weight: 0.8,
    source_chunk: "CHUNK_001"
}]->(e2:Entity)

// 엔티티가 언급된 청크
CREATE (e:Entity)-[:MENTIONED_IN {
    relevance: 0.9
}]->(t:TextUnit)

// 엔티티가 속한 커뮤니티
CREATE (e:Entity)-[:BELONGS_TO]->(c:Community)

// 커뮤니티 계층
CREATE (c1:Community)-[:PARENT_OF]->(c2:Community)

// 지식 연결
CREATE (k:Knowledge)<-[:CREATED {
    timestamp: datetime()
}]-(e:Entity {type: "PERSON"})

CREATE (k:Knowledge)-[:USES_TECH]->(e:Entity {type: "TECHNOLOGY"})
```

**제약조건과 인덱스**:

```cypher
// 유니크 제약
CREATE CONSTRAINT entity_id_unique IF NOT EXISTS
FOR (e:Entity) REQUIRE e.id IS UNIQUE;

CREATE CONSTRAINT knowledge_id_unique IF NOT EXISTS
FOR (k:Knowledge) REQUIRE k.knowledge_id IS UNIQUE;

// 검색 최적화 인덱스
CREATE INDEX entity_name_index IF NOT EXISTS
FOR (e:Entity) ON (e.name);

CREATE INDEX entity_type_index IF NOT EXISTS
FOR (e:Entity) ON (e.type);

CREATE INDEX knowledge_title_index IF NOT EXISTS
FOR (k:Knowledge) ON (k.title);

// 전문 검색 인덱스
CREATE FULLTEXT INDEX entity_fulltext IF NOT EXISTS
FOR (e:Entity) ON EACH [e.name];
```

**샘플 그래프 예시**:

실제 "김철수가 React로 프로젝트 A를 개발했다"는 정보는 다음과 같이 표현됩니다:

```cypher
// 엔티티 생성
CREATE (person:Entity {id: "E1", name: "김철수", type: "PERSON"})
CREATE (project:Entity {id: "E2", name: "프로젝트 A", type: "PROJECT"})
CREATE (tech:Entity {id: "E3", name: "React", type: "TECHNOLOGY"})
CREATE (knowledge:Knowledge {knowledge_id: "k123", title: "프로젝트 A 개발 일지", type: "ProjectReport"})

// 관계 생성
CREATE (person)-[:PARTICIPATED {role: "개발자", period: "2023"}]->(project)
CREATE (project)-[:USES]->(tech)
CREATE (person)-[:CREATED]->(knowledge)
CREATE (knowledge)-[:ABOUT]->(project)

// 청크 연결
CREATE (chunk:TextUnit {id: "CHUNK_001", document_id: "DOC_123"})
CREATE (person)-[:MENTIONED_IN]->(chunk)
CREATE (project)-[:MENTIONED_IN]->(chunk)
CREATE (tech)-[:MENTIONED_IN]->(chunk)
```

이제 다음과 같은 쿼리가 가능합니다:

```cypher
// 김철수가 참여한 프로젝트 찾기
MATCH (p:Entity {name: "김철수", type: "PERSON"})-[:PARTICIPATED]->(proj:Entity {type: "PROJECT"})
RETURN proj.name

// React를 사용하는 모든 프로젝트
MATCH (tech:Entity {name: "React"})<-[:USES]-(proj:Entity {type: "PROJECT"})
RETURN proj.name

// 김철수와 React의 연결고리
MATCH path = (p:Entity {name: "김철수"})-[*1..3]-(tech:Entity {name: "React"})
RETURN path
LIMIT 5
```

### 3.4 3-way 통합 아키텍처

실무 환경에서는 Neo4j 단독으로 운영하지 않고, PostgreSQL, Elasticsearch와 함께 3-way 통합 아키텍처를 구성합니다. 각 데이터베이스는 자신이 가장 잘하는 역할에 집중합니다.

**PostgreSQL: 마스터 레코드 관리**

PostgreSQL은 Single Source of Truth(SSOT) 역할을 합니다. 모든 문서와 메타데이터의 원본이 여기 저장됩니다.

주요 테이블:
```sql
-- 지식 마스터
CREATE TABLE knowledge_master (
    knowledge_id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    document_type VARCHAR(50),
    project_name VARCHAR(255),
    contributor_id INT,
    
    -- 시계열 정보
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valid_start_date DATE,
    valid_end_date DATE,
    
    -- GraphRAG 추출 정보
    entities JSONB,  -- {persons: [], technologies: [], keywords: []}
    summary TEXT,
    tags JSONB
);

-- 프로젝트 마스터
CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(255),
    start_date DATE,
    end_date DATE,
    status VARCHAR(50)
);
```

PostgreSQL이 하는 일:
- 문서 메타데이터의 영구 저장
- 관리자 대시보드용 통계 쿼리
- 감사 로그 관리
- 데이터 무결성 검증

**Neo4j: 관계 탐색**

Neo4j는 Slim Graph로 엔티티 간 관계만 저장합니다.

Neo4j가 하는 일:
- 그래프 탐색 쿼리 (김철수와 연결된 프로젝트는?)
- 전문가 찾기 (React 경험자는?)
- 영향도 분석 (이 기술을 사용하는 프로젝트는?)
- 최단 경로 탐색

**Elasticsearch: 제로 조인 검색**

Elasticsearch는 벡터 검색 + 메타데이터를 통합 저장하여 단일 쿼리로 모든 처리를 완결합니다.

저장 구조:
```json
{
    "text": "김철수는 2023년 프로젝트 A에서 React 개발을 담당했다...",
    "vector_field": [0.123, -0.456, ...],  // BGE-M3 임베딩
    "metadata": {
        "knowledge_id": "k123",
        "title": "프로젝트 A 개발 일지",
        "document_type": "ProjectReport",
        "project_name": "프로젝트 A",
        "valid_start_date": "2023-01-01",
        "valid_end_date": "2024-12-31",
        "entities": {
            "persons": ["김철수"],
            "technologies": ["React"],
            "projects": ["프로젝트 A"]
        }
    }
}
```

Elasticsearch가 하는 일:
- 벡터 유사도 검색
- 키워드 검색 (BM25)
- 메타데이터 필터링 (시간, 프로젝트, 타입)
- 하이브리드 검색 (벡터 + 키워드 + 필터)

**검색 흐름 예시**:

질문: "2023년 프로젝트 A의 보안 취약점 대응 방법은?"

1. **Elasticsearch**: 제로 조인 검색
```python
results = es.search(
    query="보안 취약점 대응",
    filters={
        "valid_start_date": {"lte": "2023-12-31"},
        "valid_end_date": {"gte": "2023-01-01"},
        "project_name": "프로젝트 A"
    },
    k=5
)
# 단일 쿼리로 벡터 검색 + 시간 필터 + 프로젝트 필터 완료
```

2. **Neo4j**: 관련 전문가 찾기 (선택적)
```cypher
MATCH (k:Knowledge {knowledge_id: "k123"})<-[:CREATED]-(person:Entity {type: "PERSON"})
RETURN person.name
```

3. **PostgreSQL**: 추가 메타데이터 (필요시)
```sql
SELECT tags, summary FROM knowledge_master WHERE knowledge_id = 'k123'
```

실제로는 대부분의 검색이 1번(Elasticsearch)만으로 완결됩니다. 2번과 3번은 특수한 경우에만 실행됩니다.

**데이터 동기화**:

3개 DB는 동일한 데이터의 다른 뷰를 저장하므로, 동기화가 필수입니다.

```python
def index_document(document):
    """문서를 3개 DB에 동시 저장"""
    
    # 1. GraphRAG로 엔티티 추출
    metadata = extract_with_graphrag(document)
    
    # 2. PostgreSQL: 마스터 레코드 저장
    knowledge_id = postgres.insert("knowledge_master", {
        "title": metadata["title"],
        "document_type": metadata["document_type"],
        "project_name": metadata["project_name"],
        "valid_start_date": metadata["valid_start_date"],
        "valid_end_date": metadata["valid_end_date"],
        "entities": metadata["entities"],
        "summary": metadata["summary"]
    })
    
    # 3. Neo4j: 그래프 생성
    for entity in metadata["entities"]["persons"]:
        neo4j.merge_node("Entity", {"name": entity, "type": "PERSON"})
    
    neo4j.create_relationship(
        ("Entity", {"name": metadata["contributor"]}),
        "CREATED",
        ("Knowledge", {"knowledge_id": knowledge_id})
    )
    
    # 4. Elasticsearch: 청크 + 메타데이터 저장
    chunks = split_document(document)
    for i, chunk in enumerate(chunks):
        chunk.metadata = {
            "knowledge_id": knowledge_id,
            "title": metadata["title"],
            "document_type": metadata["document_type"],
            "project_name": metadata["project_name"],
            "valid_start_date": metadata["valid_start_date"],
            "valid_end_date": metadata["valid_end_date"],
            "entities": metadata["entities"],
            "chunk_index": i,
            "total_chunks": len(chunks)
        }
        vectorstore.add_documents([chunk])
    
    return knowledge_id
```

**일관성 보장**:

3개 DB에 동시 저장하는 과정에서 실패가 발생할 수 있습니다. 이를 처리하기 위해:

```python
def index_document_transactional(document):
    """트랜잭션 보장 저장"""
    try:
        # 1. PostgreSQL 트랜잭션 시작
        with postgres.transaction():
            knowledge_id = postgres.insert(...)
            
            # 2. Neo4j 저장 (실패 시 롤백)
            try:
                neo4j.create_graph(knowledge_id, metadata)
            except Exception as e:
                raise Exception(f"Neo4j 저장 실패: {e}")
            
            # 3. Elasticsearch 저장 (실패 시 롤백)
            try:
                vectorstore.add_documents(chunks)
            except Exception as e:
                raise Exception(f"ES 저장 실패: {e}")
            
        # 모두 성공 시 커밋
        logger.info(f"문서 {knowledge_id} 저장 완료")
        
    except Exception as e:
        # 실패 시 재시도 큐에 추가
        retry_queue.add({
            "document": document,
            "error": str(e),
            "timestamp": datetime.now()
        })
        logger.error(f"저장 실패: {e}")
        raise
```

---

## 4. 비용 최적화: DeepSeek 활용

### 4.1 LLM 비용 문제

Microsoft GraphRAG의 가장 큰 운영 비용은 LLM API입니다. 엔티티와 관계를 추출하기 위해 모든 문서를 LLM에 전달해야 하는데, 문서가 많아질수록 비용이 급증합니다.

**기존 비용 구조 (Claude/GPT 사용 시)**:

1,000개 문서 처리 시나리오:
- 문서당 평균 토큰: 2,000 토큰
- 총 입력 토큰: 2,000,000 토큰 (2M)
- Claude 3.5 Sonnet 비용: $15.00 / 1M 입력 토큰
- **총 비용**: $30.00

월 5,000개 문서 처리 시: **$150/월**
연간: **$1,800**

이는 소규모 조직에도 부담이 되며, 대규모 조직에서는 수천 달러에 달할 수 있습니다.

### 4.2 DeepSeek-V3.2 소개

DeepSeek-V3.2는 중국의 DeepSeek AI가 개발한 대규모 언어 모델로, 놀라운 비용 효율성을 제공합니다.

**가격 비교**:

| 모델 | 입력 (1M 토큰) | 출력 (1M 토큰) | 품질 |
|------|----------------|----------------|------|
| Claude 3.5 Sonnet | $3.00 | $15.00 | ⭐⭐⭐⭐⭐ |
| GPT-4o | $2.50 | $10.00 | ⭐⭐⭐⭐⭐ |
| GPT-4o mini | $0.15 | $0.60 | ⭐⭐⭐⭐ |
| **DeepSeek-V3** | **$0.27** | **$1.10** | ⭐⭐⭐⭐ |

**캐시 히트 활용 시**:
- DeepSeek 캐시 히트: $0.027 / 1M 토큰 (90% 할인)

**성능**:
- 엔티티 추출 정확도: Claude 대비 95% 수준
- 추론 속도: GPT-4o 수준
- 한국어 지원: 양호

**두 가지 모드**:

1. **deepseek-chat (Non-thinking)**:
   - 빠른 처리
   - 단순 엔티티 추출에 적합
   - 비용: $0.27 / 1M 입력 토큰

2. **deepseek-reasoner (Thinking)**:
   - Chain of Thought 내장
   - 복잡한 관계 추론에 적합
   - 비용: 동일 ($0.27 / 1M)

### 4.3 비용 절감 계산

**1,000개 문서 처리 비용 비교**:

| 항목 | Claude 3.5 | DeepSeek-V3 | 절감률 |
|------|------------|-------------|--------|
| 엔티티 추출 (Non-thinking) | $30.00 | $0.54 | 98.2% |
| 관계 추론 (Thinking) | $30.00 | $0.54 | 98.2% |
| 커뮤니티 요약 | $20.00 | $0.36 | 98.2% |
| **총계** | **$80.00** | **$1.44** | **98.2%** |

**캐시 히트 활용 시** (반복 작업):
- 시스템 프롬프트 고정 → 캐시 히트율 80%
- 실질 비용: $1.44 × 0.2 + $1.44 × 0.1 = **$0.43**
- **절감률: 99.5%**

**실무 시나리오**:

- 초기 10,000개 문서 인덱싱: $14.40 (Claude: $800)
- 월간 1,000개 신규 문서: $1.44/월 (Claude: $80/월)
- 연간 운영 비용: $17.28 (Claude: $960)
- **연간 절감액: $942.72**

### 4.4 DeepSeek 통합 구현

**API 설정**:

```python
from langchain_openai import ChatOpenAI
import os

# DeepSeek API 키 설정
os.environ["DEEPSEEK_API_KEY"] = "your-api-key"

# Non-thinking 모드 (빠른 엔티티 추출)
deepseek_fast = ChatOpenAI(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    temperature=0,
    max_tokens=2000
)

# Thinking 모드 (복잡한 추론)
deepseek_reasoner = ChatOpenAI(
    model="deepseek-reasoner",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    temperature=1  # Thinking 모드는 temperature=1 권장
)
```

**엔티티 추출 프롬프트 (캐시 최적화)**:

```python
# 시스템 프롬프트를 고정하여 캐시 히트율 극대화
SYSTEM_PROMPT = """당신은 엔터프라이즈 지식 그래프 구축 전문가입니다.

주어진 문서에서 다음을 JSON 형식으로 추출하세요:

1. entities: 엔티티 목록 (persons, organizations, projects, technologies, keywords)
2. relationships: 엔티티 간 관계 [(source, type, target), ...]
3. metadata: 문서 메타데이터 (document_type, project_name, valid_start_date, valid_end_date, summary)

**엔티티 타입**:
- persons: 실명 (예: 김철수, John Doe)
- organizations: 회사/부서 (예: 개발팀, ABC Corp)
- projects: 프로젝트명 (예: 프로젝트 A, 신제품 개발)
- technologies: 기술/도구 (예: React, Python, AWS)
- keywords: 핵심 개념 (예: 보안, 성능, 애자일)

**관계 타입**:
- PARTICIPATED: 참여
- CREATED: 작성
- USES: 사용
- DEPENDS_ON: 의존
- RELATED_TO: 관련

**메타데이터**:
- document_type: 문서 유형 (기술문서, 프로젝트보고서, 회의록, 일반가이드 등)
- project_name: 관련 프로젝트명 (없으면 null)
- valid_start_date: 유효 시작일 (YYYY-MM-DD, 추정 가능)
- valid_end_date: 유효 종료일 (YYYY-MM-DD, 추정 가능, 없으면 null)
- summary: 3줄 이내 요약

출력 형식:
{
    "entities": {
        "persons": ["김철수", "이영희"],
        "organizations": ["개발팀"],
        "projects": ["프로젝트 A"],
        "technologies": ["React", "Neo4j"],
        "keywords": ["지식검색", "Graph RAG"]
    },
    "relationships": [
        {"source": "김철수", "type": "PARTICIPATED", "target": "프로젝트 A"},
        {"source": "프로젝트 A", "type": "USES", "target": "React"}
    ],
    "metadata": {
        "document_type": "프로젝트보고서",
        "project_name": "프로젝트 A",
        "valid_start_date": "2023-01-01",
        "valid_end_date": "2024-12-31",
        "summary": "프로젝트 A는 React와 Neo4j를 활용한 지식 검색 시스템 개발 프로젝트입니다. 김철수가 주도하여 2023년 시작되었습니다."
    }
}
"""

def extract_entities_fast(document_text):
    """Non-thinking 모드로 빠른 엔티티 추출"""
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},  # 캐시됨
        {"role": "user", "content": f"문서 내용:\n\n{document_text}"}
    ]
    
    response = deepseek_fast.invoke(messages)
    return json.loads(response.content)
```

**복잡한 관계 추론 (Thinking 모드)**:

```python
def extract_complex_relationships(document_text):
    """Thinking 모드로 깊은 관계 분석"""
    prompt = f"""다음 기술 문서를 깊이 분석하세요:

{document_text}

다음을 추론하여 JSON으로 반환하세요:

1. temporal_chain: 시간 순서로 연결된 이벤트
   - 예: [("프로젝트 시작", "2023-01"), ("첫 릴리스", "2023-06"), ("완료", "2024-12")]

2. dependencies: 프로젝트/기술 간 의존 관계
   - 예: [("프로젝트 B", "DEPENDS_ON", "프로젝트 A")]

3. causal_links: 인과 관계
   - 예: [("성능 저하", "CAUSED_BY", "데이터베이스 병목")]

4. implicit_relationships: 명시되지 않았지만 추론 가능한 관계
   - 예: [("김철수", "EXPERT_IN", "React") # 여러 React 프로젝트 참여로 추론]
"""
    
    response = deepseek_reasoner.invoke(prompt)
    
    # Thinking 모드는 <think>...</think> 태그 포함
    # 실제 답변만 추출
    content = response.content
    if "<think>" in content:
        content = content.split("</think>")[-1].strip()
    
    return json.loads(content)
```

**배치 처리로 효율 극대화**:

```python
import asyncio
from typing import List

async def batch_extract_entities(documents: List[str], batch_size: int = 10):
    """비동기 배치 처리로 속도 향상"""
    results = []
    
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i+batch_size]
        
        # 동시 처리
        tasks = [
            asyncio.create_task(extract_entities_async(doc))
            for doc in batch
        ]
        
        batch_results = await asyncio.gather(*tasks)
        results.extend(batch_results)
        
        # 진행상황 로깅
        logger.info(f"처리 완료: {i+len(batch)}/{len(documents)} 문서")
    
    return results

async def extract_entities_async(document_text):
    """비동기 엔티티 추출"""
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(
        None,
        extract_entities_fast,
        document_text
    )
```

**비용 추적**:

```python
class CostTracker:
    """DeepSeek API 비용 추적"""
    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0
        self.cached_tokens = 0
        
        # DeepSeek 가격 (per 1M tokens)
        self.price_input = 0.27
        self.price_output = 1.10
        self.price_cache_hit = 0.027
    
    def track(self, response):
        """API 응답에서 토큰 사용량 추출"""
        usage = response.response_metadata.get("token_usage", {})
        
        self.total_input_tokens += usage.get("prompt_tokens", 0)
        self.total_output_tokens += usage.get("completion_tokens", 0)
        self.cached_tokens += usage.get("prompt_cache_hit_tokens", 0)
    
    def get_cost(self):
        """총 비용 계산"""
        # 캐시 미스 토큰
        uncached_input = self.total_input_tokens - self.cached_tokens
        
        # 비용 계산
        input_cost = (uncached_input / 1_000_000) * self.price_input
        cache_cost = (self.cached_tokens / 1_000_000) * self.price_cache_hit
        output_cost = (self.total_output_tokens / 1_000_000) * self.price_output
        
        total_cost = input_cost + cache_cost + output_cost
        
        return {
            "total_cost": f"${total_cost:.4f}",
            "input_cost": f"${input_cost:.4f}",
            "cache_cost": f"${cache_cost:.4f}",
            "output_cost": f"${output_cost:.4f}",
            "total_tokens": self.total_input_tokens + self.total_output_tokens,
            "cache_hit_rate": f"{(self.cached_tokens / self.total_input_tokens * 100):.1f}%"
        }

# 사용 예시
tracker = CostTracker()

for doc in documents:
    response = deepseek_fast.invoke([
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": doc}
    ])
    tracker.track(response)

print(tracker.get_cost())
# {
#     "total_cost": "$1.44",
#     "input_cost": "$0.10",
#     "cache_cost": "$0.22",  # 80% 캐시 히트
#     "output_cost": "$1.12",
#     "total_tokens": 2500000,
#     "cache_hit_rate": "80.0%"
# }
```

### 4.5 품질 보장 전략

DeepSeek은 비용 효율적이지만, Claude/GPT 대비 정확도가 약간 낮을 수 있습니다. 이를 보완하는 전략:

**1. 이중 검증 (Dual Verification)**:

중요한 문서는 DeepSeek + GPT-4o로 이중 검증:

```python
def extract_with_verification(document_text, important=False):
    """중요 문서는 이중 검증"""
    
    # 1차: DeepSeek으로 추출
    deepseek_result = extract_entities_fast(document_text)
    
    if not important:
        return deepseek_result
    
    # 2차: GPT-4o로 검증 (중요 문서만)
    gpt4o = ChatOpenAI(model="gpt-4o", temperature=0)
    verification_prompt = f"""
    다음은 DeepSeek이 추출한 엔티티입니다:
    {json.dumps(deepseek_result, ensure_ascii=False)}
    
    문서:
    {document_text}
    
    누락되거나 잘못된 엔티티가 있으면 수정하세요.
    정확하면 그대로 반환하세요.
    """
    
    verified = gpt4o.invoke(verification_prompt)
    return json.loads(verified.content)
```

**2. 사람 검토 워크플로우**:

추출 결과의 신뢰도 점수를 계산하여, 낮은 경우 사람 검토:

```python
def calculate_confidence(extraction_result):
    """추출 신뢰도 계산"""
    score = 100
    
    # 엔티티 수가 너무 적거나 많으면 감점
    entity_count = sum(len(v) for v in extraction_result["entities"].values())
    if entity_count < 3:
        score -= 20
    if entity_count > 50:
        score -= 10
    
    # 관계가 없으면 감점
    if len(extraction_result["relationships"]) == 0:
        score -= 30
    
    # 메타데이터 누락 시 감점
    if not extraction_result["metadata"].get("summary"):
        score -= 20
    
    return max(0, score)

def process_document_with_review(document):
    """신뢰도 기반 검토 워크플로우"""
    result = extract_entities_fast(document.text)
    confidence = calculate_confidence(result)
    
    if confidence < 70:
        # 사람 검토 큐에 추가
        review_queue.add({
            "document_id": document.id,
            "extraction": result,
            "confidence": confidence
        })
        logger.warn(f"낮은 신뢰도 ({confidence}): 검토 필요")
    else:
        # 자동 승인
        save_to_databases(result)
    
    return result, confidence
```

**3. 피드백 루프**:

사람 검토 결과를 시스템 프롬프트에 반영:

```python
# 검토 결과 수집
corrections = collect_human_corrections()

# 예시 기반 프롬프트 개선
few_shot_examples = generate_few_shot_examples(corrections)

IMPROVED_PROMPT = SYSTEM_PROMPT + f"""

**예시**:

올바른 추출:
{few_shot_examples["good"]}

잘못된 추출 (피해야 함):
{few_shot_examples["bad"]}
"""
```

---

## 5. 데이터 파이프라인 설계

### 5.1 전체 파이프라인 흐름

문서가 시스템에 입력되면 다음 단계를 거쳐 검색 가능한 지식으로 변환됩니다:

```
[문서 수집] → [전처리] → [GraphRAG 처리] → [3-way 저장] → [인덱싱] → [검색 준비 완료]
```

각 단계를 상세히 살펴보겠습니다.

**1단계: 문서 수집**

사내 다양한 저장소에서 문서를 자동으로 수집합니다:

```python
from langchain.document_loaders import (
    PyPDFLoader,
    Docx2txtLoader,
    UnstructuredMarkdownLoader
)
import os
from pathlib import Path

class DocumentCollector:
    """다양한 소스에서 문서 수집"""
    
    def __init__(self, watch_directories):
        self.watch_dirs = watch_directories
        self.loaders = {
            '.pdf': PyPDFLoader,
            '.docx': Docx2txtLoader,
            '.md': UnstructuredMarkdownLoader,
            '.txt': TextLoader
        }
    
    def collect_all(self):
        """모든 감시 디렉토리에서 문서 수집"""
        documents = []
        
        for directory in self.watch_dirs:
            docs = self.collect_from_directory(directory)
            documents.extend(docs)
        
        logger.info(f"총 {len(documents)}개 문서 수집 완료")
        return documents
    
    def collect_from_directory(self, directory):
        """특정 디렉토리에서 문서 수집"""
        documents = []
        
        for file_path in Path(directory).rglob('*'):
            if file_path.is_file():
                ext = file_path.suffix.lower()
                
                if ext in self.loaders:
                    try:
                        loader = self.loaders[ext](str(file_path))
                        docs = loader.load()
                        
                        # 메타데이터 추가
                        for doc in docs:
                            doc.metadata.update({
                                'source_path': str(file_path),
                                'file_type': ext,
                                'collected_at': datetime.now().isoformat()
                            })
                        
                        documents.extend(docs)
                        logger.info(f"수집: {file_path.name}")
                        
                    except Exception as e:
                        logger.error(f"수집 실패 {file_path}: {e}")
        
        return documents

# 사용
collector = DocumentCollector([
    "/mnt/shared/documents",
    "/mnt/projects",
    "/mnt/wiki"
])

documents = collector.collect_all()
```

**2단계: 전처리**

수집된 문서를 정제하고 메타데이터를 보강합니다:

```python
class DocumentPreprocessor:
    """문서 전처리"""
    
    def preprocess(self, document):
        """문서 정제 및 메타데이터 추출"""
        
        # 텍스트 정제
        text = self.clean_text(document.page_content)
        
        # 파일명에서 프로젝트명 추론
        project_name = self.infer_project_name(document.metadata['source_path'])
        
        # 문서 생성일 추출
        created_at = self.extract_creation_date(document.metadata)
        
        # 메타데이터 보강
        document.page_content = text
        document.metadata.update({
            'project_name': project_name,
            'created_at': created_at,
            'word_count': len(text.split()),
            'language': self.detect_language(text)
        })
        
        return document
    
    def clean_text(self, text):
        """텍스트 정제"""
        import re
        
        # 과도한 공백 제거
        text = re.sub(r'\s+', ' ', text)
        
        # 특수문자 정규화
        text = re.sub(r'[\x00-\x1F\x7F]', '', text)
        
        # 중복 줄바꿈 제거
        text = re.sub(r'\n{3,}', '\n\n', text)
        
        return text.strip()
    
    def infer_project_name(self, file_path):
        """파일 경로에서 프로젝트명 추론"""
        # 예: /mnt/projects/project_a/docs/guide.pdf → "project_a"
        parts = Path(file_path).parts
        
        if 'projects' in parts:
            idx = parts.index('projects')
            if idx + 1 < len(parts):
                return parts[idx + 1]
        
        return None
    
    def extract_creation_date(self, metadata):
        """파일 생성일 추출"""
        source_path = metadata.get('source_path')
        
        if source_path and os.path.exists(source_path):
            stat = os.stat(source_path)
            return datetime.fromtimestamp(stat.st_ctime)
        
        return datetime.now()
    
    def detect_language(self, text):
        """언어 감지"""
        # 간단한 휴리스틱 (실무에서는 langdetect 사용)
        korean_chars = len(re.findall(r'[가-힣]', text))
        total_chars = len(re.findall(r'\S', text))
        
        if korean_chars / total_chars > 0.3:
            return 'ko'
        return 'en'
```

**3단계: DeepSeek 엔티티 추출**

전처리된 문서에서 GraphRAG 스타일로 엔티티와 관계를 추출합니다:

```python
def extract_graphrag_data(document):
    """DeepSeek으로 GraphRAG 데이터 추출"""
    
    # Non-thinking 모드로 기본 추출
    basic_extraction = extract_entities_fast(document.page_content)
    
    # 문서가 중요하거나 복잡하면 Thinking 모드 추가
    if document.metadata.get('word_count', 0) > 2000:
        complex_extraction = extract_complex_relationships(document.page_content)
        
        # 두 결과 병합
        basic_extraction["temporal_chain"] = complex_extraction.get("temporal_chain", [])
        basic_extraction["dependencies"] = complex_extraction.get("dependencies", [])
        basic_extraction["causal_links"] = complex_extraction.get("causal_links", [])
    
    return basic_extraction
```

**4단계: 3-way 데이터베이스 저장**

추출된 데이터를 PostgreSQL, Neo4j, Elasticsearch에 동시 저장:

```python
def save_to_three_databases(document, extracted_data):
    """3개 DB에 동시 저장"""
    
    try:
        # 1. PostgreSQL: 마스터 레코드
        knowledge_id = save_to_postgres(document, extracted_data)
        
        # 2. Neo4j: 그래프
        save_to_neo4j(knowledge_id, extracted_data)
        
        # 3. Elasticsearch: 벡터 + 메타데이터
        save_to_elasticsearch(knowledge_id, document, extracted_data)
        
        logger.info(f"문서 {knowledge_id} 저장 완료")
        return knowledge_id
        
    except Exception as e:
        logger.error(f"저장 실패: {e}")
        # 재시도 큐에 추가
        retry_queue.add({
            'document': document,
            'extracted_data': extracted_data,
            'error': str(e)
        })
        raise

def save_to_postgres(document, extracted_data):
    """PostgreSQL 마스터 레코드 저장"""
    metadata = extracted_data["metadata"]
    
    knowledge_id = postgres.execute("""
        INSERT INTO knowledge_master (
            title,
            document_type,
            project_name,
            created_at,
            valid_start_date,
            valid_end_date,
            entities,
            summary,
            tags,
            source_path
        ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        RETURNING knowledge_id
    """, (
        metadata["title"] or document.metadata.get('source_path'),
        metadata["document_type"],
        metadata["project_name"],
        document.metadata['created_at'],
        metadata.get("valid_start_date"),
        metadata.get("valid_end_date"),
        json.dumps(extracted_data["entities"]),
        metadata["summary"],
        json.dumps(metadata.get("tags", [])),
        document.metadata['source_path']
    ))
    
    return knowledge_id

def save_to_neo4j(knowledge_id, extracted_data):
    """Neo4j 그래프 저장 (Slim Graph)"""
    
    # Knowledge 노드 생성
    neo4j.query("""
        CREATE (k:Knowledge {
            knowledge_id: $kid,
            title: $title,
            type: $doc_type
        })
    """, {
        "kid": knowledge_id,
        "title": extracted_data["metadata"]["title"],
        "doc_type": extracted_data["metadata"]["document_type"]
    })
    
    # 엔티티 노드 생성 및 관계 연결
    for person in extracted_data["entities"]["persons"]:
        # MERGE: 이미 있으면 재사용
        neo4j.query("""
            MERGE (e:Entity {name: $name, type: 'PERSON'})
            WITH e
            MATCH (k:Knowledge {knowledge_id: $kid})
            MERGE (e)-[:MENTIONED_IN]->(k)
        """, {"name": person, "kid": knowledge_id})
    
    # 프로젝트 연결
    if extracted_data["metadata"]["project_name"]:
        neo4j.query("""
            MERGE (e:Entity {name: $name, type: 'PROJECT'})
            WITH e
            MATCH (k:Knowledge {knowledge_id: $kid})
            MERGE (k)-[:ABOUT]->(e)
        """, {"name": extracted_data["metadata"]["project_name"], "kid": knowledge_id})
    
    # 기술 스택 연결
    for tech in extracted_data["entities"]["technologies"]:
        neo4j.query("""
            MERGE (e:Entity {name: $name, type: 'TECHNOLOGY'})
            WITH e
            MATCH (k:Knowledge {knowledge_id: $kid})
            MERGE (k)-[:USES_TECH]->(e)
        """, {"name": tech, "kid": knowledge_id})
    
    # 관계 생성
    for rel in extracted_data["relationships"]:
        neo4j.query("""
            MATCH (source:Entity {name: $source})
            MATCH (target:Entity {name: $target})
            MERGE (source)-[r:RELATED_TO {type: $rel_type}]->(target)
        """, {
            "source": rel["source"],
            "target": rel["target"],
            "rel_type": rel["type"]
        })

def save_to_elasticsearch(knowledge_id, document, extracted_data):
    """Elasticsearch 벡터 + 메타데이터 저장"""
    
    # 문서 청킹
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=800,
        chunk_overlap=100
    )
    chunks = text_splitter.split_text(document.page_content)
    
    # 임베딩 생성 (BGE-M3)
    embeddings = bge_m3_model.encode(chunks)
    
    # 각 청크를 ES에 저장
    for i, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
        doc_to_index = {
            "text": chunk,
            "vector_field": embedding.tolist(),
            "metadata": {
                "knowledge_id": knowledge_id,
                "title": extracted_data["metadata"]["title"],
                "document_type": extracted_data["metadata"]["document_type"],
                "project_name": extracted_data["metadata"]["project_name"],
                "valid_start_date": extracted_data["metadata"].get("valid_start_date"),
                "valid_end_date": extracted_data["metadata"].get("valid_end_date"),
                "entities": extracted_data["entities"],
                "chunk_index": i,
                "total_chunks": len(chunks),
                "source_path": document.metadata['source_path']
            }
        }
        
        es_client.index(
            index="knowledge-base",
            id=f"{knowledge_id}_chunk_{i}",
            body=doc_to_index
        )
```

**5단계: 인덱싱 최적화**

저장이 완료되면 검색 성능을 위한 인덱스를 최적화합니다:

```python
def optimize_indexes():
    """인덱스 최적화"""
    
    # PostgreSQL 인덱스 재구축
    postgres.execute("REINDEX TABLE knowledge_master")
    
    # Neo4j 인덱스 통계 업데이트
    neo4j.query("CALL db.stats.retrieve('GRAPH COUNTS')")
    
    # Elasticsearch 강제 병합
    es_client.indices.forcemerge(index="knowledge-base", max_num_segments=1)
    
    logger.info("인덱스 최적화 완료")
```

### 5.2 배치 처리 최적화

대량의 문서를 처리할 때는 배치 처리로 효율을 극대화합니다:

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

class BatchProcessor:
    """대량 문서 배치 처리"""
    
    def __init__(self, batch_size=10, max_workers=4):
        self.batch_size = batch_size
        self.max_workers = max_workers
    
    def process_documents(self, documents):
        """전체 문서 처리 오케스트레이션"""
        
        total = len(documents)
        processed = 0
        failed = []
        
        # 배치 단위로 분할
        batches = [
            documents[i:i+self.batch_size]
            for i in range(0, total, self.batch_size)
        ]
        
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            futures = [
                executor.submit(self.process_batch, batch, batch_idx)
                for batch_idx, batch in enumerate(batches)
            ]
            
            for future in futures:
                try:
                    batch_result = future.result()
                    processed += batch_result['success']
                    failed.extend(batch_result['failed'])
                except Exception as e:
                    logger.error(f"배치 처리 실패: {e}")
        
        return {
            'total': total,
            'processed': processed,
            'failed': len(failed),
            'failed_documents': failed
        }
    
    def process_batch(self, batch, batch_idx):
        """단일 배치 처리"""
        success = 0
        failed = []
        
        logger.info(f"배치 {batch_idx} 처리 시작 ({len(batch)}개 문서)")
        
        for doc in batch:
            try:
                # 전처리
                preprocessed = preprocessor.preprocess(doc)
                
                # 엔티티 추출
                extracted = extract_graphrag_data(preprocessed)
                
                # 3-way 저장
                knowledge_id = save_to_three_databases(preprocessed, extracted)
                
                success += 1
                logger.debug(f"처리 완료: {knowledge_id}")
                
            except Exception as e:
                failed.append({
                    'document': doc.metadata.get('source_path'),
                    'error': str(e)
                })
                logger.error(f"문서 처리 실패: {e}")
        
        logger.info(f"배치 {batch_idx} 완료 (성공: {success}, 실패: {len(failed)})")
        
        return {
            'success': success,
            'failed': failed
        }

# 사용
processor = BatchProcessor(batch_size=20, max_workers=4)
result = processor.process_documents(documents)

print(f"""
배치 처리 결과:
- 총 문서: {result['total']}
- 성공: {result['processed']}
- 실패: {result['failed']}
""")
```

### 5.3 증분 업데이트

기존 문서가 수정되었을 때 전체 재처리 대신 변경 부분만 업데이트:

```python
def incremental_update(document_id, new_document):
    """증분 업데이트"""
    
    # 1. 기존 데이터 조회
    old_metadata = postgres.get_knowledge(document_id)
    old_chunks = es_client.search(
        index="knowledge-base",
        body={"query": {"term": {"metadata.knowledge_id.keyword": document_id}}},
        size=1000
    )['hits']['hits']
    
    # 2. 새 문서 처리
    new_extracted = extract_graphrag_data(new_document)
    
    # 3. 변경 감지
    changes = detect_changes(old_metadata, new_extracted)
    
    # 4. 변경 부분만 업데이트
    if changes['metadata_changed']:
        update_postgres(document_id, new_extracted["metadata"])
    
    if changes['entities_changed']:
        update_neo4j_graph(document_id, new_extracted["entities"])
    
    if changes['content_changed']:
        update_elasticsearch_chunks(document_id, new_document, new_extracted)
    
    logger.info(f"증분 업데이트 완료: {document_id}")
    return changes

def detect_changes(old_metadata, new_extracted):
    """변경 감지"""
    return {
        'metadata_changed': old_metadata['summary'] != new_extracted["metadata"]["summary"],
        'entities_changed': old_metadata['entities'] != new_extracted["entities"],
        'content_changed': True  # 내용은 항상 변경으로 간주 (간단화)
    }

def update_elasticsearch_chunks(document_id, new_document, new_extracted):
    """Elasticsearch 청크 업데이트"""
    
    # 기존 청크 삭제
    es_client.delete_by_query(
        index="knowledge-base",
        body={"query": {"term": {"metadata.knowledge_id.keyword": document_id}}}
    )
    
    # 새 청크 추가
    save_to_elasticsearch(document_id, new_document, new_extracted)
```

---

## 6. 16GB RAM 환경 최적화

### 6.1 메모리 분배 전략

16GB RAM 환경에서 안정적으로 운영하기 위한 메모리 분배:

| 컴포넌트 | 할당 메모리 | 역할 | 최적화 전략 |
|---------|------------|------|------------|
| PostgreSQL | 1-1.5GB | 마스터 레코드 | Shared buffers 512MB, 나머지는 디스크 캐시 |
| Neo4j | 2-3GB | Slim 그래프 | Heap 2GB, PageCache 512MB |
| Elasticsearch | 4GB | 벡터 + 메타 | JVM Heap 4GB (Xms=Xmx) |
| BGE-M3 | 2-3GB | 임베딩 추론 | ONNX CPU 모드 |
| OS + App | 6-7GB | 시스템 | Python, Docker, 캐시 |
| **총계** | **16GB** | - | 사용률 85% 이하 유지 |

**Docker Compose 메모리 제한**:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: knowledge
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      # 메모리 최적화
      POSTGRES_SHARED_BUFFERS: 512MB
      POSTGRES_EFFECTIVE_CACHE_SIZE: 1GB
      POSTGRES_WORK_MEM: 16MB
    deploy:
      resources:
        limits:
          memory: 1536m  # 1.5GB 상한
        reservations:
          memory: 1024m  # 1GB 보장
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

  neo4j:
    image: neo4j:5.15
    environment:
      NEO4J_AUTH: neo4j/password
      # Slim Graph 최적화
      NEO4J_server_memory_heap_initial__size: 1g
      NEO4J_server_memory_heap_max__size: 2g
      NEO4J_server_memory_pagecache__size: 512m
      # 트랜잭션 로그 최소화
      NEO4J_db_tx__log_rotation_retention__policy: "1 days"
    deploy:
      resources:
        limits:
          memory: 3g
        reservations:
          memory: 2g
    volumes:
      - neo4j-data:/data
    restart: unless-stopped

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    environment:
      - node.name=es01
      - cluster.name=es-cluster
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"  # 고정 4GB
      - xpack.security.enabled=false
      # 메모리 효율 설정
      - indices.memory.index_buffer_size=20%
      - indices.fielddata.cache.size=30%
      - indices.queries.cache.size=10%
    deploy:
      resources:
        limits:
          memory: 6g  # OS 캐시 포함
        reservations:
          memory: 4g
    volumes:
      - es-data:/usr/share/elasticsearch/data
    restart: unless-stopped

volumes:
  postgres-data:
  neo4j-data:
  es-data:
```

### 6.2 PostgreSQL 최적화

16GB 환경에서의 PostgreSQL 설정 (`postgresql.conf`):

```conf
# 메모리 설정
shared_buffers = 512MB          # 전체 메모리의 25%
effective_cache_size = 1GB       # OS 캐시 포함
work_mem = 16MB                  # 정렬/조인 작업용
maintenance_work_mem = 128MB     # 인덱스 생성용

# 연결 제한
max_connections = 50

# 체크포인트
checkpoint_completion_target = 0.9
wal_buffers = 16MB

# 쿼리 최적화
random_page_cost = 1.1          # SSD 환경
effective_io_concurrency = 200

# 로깅 (최소화)
logging_collector = off
log_statement = 'none'
```

**쿼리 최적화 예시**:

```sql
-- 시계열 쿼리 최적화: 파티셔닝
CREATE TABLE knowledge_master_2023 PARTITION OF knowledge_master
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE knowledge_master_2024 PARTITION OF knowledge_master
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- 인덱스 최적화: 부분 인덱스
CREATE INDEX idx_active_knowledge 
    ON knowledge_master (valid_end_date)
    WHERE valid_end_date > CURRENT_DATE;

-- JSONB 인덱스
CREATE INDEX idx_entities_gin 
    ON knowledge_master USING gin (entities);
```

### 6.3 Neo4j 최적화

**Slim Graph 설정 (`neo4j.conf`)**:

```conf
# 메모리 설정
server.memory.heap.initial_size=1g
server.memory.heap.max_size=2g
server.memory.pagecache.size=512m

# 트랜잭션 최소화
db.tx_log.rotation.retention_policy=1 days
db.tx_log.rotation.size=25M

# 쿼리 캐시
db.query_cache_size=100

# 로깅 최소화
dbms.logs.query.enabled=false
```

**Slim Graph 패턴**:

```cypher
// ❌ Fat Graph (메모리 과다 사용)
CREATE (k:Knowledge {
    knowledge_id: "k123",
    title: "긴 제목...",
    content: "매우 긴 텍스트 내용...",  // 수백 KB
    summary: "긴 요약...",
    tags: ["태그1", "태그2", ...],
    created_at: "2023-01-01",
    updated_at: "2023-12-15"
})

// ✅ Slim Graph (메모리 최소화)
CREATE (k:Knowledge {
    knowledge_id: "k123",
    title: "React 가이드",  // 검색용 최소 정보만
    type: "TechnicalGuide"
    // 나머지는 PostgreSQL에서 조회
})
```

**효율적인 쿼리 패턴**:

```cypher
// ❌ 비효율적: 모든 속성 반환
MATCH (k:Knowledge)-[:CREATED]-(p:Person {name: "김철수"})
RETURN k

// ✅ 효율적: ID만 반환 → PostgreSQL에서 조회
MATCH (k:Knowledge)-[:CREATED]-(p:Person {name: "김철수"})
RETURN k.knowledge_id
```

### 6.4 Elasticsearch 최적화

**JVM 설정 (`jvm.options`)**:

```
# Heap 크기 (고정)
-Xms4g
-Xmx4g

# GC 설정 (G1GC)
-XX:+UseG1GC
-XX:G1ReservePercent=25
-XX:InitiatingHeapOccupancyPercent=30

# 메모리 오류 처리
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/lib/elasticsearch/heapdump.hprof
```

**인덱스 설정**:

```json
{
  "settings": {
    "number_of_shards": 1,      // 단일 노드
    "number_of_replicas": 0,    // 복제 없음 (메모리 절약)
    "refresh_interval": "5s",   // 실시간성 완화
    "index": {
      "codec": "best_compression",
      "max_result_window": 10000
    },
    "index.mapping.total_fields.limit": 2000
  }
}
```

**검색 쿼리 최적화**:

```python
# ❌ 비효율적: 모든 필드 반환
es.search(index="knowledge-base", body={
    "query": {...},
    "size": 10
})

# ✅ 효율적: 필요한 필드만
es.search(index="knowledge-base", body={
    "query": {...},
    "size": 10,
    "_source": ["text", "metadata.title", "metadata.project_name"]
})
```

### 6.5 BGE-M3 임베딩 최적화

CPU 모드에서 효율적으로 운영:

```python
from optimum.onnxruntime import ORTModelForFeatureExtraction
from transformers import AutoTokenizer
import numpy as np

class OptimizedEmbedding:
    """CPU 최적화 BGE-M3 임베딩"""
    
    def __init__(self, model_path="./models/bge-m3-onnx"):
        # ONNX 모델 로드 (CPU)
        self.model = ORTModelForFeatureExtraction.from_pretrained(
            model_path,
            provider="CPUExecutionProvider"
        )
        self.tokenizer = AutoTokenizer.from_pretrained(model_path)
        
        # 배치 처리 설정
        self.batch_size = 16
        
        logger.info("ONNX BGE-M3 모델 로드 완료 (CPU 모드)")
    
    def encode(self, texts, show_progress=True):
        """배치 인코딩"""
        if isinstance(texts, str):
            texts = [texts]
        
        embeddings = []
        total_batches = (len(texts) + self.batch_size - 1) // self.batch_size
        
        for i in range(0, len(texts), self.batch_size):
            batch = texts[i:i+self.batch_size]
            
            # 토크나이징
            inputs = self.tokenizer(
                batch,
                padding=True,
                truncation=True,
                max_length=512,
                return_tensors="pt"
            )
            
            # 추론
            outputs = self.model(**inputs)
            
            # Mean pooling
            batch_embeddings = self.mean_pooling(
                outputs.last_hidden_state,
                inputs['attention_mask']
            )
            
            embeddings.append(batch_embeddings.detach().numpy())
            
            if show_progress:
                progress = (i // self.batch_size + 1) / total_batches * 100
                print(f"\r임베딩 진행: {progress:.1f}%", end='')
        
        if show_progress:
            print()
        
        return np.vstack(embeddings)
    
    @staticmethod
    def mean_pooling(model_output, attention_mask):
        """Mean pooling"""
        token_embeddings = model_output
        input_mask_expanded = attention_mask.unsqueeze(-1).expand(token_embeddings.size()).float()
        return torch.sum(token_embeddings * input_mask_expanded, 1) / torch.clamp(input_mask_expanded.sum(1), min=1e-9)

# 사용
embedder = OptimizedEmbedding()
vectors = embedder.encode(["텍스트 1", "텍스트 2", ...])
```

**성능 비교**:

| 환경 | 속도 (docs/sec) | 메모리 사용 |
|------|----------------|------------|
| GPU (A100) | 200-300 | 8GB VRAM |
| ONNX CPU | 15-25 | 2-3GB RAM |
| PyTorch CPU | 5-10 | 4-5GB RAM |

CPU 환경에서는 ONNX가 3-5배 빠르고 메모리도 적게 사용합니다.

### 6.6 메모리 모니터링

실시간 메모리 사용량 모니터링:

```python
import psutil
import time

class MemoryMonitor:
    """메모리 사용량 모니터링"""
    
    def __init__(self, warning_threshold=85, critical_threshold=95):
        self.warning_threshold = warning_threshold
        self.critical_threshold = critical_threshold
    
    def get_usage(self):
        """현재 메모리 사용률"""
        memory = psutil.virtual_memory()
        
        return {
            'total_gb': memory.total / (1024**3),
            'used_gb': memory.used / (1024**3),
            'available_gb': memory.available / (1024**3),
            'percent': memory.percent
        }
    
    def check_and_alert(self):
        """메모리 체크 및 알림"""
        usage = self.get_usage()
        
        if usage['percent'] > self.critical_threshold:
            logger.critical(f"⚠️ 메모리 위험: {usage['percent']:.1f}% 사용중!")
            self.trigger_emergency_cleanup()
        
        elif usage['percent'] > self.warning_threshold:
            logger.warning(f"메모리 경고: {usage['percent']:.1f}% 사용중")
            self.trigger_soft_cleanup()
        
        return usage
    
    def trigger_soft_cleanup(self):
        """소프트 클린업"""
        # Python GC 실행
        import gc
        gc.collect()
        
        # 캐시 클리어
        clear_application_cache()
        
        logger.info("소프트 클린업 완료")
    
    def trigger_emergency_cleanup(self):
        """비상 클린업"""
        self.trigger_soft_cleanup()
        
        # Elasticsearch 캐시 클리어
        es_client.indices.clear_cache(index="knowledge-base")
        
        # Neo4j 페이지 캐시 클리어
        neo4j.query("CALL db.clearQueryCaches()")
        
        logger.info("비상 클린업 완료")
    
    def monitor_continuously(self, interval=60):
        """지속적 모니터링"""
        while True:
            usage = self.check_and_alert()
            print(f"메모리: {usage['used_gb']:.1f}/{usage['total_gb']:.1f} GB ({usage['percent']:.1f}%)")
            time.sleep(interval)

# 사용
monitor = MemoryMonitor(warning_threshold=85, critical_threshold=95)
# 백그라운드 스레드로 실행
import threading
monitor_thread = threading.Thread(target=monitor.monitor_continuously, daemon=True)
monitor_thread.start()
```

---

## 7. Elasticsearch와의 3-way 통합

### 7.1 제로 조인 아키텍처

기존 아키텍처의 문제점:
- PostgreSQL에서 메타데이터 조회 (1.5초)
- Elasticsearch에서 벡터 검색 (1.5초)
- 결과 조인 (0.5초)
- **총 3.5초**

제로 조인 아키텍처:
- Elasticsearch 단일 쿼리로 완결 (0.8초)
- **77% 응답 시간 단축**

**Elasticsearch 통합 스키마**:

```json
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "korean"
      },
      "vector_field": {
        "type": "dense_vector",
        "dims": 1024,
        "index": true,
        "similarity": "cosine"
      },
      "metadata": {
        "properties": {
          "knowledge_id": {"type": "keyword"},
          "title": {
            "type": "text",
            "fields": {"keyword": {"type": "keyword"}}
          },
          "document_type": {"type": "keyword"},
          "project_name": {
            "type": "text",
            "fields": {"keyword": {"type": "keyword"}}
          },
          "valid_start_date": {"type": "date"},
          "valid_end_date": {"type": "date"},
          "entities": {
            "properties": {
              "persons": {"type": "keyword"},
              "technologies": {"type": "keyword"},
              "keywords": {"type": "keyword"}
            }
          },
          "chunk_index": {"type": "integer"},
          "total_chunks": {"type": "integer"}
        }
      }
    }
  }
}
```

**제로 조인 검색 구현**:

```python
def zero_join_search(query_text, filters=None):
    """단일 쿼리로 벡터 + 메타데이터 검색"""
    
    # 1. 쿼리 임베딩
    query_vector = embedder.encode([query_text])[0]
    
    # 2. 필터 구성
    must_clauses = []
    filter_clauses = []
    
    # 텍스트 매칭
    must_clauses.append({
        "match": {
            "text": {
                "query": query_text,
                "boost": 0.3
            }
        }
    })
    
    # 필터 적용
    if filters:
        if 'time_range' in filters:
            filter_clauses.append({
                "range": {
                    "metadata.valid_start_date": {
                        "lte": filters['time_range']['end']
                    }
                }
            })
            filter_clauses.append({
                "range": {
                    "metadata.valid_end_date": {
                        "gte": filters['time_range']['start']
                    }
                }
            })
        
        if 'project_name' in filters:
            filter_clauses.append({
                "term": {
                    "metadata.project_name.keyword": filters['project_name']
                }
            })
        
        if 'document_type' in filters:
            filter_clauses.append({
                "term": {
                    "metadata.document_type": filters['document_type']
                }
            })
    
    # 3. 하이브리드 쿼리 (벡터 + BM25 + 필터)
    search_body = {
        "query": {
            "script_score": {
                "query": {
                    "bool": {
                        "must": must_clauses,
                        "filter": filter_clauses
                    }
                },
                "script": {
                    "source": "cosineSimilarity(params.query_vector, 'vector_field') + 1.0",
                    "params": {
                        "query_vector": query_vector.tolist()
                    }
                }
            }
        },
        "size": 5,
        "_source": ["text", "metadata"]  # 필요한 필드만
    }
    
    # 4. 단일 쿼리 실행
    results = es_client.search(
        index="knowledge-base",
        body=search_body
    )
    
    # 5. 결과 포맷팅
    hits = []
    for hit in results['hits']['hits']:
        hits.append({
            'content': hit['_source']['text'],
            'metadata': hit['_source']['metadata'],
            'score': hit['_score']
        })
    
    return hits

# 사용 예시
results = zero_join_search(
    query_text="2023년 보안 취약점 대응",
    filters={
        'time_range': {
            'start': '2023-01-01',
            'end': '2023-12-31'
        },
        'project_name': '프로젝트 A',
        'document_type': '기술문서'
    }
)

# PostgreSQL 조회 없이 모든 정보 즉시 접근
for result in results:
    print(f"제목: {result['metadata']['title']}")
    print(f"프로젝트: {result['metadata']['project_name']}")
    print(f"유효기간: {result['metadata']['valid_start_date']} ~ {result['metadata']['valid_end_date']}")
    print(f"관련 인물: {', '.join(result['metadata']['entities']['persons'])}")
    print(f"기술: {', '.join(result['metadata']['entities']['technologies'])}")
```

### 7.2 하이브리드 검색 (Dense + Sparse)

BGE-M3는 Dense와 Sparse 벡터를 모두 생성할 수 있습니다.

**Dense + Sparse 벡터 생성**:

```python
from FlagEmbedding import BGEM3FlagModel

class HybridEmbedder:
    """BGE-M3 Dense + Sparse 임베딩"""
    
    def __init__(self):
        self.model = BGEM3FlagModel(
            'BAAI/bge-m3',
            use_fp16=False,  # CPU 환경
            device='cpu'
        )
    
    def encode(self, texts):
        """Dense + Sparse 동시 생성"""
        output = self.model.encode(
            sentences=texts,
            return_dense=True,
            return_sparse=True,
            return_colbert_vecs=False  # ColBERT는 선택적
        )
        
        return {
            'dense': output['dense_vecs'],      # shape: (N, 1024)
            'sparse': output['lexical_weights']  # list of dict: {token: weight}
        }

# 사용
embedder = HybridEmbedder()
result = embedder.encode(["프로젝트 A의 React 아키텍처 가이드입니다."])

print(f"Dense 벡터: {result['dense'][0][:5]}...")  # [0.123, -0.456, ...]
print(f"Sparse 벡터: {result['sparse'][0]}")       # {"react": 2.13, "아키텍처": 1.87, ...}
```

**Elasticsearch 저장**:

```python
# Sparse 벡터 필드 추가
es_client.indices.put_mapping(
    index="knowledge-base",
    body={
        "properties": {
            "sparse_vector": {
                "type": "sparse_vector"
            }
        }
    }
)

# 문서 저장
doc = {
    "text": "프로젝트 A의 React 아키텍처...",
    "vector_field": result['dense'][0].tolist(),      # Dense
    "sparse_vector": result['sparse'][0],             # Sparse
    "metadata": {...}
}

es_client.index(index="knowledge-base", body=doc)
```

**RRF 하이브리드 검색** (Python ranx 사용):

```python
from ranx import Run, fuse

def hybrid_search_with_rrf(query_text):
    """Dense + Sparse + BM25 하이브리드 검색"""
    
    # 1. 쿼리 임베딩
    query_emb = embedder.encode([query_text])
    query_dense = query_emb['dense'][0]
    query_sparse = query_emb['sparse'][0]
    
    # 2. Dense 검색
    dense_results = es_client.search(
        index="knowledge-base",
        body={
            "query": {
                "script_score": {
                    "query": {"match_all": {}},
                    "script": {
                        "source": "cosineSimilarity(params.q, 'vector_field') + 1.0",
                        "params": {"q": query_dense.tolist()}
                    }
                }
            },
            "size": 20
        }
    )
    
    # 3. Sparse 검색
    sparse_results = es_client.search(
        index="knowledge-base",
        body={
            "query": {
                "sparse_vector": {
                    "field": "sparse_vector",
                    "query_vector": query_sparse
                }
            },
            "size": 20
        }
    )
    
    # 4. BM25 검색
    bm25_results = es_client.search(
        index="knowledge-base",
        body={
            "query": {
                "match": {
                    "text": query_text
                }
            },
            "size": 20
        }
    )
    
    # 5. RRF 융합 (Python ranx)
    dense_run = Run({"q1": {doc['_id']: doc['_score'] for doc in dense_results['hits']['hits']}})
    sparse_run = Run({"q1": {doc['_id']: doc['_score'] for doc in sparse_results['hits']['hits']}})
    bm25_run = Run({"q1": {doc['_id']: doc['_score'] for doc in bm25_results['hits']['hits']}})
    
    fused = fuse(
        runs=[dense_run, sparse_run, bm25_run],
        method="rrf",
        params={"k": 60}
    )
    
    # 6. 상위 결과 반환
    top_doc_ids = list(fused.run["q1"].keys())[:5]
    
    # 7. 문서 조회
    results = []
    for doc_id in top_doc_ids:
        doc = es_client.get(index="knowledge-base", id=doc_id)
        results.append({
            'content': doc['_source']['text'],
            'metadata': doc['_source']['metadata'],
            'score': fused.run["q1"][doc_id]
        })
    
    return results
```

### 7.3 시계열 검색 최적화

시간 기반 질문에 대한 정확한 답변:

```python
def temporal_search(query_text, reference_date):
    """특정 시점 기준 검색"""
    
    return zero_join_search(
        query_text,
        filters={
            'time_range': {
                'start': reference_date,
                'end': reference_date
            }
        }
    )

def temporal_comparison(query_text, date1, date2):
    """두 시점 비교"""
    
    results1 = temporal_search(query_text, date1)
    results2 = temporal_search(query_text, date2)
    
    return {
        f'{date1}_results': results1,
        f'{date2}_results': results2,
        'comparison': compare_results(results1, results2)
    }

# 사용 예시
comparison = temporal_comparison(
    "보안 정책",
    "2023-06-01",
    "2024-06-01"
)
```

---

## 8. 실무 구현 가이드

### 8.1 프로젝트 초기 설정

**1. 환경 구성**

```bash
# 프로젝트 디렉토리 생성
mkdir graphrag-neo4j-system
cd graphrag-neo4j-system

# Python 가상환경
python3 -m venv venv
source venv/bin/activate

# 필수 패키지 설치
pip install langchain langchain-openai langchain-elasticsearch langchain-community
pip install neo4j elasticsearch psycopg2-binary
pip install pandas pyarrow  # Parquet 처리
pip install sentence-transformers optimum onnxruntime  # 임베딩
pip install ranx  # RRF
pip install python-dotenv pydantic
```

**2. Docker Compose 설정**

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: knowledge
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_SHARED_BUFFERS: 512MB
      POSTGRES_EFFECTIVE_CACHE_SIZE: 1GB
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    deploy:
      resources:
        limits:
          memory: 1536m

  neo4j:
    image: neo4j:5.15
    environment:
      NEO4J_AUTH: neo4j/password
      NEO4J_server_memory_heap_initial__size: 1g
      NEO4J_server_memory_heap_max__size: 2g
      NEO4J_server_memory_pagecache__size: 512m
    ports:
      - "7474:7474"  # Browser
      - "7687:7687"  # Bolt
    volumes:
      - neo4j-data:/data
    deploy:
      resources:
        limits:
          memory: 3g

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    environment:
      - node.name=es01
      - cluster.name=es-cluster
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms4g -Xmx4g"
      - xpack.security.enabled=false
      - indices.memory.index_buffer_size=20%
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    deploy:
      resources:
        limits:
          memory: 6g

volumes:
  postgres-data:
  neo4j-data:
  es-data:
```

**3. 데이터베이스 초기화**

`init-db.sql`:

```sql
CREATE TABLE knowledge_master (
    knowledge_id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    document_type VARCHAR(50),
    project_name VARCHAR(255),
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valid_start_date DATE,
    valid_end_date DATE,
    
    entities JSONB,
    summary TEXT,
    tags JSONB,
    source_path TEXT,
    
    CONSTRAINT valid_period_check CHECK (valid_end_date >= valid_start_date)
);

CREATE INDEX idx_knowledge_valid_period 
    ON knowledge_master (valid_start_date, valid_end_date);
CREATE INDEX idx_knowledge_project 
    ON knowledge_master (project_name);
CREATE INDEX idx_entities_gin 
    ON knowledge_master USING gin (entities);

CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(255) NOT NULL,
    start_date DATE,
    end_date DATE,
    status VARCHAR(50) DEFAULT 'ACTIVE'
);
```

**4. Neo4j 초기화**

`init-neo4j.cypher`:

```cypher
// 제약조건
CREATE CONSTRAINT entity_id_unique IF NOT EXISTS
FOR (e:Entity) REQUIRE e.id IS UNIQUE;

CREATE CONSTRAINT knowledge_id_unique IF NOT EXISTS
FOR (k:Knowledge) REQUIRE k.knowledge_id IS UNIQUE;

// 인덱스
CREATE INDEX entity_name_index IF NOT EXISTS
FOR (e:Entity) ON (e.name);

CREATE INDEX entity_type_index IF NOT EXISTS
FOR (e:Entity) ON (e.type);

CREATE FULLTEXT INDEX entity_fulltext IF NOT EXISTS
FOR (e:Entity) ON EACH [e.name];
```

### 8.2 핵심 클래스 구현

**데이터베이스 커넥터**:

```python
# db_connectors.py
from neo4j import GraphDatabase
from elasticsearch import Elasticsearch
import psycopg2
from psycopg2.extras import RealDictCursor
import os

class DatabaseConnectors:
    """3개 데이터베이스 커넥터"""
    
    def __init__(self):
        # PostgreSQL
        self.pg_conn = psycopg2.connect(
            host=os.getenv("POSTGRES_HOST", "localhost"),
            database=os.getenv("POSTGRES_DB", "knowledge"),
            user=os.getenv("POSTGRES_USER", "admin"),
            password=os.getenv("POSTGRES_PASSWORD", "secret")
        )
        
        # Neo4j
        self.neo4j_driver = GraphDatabase.driver(
            os.getenv("NEO4J_URI", "bolt://localhost:7687"),
            auth=(
                os.getenv("NEO4J_USER", "neo4j"),
                os.getenv("NEO4J_PASSWORD", "password")
            )
        )
        
        # Elasticsearch
        self.es_client = Elasticsearch(
            [os.getenv("ES_HOST", "http://localhost:9200")]
        )
        
        print("✓ PostgreSQL 연결")
        print("✓ Neo4j 연결")
        print("✓ Elasticsearch 연결")
    
    def close(self):
        """연결 종료"""
        self.pg_conn.close()
        self.neo4j_driver.close()
        print("데이터베이스 연결 종료")
```

**문서 프로세서**:

```python
# document_processor.py
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import ChatOpenAI
import json
import os

class DocumentProcessor:
    """문서 처리 파이프라인"""
    
    def __init__(self, db_connectors):
        self.dbs = db_connectors
        
        # DeepSeek 엔티티 추출기
        self.deepseek = ChatOpenAI(
            model="deepseek-chat",
            api_key=os.getenv("DEEPSEEK_API_KEY"),
            base_url="https://api.deepseek.com/v1",
            temperature=0
        )
        
        # 텍스트 분할기
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=800,
            chunk_overlap=100
        )
        
        # 임베딩 (추후 초기화)
        self.embedder = None
    
    def process_document(self, document):
        """단일 문서 처리"""
        
        # 1. 엔티티 추출
        extracted = self.extract_entities(document.page_content)
        
        # 2. PostgreSQL 저장
        knowledge_id = self.save_to_postgres(document, extracted)
        
        # 3. Neo4j 저장
        self.save_to_neo4j(knowledge_id, extracted)
        
        # 4. Elasticsearch 저장
        self.save_to_elasticsearch(knowledge_id, document, extracted)
        
        return knowledge_id
    
    def extract_entities(self, text):
        """DeepSeek 엔티티 추출"""
        SYSTEM_PROMPT = """당신은 지식 그래프 구축 전문가입니다.
문서에서 다음을 JSON으로 추출하세요:
{
    "entities": {
        "persons": [...],
        "projects": [...],
        "technologies": [...],
        "keywords": [...]
    },
    "relationships": [
        {"source": "김철수", "type": "PARTICIPATED", "target": "프로젝트 A"}
    ],
    "metadata": {
        "document_type": "기술문서",
        "project_name": "프로젝트 A",
        "valid_start_date": "2023-01-01",
        "valid_end_date": null,
        "summary": "3줄 요약..."
    }
}
"""
        
        messages = [
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": f"문서:\n\n{text[:4000]}"}  # 길이 제한
        ]
        
        response = self.deepseek.invoke(messages)
        return json.loads(response.content)
    
    def save_to_postgres(self, document, extracted):
        """PostgreSQL 저장"""
        cursor = self.dbs.pg_conn.cursor()
        
        metadata = extracted["metadata"]
        
        cursor.execute("""
            INSERT INTO knowledge_master (
                title, document_type, project_name,
                valid_start_date, valid_end_date,
                entities, summary, source_path
            ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            RETURNING knowledge_id
        """, (
            metadata.get("title", "Untitled"),
            metadata["document_type"],
            metadata["project_name"],
            metadata.get("valid_start_date"),
            metadata.get("valid_end_date"),
            json.dumps(extracted["entities"]),
            metadata["summary"],
            document.metadata.get('source_path')
        ))
        
        knowledge_id = cursor.fetchone()[0]
        self.dbs.pg_conn.commit()
        
        return knowledge_id
    
    def save_to_neo4j(self, knowledge_id, extracted):
        """Neo4j 그래프 저장"""
        with self.dbs.neo4j_driver.session() as session:
            # Knowledge 노드
            session.run("""
                CREATE (k:Knowledge {
                    knowledge_id: $kid,
                    title: $title,
                    type: $doc_type
                })
            """, {
                "kid": str(knowledge_id),
                "title": extracted["metadata"].get("title", "Untitled"),
                "doc_type": extracted["metadata"]["document_type"]
            })
            
            # 엔티티 및 관계
            for person in extracted["entities"]["persons"]:
                session.run("""
                    MERGE (e:Entity {name: $name, type: 'PERSON'})
                    WITH e
                    MATCH (k:Knowledge {knowledge_id: $kid})
                    MERGE (e)-[:MENTIONED_IN]->(k)
                """, {"name": person, "kid": str(knowledge_id)})
    
    def save_to_elasticsearch(self, knowledge_id, document, extracted):
        """Elasticsearch 저장"""
        # 청킹
        chunks = self.text_splitter.split_text(document.page_content)
        
        # 임베딩 (실제로는 배치 처리)
        if self.embedder:
            embeddings = self.embedder.encode(chunks)
        else:
            embeddings = [[0.0] * 1024] * len(chunks)  # 더미
        
        # 각 청크 저장
        for i, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
            doc = {
                "text": chunk,
                "vector_field": embedding if isinstance(embedding, list) else embedding.tolist(),
                "metadata": {
                    "knowledge_id": str(knowledge_id),
                    "title": extracted["metadata"].get("title", "Untitled"),
                    "document_type": extracted["metadata"]["document_type"],
                    "project_name": extracted["metadata"]["project_name"],
                    "valid_start_date": extracted["metadata"].get("valid_start_date"),
                    "valid_end_date": extracted["metadata"].get("valid_end_date"),
                    "entities": extracted["entities"],
                    "chunk_index": i,
                    "total_chunks": len(chunks)
                }
            }
            
            self.dbs.es_client.index(
                index="knowledge-base",
                id=f"{knowledge_id}_chunk_{i}",
                body=doc
            )
```

**검색 엔진**:

```python
# search_engine.py
class SearchEngine:
    """Hybrid RAG 검색 엔진"""
    
    def __init__(self, db_connectors, embedder):
        self.dbs = db_connectors
        self.embedder = embedder
    
    def search(self, query_text, filters=None, top_k=5):
        """제로 조인 하이브리드 검색"""
        
        # 쿼리 임베딩
        query_vector = self.embedder.encode([query_text])[0]
        
        # 필터 구성
        filter_clauses = []
        if filters:
            if 'time_range' in filters:
                filter_clauses.append({
                    "range": {
                        "metadata.valid_start_date": {
                            "lte": filters['time_range']['end']
                        }
                    }
                })
            
            if 'project_name' in filters:
                filter_clauses.append({
                    "term": {
                        "metadata.project_name.keyword": filters['project_name']
                    }
                })
        
        # Elasticsearch 쿼리
        search_body = {
            "query": {
                "script_score": {
                    "query": {
                        "bool": {
                            "must": [{"match": {"text": query_text}}],
                            "filter": filter_clauses
                        }
                    },
                    "script": {
                        "source": "cosineSimilarity(params.q, 'vector_field') + 1.0",
                        "params": {"q": query_vector.tolist()}
                    }
                }
            },
            "size": top_k,
            "_source": ["text", "metadata"]
        }
        
        results = self.dbs.es_client.search(
            index="knowledge-base",
            body=search_body
        )
        
        # 포맷팅
        hits = []
        for hit in results['hits']['hits']:
            hits.append({
                'content': hit['_source']['text'],
                'metadata': hit['_source']['metadata'],
                'score': hit['_score']
            })
        
        return hits
    
    def graph_search(self, entity_name, max_hops=2):
        """Neo4j 그래프 탐색"""
        with self.dbs.neo4j_driver.session() as session:
            result = session.run(f"""
                MATCH path = (e:Entity {{name: $name}})-[*1..{max_hops}]-(related)
                RETURN related.knowledge_id AS kid, related.name AS name
                LIMIT 10
            """, {"name": entity_name})
            
            return [dict(record) for record in result]
```

**메인 애플리케이션**:

```python
# main.py
from db_connectors import DatabaseConnectors
from document_processor import DocumentProcessor
from search_engine import SearchEngine
from langchain.document_loaders import PyPDFLoader
import os

def main():
    # 1. 데이터베이스 연결
    print("=== 데이터베이스 연결 ===")
    dbs = DatabaseConnectors()
    
    # 2. 문서 프로세서 초기화
    print("\n=== 문서 프로세서 초기화 ===")
    processor = DocumentProcessor(dbs)
    
    # 3. 샘플 문서 처리
    print("\n=== 문서 처리 ===")
    loader = PyPDFLoader("./samples/sample.pdf")
    documents = loader.load()
    
    for i, doc in enumerate(documents[:3]):  # 처음 3개만
        print(f"문서 {i+1} 처리 중...")
        knowledge_id = processor.process_document(doc)
        print(f"  → 저장 완료: knowledge_id={knowledge_id}")
    
    # 4. 검색 엔진 초기화
    print("\n=== 검색 엔진 초기화 ===")
    # 실제 임베딩 모델 로드 (예시)
    from optimized_embedding import OptimizedEmbedding
    embedder = OptimizedEmbedding()
    processor.embedder = embedder
    
    search_engine = SearchEngine(dbs, embedder)
    
    # 5. 검색 테스트
    print("\n=== 검색 테스트 ===")
    results = search_engine.search(
        "보안 취약점 대응 방법",
        filters={
            'time_range': {'start': '2023-01-01', 'end': '2023-12-31'}
        }
    )
    
    for i, result in enumerate(results):
        print(f"\n결과 {i+1}:")
        print(f"제목: {result['metadata']['title']}")
        print(f"점수: {result['score']:.4f}")
        print(f"내용: {result['content'][:200]}...")
    
    # 6. 종료
    dbs.close()

if __name__ == "__main__":
    main()
```

---

## 9. 성능 최적화 전략

### 9.1 캐싱 전략

**애플리케이션 레벨 캐시**:

```python
from functools import lru_cache
import hashlib
import time

class SmartCache:
    """시간 기반 LRU 캐시"""
    
    def __init__(self, maxsize=1000, ttl=3600):
        self.cache = {}
        self.maxsize = maxsize
        self.ttl = ttl  # 1시간
    
    def _key(self, query, filters):
        """캐시 키 생성"""
        data = f"{query}_{json.dumps(filters, sort_keys=True)}"
        return hashlib.md5(data.encode()).hexdigest()
    
    def get(self, query, filters=None):
        """캐시 조회"""
        key = self._key(query, filters or {})
        
        if key in self.cache:
            value, timestamp = self.cache[key]
            
            # TTL 체크
            if time.time() - timestamp < self.ttl:
                return value
            else:
                del self.cache[key]
        
        return None
    
    def set(self, query, filters, value):
        """캐시 저장"""
        key = self._key(query, filters or {})
        
        # 캐시 크기 제한
        if len(self.cache) >= self.maxsize:
            # 가장 오래된 항목 삭제
            oldest = min(self.cache.items(), key=lambda x: x[1][1])
            del self.cache[oldest[0]]
        
        self.cache[key] = (value, time.time())

# 검색에 캐시 적용
cache = SmartCache(maxsize=500, ttl=1800)  # 30분

def cached_search(query, filters=None):
    # 캐시 확인
    cached = cache.get(query, filters)
    if cached:
        logger.info(f"캐시 히트: {query}")
        return cached
    
    # 검색 수행
    results = search_engine.search(query, filters)
    
    # 캐시 저장
    cache.set(query, filters, results)
    
    return results
```

**Elasticsearch 캐시 활용**:

```python
# 쿼리 캐시 활성화
es_client.indices.put_settings(
    index="knowledge-base",
    body={
        "index": {
            "queries.cache.enabled": True
        }
    }
)
```

### 9.2 비동기 처리

대량 작업을 비동기로 처리:

```python
import asyncio

async def async_process_documents(documents):
    """비동기 문서 처리"""
    tasks = [
        asyncio.create_task(process_document_async(doc))
        for doc in documents
    ]
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    success = sum(1 for r in results if not isinstance(r, Exception))
    failed = len(results) - success
    
    return {'success': success, 'failed': failed}

async def process_document_async(document):
    """비동기 문서 처리"""
    loop = asyncio.get_event_loop()
    
    # CPU-bound 작업을 별도 스레드에서 실행
    return await loop.run_in_executor(
        None,
        processor.process_document,
        document
    )

# 사용
results = asyncio.run(async_process_documents(documents))
```

### 9.3 인덱스 최적화

**Elasticsearch 인덱스 설정**:

```python
# 인덱스 설정 최적화
optimal_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0,
        "refresh_interval": "30s",  # 실시간성 완화
        "index.codec": "best_compression",
        "index.mapping.total_fields.limit": 2000,
        "max_result_window": 10000
    },
    "mappings": {
        # ... (이전 매핑)
    }
}

es_client.indices.create(
    index="knowledge-base-v2",
    body=optimal_settings
)

# 별칭 사용
es_client.indices.put_alias(
    index="knowledge-base-v2",
    name="knowledge-base"
)
```

**Neo4j 인덱스 전략**:

```cypher
// 복합 인덱스
CREATE INDEX entity_type_name IF NOT EXISTS
FOR (e:Entity) ON (e.type, e.name);

// 관계 인덱스
CREATE INDEX rel_type IF NOT EXISTS
FOR ()-[r:RELATED_TO]-() ON (r.type);
```

---

## 10. 운영 및 유지보수

### 10.1 모니터링 시스템

**성능 메트릭 수집**:

```python
import time
from collections import defaultdict

class MetricsCollector:
    """성능 메트릭 수집"""
    
    def __init__(self):
        self.metrics = defaultdict(list)
    
    def record(self, name, value):
        """메트릭 기록"""
        self.metrics[name].append({
            'value': value,
            'timestamp': time.time()
        })
    
    def get_stats(self, name, window=3600):
        """통계 계산 (최근 1시간)"""
        cutoff = time.time() - window
        recent = [
            m['value'] for m in self.metrics[name]
            if m['timestamp'] > cutoff
        ]
        
        if not recent:
            return {}
        
        return {
            'count': len(recent),
            'avg': sum(recent) / len(recent),
            'min': min(recent),
            'max': max(recent),
            'p95': sorted(recent)[int(len(recent) * 0.95)]
        }

# 검색 시간 기록
metrics = MetricsCollector()

def monitored_search(query, filters=None):
    start = time.time()
    
    try:
        results = search_engine.search(query, filters)
        elapsed = time.time() - start
        
        metrics.record('search_latency', elapsed)
        metrics.record('search_count', 1)
        
        return results
    except Exception as e:
        metrics.record('search_error', 1)
        raise

# 주기적 리포트
def print_metrics_report():
    print("\n=== 성능 리포트 (최근 1시간) ===")
    
    latency = metrics.get_stats('search_latency')
    if latency:
        print(f"검색 지연시간:")
        print(f"  평균: {latency['avg']:.2f}초")
        print(f"  P95: {latency['p95']:.2f}초")
        print(f"  최소/최대: {latency['min']:.2f}초 / {latency['max']:.2f}초")
```

### 10.2 백업 및 복구

**PostgreSQL 백업**:

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# PostgreSQL 덤프
docker exec postgres pg_dump -U admin knowledge > $BACKUP_DIR/postgres_$DATE.sql

# 압축
gzip $BACKUP_DIR/postgres_$DATE.sql

# 7일 이상 백업 삭제
find $BACKUP_DIR -name "postgres_*.sql.gz" -mtime +7 -delete

echo "PostgreSQL 백업 완료: postgres_$DATE.sql.gz"
```

**Neo4j 백업**:

```bash
#!/bin/bash
# neo4j-backup.sh

NEO4J_DIR="/var/lib/neo4j/data"
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Neo4j 중지
docker stop neo4j

# 데이터 복사
tar -czf $BACKUP_DIR/neo4j_$DATE.tar.gz -C $NEO4J_DIR .

# Neo4j 재시작
docker start neo4j

# 7일 이상 백업 삭제
find $BACKUP_DIR -name "neo4j_*.tar.gz" -mtime +7 -delete

echo "Neo4j 백업 완료: neo4j_$DATE.tar.gz"
```

**Elasticsearch 스냅샷**:

```python
# es_snapshot.py
def create_snapshot():
    """Elasticsearch 스냅샷 생성"""
    
    snapshot_name = f"snapshot_{int(time.time())}"
    
    # 스냅샷 생성
    es_client.snapshot.create(
        repository="backup_repo",
        snapshot=snapshot_name,
        body={
            "indices": "knowledge-base",
            "include_global_state": False
        }
    )
    
    print(f"스냅샷 생성: {snapshot_name}")
    
    # 오래된 스냅샷 삭제
    snapshots = es_client.snapshot.get(
        repository="backup_repo",
        snapshot="_all"
    )['snapshots']
    
    cutoff = time.time() - (7 * 24 * 3600)  # 7일
    
    for snap in snapshots:
        if snap['start_time_in_millis'] / 1000 < cutoff:
            es_client.snapshot.delete(
                repository="backup_repo",
                snapshot=snap['snapshot']
            )
            print(f"오래된 스냅샷 삭제: {snap['snapshot']}")
```

### 10.3 문제 해결 가이드

**자주 발생하는 이슈**:

1. **메모리 부족**
   - 증상: OOM 에러, 시스템 느려짐
   - 해결: Docker 메모리 제한 조정, 캐시 클리어, 인덱스 최적화

2. **검색 속도 저하**
   - 증상: 응답 시간 > 5초
   - 해결: 캐시 확인, ES 인덱스 병합, Neo4j 쿼리 최적화

3. **DeepSeek API 타임아웃**
   - 증상: 엔티티 추출 실패
   - 해결: 재시도 로직, 문서 길이 제한, Thinking 모드 사용

4. **데이터 동기화 실패**
   - 증상: 3개 DB 간 불일치
   - 해결: 정합성 검증 배치, 재동기화 스크립트

---

## 결론

Microsoft GraphRAG와 Neo4j를 통합한 Hybrid RAG 시스템은 기업의 지식 자산을 효과적으로 관리하고 활용할 수 있는 강력한 플랫폼입니다.

**핵심 성과**:
- ✅ **비용 효율성**: DeepSeek 활용으로 LLM 비용 93% 절감
- ✅ **성능 최적화**: 제로 조인 아키텍처로 검색 응답 시간 77% 단축
- ✅ **리소스 효율**: 16GB RAM 환경에서 안정적 운영
- ✅ **확장성**: 초기 소규모로 시작하여 점진적 확장 가능

**주요 설계 원칙**:
1. **Slim Graph**: Neo4j에는 관계만, 상세 정보는 PostgreSQL/ES에
2. **제로 조인**: Elasticsearch 메타데이터 통합 저장
3. **비용 최적화**: DeepSeek + 캐시 히트 전략
4. **모듈화**: 컴포넌트별 독립 업그레이드 가능

이 가이드를 따라 구축하면, 실무 환경에서 즉시 활용 가능한 지능형 지식 검색 시스템을 완성할 수 있습니다.

**작성 일자: 2026-01-12**
