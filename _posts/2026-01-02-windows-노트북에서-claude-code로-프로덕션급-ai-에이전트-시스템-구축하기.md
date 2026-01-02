---
title: "Windows 노트북에서 Claude Code로 프로덕션급 AI 에이전트 시스템 구축하기"
date: 2026-01-02 09:00:00 +0900
categories: [AI,  Guide]
tags: [AI,  vibe-coding,  claude-code,  ai-agent-architecture,  Claude.write]
---


> **바이브 코딩(Vibe Coding)으로 7계층 아키텍처 완성하기**  
> Claude Code를 활용한 자연어 기반 개발 워크플로우

---

## 📋 목차

1. [환경 설정](#1-환경-설정)
2. [Claude Code 설치 및 설정](#2-claude-code-설치-및-설정)
3. [프로젝트 초기화](#3-프로젝트-초기화)
4. [계층별 바이브 코딩 가이드](#4-계층별-바이브-코딩-가이드)
5. [실전 구현 예제](#5-실전-구현-예제)
6. [디버깅 및 테스트](#6-디버깅-및-테스트)
7. [배포 준비](#7-배포-준비)
8. [트러블슈팅](#8-트러블슈팅)

---

## 1. 환경 설정

### 1.1 필수 소프트웨어 설치

#### Python 3.13 설치 (Windows)

```powershell
# 1. Python.org에서 Python 3.13 설치 프로그램 다운로드
# https://www.python.org/downloads/

# 2. 설치 시 "Add Python to PATH" 체크 필수!

# 3. 설치 확인
python --version
# Python 3.13.x

pip --version
# pip 24.x
```

#### Git 설치

```powershell
# Git for Windows 다운로드
# https://git-scm.com/download/win

# 설치 후 확인
git --version
# git version 2.x
```

#### Docker Desktop 설치 (선택)

```powershell
# Docker Desktop for Windows 다운로드
# https://www.docker.com/products/docker-desktop

# 설치 후 확인
docker --version
docker-compose --version
```

#### VSCode 설치 (권장)

```powershell
# Visual Studio Code 다운로드
# https://code.visualstudio.com/

# 필수 확장 프로그램:
# - Python
# - Pylance
# - Docker
# - GitLens
```

### 1.2 Windows 터미널 설정

#### PowerShell 7 설치 (권장)

```powershell
# Microsoft Store에서 "PowerShell" 검색 후 설치
# 또는 winget 사용:
winget install Microsoft.PowerShell

# 설치 확인
pwsh --version
```

#### Windows Terminal 설치

```powershell
# Microsoft Store에서 "Windows Terminal" 검색 후 설치
# 또는 winget 사용:
winget install Microsoft.WindowsTerminal
```

---

## 2. Claude Code 설치 및 설정

### 2.1 Claude Code CLI 설치

```powershell
# npm을 통한 설치 (Node.js 필요)
# Node.js 다운로드: https://nodejs.org/

# Claude Code 글로벌 설치
npm install -g @anthropic-ai/claude-code

# 설치 확인
claude-code --version
```

### 2.2 API 키 설정

```powershell
# 환경 변수 설정 (영구적)
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-ant-api03-...", "User")

# 현재 세션에만 적용
$env:ANTHROPIC_API_KEY = "sk-ant-api03-..."

# 확인
echo $env:ANTHROPIC_API_KEY
```

**또는 .env 파일 사용:**

```powershell
# 프로젝트 루트에 .env 파일 생성
New-Item -Path .env -ItemType File

# 내용 추가
ANTHROPIC_API_KEY=sk-ant-api03-...
OPENAI_API_KEY=sk-proj-...
"@ | Out-File -FilePath .env -Encoding UTF8
```

### 2.3 Claude Code 첫 실행

```powershell
# Claude Code 대화형 모드 시작
claude-code

# 또는 특정 작업 지시
claude-code "Create a FastAPI project structure"
```

### 2.4 Claude Code 주요 명령어

```powershell
# 도움말
claude-code --help

# 파일 컨텍스트와 함께 질문
claude-code -f app/main.py "이 파일의 보안 문제를 찾아줘"

# 여러 파일 컨텍스트
claude-code -f app/main.py -f app/models/*.py "FastAPI 라우터 추가해줘"

# 출력을 파일로 저장
claude-code "Create a Dockerfile" > Dockerfile

# 대화형 모드에서 이전 대화 이어가기
claude-code --continue

# 특정 모델 선택
claude-code --model claude-sonnet-4-20250514 "복잡한 아키텍처 설계해줘"
```

---

## 3. 프로젝트 초기화

### 3.1 프로젝트 디렉토리 생성

```powershell
# 프로젝트 폴더 생성
mkdir production-ai-agent
cd production-ai-agent

# Git 초기화
git init

# .gitignore 생성
claude-code "Create a comprehensive Python .gitignore for FastAPI project" > .gitignore
```

### 3.2 가상 환경 설정

```powershell
# Python 가상 환경 생성
python -m venv venv

# 가상 환경 활성화 (PowerShell)
.\venv\Scripts\Activate.ps1

# 또는 CMD
.\venv\Scripts\activate.bat

# pip 업그레이드
python -m pip install --upgrade pip

# uv 설치 (빠른 패키지 관리자)
pip install uv
```

### 3.3 바이브 코딩으로 pyproject.toml 생성

```powershell
# Claude Code를 사용한 자연어 요청
claude-code "
Create a pyproject.toml for a production-grade AI agent system with:
- FastAPI 0.121.0+
- LangGraph 1.0.5
- LangChain 1.0.5
- Mem0 1.0.0
- SQLModel, Pydantic, PostgreSQL drivers
- Prometheus, Langfuse integration
- Development dependencies: pytest, black, ruff

Include proper project metadata and dependency groups.
" > pyproject.toml
```

**생성된 pyproject.toml 확인 후 설치:**

```powershell
# 의존성 설치
uv pip install -e .

# 개발 의존성 포함
uv pip install -e ".[dev]"
```

### 3.4 프로젝트 구조 생성

```powershell
# Claude Code로 전체 구조 생성
claude-code "
Create a production-grade FastAPI project structure with:
- app/ (main application)
  - api/v1/ (versioned API)
  - core/ (config, security, logging)
  - models/ (database models)
  - schemas/ (Pydantic schemas)
  - services/ (business logic)
- tests/ (pytest tests)
- docker/ (Dockerfile, compose)
- scripts/ (helper scripts)

Create all necessary __init__.py files.
Show me the PowerShell commands to create this structure.
"
```

**Claude가 생성한 명령어 실행:**

```powershell
# 예시 출력 (Claude가 생성)
New-Item -ItemType Directory -Path app/api/v1, app/core, app/models, app/schemas, app/services, tests, docker, scripts
New-Item -ItemType File -Path app/__init__.py, app/api/__init__.py, app/api/v1/__init__.py
```

---

## 4. 계층별 바이브 코딩 가이드

### 4.1 Layer 1: 모듈러 코드베이스 구축

#### 바이브 코딩 프롬프트:

```powershell
claude-code -f pyproject.toml "
Create app/core/config.py with:
1. Environment-specific settings (dev, staging, prod)
2. Pydantic Settings for type safety
3. Load .env files based on APP_ENV
4. Database connection settings with pooling
5. JWT configuration
6. Rate limiting settings
7. LLM API keys management

Use latest Python 3.13 features and best practices.
" > app/core/config.py
```

#### 생성된 파일 리뷰 및 수정:

```powershell
# 파일 확인
code app/core/config.py

# 수정이 필요하면 Claude에게 요청
claude-code -f app/core/config.py "
이 설정 파일에서:
1. CORS 설정 추가
2. Prometheus 메트릭 설정 추가
3. Langfuse 연동 설정 추가

변경된 전체 파일 내용을 보여줘.
"
```

### 4.2 Layer 2: 데이터 지속성 계층

#### 베이스 모델 생성:

```powershell
claude-code "
Create app/models/base.py with SQLModel:
1. BaseModel with created_at, updated_at
2. Use UTC timezone
3. Add proper typing with Python 3.13
4. Include __repr__ for debugging
" > app/models/base.py
```

#### User 모델 생성:

```powershell
claude-code -f app/models/base.py "
Create app/models/user.py with:
1. User model inheriting from BaseModel
2. Email (unique, indexed)
3. Hashed password with bcrypt
4. verify_password and hash_password methods
5. Relationship to Session model
6. Proper SQLModel table=True configuration
" > app/models/user.py
```

#### Session 모델:

```powershell
claude-code -f app/models/base.py -f app/models/user.py "
Create app/models/session.py for chat sessions:
1. UUID primary key
2. Foreign key to User
3. Session name
4. Created timestamp
5. Relationship back to User
" > app/models/session.py
```

#### Pydantic 스키마:

```powershell
claude-code "
Create app/schemas/auth.py with:
1. UserCreate (email, password validation)
2. UserResponse (safe public fields)
3. Token (JWT token response)
4. Password strength validation (8+ chars, uppercase, number, special)
5. Use Pydantic v2 syntax
" > app/schemas/auth.py
```

### 4.3 Layer 3: 보안 계층

#### Rate Limiting:

```powershell
claude-code -f app/core/config.py "
Create app/core/limiter.py:
1. Initialize SlowAPI
2. Use settings for rate limits
3. Configure different limits per endpoint
4. Include error handlers
" > app/core/limiter.py
```

#### Sanitization:

```powershell
claude-code "
Create app/utils/sanitization.py with:
1. sanitize_string (HTML escape, script removal)
2. sanitize_email (format validation)
3. Remove null bytes
4. XSS prevention
5. Comprehensive docstrings
" > app/utils/sanitization.py
```

#### JWT Authentication:

```powershell
claude-code -f app/core/config.py "
Create app/utils/auth.py:
1. create_access_token (with JTI)
2. verify_token
3. Use python-jose
4. Include expiration handling
5. Add proper error handling
" > app/utils/auth.py
```

### 4.4 Layer 4: 서비스 계층

#### Database Service:

```powershell
claude-code -f app/core/config.py -f app/models/*.py "
Create app/services/database.py with:
1. DatabaseService class (singleton)
2. Connection pooling with QueuePool
3. CRUD operations for User, Session
4. Health check method
5. Async support where needed
6. Proper error handling
" > app/services/database.py
```

#### LLM Service with Fallback:

```powershell
claude-code "
Create app/services/llm.py with:
1. LLMRegistry for multiple models
2. LLMService with circuit breaker
3. Automatic retries with tenacity
4. Fallback to backup models
5. Support for gpt-4o, gpt-4o-mini
6. Bind tools support
" > app/services/llm.py
```

### 4.5 Layer 5: 멀티 에이전트 아키텍처

#### LangGraph Agent:

```powershell
claude-code -f app/services/llm.py "
Create app/core/langgraph/graph.py:
1. LangGraphAgent class
2. AsyncPostgresSaver for checkpointing
3. Mem0 integration for long-term memory
4. _chat and _tool_call nodes
5. get_response method for API
6. Streaming support
7. Memory update in background
" > app/core/langgraph/graph.py
```

#### Tool Integration:

```powershell
claude-code "
Create app/core/langgraph/tools/duckduckgo_search.py:
1. Import DuckDuckGoSearchResults
2. Configure with num_results=10
3. Error handling
4. Export for use in tools list
" > app/core/langgraph/tools/duckduckgo_search.py
```

#### Prompt Management:

```powershell
claude-code "
Create app/core/prompts/system.md:
A professional system prompt for an AI agent with:
1. Agent name and role
2. Instructions for behavior
3. Placeholders for {long_term_memory} and {current_date_and_time}
4. Clear guidelines

Then create app/core/prompts/__init__.py to load and format this prompt.
"
```

### 4.6 Layer 6: API 게이트웨이

#### Main Application:

```powershell
claude-code -f app/core/config.py "
Create app/main.py:
1. FastAPI app with lifespan context manager
2. CORS middleware
3. Prometheus metrics
4. Logging middleware
5. Exception handlers
6. Health check endpoints
7. Include API routers
" > app/main.py
```

#### Auth Endpoints:

```powershell
claude-code -f app/services/database.py -f app/schemas/auth.py -f app/utils/auth.py "
Create app/api/v1/auth.py:
1. /register endpoint
2. /login endpoint (OAuth2 compatible)
3. /session endpoint (create chat session)
4. /sessions endpoint (list user sessions)
5. get_current_user dependency
6. get_current_session dependency
7. Rate limiting decorators
" > app/api/v1/auth.py
```

#### Chat Endpoints:

```powershell
claude-code -f app/core/langgraph/graph.py -f app/api/v1/auth.py "
Create app/api/v1/chatbot.py:
1. POST /chat (standard request/response)
2. POST /chat/stream (SSE streaming)
3. GET /messages (fetch history)
4. DELETE /messages (clear history)
5. Use authentication dependencies
6. Rate limiting
7. Error handling
" > app/api/v1/chatbot.py
```

### 4.7 Layer 7: 관측성

#### Prometheus Metrics:

```powershell
claude-code "
Create app/core/metrics.py:
1. Define prometheus_client metrics
2. http_requests_total counter
3. http_request_duration_seconds histogram
4. llm_inference_duration_seconds histogram
5. db_connections gauge
6. setup_metrics function
" > app/core/metrics.py
```

#### Middleware:

```powershell
claude-code -f app/core/metrics.py "
Create app/core/middleware.py:
1. MetricsMiddleware (track latency, status)
2. LoggingContextMiddleware (bind user_id)
3. Use structlog for logging
4. Clear context after request
" > app/core/middleware.py
```

---

## 5. 실전 구현 예제

### 5.1 환경 파일 생성

```powershell
# .env.development 생성
claude-code "
Create .env.development with all necessary environment variables:
- APP_ENV=development
- DEBUG=true
- Database settings for local PostgreSQL
- OpenAI API key placeholder
- JWT secret
- Rate limits
- Langfuse settings
" > .env.development
```

### 5.2 Docker Compose 설정

```powershell
claude-code "
Create docker-compose.yml with:
1. PostgreSQL 16 with pgvector
2. FastAPI app service
3. Prometheus
4. Grafana
5. cAdvisor
6. Proper networking
7. Volume mounts
8. Health checks
" > docker-compose.yml
```

### 5.3 Dockerfile 생성

```powershell
claude-code -f pyproject.toml "
Create a multi-stage Dockerfile:
1. Use python:3.13.2-slim base
2. Install system dependencies
3. Copy pyproject.toml first (cache)
4. Install Python dependencies with uv
5. Copy application code
6. Create non-root user
7. Expose port 8000
8. ENTRYPOINT script
" > Dockerfile
```

### 5.4 전체 시스템 테스트

```powershell
# Docker 환경 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f app

# API 테스트
curl http://localhost:8000/health

# 또는 PowerShell로
Invoke-RestMethod -Uri http://localhost:8000/health
```

---

## 6. 디버깅 및 테스트

### 6.1 단위 테스트 작성

```powershell
# pytest 설정
claude-code "
Create pytest.ini with:
1. Test discovery patterns
2. Markers for slow tests
3. Coverage settings
4. Async support
" > pytest.ini

# 테스트 파일 생성
claude-code -f app/utils/auth.py "
Create tests/test_auth.py:
1. Test create_access_token
2. Test verify_token
3. Test token expiration
4. Use pytest fixtures
5. Mock settings
" > tests/test_auth.py
```

### 6.2 테스트 실행

```powershell
# 전체 테스트 실행
pytest

# 커버리지 포함
pytest --cov=app --cov-report=html

# 특정 테스트만
pytest tests/test_auth.py -v

# 병렬 실행
pytest -n auto
```

### 6.3 통합 테스트

```powershell
claude-code "
Create tests/test_integration.py:
1. Test full registration flow
2. Test login and session creation
3. Test chat interaction
4. Use httpx.AsyncClient
5. Test database cleanup
" > tests/test_integration.py
```

### 6.4 부하 테스트 스크립트

```powershell
claude-code "
Create tests/stress_test.py:
1. Simulate 100 concurrent users
2. Test login -> session -> chat flow
3. Use aiohttp
4. Measure latency
5. Report success rate
" > tests/stress_test.py
```

---

## 7. 배포 준비

### 7.1 환경별 설정 파일

```powershell
# 프로덕션 환경 파일
claude-code "
Create .env.production with strict settings:
- APP_ENV=production
- DEBUG=false
- Stronger JWT secret
- Tighter rate limits
- JSON logging
- Production database URL
" > .env.production
```

### 7.2 GitHub Actions CI/CD

```powershell
claude-code "
Create .github/workflows/deploy.yaml:
1. Trigger on push to main
2. Run tests
3. Build Docker image
4. Push to Docker Hub
5. Deploy to server
6. Use secrets for credentials
" > .github/workflows/deploy.yaml
```

### 7.3 헬스 체크 강화

```powershell
claude-code -f app/main.py "
Update /health endpoint to check:
1. Database connection
2. Redis (if used)
3. LLM API availability
4. Return detailed status
5. Use 503 if unhealthy
"
```

### 7.4 Makefile 생성

```powershell
claude-code "
Create Makefile for Windows (compatible with nmake or make):
1. install target (dependencies)
2. dev target (run development server)
3. test target (run tests)
4. docker-build target
5. docker-run-env target
6. clean target
" > Makefile
```

---

## 8. 트러블슈팅

### 8.1 일반적인 Windows 문제

#### 문제 1: PowerShell 실행 정책 오류

```powershell
# 오류 메시지:
# "이 시스템에서 스크립트를 실행할 수 없으므로..."

# 해결방법:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 확인
Get-ExecutionPolicy
```

#### 문제 2: Python 경로 문제

```powershell
# Python이 PATH에 없을 때
# 수동으로 PATH 추가:
$env:Path += ";C:\Users\YourName\AppData\Local\Programs\Python\Python313"

# 영구적으로 추가:
[Environment]::SetEnvironmentVariable(
    "Path",
    "$env:Path;C:\Users\YourName\AppData\Local\Programs\Python\Python313",
    "User"
)
```

#### 문제 3: 가상 환경 활성화 실패

```powershell
# PowerShell에서 스크립트 실행 허용
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process

# 그래도 안 되면 venv 재생성
Remove-Item -Recurse -Force venv
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### 8.2 Claude Code 관련 문제

#### 문제 1: API 키 인식 안 됨

```powershell
# 환경 변수 다시 확인
echo $env:ANTHROPIC_API_KEY

# 없으면 다시 설정
$env:ANTHROPIC_API_KEY = "sk-ant-api03-..."

# .env 파일 사용
claude-code --env-file .env "test command"
```

#### 문제 2: 생성된 코드가 불완전함

```powershell
# 더 구체적인 프롬프트 사용
claude-code "
Create app/main.py with FastAPI.
Requirements:
- Include ALL necessary imports
- Add CORS middleware
- Add exception handlers
- Include /health endpoint
- Use proper type hints
- Add comprehensive docstrings
- Show COMPLETE file, not snippets
"
```

#### 문제 3: 컨텍스트 길이 제한

```powershell
# 파일을 나눠서 작업
claude-code -f app/models/user.py "이 파일만 리뷰해줘"

# 또는 요약 요청
claude-code -f app/*.py "각 파일의 핵심 기능만 3줄로 요약해줘"
```

### 8.3 Docker 관련 문제

#### 문제 1: Docker Desktop이 시작되지 않음

```powershell
# WSL 2 확인
wsl --status

# WSL 2 설치
wsl --install

# Docker Desktop 재시작
```

#### 문제 2: 볼륨 마운트 권한 오류

```powershell
# Docker Desktop 설정에서 공유 드라이브 설정
# Settings > Resources > File Sharing
# 프로젝트 폴더가 있는 드라이브 추가
```

#### 문제 3: 포트 충돌

```powershell
# 사용 중인 포트 확인
netstat -ano | findstr :8000

# 프로세스 종료
taskkill /PID <PID> /F

# 또는 docker-compose.yml에서 포트 변경
ports:
  - "8001:8000"  # 호스트:컨테이너
```

---

## 9. 바이브 코딩 베스트 프랙티스

### 9.1 효과적인 프롬프트 작성법

#### ❌ 나쁜 예:

```powershell
claude-code "FastAPI 앱 만들어줘"
```

#### ✅ 좋은 예:

```powershell
claude-code "
Create app/main.py for a production FastAPI application:

Requirements:
1. FastAPI instance with title and version from settings
2. Lifespan context manager for startup/shutdown
3. CORS middleware with settings.ALLOWED_ORIGINS
4. Custom exception handlers for 422, 500
5. Include /health and / endpoints
6. Add API router with prefix from settings
7. Use Python 3.13 type hints
8. Add comprehensive docstrings

Include ALL imports at the top.
Show the COMPLETE file content.
"
```

### 9.2 반복 작업 자동화

```powershell
# 자주 사용하는 프롬프트를 함수로 저장
function New-FastAPIEndpoint {
    param(
        [string]$Name,
        [string]$Path
    )
    
    claude-code "
    Create $Path with:
    1. FastAPI APIRouter
    2. $Name CRUD endpoints (GET, POST, PUT, DELETE)
    3. Pydantic schemas for request/response
    4. Authentication dependencies
    5. Rate limiting
    6. Proper error handling
    7. Comprehensive docstrings
    "
}

# 사용 예:
New-FastAPIEndpoint -Name "Product" -Path "app/api/v1/products.py"
```

### 9.3 코드 리뷰 자동화

```powershell
# 전체 프로젝트 리뷰
claude-code -f app/**/*.py "
Review this codebase for:
1. Security vulnerabilities
2. Performance issues
3. Missing error handling
4. Inconsistent patterns
5. Missing type hints
6. Documentation gaps

Provide a prioritized list with file:line references.
"
```

---

## 10. 요약 및 다음 단계

### 핵심 포인트

✅ **자연어 중심 개발**: 코드를 직접 작성하는 대신 의도를 명확히 전달  
✅ **반복 작업 자동화**: 패턴화된 작업은 Claude에게 위임  
✅ **점진적 개선**: 작은 단위로 구축하고 테스트  
✅ **컨텍스트 관리**: 관련 파일을 함께 제공하여 정확도 향상  
✅ **검증 필수**: AI 생성 코드는 반드시 리뷰 및 테스트

### 다음 단계

1. 프로젝트 템플릿 저장소 생성
2. CI/CD 파이프라인 구축
3. 모니터링 대시보드 설정
4. 팀과 베스트 프랙티스 공유

---

**문서 버전**: 1.0  
**최종 업데이트**: 2026-01-02  
**작성자**: AI 개발 가이드 팀

**함께 읽을 문서**: [7계층 아키텍처 상세 분석](https://k82022603.github.io/posts/%ED%94%84%EB%A1%9C%EB%8D%95%EC%85%98%EA%B8%89-ai-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9D%98-7%EA%B3%84%EC%B8%B5-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EB%B6%84%EC%84%9D/)
