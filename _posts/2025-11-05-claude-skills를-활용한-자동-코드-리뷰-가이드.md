---
title: "Claude Skills를 활용한 자동 코드 리뷰 가이드"
date: 2025-11-05 13:00:00 +0900
categories: [AI,  Agent Skills]
mermaid: [True]
tags: [AI,  Guide,   claude-code,  agent-skills,  Claude.write]
---



## 📋 목차

1. [Claude Skills 개요](#claude-skills-개요)
2. [코드 리뷰용 Skills 생성하기](#코드-리뷰용-skills-생성하기)
3. [실전 Skills 예시](#실전-skills-예시)
4. [Skills 기반 워크플로우](#skills-기반-워크플로우)
5. [고급 활용 전략](#고급-활용-전략)
6. [베스트 프랙티스](#베스트-프랙티스)

---

## Claude Skills 개요

### Skills란?

Claude Skills는 Claude에게 특정 작업을 수행하는 방법을 가르치는 재사용 가능한 지침 세트입니다. 코드 리뷰에서 Skills를 사용하면:

- ✅ **일관성**: 모든 코드 리뷰에 동일한 기준 적용
- ✅ **전문성**: 특정 영역(보안, 성능 등)에 특화된 리뷰
- ✅ **효율성**: 반복적인 지침을 매번 입력할 필요 없음
- ✅ **컨텍스트**: 프로젝트별 규칙과 패턴을 기억
- ✅ **확장성**: 팀 전체가 공유하고 개선 가능

### Skills의 구조

Skills는 일반적으로 다음 요소로 구성됩니다:

```markdown
# Skill 이름

## 목적
이 Skill이 무엇을 하는지 명확히 설명

## 컨텍스트
프로젝트/기술 스택/규칙 등의 배경 정보

## 지침
구체적인 작업 수행 방법

## 예시
좋은 예시와 나쁜 예시

## 체크리스트
확인해야 할 항목들
```

---

## 코드 리뷰용 Skills 생성하기

### 1. 기본 코드 리뷰 Skill 생성

#### 단계 1: Claude에게 Skill 생성 요청

```
Claude, 우리 프로젝트를 위한 코드 리뷰 Skill을 만들어줘.

프로젝트 정보:
- 언어: Python 3.11
- 프레임워크: FastAPI
- 데이터베이스: PostgreSQL
- 주요 규칙: PEP 8, 타입 힌트 필수

중점 사항:
1. 보안 취약점
2. 성능 이슈
3. 코드 가독성
4. 테스트 커버리지
```

#### 단계 2: 생성된 Skill 저장

Claude가 생성한 Skill을 프로젝트에 저장:

**위치**: `/mnt/skills/user/code-review-python/SKILL.md`

~~~markdown
# Python FastAPI 코드 리뷰 Skill

## 목적
FastAPI 기반 Python 프로젝트의 코드를 체계적으로 리뷰하여 보안, 성능, 가독성을 보장합니다.

## 프로젝트 컨텍스트
- **언어**: Python 3.11+
- **프레임워크**: FastAPI
- **데이터베이스**: PostgreSQL + SQLAlchemy
- **스타일 가이드**: PEP 8
- **필수 요구사항**: 타입 힌트, Docstring

## 리뷰 프로세스

### 1단계: 보안 검토 (Critical)

#### SQL 인젝션 체크
```python
# ❌ 나쁜 예
query = f"SELECT * FROM users WHERE id = {user_id}"

# ✅ 좋은 예
query = select(User).where(User.id == user_id)
```

#### 인증/인가 체크
```python
# ❌ 나쁜 예
async def get_users():
    return db.query(User).all()

# ✅ 좋은 예
async def get_users(current_user: User = Depends(get_admin_user)):
    return db.query(User).all()
```

#### 민감 정보 노출 체크
```python
# ❌ 나쁜 예
class UserResponse(BaseModel):
    id: int
    email: str
    password_hash: str  # 절대 노출하면 안됨!

# ✅ 좋은 예
class UserResponse(BaseModel):
    id: int
    email: str
    # password_hash는 제외
```

### 2단계: 성능 검토 (High)

#### N+1 쿼리 문제
```python
# ❌ 나쁜 예
users = db.query(User).all()
for user in users:
    user.posts = db.query(Post).filter(Post.user_id == user.id).all()

# ✅ 좋은 예
users = db.query(User).options(joinedload(User.posts)).all()
```

#### 불필요한 데이터 로딩
```python
# ❌ 나쁜 예
user = db.query(User).first()  # 모든 필드 로딩

# ✅ 좋은 예 (일부만 필요한 경우)
user_email = db.query(User.email).filter(User.id == user_id).scalar()
```

#### 비동기 처리
```python
# ❌ 나쁜 예 (동기 I/O)
def send_email(to: str, subject: str):
    smtp.send(to, subject)  # 블로킹!

# ✅ 좋은 예
async def send_email(to: str, subject: str):
    await async_smtp.send(to, subject)
```

### 3단계: 코드 품질 검토 (Medium)

#### 타입 힌트
```python
# ❌ 나쁜 예
def calculate_total(items):
    return sum(item.price for item in items)

# ✅ 좋은 예
def calculate_total(items: List[Item]) -> Decimal:
    return sum(item.price for item in items)
```

#### Docstring
```python
# ❌ 나쁜 예
def process_payment(user_id, amount):
    pass

# ✅ 좋은 예
def process_payment(user_id: int, amount: Decimal) -> PaymentResult:
    """
    사용자 결제를 처리합니다.
    
    Args:
        user_id: 사용자 ID
        amount: 결제 금액 (양수여야 함)
        
    Returns:
        PaymentResult: 결제 처리 결과
        
    Raises:
        InsufficientFundsError: 잔액 부족 시
        InvalidAmountError: 금액이 0 이하일 때
    """
    pass
```

#### 에러 핸들링
```python
# ❌ 나쁜 예
try:
    user = db.query(User).filter(User.id == user_id).one()
except:  # bare except!
    return None

# ✅ 좋은 예
try:
    user = db.query(User).filter(User.id == user_id).one()
except NoResultFound:
    raise HTTPException(status_code=404, detail="User not found")
except MultipleResultsFound:
    logger.error(f"Multiple users found for id {user_id}")
    raise HTTPException(status_code=500, detail="Data integrity error")
```

### 4단계: 테스트 검토 (High)

```python
# ✅ 필수 테스트 패턴
import pytest
from fastapi.testclient import TestClient

def test_create_user_success():
    """정상 케이스"""
    response = client.post("/users", json=valid_user_data)
    assert response.status_code == 201

def test_create_user_duplicate_email():
    """중복 이메일 케이스"""
    response = client.post("/users", json=duplicate_email_data)
    assert response.status_code == 409

def test_create_user_invalid_email():
    """잘못된 이메일 형식"""
    response = client.post("/users", json=invalid_email_data)
    assert response.status_code == 422

def test_create_user_missing_field(field):
    """필수 필드 누락"""
    data = valid_user_data.copy()
    del data[field]
    response = client.post("/users", json=data)
    assert response.status_code == 422
```

## 리뷰 체크리스트

### 🔴 Critical (반드시 수정)
- [ ] SQL 인젝션 취약점 없음
- [ ] 인증/인가 검증 존재
- [ ] 민감 정보 노출 없음
- [ ] XSS 취약점 없음
- [ ] CSRF 보호 적용

### 🟡 High (수정 권장)
- [ ] N+1 쿼리 없음
- [ ] 적절한 인덱스 사용
- [ ] 비동기 처리 적용
- [ ] 에러 핸들링 적절
- [ ] 테스트 커버리지 80% 이상

### 🟢 Medium (개선 권장)
- [ ] 타입 힌트 존재
- [ ] Docstring 작성
- [ ] PEP 8 준수
- [ ] 변수명 명확
- [ ] 함수 길이 적절 (< 50줄)

### ⚪ Low (선택사항)
- [ ] 주석 적절
- [ ] 로깅 적절
- [ ] 성능 최적화 여지

## 리뷰 출력 형식

```markdown
## 🔍 코드 리뷰 결과

### 📊 요약
- Critical: X개
- High: X개
- Medium: X개
- Low: X개

### 🔴 Critical Issues

#### 1. SQL 인젝션 취약점
**위치**: `src/api/users.py:45`
**문제**: 사용자 입력을 직접 쿼리에 포함
**영향**: 데이터베이스 전체가 노출될 수 있음
**해결방안**:
```python
# 현재 코드
query = f"SELECT * FROM users WHERE name = '{name}'"

# 수정된 코드
query = select(User).where(User.name == name)
```

[추가 이슈들...]

### ✅ 긍정적인 부분
- 테스트 커버리지 90%로 우수
- 타입 힌트가 일관되게 사용됨
- 비동기 처리가 적절히 적용됨

### 💡 추가 제안
- 캐싱 전략 고려 (Redis)
- API 응답 압축 적용
- 로깅 레벨 조정
```

## 사용 방법

1. 코드 파일을 Claude에게 제공
2. "이 코드를 Python FastAPI 코드 리뷰 Skill로 리뷰해줘" 요청
3. 생성된 리뷰 결과 검토
4. Critical/High 이슈 즉시 수정

## 지속적 개선

이 Skill은 다음 경우에 업데이트됩니다:
- 새로운 보안 취약점 패턴 발견
- 프로젝트 규칙 변경
- False Positive 패턴 식별
- 팀 피드백 반영
~~~

### 2. 전문화된 Skill 생성

프로젝트 필요에 따라 여러 개의 전문 Skill을 만들 수 있습니다:

#### 2.1 보안 전문 Skill

**위치**: `/mnt/skills/user/security-review/SKILL.md`

~~~markdown
# 보안 중심 코드 리뷰 Skill

## 목적
OWASP Top 10 및 일반적인 보안 취약점을 집중적으로 검토합니다.

## 검토 항목

### A01: Broken Access Control
- [ ] 모든 API 엔드포인트에 인증 확인
- [ ] 권한 레벨 검증 (admin, user, guest)
- [ ] IDOR (Insecure Direct Object Reference) 체크

**체크 방법**:
```python
# 각 엔드포인트에서 확인
async def get_user(
    user_id: int,
    current_user: User = Depends(get_current_user)  # ✅ 인증 체크
):
    # ✅ 권한 체크
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(403, "Access denied")
    
    return db.query(User).get(user_id)
```

### A02: Cryptographic Failures
- [ ] 패스워드 해싱 알고리즘 (bcrypt/argon2)
- [ ] API 키/시크릿 하드코딩 여부
- [ ] HTTPS 강제 사용
- [ ] 민감 데이터 로깅 방지

**체크 방법**:
```python
# ❌ 나쁜 예
password_hash = hashlib.md5(password.encode()).hexdigest()
api_key = "sk_live_abc123"  # 하드코딩!

# ✅ 좋은 예
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
password_hash = pwd_context.hash(password)

api_key = os.environ.get("API_KEY")  # 환경변수 사용
```

### A03: Injection
- [ ] SQL 인젝션 (ORM 사용 확인)
- [ ] NoSQL 인젝션
- [ ] Command 인젝션
- [ ] LDAP 인젝션

**SQL 인젝션 체크**:
```python
# ❌ 취약한 코드
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
cursor.execute("SELECT * FROM users WHERE name = '" + name + "'")

# ✅ 안전한 코드
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
# 또는 ORM 사용
db.query(User).filter(User.id == user_id).first()
```

**Command 인젝션 체크**:
```python
# ❌ 위험한 코드
os.system(f"ping {user_input}")
subprocess.call(f"ls {directory}", shell=True)

# ✅ 안전한 코드
import shlex
subprocess.run(["ping", "-c", "1", user_input])
subprocess.run(shlex.split(f"ls {shlex.quote(directory)}"))
```

### A04: Insecure Design
- [ ] 비즈니스 로직 검증
- [ ] Rate Limiting 구현
- [ ] 트랜잭션 무결성

```python
# ✅ Rate Limiting 예시
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

async def login(credentials: LoginCredentials):
    pass
```

### A05: Security Misconfiguration
- [ ] 디버그 모드 비활성화 (프로덕션)
- [ ] 상세한 에러 메시지 노출 방지
- [ ] 불필요한 기능/엔드포인트 제거
- [ ] CORS 설정 검토

```python
# ✅ 환경별 설정
if os.getenv("ENVIRONMENT") == "production":
    DEBUG = False
    ALLOWED_HOSTS = ["example.com"]
else:
    DEBUG = True
    ALLOWED_HOSTS = ["*"]

# ✅ 에러 처리
async def generic_exception_handler(request, exc):
    if DEBUG:
        return JSONResponse(
            status_code=500,
            content={"detail": str(exc), "traceback": traceback.format_exc()}
        )
    else:
        return JSONResponse(
            status_code=500,
            content={"detail": "Internal server error"}  # 최소한의 정보만
        )
```

### A06: Vulnerable Components
- [ ] 의존성 버전 체크 (outdated)
- [ ] 알려진 취약점 존재 여부

```bash
# 자동 체크 도구
pip install safety
safety check

# requirements.txt 예시
fastapi>=0.104.0  # 최신 버전 사용
pydantic>=2.4.0
sqlalchemy>=2.0.0
```

### A07: Authentication Failures
- [ ] 약한 패스워드 정책
- [ ] 세션 관리 취약점
- [ ] 브루트포스 공격 방어

```python
# ✅ 패스워드 정책
import re

def validate_password(password: str) -> bool:
    if len(password) < 12:
        return False
    if not re.search(r"[A-Z]", password):  # 대문자
        return False
    if not re.search(r"[a-z]", password):  # 소문자
        return False
    if not re.search(r"\d", password):  # 숫자
        return False
    if not re.search(r"[!@#$%^&*]", password):  # 특수문자
        return False
    return True

# ✅ 로그인 실패 횟수 제한
from datetime import datetime, timedelta

login_attempts = {}  # Redis 사용 권장

async def login(credentials: LoginCredentials):
    ip = request.client.host
    
    if ip in login_attempts:
        attempts, last_attempt = login_attempts[ip]
        if attempts >= 5 and datetime.now() - last_attempt < timedelta(minutes=15):
            raise HTTPException(429, "Too many login attempts. Try again later.")
    
    # 로그인 로직...
```

### A08: Software and Data Integrity Failures
- [ ] 코드/설정 무결성 검증
- [ ] CI/CD 파이프라인 보안
- [ ] 자동 업데이트 검증

### A09: Security Logging Failures
- [ ] 보안 이벤트 로깅
- [ ] 로그에 민감정보 포함 여부
- [ ] 로그 모니터링/알림

```python
# ✅ 보안 로깅
import logging

security_logger = logging.getLogger("security")

async def login(credentials: LoginCredentials, request: Request):
    try:
        user = authenticate(credentials)
        security_logger.info(
            f"Successful login: user={user.id}, ip={request.client.host}"
        )
        return {"token": create_token(user)}
    except AuthenticationError:
        security_logger.warning(
            f"Failed login attempt: email={credentials.email}, "
            f"ip={request.client.host}"
        )
        raise HTTPException(401, "Invalid credentials")

# ❌ 로그에 민감정보 포함하지 않기
logger.info(f"User login: {user.email}, password: {password}")  # 절대 안됨!

# ✅ 안전한 로깅
logger.info(f"User login: {user.email}, password: [REDACTED]")
```

### A10: Server-Side Request Forgery (SSRF)
- [ ] 외부 URL 접근 검증
- [ ] 내부 IP 접근 차단

```python
# ✅ URL 검증
from urllib.parse import urlparse
import ipaddress

def is_safe_url(url: str) -> bool:
    parsed = urlparse(url)
    
    # 허용된 스키마만
    if parsed.scheme not in ['http', 'https']:
        return False
    
    # 내부 IP 차단
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        if ip.is_private or ip.is_loopback:
            return False
    except ValueError:
        pass  # 도메인인 경우
    
    return True

async def fetch_url(url: str):
    if not is_safe_url(url):
        raise HTTPException(400, "Invalid URL")
    
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.json()
```

## 출력 형식

```markdown
## 🔒 보안 리뷰 결과

### 위험도 분포
- 🔴 Critical: X개 (즉시 수정 필요)
- 🟠 High: X개 (우선 수정)
- 🟡 Medium: X개 (수정 권장)
- 🟢 Low: X개 (개선 사항)

### OWASP Top 10 체크

#### ✅ 통과
- A04: Insecure Design - Rate limiting 구현됨
- A05: Security Misconfiguration - 환경별 설정 적절

#### ❌ 발견된 이슈

**A01: Broken Access Control**
- 위치: `api/users.py:45`
- 권한 체크 누락
- 영향: 다른 사용자의 데이터 접근 가능

**A03: Injection**
- 위치: `db/queries.py:78`
- SQL 인젝션 취약점
- 영향: 데이터베이스 전체 노출 가능

### 상세 분석
[...]
```

## 사용 방법

```
Claude, 이 코드를 보안 리뷰 Skill로 철저히 검토해줘.
특히 OWASP Top 10을 중심으로 봐줘.
```
~~~

#### 2.2 성능 전문 Skill

**위치**: `/mnt/skills/user/performance-review/SKILL.md`

~~~markdown
# 성능 최적화 코드 리뷰 Skill

## 목적
코드의 성능 병목을 식별하고 최적화 방안을 제시합니다.

## 검토 영역

### 1. 데이터베이스 최적화

#### N+1 쿼리 문제
```python
# ❌ N+1 문제
users = User.query.all()  # 1번 쿼리
for user in users:
    posts = Post.query.filter_by(user_id=user.id).all()  # N번 쿼리

# ✅ 해결방법 1: JOIN
users = User.query.options(joinedload(User.posts)).all()  # 1번 쿼리

# ✅ 해결방법 2: Eager Loading
users = User.query.options(selectinload(User.posts)).all()

# ✅ 해결방법 3: 수동 최적화
users = User.query.all()
user_ids = [u.id for u in users]
posts = Post.query.filter(Post.user_id.in_(user_ids)).all()
posts_by_user = {}
for post in posts:
    posts_by_user.setdefault(post.user_id, []).append(post)
```

#### 인덱스 활용
```python
# ❌ 인덱스 없는 검색
# CREATE TABLE users (id, email, name, created_at)
users = User.query.filter(User.email == email).all()  # 전체 테이블 스캔!

# ✅ 인덱스 추가
# CREATE INDEX idx_users_email ON users(email);

# ✅ 복합 인덱스
# CREATE INDEX idx_users_status_created ON users(status, created_at);
users = User.query.filter(
    User.status == 'active',
    User.created_at > start_date
).all()  # 인덱스 활용
```

#### 페이지네이션
```python
# ❌ 전체 데이터 로딩
all_users = User.query.all()  # 메모리 폭발!

# ✅ 페이지네이션
page = 1
per_page = 50
users = User.query.paginate(page=page, per_page=per_page)

# ✅ Cursor 기반 페이지네이션 (대용량)
last_id = request.args.get('last_id', 0)
users = User.query.filter(User.id > last_id).limit(50).all()
```

### 2. API 최적화

#### 불필요한 데이터 전송
```python
# ❌ 전체 객체 반환
async def list_users():
    users = db.query(User).all()
    return users  # 모든 필드 반환!

# ✅ 필요한 필드만
async def list_users():
    users = db.query(User.id, User.name, User.email).all()
    return [{"id": u.id, "name": u.name, "email": u.email} for u in users]

# ✅ Pydantic 모델로 제어
class UserListResponse(BaseModel):
    id: int
    name: str
    email: str

async def list_users():
    return db.query(User).all()
```

#### 응답 압축
```python
# ✅ Gzip 압축
from fastapi.middleware.gzip import GZipMiddleware

app.add_middleware(GZipMiddleware, minimum_size=1000)
```

#### HTTP 캐싱
```python
# ✅ Cache-Control 헤더
from fastapi import Response

async def get_product(product_id: int, response: Response):
    product = db.query(Product).get(product_id)
    
    # 공개 캐시, 1시간 유효
    response.headers["Cache-Control"] = "public, max-age=3600"
    response.headers["ETag"] = f'"{product.updated_at.timestamp()}"'
    
    return product
```

### 3. 메모리 최적화

#### 대용량 파일 처리
```python
# ❌ 전체 파일을 메모리에
with open('large_file.csv', 'r') as f:
    data = f.read()  # 10GB 파일이면 메모리 폭발!
    process(data)

# ✅ 스트리밍 처리
with open('large_file.csv', 'r') as f:
    for line in f:  # 한 줄씩 처리
        process(line)

# ✅ 청크 단위 처리
def read_in_chunks(file_path, chunk_size=1024*1024):  # 1MB
    with open(file_path, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

for chunk in read_in_chunks('large_file.bin'):
    process(chunk)
```

#### 제너레이터 활용
```python
# ❌ 리스트 생성 (메모리 많이 사용)
def get_all_users():
    users = db.query(User).all()
    return [transform(u) for u in users]  # 전체를 메모리에!

# ✅ 제너레이터 (메모리 효율적)
def get_all_users():
    users = db.query(User).yield_per(100)  # 100개씩
    for user in users:
        yield transform(user)

# 사용
for user in get_all_users():
    send_email(user)
```

### 4. 비동기 처리

#### I/O 바운드 작업
```python
# ❌ 동기 처리 (블로킹)
def fetch_user_data(user_ids):
    results = []
    for user_id in user_ids:
        response = requests.get(f"/api/users/{user_id}")  # 각각 대기!
        results.append(response.json())
    return results

# ✅ 비동기 처리 (동시 실행)
async def fetch_user_data(user_ids):
    async with httpx.AsyncClient() as client:
        tasks = [
            client.get(f"/api/users/{user_id}") 
            for user_id in user_ids
        ]
        responses = await asyncio.gather(*tasks)  # 동시에!
        return [r.json() for r in responses]
```

#### 백그라운드 작업
```python
# ❌ 동기 처리 (사용자 대기)
async def register(user: UserCreate):
    db_user = create_user(user)
    send_welcome_email(db_user)  # 이메일 보내는 동안 대기!
    return db_user

# ✅ 백그라운드 작업
from fastapi import BackgroundTasks

async def register(user: UserCreate, background_tasks: BackgroundTasks):
    db_user = create_user(user)
    background_tasks.add_task(send_welcome_email, db_user)  # 백그라운드에서
    return db_user  # 즉시 반환
```

### 5. 캐싱 전략

#### 메모리 캐싱
```python
# ✅ functools.lru_cache
from functools import lru_cache

def get_user_permissions(user_id: int):
    # 무거운 계산...
    return permissions

# ✅ Redis 캐싱
import redis
import json

redis_client = redis.Redis(host='localhost', port=6379)

async def get_product(product_id: int):
    # 캐시 확인
    cache_key = f"product:{product_id}"
    cached = redis_client.get(cache_key)
    
    if cached:
        return json.loads(cached)
    
    # DB 조회
    product = db.query(Product).get(product_id)
    
    # 캐시 저장 (1시간)
    redis_client.setex(cache_key, 3600, json.dumps(product.dict()))
    
    return product
```

#### HTTP 캐싱
```python
# ✅ ETags
from hashlib import md5

async def get_data(request: Request, response: Response):
    data = get_expensive_data()
    
    # ETag 생성
    content = json.dumps(data)
    etag = md5(content.encode()).hexdigest()
    
    # 클라이언트 ETag 확인
    if request.headers.get('If-None-Match') == etag:
        return Response(status_code=304)  # Not Modified
    
    response.headers['ETag'] = etag
    return data
```

### 6. 알고리즘 최적화

#### 시간 복잡도
```python
# ❌ O(n²)
def find_duplicates(items):
    duplicates = []
    for i, item1 in enumerate(items):
        for j, item2 in enumerate(items[i+1:]):
            if item1 == item2:
                duplicates.append(item1)
    return duplicates

# ✅ O(n)
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return list(duplicates)
```

#### 적절한 자료구조
```python
# ❌ 리스트로 검색 (O(n))
user_ids = [1, 2, 3, 4, 5, ...]  # 수천 개
if user_id in user_ids:  # 느림!
    pass

# ✅ 집합으로 검색 (O(1))
user_ids = {1, 2, 3, 4, 5, ...}
if user_id in user_ids:  # 빠름!
    pass
```

## 성능 측정

### 프로파일링
```python
# ✅ cProfile 사용
import cProfile
import pstats

def profile_function():
    profiler = cProfile.Profile()
    profiler.enable()
    
    # 측정할 코드
    expensive_function()
    
    profiler.disable()
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(10)  # 상위 10개
```

### 타이밍
```python
# ✅ 실행 시간 측정
import time
from functools import wraps

def timeit(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.time()
        result = await func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.2f}s")
        return result
    return wrapper

async def slow_function():
    await asyncio.sleep(2)
```

## 출력 형식

```markdown
## ⚡ 성능 리뷰 결과

### 발견된 병목

#### 🔴 Critical: N+1 쿼리 문제
- **위치**: `api/posts.py:34-38`
- **영향**: 1000개 포스트 조회 시 1001번의 쿼리 실행
- **예상 개선**: 10초 → 0.1초 (100배)
- **해결방안**: 
```python
# 현재
posts = Post.query.all()
for post in posts:
    post.author = User.query.get(post.author_id)

# 개선
posts = Post.query.options(joinedload(Post.author)).all()
```

#### 🟡 High: 전체 데이터 로딩
- **위치**: `api/users.py:56`
- **영향**: 메모리 사용량 1GB
- **예상 개선**: 메모리 1GB → 10MB
- **해결방안**: 페이지네이션 적용

### 최적화 제안

1. **Redis 캐싱 도입**
   - 자주 조회되는 데이터 (제품 정보, 사용자 권한)
   - 예상 효과: 응답 시간 50% 감소

2. **DB 인덱스 추가**
   - `users.email` 컬럼
   - `orders.created_at` 컬럼
   - 예상 효과: 쿼리 속도 10배 향상

3. **비동기 처리 확대**
   - 외부 API 호출
   - 이메일 발송
   - 예상 효과: 사용자 응답 시간 70% 감소
```
~~~

---

## 실전 Skills 예시

### 예시 1: Django 프로젝트용 Skill

~~~markdown
# Django REST Framework 코드 리뷰 Skill

## 프로젝트 컨텍스트
- Framework: Django 4.2 + DRF 3.14
- Database: PostgreSQL
- Celery for background tasks
- Redis for caching

## 특별 검토 사항

### 1. Serializer 최적화
```python
# ❌ N+1 발생
class PostSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.name')
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'author_name']

# View에서
posts = Post.objects.all()  # N+1 발생!
serializer = PostSerializer(posts, many=True)

# ✅ select_related 사용
posts = Post.objects.select_related('author').all()
serializer = PostSerializer(posts, many=True)
```

### 2. ViewSet 패턴
```python
# ✅ 표준 ViewSet 구조
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.select_related('author').prefetch_related('tags')
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ['title', 'content']
    ordering_fields = ['created_at', 'updated_at']
    
    def get_queryset(self):
        queryset = super().get_queryset()
        if not self.request.user.is_staff:
            queryset = queryset.filter(status='published')
        return queryset
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.status = 'published'
        post.save()
        return Response({'status': 'published'})
```

### 3. 권한 체크
```python
# ✅ 커스텀 Permission
from rest_framework import permissions

class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        # Read permissions (GET, HEAD, OPTIONS)
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write permissions (POST, PUT, PATCH, DELETE)
        return obj.author == request.user

# 사용
class PostViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthorOrReadOnly]
```

### 4. Celery 작업
```python
# ✅ Celery Task 패턴
from celery import shared_task
from django.core.mail import send_mail

def send_notification_email(self, user_id, message):
    try:
        user = User.objects.get(id=user_id)
        send_mail(
            'Notification',
            message,
            'noreply@example.com',
            [user.email],
        )
    except Exception as exc:
        # Retry with exponential backoff
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# View에서 비동기 호출
send_notification_email.delay(user.id, "Welcome!")
```
~~~

### 예시 2: React/TypeScript 프론트엔드 Skill

~~~markdown
# React TypeScript 코드 리뷰 Skill

## 프로젝트 컨텍스트
- React 18 + TypeScript 5
- State Management: Zustand
- Styling: Tailwind CSS
- API: React Query

## 검토 사항

### 1. 타입 안정성
```typescript
// ❌ any 사용
function fetchUser(id: any): any {
    return fetch(`/api/users/${id}`).then(r => r.json())
}

// ✅ 구체적 타입
interface User {
    id: number
    name: string
    email: string
}

async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`)
    return response.json()
}
```

### 2. React Hook 사용
```typescript
// ❌ 잘못된 의존성 배열
useEffect(() => {
    fetchData(userId)
}, [])  // userId 변경 시 실행 안됨!

// ✅ 올바른 의존성
useEffect(() => {
    fetchData(userId)
}, [userId])

// ✅ useCallback으로 최적화
const handleSubmit = useCallback((data: FormData) => {
    submitForm(data)
}, [submitForm])
```

### 3. 성능 최적화
```typescript
// ❌ 매 렌더링마다 재생성
function UserList({ users }: Props) {
    return (
        <div>
            {users.map(user => (
                <UserCard 
                    key={user.id}
                    user={user}
                    onClick={() => handleClick(user.id)}  // 매번 새 함수!
                />
            ))}
        </div>
    )
}

// ✅ useCallback 사용
function UserList({ users }: Props) {
    const handleClick = useCallback((userId: number) => {
        // handle click
    }, [])
    
    return (
        <div>
            {users.map(user => (
                <UserCard 
                    key={user.id}
                    user={user}
                    onClick={handleClick}
                />
            ))}
        </div>
    )
}

// ✅ React.memo로 불필요한 리렌더링 방지
const UserCard = React.memo(({ user, onClick }: Props) => {
    return <div onClick={() => onClick(user.id)}>{user.name}</div>
})
```

### 4. React Query 패턴
```typescript
// ✅ 표준 패턴
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

function useUser(userId: number) {
    return useQuery({
        queryKey: ['user', userId],
        queryFn: () => fetchUser(userId),
        staleTime: 5 * 60 * 1000,  // 5분
        cacheTime: 10 * 60 * 1000,  // 10분
    })
}

function useUpdateUser() {
    const queryClient = useQueryClient()
    
    return useMutation({
        mutationFn: (user: User) => updateUser(user),
        onSuccess: (data, variables) => {
            // 캐시 무효화
            queryClient.invalidateQueries({ queryKey: ['user', variables.id] })
            // 또는 낙관적 업데이트
            queryClient.setQueryData(['user', variables.id], data)
        },
    })
}
```
~~~

### 예시 3: Node.js/Express 백엔드 Skill

~~~markdown
# Node.js Express 코드 리뷰 Skill

## 프로젝트 컨텍스트
- Node.js 20 + Express 4
- TypeScript
- MongoDB + Mongoose
- JWT 인증

## 검토 사항

### 1. 에러 핸들링
```typescript
// ❌ 에러 처리 없음
app.get('/users/:id', (req, res) => {
    const user = await User.findById(req.params.id)  // await 빠짐!
    res.json(user)
})

// ✅ 적절한 에러 처리
app.get('/users/:id', async (req, res, next) => {
    try {
        const user = await User.findById(req.params.id)
        if (!user) {
            return res.status(404).json({ error: 'User not found' })
        }
        res.json(user)
    } catch (error) {
        next(error)  // 에러 미들웨어로 전달
    }
})

// ✅ 글로벌 에러 핸들러
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    console.error(err.stack)
    
    if (err instanceof ValidationError) {
        return res.status(400).json({ error: err.message })
    }
    
    if (err instanceof UnauthorizedError) {
        return res.status(401).json({ error: 'Unauthorized' })
    }
    
    res.status(500).json({ error: 'Internal server error' })
})
```

### 2. 미들웨어 패턴
```typescript
// ✅ 인증 미들웨어
import jwt from 'jsonwebtoken'

interface AuthRequest extends Request {
    user?: JWTPayload
}

const authenticate = async (
    req: AuthRequest, 
    res: Response, 
    next: NextFunction
) => {
    try {
        const token = req.headers.authorization?.split(' ')[1]
        if (!token) {
            return res.status(401).json({ error: 'No token provided' })
        }
        
        const decoded = jwt.verify(token, process.env.JWT_SECRET!)
        req.user = decoded as JWTPayload
        next()
    } catch (error) {
        res.status(401).json({ error: 'Invalid token' })
    }
}

// ✅ 권한 체크 미들웨어
const authorize = (...roles: string[]) => {
    return (req: AuthRequest, res: Response, next: NextFunction) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Not authenticated' })
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' })
        }
        
        next()
    }
}

