---
title: "ai-adapter Dockerfile 가이드"
date: 2026-03-17 07:00:00 +0900
categories: [AI,  RummiArena]
mermaid: [True]
tags: [AI,  claude-code,  vibe-coding,  Claude.write]
---


>- **프로젝트**: RummiArena — `src/ai-adapter`
>- **빌드 방식**: NestJS 멀티스테이지(Multi-stage) 빌드
>- **베이스 이미지**: `node:22-alpine`
>- **엔트리포인트**: `node dist/main.js`
>- **최종 이미지 목표**: ~200 MB (LLM 클라이언트 라이브러리 포함)

```Dockerfile
# =============================================================================
# ai-adapter Dockerfile — NestJS 멀티스테이지 빌드
# Base: node:22-alpine
# 엔트리포인트: node dist/main.js
# 최종 이미지 목표: ~200 MB (LLM 클라이언트 라이브러리 포함)
# =============================================================================

# ---- Stage 1: 전체 의존성 설치 + 빌드 ----------------------------------------
FROM node:22-alpine AS builder

WORKDIR /app

# package.json / lock 파일만 먼저 복사 (레이어 캐시 최대화)
COPY package.json package-lock.json ./

# devDependencies 포함 설치 (TypeScript 컴파일에 필요)
RUN npm ci --prefer-offline

# 소스 복사 후 TypeScript 빌드
COPY . .
RUN npm run build

# ---- Stage 2: 프로덕션 의존성만 재설치 ----------------------------------------
# devDependencies를 제거해 런타임 이미지 크기를 최소화한다.
FROM node:22-alpine AS prod-deps

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --prefer-offline --omit=dev

# ---- Stage 3: 런타임 ---------------------------------------------------------
FROM node:22-alpine AS runner

WORKDIR /app

# non-root 사용자 생성 (uid=1001)
RUN addgroup -g 1001 -S nodejs && \
    adduser  -u 1001 -S nestjs -G nodejs

ENV NODE_ENV=production
ENV PORT=8081

# 컴파일된 JS + 프로덕션 의존성만 복사
COPY --from=prod-deps --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder   --chown=nestjs:nodejs /app/dist         ./dist

USER nestjs

EXPOSE 8081

# NestJS Health 엔드포인트 (src/health/health.controller.ts 에 정의됨)
HEALTHCHECK --interval=15s --timeout=5s --start-period=15s --retries=3 \
    CMD wget -qO- http://localhost:8081/health || exit 1

CMD ["node", "dist/main.js"]

```

---

## 목차

