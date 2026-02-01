---
title: "PostgreSQL: 단순한 데이터베이스를 넘어 백엔드 아키텍처의 혁명"
date: 2026-01-29 20:00:00 +0900
categories: [TechStack,  Backend Architecture]
mermaid: [True]
tags: [PostgreSQL,  Claude.write]
---


## 관련영상

[**MySQL 말고 PostgreSQL을 써야 하는 결정적 이유 (백엔드 설계의 혁명)**](https://www.youtube.com/watch?v=Gp_tizXXZM8)

## 서론: 왜 PostgreSQL이 "기술 스택의 중심"이 되는가

현대 백엔드 아키텍처를 설계할 때 개발자들은 수많은 선택지 앞에 놓인다. 메시지 큐는 RabbitMQ를 쓸 것인가, Kafka를 쓸 것인가? 캐시는 Redis로 갈 것인가? 검색은 Elasticsearch를 도입해야 하는가? 실시간 통신을 위해 Socket.io를 추가해야 하는가? NoSQL은 MongoDB로 할 것인가, DynamoDB로 할 것인가? 이러한 결정들은 각각 새로운 인프라 컴포넌트를 추가하고, 각 컴포넌트는 학습 곡선, 운영 오버헤드, 장애 지점을 가져온다.

그러나 PostgreSQL은 이러한 복잡한 선택의 많은 부분을 단순화할 수 있는 능력을 가지고 있다. 단순히 뛰어난 관계형 데이터베이스라는 정체성을 넘어서, PostgreSQL은 NoSQL의 유연성(JSONB), 메시지 브로커의 실시간성(LISTEN/NOTIFY), 검색 엔진의 전문 검색 능력(Full-Text Search), 지리정보 시스템의 공간 분석 능력(PostGIS), 그리고 심지어 외부 데이터 소스 통합(Foreign Data Wrappers)까지 제공한다. 이 모든 기능이 하나의 시스템 안에 통합되어 있다는 것은 단순히 편리함을 넘어서, 아키텍처 복잡성을 근본적으로 줄일 수 있다는 의미다.

더 중요한 것은 데이터 무결성이다. MySQL과 비교할 때 PostgreSQL이 가지는 가장 큰 차별점은 ACID 원칙에 대한 엄격한 준수다. 이는 단순히 이론적인 우수성이 아니라, 실제 프로덕션 환경에서 데이터 손실과 불일치를 방지하는 강력한 방어선이다. 특히 금융, 의료, 전자상거래 등 데이터 정확성이 중요한 분야에서 PostgreSQL의 선택은 거의 필수에 가깝다.

이 문서는 PostgreSQL의 고급 기능들을 실제 코드 예제와 함께 심층적으로 탐구한다. JSONB를 활용한 반정형 데이터 관리, Row Level Security를 통한 데이터베이스 레벨 보안, LISTEN/NOTIFY를 활용한 실시간 통신, 그리고 다양한 확장 기능들의 실용적 활용법을 다룬다. 각 섹션은 이론적 배경, 실제 구현 코드, 성능 최적화 팁, 그리고 실무에서 마주칠 수 있는 함정과 해결책을 포함한다.

## ACID: 데이터 신뢰성의 황금 표준

### ACID 원칙의 실제 의미

ACID는 Atomicity(원자성), Consistency(일관성), Isolation(격리성), Durability(영속성)의 약자로, 데이터베이스 트랜잭션의 신뢰성을 보장하는 핵심 원칙이다. 이론적으로는 간단해 보이지만, 실제 구현에서는 많은 데이터베이스가 성능을 위해 이러한 원칙을 부분적으로 완화한다. PostgreSQL은 이 원칙을 타협 없이 구현하는 것으로 유명하다.

**원자성(Atomicity)** - 트랜잭션의 모든 작업이 완전히 성공하거나 완전히 실패하는 것을 보장한다. 중간 상태는 존재하지 않는다. 예를 들어 은행 계좌 이체를 생각해보자. A 계좌에서 돈을 빼고 B 계좌에 넣는 두 작업은 하나의 원자적 단위로 처리되어야 한다. 중간에 시스템이 다운되더라도 두 작업이 모두 실행되거나 모두 취소되어야 한다.

```sql
BEGIN;

-- A 계좌에서 10만원 출금
UPDATE accounts 
SET balance = balance - 100000 
WHERE account_id = 'A123';

-- 잔액이 부족하면 전체 트랜잭션 롤백
-- B 계좌에 10만원 입금
UPDATE accounts 
SET balance = balance + 100000 
WHERE account_id = 'B456';

-- 모든 작업이 성공했을 때만 커밋
COMMIT;
```

중간에 오류가 발생하면 `ROLLBACK`이 자동으로 실행되어 A 계좌의 잔액도 원래대로 복구된다. 이것이 원자성이다.

**일관성(Consistency)** - 데이터베이스는 정의된 규칙(제약 조건)을 항상 만족해야 한다. PostgreSQL은 이를 스키마 레벨에서 강제할 수 있는 다양한 메커니즘을 제공한다.

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(id),
    total_amount NUMERIC(10, 2) NOT NULL,
    quantity INT NOT NULL,
    order_date TIMESTAMP DEFAULT NOW(),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    
    -- 제약 조건: 주문 금액은 양수여야 함
    CONSTRAINT positive_amount CHECK (total_amount > 0),
    
    -- 제약 조건: 수량은 0보다 커야 함
    CONSTRAINT positive_quantity CHECK (quantity > 0),
    
    -- 제약 조건: 상태는 특정 값만 가능
    CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);
```

이러한 제약 조건들은 애플리케이션 코드에 의존하지 않고 데이터베이스 레벨에서 강제된다. 어떤 프로그래밍 언어로 작성된 어떤 애플리케이션이든, 이 규칙을 위반하는 데이터는 절대 삽입될 수 없다.

**격리성(Isolation)** - 동시에 실행되는 트랜잭션들이 서로 간섭하지 않도록 보장한다. PostgreSQL은 MVCC(Multi-Version Concurrency Control)를 사용하여 높은 동시성을 유지하면서도 격리성을 보장한다.

```sql
-- 세션 1
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT balance FROM accounts WHERE account_id = 'A123';
-- balance = 1000000

-- 세션 2 (동시에 실행)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT balance FROM accounts WHERE account_id = 'A123';
-- balance = 1000000 (동일한 스냅샷을 봄)

-- 세션 1
UPDATE accounts SET balance = balance - 100000 WHERE account_id = 'A123';
COMMIT;

-- 세션 2
UPDATE accounts SET balance = balance - 200000 WHERE account_id = 'A123';
-- 직렬화 오류 발생! 다른 트랜잭션이 이미 이 행을 수정했음
-- ERROR: could not serialize access due to concurrent update
```

SERIALIZABLE 격리 수준에서는 동시 실행되는 트랜잭션들이 마치 순차적으로 실행된 것처럼 동작한다. 이를 통해 Lost Update, Dirty Read, Non-Repeatable Read, Phantom Read 같은 동시성 문제를 완벽하게 방지한다.

**영속성(Durability)** - 커밋된 트랜잭션의 결과는 시스템 장애가 발생하더라도 영구적으로 보존된다. PostgreSQL은 WAL(Write-Ahead Logging)을 사용하여 이를 보장한다.

### MySQL과의 근본적인 차이

MySQL(특히 InnoDB 엔진)도 ACID를 지원한다고 주장하지만, PostgreSQL과는 몇 가지 중요한 차이가 있다.

첫째, 데이터 타입의 엄격성이다. PostgreSQL은 데이터 타입 변환을 매우 엄격하게 처리한다.

```sql
-- PostgreSQL
CREATE TABLE test (id INT);
INSERT INTO test VALUES ('123');  -- 오류! 문자열을 정수로 자동 변환하지 않음

