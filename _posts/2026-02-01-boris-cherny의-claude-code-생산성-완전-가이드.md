---
title: "Boris Cherny의 Claude Code 생산성 완전 가이드"
date: 2026-02-01 16:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  Material,  claude-code,  BorisCherny,  Claude.write]
---


> Claude Code 창시자 Boris Cherny와 개발팀이 직접 공유하는 실전 생산성 향상 전략  
> 작성일: 2026-02-01


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

~~~markdown
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
~~~

### 실제 CLAUDE.md 예시 2: React 프론트엔드

~~~markdown
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
~~~

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

~~~markdown
# 실수 발생 시 프로세스

1. **Claude가 실수를 함**
   예: Transaction 없이 재고를 업데이트

2. **즉시 CLAUDE.md 업데이트 요청**
   ```
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
~~~

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
//새로운 규칙을 추가했다면 여기에 설명
````

#### CLAUDE.md 리뷰 프로세스

````markdown
# CLAUDE.md 변경 리뷰 가이드

## 확인사항
1. [ ] 규칙이 명확하고 구체적인가?
2. [ ] 코드 예시가 포함되어 있는가?
3. [ ] 날짜가 기록되어 있는가?
4. [ ] 다른 규칙과 충돌하지 않는가?
5. [ ] 너무 장황하지 않은가? (간결성 유지)

## 좋은 규칙의 예
```markdown
## API 응답 형식 (2026-01-15)

모든 API는 다음 형식으로 응답해야 합니다:

```json
{
    "success": true,
    "data": { ... },
    "error": null
}
```

❌ 잘못된 예: `return data`
✅ 올바른 예: `return { success: true, data, error: null }`
```

## 나쁜 규칙의 예
```markdown
// 너무 모호함
## 코드 작성 시 주의사항
코드를 작성할 때는 항상 조심하세요.

// 구체적 예시 없음
## 에러 처리
에러는 적절하게 처리해야 합니다.
```
````

---

## 프롬프팅의 레벨업: 지시에서 협업으로

### 플랜 모드의 전략적 활용

#### 기본 플랜 모드 사용법

```
# 프롬프트 예시 1: 복잡한 기능 개발
먼저 플랜 모드로 전환해서 사용자 인증 시스템 전체 아키텍처를 설계해줘.

다음을 고려해:
1. JWT vs Session 기반 인증 선택
2. 비밀번호 암호화 방식
3. Refresh Token 구현
4. OAuth 2.0 통합 (Google, GitHub)
5. Rate Limiting
6. 보안 베스트 프랙티스

계획이 완성되면, 내가 리뷰할 수 있게 대기해줘.
```

#### 이중 Claude 리뷰 패턴

```bash
# Terminal 1: 계획 작성 Claude
cd ~/projects/my-app-wt-a
claude

# 프롬프트:
> 결제 시스템 통합 계획을 작성해줘. Stripe과 PayPal을 지원해야 하고,
> 구독과 일회성 결제를 모두 처리할 수 있어야 해.
>
> 플랜 모드로 다음을 포함해:
> 1. 데이터베이스 스키마
> 2. API 엔드포인트 설계
> 3. 에러 처리 전략
> 4. 웹훅 처리
> 5. 테스트 계획

# Claude가 계획을 작성하면...

---

# Terminal 2: 리뷰 Claude (Worktree 별도)
cd ~/projects/my-app-wt-review
claude

# 프롬프트:
> 당신은 시니어 스태프 엔지니어입니다.
> 다음 결제 시스템 설계를 리뷰해주세요:
>
> [Terminal 1의 계획을 복사해서 붙여넣기]
>
> 다음 관점에서 검토해주세요:
> 1. 아키텍처 일관성
> 2. 보안 취약점
> 3. 확장성 문제
> 4. 에러 처리 누락
> 5. 테스트 커버리지 충분성
> 6. 비용 효율성
>
> 각 항목에 대해 구체적인 개선사항을 제안해주세요.
```

#### 중간 재계획 패턴

```
# 초기 계획대로 진행 중...
> 사용자 인증 시스템을 구현해줘. 이전에 작성한 계획을 따라서.

# 문제 발생 시
> 잠깐, 멈춰. 현재 구현에서 다음 문제를 발견했어:
> - JWT 토큰이 너무 커서 쿠키 제한을 초과함
> - Refresh token 저장소가 Redis인데 비용이 너무 높음
>
> 플랜 모드로 돌아가서 이 문제들을 해결하는 새로운 계획을 세워줘.
> 기존 계획에서 변경이 필요한 부분만 다시 작성해.
```

#### 검증 단계 플랜 모드

```
# 구현 완료 후
> 구현이 완료됐네. 이제 플랜 모드로 전환해서
> 검증 계획을 세워줘.
>
> 다음을 포함해:
> 1. 단위 테스트 시나리오
> 2. 통합 테스트 시나리오
> 3. 보안 테스트 (SQL Injection, XSS 등)
> 4. 성능 테스트
> 5. 엣지 케이스 확인
>
> 각 테스트에 대한 성공 기준을 명확히 정의해.
```

### 도전적 프롬프팅

#### 패턴 1: Claude를 테스터로

```
# 기본 프롬프트
> 이 PR의 변경사항을 테스트해줘.

# 도전적 프롬프트
> 당신은 까다로운 QA 엔지니어입니다.
> 이 PR을 통과시키기 전에 나를 테스트해주세요.
>
> 다음을 확인하세요:
> 1. 모든 엣지 케이스가 처리되는가?
> 2. 에러 메시지가 충분히 명확한가?
> 3. 성능 저하가 없는가?
> 4. 보안 취약점은 없는가?
> 5. 문서가 충분한가?
>
> 각 항목에 대해 실제로 테스트를 실행하고,
> 내가 모든 테스트를 통과할 때까지 PR을 승인하지 마세요.
>
> 테스트가 실패하면 구체적인 개선사항을 요구하세요.
```

#### 실제 대화 예시

```
User: 이 변경사항에 대해 나를 테스트해. 내가 통과할 때까지 PR 만들지 마.

Claude: 알겠습니다. QA 엔지니어로서 철저히 검토하겠습니다.

[테스트 1] 입력 검증
❌ 실패: 이메일 주소에 공백이 들어가는 경우를 처리하지 않습니다.
- 테스트: " user@example.com " 입력
- 예상: 공백 제거 후 저장
- 실제: 공백이 포함된 채로 저장됨

개선 요구: trim() 함수로 공백을 제거하세요.

[테스트 2] 보안
❌ 실패: 비밀번호가 평문으로 로깅됩니다.
- 파일: src/auth/login.js:45
- 내용: logger.info('Login attempt:', { email, password })

개선 요구: 비밀번호를 절대 로깅하지 마세요.

모든 테스트를 통과하면 다시 알려주세요.

User: 두 문제를 모두 수정했어.

Claude: 재테스트하겠습니다...

[테스트 1] 입력 검증
✅ 통과: 이메일 공백이 올바르게 제거됩니다.

[테스트 2] 보안
✅ 통과: 비밀번호 로깅이 제거되었습니다.

[테스트 3] 성능
✅ 통과: 쿼리 성능에 영향 없음 (실행시간 <50ms)

[테스트 4] 에러 처리
✅ 통과: 모든 에러 케이스가 적절히 처리됩니다.

