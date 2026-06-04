---
title: "AI 기반 Kubernetes 운영 — AI가 실제로 무엇을 만들고 어떻게 쓰이는가"
date: 2026-06-04 08:15:00 +0900
categories: [AI,  AIOps]
mermaid: [True]
tags: [AI,  Kubernetes,  K8sGPT,  kubectl-ai,  Claude.write]
---


> **이 문서의 읽는 법:** 각 절은 다음 구조로 구성됩니다.  
> ① 문제 상황 → ② AI에게 입력하는 것 → ③ AI가 만들어내는 결과물 → ④ 그 결과물을 실제로 어떻게 활용하는가

**작성일:** 2026-06-04  
**기준:** 2026년 6월 현재 공개된 실제 도구 및 기능 기반 (K8sGPT, kubectl-ai, Karpenter/EKS Auto Mode, Istio Ambient Mode, OpenTelemetry 등)

## 관련글

[**AI 기반 Kubernetes 운영 완전 가이드**](https://k82022603.github.io/posts/ai-%EA%B8%B0%EB%B0%98-kubernetes-%EC%9A%B4%EC%98%81-%EC%99%84%EC%A0%84-%EA%B0%80%EC%9D%B4%EB%93%9C/)

---

## 목차

1. [이 커리큘럼에서 "AI 기반"이 의미하는 것](#1-이-커리큘럼에서-ai-기반이-의미하는-것)
2. [파트 1 — AI 기반 Kubernetes 운영 기본](#2-파트-1--ai-기반-kubernetes-운영-기본)
   - 2.1 [EKS 아키텍처 설계 — AI가 Terraform 코드를 만들어준다](#21-eks-아키텍처-설계--ai가-terraform-코드를-만들어준다)
   - 2.2 [워크로드 배포 — AI가 Deployment YAML을 만들어준다](#22-워크로드-배포--ai가-deployment-yaml을-만들어준다)
   - 2.3 [Ingress/Service 설계 — AI가 라우팅 규칙을 설계해준다](#23-ingressservice-설계--ai가-라우팅-규칙을-설계해준다)
   - 2.4 [StorageClass 관리 — AI가 스토리지 전략을 추천해준다](#24-storageclass-관리--ai가-스토리지-전략을-추천해준다)
3. [파트 2 — AI 기반 Kubernetes 운영 심화](#3-파트-2--ai-기반-kubernetes-운영-심화)
   - 3.1 [DaemonSet/CronJob — AI가 복합 워크로드 패턴을 생성한다](#31-daemonsetcronjob--ai가-복합-워크로드-패턴을-생성한다)
   - 3.2 [StatefulSet + NetworkPolicy — AI가 상태 저장 서비스와 보안 정책을 자동화한다](#32-statefulset--networkpolicy--ai가-상태-저장-서비스와-보안-정책을-자동화한다)
   - 3.3 [Affinity/Taint 스케줄링 — AI가 노드 배치 전략을 최적화한다](#33-affinitytaint-스케줄링--ai가-노드-배치-전략을-최적화한다)
   - 3.4 [ResourceQuota 용량 계획 — AI가 적정 할당량을 추론한다](#34-resourcequota-용량-계획--ai가-적정-할당량을-추론한다)
4. [파트 3 — K8s 전용 AI 도구 및 Observability](#4-파트-3--k8s-전용-ai-도구-및-observability)
   - 4.1 [K8sGPT — 클러스터를 스캔해 AI가 진단서를 써준다](#41-k8sgpt--클러스터를-스캔해-ai가-진단서를-써준다)
   - 4.2 [kubectl-ai — 자연어로 클러스터를 조작한다](#42-kubectl-ai--자연어로-클러스터를-조작한다)
   - 4.3 [Helm 차트 리팩토링 — AI가 레거시 차트를 현대화한다](#43-helm-차트-리팩토링--ai가-레거시-차트를-현대화한다)
   - 4.4 [Istio 트래픽 제어 — AI가 트래픽 가중치와 장애 격리를 설계한다](#44-istio-트래픽-제어--ai가-트래픽-가중치와-장애-격리를-설계한다)
   - 4.5 [AI 기반 Observability — AI가 이상을 감지하고 원인을 찾아준다](#45-ai-기반-observability--ai가-이상을-감지하고-원인을-찾아준다)
   - 4.6 [EFK 로그 파이프라인 — AI가 로그를 읽고 패턴을 분류한다](#46-efk-로그-파이프라인--ai가-로그를-읽고-패턴을-분류한다)
5. [전체 흐름 요약 — AI가 개입하는 지점들](#5-전체-흐름-요약--ai가-개입하는-지점들)

---

## 1. 이 커리큘럼에서 "AI 기반"이 의미하는 것

### 오해를 먼저 정리한다

"AI 기반 Kubernetes 운영"이라는 표현을 들으면 흔히 "AI가 클러스터를 알아서 다 운영해준다"는 오해를 할 수 있다. 실제는 다르다. 2026년 현재 AI는 Kubernetes 운영에서 크게 세 가지 방식으로 개입한다.

**첫째, 코드·설정 생성기로서의 AI다.** 엔지니어가 자연어로 요구사항을 설명하면 AI가 YAML 매니페스트, Terraform 코드, Helm 차트, NetworkPolicy를 직접 작성해 준다. 엔지니어는 생성된 결과물을 검토하고 `kubectl apply`로 적용한다.

**둘째, 진단 분석기로서의 AI다.** 클러스터에서 발생한 오류를 AI가 분석하여 "왜 실패했고, 어떻게 고쳐야 하는가"를 자연어로 설명한다. K8sGPT가 대표적인 도구다. 엔지니어가 수십 개의 `kubectl describe`, `kubectl logs` 명령을 실행하며 30~60분을 소비하던 진단 과정을 수 초 안에 끝낸다.

**셋째, 대화형 운영 에이전트로서의 AI다.** kubectl-ai처럼 "프론트엔드 Pod가 왜 응답이 없죠?"라는 질문에 AI가 직접 클러스터에 진단 명령을 실행하고, 결과를 해석하여 수정 방법까지 제시한다. 단, 실제 변경 적용 전에는 반드시 엔지니어의 확인을 거친다.

아래 그림은 AI 개입 유형을 한눈에 보여준다.

```mermaid
graph LR
    ENG[엔지니어]

    subgraph "AI 개입 유형 1: 생성기"
        GEN_IN[자연어 요구사항]
        GEN_OUT[YAML / Terraform / Helm 코드]
    end

    subgraph "AI 개입 유형 2: 진단기"
        DIAG_IN[클러스터 상태 / 오류 로그]
        DIAG_OUT[근본 원인 + 해결 절차\n자연어 설명]
    end

    subgraph "AI 개입 유형 3: 에이전트"
        AGT_IN[자연어 질문 / 지시]
        AGT_EXEC[kubectl 명령 자동 실행\n확인 후]
        AGT_OUT[진단 결과 + 수정 제안]
    end

    ENG --> GEN_IN --> GEN_OUT --> |검토 후 적용| ENG
    ENG --> DIAG_IN --> DIAG_OUT --> |참고하여 조치| ENG
    ENG --> AGT_IN --> AGT_EXEC --> AGT_OUT --> |수락/거부| ENG
```

---

## 2. 파트 1 — AI 기반 Kubernetes 운영 기본

### 2.1 EKS 아키텍처 설계 — AI가 Terraform 코드를 만들어준다

#### 문제 상황

새로운 서비스를 위한 EKS 클러스터를 처음부터 구성해야 한다. 멀티 AZ, Managed Node Group, IAM IRSA, VPC 설계, Karpenter 기반 노드 자동 스케일링까지 고려해야 한다. 이를 수동으로 Terraform으로 작성하면 수백 줄의 코드와 AWS 문서를 반나절 이상 들여다봐야 한다.

#### AI에게 입력하는 것

엔지니어는 Claude나 ChatGPT, 또는 Claude Code에 다음과 같이 입력한다.

```
프로덕션 환경용 AWS EKS 클러스터를 Terraform으로 만들어줘.
요구사항:
- 리전: ap-northeast-2 (서울)
- 멀티 AZ (ap-northeast-2a, 2b, 2c)
- Kubernetes 버전: 1.32
- Managed Node Group: t3.medium × 최소 2, 최대 6
- GPU 노드 풀도 별도로 (g5.xlarge, 최소 0 최대 4) — AI 추론용
- IRSA 활성화
- Karpenter 설치 포함
- 클러스터 엔드포인트는 퍼블릭/프라이빗 모두 허용
```

#### AI가 만들어내는 결과물

AI는 아래와 같은 Terraform 코드 블록을 완성된 형태로 생성한다.

```hcl
# main.tf (AI가 생성한 코드 예시)

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name            = "eks-prod-vpc"
  cidr            = "10.0.0.0/16"
  azs             = ["ap-northeast-2a", "ap-northeast-2b", "ap-northeast-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway     = true
  single_nat_gateway     = false   # 고가용성: AZ별 NAT GW
  enable_dns_hostnames   = true

  # Kubernetes가 요구하는 서브넷 태그
  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
    "karpenter.sh/discovery"          = "eks-prod"
  }
  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }
}

module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "eks-prod"
  cluster_version = "1.32"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  # 퍼블릭+프라이빗 엔드포인트
  cluster_endpoint_public_access  = true
  cluster_endpoint_private_access = true

  # IRSA 활성화 (IAM Roles for Service Accounts)
  enable_irsa = true

  # EKS Managed Node Groups
  eks_managed_node_groups = {
    general = {
      instance_types = ["t3.medium"]
      min_size       = 2
      max_size       = 6
      desired_size   = 2

      labels = {
        role = "general"
      }
    }

    gpu = {
      instance_types = ["g5.xlarge"]
      min_size       = 0
      max_size       = 4
      desired_size   = 0

      # GPU 노드 전용 Taint: AI 추론 워크로드만 배치
      taints = [{
        key    = "nvidia.com/gpu"
        value  = "true"
        effect = "NO_SCHEDULE"
      }]

      labels = {
        role                  = "gpu"
        "accelerator"         = "nvidia"
      }
    }
  }
}

# Karpenter IAM 역할 (IRSA 기반)
module "karpenter" {
  source  = "terraform-aws-modules/eks/aws//modules/karpenter"
  version = "~> 20.0"

  cluster_name = module.eks.cluster_name

  # Karpenter가 인스턴스를 시작할 IAM 권한
  enable_irsa                     = true
  irsa_oidc_provider_arn          = module.eks.oidc_provider_arn
  irsa_namespace_service_accounts = ["karpenter:karpenter"]
}
```

위 코드 외에 AI는 `karpenter-nodeclass.yaml`, `karpenter-nodepool.yaml`(노드 프로비저닝 정책), `outputs.tf`, `variables.tf`도 함께 생성한다.

#### 실제 활용

엔지니어는 생성된 코드를 검토하여 회사 내부 정책(보안 그룹 규칙, 태그 표준 등)에 맞게 수정한 뒤 `terraform plan` → `terraform apply`를 실행한다. AWS re:Invent 2025에서 발표된 **EKS Auto Mode**가 활성화된 클러스터라면, Karpenter 대신 EKS 자체가 노드 프로비저닝을 자동 처리하므로 Karpenter 모듈 부분을 제거하고 `cluster_compute_config = { enabled = true }` 한 줄로 대체할 수 있다는 추가 안내도 AI가 제공한다.

---

### 2.2 워크로드 배포 — AI가 Deployment YAML을 만들어준다

#### 문제 상황

팀에서 새로 개발한 Node.js 기반 결제 서비스를 EKS에 배포해야 한다. 운영 경험이 적은 개발자가 Deployment YAML을 작성하면 liveness/readiness 프로브 누락, resource request/limit 미설정, 보안 컨텍스트 없음, Pod Anti-Affinity 미설정 같은 실수가 빈번하게 발생한다.

#### AI에게 입력하는 것

```
Node.js 결제 서비스 Deployment YAML 만들어줘.
- 이미지: my-ecr.amazonaws.com/payment-service:2.1.0
- 레플리카: 3개
- 포트: 3000
- 환경변수: DATABASE_URL은 Secret에서, LOG_LEVEL은 ConfigMap에서 읽기
- CPU: 200m 요청 / 500m 제한, 메모리: 256Mi 요청 / 512Mi 제한
- 헬스체크: /health 엔드포인트 사용
- 같은 Deployment의 Pod들이 서로 다른 노드에 분산 배치되어야 함
- 보안: root로 실행 금지
```

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 payment-service Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
  namespace: production
  labels:
    app: payment-service
    version: "2.1.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0     # 배포 중 무중단 보장
  template:
    metadata:
      labels:
        app: payment-service
        version: "2.1.0"
    spec:
      # Pod Anti-Affinity: 동일 Deployment Pod들을 서로 다른 노드에 분산
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - payment-service
            topologyKey: kubernetes.io/hostname

      # 보안: root 실행 금지, 읽기 전용 루트 파일시스템
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000

      containers:
      - name: payment-service
        image: my-ecr.amazonaws.com/payment-service:2.1.0
        ports:
        - containerPort: 3000

        # 리소스 요청/제한
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

        # 환경변수: Secret과 ConfigMap에서 주입
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: payment-service-secrets
              key: database-url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: payment-service-config
              key: log-level

        # Liveness Probe: 컨테이너가 살아있는지 확인
        # 실패 시 컨테이너를 재시작함
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3

        # Readiness Probe: 트래픽을 받을 준비가 됐는지 확인
        # 실패 시 Service의 엔드포인트에서 제외됨 (재시작 없음)
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3

        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

#### 실제 활용

AI가 생성한 YAML에는 직접 작성 시 빠뜨리기 쉬운 항목들이 모두 포함되어 있다. `podAntiAffinity`는 3개의 레플리카가 서로 다른 노드에 배치되게 하여 노드 장애 시에도 서비스가 지속되도록 한다. `readinessProbe`와 `livenessProbe`를 분리한 것도 중요한데, readiness가 실패하면 재시작 없이 트래픽만 차단하여 스타트업 지연을 안전하게 처리한다. `securityContext`의 `readOnlyRootFilesystem: true`는 컨테이너 침해 시 공격자가 파일을 쓰지 못하게 한다.

---

### 2.3 Ingress/Service 설계 — AI가 라우팅 규칙을 설계해준다

#### 문제 상황

세 개의 마이크로서비스(API 서버, 웹 프론트엔드, 관리자 대시보드)가 하나의 EKS 클러스터에 배포되어 있다. 각 서비스로 트래픽을 라우팅하는 Ingress를 설정해야 하며, 관리자 대시보드는 특정 IP 대역에서만 접근 가능해야 한다.

#### AI에게 입력하는 것

```
AWS ALB Ingress Controller를 사용하는 Ingress 설정 만들어줘.
- api.example.com → api-service (포트 8080)
- www.example.com, example.com → frontend-service (포트 80)
- admin.example.com → admin-service (포트 9000), 
  단 회사 IP 대역(203.0.113.0/24)에서만 접근 허용
- SSL 인증서는 ACM ARN: arn:aws:acm:ap-northeast-2:123456789:certificate/abc-123
- HTTP는 HTTPS로 자동 리다이렉트
```

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 통합 Ingress 설정
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: production-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip

    # ACM SSL 인증서 ARN
    alb.ingress.kubernetes.io/certificate-arn: >-
      arn:aws:acm:ap-northeast-2:123456789:certificate/abc-123

    # HTTP → HTTPS 리다이렉트
    alb.ingress.kubernetes.io/ssl-redirect: "443"

    # 관리자 대시보드 IP 화이트리스트
    # admin.example.com 규칙에만 적용됨
    alb.ingress.kubernetes.io/inbound-cidrs: |
      203.0.113.0/24

    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080

  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  - host: example.com     # 루트 도메인도 처리
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 9000
```

AI는 이 YAML을 생성하면서 동시에 세 개의 `ClusterIP` Service 설정과, admin 서비스에 추가 보안 레이어로 `NetworkPolicy`를 사용하는 방법도 함께 제안한다.

---

### 2.4 StorageClass 관리 — AI가 스토리지 전략을 추천해준다

#### 문제 상황

세 가지 워크로드가 서로 다른 스토리지 요구사항을 갖고 있다. MySQL 데이터베이스(높은 IOPS 필요), 로그 수집기(대용량 저비용 필요), 머신러닝 학습 데이터(여러 Pod에서 동시 읽기 가능해야 함).

#### AI에게 입력하는 것

```
다음 세 가지 워크로드에 맞는 AWS EKS용 StorageClass 세 개를 만들어줘.
1. MySQL DB — 높은 IOPS, 단일 노드 연결, 프로덕션 품질
2. 로그 수집기 — 대용량, 저비용, 순차 쓰기 위주
3. ML 학습 데이터 — 여러 Pod가 동시에 읽기 가능해야 함 (ReadWriteMany)
```

#### AI가 만들어내는 결과물

```yaml
# 1. MySQL용 — EBS gp3, 높은 IOPS 설정
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3-high-iops
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "6000"          # gp3 기본 3000 IOPS → 6000으로 증가
  throughput: "500"     # MB/s
  encrypted: "true"     # 저장 데이터 암호화
reclaimPolicy: Retain   # PVC 삭제 후에도 데이터 보존
volumeBindingMode: WaitForFirstConsumer  # Pod가 스케줄된 AZ에서 볼륨 생성
allowVolumeExpansion: true
---
# 2. 로그 수집기용 — EBS gp3 기본 설정, 비용 효율
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3-standard
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Delete     # PVC 삭제 시 볼륨도 함께 삭제
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
# 3. ML 학습 데이터용 — EFS (ReadWriteMany 지원)
# 여러 Pod가 동시에 같은 볼륨에 접근 가능
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-shared-readwritemany
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0123456789abcdef  # 실제 EFS 파일시스템 ID로 교체
  directoryPerms: "700"
  gidRangeStart: "1000"
  gidRangeEnd: "2000"
reclaimPolicy: Retain
```

AI는 각 StorageClass 아래에 사용 예시 PVC까지 함께 생성하며, EBS는 동일 AZ 내에서만 사용 가능하므로 Pod와 볼륨의 AZ를 맞추는 `WaitForFirstConsumer` 설정의 중요성도 설명한다.

---

## 3. 파트 2 — AI 기반 Kubernetes 운영 심화

### 3.1 DaemonSet/CronJob — AI가 복합 워크로드 패턴을 생성한다

#### 문제 상황 1 — DaemonSet

모든 노드에서 Fluent Bit를 실행하여 컨테이너 로그를 수집해야 한다. Control Plane 노드에는 배포하지 않고, GPU 노드에는 배포해야 하며, 기존 워크로드에 영향을 주지 않도록 리소스를 제한해야 한다.

#### AI에게 입력하는 것

```
Fluent Bit DaemonSet 만들어줘.
- Control Plane 노드 제외
- 모든 Worker Node (CPU + GPU 포함)에 배포
- 컨테이너 로그(/var/log/containers)와 K8s 이벤트 수집
- CPU 100m / 메모리 128Mi 제한
- Fluent Bit 설정은 ConfigMap으로 분리
- 출력: Elasticsearch (es.internal.company.com:9200)
```

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 Fluent Bit DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
  labels:
    app: fluent-bit
spec:
  selector:
    matchLabels:
      app: fluent-bit
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit  # K8s API 접근용 SA
      tolerations:
      # GPU 노드의 Taint를 허용 (GPU 노드에도 배포)
      - key: "nvidia.com/gpu"
        operator: "Exists"
        effect: "NoSchedule"
      # Control Plane 노드는 허용하지 않음 (아무 toleration 없음)

      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:3.2
        resources:
          requests:
            cpu: "50m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"

        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: config
          mountPath: /fluent-bit/etc/

      volumes:
      # 호스트의 /var/log를 마운트 (컨테이너 로그 접근)
      - name: varlog
        hostPath:
          path: /var/log
      - name: config
        configMap:
          name: fluent-bit-config
---
# ConfigMap으로 분리된 Fluent Bit 설정
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush        5
        Log_Level    info
        Parsers_File parsers.conf

    [INPUT]
        Name             tail
        Path             /var/log/containers/*.log
        multiline.parser docker, cri
        Tag              kube.*
        Mem_Buf_Limit    10MB
        Skip_Long_Lines  On

    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_Tag_Prefix     kube.var.log.containers.
        Merge_Log           On
        Keep_Log            Off
        K8S-Logging.Parser  On
        K8S-Logging.Exclude On

    [OUTPUT]
        Name  es
        Match *
        Host  es.internal.company.com
        Port  9200
        Index k8s-logs
        Type  _doc
```

#### 문제 상황 2 — CronJob

매일 새벽 2시에 MySQL 데이터베이스를 S3에 백업하는 작업이 필요하다. 이전 실행이 아직 완료되지 않았을 경우 새 실행을 시작하지 않아야 하며, 실패 이력은 3개만 보관한다.

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 MySQL 백업 CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup
  namespace: production
spec:
  schedule: "0 2 * * *"    # 매일 새벽 2시 (KST 기준이면 서버 시간 확인 필요)
  timeZone: "Asia/Seoul"   # K8s 1.27+ 지원: 타임존 직접 지정
  
  concurrencyPolicy: Forbid     # 이전 Job이 실행 중이면 새 Job 시작 안 함
  startingDeadlineSeconds: 300  # 5분 이내에 시작 못하면 실패로 처리
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3

  jobTemplate:
    spec:
      backoffLimit: 2       # 실패 시 최대 2회 재시도
      activeDeadlineSeconds: 3600  # 1시간 넘으면 강제 종료
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: mysql-backup
            image: mysql:8.0
            env:
            - name: MYSQL_HOST
              value: "mysql-0.mysql.production.svc.cluster.local"
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
            - name: AWS_DEFAULT_REGION
              value: "ap-northeast-2"
            - name: BACKUP_BUCKET
              value: "company-mysql-backups"
            command:
            - /bin/sh
            - -c
            - |
              TIMESTAMP=$(date +%Y%m%d_%H%M%S)
              # DB 백업 실행
              mysqldump -h $MYSQL_HOST -u root -p$MYSQL_PASSWORD \
                --all-databases --single-transaction \
                | gzip > /tmp/backup_$TIMESTAMP.sql.gz
              # S3 업로드
              aws s3 cp /tmp/backup_$TIMESTAMP.sql.gz \
                s3://$BACKUP_BUCKET/mysql/backup_$TIMESTAMP.sql.gz
              echo "백업 완료: backup_$TIMESTAMP.sql.gz"
```

---

### 3.2 StatefulSet + NetworkPolicy — AI가 상태 저장 서비스와 보안 정책을 자동화한다

#### 문제 상황

3노드 MySQL 클러스터(Primary 1개 + Replica 2개)를 EKS에 배포해야 한다. 동시에 MySQL에는 백엔드 서비스만 접근 가능하고 외부에서는 접근할 수 없도록 NetworkPolicy도 설정해야 한다.

#### AI에게 입력하는 것

```
MySQL 3노드 클러스터 StatefulSet 만들어줘.
- mysql-0: Primary, mysql-1~2: Replica
- Headless Service로 개별 Pod DNS 접근 가능
- 데이터 저장: PVC 20Gi (gp3)
- NetworkPolicy: backend 네임스페이스의 Pod만 3306 접근 허용,
  외부 및 다른 네임스페이스에서 접근 차단
```

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 MySQL StatefulSet + 관련 리소스

# 1. Headless Service (Pod별 고정 DNS 이름 부여)
# mysql-0.mysql.production.svc.cluster.local 형태로 접근 가능
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: production
  labels:
    app: mysql
spec:
  clusterIP: None          # Headless Service
  selector:
    app: mysql
  ports:
  - port: 3306
---
# 2. StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: production
spec:
  serviceName: mysql       # Headless Service 이름과 일치해야 함
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
      # Init Container: Pod 순번(ordinal)에 따라 Primary/Replica 설정 분기
      - name: init-mysql
        image: mysql:8.0
        command:
        - bash
        - -c
        - |
          # Pod 이름에서 순번 추출 (mysql-0, mysql-1, ...)
          [[ $(hostname) =~ -([0-9]+)$ ]] && ordinal=${BASH_REMATCH[1]}
          # mysql-0은 Primary, 나머지는 Replica
          if [[ $ordinal -eq 0 ]]; then
            echo "[mysqld]" > /mnt/conf.d/server-id.cnf
            echo "server-id=1" >> /mnt/conf.d/server-id.cnf
            echo "log_bin=mysql-bin" >> /mnt/conf.d/server-id.cnf
          else
            echo "[mysqld]" > /mnt/conf.d/server-id.cnf
            echo "server-id=$((ordinal + 1))" >> /mnt/conf.d/server-id.cnf
            echo "super_read_only=ON" >> /mnt/conf.d/server-id.cnf
          fi
        volumeMounts:
        - name: conf
          mountPath: /mnt/conf.d

      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "2"
            memory: "4Gi"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        - name: conf
          mountPath: /etc/mysql/conf.d

      volumes:
      - name: conf
        emptyDir: {}

  # PVC 템플릿: 각 Pod에 독립적인 PVC 자동 생성
  # mysql-0 → data-mysql-0, mysql-1 → data-mysql-1, ...
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ebs-gp3-high-iops
      resources:
        requests:
          storage: 20Gi
---
# 3. NetworkPolicy: backend 네임스페이스만 MySQL 접근 허용
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-access-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
  - Ingress
  ingress:
  # 규칙 1: backend 네임스페이스의 Pod만 3306 허용
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 3306
  # 규칙 2: 같은 MySQL Pod끼리의 통신 허용 (복제 동기화용)
  - from:
    - podSelector:
        matchLabels:
          app: mysql
    ports:
    - protocol: TCP
      port: 3306
  # 나머지 모든 Ingress는 차단 (명시적 허용 외 전부 거부)
```

NetworkPolicy 생성 시 AI는 중요한 주의사항도 함께 설명한다. NetworkPolicy는 CNI 플러그인이 이를 지원해야 실제로 적용되며, EKS에서는 기본 CNI(VPC CNI)가 NetworkPolicy를 지원하지 않기 때문에 Calico나 Cilium을 함께 설치해야 한다는 내용이다.

---

### 3.3 Affinity/Taint 스케줄링 — AI가 노드 배치 전략을 최적화한다

#### 문제 상황

세 종류의 워크로드가 있다. AI 추론 서비스(GPU 노드 전용), 일반 API 서비스(Spot 노드 사용 가능), 데이터베이스(Spot 노드 사용 불가, 안정적인 On-Demand 노드만). 각 워크로드를 올바른 노드에 배치하는 설정이 필요하다.

#### AI에게 입력하는 것

```
세 가지 워크로드의 스케줄링 전략 설계해줘.
- GPU 노드에는 Taint gpu=nvidia:NoSchedule 설정됨
- Spot 노드에는 Label spot=true 설정됨
- On-Demand 노드에는 Label node-type=on-demand 설정됨

1. AI 추론 서비스: GPU 노드에만 배치 (반드시)
2. API 서비스: Spot 노드 선호, 없으면 On-Demand에 배치 가능
3. DB (StatefulSet): On-Demand 노드에만 배치, Spot 절대 불가
```

#### AI가 만들어내는 결과물

AI는 각 워크로드에 대해 별도의 nodeAffinity/tolerations 블록을 생성한다.

```yaml
# 1. AI 추론 서비스 — GPU 노드 필수 배치
spec:
  tolerations:
  # GPU 노드의 Taint를 허용해야 GPU 노드에 스케줄 가능
  - key: "nvidia.com/gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      # GPU 노드가 없으면 배치 자체가 안 됨 (Hard 규칙)
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia

---
# 2. API 서비스 — Spot 노드 선호, 없으면 On-Demand 허용
spec:
  affinity:
    nodeAffinity:
      # Soft 규칙: 가능하면 Spot에, 없으면 아무 노드에나 배치
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80     # 80점 가중치 — 강하게 선호
        preference:
          matchExpressions:
          - key: spot
            operator: In
            values:
            - "true"
      - weight: 20     # On-Demand도 일부 허용
        preference:
          matchExpressions:
          - key: node-type
            operator: In
            values:
            - on-demand

---
# 3. 데이터베이스 — On-Demand 노드 전용 (Spot 완전 배제)
spec:
  affinity:
    nodeAffinity:
      # Hard 규칙: On-Demand 노드에만 배치
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-type
            operator: In
            values:
            - on-demand
          - key: spot    # Spot 노드는 명시적으로 제외
            operator: DoesNotExist
```

AI는 이 설정의 실제 효과도 설명한다. 예를 들어 `tolerations`와 `nodeAffinity`가 모두 필요한 이유를 명확히 한다. Taint가 있는 GPU 노드에 배치하려면 `tolerations`로 Taint를 허용해야 하고, 거기에 더해 `nodeAffinity`로 GPU 노드만 선택하도록 해야 의도한 배치가 이루어진다. `tolerations`만 있으면 GPU 노드에 배치될 수는 있지만 반드시 GPU 노드에만 배치된다고 보장할 수 없다.

---

### 3.4 ResourceQuota 용량 계획 — AI가 적정 할당량을 추론한다

#### 문제 상황

개발팀과 운영팀이 같은 클러스터를 공유한다. 개발팀이 실수로 대량의 Pod를 띄워 운영 서비스에 영향을 준 적이 있다. 네임스페이스별 할당량을 설정해야 한다.

#### AI에게 입력하는 것

```
다음 팀별로 ResourceQuota와 LimitRange 설정해줘.
클러스터 전체 자원: 노드 10개, 각 노드 8 CPU / 16Gi 메모리

팀 정보:
- production 네임스페이스: 전체의 60% 보장 필요, 최대 70%까지
- staging 네임스페이스: 전체의 20%, Pod 최대 30개
- development 네임스페이스: 전체의 10%, 개별 컨테이너 무제한 방지
```

#### AI가 만들어내는 결과물

```yaml
# AI가 생성한 ResourceQuota 및 LimitRange

# Production: CPU 48코어 / 메모리 96Gi (전체 60%)
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "48"
    requests.memory: "96Gi"
    limits.cpu: "56"       # 최대 70%까지 버스트 허용
    limits.memory: "112Gi"
    pods: "200"
    services: "50"
    persistentvolumeclaims: "30"
---
# Staging: CPU 16코어 / 메모리 32Gi (전체 20%), Pod 30개 제한
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging-quota
  namespace: staging
spec:
  hard:
    requests.cpu: "16"
    requests.memory: "32Gi"
    limits.cpu: "16"
    limits.memory: "32Gi"
    pods: "30"
---
# Development: LimitRange로 개별 컨테이너 상한선 설정
# request/limit 미설정 시 기본값 자동 주입
apiVersion: v1
kind: LimitRange
metadata:
  name: development-limitrange
  namespace: development
spec:
  limits:
  - type: Container
    default:           # request/limit 미설정 시 자동 적용되는 기본값
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:               # 컨테이너 하나가 사용 가능한 최대치
      cpu: "2"
      memory: "4Gi"
    min:
      cpu: "50m"
      memory: "64Mi"
```

---

## 4. 파트 3 — K8s 전용 AI 도구 및 Observability

### 4.1 K8sGPT — 클러스터를 스캔해 AI가 진단서를 써준다

#### 문제 상황

프로덕션 클러스터에 갑자기 여러 Pod가 오류 상태가 됐다. 숙련된 엔지니어는 `kubectl describe`, `kubectl logs`, `kubectl get events` 등을 수십 번 실행하며 원인을 찾지만, 이 과정에 30~60분이 걸린다.

#### K8sGPT에 입력하는 것

```bash
# AI 백엔드 설정 (한 번만 설정하면 됨)
k8sgpt auth add --backend openai --model gpt-4o

# 또는 Claude 사용
k8sgpt auth add --backend anthropic --model claude-sonnet-4-6

# 전체 클러스터 스캔 + AI 설명 출력
k8sgpt analyze --explain
```

#### AI가 만들어내는 결과물 (실제 출력 형태)

아래는 K8sGPT가 실제로 생성하는 출력의 예시다.

```
$ k8sgpt analyze --explain --namespace=production

AI Provider: openai

100% |████████████████████| (3/3, 12 it/min)

────────────────────────────────────────────────
0 production/payment-service-6d8f9b-x2p7k (Pod)
────────────────────────────────────────────────
Error: Back-off restarting failed container

AI Analysis:
  컨테이너가 반복 재시작(CrashLoopBackOff) 상태입니다.
  컨테이너 로그를 확인한 결과, 환경변수 'DATABASE_URL'이
  설정되지 않아 애플리케이션 시작 시 다음 오류가 발생합니다:
  
    Error: ENOTFOUND - Database connection failed:
    connect ENOTFOUND undefined

  원인: 'payment-service-secrets' Secret이 'production'
  네임스페이스에 존재하지 않거나, 키 이름이 일치하지 않습니다.

  해결 방법:
  1. Secret 확인:
     kubectl get secret payment-service-secrets -n production
  
  2. Secret이 없으면 생성:
     kubectl create secret generic payment-service-secrets \
       --from-literal=database-url='mysql://user:pass@mysql-0.mysql:3306/app' \
       -n production
  
  3. Pod 재시작:
     kubectl rollout restart deployment/payment-service -n production

────────────────────────────────────────────────
1 production/api-gateway-deploy (Deployment)
────────────────────────────────────────────────
Error: Deployment has minimum availability warning

AI Analysis:
  Deployment의 3개 레플리카 중 0개만 Ready 상태입니다.
  Event 로그에 다음 메시지가 반복됩니다:
  
    "Failed to pull image: my-ecr.amazonaws.com/api-gateway:v3.1.0:
     repository 'api-gateway' not found"

  원인: ECR에 지정된 이미지 태그(v3.1.0)가 존재하지 않습니다.
  빌드 파이프라인에서 이미지 푸시가 완료되지 않았거나
  태그가 잘못 지정된 것으로 보입니다.

  해결 방법:
  1. ECR에서 사용 가능한 태그 확인:
     aws ecr list-images --repository-name api-gateway \
       --query 'imageIds[].imageTag'
  
  2. 올바른 태그로 Deployment 수정:
     kubectl set image deployment/api-gateway-deploy \
       api-gateway=my-ecr.amazonaws.com/api-gateway:v3.0.9 \
       -n production

────────────────────────────────────────────────
2 production/mysql-2 (PersistentVolumeClaim)
────────────────────────────────────────────────
Error: PVC is in Pending state

AI Analysis:
  'data-mysql-2' PVC가 Pending 상태입니다.
  StorageClass 'ebs-gp3-high-iops'의 Provisioner가
  다음 오류를 반환하고 있습니다:
  
    "volume creation failed: no available capacity in zone
     ap-northeast-2c for gp3 volumes of 20Gi"

  원인: ap-northeast-2c AZ에서 EBS gp3 용량이 일시적으로
  부족한 상황입니다.

  해결 방법:
  1. mysql-2 Pod를 다른 AZ로 이동하거나,
     잠시 후 다시 시도하십시오.
  2. 또는 StorageClass에 allowedTopologies를 추가하여
     ap-northeast-2a 또는 2b AZ를 사용하도록 설정하세요.
```

#### 실제 활용

엔지니어는 이 출력을 보고 각 문제에 대해 AI가 제시한 해결 명령을 차례로 실행한다. 기존에 30~60분 걸리던 진단 과정이 수 초로 단축된다. K8sGPT는 또한 `Operator 모드`로 클러스터 내에 상시 배포하여 24시간 자동 모니터링하고, 이상 감지 시 Slack이나 Discord로 자동 알림을 보내도록 설정할 수 있다.

```mermaid
sequenceDiagram
    participant ENG as 엔지니어
    participant K8SGPT as K8sGPT CLI
    participant K8S as Kubernetes API
    participant LLM as LLM Backend (GPT/Claude)

    ENG->>K8SGPT: k8sgpt analyze --explain
    K8SGPT->>K8S: Pod/Deploy/PVC/SVC 상태 수집
    K8S-->>K8SGPT: 오류/이벤트/로그 데이터 반환
    K8SGPT->>K8SGPT: 20+ 내장 Analyzer로 문제 분류
    K8SGPT->>LLM: 익명화된 오류 컨텍스트 전송
    LLM-->>K8SGPT: 자연어 근본 원인 + 해결 절차
    K8SGPT-->>ENG: 진단 결과 출력 (오류 + AI 설명 + 해결 명령)
    ENG->>K8S: 제시된 해결 명령 실행
```

---

### 4.2 kubectl-ai — 자연어로 클러스터를 조작한다

#### K8sGPT vs kubectl-ai의 차이

K8sGPT가 "의사의 진단서"라면, kubectl-ai는 "직접 처방을 써주는 의사"다. K8sGPT는 문제를 설명하지만 직접 명령을 실행하지 않는다. kubectl-ai는 자연어로 지시하면 실제 kubectl 명령을 생성하고, 확인을 받은 뒤 실행까지 수행한다.

#### kubectl-ai와의 실제 대화 예시

아래는 kubectl-ai(GoogleCloudPlatform 버전)와 실제로 진행하는 멀티 턴 대화의 예시다.

```
$ kubectl-ai

> production 네임스페이스에서 Pending 상태인 Pod 모두 보여줘

[kubectl-ai가 자동 실행]
  kubectl get pods -n production --field-selector status.phase=Pending

NAME                    READY  STATUS   RESTARTS  AGE
mysql-2                 0/1    Pending  0         15m
ml-inference-7b4d9-xxx  0/1    Pending  0         3m

> mysql-2가 왜 Pending인지 분석해줘

[kubectl-ai가 자동 실행]
  kubectl describe pod mysql-2 -n production
  kubectl get pvc data-mysql-2 -n production
  kubectl get events -n production --field-selector involvedObject.name=mysql-2

AI 분석 결과:
  mysql-2 Pod는 PVC 'data-mysql-2'가 바인딩되지 않아 Pending 상태입니다.
  PVC는 ap-northeast-2c AZ에서 EBS 볼륨 생성을 시도했으나
  일시적 용량 부족으로 실패했습니다.

  두 가지 해결 방법이 있습니다:
  A) mysql-2 Pod를 다른 AZ로 강제 이동 (데이터 없으면 가능)
  B) 잠시 대기 후 EBS 용량 회복 시 자동 해결 (보통 10~30분)

  현재 StatefulSet이라 Pod를 임의로 이동하기 어렵습니다.
  B 방법을 권장합니다. 지금 당장 해결이 필요하면 말씀해주세요.

> 지금 당장 해결해야 함. PVC를 다른 AZ(ap-northeast-2b)에 다시 만들 수 있어?

AI 분석:
  StatefulSet의 volumeClaimTemplate으로 자동 생성된 PVC는
  직접 재생성이 필요합니다. 다음 절차를 제안합니다:
  
  1. mysql-2 Pod 삭제 (StatefulSet이 재생성)
  2. 기존 PVC data-mysql-2 삭제
  3. 새 PVC를 ap-northeast-2b AZ로 수동 생성
  4. StatefulSet의 volumeClaimTemplate 업데이트
  
  이 작업을 수행할까요? (데이터 손실 위험 없음 — mysql-2는 Replica이므로
  Primary에서 데이터 재동기화 가능)

  [확인/취소]

> 확인

[kubectl-ai가 실행할 명령 목록 표시 → 하나씩 확인 후 실행]

  Step 1: kubectl delete pod mysql-2 -n production
  Step 2: kubectl delete pvc data-mysql-2 -n production
  Step 3: PVC YAML 생성 및 적용 (ap-northeast-2b 지정)
  ...
```

이처럼 kubectl-ai는 단순 명령 변환이 아니라 현재 클러스터 상태를 직접 읽고, 문제를 분석하며, 해결 절차를 멀티 스텝으로 제안하고, 각 단계를 확인을 받으면서 실행하는 완전한 운영 에이전트로 동작한다.

---

### 4.3 Helm 차트 리팩토링 — AI가 레거시 차트를 현대화한다

#### 문제 상황

2년 전에 만든 레거시 Helm 차트가 있다. 여러 사람이 수정하면서 YAML이 중복되고, API 버전이 구식이며, 하드코딩된 값이 곳곳에 있다. AI에게 리팩토링을 맡기면 어떤 결과가 나오는가.

#### AI에게 입력하는 것 (Claude Code 기준)

```bash
claude "이 Helm 차트를 리팩토링해줘.
- 중복 블록을 _helpers.tpl Named Template으로 추출
- 하드코딩된 값은 values.yaml로 이동
- extensions/v1beta1 API를 최신 버전으로 업데이트
- 보안: root 실행, 불필요한 권한 제거"
```

#### AI가 만들어내는 결과물 — Before/After

**Before (리팩토링 전):**

```yaml
# templates/deployment.yaml (레거시)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app                     # 하드코딩
  namespace: production            # 하드코딩
spec:
  replicas: 3                      # 하드코딩
  template:
    spec:
      containers:
      - name: my-app
        image: myregistry/my-app:1.2.0   # 하드코딩
        resources:
          requests:
            cpu: 200m              # 하드코딩
            memory: 256Mi          # 하드코딩
        # livenessProbe 없음
        # securityContext 없음

# templates/service.yaml (레거시)
apiVersion: v1
kind: Service
metadata:
  name: my-app           # 위와 동일한 이름 반복 정의
  namespace: production  # 동일한 네임스페이스 반복 정의
```

**After (AI 리팩토링 후):**

{% raw %}

```yaml
# templates/_helpers.tpl (AI가 생성한 공통 헬퍼)
{{/*
공통 이름 생성 헬퍼 — 모든 리소스에서 재사용
*/}}
{{- define "myapp.name" -}}
{{- .Values.nameOverride | default .Chart.Name | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "myapp.labels" -}}
helm.sh/chart: {{ include "myapp.chart" . }}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/version: {{ .Values.image.tag | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

{% endraw %}

{% raw %}

```yaml
# templates/deployment.yaml (리팩토링 후)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.name" . }}          # 헬퍼 재사용
  namespace: {{ .Release.Namespace }}          # Helm 내장 변수
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}         # values.yaml에서 주입
  template:
    spec:
      securityContext:                         # AI가 추가한 보안 설정
        runAsNonRoot: true
        runAsUser: {{ .Values.securityContext.runAsUser }}
      containers:
      - name: {{ include "myapp.name" . }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        livenessProbe:                         # AI가 추가
          httpGet:
            path: {{ .Values.probes.liveness.path }}
            port: {{ .Values.service.port }}
          initialDelaySeconds: 30
          periodSeconds: 10
```
{% endraw %}

```yaml
# values.yaml (AI가 정리한 기본값)
replicaCount: 3

image:
  repository: myregistry/my-app
  tag: "1.2.0"
  pullPolicy: IfNotPresent

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

securityContext:
  runAsUser: 1000
  runAsNonRoot: true

probes:
  liveness:
    path: /health
  readiness:
    path: /ready
```

AI는 리팩토링 결과와 함께 변경 사항 요약도 제공한다. 하드코딩된 값 12개를 values.yaml로 이동했고, `_helpers.tpl`에 3개의 재사용 가능한 Named Template을 추출했으며, 보안 취약점(root 실행, livenessProbe 누락) 2개를 수정했다.

---

### 4.4 Istio 트래픽 제어 — AI가 트래픽 가중치와 장애 격리를 설계한다

#### 문제 상황

결제 서비스 v2를 배포하려 한다. 전체 트래픽을 즉시 v2로 전환하면 위험하므로, 처음에는 5%만 v2로 보내고 문제가 없으면 단계적으로 100%까지 늘리는 Canary 배포를 하고 싶다.

#### AI에게 입력하는 것

```
Istio로 payment-service Canary 배포 설정 만들어줘.
- v1: 현재 프로덕션 버전 (초기 95% 트래픽)
- v2: 신규 버전 (초기 5% 트래픽)
- v2 Pod에는 label version: v2 설정됨
- 헤더 x-canary: true가 있는 요청은 무조건 v2로
- v2에서 오류율 5% 초과 시 자동으로 트래픽을 v1으로 되돌리는
  Circuit Breaker도 설정
```

#### AI가 만들어내는 결과물

```yaml
# 1. DestinationRule: v1, v2 서브셋 정의 + Circuit Breaker
apiVersion: networking.istio.io/v1alpha1
kind: DestinationRule
metadata:
  name: payment-service-dr
  namespace: production
spec:
  host: payment-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      # Circuit Breaker 설정: v2 오류율 높으면 연결 차단
      outlierDetection:
        consecutiveGatewayErrors: 5     # 5회 연속 에러 시
        interval: 30s                   # 30초마다 평가
        baseEjectionTime: 30s           # 30초 동안 트래픽 차단
        maxEjectionPercent: 100         # v2 전체를 차단 가능
        minHealthPercent: 0

---
# 2. VirtualService: 트래픽 분배 규칙
apiVersion: networking.istio.io/v1alpha1
kind: VirtualService
metadata:
  name: payment-service-vs
  namespace: production
spec:
  hosts:
  - payment-service
  http:
  # 규칙 1: x-canary: true 헤더가 있으면 무조건 v2로
  - match:
    - headers:
        x-canary:
          exact: "true"
    route:
    - destination:
        host: payment-service
        subset: v2
      weight: 100

  # 규칙 2: 나머지는 95:5 비율로 분배
  - route:
    - destination:
        host: payment-service
        subset: v1
      weight: 95
    - destination:
        host: payment-service
        subset: v2
      weight: 5
    # v2에서 지연 발생 시 재시도
    retries:
      attempts: 3
      perTryTimeout: 5s
      retryOn: "5xx,reset,connect-failure"
```

AI는 이 설정과 함께 단계별 Canary 진행 계획도 제안한다. v2 5% → 오류율 모니터링 3~5분 → 문제 없으면 20% → 50% → 100%로 순차 증가하는 스크립트와, Kiali(Istio 관찰 가능성 도구)에서 실시간 트래픽 비율을 확인하는 방법도 함께 제공한다.

---

### 4.5 AI 기반 Observability — AI가 이상을 감지하고 원인을 찾아준다

#### OpenTelemetry Collector 자동 배포

AI에게 다음과 같이 입력한다.

```
OTel Collector를 EKS에 배포해줘.
- Agent: DaemonSet (각 노드에서 로그/메트릭 수집)
- Gateway: Deployment (집계 후 Prometheus + Elasticsearch로 전송)
- K8s 메타데이터 자동 태깅 (Pod, 네임스페이스, 노드 이름)
```

AI는 두 계층의 OTel Collector 설정 YAML을 생성한다.

```yaml
# AI가 생성한 OTel Collector Agent (DaemonSet)
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-agent
spec:
  mode: daemonset
  config: |
    receivers:
      # 컨테이너 로그 수집
      filelog:
        include: [/var/log/pods/*/*/*.log]
        include_file_path: true
        operators:
          - type: container
            id: container-parser

      # Prometheus 메트릭 수집
      prometheus:
        config:
          scrape_configs:
          - job_name: 'kubernetes-pods'
            kubernetes_sd_configs:
            - role: pod
            relabel_configs:
            - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
              action: keep
              regex: true

    processors:
      # Pod/네임스페이스/노드 메타데이터 자동 태깅
      k8sattributes:
        auth_type: serviceAccount
        extract:
          metadata:
          - k8s.pod.name
          - k8s.namespace.name
          - k8s.node.name
          - k8s.deployment.name
      batch:
        timeout: 5s

    exporters:
      # Gateway로 전달
      otlp:
        endpoint: otel-gateway:4317
        tls:
          insecure: true

    service:
      pipelines:
        logs:
          receivers: [filelog]
          processors: [k8sattributes, batch]
          exporters: [otlp]
        metrics:
          receivers: [prometheus]
          processors: [k8sattributes, batch]
          exporters: [otlp]
```

#### AI 이상 감지가 실제로 동작하는 방식

Metoro, Coroot 같은 AI Observability 플랫폼은 다음과 같은 방식으로 이상을 감지하고 근본 원인을 찾는다.

```
[AI Observability 플랫폼의 실제 알림 예시]

🔴 이상 감지: payment-service 응답 지연 급증
시각: 2026-06-04 14:23:17 KST

지표 변화:
  - P99 응답시간: 120ms → 8,400ms (+6,900%)
  - 오류율: 0.1% → 18.3% (+183배)
  - 영향 Pod: payment-service-6d8f9b-xxx (3/3개 모두)

AI 근본 원인 분석:
  동일 시각에 다음 이벤트가 동시 발생했습니다:
  
  1. [14:22:51] mysql-0의 CPU 사용률 85% 초과
  2. [14:22:58] mysql-0 → payment-service 간 TCP 재전송률 급증
  3. [14:23:02] payment-service의 DB 연결 대기 큐 포화
  
  → 근본 원인: mysql-0 CPU 과부하로 인한 DB 응답 지연이
    payment-service의 연결 풀을 소진시킨 것으로 분석됩니다.

연관 이벤트:
  14:22:45 — report-generator CronJob 시작
  → 대용량 집계 쿼리가 MySQL CPU를 과점한 것으로 보입니다.

권장 조치:
  1. 즉시: report-generator CronJob 일시 중단
     kubectl delete job report-generator-28234560 -n production
  2. 단기: MySQL ResourceQuota에 CPU 제한 추가
  3. 장기: 리포팅 쿼리를 Read Replica로 분리
```

이처럼 AI Observability는 단순히 "payment-service에 오류가 있다"는 알림이 아니라, 인과 관계 체인을 자동으로 추론하여 근본 원인과 권장 조치를 자연어로 제시한다.

---

### 4.6 EFK 로그 파이프라인 — AI가 로그를 읽고 패턴을 분류한다

#### EFK의 각 구성 요소가 하는 일

EFK는 세 컴포넌트의 합성어다. **E(Elasticsearch)** 는 모든 로그를 저장하고 역색인(Inverted Index)으로 빠른 전문 검색을 제공한다. **F(Fluent Bit)** 는 DaemonSet으로 모든 노드에 배포되어 컨테이너 로그를 수집하고, Kubernetes 메타데이터(Pod 이름, 네임스페이스, 레이블 등)를 자동으로 붙인 뒤 Elasticsearch로 전송한다. **K(Kibana)** 는 Elasticsearch에 저장된 로그를 대시보드로 시각화하고 검색할 수 있는 웹 UI를 제공한다.

#### AI가 EFK 파이프라인에서 하는 역할

전통적인 EFK는 로그를 저장하고 검색하는 수동적인 도구다. 여기에 AI를 결합하면 다음 세 가지 능동적 기능이 추가된다.

**첫째, AI 기반 자연어 검색이다.** Kibana의 KQL(Kibana Query Language)을 모르는 개발자도 "지난 1시간 동안 payment 서비스에서 발생한 DB 연결 실패 로그를 보여줘"라고 입력하면 AI가 이를 KQL로 변환하여 검색한다.

```
자연어 입력:
"지난 30분 동안 production 네임스페이스에서 500 에러가 난 서비스 이름과 횟수 알려줘"

AI가 변환한 KQL 쿼리:
{
  "query": {
    "bool": {
      "must": [
        {"term": {"kubernetes.namespace_name": "production"}},
        {"term": {"http.response.status_code": 500}},
        {"range": {"@timestamp": {"gte": "now-30m", "lte": "now"}}}
      ]
    }
  },
  "aggs": {
    "by_service": {
      "terms": {"field": "kubernetes.labels.app"}
    }
  }
}

검색 결과:
  payment-service: 847건
  auth-service: 23건
  user-service: 5건
```

**둘째, AI 기반 로그 이상 감지다.** AI가 정상적인 로그 패턴을 학습한 뒤, 이상한 로그 급증이나 새로운 에러 패턴이 등장하면 자동으로 알림을 보낸다. 단순한 "에러 로그가 N개 이상이면 알림"이 아니라, 같은 에러가 반복되는 경우와 처음 보는 에러가 나타나는 경우를 구분하여, 처음 보는 에러에 높은 우선순위를 부여한다.

**셋째, AI 기반 Fluent Bit 파싱 규칙 자동 생성이다.** 커스텀 애플리케이션의 비정형 로그를 AI가 분석하여 자동으로 파싱 규칙을 생성한다.

```
AI에게 입력:
"이 로그 포맷을 파싱하는 Fluent Bit Parser를 만들어줘"
[2026-06-04 14:23:17] [WARN] payment-service@node1 | txId=TXN-992847 | 
  amount=50000 | currency=KRW | error=INSUFFICIENT_BALANCE | 
  userId=USR-12345 | latency=234ms

AI가 생성한 Fluent Bit Parser:
[PARSER]
    Name        payment_service_log
    Format      regex
    Regex       ^\[(?<timestamp>[^\]]+)\] \[(?<level>\w+)\] (?<service>[^@]+)@(?<node>\w+) \| txId=(?<tx_id>[^\|]+) \| amount=(?<amount>\d+) \| currency=(?<currency>\w+) \| error=(?<error>[^\|]+) \| userId=(?<user_id>[^\|]+) \| latency=(?<latency_ms>\d+)ms$
    Time_Key    timestamp
    Time_Format %Y-%m-%d %H:%M:%S
```

이 파싱 규칙이 적용되면 Kibana에서 `tx_id`, `amount`, `error`, `latency_ms` 같은 구조화된 필드로 로그를 검색하고 집계할 수 있게 된다.

```mermaid
graph TB
    subgraph "로그 소스"
        CONT[컨테이너 로그\n /var/log/pods]
        K8S_EVT[K8s 이벤트]
    end

    subgraph "Fluent Bit DaemonSet\n모든 노드에 1개씩"
        FB_COLLECT[로그 수집]
        FB_META[K8s 메타데이터 태깅\nPod명 / NS / 레이블]
        AI_PARSE[AI 생성 파싱 규칙 적용\n비정형 → 구조화]
        FB_SEND[Elasticsearch로 전송]
    end

    subgraph "Elasticsearch\nStatefulSet"
        ES_INDEX[로그 인덱싱\n역색인 저장]
        AI_ANOMALY[AI 이상 패턴 감지\n새 에러 / 급증 탐지]
    end

    subgraph "Kibana"
        AI_NL[AI 자연어 검색\n자연어 → KQL 변환]
        DASHBOARD[대시보드 / 알람]
    end

    CONT --> FB_COLLECT
    K8S_EVT --> FB_COLLECT
    FB_COLLECT --> FB_META --> AI_PARSE --> FB_SEND
    FB_SEND --> ES_INDEX
    ES_INDEX --> AI_ANOMALY --> DASHBOARD
    AI_NL --> ES_INDEX
    ES_INDEX --> DASHBOARD
```

---

## 5. 전체 흐름 요약 — AI가 개입하는 지점들

아래 표는 커리큘럼 전체에서 AI가 개입하는 모든 지점을 "입력 → 출력 → 활용" 형태로 정리한 것이다.

| 파트 | 주제 | AI 입력 | AI 출력 | 활용 방식 |
|------|------|---------|---------|---------|
| 1 | EKS 아키텍처 | 자연어 요구사항 | Terraform/eksctl 코드 | 검토 후 `terraform apply` |
| 1 | Deployment | 자연어 + 스펙 | Deployment YAML (보안/프로브 포함) | 검토 후 `kubectl apply` |
| 1 | Ingress/Service | 라우팅 요구사항 | Ingress + Service YAML | 검토 후 `kubectl apply` |
| 1 | StorageClass | 워크로드 특성 | StorageClass + PVC YAML | 검토 후 `kubectl apply` |
| 2 | DaemonSet | 수집 대상 + 출력 | DaemonSet + ConfigMap YAML | 검토 후 `kubectl apply` |
| 2 | CronJob | 스케줄 + 작업 내용 | CronJob YAML (concurrencyPolicy 등) | 검토 후 `kubectl apply` |
| 2 | StatefulSet | DB 구성 요구사항 | StatefulSet + PVC Template + NetworkPolicy | 검토 후 `kubectl apply` |
| 2 | Scheduling | 워크로드 특성 설명 | Affinity/Taint YAML 블록 | 기존 Deployment에 병합 |
| 2 | ResourceQuota | 팀별 자원 비율 | Quota + LimitRange YAML | 검토 후 `kubectl apply` |
| 3 | K8sGPT | `k8sgpt analyze --explain` | 자연어 진단서 + 수정 명령 | 제시된 명령 실행 |
| 3 | kubectl-ai | 자연어 질문/지시 | kubectl 명령 생성 + 실행(확인 후) | 확인 후 자동 실행 |
| 3 | Helm 리팩토링 | 레거시 Helm 차트 | 리팩토링된 차트 + values.yaml | 검토 후 `helm upgrade` |
| 3 | Istio | Canary 요구사항 | VirtualService + DestinationRule YAML | 검토 후 `kubectl apply` |
| 3 | Observability | OTel 요구사항 | OTel Collector 배포 YAML | 검토 후 `kubectl apply` |
| 3 | EFK | 로그 포맷 설명 | Fluent Bit Parser 규칙 / KQL 쿼리 | Kibana에 직접 적용 |

### AI 활용의 일관된 원칙

이 커리큘럼 전체를 통해 AI 활용에는 일관된 원칙이 있다. AI가 생성한 결과물은 반드시 엔지니어가 검토한 후 적용한다. K8sGPT나 kubectl-ai가 제시하는 수정 명령도, AI가 자동으로 실행하지 않고 엔지니어의 확인을 받은 뒤 실행한다. AI는 전문 엔지니어를 대체하는 것이 아니라, 반복적이고 시간 소모적인 작업을 자동화하여 엔지니어가 더 중요한 의사결정에 집중할 수 있게 해주는 도구다.

---

> **작성일:** 2026-06-04  
> **근거 자료:** K8sGPT 공식 문서, GoogleCloudPlatform/kubectl-ai GitHub (DeepWiki), AWS EKS Auto Mode 공식 블로그, Istio KubeCon EU 2026 발표, OpenTelemetry 2025 서베이, Palark 기술 블로그 (K8sGPT 실사용 예시), Kubezilla (Claude Code + K8s 워크플로우)