-- MySQL (기본 설정)
CREATE TABLE test (id INT);
INSERT INTO test VALUES ('123');  -- 경고만 발생하고 삽입됨
INSERT INTO test VALUES ('abc');  -- 0으로 변환되어 삽입됨!
```

둘째, NULL 처리의 일관성이다. PostgreSQL은 SQL 표준에 더 충실하게 NULL을 처리한다.

```sql
-- PostgreSQL
SELECT NULL = NULL;  -- NULL (알 수 없음)
SELECT NULL <> NULL; -- NULL (알 수 없음)

-- MySQL
SELECT NULL = NULL;  -- NULL (표준 준수)
SELECT NULL <=> NULL; -- 1 (MySQL 고유 연산자)
```

셋째, 트랜잭션 격리 수준의 기본값이다. PostgreSQL은 기본적으로 READ COMMITTED를 사용하지만, SERIALIZABLE까지 완벽하게 구현한다. MySQL은 REPEATABLE READ를 기본으로 사용하지만, SERIALIZABLE 구현이 완벽하지 않다.

## JSONB: NoSQL의 유연성과 SQL의 강력함을 동시에

### JSON vs JSONB: 왜 B가 Better인가

PostgreSQL은 두 가지 JSON 데이터 타입을 제공한다: `json`과 `jsonb`. 이름의 'B'는 'Binary'를 의미하며, 실제로도 'Better'라고 불릴 만큼 성능과 기능 면에서 우수하다.

**json 타입**은 입력된 텍스트를 그대로 저장한다. 공백, 키의 순서, 중복 키까지 모두 보존된다. 매번 쿼리할 때마다 JSON 텍스트를 파싱해야 하므로 느리다.

**jsonb 타입**은 JSON을 분해하여 바이너리 형식으로 저장한다. 삽입 시 약간의 오버헤드가 있지만, 쿼리 시 훨씬 빠르다. 키의 순서는 보존되지 않고, 중복 키는 제거되며(마지막 값만 유지), 공백은 제거된다.

```sql
-- json vs jsonb 비교
CREATE TABLE json_test (
    id SERIAL PRIMARY KEY,
    data_json JSON,
    data_jsonb JSONB
);

INSERT INTO json_test (data_json, data_jsonb) VALUES (
    '{"name": "Alice", "age": 30, "age": 31}',
    '{"name": "Alice", "age": 30, "age": 31}'
);

SELECT data_json, data_jsonb FROM json_test;
-- data_json:  {"name": "Alice", "age": 30, "age": 31}  (중복 키 유지)
-- data_jsonb: {"age": 31, "name": "Alice"}  (마지막 값만 유지, 순서 변경 가능)
```

실무에서는 거의 항상 jsonb를 사용해야 한다. json은 입력 포맷을 정확히 보존해야 하는 매우 특수한 경우에만 사용한다.

### JSONB의 실전 활용: 전자상거래 제품 관리

전자상거래 플랫폼을 구축한다고 가정하자. 제품은 카테고리마다 완전히 다른 속성을 가진다. 의류는 사이즈와 색상이 필요하고, 전자제품은 사양과 호환성 정보가 필요하며, 도서는 ISBN과 저자 정보가 필요하다. 전통적인 관계형 스키마로는 이를 EAV(Entity-Attribute-Value) 패턴으로 복잡하게 구현해야 하지만, JSONB를 사용하면 우아하게 해결할 수 있다.

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    base_stock INT NOT NULL DEFAULT 0,
    
    -- 카테고리별로 다른 속성을 JSONB로 저장
    attributes JSONB NOT NULL DEFAULT '{}',
    
    -- 검색과 필터링을 위한 인덱스
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 의류 제품 삽입
INSERT INTO products (name, category, price, base_stock, attributes) VALUES
(
    'Classic Denim Jacket',
    'clothing',
    89.99,
    100,
    '{
        "brand": "Levi''s",
        "sizes": ["XS", "S", "M", "L", "XL", "XXL"],
        "colors": [
            {"name": "Blue Denim", "hex": "#1560BD", "stock": 50},
            {"name": "Black", "hex": "#000000", "stock": 50}
        ],
        "material": "100% Cotton",
        "care_instructions": "Machine wash cold, tumble dry low",
        "fit": "Regular fit"
    }'
);

-- 전자제품 삽입
INSERT INTO products (name, category, price, base_stock, attributes) VALUES
(
    'Wireless Noise-Cancelling Headphones',
    'electronics',
    299.99,
    50,
    '{
        "brand": "Sony",
        "model": "WH-1000XM5",
        "specifications": {
            "battery_life_hours": 30,
            "charging_time_hours": 3.5,
            "bluetooth_version": "5.2",
            "weight_grams": 250,
            "driver_size_mm": 40
        },
        "features": [
            "Active Noise Cancellation",
            "Multipoint Connection",
            "Speak-to-Chat",
            "Adaptive Sound Control"
        ],
        "colors_available": ["Black", "Silver", "Midnight Blue"],
        "warranty_months": 12,
        "compatibility": ["Android", "iOS", "Windows", "macOS"]
    }'
);

-- 도서 삽입
INSERT INTO products (name, category, price, base_stock, attributes) VALUES
(
    'Clean Code: A Handbook of Agile Software Craftsmanship',
    'books',
    42.99,
    200,
    '{
        "isbn_13": "978-0132350884",
        "author": "Robert C. Martin",
        "publisher": "Prentice Hall",
        "publication_date": "2008-08-01",
        "pages": 464,
        "language": "English",
        "format": "Hardcover",
        "dimensions": {
            "length_cm": 23.4,
            "width_cm": 17.8,
            "height_cm": 3.2,
            "weight_grams": 980
        },
        "topics": [
            "Software Development",
            "Best Practices",
            "Agile",
            "Code Quality"
        ]
    }'
);
```

### JSONB 쿼리: 강력한 연산자들

JSONB의 진정한 힘은 쿼리 능력에 있다. PostgreSQL은 JSONB를 다루기 위한 풍부한 연산자와 함수를 제공한다.

