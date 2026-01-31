---
title: "Clopus-Watcher: Claude Code 기반 자율 Kubernetes 모니터링 에이전트 완벽 가이드"
date: 2026-01-31 10:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  Clopus-Watcher,  Kubernetes,  monitoring-agents,  Claude.write]
---


## 개요

Clopus-Watcher는 Claude Code를 활용한 혁신적인 Kubernetes 네이티브 모니터링 및 자동 복구 시스템입니다. 이 도구는 전통적인 온콜 엔지니어의 역할을 AI로 대체하려는 대담한 시도로, 24시간 7일 동안 자율적으로 Pod 상태를 모니터링하고, 오류를 감지하며, 필요시 자동으로 핫픽스를 적용하는 완전 자동화된 시스템입니다.

프로젝트는 Kubernetes 환경에서 CronJob 형태로 주기적으로 실행되며, 운영자가 부재중인 시간에도 시스템의 안정성을 유지할 수 있도록 설계되었습니다. 단순한 모니터링을 넘어서, 실제로 Pod 내부에 진입하여 문제를 진단하고 수정까지 수행하는 진정한 의미의 자율 에이전트입니다.

### 핵심 가치 제안

Clopus-Watcher는 다음과 같은 근본적인 문제 의식에서 출발했습니다. 현재 AI와 개발에 관한 대부분의 논의는 "개발자가 대체될 것"이라는 우려에 집중되어 있습니다. 하지만 프로젝트 창시자인 Denislav Gavrilov는 정반대의 관점을 제시합니다. 24시간 모니터링 업무, 특히 시스템 관리자나 온콜 엔지니어의 작업이 실제로는 개발보다 훨씬 더 체계적이고 자동화하기 쉬운 영역이라는 것입니다.

온콜 엔지니어의 작업은 대부분 다음과 같은 패턴을 따릅니다. 시스템을 모니터링하고, 문제가 발생하면 문서화된 절차를 따라 시스템을 복구하며, 실패할 경우 백업 및 복구 계획을 실행하거나 상급 엔지니어에게 에스컬레이션합니다. 각 세션이 끝나면 보고서를 작성하고 인시던트를 아카이브합니다. 이러한 체계적인 프로세스는 자동화에 매우 적합합니다.

Clopus-Watcher는 바로 이 점을 증명합니다. 온콜 엔지니어가 하는 일, 즉 모니터링, 탐색, 이해, 수정, 보고서 작성을 자율적으로, 그리고 영원히 수행할 수 있습니다. 사람과 달리 교대 근무나 수면이 필요 없는 AI 에이전트는 진정한 의미의 24/7 운영을 가능하게 합니다.

## 프로젝트 배경: Clopus 시리즈의 진화

Clopus-Watcher를 제대로 이해하기 위해서는 그 선행 프로젝트들을 살펴볼 필요가 있습니다. 이 프로젝트는 "Clopus" 시리즈의 세 번째 단계로, 점진적으로 Claude Code의 자율성을 확장해온 실험의 정점입니다.

### Clopus-01: 반자율 Claude Code

첫 번째 실험에서 Denislav는 DigitalOcean에 새로운 Linux 가상 머신을 프로비저닝하고 Claude Code를 설치한 후, 전체 운영 체제에 대한 완전한 권한을 부여했습니다. 설정 파일(`/home/.claude/settings.local.json`)에서 Bash, Read, Write, Edit 권한을 모두 허용하여 Claude Code가 시스템 전체를 제어할 수 있도록 했습니다.

핵심 아이디어는 "Brain Claude"와 "Worker Claude"라는 이중 구조였습니다. Brain Claude는 오케스트레이터 역할로 5분마다 cron job을 통해 실행되며, Worker Claude에게 지시를 내립니다. Worker Claude는 tmux 세션에서 실행되어 ttyd를 통해 clopus.live에서 공개적으로 스트리밍됩니다. 두 Claude는 텍스트 파일(`/opt/claude/comms/from_brain.txt`, `/opt/claude/comms/to_brain.txt`)을 통해 통신합니다.

이 실험은 약 8-10시간 동안 실행되었으며, Worker Claude가 자율적으로 선택하고 배포한 프로젝트는 37개에 달했고, 첫 24시간 동안 749,280건의 웹 요청을 처리했습니다.

### Clopus-02: 24시간 자율 실행

두 번째 실험은 더 야심찼습니다. 목표는 Claude Code 인스턴스가 어떠한 인간의 개입 없이도 영원히 실행되고, 스스로 무엇을 할지 결정할 수 있도록 하는 것이었습니다.

이를 위해 단기 메모리로 SQLite3, 장기 메모리로 Qdrant 벡터 데이터베이스를 추가했습니다. 또한 Chromium을 설치하여 브라우저 접근을 제공했습니다. 마스터 프롬프트는 "Autonomous Claude"라는 자기 지시형 AI 에이전트로서의 정체성을 부여하고, 의사결정 루프를 정의했습니다.

1. 단기 메모리 읽기 (최근 컨텍스트)
2. 장기 메모리 쿼리 (과거 학습의 시맨틱 검색)
3. 다음에 무엇을 할지 사고
4. 행동 실행
5. 기록 (단기 메모리에 쓰기)
6. 브라우저 작업 시 스크린샷 저장
7. 중요한 학습이 있으면 장기 메모리에 임베딩 및 저장

이 시스템은 인간의 개입 없이 24시간 동안 성공적으로 실행되었으며, 토큰을 보존하기 위해 수동으로 중지할 때까지 계속 작동했습니다.

### Clopus-Watcher: 실용적 응용

Clopus-02의 성공은 중요한 깨달음을 가져왔습니다. "영원히 실행되는 Claude Code"라는 개념은 흥미롭지만, "Watcher" 개념은 실제 세계에 즉시 적용 가능한 잠재력을 가지고 있었습니다. Denislav는 자신의 첫 직장인 시스템 관리자로서 했던 일이 Clopus-Watcher가 하는 일과 거의 1:1로 일치한다는 것을 깨달았습니다.

현재 상태의 Clopus-Watcher를 실제 프로덕션 시스템에 맡기는 것은 무모할 수 있지만, 이 방향성은 온콜 엔지니어의 업무를 크게 줄이거나 아예 대체할 수 있는 가능성을 보여줍니다. 이는 단순한 가정이 아니라, 실제 클러스터에서의 결과를 통해 입증된 현실입니다.

