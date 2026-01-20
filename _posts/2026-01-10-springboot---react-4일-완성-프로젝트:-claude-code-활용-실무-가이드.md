---
title: "SpringBoot React 4일 완성 프로젝트: Claude Code 활용 실무 가이드"
date: 2026-01-10 20:30:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,   claude-code,  vibe-coding,  Developer,  Guide,  Claude.write]
---


## 4일 만에 프로덕션 시스템 구축하기

---
## 프롤로그: 바이브 코딩이 실제로 작동하는 방법

"바이브 코딩"이라는 용어를 처음 들었을 때 많은 개발자들은 회의적이다. 코드를 제대로 이해하지도 못한 채 AI가 생성한 것을 무작정 복사-붙여넣기하는 것처럼 들리기 때문이다. 하지만 [Obie Fernandez가 Nexus 프로젝트](https://obie.medium.com/what-used-to-take-months-now-takes-days-cc8883cc21e9)에서 보여준 것은 전혀 다른 차원이다. 4일 만에 13,000줄의 프로덕션 준비 시스템을 만든 것은 마법이 아니라 **체계적인 접근법, 명확한 원칙, 그리고 AI를 전략적으로 활용한 결과**다.

이 가이드는 SpringBoot 백엔드와 React 프론트엔드를 가진 실전 프로젝트를 4일 만에 구축하는 구체적인 방법을 제시한다. 팀 리더(PL) 포함 개발자 2명, PM과 QA/QC가 별도로 있는 전형적인 스타트업이나 기업 환경을 가정한다. XP(Extreme Programming) 방법론을 기반으로 하되, AI 시대에 맞게 재해석했다.

이것은 이론서가 아니다. **실전 경험에서 나온 실용적인 가이드**다. 당신이 월요일 아침에 프로젝트를 시작해서 목요일 저녁에 배포할 수 있도록 일별, 시간별로 구체적인 액션을 제시한다.

---

## 제1장: 준비 - 바이브 코딩을 위한 마인드셋과 도구

### 1.1 바이브 코딩의 핵심 원칙

바이브 코딩이 성공하려면 세 가지 핵심 원칙을 이해해야 한다.

**첫째, AI는 속도를 제공하고 인간은 방향을 제공한다.** Claude Code는 당신이 설명한 것을 빠르게 구현할 수 있지만, 무엇을 만들어야 하는지, 왜 만들어야 하는지, 어떤 트레이드오프를 선택해야 하는지는 여전히 당신의 판단이다. Fernandez가 강조했듯이, 그의 아키텍처 판단, 도메인 지식, 제품 직관이 모든 단계에서 필수적이었다. Claude Code는 Nexus를 혼자 만들 수 없었다.

**둘째, TDD는 AI 시대의 안전망이다.** 수천 줄의 코드가 몇 분 만에 생성될 때, 그 코드가 의도한 대로 작동하는지 어떻게 확신할 수 있는가? Fernandez의 답은 명확했다: 테스트 주도 개발. "테스트는 두려움을 해소한다." 테스트가 없으면 AI가 생성한 코드는 작동하는 것처럼 보이는 불안정한 카드 집이 된다. 테스트가 있으면 자신감을 가지고 리팩터링하고, 기능을 추가하고, 빠르게 반복할 수 있다.

**셋째, "문제에 대해 생각하기" 모드에 머물러라.** 전통적인 개발에서는 시간의 상당 부분을 기계적으로 코드를 타이핑하는 데 쓴다. AI 보조 개발에서는 거의 전체 시간을 문제 해결, 아키텍처 설계, 비즈니스 로직 이해에 쓸 수 있다. 키보드를 두드리는 대신 대화한다. 구글을 검색하는 대신 Claude에게 묻는다. 이 패러다임 전환을 받아들이면 생산성이 폭발적으로 증가한다.

### 1.2 필수 도구 설정

프로젝트를 시작하기 전에 개발 환경을 제대로 설정해야 한다. 잘못된 설정은 나중에 시간을 낭비하게 만든다.

**Claude Code 설치 및 설정**

Claude Code는 터미널 기반 AI 코딩 어시스턴트다. 설치는 간단하지만 제대로 설정하는 것이 중요하다. 공식 문서를 따라 설치한 후, 프로젝트 루트에서 `.claude` 디렉토리를 만들고 프로젝트 특화 설정을 추가한다.

프로젝트 컨텍스트 파일을 만들어라. `.claude/context.md`에 프로젝트의 기술 스택, 아키텍처 결정, 코딩 컨벤션을 문서화한다. Claude Code는 매번 이 파일을 읽고 맥락을 이해한다. 예를 들어:

```markdown
# 프로젝트: E-Commerce Platform MVP

## 기술 스택
- Backend: Spring Boot 3.2, Java 17
- Frontend: React 18, TypeScript, Vite
- Database: PostgreSQL 15 + pgvector
- Cache: Redis 7
- Message Queue: Kafka 3.6
- Container: Docker, Docker Compose
- CI/CD: GitLab Runner

## 아키텍처 원칙
- 헥사고날 아키텍처 (포트-어댑터 패턴)
- API-First 설계 (OpenAPI 3.0 스펙 우선)
- 테스트 피라미드 준수 (Unit > Integration > E2E)
- 12-Factor App 원칙

## 코딩 컨벤션
- Java: Google Java Style Guide
- TypeScript: Airbnb Style Guide
- Commit: Conventional Commits
- Branch: GitFlow (feature/, hotfix/, release/)
```

이렇게 하면 Claude가 일관된 스타일로 코드를 생성한다.

**Git 훅 설정**

Fernandez가 Claude Code 세션 트랜스크립트를 자동으로 캡처한 것처럼, 우리도 개발 과정을 자동으로 기록할 수 있다. `.git/hooks/post-commit`에 간단한 스크립트를 추가하여 각 커밋 후 Claude Code 세션 요약을 생성한다. 이것은 나중에 결정 사항을 추적하는 데 도움이 된다.

**개발 환경 컨테이너화**

Docker Compose로 로컬 개발 환경을 컨테이너화한다. PostgreSQL, Redis, Kafka를 한 번에 띄울 수 있어야 한다. `docker-compose.dev.yml`:

```yaml
version: '3.8'
services:
  postgres:
    image: pgvector/pgvector:pg15
    ports: ["5432:5432"]
    environment:
      POSTGRES_DB: ecommerce_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
  
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    ports: ["9092:9092"]
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
  
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
```

이제 `docker-compose -f docker-compose.dev.yml up -d`로 전체 인프라가 30초 만에 준비된다.

### 1.3 팀 워크플로 설정

2명의 개발자가 효율적으로 협업하려면 명확한 워크플로가 필요하다.

**역할 분담의 유연성**

전통적인 프론트엔드/백엔드 분리는 AI 시대에 덜 의미가 있다. Claude Code를 사용하면 백엔드 개발자도 React 컴포넌트를 만들 수 있고, 프론트엔드 개발자도 Spring Boot 엔드포인트를 추가할 수 있다. 대신 **기능 단위로 작업을 분할**한다. 한 명은 "사용자 인증" 전체를 맡고, 다른 한 명은 "상품 카탈로그" 전체를 맡는 식이다.

PL은 아키텍처 결정과 인터페이스 정의에 집중한다. OpenAPI 스펙을 먼저 정의하고, 두 개발자가 독립적으로 구현할 수 있게 한다. 하루에 두 번(오전 시작 시, 오후 중간) 15분 스탠드업으로 진행 상황을 동기화한다.

**페어 프로그래밍과 AI**

XP의 핵심인 페어 프로그래밍을 AI 시대에 맞게 재해석한다. 두 개발자가 같은 화면을 보며 한 명이 타이핑하는 대신, **한 명은 Claude Code와 대화하고 다른 한 명은 실시간으로 생성된 코드를 리뷰**한다. 이것은 전통적인 페어 프로그래밍보다 훨씬 빠르면서도 품질을 유지한다. 특히 복잡한 비즈니스 로직이나 중요한 아키텍처 결정이 필요할 때 이 방식을 사용한다.

**지속적 통합 설정**

GitLab Runner를 사용한 CI 파이프라인을 프로젝트 시작과 동시에 설정한다. `.gitlab-ci.yml`:

```yaml
stages:
  - test
  - build
  - deploy

backend-test:
  stage: test
  image: openjdk:17
  script:
    - cd backend
    - ./gradlew test
  only:
    - merge_requests
    - main

frontend-test:
  stage: test
  image: node:20
  script:
    - cd frontend
    - npm ci
    - npm run test
  only:
    - merge_requests
    - main

build:
  stage: build
  script:
    - docker-compose build
  only:
    - main

deploy-dev:
  stage: deploy
  script:
    - docker-compose -f docker-compose.prod.yml up -d
  only:
    - main
  environment:
    name: development
```

이제 main 브랜치에 푸시하면 자동으로 테스트 → 빌드 → 배포가 이루어진다. Fernandez가 강조했듯이, `needs: test`는 매우 중요하다. 테스트가 실패하면 배포가 일어나지 않는다.

### 1.4 첫 번째 세션: Claude Code와 친해지기

프로젝트를 시작하기 전에 팀 전체가 Claude Code에 익숙해져야 한다. 30분에서 1시간 정도 간단한 연습 세션을 갖는다.

**효과적인 프롬프팅 패턴**

Claude Code와 대화하는 것은 주니어 개발자에게 설명하는 것과 비슷하지만, 훨씬 더 빠르게 이해하고 구현한다. 효과적인 패턴을 익혀라:

**컨텍스트를 먼저 제공한다.** "사용자 인증 기능을 만들어줘"보다 "Spring Security와 JWT를 사용한 사용자 인증 API를 만들어줘. 로그인 시 액세스 토큰(15분)과 리프레시 토큰(7일)을 발급하고, 리프레시 토큰으로 액세스 토큰을 갱신할 수 있어야 해. PostgreSQL에 사용자 정보를 저장하고 비밀번호는 BCrypt로 해싱해."

**단계적으로 진행한다.** 한 번에 모든 것을 요구하지 말고, TDD 사이클을 따라 단계적으로 진행한다. "먼저 UserAuthenticationService의 인터페이스를 정의하고 테스트를 작성해줘" → 테스트가 실패하는 것을 확인 → "이제 구현을 추가해줘" → 테스트가 통과하는 것을 확인 → "JWT 토큰 생성 부분을 별도 유틸리티 클래스로 리팩터링해줘"

**검증 가능한 요구사항을 제시한다.** Fernandez가 강조했듯이, 쉽게 검증할 수 있는 작업을 위임한다. "성능이 좋은 캐싱"보다 "Redis를 사용해서 상품 목록 조회 API를 캐싱해줘. TTL은 5분, 캐시 키는 'products:list:{page}:{size}' 형식으로."

**실패를 포함해서 생각한다.** "로그인 API를 만들어줘"보다 "로그인 API를 만들어줘. 이메일이 없으면 404, 비밀번호가 틀리면 401, 계정이 잠겼으면 403을 반환해야 해. 각 케이스에 대한 테스트도 작성해줘."

**연습 태스크: TODO API**

간단한 TODO API를 Claude Code로 만들어보며 워크플로를 연습한다. Spring Boot로 CRUD 엔드포인트를 만들고, JPA로 PostgreSQL에 저장하고, 통합 테스트를 작성한다. 이 과정에서 Claude Code가 어떻게 작동하는지, 어떤 종류의 코드를 생성하는지, 어디서 검토가 필요한지 파악한다.

15-20분이면 충분하다. 완벽할 필요는 없다. 목표는 팀이 도구에 익숙해지는 것이다.

---

## 제2장: Day 0 - 설계와 준비 (프로젝트 시작 전날)

### 2.1 요구사항 정리와 우선순위

PM과 함께 앉아서 요구사항을 정리한다. 하지만 전통적인 상세 요구사항 문서를 만들지 않는다. 대신 **사용자 스토리와 인수 기준**에 집중한다.

예를 들어, E-Commerce 플랫폼이라면:

**핵심 스토리 (Must Have - Day 1-2):**
- 사용자는 이메일과 비밀번호로 회원가입/로그인할 수 있어야 한다
- 사용자는 상품 목록을 검색하고 필터링할 수 있어야 한다
- 사용자는 상품을 장바구니에 담을 수 있어야 한다
- 사용자는 장바구니의 상품을 주문할 수 있어야 한다

**중요 스토리 (Should Have - Day 3):**
- 관리자는 상품을 등록/수정/삭제할 수 있어야 한다
- 사용자는 주문 내역을 조회할 수 있어야 한다
- 재고가 부족하면 주문이 실패해야 한다

**추가 스토리 (Could Have - Day 4):**
- 사용자는 상품을 즐겨찾기할 수 있어야 한다
- 사용자는 상품에 리뷰를 작성할 수 있어야 한다
- 유사 상품 추천 (pgvector 활용)

Jason Fried의 [Obvious-Easy-Possible 프레임워크](https://medium.com/signal-v-noise/the-obvious-the-easy-and-the-possible-a09387ad3652)를 적용한다. 회원가입/로그인은 Obvious(항상 하는 것), 주문 내역 조회는 Easy(자주 하는 것), 리뷰 작성은 Possible(가끔 하는 것)이다. UI와 API 설계에서 이 차이를 반영한다.

> **참고영상** : [Why work doesn't happen at work - Jason Fried](https://www.youtube.com/watch?v=5XD2kNopsUs)

### 2.2 아키텍처 결정 기록 (ADR)

Kent Beck이 지적했듯이, 완전한 스펙을 미리 작성하려는 시도는 잘못된 것이다. 대신 **주요 아키텍처 결정만 미리 문서화**한다.

`docs/adr/` 디렉토리를 만들고 ADR 템플릿을 사용한다:

**ADR-001: 헥사고날 아키텍처 채택**

```markdown
# ADR-001: 헥사고날 아키텍처 (포트-어댑터 패턴) 채택

## 상태
승인됨

## 컨텍스트
4일이라는 짧은 기간에 프로덕션 시스템을 만들어야 한다. 동시에 향후 확장 가능성을 고려해야 한다.

## 결정
헥사고날 아키텍처를 채택한다. 비즈니스 로직(도메인)을 인프라(DB, API, 메시징)로부터 분리한다.

구조:
- `domain`: 비즈니스 로직, 엔티티, 유스케이스 인터페이스
- `application`: 유스케이스 구현, 서비스 레이어
- `adapter.in.web`: REST API 컨트롤러
- `adapter.out.persistence`: JPA 리포지토리 구현
- `adapter.out.messaging`: Kafka 프로듀서/컨슈머

## 결과
- 장점: 테스트 가능성 향상, 변경 용이성, 명확한 의존성 방향
- 단점: 초기 설정 복잡도, 보일러플레이트 코드
- 완화: Claude Code로 보일러플레이트 자동 생성
```

**ADR-002: API-First 설계**

```markdown
# ADR-002: API-First 설계 (OpenAPI 스펙 우선)

## 상태
승인됨

## 컨텍스트
프론트엔드와 백엔드를 병렬로 개발해야 한다.

## 결정
OpenAPI 3.0 스펙을 먼저 작성하고, 스펙에서 서버 스텁과 클라이언트 SDK를 자동 생성한다.

도구:
- Springdoc OpenAPI (서버 문서 자동 생성)
- OpenAPI Generator (TypeScript 클라이언트 SDK 생성)

## 결과
프론트엔드와 백엔드가 독립적으로 개발 가능. 인터페이스 변경 시 컴파일 에러로 즉시 감지.
```

**ADR-003: 이벤트 기반 아키텍처 (Kafka)**

```markdown
# ADR-003: 주문 처리에 이벤트 기반 아키텍처 적용

## 상태
승인됨

## 컨텍스트
주문 시 재고 확인, 결제, 알림 등 여러 작업이 필요하다.

## 결정
Kafka를 사용한 이벤트 기반 아키텍처를 적용한다.

이벤트 플로우:
1. OrderCreated 이벤트 발행
2. InventoryService가 재고 확인
3. InventoryReserved 이벤트 발행
4. PaymentService가 결제 처리
5. PaymentCompleted 이벤트 발행
6. OrderConfirmed 이벤트 발행 → 이메일 발송

## 결과
느슨한 결합, 확장 가능, 장애 격리. 단, 분산 트랜잭션 복잡도 증가 (Saga 패턴으로 해결).
```

이렇게 3-5개의 핵심 ADR만 작성한다. 30분이면 충분하다. 나머지는 개발하면서 결정하고 기록한다.

### 2.3 데이터 모델 스케치

완전한 ERD를 그리지 않는다. 대신 **핵심 엔티티와 관계만 스케치**한다. Claude Code가 이것을 보고 JPA 엔티티를 생성할 수 있을 정도면 충분하다.

```
User
- id: UUID
- email: String (unique)
- passwordHash: String
- name: String
- createdAt: Timestamp

Product
- id: UUID
- name: String
- description: Text
- price: Decimal
- stock: Integer
- categoryId: UUID
- embedding: vector(768)  -- for recommendations
- createdAt: Timestamp

Category
- id: UUID
- name: String
- parentId: UUID (nullable, self-reference)

Order
- id: UUID
- userId: UUID → User
- status: Enum (PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED)
- totalAmount: Decimal
- createdAt: Timestamp

OrderItem
- id: UUID
- orderId: UUID → Order
- productId: UUID → Product
- quantity: Integer
- priceAtOrder: Decimal

Cart
- id: UUID
- userId: UUID → User
- createdAt: Timestamp

CartItem
- id: UUID
- cartId: UUID → Cart
- productId: UUID → Product
- quantity: Integer
```

pgvector를 사용한 상품 추천을 위해 Product 테이블에 embedding 컬럼을 추가한 것을 주목하라. 이것은 나중에 "이 상품을 본 사람들이 본 다른 상품" 기능을 구현할 때 사용한다.

### 2.4 개발 환경 최종 점검

프로젝트 구조를 생성한다. Claude Code에게 "헥사고날 아키텍처를 따르는 Spring Boot 멀티 모듈 프로젝트를 생성해줘. 모듈은 domain, application, adapter-web, adapter-persistence, adapter-messaging으로 나눠줘"라고 요청하면 된다.

React 프론트엔드도 마찬가지다. "Vite + React + TypeScript 프로젝트를 생성해줘. TailwindCSS와 React Router를 설정하고, src/features 디렉토리에 feature-based 구조를 만들어줘."

이제 `docker-compose up -d`로 인프라를 띄우고, 백엔드와 프론트엔드가 각각 빌드되고 실행되는지 확인한다. 여기까지 1-2시간이면 충분하다.

---

## 제3장: Day 1 - 기초 인프라와 인증 (월요일)

### 3.1 오전 (9:00-12:00): 프로젝트 스캐폴딩과 CI/CD

**시작 스탠드업 (9:00-9:15)**

PM과 QA를 포함한 전체 팀이 모여 Day 1 목표를 확인한다:
- 백엔드: 사용자 인증 API (회원가입, 로그인, 토큰 갱신)
- 프론트엔드: 로그인/회원가입 페이지
- 인프라: CI/CD 파이프라인, 배포 환경
- QA: 테스트 계획 수립

**PL의 첫 번째 태스크: OpenAPI 스펙 정의 (9:15-10:00)**

PL은 인증 관련 API 스펙을 OpenAPI 형식으로 작성한다. Claude Code에게 "사용자 인증 API의 OpenAPI 3.0 스펙을 작성해줘. 엔드포인트는 POST /api/auth/register, POST /api/auth/login, POST /api/auth/refresh야. 각 엔드포인트의 요청/응답 스키마와 에러 코드를 정의해줘"라고 요청한다.

생성된 스펙을 리뷰하고 수정한다. 특히 에러 응답 형식을 표준화한다:

```yaml
components:
  schemas:
    ErrorResponse:
      type: object
      properties:
        timestamp:
          type: string
          format: date-time
        status:
          type: integer
        error:
          type: string
        message:
          type: string
        path:
          type: string
```

스펙이 완성되면 `backend/src/main/resources/openapi.yaml`에 저장하고 Git에 커밋한다. 이제 프론트엔드 개발자는 이 스펙을 보고 독립적으로 작업할 수 있다.

**개발자 1: 백엔드 인증 구조 (9:15-12:00)**

Claude Code 세션을 시작한다. Fernandez의 TDD 접근법을 따른다.

**첫 번째 단계: 테스트 작성**

"UserAuthenticationService의 회원가입 기능을 테스트하는 단위 테스트를 작성해줘. 이미 존재하는 이메일로 가입하면 DuplicateEmailException을 던져야 해. 비밀번호는 8자 이상이어야 하고, 그렇지 않으면 InvalidPasswordException을 던져야 해."

```java
void registerUser_WithValidData_ShouldCreateUser() {
    // given
    RegisterUserCommand command = new RegisterUserCommand(
        "user@example.com",
        "password123",
        "John Doe"
    );
    
    // when
    User user = authService.registerUser(command);
    
    // then
    assertThat(user.getEmail()).isEqualTo("user@example.com");
    assertThat(user.getName()).isEqualTo("John Doe");
    assertThat(passwordEncoder.matches("password123", user.getPasswordHash()))
        .isTrue();
}

void registerUser_WithDuplicateEmail_ShouldThrowException() {
    // given
    RegisterUserCommand command = new RegisterUserCommand(
        "existing@example.com",
        "password123",
        "John Doe"
    );
    when(userRepository.existsByEmail("existing@example.com"))
        .thenReturn(true);
    
    // when & then
    assertThatThrownBy(() -> authService.registerUser(command))
        .isInstanceOf(DuplicateEmailException.class);
}
```

테스트를 실행한다. 당연히 실패한다. 이것이 red 단계다.

**두 번째 단계: 최소 구현**

"이제 UserAuthenticationService를 구현해줘. 헥사고날 아키텍처를 따라 domain, application, adapter 레이어로 분리해줘. User는 도메인 엔티티, UserAuthenticationService는 유스케이스 인터페이스, UserAuthenticationServiceImpl은 application 레이어의 구현체야."

Claude Code가 여러 파일을 생성한다. 각 파일을 확인하면서 비즈니스 로직이 올바른지 검토한다. 특히:
- 비밀번호 해싱이 BCrypt를 사용하는가?
- 도메인 엔티티가 불변성을 유지하는가?
- 예외 처리가 적절한가?

테스트를 실행한다. 녹색이 되어야 한다. 이것이 green 단계다.

**세 번째 단계: 리팩터링**

"RegisterUserCommand와 LoginCommand를 별도의 DTO 클래스로 추출해줘. 그리고 비밀번호 검증 로직을 PasswordValidator 유틸리티 클래스로 분리해줘."

리팩터링 후 테스트를 다시 실행한다. 여전히 녹색이어야 한다. 이것이 refactor 단계다.

이 TDD 사이클을 로그인, 토큰 발급, 토큰 갱신 기능에 반복한다. 각 기능마다 30-40분이 걸린다. 12시까지 인증 비즈니스 로직의 핵심이 완성된다.

**개발자 2: 프론트엔드 인증 UI 준비 (9:15-12:00)**

OpenAPI 스펙에서 TypeScript 클라이언트 SDK를 생성한다. `openapi-generator-cli`를 사용한다:

```bash
openapi-generator-cli generate \
  -i backend/src/main/resources/openapi.yaml \
  -g typescript-axios \
  -o frontend/src/api
```

생성된 SDK를 리뷰한다. 타입이 제대로 정의되었는지 확인한다.

이제 Claude Code에게 "React Query를 사용해서 인증 API를 호출하는 custom hook을 만들어줘. useRegister, useLogin, useRefreshToken 훅이 필요해. 각 훅은 mutation을 반환하고, 성공/실패/로딩 상태를 관리해야 해."

```typescript
export function useRegister() {
  return useMutation({
    mutationFn: (data: RegisterRequest) => 
      authApi.register(data),
    onSuccess: (response) => {
      // 토큰을 localStorage에 저장
      localStorage.setItem('accessToken', response.data.accessToken);
      localStorage.setItem('refreshToken', response.data.refreshToken);
    },
    onError: (error: ApiError) => {
      // 에러 처리
      toast.error(error.message);
    }
  });
}
```

이제 로그인/회원가입 폼 컴포넌트를 만든다. "Tailwind CSS와 React Hook Form을 사용해서 회원가입 폼을 만들어줘. 이메일, 비밀번호, 이름 입력 필드가 있어야 하고, 유효성 검사를 추가해줘. 이메일 형식, 비밀번호 8자 이상을 검증해."

생성된 컴포넌트를 Storybook에 추가해서 PM과 디자이너가 리뷰할 수 있게 한다.

### 3.2 오후 (13:00-18:00): 인증 API 완성과 통합

**점심 후 스탠드업 (13:00-13:15)**

오전 진행 상황을 공유한다. 백엔드는 비즈니스 로직이 완성되었고, 프론트엔드는 UI가 준비되었다. 오후에는 API 엔드포인트를 추가하고 통합 테스트를 작성한다.

**개발자 1: REST 컨트롤러와 통합 테스트 (13:15-16:00)**

Spring MVC 컨트롤러를 추가한다. Claude Code에게 "UserAuthenticationController를 만들어줘. POST /api/auth/register, /api/auth/login, /api/auth/refresh 엔드포인트를 추가하고, OpenAPI 스펙에 맞게 요청/응답 DTO를 매핑해줘. @Valid 어노테이션으로 유효성 검증도 추가해줘."

컨트롤러가 생성되면 통합 테스트를 작성한다. "AuthenticationControllerIntegrationTest를 작성해줘. @SpringBootTest와 TestRestTemplate을 사용해서 실제 HTTP 요청을 보내고 응답을 검증해. 회원가입 → 로그인 → 토큰 갱신 흐름을 테스트해."

```java
class AuthenticationControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void authenticationFlow_ShouldWork() {
        // 1. 회원가입
        RegisterRequest registerRequest = new RegisterRequest(
            "test@example.com",
            "password123",
            "Test User"
        );
        
        ResponseEntity<AuthResponse> registerResponse = 
            restTemplate.postForEntity(
                "/api/auth/register",
                registerRequest,
                AuthResponse.class
            );
        
        assertThat(registerResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(registerResponse.getBody().getAccessToken()).isNotNull();
        
        // 2. 로그인
        LoginRequest loginRequest = new LoginRequest(
            "test@example.com",
            "password123"
        );
        
        ResponseEntity<AuthResponse> loginResponse =
            restTemplate.postForEntity(
                "/api/auth/login",
                loginRequest,
                AuthResponse.class
            );
        
        assertThat(loginResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        // 3. 토큰 갱신
        String refreshToken = loginResponse.getBody().getRefreshToken();
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(refreshToken);
        
        ResponseEntity<AuthResponse> refreshResponse =
            restTemplate.exchange(
                "/api/auth/refresh",
                HttpMethod.POST,
                new HttpEntity<>(headers),
                AuthResponse.class
            );
        
        assertThat(refreshResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```

통합 테스트가 녹색이 되면 실제 요청을 보내본다. Postman이나 curl로 API를 호출해서 동작을 확인한다.

**개발자 2: 프론트엔드 통합 및 E2E 테스트 (13:15-16:00)**

백엔드 API가 준비되는 동안 Mock Service Worker(MSW)를 사용해서 API를 모킹하고 개발을 계속한다. "MSW 핸들러를 추가해서 /api/auth/* 엔드포인트를 모킹해줘. 회원가입과 로그인 요청에 가짜 토큰을 반환하도록."

백엔드가 준비되면 실제 API로 전환한다. 환경 변수로 API URL을 설정한다:

```typescript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 
  'http://localhost:8080';
```

이제 E2E 테스트를 작성한다. Playwright를 사용한다. "Playwright로 회원가입 E2E 테스트를 작성해줘. 회원가입 폼에 정보를 입력하고 제출하면 대시보드로 리디렉션되어야 해."

```typescript
test('user can sign up and login', async ({ page }) => {
  // 회원가입 페이지로 이동
  await page.goto('/signup');
  
  // 폼 작성
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.fill('input[name="name"]', 'Test User');
  
  // 제출
  await page.click('button[type="submit"]');
  
  // 대시보드로 리디렉션 확인
  await expect(page).toHaveURL('/dashboard');
  
  // 사용자 이름이 표시되는지 확인
  await expect(page.locator('text=Test User')).toBeVisible();
});
```

E2E 테스트가 통과하면 PM에게 데모를 보여준다.

**PL: 보안 검토와 개선 (16:00-18:00)**

인증 구현을 보안 관점에서 검토한다. Claude Code에게 "현재 인증 구현의 보안 취약점을 분석해줘. OWASP Top 10을 기준으로 체크리스트를 만들어줘"라고 요청한다.

주요 체크 항목:
- SQL Injection: JPA Criteria API나 @Query with named parameters 사용 확인
- XSS: React가 기본적으로 방어하지만, dangerouslySetInnerHTML 사용 금지
- CSRF: SameSite 쿠키 설정, CORS 정책 확인
- Password Storage: BCrypt 사용 확인, 솔트 자동 생성 확인
- JWT Security: 짧은 만료 시간, HTTPS only, HttpOnly 쿠키 고려
- Rate Limiting: 로그인 시도 제한 추가 필요

Rate Limiting을 추가한다. "Bucket4j를 사용해서 로그인 API에 rate limiting을 추가해줘. IP당 분당 5회로 제한하고, 초과하면 429 Too Many Requests를 반환해."

보안 개선 사항을 적용하고 커밋한다.

### 3.3 저녁 (18:00-19:00): 리뷰와 회고

**코드 리뷰 세션 (18:00-18:30)**

두 개발자가 서로의 코드를 리뷰한다. 특히 다음을 확인한다:
- 테스트 커버리지: 주요 비즈니스 로직이 테스트되었는가?
- 에러 처리: 예외 상황이 적절히 처리되는가?
- 보안: 민감한 정보가 로그에 남지 않는가?
- 일관성: 코딩 컨벤션을 따르는가?

Claude Code에게 "이 코드의 코드 스멜을 찾아줘. SonarQube 규칙을 기준으로"라고 요청해서 자동 리뷰를 받을 수도 있다.

**일일 회고 (18:30-19:00)**

팀 전체가 모여 하루를 돌아본다. 네 가지 질문:
1. 무엇을 잘했는가? (TDD 사이클이 효과적이었다)
2. 무엇을 개선할 수 있는가? (통합 테스트를 더 일찍 작성할 수 있었다)
3. 무엇을 배웠는가? (Claude Code가 보일러플레이트를 빠르게 생성한다)
4. 내일 무엇을 할 것인가? (상품 카탈로그 기능)

회고 내용을 `docs/retrospectives/day1.md`에 기록한다. 이것은 나중에 프로젝트 전체를 돌아볼 때 귀중한 자료가 된다.

**Day 1 결과물 체크리스트:**
- ✅ 사용자 회원가입/로그인 API
- ✅ JWT 기반 인증 시스템
- ✅ 로그인/회원가입 UI
- ✅ E2E 테스트
- ✅ CI/CD 파이프라인
- ✅ 보안 rate limiting

첫날치고는 상당한 진전이다. 전통적인 개발이었다면 2-3일이 걸렸을 작업을 하루에 끝냈다.

---

## 제4장: Day 2 - 핵심 비즈니스 로직 (화요일)

### 4.1 오전 (9:00-12:00): 상품 카탈로그 API

**스탠드업 (9:00-9:15)**

Day 2 목표를 확인한다:
- 백엔드: 상품 CRUD API, 카테고리 관리, 검색/필터링
- 프론트엔드: 상품 목록 페이지, 상품 상세 페이지
- 통합: 이미지 업로드 (S3 또는 로컬 스토리지)

**PL: API 스펙 정의와 아키텍처 결정 (9:15-10:00)**

상품 관련 엔드포인트의 OpenAPI 스펙을 작성한다. 특히 검색과 필터링 파라미터를 명확히 정의한다:

```yaml
/api/products:
  get:
    parameters:
      - name: page
        in: query
        schema:
          type: integer
          default: 0
      - name: size
        in: query
        schema:
          type: integer
          default: 20
      - name: category
        in: query
        schema:
          type: string
      - name: minPrice
        in: query
        schema:
          type: number
      - name: maxPrice
        in: query
        schema:
          type: number
      - name: search
        in: query
        schema:
          type: string
      - name: sort
        in: query
        schema:
          type: string
          enum: [price_asc, price_desc, name_asc, created_desc]
```

검색 기능에 대한 아키텍처 결정을 한다. PostgreSQL의 Full-Text Search를 사용할 것인가, 아니면 Elasticsearch를 추가할 것인가? 4일이라는 제한된 시간을 고려하면 PostgreSQL의 `tsvector`로 충분하다. ADR을 작성한다.

**개발자 1: 상품 도메인 모델과 리포지토리 (9:15-12:00)**

TDD 사이클로 상품 도메인을 구현한다.

**테스트 작성:**

```java
void createProduct_WithValidData_ShouldCreateProduct() {
    // given
    CreateProductCommand command = new CreateProductCommand(
        "iPhone 15 Pro",
        "Latest iPhone with A17 Pro chip",
        new BigDecimal("1299.99"),
        100,
        categoryId
    );
    
    // when
    Product product = productService.createProduct(command);
    
    // then
    assertThat(product.getName()).isEqualTo("iPhone 15 Pro");
    assertThat(product.getPrice()).isEqualByComparingTo("1299.99");
    assertThat(product.getStock()).isEqualTo(100);
}

void searchProducts_WithKeyword_ShouldReturnMatchingProducts() {
    // given
    createTestProducts(); // "iPhone", "Samsung Galaxy", "MacBook"
    
    // when
    Page<Product> results = productService.searchProducts("iPhone", pageable);
    
    // then
    assertThat(results.getContent()).hasSize(1);
    assertThat(results.getContent().get(0).getName()).contains("iPhone");
}
```

**구현:**

Claude Code에게 "Product 엔티티를 JPA로 구현해줘. PostgreSQL의 tsvector를 사용해서 전체 텍스트 검색을 지원해야 해. name과 description을 인덱싱하고, @Formula 어노테이션으로 search_vector 컬럼을 추가해줘."

```java
public class Product {
    @Id
    @GeneratedValue
    private UUID id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;
    
    @Column(nullable = false)
    private Integer stock;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
    
    // Full-text search support
    @Formula("to_tsvector('english', coalesce(name, '') || ' ' || coalesce(description, ''))")
    private String searchVector;
}
```

리포지토리에 검색 메서드를 추가한다:

```java
public interface ProductRepository extends JpaRepository<Product, UUID> {
    
    @Query(value = "SELECT * FROM products WHERE " +
           "to_tsvector('english', name || ' ' || description) @@ " +
           "plainto_tsquery('english', :keyword)", 
           nativeQuery = true)
    Page<Product> searchByKeyword(@Param("keyword") String keyword, 
                                   Pageable pageable);
    
    Page<Product> findByCategoryIdAndPriceBetween(
        UUID categoryId,
        BigDecimal minPrice,
        BigDecimal maxPrice,
        Pageable pageable
    );
}
```

**리팩터링:**

검색 조건이 복잡해질 수 있으므로 Specification 패턴을 적용한다. "Spring Data JPA Specification을 사용해서 동적 쿼리를 구현해줘. ProductSearchCriteria DTO로 검색 조건을 받고, Specification으로 변환해줘."

```java
public class ProductSpecifications {
    
    public static Specification<Product> withSearchCriteria(
            ProductSearchCriteria criteria) {
        return (root, query, cb) -> {
            List<Predicate> predicates = new ArrayList<>();
            
            if (criteria.getSearch() != null) {
                // Full-text search
                predicates.add(cb.isTrue(
                    cb.function("ts_match", Boolean.class,
                        root.get("searchVector"),
                        cb.literal(criteria.getSearch())
                    )
                ));
            }
            
            if (criteria.getCategoryId() != null) {
                predicates.add(cb.equal(
                    root.get("category").get("id"),
                    criteria.getCategoryId()
                ));
            }
            
            if (criteria.getMinPrice() != null) {
                predicates.add(cb.greaterThanOrEqualTo(
                    root.get("price"),
                    criteria.getMinPrice()
                ));
            }
            
            if (criteria.getMaxPrice() != null) {
                predicates.add(cb.lessThanOrEqualTo(
                    root.get("price"),
                    criteria.getMaxPrice()
                ));
            }
            
            return cb.and(predicates.toArray(new Predicate[0]));
        };
    }
}
```

이제 테스트를 실행하고 녹색을 확인한다.

**개발자 2: 상품 목록 UI (9:15-12:00)**

React Query와 무한 스크롤로 상품 목록을 구현한다. Claude Code에게 "React Query의 useInfiniteQuery를 사용해서 상품 목록을 가져오는 custom hook을 만들어줘. react-intersection-observer로 무한 스크롤을 구현하고, 필터링 옵션(카테고리, 가격 범위, 검색)을 지원해줘."

```typescript
export function useProducts(filters: ProductFilters) {
  return useInfiniteQuery({
    queryKey: ['products', filters],
    queryFn: ({ pageParam = 0 }) => 
      productsApi.getProducts({
        page: pageParam,
        size: 20,
        ...filters
      }),
    getNextPageParam: (lastPage) => 
      lastPage.hasNext ? lastPage.page + 1 : undefined,
  });
}

export function ProductList() {
  const [filters, setFilters] = useState<ProductFilters>({});
  const { data, fetchNextPage, hasNextPage, isLoading } = 
    useProducts(filters);
  const { ref, inView } = useInView();
  
  useEffect(() => {
    if (inView && hasNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, fetchNextPage]);
  
  return (
    <div>
      <ProductFilters filters={filters} onChange={setFilters} />
      <div className="grid grid-cols-4 gap-4">
        {data?.pages.flatMap(page => page.content).map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
      <div ref={ref} />
      {isLoading && <Spinner />}
    </div>
  );
}
```

상품 카드 컴포넌트를 만든다. "ProductCard 컴포넌트를 만들어줘. 상품 이미지, 이름, 가격, 재고 상태를 표시하고, 클릭하면 상세 페이지로 이동해. Tailwind로 깔끔하게 스타일링해줘."

### 4.2 오후 (13:00-18:00): 장바구니와 주문

**스탠드업 (13:00-13:15)**

오전 성과를 공유하고 오후 계획을 세운다. 상품 API가 완성되었으므로 이제 장바구니와 주문 기능을 추가한다.

**개발자 1: 장바구니 API (13:15-15:00)**

장바구니는 상태 관리가 중요하다. Redis를 사용해서 세션 기반 장바구니를 구현한다.

"CartService를 구현해줘. Redis를 사용해서 장바구니 정보를 저장하고, TTL은 7일로 설정해. 장바구니에 상품 추가, 수량 변경, 제거 기능이 있어야 해. 재고를 초과하는 수량을 담으려고 하면 InsufficientStockException을 던져야 해."

```java
public class CartServiceImpl implements CartService {
    
    private final RedisTemplate<String, Cart> redisTemplate;
    private final ProductRepository productRepository;
    
    @Override
    public Cart addItem(UUID userId, UUID productId, int quantity) {
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
        
        if (product.getStock() < quantity) {
            throw new InsufficientStockException(productId, quantity);
        }
        
        String cartKey = "cart:" + userId;
        Cart cart = redisTemplate.opsForValue().get(cartKey);
        if (cart == null) {
            cart = new Cart(userId);
        }
        
        cart.addItem(new CartItem(productId, product.getName(), 
                                  product.getPrice(), quantity));
        
        redisTemplate.opsForValue().set(cartKey, cart, 
                                        Duration.ofDays(7));
        
        return cart;
    }
}
```

통합 테스트를 작성한다. "CartServiceIntegrationTest를 작성해줘. Testcontainers로 실제 Redis를 띄우고 테스트해. 장바구니에 상품을 추가하고, 수량을 변경하고, 제거하는 시나리오를 테스트해."

**개발자 1: 주문 API와 이벤트 (15:00-18:00)**

주문은 복잡한 비즈니스 로직이 있다. 재고 확인, 결제, 주문 확정이 모두 성공해야 한다. 이것을 Saga 패턴으로 구현한다.

"OrderService를 구현해줘. 주문 생성 시 다음 단계를 따라야 해:
1. 장바구니에서 상품 목록 가져오기
2. 재고 확인 및 예약
3. 주문 엔티티 생성 (상태: PENDING)
4. OrderCreated 이벤트를 Kafka로 발행
5. 트랜잭션 커밋

OrderCreated 이벤트를 받으면:
1. InventoryService가 재고를 차감하고 InventoryReserved 이벤트 발행
2. PaymentService가 결제를 처리하고 PaymentCompleted 이벤트 발행
3. OrderService가 주문 상태를 CONFIRMED로 변경

만약 어느 단계에서든 실패하면 보상 트랜잭션(compensation)을 실행해."

이것은 복잡하므로 단계별로 구현한다. 먼저 주문 생성만:

```java
public class OrderServiceImpl implements OrderService {
    
    private final OrderRepository orderRepository;
    private final CartService cartService;
    private final ProductRepository productRepository;
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    @Override
    public Order createOrder(UUID userId) {
        Cart cart = cartService.getCart(userId);
        if (cart.isEmpty()) {
            throw new EmptyCartException();
        }
        
        // 재고 확인
        for (CartItem item : cart.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
            
            if (product.getStock() < item.getQuantity()) {
                throw new InsufficientStockException(
                    item.getProductId(), 
                    item.getQuantity()
                );
            }
        }
        
        // 주문 생성
        Order order = Order.builder()
            .userId(userId)
            .status(OrderStatus.PENDING)
            .items(cart.getItems().stream()
                .map(item -> OrderItem.builder()
                    .productId(item.getProductId())
                    .quantity(item.getQuantity())
                    .priceAtOrder(item.getPrice())
                    .build())
                .collect(Collectors.toList()))
            .totalAmount(cart.getTotalAmount())
            .build();
        
        order = orderRepository.save(order);
        
        // 이벤트 발행
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getItems(),
            order.getTotalAmount()
        );
        
        kafkaTemplate.send("order-events", event);
        
        // 장바구니 비우기
        cartService.clearCart(userId);
        
        return order;
    }
}
```

이제 이벤트 리스너를 구현한다:

```java
public class OrderEventListener {
    
    @KafkaListener(topics = "order-events")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Order created: {}", event.getOrderId());
        // InventoryService 호출하여 재고 예약
        inventoryService.reserveInventory(event.getOrderId(), event.getItems());
    }
    
    @KafkaListener(topics = "inventory-events")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        log.info("Inventory reserved for order: {}", event.getOrderId());
        // PaymentService 호출하여 결제 처리
        paymentService.processPayment(event.getOrderId(), event.getAmount());
    }
    
    @KafkaListener(topics = "payment-events")
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        log.info("Payment completed for order: {}", event.getOrderId());
        // 주문 상태 업데이트
        orderService.confirmOrder(event.getOrderId());
    }
}
```

이것은 간단한 Saga 구현이다. 프로덕션에서는 Saga 상태를 저장하고, 타임아웃을 처리하고, 멱등성을 보장해야 하지만, 4일 프로젝트에서는 이 정도면 충분하다.

**개발자 2: 장바구니 UI와 주문 플로우 (13:15-18:00)**

장바구니를 전역 상태로 관리한다. Zustand를 사용한다.

"Zustand로 장바구니 상태 관리 스토어를 만들어줘. addItem, removeItem, updateQuantity, clearCart 액션이 있어야 해. localStorage에 동기화해서 새로고침해도 유지되도록."

```typescript
export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      addItem: (product, quantity) => {
        const existing = get().items.find(
          item => item.product.id === product.id
        );
        
        if (existing) {
          set({
            items: get().items.map(item =>
              item.product.id === product.id
                ? { ...item, quantity: item.quantity + quantity }
                : item
            )
          });
        } else {
          set({
            items: [...get().items, { product, quantity }]
          });
        }
      },
      removeItem: (productId) => {
        set({
          items: get().items.filter(
            item => item.product.id !== productId
          )
        });
      },
      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeItem(productId);
        } else {
          set({
            items: get().items.map(item =>
              item.product.id === productId
                ? { ...item, quantity }
                : item
            )
          });
        }
      },
      clearCart: () => set({ items: [] }),
      getTotalAmount: () => {
        return get().items.reduce(
          (sum, item) => sum + item.product.price * item.quantity,
          0
        );
      }
    }),
    {
      name: 'cart-storage',
    }
  )
);
```

장바구니 페이지를 만든다. "CartPage 컴포넌트를 만들어줘. 장바구니에 담긴 상품 목록을 테이블로 표시하고, 수량 조정, 제거 기능을 추가해. 총 금액을 계산해서 보여주고, '주문하기' 버튼을 클릭하면 주문 API를 호출해."

주문 완료 페이지도 만든다. "OrderCompletePage 컴포넌트를 만들어줘. 주문 번호, 주문 상품 목록, 총 금액을 표시하고, '주문 내역 보기' 버튼을 추가해."

### 4.3 저녁 (18:00-19:00): E2E 테스트와 회고

**E2E 테스트 작성 (18:00-18:30)**

전체 주문 플로우를 E2E로 테스트한다. "Playwright로 전체 주문 플로우 E2E 테스트를 작성해줘:
1. 로그인
2. 상품 목록에서 상품 검색
3. 상품을 장바구니에 추가
4. 장바구니 페이지로 이동
5. 수량 조정
6. 주문하기
7. 주문 완료 페이지 확인"

```typescript
test('complete order flow', async ({ page }) => {
  // 1. 로그인
  await page.goto('/login');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  // 2. 상품 검색
  await page.goto('/products');
  await page.fill('input[name="search"]', 'iPhone');
  await page.click('button[type="submit"]');
  
  // 3. 첫 번째 상품 클릭
  await page.click('.product-card:first-child');
  
  // 4. 장바구니에 추가
  await page.click('button:has-text("장바구니에 담기")');
  
  // 5. 장바구니로 이동
  await page.click('a:has-text("장바구니")');
  
  // 6. 수량 조정
  await page.click('button[aria-label="수량 증가"]');
  await expect(page.locator('input[type="number"]')).toHaveValue('2');
  
  // 7. 주문하기
  await page.click('button:has-text("주문하기")');
  
  // 8. 주문 완료 확인
  await expect(page.locator('h1')).toContainText('주문이 완료되었습니다');
  await expect(page.locator('.order-number')).toBeVisible();
});
```

E2E 테스트를 CI 파이프라인에 추가한다. GitLab CI 설정을 업데이트한다:

```yaml
e2e-test:
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0
  services:
    - postgres:15
    - redis:7
  script:
    - cd frontend
    - npm ci
    - npx playwright install
    - npm run test:e2e
  only:
    - merge_requests
    - main
```

**회고 (18:30-19:00)**

Day 2를 돌아본다. 이틀 만에 핵심 기능이 대부분 완성되었다. 인증, 상품 카탈로그, 장바구니, 주문까지. 전통적인 개발이었다면 일주일은 걸렸을 것이다.

특히 잘된 점:
- TDD 사이클이 빠른 반복을 가능하게 했다
- API-First 설계로 프론트엔드와 백엔드가 독립적으로 개발되었다
- 이벤트 기반 아키텍처가 복잡한 비즈니스 로직을 깔끔하게 분리했다

개선할 점:
- Kafka 이벤트 처리의 에러 핸들링이 미흡하다 (내일 보완)
- E2E 테스트가 너무 늦게 작성되었다 (내일부터는 더 일찍)

**Day 2 결과물:**
- ✅ 상품 CRUD API
- ✅ 카테고리 관리
- ✅ 전체 텍스트 검색
- ✅ 장바구니 (Redis)
- ✅ 주문 생성 (Saga 패턴)
- ✅ 상품 목록/상세 UI
- ✅ 장바구니 UI
- ✅ E2E 테스트

---

## 제5장: Day 3 - 고급 기능과 최적화 (수요일)

### 5.1 오전 (9:00-12:00): 관리자 기능과 캐싱

**스탠드업 (9:00-9:15)**

Day 3는 관리자 기능과 성능 최적화에 집중한다. 상품 관리 어드민 페이지, Redis 캐싱, pgvector를 사용한 상품 추천을 구현한다.

**개발자 1: 상품 관리 API와 Redis 캐싱 (9:15-12:00)**

관리자용 상품 CRUD API를 만든다. 일반 사용자와 다른 권한이 필요하므로 Spring Security의 @PreAuthorize를 사용한다.

"AdminProductController를 만들어줘. @PreAuthorize("hasRole('ADMIN')")로 권한을 체크하고, 상품 등록/수정/삭제 엔드포인트를 추가해. 상품 등록 시 이미지 업로드를 지원해야 해."

```java
public class AdminProductController {
    
    @PostMapping
    public ResponseEntity<Product> createProduct(
            @Valid @RequestBody CreateProductRequest request,
            @RequestParam("image") MultipartFile image) {
        
        String imageUrl = imageUploadService.upload(image);
        
        CreateProductCommand command = CreateProductCommand.builder()
            .name(request.getName())
            .description(request.getDescription())
            .price(request.getPrice())
            .stock(request.getStock())
            .categoryId(request.getCategoryId())
            .imageUrl(imageUrl)
            .build();
        
        Product product = productService.createProduct(command);
        
        return ResponseEntity.ok(product);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable UUID id,
            @Valid @RequestBody UpdateProductRequest request) {
        
        Product product = productService.updateProduct(id, request);
        
        return ResponseEntity.ok(product);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteProduct(@PathVariable UUID id) {
        productService.deleteProduct(id);
    }
}
```

이제 캐싱을 추가한다. Fernandez의 Nexus 프로젝트처럼 상품 목록 조회는 자주 일어나지만 변경은 드물다. 완벽한 캐싱 대상이다.

"ProductService에 Spring Cache를 적용해줘. Redis를 캐시 스토어로 사용하고, 상품 목록 조회는 5분, 상품 상세 조회는 10분 TTL로 캐싱해. 상품이 수정되거나 삭제되면 관련 캐시를 무효화해."

```java
public class ProductServiceImpl implements ProductService {
    
    @Cacheable(value = "products:list", 
               key = "#criteria.hashCode()",
               unless = "#result.isEmpty()")
    @Override
    public Page<Product> searchProducts(ProductSearchCriteria criteria, 
                                        Pageable pageable) {
        return productRepository.findAll(
            ProductSpecifications.withSearchCriteria(criteria),
            pageable
        );
    }
    
    @Cacheable(value = "products:detail", key = "#id")
    @Override
    public Product getProduct(UUID id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }
    
    @CacheEvict(value = {"products:list", "products:detail"}, 
                allEntries = true)
    @Override
    public Product updateProduct(UUID id, UpdateProductRequest request) {
        // 업데이트 로직
    }
    
    @CacheEvict(value = {"products:list", "products:detail"}, 
                allEntries = true)
    @Override
    public void deleteProduct(UUID id) {
        productRepository.deleteById(id);
    }
}
```

캐시 설정을 추가한다:

```java
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration
            .defaultCacheConfig()
            .serializeValuesWith(
                RedisSerializationContext
                    .SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer())
            );
        
        Map<String, RedisCacheConfiguration> cacheConfigurations = Map.of(
            "products:list", config.entryTtl(Duration.ofMinutes(5)),
            "products:detail", config.entryTtl(Duration.ofMinutes(10))
        );
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

성능 테스트를 작성한다. "캐싱 전후의 응답 시간을 비교하는 성능 테스트를 작성해줘. JMH를 사용해서 벤치마킹해."

**개발자 2: 관리자 대시보드 UI (9:15-12:00)**

관리자 전용 페이지를 만든다. "AdminLayout 컴포넌트를 만들어줘. 사이드바에 상품 관리, 주문 관리, 사용자 관리 메뉴가 있어야 해. 권한이 없는 사용자가 접근하면 403 페이지로 리디렉션해."

상품 관리 페이지를 구현한다. "ProductManagementPage를 만들어줘. Tanstack Table로 상품 목록을 표시하고, 검색, 필터링, 정렬 기능을 추가해. 각 행에 수정/삭제 버튼을 추가하고, '상품 등록' 버튼을 클릭하면 모달이 열려야 해."

```typescript
export function ProductManagementPage() {
  const [filters, setFilters] = useState<ProductFilters>({});
  const { data, isLoading } = useProducts(filters);
  const deleteProduct = useDeleteProduct();
  
  const columns = useMemo<ColumnDef<Product>[]>(() => [
    {
      accessorKey: 'name',
      header: '상품명',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <img 
            src={row.original.imageUrl} 
            className="w-10 h-10 rounded"
          />
          <span>{row.original.name}</span>
        </div>
      ),
    },
    {
      accessorKey: 'category.name',
      header: '카테고리',
    },
    {
      accessorKey: 'price',
      header: '가격',
      cell: ({ row }) => formatCurrency(row.original.price),
    },
    {
      accessorKey: 'stock',
      header: '재고',
      cell: ({ row }) => (
        <Badge variant={row.original.stock > 0 ? 'success' : 'error'}>
          {row.original.stock}
        </Badge>
      ),
    },
    {
      id: 'actions',
      header: '작업',
      cell: ({ row }) => (
        <div className="flex gap-2">
          <Button 
            size="sm" 
            onClick={() => openEditModal(row.original)}
          >
            수정
          </Button>
          <Button 
            size="sm" 
            variant="danger"
            onClick={() => deleteProduct.mutate(row.original.id)}
          >
            삭제
          </Button>
        </div>
      ),
    },
  ], []);
  
  return (
    <AdminLayout>
      <div className="p-6">
        <div className="flex justify-between items-center mb-6">
          <h1 className="text-2xl font-bold">상품 관리</h1>
          <Button onClick={() => setIsCreateModalOpen(true)}>
            상품 등록
          </Button>
        </div>
        
        <ProductFilters filters={filters} onChange={setFilters} />
        
        <DataTable columns={columns} data={data?.content ?? []} />
        
        <ProductFormModal 
          open={isCreateModalOpen}
          onClose={() => setIsCreateModalOpen(false)}
        />
      </div>
    </AdminLayout>
  );
}
```

이미지 업로드 기능을 추가한다. "react-dropzone을 사용해서 이미지 업로드 컴포넌트를 만들어줘. 드래그 앤 드롭과 클릭으로 이미지를 선택할 수 있어야 하고, 미리보기를 보여줘. 파일 크기는 5MB로 제한하고, 이미지 파일만 허용해."

### 5.2 오후 (13:00-18:00): pgvector 추천 시스템과 주문 관리

**스탠드업 (13:00-13:15)**

오후에는 pgvector를 사용한 상품 추천과 주문 관리 기능을 구현한다.

**개발자 1: pgvector 추천 시스템 (13:15-16:00)**

Fernandez의 Nexus 프로젝트가 지식 검색에 pgvector를 사용했듯이, 우리는 상품 추천에 사용한다.

먼저 상품 설명에서 임베딩을 생성한다. OpenAI Embedding API를 사용한다.

"EmbeddingService를 만들어줘. OpenAI의 text-embedding-ada-002 모델을 사용해서 상품 이름과 설명을 768차원 벡터로 변환해. RestTemplate으로 API를 호출하고, 결과를 캐싱해."

```java
public class EmbeddingService {
    
    private final RestTemplate restTemplate;
    private final String apiKey;
    
    @Cacheable(value = "embeddings", key = "#text.hashCode()")
    public float[] generateEmbedding(String text) {
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(apiKey);
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        Map<String, Object> request = Map.of(
            "input", text,
            "model", "text-embedding-ada-002"
        );
        
        HttpEntity<Map<String, Object>> entity = 
            new HttpEntity<>(request, headers);
        
        ResponseEntity<EmbeddingResponse> response = 
            restTemplate.postForEntity(
                "https://api.openai.com/v1/embeddings",
                entity,
                EmbeddingResponse.class
            );
        
        return response.getBody().getData().get(0).getEmbedding();
    }
}
```

상품 생성 시 자동으로 임베딩을 생성한다:

```java
public class ProductServiceImpl implements ProductService {
    
    @Override
    @Transactional
    public Product createProduct(CreateProductCommand command) {
        // 임베딩 생성
        String textToEmbed = command.getName() + " " + 
                            command.getDescription();
        float[] embedding = embeddingService.generateEmbedding(textToEmbed);
        
        Product product = Product.builder()
            .name(command.getName())
            .description(command.getDescription())
            .price(command.getPrice())
            .stock(command.getStock())
            .categoryId(command.getCategoryId())
            .embedding(embedding)
            .build();
        
        return productRepository.save(product);
    }
    
    @Override
    public List<Product> findSimilarProducts(UUID productId, int limit) {
        Product product = getProduct(productId);
        
        return productRepository.findSimilarByEmbedding(
            product.getEmbedding(),
            limit
        );
    }
}
```

리포지토리에 유사도 검색 쿼리를 추가한다:

```java
public interface ProductRepository extends JpaRepository<Product, UUID> {
    
    @Query(value = """
        SELECT * FROM products 
        WHERE id != :excludeId
        ORDER BY embedding <-> CAST(:embedding AS vector)
        LIMIT :limit
        """, nativeQuery = true)
    List<Product> findSimilarByEmbedding(
        @Param("embedding") float[] embedding,
        @Param("excludeId") UUID excludeId,
        @Param("limit") int limit
    );
}
```

이제 상품 상세 페이지에서 "유사한 상품" API를 호출할 수 있다. "GET /api/products/{id}/similar" 엔드포인트를 추가해."

**개발자 1: 주문 관리 API (16:00-18:00)**

관리자가 주문을 관리할 수 있는 API를 추가한다. 주문 목록 조회, 주문 상태 변경(배송 시작, 배송 완료), 주문 취소.

"AdminOrderController를 만들어줘. 주문 목록을 페이징과 필터링(상태, 날짜 범위, 사용자)으로 조회하고, 주문 상태를 변경할 수 있어야 해. 상태 변경 시 OrderStatusChanged 이벤트를 Kafka로 발행해서 사용자에게 이메일 알림을 보내도록."

```java
public class AdminOrderController {
    
    @GetMapping
    public ResponseEntity<Page<Order>> getOrders(
            @RequestParam(required = false) OrderStatus status,
            @RequestParam(required = false) @DateTimeFormat(iso = ISO.DATE) 
                LocalDate startDate,
            @RequestParam(required = false) @DateTimeFormat(iso = ISO.DATE) 
                LocalDate endDate,
            Pageable pageable) {
        
        OrderSearchCriteria criteria = OrderSearchCriteria.builder()
            .status(status)
            .startDate(startDate)
            .endDate(endDate)
            .build();
        
        Page<Order> orders = orderService.searchOrders(criteria, pageable);
        
        return ResponseEntity.ok(orders);
    }
    
    @PutMapping("/{id}/status")
    public ResponseEntity<Order> updateOrderStatus(
            @PathVariable UUID id,
            @RequestBody UpdateOrderStatusRequest request) {
        
        Order order = orderService.updateStatus(id, request.getStatus());
        
        // 이벤트 발행
        kafkaTemplate.send("order-events", 
            new OrderStatusChangedEvent(id, request.getStatus()));
        
        return ResponseEntity.ok(order);
    }
}
```

**개발자 2: 상품 상세 페이지와 추천 (13:15-16:00)**

상품 상세 페이지를 구현한다. "ProductDetailPage를 만들어줘. 상품 이미지, 이름, 설명, 가격, 재고 상태를 표시하고, 수량 선택 후 장바구니에 담기 버튼을 추가해. 하단에는 '유사한 상품' 섹션을 추가해서 pgvector 추천 결과를 보여줘."

```typescript
export function ProductDetailPage() {
  const { id } = useParams();
  const { data: product } = useProduct(id);
  const { data: similarProducts } = useSimilarProducts(id);
  const addToCart = useAddToCart();
  const [quantity, setQuantity] = useState(1);
  
  if (!product) return <Spinner />;
  
  return (
    <div className="container mx-auto py-8">
      <div className="grid grid-cols-2 gap-8">
        <div>
          <img 
            src={product.imageUrl} 
            alt={product.name}
            className="w-full rounded-lg"
          />
        </div>
        
        <div>
          <h1 className="text-3xl font-bold mb-4">{product.name}</h1>
          <p className="text-gray-600 mb-6">{product.description}</p>
          
          <div className="text-2xl font-bold mb-6">
            {formatCurrency(product.price)}
          </div>
          
          <div className="mb-6">
            <label className="block mb-2">수량</label>
            <div className="flex items-center gap-4">
              <Button 
                onClick={() => setQuantity(q => Math.max(1, q - 1))}
              >
                -
              </Button>
              <input 
                type="number" 
                value={quantity}
                onChange={(e) => setQuantity(parseInt(e.target.value))}
                className="w-20 text-center border rounded"
              />
              <Button 
                onClick={() => setQuantity(q => 
                  Math.min(product.stock, q + 1)
                )}
              >
                +
              </Button>
            </div>
            <p className="text-sm text-gray-600 mt-2">
              재고: {product.stock}개
            </p>
          </div>
          
          <Button 
            size="lg"
            className="w-full"
            onClick={() => addToCart.mutate({ 
              productId: product.id, 
              quantity 
            })}
            disabled={product.stock === 0}
          >
            {product.stock > 0 ? '장바구니에 담기' : '품절'}
          </Button>
        </div>
      </div>
      
      {similarProducts && similarProducts.length > 0 && (
        <div className="mt-16">
          <h2 className="text-2xl font-bold mb-6">유사한 상품</h2>
          <div className="grid grid-cols-4 gap-4">
            {similarProducts.map(p => (
              <ProductCard key={p.id} product={p} />
            ))}
          </div>
        </div>
      )}
    </div>
  );
}
```

**개발자 2: 주문 관리 UI (16:00-18:00)**

관리자용 주문 관리 페이지를 만든다. "OrderManagementPage를 만들어줘. 주문 목록을 테이블로 표시하고, 필터링(상태, 날짜 범위), 검색(주문 번호, 사용자 이메일) 기능을 추가해. 각 주문의 상태를 드롭다운으로 변경할 수 있어야 해."

```typescript
export function OrderManagementPage() {
  const [filters, setFilters] = useState<OrderFilters>({});
  const { data, isLoading } = useOrders(filters);
  const updateStatus = useUpdateOrderStatus();
  
  const columns = useMemo<ColumnDef<Order>[]>(() => [
    {
      accessorKey: 'orderNumber',
      header: '주문번호',
    },
    {
      accessorKey: 'user.email',
      header: '주문자',
    },
    {
      accessorKey: 'totalAmount',
      header: '총액',
      cell: ({ row }) => formatCurrency(row.original.totalAmount),
    },
    {
      accessorKey: 'status',
      header: '상태',
      cell: ({ row }) => (
        <Select 
          value={row.original.status}
          onChange={(status) => 
            updateStatus.mutate({ 
              orderId: row.original.id, 
              status 
            })
          }
        >
          <option value="PENDING">대기중</option>
          <option value="CONFIRMED">확인됨</option>
          <option value="SHIPPED">배송중</option>
          <option value="DELIVERED">배송완료</option>
          <option value="CANCELLED">취소됨</option>
        </Select>
      ),
    },
    {
      accessorKey: 'createdAt',
      header: '주문일시',
      cell: ({ row }) => formatDateTime(row.original.createdAt),
    },
  ], []);
  
  return (
    <AdminLayout>
      <div className="p-6">
        <h1 className="text-2xl font-bold mb-6">주문 관리</h1>
        
        <OrderFilters filters={filters} onChange={setFilters} />
        
        <DataTable columns={columns} data={data?.content ?? []} />
      </div>
    </AdminLayout>
  );
}
```

### 5.3 저녁 (18:00-19:00): 성능 모니터링과 회고

**성능 모니터링 설정 (18:00-18:30)**

Spring Boot Actuator와 Micrometer를 설정한다. Prometheus 메트릭을 노출하고, Grafana 대시보드를 준비한다.

"Spring Boot Actuator를 설정해줘. /actuator/health, /actuator/metrics, /actuator/prometheus 엔드포인트를 활성화하고, 커스텀 메트릭을 추가해. 주문 생성 시간, 상품 검색 응답 시간, 캐시 히트율을 측정해."

```java
public class MetricsConfig {
    
    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config()
            .commonTags("application", "ecommerce");
    }
    
    @Component
    public class OrderMetrics {
        private final Counter orderCreatedCounter;
        private final Timer orderCreationTimer;
        
        public OrderMetrics(MeterRegistry registry) {
            this.orderCreatedCounter = Counter.builder("orders.created")
                .description("Total orders created")
                .register(registry);
            
            this.orderCreationTimer = Timer.builder("orders.creation.time")
                .description("Order creation time")
                .register(registry);
        }
        
        public void recordOrderCreated() {
            orderCreatedCounter.increment();
        }
        
        public void recordOrderCreationTime(long milliseconds) {
            orderCreationTimer.record(milliseconds, TimeUnit.MILLISECONDS);
        }
    }
}
```

Grafana 대시보드 JSON을 준비한다. "Grafana 대시보드를 만들어줘. HTTP 요청 수, 응답 시간 분포, JVM 메모리 사용량, 데이터베이스 커넥션 풀, 캐시 히트율을 시각화해."

**회고 (18:30-19:00)**

Day 3는 기능 완성도를 높이는 날이었다. 관리자 기능, 캐싱, pgvector 추천까지 추가하면서 시스템이 프로덕션에 가까워졌다.

**Day 3 결과물:**
- ✅ 관리자 상품 관리 API 및 UI
- ✅ Redis 캐싱 (5-10분 TTL)
- ✅ pgvector 기반 상품 추천
- ✅ 주문 관리 API 및 UI
- ✅ 성능 모니터링 (Actuator, Prometheus)
- ✅ Grafana 대시보드

---

## 제6장: Day 4 - 마무리와 배포 (목요일)

### 6.1 오전 (9:00-12:00): 버그 수정과 테스트 보완

**스탠드업 (9:00-9:15)**

마지막 날이다. 오전에는 QA가 발견한 버그를 수정하고 테스트 커버리지를 높인다. 오후에는 배포 준비를 한다.

**QA 피드백 처리 (9:15-11:00)**

QA팀이 밤새 테스트한 결과를 공유한다. 발견된 이슈들:

1. 장바구니에 재고보다 많은 수량을 담을 수 있음
2. 주문 생성 후 재고가 즉시 반영되지 않음 (Kafka 지연)
3. 이미지 업로드 시 대용량 파일 처리 안 됨
4. 동시에 같은 상품을 주문하면 재고가 마이너스가 될 수 있음
5. 상품 검색 시 특수문자 입력 시 에러
6. 로그아웃 후에도 토큰이 유효함

개발자들이 나눠서 수정한다.

**개발자 1: 동시성 제어와 재고 관리 (9:15-11:00)**

가장 심각한 이슈는 4번이다. 동시성 제어가 필요하다.

"ProductService의 재고 차감 로직에 비관적 락을 추가해줘. @Lock(LockModeType.PESSIMISTIC_WRITE)를 사용해서 동시에 같은 상품을 주문할 때 재고가 정확히 차감되도록."

```java
public interface ProductRepository extends JpaRepository<Product, UUID> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdForUpdate(@Param("id") UUID id);
}

