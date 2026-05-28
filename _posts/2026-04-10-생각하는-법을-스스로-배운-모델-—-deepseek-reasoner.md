---
title: "생각하는 법을 스스로 배운 모델 — DeepSeek Reasoner"
date: 2026-04-10 23:00:00 +0900
categories: [AI,  RummiArena]
mermaid: [True]
tags: [AI,  RummiArena,  DeepSeek,  deepSeek-reasoner,  Claude.write]
---


- **작성일**: 2026-04-10
- **목적**: RummiArena AI 대전에서 DeepSeek Reasoner가 보여준 행동의 기술적 배경 이해

---

## 들어가며

우리는 DeepSeek를 오해하고 있었다.

Round 4 대전에서 DeepSeek Reasoner의 fallback 비율은 69%였다. 39턴 중 27턴이 실패로 끝났다. 우리는 그 숫자를 보고 "DeepSeek는 느리고 불안정한 모델"이라고 판단했다. 평균 응답 시간 131초를 보고 "그래도 어느 정도 쓸 만하다"고 위안을 삼았다. 하지만 그 131초라는 숫자가 어떻게 만들어진 것인지는 물어보지 않았다.

진실은 이랬다. 200초 타임아웃이 DeepSeek의 사고를 중간에 잘라버리고 있었다. 후반부에서 12,000~15,000개의 추론 토큰을 생성하며 치밀하게 보드를 분석하던 모델의 사고를 우리가 끊어버린 것이다. 타임아웃을 240초로 올리자 fallback은 0건이 되었다. DeepSeek가 변한 게 아니었다. 우리가 충분한 시간을 준 것뿐이었다.

이 글은 그 반성에서 출발한다. DeepSeek Reasoner가 어떤 모델인지, 왜 그렇게 오래 생각하는지, 그 생각이 어떤 가치를 가지는지를 제대로 이해하기 위해 쓴다.

---

## 1. 아키텍처 — 거인의 몸에 정밀한 뇌

DeepSeek-R1은 DeepSeek-V3를 기반으로 한다. 671억(671B) 파라미터를 가진 거대한 모델이지만, 요청 하나를 처리할 때 실제로 활성화되는 파라미터는 37억(37B)에 불과하다. 이것이 **Mixture-of-Experts(MoE)** 아키텍처의 핵심이다.

MoE는 수백 개의 전문가(expert) 네트워크를 보유하고, 입력 토큰의 특성에 따라 가장 적합한 전문가 몇 개만 선택적으로 활성화한다. 수학 문제가 들어오면 수학에 강한 전문가가, 코드가 들어오면 코딩에 특화된 전문가가 작동한다. 671B의 지식 용량을 가지면서도 37B 수준의 연산 비용만 소모하는 — 대형 도서관의 장서를 가지되, 필요한 책만 꺼내 읽는 구조다.

여기에 **Multi-Head Latent Attention(MLA)** 이 더해진다. 트랜스포머 모델의 고질적 병목인 KV 캐시 메모리를 획기적으로 줄이는 기술이다. 128K 토큰의 컨텍스트 윈도우를 지원하면서도 메모리 효율이 높아, 루미큐브처럼 긴 게임 히스토리를 포함하는 프롬프트에 적합하다.

비용 면에서 이 아키텍처의 위력은 압도적이다. 입력 $0.55/M토큰, 출력 $2.19/M토큰. OpenAI o1의 입력 $15, 출력 $60과 비교하면 **27분의 1** 가격이다. 캐시 히트 시에는 입력이 $0.014/M토큰까지 떨어진다. 학습 비용도 512개 H800 GPU로 약 $294,000 — GPT-4 학습 비용의 수십 분의 일이다.

---

## 2. 추론의 탄생 — 강화학습만으로 생각하는 법을 배우다

DeepSeek-R1의 가장 놀라운 점은 학습 방법에 있다. 보통 추론 능력을 가르치려면 인간이 만든 "이렇게 생각해라"라는 예시(Supervised Fine-Tuning, SFT)가 필요하다. 하지만 DeepSeek 팀은 대담한 가설을 세웠다.