```sql
-- 1. 특정 속성으로 제품 찾기
-- 예: "brand"가 "Sony"인 모든 제품
SELECT product_id, name, price
FROM products
WHERE attributes->>'brand' = 'Sony';

-- 2. 중첩된 속성 접근
-- 예: 배터리 수명이 25시간 이상인 헤드폰
SELECT name, price, attributes->'specifications'->>'battery_life_hours' as battery_life
FROM products
WHERE category = 'electronics'
  AND (attributes->'specifications'->>'battery_life_hours')::INT >= 25;

-- 3. 배열 포함 여부 확인 (@> 연산자)
-- 예: "Active Noise Cancellation" 기능이 있는 제품
SELECT name, price
FROM products
WHERE attributes->'features' @> '["Active Noise Cancellation"]';

-- 4. 키 존재 여부 확인 (? 연산자)
-- 예: "warranty_months" 속성이 있는 제품
SELECT name, attributes->>'warranty_months' as warranty
FROM products
WHERE attributes ? 'warranty_months';

-- 5. 여러 키 중 하나라도 존재 (?| 연산자)
SELECT name, category
FROM products
WHERE attributes ?| array['isbn_13', 'author'];  -- 도서 제품 찾기

-- 6. 모든 키가 존재 (?& 연산자)
SELECT name
FROM products
WHERE attributes ?& array['brand', 'model'];  -- 브랜드와 모델 모두 있는 제품

-- 7. JSONB 경로 쿼리 (복잡한 중첩 구조)
SELECT name, 
       jsonb_path_query(attributes, '$.colors[*].name') as available_colors
FROM products
WHERE category = 'clothing';

-- 8. JSONB 집계 함수
-- 예: 각 카테고리별 평균 가격과 총 재고
SELECT 
    category,
    COUNT(*) as product_count,
    AVG(price) as avg_price,
    SUM(base_stock) as total_stock,
    jsonb_agg(attributes->'brand') FILTER (WHERE attributes ? 'brand') as brands
FROM products
GROUP BY category;
```

### JSONB 인덱싱: 성능의 핵심

JSONB의 쿼리 성능을 극대화하려면 적절한 인덱싱이 필수다. PostgreSQL은 JSONB를 위한 여러 인덱스 타입을 제공한다.

**1. GIN 인덱스 (Generalized Inverted Index)**

GIN 인덱스는 JSONB의 모든 키와 값을 인덱싱한다. 가장 범용적으로 사용되며, `@>`, `?`, `?|`, `?&` 연산자를 지원한다.

```sql
-- 기본 GIN 인덱스 생성
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- 이제 이 쿼리가 인덱스를 사용함
EXPLAIN ANALYZE
SELECT * FROM products 
WHERE attributes @> '{"brand": "Sony"}';

-- 실행 계획에 "Bitmap Index Scan on idx_products_attributes" 표시됨
```

GIN 인덱스에는 두 가지 연산자 클래스가 있다:

- **jsonb_ops** (기본값): 모든 키와 값을 인덱싱. 더 큰 인덱스 크기, 더 많은 쿼리 지원
- **jsonb_path_ops**: 경로만 인덱싱. 작은 인덱스 크기, 빠른 검색, `@>` 연산자만 지원

```sql
-- jsonb_path_ops 사용 (더 작고 빠른 인덱스)
CREATE INDEX idx_products_attrs_path ON products USING GIN (attributes jsonb_path_ops);

-- 이 인덱스는 @> 연산자에만 사용됨
SELECT * FROM products WHERE attributes @> '{"brand": "Levi''s"}';
```

**2. 표현식 인덱스 (Expression Index)**

특정 키에 대한 쿼리가 빈번하다면, 해당 키만 추출하여 B-tree 인덱스를 만드는 것이 더 효율적일 수 있다.

```sql
-- 브랜드별 검색을 위한 표현식 인덱스
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));

-- 이제 브랜드 검색이 매우 빠름
EXPLAIN ANALYZE
SELECT * FROM products WHERE attributes->>'brand' = 'Sony';
-- Index Scan using idx_products_brand

-- 가격 범위 검색을 위한 복합 인덱스
CREATE INDEX idx_products_price_brand ON products (price, (attributes->>'brand'));

SELECT * FROM products 
WHERE price BETWEEN 100 AND 500
  AND attributes->>'brand' = 'Sony';
```

**3. 부분 인덱스 (Partial Index)**

특정 조건을 만족하는 행만 인덱싱하여 인덱스 크기를 줄이고 성능을 향상시킬 수 있다.

```sql
-- 전자제품 카테고리의 브랜드만 인덱싱
CREATE INDEX idx_electronics_brand ON products ((attributes->>'brand'))
WHERE category = 'electronics';

-- 재고가 있는 제품만 인덱싱
CREATE INDEX idx_available_products ON products USING GIN (attributes)
WHERE base_stock > 0;
```

### JSONB 업데이트: 효율적인 수정 방법

JSONB 데이터를 수정할 때는 전체를 교체하는 것보다 부분 수정이 더 효율적이다.

```sql
-- 나쁜 방법: 전체 JSONB를 다시 구성
UPDATE products
SET attributes = '{
    "brand": "Sony",
    "model": "WH-1000XM5",
    "specifications": {...},
    "price_updated": true
}'
WHERE product_id = 2;

-- 좋은 방법: jsonb_set 함수로 특정 키만 업데이트
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{price_updated}',
    'true'::jsonb
)
WHERE product_id = 2;

-- 중첩된 값 업데이트
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{specifications, battery_life_hours}',
    '35'::jsonb
)
WHERE attributes->>'model' = 'WH-1000XM5';

-- 여러 키 동시 업데이트 (|| 연산자)
UPDATE products
SET attributes = attributes || '{
    "last_updated": "2026-01-29",
    "featured": true
}'::jsonb
WHERE category = 'electronics';

-- 키 삭제 (- 연산자)
UPDATE products
SET attributes = attributes - 'discontinued'
WHERE product_id = 1;

-- 배열에 요소 추가
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{colors}',
    (attributes->'colors')::jsonb || '[{"name": "Red", "hex": "#FF0000", "stock": 30}]'::jsonb
)
WHERE product_id = 1;
```

### JSONB 모범 사례와 주의사항

**DO: 이렇게 사용하세요**

1. **핫 키(자주 쿼리되는 키)는 일반 컬럼으로 승격**
```sql
-- JSONB에 brand를 넣는 대신
ALTER TABLE products ADD COLUMN brand VARCHAR(100);
CREATE INDEX idx_products_brand ON products(brand);

-- 자주 변하지 않는 메타데이터는 JSONB에
-- 자주 쿼리되는 필드는 일반 컬럼에
```

2. **생성 컬럼(Generated Column) 활용**
```sql
-- PostgreSQL 12+에서 사용 가능
ALTER TABLE products 
ADD COLUMN brand_extracted VARCHAR(100) 
GENERATED ALWAYS AS (attributes->>'brand') STORED;

CREATE INDEX idx_brand_extracted ON products(brand_extracted);

-- 이제 JSONB를 쿼리하는 것처럼 보이지만 실제로는 일반 컬럼 사용
SELECT * FROM products WHERE brand_extracted = 'Sony';
```

3. **적절한 제약 조건 설정**
```sql
-- JSONB 스키마 검증을 위한 CHECK 제약
ALTER TABLE products 
ADD CONSTRAINT valid_attributes_schema CHECK (
    -- 카테고리별로 필수 키 검증
    CASE 
        WHEN category = 'electronics' THEN 
            attributes ? 'brand' AND 
            attributes ? 'model' AND
            attributes ? 'specifications'
        WHEN category = 'clothing' THEN
            attributes ? 'sizes' AND 
            attributes ? 'colors'
        WHEN category = 'books' THEN
            attributes ? 'isbn_13' AND
            attributes ? 'author'
        ELSE true
    END
);
```

**DON'T: 이렇게 사용하지 마세요**

