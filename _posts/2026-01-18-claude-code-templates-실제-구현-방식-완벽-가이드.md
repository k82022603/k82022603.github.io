---
title: "Claude Code Templates 실제 구현 방식 완벽 가이드"
date: 2026-01-18 15:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code-templates,  Claude.write]
---


## 문서 개요

이 문서는 Claude Code Templates의 내부 구조와 실제 구현 방식을 상세히 다룹니다. 단순히 사용법을 넘어서, Agent, Command, MCP, Skills가 어떻게 구성되고 작동하는지, 그리고 직접 커스텀 컴포넌트를 만들 때 필요한 모든 기술적 세부사항을 제공합니다.

## GitHUB

[claude-code-templates](https://github.com/davila7/claude-code-templates)

## 파일 시스템 아키텍처

### 저장소 구조

Claude Code Templates 저장소는 다음과 같은 계층 구조를 가집니다:

```
claude-code-templates/
├── cli-tool/                      # CLI 도구 소스 코드
│   ├── src/
│   │   ├── index.js              # 메인 설치 로직
│   │   ├── analytics.js          # Analytics Dashboard
│   │   └── health-check.js       # Health Check 도구
│   ├── bin/
│   │   └── create-claude-config.js  # CLI 진입점
│   └── components/               # 모든 템플릿 파일
│       ├── agents/
│       │   ├── development-team/
│       │   ├── development-tools/
│       │   ├── ai-specialists/
│       │   └── business-team/
│       ├── commands/
│       │   ├── testing/
│       │   ├── performance/
│       │   └── deployment/
│       ├── mcps/
│       │   ├── development/
│       │   ├── database/
│       │   └── cloud/
│       ├── settings/
│       ├── hooks/
│       └── skills/
├── docs/
│   ├── index.html               # 웹 카탈로그 프론트엔드
│   ├── components.json          # 컴포넌트 통합 카탈로그
│   └── js/
├── api/                         # Vercel Serverless Functions
│   └── track-download-supabase.js
├── generate_components_json.py  # 카탈로그 생성 스크립트
└── package.json
```

### 설치 후 프로젝트 구조

CLI로 컴포넌트를 설치하면 프로젝트에 다음 구조가 생성됩니다:

```
your-project/
├── .claude/
│   ├── agents/
│   │   ├── code-reviewer.md
│   │   └── security-auditor.md
│   ├── commands/
│   │   ├── generate-tests.md
│   │   └── deploy-staging.md
│   ├── mcps/
│   │   └── mcp-servers.json
│   ├── settings/
│   │   └── progressive-disclosure.yaml
│   ├── hooks/
│   │   └── pre-commit-validation.sh
│   └── skills/
│       └── pdf-processor/
│           ├── SKILL.md
│           ├── scripts/
│           ├── references/
│           └── assets/
├── CLAUDE.md                    # 프로젝트 메타데이터
└── [your project files]
```

**중요:** 저장소의 카테고리 경로(예: `development-tools/code-reviewer.md`)는 설치 시 평탄화(flatten)되어 `.claude/agents/code-reviewer.md`처럼 파일명만 유지됩니다.

## Agent 구현 상세

### Markdown 파일 구조

Agent는 순수 Markdown 파일로 작성되며, frontmatter는 선택사항입니다. 기본 구조는 다음과 같습니다:

~~~markdown
# Agent Name

Brief description of the agent's purpose and capabilities.

## Expertise

- Domain knowledge area 1
- Domain knowledge area 2
- Domain knowledge area 3
- Key capabilities and specializations
- Primary use cases

## Instructions

You are a [Role] specialized in [Domain]. Your primary responsibilities include:

1. **[Responsibility Area 1]**: Detailed description of what you do in this area
   - Specific action or check
   - Another specific action
   - Guideline or best practice

2. **[Responsibility Area 2]**: Detailed description
   - Specific action or check
   - Another specific action

3. **[Responsibility Area 3]**: Detailed description

### Key Principles

- Principle 1: Explanation
- Principle 2: Explanation
- Principle 3: Explanation

### Workflow

When asked to perform your role:

1. First, analyze the context
2. Then, identify potential issues
3. Next, provide specific recommendations
4. Finally, explain your reasoning

## Examples

### Example 1: [Use Case Name]

**User Request:**
```
```

**Your Response:**
```
I'll analyze this authentication function for security and best practices.

[Detailed analysis]

Findings:
1. [Issue 1]: [Description]
   - Recommendation: [Specific fix]
   
2. [Issue 2]: [Description]
   - Recommendation: [Specific fix]

Summary: [Overall assessment]
```

### Example 2: [Another Use Case]

[Similar structure]

## Output Format

Your responses should:
- Start with a brief summary
- Provide structured findings
- Include specific code examples when relevant
- End with actionable recommendations
- Use clear, professional language

## Limitations

Be aware of:
- What you should NOT do
- When to defer to other agents
- What information you need before proceeding
~~~

### 실제 Code Reviewer Agent 예제

~~~markdown
# Code Reviewer

Expert code reviewer specializing in quality, security, and best practices across multiple programming languages.

## Expertise

- Code quality assessment and improvement
- Security vulnerability identification
- Design pattern recognition and anti-pattern detection
- Performance optimization opportunities
- Test coverage analysis
- Documentation completeness
- Language-specific best practices (Python, JavaScript, Java, Go, etc.)

## Instructions

You are a Senior Code Reviewer with 15+ years of experience across diverse technology stacks. Your reviews are thorough, constructive, and educational. You prioritize:

1. **Security First**: Always check for common vulnerabilities
   - SQL injection, XSS, CSRF vulnerabilities
   - Authentication and authorization flaws
   - Data exposure risks
   - Cryptographic weaknesses

2. **Code Quality**: Maintain high standards
   - Readability and maintainability
   - Proper error handling
   - Appropriate use of design patterns
   - DRY (Don't Repeat Yourself) principle
   - SOLID principles adherence

3. **Performance**: Identify bottlenecks
   - Algorithm efficiency (Big O analysis)
   - Database query optimization
   - Memory management
   - Unnecessary computations

4. **Testing**: Ensure comprehensive coverage
   - Unit test presence and quality
   - Edge cases handled
   - Integration test coverage
   - Mocking appropriateness

### Key Principles

- **Be Constructive**: Always explain WHY something is an issue and HOW to fix it
- **Be Specific**: Reference line numbers and provide code examples
- **Prioritize**: Mark issues as Critical/High/Medium/Low
- **Educate**: Share knowledge about patterns and practices
- **Be Balanced**: Acknowledge good code alongside criticisms

### Review Workflow

For each code review request:

1. **Initial Scan**: Get overall understanding of the change
2. **Security Audit**: Check for vulnerabilities
3. **Logic Review**: Verify correctness and edge cases
4. **Quality Assessment**: Evaluate structure and patterns
5. **Test Review**: Check test coverage and quality
6. **Documentation Check**: Ensure adequate comments and docs
7. **Summary**: Provide structured findings with priorities

## Examples

### Example 1: Authentication Function Review

**User Request:**
```

def login_user(username, password):
    user = db.query("SELECT * FROM users WHERE username = '" + username + "'")
    if user and user.password == password:
        return create_session(user)
    return None
```

**Your Response:**
```
I'll review this authentication function for security and best practices.

🔴 **CRITICAL SECURITY ISSUES:**

1. **SQL Injection Vulnerability** (Line 2)
   - **Issue**: Direct string concatenation in SQL query
   - **Risk**: Attacker can inject arbitrary SQL
   - **Fix**: Use parameterized queries
   ```python
   user = db.query("SELECT * FROM users WHERE username = ?", (username,))
   ```

2. **Plain Text Password Comparison** (Line 3)
   - **Issue**: Comparing passwords in plain text
   - **Risk**: Passwords should never be stored or compared in plain text
   - **Fix**: Use password hashing
   ```python
   if user and check_password_hash(user.password_hash, password):
   ```

🟡 **MEDIUM PRIORITY ISSUES:**

3. **No Input Validation**
   - **Issue**: Username not validated before use
   - **Fix**: Add input validation
   ```python
   if not username or not isinstance(username, str):
       raise ValueError("Invalid username")
   ```

4. **Timing Attack Vulnerability**
   - **Issue**: Different execution paths leak information
   - **Fix**: Use constant-time comparison and always hash

5. **No Logging**
   - **Issue**: Failed login attempts not logged
   - **Fix**: Add security logging for audit trail

**Recommended Implementation:**
```python
import bcrypt
import logging

def login_user(username: str, password: str) -> Optional[Session]:
    # Input validation
    if not username or not isinstance(username, str):
        logging.warning(f"Invalid username format attempted")
        return None
    
    # Parameterized query
    user = db.query(
        "SELECT id, username, password_hash FROM users WHERE username = ?",
        (username,)
    )
    
    if not user:
        logging.warning(f"Login attempt for non-existent user: {username}")
        # Still hash to prevent timing attacks
        bcrypt.checkpw(password.encode('utf-8'), bcrypt.gensalt())
        return None
    
    # Constant-time password verification
    if bcrypt.checkpw(password.encode('utf-8'), user.password_hash):
        logging.info(f"Successful login: {username}")
        return create_session(user)
    
    logging.warning(f"Failed login attempt: {username}")
    return None
```

**Summary:**
- ❌ **DO NOT MERGE**: Critical security vulnerabilities must be fixed first
- 🔴 2 Critical issues (SQL injection, plain text password)
- 🟡 3 Medium issues (validation, timing attack, logging)
- ✅ After fixes, this will be production-ready
```

### Example 2: React Component Review

**User Request:**
```

function UserList() {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);
  
  return (
    <div>
      {users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}
```

**Your Response:**
```
I'll review this React component for best practices and potential issues.

🟡 **MEDIUM PRIORITY ISSUES:**

1. **Missing Error Handling** (Lines 5-7)
   - **Issue**: No error handling for failed API requests
   - **Impact**: User sees nothing when API fails
   - **Fix**: Add error state and handling

2. **No Loading State** (Lines 5-7)
   - **Issue**: No indication while data is loading
   - **Impact**: Poor UX, empty screen initially
   - **Fix**: Add loading state

3. **Memory Leak Risk** (Lines 5-7)
   - **Issue**: No cleanup if component unmounts during fetch
   - **Impact**: Potential state update on unmounted component warning
   - **Fix**: Use AbortController or cleanup flag

4. **Missing PropTypes/TypeScript**
   - **Issue**: No type checking
   - **Fix**: Add TypeScript or PropTypes

🟢 **GOOD PRACTICES OBSERVED:**

- ✅ Proper use of hooks (useState, useEffect)
- ✅ Correct dependency array in useEffect
- ✅ Proper key prop in list rendering
- ✅ Clean, readable code structure

**Recommended Implementation:**
```typescript
import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const abortController = new AbortController();
    
    async function fetchUsers() {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch('/api/users', {
          signal: abortController.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message || 'Failed to fetch users');
          console.error('Error fetching users:', err);
        }
      } finally {
        setLoading(false);
      }
    }
    
    fetchUsers();
    
    return () => {
      abortController.abort();
    };
  }, []);
  
  if (loading) {
    return <div className="loading">Loading users...</div>;
  }
  
  if (error) {
    return (
      <div className="error">
        Error: {error}
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }
  
  if (users.length === 0) {
    return <div className="empty">No users found</div>;
  }
  
  return (
    <div className="user-list">
      {users.map(user => (
        <div key={user.id} className="user-card">
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
}

export default UserList;
```

**Testing Recommendations:**
```typescript
// UserList.test.tsx
describe('UserList', () => {
  it('should show loading state initially', () => {
    render(<UserList />);
    expect(screen.getByText('Loading users...')).toBeInTheDocument();
  });
  
  it('should display users after successful fetch', async () => {
    const mockUsers = [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' }
    ];
    
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: async () => mockUsers
    });
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument();
      expect(screen.getByText('Bob')).toBeInTheDocument();
    });
  });
  
  it('should show error message on fetch failure', async () => {
    global.fetch = jest.fn().mockRejectedValue(new Error('Network error'));
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/Error:/)).toBeInTheDocument();
    });
  });
});
```

**Summary:**
- ✅ **CAN MERGE WITH IMPROVEMENTS**: No blocking issues
- 🟡 4 Medium issues (error handling, loading state, cleanup, types)
- ✅ Good foundation with proper React patterns
- 📝 Add tests before merging to production
```

