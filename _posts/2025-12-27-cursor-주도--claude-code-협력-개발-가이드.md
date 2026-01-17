---
title: "Cursor 주도 Claude Code 협력 개발 가이드"
date: 2025-12-27 09:20:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  Developer,  vibe-coding,  claude-code,  Cursor,  Guide,  Claude.write]
---



**두 AI의 강점을 결합한 최적의 개발 워크플로우**


## 서론: 왜 두 가지 도구를 함께 사용하는가

### 단일 도구의 한계

**Cursor만 사용할 때:**
- 장점: 빠른 코드 작성
- 장점: 실시간 자동완성
- 단점: 깊이 있는 아키텍처 검증 부족
- 단점: 복잡한 디버깅에서 한계

**Claude Code만 사용할 때:**
- 장점: 깊이 있는 분석과 설계
- 장점: 체계적인 문서화
- 단점: IDE 통합 부족
- 단점: 실시간 반복 개발 불편

### 협력의 시너지

**Cursor + Claude Code 조합:**
- Cursor로 빠른 프로토타이핑
- Claude Code로 심층 검증
- Claude Code로 문서화
- Cursor로 즉각적인 수정
- 최고의 속도와 품질 달성

---

## Part 1: 역할 분담 매트릭스

### Cursor의 주요 역할

#### 1. 실시간 코드 작성 (Driver)

**담당 작업:**
- 컴포넌트 구현
- API 엔드포인트 작성
- UI 레이아웃 개발
- 빠른 버그 수정
- 반복적인 CRUD 로직

**강점:**
- IDE 내에서 즉시 실행 가능
- 파일 컨텍스트 자동 유지
- 자동완성으로 빠른 작성
- 실시간 피드백

**사용 모드:**
- **Composer 모드**: 여러 파일 동시 수정
- **Chat 모드**: 빠른 질문과 수정
- **Inline 편집**: 코드 라인 단위 수정

#### 2. 프로토타이핑 (Rapid Development)

**담당 작업:**
- MVP 빠르게 만들기
- UI 목업 구현
- 기능 검증용 코드
- A/B 테스트 코드

**예시 시나리오:**

```
[Cursor에서]
사용자: "주식 검색 바를 만들어줘. 자동완성 기능 포함, 디바운싱 적용"
Cursor: [즉시 SearchBar 컴포넌트 생성 + 구현]
```

#### 3. 즉각적인 리팩토링

**담당 작업:**
- 변수명 변경
- 함수 추출
- 코드 포맷팅
- import 정리

**강점:**
- 전체 프로젝트 컨텍스트 파악
- 자동 리팩토링 제안
- 영향 범위 자동 추적

### Claude Code의 주요 역할

#### 1. 아키텍처 설계 및 검증 (Architect)

**담당 작업:**
- 시스템 설계 리뷰
- 기술 스택 결정 지원
- 확장성 검증
- 보안 아키텍처 검토

**강점:**
- 다양한 아키텍처 패턴 지식
- 장단점 비교 분석
- 미래 확장성 고려
- 트레이드오프 설명

**예시 시나리오:**

```
[Claude Code에서]
사용자: "현재 모놀리식 구조를 마이크로서비스로 전환하려고 해. 어떻게 접근해야 할까?"
Claude: [상세한 분석과 단계별 마이그레이션 계획 제시]
```

#### 2. 심층 코드 리뷰 (Reviewer)

**담당 작업:**
- 보안 취약점 분석
- 성능 병목 지점 파악
- 알고리즘 최적화 제안
- 베스트 프랙티스 검증

**강점:**
- 다각도 분석
- 근거 있는 개선 제안
- 잠재적 문제 조기 발견
- 명확한 설명

#### 3. 문서화 및 지식 관리 (Documenter)

**담당 작업:**
- README 작성
- API 문서 생성
- 아키텍처 다이어그램
- 온보딩 가이드
- 트러블슈팅 문서

**강점:**
- 체계적인 구조화
- 상세하고 명확한 설명
- 다양한 독자 고려
- 예제 코드 포함

#### 4. 복잡한 문제 해결 (Problem Solver)

**담당 작업:**
- 복잡한 버그 디버깅
- 알고리즘 설계
- 성능 최적화
- 레거시 코드 분석

**강점:**
- 단계별 분석
- 여러 해결책 제시
- 장단점 비교
- 테스트 케이스 생성

---

## Part 2: 통합 워크플로우

