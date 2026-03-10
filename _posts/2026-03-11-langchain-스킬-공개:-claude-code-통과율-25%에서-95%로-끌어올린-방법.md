---
title: "LangChain 스킬 공개: Claude Code 통과율 25%에서 95%로 끌어올린 방법"
date: 2026-03-11 00:10:00 +0900
categories: [AI,  Platform & Framework]
mermaid: [True]
tags: [AI,  LangChain,  LangGraph,  LangSmith,  agent-skills,  CLI,  Claude.write]
---


> **출처**: LangChain Blog (2026-03-04 공개)  
> **원문**: [LangChain Skills](https://blog.langchain.com/langchain-skills/) | [LangSmith CLI & Skills](https://blog.langchain.com/langsmith-cli-skills/) | [Evaluating Skills](https://blog.langchain.com/evaluating-skills/)  
> **GitHub**: [langchain-skills](https://github.com/langchain-ai/langchain-skills) | [langsmith-skills](https://github.com/langchain-ai/langsmith-skills) | [skills-benchmarks](https://github.com/langchain-ai/skills-benchmarks)

---

## 1. 서론: 왜 이 발표가 중요한가

2026년 3월 4일, LangChain은 AI 코딩 에이전트 생태계에 의미 있는 변화를 가져올 발표를 했다. AI 코딩 에이전트를 위한 첫 번째 공식 스킬 세트(Skill Set)를 오픈소스로 공개한 것이다. 이 스킬들은 LangChain, LangGraph, LangSmith로 이루어진 자체 생태계에 특화되어 있으며, 그 성능 향상 수치가 특히 눈길을 끈다.

LangChain이 Claude Code(Anthropic의 CLI 기반 코딩 에이전트)를 대상으로 직접 벤치마크를 실시한 결과, 스킬 없이 LangChain 관련 태스크를 수행할 때의 통과율은 25%에 불과했다. 그런데 스킬을 장착했을 때는 이 수치가 95%로 뛰어올랐다. 스킬 없이는 네 번 중 한 번 정도만 올바른 결과를 내던 에이전트가, 스킬을 통해 스무 번 중 열아홉 번은 성공적으로 작업을 완료하는 수준이 된 것이다. LangSmith 관련 태스크에서는 더욱 극적인 변화가 있었는데, 통과율이 17%에서 92%로 상승했다.

이 수치만 놓고 보면 단순한 성능 개선처럼 보일 수 있지만, 그 배경에는 AI 에이전트가 전문적인 라이브러리나 도구를 사용하는 방식 자체에 대한 근본적인 재고가 담겨 있다. 코딩 에이전트는 아무리 강력한 언어 모델을 기반으로 하더라도 훈련 데이터의 한계, 라이브러리의 빠른 버전 업데이트, 그리고 복잡한 API 패턴 등으로 인해 특정 생태계에서 반복적으로 실패하는 경향이 있다. 스킬은 이 간극을 메우는 해법으로 등장한 것이다.

---

## 2. 스킬이란 무엇인가: 개념과 철학

### 2-1. 스킬의 정의

스킬(Skill)은 한마디로 정의하면, 코딩 에이전트의 성능을 특정 도메인에서 향상시키기 위한 **지침, 스크립트, 자료의 큐레이션된 묶음**이다. 스킬은 마크다운 파일과 셸 스크립트로 구성되어 있으며, 특정 에이전트에 종속되지 않고 어떤 코딩 에이전트에도 이식할 수 있도록 설계되어 있다.

스킬이 단순한 프롬프트나 문서와 근본적으로 다른 점은 그 로딩 방식에 있다. 스킬은 처음부터 모두 컨텍스트에 올라오는 것이 아니라, **에이전트가 관련 태스크를 처음 마주치는 순간에만 동적으로 불러와진다.** 이 방식을 LangChain은 '프로그레시브 디스클로저(Progressive Disclosure, 점진적 공개)'라고 부른다.

### 2-2. 프로그레시브 디스클로저가 필요한 이유

AI 에이전트에게 처음부터 모든 정보를 제공하는 것이 왜 좋지 않은가에 대한 답은 LangChain의 이전 연구에서 이미 제시된 바 있다. LangChain의 ReAct 에이전트 벤치마킹 연구에서, 에이전트에게 너무 많은 도구(tools)를 동시에 부여하면 오히려 성능이 저하된다는 사실이 밝혀졌다. 이는 언어 모델이 컨텍스트 내에서 처리해야 할 정보가 지나치게 많아질 때 정작 중요한 정보에 집중하지 못하고, 불필요한 도구나 정보에 의해 판단이 흐려지는 현상과 관련이 있다.

프로그레시브 디스클로저는 이 문제를 해결하는 우아한 방법이다. 에이전트는 자신이 현재 수행 중인 태스크와 관련된 스킬의 이름과 설명만 항상 알고 있고, 실제 스킬의 내용(지침과 스크립트)은 해당 스킬이 필요한 순간에만 검색하여 가져온다. 마치 도서관에서 특정 주제의 책을 찾아 읽는 것처럼, 에이전트는 필요할 때만 필요한 지식을 꺼내 쓰는 것이다.

비유하자면, 새로운 직원에게 첫날부터 회사의 모든 매뉴얼을 통째로 읽히는 것이 아니라, 업무 중 특정 상황에 맞닥뜨렸을 때 관련 매뉴얼 챕터를 꺼내 참고하게 하는 방식과 같다.

### 2-3. 스킬의 이식성과 생태계 표준

스킬은 단순한 텍스트 파일(마크다운)과 스크립트로 구성되어 있기 때문에, Claude Code뿐 아니라 Cursor, Windsurf, OpenAI Codex, Deep Agents CLI 등 스킬 사양(Agent Skills Specification)을 지원하는 어떤 코딩 에이전트에도 그대로 사용할 수 있다. 이것은 스킬이 특정 에이전트에 묶인 독점적 기술이 아니라, 에이전트 생태계 전반에서 통용되는 개방적 표준을 지향한다는 것을 의미한다.

스킬의 설치는 Vercel Labs의 `npx skills` 도구를 통해 이루어진다. 단 한 줄의 명령어로 LangChain의 전체 스킬 세트를 Claude Code에 연결할 수 있다.

```bash
# Claude Code에 LangChain 스킬 전체를 글로벌로 설치
npx skills add langchain-ai/langchain-skills --agent claude-code --skill '*' --yes --global

# LangSmith 스킬 설치
npx skills add langchain-ai/langsmith-skills --agent claude-code --skill '*' --yes --global
```

---

## 3. LangChain이 공개한 스킬 목록: 무엇이 포함되어 있나

이번 공개에서 LangChain은 크게 두 개의 리포지토리로 나뉘어 스킬을 공개했다. 하나는 오픈소스 라이브러리(LangChain, LangGraph, Deep Agents)를 위한 스킬이고, 다른 하나는 LangSmith 플랫폼을 위한 스킬이다.

### 3-1. LangChain 오픈소스 스킬 (11종)

`langchain-skills` 리포지토리에는 총 11개의 스킬이 포함되어 있으며, 세 가지 카테고리로 분류된다.

**LangChain 코어 카테고리**는 LangChain의 기본적인 에이전트 구성을 다룬다. `create_agent()` 함수의 올바른 사용 방법, 에이전트 루프에 커스텀 동작을 삽입하는 미들웨어(middleware) 패턴, 그리고 에이전트가 외부 세계와 상호작용하는 수단인 도구(tool) 패턴이 이 카테고리에 속한다. 특히 기본적인 도구 호출 에이전트 루프(tool calling agent loop)를 제대로 구현하는 방법에 대한 가이던스가 핵심이다.

**LangGraph 카테고리**는 LangGraph의 더 정교한 기능들을 다룬다. LangGraph의 기본 프리미티브인 `StateGraph`, 노드(node), 엣지(edge), 상태 리듀서(state reducer)를 사용하는 방법이 담겨 있으며, Human-in-the-Loop(인간이 중간에 개입하는 패턴)의 구현 방법, 내구성 있는 실행(durable execution)을 위한 체크포인터(checkpointer) 사용법, `thread_id`를 통한 대화 간 메모리 유지 방법 등이 포함된다. LangGraph는 에이전트 워크플로를 유향 그래프(directed graph)로 모델링하는 프레임워크로, 복잡한 다단계 에이전트 로직을 구현하는 데 필수적이다.

**Deep Agents 카테고리**는 LangChain이 최근 공개한 오픈소스 패키지인 Deep Agents를 다룬다. 이 패키지는 사전 구축된 미들웨어와 파일 시스템 도구를 제공하는데, 이를 활용해 더 강력한 에이전트를 빠르게 구성하는 방법이 이 카테고리의 주제다.

### 3-2. LangSmith 스킬 (3종)

`langsmith-skills` 리포지토리에는 3개의 핵심 스킬이 포함되어 있다.

첫 번째 스킬은 **트레이싱 추가(Adding Tracing)**다. 에이전트가 실행되는 동안 LangSmith로 트레이스를 전송하도록 코드에 계측(instrumentation)을 추가하는 방법을 다룬다. 트레이싱은 에이전트의 내부 동작을 가시화하는 핵심 기반으로, 이후 모든 디버깅과 최적화 작업의 출발점이 된다.

두 번째 스킬은 **데이터셋 구축(Building Datasets)**이다. 에이전트의 실행 트레이스를 기반으로 평가용 데이터셋을 만드는 방법을 다룬다. 실제 실행에서 수집된 예시들로 데이터셋을 구성함으로써, 지속적인 회귀 테스트(regression test)가 가능해진다.

세 번째 스킬은 **에이전트 평가(Agent Evaluation)**다. 구축된 데이터셋을 사용해 에이전트의 성능을 체계적으로 측정하는 방법을 다룬다. 평가자(evaluator)를 만들고, 실험을 실행하고, 결과를 비교하는 전 과정이 포함된다.

### 3-3. LangSmith CLI: 에이전트 네이티브 도구

LangSmith 스킬과 함께 출시된 **LangSmith CLI**는 별도로 주목할 필요가 있다. 이 CLI는 단순히 개발자를 위한 커맨드라인 도구가 아니라, 처음부터 AI 에이전트가 사용하도록 설계된 '에이전트 네이티브(agent-native)' 도구다.

LangSmith CLI를 통해 코딩 에이전트는 터미널만으로 다음 작업을 모두 수행할 수 있다: LangSmith에서 트레이스 가져오기(fetch traces), 데이터셋 큐레이션(curate datasets), 실험 실행(run experiments). 이것은 에이전트가 웹 브라우저나 별도의 GUI 없이, 순수하게 터미널 기반으로 LangSmith의 전체 기능에 접근할 수 있다는 것을 의미한다. 이전에 출시된 LangSmith Fetch(2025년 12월)가 트레이스 접근 기능을 터미널로 가져온 것에서 한 발 더 나아간 완전한 CLI 통합이다.

LangChain은 이 CLI가 향후 에이전트 개선 루프가 "터미널 기반의 다른 에이전트에 의해 주도될 것"이라는 전망 아래 설계되었다고 밝혔다.

---

## 4. 성능 수치의 의미: 벤치마크 분석

### 4-1. 공식 발표된 수치

LangChain이 발표한 성능 향상 수치를 정리하면 다음과 같다.

| 구분 | 모델 | 스킬 없음 | 스킬 있음 |
|------|------|-----------|-----------|
| LangChain/LangGraph 태스크 | Claude Code (Sonnet 4.6) | 25% | 95% |
| LangSmith 태스크 | Claude Code (Sonnet 4.6) | 17% | 92% |

별도로 공개된 평가 방법론 블로그 포스트에서는 더 넓은 시각의 수치도 제시되었다: 스킬을 포함했을 때 전반적인 태스크 완료율은 82%였으며, 스킬 없이는 9%에 그쳤다.

### 4-2. 수치 해석 시 주의할 점

이 수치들은 인상적이지만, 맥락을 이해하는 것이 중요하다. 먼저 이 벤치마크는 LangChain 생태계에 특화된 태스크만을 측정한 것이다. 범용적인 프로그래밍 능력이나 다른 라이브러리에서의 성능은 이 수치로 평가할 수 없다. 다시 말해, 이 결과는 "전문 도메인에서 에이전트가 얼마나 도움을 받을 수 있는가"를 보여주는 것이지, 에이전트의 전반적인 능력 지표가 아니다.

또한 LangChain은 벤치마크 리포지토리를 오픈소스로 공개할 계획을 밝혔으며, 이를 통해 커뮤니티가 수치를 독립적으로 검증할 수 있을 것이다.

---

## 5. 스킬 평가 방법론: 어떻게 검증하는가

이번 발표에서 LangChain이 단순히 스킬 코드를 공개하는 데 그치지 않고, 스킬을 **어떻게 만들고 평가하는지**에 대한 방법론까지 함께 공개한 것은 매우 중요한 의미를 갖는다. 스킬은 일종의 프롬프트이기 때문에, 의도치 않은 방식으로 에이전트 동작에 영향을 미칠 수 있다. 따라서 스킬의 효과를 체계적으로 검증하지 않으면 오히려 성능 저하를 초래할 수도 있다.

### 5-1. 기본 평가 파이프라인

LangChain이 제안하는 스킬 평가의 기본 흐름은 다음과 같다.

먼저 **태스크를 정의하는 단계**가 있다. 에이전트가 성공적으로 완료해야 하는 태스크를 명확하게 정의한다. 이때 너무 개방형(open-ended)으로 태스크를 정의하면 결과를 평가하기 어려워진다. LangChain은 버그가 있는 코드를 에이전트에게 고치게 하는 방식이 효과적이었다고 소개한다. 버그가 수정되었는지는 사전에 정의된 테스트로 명확하게 검증할 수 있기 때문이다. 또한, 이 방식은 에이전트의 설계 자유도를 제한함으로써 평가의 일관성을 높인다.

한편, 태스크를 너무 어렵게 만드는 것도 피해야 한다. 태스크가 지나치게 복잡하거나 적대적(adversarial)이면, 스킬의 효과를 측정하는 것이 아니라 에이전트 자체의 문제 해결 능력을 측정하게 된다. 태스크는 실제로 관찰된 에이전트의 실패 사례를 기반으로 만드는 것이 바람직하다.

다음은 **일관된 테스트 환경 구성**이다. 코딩 에이전트는 초기 상태에 매우 민감하다. Claude Code는 작업을 시작하기 전에 먼저 디렉토리를 탐색하는 습성이 있는데, 탐색 중 어떤 파일을 발견하느냐에 따라 이후 접근 방식이 달라진다. 이 때문에 스킬 평가 시에는 반드시 격리되고 일관된 환경에서 테스트해야 한다. LangChain은 Docker 컨테이너 안에서 Claude Code를 실행하는 방식으로 이 문제를 해결했다. Harbor나 기타 샌드박스 도구도 대안이 될 수 있다.

**측정 지표를 명확히 하는 것**도 중요하다. LangChain이 실제로 추적한 지표는 여러 가지다. 스킬이 올바르게 호출되었는가(invoked), 에이전트가 태스크를 완료했는가, 태스크 완료에 몇 번의 턴(turn)이 소요되었는가, 그리고 실제 소요 시간은 얼마나 되는가를 모두 기록했다. 이 지표들은 단순한 성공/실패 이분법을 넘어, 스킬이 효율성에도 기여하는지를 파악하는 데 도움이 된다.

### 5-2. 평가 과정에서 발견된 흥미로운 인사이트

스킬 평가를 진행하면서 LangChain이 발견한 몇 가지 흥미로운 사실들이 있다.

**스킬이 항상 올바르게 호출되지는 않는다.** LangChain 에이전트를 생성하는 특정 태스크에서, Claude Code가 관련 스킬("langchain agents")을 전혀 호출하지 않는 경우가 발생했다. 프롬프트에 스킬을 호출하라는 안내를 추가해도 호출률이 70% 수준에 그쳤다. 이 문제는 `AGENTS.md`나 `CLAUDE.md` 파일에 "언제 어떤 스킬을 사용하라"는 명시적 안내를 추가함으로써 해결할 수 있었다. 이 파일들은 항상 컨텍스트에 로드되기 때문에, 일관된 스킬 호출을 보장하는 앵커 역할을 한다.

**유사한 스킬의 수가 많으면 오호출(mis-invocation)이 발생한다.** 유사한 LangGraph 스킬이 약 20개였을 때, Claude Code는 가끔 잘못된 스킬을 호출했다. 스킬 수를 12개로 줄이자 일관되게 올바른 스킬을 호출하게 되었다. 이것은 스킬 설계 시 스킬의 수와 유사도 간의 균형을 신중하게 고려해야 한다는 것을 시사한다. 다만 스킬 수를 줄이면 각 스킬이 더 많은 내용을 담게 되고, 불필요한 내용까지 컨텍스트에 올라오는 부작용이 있다. 최적의 균형점을 찾기 위해서는 반복적인 테스트가 필요하다.

**스킬 내용의 세밀한 포맷팅 변경은 대형 스킬에서 큰 영향이 없다.** 300~500줄 규모의 큰 스킬에서는 긍정형 지시("이렇게 해라") vs 부정형 지시("이렇게 하지 마라"), 마크다운 vs XML 태그와 같은 포맷팅 차이가 측정 가능한 성능 차이로 이어지지 않았다. 대신, 스킬을 섹션 단위로 구분하고 섹션 수준에서 A/B 테스트를 진행하는 것이 더 효과적이었다.

**LangSmith로 에이전트 자신의 트레이스를 분석하게 한다.** 평가 과정에서 LangChain이 활용한 특히 흥미로운 방법은, Claude Code가 자신의 실행 트레이스를 LangSmith에 전송한 뒤 그 트레이스를 직접 분석하고 무엇이 잘못되었는지 요약하게 하는 것이었다. 에이전트가 스스로의 실패 원인을 진단하는 이 방식은 스킬 개선 사이클을 대폭 가속화했다. 인간 개발자는 에이전트의 요약을 검토하고 개선 사항을 제안하면 되었으며, 그 결과를 LangSmith의 실험(experiment) 포털에서 이전 실행과 비교할 수 있었다.

---

## 6. 에이전트가 에이전트를 개선하는 루프: 미래 비전

이번 LangSmith 스킬 공개에서 가장 주목할 만한 개념은 **'선순환(virtuous cycle)'** 이다. LangChain은 LangSmith 스킬들이 단순히 LangSmith 사용 방법을 에이전트에게 가르치는 것이 아니라, 에이전트가 스스로 개선하는 자동화된 사이클을 가능하게 한다고 설명한다.

이 사이클은 다음과 같이 작동한다. 코딩 에이전트가 LangChain 스킬을 활용해 에이전트 코드를 작성하면서 LangSmith 트레이싱을 추가한다. 에이전트가 실행되면서 생성된 트레이스는 LangSmith로 자동 전송된다. 그 다음, 코딩 에이전트는 LangSmith CLI를 통해 그 트레이스를 가져와 분석하고, 발견된 문제점과 패턴을 요약한다. 이를 바탕으로 테스트 데이터셋을 구축하고, 이 데이터셋에 대한 평가자(evaluator)를 만든다. 평가 결과를 바탕으로 에이전트 코드를 개선하면 다시 새로운 트레이스가 생성되고, 이 전체 루프가 반복된다.

이 흐름에서 주목해야 할 것은, 루프의 주체가 인간이 아니라 에이전트 자신이라는 점이다. 인간 개발자는 이 루프를 설정하고 결과를 최종 검토하는 역할을 하지만, 루프 자체는 에이전트에 의해 자율적으로 진행된다. LangChain은 앞으로 에이전트 개선 루프가 점점 더 "터미널 기반의 다른 에이전트"에 의해 주도될 것으로 전망한다. LangSmith CLI와 스킬은 바로 이 미래를 위한 인프라다.

---

## 7. 에이전트 스킬 생태계의 맥락

### 7-1. 스킬이 등장한 배경

LangChain의 스킬 공개는 고립된 사건이 아니다. 이것은 2025년 하반기부터 본격화되기 시작한, AI 코딩 에이전트 생태계 전반의 스킬 표준화 움직임의 일환이다. Anthropic이 Claude Code를 위한 Agent Skills 사양을 도입하면서 촉발된 이 흐름은, 이제 LangChain처럼 주요 라이브러리 벤더들이 자체 스킬 세트를 공개하는 단계로 발전했다.

이 흐름의 핵심은 AI 에이전트가 특정 도구나 라이브러리를 효과적으로 사용하기 위해서는, 해당 도구의 제작자가 직접 그 사용법을 에이전트 친화적인 형태(스킬)로 패키징하여 제공해야 한다는 인식이다. 문서를 인터넷에 올려두는 것만으로는 부족하다. 에이전트가 필요한 순간에 올바른 방식으로 정보를 검색하고 적용할 수 있도록, 구조화되고 검증된 지침을 제공해야 한다.

### 7-2. AGENTS.md의 역할

스킬과 함께 반복적으로 언급되는 것이 `AGENTS.md`(또는 `CLAUDE.md`) 파일이다. 이 파일은 에이전트가 프로젝트의 구조와 작업 방식을 이해하는 데 도움을 주는 '에이전트 설정 파일'로, LangChain의 평가 결과는 스킬이 효과적으로 작동하기 위해서는 `AGENTS.md`에 적절한 스킬 호출 가이드를 포함시키는 것이 매우 중요하다는 것을 보여준다. 스킬 자체가 아무리 잘 만들어져 있어도, 에이전트가 그 스킬을 언제 어떻게 호출해야 하는지 모른다면 의미가 없기 때문이다.

### 7-3. 다른 에이전트와의 호환성

LangChain 스킬은 Claude Code에만 국한되지 않는다. Agent Skills 사양을 지원하는 Cursor, Windsurf 등 다른 코딩 에이전트에도 동일하게 설치하여 사용할 수 있다. 또한 LangChain의 자체 에이전트 실행 환경인 Deep Agents CLI에서도 사용 가능하다.

```bash
# Deep Agents CLI에 전역 설치 (에이전트 페르소나 포함)
./install.sh --deepagents --global
```

---

## 8. 실제 적용: 개발자가 알아야 할 것들

### 8-1. 설치 방법

LangChain 스킬의 설치는 두 가지 방법으로 할 수 있다.

첫 번째는 `npx skills` 명령어를 사용하는 방법이다:

```bash
# LangChain 스킬 (현재 프로젝트에만 적용)
npx skills add langchain-ai/langchain-skills --skill '*' --yes

# LangChain 스킬 (전역 적용)
npx skills add langchain-ai/langchain-skills --skill '*' --yes --global

# LangSmith 스킬 (전역 적용, Claude Code에 연결)
npx skills add langchain-ai/langsmith-skills --agent claude-code --skill '*' --yes --global
```

두 번째는 GitHub 리포지토리를 직접 클론하고 설치 스크립트를 실행하는 방법이다:

```bash
git clone https://github.com/langchain-ai/langchain-skills
cd langchain-skills

# Claude Code에 현재 디렉토리 기준으로 설치
./install.sh

# Claude Code에 전역으로 설치
./install.sh --global
```

환경 변수 설정도 필요하다:

```bash
export OPENAI_API_KEY=<your-key>      # OpenAI 모델 사용 시
export ANTHROPIC_API_KEY=<your-key>  # Anthropic 모델 사용 시
```

### 8-2. 스킬 목록 (langchain-skills 기준)

현재 공개된 LangChain 스킬 리포지토리에 포함된 스킬들은 다음과 같다:

- `langchain-middleware`: Human-in-the-loop 승인, 커스텀 미들웨어, Command 재개 패턴
- `langchain-rag`: RAG 파이프라인 (문서 로더, 임베딩, 벡터 스토어)
- `langgraph-fundamentals`: StateGraph, 노드, 엣지, 상태 리듀서
- `langgraph-persistence`: 체크포인터, thread_id, 크로스-스레드 메모리
- 그 외 LangChain/LangGraph 관련 스킬들 (총 11종)

### 8-3. 주의사항: 스킬이 항상 만능은 아니다

스킬이 극적인 성능 향상을 가져다주는 것은 사실이지만, 몇 가지 주의사항을 알고 사용하는 것이 중요하다.

스킬은 도메인 특화적 지식의 공백을 메워주는 역할을 하기 때문에, 에이전트가 이미 충분히 잘 알고 있는 영역이나 단순한 범용 프로그래밍 작업에서는 스킬이 큰 차이를 만들지 못할 수 있다. 스킬의 효과는 에이전트의 기존 지식과 태스크의 전문성 요구 사이의 간극이 클수록 더 크게 나타난다.

또한 앞서 언급한 대로 스킬 수가 너무 많아지면 오호출 문제가 생길 수 있다. 특히 유사한 스킬이 많을 경우에는 `AGENTS.md`에 명확한 호출 가이드를 제공하고, 스킬의 이름과 설명을 명확하게 차별화하는 것이 중요하다.

---

## 9. 요약과 전망

LangChain의 스킬 공개는 AI 코딩 에이전트가 전문적인 라이브러리와 도구를 효과적으로 사용하는 방식에 있어 중요한 전환점이다. 주요 내용을 정리하면 다음과 같다.

**스킬은 에이전트의 지식 공백을 동적으로 메운다.** 처음부터 모든 정보를 컨텍스트에 올리는 것이 아니라, 필요한 순간에만 관련 지식을 불러오는 프로그레시브 디스클로저 방식이 핵심이다. 이 방식은 에이전트의 성능 저하 없이 전문 도메인 지식을 효과적으로 제공한다.

**벤치마크 결과는 스킬의 실질적 효과를 수치로 입증했다.** Claude Code의 LangChain 관련 태스크 통과율이 25%에서 95%로, LangSmith 태스크는 17%에서 92%로 향상되었다. 이것은 "거의 작동하지 않던 것"이 "거의 항상 작동하는 것"으로 바뀐 것이다.

**스킬 평가는 필수다.** 스킬은 프롬프트와 유사한 특성을 가지기 때문에, 예상치 못한 방식으로 에이전트 동작에 영향을 줄 수 있다. Docker 기반의 격리된 환경에서 체계적인 A/B 테스트를 통해 스킬의 효과를 검증해야 한다.

**에이전트가 에이전트를 개선하는 루프가 가능해졌다.** LangSmith 스킬과 CLI를 활용하면, 코딩 에이전트가 자신의 실행을 트레이싱하고, 트레이스를 분석하고, 테스트 데이터셋을 만들고, 자신을 평가하는 자율적인 개선 사이클을 돌릴 수 있다.

**스킬 표준화는 더 넓은 생태계 흐름이다.** LangChain의 이번 공개는 Agent Skills 사양을 중심으로 형성되는 생태계 표준화의 한 사례다. 앞으로 더 많은 라이브러리와 플랫폼 벤더들이 자체 스킬 세트를 공개할 것으로 예상되며, 이는 AI 코딩 에이전트의 실용성을 전반적으로 끌어올리는 방향으로 작용할 것이다.

---

## 참고 자료

- [LangChain Skills – LangChain Blog](https://blog.langchain.com/langchain-skills/)
- [LangSmith CLI & Skills – LangChain Blog](https://blog.langchain.com/langsmith-cli-skills/)
- [Evaluating Skills – LangChain Blog](https://blog.langchain.com/evaluating-skills/)
- [LangChain Skills GitHub](https://github.com/langchain-ai/langchain-skills)
- [LangSmith Skills GitHub](https://github.com/langchain-ai/langsmith-skills)
- [Skills Benchmarks GitHub](https://github.com/langchain-ai/skills-benchmarks)
- [Vercel Labs npx skills](https://github.com/vercel-labs/skills)

---

*작성일: 2026-03-10*
