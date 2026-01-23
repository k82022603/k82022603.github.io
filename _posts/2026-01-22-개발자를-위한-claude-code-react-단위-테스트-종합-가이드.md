---
title: "개발자를 위한 Claude Code React 단위 테스트 종합 가이드"
date: 2026-01-22 21:20:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  unit-test,  sub-agents,  React,  TDD,  Guide,  Claude.write]
---

## 서브 에이전트와 TDD로 테스트 자동화 혁신하기

---

## 핵심 질문: 왜 테스트를 AI에게 맡기는가?

### 현실

```
테스트 작성을 미루는 이유:
1. 지루하다 (반복적인 setup/teardown)
2. 시간이 없다 (기능 개발에 밀려남)
3. 뭘 테스트해야 할지 모르겠다
4. Mock 설정이 복잡하다

결과: 
- 테스트 커버리지 20%
- 버그는 프로덕션에서 발견
- 리팩토링 공포증
```

### Claude Code의 해결책

```bash
$ claude
> UserProfile 컴포넌트의 테스트를 작성해줘.
> React Testing Library 사용하고,
> 모든 사용자 인터랙션을 테스트해줘.

→ 3분 후:
  - 12개 테스트 케이스
  - 95% 코드 커버리지
  - Mock 자동 생성
  - 실행 후 모두 통과
```

**시간 비교:**
```
수동 작성: 2시간 (컴포넌트당)
Claude Code: 3분

→ 40배 빠름
```

## React Testing Library 핵심 개념

### 왜 RTL인가?

**❌ Enzyme (구식):**
```javascript
// 구현 세부사항 테스트
const wrapper = shallow(<Button />);
expect(wrapper.find('.button-class')).toHaveLength(1);
```

**✅ React Testing Library (현대적):**
```javascript
// 사용자 관점 테스트
render(<Button>Click me</Button>);
expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
```

### RTL 핵심 원칙

1. **사용자가 하는 대로 테스트**
   - DOM 쿼리는 실제 사용자가 요소를 찾는 방식
   - 구현 세부사항 무시

2. **접근성 우선**
   - `getByRole`, `getByLabelText` 권장
   - 스크린 리더가 읽는 방식

3. **비동기 처리**
   - `waitFor`, `findBy*` 활용
   - 실제 앱처럼 대기

## Claude Code 설정: SKILL.md

### .claude/SKILL.md 생성

~~~markdown
# React Testing Standards

## Tech Stack
- **Testing Framework**: Jest 29 / Vitest 1.0
- **UI Testing**: React Testing Library 14
- **Mocking**: MSW (Mock Service Worker) 2.0
- **Coverage Target**: 80% lines, 75% branches

## Component Categories

### 1. Presentational Components (UI only)
**Test Count**: 5-8 tests
**Focus**: 
- Props rendering
- User interactions (click, type, submit)
- Conditional rendering
- Accessibility

### 2. Container Components (with state/effects)
**Test Count**: 8-12 tests
**Focus**:
- Data fetching
- State updates
- Side effects
- Error handling

### 3. Form Components
**Test Count**: 10-15 tests
**Focus**:
- Validation
- Submission
- Error messages
- Disabled states

## Testing Principles

### ✅ DO:
- Test user behavior, not implementation
- Use semantic queries (`getByRole`, `getByLabelText`)
- Test accessibility
- Use `userEvent` for interactions
- Test error states
- Mock external dependencies (APIs, localStorage)

### ❌ DON'T:
- Test framework internals (React.useState works)
- Test CSS classes or styles
- Use `container.querySelector`
- Test library code (axios works)
- Create separate mock files for every dependency

## Query Priority

1. `getByRole` (best - accessibility)
2. `getByLabelText` (forms)
3. `getByPlaceholderText` (if no label)
4. `getByText` (non-interactive)
5. `getByTestId` (last resort)

## File Structure

```
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx  ← Test next to component
      index.ts
```

## Test Template