## Output Format

Your code reviews should always follow this structure:

1. **Issue Classification** with emoji indicators:
   - 🔴 Critical (security, data loss, crashes)
   - 🟡 Medium (quality, performance, maintenance)
   - 🔵 Low (style, minor improvements)
   - 🟢 Good practices observed

2. **Each Issue Contains**:
   - **Issue**: Clear description
   - **Impact**: Why it matters
   - **Fix**: Specific solution with code example

3. **Final Summary**:
   - Merge recommendation (✅ Can merge / ⚠️ Needs fixes / ❌ Do not merge)
   - Count of issues by severity
   - Overall assessment
   - Next steps

## Limitations

- I do not have access to the full codebase, only the code shown
- I cannot run tests or execute code
- For complex architectural decisions, suggest discussing with the team
- For domain-specific business logic, defer to domain experts
- I provide suggestions, but final decisions rest with the team
~~~

### Agent 호출 방식

설치된 Agent는 두 가지 방식으로 호출할 수 있습니다:

**방식 1: 멘션(@) 사용**
```
```

**방식 2: 명시적 지정**
```
I need a code review. Use the code-reviewer agent.
```

Claude는 `.claude/agents/code-reviewer.md` 파일의 내용을 컨텍스트로 로드하여, 해당 Agent의 역할과 지침에 따라 응답합니다.

## Command 구현 상세

### Frontmatter 필수 사용

Anthropic 공식 문서와 Issue #65에 따르면, Command는 **반드시 frontmatter**를 사용해야 합니다. 그렇지 않으면 전체 마크다운 내용이 API 요청에 포함되어 불필요한 토큰을 소비합니다.

### 올바른 Command 구조

~~~markdown
---
name: generate-tests
description: Automatically generate unit tests for code
category: testing
version: 1.0.0
author: Your Name
---

Analyze the provided code and generate comprehensive unit tests.

## Requirements

- Use the project's existing test framework (Jest, pytest, etc.)
- Cover happy path, edge cases, and error conditions
- Include setup and teardown if needed
- Follow naming conventions: test_*, should_*, it_should_*
- Aim for 80%+ code coverage

## Test Structure

For each function/method:
1. Test normal behavior
2. Test boundary conditions
3. Test error cases
4. Test interaction with dependencies (use mocks)

## Output Format

Provide:
- Test file name following project conventions
- Complete test code ready to run
- Brief explanation of what each test covers

## Example

For a function like:
```python
def calculate_discount(price, discount_percent):
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be 0-100")
    return price * (1 - discount_percent / 100)
```