모든 테스트를 통과했습니다! PR을 생성하겠습니다.
```

#### 패턴 2: 동작 증명 요구

```
# 기본 프롬프트
> 이 버그를 수정해줘.

# 도전적 프롬프트
> 이 버그를 수정하되, 수정 전후의 동작 차이를 증명해줘.
>
> 다음 단계를 따라:
> 1. main 브랜치로 전환
> 2. 버그를 재현하는 테스트 작성
> 3. 테스트가 실패하는 것을 확인 (스크린샷)
> 4. 수정 브랜치로 전환
> 5. 버그 수정
> 6. 동일한 테스트가 통과하는 것을 확인
> 7. before/after 비교 리포트 작성
>
> 증명이 완료될 때까지 PR을 만들지 마.
```

#### 패턴 3: 우아한 솔루션 요구

```
# 1차 구현 후
User: 작동은 하는데 코드가 좀 지저분해 보여.

# 기본 프롬프트
> 코드를 리팩토링해줘.

# 도전적 프롬프트
> 지금까지 배운 모든 것을 고려해서, 현재 구현을 버리고
> 우아한 솔루션을 처음부터 다시 구현해줘.
>
> "우아함"의 기준:
> 1. 코드 라인 수 30% 이상 감소
> 2. 순환 복잡도(Cyclomatic Complexity) 감소
> 3. 재사용 가능한 함수로 분리
> 4. 명확한 책임 분리 (SRP)
> 5. 테스트하기 쉬운 구조
>
> 리팩토링 전후 메트릭을 비교해서 보여줘.
```

#### 실제 예시: 우아한 솔루션 요구

```
# 1차 구현 (지저분함)
async function processOrder(orderId) {
    const order = await db.query('SELECT * FROM orders WHERE id = $1', [orderId]);
    if (!order) throw new Error('Order not found');
    
    const items = await db.query('SELECT * FROM order_items WHERE order_id = $1', [orderId]);
    let total = 0;
    for (let item of items) {
        const product = await db.query('SELECT * FROM products WHERE id = $1', [item.product_id]);
        if (product.stock < item.quantity) throw new Error('Insufficient stock');
        total += product.price * item.quantity;
    }
    
    if (order.user_discount) {
        const user = await db.query('SELECT * FROM users WHERE id = $1', [order.user_id]);
        total = total * (1 - user.discount_rate);
    }
    
    await db.query('UPDATE orders SET total = $1, status = $2 WHERE id = $3', 
        [total, 'processed', orderId]);
    
    for (let item of items) {
        await db.query('UPDATE products SET stock = stock - $1 WHERE id = $2',
            [item.quantity, item.product_id]);
    }
    
    return order;
}

# 프롬프트
> 지금까지 배운 모든 것을 고려해서 우아한 솔루션을 구현해줘.

# Claude의 2차 구현 (우아함)
class OrderProcessor {
    async process(orderId) {
        return await db.transaction(async (trx) => {
            const order = await this.getOrder(trx, orderId);
            const items = await this.getOrderItems(trx, orderId);
            
            await this.validateStock(trx, items);
            const total = await this.calculateTotal(trx, order, items);
            
            await this.updateOrder(trx, orderId, total);
            await this.decreaseStock(trx, items);
            
            return order;
        });
    }
    
    async getOrder(trx, orderId) {
        const order = await trx('orders').where('id', orderId).first();
        if (!order) throw new NotFoundError('Order not found');
        return order;
    }
    
    async getOrderItems(trx, orderId) {
        return await trx('order_items')
            .join('products', 'order_items.product_id', 'products.id')
            .where('order_items.order_id', orderId)
            .select('order_items.*', 'products.price', 'products.stock');
    }
    
    async validateStock(trx, items) {
        const insufficientStock = items.find(item => item.stock < item.quantity);
        if (insufficientStock) {
            throw new ValidationError(`Insufficient stock for product ${insufficientStock.product_id}`);
        }
    }
    
    async calculateTotal(trx, order, items) {
        const subtotal = items.reduce((sum, item) => 
            sum + (item.price * item.quantity), 0);
        
        if (!order.user_discount) return subtotal;
        
        const user = await trx('users').where('id', order.user_id).first();
        return subtotal * (1 - user.discount_rate);
    }
    
    async updateOrder(trx, orderId, total) {
        await trx('orders')
            .where('id', orderId)
            .update({ total, status: 'processed' });
    }
    
    async decreaseStock(trx, items) {
        const updates = items.map(item =>
            trx('products')
                .where('id', item.product_id)
                .decrement('stock', item.quantity)
        );
        await Promise.all(updates);
    }
}

# 메트릭 비교
Before:
- Lines of code: 42
- Cyclomatic Complexity: 8
- Database queries: 2 + 2N (N = 상품 수)
- Transaction safety: ❌
- Testability: Low

After:
- Lines of code: 58 (더 읽기 쉬움)
- Cyclomatic Complexity: 2 (평균)
- Database queries: 3 + 1 (최적화됨)
- Transaction safety: ✅
- Testability: High (각 메서드 독립 테스트 가능)
```

### 구체성의 예술

#### 모호한 프롬프트 vs 구체적 프롬프트

```
# ❌ 모호한 프롬프트
> REST API를 만들어줘.

# ✅ 구체적 프롬프트
> 다음 요구사항을 만족하는 REST API를 만들어줘:
>
> **기능**: 사용자 관리 CRUD
>
> **기술 스택**:
> - Node.js + Express.js
> - PostgreSQL (Knex.js)
> - JWT 인증
>
> **엔드포인트**:
> 1. POST /auth/register - 사용자 등록
> 2. POST /auth/login - 로그인 (JWT 발급)
> 3. GET /users/:id - 사용자 조회 (인증 필요)
> 4. PUT /users/:id - 사용자 수정 (본인만 가능)
> 5. DELETE /users/:id - 사용자 삭제 (관리자만 가능)
>
> **검증 규칙**:
> - 이메일: 유효한 형식, 중복 불가
> - 비밀번호: 최소 8자, 대소문자+숫자+특수문자 포함
> - 이름: 2-50자
>
> **에러 응답 형식**:
> ```json
> {
>   "success": false,
>   "error": {
>     "code": "VALIDATION_ERROR",
>     "message": "이메일 형식이 올바르지 않습니다",
>     "field": "email"
>   }
> }
> ```
>
> **보안 요구사항**:
> - 비밀번호는 bcrypt로 해싱 (saltRounds: 10)
> - JWT secret은 환경변수에서 로드
> - SQL Injection 방지
> - Rate limiting (분당 100 요청)
>
> **테스트**:
> - 각 엔드포인트에 대한 성공/실패 케이스
> - 최소 커버리지 80%
>
> 먼저 플랜 모드로 전체 구조를 설계한 후,
> 내가 승인하면 구현을 시작해줘.
```

#### 작업 컨텍스트 제공

```
# ❌ 컨텍스트 없음
> 이 버그를 고쳐줘.

