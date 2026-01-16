---
title: "Google Antigravity 실전 개발자 가이드"
date: 2026-01-16 02:30:00 +0900
categories: [AI,  Antigravity]
mermaid: [True]
tags: [AI,  Antigravity,  agent-skills,  sub-agents,   agent-orchestration,  Guide,  Developer,  Claude.write]
---


> **최신 업데이트**: 2025년 11월 Gemini 3와 함께 출시 | 2026년 1월 Agent Skills 추가  
> **공식 사이트**: https://antigravity.google/download

---

## 📋 목차

1. [Antigravity란 무엇인가](#antigravity란-무엇인가)
2. [핵심 개념](#핵심-개념)
3. [워크플로우 시스템 완벽 가이드](#워크플로우-시스템-완벽-가이드)
4. [Agent Skills - 최신 기능](#-agent-skills---최신-기능-2026년-1월-업데이트)
5. [2가지 작업 모드](#2가지-작업-모드)
6. [실전 사용 시나리오](#실전-사용-시나리오)
7. [보안 및 제한 설정](#보안-및-제한-설정)
8. [최신 업데이트 내역](#최신-업데이트-내역)
9. [경쟁 도구 비교](#경쟁-도구-비교)
10. [문제 해결 및 팁](#문제-해결-및-팁)

---

## Antigravity란 무엇인가

### 기본 정보

**Google Antigravity**는 2025년 11월 18일 Gemini 3와 함께 발표된 에이전트 중심 AI IDE입니다.

- **기반**: Visual Studio Code 포크 (Windsurf 팀 인수)
- **지원 모델**: Gemini 3 Pro/Flash, Claude Sonnet 4.5, GPT-OSS-120B
- **플랫폼**: Windows, macOS, Linux
- **가격**: Public Preview 기간 무료 (제한된 quota)

### 전통적 IDE vs Antigravity

| 항목 | 전통적 IDE | Antigravity |
|------|-----------|-------------|
| **개발자 역할** | 코드 작성자 | 아키텍트/매니저 |
| **AI 역할** | 자동완성 도우미 | 자율 실행 에이전트 |
| **작업 단위** | 라인/함수 | 전체 태스크/기능 |
| **실행 방식** | 동기(순차) | 비동기(병렬) |
| **검증** | 수동 테스트 | AI 자동 검증 + 브라우저 테스트 |

### 핵심 철학: "Agent-First"

```
전통적 방식: "이 함수를 어떻게 작성할까?"
Antigravity 방식: "OAuth 로그인 기능을 구현해줘. 보안 체크도 포함하고 테스트까지 완료해줘."
```

---

## 핵심 개념

### 1. Artifacts (산출물)

에이전트가 생성하는 **검증 가능한 결과물**입니다. raw tool calls 대신 사람이 이해할 수 있는 형태로 제공됩니다.

**Artifacts 종류**:
- **Task Plan** (작업 계획서)
- **Implementation Plan** (구현 계획)
- **Walkthroughs** (실행 과정 기록)
- **Screenshots** (스크린샷)
- **Browser Recordings** (브라우저 작업 녹화)

**특징**:
- Google Docs 스타일의 실시간 코멘트 가능
- 에이전트 실행 중단 없이 피드백 전달
- 모든 산출물은 검증 가능한 형태로 제공

### 2. Knowledge Base (지식 베이스)

에이전트가 **과거 작업에서 학습**하여 축적하는 컨텍스트 저장소입니다.

- 코드 스니펫 저장
- 프로젝트별 패턴 학습
- 개발자 피드백 반영
- 반복 작업 최적화

### 3. 비동기 실행

에이전트는 백그라운드에서 작업하며, 개발자는 다른 작업을 계속할 수 있습니다.

```
동기 방식 (기존):
[요청] → [대기...] → [결과 확인] → [다음 요청]

비동기 방식 (Antigravity):
[요청 1] → [백그라운드 실행]
[요청 2] → [백그라운드 실행]
[요청 3] → [백그라운드 실행]
→ 결과는 Manager View에서 모니터링
```

---

## 워크플로우 시스템 완벽 가이드

### Rules vs Workflows vs Skills 비교표

| 특성 | Rules | Workflows | Skills |
|------|-------|-----------|--------|
| **목적** | 행동 규칙 정의 | 저장된 프롬프트 | 재사용 가능한 전문 지식 |
| **실행 시점** | 항상 활성 | 사용자가 `/` 명령으로 트리거 | 요청과 매칭될 때 자동 로드 |
| **유사 개념** | System Instructions | Saved Prompts | Plugin/Extension |
| **범위** | Global/Workspace | Global/Workspace | Global/Workspace |
| **수정 빈도** | 낮음 (프로젝트 규칙) | 중간 (자주 쓰는 작업) | 낮음 (전문 워크플로우) |

---

### 📌 Rules (규칙) 상세 가이드

#### Rules란?

에이전트가 **항상 준수해야 하는 시스템 수준의 지침**입니다. 사용자가 트리거하지 않아도 모든 작업에 자동 적용됩니다.

#### Rules 설정 위치

1. **Global Rules** (모든 프로젝트에 적용)
   ```
   ~/.gemini/antigravity/rules.md
   ```

2. **Workspace Rules** (특정 프로젝트에만 적용)
   ```
   <project-root>/.gemini/antigravity/rules.md
   ```

#### Rules 작성 예시

**예시 1: 코드 스타일 규칙**
```markdown
# Code Style Rules

## Python
- Use type hints for all function parameters and return values
- Maximum line length: 100 characters
- Use docstrings (Google style) for all public functions
- Prefer f-strings over .format() or % formatting

## Testing
- Write unit tests for all new functions
- Use pytest fixtures for setup/teardown
- Test file naming: test_<module_name>.py
- Minimum test coverage: 80%

## Documentation
- Update README.md when adding new features
- Include usage examples for public APIs
- Comment complex algorithms with time/space complexity
```

**예시 2: 보안 규칙**
```markdown
# Security Rules

- Never hardcode API keys, passwords, or secrets
- Use environment variables for sensitive configuration
- Sanitize all user inputs before database queries
- Always use parameterized queries (no string concatenation)
- Implement rate limiting on all public API endpoints
- Log all authentication attempts
```

**예시 3: 프레임워크 특화 규칙**
```~markdown
# React Project Rules

- Use functional components only (no class components)
- Use React Hooks for state management
- Component naming: PascalCase
- File structure: one component per file
- Props interface naming: <ComponentName>Props
- Always use TypeScript strict mode
- Use styled-components for CSS-in-JS
- Implement error boundaries for production code
```

#### Rules 설정 방법

**UI에서 설정**:
1. Editor 화면 우측 상단 `...` 클릭
2. `Customizations` → `Manage` 선택
3. `Rules` 탭에서 추가/수정

**직접 파일 수정**:
```bash
# Global Rules
code ~/.gemini/antigravity/rules.md

# Workspace Rules
code .gemini/antigravity/rules.md
```

#### Rules 우선순위

```
Workspace Rules > Global Rules
```

동일한 항목에 대해 Workspace Rules가 Global Rules를 **오버라이드**합니다.

---

### 🔄 Workflows (워크플로우) 상세 가이드

#### Workflows란?

**저장된 프롬프트**로, `/` 명령어로 트리거하여 반복 작업을 자동화합니다.

#### Workflows 설정 위치

1. **Global Workflows**
   ```
   ~/.gemini/antigravity/workflows/
   ```

2. **Workspace Workflows**
   ```
   <project-root>/.gemini/antigravity/workflows/
   ```

#### Workflows 파일 구조

```yaml
# generate-unit-tests.md
---
name: generate-unit-tests
description: Generate comprehensive unit tests for Python modules
---

Generate unit tests for the selected Python file with the following requirements:

1. Use pytest framework
2. Include:
   - Happy path tests
   - Edge cases
   - Error handling tests
   - Parameterized tests where appropriate
3. Mock external dependencies
4. Aim for >90% code coverage
5. Add docstrings to test functions
6. Save as test_<original_filename>.py
```

#### 실전 Workflows 예시

**예시 1: API 엔드포인트 생성**
```yaml
# create-api-endpoint.md
---
name: create-api-endpoint
description: Create a complete REST API endpoint with validation and tests
---

Create a new REST API endpoint with:

1. **Route Definition**
   - Express.js router
   - HTTP method (GET/POST/PUT/DELETE)
   - URL path with parameters

2. **Controller**
   - Request validation (using Joi)
   - Business logic separation
   - Error handling with appropriate HTTP status codes

3. **Service Layer**
   - Database interaction (if needed)
   - Business logic implementation

4. **Tests**
   - Unit tests for controller
   - Integration tests for endpoint
   - Mock database calls

5. **Documentation**
   - JSDoc comments
   - OpenAPI/Swagger spec update
```

**예시 2: 데이터베이스 마이그레이션**
```yaml
# create-migration.md
---
name: create-migration
description: Generate database migration files with rollback support
---

Create a database migration with:

1. **Up Migration**
   - Table creation/modification
   - Index creation
   - Constraint addition

2. **Down Migration**
   - Complete rollback logic
   - Drop tables/columns safely

3. **Seed Data** (if applicable)
   - Initial data insertion
   - Reference data

4. **Naming Convention**
   - YYYYMMDDHHMMSS_description.sql
   - Example: 20260116120000_add_user_roles.sql

5. **Safety Checks**
   - Add IF EXISTS/IF NOT EXISTS
   - Transaction wrapping
   - Error handling
```

**예시 3: 리팩토링 워크플로우**
```yaml
# refactor-component.md
---
name: refactor-component
description: Refactor React component with best practices
---

Refactor the selected React component:

1. **Extract Custom Hooks**
   - Identify reusable logic
   - Create separate hook files

2. **Optimize Performance**
   - Add React.memo where appropriate
   - Use useMemo/useCallback for expensive computations
   - Implement code splitting

3. **Improve Readability**
   - Break down large components
   - Extract helper functions
   - Add TypeScript types

4. **Testing**
   - Update existing tests
   - Add tests for new hooks
   - Test memoization behavior

5. **Documentation**
   - Add/update JSDoc
   - Document component props
   - Add usage examples
```

#### Workflows 사용 방법

1. Agent Chat에서 `/` 입력
2. 목록에서 workflow 선택
3. 필요시 추가 컨텍스트 제공

```
예: /create-api-endpoint
→ "Create a POST endpoint for user registration with email validation"
```

#### Workflows vs Rules 사용 시나리오

**Rules 사용 시기**:
- 프로젝트 전체에 적용되는 코딩 표준
- 항상 지켜야 하는 보안 규칙
- 팀 컨벤션 (네이밍, 구조 등)

**Workflows 사용 시기**:
- 반복적으로 수행하는 특정 작업
- 다단계 프로세스 자동화
- 팀원 간 공유할 작업 템플릿

---

### 🎯 Agent Skills - 최신 기능 (2026년 1월 업데이트)

#### Skills란?

**Progressive Disclosure** 방식으로 작동하는 **전문 지식 패키지**입니다.

전통적 방식의 문제점:
```
❌ 모든 규칙, 도구, 문서를 Context에 로드
→ Context Saturation (컨텍스트 포화)
→ 높은 비용, 느린 응답, 도구 혼란
```

Skills의 해결책:
```
✅ 가벼운 메타데이터만 노출
→ 요청과 매칭되면 해당 Skill만 로드
→ 효율적인 컨텍스트 사용
```

#### Skills vs Rules vs Workflows

```
Rules: "항상 이렇게 해"
Workflows: "내가 말하면 이렇게 해"
Skills: "이런 상황이면 자동으로 이렇게 해"
```

#### Skills 파일 구조

```
skills/
  my-skill/
    SKILL.md          # 필수: Skill 정의
    scripts/          # 선택: 실행 스크립트
    templates/        # 선택: 코드 템플릿
    examples/         # 선택: 사용 예시
```

#### SKILL.md 작성 형식

~~~markdown
---
name: database-migration-workflow
description: Generates production-ready database migration with rollback support
version: 1.0.0
author: team@company.com
tags: [database, migration, postgresql]
---

# Database Migration Skill

## Overview
This skill helps create safe, reversible database migrations following our team's standards.

## When to Use
- Creating new tables or modifying existing ones
- Adding/removing columns
- Creating indexes or constraints
- Data migrations

## Prerequisites
- PostgreSQL database
- Migration tool: Flyway or Liquibase
- Access to database schema

## Workflow

### Step 1: Analyze Changes
- Review existing schema
- Identify affected tables
- Check foreign key dependencies

### Step 2: Generate Up Migration
```sql
-- Example structure
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

### Step 3: Generate Down Migration
```sql
-- Always provide rollback
DROP INDEX IF EXISTS idx_users_email;
DROP TABLE IF EXISTS users;
```

### Step 4: Safety Checks
- Use IF EXISTS/IF NOT EXISTS
- Wrap in transaction
- Add timeout limits
- Test on development first

### Step 5: Documentation
- Add migration description
- Document breaking changes
- Update schema documentation

## Verification
- Run migration on test database
- Verify rollback works
- Check performance impact
- Review with DBA if production
~~~

#### Skills 작성 예시

**예시 1: React Component Generator**
~~~markdown
---
name: react-component-generator
description: Generate production-ready React component with TypeScript, tests, and storybook
tags: [react, typescript, testing, storybook]
---

# React Component Generator

## Generated Files
1. Component.tsx - Main component
2. Component.types.ts - TypeScript interfaces
3. Component.test.tsx - Jest + React Testing Library tests
4. Component.stories.tsx - Storybook stories
5. index.ts - Barrel export

## Component Template
```typescript
import React from 'react';
import { ComponentNameProps } from './ComponentName.types';

export const ComponentName: React.FC<ComponentNameProps> = ({
  // props
}) => {
  return (
    <div>
      {/* component JSX */}
    </div>
  );
};
```

## Test Template
```typescript
import { render, screen } from '@testing-library/react';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  it('renders correctly', () => {
    render(<ComponentName />);
    expect(screen.getByText(/expected text/i)).toBeInTheDocument();
  });
});
```
~~~

**예시 2: API Integration Skill**
~~~markdown
---
name: api-integration
description: Create complete API integration with error handling, retry logic, and types
tags: [api, axios, typescript, error-handling]
---

# API Integration Skill

## Components
1. API Client Configuration
2. Request/Response TypeScript interfaces
3. Error handling wrapper
4. Retry logic with exponential backoff
5. Request/Response interceptors
6. Unit tests with mocked responses

## Implementation Pattern

### 1. API Client Setup
```typescript
import axios, { AxiosInstance } from 'axios';

export class ApiClient {
  private client: AxiosInstance;
  
  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });
    
    this.setupInterceptors();
  }
  
  private setupInterceptors(): void {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        // Add auth token
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );
    
    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      async (error) => {
        // Retry logic
        return this.handleError(error);
      }
    );
  }
}
```

### 2. Error Handling
- 400: Bad Request → Validate input
- 401: Unauthorized → Refresh token
- 403: Forbidden → Redirect to login
- 404: Not Found → Handle gracefully
- 429: Rate Limited → Exponential backoff
- 500+: Server Error → Retry with backoff

### 3. Testing
- Mock axios responses
- Test error scenarios
- Verify retry behavior
- Check timeout handling
~~~

#### Skills 설치 위치

```bash
# Global Skills (모든 프로젝트)
~/.gemini/antigravity/skills/

# Workspace Skills (특정 프로젝트)
<project-root>/.gemini/antigravity/skills/
```

#### Skills 동작 방식

```
1. 사용자: "데이터베이스 마이그레이션 만들어줘"
   ↓
2. Antigravity: Skills 메타데이터 검색
   - "database", "migration" 태그 매칭
   ↓
3. 해당 Skill의 상세 내용만 Context에 로드
   ↓
4. Skill 워크플로우에 따라 실행
   ↓
5. 검증 및 결과 제공
```

#### Skills 사용 팁

**Skill 작성 가이드라인**:
1. **명확한 트리거 조건**: description과 tags를 정확하게
2. **단계별 워크플로우**: 에이전트가 따라갈 수 있는 명확한 프로세스
3. **예제 포함**: 코드 템플릿과 실제 사용 예시
4. **검증 기준**: 성공/실패 판단 기준 명시
5. **버전 관리**: 변경 사항 추적

**Progressive Disclosure의 이점**:
```
전통적 방식:
- 100개 도구 → Context에 모두 로드 → 혼란, 느림, 비용↑

Skills 방식:
- 100개 Skills → 메타데이터만 노출 → 필요한 1개만 로드 → 빠름, 효율적
```

---

## 2가지 작업 모드

### Editor View (편집기 모드)

**언제 사용**: 전통적인 코딩 작업, 작은 수정

**기능**:
- VS Code와 유사한 인터페이스
- AI 자동완성 (Tab completion)
- 인라인 자연어 명령
- 동기식 워크플로우

**사용 시나리오**:
```python
# 코드 작성 중
def calculate_total(items):
    # 여기서 Tab → AI가 자동완성
```

### Manager View (매니저 모드)

**언제 사용**: 복잡한 태스크, 다중 에이전트 오케스트레이션

**기능**:
- 다중 에이전트 동시 실행
- 작업 진행 상황 모니터링
- 비동기 피드백 시스템
- Plan/Fast 모드 선택

**작업 모드 비교**:

| 항목 | Plan Mode | Fast Mode |
|------|-----------|-----------|
| **속도** | 느림 (신중함) | 빠름 (즉시 실행) |
| **Artifact 생성** | Yes (상세한 계획서) | No (바로 실행) |
| **적합한 작업** | 복잡한 기능 개발, 리팩토링 | 간단한 수정, 버그 픽스 |
| **검증** | 계획 검토 후 실행 | 실행 후 검증 |

**Plan Mode 예시**:
```
요청: "사용자 인증 시스템 구현"
↓
1. Plan Artifact 생성
   - 데이터베이스 스키마
   - API 엔드포인트 설계
   - 인증 플로우
   - 보안 고려사항
↓
2. 개발자 검토 및 피드백
↓
3. 실행
```

**Fast Mode 예시**:
```
요청: "이 함수에 타입 힌트 추가해줘"
→ 즉시 실행
```

---

## 실전 사용 시나리오

### 시나리오 1: 새로운 기능 추가

```
[Manager View - Plan Mode]

개발자: "쇼핑몰에 위시리스트 기능 추가해줘. 
       사용자는 상품을 저장하고, 공유하고, 알림받을 수 있어야 해."

↓ Agent 작업 흐름

1. Plan Artifact 생성
   - 데이터베이스 스키마 설계
   - API 엔드포인트 목록
   - 프론트엔드 컴포넌트 구조
   - 알림 시스템 통합

2. 개발자 피드백
   → "공유 기능은 SNS 공유만 지원하도록 변경"

3. 병렬 실행
   [Agent 1] 데이터베이스 마이그레이션
   [Agent 2] 백엔드 API 구현
   [Agent 3] React 컴포넌트 개발
   [Agent 4] 단위 테스트 작성

4. Browser Agent 검증
   - UI 테스트
   - 플로우 검증
   - 스크린샷 캡처

5. Walkthrough Artifact
   - 구현 내용 요약
   - 스크린샷
   - 테스트 결과
```

### 시나리오 2: 버그 수정 + 테스트

```
[Editor View - Fast Mode]

개발자: "로그인 시 토큰 만료 처리 버그 수정하고 테스트 추가"

↓ Agent 작업

1. 코드 분석
   - 토큰 검증 로직 확인
   - 에러 핸들링 검토

2. 수정 사항
   - 만료 토큰 감지 로직 추가
   - 자동 리프레시 시도
   - 실패 시 로그아웃 처리

3. 테스트 추가
   - 정상 토큰 테스트
   - 만료 토큰 테스트
   - 리프레시 실패 테스트

4. Knowledge Base 업데이트
   → 토큰 처리 패턴 저장
```

### 시나리오 3: 코드 리뷰 자동화

```
[Workflow 활용]

/review-code

↓ Workflow 내용

1. 코드 품질 체크
   - 복잡도 분석
   - 코드 스멜 탐지
   - 네이밍 컨벤션 확인

2. 보안 검사
   - SQL 인젝션 취약점
   - XSS 가능성
   - 민감 정보 하드코딩

3. 성능 분석
   - 비효율적 루프
   - 불필요한 API 호출
   - 메모리 누수 가능성

4. 개선 제안
   - 리팩토링 추천
   - 최적화 방안
   - 문서화 필요 부분
```

---

## 보안 및 제한 설정

### Secure Mode (보안 모드)

**목적**: 에이전트의 위험한 작업을 제어

#### 설정 위치

Settings → Security → Agent Controls

#### 3가지 제어 메커니즘

**1. Terminal Policy (터미널 정책)**

| 정책 | 설명 | 사용 시나리오 |
|------|------|--------------|
| **Auto** | 표준 명령어 자동 실행 | 개인 프로젝트, 신뢰하는 환경 |
| **Agent Decides** | 에이전트가 위험도 판단 | 일반적인 개발 환경 |
| **Always Ask** | 모든 명령어 승인 필요 | 프로덕션 환경, 민감한 프로젝트 |

**2. Allow List (허용 목록)**

```yaml
# ~/.gemini/antigravity/security.yml

allow_commands:
  - npm install
  - npm run
  - pytest
  - git status
  - git diff
  - docker ps
  
# 허용된 명령어만 자동 실행
```

**3. Deny List (거부 목록)**

```yaml
deny_commands:
  - rm -rf
  - sudo
  - DROP DATABASE
  - DELETE FROM users
  - kill -9
  
# 절대 실행 불가 (승인해도 거부)
```

#### Browser Allow List

웹 브라우저 접근 제어:

```yaml
browser_allow_list:
  - "*.google.com"
  - "*.github.com"
  - "*.stackoverflow.com"
  - "localhost:*"
  
# 허용되지 않은 도메인은 접근 차단
```

### Usage Limits (사용량 제한)

#### Quota 정책 (2025년 12월 업데이트)

| 플랜 | Refresh 주기 | 제한 |
|------|--------------|------|
| **Free** | 매주 | 주간 할당량 |
| **AI Pro** | 5시간마다 | 시간당 할당량 |
| **AI Ultra** | 5시간마다 | 무제한 (우선 큐) |

#### 사용량 최적화 팁

**1. 작업 단위 최소화**
```
❌ "전체 앱 리팩토링해줘"
✅ "User 컴포넌트만 리팩토링해줘"
```

**2. Plan 먼저, 실행은 나중에**
```
Plan Mode로 검토 → 수정 → Fast Mode로 실행
(불필요한 재실행 방지)
```

**3. 컨텍스트 최소화**
```
❌ 프로젝트 전체 첨부
✅ 관련 파일만 선택적 첨부
```

**4. Flash 모델 활용**
```
간단한 작업: Gemini 3 Flash (3x 빠름, 토큰 절약)
복잡한 작업: Gemini 3 Pro
```

**5. 단계별 실행**
```
❌ "UI 만들고 → 로직 구현하고 → 테스트 작성해줘" (한 번에)
✅ "UI 만들어줘" → 확인 → "로직 구현해줘" → 확인 → "테스트 작성해줘"
```

---

## 최신 업데이트 내역

### 🆕 2026년 1월 (최신)

#### Agent Skills 시스템 도입
- **Progressive Disclosure**: 필요할 때만 Skill 로드
- **오픈 스탠다드**: 커뮤니티 Skill 공유 가능
- **Global/Workspace 범위**: 유연한 적용 범위

#### Gemini 3 Flash 통합
- **성능**: Gemini 2.5 Pro 대비 3배 빠름
- **사용 사례**: 빠른 코드 수정, 간단한 리팩토링
- **비용 효율**: 토큰 소비 감소

#### Secure Mode 강화
- **세밀한 제어**: Allow/Deny List
- **Browser 제어**: 도메인별 접근 제어
- **감사 로그**: 모든 에이전트 작업 기록

### 2025년 12월

- **Usage Caps 정책 변경**
  - Free: 주간 리프레시
  - Pro/Ultra: 5시간 리프레시
  
- **Gemini 3 Pro (High) 성능 개선**
  - SWE-bench Verified: 76.2%
  - Terminal-Bench 2.0: 54.2%

### 2025년 11월 (초기 릴리즈)

- **Antigravity 공개**
  - Editor/Manager View
  - Artifacts 시스템
  - Knowledge Base
  - Multi-agent orchestration

---

## 경쟁 도구 비교

### Antigravity vs Cursor vs Windsurf

| 기능 | Antigravity | Cursor | Windsurf |
|------|-------------|--------|----------|
| **자율 에이전트** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **병렬 실행** | ✅ 다중 에이전트 | ❌ | ✅ |
| **브라우저 제어** | ✅ 자동 테스트 | ❌ | ⚠️ 제한적 |
| **Artifacts** | ✅ Plan/Screenshot/Recording | ❌ | ⚠️ 기본 |
| **코드 완성** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **로컬 인덱싱** | ⚠️ 제한적 | ✅ 강력 | ✅ |
| **확장성** | ⚠️ 제한적 | ✅ Extension 생태계 | ✅ |
| **가격** | 무료 (Preview) | $20/월 | 무료 + 유료 |

### 사용 시나리오별 추천

**Antigravity 추천**:
- 복잡한 기능 개발 (다단계 작업)
- 자동화된 테스트/검증 필요
- 병렬 개발 (여러 파일 동시 작업)
- 프로토타입 빠른 생성

**Cursor 추천**:
- 정밀한 코드 수정
- 대규모 코드베이스 탐색
- 로컬 우선 워크플로우
- 확장 기능 많이 사용

**하이브리드 접근**:
```
Antigravity: 새 기능 골격 생성 + 테스트 자동화
Cursor: 세밀한 수정 + 코드 리뷰
```

---

## 문제 해결 및 팁

### 일반적인 문제

#### 1. 에이전트가 예상과 다른 결과 생성

**원인**: 모호한 지시, 부족한 컨텍스트

**해결책**:
```
❌ "이 코드 개선해줘"
✅ "이 함수의 시간 복잡도를 O(n²)에서 O(n log n)으로 개선해줘. 
   가독성은 유지하고 주석 추가해줘."
```

**팁**:
- 구체적인 요구사항 명시
- 제약사항 포함
- 예상 결과 설명

#### 2. 토큰 소진이 빠름

**해결책**:

**Plan-First 접근**:
```python
# Step 1: Plan만 생성 (토큰 절약)
"다크 모드 구현 계획 작성해줘"

# Step 2: Plan 검토 후 실행
"계획대로 구현해줘"
```

**파일 선택적 첨부**:
```
❌ 프로젝트 전체 선택
✅ 수정할 파일 + 관련 의존성만 선택
```

**Flash 모델 활용**:
```
Settings → Model → Gemini 3 Flash
(간단한 작업에 3배 빠름)
```

#### 3. Knowledge Base가 작동하지 않음

**원인**: 프로젝트 디렉토리 이동

**해결책**:
```bash
# Knowledge Items가 깨진 경우
rm -rf .gemini/antigravity/knowledge/
# Antigravity 재시작
```

**예방**:
- 프로젝트 경로는 고정
- 심볼릭 링크 사용 지양

#### 4. 에이전트가 무한 루프

**원인**: 모호한 종료 조건

**해결책**:
```
❌ "버그를 찾아서 모두 수정해줘"
✅ "로그인 버그 3개 수정해줘: 
   1. 토큰 만료 처리
   2. 비밀번호 검증
   3. 에러 메시지 표시"
```

**긴급 중지**:
- Manager View → 작업 선택 → "Stop Agent"

### 고급 팁

#### 1. Rules 계층 활용

```bash
# Global: 모든 프로젝트 기본 규칙
~/.gemini/antigravity/rules.md

# Workspace: 프로젝트 특화 규칙
project/.gemini/antigravity/rules.md
```

**예시**:
```
Global: "Always use TypeScript"
Workspace: "This project uses JavaScript (legacy)"
→ Workspace가 우선
```

#### 2. Workflows 체이닝

```markdown
# setup-new-feature.md
---
name: setup-new-feature
---

1. /create-api-endpoint
2. /generate-tests
3. /update-documentation
```

#### 3. Browser Agent 활용

```
"로그인 플로우를 실제로 테스트하고 스크린샷 찍어줘"

→ Browser Agent:
  1. 브라우저 실행
  2. 폼 입력
  3. 제출
  4. 결과 검증
  5. 스크린샷 캡처
```

#### 4. 효율적인 피드백

**Artifact에 직접 코멘트**:
```
Implementation Plan에서:
"이 부분은 Redis로 변경" → 드래그하여 코멘트
```

**에이전트는 실행 중단 없이 반영**

#### 5. Knowledge Base 적극 활용

에이전트가 자동으로 학습하도록:
```
좋은 구현: 👍 피드백 → Knowledge Base 저장
나쁜 구현: 👎 피드백 → 패턴 회피
```

---

## 추가 리소스

### 공식 문서
- https://antigravity.google/docs
- https://developers.googleblog.com/build-with-google-antigravity

### 커뮤니티
- Google AI Developers Forum
- GitHub: google/antigravity
- Reddit: r/antigravity

### 학습 자료
- Google Codelabs: Getting Started with Antigravity
- YouTube: Antigravity Tutorials

---

## 요약 체크리스트

### 초기 설정
- [ ] Antigravity 설치
- [ ] Google 계정 연결
- [ ] 모델 선택 (Gemini 3 Pro/Flash)
- [ ] Terminal Policy 설정
- [ ] Global Rules 작성

### 프로젝트 설정
- [ ] Workspace Rules 정의
- [ ] 자주 쓰는 Workflows 생성
- [ ] 프로젝트 특화 Skills 추가
- [ ] Allow/Deny List 설정

### 일상 워크플로우
- [ ] 간단한 수정: Editor View + Fast Mode
- [ ] 복잡한 기능: Manager View + Plan Mode
- [ ] 반복 작업: Workflows 활용
- [ ] 코드 리뷰: Rules 기반 자동 검증

### 최적화
- [ ] Flash 모델로 간단한 작업 처리
- [ ] Plan-First 접근으로 토큰 절약
- [ ] Artifacts에 피드백으로 개선
- [ ] Knowledge Base 활용한 학습

---

**작성일**: 2026-01-16  
**버전**: 1.0  
**다음 업데이트**: Antigravity 정식 출시 시
