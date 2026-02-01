---
title: "BM25 검색 완벽 가이드: PostgreSQL vs MongoDB vs Elasticsearch"
date: 2026-02-01 15:00:00 +0900
categories: [TechStack,  Backend Architecture]
mermaid: [True]
tags: [BM25,  PostgreSQL,  Elasticsearch,  MongoDB,  Claude.write]
---


## 서론: BM25가 검색의 표준이 된 이유

BM25(Best Matching 25)는 1990년대 Stephen Robertson과 그의 연구팀이 개발한 검색 랭킹 알고리즘으로, 현재 Google, Elasticsearch, MongoDB를 포함한 대부분의 주요 검색 엔진에서 표준 알고리즘으로 사용되고 있다. TF-IDF의 단순한 개선이 아니라, 확률론적 정보 검색 이론에 기반한 정교한 랭킹 모델이다.

### BM25가 TF-IDF를 대체한 이유

전통적인 TF-IDF는 치명적인 문제를 가지고 있었다:

**1. 용어 빈도의 무한 증가 문제**
- TF-IDF에서 단어가 100번 등장하면 10번 등장할 때보다 10배의 점수를 받음
- 실제로는 단어가 3번 등장하든 100번 등장하든 관련성에 큰 차이가 없음
- 키워드 스터핑(keyword stuffing)에 취약

**2. 문서 길이 정규화 부재**
- 긴 문서가 단순히 단어를 더 많이 포함한다는 이유로 불공정하게 높은 점수를 받음
- 짧고 관련성 높은 문서가 긴 문서에 밀려남

BM25는 이 두 문제를 수학적으로 우아하게 해결한다:

**BM25 공식:**
```
score(D, Q) = Σ IDF(qi) × [f(qi, D) × (k1 + 1)] / [f(qi, D) + k1 × (1 - b + b × |D| / avgdl)]
```

여기서:
- `IDF(qi)`: 역문서 빈도 (희귀한 단어일수록 높은 가중치)
- `f(qi, D)`: 문서 D에서 단어 qi의 빈도
- `k1`: 용어 빈도 포화 파라미터 (기본값 1.2)
- `b`: 문서 길이 정규화 파라미터 (기본값 0.75)
- `|D|`: 문서 D의 길이
- `avgdl`: 컬렉션의 평균 문서 길이

**핵심 개선사항:**

1. **용어 빈도 포화(Term Frequency Saturation)**: k1 파라미터가 로그 곡선처럼 작동하여, 단어가 추가로 등장할 때마다 점수 증가폭이 점점 감소한다. 단어가 2번 등장하면 1번일 때보다 큰 폭으로 증가하지만, 10번에서 11번으로 증가할 때는 거의 영향이 없다.

2. **문서 길이 정규화(Length Normalization)**: b 파라미터가 문서 길이를 평균과 비교하여, 평균보다 긴 문서는 페널티를, 짧은 문서는 보너스를 받는다. 같은 단어가 1번 등장해도 짧은 문서가 더 높은 점수를 받는다.

## PostgreSQL BM25: 확장을 통한 구현

PostgreSQL은 전통적으로 기본 전문 검색(Full-Text Search)에서 단순한 `ts_rank()` 함수만 제공했다. BM25를 사용하려면 확장을 설치해야 한다.

### 1. ParadeDB의 pg_search (Tantivy 기반)

ParadeDB는 Rust로 작성된 Tantivy 검색 엔진을 PostgreSQL 확장으로 통합했다.

**설치 (Docker 사용 권장):**
```bash
# ParadeDB 컨테이너 실행
docker run --name paradedb \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  -d paradedb/paradedb:latest

# 또는 기존 PostgreSQL에 설치 (Ubuntu/Debian)
curl -fsSL https://downloads.paradedb.com/paradedb/repo/deb/paradedb.gpg | sudo gpg --dearmor -o /usr/share/keyrings/paradedb.gpg
echo "deb [signed-by=/usr/share/keyrings/paradedb.gpg] https://downloads.paradedb.com/paradedb/repo/deb stable main" | sudo tee /etc/apt/sources.list.d/paradedb.list
sudo apt update && sudo apt install postgresql-16-paradedb
```

