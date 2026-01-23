---
title: "테스터를 위한 Claude Code 통합 테스트 종합 가이드"
date: 2026-01-22 21:10:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  integration-test,  Guide,  Claude.write]
---

## 서브 에이전트 오케스트레이션으로 테스트 자동화 혁신하기

---

## 핵심 질문: 왜 Claude Code인가?

### 테스터에게 Claude Code가 중요한 이유

**전통적 테스트 작성:**
```
1. 요구사항 분석 (30분)
2. 테스트 케이스 설계 (1시간)
3. 테스트 코드 작성 (2시간)
4. Mock/Fixture 준비 (1시간)
5. 실행 및 디버깅 (1시간)
총 5.5시간
```

**Claude Code 활용:**
```
$ claude
> 로그인 → 상품 검색 → 장바구니 → 결제 플로우에 대한
> 통합 테스트를 작성해줘.
> Playwright로 E2E, Jest로 API 통합, 그리고 DB 검증 포함.

→ 15분 내 완료
```

**핵심 가치:**
1. **서브 에이전트**: 여러 전문가가 동시 작업 (API 테스터 + E2E 테스터 + DB 검증자)
2. **자동 Mock 생성**: 복잡한 의존성 자동 Mocking
3. **자가 치유**: 테스트 실패 시 자동 수정 시도
4. **코드 커버리지**: 자동으로 80%+ 달성
5. **CI/CD 통합**: GitHub Actions 자동 생성

## 서브 에이전트 핵심 개념

### 서브 에이전트란?

Claude Code에서 **서브 에이전트는 특정 작업을 독립적으로 수행하는 전문 AI**입니다.

```
Main Agent (당신과 대화하는 Claude)
  ├── Explore Agent (read-only, 빠른 코드 탐색)
  ├── Plan Agent (계획 수립 전문)
  ├── General Purpose Agent (복잡한 다단계 작업)
  └── Custom Agents (사용자 정의)
      ├── test-automator (통합 테스트 작성)
      ├── e2e-specialist (E2E 테스트 전문)
      ├── api-tester (API 테스트)
      └── db-validator (데이터베이스 검증)
```

### 서브 에이전트의 특징

**1. 독립적 컨텍스트:**
```
각 서브 에이전트는 자신만의 메모리를 가짐
→ 메인 대화 컨텍스트를 오염시키지 않음
→ 병렬 실행 가능
```

**2. 도구 제한:**
```markdown
# test-automator 에이전트
tools: Read, Write, Edit, Bash, Grep
→ 테스트 작성에 필요한 최소 권한만

# code-reviewer 에이전트
tools: Read, Grep, Glob
→ Read-only, 코드 수정 불가
```

**3. 전문성:**
```
각 에이전트는 특정 도메인 전문 지식 보유
- Playwright Agent: E2E 테스트 패턴
- API Test Agent: REST API 테스트 best practices
- DB Test Agent: 트랜잭션, 격리 레벨
```

### 서브 에이전트 생성 방법

#### 방법 1: 자동 생성 (권장)

```bash
$ claude
> /agents

# 메뉴에서 선택:
# 1. Create new agent
# 2. Project-level (.claude/agents/)
# 3. Generate with Claude

# 프롬프트 입력:
> 통합 테스트 전문 에이전트를 만들어줘.
> 
> 역할:
> - API 엔드포인트 통합 테스트
> - 데이터베이스 상태 검증
> - Mock 외부 서비스
> - 에러 시나리오 포함
> 
> 프레임워크: Jest, Supertest, TypeORM
```

#### 방법 2: 수동 생성

**.claude/agents/integration-test-specialist.md:**
~~~markdown
---
name: integration-test-specialist
description: |
  Use this agent when you need to write integration tests that:
  - Test API endpoints with real database
  - Verify business logic with multiple services
  - Include error scenarios and edge cases
  - Mock external dependencies
  Trigger when user mentions "integration test", "API test", 
  "service test", or needs database-backed testing.
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
model: sonnet
---

# Integration Test Specialist

You are an expert integration test engineer specializing in API testing.

## Testing Philosophy

1. **Realistic Environment**
   - Use test database (not mocks for DB)
   - Mock only external services (payment, email)
   - Real HTTP calls through the app

2. **Coverage Focus**
   - Happy path
   - Error scenarios (404, 401, 500)
   - Edge cases (empty data, large data)
   - Concurrent requests

3. **Test Structure**
   ```javascript
   describe('Feature', () => {
     beforeAll(() => {
       // Setup: DB, test data
     });
     
     it('should handle happy path', async () => {
       // Arrange, Act, Assert
     });
     
     it('should handle error: unauthorized', async () => {
       // Test 401
     });
     
     afterAll(() => {
       // Cleanup
     });
   });
   ```