public class InventoryService {
    
    @Transactional
    public void reserveInventory(UUID orderId, List<OrderItem> items) {
        for (OrderItem item : items) {
            Product product = productRepository
                .findByIdForUpdate(item.getProductId())
                .orElseThrow(() -> 
                    new ProductNotFoundException(item.getProductId()));
            
            if (product.getStock() < item.getQuantity()) {
                throw new InsufficientStockException(
                    item.getProductId(), 
                    item.getQuantity()
                );
            }
            
            product.decreaseStock(item.getQuantity());
            productRepository.save(product);
        }
    }
}
```

통합 테스트로 검증한다. "동시에 100명이 같은 상품(재고 10개)을 주문할 때 정확히 10개만 판매되는지 테스트하는 동시성 테스트를 작성해줘. CountDownLatch를 사용해서 동시 실행을 시뮬레이션해."

```java
void concurrentOrderCreation_ShouldNotExceedStock() throws Exception {
    // given
    int stock = 10;
    int threadCount = 100;
    Product product = createProductWithStock(stock);
    
    ExecutorService executor = Executors.newFixedThreadPool(threadCount);
    CountDownLatch latch = new CountDownLatch(threadCount);
    
    AtomicInteger successCount = new AtomicInteger(0);
    AtomicInteger failureCount = new AtomicInteger(0);
    
    // when
    for (int i = 0; i < threadCount; i++) {
        executor.submit(() -> {
            try {
                latch.countDown();
                latch.await(); // 모두 동시에 시작
                
                orderService.createOrder(createOrderRequest(product.getId(), 1));
                successCount.incrementAndGet();
            } catch (InsufficientStockException e) {
                failureCount.incrementAndGet();
            } catch (Exception e) {
                // ignore
            }
        });
    }
    
    executor.shutdown();
    executor.awaitTermination(10, TimeUnit.SECONDS);
    
    // then
    assertThat(successCount.get()).isEqualTo(stock);
    assertThat(failureCount.get()).isEqualTo(threadCount - stock);
    
    Product updatedProduct = productRepository.findById(product.getId()).get();
    assertThat(updatedProduct.getStock()).isEqualTo(0);
}
```

토큰 무효화 문제를 해결한다. "로그아웃 시 토큰을 블랙리스트에 추가해줘. Redis에 토큰을 저장하고, TTL은 토큰의 남은 만료 시간으로 설정해. 인증 필터에서 블랙리스트를 체크하도록."

**개발자 2: 프론트엔드 버그 수정과 UX 개선 (9:15-11:00)**

프론트엔드 이슈들을 수정한다. 이미지 업로드 시 크기 제한, 특수문자 처리, 로딩 상태 개선 등.

"이미지 업로드 전에 클라이언트에서 크기를 체크해줘. 5MB를 초과하면 에러 메시지를 표시하고 업로드를 막아. 또한 업로드 전에 이미지를 리사이즈해서 서버 부하를 줄여."

```typescript
async function handleImageUpload(file: File) {
  // 크기 체크
  if (file.size > 5 * 1024 * 1024) {
    toast.error('이미지 크기는 5MB를 초과할 수 없습니다.');
    return;
  }
  
  // 이미지 리사이즈
  const resized = await resizeImage(file, 1200, 1200);
  
  // 업로드
  const formData = new FormData();
  formData.append('image', resized);
  
  const response = await uploadImage(formData);
  return response.data.url;
}