**사용 예제:**
```sql
-- 확장 활성화
CREATE EXTENSION pg_search;

-- 테스트 데이터 생성
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    category TEXT,
    price NUMERIC(10, 2),
    rating NUMERIC(3, 2)
);

INSERT INTO products (name, description, category, price, rating) VALUES
('Wireless Bluetooth Headphones', 'Premium noise-cancelling headphones with 30-hour battery life', 'Electronics', 299.99, 4.5),
('Gaming Laptop', 'High-performance laptop with RTX 4080 GPU and 32GB RAM', 'Electronics', 2499.99, 4.8),
('Organic Coffee Beans', 'Fair-trade Colombian coffee beans, medium roast', 'Food', 24.99, 4.3),
('Running Shoes', 'Lightweight running shoes with responsive cushioning', 'Sports', 129.99, 4.6);

-- BM25 인덱스 생성
CALL paradedb.create_bm25(
    index_name => 'products_search_idx',
    table_name => 'products',
    key_field => 'product_id',
    text_fields => paradedb.field('name', tokenizer => paradedb.tokenizer('en_stem')) || 
                   paradedb.field('description', tokenizer => paradedb.tokenizer('en_stem')),
    numeric_fields => paradedb.field('price') || 
                      paradedb.field('rating'),
    categorical_fields => paradedb.field('category')
);

-- BM25 검색 실행
SELECT 
    product_id,
    name,
    description,
    paradedb.score(product_id) AS bm25_score
FROM products
WHERE products @@@ 'description:headphones OR name:wireless'
ORDER BY bm25_score DESC
LIMIT 5;

-- 결과:
-- product_id | name                          | bm25_score
-- -----------+-------------------------------+------------
--          1 | Wireless Bluetooth Headphones | 8.234
--          2 | Gaming Laptop                 | 0.0

-- 퍼지 검색 (오타 허용)
SELECT name, paradedb.score(product_id) AS score
FROM products
WHERE products @@@ 'description:headphons~2'  -- 2글자까지 오타 허용
ORDER BY score DESC;

-- 부스팅 (필드 가중치)
SELECT name, paradedb.score(product_id) AS score
FROM products
WHERE products @@@ 'name:laptop^3 OR description:laptop'  -- name 필드에 3배 가중치
ORDER BY score DESC;

-- 복합 쿼리 (범위 + 텍스트)
SELECT name, price, paradedb.score(product_id) AS score
FROM products
WHERE products @@@ 'description:gaming AND price:[1000 TO 3000] AND rating:[4 TO 5]'
ORDER BY score DESC;

-- 하이라이팅 (매칭된 부분 강조)
SELECT 
    name,
    paradedb.highlight(product_id, 'description') AS highlighted_description,
    paradedb.score(product_id) AS score
FROM products
WHERE products @@@ 'description:noise-cancelling'
ORDER BY score DESC;

-- 집계 (faceting)
SELECT 
    category,
    COUNT(*) as count
FROM products
WHERE products @@@ 'description:coffee OR description:laptop'
GROUP BY category;
```

**pg_search의 장점:**
- Elasticsearch와 유사한 Query DSL 문법
- Rust/Tantivy 기반으로 매우 빠른 성능
- 퍼지 매칭, 구문 검색, 하이라이팅 등 고급 기능 지원
- ParadeDB가 적극 개발 중 (2024-2025년 급속 성장)

**pg_search의 단점:**
- 상대적으로 새로운 프로젝트 (성숙도 문제)
- 관리형 PostgreSQL 서비스(AWS RDS, Google Cloud SQL)에서 사용 불가능
- Docker 또는 자체 호스팅 필요

### 2. Tiger Data의 pg_textsearch

2024년 12월에 오픈소스로 공개된 PostgreSQL 네이티브에 가까운 BM25 구현.

**설치:**
```bash
# 소스 컴파일 필요 (현재 바이너리 배포 없음)
git clone https://github.com/tigerdata/pg_textsearch.git
cd pg_textsearch
make
sudo make install
```

**사용 예제:**
```sql
-- 확장 활성화
CREATE EXTENSION pg_textsearch CASCADE;

-- BM25 인덱스 생성
CREATE INDEX idx_products_description ON products 
USING bm25(description) 
WITH (text_config='english');

-- BM25 검색 (<@> 연산자 사용)
-- 주의: 점수가 낮을수록 더 관련성이 높음 (거리 개념)
SELECT 
    product_id,
    name,
    description <@> 'wireless headphones' AS bm25_score
FROM products
ORDER BY bm25_score ASC  -- 오름차순!
LIMIT 5;

-- 다중 필드 검색
CREATE INDEX idx_products_combined ON products 
USING bm25((name || ' ' || description)) 
WITH (text_config='english');

SELECT 
    name,
    (name || ' ' || description) <@> 'gaming laptop' AS score
FROM products
ORDER BY score ASC
LIMIT 5;

-- 필터와 결합
SELECT 
    name,
    price,
    description <@> 'coffee beans' AS score
FROM products
WHERE category = 'Food'
  AND price < 50
ORDER BY score ASC;

-- 병렬 인덱스 빌드 (대용량 데이터)
SET max_parallel_maintenance_workers = 4;
CREATE INDEX idx_large_table ON large_products 
USING bm25(description) 
WITH (text_config='english');
```

**pg_textsearch의 장점:**
- PostgreSQL 네이티브 스타일 문법 (`<@>` 연산자)
- PostgreSQL 라이선스 (완전 오픈소스)
- pgvector/pgvectorscale과 자연스러운 통합 (하이브리드 검색)
- 병렬 인덱스 빌드 지원

**pg_textsearch의 단점:**
- 매우 새로운 프로젝트 (2024년 12월 공개)
- 바이너리 배포 없음 (소스 컴파일 필요)
- 관리형 서비스 지원 없음

### 3. PostgreSQL 하이브리드 검색 (BM25 + Vector)

BM25(키워드)와 pgvector(의미론적)를 결합한 하이브리드 검색:

```sql
-- Vector 확장 설치
CREATE EXTENSION vector;

-- 임베딩 컬럼 추가
ALTER TABLE products ADD COLUMN embedding vector(1536);

-- 임베딩 생성 (Python/OpenAI API)
UPDATE products SET embedding = generate_embedding(description);
-- generate_embedding()은 별도 구현 필요 (OpenAI API 등)

-- Vector 인덱스 생성
CREATE INDEX idx_products_embedding ON products 
USING hnsw (embedding vector_cosine_ops);

-- 하이브리드 검색 (Reciprocal Rank Fusion)
WITH bm25_results AS (
    SELECT 
        product_id,
        name,
        description <@> 'noise cancelling headphones' AS bm25_score,
        ROW_NUMBER() OVER (ORDER BY description <@> 'noise cancelling headphones') AS bm25_rank
    FROM products
    ORDER BY bm25_score ASC
    LIMIT 20
),
vector_results AS (
    SELECT 
        product_id,
        name,
        1 - (embedding <=> '[임베딩 벡터]'::vector) AS vector_score,
        ROW_NUMBER() OVER (ORDER BY embedding <=> '[임베딩 벡터]'::vector) AS vector_rank
    FROM products
    ORDER BY embedding <=> '[임베딩 벡터]'::vector
    LIMIT 20
)
SELECT 
    COALESCE(b.product_id, v.product_id) AS product_id,
    COALESCE(b.name, v.name) AS name,
    -- Reciprocal Rank Fusion 점수 계산
    COALESCE(1.0 / (60 + b.bm25_rank), 0.0) + 
    COALESCE(1.0 / (60 + v.vector_rank), 0.0) AS rrf_score
FROM bm25_results b
FULL OUTER JOIN vector_results v ON b.product_id = v.product_id
ORDER BY rrf_score DESC
LIMIT 10;
```

## MongoDB Atlas Search: 완전 관리형 BM25

MongoDB Atlas Search는 Lucene을 기반으로 하는 완전 관리형 검색 솔루션이다. 별도의 검색 엔진 없이 MongoDB 내에서 BM25 검색을 사용할 수 있다.

### 아키텍처

MongoDB Atlas Search는 `mongot`라는 별도 프로세스가 검색 인덱스를 관리한다:
- **mongod**: MongoDB 데이터베이스 프로세스
- **mongot**: Lucene 기반 검색 프로세스
- 두 프로세스가 긴밀하게 통합되어 동작

### 기본 사용법

```javascript
// 1. Atlas Search 인덱스 생성 (Atlas UI 또는 CLI)
// products 컬렉션에 대한 검색 인덱스 정의
{
  "mappings": {
    "dynamic": true,  // 모든 필드 자동 인덱싱
    "fields": {
      "name": {
        "type": "string",
        "analyzer": "lucene.standard"
      },
      "description": {
        "type": "string",
        "analyzer": "lucene.english"  // 영어 형태소 분석
      },
      "category": {
        "type": "string",
        "analyzer": "lucene.keyword"  // 정확한 매칭
      },
      "price": {
        "type": "number"
      },
      "rating": {
        "type": "number"
      }
    }
  }
}

// 2. 샘플 데이터 삽입
db.products.insertMany([
  {
    name: "Wireless Bluetooth Headphones",
    description: "Premium noise-cancelling headphones with 30-hour battery life",
    category: "Electronics",
    price: 299.99,
    rating: 4.5
  },
  {
    name: "Gaming Laptop",
    description: "High-performance laptop with RTX 4080 GPU and 32GB RAM",
    category: "Electronics",
    price: 2499.99,
    rating: 4.8
  },
  {
    name: "Organic Coffee Beans",
    description: "Fair-trade Colombian coffee beans, medium roast",
    category: "Food",
    price: 24.99,
    rating: 4.3
  }
]);

// 3. 기본 BM25 텍스트 검색
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "wireless headphones",
        path: ["name", "description"]
      }
    }
  },
  {
    $project: {
      name: 1,
      description: 1,
      score: { $meta: "searchScore" }
    }
  },
  { $limit: 10 }
]);

// 4. 필드별 부스팅
db.products.aggregate([
  {
    $search: {
      index: "default",
      compound: {
        should: [
          {
            text: {
              query: "laptop",
              path: "name",
              score: { boost: { value: 3 } }  // name 필드에 3배 가중치
            }
          },
          {
            text: {
              query: "laptop",
              path: "description",
              score: { boost: { value: 1 } }
            }
          }
        ]
      }
    }
  },
  {
    $project: {
      name: 1,
      score: { $meta: "searchScore" }
    }
  }
]);

// 5. 복합 쿼리 (텍스트 + 범위 필터)
db.products.aggregate([
  {
    $search: {
      index: "default",
      compound: {
        must: [
          {
            text: {
              query: "gaming",
              path: "description"
            }
          }
        ],
        filter: [
          {
            range: {
              path: "price",
              gte: 1000,
              lte: 3000
            }
          },
          {
            range: {
              path: "rating",
              gte: 4.0
            }
          }
        ]
      }
    }
  },
  {
    $project: {
      name: 1,
      price: 1,
      rating: 1,
      score: { $meta: "searchScore" }
    }
  }
]);

// 6. 퍼지 검색 (오타 허용)
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "headphons",  // 오타
        path: "description",
        fuzzy: {
          maxEdits: 2,  // 최대 2글자 차이 허용
          prefixLength: 1
        }
      }
    }
  },
  {
    $project: {
      name: 1,
      score: { $meta: "searchScore" }
    }
  }
]);

// 7. 하이라이팅
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "noise-cancelling",
        path: "description"
      },
      highlight: {
        path: "description"
      }
    }
  },
  {
    $project: {
      name: 1,
      highlights: { $meta: "searchHighlights" },
      score: { $meta: "searchScore" }
    }
  }
]);

// 8. 집계 (Faceting)
db.products.aggregate([
  {
    $searchMeta: {
      index: "default",
      facet: {
        operator: {
          text: {
            query: "electronics",
            path: "description"
          }
        },
        facets: {
          categoriesFacet: {
            type: "string",
            path: "category"
          },
          priceRanges: {
            type: "number",
            path: "price",
            boundaries: [0, 100, 500, 1000, 5000]
          }
        }
      }
    }
  }
]);
```