# ✅ 풍부한 컨텍스트
> **버그 리포트**:
> 제목: 장바구니에서 상품 삭제 시 가격이 업데이트되지 않음
>
> **재현 단계**:
> 1. 장바구니에 상품 3개 추가
> 2. 총액: $100
> 3. 중간 상품($30) 삭제
> 4. 예상 총액: $70
> 5. 실제 총액: $100 (변경 없음)
>
> **환경**:
> - 브라우저: Chrome 120
> - OS: macOS 14.2
> - 앱 버전: 2.3.1
>
> **에러 로그**:
> ```
> [2026-02-01 10:23:45] ERROR: Cannot read property 'price' of undefined
>   at calculateTotal (cart.js:156)
>   at removeItem (cart.js:89)
> ```
>
> **영향 범위**:
> - 모든 사용자
> - 프로덕션 환경
> - 발생률: 100%
>
> **우선순위**: P0 (즉시 수정 필요)
>
> 다음 단계로 진행해줘:
> 1. cart.js 파일 분석
> 2. 버그 원인 파악
> 3. 수정 계획 수립 (플랜 모드)
> 4. 테스트 작성
> 5. 버그 수정
> 6. 회귀 테스트 확인
```

### 반복적 개선 프롬프팅

```
# 1차 시도
User: 사용자 대시보드 컴포넌트를 만들어줘.

Claude: [기본적인 대시보드 생성]

# 2차 개선
User: 괜찮은데, 다음을 개선해줘:
> 1. 로딩 상태 추가 (Skeleton UI)
> 2. 에러 상태 처리
> 3. 빈 상태 처리 (데이터 없을 때)
> 4. 반응형 디자인 (모바일 지원)

Claude: [개선된 버전 생성]

# 3차 최적화
User: 좋아졌어. 이제 지금까지 배운 모든 것을 바탕으로,
> 현재 구현을 버리고 프로덕션 수준의 우아한 솔루션을 만들어줘.
>
> 추가 요구사항:
> - 접근성 (ARIA 레이블, 키보드 네비게이션)
> - 성능 최적화 (메모이제이션, 가상화)
> - 애니메이션 (Framer Motion)
> - 다크모드 지원
> - 국제화 (i18n)
>
> 이전 버전과 비교해서 개선사항을 설명해줘.

Claude: [프로덕션 수준의 최종 버전 생성]
```

---

## 터미널 환경 최적화

### Ghostty 터미널 추천 사유

Claude Code 팀이 선호하는 터미널 에뮬레이터는 Ghostty입니다.

#### Ghostty의 장점
1. **Synchronized Rendering**: 화면 깜빡임 없는 부드러운 렌더링
2. **24-bit True Color**: 풍부한 색상 표현
3. **Proper Unicode Support**: 이모지와 다국어 문자 완벽 지원
4. **GPU 가속**: 빠른 성능

#### Ghostty 설치

```bash
# macOS (Homebrew)
brew install --cask ghostty

# 설정 파일 생성
mkdir -p ~/.config/ghostty
cat > ~/.config/ghostty/config << 'EOF'
# Font
font-family = "JetBrains Mono"
font-size = 14

# Colors (Tokyo Night theme)
background = 1a1b26
foreground = c0caf5
cursor-color = c0caf5

# Performance
resize-overlay = never
scrollback-lines = 10000

# Shell integration
shell-integration = true
EOF
```

### /statusline 커스터마이징

```bash
# Claude Code 설정에서 statusline 커스터마이즈
claude config

# 상태바 포맷 설정
/statusline {git_branch} | {context_usage} | {model}

# 예시 출력:
# feature/user-auth | 45k/200k | sonnet-4.5
```

#### 고급 Statusline 설정

```bash
# Starship 프롬프트와 통합
# ~/.config/starship.toml

[custom.claude_context]
command = "claude status --format='{context_used}/{context_total}'"
when = true
format = "[$output]($style) "
style = "bold yellow"

[git_branch]
symbol = " "
format = "[$symbol$branch]($style) "
style = "bold purple"

# 결과:
#  feature/user-auth 45k/200k ❯
```

### 음성 입력 활용

#### macOS 음성 딕테이션

```bash
# 활성화 방법
System Preferences → Keyboard → Dictation → Enable

# 단축키: Fn 키 2회 연속 탭

# 프롬프트 예시 (말하기)
"플랜 모드로 전환해서, 사용자 인증 시스템을 설계해줘. 
다음을 포함해. 첫째, JWT 기반 인증. 둘째, 리프레시 토큰 메커니즘. 
셋째, OAuth 통합. 넷째, 레이트 리미팅."

# 장점
- 타이핑 대비 3배 빠름
- 더 자연스럽고 상세한 프롬프트
- 손목 부담 감소
```

#### 음성 입력 팁

```
# ✅ 좋은 음성 프롬프트
"플랜 모드로 시작해줘. 콤마. 
사용자 인증 시스템 전체를 설계하는데 콤마.
다음 항목들을 고려해. 콜론. 뉴라인.
첫째 콤마, JWT 대 세션 인증 비교. 뉴라인.
둘째 콤마, 비밀번호 암호화 방식. 뉴라인.
셋째 콤마, 리프레시 토큰 구현."

# 음성 명령어
- "콤마" = ,
- "콜론" = :
- "뉴라인" = 줄바꿈
- "대시" = -
- "별표" = *
- "백틱 3개" = ```
```

### 터미널 테마 설정

#### Tokyo Night 테마

```bash
# ~/.zshrc 또는 ~/.bashrc
# Tokyo Night 색상 스킴

export LS_COLORS="di=1;35:ln=1;36:so=1;31:pi=1;33:ex=1;32:bd=34;46:cd=34;43:su=30;41:sg=30;46:tw=30;42:ow=30;43"

# fzf 테마
export FZF_DEFAULT_OPTS="--color=bg+:#283457,bg:#16161e,spinner:#bb9af7,hl:#7aa2f7 \
--color=fg:#c0caf5,header:#7aa2f7,info:#7dcfff,pointer:#bb9af7 \
--color=marker:#9ece6a,fg+:#c0caf5,prompt:#bb9af7,hl+:#7aa2f7"

# bat 테마
export BAT_THEME="TwoDark"
```

### 셸 별칭 및 함수

```bash
# ~/.zshrc 또는 ~/.bashrc에 추가

# Claude 관련 별칭
alias c='claude'
alias cp='claude --plan'  # 플랜 모드로 시작
alias cv='claude --verbose'  # 상세 출력

# Git Worktree 별칭
alias wt='git worktree'
alias wta='git worktree add'
alias wtl='git worktree list'
alias wtr='git worktree remove'
alias wtp='git worktree prune'

# 복합 명령
alias wtc='git worktree add ../$(basename $(pwd))-wt-$(git branch --show-current) $(git branch --show-current)'

# Claude + Worktree 통합 함수
cwt() {
    # 현재 브랜치로 새 worktree 생성 후 Claude 시작
    local branch=$(git branch --show-current)
    local worktree_name="../$(basename $(pwd))-wt-$branch"
    
    git worktree add "$worktree_name" "$branch"
    cd "$worktree_name"
    claude
}

# 모든 worktree에서 git pull
wtpull() {
    for worktree in $(git worktree list --porcelain | grep "^worktree" | cut -d' ' -f2); do
        echo "📥 Pulling in $worktree"
        (cd "$worktree" && git pull)
    done
}

# Worktree 상태 대시보드
wtstatus() {
    echo "📊 Worktree Status Dashboard"
    echo "================================"
    
    git worktree list --porcelain | awk '
        /^worktree/ { path=$2; getline; branch=$2 }
        /^detached/ { branch="DETACHED" }
        /^branch/ { 
            system("cd " path " && git status --short | wc -l | tr -d \" \"")
            print path " | " branch " | " 
        }
    ' | column -t -s'|'
}
```

