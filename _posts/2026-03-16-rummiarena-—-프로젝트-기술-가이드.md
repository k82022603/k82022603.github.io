---
title: "🎮 RummiArena — 프로젝트 기술 가이드"
date: 2026-03-16 13:00:00 +0900
categories: [AI,  RummiArena]
mermaid: [True]
tags: [AI,  claude-code,  vibe-coding,  Claude.write]
---




> 루미큐브(Rummikub) 보드게임 기반 **멀티 LLM 전략 실험 플랫폼**  
> Human + AI 혼합 2~4인 실시간 대전 지원 · 다양한 LLM 모델 전략 비교/분석

---

## 📋 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [전체 아키텍처](#2-전체-아키텍처)
3. [기술 스택 상세 설명](#3-기술-스택-상세-설명)
   - 3.1 [Frontend — Next.js 15](#31-frontend--nextjs-15)
   - 3.2 [Backend: Game Server — Go 1.24](#32-backend-game-server--go-124)
   - 3.3 [Backend: AI Adapter — NestJS](#33-backend-ai-adapter--nestjs)
   - 3.4 [인증 — Google OAuth 2.0 (NextAuth.js)](#34-인증--google-oauth-20-nextauthjs)
   - 3.5 [보안 & 품질 도구 — DevSecOps](#35-보안--품질-도구--devsecops)
4. [프로젝트 구조](#4-프로젝트-구조)
5. [게임 규칙 & 타일 인코딩](#5-게임-규칙--타일-인코딩)
6. [AI 캐릭터 시스템](#6-ai-캐릭터-시스템)
7. [API 명세 개요](#7-api-명세-개요)
8. [로컬 개발 환경 실행](#8-로컬-개발-환경-실행)
9. [Kubernetes 배포 (Helm)](#9-kubernetes-배포-helm)
10. [테스트 현황](#10-테스트-현황)

---

## 1. 프로젝트 개요

RummiArena는 고전 보드게임 루미큐브(Rummikub)를 온라인 멀티플레이 환경으로 구현하고, 여러 LLM(Large Language Model) AI를 플레이어로 참여시켜 전략을 비교·분석하는 실험 플랫폼입니다.

### 핵심 설계 원칙

| 원칙 | 설명 |
|------|------|
| **LLM 신뢰 금지** | LLM 응답은 항상 Game Engine으로 유효성 검증. Invalid move → 재요청 (최대 3회) → 실패 시 강제 드로우 |
| **AI Adapter 분리** | Game Engine은 특정 LLM에 의존하지 않음. 공통 인터페이스(`MoveRequest` / `MoveResponse`)로 모델 교체 가능 |
| **Stateless 서버** | 모든 게임 상태는 Redis에 저장. Pod 재시작에도 게임 유지 |
| **GitOps** | ArgoCD가 Helm chart 기반 배포 담당 |

---

## 2. 전체 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                         │
│   Frontend (Next.js 15)          Admin Panel (Next.js)      │
└──────────────────────┬──────────────────┬───────────────────┘
                       │ WebSocket / REST  │ 관리 API
┌──────────────────────▼──────────────────▼───────────────────┐
│                   Game Server (Go / gin)                     │
│   ┌─────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│   │ Game Engine │  │  REST API (gin) │  │ WebSocket Hub  │  │
│   │ (규칙 검증)  │  │                 │  │ (실시간 동기화) │  │
│   └─────────────┘  └─────────────────┘  └────────────────┘  │
└──────────┬───────────────────────────────────────────────────┘
           │
    ┌──────┴──────┐────────────────┐
    ▼             ▼                ▼
┌────────┐  ┌─────────┐  ┌──────────────────────┐
│ Redis  │  │Postgres │  │  AI Adapter (NestJS)  │
│(게임   │  │(영속    │  │  OpenAI / Claude      │
│ 상태)  │  │ 데이터) │  │  DeepSeek / Ollama    │
└────────┘  └─────────┘  └──────────────────────┘
```

### 인프라 구성

```
Docker Desktop Kubernetes
└── Namespace: rummikub
    ├── frontend        (Next.js — NodePort 30000)
    ├── game-server     (Go — NodePort 30080)
    ├── ai-adapter      (NestJS — NodePort 30081)
    ├── postgres        (PostgreSQL 16 — NodePort 30432)
    └── redis           (Redis 7)

CI/CD: GitLab CI + GitLab Runner
GitOps: ArgoCD + Helm 3
Ingress: Traefik v3
```

---

## 3. 기술 스택 상세 설명

### 전체 스택 요약

| 계층 | 기술 |
|------|------|
| Frontend | Next.js 15, TailwindCSS, Framer Motion, dnd-kit, Zustand |
| Backend (game-server) | Go 1.24, gin, gorilla/websocket, GORM, zap |
| Backend (ai-adapter) | NestJS, TypeScript, class-validator |
| Database | PostgreSQL 16, Redis 7 |
| AI Models | OpenAI API, Claude API, DeepSeek API, Ollama (LLaMA) |
| Infra | Docker Desktop Kubernetes, Helm 3, ArgoCD, Traefik v3 |
| CI/CD | GitLab CI + GitLab Runner |
| Quality | SonarQube, Trivy, OWASP ZAP |
| Auth | Google OAuth 2.0 (NextAuth.js) |

---

### 3.1 Frontend — Next.js 15

게임 UI와 실시간 상호작용을 담당하는 클라이언트 레이어입니다.

#### Next.js 15 (App Router)

- **서버 컴포넌트(RSC)**: 초기 로딩 속도를 높이고 JS 번들 크기를 최소화합니다.
- **서버 액션(Server Actions)**: 폼 제출과 데이터 변경을 서버에서 직접 처리합니다.
- **Partial Prerendering(PPR)**: 정적 콘텐츠는 즉시 전달하고, 동적 콘텐츠만 스트리밍하여 최상의 성능을 냅니다.

#### TailwindCSS

- 유틸리티 클래스만으로 빠르게 스타일링할 수 있어 별도의 CSS 파일이 불필요합니다.
- 런타임 JS 없이 정적 CSS를 생성하므로 서버 컴포넌트와 궁합이 좋습니다.

#### Framer Motion

- `layout` prop 하나로 타일이 이동할 때 부드러운 슬라이딩 효과를 구현합니다.
- `layoutId`를 활용해 타일 드래그 종료 후 자연스러운 재배치 애니메이션을 적용합니다.
- **주의**: 반드시 `'use client'` 컴포넌트에서 사용해야 합니다.

#### dnd-kit (Drag & Drop)

- 모듈형 설계로 필요한 기능만 선택해 가볍게 사용합니다.
- 키보드 조작과 스크린 리더를 기본 지원하여 접근성(A11y)을 확보합니다.
- `react-beautiful-dnd`보다 성능과 유지보수 면에서 우수합니다.

#### Zustand (상태 관리)

- Redux보다 훨씬 가볍고 보일러플레이트가 적습니다.
- 드래그 중인 타일 위치, 모달 상태, 손패(hand tile) 목록 등 전역 상태 관리에 사용합니다.
- Selector 기반 접근으로 불필요한 리렌더링을 방지합니다.

```typescript
// store/useTaskStore.ts — Zustand + dnd-kit 조합 예시
import { create } from 'zustand';
import { arrayMove } from '@dnd-kit/sortable';

interface TaskStore {
  tasks: string[];
  reorderTasks: (activeId: string, overId: string) => void;
}

export const useTaskStore = create<TaskStore>((set) => ({
  tasks: ['Task 1', 'Task 2', 'Task 3'],
  reorderTasks: (activeId, overId) => set((state) => {
    const oldIndex = state.tasks.indexOf(activeId);
    const newIndex = state.tasks.indexOf(overId);
    return { tasks: arrayMove(state.tasks, oldIndex, newIndex) };
  }),
}));
```

#### 프론트엔드 시너지 흐름

```
1. Zustand 스토어에 타일 배열 저장
2. dnd-kit SortableContext로 타일 목록 감싸기
3. onDragEnd → Zustand arrayMove로 배열 순서 업데이트
4. Framer Motion layout 애니메이션으로 부드러운 재배치
5. Next.js Server Action으로 변경 사항 DB에 저장 (useOptimistic 활용)
```

---

### 3.2 Backend: Game Server — Go 1.24

실시간 게임 로직, WebSocket 통신, 규칙 검증을 담당하는 핵심 서버입니다.

#### Go 1.24

- **고루틴(Goroutines)**: 수만 명의 동시 접속자를 가벼운 스레드로 처리합니다.
- **낮은 지연 시간(Low Latency)**: 컴파일 언어 특성상 실행 속도가 빠르며, 게임 서버의 핵심 요구사항인 빠른 응답을 보장합니다.
- Go 1.24의 제네릭 성능 향상과 메모리 최적화가 반영되었습니다.

#### Gin (HTTP 프레임워크)

- 로그인, 방 생성, 프로필 조회 등 REST API 처리에 사용합니다.
- 불필요한 기능을 제거하고 속도에 집중한 경량 프레임워크입니다.

#### Gorilla/WebSocket

- 게임 내 채팅, 타일 이동 동기화, 실시간 전투 등 양방향 통신을 담당합니다.
- Go 생태계에서 가장 안정적이고 널리 사용되는 WebSocket 구현체입니다.
- 고루틴과 결합하여 다수의 클라이언트를 동시에 효율적으로 처리합니다.

#### GORM (ORM)

- SQL 없이 Go 구조체(Struct)로 DB를 조작합니다.
- 유저 데이터, 아이템 인벤토리, 게임 전적 등을 PostgreSQL과 연동합니다.
- Auto Migration으로 데이터 모델 변경 시 테이블 구조를 자동 업데이트합니다.

#### Zap (로깅)

- Uber에서 개발한 초고속 구조화 로깅 라이브러리입니다.
- 최소한의 메모리 할당으로 로그를 기록하여 성능 저하를 방지합니다.
- JSON 형식 출력으로 대량의 게임 로그를 수집·분석하기 용이합니다.

#### 로컬 실행

```bash
cd src/game-server
go build ./cmd/server && ./server
```

> Go는 컴파일 언어이므로 `go build`로 바이너리를 생성한 후 실행합니다.

---

### 3.3 Backend: AI Adapter — NestJS

AI 모델(OpenAI, Claude, DeepSeek, Ollama)과 게임 서버 사이에서 데이터를 안전하게 중개하는 서버입니다.

#### NestJS

- **모듈/컨트롤러/서비스** 구조로 역할을 명확히 분리합니다.
- AI 응답 대기 시간이 길어지는 특성에 강한 비동기 처리 구조를 제공합니다.
- 팀 개발과 프로젝트 확장 시 코드 일관성을 유지하기 좋습니다.

#### TypeScript

- AI 모델에 보내는 프롬프트와 응답 데이터의 타입을 정의하여 런타임 에러를 방지합니다.
- 복잡한 AI 응답 객체 구조를 인터페이스로 관리하여 개발 생산성을 높입니다.

#### class-validator

- 프론트엔드 요청이 서버에 도달하기 전, 입구에서 즉시 유효성을 검사합니다.
- 데코레이터 방식으로 직관적인 검증 코드를 작성합니다.

```typescript
// DTO 설정 예시
import { IsString, IsInt, Min, Max } from 'class-validator';

export class ChatRequestDto {
  @IsString()
  message: string;       // AI에게 전달할 메시지

  @IsInt()
  @Min(1)
  @Max(500)
  maxTokens: number;     // 최대 토큰 수 제한 검증
}
```

- `ValidationPipe` 설정 시, 잘못된 데이터 입력에 자동으로 `400 Bad Request` 응답을 반환합니다.

#### AI Adapter 처리 흐름

```
1. class-validator       → 요청 데이터 유효성 검사
2. NestJS Service        → 해당 AI 모델 SDK 호출 (OpenAI / Claude / DeepSeek / Ollama)
3. 응답 파싱 및 변환     → MoveResponse 형태로 정형화
4. Game Server 반환      → 결과 전달
5. Game Engine 최종 검증 → 최대 3회 재요청 후 실패 시 강제 드로우
```

#### 로컬 실행

```bash
cd src/ai-adapter
npm install && npm run start:dev
```

> `start:dev`는 Watch mode로 실행되어 코드 수정 시 서버가 자동 재시작됩니다.

---

### 3.4 인증 — Google OAuth 2.0 (NextAuth.js)

NextAuth.js v5(Auth.js) 기반으로 Google 소셜 로그인을 구현합니다.

#### Google Cloud Console 설정

1. Google Cloud Console에서 프로젝트 생성
2. **API 및 서비스 > OAuth 동의 화면** → 외부(External) 선택 후 필수 정보 입력
3. **사용자 인증 정보 만들기 > OAuth 클라이언트 ID** 생성
   - 애플리케이션 유형: 웹 애플리케이션
   - 승인된 리디렉션 URI: `http://localhost:3000/api/auth/callback/google`
4. 발급된 클라이언트 ID와 보안 비밀번호를 `.env`에 저장

#### 환경 변수 설정 (`.env`)

```env
AUTH_GOOGLE_ID=발급받은_클라이언트_ID
AUTH_GOOGLE_SECRET=발급받은_클라이언트_보안_비밀번호
AUTH_SECRET=랜덤_시크릿_키  # openssl rand -base64 32 으로 생성 권장
```

#### 코드 구현

```bash
npm install next-auth@beta
```

```typescript
// auth.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [Google],
})
```

```typescript
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"
export const { GET, POST } = handlers
```

```tsx
// 로그인 버튼 (Client Component)
import { signIn } from "next-auth/react"

export default function LoginButton() {
  return <button onClick={() => signIn("google")}>Google로 로그인</button>
}
```

```tsx
// 세션 확인 (Server Component)
import { auth } from "@/auth"

export default async function Page() {
  const session = await auth()
  if (!session) return <div>로그인이 필요합니다.</div>
  return <div>안녕하세요, {session.user?.name}님!</div>
}
```

> **보안 주의**: `AUTH_SECRET`은 세션 쿠키 암호화에 사용됩니다. 절대 외부에 노출하지 마세요.

---

### 3.5 보안 & 품질 도구 — DevSecOps

CI/CD 파이프라인에 통합된 자동화 보안 및 품질 검사 도구입니다.

#### 도구 비교

| 도구 | 분석 방식 | 주요 대상 | 주요 목적 |
|------|-----------|-----------|-----------|
| **SonarQube** | 정적 (SAST) | 소스 코드 | 코드 품질 향상 및 초기 버그 방지 |
| **Trivy** | 정적 / 환경 | 컨테이너, 라이브러리 | 배포 환경 및 인프라 보안 강화 |
| **OWASP ZAP** | 동적 (DAST) | 실행 중인 웹 앱 | 실제 서비스 환경의 공격 방어 및 검증 |

#### SonarQube — 정적 코드 분석 (SAST)

소스 코드를 실행하지 않고 분석하여 품질과 보안 취약점을 찾습니다.

- **버그 및 코드 스멜 탐지**: 잠재적 오류와 유지보수가 어려운 코드 구조를 식별합니다.
- **보안 취약점 분석**: SQL Injection, XSS 등 코드 수준의 보안 위협을 경고합니다.
- **코드 커버리지**: 테스트가 실제 로직을 얼마나 검증하는지 시각화합니다.
- **CI/CD 통합**: GitLab CI와 연결하여 커밋마다 자동 검사를 수행합니다.

#### Trivy — 컨테이너 및 인프라 보안

Aqua Security에서 개발한 오픈소스 보안 스캐너입니다.

- **OS 패키지 취약점 스캔**: 컨테이너 이미지(Alpine, Ubuntu 등) 내 패키지 보안 결함을 탐지합니다.
- **애플리케이션 종속성 스캔**: Go, Node.js 등 프로젝트 라이브러리의 취약점을 탐지합니다.
- **설정 오류 점검**: Dockerfile, Kubernetes YAML의 보안 위험 구성을 찾습니다.
- **빠른 속도**: 가볍고 빨라 컨테이너 빌드 시 즉각적인 검사가 가능합니다.

#### OWASP ZAP — 동적 웹 보안 테스트 (DAST)

웹 애플리케이션을 실제 실행한 상태에서 공격을 시뮬레이션합니다.

- **자동 스캔**: 웹 사이트를 크롤링하며 OWASP Top 10 취약점을 자동 테스트합니다.
- **스파이더링**: 모든 페이지와 URL 구조를 자동으로 분석합니다.
- **API/CLI 지원**: 자동화된 배포 파이프라인에 보안 점검을 포함시킬 수 있습니다.

---

## 4. 프로젝트 구조

```
RummiArena/
├── docs/
│   ├── 01-planning/       # 기획 (헌장, 요구사항, WBS, 백로그)
│   ├── 02-design/         # 설계 (아키텍처, DB, API, WebSocket, AI Adapter)
│   ├── 03-development/    # 개발 가이드
│   ├── 04-testing/        # 테스트 전략 + 보고서
│   └── 05-deployment/     # 배포 가이드 + K8s 아키텍처
├── src/
│   ├── game-server/       # Go 백엔드 (REST API + Game Engine)
│   │   └── cmd/server/    # 메인 엔트리포인트
│   ├── ai-adapter/        # NestJS AI 서비스
│   ├── frontend/          # Next.js 게임 UI
│   └── admin/             # 관리자 대시보드
├── helm/
│   └── charts/
│       ├── postgres/      # PostgreSQL Helm 차트 (Bitnami 기반)
│       ├── redis/         # Redis Helm 차트 (Bitnami 기반)
│       ├── game-server/   # 커스텀 Helm 차트 (직접 작성)
│       ├── ai-adapter/    # 커스텀 Helm 차트 (직접 작성)
│       └── frontend/      # 커스텀 Helm 차트 (직접 작성)
└── work_logs/             # 세션 / 데일리 / 스크럼 로그
```

### Helm 차트 종류

| 차트 | 출처 | 비고 |
|------|------|------|
| `charts/postgres` | Bitnami 공용 차트 활용 | `bitnami/postgresql` 기반 |
| `charts/redis` | Bitnami 공용 차트 활용 | `bitnami/redis` 기반 |
| `charts/game-server` | **직접 작성** | Go 서버 이미지, 포트, 환경 변수 설정 |
| `charts/ai-adapter` | **직접 작성** | NestJS 서버 이미지, AI API 키 설정 |
| `charts/frontend` | **직접 작성** | Next.js 이미지, NodePort 설정 |

> 커스텀 차트는 `helm create charts/game-server`로 기본 템플릿 생성 후 `values.yaml`을 수정하여 사용합니다.

---

## 5. 게임 규칙 & 타일 인코딩

컴퓨터와 AI가 타일을 고유하게 식별하기 위한 코드 규칙입니다.

### 타일 코드 구조: `{Color}{Number}{Set}`

| 구성 요소 | 값 | 설명 |
|-----------|-----|------|
| **Color** | `R`, `B`, `Y`, `K` | Red, Blue, Yellow, Black |
| **Number** | `1` ~ `13` | 타일에 적힌 숫자 |
| **Set** | `a`, `b` | 동일한 타일이 2개씩 존재하므로 구분 필요 |
| **Joker** | `JK1`, `JK2` | 조커 타일 2개 구분 |

> **Black을 'K'로 표기하는 이유**: 'B'가 이미 Blue에 사용되어 충돌을 방지하기 위함입니다.

### 타일 코드 예시

| 코드 | 의미 |
|------|------|
| `R7a` | 빨간색 7번 첫 번째 타일 |
| `R7b` | 빨간색 7번 두 번째 타일 |
| `B13b` | 파란색 13번 두 번째 타일 |
| `Y1a` | 노란색 1번 첫 번째 타일 |
| `K10b` | 검정색 10번 두 번째 타일 |
| `JK1` | 첫 번째 조커 |
| `JK2` | 두 번째 조커 |

### 개발 시 활용

- **Go 서버**: WebSocket 패킷을 문자열 배열로 전송하여 패킷 크기를 최소화합니다.  
  예) `["R7a", "R8a", "R9a"]` → 같은 색 연속 3장 그룹 구성
- **AI 어댑터**: LLM 프롬프트에 인코딩 규칙을 포함하여 AI가 게임 상태를 정확히 파악하도록 합니다.
- **Zustand**: 타일 코드를 `key` 값으로 사용하여 드래그 앤 드롭 상태를 관리합니다.

---

## 6. AI 캐릭터 시스템

다양한 전략 스타일을 가진 AI 캐릭터로 게임 다양성을 확보합니다.

| 캐릭터 | 스타일 | 설명 |
|--------|--------|------|
| **Rookie** | 보수적 | 안전한 수만 선택 |
| **Calculator** | 확률 기반 | 기대값 계산으로 최적 수 도출 |
| **Shark** | 공격적 | 상대 견제 + 대량 배치 전략 |
| **Fox** | 기만적 | 의도적 지연 + 역전 패턴 구사 |
| **Wall** | 수비적 | 최소 배치 + 자원 비축 |
| **Wildcard** | 예측 불가 | 랜덤 전략 혼합 |

- **난이도**: 하수 / 중수 / 고수
- **심리전 레벨**: 0 ~ 3단계

---

## 7. API 명세 개요

### Room Management

```
POST   /api/rooms             # 방 생성
GET    /api/rooms             # 방 목록 조회
GET    /api/rooms/:id         # 방 상세 조회
POST   /api/rooms/:id/join    # 방 입장
POST   /api/rooms/:id/leave   # 방 퇴장
POST   /api/rooms/:id/start   # 게임 시작
DELETE /api/rooms/:id         # 방 삭제
```

### Game Actions

```
GET    /api/games/:id            # 게임 상태 조회 (1인칭 뷰)
POST   /api/games/:id/place      # 타일 임시 배치
POST   /api/games/:id/confirm    # 턴 확정 (Game Engine 유효성 검증)
POST   /api/games/:id/draw       # 타일 드로우
POST   /api/games/:id/reset      # 턴 되돌리기
```

### Health Check

```
GET    /health    # 서버 상태 + Redis 연결 확인
GET    /ready     # 준비 상태 확인
```

### Service Endpoints (NodePort)

| 서비스 | URL | Port |
|--------|-----|------|
| Frontend | http://localhost:30000 | 30000 |
| Game Server | http://localhost:30080 | 30080 |
| AI Adapter | http://localhost:30081 | 30081 |
| PostgreSQL | localhost:30432 | 30432 |

---

## 8. 로컬 개발 환경 실행

### 사전 요구사항

- Docker Desktop (Kubernetes 활성화)
- Helm 3
- Node.js 20+
- Go 1.24+

### 1. Game Server 실행 (Go)

```bash
cd src/game-server
go build ./cmd/server && ./server
```

- `go build ./cmd/server`: 소스 코드를 컴파일하여 바이너리 생성
- `&& ./server`: 빌드 성공 시 즉시 서버 실행
- WebSocket 통신 대기 상태가 됩니다.

### 2. AI Adapter 실행 (NestJS)

```bash
cd src/ai-adapter
npm install && npm run start:dev
```

- `npm install`: `package.json`에 명시된 모든 패키지 설치
- `npm run start:dev`: 개발 모드(Watch mode) 실행. 코드 저장 시 자동 재시작

### 3. Frontend 실행 (Next.js)

```bash
cd src/frontend
npm install && npm run dev
```

- `npm run dev`: Next.js 개발 서버 실행 (`http://localhost:3000`)
- Hot Module Replacement(HMR) 지원. 코드 수정 시 즉시 화면 반영

> **주의**: Framer Motion, dnd-kit, Zustand는 브라우저 API를 사용하므로 반드시 `'use client'` 컴포넌트에서 구현해야 합니다.

---

## 9. Kubernetes 배포 (Helm)

### 네임스페이스 생성

```bash
kubectl create namespace rummikub
```

> `rummikub` 전용 격리 공간을 생성합니다. 이후 모든 리소스는 이 네임스페이스 안에서 관리됩니다.

### Helm 차트 배포 (5개 서비스)

```bash
cd helm

# 1. 인프라 계층 (DB, 캐시)
helm install postgres    charts/postgres    -n rummikub
helm install redis       charts/redis       -n rummikub

# 2. 백엔드 서비스
helm install game-server charts/game-server -n rummikub
helm install ai-adapter  charts/ai-adapter  -n rummikub

# 3. 프론트엔드
helm install frontend    charts/frontend    -n rummikub
```

### 배포 확인

```bash
# 파드 상태 확인
kubectl get pods -n rummikub

# 서비스 확인
kubectl get services -n rummikub

# 특정 파드 로그 확인
kubectl logs -f <pod-name> -n rummikub
```

### 배포 업데이트 / 삭제

```bash
# 설정 변경 후 업데이트
helm upgrade game-server charts/game-server -n rummikub

# 특정 서비스 삭제
helm uninstall game-server -n rummikub

# 전체 네임스페이스 삭제
kubectl delete namespace rummikub
```

---

## 10. 테스트 현황

| 테스트 분류 | 결과 | 커버리지 |
|-------------|------|----------|
| Engine Unit Tests | ✅ 69/69 PASS | 96.5% |
| Smoke Tests | ✅ 16/16 PASS | — |
| Integration Tests | ✅ 31/31 PASS | REST API + DB/Redis |

### 관련 문서

| 문서 | 경로 |
|------|------|
| 프로젝트 헌장 | `docs/01-planning/01-project-charter.md` |
| 아키텍처 설계 | `docs/02-design/01-architecture.md` |
| DB 설계 | `docs/02-design/02-database-design.md` |
| API 설계 | `docs/02-design/03-api-design.md` |
| WebSocket 프로토콜 | `docs/02-design/10-websocket-protocol.md` |
| 테스트 전략 | `docs/04-testing/01-test-strategy.md` |
| 통합 테스트 보고서 | `docs/04-testing/06-integration-test-report.md` |

---

*이 문서는 RummiArena 프로젝트의 기술 스택, 아키텍처, 개발 및 배포 가이드를 포함합니다.*  
*문의 및 기여: [GitHub — RummiArena](https://github.com/k82022603/RummiArena)*