### MongoDB BM25 스코어 상세 분석

```javascript
// 스코어 계산 과정을 상세하게 확인
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "wireless headphones",
        path: "description"
      }
    }
  },
  {
    $project: {
      name: 1,
      description: 1,
      score: { $meta: "searchScore" },
      scoreDetails: { $meta: "searchScoreDetails" }  // 스코어 계산 상세
    }
  },
  { $limit: 1 }
]);

// 결과 예시:
{
  "_id": ObjectId("..."),
  "name": "Wireless Bluetooth Headphones",
  "description": "Premium noise-cancelling headphones...",
  "score": 6.011,
  "scoreDetails": {
    "value": 6.011,
    "description": "sum of:",
    "details": [
      {
        "value": 6.011,
        "description": "weight(description:wireless) [BM25Similarity]",
        "details": [
          {
            "value": 13.083,
            "description": "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5))",
            "details": [
              { "value": 1000, "description": "N, total documents" },
              { "value": 50, "description": "n, documents with term" }
            ]
          },
          {
            "value": 0.459,
            "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl))",
            "details": [
              { "value": 3, "description": "freq, occurrences of term" },
              { "value": 1.2, "description": "k1, term saturation parameter" },
              { "value": 0.75, "description": "b, length normalization" },
              { "value": 500, "description": "dl, length of field" },
              { "value": 450, "description": "avgdl, average field length" }
            ]
          }
        ]
      }
    ]
  }
}
```

### MongoDB 하이브리드 검색 (BM25 + Vector)

MongoDB는 2025년 9월 `$rankFusion`과 `$scoreFusion`을 도입하여 하이브리드 검색을 네이티브로 지원한다.

```javascript
// 1. Vector Search 인덱스 생성 (별도)
{
  "type": "vectorSearch",
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    }
  ]
}

// 2. $rankFusion을 사용한 하이브리드 검색
db.products.aggregate([
  {
    $rankFusion: {
      input: {
        pipelines: {
          // BM25 텍스트 검색
          textSearch: [
            {
              $search: {
                index: "default",
                text: {
                  query: "wireless headphones",
                  path: ["name", "description"]
                }
              }
            },
            { $limit: 20 }
          ],
          // Vector 의미론적 검색
          vectorSearch: [
            {
              $vectorSearch: {
                index: "vector_index",
                queryVector: [0.12, 0.34, ...],  // 임베딩 벡터
                path: "embedding",
                numCandidates: 100,
                limit: 20
              }
            }
          ]
        }
      }
    }
  },
  { $limit: 10 }
]);

// 3. $scoreFusion으로 가중 평균 (더 세밀한 제어)
db.products.aggregate([
  {
    $scoreFusion: {
      input: {
        pipelines: {
          textSearch: [
            {
              $search: {
                index: "default",
                text: {
                  query: "wireless headphones",
                  path: "description"
                }
              }
            },
            { $limit: 20 },
            { $set: { textScore: { $meta: "searchScore" } } }
          ],
          vectorSearch: [
            {
              $vectorSearch: {
                index: "vector_index",
                queryVector: [0.12, 0.34, ...],
                path: "embedding",
                numCandidates: 100,
                limit: 20
              }
            },
            { $set: { vectorScore: { $meta: "vectorSearchScore" } } }
          ]
        }
      },
      // 텍스트 70%, 벡터 30% 가중치
      weights: {
        textSearch: 0.7,
        vectorSearch: 0.3
      }
    }
  },
  { $limit: 10 }
]);
```

### MongoDB Atlas Search 고급 기능

**1. 동적 부스팅 (Embedded Scoring Pattern)**

문서 내에 부스팅 로직을 직접 포함시켜 실시간으로 랭킹 조정:

```javascript
// 제품에 부스팅 점수 추가
db.products.updateOne(
  { name: "Wireless Bluetooth Headphones" },
  {
    $set: {
      boost_terms: [
        { term: "premium", boost: 10 },
        { term: "wireless", boost: 5 },
        { term: "noise-cancelling", boost: 8 }
      ]
    }
  }
);

// 검색 인덱스에 boost_terms 필드 추가
{
  "mappings": {
    "fields": {
      "boost_terms": {
        "type": "embeddedDocuments",
        "fields": {
          "term": { "type": "string" },
          "boost": { "type": "number" }
        }
      }
    }
  }
}

// 부스팅을 활용한 검색
db.products.aggregate([
  {
    $search: {
      index: "default",
      embeddedDocument: {
        path: "boost_terms",
        operator: {
          compound: {
            should: [
              {
                text: {
                  query: "premium wireless",
                  path: "boost_terms.term",
                  score: {
                    boost: {
                      path: "boost_terms.boost"
                    }
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
]);
```

**2. 대체 스코어링 알고리즘**

BM25 외에도 두 가지 알고리즘 제공 (2025년 신규):

