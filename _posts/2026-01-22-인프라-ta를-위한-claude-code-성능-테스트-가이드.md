---
title: "인프라 TA를 위한 Claude Code 성능 테스트 가이드"
date: 2026-01-22 21:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  performance-test,  terminal-agent,  sub-agents,  Guide,  Claude.write]
---

## 터미널 네이티브 AI 에이전트 기반 성능 테스트 자동화

---

## Claude Code vs Google Antigravity: 무엇이 다른가?

### 핵심 차이점

| 특성 | Claude Code | Google Antigravity |
|------|-------------|-------------------|
| **실행 환경** | 순수 터미널 (CLI) | GUI IDE (VS Code 포크) |
| **주요 사용자** | 시니어 개발자, 인프라 엔지니어 | 전체 개발자 |
| **강점 영역** | 대규모 리팩토링, 인프라 자동화 | UI 개발, 시각적 작업 |
| **브라우저** | 외부 도구 (Puppeteer MCP) | 내장 Browser Agent |
| **병렬 처리** | Sub-agent 시스템 | Manager View (최대 5개) |
| **학습 곡선** | 높음 (터미널 친화적) | 낮음 (GUI 익숙) |
| **컨텍스트** | 200K 토큰 | 1M 토큰 (Gemini 3 Pro) |
| **가격** | $20/월 (Pro), $200/월 (Max) | 무료 (Preview 기간) |

### 언제 무엇을 사용할까?

**Claude Code를 선택해야 할 때:**
- 터미널 환경에서 작업하는 것이 편한 경우
- 대규모 코드베이스 리팩토링
- 인프라 자동화 스크립트 작성
- Git 워크플로우 자동화
- CLI 도구 위주 작업 (k6, Terraform, kubectl)
- 여러 독립적인 작업을 동시에 진행

**Antigravity를 선택해야 할 때:**
- UI 개발 및 시각적 테스트
- 웹 브라우저 자동화 필요
- GUI 기반 작업 선호
- 프론트엔드 개발자 팀

**성능 테스트 관점:**
```
Claude Code: 인프라 TA, DevOps 엔지니어
- 복잡한 테스트 스크립트 생성
- CI/CD 파이프라인 구축
- 모니터링 스택 자동화
- 분산 시스템 테스트

Antigravity: QA 테스터, 프론트엔드 개발자
- 브라우저 기반 E2E 테스트
- UI 성능 테스트
- 시각적 회귀 테스트
```

## Claude Code 핵심 개념

### 1. 터미널 네이티브 에이전트

Claude Code는 IDE가 아닌 **터미널에서 살아가는 AI**입니다.

```bash
# 설치
brew install anthropic/claude/claude-code  # macOS
# 또는
winget install Anthropic.Claude  # Windows

# 실행
cd /path/to/your/project
claude

# 명령 예시
> k6 스크립트를 생성해서 API 부하 테스트 해줘
> 실패한 테스트를 찾아서 고쳐줘
> 이 코드를 리팩토링하고 테스트도 업데이트해줘
> 커밋하고 PR 올려줘
```

### 2. Sub-agent 시스템

하나의 Lead Agent가 여러 Worker Agent를 생성하여 병렬 작업을 수행합니다.

```
Lead Agent (당신과 대화)
  ├── Worker Agent 1: k6 스크립트 작성
  ├── Worker Agent 2: Locust 스크립트 작성
  ├── Worker Agent 3: JMeter 설정 생성
  └── Worker Agent 4: 모니터링 대시보드 구성

각 Worker는 독립적으로 작동하며 
Lead Agent가 결과를 통합합니다.
```

**실제 사용:**
```bash
> 성능 테스트 환경을 구축해줘:
> 1. k6 스크립트 (API 테스트)
> 2. Locust 스크립트 (시나리오 테스트)
> 3. Prometheus + Grafana 설정
> 4. Docker Compose로 통합
> 
> 각 작업을 독립적인 에이전트로 처리해줘

# Claude Code가 4개의 서브에이전트 생성
# 병렬로 작업 수행 후 통합
```

### 3. SKILL.md: 프로젝트별 워크플로우