## 아키텍처 및 작동 원리

### 시스템 구성요소

Clopus-Watcher는 Kubernetes 네이티브 애플리케이션으로 설계되었으며, 다음과 같은 주요 구성요소로 이루어져 있습니다.

**1. Watcher CronJob (모니터링 엔진)**

핵심 컴포넌트인 Watcher는 Kubernetes CronJob으로 배포되며, 설정된 주기(예: 5분마다)로 실행됩니다. 각 실행 시 다음 작업을 수행합니다.

- 대상 네임스페이스의 모든 Pod 상태를 조회합니다
- CrashLoopBackOff, Error, Pending 등 비정상 상태의 Pod를 식별합니다
- 비정상 Pod의 로그를 읽어 오류의 근본 원인을 분석합니다
- 설정된 모드에 따라 다음 중 하나를 실행합니다
  - **Autonomous Mode**: Pod 내부로 진입(`kubectl exec -- /bin/bash`)하여 직접 핫픽스를 적용하거나 `kubectl` 명령을 실행하여 문제를 해결합니다
  - **Watcher Mode**: 문제를 진단하고 가능한 수정 방법을 기록하지만 실제로 적용하지는 않습니다
- 모든 활동을 SQLite 데이터베이스에 기록합니다
- 구조화된 JSON 형식의 보고서를 생성합니다

**2. Dashboard Deployment (시각화 인터페이스)**

Go와 HTML로 구성된 경량 웹 애플리케이션으로, Watcher가 기록한 모든 데이터를 시각화합니다.

- 실시간 모니터링 대시보드를 제공합니다
- 감지된 오류 목록과 상세 정보를 표시합니다
- 적용된 수정 사항의 이력을 추적합니다
- 각 실행(Run)의 성공/실패 상태를 보여줍니다
- 시간대별 통계 및 트렌드를 제공합니다

**3. 데이터 저장소 (SQLite + PVC)**

모든 모니터링 데이터와 수정 이력은 SQLite 데이터베이스에 저장되며, Kubernetes PersistentVolumeClaim(PVC)을 통해 영속성이 보장됩니다. 데이터베이스 스키마는 다음 정보를 포함합니다.

- Run ID: 각 실행의 고유 식별자
- 타임스탬프: 실행 시간
- 대상 네임스페이스
- Pod 이름 및 상태
- 감지된 문제
- 적용된 조치
- 결과 (성공/실패)
- 상세 로그

### Kubernetes 리소스 구성

프로젝트를 배포하기 위해 필요한 Kubernetes 리소스는 다음과 같습니다.

**ServiceAccount**
Watcher가 Kubernetes API와 상호작용하기 위한 서비스 계정입니다.

**ClusterRole**
Pod 조회, 로그 읽기, exec 실행, 이벤트 접근 등의 권한을 정의합니다.

**RoleBinding**
ServiceAccount에 ClusterRole을 바인딩합니다.

**CronJob**
Watcher 컨테이너를 주기적으로 실행합니다. 주요 환경 변수는 다음과 같습니다.
- `TARGET_NAMESPACE`: 모니터링할 네임스페이스 (기본값: default)
- `AUTH_MODE`: 인증 방법, api-key 또는 credentials
- `WATCHER_MODE`: 작동 모드, autonomous 또는 watcher
- `ANTHROPIC_API_KEY`: Claude API 키 (AUTH_MODE=api-key인 경우)
- `SQLITE_PATH`: SQLite 데이터베이스 경로

**Deployment (Dashboard)**
웹 대시보드를 실행하는 Deployment입니다.

**PersistentVolumeClaim**
데이터베이스를 위한 영속 스토리지입니다.

**Service**
Dashboard를 노출하는 서비스입니다.

**Ingress (선택사항)**
외부에서 Dashboard에 접근하기 위한 Ingress 리소스입니다. Gateway API로의 마이그레이션을 고려할 수 있습니다.

### 작동 흐름

전체 시스템의 작동 흐름은 다음과 같습니다.

1. **트리거**: Kubernetes CronJob이 설정된 스케줄에 따라 Watcher Pod를 생성합니다.

2. **초기화**: Watcher 컨테이너가 시작되고 Claude Code가 초기화됩니다. 인증은 API 키 또는 credentials.json 파일을 통해 이루어집니다.

3. **마스터 프롬프트 로딩**: 설정된 모드(autonomous/watcher)에 따라 적절한 마스터 프롬프트가 로드됩니다. 이 프롬프트는 Claude에게 온콜 엔지니어로서의 역할과 책임을 정의합니다.

4. **Pod 스캐닝**: `kubectl get pods -n TARGET_NAMESPACE`를 실행하여 대상 네임스페이스의 모든 Pod 상태를 확인합니다.

5. **이상 감지**: Pod 상태를 분석하여 다음과 같은 비정상 상태를 식별합니다.
   - CrashLoopBackOff
   - Error
   - Pending
   - ImagePullBackOff
   - OOMKilled
   - 재시작 횟수 증가

6. **로그 분석**: 비정상 Pod에 대해 `kubectl logs <pod-name> -n <namespace>`를 실행하여 최근 로그를 읽고, Claude가 오류의 근본 원인을 파악합니다.

7. **진단 및 결정**: Claude는 로그와 Pod 상태를 분석하여 문제의 원인을 진단하고, 가능한 해결책을 결정합니다.

8. **조치 실행** (Autonomous Mode인 경우):
   - **Pod 내부 수정이 필요한 경우**: `kubectl exec -it <pod-name> -n <namespace> -- /bin/bash`를 통해 Pod에 진입하여 파일을 편집하거나 설정을 변경합니다.
   - **Pod 외부 조작이 필요한 경우**: `kubectl` 명령을 직접 실행하여 Pod를 재시작하거나, ConfigMap을 수정하거나, 기타 필요한 작업을 수행합니다.

9. **검증**: 수정 후 문제가 해결되었는지 확인합니다. Pod 상태를 다시 확인하고, 로그에 오류가 사라졌는지 검증합니다.

10. **기록**: 모든 활동, 발견한 문제, 적용한 수정, 결과를 SQLite 데이터베이스에 기록합니다.