```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  it('should render with required props', () => {
    render(<ComponentName prop="value" />);
    expect(screen.getByRole('...')).toBeInTheDocument();
  });

  it('should handle user interaction', async () => {
    const user = userEvent.setup();
    render(<ComponentName />);
    
    await user.click(screen.getByRole('button'));
    
    expect(screen.getByText(/result/i)).toBeInTheDocument();
  });
});
```

## Mock Strategy

### API Mocking (MSW)
```javascript
// src/mocks/handlers.js
export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([{ id: 1, name: 'Test' }])
  })
];
```

### Function Mocking
```javascript
const mockOnClick = jest.fn();
```

### Module Mocking
```javascript
jest.mock('./api', () => ({
  fetchUser: jest.fn()
}));
```

## Coverage Threshold

Enforce in jest.config.js:
```javascript
coverageThreshold: {
  global: {
    statements: 80,
    branches: 75,
    functions: 80,
    lines: 80
  }
}
```

## Common Patterns to Avoid

### ❌ Testing Implementation
```javascript
// Bad
expect(component.state.count).toBe(1);
```

### ✅ Testing Behavior
```javascript
// Good
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

### ❌ Waiting with setTimeout
```javascript
// Bad
setTimeout(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
}, 1000);
```

### ✅ Waiting Properly
```javascript
// Good
await screen.findByText('Loaded');
// or
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});
```
~~~

## 서브 에이전트 생성

### 1. Component Test Writer

**.claude/agents/react-component-tester.md:**

~~~markdown
---
name: react-component-tester
description: |
  Expert in writing React component unit tests.
  Triggers when user asks to test React components,
  write tests for UI, or mentions React Testing Library.
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
model: sonnet
---

# React Component Testing Specialist

You write comprehensive React component tests using React Testing Library.

## Analysis Phase

Before writing tests:
1. Read the component file
2. Identify component type (presentational/container/form)
3. List all props and their types
4. Find all user interactions (clicks, inputs, etc.)
5. Identify conditional rendering
6. Check for API calls or side effects

## Test Structure

```javascript
describe('ComponentName', () => {
  // Group 1: Rendering
  describe('Rendering', () => {
    it('renders with required props');
    it('renders with optional props');
    it('renders without crashing');
  });

  // Group 2: User Interactions
  describe('User Interactions', () => {
    it('handles click events');
    it('handles input changes');
    it('handles form submission');
  });

  // Group 3: Conditional Logic
  describe('Conditional Rendering', () => {
    it('shows loading state');
    it('shows error state');
    it('shows empty state');
  });

  // Group 4: Integration
  describe('Integration', () => {
    it('calls API on mount');
    it('updates on prop changes');
  });
});
```

## Query Selection

1. Prefer `getByRole`:
   ```javascript
   screen.getByRole('button', { name: /submit/i })
   ```

2. Use `getByLabelText` for forms:
   ```javascript
   screen.getByLabelText(/email/i)
   ```

3. Use `getByTestId` as last resort:
   ```javascript
   screen.getByTestId('custom-element')
   ```

## User Event Pattern

Always use `userEvent.setup()`:
```javascript
const user = userEvent.setup();
await user.click(element);
await user.type(input, 'text');
```

## Async Testing

Use `findBy*` or `waitFor`:
```javascript
// Preferred
const element = await screen.findByText(/loaded/i);

// Alternative
await waitFor(() => {
  expect(screen.getByText(/loaded/i)).toBeInTheDocument();
});
```

## Mock Strategy

1. Mock API calls with MSW
2. Mock functions with `jest.fn()`
3. Mock modules only when necessary
4. Never mock React internals

## Completion Checklist

Before finishing:
- [ ] All user interactions tested
- [ ] All conditional branches tested
- [ ] All props tested
- [ ] Accessibility verified
- [ ] Tests run and pass
- [ ] No act() warnings
- [ ] Coverage > 80%
```

### 2. Hook Test Specialist

**.claude/agents/react-hook-tester.md:**

