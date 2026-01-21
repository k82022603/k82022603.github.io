---
title: "기획 템플릿의 자산화: Claude Code + Antigravity Skills 협업 실전 가이드"
date: 2026-01-21 19:30:00 +0900
categories: [AI,  Claude Code & Google Antigravity]
mermaid: [True]
tags: [AI,   claude-code,  Antigravity,   design-system,  Essence-of-Essence,  Claude.write]
---


## 들어가며: 2025-2026 AI 개발 환경의 패러다임 전환

2025년 하반기, AI 코딩 도구의 지형이 크게 변화했습니다. Anthropic은 10월에 Claude Code 2.0과 함께 Skills 기능을 공개했고, Google은 11월 18일 Gemini 3 모델과 함께 Antigravity IDE를 발표했습니다. 이 두 도구의 등장은 단순한 코드 자동완성을 넘어 "에이전트 중심(Agent-First)" 개발 환경으로의 전환을 의미합니다.

특히 웹기획자와 웹디자이너에게 이 변화는 중요합니다. 이제 여러분은 코딩을 직접 배우지 않고도 자신의 기획 방법론과 디자인 원칙을 코드로 실행 가능한 형태로 만들 수 있습니다. 우아한형제들 임동준 님이 강조한 '에센스 오브 에센스'를 Skills로 패키징하면, AI 에이전트가 여러분의 전문성을 이해하고 일관되게 실행합니다. 이것이 바로 '기획 템플릿의 자산화'입니다.

이 문서는 Claude Code의 Skills 기능과 Google Antigravity를 실무에서 어떻게 협업시켜 사용하는지, 그리고 웹기획·디자인 워크플로우를 어떻게 자산으로 만들 수 있는지에 대한 실전 가이드입니다.

## Part 1. 도구 이해하기: Claude Code vs Antigravity

### 1-1. Claude Code Skills의 핵심 개념

Claude Code는 2025년 10월 16일 Skills 기능을 출시하면서 단순한 채팅 도구에서 확장 가능한 개발 플랫폼으로 진화했습니다. Skills는 재사용 가능한 워크플로우를 패키징하는 방법으로, 다음과 같은 구조를 가집니다.

**Skills의 핵심 구조**는 폴더 기반입니다. 각 Skill은 독립된 폴더이며, 필수적으로 SKILL.md 파일을 포함합니다. 이 파일은 YAML frontmatter(메타데이터)와 Markdown 본문(실행 지침)으로 구성됩니다. 선택적으로 Python 스크립트, 템플릿 파일, 데이터 파일 등 보조 자원을 함께 포함할 수 있습니다.

예를 들어 랜딩 페이지 기획 Skill의 구조는 다음과 같습니다. `.claude/skills/landing-page-planner/` 폴더 안에 `SKILL.md`가 핵심 파일로 있고, `templates/` 폴더에는 `hero-section.md`, `cta-guide.md` 같은 템플릿들이 있으며, `checklist/` 폴더에는 `conversion-checklist.json`, `seo-checklist.json` 같은 검증 체크리스트가 있습니다.

**Skills의 작동 방식**을 보면, Claude Code는 시작할 때 `.claude/skills/` 디렉토리를 스캔하여 모든 Skill을 발견합니다. 각 SKILL.md 파일의 frontmatter에서 이름, 설명, 호출 조건을 읽어 시스템에 등록합니다. 사용자가 작업을 요청하면 Claude는 자동으로 관련 Skill을 찾아 로드하거나, 사용자가 `/skill-name` 명령으로 직접 호출할 수 있습니다. Skill이 실행되면 SKILL.md의 지침이 대화 컨텍스트에 주입되어 Claude의 행동을 가이드합니다.

**Skills와 다른 확장 방법의 차이**를 명확히 해야 합니다. 일반 프롬프트는 일회성 지시로 매번 반복 입력해야 하고, MCP 서버는 외부 도구 연결로 서버 설정과 유지관리가 필요하며, Slash 명령어는 간단한 텍스트 단축키로 복잡한 로직 불가능합니다. 반면 Skills는 재사용 가능한 워크플로우 패키지로 로컬에서 실행되며 보조 스크립트와 템플릿을 포함할 수 있고, 프로젝트 간 공유 가능하며 버전 관리가 용이합니다.

Anthropic의 내부 벤치마크에 따르면 Skills를 사용하는 팀은 반복적인 프롬프트 작성 시간을 73% 단축했습니다. 같은 워크플로우를 주 3회 이상 실행한다면 Skills로 전환하는 것이 효율적입니다.

### 1-2. Google Antigravity의 혁신적 특징

Google Antigravity는 2025년 11월 18일 Gemini 3와 함께 발표된 "에이전트 우선(Agent-First)" IDE입니다. Sergey Brin이 "Founder Mode"로 직접 개발에 참여하여 화제가 되었으며, Windsurf 팀을 24억 달러에 인수하여 그들의 기술을 기반으로 구축했습니다.

**Antigravity의 두 가지 작업 모드**를 이해하는 것이 중요합니다. Editor View는 전통적인 IDE 인터페이스로 VS Code와 유사한 환경에서 에이전트가 사이드바에 있습니다. 코드 자동완성, 인라인 수정, 실시간 협업이 가능하며, 개발자가 직접 손으로 코드를 작성할 때 사용합니다. Manager View는 여러 에이전트를 오케스트레이션하는 컨트롤 센터로 비동기 작업 실행, 병렬 처리, 작업 진행 상황 모니터링이 가능하며, 여러 작업을 동시에 위임할 때 사용합니다.

**Antigravity의 핵심 차별점**은 세 가지입니다. 첫째, 자율적 계획 및 실행입니다. 단순히 코드를 제안하는 것이 아니라 작업을 분석하고 계획을 세워 스스로 실행합니다. 둘째, 직접 시스템 접근입니다. 에디터, 터미널, 브라우저에 직접 접근하여 코드 작성, 명령 실행, UI 테스트를 자동으로 수행합니다. 셋째, 자기 검증 시스템입니다. 작업 완료 후 스스로 테스트하고 검증하여 Artifacts(스크린샷, 워크스루, 로그)를 생성합니다.

**지원 모델과 가격**을 보면 Gemini 3 Pro, Deep Think, Flash가 기본 모델이고, Claude Sonnet 4.5, Opus 4.5도 완전 지원하며, OpenAI GPT-OSS-120B(오픈소스 변형)도 사용 가능합니다. 2025년 현재 공개 프리뷰는 개인 사용자에게 무료이며, 넉넉한 속도 제한이 제공됩니다.

**비교: Cursor vs GitHub Copilot vs Antigravity**를 표로 정리하면 다음과 같습니다. Cursor는 코드 자동완성 O, 자율적 계획 X, 시스템 접근 X, 브라우저 테스트 X, 멀티 에이전트 X입니다. GitHub Copilot은 코드 자동완성 O, 자율적 계획 X, 시스템 접근 X, 브라우저 테스트 X, 멀티 에이전트 X입니다. Antigravity는 코드 자동완성 O, 자율적 계획 O, 시스템 접근 O, 브라우저 테스트 O, 멀티 에이전트 O입니다.

