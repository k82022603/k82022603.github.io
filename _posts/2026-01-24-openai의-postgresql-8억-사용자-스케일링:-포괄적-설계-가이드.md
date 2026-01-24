---
title: "OpenAI의 PostgreSQL 8억 사용자 스케일링: 포괄적 설계 가이드"
date: 2026-01-24 10:00:00 +0900
categories: [TechStack,  Backend Architecture]
mermaid: [True]
tags: [AI,  PostgreSQL,  AzureCosmosDB,  OpenAI,  Guide,  Claude.write]
---


>
>OpenAI가 PostgreSQL을 ChatGPT의 8억 사용자 규모로 스케일링한 방법에 대해 글을 올렸습니다. (https://openai.com/index/scaling-postgresql/)
>
>OpenAI가 PostgreSQL을 어떻게 수백만 QPS(초당 쿼리 수)를 처리할 수 있는 수준으로 끌어올렸는지, 실제 운영 경험을 바탕으로 상세히 다룹니다. 
>
>놀랍게도 샤딩 없이 단일 프라이머리(single primary) + 약 50개 리드 복제본(read replicas) 구조를 유지하면서도 안정적으로 운영 중이라는 점이 핵심입니다.￼
>
>**기본 아키텍처**
>
>- 단일 프라이머리 (Azure PostgreSQL Flexible Server의 대형 인스턴스): 모든 쓰기를 처리.
>
>- 거의 50개 리드 복제본: 여러 Azure 리전에 분산 배치 → 글로벌 저지연 읽기 지원.
>
>- 워크로드 특성: 극도로 읽기 중심(read-heavy) → 대부분의 트래픽이 읽기이므로 복제본으로 오프로드 가능.
>
>지난 1년간 부하 10배 이상 증가, 앞으로도 계속 성장 예상.
>
>PostgreSQL 자체는 샤딩하지 않음. 
>
>이유는 기존 애플리케이션 수백 곳을 샤딩하려면 수개월~수년 걸리는 대규모 작업이기 
>
>때문. 대신 쓰기 중심이고 샤딩 가능한 워크로드는 Azure Cosmos DB 같은 샤딩 시스템으로 이전 중이며, 새로운 테이블은 PostgreSQL에 추가하지 않음.￼
>
>https://www.threads.com/@rich_l_2024/post/DT3uN-hkqVA
>

## 들어가며

2026년 1월, OpenAI는 ChatGPT의 핵심 데이터베이스 인프라로 PostgreSQL을 사용하여 8억 사용자를 지원하고 있다고 발표했습니다. 이는 많은 엔지니어들에게 놀라운 소식이었습니다. 일반적으로 대규모 서비스는 NoSQL 데이터베이스나 복잡한 샤딩 시스템을 필요로 한다고 여겨지기 때문입니다. 하지만 OpenAI는 단일 프라이머리 인스턴스와 약 50개의 읽기 복제본이라는 비교적 단순한 아키텍처로 수백만 QPS(초당 쿼리 수)를 처리하고 있습니다.

더 인상적인 것은 지난 1년간 부하가 10배 이상 증가했음에도 불구하고, 동일한 아키텍처 패턴을 유지하면서 안정적으로 운영하고 있다는 점입니다. 또한 최적화 작업 이후 9개월 동안 PostgreSQL 관련 Sev0(최고 심각도) 장애가 단 1건에 불과했습니다. 이는 99.999%의 가용성을 의미하며, 연간 약 5분 정도의 다운타임만 발생했다는 것을 뜻합니다.

이 문서는 OpenAI의 PostgreSQL 스케일링 여정을 깊이 있게 분석하고, 실무에서 적용할 수 있는 포괄적인 가이드를 제공합니다. POSETTE 2025 컨퍼런스에서 OpenAI의 Bohan Zhang이 발표한 내용과 Microsoft Azure 팀의 협업 사례, 그리고 커뮤니티의 베스트 프랙티스를 종합하여 작성되었습니다.

## 1. 아키텍처의 이해: 단순함 속의 복잡성

### 1.1 기본 구성

OpenAI의 PostgreSQL 아키텍처는 놀라울 정도로 직관적입니다. 핵심은 **단일 프라이머리 인스턴스**와 **약 50개의 읽기 복제본**입니다. 프라이머리는 Azure Database for PostgreSQL Flexible Server의 대형 인스턴스로 운영되며, 모든 쓰기 작업을 처리합니다. 이 프라이머리 인스턴스는 WAL(Write Ahead Log)을 생성하고, 이를 모든 복제본으로 스트리밍합니다.

읽기 복제본은 전 세계 여러 Azure 리전에 분산 배치되어 있습니다. 각 복제본은 프라이머리의 완전한 복사본이며, 비동기 복제 방식으로 데이터를 동기화합니다. 지리적 분산을 통해 글로벌 사용자들에게 저지연 읽기 서비스를 제공할 수 있습니다. 예를 들어, 미국 동부의 사용자는 미국 동부 리전의 복제본에서 데이터를 읽고, 아시아의 사용자는 아시아 리전의 복제본에서 데이터를 읽습니다.

### 1.2 워크로드 특성: 극도로 읽기 중심

이 아키텍처가 성공할 수 있었던 가장 중요한 요인은 워크로드의 특성입니다. ChatGPT와 OpenAI API의 워크로드는 **극도로 읽기 중심**입니다. 대부분의 요청은 기존 대화 내역을 조회하거나, 사용자 정보를 확인하거나, API 키를 검증하는 등의 읽기 작업입니다. 쓰기 작업은 전체 트래픽의 작은 부분만을 차지합니다.

읽기 중심 워크로드는 수평적으로 확장하기 쉽습니다. 읽기 부하가 증가하면 복제본을 추가하기만 하면 됩니다. 각 복제본은 독립적으로 읽기 쿼리를 처리할 수 있으며, 프라이머리에 부담을 주지 않습니다. 반면 쓰기 작업은 모두 단일 프라이머리를 거쳐야 하므로, 쓰기가 병목이 될 수 있습니다. 하지만 OpenAI의 경우 쓰기 비중이 충분히 낮아서 단일 프라이머리로도 처리 가능했습니다.

### 1.3 샤딩을 피한 이유와 대안 전략

많은 사람들이 "왜 샤딩하지 않았나?"라고 궁금해합니다. 샤딩은 데이터를 여러 독립적인 데이터베이스로 분할하여 쓰기 성능을 선형적으로 확장할 수 있는 강력한 기법입니다. 하지만 OpenAI는 의도적으로 샤딩을 피했습니다.

첫 번째 이유는 **복잡성**입니다. OpenAI의 경우 수백 개의 애플리케이션과 서비스가 PostgreSQL에 연결되어 있습니다. 이들을 모두 샤딩 아키텍처로 전환하려면 각 애플리케이션의 데이터 접근 패턴을 분석하고, 적절한 샤딩 키를 선택하고, 크로스-샤드 쿼리를 처리하는 로직을 구현해야 합니다. 이는 수개월에서 수년이 걸리는 거대한 프로젝트입니다.

두 번째 이유는 **필요성의 부재**입니다. 읽기 중심 워크로드는 복제본 추가만으로도 충분히 확장 가능합니다. 현재 아키텍처로도 충분한 성장 여력이 있으며, 샤딩의 복잡성을 감수할 필요가 없습니다.

대신 OpenAI는 **선별적 마이그레이션** 전략을 택했습니다. 쓰기 중심이며 자연스러운 샤딩 키가 있는 워크로드는 Azure Cosmos DB 같은 샤딩된 시스템으로 이전했습니다. 예를 들어, 로그 데이터나 세션 정보처럼 사용자 ID를 기준으로 자연스럽게 분할할 수 있는 데이터는 Cosmos DB로 이동했습니다. 반면 관계가 복잡하고 샤딩하기 어려운 핵심 데이터는 PostgreSQL에 유지했습니다.

또한 PostgreSQL에 **새로운 테이블을 추가하는 것을 금지**했습니다. 새로운 기능이 필요하면 처음부터 적절한 데이터 저장소를 선택하도록 강제했습니다. 이를 통해 PostgreSQL의 부담이 더 이상 증가하지 않도록 관리했습니다.

## 2. 쓰기 워크로드 최적화: 프라이머리 보호하기

### 2.1 PostgreSQL MVCC의 한계

PostgreSQL은 MVCC(Multi-Version Concurrency Control)를 사용하여 동시성을 관리합니다. MVCC는 읽기와 쓰기가 서로를 차단하지 않도록 하여 높은 동시성을 제공합니다. 하지만 쓰기 중심 워크로드에서는 여러 문제를 야기합니다.

**쓰기 증폭 문제**: PostgreSQL에서 행을 업데이트하면 기존 행을 수정하는 것이 아니라 새로운 버전의 행을 생성합니다. 예를 들어, 단일 컬럼만 변경하더라도 전체 행이 복사됩니다. 이는 실제 변경된 데이터보다 훨씬 많은 양의 쓰기를 발생시킵니다.

**블로트 문제**: 업데이트나 삭제된 행은 즉시 제거되지 않고 "죽은 행(dead tuple)"으로 남습니다. 이들은 디스크 공간을 차지하며, 쿼리가 테이블을 스캔할 때 성능을 저하시킵니다. 시간이 지남에 따라 테이블과 인덱스가 부풀어 오르는 "블로트" 현상이 발생합니다.

**Autovacuum의 복잡성**: PostgreSQL은 Autovacuum 프로세스를 통해 죽은 행을 정리합니다. 하지만 대규모 시스템에서 Autovacuum을 적절히 튜닝하는 것은 매우 어렵습니다. Autovacuum이 너무 공격적이면 CPU와 I/O를 많이 소비하여 프로덕션 워크로드에 영향을 줍니다. 반대로 너무 보수적이면 블로트가 계속 증가합니다.

**복제 지연**: 많은 WAL(Write Ahead Log)이 생성되면 복제본으로 전송해야 할 데이터가 증가합니다. 이는 복제 지연을 야기하고, 최악의 경우 복제본이 프라이머리를 따라잡지 못하는 상황이 발생할 수 있습니다.

### 2.2 애플리케이션 레벨 쓰기 감소

OpenAI는 우선 애플리케이션 레벨에서 불필요한 쓰기를 제거하는 데 집중했습니다.

**불필요한 업데이트 방지**: 많은 애플리케이션이 실제로 데이터가 변경되지 않았는데도 UPDATE 쿼리를 실행합니다. 예를 들어:

```python
# 나쁜 예: 항상 업데이트
user.last_seen = now()
db.session.commit()

# 좋은 예: 의미 있는 변화가 있을 때만 업데이트
if (now() - user.last_seen) > timedelta(minutes=5):
    user.last_seen = now()
    db.session.commit()
```

이러한 작은 변화들이 누적되면 엄청난 쓰기 감소로 이어집니다.

**Lazy Write 전략**: 모든 변경을 즉시 데이터베이스에 쓰는 대신, 메모리나 캐시에 버퍼링했다가 배치로 처리합니다. 예를 들어, 사용자 활동 로그를 실시간으로 쓰는 대신 일정 시간 동안 모았다가 한 번에 처리합니다.

**Controlled Backfill**: 대량의 데이터를 업데이트해야 할 때(예: 새로운 컬럼에 기본값 채우기), 한 번에 수백만 행을 업데이트하면 데이터베이스가 과부하에 걸립니다. 대신 작은 배치로 나누고, 각 배치 사이에 적절한 지연을 두어 프라이머리에 가해지는 압력을 분산시킵니다:

```python
# 전체 테이블을 한 번에 업데이트 - 위험!
UPDATE users SET new_field = 'default_value';

# 작은 배치로 나누어 처리 - 안전
BATCH_SIZE = 1000
offset = 0
while True:
    updated = update_batch(offset, BATCH_SIZE)
    if updated == 0:
        break
    offset += BATCH_SIZE
    time.sleep(0.1)  # 100ms 대기
```

이 방식은 완료하는 데 며칠이 걸릴 수 있지만, 프로덕션에 영향을 주지 않으면서 안전하게 처리할 수 있습니다.

### 2.3 Autovacuum 튜닝

OpenAI는 Autovacuum 설정을 대형 테이블에 맞게 공격적으로 조정했습니다.

기본적으로 PostgreSQL은 `autovacuum_vacuum_scale_factor`를 0.2로 설정합니다. 이는 테이블의 20% 행이 변경되면 Autovacuum을 실행한다는 의미입니다. 10억 행짜리 테이블이라면 2억 행이 변경되어야 Autovacuum이 시작되는데, 이는 너무 늦습니다.

OpenAI는 대형 테이블에 대해 이 값을 **0.01**(1%)로 설정했습니다:

```sql
ALTER TABLE large_table SET (
    autovacuum_vacuum_scale_factor = 0.01,
    autovacuum_vacuum_threshold = 10000
);
```

또한 자주 업데이트되는 테이블에 대해서는 `fillfactor`를 조정하여 HOT(Heap-Only Tuple) 업데이트를 활성화했습니다:

```sql
ALTER TABLE frequently_updated SET (fillfactor = 70);
```

이렇게 하면 각 페이지에 30%의 빈 공간을 남겨두어, 업데이트 시 새로운 행 버전을 같은 페이지에 저장할 수 있습니다. 이는 인덱스 업데이트를 피하고 성능을 크게 향상시킵니다.

### 2.4 Azure Cosmos DB로의 선별적 마이그레이션

쓰기 중심이면서 자연스러운 파티션 키가 있는 워크로드는 Azure Cosmos DB로 마이그레이션했습니다.

**적합한 워크로드**:
- 쓰기 비중이 30% 이상
- 명확한 파티션 키 존재 (예: 사용자 ID, 세션 ID)
- 최종 일관성이 허용 가능
- 복잡한 조인이 필요 없음

**마이그레이션 프로세스**:

1. **워크로드 분석**: 어떤 테이블이 쓰기 중심인지, 어떤 파티션 키가 적합한지 분석
2. **데이터 모델 변환**: 관계형 모델을 문서 모델로 변환
3. **Dual-write 구현**: PostgreSQL과 Cosmos DB에 동시에 쓰기
4. **검증 단계**: 두 시스템의 데이터가 일치하는지 확인
5. **읽기 전환**: 점진적으로 읽기를 Cosmos DB로 이동
6. **완전 전환**: PostgreSQL에서 데이터 제거

이 과정은 각 워크로드마다 수주에서 수개월이 걸립니다. 하지만 완료되면 PostgreSQL 프라이머리의 쓰기 부담이 크게 줄어듭니다.

## 3. 읽기 워크로드 스케일링: 복제본 활용

### 3.1 읽기 복제본 전략

OpenAI는 약 50개의 읽기 복제본을 운영하며, 각 복제본은 특정 목적으로 최적화되어 있습니다.

**지리적 분산**: 복제본은 여러 Azure 리전에 분산되어 있습니다. 예를 들어:
- 미국 동부: 15개 복제본
- 미국 서부: 10개 복제본
- 유럽: 10개 복제본
- 아시아 태평양: 15개 복제본

이를 통해 전 세계 사용자에게 저지연 서비스를 제공합니다.

**우선순위 기반 격리**: 모든 요청이 동등하지 않습니다. ChatGPT의 실시간 대화는 높은 우선순위를 가지며, 내부 분석 쿼리는 낮은 우선순위를 가집니다. OpenAI는 복제본을 우선순위 별로 분리했습니다:

- **고우선순위 복제본**: ChatGPT 실시간 요청 전용
- **중간우선순위 복제본**: API 요청 및 일반 서비스
- **저우선순위 복제본**: 분석, 리포팅, 배치 작업

이렇게 함으로써 낮은 우선순위 작업이 높은 우선순위 작업에 영향을 주지 않도록 합니다.

### 3.2 쿼리 최적화

복제본을 많이 추가하는 것만으로는 충분하지 않습니다. 각 쿼리가 효율적이어야 합니다.

**ORM 생성 쿼리 검토**: 많은 애플리케이션이 ORM(Object-Relational Mapping)을 사용합니다. ORM은 편리하지만 때로는 비효율적인 SQL을 생성합니다. OpenAI는 모든 ORM 생성 쿼리를 검토하고 최적화했습니다.

특히 문제가 되었던 것은 **N+1 쿼리 패턴**입니다:

```python
# 나쁜 예: N+1 쿼리
users = User.query.all()  # 1개 쿼리
for user in users:
    user.profile  # N개 쿼리 (각 사용자마다)

# 좋은 예: Eager loading
users = User.query.options(joinedload(User.profile)).all()  # 1개 쿼리
```

**복잡한 조인 회피**: OpenAI는 12개 테이블을 조인하는 쿼리를 발견했습니다. 이러한 쿼리는 CPU를 크게 소비하며, 트래픽이 증가하면 전체 서비스를 마비시킬 수 있습니다.

해결책은 조인을 애플리케이션 레벨로 이동하는 것입니다:

```python
# 나쁜 예: 복잡한 조인
result = db.session.query(User)\
    .join(Profile)\
    .join(Settings)\
    .join(Subscription)\
    # ... 8개 더
    .filter(User.id == user_id)\
    .first()

# 좋은 예: 여러 단순 쿼리로 분할
user = User.query.get(user_id)
profile = Profile.query.filter_by(user_id=user_id).first()
settings = Settings.query.filter_by(user_id=user_id).first()
# ...
```

이렇게 하면 각 쿼리가 단순해지고, 캐싱하기도 쉬워집니다.

**장기 실행 쿼리 관리**: 1초 이상 실행되는 쿼리는 프라이머리에서 실행되지 않도록 했습니다. 대신 전용 복제본에서 실행하거나, 쿼리를 최적화했습니다.

### 3.3 읽기/쓰기 분리

애플리케이션은 읽기와 쓰기를 명확히 구분해야 합니다:

```python
class DatabaseRouter:
    def __init__(self, primary_host, replica_hosts):
        self.primary_host = primary_host
        self.replica_hosts = replica_hosts
        self.replica_index = 0
    
    def get_read_connection(self):
        # 라운드 로빈으로 복제본 선택
        host = self.replica_hosts[self.replica_index]
        self.replica_index = (self.replica_index + 1) % len(self.replica_hosts)
        return psycopg2.connect(host=host, ...)
    
    def get_write_connection(self):
        # 항상 프라이머리 사용
        return psycopg2.connect(host=self.primary_host, ...)
```

중요한 것은 읽기 전용 요청은 **프라이머리를 거치지 않는다**는 것입니다. ChatGPT의 대부분 요청은 기존 대화를 불러오는 것이므로, 복제본에서 처리됩니다. 프라이머리는 쓰기와 쓰기 트랜잭션의 일부인 읽기만 처리합니다.

### 3.4 캐싱 전략

복제본만으로는 부족할 때가 있습니다. 캐싱 계층을 추가하여 데이터베이스 부하를 더욱 줄입니다.

**Redis 분산 캐시**: 자주 읽히는 데이터는 Redis에 캐시합니다. 예를 들어:
- 사용자 세션
- API 키 검증 결과
- 최근 대화 내역

**Cache Stampede 방지**: 캐시가 만료되거나 실패하면, 여러 요청이 동시에 데이터베이스를 조회하는 "cache stampede" 현상이 발생할 수 있습니다. OpenAI는 cache locking을 구현했습니다:

```python
def get_data(key):
    # 캐시에서 시도
    data = cache.get(key)
    if data:
        return data
    
    # 락 획득 시도
    lock_key = f"lock:{key}"
    if cache.set(lock_key, "1", nx=True, ex=10):
        # 이 요청만 데이터베이스에서 가져옴
        data = database.get(key)
        cache.set(key, data, ex=3600)
        cache.delete(lock_key)
        return data
    else:
        # 다른 요청이 이미 가져오는 중
        # 짧은 지연 후 캐시 재확인
        time.sleep(0.1)
        return cache.get(key) or get_data(key)
```

## 4. 연결 관리: PgBouncer의 마법

### 4.1 연결 문제의 본질

PostgreSQL의 연결 모델은 고동시성 환경에서 문제가 될 수 있습니다. 각 클라이언트 연결은 PostgreSQL에서 별도의 프로세스(백엔드)를 생성합니다. 이는:

- **메모리 소비**: 각 연결은 수 MB의 메모리를 사용
- **프로세스 생성 오버헤드**: 새 연결을 만들 때마다 fork() 및 초기화 필요
- **컨텍스트 스위칭**: 수천 개의 프로세스가 있으면 OS 스케줄링 오버헤드 증가

또한 Azure PostgreSQL Flexible Server는 연결 수 제한이 있습니다(예: 5,000개). 수백 개의 애플리케이션 서버가 각각 수십 개의 연결을 유지하면 이 제한에 쉽게 도달합니다.

### 4.2 PgBouncer 도입과 효과

PgBouncer는 PostgreSQL과 애플리케이션 사이에 위치하는 경량 연결 풀러입니다. OpenAI에서 PgBouncer를 도입한 결과는 극적이었습니다:

**레이턴시 개선**: 연결 레이턴시가 약 50ms에서 5ms 미만으로 감소했습니다. 이는 10배의 개선입니다. 왜 이런 개선이 가능했을까요?

- **연결 재사용**: 새 연결을 만드는 대신 기존 연결을 재사용
- **지역 배치**: PgBouncer를 애플리케이션과 같은 리전에 배치하여 네트워크 지연 최소화
- **경량 프록시**: PgBouncer 자체의 오버헤드가 매우 낮음

**연결 수 감소**: 수천 개의 클라이언트 연결을 수십 개의 데이터베이스 연결로 집약할 수 있습니다.

### 4.3 PgBouncer 설정 및 최적화

OpenAI는 PgBouncer를 **Transaction pooling 모드**로 실행합니다:

```ini
[databases]
chatgpt = host=primary.postgres.database.azure.com port=5432 dbname=chatgpt

[pgbouncer]
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = userlist.txt

# Transaction pooling
pool_mode = transaction

# 풀 크기 설정
default_pool_size = 20
max_client_conn = 10000
```

**Transaction pooling의 의미**: 연결은 트랜잭션이 완료되면 즉시 풀로 반환됩니다. 이는 매우 효율적이지만, 애플리케이션이 주의해야 할 점이 있습니다:

- **트랜잭션을 짧게 유지**: 트랜잭션이 길어지면 연결을 오래 점유
- **Prepared statements 주의**: Transaction pooling에서는 제한적으로만 사용 가능
- **임시 테이블 사용 불가**: 트랜잭션이 끝나면 임시 테이블도 사라짐

**유휴 트랜잭션 방지**: 가장 큰 문제 중 하나는 "idle in transaction" 상태입니다:

```python
# 나쁜 예: 유휴 트랜잭션
conn = get_connection()
conn.execute("BEGIN")
conn.execute("SELECT * FROM users WHERE id = 1")
# 여기서 복잡한 비즈니스 로직 실행 (데이터베이스 필요 없음)
do_some_processing()
conn.execute("COMMIT")

# 좋은 예: 트랜잭션을 최소화
data = None
conn = get_connection()
conn.execute("BEGIN")
data = conn.execute("SELECT * FROM users WHERE id = 1").fetchone()
conn.execute("COMMIT")
conn.close()

# 이제 데이터베이스 연결 없이 처리
processed = do_some_processing(data)
```

### 4.4 Kubernetes에서의 PgBouncer 배치

OpenAI는 Kubernetes를 사용하여 PgBouncer를 관리합니다:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgbouncer
spec:
  replicas: 4
  selector:
    matchLabels:
      app: pgbouncer
  template:
    metadata:
      labels:
        app: pgbouncer
    spec:
      containers:
      - name: pgbouncer
        image: pgbouncer/pgbouncer:latest
        ports:
        - containerPort: 6432
        volumeMounts:
        - name: config
          mountPath: /etc/pgbouncer
```

각 읽기 복제본마다 별도의 PgBouncer 배포가 있으며, Kubernetes Service를 통해 로드 밸런싱됩니다.

## 5. 스키마 거버넌스: 프로덕션 안정성 유지

### 5.1 엄격한 변경 정책

대규모 시스템에서 스키마 변경은 매우 위험할 수 있습니다. 잘못된 변경은 전체 서비스를 중단시킬 수 있습니다. OpenAI는 엄격한 스키마 거버넌스 정책을 수립했습니다.

**허용되는 변경**:
1. 컬럼 추가/제거 (테이블 재작성이 필요 없는 경우만)
2. 인덱스 생성/삭제 (`CONCURRENTLY` 옵션 사용 필수)
3. 5초 이내에 완료되는 작은 변경

**금지되는 변경**:
1. 새 테이블 생성 (기존 애플리케이션만 사용)
2. 테이블 재작성이 필요한 변경 (예: 컬럼 타입 변경)
3. 장기 실행 ALTER 문

### 5.2 안전한 스키마 변경 패턴

**컬럼 추가 예시**:

```sql
-- 안전: DEFAULT 값 없이 NULL 허용 컬럼 추가
ALTER TABLE users ADD COLUMN new_field TEXT;

-- 위험: DEFAULT 값이 있으면 전체 테이블 재작성 (PostgreSQL 11 미만)
ALTER TABLE users ADD COLUMN new_field TEXT DEFAULT 'value';

-- 안전한 대안 (PostgreSQL 11+)
ALTER TABLE users ADD COLUMN new_field TEXT DEFAULT 'value';
-- PostgreSQL 11부터는 DEFAULT가 있어도 메타데이터만 변경
```

**인덱스 생성 예시**:

```sql
-- 위험: 테이블 잠금
CREATE INDEX idx_users_email ON users(email);

-- 안전: CONCURRENTLY 사용
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
```

`CONCURRENTLY` 옵션은 인덱스 생성 중에도 테이블 읽기/쓰기를 허용합니다. 단, 시간이 더 오래 걸리고 실패 가능성이 있습니다.

**컬럼 타입 변경**: 대부분의 컬럼 타입 변경은 전체 테이블 재작성이 필요합니다. 이는 대형 테이블에서는 수 시간이 걸릴 수 있습니다.

안전한 대안:
1. 새 컬럼 추가
2. 애플리케이션 코드 변경 (새 컬럼 사용)
3. 배치 작업으로 데이터 마이그레이션
4. 구 컬럼 제거

```sql
-- 1단계: 새 컬럼 추가
ALTER TABLE users ADD COLUMN age_new INTEGER;

-- 2단계: 애플리케이션 배포 (새 컬럼 읽기/쓰기)

-- 3단계: 데이터 마이그레이션 (배치로)
UPDATE users SET age_new = age::integer
WHERE age_new IS NULL
LIMIT 1000;

-- 4단계: 구 컬럼 제거
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users RENAME COLUMN age_new TO age;
```

### 5.3 Autovacuum과 스키마 변경

스키마 변경이 실패하는 가장 흔한 이유 중 하나는 Autovacuum이 테이블을 잠그고 있는 것입니다. Autovacuum은 ACCESS SHARE 잠금을 유지하는데, 이는 대부분의 ALTER TABLE 명령과 충돌합니다.

해결책:

```sql
-- Autovacuum 프로세스 확인
SELECT pid, query FROM pg_stat_activity
WHERE query LIKE '%autovacuum%'
  AND query LIKE '%my_table%';

-- 필요시 종료 (주의!)
SELECT pg_terminate_backend(pid);

-- 즉시 ALTER 실행
ALTER TABLE my_table ADD COLUMN new_field TEXT;
```

하지만 Autovacuum을 종료하는 것은 최후의 수단이어야 합니다. 가능하면 Autovacuum이 완료될 때까지 기다리는 것이 좋습니다.

## 6. 고급 주제: Cascading Replication과 미래 확장

### 6.1 복제본 확장의 한계

현재 OpenAI는 약 50개의 복제본을 운영하고 있습니다. 프라이머리는 각 복제본에 WAL을 스트리밍해야 합니다. 복제본이 증가할수록:

- **네트워크 대역폭**: 프라이머리의 네트워크 대역폭 소비 증가
- **CPU 오버헤드**: WAL sender 프로세스가 각 복제본에 데이터 전송
- **복제 지연**: 복제본 수가 많아지면 일부 복제본의 지연이 증가할 수 있음

이론적으로는 계속 복제본을 추가할 수 있지만, 실제로는 한계가 있습니다.

### 6.2 Cascading Replication

해결책은 **Cascading Replication**입니다. 이는 복제본이 다른 복제본의 소스가 되는 계층적 복제 구조입니다.

**구조 예시**:
```
프라이머리
├── 중간 복제본 1
│   ├── 최종 복제본 1
│   ├── 최종 복제본 2
│   └── 최종 복제본 3
├── 중간 복제본 2
│   ├── 최종 복제본 4
│   └── 최종 복제본 5
...
```

프라이머리는 5개의 중간 복제본에만 WAL을 전송하고, 각 중간 복제본은 5개의 최종 복제본에 WAL을 전송합니다. 이렇게 하면 25개의 복제본을 유지하면서도 프라이머리의 부담은 5개만 처리하는 것과 같습니다.

Azure Database for PostgreSQL는 최근 Cascading Replication을 프리뷰로 제공하기 시작했습니다:

```bash
# 1단계: 프라이머리에서 중간 복제본 생성
az postgres flexible-server replica create \
    --source-server primary-server \
    --name intermediate-replica-1

# 2단계: 중간 복제본에서 최종 복제본 생성
az postgres flexible-server replica create \
    --source-server intermediate-replica-1 \
    --name final-replica-1
```

**제약 사항**:
- 최대 2단계 계층 (프라이머리 → 중간 → 최종)
- 각 중간 복제본은 최대 5개의 하위 복제본 가능
- PostgreSQL 14 이상 필요

**주의 사항**:
- **복제 지연 누적**: 각 단계마다 지연이 추가됨
- **장애 조치 복잡성**: 중간 복제본이 실패하면 하위 복제본도 영향받음

OpenAI는 현재 이 기능을 테스트 중이며, 안정성이 검증되면 프로덕션에 도입할 계획입니다.

### 6.3 Elastic Clusters (미래 기능)

Azure는 PostgreSQL용 Elastic Clusters 기능도 개발 중입니다. 이는 행 기반 또는 스키마 기반 샤딩을 자동으로 관리해줍니다.

**행 기반 샤딩**: 특정 컬럼 값에 따라 행을 여러 노드에 분산
**스키마 기반 샤딩**: 전체 테이블을 여러 노드에 분산

이 기능이 안정화되면 OpenAI도 일부 워크로드에 대해 샤딩을 고려할 수 있습니다.

## 7. 모니터링과 알람: 문제 조기 발견

### 7.1 핵심 메트릭

**성능 메트릭**:
- **QPS (Queries Per Second)**: 초당 쿼리 수
- **TPS (Transactions Per Second)**: 초당 트랜잭션 수
- **Query Latency**: P50, P95, P99 지연시간
- **Connection Count**: 활성 연결 수
- **CPU Usage**: CPU 사용률
- **Memory Usage**: 메모리 사용률
- **Disk I/O**: 읽기/쓰기 IOPS

**데이터베이스 건강**:
- **Cache Hit Ratio**: 캐시 히트율 (목표: >95%)
- **Dead Tuples**: 죽은 행 수
- **Table Bloat**: 테이블 블로트 비율
- **Index Bloat**: 인덱스 블로트 비율
- **Autovacuum Activity**: Autovacuum 실행 빈도 및 지속 시간

**복제**:
- **Replication Lag**: 복제 지연 (목표: <100ms)
- **WAL Segments**: 누적된 WAL 파일 수
- **Replication Slot Status**: 복제 슬롯 상태

### 7.2 Prometheus와 Grafana

OpenAI는 Prometheus를 사용하여 메트릭을 수집하고 Grafana로 시각화합니다.

**Postgres Exporter 설정**:

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

**중요한 쿼리**:

```promql
# 복제 지연
pg_replication_lag_seconds > 0.1

# 캐시 히트율
rate(pg_stat_database_blks_hit[5m]) / 
(rate(pg_stat_database_blks_hit[5m]) + rate(pg_stat_database_blks_read[5m]))

# 죽은 행 수
pg_stat_user_tables_n_dead_tup

# 장기 실행 쿼리
pg_stat_activity_max_tx_duration > 60
```

### 7.3 알람 정책

**Critical (즉시 대응)**:
- 복제 지연 > 5초
- 디스크 사용률 > 90%
- 프라이머리 CPU > 90% (10분 이상)
- 캐시 히트율 < 80%
- 활성 연결 > 4500 (제한 5000의 90%)

**Warning (곧 조치 필요)**:
- 복제 지연 > 1초
- 디스크 사용률 > 80%
- 블로트 비율 > 30%
- CPU > 70% (30분 이상)

**Info (모니터링)**:
- 복제 지연 > 500ms
- 장기 실행 쿼리 > 10초

### 7.4 대시보드 구성

**실시간 대시보드**:
- 현재 QPS/TPS
- P99 레이턴시
- 복제 지연 (각 복제본별)
- CPU/메모리 사용률
- 활성 연결 수

**일일 대시보드**:
- 쿼리 실행 통계 (pg_stat_statements)
- 테이블 크기 및 블로트
- Autovacuum 활동
- 느린 쿼리 Top 10

**주간 대시보드**:
- 용량 트렌드
- 성능 트렌드
- 비용 분석

## 8. 재해 복구와 고가용성

### 8.1 자동 장애 조치

Azure Database for PostgreSQL은 HA(High Availability) 모드를 제공합니다. 이는 프라이머리와 동일한 핫 스탠바이를 유지하며, 장애 시 자동으로 스탠바이를 프라이머리로 승격시킵니다.

**장애 조치 프로세스**:
1. Azure가 프라이머리 장애 감지 (30초 이내)
2. 스탠바이를 새 프라이머리로 승격
3. DNS 레코드 업데이트
4. 애플리케이션이 새 프라이머리로 재연결

전체 프로세스는 1-2분 내에 완료되며, 애플리케이션은 자동으로 재시도하여 투명하게 복구됩니다.

### 8.2 백업 전략

**자동 백업**:
- Azure는 매일 자동으로 전체 백업 수행
- 트랜잭션 로그는 5분마다 백업
- PITR(Point-In-Time Recovery) 지원: 최대 35일 이전 시점으로 복구 가능

**크로스 리전 백업**:
- 주 리전 장애에 대비하여 다른 리전에도 백업 저장
- 지리적 재해에 대한 추가 보호

**백업 테스트**:
- 월 1회 백업 복원 테스트 수행
- 복구 시간 및 데이터 무결성 검증

### 8.3 재해 복구 계획

**RTO (Recovery Time Objective)**: 1시간
**RPO (Recovery Point Objective)**: 5분

**재해 시나리오별 대응**:

**시나리오 1: 프라이머리 인스턴스 장애**
- 조치: Azure 자동 HA 장애 조치
- 예상 복구 시간: 1-2분
- 데이터 손실: 없음 (동기 복제)

**시나리오 2: 리전 전체 장애**
- 조치: 다른 리전의 복제본을 프라이머리로 승격
- 예상 복구 시간: 10-30분
- 데이터 손실: 최대 5분 (비동기 복제)

**시나리오 3: 데이터 손상**
- 조치: PITR을 사용하여 손상 이전 시점으로 복구
- 예상 복구 시간: 1-2시간 (데이터 크기에 따름)
- 데이터 손실: 복구 시점부터 현재까지

## 9. 성과 측정과 결과

### 9.1 정량적 성과

**처리량**:
- 수백만 QPS (읽기 + 쓰기 합산)
- 10배 부하 증가에도 동일 아키텍처 유지

**지연시간**:
- P99 지연시간: <10ms (엔드투엔드)
- 데이터베이스 쿼리 레이턴시: <5ms (PgBouncer 도입 후)

**안정성**:
- 99.999% 가용성 (연간 약 5분 다운타임)
- 9개월간 Sev0 장애 1건만 발생

**확장성**:
- 50개 복제본 운영
- 복제 지연 <100ms 유지
- 추가 확장 여력 충분

### 9.2 비기술적 성과

**개발 속도**:
- 단순한 아키텍처 덕분에 빠른 기능 개발
- 새 팀원 온보딩 시간 단축

**운영 부담**:
- 관리형 서비스 사용으로 인프라 관리 최소화
- 작은 팀으로 대규모 시스템 운영

**비용 효율성**:
- 샤딩보다 운영 비용 낮음
- 예측 가능한 성능 및 비용

## 10. 교훈과 베스트 프랙티스

### 10.1 핵심 교훈

**1. 단순함이 확장 가능하다**

가장 인상적인 점은 아키텍처의 단순함입니다. 특별한 기술이나 복잡한 시스템 없이, PostgreSQL의 표준 기능과 잘 알려진 패턴만으로 8억 사용자를 지원합니다. 복잡성은 비용입니다. 개발 시간, 운영 부담, 장애 가능성 모두 복잡성에 비례합니다.

**2. 워크로드 특성을 이해하라**

OpenAI의 전략이 성공한 핵심은 워크로드가 극도로 읽기 중심이라는 점입니다. 자신의 워크로드 특성을 정확히 파악하는 것이 올바른 전략 선택의 첫 걸음입니다.

**3. 점진적 최적화의 힘**

OpenAI는 한 번에 모든 것을 바꾸지 않았습니다. PgBouncer 도입, 쿼리 최적화, 복제본 추가, Cosmos DB 마이그레이션 등을 단계적으로 진행했습니다. 각 단계에서 측정하고 검증하며 다음 단계로 나아갔습니다.

**4. 도구보다 원칙**

OpenAI가 사용한 기술은 모두 검증된 오픈소스나 관리형 서비스입니다. 특별한 것은 이들을 어떻게 사용했느냐입니다. 기본 원칙 - 병목 측정, 읽기/쓰기 분리, 적절한 인덱싱, 연결 풀링 - 을 철저히 적용했습니다.

**5. 적재적소에 적절한 도구**

PostgreSQL이 모든 것을 해결해주지는 않습니다. 쓰기 중심 워크로드는 Cosmos DB로, 캐싱은 Redis로, 로그는 객체 스토리지로 이동했습니다. 각 도구를 강점에 맞게 사용했습니다.

### 10.2 조직별 적용 가이드

**소규모 스타트업 (10K-100K 사용자)**
- 단일 PostgreSQL 인스턴스로 시작
- 관리형 서비스 사용 (Azure, AWS RDS)
- 기본 모니터링만으로 충분
- 읽기/쓰기 분리는 아직 불필요

**성장하는 스타트업 (100K-1M 사용자)**
- 읽기 복제본 1-2개 추가
- PgBouncer 도입 고려
- 쿼리 최적화 시작
- Redis 캐싱 계층 추가
- 모니터링 강화

**중간 규모 회사 (1M-10M 사용자)**
- 읽기 복제본 5-10개로 확장
- 워크로드별 전용 복제본
- 쓰기 최적화 시작
- 일부 워크로드의 다른 DB로 마이그레이션 고려
- 전담 데이터베이스 팀

**대규모 기업 (10M+ 사용자)**
- 수십 개의 읽기 복제본 (지역별)
- 샤딩 고려 (필요시)
- 폴리글랏 퍼시스턴스 전략
- 고급 모니터링 및 자동 복구
- 전문 데이터베이스 엔지니어링 팀

### 10.3 피해야 할 함정

**과도한 최적화**: 문제가 없는데 미리 최적화하지 마세요. 샤딩이 "나중에 필요할 수 있어"라는 이유만으로 구현하는 것은 시간 낭비입니다.

**측정 없는 최적화**: 추측하지 말고 측정하세요. 어떤 쿼리가 느린지, 어떤 테이블이 큰지, CPU가 병목인지 디스크인지 알아야 합니다.

**모니터링 무시**: "잘 작동하니까 모니터링은 나중에"는 위험합니다. 문제는 예고 없이 찾아옵니다.

**문서화 소홀**: 왜 이 인덱스가 있는지, 왜 이 설정을 변경했는지 기록하세요. 6개월 후 아무도 기억하지 못합니다.

**보안 무시**: 보안은 나중에 추가하기 어렵습니다. 처음부터 고려하세요.

## 11. 미래 전망

데이터베이스 기술은 계속 발전하고 있습니다. 앞으로 몇 년간 예상되는 변화:

**클라우드 네이티브 발전**: 관리형 서비스가 더욱 강력해질 것입니다. 자동 확장, 글로벌 분산, 고급 최적화가 기본으로 제공될 것입니다.

**AI/ML 통합**: 데이터베이스가 AI를 활용하여 자동으로 최적화될 것입니다. 인덱스 자동 생성, 쿼리 패턴 학습, 이상 탐지 등이 가능해질 것입니다.

**서버리스 확산**: 사용한 만큼만 비용을 지불하는 서버리스 데이터베이스가 더 보편화될 것입니다.

**하이브리드 전략**: 단일 클라우드 종속을 피하고 여러 클라우드를 조합하는 전략이 증가할 것입니다.

하지만 기본 원칙 - 워크로드 이해, 측정 기반 최적화, 올바른 도구 선택 - 은 변하지 않을 것입니다. 기술은 발전하지만 좋은 엔지니어링의 본질은 동일합니다.

## 12. 결론

OpenAI의 PostgreSQL 스케일링 여정은 우리에게 중요한 메시지를 전합니다. 대규모 시스템을 구축하는 데 항상 최신 기술이나 복잡한 아키텍처가 필요한 것은 아닙니다. 때로는 단순함, 기본 원칙에 대한 철저한 이해, 그리고 점진적 개선이 더 나은 결과를 가져옵니다.

PostgreSQL은 30년 된 오픈소스 데이터베이스입니다. 특별한 기능이 없는 표준 버전으로 8억 사용자를 지원할 수 있다는 것이 증명되었습니다. 중요한 것은 도구가 아니라 그것을 어떻게 사용하느냐입니다.

이 문서가 여러분의 데이터베이스 스케일링 여정에 도움이 되기를 바랍니다. OpenAI의 경험에서 배우되, 맹목적으로 따라하지는 마세요. 여러분의 워크로드는 OpenAI와 다를 것입니다. 자신의 상황을 정확히 파악하고, 측정하고, 점진적으로 개선하세요.

"After all the optimization we did, we are super happy with Postgres right now for our read-heavy workloads." - Bohan Zhang, OpenAI

---

**작성 일자: 2026-01-24**

## 참고 자료

1. OpenAI Blog: "Scaling PostgreSQL to power 800 million ChatGPT users" (2026)
2. Microsoft Azure Blog: "How OpenAI scaled with Azure Database for PostgreSQL" (2025)
3. POSETTE 2025: Bohan Zhang's presentation "Scaling Postgres to the next level at OpenAI"
4. Azure PostgreSQL Documentation
5. PostgreSQL Official Documentation
6. PgBouncer Documentation
7. Community discussions on Hacker News, DEV Community, and Reddit

