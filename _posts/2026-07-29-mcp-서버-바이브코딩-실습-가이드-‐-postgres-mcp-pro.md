---
title: "MCP 서버 바이브코딩 실습 가이드 - Postgres MCP Pro"
date: 2026-07-29 22:00:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  vibe-coding,  MCP,  PostgreSQL,  Claude.write]
---

## — Postgres MCP Pro로 데이터베이스를 "대화"로 다루기 (Windows 실습판)

- 실습 환경: Windows 10/11 노트북 또는 PC · Docker Desktop · Claude Desktop
- 대상: IT 비전공자 및 데이터베이스 초보자 (SQL을 몰라도 따라갈 수 있습니다)
- 소요 시간: 약 90분 (설치 30분 · 데이터 준비 20분 · 연동 15분 · 실습 25분)
- 작성 기준일: 2026년 7월 29일 (아래 모든 명령과 설정은 이 시점에 공식 문서로 확인한 내용입니다)

---

## 이 가이드를 시작하기 전에 — 무엇을 하게 되는 건가요?

이 실습을 끝내면, 여러분은 Claude에게 이렇게 말할 수 있게 됩니다.

> "내 데이터베이스 상태를 점검해줘."
> "가장 느린 쿼리를 찾아서 왜 느린지 설명하고, 어떤 인덱스를 만들면 좋을지 알려줘."
> "극장별 매출 순위를 조회해서 보기 좋은 대시보드로 만들어줘."

그러면 Claude가 **실제로 여러분 PC에 설치된 데이터베이스에 접속해서** 직접 조회하고, 분석하고, 그 결과를 화면에 그려 줍니다. SQL을 한 줄도 직접 쓰지 않습니다.