async function resizeImage(
  file: File, 
  maxWidth: number, 
  maxHeight: number
): Promise<Blob> {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement('canvas');
        let width = img.width;
        let height = img.height;
        
        if (width > height) {
          if (width > maxWidth) {
            height *= maxWidth / width;
            width = maxWidth;
          }
        } else {
          if (height > maxHeight) {
            width *= maxHeight / height;
            height = maxHeight;
          }
        }
        
        canvas.width = width;
        canvas.height = height;
        
        const ctx = canvas.getContext('2d')!;
        ctx.drawImage(img, 0, 0, width, height);
        
        canvas.toBlob((blob) => resolve(blob!), 'image/jpeg', 0.9);
      };
      img.src = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  });
}
```

로딩 상태를 개선한다. "모든 API 호출에 로딩 인디케이터를 추가하고, Skeleton UI를 사용해서 로딩 중에도 레이아웃이 흔들리지 않도록 해줘."

### 6.2 오전 후반 (11:00-12:00): 테스트 커버리지 향상

**테스트 커버리지 측정 (11:00-11:30)**

JaCoCo를 사용해서 테스트 커버리지를 측정한다. "JaCoCo Maven 플러그인을 추가하고, mvn test 실행 시 커버리지 리포트를 생성하도록 설정해줘. 커버리지가 70% 미만이면 빌드를 실패시켜."

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.70</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

커버리지 리포트를 확인하고 부족한 부분을 식별한다. Claude Code에게 "현재 테스트 커버리지 리포트를 분석하고, 커버되지 않은 중요한 로직을 찾아줘. 특히 비즈니스 로직과 에러 처리 부분을 우선순위로."

**누락된 테스트 추가 (11:30-12:00)**

Claude Code가 식별한 부분에 테스트를 추가한다. 예를 들어:
- OrderService의 보상 트랜잭션 로직
- JWT 토큰 검증 실패 케이스
- 재고 부족 시나리오
- 동시 주문 시나리오

이런 테스트들은 엣지 케이스를 다루므로 프로덕션 안정성에 중요하다.

### 6.3 오후 (13:00-18:00): 배포 준비와 문서화

**점심 후 스탠드업 (13:00-13:15)**

오후 계획을 확인한다. 프로덕션 배포 준비, API 문서 생성, 운영 가이드 작성, 최종 데모를 진행한다.

**PL: 프로덕션 환경 설정 (13:15-15:00)**

프로덕션용 Docker Compose 파일을 준비한다. 개발 환경과 다른 점:
- 환경 변수로 민감한 정보 관리
- 볼륨 마운트로 데이터 영속화
- 헬스체크 설정
- 리소스 제한
- 로그 수집 (Fluentd → Elasticsearch)

```yaml
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
  
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://${KAFKA_HOST}:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
    volumes:
      - kafka_data:/var/lib/kafka/data
  
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      kafka:
        condition: service_started
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/${DB_NAME}
      SPRING_DATASOURCE_USERNAME: ${DB_USER}
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_KAFKA_BOOTSTRAP_SERVERS: kafka:9092
      JWT_SECRET: ${JWT_SECRET}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
    logging:
      driver: fluentd
      options:
        tag: backend
  
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
    depends_on:
      - backend
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