```javascript
// stableTfl: 용어 길이 기반 희귀도 (일관된 페이지네이션)
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "laptop",
        path: "description",
        score: {
          function: {
            score: "stableTfl"
          }
        }
      }
    }
  }
]);

// boolean: 용어 존재 여부만 카운트 (엔티티 매칭)
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "wireless bluetooth headphones",
        path: "description",
        score: {
          function: {
            score: "boolean"  // 3개 용어 중 매칭 개수만
          }
        }
      }
    }
  }
]);
```

## Elasticsearch: BM25의 원조 구현

Elasticsearch는 Lucene 기반으로 BM25를 구현한 최초의 대중적 검색 엔진이다. 2016년 Elasticsearch 5.0부터 BM25가 기본 알고리즘이 되었다.

### 기본 설정과 사용법

```json
// 1. 인덱스 생성
PUT /products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index": {
      "similarity": {
        "default": {
          "type": "BM25"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "standard"
      },
      "description": {
        "type": "text",
        "analyzer": "english"
      },
      "category": {
        "type": "keyword"
      },
      "price": {
        "type": "float"
      },
      "rating": {
        "type": "float"
      }
    }
  }
}

// 2. 문서 색인
POST /products/_bulk
{"index": {"_id": 1}}
{"name": "Wireless Bluetooth Headphones", "description": "Premium noise-cancelling headphones with 30-hour battery life", "category": "Electronics", "price": 299.99, "rating": 4.5}
{"index": {"_id": 2}}
{"name": "Gaming Laptop", "description": "High-performance laptop with RTX 4080 GPU and 32GB RAM", "category": "Electronics", "price": 2499.99, "rating": 4.8}
{"index": {"_id": 3}}
{"name": "Organic Coffee Beans", "description": "Fair-trade Colombian coffee beans, medium roast", "category": "Food", "price": 24.99, "rating": 4.3}

// 3. 기본 BM25 검색
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "wireless headphones",
      "fields": ["name^3", "description"]  // name 필드에 3배 가중치
    }
  }
}

// 4. Bool 쿼리 (복합 조건)
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "description": "gaming"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "price": {
              "gte": 1000,
              "lte": 3000
            }
          }
        },
        {
          "range": {
            "rating": {
              "gte": 4.0
            }
          }
        }
      ]
    }
  }
}

// 5. 퍼지 검색
GET /products/_search
{
  "query": {
    "match": {
      "description": {
        "query": "headphons",
        "fuzziness": "AUTO"  // 자동으로 편집 거리 결정
      }
    }
  }
}

// 6. 하이라이팅
GET /products/_search
{
  "query": {
    "match": {
      "description": "noise-cancelling"
    }
  },
  "highlight": {
    "fields": {
      "description": {
        "fragment_size": 150,
        "number_of_fragments": 3
      }
    }
  }
}

// 7. 집계 (Aggregations)
GET /products/_search
{
  "size": 0,
  "query": {
    "match": {
      "description": "electronics"
    }
  },
  "aggs": {
    "categories": {
      "terms": {
        "field": "category"
      }
    },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500, "to": 1000 },
          { "from": 1000 }
        ]
      }
    }
  }
}
```

### BM25 파라미터 커스터마이징

Elasticsearch는 k1과 b 파라미터를 세밀하게 조정할 수 있다.

```json
// 커스텀 BM25 설정
PUT /custom_products
{
  "settings": {
    "index": {
      "similarity": {
        "custom_bm25": {
          "type": "BM25",
          "k1": 1.5,    // 기본값 1.2보다 높음 (용어 빈도 영향 증가)
          "b": 0.7      // 기본값 0.75보다 낮음 (길이 정규화 약화)
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "similarity": "custom_bm25"  // 커스텀 유사도 적용
      },
      "content": {
        "type": "text",
        "similarity": "custom_bm25"
      }
    }
  }
}

// k1과 b 파라미터 의미:
// k1 (기본값 1.2):
//   - 낮을수록 (예: 0.5): 용어 빈도 포화가 빠르게 일어남 (중복 단어 영향 감소)
//   - 높을수록 (예: 3.0): 용어가 많이 등장할수록 점수가 계속 증가
//   - 0에 가까우면: 거의 Boolean 검색 (용어 존재 여부만)
//   
// b (기본값 0.75):
//   - 0에 가까우면: 문서 길이 무시 (긴 문서 불이익 없음)
//   - 1에 가까우면: 문서 길이를 강하게 반영 (짧은 문서 선호)
```

**파라미터 튜닝 가이드라인:**

| 상황 | k1 조정 | b 조정 |
|---|---|---|
| 짧은 문서 많음 (트윗, 제목) | 낮게 (0.8-1.0) | 낮게 (0.5-0.6) |
| 긴 문서 많음 (논문, 기사) | 높게 (1.5-2.0) | 높게 (0.8-0.9) |
| 정확한 용어 매칭 중요 | 낮게 (0.5-1.0) | 중간 (0.75) |
| 용어 빈도 중요 | 높게 (2.0-3.0) | 중간 (0.75) |
| 문서 길이 균일 | 중간 (1.2) | 낮게 (0.3-0.5) |

### Explain API로 스코어 분석