11. **보고서 생성**: 구조화된 JSON 형식의 보고서를 생성합니다. 보고서에는 다음이 포함됩니다.
    ```json
    {
      "pod_count": 1,
      "error_count": 0,
      "fix_count": 0,
      "status": "ok",
      "summary": "All pods in namespace are healthy",
      "details": [
        {
          "pod": "app-pod-xyz",
          "issue": "none",
          "action": "Monitored logs and status - all systems healthy",
          "result": "success"
        }
      ]
    }
    ```

12. **종료**: CronJob Pod가 정상적으로 종료되고, 다음 스케줄까지 대기합니다.

13. **시각화**: Dashboard는 주기적으로 SQLite 데이터베이스를 쿼리하여 최신 데이터를 웹 UI에 표시합니다.

### 마스터 프롬프트

Clopus-Watcher의 핵심은 Claude에게 제공되는 마스터 프롬프트입니다. 이 프롬프트는 Claude를 온콜 엔지니어로 변환하며, 다음과 같은 지침을 포함합니다.

- 역할 정의: 당신은 24/7 자율 온콜 엔지니어입니다
- 작업 범위: 지정된 네임스페이스의 Pod를 모니터링합니다
- 감지 기준: 비정상 상태(CrashLoopBackOff, Error 등)를 식별합니다
- 분석 방법: 로그를 읽고 근본 원인을 파악합니다
- 조치 권한: (Autonomous Mode) 문제를 직접 수정할 수 있습니다
- 검증 책임: 수정이 효과적인지 확인합니다
- 보고 의무: 모든 활동을 구조화된 형식으로 보고합니다
- 제약 조건: 프로덕션 시스템에 손상을 주지 않도록 주의합니다

프롬프트는 GitHub 저장소의 `/watcher` 디렉토리에서 확인할 수 있으며, 이미지를 다시 빌드하여 커스터마이징할 수 있습니다.

## 설치 및 구성

### 사전 요구사항

**클러스터 환경**
- Kubernetes 클러스터 (1.20 이상 권장)
- Sealed Secrets 또는 기타 Secret 관리 솔루션

**로컬 환경 (이미지 빌드 및 배포용)**
- Podman, Docker 또는 기타 컨테이너 런타임
- kubectl CLI
- 컨테이너 레지스트리 접근 권한

**Claude 인증**
- Anthropic API 키 또는
- Claude Code credentials.json 파일

### 환경 변수 구성

시스템 동작을 제어하는 주요 환경 변수는 다음과 같습니다.

| 변수명 | 설명 | 기본값 | 가능한 값 |
|--------|------|--------|-----------|
| `TARGET_NAMESPACE` | 모니터링할 Kubernetes 네임스페이스 | `default` | 임의의 네임스페이스명 |
| `AUTH_MODE` | Claude 인증 방법 | `api-key` | `api-key`, `credentials` |
| `WATCHER_MODE` | Watcher 작동 모드 | `autonomous` | `autonomous`, `watcher` |
| `ANTHROPIC_API_KEY` | Claude API 키 (AUTH_MODE=api-key 시) | - | `sk-ant-xxxxx` |
| `SQLITE_PATH` | SQLite 데이터베이스 경로 | `/data/watcher.db` | 파일 경로 |

**AUTH_MODE 선택 가이드**

- **api-key**: 가장 간단한 방법입니다. Anthropic API 키만 있으면 됩니다. 프로덕션 환경에 권장됩니다.
- **credentials**: Claude Code의 OAuth 인증을 사용합니다. `~/.claude/.credentials.json` 파일이 필요합니다.

**WATCHER_MODE 선택 가이드**

- **autonomous**: Watcher가 문제를 감지하면 자동으로 수정을 시도합니다. 위험도가 높지만 진정한 자동화를 제공합니다.
- **watcher**: 문제를 감지하고 보고하지만 실제로 수정하지는 않습니다. 안전하며 초기 테스트에 적합합니다.

### 배포 방법

#### 옵션 1: API 키 사용 (권장)

이 방법이 가장 간단하며 프로덕션 환경에 권장됩니다.

**1단계: 네임스페이스 생성**

```bash
kubectl create namespace clopus-watcher
```

**2단계: Secret 생성**

API 키를 Kubernetes Secret에 저장합니다.

```bash
kubectl create secret generic claude-auth \
  --namespace clopus-watcher \
  --from-literal=api-key=sk-ant-xxxxx
```

실제 API 키로 `sk-ant-xxxxx`를 교체하세요.

**3단계: 구성 확인**

`k8s/cronjob.yaml` 파일에서 다음을 확인합니다.
- `AUTH_MODE` 환경 변수가 `api-key`로 설정되어 있는지 (기본값)
- Secret 이름이 `claude-auth`와 일치하는지

**4단계: 배포**

제공된 배포 스크립트를 실행합니다.

```bash
./scripts/deploy.sh
```

이 스크립트는 다음을 수행합니다.
- 컨테이너 이미지를 빌드합니다 (Watcher와 Dashboard)
- 이미지를 컨테이너 레지스트리에 푸시합니다
- Kubernetes 리소스를 적용합니다 (ServiceAccount, ClusterRole, RoleBinding, CronJob, Deployment, PVC, Service, Ingress)

#### 옵션 2: Credentials 파일 사용 (OAuth)

Claude Code의 OAuth 인증을 사용하려면 이 방법을 선택하세요.

**1단계: 네임스페이스 생성**

```bash
kubectl create namespace clopus-watcher
```

**2단계: Credentials Secret 생성**

로컬의 Claude credentials 파일에서 Secret을 생성합니다.

```bash
kubectl create secret generic claude-credentials \
  --namespace clopus-watcher \
  --from-file=credentials.json=$HOME/.claude/.credentials.json
```

**3단계: CronJob 매니페스트 수정**

`k8s/cronjob.yaml` 파일을 편집합니다.

- `AUTH_MODE` 환경 변수를 `credentials`로 변경합니다
- `claude-credentials` volume과 volumeMount 섹션의 주석을 제거합니다

예시:
```yaml
env:
  - name: AUTH_MODE
    value: "credentials"  # api-key에서 변경
# ...
volumes:
  - name: claude-credentials  # 주석 제거
    secret:
      secretName: claude-credentials
# ...
volumeMounts:
  - name: claude-credentials  # 주석 제거
    mountPath: /root/.claude
    readOnly: true
```

**4단계: 배포**

```bash
./scripts/deploy.sh
```