프로젝트의 `.claude/SKILL.md` 파일에 성능 테스트 표준을 정의합니다.

```markdown
# Performance Testing Standards

## Tools
- Primary: k6 (API load testing)
- Secondary: Locust (scenario testing)
- Monitoring: Prometheus + Grafana

## Test Scenarios
All performance tests must include:
1. Ramp-up period (2 minutes)
2. Sustained load (5 minutes)
3. Ramp-down period (2 minutes)

## Thresholds
- P95 response time: < 500ms
- Error rate: < 1%
- Throughput: > 1000 RPS

## Workflow
When creating k6 scripts:
1. Use modular structure (config, scenarios, checks)
2. Export HTML reports via k6-reporter
3. Send metrics to Prometheus
4. Include realistic think time (1-3s)

## Naming Convention
- Test files: `load-test-{feature}.js`
- Reports: `report-{feature}-{date}.html`
```

Claude Code는 이 파일을 자동으로 읽고 모든 작업에 적용합니다.

### 4. Custom Commands

반복 작업을 슬래시 명령어로 저장합니다.

**.claude/commands/perf-test.md:**
```markdown
# /perf-test

Please run a comprehensive performance test:

1. Analyze the API endpoints in the current project
2. Generate k6 test scripts for each critical endpoint
3. Run the tests with these parameters:
   - Users: 100 → 500 (ramp-up 5min)
   - Duration: 15 minutes
4. Collect metrics to Prometheus
5. Generate HTML report
6. Compare against baseline (baseline.json)
7. If performance degraded >10%, create GitHub issue

Parameters: $ARGUMENTS (e.g., "staging" or "production")
```

**사용:**
```bash
> /perf-test staging
# 전체 워크플로우 자동 실행
```

### 5. MCP (Model Context Protocol)

외부 도구와의 통합을 위한 프로토콜입니다.

**예: Puppeteer MCP (브라우저 자동화)**
```bash
# MCP 서버 설치
npm install -g @modelcontextprotocol/server-puppeteer

# Claude Code 설정에 추가
> /mcp add puppeteer

# 사용
> Puppeteer로 로그인 페이지를 열고 성능 메트릭 수집해줘
```

**유용한 MCP 서버:**
- `@modelcontextprotocol/server-puppeteer`: 브라우저 제어
- `@modelcontextprotocol/server-postgres`: 데이터베이스 쿼리
- `@modelcontextprotocol/server-prometheus`: 메트릭 조회

## 실전 1: k6 API 부하 테스트 완전 자동화

### 시나리오

인증 API의 성능을 검증하고 CI/CD에 통합

### Claude Code 실행

```bash
$ cd my-api-project
$ claude

> 다음 성능 테스트를 수행해줘:
> 
> 목표: POST /api/auth/login의 성능 검증
> 
> 요구사항:
> 1. k6 스크립트 작성
>    - 시나리오: 0 → 100 → 300 → 500 사용자 (단계별 2분)
>    - 각 사용자 플로우:
>      a) POST /api/auth/login (이메일/비밀번호)
>      b) 응답에서 JWT 토큰 추출
>      c) GET /api/user/profile (토큰 인증)
>      d) 1-3초 랜덤 대기
> 
> 2. 검증 기준
>    - 로그인 P95 < 500ms
>    - 프로필 조회 P95 < 200ms
>    - 전체 에러율 < 1%
>    - RPS > 1000
> 
> 3. 출력
>    - JSON 메트릭 (results.json)
>    - HTML 리포트
>    - Prometheus 메트릭 전송
> 
> 4. 베이스라인 비교
>    - baseline.json과 비교
>    - 10% 이상 성능 저하 시 경고
> 
> 5. 실행 후 자동으로 다음 수행:
>    - 결과 분석
>    - 문제 발견 시 GitHub 이슈 생성
>    - Slack #perf-tests 채널에 결과 포스팅
```

### Claude Code의 작동