volumes:
  postgres_data:
  redis_data:
  kafka_data:
```

GitLab CI를 프로덕션 배포까지 확장한다:

```yaml
stages:
  - test
  - build
  - deploy-dev
  - deploy-prod

# ... 이전 test, build 단계

deploy-production:
  stage: deploy-prod
  script:
    - docker-compose -f docker-compose.prod.yml pull
    - docker-compose -f docker-compose.prod.yml up -d
  only:
    - tags  # 태그가 푸시될 때만 프로덕션 배포
  when: manual  # 수동 승인 필요
  environment:
    name: production
    url: https://ecommerce.example.com
```

**개발자 1: API 문서 생성 (13:15-15:00)**

Springdoc OpenAPI가 자동 생성한 문서를 리뷰하고 보완한다. "각 API 엔드포인트에 @Operation, @ApiResponse 어노테이션을 추가해서 문서를 풍부하게 만들어줘. 예제 요청/응답도 추가해."

```java
public class ProductController {
    
    @GetMapping
    @Operation(
        summary = "상품 목록 조회",
        description = "상품 목록을 페이징과 필터링으로 조회합니다."
    )
    @ApiResponses({
        @ApiResponse(
            responseCode = "200",
            description = "조회 성공",
            content = @Content(
                schema = @Schema(implementation = Page.class)
            )
        ),
        @ApiResponse(
            responseCode = "400",
            description = "잘못된 요청",
            content = @Content(
                schema = @Schema(implementation = ErrorResponse.class)
            )
        )
    })
    public ResponseEntity<Page<Product>> getProducts(
            @Parameter(description = "페이지 번호", example = "0")
            @RequestParam(defaultValue = "0") int page,
            @Parameter(description = "페이지 크기", example = "20")
            @RequestParam(defaultValue = "20") int size,
            @Parameter(description = "검색 키워드")
            @RequestParam(required = false) String search) {
        // ...
    }
}
```

Swagger UI를 확인하고, PM과 QA가 API를 쉽게 테스트할 수 있는지 검증한다.

**개발자 2: 사용자 가이드 작성 (13:15-15:00)**

Claude Code에게 "사용자 가이드를 작성해줘. 회원가입, 상품 검색, 주문, 주문 조회 흐름을 스크린샷과 함께 설명하는 마크다운 문서를 만들어줘."

```markdown
# E-Commerce 플랫폼 사용자 가이드

