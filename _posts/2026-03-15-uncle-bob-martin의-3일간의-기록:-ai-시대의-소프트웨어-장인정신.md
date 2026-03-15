---
title: "Uncle Bob Martin의 \"3일간의 기록\": AI 시대의 소프트웨어 장인정신"
date: 2026-03-15 10:00:00 +0900
categories: [AI,  Vibe Coding]
mermaid: [True]
tags: [AI,  Uncle-Bob-Martin,  TDD,  ATDD,  Codex,  software-craftsmanship,  Claude.write]
---


> *"And that, boys and girls, is a freaking miracle."*
> — Robert C. Martin ([@unclebobmartin](https://x.com/unclebobmartin/status/2032562619849101804)), 2026

---

## 1. 들어가며: 이 트윗은 무엇인가

소프트웨어 업계에서 "클린 코드(Clean Code)"의 아버지로 불리는 Robert C. Martin, 일명 **"Uncle Bob"** 이 자신의 X(구 트위터) 계정에 남긴 이 글은 단순한 자랑이 아니다. 이것은 AI 코딩 도구가 범람하는 시대에, 수십 년간 갈고 닦은 소프트웨어 장인정신이 어떤 결과를 만들어내는지를 보여주는 생생한 실험 보고서이다.

Uncle Bob은 *단 3일* 동안 다섯 가지 대형 프로젝트를 완성했으며, 그 모든 과정에서 엄격한 품질 관리 워크플로우를 유지했다. 그리고 믿기 힘든 결과를 얻었다. 완성된 프로그램들이 **처음 실행했을 때 바로 동작했다(worked first time)** 는 것이다. 그는 이것을 "freaking miracle(기적)"이라고 불렀다.

이 문서는 그 트윗의 내용을 항목별로 깊이 분석하고, 각 기술 개념의 의미를 해설한다.

---

## 2. 저자 소개: Uncle Bob은 누구인가

**Robert Cecil Martin** (1952년 12월 5일 생)은 미국의 소프트웨어 엔지니어, 저자, 강연자이다. 그는 소프트웨어 개발 역사에서 가장 영향력 있는 인물 중 하나로 꼽힌다.

### 주요 업적

- **애자일 선언문(Agile Manifesto)의 공동 서명자** (2001년)
- **SOLID 원칙**의 제안자 — 객체지향 설계의 5가지 핵심 원칙
- **Clean Code**, **Clean Architecture**, **Clean Agile** 등 수십 권의 저서 출간
- Uncle Bob Consulting LLC 설립자 및 cleancoders.com 공동 설립자
- C++ Report 편집장 역임, Agile Alliance 초대 의장

그는 특히 Clojure 언어를 매우 선호하며, "JVM 위에서 동작하는 Lisp 방언"인 Clojure를 자신의 주 언어로 사용한다. 오랫동안 Java 환경에서 작업해온 그는 JVM 생태계에 깊은 조예를 가지고 있다.

---

## 3. 3일간 완성한 다섯 가지 프로젝트

### 3-1. AIR-J: AI를 위한 JVM 언어 설계 및 구현

Uncle Bob은 **AIR-J**라는 이름의 완전한 JVM 언어를 설계하고 구현했다. GitHub의 저장소 설명에는 다음과 같이 적혀 있다:

> *"A simple language for AIs to use and humans to ignore. This version is JVM based."*
> (AI가 사용하고 인간이 무시해도 되는 단순한 언어. 이 버전은 JVM 기반이다.)

이 언어는 JVM 바이트코드(bytecode)로 컴파일된다. 즉, Java Virtual Machine 위에서 실행된다. Uncle Bob은 OpenAI의 **Codex**가 이 언어를 "AI가 사용하기 이상적"이라고 평가했다고 언급했다 — 물론 그 "believe"가 정확히 무엇을 의미하는지에 대한 의구심을 괄호 안에 표현하면서.

**왜 이것이 흥미로운가?**

현재 대부분의 AI 코딩 도구는 인간이 읽기 위해 설계된 언어(Python, JavaScript, Java 등)로 코드를 생성한다. AIR-J는 인간의 가독성보다 AI의 생성 및 처리 효율성에 최적화된 언어를 목표로 한다는 점에서 실험적인 아이디어를 구현한 것이다.

그는 이 언어를 완성할 때까지 단 한 번도 컴파일하지 않았고("I never compiled a program with AIR-J until it was done"), 최초 컴파일 시 바로 동작했다고 밝혔다.

---

### 3-2. 내부 웹서버를 가진 위키(Wiki) 시스템 구현

Uncle Bob은 자체 내부 웹서버를 포함한 위키 시스템을 **처음부터(from scratch)** 설계하고 구현했다. 이 시스템의 모든 동작은 **Gherkin 스타일의 인수 테스트(acceptance tests)** 로 완전히 기술되었다.

Gherkin은 **BDD(Behavior Driven Development, 행동 주도 개발)** 에서 사용하는 자연어에 가까운 테스트 기술 문법이다. 테스트를 다음과 같은 형태로 작성한다:

```gherkin
GIVEN [사전 조건].
WHEN [사용자/시스템의 행동].
THEN [관찰 가능한 결과].
```

이 위키 역시 완성 전까지 단 한 번도 실행하지 않았고("I never ran the wiki until it was done"), 최초 실행 시 바로 동작했다.

---

### 3-3. Empire 게임의 컴퓨터 전략 대폭 업데이트

Uncle Bob은 **empire-2025**라고 불리는 제국(Empire) 게임 프로젝트를 진행해왔다. 이번에는 컴퓨터 플레이어의 전략 로직을 대폭 개선했다. 그는 이 업데이트 과정에서 새로운 버그가 하나도 도입되지 않았다고 밝혔다("No bugs have been introduced into the Empire game, so far").

이것이 가능했던 이유는 뒤에서 설명할 엄격한 TDD 워크플로우 덕분이다.

---

### 3-4. crap4java와 mutate4java 도구 제작

Uncle Bob은 Java 프로젝트를 위한 두 가지 핵심 품질 도구를 직접 만들었다.

- **crap4java**: Java 코드의 CRAP 점수를 분석하는 도구
- **mutate4java**: Java 코드의 뮤테이션 테스팅을 수행하는 도구

이 도구들은 위에서 언급한 위키 프로젝트를 개발하는 데 직접 사용되었다. 그는 이 도구들을 GitHub에 업로드하면서 다음과 같이 언급했다:

> *"You should consider these tools as examples and templates. Point your AI at them and have it build a similar tool for you customized for your needs."*
> (이 도구들을 예제와 템플릿으로 생각하세요. AI에게 이것들을 보여주고 당신의 필요에 맞게 커스터마이징된 유사 도구를 만들게 하세요.)

---

### 3-5. 차분 뮤테이션(Differential Mutation) 전략 고안 및 구현

Uncle Bob이 이번에 발명한 가장 독창적인 아이디어는 **"차분 뮤테이션(differential mutation)"** 전략이다. 이것은 Clojure와 Java 양쪽의 뮤테이션 도구에 모두 적용되었다.

기존의 뮤테이션 테스팅은 코드 전체에 대해 매번 처음부터 모든 변이(mutation)를 생성하고 테스트를 실행한다. "차분" 방식은 변경된 부분에 대해서만 뮤테이션을 실행하는 증분(incremental) 접근법으로, 처리 속도를 크게 향상시킨다.

---

## 4. 핵심 워크플로우: TDD, ATDD, Crap, Mutate

Uncle Bob이 이 모든 프로젝트에서 공통으로 적용한 워크플로우는 네 가지 요소로 구성된다.

### 4-1. TDD (Test-Driven Development, 테스트 주도 개발)

TDD는 Uncle Bob이 수십 년간 전도해온 핵심 개발 방법론이다. 그 규칙은 단순하다:

1. 실패하는 단위 테스트가 없으면 프로덕션 코드를 한 줄도 작성하지 않는다.
2. 컴파일에 실패할 정도 이상의 테스트 코드는 작성하지 않는다.
3. 현재 실패하는 테스트를 통과시킬 만큼만의 프로덕션 코드를 작성한다.

이 세 가지 법칙은 테스트와 프로덕션 코드를 거의 줄 단위로 번갈아 작성하게 강제한다. 그 결과 코드가 항상 테스트에 의해 보호되는 상태를 유지하게 된다.

### 4-2. ATDD (Acceptance Test-Driven Development, 인수 테스트 주도 개발)

ATDD는 TDD의 외부 계층이다. 단위 테스트가 개별 함수나 모듈의 동작을 검증한다면, 인수 테스트는 시스템 전체의 행동을 사용자 관점에서 검증한다.

Uncle Bob의 접근 방식에서 ATDD는 다음과 같은 역할을 한다:

- AI가 코드를 작성할 때 "무엇을 만들어야 하는지"를 명확하게 정의하는 앵커(anchor) 역할
- 인수 테스트가 있어야만 AI가 맥락 없이 코드를 "마구 이어붙이는(willy-nilly plop code around)" 행동을 방지할 수 있다.
- 단위 테스트와 인수 테스트라는 두 가지 독립적인 테스트 흐름이 AI로 하여금 코드 구조에 대해 훨씬 깊이 생각하게 만든다.

Gherkin 스타일의 인수 테스트는 다음과 같은 원칙을 따른다:

- 도메인 언어로 기술되어야 하며, 클래스명, API 엔드포인트, 데이터베이스 테이블명 같은 구현 세부사항이 들어가면 안 된다.
- 시스템이 "어떻게(how)" 동작하는지가 아니라 "무엇을(what)" 하는지를 기술해야 한다.

### 4-3. CRAP 메트릭 (Change Risk Anti-Patterns)

**CRAP**은 "변경 위험 안티패턴(Change Risk Anti-Patterns)"의 약자로, 코드의 유지보수 위험도를 측정하는 복합 지표이다. 공식은 다음과 같다:

```
CRAP(m) = comp(m)² × (1 - cov(m)/100)³ + comp(m)
```

- **comp(m)** : 메서드 m의 순환 복잡도(Cyclomatic Complexity) — 코드 내의 분기(if, loop 등) 수로 결정됨
- **cov(m)** : 자동화된 테스트가 제공하는 코드 커버리지(%)

**해석 방법:**

| 순환 복잡도 | CRAP 점수 30 미만을 위해 필요한 커버리지 |
|---|---|
| 0 ~ 5 | 0% (테스트 없어도 무방) |
| 10 | 42% |
| 15 | 57% |
| 20 | 71% |
| 25 | 80% |
| 30 | 100% |
| 31 이상 | 어떤 테스트로도 CRAP 영역 탈출 불가 |

즉, 코드가 복잡할수록 더 많은 테스트가 필요하다. 코드가 단순하면 적은 테스트로도 충분하다. Uncle Bob은 이번 프로젝트들에서 **CRAP 점수를 8 미만**으로 유지했다고 밝혔다. 이는 극히 낮은 유지보수 위험을 의미한다.

CRAP 점수를 낮추는 두 가지 방법은 ① 테스트를 추가하거나, ② 코드를 단순하게 리팩터링하는 것이다. 가장 이상적인 것은 두 가지를 동시에 하는 것이며, 테스트를 먼저 작성해야 안전하게 리팩터링할 수 있다.

### 4-4. 뮤테이션 테스팅 (Mutation Testing)

**"테스팅은 버그의 존재를 보여줄 수 있지만, 버그의 부재를 보여줄 수는 없다."**
— Edsger Dijkstra

코드 커버리지가 100%라고 해서 테스트가 완벽한 것은 아니다. 테스트가 코드의 모든 줄을 *실행*하더라도, 그 테스트가 코드의 *의미론적 동작*을 제대로 검증하는지는 별개의 문제이다.

뮤테이션 테스팅은 이 문제를 해결하기 위한 기법이다. 작동 방식은 다음과 같다:

1. 도구가 프로덕션 코드에 작은 변이(mutation)를 주입한다. 예: `+`를 `-`로, `>`를 `>=`로, `return true`를 `return false`로 변경
2. 변이된 코드에 대해 기존 테스트 스위트를 실행한다.
3. 테스트가 실패하면 — 즉 변이가 "죽었으면(killed)" — 그 테스트는 해당 코드의 의미를 제대로 검증하고 있다는 의미이다.
4. 테스트가 여전히 통과하면 — 즉 변이가 "살아남았으면(survived)" — 그 코드 영역은 테스트로부터 보호받지 못하고 있다는 의미이다.

Uncle Bob은 모든 뮤테이션 사이트(mutation site)가 50개를 초과하는 파일은 분리(split)했다고 밝혔다. 이는 파일이 너무 많은 책임을 지는 것을 방지하기 위한 조치이다.

**결과적으로 Uncle Bob의 테스트 커버리지는 90% 중반대 이상을 기록했다.** 이는 단순한 커버리지 지표가 아니라, 뮤테이션 테스팅까지 통과한 *의미론적으로 안정적인(semantically stable)* 커버리지이다.

---

## 5. 물리적 실재: 불타는 노트북

Uncle Bob은 이 경험의 강도를 매우 생생하게 묘사했다.

> *"My poor laptop had all 16 (8 hyperthreaded) cores burning at 100%. The fan was raging the whole time. I was hopping from window to window overseeing the entire campaign. It was exhausting!"*

그의 노트북은 8코어(하이퍼스레딩으로 논리 코어 16개)를 가지고 있으며, 이 모든 코어가 100%로 가동되었다. 팬은 지속적으로 최고 속도로 돌아갔다. 그는 여러 창을 오가며 전체 작업을 감독했다. 그것은 "진이 빠지는(exhausting)" 경험이었다.

뮤테이션 테스팅은 매우 연산 집약적인 작업이다. 코드의 변이된 버전마다 전체 테스트 스위트를 실행해야 하기 때문에, 수백 수천 개의 변이가 있는 프로젝트에서는 엄청난 CPU 자원이 필요하다. Uncle Bob이 새로 개발한 "차분 뮤테이션" 전략은 바로 이 병목을 해소하기 위한 혁신이었다.

---

## 6. 핵심 질문: 이 워크플로우는 개발 속도를 늦추었는가?

Uncle Bob은 이 질문에 대해 솔직하게 대답한다.

> *"Did that workflow slow the process down? Probably. Probably a lot."*
> (이 워크플로우가 프로세스를 느리게 했는가? 아마도. 아마도 많이.)

그렇다. 속도를 늦추었다. 그러나 트레이드오프는 명확했다.

**잃은 것:** 단기적인 개발 속도

**얻은 것:**
- 모든 단위 테스트와 인수 테스트 통과 — **엄격한 의미론적 안정성(rigorous semantic stability)**
- 위키: 완성 후 최초 실행에서 정상 동작
- AIR-J: 완성 후 최초 컴파일에서 정상 동작
- Empire 게임: 업데이트 후 새로운 버그 없음

이것이 Uncle Bob이 "miracle(기적)"이라고 부른 이유다. 테스트 없이 빠르게 개발하면 나중에 디버깅에 훨씬 더 많은 시간을 쓰게 된다. 반면 처음부터 엄격한 품질을 유지하면, 결과물은 처음 실행에서 동작한다.

---

## 7. AI와의 협업: Uncle Bob의 접근 방식

이 트윗은 AI 도구를 *무시*하거나 *맹목적으로 신뢰*하지 않는 Uncle Bob의 균형 잡힌 시각을 보여준다.

그는 OpenAI Codex와 협업하여 AIR-J를 설계했고, Anthropic Claude를 Clojure/Speclj 개발 보조 도구로 사용한다. 그의 GitHub에는 "A Claude skill for Clojure and Speclj that keeps the parentheses and the spec structure sane"라는 설명의 저장소도 있다.

**그러나 그의 철학은 분명하다:** AI는 코드를 생성하는 도구이지만, 그 코드를 신뢰할 수 있게 만드는 것은 여전히 인간의 규율(discipline)이다. ATDD, TDD, CRAP, 뮤테이션 테스팅이라는 네 겹의 안전망이 없다면 AI가 생성한 코드는 겉보기엔 작동하는 것처럼 보이지만 의미론적으로 불안정할 수 있다.

인수 테스트는 AI에게 "무엇을 만들어야 하는가"를 명확하게 알려주는 앵커 역할을 한다. 이것 없이 AI는 구현 세부사항에 집착하거나, 요구사항을 잘못 이해한 채로 많은 코드를 생성할 수 있다.

---

## 8. 이 트윗이 소프트웨어 업계에 던지는 메시지

Uncle Bob의 이 경험은 AI 코딩 도구가 폭발적으로 성장하는 시대에 중요한 질문을 던진다.

**"AI가 코드를 빠르게 생성하는 시대에, 소프트웨어 장인정신은 더 중요해지는가, 덜 중요해지는가?"**

Uncle Bob의 대답은 명확하다. **더 중요해진다.** AI는 코드 생성 속도를 높이지만, 그 코드가 올바르게 동작하는지를 보장하는 것은 여전히 인간의 규율과 검증 도구에 달려있다.

오히려 AI가 코드를 빠르게 생성할수록, 그 코드의 품질을 보장하는 테스트와 메트릭의 역할이 더 커진다. Uncle Bob은 그것을 몸소 증명했다. 3일 만에 다섯 가지 프로젝트를 완성하면서도, 모든 것이 처음 실행에서 동작하는 "기적"을 만들어냈다.

그리고 그것은 기적이 아니라 — 수십 년에 걸쳐 정제된 규율의 자연스러운 결과였다.

---

## 9. 용어 정리

| 용어 | 설명 |
|---|---|
| **JVM** | Java Virtual Machine. Java 바이트코드를 실행하는 가상 머신 |
| **TDD** | Test-Driven Development. 테스트를 먼저 작성하고 코드를 작성하는 개발 방법론 |
| **ATDD** | Acceptance Test-Driven Development. 인수 테스트를 먼저 작성하는 개발 방법론 |
| **Gherkin** | BDD에서 사용하는 Given/When/Then 형태의 자연어 테스트 기술 문법 |
| **CRAP** | Change Risk Anti-Patterns. 순환 복잡도와 커버리지를 결합한 코드 위험도 지표 |
| **순환 복잡도** | 코드 내 분기(if, loop 등)의 수로 측정하는 복잡도 지표 |
| **뮤테이션 테스팅** | 코드에 작은 변이를 주입해 테스트가 그것을 잡아내는지 확인하는 기법 |
| **차분 뮤테이션** | 변경된 부분에 대해서만 뮤테이션을 실행하는 증분 방식 |
| **Codex** | OpenAI의 코드 생성 AI 모델 |
| **Clojure** | JVM 위에서 동작하는 Lisp 방언의 함수형 프로그래밍 언어 |
| **Speclj** | Clojure용 BDD 테스트 프레임워크 |
| **의미론적 안정성** | 코드의 동작이 예상대로 유지됨을 테스트로 보장하는 상태 |

---

*이 문서는 Robert C. Martin(@unclebobmartin)의 X 게시물(2026년 3월)을 기반으로 작성되었습니다.*