Generate tests like:
```python
import pytest
from calculator import calculate_discount

def test_calculate_discount_normal_case():
    assert calculate_discount(100, 10) == 90.0
    
def test_calculate_discount_no_discount():
    assert calculate_discount(100, 0) == 100.0
    
def test_calculate_discount_full_discount():
    assert calculate_discount(100, 100) == 0.0
    
def test_calculate_discount_invalid_negative():
    with pytest.raises(ValueError):
        calculate_discount(100, -5)
        
def test_calculate_discount_invalid_over_100():
    with pytest.raises(ValueError):
        calculate_discount(100, 150)
```
~~~

### 복잡한 Command 예제: CI/CD 자동화

~~~markdown
---
name: run-ci
description: Run complete CI pipeline including tests, lint, build
category: testing
version: 2.1.0
dependencies:
  - test-framework
  - linter
  - build-tools
---

Execute the complete CI pipeline as it would run in the CI/CD system.

## Execution Steps

1. **Environment Check**
   - Verify all required tools are installed
   - Check environment variables
   - Validate configuration files

2. **Dependency Installation**
   - Install/update project dependencies
   - Verify lockfile integrity

3. **Code Quality**
   - Run linter (ESLint, Pylint, etc.)
   - Check code formatting (Prettier, Black, etc.)
   - Run type checker (TypeScript, mypy, etc.)

4. **Testing**
   - Run unit tests
   - Generate coverage report
   - Run integration tests if available
   - Run E2E tests if configured

5. **Build**
   - Compile/bundle the project
   - Verify build artifacts
   - Check bundle size

6. **Security**
   - Scan dependencies for vulnerabilities
   - Check for exposed secrets

## Failure Handling

If any step fails:
- Stop execution immediately
- Report which step failed
- Show relevant error messages
- Suggest fixes for common issues

## Success Criteria

All of the following must pass:
- ✅ Zero linting errors
- ✅ All tests passing
- ✅ Code coverage ≥ 80% (or project threshold)
- ✅ Build completes successfully
- ✅ No high-severity security vulnerabilities
- ✅ Type checking passes (if applicable)

## Output Format

Provide a structured summary:

```
CI Pipeline Results
===================

Environment: ✅ OK
Dependencies: ✅ OK
Linting: ✅ OK (0 errors, 3 warnings)
Type Check: ✅ OK
Tests: ✅ OK (145 passed, 0 failed)
Coverage: ✅ OK (87.3%)
Build: ✅ OK (bundle size: 234 KB)
Security: ⚠️  WARNING (2 moderate vulnerabilities)

Overall: ✅ PASS (1 warning)

Details:
- Linting warnings:
  src/utils/legacy.js:45 - Deprecated API usage
  src/components/Modal.jsx:12 - Missing PropTypes
  src/api/client.js:89 - Console.log statement

- Security warnings:
  axios@0.21.1 - Update to 0.21.2 (moderate)
  lodash@4.17.19 - Update to 4.17.21 (moderate)

Recommendations:
1. Address linting warnings before next release
2. Update dependencies: npm audit fix
3. Consider adding PropTypes or migrating to TypeScript
```

## Project-Specific Adaptation

Check for project configuration and adapt commands:

**package.json scripts:**
```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "webpack"
  }
}
```

**Detected setup:**
- Language: JavaScript
- Framework: React (inferred from dependencies)
- Test runner: Jest
- Linter: ESLint
- Bundler: Webpack

**Execution commands:**
```bash
npm run lint
npm run test -- --coverage
npm run build
npm audit
```

For Python projects with pytest:
```bash
pylint src/
pytest --cov=src tests/
python setup.py build
safety check
```

## Interactive Mode

If critical failures occur, ask user:

```
The CI pipeline failed at the Testing stage.

Error: 15 tests failed in authentication module

Options:
1. Show detailed test output
2. Skip tests and continue to build
3. Abort pipeline
4. Run only failed tests again

What would you like to do?
```
~~~

### Command 사용 방법

```bash
# Claude Code 세션에서
/generate-tests

# 또는 파일 지정
/generate-tests src/auth/login.py

# 또는 전체 디렉토리
/generate-tests src/
```

## MCP(Model Context Protocol) 구현 상세

### JSON 설정 파일 구조

MCP는 `.claude/mcp-servers.json` 파일에 정의됩니다:

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.github.com",
      "auth": {
        "type": "bearer",
        "token": "${GITHUB_TOKEN}"
      },
      "tools": [
        {
          "name": "create_issue",
          "description": "Create a new GitHub issue",
          "parameters": {
            "title": "string",
            "body": "string",
            "labels": "array",
            "assignees": "array"
          }
        },
        {
          "name": "list_pull_requests",
          "description": "List pull requests in a repository",
          "parameters": {
            "state": "enum[open,closed,all]",
            "sort": "enum[created,updated,popularity]",
            "direction": "enum[asc,desc]"
          }
        },
        {
          "name": "get_commit_history",
          "description": "Get commit history for a repository",
          "parameters": {
            "since": "date",
            "until": "date",
            "author": "string"
          }
        }
      ],
      "rateLimits": {
        "requestsPerHour": 5000,
        "requestsPerMinute": 60
      },
      "timeout": 30000,
      "retries": 3
    },
    
    "postgresql": {
      "url": "http://localhost:5432",
      "auth": {
        "type": "basic",
        "username": "${DB_USER}",
        "password": "${DB_PASSWORD}"
      },
      "tools": [
        {
          "name": "execute_query",
          "description": "Execute a SQL query and return results",
          "parameters": {
            "query": "string",
            "params": "array"
          },
          "security": {
            "readOnly": true,
            "allowedTables": ["users", "products", "orders"],
            "forbiddenKeywords": ["DROP", "DELETE", "TRUNCATE"]
          }
        },
        {
          "name": "get_schema",
          "description": "Get database schema information",
          "parameters": {
            "table": "string"
          }
        },
        {
          "name": "explain_query",
          "description": "Get query execution plan",
          "parameters": {
            "query": "string"
          }
        }
      ],
      "connectionPool": {
        "min": 2,
        "max": 10,
        "idleTimeout": 30000
      }
    },
    
    "slack": {
      "url": "https://slack.com/api",
      "auth": {
        "type": "bearer",
        "token": "${SLACK_BOT_TOKEN}"
      },
      "tools": [
        {
          "name": "post_message",
          "description": "Post a message to a Slack channel",
          "parameters": {
            "channel": "string",
            "text": "string",
            "blocks": "array",
            "thread_ts": "string"
          }
        },
        {
          "name": "upload_file",
          "description": "Upload a file to Slack",
          "parameters": {
            "channels": "array",
            "file": "binary",
            "title": "string",
            "initial_comment": "string"
          }
        },
        {
          "name": "list_channels",
          "description": "List all channels the bot has access to",
          "parameters": {
            "types": "enum[public,private,im,mpim]"
          }
        }
      ]
    }
  },
  
  "globalSettings": {
    "defaultTimeout": 30000,
    "enableLogging": true,
    "logLevel": "info",
    "logFile": ".claude/logs/mcp.log"
  }
}
```

### 커스텀 MCP 서버 구현

Node.js로 커스텀 MCP 서버를 만드는 완전한 예제:

```javascript
// mcp-timetracking-server.js
const express = require('express');
const morgan = require('morgan');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// 보안 및 미들웨어
app.use(helmet());
app.use(express.json());
app.use(morgan('combined'));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100 // 최대 100 요청
});
app.use(limiter);