### 배포 후 검증

배포가 성공적으로 완료되었는지 확인합니다.

**1. CronJob 확인**

```bash
kubectl get cronjob -n clopus-watcher
```

예상 출력:
```
NAME             SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
clopus-watcher   */5 * * * *   False     0        2m              10m
```

**2. 최근 실행 확인**

```bash
kubectl get jobs -n clopus-watcher
```

CronJob이 최소 한 번 실행되었다면 Job이 표시됩니다.

**3. 로그 확인**

최근 실행의 로그를 확인합니다.

```bash
# 최근 Job의 Pod 이름 가져오기
POD_NAME=$(kubectl get pods -n clopus-watcher -l job-name --sort-by=.metadata.creationTimestamp -o jsonpath='{.items[-1].metadata.name}')

# 로그 확인
kubectl logs -n clopus-watcher $POD_NAME
```

성공적인 실행은 다음과 유사한 출력을 보여줍니다:
```
=== Run #1 started at 2026-01-31T10:00:00+00:00 ===
Mode: autonomous | Namespace: default
----------------------------------------
Scanning pods in namespace 'default'...
Found 3 pods, all healthy
No errors detected

===REPORT_START===
{
  "pod_count": 3,
  "error_count": 0,
  "fix_count": 0,
  "status": "ok",
  "summary": "All pods healthy"
}
===REPORT_END===

=== Run #1 Complete ===
```

**4. Dashboard 접근**

Dashboard Service를 확인합니다.

```bash
kubectl get svc -n clopus-watcher
```

Ingress를 설정했다면 해당 도메인으로 접근합니다. 그렇지 않으면 포트 포워딩을 사용합니다.

```bash
kubectl port-forward -n clopus-watcher svc/dashboard 8080:80
```

브라우저에서 `http://localhost:8080`으로 접근하여 Dashboard를 확인합니다.

### 커스터마이징

#### CronJob 스케줄 변경

모니터링 주기를 변경하려면 `k8s/cronjob.yaml`의 `schedule` 필드를 수정합니다.

```yaml
spec:
  schedule: "*/5 * * * *"  # 5분마다 (기본값)
  # schedule: "*/10 * * * *"  # 10분마다
  # schedule: "0 * * * *"     # 매시간
  # schedule: "0 */2 * * *"   # 2시간마다
```

변경 후 재배포:
```bash
kubectl apply -f k8s/cronjob.yaml
```

#### 대상 네임스페이스 변경

모니터링할 네임스페이스를 변경하려면 `k8s/cronjob.yaml`의 `TARGET_NAMESPACE` 환경 변수를 수정합니다.

```yaml
env:
  - name: TARGET_NAMESPACE
    value: "production"  # default에서 변경
```

#### 마스터 프롬프트 커스터마이징

Claude에게 제공되는 지침을 수정하려면:

1. `/watcher` 디렉토리의 프롬프트 파일을 편집합니다
2. Docker 이미지를 다시 빌드합니다
3. 새 이미지로 CronJob을 업데이트합니다

```bash
# 이미지 빌드
docker build -f Dockerfile.watcher -t your-registry/clopus-watcher:custom .

# 이미지 푸시
docker push your-registry/clopus-watcher:custom

# CronJob 이미지 업데이트
kubectl set image cronjob/clopus-watcher -n clopus-watcher \
  watcher=your-registry/clopus-watcher:custom
```

## 실제 사용 사례 및 데모

프로젝트 창시자인 Denislav Gavrilov는 실제 환경에서 Clopus-Watcher를 테스트하여 그 효과를 입증했습니다. 두 가지 주요 시나리오가 문서화되어 있습니다.

### 사례 1: 메모리 누수 및 메트릭 서버 부재

**시나리오 설정**

Denislav는 자신의 애플리케이션인 txtwrite의 의도적으로 버그가 있는 이미지를 배포했습니다. 이 버그는 다음과 같은 문제를 포함했습니다.

1. **클라이언트 측 메모리 누수**: React의 `useEffect` 훅이 cleanup 함수 없이 구현되어, 컴포넌트가 언마운트될 때 메모리 누수가 발생합니다.

2. **메트릭 엔드포인트 오류**: 텔레메트리 서비스가 메모리 누수를 감지하고 존재하지 않는 `/metrics` 엔드포인트로 메트릭을 푸시하려고 시도하여 반복적인 오류가 발생합니다.

**Watcher의 반응**

먼저 정상 이미지가 실행 중일 때, Watcher는 아무런 오류도 발견하지 못하고 다음과 같이 보고했습니다.

```
=== Run #42 started at 2025-12-28T14:30:00+00:00 ===
Mode: autonomous | Namespace: txtwrite
----------------------------------------
Scanning pods...
All pods healthy, no action required

===REPORT===
{
  "pod_count": 1,
  "error_count": 0,
  "fix_count": 0,
  "status": "ok",
  "summary": "All systems nominal"
}
```

그러나 버그가 있는 이미지로 전환한 후, Watcher는 즉시 문제를 감지했습니다.

```
=== Run #43 started at 2025-12-28T14:35:00+00:00 ===
Mode: autonomous | Namespace: txtwrite
----------------------------------------
Detected errors in pod txtwrite-6674cb8f65-xyz:
- Repeated 404 errors on /metrics endpoint
- Memory usage increasing over time

Analyzing logs...
Root cause identified:
1. Telemetry service attempting to push to non-existent /metrics
2. React component cleanup missing, causing memory leak

Attempting fixes...
1. Exec into pod to add /metrics endpoint stub
2. Cannot fix React code without redeployment
3. Attempted pod restart to clear memory

Result: Partial fix
- /metrics errors stopped
- Memory leak persists (requires code fix)

Marking as: FAILED (hotfix insufficient)
```

**분석**

흥미롭게도, Watcher는 실패로 표시했지만 실제로는 `/metrics` 오류를 성공적으로 중지시켰습니다. 실패로 판단한 이유는 Pod를 재시작하려고 시도했을 때 적용했던 핫픽스가 초기 상태로 돌아가 사라졌기 때문입니다.

Watcher의 마스터 프롬프트에는 "문제가 완전히 해결될 때까지 작업하라"는 지침이 포함되어 있어, 부분적인 해결책을 완전한 성공으로 간주하지 않았습니다. 이는 시스템의 엄격한 품질 기준을 보여줍니다.