4. **Database Management**
   - Use transactions for isolation
   - Rollback after each test
   - Seed minimal required data

5. **Assertion Best Practices**
   - Verify status code
   - Verify response schema
   - Verify database state
   - Verify side effects (logs, events)

## When You're Done

Report back with:
1. Test file path
2. Number of test cases created
3. Coverage (happy path + error scenarios)
4. Any setup requirements (env vars, seeds)
~~~

## 실전 1: E-commerce 통합 테스트 (서브 에이전트 오케스트레이션)

### 시나리오

사용자 인증 → 상품 검색 → 장바구니 → 주문 생성 전체 플로우 통합 테스트

### Claude Code 실행

```bash
$ cd ecommerce-api
$ claude

> 다음 통합 테스트를 작성해줘:
> 
> **시나리오: 전체 주문 플로우**
> 
> 1. API 통합 테스트 (Jest + Supertest)
>    - POST /auth/login → JWT 발급
>    - GET /products?category=electronics
>    - POST /cart/items (상품 추가)
>    - POST /orders (주문 생성)
>    - GET /orders/:id (주문 확인)
> 
> 2. E2E 테스트 (Playwright)
>    - 브라우저에서 동일 플로우
>    - 실제 UI 인터랙션
> 
> 3. 데이터베이스 검증
>    - 각 단계마다 DB 상태 확인
>    - 트랜잭션 무결성
> 
> 4. 에러 시나리오
>    - 재고 부족
>    - 결제 실패
>    - 동시성 문제
> 
> 서브 에이전트를 활용해서:
> - API 테스트
> - E2E 테스트
> - DB 검증
> 
> 병렬로 작성하고, 마지막에 통합해줘.
```

### Claude Code의 작동 과정

**Phase 1: Planning**
```
⏺ Analyzing request...
⏺ Creating execution plan with 3 parallel sub-agents:
  
  1. api-test-specialist
     - API integration tests (Jest + Supertest)
     - Mock external services (payment gateway)
     - Database setup/teardown
  
  2. e2e-test-specialist
     - Playwright E2E tests
     - Browser automation
     - Visual assertions
  
  3. db-validator
     - Database state verification
     - Transaction isolation tests
     - Concurrent access scenarios

⏺ Starting parallel execution...
```

**Sub-agent 1: API Test Specialist**