// 인증 미들웨어
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token || token !== process.env.MCP_AUTH_TOKEN) {
    return res.status(401).json({ 
      error: 'Unauthorized',
      message: 'Valid authentication token required' 
    });
  }
  
  // 토큰에서 사용자 ID 추출 (실제로는 JWT 디코딩 등)
  req.userId = 'user-from-token';
  next();
};

// 모든 MCP 엔드포인트에 인증 적용
app.use('/mcp', authenticate);

// MCP 서버 메타데이터
app.get('/mcp/info', (req, res) => {
  res.json({
    name: "timetracking-mcp",
    version: "1.0.0",
    description: "Time tracking and project management integration",
    capabilities: [
      "time_entry",
      "time_query",
      "project_list",
      "report_generation"
    ],
    author: "Your Organization",
    documentation: "https://docs.yourorg.com/mcp/timetracking"
  });
});

// 도구 1: 작업 시간 기록
app.post('/mcp/tools/time_entry', async (req, res) => {
  const { project_id, duration, description, date } = req.body;
  
  try {
    // 입력 검증
    if (!project_id || !duration || !description) {
      return res.status(400).json({
        success: false,
        error: 'Missing required fields',
        required: ['project_id', 'duration', 'description']
      });
    }
    
    if (duration <= 0 || duration > 24) {
      return res.status(400).json({
        success: false,
        error: 'Duration must be between 0 and 24 hours'
      });
    }
    
    // 내부 API 호출
    const result = await internalTimeAPI.createEntry({
      user_id: req.userId,
      project_id,
      duration,
      description,
      date: date || new Date().toISOString(),
      created_at: new Date().toISOString()
    });
    
    // 감사 로그
    await auditLog.log({
      action: 'TIME_ENTRY_CREATED',
      user_id: req.userId,
      project_id,
      duration,
      timestamp: new Date().toISOString()
    });
    
    res.json({
      success: true,
      entry_id: result.id,
      message: `Logged ${duration} hours to ${result.project_name}`,
      data: {
        id: result.id,
        project_id,
        project_name: result.project_name,
        duration,
        description,
        date: result.date
      }
    });
    
  } catch (error) {
    console.error('Error creating time entry:', error);
    
    res.status(500).json({
      success: false,
      error: error.message || 'Internal server error',
      error_code: error.code || 'UNKNOWN_ERROR'
    });
  }
});

// 도구 2: 작업 시간 조회
app.post('/mcp/tools/time_query', async (req, res) => {
  const { start_date, end_date, project_id, group_by } = req.body;
  
  try {
    // 날짜 검증
    const start = new Date(start_date || Date.now() - 7 * 24 * 60 * 60 * 1000);
    const end = new Date(end_date || Date.now());
    
    if (start > end) {
      return res.status(400).json({
        success: false,
        error: 'start_date must be before end_date'
      });
    }
    
    // 데이터 조회
    const entries = await internalTimeAPI.queryEntries({
      user_id: req.userId,
      start_date: start.toISOString(),
      end_date: end.toISOString(),
      project_id: project_id || null
    });
    
    // 데이터 집계
    let summary = {
      total_hours: 0,
      total_entries: entries.length,
      period: {
        start: start.toISOString().split('T')[0],
        end: end.toISOString().split('T')[0]
      },
      breakdown: {}
    };
    
    if (group_by === 'project') {
      entries.forEach(entry => {
        const key = entry.project_name;
        if (!summary.breakdown[key]) {
          summary.breakdown[key] = {
            hours: 0,
            entries: 0
          };
        }
        summary.breakdown[key].hours += entry.duration;
        summary.breakdown[key].entries += 1;
        summary.total_hours += entry.duration;
      });
    } else if (group_by === 'date') {
      entries.forEach(entry => {
        const key = entry.date.split('T')[0];
        if (!summary.breakdown[key]) {
          summary.breakdown[key] = {
            hours: 0,
            entries: 0
          };
        }
        summary.breakdown[key].hours += entry.duration;
        summary.breakdown[key].entries += 1;
        summary.total_hours += entry.duration;
      });
    } else {
      summary.total_hours = entries.reduce((sum, e) => sum + e.duration, 0);
      summary.entries = entries;
    }
    
    res.json({
      success: true,
      summary,
      raw_entries: group_by ? undefined : entries
    });
    
  } catch (error) {
    console.error('Error querying time entries:', error);
    
    res.status(500).json({
      success: false,
      error: error.message || 'Internal server error'
    });
  }
});

// 도구 3: 프로젝트 목록
app.post('/mcp/tools/project_list', async (req, res) => {
  const { active_only = true, include_archived = false } = req.body;
  
  try {
    const projects = await internalTimeAPI.listProjects({
      user_id: req.userId,
      active: active_only,
      include_archived
    });
    
    res.json({
      success: true,
      count: projects.length,
      projects: projects.map(p => ({
        id: p.id,
        name: p.name,
        client: p.client,
        status: p.status,
        budget_hours: p.budget_hours,
        spent_hours: p.spent_hours,
        budget_remaining: p.budget_hours - p.spent_hours
      }))
    });
    
  } catch (error) {
    console.error('Error listing projects:', error);
    
    res.status(500).json({
      success: false,
      error: error.message || 'Internal server error'
    });
  }
});

// 도구 4: 리포트 생성
app.post('/mcp/tools/generate_report', async (req, res) => {
  const { 
    start_date, 
    end_date, 
    format = 'markdown',
    include_details = false 
  } = req.body;
  
  try {
    // 데이터 수집
    const entries = await internalTimeAPI.queryEntries({
      user_id: req.userId,
      start_date,
      end_date
    });
    
    // 리포트 생성
    let report = '';
    
    if (format === 'markdown') {
      report = generateMarkdownReport(entries, { start_date, end_date, include_details });
    } else if (format === 'csv') {
      report = generateCSVReport(entries);
    } else if (format === 'json') {
      report = JSON.stringify(entries, null, 2);
    }
    
    res.json({
      success: true,
      format,
      report,
      generated_at: new Date().toISOString()
    });
    
  } catch (error) {
    console.error('Error generating report:', error);
    
    res.status(500).json({
      success: false,
      error: error.message || 'Internal server error'
    });
  }
});

// 리포트 생성 헬퍼 함수
function generateMarkdownReport(entries, options) {
  const { start_date, end_date, include_details } = options;
  
  let report = `# Time Tracking Report\n\n`;
  report += `**Period:** ${start_date} to ${end_date}\n\n`;
  
  const totalHours = entries.reduce((sum, e) => sum + e.duration, 0);
  report += `**Total Hours:** ${totalHours.toFixed(2)}\n\n`;
  
  // 프로젝트별 집계
  const byProject = {};
  entries.forEach(entry => {
    if (!byProject[entry.project_name]) {
      byProject[entry.project_name] = {
        hours: 0,
        entries: []
      };
    }
    byProject[entry.project_name].hours += entry.duration;
    byProject[entry.project_name].entries.push(entry);
  });
  
  report += `## Breakdown by Project\n\n`;
  Object.keys(byProject).forEach(project => {
    const data = byProject[project];
    report += `### ${project}\n`;
    report += `- Hours: ${data.hours.toFixed(2)}\n`;
    report += `- Entries: ${data.entries.length}\n\n`;
    
    if (include_details) {
      report += `#### Details\n`;
      data.entries.forEach(entry => {
        report += `- ${entry.date}: ${entry.duration}h - ${entry.description}\n`;
      });
      report += `\n`;
    }
  });
  
  return report;
}

function generateCSVReport(entries) {
  let csv = 'Date,Project,Duration,Description\n';
  entries.forEach(entry => {
    csv += `${entry.date},${entry.project_name},${entry.duration},"${entry.description}"\n`;
  });
  return csv;
}