// 사용
app.delete('/users/:id', authenticate, authorize('admin'), deleteUser)
```

### 3. 입력 검증
```typescript
// ✅ Joi 스키마 검증
import Joi from 'joi'

const userSchema = Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required(),
    name: Joi.string().min(2).max(50).required(),
})

const validateRequest = (schema: Joi.Schema) => {
    return (req: Request, res: Response, next: NextFunction) => {
        const { error } = schema.validate(req.body)
        if (error) {
            return res.status(400).json({ 
                error: error.details[0].message 
            })
        }
        next()
    }
}

// 사용
app.post('/users', validateRequest(userSchema), createUser)
```

### 4. 데이터베이스 최적화
```typescript
// ❌ N+1 쿼리
const posts = await Post.find()
for (const post of posts) {
    post.author = await User.findById(post.authorId)  // N번!
}

// ✅ populate 사용
const posts = await Post.find().populate('author')

// ✅ lean() 사용 (읽기 전용)
const posts = await Post.find().populate('author').lean()  // 더 빠름

// ✅ 필요한 필드만 선택
const posts = await Post.find()
    .select('title content createdAt')
    .populate('author', 'name email')
```
~~~

---

## Skills 기반 워크플로우

### 개인 개발자 워크플로우

```bash
# 1. 코드 작성
vim src/api/users.py

