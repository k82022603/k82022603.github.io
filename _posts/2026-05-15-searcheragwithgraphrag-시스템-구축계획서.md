---
title: "SearcheRAGWithGraphRAG 시스템 구축계획서"
date: 2026-05-15 21:30:00 +0900
categories: [AI,  RAG & Knowledge Retrieval]
mermaid: [True]
tags: [AI,  RAG,  Elasticsearch,  GraphRAG,  Neo4j,  hybrid-rag,  MCP,  RRF,  hybrid-rag-knowledge-ops,  DeepSeek,  FastMCP,  PostgreSQL,  Ontology,  Cypher,  SSOT,  sparse-vector,  Claude.write]
---


> **문서 버전**: v0.5
> **작성일**: 2026-05-15
> **변경 이력**:
> - v0.1 (2026-05-14): 초안
> - v0.2 (2026-05-14): Sparse Vector 검토, LLM 비교, 도메인 확정, 수치 추출, 파서 다양화, 하드웨어 제약
> - v0.3 (2026-05-14): Neo4j 중심 아키텍처, ETL 병목 분석, 실시간 ETL, ES Sparse 심화, 도메인 MCP, API 전용 정책
> - v0.4 (2026-05-14): 벡터 저장소 역할 재정의(Dense=ES 전담, Neo4j=Graph 전담), RRF 심화 설계, MCP 역할 명확화(DeepSeek V4 Pro + LangChain MCP Adapter 오케스트레이터), 기술 스택 버전 명세, IT SI/SM 온톨로지, 기술결정 로그(부록 D), 미결 사항 M-1~M-7 전면 교체(HRKP hybrid-rag-knowledge-ops 실 구현값 반영), H-5 임베딩 전략 BGE-M3 로컬 단독 확정
> - v0.5 (2026-05-15): **[수정] v0.4 내부 모순 일괄 정리** — (1) 1.2절 요약표 MCP 행을 "DeepSeek + LangChain MCP Adapter"로 정정, (2) 4.2 아키텍처 다이어그램에서 "Claude (MCP)" 클라이언트 제거 및 Reranker "도입 확정" 반영, (3) Jina v3 API 잔존 표기 6곳 일괄 제거(BGE-M3 로컬 단독으로 통일 — H-5와 정합), (4) DeepSeek 모델명 "V4 Pro"로 통일(V3 표기 일괄 정정 — H-2와 정합), (5) 6장 RRF 가중치/후보 수를 HRKP 실 구현값(Graph weight=0.8, graph_search_top_k=3)으로 정정(M-6/M-7과 정합), (6) 부록 D TD-04 v0.4 변경 반영하여 재작성, TD-03 모델명 정정, TD-06 신설(MCP 오케스트레이터 LLM 결정), (7) Step 2 로드맵 "DeepSeek V3/Claude 연동" 정정, (8) M-1 SemanticChunker · M-4 Gleaning 2-pass 본문 반영, (9) 9.4 프로젝트 구조 `api_embedding.py` 주석을 H-5와 일치하도록 수정
>
> **핵심 참고 자료**:
> - [RAG 기술 아키텍처 세미나 (7) — GraphRAG와 Neo4j로 만드는 지능형 지식 검색](https://k82022603.github.io/posts/rag-%EA%B8%B0%EC%88%A0-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%84%B8%EB%AF%B8%EB%82%98-(7)-graphrag%EC%99%80-neo4j%EB%A1%9C-%EB%A7%8C%EB%93%9C%EB%8A%94-%EC%A7%80%EB%8A%A5%ED%98%95-%EC%A7%80%EC%8B%9D-%EA%B2%80%EC%83%89/) ← **아키텍처 정합 기준**
> - RAGChatbotServer 시스템 분석 ([`docs/analysis/RAGChatbotServer-system-analysis.md`](https://k82022603.github.io/posts/ragchatbotserver-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EB%B6%84%EC%84%9D-%EB%B3%B4%EA%B3%A0%EC%84%9C/))
> - [HRKP(hybrid-rag-knowledge-ops)](https://github.com/k82022603/hybrid-rag-knowledge-ops) 실 구현 — 미결 사항 M-1~M-7 결정 근거
> - 샘플 PDF: [`ZMP_55851_XBO_10000_W_HS_OFR.pdf`](https://www.prolites.com/wp-content/uploads/2019/07/ZMP_55851_XBO_10000_W_HS_OFR.pdf) (OSRAM XBO 10000W)

---

## 목차

1. [프로젝트 개요 및 목적](#1-프로젝트-개요-및-목적)
2. [도메인 정의 및 분석](#2-도메인-정의-및-분석)
3. [하드웨어 제약 및 운영 전략](#3-하드웨어-제약-및-운영-전략)
4. [목표 아키텍처 (세미나(7) 정합 확정)](#4-목표-아키텍처-세미나7-정합-확정)
5. [Triple-Store 설계 (역할 확정)](#5-triple-store-설계-역할-확정)
6. [RRF 심화 설계 — Hybrid RAG의 핵심](#6-rrf-심화-설계--hybrid-rag의-핵심)
7. [ES Sparse Vector 기술 검토](#7-es-sparse-vector-기술-검토)
8. [ETL 파이프라인 설계](#8-etl-파이프라인-설계)
9. [도메인 MCP 아키텍처 (DeepSeek + LangChain MCP 오케스트레이터)](#9-도메인-mcp-아키텍처-deepseek--langchain-mcp-오케스트레이터)
10. [LLM 선택 전략 (API 전용 정책)](#10-llm-선택-전략-api-전용-정책)
11. [PostgreSQL 스키마 설계](#11-postgresql-스키마-설계)
12. [온톨로지 설계](#12-온톨로지-설계)
13. [레거시 자산 계승 전략](#13-레거시-자산-계승-전략)
14. [기술 스택 확정](#14-기술-스택-확정)
15. [단계별 구현 로드맵](#15-단계별-구현-로드맵)
16. [미결 사항](#16-미결-사항)
17. [부록](#17-부록)

---

## 1. 프로젝트 개요 및 목적

### 1.1 핵심 목적

**목적 1 — GraphRAG 성능 기여도 측정**
> "Neo4j 기반 GraphRAG가 기존 BM25 + Dense Vector 검색 대비 RAGAS 지표를 얼마나 향상시키는가?"

GraphRAG 없는 Baseline(Dense + BM25)과 GraphRAG 추가 후를 정량 비교한다. Sparse Vector는 별도 실험 변수로 분리하여 기여도를 독립 측정한다.

**목적 2 — 수치 데이터 한계 극복**
> "기존 RAG가 답하지 못하는 수치 기반 질의(범위, 정렬, 집계)를 PostgreSQL SQL로 처리한다."

### 1.2 v0.4 → v0.5 핵심 변경 요약

| 항목 | v0.3 (수정 전) | v0.4 / v0.5 (수정 후) | 수정 근거 |
|------|--------------|---------------------|---------|
| Dense Vector 저장소 | Neo4j 벡터 인덱스 (중심) | **ES HNSW 단독** | 세미나(7) 아키텍처 정합 |
| Neo4j 역할 | Semantic Hub (벡터 포함) | **Knowledge Graph 전담** | 세미나(7) 역할 분리 |
| 벡터 중복 저장 | ES + Neo4j 양쪽 언급 | **ES 단독 (중복 없음)** | 불필요한 동기화 제거 |
| RRF 설계 | 언급만, 미구현 | **완전한 RRF 설계 신설** | RAGChatbotServer 분석 |
| MCP 오케스트레이터 | 모호 (Claude/DeepSeek 혼재) | **DeepSeek V4 Pro + LangChain MCP Adapter** | 단일 LLM로 단순화 |
| 임베딩 전략 | BGE-M3 + Jina v3 이중화 | **BGE-M3 로컬 단독** | HRKP 실 구현 검증 (H-5) |
| RRF 가중치 | 임의값 (Graph=1.5 등) | **HRKP 실 운용값** (Graph=0.8, top_k=3) | HRKP 튜닝 결과 (M-6, M-7) |

> **v0.5의 성격**: v0.5는 신규 의사결정이 없는 **정합성 정리 릴리스**다. v0.4에서 결정된 사항이 본문 곳곳에 일관되게 반영되지 않은 부분을 일괄 정리했다.

---

## 2. 도메인 정의 및 분석

### 2.1 1차 도메인: 조명 제품 카탈로그 (확정)

OSRAM XBO 10000W/HS OFR 샘플 PDF 분석 결과, **산업/전문 조명 제품 카탈로그**로 확정.

**도메인별 질의 유형과 처리 주체**:

| 질의 예시 | 처리 주체 |
|---------|---------|
| "광속 300,000lm 이상, 전력 5kW 미만 제품" | PostgreSQL (수치 범위 쿼리) |
| "XBO 시리즈 중 영화 조명 용도 제품들" | Neo4j Cypher (그래프 탐색) |
| "고휘도 크세논 램프와 유사한 제품" | ES Dense Vector (의미 검색) |
| "XBO 10000W OFR 제품 코드 매칭" | ES BM25 (정확 키워드) |
| "XBO와 유사하면서 대체 가능한 제품" | RRF (ES + Neo4j 통합) |

### 2.2 2차 도메인: IT SI/SM 산출물

서버 스펙시트, 네트워크 장비 사양, 운영 매뉴얼 등 수치 데이터 풍부. 온톨로지는 12절 참조.

---

## 3. 하드웨어 제약 및 운영 전략

### 3.1 개발 환경

| 항목 | 사양 |
|------|------|
| OS | Windows (노트북) |
| CPU | CPU Only (No GPU) |
| RAM | **16GB** |

### 3.2 메모리 예산

```
상시 서비스 동작:
  Elasticsearch:  1.5 GB (heap 512m 제한)
  Neo4j:          1.0 GB (heap_max 512m + pagecache 256m + 기타)
  PostgreSQL:     0.3 GB
  Redis:          0.1 GB
  FastAPI:        0.3 GB
  BGE-M3 모델:    1.2 GB
  합계:           ~4.4 GB ✅ 여유 11.6GB

개발/테스트 로컬 LLM 추가:
  Qwen3 8B Q4:   +5.0 GB → 총 ~9.4 GB ✅ 안전
```

### 3.3 임베딩 운용 전략 (BGE-M3 로컬 단독)

**로컬 BGE-M3 CPU 추론 속도**:

| 모드 | 처리량 | 10,000 청크 기준 | 비고 |
|------|--------|-----------------|------|
| Dense only (배치 4) | ~286 chunks/min | ~35분 | 실시간 처리 가능 |
| Dense + Sparse (배치 4) | ~179 chunks/min | ~56분 | Step 3 이후 |
| 배치 처리 (HRKP 실측) | **7.5 texts/sec** | **약 22분** | Redis TTL=7일 캐시 적용 |

**운용 정책 (H-5 확정)**:
- 모든 임베딩은 **로컬 BGE-M3(BAAI/bge-m3, 1024차원)** 단독 사용
- 중복 임베딩 방지를 위한 **Redis 캐시(TTL=7일)** 적용 — HRKP 실 운용 검증
- 대량 인덱싱은 **Redis Queue + RQ Worker**로 야간/비동기 배치 처리
- Sparse 임베딩(Step 3)도 동일 BGE-M3 모델에서 동시 생성 (return_sparse=True)
- **Jina/OpenAI 등 외부 임베딩 API 미사용** — 추가 API 비용 없이 완전 로컬 운용

> **v0.4 변경 사항**: v0.3까지의 "Jina v3 API 대량 배치 처리" 정책은 폐기되었다. HRKP 실 운용 결과 BGE-M3 단독으로 운영 요건을 충족함이 검증되었기 때문이다(부록 D TD-05 참조).

**중국 모델 API 전용 정책**: DeepSeek, Qwen, GLM 등 중국 모델은 **모두 API로만 사용**. 로컬 실행 시도 금지 (메모리/속도 한계).

---

## 4. 목표 아키텍처 (세미나(7) 정합 확정)

### 4.1 핵심 설계 원칙 — 세미나(7) 정합

> **세미나(7)의 명시적 아키텍처**:
> - ES: Dense Vector + Sparse Vector + BM25 (Nori) — **모든 벡터 및 텍스트 검색**
> - Neo4j: Knowledge Graph (엔터티, 관계, 멀티홉 Cypher) — **그래프 탐색 전담**
> - PostgreSQL: SSOT + 수치 데이터
> - RRF: ES 결과 + Neo4j 결과 통합

**v0.3의 오류 수정**: v0.3에서 "Neo4j를 Semantic Hub로, Dense Vector를 Neo4j에 저장"으로 설계했으나, 세미나(7) 아키텍처 및 실제 운영 고려 사항을 검토한 결과 **Dense Vector는 ES에 통합 관리하는 것이 올바름**. 이유:

| 검토 항목 | ES Dense | Neo4j Vector Index |
|---------|---------|-------------------|
| 세미나(7) 설계 | ✅ ES에 Dense | ❌ Neo4j에 Dense 없음 |
| BM25 + Dense 통합 쿼리 | ✅ 하나의 ES 쿼리 | ❌ 별도 쿼리 필요 |
| Sparse + Dense 통합 | ✅ ES sparse_vector | ❌ 미지원 |
| 한국어 Nori | ✅ ES 내장 | ❌ 미지원 |
| Graph + Vector 단일 Cypher | ❌ 불가 | ✅ 가능 (단, 쿼리 복잡) |
| 운영 복잡도 (중복 동기화) | ✅ 단순 (ES만) | ❌ 복잡 (양쪽 관리) |
| 검색 성능 (순수 ANN) | ✅ 우수 | △ 중간 |

**결론**: Dense Vector는 ES 단독 저장. Neo4j는 Knowledge Graph(엔터티/관계/Cypher) 전담. 두 저장소에 동일 벡터를 중복 저장하지 않는다.

### 4.2 전체 시스템 구성 (세미나(7) 정합)

```
┌─────────────────────────────────────────────────────────────────────┐
│                  SearcheRAGWithGraphRAG 목표 아키텍처                 │
│                  (세미나(7) 아키텍처 정합)                             │
├─────────────────────────────────────────────────────────────────────┤
│  [Client Layer]                                                     │
│    Web UI / REST API Client                                         │
│              ↓ (JWT 인증)                                           │
│                                                                     │
│  [Application Layer — FastAPI :8000]                                │
│    ┌────────────────────────────────────────────────────────────┐  │
│    │  LangChain Agent (DeepSeek V4 Pro + MCP Adapter)           │  │
│    │   └─ 복합 도구 조합이 필요한 질의에 대해 MCP 도구 오케스트레이션   │  │
│    │                                                            │  │
│    │               4-Way Hybrid RAG Engine                      │  │
│    │                                                            │  │
│    │  ① ES BM25 (Nori 한국어)  → Elasticsearch                  │  │
│    │  ② ES Dense Vector (BGE-M3, 1024d, HNSW) → Elasticsearch  │  │
│    │  ③ ES Sparse Vector (BGE-M3) → Elasticsearch  [Step 3]    │  │
│    │  ④ Neo4j Graph (Cypher 멀티홉) → Neo4j                     │  │
│    │                           ↓                               │  │
│    │            RRF (Reciprocal Rank Fusion)                   │  │
│    │            각 소스 독립 실행 → 순위 기반 통합 (상위 50건)   │  │
│    │                           ↓                               │  │
│    │            BGE-Reranker-v2-m3 (Step 2부터 도입 확정)        │  │
│    │            50건 → top_k 재정렬                              │  │
│    │                           ↓                               │  │
│    │            LLM 응답 생성 (DeepSeek V4 Pro API)              │  │
│    └────────────────────────────────────────────────────────────┘  │
│    └── Numeric Query Engine → PostgreSQL (수치 범위/집계)            │
│                                                                     │
│  [Triple-Store]                                                     │
│    Elasticsearch :9200  — Dense + BM25 + Sparse (모든 벡터/텍스트)  │
│    Neo4j :7474/:7687    — Knowledge Graph (엔터티/관계)              │
│    PostgreSQL :5432     — SSOT + 수치 데이터                        │
│    Redis :6379          — 세션 + ETL 비동기 큐 + 임베딩 캐시          │
└─────────────────────────────────────────────────────────────────────┘
```

> **v0.5 정정 사항**:
> - Client Layer에서 "Claude (MCP)" 항목 제거 — MCP 오케스트레이터는 DeepSeek V4 Pro(LangChain Agent 내부)로 결정되었으며, Claude는 사용자 단의 외부 클라이언트로 호출되지 않는다.
> - Reranker를 "선택적, Step 2+"에서 "**도입 확정, Step 2부터**"로 변경(M-2 결정 반영).

### 4.3 성능 측정 실험 설계

| 단계 | 구성 | 측정 목적 |
|------|------|---------|
| **Baseline** | ES Dense + BM25 | 그래프 없는 기준선 |
| **+ GraphRAG** | Dense + BM25 + **Neo4j Graph** | **GraphRAG 기여도 (핵심)** |
| **+ Sparse** | Dense + BM25 + Graph + **Sparse** | Sparse 추가 효과 |

평가: RAGAS (Faithfulness, Answer Relevancy, Context Recall, Context Precision)

---

## 5. Triple-Store 설계 (역할 확정)

```
Elasticsearch (검색 엔진 — 벡터/텍스트 전담)
  ├── ① Dense Vector 인덱스 (BGE-M3, 1024차원, HNSW) ← 세미나(7) 확정
  ├── ② BM25 전문 검색 인덱스 (Nori 한국어 형태소)
  └── ③ Sparse Vector 인덱스 (BGE-M3 lexical_weights) [Step 3]

Neo4j (Knowledge Graph 전담 — 벡터 저장 없음)
  ├── 엔터티 노드 (Product, Category, Application, Vendor…)
  ├── 관계 엣지 (BELONGS_TO, SUITABLE_FOR, SIMILAR_TO…)
  └── 렉시컬 그래프 (Document → Chunk, Chunk -MENTIONS-> Entity)

PostgreSQL (SSOT + 수치 전담)
  ├── 문서 처리 상태 추적 (ETL 파이프라인 기준점)
  ├── 제품 마스터 데이터
  └── 수치 속성 (전력/광속/크기 — 범위/집계/정렬)
```

---

## 6. RRF 심화 설계 — Hybrid RAG의 핵심

### 6.1 RAGChatbotServer 분석 — 기존 방식의 문제점

RAGChatbotServer를 분석한 결과, **RRF가 전혀 구현되어 있지 않다**. 대신 단순 가중 점수 평균(Weighted Score Fusion)을 사용한다.

```python
# RAGChatbotServer hybrid_search() — 실제 구현 (vector_store_elasticsearch.py L677)
final_score = (alpha * scores["vector_score"]) + ((1 - alpha) * scores["bm25_score"])
# alpha = 0.6 고정 (벡터 60%, BM25 40%)
```

**이 방식의 문제점**:

| 문제 | 설명 |
|------|------|
| 스코어 스케일 불일치 | 벡터 유사도(0~1)와 BM25 스코어(0~N) 단위가 달라 단순 합산이 의미 없음 |
| alpha 튜닝 필요 | 도메인/질의 유형마다 최적 alpha가 다름, 고정값 0.6은 임의적 |
| Neo4j 통합 불가 | 가중 합산 방식으로는 3번째 소스(그래프) 추가 어려움 |
| RRF가 아님 | RAGChatbotServer의 hybrid는 ES 내부 2개 소스만 결합, 진정한 Hybrid RAG가 아님 |

### 6.2 RRF 알고리즘 원리

RRF(Reciprocal Rank Fusion, Cormack et al. 2009)는 서로 다른 검색 방식의 결과를 **순위(rank)** 기반으로 통합하는 알고리즘이다. 점수 체계가 달라도 편향 없이 결합할 수 있다는 것이 핵심이다.

```
RRF score(d) = Σ_i  weight_i / (k + rank_i(d))

- d: 문서(청크)
- i: 검색 소스 (BM25, Dense, Graph, Sparse…)
- k: 상수 (기본 60 — 상위 결과에 집중, 낮을수록 top-1 가중 증가)
- rank_i(d): 소스 i에서 d의 순위 (1-based)
- weight_i: 소스 i의 가중치
```

**RRF 작동 예시** (k=60, 단순화를 위해 weight 모두 1.0):

| 문서 | BM25 순위 | Dense 순위 | Graph 순위 | RRF 점수 |
|------|----------|----------|----------|---------|
| 문서 A | 1위 | 3위 | 2위 | 1/61 + 1/63 + 1/62 = **0.0484** |
| 문서 B | 5위 | 1위 | - | 1/65 + 1/61 + 0 = **0.0320** |
| 문서 C | - | 2위 | 1위 | 0 + 1/62 + 1/61 = **0.0326** |

→ 여러 소스에서 공통으로 높은 순위를 차지한 문서 A가 최종 1위.

### 6.3 우리 시스템 RRF 구현 설계

```python
from collections import defaultdict
from typing import List, Tuple, Optional
from langchain.schema import Document

def reciprocal_rank_fusion(
    result_lists: List[List[Tuple[str, Document]]],
    k: int = 60,
    weights: Optional[List[float]] = None
) -> List[Tuple[str, float, Document]]:
    """
    n-Way RRF: 여러 검색 소스의 순위 리스트를 RRF로 통합

    Args:
        result_lists: [[(doc_id, Document), ...], ...] 형태의 소스별 순위 리스트
                      각 리스트 내 순서가 곧 해당 소스의 순위(rank)
        k: RRF 상수 (기본 60, Cormack et al. 표준값)
        weights: 소스별 가중치 (None이면 모두 1.0)

    Returns:
        [(doc_id, rrf_score, Document), ...] RRF 점수 내림차순 정렬

    Note:
        동일 doc_id가 여러 소스에서 등장할 경우, 첫 번째로 등장한 소스의
        Document 객체를 유지한다(메타데이터 보존). 점수는 모든 소스의 기여를 누적한다.
    """
    if weights is None:
        weights = [1.0] * len(result_lists)

    scores = defaultdict(lambda: {"score": 0.0, "doc": None})

    for result_list, weight in zip(result_lists, weights):
        for rank, (doc_id, doc) in enumerate(result_list, start=1):
            scores[doc_id]["score"] += weight / (k + rank)
            if scores[doc_id]["doc"] is None:
                scores[doc_id]["doc"] = doc

    return sorted(
        [(doc_id, info["score"], info["doc"])
         for doc_id, info in scores.items()],
        key=lambda x: x[1],
        reverse=True
    )


class HybridGraphRAG:
    """4-Way Hybrid RAG 엔진 (RRF + Reranker 통합)"""

    # HRKP 실 운용값 (M-6, M-7 결정 사항)
    DEFAULT_WEIGHTS = {
        "bm25":   1.0,   # 키워드 검색 기본 가중치
        "dense":  1.0,   # Dense Vector 기본 가중치
        "graph":  0.8,   # 그래프 결과는 약간 보조 가중 (1.5 → 0.8 조정)
        "sparse": 0.7,   # Sparse는 Dense 보조 역할
    }

    def __init__(self, es_manager, neo4j_manager, pg_manager,
                 llm_client, embedding_client, reranker,
                 k_rrf: int = 60, top_k: int = 10,
                 graph_search_top_k: int = 3,   # M-7: Graph 소스 후보 수 제한
                 rrf_pool_size: int = 50):       # Reranker 입력 풀 크기 (M-2)
        self.es = es_manager
        self.neo4j = neo4j_manager
        self.pg = pg_manager
        self.llm = llm_client
        self.embed = embedding_client
        self.reranker = reranker
        self.k_rrf = k_rrf
        self.top_k = top_k
        self.graph_search_top_k = graph_search_top_k
        self.rrf_pool_size = rrf_pool_size

    async def search(self, query: str, mode: str = "full") -> List[Document]:
        """
        mode:
          - "baseline": ES BM25 + Dense (Step 1)
          - "graph":    + Neo4j Graph (Step 2)
          - "full":     + Sparse (Step 3)
        """
        query_vector = await self.embed.embed_query(query)

        # ① ES BM25 검색 — 후보 top_k×2 = 20건
        bm25_results = await self.es.bm25_search(query, k=self.top_k * 2)

        # ② ES Dense Vector 검색 — 후보 top_k×2 = 20건
        dense_results = await self.es.dense_search(query_vector, k=self.top_k * 2)

        result_lists = [bm25_results, dense_results]
        weights = [self.DEFAULT_WEIGHTS["bm25"], self.DEFAULT_WEIGHTS["dense"]]

        # ③ Neo4j Graph 검색 — Graph 후보 수는 graph_search_top_k(3)로 고정 (M-7)
        if mode in ("graph", "full"):
            graph_results = await self.neo4j.graph_search(
                query, k=self.graph_search_top_k
            )
            result_lists.append(graph_results)
            weights.append(self.DEFAULT_WEIGHTS["graph"])

        # ④ ES Sparse Vector 검색 — 후보 top_k×2 = 20건 (mode=full)
        if mode == "full":
            sparse_query = await self.embed.generate_sparse(query)
            sparse_results = await self.es.sparse_search(
                sparse_query, k=self.top_k * 2
            )
            result_lists.append(sparse_results)
            weights.append(self.DEFAULT_WEIGHTS["sparse"])

        # RRF 통합 → 상위 rrf_pool_size(50)건을 Reranker 입력으로
        fused = reciprocal_rank_fusion(result_lists, k=self.k_rrf, weights=weights)
        rrf_pool = [doc for _, _, doc in fused[:self.rrf_pool_size]]

        # Reranker 적용 (BGE-Reranker-v2-m3, M-2)
        reranked = await self.reranker.rerank(query, rrf_pool, top_k=self.top_k)
        return reranked
```

### 6.4 Neo4j 그래프 검색 결과의 RRF 통합

Neo4j는 점수 기반이 아닌 **관계 탐색** 기반으로 결과를 반환한다. 이를 RRF의 `(doc_id, Document)` 형식으로 변환한다. Graph 후보 수는 **3건으로 고정**(M-7)하여 Graph 결과가 RRF 결과를 과도하게 점유하는 현상을 방지한다.

```python
async def graph_search(self, query: str, k: int = 3) -> List[Tuple[str, Document]]:
    """Neo4j 그래프 탐색 → RRF 입력 형식 변환

    Note:
        k(graph_search_top_k)는 기본 3으로 제한된다.
        HRKP 튜닝(34_graph_search_rrf_tuning.md, 2026-03-09 ADR)에서
        Graph 후보 수를 늘리면 RRF top-10 중 Graph 결과가 8건을 점유하는
        편향이 관찰됐기 때문이다. weight 조정 대신 후보 수 제한으로 해결.
    """
    cypher = """
    CALL db.index.fulltext.queryNodes("entity_search", $query)
    YIELD node AS entity, score
    MATCH (chunk:Chunk)-[:MENTIONS]->(entity)
    WITH chunk, entity, score
    ORDER BY score DESC
    LIMIT $k
    RETURN chunk.id AS doc_id, chunk.text AS text, chunk.metadata AS meta
    """
    records = await self.neo4j_session.run(cypher, query=query, k=k)

    results = []
    for record in records:
        doc = Document(
            page_content=record["text"],
            metadata={**record["meta"], "source": "neo4j_graph"}
        )
        results.append((record["doc_id"], doc))

    return results  # 이미 score 기준 정렬 → index가 RRF의 rank
```

### 6.5 RRF vs RAGChatbotServer 방식 비교

| 항목 | RAGChatbotServer (가중 평균) | 우리 시스템 (RRF) |
|------|--------------------------|----------------|
| 알고리즘 | `alpha * v_score + (1-alpha) * bm25_score` | `Σ weight / (k + rank)` |
| 점수 정규화 | ❌ 필요 (단위 불일치) | ✅ 불필요 (순위 기반) |
| 소스 수 | ES 내부 2개 | ES + Neo4j 3~4개 |
| 파라미터 | alpha (튜닝 민감) | k (robust, 60 표준값) |
| Neo4j 통합 | ❌ 불가 | ✅ 가능 |
| 설계 견고성 | 낮음 (alpha 임의적) | 높음 (수학적 근거) |
| 레거시 계승 | 부분 (가중치 개념) | 구조적으로 상위 호환 |

### 6.6 RRF 파라미터 (HRKP 실 운용값 — M-6, M-7)

| 파라미터 | 값 | 결정 근거 |
|---------|-----|---------|
| `k_rrf` | **60** | Cormack et al. 2009 표준값, HRKP 검증 완료 |
| BM25 weight | **1.0** | 기본값 |
| Dense weight | **1.0** | 기본값 |
| Graph weight | **0.8** | HRKP 튜닝 결과. Graph 결과 편향 방지를 위해 약간 보조 가중 |
| Sparse weight | **0.7** | Dense 대비 보조 역할 |
| BM25/Dense/Sparse 후보 수 | **top_k × 2 = 20건** | 충분한 후보 확보 |
| Graph 후보 수 (`graph_search_top_k`) | **3** | Graph 편향 방지 (weight 조정보다 후보 수 제한이 효과적) |
| RRF → Reranker 풀 크기 | **50** | 상위 50건을 Reranker에 입력 |
| 최종 top_k | **10** | RAGAS 실험으로 추가 최적화 |

> **이전 v0.4 문서에 표기되었던 "Graph weight=1.5, top_k×2 후보" 설정은 HRKP에서 실제로 운용한 결과 Graph 결과 독식 문제를 일으켰다. v0.5는 HRKP 결정값을 본문 코드와 표 양쪽에 반영한다.**

---

## 7. ES Sparse Vector 기술 검토

### 7.1 ELSER vs BGE-M3 Sparse

| 항목 | ELSER v2 | BGE-M3 Sparse |
|------|---------|--------------|
| 한국어 지원 | ❌ 영어 전용 | ✅ 100+ 언어 |
| ES 통합 | 네이티브 (ML node) | 외부 처리 → ES 적재 |
| 한국어 품질 | ❌ 매우 낮음 | ✅ 높음 |
| CPU 추론 | ES 내부 (GPU 권장) | Python CPU 가능 |
| 라이선스 | ES Basic 8.9+ 사용 가능 | Apache 2.0 |

**결론**: 한국어 환경에서 ELSER는 실용 불가. BGE-M3 Sparse만이 선택지.

### 7.2 BGE-M3 Sparse → ES 변환 구현

```python
from FlagEmbedding import BGEM3FlagModel
from transformers import AutoTokenizer

model = BGEM3FlagModel('BAAI/bge-m3', use_fp16=False)  # CPU: fp16 비활성화
tokenizer = AutoTokenizer.from_pretrained('BAAI/bge-m3')

def generate_sparse_vector(text: str, threshold: float = 0.01) -> dict:
    """
    BGE-M3 lexical_weights → ES sparse_vector 형식 변환
    핵심: token_id(int) → token_string 변환 필수
    """
    output = model.encode([text], return_dense=False, return_sparse=True)
    lexical_weights = output['lexical_weights'][0]  # {token_id: weight}

    token_ids = list(lexical_weights.keys())
    token_strings = tokenizer.convert_ids_to_tokens(token_ids)

    return {
        token_str: float(weight)
        for token_str, (_, weight) in zip(
            token_strings, lexical_weights.items()
        )
        if weight > threshold
        and token_str not in ['[CLS]', '[SEP]', '[PAD]', '[UNK]']
    }
```

### 7.3 최종 전략

```
Step 1: Dense + BM25 (ES) → RAGAS Baseline
Step 2: + Neo4j Graph + Reranker → GraphRAG 기여도 측정
Step 3: + BGE-M3 Sparse         → Sparse 추가 효과 측정
  - Sparse는 배치 야간 처리 (CPU 오버헤드)
  - ELSER 배제 (한국어 미지원)
```

---

## 8. ETL 파이프라인 설계

### 8.1 전체 ETL 흐름 (실시간 + 배치)

```
문서 수신 (REST API 업로드 or 파일 감시)
  ↓
PostgreSQL SSOT 등록 (status: 'pending')
  ↓
Redis Queue 비동기 적재 (RQ Worker)
  ↓
청킹: SemanticChunker (BGE-M3 임베딩 기반, target=1024 tokens, overlap=128 tokens) [M-1]
  - 임베딩 유사도로 의미 경계 감지
  - 허용 범위 256~2048 tokens (벗어나면 재분할)
  ↓ 병렬 처리
  ├── Phase 2A: Dense 임베딩 → ES HNSW 적재
  │     BGE-M3 로컬 CPU 단독 운용
  │     Redis 캐시(TTL=7일) — 동일 텍스트 재임베딩 방지
  │     → PostgreSQL es_sync_log status 'synced'
  │
  ├── Phase 2B: 엔터티 추출 → Neo4j 적재 [M-4]
  │     DeepSeek V4 Pro API → 엔터티/관계 JSON
  │     Gleaning 2-pass 적용 (1-pass 대비 Recall +33%)
  │     신뢰도 임계값 ≥ 0.7로 필터링
  │     엔터티 타입(조명): Product, Series, Category, Application,
  │                       Technology, Vendor, Certification
  │     엔터티 타입(IT SI/SM): Person, Organization, Technology, Project,
  │                          Concept, Date, Location 등
  │     Neo4j MERGE (중복 방지) → Chunk -MENTIONS-> Entity 연결
  │     → PostgreSQL neo4j_sync_log status 'synced'
  │
  └── Phase 3: 수치 추출 → PostgreSQL
        pdfplumber → 표 파싱
        ProductSpecExtractor 규칙 기반 매핑
        미매핑 → spec_json JSONB
        → products + product_specs 적재

  [Step 3 이후, 배치/야간]
  Phase 4: BGE-M3 Sparse 생성 → ES sparse_vector 적재
```

> **v0.5 반영**: M-1(SemanticChunker)과 M-4(Gleaning 2-pass + 신뢰도 임계값) 결정이 본문 파이프라인 흐름에 명시되도록 보강했다.

### 8.2 문서 파서 전략

```python
class PDFParserFactory:
    @staticmethod
    def get_parser(doc_type: str, file_path: str):
        if doc_type == "product_catalog":
            return PdfplumberParser(file_path)   # 표 추출 강점 (기본값)
        elif doc_type == "manual":
            return PyMuPDFParser(file_path)       # 긴 텍스트
        elif doc_type == "hwp":
            return DoclingParser(file_path)       # HWP 전용
        else:
            return PdfplumberParser(file_path)
```

| 파서 | 강점 | 권장 사용 |
|------|------|----------|
| **pdfplumber** | 테이블 추출 최강, 레거시 계승 | 제품 카탈로그 (기본값) |
| PyMuPDF | 빠른 텍스트, 좌표 정보 | 매뉴얼, 긴 텍스트 |
| Unstructured | 다형식 (HTML, DOCX) | 혼합 환경 |
| Docling 2.x | 복잡 레이아웃, HWP | HWP 파일만 |

### 8.3 수치 데이터 추출

```python
class ProductSpecExtractor:
    FIELD_MAP = {
        "nominal wattage":       ("wattage_w",         "float"),
        "nominal voltage":       ("voltage_v",          "float"),
        "nominal luminous flux": ("luminous_flux_lm",  "float"),
        "luminance":             ("luminance_cd_cm2",   "float"),
        "color temperature":     ("color_temp_k",       "int"),
        "length":                ("height_mm",          "float"),
        "diameter":              ("width_mm",           "float"),
        "product weight":        ("weight_g",           "float"),
        "nominal lifetime":      ("lifetime_hr",        "float"),
    }

    def extract(self, tables: list) -> dict:
        typed_fields, json_fields = {}, {}
        for table in tables:
            for row in table:
                if len(row) < 2: continue
                key = str(row[0]).lower().strip()
                val = str(row[1]).strip()
                matched = False
                for pattern, (col, dtype) in self.FIELD_MAP.items():
                    if pattern in key:
                        typed_fields[col] = self._parse_value(val, dtype)
                        matched = True
                        break
                if not matched:
                    json_fields[key] = val   # → JSONB
        return {"typed": typed_fields, "json": json_fields}
```

---

## 9. 도메인 MCP 아키텍처 (DeepSeek + LangChain MCP 오케스트레이터)

### 9.1 MCP 역할 명확화

**MCP(Model Context Protocol)는 Anthropic이 제안한 오픈 스탠다드다. MCP 클라이언트 역할은 모델이 아니라, 모델을 감싸는 런타임/프레임워크가 구현한다.**

> MCP는 Cursor, VS Code, Claude Desktop 등 다양한 에이전트 런타임이 지원하는 개방형 프로토콜이다.
> 이 시스템에서는 `langchain-mcp-adapters`를 MCP 런타임으로 채택한다. DeepSeek V4 Pro API(OpenAI 호환)를 LangChain Agent의 LLM으로 연결하면, DeepSeek이 MCP 서버 도구를 직접 오케스트레이션한다.

| 구성 요소 | MCP와의 관계 |
|---------|------------|
| **DeepSeek V4 Pro** (LLM) | ✅ MCP 오케스트레이터 — LangChain MCP Adapter 런타임 하에서 도구 호출을 판단 |
| **LangChain MCP Adapter** | ✅ MCP 클라이언트 런타임 — MCP 서버와 통신, 도구 결과를 DeepSeek에 전달 |
| **FastAPI 메인 서버** | MCP와 무관 — LangChain Agent를 내부 모듈로 호출 |
| **도메인 MCP 서버** | MCP 서버 — DeepSeek(LangChain 경유)이 호출하는 도구 모음 |

**결론**: 도메인 MCP 서버(osram-product-mcp 등)는 **DeepSeek이 LangChain MCP Adapter를 통해 제품 데이터에 접근하기 위한 인터페이스**다. DeepSeek 하나가 엔터티 추출·MCP 오케스트레이션·최종 응답 생성을 모두 담당하며, 별도 오케스트레이터 LLM이 필요 없어 아키텍처가 단순해진다.

### 9.2 도입 배경 및 가치

OSRAM 같은 특정 제조사 데이터는:
- 고유한 PDF 레이아웃과 필드 명칭 (OSRAM 전용 파싱 로직 필요)
- 반복적 카탈로그 업데이트 시 전용 처리 로직이 유리
- **모든 데이터는 반드시 Triple Store에 동기화** — MCP는 접근 레이어일 뿐

### 9.3 쿼리 처리 흐름

```
사용자 질의
    │
    ▼
FastAPI (api/routers/chat.py)
    │
    ▼
LangChain Agent
├── LLM: DeepSeek V4 Pro (OpenAI 호환 API)
└── Tools: langchain-mcp-adapters → MCP 서버 연결
    │
    ├── [도구 호출 필요 시] osram-product-mcp 서버
    │       ├── search_osram_products()   # ES/Neo4j/PG 하이브리드 검색
    │       ├── find_similar_products()   # Neo4j SIMILAR_TO 그래프 탐색
    │       └── numeric_filter_products() # PostgreSQL 수치 범위 쿼리
    │
    └── [단순 RAG] HybridGraphRAG 직접 호출 (MCP 미경유)
    │
    ▼
DeepSeek V4 Pro 최종 답변 생성
    │
    ▼
사용자 응답
```

> **판단 기준**: MCP 도구 경유 vs. HybridGraphRAG 직접 호출
> - 단일 도메인 단순 질의 → HybridGraphRAG 직접 (빠름)
> - 복합 도구 조합 필요 (수치 필터 + 유사 제품 + 사양 비교 등) → LangChain Agent → MCP

### 9.4 프로젝트 구조

```
SearcheRAGWithGraphRAG/
├── core/                             # 공통 인프라
│   ├── triple_store/
│   │   ├── neo4j_manager.py          # Neo4j Cypher + 노드/엣지 CRUD
│   │   ├── es_manager.py             # ES Dense + BM25 + Sparse
│   │   └── pg_manager.py             # SSOT + 수치 데이터
│   ├── embedding/
│   │   └── local_bge_m3.py           # 로컬 BGE-M3 (Dense + Sparse 단독)
│   ├── chunking/
│   │   └── semantic_chunker.py       # BGE-M3 임베딩 기반 의미 경계 청킹 (M-1)
│   ├── rerank/
│   │   └── bge_reranker.py           # BGE-Reranker-v2-m3 (M-2)
│   ├── rag/
│   │   ├── hybrid_graph_rag.py       # HybridGraphRAG + RRF + Reranker
│   │   └── rrf.py                    # reciprocal_rank_fusion()
│   └── agent/
│       └── mcp_agent.py              # LangChain Agent + MCP Adapter (DeepSeek 오케스트레이터)
│
├── mcp_servers/                      # MCP 서버 (도메인별 도구 모음)
│   ├── osram_product_mcp/
│   │   ├── server.py                 # FastMCP 서버
│   │   ├── extractor.py              # OSRAM PDF 특화 파서
│   │   └── tools/
│   │       ├── ingest_catalog.py     # 카탈로그 수집 → Triple Store
│   │       ├── search_products.py    # RRF 하이브리드 검색
│   │       ├── find_similar.py       # Neo4j SIMILAR_TO
│   │       └── numeric_filter.py     # PostgreSQL 수치 쿼리
│   └── it_server_mcp/                # IT SI/SM 도메인 (2차)
│
└── api/                              # FastAPI
    ├── app.py
    └── routers/
        ├── chat.py                   # RAG 질의응답 (Agent / HybridRAG 분기)
        ├── documents.py              # 문서 관리 + 실시간 ETL
        └── eval.py                   # RAGAS 평가
```

> **v0.5 정정 사항**: `core/embedding/` 하위 `api_embedding.py`(Jina/OpenAI API 어댑터) 제거. BGE-M3 로컬 단독으로 통일(H-5). 신규 `core/chunking/`(M-1), `core/rerank/`(M-2) 디렉터리 명시.

### 9.5 LangChain MCP Agent 예시

```python
# core/agent/mcp_agent.py
# 의존 패키지: langchain-mcp-adapters>=0.2.2, langgraph>=1.0, langchain-openai>=0.3
from langchain_openai import ChatOpenAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent

# DeepSeek V4 Pro — OpenAI 호환 엔드포인트
llm = ChatOpenAI(
    model="deepseek-v4-pro",
    base_url="https://api.deepseek.com/v1",
    api_key="<DEEPSEEK_API_KEY>",
)

# MCP 서버 연결 (stdio 또는 SSE 방식)
async def get_agent():
    client = MultiServerMCPClient({
        "osram-product-mcp": {
            "command": "python",
            "args": ["mcp_servers/osram_product_mcp/server.py"],
            "transport": "stdio",
        }
    })
    tools = await client.get_tools()
    return create_react_agent(llm, tools)
```

### 9.6 FastMCP 서버 예시

```python
# mcp_servers/osram_product_mcp/server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("osram-product-mcp")

async def search_osram_products(
    query: str,
    wattage_min: float = None,
    wattage_max: float = None,
    application: str = None,
    top_k: int = 10
) -> list[dict]:
    """OSRAM 제품 Hybrid 검색 (ES BM25/Dense + Neo4j Graph + PG 수치 필터)"""
    ...

async def numeric_filter_products(
    luminous_flux_min: float = None,
    wattage_max: float = None,
    sort_by: str = "luminous_flux_lm"
) -> list[dict]:
    """PostgreSQL 수치 범위 쿼리 전용 — '광속 300k~600k lm' 등"""
    ...
```

---

## 10. LLM 선택 전략 (API 전용 정책)

### 10.1 API 전용 정책 (확정)

| 모델 계열 | 정책 |
|---------|------|
| 중국 모델 (DeepSeek, Qwen, GLM) | **API 전용 — 로컬 실행 금지** |
| 임베딩 (모든 용도) | **로컬 BGE-M3 단독** (H-5) |
| 개발 테스트 전용 | Qwen3 8B Q4 로컬 (기능 확인만) |

> **v0.5 정정 사항**: 이전 표에 있던 "임베딩 대규모 → Jina v3 API" 행을 제거하고 H-5(BGE-M3 단독) 정책으로 통일.

### 10.2 용도별 LLM

| 용도 | 모델 | 방식 |
|------|------|------|
| 엔터티 추출 (배치 ETL) | DeepSeek V4 Pro API | API (Gleaning 2-pass, M-4) |
| MCP 오케스트레이션 + RAG 응답 생성 | DeepSeek V4 Pro API | API (LangChain Agent 경유) |
| 단순 RAG 응답 생성 | DeepSeek V4 Pro API | API (HybridGraphRAG 직접) |
| 개발/기능 테스트 | Qwen3 8B Q4 (Ollama) | 로컬 (테스트 전용) |

### 10.3 비용 추정

**엔터티 추출 (DeepSeek V4 Pro 기준 단가 — 실제 청구액은 API 가격 정책 변동에 따라 조정)**:
- 100 문서 (500p) Gleaning 2-pass: 1-pass 대비 약 2배 토큰 소비
- 500 문서 (2,500p)도 동일 배율
- 정확한 단가는 deepseek.com 가격표를 따르되, DeepSeek 계열은 입력/출력 모두 동급 API 대비 저렴 수준 유지 가정

**임베딩 비용**:
- **0원 (BGE-M3 로컬 단독, 외부 API 미사용 — H-5)**
- 단, 노트북 CPU 시간 비용은 별도 — Redis 캐시(TTL=7일)로 중복 임베딩 방지

> **v0.5 정정 사항**: v0.4의 "DeepSeek V3" 단가 표기와 "Jina v3 API" 임베딩 비용 항목을 모두 정정. 단가는 V4 Pro로 통일하고, 정확한 수치는 운영 시 실측치로 갱신.

---

## 11. PostgreSQL 스키마 설계

```sql
CREATE TABLE documents (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_name    VARCHAR(500) NOT NULL,
    file_path    TEXT,
    doc_type     VARCHAR(50),       -- 'product_datasheet' | 'manual' | 'si_spec'
    domain       VARCHAR(50),       -- 'lighting' | 'it_server'
    vendor       VARCHAR(100),      -- 'osram' | 'cisco'
    mcp_source   VARCHAR(100),      -- 수집 MCP 서버명
    created_at   TIMESTAMPTZ DEFAULT NOW(),
    updated_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE products (
    id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_code   VARCHAR(200) UNIQUE NOT NULL,
    name           VARCHAR(500) NOT NULL,
    category_l1    VARCHAR(100),
    category_l2    VARCHAR(100),
    domain         VARCHAR(50),
    vendor         VARCHAR(100),
    document_id    UUID REFERENCES documents(id),
    created_at     TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE product_specs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id      UUID REFERENCES products(id) ON DELETE CASCADE,
    -- [공통 수치]
    wattage_w       NUMERIC(12,3),
    voltage_v       NUMERIC(10,3),
    weight_g        NUMERIC(10,2),
    width_mm        NUMERIC(10,2),
    height_mm       NUMERIC(10,2),
    depth_mm        NUMERIC(10,2),
    -- [조명 도메인 타입 컬럼]
    luminous_flux_lm NUMERIC(12,2),
    luminance_cd_cm2 NUMERIC(12,2),
    color_temp_k     INTEGER,
    cri              NUMERIC(5,2),
    lifetime_hr      NUMERIC(10,1),
    -- [도메인 확장 — JSONB]
    spec_json       JSONB,
    -- 조명: {"current_a":195, "electrode_gap_mm":13.5, "burning_position":"s15"}
    -- IT 서버: {"cpu_cores":32, "ram_gb":256, "rack_unit":2, "bandwidth_gbps":10}
    UNIQUE(product_id)
);

CREATE TABLE es_sync_log (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id  UUID REFERENCES documents(id),
    es_index     VARCHAR(200),
    es_doc_ids   TEXT[],
    chunk_count  INTEGER,
    synced_at    TIMESTAMPTZ DEFAULT NOW(),
    status       VARCHAR(20) DEFAULT 'pending'
);

CREATE TABLE neo4j_sync_log (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id  UUID REFERENCES documents(id),
    node_count   INTEGER,
    edge_count   INTEGER,
    synced_at    TIMESTAMPTZ DEFAULT NOW(),
    status       VARCHAR(20) DEFAULT 'pending'
);

-- 인덱스
CREATE INDEX idx_specs_flux       ON product_specs(luminous_flux_lm);
CREATE INDEX idx_specs_watt       ON product_specs(wattage_w);
CREATE INDEX idx_specs_ctemp      ON product_specs(color_temp_k);
CREATE INDEX idx_products_vendor  ON products(vendor);
CREATE INDEX idx_specs_json       ON product_specs USING GIN(spec_json);
```

---

## 12. 온톨로지 설계

### 12.1 조명 제품 도메인

#### 엔터티 유형

| 유형 | 설명 | 예시 |
|------|------|------|
| `Product` | 개별 제품 (노드의 핵심) | XBO 10000 W/HS OFR |
| `Series` | 제품 시리즈 | XBO 시리즈 |
| `Category` | 제품 분류 계층 | Xenon Lamps > Traditional |
| `Application` | 적용 용도 | Film Projection, Searchlights |
| `Technology` | 핵심 기술 | eXtreme Life (XL), DC Operation |
| `Vendor` | 제조사 | OSRAM, Philips, LEDVANCE |
| `Certification` | 규격/인증 | REACh, RoHS, WEEE, IEC |
| `Chunk` | 문서 청크 (렉시컬 그래프용) | — |
| `Document` | 원본 문서 | ZMP_55851 데이터시트 |

> **참고**: `BurningPosition`(수평/수직 점등 방향)은 Product 노드의 **속성**으로 처리. 엔터티로 별도 구성 시 그래프가 과도하게 세분화됨.

#### 관계 유형

| 관계 | 방향 | 의미 | 속성 |
|------|------|------|------|
| `BELONGS_TO` | Product → Category | 카테고리 소속 | — |
| `IS_PART_OF` | Product → Series | 시리즈 구성원 | — |
| `SUITABLE_FOR` | Product → Application | 적용 용도 | primary(boolean) |
| `USES_TECHNOLOGY` | Product → Technology | 적용 기술 | — |
| `MANUFACTURED_BY` | Product → Vendor | 제조사 | — |
| `HAS_CERTIFICATION` | Product → Certification | 규격 인증 보유 | issued_year |
| `SIMILAR_TO` | Product ↔ Product | 유사 제품 (사양 유사) | similarity_score |
| `REPLACES` | Product → Product | 대체 제품 (단종 등) | since |
| `PART_OF` | Chunk → Document | 청크-문서 연결 | chunk_index |
| `MENTIONS` | Chunk → Entity | 청크-엔터티 연결 | confidence |
| `NEXT_CHUNK` | Chunk → Chunk | 청크 순서 | — |

#### 노드 주요 속성

```
Product {
    product_code: string,          // 주문 코드 (유일키)
    name: string,
    burning_position: string,      // "s15/p15" (엔터티 아님, 속성)
    cooling: string,               // "Forced" / "Natural"
    base_type: string,             // "SFaX30-9.5"
    luminous_class: string,        // "HID"
    lifecycle: string              // "active" / "obsolete"
}
```

### 12.2 IT SI/SM 도메인 온톨로지

IT SI(System Integration) 및 SM(System Management) 산출물 도메인. 서버 스펙, 네트워크 구성도, 운영 매뉴얼, 장애 보고서 등에서 엔터티 추출.

#### 엔터티 유형

| 유형 | 설명 | 예시 |
|------|------|------|
| `Project` | SI/SM 프로젝트 계약 단위 | 인사시스템 고도화 2차, ERP SM |
| `System` | IT 시스템 단위 | 결제시스템, 인사관리시스템 |
| `Server` | 물리/가상 서버 | WEB-01, DB-PROD-01 |
| `NetworkDevice` | 방화벽/스위치/라우터 | FW-DMZ-01, L4-01 |
| `Application` | 소프트웨어 애플리케이션 | Tomcat 9.0, Nginx 1.24 |
| `Database` | DBMS 인스턴스 | Oracle 19c, PostgreSQL 15 |
| `Middleware` | WAS/MQ/ESB | WebLogic 14c, RabbitMQ 3.11 |
| `Library` | 소프트웨어 라이브러리 | Log4j 2.14.1, Spring 5.3 |
| `OS` | 운영체제 | RHEL 8.5, Windows Server 2022 |
| `Team` | 담당 팀/부서 | 결제개발팀, 인프라운영팀 |
| `Vendor` | 납품사/유지보수사 | 삼성SDS, LG CNS |
| `SLA` | 서비스 수준 협약 | 가용성 99.9%, RTO 4hr |
| `Vulnerability` | 보안 취약점 | CVE-2021-44228 (Log4Shell) |
| `Regulation` | 규정/법령/인증 | 개인정보보호법, ISMS-P |
| `Incident` | 장애/이슈 | INC-20240115 DB 접속 실패 |
| `Document` | 산출물 | 설계서, 운영매뉴얼, 변경관리서 |
| `Chunk` | 문서 청크 | — |

#### 관계 유형

| 관계 | 방향 | 의미 | 속성 |
|------|------|------|------|
| `PART_OF` | System → Project | 프로젝트 구성 시스템 | — |
| `DEPLOYED_ON` | Application → Server | 서버 배포 관계 | environment |
| `RUNS_ON` | Application → OS | OS 기반 | version |
| `USES` | Application → Library | 라이브러리 사용 | version |
| `USES` | Application → Database | DB 사용 | — |
| `USES` | Application → Middleware | WAS/MQ 사용 | — |
| `CONNECTS_TO` | Server → NetworkDevice | 네트워크 연결 | port, protocol |
| `CONNECTS_TO` | System → System | 시스템 간 연동 | interface_type |
| `MANAGED_BY` | System → Team | 운영 담당 | role |
| `OWNED_BY` | System → Team | 개발 소유 | — |
| `MAINTAINED_BY` | System → Vendor | 유지보수 계약 | contract_end |
| `GOVERNED_BY` | System → SLA | SLA 적용 | — |
| `GOVERNED_BY` | System → Regulation | 규정 적용 | effective_date |
| `HAS_VULNERABILITY` | Library → Vulnerability | 취약점 해당 | severity |
| `HAS_VULNERABILITY` | OS → Vulnerability | OS 취약점 | — |
| `CAUSED_BY` | Incident → Library | 장애 원인 라이브러리 | — |
| `CAUSED_BY` | Incident → Vulnerability | 취약점으로 인한 장애 | — |
| `AFFECTS` | Incident → System | 영향받은 시스템 | duration_min |
| `DOCUMENTS` | Document → System | 산출물 대상 | doc_type |
| `PART_OF` | Chunk → Document | 렉시컬 그래프 | chunk_index |
| `MENTIONS` | Chunk → Entity | 청크-엔터티 연결 | confidence |
| `NEXT_CHUNK` | Chunk → Chunk | 순서 연결 | — |

#### 수치 데이터 (PostgreSQL — IT SI/SM)

IT SI/SM 서버 스펙은 공통 컬럼 + `spec_json JSONB` 확장:

```sql
-- IT 서버 수치 데이터 (spec_json 예시)
{
  "cpu_cores": 32,
  "cpu_model": "Intel Xeon Gold 6330",
  "ram_gb": 256,
  "storage_tb": 10.0,
  "storage_type": "NVMe SSD",
  "network_gbps": 25,
  "rack_unit": 2,
  "power_redundancy": "N+1"
}

-- SLA 수치 (spec_json 예시)
{
  "availability_pct": 99.9,
  "rto_hr": 4,
  "rpo_hr": 1,
  "response_time_min": 30
}
```

#### IT SI/SM 멀티홉 Cypher 예시

```cypher
-- "Log4j 취약점에 영향받는 시스템과 담당팀"
MATCH (vuln:Vulnerability {cve: 'CVE-2021-44228'})
<-[:HAS_VULNERABILITY]-(lib:Library {name: 'Log4j'})
<-[:USES]-(app:Application)
-[:DEPLOYED_ON]->(srv:Server)
<-[:PART_OF|DEPLOYED_ON*1..2]-(sys:System)
-[:MANAGED_BY]->(team:Team)
RETURN DISTINCT sys.name, team.name, lib.version, vuln.severity
ORDER BY vuln.severity DESC

-- "A 시스템과 연동된 모든 시스템 및 그 담당팀 (2홉)"
MATCH (s1:System {name: $system_name})
-[:CONNECTS_TO*1..2]-(s2:System)
-[:MANAGED_BY]->(t:Team)
RETURN DISTINCT s2.name, t.name
```

---

## 13. 레거시 자산 계승 전략

| 자산 | 파일 | 계승 방법 |
|------|------|----------|
| ES 벡터 스토어 | `vector_store_elasticsearch.py` | `ElasticsearchManager` 재활용, BGE-M3 교체, Sparse 추가 |
| ES Docker | `ElasticSearch/` 전체 | 직접 이식 (Nori 포함) |
| PDF 표 파싱 | `pdf_processor.py` | 표 추출 + 메타데이터 로직 계승 |
| FastAPI 구조 | `app.py` | 재활용, JWT/lifespan 마이그레이션 |
| 한국어 프롬프트 | `rag_system.py` | Graph 컨텍스트 섹션 추가 |
| 검색 벤치마크 | `test_search_modes.py` | RRF + RAGAS 포함 확장 |

**RAGChatbotServer 가중 평균 방식 vs 우리 RRF 방식**:
- 레거시의 `alpha` 파라미터 개념은 RRF `weight` 파라미터로 계승
- 레거시의 ES `hybrid_search` 구조는 RRF 소스 중 하나로 흡수
- 레거시의 `EnsembleRetriever` import는 RRF로 대체

**폐기**: OpenSearch 전체, PGVector 클래스, `PGVectorRAG` 클래스, Jina/OpenAI 임베딩 API 어댑터

---

## 14. 기술 스택 확정

### 14.1 Python 패키지

| 역할 | 패키지 | 최신 버전 | 비고 |
|------|--------|----------|------|
| **API Server** | `fastapi` | **0.136.1** | 레거시 계승 |
| **API Server** | `uvicorn[standard]` | **0.46.0** | ASGI 서버 |
| **RAG 프레임워크** | `langchain` | **1.3.0** | 레거시 계승 |
| **LLM 연동** | `langchain-openai` | **1.1.10** | DeepSeek OpenAI 호환 |
| **Agent 프레임워크** | `langgraph` | **1.1.0** | ReAct Agent (v1.0 LTS) |
| **MCP 클라이언트 런타임** | `langchain-mcp-adapters` | **0.2.2** | DeepSeek ↔ MCP 서버 연결 |
| **MCP 서버** | `fastmcp` | **3.2.4** | 도메인 MCP 서버 구현 |
| **임베딩 (로컬, BGE-M3)** | `FlagEmbedding` | **1.4.0** | Dense+Sparse, ~1.1GB, CPU |
| **Reranker** | `FlagEmbedding` | **1.4.0** | BGE-Reranker-v2-m3 |
| **Vector + BM25 클라이언트** | `elasticsearch8` | **8.19.3** | ES 8.x 서버 전용 |
| **Graph DB 클라이언트** | `neo4j` | **6.2.0** | Bolt 드라이버 |
| **SSOT DB 클라이언트** | `psycopg[binary]` | **3.3.4** | PostgreSQL async 드라이버 |
| **Redis 클라이언트** | `redis` | **7.4.0** | 세션 + ETL 큐 + 임베딩 캐시 |
| **비동기 ETL 큐** | `rq` | **2.8.0** | Redis Queue |
| **파일 감시** | `watchdog` | **6.0.0** | 신규 문서 감지 |
| **PDF 파서 (기본)** | `pdfplumber` | **0.11.9** | 표 구조 추출 |
| **PDF 파서 (보조)** | `PyMuPDF` | **1.27.2** | 이미지·레이아웃 추출 |
| **RAG 평가** | `ragas` | **0.4.3** | Baseline/Graph/Sparse 비교 |

> **v0.5 변경**: `langchain-community`(Jina) 의존성 제거. BGE-M3 단독 정책으로 외부 임베딩 API 어댑터 불필요.

### 14.2 인프라 (Docker Compose)

| 역할 | 이미지 | 버전 | 비고 |
|------|--------|------|------|
| **Vector + BM25** | `elasticsearch` | **8.19.x** | Nori, HNSW, Sparse **← 벡터 단독** |
| **Knowledge Graph** | `neo4j` | **5.x (latest)** | Cypher, 멀티홉 **← 그래프 전담** |
| **SSOT + 수치** | `postgres` | **17** | JSONB 확장 |
| **세션/큐/캐시** | `redis` | **7.4** | 세션 + RQ 백엔드 + 임베딩 캐시 |

### 14.3 LLM / 임베딩

| 역할 | 모델 | 방식 | 비고 |
|------|------|------|------|
| 엔터티 추출 · MCP 오케스트레이션 · 응답 생성 | DeepSeek V4 Pro | API | OpenAI 호환 엔드포인트 |
| 임베딩 (모든 용도) | **BGE-M3 (FlagEmbedding 1.4.0)** | **로컬 CPU** | Dense+Sparse 동시 생성, Redis 캐시 7일 |
| Reranker | BGE-Reranker-v2-m3 | 로컬 CPU | CPU 환경 50건 < 500ms (HRKP 실측) |
| 개발/기능 테스트 | Qwen3 8B Q4 (Ollama) | 로컬 | 기능 확인 전용 |

> **v0.5 변경**: "임베딩 대규모 배치 | Jina Embeddings v3 API" 행 삭제. BGE-M3 로컬 단독으로 통일(H-5).

### 14.4 한국어 처리

| 역할 | 기술 | 비고 |
|------|------|------|
| 한국어 형태소 분석 | Nori (ES 내장 플러그인) | BM25 품질 필수 |

---

## 15. 단계별 구현 로드맵

### Step 1 — ES 기반 RAG + PostgreSQL SSOT (Baseline)

**목표**: Dense + BM25 검색, PostgreSQL SSOT, RAGAS Baseline 측정

- [ ] OpenSearch/PGVector 코드 전체 제거
- [ ] BGE-M3 로컬 임베딩 모듈 (`core/embedding/local_bge_m3.py`) + Redis 캐시
- [ ] SemanticChunker 구현 (`core/chunking/semantic_chunker.py`, M-1)
- [ ] ES Dense (HNSW) + BM25 (Nori) 인덱스 구성
- [ ] RRF 모듈 구현 (`core/rag/rrf.py`) — Step 1에서는 ES 2소스(BM25 + Dense)만 통합
- [ ] FastAPI `lifespan` 마이그레이션, Health Check LLM 호출 제거
- [ ] PostgreSQL SSOT 스키마 구성
- [ ] Redis Queue 비동기 ETL 파이프라인
- [ ] PDFParserFactory + ProductSpecExtractor
- [ ] **RAGAS Baseline 측정** (Dense + BM25)

**완료 기준**: 제품 카탈로그 PDF 적재 → 검색 → RAGAS 기준선 확보

---

### Step 2 — Neo4j GraphRAG 통합 (핵심 실험)

**목표**: GraphRAG 추가 후 RAGAS 향상 측정

- [ ] Neo4j Docker Compose 추가 (통합: ES + Neo4j + PG + Redis)
- [ ] 온톨로지 기반 엔터티 추출 파이프라인 (DeepSeek V4 Pro API, Gleaning 2-pass, M-4)
- [ ] Neo4j 노드/엣지 MERGE + Chunk -MENTIONS-> Entity
- [ ] `HybridGraphRAG` 클래스 + Neo4j 소스 RRF 통합 (graph_search_top_k=3, M-7)
- [ ] BGE-Reranker-v2-m3 도입 (`core/rerank/bge_reranker.py`, M-2)
- [ ] Cypher 멀티홉 패턴 (조명 도메인 + IT SI/SM)
- [ ] **RAGAS 측정: Baseline vs +GraphRAG** ← 핵심 비교
- [ ] JWT 인증 미들웨어 + Redis 세션
- [ ] Osram MCP 서버 기본 구현 + LangChain MCP Adapter(DeepSeek 오케스트레이터) 연동 테스트

**완료 기준**: GraphRAG 추가 전후 RAGAS 점수 비교 결과 확보

> **v0.5 정정 사항**: "DeepSeek V3 API" → "DeepSeek V4 Pro API"로 통일, "Claude 연동 테스트" → "LangChain MCP Adapter(DeepSeek 오케스트레이터) 연동 테스트"로 정정.

---

### Step 3 — Sparse Vector 실험

**목표**: Sparse Vector 추가 효과 정량 측정

- [ ] BGE-M3 → ES sparse_vector 변환 (`generate_sparse_vector()`)
- [ ] ES sparse_vector 인덱스 추가 (재인덱싱 또는 업데이트)
- [ ] RRF에 Sparse 소스 추가 (weight=0.7, M-6)
- [ ] **RAGAS 측정: +GraphRAG vs +GraphRAG+Sparse**
- [ ] CPU 인덱싱 속도 벤치마크 (Dense vs Dense+Sparse)

---

### Step 4 — 운영화 및 MCP 고도화

- [ ] Osram MCP 전체 도구 완성
- [ ] IT SI/SM 도메인 MCP 초안
- [ ] 문서 갱신 파이프라인 (변경 시 Triple Store 재동기화)
- [ ] RAGAS 자동화 측정
- [ ] 패키지 구조 리팩토링 (`core/` + `mcp_servers/` + `api/`)

---

## 16. 미결 사항

### ✅ 결정 완료

| # | 항목 | 결정 내용 | 근거 |
|---|------|----------|------|
| H-1 | 도메인 | 조명 제품 카탈로그 (1차), IT SI/SM (2차) | 기획 확정 |
| H-2 | LLM 선택 | **DeepSeek V4 Pro API** (엔터티 추출 · MCP 오케스트레이션 · 응답 생성 통합) | v0.4 수정 — V3 → V4 Pro |
| H-3 | Sparse Vector | BGE-M3, Step 3 실험 변수, ELSER 배제 | 하드웨어 제약 |
| H-4 | 수치 추출 방식 | DeepSeek API 추출 (레거시 정규식 방식 탈피) | 레거시 regex 한계 확인 |
| H-5 | 임베딩 전략 | **BGE-M3 로컬 단독 (BAAI/bge-m3, 1024차원)** — Jina/OpenAI 미사용 | HRKP 실 구현 확인 (embedding_test_results_2026-02-02): BGE-M3 단독 적용, Redis 캐시, 7.5 texts/sec 배치 성능 실증. 추가 API 비용 없이 완전 로컬 운용 |
| H-6 | 벡터 저장소 | ES 단독 (Neo4j 중복 저장 없음) | 역할 분리 원칙 |
| H-7 | RRF 설계 | 커스텀 RRF (레거시 가중 평균 방식 불채택) | 레거시 custom-hybrid alpha=0.6 분석 |
| H-8 | MCP 오케스트레이터 | **DeepSeek V4 Pro + LangChain MCP Adapter** (Claude 클라이언트 가설 폐기) | v0.4 수정 — 단일 LLM 단순화 |
| H-9 | IT SI/SM 온톨로지 | v0.4에서 신규 작성 완료 | 문서 내 확정 |
| M-1 | 청크 크기 | **SemanticChunker (BGE-M3 임베딩 기반) — target=1024 tokens / overlap=128 tokens (12.5%) / 허용 범위 256~2048 tokens** | HRKP STORY-003 (Done, 2026-01-26): 고정 크기 분할 대신 BGE-M3 임베딩으로 의미 경계를 감지하는 SemanticChunker 채택. 기술 매뉴얼 도메인에서 문단 단위 의미 완결성을 보장하며, 너무 작은 청크(< 256)와 너무 큰 청크(> 2048)는 자동 재분할 |
| M-2 | Reranker | **BGE-Reranker-v2-m3 도입 확정 — RRF 상위 50건 대상 rerank → top_k 반환, CPU/GPU 자동 감지** | HRKP STORY-032 (Done): `BAAI/bge-reranker-v2-m3` 크로스 인코더 기반 관련성 점수 계산, CPU 환경에서 50건 < 500ms 실증, 65 tests passed. 구현: `asyncio.to_thread()` 비동기 래핑, batch_size=32, Sigmoid 정규화(0~1) |
| M-3 | Neo4j Community 메모리 | **heap_initial=256m / heap_max=512m / pagecache=256m / container limit=1GB** (Docker Compose 고정) | HRKP container_memory_tuning_guide.md 실측 적용값 (2026-01-21): ES(2GB)+Neo4j(1GB)+PG(512MB)+Redis(256MB) 동시 운용 기준 Windows 노트북 16GB 환경 안정 확인. heap_max=512m은 Neo4j 공식 권장 최솟값; pagecache는 자주 접근하는 그래프 페이지 캐시 |
| M-4 | 엔티티 추출 방식 | **LLM 기반 엔티티 추출 (DeepSeek V4 Pro, deepseek-chat 엔드포인트) + Gleaning 2-pass (+33% Recall) / 신뢰도 임계값 ≥ 0.7 — 7개 타입 확정: Person, Organization, Technology, Project, Concept, Date, Location** | HRKP entity_extraction.py 실 구현 + 2026-02-15 배치 완료 보고서: 16,185 청크 처리 → 70,855개 고유 엔티티, 375,229개 관계 추출. Gleaning(2-pass) 적용으로 1-pass 대비 Recall +33%. 수치형 데이터(wattage, lifespan 등)는 ETL 파이프라인에서 PostgreSQL 별도 스키마로 분리 저장 |
| M-5 | 임베딩 모델 선택 | **BGE-M3 로컬 단독 채택 (BAAI/bge-m3, 1024차원) — Jina v3/OpenAI ada-002 미사용** | HRKP embedding_test_results_2026-02-02 실증: BGE-M3 단독으로 다국어 임베딩 운용, Redis TTL=7일 캐시, 배치 7.5 texts/sec, 벡터 검색 cosine similarity 정확도 검증 완료. Jina/OpenAI는 HRKP 전체 코드베이스에 미존재; 추가 API 비용 없이 완전 로컬 운용 |
| M-6 | RRF 파라미터 | **k=60 / rrf_weight_vector=1.0 / rrf_weight_keyword=1.0 / rrf_weight_sparse=0.7 / rrf_weight_graph=0.8** | HRKP 34_graph_search_rrf_tuning.md (2026-03-09 확정): Vector·Keyword 각 top_k×2=20건 후보 생성, Sparse=0.7(Dense 대비 보조 역할), Graph=0.8(그래프 경로 기여 반영). k=60은 Cormack et al. 표준값으로 HRKP에서 운용 검증 완료 |
| M-7 | Graph 검색 후보 수 | **graph_search_top_k=3 고정 (Graph 소스 후보 수 제한 방식) — weight 조정 방식 ADR 거부** | HRKP 34_graph_search_rrf_tuning.md ADR (Accepted 2026-03-09): Graph weight를 높게(1.5) 설정하면 Graph 결과가 10건 중 8건 1위를 독식하는 문제 발생. weight 조정 대신 Graph 후보 수를 top_k×2(=20)에서 3으로 고정하여 해결. Vector·Keyword는 각 20건 유지로 균형 있는 RRF 퓨전 보장 |

### ⏳ 향후 검토 사항 (v0.5 신규)

| # | 항목 | 메모 |
|---|------|------|
| F-1 | DeepSeek V4 Pro 실제 단가 | API 가격 정책 확정 후 10.3 비용 추정 갱신 |
| F-2 | Step 1 RAGAS Baseline 결과 | v0.6에서 실측 수치 반영 |
| F-3 | Neo4j 5.x ↔ neo4j-python 6.2.0 호환성 | Step 2 진입 전 통합 테스트로 확인 |

---

## 17. 부록

### 부록 A. RAGAS 평가 지표

| 지표 | 설명 |
|------|------|
| Faithfulness | 응답이 컨텍스트에 충실한가 (환각 방지) |
| Answer Relevancy | 응답이 질문에 관련 있는가 |
| Context Recall | 관련 정보가 검색됐는가 |
| Context Precision | 검색된 정보가 실제 유용한가 |

### 부록 B. 임베딩 전략

```
문서 수신
  ↓
로컬 BGE-M3 (BAAI/bge-m3) — 단독 운용
  - Dense Vector (1024차원) → Elasticsearch
  - Sparse Vector (BGE-M3 SPLADE) → Elasticsearch (Step 3 이후)
  - 캐시: Redis TTL=7일 (중복 임베딩 방지)
  - 배치 처리: 7.5 texts/sec (CPU 환경 실측)
```

> **HRKP 실증**: BGE-M3 단독으로 다국어(한국어 포함) 임베딩 운용. Jina/OpenAI 외부 API 미사용.

### 부록 C. 참고 자료

- [RAG 기술 아키텍처 세미나 (1~7)](https://k82022603.github.io) ← **아키텍처 기준**
- [Neo4j GraphRAG Python 패키지](https://github.com/neo4j/neo4j-graphrag-python)
- [BGE-M3 HuggingFace](https://huggingface.co/BAAI/bge-m3)
- [RAGAS 평가 프레임워크](https://github.com/explodinggradients/ragas)
- [DeepSeek API](https://platform.deepseek.com/docs)
- [BGE-Reranker-v2-m3 HuggingFace](https://huggingface.co/BAAI/bge-reranker-v2-m3)
- [FastMCP](https://gofastmcp.com)
- [RRF 원논문: Cormack et al. 2009](https://dl.acm.org/doi/10.1145/1571941.1572114)

### 부록 D. 기술결정 로그

> 각 결정 사항의 **배경, 검토된 대안, 채택 이유**를 기록한다. 향후 재검토 시 맥락 유지를 위함.

---

**[TD-01] Dense Vector 저장소: ES 단독 채택 (Neo4j 중복 배제)**
- **결정일**: 2026-05-14 (v0.4)
- **배경**: v0.3에서 Neo4j를 Semantic Hub로 설계하며 Dense 벡터를 Neo4j에 저장하는 방안을 검토했음
- **검토된 대안**:
  - A) ES + Neo4j 양쪽에 Dense 벡터 저장 → 거부: 동기화 복잡도, 메모리 낭비
  - B) Neo4j 단독 Dense 벡터 → 거부: BM25/Nori/Sparse와 통합 불가
  - C) ES 단독 Dense + Neo4j 그래프 ← **채택**
- **채택 근거**: 세미나(7) 아키텍처 명시. ES가 Dense+Sparse+BM25 통합 처리. Neo4j는 Cypher 멀티홉에 집중.
- **제약**: Neo4j의 "벡터+그래프 단일 Cypher" 장점 포기. 단, RRF로 결과 통합하면 동일 효과 달성 가능.

---

**[TD-02] RRF 채택 (가중 평균 방식 불채택)**
- **결정일**: 2026-05-14 (v0.4)
- **배경**: RAGChatbotServer는 `alpha=0.6` 고정 가중 점수 평균을 사용. ES 2개 소스만 통합.
- **검토된 대안**:
  - A) 레거시 방식 계승 (가중 평균) → 거부: 점수 스케일 불일치, Neo4j 통합 불가
  - B) ES 내장 hybrid (BM25+Dense) + Neo4j 별도 → 거부: 진정한 RRF 아님
  - C) 커스텀 RRF ← **채택**: 순위 기반, 스케일 정규화 불필요, 3+ 소스 통합 가능
- **채택 근거**: 수학적 근거 있음(2009 논문), k=60 표준값 robust, weight로 GraphRAG 강조 가능. 단, weight 조정만으로는 Graph 결과 편향이 발생하여 후보 수 제한 방식(M-7)을 병행한다.

---

**[TD-03] DeepSeek API 전용 정책**
- **결정일**: 2026-05-14 (v0.3) / **모델 명세 갱신**: 2026-05-14 (v0.4)
- **배경**: 중국 모델의 로컬 실행 가능 여부 검토
- **결론**: DeepSeek V4 Pro full(~220GB), Q4(~100GB) 모두 16GB RAM에서 실행 불가. Qwen3 8B Q4(~5GB)는 가능하나 속도 1~3 token/s로 운영 용도 부적합.
- **정책**: 중국 계열 모델 전체 API 전용. Qwen3 8B Q4는 개발/기능 테스트 전용.
- **선정 모델**: **DeepSeek V4 Pro** (OpenAI 호환 엔드포인트) — 엔터티 추출/MCP 오케스트레이션/응답 생성 통합 담당.

> **v0.5 정정 사항**: v0.4 문서의 "DeepSeek V3" 표기는 v0.3 시점 잔재였다. 현재 결정 모델은 V4 Pro다(H-2).

---

**[TD-04] MCP 오케스트레이터: DeepSeek V4 Pro + LangChain MCP Adapter 채택**
- **결정일**: 2026-05-14 (v0.4)
- **배경**: 사용자가 도메인별 MCP 서버 아이디어를 제안. 초기에는 "MCP는 Anthropic 프로토콜이라 Claude만 클라이언트가 될 수 있다"는 가설로 검토했으나, MCP 프로토콜이 모델 비특정 오픈 스탠다드이며 `langchain-mcp-adapters`를 통해 임의의 LLM(OpenAI 호환 포함)이 MCP 클라이언트가 될 수 있음을 확인.
- **검토된 대안**:
  - A) Claude 클라이언트 전용으로 MCP 서버 노출 → 거부: 시스템 외부 의존 추가, DeepSeek 단일 LLM 정책과 상충
  - B) DeepSeek는 OpenAI 호환 function calling만 사용, MCP 서버 미도입 → 거부: 도메인별 MCP 자산을 활용하지 못함
  - C) **DeepSeek V4 Pro + LangChain MCP Adapter** ← **채택**: 단일 LLM이 엔터티 추출·오케스트레이션·응답 생성을 모두 담당하여 아키텍처가 단순해짐
- **채택 근거**: ① LangChain Agent의 Tool 인터페이스로 MCP 서버를 노출 가능, ② DeepSeek 단일 LLM로 비용/관리 단순화, ③ MCP 자산은 그대로 유지하면서 외부 Claude 의존 제거.

> **v0.5 정정 사항**: v0.4 문서에는 "Claude 전용 명시"라는 v0.3 시점 결론이 부록 D에 잔존해 있었다. v0.5에서 본문 9절·H-8과 일치하도록 결정 내용을 갱신했다.

---

**[TD-05] BGE-M3 로컬 단독 채택 (Jina/OpenAI 임베딩 API 배제)**
- **결정일**: 2026-05-14 (v0.4)
- **배경**: v0.3까지는 "소량은 로컬 BGE-M3, 대량 배치는 Jina v3 API" 이중 전략을 검토했음.
- **검토된 대안**:
  - A) Jina v3 API 병행 (소량 로컬 / 대량 API) → 거부: 외부 API 비용 발생, 캐시·재현성 관리 복잡
  - B) OpenAI text-embedding-3 API → 거부: 1536차원으로 BGE-M3와 차원 불일치, 한국어 품질 BGE-M3 대비 우위 없음
  - C) **BGE-M3 로컬 단독** + Redis TTL=7일 캐시 ← **채택**
- **채택 근거**: HRKP 실 운용 결과 7.5 texts/sec(배치)로 운영 요건 충족. Redis 캐시로 중복 임베딩 비용 최소화. 외부 API 비용 0원, 모든 데이터 사내 보존.

---

**[TD-06] ELSER 배제, BGE-M3 Sparse 채택**
- **결정일**: 2026-05-14 (v0.3)
- **근거**: ELSER v2는 영어 전용. 한국어 문서에서 성능 매우 낮음. BGE-M3 Sparse는 100+ 언어 지원, 한국어 MIRACL 벤치마크 최고 수준. token_id→token_string 변환 코드 구현 필요하나 실용적.

---

**[TD-07] RRF Graph 가중치보다 후보 수 제한 우선 (HRKP 튜닝 반영)**
- **결정일**: 2026-03-09 (HRKP), v0.5에서 본 문서 반영
- **배경**: v0.4 본문 6.6에서는 Graph weight=1.5로 GraphRAG를 강조하도록 설계했으나, HRKP 실 운용에서 Graph 결과가 RRF top-10 중 8건을 점유하는 편향이 관찰됨.
- **검토된 대안**:
  - A) Graph weight 조정(1.5 → 1.0 → 0.5) → 거부: 가중치 미세 조정으로는 편향이 충분히 완화되지 않음
  - B) **graph_search_top_k=3 고정 + Graph weight=0.8** ← **채택**
- **채택 근거**: Graph 소스에서 반환되는 후보 수 자체를 제한하면 RRF 누적 점수 기여가 자연스럽게 균형. Vector·Keyword 후보 수(20건)는 유지. HRKP ADR 34_graph_search_rrf_tuning.md(Accepted) 참조.

---

>- **작성일**: 2026-05-15
>- **성격**: 정합성 정리 릴리스 (v0.4 내부 모순 일괄 수정, 신규 의사결정 없음)
>- **다음 수정 예정**: Step 1 구현 완료 후 RAGAS Baseline 결과 + DeepSeek V4 Pro 단가 실측 반영 (v0.6)