이걸 가능하게 해 주는 것이 **MCP(Model Context Protocol)** 라는 표준 규약이고, 그중에서도 데이터베이스 전문 도구인 [**Postgres MCP Pro**](https://github.com/crystaldba/postgres-mcp)를 오늘 사용합니다.

한 가지 중요한 점을 미리 말씀드리면, 이 실습에서 여러분이 직접 코드를 작성하는 부분은 거의 없습니다. 대부분은 **정해진 명령어를 복사해서 붙여넣고**, **Claude에게 한국어로 부탁하는** 일입니다. 그래서 "바이브코딩(Vibe Coding)" 실습이라고 부릅니다. 문법을 외우는 대신 원하는 것을 정확하게 설명하는 연습입니다.

---

## 1. 전체 구조 이해하기 — 부품이 세 개입니다

무엇을 설치하는지 모른 채 명령어만 따라 치면 중간에 막혔을 때 손을 쓸 수 없습니다. 그래서 먼저 큰 그림을 잡고 갑니다. 오늘 우리가 다루는 부품은 딱 세 개입니다.

```mermaid
flowchart LR
    subgraph PC["내 Windows 노트북"]
        CD["Claude Desktop<br/>(내가 대화하는 창)"]
        MCP["Postgres MCP Pro<br/>(DB 전문 도구 모음)"]
        DB["PostgreSQL<br/>(실제 데이터가 있는 창고)"]
    end

    CD -->|"1. 한국어 질문"| MCP
    MCP -->|"2. SQL 실행"| DB
    DB -->|"3. 데이터 반환"| MCP
    MCP -->|"4. 분석 결과"| CD

    style CD fill:#E2F1F2,stroke:#0E7C86
    style MCP fill:#FDF3E3,stroke:#B9741B
    style DB fill:#E7F4EE,stroke:#1A7355
```

각 부품의 역할을 사람에 비유하면 이해가 쉽습니다.

**PostgreSQL**은 데이터가 실제로 보관된 **창고**입니다. 영화 정보, 상영 시간표, 관객 리뷰 같은 것들이 표(테이블) 형태로 정리되어 들어 있습니다. 오늘은 이 창고를 여러분 PC 안에 직접 하나 만듭니다.

**Postgres MCP Pro**는 그 창고를 잘 아는 **베테랑 창고 관리자**입니다. 단순히 "물건 가져와"만 하는 게 아니라, "이 통로가 자주 막히는데 여기에 지름길을 하나 내면 100배 빨라집니다" 같은 전문적인 진단을 해 줍니다. 이 부분이 일반적인 데이터베이스 연결 도구와 결정적으로 다른 점입니다.

**Claude Desktop**은 여러분이 창고 관리자에게 말을 거는 **통역사이자 비서**입니다. 여러분이 한국어로 말하면 그것을 관리자가 알아듣는 명령으로 바꿔 전달하고, 돌아온 결과를 다시 한국어로 정리해서 보여 줍니다.

### 1.1 왜 Docker를 사용하나요?

이 실습에서는 PostgreSQL과 Postgres MCP Pro를 모두 **Docker**라는 도구로 실행합니다. Docker를 처음 듣는 분을 위해 설명하면, Docker는 프로그램을 **미리 포장된 도시락처럼** 통째로 가져와 실행하는 도구입니다.

원래대로라면 PostgreSQL을 설치하려면 설치 프로그램을 받아 실행하고, 계정을 만들고, 설정 파일을 고치고, 확장 기능을 따로 컴파일해야 합니다. 초보자가 한두 시간은 헤매는 과정입니다. Docker를 쓰면 이 모든 것이 **명령어 한 줄**로 끝나고, 실습이 끝난 뒤 **깨끗하게 지우는 것도 한 줄**로 끝납니다. 내 PC의 다른 프로그램에 영향을 주지 않는다는 점도 큰 장점입니다.

Postgres MCP Pro 공식 문서 역시 Docker와 Python 두 가지 설치 방법 중 **Docker를 일반적으로 권장**하고 있습니다. Python으로 설치하면 환경에 따라 문제가 생기기 쉽다는 이유입니다. 이 가이드도 그 권장을 따릅니다.

### 1.2 Postgres MCP Pro가 제공하는 도구 9개

연동이 끝나면 Claude가 아래 9개의 도구를 사용할 수 있게 됩니다. 지금 다 외울 필요는 전혀 없고, "이런 게 있구나" 정도만 보고 넘어가세요. 나중에 실습하면서 자연스럽게 익숙해집니다.

| 도구 이름 | 하는 일 | 쉬운 설명 |
|---|---|---|
| `list_schemas` | 데이터베이스의 스키마 목록 조회 | 창고의 층 목록 보기 |
| `list_objects` | 특정 스키마의 테이블·뷰·시퀀스·확장 목록 조회 | 한 층에 있는 선반 목록 보기 |
| `get_object_details` | 특정 테이블의 컬럼·제약조건·인덱스 정보 조회 | 선반 하나를 자세히 들여다보기 |
| `execute_sql` | SQL 문장 실행 (제한 모드에서는 읽기 전용) | 실제로 물건을 꺼내오기 |
| `explain_query` | 쿼리 실행 계획 조회, 가상 인덱스 효과 시뮬레이션 | "이렇게 찾으면 몇 걸음 걸릴까?" 미리 계산 |
| `get_top_queries` | 총 실행 시간 기준으로 가장 느린 쿼리 보고 | 가장 오래 걸리는 작업 순위표 |
| `analyze_workload_indexes` | 전체 작업 부하를 분석해 최적 인덱스 추천 | 창고 전체를 보고 지름길 설계 |
| `analyze_query_indexes` | 지정한 쿼리들(최대 10개)에 대한 인덱스 추천 | 특정 작업만 골라서 지름길 설계 |
| `analyze_db_health` | 버퍼 캐시·연결·제약조건·인덱스·시퀀스·배큠 상태 종합 점검 | 창고 전체 건강검진 |

여기서 특별히 주목할 것은 `analyze_workload_indexes`입니다. 이 도구는 AI가 "감으로 추측"하는 게 아니라, Microsoft SQL Server의 **Anytime Algorithm**이라는 검증된 인덱스 튜닝 알고리즘을 구현해서 수천 가지 인덱스 조합을 실제로 탐색합니다. 그래서 결과가 재현 가능하고 신뢰할 수 있습니다. AI는 그 결과를 사람이 알아듣게 설명하는 역할을 맡습니다.

---

## 2. 사전 준비 — 설치할 것 두 가지

### 2.1 Docker Desktop 설치

Docker Desktop을 아직 설치하지 않았다면 `docker.com`에서 Windows용 설치 파일을 내려받아 설치합니다. 설치 중 **WSL 2** 관련 안내가 나오면 그대로 동의하고 진행하세요. WSL 2는 Windows에서 Linux 프로그램을 돌리기 위한 기반이며, Docker가 자동으로 설정해 줍니다.

설치 후 **Docker Desktop 프로그램을 실행**하고, 창 왼쪽 아래의 상태 표시가 초록색(Running)이 되기를 기다립니다. 이게 켜져 있지 않으면 이후 모든 명령이 실패합니다.

이제 터미널을 엽니다. Windows 검색창에 `PowerShell`을 입력해 실행하세요. 아래 명령으로 정상 설치를 확인합니다.

```powershell
docker --version
docker compose version
```

두 명령 모두 버전 번호가 나오면 준비 완료입니다.

> **[갱신 사항] `docker-compose`가 아니라 `docker compose`입니다**
> 원본 자료에는 `docker-compose --version`처럼 하이픈이 들어간 옛 방식이 적혀 있습니다. 현재 Docker Desktop에 들어 있는 것은 Compose V2이며, **하이픈 없이 띄어쓰기**로 씁니다. 옛 명령도 호환을 위해 동작하는 경우가 있지만, 새 방식으로 쓰는 것이 맞습니다.

### 2.2 Claude Desktop 설치

`claude.ai/download`에서 Windows용 설치 파일을 받아 설치하고, 계정으로 로그인합니다.

> **가장 흔한 첫 실수 — 웹 브라우저로는 안 됩니다**
> 인터넷 브라우저에서 claude.ai에 접속해 사용하는 것은 "웹 버전"입니다. 웹 버전은 여러분 PC 안의 프로그램을 실행할 수 없어서 로컬 데이터베이스에 연결할 수 없습니다. 반드시 **PC에 설치한 데스크톱 앱**을 실행한 상태로 실습하세요.

설치 후 앱 메뉴에서 **업데이트 확인**을 눌러 최신 버전으로 맞춰 두는 것이 좋습니다. MCP 관련 기능은 계속 개선되고 있습니다.

### 2.3 작업 폴더 만들기

실습에 쓸 파일들을 한곳에 모아 둘 폴더를 만듭니다. PowerShell에서 아래를 그대로 실행하세요.

```powershell
cd $HOME
mkdir postgres-mcp-lab
cd postgres-mcp-lab
mkdir init-scripts
```

지금 만든 폴더의 **절대경로**를 확인해 메모장에 적어 두세요. 나중에 필요합니다.

```powershell
pwd
# 출력 예: C:\Users\내이름\postgres-mcp-lab
```

---

## 3. Step 1 — PostgreSQL 데이터베이스 만들기

### 3.1 왜 그냥 공식 이미지를 쓰지 않고 조금 손을 대나요?

Postgres MCP Pro의 진짜 실력인 **인덱스 자동 추천** 기능을 쓰려면 PostgreSQL에 두 가지 **확장 기능**이 설치되어 있어야 합니다. 공식 문서에도 명시된 내용입니다.

첫째는 `pg_stat_statements`입니다. 어떤 쿼리가 몇 번 실행되었고 얼마나 오래 걸렸는지를 기록해 주는 확장입니다. 이게 없으면 "가장 느린 쿼리 찾아줘"라는 요청에 답할 수 없습니다.

둘째는 `hypopg`입니다. **인덱스를 실제로 만들지 않은 상태에서, 만들었다고 가정하면 얼마나 빨라질지를 미리 계산**해 주는 확장입니다. 이 덕분에 "이 인덱스를 만들면 87% 빨라집니다"라는 구체적인 예측이 가능해집니다.

문제는 `hypopg`가 PostgreSQL 공식 Docker 이미지에 **포함되어 있지 않다**는 점입니다. 그래서 우리는 공식 이미지에 이 확장 하나만 추가한 나만의 이미지를 만듭니다. 어렵지 않습니다. 파일 하나에 다섯 줄만 적으면 됩니다.

### 3.2 Dockerfile 만들기

`postgres-mcp-lab` 폴더 안에 **`Dockerfile`** 이라는 이름의 파일을 만듭니다. 확장자는 없습니다. 메모장으로 만들 때는 저장할 때 파일 형식을 "모든 파일"로 바꾸고 파일 이름을 `Dockerfile`로 적어야 `.txt`가 붙지 않습니다.

```dockerfile
FROM postgres:17

# hypopg 확장 설치 (인덱스 시뮬레이션용)
# 공식 postgres 이미지는 PostgreSQL 공식 apt 저장소(PGDG)가 이미 등록되어 있어
# 아래 한 줄로 설치가 됩니다.
RUN apt-get update \
    && apt-get install -y --no-install-recommends postgresql-17-hypopg \
    && rm -rf /var/lib/apt/lists/*
```

> **만약 위 설치가 실패한다면**
> 네트워크나 저장소 상황에 따라 패키지를 찾지 못할 수 있습니다. 그럴 때는 `hypopg`가 이미 들어 있는 공개 이미지를 쓰는 것이 가장 빠른 우회로입니다. Dockerfile을 지우고, 다음 절의 `docker-compose.yml`에서 `build: .` 부분을 `image: postgresai/extended-postgres:17`로 바꾸면 됩니다. 이 이미지는 HypoPG 등을 미리 포함해 배포되는 확장 이미지입니다.
> 그래도 안 되면 `hypopg` 없이 진행해도 됩니다. 인덱스 효과 예측 기능만 제한되고, 나머지 실습(스키마 조회·헬스체크·느린 쿼리 분석·시각화)은 모두 정상 동작합니다.

### 3.3 docker-compose.yml 만들기

같은 폴더에 **`docker-compose.yml`** 파일을 만들고 아래 내용을 넣습니다.

```yaml
services:
  postgres:
    build: .
    container_name: pg-mcp-lab
    environment:
      POSTGRES_DB: moviedb
      POSTGRES_USER: mcpuser
      POSTGRES_PASSWORD: mcppass1234
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    command: >
      postgres
      -c shared_preload_libraries=pg_stat_statements
      -c pg_stat_statements.track=all
      -c pg_stat_statements.max=10000

volumes:
  pgdata:
```

각 부분이 무슨 뜻인지 짚어 보겠습니다.

`environment` 블록의 세 줄은 데이터베이스 이름을 `moviedb`, 접속 계정을 `mcpuser`, 비밀번호를 `mcppass1234`로 정한 것입니다. 이 세 값은 뒤에서 Claude Desktop 설정에 그대로 다시 쓰이니 기억해 두세요.

`ports`의 `"5432:5432"`는 내 PC의 5432번 문을 컨테이너의 5432번 문에 연결한다는 뜻입니다. 이 연결이 있어야 PC에서 데이터베이스에 접근할 수 있습니다.

`volumes`의 첫 줄은 데이터를 컨테이너 밖에 저장해 컨테이너를 지웠다 다시 만들어도 데이터가 남게 하는 설정입니다. 두 번째 줄은 우리가 만든 `init-scripts` 폴더를 컨테이너 안의 특별한 위치에 연결하는데, 이 위치에 넣어 둔 `.sql` 파일들은 **데이터베이스가 처음 만들어질 때 자동으로 실행**됩니다. 다음 단계에서 이 성질을 이용해 실습 데이터를 넣습니다.

`command` 블록은 `pg_stat_statements`를 미리 메모리에 올리라고 지시하는 부분입니다. 이 확장은 다른 확장들과 달리 데이터베이스가 켜질 때부터 준비되어 있어야 하기 때문에 이렇게 별도로 지정해야 합니다.

> **[갱신 사항] `version: '3.8'` 줄은 이제 쓰지 않습니다**
> 원본 자료의 `docker-compose.yml` 첫 줄에는 `version: '3.8'`이 있습니다. Compose V2에서 이 항목은 **더 이상 사용하지 않는 항목(obsolete)** 으로 분류되어, 남겨 두면 실행할 때 경고 메시지가 출력됩니다. 그래서 위 예시에서는 제거했습니다.
>
> **[주의] 데이터 볼륨 경로**
> PostgreSQL 17 이하에서는 데이터 볼륨을 반드시 `/var/lib/postgresql/data`에 연결해야 합니다. 한 단계 위인 `/var/lib/postgresql`에 연결하면 컨테이너를 다시 만들 때 데이터가 사라집니다. 위 설정은 올바른 경로를 사용하고 있습니다.

---

## 4. Step 2 — 실습용 데이터 만들기

### 4.1 어떤 데이터를 만드나요?

**영화관 예약 시스템**을 주제로 6개의 테이블을 만듭니다. 원본 자료는 8개 테이블을 사용했지만, 초보자가 한눈에 관계를 파악할 수 있도록 배우 관련 테이블 2개를 덜어 냈습니다. 인덱스 튜닝 실습에 필요한 복잡도는 충분히 유지됩니다.

```mermaid
erDiagram
    GENRES ||--o{ MOVIES : "장르"
    DIRECTORS ||--o{ MOVIES : "감독"
    MOVIES ||--o{ SHOWTIMES : "상영"
    THEATERS ||--o{ SHOWTIMES : "장소"
    MOVIES ||--o{ REVIEWS : "리뷰"

    GENRES {
        int id PK
        string name
    }
    DIRECTORS {
        int id PK
        string name
        int birth_year
        string nationality
    }
    MOVIES {
        int id PK
        string title
        int director_id FK
        int genre_id FK
        int release_year
        int duration_minutes
        bigint budget
        bigint box_office
        decimal rating
    }
    THEATERS {
        int id PK
        string name
        string city
        int total_seats
    }
    SHOWTIMES {
        int id PK
        int movie_id FK
        int theater_id FK
        date show_date
        time show_time
        decimal ticket_price
        int tickets_sold
    }
    REVIEWS {
        int id PK
        int movie_id FK
        string reviewer_name
        int rating
        text review_text
        date review_date
        int helpful_votes
    }
```

데이터 양은 아래와 같이 준비합니다. **일부러 인덱스를 만들지 않은 상태**로 시작하는 것이 핵심입니다. 그래야 Claude가 "여기가 느립니다, 이 인덱스를 만드세요"라고 진단할 거리가 생깁니다.

| 테이블 | 건수 | 목적 |
|---|---|---|
| `genres` | 10건 | 기준 데이터 |
| `directors` | 10건 | 기준 데이터 |
| `theaters` | 10건 | 기준 데이터 |
| `movies` | 1,010건 | 실제 영화 10건 + 성능 테스트용 1,000건 |
| `reviews` | 20,000건 | 대량 조회·집계 실습용 |
| `showtimes` | 50,000건 | 날짜 범위 검색·매출 집계 실습용 |

### 4.2 스키마 스크립트 만들기

`init-scripts` 폴더 안에 **`01-schema.sql`** 파일을 만들고 아래 내용을 넣습니다.

```sql
-- ============================================
-- 01-schema.sql : 확장 설치 + 테이블 생성
-- ============================================

-- 성능 분석용 확장
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 인덱스 시뮬레이션용 확장 (없으면 이 줄에서 오류가 나지만 무시해도 됩니다)
CREATE EXTENSION IF NOT EXISTS hypopg;

-- 장르
CREATE TABLE genres (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL UNIQUE,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 감독
CREATE TABLE directors (
    id           SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    birth_year   INTEGER,
    nationality  VARCHAR(100),
    created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 영화
CREATE TABLE movies (
    id                SERIAL PRIMARY KEY,
    title             VARCHAR(300) NOT NULL,
    director_id       INTEGER REFERENCES directors(id),
    genre_id          INTEGER REFERENCES genres(id),
    release_year      INTEGER,
    duration_minutes  INTEGER,
    budget            BIGINT,
    box_office        BIGINT,
    rating            DECIMAL(3,1),
    description       TEXT,
    created_at        TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 극장
CREATE TABLE theaters (
    id           SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    city         VARCHAR(100),
    total_seats  INTEGER,
    created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 상영 정보
CREATE TABLE showtimes (
    id            SERIAL PRIMARY KEY,
    movie_id      INTEGER REFERENCES movies(id),
    theater_id    INTEGER REFERENCES theaters(id),
    show_date     DATE,
    show_time     TIME,
    ticket_price  DECIMAL(10,2),
    tickets_sold  INTEGER DEFAULT 0,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 리뷰
CREATE TABLE reviews (
    id             SERIAL PRIMARY KEY,
    movie_id       INTEGER REFERENCES movies(id),
    reviewer_name  VARCHAR(200),
    rating         INTEGER CHECK (rating >= 1 AND rating <= 10),
    review_text    TEXT,
    review_date    DATE DEFAULT CURRENT_DATE,
    helpful_votes  INTEGER DEFAULT 0,
    created_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

`hypopg` 확장 설치가 실패해도 뒤 단계는 정상 진행됩니다. 이 확장이 없으면 인덱스 효과를 미리 계산하는 기능만 제한됩니다.

### 4.3 데이터 스크립트 만들기

같은 폴더에 **`02-data.sql`** 파일을 만들고 아래 내용을 넣습니다. 앞부분은 사람이 알아볼 수 있는 실제 데이터이고, 뒷부분은 `generate_series`라는 기능으로 대량 데이터를 자동 생성하는 부분입니다.

```sql
-- ============================================
-- 02-data.sql : 실습 데이터 삽입
-- ============================================

-- 장르 10건
INSERT INTO genres (name) VALUES
('액션'), ('드라마'), ('코미디'), ('스릴러'), ('SF'),
('로맨스'), ('호러'), ('판타지'), ('애니메이션'), ('다큐멘터리');

-- 감독 10건
INSERT INTO directors (name, birth_year, nationality) VALUES
('크리스토퍼 놀란', 1970, '영국'),
('봉준호', 1969, '한국'),
('스티븐 스필버그', 1946, '미국'),
('박찬욱', 1963, '한국'),
('쿠엔틴 타란티노', 1963, '미국'),
('미야자키 하야오', 1941, '일본'),
('드니 빌뇌브', 1967, '캐나다'),
('조던 필', 1979, '미국'),
('그레타 거윅', 1983, '미국'),
('라이언 쿠글러', 1986, '미국');

-- 극장 10건
INSERT INTO theaters (name, city, total_seats) VALUES
('CGV 강남', '서울', 300),
('롯데시네마 월드타워', '서울', 400),
('메가박스 코엑스', '서울', 350),
('CGV 센텀시티', '부산', 280),
('롯데시네마 대구', '대구', 320),
('메가박스 광주', '광주', 290),
('CGV 대전', '대전', 310),
('롯데시네마 인천', '인천', 340),
('메가박스 울산', '울산', 270),
('CGV 수원', '수원', 330);

-- 영화 10건 (실제 작품)
INSERT INTO movies
  (title, director_id, genre_id, release_year, duration_minutes, budget, box_office, rating, description)
VALUES
('기생충', 2, 4, 2019, 132, 11400000, 258800000, 8.6, '반지하 가족과 상류층 가족이 만나 벌어지는 사건'),
('인셉션', 1, 5, 2010, 148, 160000000, 836800000, 8.8, '꿈 속의 꿈을 파고드는 SF 스릴러'),
('올드보이', 4, 4, 2003, 120, 4000000, 15000000, 8.4, '15년간 감금당한 남자의 복수 이야기'),
('쥬라기 공원', 3, 5, 1993, 127, 63000000, 1046000000, 8.2, '복원된 공룡들이 살아나는 테마파크'),
('킬 빌', 5, 1, 2003, 111, 30000000, 180900000, 8.2, '배신당한 신부의 복수극'),
('센과 치히로의 행방불명', 6, 9, 2001, 125, 19000000, 395800000, 8.6, '신비한 세계에 빠진 소녀의 모험'),
('듄', 7, 5, 2021, 155, 165000000, 401800000, 8.0, '사막 행성 아라키스의 대서사시'),
('겟 아웃', 8, 7, 2017, 104, 4500000, 255400000, 7.7, '인종차별을 소재로 한 심리 호러'),
('레이디 버드', 9, 2, 2017, 94, 10000000, 79000000, 7.4, '고등학생 소녀의 성장 이야기'),
('블랙 팬서', 10, 1, 2018, 134, 200000000, 1348000000, 7.3, '와칸다의 왕 티찰라의 이야기');

-- 영화 1,000건 추가 (성능 테스트용)
INSERT INTO movies
  (title, director_id, genre_id, release_year, duration_minutes, budget, box_office, rating, description)
SELECT
    '테스트 영화 ' || g,
    (g % 10) + 1,
    (g % 10) + 1,
    2000 + (g % 25),
    90 + (g % 60),
    1000000 + (g * 100000),
    5000000 + (g * 500000),
    ROUND((5.0 + (g % 50) / 10.0)::numeric, 1),
    '성능 테스트용 영화 설명 ' || g
FROM generate_series(1, 1000) AS g;

-- 상영 정보 50,000건
-- movies.id 는 1~1010, theaters.id 는 1~10 범위입니다.
INSERT INTO showtimes
  (movie_id, theater_id, show_date, show_time, ticket_price, tickets_sold)
SELECT
    1 + floor(random() * 1010)::int,
    1 + floor(random() * 10)::int,
    CURRENT_DATE - floor(random() * 365)::int,
    ('10:00'::time + (floor(random() * 12) || ' hours')::interval)::time,
    8000 + floor(random() * 7000)::int,
    floor(random() * 300)::int
FROM generate_series(1, 50000);

-- 리뷰 20,000건
INSERT INTO reviews
  (movie_id, reviewer_name, rating, review_text, review_date, helpful_votes)
SELECT
    1 + floor(random() * 1010)::int,
    '리뷰어' || g,
    1 + floor(random() * 10)::int,
    '영화 리뷰 내용 ' || g || '. 이 영화는 인상적이었습니다.',
    CURRENT_DATE - floor(random() * 400)::int,
    floor(random() * 100)::int
FROM generate_series(1, 20000) AS g;

-- 통계 정보 갱신 (실행 계획 정확도를 높이기 위해)
ANALYZE;
```

### 4.4 느린 쿼리를 미리 만들어 두기 (중요)

Claude에게 "가장 느린 쿼리를 찾아줘"라고 부탁하려면, **먼저 느린 쿼리가 실행된 기록이 남아 있어야** 합니다. `pg_stat_statements`는 실행된 쿼리를 기록하는 장치일 뿐, 없는 기록을 만들어 내지는 못합니다.

그래서 일부러 무거운 쿼리를 몇 번 돌려 기록을 쌓아 둡니다. `init-scripts` 폴더에 **`03-warmup.sql`** 파일을 만듭니다.

```sql
-- ============================================
-- 03-warmup.sql : 느린 쿼리 기록 쌓기
-- 인덱스가 없는 상태에서 무거운 쿼리를 반복 실행해
-- pg_stat_statements 에 분석 대상을 만들어 둡니다.
-- ============================================

DO $$
DECLARE
    i INTEGER;
BEGIN
    FOR i IN 1..5 LOOP

        -- (1) 날짜 범위 + 도시 조건 조회 : 인덱스가 없어 전체 탐색
        PERFORM COUNT(*)
        FROM showtimes s
        JOIN movies   m ON s.movie_id   = m.id
        JOIN theaters t ON s.theater_id = t.id
        WHERE s.show_date BETWEEN CURRENT_DATE - 90 AND CURRENT_DATE
          AND t.city = '서울';

        -- (2) 영화별 리뷰 집계
        PERFORM m.title, COUNT(r.id), AVG(r.rating)
        FROM movies m
        LEFT JOIN reviews r ON m.id = r.movie_id
        GROUP BY m.id, m.title;

        -- (3) 극장별 매출 집계
        PERFORM t.name, SUM(s.tickets_sold * s.ticket_price)
        FROM theaters t
        JOIN showtimes s ON t.id = s.theater_id
        WHERE s.show_date >= CURRENT_DATE - 180
        GROUP BY t.id, t.name;

        -- (4) 제목 부분 일치 검색 : 인덱스가 없어 전체 탐색
        PERFORM COUNT(*)
        FROM movies
        WHERE title ILIKE '%테스트%';

        -- (5) 인기 리뷰 조회
        PERFORM r.id, r.reviewer_name, r.helpful_votes
        FROM reviews r
        WHERE r.helpful_votes >= 50
        ORDER BY r.helpful_votes DESC, r.review_date DESC
        LIMIT 100;

    END LOOP;
END $$;
```

### 4.5 데이터를 DB에 넣는 네 가지 방법

여기가 이 가이드에서 가장 실무적인 부분입니다. 위에서 만든 SQL 파일을 실제로 데이터베이스에 반영하는 방법은 여러 가지가 있고, 상황에 따라 골라 쓰면 됩니다.

```mermaid
flowchart TB
    SQL["작성한 SQL 파일"]

    subgraph WAYS["데이터를 DB에 넣는 4가지 경로"]
        A["A. init-scripts 자동 실행<br/>컨테이너 첫 기동 시 자동"]
        B["B. psql 명령으로 파일 실행<br/>이미 돌고 있는 DB에 추가"]
        C["C. GUI 도구로 붙여넣기<br/>DBeaver / pgAdmin"]
        D["D. Claude에게 부탁하기<br/>MCP 연동 후, 대화로 INSERT"]
    end

    SQL --> A
    SQL --> B
    SQL --> C
    SQL --> D

    style A fill:#E7F4EE,stroke:#1A7355
    style B fill:#E2F1F2,stroke:#0E7C86
    style C fill:#F3FAFA,stroke:#0E7C86
    style D fill:#FDF3E3,stroke:#B9741B
```

#### 방법 A — init-scripts 자동 실행 (이번 실습의 기본)

`init-scripts` 폴더에 `.sql` 파일을 넣어 두면, **데이터베이스가 처음 만들어질 때 파일 이름 순서대로 자동 실행**됩니다. 우리가 파일 이름을 `01-`, `02-`, `03-`으로 시작한 이유가 바로 이 순서를 통제하기 위한 것입니다.

이제 데이터베이스를 시작합니다.

```powershell
cd $HOME\postgres-mcp-lab
docker compose up -d --build
```

`--build`는 우리가 만든 Dockerfile로 이미지를 만들라는 뜻입니다. 처음에는 이미지를 내려받고 만드는 데 몇 분 걸립니다. 완료되면 아래로 상태를 확인합니다.

```powershell
docker compose ps
docker compose logs -f postgres
```

로그에 `database system is ready to accept connections`가 보이면 성공입니다. `Ctrl + C`로 로그 보기를 종료합니다.

데이터가 잘 들어갔는지 확인합니다.

```powershell
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb -c "SELECT (SELECT COUNT(*) FROM movies) AS movies, (SELECT COUNT(*) FROM showtimes) AS showtimes, (SELECT COUNT(*) FROM reviews) AS reviews;"
```

`movies 1010 | showtimes 50000 | reviews 20000` 형태로 나오면 완벽합니다.

> **[초보자 필수] init-scripts는 "처음 한 번만" 실행됩니다**
> 이 방식의 가장 흔한 함정입니다. `init-scripts`의 SQL은 데이터 저장소가 **비어 있을 때만** 실행됩니다. 이미 데이터가 들어 있는 상태에서 SQL 파일을 고치고 `docker compose restart`를 해도 **아무 일도 일어나지 않습니다.**
> 처음부터 다시 하려면 데이터 저장소까지 지워야 합니다.
> ```powershell
> docker compose down -v
> docker compose up -d --build
> ```
> `-v` 옵션이 데이터 저장소를 함께 지우는 부분입니다. 이 옵션 없이 `down`만 하면 데이터가 남아 있어 초기화 스크립트가 다시 실행되지 않습니다.

#### 방법 B — psql 명령으로 파일 실행 (돌고 있는 DB에 추가할 때)

데이터베이스가 이미 실행 중인 상태에서 SQL 파일을 추가로 반영하고 싶을 때 씁니다. 실무에서 가장 많이 쓰는 방식입니다.

```powershell
# PC의 SQL 파일을 컨테이너로 복사한 뒤 실행
docker cp .\init-scripts\03-warmup.sql pg-mcp-lab:/tmp/03-warmup.sql
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb -f /tmp/03-warmup.sql
```

파일을 복사하지 않고 곧바로 흘려 넣는 방법도 있습니다. 한 줄로 끝나서 편합니다.

```powershell
Get-Content .\init-scripts\03-warmup.sql | docker exec -i pg-mcp-lab psql -U mcpuser -d moviedb
```

SQL 한두 줄만 바로 실행하고 싶을 때는 `-c` 옵션을 씁니다.

```powershell
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb -c "SELECT COUNT(*) FROM movies;"
```

대화형으로 계속 작업하려면 `psql` 세션에 들어가면 됩니다. 나올 때는 `\q`입니다.

```powershell
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb
# moviedb=# \dt          <- 테이블 목록
# moviedb=# \dx          <- 설치된 확장 목록
# moviedb=# \q           <- 종료
```

#### 방법 C — GUI 도구로 붙여넣기 (검은 화면이 부담스러울 때)

명령줄이 익숙하지 않다면 화면을 보며 클릭으로 작업할 수 있는 도구를 쓰면 됩니다. **DBeaver**(무료)나 **pgAdmin**을 설치하고 아래 정보로 연결합니다.

| 항목 | 값 |
|---|---|
| Host | `localhost` |
| Port | `5432` |
| Database | `moviedb` |
| Username | `mcpuser` |
| Password | `mcppass1234` |

연결한 뒤 SQL 편집기 창에 SQL을 붙여넣고 실행 버튼을 누르면 됩니다. 결과가 표로 보이니 데이터를 눈으로 확인하기에 가장 편합니다.

#### 방법 D — Claude에게 부탁하기 (가장 바이브코딩다운 방법)

다음 단계에서 MCP 연동을 마치면, **Claude에게 한국어로 부탁해서 데이터를 넣을 수 있습니다.** 다만 이 방법은 Claude가 데이터를 변경할 권한을 가져야 하므로 `unrestricted` 모드가 필요합니다. 로컬 연습용 데이터베이스에서만 쓰세요.

연동 후 이렇게 부탁하면 됩니다.

```
moviedb 데이터베이스에 "즐겨찾기" 기능을 위한 테이블을 하나 추가하고 싶습니다.

요구사항:
- 테이블명: favorites
- 컬럼: id(자동증가), user_name(문자열), movie_id(movies 테이블 참조),
       created_at(생성시각 기본값)
- 같은 사용자가 같은 영화를 중복 등록할 수 없도록 제약조건 추가

테이블을 만든 뒤, movies 테이블에서 평점(rating)이 8.0 이상인 영화를 골라
'테스트유저1' ~ '테스트유저5' 이름으로 무작위 즐겨찾기 데이터 100건을 넣어주세요.

작업 전에 실행할 SQL을 먼저 보여주고, 제가 확인하면 실행해주세요.
```

마지막 줄이 중요합니다. **실행 전에 SQL을 먼저 보여 달라고 요청**하는 습관을 들이면, AI가 의도와 다른 작업을 하는 것을 미리 막을 수 있습니다.

---

## 5. Step 3 — Postgres MCP Pro 연동하기

### 5.1 이미지 내려받기

PowerShell에서 아래를 실행합니다.

```powershell
docker pull crystaldba/postgres-mcp
```

이 이미지에는 필요한 모든 것이 들어 있어서, 별도로 Python을 설치하거나 라이브러리를 맞출 필요가 없습니다.

### 5.2 Claude Desktop 설정 파일 열기

Windows에서 설정 파일 위치는 다음과 같습니다.

```
%APPDATA%\Claude\claude_desktop_config.json
```

파일을 여는 가장 쉬운 방법은 두 가지입니다. 하나는 `Win + R`을 누르고 `%APPDATA%\Claude`를 입력해 폴더를 여는 것이고, 다른 하나는 Claude Desktop에서 **설정(Settings) → 개발자(Developer) → 설정 편집(Edit Config)** 을 누르는 것입니다. 파일이 없으면 새로 만들면 됩니다.

### 5.3 설정 내용 작성

파일에 아래 내용을 넣습니다. 이미 다른 MCP 서버가 등록되어 있다면 `mcpServers` 안에 `postgres` 항목만 추가하세요.

```json
{
  "mcpServers": {
    "postgres": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "DATABASE_URI",
        "crystaldba/postgres-mcp",
        "--access-mode=unrestricted"
      ],
      "env": {
        "DATABASE_URI": "postgresql://mcpuser:mcppass1234@host.docker.internal:5432/moviedb"
      }
    }
  }
}
```

`DATABASE_URI`의 구조를 풀어 보면 이렇습니다.

```
postgresql://[계정]:[비밀번호]@[주소]:[포트]/[데이터베이스명]
             mcpuser  mcppass1234  host.docker.internal  5432  moviedb