```json
GET /products/_explain/1
{
  "query": {
    "multi_match": {
      "query": "wireless headphones",
      "fields": ["name", "description"]
    }
  }
}

// 결과 (간략화):
{
  "value": 6.234,
  "description": "sum of:",
  "details": [
    {
      "value": 2.145,
      "description": "weight(description:wireless in 0) [PerFieldSimilarity], result of:",
      "details": [
        {
          "value": 2.145,
          "description": "score(freq=1.0), computed as boost * idf * tf from:",
          "details": [
            {
              "value": 3.967,
              "description": "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
              "details": [
                { "value": 3, "description": "n, number of documents containing term" },
                { "value": 100, "description": "N, total number of documents with field" }
              ]
            },
            {
              "value": 0.541,
              "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
              "details": [
                { "value": 1.0, "description": "freq, occurrences of term within document" },
                { "value": 1.2, "description": "k1, term saturation parameter" },
                { "value": 0.75, "description": "b, length normalization parameter" },
                { "value": 45.0, "description": "dl, length of field" },
                { "value": 50.0, "description": "avgdl, average length of field" }
              ]
            }
          ]
        }
      ]
    },
    {
      "value": 4.089,
      "description": "weight(description:headphones in 0)...",
      "details": [ /* 유사한 구조 */ ]
    }
  ]
}
```

### Function Score로 비즈니스 로직 통합

BM25 점수에 비즈니스 메트릭(가격, 재고, 인기도 등)을 곱셈으로 결합:

```json
GET /products/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query": "laptop",
          "fields": ["name", "description"]
        }
      },
      "functions": [
        {
          // 평점에 따른 부스팅
          "filter": { "range": { "rating": { "gte": 4.5 } } },
          "weight": 1.5
        },
        {
          // 가격 역수 (저렴할수록 높은 점수)
          "script_score": {
            "script": {
              "source": "1 / (doc['price'].value / 100)"
            }
          }
        },
        {
          // 필드 값 기반 부스팅
          "field_value_factor": {
            "field": "rating",
            "factor": 1.2,
            "modifier": "sqrt",
            "missing": 1
          }
        }
      ],
      "score_mode": "multiply",  // 모든 함수를 곱셈
      "boost_mode": "multiply"   // BM25 점수와도 곱셈
    }
  }
}
```

### Elasticsearch 샤딩과 BM25

Elasticsearch의 분산 아키텍처는 BM25 점수에 영향을 줄 수 있다:

```json
// 문제: 샤드별로 IDF가 다르게 계산됨
PUT /products
{
  "settings": {
    "number_of_shards": 3  // 3개 샤드로 분산
  }
}

// 샤드 0: "laptop" 용어가 10개 문서 중 2개에 등장 -> IDF 높음
// 샤드 1: "laptop" 용어가 10개 문서 중 8개에 등장 -> IDF 낮음
// 샤드 2: "laptop" 용어가 10개 문서 중 5개에 등장 -> IDF 중간
// => 같은 단어인데 샤드에 따라 다른 점수!

// 해결책 1: DFS Query Then Fetch (전체 통계 수집)
GET /products/_search?search_type=dfs_query_then_fetch
{
  "query": {
    "match": { "description": "laptop" }
  }
}
// - 모든 샤드에서 통계를 먼저 수집한 후 검색
// - 정확한 점수, 하지만 추가 네트워크 왕복 필요
// - 대부분의 경우 기본 설정으로 충분 (데이터가 많으면 통계 수렴)

// 해결책 2: 샤드 수 줄이기
PUT /products_single_shard
{
  "settings": {
    "number_of_shards": 1  // 단일 샤드
  }
}
// - 가장 정확한 점수
// - 확장성 제한 (단일 노드)
```

## 3대 플랫폼 종합 비교

### 기능 비교표

| 기능 | PostgreSQL (pg_search) | PostgreSQL (pg_textsearch) | MongoDB Atlas Search | Elasticsearch |
|---|---|---|---|---|
| **설치 복잡도** | 중간 (Docker 권장) | 높음 (소스 컴파일) | 없음 (관리형) | 중간 (Docker/클라우드) |
| **BM25 구현** | Tantivy | 자체 구현 | Lucene | Lucene |
| **k1, b 조정** | ❌ | ❌ | ❌ | ✅ |
| **퍼지 매칭** | ✅ (~2) | ❌ | ✅ (maxEdits) | ✅ (AUTO) |
| **구문 검색** | ✅ | ✅ | ✅ | ✅ |
| **하이라이팅** | ✅ | ❌ | ✅ | ✅ |
| **집계/Faceting** | ✅ | ❌ | ✅ | ✅ |
| **하이브리드 검색** | ✅ (pgvector) | ✅ (pgvector) | ✅ (네이티브) | ❌ (별도 구현) |
| **실시간 업데이트** | ✅ | ✅ | ✅ (~1초) | ✅ (~1초) |
| **ACID 트랜잭션** | ✅ | ✅ | ❌ | ❌ |
| **확장성** | 수직 (단일 서버) | 수직 (단일 서버) | 수평 (샤딩) | 수평 (샤딩) |
| **관리형 서비스** | ❌ | ❌ | ✅ (Atlas) | ✅ (Elastic Cloud) |
| **가격** | 무료 (오픈소스) | 무료 (오픈소스) | Atlas 요금 | Elastic Cloud 요금 |

### 성능 비교 (추정치, 100만 문서 기준)

| 플랫폼 | 인덱싱 속도 | 검색 지연시간 | 메모리 사용량 |
|---|---|---|---|
| **PostgreSQL (pg_search)** | ~50초 (전체) | 10-50ms | 2-4GB |
| **PostgreSQL (pg_textsearch)** | ~60초 (전체) | 10-50ms | 2-3GB |
| **MongoDB Atlas Search** | ~2분 (백그라운드) | 20-100ms | 3-6GB |
| **Elasticsearch** | ~1분 (병렬) | 5-50ms | 4-8GB |