### 1-3. 두 도구를 함께 사용하는 이유

Claude Code와 Antigravity는 경쟁 관계가 아니라 보완 관계입니다. 각각의 강점이 명확하기 때문에 작업 성격에 따라 선택하거나 함께 사용하면 시너지가 발생합니다.

**Claude Code의 강점**은 섬세한 대화형 개발입니다. 단계별 질문과 피드백으로 요구사항을 명확히 하며, Skills와 MCP를 통한 높은 커스터마이징 가능성이 있습니다. CLAUDE.md와 메모리 시스템으로 프로젝트 컨텍스트를 장기간 유지하고, 웹기획자와 디자이너가 사용하기 쉬운 자연어 인터페이스를 제공합니다.

**Antigravity의 강점**은 대규모 자율 작업 처리입니다. 여러 에이전트가 병렬로 작업하여 처리 속도가 빠르고, 브라우저 자동화로 UI 테스트까지 완료하며, Manager View로 복잡한 프로젝트를 한눈에 관리할 수 있습니다. Google Cloud와의 통합으로 기업 환경에 적합합니다.

**실전 협업 시나리오**를 구체적으로 보겠습니다. 기획 단계에서는 Claude Code로 요구사항을 대화형으로 정리하고 Skills를 활용해 구조화된 기획서를 생성합니다. 개발 단계에서는 Antigravity에서 여러 에이전트를 동시에 실행하여 프론트엔드, 백엔드, DB 스키마를 병렬로 작업합니다. 테스트 단계에서는 Antigravity의 브라우저 에이전트가 자동으로 UI를 테스트하고 검증합니다. 문서화 단계에서는 다시 Claude Code로 돌아와 API 문서, 사용자 가이드를 생성합니다.

한 개발팀의 실제 사례를 보면, "우리는 기획과 아키텍처 설계는 Claude Code로, 대량의 보일러플레이트 코드 생성은 Antigravity로, 최종 리팩토링과 문서화는 다시 Claude Code로 처리합니다. 이 방식으로 프로젝트 초기 개발 기간을 40% 단축했습니다"라고 합니다.

## Part 2. Claude Code Skills 만들기: 웹기획 템플릿 자산화

### 2-1. Skills 개발 환경 설정

Claude Code Skills를 만들기 전에 개발 환경을 제대로 설정해야 합니다.

**시스템 요구사항**은 다음과 같습니다. Claude Pro 구독이 필수이며 월 $20입니다. Skills는 무료 티어에서는 사용할 수 없습니다. Claude Desktop 앱은 v0.7.0 이상이거나 Claude CLI v2.1.0 이상이어야 합니다. 운영 체제는 macOS 11+, Windows 10+, Linux (Ubuntu 20.04+)를 지원합니다. 디스크 공간은 Skills 런타임용 50MB, 개별 Skill당 200-500MB(종속성 포함)가 필요합니다. Git은 선택사항이지만 버전 관리를 위해 권장됩니다.

**설치 단계**는 다음과 같습니다. Claude Desktop을 claude.ai에서 다운로드하여 설치하고 Google 계정으로 로그인합니다. Skills 디렉토리를 생성합니다. macOS/Linux는 `mkdir -p ~/.claude/skills`, Windows는 `mkdir %USERPROFILE%\.claude\skills`로 생성합니다. Git 저장소를 초기화합니다. `cd ~/.claude/skills`로 이동한 후 `git init`로 초기화합니다. 이렇게 하면 Skills를 버전 관리하고 팀과 공유할 수 있습니다.

**디렉토리 구조 이해하기**를 보면 개인 Skills는 `~/.claude/skills/` 아래에 위치하며 모든 프로젝트에서 사용 가능합니다. 프로젝트별 Skills는 `{프로젝트루트}/.claude/skills/` 아래에 위치하며 해당 프로젝트에만 적용됩니다. 시스템 Skills는 `~/.claude/skills/.system/` 아래에 위치하며 Claude가 기본 제공하는 Skills입니다.

**첫 Skill 만들기 (Hello World)** 로 실습해보겠습니다. Skills 폴더에 새 디렉토리를 만듭니다. `mkdir ~/.claude/skills/hello-planner`로 만듭니다. SKILL.md 파일을 생성합니다. 내용은 다음과 같습니다.

```markdown
---
name: hello-planner
description: 간단한 인사와 함께 프로젝트 기획을 시작하는 Skill입니다.
---

# Hello Planner Skill

당신은 친절한 웹기획 어시스턴트입니다. 사용자가 이 Skill을 호출하면 다음과 같이 행동하세요:

1. 따뜻한 인사로 시작하기
2. 프로젝트의 목표를 질문하기
3. 타겟 사용자에 대해 질문하기
4. 핵심 기능 3가지를 정리해주겠다고 제안하기

항상 친절하고 명확한 언어를 사용하세요.
```

Claude Code에서 테스트합니다. Claude Desktop을 열고 새 대화를 시작한 후 `/hello-planner`를 입력하면 Skill이 활성화되어 지침대로 행동하는 것을 확인할 수 있습니다.

### 2-2. 랜딩 페이지 기획 Skill 만들기 (실전 예제)

이제 실무에서 바로 사용할 수 있는 랜딩 페이지 기획 Skill을 만들어보겠습니다. 이 Skill은 우아한형제들 임동준 님이 제시한 '에센스 오브 에센스' 철학을 구현한 것으로, 랜딩 페이지 기획의 본질적인 5단계를 체계화했습니다.

**Skill 구조 설계**는 다음과 같습니다. 메인 디렉토리는 `~/.claude/skills/landing-page-planner/`입니다. 필수 파일로 SKILL.md(핵심 기획 프로세스)가 있습니다. 보조 자료로는 `templates/hero-section-template.md`(히어로 섹션 템플릿), `templates/value-proposition-template.md`(가치 제안 템플릿), `templates/cta-template.md`(행동 유도 템플릿), `checklists/conversion-checklist.json`(전환율 최적화 체크리스트), `checklists/seo-checklist.json`(SEO 체크리스트)가 있습니다. 스크립트로는 `scripts/competitor-analyzer.py`(경쟁사 랜딩 페이지 분석)가 있습니다.

**SKILL.md 파일 작성**은 다음과 같습니다.

~~~markdown
---
name: landing-page-planner
description: 전환율 최적화에 집중한 랜딩 페이지 기획 전문 Skill. 사용자가 '랜딩 페이지', '리딩 페이지', 'LP 기획' 같은 키워드를 언급하면 자동으로 활성화됩니다.
auto_invoke: true
tags: [기획, 웹디자인, 마케팅, UX]
version: 1.0.0
---

# 랜딩 페이지 기획 5단계 프로세스

당신은 15년 경력의 웹기획 전문가입니다. 사용자가 랜딩 페이지 기획을 요청하면, 다음 5단계 프로세스를 **반드시 순서대로** 진행하세요.

## 불변의 원칙 (Invariants)

이 원칙들은 **어떤 상황에서도 절대 양보할 수 없습니다**:

1. **3초 법칙**: 사용자가 페이지에 도착한 후 3초 안에 핵심 가치를 이해해야 함
2. **단일 CTA 원칙**: 한 페이지에는 하나의 명확한 행동 유도만 존재
3. **모바일 우선**: 모든 요소는 모바일 화면에서 먼저 최적화된 후 데스크톱으로 확장
4. **신뢰 요소 필수**: 사회적 증거(리뷰, 고객사, 수치)가 반드시 포함되어야 함

## 1단계: 목표 명확화 (Goal Clarification)

사용자에게 다음 질문들을 순서대로 던지세요:

- 이 랜딩 페이지의 **단 하나의 목표**는 무엇인가요? (예: 이메일 수집, 무료 체험 신청, 제품 구매)
- 성공을 어떻게 측정할 건가요? (예: 전환율 5%, 월 1,000명 가입)
- 사용자는 어떤 경로로 이 페이지에 도착하나요? (예: Google 광고, 소셜 미디어, 블로그 링크)

**출력**: 한 문장으로 압축된 목표 진술
예시: "유료 광고를 통해 유입된 스타트업 창업자들이 14일 무료 체험을 신청하도록 유도. 목표 전환율 8%."

## 2단계: 사용자 진입 맥락 파악 (Entry Context)

사용자가 페이지에 도착하기 직전의 상황을 상세히 파악하세요:

- **진입 전 감정 상태**: 사용자는 무엇 때문에 불편하거나 궁금해하나요?
- **기대치**: 사용자는 클릭하기 전 어떤 정보나 해결책을 기대했나요?
- **장벽**: 사용자가 전환하지 않고 떠날 가능성이 있는 이유는 무엇인가요?

**출력**: 3문장으로 정리된 진입 맥락
예시: "사용자는 '직원 성과 관리 자동화'를 검색하다가 광고를 클릭했습니다. 현재 엑셀로 수동 관리 중이며 시간이 너무 많이 소요됩니다. 새로운 도구 도입의 러닝 커브를 걱정하고 있습니다."

## 3단계: 히어로 섹션 핵심 메시지 추출 (Hero Message)

히어로 섹션은 랜딩 페이지의 가장 중요한 부분입니다. 다음 공식을 따르세요:

**공식**: [타겟 고객]이 [문제/욕구]를 [솔루션]으로 [구체적 결과]를 달성

`templates/hero-section-template.md` 파일을 참조하여 작성하되, 다음 요소를 **반드시** 포함하세요:

- **메인 헤드라인**: 핵심 가치 제안 (25자 이내)
- **서브 헤드라인**: 구체적인 혜택 설명 (50자 이내)
- **비주얼 힌트**: 어떤 이미지나 영상이 적합한지
- **Primary CTA**: 행동 유도 버튼 문구

**나쁜 예**: "최고의 HR 솔루션으로 생산성을 높이세요" (너무 추상적)
**좋은 예**: "엑셀 지옥에서 탈출 → 14일 무료로 성과 관리 자동화" (구체적, 감정 자극)

## 4단계: 신뢰 요소 배치 (Trust Building)

사용자는 기본적으로 의심합니다. 신뢰를 구축하기 위한 요소를 다음 우선순위로 배치하세요:

1. **사회적 증거 (Social Proof)**
   - 고객 리뷰 및 평점
   - 사용 기업 로고 (특히 유명 브랜드)
   - 사용자 수 / 거래량 등 정량적 지표

2. **권위 (Authority)**
   - 수상 경력, 인증, 미디어 노출
   - 전문가 추천사
   - 업계 리더와의 파트너십

3. **투명성 (Transparency)**
   - 명확한 가격 정책 (숨은 비용 없음)
   - 환불 정책 / 무료 체험 기간
   - 실제 사용 사례 및 성과 데이터

`checklists/conversion-checklist.json`을 참조하여 각 요소가 누락되지 않았는지 확인하세요.

## 5단계: CTA 최적화 (Call-to-Action Optimization)

행동 유도 버튼(CTA)은 전환의 핵심입니다. `templates/cta-template.md`를 참조하여 다음을 최적화하세요:

- **위치**: 히어로 섹션, 본문 중간, 페이지 끝 최소 3곳
- **문구**: 능동적이고 구체적인 동사 사용
  - 나쁜 예: "시작하기", "자세히 보기"
  - 좋은 예: "14일 무료로 시작", "내 성과 관리 자동화하기"
- **색상**: 페이지 전체 색상과 대비되는 색 (일반적으로 주황색, 빨간색, 초록색 계열)
- **긴급성**: 가능하면 시간 제약이나 희소성 추가
  - 예: "오늘만 20% 할인", "선착순 100명"

## 출력 형식

5단계를 모두 완료하면, 다음과 같은 구조화된 기획서를 Markdown 형식으로 출력하세요:

```markdown
# [프로젝트명] 랜딩 페이지 기획서

## 1. 프로젝트 목표
[1단계 결과]

## 2. 사용자 진입 맥락
[2단계 결과]

## 3. 히어로 섹션
### 메인 헤드라인
[헤드라인]

### 서브 헤드라인
[서브헤드라인]

### 비주얼
[비주얼 설명]

### CTA
버튼 문구: [문구]
버튼 색상: [색상]

## 4. 신뢰 요소
- [ ] 고객 리뷰
- [ ] 기업 로고
- [ ] 수치적 지표
- [ ] 전문가 추천
- [ ] 미디어 노출
- [ ] 가격 투명성

## 5. CTA 배치 계획
1. 히어로 섹션: [위치 및 문구]
2. 중간 섹션: [위치 및 문구]
3. 하단 섹션: [위치 및 문구]

## 6. 다음 단계
- [ ] 와이어프레임 제작
- [ ] 카피라이팅 상세화
- [ ] 비주얼 자산 준비
```

## 도구 활용

- 경쟁사 분석이 필요하면 `scripts/competitor-analyzer.py`를 실행하세요
- SEO 최적화가 필요하면 `checklists/seo-checklist.json`을 참조하세요

## 중요 알림

- 사용자가 중간에 방향을 바꾸려 하면, 먼저 현재 단계를 완료한 후 수정하도록 안내하세요
- 모든 결정은 "왜?"라는 질문에 답할 수 있어야 합니다
- 애매한 답변이 나오면 더 구체적인 예시를 요청하세요
~~~

**보조 템플릿 파일 작성**: `templates/hero-section-template.md`

```markdown
# 히어로 섹션 템플릿

## 메인 헤드라인 작성 공식

[동사] + [구체적 결과] + [시간/수치]

예시:
- "3분 만에 완벽한 이력서 생성"
- "월 10시간 절약하는 회계 자동화"
- "코딩 없이 앱 출시까지 단 2주"

## 서브 헤드라인 작성 공식

[타겟 고객]을 위한 [차별점]

예시:
- "스타트업 대표를 위한 AI 기반 급여 계산기"
- "1인 마케터도 쉽게 만드는 랜딩 페이지 빌더"

## CTA 버튼 문구 공식

[행동] + [혜택] (+ [무료/할인])

예시:
- "14일 무료로 시작하기"
- "내 시간 절약 계산해보기"
- "지금 30% 할인받기"
```

**체크리스트 파일 작성**: `checklists/conversion-checklist.json`

