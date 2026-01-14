---
title: "Kent Beck의 TDD와 Tidy First 원칙 완벽 가이드"
date: 2025-12-24 19:00:00 +0900
categories: [AI,  Vibe Coding]
mermaid: [True]
tags: [AI,   vibe-coding,  KentBeck,  TDD,  tidy-first,  Developer,  Guide,  Claude.write]
---

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