```

여기서 주소가 `localhost`가 아니라 `host.docker.internal`인 이유가 있습니다. MCP 서버 자신도 Docker 컨테이너 안에서 돌기 때문에, 그 안에서의 `localhost`는 "내 PC"가 아니라 "그 컨테이너 자기 자신"을 가리킵니다. `host.docker.internal`이 컨테이너에서 내 PC를 부르는 정식 이름입니다.

> **참고**: Postgres MCP Pro의 Docker 이미지는 `localhost`를 자동으로 알맞은 주소로 바꿔 주는 기능을 갖고 있습니다(Windows/macOS는 `host.docker.internal`, Linux는 호스트 주소). 그래서 `localhost`로 적어도 동작합니다. 다만 초보자에게는 왜 되는지가 불투명하므로, 이 가이드에서는 명시적으로 `host.docker.internal`을 적었습니다.

### 5.4 접근 권한 모드 — 반드시 이해하고 넘어가세요

`--access-mode` 옵션은 **AI가 여러분 데이터베이스에 무엇을 할 수 있는지**를 정하는 스위치입니다. 이 실습에서 가장 중요한 안전 장치이므로 꼭 짚고 갑니다.

| 모드 | 권한 | 적합한 곳 |
|---|---|---|
| `unrestricted` | 읽기 + 쓰기 + 스키마 변경 전부 가능 | 개발·연습 환경 |
| `restricted` | 읽기 전용 트랜잭션만 허용, 쿼리 실행 시간 제한 | 운영·중요 데이터 환경 |

`restricted` 모드는 단순히 "쓰기 금지"에 그치지 않습니다. AI가 `ROLLBACK; DROP TABLE users;`처럼 읽기 전용 상태를 우회하려는 SQL을 보내는 경우까지 막기 위해, `pglast`라는 파서로 SQL을 먼저 해석해서 `COMMIT`이나 `ROLLBACK`이 포함된 문장을 거부합니다. 실제로 이런 우회가 가능하기 때문에 설계된 방어 장치입니다.

이 실습에서는 방법 D(Claude에게 데이터 넣기)를 해 보기 위해 `unrestricted`를 사용합니다. 하지만 **실제 업무 데이터베이스에는 절대 `unrestricted`로 연결하지 마세요.** 실습이 끝나면 설정을 `--access-mode=restricted`로 바꿔 두는 것을 권합니다.

> **[안전 수칙]** 이 실습용 데이터베이스는 언제든 지우고 다시 만들 수 있는 연습용입니다. 그래서 마음껏 실험해도 됩니다. 회사 데이터베이스나 실제 서비스 데이터베이스로는 이 실습을 하지 마세요. 계정 정보가 설정 파일에 평문으로 저장된다는 점도 함께 기억해 두시기 바랍니다.

### 5.5 Claude Desktop 재시작 및 확인

설정을 저장한 뒤 다음 순서를 지킵니다.

먼저 Claude Desktop을 **완전히 종료**합니다. 창의 X 버튼만 누르면 백그라운드에 남아 있으므로, 작업 관리자에서 Claude 프로세스를 종료하거나 트레이 아이콘에서 종료를 선택해야 합니다. 설정 파일은 앱이 켜질 때 한 번만 읽히기 때문에 이 과정이 꼭 필요합니다.

그다음 Claude Desktop을 다시 실행하고, 채팅 입력창 근처의 도구 아이콘을 눌러 봅니다. `postgres` 서버와 함께 앞에서 표로 정리한 9개 도구가 보이면 연동 성공입니다.

마지막으로 실제 대화로 확인합니다.

```
내 postgres 데이터베이스에 어떤 테이블들이 있는지 보여주세요.
각 테이블의 대략적인 행 수도 함께 알려주세요.
```

Claude가 도구 사용 승인을 요청하면 **허용**을 누릅니다. 6개 테이블과 행 수가 나오면 모든 준비가 끝났습니다.

---

## 6. Step 4 — 실습 시나리오 (프롬프트 그대로 복사해 사용)

이제부터가 본편입니다. 아래 프롬프트를 순서대로 하나씩 복사해서 Claude Desktop에 붙여넣으세요. 각 프롬프트가 어떤 도구를 쓰게 되는지도 함께 적었습니다.

### 6.1 데이터베이스 건강검진

```
내 데이터베이스의 전반적인 건강 상태를 점검해주세요.