### 3단계 협력 사이클

```
Phase 1: Cursor로 빠르게 구현
         ↓
Phase 2: Claude Code로 검증 및 개선
         ↓
Phase 3: Cursor로 수정 적용
```

### 실전 시나리오: 주식 포트폴리오 기능 개발

#### Phase 0: 기획 (Claude Code 주도)

**Claude Code에서 수행:**

프롬프트 예시:

```markdown
주식 포트폴리오 관리 기능을 만들려고 해.

요구사항:
- 사용자가 보유 주식 등록
- 실시간 수익률 계산
- 포트폴리오 분석 차트

다음을 작성해줘:
1. 기능 명세서 (PRD)
2. 기술 요구사항 (TRD)
3. 데이터베이스 스키마 설계
4. API 엔드포인트 설계
5. 프론트엔드 컴포넌트 구조
```

**산출물:**
- PORTFOLIO_PRD.md
- PORTFOLIO_TRD.md
- database_schema.sql
- API_SPEC.md
- COMPONENT_STRUCTURE.md

**왜 Claude Code인가?**
- 체계적인 문서 작성에 강함
- 다양한 관점에서 요구사항 분석
- 빠뜨린 부분 발견
- 명확한 구조화

#### Phase 1: 빠른 구현 (Cursor 주도)

**Cursor Composer 모드에서:**

프롬프트 예시:

```

위 명세를 바탕으로 포트폴리오 기능을 구현해줘:

백엔드:
- PortfolioController.java
- PortfolioService.java
- PortfolioRepository.java
- Portfolio.java (Entity)

프론트엔드:
- PortfolioList.tsx
- PortfolioForm.tsx
- PortfolioChart.tsx

한 번에 모든 파일 생성하고, 기본적인 CRUD 로직 포함해줘.
```

**Cursor의 작업:**

1. 모든 파일 자동 생성
2. import 자동 추가
3. 타입 정의 자동 완성
4. 기본 로직 구현

**진행 시간:** 5-10분

**왜 Cursor인가?**
- 여러 파일 동시 생성/수정
- IDE 컨텍스트 자동 유지
- 즉시 실행 및 테스트 가능
- 빠른 반복 개발

#### Phase 2: 심층 검증 (Claude Code 주도)

**생성된 코드를 Claude Code에 복사:**

프롬프트 예시:

```markdown
다음은 Cursor로 생성한 포트폴리오 기능 코드야:

[PortfolioController.java 전체 코드]
[PortfolioService.java 전체 코드]
[PortfolioList.tsx 전체 코드]

다음 관점에서 깊이 있게 리뷰해줘:

1. 보안 검증
   - SQL 인젝션 가능성
   - XSS 취약점
   - 인증/인가 확인
   - 민감 데이터 노출

2. 성능 분석
   - N+1 쿼리 문제
   - 불필요한 데이터 로딩
   - 캐싱 전략
   - 프론트엔드 렌더링 최적화

3. 코드 품질
   - SOLID 원칙 준수
   - 에러 처리 적절성
   - 테스트 가능성
   - 가독성

4. 비즈니스 로직
   - 수익률 계산 정확성
   - Edge case 처리
   - 데이터 정합성

각 문제점에 대해:
- 심각도 (Critical/High/Medium/Low)
- 구체적인 문제 설명
- 개선된 코드 제시
- 테스트 케이스
```

**Claude Code의 분석 결과 예시:**

```markdown
# 코드 리뷰 리포트

## Critical Issues

### 1. SQL 인젝션 취약점
파일: PortfolioRepository.java, line 45

문제:
String query = "SELECT * FROM portfolio WHERE user_id = " + userId;

위험:
- userId를 직접 문자열 결합하여 SQL 인젝션 가능
- 악의적 사용자가 다른 사용자 데이터 접근 가능

개선 코드:
List<Portfolio> findByUserId(@Param("userId") Long userId);

테스트 케이스:
void testSqlInjectionPrevention() {
    String maliciousInput = "1 OR 1=1";
    // 예외 발생 또는 빈 결과 확인
}

## High Priority Issues

### 2. N+1 쿼리 문제
파일: PortfolioService.java, line 67

문제: 포트폴리오 목록 조회 시 각 항목마다 주식 가격 조회

영향:
- 100개 포트폴리오 → 101번의 DB 쿼리
- 응답 시간 느림 (평균 5초+)

개선 코드:
List<Portfolio> findByUserIdWithStock(@Param("userId") Long userId);

성능 측정:
- Before: 평균 5.2초
- After: 평균 0.3초 (17배 개선)
```