### 사례 2: Health Check 파일 삭제

**시나리오 설정**

두 번째 테스트는 더 간단하지만 실제 환경에서 자주 발생할 수 있는 시나리오를 시뮬레이션했습니다. 버그가 있는 코드가 예상치 못한 결과를 초래하는 상황입니다.

애플리케이션의 동작은 다음과 같습니다.

1. 서버는 5초마다 `/tmp/healthy` 파일의 존재를 확인합니다
2. 파일이 존재하면 → `[OK] System healthy` 로그를 기록합니다
3. 파일이 없으면 → `[ERROR] Health check failed` 로그를 기록합니다
4. 클라이언트의 특정 버튼을 누르면 `/tmp/healthy` 파일이 삭제됩니다 (버그)

**예상되는 수정 방법**

```bash
touch /tmp/healthy  # Pod 내부에서 실행
```

**Watcher의 반응**

Watcher는 이 문제를 완벽하게 처리했습니다.

```
=== Run #44 started at 2025-12-28T15:00:00+00:00 ===
Mode: autonomous | Namespace: txtwrite
----------------------------------------

Detected pod error: txtwrite-6674cb8f65-abc
Status: Running but logging errors

Analyzing logs...
Recent logs show:
[ERROR] Health check failed: /tmp/healthy not found
[ERROR] Health check failed: /tmp/healthy not found
[ERROR] Health check failed: /tmp/healthy not found

Root cause analysis:
- Health check expects /tmp/healthy file
- File is missing
- Error pattern started at 14:57:32 (user action correlation)

Hypothesis: User action deleted the health check file

Applying fix...
Executing: kubectl exec txtwrite-6674cb8f65-abc -n txtwrite -- touch /tmp/healthy

Verifying fix...
Waiting 10 seconds...
Checking logs...

Success! Logs now show:
[OK] System healthy
[OK] System healthy
[OK] System healthy

===REPORT===
{
  "pod_count": 1,
  "error_count": 1,
  "fix_count": 1,
  "status": "fixed",
  "summary": "Health check file restored",
  "details": [
    {
      "pod": "txtwrite-6674cb8f65-abc",
      "issue": "Missing /tmp/healthy file",
      "action": "kubectl exec to recreate file",
      "result": "success"
    }
  ]
}
```

**분석**

이 경우 Watcher는 다음과 같은 인상적인 능력을 보여주었습니다.

1. **근본 원인 분석**: 단순히 오류를 감지하는 것을 넘어, 로그 패턴을 분석하여 사용자 작업과의 상관관계를 파악했습니다.

2. **최소 침습 수정**: Pod 내부로 진입하여 복잡한 작업을 하는 대신, 간단한 `kubectl exec` 명령으로 문제를 해결했습니다.

3. **검증**: 수정 후 실제로 문제가 해결되었는지 로그를 다시 확인하여 검증했습니다.

4. **구조화된 보고**: 문제, 조치, 결과를 명확하게 문서화했습니다.

### 라이브 데모

프로젝트는 공개 대시보드를 제공하여 실시간으로 Clopus-Watcher의 작동을 확인할 수 있습니다: https://k8s-watcher.clopus.live

이 대시보드에서는 다음을 볼 수 있습니다.
- 최근 실행 목록
- 각 실행의 상세 정보
- 감지된 문제와 적용된 수정
- 시간대별 성공/실패 통계
- 실시간 모니터링 상태

## 기술 스택 및 구현 세부사항

### 컨테이너 이미지

프로젝트는 두 개의 별도 Docker 이미지를 사용합니다.

**Watcher 이미지 (Dockerfile.watcher)**
- 베이스: Claude Code를 포함한 커스텀 이미지
- 포함 내용:
  - Claude Code CLI
  - kubectl 바이너리
  - 마스터 프롬프트 파일
  - Watcher 실행 스크립트
  - SQLite 클라이언트
- 크기: 경량화를 위해 멀티 스테이지 빌드 사용

**Dashboard 이미지 (Dockerfile.dashboard)**
- 베이스: Go Alpine
- 포함 내용:
  - Go로 작성된 웹 서버
  - HTML 템플릿
  - SQLite 드라이버
- 크기: 10-20MB (경량)

### 프로그래밍 언어 및 도구

**Go (40.8%)**
Dashboard 백엔드는 Go로 작성되었습니다. Go의 장점:
- 정적 바이너리 생성으로 배포 간소화
- 우수한 성능
- 내장된 HTTP 서버
- 컴파일 시간이 빠름

주요 패키지:
- `net/http`: 웹 서버
- `html/template`: 템플릿 렌더링
- `database/sql`: SQLite 데이터베이스 접근
- `time`: 타임스탬프 처리

**HTML (38.1%)**
Dashboard의 프론트엔드는 순수 HTML과 최소한의 JavaScript로 구성되어 있습니다. 복잡한 프레임워크 대신 단순성과 성능을 우선합니다.

특징:
- 서버 사이드 렌더링
- 자동 새로고침 (meta refresh 또는 JavaScript)
- 반응형 디자인
- 최소한의 의존성

**Shell (21.1%)**
배포 및 유틸리티 스크립트는 Shell로 작성되었습니다.

주요 스크립트:
- `deploy.sh`: 전체 배포 자동화
- 빌드 스크립트
- 환경 설정 스크립트

### 데이터베이스 스키마

SQLite 데이터베이스는 다음과 같은 테이블 구조를 가집니다 (추정).

**runs 테이블**
```sql
CREATE TABLE runs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id INTEGER NOT NULL,
    started_at TIMESTAMP NOT NULL,
    completed_at TIMESTAMP,
    namespace VARCHAR(255) NOT NULL,
    mode VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL,
    pod_count INTEGER,
    error_count INTEGER,
    fix_count INTEGER,
    summary TEXT
);
```

**fixes 테이블**
```sql
CREATE TABLE fixes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id INTEGER NOT NULL,
    pod_name VARCHAR(255) NOT NULL,
    issue TEXT NOT NULL,
    action TEXT NOT NULL,
    result VARCHAR(50) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    logs TEXT,
    FOREIGN KEY (run_id) REFERENCES runs(run_id)
);
```

### 보안 고려사항

**1. RBAC (Role-Based Access Control)**