### tmux/zellij 세션 관리

#### tmux 세션 스크립트

```bash
#!/bin/bash
# ~/bin/dev-session.sh

SESSION_NAME="dev"
PROJECT_PATH="$HOME/projects/my-app"

# 기존 세션 종료
tmux kill-session -t $SESSION_NAME 2>/dev/null

# 새 세션 생성
tmux new-session -d -s $SESSION_NAME -n main -c $PROJECT_PATH

# Worktree 윈도우 생성
tmux new-window -t $SESSION_NAME:1 -n auth -c "$PROJECT_PATH-wt-a"
tmux new-window -t $SESSION_NAME:2 -n payment -c "$PROJECT_PATH-wt-b"
tmux new-window -t $SESSION_NAME:3 -n analysis -c "$PROJECT_PATH-wt-analysis"

# 각 윈도우를 2개 pane으로 분할 (상: 에디터, 하: Claude)
for i in 0 1 2 3; do
    tmux split-window -t $SESSION_NAME:$i -v -p 40
    tmux send-keys -t $SESSION_NAME:$i.1 'claude' C-m
done

# 세션 attach
tmux attach-session -t $SESSION_NAME
```

---

## 서브에이전트 활용 전략

### 서브에이전트 개념

서브에이전트는 메인 에이전트가 생성하는 보조 에이전트로, 특정 탐색 작업을 병렬로 수행합니다.

### 기본 사용법

```
# 기본 프롬프트에 "use subagents" 추가
> 이 프로젝트의 아키텍처를 분석해줘. use subagents

# Claude는 다음과 같은 서브에이전트들을 생성:
# - Subagent 1: 엔트리 포인트 탐색
# - Subagent 2: 데이터베이스 스키마 분석
# - Subagent 3: API 엔드포인트 수집
# - Subagent 4: 의존성 그래프 생성
```

### 고급 서브에이전트 패턴

#### 패턴 1: 대규모 코드베이스 분석

```
> 이 레포지토리의 전체 구조를 분석해줘.
> 
> use subagents로 다음을 병렬 수행해:
> 1. /src 디렉토리의 모듈 구조 파악
> 2. /tests 디렉토리의 테스트 커버리지 분석  
> 3. package.json의 의존성 분석
> 4. 문서화 상태 평가 (README, JSDoc 등)
> 5. 보안 취약점 스캔 (npm audit)
>
> 각 서브에이전트의 결과를 종합해서
> 프로젝트 상태 리포트를 작성해줘.
```

#### 패턴 2: 다중 파일 리팩토링

```
> 다음 파일들을 동시에 리팩토링해줘. use subagents
>
> 파일 목록:
> - src/auth/login.js
> - src/auth/register.js
> - src/auth/password-reset.js
> - src/auth/email-verification.js
>
> 리팩토링 목표:
> 1. 공통 로직을 auth/common.js로 추출
> 2. 에러 처리 표준화
> 3. 검증 로직 통합
>
> 각 파일을 서브에이전트에 할당해서
> 동시에 작업한 뒤 결과를 통합해줘.
```

#### 패턴 3: 테스트 생성

```
> 다음 모듈들에 대한 테스트를 작성해줘. use subagents
>
> 모듈:
> - src/services/userService.js
> - src/services/orderService.js
> - src/services/paymentService.js
> - src/services/emailService.js
>
> 각 모듈당 서브에이전트 하나씩 할당해서:
> 1. 유닛 테스트 작성 (모든 public 메서드)
> 2. 모킹 설정
> 3. 엣지 케이스 커버
>
> 최소 커버리지: 85%
```

### 컨텍스트 윈도우 관리

```
> 이 대규모 프로젝트를 분석하되, 메인 에이전트의
> 컨텍스트 윈도우를 최대한 깨끗하게 유지해줘.
>
> use subagents로 다음 작업을 오프로드:
> 1. 각 마이크로서비스별 API 문서 생성
> 2. 데이터베이스 ERD 추출
> 3. 의존성 그래프 시각화
> 4. 보안 감사 리포트
>
> 서브에이전트가 작업을 완료하면,
> 요약만 메인 에이전트에 전달해줘.
```

### 권한 요청 자동화 (Opus 4.5 Hook)

```bash
# hooks/permission-request.js
// Opus 4.5를 사용해 권한 요청을 자동으로 검토하고 승인

module.exports = async (request) => {
    // Opus 4.5로 보안 스캔
    const analysis = await opus45.analyze(request, {
        checkFor: [
            'malicious_commands',
            'data_exfiltration',
            'destructive_operations',
            'credential_exposure'
        ]
    });
    
    // 안전한 요청은 자동 승인
    if (analysis.isSafe) {
        return { approved: true };
    }
    
    // 의심스러운 요청은 사용자에게 확인
    return {
        approved: false,
        reason: analysis.concerns,
        requiresManualReview: true
    };
};
```

#### Hook 설정

```bash
# claude.config.json
{
    "hooks": {
        "permissionRequest": {
            "enabled": true,
            "handler": "./hooks/permission-request.js",
            "model": "claude-opus-4-5",
            "autoApprove": [
                "file_read",
                "file_create",
                "bash_safe_commands"
            ]
        }
    }
}
```

---

## 데이터 분석 워크플로우

### BigQuery CLI 통합

#### 기본 설정

```bash
# BigQuery CLI 설치
brew install google-cloud-sdk

# 인증
gcloud auth login

# 프로젝트 설정
gcloud config set project my-project-id

# Claude Code에서 사용
claude

> BigQuery CLI를 사용해서 지난 7일간의 사용자 활동을 분석해줘.
>
> 데이터셋: analytics.user_events
> 
> 다음 메트릭 계산:
> 1. 일별 활성 사용자 수 (DAU)
> 2. 세션 평균 길이
> 3. 가장 인기 있는 기능 (상위 10개)
> 4. 전환율 (signup → first_purchase)
>
> 결과를 시각화해줘 (chart_display 사용)
```

#### BigQuery Skill 예시

````markdown
# skills/bigquery/SKILL.md

# BigQuery Analysis Skill

이 스킬은 BigQuery를 사용한 데이터 분석을 지원합니다.

## 사용 가능한 명령어

### 기본 쿼리 실행
```bash
bq query --use_legacy_sql=false '
SELECT
    DATE(timestamp) as date,
    COUNT(DISTINCT user_id) as dau
FROM `my-project.analytics.user_events`
WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
GROUP BY date
ORDER BY date
'
```

### 대용량 데이터 처리
```bash
# --destination_table로 결과 저장
bq query \
    --use_legacy_sql=false \
    --destination_table=my-project:analytics.user_summary \
    --replace \
    'SELECT ...'
```

### 데이터 내보내기
```bash
# CSV로 내보내기
bq extract \
    --destination_format=CSV \
    analytics.user_summary \
    gs://my-bucket/output.csv
```

## 분석 패턴

### 시계열 분석
```sql
WITH daily_metrics AS (
    SELECT
        DATE(timestamp) as date,
        COUNT(DISTINCT user_id) as users,
        COUNT(*) as events
    FROM `analytics.user_events`
    WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
    GROUP BY date
)
SELECT
    date,
    users,
    AVG(users) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as users_7day_avg
FROM daily_metrics
ORDER BY date
```

