---
title: "PostgreSQL이 Elasticsearch를 대체할 수 있는가: BM25 전문 검색의 패러다임 전환"
date: 2026-01-29 01:00:00 +0900
categories: [TechStack,  Backend Architecture]
mermaid: [True]
tags: [PostgreSQL,  Elasticsearch,  BM25,  Claude.write]
---


## 관련영상

[**Postgres Just Killed Elasticsearch**](https://www.youtube.com/watch?v=XEiQV4zRC-U)

## 서론: 검색 엔진 아키텍처의 근본적인 재고찰

현대 애플리케이션 개발에서 검색 기능은 더 이상 선택 사항이 아니라 필수 요구사항이 되었다. 사용자들은 구글과 같은 정교한 검색 경험에 익숙해졌으며, 이제 모든 애플리케이션에서 동일한 수준의 검색 품질을 기대한다. 이러한 요구를 충족하기 위해 많은 조직들은 PostgreSQL과 같은 관계형 데이터베이스와 함께 Elasticsearch와 같은 전용 검색 엔진을 별도로 운영하는 이중 데이터 저장소 아키텍처를 채택해 왔다.

그러나 최근 PostgreSQL 생태계에서 일어난 혁신적인 변화는 이러한 전통적인 아키텍처 패턴에 근본적인 의문을 제기하고 있다. 특히 BM25 알고리즘을 PostgreSQL 내부에 직접 구현한 여러 오픈소스 확장 기능들의 등장은 "데이터베이스와 검색 엔진을 분리해야 한다"는 기존의 통념을 뒤집고 있다. 2024년 12월 Tiger Data가 pg_textsearch를 오픈소스로 공개한 것을 시작으로, ParadeDB의 pg_search, VectorChord의 BM25 구현 등 여러 프로젝트들이 PostgreSQL 단일 데이터베이스 내에서 Elasticsearch 수준의 검색 품질을 제공할 수 있음을 입증하고 있다.

이 문서는 PostgreSQL의 BM25 전문 검색 기능이 어떻게 검색 엔진 시장의 판도를 바꾸고 있는지 심층적으로 분석한다. 단순히 새로운 기술을 소개하는 수준을 넘어, 이 변화가 시스템 아키텍처, 운영 복잡성, 비용 구조, 그리고 개발 생산성에 미치는 영향을 종합적으로 검토한다. 또한 기존 Elasticsearch 기반 아키텍처와의 상세한 비교를 통해 각 접근 방식의 장단점을 객관적으로 평가하고, 실무자들이 자신의 상황에 맞는 합리적인 의사결정을 내릴 수 있도록 실용적인 가이드를 제공한다.

## 검색 기술의 진화: 단순 매칭에서 확률론적 랭킹까지

검색 기술의 역사는 정보 검색(Information Retrieval)의 정확도와 효율성을 개선하기 위한 끊임없는 노력의 과정이었다. 초기의 검색 시스템들은 단순한 키워드 매칭에 의존했다. 사용자가 입력한 검색어가 문서에 포함되어 있는지를 확인하는 것이 전부였으며, 여러 문서가 매칭될 경우 어떤 문서가 더 관련성이 높은지를 판단하는 정교한 메커니즘은 존재하지 않았다.

이러한 한계를 극복하기 위해 1970년대부터 1980년대에 걸쳐 TF-IDF(Term Frequency-Inverse Document Frequency) 알고리즘이 개발되었다. TF-IDF는 검색 결과의 순위를 매기기 위한 최초의 체계적인 방법론으로, 두 가지 직관적인 원칙에 기반한다. 첫째, 특정 검색어가 문서 내에서 자주 등장할수록 해당 문서는 그 검색어와 더 관련성이 높을 것이다(Term Frequency). 둘째, 전체 문서 집합에서 드물게 등장하는 검색어일수록 더 중요한 의미를 갖는다(Inverse Document Frequency). 예를 들어 "the"나 "is"와 같은 흔한 단어는 문서의 관련성을 판단하는 데 거의 도움이 되지 않지만, "양자역학"이나 "블록체인"과 같은 특수한 용어는 문서의 주제를 판단하는 데 매우 중요한 단서가 된다.

TF-IDF는 많은 성공을 거두었지만, 실제 환경에서 적용하면서 몇 가지 근본적인 한계가 드러났다. 가장 큰 문제는 용어 빈도(Term Frequency)가 무한정 증가한다는 점이다. 어떤 문서에서 특정 단어가 10번 등장하는 것과 100번 등장하는 것 사이에 TF-IDF는 선형적으로 증가하는 점수를 부여한다. 그러나 실제로는 특정 단어가 일정 횟수 이상 등장하면 추가적인 등장이 관련성을 크게 증가시키지 않는다. 두 번째 문제는 문서 길이에 대한 고려가 부족하다는 점이다. 긴 문서는 단순히 더 많은 단어를 포함할 수 있는 공간이 있기 때문에 짧은 문서에 비해 불공정한 이점을 갖게 된다. 짧은 블로그 포스트에서 특정 단어가 한 번 등장하는 것과 100페이지짜리 학술 논문에서 한 번 등장하는 것은 그 의미가 완전히 다름에도 불구하고, TF-IDF는 이를 제대로 구분하지 못한다.

### BM25의 탄생과 확률론적 접근

1990년대 초반, 영국 런던 City University의 Okapi 정보 검색 시스템 연구팀은 이러한 TF-IDF의 한계를 극복하기 위한 새로운 접근 방식을 개발했다. Stephen E. Robertson, Karen Spärck Jones 등의 연구자들은 정보 검색을 확률론적 관점에서 재정의했다. 그들의 핵심 통찰은 검색이 단순히 유사도를 계산하는 문제가 아니라, 주어진 검색어에 대해 특정 문서가 관련성이 있을 확률을 추정하는 문제로 봐야 한다는 것이었다.

이러한 확률론적 프레임워크에서 개발된 알고리즘이 바로 BM25(Best Matching 25)이다. 여기서 "25"라는 숫자는 실제로는 알고리즘이 25번의 반복을 거쳐 최종 형태에 도달했음을 의미한다. BM25는 TF-IDF의 기본 아이디어를 유지하면서도, 두 가지 핵심적인 개선을 도입했다.

첫 번째는 용어 빈도 포화(Term Frequency Saturation)이다. BM25는 용어 빈도가 증가함에 따라 관련성 점수가 점진적으로 수렴하도록 하는 포화 함수를 사용한다. 이는 k₁이라는 하이퍼파라미터로 제어되며, 일반적으로 1.2에서 2.0 사이의 값이 사용된다. 이 메커니즘 덕분에 특정 단어가 문서에서 처음 몇 번 등장할 때는 점수가 빠르게 증가하지만, 이후에는 추가 등장이 점수에 미치는 영향이 점차 감소한다. 예를 들어 "머신러닝"이라는 단어가 1번에서 2번으로 증가할 때의 점수 상승폭은 크지만, 50번에서 51번으로 증가할 때의 점수 상승폭은 거의 무시할 수 있을 정도로 작아진다.

두 번째는 문서 길이 정규화(Document Length Normalization)이다. BM25는 각 문서의 길이를 전체 문서 집합의 평균 길이와 비교하여 점수를 조정한다. 평균보다 긴 문서는 점수에 페널티를 받고, 평균보다 짧은 문서는 보너스를 받는다. 이 조정의 강도는 b라는 하이퍼파라미터로 제어되며, 일반적으로 0.75 정도의 값이 사용된다. b=0이면 문서 길이를 전혀 고려하지 않고, b=1이면 완전히 정규화한다. 이러한 정규화 덕분에 짧은 문서에서 핵심 키워드가 집중적으로 등장하는 경우 그 문서가 더 관련성 높은 것으로 평가될 수 있다.

BM25의 수학적 형태를 살펴보면 다음과 같다. 검색어 Q와 문서 D에 대한 BM25 점수는 검색어를 구성하는 각 용어 qᵢ에 대해 다음 공식으로 계산된 값들의 합이다:

```
score(D, Q) = Σ IDF(qᵢ) × [f(qᵢ, D) × (k₁ + 1)] / [f(qᵢ, D) + k₁ × (1 - b + b × |D| / avgdl)]
```

여기서:
- IDF(qᵢ)는 용어 qᵢ의 역문서 빈도로, log[(N - df + 0.5) / (df + 0.5)]로 계산된다
- f(qᵢ, D)는 문서 D에서 용어 qᵢ가 등장하는 횟수
- |D|는 문서 D의 길이(단어 수)
- avgdl은 전체 문서 집합의 평균 문서 길이
- k₁과 b는 조정 가능한 하이퍼파라미터

이 공식을 자세히 들여다보면, 분자 부분인 f(qᵢ, D) × (k₁ + 1)은 용어 빈도에 비례하여 증가하지만, 분모 부분인 f(qᵢ, D) + k₁ × (1 - b + b × |D| / avgdl) 때문에 그 증가율이 점차 둔화된다. 특히 문서가 평균보다 길 경우 분모가 더 커지므로 전체 점수가 하향 조정되는 것을 볼 수 있다.

BM25는 학술 연구에서 벗어나 실제 검색 엔진에서 빠르게 채택되었다. Elasticsearch의 기반이 되는 Apache Lucene은 2016년 버전 6부터 기본 랭킹 함수로 BM25를 채택했으며, 이후 Google, Bing을 비롯한 주요 검색 엔진들도 BM25 또는 그 변형을 사용하고 있다. BM25의 성공은 단순히 이론적으로 우수하다는 점을 넘어서, 실제 환경에서 다양한 유형의 검색 쿼리에 대해 일관되게 높은 품질의 결과를 제공한다는 경험적 증거에 기반한다.

## PostgreSQL 전문 검색의 역사적 한계와 새로운 가능성

PostgreSQL은 오랫동안 강력한 전문 검색(Full-Text Search) 기능을 제공해 왔다. 기본적으로 제공되는 tsvector와 tsquery 데이터 타입, 그리고 GIN(Generalized Inverted Index) 인덱스는 많은 애플리케이션에서 충분히 훌륭한 검색 기능을 제공했다. 개발자들은 별도의 인프라 없이 SQL 쿼리만으로 전문 검색을 구현할 수 있었으며, 트랜잭션과 일관성이 보장되는 환경에서 검색 기능을 사용할 수 있다는 점은 큰 장점이었다.

그러나 PostgreSQL의 기본 전문 검색은 몇 가지 근본적인 한계를 가지고 있었다. 가장 큰 문제는 랭킹 품질이었다. PostgreSQL은 ts_rank와 ts_rank_cd라는 두 가지 랭킹 함수를 제공하는데, 이들은 본질적으로 단순화된 TF-IDF 변형에 가깝다. ts_rank는 문서 내에서 검색어가 등장하는 빈도와 위치를 고려하지만, 전체 코퍼스(corpus)에 걸친 역문서 빈도나 문서 길이 정규화를 제대로 처리하지 못한다. 결과적으로 긴 문서가 불공정하게 높은 순위를 받거나, 흔한 단어가 과도하게 가중치를 받는 등의 문제가 발생한다.

두 번째 문제는 성능이었다. 수백만 행 이상의 대규모 데이터셋에서 복잡한 전문 검색 쿼리를 실행하면 응답 시간이 수 분까지 늘어나는 경우가 빈번했다. GIN 인덱스가 있더라도 대규모 결과 집합에 대한 랭킹 계산은 여전히 느렸다. 이는 Elasticsearch가 수십억 개의 문서에서 밀리초 단위로 결과를 반환하는 것과 극명한 대조를 이뤘다.

세 번째는 기능의 제한이었다. Elasticsearch가 제공하는 퍼지 매칭(fuzzy matching), 구문 근접성 검색(phrase proximity), 필드별 부스팅(field boosting), 복잡한 집계(aggregation) 기능 등은 PostgreSQL의 기본 전문 검색으로는 구현하기 어렵거나 불가능했다. 이러한 고급 기능들은 현대 검색 경험에서 필수적인 요소들이다.

이러한 한계들 때문에 많은 조직들은 PostgreSQL을 기본 데이터 저장소로 사용하면서도, 검색 기능을 위해서는 Elasticsearch를 별도로 운영하는 이중 데이터 저장소 아키텍처를 채택할 수밖에 없었다. 그러나 이 아키텍처는 새로운 복잡성을 도입한다. 데이터를 PostgreSQL에서 Elasticsearch로 지속적으로 동기화해야 하며, 이 과정에서 데이터 일관성 문제, ETL 파이프라인 유지보수, 인프라 비용 증가 등 다양한 문제들이 발생한다.

### 패러다임 전환: BM25의 PostgreSQL 통합

2023년부터 2024년에 걸쳐 PostgreSQL 생태계에서 일어난 가장 중요한 변화는 BM25 알고리즘을 PostgreSQL 내부에 직접 구현한 여러 오픈소스 확장 기능들의 등장이다. 이들은 단순히 기존 PostgreSQL 전문 검색을 개선하는 수준을 넘어, Elasticsearch와 동등한 수준의 검색 품질을 PostgreSQL 단독으로 제공할 수 있음을 입증하고 있다.

현재 주요한 PostgreSQL BM25 구현은 세 가지로 정리할 수 있다. 각각은 서로 다른 설계 철학과 구현 방식을 채택하고 있지만, 모두 BM25 알고리즘을 PostgreSQL 내부에서 네이티브하게 실행한다는 공통점을 가진다.

첫 번째는 **pg_search**(이전 이름 pg_bm25)로, ParadeDB 프로젝트에서 개발했다. pg_search는 Rust 언어로 작성되었으며, pgrx 프레임워크를 활용하여 PostgreSQL 확장으로 컴파일된다. 내부적으로는 Rust 기반의 검색 라이브러리인 Tantivy를 사용한다. Tantivy는 Apache Lucene의 Rust 대안으로 개발되었으며, Elasticsearch가 사용하는 것과 동일한 역색인(inverted index) 구조를 제공한다. pg_search의 가장 큰 특징은 Elasticsearch의 Query DSL과 유사한 쿼리 인터페이스를 제공한다는 점이다. 이를 통해 Elasticsearch 사용자들이 친숙하게 느낄 수 있는 쿼리 문법을 SQL 내에서 사용할 수 있다.

pg_search는 새로운 인덱스 타입인 BM25 인덱스를 도입한다. 이 인덱스는 PostgreSQL의 Index Access Method API를 활용하여 구현되었으며, PostgreSQL이 일반적인 B-tree나 GIN 인덱스를 관리하는 것과 동일한 방식으로 관리된다. 데이터가 삽입, 수정, 삭제될 때 BM25 인덱스가 자동으로 업데이트되므로, 별도의 재색인 작업이 필요 없다. 이는 Elasticsearch와 달리 실시간 일관성(real-time consistency)을 보장한다는 의미이다.

pg_search는 또한 @@@라는 특수한 연산자를 도입하여 BM25 검색을 수행한다. 이는 PostgreSQL의 기본 전문 검색 연산자인 @@와 유사하지만, BM25 인덱스를 활용한다는 점에서 차별화된다. 예를 들어 다음과 같은 쿼리가 가능하다:

```sql
SELECT * FROM articles 
WHERE articles @@@ ('bm25_idx', 'machine learning algorithms')
ORDER BY articles @@@ ('bm25_idx', 'machine learning algorithms') DESC
LIMIT 10;
```

ParadeDB는 pg_search를 단순한 검색 확장을 넘어 "Elasticsearch alternative built on Postgres"로 포지셔닝하고 있다. 이들은 검색뿐만 아니라 분석(analytics) 기능도 통합하여 PostgreSQL을 완전한 검색 및 분석 플랫폼으로 변모시키는 것을 목표로 하고 있다.

두 번째는 **pg_textsearch**로, Tiger Data(이전 Timescale)에서 개발했다. 2024년 12월 PostgreSQL License 하에 오픈소스로 공개된 pg_textsearch는 메모리 테이블(memtable) 아키텍처를 사용하여 효율적인 인덱싱과 랭킹을 제공한다. pg_textsearch는 Tiger Cloud에서 먼저 상용 서비스로 제공되었으며, 실제 프로덕션 환경에서 검증된 후 오픈소스로 전환되었다.

pg_textsearch의 핵심 설계 철학은 단순성과 성능의 균형이다. pg_search가 Elasticsearch와 유사한 풍부한 쿼리 DSL을 제공하는 것과 달리, pg_textsearch는 PostgreSQL의 기존 전문 검색 문법과 최대한 유사하게 설계되었다. 기존 PostgreSQL 전문 검색 사용자들이 최소한의 학습 곡선으로 BM25의 이점을 누릴 수 있도록 한 것이다. 인덱스 생성도 간단하다:

```sql
CREATE EXTENSION IF NOT EXISTS pg_textsearch CASCADE;
CREATE INDEX articles_idx ON articles USING bm25(content) 
WITH (text_config='english');
```

pg_textsearch는 코퍼스 인식 랭킹(corpus-aware ranking)을 강조한다. 이는 BM25의 IDF 컴포넌트가 전체 문서 집합의 통계를 실시간으로 반영한다는 의미이다. 새로운 문서가 삽입되거나 기존 문서가 삭제되면 전체 코퍼스의 통계가 변경되고, 이는 즉시 검색 결과의 랭킹에 반영된다. 또한 병렬 인덱스 빌드(parallel index build)를 지원하여 대규모 테이블에서도 빠른 인덱스 생성이 가능하다.

Tiger Data는 pg_textsearch를 pgvector 및 pgvectorscale와 결합하여 "Postgres Search Stack"을 구성하는 것을 권장한다. 이를 통해 키워드 검색과 시맨틱 검색을 단일 데이터베이스 내에서 통합할 수 있다.

세 번째는 **VectorChord-BM25**로, VectorChord 프로젝트의 일부로 개발되었다. VectorChord는 원래 pgvector의 성능을 개선하기 위한 벡터 검색 확장으로 시작했지만, 최근 BM25 구현을 추가하여 하이브리드 검색을 지원한다. VectorChord-BM25의 독특한 점은 BM25 점수를 전용 데이터 타입으로 관리하고, 최적화된 인덱싱 전략을 사용한다는 것이다.

VectorChord 팀은 PostgreSQL 기본 전문 검색의 성능 문제가 실제로는 최적화 부족 때문이라고 주장한다. 그들의 벤치마크에 따르면, GIN 인덱스에 적절한 옵션(fastupdate=off)을 설정하고 tsvector를 제대로 설정하면 기본 PostgreSQL 전문 검색도 상당히 빠를 수 있다. 그러나 랭킹 품질 면에서는 BM25가 명백히 우수하기 때문에 VectorChord-BM25를 개발했다고 설명한다.

이 세 가지 구현 외에도 여러 커뮤니티 프로젝트들이 PostgreSQL에서 BM25를 사용하려는 시도를 하고 있다. 일부는 PostgreSQL의 기존 함수와 통계 기능을 조합하여 순수 SQL로 BM25를 구현하려는 시도도 있지만, 성능과 편의성 면에서 전용 확장 기능들에 비해 한계가 있다.

## Elasticsearch와 PostgreSQL: 근본적인 설계 철학의 차이

Elasticsearch와 PostgreSQL을 비교할 때 가장 먼저 이해해야 할 점은, 이 두 시스템이 근본적으로 다른 목적을 위해 설계되었다는 것이다. PostgreSQL은 ACID 트랜잭션을 보장하는 관계형 데이터베이스 관리 시스템(RDBMS)으로, 데이터의 일관성과 무결성을 최우선 가치로 한다. 반면 Elasticsearch는 대규모 텍스트 검색과 로그 분석을 위한 분산 검색 및 분석 엔진으로, 검색 성능과 확장성을 최우선 가치로 한다.

이러한 설계 철학의 차이는 두 시스템의 모든 측면에 영향을 미친다. PostgreSQL은 단일 마스터(single master) 아키텍처를 채택하여 모든 쓰기 작업이 하나의 노드에서 처리된다. 이를 통해 강력한 일관성 보장이 가능하지만, 쓰기 작업의 수평 확장에는 한계가 있다. 읽기 작업은 리플리카(read replica)를 통해 확장할 수 있다. 반면 Elasticsearch는 처음부터 샤딩(sharding)을 통한 수평 확장을 염두에 두고 설계되었다. 데이터는 여러 샤드에 분산되고, 각 샤드는 여러 노드에 복제될 수 있다. 이를 통해 매우 큰 데이터셋도 처리할 수 있지만, 그 대가로 일관성 보장이 약화된다.

네트워크 프로토콜 측면에서도 차이가 있다. PostgreSQL은 연결 지향(connection-oriented) 프로토콜을 사용한다. 클라이언트는 데이터베이스와 영구적인 연결을 맺고, 이 연결을 통해 여러 쿼리를 순차적으로 실행할 수 있다. 트랜잭션 관리, 준비된 명령문(prepared statements), 커서(cursors) 등의 고급 기능들이 이 연결 기반 모델에서 자연스럽게 지원된다. 반면 Elasticsearch는 HTTP 기반의 RESTful API를 제공한다. 각 요청은 독립적이며, 상태를 유지하지 않는다(stateless). 이는 로드 밸런싱과 확장성 측면에서 유리하지만, 트랜잭션과 같은 기능은 본질적으로 제한된다.

데이터 모델도 근본적으로 다르다. PostgreSQL은 엄격한 스키마를 요구하는 관계형 모델을 사용한다. 테이블은 사전에 정의된 컬럼들로 구성되며, 각 컬럼은 명시적인 데이터 타입을 가진다. 외래 키 제약, 유니크 제약 등을 통해 데이터 무결성을 강제할 수 있다. JSON과 JSONB 타입의 도입으로 반구조화(semi-structured) 데이터도 저장할 수 있지만, 기본적으로는 구조화된 데이터를 위해 설계되었다. Elasticsearch는 스키마가 없는(schema-free) 문서 저장소이다. JSON 문서를 그대로 인덱싱하며, 스키마는 동적으로 추론된다. 필드를 추가하거나 변경하는 것이 매우 자유롭다. 이는 로그 분석과 같이 스키마가 자주 변경되는 환경에서 유리하지만, 데이터 무결성 보장은 애플리케이션 레벨에서 처리해야 한다.

### 트랜잭션과 일관성 보장

PostgreSQL의 가장 강력한 특징 중 하나는 완전한 ACID 트랜잭션 지원이다. 애플리케이션은 여러 테이블에 걸친 복잡한 데이터 변경을 하나의 원자적 단위로 묶을 수 있다. 트랜잭션이 커밋되기 전까지는 변경사항이 다른 세션에 보이지 않으며, 롤백하면 모든 변경이 취소된다. MVCC(Multi-Version Concurrency Control)를 통해 읽기 작업이 쓰기 작업을 블록하지 않으면서도 일관된 스냅샷을 제공한다. 이는 금융 거래, 재고 관리 등 데이터 정확성이 중요한 애플리케이션에서 필수적이다.

Elasticsearch는 근본적으로 트랜잭션을 지원하지 않는다. 각 문서의 색인(indexing) 작업은 개별적으로 처리되며, 여러 문서에 걸친 원자적 업데이트는 불가능하다. 낙관적 동시성 제어(optimistic concurrency control)를 위한 버전 번호(version number)를 제공하지만, 이는 PostgreSQL의 트랜잭션과는 비교할 수 없을 정도로 제한적이다. 더 중요한 것은 Elasticsearch가 "near real-time" 검색을 제공한다는 점이다. 문서가 색인되더라도 즉시 검색 가능해지는 것이 아니라, 일정 시간(기본적으로 1초)의 지연이 있다. 이는 성능을 위한 의도적인 설계 선택이지만, 실시간 일관성이 필요한 애플리케이션에서는 문제가 될 수 있다.

예를 들어 전자상거래 애플리케이션을 생각해보자. 사용자가 주문을 하면 재고가 감소하고, 주문 기록이 생성되며, 결제가 처리되어야 한다. PostgreSQL에서는 이 모든 작업을 하나의 트랜잭션으로 묶을 수 있다. 중간에 오류가 발생하면 모든 것이 롤백되어 데이터 무결성이 보장된다. 그러나 Elasticsearch를 검색 기능에 사용한다면, 주문 데이터를 Elasticsearch에 동기화하는 과정에서 불일치가 발생할 수 있다. PostgreSQL에는 주문이 기록되었지만 Elasticsearch에는 아직 반영되지 않아 검색 결과에 나타나지 않거나, 반대로 Elasticsearch에는 있지만 PostgreSQL에서는 롤백된 주문이 검색될 수 있다.

### 검색 기능과 성능 특성

검색 기능 측면에서 Elasticsearch는 명백한 우위를 가지고 있었다. Elasticsearch Query DSL은 매우 표현력이 풍부하여 거의 모든 종류의 검색 요구사항을 표현할 수 있다. 불린 쿼리(boolean query)를 통해 복잡한 조건을 조합할 수 있고, 퍼지 매칭을 통해 오타를 허용할 수 있으며, 구문 쿼리(phrase query)를 통해 단어의 순서와 근접성을 고려할 수 있다. 필드별로 다른 가중치를 부여하는 부스팅(boosting)도 자유롭게 설정할 수 있다. 예를 들어 제목 필드의 매칭을 본문 필드보다 더 중요하게 취급할 수 있다.

집계(aggregation) 기능도 강력하다. Elasticsearch는 검색 결과에 대해 다양한 통계를 실시간으로 계산할 수 있다. 가격 범위별 제품 수, 카테고리별 분포, 시간대별 트렌드 등을 빠르게 계산하여 패싯 검색(faceted search)을 구현할 수 있다. 이는 전자상거래 사이트의 필터링 기능에서 흔히 볼 수 있는 "가격: $0-$50 (123개), $50-$100 (456개)" 같은 UI를 구현하는 데 필수적이다.

하이라이팅(highlighting) 기능도 기본 제공된다. 검색 결과에서 매칭된 부분을 자동으로 표시하여 사용자가 왜 해당 문서가 검색되었는지 쉽게 이해할 수 있게 한다. 자동완성(autocomplete), "More Like This" 추천, 제안(suggestions) 등의 고급 기능들도 기본 제공된다.

성능 측면에서도 Elasticsearch는 설계상 유리하다. 역색인(inverted index) 구조는 검색에 최적화되어 있으며, 샤딩을 통해 검색 부하를 여러 노드에 분산할 수 있다. 벤치마크 결과들은 일관되게 Elasticsearch가 수십억 개의 문서에서도 밀리초 단위의 응답 시간을 제공할 수 있음을 보여준다. 대규모 로그 분석, 실시간 모니터링 등의 시나리오에서 Elasticsearch의 성능은 거의 대안이 없다.

그러나 이제 PostgreSQL의 BM25 확장들이 등장하면서 상황이 변하고 있다. pg_search는 Tantivy 기반의 역색인을 사용하여 Elasticsearch와 동일한 데이터 구조를 PostgreSQL 내부에서 구현한다. ParadeDB의 벤치마크에 따르면, 100만 행의 데이터셋에서 pg_search는 기본 PostgreSQL tsvector보다 50초 빠르게 인덱스를 생성하고, 랭킹은 20배 빠르다. Elasticsearch와 직접 비교한 벤치마크에서도 pg_search는 많은 쿼리에서 비슷하거나 더 나은 성능을 보였다.

물론 여기에는 한계가 있다. 단일 PostgreSQL 서버는 물리적 리소스의 제한을 받는다. Elasticsearch처럼 수십 대의 노드에 걸쳐 샤딩하여 페타바이트 규모의 데이터를 처리하는 것은 PostgreSQL의 설계 목표가 아니다. 그러나 실제로 대부분의 애플리케이션은 그 정도 규모를 필요로 하지 않는다. 수백만에서 수천만 건의 문서를 검색하는 것이 일반적인 요구사항이며, 이 범위에서는 PostgreSQL BM25 확장이 충분히 경쟁력 있는 성능을 제공한다.

### 운영 복잡성과 비용 구조

Elasticsearch를 도입하기로 결정하면 단순히 검색 기능을 추가하는 것 이상의 의미를 갖는다. 이는 전체 인프라에 새로운 컴포넌트를 추가하는 것이며, 이에 따른 운영 오버헤드가 발생한다. Elasticsearch 클러스터를 안정적으로 운영하려면 상당한 전문 지식이 필요하다. 샤드 할당, 리플리카 관리, 힙 메모리 튜닝, GC 최적화 등 고려해야 할 요소들이 많다.

특히 데이터 동기화는 지속적인 운영 부담이다. PostgreSQL과 Elasticsearch 사이에 데이터를 동기화하는 방법은 크게 두 가지다. 첫 번째는 애플리케이션 레벨에서 양쪽에 모두 쓰기를 수행하는 것이다. 이는 구현이 간단하지만, 실패 처리가 복잡해진다. PostgreSQL에는 쓰기가 성공했지만 Elasticsearch에는 실패하면 어떻게 할 것인가? 재시도 로직, 보상 트랜잭션(compensating transaction) 등을 구현해야 한다.

두 번째는 Change Data Capture(CDC)를 사용하는 것이다. PostgreSQL의 논리적 복제(logical replication)를 활용하여 변경 사항을 스트림으로 캡처하고, 이를 Elasticsearch에 반영한다. Debezium 같은 도구들이 이를 지원한다. 이 방식은 애플리케이션 코드가 간단해지지만, CDC 파이프라인 자체를 운영해야 하는 부담이 있다. Kafka 같은 메시지 큐가 중간에 들어가면 복잡성은 더욱 증가한다.

어느 방식을 선택하든 데이터 일관성 문제는 남는다. Elasticsearch는 최종 일관성(eventual consistency)을 제공한다. 데이터가 PostgreSQL에 쓰여진 후 Elasticsearch에 반영되기까지 지연이 있다. 이 지연은 수 초에서 수 분까지 다양하며, 부하 상황에 따라 변동한다. 사용자가 데이터를 업데이트한 직후 검색했을 때 변경사항이 반영되지 않았다면 혼란스러울 것이다. 이를 처리하기 위한 추가 로직이 필요하다.

스키마 변경도 문제다. PostgreSQL 테이블의 스키마를 변경하면 Elasticsearch의 매핑(mapping)도 함께 변경해야 한다. 대규모 데이터셋에서 매핑 변경은 전체 재색인(reindexing)을 필요로 할 수 있으며, 이는 수 시간에서 수 일이 걸릴 수 있다. 이 기간 동안 서비스에 영향을 주지 않으려면 제로 다운타임 재색인 전략을 구현해야 한다.

백업과 복구 전략도 복잡해진다. PostgreSQL의 백업은 잘 확립된 도구들(pg_dump, pg_basebackup, WAL archiving 등)로 처리할 수 있다. 그러나 Elasticsearch는 별도의 스냅샷 메커니즘을 가지고 있다. 두 시스템의 백업이 서로 일관된 시점을 보장하는 것은 쉽지 않다.

비용 측면에서도 Elasticsearch는 상당한 부담이다. ParadeDB가 여러 기업과 인터뷰한 결과에 따르면, Elasticsearch가 전체 소프트웨어 비용에서 가장 큰 비중을 차지하는 경우가 많았다. Elasticsearch Cloud를 사용하면 편리하지만 비용이 높고, 자체 운영으로 전환하면 클라우드 비용은 줄어들지만 인건비가 증가한다. Elasticsearch를 안정적으로 운영할 수 있는 엔지니어를 고용하고 유지하는 비용이 만만치 않다.

반면 PostgreSQL BM25 확장을 사용하면 이러한 복잡성이 대부분 사라진다. 데이터는 하나의 시스템에만 존재한다. 동기화 문제가 없고, 데이터 일관성은 PostgreSQL의 트랜잭션으로 보장된다. 백업과 복구는 기존 PostgreSQL 도구를 그대로 사용할 수 있다. 개발 환경과 CI/CD 파이프라인도 간단해진다. 개발자들이 학습해야 할 기술도 하나 줄어든다.

물론 PostgreSQL도 적절한 튜닝과 운영이 필요하다. 그러나 대부분의 조직은 이미 PostgreSQL을 운영하고 있으므로, 추가적인 학습 곡선이 크지 않다. Elasticsearch라는 완전히 새로운 시스템을 도입하는 것과는 비교할 수 없다.

## 하이브리드 검색: 키워드와 시맨틱의 융합

최근 AI와 대규모 언어 모델(LLM)의 발전으로 검색 패러다임에 또 다른 변화가 일어나고 있다. 전통적인 키워드 기반 검색만으로는 사용자의 의도를 완전히 파악하기 어렵다는 인식이 확산되면서, 시맨틱 검색(semantic search) 또는 벡터 검색(vector search)이 주목받고 있다. 시맨틱 검색은 단어의 표면적 매칭을 넘어서 의미를 이해한다. 예를 들어 "강아지"를 검색했을 때 "개", "반려동물", "애완견"을 포함하는 문서도 찾아낼 수 있다.

시맨틱 검색은 임베딩(embedding)을 기반으로 한다. 텍스트를 고차원 벡터 공간에 매핑하고, 벡터 간의 거리나 유사도를 계산하여 관련성을 판단한다. BERT, Sentence Transformers, OpenAI의 임베딩 모델 등이 이러한 목적으로 사용된다. 이 방식은 동의어, 다의어, 문맥을 자연스럽게 처리할 수 있다는 장점이 있다.

그러나 벡터 검색만으로도 완벽하지 않다. 고유명사, 전문 용어, 정확한 날짜나 숫자 등은 키워드 매칭이 더 효과적일 수 있다. "iPhone 15 Pro"를 검색할 때 "스마트폰"이나 "애플 제품" 같은 의미적으로 유사한 것이 아니라, 정확히 "iPhone 15 Pro"가 포함된 문서를 원한다. 또한 임베딩 모델은 학습 데이터에 포함되지 않은 최신 용어나 도메인 특화 용어를 제대로 이해하지 못할 수 있다.

이러한 이유로 하이브리드 검색(hybrid search)이 등장했다. 하이브리드 검색은 키워드 검색과 시맨틱 검색을 결합하여 두 방식의 장점을 모두 취한다. 일반적으로 BM25로 계산된 키워드 점수와 벡터 유사도 점수를 결합하여 최종 랭킹을 결정한다. 결합 방식은 다양하다. 가장 간단한 방법은 두 점수의 가중 평균을 사용하는 것이다. 더 정교한 방법으로는 Reciprocal Rank Fusion(RRF)이 있는데, 각 방법에서 나온 순위를 결합하는 방식이다.

Elasticsearch는 버전 8부터 하이브리드 검색을 공식 지원한다. Elasticsearch Relevance Engine이라는 이름으로 마케팅되며, BM25 검색과 KNN(k-nearest neighbors) 벡터 검색을 결합할 수 있다. 그러나 이를 위해서는 Elasticsearch에 벡터 임베딩을 저장하고 인덱싱해야 한다. 이는 추가적인 저장 공간과 계산 비용을 의미한다.

PostgreSQL에서는 pgvector 확장이 벡터 검색을 지원한다. pgvector는 벡터 데이터 타입을 추가하고, 벡터 간의 거리 계산과 인덱싱을 제공한다. HNSW(Hierarchical Navigable Small World) 인덱스를 사용하여 대규모 벡터 집합에서도 빠른 근사 최근접 이웃 검색(Approximate Nearest Neighbor Search)이 가능하다. pgvector는 OpenAI의 임베딩, Cohere, Hugging Face 모델 등 다양한 임베딩과 호환된다.

PostgreSQL BM25 확장들은 모두 pgvector와의 통합을 강조한다. pg_search는 BM25 인덱스와 pgvector 인덱스를 동일한 테이블에 함께 생성할 수 있으며, 쿼리에서 두 점수를 결합할 수 있다. pg_textsearch도 마찬가지로 pgvector 및 pgvectorscale(pgvector의 성능 개선 버전)과 함께 사용할 수 있다. 이를 통해 단일 데이터베이스 내에서 완전한 하이브리드 검색을 구현할 수 있다.

예를 들어 다음과 같은 시나리오를 생각해보자. RAG(Retrieval-Augmented Generation) 시스템을 구축한다고 하자. 사용자의 질문에 답하기 위해 관련 문서를 검색하고, 이를 LLM의 컨텍스트로 제공하는 것이다. 사용자가 "2024년 테슬라의 전기차 판매량은?"이라고 물었다면, "2024년"과 "테슬라"는 정확한 키워드 매칭이 중요하지만, "전기차 판매량"은 의미적으로 유사한 표현들도 고려해야 한다.

PostgreSQL에서는 다음과 같이 구현할 수 있다:

```sql
-- BM25 점수와 벡터 유사도를 결합
SELECT 
    id, 
    content,
    ts_rank_cd(to_tsvector('english', content), query) AS bm25_score,
    1 - (embedding <=> query_embedding) AS vector_score,
    0.7 * (content <@> search_terms) + 0.3 * (1 - (embedding <=> query_embedding)) AS combined_score
FROM documents
WHERE content @@ to_tsquery('english', search_terms)
   OR (embedding <=> query_embedding) < 0.8
ORDER BY combined_score DESC
LIMIT 10;
```

이 쿼리는 BM25 기반 키워드 검색과 벡터 검색을 동시에 수행하고, 결과를 가중 평균으로 결합한다. 가중치(0.7과 0.3)는 애플리케이션의 특성에 따라 조정할 수 있다. 중요한 점은 이 모든 것이 단일 SQL 쿼리로, 단일 트랜잭션 내에서 실행된다는 것이다.

Elasticsearch로 동일한 것을 구현하려면 PostgreSQL에서 데이터를 읽고, 임베딩을 생성하고, Elasticsearch에 색인하고, 검색 시에는 Elasticsearch에서 결과를 가져오는 복잡한 파이프라인이 필요하다. 각 단계마다 실패 가능성이 있고, 데이터 일관성을 보장하기 어렵다.

실제로 Anthropic이 최근 발표한 "Contextual Retrieval" 기법도 BM25를 핵심 구성 요소로 사용한다. 이 기법은 각 텍스트 청크에 문맥 정보를 추가하여 임베딩의 품질을 개선하는데, 최종 검색에서는 BM25와 벡터 검색을 결합한다. Anthropic의 실험 결과 하이브리드 접근이 순수 벡터 검색보다 일관되게 더 나은 결과를 제공했다.

## 실제 구현 가이드: PostgreSQL BM25 확장 활용하기

이론적 배경을 이해했다면, 이제 실제로 PostgreSQL에서 BM25 검색을 구현하는 방법을 살펴보자. 여기서는 가장 널리 사용되는 두 가지 확장인 pg_search와 pg_textsearch를 중심으로 설명한다.

### pg_search 구현 가이드

pg_search는 ParadeDB에서 개발한 확장으로, Elasticsearch와 유사한 풍부한 쿼리 기능을 제공한다. Docker를 사용하면 가장 쉽게 시작할 수 있다:

```bash
docker run --name paradedb \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  -d paradedb/paradedb:latest
```

이렇게 하면 pg_search가 이미 설치된 PostgreSQL 인스턴스가 실행된다. 또는 기존 PostgreSQL에 pg_search를 추가 설치할 수도 있다. ParadeDB는 Debian, Ubuntu, Red Hat Enterprise Linux, macOS용 사전 빌드된 바이너리를 제공한다.

데이터베이스에 연결한 후 확장을 활성화한다:

```sql
CREATE EXTENSION pg_search;
```

테스트를 위한 샘플 데이터를 생성해보자. ParadeDB는 편리하게도 테스트 테이블을 자동 생성하는 프로시저를 제공한다:

```sql
CALL paradedb.create_bm25_test_table(
    schema_name => 'public',
    table_name => 'mock_items'
);
```

이렇게 하면 mock_items라는 테이블이 생성되며, id, description, rating, category 컬럼을 포함한다. 실제 데이터로 채워져 있어 즉시 테스트할 수 있다.

BM25 인덱스를 생성한다:

```sql
CALL paradedb.create_bm25(
    index_name => 'search_idx',
    table_name => 'mock_items',
    key_field => 'id',
    text_fields => paradedb.field('description'),
    numeric_fields => paradedb.field('rating'),
    categorical_fields => paradedb.field('category')
);
```

이 명령은 description 필드를 전문 검색 가능하게 인덱싱하고, rating과 category는 필터링에 사용할 수 있도록 인덱싱한다. key_field는 각 문서를 고유하게 식별하는 필드다.

이제 검색을 수행할 수 있다. pg_search는 @@@ 연산자를 사용한다:

```sql
SELECT description, rating, category
FROM mock_items
WHERE mock_items @@@ 'description:keyboard OR category:electronics'
ORDER BY paradedb.score(id) DESC
LIMIT 10;
```

이 쿼리는 description 필드에 "keyboard"가 포함되거나 category가 "electronics"인 문서를 찾는다. paradedb.score() 함수는 BM25 점수를 반환하며, 이를 기준으로 정렬한다.

더 복잡한 쿼리도 가능하다. 퍼지 매칭을 사용하여 오타를 허용할 수 있다:

```sql
SELECT description
FROM mock_items
WHERE mock_items @@@ 'description:keybaord~2'
LIMIT 5;
```

여기서 ~2는 편집 거리(edit distance) 2까지 허용한다는 의미다. "keybaord"라는 오타가 "keyboard"와 매칭된다.

구문 검색(phrase search)도 지원한다:

```sql
SELECT description
FROM mock_items
WHERE mock_items @@@ 'description:"wireless keyboard"'
LIMIT 5;
```

인용부호를 사용하면 단어들이 정확히 이 순서로 인접하게 등장하는 문서만 찾는다.

필드별 부스팅도 가능하다:

```sql
SELECT description, rating
FROM mock_items
WHERE mock_items @@@ 'description:laptop^2 OR category:electronics'
ORDER BY paradedb.score(id) DESC
LIMIT 10;
```

^2는 description 필드의 매칭에 2배 가중치를 준다는 의미다.

범위 쿼리와 결합할 수도 있다:

```sql
SELECT description, rating
FROM mock_items
WHERE mock_items @@@ 'description:laptop AND rating:[4 TO 5]'
ORDER BY paradedb.score(id) DESC
LIMIT 10;
```

이는 "laptop"을 포함하면서 rating이 4 이상 5 이하인 문서를 찾는다.

집계(aggregation)도 지원한다:

```sql
SELECT category, COUNT(*) as count, AVG(rating) as avg_rating
FROM mock_items
WHERE mock_items @@@ 'description:electronic'
GROUP BY category;
```

하이라이팅 기능을 사용하면 매칭된 부분을 표시할 수 있다:

```sql
SELECT paradedb.highlight(id, 'description') as highlighted_description
FROM mock_items
WHERE mock_items @@@ 'description:gaming'
LIMIT 5;
```

이 함수는 "gaming"이라는 단어 주변에 HTML 태그를 추가하여 반환한다.

### pg_textsearch 구현 가이드

pg_textsearch는 Tiger Data에서 개발했으며, PostgreSQL의 기존 전문 검색 문법과 더 가까운 인터페이스를 제공한다. Tiger Cloud를 사용하면 관리 콘솔에서 확장을 활성화할 수 있다. 자체 호스팅 환경에서는 소스에서 컴파일해야 한다:

```bash
git clone https://github.com/timescale/pg_textsearch.git
cd pg_textsearch
make
make install
```

데이터베이스에서 확장을 활성화한다:

```sql
CREATE EXTENSION IF NOT EXISTS pg_textsearch CASCADE;
```

테스트 테이블을 생성한다:

```sql
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO documents (title, content) VALUES
    ('PostgreSQL Tutorial', 'PostgreSQL is a powerful open-source relational database system'),
    ('Full-Text Search', 'BM25 is an effective ranking function for full-text search'),
    ('Vector Databases', 'pgvector enables similarity search in PostgreSQL'),
    ('Hybrid Search', 'Combining keyword and semantic search improves accuracy');
```

BM25 인덱스를 생성한다:

```sql
CREATE INDEX docs_content_idx ON documents 
USING bm25(content) 
WITH (text_config='english');
```

text_config 파라미터는 언어별 텍스트 처리 설정을 지정한다. PostgreSQL은 다양한 언어를 지원한다(english, french, german, spanish 등). 이 설정에 따라 스템ming, 스탑워드 제거 등이 처리된다.

검색을 수행한다:

```sql
SELECT id, title, content, content <@> 'postgresql search' AS bm25_score
FROM documents
ORDER BY bm25_score
LIMIT 5;
```

pg_textsearch는 <@> 연산자를 사용한다. 점수가 낮을수록 더 관련성이 높다(PostgreSQL의 거리 개념을 따름).

여러 필드를 결합하여 검색할 수도 있다. 먼저 복합 인덱스를 생성한다:

```sql
CREATE INDEX docs_combined_idx ON documents 
USING bm25((title || ' ' || content)) 
WITH (text_config='english');
```

검색 시 동일한 표현식을 사용한다:

```sql
SELECT id, title, (title || ' ' || content) <@> 'database' AS score
FROM documents
ORDER BY score
LIMIT 5;
```

필터 조건과 결합할 수 있다:

```sql
SELECT id, title, content <@> 'search' AS score
FROM documents
WHERE created_at > '2024-01-01'
ORDER BY score
LIMIT 5;
```

pg_textsearch는 병렬 인덱스 빌드를 지원한다. 대규모 테이블에서 인덱스를 생성할 때 여러 워커를 사용하여 속도를 높일 수 있다:

```sql
SET max_parallel_maintenance_workers = 4;
CREATE INDEX large_docs_idx ON large_documents 
USING bm25(content) 
WITH (text_config='english');
```

인덱스 사용 통계를 확인하여 성능을 모니터링할 수 있다:

```sql
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE indexrelid::regclass::text LIKE '%bm25%';
```

### 하이브리드 검색 구현

BM25 키워드 검색과 pgvector 시맨틱 검색을 결합하는 방법을 살펴보자. 먼저 pgvector를 설치한다:

```sql
CREATE EXTENSION vector;
```

테이블에 벡터 컬럼을 추가한다:

```sql
ALTER TABLE documents ADD COLUMN embedding vector(1536);
```

여기서 1536은 임베딩 차원 수다. OpenAI의 text-embedding-3-small 모델을 사용한다면 이 값이 맞다.

임베딩을 생성하여 저장한다. 이는 일반적으로 애플리케이션 코드에서 처리한다:

```python
import openai
import psycopg2

# OpenAI 클라이언트 초기화
client = openai.OpenAI(api_key='your-api-key')

# 데이터베이스 연결
conn = psycopg2.connect('postgresql://user:password@localhost/dbname')
cur = conn.cursor()

# 문서 가져오기
cur.execute("SELECT id, content FROM documents WHERE embedding IS NULL")
documents = cur.fetchall()

# 각 문서의 임베딩 생성
for doc_id, content in documents:
    response = client.embeddings.create(
        input=content,
        model="text-embedding-3-small"
    )
    embedding = response.data[0].embedding
    
    # 임베딩 저장
    cur.execute(
        "UPDATE documents SET embedding = %s WHERE id = %s",
        (embedding, doc_id)
    )

conn.commit()
```

벡터 인덱스를 생성한다:

```sql
CREATE INDEX ON documents 
USING hnsw (embedding vector_cosine_ops);
```

이제 하이브리드 검색을 수행할 수 있다. 사용자 쿼리를 받으면 먼저 임베딩을 생성한다:

```python
query = "how to do full-text search in postgres"
query_embedding = client.embeddings.create(
    input=query,
    model="text-embedding-3-small"
).data[0].embedding

# 하이브리드 검색 실행
cur.execute("""
    WITH bm25_results AS (
        SELECT id, content, 
               content <@> %s AS bm25_score,
               ROW_NUMBER() OVER (ORDER BY content <@> %s) AS bm25_rank
        FROM documents
        ORDER BY bm25_score
        LIMIT 20
    ),
    vector_results AS (
        SELECT id, content,
               1 - (embedding <=> %s::vector) AS vector_score,
               ROW_NUMBER() OVER (ORDER BY embedding <=> %s::vector) AS vector_rank
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT 20
    )
    SELECT 
        COALESCE(b.id, v.id) AS id,
        COALESCE(b.content, v.content) AS content,
        COALESCE(b.bm25_score, 0) AS bm25_score,
        COALESCE(v.vector_score, 0) AS vector_score,
        COALESCE(1.0 / (60 + b.bm25_rank), 0.0) + 
        COALESCE(1.0 / (60 + v.vector_rank), 0.0) AS combined_score
    FROM bm25_results b
    FULL OUTER JOIN vector_results v ON b.id = v.id
    ORDER BY combined_score DESC
    LIMIT 10
""", (query, query, query_embedding, query_embedding, query_embedding))

results = cur.fetchall()
```

이 쿼리는 Reciprocal Rank Fusion(RRF) 알고리즘을 사용한다. BM25와 벡터 검색 각각에서 상위 20개 결과를 가져오고, 순위를 기반으로 점수를 결합한다. RRF 공식은 1/(k + rank)인데, 여기서 k는 일반적으로 60이 사용된다.

더 간단한 가중 평균 방식도 가능하다:

```sql
SELECT id, content,
       0.7 * (1 - (content <@> 'search query')) + 
       0.3 * (1 - (embedding <=> query_embedding::vector)) AS combined_score
FROM documents
WHERE content @@ to_tsquery('english', 'search & query')
   OR (embedding <=> query_embedding::vector) < 0.5
ORDER BY combined_score DESC
LIMIT 10;
```

### 성능 최적화 팁

BM25 검색의 성능을 최적화하기 위한 몇 가지 팁이 있다.

첫째, 선택적 필터를 먼저 적용한다. 전체 데이터의 10% 미만만 매칭되는 조건이 있다면, 이를 먼저 적용하고 그 결과에 대해 BM25 스코어링을 수행하는 것이 효율적이다:

```sql
SELECT id, content, content <@> 'query' AS score
FROM documents
WHERE category = 'technology'  -- 선택적 필터
  AND content <@> 'query' IS NOT NULL
ORDER BY score
LIMIT 10;
```

둘째, 점수 임계값을 사용하여 관련성이 낮은 결과를 제거한다:

```sql
SELECT id, content, content <@> 'query' AS score
FROM documents
WHERE content <@> 'query' < 0.5  -- 점수 임계값
ORDER BY score
LIMIT 10;
```

셋째, 파티셔닝된 테이블에서는 각 파티션이 독립적으로 인덱스를 가진다. 시간 기반 파티셔닝을 사용하면 최근 데이터만 검색할 때 성능이 향상된다:

```sql
CREATE TABLE documents_partitioned (
    id SERIAL,
    content TEXT,
    created_at DATE
) PARTITION BY RANGE (created_at);

CREATE TABLE documents_2024 PARTITION OF documents_partitioned
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE INDEX ON documents_2024 USING bm25(content);
```

넷째, 인덱스 메모리 사용량을 모니터링한다. pg_textsearch는 인메모리 멤테이블을 사용하므로, 메모리 사용량을 확인해야 한다:

```sql
SELECT schemaname, tablename, indexname,
       pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE indexrelid::regclass::text LIKE '%bm25%';
```

다섯째, EXPLAIN ANALYZE를 사용하여 쿼리 플랜을 분석한다:

```sql
EXPLAIN ANALYZE
SELECT id, content, content <@> 'query' AS score
FROM documents
ORDER BY score
LIMIT 10;
```

이를 통해 인덱스가 제대로 사용되는지, 예상치 못한 순차 스캔이 발생하는지 확인할 수 있다.

## 아키텍처 설계 결정: 언제 PostgreSQL만으로 충분한가

PostgreSQL BM25 확장이 기술적으로 가능하다는 것과 실제로 채택해야 한다는 것은 별개의 문제다. 언제 PostgreSQL만으로 충분하고, 언제 Elasticsearch가 필요한지 판단하는 기준을 살펴보자.

### PostgreSQL BM25가 적합한 경우

첫째, 데이터 규모가 단일 PostgreSQL 서버로 처리 가능한 범위라면 PostgreSQL BM25가 적합하다. 일반적으로 수백만에서 수천만 건의 문서는 적절히 튜닝된 PostgreSQL에서 충분히 빠르게 검색할 수 있다. 최신 서버 하드웨어(NVMe SSD, 충분한 RAM)를 사용한다면 이 범위는 더 넓어진다.

둘째, 실시간 일관성이 중요한 애플리케이션이라면 PostgreSQL이 유리하다. 사용자가 데이터를 생성하거나 수정한 직후 즉시 검색 결과에 반영되어야 한다면, PostgreSQL의 트랜잭션 보장이 큰 장점이다. Elasticsearch의 "near real-time" 지연은 받아들이기 어려울 수 있다.

셋째, 복잡한 트랜잭션 로직과 검색이 결합되어 있다면 PostgreSQL이 적합하다. 예를 들어 전자상거래 플랫폼에서 재고 확인, 결제 처리, 주문 생성을 하나의 트랜잭션으로 묶으면서 동시에 제품 검색 결과를 업데이트해야 한다면, 단일 데이터베이스 내에서 모든 것을 처리하는 것이 훨씬 간단하다.

넷째, 인프라를 단순하게 유지하고 싶다면 PostgreSQL이 답이다. 특히 스타트업이나 소규모 팀에서는 관리해야 할 시스템 수를 최소화하는 것이 중요하다. 이미 PostgreSQL을 운영하고 있다면, 검색 기능을 위해 완전히 새로운 시스템을 도입하는 부담을 피할 수 있다.

다섯째, 비용 민감도가 높다면 PostgreSQL이 유리하다. Elasticsearch Cloud는 비용이 높으며, 자체 운영은 전문 인력이 필요하다. PostgreSQL은 이미 가지고 있는 리소스로 검색 기능을 추가할 수 있다.

여섯째, 관계형 쿼리와 검색을 자주 결합한다면 PostgreSQL이 적합하다. JOIN, 집계, 윈도우 함수 등 SQL의 강력한 기능을 검색과 함께 사용할 수 있다. Elasticsearch에서는 이러한 관계형 작업이 제한적이거나 불가능하다.

### Elasticsearch가 여전히 필요한 경우

반대로 Elasticsearch가 더 적합한 경우도 있다.

첫째, 데이터 규모가 페타바이트급이거나 초당 수십만 건의 로그를 처리해야 한다면 Elasticsearch의 분산 아키텍처가 필요하다. 단일 PostgreSQL 서버로는 이러한 규모를 처리하기 어렵다.

둘째, 복잡한 집계와 분석이 핵심 요구사항이라면 Elasticsearch가 유리하다. 특히 로그 분석, 모니터링, 비즈니스 인텔리전스 같은 시나리오에서 Elasticsearch의 집계 프레임워크는 매우 강력하다. Kibana와 같은 시각화 도구와의 통합도 잘 되어 있다.

셋째, 여러 언어, 여러 필드에 걸친 매우 복잡한 검색 요구사항이 있다면 Elasticsearch Query DSL이 더 표현력이 풍부하다. 퍼지 매칭, 구문 근접성, 필드별 다른 분석기 적용 등의 고급 기능을 광범위하게 사용한다면 Elasticsearch가 적합하다.

넷째, 이미 Elasticsearch에 투자했고 잘 운영하고 있다면 굳이 변경할 필요가 없다. 마이그레이션 자체가 위험과 비용을 수반한다. 현재 시스템이 요구사항을 잘 만족시키고 있다면 유지하는 것이 합리적이다.

다섯째, Elasticsearch의 다른 기능들(예: security features, machine learning, APM)을 활용하고 있다면 Elasticsearch 생태계 내에 머무르는 것이 유리하다.

### 하이브리드 아키텍처

완전히 둘 중 하나를 선택하는 것이 아니라, 하이브리드 아키텍처도 가능하다. 예를 들어:

- 실시간 트랜잭션 데이터는 PostgreSQL에서 검색하고, 대규모 히스토리 로그는 Elasticsearch에서 분석한다.
- 주요 제품 카탈로그는 PostgreSQL BM25로 검색하고, 사용자 행동 로그 기반 추천은 Elasticsearch로 처리한다.
- 개발 및 스테이징 환경에서는 PostgreSQL만 사용하여 인프라를 단순화하고, 프로덕션에서만 필요시 Elasticsearch를 추가한다.

중요한 것은 무조건적인 선택이 아니라, 자신의 상황을 객관적으로 평가하고 각 도구의 장단점을 이해한 위에서 합리적인 결정을 내리는 것이다.

## 실제 사례 연구와 마이그레이션 경험

여러 조직들이 Elasticsearch에서 PostgreSQL로 또는 그 반대로 마이그레이션한 경험을 공유했다. 이들의 경험에서 배울 수 있는 실용적인 교훈들을 살펴보자.

### Qonto의 Elasticsearch 탈피 경험

유럽의 핀테크 스타트업 Qonto는 카드 정렬 기능을 위해 Elasticsearch를 사용하고 있었다. 사용자들이 회사 카드를 이름, 유형, 상태 등으로 필터링하고 정렬할 수 있는 기능이었다. 처음에는 Elasticsearch의 강력한 검색 기능이 필요할 것으로 생각했지만, 실제로 사용해보니 몇 가지 문제가 발생했다.

첫째, 데이터 동기화가 복잡했다. PostgreSQL에서 카드 정보가 변경될 때마다 Elasticsearch에도 반영해야 했다. 동기화 지연으로 인해 사용자가 방금 변경한 내용이 검색 결과에 즉시 나타나지 않는 문제가 발생했다.

둘째, 인덱싱 설정이 까다로웠다. 너무 관대한 인덱스 타입을 사용하면 예상치 못한 성능 저하가 발생했다. 한 번은 인덱싱 설정 오류로 인해 프로덕션 장애가 발생했다.

셋째, 동일한 데이터가 PostgreSQL과 Elasticsearch 양쪽에 존재하는 것이 비효율적이었다. 스토리지 비용도 두 배로 들었고, 백업과 복구 전략도 복잡해졌다.

Qonto 팀은 PostgreSQL만으로 이 기능을 구현할 수 있는지 검토했다. 카드 데이터는 수만 건 정도로 그리 크지 않았다. PostgreSQL의 GiST 인덱스와 trigram을 사용하여 이름 검색을 구현하고, 일반적인 B-tree 인덱스로 다른 필터링을 처리했다.

결과는 놀라웠다. Elasticsearch를 사용했을 때 쿼리당 평균 20ms가 걸렸던 것이 PostgreSQL로 전환 후 2-4ms로 80% 단축되었다. 성능이 향상된 주된 이유는 Elasticsearch로의 네트워크 홉이 사라졌기 때문이다. 데이터가 이미 PostgreSQL에 있으므로 추가 네트워크 통신이 필요 없었다.

코드 유지보수도 훨씬 간단해졌다. Ruby on Rails의 ActiveRecord를 사용하여 일관된 인터페이스로 데이터에 접근할 수 있었다. Elasticsearch 전용 코드를 제거하고, 모든 것을 표준 SQL로 통일했다.

Qonto의 경험은 중요한 교훈을 준다. Elasticsearch가 강력한 도구이긴 하지만, 모든 검색 시나리오에 필수적인 것은 아니다. 데이터 규모와 요구사항을 정확히 평가하면, 더 단순한 솔루션이 더 나은 결과를 줄 수 있다.

### Xata의 비교 분석

Xata는 Postgres 기반 플랫폼 제공 업체로, PostgreSQL과 Elasticsearch를 비교하는 상세한 벤치마크를 수행했다. 그들은 230만 개의 레시피를 포함하는 데이터셋으로 테스트했다.

PostgreSQL 기본 전문 검색(tsvector + GIN 인덱스)은 소규모 데이터셋에서는 잘 작동했지만, 수백만 건 규모에서는 쿼리 시간이 수백 밀리초로 증가했다. 특히 복잡한 쿼리에서 성능 저하가 두드러졌다.

반면 Elasticsearch는 동일한 데이터셋에서 일관되게 빠른 응답 시간을 보였다. 대부분의 쿼리가 100ms 이하로 완료되었다. BM25 랭킹도 PostgreSQL의 ts_rank보다 더 관련성 높은 결과를 제공했다.

그러나 Xata는 이것이 전체 그림이 아니라고 강조한다. Elasticsearch를 운영하는 총 비용(TCO)을 고려해야 한다. 인프라 비용, 동기화 복잡성, 데이터 일관성 문제, 개발 및 운영 오버헤드를 모두 포함하면, PostgreSQL의 성능이 조금 떨어지더라도 전체적으로는 더 경제적일 수 있다.

Xata의 결론은 "맥락이 중요하다"는 것이다. 수십억 건의 로그를 실시간 분석해야 한다면 Elasticsearch가 명백한 선택이다. 그러나 수백만 건의 사용자 생성 콘텐츠를 검색한다면 PostgreSQL로도 충분할 수 있으며, 아키텍처는 훨씬 단순해진다.

이제 pg_search나 pg_textsearch 같은 BM25 확장이 등장하면서 균형이 더욱 PostgreSQL 쪽으로 기울고 있다. Xata의 후속 벤치마크에서 pg_search는 많은 쿼리에서 Elasticsearch와 비슷하거나 더 나은 성능을 보였다.

### Neon의 pg_search 벤치마크

Neon은 서버리스 Postgres 플랫폼으로, pg_search의 성능을 테스트했다. 그들은 Hacker News 데이터셋(수백만 개의 포스트와 댓글)을 사용했다.

순수 인덱싱 속도에서 pg_search는 tsvector보다 50초 빠랐다. 더 중요한 것은 랭킹 속도인데, pg_search는 tsvector보다 20배 빠른 결과를 보였다. 이는 Tantivy의 최적화된 역색인 구조 덕분이다.

Elasticsearch와 직접 비교에서도 pg_search는 대부분의 쿼리 유형에서 비슷하거나 더 나은 성능을 보였다. 특히 복잡한 불린 쿼리와 퍼지 매칭에서 pg_search가 우수했다.

Neon은 pg_search가 "Elasticsearch의 성능을 PostgreSQL로 가져왔다"고 평가했다. 이제 개발자들은 검색 품질과 아키텍처 단순성 사이에서 타협할 필요가 없어졌다.

### ParadeDB의 엔터프라이즈 고객 사례

ParadeDB는 여러 엔터프라이즈 고객들이 Elasticsearch에서 pg_search로 마이그레이션한 사례를 공유했다(고객사 이름은 익명).

한 기업은 Elasticsearch가 전체 소프트웨어 비용에서 가장 큰 비중을 차지한다는 것을 발견했다. 클러스터를 안정적으로 운영하기 위해 전담 엔지니어를 고용해야 했으며, AWS Elasticsearch Service 비용도 월 수만 달러에 달했다. pg_search로 마이그레이션한 후 검색 관련 인프라 비용이 90% 이상 감소했다. 기존 PostgreSQL 인프라를 활용할 수 있었기 때문이다.

다른 기업은 ETL 파이프라인 유지보수가 가장 큰 고민이었다. PostgreSQL 스키마가 변경될 때마다 Elasticsearch 매핑도 업데이트해야 했고, 이 과정에서 종종 버그가 발생했다. pg_search로 전환 후 데이터가 하나의 소스에만 존재하므로 이러한 문제가 완전히 사라졌다.

실시간 일관성도 큰 이점이었다. 한 전자상거래 회사는 사용자가 제품 리뷰를 작성한 직후 그것이 검색 결과에 나타나지 않는 문제로 고객 불만을 받았다. Elasticsearch의 refresh interval 때문에 최대 수 초의 지연이 있었다. pg_search는 트랜잭션이 커밋되는 즉시 검색 가능하므로 이 문제가 해결되었다.

## 미래 전망: 단일 데이터베이스 시대로의 회귀

데이터베이스 기술의 역사를 돌아보면 흥미로운 패턴을 발견할 수 있다. 초기에는 단일 데이터베이스가 모든 것을 처리했다. 그러다 특화된 시스템들이 등장하면서 "적재적소에 맞는 도구(right tool for the job)"라는 철학 하에 폴리글랏 퍼시스턴스(polyglot persistence)가 유행했다. 관계형 데이터는 MySQL, 캐시는 Redis, 검색은 Elasticsearch, 그래프는 Neo4j 식으로 여러 전문 시스템을 조합했다.

그러나 이제 진자가 다시 반대 방향으로 움직이고 있다. PostgreSQL을 중심으로 한 "통합 데이터 플랫폼" 트렌드가 강화되고 있다. JSON 지원으로 문서 데이터베이스 기능을, pgvector로 벡터 데이터베이스 기능을, 이제 pg_search로 검색 엔진 기능을 흡수하고 있다. PostGIS는 이미 지리공간 데이터베이스로서 독립 제품과 경쟁할 만한 수준이다.

이러한 통합의 배경에는 여러 요인이 있다. 첫째, PostgreSQL의 확장성(extensibility)이다. PostgreSQL은 처음부터 확장 가능하도록 설계되었다. 새로운 데이터 타입, 인덱스 방법, 함수를 추가하는 것이 상대적으로 쉽다. pgrx 같은 프레임워크는 Rust로 안전하고 효율적인 확장을 작성할 수 있게 한다.

둘째, 클라우드 네이티브 시대의 운영 복잡성에 대한 반발이다. 여러 데이터베이스를 운영하는 것은 DevOps 오버헤드를 크게 증가시킨다. 각 시스템마다 다른 백업 전략, 모니터링 도구, 보안 설정이 필요하다. 단일 데이터베이스로 통합하면 운영이 훨씬 단순해진다.

셋째, 데이터 일관성과 트랜잭션의 중요성이 재인식되고 있다. 여러 데이터 저장소에 걸친 분산 트랜잭션은 본질적으로 어렵다. ACID 트랜잭션을 포기하고 최종 일관성으로 타협했던 것이 많은 버그와 데이터 손실의 원인이 되었다. 단일 데이터베이스 내에서는 이러한 문제가 자연스럽게 해결된다.

넷째, 하드웨어의 발전이다. 현대 서버는 테라바이트급 RAM과 수십 개의 CPU 코어를 가진다. NVMe SSD는 이전 세대 스토리지보다 수십 배 빠르다. 단일 서버의 처리 능력이 크게 증가하면서, 과거에는 분산 시스템이 필요했던 많은 워크로드를 이제 단일 데이터베이스로 처리할 수 있게 되었다.

다섯째, AI와 머신러닝의 통합이다. 벡터 임베딩을 활용한 시맨틱 검색은 이제 많은 애플리케이션에서 필수적이다. pgvector를 통해 PostgreSQL은 벡터 데이터베이스로도 기능한다. BM25 검색과 결합하면 하이브리드 검색을 단일 SQL 쿼리로 구현할 수 있다. 이는 별도의 벡터 데이터베이스와 검색 엔진을 운영하는 것에 비해 엄청난 단순화다.

물론 PostgreSQL이 모든 것을 완벽하게 대체할 수는 없다. 페타바이트급 로그 분석, 초고속 캐싱, 매우 복잡한 그래프 쿼리 등은 여전히 전문 시스템이 더 적합할 수 있다. 그러나 대부분의 애플리케이션이 실제로 필요로 하는 기능의 80-90%는 PostgreSQL만으로도 충분히 처리할 수 있다.

Tiger Data의 CTO Michael Freedman은 이를 "Postgres Search Stack"이라고 부른다. pg_textsearch로 키워드 검색을, pgvector/pgvectorscale로 시맨틱 검색을, PostgreSQL의 관계형 기능으로 복잡한 필터링과 조인을 모두 하나의 시스템에서 처리한다. 이는 검색뿐만 아니라 추천, 개인화, 분석까지 포괄하는 완전한 데이터 플랫폼이다.

ParadeDB는 한 걸음 더 나아가 PostgreSQL을 "Elasticsearch alternative"로 포지셔닝한다. 그들의 비전은 Elasticsearch가 제공하는 모든 핵심 기능을 PostgreSQL 내에서 구현하는 것이다. 검색뿐만 아니라 집계, 분석, 시각화까지 포함한다. 이미 pg_analytics라는 확장을 통해 컬럼 기반 스토리지와 분석 쿼리 최적화를 제공하고 있다.

이러한 트렌드는 단순히 기술적 변화를 넘어 소프트웨어 아키텍처에 대한 철학의 변화를 반영한다. "마이크로서비스와 폴리글랏 퍼시스턴스"에서 "모놀리식 데이터베이스와 분산 컴퓨트"로 전환되고 있다. 데이터는 하나의 신뢰할 수 있는 소스에 존재하고, 애플리케이션 로직은 필요에 따라 확장한다.

클라우드 제공업체들도 이 트렌드를 인식하고 있다. Neon, Supabase, Crunchy Data 등은 PostgreSQL 위에 다양한 확장을 통합하여 "all-in-one 데이터 플랫폼"을 제공한다. 사용자는 하나의 데이터베이스 연결만으로 관계형 데이터, 전문 검색, 벡터 검색, 시계열 분석 등을 모두 수행할 수 있다.

앞으로 몇 년 동안 이러한 통합 트렌드는 더욱 가속화될 것으로 보인다. PostgreSQL 16과 17에서 이미 많은 성능 개선이 이루어졌으며, 18버전에서는 더 많은 확장 기능이 코어에 통합될 가능성이 있다. 특히 AI 시대에 벡터 검색과 키워드 검색의 하이브리드는 거의 모든 애플리케이션에서 필요하게 될 것이며, PostgreSQL은 이를 가장 간단하게 구현할 수 있는 플랫폼이 될 것이다.

## 결론: 합리적 선택을 위한 의사결정 프레임워크

PostgreSQL의 BM25 전문 검색 확장들은 단순한 기술적 진보를 넘어 시스템 아키텍처에 대한 새로운 사고를 요구한다. "Postgres Just Killed Elasticsearch"라는 도발적인 제목은 과장이지만, 그 이면의 메시지는 진지하게 받아들일 만하다. 많은 경우 우리는 필요 이상으로 복잡한 아키텍처를 채택해 왔으며, 더 단순한 대안이 실제로는 더 나은 결과를 줄 수 있다.

의사결정은 무조건적인 선택이 아니라 트레이드오프에 대한 이해에서 시작되어야 한다. PostgreSQL BM25는 아키텍처 단순성, 데이터 일관성, 개발 생산성을 제공하지만, 극단적인 규모나 복잡한 분석 기능에서는 한계가 있다. Elasticsearch는 대규모 분산 검색, 복잡한 집계, 성숙한 생태계를 제공하지만, 운영 복잡성, 데이터 동기화, 높은 비용이라는 대가를 치른다.

다음 체크리스트를 통해 자신의 상황을 평가해보자:

**PostgreSQL BM25를 강력히 고려해야 하는 경우:**
- 데이터 규모가 수천만 건 이하인가?
- 실시간 일관성이 중요한가?
- 검색과 트랜잭션 로직이 긴밀하게 결합되어 있는가?
- 인프라를 단순하게 유지하고 싶은가?
- 개발 팀이 작거나 PostgreSQL 전문성만 있는가?
- 비용 민감도가 높은가?
- 주로 텍스트 검색과 기본 필터링만 필요한가?

**Elasticsearch를 고려해야 하는 경우:**
- 데이터 규모가 수십억 건 이상인가?
- 복잡한 집계와 분석이 핵심 요구사항인가?
- 로그 분석, 모니터링, BI가 주 용도인가?
- 이미 Elasticsearch를 잘 운영하고 있는가?
- Elasticsearch 전문 인력이 있는가?
- Kibana 등 생태계 도구를 활용하고 있는가?

**하이브리드 접근을 고려해야 하는 경우:**
- 서로 다른 특성의 데이터가 있는가?
- 일부는 실시간이 중요하고 일부는 배치 분석인가?
- 점진적 마이그레이션을 원하는가?

무엇보다 중요한 것은 실험과 측정이다. 실제 데이터와 쿼리 패턴으로 두 접근 방식을 모두 테스트해보라. 벤치마크는 자신의 환경에서 직접 수행해야 의미가 있다. ParadeDB는 Docker 이미지를 제공하므로 pg_search를 몇 분 만에 시작할 수 있다. 실제 워크로드로 테스트하고, 성능, 기능, 복잡성을 종합적으로 평가한 후 결정하라.

또한 결정은 영구적이지 않다는 점을 기억하라. PostgreSQL로 시작했다가 규모가 커지면 Elasticsearch를 추가할 수 있다. 반대로 Elasticsearch를 사용하다가 요구사항이 단순화되면 PostgreSQL로 통합할 수도 있다. 중요한 것은 현재 시점에서 가장 합리적인 선택을 하는 것이다.

마지막으로 "Postgres Just Killed Elasticsearch"라는 제목으로 돌아가자. Elasticsearch는 죽지 않았다. 여전히 많은 환경에서 최고의 선택이다. 그러나 PostgreSQL이 더 이상 "검색은 못하는 데이터베이스"가 아니라는 점은 분명하다. BM25 확장들은 PostgreSQL을 진정한 검색 플랫폼으로 변모시켰다. 이제 개발자들은 더 많은 선택지를 가지게 되었으며, 많은 경우 더 단순한 길이 더 나은 길임을 발견하고 있다.

시스템 아키텍처의 본질은 복잡성과의 싸움이다. 우리는 종종 문제를 해결하기 위해 새로운 도구를 추가하지만, 각 도구는 새로운 복잡성을 가져온다. PostgreSQL BM25 확장들이 제공하는 가장 큰 가치는 기술적 성능이 아니라 아키텍처의 단순성이다. 하나의 데이터베이스, 하나의 트랜잭션 경계, 하나의 백업 전략. 이것이 진정한 혁신이다.

---

**작성 일자: 2026-01-29**