```markdown
---
name: react-hook-tester
description: |
  Specialist in testing React custom hooks.
  Triggers for "test hook", "custom hook testing".
tools: [Read, Write, Edit, Bash]
model: sonnet
---

# React Hook Testing Expert

Test React hooks using `@testing-library/react-hooks`.

## Hook Test Pattern

```javascript
import { renderHook, act } from '@testing-library/react';
import { useCustomHook } from './useCustomHook';

describe('useCustomHook', () => {
  it('initializes with default values', () => {
    const { result } = renderHook(() => useCustomHook());
    
    expect(result.current.value).toBe(initialValue);
  });

  it('updates state on action', () => {
    const { result } = renderHook(() => useCustomHook());
    
    act(() => {
      result.current.updateValue(newValue);
    });
    
    expect(result.current.value).toBe(newValue);
  });
});
```

## Testing Hooks with Dependencies

```javascript
it('refetches on dependency change', () => {
  const { result, rerender } = renderHook(
    ({ id }) => useUser(id),
    { initialProps: { id: 1 } }
  );

  expect(result.current.user.id).toBe(1);

  rerender({ id: 2 });

  expect(result.current.user.id).toBe(2);
});
```

## Testing Async Hooks

```javascript
it('fetches data asynchronously', async () => {
  const { result } = renderHook(() => useAsyncData());

  expect(result.current.loading).toBe(true);

  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });

  expect(result.current.data).toBeDefined();
});
```
```

### 3. Mock Generator

**.claude/agents/mock-generator.md:**

```markdown
---
name: mock-generator
description: |
  Generates realistic mocks and fixtures for tests.
  Triggers for "create mock", "generate test data".
tools: [Read, Write]
model: sonnet
---

# Mock Data Generator

Create realistic mocks for React tests.

## MSW Handlers

```javascript
// src/mocks/handlers.js
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({
      id: params.id,
      name: 'Test User',
      email: 'test@example.com'
    });
  }),

  http.post('/api/users', async ({ request }) => {
    const data = await request.json();
    return HttpResponse.json(
      { id: '123', ...data },
      { status: 201 }
    );
  }),

  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: 'User 1' },
      { id: '2', name: 'User 2' }
    ]);
  })
];
```

## Test Fixtures

```javascript
// src/test-utils/fixtures.js
export const mockUser = {
  id: '1',
  name: 'Test User',
  email: 'test@example.com',
  role: 'admin'
};

export const mockUsers = [
  mockUser,
  { id: '2', name: 'User 2', email: 'user2@example.com', role: 'user' }
];

export const mockPost = {
  id: '1',
  title: 'Test Post',
  content: 'Test content',
  author: mockUser
};
```

## Component Mocks

```javascript
// __mocks__/MyComponent.tsx
export const MyComponent = ({ children, ...props }) => (
  <div data-testid="my-component-mock" {...props}>
    {children}
  </div>
);
```
~~~

## 커스텀 명령어

**.claude/commands/test.md:**

```markdown
# /test

Generate comprehensive React component tests.

Usage: `/test ComponentName`

## Workflow

1. Read component file: `src/components/$ARGUMENTS.tsx`
2. Analyze:
   - Component type (presentational/container)
   - Props and their types
   - User interactions
   - Conditional rendering
   - API calls
3. Generate test file: `src/components/$ARGUMENTS.test.tsx`
4. Run tests: `npm test $ARGUMENTS.test`
5. Fix any failures
6. Report coverage

## Test Requirements

- Use React Testing Library
- Follow SKILL.md standards
- Test all user interactions
- Mock API calls with MSW
- Aim for 80%+ coverage
- No act() warnings
- All tests must pass

## Output

Report:
- Number of tests created
- Coverage percentage
- Any issues found
- Suggestions for improvement
```

## 실전 1: Button 컴포넌트 테스트

### 컴포넌트

```typescript
// src/components/Button/Button.tsx
import { ButtonHTMLAttributes } from 'react';
import './Button.css';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  loading?: boolean;
  icon?: React.ReactNode;
}