# 2. Claude에게 업로드
# Claude.ai에서 파일 업로드

# 3. Skills 기반 리뷰 요청
"이 코드를 다음 Skills로 리뷰해줘:
1. Python FastAPI 코드 리뷰 Skill
2. 보안 리뷰 Skill
3. 성능 리뷰 Skill"

# 4. 리뷰 결과 확인 및 수정

# 5. 재검토
"수정한 코드를 다시 리뷰해줘"

# 6. 통과하면 커밋
git add src/api/users.py
git commit -m "Add user API endpoint"
```

### 팀 워크플로우

```bash
# 1. PR 생성
git push origin feature/user-api
# GitHub에서 PR 생성

# 2. Claude Code CLI로 자동 리뷰
cd /mnt/skills/user
cat > code-review.sh << 'EOF'
#!/bin/bash
# 모든 Skills를 순차적으로 실행

echo "=== Python FastAPI Review ==="
claude review --skill python-fastapi

echo "=== Security Review ==="
claude review --skill security

echo "=== Performance Review ==="
claude review --skill performance

echo "=== Test Coverage Check ==="
pytest --cov=src --cov-report=term-missing
EOF

chmod +x code-review.sh
./code-review.sh > review-result.md

# 3. 결과를 PR 코멘트로 추가
gh pr comment --body-file review-result.md