## 1. 회원가입 및 로그인

### 1.1 회원가입
1. 홈페이지 우측 상단의 "회원가입" 버튼을 클릭합니다.
2. 이메일, 비밀번호, 이름을 입력합니다.
   - 이메일: 유효한 이메일 형식
   - 비밀번호: 8자 이상
3. "회원가입" 버튼을 클릭합니다.
4. 자동으로 로그인되어 대시보드로 이동합니다.

### 1.2 로그인
1. 홈페이지 우측 상단의 "로그인" 버튼을 클릭합니다.
2. 이메일과 비밀번호를 입력합니다.
3. "로그인" 버튼을 클릭합니다.

## 2. 상품 검색 및 필터링

### 2.1 상품 검색
1. 상단의 검색창에 키워드를 입력합니다.
2. Enter 키를 누르거나 검색 버튼을 클릭합니다.
3. 검색 결과가 표시됩니다.

### 2.2 필터링
1. 좌측 사이드바에서 카테고리를 선택합니다.
2. 가격 범위를 설정합니다.
3. 정렬 옵션(가격 낮은 순, 높은 순, 최신 순)을 선택합니다.

## 3. 장바구니와 주문

### 3.1 장바구니에 담기
1. 상품 상세 페이지에서 수량을 선택합니다.
2. "장바구니에 담기" 버튼을 클릭합니다.
3. 우측 상단의 장바구니 아이콘에 숫자가 표시됩니다.