**왜 Claude Code인가?**
- 깊이 있는 다각도 분석
- 보안/성능/품질 종합 검토
- 구체적인 개선 방안 제시
- 테스트 케이스까지 생성

#### Phase 3: 개선 적용 (Cursor 주도)

**Claude Code의 리뷰 결과를 Cursor로 가져옴:**

**Cursor Chat에서:**

```

다음 보안 이슈를 수정해줘:

[Claude Code가 지적한 SQL 인젝션 문제 설명]
[개선 코드]

위 개선사항을 적용하고, 관련된 모든 파일도 함께 업데이트해줘.
```

**Cursor의 작업:**

1. 문제 코드 자동 찾기
2. 개선 코드 적용
3. 관련 파일 자동 수정
4. import 자동 업데이트
5. 즉시 컴파일 확인

**추가 개선 (Cursor Chat):**

```
N+1 쿼리 문제를 해결해줘.
JOIN FETCH를 사용해서 한 번에 로딩하도록.
```

**왜 Cursor인가?**
- 빠른 코드 수정
- 영향받는 파일 자동 탐지
- 즉시 테스트 가능
- 반복 수정 용이

---

## Part 3: 실전 협력 패턴

### 패턴 1: 기능 개발 사이클

**5단계 프로세스:**

1. **Claude Code: 설계 및 명세서 작성**
   - PRD, TRD 작성
   - API 명세 정의
   - 데이터 모델 설계

2. **Cursor: 빠른 구현**
   - Composer로 전체 스켈레톤 생성
   - 기본 CRUD 로직 구현
   - UI 컴포넌트 작성

3. **Claude Code: 검증 및 개선**
   - 보안 검증
   - 성능 분석
   - 코드 품질 리뷰
   - 개선 방안 제시

4. **Cursor: 개선사항 적용**
   - 문제점 수정
   - 최적화 적용
   - 테스트 작성

5. **Claude Code: 문서화**
   - README 업데이트
   - API 문서 생성
   - 코드 주석 개선

### 패턴 2: 버그 수정 플로우

**간단한 버그:**
1. 버그 발견
2. Cursor로 즉시 수정
3. 완료

**복잡한 버그:**
1. 버그 발견
2. Claude Code로 원인 분석
3. Cursor로 해결책 적용
4. Claude Code로 검증 및 문서화

### 패턴 3: 리팩토링 전략

**Step 1: Claude Code로 전략 수립**

프롬프트:

```markdown
현재 프로젝트의 기술 부채를 분석하고 리팩토링 우선순위를 제시해줘:

프로젝트 구조:
[디렉토리 트리 붙여넣기]

주요 파일:

분석 요청:
1. 코드 중복 분석
2. 복잡도가 높은 메서드 식별
3. 아키텍처 개선 포인트
4. 리팩토링 우선순위 (ROI 기준)
5. 단계별 실행 계획
```

**Step 2: Cursor로 자동 리팩토링**

```

Claude Code가 제안한 리팩토링 적용:

1. extractAuthenticationLogic() 메서드 추출
2. 중복 코드 제거
3. 복잡한 if문을 전략 패턴으로 변경

안전하게 리팩토링해줘. 모든 테스트가 통과하는지 확인.
```

**Step 3: Claude Code로 검증**

```markdown
리팩토링된 코드를 검토해줘:

[리팩토링된 UserService.java]

확인 사항:
1. 기능적 동등성 (기존과 동일하게 동작)
2. 코드 품질 개선 확인
3. 새로운 문제 발생 여부
4. 추가 개선 가능성
```

### 패턴 4: 성능 최적화

**Claude Code: 병목 지점 분석**

```markdown
다음 API가 너무 느려:

GET /api/portfolio/summary
- 평균 응답 시간: 5.2초
- 목표: 0.5초 이하

관련 코드:
[PortfolioService.java]
[StockPriceService.java]

성능 분석:
1. 프로파일링 결과 해석
2. 병목 지점 식별
3. 최적화 전략 (여러 옵션)
4. 각 전략의 예상 효과
5. 우선순위 추천
```

**Cursor: 최적화 적용**

```

Claude Code가 제안한 최적화 적용:

1. 캐싱 레이어 추가 (Redis)
2. 데이터베이스 쿼리 최적화
3. 병렬 처리 도입

성능 측정 코드도 함께 추가해줘.
```