```javascript
⏺ Creating tests/integration/order-flow.test.ts...

import request from 'supertest';
import { app } from '../../src/app';
import { setupTestDB, teardownTestDB } from '../helpers/db';
import { createTestUser, createTestProduct } from '../fixtures';

describe('Order Flow Integration', () => {
  let testDB;
  let authToken;
  let testUser;
  let testProduct;

  beforeAll(async () => {
    testDB = await setupTestDB();
    testUser = await createTestUser({
      email: 'test@example.com',
      password: 'password123'
    });
    testProduct = await createTestProduct({
      name: 'Test Laptop',
      price: 1200,
      stock: 10
    });
  });

  afterAll(async () => {
    await teardownTestDB(testDB);
  });

  describe('Happy Path', () => {
    it('should complete full order flow', async () => {
      // Step 1: Login
      const loginRes = await request(app)
        .post('/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        })
        .expect(200);

      expect(loginRes.body).toHaveProperty('token');
      authToken = loginRes.body.token;

      // Step 2: Search products
      const searchRes = await request(app)
        .get('/products?category=electronics')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(searchRes.body.products).toBeInstanceOf(Array);
      expect(searchRes.body.products.length).toBeGreaterThan(0);

      // Step 3: Add to cart
      const cartRes = await request(app)
        .post('/cart/items')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          productId: testProduct.id,
          quantity: 2
        })
        .expect(201);

      expect(cartRes.body.cart.items).toHaveLength(1);
      expect(cartRes.body.cart.total).toBe(2400); // 1200 * 2

      // Step 4: Create order
      const orderRes = await request(app)
        .post('/orders')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          paymentMethod: 'credit_card',
          shippingAddress: '123 Test St'
        })
        .expect(201);

      expect(orderRes.body.order).toHaveProperty('id');
      expect(orderRes.body.order.status).toBe('pending');
      expect(orderRes.body.order.total).toBe(2400);

      const orderId = orderRes.body.order.id;

      // Step 5: Verify order
      const verifyRes = await request(app)
        .get(`/orders/${orderId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(verifyRes.body.order.id).toBe(orderId);
      expect(verifyRes.body.order.items).toHaveLength(1);

      // Step 6: Verify database state
      const dbOrder = await testDB.query(
        'SELECT * FROM orders WHERE id = $1',
        [orderId]
      );
      expect(dbOrder.rows[0].user_id).toBe(testUser.id);
      expect(dbOrder.rows[0].total).toBe('2400.00');

      // Step 7: Verify stock updated
      const dbProduct = await testDB.query(
        'SELECT stock FROM products WHERE id = $1',
        [testProduct.id]
      );
      expect(dbProduct.rows[0].stock).toBe(8); // 10 - 2
    });
  });

  describe('Error Scenarios', () => {
    it('should fail when stock insufficient', async () => {
      // Login first
      const loginRes = await request(app)
        .post('/auth/login')
        .send({ email: 'test@example.com', password: 'password123' })
        .expect(200);

      authToken = loginRes.body.token;

      // Try to order more than available
      await request(app)
        .post('/cart/items')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          productId: testProduct.id,
          quantity: 100 // Only 8 left
        })
        .expect(400);

      // Verify stock unchanged
      const dbProduct = await testDB.query(
        'SELECT stock FROM products WHERE id = $1',
        [testProduct.id]
      );
      expect(dbProduct.rows[0].stock).toBe(8);
    });

    it('should handle unauthorized access', async () => {
      await request(app)
        .get('/orders/123')
        .expect(401);
    });

    it('should handle payment failure', async () => {
      // Mock payment gateway failure
      jest.spyOn(paymentService, 'charge').mockRejectedValue(
        new Error('Payment declined')
      );

      const loginRes = await request(app)
        .post('/auth/login')
        .send({ email: 'test@example.com', password: 'password123' });

      authToken = loginRes.body.token;

      await request(app)
        .post('/orders')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          paymentMethod: 'credit_card',
          shippingAddress: '123 Test St'
        })
        .expect(402); // Payment Required

      // Verify no order created
      const orders = await testDB.query(
        'SELECT * FROM orders WHERE user_id = $1',
        [testUser.id]
      );
      expect(orders.rows.length).toBe(1); // Only the successful one
    });
  });

  describe('Concurrency', () => {
    it('should handle concurrent orders correctly', async () => {
      // Create product with limited stock
      const limitedProduct = await createTestProduct({
        name: 'Limited Edition',
        stock: 5
      });

      const loginRes = await request(app)
        .post('/auth/login')
        .send({ email: 'test@example.com', password: 'password123' });

      authToken = loginRes.body.token;

      // Simulate 10 concurrent requests for 1 item each
      const requests = Array(10).fill(null).map(() =>
        request(app)
          .post('/cart/items')
          .set('Authorization', `Bearer ${authToken}`)
          .send({
            productId: limitedProduct.id,
            quantity: 1
          })
      );

      const results = await Promise.allSettled(requests);

      // Only 5 should succeed
      const successful = results.filter(r => 
        r.status === 'fulfilled' && r.value.status === 201
      );
      expect(successful.length).toBe(5);

      // Remaining 5 should fail
      const failed = results.filter(r =>
        r.status === 'fulfilled' && r.value.status === 400
      );
      expect(failed.length).toBe(5);

      // Verify final stock is 0
      const finalStock = await testDB.query(
        'SELECT stock FROM products WHERE id = $1',
        [limitedProduct.id]
      );
      expect(finalStock.rows[0].stock).toBe(0);
    });
  });
});

⏺ API integration tests completed
⏺ 15 test cases created
⏺ Coverage: happy path + 3 error scenarios + concurrency
```

**Sub-agent 2: E2E Test Specialist**

```typescript
⏺ Creating tests/e2e/order-flow.spec.ts...

import { test, expect, Page } from '@playwright/test';