1. **거대한 JSONB 문서 저장 금지**
```sql
-- 나쁜 예: 2KB 이상의 큰 JSONB (TOASTing 발생)
-- 특히 전체 주문 히스토리를 하나의 JSONB에 저장하는 것은 최악
INSERT INTO customers (id, order_history) VALUES 
(1, '{...수백 개의 주문...}');  -- 절대 금지!

-- 좋은 예: 정규화된 테이블 사용
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_data JSONB  -- 개별 주문의 메타데이터만
);
```

2. **JSONB를 관계의 대체로 사용 금지**
```sql
-- 나쁜 예
CREATE TABLE blog_posts (
    id SERIAL PRIMARY KEY,
    content TEXT,
    comments JSONB  -- [{author: "...", text: "..."}]
);

-- 좋은 예
CREATE TABLE blog_posts (
    id SERIAL PRIMARY KEY,
    content TEXT
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INT REFERENCES blog_posts(id),
    author_id INT REFERENCES users(id),
    content TEXT,
    metadata JSONB  -- 추가적인 비정형 데이터만
);
```

3. **과도한 인덱싱 금지**
```sql
-- 나쁜 예: 모든 것을 인덱싱
CREATE INDEX idx1 ON products USING GIN (attributes);
CREATE INDEX idx2 ON products ((attributes->>'brand'));
CREATE INDEX idx3 ON products ((attributes->>'model'));
CREATE INDEX idx4 ON products ((attributes->'specifications'));
-- ... 수십 개의 인덱스

-- 좋은 예: 실제 쿼리 패턴 분석 후 필요한 것만
CREATE INDEX idx_attrs ON products USING GIN (attributes jsonb_path_ops);
-- 그리고 가장 자주 쿼리되는 1-2개만 표현식 인덱스
```

### JSONB 성능 모니터링

```sql
-- JSONB 쿼리 성능 분석
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM products 
WHERE attributes @> '{"brand": "Sony"}';

-- 인덱스 사용 통계 확인
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
WHERE tablename = 'products'
ORDER BY idx_scan DESC;

-- 사용되지 않는 인덱스 찾기
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE 'pg_%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Row Level Security (RLS): 데이터베이스 레벨의 완벽한 보안

### RLS가 해결하는 문제

전통적으로 데이터 접근 제어는 애플리케이션 코드에서 처리했다. 사용자가 자신의 데이터만 볼 수 있도록 하기 위해 모든 쿼리에 `WHERE user_id = current_user_id` 같은 조건을 추가했다. 하지만 이 접근 방식에는 치명적인 문제가 있다.

첫째, **개발자의 실수로 보안 사고 발생**. 수백 개의 쿼리 중 하나라도 WHERE 절을 빠뜨리면 데이터 유출이 발생한다. 특히 급하게 개발하거나 여러 개발자가 참여하는 프로젝트에서 이런 실수는 불가피하다.

둘째, **일관성 유지의 어려움**. 같은 테이블에 접근하는 여러 API 엔드포인트마다 동일한 필터링 로직을 반복해야 한다. 비즈니스 규칙이 변경되면 모든 곳을 수정해야 한다.

셋째, **데이터베이스 직접 접근 시 무방비**. 관리자 도구, 분석 쿼리, 레거시 시스템 등에서 데이터베이스에 직접 접근할 때 애플리케이션 레벨 보안은 무용지물이다.

PostgreSQL의 Row Level Security는 이 모든 문제를 해결한다. 보안 정책을 테이블 정의에 포함시켜, 어떤 방식으로 접근하든 자동으로 적용되도록 한다.

### RLS 기본 구현: 멀티테넌트 SaaS

실제 멀티테넌트 SaaS 애플리케이션을 구축한다고 가정하자. 각 회사(tenant)는 자신의 데이터만 볼 수 있어야 한다.

```sql
-- 1. 테넌트 테이블 생성
CREATE TABLE tenants (
    tenant_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    subscription_tier VARCHAR(20) CHECK (subscription_tier IN ('free', 'pro', 'enterprise'))
);

-- 2. 프로젝트 테이블 (테넌트별 데이터)
CREATE TABLE projects (
    project_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id),
    project_name VARCHAR(255) NOT NULL,
    description TEXT,
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'archived', 'deleted'))
);

-- 3. 태스크 테이블
CREATE TABLE tasks (
    task_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(project_id),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id), -- 비정규화하여 성능 향상
    title VARCHAR(500) NOT NULL,
    description TEXT,
    assigned_to UUID,
    status VARCHAR(20) DEFAULT 'pending',
    priority VARCHAR(10) CHECK (priority IN ('low', 'medium', 'high', 'critical')),
    due_date DATE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- 4. RLS 활성화
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- 5. 테넌트 격리 정책 생성
-- 현재 세션의 tenant_id와 일치하는 행만 접근 가능
CREATE POLICY tenant_isolation_policy ON projects
    FOR ALL  -- SELECT, INSERT, UPDATE, DELETE 모두 적용
    TO PUBLIC  -- 모든 사용자에게 적용
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
    WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);

CREATE POLICY tenant_isolation_policy ON tasks
    FOR ALL
    TO PUBLIC
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
    WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- 테넌트 테이블 자체는 자신의 정보만 볼 수 있음
CREATE POLICY tenant_self_policy ON tenants
    FOR ALL
    TO PUBLIC
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

이제 애플리케이션 코드에서는 다음과 같이 사용한다:

```javascript
// Node.js with pg library 예제
const { Pool } = require('pg');
const pool = new Pool({ /* 연결 설정 */ });

async function getProjectsForTenant(tenantId) {
    const client = await pool.connect();
    try {
        // 세션 변수 설정 (중요!)
        await client.query('SET app.current_tenant_id = $1', [tenantId]);
        
        // 이제 단순한 쿼리만 작성하면 됨
        // WHERE tenant_id = ... 조건 불필요!
        const result = await client.query(`
            SELECT project_id, project_name, status, created_at
            FROM projects
            WHERE status = 'active'
            ORDER BY created_at DESC
        `);
        
        return result.rows;
    } finally {
        client.release();
    }
}

async function createTask(tenantId, taskData) {
    const client = await pool.connect();
    try {
        await client.query('BEGIN');
        await client.query('SET app.current_tenant_id = $1', [tenantId]);
        
        // RLS가 자동으로 tenant_id를 검증
        // 다른 테넌트의 project_id를 지정해도 삽입 실패!
        const result = await client.query(`
            INSERT INTO tasks (project_id, title, description, assigned_to, priority)
            VALUES ($1, $2, $3, $4, $5)
            RETURNING *
        `, [taskData.project_id, taskData.title, taskData.description, 
            taskData.assigned_to, taskData.priority]);
        
        await client.query('COMMIT');
        return result.rows[0];
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
}
```

### RLS 고급 패턴: 역할 기반 접근 제어

단순한 테넌트 격리를 넘어 역할 기반 접근 제어(RBAC)를 구현해보자.