# 4. CI/CD에서 자동 실행
# .github/workflows/code-review.yml
name: Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Claude Code Review
        run: |
          ./code-review.sh
          gh pr comment --body-file review-result.md
```

### 오픈소스 프로젝트 워크플로우

~~~bash
# 1. 컨트리뷰션 가이드에 Skills 명시
# CONTRIBUTING.md
"""
## Code Review Process

All PRs are automatically reviewed using Claude Skills:

1. **Python Style Review**: PEP 8 compliance
2. **Security Review**: OWASP Top 10
3. **Performance Review**: Common bottlenecks
4. **Test Review**: Coverage and quality

Please run local review before submitting:
```bash
claude review --skill python-style
claude review --skill security
```
"""

# 2. Pre-commit hook 설정
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: claude-review
        name: Claude Code Review
        entry: claude review --skill python-style --skill security
        language: system
        pass_filenames: false
~~~

---

## 고급 활용 전략

### 1. Skills 조합

여러 Skills를 조합하여 포괄적인 리뷰 수행:

```
Claude, 이 PR을 다음 순서로 리뷰해줘:

1. 먼저 "보안 리뷰 Skill"로 취약점 체크
2. "성능 리뷰 Skill"로 병목 확인
3. "Python FastAPI Skill"로 코드 품질 검토
4. 각 Skill의 결과를 종합한 최종 리포트 작성

