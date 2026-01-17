---
title: "GSD Daily Work Log 활용 및 Claude Code + Google Antigravity 통합 전략 가이드"
date: 2026-01-11 13:00:00 +0900
categories: [AI,  Guide]
mermaid: [True]
tags: [AI,  Guide,  Claude,  claude-code,  Antigravity,  GSD,  MCP,  Guide,  Claude.write]
---



## 목차
1. [개요](#개요)
2. [Daily Work Log로 GSD 활용하기](#daily-work-log로-gsd-활용하기)
3. [Claude Code와 Google Antigravity 이해하기](#claude-code와-google-antigravity-이해하기)
4. [통합 아키텍처 설계](#통합-아키텍처-설계)
5. [실전 통합 전략](#실전-통합-전략)
6. [MCP를 활용한 고급 워크플로우](#mcp를-활용한-고급-워크플로우)
7. [Daily Work Log 자동화 시스템](#daily-work-log-자동화-시스템)
8. [실전 시나리오 및 베스트 프랙티스](#실전-시나리오-및-베스트-프랙티스)
9. [문제 해결 및 최적화](#문제-해결-및-최적화)

---

## 개요

현대적인 AI 기반 개발 환경은 단일 도구의 한계를 넘어서고 있습니다. 이 가이드는 세 가지 강력한 도구를 전략적으로 결합하여 개발 생산성을 극대화하는 방법을 제시합니다:

### 핵심 도구 생태계

**GSD (Get Shit Done)**
- 역할: 프로젝트 구조화 및 작업 관리 레이어
- 강점: 컨텍스트 엔지니어링, 원자적 작업 분해, 자동 문서화
- 활용: 프로젝트의 "두뇌" 역할

**Claude Code (Anthropic)**
- 역할: AI 기반 코딩 에이전트
- 강점: 코드 이해, 복잡한 리팩토링, MCP 통합
- 활용: 코드 생성 및 수정의 "실행자"

**Google Antigravity**
- 역할: 에이전트 우선 통합 개발 환경 (IDE)
- 강점: 브라우저 제어, 시각적 피드백, 멀티모달 기능
- 활용: 엔드-투-엔드 작업 자동화의 "오케스트레이터"

### 왜 통합이 필요한가?

각 도구는 특정 영역에서 탁월하지만, 진정한 힘은 통합에서 나옵니다:

1. **GSD의 한계**: 코드 생성은 우수하지만 브라우저 기반 테스트나 UI 검증에는 제한적
2. **Claude Code의 한계**: 강력한 코딩 능력이지만 시각적 피드백이 부족
3. **Antigravity의 한계**: 훌륭한 IDE이지만 장기 프로젝트 구조화에는 GSD만큼 체계적이지 않음

통합을 통해 각 도구의 강점을 극대화하고 약점을 보완할 수 있습니다.

---

## Daily Work Log로 GSD 활용하기

GSD는 기본적으로 작업 기록을 자동으로 생성하는 시스템입니다. 이를 효과적인 Daily Work Log 시스템으로 전환하는 방법을 알아보겠습니다.

### GSD의 기본 작업 기록 메커니즘

GSD는 프로젝트 진행 중 세 가지 핵심 문서를 지속적으로 업데이트합니다:

**STATE.md - 현재 상태 기록**
~~~markdown
# 프로젝트 상태 - 2026년 1월 11일

## 현재 단계
Phase 3: 사용자 인증 시스템 구현 (진행 중)

## 완료된 작업
✅ Phase 1: 프로젝트 초기 설정 (2026-01-08 완료)
  - Next.js 15 프로젝트 생성
  - Tailwind CSS 설정
  - TypeScript strict 모드 활성화

✅ Phase 2: 데이터베이스 스키마 (2026-01-09 완료)
  - Supabase 연결 설정
  - users, tasks 테이블 생성
  - RLS 정책 적용

🔄 Phase 3: 사용자 인증 (진행 중, 60% 완료)
  ✅ Task 3-1: Google OAuth 설정
  ✅ Task 3-2: 로그인 페이지 UI
  🔄 Task 3-3: 세션 관리 (현재 작업)
  ⏳ Task 3-4: 비밀번호 재설정 플로우

## 오늘의 결정 사항
1. JWT 대신 Supabase Auth의 내장 세션 사용하기로 결정
   - 이유: 보안 업데이트 자동 적용, 관리 부담 감소
   
2. 리마인더 기능 Phase 5로 연기
   - 이유: 핵심 인증 기능에 먼저 집중

## 발견된 이슈
- 이슈 #12: Supabase RLS 정책이 특정 쿼리에서 과도한 지연 발생
  - 우선순위: 중간
  - 해결 방안: 인덱스 추가 예정

## 성능 지표
- 커밋 수: 18개
- 테스트 커버리지: 72%
- 평균 작업 완료 시간: 25분/작업
~~~

**Git 커밋 히스토리 - 자동 생성되는 상세 로그**
```bash
# GSD가 자동으로 생성한 커밋들
d3a1c9f feat(03-03): 세션 관리 미들웨어 구현
b7e4f2a test(03-03): 세션 만료 테스트 추가
8c6d1e3 feat(03-02): 로그인 페이지 UI 구현
5a9b2c7 docs(03-02): 로그인 페이지 완료 요약
2f8e4a1 feat(03-01): Google OAuth 콜백 처리

```

**SUMMARY.md - 작업 완료 후 생성되는 요약**
```markdown
# Phase 3, Task 3-1 완료 요약
작업일: 2026-01-10
소요 시간: 28분

## 수행 내용
Google OAuth 인증 플로우를 완전히 구현했습니다:
- Supabase Auth 설정에 Google Provider 추가
- 환경 변수에 Google Client ID/Secret 설정
- 콜백 라우트 (/api/auth/callback) 생성
- 로그인 버튼 컴포넌트 구현

## 변경된 파일
- `app/api/auth/callback/route.ts` (신규)
- `components/LoginButton.tsx` (신규)
- `lib/supabase.ts` (수정)
- `.env.local` (업데이트)

## 검증 결과
✅ Google 로그인 버튼 클릭 시 OAuth 플로우 시작
✅ 인증 성공 후 대시보드로 리디렉션
✅ 사용자 정보가 Supabase users 테이블에 저장
✅ 세션 쿠키 정상 설정

## 발견된 문제 및 해결
- 문제: 개발 환경에서 리디렉션 URL 불일치
- 해결: Supabase 콘솔에 localhost:3000 추가

## 다음 단계
Task 3-2: 로그인 페이지 UI 구현 예정
```

### Daily Work Log 시스템 구축

GSD의 자동 기록을 활용하여 체계적인 Daily Work Log를 만드는 방법입니다.

#### 1단계: 자동화된 일일 요약 생성

GSD 커스텀 명령어를 생성하여 하루 작업을 요약합니다:

`.claude/commands/daily-log.md`:
~~~markdown
# /daily-log 명령어

오늘 수행한 모든 작업을 Daily Work Log 형식으로 요약합니다.

## 수행 작업

1. STATE.md에서 오늘 완료된 작업 추출
2. Git 커밋 로그 분석 (오늘 날짜)
3. SUMMARY.md 파일들 취합
4. ISSUES.md에서 오늘 발견된 이슈 확인
5. 다음 작업일을 위한 To-Do 생성

## 출력 형식

```markdown
# Daily Work Log - [날짜]

## 📊 요약
- 완료한 Phase/Task: X개
- 생성한 커밋: X개
- 작성한 코드: +XXX -XXX 라인
- 해결한 이슈: X개
- 발견한 새 이슈: X개

## ✅ 완료한 작업
### [Task ID]: [작업 이름]
- 설명: [간단한 설명]
- 소요 시간: [XX분]
- 주요 변경 사항:
  - [파일명]: [변경 내용]
- 검증 완료: [검증 결과]

## 🔧 기술적 결정
1. [결정 사항]
   - 이유: [근거]
   - 영향: [프로젝트에 미치는 영향]

## 🐛 발견한 이슈
- [이슈 #X]: [이슈 설명]
  - 우선순위: [높음/중간/낮음]
  - 계획된 해결 방법: [방법]

## 📚 학습한 내용
- [새로 배운 기술/개념/패턴]

## ⏭️ 다음 작업일 계획
- [ ] [다음에 할 작업 1]
- [ ] [다음에 할 작업 2]

## 💭 회고
- 잘된 점: [무엇이 잘 됐는지]
- 개선할 점: [다음에 더 잘할 수 있는 부분]
```

## 파일 저장 위치
`.planning/daily-logs/YYYY-MM-DD.md`
~~~

#### 사용 방법

업무 종료 시:
```bash
# Claude Code에서
/daily-log
```

GSD는 자동으로:
1. 오늘의 모든 활동 분석
2. Daily Work Log 생성
3. `.planning/daily-logs/` 디렉토리에 저장
4. 선택적으로 Slack/이메일로 전송 (설정 시)

#### 2단계: 주간/월간 요약 자동 생성

일일 로그를 기반으로 주간 및 월간 요약을 생성하는 명령어:

`.claude/commands/weekly-summary.md`:
~~~markdown
# /weekly-summary 명령어

지난 주의 모든 Daily Work Log를 분석하여 주간 요약을 생성합니다.

## 수행 작업
1. `.planning/daily-logs/`에서 지난 7일간의 로그 로드
2. 주요 성과 취합
3. 시간 투자 분석 (어느 Phase에 가장 많은 시간을 쓰는지)
4. 이슈 트렌드 분석
5. 다음 주 우선순위 제안

## 출력 형식
```markdown
# Weekly Summary - Week of [날짜]

## 🎯 주요 성과
- 완료한 Phase: X개
- 완료한 작업: X개
- 총 커밋 수: X개
- 코드 변경: +XXXX -XXXX 라인

## 📈 시간 투자 분석
[막대 그래프 형식 텍스트]
Phase 1: ████████░░ 45%
Phase 2: ███░░░░░░░ 30%
Phase 3: ██░░░░░░░░ 25%

## 🏆 주요 마일스톤
1. [달성한 마일스톤 1]
2. [달성한 마일스톤 2]

## ⚠️ 블로커 및 지연
- [발생한 문제와 해결 방법]

## 📊 성능 지표 추이
- 평균 작업 완료 시간: XX분 (전주 대비 ±X%)
- 테스트 커버리지: XX% (전주 대비 ±X%)
- 재작업률: X% (버그 수정/전체 작업)

## 🔮 다음 주 계획
- [ ] [우선순위 1]
- [ ] [우선순위 2]
- [ ] [우선순위 3]
```
~~~

#### 3단계: 팀 공유 및 협업

GSD의 Daily Work Log를 팀과 공유하는 워크플로우:

**Slack 통합 예시**

`.claude/commands/share-daily-log.md`:
~~~markdown
# /share-daily-log 명령어

오늘의 Daily Work Log를 Slack 채널에 공유합니다.

## 전제 조건
- Slack Webhook URL이 환경 변수에 설정되어 있어야 함
- `SLACK_WEBHOOK_URL=https://hooks.slack.com/...`

## 수행 작업
1. `/daily-log` 실행하여 로그 생성
2. Slack 포맷으로 변환 (Markdown → Slack Blocks)
3. 지정된 채널에 포스팅
4. 중요 이슈가 있으면 @channel 멘션

## Slack 메시지 포맷
```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📅 Daily Work Log - 2026-01-11"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*완료한 작업:* 3개"
        },
        {
          "type": "mrkdwn",
          "text": "*생성한 커밋:* 12개"
        }
      ]
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*주요 성과*\n✅ 사용자 인증 시스템 완료\n✅ E2E 테스트 80% 도달"
      }
    }
  ]
}
```
~~~

#### 4단계: 성과 추적 대시보드

GSD의 STATE.md와 커밋 히스토리를 활용한 실시간 대시보드:

**메트릭 수집 스크립트**

`.claude/scripts/collect-metrics.js`:
```javascript
#!/usr/bin/env node

const fs = require('fs');
const { execSync } = require('child_process');

function collectMetrics() {
  const metrics = {
    date: new Date().toISOString().split('T')[0],
    tasks: {
      completed: 0,
      inProgress: 0,
      blocked: 0
    },
    commits: 0,
    linesChanged: { added: 0, deleted: 0 },
    testCoverage: 0,
    issues: {
      opened: 0,
      closed: 0,
      critical: 0
    }
  };

  // STATE.md 파싱
  const stateContent = fs.readFileSync('.planning/STATE.md', 'utf-8');
  metrics.tasks.completed = (stateContent.match(/✅/g) || []).length;
  metrics.tasks.inProgress = (stateContent.match(/🔄/g) || []).length;
  metrics.tasks.blocked = (stateContent.match(/🚫/g) || []).length;

  // Git 통계
  const today = new Date().toISOString().split('T')[0];
  const commitCount = execSync(
    `git log --since="${today}T00:00:00" --until="${today}T23:59:59" --oneline | wc -l`
  ).toString().trim();
  metrics.commits = parseInt(commitCount);

  // 코드 변경 통계
  const diffStats = execSync(
    `git diff --shortstat origin/main`
  ).toString();
  const addedMatch = diffStats.match(/(\d+) insertions/);
  const deletedMatch = diffStats.match(/(\d+) deletions/);
  metrics.linesChanged.added = addedMatch ? parseInt(addedMatch[1]) : 0;
  metrics.linesChanged.deleted = deletedMatch ? parseInt(deletedMatch[1]) : 0;

  // ISSUES.md 파싱
  if (fs.existsSync('.planning/ISSUES.md')) {
    const issuesContent = fs.readFileSync('.planning/ISSUES.md', 'utf-8');
    const openIssues = issuesContent.match(/- \[ \]/g) || [];
    const closedIssues = issuesContent.match(/- \[x\]/gi) || [];
    const criticalIssues = issuesContent.match(/우선순위.*높음/gi) || [];
    
    metrics.issues.opened = openIssues.length;
    metrics.issues.closed = closedIssues.length;
    metrics.issues.critical = criticalIssues.length;
  }

  return metrics;
}

// 메트릭 저장
const metrics = collectMetrics();
const metricsDir = '.planning/metrics';
if (!fs.existsSync(metricsDir)) {
  fs.mkdirSync(metricsDir, { recursive: true });
}

const filename = `${metricsDir}/${metrics.date}.json`;
fs.writeFileSync(filename, JSON.stringify(metrics, null, 2));

console.log('📊 Metrics collected:', metrics);
console.log(`📁 Saved to: ${filename}`);
```

**대시보드 HTML 생성**

`.claude/scripts/generate-dashboard.js`:
```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

function generateDashboard() {
  const metricsDir = '.planning/metrics';
  const files = fs.readdirSync(metricsDir)
    .filter(f => f.endsWith('.json'))
    .sort()
    .reverse()
    .slice(0, 30); // 최근 30일

  const metricsData = files.map(f => {
    return JSON.parse(fs.readFileSync(path.join(metricsDir, f), 'utf-8'));
  }).reverse();

  const html = `
<!DOCTYPE html>
<html>
<head>
  <title>GSD Project Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
      background: #f5f5f5;
    }
    .dashboard-header {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 30px;
      border-radius: 10px;
      margin-bottom: 20px;
    }
    .metrics-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      margin-bottom: 30px;
    }
    .metric-card {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .metric-value {
      font-size: 36px;
      font-weight: bold;
      color: #667eea;
    }
    .metric-label {
      color: #666;
      margin-top: 5px;
    }
    .chart-container {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <div class="dashboard-header">
    <h1>🚀 GSD Project Dashboard</h1>
    <p>Last updated: ${new Date().toLocaleString()}</p>
  </div>

  <div class="metrics-grid">
    <div class="metric-card">
      <div class="metric-value">${metricsData[metricsData.length - 1].tasks.completed}</div>
      <div class="metric-label">Tasks Completed</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">${metricsData[metricsData.length - 1].commits}</div>
      <div class="metric-label">Today's Commits</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">${metricsData[metricsData.length - 1].tasks.inProgress}</div>
      <div class="metric-label">In Progress</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">${metricsData[metricsData.length - 1].issues.critical}</div>
      <div class="metric-label">Critical Issues</div>
    </div>
  </div>

  <div class="chart-container">
    <canvas id="tasksChart"></canvas>
  </div>

  <div class="chart-container">
    <canvas id="commitsChart"></canvas>
  </div>

  <script>
    const dates = ${JSON.stringify(metricsData.map(m => m.date))};
    const tasksCompleted = ${JSON.stringify(metricsData.map(m => m.tasks.completed))};
    const commits = ${JSON.stringify(metricsData.map(m => m.commits))};

    // Tasks Chart
    new Chart(document.getElementById('tasksChart'), {
      type: 'line',
      data: {
        labels: dates,
        datasets: [{
          label: 'Completed Tasks',
          data: tasksCompleted,
          borderColor: '#667eea',
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          tension: 0.4
        }]
      },
      options: {
        responsive: true,
        plugins: {
          title: {
            display: true,
            text: 'Tasks Completed Over Time'
          }
        }
      }
    });

    // Commits Chart
    new Chart(document.getElementById('commitsChart'), {
      type: 'bar',
      data: {
        labels: dates,
        datasets: [{
          label: 'Daily Commits',
          data: commits,
          backgroundColor: '#764ba2'
        }]
      },
      options: {
        responsive: true,
        plugins: {
          title: {
            display: true,
            text: 'Daily Commit Activity'
          }
        }
      }
    });
  </script>
</body>
</html>
  `;

  fs.writeFileSync('.planning/dashboard.html', html);
  console.log('📊 Dashboard generated: .planning/dashboard.html');
}

generateDashboard();
```

**자동화 설정**

매일 자동으로 메트릭 수집 및 대시보드 업데이트:

```bash
# crontab 설정
0 18 * * * cd /path/to/project && node .claude/scripts/collect-metrics.js
5 18 * * * cd /path/to/project && node .claude/scripts/generate-dashboard.js
```

---

## Claude Code와 Google Antigravity 이해하기

두 도구의 차이점과 상호 보완적인 역할을 이해하는 것이 효과적인 통합의 첫걸음입니다.

### Claude Code 심층 분석

**핵심 철학**: "코드 중심의 대화형 개발"

Claude Code는 터미널에서 실행되는 AI 코딩 어시스턴트로, 다음과 같은 특징을 가집니다:

#### 강점

**1. 탁월한 코드 이해력**
```bash
# 복잡한 레거시 코드도 이해하고 리팩토링
claude "이 Redux 코드를 Zustand로 마이그레이션하고, 
타입 안전성을 유지하면서 보일러플레이트를 최소화해줘"
```

Claude Code는:
- 전체 코드베이스의 패턴을 이해
- 의존성 그래프 분석
- 변경 사항의 ripple effect 예측
- 안전한 리팩토링 경로 제시

**2. MCP (Model Context Protocol) 생태계**

Claude Code의 진정한 힘은 MCP 통합에 있습니다:

```bash
# GitHub MCP 서버 연결
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# Slack MCP 서버 연결
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack

# PostgreSQL MCP 서버 연결
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
```

이를 통해 Claude Code는:
- GitHub 이슈와 PR을 직접 조회하고 생성
- Slack 메시지 전송 및 대화 검색
- 데이터베이스 쿼리 및 스키마 분석
- 파일 시스템 접근 및 조작

**3. 컨텍스트 관리의 달인**

`CLAUDE.md` 파일을 통한 프로젝트별 지침:
```markdown
# CLAUDE.md

## 코딩 스타일
- 함수는 최대 20줄로 제한
- 모든 public 함수에 JSDoc 주석 필수
- async/await 사용, Promise.then() 금지

## 아키텍처 규칙
- 비즈니스 로직은 services/ 디렉토리에만
- React 컴포넌트는 UI 로직만 포함
- 글로벌 상태는 Zustand 스토어에서만

## 테스팅 규칙
- 모든 서비스 함수는 단위 테스트 필수
- React 컴포넌트는 RTL로 테스트
- E2E 테스트는 주요 사용자 플로우만

## 금지 사항
- any 타입 사용 금지
- console.log 커밋 금지
- 인라인 스타일 금지
```

Claude는 이 규칙을 모든 작업에 자동으로 적용합니다.

#### 제한사항

1. **시각적 피드백 부족**: 터미널 기반이므로 UI를 직접 볼 수 없음
2. **브라우저 상호작용 불가**: E2E 테스트를 실행할 수는 있지만 브라우저를 직접 제어하지는 못함
3. **비동기 장기 작업에 제한적**: 수십 분 걸리는 작업을 완전 자동화하기 어려움

### Google Antigravity 심층 분석

**핵심 철학**: "에이전트 우선 개발 환경"

Antigravity는 AI 에이전트가 자율적으로 작업할 수 있도록 설계된 IDE입니다.

#### 강점

**1. 에이전트 우선 아키텍처**

Antigravity는 단순한 에디터가 아니라, 에이전트가 자율적으로 복잡한 작업을 계획, 실행, 검증할 수 있는 전용 공간을 제공하는 개발 플랫폼입니다.

**Editor View vs Agent View**

*Editor View*: 전통적인 IDE 경험
- VS Code 기반 인터페이스
- 탭 자동완성
- 인라인 편집 제안
- 동기적 작업 흐름

*Agent View*: 비동기 에이전트 작업
- 에이전트에게 높은 수준의 작업 위임
- 에이전트가 계획을 수립하고 실행
- 진행 상황을 Artifacts로 시각화
- 필요 시 사용자에게 피드백 요청

**2. 브라우저 제어 능력**

Antigravity의 Agent-Controlled Browser는 에이전트가 웹 기반 리소스와 직접 상호작용하여 컨텍스트 정보를 검색하고 검증할 수 있게 합니다.

실제 활용 예시:
```javascript
// Antigravity 에이전트에게 지시
"로그인 플로우를 구현한 후, 실제로 브라우저에서 
다음을 확인해줘:
1. 로그인 버튼 클릭
2. 이메일/비밀번호 입력
3. 제출 후 대시보드로 리디렉션되는지
4. 스크린샷과 함께 보고서 작성"
```

에이전트는:
- Chromium 브라우저 자동 실행
- 각 단계 수행 및 스크린샷 캡처
- 오류 발생 시 자동으로 디버깅 시도
- 완료 후 시각적 보고서(Artifact) 생성

**3. Artifacts를 통한 동적 피드백**

Artifacts는 에이전트가 생성하는 유형별 산출물로, 작업 목록, 구현 계획, 스크린샷, 브라우저 녹화 등을 포함합니다.

Artifact 유형:
- **계획(Plan)**: 작업을 어떻게 나눌 것인지
- **다이어그램(Diagram)**: 아키텍처 시각화
- **스크린샷(Screenshot)**: UI 검증 증거
- **비디오 녹화(Recording)**: 사용자 플로우 시연
- **테스트 결과(Test Results)**: 자동화된 테스트 출력

사용자는 Artifact에 직접 코멘트를 달 수 있고, 에이전트는 실행을 멈추지 않고 피드백을 통합합니다.

**4. 멀티모달 기능**

Antigravity는 텍스트, 이미지, 기타 미디어를 지원하여 더 풍부하고 다재다능한 개발 프로세스를 가능하게 합니다.

예시:
```
[Figma 디자인 파일을 Antigravity에 드래그 앤 드롭]

"이 디자인을 React + Tailwind CSS로 구현해줘. 
색상, 간격, 타이포그래피를 정확히 맞춰줘."
```

에이전트는:
- 이미지를 분석하여 색상 팔레트 추출
- 레이아웃 구조 파악
- 정확한 CSS 값으로 컴포넌트 생성
- 결과를 디자인과 나란히 비교하는 Artifact 생성

**5. 지식 베이스 시스템**

Antigravity는 학습을 핵심 기본 기능으로 다루며, 에이전트가 유용한 컨텍스트와 코드 스니펫을 지식 베이스에 저장하여 향후 작업을 개선할 수 있습니다.

워크플로우:
1. 에이전트가 까다로운 버그 해결
2. 해결 과정과 패턴을 지식 베이스에 저장
3. 유사한 문제 발생 시 자동으로 참조
4. 프로젝트가 진행될수록 에이전트가 점점 "똑똑해짐"

#### 제한사항

1. **장기 프로젝트 구조화 부족**: GSD 같은 체계적인 Phase/Task 분해 시스템이 없음
2. **컨텍스트 부패**: 여전히 컨텍스트 창 제한에 영향을 받음 (단, Gemini 3의 큰 컨텍스트 창으로 완화)
3. **안정성**: 초기 사용자들은 오류와 느린 생성 속도를 경험했다고 보고
4. **학습 곡선**: 에이전트 우선 사고방식에 적응 필요

### 도구 간 비교 및 상호 보완성

| 특성 | Claude Code | Google Antigravity | GSD |
|------|-------------|-------------------|-----|
| **실행 환경** | 터미널 | IDE (VS Code 포크) | Claude Code 플러그인 |
| **주요 AI 모델** | Claude Opus/Sonnet | Gemini 3, Claude, GPT-OSS | Claude (Claude Code 사용) |
| **브라우저 제어** | ❌ | ✅ | ❌ |
| **MCP 지원** | ✅ 네이티브 | ⚠️ 제한적 | ✅ (Claude Code 통해) |
| **시각적 피드백** | ❌ | ✅ (Artifacts) | ❌ |
| **프로젝트 구조화** | ⚠️ 기본적 | ⚠️ 기본적 | ✅ (Phase/Task 시스템) |
| **컨텍스트 부패 방지** | ⚠️ CLAUDE.md | ⚠️ 큰 컨텍스트 창 | ✅ (서브 에이전트) |
| **자동 문서화** | ❌ | ⚠️ Artifacts | ✅ (STATE, SUMMARY) |
| **Git 통합** | ✅ | ✅ | ✅ (자동 커밋) |
| **비동기 작업** | ❌ | ✅ | ✅ |
| **학습 곡선** | 낮음 | 중간-높음 | 중간 |
| **가격** | 유료 (Claude 구독) | 무료 (Preview) | 무료 (오픈소스) |

**상호 보완 전략:**

1. **GSD + Claude Code**: 구조화된 프로젝트 관리 + 강력한 코드 생성
2. **GSD + Antigravity**: 체계적 계획 + 시각적 검증 및 브라우저 테스트
3. **Claude Code + Antigravity**: MCP 생태계 + 브라우저 자동화
4. **GSD + Claude Code + Antigravity**: 완벽한 통합 (이 가이드의 목표!)

---

## 통합 아키텍처 설계

세 도구를 효과적으로 통합하려면 각 도구의 역할을 명확히 정의하고, 데이터 흐름을 설계해야 합니다.

### 통합 레이어 구조

```
┌─────────────────────────────────────────────────────────┐
│                    사용자 (개발자)                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              GSD (프로젝트 오케스트레이션)                 │
│  - PROJECT.md: 프로젝트 비전                              │
│  - ROADMAP.md: Phase/Task 구조                          │
│  - STATE.md: 현재 상태 추적                              │
│  - 작업 분해 및 스케줄링                                   │
└─────────────────────────────────────────────────────────┘
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
┌─────────────────────────┐   ┌─────────────────────────┐
│   Claude Code           │   │   Google Antigravity    │
│  (코드 생성 엔진)         │   │   (UI/E2E 검증 엔진)     │
├─────────────────────────┤   ├─────────────────────────┤
│ • 코드 작성              │   │ • 브라우저 자동화        │
│ • 리팩토링               │   │ • 시각적 검증            │
│ • 단위 테스트            │   │ • E2E 테스트             │
│ • MCP 통합               │   │ • 스크린샷/비디오        │
│ • Git 조작               │   │ • Artifacts 생성         │
└─────────────────────────┘   └─────────────────────────┘
            │                             │
            └──────────────┬──────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  MCP 생태계                              │
│  ┌──────────┬──────────┬──────────┬──────────┐         │
│  │ GitHub   │ Slack    │ Database │ Custom   │         │
│  │ MCP      │ MCP      │ MCP      │ MCP      │         │
│  └──────────┴──────────┴──────────┴──────────┘         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              외부 시스템 및 데이터                          │
│  • GitHub Repos  • Slack Workspace  • PostgreSQL       │
│  • Supabase      • Notion           • APIs             │
└─────────────────────────────────────────────────────────┘
```

### 역할 분담 전략

**GSD: 프로젝트 관리 레이어**
- **책임**: What (무엇을 만들 것인가), When (언제), Why (왜)
- **산출물**: 문서 (PROJECT.md, ROADMAP.md, STATE.md)
- **의사결정**: Phase 분해, 우선순위, 의존성 관리

**Claude Code: 코드 생성 및 조작 레이어**
- **책임**: How (어떻게 구현할 것인가) - 코드 레벨
- **산출물**: 코드, 단위 테스트, 리팩토링
- **의사결정**: 알고리즘, 데이터 구조, 아키텍처 패턴

**Google Antigravity: 검증 및 통합 레이어**
- **책임**: Verify (제대로 작동하는가) - 시각적 검증
- **산출물**: E2E 테스트, 스크린샷, 사용자 플로우 비디오
- **의사결정**: UI/UX 품질, 크로스 브라우저 호환성

### 데이터 흐름 설계

```
[사용자 요구사항]
       ↓
[GSD: 프로젝트 초기화]
       ↓
[GSD: Phase/Task 분해]
       ↓
[GSD: PLAN.md 생성]
       ↓
┌──────┴──────┐
↓             ↓
[Claude Code]  [Antigravity]
코드 생성      UI 구현
       ↓             ↓
[Git 커밋]     [스크린샷]
       ↓             ↓
[GSD: SUMMARY.md 생성]
       ↓
[STATE.md 업데이트]
       ↓
[다음 Task로]
```

### 통신 메커니즘

세 도구 간의 통신은 주로 파일 시스템을 통해 이루어집니다:

**공유 디렉토리 구조:**
```
project/
├── .planning/              # GSD 관리 문서
│   ├── PROJECT.md
│   ├── ROADMAP.md
│   ├── STATE.md
│   ├── PLAN.md
│   ├── SUMMARY.md
│   ├── ISSUES.md
│   └── daily-logs/
├── .claude/                # Claude Code 설정
│   ├── settings.json
│   ├── mcp-config.json
│   └── commands/
├── .antigravity/           # Antigravity 설정
│   ├── agents/
│   ├── artifacts/
│   └── knowledge-base/
├── CLAUDE.md               # 공통 프로젝트 규칙
└── [프로젝트 코드]
```

**메시지 전달:**

1. **GSD → Claude Code**
   - PLAN.md에 작업 지시 작성
   - Claude Code가 PLAN.md 읽고 실행
   - 완료 후 SUMMARY.md 생성

2. **GSD → Antigravity**
   - 검증 작업을 `.antigravity/tasks/` 에 JSON으로 저장
   - Antigravity 에이전트가 폴링하여 실행
   - 결과를 `.antigravity/artifacts/` 에 저장

3. **Claude Code ↔ Antigravity**
   - Git 커밋을 통한 간접 통신
   - MCP 서버를 공유하여 데이터 동기화

---

## 실전 통합 전략

이론을 실전으로 옮기는 구체적인 전략들입니다.

### 전략 1: 단계별 도구 선택 (Sequential Handoff)

각 Phase의 특성에 따라 최적의 도구를 선택합니다.

**워크플로우:**

```
Phase 1: 프로젝트 설정
→ GSD: 구조 설계
→ Claude Code: 보일러플레이트 생성
→ Antigravity: 개발 환경 검증

Phase 2: 데이터베이스 스키마
→ GSD: ERD 및 요구사항 정의
→ Claude Code: Prisma/마이그레이션 스크립트 작성
→ Claude Code MCP: PostgreSQL 연결 및 검증

Phase 3: 백엔드 API
→ GSD: API 엔드포인트 목록 및 스펙 정의
→ Claude Code: 코드 구현 및 단위 테스트
→ Claude Code MCP: 통합 테스트 (DB와 실제 연결)

Phase 4: 프론트엔드 UI
→ GSD: 화면 목록 및 사용자 플로우 정의
→ Antigravity: 컴포넌트 구현 (디자인 파일 기반)
→ Antigravity: 실시간 브라우저 미리보기

Phase 5: E2E 테스트
→ GSD: 테스트 시나리오 정의
→ Antigravity: Playwright 테스트 작성 및 실행
→ Antigravity: 스크린샷/비디오 증거 수집
```

**실제 명령어 예시:**

```bash
# Phase 1: 프로젝트 설정
cd my-project
claude --dangerously-skip-permissions

# GSD로 프로젝트 초기화
/gsd:new-project
# ... 프로젝트 정보 입력 ...
/gsd:create-roadmap

# Claude Code로 보일러플레이트 생성
/gsd:plan-phase 1
/gsd:execute-plan

# 완료 후 Antigravity로 전환하여 검증
# (Antigravity IDE 오픈)
"프로젝트를 개발 서버로 실행하고, 
브라우저에서 기본 페이지가 제대로 로드되는지 확인해줘.
스크린샷과 함께 보고서를 만들어줘."
```

### 전략 2: 병렬 실행 (Parallel Execution)

독립적인 작업을 동시에 실행하여 속도를 높입니다.

**시나리오**: API와 UI를 동시에 개발

**터미널 1 (Claude Code - 백엔드):**
```bash
cd my-project/backend
claude --dangerously-skip-permissions

/gsd:plan-phase 3  # API 구현
/gsd:execute-plan
```

**Antigravity IDE (프론트엔드):**
```
프로젝트 frontend/ 디렉토리 열기

"Phase 4의 UI 컴포넌트를 구현해줘.
API는 아직 준비 안 됐으니 Mock 데이터 사용.
완료되면 Storybook으로 컴포넌트 갤러리 만들어줘."
```

**통합 지점:**
- 백엔드 API 완료 → Claude Code가 Slack MCP로 알림
- 프론트엔드가 Mock에서 실제 API로 전환
- Antigravity가 E2E 테스트로 통합 검증

### 전략 3: MCP 기반 통합 (MCP-Driven Integration)

MCP 서버를 중앙 허브로 활용하여 도구 간 데이터 공유.

**커스텀 GSD MCP 서버 생성:**

`.claude/mcp-servers/gsd-server.js`:
```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import fs from 'fs';
import path from 'path';

const server = new Server(
  { name: "gsd-server", version: "1.0.0" },
  { capabilities: { resources: {}, tools: {} } }
);

// 리소스: GSD 문서들을 읽을 수 있게
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "gsd://project",
        name: "Project Specification",
        description: "PROJECT.md content",
        mimeType: "text/markdown"
      },
      {
        uri: "gsd://roadmap",
        name: "Project Roadmap",
        description: "ROADMAP.md content",
        mimeType: "text/markdown"
      },
      {
        uri: "gsd://state",
        name: "Current State",
        description: "STATE.md content",
        mimeType: "text/markdown"
      }
    ]
  };
});

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const { uri } = request.params;
  const filePath = {
    "gsd://project": ".planning/PROJECT.md",
    "gsd://roadmap": ".planning/ROADMAP.md",
    "gsd://state": ".planning/STATE.md"
  }[uri];

  if (!filePath) {
    throw new Error(`Unknown resource: ${uri}`);
  }

  const content = fs.readFileSync(filePath, 'utf-8');
  return {
    contents: [{
      uri,
      mimeType: "text/markdown",
      text: content
    }]
  };
});

// 도구: GSD 작업 조회 및 업데이트
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "get_current_task":
      const state = fs.readFileSync('.planning/STATE.md', 'utf-8');
      const currentTask = extractCurrentTask(state);
      return {
        content: [{
          type: "text",
          text: JSON.stringify(currentTask, null, 2)
        }]
      };

    case "complete_task":
      const { taskId, summary } = args;
      updateState(taskId, 'completed');
      createSummary(taskId, summary);
      return {
        content: [{
          type: "text",
          text: `Task ${taskId} marked as completed`
        }]
      };

    case "add_issue":
      const { title, priority, description } = args;
      addIssue(title, priority, description);
      return {
        content: [{
          type: "text",
          text: `Issue added: ${title}`
        }]
      };

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

function extractCurrentTask(stateContent) {
  // STATE.md 파싱 로직
  const match = stateContent.match(/🔄\s+Task\s+(\d+-\d+):\s+(.+)/);
  if (!match) return null;
  
  return {
    id: match[1],
    name: match[2],
    status: 'in_progress'
  };
}

function updateState(taskId, status) {
  let state = fs.readFileSync('.planning/STATE.md', 'utf-8');
  const emoji = status === 'completed' ? '✅' : '🔄';
  state = state.replace(
    new RegExp(`🔄\\s+Task\\s+${taskId}:`),
    `${emoji} Task ${taskId}:`
  );
  fs.writeFileSync('.planning/STATE.md', state);
}

function createSummary(taskId, summary) {
  const date = new Date().toISOString().split('T')[0];
  const summaryPath = `.planning/summaries/${taskId}-${date}.md`;
  fs.writeFileSync(summaryPath, summary);
}

function addIssue(title, priority, description) {
  let issues = fs.readFileSync('.planning/ISSUES.md', 'utf-8');
  const prioritySection = priority === 'high' ? '## 높은 우선순위' : 
                          priority === 'medium' ? '## 중간 우선순위' : 
                          '## 낮은 우선순위';
  
  const issueNum = (issues.match(/- \[.\] #(\d+):/g) || []).length + 1;
  const newIssue = `- [ ] #${issueNum}: ${title}\n  ${description}`;
  
  issues = issues.replace(
    prioritySection,
    `${prioritySection}\n${newIssue}`
  );
  
  fs.writeFileSync('.planning/ISSUES.md', issues);
}

// 서버 시작
const transport = new StdioServerTransport();
await server.connect(transport);
```

**Claude Code에서 GSD MCP 서버 사용:**

```bash
# MCP 서버 등록
claude mcp add gsd -- node .claude/mcp-servers/gsd-server.js

# 이제 Claude Code에서 GSD 정보를 직접 조회 가능
"@gsd:state 를 보고 현재 진행 중인 작업이 뭔지 알려줘"

"현재 작업을 완료하고, 다음 요약으로 기록해줘:
[요약 내용]"

"높은 우선순위 이슈로 'API 응답 속도 느림' 추가해줘"
```

**Antigravity에서 동일한 MCP 서버 사용:**

Antigravity 설정 파일에 GSD MCP 서버 추가:
```json
{
  "mcpServers": {
    "gsd": {
      "command": "node",
      "args": [".claude/mcp-servers/gsd-server.js"]
    }
  }
}
```

이제 Antigravity 에이전트도 GSD 상태를 인식하고 작업을 조율할 수 있습니다.

### 전략 4: 브라우저 검증 파이프라인 (Antigravity-Powered Verification)

Claude Code가 코드를 작성하면, Antigravity가 자동으로 브라우저에서 검증합니다.

**자동화 스크립트:**

`.claude/scripts/verify-with-antigravity.sh`:
```bash
#!/bin/bash

# Claude Code가 코드 작성 완료 후 이 스크립트 호출

TASK_ID=$1
VERIFICATION_SPEC=".antigravity/tasks/${TASK_ID}-verify.json"

# Antigravity 검증 작업 생성
cat > $VERIFICATION_SPEC <<EOF
{
  "taskId": "${TASK_ID}",
  "type": "browser-verification",
  "steps": [
    {
      "action": "navigate",
      "url": "http://localhost:3000"
    },
    {
      "action": "screenshot",
      "selector": "body",
      "filename": "homepage-${TASK_ID}.png"
    },
    {
      "action": "click",
      "selector": "#login-button"
    },
    {
      "action": "waitForNavigation"
    },
    {
      "action": "screenshot",
      "filename": "login-page-${TASK_ID}.png"
    },
    {
      "action": "type",
      "selector": "#email",
      "value": "test@example.com"
    },
    {
      "action": "type",
      "selector": "#password",
      "value": "password123"
    },
    {
      "action": "click",
      "selector": "#submit"
    },
    {
      "action": "waitForNavigation"
    },
    {
      "action": "screenshot",
      "filename": "dashboard-${TASK_ID}.png"
    },
    {
      "action": "assertUrl",
      "expected": "http://localhost:3000/dashboard"
    }
  ],
  "reportPath": ".antigravity/artifacts/${TASK_ID}-report.md"
}
EOF

echo "Verification task created: $VERIFICATION_SPEC"
echo "Antigravity will process this task automatically."

# Antigravity에 알림 (선택적)
# curl -X POST http://localhost:8080/api/tasks -d @$VERIFICATION_SPEC
```

**Antigravity 에이전트 (자동 실행):**

Antigravity 설정에서 `.antigravity/tasks/` 디렉토리를 모니터링하도록 설정:

```javascript
// .antigravity/agent-config.js
export default {
  watchers: [
    {
      path: ".antigravity/tasks/",
      pattern: "*-verify.json",
      handler: "browser-verification-agent"
    }
  ]
}
```

**결과:**
1. Claude Code가 Task 3-2 (로그인 페이지) 완료
2. 자동으로 `verify-with-antigravity.sh 3-2` 실행
3. Antigravity 에이전트가 검증 작업 감지
4. 브라우저 자동화 실행
5. 스크린샷 및 보고서 생성
6. 문제 발견 시 자동으로 ISSUES.md에 추가

---

## MCP를 활용한 고급 워크플로우

Model Context Protocol은 세 도구를 통합하는 핵심 기술입니다. 고급 활용법을 살펴보겠습니다.

### MCP 아키텍처 개요

MCP는 AI 애플리케이션이 외부 데이터 소스 및 도구와 연결하는 방식을 표준화하는 프로토콜로, 개발자들이 각 데이터 소스마다 별도의 커넥터를 유지 관리하는 대신 표준 프로토콜에 대해 구축할 수 있게 합니다.

```
┌──────────────────────────────────────────────┐
│           Host Application                    │
│  (Claude Code / Antigravity)                 │
│  ┌────────────────────────────────────┐     │
│  │         MCP Client                  │     │
│  │  - Request Handling                 │     │
│  │  - Response Processing              │     │
│  │  - Authentication                   │     │
│  └────────────────────────────────────┘     │
└──────────────────────────────────────────────┘
                    ↕ (JSON-RPC 2.0)
┌──────────────────────────────────────────────┐
│           MCP Server                          │
│  ┌────────────────────────────────────┐     │
│  │  Resources   Tools   Prompts       │     │
│  └────────────────────────────────────┘     │
│  ┌────────────────────────────────────┐     │
│  │  Business Logic                     │     │
│  └────────────────────────────────────┘     │
└──────────────────────────────────────────────┘
                    ↕
┌──────────────────────────────────────────────┐
│      External Systems                         │
│  (GitHub, Slack, Database, APIs...)          │
└──────────────────────────────────────────────┘
```

### 커스텀 MCP 서버 구축

**시나리오**: Notion을 프로젝트 문서 허브로 사용

Notion MCP 서버를 만들어 GSD 문서를 Notion과 동기화합니다.

`.claude/mcp-servers/notion-sync.js`:
```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { Client } from "@notionhq/client";
import fs from 'fs';

const notion = new Client({ auth: process.env.NOTION_API_KEY });
const DATABASE_ID = process.env.NOTION_DATABASE_ID;

const server = new Server(
  { name: "notion-sync", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "sync_project_to_notion":
      return await syncProjectToNotion();
    
    case "sync_roadmap_to_notion":
      return await syncRoadmapToNotion();
    
    case "create_daily_log_page":
      return await createDailyLogPage(args.date);
    
    case "update_task_status":
      return await updateTaskStatus(args.taskId, args.status);
    
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

async function syncProjectToNotion() {
  const projectContent = fs.readFileSync('.planning/PROJECT.md', 'utf-8');
  
  // PROJECT.md 파싱
  const title = projectContent.match(/# (.+)/)[1];
  const sections = parseMarkdownSections(projectContent);

  // Notion 페이지 생성 또는 업데이트
  const page = await notion.pages.create({
    parent: { database_id: DATABASE_ID },
    properties: {
      "Name": {
        title: [{ text: { content: title } }]
      },
      "Type": {
        select: { name: "Project Spec" }
      },
      "Last Updated": {
        date: { start: new Date().toISOString().split('T')[0] }
      }
    },
    children: convertToNotionBlocks(sections)
  });

  return {
    content: [{
      type: "text",
      text: `Project synced to Notion: ${page.url}`
    }]
  };
}

async function createDailyLogPage(date) {
  const logPath = `.planning/daily-logs/${date}.md`;
  if (!fs.existsSync(logPath)) {
    return {
      content: [{
        type: "text",
        text: `No daily log found for ${date}`
      }]
    };
  }

  const logContent = fs.readFileSync(logPath, 'utf-8');
  const sections = parseMarkdownSections(logContent);

  const page = await notion.pages.create({
    parent: { database_id: DATABASE_ID },
    properties: {
      "Name": {
        title: [{ text: { content: `Daily Log - ${date}` } }]
      },
      "Type": {
        select: { name: "Daily Log" }
      },
      "Date": {
        date: { start: date }
      }
    },
    children: convertToNotionBlocks(sections)
  });

  return {
    content: [{
      type: "text",
      text: `Daily log created in Notion: ${page.url}`
    }]
  };
}

function parseMarkdownSections(markdown) {
  // Markdown을 섹션별로 파싱
  const lines = markdown.split('\n');
  const sections = [];
  let currentSection = null;

  for (const line of lines) {
    if (line.startsWith('##')) {
      if (currentSection) {
        sections.push(currentSection);
      }
      currentSection = {
        heading: line.replace(/^#+ /, ''),
        content: []
      };
    } else if (currentSection) {
      currentSection.content.push(line);
    }
  }

  if (currentSection) {
    sections.push(currentSection);
  }

  return sections;
}

function convertToNotionBlocks(sections) {
  const blocks = [];

  for (const section of sections) {
    // 헤딩
    blocks.push({
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{ text: { content: section.heading } }]
      }
    });

    // 내용
    const content = section.content.join('\n').trim();
    if (content) {
      // 간단한 Markdown → Notion 변환
      const paragraphs = content.split('\n\n');
      for (const para of paragraphs) {
        if (para.startsWith('- ')) {
          // 리스트
          const items = para.split('\n');
          for (const item of items) {
            blocks.push({
              object: "block",
              type: "bulleted_list_item",
              bulleted_list_item: {
                rich_text: [{
                  text: { content: item.replace(/^- /, '') }
                }]
              }
            });
          }
        } else {
          // 일반 단락
          blocks.push({
            object: "block",
            type: "paragraph",
            paragraph: {
              rich_text: [{ text: { content: para } }]
            }
          });
        }
      }
    }
  }

  return blocks;
}

const transport = new StdioServerTransport();
await server.connect(transport);
```

**사용법:**

```bash
# MCP 서버 등록
export NOTION_API_KEY="your-notion-integration-key"
export NOTION_DATABASE_ID="your-database-id"
claude mcp add notion -- node .claude/mcp-servers/notion-sync.js

# Claude Code에서 사용
"프로젝트 스펙을 Notion에 동기화해줘"
# → sync_project_to_notion 호출

"오늘 Daily Log를 Notion에 게시해줘"
# → create_daily_log_page(today) 호출
```

### MCP 서버 체이닝

여러 MCP 서버를 조합하여 복잡한 워크플로우를 구현합니다.

**시나리오**: GitHub 이슈 → Slack 알림 → Notion 문서화

```
[GitHub MCP]
    ↓ (이슈 생성 감지)
[Custom Orchestrator MCP]
    ↓ (Slack 알림 트리거)
[Slack MCP]
    ↓ (팀 채널에 포스팅)
[Notion MCP]
    ↓ (이슈 문서화)
[완료]
```

**Orchestrator MCP 서버:**

`.claude/mcp-servers/issue-orchestrator.js`:
```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { execSync } from 'child_process';

const server = new Server(
  { name: "issue-orchestrator", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "handle_new_issue") {
    const { issueNumber, title, body, priority } = args;

    // 1. Slack에 알림
    await callMCP('slack', 'send_message', {
      channel: '#dev-alerts',
      text: `🚨 New ${priority} priority issue: #${issueNumber} - ${title}`,
      blocks: [
        {
          type: "section",
          text: {
            type: "mrkdwn",
            text: `*Issue #${issueNumber}*: ${title}\n${body.substring(0, 200)}...`
          }
        },
        {
          type: "actions",
          elements: [
            {
              type: "button",
              text: { type: "plain_text", text: "View on GitHub" },
              url: `https://github.com/owner/repo/issues/${issueNumber}`
            }
          ]
        }
      ]
    });

    // 2. Notion에 문서화
    await callMCP('notion', 'create_issue_page', {
      issueNumber,
      title,
      body,
      priority,
      status: 'Open'
    });

    // 3. GSD ISSUES.md에 추가
    await callMCP('gsd', 'add_issue', {
      title: `GitHub #${issueNumber}: ${title}`,
      priority: priority.toLowerCase(),
      description: `See: https://github.com/owner/repo/issues/${issueNumber}`
    });

    return {
      content: [{
        type: "text",
        text: `Issue #${issueNumber} processed: Slack notified, Notion updated, GSD tracked`
      }]
    };
  }
});

async function callMCP(serverName, tool, args) {
  // 다른 MCP 서버 호출 (실제로는 Claude를 통해 간접 호출)
  const command = `echo '${JSON.stringify({ tool, args })}' | claude mcp call ${serverName}`;
  const result = execSync(command).toString();
  return JSON.parse(result);
}

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 양방향 MCP 통신

Claude Code와 Antigravity가 서로 통신하도록 MCP 서버를 설정합니다.

**Bridge MCP 서버:**

`.claude/mcp-servers/claude-antigravity-bridge.js`:
```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import fs from 'fs';
import path from 'path';

const BRIDGE_DIR = '.bridge';
if (!fs.existsSync(BRIDGE_DIR)) {
  fs.mkdirSync(BRIDGE_DIR);
}

const server = new Server(
  { name: "claude-antigravity-bridge", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "send_to_antigravity":
      return sendToAntigravity(args.message, args.taskType);
    
    case "receive_from_antigravity":
      return receiveFromAntigravity();
    
    case "request_verification":
      return requestVerification(args.component, args.criteria);
    
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

function sendToAntigravity(message, taskType = 'general') {
  const messageFile = path.join(BRIDGE_DIR, `to-antigravity-${Date.now()}.json`);
  fs.writeFileSync(messageFile, JSON.stringify({
    from: 'claude-code',
    type: taskType,
    message,
    timestamp: new Date().toISOString()
  }));

  return {
    content: [{
      type: "text",
      text: `Message sent to Antigravity: ${messageFile}`
    }]
  };
}

function receiveFromAntigravity() {
  const files = fs.readdirSync(BRIDGE_DIR)
    .filter(f => f.startsWith('from-antigravity-'))
    .sort()
    .reverse();

  if (files.length === 0) {
    return {
      content: [{
        type: "text",
        text: "No messages from Antigravity"
      }]
    };
  }

  const latestFile = path.join(BRIDGE_DIR, files[0]);
  const message = JSON.parse(fs.readFileSync(latestFile, 'utf-8'));
  
  // 읽은 메시지 삭제
  fs.unlinkSync(latestFile);

  return {
    content: [{
      type: "text",
      text: JSON.stringify(message, null, 2)
    }]
  };
}

function requestVerification(component, criteria) {
  const taskFile = path.join(BRIDGE_DIR, `verification-request-${Date.now()}.json`);
  fs.writeFileSync(taskFile, JSON.stringify({
    from: 'claude-code',
    type: 'verification',
    component,
    criteria,
    timestamp: new Date().toISOString(),
    status: 'pending'
  }));

  return {
    content: [{
      type: "text",
      text: `Verification requested for ${component}. Antigravity will process this.`
    }]
  };
}

const transport = new StdioServerTransport();
await server.connect(transport);
```

**사용 예시:**

Claude Code:
```
"LoginButton 컴포넌트를 구현했어. 
Antigravity에게 다음을 검증해달라고 요청해줘:
1. 버튼이 중앙에 정렬되는지
2. 호버 시 색상이 변하는지
3. 클릭 시 로그인 모달이 뜨는지"

# → request_verification 호출
```

Antigravity (자동 감지):
```javascript
// Antigravity agent watches .bridge/ directory
const fs = require('fs').promises;
const watcher = require('chokidar');

watcher.watch('.bridge/verification-request-*.json')
  .on('add', async (filePath) => {
    const task = JSON.parse(await fs.readFile(filePath, 'utf-8'));
    
    // 브라우저 자동화 실행
    const result = await performVerification(task);
    
    // 결과를 Claude Code로 전송
    const responseFile = filePath.replace(
      'verification-request-',
      'from-antigravity-'
    );
    await fs.writeFile(responseFile, JSON.stringify({
      from: 'antigravity',
      originalTask: task,
      result,
      artifacts: [
        `.antigravity/artifacts/screenshot-${Date.now()}.png`,
        `.antigravity/artifacts/video-${Date.now()}.webm`
      ],
      timestamp: new Date().toISOString()
    }));
    
    // 원본 파일 삭제
    await fs.unlink(filePath);
  });
```

Claude Code (결과 확인):
```
"Antigravity로부터 검증 결과 받아왔어?"

# → receive_from_antigravity 호출
# → Antigravity의 보고서 출력
```

---

## Daily Work Log 자동화 시스템

GSD, Claude Code, Antigravity를 활용한 완전 자동화 Daily Work Log 시스템을 구축합니다.

### 자동 수집 파이프라인

**1. 실시간 활동 추적**

`.claude/scripts/activity-tracker.js`:
```javascript
#!/usr/bin/env node

import fs from 'fs';
import { execSync } from 'child_process';
import chokidar from 'chokidar';

const ACTIVITY_LOG = '.planning/activity-log.jsonl';

class ActivityTracker {
  constructor() {
    this.currentSession = {
      startTime: new Date().toISOString(),
      activities: []
    };
    this.setupWatchers();
  }

  setupWatchers() {
    // Git 커밋 감지
    this.watchGitCommits();
    
    // 파일 변경 감지
    this.watchFileChanges();
    
    // GSD 문서 업데이트 감지
    this.watchGSDDocuments();
    
    // 주기적으로 로그 저장
    setInterval(() => this.saveLog(), 60000); // 1분마다
  }

  watchGitCommits() {
    let lastCommit = this.getLastCommitHash();
    
    setInterval(() => {
      const currentCommit = this.getLastCommitHash();
      if (currentCommit !== lastCommit) {
        const commitInfo = this.getCommitInfo(currentCommit);
        this.logActivity({
          type: 'git_commit',
          commit: currentCommit,
          message: commitInfo.message,
          files: commitInfo.files,
          additions: commitInfo.additions,
          deletions: commitInfo.deletions
        });
        lastCommit = currentCommit;
      }
    }, 5000); // 5초마다 체크
  }

  watchFileChanges() {
    const watcher = chokidar.watch([
      'src/**/*',
      'app/**/*',
      'pages/**/*',
      'components/**/*'
    ], {
      ignored: /(^|[\/\\])\../,
      persistent: true
    });

    let changeBuffer = [];
    let bufferTimer = null;

    watcher.on('change', (path) => {
      changeBuffer.push(path);
      
      // 변경이 1초 동안 없으면 로그 기록
      clearTimeout(bufferTimer);
      bufferTimer = setTimeout(() => {
        this.logActivity({
          type: 'file_changes',
          files: [...new Set(changeBuffer)],
          count: changeBuffer.length
        });
        changeBuffer = [];
      }, 1000);
    });
  }

  watchGSDDocuments() {
    const watcher = chokidar.watch('.planning/*.md');
    
    watcher.on('change', (path) => {
      this.logActivity({
        type: 'gsd_update',
        document: path.split('/').pop()
      });
    });
  }

  logActivity(activity) {
    activity.timestamp = new Date().toISOString();
    this.currentSession.activities.push(activity);
    
    // JSONL 형식으로 즉시 저장 (세션이 중단되어도 데이터 보존)
    fs.appendFileSync(
      ACTIVITY_LOG,
      JSON.stringify(activity) + '\n'
    );
  }

  saveLog() {
    const logFile = `.planning/sessions/${this.currentSession.startTime}.json`;
    fs.writeFileSync(logFile, JSON.stringify(this.currentSession, null, 2));
  }

  getLastCommitHash() {
    try {
      return execSync('git rev-parse HEAD').toString().trim();
    } catch (e) {
      return null;
    }
  }

  getCommitInfo(hash) {
    const message = execSync(`git log -1 --format=%B ${hash}`).toString().trim();
    const stats = execSync(`git show --stat --format='' ${hash}`).toString();
    
    const files = [];
    const lines = stats.split('\n').filter(l => l.trim());
    for (const line of lines) {
      const match = line.match(/(.+?)\s+\|\s+(\d+)\s+([+\-]+)/);
      if (match) {
        files.push(match[1].trim());
      }
    }

    const diffStats = execSync(`git show --shortstat ${hash}`).toString();
    const addMatch = diffStats.match(/(\d+) insertions/);
    const delMatch = diffStats.match(/(\d+) deletions/);

    return {
      message,
      files,
      additions: addMatch ? parseInt(addMatch[1]) : 0,
      deletions: delMatch ? parseInt(delMatch[1]) : 0
    };
  }
}

// 백그라운드에서 실행
const tracker = new ActivityTracker();

process.on('SIGINT', () => {
  tracker.saveLog();
  console.log('Activity tracker stopped');
  process.exit();
});

console.log('Activity tracker started');
```

**자동 시작 설정:**

```bash
# .claude/start-tracking.sh
#!/bin/bash

# 이미 실행 중인지 확인
if [ -f .planning/tracker.pid ]; then
  PID=$(cat .planning/tracker.pid)
  if ps -p $PID > /dev/null; then
    echo "Activity tracker already running (PID: $PID)"
    exit 0
  fi
fi

# 백그라운드로 실행
nohup node .claude/scripts/activity-tracker.js > .planning/tracker.log 2>&1 &
echo $! > .planning/tracker.pid

echo "Activity tracker started (PID: $(cat .planning/tracker.pid))"
```

**2. AI 기반 요약 생성**

활동 로그를 분석하여 Daily Work Log를 자동 생성합니다.

`.claude/scripts/generate-daily-summary.js`:
```javascript
#!/usr/bin/env node

import fs from 'fs';
import { execSync } from 'child_process';

async function generateDailySummary(date = new Date().toISOString().split('T')[0]) {
  const activityLog = loadActivityLog(date);
  const gitHistory = loadGitHistory(date);
  const gsdState = loadGSDState();

  // Claude에게 요약 요청
  const prompt = `
다음은 ${date}의 개발 활동 로그입니다. 이를 분석하여 Daily Work Log를 작성해주세요.

## 활동 로그
${JSON.stringify(activityLog, null, 2)}

## Git 히스토리
${gitHistory.map(c => `- ${c.hash.substring(0, 7)}: ${c.message}`).join('\n')}

## GSD 상태
${gsdState}

## 요청사항
다음 형식으로 Daily Work Log를 작성해주세요:

# Daily Work Log - ${date}

## 📊 요약
- 완료한 작업: X개
- 생성한 커밋: ${gitHistory.length}개
- 작성한 코드: +XXX -XXX 라인
- 작업 시간: X시간 Y분

## ✅ 완료한 작업
[활동 로그를 분석하여 작업 목록 생성]

## 🔧 기술적 결정
[커밋 메시지와 파일 변경을 분석하여 주요 기술적 결정 추출]

## 🐛 발견한 이슈
[Git 커밋 중 fix, bug 관련 항목 추출]

## 📚 학습한 내용
[새로 사용한 라이브러리, 패턴, 기술]

## ⏭️ 다음 작업일 계획
[GSD STATE.md의 다음 작업]

## 💭 회고
[하루 활동의 전체적인 흐름과 패턴 분석]
  `;

  // Claude 호출
  const summary = await callClaude(prompt);
  
  // 저장
  const outputPath = `.planning/daily-logs/${date}.md`;
  fs.writeFileSync(outputPath, summary);
  
  console.log(`Daily summary generated: ${outputPath}`);
  return outputPath;
}

function loadActivityLog(date) {
  const logFile = '.planning/activity-log.jsonl';
  if (!fs.existsSync(logFile)) {
    return [];
  }

  const lines = fs.readFileSync(logFile, 'utf-8').split('\n').filter(Boolean);
  return lines
    .map(line => JSON.parse(line))
    .filter(activity => activity.timestamp.startsWith(date));
}

function loadGitHistory(date) {
  const since = `${date}T00:00:00`;
  const until = `${date}T23:59:59`;
  
  const log = execSync(
    `git log --since="${since}" --until="${until}" --format="%H|%s|%an|%ad" --date=iso`
  ).toString();

  return log.split('\n').filter(Boolean).map(line => {
    const [hash, message, author, date] = line.split('|');
    return { hash, message, author, date };
  });
}

function loadGSDState() {
  if (!fs.existsSync('.planning/STATE.md')) {
    return "No GSD state available";
  }
  return fs.readFileSync('.planning/STATE.md', 'utf-8');
}

async function callClaude(prompt) {
  // Claude API 호출 (실제로는 MCP나 API 사용)
  const tempFile = '/tmp/claude-prompt.txt';
  fs.writeFileSync(tempFile, prompt);
  
  const result = execSync(`claude --prompt-file ${tempFile}`).toString();
  fs.unlinkSync(tempFile);
  
  return result;
}

// 실행
const date = process.argv[2] || new Date().toISOString().split('T')[0];
generateDailySummary(date);
```

**자동 실행 설정:**

```bash
# crontab
# 매일 오후 6시에 Daily Summary 생성
0 18 * * * cd /path/to/project && node .claude/scripts/generate-daily-summary.js
```

### 시각화 대시보드

**실시간 활동 모니터**

`.claude/scripts/activity-dashboard.html`:
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Real-time Activity Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      margin: 0;
      padding: 20px;
      background: #0d1117;
      color: #c9d1d9;
    }
    .container {
      max-width: 1400px;
      margin: 0 auto;
    }
    .header {
      background: linear-gradient(135deg, #1f6feb 0%, #58a6ff 100%);
      padding: 30px;
      border-radius: 12px;
      margin-bottom: 20px;
    }
    .stats-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 15px;
      margin-bottom: 30px;
    }
    .stat-card {
      background: #161b22;
      padding: 20px;
      border-radius: 8px;
      border: 1px solid #30363d;
    }
    .stat-value {
      font-size: 32px;
      font-weight: bold;
      color: #58a6ff;
    }
    .stat-label {
      color: #8b949e;
      margin-top: 5px;
      font-size: 14px;
    }
    .chart-container {
      background: #161b22;
      padding: 20px;
      border-radius: 8px;
      border: 1px solid #30363d;
      margin-bottom: 20px;
    }
    .activity-feed {
      background: #161b22;
      padding: 20px;
      border-radius: 8px;
      border: 1px solid #30363d;
      max-height: 400px;
      overflow-y: auto;
    }
    .activity-item {
      padding: 10px;
      border-bottom: 1px solid #30363d;
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .activity-icon {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 16px;
    }
    .activity-icon.commit { background: #1f6feb; }
    .activity-icon.file { background: #f78166; }
    .activity-icon.gsd { background: #56d364; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>🚀 Real-time Activity Dashboard</h1>
      <p>Monitoring: GSD + Claude Code + Antigravity</p>
      <p id="lastUpdate">Last update: <span>-</span></p>
    </div>

    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-value" id="totalCommits">0</div>
        <div class="stat-label">Total Commits Today</div>
      </div>
      <div class="stat-card">
        <div class="stat-value" id="filesChanged">0</div>
        <div class="stat-label">Files Changed</div>
      </div>
      <div class="stat-card">
        <div class="stat-value" id="linesAdded">+0</div>
        <div class="stat-label">Lines Added</div>
      </div>
      <div class="stat-card">
        <div class="stat-value" id="activeTime">0h 0m</div>
        <div class="stat-label">Active Time</div>
      </div>
    </div>

    <div class="chart-container">
      <canvas id="activityChart"></canvas>
    </div>

    <div class="chart-container">
      <canvas id="filesChart"></canvas>
    </div>

    <div class="activity-feed" id="activityFeed">
      <h3>Recent Activity</h3>
      <div id="activities"></div>
    </div>
  </div>

  <script>
    let activityChart, filesChart;

    async function loadActivityData() {
      const response = await fetch('.planning/activity-log.jsonl');
      const text = await response.text();
      const lines = text.split('\n').filter(Boolean);
      const today = new Date().toISOString().split('T')[0];
      
      return lines
        .map(line => JSON.parse(line))
        .filter(activity => activity.timestamp.startsWith(today));
    }

    async function updateDashboard() {
      const activities = await loadActivityData();
      
      // 통계 업데이트
      const commits = activities.filter(a => a.type === 'git_commit');
      const fileChanges = activities.filter(a => a.type === 'file_changes');
      
      document.getElementById('totalCommits').textContent = commits.length;
      
      const uniqueFiles = new Set();
      fileChanges.forEach(fc => fc.files.forEach(f => uniqueFiles.add(f)));
      document.getElementById('filesChanged').textContent = uniqueFiles.size;
      
      const totalAdditions = commits.reduce((sum, c) => sum + (c.additions || 0), 0);
      document.getElementById('linesAdded').textContent = `+${totalAdditions}`;
      
      // 활동 시간 계산 (첫 활동부터 마지막 활동까지)
      if (activities.length > 0) {
        const firstTime = new Date(activities[0].timestamp);
        const lastTime = new Date(activities[activities.length - 1].timestamp);
        const diffMs = lastTime - firstTime;
        const hours = Math.floor(diffMs / 3600000);
        const minutes = Math.floor((diffMs % 3600000) / 60000);
        document.getElementById('activeTime').textContent = `${hours}h ${minutes}m`;
      }
      
      // 차트 업데이트
      updateActivityChart(activities);
      updateFilesChart(fileChanges);
      
      // 활동 피드 업데이트
      updateActivityFeed(activities.slice(-10).reverse());
      
      document.querySelector('#lastUpdate span').textContent = 
        new Date().toLocaleTimeString();
    }

    function updateActivityChart(activities) {
      // 시간대별 활동 집계
      const hourlyActivity = new Array(24).fill(0);
      activities.forEach(a => {
        const hour = new Date(a.timestamp).getHours();
        hourlyActivity[hour]++;
      });

      if (activityChart) {
        activityChart.data.datasets[0].data = hourlyActivity;
        activityChart.update();
      } else {
        activityChart = new Chart(document.getElementById('activityChart'), {
          type: 'bar',
          data: {
            labels: Array.from({length: 24}, (_, i) => `${i}:00`),
            datasets: [{
              label: 'Activities per Hour',
              data: hourlyActivity,
              backgroundColor: 'rgba(88, 166, 255, 0.5)',
              borderColor: 'rgba(88, 166, 255, 1)',
              borderWidth: 1
            }]
          },
          options: {
            responsive: true,
            plugins: {
              title: {
                display: true,
                text: 'Activity Distribution',
                color: '#c9d1d9'
              },
              legend: {
                labels: {
                  color: '#c9d1d9'
                }
              }
            },
            scales: {
              y: {
                beginAtZero: true,
                ticks: { color: '#8b949e' },
                grid: { color: '#30363d' }
              },
              x: {
                ticks: { color: '#8b949e' },
                grid: { color: '#30363d' }
              }
            }
          }
        });
      }
    }

    function updateFilesChart(fileChanges) {
      // 파일 타입별 변경 횟수
      const fileTypes = {};
      fileChanges.forEach(fc => {
        fc.files.forEach(file => {
          const ext = file.split('.').pop();
          fileTypes[ext] = (fileTypes[ext] || 0) + 1;
        });
      });

      const labels = Object.keys(fileTypes);
      const data = Object.values(fileTypes);

      if (filesChart) {
        filesChart.data.labels = labels;
        filesChart.data.datasets[0].data = data;
        filesChart.update();
      } else {
        filesChart = new Chart(document.getElementById('filesChart'), {
          type: 'doughnut',
          data: {
            labels,
            datasets: [{
              data,
              backgroundColor: [
                '#1f6feb',
                '#f78166',
                '#56d364',
                '#e3b341',
                '#db6d28',
                '#b083f0'
              ]
            }]
          },
          options: {
            responsive: true,
            plugins: {
              title: {
                display: true,
                text: 'Files Changed by Type',
                color: '#c9d1d9'
              },
              legend: {
                labels: {
                  color: '#c9d1d9'
                }
              }
            }
          }
        });
      }
    }

    function updateActivityFeed(activities) {
      const container = document.getElementById('activities');
      container.innerHTML = activities.map(activity => {
        const icon = activity.type === 'git_commit' ? '📝' :
                     activity.type === 'file_changes' ? '📄' :
                     activity.type === 'gsd_update' ? '📋' : '🔧';
        
        const iconClass = activity.type.replace('_', ' ');
        const time = new Date(activity.timestamp).toLocaleTimeString();
        
        let description = '';
        if (activity.type === 'git_commit') {
          description = activity.message;
        } else if (activity.type === 'file_changes') {
          description = `${activity.count} files changed`;
        } else if (activity.type === 'gsd_update') {
          description = `Updated ${activity.document}`;
        }

        return `
          <div class="activity-item">
            <div class="activity-icon ${iconClass}">${icon}</div>
            <div>
              <div style="font-weight: 500;">${description}</div>
              <div style="font-size: 12px; color: #8b949e;">${time}</div>
            </div>
          </div>
        `;
      }).join('');
    }

    // 초기 로드
    updateDashboard();
    
    // 5초마다 업데이트
    setInterval(updateDashboard, 5000);
  </script>
</body>
</html>
```

**로컬 서버 실행:**

```bash
# .claude/scripts/serve-dashboard.sh
#!/bin/bash

python3 -m http.server 8000 --directory .claude/scripts &
echo "Dashboard available at: http://localhost:8000/activity-dashboard.html"
```

### 팀 공유 자동화

**Slack 자동 포스팅:**

`.claude/commands/auto-share-daily-log.md`:
~~~markdown
# /auto-share-daily-log 설정

매일 자동으로 Daily Work Log를 생성하고 Slack에 공유합니다.

## 설정 단계

1. Slack Webhook URL 설정
```bash
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
```

2. 자동 실행 스크립트 생성
```bash
cat > .claude/scripts/auto-daily-share.sh <<'EOF'
#!/bin/bash

# Daily Summary 생성
node .claude/scripts/generate-daily-summary.js

# Slack으로 전송
TODAY=$(date +%Y-%m-%d)
LOG_FILE=".planning/daily-logs/${TODAY}.md"

if [ -f "$LOG_FILE" ]; then
  # Markdown을 Slack 형식으로 변환
  node .claude/scripts/slack-formatter.js "$LOG_FILE" | \
  curl -X POST -H 'Content-type: application/json' \
    --data @- "$SLACK_WEBHOOK_URL"
  
  echo "Daily log shared to Slack"
else
  echo "No daily log found for $TODAY"
fi
EOF

chmod +x .claude/scripts/auto-daily-share.sh
```

3. Cron 설정
```bash
# 매일 오후 6시 30분에 실행
30 18 * * * cd /path/to/project && .claude/scripts/auto-daily-share.sh
```
~~~

---

## 실전 시나리오 및 베스트 프랙티스

실제 프로젝트에서 세 도구를 통합하여 사용하는 구체적인 시나리오입니다.

### 시나리오 1: 신규 SaaS 제품 개발 (0 → MVP)

**프로젝트**: B2B SaaS 고객 관리 시스템

**기간**: 2주 스프린트

**목표**: 로그인, 대시보드, 고객 목록, 고객 상세 페이지 구현

#### Day 1-2: 기획 및 설정

**도구**: GSD (주도), Claude Code (보조)

```bash
# GSD로 프로젝트 초기화
/gsd:new-project

프로젝트 이름: CustomerHub
설명: B2B 고객 관리 SaaS 플랫폼
주요 기능:
- 고객 정보 관리
- 활동 타임라인
- 태그 및 세그먼트
- 팀 협업

기술 스택:
- Frontend: Next.js 15, React, TypeScript, Tailwind CSS
- Backend: Next.js API Routes, Prisma
- Database: PostgreSQL (Supabase)
- Auth: Supabase Auth
- Deployment: Vercel

/gsd:create-roadmap

# GSD가 자동으로 7개 Phase 생성:
# Phase 1: 프로젝트 설정 (1일)
# Phase 2: 인증 시스템 (2일)
# Phase 3: 고객 CRUD (2일)
# Phase 4: 대시보드 (2일)
# Phase 5: 활동 타임라인 (2일)
# Phase 6: 태그/세그먼트 (2일)
# Phase 7: 배포 (1일)

/gsd:plan-phase 1
/gsd:execute-plan
```

Claude Code가 보일러플레이트 생성:
- Next.js 프로젝트 초기화
- Prisma 설정 및 스키마 작성
- Tailwind 및 TypeScript 설정
- 기본 디렉토리 구조

#### Day 3-4: 인증 시스템

**도구**: Claude Code (코드), Antigravity (검증)

```bash
# Claude Code로 백엔드 구현
/gsd:plan-phase 2
/gsd:execute-plan

# Antigravity로 UI 및 E2E 테스트
```

Antigravity IDE에서:
```
"로그인 페이지를 디자인 파일을 기반으로 구현해줘.
[Figma 링크 첨부]

완료되면 다음을 브라우저로 검증:
1. 로그인 폼이 디자인과 일치하는지
2. 이메일 유효성 검증이 작동하는지
3. Google 로그인이 제대로 되는지
4. 로그인 후 대시보드로 리디렉션되는지

각 단계별 스크린샷과 함께 보고서 작성해줘."
```

Antigravity가 자동으로:
- Chromium 브라우저 실행
- 각 시나리오 테스트
- 스크린샷 캡처
- Artifact로 결과 보고서 생성

검증 완료 후 Claude Code로 버그 수정:
```
"Antigravity 보고서에 따르면 이메일 검증이 제대로 안 되고 있어.
lib/validation.ts 파일을 수정해서 RFC 5322 표준을 따르도록 해줘."
```

#### Day 5-6: 고객 CRUD

**도구**: Claude Code (백엔드), Antigravity (프론트엔드)

병렬 작업:

**터미널 1 (Claude Code):**
```bash
# API 엔드포인트 구현
/gsd:plan-phase 3

# GSD가 생성한 PLAN.md:
# Task 3-1: 고객 생성 API
# Task 3-2: 고객 조회 API (목록 및 상세)
# Task 3-3: 고객 수정/삭제 API
# Task 3-4: 단위 테스트 및 통합 테스트

/gsd:execute-plan
```

**Antigravity IDE:**
```
"고객 목록 및 상세 페이지 UI를 구현해줘.
API는 아직 준비 안 됐으니 다음 Mock 데이터 사용:

[Mock 데이터 JSON 제공]

컴포넌트는 shadcn/ui 사용하고,
테이블은 정렬, 필터링, 페이지네이션 지원해야 해.

완료되면 Storybook으로 모든 상태(로딩, 에러, 빈 상태)를
시각적으로 확인할 수 있게 해줘."
```

API 완료 후 통합:
```
"이제 API가 준비됐어. Mock 데이터 대신 실제 API 호출로 전환하고,
E2E 테스트를 작성해줘:
1. 고객 생성 플로우
2. 고객 목록 조회 및 상세 페이지 이동
3. 고객 정보 수정
4. 고객 삭제

각 플로우를 비디오로 녹화해서 보고서에 포함해줘."
```

#### Day 7-14: 나머지 기능 및 마무리

같은 패턴 반복:
- GSD로 Phase 계획
- Claude Code로 백엔드/비즈니스 로직
- Antigravity로 UI 및 E2E 테스트
- MCP로 GitHub 이슈, Slack 알림, Notion 문서화 자동 연동

#### 매일 오후 6시: Daily Work Log

자동화 시스템이:
1. 하루 활동 수집 (커밋, 파일 변경, GSD 업데이트)
2. Claude에게 요약 요청
3. Daily Work Log 생성
4. Slack에 공유
5. Notion에 문서화

#### 스프린트 종료: 회고 및 다음 계획

```bash
/gsd:complete-milestone

/gsd:weekly-summary

"이번 주 Daily Log를 모두 분석해서 다음을 추출해줘:
1. 가장 시간이 많이 걸린 작업과 이유
2. 반복적으로 발생한 문제 패턴
3. 다음 스프린트에서 개선할 점
4. 팀 프로세스 제안"
```

### 시나리오 2: 레거시 시스템 리팩토링

**프로젝트**: 5년된 Monolith → Microservices 마이그레이션

**기간**: 3개월

**목표**: 사용자 서비스 분리 (나머지는 단계적)

#### Week 1: 코드베이스 분석

**도구**: GSD (코드베이스 매핑), Claude Code (분석)

```bash
/gsd:map-codebase

# GSD가 생성:
# STACK.md - 현재 사용 중인 모든 기술
# ARCHITECTURE.md - Monolith 구조 분석
# CONVENTIONS.md - 기존 코딩 스타일
# CONCERNS.md - 기술 부채 목록
```

Claude Code로 심층 분석:
```
"ARCHITECTURE.md를 보고 users 관련 코드를 모두 추출해줘.
의존성 그래프를 만들고, 어떤 다른 모듈과 연결되어 있는지 분석해줘.

마이크로서비스로 분리할 때 끊어야 할 의존성과
API로 전환해야 할 함수 호출을 리스트업 해줘."
```

#### Week 2-4: 점진적 분리

**도구**: Claude Code (리팩토링), Antigravity (테스트)

Strangler Fig 패턴으로 점진적 전환:

```bash
# Phase 1: User Service 인터페이스 정의
/gsd:plan-phase 1

# Claude Code가 OpenAPI 스펙 생성

# Phase 2: User Service 구현 (기존 DB 공유)
/gsd:plan-phase 2
/gsd:execute-plan

# Antigravity로 기존 기능 회귀 테스트
```

Antigravity에서:
```
"기존 Monolith와 새 User Service가 동일하게 작동하는지 검증해줘.

1. 100개의 랜덤 사용자 데이터로 양쪽 모두 테스트
2. 응답 시간, 응답 형식, 데이터 정확성 비교
3. 차이가 있으면 상세히 보고

비디오 녹화와 함께 종합 보고서 작성해줘."
```

#### Week 5-8: 데이터 마이그레이션

**도구**: Claude Code (마이그레이션 스크립트), MCP (DB 접근)

```bash
# PostgreSQL MCP 서버 설정
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres

# Claude Code에서
"@postgres:legacy-db 와 @postgres:user-service-db 를 비교해서
스키마 차이를 분석하고, 마이그레이션 스크립트를 작성해줘.

Zero-downtime 마이그레이션을 위해:
1. Dual-write 페이즈 (양쪽 DB에 쓰기)
2. 백필 스크립트 (레거시 → 신규)
3. 검증 스크립트 (데이터 일치 확인)
4. Read 전환 (레거시 → 신규 읽기)
5. Write 전환 (레거시 중단)

각 단계별 롤백 계획도 포함해줘."
```

#### Week 9-12: 모니터링 및 최적화

**도구**: 모든 도구 통합

GSD로 성능 최적화 Phase 관리:
```bash
/gsd:new-phase "성능 최적화 및 모니터링"
```

Claude Code로 성능 분석:
```
"User Service의 모든 API 엔드포인트에 대해:
1. 응답 시간 분석 (P50, P95, P99)
2. 데이터베이스 쿼리 최적화 기회
3. 캐싱 전략 제안
4. 인덱스 추가 권장사항

구체적인 개선안을 제시해줘."
```

Antigravity로 부하 테스트:
```
"User Service에 대한 부하 테스트를 실행해줘:
- 동시 사용자: 100, 500, 1000, 5000
- 각 시나리오마다 10분간 실행
- 응답 시간, 에러율, 처리량 측정
- 병목 지점 식별

그래프와 함께 상세 보고서 작성해줘."
```

### 시나리오 3: 오픈소스 기여

**프로젝트**: 인기있는 React 라이브러리에 기여

**목표**: 새로운 기능 추가 + 문서화

#### 단계 1: 프로젝트 이해

```bash
cd react-awesome-library

# GSD로 코드베이스 분석
/gsd:map-codebase

# Claude Code로 기여 가이드 읽기
"CONTRIBUTING.md를 읽고 요약해줘. 
특히 다음 사항:
1. 코딩 스타일 규칙
2. 테스트 요구사항
3. PR 프로세스
4. 커밋 메시지 형식"
```

#### 단계 2: 이슈 선택 및 계획

```bash
# GitHub MCP로 이슈 조회
"@github:issues에서 'good first issue' 태그가 있는 것들을 찾아줘.
각 이슈의 복잡도를 평가하고 추천해줘."

# 선택한 이슈: #1234 "Add TypeScript support for useCustomHook"

/gsd:new-project "TypeScript support for useCustomHook"
/gsd:create-roadmap
```

#### 단계 3: 구현

Claude Code로 타입 정의 작성:
```
"src/hooks/useCustomHook.js를 분석하고
TypeScript 타입 정의를 작성해줘.

기존 프로젝트의 타이핑 패턴을 참고하고,
제네릭을 사용해서 유연하게 만들어줘.

타입 테스트도 함께 작성해줘."
```

#### 단계 4: 테스트

Antigravity로 예제 앱 테스트:
```
"examples/ 디렉토리의 모든 예제 앱에서
새로운 TypeScript 타입이 제대로 작동하는지 확인해줘.

각 예제에서:
1. 타입 에러가 없는지
2. IntelliSense가 제대로 작동하는지 (스크린샷)
3. 빌드가 성공하는지

보고서 작성해줘."
```

#### 단계 5: 문서화

```
"docs/ 디렉토리에 TypeScript 사용 가이드를 추가해줘.
기존 문서 스타일을 따르고, 코드 예제를 포함해서
초보자도 이해하기 쉽게 작성해줘."
```

#### 단계 6: PR 제출

```
"다음 변경사항으로 PR 설명을 작성해줘:
[변경 파일 목록]

프로젝트의 CONTRIBUTING.md 형식을 따르고,
이슈 #1234를 언급하며, 변경사항을 명확히 설명해줘.

스크린샷과 함께 Before/After를 보여주는
비교 섹션도 포함해줘."

# GitHub MCP로 PR 생성
"@github:pr-create로 PR을 만들어줘"
```

---

## 문제 해결 및 최적화

통합 환경에서 발생할 수 있는 문제와 해결 방법입니다.

### 문제 1: 도구 간 동기화 이슈

**증상**: Claude Code가 작업을 완료했는데 Antigravity가 인식하지 못함

**원인**: 파일 시스템 기반 통신의 지연

**해결**:

1. **즉시 알림 메커니즘**

```javascript
// .claude/scripts/notify-antigravity.js
#!/usr/bin/env node

import { WebSocket } from 'ws';

const ws = new WebSocket('ws://localhost:8080/antigravity-events');

ws.on('open', () => {
  ws.send(JSON.stringify({
    type: 'task_completed',
    taskId: process.argv[2],
    timestamp: new Date().toISOString()
  }));
  
  console.log('Notification sent to Antigravity');
  ws.close();
});
```

Claude Code 작업 완료 후:
```bash
node .claude/scripts/notify-antigravity.js "3-2"
```

2. **Webhook 기반 통신**

Antigravity 설정에 webhook 추가:
```json
{
  "webhooks": {
    "claude_code_completion": "http://localhost:8080/api/webhooks/claude-completed"
  }
}
```

### 문제 2: 컨텍스트 중복 및 충돌

**증상**: GSD의 PROJECT.md와 CLAUDE.md에 중복된 정보, 일관성 없음

**원인**: 두 파일의 역할이 명확하지 않음

**해결**:

**CLAUDE.md** - 코드 레벨 규칙만
```markdown
# 코딩 규칙

## 스타일
- ESLint: airbnb
- Prettier: 활성화
- 세미콜론: 필수

## 아키텍처
- 컴포넌트: Atomic Design
- 상태 관리: Zustand
- 라우팅: App Router

## 테스팅
- Jest + RTL
- 커버리지 최소 80%
```

**PROJECT.md** - 제품/비즈니스 요구사항
```markdown
# 프로젝트 목표

사용자가 고객 정보를 효율적으로 관리할 수 있는 SaaS

## 핵심 가치
- 간편성: 3클릭 이내 모든 작업 완료
- 속도: 페이지 로드 1초 이내
- 협업: 실시간 팀 공유
```

### 문제 3: MCP 서버 과부하

**증상**: MCP 서버 응답이 느려지거나 타임아웃

**원인**: 너무 많은 동시 요청

**해결**:

**요청 큐잉 및 제한**

`.claude/mcp-servers/rate-limited-wrapper.js`:
```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import pQueue from 'p-queue';

const queue = new pQueue({ concurrency: 3 }); // 최대 3개 동시 실행

const server = new Server(
  { name: "rate-limited-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  // 모든 요청을 큐에 넣어 순차 처리
  return await queue.add(() => handleRequest(request));
});

async function handleRequest(request) {
  // 실제 비즈니스 로직
  const { name, arguments: args } = request.params;
  
  // 시간이 오래 걸리는 작업
  await heavyOperation(args);
  
  return { content: [{ type: "text", text: "Done" }] };
}
```

### 문제 4: 리소스 고갈

**증상**: 장시간 실행 시 메모리 부족 또는 디스크 용량 초과

**원인**: 로그 파일 무한 증가, Antigravity artifacts 누적

**해결**:

**자동 정리 스크립트**

`.claude/scripts/cleanup.sh`:
```bash
#!/bin/bash

echo "🧹 Starting cleanup..."

# 30일 이상 된 activity log 삭제
find .planning/activity-log.jsonl -mtime +30 -delete

# 일주일 이상 된 Antigravity artifacts 압축
find .antigravity/artifacts/ -name "*.png" -mtime +7 -exec gzip {} \;
find .antigravity/artifacts/ -name "*.webm" -mtime +7 -exec gzip {} \;

# 90일 이상 된 artifacts 삭제
find .antigravity/artifacts/ -mtime +90 -delete

# 임시 파일 삭제
rm -rf .bridge/*.json
rm -rf /tmp/claude-*

# 디스크 사용량 보고
echo "📊 Disk usage after cleanup:"
du -sh .planning .antigravity .bridge

echo "✅ Cleanup completed"
```

Cron으로 매일 실행:
```bash
0 3 * * * cd /path/to/project && .claude/scripts/cleanup.sh
```

### 최적화 팁

#### 1. 선택적 도구 활용

모든 작업에 모든 도구를 사용할 필요는 없습니다.

```
간단한 버그 수정 → Claude Code만
UI 프로토타입 → Antigravity만
복잡한 기능 개발 → GSD + Claude Code + Antigravity
```

#### 2. 배치 작업

여러 작업을 한 번에 처리:

```bash
# 여러 Phase를 한 번에 계획
for i in {1..3}; do
  /gsd:plan-phase $i
done

# 배치 실행 (사용자 승인 1번만)
/gsd:execute-plans 1-3
```

#### 3. 캐싱 활용

자주 사용하는 MCP 응답 캐싱:

```javascript
// MCP 서버에 캐싱 레이어 추가
import NodeCache from 'node-cache';
const cache = new NodeCache({ stdTTL: 3600 }); // 1시간

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const cacheKey = JSON.stringify(request.params);
  
  // 캐시 확인
  const cached = cache.get(cacheKey);
  if (cached) {
    return cached;
  }
  
  // 실제 처리
  const result = await handleRequest(request);
  
  // 캐시 저장
  cache.set(cacheKey, result);
  
  return result;
});
```

#### 4. 병렬화 극대화

독립적인 작업은 동시 실행:

```bash
# 터미널 멀티플렉서 (tmux) 사용
tmux new-session -d -s dev

# 창 분할
tmux split-window -h
tmux split-window -v

# 각 창에서 다른 도구 실행
tmux send-keys -t 0 "cd backend && claude" C-m
tmux send-keys -t 1 "cd frontend && antigravity" C-m
tmux send-keys -t 2 ".claude/scripts/activity-tracker.js" C-m

# 세션 연결
tmux attach -t dev
```

---

## 결론

GSD, Claude Code, Google Antigravity를 통합하면 개발 생산성을 획기적으로 높일 수 있습니다. 각 도구의 강점을 이해하고 적재적소에 활용하는 것이 핵심입니다.

### 통합의 핵심 원칙

1. **명확한 역할 분담**: 각 도구의 강점에 맞게 작업 할당
2. **데이터 중심 통신**: 파일 시스템과 MCP를 통한 표준화된 통신
3. **자동화 우선**: 반복 작업은 스크립트와 MCP 서버로 자동화
4. **점진적 도입**: 한 번에 모든 것을 통합하려 하지 말고 단계적으로
5. **측정 및 개선**: Daily Work Log로 효과를 추적하고 지속적으로 개선

### 다음 단계

이 가이드를 읽고 나서:

1. **작은 프로젝트로 시작**: 개인 프로젝트나 사이드 프로젝트에서 먼저 시도
2. **하나씩 추가**: GSD → Claude Code → Antigravity → MCP 순으로 점진적 도입
3. **팀과 공유**: Daily Work Log를 팀과 공유하며 피드백 수집
4. **커스터마이징**: 프로젝트에 맞게 스크립트와 MCP 서버 수정

### 더 알아보기

- GSD GitHub: https://github.com/glittercowboy/get-shit-done
- Claude Code 문서: https://code.claude.com/docs
- Google Antigravity: https://antigravity.google/
- MCP 프로토콜: https://modelcontextprotocol.io/

행복한 코딩 되세요! 🚀

---

**작성 일자**: 2026-01-11
