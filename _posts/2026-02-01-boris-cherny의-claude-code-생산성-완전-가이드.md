---
title: "Boris Cherny의 Claude Code 생산성 완전 가이드"
date: 2026-02-01 16:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  Material,  claude-code,  BorisCherny,  Claude.write]
---


> Claude Code 창시자 Boris Cherny와 개발팀이 직접 공유하는 실전 생산성 향상 전략  
> 작성일: 2026-02-01

## 관련글

[https://www.threads.com/@boris_cherny/post/DUMZr4VElyb](https://www.threads.com/@boris_cherny/post/DUMZr4VElyb)

## 목차

1. [핵심 요약](#핵심-요약)
2. [병렬성의 재발견: Git Worktree 마스터하기](#병렬성의-재발견-git-worktree-마스터하기)
3. [자기 학습하는 시스템: CLAUDE.md의 복리 효과](#자기-학습하는-시스템-claudemd의-복리-효과)
4. [프롬프팅의 레벨업: 지시에서 협업으로](#프롬프팅의-레벨업-지시에서-협업으로)
5. [터미널 환경 최적화](#터미널-환경-최적화)
6. [서브에이전트 활용 전략](#서브에이전트-활용-전략)
7. [데이터 분석 워크플로우](#데이터-분석-워크플로우)
8. [학습 도구로서의 Claude Code](#학습-도구로서의-claude-code)
9. [버그 수정 자동화](#버그-수정-자동화)
10. [실전 팁 총정리](#실전-팁-총정리)

---

## 핵심 요약

**3줄 요약:**
- **병렬화**: git worktree를 활용하여 3~5개 세션을 동시에 운영하라
- **체계화**: 플랜 모드를 사용하고 CLAUDE.md로 실수를 제도화하여 팀의 집단 지성을 구축하라
- **고도화**: 고급 프롬프팅과 서브에이전트를 활용해 Claude를 협력자로 진화시켜라

---

## 병렬성의 재발견: Git Worktree 마스터하기

### 왜 병렬화인가?

대부분의 개발자는 여전히 단일 세션 패러다임에 갇혀 있습니다. 터미널 하나, Claude 세션 하나, 작업 하나. 이 방식은 직관적이지만 AI 에이전트의 특성을 제대로 활용하지 못합니다.

Claude Code 팀의 1번 생산성 팁은 **"병렬화"**입니다. 그들은 평균 3~5개의 git worktree를 동시에 운영하며, 각각에서 독립적인 Claude 세션을 실행합니다.

### Git Worktree 기초

#### Worktree란?

Git worktree는 하나의 저장소에서 여러 개의 독립적인 작업 디렉토리를 만드는 기능입니다. 각 worktree는 서로 다른 브랜치를 체크아웃할 수 있으며, 동시에 작업할 수 있습니다.

#### 기본 설정 예시

```bash
# 메인 저장소 구조
~/projects/my-app/          # 메인 worktree (main 브랜치)
~/projects/my-app-wt-a/     # 기능 A 개발용
~/projects/my-app-wt-b/     # 기능 B 개발용
~/projects/my-app-wt-analysis/  # 분석 전용
```

#### Worktree 생성 명령어

```bash
# 현재 디렉토리에서 worktree 추가
git worktree add ../my-app-wt-a feature/user-auth

# 새 브랜치를 만들면서 worktree 생성
git worktree add -b feature/payment ../my-app-wt-b

# 분석 전용 worktree (main 브랜치)
git worktree add ../my-app-wt-analysis main

# worktree 목록 확인
git worktree list
# 출력:
# /Users/boris/projects/my-app           abc1234 [main]
# /Users/boris/projects/my-app-wt-a      def5678 [feature/user-auth]
# /Users/boris/projects/my-app-wt-b      ghi9012 [feature/payment]
```

### 실전 Worktree 워크플로우

#### 시나리오 1: 다중 기능 개발

```bash
# Worktree A: 사용자 인증 기능
cd ~/projects/my-app-wt-a
claude

# 프롬프트:
# "사용자 인증 시스템을 구현해줘. OAuth 2.0과 JWT를 사용하고,
# 먼저 플랜 모드로 전체 아키텍처를 설계해줘"

# Claude가 작업하는 동안...

# Worktree B: 결제 시스템 (다른 터미널)
cd ~/projects/my-app-wt-b
claude

# 프롬프트:
# "Stripe 결제 통합을 구현해줘. 구독과 일회성 결제를 모두 지원해야 해.
# 플랜 모드로 시작해서 단계별 계획을 세워줘"

# Claude가 작업하는 동안...

# Worktree Analysis: 로그 분석 (세 번째 터미널)
cd ~/projects/my-app-wt-analysis
claude

# 프롬프트:
# "지난 7일간의 에러 로그를 분석해서 패턴을 찾아줘.
# BigQuery에서 데이터를 가져와서 시각화해줘"
```

#### 시나리오 2: 핫픽스 + 기능 개발 병행

```bash
# Worktree Main: 프로덕션 핫픽스
cd ~/projects/my-app
git checkout main
claude

# 프롬프트:
# "프로덕션에서 발생한 메모리 누수를 수정해줘.
# /var/log/app/error.log 파일을 분석하고 원인을 찾아줘"

# 동시에...

# Worktree Feature: 신규 기능 개발
cd ~/projects/my-app-wt-dashboard
claude

# 프롬프트:
# "실시간 대시보드 컴포넌트를 개발해줘. WebSocket으로
# 서버와 통신하고 Chart.js로 데이터를 시각화해"
```

### 터미널 전환 최적화

#### Shell 별칭 설정

```bash
# ~/.zshrc 또는 ~/.bashrc에 추가

# Worktree 빠른 전환
alias za='cd ~/projects/my-app-wt-a'
alias zb='cd ~/projects/my-app-wt-b'
alias zc='cd ~/projects/my-app-wt-analysis'
alias zm='cd ~/projects/my-app'  # main

# Worktree + Claude 세션 시작
alias cla='za && claude'
alias clb='zb && claude'
alias clc='zc && claude'

# 모든 worktree 상태 확인
alias wtstat='git worktree list && echo "\n=== Git Status ===" && git status'

# Worktree 정리
alias wtclean='git worktree prune'
```

#### 사용 예시

```bash
# 한 번의 명령으로 worktree 전환 + Claude 시작
$ cla
# -> ~/projects/my-app-wt-a로 이동하고 Claude Code 실행

# 모든 worktree 상태 한눈에 보기
$ wtstat
/Users/boris/projects/my-app           abc1234 [main]
/Users/boris/projects/my-app-wt-a      def5678 [feature/user-auth]
/Users/boris/projects/my-app-wt-b      ghi9012 [feature/payment]

=== Git Status ===
On branch feature/user-auth
Your branch is up to date with 'origin/feature/user-auth'.
```

### Tmux/Zellij를 활용한 멀티플렉싱

#### Tmux 세션 구성 예시

```bash
# tmux 세션 생성 및 worktree별 윈도우 설정
tmux new-session -d -s dev -n main -c ~/projects/my-app
tmux new-window -t dev:1 -n auth -c ~/projects/my-app-wt-a
tmux new-window -t dev:2 -n payment -c ~/projects/my-app-wt-b
tmux new-window -t dev:3 -n analysis -c ~/projects/my-app-wt-analysis

# 각 윈도우에서 Claude 실행 (자동화 스크립트)
tmux send-keys -t dev:0 'claude' C-m
tmux send-keys -t dev:1 'claude' C-m
tmux send-keys -t dev:2 'claude' C-m
tmux send-keys -t dev:3 'claude' C-m

# 세션 attach
tmux attach-session -t dev
```

#### Tmux 설정 파일 (~/.tmux.conf)

```bash
# 상태바에 현재 git 브랜치 표시
set -g status-right '#(cd #{pane_current_path}; git rev-parse --abbrev-ref HEAD 2>/dev/null) | %Y-%m-%d %H:%M'

# 윈도우 전환 단축키
bind -n M-1 select-window -t 1
bind -n M-2 select-window -t 2
bind -n M-3 select-window -t 3
bind -n M-4 select-window -t 4

# Pane 간 이동 단축키
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

### 병렬 작업의 실제 효과

#### 시간 절약 계산

**단일 세션 방식:**
```
작업 A: 계획(2분) + 구현(10분) + 대기(3분) = 15분
작업 B: 계획(2분) + 구현(8분) + 대기(2분) = 12분
작업 C: 계획(1분) + 분석(5분) + 대기(2분) = 8분
총 시간: 35분
```

**병렬 세션 방식 (3개 worktree):**
```
작업 A, B, C 동시 시작
최장 작업 완료 시간: 15분 (작업 A)
대기 시간 활용: A 대기 중 B 확인, B 대기 중 C 확인
실제 소요 시간: 약 18-20분 (전환 비용 포함)
```

**생산성 향상: 약 40-45%**

### 고급 Worktree 패턴

#### 1. 전용 분석 Worktree

```bash
# 분석 전용 worktree는 main 브랜치를 유지
git worktree add ../my-app-analysis main

cd ../my-app-analysis
claude

# CLAUDE.md에 분석 전용 규칙 설정
cat > CLAUDE.md << 'EOF'
# Analysis Worktree Rules

이 worktree는 분석 전용입니다. 절대 코드를 수정하지 마세요.

## 허용되는 작업:
- 로그 파일 읽기 및 분석
- BigQuery/데이터베이스 쿼리 실행
- 성능 메트릭 수집
- 버그 패턴 분석
- 리포트 생성

## 금지되는 작업:
- 소스 코드 수정
- git commit
- 파일 삭제

## 기본 분석 도구:
- bq (BigQuery CLI)
- jq (JSON 처리)
- grep, awk (로그 분석)
EOF
```

#### 2. 리뷰 전용 Worktree

```bash
# PR 리뷰용 worktree
git worktree add ../my-app-review pr/user-auth-123

cd ../my-app-review
claude

# 프롬프트:
# "이 PR을 스태프 엔지니어 관점에서 리뷰해줘.
# 다음을 확인해줘:
# 1. 아키텍처 일관성
# 2. 보안 취약점
# 3. 성능 영향
# 4. 테스트 커버리지
# 5. 문서화 누락
#
# 각 항목에 대해 구체적인 개선 사항을 제안해줘"
```

#### 3. 실험 Worktree

```bash
# 실험적 기능 테스트용 worktree
git worktree add ../my-app-experiment experiment/new-architecture

cd ../my-app-experiment
claude

# 프롬프트:
# "현재 MVC 아키텍처를 Clean Architecture로 마이그레이션하는
# POC를 만들어줘. 먼저 플랜 모드로 마이그레이션 전략을 수립하고,
# 한 개 모듈만 먼저 변환해서 효과를 검증해줘"
```

### Worktree 관리 베스트 프랙티스

#### 정기적인 정리

```bash
# 주간 정리 스크립트
cat > ~/bin/clean-worktrees.sh << 'EOF'
#!/bin/bash

echo "🧹 Cleaning up worktrees..."

# 머지된 브랜치의 worktree 제거
cd ~/projects/my-app

for worktree in $(git worktree list | awk '{print $1}' | tail -n +2); do
    cd $worktree
    branch=$(git branch --show-current)
    
    # main에 머지되었는지 확인
    if git branch --merged main | grep -q "$branch"; then
        echo "🗑️  Removing worktree for merged branch: $branch"
        cd ~/projects/my-app
        git worktree remove $worktree
    fi
done

# 고아 worktree 정리
git worktree prune

echo "✅ Cleanup complete!"
EOF

chmod +x ~/bin/clean-worktrees.sh
```

#### Worktree 네이밍 컨벤션

```bash
# 프로젝트-타입-설명 형식
~/projects/my-app-feat-auth      # 기능 개발
~/projects/my-app-fix-memory     # 버그 수정
~/projects/my-app-refactor-db    # 리팩토링
~/projects/my-app-experiment-grpc  # 실험
~/projects/my-app-review-pr123   # PR 리뷰
~/projects/my-app-analysis       # 분석
```

---

## 자기 학습하는 시스템: CLAUDE.md의 복리 효과

### CLAUDE.md의 본질

CLAUDE.md는 단순한 스타일 가이드가 아닙니다. 이는 **"실수 교정의 제도화"**이자 **"팀 지식의 실행 가능한 인코딩"**입니다.

### 기본 CLAUDE.md 구조

~~~markdown
# Project Context

## 프로젝트 개요
이 프로젝트는 [프로젝트 설명]입니다.

## 핵심 아키텍처 결정
- [중요한 아키텍처 결정과 그 이유]

## 절대 하지 말아야 할 것
1. [과거 실수에서 배운 금지 사항]
2. [보안/성능상 중요한 제약사항]

## 코딩 스타일
- [팀 코딩 컨벤션]

## 테스트 요구사항
- [테스트 작성 규칙]

## 배포 프로세스
- [배포 시 확인사항]
~~~

### 실제 CLAUDE.md 예시 1: Node.js API 서버

````markdown
# E-Commerce API Server - Claude Guidelines

## 프로젝트 컨텍스트
이 프로젝트는 Express.js 기반 e-commerce API 서버입니다.
PostgreSQL 데이터베이스와 Redis 캐시를 사용합니다.

## 아키텍처 원칙

### 계층 구조 (엄격하게 준수)
````
routes/ → controllers/ → services/ → repositories/ → models/
```
- Controllers는 절대 DB에 직접 접근하지 않습니다
- Services는 비즈니스 로직을 담당합니다
- Repositories만 DB 쿼리를 실행합니다

### 잘못된 예 (과거 실수)
```javascript
// ❌ Controller에서 직접 DB 접근 (2024-11-03 실수)
app.get('/users/:id', async (req, res) => {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
    res.json(user);
});
```

### 올바른 예
```javascript
// ✅ 계층 구조 준수
// routes/users.js
router.get('/:id', userController.getUser);

// controllers/userController.js
async function getUser(req, res) {
    const user = await userService.getUserById(req.params.id);
    res.json(user);
}

// services/userService.js
async function getUserById(id) {
    const user = await userRepository.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return sanitizeUser(user);
}

// repositories/userRepository.js
async function findById(id) {
    return await db.query('SELECT * FROM users WHERE id = $1', [id]);
}
```

## 에러 처리 규칙

### 커스텀 에러 클래스 사용 필수
```javascript
// ✅ 올바른 방식
throw new NotFoundError('User not found');
throw new ValidationError('Invalid email format');
throw new UnauthorizedError('Invalid credentials');

// ❌ 절대 사용 금지
throw new Error('User not found');  // 일반 Error 사용 금지
```

### 비동기 에러 처리
모든 async route handler는 asyncHandler로 감싸야 합니다:
```javascript
// ✅ 올바른 방식
router.get('/:id', asyncHandler(async (req, res) => {
    const user = await userService.getUserById(req.params.id);
    res.json(user);
}));

// ❌ 잘못된 방식 (2024-10-15 프로덕션 에러)
router.get('/:id', async (req, res) => {
    const user = await userService.getUserById(req.params.id);
    res.json(user);
});
```

## 데이터베이스 규칙

### Transaction 사용 필수 케이스
다음 작업은 반드시 transaction으로 묶어야 합니다:
1. 돈이 관련된 모든 작업 (주문, 결제, 환불)
2. 여러 테이블을 업데이트하는 작업
3. 인벤토리 업데이트

```javascript
// ✅ 올바른 주문 생성
async function createOrder(userId, items) {
    return await db.transaction(async (trx) => {
        // 1. 주문 생성
        const order = await trx('orders').insert({
            user_id: userId,
            status: 'pending'
        }).returning('*');
        
        // 2. 주문 항목 생성
        const orderItems = items.map(item => ({
            order_id: order.id,
            product_id: item.productId,
            quantity: item.quantity
        }));
        await trx('order_items').insert(orderItems);
        
        // 3. 인벤토리 차감
        for (const item of items) {
            await trx('products')
                .where('id', item.productId)
                .decrement('stock', item.quantity);
        }
        
        return order;
    });
}
```

### N+1 쿼리 방지 (2024-09-22 성능 이슈)
```javascript
// ❌ N+1 쿼리 문제
async function getOrdersWithItems(userId) {
    const orders = await db('orders').where('user_id', userId);
    
    for (const order of orders) {
        // 각 주문마다 쿼리 실행 (N+1 문제)
        order.items = await db('order_items').where('order_id', order.id);
    }
    
    return orders;
}

// ✅ JOIN으로 해결
async function getOrdersWithItems(userId) {
    const result = await db('orders')
        .leftJoin('order_items', 'orders.id', 'order_items.order_id')
        .where('orders.user_id', userId)
        .select('orders.*', 'order_items.*');
    
    // 결과를 그룹화
    return groupByOrder(result);
}
```

## 보안 규칙

### 민감 정보 로깅 금지 (2024-08-10 보안 감사 지적)
```javascript
// ❌ 절대 금지
logger.info('User login:', { email, password });  // 비밀번호 로깅
logger.debug('Credit card:', cardNumber);         // 카드번호 로깅
logger.info('Session:', sessionToken);            // 세션 토큰 로깅

// ✅ 올바른 방식
logger.info('User login:', { email, passwordLength: password.length });
logger.debug('Payment processed:', { last4: cardNumber.slice(-4) });
logger.info('Session created:', { userId, expiresAt });
```

### SQL Injection 방지
```javascript
// ❌ 문자열 결합 절대 금지 (2024-07-15 보안 취약점)
const users = await db.raw(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ 파라미터 바인딩 사용
const users = await db.raw('SELECT * FROM users WHERE email = ?', [email]);
// 또는
const users = await db('users').where('email', email);
```

## 테스트 요구사항

### 테스트 커버리지
- 모든 service 함수: 단위 테스트 필수
- 모든 API 엔드포인트: 통합 테스트 필수
- 최소 커버리지: 80%

### 테스트 작성 패턴
```javascript
// ✅ 올바른 테스트 구조
describe('UserService', () => {
    describe('createUser', () => {
        it('should create user with valid data', async () => {
            // Arrange
            const userData = {
                email: 'test@example.com',
                password: 'securePassword123'
            };
            
            // Act
            const user = await userService.createUser(userData);
            
            // Assert
            expect(user).toBeDefined();
            expect(user.email).toBe(userData.email);
            expect(user.password).not.toBe(userData.password); // 해시됨
        });
        
        it('should throw ValidationError for invalid email', async () => {
            const userData = { email: 'invalid', password: 'password' };
            
            await expect(userService.createUser(userData))
                .rejects
                .toThrow(ValidationError);
        });
    });
});
```

## 배포 전 체크리스트

코드를 main 브랜치에 머지하기 전 반드시 확인:
1. [ ] 모든 테스트 통과
2. [ ] 마이그레이션 파일 생성 (DB 스키마 변경 시)
3. [ ] 환경 변수 업데이트 문서화
4. [ ] CHANGELOG.md 업데이트
5. [ ] API 문서 업데이트 (엔드포인트 변경 시)

## 과거 중대 사고 교훈

### 2024-12-01: 재고 관리 버그
**문제**: Transaction 없이 재고를 업데이트하여 동시성 이슈 발생
**영향**: 초과 판매로 100건의 주문 취소
**해결책**: 모든 재고 업데이트에 pessimistic locking 적용
```javascript
// ✅ 현재 방식
await trx('products')
    .where('id', productId)
    .forUpdate()  // Row-level lock
    .first();
```

### 2024-10-20: 메모리 누수
**문제**: Event listener 해제 누락
**영향**: 서버 메모리 사용량 지속 증가로 매일 재시작 필요
**해결책**: 모든 event listener에 cleanup 함수 작성
```javascript
// ✅ 올바른 패턴
function setupWebSocket() {
    const ws = new WebSocket(url);
    
    ws.on('message', handleMessage);
    
    // cleanup 함수 반환
    return () => {
        ws.off('message', handleMessage);
        ws.close();
    };
}
```

## 성능 최적화 가이드

### Redis 캐싱 전략
- 사용자 프로필: TTL 15분
- 상품 목록: TTL 5분
- 장바구니: TTL 1시간

```javascript
// ✅ 캐싱 패턴
async function getUser(id) {
    const cacheKey = `user:${id}`;
    
    // 캐시 확인
    let user = await redis.get(cacheKey);
    if (user) return JSON.parse(user);
    
    // DB 조회
    user = await userRepository.findById(id);
    
    // 캐시 저장
    await redis.setex(cacheKey, 900, JSON.stringify(user)); // 15분
    
    return user;
}
```

## 모니터링 및 알림

### 필수 로깅 이벤트
```javascript
// 반드시 로깅해야 하는 이벤트
logger.info('order.created', { orderId, userId, amount });
logger.info('payment.processed', { orderId, paymentId, method });
logger.error('payment.failed', { orderId, error, retryCount });
logger.warn('stock.low', { productId, currentStock, threshold });
```

## 참고 문서
- API 문서: `/docs/api.md`
- 데이터베이스 스키마: `/docs/schema.md`
- 배포 가이드: `/docs/deployment.md`
```

### 실제 CLAUDE.md 예시 2: React 프론트엔드

````markdown
# Dashboard App - Claude Guidelines

## 프로젝트 구조
````
src/
├── components/     # 재사용 가능한 UI 컴포넌트
├── features/       # 기능별 모듈 (Redux Toolkit 슬라이스 포함)
├── hooks/          # 커스텀 훅
├── services/       # API 클라이언트
├── utils/          # 유틸리티 함수
└── pages/          # 라우트 페이지 컴포넌트
```

## 컴포넌트 작성 규칙

### 함수형 컴포넌트 + Hooks 사용
```typescript
// ✅ 올바른 방식
import { useState, useEffect } from 'react';

interface UserProfileProps {
    userId: string;
}

export function UserProfile({ userId }: UserProfileProps) {
    const [user, setUser] = useState<User | null>(null);
    
    useEffect(() => {
        loadUser(userId);
    }, [userId]);
    
    return <div>{user?.name}</div>;
}

// ❌ 클래스 컴포넌트 사용 금지 (2024-11-01 결정)
class UserProfile extends React.Component {
    // ...
}
```

### Props Destructuring
```typescript
// ✅ 올바른 방식
export function Button({ label, onClick, variant = 'primary' }: ButtonProps) {
    return <button onClick={onClick}>{label}</button>;
}

// ❌ 피해야 할 방식
export function Button(props: ButtonProps) {
    return <button onClick={props.onClick}>{props.label}</button>;
}
```

## 상태 관리 규칙

### Redux Toolkit 슬라이스 패턴
```typescript
// ✅ 표준 슬라이스 구조
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk
export const fetchUsers = createAsyncThunk(
    'users/fetch',
    async () => {
        const response = await api.get('/users');
        return response.data;
    }
);

// Slice
const usersSlice = createSlice({
    name: 'users',
    initialState: {
        items: [],
        status: 'idle',
        error: null
    },
    reducers: {
        // Sync actions
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchUsers.pending, (state) => {
                state.status = 'loading';
            })
            .addCase(fetchUsers.fulfilled, (state, action) => {
                state.status = 'succeeded';
                state.items = action.payload;
            })
            .addCase(fetchUsers.rejected, (state, action) => {
                state.status = 'failed';
                state.error = action.error.message;
            });
    }
});
```

### 로컬 vs 글로벌 상태 (2024-09-10 아키텍처 결정)
```typescript
// ✅ 로컬 상태 사용 (컴포넌트 내부에서만 사용)
function SearchBar() {
    const [query, setQuery] = useState('');  // 로컬 상태 OK
    return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// ✅ 글로벌 상태 사용 (여러 컴포넌트에서 공유)
function UserList() {
    const users = useSelector(state => state.users.items);  // Redux
    return <ul>{users.map(user => <li>{user.name}</li>)}</ul>;
}

// ❌ 잘못된 예: 로컬 상태를 Redux에 저장
// 단순한 UI 상태는 Redux에 넣지 마세요
```

## API 호출 패턴

### React Query 사용 (2024-10-15 도입)
```typescript
// ✅ 올바른 방식
import { useQuery, useMutation } from '@tanstack/react-query';

function UserProfile({ userId }: { userId: string }) {
    const { data: user, isLoading, error } = useQuery({
        queryKey: ['user', userId],
        queryFn: () => api.get(`/users/${userId}`),
        staleTime: 5 * 60 * 1000  // 5분
    });
    
    if (isLoading) return <Spinner />;
    if (error) return <ErrorMessage error={error} />;
    
    return <div>{user.name}</div>;
}

// Mutation 예시
function UpdateUserForm({ userId }: { userId: string }) {
    const mutation = useMutation({
        mutationFn: (userData) => api.put(`/users/${userId}`, userData),
        onSuccess: () => {
            queryClient.invalidateQueries(['user', userId]);
        }
    });
    
    return <form onSubmit={e => mutation.mutate(formData)}>...</form>;
}
```

### 에러 처리
```typescript
// ✅ 에러 바운더리 사용
import { ErrorBoundary } from 'react-error-boundary';

function App() {
    return (
        <ErrorBoundary
            FallbackComponent={ErrorFallback}
            onError={(error, errorInfo) => {
                logger.error('React error:', error, errorInfo);
            }}
        >
            <YourApp />
        </ErrorBoundary>
    );
}

// ❌ try-catch를 컴포넌트 렌더링에 사용하지 마세요
function BadComponent() {
    try {
        return <div>{data.value}</div>;
    } catch (error) {
        return <div>Error</div>;
    }
}
```

## 성능 최적화 규칙

### 메모이제이션 (2024-08-05 성능 개선)
```typescript
// ✅ 비용이 큰 계산에 useMemo 사용
function DataGrid({ data }: { data: Item[] }) {
    const sortedData = useMemo(() => {
        return [...data].sort((a, b) => a.name.localeCompare(b.name));
    }, [data]);
    
    return <Table data={sortedData} />;
}

// ✅ 콜백 함수에 useCallback 사용
function SearchableList({ items }: { items: Item[] }) {
    const [query, setQuery] = useState('');
    
    const handleSearch = useCallback((value: string) => {
        setQuery(value);
        analytics.track('search', { query: value });
    }, []);  // 의존성 없음
    
    return <SearchBar onSearch={handleSearch} />;
}

// ❌ 과도한 메모이제이션 금지
function SimpleComponent({ name }: { name: string }) {
    // 불필요한 useMemo
    const displayName = useMemo(() => name.toUpperCase(), [name]);
    return <div>{displayName}</div>;
}
```

### 리스트 렌더링 최적화
```typescript
// ✅ key prop 올바르게 사용
function UserList({ users }: { users: User[] }) {
    return (
        <ul>
            {users.map(user => (
                <UserItem key={user.id} user={user} />  // 안정적인 ID 사용
            ))}
        </ul>
    );
}

// ❌ 인덱스를 key로 사용 금지 (2024-07-20 버그)
function BadUserList({ users }: { users: User[] }) {
    return (
        <ul>
            {users.map((user, index) => (
                <UserItem key={index} user={user} />  // 순서 변경 시 문제
            ))}
        </ul>
    );
}
```

## 스타일링 규칙

### Tailwind CSS 클래스 순서
```typescript
// ✅ 올바른 클래스 순서 (prettier-plugin-tailwindcss 사용)
// 레이아웃 → 간격 → 크기 → 타이포그래피 → 색상 → 기타
<div className="flex items-center justify-between gap-4 p-4 text-lg font-bold text-blue-600 rounded-lg shadow-md">
```

### 조건부 스타일링
```typescript
// ✅ clsx/classnames 사용
import clsx from 'clsx';

function Button({ variant, disabled }: ButtonProps) {
    return (
        <button
            className={clsx(
                'px-4 py-2 rounded-lg font-medium',
                variant === 'primary' && 'bg-blue-600 text-white',
                variant === 'secondary' && 'bg-gray-200 text-gray-900',
                disabled && 'opacity-50 cursor-not-allowed'
            )}
        >
            Click me
        </button>
    );
}
```

## 접근성 규칙 (2024-11-15 추가)

### ARIA 레이블 필수
```typescript
// ✅ 올바른 방식
<button aria-label="Close modal" onClick={onClose}>
    <XIcon />
</button>

<input
    type="search"
    aria-label="Search users"
    placeholder="Search..."
/>

// ❌ 아이콘만 있는 버튼에 레이블 없음
<button onClick={onClose}>
    <XIcon />
</button>
```

### 키보드 네비게이션
```typescript
// ✅ 키보드 이벤트 핸들링
function Modal({ onClose }: ModalProps) {
    useEffect(() => {
        const handleEscape = (e: KeyboardEvent) => {
            if (e.key === 'Escape') onClose();
        };
        
        window.addEventListener('keydown', handleEscape);
        return () => window.removeEventListener('keydown', handleEscape);
    }, [onClose]);
    
    return <div role="dialog" aria-modal="true">...</div>;
}
```

## 테스트 규칙

### 테스트 작성 우선순위
1. 사용자 인터랙션 (버튼 클릭, 폼 제출)
2. 조건부 렌더링
3. API 연동
4. 에러 상태

```typescript
// ✅ React Testing Library 사용
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

describe('LoginForm', () => {
    it('should submit form with valid credentials', async () => {
        // Arrange
        const mockLogin = jest.fn();
        render(<LoginForm onLogin={mockLogin} />);
        
        // Act
        fireEvent.change(screen.getByLabelText('Email'), {
            target: { value: 'user@example.com' }
        });
        fireEvent.change(screen.getByLabelText('Password'), {
            target: { value: 'password123' }
        });
        fireEvent.click(screen.getByRole('button', { name: 'Login' }));
        
        // Assert
        await waitFor(() => {
            expect(mockLogin).toHaveBeenCalledWith({
                email: 'user@example.com',
                password: 'password123'
            });
        });
    });
});
```

## 과거 실수 모음

### 2024-12-10: 무한 루프 (useEffect 의존성)
```typescript
// ❌ 문제 코드
function UserProfile() {
    const [user, setUser] = useState(null);
    
    useEffect(() => {
        fetchUser().then(setUser);
    }, [user]);  // user가 변경되면 다시 fetch → 무한 루프
}

// ✅ 해결
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    
    useEffect(() => {
        fetchUser(userId).then(setUser);
    }, [userId]);  // userId 변경 시에만 fetch
}
```

### 2024-11-25: 메모리 누수 (이벤트 리스너)
```typescript
// ❌ 문제 코드
function WindowSize() {
    const [size, setSize] = useState({ width: 0, height: 0 });
    
    useEffect(() => {
        const handleResize = () => {
            setSize({ width: window.innerWidth, height: window.innerHeight });
        };
        window.addEventListener('resize', handleResize);
        // cleanup 함수 없음 → 메모리 누수
    }, []);
}

// ✅ 해결
function WindowSize() {
    const [size, setSize] = useState({ width: 0, height: 0 });
    
    useEffect(() => {
        const handleResize = () => {
            setSize({ width: window.innerWidth, height: window.innerHeight });
        };
        window.addEventListener('resize', handleResize);
        return () => window.removeEventListener('resize', handleResize);
    }, []);
}
```
```

### Notes 디렉토리 패턴

```bash
# 프로젝트 구조
my-app/
├── CLAUDE.md
├── notes/
│   ├── architecture-decisions.md
│   ├── api-integration.md
│   ├── performance-optimization.md
│   └── troubleshooting.md
└── src/

# CLAUDE.md에서 notes 참조
cat >> CLAUDE.md << 'EOF'
## 참고 문서

작업 시 다음 노트를 참조하세요:
- 아키텍처 결정사항: `notes/architecture-decisions.md`
- API 통합 가이드: `notes/api-integration.md`
- 성능 최적화 기록: `notes/performance-optimization.md`
- 문제 해결 로그: `notes/troubleshooting.md`
EOF
```

### 실수 교정 워크플로우

````markdown
# 실수 발생 시 프로세스

1. **Claude가 실수를 함**
   예: Transaction 없이 재고를 업데이트

2. **즉시 CLAUDE.md 업데이트 요청**
````
   이 실수를 다시 하지 않도록 CLAUDE.md를 업데이트해줘.
   
   다음 내용을 포함해:
   1. 무엇이 잘못되었는지
   2. 왜 문제인지
   3. 올바른 방법은 무엇인지
   4. 코드 예시 (잘못된 것 vs 올바른 것)
   ```

3. **Claude의 CLAUDE.md 업데이트 예시**
````markdown
   ## 재고 관리 규칙 (2026-02-01 추가)
   
   ### 문제
   Transaction 없이 재고를 업데이트하면 동시성 이슈가 발생합니다.
   
   ### 영향
   - 같은 상품에 대한 동시 주문 시 초과 판매 가능
   - 재고가 음수가 될 수 있음
   
   ### 해결책
   모든 재고 변경은 반드시 transaction 내에서 수행하고,
   pessimistic locking을 사용하세요.
   
   #### ❌ 잘못된 방식
   ```javascript
   async function decreaseStock(productId, quantity) {
       const product = await db('products').where('id', productId).first();
       await db('products')
           .where('id', productId)
           .update({ stock: product.stock - quantity });
   }
   ```
   
   #### ✅ 올바른 방식
   ```javascript
   async function decreaseStock(productId, quantity) {
       return await db.transaction(async (trx) => {
           const product = await trx('products')
               .where('id', productId)
               .forUpdate()  // Row-level lock
               .first();
           
           if (product.stock < quantity) {
               throw new Error('Insufficient stock');
           }
           
           await trx('products')
               .where('id', productId)
               .update({ stock: product.stock - quantity });
       });
   }
   ```
````

4. **변경사항 커밋**
   ```bash
   git add CLAUDE.md
   git commit -m "docs: Add inventory management rules to prevent concurrency issues"
   ```

### 팀 차원의 CLAUDE.md 관리

#### PR 체크리스트에 CLAUDE.md 포함

````markdown
# Pull Request Template

## 변경사항
- [ ] 코드 변경
- [ ] 테스트 추가
- [ ] 문서 업데이트
- [ ] **CLAUDE.md 업데이트** (새로운 규칙/패턴이 있다면)

## CLAUDE.md 업데이트 내용