**Planning Phase:**
```
⏺ Analyzing requirements...
⏺ Creating execution plan with 4 sub-agents:
  1. Script Generator: k6 test script
  2. Infrastructure: Prometheus setup
  3. Executor: Run test and collect metrics
  4. Analyzer: Compare with baseline and report

⏺ Starting parallel execution...
```

**Sub-agent 1: Script Generator**
```javascript
// load-test-auth.js (자동 생성)
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Counter, Trend } from 'k6/metrics';
import { htmlReport } from 'https://raw.githubusercontent.com/benc-uk/k6-reporter/main/dist/bundle.js';

// Custom metrics
const loginDuration = new Trend('login_duration');
const profileDuration = new Trend('profile_duration');
const authErrors = new Counter('auth_errors');

export const options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '2m', target: 100 },
    { duration: '2m', target: 300 },
    { duration: '2m', target: 300 },
    { duration: '2m', target: 500 },
    { duration: '2m', target: 500 },
    { duration: '2m', target: 0 },
  ],
  thresholds: {
    'http_req_duration{endpoint:login}': ['p(95)<500'],
    'http_req_duration{endpoint:profile}': ['p(95)<200'],
    'http_req_failed': ['rate<0.01'],
  },
  ext: {
    loadimpact: {
      projectID: 3512345,
      name: 'Auth API Load Test'
    }
  }
};

export default function () {
  // 1. Login
  const loginRes = http.post(
    'https://api.example.com/auth/login',
    JSON.stringify({
      email: 'test@example.com',
      password: 'password123'
    }),
    {
      headers: { 'Content-Type': 'application/json' },
      tags: { endpoint: 'login' }
    }
  );

  const loginSuccess = check(loginRes, {
    'login status 200': (r) => r.status === 200,
    'login has token': (r) => r.json('token') !== undefined,
  });

  if (!loginSuccess) {
    authErrors.add(1);
    return;
  }

  loginDuration.add(loginRes.timings.duration);
  const token = loginRes.json('token');

  // 2. Get Profile
  const profileRes = http.get(
    'https://api.example.com/user/profile',
    {
      headers: { 'Authorization': `Bearer ${token}` },
      tags: { endpoint: 'profile' }
    }
  );

  check(profileRes, {
    'profile status 200': (r) => r.status === 200,
    'profile has data': (r) => r.json('id') !== undefined,
  });

  profileDuration.add(profileRes.timings.duration);

  // 3. Think time
  sleep(Math.random() * 2 + 1); // 1-3초
}

export function handleSummary(data) {
  return {
    'results.json': JSON.stringify(data),
    'summary.html': htmlReport(data),
    'stdout': textSummary(data, { indent: ' ', enableColors: true }),
  };
}
```

**Sub-agent 2: Infrastructure Setup**
```bash
⏺ Setting up Prometheus for k6 metrics...
⏺ Creating prometheus.yml configuration...
⏺ Starting Prometheus container...
```

**Sub-agent 3: Test Execution**
```bash
⏺ Running k6 test with Prometheus output...
⏺ Bash(k6 run load-test-auth.js \
  --out experimental-prometheus-rw=http://localhost:9090/api/v1/write)

          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: load-test-auth.js
     output: Prometheus (http://localhost:9090)

  scenarios: (100.00%) 1 scenario, 500 max VUs, 16m30s max duration

...

⏺ Test completed successfully
⏺ Collected 96,000 requests
⏺ Average RPS: 1,142
⏺ P95 login: 487ms ✅
⏺ P95 profile: 178ms ✅
⏺ Error rate: 0.28% ✅
```

**Sub-agent 4: Analysis & Reporting**
```python
# compare_baseline.py (자동 생성 및 실행)
import json
import sys

def compare_results(current_file, baseline_file):
    with open(current_file) as f:
        current = json.load(f)
    
    with open(baseline_file) as f:
        baseline = json.load(f)
    
    current_p95 = current['metrics']['http_req_duration']['values']['p(95)']
    baseline_p95 = baseline['metrics']['http_req_duration']['values']['p(95)']
    
    degradation = ((current_p95 - baseline_p95) / baseline_p95) * 100
    
    print(f"Current P95: {current_p95:.2f}ms")
    print(f"Baseline P95: {baseline_p95:.2f}ms")
    print(f"Degradation: {degradation:.1f}%")
    
    if degradation > 10:
        print("⚠️ PERFORMANCE DEGRADATION DETECTED")
        create_github_issue(current_p95, baseline_p95, degradation)
    
    post_to_slack(current_p95, degradation)

⏺ Bash(python compare_baseline.py results.json baseline.json)

Current P95: 487ms
Baseline P95: 445ms
Degradation: 9.4%

✅ Performance within acceptable range
⏺ Posting results to Slack...
⏺ Posted to #perf-tests
```