1. [개요](#1-개요)
2. [멀티스테이지 빌드 구조](#2-멀티스테이지-빌드-구조)
3. [Stage 1 — Builder (빌드 스테이지)](#3-stage-1--builder-빌드-스테이지)
4. [Stage 2 — prod-deps (프로덕션 의존성)](#4-stage-2--prod-deps-프로덕션-의존성)
5. [Stage 3 — Runner (런타임 스테이지)](#5-stage-3--runner-런타임-스테이지)
6. [보안 설계](#6-보안-설계)
7. [헬스체크](#7-헬스체크)
8. [환경 변수](#8-환경-변수)
9. [레이어 캐시 전략](#9-레이어-캐시-전략)
10. [빌드 및 실행 명령어](#10-빌드-및-실행-명령어)
11. [이미지 크기 최적화 요약](#11-이미지-크기-최적화-요약)

---

## 1. 개요

`ai-adapter`는 RummiArena 프로젝트 내에서 LLM(대형 언어 모델) 클라이언트와의 통신을 담당하는 NestJS 마이크로서비스입니다.

이 Dockerfile은 **3단계 멀티스테이지 빌드**를 채택하여 다음 목표를 달성합니다.

| 목표 | 방법 |
|------|------|
| 최소화된 런타임 이미지 | `devDependencies` 완전 제거 |
| 레이어 캐시 최대 활용 | `package.json` 먼저 복사 후 소스 복사 |
| 보안 강화 | non-root 사용자(`nestjs`, uid=1001)로 실행 |
| 빠른 장애 감지 | 내장 `HEALTHCHECK` 설정 |

---

## 2. 멀티스테이지 빌드 구조

```
┌──────────────────────────────────────────┐
│  Stage 1: builder                        │
│  - 전체 의존성 설치 (devDeps 포함)          │
│  - TypeScript → JavaScript 컴파일         │
│  - 산출물: /app/dist/**                   │
└───────────────────┬──────────────────────┘
                    │  COPY dist
                    ▼
┌──────────────────────────────────────────┐
│  Stage 2: prod-deps                      │
│  - npm ci --omit=dev                     │
│  - 프로덕션 전용 node_modules 구성         │
└───────────────────┬──────────────────────┘
                    │  COPY node_modules
                    ▼
┌──────────────────────────────────────────┐
│  Stage 3: runner  (최종 이미지)            │
│  - non-root 사용자 실행                    │
│  - dist/ + node_modules/ 만 포함          │
│  - 목표 크기 ~200 MB                      │
└──────────────────────────────────────────┘
```

---

## 3. Stage 1 — Builder (빌드 스테이지)

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --prefer-offline

COPY . .
RUN npm run build
```

### 단계별 설명

#### `FROM node:22-alpine AS builder`
- `node:22-alpine`: Node.js 22 LTS 기반의 경량 Alpine Linux 이미지 사용.
- `AS builder`: 이 스테이지를 `builder`라는 이름으로 참조 가능하게 지정.

#### `WORKDIR /app`
- 컨테이너 내 작업 디렉터리를 `/app`으로 설정.
- 이후 모든 `COPY`, `RUN` 명령은 `/app` 기준으로 동작.

#### `COPY package.json package-lock.json ./`
- **소스 파일보다 먼저** 패키지 메타데이터만 복사.
- 소스 코드가 변경되어도 `package.json`이 동일하면 `npm ci` 레이어 캐시가 재사용됨.

#### `RUN npm ci --prefer-offline`
- `npm ci`: `package-lock.json`을 기준으로 정확한 버전을 설치 (재현성 보장).
- `--prefer-offline`: 로컬 캐시를 우선 사용해 네트워크 의존성을 줄임.
- `devDependencies` 포함 설치 — TypeScript 컴파일러(`tsc`) 등이 필요하기 때문.

#### `COPY . .`
- 전체 소스 코드를 복사.
- `.dockerignore` 파일로 `node_modules/`, `.git/`, `dist/` 등을 제외하는 것을 권장.

#### `RUN npm run build`
- NestJS의 TypeScript 소스(`src/`)를 컴파일하여 `dist/` 디렉터리에 JavaScript로 출력.
- `package.json`의 `"build"` 스크립트 (`nest build` 또는 `tsc`) 실행.

---

## 4. Stage 2 — prod-deps (프로덕션 의존성)

```dockerfile
FROM node:22-alpine AS prod-deps
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --prefer-offline --omit=dev
```

### 단계별 설명

#### `FROM node:22-alpine AS prod-deps`
- builder 스테이지와 **독립된** 새 스테이지 시작.
- 이 스테이지의 목적은 런타임에 필요한 의존성만 포함된 `node_modules`를 생성하는 것.

#### `RUN npm ci --prefer-offline --omit=dev`
- `--omit=dev`: `devDependencies`를 제외하고 설치.
- TypeScript, ESLint, Jest 등 빌드/테스트 도구가 최종 이미지에 포함되지 않도록 차단.
- 이 과정에서 수십~수백 MB의 이미지 크기 절감 효과를 얻음.

> **왜 별도 스테이지인가?**
> builder에서 `node_modules`를 삭제 후 재설치하는 방식보다, 별도 스테이지에서 클린하게 설치하는 방식이 레이어 캐시 효율이 더 높음.

---

## 5. Stage 3 — Runner (런타임 스테이지)

```dockerfile
FROM node:22-alpine AS runner
WORKDIR /app

RUN addgroup -g 1001 -S nodejs && \
    adduser  -u 1001 -S nestjs -G nodejs

ENV NODE_ENV=production
ENV PORT=8081

COPY --from=prod-deps --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder   --chown=nestjs:nodejs /app/dist         ./dist

USER nestjs
EXPOSE 8081

HEALTHCHECK --interval=15s --timeout=5s --start-period=15s --retries=3 \
    CMD wget -qO- http://localhost:8081/health || exit 1

CMD ["node", "dist/main.js"]
```

### 단계별 설명

#### `FROM node:22-alpine AS runner`
- 최종 프로덕션 이미지를 위한 새 깨끗한 베이스.
- 이전 스테이지의 빌드 도구나 devDependencies가 전혀 포함되지 않음.

#### non-root 사용자 생성
```dockerfile
RUN addgroup -g 1001 -S nodejs && \
    adduser  -u 1001 -S nestjs -G nodejs
```
- `-S`: 시스템 계정으로 생성 (로그인 불가, 쉘 없음).
- `gid=1001`, `uid=1001`: 일관된 ID로 파일 권한 관리 용이.
- 컨테이너가 루트 권한으로 실행되지 않아 침해 시 피해 범위가 제한됨.

#### 환경 변수
```dockerfile
ENV NODE_ENV=production
ENV PORT=8081
```
- `NODE_ENV=production`: NestJS 및 npm 패키지들이 프로덕션 모드로 동작하도록 지정.
- `PORT=8081`: 서비스 리스닝 포트를 명시적으로 고정.

#### 파일 복사 (이전 스테이지에서)
```dockerfile
COPY --from=prod-deps --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder   --chown=nestjs:nodejs /app/dist         ./dist
```
- `--from=prod-deps`: 프로덕션 의존성만 포함된 `node_modules` 복사.
- `--from=builder`: 컴파일된 JavaScript(`dist/`) 복사.
- `--chown=nestjs:nodejs`: 복사 시점에 파일 소유권을 `nestjs` 사용자로 지정.

#### `USER nestjs`
- 이후 모든 프로세스를 `nestjs` 사용자(uid=1001)로 실행.
- `COPY` 이후 선언하여 복사 자체는 (내부적으로) 문제없이 수행.

#### `EXPOSE 8081`
- 컨테이너가 8081 포트를 사용함을 Docker에 명시 (문서화 목적).
- 실제 포트 바인딩은 `docker run -p` 또는 docker-compose `ports` 설정으로 처리.

#### `CMD ["node", "dist/main.js"]`
- NestJS 애플리케이션의 진입점 실행.
- exec 형식(`[]`)을 사용하여 PID 1로 직접 실행 — `SIGTERM` 신호가 Node.js에 정상 전달됨.

---

## 6. 보안 설계

| 항목 | 내용 |
|------|------|
| **non-root 실행** | `nestjs` 사용자(uid=1001)로만 프로세스 실행 |
| **최소 파일셋** | `dist/` + `node_modules/` 만 최종 이미지에 포함 |
| **소스 코드 미포함** | TypeScript 원본(`src/`) 및 설정 파일이 이미지에 없음 |
| **devDeps 제거** | 빌드 도구, 테스트 라이브러리 미포함으로 공격 표면 감소 |
| **Alpine 기반** | 최소한의 시스템 패키지로 CVE 노출 최소화 |

---

## 7. 헬스체크

```dockerfile
HEALTHCHECK --interval=15s --timeout=5s --start-period=15s --retries=3 \
    CMD wget -qO- http://localhost:8081/health || exit 1
```

| 옵션 | 값 | 의미 |
|------|-----|------|
| `--interval` | 15s | 15초마다 헬스체크 실행 |
| `--timeout` | 5s | 5초 내 응답 없으면 실패 처리 |
| `--start-period` | 15s | 컨테이너 시작 후 15초 동안은 실패 무시 (초기 부팅 허용) |
| `--retries` | 3 | 3회 연속 실패 시 `unhealthy` 상태로 전환 |

- 헬스 엔드포인트는 `src/health/health.controller.ts`에 정의된 `GET /health`.
- Alpine 이미지에 기본 포함된 `wget`을 활용 (`curl` 대신).

---

## 8. 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `NODE_ENV` | `production` | Node.js 실행 환경 모드 |
| `PORT` | `8081` | HTTP 서버 리스닝 포트 |

> 추가적인 LLM API 키(`OPENAI_API_KEY` 등) 및 서비스 설정은 런타임에 `-e` 옵션 또는 Kubernetes Secret / docker-compose `env_file`로 주입하는 것을 권장합니다.

---

## 9. 레이어 캐시 전략

멀티스테이지 빌드에서 `package.json`을 소스보다 먼저 복사하는 이유:

```
변경 없음: package.json  →  npm ci 레이어 캐시 HIT  →  빠른 빌드
변경 있음: src/*.ts      →  npm ci 레이어 캐시 HIT  →  빌드만 재실행
변경 있음: package.json  →  npm ci 레이어 캐시 MISS →  전체 재설치
```

이 순서를 지키면 **소스 코드만 변경되는 일반적인 개발 사이클에서 의존성 재설치를 건너뛸 수 있어** CI/CD 파이프라인 시간이 크게 단축됩니다.

---

## 10. 빌드 및 실행 명령어

### 이미지 빌드

```bash
# Dockerfile이 있는 디렉터리(src/ai-adapter)에서 실행
docker build -t rummi-arena/ai-adapter:latest .
```

### 컨테이너 실행

```bash
docker run -d \
  -p 8081:8081 \
  -e NODE_ENV=production \
  -e OPENAI_API_KEY=<your-key> \
  --name ai-adapter \
  rummi-arena/ai-adapter:latest
```

### 헬스 상태 확인

```bash
docker inspect --format='{{.State.Health.Status}}' ai-adapter
# 기대 출력: healthy
```

### 로그 확인

```bash
docker logs -f ai-adapter
```

### 이미지 크기 확인

```bash
docker image inspect rummi-arena/ai-adapter:latest \
  --format='{{.Size}}' | numfmt --to=iec
```

---

## 11. 이미지 크기 최적화 요약

```
일반 단일 스테이지 빌드 (예상)
  node:22  + devDeps + src + dist  ≈  800 MB ~ 1.2 GB

멀티스테이지 빌드 (현재 방식)
  node:22-alpine + prodDeps + dist  ≈  ~200 MB
                                       ↑ 목표치
```

| 절감 요소 | 효과 |
|-----------|------|
| Alpine 베이스 이미지 | `node:22` 대비 ~70% 감소 |
| devDependencies 제거 | TypeScript, Jest 등 수십~수백 MB 절감 |
| 소스 파일 미포함 | `src/`, `*.ts` 파일 없음 |
| 빌드 캐시 레이어 미포함 | npm 캐시(`~/.npm`) 최종 이미지에서 제외 |

---

- *이 문서는 [`src/ai-adapter/Dockerfile`](https://github.com/k82022603/RummiArena/blob/main/src/ai-adapter/Dockerfile) 기준으로 작성되었습니다.*
- *최종 업데이트: 2026-03-17*
