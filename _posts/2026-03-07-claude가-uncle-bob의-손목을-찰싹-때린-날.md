---
title: "Claude가 Uncle Bob의 손목을 찰싹 때린 날"
date: 2026-03-07 23:00:00 +0900
categories: [AI,  Material]
mermaid: [True]
tags: [AI,  Uncle-Bob-Martin,  claude-code,  Codex,  YAGNI,  Claude.write]
---


> [*"Here's Claude slapping my wrists."*](https://x.com/unclebobmartin/status/2029659889216213102)
> — Robert C. Martin (@unclebobmartin), 2026-03-07

---

## 사건의 개요

Clean Code의 저자이자 클린 아키텍처의 창시자, Robert C. Martin — 개발자 커뮤니티의 Uncle Bob — 이 자신의 코드에 대해 Claude로부터 설계 지적을 받았다. 그리고 그것을 공개적으로 인정했다.

Claude가 남긴 피드백의 전문은 다음과 같다.

> *"Honest dispatcher. The data map says what it is: a lookup table. No multimethod ceremony pretending to be extensible when the unit types are closed. If you ever need real dispatch (say, modding support), you can add it then — but you won't."*

짧지만 밀도가 높은 문장이다. 하나씩 풀어보자.

---

## Claude의 지적: 무엇이 문제였나

Uncle Bob이 작성한 코드는 `dispatcher`라는 이름을 달고 있었다. 이름부터가 설계 의도를 암시한다 — 다양한 타입에 따라 다른 동작을 분기시키는, 확장 가능한 dispatch 시스템. 실제로 구현에는 **multimethod** 패턴이 사용되었다. 멀티메서드는 런타임에 인자의 타입에 따라 적절한 메서드를 동적으로 선택하는 다형성 메커니즘으로, 특히 Clojure 같은 함수형 언어에서 강력한 확장 포인트로 활용된다.

그런데 Claude의 눈에 포착된 것은 바로 이 지점이었다. 이 코드에서 다루는 unit type들은 이미 **닫혀 있었다(closed)**. 런타임에 새로운 타입이 추가될 여지가 없는 상황이었다. 다시 말해, 코드가 실질적으로 하는 일은 그냥 특정 타입에 특정 값을 매핑하는 **조회 테이블(lookup table)** 이었다.

multimethod의 의식(ceremony) — 프로토콜 정의, 디스패치 함수, 개별 메서드 구현 — 은 이 맥락에서 아무런 실질적 역할을 하지 않는다. 확장 가능성을 위한 구조인데, 확장이 일어날 공간이 없는 것이다. Claude는 이것을 정확히 짚어냈다: "확장 가능한 척하는 multimethod 의식은 필요 없다."

---

## YAGNI: 오래된 원칙이 다시 소환된 순간

Claude의 피드백은 소프트웨어 공학의 오래된 원칙 하나를 정확히 적용한 것이다. **YAGNI — You Ain't Gonna Need It.** 지금 필요하지 않은 기능을 나중을 위해 미리 구현하지 말라는 원칙이다. XP(Extreme Programming)에서 비롯된 이 원칙은, 예방적 추상화가 실제로는 코드베이스에 불필요한 복잡성만 더한다는 경험적 교훈에서 출발한다.

Claude가 "If you ever need real dispatch, you can add it then — but you won't"이라고 쓴 것은 이 원칙의 교과서적 표현이다. 필요해지면 그때 추가하면 된다, 그리고 솔직히 그럴 일은 없을 것이다. 이 마지막 문장 — "but you won't" — 이 특히 인상적이다. 단순한 조언이 아니라 설계의 현실에 대한 솔직한 예측이다.

Uncle Bob 본인이 이 원칙을 모를 리 없다. 그는 수십 년간 이런 종류의 과잉 설계를 비판해온 인물이다. 그런 그가 자신의 코드에서 같은 함정에 빠졌다는 것 — Claude는 그것을 짚어냈고, Uncle Bob은 공개적으로 인정했다.

---

## "Honest Dispatcher": 이름이 말하는 것

Claude가 피드백 첫 줄에 쓴 표현이 흥미롭다. "Honest dispatcher." 코드가 실제로 하는 일에 솔직한 이름, 솔직한 구현. 조회 테이블이라면 조회 테이블답게 작성하라.

이것은 단순한 코드 스타일 이야기가 아니다. 코드의 이름과 구조가 그 코드가 **실제로 하는 일**을 정직하게 반영해야 한다는 원칙이다. `dispatcher`라는 이름 아래 multimethod 구조를 두는 것은, 이 코드가 확장 가능한 dispatch 시스템이라는 인식을 독자에게 심어준다. 그러나 실제로는 그렇지 않다. 이 불일치가 미래의 독자 — 혹은 미래의 자신 — 를 혼란스럽게 만든다.

Claude의 제안은 결국 이것이다: **코드는 자신이 실제로 무엇인지를 솔직하게 말해야 한다.** 데이터 맵은 데이터 맵처럼, 조회 테이블은 조회 테이블처럼.

---

## 아이러니: 같은 날의 두 사건

이 트윗이 의미심장한 이유 중 하나는 타이밍이다. 바로 같은 날, Uncle Bob은 Codex가 아키텍처 준수를 위해 코드를 "갈가리 찢었다"는 트윗을 남겼다. Codex는 규칙에 맹목적으로 복종하느라 과잉 재편을 저질렀다. 그리고 몇 시간 후, Uncle Bob 자신은 Claude에게 반대 방향의 과잉 — 필요하지 않은 추상화를 예방적으로 깔아두는 것 — 을 지적받았다.

두 사건을 나란히 놓으면 하나의 그림이 완성된다. AI가 인간의 의도를 지나치게 해석하여 구조를 폭력적으로 재편하는 것도 문제고, 인간 전문가가 설계 관성으로 인해 불필요한 추상화를 쌓는 것도 문제다. 그리고 Claude는 이 두 번째 문제를 정확히 포착했다.

---

## Uncle Bob이 공개적으로 인정한 것의 의미

"Here's Claude slapping my wrists"라는 표현에는 불쾌함이 없다. 오히려 경쾌한 인정의 뉘앙스가 담겨 있다. Clean Code를 쓴 사람이 자신의 코드에서 불필요한 추상화를 들키고, 그것을 공개적으로 트윗으로 공유한 것이다.

이것은 두 가지를 동시에 보여준다. 첫째, AI 코딩 도구가 전문가 수준의 설계 피드백을 줄 수 있는 단계에 이르렀다는 것. 둘째, 그 피드백이 가치 있으려면 받아들일 수 있는 개방성이 있어야 한다는 것. Uncle Bob이 보여준 것은 기술적 겸손함이다 — 수십 년의 경험을 가진 전문가도 AI의 피드백에서 배울 수 있다는 태도.

코드를 자아와 분리하라는 원칙 — Uncle Bob 자신이 오랫동안 강조해온 그 원칙 — 을 그가 몸소 실천한 순간이기도 하다.

---

*작성일: 2026-03-07*

*출처: Robert C. Martin (@unclebobmartin), Twitter/X, 2026-03-07*
