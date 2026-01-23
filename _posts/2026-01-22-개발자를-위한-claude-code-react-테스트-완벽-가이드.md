---
title: "개발자를 위한 Claude Code React 테스트 완벽 가이드"
date: 2026-01-22 21:30:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  unit-test,  React,  Playwright,  MCP,  Guide,  Claude.write]
---

## UI 단위 테스트 + E2E 브라우저 테스트 통합 솔루션

---

## 🚨 중요: 브라우저 지원 이해하기

### Claude Code vs Antigravity 핵심 차이

```
❌ Claude Code (기본 상태)
- 터미널 기반 CLI 도구
- 브라우저 없음
- E2E 테스트 직접 불가

✅ Google Antigravity (기본 상태)
- GUI 기반 IDE
- Built-in Browser Agent
- 브라우저 자동화 즉시 가능
```

### 해결책: MCP (Model Context Protocol)

Claude Code가 브라우저를 사용하려면 **MCP 서버 설치 필수**

```bash
# Playwright MCP (권장)
$ claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright

# 또는 Puppeteer MCP
$ claude mcp add puppeteer -- npx -y puppeteer-mcp-claude serve
```

## 테스트 전략: 2-Tier 접근법

### Tier 1: 단위 테스트 (브라우저 불필요)

```
도구: React Testing Library + Jest/Vitest
환경: jsdom (가상 DOM)
속도: 매우 빠름 (밀리초)
용도: 컴포넌트 로직, UI 상태, 이벤트
```

### Tier 2: E2E 테스트 (브라우저 필요)

```
도구: Playwright + MCP
환경: 실제 Chrome/Firefox/Safari
속도: 느림 (초 단위)
용도: 사용자 플로우, 통합 시나리오
```

## 1단계: 환경 설정

### Claude Code 설치

```bash
# macOS
$ brew install anthropic/claude/claude-code

# Windows
$ winget install Anthropic.Claude

# 확인
$ claude --version
```

### Playwright MCP 설치

```bash
# MCP 서버 추가
$ claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright

# 연결 확인
$ claude
> /mcp

출력:
✓ playwright - connected
```

### 프로젝트 초기화

```bash
$ cd my-react-app

# React Testing Library 설치 (단위 테스트)
$ npm install -D @testing-library/react @testing-library/user-event @testing-library/jest-dom

# Playwright 설치 (E2E 테스트)
$ npm install -D @playwright/test

# MSW (API Mocking)
$ npm install -D msw
```

## 2단계: 프로젝트 설정

### .claude/SKILL.md 생성

~~~markdown
# React Testing Standards

## 테스트 전략

### Tier 1: 단위 테스트 (React Testing Library)
**목적**: 컴포넌트 로직, UI 상태, 사용자 인터랙션
**환경**: jsdom (브라우저 불필요)
**속도**: 빠름 (< 100ms per test)

**적용 대상**:
- 모든 React 컴포넌트
- Custom Hooks
- 유틸리티 함수
- 컨텍스트 Provider

### Tier 2: E2E 테스트 (Playwright + MCP)
**목적**: 전체 사용자 플로우, 브라우저 통합
**환경**: 실제 Chrome/Firefox (Playwright MCP 사용)
**속도**: 느림 (초 단위)

**적용 대상**:
- 핵심 사용자 플로우 (로그인 → 구매)
- 크리티컬 비즈니스 로직
- 크로스 브라우저 검증

## Tech Stack

### 단위 테스트
- **Framework**: Jest 29 / Vitest 1.0
- **UI Testing**: React Testing Library 14
- **User Events**: @testing-library/user-event
- **Mocking**: MSW (Mock Service Worker) 2.0

### E2E 테스트
- **Framework**: Playwright 1.40+
- **Browser Control**: Playwright MCP
- **Browsers**: Chromium, Firefox, WebKit

## 단위 테스트 원칙