**Claude Code: 결과 분석 및 문서화**

```markdown
최적화 전후 성능 비교:

Before:
- 응답 시간: 5.2초
- DB 쿼리: 50회
- 메모리: 500MB

After:
- 응답 시간: 0.3초
- DB 쿼리: 3회
- 메모리: 100MB

성능 개선 보고서 작성:
1. 문제 정의
2. 해결 방법
3. 결과 분석
4. 향후 모니터링 방안
```

---

## Part 4: 도구별 최적 사용 시나리오

### Cursor를 주도적으로 사용할 때

#### 시나리오 1: UI 컴포넌트 빠르게 만들기

**상황:**
- 새로운 대시보드 페이지 필요
- 디자인 시안은 있음
- 빠르게 프로토타입 필요

**Cursor Composer:**

```
다음 디자인을 구현해줘:

레이아웃:
- 왼쪽: 사이드바 (네비게이션)
- 가운데: 메인 콘텐츠 (포트폴리오 카드 그리드)
- 오른쪽: 위젯 패널 (최근 거래, 뉴스)

스타일:
- Tailwind CSS 사용
- 다크 모드 지원
- 반응형 디자인

컴포넌트:
- DashboardLayout.tsx
- PortfolioGrid.tsx
- RecentTrades.tsx
- NewsWidget.tsx

모든 컴포넌트를 한 번에 생성하고, 기본 데이터로 미리보기 가능하게 해줘.
```

**결과:**
- 5분 내 전체 레이아웃 완성
- 즉시 브라우저에서 확인 가능
- 실시간으로 수정 반복

#### 시나리오 2: API 엔드포인트 일괄 생성

**Cursor Chat + Composer:**

```

위 API 명세에 있는 모든 엔드포인트를 구현해줘:

1. /api/stocks/* (5개)
2. /api/portfolio/* (7개)
3. /api/watchlist/* (4개)

각 엔드포인트:
- Controller 메서드
- Service 레이어
- DTO 클래스
- 입력 검증
- 에러 처리

한 번에 모두 생성해줘.
```

**장점:**
- 일관된 코드 스타일
- 중복 코드 최소화
- 파일 간 자동 연결
- 즉시 테스트 가능

#### 시나리오 3: 긴급 버그 수정

**Cursor Chat:**

```

이 라인에서 NullPointerException 발생.
원인 찾아서 즉시 수정해줘.

재현 방법:
1. 로그인 시도
2. 소셜 로그인 (Google)
3. 첫 로그인 시 발생
```

**Cursor 작업:**

1. 해당 라인으로 즉시 이동
2. 문제 코드 하이라이트
3. null 체크 추가
4. 관련 테스트 실행
5. 수정 확인

**시간:** 2-3분

### Claude Code를 주도적으로 사용할 때

#### 시나리오 1: 아키텍처 리뷰

**상황:**
- 시스템이 느려짐
- 사용자 증가로 확장 필요
- 근본적인 개선 필요

**Claude Code 프롬프트:**

```markdown
현재 시스템 아키텍처를 검토하고 확장성 개선 방안을 제시해줘:

현재 구조:
- 모놀리식 Spring Boot 애플리케이션
- 단일 PostgreSQL 데이터베이스
- 모든 로직이 동기 처리
- 파일 저장: 로컬 디스크

문제:
- 동시 접속자 100명 이상 시 응답 느림
- 데이터베이스 CPU 사용률 90%
- 이미지 업로드 시 메모리 부족

목표:
- 1000명 동시 접속 지원
- 응답 시간 1초 이하 유지
- 99.9% 가용성

다음을 제시해줘:
1. 현재 아키텍처의 병목 지점 분석
2. 단계별 개선 로드맵 (3단계)
3. 각 단계별 기술 스택 추천
4. 마이그레이션 전략
5. 비용 추정
6. 리스크 분석
```

**왜 Claude Code인가?**
- 복잡한 시스템 설계 능력
- 다양한 옵션 비교 분석
- 단계별 구체적 계획
- 비용과 리스크 고려

#### 시나리오 2: 복잡한 알고리즘 설계

**Claude Code 프롬프트:**