```sql
-- 사용자 테이블
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id),
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL CHECK (role IN ('owner', 'admin', 'member', 'viewer')),
    created_at TIMESTAMP DEFAULT NOW()
);

ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 프로젝트 멤버십 테이블
CREATE TABLE project_members (
    project_id UUID REFERENCES projects(project_id),
    user_id UUID REFERENCES users(user_id),
    role VARCHAR(20) NOT NULL CHECK (role IN ('manager', 'contributor', 'viewer')),
    added_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (project_id, user_id)
);

ALTER TABLE project_members ENABLE ROW LEVEL SECURITY;

-- 복잡한 RLS 정책: 프로젝트 접근 권한
CREATE POLICY project_access_policy ON projects
    FOR SELECT
    TO PUBLIC
    USING (
        -- 같은 테넌트이고
        tenant_id = current_setting('app.current_tenant_id')::UUID
        AND (
            -- 테넌트 소유자/관리자이거나
            EXISTS (
                SELECT 1 FROM users
                WHERE users.user_id = current_setting('app.current_user_id')::UUID
                  AND users.tenant_id = projects.tenant_id
                  AND users.role IN ('owner', 'admin')
            )
            -- 또는 프로젝트 멤버
            OR EXISTS (
                SELECT 1 FROM project_members
                WHERE project_members.project_id = projects.project_id
                  AND project_members.user_id = current_setting('app.current_user_id')::UUID
            )
        )
    );

-- 프로젝트 수정 권한 (더 제한적)
CREATE POLICY project_modify_policy ON projects
    FOR UPDATE
    TO PUBLIC
    USING (
        tenant_id = current_setting('app.current_tenant_id')::UUID
        AND (
            -- 테넌트 소유자/관리자이거나
            EXISTS (
                SELECT 1 FROM users
                WHERE users.user_id = current_setting('app.current_user_id')::UUID
                  AND users.tenant_id = projects.tenant_id
                  AND users.role IN ('owner', 'admin')
            )
            -- 또는 프로젝트 매니저
            OR EXISTS (
                SELECT 1 FROM project_members
                WHERE project_members.project_id = projects.project_id
                  AND project_members.user_id = current_setting('app.current_user_id')::UUID
                  AND project_members.role = 'manager'
            )
        )
    );

-- 태스크에 대한 세밀한 접근 제어
CREATE POLICY task_view_policy ON tasks
    FOR SELECT
    TO PUBLIC
    USING (
        tenant_id = current_setting('app.current_tenant_id')::UUID
        AND (
            -- 프로젝트 접근 권한이 있으면 태스크도 볼 수 있음
            EXISTS (
                SELECT 1 FROM projects
                WHERE projects.project_id = tasks.project_id
                -- 여기서 projects 테이블의 RLS 정책이 자동 적용됨!
            )
        )
    );

-- 태스크 수정은 할당받은 사람 또는 프로젝트 매니저만
CREATE POLICY task_modify_policy ON tasks
    FOR UPDATE
    TO PUBLIC
    USING (
        tenant_id = current_setting('app.current_tenant_id')::UUID
        AND (
            -- 자신에게 할당된 태스크
            assigned_to = current_setting('app.current_user_id')::UUID
            -- 또는 프로젝트 매니저
            OR EXISTS (
                SELECT 1 FROM project_members
                WHERE project_members.project_id = tasks.project_id
                  AND project_members.user_id = current_setting('app.current_user_id')::UUID
                  AND project_members.role = 'manager'
            )
        )
    );
```

### RLS 성능 최적화: 반드시 알아야 할 것들

RLS는 강력하지만 잘못 사용하면 성능 문제를 일으킬 수 있다.

**1. RLS 정책에 사용되는 컬럼은 반드시 인덱싱**

```sql
-- 나쁜 예: 인덱스 없음
CREATE POLICY tenant_policy ON projects
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
-- 전체 테이블 스캔 발생!

-- 좋은 예: 인덱스 추가
CREATE INDEX idx_projects_tenant ON projects(tenant_id);
CREATE INDEX idx_tasks_tenant ON tasks(tenant_id);

-- 복합 인덱스로 더 최적화
CREATE INDEX idx_tasks_tenant_status ON tasks(tenant_id, status)
WHERE status != 'deleted';
```

**2. LEAKPROOF 함수 사용 (보안과 성능 동시 확보)**

```sql
-- 일반 함수는 RLS 정책 적용 후 실행됨 (느림)
CREATE FUNCTION is_project_manager(p_project_id UUID, p_user_id UUID)
RETURNS BOOLEAN AS $$
    SELECT EXISTS (
        SELECT 1 FROM project_members
        WHERE project_id = p_project_id
          AND user_id = p_user_id
          AND role = 'manager'
    );
$$ LANGUAGE SQL;

-- LEAKPROOF 함수는 인덱스 사용 가능 (빠름)
CREATE FUNCTION is_project_manager(p_project_id UUID, p_user_id UUID)
RETURNS BOOLEAN AS $$
    SELECT EXISTS (
        SELECT 1 FROM project_members
        WHERE project_id = p_project_id
          AND user_id = p_user_id
          AND role = 'manager'
    );
$$ LANGUAGE SQL LEAKPROOF;
-- LEAKPROOF: 이 함수는 입력 외에 다른 정보를 유출하지 않음을 보증
```

**3. FORCE ROW LEVEL SECURITY로 소유자 우회 방지**

```sql
-- 기본적으로 테이블 소유자와 superuser는 RLS를 우회함
-- 이는 테스트 시 혼란을 줄 수 있음

-- 소유자도 RLS를 따르도록 강제
ALTER TABLE projects FORCE ROW LEVEL SECURITY;
ALTER TABLE tasks FORCE ROW LEVEL SECURITY;

-- 테스트용 일반 사용자 생성
CREATE ROLE app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- 이 사용자로 테스트
SET ROLE app_user;
SET app.current_tenant_id = 'some-tenant-uuid';
SELECT * FROM projects;  -- RLS 정책이 적용됨
RESET ROLE;
```

**4. PERMISSIVE vs RESTRICTIVE 정책**

```sql
-- PERMISSIVE 정책 (기본값): OR로 결합
CREATE POLICY owner_access ON projects
    FOR ALL
    TO PUBLIC
    USING (created_by = current_setting('app.current_user_id')::UUID);

CREATE POLICY member_access ON projects
    FOR ALL
    TO PUBLIC
    USING (
        EXISTS (
            SELECT 1 FROM project_members
            WHERE project_id = projects.project_id
              AND user_id = current_setting('app.current_user_id')::UUID
        )
    );
-- 두 정책 중 하나라도 만족하면 접근 허용 (owner OR member)

-- RESTRICTIVE 정책: AND로 결합
CREATE POLICY active_projects_only ON projects
    FOR SELECT
    TO PUBLIC
    USING (status = 'active')
    AS RESTRICTIVE;  -- 명시적으로 RESTRICTIVE 지정
-- 다른 정책을 만족하더라도 status = 'active'여야만 함
```

### RLS 함정과 해결책

**함정 1: 뷰(View)는 기본적으로 RLS를 우회함**

```sql
-- 나쁜 예: 뷰는 security definer로 생성됨 (superuser 권한으로 실행)
CREATE VIEW active_projects AS
SELECT * FROM projects WHERE status = 'active';
-- 이 뷰를 조회하면 RLS가 적용되지 않음!

-- 좋은 예 (PostgreSQL 15+): security_invoker 옵션
CREATE VIEW active_projects WITH (security_invoker = true) AS
SELECT * FROM projects WHERE status = 'active';
-- 이제 뷰를 조회하는 사용자의 권한으로 RLS 적용됨

-- 또는 함수 사용
CREATE FUNCTION get_active_projects()
RETURNS SETOF projects
SECURITY INVOKER  -- 호출자의 권한으로 실행
AS $$
    SELECT * FROM projects WHERE status = 'active';
$$ LANGUAGE SQL;
```