### 코호트 분석
```sql
WITH user_cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC(MIN(DATE(timestamp)), MONTH) as cohort_month
    FROM `analytics.user_events`
    GROUP BY user_id
)
SELECT
    cohort_month,
    DATE_DIFF(DATE_TRUNC(DATE(e.timestamp), MONTH), c.cohort_month, MONTH) as months_since_signup,
    COUNT(DISTINCT e.user_id) as active_users
FROM user_cohorts c
JOIN `analytics.user_events` e ON c.user_id = e.user_id
GROUP BY cohort_month, months_since_signup
ORDER BY cohort_month, months_since_signup
```

## 성능 최적화

1. **파티션 사용**: WHERE 절에 파티션 컬럼 포함
2. **집계 최적화**: 미리 집계된 테이블 활용
3. **비용 관리**: --dry_run으로 쿼리 비용 확인

## 시각화

분석 결과는 chart_display 도구로 시각화:

```javascript
// 예시
await chart_display({
    style: 'line',
    series: [{
        name: 'DAU',
        values: dailyActiveUsers
    }],
    xAxis: {
        data: dates,
        title: '날짜'
    },
    yAxis: {
        title: '사용자 수'
    },
    title: '일별 활성 사용자 (7일간)'
});
```
````

### 실전 분석 예시

```
User: 지난 30일간의 사용자 retention을 분석해줘.

Claude: BigQuery로 retention 분석을 실행하겠습니다.

[실행 중: bq query ...]

분석 결과:

코호트별 Retention Rate (30일):
- 2025-01 코호트: D1=45%, D7=28%, D30=15%
- 2025-12 코호트: D1=42%, D7=25%, D30=12%
- 2025-11 코호트: D1=40%, D7=22%, D30=10%

[chart_display로 시각화]

주요 인사이트:
1. D1 retention이 개선되고 있습니다 (40% → 45%)
2. D30 retention은 여전히 낮습니다 (10-15%)
3. D7-D30 구간의 이탈이 큽니다 (drop-off: ~50%)

권장사항:
1. 첫 주 이후 재참여 캠페인 강화
2. D7-D14 구간의 이탈 원인 분석
3. 핵심 기능 사용 유도 온보딩 개선

더 자세한 분석이 필요하면 말씀해주세요.
```

### 슬래시 커맨드로 정기 분석 자동화

~~~bash
# /daily-metrics 슬래시 커맨드 생성

cat > ~/.claude/commands/daily-metrics.md << 'EOF'
# Daily Metrics Report

BigQuery로 일일 핵심 메트릭을 조회하고 리포트를 생성합니다.

## 실행 쿼리

```sql
SELECT
    DATE(timestamp) as date,
    COUNT(DISTINCT user_id) as dau,
    COUNT(DISTINCT session_id) as sessions,
    COUNT(DISTINCT CASE WHEN event_name = 'signup' THEN user_id END) as signups,
    COUNT(DISTINCT CASE WHEN event_name = 'purchase' THEN user_id END) as purchases,
    SUM(CASE WHEN event_name = 'purchase' THEN revenue END) as revenue
FROM `analytics.user_events`
WHERE DATE(timestamp) = CURRENT_DATE() - 1
GROUP BY date
```

## 출력 형식

```
📊 Daily Metrics Report (YYYY-MM-DD)
================================
👥 DAU: X,XXX
📱 Sessions: X,XXX
✨ Signups: XXX
💰 Purchases: XXX
💵 Revenue: $X,XXX
```

## 알림 조건

다음 경우 경고:
- DAU < 전일 대비 10% 감소
- Revenue < 전일 대비 20% 감소
- Signups < 100
EOF
~~~

---

## 학습 도구로서의 Claude Code

### Explanatory 출력 스타일

```bash
# Claude Code 설정
claude config

# 출력 스타일 변경
/config output-style explanatory

# 또는
/config output-style learning
```

#### Explanatory 모드 예시

~~~
User: 이 React 컴포넌트를 최적화해줘.

Claude (Explanatory 모드):
이 컴포넌트를 최적화하겠습니다. 제가 하는 각 변경사항의 이유를 설명드리겠습니다.

1️⃣ useMemo 추가
━━━━━━━━━━━━━━━
현재 문제: sortedData가 매 렌더링마다 재계산됩니다.
왜 문제인가: data가 변경되지 않아도 불필요한 정렬이 발생합니다.
해결 방법: useMemo로 data가 실제로 변경될 때만 재계산합니다.

```javascript
const sortedData = useMemo(() => {
    return [...data].sort((a, b) => a.name.localeCompare(b.name));
}, [data]);
```

성능 개선: 100ms → 0.1ms (렌더링당)

2️⃣ useCallback으로 함수 메모이제이션
━━━━━━━━━━━━━━━━━━━━━━━━━
현재 문제: handleClick이 매번 새로 생성되어 자식 컴포넌트가 불필요하게 리렌더링됩니다.
왜 문제인가: React.memo로 감싼 자식 컴포넌트도 props가 변경되었다고 판단합니다.
해결 방법: useCallback으로 함수를 메모이제이션합니다.

```javascript
const handleClick = useCallback((id) => {
    setSelected(id);
    analytics.track('item_clicked', { id });
}, []);  // 의존성이 없으므로 한 번만 생성
```

렌더링 횟수 감소: 부모 렌더링 10회 → 자식 리렌더링 0회

3️⃣ React.memo로 컴포넌트 메모이제이션
━━━━━━━━━━━━━━━━━━━━━━━━━━━
현재 문제: ListItem 컴포넌트가 다른 아이템의 변경으로도 리렌더링됩니다.
왜 문제인가: 100개 아이템 중 1개만 변경되어도 100개 모두 리렌더링됩니다.
해결 방법: React.memo로 props가 실제로 변경된 컴포넌트만 리렌더링합니다.

```javascript
const ListItem = React.memo(({ item, onClick }) => {
    return <div onClick={() => onClick(item.id)}>{item.name}</div>;
});
```

리렌더링 횟수: 100회 → 1회 (실제 변경된 아이템만)

📊 전체 성능 개선 요약
━━━━━━━━━━━━━━━━━
- 초기 렌더링: 450ms → 380ms (15% 개선)
- 업데이트 렌더링: 120ms → 8ms (93% 개선)
- 메모리 사용량: 변화 없음

이해가 안 되는 부분이 있으면 질문해주세요!
~~~

### HTML 프레젠테이션 생성

```
User: React Hooks의 작동 원리를 설명하는 프레젠테이션을 만들어줘.

Claude: 인터랙티브한 HTML 프레젠테이션을 만들겠습니다.

[파일 생성: react-hooks-presentation.html]

프레젠테이션에 다음 슬라이드가 포함됩니다:
1. React Hooks 소개
2. useState의 내부 동작
3. useEffect의 실행 타이밍
4. 클로저와 Hooks
5. 커스텀 Hooks 패턴
6. 실습 예제

브라우저에서 열어서 확인해보세요!
```

#### 프레젠테이션 예시 코드