**주의**: 실제 성능은 하드웨어, 문서 크기, 쿼리 복잡도에 따라 크게 달라집니다.

### 아키텍처 선택 가이드

**PostgreSQL을 선택하세요:**
- 데이터가 이미 PostgreSQL에 있고, 별도 시스템을 추가하고 싶지 않을 때
- 강력한 ACID 트랜잭션이 필요할 때
- 데이터 볼륨이 수백만~수천만 건일 때
- 검색과 트랜잭션 로직이 긴밀하게 결합되어 있을 때
- 인프라를 단순하게 유지하고 싶을 때
- 하이브리드 검색(키워드+벡터)을 단일 DB에서 구현하고 싶을 때

**MongoDB Atlas Search를 선택하세요:**
- 데이터가 이미 MongoDB에 있고, 완전 관리형 솔루션을 원할 때
- JSON 문서 구조와 검색을 자연스럽게 통합하고 싶을 때
- 검색 인프라 관리 부담을 최소화하고 싶을 때
- 하이브리드 검색을 네이티브로 지원받고 싶을 때 ($rankFusion/$scoreFusion)
- 샤딩을 통한 수평 확장이 필요할 때
- 동적 스키마의 이점을 살리고 싶을 때

**Elasticsearch를 선택하세요:**
- 수억~수십억 건의 문서를 검색해야 할 때
- 복잡한 집계와 분석이 핵심 요구사항일 때
- 로그 분석, 모니터링, BI가 주 목적일 때
- BM25 파라미터(k1, b)를 세밀하게 튜닝해야 할 때
- Kibana 생태계가 필요할 때
- 이미 Elasticsearch를 성공적으로 운영 중일 때
- 검색 전용 시스템으로 최고 성능이 필요할 때

### 실무 의사결정 프레임워크

```
START
  |
  ├─ 데이터가 어디 있는가?
  |   ├─ PostgreSQL → pg_search 또는 pg_textsearch 시작
  |   ├─ MongoDB → Atlas Search 시작
  |   └─ 없음/신규 → 아래 계속
  |
  ├─ 데이터 볼륨은?
  |   ├─ <1천만 건 → PostgreSQL 충분
  |   ├─ 1천만~1억 건 → PostgreSQL 또는 MongoDB
  |   └─ >1억 건 → Elasticsearch 고려
  |
  ├─ 트랜잭션이 중요한가?
  |   ├─ Yes (금융, 전자상거래) → PostgreSQL
  |   └─ No (로그, 분석) → MongoDB/Elasticsearch
  |
  ├─ 팀의 전문성은?
  |   ├─ PostgreSQL DBA → PostgreSQL
  |   ├─ MongoDB 개발자 → MongoDB
  |   └─ Elasticsearch 엔지니어 → Elasticsearch
  |
  ├─ 인프라 관리 역량은?
  |   ├─ 최소화 원함 → MongoDB Atlas (관리형)
  |   ├─ 자체 운영 가능 → PostgreSQL/Elasticsearch
  |   └─ 클라우드 선호 → Elastic Cloud 또는 Atlas
  |
  └─ 하이브리드 검색 필요?
      ├─ Yes, 네이티브 → MongoDB ($rankFusion)
      ├─ Yes, 커스텀 OK → PostgreSQL (pgvector)
      └─ No → 위 조건만 고려
```

## 마이그레이션 전략

### Elasticsearch → PostgreSQL

```sql
-- 1. PostgreSQL 스키마 생성
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    author TEXT,
    published_at TIMESTAMP,
    tags TEXT[],
    metadata JSONB
);

-- 2. pg_search 인덱스
CREATE EXTENSION pg_search;

CALL paradedb.create_bm25(
    index_name => 'articles_idx',
    table_name => 'articles',
    key_field => 'id',
    text_fields => paradedb.field('title', tokenizer => paradedb.tokenizer('en_stem')) ||
                   paradedb.field('content', tokenizer => paradedb.tokenizer('en_stem'))
);

-- 3. Elasticsearch 데이터 추출 및 마이그레이션
-- Python 스크립트 예제
from elasticsearch import Elasticsearch
from elasticsearch.helpers import scan
import psycopg2

es = Elasticsearch(['http://localhost:9200'])
pg_conn = psycopg2.connect("dbname=mydb user=postgres")
pg_cur = pg_conn.cursor()

# Elasticsearch에서 스크롤
for doc in scan(es, index="articles", query={"query": {"match_all": {}}}):
    pg_cur.execute("""
        INSERT INTO articles (title, content, author, published_at, tags, metadata)
        VALUES (%s, %s, %s, %s, %s, %s)
    """, (
        doc['_source']['title'],
        doc['_source']['content'],
        doc['_source']['author'],
        doc['_source']['published_at'],
        doc['_source']['tags'],
        json.dumps(doc['_source'].get('metadata', {}))
    ))

pg_conn.commit()
```

### MongoDB → Elasticsearch