**함정 2: 연결 풀링 시 세션 변수 관리**

```javascript
// 나쁜 예: 세션 변수가 연결에 남아있음
async function badExample() {
    const client = await pool.connect();
    await client.query('SET app.current_tenant_id = $1', [tenant1]);
    await client.query('SELECT * FROM projects');  // tenant1 데이터
    client.release();  // 세션 변수가 그대로!
    
    // 다음 요청에서 같은 연결을 재사용할 수 있음
    const client2 = await pool.connect();
    // client2 === client일 수 있음 (연결 풀)
    await client2.query('SELECT * FROM projects');  // 여전히 tenant1 데이터!
}

// 좋은 예: 트랜잭션 사용 또는 명시적 리셋
async function goodExample() {
    const client = await pool.connect();
    try {
        await client.query('BEGIN');
        await client.query('SET LOCAL app.current_tenant_id = $1', [tenant1]);
        // SET LOCAL은 트랜잭션 종료 시 자동 리셋
        
        const result = await client.query('SELECT * FROM projects');
        await client.query('COMMIT');
        return result.rows;
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
}

// 또는 연결 해제 시 명시적 리셋
pool.on('remove', (client) => {
    client.query('RESET ALL');
});
```

**함정 3: WITH CHECK와 USING의 차이 이해 부족**

```sql
-- USING: 기존 행을 읽거나 수정할 때의 조건
-- WITH CHECK: 새로운 행을 삽입하거나 기존 행을 수정한 결과가 만족해야 하는 조건

-- 잘못된 정책: WITH CHECK 없음
CREATE POLICY bad_policy ON projects
    FOR ALL
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
-- 문제: 다른 테넌트의 데이터를 삽입할 수 있음!

-- 올바른 정책: WITH CHECK 포함
CREATE POLICY good_policy ON projects
    FOR ALL
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID)
    WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- 또는 다르게 설정 가능 (읽기는 자유롭게, 쓰기는 제한적으로)
CREATE POLICY read_policy ON projects
    FOR SELECT
    USING (true);  -- 모든 프로젝트를 볼 수 있음

CREATE POLICY write_policy ON projects
    FOR INSERT
    WITH CHECK (
        tenant_id = current_setting('app.current_tenant_id')::UUID
        AND created_by = current_setting('app.current_user_id')::UUID
    );
```

## LISTEN/NOTIFY: 내장 메시지 브로커

### 왜 별도의 메시지 큐가 필요 없는가

실시간 기능을 구현할 때 개발자들은 자동으로 Redis Pub/Sub, RabbitMQ, Kafka 같은 전용 메시지 브로커를 떠올린다. 하지만 PostgreSQL의 LISTEN/NOTIFY는 많은 경우 이러한 외부 시스템을 대체할 수 있다. 특히 다음과 같은 시나리오에서 유용하다:

- 내부 대시보드의 실시간 업데이트
- 캐시 무효화 신호
- 백그라운드 작업 트리거
- WebSocket을 통한 클라이언트 알림
- 데이터베이스 변경에 대한 마이크로서비스 간 통신

LISTEN/NOTIFY의 장점은 명확하다. 추가 인프라가 필요 없고, 데이터베이스와 메시지 시스템 간 동기화 문제가 없으며, 트랜잭션 내에서 동작하여 일관성이 보장된다.

### LISTEN/NOTIFY 기본 메커니즘

PostgreSQL의 LISTEN/NOTIFY는 발행-구독(Pub/Sub) 패턴을 구현한다. 기본 개념은 간단하다:

1. 클라이언트가 특정 채널을 `LISTEN`
2. 다른 세션이 해당 채널에 `NOTIFY` (선택적으로 페이로드 포함)
3. 리스닝 중인 모든 클라이언트가 비동기 알림 수신

```sql
-- 터미널 1: 리스너
LISTEN order_updates;
-- 이제 이 세션은 'order_updates' 채널을 구독함

-- 터미널 2: 발행자
NOTIFY order_updates, '{"order_id": "12345", "status": "shipped"}';
-- JSON 문자열을 페이로드로 전송 (최대 8000바이트)

-- 터미널 1에서 즉시 수신:
-- Asynchronous notification "order_updates" with payload 
-- '{"order_id": "12345", "status": "shipped"}' received from server process with PID 12345
```

### 실전 예제 1: 실시간 주문 대시보드

전자상거래 관리자 대시보드를 구축한다고 가정하자. 새 주문이 들어올 때마다 대시보드를 실시간으로 업데이트해야 한다.

**1. 데이터베이스 트리거 설정**

```sql
-- 주문 테이블
CREATE TABLE orders (
    order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 주문 변경 시 알림을 보내는 트리거 함수
CREATE OR REPLACE FUNCTION notify_order_change()
RETURNS TRIGGER AS $$
DECLARE
    payload JSON;
BEGIN
    -- 변경 내용을 JSON으로 구성
    IF TG_OP = 'INSERT' THEN
        payload = json_build_object(
            'operation', 'INSERT',
            'order_id', NEW.order_id,
            'customer_id', NEW.customer_id,
            'total_amount', NEW.total_amount,
            'status', NEW.status,
            'created_at', NEW.created_at
        );
    ELSIF TG_OP = 'UPDATE' THEN
        payload = json_build_object(
            'operation', 'UPDATE',
            'order_id', NEW.order_id,
            'old_status', OLD.status,
            'new_status', NEW.status,
            'updated_at', NEW.updated_at
        );
    ELSIF TG_OP = 'DELETE' THEN
        payload = json_build_object(
            'operation', 'DELETE',
            'order_id', OLD.order_id
        );
    END IF;
    
    -- 알림 전송
    PERFORM pg_notify('order_updates', payload::text);
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 트리거 등록
CREATE TRIGGER order_change_trigger
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION notify_order_change();

-- 이제 주문이 변경될 때마다 자동으로 알림 전송!
INSERT INTO orders (customer_id, total_amount, status) VALUES
    ('c1234567-89ab-cdef-0123-456789abcdef'::UUID, 199.99, 'pending');
-- 'order_updates' 채널로 알림 전송됨
```

**2. Node.js 백엔드 리스너 (WebSocket 서버)**