```html
<!DOCTYPE html>
<html>
<head>
    <title>React Hooks 완전 정복</title>
    <style>
        body { 
            font-family: 'SF Pro', system-ui; 
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 0;
        }
        .slide {
            display: none;
            height: 100vh;
            padding: 60px;
            box-sizing: border-box;
        }
        .slide.active { display: flex; flex-direction: column; justify-content: center; }
        h1 { font-size: 72px; color: white; margin: 0; }
        h2 { font-size: 48px; color: #f0f0f0; }
        .code-block {
            background: #1e1e1e;
            border-radius: 12px;
            padding: 24px;
            margin: 24px 0;
            font-family: 'JetBrains Mono', monospace;
            font-size: 18px;
            color: #d4d4d4;
        }
        .navigation {
            position: fixed;
            bottom: 40px;
            right: 40px;
            display: flex;
            gap: 16px;
        }
        button {
            padding: 12px 24px;
            font-size: 18px;
            border: none;
            border-radius: 8px;
            background: white;
            cursor: pointer;
            transition: transform 0.2s;
        }
        button:hover { transform: scale(1.05); }
        .highlight { color: #61dafb; font-weight: bold; }
        .diagram {
            background: white;
            border-radius: 12px;
            padding: 32px;
            margin: 24px 0;
        }
    </style>
</head>
<body>
    <div class="slide active">
        <h1>React Hooks 완전 정복</h1>
        <p style="font-size: 24px; color: #f0f0f0;">함수형 컴포넌트의 상태 관리</p>
    </div>

    <div class="slide">
        <h2>useState는 어떻게 작동할까?</h2>
        <div class="code-block">
let componentState = [];
let currentIndex = 0;

function useState(initialValue) {
    const index = currentIndex;
    
    // 첫 렌더링: 초기값 저장
    if (componentState[index] === undefined) {
        componentState[index] = initialValue;
    }
    
    const setState = (newValue) => {
        componentState[index] = newValue;
        reRender();  // 컴포넌트 리렌더링
    };
    
    currentIndex++;
    return [componentState[index], setState];
}
        </div>
        <p style="color: white; font-size: 20px;">
            💡 Hooks는 <span class="highlight">호출 순서</span>에 의존합니다!
        </p>
    </div>

    <div class="slide">
        <h2>useEffect 실행 타이밍</h2>
        <div class="diagram">
            <svg width="800" height="300">
                <rect x="50" y="50" width="150" height="80" fill="#61dafb" rx="8"/>
                <text x="125" y="95" text-anchor="middle" fill="white" font-size="18">Render</text>
                
                <rect x="250" y="50" width="150" height="80" fill="#764ba2" rx="8"/>
                <text x="325" y="95" text-anchor="middle" fill="white" font-size="18">DOM 업데이트</text>
                
                <rect x="450" y="50" width="150" height="80" fill="#667eea" rx="8"/>
                <text x="525" y="95" text-anchor="middle" fill="white" font-size="18">화면 그리기</text>
                
                <rect x="650" y="50" width="150" height="80" fill="#f06292" rx="8"/>
                <text x="725" y="95" text-anchor="middle" fill="white" font-size="18">useEffect</text>
                
                <defs>
                    <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="5" refY="3" orient="auto">
                        <polygon points="0 0, 10 3, 0 6" fill="#333"/>
                    </marker>
                </defs>
                <line x1="200" y1="90" x2="250" y2="90" stroke="#333" stroke-width="3" marker-end="url(#arrowhead)"/>
                <line x1="400" y1="90" x2="450" y2="90" stroke="#333" stroke-width="3" marker-end="url(#arrowhead)"/>
                <line x1="600" y1="90" x2="650" y2="90" stroke="#333" stroke-width="3" marker-end="url(#arrowhead)"/>
            </svg>
        </div>
        <p style="color: white; font-size: 20px;">
            useEffect는 <span class="highlight">비동기</span>로 실행됩니다!
        </p>
    </div>

    <div class="navigation">
        <button onclick="prevSlide()">← 이전</button>
        <button onclick="nextSlide()">다음 →</button>
    </div>

    <script>
        let currentSlide = 0;
        const slides = document.querySelectorAll('.slide');

        function showSlide(index) {
            slides.forEach(s => s.classList.remove('active'));
            slides[index].classList.add('active');
        }

        function nextSlide() {
            currentSlide = (currentSlide + 1) % slides.length;
            showSlide(currentSlide);
        }

        function prevSlide() {
            currentSlide = (currentSlide - 1 + slides.length) % slides.length;
            showSlide(currentSlide);
        }

        // 키보드 네비게이션
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowRight') nextSlide();
            if (e.key === 'ArrowLeft') prevSlide();
        });
    </script>
</body>
</html>
```

### ASCII 다이어그램

```
User: gRPC 프로토콜의 통신 흐름을 ASCII 다이어그램으로 그려줘.

Claude: gRPC 통신 흐름을 시각화하겠습니다.

┌─────────────┐                                    ┌─────────────┐
│   Client    │                                    │   Server    │
│  (gRPC)     │                                    │  (gRPC)     │
└─────┬───────┘                                    └──────┬──────┘
      │                                                   │
      │  1. HTTP/2 Connection Establishment               │
      │ ─────────────────────────────────────────────────>│
      │                                                   │
      │  2. Client Send Metadata (Headers)                │
      │ ─────────────────────────────────────────────────>│
      │     - content-type: application/grpc              │
      │     - grpc-encoding: gzip                         │
      │     - authorization: Bearer token                 │
      │                                                   │
      │  3. Client Send Message (Protobuf)                │
      │ ─────────────────────────────────────────────────>│
      │     ┌──────────────────┐                         │
      │     │  Length Prefix   │ (4 bytes)               │
      │     ├──────────────────┤                         │
      │     │  Protobuf Data   │                         │
      │     └──────────────────┘                         │
      │                                                   │
      │                                      4. Server Processing
      │                                         ┌─────────┴─────────┐
      │                                         │  Deserialize      │
      │                                         │  Business Logic   │
      │                                         │  Serialize        │
      │                                         └─────────┬─────────┘
      │                                                   │
      │  5. Server Send Response                          │
      │ <─────────────────────────────────────────────────│
      │     ┌──────────────────┐                         │
      │     │  Length Prefix   │                         │
      │     ├──────────────────┤                         │
      │     │  Protobuf Data   │                         │
      │     └──────────────────┘                         │
      │                                                   │
      │  6. Server Send Trailer (Status)                  │
      │ <─────────────────────────────────────────────────│
      │     - grpc-status: 0 (OK)                         │
      │     - grpc-message: ""                            │
      │                                                   │
      │  7. Connection Close (or Keep-Alive)              │
      │ <────────────────────────────────────────────────>│
      │                                                   │
      ▼                                                   ▼

📝 주요 특징:
━━━━━━━━━━━━
• HTTP/2 기반: 멀티플렉싱, 헤더 압축
• Protobuf 직렬화: 빠르고 컴팩트
• 양방향 스트리밍: 클라이언트/서버 모두 가능
• Trailer: 최종 상태 전송
```

---

## 버그 수정 자동화

### Slack MCP 통합

```bash
# Slack MCP 서버 활성화
claude mcp enable slack

# Slack 워크스페이스 연결
claude mcp configure slack --workspace my-team
```

#### 버그 수정 워크플로우

~~~
User: [Slack 버그 쓰레드 URL을 붙여넣기]

Claude: Slack 쓰레드를 분석하겠습니다...

📩 버그 리포트 요약
━━━━━━━━━━━━━━━━
제목: 장바구니 총액 계산 오류
보고자: @john (Engineering)
우선순위: P1 - Critical
영향: 전체 사용자

재현 단계:
1. 상품 3개 장바구니에 추가
2. 쿠폰 적용
3. 한 상품 삭제
4. 총액이 업데이트되지 않음