Critical 이슈가 있으면 즉시 알려주고,
모든 리뷰가 끝나면 우선순위별로 정리해줘.
```

### 2. 컨텍스트 유지

프로젝트별 컨텍스트를 Skills에 저장:

```markdown
# 프로젝트별 Skill: my-saas-project

## 프로젝트 정보
- 이름: MyAwesomeSaaS
- 도메인: 구독 기반 프로젝트 관리 툴
- 주요 기능: 프로젝트, 태스크, 팀 관리

## 아키텍처
- 프론트: React 18 + TypeScript
- 백엔드: Python FastAPI
- DB: PostgreSQL
- 캐시: Redis
- 큐: Celery + RabbitMQ

## 비즈니스 로직 규칙
1. 무료 플랜: 프로젝트 최대 3개
2. 프로 플랜: 프로젝트 무제한
3. 팀원 초대는 관리자만 가능
4. 데이터 수정은 30일 히스토리 유지

## 특별 검토 사항
- 플랜별 제한 로직 철저히 체크
- 권한 체크 필수 (프로젝트/조직 레벨)
- 데이터 변경 시 히스토리 기록 확인
- API Rate Limiting (플랜별 차등)

## 알려진 이슈
- User 모델의 created_at 인덱스 없음 (TODO)
- 대량 이메일 발송 시 throttling 미구현
```

이제 단순히 "이 코드를 리뷰해줘"가 아니라:

```
"이 코드를 my-saas-project Skill로 리뷰해줘"
```

Claude가 프로젝트 컨텍스트를 알고 있어 더 정확한 리뷰 가능!

### 3. 점진적 개선

Skills는 살아있는 문서입니다. 지속적으로 개선하세요:

```markdown
# Skill 개선 로그