### ✅ DO:
- 사용자 행동 테스트 (클릭, 타이핑, 제출)
- 접근성 기반 쿼리 (`getByRole`, `getByLabelText`)
- 비동기 처리 (`waitFor`, `findBy*`)
- API 모킹 (MSW)
- 에러 상태 테스트

### ❌ DON'T:
- 구현 세부사항 테스트 (state, props 직접 접근)
- CSS 클래스/스타일 테스트
- 프레임워크 내부 테스트
- `querySelector` 사용

## E2E 테스트 원칙

### ✅ DO:
- 실제 사용자 플로우 시뮬레이션
- 네트워크 조건 테스트 (slow 3G, offline)
- 크로스 브라우저 검증
- 스크린샷 비교 (visual regression)

### ❌ DON'T:
- 모든 것을 E2E로 테스트 (느림)
- 단위 테스트 대체용으로 사용
- 세밀한 컴포넌트 테스트

## 파일 구조

```
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx      ← 단위 테스트
      index.ts
  
tests/
  e2e/
    login-flow.spec.ts     ← E2E 테스트
    checkout.spec.ts
```

## Query 우선순위 (RTL)

1. `getByRole` - 접근성 최우선
2. `getByLabelText` - 폼 요소
3. `getByPlaceholderText` - label 없을 때
4. `getByText` - 텍스트 콘텐츠
5. `getByTestId` - 최후의 수단

## 커버리지 목표

```javascript
// jest.config.js
coverageThreshold: {
  global: {
    statements: 80,
    branches: 75,
    functions: 80,
    lines: 80
  }
}
```

## E2E 테스트 범위

- 전체 테스트의 10-20%만 E2E
- 핵심 비즈니스 플로우 3-5개
- CI에서 병렬 실행
~~~

## 3단계: 서브 에이전트 생성

### Agent 1: 단위 테스트 전문가

**.claude/agents/react-unit-tester.md:**

~~~markdown
---
name: react-unit-tester
description: |
  React 컴포넌트 단위 테스트 전문가.
  React Testing Library 사용.
  브라우저 불필요 (jsdom).
tools: [Read, Write, Edit, Bash, Grep]
model: sonnet
---

# React Unit Test Specialist

React Testing Library로 단위 테스트를 작성합니다.

## 테스트 범위

- ✅ 컴포넌트 렌더링
- ✅ Props 변경 반응
- ✅ 사용자 인터랙션
- ✅ 조건부 렌더링
- ✅ Hooks 상태 관리
- ❌ 실제 브라우저 필요 (E2E 에이전트 사용)

## 테스트 템플릿

```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('ComponentName', () => {
  it('should render with default props', () => {
    render(<ComponentName />);
    expect(screen.getByRole('button')).toBeInTheDocument();
  });

  it('should handle user interaction', async () => {
    const user = userEvent.setup();
    render(<ComponentName />);
    
    await user.click(screen.getByRole('button'));
    
    expect(screen.getByText(/clicked/i)).toBeInTheDocument();
  });
});
```

## API Mocking (MSW)