버퍼 캐시 적중률, 연결 상태, 인덱스 건강(사용되지 않는 인덱스·중복 인덱스),
제약조건 유효성, 배큠 상태, 시퀀스 한계를 모두 확인하고,
문제가 있는 항목은 심각도 순서로 정리해서 알려주세요.

각 항목이 왜 중요한지도 초보자가 이해할 수 있게 한 줄씩 설명해주세요.
```

이 프롬프트는 `analyze_db_health` 도구를 호출합니다. 이 도구의 점검 항목들은 PgHero라는 널리 쓰이는 오픈소스 성능 대시보드의 검사 항목을 그대로 가져와 구현한 것이라, 실무에서도 통하는 기준입니다.

### 6.2 느린 쿼리 찾기

```
현재 데이터베이스에서 가장 느린 쿼리 5개를 찾아주세요.

각 쿼리에 대해 다음을 알려주세요:
1. 무엇을 조회하는 쿼리인지 한국어로 설명
2. 총 실행 시간과 평균 실행 시간
3. 왜 느린지 (어떤 부분이 병목인지)
4. 개선하면 얼마나 빨라질 것으로 예상되는지

전문 용어는 괄호 안에 쉬운 설명을 덧붙여주세요.
```

`get_top_queries` 도구가 사용됩니다. 앞에서 `03-warmup.sql`을 실행해 두었기 때문에 분석할 대상이 존재합니다. 만약 "느린 쿼리가 없습니다"라는 답이 오면 워밍업 스크립트를 다시 실행해 보세요.

### 6.3 인덱스 추천받기

```
데이터베이스 워크로드 전체를 분석해서, 성능 향상에 가장 효과적인 인덱스를 추천해주세요.