> *"정답 여부만 알려주고, 생각하는 방법은 모델이 스스로 찾게 하면 어떨까?"*

이렇게 탄생한 것이 **DeepSeek-R1-Zero**다. SFT 없이, 오직 강화학습(RL)만으로 훈련된 모델이다. 보상 신호는 단 하나 — 최종 답이 맞는지 틀리는지. 풀이 과정에는 어떤 제약도 걸지 않았다.

결과는 경이로웠다. AIME 2024(미국 수학 초청 시험)에서 초기 15.6%이던 정답률이 훈련 과정에서 71.0%까지 치솟았다. 다수결 투표 적용 시 86.7%에 도달했는데, 이는 OpenAI o1-0912 수준이다. 그리고 이 과정에서 **아무도 가르치지 않은 행동들**이 나타났다.

---

## 3. "아하 순간" — 기계가 스스로 깨달음을 얻다

DeepSeek 팀이 R1-Zero의 훈련 과정을 관찰하던 중, 중간 체크포인트에서 놀라운 현상을 발견한다. 모델이 문제를 풀다가 갑자기 멈추고 이렇게 말한 것이다:

> *"Wait, wait. Wait. That's an aha moment I can flag here."*

모델이 초기 접근 방식에 문제가 있음을 스스로 깨닫고, 되돌아가서 재평가하는 순간이었다. DeepSeek 팀은 이것을 **"Aha Moment"** 라고 불렀다. 이 현상은 논문에서 가장 많이 인용되는 부분이 되었고, AI 연구 커뮤니티에 깊은 인상을 남겼다.

Aha Moment 이후 모델에서 다음과 같은 행동들이 자발적으로 출현(emerge)했다:

- **Self-reflection**: 이전 추론 단계를 되돌아보며 논리적 오류를 점검
- **Self-verification**: 도출한 답이 문제 조건을 만족하는지 자체 검증
- **Alternative exploration**: 하나의 접근이 막히면 다른 전략을 탐색
- **Dynamic strategy adaptation**: 문제의 난이도에 따라 사고 깊이를 조절

가장 주목할 점은 **사고 시간의 자발적 확장**이다. 논문은 이렇게 기술한다:

> *"The thinking time of DeepSeek-R1-Zero shows consistent improvement throughout the training process. This improvement is not the result of external adjustments but rather an intrinsic development within the model."*

복잡한 문제를 만나면 모델이 스스로 더 오래 생각하기로 결정한다. 수백 개에서 수천 개로 추론 토큰을 늘려가며 사고를 심화시킨다. 이것은 외부에서 강제한 것이 아니라, 보상 신호에 의한 자율적 발달이다.

---

## 4. 4단계 파이프라인 — 야생의 천재를 다듬다

R1-Zero는 뛰어났지만 문제가 있었다. 가독성이 떨어지고, 여러 언어가 뒤섞이며, 때로는 끝없이 반복했다. DeepSeek 팀은 이 "야생의 천재"를 다듬기 위해 4단계 파이프라인을 설계했다.

**Stage 1 — Cold Start**: 수천 개의 정리된 Chain-of-Thought 예시로 기초를 잡는다. `<reasoning>` 태그와 `<summary>` 태그로 구분된 형식을 학습시켜 가독성을 확보한다.

**Stage 2 — Reasoning RL**: R1-Zero와 동일한 대규모 강화학습. 다만 언어 일관성 보상을 추가하여 언어 혼합 문제를 해결한다. 수학, 코딩, 논리 추론에 집중.

**Stage 3 — Rejection Sampling & SFT**: RL 체크포인트에서 약 60만 개의 추론 샘플을 생성하고, 그중 정확한 것만 선별한다. 여기에 20만 개의 비추론 데이터(작문, 사실 QA)를 추가하여 총 80만 샘플로 미세조정.

**Stage 4 — All Scenarios RL**: 추론과 일반 태스크 모두에 대해 최종 강화학습. 도움말 평가는 최종 요약에, 무해성 평가는 전체 응답에 적용.