### 3.2 주문하기
1. 우측 상단의 장바구니 아이콘을 클릭합니다.
2. 장바구니 페이지에서 상품 목록을 확인합니다.
3. 필요시 수량을 조정하거나 상품을 제거합니다.
4. "주문하기" 버튼을 클릭합니다.
5. 주문 확인 페이지에서 정보를 확인합니다.
6. "결제하기" 버튼을 클릭합니다.
7. 주문 완료 페이지에서 주문 번호를 확인합니다.

## 4. 주문 조회
1. 우측 상단의 사용자 메뉴를 클릭합니다.
2. "주문 내역"을 선택합니다.
3. 주문 목록이 표시됩니다.
4. 주문을 클릭하면 상세 정보를 확인할 수 있습니다.
```

**PL: 운영 가이드 작성 (15:00-17:00)**

운영팀을 위한 가이드를 작성한다. "운영 가이드를 작성해줘. 배포 방법, 모니터링, 로그 확인, 일반적인 문제 해결, 백업/복구 절차를 포함해."

~~~markdown
# E-Commerce 플랫폼 운영 가이드

## 1. 배포

### 1.1 프로덕션 배포
```bash
# 1. 저장소 클론
git clone https://gitlab.example.com/ecommerce/platform.git
cd platform

# 2. 환경 변수 설정
cp .env.example .env.prod
# .env.prod 파일을 편집하여 프로덕션 값 입력