## 2025-11-05: v1.1
- False Positive 패턴 추가:
  - `test_` 함수에서는 타입 힌트 선택사항으로 변경
  - Mock 객체 사용 시 attribute 체크 완화

## 2025-11-03: v1.0
- 초기 버전 생성
```

### 4. Skills 공유

팀 전체가 동일한 Skills 사용:

```bash
# Git 레포지토리로 관리
git clone git@github.com:mycompany/claude-skills.git /mnt/skills/team

# 또는 공유 스토리지
ln -s /shared/claude-skills /mnt/skills/team
```

---

## 베스트 프랙티스

### 1. Skills 작성 가이드

#### ✅ 좋은 Skill
~~~markdown
# 구체적인 예시 제공
# 긍정/부정 사례 모두 포함
# 실행 가능한 해결책 제시
# 컨텍스트 명확히

## SQL 인젝션 체크

❌ 나쁜 예:
```python
query = f"SELECT * FROM users WHERE id = {user_id}"
```

✅ 좋은 예:
```python
from sqlalchemy import select
query = select(User).where(User.id == user_id)
```

이유: f-string으로 쿼리 생성 시 악의적인 입력으로
     전체 DB가 노출될 수 있음
~~~

#### ❌ 나쁜 Skill
```markdown
# 추상적이고 모호함
# 예시 없음