```markdown
주식 포트폴리오 리밸런싱 알고리즘을 설계해줘:

요구사항:
1. 목표 자산 배분 비율 유지
2. 거래 비용 최소화
3. 세금 효율적인 매도 순서
4. 최소 거래 금액 제약

제약조건:
- 단일 종목 거래는 최소 10만원 이상
- 총 거래 비용은 리밸런싱 필요 금액의 2% 이하
- 손실 종목 우선 매도 (세금 절감)

다음을 제시해줘:
1. 알고리즘 설계 (수도코드)
2. 복잡도 분석
3. 테스트 케이스
4. Java 구현 코드
5. 최적화 방안
```

**왜 Claude Code인가?**
- 복잡한 로직의 명확한 설명
- 여러 제약조건 고려
- 수학적 계산 검증
- 다양한 엣지 케이스 고려

#### 시나리오 3: 보안 감사

**Claude Code 프롬프트:**

```markdown
전체 프로젝트의 보안 감사를 수행해줘:

점검 대상:
1. 인증/인가 로직
2. SQL 인젝션 취약점
3. XSS 취약점
4. CSRF 보호
5. 민감 데이터 처리
6. API 보안
7. 의존성 취약점

코드:
[주요 파일들 붙여넣기]

산출물:
1. 보안 취약점 리포트 (CVSS 점수 포함)
2. 각 취약점의 공격 시나리오
3. 수정 방법 (코드 예시)
4. 보안 체크리스트
5. 자동화 테스트 코드
```

**왜 Claude Code인가?**
- 종합적인 보안 지식
- 다양한 공격 벡터 분석
- 구체적인 수정 방안
- 체계적인 문서화

---

## Part 5: 협력 최적화 팁

### 팁 1: 컨텍스트 공유하기

**문제:** Cursor에서 작업한 내용을 Claude Code에게 설명하기 번거로움

**해결책:**

**Cursor에서 작업 후:**

1. 주요 변경사항을 요약 요청
   - "지금까지의 변경사항을 요약해줘"

2. Claude Code에 전달할 컨텍스트 생성
   - "Claude Code로 코드 리뷰를 받으려고 해. 필요한 컨텍스트를 정리해줘"
   - 변경된 파일 목록
   - 주요 로직 설명
   - 의도한 동작

3. 생성된 컨텍스트를 Claude Code에 복사

### 팁 2: 파일 기반 협력

**프로젝트 구조:**

```
project/
├── docs/
│   ├── PRD.md              (Claude Code 작성)
│   ├── TRD.md              (Claude Code 작성)
│   ├── API_SPEC.md         (Claude Code 작성)
│   └── REVIEW_REPORT.md    (Claude Code 작성)
├── src/
│   └── ...                 (Cursor 구현)
└── tests/
    └── ...                 (Cursor 작성, Claude Code 보완)
```

**워크플로우:**

1. Claude Code로 docs/ 작성
2. Cursor에서 @docs/TRD.md 참조하여 구현
3. 구현 완료 후 Claude Code에게 리뷰 요청
4. Claude Code가 docs/REVIEW_REPORT.md 작성
5. Cursor에서 리포트 참고하여 수정

### 팁 3: 체크리스트 활용

**개발 단계별 체크리스트:**

**Phase 1: 설계 (Claude Code)**
- [ ] PRD 작성
- [ ] TRD 작성
- [ ] API 명세 작성
- [ ] 데이터 모델 설계
- [ ] 컴포넌트 구조 설계

**Phase 2: 구현 (Cursor)**
- [ ] 백엔드 API 구현
- [ ] 프론트엔드 UI 구현
- [ ] 기본 테스트 작성
- [ ] 로컬에서 동작 확인

**Phase 3: 검증 (Claude Code)**
- [ ] 보안 리뷰
- [ ] 성능 분석
- [ ] 코드 품질 검토
- [ ] 개선사항 문서화

**Phase 4: 개선 (Cursor)**
- [ ] 지적사항 수정
- [ ] 테스트 보완
- [ ] 문서 업데이트

**Phase 5: 완료 (Claude Code)**
- [ ] 최종 검증
- [ ] README 업데이트
- [ ] 릴리즈 노트 작성

### 팁 4: 버전 관리 전략

**브랜치 전략:**

```
main
  ├── feature/portfolio-management
  │     ├── docs (Claude Code 작업)
  │     └── impl (Cursor 작업)
  └── feature/stock-search
        ├── docs
        └── impl
```

**커밋 메시지 컨벤션:**