# 3. Docker 이미지 빌드 및 배포
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml up -d

# 4. 헬스체크 확인
curl http://localhost:8080/actuator/health
```

### 1.2 롤백
```bash
# 이전 버전으로 롤백
docker-compose -f docker-compose.prod.yml down
git checkout <이전_태그>
docker-compose -f docker-compose.prod.yml up -d
```

## 2. 모니터링

### 2.1 Grafana 대시보드
- URL: http://monitoring.example.com:3000
- 주요 메트릭:
  - HTTP 요청 수 및 응답 시간
  - JVM 메모리 사용량
  - 데이터베이스 커넥션 풀
  - 캐시 히트율
  - Kafka 메시지 처리량

### 2.2 알림 설정
다음 조건에서 Slack 알림:
- HTTP 5xx 에러율 > 1%
- JVM 메모리 사용량 > 80%
- 데이터베이스 커넥션 풀 > 90%
- API 응답 시간 p99 > 2초

## 3. 로그 확인

### 3.1 애플리케이션 로그
```bash
# 백엔드 로그
docker-compose logs -f backend

# 특정 시간대 로그
docker-compose logs --since 2024-01-01T09:00:00 backend
```

### 3.2 Elasticsearch 쿼리
Kibana에서 로그 검색:
```
level: ERROR AND @timestamp > now-1h
```

## 4. 일반적인 문제 해결

### 4.1 데이터베이스 연결 실패
증상: 백엔드가 시작하지 않음, "Connection refused" 에러
해결:
```bash
# PostgreSQL 상태 확인
docker-compose ps postgres
docker-compose logs postgres

# 재시작
docker-compose restart postgres
```

### 4.2 재고 불일치
증상: 주문 가능한데 재고가 없음
해결:
```sql
-- 재고 확인
SELECT id, name, stock FROM products WHERE stock < 0;

-- 재고 수정
UPDATE products SET stock = <올바른_값> WHERE id = '<상품_ID>';
```

### 4.3 Kafka 메시지 지연
증상: 주문 후 상태가 업데이트되지 않음
해결:
```bash
# Kafka 컨슈머 그룹 확인
docker-compose exec kafka kafka-consumer-groups \
  --bootstrap-server localhost:9092 \
  --describe --group order-service

# Lag이 큰 경우 컨슈머 재시작
docker-compose restart backend
```

## 5. 백업 및 복구

### 5.1 데이터베이스 백업
```bash
# 백업 생성
docker-compose exec postgres pg_dump -U $DB_USER $DB_NAME > backup.sql

# S3에 업로드
aws s3 cp backup.sql s3://backups/db/backup-$(date +%Y%m%d).sql
```

### 5.2 복구
```bash
# S3에서 다운로드
aws s3 cp s3://backups/db/backup-20240101.sql backup.sql

