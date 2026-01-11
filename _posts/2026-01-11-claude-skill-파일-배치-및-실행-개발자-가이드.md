---
title: "Claude SKILL 파일 배치 및 실행 개발자 가이드"
date: 2026-01-11 19:00:00 +0900
categories: [AI,  Claude Skills]
mermaid: [True]
tags: [AI,  Guide,  claude-skills,  Developer,  Claude.write]
---


## 목차
- [SKILL 시스템 개요](#skill-시스템-개요)
- [SKILL 파일 구조](#skill-파일-구조)
- [SKILL 작성 기본](#skill-작성-기본)
- [SKILL 배치 방법](#skill-배치-방법)
- [SKILL 실행 및 트리거](#skill-실행-및-트리거)
- [고급 SKILL 패턴](#고급-skill-패턴)
- [SKILL 디버깅](#skill-디버깅)
- [실전 예시 모음](#실전-예시-모음)
- [팀 협업 및 공유](#팀-협업-및-공유)
- [트러블슈팅](#트러블슈팅)

---

## SKILL 시스템 개요

### SKILL이란?

Claude의 SKILL은 특정 작업이나 도메인에 대한 전문화된 지침을 제공하는 텍스트 기반 설정 파일입니다. SKILL을 사용하면 Claude가:

- 특정 맥락에서 일관된 행동을 보입니다
- 도메인 전문 지식을 활용합니다
- 반복적인 작업을 자동화합니다
- 품질 기준을 유지합니다

### SKILL의 작동 원리

```
사용자 입력
    ↓
SKILL 매칭 (자동 트리거 또는 명시적 호출)
    ↓
SKILL 내용이 시스템 프롬프트에 추가
    ↓
Claude 응답 생성 (SKILL 가이드라인 적용)
    ↓
결과 출력
```

### SKILL 파일 위치

Claude.ai 웹 인터페이스에서:
- **Settings → Profile → Custom Instructions** (개인 스타일)
- **Projects → Project Settings → Project Knowledge** (프로젝트별 SKILL)
- **Settings → Features → Skills** (전역 SKILL 관리)

---

## SKILL 파일 구조

### 기본 SKILL 파일 형식

SKILL 파일은 마크다운(.md) 형식으로 작성되며, 다음 구조를 권장합니다:

```markdown
# SKILL 이름

## 목적
이 SKILL이 해결하려는 문제를 1-2문장으로 명확히 기술합니다.

## 적용 범위
- 이 SKILL이 적용되는 상황
- 이 SKILL이 적용되지 않는 상황

## 핵심 원칙
1. 원칙 1
2. 원칙 2
3. 원칙 3

## 실행 절차
1. 단계 1
2. 단계 2
3. 단계 3

## 예시
### 좋은 예시
[구체적인 예시]

### 나쁜 예시
[피해야 할 예시]

## 제약사항
- 제약사항 1
- 제약사항 2

## 메타데이터
- 버전: 1.0.0
- 작성자: [이름]
- 작성일: 2026-01-11
- 카테고리: [카테고리명]
```

### SKILL 메타데이터 규칙

```markdown
---
skill_name: "API 응답 검토"
version: "1.0.0"
author: "개발팀"
created: "2026-01-11"
updated: "2026-01-11"
category: "백엔드 개발"
tags: ["API", "REST", "JSON", "검토"]
trigger_keywords: ["API 응답", "REST API", "응답 형식", "JSON 검토"]
auto_trigger: true
---
```

---

## SKILL 작성 기본

### 1. 최소 SKILL 예시

가장 간단한 형태의 SKILL:

```markdown
# 간결한 응답

## 목적
모든 응답을 3문장 이내로 제한합니다.

## 규칙
- 최대 3문장
- 핵심만 전달
- 불필요한 설명 생략

## 적용
이 SKILL은 빠른 답변이 필요한 모든 상황에 적용됩니다.
```

### 2. 중급 SKILL 예시

~~~markdown
# Python 코드 리뷰

## 목적
Python 코드의 품질, 보안, 성능을 검토합니다.

## 적용 범위
- Python 코드 (.py 파일)
- 코드 스니펫 및 전체 모듈
- 적용 안 됨: JavaScript, Java 등 다른 언어

## 검토 체크리스트

### 1. 코드 품질
- [ ] PEP 8 스타일 가이드 준수
- [ ] 의미 있는 변수명 사용
- [ ] 적절한 주석
- [ ] 함수는 단일 책임 원칙 준수

### 2. 보안
- [ ] SQL Injection 방지 (파라미터화된 쿼리)
- [ ] XSS 방지 (입력 검증)
- [ ] 민감 정보 하드코딩 여부
- [ ] 인증/권한 검증

### 3. 성능
- [ ] 불필요한 반복문 제거
- [ ] 효율적인 자료구조 사용
- [ ] 메모리 누수 가능성
- [ ] DB 쿼리 최적화

## 출력 형식

각 이슈를 다음 형식으로 보고:

**[심각도] 이슈 제목**
- 위치: 파일명:라인번호
- 문제: 구체적인 문제 설명
- 영향: 발생 가능한 문제
- 해결: 권장 수정 방법

심각도: 🔴 Critical | 🟡 Warning | 🔵 Info

## 예시

### 입력 코드
```python
def get_user(user_id):
    query = "SELECT * FROM users WHERE id = " + user_id
    return db.execute(query)
```

### 출력
🔴 **[Critical] SQL Injection 취약점**
- 위치: user_service.py:15
- 문제: 사용자 입력을 직접 SQL에 연결
- 영향: 데이터베이스 전체 노출 가능
- 해결:
```python
def get_user(user_id):
    query = "SELECT * FROM users WHERE id = %s"
    return db.execute(query, (user_id,))
```
~~~

### 3. 고급 SKILL 예시 (조건부 로직 포함)

~~~markdown
# 다국어 기술 문서 작성

## 목적
영어와 한국어로 기술 문서를 작성하며, 독자 수준에 맞춰 내용을 조정합니다.

## 언어 선택 규칙
- 사용자가 한국어로 요청 → 한국어 문서
- 사용자가 영어로 요청 → 영어 문서
- 명시적 언어 지정 → 해당 언어 사용

## 독자 수준 판단

### 초급 (Beginner)
**신호**: "처음", "기초", "입문", "배우고 싶어요"
**전략**:
- 전문 용어 최소화
- 각 개념 상세 설명
- 많은 예시 제공
- 단계별 가이드

### 중급 (Intermediate)
**신호**: "경험 있음", "알고 있음", "개선하고 싶음"
**전략**:
- 전문 용어 사용 (첫 언급 시 간단 설명)
- 개념보다 실전 중심
- 베스트 프랙티스 강조

### 고급 (Advanced)
**신호**: "최적화", "아키텍처", "고급", "성능"
**전략**:
- 전문 용어 자유롭게 사용
- 깊이 있는 기술적 분석
- 트레이드오프 논의
- 대안 제시

## 문서 구조

### 초급용
```
1. 개요 (이것이 무엇인가?)
2. 왜 필요한가?
3. 기본 개념
4. 단계별 튜토리얼
5. 자주 묻는 질문
6. 다음 단계
```

### 중급/고급용
```
1. 개요
2. 핵심 개념 (간략)
3. 구현 가이드
4. 고급 패턴
5. 성능 최적화
6. 문제 해결
```

## 예시

### 초급 독자 응답
```markdown
# Git 브랜치란 무엇인가요?

## 개요
Git 브랜치는 마치 나무의 가지처럼, 메인 코드에서 분리하여 독립적으로 
작업할 수 있게 해주는 기능입니다.

## 왜 필요한가요?
- 여러 기능을 동시에 개발할 수 있습니다
- 실험적인 코드를 안전하게 테스트할 수 있습니다
- 다른 사람의 작업과 충돌하지 않습니다
...
```

### 고급 독자 응답
```markdown
# Git 브랜칭 전략 최적화

## Git Flow vs GitHub Flow vs Trunk-Based Development

### 트레이드오프 분석
- **Git Flow**: 릴리스 주기가 명확한 대규모 팀
- **GitHub Flow**: CI/CD 환경에서 빠른 배포
- **Trunk-Based**: 높은 테스트 커버리지와 짧은 브랜치 수명
...
```

## 메타 규칙
- 사용자가 독자 수준을 명시하면 그대로 따름
- 첫 응답 후 사용자 반응으로 수준 재조정
- 기술 문서는 코드 예시 필수 포함
~~~

---

## SKILL 배치 방법

### 방법 1: Claude.ai 웹 인터페이스 (개인 사용)

#### 단계별 가이드

1. **Settings 접속**
   - Claude.ai 우측 상단 프로필 아이콘 클릭
   - "Settings" 선택

2. **Custom Instructions 또는 Skills 메뉴**
   - "Profile" → "Custom instructions" (전역 스타일)
   - 또는 "Features" → "Skills" (SKILL 관리)

3. **SKILL 추가**
   ```
   - "Add Skill" 버튼 클릭
   - SKILL 이름 입력
   - SKILL 내용 붙여넣기
   - "Save" 클릭
   ```

4. **SKILL 활성화**
   - 토글 스위치로 ON/OFF
   - 여러 SKILL 동시 활성화 가능

### 방법 2: Project에 SKILL 배치

#### Project Knowledge 활용

1. **프로젝트 생성 또는 선택**
   - 좌측 사이드바에서 "Projects" 클릭
   - 기존 프로젝트 선택 또는 "New Project" 생성

2. **Project Knowledge 추가**
   ```
   - 프로젝트 내에서 "Add content" 클릭
   - SKILL.md 파일 업로드
   - 또는 텍스트 직접 입력
   ```

3. **자동 적용**
   - 프로젝트 내 모든 대화에 자동 적용
   - 프로젝트 범위 내에서만 활성화

### 방법 3: 대화 중 즉석 SKILL 생성

#### Skill Creator 활용

1. **Skill Creator 활성화**
   ```
   - Settings → Features → Skill creator (ON)
   ```

2. **대화 중 SKILL 생성**
   ```
   사용자: "이 대화를 바탕으로 'API 문서 작성' SKILL을 만들어줘"
   Claude: [SKILL 내용 생성]
   사용자: "이걸 SKILL로 저장해줘"
   Claude: [SKILL 저장 인터페이스 제공]
   ```

### 방법 4: 파일 시스템 방식 (로컬 관리)

로컬에서 SKILL을 관리하는 경우:

```bash
# SKILL 파일 구조
skills/
├── backend/
│   ├── python-code-review.md
│   ├── api-design.md
│   └── database-optimization.md
├── frontend/
│   ├── react-component-review.md
│   └── css-best-practices.md
├── documentation/
│   ├── technical-writing.md
│   └── api-documentation.md
└── general/
    ├── concise-response.md
    └── code-explanation.md
```

**버전 관리**:
```bash
git init
git add skills/
git commit -m "Initial SKILL repository"
git push origin main
```

---

## SKILL 실행 및 트리거

### 자동 트리거

SKILL이 자동으로 활성화되는 조건:

```markdown
# SKILL 메타데이터에 트리거 키워드 정의

---
trigger_keywords: 
  - "코드 리뷰"
  - "code review"
  - "리팩토링"
  - "코드 검토"
auto_trigger: true
---
```

**작동 방식**:
```
사용자: "이 Python 코드를 리뷰해줘"
         ↓
시스템이 "코드 리뷰" 키워드 감지
         ↓
관련 SKILL 자동 활성화
         ↓
SKILL 가이드라인에 따라 응답
```

### 명시적 호출

사용자가 직접 SKILL을 호출:

```
# 방법 1: SKILL 이름 언급
"Python 코드 리뷰 SKILL을 사용해서 이 코드를 검토해줘"

# 방법 2: @ 멘션 (지원되는 경우)
"@python-code-review 이 코드 확인해줘"

# 방법 3: 명시적 지시
"다음 SKILL을 적용해줘: [SKILL 이름]"
```

### 다중 SKILL 조합

여러 SKILL을 순차적으로 적용:

```
사용자: "이 코드를 먼저 'Python 코드 리뷰' SKILL로 검토하고,
        그 다음 '성능 최적화' SKILL로 개선안을 제시해줘"

Claude:
1. Python 코드 리뷰 SKILL 적용 → 문제점 발견
2. 성능 최적화 SKILL 적용 → 개선안 제시
```

### SKILL 우선순위

여러 SKILL이 동시에 활성화된 경우:

```markdown
# 우선순위 설정

---
priority: high  # high, medium, low
override: true  # 다른 SKILL보다 우선
---
```

**우선순위 규칙**:
1. 명시적으로 호출된 SKILL
2. priority: high SKILL
3. 가장 최근에 활성화된 SKILL
4. 프로젝트 레벨 SKILL
5. 전역 SKILL

---

## 고급 SKILL 패턴

### 패턴 1: 조건부 SKILL

```markdown
# 상황별 코딩 스타일

## 언어 감지 및 스타일 적용

### Python 코드인 경우
- PEP 8 준수
- Type hints 사용 권장
- Docstring (Google 스타일)

### JavaScript 코드인 경우
- ESLint 규칙 준수
- JSDoc 주석
- async/await 선호

### Java 코드인 경우
- Google Java Style Guide
- JavaDoc
- Stream API 활용

## 자동 감지 로직
첫 줄에서 언어 식별:
- `def`, `import` → Python
- `function`, `const`, `let` → JavaScript
- `public class`, `package` → Java

해당 언어 스타일 가이드 자동 적용
```

### 패턴 2: 계층적 SKILL

```markdown
# 마스터 SKILL: 풀스택 개발

## Sub-Skills

### 1. 백엔드 개발
이 섹션이 활성화되면 다음 Sub-Skills 적용:
- API 설계 SKILL
- 데이터베이스 최적화 SKILL
- 보안 검토 SKILL

### 2. 프론트엔드 개발
이 섹션이 활성화되면 다음 Sub-Skills 적용:
- React 컴포넌트 SKILL
- CSS 최적화 SKILL
- 접근성 검토 SKILL

### 3. DevOps
- Docker 컨테이너화 SKILL
- CI/CD 파이프라인 SKILL
- 모니터링 설정 SKILL

## 작업 유형별 활성화
사용자 요청 분석:
- "API 만들어줘" → 백엔드 Sub-Skills 활성화
- "UI 개선해줘" → 프론트엔드 Sub-Skills 활성화
- "배포 자동화" → DevOps Sub-Skills 활성화
```

### 패턴 3: 반복 개선 SKILL

```markdown
# 반복적 코드 개선

## 프로세스

### 1단계: 초기 분석
현재 코드의 문제점 식별

### 2단계: 개선안 제시
3가지 개선 방향 제안

### 3단계: 사용자 선택
사용자가 선택한 방향으로 구현

### 4단계: 검증
개선된 코드 재검토

### 5단계: 추가 개선 여부
"더 개선할 부분이 있나요?" 질문
- 있으면 → 1단계로 돌아가기
- 없으면 → 최종 완성

## 종료 조건
- 사용자가 만족 표현
- 3회 반복 후 자동 종료
- "완료" 또는 "끝" 키워드
```

### 패턴 4: 컨텍스트 누적 SKILL

```markdown
# 프로젝트 컨텍스트 관리

## 세션 정보 저장

### 기술 스택
처음 언급 시 기록:
- 언어: [...]
- 프레임워크: [...]
- 데이터베이스: [...]

### 코딩 규칙
대화 중 확인된 팀 규칙:
- 네이밍 컨벤션: [...]
- 파일 구조: [...]
- 테스트 요구사항: [...]

### 비즈니스 컨텍스트
- 도메인: [...]
- 주요 사용자: [...]
- 성능 요구사항: [...]

## 컨텍스트 활용
모든 후속 응답에 위 정보 자동 반영
사용자가 반복 설명 불필요
```

---

## SKILL 디버깅

### 디버깅 모드 SKILL

~~~markdown
# SKILL 디버깅 모드

## 목적
SKILL이 예상대로 작동하는지 확인합니다.

## 활성화
사용자가 "디버그 모드"라고 말하면 활성화

## 디버깅 출력

### 1. 활성화된 SKILL 목록
```
현재 활성화된 SKILL:
1. Python 코드 리뷰 (auto-triggered)
2. 성능 최적화 (explicitly called)
3. 디버그 모드 (active)
```

### 2. 적용된 규칙
```
적용 중인 규칙:
- PEP 8 스타일 검사
- 보안 취약점 검토
- 성능 병목 지점 분석
```

### 3. 의사결정 과정
```
의사결정 트레이스:
1. 입력 분석: Python 코드 감지
2. SKILL 매칭: "Python 코드 리뷰" 트리거
3. 규칙 적용: 보안 검사 우선순위 high
4. 출력 생성: 발견된 이슈 3건
```

## 사용 예시
```
사용자: "디버그 모드 켜줘"
Claude: [디버그 모드 활성화]

사용자: "이 코드 리뷰해줘 [코드]"
Claude: 
--- 디버그 정보 ---
활성화된 SKILL: Python 코드 리뷰
트리거 이유: "리뷰" 키워드 감지
적용 규칙: PEP8, 보안, 성능
--- 실제 응답 ---
[코드 리뷰 결과]
```
~~~

### SKILL 성능 측정

~~~markdown
# SKILL 효과성 측정

## 메트릭

### 정확도
- 요구사항 충족도: X/Y
- 오류율: Z%

### 효율성
- 응답 시간: 적정/느림
- 반복 질문 횟수: N회

### 만족도
각 응답 후 확인:
"이 응답이 도움이 되었나요? (예/아니오/부분적으로)"

## 개선 제안
만족도가 낮은 경우:
"SKILL을 어떻게 개선하면 좋을까요?"
~~~

---

## 실전 예시 모음

### 예시 1: Git 커밋 메시지 SKILL

~~~markdown
# Git 커밋 메시지 작성

## 목적
Conventional Commits 규칙에 따라 일관된 커밋 메시지를 작성합니다.

## 형식
```
<type>(<scope>): <subject>

<body>

<footer>
```

## Type 종류
- **feat**: 새로운 기능
- **fix**: 버그 수정
- **docs**: 문서 변경
- **style**: 코드 포맷팅
- **refactor**: 리팩토링
- **test**: 테스트 추가/수정
- **chore**: 빌드/설정 변경

## 규칙
1. subject는 50자 이내
2. 명령형 현재 시제 ("Add" not "Added")
3. 첫 글자는 대문자
4. 마침표 없음
5. body는 72자마다 줄바꿈
6. footer는 이슈 번호 참조

## 예시

### 입력
"로그인 API에 JWT 토큰 검증 로직을 추가했어요. 이슈 #123과 관련이 있습니다."

### 출력
```
feat(auth): Add JWT token validation to login API

Implement JWT token validation logic to enhance security.
The validation checks token expiration, signature, and claims.

Closes #123
```

### 입력
"README 파일에 설치 방법을 추가했습니다."

### 출력
```
docs(readme): Add installation instructions

Include step-by-step guide for local development setup
covering npm install and environment configuration.
```
~~~

### 예시 2: SQL 쿼리 최적화 SKILL

~~~markdown
# SQL 쿼리 최적화

## 목적
느린 SQL 쿼리를 분석하고 최적화합니다.

## 분석 체크리스트

### 1. 인덱스 확인
- [ ] WHERE 절 컬럼에 인덱스 존재?
- [ ] JOIN 컬럼에 인덱스 존재?
- [ ] 복합 인덱스 필요 여부

### 2. 쿼리 구조
- [ ] SELECT * 사용 여부 → 필요한 컬럼만
- [ ] 서브쿼리 존재 → JOIN으로 변환 가능?
- [ ] DISTINCT 사용 → 중복 데이터 원인 파악

### 3. 조인 최적화
- [ ] 조인 순서 최적화
- [ ] INNER JOIN vs LEFT JOIN 적절성
- [ ] 조인 조건 인덱스 활용

### 4. 기타
- [ ] LIMIT 사용 여부
- [ ] 함수 사용으로 인한 인덱스 미사용
- [ ] OR 조건을 UNION으로 분리 가능성

## 출력 형식

### 원본 쿼리
```sql
[원본 쿼리]
```

### 문제점
1. [문제 1]
2. [문제 2]

### 최적화된 쿼리
```sql
[최적화된 쿼리]
```

### 추가 권장사항
- [인덱스 생성 SQL]
- [테이블 구조 개선안]

### 예상 성능 개선
- 실행 시간: X초 → Y초 (Z% 개선)
~~~

### 예시 3: 에러 메시지 분석 SKILL

~~~markdown
# 에러 메시지 분석 및 해결

## 목적
에러 메시지를 분석하고 해결 방법을 단계별로 제시합니다.

## 분석 프로세스

### 1. 에러 타입 식별
- Syntax Error
- Runtime Error
- Logic Error
- Configuration Error
- Dependency Error

### 2. 근본 원인 파악
에러 스택 트레이스 분석:
- 에러 발생 위치
- 호출 스택
- 관련 파일/함수

### 3. 해결 방법 제시

## 출력 템플릿

### 📋 에러 요약
```
타입: [에러 타입]
위치: [파일명:라인번호]
메시지: [에러 메시지]
```

### 🔍 원인 분석
[상세 원인 설명]

### ✅ 해결 방법

#### 방법 1: [빠른 수정]
```
[코드 또는 명령]
```
**장점**: 즉시 해결
**단점**: 임시 방편

#### 방법 2: [권장 방법]
```
[코드 또는 명령]
```
**장점**: 근본적 해결
**단점**: 시간 소요

### 🔧 예방 방법
향후 동일한 에러 방지:
- [예방책 1]
- [예방책 2]

### 📚 관련 문서
- [공식 문서 링크]
- [Stack Overflow 관련 질문]

## 예시

### 입력
```
Error: Cannot find module 'express'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:815:15)
    at Function.Module._load (internal/modules/cjs/loader.js:667:27)
```

### 출력
### 📋 에러 요약
```
타입: Dependency Error
위치: 모듈 로딩 단계
메시지: Cannot find module 'express'
```

### 🔍 원인 분석
Express 모듈이 설치되지 않았거나 node_modules가 손상되었습니다.

### ✅ 해결 방법

#### 방법 1: Express 설치
```bash
npm install express
```
**장점**: 빠른 해결
**단점**: 다른 누락 패키지도 있을 수 있음

#### 방법 2: 전체 재설치 (권장)
```bash
rm -rf node_modules package-lock.json
npm install
```
**장점**: 모든 의존성 정확히 재설치
**단점**: 시간 소요

### 🔧 예방 방법
- package.json의 dependencies에 express 버전 명시
- .gitignore에 node_modules 추가
- CI/CD에서 npm ci 사용
~~~

---

## 팀 협업 및 공유

### SKILL 저장소 구조

```
team-skills/
├── README.md
├── CONTRIBUTING.md
├── skills/
│   ├── backend/
│   │   ├── python/
│   │   │   ├── code-review.md
│   │   │   ├── django-best-practices.md
│   │   │   └── fastapi-patterns.md
│   │   ├── java/
│   │   │   ├── spring-boot-review.md
│   │   │   └── java-security.md
│   │   └── shared/
│   │       ├── api-design.md
│   │       └── database-optimization.md
│   ├── frontend/
│   │   ├── react/
│   │   │   ├── component-review.md
│   │   │   └── hooks-best-practices.md
│   │   └── vue/
│   │       └── composition-api.md
│   ├── devops/
│   │   ├── docker.md
│   │   ├── kubernetes.md
│   │   └── ci-cd.md
│   └── general/
│       ├── git-commit-messages.md
│       ├── code-documentation.md
│       └── testing-guidelines.md
└── templates/
    ├── basic-skill-template.md
    └── advanced-skill-template.md
```

### README.md 예시

~~~markdown
# 팀 SKILL 저장소

## 사용 방법

### 1. SKILL 찾기
```bash
# 전체 SKILL 목록
ls skills/

# 카테고리별 검색
ls skills/backend/python/
```

### 2. SKILL 사용
1. 원하는 SKILL 파일 열기
2. 내용 전체 복사
3. Claude.ai에 붙여넣기 또는 프로젝트에 추가

### 3. SKILL 기여

#### 새 SKILL 추가
1. `templates/basic-skill-template.md` 복사
2. 적절한 카테고리 폴더에 배치
3. SKILL 작성
4. Pull Request 생성

#### 기존 SKILL 수정
1. 해당 SKILL 파일 수정
2. 버전 번호 증가
3. Changelog 업데이트
4. Pull Request 생성

## SKILL 네이밍 규칙
- 소문자 사용
- 단어는 하이픈(-)으로 연결
- .md 확장자
- 예: `python-code-review.md`

## 버전 관리
- 주 변경: X.0.0 (호환성 깨짐)
- 부 변경: 0.X.0 (기능 추가)
- 패치: 0.0.X (버그 수정)

## 리뷰 프로세스
1. PR 생성
2. 최소 2명의 승인 필요
3. CI 통과 확인
4. Merge
~~~

### SKILL 공유 프로세스

```bash
# 1. SKILL 저장소 클론
git clone https://github.com/your-team/skills.git
cd skills

# 2. 새 SKILL 작성
cp templates/basic-skill-template.md skills/backend/python/new-skill.md
# 편집...

# 3. 커밋 및 푸시
git add skills/backend/python/new-skill.md
git commit -m "feat(python): Add new-skill for [purpose]"
git push origin feature/new-skill

# 4. Pull Request 생성
# GitHub에서 PR 생성

# 5. 리뷰 및 병합
# 팀원 리뷰 후 승인

# 6. 최신 SKILL 가져오기
git pull origin main
```

---

## 트러블슈팅

### 문제 1: SKILL이 트리거되지 않음

**증상**: SKILL을 활성화했지만 적용되지 않음

**원인 및 해결**:

1. **트리거 키워드 불일치**
   ```markdown
   # 수정 전
   trigger_keywords: ["코드리뷰"]
   
   # 수정 후 (다양한 변형 포함)
   trigger_keywords: 
     - "코드리뷰"
     - "코드 리뷰"
     - "code review"
     - "리뷰해줘"
   ```

2. **SKILL 비활성화 상태**
   - Settings에서 SKILL이 ON인지 확인
   - 프로젝트 내에서 올바른 프로젝트 선택 확인

3. **명시적 호출 시도**
   ```
   "[SKILL 이름] SKILL을 사용해서 이 작업을 해줘"
   ```

### 문제 2: 여러 SKILL이 충돌

**증상**: 예상과 다른 결과 또는 혼란스러운 응답

**해결**:

1. **우선순위 설정**
   ```markdown
   ---
   priority: high
   override: true
   ---
   ```

2. **SKILL 범위 명확화**
   ```markdown
   ## 적용 범위
   이 SKILL은 Python 코드에만 적용됩니다.
   
   ## 적용 제외
   - JavaScript 코드
   - Java 코드
   - 일반 텍스트
   ```

3. **불필요한 SKILL 비활성화**
   - 현재 작업과 무관한 SKILL은 OFF

### 문제 3: SKILL이 너무 제한적

**증상**: SKILL이 유연성 없이 항상 같은 방식만 적용

**해결**:

```markdown
# 수정 전
## 규칙
반드시 3단계를 따라야 합니다.

# 수정 후
## 권장 프로세스
일반적으로 다음 3단계를 따릅니다:
1. ...
2. ...
3. ...

단, 상황에 따라 단계 생략 또는 순서 변경 가능합니다.

## 예외 처리
다음 경우 프로세스를 조정합니다:
- 간단한 작업: 1단계만 수행
- 복잡한 작업: 추가 단계 삽입
- 긴급 상황: 검증 단계 간소화
```

### 문제 4: SKILL 성능 저하

**증상**: 응답이 느리거나 불필요하게 복잡함

**해결**:

1. **SKILL 간소화**
   ```markdown
   # 수정 전 (1000 라인)
   [매우 상세한 모든 케이스]
   
   # 수정 후 (200 라인)
   ## 핵심 원칙만 유지
   - 원칙 1
   - 원칙 2
   - 원칙 3
   
   ## 상세 내용은 별도 문서 참조
   [링크]
   ```

2. **조건부 활성화**
   ```markdown
   ## 세부 규칙 (고급 모드에서만)
   사용자가 "상세 분석" 요청 시에만 적용:
   - [상세 규칙 1]
   - [상세 규칙 2]
   ```

### 문제 5: SKILL 버전 관리 혼란

**증상**: 팀원들이 서로 다른 버전 사용

**해결**:

1. **버전 명시**
   ```markdown
   ---
   skill_name: "Python 코드 리뷰"
   version: "2.1.0"
   changelog:
     - "2.1.0 (2026-01-11): 타입 힌트 검사 추가"
     - "2.0.0 (2026-01-05): 보안 규칙 강화 (Breaking)"
     - "1.2.0 (2025-12-20): 성능 체크 추가"
   ---
   ```

2. **Git 태그 활용**
   ```bash
   git tag -a v2.1.0 -m "Python 코드 리뷰 v2.1.0"
   git push origin v2.1.0
   ```

3. **최신 버전 확인 스크립트**
   ```bash
   #!/bin/bash
   # check-skill-version.sh
   
   SKILL_FILE="skills/backend/python/code-review.md"
   CURRENT_VERSION=$(grep "version:" $SKILL_FILE | cut -d'"' -f2)
   LATEST_TAG=$(git describe --tags --abbrev=0)
   
   if [ "$CURRENT_VERSION" != "$LATEST_TAG" ]; then
       echo "⚠️  버전 불일치!"
       echo "현재: $CURRENT_VERSION"
       echo "최신: $LATEST_TAG"
       echo "git pull 실행을 권장합니다."
   else
       echo "✅ 최신 버전 사용 중"
   fi
   ```

---

## 마치며: SKILL 활용 베스트 프랙티스

### 1. 작게 시작하고 반복 개선
- 처음엔 간단한 SKILL로 시작
- 실제 사용하면서 부족한 부분 추가
- 과도한 복잡성 피하기

### 2. 팀과 공유
- 유용한 SKILL은 팀 저장소에 기여
- 피드백 수집 및 반영
- 버전 관리 철저히

### 3. 정기적 리뷰
- 월 1회 SKILL 효과성 검토
- 사용하지 않는 SKILL 제거
- 새로운 필요에 맞춰 업데이트

### 4. 문서화
- SKILL의 목적과 사용법 명확히 기술
- 예시 풍부하게 제공
- 변경 이력 관리

### 5. 측정 및 개선
- SKILL 사용 빈도 추적
- 만족도 수집
- 데이터 기반 개선

---

**작성 일자: 2026-01-11**