**Final Output:**
```
✅ Performance test completed successfully

Summary:
- Total requests: 96,000
- RPS: 1,142 (target: >1000) ✅
- P95 login: 487ms (target: <500ms) ✅
- P95 profile: 178ms (target: <200ms) ✅
- Error rate: 0.28% (target: <1%) ✅

Compared to baseline:
- P95 degradation: 9.4% (acceptable)

Generated files:
- load-test-auth.js (test script)
- results.json (raw metrics)
- summary.html (visual report)
- comparison-report.txt

Next steps:
- Results posted to Slack #perf-tests
- Metrics available in Grafana
- Baseline updated for next comparison

Would you like me to commit these files?
```

### 핵심 장점

1. **완전 자동화**: 한 번의 자연어 요청으로 전체 워크플로우 실행
2. **병렬 처리**: 4개의 서브에이전트가 동시 작업
3. **자체 검증**: 실행 후 자동 분석 및 의사결정
4. **통합**: GitHub, Slack, Prometheus 자동 연동
5. **재현 가능**: 생성된 스크립트는 독립적으로 실행 가능

## 실전 2: TDD 기반 성능 테스트

### Test-Driven Development for Performance

Claude Code는 TDD를 강력하게 지원합니다.

```bash
> 성능 테스트를 TDD 방식으로 작성해줘:
> 
> 1. 먼저 테스트를 작성 (실패하는 테스트)
>    - API /api/products/search의 P95 < 300ms
>    - 동시 사용자 1000명
> 
> 2. 테스트 실행해서 실패 확인
> 
> 3. 코드 최적화 제안
>    - 데이터베이스 인덱스
>    - 캐싱 전략
>    - 쿼리 최적화
> 
> 4. 최적화 적용 후 테스트 재실행
> 
> 5. 통과할 때까지 반복
```

**Claude Code 실행:**

**Phase 1: Write Failing Test**
```javascript
⏺ Creating performance test (expected to fail)...

// test-search-performance.js
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 1000,
  duration: '5m',
  thresholds: {
    'http_req_duration': ['p(95)<300'], // 목표: 300ms
  },
};

export default function () {
  const res = http.get('https://api.example.com/products/search?q=laptop');
  check(res, {
    'status is 200': (r) => r.status === 200,
  });
}

⏺ Running initial test...
⏺ Bash(k6 run test-search-performance.js)

✗ P95: 1,247ms (threshold: <300ms) ❌
❌ Test FAILED as expected
```

**Phase 2: Analyze & Optimize**
```
⏺ Analyzing API performance...
⏺ Reading database query logs...
⏺ Identified bottlenecks:

1. No index on products.name column (full table scan)
2. No caching for search results
3. Fetching unnecessary columns (SELECT *)

⏺ Proposing optimizations...
```

**Phase 3: Apply Optimizations**
```sql
-- migrations/001_add_search_index.sql (자동 생성)
⏺ Creating database migration...

CREATE INDEX idx_products_name_gin ON products 
USING gin(to_tsvector('english', name));

CREATE INDEX idx_products_category ON products(category_id);

ANALYZE products;
```

```javascript
// Caching layer (자동 생성)
⏺ Adding Redis caching...

const redis = require('redis');
const client = redis.createClient();

async function searchProducts(query) {
  const cacheKey = `search:${query}`;
  
  // Check cache
  const cached = await client.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Query database
  const results = await db.query(
    'SELECT id, name, price FROM products WHERE name ILIKE $1',
    [`%${query}%`]
  );
  
  // Cache for 5 minutes
  await client.setex(cacheKey, 300, JSON.stringify(results));
  
  return results;
}

⏺ Applying migrations...
⏺ Deploying code changes...
```