```bash
# Claude Code 작업
docs: Add portfolio management PRD
docs: Update API specification for portfolio

# Cursor 작업
feat: Implement portfolio CRUD API
feat: Add portfolio list component

# 협력 작업
refactor: Apply security improvements from code review
perf: Optimize queries based on performance analysis
```

---

## Part 6: 실전 프로젝트 전체 플로우

### 완전한 기능 개발 예시: 주식 알림 시스템

#### Week 1: 설계 단계 (Claude Code 주도)

**Day 1-2: 요구사항 분석**

Claude Code 프롬프트:

```markdown
주식 가격 알림 시스템을 만들려고 해:

기능:
1. 사용자가 특정 종목의 목표가 설정
2. 실시간으로 주가 모니터링
3. 목표가 도달 시 알림 (푸시, 이메일)
4. 알림 히스토리 조회

기술 고려사항:
- 실시간성 (1분 이내)
- 확장성 (10만 사용자)
- 신뢰성 (알림 누락 최소화)

다음을 작성해줘:
1. 상세 PRD
2. 시스템 아키텍처 설계
3. 기술 스택 추천
4. 데이터 모델
5. API 설계
6. 주요 도전과제와 해결방안
```

**산출물:**
- docs/ALERT_SYSTEM_PRD.md
- docs/ALERT_SYSTEM_ARCHITECTURE.md
- docs/ALERT_SYSTEM_API.md
- docs/database/alert_schema.sql

**Day 3-4: 기술 검증**

```markdown
알림 시스템의 핵심 기술 결정을 검증해줘:

옵션 1: WebSocket + Server-Sent Events
옵션 2: Polling + Push Notification Service
옵션 3: Message Queue + Worker Pool

각 옵션에 대해:
1. 장단점 비교
2. 확장성 분석
3. 비용 추정
4. 구현 복잡도
5. 추천 의견 (근거 포함)
```

#### Week 2: 프로토타입 (Cursor 주도)

**Day 1-3: 기본 기능 구현**

Cursor Composer:

```

알림 시스템의 기본 기능을 구현해줘:

백엔드:
1. Alert CRUD API
2. 가격 모니터링 서비스 (폴링 방식으로 일단 시작)
3. 알림 발송 서비스

프론트엔드:
1. 알림 설정 폼
2. 알림 목록
3. 알림 히스토리

WebSocket은 나중에 추가하고, 일단 기본 HTTP API로 동작하게 해줘.
```

**Day 4-5: 로컬 테스트**

```

다음 시나리오를 테스트할 수 있는 코드 작성:

1. 사용자가 알림 설정
2. 가격 조건 충족
3. 알림 발송
4. 히스토리 기록

MockMvc로 통합 테스트 작성해줘.
```

#### Week 3: 검증 및 개선 (Claude Code 주도)

**Day 1-2: 아키텍처 리뷰**

```markdown
프로토타입 코드를 리뷰해줘:

[전체 코드 첨부]

특히 다음을 중점적으로:

1. 확장성
   - 10만 사용자 지원 가능한가?
   - 병목 지점은?
   
2. 신뢰성
   - 알림 누락 가능성은?
   - 중복 알림 방지는?
   
3. 성능
   - 폴링 방식의 한계는?
   - 최적화 방안은?
   
4. 코드 품질
   - 에러 처리는 적절한가?
   - 테스트 커버리지는?
```

**Day 3-5: 개선 계획 수립**

```markdown
리뷰 결과를 바탕으로 개선 계획을 수립해줘:

발견된 문제:
1. 폴링 방식으로 인한 지연 (평균 30초)
2. 데이터베이스 부하 (1초마다 전체 알림 조회)
3. 동시 알림 발송 시 성능 저하
4. 중복 알림 가능성

개선 계획:
1. 우선순위 (High/Medium/Low)
2. 각 문제의 해결 방안 (여러 옵션)
3. 단계별 구현 순서
4. 예상 소요 시간
5. 리스크 분석
```

#### Week 4: 최적화 적용 (Cursor + Claude Code 협력)

**실시간 처리 전환 (Cursor):**

```

계획서의 Phase 1을 구현해줘:

1. WebSocket 서버 추가
2. Redis Pub/Sub 도입
3. 가격 변동 이벤트 스트리밍
4. 프론트엔드 WebSocket 클라이언트

기존 폴링 방식은 fallback으로 유지하면서 WebSocket을 주 방식으로 전환해줘.
```

**성능 검증 (Claude Code):**

