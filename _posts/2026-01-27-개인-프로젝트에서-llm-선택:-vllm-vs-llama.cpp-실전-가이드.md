---
title: "개인 프로젝트에서 LLM 선택: vLLM vs llama.cpp 실전 가이드"
date: 2026-01-27 07:10:00 +0900
categories: [AI,  MLOps]
mermaid: [True]
tags: [AI,  ai-governance,  MLOps,  LLM,  lama.cpp,  vLLM,  Claude.write]
---


## 관련글

[**ML 서빙 패러다임의 극적인 전환: 전통적 프레임워크에서 LLM 특화 엔진으로**](https://k82022603.github.io/posts/ml-%EC%84%9C%EB%B9%99-%ED%8C%A8%EB%9F%AC%EB%8B%A4%EC%9E%84%EC%9D%98-%EA%B7%B9%EC%A0%81%EC%9D%B8-%EC%A0%84%ED%99%98-%EC%A0%84%ED%86%B5%EC%A0%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EC%97%90%EC%84%9C-llm-%ED%8A%B9%ED%99%94-%EC%97%94%EC%A7%84%EC%9C%BC%EB%A1%9C/)

## 관련글

>
>진짜 여러분, **개인 프로젝트에서 LLM 모델 로딩할 때, vLLM 절대 쓰지마세요.**
>
>GGUF 포맷을 지원하는 프레임워크를 사용하세요.
>
>GPU Offloading 이 가능한 프레임워크를 사용하세요.
>
>llama.cpp 를 사용하세요.
>
>vLLM으로 날새면서 vRAM 나눠먹느라 이틀을 밤을 샜는데 너무 느린데다 토큰량이 험악해서 쓰지도 못하더라고요. 
>
>vLLM 은 실제 서비스를 위한 추론 프레임워크입니다. 애초에 목적이 달라서 vLLM은 적합하지 않습니다. 
>
>GPU Offloading 기술은 부족한 GPU 메모리의 한계를 극복하고자 나왔는데요, 모델의 일부 레이어를 GPU가 아닌 CPU로 내려줍니다. 
>
>6GB의 낮은 vRAM을 가진 GPU로도 7B 파라미터를 가진 모델을 32K나 16K 토큰 사이즈로 올릴 수가 있습니다. 
>
>메모리가 더 있으시다면 더 늘릴 수도 있습니다.
>
>다들 어떻게 알고 하고 계셨던건지. 
>
>역시 배움엔 끝이 없군요. 
>
>눈물이 납니다. ㅠㅠ
>
>https://www.threads.com/@edjn_coder/post/DT7n1jqkr1i
>
>**Gguf로 올렸다가 너무 느려서 결국 vllm으로 올리고 사용하고 있습니다..**
>

## 서론: 두 가지 경험담이 모두 옳은 이유

최근 커뮤니티에서 흥미로운 두 가지 상반된 경험담이 공유되었습니다:

**경험 A**: "개인 프로젝트에서 vLLM 절대 쓰지마세요. 이틀 밤샜는데 너무 느리고 토큰량이 험악했어요. llama.cpp 쓰세요!"

**경험 B**: "GGUF로 올렸다가 너무 느려서 결국 vLLM으로 올리고 사용하고 있습니다."

두 경험이 완전히 반대인데, 둘 다 틀린 게 아닙니다. 이것이 바로 LLM 서빙에서 "상황에 맞는 도구 선택"이 얼마나 중요한지 보여주는 완벽한 사례입니다. 

이 문서에서는 각 경험이 발생한 맥락을 분석하고, 여러분의 상황에서 올바른 선택을 하는 방법을 제시합니다.

---

## 경험 A 분석: 왜 개인 프로젝트에서 vLLM이 과도했는가?

### 상황 재구성

원글 작성자의 상황을 보면:
- **목적**: 개인 프로젝트 (프로토타입/개발 환경)
- **문제**: vRAM 부족으로 GPU 메모리를 나눠먹느라 고생
- **결과**: vLLM이 너무 느리고 토큰량 제한 심각
- **해결**: llama.cpp로 전환 후 6GB vRAM으로 7B 모델을 32K 토큰으로 실행 성공

### 왜 vLLM이 실패했는가?

**1. vLLM의 메모리 요구사항**

vLLM은 고성능을 위해 설계되었고, 이는 메모리 오버헤드를 수반합니다:

```
vLLM 메모리 구성:
- 모델 가중치 (전체를 GPU에 로드)
- KV cache (PagedAttention용 페이지 테이블)
- 활성화 메모리 (배치 처리용)
- CUDA 런타임 오버헤드
= 실제 모델 크기의 1.5~2배 이상
```

예를 들어, Llama 3.1 8B 모델 (FP16):
- 순수 모델 크기: ~16GB
- vLLM에서 필요한 메모리: 20-24GB
- 6GB vRAM으로는 불가능

**2. 양자화 지원의 차이**

vLLM은 최근까지 GGUF를 공식적으로 지원하지 않았습니다:
- AWQ, GPTQ 같은 양자화는 지원
- 하지만 이들도 메모리 절감이 GGUF만큼 극적이지 않음
- GGUF의 4비트 양자화만큼 작게 만들기 어려움

원글 작성자가 "vRAM을 나눠먹느라" 고생한 것은 아마도:
- FP16이나 Int8 모델을 사용했거나
- 텐서 병렬화를 시도했거나
- CPU 오프로딩을 시도했지만 vLLM의 제한적 지원으로 실패

**3. 단일 사용자 환경에서의 오버킬**

vLLM의 강점인 기능들이 개인 프로젝트에서는 오히려 부담:
- **Continuous batching**: 혼자 쓰는데 배치가 필요 없음
- **PagedAttention**: 메모리 효율성보다 오버헤드가 더 큼
- **멀티 GPU 최적화**: 1개 GPU만 있으면 불필요
- **복잡한 스케줄링**: CPU 사이클만 낭비

### llama.cpp로 문제가 해결된 이유

**1. GPU Offloading의 마법**

llama.cpp의 핵심 장점:

```bash
# 6GB GPU에서 7B 모델 32K 컨텍스트 실행
./llama-server \
  -m model-Q4_K_M.gguf \
  --n-gpu-layers 20 \    # 일부 레이어만 GPU에
  --ctx-size 32768 \     # 32K 컨텍스트
  --threads 8
```

**작동 방식:**
1. 95개 레이어 중 20개만 GPU에 올림 (~2-3GB 사용)
2. 나머지 75개 레이어는 CPU RAM에서 실행
3. KV cache는 필요에 따라 GPU/CPU 분산
4. 총 GPU 메모리 사용: ~4-5GB (6GB 안에 여유있게 수용)

**2. GGUF 포맷의 압축 효율**

```
동일 모델의 크기 비교 (Llama 3.1 8B):
- FP16 (원본): 16GB
- Int8: 8GB  
- Q8_0 (GGUF): 8GB
- Q6_K (GGUF): 6GB
- Q4_K_M (GGUF): 4.5GB
- Q4_0 (GGUF): 4GB
- Q2_K (GGUF): 3GB
```

Q4_K_M으로 양자화하면:
- 모델 크기가 원본의 1/4로 감소
- 품질 손실은 미미 (대부분 태스크에서 구분 불가)
- 6GB GPU에 모델 + KV cache + 오버헤드 모두 수용 가능

**3. 단순한 아키텍처**

llama.cpp는 단일 사용자를 위해 설계됨:
- 최소한의 오버헤드
- 직관적인 메모리 관리
- 빠른 시작 시간 (수 초)
- 복잡한 의존성 없음

### 원글 작성자의 조언이 타당한 경우

다음 상황에서는 llama.cpp가 절대적으로 우수합니다:

1. **제한된 GPU 메모리** (8GB 이하)
2. **개인 개발/프로토타입**
3. **단일 사용자**
4. **CPU 리소스가 충분** (8+ 코어)
5. **빠른 실험/반복이 중요**

---

## 경험 B 분석: 왜 누군가는 vLLM으로 돌아갔는가?

### "GGUF로 올렸다가 너무 느려서..."

이 댓글은 완전히 다른 상황을 시사합니다. 어떤 경우에 GGUF/llama.cpp가 "너무 느릴" 수 있을까요?

### 1. 높은 처리량이 필요한 경우

**시나리오**: 데모 서비스나 소규모 베타 테스트

```
상황:
- 10-50명의 동시 사용자
- 실시간 응답 필요
- 각 사용자가 긴 대화 세션

llama.cpp 결과:
- 순차 처리로 인한 큐 대기
- 10번째 사용자는 30초+ 대기
- 사용자 경험 악화

vLLM 결과:
- Continuous batching으로 모든 요청 병렬 처리
- 평균 응답 시간 5초 이내
- 부드러운 사용자 경험
```

실제 벤치마크 (이전 문서 참조):
- llama.cpp: 동시성 증가해도 처리량 평평 (~10 TPS)
- vLLM: 동시성 256에서 793 TPS

**10명의 동시 사용자만 있어도** vLLM이 체감 속도에서 압도적입니다.

### 2. 충분한 GPU 메모리가 있는 경우

**하드웨어**: RTX 4090 (24GB) 또는 A100 (40GB/80GB)

```python
# GGUF/llama.cpp로 실행
# 메모리: 8GB 사용 (16GB 여유)
# 속도: 단일 요청 15 t/s

# vLLM으로 실행  
# 메모리: 20GB 사용 (4GB 여유)
# 속도: 
#   - 단일 요청: 18 t/s (20% 빠름)
#   - 5개 동시 요청: 80 t/s total (4배 빠름)
#   - 10개 동시 요청: 150 t/s total (10배 빠름)
```

메모리가 충분하면 vLLM의 최적화가 빛을 발합니다:
- Flash Attention 구현
- CUDA 커널 최적화
- 효율적인 메모리 레이아웃

### 3. 모델 크기에 따른 Trade-off

**작은 모델 (7B)**:
- GGUF Q4: ~4GB
- vLLM FP16: ~16GB
- GPU가 24GB라면? vLLM 선택이 합리적

**큰 모델 (70B)**:
- GGUF Q4: ~40GB  
- vLLM FP16: ~140GB
- 단일 GPU로 불가능
- 이때는 GGUF + GPU offloading이 유일한 선택

### 4. 개발 편의성

vLLM이 더 편한 경우:
- **OpenAI 호환 API**: 기존 코드 그대로 사용
- **자동 스케일링**: Ray와 통합하면 Kubernetes에 쉽게 배포
- **모니터링**: 내장된 메트릭 엔드포인트
- **문서화**: 대규모 커뮤니티와 풍부한 예제

llama.cpp는:
- 명령줄 플래그가 복잡 (특히 멀티 GPU)
- Python 바인딩이 있지만 공식 지원은 아님
- 프로덕션 배포 시 직접 래핑 필요

### 5. 성능 프로파일의 차이

**Ahmad Osman의 경험** (이전 검색 결과):
```
하드웨어: 14x GPU 서버
llama.cpp (CPU offloading):
- DeepSeek v2.5 236B BF16
- 1 token/second
- "okay but it really is not"

vLLM (Tensor Parallelism):
- Llama 3.1 70B BF16
- 8x GPU 사용
- 800 tokens/second
- 50개 비동기 요청 동시 처리
```

800배 차이입니다. 하드웨어가 있다면 vLLM이 압도적입니다.

---

## 의사 결정 프레임워크: 나는 무엇을 선택해야 하는가?

### Step 1: 하드웨어 평가

```
GPU VRAM이 8GB 이하인가?
└─ YES → llama.cpp (필수)
└─ NO → Step 2로

GPU가 여러 개인가?
└─ YES → Step 3으로
└─ NO → Step 2로
```

### Step 2: 사용 패턴 평가

```
동시 사용자가 5명 이상인가?
├─ YES → vLLM 강력 추천
└─ NO → 혼자만 사용하는가?
    ├─ YES → llama.cpp 추천
    └─ NO (2-4명) → Step 3으로
```

### Step 3: 성능 요구사항 평가

```
응답 시간이 중요한가?
├─ 매우 중요 (실시간 챗봇) → vLLM
├─ 중요 (데모/프로토타입) → vLLM
└─ 덜 중요 (배치/연구) → llama.cpp

처리량이 중요한가?
├─ 매우 중요 (API 서비스) → vLLM
├─ 중요 (내부 도구) → vLLM
└─ 덜 중요 (개인 사용) → llama.cpp
```

### Step 4: 개발 단계 고려

```
현재 단계:
├─ 탐색/학습 → llama.cpp (빠른 실험)
├─ 프로토타입 → llama.cpp (간편함)
├─ 알파/베타 → vLLM (확장성)
└─ 프로덕션 → vLLM (안정성)
```

---

## 실전 가이드: 상황별 최적 설정

### 사례 1: GPU 6GB, 개인 프로젝트

**문제**: "7B 모델을 긴 컨텍스트로 사용하고 싶어요"

**해결책**: llama.cpp + GGUF + GPU Offloading

```bash
# 1. GGUF 모델 다운로드
wget https://huggingface.co/TheBloke/Llama-3.1-8B-GGUF/\
  resolve/main/llama-3.1-8b.Q4_K_M.gguf

# 2. 최적 레이어 수 찾기
# 시작: 모든 레이어를 GPU로
./llama-server -m llama-3.1-8b.Q4_K_M.gguf -ngl 999

# GPU OOM 발생 시, 레이어 수 줄이기
./llama-server -m llama-3.1-8b.Q4_K_M.gguf -ngl 20

# 3. 컨텍스트 크기 조정
./llama-server \
  -m llama-3.1-8b.Q4_K_M.gguf \
  -ngl 20 \                    # GPU에 20개 레이어
  -c 32768 \                   # 32K 컨텍스트
  --host 0.0.0.0 \
  --port 8080
```

**기대 성능**:
- 메모리 사용: ~5GB GPU + 8GB RAM
- 속도: 10-15 t/s (단일 사용자)
- 컨텍스트: 32K 토큰 가능

### 사례 2: GPU 24GB (RTX 4090), 소규모 데모

**문제**: "친구들에게 챗봇 데모 보여주고 싶어요 (10-20명)"

**해결책**: vLLM + FP16 또는 AWQ

```bash
# 방법 1: FP16 (최고 품질)
vllm serve meta-llama/Llama-3.1-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --max-model-len 8192

# 방법 2: AWQ (더 많은 메모리 여유)
vllm serve TheBloke/Llama-3.1-8B-AWQ \
  --gpu-memory-utilization 0.9 \
  --max-model-len 16384

# 실제 사용
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.1-8B-Instruct",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

**기대 성능**:
- 메모리 사용: ~18GB GPU
- 속도: 
  - 1명: 20 t/s
  - 10명 동시: 총 150 t/s (각 15 t/s)
  - 20명 동시: 총 250 t/s (각 12.5 t/s)
- 대기 시간: 평균 1초 이내

### 사례 3: GPU 2x4090 (48GB total), 실제 서비스

**문제**: "70B 모델로 API 서비스를 만들고 싶어요"

**해결책**: vLLM + Tensor Parallelism

```bash
# 텐서 병렬화로 모델을 2개 GPU에 분산
vllm serve meta-llama/Llama-3.1-70B-Instruct \
  --tensor-parallel-size 2 \
  --gpu-memory-utilization 0.95 \
  --max-model-len 4096

# Kubernetes 배포 (선택)
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-llama70b
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: vllm
        image: vllm/vllm-openai:latest
        resources:
          limits:
            nvidia.com/gpu: 2
        args:
        - serve
        - meta-llama/Llama-3.1-70B-Instruct
        - --tensor-parallel-size=2
```

**기대 성능**:
- 메모리 사용: 각 GPU ~40GB (총 80GB 중 사용)
- 속도:
  - 1명: 15 t/s
  - 50명 동시: 총 500+ t/s
  - 100명 동시: 총 800+ t/s

### 사례 4: GPU 8GB, 70B 모델 실험

**문제**: "작은 GPU로 큰 모델을 돌려보고 싶어요"

**해결책**: llama.cpp + 극한 양자화 + CPU Offloading

```bash
# 1. Q2_K 양자화 다운로드 (가장 작은 크기)
wget https://huggingface.co/TheBloke/Llama-3.1-70B-GGUF/\
  resolve/main/llama-3.1-70b.Q2_K.gguf

# 2. 신중한 레이어 분배
./llama-server \
  -m llama-3.1-70b.Q2_K.gguf \
  -ngl 10 \                    # GPU에 10개만
  -c 2048 \                    # 작은 컨텍스트
  -t 16 \                      # 16 CPU 스레드 사용
  --mlock                      # RAM을 lock하여 스왑 방지

# 3. 성능 모니터링
htop  # CPU 사용률
nvidia-smi -l 1  # GPU 사용률
```

**trade-off 이해하기**:
- ✅ 가능: 8GB GPU로 70B 모델 실행
- ❌ 느림: 2-3 t/s (CPU 병목)
- ❌ 품질 손실: Q2_K는 눈에 띄는 품질 저하
- 🤔 용도: 실험/탐색용, 프로덕션 X

**더 나은 대안**:
```bash
# Q4_K_M 사용 (품질 ↑, 크기 ↑)
# GPU 레이어 0으로 설정 (모두 CPU)
./llama-server \
  -m llama-3.1-70b.Q4_K_M.gguf \
  -ngl 0 \                     # 모든 레이어 CPU
  -c 2048 \
  -t 32                        # CPU 코어 모두 사용

# 64GB+ RAM 필요
```

---

## 하이브리드 전략: 양쪽의 장점 취하기

### 전략 1: 개발은 llama.cpp, 배포는 vLLM

```
워크플로우:
1. [개발 단계] llama.cpp로 빠른 실험
   - Ollama로 여러 모델 테스트
   - 프롬프트 엔지니어링
   - 기능 검증
   
2. [프로토타입] llama.cpp로 초기 사용자 테스트
   - 간단한 배포
   - 빠른 반복
   
3. [스케일링] vLLM으로 전환
   - OpenAI API 덕분에 코드 변경 최소
   - 동시 사용자 증가에 대응
   
4. [프로덕션] vLLM + Kubernetes
   - 자동 스케일링
   - 모니터링
   - 안정성
```

**코드 예시** (플랫폼 독립적):

```python
# config.py
import os

# 환경 변수로 백엔드 선택
BACKEND = os.getenv("LLM_BACKEND", "llama.cpp")

if BACKEND == "llama.cpp":
    BASE_URL = "http://localhost:8080/v1"
elif BACKEND == "vllm":
    BASE_URL = "http://localhost:8000/v1"
else:
    BASE_URL = "https://api.openai.com/v1"

# 클라이언트 코드는 동일
from openai import OpenAI

client = OpenAI(base_url=BASE_URL, api_key="dummy")

response = client.chat.completions.create(
    model="llama-3.1-8b",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(response.choices[0].message.content)
```

**배포 시나리오**:
```bash
# 개발자 로컬
export LLM_BACKEND=llama.cpp
python app.py

# 스테이징 서버 (작은 트래픽)
export LLM_BACKEND=llama.cpp
docker-compose up

# 프로덕션 (높은 트래픽)
export LLM_BACKEND=vllm
kubectl apply -f deployment.yaml
```

### 전략 2: 모델 크기별 분리

```
아키텍처:
┌─────────────────────────────────────┐
│ API Gateway (LiteLLM or OpenRouter) │
└────────────┬────────────────────────┘
             │
      ┌──────┴──────┐
      │             │
┌─────▼─────┐ ┌────▼────────┐
│ Small     │ │ Large       │
│ Models    │ │ Models      │
│ (7B-13B)  │ │ (70B-405B)  │
│           │ │             │
│ vLLM      │ │ llama.cpp   │
│ RTX 4090  │ │ CPU cluster │
└───────────┘ └─────────────┘
```

**라우팅 로직**:
```python
def route_request(prompt, required_quality):
    if required_quality == "best":
        # 큰 모델 (느리지만 정확)
        return query_llamacpp_70b(prompt)
    elif len(prompt) > 4000:
        # 긴 컨텍스트 (llama.cpp가 더 효율적)
        return query_llamacpp_8b(prompt)
    else:
        # 일반 쿼리 (빠른 응답)
        return query_vllm_8b(prompt)
```

### 전략 3: 시간대별 백엔드 전환

```
스케줄:
00:00-08:00 (오프 피크)
  → llama.cpp (CPU only)
  → 전력 절약, GPU 유휴

08:00-20:00 (피크)
  → vLLM (GPU accelerated)
  → 빠른 응답

20:00-24:00 (중간)
  → llama.cpp (GPU offloading)
  → GPU 일부만 사용
```

---

## 흔한 실수와 해결책

### 실수 1: "vLLM이 더 좋다고 해서 무조건 vLLM"

**증상**:
```
Loading model...
OutOfMemoryError: CUDA out of memory. 
Tried to allocate 20.00 GiB but only 6GB available.
```

**원인**: 하드웨어 무시

**해결**:
1. GPU 메모리 확인: `nvidia-smi`
2. 모델 크기 계산: 파라미터 × bytes per parameter
3. 여유 공간 확인: 최소 20% 오버헤드 필요

### 실수 2: "llama.cpp가 느리다고 vLLM으로 전환"

**증상**:
```python
# llama.cpp에서
response_time = 8 seconds  # 혼자 사용

# vLLM으로 전환 후
response_time = 15 seconds  # 혼자 사용
```

**원인**: 단일 사용자 환경에서 vLLM의 오버헤드

**해결**: 실제 사용 패턴 측정
```python
import time

# 동시 요청 시뮬레이션
concurrent_users = 1

start = time.time()
# ... 요청 ...
end = time.time()

print(f"{concurrent_users} users: {end-start}s")

# 여러 동시성 레벨로 반복
```

### 실수 3: "CPU offloading 설정 잘못"

**나쁜 예**:
```bash
# GPU 레이어를 너무 많이
./llama-server -m model.gguf -ngl 999
# → GPU OOM

# GPU 레이어를 너무 적게  
./llama-server -m model.gguf -ngl 1
# → 거의 CPU만 사용, 매우 느림
```

**좋은 예**:
```bash
# 1. GPU 메모리 확인
nvidia-smi
# Available: 6144 MiB

# 2. 이진 탐색으로 최적점 찾기
# 시도 1: 중간값
./llama-server -m model.gguf -ngl 30
# → OOM

# 시도 2: 줄이기
./llama-server -m model.gguf -ngl 15
# → 성공, 4GB 사용

# 시도 3: 늘리기
./llama-server -m model.gguf -ngl 20
# → 성공, 5.5GB 사용 (최적!)

# 3. 로그 확인
# llm_load_tensors: ggml ctx size = 0.15 MiB
# llm_load_tensors: offloading 20 repeating layers to GPU
# llm_load_tensors: offloaded 20/33 layers to GPU
```

### 실수 4: "양자화 레벨 잘못 선택"

**품질 vs 크기**:
```
Llama 3.1 8B 모델:
Q2_K: 3GB   - 심각한 품질 저하, 비추천
Q4_0: 4GB   - 적당한 품질, 빠름
Q4_K_M: 5GB - 좋은 품질, 추천 ⭐
Q5_K_M: 6GB - 매우 좋은 품질
Q6_K: 6.5GB - 원본에 가까움
Q8_0: 8GB   - 거의 손실 없음
FP16: 16GB  - 원본
```

**선택 가이드**:
- 일반 채팅: Q4_K_M
- 코딩 도움: Q5_K_M or Q6_K
- 창작 글쓰기: Q6_K or Q8_0
- 연구/벤치마크: FP16

### 실수 5: "메모리 단편화 무시"

**문제**:
```
1st request: 4GB used, 2GB free  
2nd request: 5GB used, 1GB free
3rd request: OOM!
# 여유 1GB인데 왜 OOM?
# → 단편화로 연속된 1GB 블록 없음
```

**llama.cpp 해결**:
```bash
--mlock  # RAM을 lock, 스왑 방지
--no-mmap  # mmap 대신 직접 로드
```

**vLLM 해결**:
- PagedAttention이 자동으로 처리
- 별도 조치 불필요

---

## 성능 벤치마크: 직접 측정하기

### llama.cpp 벤치마크

```bash
# llama-bench 사용
./llama-bench \
  -m model.gguf \
  -ngl 20 \
  -p 512,1024,2048 \   # 프롬프트 길이
  -n 128,256 \         # 생성 토큰 수
  -r 3                 # 3번 반복

# 출력 예시:
| model | size | params | backend | ngl | test      | t/s    |
|-------|------|--------|---------|-----|-----------|--------|
| 7B    | 4GB  | 7B     | CUDA    | 20  | pp512     | 250.5  |
| 7B    | 4GB  | 7B     | CUDA    | 20  | tg128     | 15.2   |
```

### vLLM 벤치마크

```python
# benchmark_vllm.py
from vllm import LLM, SamplingParams
import time

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")
prompts = ["Hello!"] * 10  # 10개 동시 요청

start = time.time()
outputs = llm.generate(prompts, SamplingParams(max_tokens=128))
end = time.time()

total_tokens = sum(len(o.outputs[0].token_ids) for o in outputs)
throughput = total_tokens / (end - start)

print(f"Throughput: {throughput:.2f} tokens/s")
print(f"Latency: {(end-start)/len(prompts):.2f}s per request")
```

### 비교 스크립트

```bash
#!/bin/bash
# compare.sh - llama.cpp vs vLLM 성능 비교

echo "=== llama.cpp Benchmark ==="
./llama-bench -m model.gguf -ngl 20 -p 512 -n 128

echo ""
echo "=== vLLM Benchmark ==="
python benchmark_vllm.py

# 예상 결과:
# llama.cpp:
#   - Single request: 15 t/s
#   - 10 concurrent: ~15 t/s total (no batching)
#
# vLLM:
#   - Single request: 18 t/s
#   - 10 concurrent: 150 t/s total (10x!)
```

---

## 결론: 원글의 조언과 댓글 모두 옳다

### 원글이 옳은 경우

**"개인 프로젝트에서 vLLM 절대 쓰지마세요"**

✅ 맞습니다, 만약:
- GPU 메모리가 8GB 이하
- 혼자만 사용
- 빠른 실험이 목표
- CPU 리소스가 충분

이런 상황에서는 llama.cpp + GGUF가 **명백하게** 우수합니다.

### 댓글이 옳은 경우

**"GGUF로 올렸다가 너무 느려서 vLLM으로 전환"**

✅ 맞습니다, 만약:
- GPU 메모리가 24GB+
- 여러 사용자 (5명+)
- 실시간 응답 필요
- 처리량이 중요

이런 상황에서는 vLLM이 **명백하게** 우수합니다.

### 핵심 교훈

**"상황에 맞는 도구를 선택하라"**

1. **하드웨어 제약을 먼저 확인**하라
2. **실제 사용 패턴을 측정**하라
3. **미리 오버 엔지니어링하지 말고** 필요할 때 업그레이드하라
4. **하이브리드 전략**을 고려하라

### 실천 가이드

**지금 당장 해야 할 것:**

1. GPU 메모리 확인
```bash
nvidia-smi --query-gpu=memory.total --format=csv
```

2. 사용 패턴 추정
```
혼자 사용? → llama.cpp
팀 (5명+)? → vLLM 고려
서비스 (50명+)? → vLLM 필수
```

3. 시작점 선택
```
메모리 < 12GB → llama.cpp
메모리 ≥ 12GB → 둘 다 시도, 벤치마크
```

4. 반복과 개선
```
llama.cpp로 시작
→ 성능 측정
→ 병목 파악
→ 필요시 vLLM 전환
```

### 마지막 조언

**원글 작성자의 경험에서 배울 점**:
- ✅ 실제 경험을 공유해 다른 사람의 시간을 절약
- ✅ GPU Offloading 같은 핵심 기능을 강조
- ✅ 실용적인 수치 제공 (6GB로 32K 가능)

**댓글 작성자의 경험에서 배울 점**:
- ✅ 하나의 해답이 모든 상황에 맞지 않음
- ✅ 충분한 리소스가 있으면 최적화된 도구 사용
- ✅ 실제 테스트가 이론보다 중요

**양쪽 모두**: 자신의 상황을 이해하고 측정하고 선택했습니다. 이것이 올바른 접근입니다.

---

**이 문서를 읽는 당신에게**:

커뮤니티의 조언을 맹목적으로 따르지 마세요. 당신의 하드웨어를 확인하고, 사용 패턴을 측정하고, 실제로 벤치마크하세요. 

원글 작성자처럼 고생할 수도 있고, 댓글 작성자처럼 다른 길을 가야 할 수도 있습니다. 둘 다 괜찮습니다. 중요한 것은 **당신의 상황**입니다.

행운을 빕니다! 🚀

---

**작성 일자: 2026-01-27**