```javascript
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([{ id: 1, name: 'Test' }]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## 완료 체크리스트

- [ ] 모든 props 조합 테스트
- [ ] 모든 사용자 이벤트 테스트
- [ ] 로딩/에러 상태 테스트
- [ ] API 호출 모킹
- [ ] 접근성 검증
- [ ] 80%+ 커버리지
~~~

### Agent 2: E2E 테스트 전문가

**.claude/agents/react-e2e-tester.md:**

~~~markdown
---
name: react-e2e-tester
description: |
  Playwright E2E 테스트 전문가.
  실제 브라우저 사용 (Playwright MCP).
  전체 사용자 플로우 테스트.
tools: [Read, Write, Edit, Bash, Grep]
model: sonnet
---

# React E2E Test Specialist

Playwright + MCP로 E2E 테스트를 작성합니다.

## ⚠️ 중요: MCP 의존성

이 에이전트는 Playwright MCP 서버가 필요합니다.

확인 방법:
```bash
$ claude
> /mcp
# "playwright - connected" 확인
```

설치:
```bash
$ claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright
```

## 테스트 범위

- ✅ 전체 사용자 플로우 (로그인 → 구매)
- ✅ 실제 브라우저 인터랙션
- ✅ 네트워크 조건 테스트
- ✅ 크로스 브라우저 검증
- ❌ 개별 컴포넌트 (단위 테스트 사용)

## 테스트 템플릿

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Login Flow', () => {
  test('should login and navigate to dashboard', async ({ page }) => {
    // Navigate
    await page.goto('http://localhost:3000/login');
    
    // Fill form
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    
    // Submit
    await page.click('[data-testid="submit"]');
    
    // Verify
    await expect(page).toHaveURL('http://localhost:3000/dashboard');
    await expect(page.locator('text=Welcome')).toBeVisible();
    
    // Screenshot
    await page.screenshot({ path: 'screenshots/dashboard.png' });
  });
});
```

## 네트워크 모킹

```typescript
test('should handle API errors gracefully', async ({ page }) => {
  // Intercept API call
  await page.route('**/api/login', route => {
    route.fulfill({
      status: 401,
      body: JSON.stringify({ error: 'Invalid credentials' })
    });
  });

  await page.goto('http://localhost:3000/login');
  await page.fill('[data-testid="email"]', 'user@example.com');
  await page.fill('[data-testid="password"]', 'wrong');
  await page.click('[data-testid="submit"]');

  await expect(page.locator('text=Invalid credentials')).toBeVisible();
});
```

## 크로스 브라우저 설정

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

## 완료 체크리스트

- [ ] 핵심 사용자 플로우 커버
- [ ] 에러 시나리오 테스트
- [ ] 네트워크 조건 테스트
- [ ] 스크린샷 캡처
- [ ] 3개 브라우저 검증
~~~

## 4단계: 커스텀 명령어

### /test (단위 테스트)

**.claude/commands/test.md:**

```markdown
# /test

React 컴포넌트 단위 테스트 생성

Usage: `/test ComponentName`

## 워크플로우

1. 컴포넌트 읽기: `src/components/$ARGUMENTS.tsx`
2. react-unit-tester 에이전트 사용
3. 테스트 생성: `src/components/$ARGUMENTS.test.tsx`
4. 실행: `npm test $ARGUMENTS`
5. 커버리지 확인

## 요구사항

- React Testing Library 사용
- jsdom 환경 (브라우저 불필요)
- MSW로 API 모킹
- 80%+ 커버리지
- userEvent.setup() 사용
```

### /e2e (E2E 테스트)

**.claude/commands/e2e.md:**

```markdown
# /e2e

Playwright E2E 테스트 생성

Usage: `/e2e flow-name`

## 워크플로우

1. react-e2e-tester 에이전트 사용
2. Playwright MCP 연결 확인
3. 테스트 생성: `tests/e2e/$ARGUMENTS.spec.ts`
4. 실행: `npx playwright test $ARGUMENTS`
5. 스크린샷 확인

## 요구사항

- Playwright MCP 설치 필수
- 실제 브라우저 사용
- 전체 사용자 플로우
- 3개 브라우저 검증
```

## 실전 1: Button 컴포넌트 (단위 테스트)

### 컴포넌트

```typescript
// src/components/Button/Button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary';
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export const Button = ({ 
  variant = 'primary', 
  loading, 
  children,
  onClick 
}: ButtonProps) => {
  return (
    <button
      className={`btn btn-${variant}`}
      disabled={loading}
      onClick={onClick}
    >
      {loading && <span className="spinner" />}
      {children}
    </button>
  );
};
```

### Claude Code 실행

```bash
$ claude
> /test Button

⏺ Spawning react-unit-tester agent...
⏺ Analyzing Button.tsx...
⏺ Component type: Presentational
⏺ Props: variant, loading, children, onClick
⏺ Generating tests...
```