```json
{
  "checklist_version": "1.0.0",
  "categories": [
    {
      "name": "히어로 섹션",
      "items": [
        {
          "id": "hero-01",
          "check": "메인 헤드라인이 25자 이내인가?",
          "priority": "high"
        },
        {
          "id": "hero-02",
          "check": "3초 안에 가치 제안이 이해되는가?",
          "priority": "high"
        },
        {
          "id": "hero-03",
          "check": "CTA 버튼이 스크롤 없이 보이는가?",
          "priority": "high"
        }
      ]
    },
    {
      "name": "신뢰 요소",
      "items": [
        {
          "id": "trust-01",
          "check": "최소 3개 이상의 고객 리뷰가 있는가?",
          "priority": "medium"
        },
        {
          "id": "trust-02",
          "check": "사용 기업 로고가 5개 이상 표시되는가?",
          "priority": "medium"
        },
        {
          "id": "trust-03",
          "check": "정량적 지표(사용자 수, 거래량)가 명시되는가?",
          "priority": "high"
        }
      ]
    }
  ]
}
```

### 2-3. Skill 테스트 및 디버깅

Skill을 만들었으면 제대로 작동하는지 테스트해야 합니다.

**기본 테스트 시나리오**는 다음과 같습니다. Claude Desktop을 열고 새 대화를 시작합니다. `/landing-page-planner`를 입력하여 직접 호출합니다. 5단계가 순서대로 진행되는지 확인합니다. 각 단계에서 질문이 명확하게 나오는지 확인합니다. 최종 출력이 Markdown 형식으로 잘 정리되는지 확인합니다.

**자동 호출 테스트**를 해봅니다. "온라인 강의 판매를 위한 랜딩 페이지를 기획하고 싶어"라고 입력하면 auto_invoke: true 설정 덕분에 자동으로 Skill이 활성화되어야 합니다.

**일반적인 문제와 해결책**을 보면 다음과 같습니다. Skill이 인식되지 않으면 SKILL.md 파일이 정확히 `.claude/skills/[skill-name]/SKILL.md` 경로에 있는지 확인하고, YAML frontmatter가 `---`로 제대로 감싸져 있는지 확인하며, Claude Desktop을 재시작합니다. Skill이 중간에 멈추면 SKILL.md의 지침이 너무 모호한지 확인하고, "반드시", "필수", "순서대로" 같은 강한 지시어를 추가하며, 각 단계 끝에 명확한 출력 형식을 명시합니다. 보조 파일이 로드되지 않으면 상대 경로가 정확한지 확인하고 (`templates/`, `checklists/`), 파일 존재 여부를 확인합니다 (`ls -la ~/.claude/skills/landing-page-planner/`).

**디버깅 팁**을 보면 Claude에게 "현재 어떤 Skill이 활성화되어 있나요?"라고 물어보면 활성 Skill 목록을 확인할 수 있습니다. Skill 실행 중 "지금 SKILL.md의 어느 부분을 따르고 있나요?"라고 물어보면 현재 진행 단계를 확인할 수 있습니다. `!cat ~/.claude/skills/landing-page-planner/SKILL.md` 명령으로 Claude가 실제로 읽는 내용을 확인할 수 있습니다.

### 2-4. 팀과 Skills 공유하기

Skills의 진짜 가치는 팀 전체가 사용할 때 나타납니다. 개인의 전문성을 조직의 자산으로 만드는 방법을 알아보겠습니다.

**Git으로 버전 관리하기**는 다음과 같습니다. Skills 저장소를 초기화합니다. `cd ~/.claude/skills`로 이동한 후 `git init`, `git add .`, `git commit -m "Initial commit: landing-page-planner skill"`을 실행합니다. 원격 저장소에 푸시합니다. GitHub나 GitLab에 저장소를 만든 후 `git remote add origin [저장소 URL]`, `git push -u origin main`을 실행합니다.

**팀원들의 설치 방법**은 다음과 같습니다. 저장소를 클론합니다. `cd ~/.claude`, `git clone [저장소 URL] skills`를 실행합니다. Claude Desktop을 재시작하면 자동으로 인식됩니다. 업데이트할 때는 `cd ~/.claude/skills`, `git pull`을 실행합니다.

**회사 표준 Skills 라이브러리 구축**은 다음과 같습니다. 조직 GitHub에 `company-claude-skills` 저장소를 만듭니다. 각 팀별로 디렉토리를 구성합니다 (예: `planning/`, `design/`, `development/`, `qa/`). README.md에 각 Skill의 사용법과 사례를 문서화합니다. 버전 태그를 활용하여 안정 버전을 관리합니다 (`git tag v1.0.0`, `git push --tags`).

**Skills 공유 시 베스트 프랙티스**는 다음과 같습니다. 명확한 네이밍을 사용합니다 (예: `landing-page-planner`가 `lp-plan`보다 명확). 상세한 description을 작성하여 언제 사용하는지 명시합니다. 버전 정보를 frontmatter에 명시합니다 (`version: 1.0.0`). 예제를 포함합니다 (SKILL.md 하단에 사용 예시 추가). 업데이트 로그를 유지합니다 (CHANGELOG.md 파일 관리).

## Part 3. Google Antigravity 설정 및 활용

### 3-1. Antigravity 설치 및 기본 설정

Antigravity는 2025년 11월 출시된 신기술이므로 설치와 초기 설정이 중요합니다.

**시스템 요구사항**은 다음과 같습니다. RAM은 최소 8GB, 권장 16GB입니다. 디스크 공간은 초기 설치 2GB, 프로젝트당 500MB-2GB가 필요합니다. 운영체제는 macOS 11+, Windows 10+, Linux (Ubuntu 20.04+)를 지원합니다. Google 계정이 필수이며 무료 티어 사용 가능합니다.

**설치 단계**는 다음과 같습니다. antigravity.google/download로 이동하여 운영체제에 맞는 설치 파일을 다운로드합니다. 설치 파일을 실행하고 Google 계정으로 로그인합니다. 초기 설정 마법사가 나타나면 테마를 선택하고, 사용 목적을 선택합니다 (권장: Review-Driven Development). 작업 폴더를 지정합니다 (예: `~/antigravity-projects`).

**인터페이스 이해하기**는 다음과 같습니다. 5개 주요 영역이 있습니다. Editor Panel은 중앙의 코드 편집 영역으로 VS Code와 동일한 편집 경험을 제공합니다. Agent Sidebar는 우측의 에이전트 대화 패널로 작업 지시 및 피드백을 주고받습니다. Terminal Panel은 하단의 터미널로 bash 명령을 실행합니다. Browser Panel은 좌측의 브라우저 프리뷰로 UI 자동 테스트 및 스크린샷을 제공합니다. Manager View는 상단 탭으로 여러 에이전트를 병렬 관리합니다.

**첫 프로젝트 만들기**는 다음과 같습니다. Antigravity를 열고 "Open Workspace"를 클릭합니다. 새 폴더를 만들거나 기존 프로젝트 폴더를 선택합니다. Agent Sidebar에서 "간단한 HTML 페이지를 만들어줘. 헤더, 본문, 푸터 구조로."라고 입력합니다. 에이전트가 자동으로 index.html을 생성하고 Browser Panel에서 미리보기를 보여줍니다. 코드를 검토하고 "헤더 색상을 파란색으로 바꿔줘"라고 추가 요청합니다.