```javascript
const { Client } = require('pg');
const WebSocket = require('ws');

// PostgreSQL 클라이언트 (LISTEN 전용 연결)
const pgClient = new Client({
    host: 'localhost',
    database: 'ecommerce',
    user: 'postgres',
    password: 'password'
});

// WebSocket 서버
const wss = new WebSocket.Server({ port: 8080 });
const clients = new Set();

wss.on('connection', (ws) => {
    console.log('New WebSocket client connected');
    clients.add(ws);
    
    ws.on('close', () => {
        clients.delete(ws);
        console.log('Client disconnected');
    });
});

// PostgreSQL 연결 및 LISTEN 시작
async function startListening() {
    try {
        await pgClient.connect();
        console.log('Connected to PostgreSQL');
        
        // LISTEN 명령 실행
        await pgClient.query('LISTEN order_updates');
        console.log('Listening for order updates...');
        
        // 알림 수신 핸들러
        pgClient.on('notification', (msg) => {
            console.log('Received notification:', msg);
            
            try {
                const payload = JSON.parse(msg.payload);
                
                // 모든 WebSocket 클라이언트에게 브로드캐스트
                const message = JSON.stringify({
                    channel: msg.channel,
                    timestamp: new Date().toISOString(),
                    data: payload
                });
                
                clients.forEach((client) => {
                    if (client.readyState === WebSocket.OPEN) {
                        client.send(message);
                    }
                });
                
                console.log(`Broadcasted to ${clients.size} clients`);
            } catch (error) {
                console.error('Error parsing notification:', error);
            }
        });
        
        // 연결 끊김 처리
        pgClient.on('error', async (err) => {
            console.error('PostgreSQL connection error:', err);
            await reconnect();
        });
        
    } catch (error) {
        console.error('Failed to start listening:', error);
        setTimeout(startListening, 5000);  // 5초 후 재시도
    }
}

// 재연결 로직
async function reconnect() {
    console.log('Attempting to reconnect...');
    try {
        await pgClient.end();
    } catch (e) {
        // 이미 연결 끊김
    }
    
    setTimeout(startListening, 5000);
}

// 서버 시작
startListening();

// Graceful shutdown
process.on('SIGINT', async () => {
    console.log('Shutting down...');
    await pgClient.end();
    wss.close();
    process.exit(0);
});
```

**3. 프론트엔드 (React 예제)**

```javascript
import React, { useEffect, useState } from 'react';

function OrderDashboard() {
    const [orders, setOrders] = useState([]);
    const [notifications, setNotifications] = useState([]);
    
    useEffect(() => {
        // WebSocket 연결
        const ws = new WebSocket('ws://localhost:8080');
        
        ws.onopen = () => {
            console.log('Connected to real-time updates');
        };
        
        ws.onmessage = (event) => {
            const notification = JSON.parse(event.data);
            console.log('Received update:', notification);
            
            // 알림 목록에 추가
            setNotifications(prev => [notification, ...prev].slice(0, 10));
            
            // 주문 데이터 업데이트
            if (notification.data.operation === 'INSERT') {
                setOrders(prev => [notification.data, ...prev]);
                
                // 효과음이나 알림 표시
                new Audio('/notification.mp3').play();
                showToast(`New order: $${notification.data.total_amount}`);
            } else if (notification.data.operation === 'UPDATE') {
                setOrders(prev => prev.map(order => 
                    order.order_id === notification.data.order_id
                        ? { ...order, status: notification.data.new_status }
                        : order
                ));
            }
        };
        
        ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };
        
        ws.onclose = () => {
            console.log('WebSocket connection closed');
            // 재연결 로직
            setTimeout(() => window.location.reload(), 5000);
        };
        
        return () => {
            ws.close();
        };
    }, []);
    
    return (
        <div className="dashboard">
            <h1>Live Order Dashboard</h1>
            
            <div className="notifications">
                <h2>Recent Updates</h2>
                {notifications.map((notif, idx) => (
                    <div key={idx} className="notification">
                        <span className="timestamp">{notif.timestamp}</span>
                        <span className="operation">{notif.data.operation}</span>
                        <span className="details">
                            Order {notif.data.order_id?.slice(0, 8)}...
                        </span>
                    </div>
                ))}
            </div>
            
            <div className="orders">
                <h2>Orders ({orders.length})</h2>
                <table>
                    <thead>
                        <tr>
                            <th>Order ID</th>
                            <th>Amount</th>
                            <th>Status</th>
                            <th>Created</th>
                        </tr>
                    </thead>
                    <tbody>
                        {orders.map(order => (
                            <tr key={order.order_id} className={`status-${order.status}`}>
                                <td>{order.order_id.slice(0, 8)}...</td>
                                <td>${order.total_amount}</td>
                                <td><span className="badge">{order.status}</span></td>
                                <td>{new Date(order.created_at).toLocaleString()}</td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            </div>
        </div>
    );
}

function showToast(message) {
    // 토스트 알림 표시 로직
    const toast = document.createElement('div');
    toast.className = 'toast';
    toast.textContent = message;
    document.body.appendChild(toast);
    setTimeout(() => toast.remove(), 3000);
}

export default OrderDashboard;
```

### 실전 예제 2: 캐시 무효화

Redis를 캐시로 사용하는 시스템에서 데이터가 변경될 때 캐시를 자동으로 무효화하는 패턴:

```sql
-- 제품 테이블
CREATE TABLE products (
    product_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    stock_quantity INT NOT NULL,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 캐시 무효화 트리거
CREATE OR REPLACE FUNCTION invalidate_product_cache()
RETURNS TRIGGER AS $$
BEGIN
    -- 제품 캐시 키를 페이로드로 전송
    PERFORM pg_notify(
        'cache_invalidation',
        json_build_object(
            'type', 'product',
            'product_id', NEW.product_id,
            'cache_keys', json_build_array(
                'product:' || NEW.product_id,
                'product:list',
                'product:category:' || NEW.category
            )
        )::text
    );
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_cache_invalidation
    AFTER UPDATE OR DELETE ON products
    FOR EACH ROW
    EXECUTE FUNCTION invalidate_product_cache();
```

Node.js 캐시 무효화 워커:

```javascript
const { Client } = require('pg');
const Redis = require('ioredis');

const pgClient = new Client({ /* config */ });
const redis = new Redis({ /* config */ });

async function startCacheInvalidationWorker() {
    await pgClient.connect();
    await pgClient.query('LISTEN cache_invalidation');
    
    pgClient.on('notification', async (msg) => {
        const { type, product_id, cache_keys } = JSON.parse(msg.payload);
        
        console.log(`Invalidating cache for ${type}:`, cache_keys);
        
        // Redis에서 해당 키들 삭제
        const pipeline = redis.pipeline();
        cache_keys.forEach(key => pipeline.del(key));
        await pipeline.exec();
        
        console.log(`Invalidated ${cache_keys.length} cache keys`);
    });
}

startCacheInvalidationWorker();
```

### LISTEN/NOTIFY의 한계와 대안

LISTEN/NOTIFY는 강력하지만 모든 상황에 적합하지는 않다.

**한계점:**

1. **메시지 영속성 없음** - 리스너가 연결되어 있지 않으면 메시지가 손실됨
2. **보장된 전달 없음** - at-most-once 의미론만 제공
3. **연결당 하나의 리스너** - 각 LISTEN 명령은 하나의 데이터베이스 연결을 차지
4. **페이로드 크기 제한** - 8000바이트까지만 가능
5. **순서 보장 제한** - 같은 트랜잭션 내에서만 순서 보장

**적합한 사용 사례:**
- 내부 대시보드 (손실 허용 가능)
- 캐시 무효화 신호
- 상태 변경 알림
- 낮은 처리량 (<수백 msg/sec)

**부적합한 사용 사례:**
- 금융 거래 (보장된 전달 필요)
- 대규모 실시간 채팅 (>1000 동시 사용자)
- 메시지 큐잉 (영속성 필요)
- 높은 처리량 요구 (>수천 msg/sec)

이러한 경우 RabbitMQ, Kafka, Redis Streams 같은 전용 메시지 브로커를 사용해야 한다.

## Full-Text Search, Foreign Data Wrappers, 그리고 더 많은 확장 기능들

### Full-Text Search: 내장 검색 엔진