export const Button = ({
  children,
  variant = 'primary',
  size = 'medium',
  loading = false,
  icon,
  disabled,
  ...props
}: ButtonProps) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      {...props}
    >
      {loading && <span className="spinner" />}
      {icon && <span className="icon">{icon}</span>}
      {children}
    </button>
  );
};
```

### Claude Code 실행

```bash
$ claude
> Button 컴포넌트의 테스트를 작성해줘.
> React Testing Library 사용하고,
> 모든 props와 상태를 테스트해줘.

⏺ Spawning react-component-tester agent...

# Agent가 컴포넌트 분석
⏺ Reading Button.tsx...
⏺ Analyzing component:
  - Type: Presentational
  - Props: variant, size, loading, icon, disabled, children
  - Interactions: click
  - Conditional: loading spinner, icon

⏺ Generating comprehensive tests...
```

### 생성된 테스트

```typescript
// src/components/Button/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  describe('Rendering', () => {
    it('renders with children', () => {
      render(<Button>Click me</Button>);
      
      expect(screen.getByRole('button', { name: /click me/i }))
        .toBeInTheDocument();
    });

    it('renders with default props', () => {
      render(<Button>Default</Button>);
      
      const button = screen.getByRole('button');
      expect(button).toHaveClass('btn-primary');
      expect(button).toHaveClass('btn-medium');
    });

    it('applies variant class', () => {
      render(<Button variant="danger">Delete</Button>);
      
      expect(screen.getByRole('button')).toHaveClass('btn-danger');
    });

    it('applies size class', () => {
      render(<Button size="large">Large</Button>);
      
      expect(screen.getByRole('button')).toHaveClass('btn-large');
    });

    it('renders with icon', () => {
      const icon = <span data-testid="test-icon">🔥</span>;
      render(<Button icon={icon}>With Icon</Button>);
      
      expect(screen.getByTestId('test-icon')).toBeInTheDocument();
      expect(screen.getByRole('button')).toHaveTextContent('With Icon');
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

    it('does not trigger onClick when loading', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button loading onClick={handleClick}>Loading</Button>);
      
      await user.click(screen.getByRole('button'));
      
      expect(handleClick).not.toHaveBeenCalled();
    });
  });

  describe('Disabled State', () => {
    it('disables button when disabled prop is true', () => {
      render(<Button disabled>Disabled</Button>);
      
      expect(screen.getByRole('button')).toBeDisabled();
    });

    it('does not trigger onClick when disabled', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button disabled onClick={handleClick}>Disabled</Button>);
      
      await user.click(screen.getByRole('button'));
      
      expect(handleClick).not.toHaveBeenCalled();
    });
  });

  describe('User Interactions', () => {
    it('calls onClick when clicked', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button onClick={handleClick}>Click me</Button>);
      
      await user.click(screen.getByRole('button'));
      
      expect(handleClick).toHaveBeenCalledTimes(1);
    });

    it('calls onClick multiple times', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button onClick={handleClick}>Click me</Button>);
      
      await user.tripleClick(screen.getByRole('button'));
      
      expect(handleClick).toHaveBeenCalledTimes(3);
    });
  });

  describe('Accessibility', () => {
    it('is keyboard accessible', async () => {
      const handleClick = jest.fn();
      const user = userEvent.setup();
      
      render(<Button onClick={handleClick}>Accessible</Button>);
      
      const button = screen.getByRole('button');
      button.focus();
      
      await user.keyboard('{Enter}');
      expect(handleClick).toHaveBeenCalled();
    });

    it('has correct ARIA attributes when disabled', () => {
      render(<Button disabled>Disabled</Button>);
      
      const button = screen.getByRole('button');
      expect(button).toHaveAttribute('disabled');
    });
  });

  describe('HTML Attributes', () => {
    it('spreads additional props', () => {
      render(<Button data-testid="custom-button" id="my-button">Button</Button>);
      
      const button = screen.getByRole('button');
      expect(button).toHaveAttribute('data-testid', 'custom-button');
      expect(button).toHaveAttribute('id', 'my-button');
    });

    it('supports type attribute', () => {
      render(<Button type="submit">Submit</Button>);
      
      expect(screen.getByRole('button')).toHaveAttribute('type', 'submit');
    });
  });
});