**Phase 4: Re-run Test**
```bash
⏺ Waiting for deployment (30s)...
⏺ Re-running performance test...

✓ P95: 278ms (threshold: <300ms) ✅
✅ Test PASSED

Improvement: 1,247ms → 278ms (77.7% faster)
```

**Phase 5: Document & Commit**
```markdown
⏺ Creating optimization report...

# Performance Optimization: Search API

## Problem
Search API was too slow (P95: 1,247ms)

## Root Causes
1. No database indexes
2. No caching
3. Inefficient queries

## Solutions Applied
1. Added GIN index on products.name
2. Implemented Redis caching (5min TTL)
3. Optimized SELECT query

## Results
- P95: 1,247ms → 278ms (77.7% improvement)
- Throughput: 87 RPS → 412 RPS (374% improvement)
- CPU usage: 82% → 34% (58% reduction)

⏺ Committing changes...
⏺ Creating pull request...
⏺ PR #123 created: "Optimize search API performance"
```

## 실전 3: 분산 시스템 성능 테스트

### Swarming: 여러 머신에서 동시 실행

```bash
> 분산 부하 테스트를 설정해줘:
> 
> 목표: 10,000 동시 사용자 시뮬레이션
> 
> 아키텍처:
> - 1 마스터 노드
> - 5 워커 노드 (각 2,000 사용자)
> 
> 구현:
> 1. Locust 마스터/워커 설정
> 2. Docker Compose로 오케스트레이션
> 3. 네트워크 설정
> 4. 결과 집계
```

**Claude Code 출력:**

**docker-compose.yml (자동 생성):**
```yaml
⏺ Creating distributed load test infrastructure...

version: '3.8'

services:
  locust-master:
    image: locustio/locust:latest
    ports:
      - "8089:8089"
      - "5557:5557"
    volumes:
      - ./locustfile.py:/mnt/locust/locustfile.py
    command: -f /mnt/locust/locustfile.py --master --expect-workers=5
    networks:
      - locust-net

  locust-worker-1:
    image: locustio/locust:latest
    volumes:
      - ./locustfile.py:/mnt/locust/locustfile.py
    command: -f /mnt/locust/locustfile.py --worker --master-host=locust-master
    networks:
      - locust-net

  locust-worker-2:
    image: locustio/locust:latest
    volumes:
      - ./locustfile.py:/mnt/locust/locustfile.py
    command: -f /mnt/locust/locustfile.py --worker --master-host=locust-master
    networks:
      - locust-net

  # ... workers 3-5 similar ...

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - locust-net

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - locust-net

networks:
  locust-net:
    driver: bridge
```

**locustfile.py (자동 생성):**
```python
⏺ Creating distributed test scenario...

from locust import HttpUser, task, between, events
import time
import logging

# Prometheus 메트릭 수집
from prometheus_client import Counter, Histogram, start_http_server

request_counter = Counter('locust_requests_total', 'Total requests')
response_time_histogram = Histogram('locust_response_time_seconds', 'Response time')

def on_request(request_type, name, response_time, response_length, exception, **kwargs):
    request_counter.inc()
    response_time_histogram.observe(response_time / 1000)

class EcommerceUser(HttpUser):
    wait_time = between(1, 3)
    host = "https://api.example.com"
    
    def on_start(self):
        # 로그인
        response = self.client.post("/auth/login", json={
            "email": f"user{self.user_id}@example.com",
            "password": "password123"
        })
        self.token = response.json().get("token")
    
    @task(10)
    def browse_products(self):
        self.client.get("/products", headers={
            "Authorization": f"Bearer {self.token}"
        })
    
    @task(5)
    def search_products(self):
        queries = ["laptop", "phone", "tablet"]
        query = random.choice(queries)
        self.client.get(f"/products/search?q={query}", headers={
            "Authorization": f"Bearer {self.token}"
        })
    
    @task(3)
    def add_to_cart(self):
        product_id = random.randint(1, 1000)
        self.client.post("/cart", json={
            "product_id": product_id,
            "quantity": 1
        }, headers={
            "Authorization": f"Bearer {self.token}"
        })
    
    @task(1)
    def checkout(self):
        self.client.post("/orders", json={
            "payment_method": "credit_card"
        }, headers={
            "Authorization": f"Bearer {self.token}"
        })

# Prometheus 메트릭 서버 시작
start_http_server(8000)
```