PostgreSQL은 강력한 전문 검색 기능을 기본 제공한다. Elasticsearch를 도입하기 전에 PostgreSQL의 전문 검색으로 충분한지 먼저 평가해보라.

```sql
-- 블로그 포스트 테이블
CREATE TABLE blog_posts (
    post_id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    content TEXT NOT NULL,
    author_id INT NOT NULL,
    published_at TIMESTAMP DEFAULT NOW(),
    
    -- tsvector 컬럼 (검색 최적화)
    search_vector tsvector
);

-- 검색 벡터 자동 업데이트 트리거
CREATE OR REPLACE FUNCTION update_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector := 
        setweight(to_tsvector('english', coalesce(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(NEW.content, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tsvector_update
    BEFORE INSERT OR UPDATE ON blog_posts
    FOR EACH ROW
    EXECUTE FUNCTION update_search_vector();

-- GIN 인덱스 생성 (검색 성능 향상)
CREATE INDEX idx_blog_posts_search ON blog_posts USING GIN(search_vector);

-- 샘플 데이터
INSERT INTO blog_posts (title, content, author_id) VALUES
('Getting Started with PostgreSQL', 
 'PostgreSQL is a powerful open-source relational database...', 1),
('Advanced SQL Techniques', 
 'Learn about window functions, CTEs, and recursive queries...', 1),
('NoSQL vs SQL: The Debate Continues',
 'Choosing between NoSQL and SQL databases depends on...', 2);

-- 전문 검색 쿼리
SELECT 
    post_id,
    title,
    ts_rank(search_vector, query) AS rank
FROM blog_posts, 
     to_tsquery('english', 'postgresql & database') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;

-- 구문 검색 (phrase search)
SELECT title, content
FROM blog_posts
WHERE search_vector @@ phraseto_tsquery('english', 'open source database');

-- 하이라이팅 (매칭된 부분 강조)
SELECT 
    title,
    ts_headline('english', content, query, 'MaxWords=20, MinWords=15') AS snippet
FROM blog_posts,
     to_tsquery('english', 'postgresql') AS query
WHERE search_vector @@ query;
```

### Foreign Data Wrappers: 외부 데이터 통합

FDW를 사용하면 외부 데이터베이스나 파일을 마치 로컬 테이블처럼 쿼리할 수 있다.

```sql
-- PostgreSQL FDW 확장 설치
CREATE EXTENSION postgres_fdw;

-- 외부 서버 정의 (다른 PostgreSQL 인스턴스)
CREATE SERVER legacy_db
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'legacy.example.com', dbname 'old_system', port '5432');

-- 사용자 매핑
CREATE USER MAPPING FOR current_user
    SERVER legacy_db
    OPTIONS (user 'readonly', password 'secret');

-- 외부 테이블 가져오기
IMPORT FOREIGN SCHEMA public
    LIMIT TO (customers, orders)
    FROM SERVER legacy_db
    INTO public;

-- 이제 외부 테이블을 로컬 테이블처럼 쿼리 가능!
SELECT c.name, COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;

-- CSV 파일도 직접 쿼리 가능
CREATE EXTENSION file_fdw;

CREATE SERVER csv_server
    FOREIGN DATA WRAPPER file_fdw;

CREATE FOREIGN TABLE sales_data (
    date DATE,
    product VARCHAR(100),
    quantity INT,
    revenue NUMERIC(10, 2)
)
SERVER csv_server
OPTIONS (filename '/data/sales.csv', format 'csv', header 'true');

SELECT 
    product,
    SUM(revenue) as total_revenue
FROM sales_data
WHERE date >= '2026-01-01'
GROUP BY product
ORDER BY total_revenue DESC;
```

## 아키텍처 설계 패턴과 모범 사례

### 패턴 1: JSONB + Generated Columns 하이브리드

자주 쿼리되는 필드는 generated column으로 추출하여 인덱싱하고, 나머지는 JSONB로 유연하게 관리:

```sql
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,
    profile_data JSONB NOT NULL,
    
    -- Generated columns (자주 쿼리되는 필드)
    email VARCHAR(255) GENERATED ALWAYS AS (profile_data->>'email') STORED,
    country VARCHAR(2) GENERATED ALWAYS AS (profile_data->>'country') STORED,
    created_at TIMESTAMP GENERATED ALWAYS AS (
        (profile_data->>'created_at')::TIMESTAMP
    ) STORED,
    
    -- 인덱스
    CONSTRAINT valid_email CHECK (email ~ '^[^@]+@[^@]+\.[^@]+$')
);

CREATE INDEX idx_users_email ON user_profiles(email);
CREATE INDEX idx_users_country_created ON user_profiles(country, created_at DESC);

-- 쿼리는 간단하면서도 빠름
SELECT user_id, email, created_at
FROM user_profiles
WHERE country = 'US'
  AND created_at >= NOW() - INTERVAL '30 days';
```

### 패턴 2: RLS + Application Context

애플리케이션별 컨텍스트를 설정하고 RLS로 자동 필터링:

```sql
-- 미들웨어에서 설정
async function setUserContext(req, res, next) {
    const { userId, tenantId, roles } = req.user;
    
    await req.dbClient.query(`
        SELECT set_config('app.user_id', $1, true),
               set_config('app.tenant_id', $2, true),
               set_config('app.roles', $3, true)
    `, [userId, tenantId, JSON.stringify(roles)]);
    
    next();
}

-- RLS 정책이 자동으로 컨텍스트 사용
-- 개발자는 WHERE 절 작성 불필요!
```

### 패턴 3: LISTEN/NOTIFY + Background Jobs

데이터베이스 트리거로 백그라운드 작업을 비동기 실행:

```sql
-- 파일 업로드 후 처리 트리거
CREATE OR REPLACE FUNCTION enqueue_image_processing()
RETURNS TRIGGER AS $$
BEGIN
    PERFORM pg_notify(
        'image_processing_queue',
        json_build_object(
            'image_id', NEW.image_id,
            'user_id', NEW.user_id,
            'file_path', NEW.file_path,
            'operations', ARRAY['thumbnail', 'watermark', 'optimize']
        )::text
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER image_upload_trigger
    AFTER INSERT ON uploaded_images
    FOR EACH ROW
    EXECUTE FUNCTION enqueue_image_processing();
```

워커 프로세스가 LISTEN하고 작업 처리:

```javascript
pgClient.on('notification', async (msg) => {
    const job = JSON.parse(msg.payload);
    await processImage(job);
});
```

## 결론: PostgreSQL First 아키텍처

PostgreSQL을 단순한 데이터 저장소로만 보는 시대는 끝났다. JSONB로 NoSQL의 유연성을, RLS로 데이터베이스 레벨 보안을, LISTEN/NOTIFY로 실시간 통신을, Full-Text Search로 검색 기능을 구현할 수 있다. 새로운 프로젝트를 시작할 때 "PostgreSQL로 가능한가?"를 먼저 물어보라. 대부분의 경우 답은 "예"일 것이다.

복잡한 기술 스택보다 단순함이 승리한다. PostgreSQL이 제공하는 강력한 기능들을 활용하여 아키텍처를 단순하게 유지하면서도 엔터프라이즈급 요구사항을 충족할 수 있다. 이것이 PostgreSQL이 단순한 데이터베이스를 넘어 백엔드 아키텍처의 혁명이 되는 이유다.

---

**작성 일자: 2026-01-29**