조건:
- 디스크 공간은 최대 500MB까지만 쓸 수 있습니다
- 읽기와 쓰기 비율은 대략 8:2 입니다

추천하는 각 인덱스에 대해 다음을 알려주세요:
1. 생성할 CREATE INDEX 문장
2. 어떤 쿼리가 빨라지는지
3. 예상 성능 개선 폭 (몇 배 또는 몇 퍼센트)
4. 인덱스 크기와 유지보수 부담

바로 실행하지는 말고, 추천 목록만 먼저 보여주세요.
```

`analyze_workload_indexes` 도구가 사용됩니다. 이 도구는 앞서 설명한 Anytime Algorithm으로 인덱스 조합을 탐색하고, `hypopg`로 각 후보의 효과를 실제 PostgreSQL 비용 모델로 예측합니다. 그래서 결과에 근거가 있습니다.

### 6.4 특정 쿼리 최적화 (인덱스 적용 전후 비교)

```
다음 쿼리가 너무 느립니다. 실행 계획을 분석하고 최적화해주세요.

SELECT s.show_date, s.show_time, m.title, t.name AS theater, t.city,
       s.ticket_price, s.tickets_sold
FROM showtimes s
JOIN movies m   ON s.movie_id = m.id
JOIN theaters t ON s.theater_id = t.id
WHERE s.show_date BETWEEN CURRENT_DATE - 90 AND CURRENT_DATE
  AND t.city = '서울'