⏺ Running tests...
⏺ Bash(npm test Button.test)

 PASS  src/components/Button/Button.test.tsx
  Button
    Rendering
      ✓ renders with children
      ✓ renders with default props
      ✓ applies variant class
      ✓ applies size class
      ✓ renders with icon
    Loading State
      ✓ shows spinner when loading
      ✓ disables button when loading
      ✓ does not trigger onClick when loading
    Disabled State
      ✓ disables button when disabled prop is true
      ✓ does not trigger onClick when disabled
    User Interactions
      ✓ calls onClick when clicked
      ✓ calls onClick multiple times
    Accessibility
      ✓ is keyboard accessible
      ✓ has correct ARIA attributes when disabled
    HTML Attributes
      ✓ spreads additional props
      ✓ supports type attribute

Test Suites: 1 passed, 1 total
Tests:       16 passed, 16 total
Coverage:    95.2% lines, 92.5% branches

✅ All tests passed
✅ Coverage threshold met (95.2% > 80%)
```

## 실전 2: LoginForm 테스트 (TDD 방식)

### Claude Code 요청

```bash
$ claude
> TDD 방식으로 LoginForm 컴포넌트를 개발해줘:
> 
> 요구사항:
> - email, password 입력
> - 유효성 검사 (이메일 형식, 비밀번호 최소 8자)
> - 제출 시 API 호출
> - 로딩/에러 상태
> 
> 프로세스:
> 1. 실패하는 테스트 작성 (Red)
> 2. 최소 구현 (Green)
> 3. 리팩토링
> 
> 서브 에이전트 활용해서 병렬로 처리해줘.
```

### Claude Code 작동

```
⏺ Starting TDD workflow with 3 sub-agents:
  1. test-writer: Write failing tests
  2. code-writer: Minimal implementation
  3. refactorer: Clean up code

⏺ Phase 1: RED - Writing failing tests...
```

**Sub-agent 1: Test Writer**

```typescript
// src/components/LoginForm/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  describe('Rendering', () => {
    it('renders email and password fields', () => {
      render(<LoginForm onSubmit={jest.fn()} />);
      
      expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
      expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    });

    it('renders submit button', () => {
      render(<LoginForm onSubmit={jest.fn()} />);
      
      expect(screen.getByRole('button', { name: /sign in/i }))
        .toBeInTheDocument();
    });
  });

  describe('Validation', () => {
    it('shows error for invalid email format', async () => {
      const user = userEvent.setup();
      render(<LoginForm onSubmit={jest.fn()} />);
      
      await user.type(screen.getByLabelText(/email/i), 'invalid-email');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(await screen.findByText(/invalid email/i)).toBeInTheDocument();
    });

    it('shows error for short password', async () => {
      const user = userEvent.setup();
      render(<LoginForm onSubmit=jest.fn()} />);
      
      await user.type(screen.getByLabelText(/password/i), '123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(await screen.findByText(/password must be at least 8 characters/i))
        .toBeInTheDocument();
    });

    it('does not show errors for valid input', async () => {
      const user = userEvent.setup();
      render(<LoginForm onSubmit={jest.fn()} />);
      
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      
      expect(screen.queryByText(/invalid/i)).not.toBeInTheDocument();
    });
  });

  describe('Submission', () => {
    it('calls onSubmit with form data', async () => {
      const handleSubmit = jest.fn();
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      await waitFor(() => {
        expect(handleSubmit).toHaveBeenCalledWith({
          email: 'user@example.com',
          password: 'password123'
        });
      });
    });

    it('prevents submission with invalid data', async () => {
      const handleSubmit = jest.fn();
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      await user.type(screen.getByLabelText(/email/i), 'invalid');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(handleSubmit).not.toHaveBeenCalled();
    });
  });

  describe('Loading State', () => {
    it('disables form during submission', async () => {
      const handleSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 100)));
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(screen.getByRole('button')).toBeDisabled();
      expect(screen.getByLabelText(/email/i)).toBeDisabled();
      expect(screen.getByLabelText(/password/i)).toBeDisabled();
    });

    it('shows loading indicator', async () => {
      const handleSubmit = jest.fn(() => new Promise(resolve => setTimeout(resolve, 100)));
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(screen.getByText(/signing in/i)).toBeInTheDocument();
    });
  });

  describe('Error Handling', () => {
    it('shows error message on submission failure', async () => {
      const handleSubmit = jest.fn(() => Promise.reject(new Error('Invalid credentials')));
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(await screen.findByText(/invalid credentials/i)).toBeInTheDocument();
    });

    it('allows retry after error', async () => {
      let callCount = 0;
      const handleSubmit = jest.fn(() => {
        callCount++;
        if (callCount === 1) return Promise.reject(new Error('Error'));
        return Promise.resolve();
      });
      const user = userEvent.setup();
      
      render(<LoginForm onSubmit={handleSubmit} />);
      
      // First attempt (fails)
      await user.type(screen.getByLabelText(/email/i), 'user@example.com');
      await user.type(screen.getByLabelText(/password/i), 'password123');
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      await screen.findByText(/error/i);
      
      // Second attempt (succeeds)
      await user.click(screen.getByRole('button', { name: /sign in/i }));
      
      expect(handleSubmit).toHaveBeenCalledTimes(2);
    });
  });
});