**실행:**
```bash
⏺ Starting distributed load test...
⏺ Bash(docker-compose up -d)

[+] Running 8/8
 ✔ Network locust-net Created
 ✔ Container locust-master Started
 ✔ Container locust-worker-1 Started
 ✔ Container locust-worker-2 Started
 ✔ Container locust-worker-3 Started
 ✔ Container locust-worker-4 Started
 ✔ Container locust-worker-5 Started
 ✔ Container prometheus Started
 ✔ Container grafana Started

⏺ Waiting for workers to connect...
⏺ All 5 workers connected
⏺ Starting test: 10,000 users, spawn rate 100/sec

⏺ Test running... 
  - Current users: 10,000
  - RPS: 8,742
  - P50: 87ms
  - P95: 234ms
  - Failures: 0.12%

⏺ Test completed after 15 minutes
⏺ Generating report...

Results:
- Total requests: 7,866,600
- Average RPS: 8,742
- P95 response time: 234ms
- Error rate: 0.12%
- Peak CPU (app servers): 67%
- Peak memory: 8.2GB

⏺ Shutting down infrastructure...
⏺ Bash(docker-compose down)
```

## 실전 4: CI/CD 통합

### GitHub Actions with Claude Code

```bash
> GitHub Actions workflow를 만들어줘:
> 
> 트리거: PR to main
> 
> 작업:
> 1. 코드 체크아웃
> 2. 스테이징 배포
> 3. k6 성능 테스트
> 4. 베이스라인 비교
> 5. 임계값 초과 시 PR 차단
> 6. 결과를 PR 코멘트로 게시
> 7. Slack 알림
```

**.github/workflows/performance-test.yml (자동 생성):**
```yaml
name: Performance Regression Test

on:
  pull_request:
    branches: [main]

jobs:
  performance-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup k6
        run: |
          curl -L https://github.com/grafana/k6/releases/download/v0.49.0/k6-v0.49.0-linux-amd64.tar.gz | tar xvz
          sudo mv k6-v0.49.0-linux-amd64/k6 /usr/local/bin/
      
      - name: Deploy to Staging
        run: ./scripts/deploy-staging.sh
        env:
          STAGING_TOKEN: ${{ secrets.STAGING_TOKEN }}
      
      - name: Wait for deployment
        run: sleep 60
      
      - name: Run performance test
        run: |
          k6 run tests/performance/load-test.js \
            --out json=results.json \
            --env STAGING_URL=${{ secrets.STAGING_URL }}
      
      - name: Compare with baseline
        id: compare
        run: |
          python scripts/compare-baseline.py results.json baseline.json
          echo "degradation=$(cat degradation.txt)" >> $GITHUB_OUTPUT
      
      - name: Check thresholds
        run: |
          if [ $(cat degradation.txt) -gt 10 ]; then
            echo "❌ Performance degradation >10%"
            exit 1
          fi
      
      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('results.json'));
            const degradation = fs.readFileSync('degradation.txt', 'utf8');
            
            const p95 = results.metrics.http_req_duration.values['p(95)'];
            const rps = results.metrics.http_reqs.values.rate;
            const errors = results.metrics.http_req_failed.values.rate * 100;
            
            const body = `## 🚀 Performance Test Results
            
**Metrics:**
- **P95 Response Time**: ${p95.toFixed(2)}ms
- **Throughput**: ${rps.toFixed(0)} RPS
- **Error Rate**: ${errors.toFixed(2)}%

**Compared to Baseline:**
- ${degradation > 0 ? '⚠️' : '✅'} Performance change: ${degradation}%