ORDER BY s.show_date DESC, s.tickets_sold DESC
LIMIT 50;

요청사항:
1. 현재 실행 계획을 보여주고, 어느 단계에서 시간이 가장 많이 걸리는지 설명
2. 가상 인덱스를 적용했을 때의 실행 계획을 시뮬레이션해서 비교
3. 실제로 만들어야 할 인덱스를 추천
4. 개선 전후를 표로 정리
```

`explain_query`가 사용됩니다. 3번 항목이 이 도구의 백미인데, **인덱스를 실제로 만들지 않은 상태에서** 만들었다고 가정한 실행 계획을 뽑아 비교해 줍니다. 실무에서는 인덱스를 잘못 만들면 지우는 것도 부담이므로 이 기능의 가치가 큽니다.

### 6.5 인덱스를 실제로 적용하고 효과 측정하기

```
앞에서 추천받은 인덱스 중 효과가 가장 큰 것 2~3개를 실제로 만들어주세요.

절차:
1. 만들기 전에 대상 쿼리의 실행 시간을 측정해서 기록
2. CREATE INDEX 실행 (실행할 SQL을 먼저 보여주고 진행)
3. ANALYZE 로 통계 갱신
4. 같은 쿼리를 다시 측정
5. 개선 전후를 표로 비교 정리

각 단계에서 무엇을 하는지 설명해주세요.
```

이 프롬프트는 데이터베이스를 변경하므로 `unrestricted` 모드에서만 동작합니다. "실행할 SQL을 먼저 보여주고 진행"이라는 조건을 넣은 이유는 앞에서 설명한 안전 습관 때문입니다.

### 6.6 데이터 자체를 탐색해 보기

성능 분석만이 아니라 평범한 데이터 조회도 물론 됩니다.

```
우리 영화 데이터베이스에서 다음을 알려주세요:

1. 감독별 평균 평점 순위 (영화 3편 이상인 감독만)
2. 장르별 총 매출 순위
3. 최근 90일 동안 극장별 매출 순위
4. 리뷰 수가 가장 많은 영화 10편과 그 평균 평점

결과를 표로 정리하고, 눈에 띄는 특징이 있으면 함께 설명해주세요.
```

---

## 7. Step 5 — 결과를 아티팩트와 시각화로 보여주기

여기가 이번 가이드의 하이라이트입니다. 지금까지는 결과가 글과 표로만 나왔는데, Claude에게 부탁하면 **보기 좋은 대시보드나 다이어그램**으로 만들어 줍니다.

### 7.1 먼저 알아야 할 두 가지 방식과 결정적인 제약

Claude가 시각적 결과물을 만들어 주는 방식은 크게 두 가지입니다.

**아티팩트(Artifacts)** 는 대화창 옆에 별도 패널이 열리면서 그 안에 HTML이나 React로 만든 **동작하는 화면**이 나타나는 방식입니다. 버튼을 누르고 값을 바꿀 수 있는 인터랙티브한 대시보드를 만들 수 있고, 파일로 내려받아 브라우저에서 열 수도 있습니다. Claude Desktop 설정에서 아티팩트 기능이 켜져 있어야 합니다.

**대화 내 시각화**는 다이어그램이나 차트를 대화 흐름 안에 바로 그려 주는 방식입니다. 구조도나 관계도처럼 "한 장 그림"이 필요할 때 적합합니다.

그리고 **초보자가 반드시 알아야 할 결정적인 제약**이 하나 있습니다.

> **아티팩트는 데이터베이스에 직접 접속할 수 없습니다.**
> 아티팩트는 안전을 위해 격리된 환경에서 실행되기 때문에, 그 안의 코드가 여러분의 PostgreSQL에 접속하거나 MCP 도구를 호출할 수 없습니다. 즉 아티팩트는 **실시간으로 갱신되는 대시보드가 아닙니다.**
>
> 그래서 순서가 중요합니다. **① Claude가 MCP로 데이터를 조회한다 → ② 그 결과 값을 아티팩트 코드 안에 박아 넣어 화면을 만든다** 는 두 단계로 진행해야 합니다. 프롬프트에 이 순서를 명시하면 Claude가 정확히 그렇게 해 줍니다.

```mermaid
flowchart LR
    Q["1. 프롬프트<br/>'조회한 뒤 대시보드로 만들어줘'"]
    M["2. MCP 도구로<br/>실제 DB 조회"]
    R["3. 조회 결과<br/>(숫자·표)"]
    A["4. 결과를 담은<br/>아티팩트 생성"]

    Q --> M --> R --> A

    style Q fill:#E2F1F2,stroke:#0E7C86
    style M fill:#FDF3E3,stroke:#B9741B
    style R fill:#F3FAFA,stroke:#0E7C86
    style A fill:#E7F4EE,stroke:#1A7355
```

### 7.2 시각화 프롬프트 1 — 데이터베이스 구조도 (ERD)

```
내 moviedb 데이터베이스의 전체 구조를 파악해서 관계도(ERD)로 그려주세요.

절차:
1. 먼저 모든 테이블과 각 테이블의 컬럼, 기본키, 외래키를 조회해주세요
2. 조회한 실제 구조를 바탕으로 관계도를 그려주세요

표현 요구사항:
- 테이블 간 관계(1:N)를 선으로 표시
- 각 테이블의 주요 컬럼과 데이터 타입 표기
- 기본키(PK)와 외래키(FK) 구분 표시
- 각 테이블의 실제 행 수를 함께 표기

추측하지 말고 반드시 실제 조회 결과만 사용해주세요.
```

마지막 줄이 중요합니다. 이렇게 못 박아 두면 Claude가 "아마 이런 구조일 것"이라고 지어내지 않고 반드시 도구로 확인합니다.

### 7.3 시각화 프롬프트 2 — DB 건강검진 대시보드 (아티팩트)

```
내 데이터베이스의 건강 상태를 점검하고, 그 결과를 한 화면짜리 대시보드로 만들어주세요.

1단계 — 데이터 수집:
데이터베이스 헬스 체크를 실행해서 버퍼 캐시 적중률, 연결 상태,
인덱스 건강, 배큠 상태, 제약조건 유효성 결과를 수집해주세요.