// 에러 핸들러
app.use((err, req, res, next) => {
  console.error('Unhandled error:', err);
  
  res.status(500).json({
    success: false,
    error: 'Internal server error',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
});

// 서버 시작
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`MCP Time Tracking Server running on port ${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});

// 정상 종료 처리
process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
  });
});
```

### MCP 서버 등록

```json
// .claude/mcp-servers.json
{
  "mcpServers": {
    "timetracking": {
      "url": "http://localhost:3000",
      "auth": {
        "type": "bearer",
        "token": "${TIMETRACKING_TOKEN}"
      },
      "tools": [
        {
          "name": "time_entry",
          "description": "Log work hours to a project",
          "parameters": {
            "project_id": "string",
            "duration": "number",
            "description": "string",
            "date": "string (optional)"
          }
        },
        {
          "name": "time_query",
          "description": "Query logged hours for a period",
          "parameters": {
            "start_date": "date",
            "end_date": "date",
            "project_id": "string (optional)",
            "group_by": "enum[project,date] (optional)"
          }
        },
        {
          "name": "project_list",
          "description": "List available projects",
          "parameters": {
            "active_only": "boolean (optional)",
            "include_archived": "boolean (optional)"
          }
        },
        {
          "name": "generate_report",
          "description": "Generate time tracking report",
          "parameters": {
            "start_date": "date",
            "end_date": "date",
            "format": "enum[markdown,csv,json] (optional)",
            "include_details": "boolean (optional)"
          }
        }
      ]
    }
  }
}
```

### MCP 사용 예제

```
User: "오늘 Project Alpha에 3시간 작업했어. 인증 기능 구현했다고 기록해줘."

Claude: [time_entry 도구를 호출합니다...]

POST /mcp/tools/time_entry
{
  "project_id": "alpha",
  "duration": 3,
  "description": "인증 기능 구현",
  "date": "2026-01-18"
}

"3시간이 Project Alpha에 기록되었습니다."

User: "이번 주에 총 몇 시간 일했는지 확인해줘."

Claude: [time_query 도구로 이번 주 기록을 조회합니다...]

POST /mcp/tools/time_query
{
  "start_date": "2026-01-13",
  "end_date": "2026-01-18",
  "group_by": "project"
}

"이번 주(1월 13일 - 1월 18일) 총 작업 시간: 28.5시간

프로젝트별 분석:
- Project Alpha: 15시간 (7건)
- Project Beta: 10시간 (5건)
- Internal Tools: 3.5시간 (3건)"
```

## Skills 구현 상세

### SKILL.md 파일 구조

Skills는 frontmatter + markdown 구조로, 가장 복잡한 컴포넌트입니다:

~~~markdown
---
name: pdf-processor
description: Comprehensive PDF processing with text extraction, metadata parsing, and page manipulation
allowed-tools: Bash, Read, Write, FileEdit
version: 2.0.0
author: Your Organization
model: sonnet-4  # 선택적: 특정 모델 지정
---

# PDF Processor Skill

Advanced PDF processing capabilities including text extraction, metadata manipulation, page splitting, and content analysis.

## Purpose

This skill enables Claude to:
- Extract text content from PDF documents
- Parse and modify PDF metadata
- Split and merge PDF files
- Analyze document structure
- Convert PDFs to other formats

## Progressive Disclosure

This skill uses progressive disclosure to load only necessary resources:

1. **Initial Request**: Load core processing logic
2. **Text Extraction**: Load extraction scripts when needed
3. **Metadata Operations**: Load metadata tools on demand
4. **Advanced Features**: Load splitting/merging only if requested

### File Organization

- `scripts/extract_text.py`: Text extraction logic
- `scripts/parse_metadata.py`: Metadata parsing
- `scripts/split_pdf.py`: Page splitting
- `scripts/merge_pdf.py`: PDF merging
- `references/pdf_structure.md`: PDF format reference
- `assets/default_metadata.json`: Metadata templates

## Usage

### Basic Text Extraction

To extract text from a PDF:

```
Use the pdf-processor skill to extract text from document.pdf
```

I will:
1. Run `scripts/extract_text.py document.pdf`
2. Return structured text with page numbers
3. Provide summary statistics

### Metadata Operations

To view or modify PDF metadata:

```
Use pdf-processor to show metadata for report.pdf
```

I will:
1. Run `scripts/parse_metadata.py report.pdf --show`
2. Display title, author, creation date, etc.
3. Offer to modify any field

### Advanced: Page Manipulation

To split a PDF:

```
Split pages 1-10 from large_document.pdf into separate_file.pdf
```

I will:
1. Validate page range
2. Run `scripts/split_pdf.py large_document.pdf 1-10 separate_file.pdf`
3. Confirm successful creation

## Implementation Details

### Text Extraction Process

1. Check if pypdf2 is installed
2. Open PDF file
3. Extract text page by page
4. Clean and format output
5. Return structured data

### Error Handling

Common errors and solutions:

- **Encrypted PDF**: Ask user for password
- **Scanned PDF (image)**: Suggest OCR processing
- **Corrupted file**: Report specific error
- **Missing dependencies**: Provide installation command

### Output Format

Text extraction returns:

```json
{
  "filename": "document.pdf",
  "pages": 15,
  "extracted_text": [
    {
      "page": 1,
      "text": "Page 1 content...",
      "word_count": 245
    },
    ...
  ],
  "metadata": {
    "title": "Document Title",
    "author": "Author Name",
    "created": "2026-01-18"
  },
  "summary": {
    "total_words": 3421,
    "total_characters": 18932,
    "average_words_per_page": 228
  }
}
```

## Examples

### Example 1: Extract and Summarize

**User Request:**
```
Extract text from meeting_notes.pdf and summarize key points
```

**Execution:**
1. Load extraction script
2. Extract all text
3. Analyze content
4. Generate summary

**Output:**
```
Extracted text from meeting_notes.pdf (5 pages, 1,234 words)

Key Points:
1. Q4 revenue target: $2.5M
2. New product launch scheduled for March
3. Team expansion: hiring 3 engineers
4. Customer retention rate: 94%
5. Action items assigned to department leads

Full text available in output.txt
```

### Example 2: Metadata Modification

**User Request:**
```
Update the author of report.pdf to "Engineering Team" and set creation date to today
```

**Execution:**
1. Load metadata script
2. Read current metadata
3. Update specified fields
4. Write back to PDF

**Output:**
```
Updated metadata for report.pdf:

Changes:
- Author: "John Doe" → "Engineering Team"
- Creation Date: "2025-11-15" → "2026-01-18"

Other metadata preserved:
- Title: "Q4 Engineering Report"
- Subject: "Performance Analysis"
- Keywords: "engineering, performance, Q4"

File successfully updated.
```

### Example 3: Complex Operation

**User Request:**
```
Split the first 20 pages of handbook.pdf, extract text from those pages, and create a summary document
```

**Execution:**
1. Split pages 1-20 to temp file
2. Extract text from temp file
3. Generate summary
4. Create new PDF with summary
5. Clean up temp files

**Output:**
```
Completed multi-step PDF processing:

Step 1: ✅ Split pages 1-20 from handbook.pdf
Step 2: ✅ Extracted text (4,532 words)
Step 3: ✅ Generated summary (350 words)
Step 4: ✅ Created summary.pdf

Output files:
- handbook_pages_1-20.pdf (extracted pages)
- summary.pdf (summary document)
- handbook_summary.txt (text version)

Summary highlights:
- Introduction and company overview
- Core values and mission statement
- Organizational structure
- Key policies and procedures
```

## Limitations

- Maximum PDF size: 100 MB
- Encrypted PDFs require password
- Scanned documents need OCR (not included)
- Complex forms may not extract perfectly
- Embedded multimedia not processed

## Dependencies

Required Python packages:
- pypdf2>=3.0.0
- pdfplumber>=0.9.0
- reportlab>=4.0.0 (for PDF creation)

Install via:
```bash
pip install pypdf2 pdfplumber reportlab --break-system-packages
```
~~~

### 디렉토리 구조

```
.claude/skills/pdf-processor/
├── SKILL.md                    # 메인 스킬 정의
├── LICENSE.txt                 # 라이센스
├── scripts/
│   ├── extract_text.py        # 텍스트 추출
│   ├── parse_metadata.py      # 메타데이터 파싱
│   ├── split_pdf.py           # PDF 분할
│   ├── merge_pdf.py           # PDF 병합
│   └── convert_to_text.py     # 텍스트 변환
├── references/
│   ├── pdf_structure.md       # PDF 구조 설명
│   ├── metadata_fields.md     # 메타데이터 필드 목록
│   └── best_practices.md      # 모범 사례
└── assets/
    ├── default_metadata.json  # 기본 메타데이터 템플릿
    └── output_template.md     # 출력 템플릿
```

### 실제 Python 스크립트 예제

```python
# scripts/extract_text.py
#!/usr/bin/env python3
"""
PDF Text Extraction Script
Extracts text content from PDF files with page-level granularity
"""

import sys
import json
from pathlib import Path

try:
    import PyPDF2
    import pdfplumber
except ImportError:
    print("Error: Required packages not installed")
    print("Install with: pip install pypdf2 pdfplumber --break-system-packages")
    sys.exit(1)

def extract_text_pypdf2(pdf_path):
    """Extract text using PyPDF2 (faster but less accurate)"""
    extracted_pages = []
    
    try:
        with open(pdf_path, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            
            for page_num, page in enumerate(pdf_reader.pages, start=1):
                text = page.extract_text()
                word_count = len(text.split())
                
                extracted_pages.append({
                    'page': page_num,
                    'text': text,
                    'word_count': word_count
                })
    
    except Exception as e:
        return {'error': str(e), 'method': 'pypdf2'}
    
    return {'pages': extracted_pages, 'method': 'pypdf2'}

def extract_text_pdfplumber(pdf_path):
    """Extract text using pdfplumber (slower but more accurate)"""
    extracted_pages = []
    
    try:
        with pdfplumber.open(pdf_path) as pdf:
            for page_num, page in enumerate(pdf.pages, start=1):
                text = page.extract_text()
                
                # Extract tables if present
                tables = page.extract_tables()
                
                word_count = len(text.split())
                
                page_data = {
                    'page': page_num,
                    'text': text,
                    'word_count': word_count,
                    'has_tables': len(tables) > 0,
                    'table_count': len(tables)
                }
                
                if tables:
                    page_data['tables'] = tables
                
                extracted_pages.append(page_data)
    
    except Exception as e:
        return {'error': str(e), 'method': 'pdfplumber'}
    
    return {'pages': extracted_pages, 'method': 'pdfplumber'}

def get_metadata(pdf_path):
    """Extract PDF metadata"""
    try:
        with open(pdf_path, 'rb') as file:
            pdf_reader = PyPDF2.PdfReader(file)
            metadata = pdf_reader.metadata
            
            return {
                'title': metadata.get('/Title', 'Unknown'),
                'author': metadata.get('/Author', 'Unknown'),
                'subject': metadata.get('/Subject', ''),
                'creator': metadata.get('/Creator', ''),
                'producer': metadata.get('/Producer', ''),
                'creation_date': metadata.get('/CreationDate', ''),
                'modification_date': metadata.get('/ModDate', '')
            }
    except Exception as e:
        return {'error': str(e)}

def calculate_summary(pages):
    """Calculate summary statistics"""
    total_words = sum(p['word_count'] for p in pages)
    total_chars = sum(len(p['text']) for p in pages)
    avg_words_per_page = total_words / len(pages) if pages else 0
    
    return {
        'total_pages': len(pages),
        'total_words': total_words,
        'total_characters': total_chars,
        'average_words_per_page': round(avg_words_per_page, 2)
    }

def main():
    if len(sys.argv) < 2:
        print("Usage: python extract_text.py <pdf_file> [--method pypdf2|pdfplumber]")
        sys.exit(1)
    
    pdf_path = sys.argv[1]
    method = 'pdfplumber'  # Default to more accurate method
    
    if len(sys.argv) > 2 and sys.argv[2] == '--method':
        method = sys.argv[3] if len(sys.argv) > 3 else 'pdfplumber'
    
    if not Path(pdf_path).exists():
        print(f"Error: File not found: {pdf_path}")
        sys.exit(1)
    
    # Extract text
    if method == 'pypdf2':
        result = extract_text_pypdf2(pdf_path)
    else:
        result = extract_text_pdfplumber(pdf_path)
    
    if 'error' in result:
        print(f"Error extracting text: {result['error']}")
        sys.exit(1)
    
    # Get metadata
    metadata = get_metadata(pdf_path)
    
    # Calculate summary
    summary = calculate_summary(result['pages'])
    
    # Compile final output
    output = {
        'filename': Path(pdf_path).name,
        'extracted_text': result['pages'],
        'metadata': metadata,
        'summary': summary,
        'extraction_method': result['method']
    }
    
    # Output as JSON
    print(json.dumps(output, indent=2, ensure_ascii=False))

if __name__ == '__main__':
    main()
```

### Progressive Disclosure 구현

Progressive Disclosure는 필요한 순간에만 관련 파일을 로드하여 토큰을 절약하는 패턴입니다:

~~~markdown
---
name: advanced-pdf-skill
description: Advanced PDF processing with progressive disclosure
allowed-tools: Bash, Read, Write
version: 3.0.0
---

# Advanced PDF Processing Skill

## Progressive Disclosure Strategy

Information is loaded in stages based on user requests:

### Stage 1: Initial Request (Always Loaded)
- Core skill description
- Available operations overview
- Basic usage instructions
**Token Cost: ~500 tokens**

### Stage 2: Text Extraction (Loaded on demand)
**Trigger**: User asks to extract text
**Files Loaded**:
- `scripts/extract_text.py`
- `references/extraction_methods.md`
**Token Cost: ~1,200 tokens**

### Stage 3: Metadata Operations (Loaded on demand)
**Trigger**: User asks about metadata
**Files Loaded**:
- `scripts/parse_metadata.py`
- `references/metadata_fields.md`
**Token Cost: ~800 tokens**

### Stage 4: Advanced Operations (Loaded on demand)
**Trigger**: User asks to split/merge/convert
**Files Loaded**:
- `scripts/split_pdf.py`
- `scripts/merge_pdf.py`
- `scripts/convert_pdf.py`
- `references/advanced_techniques.md`
**Token Cost: ~2,000 tokens**

## Implementation

When user makes a request:

1. **Identify Operation Type**
   ```
   User: "Extract text from document.pdf"
   → Trigger: Text Extraction (Stage 2)
   ```

2. **Load Only Necessary Files**
   ```
   Load: scripts/extract_text.py
   Load: references/extraction_methods.md
   Skip: All metadata and advanced files
   ```

3. **Execute and Return**
   ```
   Run extraction script
   Return formatted results
   ```

4. **Unload After Use**
   ```
   Once task complete, release loaded context
   Ready for next operation
   ```

## Workflow Example

**First Request:**
```
User: "What can you do with PDFs?"
Claude loads: SKILL.md only (~500 tokens)
Response: Overview of capabilities
```

**Second Request:**
```
User: "Extract text from report.pdf"
Claude loads: extract_text.py + extraction_methods.md (~1,700 total tokens)
Executes: Text extraction
Response: Extracted text and summary
```

**Third Request:**
```
User: "Now split the first 10 pages"
Claude loads: split_pdf.py + advanced_techniques.md (~2,000 additional tokens)
Executes: PDF splitting
Response: Confirmation and file location
```

**Total Token Usage: ~4,200 tokens**
**Traditional Approach (load everything): ~15,000 tokens**
**Savings: 72%**
~~~

## 설치 메커니즘 상세

### CLI 도구 작동 방식

```javascript
// cli-tool/src/index.js (간소화된 버전)

const axios = require('axios');
const fs = require('fs-extra');
const path = require('path');

const GITHUB_RAW_BASE = 'https://raw.githubusercontent.com/davila7/claude-code-templates/main';
const API_BASE = 'https://aitmpl.com/api';

async function installComponent(type, name, options = {}) {
  console.log(`Installing ${type}: ${name}`);
  
  try {
    // 1. 컴포넌트 정보 조회
    const componentInfo = await getComponentInfo(type, name);
    
    if (!componentInfo) {
      throw new Error(`Component not found: ${type}/${name}`);
    }
    
    // 2. GitHub에서 파일 다운로드
    const url = `${GITHUB_RAW_BASE}/cli-tool/components/${type}/${componentInfo.path}`;
    const response = await axios.get(url);
    
    // 3. 로컬 경로 결정 (카테고리 제거)
    const filename = path.basename(componentInfo.path);
    const targetDir = `.claude/${type}`;
    const targetPath = path.join(targetDir, filename);
    
    // 4. 디렉토리 생성
    await fs.ensureDir(targetDir);
    
    // 5. 파일 저장
    await fs.writeFile(targetPath, response.data);
    
    // 6. 추가 파일 처리 (Skills의 경우)
    if (type === 'skills') {
      await installSkillAssets(name, componentInfo);
    }
    
    // 7. 다운로드 통계 전송
    if (!options.dryRun) {
      await trackDownload(type, name, componentInfo.path);
    }
    
    console.log(`✅ Installed: ${targetPath}`);
    return { success: true, path: targetPath };
    
  } catch (error) {
    console.error(`❌ Failed to install ${type}/${name}:`, error.message);
    return { success: false, error: error.message };
  }
}

async function getComponentInfo(type, name) {
  // components.json에서 컴포넌트 정보 조회
  const catalogUrl = `${GITHUB_RAW_BASE}/docs/components.json`;
  const response = await axios.get(catalogUrl);
  const catalog = response.data;
  
  const components = catalog[type + 's'] || []; // 'agent' → 'agents'
  return components.find(c => c.name === name);
}

async function installSkillAssets(skillName, componentInfo) {
  // Skill의 경우 scripts/, references/, assets/ 디렉토리도 다운로드
  const skillDir = `.claude/skills/${skillName}`;
  await fs.ensureDir(skillDir);
  
  const subdirs = ['scripts', 'references', 'assets'];
  
  for (const subdir of subdirs) {
    const url = `${GITHUB_RAW_BASE}/cli-tool/components/skills/${componentInfo.category}/${skillName}/${subdir}`;
    
    try {
      // 디렉토리 내 파일 목록 가져오기
      const files = await listGitHubDirectory(url);
      
      for (const file of files) {
        const fileUrl = `${url}/${file}`;
        const targetPath = path.join(skillDir, subdir, file);
        
        const response = await axios.get(fileUrl);
        await fs.ensureFile(targetPath);
        await fs.writeFile(targetPath, response.data);
        
        // Python 스크립트는 실행 권한 부여
        if (file.endsWith('.py') || file.endsWith('.sh')) {
          await fs.chmod(targetPath, 0o755);
        }
      }
    } catch (error) {
      // 선택적 디렉토리이므로 에러 무시
      console.log(`  ℹ️  No ${subdir}/ directory for ${skillName}`);
    }
  }
}

async function trackDownload(type, name, path) {
  // Supabase로 다운로드 통계 전송
  try {
    await axios.post(`${API_BASE}/track-download-supabase`, {
      type,
      name,
      path,
      category: path.split('/')[0],
      timestamp: new Date().toISOString(),
      cli_version: require('../package.json').version
    });
  } catch (error) {
    // 통계 전송 실패는 무시
    console.log('  ℹ️  Could not track download (offline mode)');
  }
}

// 배치 설치
async function batchInstall(components) {
  const results = [];
  
  for (const component of components) {
    const { type, name } = component;
    const result = await installComponent(type, name);
    results.push({ type, name, ...result });
  }
  
  return results;
}

module.exports = {
  installComponent,
  batchInstall
};
```

### 컴포넌트 카탈로그 생성

```python
# generate_components_json.py (간소화된 버전)

import os
import json
import hashlib
from pathlib import Path

def generate_catalog():
    catalog = {
        'agents': [],
        'commands': [],
        'mcps': [],
        'settings': [],
        'hooks': [],
        'skills': []
    }
    
    components_dir = Path('cli-tool/components')
    
    # 각 컴포넌트 타입별로 스캔
    for component_type in catalog.keys():
        type_dir = components_dir / component_type
        
        if not type_dir.exists():
            continue
        
        # 카테고리별로 순회
        for category_dir in type_dir.iterdir():
            if not category_dir.is_dir():
                continue
            
            category_name = category_dir.name
            
            # 각 컴포넌트 파일 처리
            for component_file in category_dir.glob('*.md'):
                component_info = parse_component_file(
                    component_file,
                    component_type,
                    category_name
                )
                
                if component_info:
                    catalog[component_type].append(component_info)
    
    # 보안 검증 추가
    catalog = add_security_scores(catalog)
    
    # 다운로드 통계 추가
    catalog = add_download_stats(catalog)
    
    # 정렬 (인기도순)
    for component_type in catalog.keys():
        catalog[component_type].sort(
            key=lambda x: x.get('downloads', 0),
            reverse=True
        )
    
    # JSON 파일로 저장
    output_path = Path('docs/components.json')
    with open(output_path, 'w', encoding='utf-8') as f:
        json.dump(catalog, f, indent=2, ensure_ascii=False)
    
    print(f'✅ Generated catalog: {output_path}')
    print(f'   Total components: {sum(len(v) for v in catalog.values())}')

def parse_component_file(file_path, component_type, category):
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    # Frontmatter 파싱
    frontmatter = {}
    if content.startswith('---'):
        parts = content.split('---', 2)
        if len(parts) >= 3:
            frontmatter_text = parts[1]
            for line in frontmatter_text.strip().split('\n'):
                if ':' in line:
                    key, value = line.split(':', 1)
                    frontmatter[key.strip()] = value.strip()
    
    # 첫 번째 헤딩 추출 (제목)
    title = file_path.stem.replace('-', ' ').title()
    for line in content.split('\n'):
        if line.startswith('# '):
            title = line[2:].strip()
            break
    
    # 설명 추출 (첫 번째 단락)
    description = ''
    in_content = False
    for line in content.split('\n'):
        if line.startswith('---'):
            in_content = not in_content
            continue
        if in_content and line.strip() and not line.startswith('#'):
            description = line.strip()
            break
    
    # 상대 경로
    relative_path = f'{category}/{file_path.name}'
    
    # 파일 해시 (무결성 검증용)
    file_hash = hashlib.sha256(content.encode()).hexdigest()[:16]
    
    return {
        'name': file_path.stem,
        'title': title,
        'description': description or frontmatter.get('description', ''),
        'category': category.replace('-', ' ').title(),
        'path': relative_path,
        'version': frontmatter.get('version', '1.0.0'),
        'author': frontmatter.get('author', 'Community'),
        'hash': file_hash,
        'file_size': file_path.stat().st_size,
        'last_modified': file_path.stat().st_mtime
    }

def add_security_scores(catalog):
    # 보안 검증 결과 추가
    security_report_path = Path('cli-tool/security-report.json')
    
    if not security_report_path.exists():
        return catalog
    
    with open(security_report_path, 'r') as f:
        security_data = json.load(f)
    
    for component_type in catalog.keys():
        for component in catalog[component_type]:
            component_key = f"{component_type}/{component['path']}"
            
            if component_key in security_data:
                component['security_score'] = security_data[component_key]['score']
                component['security_issues'] = security_data[component_key]['issues']
            else:
                component['security_score'] = 100
                component['security_issues'] = []
    
    return catalog

def add_download_stats(catalog):
    # Supabase에서 다운로드 통계 가져오기 (실제로는 API 호출)
    # 여기서는 모의 데이터
    
    for component_type in catalog.keys():
        for component in catalog[component_type]:
            # 실제로는 DB 조회
            component['downloads'] = 0
            component['last_downloaded'] = None
    
    return catalog

if __name__ == '__main__':
    generate_catalog()
```

## Global Agents 구현

### Global Agent 생성 메커니즘

```javascript
// cli-tool/src/global-agents.js

const fs = require('fs-extra');
const path = require('path');
const os = require('os');

async function createGlobalAgent(agentName) {
  console.log(`Creating global agent: ${agentName}`);
  
  try {
    // 1. Agent 파일 다운로드
    const agentFile = await downloadAgent(agentName);
    
    // 2. 실행 가능한 스크립트 생성
    const binDir = getGlobalBinDir();
    const scriptPath = path.join(binDir, agentName);
    
    const script = generateAgentScript(agentName, agentFile);
    
    await fs.ensureDir(binDir);
    await fs.writeFile(scriptPath, script);
    await fs.chmod(scriptPath, 0o755); // 실행 권한
    
    // 3. PATH에 추가 (필요한 경우)
    await ensureInPath(binDir);
    
    console.log(`✅ Global agent created: ${agentName}`);
    console.log(`   Usage: ${agentName} "your prompt here"`);
    
    return { success: true, path: scriptPath };
    
  } catch (error) {
    console.error(`❌ Failed to create global agent:`, error.message);
    return { success: false, error: error.message };
  }
}

function getGlobalBinDir() {
  // OS별 글로벌 bin 디렉토리
  if (os.platform() === 'win32') {
    return path.join(os.homedir(), 'AppData', 'Local', 'claude-agents');
  } else {
    return path.join(os.homedir(), '.local', 'bin');
  }
}

function generateAgentScript(agentName, agentFilePath) {
  // Unix 셸 스크립트 (Windows는 .cmd 파일)
  if (os.platform() === 'win32') {
    return generateWindowsScript(agentName, agentFilePath);
  }
  
  return `#!/usr/bin/env node

/**
 * Global Agent: ${agentName}
 * Auto-generated by Claude Code Templates
 */

const { spawn } = require('child_process');
const path = require('path');
const fs = require('fs');

const AGENT_FILE = '${agentFilePath}';
const AGENT_NAME = '${agentName}';

async function main() {
  // 사용자 입력 수집
  const userPrompt = process.argv.slice(2).join(' ');
  
  if (!userPrompt) {
    console.error('Usage: ${agentName} "your prompt here"');
    console.error('Example: ${agentName} "Review this code for security issues"');
    process.exit(1);
  }
  
  // Agent 파일 존재 확인
  if (!fs.existsSync(AGENT_FILE)) {
    console.error(\`Error: Agent file not found: \${AGENT_FILE}\`);
    console.error('Run: npx claude-code-templates@latest --update-agent ${agentName}');
    process.exit(1);
  }
  
  // Claude Code 실행
  const claude = spawn('claude', [
    '--agent', AGENT_NAME,
    '--prompt', userPrompt
  ], {
    stdio: 'inherit',
    env: {
      ...process.env,
      CLAUDE_AGENT_FILE: AGENT_FILE
    }
  });
  
  claude.on('error', (error) => {
    console.error('Error executing Claude Code:', error.message);
    console.error('Make sure Claude Code is installed: https://claude.ai/code');
    process.exit(1);
  });
  
  claude.on('close', (code) => {
    process.exit(code);
  });
}

main().catch((error) => {
  console.error('Unexpected error:', error);
  process.exit(1);
});
`;
}

function generateWindowsScript(agentName, agentFilePath) {
  return `@echo off
REM Global Agent: ${agentName}
REM Auto-generated by Claude Code Templates

node "${agentFilePath}" %*
`;
}

async function ensureInPath(binDir) {
  const currentPath = process.env.PATH || '';
  
  if (currentPath.includes(binDir)) {
    return; // 이미 PATH에 있음
  }
  
  console.log(`\n⚠️  Please add ${binDir} to your PATH:`);
  
  if (os.platform() === 'win32') {
    console.log(`   setx PATH "%PATH%;${binDir}"`);
  } else {
    const shell = process.env.SHELL || '/bin/bash';
    const rcFile = shell.includes('zsh') ? '~/.zshrc' : '~/.bashrc';
    
    console.log(`   echo 'export PATH="$PATH:${binDir}"' >> ${rcFile}`);
    console.log(`   source ${rcFile}`);
  }
  
  console.log('');
}

module.exports = {
  createGlobalAgent,
  listGlobalAgents,
  updateGlobalAgent,
  removeGlobalAgent
};
```

### 사용 예제

```bash
# Global agent 생성
npx claude-code-templates@latest --create-agent code-reviewer

# 어디서든 사용
cd ~/projects/my-app
code-reviewer "Review this authentication function"

# 여러 agent 생성
npx claude-code-templates@latest --create-agent security-auditor
npx claude-code-templates@latest --create-agent performance-optimizer

# 사용
security-auditor "Scan this codebase for vulnerabilities"
performance-optimizer "Analyze bundle size and suggest improvements"

# 목록 보기
npx claude-code-templates@latest --list-agents

# 업데이트
npx claude-code-templates@latest --update-agent code-reviewer

# 제거
npx claude-code-templates@latest --remove-agent code-reviewer
```

## 결론

이 문서는 Claude Code Templates의 실제 구현 방식을 기술적으로 상세히 다루었습니다. 각 컴포넌트 타입(Agent, Command, MCP, Skills)의 파일 구조, 작성 방법, 그리고 내부 메커니즘을 실제 코드 예제와 함께 설명했습니다.

핵심 포인트:
- **Agents**: Markdown 파일, 역할과 지침 정의
- **Commands**: Frontmatter 필수, 슬래시 명령어
- **MCPs**: JSON 설정, Node.js 서버 구현
- **Skills**: 가장 복잡, Progressive Disclosure 패턴
- **Global Agents**: 전역 실행 가능, CLI 래퍼
- **설치 메커니즘**: GitHub → 로컬, 카테고리 평탄화
- **카탈로그 생성**: Python 스크립트, 자동화

이 정보를 바탕으로 커스텀 컴포넌트를 만들거나, 기존 컴포넌트를 수정하거나, 전체 시스템의 작동 방식을 이해할 수 있습니다.

**작성 일자: 2026-01-18**