스택 트레이스:
```
TypeError: Cannot read property 'price' of undefined
    at calculateTotal (cart.js:156)
    at applyDiscount (cart.js:89)
```

관련 파일 분석 중...

[파일 분석 완료]

🔍 근본 원인
━━━━━━━━━━━
cart.js:156에서 삭제된 상품을 참조하려고 시도합니다.
상품 삭제 후 cartItems 배열이 업데이트되지만,
calculateTotal 함수는 이전 참조를 유지하고 있습니다.

🛠️ 수정 계획
━━━━━━━━━━━
1. cartItems 상태 업데이트 시 즉시 재계산
2. 방어적 프로그래밍: null 체크 추가
3. 단위 테스트 추가

수정을 진행할까요? (yes/no)

User: yes

Claude: 수정 중...

[코드 수정 완료]

✅ 수정 완료
━━━━━━━━━━
변경 파일:
- src/cart/cart.js (수정)
- src/cart/cart.test.js (추가)

테스트 실행:
✓ 상품 삭제 후 총액 재계산
✓ 쿠폰 적용 후 총액 재계산
✓ null 상품 방어 처리
✓ 엣지 케이스: 빈 장바구니

모든 테스트 통과 (12/12)

PR 생성할까요? (yes/no)

User: yes

Claude: PR 생성 완료!
🔗 https://github.com/my-team/my-app/pull/1234

PR 내용:
- 제목: Fix cart total calculation after item removal
- 설명: Slack 버그 리포트 링크 포함
- 리뷰어: @john, @sarah 자동 할당
- 라벨: bug, p1-critical 추가

Slack 쓰레드에 PR 링크를 댓글로 남겼습니다.
~~~

### CI 실패 자동 수정

```
User: CI가 실패했어. 자동으로 고쳐줘.

Claude: GitHub Actions CI 로그를 확인하겠습니다...

[CI 로그 분석]

❌ 실패한 테스트
━━━━━━━━━━━━━━━
Test Suite: UserService
Test: should validate email format
Error: Expected validation to fail for "invalid@"
       but validation passed

근본 원인:
이메일 검증 정규식이 불완전합니다.
현재: /^.+@.+$/
문제: "@" 뒤에 도메인이 없어도 통과

수정 중...

[코드 수정]

✅ 수정 완료
━━━━━━━━━━
변경: src/validators/email.js
- 정규식 업데이트: /^[^@]+@[^@]+\.[^@]+$/
- 추가 테스트 케이스 작성

로컬 테스트 실행:
✓ All 45 tests passed

커밋 및 푸시 중...

✅ CI 재실행 트리거됨
━━━━━━━━━━━━━━━━━
🔗 https://github.com/my-team/my-app/actions/runs/12345

대기 중... (30초)

✅ CI 통과!
━━━━━━━━━
모든 테스트가 통과했습니다.
```

### Docker 로그 트러블슈팅

~~~
User: Docker 컨테이너가 계속 재시작해. 로그를 확인해줘.

Claude: Docker 로그를 분석하겠습니다...

$ docker logs my-app-container --tail=100

[로그 분석 중]

🔴 치명적 에러 발견
━━━━━━━━━━━━━━━━
Error: ECONNREFUSED 127.0.0.1:5432
    at TCPConnectWrap.afterConnect
    
PostgreSQL 연결 실패

🔍 원인 분석
━━━━━━━━━━
1. 컨테이너가 localhost(127.0.0.1)를 사용 중
2. Docker 네트워크에서는 서비스 이름 사용해야 함
3. docker-compose.yml의 DB_HOST 설정 문제

📂 docker-compose.yml 확인:
```yaml
services:
  app:
    environment:
      DB_HOST: localhost  # ❌ 문제!
      DB_PORT: 5432
  db:
    image: postgres:15
```

🛠️ 수정 방안
━━━━━━━━━━
DB_HOST를 'db'(서비스 이름)로 변경

수정할까요? (yes/no)

User: yes

Claude: 수정 중...

[docker-compose.yml 수정]

```yaml
services:
  app:
    environment:
      DB_HOST: db  # ✅ 수정됨
      DB_PORT: 5432
  db:
    image: postgres:15
```

컨테이너 재시작 중...

$ docker-compose down
$ docker-compose up -d

✅ 해결 완료
━━━━━━━━━━
✓ 컨테이너 정상 실행 중
✓ PostgreSQL 연결 성공
✓ 헬스체크 통과

로그 확인:
[2026-02-01 10:30:45] INFO: Database connected
[2026-02-01 10:30:46] INFO: Server listening on port 3000
~~~

---

## 실전 팁 총정리

### 스킬 및 슬래시 커맨드 전략

#### 반복 작업 자동화

````markdown
# 규칙: 하루에 한 번 이상 반복하는 작업 → 스킬/커맨드화

예시:
- /techdebt: 중복 코드 찾기
- /sync-context: 7일치 컨텍스트 수집 (Slack, GDrive, GitHub)
- /daily-standup: 어제/오늘 작업 요약
- /pr-checklist: PR 제출 전 체크리스트
````

#### /techdebt 커맨드 예시

~~~bash
# ~/.claude/commands/techdebt.md

# Technical Debt Scanner

세션 종료 시 실행하여 기술 부채를 찾아냅니다.

## 검사 항목

1. **코드 중복**
   - 동일/유사 함수 3개 이상
   - 복사-붙여넣기 패턴

2. **복잡도**
   - 순환 복잡도 > 10
   - 함수 길이 > 50줄

3. **테스트 누락**
   - public 함수에 테스트 없음
   - 커버리지 < 80%

4. **문서화**
   - JSDoc 누락
   - README 오래됨

## 실행 로직

```bash
# 중복 코드 탐지
npx jscpd src/ --min-lines 5 --min-tokens 50

# 복잡도 분석
npx complexity-report src/

# 커버리지 확인
npm test -- --coverage
```

## 리포트 형식

````markdown
## 기술 부채 리포트 (YYYY-MM-DD)

### 🔴 높은 우선순위
- [ ] src/auth/login.js:45 - 중복 코드 (3개 파일)
- [ ] src/utils/parser.js:123 - 복잡도 15

### 🟡 중간 우선순위
- [ ] src/services/email.js - 테스트 누락
- [ ] src/components/Dashboard.js - JSDoc 없음

### 추천 작업
1. 인증 로직 공통 함수로 추출
2. parser 함수 리팩토링
3. email service 단위 테스트 추가
````
~~~

#### /sync-context 커맨드

```bash
# Slack, Google Drive, Asana, GitHub에서
# 지난 7일간의 컨텍스트를 한 번에 수집

#!/bin/bash

echo "🔄 Syncing context from all sources..."

# Slack 메시지
claude mcp call slack.search --query="in:engineering after:7days" > /tmp/slack.txt

# Google Drive 문서
claude mcp call gdrive.recent --days=7 > /tmp/gdrive.txt

# GitHub PR & Issues
gh pr list --limit 50 --json title,url,createdAt > /tmp/github.txt

# Asana 태스크
curl "https://app.asana.com/api/1.0/tasks?completed_since=now-7d" \
    -H "Authorization: Bearer $ASANA_TOKEN" > /tmp/asana.txt

# 요약 생성
claude << EOF
다음 소스에서 수집한 7일치 컨텍스트를 요약해줘:

**Slack**: $(cat /tmp/slack.txt)
**Google Drive**: $(cat /tmp/gdrive.txt)  
**GitHub**: $(cat /tmp/github.txt)
**Asana**: $(cat /tmp/asana.txt)

다음 형식으로 요약:
1. 주요 결정사항
2. 진행 중인 작업
3. 블로커
4. 다음 주 우선순위

EOF
```

