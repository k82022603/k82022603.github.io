---
title: "Cloud Native Kubernetes 구축 가이드"
date: 2026-05-26 19:00:00 +0900
categories: [TechStack,  Architecture]
mermaid: [True]
tags: [Kubernetes,  Ingress,  istio-service-mesh,  RummiArena,  Claude.write]
---


> **대상**: 운영 환경 Kubernetes 클러스터 구축 담당자
>
> **샘플 프로젝트**: [RummiArena](https://github.com/k82022603/RummiArena) (루미큐브 멀티 LLM 대전 플랫폼)
>
> **버전**: 1.0 (2026-05-26)

---

## 목차

1. [이 가이드의 구성](#1-이-가이드의-구성)
2. [샘플 프로젝트: RummiArena 소개](#2-샘플-프로젝트-rummiarena-소개)
3. [운영 환경 클러스터 설계 원칙](#3-운영-환경-클러스터-설계-원칙)
4. [HTTPS 전략: 인증서 관리와 TLS 종료](#4-https-전략-인증서-관리와-tls-종료)
5. [Ingress 전략 비교 및 선택](#5-ingress-전략-비교-및-선택)
6. [Istio Service Mesh 구성](#6-istio-service-mesh-구성)
7. [내부 통신에서 HTTPS는 반드시 필요한가](#7-내부-통신에서-https는-반드시-필요한가)
8. [RummiArena 실제 구성 사례](#8-rummiarena-실제-구성-사례)
9. [운영 체크리스트](#9-운영-체크리스트)
10. [별첨 A: OpenShift vs Vanilla Kubernetes 비교](#별첨-a-openshift-vs-vanilla-kubernetes-비교)
11. [별첨 B: 주요 YAML 예제 모음](#별첨-b-주요-yaml-예제-모음)

---

## 1. 이 가이드의 구성

이 가이드는 Cloud Native 애플리케이션을 Kubernetes 위에서 운영 환경 수준으로 구축하는 실무 지침을 담는다. 단순한 "설치법"이 아니라 **왜 그렇게 설계하는가**를 중심에 두고 작성했다.

### 샘플로 RummiArena를 선택한 이유

RummiArena는 다음 요소를 동시에 갖춘 적절한 샘플이다.

| 요소 | 내용 |
|------|------|
| 다중 서비스 | Go 백엔드, NestJS AI 어댑터, Next.js 프론트엔드, PostgreSQL, Redis (5개 서비스) |
| WebSocket | 실시간 대전이므로 HTTP → WebSocket 업그레이드 처리 필요 |
| 외부 API 연동 | LLM API (DeepSeek, OpenAI, Anthropic) — 장시간 연결, 타임아웃 설계 필수 |
| 서비스 메시 | Istio selective mesh (game-server + ai-adapter만 사이드카) |
| GitOps | ArgoCD + GitLab CI 완전 자동화 |
| 보안 등급 차이 | 퍼블릭 프론트엔드 vs 내부 전용 AI 어댑터 |

이 가이드의 모든 예제는 RummiArena의 실제 YAML을 기반으로 작성됐다.

---

## 2. 샘플 프로젝트: RummiArena 소개

### 2.1 아키텍처 개요

```mermaid
graph TB
    User["사용자 (HTTPS)"]

    subgraph Ingress["Ingress Layer"]
        IC["Ingress Controller\n(Traefik / NGINX / Gateway API)"]
        Cert["cert-manager\n(TLS 인증서)"]
    end

    subgraph Mesh["Service Mesh (Istio)"]
        direction LR
        GS["game-server\n(sidecar ✅)"]
        AA["ai-adapter\n(sidecar ✅ / STRICT)"]
    end

    subgraph Storage["Storage"]
        PG["PostgreSQL"]
        Redis["Redis"]
    end

    subgraph External["External APIs"]
        LLM["DeepSeek / OpenAI\n/ Anthropic"]
    end

    FE["frontend\n(Next.js, sidecar ❌)"]

    User --> IC
    IC --> FE
    IC --> GS
    FE --> GS
    GS <-->|"mTLS (mesh)"| AA
    GS --> PG
    GS --> Redis
    AA --> LLM
```

### 2.2 서비스별 특성

| 서비스 | 프로토콜 | 외부 노출 | Istio 사이드카 | 특이사항 |
|--------|----------|----------|--------------|---------|
| frontend | HTTP/HTTPS | ✅ 공개 | ❌ | SSR + 정적 자산 |
| game-server | HTTP + WS | ✅ 공개 | ✅ PERMISSIVE | WebSocket 장시간 연결 |
| ai-adapter | HTTP | ❌ 내부 전용 | ✅ STRICT | LLM 호출 최대 700s |
| PostgreSQL | TCP | ❌ 내부 전용 | ❌ | 영속 데이터 |
| Redis | TCP | ❌ 내부 전용 | ❌ | 게임 상태 |

### 2.3 현재 운영 구성 (실제)

```mermaid
graph LR
    User["사용자"] -->|"NodePort :30000"| FE["frontend"]
    User -->|"NodePort :30080"| GS["game-server"]
    GS <-->|"mTLS ClusterIP"| AA["ai-adapter"]
    GS --> PG["PostgreSQL"]
    GS --> Redis["Redis"]
    ArgoCD["ArgoCD"] -->|"GitOps sync"| GS
    ArgoCD --> AA
```

> **참고**: RummiArena는 연구용 단일 노드 클러스터로 현재 NodePort를 사용 중이다.
> 운영 환경에서는 반드시 Ingress Controller를 통한 HTTPS 단일 진입점을 사용한다.

---

## 3. 운영 환경 클러스터 설계 원칙

### 3.1 단일 진입점 (Single Entry Point)

운영 환경에서는 외부 트래픽이 반드시 **하나의 Ingress 레이어**를 통과해야 한다.

```
인터넷 → LoadBalancer (클라우드) or MetalLB (베어메탈)
       → Ingress Controller (Traefik / NGINX / Gateway API)
       → TLS 종료 (cert-manager 자동 발급)
       → 내부 서비스 (HTTP 평문 또는 mTLS)
```

이유:
- TLS 관리 중앙화 (인증서 갱신 자동화)
- 인증/접근 제어 단일 지점
- Rate limiting, WAF 적용 용이
- 관찰 가능성 (모든 요청 로그 집중)

### 3.2 네임스페이스 전략

RummiArena의 실제 네임스페이스 구성을 기반으로 한 권장 패턴:

```
ingress-system      # Ingress Controller 전용
cert-manager        # TLS 인증서 관리
istio-system        # Service Mesh 컨트롤 플레인
argocd              # GitOps
monitoring          # Prometheus, Grafana
{app-name}          # 애플리케이션 (예: rummikub)
sonarqube           # 코드 품질 (CI 인프라)
```

**원칙**: 인프라 컴포넌트는 애플리케이션 네임스페이스와 분리한다. Istio Peer Authentication 정책을 네임스페이스 단위로 적용하기 때문에 혼용하면 정책 범위가 복잡해진다.

### 3.3 리소스 계층

```mermaid
graph TB
    subgraph "클러스터 수준"
        CRQ["ClusterResourceQuota\n(전체 쿼터)"]
        NP["NetworkPolicy\n(기본 deny-all)"]
        PSA["PodSecurity Admission\n(restricted)"]
    end

    subgraph "네임스페이스 수준"
        RQ["ResourceQuota\n(네임스페이스 쿼터)"]
        LR["LimitRange\n(Pod 기본값)"]
    end

    subgraph "워크로드 수준"
        REQ["requests/limits\n(컨테이너별)"]
        SA["ServiceAccount\n(최소 권한)"]
    end
```

---

## 4. HTTPS 전략: 인증서 관리와 TLS 종료

운영 환경에서 사용자는 HTTPS로만 접근한다는 것을 전제로 한다.

### 4.1 cert-manager 개요

cert-manager(v1.16.4+)는 Kubernetes 네이티브 인증서 관리 도구다. Let's Encrypt, Vault, 자체 CA 등 다양한 발급자를 지원하며, 만료 30일 전 자동 갱신한다.

```mermaid
sequenceDiagram
    participant IC as Ingress Controller
    participant CM as cert-manager
    participant LE as Let's Encrypt
    participant DNS as DNS Provider

    IC->>CM: TLS 인증서 필요 (Ingress annotation)
    CM->>LE: ACME 챌린지 요청
    alt HTTP-01 (공개 도메인)
        LE->>IC: /.well-known/acme-challenge/{token} 검증
    else DNS-01 (와일드카드 / 프라이빗)
        LE->>DNS: TXT 레코드 생성 요청
        CM->>DNS: _acme-challenge.{domain} TXT 생성
        LE->>DNS: TXT 레코드 확인
    end
    LE-->>CM: 인증서 발급
    CM-->>IC: Secret에 tls.crt / tls.key 저장
```

### 4.2 HTTP-01 vs DNS-01 선택 기준

| 항목 | HTTP-01 | DNS-01 |
|------|---------|--------|
| 와일드카드 인증서 | ❌ 불가 | ✅ 가능 (`*.example.com`) |
| 프라이빗 클러스터 | ❌ 80포트 공개 필요 | ✅ 가능 |
| 설정 복잡도 | 낮음 | DNS 프로바이더 설정 필요 |
| 권장 상황 | 단일 도메인, 퍼블릭 클러스터 | 멀티 서비스 서브도메인, 프라이빗 |
| 포트 요건 | 80/443 공개 | DNS 권한만 필요 |

**RummiArena 적용 시 권장**: 여러 서브도메인 (`game.example.com`, `api.example.com`, `argocd.example.com`) 을 사용하는 경우 와일드카드 + DNS-01.

### 4.3 cert-manager 설치 및 ClusterIssuer 설정

```yaml
# cert-manager 설치 (Helm)
# helm repo add jetstack https://charts.jetstack.io
# helm install cert-manager jetstack/cert-manager \
#   --namespace cert-manager --create-namespace \
#   --set crds.enabled=true

---
# ClusterIssuer (Let's Encrypt 프로덕션)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: ops@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
    # HTTP-01: 퍼블릭 서비스용
    - http01:
        ingress:
          ingressClassName: traefik
      selector:
        dnsNames:
        - "game.example.com"
    # DNS-01: 와일드카드 / 내부 서비스용
    - dns01:
        route53:  # 또는 cloudflare, google 등
          region: ap-northeast-2
          accessKeyID: AKIAXXXXXXXX
          secretAccessKeySecretRef:
            name: route53-credentials
            key: secret-access-key
      selector:
        dnsNames:
        - "*.example.com"
```

### 4.4 TLS 종료 위치별 전략

```mermaid
graph LR
    subgraph "전략 1: Edge TLS 종료"
        U1["사용자 HTTPS"] --> IC1["Ingress Controller\n(TLS 종료)"] --> S1["서비스 HTTP\n(평문)"]
    end

    subgraph "전략 2: Passthrough + 서비스 TLS"
        U2["사용자 HTTPS"] --> IC2["Ingress Controller\n(TCP 패스스루)"] --> S2["서비스 HTTPS\n(자체 TLS)"]
    end

    subgraph "전략 3: Re-encrypt (mTLS)"
        U3["사용자 HTTPS"] --> IC3["Ingress Controller\n(TLS 종료)"] -->|"mTLS"| S3["서비스 mTLS\n(Istio)"]
    end
```

| 전략 | 보안 수준 | 복잡도 | 권장 상황 |
|------|----------|--------|---------|
| Edge 종료 | 중간 | 낮음 | Istio 미사용, 트러스트 경계 = Ingress |
| Passthrough | 높음 | 중간 | gRPC, DB 직접 노출, 엔드투엔드 TLS |
| Re-encrypt (mTLS) | 최고 | 높음 | Zero-trust, PCI-DSS, 금융 / 의료 |

---

## 5. Ingress 전략 비교 및 선택

### 5.1 현황 개요 (2026 기준)

```mermaid
graph LR
    Legacy["Legacy Ingress API\n(networking.k8s.io/v1)"]
    New["Gateway API\n(gateway.networking.k8s.io)"]

    Legacy -->|"마이그레이션 권장"| New

    subgraph "Legacy 구현체"
        NGINX["ingress-nginx\n(⚠️ EOL 2026-03)"]
        Traefik1["Traefik v3\n(✅ 활발히 유지)"]
        HAProxy["HAProxy Ingress\n(✅ 유지)"]
    end

    subgraph "Gateway API 구현체"
        Envoy["Envoy Gateway\n(✅ CNCF 공식)"]
        Traefik2["Traefik v3\n(Gateway API 지원)"]
        Istio["Istio Gateway\n(✅ Mesh 통합)"]
        NGINXGW["NGINX Gateway Fabric\n(✅ 신규)"]
    end
```

### 5.2 ingress-nginx (커뮤니티) — EOL 주의

> **⚠️ 중요**: 커뮤니티 `ingress-nginx`는 2026년 3월 공식 EOL 선언. 보안 패치 중단. 현재 K8s 클러스터의 약 50%가 이를 사용 중으로, 마이그레이션이 업계의 주요 과제다.

**현재 선택지**:
- **NGINX Inc.** (`nginx-ingress` / `nginx-gateway-fabric`): 상용 지원 + Gateway API 지원 신규 구현체
- **Traefik**: ingress-nginx 대체제로 가장 주목받는 옵션
- **Envoy Gateway**: CNCF 공식 Gateway API 레퍼런스 구현

```yaml
# ingress-nginx 사용 예시 (레거시 — 마이그레이션 필요)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rummiarena
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"    # WebSocket
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - game.example.com
    secretName: rummiarena-tls
  rules:
  - host: game.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 3000
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: game-server
            port:
              number: 8080
      - path: /ws
        pathType: Prefix
        backend:
          service:
            name: game-server
            port:
              number: 8080
```

### 5.3 Traefik v3

Traefik은 GitOps 친화적이며, Kubernetes CRD(`IngressRoute`)와 표준 `Ingress` 모두 지원한다. 2026년 현재 25,000+ GitHub 스타를 보유한 가장 활발한 커뮤니티 중 하나다.

**주요 특징**:
- 동적 설정 재로드 (재시작 없음)
- 자동 TLS (Let's Encrypt 내장)
- 미들웨어 체인 (인증, Rate limit, Circuit breaker)
- Dashboard UI 기본 제공
- WebSocket 기본 지원

```yaml
# Traefik IngressRoute 예시 (RummiArena 기반)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: rummiarena-web
  namespace: rummikub
spec:
  entryPoints:
    - websecure       # HTTPS (443)
  routes:
    # 프론트엔드
    - match: Host(`game.example.com`) && PathPrefix(`/`)
      kind: Rule
      services:
        - name: frontend
          port: 3000
      middlewares:
        - name: security-headers

    # API (game-server REST)
    - match: Host(`game.example.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: game-server
          port: 8080
      middlewares:
        - name: rate-limit-api

    # WebSocket (실시간 게임)
    - match: Host(`game.example.com`) && PathPrefix(`/ws`)
      kind: Rule
      services:
        - name: game-server
          port: 8080
          # Traefik은 Connection: Upgrade 자동 감지
  tls:
    certResolver: letsencrypt    # 자동 발급
---
# 보안 헤더 미들웨어
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: security-headers
  namespace: rummikub
spec:
  headers:
    forceSTSHeader: true
    stsSeconds: 31536000
    stsIncludeSubdomains: true
    contentTypeNosniff: true
    browserXssFilter: true
    frameDeny: true
---
# Rate limit (API 보호)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-api
  namespace: rummikub
spec:
  rateLimit:
    average: 100
    burst: 200
    period: 1m
```

**Traefik 설치 (Helm)**:

```bash
helm repo add traefik https://traefik.github.io/charts
helm install traefik traefik/traefik \
  --namespace traefik --create-namespace \
  --set providers.kubernetesCRD.enabled=true \
  --set providers.kubernetesIngress.enabled=true \
  --set service.type=LoadBalancer \
  --set logs.general.level=WARN
```

> **RummiArena 구성 참고**: `helm/traefik/` 디렉터리에 IngressRoute, Middleware, TLSOption, ServersTransport YAML이 있다. 현재는 적용되지 않은 상태(NodePort 운영 중)이나 운영 환경 전환 시 바로 사용 가능한 구조다.

### 5.4 Gateway API (차세대 표준)

Kubernetes SIG-Network에서 공식 Ingress 후속으로 지정한 API. 역할 분리(인프라/개발), 멀티-테넌트, L4/L7 통합이 핵심이다.

```mermaid
classDiagram
    class GatewayClass {
        +controller: envoygateway.io/gateway
        클러스터 관리자 소유
    }
    class Gateway {
        +listeners: HTTPS 443
        +tls: letsencrypt-prod
        인프라 팀 소유
    }
    class HTTPRoute {
        +rules: path-based routing
        개발 팀 소유 (네임스페이스별)
    }

    GatewayClass --> Gateway
    Gateway --> HTTPRoute
```

```yaml
# GatewayClass (인프라 팀)
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
---
# Gateway (인프라 팀)
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: rummiarena-gw
  namespace: ingress-system
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  gatewayClassName: envoy-gateway
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: rummiarena-tls
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            gateway-access: "true"
---
# HTTPRoute (개발 팀 — rummikub 네임스페이스)
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: rummiarena
  namespace: rummikub
spec:
  parentRefs:
  - name: rummiarena-gw
    namespace: ingress-system
  hostnames:
  - "game.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /ws
    backendRefs:
    - name: game-server
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: game-server
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: frontend
      port: 3000
```

### 5.5 Ingress 전략 최종 비교표

| 항목 | ingress-nginx | Traefik v3 | Envoy Gateway | Istio Gateway |
|------|-------------|-----------|--------------|--------------|
| **상태** | ⚠️ EOL 2026-03 | ✅ 활발 | ✅ CNCF 공식 | ✅ Mesh 통합 |
| **API 지원** | Ingress | Ingress + CRD + Gateway API | Gateway API 네이티브 | Istio CRD + Gateway API |
| **TLS 자동화** | cert-manager 연동 | 내장 + cert-manager | cert-manager 연동 | cert-manager 연동 |
| **WebSocket** | annotation 필요 | 기본 지원 | 기본 지원 | 기본 지원 |
| **미들웨어** | annotation 기반 | 풍부한 CRD | EnvoyFilter | VirtualService |
| **Istio 공존** | PERMISSIVE 모드 필요 | PERMISSIVE 모드 필요 | PERMISSIVE 모드 필요 | 자연스러운 통합 |
| **학습 곡선** | 낮음 | 중간 | 높음 | 높음 |
| **운영 규모 적합** | 소-중 | 중-대 | 대 | 대 (Mesh 이미 사용 시) |
| **신규 구축 권장** | ❌ 마이그레이션 필요 | ✅ | ✅ | ✅ (Istio 사용 시) |

**선택 지침**:
- Istio를 사용하지 않는 경우: **Traefik** (운영 편의성) 또는 **Envoy Gateway** (Gateway API 표준 준수)
- Istio를 사용하는 경우: **Istio Gateway** (Mesh와 통합) + HTTPRoute
- 기존 ingress-nginx 사용 중: **Traefik** 으로 마이그레이션 우선 검토

---

## 6. Istio Service Mesh 구성

### 6.1 Service Mesh가 필요한 시점

Service Mesh는 모든 환경에 필요하지 않다. 도입 기준을 명확히 해야 한다.

```
도입 권장:
✅ 서비스 간 암호화 (mTLS) 가 규정/컴플라이언스 요건
✅ 10개 이상의 서비스가 복잡하게 상호작용
✅ 세밀한 트래픽 제어 (카나리, 서킷 브레이커, 리트라이 정책)
✅ 서비스 간 관찰 가능성 (분산 트레이싱, 메트릭)
✅ Zero-trust 네트워크 모델

도입 불필요:
❌ 서비스 수 5개 이하, 단순 구조
❌ 팀 규모 소규모 (운영 부담 > 이점)
❌ 레이턴시에 극도로 민감한 서비스 (사이드카 오버헤드 ~1-5ms)
```

### 6.2 Istio 아키텍처

```mermaid
graph TB
    subgraph "컨트롤 플레인 (istiod)"
        Pilot["Pilot\n(xDS 설정 배포)"]
        Citadel["Citadel\n(인증서 관리)"]
        Galley["Galley\n(설정 검증)"]
    end

    subgraph "데이터 플레인"
        subgraph "game-server Pod"
            GS_App["app 컨테이너"]
            GS_Proxy["envoy proxy\n(sidecar)"]
        end
        subgraph "ai-adapter Pod"
            AA_App["app 컨테이너"]
            AA_Proxy["envoy proxy\n(sidecar)"]
        end
        subgraph "frontend Pod (사이드카 없음)"
            FE_App["app 컨테이너"]
        end
    end

    Pilot -->|"xDS"| GS_Proxy
    Pilot -->|"xDS"| AA_Proxy
    Citadel -->|"인증서"| GS_Proxy
    Citadel -->|"인증서"| AA_Proxy
    GS_Proxy <-->|"mTLS"| AA_Proxy
```

### 6.3 Selective Mesh 전략 (RummiArena 실제 적용)

전체 서비스에 사이드카를 주입하면 약 250Mi 추가 메모리가 필요하다. RummiArena는 **보안이 중요한 서비스만 선택적으로 사이드카를 주입**한다.

```yaml
# 사이드카 주입 활성화 (네임스페이스 레벨)
# kubectl label namespace rummikub istio-injection=enabled

# 특정 Pod만 제외 (frontend, postgres, redis)
# Pod spec에 annotation 추가:
metadata:
  annotations:
    sidecar.istio.io/inject: "false"   # 이 Pod는 메시 제외
```

| 서비스 | 사이드카 | 이유 |
|--------|---------|------|
| game-server | ✅ | LLM 응답 수신, 내부 통신 보호 필요 |
| ai-adapter | ✅ | LLM API 키 보유, 외부 노출 금지 |
| frontend | ❌ | 퍼블릭 서비스, Ingress가 이미 TLS 처리 |
| postgres | ❌ | DB 자체 인증으로 충분, 오버헤드 불필요 |
| redis | ❌ | 동일 |

### 6.4 mTLS 정책 계층

```mermaid
graph TB
    subgraph "네임스페이스 정책 (기본값)"
        NS_PERM["PeerAuthentication\nmode: PERMISSIVE\n사이드카 없는 Pod도 접근 가능"]
    end

    subgraph "서비스별 오버라이드"
        AA_STRICT["PeerAuthentication (ai-adapter)\nmode: STRICT\n사이드카만 접근 가능"]
    end

    GS["game-server\n(사이드카 ✅)"] -->|"mTLS — 허용"| AA["ai-adapter"]
    FE["frontend\n(사이드카 ❌)"] -->|"평문 — 차단됨"| AA
    IC["Ingress Controller\n(사이드카 ❌)"] -->|"평문 — 허용 (PERMISSIVE)"| GS
```

```yaml
# 네임스페이스 기본: PERMISSIVE (Ingress Controller 접근 허용)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: rummikub
spec:
  mtls:
    mode: PERMISSIVE
---
# ai-adapter: STRICT (사이드카만 접근 가능)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: ai-adapter-strict
  namespace: rummikub
spec:
  selector:
    matchLabels:
      app: ai-adapter
  mtls:
    mode: STRICT
```

### 6.5 트래픽 관리 (VirtualService + DestinationRule)

**RummiArena의 실제 ai-adapter 설정**:

```yaml
# VirtualService: LLM 장시간 호출을 위한 타임아웃 설정
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: ai-adapter
  namespace: rummikub
spec:
  hosts:
  - ai-adapter
  http:
  - timeout: 1010s      # AI_ADAPTER_TIMEOUT_SEC(1000) + 10s 버퍼
    retries:
      attempts: 1        # LLM 비용 때문에 재시도 1회로 제한
      perTryTimeout: 1010s
      retryOn: "5xx,reset"
    route:
    - destination:
        host: ai-adapter
---
# DestinationRule: 서킷 브레이커
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: ai-adapter
  namespace: rummikub
spec:
  host: ai-adapter
  trafficPolicy:
    outlierDetection:
      consecutive5xxErrors: 3      # 연속 3회 5xx → 격리
      interval: 60s                 # 평가 주기 (LLM 700s 응답 고려해 여유있게)
      baseEjectionTime: 120s        # 2분 격리 후 재시도
      maxEjectionPercent: 50
      splitExternalLocalOriginErrors: true
```

> **타임아웃 체인 설계 원칙** (RummiArena ADR #41):
> ```
> script_ws(770s) > gs_ctx(760s) > http_client(760s) > istio_vs(1010s) > adapter_floor(1000s) > llm_vendor
> ```
> 각 레이어의 타임아웃은 하위 레이어보다 반드시 크게 설정한다. 역전 시 정상 응답이 타임아웃으로 오분류된다.

### 6.6 Istio Ambient Mode (사이드카 없는 Mesh)

Istio v1.24 (2024년 11월)에서 Ambient Mode GA. 사이드카 없이 Service Mesh 기능을 제공한다.

```mermaid
graph TB
    subgraph "Sidecar Mode (현재)"
        Pod1_App["앱 컨테이너"] --- Pod1_Proxy["envoy sidecar\n(Pod 내부)"]
        Pod2_App["앱 컨테이너"] --- Pod2_Proxy["envoy sidecar\n(Pod 내부)"]
    end

    subgraph "Ambient Mode (신규)"
        Pod3_App["앱 컨테이너\n(사이드카 없음)"]
        Pod4_App["앱 컨테이너\n(사이드카 없음)"]
        ZT["ztunnel\n(노드당 1개, L4 mTLS)"]
        WP["waypoint proxy\n(서비스당 1개, L7 정책)"]
        Pod3_App --> ZT --> WP
        Pod4_App --> ZT
    end
```

| 항목 | Sidecar Mode | Ambient Mode |
|------|-------------|-------------|
| 배포 단위 | Pod마다 envoy | 노드마다 ztunnel + 서비스마다 waypoint |
| 메모리 오버헤드 | Pod당 ~50-100Mi | 노드당 ztunnel ~50Mi (전체적으로 적음) |
| 롤링 업데이트 | Pod 재시작 필요 | 노드 수준 무중단 |
| L7 정책 | 항상 가능 | waypoint 배포 시 가능 |
| 성숙도 | 검증된 | GA이나 신규 (2024-11) |

**마이그레이션 전략**: 신규 클러스터는 Ambient Mode 검토. 기존 Sidecar Mode는 안정적이므로 유지.

---

## 7. 내부 통신에서 HTTPS는 반드시 필요한가

이 질문은 "트러스트 모델"의 문제다. 답은 환경과 요구사항에 따라 다르다.

### 7.1 세 가지 관점

**관점 1: 클러스터 내부는 신뢰할 수 있다 (Plain HTTP 허용)**

```
전제: Kubernetes NetworkPolicy로 Pod 간 통신이 제어되고,
      클러스터 자체가 물리적으로 격리된 환경.

근거:
- 클러스터 내부 공격자 = 이미 Pod 탈취 = 더 큰 문제
- TLS 오버헤드 (CPU, 레이턴시) 회피
- 단순한 디버깅 환경

적합 환경:
- 소규모 단일 팀 프로젝트
- 연구/개발 환경 (RummiArena 현재 구성)
- 클라우드 플랫폼 자체 암호화를 신뢰하는 경우
```

**관점 2: Zero-Trust — 내부도 암호화 필수**

```
전제: "네트워크 위치는 신뢰의 근거가 아니다."
      컨테이너 탈취, 악의적 내부자, 측면 이동(lateral movement) 대비.

근거:
- NIST SP 800-207 Zero Trust Architecture
- PCI-DSS 4.0: 전송 중 데이터 암호화 (클러스터 내부 포함)
- HIPAA: PHI 데이터 암호화 의무
- 실제 클러스터 침해 사례: Pod 탈취 후 평문 내부 통신 도청

적합 환경:
- 금융, 의료, 공공 등 규정 적용 대상
- 멀티 테넌트 클러스터
- 민감 데이터 처리 서비스
```

**관점 3: 실용적 중간 지점 (L4 mTLS만으로 충분)**

```
Istio Ambient Mode의 ztunnel이 제공하는 L4 mTLS:
- 전송 암호화 + 상호 인증 제공
- L7 정책(헤더 기반 인증 등)은 불필요하면 생략
- waypoint 없이 ztunnel만으로 기본 zero-trust 달성

이 접근이 유효한 경우:
- 서비스 간 인증은 필요하지만 세밀한 L7 정책은 불필요
- 성능과 보안의 균형점
```

### 7.2 RummiArena의 선택과 이유

| 통신 구간 | 프로토콜 | 이유 |
|----------|----------|------|
| 사용자 → Ingress | HTTPS | 공개 인터넷, 필수 |
| Ingress → game-server | HTTP (PERMISSIVE) | Ingress Controller가 사이드카 없음 |
| game-server → ai-adapter | mTLS (Istio) | LLM API 키 보호, STRICT 정책 |
| game-server → PostgreSQL | TCP (평문) | 동일 클러스터 내부, 연구용 |
| game-server → Redis | TCP (평문) | 동일 |

**결론**: 완전한 내부 HTTPS는 연구 환경에서 과도한 복잡도. 핵심 보안 경계(ai-adapter)만 STRICT mTLS 적용하는 Selective 전략이 실용적이다.

### 7.3 결정 프레임워크

```mermaid
flowchart TB
    Start["내부 통신 암호화 검토"] --> Q1{규정 준수 대상인가?\nPCI/HIPAA/금융}
    Q1 -->|Yes| MandatoryTLS["전 구간 mTLS 필수\n(Istio STRICT + 전체 사이드카)"]
    Q1 -->|No| Q2{민감 데이터가\n내부 통신에 있는가?}
    Q2 -->|Yes| SelectiveTLS["핵심 서비스 STRICT mTLS\n(RummiArena 방식)"]
    Q2 -->|No| Q3{멀티 테넌트\n클러스터인가?}
    Q3 -->|Yes| NetworkPolicy["NetworkPolicy + L4 mTLS\n(Ambient Mode 검토)"]
    Q3 -->|No| PlainHTTP["내부 HTTP 허용\n(Ingress에서 TLS 종료)"]
```

---

## 8. RummiArena 실제 구성 사례

### 8.1 운영 환경 전환 시나리오

현재 RummiArena는 연구용 NodePort 구성이다. 운영 환경으로 전환하면:

```mermaid
graph TB
    subgraph "현재 (연구용)"
        NP_FE["NodePort :30000"] --> FE1["frontend"]
        NP_GS["NodePort :30080"] --> GS1["game-server"]
    end

    subgraph "운영 환경 전환"
        LB["LoadBalancer / MetalLB"] --> Traefik["Traefik Ingress Controller"]
        Traefik -->|"TLS 종료"| FE2["frontend"]
        Traefik -->|"TLS 종료"| GS2["game-server"]
        GS2 <-->|"mTLS (Istio STRICT)"| AA2["ai-adapter"]
        CM["cert-manager"] -->|"인증서 발급"| Traefik
    end
```

### 8.2 단계별 구축 순서

```mermaid
gantt
    title RummiArena 운영 환경 구축 순서
    dateFormat  X
    axisFormat  %s단계

    section 기반 인프라
    Kubernetes 클러스터 (멀티노드)  :done, 0, 1
    cert-manager 설치               :done, 1, 2
    Istio 설치 (istiod)            :done, 2, 3

    section Ingress
    Traefik 또는 Gateway API 설치  :active, 3, 4
    DNS 설정 + TLS 인증서 발급     :4, 5

    section 애플리케이션
    Helm Chart 배포 (5개 서비스)   :5, 7
    Istio PeerAuthentication 적용  :6, 7

    section 검증
    E2E 테스트 + 헬스체크          :7, 8
    보안 스캔 (Trivy)              :7, 8
```

### 8.3 실제 배포 커맨드 순서

```bash
# 1. cert-manager 설치
helm repo add jetstack https://charts.jetstack.io && helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set crds.enabled=true

# 2. Istio 설치 (istioctl 권장)
istioctl install --set profile=default -y
kubectl label namespace rummikub istio-injection=enabled

# 3. Traefik 설치
helm repo add traefik https://traefik.github.io/charts && helm repo update
helm install traefik traefik/traefik \
  --namespace traefik --create-namespace \
  -f helm/traefik/values.yaml

# 4. RummiArena 배포 (Helm umbrella)
helm upgrade --install rummiarena helm/ \
  --namespace rummikub --create-namespace \
  -f helm/values.yaml

# 5. Istio 정책 적용
kubectl apply -f istio/

# 6. 헬스 확인
bash helm/deploy.sh health
```

### 8.4 WebSocket 특수 처리

RummiArena의 실시간 대전은 WebSocket이 필수다. Ingress에서 WebSocket을 올바르게 처리하지 않으면 연결이 즉시 끊어진다.

**Traefik**: 기본 지원 (별도 설정 불필요, `Connection: Upgrade` 자동 처리)

**NGINX** (레거시 사용 시):
```yaml
annotations:
  nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
  nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
  nginx.ingress.kubernetes.io/configuration-snippet: |
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
```

**Gateway API**:
```yaml
# HTTPRoute에서 WebSocket 명시적 허용 (구현체마다 다름)
# Envoy Gateway: 기본 지원
# Traefik via Gateway API: 기본 지원
```

### 8.5 LLM 장시간 요청 처리

AI 어댑터는 DeepSeek Reasoner 호출 시 최대 700초를 기다린다. 이 요청이 타임아웃으로 끊기지 않도록 전체 체인을 설계해야 한다.

```
클라이언트 (브라우저) — 게임 결과는 WebSocket으로 받으므로 HTTP 타임아웃 무관
game-server (Go) — context timeout: 760s
HTTP client → ai-adapter: 760s
Istio VirtualService: 1010s (가장 길게 — 내부 체인 모두 커버)
ai-adapter 내부: 1000s
LLM API (DeepSeek): ~200-700s
```

**핵심**: Istio VirtualService 타임아웃이 항상 실제 LLM 응답 시간보다 길어야 한다.

---

## 9. 운영 체크리스트

### 9.1 보안

- [ ] Ingress Controller `insecure: false` (Traefik dashboard 외부 노출 금지)
- [ ] 모든 시크릿 Sealed Secrets 또는 Vault로 관리 (평문 Secret 금지)
- [ ] 민감 서비스 (ai-adapter) PeerAuthentication STRICT
- [ ] NetworkPolicy 기본 deny-all + 필요한 접근만 허용
- [ ] 이미지 digest pinning (`image:sha256:...`)
- [ ] Trivy 이미지 스캔 CI 게이트
- [ ] RBAC 최소 권한 (서비스별 개별 ServiceAccount)
- [ ] ArgoCD 접근 Internal 전용 (외부 노출 금지)
- [ ] PostgreSQL NodePort 제거 (ClusterIP만)

### 9.2 가용성

- [ ] 각 서비스 replica ≥ 2 (PodDisruptionBudget 설정)
- [ ] HPA (Horizontal Pod Autoscaler) 설정
- [ ] ResourceQuota + LimitRange (OOMKill 방지)
- [ ] Liveness / Readiness / Startup probe 모두 설정
- [ ] PVC ReadWriteMany 또는 StatefulSet (DB)

### 9.3 관찰 가능성

- [ ] Prometheus + Grafana (메트릭)
- [ ] Jaeger / Tempo (분산 트레이싱, Istio 연동)
- [ ] EFK / Loki (로그 집중화)
- [ ] Istio Kiali (서비스 토폴로지 시각화)
- [ ] AlertManager (장애 알림)

### 9.4 CI/CD

- [ ] 이미지 빌드 → 레지스트리 → GitOps 레포 업데이트 자동화
- [ ] ArgoCD auto-sync + self-heal 활성화
- [ ] 배포 전 Playbook (smoke test) 자동 실행
- [ ] 롤백 절차 문서화 (`helm rollback` 또는 ArgoCD UI)

---

## 별첨 A: OpenShift vs Vanilla Kubernetes 비교

OpenShift는 Red Hat이 만든 엔터프라이즈 Kubernetes 배포판이다. Kubernetes가 빌딩 블록이라면 OpenShift는 완성된 플랫폼에 가깝다.

### A.1 핵심 차이점 비교표

| 카테고리 | Vanilla Kubernetes | OpenShift (OCP) |
|---------|-------------------|----------------|
| **Ingress** | Ingress 리소스 (구현체 별도 설치) | **Route** 리소스 (내장 HAProxy Router) + Ingress 호환 |
| **TLS 관리** | cert-manager 수동 설치 필요 | Route에서 Edge/Passthrough/Re-encrypt 기본 지원 |
| **컨테이너 보안** | PodSecurity Admission (PSA) - 3단계 | **SCC (Security Context Constraints)** - 세밀한 제어 |
| **기본 사용자 권한** | root 허용 (설정에 따라) | **root 실행 기본 차단** (임의 UID 강제) |
| **레지스트리** | 외부 레지스트리 사용 | **내장 레지스트리** (OpenShift Image Registry) |
| **운영자 관리** | Helm, kubectl | **OLM (Operator Lifecycle Manager)** + OperatorHub |
| **빌드 시스템** | 없음 (외부 CI 사용) | **BuildConfig** + S2I (Source-to-Image) 내장 |
| **네트워크** | CNI 플러그인 선택 (Calico, Flannel 등) | **OVN-Kubernetes** 기본 (SDN 대체) |
| **스토리지** | StorageClass (CSI 드라이버 설치 필요) | 동일 + 내장 스토리지 솔루션 |
| **멀티클러스터** | 별도 구성 (Submariner, Cilium) | **ACM (Advanced Cluster Management)** 내장 |
| **콘솔** | 없음 (별도 설치) | **웹 콘솔** 기본 제공 (개발자/관리자 뷰) |
| **인증** | OIDC, RBAC (수동 설정) | OAuth 서버 내장 + LDAP/AD 통합 용이 |
| **업그레이드** | 수동 또는 kubeadm | **OTA 업그레이드** (Operator 기반 자동화) |
| **지원** | CNCF 생태계 (커뮤니티 + 상용) | Red Hat 공식 엔터프라이즈 지원 |
| **라이선스** | 오픈소스 (무료) | 상용 구독 (OCP), 무료 버전 (OKD) |
| **Kubernetes 버전 래그** | 없음 (최신) | 약 1-2 마이너 버전 래그 |

### A.2 Ingress: Route vs Ingress/Gateway API

```mermaid
graph LR
    subgraph "Vanilla K8s"
        IC["Ingress Controller\n(별도 설치 필요)"]
        IR["Ingress 리소스"]
        GW["Gateway API\n(미래 표준)"]
    end

    subgraph "OpenShift"
        Router["HAProxy Router\n(기본 설치됨)"]
        Route["Route 리소스\n(OpenShift 전용)"]
        ING["Ingress 리소스\n(호환 지원)"]
    end
```

**OpenShift Route 예시** (Traefik IngressRoute와 기능적으로 동일):

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: rummiarena-frontend
  namespace: rummikub
spec:
  host: game.example.com
  path: /
  to:
    kind: Service
    name: frontend
    weight: 100
  port:
    targetPort: 3000
  tls:
    termination: edge           # Edge TLS 종료 (cert-manager 불필요)
    certificate: |
      -----BEGIN CERTIFICATE-----
      ...
    key: |
      -----BEGIN PRIVATE KEY-----
      ...
    insecureEdgeTerminationPolicy: Redirect
```

### A.3 SCC vs PSA (컨테이너 보안)

**OpenShift SCC**는 Kubernetes PSA보다 훨씬 세밀한 제어를 제공한다.

```yaml
# Vanilla K8s: PodSecurity Admission (네임스페이스 레이블)
apiVersion: v1
kind: Namespace
metadata:
  name: rummikub
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted

---
# OpenShift SCC (훨씬 세밀한 제어)
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: rummiarena-scc
allowPrivilegedContainer: false
allowPrivilegeEscalation: false
runAsUser:
  type: MustRunAsRange      # 특정 UID 범위 강제
  uidRangeMin: 1000
  uidRangeMax: 65535
seLinuxContext:
  type: MustRunAs
fsGroup:
  type: MustRunAs
  ranges:
  - min: 1000
    max: 65535
volumes:
- configMap
- emptyDir
- persistentVolumeClaim
- secret
```

**실무 영향**: RummiArena를 OpenShift로 이관할 경우, Go 및 Node.js 컨테이너가 root로 실행되면 기본 SCC에 의해 **Pod 시작이 거부**된다. Dockerfile에서 non-root 사용자 설정이 필수다.

```dockerfile
# OpenShift 호환 Dockerfile 예시 (game-server)
FROM golang:1.23-alpine AS builder
# ... 빌드 ...

FROM alpine:3.21
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
USER 1001                    # ← OpenShift SCC 대응: non-root 필수
WORKDIR /app
COPY --from=builder /app/game-server .
ENTRYPOINT ["./game-server"]
```

### A.4 OLM vs Helm

| 항목 | Helm | OLM (OpenShift) |
|------|------|----------------|
| 패키지 형식 | Chart (YAML 템플릿) | Operator + Bundle |
| 업그레이드 | `helm upgrade` (수동) | 자동 채널 구독 |
| CRD 관리 | 수동 | Operator가 자동 관리 |
| 의존성 | 제한적 | Operator 의존성 그래프 |
| 생태계 | ArtifactHub | OperatorHub (Red Hat 인증) |
| 롤백 | `helm rollback` | Subscription 채널 변경 |

**RummiArena 이관 시**: Helm Chart를 그대로 사용 가능 (OpenShift도 Helm 지원). OLM 변환은 선택사항.

### A.5 네트워크: OVN-Kubernetes

OpenShift 4.12+는 기본 CNI로 OVN-Kubernetes를 사용한다.

| 기능 | Calico (Vanilla 대표) | OVN-Kubernetes (OpenShift) |
|------|----------------------|--------------------------|
| NetworkPolicy | ✅ | ✅ |
| Egress IP | ✅ (Enterprise) | ✅ 기본 지원 |
| Egress 방화벽 | ✅ | ✅ (EgressNetworkPolicy) |
| 멀티캐스트 | ❌ | ✅ |
| IPsec 암호화 | ✅ | ✅ |
| BGP 라우팅 | ✅ | ❌ (OVS 기반) |

### A.6 마이그레이션 체크리스트 (Vanilla K8s → OpenShift)

- [ ] 모든 컨테이너 non-root UID 확인 (`USER 1001` 등)
- [ ] SCC 정책 검토 (기존 runAsRoot 사용 서비스 파악)
- [ ] Ingress 리소스 → Route 또는 OpenShift Ingress 호환 확인
- [ ] PVC StorageClass 매핑 (기존 CSI 드라이버 → OpenShift 스토리지)
- [ ] Docker Hub → OpenShift 내장 레지스트리 또는 Quay.io 이관
- [ ] BuildConfig / ImageStream 활용 여부 결정
- [ ] OAuth / LDAP 통합 설정
- [ ] ArgoCD (OCP용 ArgoCD Operator 또는 OpenShift GitOps Operator 사용)
- [ ] Helm Chart 테스트 (OCP 호환성 검증)
- [ ] SonarQube, GitLab Runner 등 CI 인프라 재배포

---

## 별첨 B: 주요 YAML 예제 모음

### B.1 NetworkPolicy (기본 deny-all + 허용 규칙)

```yaml
# 기본 deny-all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: rummikub
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
# game-server: Ingress Controller + ai-adapter 접근 허용
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-game-server
  namespace: rummikub
spec:
  podSelector:
    matchLabels:
      app: game-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: traefik
    ports:
    - port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: ai-adapter
    ports:
    - port: 3001
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - port: 6379
  # DNS 허용
  - ports:
    - port: 53
      protocol: UDP
```

### B.2 ResourceQuota + LimitRange

```yaml
# ResourceQuota (네임스페이스 전체 쿼터)
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rummikub-quota
  namespace: rummikub
spec:
  hard:
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "16"
    limits.memory: 32Gi
    pods: "30"
    persistentvolumeclaims: "10"
---
# LimitRange (Pod 기본값)
apiVersion: v1
kind: LimitRange
metadata:
  name: rummikub-limits
  namespace: rummikub
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "4"
      memory: "4Gi"
```

### B.3 HPA (Horizontal Pod Autoscaler)

```yaml
# game-server HPA (트래픽 급증 대응)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: game-server-hpa
  namespace: rummikub
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: game-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # 5분 안정화 (게임 중 스케일다운 방지)
```

### B.4 PodDisruptionBudget

```yaml
# 게임 진행 중 최소 1개 Pod 보장
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: game-server-pdb
  namespace: rummikub
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: game-server
```

---

## 참고 문서

| 문서 | 경로 |
|------|------|
| K8s 아키텍처 (RummiArena) | `docs/05-deployment/04-k8s-architecture.md` |
| K8s 보안 평가 | `docs/05-deployment/12-k8s-security-assessment.md` |
| 타임아웃 체인 SSOT | `docs/02-design/41-timeout-chain-breakdown.md` |
| 배포 가이드 (Helm) | `docs/05-deployment/01-deployment-guide.md` |
| Istio 설치 스크립트 | `scripts/istio/` |
| Traefik YAML | `helm/traefik/` |

---

*작성일: 2026-05-26 | 버전: 1.0*

*작성자: Claude Sonnet 4.6*