이 파이프라인의 결과가 DeepSeek-R1이다. AIME 2024에서 79.8%(o1의 79.2% 상회), MATH-500에서 97.3%(o1의 96.4% 상회), Codeforces에서 상위 96.3% 달성.

---

## 5. 증류의 마법 — 거인의 사고법을 소인에게

R1의 또 다른 혁신은 **지식 증류(distillation)** 다. R1이 생성한 80만 개의 추론 샘플로 작은 모델들을 훈련시켰더니, 놀라운 결과가 나왔다.

7B짜리 모델(R1-Distill-Qwen-7B)이 GPT-4o를 능가했다. 32B 모델은 o1-mini를 대부분의 벤치마크에서 넘어섰다. 더 충격적인 것은 — 같은 크기의 모델에 직접 RL을 적용한 것보다, R1의 출력을 보고 배운 쪽이 훨씬 뛰어났다는 점이다.

이것은 "생각하는 방법"이 전이 가능한 지식이라는 것을 의미한다. 거인의 사고 패턴을 관찰하는 것만으로도 소인이 거인처럼 생각할 수 있게 된다. 우리 프로젝트에서 사용하는 Ollama의 qwen2.5:3b가 루미큐브를 전혀 못 두는 것과 대조적이다 — 3B 규모에서는 아직 증류의 혜택이 충분히 미치지 못하는 것이다.

---

## 6. RummiArena에서의 DeepSeek — 데이터가 말하는 것

이제 우리 프로젝트의 데이터를 이 맥락에서 다시 읽어보자.

### 성장 궤적

| Round | Place Rate | 등급 | 비고 |
|-------|-----------|------|------|
| Round 2 (03-31) | 5% | F | v1 프롬프트, timeout 150초 |
| Round 3 (04-03) | 12.5% | C | v2 프롬프트 적용 |
| Round 4 (04-06) | 17.9~30.8% | A~A+ | max_tokens 16384 수정 |
| 클린 실행 (04-10) | 측정 중 | - | timeout 240초, fallback 0건 |

5%에서 30.8%로. 6배 성장이다. 그리고 이 성장의 핵심 동인은 모델이 변한 것이 아니라 **우리가 모델에게 맞는 환경을 제공한 것**이다. v2 프롬프트로 루미큐브 규칙을 더 명확히 전달했고, max_tokens를 충분히 확보했고, timeout을 늘려 추론을 끊지 않았다.

### 후반부 추론 폭발의 의미

클린 실행에서 관찰된 토큰 추이:

```
초반 (턴 1~5):  출력 2,073~3,415 토큰, 49~78초
중반 (턴 6~10): 출력 5,200~6,398 토큰, 120~169초
후반 (턴 15~): 출력 12,090~15,494 토큰, 265~349초
```

이것은 논문에서 기술한 "사고 시간의 자발적 확장"과 정확히 일치한다. 게임 초반에는 보드가 비어 있어 판단이 단순하다. 하지만 후반부에는 보드 위에 수십 개의 타일 조합이 깔려 있고, 자기 손에 남은 타일로 기존 조합을 재배치하거나 새로운 세트를 만들 수 있는 경우의 수가 폭발적으로 늘어난다.

DeepSeek Reasoner는 이 복잡도를 감지하고, 스스로 더 오래 생각하기로 결정한다. 15,000개의 추론 토큰 안에서 가능한 조합을 탐색하고, 자기 답을 검증하고, 더 나은 수가 있는지 되돌아본다. 349초가 걸린 턴에서 place에 성공한 것은 우연이 아니다. 충분히 생각했기 때문에 답을 찾은 것이다.

### 비용 대비 성능

| 모델 | 턴당 비용 | Place Rate | $/Place |
|------|----------|-----------|---------|
| GPT-5-mini | $0.025 | 30.8% | $0.081 |
| Claude Sonnet 4 | $0.074 | 33.3% | $0.222 |
| DeepSeek Reasoner | $0.001 | 30.8% | **$0.003** |

Place 하나를 성공시키는 데 드는 비용으로 보면, DeepSeek는 GPT의 **27분의 1**, Claude의 **74분의 1**이다. 동일한 $5 예산으로 GPT는 62회, Claude는 23회, DeepSeek는 **1,667회** place를 시도할 수 있다.