### 3-2. Antigravity의 Skills 시스템 활용

Antigravity는 Claude Code와 다른 방식으로 Skills를 관리합니다. Google은 이를 "Knowledge Base"라고 부르며, 프로젝트별로 학습하는 방식입니다.

**Antigravity Skills의 특징**은 다음과 같습니다. 프로젝트 컨텍스트 학습을 통해 작업을 진행하면서 에이전트가 자동으로 유용한 패턴을 저장합니다. 코드 스니펫 저장을 통해 자주 사용하는 코드 블록을 재사용합니다. 작업 이력 기반 제안을 통해 이전에 성공한 작업 방식을 반복할 때 자동 제안합니다. Claude Code Skills와의 호환성을 갖추고 있어 SKILL.md 파일도 읽을 수 있습니다 (실험적 기능).

**Antigravity에서 Skills 등록하기**는 다음과 같습니다. Claude Code에서 만든 Skills 폴더를 Antigravity 프로젝트에 복사합니다. `cp -r ~/.claude/skills/landing-page-planner ~/antigravity-projects/my-project/.antigravity/skills/`로 복사합니다. Antigravity를 재시작하면 자동으로 인식합니다. Agent Sidebar에서 "사용 가능한 skills를 보여줘"라고 물어보면 등록된 Skills 목록이 나타납니다.

**Antigravity의 자동 학습 활용하기**는 다음과 같습니다. 작업을 완료한 후 Agent에게 "이 작업 방식을 기억해줘. 다음에 비슷한 요청이 오면 같은 방식으로 해줘"라고 말하면 Knowledge Base에 자동 저장됩니다. 나중에 유사한 작업을 요청하면 에이전트가 "저번에 이렇게 했었는데, 같은 방식으로 할까요?"라고 물어봅니다.

### 3-3. Manager View로 멀티 에이전트 관리하기

Antigravity의 진짜 힘은 여러 작업을 동시에 처리하는 Manager View에 있습니다.

**Manager View 전환하기**는 상단 탭에서 "Editor"에서 "Manager"로 전환하거나 단축키 `Ctrl+Shift+M` (Windows/Linux) 또는 `Cmd+Shift+M` (Mac)을 사용합니다.

**에이전트 생성 및 작업 할당**은 다음과 같습니다. "New Agent" 버튼을 클릭합니다. 에이전트에 이름과 작업을 부여합니다. 예를 들어, Agent 1 "Frontend-Builder"는 "React 컴포넌트를 만들고 Tailwind로 스타일링"하고, Agent 2 "Backend-API"는 "Node.js Express API 서버 구축"하며, Agent 3 "DB-Schema"는 "PostgreSQL 스키마 설계 및 마이그레이션"합니다. 세 에이전트가 동시에 작업을 시작합니다.

**에이전트 간 의존성 설정**은 다음과 같습니다. Agent 2는 Agent 3가 완료된 후 시작하도록 설정할 수 있습니다. "Agent 2, DB 스키마가 완료되면 그걸 기반으로 API 엔드포인트를 만들어줘"라고 지시하면 Antigravity가 자동으로 의존성을 관리합니다.

**진행 상황 모니터링**은 Manager View 대시보드에서 각 에이전트의 상태(대기 중, 작업 중, 완료, 오류), 완료된 서브태스크 목록, 생성된 Artifacts (스크린샷, 로그, 테스트 결과)를 실시간으로 확인할 수 있습니다.

**실전 시나리오: 이커머스 프로젝트**를 보겠습니다. 오전 9시에 "이커머스 웹사이트를 만들어줘. 상품 목록, 장바구니, 결제 기능이 필요해"라고 요청합니다. Antigravity가 자동으로 프로젝트를 분석하고 3개 에이전트를 생성합니다. Agent 1은 상품 목록 페이지, Agent 2는 장바구니 기능, Agent 3은 결제 통합을 담당합니다. 오전 11시에 Manager View를 확인하면 Agent 1이 완료되어 상품 목록 페이지의 스크린샷을 보여주고, Agent 2가 70% 진행 중이며 "장바구니 수량 조절 로직을 추가 중"이라고 보고하고, Agent 3이 대기 중으로 "Agent 2 완료 대기"라고 표시됩니다. 오후 2시에 모든 에이전트가 완료되면 통합 테스트를 자동으로 실행하고 전체 워크스루 영상을 생성합니다.

### 3-4. Antigravity의 브라우저 자동화 활용

Antigravity의 독보적인 기능은 브라우저를 직접 제어하여 UI를 테스트하는 것입니다.

**브라우저 에이전트 활성화**는 Agent Sidebar에서 "Browser" 탭을 클릭하거나 작업 지시에 "브라우저에서 테스트해줘"를 포함시킵니다.

**자동 UI 테스트 시나리오**는 다음과 같습니다. "로그인 폼을 만든 후 실제로 작동하는지 테스트해줘"라고 요청하면 에이전트가 로그인 폼 HTML/CSS/JS를 생성합니다. 브라우저를 자동으로 열어 폼을 렌더링합니다. 테스트 데이터를 입력하고 제출 버튼을 클릭합니다. 검증 로직이 제대로 작동하는지 확인합니다. 스크린샷과 함께 "테스트 완료. 이메일 형식 검증 작동 확인"이라고 보고합니다.

**반응형 디자인 테스트**는 "이 페이지를 모바일, 태블릿, 데스크톱에서 모두 테스트해줘"라고 요청하면 에이전트가 자동으로 뷰포트 크기를 변경하면서 각각 스크린샷을 찍고, 레이아웃 깨짐이 있으면 자동으로 감지하여 수정 제안을 합니다.

**접근성 테스트**는 "이 페이지가 웹 접근성 기준을 만족하는지 확인해줘"라고 요청하면 색상 대비 비율 검사, 키보드 네비게이션 테스트, 스크린 리더 호환성 확인을 자동으로 수행하고 문제점과 해결 방법을 보고합니다.

## Part 4. Claude Code + Antigravity 통합 워크플로우

### 4-1. 언제 어떤 도구를 쓸 것인가?

두 도구를 효과적으로 조합하려면 각각의 최적 사용 시점을 이해해야 합니다.

**Claude Code 사용이 적합한 경우**는 다음과 같습니다. 요구사항이 모호하여 대화를 통해 명확히 해야 할 때, 복잡한 비즈니스 로직을 설계할 때 단계별 검증이 필요한 경우, 기획서나 문서를 작성할 때, Skills를 활용한 표준화된 프로세스 실행이 필요할 때, 코드 리뷰나 리팩토링처럼 섬세한 판단이 필요할 때 사용합니다.

**Antigravity 사용이 적합한 경우**는 다음과 같습니다. 요구사항이 명확하고 대량의 코드를 빠르게 생성해야 할 때, 여러 기능을 동시에 개발해야 하는 병렬 작업이 필요할 때, UI/UX를 실제 브라우저에서 테스트하고 검증해야 할 때, 프론트엔드, 백엔드, DB 등 풀스택 개발이 필요할 때, 시간이 촉박하여 자율적인 에이전트에게 위임해야 할 때 사용합니다.

