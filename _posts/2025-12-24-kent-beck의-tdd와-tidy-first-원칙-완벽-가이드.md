---
title: "Kent Beck의 TDD와 Tidy First 원칙 완벽 가이드"
date: 2025-12-24 19:00:00 +0900
categories: [AI,  Vibe Coding]
mermaid: [True]
tags: [AI,  vibe-coding,  KentBeck,  TDD,  tidy-first,  Guide,  Claude.write]
---

## 들어가며

Kent Beck은 소프트웨어 개발 방법론의 선구자입니다. 그가 만든 **Test-Driven Development(TDD)** 와 **Tidy First** 원칙은 단순히 코드를 작성하는 방법이 아니라, 코드의 품질과 유지보수성을 근본적으로 향상시키는 철학입니다.

이 문서는 "구조/기능을 절대 섞지 않는다"는 규칙이 정확히 무엇을 의미하는지, 왜 중요한지, 그리고 실제로 어떻게 적용하는지를 처음부터 끝까지 상세하게 설명합니다.

---

## 목차

1. [TDD (Test-Driven Development)란?](#tdd-test-driven-development란)
2. [Tidy First 원칙이란?](#tidy-first-원칙이란)
3. [구조 변경 vs 기능 변경](#구조-변경-vs-기능-변경)
4. [왜 절대 섞으면 안 되는가?](#왜-절대-섞으면-안-되는가)
5. [실전 예시: Todo 앱 개발](#실전-예시-todo-앱-개발)
6. [커밋 전략 상세 가이드](#커밋-전략-상세-가이드)
7. [Red-Green-Refactor 사이클](#red-green-refactor-사이클)
8. [실무 적용 시나리오](#실무-적용-시나리오)
9. [자주 묻는 질문](#자주-묻는-질문)
10. [연습 문제](#연습-문제)

---

## TDD (Test-Driven Development)란?

### 기본 개념

TDD는 "테스트 주도 개발"이라는 의미입니다. 코드를 먼저 작성하는 것이 아니라, **테스트를 먼저 작성**합니다.

전통적인 개발 방식:
```
1. 기능 구현
2. 테스트 작성
3. 버그 발견
4. 수정
5. 다시 테스트
```

TDD 방식:
```
1. 테스트 작성 (실패하는 테스트)
2. 최소한의 코드로 테스트 통과
3. 코드 개선
4. 다음 테스트로 이동
```

### 왜 테스트를 먼저 작성할까?

일반적으로 우리는 이렇게 생각합니다:
- "기능을 먼저 만들고, 잘 돌아가면 테스트를 작성하자"
- "테스트는 나중에 시간 나면 추가하자"

하지만 현실은:
- 나중에 시간이 안 남
- 테스트 작성이 귀찮아짐
- 기능은 동작하는데 왜 테스트를 작성해야 하는지 의문
- 결국 테스트 없이 개발

TDD는 이 순서를 뒤집습니다. 테스트를 먼저 작성하면:
- **요구사항이 명확해집니다** - "이 함수는 정확히 무엇을 해야 하는가?"
- **설계가 개선됩니다** - "이 함수를 어떻게 사용하면 편할까?"
- **버그가 줄어듭니다** - "모든 경우를 테스트했으니 안전하다"
- **리팩토링이 안전해집니다** - "코드를 바꿔도 테스트가 보장해줌"

### TDD의 세 가지 법칙

Kent Beck이 정의한 TDD의 법칙:

**법칙 1: 실패하는 테스트 없이는 제품 코드를 작성하지 않는다**
- 먼저 "이것이 동작해야 한다"는 테스트를 작성
- 당연히 테스트는 실패함 (코드가 아직 없으니까)

**법칙 2: 실패를 증명하는 데 충분한 만큼만 테스트를 작성한다**
- 거대한 테스트를 작성하지 않음
- 하나의 작은 기능만 테스트

**법칙 3: 테스트를 통과시키는 데 충분한 만큼만 코드를 작성한다**
- 완벽한 코드를 작성하지 않음
- 테스트만 통과하면 OK
- 나중에 리팩토링으로 개선

### 구체적인 예시

예를 들어, "두 숫자를 더하는 함수"를 만든다고 가정합시다.

**전통적인 방식:**
```typescript
// 1. 함수부터 구현
function add(a: number, b: number): number {
  return a + b;
}

// 2. 나중에 테스트 작성 (아마도 안 함)
```

**TDD 방식:**
```typescript
// 1. 테스트 먼저 작성
test('should add two numbers', () => {
  expect(add(2, 3)).toBe(5);
});

// 2. 함수 구현 (테스트를 통과시키기 위해)
function add(a: number, b: number): number {
  return a + b;
}
```

단순해 보이지만, 이 순서의 차이가 엄청난 결과를 만듭니다.

---

## Tidy First 원칙이란?

### 기본 개념

"Tidy First"는 직역하면 "먼저 정리하라"입니다. Kent Beck의 책 제목이기도 합니다.

핵심 아이디어: **코드 구조를 개선하는 것과 기능을 추가하는 것을 절대 섞지 마라**

### 농사 비유로 이해하기

Kent Beck은 "씨앗 옥수수" 비유를 사용합니다.

농부가 옥수수를 수확할 때:
- 대부분은 먹거나 팔기 위해 수확
- 하지만 일부는 **다음 시즌을 위한 씨앗**으로 남겨둠
- 만약 씨앗까지 다 먹어버리면? → 다음 시즌에 심을 게 없음

소프트웨어 개발에서:
- "수확하는 것" = 새로운 기능 추가
- "씨앗 남기기" = 코드 구조 유지/개선
- 씨앗까지 먹어버림 = 구조 무시하고 기능만 추가

**결과:**
- 단기적으로는 빠름 (씨앗까지 다 먹으니까)
- 장기적으로는 재앙 (다음 시즌에 심을 게 없음)

### 두 가지 종류의 변경

모든 코드 변경은 두 가지 중 하나입니다:

**1. 구조적 변경 (Structural Change)**
- 코드의 **동작**은 바꾸지 않음
- 코드의 **구조**만 개선
- 예시: 변수 이름 바꾸기, 함수 추출하기, 파일 이동하기

**2. 기능적 변경 (Behavioral Change)**
- 실제로 **동작**이 바뀜
- 새로운 기능 추가, 버그 수정 등
- 예시: 새로운 버튼 추가, 검증 로직 추가

### Tidy First의 핵심 규칙

**규칙 1: 구조와 기능을 절대 섞지 않는다**
- 하나의 커밋에는 구조 변경만, 또는 기능 변경만
- 절대 둘을 함께 넣지 않음

**규칙 2: 구조 변경을 먼저 한다 (Tidy First)**
- 기능을 추가하기 전에
- 먼저 코드 구조를 정리
- 그 다음 기능 추가

**규칙 3: 각 변경 후 테스트를 실행한다**
- 구조 변경 후 → 테스트 실행 → 통과 확인
- 기능 변경 후 → 테스트 실행 → 통과 확인

### 왜 이것이 중요한가?

코드는 두 가지 목적을 가집니다:
1. **지금 동작하게 만들기** (기능)
2. **나중에 쉽게 바꿀 수 있게 하기** (구조)

대부분의 개발자는 1번만 신경 씁니다. Tidy First는 2번의 중요성을 강조합니다.

---

## 구조 변경 vs 기능 변경

### 구조 변경 (Structural Change)의 예시

**동작은 그대로, 코드만 개선:**

**예시 1: 변수 이름 바꾸기**
```typescript
// BEFORE
function calc(x: number, y: number): number {
  return x + y;
}

// AFTER
function calculateTotal(price: number, tax: number): number {
  return price + tax;
}
```
- 동작은 완전히 동일
- 이름만 명확해짐
- 이것은 **구조 변경**

**예시 2: 함수 추출하기**
```typescript
// BEFORE
function processOrder(order: Order) {
  // 검증 로직
  if (!order.customerId) throw new Error('Invalid customer');
  if (order.items.length === 0) throw new Error('Empty order');
  
  // 가격 계산
  const total = order.items.reduce((sum, item) => sum + item.price, 0);
  
  // 저장
  database.save(order);
}

// AFTER - 구조 개선
function processOrder(order: Order) {
  validateOrder(order);
  const total = calculateTotal(order);
  saveOrder(order);
}

function validateOrder(order: Order) {
  if (!order.customerId) throw new Error('Invalid customer');
  if (order.items.length === 0) throw new Error('Empty order');
}

function calculateTotal(order: Order): number {
  return order.items.reduce((sum, item) => sum + item.price, 0);
}

function saveOrder(order: Order) {
  database.save(order);
}
```
- 동작은 완전히 동일
- 코드가 읽기 쉬워짐
- 이것은 **구조 변경**

**예시 3: 파일 이동**
```
// BEFORE
src/
  components/
    TodoInput.tsx
    TodoList.tsx
    Stats.tsx
    FilterButtons.tsx

// AFTER
src/
  components/
    todo/
      TodoInput.tsx
      TodoList.tsx
    stats/
      Stats.tsx
    filters/
      FilterButtons.tsx
```
- 기능은 그대로
- 파일 구조만 정리
- 이것은 **구조 변경**

**예시 4: 중복 코드 제거**
```typescript
// BEFORE
function formatUserName(user: User): string {
  return user.firstName + ' ' + user.lastName;
}

function displayUserName(user: User): string {
  return user.firstName + ' ' + user.lastName;
}

// AFTER
function formatUserName(user: User): string {
  return user.firstName + ' ' + user.lastName;
}

function displayUserName(user: User): string {
  return formatUserName(user);
}
```
- 동작은 동일
- 중복 제거
- 이것은 **구조 변경**

### 기능 변경 (Behavioral Change)의 예시

**실제 동작이 바뀜:**

**예시 1: 새로운 기능 추가**
```typescript
// BEFORE - 할 일만 추가 가능
function addTodo(text: string) {
  todos.push({ text, completed: false });
}

// AFTER - 우선순위 추가
function addTodo(text: string, priority: 'high' | 'medium' | 'low') {
  todos.push({ text, completed: false, priority });
}
```
- 기능이 추가됨
- 이것은 **기능 변경**

**예시 2: 검증 로직 추가**
```typescript
// BEFORE - 검증 없음
function addTodo(text: string) {
  todos.push({ text });
}

// AFTER - 검증 추가
function addTodo(text: string) {
  if (text.trim() === '') {
    throw new Error('Todo cannot be empty');
  }
  if (text.length > 200) {
    throw new Error('Todo is too long');
  }
  todos.push({ text });
}
```
- 동작이 바뀜 (에러를 던지게 됨)
- 이것은 **기능 변경**

**예시 3: 버그 수정**
```typescript
// BEFORE - 버그 있음
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// AFTER - 버그 수정 (세금 포함)
function calculateTotal(items: Item[]): number {
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  const tax = subtotal * 0.1;
  return subtotal + tax;
}
```
- 계산 결과가 바뀜
- 이것은 **기능 변경**

### 헷갈리는 경우들

**케이스 1: 성능 개선**
```typescript
// BEFORE
function findUser(id: string): User | undefined {
  return users.find(u => u.id === id); // O(n)
}

// AFTER
const userMap = new Map(users.map(u => [u.id, u]));
function findUser(id: string): User | undefined {
  return userMap.get(id); // O(1)
}
```
**질문:** 이것은 구조 변경? 기능 변경?
**답:** **구조 변경**
- 결과는 동일 (같은 User 반환)
- 내부 구현만 바뀜
- 성능 개선은 구조 변경

**케이스 2: 에러 메시지 개선**
```typescript
// BEFORE
throw new Error('Error');

// AFTER
throw new Error('Todo text cannot be empty');
```
**질문:** 이것은 구조 변경? 기능 변경?
**답:** **기능 변경**
- 에러 메시지가 바뀌면 사용자에게 보이는 것이 바뀜
- 동작이 변경된 것

**케이스 3: 타입 추가**
```typescript
// BEFORE
function add(a, b) {
  return a + b;
}

// AFTER
function add(a: number, b: number): number {
  return a + b;
}
```
**질문:** 이것은 구조 변경? 기능 변경?
**답:** **구조 변경**
- 런타임 동작은 동일
- 타입은 컴파일 타임에만 영향
- 단, 타입 오류를 강제하면 기능 변경

---

## 왜 절대 섞으면 안 되는가?

### 문제 상황: 섞었을 때

실제 개발에서 이런 일이 자주 발생합니다:

```typescript
// 한 커밋에서 여러 가지를 동시에 함

// 1. 변수 이름 변경 (구조)
// 2. 함수 추출 (구조)
// 3. 새로운 검증 로직 추가 (기능)
// 4. 버그 수정 (기능)

// Commit: "Update todo validation and refactor code"
```

**이렇게 하면 무슨 문제가?**

**문제 1: 버그가 생겼을 때 원인 파악 어려움**
- 버그가 발생했다고 가정
- 이 커밋을 되돌려야 하나?
- 근데 구조 개선도 함께 되돌려지는데...
- 어느 부분 때문에 버그가 생긴 거지?

**문제 2: 코드 리뷰가 어려움**
- 리뷰어: "이 변경은 왜 한 거죠?"
- 개발자: "음... 변수 이름 바꾸다가 버그를 발견해서 수정하고, 그러다 보니 검증도 추가하고..."
- 리뷰어: "그래서 정확히 무엇을 테스트해야 하죠?"

**문제 3: 테스트 실패 시 문제 위치 불명확**
- 테스트가 실패함
- 구조 변경 때문? 기능 변경 때문?
- 전체를 다시 봐야 함

**문제 4: 협업이 어려움**
- 다른 팀원이 같은 파일 수정 중
- Merge conflict 발생
- 어느 부분이 중요한 변경인지 알 수 없음

### 해결책: 완전히 분리

**올바른 방법:**

```bash
# Commit 1: [STRUCTURAL] Extract validation logic
# - validateTodoText 함수 추출
# - 테스트 실행 → 통과

# Commit 2: [STRUCTURAL] Rename variables for clarity  
# - text → todoText
# - 테스트 실행 → 통과

# Commit 3: [BEHAVIORAL] Add length validation
# - 200자 제한 추가
# - 테스트 추가 및 통과

# Commit 4: [FIX] Fix empty string validation bug
# - trim() 추가
# - 테스트 추가 및 통과
```

**이렇게 하면:**
- 각 커밋의 목적이 명확
- 버그 발생 시 정확히 어느 커밋인지 알 수 있음
- 코드 리뷰가 쉬움 (각 커밋을 독립적으로 리뷰)
- 필요 시 특정 커밋만 되돌리기 가능

### 실제 사례: 버그 추적

**섞었을 때:**
```bash
# Commit: "Refactor validation and add new features"
# 파일 10개 변경, +500줄 -300줄

# 일주일 후 버그 발견
# 이 커밋 때문인가?
# 500줄을 다 확인해야 함
```

**분리했을 때:**
```bash
# Commit 1: [STRUCTURAL] Extract validation functions
# Commit 2: [STRUCTURAL] Rename for clarity
# Commit 3: [BEHAVIORAL] Add email validation
# Commit 4: [BEHAVIORAL] Add password strength check

# 일주일 후 버그 발견
# git bisect로 빠르게 찾기
# Commit 4가 문제!
# 30줄만 확인하면 됨
```

---

## 실전 예시: Todo 앱 개발

### 시나리오: 할 일 삭제 기능 추가

우리는 Todo 앱에 "삭제" 기능을 추가하려고 합니다. 코드를 보니 구조가 복잡하고 중복도 많습니다.

**잘못된 방법 (섞어서 하기):**

```typescript
// ❌ 한 번에 모두 변경
function TodoList({ todos, onToggle, onDelete }: Props) {
  // 1. 변수 이름 변경 (구조)
  const todoItems = todos; // 기존: items
  
  // 2. 함수 추출 (구조)
  const renderTodo = (todo: Todo) => (
    <div key={todo.id}>
      <input 
        type="checkbox" 
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      {/* 3. 새 기능 추가 (기능) */}
      <button onClick={() => {
        if (confirm('정말 삭제하시겠습니까?')) {
          onDelete(todo.id);
        }
      }}>
        삭제
      </button>
    </div>
  );
  
  return <div>{todoItems.map(renderTodo)}</div>;
}

// Commit: "Add delete feature and refactor TodoList"
```

**이 방법의 문제점:**
- 구조 변경과 기능 변경이 섞임
- 버그가 생기면 어느 부분이 원인인지 불명확
- 코드 리뷰 어려움

**올바른 방법 (Tidy First):**

**Step 1: 구조 정리 먼저 ([STRUCTURAL])**

```typescript
// Commit 1: [STRUCTURAL] Rename items to todoItems for clarity

function TodoList({ todos, onToggle }: Props) {
  const todoItems = todos; // 이름만 변경
  
  return (
    <div>
      {todoItems.map(todo => (
        <div key={todo.id}>
          <input 
            type="checkbox" 
            checked={todo.completed}
            onChange={() => onToggle(todo.id)}
          />
          <span>{todo.text}</span>
        </div>
      ))}
    </div>
  );
}

// 테스트 실행 → 통과 확인
// 커밋!
```

**Step 2: 더 많은 구조 정리 ([STRUCTURAL])**

```typescript
// Commit 2: [STRUCTURAL] Extract TodoItem component

function TodoItem({ todo, onToggle }: TodoItemProps) {
  return (
    <div>
      <input 
        type="checkbox" 
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
    </div>
  );
}

function TodoList({ todos, onToggle }: Props) {
  return (
    <div>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
        />
      ))}
    </div>
  );
}

// 테스트 실행 → 통과 확인
// 커밋!
```

**Step 3: 이제 기능 추가 ([BEHAVIORAL])**

```typescript
// Commit 3: [BEHAVIORAL] Add delete button to TodoItem

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  return (
    <div>
      <input 
        type="checkbox" 
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>
        삭제
      </button>
    </div>
  );
}

// 새 테스트 작성:
// - 삭제 버튼이 렌더링되는지
// - 클릭 시 onDelete 호출되는지
// 테스트 실행 → 통과
// 커밋!
```

**Step 4: 확인 대화상자 추가 ([BEHAVIORAL])**

```typescript
// Commit 4: [BEHAVIORAL] Add confirmation dialog for delete

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  const handleDelete = () => {
    if (confirm('정말 삭제하시겠습니까?')) {
      onDelete(todo.id);
    }
  };
  
  return (
    <div>
      <input 
        type="checkbox" 
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={handleDelete}>
        삭제
      </button>
    </div>
  );
}

// 테스트 추가:
// - confirm이 호출되는지
// - 취소 시 onDelete가 호출 안 되는지
// 테스트 실행 → 통과
// 커밋!
```

**결과:**
- 4개의 명확한 커밋
- 각 커밋은 하나의 목적만
- 버그 발생 시 정확한 원인 파악 가능
- 코드 리뷰 쉬움

---

## 커밋 전략 상세 가이드

### 4가지 커밋 타입

Kent Beck 스타일에서는 커밋을 4가지로 분류합니다:

**1. [STRUCTURAL] - 구조 변경**
```bash
[STRUCTURAL] Extract validation logic into separate function
[STRUCTURAL] Rename getUserData to fetchUserProfile
[STRUCTURAL] Move utility functions to utils/ directory
[STRUCTURAL] Remove duplicate code in formatters
```

**특징:**
- 동작은 변하지 않음
- 코드 구조만 개선
- 반드시 테스트가 그대로 통과해야 함

**2. [BEHAVIORAL] - 기능 변경**
```bash
[BEHAVIORAL] Add todo deletion feature
[BEHAVIORAL] Implement user authentication
[BEHAVIORAL] Add email validation to signup form
```

**특징:**
- 새로운 기능 추가
- 사용자가 체감할 수 있는 변화
- 새로운 테스트 추가

**3. [FIX] - 버그 수정**
```bash
[FIX] Handle empty string in todo validation
[FIX] Prevent negative values in quantity input
[FIX] Fix memory leak in useEffect cleanup
```

**특징:**
- 잘못된 동작 수정
- 보통 새 테스트 추가 (버그를 재현하는 테스트)
- 기존 테스트 수정할 수도 있음

**4. [REFACTOR] - 리팩토링**
```bash
[REFACTOR] Simplify conditional logic in validator
[REFACTOR] Replace nested callbacks with async/await
[REFACTOR] Convert class component to functional component
```

**특징:**
- STRUCTURAL의 특수한 경우
- 동작 불변
- 코드를 "더 좋게" 만듦
- 리팩토링 = 구조 변경

### 커밋 메시지 작성 규칙

**좋은 커밋 메시지:**
```bash
[STRUCTURAL] Extract todo validation into validateTodoText function

- Moved validation logic from TodoInput component
- Created validateTodoText in utils/validation.ts
- All existing tests still pass
- No behavioral changes
```

**나쁜 커밋 메시지:**
```bash
Update code

- 무엇을 했는지 불명확
- 타입도 없음
- 이유도 없음
```

**커밋 메시지 템플릿:**
```bash
[TYPE] 한 줄 요약 (50자 이내)

- 변경 내용 상세 설명
- 왜 이 변경을 했는지
- 영향 받는 부분
- 테스트 결과

(선택) 관련 이슈: #123
```

### 커밋 타이밍

**언제 커밋해야 하는가?**

**STRUCTURAL 커밋:**
```bash
1. 구조 변경 완료
2. 테스트 실행
3. 모든 테스트 통과
4. 즉시 커밋
```

**BEHAVIORAL 커밋:**
```bash
1. 테스트 작성 (Red)
2. 최소 구현 (Green)
3. 테스트 통과 확인
4. 즉시 커밋
5. 리팩토링 필요하면 → STRUCTURAL 커밋으로
```

**자주 커밋하기:**
- 큰 커밋 1개 < 작은 커밋 10개
- 각 커밋은 의미 있는 단위
- "잠깐 커피 마시러 갈까?" → 커밋 먼저!

### 실전 커밋 시나리오

**시나리오: 로그인 기능 구현**

```bash
# 1단계: 기존 코드 정리
git commit -m "[STRUCTURAL] Extract form validation logic"

# 2단계: 더 정리
git commit -m "[STRUCTURAL] Create AuthService class"

# 3단계: 테스트 추가
git commit -m "[BEHAVIORAL] Add login form UI"

# 4단계: 기능 구현
git commit -m "[BEHAVIORAL] Implement login authentication"

# 5단계: 에러 처리
git commit -m "[BEHAVIORAL] Add error handling for login"

# 6단계: 코드 개선
git commit -m "[REFACTOR] Simplify error message display"

# 7단계: 버그 발견 및 수정
git commit -m "[FIX] Handle empty email field in login"
```

**결과:**
- 7개의 작은 커밋
- 각각 명확한 목적
- 쉬운 추적과 디버깅

---

## Red-Green-Refactor 사이클

### 사이클의 3단계

TDD의 핵심은 이 3단계를 반복하는 것입니다:

**🔴 RED: 실패하는 테스트 작성**
```typescript
// 테스트 먼저!
describe('validateTodoText', () => {
  it('should reject empty string', () => {
    const result = validateTodoText('');
    expect(result.isValid).toBe(false);
    expect(result.errorMessage).toBe('할 일을 입력해주세요');
  });
});

// 실행하면 실패!
// → validateTodoText 함수가 아직 없음
// → 에러: validateTodoText is not defined
```

**✅ GREEN: 테스트를 통과시키는 최소 코드**
```typescript
// 테스트만 통과시키기
function validateTodoText(text: string) {
  if (text === '') {
    return {
      isValid: false,
      errorMessage: '할 일을 입력해주세요'
    };
  }
  return { isValid: true };
}

// 테스트 실행 → 통과!
// 커밋: [BEHAVIORAL] Add empty string validation
```

**🔧 REFACTOR: 코드 개선**
```typescript
// 이제 코드를 더 좋게 만들기
function validateTodoText(text: string): ValidationResult {
  const trimmedText = text.trim();
  
  if (trimmedText === '') {
    return {
      isValid: false,
      errorMessage: '할 일을 입력해주세요'
    };
  }
  
  return { isValid: true };
}

// 테스트 실행 → 여전히 통과!
// 커밋: [STRUCTURAL] Add trim to validation
```

### 상세 흐름도

```
┌─────────────────────────────────────┐
│  기능 요구사항                        │
│  "빈 문자열은 거부해야 함"             │
└─────────────┬───────────────────────┘
              │
              ▼
         ┌─────────┐
         │   RED   │ 실패하는 테스트 작성
         └────┬────┘
              │ "테스트가 실패해야 함"
              │
              ▼
      [테스트 실행]
              │
              ▼
         실패! 😢
         (예상된 결과)
              │
              ▼
        ┌──────────┐
        │  GREEN   │ 최소 구현
        └────┬─────┘
             │ "테스트만 통과시키기"
             │
             ▼
      [테스트 실행]
             │
             ▼
         통과! 🎉
             │
             ▼
    ┌────────────┐
    │  REFACTOR  │ 코드 개선
    └────┬───────┘
         │ "더 좋은 코드로"
         │
         ▼
  [테스트 실행]
         │
         ▼
     여전히 통과! ✅
         │
         ▼
      [커밋]
         │
         ▼
    다음 기능으로!
```

### 실전 예시: 한 기능을 완성하기까지

**기능: Todo 텍스트 검증 (빈 문자열, 200자 제한)**

**Iteration 1: 빈 문자열 검증**

```typescript
// 🔴 RED
test('should reject empty string', () => {
  const result = validateTodoText('');
  expect(result.isValid).toBe(false);
});

// ✅ GREEN
function validateTodoText(text: string) {
  if (text === '') {
    return { isValid: false, errorMessage: '할 일을 입력해주세요' };
  }
  return { isValid: true };
}

// 커밋: [BEHAVIORAL] Add empty string validation

// 🔧 REFACTOR
function validateTodoText(text: string): ValidationResult {
  if (text.trim() === '') {
    return { 
      isValid: false, 
      errorMessage: '할 일을 입력해주세요' 
    };
  }
  return { isValid: true };
}

// 커밋: [STRUCTURAL] Add trim to handle whitespace
```

**Iteration 2: 200자 제한**

```typescript
// 🔴 RED
test('should reject text over 200 characters', () => {
  const longText = 'a'.repeat(201);
  const result = validateTodoText(longText);
  expect(result.isValid).toBe(false);
});

// ✅ GREEN
function validateTodoText(text: string): ValidationResult {
  const trimmedText = text.trim();
  
  if (trimmedText === '') {
    return { 
      isValid: false, 
      errorMessage: '할 일을 입력해주세요' 
    };
  }
  
  if (trimmedText.length > 200) {
    return {
      isValid: false,
      errorMessage: '200자 이내로 입력해주세요'
    };
  }
  
  return { isValid: true };
}

// 커밋: [BEHAVIORAL] Add 200 character limit

// 🔧 REFACTOR - 상수 추출
const MAX_TODO_LENGTH = 200;

function validateTodoText(text: string): ValidationResult {
  const trimmedText = text.trim();
  
  if (trimmedText === '') {
    return { 
      isValid: false, 
      errorMessage: '할 일을 입력해주세요' 
    };
  }
  
  if (trimmedText.length > MAX_TODO_LENGTH) {
    return {
      isValid: false,
      errorMessage: `${MAX_TODO_LENGTH}자 이내로 입력해주세요`
    };
  }
  
  return { isValid: true };
}

// 커밋: [STRUCTURAL] Extract MAX_TODO_LENGTH constant
```

**최종 결과:**
- 4개의 커밋
- 각 단계가 명확
- 안전한 리팩토링

---

## 실무 적용 시나리오

### 시나리오 1: 레거시 코드에 기능 추가

**상황:** 1년 된 코드에 새 기능을 추가해야 함

**잘못된 접근:**
```bash
# 코드가 엉망이지만 일단 기능부터 추가
git commit -m "Add new feature (코드 좀 수정함)"
```

**Tidy First 접근:**
```bash
# 1. 먼저 관련 코드 정리
git commit -m "[STRUCTURAL] Extract duplicated validation logic"

# 2. 더 정리
git commit -m "[STRUCTURAL] Rename confusing variable names"

# 3. 이제 기능 추가
git commit -m "[BEHAVIORAL] Add priority field to todos"

# 4. 테스트 추가
git commit -m "[BEHAVIORAL] Add tests for priority feature"
```

**결과:**
- 코드가 깨끗해짐
- 새 기능 추가가 쉬워짐
- 미래의 나(또는 동료)가 고마워함

### 시나리오 2: 버그 수정

**상황:** 프로덕션에서 버그 발견

**잘못된 접근:**
```bash
# 급하게 버그 수정하면서 코드도 정리
git commit -m "Fix bug and improve code"
```

**올바른 접근:**
```bash
# 1. 버그를 재현하는 테스트 작성
git commit -m "[BEHAVIORAL] Add failing test for empty todo bug"

# 2. 최소한으로 버그 수정
git commit -m "[FIX] Handle empty string in todo validation"

# 3. 테스트 확인 후, 관련 코드 정리
git commit -m "[STRUCTURAL] Extract validation logic for clarity"
```

**장점:**
- 버그 수정이 정확히 어디인지 명확
- 나중에 같은 버그 재발 방지 (테스트 있음)
- 코드 정리는 별도로 관리

### 시나리오 3: 대규모 리팩토링

**상황:** 컴포넌트 구조를 완전히 바꿔야 함

**단계별 접근:**

```bash
# Phase 1: 작은 구조 변경들
git commit -m "[STRUCTURAL] Extract TodoItem component"
git commit -m "[STRUCTURAL] Move validation to utils"
git commit -m "[STRUCTURAL] Rename props for consistency"

# Phase 2: 중간 크기 변경들
git commit -m "[STRUCTURAL] Split TodoList into separate files"
git commit -m "[STRUCTURAL] Create custom useTodos hook"

# Phase 3: 큰 변경
git commit -m "[STRUCTURAL] Reorganize component hierarchy"

# 모든 단계에서 테스트는 계속 통과!
```

**핵심:**
- 한 번에 하나씩
- 각 단계 후 테스트
- 작은 커밋들의 시리즈

### 시나리오 4: 새로운 기능 개발

**상황:** 완전히 새로운 기능 (필터링) 추가

**TDD + Tidy First 통합:**

```bash
# 1. 타입 정의
git commit -m "[BEHAVIORAL] Add FilterType to types"

# 2. 테스트 작성
git commit -m "[BEHAVIORAL] Add test for filterTodos utility"

# 3. 최소 구현
git commit -m "[BEHAVIORAL] Implement basic filterTodos"

# 4. 구조 개선
git commit -m "[STRUCTURAL] Extract filter logic to separate file"

# 5. UI 추가
git commit -m "[BEHAVIORAL] Add FilterButtons component"

# 6. 통합
git commit -m "[BEHAVIORAL] Connect filters to TodoList"

# 7. 마지막 정리
git commit -m "[STRUCTURAL] Simplify filter state management"
```

---

## 자주 묻는 질문

### Q1: 너무 많은 커밋이 생기는데, 괜찮나요?

**A:** 네, 완전히 괜찮습니다!

많은 작은 커밋 >>> 적은 큰 커밋

**이유:**
- 각 커밋이 명확한 목적
- 버그 추적이 쉬움
- 코드 리뷰가 편함
- 필요시 특정 변경만 되돌리기 가능

**예시:**
```bash
# 나쁨: 큰 커밋 1개
git commit -m "Implement todo feature" (+500줄 -200줄)

# 좋음: 작은 커밋 10개
git commit -m "[STRUCTURAL] Extract validation" (+20줄)
git commit -m "[STRUCTURAL] Rename variables" (+10줄)
git commit -m "[BEHAVIORAL] Add todo input" (+50줄)
...
```

### Q2: 이미 코드를 다 작성했는데, 어떻게 분리하나요?

**A:** `git add -p` (patch mode) 사용!

```bash
# 파일의 일부만 stage하기
git add -p src/components/TodoList.tsx

# Git이 물어봄:
# Stage this hunk [y,n,q,a,d,s,e,?]?

# y = 이 부분 stage
# n = 이 부분 건너뛰기
# s = 더 작게 나누기
# e = 수동으로 편집
```

**실전 예시:**
```bash
# TodoList.tsx에서
# 1. 구조 변경 부분만 선택
git add -p src/components/TodoList.tsx
# 구조 변경 부분만 선택 → y
# 기능 변경 부분 건너뛰기 → n

git commit -m "[STRUCTURAL] Extract TodoItem component"

# 2. 남은 기능 변경 커밋
git add src/components/TodoList.tsx
git commit -m "[BEHAVIORAL] Add delete functionality"
```

### Q3: 구조와 기능을 정말 100% 분리해야 하나요?

**A:** 예, 가능한 한 100% 분리해야 합니다.

**예외가 있다면:**
- 매우 작은 변경 (변수 이름 하나 바꾸기 + 한 줄 기능)
- 프로토타입/실험 단계

**하지만 프로덕션에서는:**
- 엄격하게 분리
- 예외를 인정하면 원칙이 무너짐

**팁:**
"섞고 싶은 유혹이 든다면, 그것이 분리해야 한다는 신호"

### Q4: 테스트 작성이 너무 시간이 오래 걸려요

**A:** 처음에는 느리지만, 장기적으로는 빨라집니다.

**시간 비교:**

**테스트 없이:**
```
기능 구현: 1시간
버그 발생: ?
버그 찾기: 2시간
수정: 30분
또 다른 버그: ?
---
총: 예측 불가
```

**TDD로:**
```
테스트 작성: 30분
구현: 1시간
리팩토링: 30분
버그: 거의 없음
---
총: 2시간 (예측 가능)
```

**추가 이점:**
- 리팩토링 안전
- 회귀 버그 방지
- 코드 문서화

### Q5: 어떤 것을 테스트해야 하나요?

**A:** 공개 API(함수/컴포넌트의 인터페이스)를 테스트

**테스트해야 할 것:**
```typescript
// ✅ 공개 함수
export function validateTodoText(text: string) {...}
// → 테스트 작성

// ✅ 컴포넌트의 동작
export function TodoInput({ onAdd }: Props) {...}
// → "버튼 클릭 시 onAdd 호출되는가?" 테스트

// ✅ 유틸리티 함수
export function formatDate(date: Date): string {...}
// → 여러 입력에 대한 출력 테스트
```

**테스트 안 해도 되는 것:**
```typescript
// ❌ 내부 함수 (export 안 됨)
function helperFunction() {...}
// → 공개 함수 테스트로 간접 검증

// ❌ 프레임워크 기능
// → React, Vite 등은 이미 테스트됨

// ❌ 너무 단순한 것
const MAX_LENGTH = 200;
// → 테스트 불필요
```

### Q6: Refactor와 Structural의 차이는?

**A:** Refactor는 Structural의 하위 집합입니다.

**Structural (구조 변경):**
- 이름 바꾸기
- 파일 이동
- 함수 추출
- 코드 정리
- **리팩토링 포함**

**Refactor (리팩토링):**
- 코드를 "더 좋게" 만들기
- 중복 제거
- 복잡도 감소
- 성능 개선 (동작 불변)

**실전에서:**
- 둘 다 사용 가능
- `[REFACTOR]`가 더 구체적
- `[STRUCTURAL]`이 더 포괄적

**예시:**
```bash
[STRUCTURAL] Rename getUserData to fetchUserProfile
[REFACTOR] Simplify nested if statements

# 둘 다 OK!
# 팀에서 하나로 통일해도 됨
```

### Q7: 실제 회사에서도 이렇게 하나요?

**A:** 회사마다 다르지만, 점점 더 많이 도입되고 있습니다.

**큰 회사들:**
- Google, Facebook, Microsoft 등에서 유사한 원칙 사용
- 자동화 도구로 강제하기도 함

**스타트업:**
- 초기에는 빠른 개발 우선
- 성장하면서 도입
- 기술 부채 관리 차원

**추세:**
- TDD: 천천히 확산 중
- 구조/기능 분리: 점점 더 일반화

**여러분의 선택:**
- 개인 프로젝트에서 연습
- 팀에 제안
- 점진적 도입

---

## 연습 문제

### 연습 1: 구조 vs 기능 구분하기

다음 변경 사항이 구조 변경인지, 기능 변경인지 판단하세요:

**A. 변수 이름을 `data`에서 `userData`로 변경**
- [ ] 구조 변경
- [ ] 기능 변경

**B. 버튼에 로딩 스피너 추가**
- [ ] 구조 변경
- [ ] 기능 변경

**C. 중복된 코드를 함수로 추출**
- [ ] 구조 변경
- [ ] 기능 변경

**D. 비밀번호 최소 길이를 8자로 설정**
- [ ] 구조 변경
- [ ] 기능 변경

**E. O(n²) 알고리즘을 O(n)으로 개선 (결과 동일)**
- [ ] 구조 변경
- [ ] 기능 변경

**정답:**
```
A: 구조 변경 (이름만 변경, 동작 동일)
B: 기능 변경 (UI가 바뀜, 사용자가 봄)
C: 구조 변경 (동작 동일, 구조만 개선)
D: 기능 변경 (새로운 규칙 추가)
E: 구조 변경 (결과 동일, 내부 개선)
```

### 연습 2: 올바른 커밋 순서

Todo 앱에 "완료된 항목 일괄 삭제" 기능을 추가합니다.
다음 커밋을 올바른 순서로 배열하세요:

**커밋 목록:**
- A. [BEHAVIORAL] Add clear completed button
- B. [STRUCTURAL] Extract todo filter logic
- C. [BEHAVIORAL] Implement clear completed function
- D. [STRUCTURAL] Rename confusing variable names
- E. [BEHAVIORAL] Add confirmation dialog

**질문:** 올바른 순서는?

**정답:**
```
D → B → C → A → E

설명:
1. D: 변수 이름 정리 (구조)
2. B: 필터 로직 추출 (구조)
3. C: 삭제 기능 구현 (기능)
4. A: 버튼 UI 추가 (기능)
5. E: 확인 대화상자 (기능)

원칙: 구조 개선 먼저, 그 다음 기능 추가
```

### 연습 3: 실전 시나리오

**시나리오:**
로그인 폼에 "비밀번호 표시/숨기기" 버튼을 추가하려고 합니다.
현재 코드는 복잡하고 중복이 많습니다.

**현재 코드:**
```typescript
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // 복잡한 검증 로직이 인라인으로
  const handleSubmit = () => {
    if (email === '' || !email.includes('@')) {
      alert('Invalid email');
      return;
    }
    if (password === '' || password.length < 8) {
      alert('Invalid password');
      return;
    }
    // 로그인 처리
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">로그인</button>
    </form>
  );
}
```

**과제:** 이 기능을 추가하는 커밋 순서를 작성하세요.

**힌트:**
1. 먼저 무엇을 정리해야 할까?
2. 어떤 순서로?
3. 각 커밋의 타입은?

**예시 답안:**
```bash
# Phase 1: 구조 정리
git commit -m "[STRUCTURAL] Extract email validation logic"
git commit -m "[STRUCTURAL] Extract password validation logic"
git commit -m "[STRUCTURAL] Create useLoginForm custom hook"

# Phase 2: 기능 추가
git commit -m "[BEHAVIORAL] Add password visibility state"
git commit -m "[BEHAVIORAL] Add password toggle button"
git commit -m "[BEHAVIORAL] Implement toggle functionality"

# Phase 3: 정리
git commit -m "[STRUCTURAL] Simplify password input component"
```

---

## 마치며

### 핵심 원칙 정리

**1. TDD는 순서의 혁명**
- 테스트 → 코드 (기존: 코드 → 테스트)
- Red → Green → Refactor
- 작은 단계로, 자주

**2. Tidy First는 분리의 원칙**
- 구조 변경 ≠ 기능 변경
- 절대 섞지 않기
- 구조를 먼저 개선

**3. 커밋은 이야기**
- [STRUCTURAL], [BEHAVIORAL], [FIX], [REFACTOR]
- 각 커밋은 하나의 목적
- 작고 자주

### 실천 가이드

**처음 시작할 때:**
1. 작은 프로젝트로 시작
2. 한 번에 하나씩 (TDD 먼저, 또는 Tidy First 먼저)
3. 완벽하지 않아도 OK
4. 점진적으로 개선

**익숙해지면:**
1. 더 큰 프로젝트에 적용
2. 팀에 공유
3. 자동화 도구 도입
4. 지속적 개선

### 왜 이것이 중요한가?

**단기적으로:**
- 버그가 줄어듦
- 코드 리뷰가 쉬워짐
- 자신감이 생김

**장기적으로:**
- 코드베이스가 건강해짐
- 새 기능 추가가 빨라짐
- 기술 부채가 쌓이지 않음

**가장 중요한 것:**
- 미래의 나(그리고 동료)를 위한 투자
- "씨앗 옥수수"를 남겨두기
- 지속 가능한 개발

### 다음 단계

**이 문서를 읽었다면:**
1. Todo 앱 프로젝트 시작
2. CLAUDE-v2.md 사용
3. 각 커밋을 의식적으로 분리
4. 실수해도 괜찮음 (배우는 과정)
5. 점점 자연스러워짐

**기억하세요:**
- 완벽하지 않아도 됨
- 조금씩 나아지면 됨
- 과정을 즐기세요

---

## 부록: 빠른 참조 가이드

### 커밋 타입 결정 플로우차트

```
변경을 했는가?
    │
    ├─ 동작이 바뀌었는가?
    │   │
    │   ├─ 예 → 버그 수정인가?
    │   │   │
    │   │   ├─ 예 → [FIX]
    │   │   └─ 아니오 → [BEHAVIORAL]
    │   │
    │   └─ 아니오 → 코드가 개선되었는가?
    │       │
    │       ├─ 예 → [REFACTOR] 또는 [STRUCTURAL]
    │       └─ 아니오 → [STRUCTURAL]
    │
    └─ 아직 변경 안 함 → 먼저 정리할 것이 있는가?
        │
        ├─ 예 → Tidy First! (구조 먼저)
        └─ 아니오 → TDD 시작 (테스트 먼저)
```

### 체크리스트

**커밋 전에 확인:**
- [ ] 모든 테스트가 통과하는가?
- [ ] 구조와 기능이 섞이지 않았는가?
- [ ] 커밋 메시지에 타입이 있는가?
- [ ] 한 가지 목적만 있는가?

**기능 추가 전에 확인:**
- [ ] 관련 코드를 먼저 정리했는가?
- [ ] 테스트를 먼저 작성했는가?
- [ ] 최소 구현만 하고 있는가?

**리팩토링 전에 확인:**
- [ ] 모든 테스트가 통과하는가?
- [ ] 한 번에 하나만 바꾸는가?
- [ ] 각 변경 후 테스트하는가?

---

**이제 시작할 준비가 되었습니다!**

Kent Beck의 TDD와 Tidy First 원칙을 실천하면서, 더 나은 개발자로 성장하시길 바랍니다. 🚀

**작성일: 2024-12-24**
