---
title: "\"Claude Code 시대의 종말\""
date: 2026-07-22 20:00:00 +0900
categories: [AI,  Claude Code]
mermaid: [True]
tags: [AI,  claude-code,  Gemini-3.6,  GPT-5.6,  Opus-4.8,  Fable-5,  Kimi-K3,  Claude.write]
---


- 원문 출처: Threads, [@vesperchant](https://www.threads.com/@vesperchant/post/DbEgGWjEzjQ), 2026-07-22 게시
- 본 문서 작성일자: 2026-07-22

> 
> <Claude Code 시대의 종말, Gemini 3.6 Flash 후기>
> 
> 우선, 아직 잘 모르는 분들 위한 기초 설명
> 1. 현세대 AI는 자연어 기반으로 동작하는 초고성능 계산기이나, 많이들 생각하는 `최고 정확도`가 아닌 `최대 근사도` 결과를 도출.
> 2. AI 모델 이름 다르다고 완전 별개가 아니라, 베이스 모델을 근간으로 각 유저의 요청마다 컴퓨팅 자원 투입 수준과 구동 시간에 차등을 두도록 제품화한 것임.
> 3. 낮은 성능의 모델을 충분히 오랜시간 돌리면 상위 모델보다 더 높은 성능이 나올 수 있음을 발견, 당시 2024년 o1 모델 출시가 그 상용화 대표적인 사례임.
> 4. 현재 GPT, Gemini 계열은 모델별로 위 1번 설명의 차등 경계가 체감상으로도 명확히 보이는 중이고, 모델 수준과 버전의 사용 설계에서 일관성이 잘 보임.
> 
> --
> 
> 이러한 기술적 배경 위에서 오늘의 Claude 모델과 다양한 모드는 제대로 동작하는지 따져봐야할 시점이 결국 와버린 것 같다.


---

## 목차

1. 문서의 목적과 접근 방식
2. 원문 핵심 주장 정리 (서술형 요약)
3. 먼저 이해해야 할 배경 개념
4. 언급된 모델들의 현재 상태 — 항목별 사실관계 검증
   4.1 Gemini 3.6 Flash + Antigravity
   4.2 GPT-5.6 Sol / Terra / Luna
   4.3 Claude Opus 4.8
   4.4 Claude Fable 5 — 수출통제, 접근 롤러코스터, 가격, 사용한도
5. 벤치마크 교차 검증 — Artificial Analysis 두 지표 비교
6. "월 2부제"라는 표현, 어디까지가 사실이고 어디부터가 수사인가
7. 원문 주장 총정리 — 사실 / 의견 / 추측 구분표
8. 더 넓은 맥락 — Claude Code를 둘러싼 그간의 논쟁
9. 종합 판단
10. 참고자료

---

## 1. 문서의 목적과 접근 방식

이 문서는 Threads 사용자 @vesperchant가 2026년 7월 22일에 올린 "Claude Code 시대의 종말"이라는 제목의 글을 대상으로, 그 안에 담긴 개별 주장들을 하나씩 떼어내어 사실관계를 확인한 결과물입니다. 원문은 크게 두 층위로 이루어져 있습니다. 하나는 현세대 AI 모델의 작동 방식에 대한 일반론적 설명이고, 다른 하나는 그 위에서 전개되는 저자 개인의 체감과 평가, 즉 Anthropic의 최근 모델 운영 방식에 대한 실망과 비판입니다. 앞의 것은 검증 가능한 사실 진술에 가깝고, 뒤의 것은 저자 한 사람의 업무 경험에서 나온 주관적 판단에 가깝습니다. 이 둘을 섞지 않고 따로 다루는 것이 이 문서의 핵심 원칙입니다.

검증에는 2026년 7월 22일 현재 시점에서 확인 가능한 웹 검색 결과를 사용했으며, 서로 다른 출처의 수치가 충돌하는 경우에는 어느 쪽이 더 권위 있는 출처인지 밝히고, 나머지는 참고용으로만 남겨두었습니다. 추측이 필요한 대목에서는 추측이라고 명시했고, 확인이 안 되는 부분은 확인이 안 된다고 그대로 적었습니다.

---

## 2. 원문 핵심 주장 정리 (서술형 요약)

저자는 우선 네 가지 전제를 깔아둡니다. 첫째, 오늘날의 생성형 AI는 자연어로 동작하는 계산기이지만 그 결과물은 '가장 정확한 답'이 아니라 '가장 그럴듯한 근사치'라는 것. 둘째, 같은 이름 아래 존재하는 여러 모델(예: Sol/Terra/Luna, 혹은 Max/Ultra 같은 모드)은 서로 다른 모델이라기보다, 같은 기반 모델에 얼마나 많은 컴퓨팅 자원과 시간을 투입하느냐를 상품화한 것에 가깝다는 것. 셋째, 2024년 OpenAI의 o1 모델 출시 이후, 상대적으로 낮은 성능의 모델이라도 충분히 오랜 시간 추론시키면 더 상위 모델의 결과를 능가할 수 있다는 사실(테스트 타임 컴퓨트, test-time compute)이 실증되었다는 것. 넷째, 현재 GPT 계열과 Gemini 계열은 이 등급 구분이 사용자 입장에서도 비교적 선명하게 느껴지는 반면, Claude 쪽은 그렇지 않다는 체감입니다.

이 전제 위에서 저자는 최근 이틀 넘게 Gemini 3.6 Flash(High 모드)와 Google Antigravity 조합으로 실제 고객사 업무를 처리해 본 경험을 근거로, Claude Opus 4.x 계열은 이제 "고전(구식)"이 되어버렸다고 평가합니다. 평소 GPT-5.6 Sol과 Gemini 3.6 Flash(저자는 이를 지연되고 있는 Gemini 3.5 Pro의 사실상의 리브랜딩으로 의심합니다)를 각각 Codex, Antigravity로 매일 사용하는 입장에서 볼 때, 최근의 Claude Code와 Claude 신규 모델들이 오히려 업무의 병목이 되고 있다는 것이 저자의 핵심 체감입니다.

이어 저자는 Anthropic이 신제품 출시 때는 성능을 대대적으로 홍보해 놓고, 정작 사용자에게는 그 성능을 "절반만" 쓰게 하는 방식에 대해 강한 실망감을 표합니다. 구체적으로는 Claude Fable 5의 API 과금이 분당 상당한 비용이 들 정도로 비싸다는 점, 그리고 경쟁사들은 시도한 적 없는 방식으로 최상위 모델의 월간 사용을 제한한다는 점(저자의 표현으로 "월간 2부제")을 지적합니다. 저자는 이런 고비용·고연산 기반의 과금 정책 자체는 논리적으로 이해할 수 있다고 인정하면서도, 비슷하거나 그 이하의 비용으로 쓸 수 있는 경쟁사의 최신 최상위 모델이 이미 두 갈래나 존재하는 상황에서 Fable 5를 굳이 비싸게 쓸 이유가 있는지 의문을 제기합니다. 글은 2025년 한때 업계를 압도했던 것으로 기억되는 Claude Code의 전성기가 다시 오지 않을 것 같다는 아쉬움, 그리고 Anthropic이 마케팅으로 쌓아 올린 이미지가 결국 마케팅 방식 자체 때문에 흔들리고 있다는 관찰로 마무리됩니다.

---

## 3. 먼저 이해해야 할 배경 개념

본격적인 사실 검증에 들어가기 전에, 원문이 전제로 깔고 있는 두 가지 개념은 별도의 검증이 필요 없는, 업계에서 이미 널리 받아들여지는 사실에 가깝습니다.

첫째, LLM이 "최대 근사도"의 결과를 낸다는 설명은 트랜스포머 기반 언어모델의 확률적 다음 토큰 예측이라는 근본 구조를 감안하면 개념적으로 타당한 서술입니다. 이는 특정 수치를 검증해야 하는 사실 주장이라기보다, LLM의 작동 원리에 대한 일반적 설명입니다.

둘째, 하나의 모델 이름 아래 여러 등급(Effort, Max, Ultra 등)이 존재하고 이것이 컴퓨팅 자원 투입량의 차등화라는 설명도 실제로 업계에서 채택하고 있는 제품화 방식과 일치합니다. 아래에서 다룰 GPT-5.6의 Sol/Terra/Luna 3단 티어, 그리고 별도 모드인 Ultra(4개 에이전트를 병렬로 돌려 연산량을 의도적으로 늘리는 방식)가 대표적인 사례이며, Anthropic 쪽에서도 Claude Code에 effort 파라미터를 도입해 추론 강도를 조절하도록 한 바 있습니다.

셋째, 2024년 OpenAI o1 출시로 대중화된 테스트 타임 컴퓨트 개념, 즉 추론 단계에 더 많은 시간과 연산을 투입하면 더 작은 기반 모델로도 더 큰 모델을 능가하는 결과를 낼 수 있다는 것은 2024년 하반기 이후 업계에서 반복적으로 확인된 사실입니다. 이 부분은 역사적으로 이미 정립된 사실이라 별도의 실시간 검색 없이도 신뢰도 높게 서술할 수 있는 대목입니다.

이 세 가지 전제는 원문 전체의 논리 구조를 지탱하는 토대이며, 이후에 이어지는 저자의 체감 평가와는 성격이 다른, 비교적 검증된 배경 지식입니다.

---

## 4. 언급된 모델들의 현재 상태 — 항목별 사실관계 검증

### 4.1 Gemini 3.6 Flash + Antigravity

저자가 언급한 "Gemini 3.6 Flash (High) + Antigravity" 조합은 실제로 존재하는 최신 조합입니다. Gemini 3.6 Flash는 2026년 7월 21일 무렵 Google Antigravity(구글의 에이전트 기반 IDE)에서 처음 식별자(gemini-3.6-flash-tiered)가 커뮤니티에 의해 발견되었고, 이후 Google이 공식 블로그를 통해 정식으로 발표했습니다. Google Antigravity 공식 블로그는 이 모델이 코딩, 지식노동, 멀티모달 성능에서 이전 세대인 Gemini 3.5 Flash보다 개선되었고, 출력 토큰 사용량을 최대 17퍼센트까지 줄이면서도 3.5 Flash보다 토큰당 비용이 낮다고 설명합니다. thinking_level을 minimal/medium/high로 조절할 수 있다는 점도 Google 공식 개발자 문서에 명시되어 있어, 저자가 언급한 "High" 모드는 이 thinking_level 설정을 가리키는 것으로 보입니다.

다만 저자가 3.6 Flash를 "지연되고 있는 Gemini 3.5 Pro의 택갈이(리브랜딩)"로 의심한 부분은, 실제로 일부 해외 매체에서도 유사한 추측이 제기된 바 있습니다. Gemini 3.5 Pro는 애초 2026년 6월 출시 예정이었으나 코딩 벤치마크에서 기대에 못 미치는 결과가 나오면서 지연되었고, 6월 말 재훈련에도 만족스럽지 않은 결과가 나왔다는 보도가 있었습니다. 이 때문에 일부 분석에서는 3.6 Flash가 지연된 3.5 Pro의 공백을 메우는 임시 조치일 수 있다는 추측을 내놓았습니다. 다만 이는 어디까지나 커뮤니티 차원의 추측이며 Google이 공식적으로 이런 관계를 인정한 적은 없습니다. 따라서 저자의 의심은 "근거 없는 억측"은 아니지만, "확인된 사실"도 아닌, 정황상 그럴듯한 추측 수준으로 분류하는 것이 정확합니다.

### 4.2 GPT-5.6 Sol / Terra / Luna

GPT-5.6은 2026년 7월 9일 정식 일반공개(GA)되었으며, Sol·Terra·Luna 세 단계 티어로 구성되어 있습니다. API 가격은 100만 토큰 기준으로 Sol이 입력 5달러/출력 30달러, Terra가 입력 2.5달러/출력 15달러, Luna가 입력 1달러/출력 6달러입니다. 여기에 더해 여러 에이전트를 병렬로 돌려 응답 품질과 속도를 동시에 끌어올리는 별도 고연산 모드인 Ultra가 있으며, 이는 Codex와 ChatGPT Work의 Plus 이상 플랜에서 제공됩니다. Ultra 모드는 예를 들어 Terminal-Bench 2.1에서 Sol의 점수를 88.8퍼센트에서 91.9퍼센트까지 끌어올리는 것으로 보고되었는데, 이는 대략 4배 가량의 토큰 소모를 대가로 하는 것으로 알려져 있습니다.

저자가 매일 사용 중이라고 언급한 GPT-5.6 Sol은 독립 벤치마크 기관인 Artificial Analysis 기준으로 Claude Fable 5에 근소하게 못 미치는 지능 지수를 기록하면서도, 작업당 실제 비용은 Fable 5의 3분의 1 수준에 그친다는 점이 여러 출처에서 공통적으로 확인됩니다. 이 부분은 뒤의 5번 항목에서 표로 자세히 다룹니다.

### 4.3 Claude Opus 4.8

Claude Opus 4.8은 2026년 5월 말에 출시된 모델로, 가격은 100만 토큰당 입력 5달러/출력 25달러입니다. 출시 당시 Anthropic 스스로 "겸손한 개선(honest upgrade)"이라는 표현을 쓸 정도로 극적인 성능 도약보다는 정직성 개선과 Claude Code의 Dynamic Workflows, 그리고 더 저렴한 고속 모드 도입에 초점을 맞춘 업데이트였다는 점이 국내외 기술 매체 보도에서 공통적으로 확인됩니다.

2026년 7월 현재 시점에서 Opus 4.8은 Artificial Analysis Intelligence Index 기준 56점 안팎으로, Claude Fable 5(60점)나 GPT-5.6 Sol(59점)보다 낮은 지능 지수를 기록하고 있습니다. 다만 SWE-bench 계열 벤치마크에서는 평가 기준(Verified냐 Pro냐)에 따라 보고되는 수치가 크게 다르므로 주의가 필요합니다. 6월 중순 기준으로는 Opus 4.8이 SWE-bench Verified에서 88.6퍼센트를 기록했다는 보도가 있었던 반면, 7월 하순의 다른 보도에서는 더 어려운 문항으로 구성된 SWE-bench Pro 기준으로 69.2퍼센트라는 수치가 언급됩니다. 이 두 수치는 서로 다른 벤치마크 세트를 가리키는 것이므로 "성능이 떨어졌다"는 식으로 단순 비교해서는 안 됩니다.

한편 Claude Code 커뮤니티 안에서는 Opus 4.7·4.8로 이어지는 과정에서 "이전 버전과 체감이 다르다"거나 "위화감이 든다"는 식의 반응이 국내외에서 반복적으로 관찰되었고, 제3자 기관이 SWE-bench Pro의 일부 문항을 매일 재현 측정해 통계적으로 유의미한 성능 변동을 추적하는 시도도 있었습니다(자세한 내용은 8번 항목 참고). 이런 정황은 저자가 느낀 "체감 저하"가 저자 한 사람만의 고립된 인상은 아니라는 근거는 될 수 있지만, 그 자체가 Opus 4.8이 "구식"이라는 것을 객관적으로 증명하지는 않습니다.

### 4.4 Claude Fable 5 — 수출통제, 접근 롤러코스터, 가격, 사용한도

Claude Fable 5는 Anthropic이 새로 도입한 Opus보다 상위 등급인 Mythos급 모델의 대중 공개판으로, 2026년 6월 9일 Claude Mythos 5와 함께 최초 출시되었습니다. 가격은 API 기준 100만 토큰당 입력 10달러/출력 50달러로, 이는 GPT-5.6 Sol(5달러/30달러)의 정확히 2배 수준입니다.

이 모델의 접근성은 지난 한 달 반 동안 매우 불안정했습니다. 정리하면 다음과 같은 흐름입니다.

- 2026년 6월 9일: Fable 5 / Mythos 5 최초 출시
- 2026년 6월 12일: 미국 상무부의 수출통제 조치에 따라 Anthropic이 두 모델에 대한 접근을 일시 중단
- 2026년 6월 30일: 해당 수출통제 해제
- 2026년 7월 1일: Fable 5가 Claude Platform, Claude.ai, Claude Code, Claude Cowork 전반에서 전 세계 사용자에게 재개
- 2026년 7월 7일까지(일부 보도에서는 7월 19일까지로 안내되기도 함): Pro, Max, Team, 일부 Enterprise 플랜에서 주간 사용한도의 최대 50%까지만 Fable 5 사용이 플랜에 포함되고, 초과분은 별도 사용량 크레딧으로 과금
- 이후 프로모션 종료: Fable 5가 플랜 기본 사용한도에서 완전히 빠지고 사용량 크레딧 기반으로 전환
- 2026년 7월 21일 전후: 재조정을 거쳐 Max와 Team Premium 플랜에는 Fable 5가 표준 사용한도의 약 50% 수준으로 다시 포함되고, Pro와 Team Standard 플랜은 계속 사용량 크레딧 방식을 쓰되 1회성 100달러 크레딧을 지급받는 구조로 안정화

이 과정에서 국내 커뮤니티(클리앙 등)에서는 "예전에는 100퍼센트 사용을 2주가량 주기로 제공했는데, 이번에는 일주일도 안 되는 기간 동안 50퍼센트만 준다"는 식의 불만이 다수 확인되며, 실제 사용자들 사이에서 한도 소모 속도가 매우 빠르다는 공통된 반응도 나타납니다. 또한 Fable 5는 더 엄격해진 안전 분류기를 함께 적용받고 있어, 사이버보안·병원체 연구·위험 화학물질 합성 등 민감한 요청으로 분류되면 별도 고지 없이 혹은 고지와 함께 Opus 4.8로 요청이 우회 처리될 수 있다는 점도 여러 매체에서 공통적으로 확인됩니다.

정리하면, 저자가 "비싸게 줘야 하고 그마저도 절반만 쓰게 한다"고 표현한 부분은 가격 측면(Sol 대비 2배)과 사용한도 측면(적어도 프로모션 기간 동안 주간 한도의 50% 제한이 실제로 존재했다는 점) 모두에서 사실에 부합하는 서술입니다.

---

## 5. 벤치마크 교차 검증 — Artificial Analysis 두 지표 비교

독립 벤치마크 기관인 Artificial Analysis는 서로 성격이 다른 두 지표를 운영하고 있으며, 두 지표에서 순위가 엇갈린다는 점이 이번 논쟁을 이해하는 데 중요합니다. 하나는 종합 지능을 측정하는 Intelligence Index이고, 다른 하나는 실제 코딩 에이전트 하네스(Codex, Claude Code 등)와 모델을 짝지어 평가하는 Coding Agent Index입니다.

| 모델 (max 모드 기준) | 제공사 | Intelligence Index (2026-07-11 기준) | Coding Agent Index | API 가격 (입력/출력, 100만 토큰) |
|---|---|---|---|---|
| Claude Fable 5 | Anthropic | 60 (세부치 59.9) | 77 | $10 / $50 |
| GPT-5.6 Sol | OpenAI | 59 (세부치 58.9) | 80 (1위) | $5 / $30 |
| GPT-5.6 Terra | OpenAI | — | 77 | $2.50 / $15 |
| GPT-5.6 Luna | OpenAI | — | 75 | $1 / $6 |
| Claude Opus 4.8 | Anthropic | 56 | — | $5 / $25 |
| GPT-5.5 | OpenAI | 55 | — | — |
| Kimi K3 (2026-07-21 별도 스냅샷) | Moonshot AI | 57.1 | — | — |

이 표에서 확인할 수 있는 핵심은 두 가지입니다. 첫째, 종합 지능 지표에서는 Fable 5가 여전히 1위이지만 GPT-5.6 Sol이 단 1점 차이로 근접해 있고, 이때 실제 작업당 비용은 Sol이 약 1.04달러, Fable 5가 약 2.75달러로 Sol이 대략 3분의 1 수준입니다. 둘째, 실제 코딩 에이전트 작업만 놓고 보는 Coding Agent Index에서는 순위가 뒤집혀 GPT-5.6 Sol(Codex 하네스 기준)이 80점으로 1위이고 Fable 5(Claude Code 하네스 기준)가 77점으로 2위이며, 심지어 더 저렴한 GPT-5.6 Terra도 77점으로 Fable 5와 동률입니다. Artificial Analysis 공식 자료에 따르면 Sol의 작업당 비용은 Claude Code 환경에서 Fable 5보다 약 40퍼센트, Opus 4.8보다 약 10퍼센트 더 저렴합니다.

이 두 지표를 종합하면, 저자가 체감한 "GPT-5.6 Sol 중심 워크플로가 실제로 더 낫더라"는 인상은 적어도 코딩 에이전트 실전 작업이라는 좁은 영역에서는 벤치마크로도 뒷받침되는 관찰입니다. 다만 종합적인 지능 자체는 Fable 5가 여전히 근소하게 앞선다는 점, 그리고 Opus 4.8은 두 지표 모두에서 Fable 5나 Sol보다는 낮은 위치에 있지만 그 격차가 "구식"이라고 부를 만큼 극단적이지는 않다는 점도 함께 짚어둘 필요가 있습니다.

---

## 6. "월 2부제"라는 표현, 어디까지가 사실이고 어디부터가 수사인가

원문에서 가장 눈에 띄는 표현 중 하나는 "경쟁사들은 해본 적도 없는 하이엔드 모델의 월간 2부제 사용"이라는 대목입니다. 이 표현에서 "2부제"는 한국에서 흔히 쓰이는 차량 홀짝제 같은 격일제 규제를 연상시키는 비유입니다. 검색 결과를 종합하면, Anthropic이 "2부제"라는 명칭이나 그에 대응하는 격일 단위의 공식 제도를 발표한 사실은 확인되지 않습니다. 다만 그 비유가 가리키는 실제 정책, 즉 최상위 모델의 사용량을 플랜 기본 한도 중 절반으로 인위적으로 축소해 운영하는 방식(주간 한도의 50% 제한), 그리고 그 한도와 정책이 6월부터 7월 사이에 여러 차례 바뀌며 사용자 입장에서는 예측하기 어려운 형태로 운영된 것은 실제로 존재했던 사실입니다.

따라서 "2부제"라는 단어 자체는 저자가 만든 수사적 표현으로 보는 것이 정확하며, 그 수사가 가리키는 실질적 내용(사용량 절반 제한, 반복적인 정책 변경, 상대적으로 비싼 가격)은 사실관계로 뒷받침됩니다. 이 구분, 즉 "비유는 저자의 것이지만 비유의 근거는 사실"이라는 점을 분명히 해 두는 것이 이 문서의 목적에 맞습니다.

---

## 7. 원문 주장 총정리 — 사실 / 의견 / 추측 구분표

| 원문 주장 | 분류 | 검증 결과 |
|---|---|---|
| LLM은 최대 근사도 결과를 낸다 | 개념 설명 | 트랜스포머 기반 LLM의 일반적 특성에 대한 타당한 서술 |
| 모델 등급은 컴퓨팅 자원 차등화 상품이다 | 개념 설명 | GPT-5.6의 Sol/Terra/Luna, Ultra 모드 등에서 실제로 확인되는 제품화 방식 |
| 2024년 o1 이후 테스트 타임 컴퓨트가 실증됨 | 역사적 사실 | 이미 정립된 업계 사실 |
| GPT/Gemini는 등급 경계가 명확, Claude는 불명확 | 주관적 체감 | 정량적으로 검증하기 어려운 저자 개인의 인상 |
| Gemini 3.6 Flash + Antigravity 조합이 실재한다 | 사실 | 2026년 7월 21일 전후 확인, Google 공식 발표로 뒷받침됨 |
| 3.6 Flash는 지연된 3.5 Pro의 리브랜딩일 수 있다 | 추측 | 일부 매체의 정황상 추측일 뿐, Google 공식 인정 없음 |
| Opus 4.x가 "고전(구식)"이 되었다 | 주관적 평가 | 벤치마크상 Fable5·Sol보다 낮은 점수인 것은 사실이나 "구식"이라는 표현 자체는 저자의 가치판단 |
| Fable 5가 Sol보다 비싸다 | 사실 | $10/$50 대 $5/$30로 약 2배 |
| Fable 5 접근이 절반으로 제한된 시기가 있었다 | 사실 | 2026년 7월 초 주간 한도 50% 제한이 실제로 시행됨 |
| "월간 2부제"라는 공식 제도가 있다 | 수사적 표현 | 공식 명칭 아님. 다만 그 취지에 해당하는 사용량 제한 정책은 실재 |
| Fable 5보다 절반 이하 비용으로 쓸 수 있는 하이엔드 경쟁 모델이 두 갈래 있다 | 사실에 근접 | GPT-5.6 Sol이 Fable5 대비 절반 가격대이며 지능 지수도 근접. Gemini 계열은 가격 공개가 제한적이나 Flash 라인은 대체로 더 저렴 |
| 2025년 Claude Code의 "전성기"가 다시 오지 않을 것 같다 | 주관적 회고 | 검증 대상이 아닌 저자의 정서적 서술 |

---

## 8. 더 넓은 맥락 — Claude Code를 둘러싼 그간의 논쟁

이번 게시물은 고립된 사건이 아니라, 지난 몇 달간 이어져 온 Claude Code 성능 체감 논쟁의 연장선에 있습니다. Opus 4.7 출시 무렵부터 일본어권 커뮤니티에서는 "Opus 4.8로 바뀐 뒤 Claude Code의 동작이 이질적으로 느껴진다"는 식의 반응이 나타났고, 국내 기술 커뮤니티에서도 유사한 톤의 게시물이 반복적으로 공유되었습니다. 특히 주목할 만한 것은, 제3자 독립 기관이 SWE-bench Pro의 문항 일부를 매일 재현 실행하여 Claude Code / Opus 4.5 계열의 성능을 추적한 사례입니다. 이 추적에서는 최근 30일 평균 통과율이 기준선 대비 통계적으로 유의미하게 하락한 구간이 관찰되었는데, 그 원인에 대해서는 A/B 테스트 중인 여러 체크포인트의 영향일 수 있다는 추정, GPU 부하에 따른 동적 성능 조정 가능성, 혹은 단순한 표본 변동일 가능성 등 여러 해석이 커뮤니티 내에서 엇갈렸을 뿐 하나로 확정되지는 않았습니다.

이런 배경은 저자의 이번 게시물이 왜 많은 공감을 얻을 수 있는 정서적 토양을 갖고 있는지를 설명해 줍니다. 다만 동시에, "성능 체감 저하"와 "정책적으로 절반만 쓰게 강제한다"는 것은 서로 다른 층위의 문제라는 점도 구분해서 볼 필요가 있습니다. 전자는 모델 자체의 실제 출력 품질에 관한 것이고, 후자는 그 모델에 접근할 수 있는 한도와 가격에 관한 것입니다. 원문은 이 둘을 함께 엮어 하나의 큰 실망감으로 서술하고 있지만, 사실관계 차원에서는 각각 따로 확인되어야 하는 별개의 사안입니다.

---

## 9. 종합 판단

이번 게시물을 관통하는 두 개의 굵은 뼈대, 즉 (1) 컴퓨팅 자원 투입량에 따라 모델 등급이 차등화된다는 산업 전반의 구조와 (2) 그 구조 위에서 Anthropic이 유독 접근성과 가격 정책을 예측 불가능하고 인색하게 운영해왔다는 지적은, 이번 검증 결과 대체로 사실관계에 부합합니다. Claude Fable 5는 실제로 GPT-5.6 Sol보다 약 2배 비싸고, 종합 지능 지수에서는 근소하게 앞서지만 정작 실전 코딩 에이전트 지표에서는 오히려 뒤처지며, 지난 한 달 반 동안 수출통제로 인한 서비스 중단과 재개, 그리고 사용한도를 둘러싼 여러 차례의 정책 변경을 거쳐왔습니다. Claude Opus 4.8 역시 두 벤치마크 모두에서 Fable 5와 Sol에 뒤처지는 위치에 있는 것은 사실입니다.

다만 "고전이 되었다", "역체감으로 깨달았다", "다시는 오지 않을 것 같다", "2부제"와 같은 표현들은 수치로 증명되는 사실이 아니라 저자가 자신의 실무 경험을 바탕으로 내린 가치판단이자 정서적 서술입니다. 이 부분은 동의하거나 동의하지 않을 수는 있어도, "맞다/틀리다"를 가릴 수 있는 성격의 진술은 아닙니다. 결국 이번 게시물은 검증 가능한 사실들(가격 격차, 벤치마크 순위 역전, 접근 제한 이력)과 그 사실들에 대한 한 실무자의 강한 정서적 반응이 함께 엮인 글로 이해하는 것이 가장 정확합니다.

---

## 10. 참고자료

- Google Antigravity 공식 블로그, "Gemini 3.6 Flash in Google Antigravity" — https://antigravity.google/blog/gemini-3-6-flash-in-google-antigravity
- Google 공식 블로그, "Introducing Gemini 3.6 Flash, 3.5 Flash-Lite, and 3.5 Flash Cyber" — https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/
- Google AI for Developers, "Using the latest Gemini models" — https://ai.google.dev/gemini-api/docs/latest-model
- officechai.com, "Google's Gemini Flash 3.6 Spotted In Antigravity, Could Launch Soon" (2026-07-21 전후) — https://officechai.com/ai/googles-gemini-flash-3-6-spotted-in-antigravity-could-launch-soon/
- Artificial Analysis 공식 아티클, "GPT-5.6 has landed" — https://artificialanalysis.ai/articles/gpt-5-6-has-landed
- apidog.com, "GPT-5.6 Pricing Explained" (2026-07-09 GA 기준) — https://apidog.com/blog/gpt-5-6-pricing/
- 오픈위키(wikidocs.net), "GPT-5.6 가격·구독 가이드" (2026-07-11 기준) — https://wikidocs.net/blog/@openwiki/23129/
- aivy.com.au, "GPT-5.6 vs Claude Fable 5 and Opus 4.8: the verdict" (2026-07-11 AA 지표 인용) — https://aivy.com.au/resources/gpt-5-6-vs-claude-fable-5/
- cruxdigits.nl, "GPT-5.6 vs Claude Fable 5, Gemini & Chinese Models" — https://cruxdigits.nl/blog/gpt-5-6-vs-claude-fable-5/
- benchlm.ai, "Artificial Analysis Intelligence Index Leaderboard" (2026-07-21 스냅샷) — https://benchlm.ai/benchmarks/artificialAnalysis
- felloai.com, "Best AI Models in July 2026" — https://felloai.com/best-ai-models/
- divkix.me, "Best AI Models July 2026" — https://divkix.me/blog/ai-models-compared-2026/
- OpenRouter, Claude Fable 5 모델 페이지 — https://openrouter.ai/anthropic/claude-fable-5
- androidauthority.com, "Claude's Fable 5 is now easier to access if you're a subscriber" (2026-07-21 전후) — https://www.androidauthority.com/claude-fable-5-access-3689068/
- 이랜서 블로그, "Claude Fable 5 사용법" — https://www.elancer.co.kr/blog/detail/1110
- 오픈위키(wikidocs.net), "클로드 Fable 5 사용법과 요금" — https://wikidocs.net/blog/@openwiki/22388/
- 클리앙, "Fable5 주간한도 50%만 사용 가능??" — https://www.clien.net/service/board/park/19219726
- 짐코딩 블로그, "Claude Fable 5 완전정리" — https://www.gymcoding.co/blog/claude-fable-5-practical-guide
- 나무위키, "Claude/모델" 항목 (수출통제·재개 타임라인) — https://namu.wiki/w/Claude/%EB%AA%A8%EB%8D%B8
- DataCamp, "Claude Opus 4.8: Anthropic's More Honest Flagship Model" — https://www.datacamp.com/blog/claude-opus-4-8
- 박재홍의 실리콘밸리(wikidocs.net), "Claude Opus 4.8, 겸손한 업그레이드가 던지는 진짜 질문" — https://wikidocs.net/blog/@jaehong/15318/
- GeekNews(news.hada.io), "Claude Code Opus 4.5 SWE 작업 성능 추적기" — https://news.hada.io/topic?id=26243
- ofox.ai, "Best AI Models June 2026" (Fable5 수출통제 중단 관련 6월 시점 기록) — https://ofox.ai/blog/llm-leaderboard-best-ai-models-ranked-2026/

---

작성일자: 2026-07-22