# 복구
docker-compose exec -T postgres psql -U $DB_USER $DB_NAME < backup.sql
```

## 6. 긴급 연락처
- 백엔드 개발자: backend-team@example.com
- 프론트엔드 개발자: frontend-team@example.com
- PL: pl@example.com
- 인프라: infra@example.com
~~~

### 6.4 최종 데모와 인수 (17:00-18:00)

**PM 및 QA와 최종 데모 (17:00-17:30)**

전체 시스템을 PM과 QA에게 데모한다. 주요 사용자 플로우를 실제로 실행하며 보여준다:
1. 회원가입 → 로그인
2. 상품 검색 → 필터링 → 상품 상세
3. 장바구니에 담기 → 수량 조정
4. 주문하기 → 주문 완료
5. 주문 내역 조회
6. 관리자로 로그인 → 상품 등록 → 주문 관리

PM이 인수 기준을 확인한다:
- ✅ 모든 핵심 기능 작동
- ✅ E2E 테스트 통과
- ✅ API 문서 완성
- ✅ 사용자 가이드 제공
- ✅ 운영 가이드 제공
- ✅ 프로덕션 배포 준비 완료

**프로덕션 배포 (17:30-18:00)**

PL이 프로덕션에 배포한다. GitLab에서 태그를 생성하고, CI 파이프라인이 자동으로 실행되도록 한다.

```bash
git tag -a v1.0.0 -m "Initial production release"
git push origin v1.0.0
```

GitLab CI가 테스트 → 빌드 → 배포를 자동으로 진행한다. 팀 전체가 모니터링 대시보드를 보며 배포 진행 상황을 확인한다.

배포가 완료되면 smoke test를 실행한다:
```bash
# 헬스체크
curl https://ecommerce.example.com/actuator/health

# 주요 API 테스트
curl https://ecommerce.example.com/api/products

# E2E 테스트 (프로덕션 환경)
npm run test:e2e:prod
```

모든 것이 녹색이면 성공이다.

### 6.5 저녁 (18:00-19:00): 최종 회고

**프로젝트 회고 (18:00-19:00)**

4일간의 여정을 돌아본다. 팀 전체가 모여 다음 질문에 답한다:

**1. 무엇을 달성했는가?**
- 완전히 작동하는 E-Commerce 플랫폼
- 사용자 인증, 상품 카탈로그, 장바구니, 주문, 관리자 기능
- Redis 캐싱, Kafka 이벤트, pgvector 추천
- 포괄적인 테스트 (단위, 통합, E2E)
- CI/CD 파이프라인
- 프로덕션 배포

전통적인 개발이었다면 2-3주는 걸렸을 것이다. Claude Code 덕분에 4일 만에 가능했다.

**2. 무엇이 잘 작동했는가?**
- TDD 사이클: 테스트가 안전망 역할을 했다
- API-First 설계: 프론트엔드와 백엔드 병렬 개발
- Claude Code: 보일러플레이트를 빠르게 생성
- 페어 프로그래밍: 한 명은 Claude와 대화, 다른 한 명은 리뷰
- 일일 회고: 문제를 빨리 발견하고 수정

**3. 무엇을 개선할 수 있었는가?**
- 처음부터 E2E 테스트를 더 많이 작성
- 보안 검토를 더 일찍
- 성능 테스트 추가 (부하 테스트, 스트레스 테스트)
- 더 명확한 에러 메시지

**4. 배운 교훈**
- AI는 속도를 제공하지만 방향은 인간이 제공한다
- TDD는 AI 시대에 더욱 중요하다
- 검증 가능한 작은 단계로 진행하라
- 코드 리뷰는 여전히 필수다
- 문서화를 미루지 마라

**Day 4 결과물:**
- ✅ 모든 버그 수정
- ✅ 테스트 커버리지 70% 이상
- ✅ API 문서 (Swagger UI)
- ✅ 사용자 가이드
- ✅ 운영 가이드
- ✅ 프로덕션 배포 완료

---

## 제7장: 바이브 코딩 베스트 프랙티스

### 7.1 효과적인 Claude Code 활용법

**프롬프트 작성 원칙**

명확하고 구체적으로 요청한다. "API를 만들어줘"보다 "Spring Boot로 사용자 인증 REST API를 만들어줘. POST /api/auth/register 엔드포인트에서 이메일, 비밀번호, 이름을 받아 사용자를 생성하고, JWT 토큰을 반환해. 이메일 중복 체크를 하고, 비밀번호는 BCrypt로 해싱해."

컨텍스트를 제공한다. 프로젝트의 아키텍처, 사용 중인 라이브러리, 코딩 컨벤션을 설명한다. `.claude/context.md` 파일을 활용한다.

단계적으로 진행한다. 한 번에 모든 것을 요구하지 말고, TDD 사이클을 따라 테스트 → 구현 → 리팩터링 순서로 진행한다.

**리뷰와 검증**

Claude가 생성한 코드를 무조건 신뢰하지 않는다. 특히:
- 보안: SQL Injection, XSS, 인증/권한
- 성능: N+1 쿼리, 불필요한 반복
- 비즈니스 로직: 엣지 케이스, 예외 처리
- 테스트: 의미 있는 테스트인가, 아니면 형식적인가

코드 리뷰 세션을 갖는다. 두 개발자가 서로의 코드를 리뷰하고, Claude가 생성한 부분을 특히 주의 깊게 본다.

**지속적 학습**

Claude Code 세션을 기록한다. Fernandez처럼 트랜스크립트를 저장하고, 중요한 결정과 학습을 추출한다. 나중에 비슷한 문제를 만날 때 참고할 수 있다.

실패를 문서화한다. Claude가 잘못된 코드를 생성했을 때, 왜 잘못되었는지, 어떻게 수정했는지 기록한다. 팀 전체가 같은 실수를 반복하지 않도록.

### 7.2 TDD와 AI의 시너지

**왜 TDD가 중요한가**

Fernandez가 강조했듯이, AI가 수천 줄의 코드를 생성할 때 테스트는 개발자를 루프 안에 유지하는 강제 기능이다. 테스트를 작성하려면 요구사항을 이해해야 하고, 테스트를 검증하려면 구현을 이해해야 한다.

테스트는 안전망이다. AI가 생성한 코드를 리팩터링할 때, 테스트가 녹색을 유지하면 기능이 깨지지 않았다는 것을 확신할 수 있다.

**TDD 사이클을 빠르게 돌리기**

Claude Code와 함께하는 TDD는 전통적인 TDD보다 훨씬 빠르다:
1. Red (30초): Claude에게 테스트 작성 요청
2. Green (1-2분): Claude에게 최소 구현 요청
3. Refactor (1-2분): Claude에게 리팩터링 요청

전체 사이클이 5분 이내다. 전통적으로는 20-30분 걸렸을 것이다.

**테스트 품질 유지**

Claude가 생성한 테스트가 의미 있는지 확인한다. 다음을 체크:
- Given-When-Then 구조가 명확한가?
- 엣지 케이스를 다루는가?
- 테스트 이름이 의도를 명확히 전달하는가?
- 모킹이 적절한가? (과도한 모킹은 피한다)
- 실패 메시지가 도움이 되는가?

의미 없는 테스트는 제거한다. 100% 커버리지가 목표가 아니라, 의미 있는 테스트로 핵심 로직을 보호하는 것이 목표다.

### 7.3 팀 협업 패턴

**페어 프로그래밍의 재정의**

전통적 페어: Driver(타이핑) + Navigator(방향)  
AI 시대 페어: Prompter(Claude와 대화) + Reviewer(실시간 검토)

Prompter는 Claude에게 무엇을 만들지 설명하고, Reviewer는 생성된 코드를 즉시 검토한다. 이것은 전통적 페어보다 효율적이다. Reviewer가 문제를 발견하면 즉시 Prompter에게 알리고, Prompter가 Claude에게 수정을 요청한다.

30-60분마다 역할을 바꾼다. 둘 다 Claude와 대화하는 법을 익히고, 둘 다 리뷰 능력을 키워야 한다.

**비동기 협업**

GitLab의 Merge Request를 활용한다. 각 개발자가 feature 브랜치에서 작업하고, MR을 생성하면 다른 개발자가 리뷰한다.

Claude Code 세션 트랜스크립트를 MR 설명에 포함한다. 리뷰어가 왜 이런 결정을 했는지 컨텍스트를 이해할 수 있다.

**지식 공유**

일일 회고에서 배운 것을 공유한다. "오늘 Claude에게 이렇게 요청했더니 이런 결과가 나왔다" 같은 실용적인 팁을 나눈다.

프로젝트 위키를 만든다. 효과적인 프롬프트 패턴, 흔한 실수, 유용한 Claude Code 명령어 등을 문서화한다.

### 7.4 피해야 할 함정

**과도한 의존**

Claude Code가 강력하지만 만능은 아니다. 다음은 여전히 인간이 해야 한다:
- 아키텍처 결정
- 비즈니스 로직 설계
- 보안 정책 수립
- 성능 요구사항 정의
- 사용자 경험 설계

Claude에게 이런 것을 물어볼 수는 있지만, 최종 결정은 인간이 한다.

**검증 없는 복사-붙여넣기**

Claude가 생성한 코드를 이해하지 못한 채 그대로 사용하지 않는다. 각 줄을 읽고 이해한다. 특히:
- 외부 라이브러리 사용: 라이선스 확인, 보안 취약점 확인
- 알고리즘: 시간/공간 복잡도 이해
- 데이터베이스 쿼리: 성능 영향 평가

**테스트 부재**

"빨리 만들어야 하니까 테스트는 나중에"라는 함정에 빠지지 않는다. 테스트 없는 빠른 개발은 결국 버그로 느려진다.

Fernandez의 조언을 기억하라: "TDD는 AI 보조 개발에서 선택이 아니라 필수다."

**문서화 지연**

"코드가 문서다"라는 말은 틀렸다. 특히 AI가 생성한 코드는 더욱 그렇다. 왜 이런 결정을 했는지, 어떤 트레이드오프가 있었는지 문서화한다.

ADR, 코드 주석, README를 성실히 작성한다. 미래의 당신과 팀원이 고마워할 것이다.

---

## 결론: 4일 프로젝트의 교훈

이 가이드는 SpringBoot와 React로 4일 만에 프로덕션 수준의 E-Commerce 플랫폼을 만드는 구체적인 방법을 제시했다. Obie Fernandez의 Nexus 프로젝트에서 배운 원칙 - TDD, API-First 설계, 단계적 접근, 지속적 검증 - 을 실전에 적용했다.

**핵심 교훈:**

1. **AI는 레버리지다.** Claude Code는 생산성을 2-3배 높일 수 있지만, 그것을 효과적으로 사용하는 법을 배워야 한다.

2. **TDD는 안전망이다.** 수천 줄의 코드가 몇 분 만에 생성될 때, 테스트는 당신이 올바른 방향으로 가고 있다는 것을 확신시켜준다.

3. **방향은 인간이 제공한다.** 아키텍처, 비즈니스 로직, 사용자 경험은 여전히 인간의 판단이 필요하다.

4. **검증은 필수다.** Claude가 생성한 코드를 맹목적으로 신뢰하지 않는다. 리뷰하고, 테스트하고, 이해한다.

5. **문서화를 미루지 마라.** 결정의 맥락, 트레이드오프, 배운 교훈을 기록한다.

**다음 단계:**

이 가이드는 시작일 뿐이다. 실제로 4일 프로젝트를 진행하면서 자신만의 패턴을 발견할 것이다. 팀의 강점, 도메인의 특성, 기술 스택의 제약에 맞게 조정하라.

가장 중요한 것은 실험하고, 배우고, 공유하는 것이다. AI 보조 개발은 아직 초기 단계다. 우리 모두가 베스트 프랙티스를 함께 만들어가고 있다.

Fernandez의 말로 마무리한다: **"당신의 원칙을 가지고 오라. 당신의 테스트를 가지고 오라. 당신의 판단을 가지고 오라. AI는 속도를 제공한다. 당신은 방향을 제공한다. 함께, 당신은 혼자서는 만들 수 없는 것을 만든다."**

이제 시작할 시간이다. 월요일 아침, 당신의 팀과 함께.

---

**문서 작성일: 2026-01-10**