```javascript
// 1. Elasticsearch 매핑 생성
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "description": { "type": "text" },
      "category": { "type": "keyword" },
      "price": { "type": "float" },
      "attributes": { "type": "object", "enabled": false }  // JSONB 데이터
    }
  }
}

// 2. MongoDB Change Streams로 실시간 동기화
const { MongoClient } = require('mongodb');
const { Client } = require('@elastic/elasticsearch');

const mongoClient = new MongoClient('mongodb://localhost:27017');
const esClient = new Client({ node: 'http://localhost:9200' });

async function syncToElasticsearch() {
    await mongoClient.connect();
    const db = mongoClient.db('mydb');
    const collection = db.collection('products');
    
    // Change Stream 시작
    const changeStream = collection.watch();
    
    changeStream.on('change', async (change) => {
        if (change.operationType === 'insert' || change.operationType === 'update') {
            const doc = change.fullDocument;
            await esClient.index({
                index: 'products',
                id: doc._id.toString(),
                document: {
                    name: doc.name,
                    description: doc.description,
                    category: doc.category,
                    price: doc.price,
                    attributes: doc.attributes
                }
            });
        } else if (change.operationType === 'delete') {
            await esClient.delete({
                index: 'products',
                id: change.documentKey._id.toString()
            });
        }
    });
}

syncToElasticsearch();
```

## 실전 튜닝 가이드

### PostgreSQL BM25 최적화

```sql
-- 1. 인덱스 통계 확인
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

-- 2. 쿼리 성능 분석
EXPLAIN (ANALYZE, BUFFERS)
SELECT name, paradedb.score(product_id)
FROM products
WHERE products @@@ 'description:laptop AND price:[1000 TO 3000]'
ORDER BY paradedb.score(product_id) DESC
LIMIT 10;

-- 3. VACUUM 및 ANALYZE
VACUUM ANALYZE products;

-- 4. 파티셔닝 (시계열 데이터)
CREATE TABLE articles_2025_q1 PARTITION OF articles
    FOR VALUES FROM ('2025-01-01') TO ('2025-04-01');

CREATE TABLE articles_2025_q2 PARTITION OF articles
    FOR VALUES FROM ('2025-04-01') TO ('2025-07-01');

-- 파티션별 BM25 인덱스
CALL paradedb.create_bm25(
    index_name => 'articles_2025_q1_idx',
    table_name => 'articles_2025_q1',
    key_field => 'id',
    text_fields => paradedb.field('content')
);
```

### MongoDB Atlas Search 최적화

```javascript
// 1. 인덱스 분석
db.products.aggregate([
  { $indexStats: {} }
]);

// 2. 쿼리 프로파일링
db.setProfilingLevel(2);  // 모든 쿼리 프로파일링

// 검색 수행
db.products.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "laptop",
        path: "description"
      }
    }
  }
]);

// 프로파일 확인
db.system.profile.find().sort({ ts: -1 }).limit(1).pretty();

// 3. 선택적 필드 인덱싱 (인덱스 크기 축소)
{
  "mappings": {
    "dynamic": false,  // 자동 인덱싱 비활성화
    "fields": {
      "name": { "type": "string" },
      "description": { "type": "string" }
      // price, rating 등은 인덱싱하지 않음
    }
  }
}

// 4. 전용 Search Nodes (격리된 리소스)
// Atlas UI에서 설정:
// - Cluster Tier: M30 이상
// - Analytics Nodes: 검색 전용 노드 추가
// - 쿼리가 자동으로 Search Nodes로 라우팅됨
```

### Elasticsearch 최적화

```json
// 1. 샤드 크기 최적화
// 권장: 샤드당 20-50GB
PUT /products
{
  "settings": {
    "number_of_shards": 3,  // 150GB 데이터 -> 3샤드
    "number_of_replicas": 1
  }
}

// 2. 리프레시 간격 조정
PUT /products/_settings
{
  "index": {
    "refresh_interval": "30s"  // 기본 1s -> 30s (색인 속도 향상)
  }
}

// 3. 병합 정책 조정
PUT /products/_settings
{
  "index": {
    "merge": {
      "policy": {
        "max_merged_segment": "5gb",
        "segments_per_tier": 10
      }
    }
  }
}

// 4. 필드 데이터 캐시 모니터링
GET /_nodes/stats/indices/fielddata?human

// 5. 인덱스 라이프사이클 관리 (ILM)
PUT /_ilm/policy/products_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "7d"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

## 결론: 올바른 도구 선택하기

BM25는 검색의 표준이지만, 구현 플랫폼에 따라 특성이 크게 다르다. PostgreSQL은 단순성과 ACID 보장으로, MongoDB는 유연성과 관리 편의성으로, Elasticsearch는 확장성과 성능으로 각각 차별화된다.

**핵심 선택 기준:**

1. **데이터 위치**: 이미 사용 중인 데이터베이스에 검색 기능을 추가하는 것이 가장 단순하다.
2. **데이터 볼륨**: 수천만 건 이하면 PostgreSQL, 수억 건 이상이면 Elasticsearch를 고려한다.
3. **트랜잭션**: ACID가 중요하면 PostgreSQL이 유일한 선택지다.
4. **팀 역량**: 기존 팀의 전문성을 활용하는 것이 장기적으로 유리하다.
5. **복잡도 허용**: 단순성을 원하면 PostgreSQL, 최고 성능을 원하면 Elasticsearch다.

대부분의 애플리케이션은 PostgreSQL의 BM25로 충분하다. 복잡성을 추가하기 전에 먼저 시도해보라. 정말 Elasticsearch가 필요한 시점은 생각보다 늦게 온다.

---

**작성 일자: 2026-02-01**