```markdown
WebSocket 구현을 검증해줘:

[WebSocket 관련 코드]

검증 항목:
1. 연결 관리 (연결 수 제한, 재연결 로직)
2. 메시지 전달 보장
3. 에러 처리
4. 부하 테스트 시나리오
5. 모니터링 방안
```

**최적화 적용 (Cursor):**

```
Claude Code의 피드백을 반영해서 WebSocket 연결 관리를 개선해줘:

1. 연결 풀 관리
2. 하트비트 메커니즘
3. 자동 재연결
4. 에러 복구
```

---

## Part 7: 효율성 극대화 전략

### 전략 1: 동시 작업 패턴

**시나리오: 두 개의 관련 기능 동시 개발**

왼쪽 화면: Cursor (기능 A 구현)
오른쪽 화면: Claude Code (기능 B 설계)

**작업 흐름:**

1. Cursor로 기능 A 구현 중
2. 동시에 Claude Code로 기능 B PRD 작성
3. 기능 A 완료 → Claude Code 리뷰
4. 기능 B 설계 완료 → Cursor 구현

**효과:**
- 대기 시간 제로
- 지속적인 생산성
- 병렬 작업으로 속도 2배

### 전략 2: 워크스페이스 설정

**Cursor 설정 (.cursor/settings.json):**

```json
{
  "cursor.chat.historySize": 100,
  "cursor.composer.autoSave": true,
  "cursor.chat.copyCodeOnClick": true
}
```

**워크스페이스 레이아웃:**

- 왼쪽 50%: Cursor IDE
- 오른쪽 50%: Claude Code (브라우저)

### 전략 3: 템플릿 라이브러리

**Cursor용 프롬프트 템플릿:**

```
템플릿 1: 컴포넌트 생성
- Props 타입 정의
- 상태 관리
- 이벤트 핸들러
- 스타일링
```

**Claude Code용 프롬프트 템플릿:**

```markdown
템플릿 2: 코드 리뷰

[코드]

체크리스트:
- [ ] 보안
- [ ] 성능
- [ ] 가독성
- [ ] 테스트 가능성
```

### 전략 4: 피드백 루프 최소화

**Before (느림):**

```
Cursor 구현 → 복사 → Claude Code 리뷰 → 
복사 → Cursor 수정 → 복사 → Claude Code 검증
```

**After (빠름):**

```
1. Cursor로 큰 덩어리 구현
2. 한 번에 Claude Code로 전달
3. 종합 리뷰 받기
4. Cursor에서 일괄 수정
5. 최종 검증만 Claude Code
```

---

## 결론: 효과적인 협력의 핵심

### 핵심 원칙

**1. 명확한 역할 분담**
- Cursor = 빠른 실행
- Claude Code = 깊은 사고

**2. 적재적소 활용**
- 급할 때 Cursor
- 중요할 때 Claude Code

**3. 컨텍스트 공유**
- 문서로 연결
- 명확한 인수인계

**4. 반복 최소화**
- 큰 단위로 작업
- 일괄 처리

**5. 지속적 개선**
- 협력 패턴 문서화
- 템플릿 누적
- 프로세스 최적화

### 기대 효과

**속도:**
- 개발 속도: 3-5배 향상
- 리뷰 시간: 70% 단축
- 버그 수정: 80% 빨라짐

**품질:**
- 보안 취약점: 90% 감소
- 코드 품질: 현저히 향상
- 문서화: 항상 최신 유지

**만족도:**
- 개발자 만족도 증가
- 코드 리뷰 스트레스 감소
- 학습 기회 증가

---

## Quick Reference

### Cursor 주도 작업

- UI 컴포넌트 개발
- API 엔드포인트 구현
- 빠른 버그 수정
- 코드 리팩토링
- 테스트 코드 작성
- 반복적인 CRUD 로직

### Claude Code 주도 작업

- 아키텍처 설계
- 보안 감사
- 성능 분석
- 복잡한 알고리즘
- 문서 작성
- 코드 리뷰
- 장기 전략 수립

### 협력 필요 작업

- 신규 기능 개발
- 대규모 리팩토링
- 성능 최적화
- 보안 강화
- 기술 부채 해소

---

**문서 버전:** 1.0  
**작성일:** 2025-12-27  
**대상:** Cursor와 Claude Code를 함께 사용하는 개발자  
**라이선스:** CC BY-SA 4.0

이 가이드가 여러분의 AI 협력 개발에 도움이 되기를 바랍니다!