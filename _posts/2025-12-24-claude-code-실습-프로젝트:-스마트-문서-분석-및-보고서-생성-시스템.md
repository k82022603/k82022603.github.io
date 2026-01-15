---
title: "Claude Code 실습 프로젝트: 스마트 문서 분석 및 보고서 생성 시스템"
date: 2025-12-24 20:10:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Claude,  claude-code,  MCP,  claude-skills,  playwright-mcp,  Claude.write]
---

## 목차
1. [프로젝트 개요](#프로젝트-개요)
2. [시스템 설계](#시스템-설계)
3. [환경 준비](#환경-준비)
4. [실습 1: Claude.ai에서 시작하기](#실습-1-claudeai에서-시작하기)
5. [실습 2: Claude Code 설정](#실습-2-claude-code-설정)
6. [실습 3: MCP 서버 구축](#실습-3-mcp-서버-구축)
7. [실습 4: 커스텀 스킬 개발](#실습-4-커스텀-스킬-개발)
8. [실습 5: 전체 워크플로우 실행](#실습-5-전체-워크플로우-실행)
9. [고급 확장](#고급-확장)
10. [트러블슈팅](#트러블슈팅)

---

## 관련문서
[Claude Code 기반 지식 관리 에이전트 개발 가이드](https://k82022603.github.io/posts/claude-code-%EA%B8%B0%EB%B0%98-%EC%A7%80%EC%8B%9D-%EA%B4%80%EB%A6%AC-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8-%EA%B0%9C%EB%B0%9C-%EA%B0%80%EC%9D%B4%EB%93%9C/)

## 프로젝트 개요

### 프로젝트 목표
여러 소스(웹 기사, PDF 논문, 로컬 문서)에서 정보를 수집하고 분석하여 구조화된 보고서를 자동 생성하는 시스템을 구축합니다.

### 주요 기능
- 다양한 형식의 문서 자동 수집 (URL, PDF, DOCX)
- 내용 분석 및 핵심 정보 추출
- 여러 소스의 정보를 통합한 종합 보고서 생성
- Markdown 형식의 구조화된 출력
- 자동 태깅 및 분류

### 학습 목표
1. Claude Code의 기본 사용법 익히기
2. MCP 서버 설정 및 활용
3. 커스텀 스킬 작성 및 관리
4. Progressive Context Loading 이해
5. 체크포인트 시스템 활용

### 난이도별 경로
- **초급**: Claude.ai만 사용하여 기본 워크플로우 체험
- **중급**: Claude Code + 기본 MCP 서버 활용
- **고급**: 커스텀 스킬 개발 + 고급 MCP 통합

---

## 시스템 설계

### 아키텍처 다이어그램

```
┌─────────────────────────────────────────────────────────┐
│                    입력 소스                              │
├─────────────────────────────────────────────────────────┤
│  • 웹 URL (뉴스, 블로그, 기술 문서)                       │
│  • PDF 파일 (논문, 보고서)                                │
│  • DOCX 파일 (기획서, 제안서)                             │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│              콘텐츠 수집 계층                             │
├─────────────────────────────────────────────────────────┤
│  MCP 서버:                                               │
│  • web_fetch (기본)                                      │
│  • Playwright (동적 웹페이지)                            │
│  • Filesystem (로컬 파일)                                │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│              분석 및 처리 계층                            │
├─────────────────────────────────────────────────────────┤
│  커스텀 스킬:                                             │
│  • document_analyzer.md                                  │
│  • content_extractor.md                                  │
│  • report_generator.md                                   │
│  • metadata_tagger.md                                    │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                 출력 계층                                │
├─────────────────────────────────────────────────────────┤
│  • 종합 분석 보고서 (Markdown)                           │
│  • 개별 소스 요약 파일                                   │
│  • 메타데이터 인덱스                                     │
│  • 시각화 자료 (선택)                                    │
└─────────────────────────────────────────────────────────┘
```

### 데이터 플로우

1. **입력 단계**: 사용자가 분석할 소스 목록 제공
2. **수집 단계**: 각 소스에서 콘텐츠 추출
3. **분석 단계**: 문서 구조 파악 및 핵심 정보 추출
4. **통합 단계**: 여러 소스의 정보를 주제별로 통합
5. **생성 단계**: 구조화된 보고서 작성
6. **저장 단계**: 결과물을 지정된 위치에 저장

### 디렉토리 구조

```
smart-document-analyzer/
├── README.md
├── CLAUDE.md                    # Claude Code 설정
├── .claude/
│   └── mcp_config.json         # MCP 서버 설정
├── skills/
│   ├── document_analyzer.md
│   ├── content_extractor.md
│   ├── report_generator.md
│   └── metadata_tagger.md
├── input/
│   ├── urls.txt                # 분석할 URL 목록
│   └── documents/              # PDF, DOCX 파일
├── output/
│   ├── reports/                # 생성된 보고서
│   ├── summaries/              # 개별 요약
│   └── metadata/               # 메타데이터 파일
└── scripts/
    └── batch_processor.js      # 배치 처리 스크립트
```

---

## 환경 준비

### 필수 요구사항

#### 모든 실습 공통
- Claude.ai 계정 (Pro 구독 권장)
- 텍스트 에디터 (VS Code 추천)
- 인터넷 연결

#### Claude Code 사용 시 추가 요구사항
- Node.js 18+ 설치
- npm 또는 yarn
- Git
- 터미널 환경 (Windows: PowerShell, macOS/Linux: Bash)

### 설치 단계

#### 1. Node.js 설치 확인
```bash
node --version  # v18.0.0 이상이어야 함
npm --version
```

#### 2. Claude Code 설치
```bash
npm install -g @anthropic-ai/claude-code
```

#### 3. Claude Code 인증
```bash
claude-code auth
# 브라우저가 열리면 Claude.ai 계정으로 로그인
```

#### 4. 프로젝트 디렉토리 생성
```bash
mkdir smart-document-analyzer
cd smart-document-analyzer
```

---

## 실습 1: Claude.ai에서 시작하기

> **목표**: 웹 인터페이스에서 기본 워크플로우를 체험하고 개념을 이해합니다.

### 1-1. 간단한 문서 분석

**단계 1**: Claude.ai에 접속하여 새 대화 시작

**단계 2**: 다음 프롬프트 입력

```
다음 URL의 기사를 분석해주세요:
https://www.bbc.com/korean/articles/cly2341your-article-id

분석 항목:
1. 주요 주제 및 핵심 메시지
2. 중요 인용문 3개
3. 팩트와 의견 구분
4. 핵심 키워드 5개
5. 200자 요약

결과를 구조화된 마크다운으로 작성해주세요.
```

**단계 3**: Claude의 응답을 관찰하고 다음 사항 확인
- web_search 또는 web_fetch 도구 사용 여부
- 구조화된 형식으로 정보 추출
- 정확성과 완성도

### 1-2. 여러 소스 비교 분석

**단계 1**: 다음 프롬프트로 확장

```
이제 같은 주제에 대한 다른 소스도 분석해주세요:
- URL 1: [첫 번째 URL]
- URL 2: [두 번째 URL]
- URL 3: [세 번째 URL]

각 소스를 분석한 후, 다음 형식의 비교 보고서를 작성해주세요:

# 주제별 비교 분석

## 공통점
[3개 소스 모두 언급한 내용]

## 차이점
[각 소스의 독특한 관점]

## 종합 평가
[전체적인 인사이트]
```

**단계 2**: 결과 검토 및 학습 포인트
- 여러 소스를 어떻게 통합하는지 관찰
- 정보의 일관성과 모순 처리 방식
- 구조화된 출력의 장점

### 1-3. 마크다운 문서로 저장

**단계 1**: 분석 결과를 파일로 저장 요청

```
위 분석 결과를 다음 형식의 마크다운 문서로 만들어주세요:

파일명: analysis-report-[날짜].md

구조:
- YAML 프론트매터 (제목, 날짜, 태그, 소스)
- 요약
- 상세 분석
- 비교표
- 참고 자료
```

**단계 2**: Claude가 생성한 파일을 다운로드

> **학습 포인트**: Claude.ai는 간단한 분석과 프로토타이핑에 적합하지만, 대량 처리나 복잡한 자동화에는 Claude Code가 필요합니다.

---

## 실습 2: Claude Code 설정

> **목표**: 로컬 환경에서 Claude Code를 설정하고 기본 명령어를 익힙니다.

### 2-1. 프로젝트 초기화

**단계 1**: 프로젝트 디렉토리에서 초기화

```bash
cd smart-document-analyzer
claude-code init
```

**단계 2**: 기본 파일 구조 생성

```bash
mkdir -p skills input output/{reports,summaries,metadata} scripts
touch CLAUDE.md
touch input/urls.txt
```

**단계 3**: CLAUDE.md 파일 작성

```markdown
# Smart Document Analyzer

당신은 문서 분석 및 보고서 생성 전문 AI 에이전트입니다.

## 역할

다양한 소스에서 정보를 수집하고 분석하여 구조화된 보고서를 생성합니다.

## 작업 흐름

1. **입력 확인**: input/urls.txt 또는 input/documents/ 에서 분석할 소스 확인
2. **콘텐츠 수집**: 각 소스에서 내용 추출
3. **분석**: 핵심 정보 파악 및 구조화
4. **보고서 생성**: 통합 분석 결과 작성
5. **저장**: output/reports/ 에 결과 저장

## 스킬 사용 규칙

- `document_analyzer`: 항상 가장 먼저 실행하여 문서 구조 파악
- `content_extractor`: 핵심 정보 추출
- `metadata_tagger`: 태그 및 분류 정보 생성
- `report_generator`: 최종 보고서 작성

## 출력 형식

- 모든 보고서는 Markdown 형식
- YAML 프론트매터 필수
- 파일명: `report-YYYYMMDD-HHMM.md`

## MCP 도구 사용

- URL 수집: web_fetch 먼저 시도, 실패 시 Playwright 사용
- 로컬 파일: filesystem MCP 사용
- PDF: 텍스트 추출 후 처리

## 에러 처리

- 접근 불가 URL: 로그에 기록하고 계속 진행
- 파싱 실패: 원본 텍스트 저장 후 수동 검토 표시
```

### 2-2. 첫 번째 실행

**단계 1**: input/urls.txt 파일 생성

```txt
https://example.com/article1
https://example.com/article2
https://example.com/article3
```

**단계 2**: Claude Code 실행

```bash
claude-code chat "input/urls.txt의 URL들을 분석하고 종합 보고서를 작성해주세요."
```

**단계 3**: 진행 상황 관찰
- 각 URL 접근 시도
- 콘텐츠 추출 과정
- 보고서 생성 과정
- 결과 파일 저장

### 2-3. 체크포인트 시스템 체험

**단계 1**: 장시간 작업 시작

```bash
claude-code chat "input/documents/ 폴더의 모든 PDF 파일을 분석해주세요. 각 파일마다 개별 요약을 만들고, 최종적으로 통합 보고서를 작성해주세요."
```

**단계 2**: 중간에 Ctrl+C로 중단

**단계 3**: 재개

```bash
claude-code resume
```

> **학습 포인트**: Claude Code는 자동으로 체크포인트를 생성하여 작업을 안전하게 재개할 수 있습니다.

---

## 실습 3: MCP 서버 구축

> **목표**: Playwright MCP 서버를 설정하여 동적 웹페이지 처리 능력을 추가합니다.

### 3-1. Playwright MCP 서버 설치

**단계 1**: 필요한 패키지 설치

```bash
npm install -g @modelcontextprotocol/server-playwright
npm install -g playwright
playwright install chromium
```

**단계 2**: MCP 설정 파일 생성

`.claude/mcp_config.json` 파일 생성:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-playwright"
      ],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true",
        "PLAYWRIGHT_BROWSER": "chromium"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "${workspaceFolder}/input",
        "${workspaceFolder}/output"
      ]
    }
  }
}
```

### 3-2. MCP 서버 테스트

**단계 1**: Claude Code에서 Playwright 사용 테스트

```bash
claude-code chat "Playwright를 사용하여 https://news.ycombinator.com의 최신 기사 5개의 제목과 링크를 추출해주세요."
```

**단계 2**: 결과 확인
- Playwright 서버 시작 로그
- 브라우저 자동화 과정
- 추출된 데이터

### 3-3. 고급 웹 스크래핑

**단계 1**: 로그인이 필요한 사이트 처리

CLAUDE.md에 추가:

```markdown
## Playwright 사용 가이드

### 동적 콘텐츠 처리
- 페이지 로드 후 waitForSelector 사용
- JavaScript 렌더링 완료 대기
- 스크롤하여 지연 로딩 콘텐츠 로드

### 로그인 처리 (주의: 인증 정보 보안)
1. 환경 변수에서 자격증명 읽기
2. 로그인 폼 찾기
3. 자격증명 입력
4. 로그인 완료 대기
5. 목표 페이지 이동
```

**단계 2**: 실습 - Medium 기사 수집

```bash
claude-code chat "Medium의 특정 태그(예: #ai)에서 최근 기사 10개를 수집하고 요약해주세요."
```

---

## 실습 4: 커스텀 스킬 개발

> **목표**: 재사용 가능한 커스텀 스킬을 작성하여 작업을 모듈화합니다.

### 4-1. Document Analyzer 스킬

**파일**: `skills/document_analyzer.md`

~~~markdown
# Document Analyzer Skill

## 목적
입력된 문서의 구조를 분석하고 핵심 요소를 식별합니다.

## 입력
- 전체 문서 텍스트 (Markdown, HTML, 또는 일반 텍스트)

## 처리 과정

### 1단계: 문서 유형 판별
문서의 특성을 파악하여 다음 중 하나로 분류:
- 뉴스 기사
- 학술 논문
- 기술 문서
- 블로그 포스트
- 비즈니스 보고서

### 2단계: 구조 분석
문서 유형에 따라 적절한 구조 요소 식별:

**뉴스 기사**:
- 헤드라인
- 리드 (첫 문단)
- 본문
- 인용문
- 결론

**학술 논문**:
- 제목 및 저자
- 초록
- 서론
- 방법론
- 결과
- 논의
- 결론
- 참고문헌

**기술 문서**:
- 제목
- 개요
- 전제 조건
- 단계별 설명
- 코드 예제
- 문제 해결

### 3단계: 메타데이터 추출
- 작성자
- 발행일
- 출처/출판사
- 주요 키워드 (5-10개)
- 카테고리/태그

### 4단계: 핵심 내용 식별
- 주요 주장/메시지 (3-5개)
- 중요 데이터/통계
- 핵심 인용문 (최대 3개)
- 실행 가능한 인사이트

## 출력 형식

JSON 형식으로 반환:

```json
{
  "document_type": "news_article",
  "structure": {
    "headline": "...",
    "lead": "...",
    "body_sections": [...],
    "quotes": [...],
    "conclusion": "..."
  },
  "metadata": {
    "author": "...",
    "date": "YYYY-MM-DD",
    "source": "...",
    "keywords": [...],
    "categories": [...]
  },
  "key_content": {
    "main_points": [...],
    "data": [...],
    "key_quotes": [...],
    "insights": [...]
  },
  "summary": "200자 이내 요약"
}
```

## 사용 예시

```
document_analyzer를 사용하여 다음 텍스트를 분석해주세요:

[문서 텍스트]
```

## 에러 처리
- 구조가 불명확한 경우: best-effort로 처리하고 신뢰도 표시
- 메타데이터 누락: 가능한 정보만 제공
- 빈 섹션: null 대신 빈 배열/문자열 사용
~~~

### 4-2. Content Extractor 스킬

**파일**: `skills/content_extractor.md`

~~~markdown
# Content Extractor Skill

## 목적
분석된 문서에서 특정 유형의 콘텐츠를 추출합니다.

## 입력
- document_analyzer의 출력 JSON
- 추출할 콘텐츠 유형 지정

## 추출 가능한 콘텐츠 유형

### 1. 팩트 vs 의견
문장을 분석하여 객관적 사실과 주관적 의견을 구분:

**팩트**:
- 검증 가능한 데이터
- 통계
- 인용된 연구 결과
- 역사적 사실

**의견**:
- 저자의 주장
- 전망/예측
- 평가/비판
- 제안

### 2. 핵심 데이터 포인트
숫자, 통계, 측정값을 추출하고 컨텍스트와 함께 정리:

```
- "매출 20% 증가" → {metric: "매출", change: "+20%", context: "전년 대비"}
- "1,000명 참여" → {metric: "참여자", value: 1000, unit: "명"}
```

### 3. 액션 아이템
문서에서 실행 가능한 항목 추출:
- 명령문
- 권장사항
- 다음 단계
- 체크리스트

### 4. 질문 및 답변
문서에서 제기되는 질문과 그에 대한 답변 쌍을 추출

## 출력 형식

```json
{
  "facts": [
    {
      "statement": "...",
      "source": "paragraph 3",
      "verifiable": true
    }
  ],
  "opinions": [
    {
      "statement": "...",
      "author_view": true,
      "confidence": "high|medium|low"
    }
  ],
  "data_points": [
    {
      "metric": "...",
      "value": "...",
      "context": "...",
      "source": "..."
    }
  ],
  "action_items": [
    {
      "action": "...",
      "priority": "high|medium|low",
      "category": "..."
    }
  ],
  "qa_pairs": [
    {
      "question": "...",
      "answer": "...",
      "section": "..."
    }
  ]
}
```

## 사용 예시

```
content_extractor를 사용하여 다음 분석 결과에서 팩트와 의견을 구분해주세요:

[document_analyzer 출력 JSON]
```
~~~

### 4-3. Report Generator 스킬

**파일**: `skills/report_generator.md`

~~~markdown
# Report Generator Skill

## 목적
여러 소스의 분석 결과를 통합하여 종합 보고서를 생성합니다.

## 입력
- 여러 document_analyzer 출력의 배열
- 보고서 유형 (비교, 종합, 트렌드 등)
- 대상 독자 (전문가, 일반인, 경영진)

## 보고서 유형

### 1. 비교 분석 보고서
여러 소스를 비교하여 공통점과 차이점 강조

**구조**:
```markdown
# [주제] 비교 분석 보고서

## 요약
[전체 내용 200자 요약]

## 소스 개요
| 소스 | 유형 | 날짜 | 관점 |
|------|------|------|------|
| ... | ... | ... | ... |

## 주요 발견사항

### 공통점
[모든 소스가 동의하는 내용]

### 차이점
#### 관점 1: [소스 A의 입장]
- ...

#### 관점 2: [소스 B의 입장]
- ...

## 주제별 심층 분석

### [주제 1]
[각 소스의 입장 비교]

### [주제 2]
...

## 데이터 비교
[수치, 통계 비교표]

## 결론 및 시사점

## 참고 자료
```

### 2. 트렌드 분석 보고서
시간 순서대로 정보를 배열하여 변화와 패턴 파악

### 3. 종합 요약 보고서
모든 소스의 핵심 내용을 주제별로 통합

## 출력 규칙

1. **YAML 프론트매터 필수**
```yaml
---
title: "보고서 제목"
date: "YYYY-MM-DD"
sources: 
  - "소스 1 URL/경로"
  - "소스 2 URL/경로"
tags: [tag1, tag2, tag3]
report_type: "비교|종합|트렌드"
audience: "전문가|일반인|경영진"
---
```

2. **명확한 섹션 구분**
- H1: 보고서 제목
- H2: 주요 섹션
- H3: 하위 항목

3. **시각적 요소 활용**
- 표
- 블록 인용
- 코드 블록 (데이터, JSON)
- 체크리스트

4. **참조 및 출처**
- 모든 주장에 출처 표시
- 링크는 각주 형식으로

## 사용 예시

```
report_generator를 사용하여 다음 3개 분석 결과를 비교 보고서로 작성해주세요:

대상 독자: 일반인
보고서 유형: 비교 분석

[분석 결과 1]
[분석 결과 2]
[분석 결과 3]
```
~~~

### 4-4. 스킬 테스트

**단계 1**: 개별 스킬 테스트

```bash
claude-code chat "document_analyzer 스킬을 사용하여 input/documents/sample.pdf를 분석해주세요."
```

**단계 2**: 스킬 체이닝 테스트

```bash
claude-code chat "
1. document_analyzer로 input/urls.txt의 첫 번째 URL 분석
2. content_extractor로 팩트와 의견 구분
3. 결과를 output/summaries/에 저장
"
```

**단계 3**: 전체 파이프라인 테스트

```bash
claude-code chat "
input/urls.txt의 모든 URL을:
1. document_analyzer로 분석
2. content_extractor로 핵심 정보 추출
3. report_generator로 통합 보고서 생성
4. output/reports/에 저장
"
```

---

## 실습 5: 전체 워크플로우 실행

> **목표**: 실제 사용 사례로 전체 시스템을 테스트합니다.

### 5-1. 실습 시나리오: AI 뉴스 분석

**배경**: 최근 1주일간 발표된 AI 관련 주요 뉴스를 수집하고 분석하여 트렌드 보고서를 작성합니다.

**단계 1**: 소스 준비

`input/urls.txt`:
```
https://techcrunch.com/ai
https://www.theverge.com/ai-artificial-intelligence
https://openai.com/blog
https://anthropic.com/news
https://deepmind.google/discover/blog/
```

**단계 2**: 배치 처리 스크립트 작성

`scripts/batch_processor.js`:
```javascript
// Claude Code가 실행할 배치 작업 정의
const sources = [
  {type: 'url', path: 'https://techcrunch.com/ai', category: 'tech-news'},
  {type: 'url', path: 'https://www.theverge.com/ai', category: 'tech-news'},
  {type: 'url', path: 'https://openai.com/blog', category: 'company'},
  {type: 'url', path: 'https://anthropic.com/news', category: 'company'},
  {type: 'file', path: './input/documents/research-paper.pdf', category: 'research'}
];

module.exports = { sources };
```

**단계 3**: Claude Code 실행

```bash
claude-code chat "
scripts/batch_processor.js의 모든 소스를 처리하세요:

1. 각 소스마다:
   - document_analyzer로 분석
   - content_extractor로 핵심 정보 추출
   - output/summaries/[소스명]-YYYYMMDD.md에 저장

2. 모든 소스 처리 후:
   - report_generator로 트렌드 분석 보고서 생성
   - 카테고리별 섹션 구성
   - output/reports/ai-trend-report-YYYYMMDD.md에 저장

3. 처리 로그:
   - 성공/실패 항목
   - 처리 시간
   - output/metadata/processing-log.json에 저장
"
```

**단계 4**: 결과 확인

```bash
# 생성된 파일 목록
ls -la output/summaries/
ls -la output/reports/

# 보고서 미리보기
cat output/reports/ai-trend-report-*.md | head -50
```

### 5-2. 헤드리스 모드 실행

**단계 1**: 설정 파일 작성

`task-config.json`:
```json
{
  "task": "AI 트렌드 분석",
  "sources": "input/urls.txt",
  "output": "output/reports/",
  "skills": [
    "document_analyzer",
    "content_extractor",
    "report_generator"
  ],
  "options": {
    "report_type": "trend",
    "audience": "general",
    "include_charts": false
  }
}
```

**단계 2**: 헤드리스 실행

```bash
claude-code run --config task-config.json --headless
```

**단계 3**: 진행 상황 모니터링

```bash
# 별도 터미널에서
tail -f ~/.claude-code/logs/latest.log
```

### 5-3. 스케줄링 (선택)

**cron 작업 설정** (Linux/macOS):

```bash
# 매일 오전 9시에 AI 뉴스 분석 실행
crontab -e

# 추가할 라인:
0 9 * * * cd /path/to/smart-document-analyzer && claude-code run --config task-config.json --headless
```

**Windows 작업 스케줄러**:
1. 작업 스케줄러 열기
2. "기본 작업 만들기"
3. 트리거: 매일 오전 9시
4. 작업: `claude-code run --config task-config.json --headless`

---

## 고급 확장

### 6-1. 이미지 분석 추가

**새 스킬**: `skills/image_analyzer.md`

```markdown
# Image Analyzer Skill

## 목적
문서에 포함된 이미지를 분석하여 설명을 생성합니다.

## 처리
1. 이미지 추출
2. 비전 모델로 내용 분석
3. 차트/그래프인 경우 데이터 추출
4. Alt 텍스트 생성

## 출력
- 이미지 설명
- 추출된 텍스트 (있는 경우)
- 차트 데이터 (있는 경우)
```

### 6-2. Notion 통합

**MCP 서버 추가**:

`.claude/mcp_config.json`에 추가:
```json
{
  "notion": {
    "command": "npx",
    "args": ["@notionhq/client"],
    "env": {
      "NOTION_API_KEY": "${NOTION_API_KEY}",
      "NOTION_DATABASE_ID": "${NOTION_DATABASE_ID}"
    }
  }
}
```

**사용 예시**:
```bash
claude-code chat "생성된 보고서를 Notion 데이터베이스에 업로드해주세요."
```

### 6-3. 시각화 추가

**새 스킬**: `skills/data_visualizer.md`

```markdown
# Data Visualizer Skill

## 목적
추출된 데이터를 차트로 시각화합니다.

## 도구
- Mermaid (다이어그램)
- Chart.js (차트)
- D3.js (복잡한 시각화)

## 출력
- SVG 파일
- HTML 인터랙티브 차트
```

---

## 트러블슈팅

### 일반적인 문제

#### 1. MCP 서버 연결 실패

**증상**:
```
Error: Failed to connect to MCP server 'playwright'
```

**해결**:
```bash
# MCP 서버 수동 테스트
npx @modelcontextprotocol/server-playwright

# 경로 확인
which npx

# 권한 확인
chmod +x $(which npx)
```

#### 2. 스킬을 찾을 수 없음

**증상**:
```
Skill 'document_analyzer' not found
```

**해결**:
1. 스킬 파일 위치 확인: `ls skills/`
2. CLAUDE.md에서 스킬 경로 확인
3. 스킬 파일 이름 확인 (대소문자 구분)

#### 3. 메모리 부족

**증상**:
```
Out of memory / Context limit exceeded
```

**해결**:
```markdown
# CLAUDE.md 최적화
- 한 번에 처리할 소스 수 제한 (5-10개)
- 문서 크기 체크 후 긴 문서는 청크 단위로 처리
- 불필요한 스킬 로드 제거
```

#### 4. URL 접근 실패

**증상**:
```
Failed to fetch URL: 403 Forbidden
```

**해결**:
- User-Agent 헤더 추가
- Playwright로 전환 (JavaScript 실행 필요 시)
- robots.txt 확인

### 디버깅 팁

**1. 상세 로그 활성화**:
```bash
claude-code chat --verbose "..."
```

**2. 단계별 실행**:
```bash
# 한 단계씩 실행하며 중간 결과 확인
claude-code chat "document_analyzer만 실행하고 결과를 보여주세요"
```

**3. 체크포인트 확인**:
```bash
ls ~/.claude-code/checkpoints/
cat ~/.claude-code/checkpoints/latest/state.json
```

---

## 다음 단계

### 학습 리소스

1. **공식 문서**
   - Claude Code: https://docs.claude.ai/code
   - MCP 프로토콜: https://modelcontextprotocol.io
   - Anthropic API: https://docs.anthropic.com

2. **커뮤니티**
   - Discord: Anthropic 공식 커뮤니티
   - GitHub: MCP 예제 저장소
   - Reddit: r/ClaudeAI

3. **고급 주제**
   - 커스텀 MCP 서버 개발
   - Progressive Context Loading 최적화
   - 멀티 에이전트 협업 시스템

### 프로젝트 아이디어

1. **학술 논문 리뷰 시스템**
   - arXiv에서 논문 자동 수집
   - 분야별 분류
   - 주간 요약 이메일 발송

2. **경쟁사 모니터링 대시보드**
   - 경쟁사 블로그/뉴스 추적
   - 제품 출시 알림
   - 트렌드 분석 리포트

3. **법률 문서 분석기**
   - 계약서 자동 검토
   - 리스크 포인트 식별
   - 표준 조항 비교

4. **콘텐츠 큐레이션 봇**
   - 관심 주제 자동 수집
   - 품질 평가 및 필터링
   - 개인화된 뉴스레터 생성

---

## 부록

### A. 전체 파일 체크리스트

```
□ README.md
□ CLAUDE.md
□ .claude/mcp_config.json
□ skills/document_analyzer.md
□ skills/content_extractor.md
□ skills/report_generator.md
□ skills/metadata_tagger.md
□ input/urls.txt
□ scripts/batch_processor.js
□ task-config.json
```

### B. 환경 변수 템플릿

`.env` 파일:
```bash
# API Keys (필요시)
NOTION_API_KEY=secret_xxx
OPENAI_API_KEY=sk-xxx

# Paths
WORKSPACE_DIR=/path/to/smart-document-analyzer
OUTPUT_DIR=${WORKSPACE_DIR}/output

# MCP Settings
PLAYWRIGHT_HEADLESS=true
PLAYWRIGHT_BROWSER=chromium

# Processing Options
MAX_SOURCES_PER_BATCH=10
DEFAULT_REPORT_TYPE=comparison
DEFAULT_AUDIENCE=general
```

### C. 샘플 출력

**보고서 예시**: `output/reports/sample-report.md`

```markdown
---
title: "AI 트렌드 분석 보고서"
date: "2024-12-24"
sources: 
  - "https://techcrunch.com/ai"
  - "https://openai.com/blog"
  - "https://anthropic.com/news"
tags: [AI, trends, tech-news, 2024]
report_type: "trend"
audience: "general"
---

# AI 트렌드 분석 보고서

## 요약

최근 1주일간 AI 업계에서는 멀티모달 모델의 발전, 에이전트 시스템의 상용화, 그리고 AI 안전성 연구가 주요 화두였습니다. 주요 AI 기업들은 더욱 강력하면서도 효율적인 모델을 출시했으며, 실제 업무 환경에서의 적용 사례가 증가하고 있습니다.

## 주요 발견사항

### 기술 발전
1. **멀티모달 능력 향상**
   - 이미지, 텍스트, 오디오 통합 처리
   - 실시간 비전 분석 기능
   
2. **에이전트 시스템**
   - 장시간 자율 작업 수행
   - 복잡한 다단계 태스크 처리

[... 계속 ...]
```

---

**작성 일자: 2025-12-24**
**버전**: 1.0
**작성자**: Claude AI
**라이선스**: MIT