**의사결정 플로우차트**는 다음과 같습니다. 요구사항이 명확한가? 아니오라면 Claude Code로 대화하며 명확화합니다. 예라면 다음 질문으로 넘어갑니다. 병렬 작업이 가능한가? 예라면 Antigravity Manager View를 사용합니다. 아니오라면 다음 질문으로 넘어갑니다. UI 테스트가 필요한가? 예라면 Antigravity Browser 기능을 사용합니다. 아니오라면 다음 질문으로 넘어갑니다. 표준 프로세스가 있는가? 예라면 Claude Code Skills를 사용합니다. 아니오라면 둘 중 편한 도구를 사용합니다.

### 4-2. 실전 사례: 스타트업 MVP 개발 (48시간 프로젝트)

한 스타트업이 투자 미팅 전에 48시간 안에 작동하는 MVP를 만들어야 하는 상황입니다. Claude Code와 Antigravity를 조합하여 어떻게 완성했는지 살펴보겠습니다.

**Day 1 오전 (기획 단계) - Claude Code 사용**

09:00-10:00 요구사항 정리는 Claude Code를 열고 `/landing-page-planner` Skill을 호출합니다. 30분간 대화하며 핵심 가치 제안, 타겟 사용자, 필수 기능을 명확히 합니다. 출력된 기획서를 Markdown 파일로 저장합니다 (`project-plan.md`).

10:00-11:00 기술 스택 결정은 Claude에게 "이 기획서를 기반으로 최적의 기술 스택을 제안해줘. 48시간 안에 완성 가능해야 하고 비용은 최소화해야 해"라고 요청합니다. Claude가 프론트엔드로 React + Tailwind, 백엔드로 Firebase (인증, DB, 호스팅 통합), 결제로 Stripe를 제안합니다. 각 선택의 이유와 대안을 설명받습니다.

11:00-12:00 프로젝트 구조 설계는 Claude에게 "프로젝트 폴더 구조와 파일 목록을 만들어줘"라고 요청합니다. Claude가 디렉토리 구조, 주요 파일, 각 파일의 역할을 설명한 문서를 생성합니다. 이를 `architecture.md`로 저장하고 Git에 커밋합니다.

**Day 1 오후 (개발 단계) - Antigravity 사용**

13:00-14:00 프로젝트 셋업은 Antigravity를 열고 새 워크스페이스를 생성합니다. Claude Code에서 만든 `architecture.md` 파일을 Antigravity로 가져옵니다. Agent에게 "이 아키텍처 문서를 기반으로 초기 프로젝트 셋업을 해줘. Create React App과 Firebase 설정까지"라고 요청합니다. 에이전트가 자동으로 `npx create-react-app`, Firebase 프로젝트 생성, 환경 변수 설정까지 완료합니다.

14:00-17:00 병렬 개발은 Manager View로 전환하고 3개 에이전트를 생성합니다. Agent 1 "UI-Components"는 "Tailwind로 스타일링된 재사용 가능한 React 컴포넌트 제작"합니다. Agent 2 "Auth-Flow"는 "Firebase Authentication으로 로그인/회원가입 구현"합니다. Agent 3 "Payment-Integration"은 "Stripe 결제 통합 및 테스트"합니다. 3시간 동안 에이전트들이 자율적으로 작업하는 동안 다른 업무를 처리합니다. 중간중간 Manager View에서 진행 상황만 확인합니다.

17:00-18:00 통합 및 테스트는 Antigravity의 Browser 기능을 활성화합니다. "전체 사용자 플로우를 테스트해줘. 회원가입 → 로그인 → 결제까지"라고 요청합니다. 에이전트가 자동으로 각 단계를 실행하고 스크린샷을 찍습니다. 발견된 버그 2개를 즉시 수정합니다.

**Day 2 오전 (문서화 및 배포) - Claude Code 사용**

09:00-10:00 API 문서 작성은 Antigravity에서 만든 코드를 Claude Code로 가져옵니다. Claude에게 "이 코드베이스의 API 문서를 Swagger 형식으로 만들어줘"라고 요청합니다. 자동 생성된 문서를 검토하고 누락된 부분을 수정합니다.

10:00-11:00 사용자 가이드 작성은 `/user-guide-writer` Skill을 호출합니다 (팀에서 미리 만들어둔 커스텀 Skill). 기획서와 코드를 기반으로 사용자 가이드를 자동 생성합니다.

**Day 2 오후 (최종 검증 및 피칭 준비) - 두 도구 혼용**

13:00-15:00 성능 최적화는 Antigravity의 Agent에게 "Lighthouse 성능 테스트를 실행하고 개선점을 찾아줘"라고 요청합니다. 에이전트가 자동으로 테스트를 실행하고 이미지 최적화, 코드 스플리팅 등을 적용합니다.

15:00-17:00 데모 시나리오 준비는 Claude Code로 돌아와 투자자 미팅용 데모 스크립트를 작성합니다. Antigravity로 데모 영상을 자동 녹화합니다 (Browser 기능 활용).

17:00-18:00 최종 점검은 Claude Code의 `/project-review` Skill로 전체 프로젝트를 검토합니다. 보안 취약점, 접근성 문제, 성능 병목을 체크합니다.

**결과**: 48시간 만에 작동하는 MVP 완성. 투자 미팅에서 라이브 데모 성공. 개발자 1명이 보통 2주 걸릴 작업을 2일에 완료했습니다.

### 4-3. 고급 협업 패턴: Skills 파이프라인

두 도구를 정말 효과적으로 사용하려면 Skills를 파이프라인처럼 연결해야 합니다.

**파이프라인 개념**은 다음과 같습니다. 하나의 Skill 출력이 다음 Skill의 입력이 되는 구조입니다. Claude Code에서 기획 → Antigravity에서 개발 → Claude Code에서 검증 → Antigravity에서 배포 순으로 진행됩니다.

**Skills 파이프라인 구축 예시: "웹사이트 제작 자동화 파이프라인"**

Skill 1: Requirements Gatherer (Claude Code)는 사용자와 대화하며 요구사항을 JSON 형식으로 정리합니다. 출력 파일은 `requirements.json`입니다.

Skill 2: Tech Stack Advisor (Claude Code)는 `requirements.json`을 읽어 최적 기술 스택을 제안합니다. 출력 파일은 `tech-stack.md`입니다.

Skill 3: Project Generator (Antigravity)는 `tech-stack.md`를 기반으로 프로젝트 초기 구조를 생성합니다. 출력 파일은 전체 프로젝트 폴더입니다.

Skill 4: Component Builder (Antigravity)는 프로젝트 폴더에서 필요한 컴포넌트들을 병렬로 생성합니다. 출력 파일은 `/src/components/` 폴더입니다.

Skill 5: Quality Checker (Claude Code)는 생성된 코드를 분석하여 코드 품질, 보안, 성능을 검증합니다. 출력 파일은 `quality-report.md`입니다.

Skill 6: Deployment Automator (Antigravity)는 검증이 완료된 프로젝트를 Vercel이나 Netlify에 자동 배포합니다. 출력 파일은 `deployment-url.txt`입니다.