${degradation > 10 ? '**❌ FAILED: Performance degraded >10%**' : '**✅ PASSED**'}
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
      
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Performance test completed for PR #${{ github.event.pull_request.number }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "${{ steps.compare.outputs.degradation > 10 && '❌ *FAILED*' || '✅ *PASSED*' }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## Claude Code의 독특한 강점

### 1. 멀티-세션 작업

여러 터미널에서 동시에 작업하고 서로 통신할 수 있습니다.

```bash
# Terminal 1: k6 테스트 작성
$ claude
> k6 스크립트 작성해줘

# Terminal 2: Locust 테스트 작성
$ claude
> Locust 스크립트 작성해줘. 
> Terminal 1의 k6 스크립트를 참고해서
> 동일한 시나리오로 만들어줘

# Claude Code가 두 세션 간 컨텍스트 공유
```

### 2. Session Teleportation

작업을 다른 머신으로 이동할 수 있습니다.

```bash
# 로컬 머신
$ claude
> 복잡한 테스트 스크립트 생성 시작...

# 회의 참석으로 중단 필요

> /teleport create
Session URL: https://claude.ai/teleport/abc123

# 다른 머신 (또는 동료)
$ claude /teleport join abc123
> 작업 계속 진행...
```

### 3. Git 워크플로우 완전 자동화

```bash
> 성능 최적화 작업을 처음부터 끝까지 해줘:
> 
> 1. feature/perf-optimization 브랜치 생성
> 2. 현재 성능 측정 (베이스라인)
> 3. 병목 지점 식별
> 4. 코드 최적화
> 5. 테스트로 개선 확인
> 6. 커밋
> 7. PR 생성
> 8. 리뷰어 할당

⏺ Bash(git checkout -b feature/perf-optimization)
⏺ Running baseline performance test...
⏺ Analyzing code for bottlenecks...
⏺ Applying optimizations...
⏺ Running validation tests...
⏺ Bash(git add -A)
⏺ Bash(git commit -m "Optimize API performance by 65%")
⏺ Bash(git push origin feature/perf-optimization)
⏺ Bash(gh pr create --title "Performance optimization" --reviewer @team/backend)
⏺ PR #124 created: https://github.com/org/repo/pull/124
```

## 베스트 프랙티스

### 1. SKILL.md 활용

프로젝트 루트에 `.claude/SKILL.md` 생성:

```markdown
# Performance Testing Standards for [Project Name]

## Tools & Versions
- k6: v0.49.0
- Locust: v2.19.0
- Prometheus: v2.45.0

## Environment URLs
- Development: https://dev-api.example.com
- Staging: https://staging-api.example.com
- Production: https://api.example.com

## Test Credentials
- Use environment variables: TEST_EMAIL, TEST_PASSWORD
- Never hardcode credentials

## Performance Targets
| Endpoint | P95 (ms) | P99 (ms) | RPS | Error % |
|----------|----------|----------|-----|---------|
| /auth/login | 500 | 1000 | 1000 | <1 |
| /products | 200 | 400 | 5000 | <0.5 |
| /orders | 800 | 1500 | 500 | <2 |

## Workflow
1. Always create a test plan before writing scripts
2. Start with 10 users, gradually increase to target
3. Run for at least 10 minutes (5min ramp-up, 5min sustained)
4. Collect metrics to Prometheus
5. Compare with baseline
6. Generate HTML report
7. If regression detected, create GitHub issue automatically

## Naming Conventions
- Scripts: `perf-{feature}-{type}.js`
- Reports: `report-{feature}-{date}.html`
- Baselines: `baseline-{feature}.json`
```

### 2. Custom Commands 라이브러리

**.claude/commands/quick-perf.md:**
```markdown
# /quick-perf

Run a quick performance sanity check on $ARGUMENTS endpoint:
1. 50 users for 2 minutes
2. Check P95 < 500ms
3. Report pass/fail
```

**.claude/commands/full-perf.md:**
```markdown
# /full-perf

Run comprehensive performance test suite:
1. All critical endpoints
2. Realistic user scenarios
3. Full ramp-up: 0 → 100 → 500 → 1000
4. 30-minute sustained load
5. Compare all baselines
6. Generate executive summary
7. Post to Slack
```