2단계 — 대시보드 제작:
수집한 실제 수치를 사용해서 HTML 아티팩트로 대시보드를 만들어주세요.

디자인 요구사항:
- 맨 위에 종합 점수(100점 만점)와 한 줄 요약
- 항목별 카드 배치, 상태에 따라 초록(정상)/노랑(주의)/빨강(위험) 색상 구분
- 각 카드에 수치, 권장 기준값, 초보자용 한 줄 설명
- 문제가 있는 항목은 해결 방법을 함께 표시
- 한국어 표기, 별도 라이브러리 없이 단일 HTML 파일로 동작

수치는 반드시 1단계에서 실제로 조회한 값을 사용하고,
값을 얻지 못한 항목은 "측정 불가"로 표시해주세요.
```

### 7.4 시각화 프롬프트 3 — 느린 쿼리 순위 차트 (아티팩트)

```
가장 느린 쿼리 상위 10개를 조회해서 시각적으로 비교할 수 있는 차트를 만들어주세요.

1단계: 느린 쿼리 상위 10개의 쿼리 내용, 총 실행 시간, 호출 횟수,
       평균 실행 시간을 조회해주세요.

2단계: 조회한 실제 데이터로 React 아티팩트를 만들어주세요.
- 가로 막대 차트로 총 실행 시간 비교
- 막대를 클릭하면 해당 쿼리의 전문과 상세 지표가 아래에 표시
- "총 시간 기준" / "평균 시간 기준" 정렬 전환 버튼
- 각 쿼리가 무슨 일을 하는지 한국어 한 줄 요약을 함께 표시
- recharts 라이브러리 사용, 한국어 UI

쿼리 전문이 너무 길면 앞부분 200자만 표시하고 나머지는 접어주세요.
```

### 7.5 시각화 프롬프트 4 — 인덱스 적용 전후 비교 (아티팩트)

```
인덱스 최적화의 효과를 한눈에 볼 수 있는 비교 자료를 만들어주세요.

1단계: 아래 쿼리의 현재 실행 계획과 예상 비용을 조회해주세요.

SELECT s.show_date, m.title, t.name, s.tickets_sold
FROM showtimes s
JOIN movies m   ON s.movie_id = m.id
JOIN theaters t ON s.theater_id = t.id
WHERE s.show_date BETWEEN CURRENT_DATE - 90 AND CURRENT_DATE
  AND t.city = '서울'
ORDER BY s.show_date DESC
LIMIT 50;

2단계: 추천 인덱스를 가상으로 적용한 실행 계획을 시뮬레이션해주세요.

3단계: 1단계와 2단계 결과를 비교하는 아티팩트를 만들어주세요.
- 화면을 좌우로 나눠 "인덱스 없음" vs "인덱스 적용" 배치
- 각 쪽에 실행 계획을 트리 형태로 표시하고, 비용이 큰 단계를 강조
- 가운데에 개선 배수를 큰 숫자로 표시
- 아래에 실제로 실행해야 할 CREATE INDEX 문장을 복사 버튼과 함께 배치
- 전체 한국어, 단일 HTML
```

### 7.6 시각화 프롬프트 5 — 극장 매출 대시보드 (아티팩트)

성능 분석이 아니라 **업무 데이터 대시보드**를 만드는 예시입니다. 실무에서 가장 많이 쓰게 될 형태입니다.

```
극장 운영 현황 대시보드를 만들어주세요.

1단계 — 아래 데이터를 조회해주세요:
- 극장별 최근 90일 총 매출과 총 관객 수
- 극장별 좌석 점유율 (tickets_sold 합계 / (total_seats × 상영 횟수))
- 최근 30일 일별 전체 매출 추이
- 매출 상위 영화 10편
- 시간대별 평균 관객 수

2단계 — 조회한 실제 데이터로 React 아티팩트 대시보드를 만들어주세요:
- 상단에 핵심 지표 4개 (총 매출, 총 관객, 평균 점유율, 상영 횟수)를 큰 숫자로
- 극장별 매출 막대 차트
- 일별 매출 추이 선 차트
- 매출 상위 영화 목록 표
- 시간대별 관객 수 차트
- 극장을 선택하면 해당 극장 기준으로 필터링되는 드롭다운

디자인:
- 금액은 천 단위 구분 기호와 "원" 단위로 표기
- 한국어 UI, recharts 사용
- 노트북 화면에서 스크롤 없이 보이도록 배치

조회 결과가 많으면 상위 항목만 사용하고, 그 사실을 화면에 표시해주세요.
```

### 7.7 시각화가 잘 나오게 하는 다섯 가지 요령

경험상 아래 다섯 가지를 챙기면 결과물의 품질이 크게 올라갑니다.

첫째, **단계를 명시적으로 나눠 지시**합니다. "조회해서 대시보드 만들어줘"보다 "1단계에서 조회하고, 2단계에서 그 결과로 만들어줘"가 훨씬 정확합니다.

둘째, **추측을 금지하는 문장을 넣습니다.** "실제 조회 결과만 사용하고, 값을 얻지 못한 항목은 측정 불가로 표시해주세요"라는 한 줄이 그럴듯한 가짜 숫자를 막아 줍니다.

셋째, **결과물의 형태를 구체적으로 지정**합니다. 차트 종류, 색상 규칙, 배치, 언어, 사용할 라이브러리까지 적어 주면 다시 요청하는 횟수가 줄어듭니다.

넷째, **한 번에 완성하려 하지 않습니다.** 일단 받아 보고 "막대 색을 매출 규모에 따라 진하게 해줘", "모바일에서도 보이게 해줘"처럼 같은 대화에서 이어 고쳐 나가는 편이 빠릅니다. 새 대화를 열면 앞 결과를 기억하지 못하므로 반드시 같은 대화에서 요청하세요.

다섯째, **데이터가 크면 미리 줄이라고 지시**합니다. 5만 건을 그대로 아티팩트에 담으려 하면 화면이 무거워집니다. "상위 20개만" 또는 "일별로 집계한 값만" 같은 조건을 넣는 것이 좋습니다.

---

## 8. 문제가 생겼을 때

### 8.1 증상별 해결 방법

| 증상 | 원인 | 해결 |
|---|---|---|
| Claude Desktop에 postgres 도구가 안 보임 | 설정 JSON 문법 오류 또는 완전 종료를 안 함 | 쉼표·따옴표·중괄호 확인, 작업 관리자에서 Claude 완전 종료 후 재시작 |
| 도구는 보이는데 연결 실패 | Docker Desktop이 꺼져 있음 | Docker Desktop 실행 후 상태가 Running인지 확인 |
| `could not connect to server` | DB 컨테이너가 안 떠 있거나 주소가 틀림 | `docker compose ps` 확인, `DATABASE_URI`의 주소를 `host.docker.internal`로 확인 |
| 테이블이 하나도 없음 | init-scripts가 실행되지 않았음 | `docker compose down -v` 후 `docker compose up -d --build` 로 처음부터 재생성 |
| 느린 쿼리가 없다고 나옴 | `pg_stat_statements`에 기록이 없음 | `03-warmup.sql`을 방법 B로 다시 실행 |
| 인덱스 추천이 부실함 | `hypopg` 확장이 없음 | `\dx`로 확인, 없으면 Dockerfile 재빌드 또는 확장 포함 이미지 사용 |
| `pg_stat_statements` 조회 오류 | 확장이 메모리에 올라가지 않음 | `docker compose down` 후 `up`으로 재시작 (compose의 command 설정 반영) |
| 포트 5432 충돌 | PC에 이미 PostgreSQL이 설치되어 있음 | compose의 포트를 `"5433:5432"`로 바꾸고 URI도 5433으로 수정 |
| 파일이 `.txt`로 저장됨 | 메모장 기본 저장 방식 | 저장 시 파일 형식을 "모든 파일"로 변경 |

### 8.2 상태 확인 명령 모음

```powershell
# 컨테이너 상태
docker compose ps

# DB 로그 보기 (Ctrl+C 로 종료)
docker compose logs -f postgres

# 설치된 확장 확인
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb -c "\dx"

# pg_stat_statements 동작 확인
docker exec -it pg-mcp-lab psql -U mcpuser -d moviedb -c "SELECT calls, round(total_exec_time::numeric,1) AS total_ms, left(query,60) AS q FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 5;"