### 자동 생성된 테스트

```typescript
// src/components/Button/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  describe('Rendering', () => {
    it('renders children', () => {
      render(<Button>Click me</Button>);
      expect(screen.getByRole('button', { name: /click me/i }))
        .toBeInTheDocument();
    });

    it('applies variant class', () => {
      render(<Button variant="secondary">Button</Button>);
      expect(screen.getByRole('button')).toHaveClass('btn-secondary');
    });
  });

  describe('Loading State', () => {
    it('shows spinner when loading', () => {
      const { container } = render(<Button loading>Loading</Button>);
      expect(container.querySelector('.spinner')).toBeInTheDocument();
    });

    it('disables button when loading', () => {
      render(<Button loading>Loading</Button>);
      expect(screen.getByRole('button')).toBeDisabled();
    });
  });

  describe('User Interactions', () => {
    it('calls onClick when clicked', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button onClick={handleClick}>Click</Button>);
      await user.click(screen.getByRole('button'));
      
      expect(handleClick).toHaveBeenCalledTimes(1);
    });

    it('does not call onClick when loading', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button loading onClick={handleClick}>Loading</Button>);
      await user.click(screen.getByRole('button'));
      
      expect(handleClick).not.toHaveBeenCalled();
    });
  });
});

⏺ Running tests...
✅ 8 tests passed
✅ Coverage: 95.5%
⏺ Completed in 234ms
```

## 실전 2: 로그인 플로우 (E2E 테스트)

### Claude Code 실행

```bash
$ claude
> /e2e login-flow

⏺ Checking Playwright MCP connection...
✅ playwright - connected

⏺ Spawning react-e2e-tester agent...
⏺ Generating E2E test for login flow...
```

### 자동 생성된 E2E 테스트

```typescript
// tests/e2e/login-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:3000');
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    // Navigate to login
    await page.click('text=Sign In');
    await expect(page).toHaveURL(/.*login/);

    // Fill form
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');

    // Submit
    await page.click('[data-testid="submit"]');

    // Wait for navigation
    await page.waitForURL(/.*dashboard/);

    // Verify logged in
    await expect(page.locator('text=Welcome, User')).toBeVisible();
    
    // Screenshot for verification
    await page.screenshot({ 
      path: 'screenshots/login-success.png',
      fullPage: true 
    });
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.click('text=Sign In');
    
    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrongpass');
    await page.click('[data-testid="submit"]');

    // Should stay on login page
    await expect(page).toHaveURL(/.*login/);
    
    // Error message visible
    await expect(page.locator('text=Invalid credentials')).toBeVisible();
  });

  test('should handle network error', async ({ page }) => {
    // Simulate network failure
    await page.route('**/api/login', route => route.abort());

    await page.click('text=Sign In');
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="submit"]');

    await expect(page.locator('text=Network error')).toBeVisible();
  });
});

test.describe('Login Flow - Cross Browser', () => {
  test('works on Firefox', async ({ page, browserName }) => {
    test.skip(browserName !== 'firefox', 'Firefox only');
    
    await page.goto('http://localhost:3000/login');
    await page.fill('[data-testid="email"]', 'user@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="submit"]');
    
    await expect(page).toHaveURL(/.*dashboard/);
  });
});

⏺ Running E2E tests...
⏺ Using Playwright MCP...

Running 4 tests using 3 workers

  ✓ login-flow.spec.ts:5:3 › Login Flow › should login successfully (2.3s)
  ✓ login-flow.spec.ts:25:3 › Login Flow › should show error (1.8s)
  ✓ login-flow.spec.ts:38:3 › Login Flow › should handle network error (1.5s)
  ✓ login-flow.spec.ts:50:3 › Login Flow - Cross Browser › works on Firefox (2.1s)

  4 passed (7.7s)

⏺ Screenshots saved to screenshots/
⏺ Completed
```