### 3. Sub-agent 전략

```bash
> 성능 테스트 환경을 구축해줘.
> 다음을 독립적인 서브에이전트로 처리:
> 
> Agent 1: k6 스크립트 작성 (API 테스트)
> Agent 2: Locust 스크립트 작성 (시나리오 테스트)
> Agent 3: Prometheus + Grafana 설정
> Agent 4: Docker Compose 통합
> Agent 5: CI/CD workflow 작성
> 
> 각 에이전트는 독립적으로 작업하고
> 최종적으로 통합해줘
```

### 4. 반복 작업 자동화

```bash
> 매일 오전 9시에 자동으로 실행될
> 성능 회귀 테스트를 설정해줘:
> 
> 1. cron job 또는 GitHub Actions 스케줄
> 2. 전체 API 엔드포인트 테스트
> 3. 베이스라인과 비교
> 4. 문제 발견 시 Slack 알림 + GitHub Issue
> 5. 주간 리포트 생성 (금요일)
```

## Claude Code 제약사항

### 1. 비용

| 플랜 | 가격 | 모델 | 용도 |
|------|------|------|------|
| Free | $0 | Sonnet 3.5 | 제한적 테스트 |
| Pro | $20/월 | Sonnet 4.5 | 개인 개발자 |
| Max | $200/월 | Opus 4.5 | 프로페셔널 |

**성능 테스트 관점:**
- Pro 플랜으로 대부분의 작업 가능
- 대규모 자동화는 Max 플랜 권장
- API 사용도 고려 가능 (별도 요금)

### 2. 컨텍스트 제한

- 200K 토큰 (vs Antigravity의 1M)
- 매우 큰 코드베이스는 부분 처리 필요

**해결책:**
```bash
> 이 프로젝트는 너무 커서 한 번에 처리할 수 없어.
> /api 디렉토리만 집중해서 성능 테스트 작성해줘
```

### 3. GUI 작업 제한

- 터미널 기반이므로 시각적 확인 어려움
- 웹 UI 테스트는 Puppeteer MCP 필요

**대안:**
- Antigravity는 GUI 작업에 더 적합
- 두 도구를 상황에 맞게 병용

## Antigravity vs Claude Code: 성능 테스트 비교

| 작업 | Claude Code | Antigravity |
|------|-------------|-------------|
| k6 스크립트 생성 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 브라우저 E2E 테스트 | ⭐⭐⭐ (MCP 필요) | ⭐⭐⭐⭐⭐ |
| 인프라 자동화 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| CI/CD 통합 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 대규모 리팩토링 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 터미널 워크플로우 | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 학습 곡선 | 높음 | 낮음 |
| 가격 | $20-200/월 | 무료 (Preview) |

**추천:**
- **인프라 TA, DevOps**: Claude Code
- **QA 테스터, 프론트엔드**: Antigravity
- **최선**: 두 도구 모두 활용

## 결론

Claude Code는 **터미널에서 살아가는 시니어 엔지니어**입니다.

**핵심 가치:**
1. **완전 자동화**: 자연어 → 실행 → 검증 → 배포
2. **병렬 처리**: 서브에이전트로 복잡한 작업 분해
3. **Git 통합**: 브랜치, 커밋, PR 자동화
4. **재현 가능**: 생성된 코드는 독립 실행 가능
5. **학습하는 도구**: SKILL.md로 프로젝트 특화

**성능 테스트 관점:**
- 스크립트 작성 시간 95% 단축
- 인프라 설정 완전 자동화
- CI/CD 통합 간소화
- 회귀 테스트 자동 감지

**시작하기:**
```bash
$ brew install anthropic/claude/claude-code
$ cd my-project
$ claude
> 이 프로젝트의 성능 테스트를 만들어줘
```

성능 테스트는 이제 "스크립트를 작성하는 것"이 아니라 "성능 목표를 정의하고 검증하는 것"입니다. Claude Code는 나머지를 처리합니다.

---

**문서 작성 일자: 2026-01-22**