---

## 7. DeepSeek V4 — 다가오는 다음 세대

2026년 4월 현재, DeepSeek V4의 출시가 임박해 있다. 알려진 스펙은 놀랍다:

- **1조(1T) 파라미터** MoE, 활성화는 여전히 ~37B
- **1M 토큰 컨텍스트 윈도우** — Engram 조건부 메모리 아키텍처
- **네이티브 멀티모달** — 텍스트, 이미지, 비디오 생성
- **SWE-bench 81%** — 코딩 벤치마크 신기록
- 가격은 GPT-5 대비 약 **1/10**

Expert Mode와 Fast Mode의 도입도 진행 중이다. 추론이 필요한 태스크에는 Expert Mode가, 빠른 응답이 필요한 곳에는 Fast Mode가 적용된다. 이는 우리 프로젝트에서 루미큐브 대전(Expert Mode)과 일반 채팅(Fast Mode)을 구분하는 것과 유사한 접근이다.

---

## 나가며 — 시간을 주면 생각을 한다

DeepSeek Reasoner는 "저렴하지만 느린 대안"이 아니다. 강화학습만으로 사고하는 법을 스스로 체득한, 독특한 계보의 모델이다. 그 사고 과정에서 나타나는 self-reflection, verification, alternative exploration은 프로그래밍된 것이 아니라 보상 신호에 의해 자발적으로 출현한 행동이다.

우리가 RummiArena에서 관찰한 후반부 추론 토큰 폭발 — 2,000개에서 15,000개로의 증가 — 은 논문에서 기술한 "사고 시간의 자율적 확장"과 정확히 동일한 현상이다. 모델이 보드의 복잡도를 감지하고, 스스로 더 깊이 생각하기로 결정하는 것이다.

69% fallback은 DeepSeek의 한계가 아니었다. 우리의 한계였다. 200초라는 천장이 모델의 사고를 억지로 끊어버리고 있었다는 사실을 우리는 너무 늦게 깨달았다. 그리고 그 깨달음마저도, 데이터를 표면적으로 읽는 습관 때문에 몇 번의 잘못된 분석을 거쳐서야 도달했다.

DeepSeek에게 시간을 주면, 생각을 한다. 충분한 시간을 주면, 답을 찾는다. 턴당 $0.001의 비용으로.

그것을 존중하는 것이, 이 모델을 제대로 활용하는 첫 번째 조건이다.

---

## Sources

- [DeepSeek-R1 논문 (arXiv)](https://arxiv.org/abs/2501.12948)
- [DeepSeek-R1 Nature 게재](https://www.nature.com/articles/s41586-025-09422-z)
- [DeepSeek-R1 Architecture Analysis (HiddenLayer)](https://hiddenlayer.com/innovation-hub/analysing-deepseek-r1s-architecture)
- [DeepSeek Inference Cost Explained (IntuitionLabs)](https://intuitionlabs.ai/articles/deepseek-inference-cost-explained)
- [DeepSeek R1 vs OpenAI o1 Comparison (Vellum AI)](https://www.vellum.ai/blog/analysis-openai-o1-vs-deepseek-r1)
- [DeepSeek R1 Deep Dive (Fireworks AI)](https://fireworks.ai/blog/deepseek-r1-deepdive)
- [DeepSeek-V3 & R1 Technical Reports (Graphcore)](https://graphcore-research.github.io/deepseek/)
- [DeepSeek R1 Aha Moment Analysis (HuggingFace Blog)](https://huggingface.co/blog/NormalUhr/deepseek-r1-explained)
- [Artificial Analysis - DeepSeek](https://artificialanalysis.ai/providers/deepseek)
- [DeepSeek V4 Specs (NxCode)](https://www.nxcode.io/resources/news/deepseek-v4-release-specs-benchmarks-2026)
- [LLM API Pricing 2026 (TLDL)](https://www.tldl.io/resources/llm-api-pricing-2026)
- [Sebastian Raschka - Technical Tour of DeepSeek Models](https://magazine.sebastianraschka.com/p/technical-deepseek)