## 실전 3: 통합 워크플로우 (단위 + E2E)

### 시나리오: 쇼핑 카트 기능

```bash
$ claude
> 쇼핑 카트 기능을 완전히 테스트해줘:
> 
> 1. AddToCartButton 컴포넌트 단위 테스트
> 2. CartSummary 컴포넌트 단위 테스트
> 3. 전체 쇼핑 플로우 E2E 테스트
>    (상품 선택 → 카트 추가 → 수량 변경 → 결제)
> 
> 서브 에이전트를 활용해서 병렬로 처리해줘.

⏺ Creating test plan with 3 parallel agents:
  1. react-unit-tester → AddToCartButton
  2. react-unit-tester → CartSummary
  3. react-e2e-tester → Shopping flow

⏺ Starting parallel execution...
```

**Agent 1: AddToCartButton 단위 테스트**

```typescript
// src/components/AddToCartButton/AddToCartButton.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { AddToCartButton } from './AddToCartButton';

const server = setupServer(
  http.post('/api/cart', async ({ request }) => {
    const { productId, quantity } = await request.json();
    return HttpResponse.json({
      success: true,
      cartId: '123',
      itemCount: quantity
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('AddToCartButton', () => {
  it('adds item to cart on click', async () => {
    const user = userEvent.setup();
    render(<AddToCartButton productId="prod-1" />);

    await user.click(screen.getByRole('button', { name: /add to cart/i }));

    await screen.findByText(/added to cart/i);
  });

  it('shows loading state during API call', async () => {
    const user = userEvent.setup();
    render(<AddToCartButton productId="prod-1" />);

    const button = screen.getByRole('button', { name: /add to cart/i });
    await user.click(button);

    expect(button).toBeDisabled();
    expect(screen.getByText(/adding/i)).toBeInTheDocument();
  });

  it('handles API error', async () => {
    server.use(
      http.post('/api/cart', () => {
        return HttpResponse.json(
          { error: 'Out of stock' },
          { status: 400 }
        );
      })
    );

    const user = userEvent.setup();
    render(<AddToCartButton productId="prod-1" />);

    await user.click(screen.getByRole('button'));

    await expect(screen.findByText(/out of stock/i)).resolves.toBeInTheDocument();
  });
});

⏺ 8 tests passed, Coverage: 92%
```

**Agent 2: CartSummary 단위 테스트**

```typescript
// src/components/CartSummary/CartSummary.test.tsx
import { render, screen } from '@testing-library/react';
import { CartSummary } from './CartSummary';

describe('CartSummary', () => {
  it('displays empty cart message', () => {
    render(<CartSummary items={[]} />);
    expect(screen.getByText(/cart is empty/i)).toBeInTheDocument();
  });

  it('calculates total correctly', () => {
    const items = [
      { id: '1', name: 'Item 1', price: 10, quantity: 2 },
      { id: '2', name: 'Item 2', price: 15, quantity: 1 }
    ];

    render(<CartSummary items={items} />);

    expect(screen.getByText(/total: \$35/i)).toBeInTheDocument();
  });

  it('shows item count', () => {
    const items = [
      { id: '1', name: 'Item 1', price: 10, quantity: 2 },
      { id: '2', name: 'Item 2', price: 15, quantity: 3 }
    ];

    render(<CartSummary items={items} />);

    expect(screen.getByText(/5 items/i)).toBeInTheDocument();
  });
});

⏺ 6 tests passed, Coverage: 88%
```

**Agent 3: 쇼핑 플로우 E2E 테스트**

