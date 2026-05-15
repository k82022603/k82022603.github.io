---
title: "RAGChatbotServer 시스템 분석 보고서"
date: 2026-05-15 21:00:00 +0900
categories: [AI,  RAG & Knowledge Retrieval]
mermaid: [True]
tags: [AI,  RAG,  Elasticsearch,  OpenSearch,  PGVector,  hybrid-rag,  Claude.write]
---


> **분석 일자**: 2026-05-14  
> **분석 대상**: [`RAGChatbotServer`](https://github.com/k82022603/RAGChatbotServer-20250819-InsightHub) (Legacy)  
> **분석 목적**: 현재 시스템 구조 파악 및 **SearcheRAGWithGraphRAG** (Neo4j GraphRAG + Elasticsearch + PostgreSQL) 전환 설계를 위한 기반 분석

---

## 목차

1. [시스템 개요](#1-시스템-개요)
2. [현재 아키텍처 분석](#2-현재-아키텍처-분석)
3. [핵심 구성요소 분석](#3-핵심-구성요소-분석)
4. [발견된 문제점](#4-발견된-문제점)
5. [향후 Hybrid RAG 전환 설계](#5-향후-hybrid-rag-전환-설계)
6. [전환 로드맵](#6-전환-로드맵)
7. [결론 및 권고사항](#7-결론-및-권고사항)

---

## 1. 시스템 개요

### 1.1 프로젝트 목적

RAGChatbotServer는 PDF 문서 기반 RAG(Retrieval-Augmented Generation) 챗봇 서버로, 3종의 벡터 DB(Elasticsearch, OpenSearch, PGVector)를 환경변수 한 줄로 교체하며 다양한 검색 모드를 실험할 수 있는 **PoC(Proof of Concept) / Legacy 시스템**이다.

본 분석은 이 레거시 시스템에서 **계승할 자산과 버려야 할 부채를 식별**하고, SearcheRAGWithGraphRAG 신규 시스템 설계의 기반 자료로 활용하기 위해 수행되었다.

### 1.2 기술 스택 요약

| 영역 | 기술 | 비고 |
|------|------|------|
| API Server | FastAPI 3.0.0 + Uvicorn | |
| LLM | OpenAI GPT-4o-mini | LangChain 경유 |
| Embedding | OpenAI Embeddings (기본), Cohere, HuggingFace | 환경변수 선택 |
| Vector DB | Elasticsearch 8.x / OpenSearch 2.x / PGVector | 택 1 (신규: ES 단일화) |
| PDF 처리 | pdfplumber, PyPDF, Unstructured | |
| 프레임워크 | LangChain 0.3.x | |
| 인프라 | Docker Compose (DB별 분리) | |
| 모니터링 | Kibana / OpenSearch Dashboards / pgAdmin | |

### 1.3 현재 운영 설정 (.env 기준)

현재 `.env`에서 **Elasticsearch**를 기본 Vector Store로 설정하고 있으며, 이는 신규 시스템과 일치한다.

```
VECTOR_STORE_TYPE=elasticsearch
SEARCH_MODE=custom-hybrid
ELASTICSEARCH_INDEX=rcsv-pdf-documents
CHUNK_SIZE=2000 / CHUNK_OVERLAP=300
```

---

## 2. 현재 아키텍처 분석

### 2.1 전체 시스템 흐름

```
[Client]
  ├── React Frontend (:5175)
  └── Terminal Chat

       ↓ HTTP

[FastAPI Server :8000]  ←  app.py (REST API)
       ↓
  rag_system.py  →  SearchRAG (base class)
       │               ├── ElasticsearchRAG
       │               ├── OpenSearchRAG        ← 신규 시스템에서 제거
       │               └── PGVectorRAG          ← 신규 시스템에서 역할 변경
       ↓
  vector_store.py  →  환경변수 기반 DB 라우팅
       ↓
  [Vector DB - 택 1]
       ├── Elasticsearch :9200                  ← 신규 시스템: 이것만 사용
       ├── OpenSearch :9201                     ← 신규 시스템: 제거
       └── PGVector :5432                       ← 신규 시스템: SSOT + 수치 데이터로 역할 전환

[Data Pipeline]
  pdf_processor.py  →  embed_pdfs.py  →  Vector DB
       ↑
  OpenAI API (GPT-4o-mini + Embeddings)
```

### 2.2 파일 구조

```
RAGChatbotServer/  (Legacy)
├── .env                           # 환경변수 (현재: ES + custom-hybrid)
├── requirements.txt               # Python 의존성
│
├── app.py                         # FastAPI 메인 (432줄)
├── rag_system.py                  # RAG 기본 클래스 + DB별 서브클래스 (898줄)
├── vector_store.py                # 벡터 스토어 라우터
├── pdf_processor.py               # PDF 파싱 + LLM 메타데이터 생성
├── embed_pdfs.py                  # 배치 임베딩 CLI
├── chat_interface.py              # 터미널 챗봇
├── run_terminal_chat.py           # 터미널 실행 스크립트
├── test_search_modes.py           # 검색 모드 벤치마크
│
├── vector_store_elasticsearch.py  # ES 구현 (1,130줄) ← 재활용 대상
├── vector_store_opensearch.py     # OS 구현 (1,100줄) ← 제거 대상
├── vector_store_pgvector.py       # PGVector 구현 (1,380줄) ← 역할 전환
│
├── ElasticSearch/                 # ES 인프라 (Nori 한국어 포함) ← 재활용
├── OpenSearch/                    # OS 인프라 ← 제거
└── PGVector/                      # PG 인프라 (초기화 SQL + 백업) ← 스키마 재설계
```

### 2.3 클래스 계층 구조

```
SearchRAG (기본 클래스)
  ├── vectorstore
  ├── llm: ChatOpenAI (gpt-4o-mini)
  ├── retriever
  ├── search_mode: str
  ├── prompt_template: str
  ├── query(question, chat_history) → Dict
  ├── format_chat_history(history) → str
  ├── format_sources_info(docs) → str
  └── setup_retriever()

ElasticsearchRAG(SearchRAG)             ← 재활용 및 확장 예정
  └── configure_search_strategy(type, hybrid) → bool

OpenSearchRAG(SearchRAG)                ← 제거 예정
  └── set_search_weights(vector_w, keyword_w) → bool

PGVectorRAG(SearchRAG)                  ← 제거 예정 (PG는 SSOT 역할로 분리)
  ├── pgvector_manager: PGVectorManager
  └── ...
```

### 2.4 검색 모드 지원 매트릭스 (레거시 기준)

| 검색 모드 | Elasticsearch | OpenSearch | PGVector | 신규 시스템 |
|-----------|:---:|:---:|:---:|:---:|
| `vector` | ✅ | ✅ | ✅ | ✅ ES |
| `bm25` | ✅ | ✅ | ✅ | ✅ ES |
| `custom-hybrid` | ✅ | ✅ | ✅ | ✅ ES |
| `hybrid_search` | - | ✅ | - | - (제거) |
| `dense-vector` | ✅ | - | - | ✅ ES |
| `script-score` | ✅ | ✅ | - | ✅ ES |
| `similarity` | - | - | ✅ | - (제거) |
| `mmr` | - | - | ✅ | - (제거) |
| Graph Search | - | - | - | ✅ Neo4j 신규 |

### 2.5 API 엔드포인트 (레거시)

| Method | Path | 기능 | 신규 시스템 |
|--------|------|------|------------|
| `POST` | `/ai/chat` | 채팅 (질문 → 답변) | 계승 + GraphRAG 통합 |
| `POST` | `/reset_session` | 세션 초기화 | 계승 |
| `GET` | `/health` | 서버 상태 확인 | 계승 (LLM 쿼리 제거 필요) |
| `GET` | `/vector-store/info` | 벡터 스토어 정보 | 계승 + Graph 정보 추가 |
| `POST` | `/vector-store/search-mode` | 검색 모드 변경 | 계승 |
| `GET` | `/search-modes` | 사용 가능 검색 모드 목록 | 계승 |
| `GET` | `/sessions` | 활성 세션 목록 | 계승 |

---

## 3. 핵심 구성요소 분석

### 3.1 app.py (FastAPI 메인 서버)

**역할**: REST API 진입점, 세션 관리, RAG 시스템 라이프사이클 관리

**주요 특징**:
- 전역 변수로 `rag_system`, `chat_sessions` 관리 (Dict 기반 인메모리)
- 검색 모드 변경 시 `create_rag_instance()` 전체 재생성 방식
- `@app.on_event("startup/shutdown")` 사용 (deprecated API)
- 세션 히스토리 최근 10개 유지

**문제점**: `health_check()` 내부에서 `rag_system.query("테스트", [])` 실제 호출 → OpenAI API 과금 발생

### 3.2 rag_system.py (RAG 엔진)

**역할**: LLM + 벡터 검색 결합, 검색 모드별 retriever 구성

**주요 특징**:
- Strategy 패턴 적용: SearchRAG → ES/OS/PG 서브클래스
- Factory 함수 `create_rag_instance()`로 통일된 생성 인터페이스
- LangChain EnsembleRetriever 활용한 custom-hybrid 지원
- 한국어 최적화 프롬프트 템플릿 내장 → **신규 시스템에 계승**

**심각한 버그**: `PGVectorRAG.__init__` 이중 초기화 (미정의 변수 참조 → NameError)

### 3.3 pdf_processor.py (PDF 처리) — 핵심 재활용 자산

**역할**: PDF → 청킹 → LLM 메타데이터 자동 생성

**주요 특징 (신규 시스템에 계승 가치 높음)**:
- **LLM 기반 메타데이터 자동 생성**: 키워드/카테고리/요약/기술용어/개체명
- BM25 검색 최적화용 `all_search_text` 필드 합성
- 빈도 분석 기반 자동 키워드 보강 (`_extract_frequent_terms`)
- 3종 엔진 지원: pdfplumber(기본), pypdf, unstructured

→ **GraphRAG 전환 시 NER + 관계 추출 단계 추가 가능한 기반 코드**

### 3.4 vector_store_elasticsearch.py — 핵심 재활용 자산

**역할**: Elasticsearch 벡터 스토어 전체 구현 (1,130줄)

**계승 가치**:
- `ElasticsearchManager` 클래스: ES 연결, 인덱스 관리, 문서 추가/삭제
- DenseVector / BM25 / custom-hybrid 전략 구현
- 한국어 Nori 분석기 설정 포함
- 비동기 처리 지원 (`AsyncElasticsearch`)

### 3.5 인프라 구성 (신규 시스템 대비)

| 서비스 | 레거시 포트 | 신규 시스템 | 비고 |
|--------|------------|------------|------|
| FastAPI Server | 8000 | 8000 | 계승 |
| Elasticsearch | 9200 | 9200 | 계승 + 단일화 |
| Kibana | 5601 | 5601 | 계승 |
| OpenSearch | 9201 | **제거** | 신규에서 미사용 |
| OpenSearch Dashboards | 5602 | **제거** | |
| PostgreSQL | 5432 | 5432 | 역할 전환: SSOT + 수치 관리 |
| pgAdmin | 5050 | 5050 | 계승 |
| **Neo4j** | - | **7474/7687** | **신규 추가** |

---

## 4. 발견된 문제점

### 4.1 Critical (신규 시스템 설계 시 반드시 해결)

#### C-1. PGVectorRAG 이중 초기화 버그 (`rag_system.py`)

```python
class PGVectorRAG(SearchRAG):
    def __init__(self, pgvector_manager):
        super().__init__(...)   # ← 1차 초기화 (정상)
        
        # ↓ 2차 초기화 — connection_string 등 미정의 변수 참조
        self.pgvector_manager = create_pgvector_manager(
            connection_string=connection_string,   # NameError!
            collection_name=collection_name,       # NameError!
        )
```

**신규 시스템 교훈**: PGVectorRAG를 완전히 제거하고 PostgreSQL은 SSOT/수치 데이터 전용으로 분리. `super().__init__()` 중복 호출 패턴 금지.

#### C-2. Health Check에서 실제 RAG 쿼리 실행 (`app.py`)

```python
async def health_check():
    test_response = rag_system.query("테스트", [])  # 매 호출마다 LLM API 호출!
```

**신규 시스템 교훈**: Health Check은 DB ping + 연결 상태 확인만. LLM 호출 절대 금지.

#### C-3. API 인증/인가 완전 부재

**신규 시스템 교훈**: 처음부터 JWT / API Key 기반 인증 미들웨어 설계 필수.

### 4.2 Major (신규 시스템 설계 반영 필요)

| # | 문제 | 신규 시스템 대응 |
|---|------|----------------|
| M-1 | In-memory 세션 저장소 | Redis 기반 분산 세션 저장소 |
| M-2 | embed_pdfs.py PGVector 전용 하드코딩 | ES 전용 임베딩 파이프라인으로 재구현 |
| M-3 | `@app.on_event` deprecated | `lifespan` context manager 사용 |
| M-4 | 플랫 파일 구조 (루트에 모든 .py) | `src/` 패키지 구조 도입 |

### 4.3 설계 부채 (신규 시스템에서 처음부터 해결)

| 항목 | 레거시 현황 | 신규 시스템 목표 |
|------|-----------|----------------|
| 테스트 | 벤치마크 스크립트만 존재 | 유닛/통합 테스트 + CI |
| 컨테이너화 | DB용 Dockerfile만 존재 | 앱 포함 통합 Compose |
| 로깅 | 한/영 혼용, 무제한 파일 생성 | 구조화 로깅 (JSON) |
| 보안 | API 키 콘솔 출력 | 시크릿 관리 (Vault 등) |

---

## 5. 향후 Hybrid RAG 전환 설계

### 5.1 목표 아키텍처 (SearcheRAGWithGraphRAG)

```
┌─────────────────────────────────────────────────────────────────┐
│                SearcheRAGWithGraphRAG 목표 아키텍처              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Client / API Layer]                                           │
│      ↓ (JWT 인증)                                               │
│  [FastAPI Server :8000]                                         │
│      ↓                                                          │
│  ┌────────────────────────────────────────────────────────┐     │
│  │              Hybrid RAG Engine                         │     │
│  │                                                        │     │
│  │   ① Vector Search  ②  Graph Search  ③  Numeric Query  │     │
│  │         ↓                  ↓                 ↓         │     │
│  │   Elasticsearch        Neo4j           PostgreSQL      │     │
│  │   (Vector Store)     (GraphRAG)      (SSOT + 수치)     │     │
│  │   - Dense Vector     - 엔티티 노드    - 크기/밝기       │     │
│  │   - BM25             - 관계 엣지      - 색온도/전력     │     │
│  │   - custom-hybrid    - Cypher 쿼리   - 마스터 데이터   │     │
│  └────────────────────────────────────────────────────────┘     │
│      ↓                                                          │
│  [LLM Response Generation (통합 컨텍스트 기반)]                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 벡터 스토어: Elasticsearch 단일화

SearcheRAGWithGraphRAG 시스템에서는 **Elasticsearch만을 Vector Store로 사용**한다. OpenSearch와 PGVector(벡터 역할)는 제거한다.

#### 선택 근거

| 항목 | Elasticsearch | OpenSearch | PGVector |
|------|:---:|:---:|:---:|
| HNSW Dense Vector | ✅ 네이티브 | ✅ | ✅ |
| BM25 전문 검색 | ✅ 최적화 | ✅ | 제한적 |
| Nori 한국어 분석기 | ✅ 공식 | ✅ | - |
| Neo4j GraphRAG 통합 용이성 | ✅ | 보통 | 낮음 |
| Kibana 시각화 | ✅ | - | - |
| 운영 단순성 | ✅ 단일 집중 | - | - |

#### 레거시에서 계승할 ES 자산

| 자산 | 파일 | 계승 방법 |
|------|------|----------|
| ES 벡터 스토어 구현 | `vector_store_elasticsearch.py` | `ElasticsearchManager` 클래스 정제 후 재활용 |
| ES Docker 인프라 | `ElasticSearch/` 폴더 전체 | 직접 이식 (Nori 플러그인 포함) |
| ES 설정 파일 | `ElasticSearch/elasticsearch.yml` | 재활용 |
| ES Compose | `ElasticSearch/elasticsearch-compose.yml` | 통합 Compose로 흡수 |

### 5.3 Neo4j GraphRAG 통합 설계

#### 역할 및 책임

Neo4j는 **지식 그래프 기반 관계 추론** 역할을 담당한다. Elasticsearch의 벡터/키워드 검색만으로 포착하기 어려운 **엔티티 간 관계, 계층 구조, 경로 기반 추론**을 제공한다.

```
[문서 처리 파이프라인]
     ↓
[PDF 파싱 (pdf_processor.py 재활용)]
     ↓
[NER + 관계 추출 (LLM)] ← 신규 추가 단계
     ↓                            ↓
[Elasticsearch]              [Neo4j]
 - 텍스트 청크                - 엔티티 노드 (Product, Category 등)
 - 벡터 임베딩                - 관계 엣지 (RELATES_TO, IS_PART_OF)
 - 메타데이터                 - 속성 (PG의 product_id FK 포함)
     ↓                            ↓
     └────────────┬───────────────┘
                  ↓
         [HybridGraphRAG Engine]
         ① ES Vector Search
         ② Neo4j Cypher Graph Traversal
         ③ PostgreSQL Numeric Query
                  ↓
         [LLM 통합 응답 생성]
```

#### 신규 클래스 설계안

```python
# 신규 구현 예정: HybridGraphRAG
class HybridGraphRAG(SearchRAG):
    """
    Elasticsearch + Neo4j GraphRAG + PostgreSQL 수치 데이터 통합 RAG
    - SearchRAG (레거시)에서 ElasticsearchRAG만 계승
    """
    def __init__(self, es_manager, neo4j_driver, pg_conn):
        super().__init__(es_manager.vectorstore)
        self.neo4j_driver = neo4j_driver
        self.pg_conn = pg_conn
    
    def query(self, question, chat_history):
        # ① ES 벡터/BM25 검색
        vector_docs = self._vector_search(question)
        
        # ② Neo4j 그래프 탐색 (엔티티 관계 추론)
        graph_context = self._graph_search(question)
        
        # ③ PostgreSQL 수치 데이터 조회 (크기/밝기 등)
        numeric_data = self._pg_numeric_query(question)
        
        # ④ 통합 컨텍스트로 LLM 응답 생성
        return self._generate_response(
            vector_docs, graph_context, numeric_data,
            question, chat_history
        )
```

### 5.4 PostgreSQL: SSOT 및 수치 데이터 관리

#### 역할 전환 요약

| 항목 | 레거시 역할 (PGVector) | 신규 역할 (SSOT + 수치 관리) |
|------|----------------------|---------------------------|
| 주요 목적 | 벡터 유사도 검색 | 마스터 데이터 + 수치 관리 |
| 벡터 저장 | pgvector 익스텐션 사용 | **불필요** → ES로 완전 이전 |
| 데이터 구조 | Document 청크 + 벡터 컬럼 | 정규화된 관계형 스키마 |
| 쿼리 방식 | `<=>` (벡터 유사도) | SQL 집계/범위/조인 |
| 연동 | 단독 운영 | ES + Neo4j 데이터 허브 역할 |

#### 권장 스키마 설계 (초안)

```sql
-- 제품 마스터 (SSOT — 모든 데이터의 원천)
CREATE TABLE products (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_code VARCHAR(50)  UNIQUE NOT NULL,
    name         VARCHAR(200) NOT NULL,
    category     VARCHAR(100),
    created_at   TIMESTAMPTZ  DEFAULT NOW(),
    updated_at   TIMESTAMPTZ  DEFAULT NOW()
);

-- 수치 속성 (크기, 밝기 등 정형 데이터)
CREATE TABLE product_specs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id      UUID REFERENCES products(id) ON DELETE CASCADE,
    -- 크기
    width_mm        NUMERIC(10,2),   -- 가로 (mm)
    height_mm       NUMERIC(10,2),   -- 세로 (mm)
    depth_mm        NUMERIC(10,2),   -- 깊이 (mm)
    -- 광학 특성
    brightness_lm   NUMERIC(10,2),   -- 밝기 (루멘)
    color_temp_k    INTEGER,         -- 색온도 (K)
    -- 전기 특성
    wattage_w       NUMERIC(8,2),    -- 소비전력 (W)
    -- 물리 특성
    weight_kg       NUMERIC(8,3),    -- 무게 (kg)
    UNIQUE(product_id)
);

-- ES 인덱스 동기화 추적 (SSOT 역할의 핵심)
CREATE TABLE es_sync_log (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id  UUID REFERENCES products(id),
    es_doc_id   VARCHAR(200),  -- ES 문서 ID
    synced_at   TIMESTAMPTZ    DEFAULT NOW(),
    status      VARCHAR(20)    -- 'synced' | 'pending' | 'failed'
);

-- Neo4j 노드 동기화 추적
CREATE TABLE neo4j_sync_log (
    id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id     UUID REFERENCES products(id),
    neo4j_node_id  VARCHAR(200),  -- Neo4j 노드 ID
    synced_at      TIMESTAMPTZ    DEFAULT NOW(),
    status         VARCHAR(20)
);
```

### 5.5 레거시 → 신규 시스템 계승/폐기 결정표

#### ✅ 계승 (재활용)

| 자산 | 파일 | 계승 방법 |
|------|------|----------|
| ES 벡터 스토어 구현 | `vector_store_elasticsearch.py` | `ElasticsearchManager` 클래스 정제 |
| ES Docker 인프라 | `ElasticSearch/` | 직접 이식 (Nori 포함) |
| LLM 메타데이터 생성 | `pdf_processor.py` | `generate_complete_metadata_with_llm()` |
| FastAPI 기본 구조 | `app.py` | 인증 추가 후 재활용 |
| 한국어 최적화 프롬프트 | `rag_system.py` | 신규 `HybridGraphRAG`에 계승 |
| 검색 모드 벤치마크 | `test_search_modes.py` | GraphRAG 포함 확장 버전으로 발전 |
| PGVector 초기화 SQL | `PGVector/postgresql/init/` | 스키마 재설계 후 부분 재활용 |
| PG 성능 튜닝 설정 | `PGVector/postgresql/conf/` | 재활용 |

#### ❌ 폐기 (제거)

| 자산 | 파일 | 폐기 이유 |
|------|------|----------|
| OpenSearch 전체 | `OpenSearch/`, `vector_store_opensearch.py` | ES 단일화 결정 |
| PGVectorRAG 클래스 | `rag_system.py` 내 | 역할 분리 (SSOT로 전환) |
| vector_store_pgvector.py | `vector_store_pgvector.py` | 벡터 역할 제거 |
| embed_pdfs.py 현재 버전 | `embed_pdfs.py` | ES 전용 재구현 필요 |

#### 🆕 신규 구현 필요

| 항목 | 설명 |
|------|------|
| Neo4j 연동 모듈 | `vector_store_neo4j.py` — GraphRAG 검색 엔진 |
| HybridGraphRAG 클래스 | ES + Neo4j + PG 통합 쿼리 클래스 |
| NER + 관계 추출 파이프라인 | PDF → 엔티티/관계 → Neo4j 저장 |
| PostgreSQL SSOT 스키마 | 수치 데이터 테이블 + 동기화 로그 |
| 데이터 동기화 파이프라인 | PG → ES + Neo4j 일관성 유지 |
| API 인증 미들웨어 | JWT/API Key 기반 |
| Redis 세션 저장소 | 분산 세션 관리 |
| 통합 Docker Compose | ES + Neo4j + PG + FastAPI 일괄 실행 |

---

## 6. 전환 로드맵

### Phase 1 — ES 단일화 + 기반 안정화

- OpenSearch 관련 코드 전체 제거
- PGVector → 순수 SSOT + 수치 데이터 역할로 스키마 재설계
- Health Check에서 LLM 쿼리 제거
- `lifespan` context manager 마이그레이션
- 패키지 구조 (`src/`) 도입

### Phase 2 — PostgreSQL SSOT 구성

- 수치 데이터 스키마 마이그레이션 (크기, 밝기, 색온도, 소비전력)
- ES 동기화 로그 테이블 구성
- PostgreSQL → Elasticsearch 데이터 동기화 파이프라인 구현

### Phase 3 — Neo4j GraphRAG 통합

- Neo4j Docker Compose 추가 (ES Compose와 통합)
- `vector_store_neo4j.py` 구현
- PDF 파이프라인에 NER + 관계 추출 단계 추가
- `HybridGraphRAG` 클래스 구현 (ES + Neo4j + PG 통합)

### Phase 4 — 운영화

- API 인증/인가 (JWT 기반)
- Redis 세션 저장소 도입
- 유닛/통합 테스트 추가
- CI/CD 파이프라인

---

## 7. 결론 및 권고사항

### 7.1 레거시 시스템 평가

| 영역 | 점수 | 평가 |
|------|:----:|------|
| 아키텍처 설계 | 7/10 | Strategy + Factory 패턴 적용, 다중 DB 추상화 양호 |
| 코드 품질 | 5/10 | PGVectorRAG 초기화 버그, deprecated API, 플랫 구조 |
| 보안 | 3/10 | 인증/인가 없음, API 키 노출 위험 |
| 성능/확장성 | 5/10 | 인메모리 세션, 수평 확장 불가 |
| 테스트 | 4/10 | 벤치마크만 존재, 유닛/통합 테스트 없음 |
| 인프라 | 7/10 | DB별 Docker Compose 분리, 한국어 분석기 포함 |
| 문서화 | 6/10 | Jupyter 가이드 존재, 인라인 문서 부족 |
| **종합** | **5.3/10** | PoC 실험용으로 양호, 신규 시스템 설계 기반으로 활용 |

### 7.2 SearcheRAGWithGraphRAG 핵심 결정사항

| 결정 항목 | 결정 내용 |
|----------|----------|
| Vector Store | **Elasticsearch 단일 사용** (OpenSearch, PGVector 벡터 역할 제거) |
| Graph Store | **Neo4j 신규 도입** (엔티티/관계 그래프 + Cypher 쿼리) |
| SSOT 데이터 | **PostgreSQL** (마스터 데이터 + 수치 데이터: 크기, 밝기, 색온도, 소비전력) |
| 임베딩 | 현재 OpenAI → 장기적으로 BGE-M3 등 로컬 모델 검토 |
| LLM | 현재 GPT-4o-mini → DeepSeek V3 등 비용 효율 모델 검토 |
| 인증 | 처음부터 JWT 기반 인증 설계 |
| 세션 | Redis 분산 세션 저장소 |

### 7.3 즉시 착수 권고

```
P0: ES 단일화 — OpenSearch 코드 제거, ES 전용 구조로 단순화
P0: PG 스키마 재설계 — 벡터 제거, SSOT + 수치 데이터 테이블 설계
P1: Neo4j Docker 환경 구성 + 프로토타이핑
P1: HybridGraphRAG 클래스 설계 (ES + Neo4j + PG 통합)
P2: 통합 Docker Compose 작성 (ES + Neo4j + PG + FastAPI)
```

---

> **분석 대상**: RAGChatbotServer (Legacy)  
> **작성일**: 2026-05-14  
> **다음 검토 예정**: Phase 2 완료 후 (PostgreSQL SSOT 구성 완료 시점)