### 고급 Git 워크플로우

#### 자동 브랜치 생성 + Worktree

```bash
# ~/bin/new-feature.sh

#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: new-feature <feature-name>"
    exit 1
fi

FEATURE_NAME=$1
BRANCH_NAME="feature/$FEATURE_NAME"
WORKTREE_PATH="../$(basename $(pwd))-wt-$FEATURE_NAME"

# 1. main에서 최신 코드 가져오기
git fetch origin main

# 2. 새 브랜치 생성
git branch $BRANCH_NAME origin/main

# 3. Worktree 생성
git worktree add $WORKTREE_PATH $BRANCH_NAME

# 4. Worktree로 이동
cd $WORKTREE_PATH

# 5. Claude 실행
echo "✅ Worktree created: $WORKTREE_PATH"
echo "🚀 Starting Claude..."
claude
```

사용법:
```bash
$ new-feature user-authentication
✅ Worktree created: ../my-app-wt-user-authentication
🚀 Starting Claude...
```

### 최적의 모델 선택 전략

````markdown
# 작업별 최적 모델 가이드

## Claude Opus 4.5
**사용 시기**:
- 복잡한 아키텍처 설계
- 장기간 지속되는 작업 (>30분)
- 깊은 코드 이해 필요
- 보안 감사

**예시**:
> 전체 마이크로서비스 아키텍처를 설계해줘.
> 각 서비스의 책임, API 계약, 데이터 흐름을 정의하고
> 확장성과 장애 복구 전략을 포함해.

## Claude Sonnet 4.5
**사용 시기**:
- 일반적인 개발 작업
- 빠른 프로토타이핑
- 코드 리뷰
- 문서 작성

**예시**:
> REST API 엔드포인트를 구현해줘.
> 사용자 CRUD와 JWT 인증을 포함해.

## Claude Haiku 4.5
**사용 시기**:
- 간단한 코드 수정
- 빠른 질문 답변
- 반복적인 작업
- 비용 절감 필요

**예시**:
> 이 함수에 에러 처리를 추가해줘.
````

### 프로덕션 체크리스트 자동화

```bash
# /pre-deploy 슬래시 커맨드

#!/bin/bash

echo "🚀 Pre-Deployment Checklist"
echo "============================="

PASSED=0
FAILED=0

# 1. 테스트
echo -n "1. Running tests... "
if npm test > /dev/null 2>&1; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    ((FAILED++))
fi

# 2. 린트
echo -n "2. Linting... "
if npm run lint > /dev/null 2>&1; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    ((FAILED++))
fi

# 3. 타입 체크
echo -n "3. Type checking... "
if npm run type-check > /dev/null 2>&1; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    ((FAILED++))
fi

# 4. 보안 감사
echo -n "4. Security audit... "
if npm audit --production > /dev/null 2>&1; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    ((FAILED++))
fi

# 5. 빌드
echo -n "5. Building... "
if npm run build > /dev/null 2>&1; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    ((FAILED++))
fi

# 6. 환경 변수 확인
echo -n "6. Environment variables... "
REQUIRED_VARS=("DATABASE_URL" "API_KEY" "JWT_SECRET")
MISSING_VARS=()

for var in "${REQUIRED_VARS[@]}"; do
    if [ -z "${!var}" ]; then
        MISSING_VARS+=($var)
    fi
done

if [ ${#MISSING_VARS[@]} -eq 0 ]; then
    echo "✅"
    ((PASSED++))
else
    echo "❌"
    echo "   Missing: ${MISSING_VARS[*]}"
    ((FAILED++))
fi

# 결과
echo ""
echo "============================="
echo "Passed: $PASSED / $((PASSED + FAILED))"

if [ $FAILED -eq 0 ]; then
    echo "✅ Ready to deploy!"
    exit 0
else
    echo "❌ Fix issues before deploying"
    exit 1
fi
```

### 성능 모니터링 대시보드

```bash
# /perf-dashboard 슬래시 커맨드

# Lighthouse CI 실행 + 결과 시각화

#!/bin/bash

echo "📊 Performance Dashboard"
echo "======================="

# Lighthouse 실행
lhci autorun --config=.lighthouserc.json

# 결과 파싱
claude << 'EOF'
Lighthouse 결과를 분석하고 다음 형식으로 리포트를 생성해줘:

1. 성능 점수 추이 (지난 7일)
2. Core Web Vitals
   - LCP (Largest Contentful Paint)
   - FID (First Input Delay)
   - CLS (Cumulative Layout Shift)
3. 개선이 필요한 항목 (우선순위별)
4. 권장 조치사항

chart_display를 사용해서 추이를 시각화해줘.
EOF
```

---

## 맺음말

Boris Cherny와 Claude Code 팀이 공유한 이 생산성 팁들은 단순한 "사용법"을 넘어, AI 시대 개발자의 새로운 워크플로우를 제시합니다.

### 핵심 원칙 요약

1. **병렬성**: 여러 작업을 동시에 진행하여 대기 시간을 작업 시간으로 전환
2. **자기 학습**: CLAUDE.md와 스킬로 실수를 제도화하여 점진적 개선
3. **협력**: Claude를 단순 실행자가 아닌 사고 파트너로 활용
4. **자동화**: 반복 작업을 스킬/커맨드로 자동화
5. **최적화**: 환경을 철저히 최적화하여 전환 비용 최소화

### 시작하기

오늘 당장 시도해볼 수 있는 것들:

1. **Worktree 2개 더 띄우기**
   ```bash
   git worktree add ../my-app-wt-a feature/new-feature
   git worktree add ../my-app-wt-b feature/another-feature
   ```

2. **CLAUDE.md 만들기**
   ```bash
   touch CLAUDE.md
   echo "# Project Guidelines" > CLAUDE.md
   ```

3. **플랜 모드 사용하기**
   ```
   다음 작업을 플랜 모드로 시작해줘: [작업 내용]
   ```

4. **첫 슬래시 커맨드 만들기**
   ```bash
   mkdir -p ~/.claude/commands
   cat > ~/.claude/commands/hello.md << 'EOF'
   # Hello Command
   Claude에게 인사하는 테스트 커맨드입니다.
   EOF
   ```

### 지속적 개선

이 가이드는 시작점입니다. 여러분의 워크플로우에 맞게 조정하고, 팀과 공유하며, 지속적으로 개선하세요.

기억하세요: 
> "완벽한 도구는 없습니다. 하지만 도구를 완벽하게 사용하는 방법은 있습니다."

---

## 참고 자료

- Claude Code 공식 문서: https://code.claude.com/docs
- Boris Cherny Threads: https://www.threads.com/@boris_cherny
- Git Worktree 가이드: https://git-scm.com/docs/git-worktree
- BigQuery CLI: https://cloud.google.com/bigquery/docs/bq-command-line-tool

---

**작성**: 2026-02-01  
**작성자**: Claude (Anthropic)  
**기반**: Boris Cherny와 Claude Code 팀의 생산성 가이드
