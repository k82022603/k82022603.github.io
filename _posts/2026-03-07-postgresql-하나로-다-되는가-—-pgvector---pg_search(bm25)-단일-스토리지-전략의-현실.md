---
title: "PostgreSQL 하나로 다 되는가 — pgvector + pg_search(BM25) 단일 스토리지 전략의 현실"
date: 2026-03-07 22:30:00 +0900
categories: [TechStack,  Backend Architecture]
mermaid: [True]
tags: [BM25,  PostgreSQL,  Elasticsearch,  MongoDB,  pgvector,  pg_search,  Pinecone,  Claude.write]
---


> **태그:** #PostgreSQL #데이터베이스 #백엔드개발  
> **참고:** Redis, Elasticsearch, Pinecone, MongoDB를 직접 운영한 경험을 바탕으로, PostgreSQL 단일화 전략의 기술적 타당성을 실측 벤치마크와 함께 분석한다.

## 관련영상

[**Redis, Elasticsearch 다 버렸습니다. 결과는?**](https://www.youtube.com/watch?v=2-AFK8JJpig)


---

## 1. 문제 제기: DB 4개를 운영한다는 것의 실제 의미

많은 팀들이 "각 도구의 장점을 극대화한다"는 명분 아래 아키텍처를 분화시킨다. 트랜잭션은 PostgreSQL, 검색은 Elasticsearch, 캐시는 Redis, 벡터는 Pinecone. 이론상으로는 나무랄 데 없는 역할 분담이다. 문제는 이게 실제 운영 환경에 들어서는 순간부터 복잡도가 가시적으로 드러난다는 점이다.

가장 먼저 느끼는 것은 **비용**이다. Elasticsearch 클러스터 유지비는 규모에 따라 월 수백만 원을 훌쩍 넘는다. ParadeDB가 50개 이상의 기업들을 인터뷰한 결과, 일부 엔터프라이즈 팀에서는 Elasticsearch가 소프트웨어 비용 중 가장 큰 항목으로 성장했다는 사례가 적지 않았다. Pinecone Pro 플랜 역시 벡터 수와 쿼리 볼륨에 따라 비용이 가파르게 올라간다.

두 번째 문제는 **데이터 동기화**다. PostgreSQL이 원본 데이터를 갖고 있는 상태에서 Elasticsearch로 데이터를 복제하면, 이 두 저장소 사이의 일관성을 유지하는 것이 생각보다 훨씬 어렵다. ETL 파이프라인이 필요하고, 파이프라인이 깨지면 검색 결과는 수 시간 전의 데이터를 반환하게 된다. Kafka나 Kinesis 같은 메시지 큐를 붙이면 이 문제를 일부 해소할 수 있지만, 그것은 스택에 또 하나의 복잡성을 추가하는 것이다. 결국 데이터 동기화 문제를 해결하기 위해 메시지 큐를 붙이고, 메시지 큐를 운영하기 위해 전문 인력이 필요해진다.

세 번째는 **운영 부담의 배수 효과**다. DB가 하나 추가될 때마다 백업 전략, 모니터링 체계, 접근 제어 설정, 개발 환경 구성, CI/CD 파이프라인이 그 DB에 맞게 별도로 존재해야 한다. 신규 팀원이 온보딩할 때 배워야 할 쿼리 언어도 하나씩 늘어난다. 장애가 발생했을 때 원인을 찾아야 할 곳도 DB 수만큼 늘어난다.

이 구조적 복잡성이 축적되면 팀은 제품을 만드는 시간보다 인프라를 유지하는 데 더 많은 시간을 소비하게 된다. 그리고 이 지점에서 자연스럽게 "PostgreSQL 하나로 다 할 수 있지 않을까?"라는 질문이 제기된다.

---

## 2. PostgreSQL BM25 확장 지형도: pg_search vs pg_textsearch

이 주제를 논하기 전에 현재 PostgreSQL 생태계에서 BM25를 구현하는 두 확장의 차이를 먼저 짚어야 한다.

| 확장 | 제작 | 엔진 | 상태 |
|---|---|---|---|
| `pg_search` | ParadeDB | Rust + Tantivy (Lucene 대안) | Production-ready |
| `pg_textsearch` | Timescale / Tiger Data | C | 2025.12 오픈소스 신규 공개 |
| 네이티브 `tsvector + ts_rank` | PostgreSQL 코어 | C | BM25 미지원, IDF 없음 |

`pg_search`는 Rust 기반 검색 라이브러리 Tantivy 위에 구축되어 BM25 알고리즘 기반 전문 검색, 패싯 검색, 하이브리드 검색을 지원하며 현재 가장 성숙한 구현체다. `pg_textsearch`는 pgvectorscale을 만든 Timescale 팀이 2025년 12월에 오픈소스로 공개한 신규 확장으로, pgvector와의 하이브리드 검색 스택을 목표로 한다. 두 확장이 공존하는 상황이며, pg_search가 역사와 레퍼런스 면에서 앞선다.

네이티브 `tsvector + ts_rank`는 IDF(역문서빈도)를 계산하지 않기 때문에 BM25가 아니다. 이후 이 문서에서 "PostgreSQL 전문 검색"이라고 할 때는 **pg_search(BM25 인덱스 기반)** 을 가리키며, 네이티브 FTS와 혼동하지 않도록 주의해야 한다.

---

## 3. PostgreSQL 확장 생태계의 현재: 무엇이 가능해졌는가

2025~2026년 기준으로 PostgreSQL 확장 생태계는 상당히 성숙했다. 핵심 세 가지는 다음과 같다.

**pgvector / pgvectorscale** — 벡터 임베딩 저장과 유사도 검색을 PostgreSQL 안에서 처리한다. Timescale이 개발한 pgvectorscale은 기존 pgvector에 StreamingDiskANN이라는 새로운 ANN(Approximate Nearest Neighbor) 인덱스 알고리즘을 추가했다. 이 인덱스는 HNSW가 벡터를 메모리에 올려두는 것과 달리 디스크 기반으로 동작하기 때문에, 벡터 수가 늘어날수록 RAM이 아닌 SSD 비용만 추가된다.

**pg_search (ParadeDB)** — Rust로 구현된 Tantivy 위에 올라가는 확장으로, BM25 알고리즘 기반의 전문 검색을 PostgreSQL 안에서 제공한다. `CREATE INDEX ... USING bm25` 문법으로 인덱스를 만들고 `@@@` 연산자로 쿼리한다. 실시간 검색이 가능하며 별도 재인덱싱 없이 신규 데이터가 즉시 검색된다.

**pg_textsearch (Timescale / Tiger Data)** — 2025년 12월에 OSS로 공개된 신규 확장으로, `<@>` 연산자를 통한 BM25 관련도 정렬 검색을 지원하며 pgvector와의 하이브리드 검색을 목표로 한다.

이 세 확장이 하나의 PostgreSQL 인스턴스 위에서 동작한다는 것이 핵심이다. 하나의 연결, 하나의 백업 전략, 하나의 권한 체계. 동기화 파이프라인이 불필요하다.

---

## 4. "100만 건을 단일 PostgreSQL에 때려넣으면?" — 기술적 현실

결론부터 말하면 **"할 수는 있다. 그런데 누가 운영하냐가 문제다."**

BM25 인덱스는 LSM 트리로 디스크에 저장되며, 각 세그먼트는 역색인과 컬럼 인덱스를 포함한다. 역/컬럼 인덱스는 빠른 읽기를, LSM 트리는 고빈도 쓰기를 최적화한다. 100만 건 정도는 인덱스 구조상 충분히 감당 가능하며, 단일 스토리지 구성 자체가 기술적 허들이 되지는 않는다.

문제는 운영 환경에서 실제로 맞닥뜨리는 다섯 가지 현실적 복잡성이다.

**1. 인덱스 중첩 비용**

하나의 테이블에 pgvector HNSW/IVFFlat 인덱스, pg_search BM25 인덱스, 기본 B-tree 인덱스가 동시에 존재하면 INSERT/UPDATE 시 세 종류의 인덱스를 동시에 갱신해야 한다. 특히 HNSW는 인덱스 빌드 비용이 매우 무겁고, BM25 인덱스도 Tantivy 세그먼트 병합(merge) 작업이 백그라운드에서 발생한다. 실시간 대량 ingestion 워크로드에서 write amplification이 심해진다.

**2. 메모리 압박**

`paradedb.statement_memory_budget` 설정의 기본값은 인덱싱 스레드당 1GB이며, 대용량 삽입 시 더 높은 값이 성능을 개선할 수 있다. pgvector HNSW는 `hnsw.ef_construction` 값에 따라 빌드 시 별도 메모리를 사용한다. 단일 머신에서 두 인덱스가 동시에 빌드/쿼리되는 상황은 메모리 계획을 매우 세밀하게 잡아야 한다.

**3. 파티셔닝 시 BM25 점수 신뢰성 저하**

파티셔닝된 테이블의 BM25 인덱스는 파티션별로 독립적인 통계를 사용한다. 파티션 A에 문서 1,000개, 파티션 B에 10개가 있다면 "database"라는 단어의 IDF 값이 각 파티션에서 다르게 계산되므로, 크로스-파티션 쿼리의 점수는 직접 비교할 수 없다. 100만 건이 파티셔닝된 구조라면 하이브리드 검색 점수의 신뢰성이 무너진다.

**4. 하이브리드 검색의 점수 결합 문제**

BM25 점수와 벡터 유사도 점수는 완전히 다른 스케일에서 측정되므로 단순히 더할 수 없다. RRF(Reciprocal Rank Fusion) 같은 기법이 필요하며, SQL 레벨에서 직접 구현하거나 ParadeDB의 헬퍼 함수를 써야 하는데 쿼리 복잡도가 크게 높아진다.

**5. 관리형 서비스(Supabase 등) 미지원**

2025년 기준으로 Supabase와 ParadeDB의 통합은 아직 불확실한 상태다. Supabase RLS(Row Level Security)를 사용하는 경우 외부 ParadeDB 인스턴스로 논리적 복제를 하면 인증 로직을 ParadeDB 쪽에서 재구현해야 하는 문제가 생긴다. 관리형 PostgreSQL 서비스에서는 아직 이 확장들을 자유롭게 쓸 수 없는 경우가 많다.

---

## 5. 성능 현실: "전문 DB보다 느리지 않냐"는 질문에 대한 답

### 5-1. pgvectorscale vs Pinecone 실측 벤치마크

"PostgreSQL은 벡터 검색에서 Pinecone 같은 전문 DB보다 느리다"는 통념이 있었다. 이와 관련해 가장 많이 인용되는 수치가 있다. **pgvectorscale을 개발한 Timescale이** 5천만 개의 Cohere 임베딩(768차원)을 대상으로 ANN-Benchmarks 도구를 사용해 직접 수행한 벤치마크 결과다.

- **Pinecone s1(스토리지 최적화) 대비:** p95 레이턴시 기준 28배 낮음, 쿼리 처리량 16배 높음, 비용 75% 절감 (AWS EC2 셀프호스팅 기준)
- **Pinecone p2(성능 최적화) 대비:** p95 레이턴시 1.4배 낮음, 쿼리 처리량 1.5배 높음

이 수치를 해석할 때 중요한 맥락이 있다. **이 벤치마크는 pgvectorscale을 만든 Timescale이 직접 수행한 자체 조사**다. 방법론과 데이터셋은 공개되어 있으나, 제품을 보유한 측이 직접 수행한 비교라는 점은 감안해야 한다. 다만 성능 격차의 구조적 이유는 분명하다. Pinecone은 별도 서비스이므로 모든 벡터 쿼리에 네트워크 레이턴시가 붙고, pgvector는 데이터가 이미 존재하는 PostgreSQL 안에서 직접 실행된다. 또한 Pinecone은 정확도-성능 트레이드오프 파라미터를 사용자에게 노출하지 않지만, PostgreSQL은 인덱스 파라미터를 직접 튜닝할 수 있다. 이 방향의 결론을 지지하는 독립적인 비교 자료들도 존재하며, 단, 이는 1억 건 미만의 규모에서의 이야기다. 수십억 벡터를 밀리초 단위로 처리해야 하는 Google, Netflix, TikTok 수준의 워크로드에서는 여전히 전문 벡터 DB가 맞다.

### 5-2. PostgreSQL + pg_search vs Elasticsearch 비교

전문 검색 영역에서 Elasticsearch는 분산 아키텍처와 Lucene 기반 BM25 구현으로 수십억 건 데이터를 밀리초 단위에 처리하는 명실상부한 최강자다. 이 우위는 규모가 커질수록 뚜렷해진다.

중요한 전제를 먼저 짚어야 한다. 여기서 비교 대상은 **pg_search(BM25 인덱스 기반)** 이며, tsvector 기반의 네이티브 PostgreSQL FTS와는 완전히 다른 이야기다. 네이티브 FTS는 IDF(역문서빈도)를 계산하지 않고 ts_rank 방식으로 동작하기 때문에 대용량에서 Elasticsearch 대비 성능 열세가 뚜렷하다. 반면 pg_search는 Tantivy(Rust 기반 Lucene 대안) 위에서 BM25를 구현하므로, 중간 규모 데이터셋에서 Elasticsearch와 같은 방식으로 관련도 점수를 계산하며 경쟁력 있는 성능을 보인다. pg_search는 tsvector가 갖지 못했던 BM25 스코어링, 퍼지 매칭, 구문 검색을 PostgreSQL SQL 안에서 제공한다.

실제로 Elasticsearch를 PostgreSQL FTS로 교체한 사례들을 보면 공통된 동기가 있다. 인덱스 동기화 유지의 어려움, Elasticsearch 전문가 인력 확보의 어려움, 그리고 "우리 팀은 SQL을 이미 잘 아는데 왜 굳이 다른 쿼리 DSL을 배워야 하는가"라는 실용적 판단이다.

Elasticsearch가 진짜로 필요한 순간은 명확하다. 매우 복잡한 집계 쿼리를 초저지연으로 처리해야 할 때, 수십억 문서를 밀리초 안에 검색해야 할 때, ELK 스택 전체(보안, 옵저버빌리티, 로그 분석)를 사용하는 경우다. 이 기준을 넘지 않는 팀이라면 Elasticsearch는 솔직히 오버스펙이다.

---

## 6. 하이브리드 검색의 실제: pgvector + pg_search 조합

두 확장을 함께 쓰는 하이브리드 검색은 렉시컬 정밀도(BM25)와 시맨틱 이해(벡터 유사도)를 동시에 활용한다. "PG-15.4"같은 정확한 코드나 기술 용어는 벡터 검색이 놓칠 수 있다. 벡터 공간에서 "PostgreSQL 15.4"와 "PostgreSQL 14.2"는 거의 같은 위치에 있기 때문이다. 반대로 BM25는 시맨틱 유사성을 모른다. 두 방식을 결합하면 각자의 약점을 보완할 수 있다.

다만 BM25 점수와 벡터 유사도 점수는 서로 다른 스케일에서 측정되므로 단순히 더할 수 없다. 실무에서 가장 많이 쓰이는 결합 방식은 **Reciprocal Rank Fusion(RRF)** 이다. 두 방식의 랭킹 순위를 각각 구한 뒤, 순위 기반 점수로 변환하여 합산하는 방식이다.

아래는 RRF의 개념을 SQL로 표현한 **개념적 예시 코드**다. `paradedb.score()` 함수의 실제 시그니처나 동작 방식은 pg_search 버전에 따라 다를 수 있으므로, 실제 환경에서는 반드시 [ParadeDB 공식 문서](https://docs.paradedb.com)를 기준으로 검증 후 사용해야 한다.

```sql
-- RRF를 이용한 하이브리드 검색 예시
WITH bm25_results AS (
    SELECT id, paradedb.score(id) AS bm25_score,
           ROW_NUMBER() OVER (ORDER BY paradedb.score(id) DESC) AS bm25_rank
    FROM documents
    WHERE description @@@ 'postgresql performance'
    LIMIT 100
),
vector_results AS (
    SELECT id,
           1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS cosine_sim,
           ROW_NUMBER() OVER (ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector) AS vec_rank
    FROM documents
    LIMIT 100
)
SELECT
    COALESCE(b.id, v.id) AS id,
    (1.0 / (60 + COALESCE(b.bm25_rank, 100))) +
    (1.0 / (60 + COALESCE(v.vec_rank, 100))) AS rrf_score
FROM bm25_results b
FULL OUTER JOIN vector_results v ON b.id = v.id
ORDER BY rrf_score DESC
LIMIT 10;
```

이 쿼리가 단일 PostgreSQL 연결 안에서 실행된다는 점이 핵심이다. 외부 서비스 호출 없이, 네트워크 레이턴시 없이, 별도 오케스트레이션 레이어 없이.

---

## 7. 단일화 전략이 맞는 팀과 맞지 않는 팀

핵심 판단 기준은 데이터 규모보다 **팀의 역량과 워크로드 패턴**이다. 100만 건은 기술적으로는 전혀 문제 없는 규모이지만, "단일 스토리지 올인"이 주는 운영 단순성의 혜택을 온전히 받으려면 이 스택의 특성을 깊이 이해하는 사람이 반드시 한 명은 있어야 한다. 없으면 Elasticsearch가 오히려 낫다 — 적어도 운영 레퍼런스는 풍부하니까.

| 케이스 | 적합 여부 | 이유 |
|---|---|---|
| **소규모 팀, 100만 건, 읽기 위주** | ✅ 적합 | 단순하고 운영 오버헤드 최소, Elasticsearch 없어도 됨 |
| **실시간 고빈도 write + 검색** | ⚠️ 주의 | 인덱스 write amplification, 인덱스 머지 지연 |
| **데이터 수천만 건 이상** | ❌ 재검토 | pgvector 정확도 저하, BM25 통계 왜곡 가능 |
| **엔터프라이즈 SLA, 전문 검색팀 없음** | ❌ 위험 | 두 인덱스 동시 튜닝 + 장애 대응 역량 필요 |
| **PostgreSQL DBA + MLOps 내재화된 팀** | ✅ 적합 | 아키텍처를 단순하게 유지할 수 있는 최강의 선택지 |

### 적합한 케이스

규모가 데이터 100만 건, DAU 수만 명 이하인 팀이라면 단일 PostgreSQL 전략은 매우 강력한 선택이다. 기술적 이유 이상으로 조직적 이유가 강하다. 새 팀원이 PostgreSQL만 알면 검색, 벡터, 큐 시스템까지 모두 이해하고 기여할 수 있다. 장애가 발생했을 때 들여다볼 곳이 하나다. 비개발 직군이 데이터를 조회할 때 SQL 하나로 해결된다.

스타트업, 스케일업 초기, 사이드 프로젝트, AI 애플리케이션 프로토타입 — 이 모든 상황에서 복잡성보다 단순성이 더 강한 아키텍처다.

---

## 8. 마이그레이션 실전 접근법

새 프로젝트라면 처음부터 PostgreSQL로 시작하고, 필요해질 때 확장을 하나씩 붙이는 것이 가장 합리적이다. 벡터가 필요하면 그때 pgvector를 설치하고, 검색 관련도가 중요해지면 그때 pg_search를 붙인다.

기존에 Elasticsearch나 Pinecone을 운영 중인 팀이 전환을 고려한다면 점진적 접근이 안전하다. 먼저 같은 데이터에 대해 pg_search와 pgvector를 병렬로 구성하고, 쿼리 성능과 검색 품질을 직접 비교한다. 실제 차이가 없거나 오히려 빠르다면, 가장 덜 중요한 검색 엔드포인트부터 전환하여 검증한다.

모니터링 도구로는 PostgreSQL이 이미 제공하는 `EXPLAIN ANALYZE`와 `pg_stat_statements`가 충분히 강력하다. 슬로우 쿼리 포착, 인덱스 활용 여부 확인, 인덱스 사이즈 모니터링을 통해 단일 PostgreSQL 환경을 안정적으로 운영할 수 있다.

---

## 9. 결론: 복잡성은 필요할 때 사는 것이다

벡터 DB, BM25 검색 엔진, 메시지 큐를 전문 서비스로 분리하는 것은 올바른 도구를 선택하는 것처럼 보이지만, 그 복잡성을 감당할 팀 역량과 규모가 전제되어야 한다. 그 전제가 충족되지 않은 상태에서 미래를 대비한 복잡성은 지금 당장의 고통이 된다.

pgvectorscale이 Pinecone보다 같거나 더 빠른 벤치마크를 보이고, pg_search가 Elasticsearch의 핵심 검색 기능을 PostgreSQL 안에서 제공하는 지금, 단일 PostgreSQL 전략이 현실적 선택지가 된 것은 분명하다. 모든 팀에 맞는 아키텍처는 없지만, 충분히 커지기 전까지는 단순한 것이 강하다는 원칙은 유효하다.

전문 DB가 진짜로 필요해지는 임계값에 도달하면 — 그 시점에 분리해도 전혀 늦지 않는다.

---

*참고 자료: Timescale pgvector vs Pinecone Benchmark (2024), ParadeDB pg_search 공식 문서, Neon — Postgres FTS vs Elasticsearch, Tiger Data pg_textsearch GitHub (2025)*