**파이프라인 실행 방법**은 다음과 같습니다. Claude Code에서 `/pipeline-start` 명령을 실행합니다. 자동으로 Skill 1-2가 순차 실행됩니다. Skill 2가 완료되면 "Antigravity로 전환하시겠습니까?"라고 묻습니다. Antigravity를 열고 생성된 `tech-stack.md` 파일을 로드합니다. Skill 3-4가 자동 실행됩니다. 완료 후 다시 Claude Code로 돌아와 Skill 5를 실행합니다. 최종적으로 Antigravity에서 Skill 6을 실행하여 배포합니다.

**파이프라인 자동화 스크립트**는 bash 스크립트로 만들 수 있습니다.

```bash
#!/bin/bash
# website-pipeline.sh

echo "=== 웹사이트 제작 파이프라인 시작 ==="

# Step 1-2: Claude Code에서 기획 (수동)
echo "Claude Code를 열고 /requirements-gatherer를 실행하세요."
read -p "완료되었으면 Enter를 누르세요..."

# Step 3-4: Antigravity에서 개발 (자동)
echo "Antigravity로 프로젝트 생성 중..."
cd ~/antigravity-projects/
antigravity-cli create-project --from requirements.json

# Step 5: Claude Code에서 검증 (자동)
echo "품질 검증 중..."
claude-cli run-skill quality-checker --target ./new-project

# Step 6: Antigravity에서 배포 (자동)
echo "배포 중..."
antigravity-cli deploy --target ./new-project --platform vercel

echo "=== 파이프라인 완료! ==="
cat deployment-url.txt
```

이 스크립트를 실행하면 전체 과정이 자동화되어 인간은 초기 기획 대화만 하고 나머지는 AI가 처리합니다.

## Part 5. 고급 활용: Skills 최적화 및 보안

### 5-1. Skills 성능 최적화

Skills가 많아지면 Claude의 응답 속도가 느려질 수 있습니다. 최적화 방법을 알아봅시다.

**문제: 너무 많은 Skills**는 Claude Code가 시작할 때 모든 Skills를 시스템 프롬프트에 로드하기 때문에 Skills가 50개 이상이면 초기 로딩이 느려집니다. 해결책은 자주 사용하는 Skills만 개인 폴더에 두고 나머지는 별도 저장소에서 필요할 때만 가져오는 것입니다. 또한 `auto_invoke: false`를 기본값으로 설정하여 명시적으로 호출할 때만 로드되게 합니다.

**문제: 너무 긴 SKILL.md**는 SKILL.md가 1000줄 이상이면 토큰 소비가 크고 응답이 느려집니다. 해결책은 핵심 지침만 SKILL.md에 두고 상세 내용은 별도 파일로 분리하는 것입니다. 예를 들어 상세 템플릿은 `templates/` 폴더에, 긴 체크리스트는 JSON 파일로 외부화하며, 예시 코드는 별도 파일로 분리합니다.

**문제: 중복되는 Skills**는 비슷한 기능의 Skills가 여러 개 있으면 Claude가 혼란스러워합니다. 해결책은 공통 로직을 추출하여 Base Skill을 만들고 구체적인 Skills는 Base를 확장하는 방식으로 재구성하는 것입니다.

**최적화 체크리스트**는 다음과 같습니다. SKILL.md가 500줄 이하인가? 보조 파일은 적절히 분리되었는가? auto_invoke는 정말 필요한 경우만 true인가? 비슷한 Skills를 통합할 수 있는가? description이 명확하여 Claude가 적절히 선택할 수 있는가? 입니다.

### 5-2. 보안 고려사항

Skills는 로컬에서 실행되고 시스템 접근 권한이 있으므로 보안에 주의해야 합니다.

**위험 1: 악의적인 스크립트**는 타인이 만든 Skill을 설치할 때 Python 스크립트나 bash 명령이 포함되어 있을 수 있습니다. 해결책은 Skills를 설치하기 전 SKILL.md와 모든 스크립트를 직접 검토하고, 신뢰할 수 있는 출처(공식 저장소, 검증된 팀)의 Skills만 사용하며, 처음 사용 시 샌드박스 환경에서 테스트하는 것입니다.

**위험 2: 민감 정보 노출**은 Skills에 API 키, 비밀번호, 개인정보를 하드코딩하면 Git에 커밋될 위험이 있습니다. 해결책은 민감 정보는 절대 SKILL.md에 직접 쓰지 않고, 환경 변수를 사용하거나 (`os.environ.get('API_KEY')`), 별도 `.env` 파일로 관리하고 `.gitignore`에 추가하는 것입니다.

**위험 3: Antigravity의 터미널 접근**은 Antigravity 에이전트가 터미널에 직접 접근하여 rm -rf / 같은 위험한 명령을 실행할 수 있습니다. 해결책은 Antigravity 설정에서 Terminal Command Auto Execution 정책을 "Ask Before Execute"로 변경하고, Allow List와 Deny List를 설정하여 특정 명령만 허용하며, 중요한 프로젝트는 Git으로 백업하여 언제든 복구 가능하게 하는 것입니다.

**Antigravity 보안 설정 방법**은 Settings → Security → Terminal Permissions로 이동합니다. "Auto Execute Policy"를 "Always Ask"로 변경합니다. Allow List에 안전한 명령을 추가합니다 (예: `npm install`, `git status`, `ls`, `cat`). Deny List에 위험한 명령을 추가합니다 (예: `rm -rf`, `sudo`, `chmod 777`).

**보안 체크리스트**는 다음과 같습니다. Skill 스크립트를 검토했는가? 민감 정보가 하드코딩되지 않았는가? Antigravity 터미널 권한을 제한했는가? 중요 프로젝트는 백업했는가? 팀과 공유하는 Skills는 코드 리뷰를 거쳤는가? 입니다.

### 5-3. Skills 디버깅 및 로깅

Skills가 예상대로 작동하지 않을 때 디버깅 방법입니다.

**로깅 추가하기**는 SKILL.md에 디버그 지침을 추가하는 것입니다.

~~~markdown
---
name: my-skill
debug: true
---

# My Skill

작업을 시작할 때마다 다음을 출력하세요:
```
[DEBUG] Skill 'my-skill' 시작
[DEBUG] 현재 단계: {단계 이름}
```

작업이 완료되면:
```
[DEBUG] Skill 'my-skill' 완료
```
~~~

**Claude에게 디버깅 요청하기**는 "현재 활성화된 Skill이 뭐야?", "지금 SKILL.md의 어느 부분을 따르고 있어?", "이 Skill이 실패한 이유를 설명해줘"라고 물어보는 것입니다.

**스크립트 디버깅**은 Python 스크립트에 로깅을 추가하는 것입니다.

```python
import logging

logging.basicConfig(
    filename='~/.claude/skills/my-skill/debug.log',
    level=logging.DEBUG
)

logging.debug("스크립트 시작")
# ... 코드 ...
logging.debug("스크립트 완료")
```

로그 파일을 확인합니다. `tail -f ~/.claude/skills/my-skill/debug.log`

**Antigravity 에이전트 로그 확인하기**는 Antigravity의 Manager View에서 각 Agent를 클릭하면 상세 로그를 볼 수 있습니다. 로그에는 실행한 명령, 생성한 파일, 에러 메시지, 토큰 사용량이 포함됩니다.