```typescript
// tests/e2e/shopping-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Complete Shopping Flow', () => {
  test('should complete purchase from product to checkout', async ({ page }) => {
    // 1. Browse products
    await page.goto('http://localhost:3000/products');
    await expect(page.locator('[data-testid="product-card"]').first())
      .toBeVisible();

    // 2. Select product
    await page.click('[data-testid="product-card"]:first-child');
    await expect(page).toHaveURL(/.*product\/[^/]+/);

    // 3. Add to cart
    await page.click('[data-testid="add-to-cart"]');
    await expect(page.locator('text=Added to cart')).toBeVisible();

    // 4. Verify cart badge
    await expect(page.locator('[data-testid="cart-badge"]')).toHaveText('1');

    // 5. Go to cart
    await page.click('[data-testid="cart-icon"]');
    await expect(page).toHaveURL(/.*cart/);

    // 6. Update quantity
    await page.click('[data-testid="quantity-increase"]');
    await expect(page.locator('[data-testid="quantity"]')).toHaveText('2');

    // 7. Verify total updated
    const totalBefore = await page.locator('[data-testid="total"]').textContent();
    await page.click('[data-testid="quantity-increase"]');
    const totalAfter = await page.locator('[data-testid="total"]').textContent();
    expect(totalBefore).not.toBe(totalAfter);

    // 8. Proceed to checkout
    await page.click('text=Checkout');
    await expect(page).toHaveURL(/.*checkout/);

    // 9. Fill shipping info
    await page.fill('[data-testid="name"]', 'Test User');
    await page.fill('[data-testid="address"]', '123 Test St');
    await page.fill('[data-testid="city"]', 'Test City');
    await page.fill('[data-testid="zip"]', '12345');

    // 10. Fill payment (test card)
    await page.fill('[data-testid="card-number"]', '4242424242424242');
    await page.fill('[data-testid="card-expiry"]', '12/25');
    await page.fill('[data-testid="card-cvc"]', '123');

    // 11. Place order
    await page.click('text=Place Order');

    // 12. Verify confirmation
    await expect(page).toHaveURL(/.*order\/confirmation/);
    await expect(page.locator('text=Order Confirmed')).toBeVisible();

    // 13. Screenshot confirmation
    await page.screenshot({ 
      path: 'screenshots/order-confirmation.png',
      fullPage: true 
    });
  });

  test('should update cart total when quantity changes', async ({ page }) => {
    await page.goto('http://localhost:3000/cart');
    
    const initialTotal = await page.locator('[data-testid="total"]').textContent();
    
    await page.click('[data-testid="quantity-increase"]');
    
    await expect(async () => {
      const newTotal = await page.locator('[data-testid="total"]').textContent();
      expect(newTotal).not.toBe(initialTotal);
    }).toPass();
  });

  test('should remove item from cart', async ({ page }) => {
    await page.goto('http://localhost:3000/cart');
    
    const itemCount = await page.locator('[data-testid="cart-item"]').count();
    
    await page.click('[data-testid="remove-item"]:first-child');
    
    await expect(page.locator('[data-testid="cart-item"]'))
      .toHaveCount(itemCount - 1);
  });
});

⏺ 3 E2E tests passed in 18.5s
```

**통합 결과:**

```
✅ 종합 테스트 완료

단위 테스트:
- AddToCartButton: 8 tests, 92% coverage
- CartSummary: 6 tests, 88% coverage
Total: 14 tests in 456ms

E2E 테스트:
- Shopping flow: 3 tests in 18.5s
- 3 browsers tested (Chrome, Firefox, Safari)
- 3 screenshots captured

전체 커버리지: 90%

파일 생성:
- src/components/AddToCartButton/AddToCartButton.test.tsx
- src/components/CartSummary/CartSummary.test.tsx
- tests/e2e/shopping-flow.spec.ts
- screenshots/ (3 images)
```

## CI/CD 통합

### GitHub Actions

