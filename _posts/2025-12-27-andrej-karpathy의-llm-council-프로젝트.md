---
title: "Andrej Karpathy의 LLM Council 프로젝트"
date: 2025-12-27
categories: [AI,  Material]
mermaid: [True]
tags: [AI,  AndrejKarpathy,  skill-issue,  llm-council,  Claude.write]
---


Andrej Karpathy의 LLM Council 프로젝트는 그가 2025년 12월 27일 올린 [**“스킬 이슈”** 트윗](https://x.com/karpathy/status/2004607146781278521?s=20)의 완벽한 실천 사례입니다. 이것은 단순한 주말 프로젝트가 아니라, AI 개발의 새로운 패러다임인 “vibe coding”과 멀티모델 오케스트레이션의 미래를 보여주는 중요한 실험입니다.

## **LLM Council의 구조와 작동 원리**

카파시는 ChatGPT와 똑같이 생긴 웹 앱을 만들었지만, 뒷단에서는 완전히 다른 일이 벌어집니다. 사용자가 쿼리를 입력하면 3단계 워크플로우가 시작됩니다. 첫 번째 단계에서는 OpenRouter를 통해 GPT-5.1, Gemini 3 Pro Preview, Claude Sonnet 4.5, Grok-4 같은 최상위 모델들에게 동시에 질문을 보냅니다. 각 모델은 독립적으로 답변을 생성하는데, 이것이 “collect” 단계입니다.

두 번째 “rank” 단계가 흥미롭습니다. 각 모델은 익명화된 다른 모델들의 답변을 보고 평가하고 순위를 매깁니다. 누가 작성한 답변인지 모르는 상태에서 순수하게 품질만으로 판단하는 것입니다. 이것은 AI 피어 리뷰(peer review)의 실현입니다. 마지막 “synthesize” 단계에서는 “Chairman LLM”이 모든 답변과 평가를 종합해서 최종 응답을 생성합니다.

기술적으로는 FastAPI 백엔드와 React 프론트엔드로 구성된 “얇은 아키텍처(thin architecture)“입니다. 수백 줄의 Python과 JavaScript 코드로 구현되어 있으며, OpenRouter를 통해 여러 AI 제공업체의 모델을 단일 API로 통합합니다. 이것이 핵심입니다. OpenAI, Google, Anthropic 각각을 위한 별도의 통합 코드를 작성할 필요가 없습니다. 단순히 config 파일의 COUNCIL_MODELS 리스트만 수정하면 새 모델을 추가하거나 교체할 수 있습니다.

## **예상 밖의 발견: 모델들의 자기 평가**

카파시가 발견한 가장 흥미로운 점은 모델들이 놀랍도록 객관적으로 다른 모델을 평가한다는 것입니다. 자신의 응답보다 다른 모델의 응답이 더 우수하다고 인정하는 경우가 많았습니다. 책 읽기 세션에서 모델들은 일관되게 GPT-5.1을 가장 통찰력 있다고 칭찬했고, Claude를 최하위로 평가했습니다. Gemini 3와 Grok-4는 그 사이에 위치했습니다.

하지만 카파시 본인의 정성적 평가는 달랐습니다. 그는 GPT-5.1이 “너무 장황하고 산만하다(too wordy and sprawled)“고 느꼈고, Gemini 3가 “더 압축적이고 정제되어 있다(more condensed and processed)“고 평가했습니다. Claude는 “너무 간결하다(too terse)“는 것이 그의 판단이었습니다. 즉, AI 모델들의 집단 평가와 인간 전문가의 평가가 반드시 일치하지는 않는다는 것입니다.

이것은 중요한 시사점을 줍니다. AI 벤치마크와 자동 평가의 한계를 보여주는 것이기도 합니다. 카파시가 자신의 2025 연간 리뷰에서 “벤치마크에 대한 신뢰를 잃었다”고 고백한 것과도 연결됩니다. 벤치마크는 구조적으로 검증 가능한 환경이기 때문에 RLVR(Reinforcement Learning with Verifiable Rewards)에 취약합니다. 모델들이 벤치마크를 지배하면서도 실제 세계에서는 여전히 신뢰할 수 없는 상황이 가능한 것입니다.

## **Vibe Coding의 실천: 코드는 이제 일회용이다**

LLM Council은 카파시가 “99% vibe coded”했다고 밝힌 프로젝트입니다. 그는 GitHub 저장소에 “어떤 식으로도 지원하지 않을 것이다… 코드는 이제 일회용이고 라이브러리는 끝났다”라는 강력한 면책 조항을 달았습니다. 이것은 농담이 아니라 새로운 개발 패러다임의 선언입니다.

그의 2025 연간 리뷰에서 카파시는 올해 vibe coding으로 만든 여러 프로젝트를 나열합니다. menugen(메뉴 생성기), reader3(리더 앱), HN time capsule(해커뉴스 타임캡슐), 그리고 이 llm-council까지. 그는 심지어 “단 하나의 버그를 찾기 위해 일회용 앱 전체를 vibe coding했다”고 말합니다. 왜냐하면 “코드가 갑자기 무료이고, 일회용이고, 유연하고, 단 한 번 사용 후 버릴 수 있게 되었기 때문”입니다.

전통적인 소프트웨어 개발 관점에서는 이것이 끔찍해 보일 수 있습니다. 유지보수도 없고, 문서도 없고, 장기적 지원도 없습니다. 하지만 카파시는 정확히 이런 식으로 nanochat에서 Rust로 커스텀 BPE 토크나이저를 vibe coding했습니다. 기존 라이브러리를 채택하거나 Rust를 그 수준까지 배우는 대신에 말입니다.

## **엔터프라이즈 AI 오케스트레이션의 참조 아키텍처**

VentureBeat의 분석은 LLM Council의 진짜 가치를 정확히 짚어냅니다. 이것은 주말 장난감이 아니라 “2025년 후반 현대적이고 최소한의 AI 스택이 어떻게 생겼는지를 보여주는 1차 문서”입니다. 비록 카파시가 “지원하지 않겠다”고 했지만, CTO들과 플랫폼 아키텍트들에게는 이것이 기업용 AI 인프라의 가장 중요하고도 정의되지 않은 레이어를 스케치한 참조 아키텍처입니다.

VentureBeat은 이것을 “기업 애플리케이션과 변동성 높은 AI 모델 시장 사이에 위치한 오케스트레이션 미들웨어”라고 표현합니다. 수백 줄의 Python과 JavaScript로 카파시는 현대 소프트웨어 스택에서 가장 중요하면서도 아직 정의되지 않은 레이어를 스케치했다는 것입니다.

핵심 통찰은 이것입니다. 모델 자체는 상품화(commoditization)되고 있습니다. GPT, Claude, Gemini를 config 파일 한 줄만 바꿔서 교체할 수 있습니다. Meta나 Mistral의 새 모델이 다음 주에 리더보드를 장악하면 몇 초 만에 council에 추가할 수 있습니다. 진짜 가치는 모델을 라우팅하고 집계하는 로직에 있으며, 이것은 “놀라울 정도로 단순”합니다. 하지만 이것을 엔터프라이즈급으로 만드는 운영 래퍼가 진짜 복잡성이 존재하는 곳입니다.

## **LLM 앙상블과 멀티모델 오케스트레이션의 부상**

카파시의 프로젝트는 더 넓은 트렌드의 일부입니다. 2025년 연구들은 LLM 앙상블과 멀티모델 오케스트레이션이 단일 모델보다 일관되게 더 나은 결과를 낸다는 것을 보여줍니다. 기업 구현 연구에 따르면 앙상블 접근법은 단일 모델 솔루션보다 20-30%의 정확도 향상을 제공합니다.

JPMorgan Chase는 멀티모델 COiN 시스템으로 12,000개의 신용 계약을 몇 초 만에 처리합니다. Goldman Sachs는 AI 오케스트레이션 트레이딩 데스크가 인간 전용 데스크보다 27% 더 높은 수익성을 보고했습니다. 이것은 전문화(specialization)가 중요하다는 증거입니다.

다양한 협업 모드가 연구되고 있습니다. 협력적(cooperative) 접근, 경쟁적/대립적(competitive/adversarial) 접근, 앙상블 기반 방식 등입니다. 카파시의 LLM Council은 이 세 가지를 모두 결합합니다. 각 모델은 독립적으로 답변을 생성하고(앙상블), 서로의 답변을 평가하며(경쟁적), Chairman이 최종 합의를 도출합니다(협력적).

최신 연구들은 라우팅 전략, 계층적 기법, 동적 모델 선택 등을 탐구하고 있습니다. Microsoft의 AutoGen, LangChain의 crewAI, 그리고 다양한 오케스트레이션 프레임워크들이 등장하고 있습니다. 하지만 카파시의 접근법이 특별한 이유는 그 단순성과 투명성입니다. 복잡한 프레임워크 없이 핵심 아이디어를 몇 백 줄로 구현했습니다.

## **디자인 공간의 미개척 영역**

카파시는 트윗에서 “LLM council의 데이터 플로우에는 전체 디자인 공간이 있다”고 언급하며 “LLM 앙상블의 구축은 과소 탐구되어 있다(under-explored)“고 지적합니다. 이것은 중요한 관찰입니다.

현재 구현은 단순한 3단계 파이프라인이지만, 가능성은 무한합니다. 모델들이 여러 라운드에 걸쳐 토론할 수도 있습니다. 특정 모델에게 더 높은 가중치를 줄 수도 있습니다. 작업 유형에 따라 동적으로 council 구성원을 선택할 수도 있습니다. 모델들이 스스로 메타학습을 통해 어떤 모델이 어떤 작업에 강한지 학습할 수도 있습니다.

VentureBeat이 지적한 것처럼, “모델은 상품이고, 오케스트레이션 로직이 방어 가능한 자산”입니다. 경쟁자들은 같은 LLM들에 접근할 수 있지만, 의사결정 트리, 오류 처리, 최적화 로직을 쉽게 복제할 수는 없습니다. Shell은 14,000개 이상의 중요 장비에 멀티모델 AI 시스템을 운영하는데, 가치는 오케스트레이션에 있지 개별 모델에 있지 않습니다.

## **결론: “스킬 이슈”의 실천적 해법**

LLM Council로 돌아가면, 이것은 카파시가 말한 “스킬 이슈”의 완벽한 구체화입니다. 그가 나열한 새로운 추상화 계층들 - 에이전트, 서브에이전트, 프롬프트, 컨텍스트, 메모리, 도구, 플러그인, MCP, LSP, 워크플로우 - 이 모든 것이 LLM Council에 녹아 있습니다.

하지만 가장 중요한 교훈은 이것입니다. 카파시조차도 모델들의 평가와 자신의 평가가 다르다는 것을 발견했습니다. GPT-5.1이 가장 통찰력 있다고 AI들은 말하지만, 카파시는 그것이 너무 장황하다고 느낍니다. 이것은 AI 시대에도, 아니 AI 시대이기 때문에, 인간의 판단력이 여전히 중요하다는 것을 보여줍니다.

10배 더 강력해지는 방법은 AI에게 모든 것을 맡기는 것이 아닙니다. 다양한 AI 모델들을 이해하고, 그들의 강점과 약점을 파악하며, 적절한 오케스트레이션을 설계하고, 최종적으로는 인간의 판단으로 검증하는 것입니다. 카파시는 이것을 주말 프로젝트로 보여줬습니다. 이제 나머지는 우리의 “스킬 이슈”입니다.

**작성 일자: 2025-12-27**

---

[https://x.com/karpathy/status/1992381094667411768](https://x.com/karpathy/status/1992381094667411768)

2025.11.23

As a fun Saturday vibe code project and following up on this tweet earlier, I hacked up an **llm-council** web app. It looks exactly like ChatGPT except each user query is 1) dispatched to multiple models on your council using OpenRouter, e.g. currently:

“openai/gpt-5.1”,

“google/gemini-3-pro-preview”,

“anthropic/claude-sonnet-4.5”,

“x-ai/grok-4”,

Then 2) all models get to see each other’s (anonymized) responses and they review and rank them, and then 3) a “Chairman LLM” gets all of that as context and produces the final response.

It’s interesting to see the results from multiple models side by side on the same query, and even more amusingly, to read through their evaluation and ranking of each other’s responses.

Quite often, the models are surprisingly willing to select another LLM’s response as superior to their own, making this an interesting model evaluation strategy more generally. For example, reading book chapters together with my LLM Council today, the models consistently praise GPT 5.1 as the best and most insightful model, and consistently select Claude as the worst model, with the other models floating in between. But I’m not 100% convinced this aligns with my own qualitative assessment. For example, qualitatively I find GPT 5.1 a little too wordy and sprawled and Gemini 3 a bit more condensed and processed. Claude is too terse in this domain.

That said, there’s probably a whole design space of the data flow of your LLM council. The construction of LLM ensembles seems under-explored.

I pushed the vibe coded app to


[https://github.com/karpathy/llm-council](https://github.com/karpathy/llm-council)

if others would like to play. ty nano banana pro for fun header image for the repo

[https://x.com/karpathy/status/1990577951671509438?s=20](https://x.com/karpathy/status/1990577951671509438?s=20)