⏺ Running tests...
❌ All 13 tests failed (component doesn't exist yet)

⏺ Phase 2: GREEN - Writing minimal implementation...
```

**Sub-agent 2: Code Writer**

```typescript
// src/components/LoginForm/LoginForm.tsx
import { useState, FormEvent } from 'react';

interface LoginFormProps {
  onSubmit: (data: { email: string; password: string }) => Promise<void>;
}

export const LoginForm = ({ onSubmit }: LoginFormProps) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<{ email?: string; password?: string }>({});
  const [loading, setLoading] = useState(false);
  const [submitError, setSubmitError] = useState('');

  const validateEmail = (email: string) => {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  };

  const validate = () => {
    const newErrors: { email?: string; password?: string } = {};

    if (!validateEmail(email)) {
      newErrors.email = 'Invalid email format';
    }

    if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    setSubmitError('');

    if (!validate()) {
      return;
    }

    setLoading(true);
    
    try {
      await onSubmit({ email, password });
    } catch (error) {
      setSubmitError((error as Error).message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          disabled={loading}
        />
        {errors.email && <span>{errors.email}</span>}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          disabled={loading}
        />
        {errors.password && <span>{errors.password}</span>}
      </div>

      {submitError && <div>{submitError}</div>}

      <button type="submit" disabled={loading}>
        {loading ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
};

⏺ Running tests again...
✅ All 13 tests passed

⏺ Phase 3: REFACTOR - Improving code quality...
```

**Sub-agent 3: Refactorer**

```typescript
// src/components/LoginForm/LoginForm.tsx (refactored)
import { useState, FormEvent, ChangeEvent } from 'react';
import { validateEmail } from '@/utils/validation';
import { Input } from '@/components/Input';
import { Button } from '@/components/Button';
import { ErrorMessage } from '@/components/ErrorMessage';
import './LoginForm.css';

interface LoginFormProps {
  onSubmit: (data: LoginFormData) => Promise<void>;
}

interface LoginFormData {
  email: string;
  password: string;
}

interface FormErrors {
  email?: string;
  password?: string;
}

const MIN_PASSWORD_LENGTH = 8;

export const LoginForm = ({ onSubmit }: LoginFormProps) => {
  const [formData, setFormData] = useState<LoginFormData>({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState<FormErrors>({});
  const [loading, setLoading] = useState(false);
  const [submitError, setSubmitError] = useState('');

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error when user types
    if (errors[name as keyof FormErrors]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  };

  const validate = (): boolean => {
    const newErrors: FormErrors = {};

    if (!validateEmail(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (formData.password.length < MIN_PASSWORD_LENGTH) {
      newErrors.password = `Password must be at least ${MIN_PASSWORD_LENGTH} characters`;
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    setSubmitError('');

    if (!validate()) return;

    setLoading(true);
    
    try {
      await onSubmit(formData);
    } catch (error) {
      setSubmitError((error as Error).message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
      <Input
        label="Email"
        id="email"
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        error={errors.email}
        disabled={loading}
      />

      <Input
        label="Password"
        id="password"
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
        error={errors.password}
        disabled={loading}
      />

      {submitError && <ErrorMessage message={submitError} />}

      <Button type="submit" loading={loading} fullWidth>
        Sign In
      </Button>
    </form>
  );
};

⏺ Running tests final time...
✅ All 13 tests still passed
✅ Code coverage: 96.8%
✅ No ESLint warnings
✅ No act() warnings

⏺ TDD cycle completed successfully
```

**결과 요약:**
```
✅ TDD Workflow Completed

Phase 1 (RED): 13 failing tests written
Phase 2 (GREEN): Minimal implementation → all tests pass
Phase 3 (REFACTOR): Code quality improved, tests still pass

Files Created:
- LoginForm.tsx (95 lines)
- LoginForm.test.tsx (180 lines)
- LoginForm.css (25 lines)

Test Results:
- 13 tests, all passing
- Coverage: 96.8% lines, 95% branches
- Execution time: 1.2s

Next Steps:
- Add to Storybook?
- Create integration test?
- Add to design system docs?
```

## 실전 3: Custom Hook 테스트

### useAuth Hook

```typescript
// src/hooks/useAuth.ts
import { useState, useEffect } from 'react';
import { authAPI } from '@/api/auth';

export const useAuth = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const userData = await authAPI.me();
      setUser(userData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email, password) => {
    setLoading(true);
    try {
      const userData = await authAPI.login(email, password);
      setUser(userData);
      return userData;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  const logout = async () => {
    await authAPI.logout();
    setUser(null);
  };

  return { user, loading, error, login, logout };
};
```

### Claude Code 요청

```bash
$ claude
> useAuth 훅의 테스트를 작성해줘.
> renderHook 사용하고 MSW로 API 모킹해줘.

⏺ Spawning react-hook-tester...
```

### 생성된 테스트

```typescript
// src/hooks/useAuth.test.ts
import { renderHook, waitFor, act } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { useAuth } from './useAuth';

const server = setupServer(
  http.get('/api/auth/me', () => {
    return HttpResponse.json({
      id: '1',
      name: 'Test User',
      email: 'test@example.com'
    });
  }),

  http.post('/api/auth/login', async ({ request }) => {
    const { email, password } = await request.json();
    
    if (email === 'test@example.com' && password === 'password123') {
      return HttpResponse.json({
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
        token: 'fake-jwt-token'
      });
    }
    
    return HttpResponse.json(
      { message: 'Invalid credentials' },
      { status: 401 }
    );
  }),

  http.post('/api/auth/logout', () => {
    return new HttpResponse(null, { status: 204 });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('useAuth', () => {
  describe('Initialization', () => {
    it('starts with loading state', () => {
      const { result } = renderHook(() => useAuth());
      
      expect(result.current.loading).toBe(true);
      expect(result.current.user).toBeNull();
    });

    it('loads user on mount', async () => {
      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      expect(result.current.user).toEqual({
        id: '1',
        name: 'Test User',
        email: 'test@example.com'
      });
    });

    it('handles auth check error', async () => {
      server.use(
        http.get('/api/auth/me', () => {
          return HttpResponse.json(
            { message: 'Unauthorized' },
            { status: 401 }
          );
        })
      );

      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      expect(result.current.user).toBeNull();
      expect(result.current.error).toBeDefined();
    });
  });

  describe('Login', () => {
    it('logs in successfully', async () => {
      const { result } = renderHook(() => useAuth());

      // Wait for initial load
      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      // Perform login
      await act(async () => {
        await result.current.login('test@example.com', 'password123');
      });

      expect(result.current.user).toEqual({
        id: '1',
        name: 'Test User',
        email: 'test@example.com',
        token: 'fake-jwt-token'
      });
      expect(result.current.error).toBeNull();
    });

    it('handles login error', async () => {
      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      await expect(
        act(async () => {
          await result.current.login('wrong@example.com', 'wrongpass');
        })
      ).rejects.toThrow();

      expect(result.current.user).toBeNull();
      expect(result.current.error).toBeDefined();
    });

    it('sets loading during login', async () => {
      const { result } = renderHook(() => useAuth());

      await waitFor(() => {
        expect(result.current.loading).toBe(false);
      });

      let loginPromise;
      act(() => {
        loginPromise = result.current.login('test@example.com', 'password123');
      });

      expect(result.current.loading).toBe(true);

      await act(async () => {
        await loginPromise;
      });

      expect(result.current.loading).toBe(false);
    });
  });

  describe('Logout', () => {
    it('logs out successfully', async () => {
      const { result } = renderHook(() => useAuth());

      // Wait for initial load (user loaded)
      await waitFor(() => {
        expect(result.current.user).toBeDefined();
      });

      // Logout
      await act(async () => {
        await result.current.logout();
      });

      expect(result.current.user).toBeNull();
    });
  });
});

⏺ Running tests...
✅ All 8 tests passed
✅ Coverage: 94.3%
```

## 베스트 프랙티스

### 1. 올바른 쿼리 우선순위

```javascript
// ✅ 최선: Accessible role
screen.getByRole('button', { name: /submit/i })

// ✅ 좋음: Label
screen.getByLabelText(/email/i)

// ⚠️ 괜찮음: Placeholder (label 없을 때만)
screen.getByPlaceholderText(/search/i)

// ⚠️ 최후: Test ID
screen.getByTestId('submit-button')

// ❌ 나쁨: querySelector
container.querySelector('.submit-button')
```

### 2. 비동기 처리

```javascript
// ✅ 권장: findBy (자동 waitFor)
const element = await screen.findByText(/loaded/i);

// ✅ 좋음: waitFor 
await waitFor(() => {
  expect(screen.getByText(/loaded/i)).toBeInTheDocument();
});

// ❌ 나쁨: setTimeout
setTimeout(() => {
  expect(screen.getByText(/loaded/i)).toBeInTheDocument();
}, 1000);
```

### 3. 사용자 이벤트

```javascript
// ✅ 최선: userEvent.setup()
const user = userEvent.setup();
await user.click(button);
await user.type(input, 'text');

// ⚠️ 괜찮음: fireEvent (간단한 경우)
fireEvent.click(button);

// ❌ 나쁨: 직접 핸들러 호출
props.onClick();
```

### 4. Mock 전략

```javascript
// ✅ API: MSW
server.use(
  http.get('/api/users', () => HttpResponse.json([]))
);

// ✅ 함수: jest.fn()
const mockFn = jest.fn();

// ⚠️ 모듈: 필요시만
jest.mock('./api');

// ❌ React 내부
jest.mock('react');
```

## CI/CD 통합

### GitHub Actions

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
      
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80%"
            exit 1
          fi
```

## 결론

Claude Code는 **React 개발자를 위한 테스트 자동화 시스템**입니다.

**핵심 가치:**
1. **40배 빠른 테스트 작성**: 2시간 → 3분
2. **서브 에이전트 활용**: 병렬 처리로 효율 극대화
3. **TDD 자동화**: Red-Green-Refactor 자동 순환
4. **80%+ 커버리지**: 자동으로 높은 품질 달성
5. **Best Practices**: RTL 패턴 자동 적용

**시작하기:**
```bash
$ brew install anthropic/claude/claude-code
$ cd my-react-app
$ claude

> UserProfile 컴포넌트 테스트 작성해줘
```

테스트는 이제 "작성하는 것"이 아니라 "요청하는 것"입니다!

---

**문서 작성 일자: 2026-01-22**