## Part 6. 미래 전망 및 준비

### 6-1. 2026년 AI 코딩 도구의 진화 방향

2025년 하반기에 출시된 Claude Code와 Antigravity는 빠르게 발전하고 있습니다. 2026년에는 어떻게 변화할까요?

**예상 트렌드 1: 완전 자율 에이전트**는 현재는 인간이 작업을 할당하지만, 2026년에는 에이전트가 스스로 우선순위를 판단하고 작업을 계획할 것입니다. 예를 들어 "이커머스 웹사이트를 만들어줘"라고 하면 에이전트가 자동으로 시장 조사, 경쟁사 분석, 기획, 디자인, 개발, 테스트, 배포까지 처음부터 끝까지 수행합니다.

**예상 트렌드 2: Multi-Agent Collaboration**은 Claude Code 에이전트와 Antigravity 에이전트가 직접 통신하며 협업할 것입니다. 예를 들어 Claude Code가 기획을 완료하면 자동으로 Antigravity에게 개발을 넘기고, Antigravity가 개발을 완료하면 Claude Code에게 문서화를 요청하는 식입니다.

**예상 트렌드 3: 음성 및 비전 통합**은 Skills를 음성으로 호출하거나 화면을 보여주면서 "이렇게 만들어줘"라고 요청할 것입니다. Antigravity가 디자이너의 손그림 스케치를 인식하여 HTML로 변환하는 식입니다.

**예상 트렌드 4: 산업별 특화 Skills**는 각 산업에 특화된 전문 Skills가 등장할 것입니다. 예를 들어 의료 분야는 HIPAA 준수 자동 검증, 금융 분야는 금융 규제 준수 체크, 이커머스는 PCI-DSS 보안 기준 적용 같은 것입니다.

### 6-2. 웹 전문가를 위한 준비 전략

AI 도구가 빠르게 발전하는 상황에서 웹기획자와 웹디자이너는 어떻게 준비해야 할까요?

**전략 1: 본질에 집중하라**는 AI가 실행을 담당한다면, 인간은 '무엇을 만들 것인가'와 '왜 만드는가'에 집중해야 합니다. 사용자 리서치, 비즈니스 목표 이해, 전략적 의사결정은 여전히 인간의 영역입니다. 에센스 오브 에센스처럼 복잡한 요구사항에서 핵심을 추출하는 능력을 키우세요.

**전략 2: Skills를 자산으로 축적하라**는 자신의 전문성을 Skills로 패키징하여 조직 자산으로 만드세요. 10년 경력의 노하우를 Skills 10개로 만들면, 신입도 숙련자처럼 일할 수 있습니다. Skills는 퇴사해도 남는 자산이며, 이직 시 포트폴리오로 활용할 수 있습니다.

**전략 3: AI 도구를 두려워하지 말라**는 AI는 여러분을 대체하는 것이 아니라 능력을 증폭시키는 도구입니다. AI를 잘 쓰는 기획자가 그렇지 않은 기획자보다 10배 빠릅니다. 새로운 도구가 나올 때마다 일주일만 투자하여 익히세요. 장기적으로 엄청난 생산성 차이를 만듭니다.

**전략 4: 커뮤니티에 참여하라**는 Skills를 공유하고 피드백을 받으세요. GitHub에 Skills 저장소를 만들어 오픈소스로 공개하면 다른 사람들이 개선해주고 새로운 아이디어를 얻을 수 있습니다. X(Twitter), LinkedIn, 개발자 커뮤니티에서 자신의 경험을 공유하세요.

**전략 5: 평생 학습 마인드**는 2026년의 도구는 2025년과 완전히 다를 것입니다. 특정 도구에 종속되지 말고 'AI와 협업하는 방법'이라는 메타 스킬을 키우세요. Anthropic, Google, OpenAI의 공식 블로그를 주기적으로 확인하세요.

## 맺으며: 에이전트 시대의 웹 전문가

2025년 하반기, 우리는 AI 개발 도구의 역사에서 중요한 전환점을 목격했습니다. Claude Code의 Skills와 Google Antigravity의 Agent-First 아키텍처는 단순한 코드 자동완성을 넘어, AI가 자율적으로 사고하고 실행하는 시대를 열었습니다.

우아한형제들 임동준 님이 제시한 '에센스 오브 에센스' 철학은 이 새로운 시대에 더욱 중요해졌습니다. AI 에이전트가 아무리 강력해도, '무엇이 본질인가'를 정의하는 것은 인간의 몫입니다. 웹기획자와 웹디자이너의 미래는 밝습니다. 여러분은 더 이상 와이어프레임을 수백 개 그리거나 HTML 코드를 한 줄 한 줄 작성할 필요가 없습니다. 대신 프로젝트의 본질을 정의하고, Skills로 자산화하며, AI 에이전트를 오케스트레이션하는 역할로 진화할 것입니다.

이 문서에서 다룬 Claude Code + Antigravity 협업 방법론은 2026년으로 가는 여정의 시작점입니다. 지금 당장 첫 Skill을 만들어보세요. 작은 것부터 시작하여 점차 확장하세요. 여러분의 전문성을 AI가 실행할 수 있는 형태로 만드는 순간, 여러분은 AI 시대의 진정한 프로페셔널이 됩니다.

Skills를 만드는 것은 단순히 작업을 자동화하는 것이 아닙니다. 여러분의 사고방식, 문제 해결 접근법, 심지어 가치관까지 코드화하는 과정입니다. 이것이 바로 '기획 템플릿의 자산화'가 의미하는 바입니다. 여러분의 10년 경력이 10개의 Skills로 패키징되어, 조직 전체가 여러분처럼 일할 수 있게 되는 것입니다.

2026년을 넘어 그 이후에도 살아남을 웹 전문가는 AI 도구를 가장 빨리 배운 사람이 아니라, 프로젝트의 본질을 가장 명확히 정의하고 AI를 효과적으로 지휘할 수 있는 사람일 것입니다. Claude Code와 Antigravity는 그 여정을 함께할 강력한 파트너입니다.

---

**문서 작성 일자: 2026-01-21**

## 부록: 빠른 참조 가이드

### Claude Code Skills 기본 구조
```markdown
---
name: skill-name
description: 간단한 설명
auto_invoke: true/false
---

# Skill 제목

지침 내용...
```

### Antigravity 기본 명령어
- Editor/Manager 전환: `Ctrl+Shift+M`
- 새 Agent 생성: Manager View에서 "New Agent"
- Browser 테스트: "브라우저에서 테스트해줘"

### 문제 해결 체크리스트
- [ ] Skills 경로가 올바른가?
- [ ] YAML frontmatter가 올바른가?
- [ ] Claude Desktop/Antigravity 재시작했는가?
- [ ] 로그 파일을 확인했는가?

### 유용한 리소스
- Claude Code 공식 문서: https://code.claude.com/docs
- Antigravity 공식 사이트: https://antigravity.google
- Skills 오픈 표준: https://github.com/agent-skills-spec
- 커뮤니티 Skills 저장소: (GitHub에서 "claude-skills" 검색)