Watcher는 최소 권한 원칙을 따릅니다. ClusterRole은 다음 권한만 부여합니다.
- pods: get, list, watch
- pods/log: get
- pods/exec: create (Autonomous Mode에서만 필요)
- events: get, list

**2. Secret 관리**

민감한 정보(API 키, credentials)는 Kubernetes Secret에 저장됩니다. 프로덕션에서는 다음을 고려하세요.
- Sealed Secrets 사용
- External Secrets Operator
- HashiCorp Vault 통합

**3. 네트워크 정책**

필요에 따라 NetworkPolicy를 적용하여 Watcher의 네트워크 접근을 제한할 수 있습니다.

**4. Pod Security Standards**

Dashboard와 Watcher Pod는 다음 보안 컨텍스트를 설정할 수 있습니다.
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: true
```

**5. 감사 로깅**

모든 Watcher 활동은 SQLite에 기록되어 감사 추적을 제공합니다.

## 장점 및 혁신적 측면

### 1. 진정한 자율성

Clopus-Watcher는 단순한 자동화를 넘어 진정한 자율성을 구현합니다. 미리 정의된 규칙이나 플레이북을 따르는 대신, Claude가 각 상황을 독립적으로 분석하고 결정합니다. 이는 다음을 의미합니다.

- **적응성**: 이전에 본 적 없는 오류도 처리할 수 있습니다
- **컨텍스트 이해**: 로그와 시스템 상태를 종합적으로 분석합니다
- **창의적 문제 해결**: 여러 가능한 해결책 중에서 최선을 선택합니다

### 2. 경량 아키텍처

전체 시스템은 Go, HTML, Shell만으로 구성되어 있어 매우 경량입니다.

- **작은 리소스 풋프린트**: Dashboard는 최소한의 CPU와 메모리만 사용합니다
- **빠른 시작 시간**: CronJob Pod는 몇 초 안에 시작되고 작업을 완료합니다
- **간단한 배포**: 복잡한 의존성이 없습니다

### 3. Kubernetes 네이티브

Kubernetes의 네이티브 리소스(CronJob, ServiceAccount, RBAC)를 활용하여 플랫폼과 자연스럽게 통합됩니다.

- **스케줄링**: CronJob의 안정적인 스케줄링 사용
- **권한 관리**: RBAC를 통한 세밀한 권한 제어
- **서비스 디스커버리**: Kubernetes Service와 DNS 활용
- **스토리지**: PVC를 통한 영속성

### 4. 투명성

모든 활동이 기록되고 Dashboard를 통해 시각화됩니다.

- **완전한 감사 추적**: 누가, 언제, 무엇을, 왜 했는지 추적
- **실시간 모니터링**: 현재 시스템 상태를 즉시 확인
- **이력 분석**: 과거 문제와 해결책 검토

### 5. 모드 유연성

Autonomous와 Watcher 모드를 제공하여 다양한 사용 사례를 지원합니다.

- **초기 테스트**: Watcher 모드로 안전하게 시작
- **점진적 신뢰 구축**: 결과를 검토한 후 Autonomous로 전환
- **감사 및 학습**: Watcher 모드로 AI의 판단 품질 평가

### 6. 확장 가능성

기본 구조는 확장 가능하도록 설계되었습니다.

- **멀티 네임스페이스**: 여러 네임스페이스를 동시에 모니터링하도록 확장 가능
- **커스텀 프롬프트**: 특정 애플리케이션이나 산업에 맞게 조정 가능
- **통합**: Slack, PagerDuty 등과의 통합 추가 가능

## 제한사항 및 고려사항

### 1. AI 판단의 신뢰성

Claude는 매우 강력하지만 완벽하지 않습니다.

- **오진 가능성**: 문제를 잘못 진단할 수 있습니다
- **부적절한 수정**: 상황을 악화시킬 수 있는 수정을 시도할 수 있습니다
- **예측 불가능성**: 100% 결정론적이지 않습니다

**완화 전략**:
- 처음에는 Watcher 모드로 시작하여 AI의 판단을 검토합니다
- 중요한 프로덕션 환경에서는 신중하게 사용합니다
- 백업 및 복구 계획을 항상 준비합니다
- 인간 온콜 엔지니어와 함께 사용합니다 (완전 대체가 아닌 보조)

### 2. 비용

Claude API 호출에는 비용이 발생합니다.

- **빈번한 실행**: 5분마다 실행하면 하루 288회
- **긴 컨텍스트**: 로그가 길면 토큰 사용량이 증가합니다
- **복잡한 분석**: 여러 번의 API 호출이 필요할 수 있습니다

**비용 최적화**:
- 실행 빈도를 조정합니다 (예: 15분 또는 30분)
- 로그 양을 제한합니다 (최근 100줄만)
- 건강한 Pod는 빠르게 건너뜁니다
- Haiku 모델 사용을 고려합니다 (간단한 작업의 경우)

### 3. 범위 제한

현재 버전은 Pod 모니터링에 초점을 맞추고 있습니다.

- **Node 문제**: 노드 레벨 문제는 감지하지 못합니다
- **네트워크 문제**: 복잡한 네트워크 문제는 진단하기 어렵습니다
- **외부 의존성**: 클러스터 외부의 서비스 문제는 보이지 않습니다

**향후 확장**:
- Node 상태 모니터링 추가
- Service와 Ingress 검사
- 외부 엔드포인트 헬스 체크

### 4. 동시성 문제

여러 Watcher 인스턴스가 동시에 실행되면 충돌할 수 있습니다.

- **경쟁 조건**: 두 Watcher가 동일한 문제를 동시에 수정하려고 시도
- **데이터베이스 잠금**: SQLite는 동시 쓰기에 제한적

**해결책**:
- CronJob의 `concurrencyPolicy`를 `Forbid`로 설정
- 분산 잠금 메커니즘 구현 (Redis, etcd)
- 각 네임스페이스에 대해 별도의 Watcher 인스턴스

### 5. 복잡한 문제

모든 문제가 핫픽스로 해결 가능한 것은 아닙니다.

- **구조적 문제**: 코드 재배포가 필요한 경우
- **리소스 부족**: 노드 용량 증가가 필요한 경우
- **설계 결함**: 아키텍처 변경이 필요한 경우

**현실적 기대치**:
- Watcher를 1차 대응자로 간주합니다
- 해결할 수 없는 문제는 인간에게 에스컬레이션합니다
- 완전 대체가 아닌 보조 도구로 사용합니다

### 6. 보안 위험

Autonomous 모드는 강력한 권한을 부여합니다.

- **오작동**: 잘못된 명령이 시스템을 손상시킬 수 있습니다
- **권한 남용**: exec 권한은 모든 작업을 허용합니다
- **데이터 유출**: 로그에 민감한 정보가 포함될 수 있습니다

**보안 강화**:
- RBAC를 엄격하게 설정합니다
- 중요한 네임스페이스는 제외합니다
- 감사 로깅을 활성화합니다
- 정기적으로 활동을 검토합니다

## 사용 모범 사례

### 1. 점진적 도입

**1단계: Watcher 모드로 시작**
```yaml
env:
  - name: WATCHER_MODE
    value: "watcher"