# MCP 서버 이미지가 잘 받아졌는지 확인
docker images crystaldba/postgres-mcp
```

### 8.3 막혔을 때 가장 빠른 방법

이 가이드의 어느 단계에서든 오류가 나면, **오류 메시지 전체를 그대로 복사해서 Claude Desktop에 붙여넣고 "이 오류 해결해줘"라고 요청**하세요. 요약하거나 일부만 옮기지 말고, 파일 경로와 줄 번호까지 전부 포함해야 정확한 답이 나옵니다.

이것이 이 실습에서 배우는 기술 중 실무에서 가장 오래 쓰이는 습관입니다.

---

## 9. 실습 마무리 — 정리하기

### 9.1 다음에 또 쓰려면

```powershell
# 잠시 중단 (데이터 유지)
cd $HOME\postgres-mcp-lab
docker compose stop

# 다시 시작
docker compose start
```

### 9.2 완전히 지우려면

```powershell
# 컨테이너와 데이터 전부 삭제
cd $HOME\postgres-mcp-lab
docker compose down -v

# MCP 서버 이미지도 삭제
docker rmi crystaldba/postgres-mcp
```

그리고 `claude_desktop_config.json`에서 `postgres` 항목을 지우고 Claude Desktop을 재시작하면 PC가 원래 상태로 돌아갑니다.

### 9.3 실습이 끝난 뒤 권장 설정

계속 사용할 계획이라면 설정을 안전한 쪽으로 바꿔 두세요.

```json
"crystaldba/postgres-mcp",
"--access-mode=restricted"
```

읽기 전용으로 바뀌므로 실수로 데이터가 변경되는 일을 막을 수 있습니다.

---

## 10. 원본 자료 대비 갱신된 내용

첨부 자료들은 작성 시점 기준으로 정확했지만, 그 이후 확인된 변화와 이번 실습 환경에 맞춘 조정 사항을 아래에 정리했습니다. **확인된 사실**로 표시한 항목은 공식 저장소 문서와 공식 배포처에서 직접 확인한 내용입니다.

| 항목 | 원본 내용 | 2026-07-29 기준 확인 사실 | 구분 |
|---|---|---|---|
| Compose 명령 | `docker-compose --version`, `docker-compose up -d` | 현재 표준은 Compose V2로 하이픈 없이 `docker compose` | 확인된 사실 |
| Compose 파일 | 첫 줄에 `version: '3.8'` | Compose V2에서 `version` 항목은 더 이상 사용하지 않는 항목으로 분류되어 경고가 출력됨 | 확인된 사실 |
| Postgres 버전 | `postgres:15` | Postgres MCP Pro는 15·16·17을 중심으로 테스트하며 13~17 지원 예정. 본 가이드는 17 사용 | 확인된 사실 |
| hypopg 확장 | "포함되지 않을 수 있음, 별도 설치 필요"라고만 언급 | 실제 설치 방법을 Dockerfile로 제시. 공식 postgres Debian 이미지는 PGDG apt 저장소가 등록되어 있어 `postgresql-17-hypopg` 설치 가능. 대안으로 확장 포함 이미지 사용 가능 | 확인된 사실 |
| DB 접속 주소 | `host.docker.internal` 명시 | 여전히 유효. 추가로 Postgres MCP Pro Docker 이미지는 `localhost`를 자동 변환하는 기능이 있어 `localhost`로도 동작 | 확인된 사실 |
| 접근 권한 모드 | `--access-mode=unrestricted` 사용 | 두 모드의 차이와 위험을 명시하고, 실습 후 `restricted` 전환을 권고하도록 보강 | 가이드 보강 |
| MCP 서버 버전 | 별도 언급 없음 | 최신 정식 릴리스는 v0.3.0(2025년 5월 16일)이며 2026-07-29 시점에도 최신. Windows 지원과 LLM 기반 인덱스 최적화가 이 버전에서 추가됨. 저장소는 계속 활동 중(별 약 3천 개) | 확인된 사실 |
| 전송 방식 | 언급 없음 | 로컬 실습은 STDIO 방식 사용. Postgres MCP Pro는 SSE 전송도 지원하지만, MCP 표준에서는 HTTP+SSE가 지원 종료 단계로 재분류되었으므로 원격 공유는 신중히 검토 필요 | 확인된 사실 |
| 테이블 구성 | 8개 테이블(배우·영화배우 관계 포함) | 초보자 학습 부담을 줄여 6개로 축소. 인덱스 튜닝 실습에 필요한 복잡도는 유지 | 가이드 보강 |
| 대량 데이터 생성 | `(random() * 1009 + 1)::INTEGER` | `::INTEGER`는 반올림이므로 경계값에서 실제 ID 범위를 벗어날 수 있음. 본 가이드는 `1 + floor(random() * 1010)::int` 형태로 수정 | 가이드 보정 |
| 좌석 점유율 쿼리 | `s.total_seats` 참조 | 원본 스키마의 `showtimes`에는 해당 컬럼이 없어 실행 시 오류. 본 가이드는 `theaters.total_seats`를 두고 조인해 계산하도록 수정 | 가이드 보정 |
| 느린 쿼리 준비 | 초기화 스크립트 끝에 SELECT 몇 개 실행 | `PERFORM`을 사용한 반복 워밍업 스크립트로 분리해 기록이 확실히 쌓이도록 보강 | 가이드 보강 |
| 시각화 | 언급 없음 | 아티팩트가 DB에 직접 접속할 수 없다는 제약과, 2단계로 나눠 지시하는 프롬프트 패턴을 신규 추가 | 신규 |

---

## 11. 참고 자료

아래는 이 가이드를 작성하며 실제로 확인한 출처입니다. 이 분야는 변화가 빠르므로 실습 시점에 한 번 더 확인하시기를 권합니다.

- Postgres MCP Pro 공식 저장소 (설치·설정·도구 목록·기술 노트) — https://github.com/crystaldba/postgres-mcp
- Postgres MCP Pro 릴리스 목록 — https://github.com/crystaldba/postgres-mcp/releases
- PostgreSQL 공식 Docker 이미지 문서 (환경변수·볼륨 경로·초기화 스크립트) — https://hub.docker.com/_/postgres
- PostgreSQL APT 저장소(PGDG) 안내 — https://apt.postgresql.org/
- 확장 포함 Postgres 이미지 (HypoPG 등 포함) — https://hub.docker.com/r/postgresai/extended-postgres
- Model Context Protocol 공식 문서 — https://modelcontextprotocol.io/
- Claude Desktop 로컬 MCP 서버 연결 안내(한국어) — https://support.claude.com/ko/articles/10949351
- PgHero (Postgres MCP Pro의 헬스 체크 항목 출처) — https://github.com/ankane/pghero

---

## 부록. 한 장 요약 — 전체 순서

```mermaid
flowchart TB
    S1["1. Docker Desktop + Claude Desktop 설치"]
    S2["2. 작업 폴더 · Dockerfile · docker-compose.yml 작성"]
    S3["3. SQL 3개 작성<br/>01-schema / 02-data / 03-warmup"]
    S4["4. docker compose up -d --build<br/>→ 데이터 자동 삽입"]
    S5["5. docker pull crystaldba/postgres-mcp"]
    S6["6. claude_desktop_config.json 작성"]
    S7["7. Claude Desktop 완전 종료 후 재시작"]
    S8["8. 도구 9개 확인"]
    S9["9. 실습 프롬프트로 분석"]
    S10["10. 아티팩트로 시각화"]

    S1 --> S2 --> S3 --> S4 --> S5 --> S6 --> S7 --> S8 --> S9 --> S10

    style S1 fill:#E2F1F2,stroke:#0E7C86
    style S4 fill:#E7F4EE,stroke:#1A7355
    style S7 fill:#FDF3E3,stroke:#B9741B
    style S10 fill:#E7F4EE,stroke:#1A7355
```

| 단계 | 핵심 명령 또는 행동 | 성공 확인 방법 |
|---|---|---|
| 1 | Docker Desktop 실행 | `docker --version` 출력 |
| 2~3 | 파일 5개 작성 | 폴더에 Dockerfile, yml, sql 3개 |
| 4 | `docker compose up -d --build` | `movies 1010` 조회됨 |
| 5 | `docker pull crystaldba/postgres-mcp` | `docker images`에 표시 |
| 6 | JSON 설정 작성 | 문법 오류 없음 |
| 7 | 완전 종료 후 재시작 | 도구 아이콘 표시 |
| 8 | "테이블 목록 보여줘" | 6개 테이블 응답 |
| 9 | 분석 프롬프트 입력 | 헬스체크·느린쿼리 결과 |
| 10 | 시각화 프롬프트 입력 | 아티팩트 패널 표시 |

---

작성일자: 2026-07-29
