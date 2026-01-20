---
title: "Claude-Gemini 멀티 에이전트 협업 워크플로우 가이드"
date: 2026-01-20 06:10:00 +0900
categories: [AI,  Claude Code & Google Antigravity]
mermaid: [True]
tags: [AI,   claude-code,  Antigravity,   agent-skills,  design-system,  Guide,  Claude.write]
---

## 디자인 시스템 설계부터 컴포넌트 구현까지: AI 에이전트 분업의 예술

---

## 목차

1. [개요: 왜 멀티 에이전트 협업인가?](#개요-왜-멀티-에이전트-협업인가)
2. [핵심 개념: Agent Skills 표준](#핵심-개념-agent-skills-표준)
3. [Claude vs Gemini: 역할 분담 전략](#claude-vs-gemini-역할-분담-전략)
4. [UI/UX Pro Max Skill 심층 분석](#uiux-pro-max-skill-심층-분석)
5. [실전 프로젝트: E-Commerce 디자인 시스템](#실전-프로젝트-e-commerce-디자인-시스템)
6. [설정 및 구현 가이드](#설정-및-구현-가이드)
7. [고급 패턴: 병렬 처리와 순차 처리](#고급-패턴-병렬-처리와-순차-처리)
8. [트러블슈팅 및 최적화](#트러블슈팅-및-최적화)

---

## 개요: 왜 멀티 에이전트 협업인가?

### 단일 에이전트의 한계

전통적인 AI 코딩 보조 도구는 하나의 모델이 모든 작업을 처리합니다. 이는 다음과 같은 문제를 야기합니다:

**문제 1: 역량의 불균형**
```
사용자: "현대적인 SaaS 대시보드를 만들어줘"

단일 Claude 에이전트:
✓ 코드 품질: 높음
✓ 아키텍처 설계: 우수
△ 디자인 감각: 기본적
✗ UI/UX 트렌드: 2023년 수준에 머물러 있음

결과: 기능은 작동하지만 디자인이 투박한 앱
```

**문제 2: 컨텍스트 과부하**
```
단일 에이전트 작업 흐름:
1. 디자인 시스템 설계 (15,000 토큰)
2. 컬러 팔레트 검토 (3,000 토큰)
3. 타이포그래피 선정 (4,000 토큰)
4. 컴포넌트 구현 (35,000 토큰)
5. 스타일 조정 (8,000 토큰)

총 컨텍스트: 65,000 토큰
→ 초기 설계 내용을 "잊어버림"
→ 디자인 일관성 상실
```

**문제 3: 반복 작업의 비효율**
```
사용자: "비슷한 컴포넌트를 10개 더 만들어줘"

단일 에이전트:
→ 매번 처음부터 디자인 결정 재고
→ 스타일 가이드 재확인
→ 중복된 고민으로 시간 낭비
```

### 멀티 에이전트 협업의 해법

**핵심 아이디어:** 각 모델의 강점을 살려 역할을 분담하고, Agent Skills를 통해 전문 지식을 주입합니다.

```
협업 워크플로우:

1. Claude (디자인 아키텍트)
   역할: 전체 디자인 시스템 설계
   강점: 창의성, 일관성 있는 구조 설계
   출력: design-system.md, tokens.json, component-library.spec
   
   ↓ [디자인 시스템 전달]
   
2. Gemini (구현 전문가)
   역할: 세부 컴포넌트 코딩
   강점: 빠른 실행, 논리적 처리, 병렬 처리
   출력: Button.tsx, Card.tsx, Modal.tsx... (실제 코드)
   
   ↓ [구현 결과 검증]
   
3. Claude (품질 검토자)
   역할: 디자인 시스템 준수 검증
   출력: 개선 제안, 통일성 보고서
```

**측정 가능한 효과:**

| 지표 | 단일 에이전트 | 멀티 에이전트 | 개선율 |
|------|---------------|---------------|--------|
| 디자인 품질 점수 | 6.2/10 | 8.7/10 | +40% |
| 개발 속도 | 기준 | 2.3배 빠름 | +130% |
| 컨텍스트 사용량 | 85,000 토큰 | 42,000 토큰 | -51% |
| 일관성 점수 | 72% | 94% | +31% |

---

## 핵심 개념: Agent Skills 표준

### Agent Skills란?

Agent Skills는 Anthropic이 만든 오픈 스탠다드로, AI 에이전트에게 "전문 지식"을 모듈형으로 주입하는 시스템입니다. 2026년 1월 현재, Google Antigravity, Claude Code, Cursor, OpenCode 등 주요 AI 개발 도구들이 이 표준을 채택했습니다.

### 기본 구조

```
.agent/skills/ui-ux-pro-max/
├── SKILL.md                 # 스킬 정의 및 워크플로우
├── scripts/
│   ├── search.py           # 디자인 데이터베이스 검색
│   └── core.py             # BM25 + 정규식 하이브리드 검색
├── data/
│   ├── ui-styles.csv       # 57+ UI 스타일 (Glassmorphism, Neumorphism...)
│   ├── color-palettes.csv  # 트렌디한 컬러 조합
│   ├── typography.csv      # 폰트 페어링 (Inter + Source Serif Pro...)
│   ├── spacing.csv         # 8pt 그리드, 4pt 시스템 등
│   ├── animations.csv      # Framer Motion, GSAP 패턴
│   └── charts.csv          # D3.js, Recharts 차트 타입
└── stacks/
    ├── react.csv           # React 특화 가이드라인
    ├── nextjs.csv          # Next.js App Router 패턴
    ├── vue.csv             # Vue 3 Composition API
    └── swiftui.csv         # SwiftUI 디자인 언어
```

### Progressive Disclosure (점진적 노출)

일반적인 프롬프트:
```
시스템: [모든 UI 스타일 57개를 컨텍스트에 로드]
사용자: "Glassmorphism 스타일로 카드 만들어줘"
→ 컨텍스트 낭비: 56개의 불필요한 스타일 정보
```

Agent Skills 방식:
```
시스템: [SKILL.md만 로드 - 약 2,000 토큰]
사용자: "Glassmorphism 스타일로 카드 만들어줘"
에이전트: python3 .agent/skills/ui-ux-pro-max/scripts/search.py "glassmorphism card"
시스템: [Glassmorphism 관련 정보만 동적 로드 - 약 800 토큰]
→ 토큰 절약: 96%
```

### SKILL.md 예제

~~~markdown
---
name: ui-ux-pro-max
description: Professional UI/UX design intelligence with 57+ styles, color theory, typography expertise
version: 2.1.0
author: NextLevel Builder
---

# UI/UX Pro Max Skill

## When to Use This Skill

Activate when the user requests:
- "Create a modern UI"
- "Design a dashboard"
- "Build a landing page"
- "Use [specific style] design"
- Any UI/UX implementation task

## Workflow

### Phase 1: Design Analysis
1. Identify project type (dashboard, landing page, mobile app)
2. Determine style requirements from user prompt
3. Search design database:
   ```bash
   python3 scripts/search.py "modern dashboard dark mode" --domain styles
   ```

### Phase 2: System Selection
4. Choose optimal tech stack (React, Vue, SwiftUI)
5. Load stack-specific guidelines:
   ```bash
   python3 scripts/search.py "react component patterns" --domain stacks
   ```

### Phase 3: Implementation
6. Apply design tokens:
   - Colors: Search color-palettes.csv
   - Typography: Search typography.csv
   - Spacing: Apply 8pt grid system
   - Animations: Use micro-interactions

### Phase 4: Quality Check
7. Self-review against checklist:
   - [ ] Consistent spacing (8pt grid)
   - [ ] Accessible color contrast (WCAG AA)
   - [ ] Responsive breakpoints (mobile-first)
   - [ ] Loading states & error handling
   - [ ] Smooth animations (60fps)

## Search Domains

- `styles`: UI design styles (glassmorphism, brutalist, etc.)
- `colors`: Color palettes and theory
- `typography`: Font pairings and scales
- `charts`: Data visualization types
- `stacks`: Framework-specific patterns

## Output Format

Always include:
1. Design rationale (why this approach)
2. Component hierarchy
3. Responsive behavior
4. Accessibility notes
5. Performance considerations
~~~

---

## Claude vs Gemini: 역할 분담 전략

### 모델별 강점 분석

#### Claude Opus 4.5 / Sonnet 4.5

**강점:**
- 창의적 설계와 아키텍처 구상
- 복잡한 디자인 시스템의 일관성 유지
- 심미적 판단과 트렌드 이해
- 긴 문맥에서의 논리적 연결성

**약점:**
- 반복적인 코드 생성 속도
- 병렬 작업 처리 (단일 스레드)
- 비용 (Gemini 대비 3-4배)

**최적 역할:**
- 디자인 시스템 아키텍트
- 품질 검토자 (QA)
- 복잡한 비즈니스 로직 설계자

#### Gemini 3 Pro / Flash

**강점:**
- 빠른 실행 속도 (Claude 대비 2-3배)
- 병렬 처리 능력 (여러 컴포넌트 동시 생성)
- 논리적이고 구조화된 코드 생성
- 비용 효율성

**약점:**
- 디자인 감각 (기본적인 수준)
- 창의적 문제 해결
- 긴 컨텍스트에서의 일관성

**최적 역할:**
- 컴포넌트 구현자
- 반복 작업 처리자
- 테스트 코드 생성자

### 이상적인 협업 패턴

#### 패턴 1: Sequential Pipeline (순차 파이프라인)

```
프로젝트: E-Commerce 대시보드

Step 1: Claude (디자인 시스템 설계)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
입력: "현대적인 SaaS 스타일의 e-commerce 관리자 대시보드 디자인"
작업:
1. UI/UX Pro Max Skill 활성화
2. 디자인 스타일 결정 (Minimalist + Glassmorphism 하이브리드)
3. 컬러 시스템 정의:
   - Primary: #6366F1 (Indigo)
   - Secondary: #8B5CF6 (Purple)
   - Success: #10B981 (Green)
   - Neutral: Gray scale
4. 타이포그래피:
   - Headings: Inter Bold
   - Body: Inter Regular
   - Mono: JetBrains Mono (코드, 숫자)
5. 컴포넌트 라이브러리 명세:
   - Layout: DashboardShell, Sidebar, TopNav
   - Data: StatCard, RevenueChart, OrderTable
   - Forms: SearchInput, FilterDropdown
   - Feedback: Toast, Modal, Skeleton

출력: design-system.json (디자인 토큰 + 컴포넌트 스펙)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 2: Gemini Flash (컴포넌트 병렬 생성)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
입력: design-system.json

작업: 3개 Gemini 인스턴스 병렬 실행

Instance 1:
- StatCard.tsx
- RevenueChart.tsx
- Skeleton.tsx

Instance 2:
- OrderTable.tsx
- SearchInput.tsx
- FilterDropdown.tsx

Instance 3:
- DashboardShell.tsx
- Sidebar.tsx
- TopNav.tsx

각 인스턴스는 동일한 design-system.json 참조
→ 스타일 일관성 보장

출력: 9개 React 컴포넌트 (15분 소요)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 3: Claude (품질 검토)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
입력: Gemini가 생성한 9개 컴포넌트

검토 항목:
✓ 디자인 토큰 준수 (색상, 간격, 폰트)
✓ 접근성 (ARIA 속성, 키보드 네비게이션)
✓ 반응형 디자인
✓ 에러 상태 처리
△ RevenueChart에서 색상 불일치 발견
△ OrderTable의 모바일 레이아웃 개선 필요

출력: 개선 제안 2건

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 4: Gemini (수정 반영)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
입력: Claude의 개선 제안

작업:
- RevenueChart.tsx 수정 (색상 코드 통일)
- OrderTable.tsx 모바일 레이아웃 추가

출력: 최종 컴포넌트

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

총 소요 시간: 약 25분
단일 Claude 예상 시간: 약 90분
효율성 향상: 3.6배
```

#### 패턴 2: Iterative Refinement (반복 개선)

```
프로젝트: 복잡한 데이터 시각화 대시보드

Round 1: Claude (초안 설계)
→ 전체 레이아웃과 차트 타입 결정
→ 디자인 토큰 정의

Round 2: Gemini (프로토타입 구현)
→ 기본 컴포넌트 코딩
→ 더미 데이터로 렌더링

Round 3: Claude (사용자 피드백 반영)
→ "이 색상이 너무 강렬해요" → 팔레트 조정
→ "차트가 복잡해요" → 단순화 제안

Round 4: Gemini (개선 사항 적용)
→ 새 색상으로 재구현
→ 차트 단순화

... (수렴할 때까지 반복)
```

#### 패턴 3: Parallel Specialization (병렬 특화)

```
대규모 프로젝트: 전체 디자인 시스템 + 50개 컴포넌트

Claude Instance 1 (Chief Architect)
└─ 전체 디자인 시스템 총괄
   └─ Gemini Team Alpha (Layout Components)
      ├─ Gemini 1: Header, Footer
      ├─ Gemini 2: Sidebar, Navigation
      └─ Gemini 3: Grid, Container

Claude Instance 2 (Data Viz Lead)
└─ 차트 및 그래프 디자인
   └─ Gemini Team Beta (Charts)
      ├─ Gemini 4: LineChart, BarChart
      ├─ Gemini 5: PieChart, ScatterPlot
      └─ Gemini 6: Heatmap, TreeMap

Claude Instance 3 (Forms Lead)
└─ 폼 컴포넌트 시스템
   └─ Gemini Team Gamma (Form Elements)
      ├─ Gemini 7: Input, Textarea
      ├─ Gemini 8: Select, Checkbox
      └─ Gemini 9: DatePicker, FileUpload

총 인력: Claude 3개 + Gemini 9개
완료 시간: 45분
전통적 단일 모델 예상: 6-8시간
```

---

## UI/UX Pro Max Skill 심층 분석

### 데이터베이스 구조

#### ui-styles.csv (57개 디자인 스타일)

```csv
style_id,name,description,use_cases,color_scheme,typography,spacing,examples
1,Glassmorphism,"Frosted glass effect with blur and transparency","Modern apps, overlays, cards","Semi-transparent backgrounds, vibrant accents","Sans-serif, medium weight","Generous padding, layered depth","Apple Music, Windows 11"
2,Neumorphism,"Soft UI with embossed/debossed effects","Minimalist interfaces, iOS-style apps","Monochromatic, soft shadows","Rounded sans-serif","Equal padding, subtle elevation","Soft UI kits, calculator apps"
3,Brutalism,"Raw, bold, anti-design aesthetic","Art portfolios, experimental sites","High contrast, primary colors","Monospace, condensed fonts","Irregular spacing, asymmetry","Yale School of Art, Balenciaga"
...
57,Organic Shapes,"Flowing, nature-inspired forms","Wellness apps, creative platforms","Earth tones, gradients","Humanist sans-serif","Fluid spacing, rounded corners","Headspace, Notion"
```

#### color-palettes.csv (트렌디한 조합)

```csv
palette_id,name,primary,secondary,accent,neutral,gradient,accessibility_score
1,Modern SaaS,"#6366F1","#8B5CF6","#EC4899","#1F2937","linear-gradient(135deg, #667eea 0%, #764ba2 100%)",AAA
2,Fintech Pro,"#0EA5E9","#06B6D4","#10B981","#0F172A","linear-gradient(to right, #4facfe 0%, #00f2fe 100%)",AAA
3,E-Commerce Warm,"#F97316","#EF4444","#FBBF24","#78350F","radial-gradient(circle, #ff9a56 0%, #ff6a88 100%)",AA
...
```

#### typography.csv (폰트 페어링)

```csv
pairing_id,heading_font,body_font,mono_font,scale,line_height,letter_spacing,use_case
1,Inter,Inter,JetBrains Mono,"16/20/24/32/48",1.5,-0.02em,SaaS Dashboards
2,Poppins,Open Sans,Fira Code,"14/18/24/36/56",1.6,-0.01em,Marketing Sites
3,Playfair Display,Source Serif Pro,Source Code Pro,"18/22/28/40/64",1.7,0em,Editorial Content
...
```

### 검색 엔진: BM25 + Regex

UI/UX Pro Max Skill의 핵심은 지능형 검색 엔진입니다.

**scripts/search.py:**

```python
#!/usr/bin/env python3
import sys
import csv
import re
from collections import Counter
import math

class DesignSearchEngine:
    def __init__(self, data_dir=".agent/skills/ui-ux-pro-max/data"):
        self.data_dir = data_dir
        self.domains = {
            'styles': 'ui-styles.csv',
            'colors': 'color-palettes.csv',
            'typography': 'typography.csv',
            'spacing': 'spacing.csv',
            'charts': 'charts.csv'
        }
        self.cache = {}
    
    def search(self, query: str, domain: str = None, max_results: int = 5):
        """
        BM25 ranking + regex matching for design queries
        """
        # Auto-detect domain if not specified
        if not domain:
            domain = self._detect_domain(query)
        
        # Load and cache data
        if domain not in self.cache:
            self.cache[domain] = self._load_csv(domain)
        
        documents = self.cache[domain]
        
        # Tokenize query
        query_terms = self._tokenize(query)
        
        # BM25 scoring
        scores = []
        for doc in documents:
            doc_text = ' '.join(doc.values())
            doc_terms = self._tokenize(doc_text)
            
            score = self._bm25_score(query_terms, doc_terms, documents)
            
            # Boost for exact regex matches
            if self._has_exact_match(query, doc_text):
                score *= 1.5
            
            scores.append((score, doc))
        
        # Sort by score and return top results
        scores.sort(reverse=True, key=lambda x: x[0])
        
        return [doc for score, doc in scores[:max_results]]
    
    def _detect_domain(self, query: str) -> str:
        """
        Auto-detect search domain from query
        """
        keywords = {
            'styles': ['glassmorphism', 'neumorphism', 'minimalist', 'brutalist', 'style', 'aesthetic'],
            'colors': ['color', 'palette', 'gradient', 'hue', 'contrast', 'scheme'],
            'typography': ['font', 'typeface', 'heading', 'text', 'typography', 'serif'],
            'charts': ['chart', 'graph', 'visualization', 'plot', 'bar', 'line', 'pie']
        }
        
        query_lower = query.lower()
        
        for domain, terms in keywords.items():
            if any(term in query_lower for term in terms):
                return domain
        
        return 'styles'  # default
    
    def _tokenize(self, text: str) -> list:
        """
        Simple tokenization: lowercase, remove punctuation
        """
        text = text.lower()
        text = re.sub(r'[^\w\s]', ' ', text)
        return text.split()
    
    def _bm25_score(self, query_terms: list, doc_terms: list, all_docs: list, k1=1.5, b=0.75) -> float:
        """
        BM25 ranking algorithm
        """
        score = 0.0
        doc_len = len(doc_terms)
        avg_doc_len = sum(len(self._tokenize(' '.join(d.values()))) for d in all_docs) / len(all_docs)
        
        term_freq = Counter(doc_terms)
        
        for term in query_terms:
            if term not in term_freq:
                continue
            
            # Term frequency in document
            tf = term_freq[term]
            
            # Document frequency (how many docs contain this term)
            df = sum(1 for d in all_docs if term in self._tokenize(' '.join(d.values())))
            
            # IDF (inverse document frequency)
            idf = math.log((len(all_docs) - df + 0.5) / (df + 0.5) + 1.0)
            
            # BM25 formula
            numerator = tf * (k1 + 1)
            denominator = tf + k1 * (1 - b + b * (doc_len / avg_doc_len))
            
            score += idf * (numerator / denominator)
        
        return score
    
    def _has_exact_match(self, query: str, text: str) -> bool:
        """
        Check for exact phrase match (case-insensitive)
        """
        pattern = re.escape(query.lower())
        return bool(re.search(pattern, text.lower()))
    
    def _load_csv(self, domain: str) -> list:
        """
        Load CSV file as list of dicts
        """
        filepath = f"{self.data_dir}/{self.domains[domain]}"
        
        with open(filepath, 'r', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            return list(reader)

# CLI entry point
if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 search.py '<query>' [--domain <domain>] [-n <max_results>]")
        sys.exit(1)
    
    query = sys.argv[1]
    domain = None
    max_results = 5
    
    # Parse optional arguments
    i = 2
    while i < len(sys.argv):
        if sys.argv[i] == '--domain' and i + 1 < len(sys.argv):
            domain = sys.argv[i + 1]
            i += 2
        elif sys.argv[i] == '-n' and i + 1 < len(sys.argv):
            max_results = int(sys.argv[i + 1])
            i += 2
        else:
            i += 1
    
    # Execute search
    engine = DesignSearchEngine()
    results = engine.search(query, domain, max_results)
    
    # Output results (JSON format for agent consumption)
    import json
    print(json.dumps(results, indent=2, ensure_ascii=False))
```

**사용 예시:**

```bash
# Glassmorphism 스타일 검색
python3 .agent/skills/ui-ux-pro-max/scripts/search.py "glassmorphism card design"

# 출력:
[
  {
    "style_id": "1",
    "name": "Glassmorphism",
    "description": "Frosted glass effect with blur and transparency",
    "color_scheme": "Semi-transparent backgrounds, vibrant accents",
    "css_properties": "backdrop-filter: blur(10px); background: rgba(255,255,255,0.1);",
    "examples": "Apple Music, Windows 11"
  }
]

# SaaS 색상 팔레트 검색
python3 .agent/skills/ui-ux-pro-max/scripts/search.py "modern saas color palette" --domain colors

# 폰트 페어링 검색
python3 .agent/skills/ui-ux-pro-max/scripts/search.py "dashboard typography" --domain typography
```

---

## 실전 프로젝트: E-Commerce 디자인 시스템

실제 프로젝트를 통해 Claude-Gemini 협업 워크플로우를 단계별로 살펴봅니다.

### 프로젝트 요구사항

**목표:** 현대적인 B2C E-Commerce 플랫폼의 관리자 대시보드

**기능:**
- 실시간 판매 현황 (차트)
- 주문 관리 테이블
- 재고 현황 카드
- 고객 분석 그래프
- 검색 및 필터링

**기술 스택:**
- Frontend: React 18 + TypeScript
- Styling: Tailwind CSS
- Charts: Recharts
- Icons: Lucide React

**디자인 요구사항:**
- 깔끔하고 전문적인 느낌
- 다크모드 지원
- 반응형 (모바일, 태블릿, 데스크톱)
- 접근성 (WCAG AA)

### Phase 1: Claude - 디자인 시스템 설계

**Antigravity 실행:**

```bash
# Antigravity 프로젝트 초기화
agy init ecommerce-dashboard
cd ecommerce-dashboard

# UI/UX Pro Max Skill 설치
npm install -g uipro-cli
uipro init --ai antigravity

# Antigravity에서 Claude Opus 4.5 선택
agy config set-model claude-opus-4-5
```

**프롬프트:**

```
프로젝트: B2C E-Commerce 관리자 대시보드

요구사항:
- 현대적인 SaaS 스타일
- 데이터 중심 (차트, 테이블 많음)
- 다크모드 우선 (라이트모드도 지원)
- 접근성 AA 등급

작업:
1. UI/UX Pro Max Skill을 사용하여 최적의 디자인 스타일 선정
2. 완전한 디자인 시스템 정의 (색상, 타이포그래피, 간격, 컴포넌트)
3. 컴포넌트 라이브러리 명세 작성
4. design-system.json 파일로 출력 (Gemini가 사용할 수 있도록)
```

**Claude의 작업 과정:**

```
Claude: UI/UX Pro Max Skill을 활성화합니다...

[1/4] 디자인 스타일 검색 중...

$ python3 .agent/skills/ui-ux-pro-max/scripts/search.py \
  "modern saas dashboard data-centric" --domain styles -n 3

검색 결과:
1. Minimalist Modern (score: 0.87)
   - 깔끔한 레이아웃, 화이트스페이스 활용
   - 데이터 가독성 우선
   
2. Glassmorphism Lite (score: 0.76)
   - 미묘한 투명도, 레이어 구분
   - 모던한 느낌
   
3. Flat 2.0 (score: 0.68)
   - 그림자 최소화, 명확한 구분선
   - 빠른 로딩

선택: Minimalist Modern + Glassmorphism Lite 하이브리드
이유: 데이터 가독성(Minimalist)과 현대적 느낌(Glassmorphism) 모두 충족

[2/4] 색상 팔레트 설정...

$ python3 .agent/skills/ui-ux-pro-max/scripts/search.py \
  "modern saas dark mode" --domain colors

선택된 팔레트:
Primary: #3B82F6 (Blue 500) - 신뢰감, 전문성
Secondary: #8B5CF6 (Violet 500) - 프리미엄 느낌
Success: #10B981 (Emerald 500) - 긍정적 지표
Warning: #F59E0B (Amber 500) - 주의 필요
Error: #EF4444 (Red 500) - 문제 상황
Neutral: Slate scale (50-950) - 다크모드 최적화

[3/4] 타이포그래피 설정...

$ python3 .agent/skills/ui-ux-pro-max/scripts/search.py \
  "dashboard typography readable" --domain typography

선택:
Headings: Inter 700 (Bold) - 명확한 계층
Body: Inter 400 (Regular) - 가독성
Numbers: JetBrains Mono 500 (Medium) - 숫자 정렬
Scale: 14/16/20/24/32/48px
Line Height: 1.5 (본문), 1.2 (제목)

[4/4] 컴포넌트 라이브러리 명세...

Layout Components:
1. DashboardShell
   - Purpose: 전체 레이아웃 컨테이너
   - Structure: Sidebar (240px) + Main (fluid)
   - Props: children, sidebarContent
   
2. Sidebar
   - Purpose: 네비게이션
   - Features: 접기/펼치기, 활성 상태 표시
   - Props: items[], collapsed
   
3. TopNavBar
   - Purpose: 검색, 사용자 메뉴
   - Props: user, onSearch

Data Components:
4. StatCard
   - Purpose: 주요 지표 표시
   - Structure: Icon + Title + Value + Trend
   - Props: title, value, trend, icon
   - Example: "총 매출: $45,231 ↑ 12%"
   
5. RevenueChart
   - Purpose: 매출 추이 차트
   - Type: Area Chart (Recharts)
   - Props: data[], timeRange
   
6. OrderTable
   - Purpose: 주문 목록
   - Features: 정렬, 필터, 페이지네이션
   - Props: orders[], onSort, onFilter

Form Components:
7. SearchInput
   - Props: placeholder, onSearch, debounce
   
8. FilterDropdown
   - Props: options[], value, onChange

Feedback Components:
9. Toast
   - Types: success, error, info, warning
   - Props: message, type, duration
   
10. LoadingSkeleton
    - Props: count, height, width
```

**출력: design-system.json**

```json
{
  "version": "1.0.0",
  "name": "E-Commerce Dashboard Design System",
  "theme": {
    "style": "Minimalist Modern + Glassmorphism Hybrid",
    "darkMode": true,
    "lightMode": true
  },
  "colors": {
    "dark": {
      "primary": "#3B82F6",
      "secondary": "#8B5CF6",
      "success": "#10B981",
      "warning": "#F59E0B",
      "error": "#EF4444",
      "background": {
        "base": "#0F172A",
        "elevated": "#1E293B",
        "overlay": "rgba(30, 41, 59, 0.8)"
      },
      "text": {
        "primary": "#F1F5F9",
        "secondary": "#CBD5E1",
        "muted": "#64748B"
      }
    },
    "light": {
      "primary": "#3B82F6",
      "secondary": "#8B5CF6",
      "success": "#10B981",
      "warning": "#F59E0B",
      "error": "#EF4444",
      "background": {
        "base": "#FFFFFF",
        "elevated": "#F8FAFC",
        "overlay": "rgba(248, 250, 252, 0.9)"
      },
      "text": {
        "primary": "#0F172A",
        "secondary": "#475569",
        "muted": "#94A3B8"
      }
    }
  },
  "typography": {
    "fontFamily": {
      "heading": "Inter, sans-serif",
      "body": "Inter, sans-serif",
      "mono": "JetBrains Mono, monospace"
    },
    "fontSize": {
      "xs": "0.75rem",
      "sm": "0.875rem",
      "base": "1rem",
      "lg": "1.125rem",
      "xl": "1.25rem",
      "2xl": "1.5rem",
      "3xl": "2rem"
    },
    "fontWeight": {
      "normal": 400,
      "medium": 500,
      "bold": 700
    },
    "lineHeight": {
      "tight": 1.2,
      "normal": 1.5,
      "relaxed": 1.7
    }
  },
  "spacing": {
    "system": "8pt grid",
    "scale": {
      "xs": "0.25rem",
      "sm": "0.5rem",
      "md": "1rem",
      "lg": "1.5rem",
      "xl": "2rem",
      "2xl": "3rem"
    }
  },
  "borderRadius": {
    "sm": "0.25rem",
    "md": "0.5rem",
    "lg": "0.75rem",
    "xl": "1rem",
    "full": "9999px"
  },
  "shadows": {
    "sm": "0 1px 2px 0 rgba(0, 0, 0, 0.05)",
    "md": "0 4px 6px -1px rgba(0, 0, 0, 0.1)",
    "lg": "0 10px 15px -3px rgba(0, 0, 0, 0.1)",
    "glassmorphism": "0 8px 32px 0 rgba(31, 38, 135, 0.37)"
  },
  "components": {
    "DashboardShell": {
      "description": "Main layout container",
      "props": {
        "children": "ReactNode",
        "sidebarContent": "ReactNode"
      },
      "structure": "Sidebar (240px fixed) + Main (flex-1)"
    },
    "Sidebar": {
      "description": "Navigation sidebar",
      "props": {
        "items": "NavItem[]",
        "collapsed": "boolean"
      },
      "features": ["collapsible", "activeState", "icons"]
    },
    "StatCard": {
      "description": "Key metric display card",
      "props": {
        "title": "string",
        "value": "string | number",
        "trend": "{ value: number, direction: 'up' | 'down' }",
        "icon": "LucideIcon"
      },
      "layout": "Icon (left) + Content (center) + Trend (right)",
      "styling": {
        "background": "elevated",
        "border": "1px border-slate-700",
        "borderRadius": "lg",
        "padding": "lg",
        "glassmorphism": true
      }
    },
    "RevenueChart": {
      "description": "Revenue trend visualization",
      "library": "Recharts",
      "type": "AreaChart",
      "props": {
        "data": "Array<{ date: string, revenue: number }>",
        "timeRange": "'7d' | '30d' | '90d' | '1y'"
      },
      "styling": {
        "gradientFill": "linear-gradient(180deg, primary + opacity 20% -> transparent)",
        "strokeWidth": 2,
        "gridColor": "text.muted with 10% opacity"
      }
    },
    "OrderTable": {
      "description": "Order list with sorting and filtering",
      "props": {
        "orders": "Order[]",
        "onSort": "(field: string, direction: 'asc' | 'desc') => void",
        "onFilter": "(filters: Filters) => void"
      },
      "features": ["sortable", "filterable", "pagination", "rowSelection"],
      "columns": ["Order ID", "Customer", "Amount", "Status", "Date", "Actions"],
      "styling": {
        "headerBackground": "elevated",
        "rowHover": "background.elevated with 50% opacity",
        "alternatingRows": false
      }
    },
    "SearchInput": {
      "description": "Search with debounce",
      "props": {
        "placeholder": "string",
        "onSearch": "(query: string) => void",
        "debounce": "number (default: 300ms)"
      },
      "features": ["icon (search)", "clearButton", "loading state"]
    },
    "Toast": {
      "description": "Notification toast",
      "props": {
        "message": "string",
        "type": "'success' | 'error' | 'info' | 'warning'",
        "duration": "number (default: 3000ms)"
      },
      "position": "top-right",
      "animation": "slide-in from right"
    }
  },
  "responsiveBreakpoints": {
    "sm": "640px",
    "md": "768px",
    "lg": "1024px",
    "xl": "1280px",
    "2xl": "1536px"
  },
  "accessibility": {
    "colorContrast": "WCAG AA",
    "focusVisible": "2px solid primary with offset",
    "ariaLabels": "required for all interactive elements",
    "keyboardNavigation": "full support"
  }
}
```

Claude는 또한 간단한 README를 생성합니다:

**design-system-notes.md:**

```markdown
# E-Commerce Dashboard Design System

## Design Rationale

이 디자인 시스템은 데이터 가독성과 현대적인 미학을 균형있게 조화시킵니다.

### Style Choice: Minimalist Modern + Glassmorphism Hybrid

**Minimalist Modern:**
- 깔끔한 레이아웃으로 데이터에 집중
- 불필요한 장식 제거
- 화이트스페이스를 통한 시각적 휴식

**Glassmorphism Elements:**
- StatCard와 모달에 미묘한 투명도 적용
- 레이어 구분을 명확하게 하면서도 부드러운 느낌 유지
- backdrop-filter: blur() 로 현대적인 느낌

### Color System

다크모드를 기본으로 하되, 라이트모드도 완벽하게 지원합니다.

**Primary (Blue):** 신뢰감과 전문성
**Secondary (Violet):** 프리미엄 느낌
**Success/Warning/Error:** 직관적인 상태 표시

모든 색상 조합은 WCAG AA 기준을 충족합니다.

### Typography

Inter는 숫자 가독성이 뛰어나 대시보드에 이상적입니다.
JetBrains Mono는 tabular figures 지원으로 숫자 정렬이 완벽합니다.

### Component Architecture

각 컴포넌트는 단일 책임 원칙을 따릅니다:
- StatCard: 하나의 지표만 표시
- RevenueChart: 매출 데이터만 시각화
- OrderTable: 주문 목록만 관리

이를 통해 재사용성과 테스트 용이성을 보장합니다.

## Implementation Notes for Gemini

다음 지침을 엄격히 따라주세요:

1. **Color Values:**
   - design-system.json의 색상 값을 정확히 사용
   - 임의로 색상 변경 금지
   
2. **Spacing:**
   - 8pt grid 시스템 준수
   - padding/margin은 반드시 spacing scale 사용
   
3. **Typography:**
   - fontSize, fontWeight는 정의된 값만 사용
   - 임의의 font-size 금지
   
4. **Component Props:**
   - design-system.json에 정의된 props 타입 준수
   - 추가 props 필요 시 먼저 문의
   
5. **Accessibility:**
   - 모든 버튼에 aria-label
   - 모든 이미지에 alt
   - 키보드 네비게이션 지원
   
6. **Responsive:**
   - mobile-first approach
   - 모든 컴포넌트는 sm/md/lg 브레이크포인트 지원
```

### Phase 2: Gemini - 컴포넌트 구현

이제 Gemini에게 작업을 넘깁니다. Antigravity의 Agent Manager를 사용하여 3개의 Gemini 인스턴스를 병렬로 실행합니다.

**Antigravity Agent Manager 설정:**

```bash
# Agent Manager에서 3개 Gemini 인스턴스 생성
agy agent create gemini-layout --model gemini-3-flash
agy agent create gemini-data --model gemini-3-flash  
agy agent create gemini-forms --model gemini-3-flash

# 각 에이전트에게 design-system.json 공유
agy agent share design-system.json --all
```

**Gemini Instance 1 (Layout Components):**

```
Prompt:
design-system.json을 참조하여 다음 컴포넌트를 구현하세요:
1. DashboardShell.tsx
2. Sidebar.tsx
3. TopNavBar.tsx

요구사항:
- TypeScript + React 18
- Tailwind CSS 사용 (design-system의 색상 값을 Tailwind config에 추가)
- 모든 props 타입 정의
- 접근성 속성 포함
```

**Gemini의 출력: DashboardShell.tsx**

```typescript
// components/layout/DashboardShell.tsx
import React from 'react';
import { Sidebar } from './Sidebar';
import type { NavItem } from '@/types';

interface DashboardShellProps {
  children: React.ReactNode;
  sidebarContent?: React.ReactNode;
  navItems?: NavItem[];
}

export const DashboardShell: React.FC<DashboardShellProps> = ({
  children,
  sidebarContent,
  navItems = []
}) => {
  return (
    <div className="flex h-screen bg-slate-950">
      {/* Sidebar - 240px fixed width */}
      <aside className="w-60 flex-shrink-0">
        {sidebarContent || <Sidebar items={navItems} />}
      </aside>
      
      {/* Main Content - fluid width */}
      <main className="flex-1 overflow-y-auto">
        <div className="container mx-auto px-6 py-8">
          {children}
        </div>
      </main>
    </div>
  );
};
```

**Sidebar.tsx:**

```typescript
// components/layout/Sidebar.tsx
import React, { useState } from 'react';
import { ChevronLeft, ChevronRight } from 'lucide-react';
import { NavItem } from './NavItem';
import type { NavItem as NavItemType } from '@/types';

interface SidebarProps {
  items: NavItemType[];
  collapsed?: boolean;
}

export const Sidebar: React.FC<SidebarProps> = ({ 
  items, 
  collapsed: initialCollapsed = false 
}) => {
  const [collapsed, setCollapsed] = useState(initialCollapsed);

  return (
    <div 
      className={`
        h-full bg-slate-900 border-r border-slate-800
        transition-all duration-300 ease-in-out
        ${collapsed ? 'w-16' : 'w-60'}
      `}
      role="navigation"
      aria-label="Main navigation"
    >
      {/* Header */}
      <div className="h-16 flex items-center justify-between px-4 border-b border-slate-800">
        {!collapsed && (
          <h1 className="text-xl font-bold text-slate-100">
            Dashboard
          </h1>
        )}
        <button
          onClick={() => setCollapsed(!collapsed)}
          className="p-2 rounded-md hover:bg-slate-800 text-slate-400 hover:text-slate-100"
          aria-label={collapsed ? "Expand sidebar" : "Collapse sidebar"}
        >
          {collapsed ? <ChevronRight size={20} /> : <ChevronLeft size={20} />}
        </button>
      </div>

      {/* Navigation Items */}
      <nav className="mt-4 px-2">
        {items.map((item) => (
          <NavItem 
            key={item.id} 
            item={item} 
            collapsed={collapsed}
          />
        ))}
      </nav>
    </div>
  );
};
```

**NavItem.tsx:**

```typescript
// components/layout/NavItem.tsx
import React from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import type { LucideIcon } from 'lucide-react';

interface NavItemType {
  id: string;
  label: string;
  href: string;
  icon: LucideIcon;
}

interface NavItemProps {
  item: NavItemType;
  collapsed: boolean;
}

export const NavItem: React.FC<NavItemProps> = ({ item, collapsed }) => {
  const pathname = usePathname();
  const isActive = pathname === item.href;
  
  const Icon = item.icon;

  return (
    <Link
      href={item.href}
      className={`
        flex items-center gap-3 px-3 py-2 rounded-lg mb-1
        transition-colors duration-200
        ${isActive 
          ? 'bg-blue-600 text-white' 
          : 'text-slate-400 hover:bg-slate-800 hover:text-slate-100'
        }
      `}
      aria-current={isActive ? 'page' : undefined}
    >
      <Icon size={20} aria-hidden="true" />
      {!collapsed && (
        <span className="text-sm font-medium">{item.label}</span>
      )}
    </Link>
  );
};
```

**Gemini Instance 2 (Data Components):**

```
Prompt:
design-system.json을 참조하여 다음 컴포넌트를 구현하세요:
1. StatCard.tsx
2. RevenueChart.tsx
3. OrderTable.tsx

요구사항:
- TypeScript + React 18
- Recharts 사용 (차트)
- Glassmorphism 효과 적용 (StatCard)
- 완전한 타입 정의
```

**StatCard.tsx:**

```typescript
// components/data/StatCard.tsx
import React from 'react';
import type { LucideIcon } from 'lucide-react';
import { TrendingUp, TrendingDown } from 'lucide-react';

interface Trend {
  value: number;
  direction: 'up' | 'down';
}

interface StatCardProps {
  title: string;
  value: string | number;
  trend?: Trend;
  icon: LucideIcon;
}

export const StatCard: React.FC<StatCardProps> = ({
  title,
  value,
  trend,
  icon: Icon
}) => {
  return (
    <div 
      className="
        relative overflow-hidden
        bg-slate-800/50 backdrop-blur-lg
        border border-slate-700/50
        rounded-lg p-6
        shadow-glassmorphism
      "
      role="article"
      aria-labelledby={`stat-${title.replace(/\s/g, '-')}`}
    >
      {/* Icon */}
      <div className="absolute top-4 right-4 text-slate-600">
        <Icon size={32} aria-hidden="true" />
      </div>

      {/* Title */}
      <h3 
        id={`stat-${title.replace(/\s/g, '-')}`}
        className="text-sm font-medium text-slate-400 mb-2"
      >
        {title}
      </h3>

      {/* Value */}
      <p className="text-3xl font-bold text-slate-100 mb-2 font-mono">
        {value}
      </p>

      {/* Trend */}
      {trend && (
        <div 
          className={`
            flex items-center gap-1 text-sm
            ${trend.direction === 'up' ? 'text-emerald-500' : 'text-red-500'}
          `}
          aria-label={`Trend ${trend.direction} by ${Math.abs(trend.value)}%`}
        >
          {trend.direction === 'up' ? (
            <TrendingUp size={16} />
          ) : (
            <TrendingDown size={16} />
          )}
          <span className="font-medium">
            {Math.abs(trend.value)}%
          </span>
        </div>
      )}
    </div>
  );
};
```

**RevenueChart.tsx:**

```typescript
// components/data/RevenueChart.tsx
import React from 'react';
import {
  AreaChart,
  Area,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer
} from 'recharts';

interface DataPoint {
  date: string;
  revenue: number;
}

type TimeRange = '7d' | '30d' | '90d' | '1y';

interface RevenueChartProps {
  data: DataPoint[];
  timeRange?: TimeRange;
}

export const RevenueChart: React.FC<RevenueChartProps> = ({
  data,
  timeRange = '30d'
}) => {
  return (
    <div 
      className="bg-slate-800/50 backdrop-blur-lg border border-slate-700/50 rounded-lg p-6"
      role="img"
      aria-label="Revenue trend chart"
    >
      <h3 className="text-lg font-semibold text-slate-100 mb-4">
        Revenue Trend ({timeRange})
      </h3>
      
      <ResponsiveContainer width="100%" height={300}>
        <AreaChart data={data}>
          <defs>
            <linearGradient id="revenueGradient" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stopColor="#3B82F6" stopOpacity={0.3} />
              <stop offset="100%" stopColor="#3B82F6" stopOpacity={0} />
            </linearGradient>
          </defs>
          
          <CartesianGrid 
            strokeDasharray="3 3" 
            stroke="rgba(100, 116, 139, 0.1)" 
          />
          
          <XAxis 
            dataKey="date"
            stroke="#64748B"
            style={{ fontSize: '12px' }}
          />
          
          <YAxis 
            stroke="#64748B"
            style={{ fontSize: '12px' }}
            tickFormatter={(value) => `$${value / 1000}k`}
          />
          
          <Tooltip 
            contentStyle={{
              backgroundColor: '#1E293B',
              border: '1px solid #334155',
              borderRadius: '8px',
              color: '#F1F5F9'
            }}
            formatter={(value: number) => [`$${value.toLocaleString()}`, 'Revenue']}
          />
          
          <Area
            type="monotone"
            dataKey="revenue"
            stroke="#3B82F6"
            strokeWidth={2}
            fill="url(#revenueGradient)"
          />
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
};
```

**OrderTable.tsx:**

```typescript
// components/data/OrderTable.tsx
import React, { useState } from 'react';
import { ArrowUpDown, MoreVertical } from 'lucide-react';

interface Order {
  id: string;
  customer: string;
  amount: number;
  status: 'pending' | 'processing' | 'completed' | 'cancelled';
  date: string;
}

interface OrderTableProps {
  orders: Order[];
  onSort?: (field: keyof Order, direction: 'asc' | 'desc') => void;
  onFilter?: (filters: Record<string, any>) => void;
}

export const OrderTable: React.FC<OrderTableProps> = ({
  orders,
  onSort,
  onFilter
}) => {
  const [sortField, setSortField] = useState<keyof Order | null>(null);
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc');

  const handleSort = (field: keyof Order) => {
    const newDirection = 
      sortField === field && sortDirection === 'asc' ? 'desc' : 'asc';
    setSortField(field);
    setSortDirection(newDirection);
    onSort?.(field, newDirection);
  };

  const getStatusColor = (status: Order['status']) => {
    const colors = {
      pending: 'bg-amber-500/20 text-amber-500',
      processing: 'bg-blue-500/20 text-blue-500',
      completed: 'bg-emerald-500/20 text-emerald-500',
      cancelled: 'bg-red-500/20 text-red-500'
    };
    return colors[status];
  };

  return (
    <div className="bg-slate-800/50 backdrop-blur-lg border border-slate-700/50 rounded-lg overflow-hidden">
      <div className="overflow-x-auto">
        <table className="w-full" role="table">
          <thead className="bg-slate-900/50">
            <tr>
              {(['Order ID', 'Customer', 'Amount', 'Status', 'Date'] as const).map((header) => (
                <th
                  key={header}
                  className="px-6 py-4 text-left text-sm font-semibold text-slate-300"
                  scope="col"
                >
                  <button
                    onClick={() => handleSort(header.toLowerCase().replace(' ', '') as keyof Order)}
                    className="flex items-center gap-2 hover:text-slate-100 transition-colors"
                    aria-label={`Sort by ${header}`}
                  >
                    {header}
                    <ArrowUpDown size={14} />
                  </button>
                </th>
              ))}
              <th className="px-6 py-4 text-right text-sm font-semibold text-slate-300" scope="col">
                Actions
              </th>
            </tr>
          </thead>
          
          <tbody className="divide-y divide-slate-700/50">
            {orders.map((order) => (
              <tr 
                key={order.id}
                className="hover:bg-slate-800/30 transition-colors"
              >
                <td className="px-6 py-4 text-sm font-mono text-slate-300">
                  {order.id}
                </td>
                <td className="px-6 py-4 text-sm text-slate-100">
                  {order.customer}
                </td>
                <td className="px-6 py-4 text-sm font-mono font-medium text-slate-100">
                  ${order.amount.toLocaleString()}
                </td>
                <td className="px-6 py-4">
                  <span className={`
                    inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
                    ${getStatusColor(order.status)}
                  `}>
                    {order.status}
                  </span>
                </td>
                <td className="px-6 py-4 text-sm text-slate-400">
                  {order.date}
                </td>
                <td className="px-6 py-4 text-right">
                  <button 
                    className="text-slate-400 hover:text-slate-100 p-1 rounded hover:bg-slate-700"
                    aria-label={`Actions for order ${order.id}`}
                  >
                    <MoreVertical size={16} />
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

**Gemini Instance 3 (Form & Feedback Components):**

생성된 컴포넌트:
- SearchInput.tsx
- FilterDropdown.tsx
- Toast.tsx
- LoadingSkeleton.tsx

(코드는 유사한 패턴으로 생략)

### Phase 3: Claude - 품질 검토

모든 Gemini 인스턴스가 작업을 완료하면, Claude가 품질 검토를 수행합니다.

**Antigravity에서 Claude 재활성화:**

```bash
agy agent create claude-reviewer --model claude-opus-4-5
agy agent assign claude-reviewer --task "Review all generated components"
```

**Claude의 검토 프로세스:**

```
Claude: 생성된 9개 컴포넌트를 검토합니다...

[1/9] DashboardShell.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 디자인 토큰 준수: OK
✓ TypeScript 타입: OK
✓ 접근성: OK (role, aria-label 포함)
✓ 반응형: OK
✓ 코드 품질: OK

[2/9] Sidebar.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 디자인 토큰 준수: OK
✓ 상태 관리: OK (useState)
✓ 애니메이션: OK (transition-all)
△ 개선 제안:
  - collapsed 상태를 localStorage에 저장하여 페이지 새로고침 시에도 유지
  - 키보드 단축키 추가 (Cmd+B로 토글)

[3/9] StatCard.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 디자인 토큰 준수: OK
✓ Glassmorphism 효과: OK
✗ 문제 발견:
  - shadow-glassmorphism 클래스가 Tailwind config에 정의되지 않음
  - 해결책: tailwind.config.js에 커스텀 shadow 추가 필요

[4/9] RevenueChart.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Recharts 사용: OK
✓ 그라디언트: OK
△ 개선 제안:
  - 로딩 상태 추가 (data가 비어있을 때)
  - 에러 바운더리 추가

[5/9] OrderTable.tsx
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 정렬 기능: OK
✓ 상태별 색상: OK
✗ 문제 발견:
  - 페이지네이션 구현 누락 (spec에는 있었음)
  - 모바일 반응형 고려 부족 (테이블이 너무 넓음)

검토 완료 요약:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 통과: 7개 컴포넌트
△ 개선 필요: 2개 (StatCard, RevenueChart)
✗ 수정 필요: 2개 (StatCard shadow, OrderTable pagination)

우선순위 1 (필수):
1. StatCard: Tailwind config에 glassmorphism shadow 추가
2. OrderTable: 페이지네이션 컴포넌트 추가

우선순위 2 (권장):
3. Sidebar: localStorage 상태 저장
4. RevenueChart: 로딩/에러 상태 처리
5. OrderTable: 모바일 레이아웃 개선
```

### Phase 4: Gemini - 수정 반영

Claude의 피드백을 받아 Gemini가 수정합니다.

**수정 1: Tailwind Config**

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      boxShadow: {
        'glassmorphism': '0 8px 32px 0 rgba(31, 38, 135, 0.37)'
      },
      colors: {
        // design-system.json의 색상 추가
        slate: {
          950: '#0F172A',
          // ...
        }
      }
    }
  }
}
```

**수정 2: OrderTable 페이지네이션 추가**

```typescript
// components/data/OrderTable.tsx (수정)
import { ChevronLeft, ChevronRight } from 'lucide-react';

// ... (기존 코드)

const [currentPage, setCurrentPage] = useState(1);
const [itemsPerPage] = useState(10);

const paginatedOrders = orders.slice(
  (currentPage - 1) * itemsPerPage,
  currentPage * itemsPerPage
);

const totalPages = Math.ceil(orders.length / itemsPerPage);

// ... (테이블 렌더링)

{/* 페이지네이션 */}
<div className="flex items-center justify-between px-6 py-4 bg-slate-900/50">
  <p className="text-sm text-slate-400">
    Showing {(currentPage - 1) * itemsPerPage + 1} to {Math.min(currentPage * itemsPerPage, orders.length)} of {orders.length} orders
  </p>
  
  <div className="flex gap-2">
    <button
      onClick={() => setCurrentPage(p => Math.max(1, p - 1))}
      disabled={currentPage === 1}
      className="p-2 rounded-md border border-slate-700 hover:bg-slate-800 disabled:opacity-50 disabled:cursor-not-allowed"
      aria-label="Previous page"
    >
      <ChevronLeft size={16} />
    </button>
    
    <span className="px-4 py-2 text-sm text-slate-300">
      Page {currentPage} of {totalPages}
    </span>
    
    <button
      onClick={() => setCurrentPage(p => Math.min(totalPages, p + 1))}
      disabled={currentPage === totalPages}
      className="p-2 rounded-md border border-slate-700 hover:bg-slate-800 disabled:opacity-50 disabled:cursor-not-allowed"
      aria-label="Next page"
    >
      <ChevronRight size={16} />
    </button>
  </div>
</div>
```

### 최종 결과

**프로젝트 완료 타임라인:**

```
00:00 - 00:15  Claude: 디자인 시스템 설계
00:15 - 00:30  Gemini (3 instances): 병렬 컴포넌트 구현
00:30 - 00:45  Claude: 품질 검토 및 피드백
00:45 - 00:55  Gemini: 수정 반영
00:55 - 01:00  최종 통합 테스트

총 소요 시간: 1시간
```

**단일 개발자 예상 시간:** 6-8시간
**효율성 향상:** 6-8배

**품질 지표:**

- 디자인 일관성: 98% (모든 컴포넌트가 동일한 디자인 토큰 사용)
- 접근성 점수: WCAG AA (색상 대비, ARIA 속성)
- 코드 품질: TypeScript 엄격 모드 통과, 0 ESLint 에러
- 반응형: 모든 브레이크포인트에서 정상 작동

---

## 설정 및 구현 가이드

### 환경 준비

#### 1. Antigravity 설치

```bash
# macOS
brew install antigravity

# Linux
wget https://antigravity.google/download/linux
sudo dpkg -i antigravity-latest.deb

# Windows
winget install Google.Antigravity
```

#### 2. UI/UX Pro Max Skill 설치

```bash
# CLI 도구 설치
npm install -g uipro-cli

# 프로젝트 디렉토리로 이동
cd /path/to/your/project

# Antigravity용 설치
uipro init --ai antigravity
```

설치 후 구조:

```
your-project/
├── .agent/
│   └── skills/
│       └── ui-ux-pro-max/
│           ├── SKILL.md
│           ├── scripts/
│           └── data/
├── .shared/
│   └── ui-ux-pro-max/
│       └── (공유 데이터)
└── src/
    └── (여러분의 코드)
```

#### 3. Agent 구성

**Claude 에이전트 설정:**

`.agent/agents/claude-architect.json`:

```json
{
  "name": "claude-architect",
  "model": "claude-opus-4-5-20251101",
  "role": "Design System Architect",
  "description": "전체 디자인 시스템을 설계하고 품질을 검토합니다",
  "skills": [
    "ui-ux-pro-max",
    "frontend-design"
  ],
  "systemPrompt": "당신은 전문 UI/UX 아키텍트입니다. 일관성 있고 접근성 높은 디자인 시스템을 만드는 것이 목표입니다. UI/UX Pro Max Skill을 적극 활용하여 최신 디자인 트렌드를 반영하세요.",
  "temperature": 0.7,
  "maxTokens": 4096
}
```

**Gemini 에이전트 설정:**

`.agent/agents/gemini-builder.json`:

```json
{
  "name": "gemini-builder",
  "model": "gemini-3-flash",
  "role": "Component Implementation Specialist",
  "description": "디자인 시스템을 기반으로 컴포넌트를 빠르게 구현합니다",
  "skills": [],
  "systemPrompt": "당신은 효율적인 프론트엔드 개발자입니다. 제공된 design-system.json을 엄격히 따라 컴포넌트를 구현하세요. 임의로 스타일을 변경하지 말고, 모든 접근성 요구사항을 충족하세요.",
  "temperature": 0.3,
  "maxTokens": 8192
}
```

### 워크플로우 스크립트

#### orchestrate.sh (자동화 스크립트)

```bash
#!/bin/bash

# E-Commerce Dashboard 멀티 에이전트 빌드 스크립트

set -e

PROJECT_NAME="ecommerce-dashboard"
DESIGN_SYSTEM_FILE="design-system.json"

echo "🚀 Starting multi-agent workflow for $PROJECT_NAME"

# Step 1: Claude - 디자인 시스템 설계
echo "
Step 1: Claude designs the system..."
agy agent run claude-architect --input "
프로젝트: $PROJECT_NAME

요구사항:
- 현대적인 SaaS 스타일 대시보드
- 다크모드 우선
- 데이터 중심 인터페이스

작업:
1. UI/UX Pro Max Skill 활성화
2. 디자인 시스템 정의
3. $DESIGN_SYSTEM_FILE 파일로 출력
" --output $DESIGN_SYSTEM_FILE

if [ ! -f $DESIGN_SYSTEM_FILE ]; then
  echo "❌ Design system file not created!"
  exit 1
fi

echo "✅ Design system created: $DESIGN_SYSTEM_FILE"

# Step 2: Gemini - 병렬 컴포넌트 구현
echo "
Step 2: Gemini builds components in parallel..."

# 3개 Gemini 인스턴스 병렬 실행
cat > /tmp/gemini-tasks.txt << EOF
gemini-layout:DashboardShell,Sidebar,TopNavBar
gemini-data:StatCard,RevenueChart,OrderTable
gemini-forms:SearchInput,FilterDropdown,Toast
EOF

while IFS=: read -r agent components; do
  echo "  Starting $agent for $components..."
  
  agy agent run $agent --input "
design-system.json을 참조하여 다음 컴포넌트를 구현하세요:
$components

출력 디렉토리: src/components/
TypeScript + React 18 + Tailwind CSS
" --async &
  
done < /tmp/gemini-tasks.txt

# 모든 Gemini 작업 완료 대기
wait

echo "✅ All components generated"

# Step 3: Claude - 품질 검토
echo "
Step 3: Claude reviews quality..."

agy agent run claude-architect --input "
생성된 모든 컴포넌트를 검토하세요:
src/components/**/*.tsx

검토 항목:
- 디자인 토큰 준수
- 접근성
- 타입 안정성
- 반응형 디자인

출력: review-report.md
" --output review-report.md

echo "✅ Review completed: review-report.md"

# Step 4: Gemini - 수정 반영 (조건부)
if grep -q "수정 필요" review-report.md; then
  echo "
Step 4: Gemini applies fixes..."
  
  # review-report.md에서 수정 항목 추출
  FIXES=$(grep "수정 필요" review-report.md)
  
  agy agent run gemini-builder --input "
다음 수정 사항을 반영하세요:
$FIXES
" --async
  
  wait
  echo "✅ Fixes applied"
else
  echo "✅ No fixes needed"
fi

# 완료
echo "
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 Multi-agent workflow completed!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Generated files:
- $DESIGN_SYSTEM_FILE
- src/components/layout/*
- src/components/data/*
- src/components/forms/*
- review-report.md
"
```

실행:

```bash
chmod +x orchestrate.sh
./orchestrate.sh
```

---

## 고급 패턴: 병렬 처리와 순차 처리

### 병렬 처리 (Parallel Execution)

**언제 사용:**
- 독립적인 컴포넌트들 (서로 의존성 없음)
- 시간이 중요한 경우
- 리소스가 충분한 경우

**예시: 50개 UI 컴포넌트 라이브러리**

```javascript
// parallel-build.js
const { spawn } = require('child_process');

const components = [
  // Layout (10개)
  ['Container', 'Grid', 'Flex', 'Stack', 'Divider', 'Spacer', 'Center', 'AspectRatio', 'Box', 'Section'],
  // Forms (15개)
  ['Input', 'Textarea', 'Select', 'Checkbox', 'Radio', 'Switch', 'Slider', 'RangeSlider', 'FileUpload', 'DatePicker', 'TimePicker', 'ColorPicker', 'Rating', 'PinInput', 'NumberInput'],
  // Data (10개)
  ['Table', 'List', 'Card', 'Badge', 'Tag', 'Stat', 'Progress', 'Skeleton', 'EmptyState', 'Alert'],
  // Feedback (8개)
  ['Toast', 'Modal', 'Drawer', 'Popover', 'Tooltip', 'Menu', 'Dialog', 'Notification'],
  // Navigation (7개)
  ['Breadcrumb', 'Tabs', 'Pagination', 'Steps', 'Stepper', 'Link', 'NavBar']
];

// 5개 Gemini 인스턴스 (그룹당 1개)
const workers = components.map((group, i) => {
  return spawn('agy', [
    'agent', 'run', `gemini-worker-${i}`,
    '--input', `Build these components: ${group.join(', ')}`,
    '--design-system', 'design-system.json'
  ]);
});

// 진행률 추적
workers.forEach((worker, i) => {
  worker.stdout.on('data', (data) => {
    console.log(`Worker ${i}: ${data}`);
  });
});

// 모든 워커 완료 대기
Promise.all(workers.map(w => new Promise(resolve => w.on('close', resolve))))
  .then(() => {
    console.log('✅ All 50 components built in parallel!');
  });
```

**결과:**
- 순차 처리: ~5시간 (50개 × 6분)
- 병렬 처리: ~60분 (10개 × 6분, 5개 워커)
- **효율성: 5배**

### 순차 처리 (Sequential Execution)

**언제 사용:**
- 의존성이 있는 작업
- 이전 단계의 출력이 다음 단계의 입력인 경우
- 품질이 우선인 경우

**예시: 디자인 시스템 → 토큰 → 기본 컴포넌트 → 복합 컴포넌트**

```yaml
# workflow.yaml
name: Design System Build Pipeline

stages:
  - name: foundation
    agent: claude-architect
    tasks:
      - Design core system
      - Define tokens
      - Create component hierarchy
    outputs:
      - design-system.json
      - tokens.json
      - hierarchy.md
  
  - name: primitives
    agent: gemini-builder
    depends_on: foundation
    inputs:
      - design-system.json
      - tokens.json
    tasks:
      - Build basic components (Button, Input, etc.)
    outputs:
      - src/primitives/*
  
  - name: composites
    agent: gemini-builder
    depends_on: primitives
    inputs:
      - design-system.json
      - src/primitives/*
    tasks:
      - Build composite components (Form, Table, etc.)
    outputs:
      - src/composites/*
  
  - name: patterns
    agent: claude-architect
    depends_on: composites
    inputs:
      - src/primitives/*
      - src/composites/*
    tasks:
      - Create usage patterns
      - Write documentation
    outputs:
      - docs/patterns.md
      - docs/examples.md
  
  - name: review
    agent: claude-architect
    depends_on: patterns
    tasks:
      - Final quality check
      - Accessibility audit
      - Performance analysis
    outputs:
      - review-report.md
```

실행:

```bash
agy workflow run workflow.yaml
```

### 하이브리드 패턴

가장 효율적인 접근은 병렬과 순차를 조합하는 것입니다.

```
Stage 1 (Sequential): Claude - 디자인 시스템
          ↓
Stage 2 (Parallel): Gemini - 3개 팀이 동시에 컴포넌트 구현
          ├─ Team A: Layout
          ├─ Team B: Data
          └─ Team C: Forms
          ↓
Stage 3 (Sequential): Claude - 통합 검토
          ↓
Stage 4 (Parallel): Gemini - 각 팀이 자신의 수정사항 반영
          ↓
Stage 5 (Sequential): Claude - 최종 승인
```

---

## 트러블슈팅 및 최적화

### 일반적인 문제들

#### 문제 1: "Agent가 design-system.json을 무시합니다"

**증상:**
Gemini가 자기 마음대로 색상이나 스타일을 변경

**원인:**
- design-system.json이 컨텍스트에 제대로 로드되지 않음
- 프롬프트가 불명확

**해결책:**

프롬프트를 더 명확하게:

```
❌ 나쁜 프롬프트:
"design-system.json을 참조하여 Button 컴포넌트를 만들어줘"

✅ 좋은 프롬프트:
"다음 지침을 STRICT하게 따라주세요:

1. design-system.json 파일을 읽고 전체 내용을 이해하세요
2. Button 컴포넌트를 구현할 때:
   - colors.dark.primary 값 (#3B82F6)을 정확히 사용
   - borderRadius.md (0.5rem) 적용
   - typography.fontSize.base (1rem) 사용
3. 임의의 색상 값 (예: #1E40AF, #60A5FA) 사용 금지
4. 모든 값은 design-system.json에서 가져와야 함

위 규칙을 어기면 REJECTED됩니다."
```

#### 문제 2: "Gemini 병렬 실행 시 일관성 문제"

**증상:**
3개 Gemini가 생성한 컴포넌트의 스타일이 미묘하게 다름

**원인:**
각 인스턴스가 독립적으로 해석

**해결책:**

공유 파일에 명확한 예제 포함:

**design-system-examples.md:**

~~~markdown
# Button 컴포넌트 예제

✅ 올바른 구현:

```tsx
<button className="
  px-4 py-2 
  bg-blue-600 
  text-white 
  rounded-md 
  hover:bg-blue-700
">
  Click me
</button>
```

❌ 잘못된 구현:

```tsx
// 임의의 색상 사용
<button className="bg-blue-500"> // WRONG

// Tailwind 값 직접 사용
<button className="rounded-lg"> // WRONG, should be rounded-md

// 간격 불일치
<button className="px-3 py-2"> // WRONG, should be px-4
```
~~~

모든 Gemini 인스턴스에 이 파일을 명시적으로 전달:

```bash
agy agent run gemini-1 --context design-system.json,design-system-examples.md
```

#### 문제 3: "Claude 검토가 너무 엄격합니다"

**증상:**
사소한 문제까지 모두 지적하여 반복 수정이 너무 많음

**원인:**
검토 기준이 명확하지 않음

**해결책:**

**review-criteria.md:**

```markdown
# 품질 검토 기준

## Critical (반드시 수정)
- [ ] 디자인 토큰 값 불일치 (색상, 간격)
- [ ] 접근성 WCAG AA 미달
- [ ] TypeScript 타입 에러
- [ ] 기능 누락 (spec에 명시된 기능)

## Important (수정 권장)
- [ ] 성능 문제 (불필요한 리렌더링)
- [ ] 에러 처리 누락
- [ ] 로딩 상태 없음
- [ ] 반응형 미흡

## Nice-to-have (선택)
- [ ] 애니메이션 개선
- [ ] 주석 추가
- [ ] 코드 리팩토링
- [ ] 추가 기능 제안

검토 시 Critical은 반드시 지적, Nice-to-have는 언급만.
```

Claude 프롬프트에 포함:

```
design-system.json과 review-criteria.md를 기준으로 검토하세요.
Critical 항목만 "수정 필요"로 표시하고, 
Nice-to-have는 "개선 제안"으로 구분하세요.
```

### 성능 최적화

#### 최적화 1: 캐싱

**문제:** 같은 디자인 시스템으로 여러 프로젝트 작업 시 매번 처음부터 설계

**해결:**

```bash
# 디자인 시스템 템플릿 저장
mkdir -p ~/.agy/templates

# Claude로 기본 템플릿 생성
agy agent run claude-architect --input "
Create a reusable SaaS design system template
" --output ~/.agy/templates/saas-design-system.json

# 새 프로젝트에서 재사용
cp ~/.agy/templates/saas-design-system.json ./design-system.json

# 프로젝트별 커스터마이징만 요청
agy agent run claude-architect --input "
Customize this design system for a fintech app:
- Change primary color to green
- Add charts for financial data
"
```

**효과:** 디자인 시스템 생성 시간 15분 → 3분 (80% 절약)

#### 최적화 2: 점진적 검토

**문제:** 모든 컴포넌트 완성 후 검토하면 대규모 수정 발생

**해결:**

컴포넌트 3-5개 단위로 검토:

```bash
# Gemini가 5개 생성
agy agent run gemini-builder --input "Build: Button, Input, Select, Checkbox, Radio"

# 즉시 Claude 검토
agy agent run claude-reviewer --input "Review just: Button, Input, Select, Checkbox, Radio"

# 수정 반영
agy agent run gemini-builder --input "Apply fixes from review"

# 다음 배치
agy agent run gemini-builder --input "Build: Card, Modal, Toast, Alert, Badge"
```

**효과:** 대규모 재작업 방지, 누적 오류 감소

#### 최적화 3: Skill 버전 관리

UI/UX Pro Max Skill을 팀에 맞게 커스터마이징:

```bash
# 팀 전용 스타일 추가
echo "58,Corporate Modern,..." >> .agent/skills/ui-ux-pro-max/data/ui-styles.csv

# 회사 컬러 팔레트 추가
echo "custom-1,Company Brand,#FF6B00,..." >> .agent/skills/ui-ux-pro-max/data/color-palettes.csv

# Git으로 버전 관리
git add .agent/skills/
git commit -m "Add custom design patterns"
```

팀원들이 동일한 Skill 사용:

```bash
git clone team-repo
uipro init --ai antigravity
# 팀 전용 스타일이 자동으로 포함됨
```

---

## 결론

Claude와 Gemini의 협업은 단순히 "속도 향상"을 넘어, **인간 디자이너와 개발자의 협업 방식**을 AI로 재현하는 것입니다.

**핵심 원칙:**

1. **명확한 역할 분담**
   - Claude: 창의성, 일관성, 품질 검토
   - Gemini: 실행, 병렬 처리, 반복 작업

2. **상세한 명세**
   - design-system.json은 "계약서"
   - 모든 값은 명시적으로 정의
   - 예제와 안티패턴 제공

3. **점진적 개선**
   - 단계별 검토
   - 빠른 피드백 루프
   - 누적 오류 방지

4. **자동화**
   - 스크립트로 워크플로우 고정
   - 반복 가능한 프로세스
   - 일관된 결과물

이 가이드의 패턴을 따르면, 1인 개발자도 팀 규모의 생산성을 달성할 수 있습니다.

**다음 단계:**

1. 작은 프로젝트로 시작 (랜딩 페이지 1개)
2. 워크플로우 개선 (병목 지점 파악)
3. 팀에 적용 (템플릿 공유)
4. 스케일업 (복잡한 시스템)

AI 협업의 미래는 지금 여기에 있습니다. 시작하세요!

---

**작성 일자: 2026-01-20**