```

최소 2주 동안 AI의 판단을 관찰합니다. 다음을 확인하세요.
- 문제 감지 정확도
- 제안된 수정의 적절성
- 거짓 양성 비율

**2단계: 제한된 네임스페이스에서 Autonomous**

비중요 개발 또는 스테이징 환경에서 Autonomous 모드를 활성화합니다.

```yaml
env:
  - name: WATCHER_MODE
    value: "autonomous"
  - name: TARGET_NAMESPACE
    value: "dev"
```

**3단계: 프로덕션 확장**

신뢰가 구축되면 점진적으로 프로덕션 네임스페이스로 확장합니다. 항상 백업과 롤백 계획을 준비합니다.

### 2. 모니터링 및 알림

Watcher 자체도 모니터링해야 합니다.

**Prometheus 메트릭**
Dashboard를 확장하여 Prometheus 메트릭을 노출할 수 있습니다.
- `watcher_runs_total`: 총 실행 횟수
- `watcher_errors_detected`: 감지된 오류
- `watcher_fixes_applied`: 적용된 수정
- `watcher_fixes_failed`: 실패한 수정

**AlertManager 통합**
중요한 이벤트에 대한 알림을 설정합니다.
- Watcher 실행 실패
- 여러 번 수정 실패
- 예상치 못한 오류 패턴

**Slack/PagerDuty 통합**
Dashboard를 수정하여 웹훅을 통해 알림을 전송합니다.

```go
func notifySlack(message string) {
    // Slack 웹훅 호출
}

func notifyPagerDuty(incident Incident) {
    // PagerDuty API 호출
}
```

### 3. 정기적인 검토

**주간 리뷰**
- Dashboard에서 모든 실행을 검토합니다
- 패턴을 식별합니다 (반복되는 문제)
- AI의 판단 품질을 평가합니다

**월간 분석**
- 감지/수정 비율을 계산합니다
- 비용 대비 가치를 평가합니다
- 마스터 프롬프트 개선 기회를 식별합니다

**분기별 감사**
- 보안 검토를 수행합니다
- 권한이 여전히 적절한지 확인합니다
- 새로운 기능이나 개선 사항을 계획합니다

### 4. 문서화

**Runbook 유지**
Watcher가 일반적으로 처리하는 문제에 대한 Runbook을 작성합니다. AI가 실패할 경우 인간이 참조할 수 있습니다.

**인시던트 리포트**
각 중요한 수정에 대해 사후 분석을 작성합니다.
- 무엇이 잘못되었는가?
- Watcher가 어떻게 감지했는가?
- 어떤 조치를 취했는가?
- 결과는 무엇인가?
- 교훈은 무엇인가?

### 5. 백업 및 복구

**데이터베이스 백업**
SQLite 데이터베이스를 정기적으로 백업합니다.

```bash
# CronJob으로 설정
kubectl create cronjob db-backup \
  --schedule="0 2 * * *" \
  --image=alpine \
  -- sh -c "cp /data/watcher.db /backup/watcher-$(date +%Y%m%d).db"
```

**Disaster Recovery 계획**
- Watcher가 시스템을 손상시킨 경우 롤백 절차
- 데이터베이스 손상 시 복구 방법
- 전체 시스템 재배포 절차

## 향후 발전 방향

프로젝트는 여전히 초기 단계이며, 다음과 같은 잠재적 개선 사항이 있습니다.

### 1. 멀티 클러스터 지원

여러 Kubernetes 클러스터를 단일 Watcher 인스턴스로 모니터링합니다.

```yaml
env:
  - name: CLUSTERS
    value: "prod-us,prod-eu,prod-asia"
```

### 2. 머신러닝 기반 예측

과거 데이터를 분석하여 문제를 사전에 예측합니다.

- 리소스 사용 패턴
- 오류 발생 주기
- 계절적 트래픽 변화

### 3. 자동 스케일링 통합

HPA(Horizontal Pod Autoscaler) 및 VPA(Vertical Pod Autoscaler)와 통합하여 리소스 관련 문제를 자동으로 해결합니다.

### 4. GitOps 통합

문제가 코드 변경을 필요로 하는 경우, 자동으로 Pull Request를 생성합니다.

```
1. 문제 감지
2. 코드 수정 제안
3. PR 생성 (GitHub/GitLab)
4. 인간 검토 요청
5. 승인 후 자동 배포
```

### 5. 고급 분석

더 정교한 분석 기능을 추가합니다.

- 근본 원인 분석 (RCA) 자동화
- 연쇄 장애 감지
- 의존성 매핑
- 영향 범위 분석

### 6. 커스터마이징 가능한 플러그인

사용자가 자신만의 감지 및 수정 로직을 추가할 수 있도록 플러그인 시스템을 제공합니다.

```python
# custom_detector.py
class CustomDetector:
    def detect(self, pod):
        # 커스텀 감지 로직
        pass
    
    def fix(self, pod):
        # 커스텀 수정 로직
        pass