```yaml
name: Tests

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npx playwright test
      
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

## 비교표: 도구 선택 가이드

### 단위 테스트 vs E2E 테스트

| 특성 | 단위 테스트 (RTL) | E2E 테스트 (Playwright) |
|------|------------------|------------------------|
| **환경** | jsdom (가상) | 실제 브라우저 |
| **속도** | 매우 빠름 (ms) | 느림 (초) |
| **설정** | 간단 | MCP 설치 필요 |
| **범위** | 컴포넌트 단위 | 전체 플로우 |
| **디버깅** | 쉬움 | 중간 |
| **CI 비용** | 낮음 | 높음 |
| **비율** | 80-90% | 10-20% |

### Claude Code vs Antigravity

| 특성 | Claude Code + MCP | Antigravity |
|------|-------------------|-------------|
| **환경** | CLI (터미널) | GUI (IDE) |
| **브라우저** | MCP 설치 필요 | Built-in |
| **단위 테스트** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **E2E 테스트** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **학습 곡선** | 중간 | 낮음 |
| **서브 에이전트** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **가격** | $20-200/월 | 무료 (Preview) |

## 권장사항

### 단위 테스트만 필요한 경우

```
✅ Claude Code (MCP 불필요)
- React Testing Library
- Jest/Vitest
- jsdom
- 빠르고 효율적
```

### E2E도 필요한 경우

**옵션 1: Claude Code + Playwright MCP**
```bash
장점:
- 터미널 워크플로우
- 서브 에이전트 활용
- CI/CD 통합 용이

단점:
- MCP 설정 필요
```

**옵션 2: Antigravity**
```bash
장점:
- 설정 없이 즉시 사용
- GUI 친화적

단점:
- 터미널 워크플로우 X
- 서브 에이전트 제한적
```

**옵션 3: 하이브리드 (최선)**
```bash
Claude Code:
- 단위 테스트 (빠름)
- 코드 리뷰
- 리팩토링

Antigravity:
- E2E 테스트 (간편)
- 브라우저 자동화
- 시각적 확인
```

## 트러블슈팅

### MCP 연결 실패

```bash
# 1. MCP 목록 확인
$ claude
> /mcp

# 2. 재연결
> /mcp
# Select server → Reconnect

# 3. 재설치
$ claude mcp remove playwright
$ claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright
```

### 테스트 실행 안됨

```bash
# Jest 캐시 클리어
$ npm test -- --clearCache

# Playwright 재설치
$ npx playwright install --with-deps
```

### act() 경고

```javascript
// ❌ 잘못됨
fireEvent.click(button);

// ✅ 올바름
const user = userEvent.setup();
await user.click(button);
```

## 베스트 프랙티스 요약

### 1. 테스트 피라미드 유지

```
       /\
      /E2E\      ← 10-20% (느림, 비쌈)
     /------\
    /  통합  \    ← 20-30%
   /----------\
  /   단위    \  ← 50-70% (빠름, 저렴)
 /--------------\
```

### 2. 올바른 도구 선택

```
컴포넌트 로직 → RTL (단위)
사용자 플로우 → Playwright (E2E)
API 통합 → MSW (Mock)
```

### 3. MCP 활용

```
단위 테스트: MCP 불필요
E2E 테스트: Playwright MCP 필수
```

### 4. 서브 에이전트 최대 활용

```
react-unit-tester: 컴포넌트 테스트
react-e2e-tester: E2E 플로우
병렬 실행으로 시간 절약
```

## 결론

**Claude Code는 React 테스트 자동화의 완벽한 솔루션입니다.**

**핵심 가치:**
1. **40배 빠른 단위 테스트**: 2시간 → 3분
2. **Playwright MCP로 E2E 지원**: 실제 브라우저
3. **2-Tier 전략**: 단위 (빠름) + E2E (정확)
4. **서브 에이전트**: 병렬 처리
5. **80%+ 커버리지**: 자동 달성

**시작하기:**
```bash
# 1. 설치
$ brew install anthropic/claude/claude-code

# 2. Playwright MCP 추가 (E2E 필요시)
$ claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright

# 3. 테스트 생성
$ claude
> /test Button          # 단위 테스트
> /e2e login-flow       # E2E 테스트
```

**테스트는 이제 "작성"이 아니라 "요청"입니다!**

---

**문서 작성 일자: 2026-01-22**