## 보안에 주의하세요
코드를 작성할 때 보안을 고려해야 합니다.
SQL 인젝션, XSS 등을 방지하세요.
```

### 2. Skills 조직화

```
/mnt/skills/
├── public/              # 공개 Skills (Anthropic 제공)
│   ├── docx/
│   ├── pdf/
│   └── pptx/
│
├── user/                # 개인 Skills
│   ├── python-fastapi/
│   ├── security/
│   ├── performance/
│   └── my-project/
│
├── team/                # 팀 공유 Skills
│   ├── company-style/
│   ├── api-standards/
│   └── security-policy/
│
└── examples/            # 예시 Skills
    ├── django-review/
    └── react-review/
```

### 3. 효과적인 프롬프트

```
# ❌ 모호한 요청
"이 코드 리뷰해줘"

# ✅ 구체적인 요청
"이 코드를 다음 관점에서 리뷰해줘:
1. Python FastAPI Skill로 코드 품질 체크
2. 보안 Skill로 OWASP Top 10 체크
3. 특히 결제 로직이니까 트랜잭션 무결성 집중

Critical 이슈는 즉시 알려주고,
수정 코드도 함께 제공해줘."
```

### 4. 지속적 개선

```markdown
# Skill 피드백 로그

## 날짜: 2025-11-05
**False Positive**: 테스트 코드의 긴 함수명을 경고
**개선**: test_ 함수는 길이 제한 완화