test.describe('Order Flow E2E', () => {
  let page: Page;

  test.beforeEach(async ({ page: p }) => {
    page = p;
    await page.goto('http://localhost:3000');
  });

  test('should complete order from login to confirmation', async () => {
    // Step 1: Login
    await page.click('text=Login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');

    // Wait for redirect to home
    await page.waitForURL('**/home');
    await expect(page.locator('text=Welcome')).toBeVisible();

    // Step 2: Search products
    await page.fill('[data-testid="search-input"]', 'laptop');
    await page.click('[data-testid="search-button"]');

    // Wait for results
    await page.waitForSelector('[data-testid="product-card"]');
    
    // Verify search results displayed
    const products = await page.locator('[data-testid="product-card"]').count();
    expect(products).toBeGreaterThan(0);

    // Step 3: Add to cart
    await page.click('[data-testid="product-card"]:first-child >> text=Add to Cart');
    
    // Verify cart badge updated
    await expect(page.locator('[data-testid="cart-badge"]')).toHaveText('1');

    // Step 4: Go to cart
    await page.click('[data-testid="cart-icon"]');
    await page.waitForURL('**/cart');

    // Verify cart contents
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(1);
    await expect(page.locator('[data-testid="cart-total"]')).toContainText('$1,200');

    // Step 5: Proceed to checkout
    await page.click('text=Checkout');
    await page.waitForURL('**/checkout');

    // Fill shipping info
    await page.fill('[data-testid="address"]', '123 Test St');
    await page.fill('[data-testid="city"]', 'Test City');
    await page.fill('[data-testid="zip"]', '12345');

    // Fill payment info (using test card)
    await page.fill('[data-testid="card-number"]', '4242424242424242');
    await page.fill('[data-testid="card-expiry"]', '12/25');
    await page.fill('[data-testid="card-cvc"]', '123');

    // Step 6: Place order
    await page.click('text=Place Order');

    // Wait for confirmation
    await page.waitForURL('**/order/confirmation/*');
    
    // Verify confirmation page
    await expect(page.locator('text=Order Confirmed')).toBeVisible();
    await expect(page.locator('[data-testid="order-number"]')).toBeVisible();

    // Take screenshot for verification
    await page.screenshot({ path: 'tests/screenshots/order-success.png' });

    // Verify order appears in order history
    await page.click('[data-testid="user-menu"]');
    await page.click('text=My Orders');
    await page.waitForURL('**/orders');

    const orders = await page.locator('[data-testid="order-item"]');
    await expect(orders.first()).toBeVisible();
    await expect(orders.first()).toContainText('$1,200');
  });

  test('should show error for out of stock items', async () => {
    // Login
    await page.click('text=Login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');

    // Navigate to out-of-stock product
    await page.goto('http://localhost:3000/product/out-of-stock-item');

    // Try to add to cart
    await page.click('text=Add to Cart');

    // Verify error message
    await expect(page.locator('[data-testid="error-toast"]'))
      .toContainText('Out of stock');

    // Verify cart not updated
    await expect(page.locator('[data-testid="cart-badge"]')).toHaveText('0');
  });

  test('should handle network errors gracefully', async () => {
    // Intercept and fail API calls
    await page.route('**/api/**', route => route.abort());

    await page.click('text=Login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');

    // Verify error message
    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('Network error');
  });
});

⏺ E2E tests completed
⏺ 3 test scenarios created
⏺ Screenshots captured for visual verification
```

**Sub-agent 3: Database Validator**

```typescript
⏺ Creating tests/integration/db-validation.test.ts...

import { Pool } from 'pg';
import { setupTestDB, teardownTestDB } from '../helpers/db';

