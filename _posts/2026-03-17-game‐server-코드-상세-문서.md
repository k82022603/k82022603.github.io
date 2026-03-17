---
title: "game-server 코드 상세 문서"
date: 2026-03-17 20:00:00 +0900
categories: [AI,  RummiArena]
mermaid: [True]
tags: [AI,  claude-code,  vibe-coding,  Claude.write]
---


> **대상 파일**
> - [`cmd/server/main.go`](https://github.com/k82022603/RummiArena/blob/main/src/game-server/cmd/server/main.go) — 서버 엔트리포인트
> - [`Dockerfile`](https://github.com/k82022603/RummiArena/blob/main/src/game-server/Dockerfile) — 멀티스테이지 컨테이너 빌드

---

## 목차

1. [전체 아키텍처 개요](#1-전체-아키텍처-개요)
2. [main.go 상세 설명](#2-maingo-상세-설명)
   - 2.1 [패키지 임포트](#21-패키지-임포트)
   - 2.2 [main()](#22-main)
   - 2.3 [initInfra()](#23-initinfra)
   - 2.4 [initGameStateRepo()](#24-initgamestaterepo)
   - 2.5 [buildRouter()](#25-buildrouter)
   - 2.6 [registerSystemRoutes()](#26-registersystemroutes)
   - 2.7 [registerWSRoutes()](#27-registerwsroutes)
   - 2.8 [registerAPIRoutes()](#28-registerapiroutes)
   - 2.9 [runServer()](#29-runserver)
   - 2.10 [shutdownConnections()](#210-shutdownconnections)
3. [Dockerfile 상세 설명](#3-dockerfile-상세-설명)
   - 3.1 [Stage 1 — deps](#31-stage-1--deps-의존성-다운로드)
   - 3.2 [Stage 2 — builder](#32-stage-2--builder-빌드)
   - 3.3 [Stage 3 — runner](#33-stage-3--runner-런타임)
4. [엔드포인트 목록](#4-엔드포인트-목록)
5. [의존성 요약](#5-의존성-요약)
6. [운영 고려사항](#6-운영-고려사항)

---

## 1. 전체 아키텍처 개요

```
┌─────────────────────────────────────────────────────┐
│                    game-server                      │
│                                                     │
│  ┌───────────┐   ┌──────────┐   ┌───────────────┐  │
│  │  REST API │   │WebSocket │   │  System       │  │
│  │  /api/... │   │   /ws    │   │  /health      │  │
│  └─────┬─────┘   └────┬─────┘   │  /ready       │  │
│        │              │         └───────────────┘  │
│  ┌─────▼──────────────▼──────┐                     │
│  │        gin Router         │                     │
│  └─────┬──────────────┬──────┘                     │
│        │              │                            │
│  ┌─────▼─────┐  ┌─────▼─────┐                     │
│  │ Services  │  │  WS Hub   │                     │
│  │ Room/Game │  │           │                     │
│  │ Turn      │  └───────────┘                     │
│  └─────┬─────┘                                    │
│        │                                          │
│  ┌─────▼────────────────────┐                     │
│  │      Repositories        │                     │
│  │  GameState │ Room        │                     │
│  └─────┬──────┴─────────────┘                     │
│        │                                          │
│  ┌─────▼──────┐  ┌──────────┐                     │
│  │   Redis    │  │ Postgres │                     │
│  └────────────┘  └──────────┘                     │
└─────────────────────────────────────────────────────┘
```

`main.go`는 서버의 **부트스트랩 계층**으로, 인프라 초기화 → 라우터 구성 → 서버 실행 → Graceful Shutdown의 전체 생명주기를 관리합니다.

---

## 2. main.go 상세 설명

### 2.1 패키지 임포트

```go
import (
    "context"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/redis/go-redis/v9"
    "go.uber.org/zap"
    "gorm.io/gorm"

    "github.com/k82022603/RummiArena/game-server/internal/config"
    "github.com/k82022603/RummiArena/game-server/internal/handler"
    "github.com/k82022603/RummiArena/game-server/internal/infra"
    "github.com/k82022603/RummiArena/game-server/internal/middleware"
    "github.com/k82022603/RummiArena/game-server/internal/repository"
    "github.com/k82022603/RummiArena/game-server/internal/service"
)
```

| 패키지 | 역할 |
|--------|------|
| `gin` | HTTP 라우터 및 미들웨어 프레임워크 |
| `go-redis/v9` | Redis 클라이언트 (Redis 7+ 지원) |
| `zap` | 고성능 구조화 로깅 (Uber 오픈소스) |
| `gorm` | ORM — PostgreSQL 연결 관리 |
| `config` | 환경변수/설정 파일 로딩 |
| `handler` | HTTP/WebSocket 요청 핸들러 |
| `infra` | DB·Redis 연결 팩토리 |
| `middleware` | JWT 인증, ZapLogger 등 gin 미들웨어 |
| `repository` | 데이터 접근 계층 (인터페이스 + 구현체) |
| `service` | 비즈니스 로직 계층 |

---

### 2.2 `main()`

```go
func main() {
    logger, err := zap.NewProduction()
    ...
    cfg, err := config.Load()
    ...
    db, redisClient := initInfra(cfg, logger)
    gameStateRepo := initGameStateRepo(redisClient, logger)
    roomRepo := repository.NewMemoryRoomRepo()
    gin.SetMode(cfg.Server.Mode)
    router := buildRouter(cfg, logger, redisClient, gameStateRepo, roomRepo)
    srv := &http.Server{ ... }
    runServer(srv, cfg, logger)
    shutdownConnections(db, redisClient, logger)
    logger.Info("server stopped")
}
```

**실행 순서**

```
[1] zap 프로덕션 로거 초기화
        ↓
[2] config.Load() — 환경변수/설정파일 파싱
        ↓
[3] initInfra() — PostgreSQL & Redis 연결
        ↓
[4] initGameStateRepo() — Redis 가용 여부에 따라 구현체 선택
        ↓
[5] repository.NewMemoryRoomRepo() — 룸 저장소 초기화
        ↓
[6] gin.SetMode() — debug / release / test 모드 설정
        ↓
[7] buildRouter() — 전체 라우트 구성
        ↓
[8] http.Server 생성 (타임아웃 설정 포함)
        ↓
[9] runServer() — goroutine으로 서버 기동 + SIGINT/SIGTERM 대기
        ↓
[10] shutdownConnections() — Redis, PostgreSQL 연결 종료
        ↓
[11] "server stopped" 로그 출력 후 프로세스 종료
```

**`http.Server` 타임아웃 설정 근거**

| 필드 | 값 | 이유 |
|------|-----|------|
| `ReadTimeout` | 15s | 느린 클라이언트로 인한 커넥션 고갈 방지 |
| `WriteTimeout` | 15s | 응답 지연으로 인한 리소스 낭비 방지 |
| `IdleTimeout` | 60s | Keep-Alive 연결 재사용 허용 범위 |

---

### 2.3 `initInfra()`

```go
func initInfra(cfg *config.Config, logger *zap.Logger) (*gorm.DB, *redis.Client) {
    db, err := infra.NewPostgresDB(cfg.DB, logger)
    if err != nil {
        logger.Warn("postgres unavailable — running without DB", zap.Error(err))
    } else if err := infra.AutoMigrate(db, logger); err != nil {
        logger.Warn("auto migrate failed", zap.Error(err))
    }
    redisClient, _ := infra.NewRedisClient(cfg.Redis, logger)
    return db, redisClient
}
```

**핵심 설계 포인트**

- PostgreSQL 연결 실패 시 **`Fatal` 대신 `Warn`** 처리 → 서버는 DB 없이도 기동 가능 (개발/테스트 환경 유연성 확보)
- `AutoMigrate`: GORM의 자동 스키마 마이그레이션을 실행 — 테이블이 없으면 생성, 컬럼 변경 사항 반영
- Redis 연결 실패도 마찬가지로 경고 처리 (`_`로 에러 무시); `initGameStateRepo`에서 fallback 처리

---

### 2.4 `initGameStateRepo()`

```go
func initGameStateRepo(redisClient *redis.Client, logger *zap.Logger) repository.MemoryGameStateRepository {
    if infra.IsRedisAvailable(redisClient) {
        logger.Info("using redis game state repository")
        return repository.NewRedisGameStateMemAdapter(redisClient)
    }
    logger.Warn("using in-memory game state repository (redis unavailable)")
    return repository.NewMemoryGameStateRepoAdapter()
}
```

**전략 패턴 적용**

```
MemoryGameStateRepository (인터페이스)
        ├── RedisGameStateMemAdapter   ← Redis 가용 시
        └── MemoryGameStateRepoAdapter ← Redis 불가 시 (in-process 메모리)
```

- `infra.IsRedisAvailable()`: Redis `PING` 명령으로 실시간 연결 상태 확인
- In-memory fallback은 **단일 서버 환경/로컬 개발** 용도로만 적합 (서버 재시작 시 게임 상태 소멸)

---

### 2.5 `buildRouter()`

```go
func buildRouter(...) *gin.Engine {
    roomSvc  := service.NewRoomService(roomRepo, gameStateRepo)
    gameSvc  := service.NewGameService(gameStateRepo)
    turnSvc  := service.NewTurnService(gameStateRepo, gameSvc)
    wsHub    := handler.NewHub(logger)

    roomHandler := handler.NewRoomHandler(roomSvc)
    gameHandler := handler.NewGameHandler(gameSvc)
    wsHandler   := handler.NewWSHandler(wsHub, roomSvc, gameSvc, turnSvc, cfg.JWT.Secret, logger)
    authHandler := handler.NewAuthHandler(cfg.JWT.Secret)

    router := gin.New()
    router.Use(gin.Recovery())
    router.Use(middleware.ZapLogger(logger))
    ...
}
```

**의존성 주입(DI) 흐름**

```
Repository 계층
    └→ Service 계층
            └→ Handler 계층
                    └→ Router에 등록
```

**전역 미들웨어**

| 미들웨어 | 역할 |
|----------|------|
| `gin.Recovery()` | 핸들러 panic → 500 응답으로 복구 (서버 크래시 방지) |
| `middleware.ZapLogger(logger)` | 모든 요청/응답을 구조화된 JSON 로그로 기록 |

`gin.New()` 사용 이유: `gin.Default()`는 기본 logger와 recovery를 포함하지만, 여기서는 **zap 기반 커스텀 로거**를 사용하기 위해 `gin.New()`로 시작 후 수동으로 미들웨어를 추가합니다.

---

### 2.6 `registerSystemRoutes()`

```go
func registerSystemRoutes(router *gin.Engine, redisClient *redis.Client) {
    router.GET("/health", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "status":    "ok",
            "timestamp": time.Now().UTC().Format(time.RFC3339),
            "redis":     infra.IsRedisAvailable(redisClient),
        })
    })
    router.GET("/ready", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"status": "ready"})
    })
}
```

| 엔드포인트 | 용도 | 응답 예시 |
|------------|------|-----------|
| `GET /health` | 생존 여부 + 의존성 상태 | `{"status":"ok","timestamp":"...","redis":true}` |
| `GET /ready` | 트래픽 수신 준비 완료 | `{"status":"ready"}` |

- `/health`는 Dockerfile의 `HEALTHCHECK`와 연동됨
- `/ready`는 Kubernetes `readinessProbe`에 사용 (트래픽 라우팅 전 준비 확인)
- JWT 인증 없이 접근 가능 (인프라/오케스트레이션 레벨 확인용)

---

### 2.7 `registerWSRoutes()`

```go
func registerWSRoutes(router *gin.Engine, wsHandler *handler.WSHandler) {
    router.GET("/ws", wsHandler.HandleWS)
}
```

- 단일 엔드포인트 `/ws`로 WebSocket 업그레이드 처리
- 인증은 `WSHandler` 내부에서 JWT Secret으로 수행 (`cfg.JWT.Secret` 주입됨)
- `wsHub`를 통해 연결된 클라이언트 간 메시지 브로드캐스트 관리

---

### 2.8 `registerAPIRoutes()`

```go
func registerAPIRoutes(...) {
    api := router.Group("/api")

    // 개발 전용
    if cfg.AppEnv == "dev" {
        auth := api.Group("/auth")
        auth.POST("/dev-login", authHandler.DevLogin)
    }

    rooms := api.Group("/rooms")
    rooms.Use(middleware.JWTAuth(cfg.JWT.Secret))
    { ... }

    games := api.Group("/games")
    games.Use(middleware.JWTAuth(cfg.JWT.Secret))
    { ... }
}
```

**라우트 그룹 구조**

```
/api
 ├── /auth
 │    └── POST /dev-login          [APP_ENV=dev 전용, JWT 불필요]
 ├── /rooms                        [JWT 필요]
 │    ├── POST   /                 방 생성
 │    ├── GET    /                 방 목록 조회
 │    ├── GET    /:id              방 단건 조회
 │    ├── POST   /:id/join         방 입장
 │    ├── POST   /:id/leave        방 퇴장
 │    ├── POST   /:id/start        게임 시작
 │    └── DELETE /:id              방 삭제
 └── /games                        [JWT 필요]
      ├── GET    /:id              게임 상태 조회
      ├── POST   /:id/place        타일 배치
      ├── POST   /:id/confirm      턴 확정
      ├── POST   /:id/draw         타일 뽑기
      └── POST   /:id/reset        턴 초기화
```

`middleware.JWTAuth`는 그룹 단위로 적용되어 하위 모든 라우트에 인증이 강제됩니다.

---

### 2.9 `runServer()`

```go
func runServer(srv *http.Server, cfg *config.Config, logger *zap.Logger) {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    go func() {
        logger.Info("server starting", zap.String("port", cfg.Server.Port))
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            logger.Fatal("server error", zap.Error(err))
        }
    }()

    <-quit
    logger.Info("shutting down server...")

    shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    if err := srv.Shutdown(shutdownCtx); err != nil {
        logger.Error("server shutdown error", zap.Error(err))
    }
}
```

**Graceful Shutdown 흐름**

```
goroutine: srv.ListenAndServe() ──── 정상 서비스 중
                                              │
                              SIGINT / SIGTERM 수신
                                              │
                              quit 채널에 신호 전달
                                              │
                          context.WithTimeout(10s) 생성
                                              │
                              srv.Shutdown(ctx) 호출
                                    ┌─────────┴──────────┐
                              진행 중인 요청         새 요청 거부
                              10초 내 완료 대기        즉시 거부
                                              │
                                    10s 타임아웃 또는 완료
                                              │
                                       함수 반환
```

- `SIGINT`: Ctrl+C (터미널 인터럽트)
- `SIGTERM`: `docker stop` / Kubernetes Pod 종료 시 전송
- 10초 이내 진행 중인 요청이 완료되지 않으면 강제 종료

---

### 2.10 `shutdownConnections()`

```go
func shutdownConnections(db *gorm.DB, redisClient *redis.Client, logger *zap.Logger) {
    if redisClient != nil {
        if err := redisClient.Close(); err != nil {
            logger.Warn("redis close error", zap.Error(err))
        }
    }
    if db != nil {
        if sqlDB, err := db.DB(); err == nil {
            if err := sqlDB.Close(); err != nil {
                logger.Warn("postgres close error", zap.Error(err))
            }
        }
    }
}
```

- `nil` 체크 후 닫기 → `initInfra`에서 연결 실패 시 `nil`이 전달될 수 있음
- Redis → PostgreSQL 순서로 닫음 (게임 상태 저장 후 DB 커넥션 풀 해제)
- 연결 해제 실패는 `Warn`으로 기록하고 프로세스를 종료 (이미 shutdown 중이므로 Fatal 불필요)

---

## 3. Dockerfile 상세 설명

멀티스테이지 빌드를 사용하여 **빌드 환경과 런타임 환경을 분리**합니다. 최종 이미지 목표 크기는 **~20 MB**입니다.

```
Stage 1 (deps)          Stage 2 (builder)         Stage 3 (runner)
golang:1.24-alpine  →   golang:1.24-alpine    →    alpine:3.21
  go.mod, go.sum          소스 전체 복사               바이너리만 복사
  go mod download         go build                     ~20 MB
  캐시 레이어 최대화       정적 바이너리 생성
```

---

### 3.1 Stage 1 — `deps` (의존성 다운로드)

```dockerfile
FROM golang:1.24-alpine AS deps
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify
```

**레이어 캐싱 전략**

Docker는 각 `RUN`, `COPY` 명령을 별도 레이어로 저장합니다.

```
Layer 1: FROM golang:1.24-alpine    ← 거의 변경 없음 (베이스 이미지)
Layer 2: WORKDIR /build             ← 변경 없음
Layer 3: COPY go.mod go.sum ./      ← 의존성 변경 시에만 무효화
Layer 4: RUN go mod download        ← 의존성 변경 시에만 재실행
```

- `go.mod`, `go.sum`만 먼저 복사하여 **소스 변경과 의존성 다운로드를 분리**
- 소스만 수정했을 경우 Layer 3, 4의 캐시가 유지되어 **빌드 속도 대폭 향상**
- `go mod verify`: 다운로드된 모듈의 체크섬 검증 (공급망 보안)

---

### 3.2 Stage 2 — `builder` (빌드)

```dockerfile
FROM deps AS builder
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build \
    -trimpath \
    -ldflags="-s -w" \
    -o /app/server \
    ./cmd/server
```

**빌드 플래그 상세**

| 플래그/환경변수 | 효과 |
|----------------|------|
| `CGO_ENABLED=0` | C 라이브러리 의존성 제거 → **순수 정적 바이너리** 생성. alpine에 glibc 없어도 실행 가능 |
| `GOOS=linux` | Linux용 바이너리 생성 (macOS에서 빌드해도 Linux 바이너리 출력) |
| `GOARCH=amd64` | x86-64 아키텍처 타깃 |
| `-trimpath` | 바이너리 내 절대 빌드 경로 제거 → **재현 가능한 빌드** + 개발자 로컬 경로 노출 방지 |
| `-ldflags="-s"` | 심볼 테이블 제거 → 바이너리 크기 축소 |
| `-ldflags="-w"` | DWARF 디버그 정보 제거 → 바이너리 크기 추가 축소 |
| `-o /app/server` | 출력 바이너리 경로 지정 |

> **정적 바이너리의 이점**: 런타임 이미지에 Go 런타임이나 libc가 불필요하여 최소한의 alpine 베이스만으로 실행 가능합니다.

---

### 3.3 Stage 3 — `runner` (런타임)

```dockerfile
FROM alpine:3.21 AS runner

RUN apk --no-cache add ca-certificates tzdata && \
    addgroup -g 1001 -S appgroup && \
    adduser  -u 1001 -S appuser -G appgroup

WORKDIR /app
COPY --from=builder --chown=appuser:appgroup /app/server ./server

USER appuser
EXPOSE 8080

HEALTHCHECK --interval=15s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:8080/health || exit 1

ENTRYPOINT ["/app/server"]
```

**패키지 설치**

| 패키지 | 필요 이유 |
|--------|-----------|
| `ca-certificates` | HTTPS 아웃바운드 요청 시 TLS CA 인증서 체인 검증 |
| `tzdata` | 타임존 데이터 — `time.LoadLocation()` 사용 시 필요 |
| `--no-cache` | apk 인덱스 캐시를 이미지에 포함하지 않음 (이미지 크기 절약) |

**Non-root 실행 (보안)**

```
addgroup -g 1001 -S appgroup   # 시스템 그룹 (GID 1001)
adduser  -u 1001 -S appuser    # 시스템 유저 (UID 1001), 홈 디렉토리/비밀번호 없음
```

- `-S` 플래그: 시스템 계정 (no login shell, no home dir)
- UID/GID `1001` 고정: Kubernetes의 `securityContext.runAsUser: 1001`과 일치시켜 파드 보안 정책 준수
- `--chown=appuser:appgroup`: 바이너리 소유권을 빌드 시점에 설정

**HEALTHCHECK 설정**

```dockerfile
HEALTHCHECK --interval=15s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:8080/health || exit 1
```

| 파라미터 | 값 | 의미 |
|----------|-----|------|
| `--interval` | 15s | 헬스체크 실행 주기 |
| `--timeout` | 5s | 응답 대기 최대 시간 |
| `--start-period` | 10s | 컨테이너 시작 후 첫 체크까지 유예 기간 |
| `--retries` | 3 | 연속 실패 횟수가 3회일 때 `unhealthy` 판정 |

- `wget -qO-`: curl 대신 alpine 기본 포함 wget 사용 (이미지 크기 절약)
- `/health` 엔드포인트: `main.go`의 `registerSystemRoutes`에 정의됨

**ENTRYPOINT vs CMD**

`ENTRYPOINT ["/app/server"]` (exec 형식) 사용:
- PID 1로 직접 실행 → OS 시그널(`SIGTERM`, `SIGINT`)이 애플리케이션에 직접 전달
- shell 형식(`CMD server`)은 sh를 PID 1로 실행하여 시그널이 자식 프로세스에 전달되지 않는 문제 발생

---

## 4. 엔드포인트 목록

| 메서드 | 경로 | 인증 | 설명 |
|--------|------|------|------|
| GET | `/health` | 없음 | 서버 생존 + Redis 상태 |
| GET | `/ready` | 없음 | 트래픽 수신 준비 여부 |
| GET | `/ws` | JWT (내부) | WebSocket 연결 업그레이드 |
| POST | `/api/auth/dev-login` | 없음 (dev만) | 개발용 JWT 발급 |
| POST | `/api/rooms` | JWT | 방 생성 |
| GET | `/api/rooms` | JWT | 방 목록 |
| GET | `/api/rooms/:id` | JWT | 방 단건 조회 |
| POST | `/api/rooms/:id/join` | JWT | 방 입장 |
| POST | `/api/rooms/:id/leave` | JWT | 방 퇴장 |
| POST | `/api/rooms/:id/start` | JWT | 게임 시작 |
| DELETE | `/api/rooms/:id` | JWT | 방 삭제 |
| GET | `/api/games/:id` | JWT | 게임 상태 조회 |
| POST | `/api/games/:id/place` | JWT | 타일 배치 |
| POST | `/api/games/:id/confirm` | JWT | 턴 확정 |
| POST | `/api/games/:id/draw` | JWT | 타일 뽑기 |
| POST | `/api/games/:id/reset` | JWT | 턴 초기화 |

---

## 5. 의존성 요약

| 라이브러리 | 버전 | 용도 |
|------------|------|------|
| `gin-gonic/gin` | latest | HTTP 라우터 |
| `redis/go-redis` | v9 | Redis 클라이언트 |
| `go.uber.org/zap` | latest | 구조화 로깅 |
| `gorm.io/gorm` | latest | ORM (PostgreSQL) |
| Go | 1.24 | 언어 런타임 |
| alpine | 3.21 | 런타임 베이스 이미지 |

---

## 6. 운영 고려사항

### 환경변수

`config.Load()`가 읽는 주요 설정:

| 키 | 예시 | 설명 |
|----|------|------|
| `SERVER_PORT` | `8080` | 서버 포트 |
| `SERVER_MODE` | `release` | gin 모드 (debug/release/test) |
| `APP_ENV` | `prod` | 개발 전용 엔드포인트 활성화 여부 |
| `JWT_SECRET` | `(비밀값)` | JWT 서명 키 |
| `DB_DSN` | `postgres://...` | PostgreSQL 연결 문자열 |
| `REDIS_ADDR` | `redis:6379` | Redis 주소 |

### 프로덕션 체크리스트

- [ ] `APP_ENV`를 `prod`로 설정 (dev-login 엔드포인트 비활성화)
- [ ] `SERVER_MODE`를 `release`로 설정 (gin 디버그 로그 비활성화)
- [ ] `JWT_SECRET`을 강력한 랜덤 값으로 설정
- [ ] Redis가 가용한 환경에서 실행 (in-memory fallback은 단일 서버만 지원)
- [ ] Kubernetes 사용 시 `securityContext.runAsUser: 1001` 설정
- [ ] `/health`를 `livenessProbe`, `/ready`를 `readinessProbe`로 설정

### 이미지 크기 최적화 효과

| 구성 요소 | 절약 내용 |
|-----------|-----------|
| 멀티스테이지 빌드 | Go 툴체인(~300MB) 제외 |
| CGO 비활성화 | glibc, gcc 불필요 |
| `-s -w` 플래그 | 바이너리 크기 약 30% 감소 |
| alpine 베이스 | ubuntu 대비 약 80% 이미지 크기 절감 |
| `--no-cache` apk | apk 인덱스 캐시 제외 |

---

*문서 생성일: 2026-03-17*