```

### 7. 다중 LLM 지원

Claude 외에 다른 LLM도 지원합니다.

- GPT-4
- Gemini
- 로컬 모델 (Llama 등)

사용자가 비용, 성능, 기능에 따라 선택할 수 있습니다.

## 커뮤니티 및 기여

### 프로젝트 정보

- **저장소**: https://github.com/kubeden/clopus-watcher
- **라이선스**: MIT
- **관리자**: Denislav Gavrilov (kubeden)
- **Stars**: 33 (2026년 1월 기준)
- **Forks**: 7

### 기여 방법

프로젝트는 오픈 소스이며 기여를 환영합니다.

**버그 리포트**
GitHub Issues에 버그를 보고할 때 다음을 포함하세요.
- Watcher 버전
- Kubernetes 버전
- 재현 단계
- 예상 동작 vs 실제 동작
- 관련 로그

**기능 제안**
새로운 기능을 제안할 때:
- 사용 사례를 설명합니다
- 현재 해결 방법이 있는지 설명합니다
- 가능하다면 구현 아이디어를 제공합니다

**Pull Request**
코드 기여 시:
- 명확한 설명과 함께 PR을 제출합니다
- 기존 코드 스타일을 따릅니다
- 테스트를 추가합니다 (해당하는 경우)
- 문서를 업데이트합니다

### 주목할 만한 지지자

- **Jerry Liu** (LlamaIndex 공동 창립자)가 프로젝트에 Star를 표시했습니다. 이는 AI 및 개발자 도구 커뮤니티에서의 관심을 나타냅니다.

## 결론 및 영향

Clopus-Watcher는 단순한 기술 데모를 넘어서, AI가 운영 업무를 어떻게 변화시킬 수 있는지에 대한 중요한 질문을 제기합니다.

### 기술적 의미

이 프로젝트는 다음을 증명합니다.

1. **LLM의 운영 능력**: Claude와 같은 대규모 언어 모델은 코드 생성을 넘어 실제 시스템 운영 작업을 수행할 수 있습니다.

2. **자율 에이전트의 실용성**: 완전 자율적인 AI 에이전트가 더 이상 SF가 아니라 현재 기술로 구현 가능합니다.

3. **Kubernetes의 확장성**: Kubernetes의 네이티브 기능(CronJob, RBAC, PVC)만으로도 정교한 AI 시스템을 구축할 수 있습니다.

### 산업적 영향

Denislav의 대담한 주장, "온콜 엔지니어가 개발자보다 먼저 자동화될 수 있다"는 심각하게 고려해볼 가치가 있습니다.

**온콜 업무가 자동화하기 쉬운 이유**:
- 더 체계적이고 문서화되어 있습니다
- 명확한 절차와 플레이북을 따릅니다
- 성공/실패가 명확하게 측정 가능합니다
- 창의성보다는 신속성과 정확성이 중요합니다

**개발이 더 어려운 이유**:
- 요구사항이 모호하고 변화합니다
- 창의적 문제 해결이 필요합니다
- 장기적 아키텍처 결정이 필요합니다
- 비즈니스 맥락 이해가 중요합니다

그러나 완전 대체보다는 **인간-AI 협업**이 더 현실적인 미래입니다.
- AI는 일상적이고 반복적인 작업을 처리합니다
- 인간은 복잡하고 전략적인 결정에 집중합니다
- 함께 일하면서 서로의 강점을 활용합니다

### 사회적 영향

**긍정적 측면**:
- 온콜 엔지니어의 삶의 질 향상 (야간 호출 감소)
- 더 빠른 문제 해결 (즉각적인 대응)
- 비용 절감 (인건비 vs AI 비용)
- 24/7 커버리지 (휴일, 주말 포함)

**우려 사항**:
- 일자리 대체 (특히 주니어 레벨)
- 기술 의존도 증가
- AI 오작동 시 위험
- 숙련도 감소 (인간이 연습할 기회 감소)

### 윤리적 고려사항

AI를 운영에 사용할 때 다음을 고려해야 합니다.

**투명성**: AI의 모든 결정과 조치는 기록되고 검토 가능해야 합니다.

**책임성**: AI가 실수를 하면 누가 책임을 지는가? 명확한 책임 체계가 필요합니다.

**공정성**: AI 기반 자동화는 모든 사람에게 공정해야 하며, 특정 그룹을 불리하게 해서는 안 됩니다.

**안전성**: 최우선 순위는 항상 시스템과 데이터의 안전이어야 합니다.

### 마지막 생각

Clopus-Watcher는 야심찬 실험에서 실용적인 도구로 진화하고 있습니다. 현재 상태에서 프로덕션에 완전히 의존하기에는 이르지만, 그 방향성과 잠재력은 부인할 수 없습니다.

이 프로젝트는 우리에게 다음을 상기시킵니다.

1. **AI의 능력은 빠르게 확장되고 있습니다**: 몇 년 전만 해도 불가능했던 일이 이제는 몇 줄의 코드로 가능합니다.

2. **자동화의 경계는 계속 이동합니다**: "자동화할 수 없다"고 생각했던 많은 것들이 실제로는 가능합니다.

3. **인간의 역할은 진화합니다**: 대체되는 것이 아니라, 더 높은 수준의 작업으로 이동합니다.

4. **실험과 혁신의 중요성**: Denislav처럼 대담하게 실험하는 사람들이 미래를 만듭니다.

Clopus-Watcher가 온콜 엔지니어링의 미래를 어떻게 형성할지는 시간이 말해줄 것입니다. 하지만 한 가지는 분명합니다. AI와 자동화는 더 이상 선택이 아니라 필수이며, 이를 현명하게 활용하는 조직이 경쟁에서 앞서갈 것입니다.

---

## 참고 자료

### 공식 문서 및 링크

- **GitHub 저장소**: https://github.com/kubeden/clopus-watcher
- **라이브 데모 대시보드**: https://k8s-watcher.clopus.live
- **Clopus 시리즈 블로그**:
  - Clopus-01 (반자율 Claude): https://denislavgavrilov.com/p/clopus-01-a-semi-autonomous-claude
  - Clopus-02 (24시간 실행): https://denislavgavrilov.com/p/clopus-02-a-24-hour-claude-code-run
  - Clopus-Watcher (모니터링 에이전트): https://denislavgavrilov.com/p/clopus-watcher-an-autonomous-monitoring

### 관련 기술

- **Claude Code**: Anthropic의 AI 기반 코딩 어시스턴트
- **Kubernetes**: 컨테이너 오케스트레이션 플랫폼
- **CronJob**: Kubernetes의 스케줄링 리소스
- **SQLite**: 경량 임베디드 데이터베이스
- **Go**: 시스템 프로그래밍 언어

### 추가 읽기

- Anthropic Claude Documentation
- Kubernetes CronJob Documentation
- RBAC in Kubernetes
- AI Agents and Autonomous Systems
- DevOps Automation Best Practices

---

**작성 일자**: 2026-01-31