describe('Database Validation', () => {
  let db: Pool;

  beforeAll(async () => {
    db = await setupTestDB();
  });

  afterAll(async () => {
    await teardownTestDB(db);
  });

  describe('Transaction Integrity', () => {
    it('should rollback order on payment failure', async () => {
      const client = await db.connect();
      
      try {
        await client.query('BEGIN');

        // Create order
        const orderResult = await client.query(
          'INSERT INTO orders (user_id, total, status) VALUES ($1, $2, $3) RETURNING id',
          [1, 1200, 'pending']
        );
        const orderId = orderResult.rows[0].id;

        // Create order items
        await client.query(
          'INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)',
          [orderId, 1, 2, 1200]
        );

        // Update product stock
        await client.query(
          'UPDATE products SET stock = stock - 2 WHERE id = $1',
          [1]
        );

        // Simulate payment failure
        throw new Error('Payment failed');

        await client.query('COMMIT');
      } catch (error) {
        await client.query('ROLLBACK');
      } finally {
        client.release();
      }

      // Verify nothing was committed
      const order = await db.query(
        'SELECT * FROM orders WHERE user_id = $1',
        [1]
      );
      expect(order.rows.length).toBe(0);

      // Verify stock unchanged
      const product = await db.query(
        'SELECT stock FROM products WHERE id = $1',
        [1]
      );
      expect(product.rows[0].stock).toBe(10); // Original stock
    });

    it('should maintain referential integrity', async () => {
      // Create order
      const orderResult = await db.query(
        'INSERT INTO orders (user_id, total, status) VALUES ($1, $2, $3) RETURNING id',
        [1, 1200, 'completed']
      );
      const orderId = orderResult.rows[0].id;

      // Try to delete user with orders (should fail)
      await expect(
        db.query('DELETE FROM users WHERE id = $1', [1])
      ).rejects.toThrow();

      // Verify order still exists
      const order = await db.query(
        'SELECT * FROM orders WHERE id = $1',
        [orderId]
      );
      expect(order.rows.length).toBe(1);
    });
  });

  describe('Isolation Levels', () => {
    it('should prevent dirty reads', async () => {
      const client1 = await db.connect();
      const client2 = await db.connect();

      try {
        // Client 1 starts transaction
        await client1.query('BEGIN ISOLATION LEVEL READ COMMITTED');
        await client1.query(
          'UPDATE products SET stock = 5 WHERE id = $1',
          [1]
        );

        // Client 2 should not see uncommitted changes
        const result = await client2.query(
          'SELECT stock FROM products WHERE id = $1',
          [1]
        );
        expect(result.rows[0].stock).toBe(10); // Original value

        await client1.query('COMMIT');

        // Now client 2 should see the change
        const result2 = await client2.query(
          'SELECT stock FROM products WHERE id = $1',
          [1]
        );
        expect(result2.rows[0].stock).toBe(5);
      } finally {
        client1.release();
        client2.release();
      }
    });

    it('should handle concurrent updates correctly', async () => {
      // Create product with stock = 10
      await db.query(
        'INSERT INTO products (id, name, stock) VALUES ($1, $2, $3)',
        [999, 'Test Product', 10]
      );

      // Simulate 5 concurrent updates (each reducing stock by 2)
      const updates = Array(5).fill(null).map(async () => {
        const client = await db.connect();
        try {
          await client.query('BEGIN');
          
          // Read current stock
          const result = await client.query(
            'SELECT stock FROM products WHERE id = $1 FOR UPDATE',
            [999]
          );
          const currentStock = result.rows[0].stock;

          // Check if enough stock
          if (currentStock >= 2) {
            await client.query(
              'UPDATE products SET stock = stock - 2 WHERE id = $1',
              [999]
            );
            await client.query('COMMIT');
            return true;
          } else {
            await client.query('ROLLBACK');
            return false;
          }
        } finally {
          client.release();
        }
      });

      const results = await Promise.all(updates);

      // Exactly 5 should succeed (10 / 2 = 5)
      const successful = results.filter(r => r === true);
      expect(successful.length).toBe(5);

      // Final stock should be 0
      const finalStock = await db.query(
        'SELECT stock FROM products WHERE id = $1',
        [999]
      );
      expect(finalStock.rows[0].stock).toBe(0);
    });
  });

  describe('Data Consistency', () => {
    it('should maintain cart and order totals consistency', async () => {
      // Create order with items
      const orderResult = await db.query(
        'INSERT INTO orders (user_id, total, status) VALUES ($1, $2, $3) RETURNING id',
        [1, 0, 'pending'] // Will be updated
      );
      const orderId = orderResult.rows[0].id;

      await db.query(
        'INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)',
        [orderId, 1, 2, 600]
      );
      await db.query(
        'INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ($1, $2, $3, $4)',
        [orderId, 2, 1, 300]
      );

      // Calculate total from items
      const itemsTotal = await db.query(
        'SELECT SUM(quantity * price) as total FROM order_items WHERE order_id = $1',
        [orderId]
      );

      // Update order total
      await db.query(
        'UPDATE orders SET total = $1 WHERE id = $2',
        [itemsTotal.rows[0].total, orderId]
      );

      // Verify consistency
      const order = await db.query(
        'SELECT total FROM orders WHERE id = $1',
        [orderId]
      );
      expect(parseFloat(order.rows[0].total)).toBe(1500); // 1200 + 300
    });
  });
});

⏺ Database validation tests completed
⏺ 6 test cases created
⏺ Coverage: transactions, isolation, referential integrity
```

**Phase 2: Integration**

```
⏺ All sub-agents completed. Integrating results...

⏺ Creating test suite configuration...
⏺ File: jest.config.integration.js

module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/tests/integration/**/*.test.ts'],
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  coverageDirectory: 'coverage/integration',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};

⏺ Creating Playwright configuration...
⏺ File: playwright.config.ts

import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run start:test',
    port: 3000,
  },
});

⏺ Creating test runner script...
⏺ File: scripts/run-integration-tests.sh

#!/bin/bash

echo "🚀 Starting Integration Test Suite"

# Start test database
docker-compose -f docker-compose.test.yml up -d postgres
sleep 5

# Run database migrations
npm run migrate:test

# Run API integration tests
echo "📋 Running API Integration Tests..."
npm run test:integration

# Start test server
npm run start:test &
SERVER_PID=$!
sleep 10

# Run E2E tests
echo "🌐 Running E2E Tests..."
npm run test:e2e

# Cleanup
kill $SERVER_PID
docker-compose -f docker-compose.test.yml down

echo "✅ Integration Test Suite Completed"

⏺ Creating package.json scripts...

"scripts": {
  "test:integration": "jest --config jest.config.integration.js",
  "test:e2e": "playwright test",
  "test:all": "./scripts/run-integration-tests.sh"
}
```

**Final Output:**

```
✅ Integration test suite completed

📊 Summary:
- API Integration Tests: 15 cases
- E2E Tests: 3 scenarios
- Database Validation: 6 cases
- Total: 24 test cases

📁 Files Created:
- tests/integration/order-flow.test.ts (API tests)
- tests/e2e/order-flow.spec.ts (E2E tests)
- tests/integration/db-validation.test.ts (DB tests)
- jest.config.integration.js
- playwright.config.ts
- scripts/run-integration-tests.sh

