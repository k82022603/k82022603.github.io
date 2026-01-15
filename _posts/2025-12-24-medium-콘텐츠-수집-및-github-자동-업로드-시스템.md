---
title: "Medium 콘텐츠 수집 및 GitHub 자동 업로드 시스템"
date: 2025-12-24 20:20:00 +0900
categories: [AI,  MCP]
mermaid: [True]
tags: [AI,  Claude,  claude-code,  MCP,  claude-skills,  playwright-mcp,  Claude.write]
---

## 목차
1. [프로젝트 개요](#프로젝트-개요)
2. [시스템 아키텍처](#시스템-아키텍처)
3. [환경 설정](#환경-설정)
4. [Medium 수집 전략](#medium-수집-전략)
5. [GitHub 통합](#github-통합)
6. [전체 워크플로우](#전체-워크플로우)
7. [자동화 및 스케줄링](#자동화-및-스케줄링)
8. [실전 예제](#실전-예제)

---

## 관련문서
[Claude Code 기반 지식 관리 에이전트 개발 가이드](https://k82022603.github.io/posts/claude-code-%EA%B8%B0%EB%B0%98-%EC%A7%80%EC%8B%9D-%EA%B4%80%EB%A6%AC-%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8-%EA%B0%9C%EB%B0%9C-%EA%B0%80%EC%9D%B4%EB%93%9C/)

## 프로젝트 개요

### 목표
Medium의 특정 태그/작성자/큐레이션 글을 자동으로 수집, 분석, 정리하여 GitHub 저장소에 체계적으로 업로드하는 시스템 구축

### 주요 기능
- Medium 페이지 스크래핑 (Playwright 활용)
- 이미지 다운로드 및 로컬 저장
- Markdown 형식 변환
- YAML 프론트매터 추가
- GitHub 자동 커밋 및 푸시
- 중복 방지 및 버전 관리

### 예상 결과물
```
my-medium-archive/
├── README.md
├── articles/
│   ├── 2024/
│   │   ├── 12/
│   │   │   ├── ai-trends-2024.md
│   │   │   ├── claude-code-guide.md
│   │   │   └── images/
│   │   │       ├── ai-trends-fig1.png
│   │   │       └── claude-code-demo.gif
├── metadata/
│   ├── index.json
│   └── tags.json
└── .github/
    └── workflows/
        └── daily-sync.yml
```

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────┐
│                  Medium 소스                             │
├─────────────────────────────────────────────────────────┤
│  • 특정 태그 (예: #ai, #programming)                     │
│  • 특정 작성자 팔로잉                                     │
│  • 큐레이션 리스트                                       │
│  • 북마크한 글                                           │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│            수집 계층 (Playwright MCP)                     │
├─────────────────────────────────────────────────────────┤
│  1. 로그인 (선택)                                         │
│  2. 목록 페이지 탐색                                      │
│  3. 개별 글 URL 추출                                      │
│  4. 각 글 본문 수집                                       │
│  5. 이미지/리소스 다운로드                                │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│              처리 계층 (Claude Skills)                    │
├─────────────────────────────────────────────────────────┤
│  • medium_parser.md: HTML → Markdown 변환                │
│  • content_cleaner.md: 정리 및 포맷팅                    │
│  • metadata_generator.md: 프론트매터 생성                │
│  • image_optimizer.md: 이미지 최적화                     │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│           저장 및 버전 관리 (Git MCP)                     │
├─────────────────────────────────────────────────────────┤
│  1. 로컬 저장소에 파일 저장                               │
│  2. Git add, commit                                      │
│  3. GitHub push                                          │
│  4. 메타데이터 인덱스 업데이트                            │
└─────────────────────────────────────────────────────────┘
```

---

## 환경 설정

### 1. 필수 도구 설치

```bash
# Node.js 패키지
npm install -g @modelcontextprotocol/server-playwright
npm install -g @modelcontextprotocol/server-git
npm install turndown  # HTML to Markdown
npm install axios     # 이미지 다운로드
npm install sharp     # 이미지 최적화

# Playwright 브라우저
playwright install chromium
```

### 2. GitHub 저장소 준비

```bash
# 새 저장소 생성
mkdir my-medium-archive
cd my-medium-archive
git init
git remote add origin https://github.com/YOUR_USERNAME/my-medium-archive.git

# 디렉토리 구조 생성
mkdir -p articles/{2024,2025}/{01..12}/images
mkdir -p metadata
mkdir -p scripts
```

### 3. MCP 서버 설정

`.claude/mcp_config.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true",
        "PLAYWRIGHT_BROWSER": "chromium",
        "PLAYWRIGHT_TIMEOUT": "30000"
      }
    },
    "git": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-git",
        "--repository",
        "${workspaceFolder}"
      ]
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "${workspaceFolder}/articles",
        "${workspaceFolder}/metadata",
        "${workspaceFolder}/scripts"
      ]
    }
  }
}
```

### 4. 환경 변수 설정

`.env`:

```bash
# Medium 계정 (선택 - 비공개 글 접근 시 필요)
MEDIUM_EMAIL=your-email@example.com
MEDIUM_PASSWORD=your-password

# GitHub 설정
GITHUB_USERNAME=your-username
GITHUB_REPO=my-medium-archive
GITHUB_TOKEN=ghp_your_personal_access_token

# 수집 설정
MEDIUM_TAGS=ai,programming,claude,automation
MEDIUM_AUTHORS=@author1,@author2
MAX_ARTICLES_PER_RUN=20
```

---

## Medium 수집 전략

### Medium의 구조 이해

Medium은 동적 콘텐츠 로딩을 사용하므로 Playwright가 필수입니다.

**주요 URL 패턴**:
- 태그: `https://medium.com/tag/{tag-name}`
- 작성자: `https://medium.com/@{username}`
- 출판물: `https://medium.com/{publication-name}`
- 개별 글: `https://medium.com/@{username}/{article-slug}`

### 커스텀 스킬: medium_scraper.md

~~~markdown
# Medium Scraper Skill

## 목적
Playwright를 사용하여 Medium 페이지를 스크래핑하고 콘텐츠를 추출합니다.

## 입력
- 대상 URL (태그, 작성자, 개별 글)
- 수집할 글 개수 (기본: 10)
- 이미지 다운로드 여부 (기본: true)

## 처리 과정

### 1. 페이지 접근
```javascript
// Playwright 명령
1. 브라우저 시작
2. URL 이동
3. 페이지 로드 대기
4. 무한 스크롤 처리 (필요시)
```

### 2. 목록 페이지 처리 (태그/작성자 페이지)

```javascript
// 글 링크 추출
const articles = await page.evaluate(() => {
  const links = Array.from(document.querySelectorAll('article a[data-post-id]'));
  return links.map(link => ({
    url: link.href,
    title: link.querySelector('h2')?.textContent,
    preview: link.querySelector('p')?.textContent,
    author: link.querySelector('[data-author]')?.textContent,
    date: link.querySelector('time')?.getAttribute('datetime')
  }));
});
```

### 3. 개별 글 추출

```javascript
// 본문 콘텐츠 추출
const content = await page.evaluate(() => {
  const article = document.querySelector('article');
  
  return {
    title: article.querySelector('h1')?.textContent,
    subtitle: article.querySelector('h2.subtitle')?.textContent,
    author: document.querySelector('[data-author-name]')?.textContent,
    publishDate: document.querySelector('time')?.getAttribute('datetime'),
    readTime: document.querySelector('[data-read-time]')?.textContent,
    tags: Array.from(article.querySelectorAll('a[href^="/tag/"]'))
      .map(a => a.textContent),
    content: article.querySelector('.article-content')?.innerHTML,
    images: Array.from(article.querySelectorAll('img'))
      .map(img => ({
        src: img.src,
        alt: img.alt,
        caption: img.parentElement.querySelector('figcaption')?.textContent
      })),
    claps: document.querySelector('[data-claps]')?.textContent,
    responses: document.querySelector('[data-responses]')?.textContent
  };
});
```

### 4. 이미지 다운로드

로컬에 이미지를 저장하고 상대 경로로 변경:

```javascript
// 각 이미지에 대해
for (const img of content.images) {
  const filename = generateImageFilename(img.src);
  const localPath = `images/${filename}`;
  
  // 다운로드
  await downloadImage(img.src, localPath);
  
  // 경로 업데이트
  img.localPath = localPath;
}
```

## 출력 형식

```json
{
  "url": "https://medium.com/@author/article-slug",
  "metadata": {
    "title": "...",
    "subtitle": "...",
    "author": "...",
    "publishDate": "2024-12-24",
    "readTime": "5 min read",
    "tags": ["ai", "programming"],
    "claps": "234",
    "responses": "12"
  },
  "content": {
    "html": "...",
    "images": [
      {
        "original": "https://...",
        "local": "images/img-001.png",
        "alt": "...",
        "caption": "..."
      }
    ]
  }
}
```

## 주의사항

1. **Rate Limiting**: 요청 사이 2-3초 대기
2. **로그인**: 비공개 글 접근 시 필요
3. **Paywall**: Member-only 글은 제한적 접근
4. **동적 로딩**: 스크롤하여 모든 콘텐츠 로드 확인
~~~

### 커스텀 스킬: medium_to_markdown.md

~~~markdown
# Medium to Markdown Converter Skill

## 목적
Medium의 HTML 콘텐츠를 깔끔한 Markdown으로 변환합니다.

## 변환 규칙

### 1. 제목 처리
```
H1 → # 제목
H2 → ## 부제목
```

### 2. 텍스트 포맷
```
<strong> → **굵게**
<em> → *기울임*
<code> → `코드`
<pre> → ```코드블록```
```

### 3. 링크
```
<a href="...">text</a> → [text](...)
```

### 4. 이미지
```
<img src="remote-url"> → ![alt](images/local-filename.png)
*caption* (별도 처리)
```

### 5. 인용문
```
<blockquote> → > 인용문
```

### 6. 리스트
```
<ul> → - 항목
<ol> → 1. 항목
```

### 7. 코드 블록
Medium의 GitHub Gist 임베드를 코드 블록으로 변환:
```
<script src="gist..."></script> 
→ 
```javascript
// 코드 내용
```
```

### 8. 특수 처리

**Medium의 특수 요소**:
- 강조 박스 → Markdown 인용문
- 구분선 → `---`
- 이미지 갤러리 → 개별 이미지로 분리

## YAML 프론트매터 생성

```yaml
---
title: "원본 제목"
subtitle: "부제목 (있는 경우)"
author: "작성자명"
author_url: "https://medium.com/@author"
publish_date: "2024-12-24"
read_time: "5 min"
source_url: "https://medium.com/..."
tags: [ai, programming, tutorial]
collected_date: "2024-12-24"
claps: 234
responses: 12
---
```

## 출력 형식

완전한 Markdown 문서:

```markdown
---
[YAML 프론트매터]
---

# 제목

> 부제목

![커버 이미지](images/cover.png)

**작성자**: [Author Name](https://medium.com/@author) • **발행일**: 2024-12-24 • **읽는 시간**: 5분

---

[본문 내용...]

## 섹션 1

[...]

---

## 메타정보

- **원본**: [Medium 링크](https://...)
- **수집일**: 2024-12-24
- **태그**: #ai, #programming
```

## 품질 체크

변환 후 다음 사항 확인:
1. 모든 이미지 로컬 경로로 변경됨
2. 코드 블록 올바르게 변환
3. 링크 정상 작동
4. 특수 문자 이스케이프 처리
5. Markdown 문법 유효성
~~~

---

## GitHub 통합

### Git MCP 서버 활용

Git MCP를 사용하면 Claude Code가 직접 Git 명령을 실행할 수 있습니다.

### 커스텀 스킬: github_publisher.md

~~~markdown
# GitHub Publisher Skill

## 목적
수집된 콘텐츠를 GitHub 저장소에 커밋하고 푸시합니다.

## 입력
- 저장할 파일들의 경로 리스트
- 커밋 메시지
- 브랜치 (기본: main)

## 처리 과정

### 1. 변경사항 확인
```bash
# Git MCP 명령
git_status
```

### 2. 파일 추가
```bash
# 새 글 추가
git_add articles/2024/12/new-article.md
git_add articles/2024/12/images/*.png

# 메타데이터 업데이트
git_add metadata/index.json
```

### 3. 커밋 생성
```bash
git_commit -m "Add: [글 제목] from Medium

- Author: [작성자]
- Tags: [태그들]
- Date: [발행일]
- Images: [이미지 개수]개
"
```

### 4. 푸시
```bash
git_push origin main
```

## 자동 커밋 메시지 생성

포맷:
```
[Action]: [Article Title]

- Author: @username
- Published: YYYY-MM-DD
- Tags: #tag1, #tag2, #tag3
- Read time: X min
- Images: N
- Source: [Medium URL]

Automated collection via Claude Code
```

Actions:
- `Add`: 새 글 추가
- `Update`: 기존 글 수정
- `Fix`: 오류 수정
- `Sync`: 메타데이터 동기화

## 충돌 처리

1. 리모트 변경사항 확인
```bash
git_fetch
git_status
```

2. 충돌 발생 시
```bash
git_pull --rebase
# 수동 해결 필요 시 사용자에게 알림
```

## 메타데이터 인덱스 관리

`metadata/index.json` 업데이트:

```json
{
  "last_updated": "2024-12-24T10:30:00Z",
  "total_articles": 42,
  "articles": [
    {
      "id": "abc123",
      "title": "...",
      "author": "...",
      "date": "2024-12-24",
      "path": "articles/2024/12/article-name.md",
      "tags": ["ai", "programming"],
      "images_count": 3
    }
  ],
  "tags": {
    "ai": 15,
    "programming": 20,
    "tutorial": 10
  }
}
```
~~~

### GitHub Actions 설정 (선택)

자동 배포 및 검증을 위한 워크플로우:

`.github/workflows/daily-sync.yml`:

```yaml
name: Daily Medium Sync

on:
  schedule:
    - cron: '0 9 * * *'  # 매일 오전 9시
  workflow_dispatch:  # 수동 실행 가능

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          npm install -g @anthropic-ai/claude-code
          npm install -g @modelcontextprotocol/server-playwright
          playwright install chromium
      
      - name: Run sync
        env:
          CLAUDE_API_KEY: ${{ secrets.CLAUDE_API_KEY }}
          MEDIUM_EMAIL: ${{ secrets.MEDIUM_EMAIL }}
          MEDIUM_PASSWORD: ${{ secrets.MEDIUM_PASSWORD }}
        run: |
          claude-code run --config sync-config.json --headless
      
      - name: Commit and push
        run: |
          git config user.name "GitHub Actions Bot"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Auto-sync: $(date +'%Y-%m-%d')" || echo "No changes"
          git push

  validate:
    needs: sync
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate Markdown
        run: |
          npm install -g markdownlint-cli
          markdownlint 'articles/**/*.md'
      
      - name: Check broken links
        run: |
          npm install -g markdown-link-check
          find articles -name '*.md' -exec markdown-link-check {} \;
```

---

## 전체 워크플로우

### CLAUDE.md 설정

~~~markdown
# Medium Archive Project

당신은 Medium 콘텐츠 수집 및 GitHub 아카이빙 전문 에이전트입니다.

## 역할

Medium에서 글을 수집하고, 정리하여, GitHub 저장소에 체계적으로 저장합니다.

## 작업 프로세스

### 1. 수집 단계
1. `scripts/targets.json`에서 수집 대상 확인
2. Playwright로 각 대상 페이지 방문
3. `medium_scraper` 스킬로 콘텐츠 추출
4. 이미지 다운로드 및 로컬 저장

### 2. 변환 단계
1. `medium_to_markdown` 스킬로 HTML → Markdown 변환
2. 이미지 경로를 로컬 경로로 변경
3. YAML 프론트매터 추가
4. 파일명 생성: `YYYY-MM-DD-{slug}.md`

### 3. 저장 단계
1. 날짜별 폴더에 파일 저장: `articles/YYYY/MM/`
2. 이미지를 `articles/YYYY/MM/images/`에 저장
3. `metadata/index.json` 업데이트

### 4. 발행 단계
1. Git MCP로 변경사항 스테이징
2. 적절한 커밋 메시지 생성
3. GitHub에 푸시
4. 로그 파일 업데이트

## 스킬 사용 순서

```
targets.json 읽기
  ↓
FOR EACH target:
  medium_scraper (URL → JSON)
    ↓
  medium_to_markdown (JSON → MD)
    ↓
  저장 (MD + images)
    ↓
  metadata 업데이트
  ↓
github_publisher (Git commit & push)
  ↓
로그 기록
```

## 중복 방지

수집 전 확인:
1. `metadata/index.json`에서 URL 존재 여부 체크
2. 이미 있으면 스킵 또는 업데이트 여부 확인
3. 새 글만 처리

## 에러 처리

- 접근 불가 URL: 로그 기록 후 계속
- 이미지 다운로드 실패: 원본 URL 유지
- Git 충돌: 사용자에게 알림 후 중단
- Playwright 타임아웃: 재시도 (최대 3회)

## 파일 명명 규칙

```
articles/YYYY/MM/YYYY-MM-DD-article-slug.md
articles/YYYY/MM/images/article-slug-img-001.png
```
~~~

### 수집 대상 설정

`scripts/targets.json`:

```json
{
  "tags": [
    {
      "name": "ai",
      "url": "https://medium.com/tag/artificial-intelligence",
      "max_articles": 10
    },
    {
      "name": "claude",
      "url": "https://medium.com/tag/claude-ai",
      "max_articles": 5
    }
  ],
  "authors": [
    {
      "username": "@anthropic",
      "url": "https://medium.com/@anthropic",
      "max_articles": 20
    }
  ],
  "specific_articles": [
    "https://medium.com/@author/specific-article-slug-123abc"
  ],
  "publications": [
    {
      "name": "Towards Data Science",
      "url": "https://towardsdatascience.com",
      "filter_tags": ["ai", "machine-learning"],
      "max_articles": 15
    }
  ]
}
```

---

## 자동화 및 스케줄링

### 1. 로컬 Cron (Linux/macOS)

```bash
# crontab -e
# 매일 오전 9시에 실행
0 9 * * * cd /path/to/my-medium-archive && claude-code run --config sync-config.json --headless

# 주간 전체 동기화 (일요일 오전 2시)
0 2 * * 0 cd /path/to/my-medium-archive && claude-code run --config full-sync-config.json --headless
```

### 2. Windows 작업 스케줄러

**배치 파일 생성** (`sync-medium.bat`):

```batch
cd C:\path\to\my-medium-archive
claude-code run --config sync-config.json --headless
```

**작업 스케줄러 설정**:
1. "작업 스케줄러" 실행
2. "기본 작업 만들기"
3. 이름: "Medium Daily Sync"
4. 트리거: 매일 오전 9시
5. 작업: `C:\path\to\sync-medium.bat` 실행

### 3. GitHub Actions (클라우드 자동화)

위에서 작성한 `.github/workflows/daily-sync.yml` 사용

**장점**:
- 로컬 컴퓨터 꺼져있어도 실행
- 무료 (월 2000분 제공)
- 로그 확인 용이

**단점**:
- API 키 필요
- 실행 시간 제한 (최대 6시간)

---

## 실전 예제

### 예제 1: AI 관련 Medium 글 수집

**목표**: #ai, #machine-learning 태그의 최신 글 20개 수집

**단계 1**: 대상 설정

`scripts/ai-collection.json`:
```json
{
  "targets": [
    {
      "type": "tag",
      "name": "ai",
      "url": "https://medium.com/tag/artificial-intelligence",
      "max": 10
    },
    {
      "type": "tag",
      "name": "machine-learning",
      "url": "https://medium.com/tag/machine-learning",
      "max": 10
    }
  ],
  "options": {
    "download_images": true,
    "skip_existing": true,
    "commit_per_article": false
  }
}
```

**단계 2**: Claude Code 실행

```bash
claude-code chat "
scripts/ai-collection.json의 설정대로 Medium 글을 수집하세요:

1. 각 태그 페이지에서 최신 글 링크 추출
2. 각 글을 medium_scraper로 수집
3. medium_to_markdown으로 변환
4. articles/2024/12/ 에 저장
5. metadata/index.json 업데이트
6. github_publisher로 커밋 및 푸시

진행 상황을 실시간으로 보고해주세요.
"
```

**예상 출력**:
```
[1/20] Scraping: "Understanding GPT-4 Architecture"
  ✓ Content extracted
  ✓ 3 images downloaded
  ✓ Converted to Markdown
  ✓ Saved: articles/2024/12/2024-12-15-understanding-gpt4.md

[2/20] Scraping: "Fine-tuning LLMs for Production"
  ✓ Content extracted
  ✓ 5 images downloaded
  ✓ Converted to Markdown
  ✓ Saved: articles/2024/12/2024-12-18-finetuning-llms.md

...

[20/20] Complete!

📊 Summary:
- Total collected: 20 articles
- Images downloaded: 67
- Total size: 12.3 MB
- Time taken: 5m 23s

🔄 Git operations:
- Files added: 87 (20 MD + 67 images)
- Commit: "Add: 20 AI articles from Medium (2024-12-24)"
- Pushed to: origin/main

✅ Success! View at: https://github.com/YOUR_USERNAME/my-medium-archive
```

### 예제 2: 특정 작성자 팔로우

**목표**: @anthropic 계정의 모든 글 백업

```bash
claude-code chat "
Playwright로 https://medium.com/@anthropic 접속하여:
1. 모든 발행 글 목록 추출
2. 각 글을 순차적으로 수집
3. Markdown 변환 후 저장
4. 작성자별 폴더 생성: articles/authors/anthropic/
5. GitHub에 커밋

로그인은 하지 말고 공개 글만 수집하세요.
"
```

### 예제 3: 북마크한 글 수집 (로그인 필요)

**주의**: 이 경우 Medium 로그인이 필요합니다.

```bash
claude-code chat "
Playwright로 Medium에 로그인 후:

1. 로그인:
   - https://medium.com/m/signin 접속
   - 환경변수 MEDIUM_EMAIL, MEDIUM_PASSWORD 사용
   - 로그인 완료 대기

2. 북마크 페이지:
   - https://medium.com/me/list/reading-list 접속
   - 모든 북마크 글 링크 추출

3. 각 글 수집 및 저장:
   - articles/bookmarks/ 폴더에 저장
   - 특별한 태그 추가: 'bookmarked'

4. GitHub 커밋
"
```

### 예제 4: 헤드리스 배치 실행

**설정 파일** (`batch-config.json`):

```json
{
  "mode": "headless",
  "log_level": "info",
  "tasks": [
    {
      "name": "Daily AI News",
      "targets": "scripts/ai-collection.json",
      "output": "articles/",
      "git_commit": true,
      "git_message_template": "Daily sync: AI articles ({count} new)"
    }
  ],
  "error_handling": {
    "continue_on_error": true,
    "max_retries": 3,
    "notify_on_failure": true
  }
}
```

**실행**:

```bash
claude-code run --config batch-config.json --headless > logs/sync-$(date +%Y%m%d).log 2>&1
```

---

## 고급 기능

### 1. 이미지 최적화

`skills/image_optimizer.md`:

```markdown
# Image Optimizer Skill

## 목적
다운로드한 이미지를 최적화하여 저장소 크기 감소

## 처리
1. WebP 변환 (PNG/JPG → WebP)
2. 리사이즈 (max width: 1200px)
3. 압축 (quality: 85%)
4. 썸네일 생성 (선택)

## 도구
- Sharp 라이브러리 사용
```

**사용**:
```bash
npm install sharp

# Claude Code에서
claude-code chat "articles/2024/12/images/ 폴더의 모든 이미지를 image_optimizer로 최적화해주세요."
```

### 2. 태그 기반 자동 분류

`metadata/tag-rules.json`:

```json
{
  "rules": [
    {
      "tags": ["ai", "machine-learning", "deep-learning"],
      "category": "AI & ML",
      "folder": "articles/categories/ai-ml/"
    },
    {
      "tags": ["programming", "coding", "development"],
      "category": "Programming",
      "folder": "articles/categories/programming/"
    }
  ]
}
```

### 3. README 자동 생성

`skills/readme_generator.md`:

```markdown
# README Generator Skill

## 목적
저장소의 README.md를 메타데이터 기반으로 자동 생성/업데이트

## 포함 내용
- 총 수집 글 개수
- 최신 글 10개
- 태그별 통계
- 작성자별 통계
- 월별 아카이브 링크
```

**생성 예시**:

```markdown
# My Medium Archive

자동으로 수집된 Medium 글 아카이브

## 📊 통계

- **총 글 개수**: 156
- **총 작성자**: 42
- **수집 기간**: 2024-01-01 ~ 2024-12-24
- **마지막 업데이트**: 2024-12-24 09:00:00

## 📝 최신 글

1. [Understanding GPT-4 Architecture](articles/2024/12/2024-12-15-understanding-gpt4.md) - @author1
2. [Fine-tuning LLMs for Production](articles/2024/12/2024-12-18-finetuning-llms.md) - @author2
...

## 🏷️ 태그별 분류

- [AI & Machine Learning](tags/ai.md) (45)
- [Programming](tags/programming.md) (38)
- [Web Development](tags/webdev.md) (29)
...

## 📅 월별 아카이브

- [2024년 12월](articles/2024/12/) (23 글)
- [2024년 11월](articles/2024/11/) (31 글)
...

## 🔍 검색

전체 글 인덱스: [index.md](metadata/index.md)

## 🤖 자동화

이 저장소는 Claude Code를 통해 자동으로 관리됩니다.
```

### 4. 중복 탐지 및 버전 관리

```markdown
# Duplicate Detector Skill

## 목적
동일 글의 중복 수집 방지 및 업데이트 탐지

## 판별 기준
1. URL 일치 (정확한 중복)
2. 제목 유사도 > 90% (잠재적 중복)
3. 본문 해시 비교 (내용 변경 탐지)

## 처리
- 신규: 새로 추가
- 중복: 스킵
- 업데이트: 버전 번호 증가 (v1, v2, ...)
```

---

## 트러블슈팅

### 1. Medium 접근 차단

**증상**:
```
Error: Access denied (429 Too Many Requests)
```

**해결**:
1. 요청 간 대기 시간 증가 (3-5초)
2. User-Agent 헤더 변경
3. Residential Proxy 사용 (유료)
4. VPN 사용

### 2. Paywall 글 접근

**증상**:
```
Member-only story
```

**해결**:
- Medium 구독 계정으로 로그인 필요
- 또는 미리보기 부분만 수집
- Paywall 글은 별도 표시

### 3. 이미지 다운로드 실패

**증상**:
```
Failed to download image: https://...
```

**해결**:
1. Referer 헤더 추가
2. 쿠키 포함
3. 실패한 이미지는 원본 URL 유지

### 4. Git 충돌

**증상**:
```
CONFLICT (content): Merge conflict in metadata/index.json
```

**해결**:
```bash
# 자동 해결 시도
git pull --rebase --strategy-option=theirs

# 수동 해결 필요시
git mergetool
```

---

## 체크리스트

### 초기 설정
- [ ] GitHub 저장소 생성
- [ ] Claude Code 설치 및 인증
- [ ] MCP 서버 설정 (Playwright, Git, Filesystem)
- [ ] 환경 변수 설정 (.env)
- [ ] 디렉토리 구조 생성

### 스킬 개발
- [ ] medium_scraper.md
- [ ] medium_to_markdown.md
- [ ] github_publisher.md
- [ ] image_optimizer.md (선택)
- [ ] readme_generator.md (선택)

### 설정 파일
- [ ] CLAUDE.md
- [ ] scripts/targets.json
- [ ] .claude/mcp_config.json
- [ ] .github/workflows/daily-sync.yml (선택)

### 테스트
- [ ] 단일 글 수집 테스트
- [ ] 이미지 다운로드 테스트
- [ ] Markdown 변환 품질 확인
- [ ] Git 커밋/푸시 테스트
- [ ] 중복 방지 테스트

### 자동화
- [ ] 로컬 Cron/스케줄러 설정
- [ ] 또는 GitHub Actions 설정
- [ ] 에러 알림 설정
- [ ] 로그 모니터링 설정

---

## 추가 리소스

### 유용한 도구

1. **Markdown 에디터**
   - Obsidian (로컬 Markdown 관리)
   - Typora (WYSIWYG Markdown)

2. **Git GUI**
   - GitHub Desktop (초보자용)
   - Sourcetree (고급 기능)

3. **이미지 처리**
   - ImageMagick (CLI 도구)
   - Sharp (Node.js 라이브러리)

### 참고 링크

- [Playwright 문서](https://playwright.dev)
- [MCP 프로토콜](https://modelcontextprotocol.io)
- [GitHub API](https://docs.github.com/en/rest)
- [Medium API (비공식)](https://github.com/Medium/medium-api-docs)

### 확장 아이디어

1. **전문 검색 시스템**
   - Algolia 또는 MeiliSearch 통합
   - 전문 검색 웹 인터페이스

2. **AI 요약 추가**
   - 각 글마다 AI 요약 생성
   - 핵심 키워드 자동 추출

3. **RSS 피드 생성**
   - 수집된 글의 RSS 피드
   - 태그별 필터링 가능

4. **정적 사이트 생성**
   - Jekyll/Hugo로 블로그 형태 변환
   - GitHub Pages로 호스팅

---

**작성 일자: 2025-12-24**
**버전**: 1.0
**라이선스**: MIT