## 날짜: 2025-11-04
**True Positive**: 실제 SQL 인젝션 발견! 🎉
**확인**: Skill이 정상 작동

## 날짜: 2025-11-03
**False Negative**: Race condition 놓침
**개선**: 동시성 체크 로직 추가
```

---

## 결론

Claude Skills를 활용하면:

1. ✅ **일관성**: 모든 코드에 동일한 기준 적용
2. ✅ **전문성**: 각 영역별 깊이 있는 리뷰
3. ✅ **효율성**: 반복 작업 자동화
4. ✅ **확장성**: 팀 전체가 공유하고 개선
5. ✅ **컨텍스트**: 프로젝트 특성을 반영한 리뷰

### 시작 단계

1. **Week 1**: 기본 코드 리뷰 Skill 생성
2. **Week 2**: 보안/성능 전문 Skills 추가
3. **Week 3**: 프로젝트별 Skill 커스터마이징
4. **Week 4**: 팀 워크플로우에 통합

### 다음 단계

- [ ] 첫 번째 Skill 생성
- [ ] 실제 코드로 테스트
- [ ] 팀원들과 공유
- [ ] 피드백 수집 및 개선
- [ ] CI/CD 파이프라인 통합

이 가이드가 여러분의 코드 품질 향상에 도움이 되기를 바랍니다! 🚀

---

**문서 버전**: 2.0  
**최종 업데이트**: 2025년 11월 5일