🎯 Coverage Areas:
- Happy path: Complete order flow
- Error scenarios: Stock, payment, auth
- Concurrency: Race conditions
- Database: Transactions, isolation, integrity

🚀 To run:
npm run test:all

📌 Next Steps:
1. Review test cases
2. Add environment-specific configs
3. Integrate with CI/CD
4. Set up test data fixtures

Would you like me to:
- Add more error scenarios?
- Create CI/CD workflow?
- Generate test data fixtures?
```

## 실전 2: Playwright 에이전트로 자가 치유 E2E 테스트

### Playwright 전문 에이전트

Playwright는 Claude Code용 3가지 전문 에이전트를 제공합니다.

```bash
# Playwright MCP 서버 설치
npm install -g @playwright/test
npx playwright-mcp install

# Claude Code에서 사용
$ claude
> /mcp add playwright
```

### 3가지 Playwright 에이전트

**1. Planner Agent (탐색자)**
- 애플리케이션을 실제로 탐색
- QA 엔지니어처럼 시나리오 작성
- Markdown 테스트 계획 생성

**2. Generator Agent (생성자)**
- Markdown 계획을 Playwright 코드로 변환
- 시맨틱 로케이터 사용
- Best practices 적용

**3. Healer Agent (치유자)**
- 테스트 실패 시 자동 수정
- UI 변경 감지
- 셀렉터 자동 업데이트

### 사용 예시

```bash
$ claude
> Playwright 에이전트를 사용해서
> 로그인 페이지 E2E 테스트를 작성해줘.
> 
> 애플리케이션을 먼저 탐색하고,
> 테스트 계획을 만든 다음,
> 코드를 생성해줘.

⏺ Spawning Planner Agent...

# Planner Agent 실행
⏺ Exploring application at http://localhost:3000/login...
⏺ Found login form with:
  - Email input (id="email")
  - Password input (id="password")
  - Submit button (text="Sign In")
  - "Forgot password?" link
  - "Create account" link

⏺ Creating test plan...

# Test Plan: Login Page

## Scenario 1: Successful Login
1. Navigate to /login
2. Enter valid email
3. Enter valid password
4. Click "Sign In"
5. Verify redirect to /dashboard
6. Verify user name displayed in header

## Scenario 2: Invalid Credentials
1. Navigate to /login
2. Enter invalid email
3. Enter invalid password
4. Click "Sign In"
5. Verify error message "Invalid credentials"
6. Verify still on /login page

## Scenario 3: Empty Fields
1. Navigate to /login
2. Click "Sign In" without entering data
3. Verify validation errors shown
4. Verify "Email required" message
5. Verify "Password required" message

⏺ Spawning Generator Agent...

# Generator Agent 실행
⏺ Transforming plan to Playwright test...
⏺ Using semantic locators...
⏺ Implementing proper waits...

// login.spec.ts (자동 생성)
import { test, expect } from '@playwright/test';

test.describe('Login Page', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    // Fill email
    await page.getByLabel('Email').fill('user@example.com');
    
    // Fill password
    await page.getByLabel('Password').fill('password123');
    
    // Click sign in
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    // Wait for navigation
    await page.waitForURL('/dashboard');
    
    // Verify user logged in
    await expect(page.getByText('Welcome, User')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.getByLabel('Email').fill('wrong@example.com');
    await page.getByLabel('Password').fill('wrongpass');
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    // Should stay on login page
    await expect(page).toHaveURL('/login');
    
    // Error message should appear
    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });

  test('should validate empty fields', async ({ page }) => {
    await page.getByRole('button', { name: 'Sign In' }).click();
    
    // Validation errors
    await expect(page.getByText('Email required')).toBeVisible();
    await expect(page.getByText('Password required')).toBeVisible();
  });
});

⏺ Running tests...
⏺ All tests passed ✅

# 시간이 지나서 UI가 변경된 경우...
⏺ Test failed: Cannot find element with label "Email"

⏺ Spawning Healer Agent...

# Healer Agent 실행
⏺ Analyzing test failure...
⏺ Inspecting current UI...
⏺ Found email input with id="user-email" (changed from label)
⏺ Suggesting fix...

// Updated selector
- await page.getByLabel('Email').fill('user@example.com');
+ await page.locator('#user-email').fill('user@example.com');

⏺ Applying fix and re-running...
⏺ Test passed ✅
```

## 실전 3: TDD 워크플로우 with 서브 에이전트

### Test-First Development

```bash
$ claude
> TDD 방식으로 장바구니 기능을 개발해줘:
> 
> 1. 먼저 통합 테스트 작성 (실패하는 테스트)
> 2. 최소한의 코드로 테스트 통과
> 3. 리팩토링
> 
> 서브 에이전트 활용:
> - test-writer: 테스트 작성
> - code-writer: 구현 작성
> - code-reviewer: 리팩토링 제안

⏺ Spawning test-writer sub-agent...

# Test Writer Agent
⏺ Writing failing integration tests first...

// cart.test.ts
describe('Shopping Cart', () => {
  it('should add item to cart', async () => {
    const cart = new ShoppingCart();
    const item = { id: 1, name: 'Product', price: 100 };
    
    await cart.addItem(item, 2);
    
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].quantity).toBe(2);
    expect(cart.getTotal()).toBe(200);
  });

  it('should update quantity if item already exists', async () => {
    const cart = new ShoppingCart();
    const item = { id: 1, name: 'Product', price: 100 };
    
    await cart.addItem(item, 2);
    await cart.addItem(item, 3);
    
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].quantity).toBe(5);
    expect(cart.getTotal()).toBe(500);
  });

  it('should remove item from cart', async () => {
    const cart = new ShoppingCart();
    const item = { id: 1, name: 'Product', price: 100 };
    
    await cart.addItem(item, 2);
    await cart.removeItem(1);
    
    expect(cart.items).toHaveLength(0);
    expect(cart.getTotal()).toBe(0);
  });
});

⏺ Running tests...
❌ All tests failed (as expected - no implementation)

⏺ Spawning code-writer sub-agent...

# Code Writer Agent
⏺ Writing minimal implementation to pass tests...

// cart.ts
export class ShoppingCart {
  items: CartItem[] = [];

  async addItem(item: Product, quantity: number) {
    const existing = this.items.find(i => i.id === item.id);
    
    if (existing) {
      existing.quantity += quantity;
    } else {
      this.items.push({ ...item, quantity });
    }
  }

  async removeItem(itemId: number) {
    this.items = this.items.filter(i => i.id !== itemId);
  }

  getTotal(): number {
    return this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }
}

⏺ Running tests again...
✅ All tests passed

⏺ Spawning code-reviewer sub-agent...

# Code Reviewer Agent
⏺ Analyzing code for refactoring opportunities...

Suggestions:
1. Add input validation (negative quantities, null items)
2. Extract total calculation to cached property
3. Add events for cart changes
4. Consider immutability for items array

⏺ Applying refactoring...

// cart.ts (refactored)
export class ShoppingCart {
  private _items: CartItem[] = [];
  private _totalCache: number | null = null;

  get items(): ReadonlyArray<CartItem> {
    return this._items;
  }

  async addItem(item: Product, quantity: number) {
    if (!item || quantity <= 0) {
      throw new Error('Invalid item or quantity');
    }

    const existing = this._items.find(i => i.id === item.id);
    
    if (existing) {
      existing.quantity += quantity;
    } else {
      this._items.push({ ...item, quantity });
    }

    this._totalCache = null; // Invalidate cache
    this.emit('cart:updated');
  }

  async removeItem(itemId: number) {
    const initialLength = this._items.length;
    this._items = this._items.filter(i => i.id !== itemId);
    
    if (this._items.length < initialLength) {
      this._totalCache = null;
      this.emit('cart:updated');
    }
  }

  getTotal(): number {
    if (this._totalCache === null) {
      this._totalCache = this._items.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
    }
    return this._totalCache;
  }
}

⏺ Running tests again...
✅ All tests still pass

⏺ Adding new tests for edge cases...

it('should throw error for invalid quantity', async () => {
  const cart = new ShoppingCart();
  const item = { id: 1, name: 'Product', price: 100 };
  
  await expect(cart.addItem(item, -1)).rejects.toThrow();
  await expect(cart.addItem(item, 0)).rejects.toThrow();
});

⏺ Running full test suite...
✅ All 5 tests passed
```

## 서브 에이전트 베스트 프랙티스

### 1. 명확한 역할 분리

```markdown
# ❌ 나쁜 예: 모호한 역할
general-tester.md
→ 뭐든지 테스트

# ✅ 좋은 예: 명확한 역할
integration-test-specialist.md → API + DB 통합 테스트
e2e-test-specialist.md → 브라우저 E2E
unit-test-specialist.md → 단위 테스트
performance-test-specialist.md → 성능/부하 테스트
```

### 2. 적절한 도구 권한

```markdown
# Test Writer (생성)
tools:
  - Read    # 기존 코드 읽기
  - Write   # 새 테스트 작성
  - Bash    # 테스트 실행

# Test Reviewer (검증)
tools:
  - Read    # 코드만 읽기
  - Grep    # 패턴 검색
  # Write 없음 - 수정 불가
```

### 3. 체계적인 커뮤니케이션

```bash
# 서브 에이전트 간 통신
$ claude
> API 테스트를 작성하고,
> 실패하는 케이스를 별도 에이전트가 분석하게 해줘.
> 
> 통신 방법: scratchpad.md 파일 사용

# Agent 1: Test Writer
⏺ Writing tests to scratchpad.md...
⏺ Results: 10 tests, 3 failures

# Agent 2: Test Analyzer
⏺ Reading scratchpad.md...
⏺ Analyzing failures...
⏺ Writing analysis to scratchpad.md...

# Main Agent
⏺ Reading final results from scratchpad.md...
⏺ Summary: 3 tests need mock data updates
```

### 4. 진행 상황 추적

```markdown
# .claude/agents/test-progress-tracker.md

---
name: test-progress-tracker
description: Track testing progress and report status
---

## Output Format

After each test run, report:
1. Total tests: X
2. Passed: Y
3. Failed: Z
4. Coverage: N%
5. Next actions: [list]

## Progress Persistence

Save results to: `.claude/test-results.json`
```

## 커스텀 서브 에이전트 라이브러리

### 테스터용 추천 에이전트

**.claude/agents/api-integration-tester.md:**
```markdown
---
name: api-integration-tester
description: API integration test specialist
tools: [Read, Write, Edit, Bash, Grep]
model: sonnet
---

Specializes in:
- REST API testing (Jest + Supertest)
- Database state verification
- Mock external services
- Error scenario coverage
```

**.claude/agents/e2e-playwright-specialist.md:**
```markdown
---
name: e2e-playwright-specialist
description: E2E testing with Playwright
tools: [Read, Write, Edit, Bash]
model: sonnet
---

Expertise:
- Semantic selectors
- Proper wait strategies
- Screenshot on failure
- Cross-browser testing
```

**.claude/agents/test-data-generator.md:**
```markdown
---
name: test-data-generator
description: Generate realistic test fixtures
tools: [Read, Write]
model: sonnet
---

Creates:
- Faker.js-based fixtures
- Edge case data
- Relationship-aware data
- Database seeds
```

**.claude/agents/test-failure-analyzer.md:**
```markdown
---
name: test-failure-analyzer
description: Diagnose test failures
tools: [Read, Grep, Bash]
model: sonnet
---

Analyzes:
- Stack traces
- Error messages
- Timing issues
- Environment problems
```

### 실전 활용

```bash
$ claude
> 테스트 실패를 분석해줘

# Claude가 자동으로 test-failure-analyzer 호출
⏺ Spawning test-failure-analyzer...
⏺ Reading test output...
⏺ Analyzing stack trace...

Found issues:
1. Mock data mismatch (expected User.id, got undefined)
2. Async timing issue (callback before promise resolves)
3. Database connection not cleaned up

Suggested fixes:
1. Update mock in tests/mocks/user.ts
2. Add await to line 45
3. Add afterAll cleanup
```

## CI/CD 통합

### GitHub Actions 자동 생성

```bash
$ claude
> GitHub Actions workflow를 만들어줘:
> 
> 트리거: PR to main
> 작업:
> 1. 단위 테스트
> 2. 통합 테스트
> 3. E2E 테스트
> 4. 테스트 커버리지 리포트
> 5. 실패 시 PR 차단

⏺ Creating .github/workflows/integration-tests.yml...

(자동 생성된 워크플로우 - 안티그래비티 가이드 참조)
```

## 결론

Claude Code는 **테스터를 위한 터미널 기반 테스트 오케스트레이터**입니다.

**핵심 가치:**
1. **서브 에이전트 병렬 실행**: 3-5배 빠른 테스트 작성
2. **전문화된 에이전트**: API, E2E, DB 각 분야 전문가
3. **자가 치유**: Playwright Healer Agent로 테스트 자동 수정
4. **TDD 자동화**: 테스트 먼저, 구현, 리팩토링 자동 순환
5. **완전 자동화**: 작성 → 실행 → 분석 → 수정

**안티그래비티 vs Claude Code (테스터 관점):**

| 특성 | Antigravity | Claude Code |
|------|-------------|-------------|
| E2E 테스트 | ⭐⭐⭐⭐⭐ (Browser Agent 내장) | ⭐⭐⭐⭐ (Playwright MCP) |
| API 테스트 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 서브 에이전트 | ⭐⭐⭐ (제한적) | ⭐⭐⭐⭐⭐ (완전한 오케스트레이션) |
| 학습 곡선 | 낮음 | 높음 |
| 병렬 처리 | 5개 제한 | 무제한 |
| 터미널 친화성 | 낮음 | 높음 |

**추천:**
- **GUI 선호, E2E 중심**: Antigravity
- **CLI 선호, 통합 테스트 중심**: Claude Code
- **최선**: 두 도구 병용

**시작하기:**
```bash
$ brew install anthropic/claude/claude-code
$ cd my-project
$ claude
> 통합 테스트를 작성해줘. 서브 에이전트를 활용해서.
```

테스트는 이제 "작성하는 것"이 아니라 "오케스트레이션하는 것"입니다!

---

**문서 작성 일자: 2026-01-22**
