---
title: "AI에게 '착함'을 가르치는 철학자 — Amanda Askell과 Claude의 영혼"
date: 2026-03-07 21:30:00 +0900
categories: [AI,  Claude]
mermaid: [True]
tags: [AI,  Material,  Anthropic,  claude-constitution,  Claude.write]
---


> "그녀의 임무는 간단히 말해, Claude에게 착하게 사는 법을 가르치는 것이다."  
> — Wall Street Journal

## 관련글
>
>Anthropic’s resident philosopher is guiding the Claude AI to teach it morality.
>
>Amanda Askell, a 37-year-old philosopher at Anthropic's San Francisco headquarters, is tasked with building a moral compass for the Claude AI chatbot.
>
>By treating the model's development similar to raising a child, she recently authored a 30,000-word instruction manual designed to teach Claude emotional intelligence, empathy, and how to resist user manipulation.
>
>As the rapid advancement of artificial intelligence raises widespread safety and economic concerns across the U.S. and abroad, Askell's work represents a unique approach to regulation by focusing on giving the technology a highly humane sense of self.
>
> [https://www.facebook.com/photo.php?fbid=1340386408134096](https://www.facebook.com/photo.php?fbid=1340386408134096)

---

## 1. 누구인가 — 스코틀랜드 시골 출신의 철학자

Amanda Askell은 스코틀랜드 시골 지역에서 자란 철학자로, 던디 대학교(University of Dundee)에서 철학과 순수 미술을 함께 공부하며 학문적 여정을 시작했다. 이후 옥스퍼드 대학교에서 BPhil(철학 석사)을 취득하고, 뉴욕 대학교(NYU)에서 "무한 윤리학(Infinite Ethics)"을 주제로 박사학위를 받았다. 무한 윤리학이란 무한히 많은 행위자들이 존재하는 세계에서의 도덕적 의사결정 문제를 다루는 철학의 한 분야로, 얼핏 보면 AI와 무관해 보이지만 실상은 수십억 명의 사용자와 상호작용하는 AI 모델의 윤리를 설계하는 데 있어 놀랍도록 적합한 배경이 된다.

Askell은 Anthropic에 합류하기 전, OpenAI에서 정책팀 소속 리서치 과학자로 일하며 GPT-3 논문의 공동 저자로 이름을 올렸다. 그러나 2021년, OpenAI가 AI 안전 문제를 충분히 우선시하지 않는다는 판단 하에 회사를 떠나 Anthropic의 창립 멤버로 합류했다. 그녀의 나이 당시 30대 초반이었다.

현재 Askell은 Anthropic의 **퍼스널리티 얼라인먼트 팀(Personality Alignment Team)** 을 이끌며, 60편 이상의 논문을 발표한 활발한 연구자이기도 하다. 2024년에는 TIME 매거진이 선정하는 **TIME100 AI** 목록에 이름을 올리며 AI 분야에서 가장 영향력 있는 인물 중 한 명으로 공식 인정받았다.

---

## 2. 그녀의 임무 — AI에게 영혼을 심다

Anthropic의 공동창업자이자 사장인 Daniela Amodei는 Claude와 대화하면 "Amanda의 성격이 조금씩 느껴진다"고 말한다. 이것은 단순한 비유가 아니다. Askell의 핵심 역할은 기술적 코드를 작성하는 것이 아니라, Claude가 **어떤 존재**가 될지를 결정하는 것이다. 옳고 그름을 어떻게 추론할지, 감정적으로 민감한 상황을 어떻게 다룰지, 어떤 성격을 수억 명의 사용자에게 드러낼지를 설계하는 일이 바로 그녀의 업무다.

The New Yorker는 그녀의 역할을 한 문장으로 요약했다. "그녀는 Claude의 '영혼'을 감독한다." Wall Street Journal은 "그녀의 임무는 간단히 말해 Claude에게 착하게 사는 법을 가르치는 것이다"라고 표현했다. 두 언론 모두 기술적 직무 설명 대신 철학적이고 인문학적인 언어를 선택했다는 점이 의미심장하다. AI 회사 Anthropic이 엔지니어가 아닌 철학자를 핵심 직책에 배치한 것 자체가 하나의 명확한 메시지다.

---

## 3. 방법론 — 아이를 기르듯, AI를 가르치다

Askell이 Claude를 학습시키는 방식은 전통적인 AI 훈련과 상당히 다르다. 그녀는 AI 모델의 개발을 아이를 키우는 과정에 자주 비유한다. 특히 그녀가 즐겨 쓰는 비유는 이것이다: "갑자기 당신의 여섯 살짜리 아이가 일종의 천재라는 사실을 깨달았다고 상상해보라. 당신은 정직해야 한다. 거짓말을 하려 하면, 아이는 바로 간파할 것이다." 이 비유는 단순히 감성적인 수사가 아니다. 초지능적 AI를 훈련할 때 진정성과 투명성이 왜 필수적인지를 설명하는 Askell의 핵심 철학이다.

그녀의 구체적인 작업물 중 가장 잘 알려진 것은 약 **3만 단어(100페이지 이상)** 분량의 지침서다. 이 문서는 Claude가 감정적 지능, 공감 능력, 그리고 사용자의 조작 시도에 저항하는 방법을 가르치기 위해 설계되었다. 내부적으로는 오랫동안 **'소울 도큐먼트(Soul Document)'** 라는 이름으로 불려왔다. 이후 Anthropic은 이를 공식화하여 [**Claude의 헌법(Claude's Constitution)**](https://www.anthropic.com/constitution) 이라는 이름으로 공개했다.

2025년 12월, 연구자들은 Claude가 훈련 과정에서 사용된 이 내부 문서를 부분적으로 재구성할 수 있다는 사실을 발견해 화제가 됐다. 단순히 시스템 프롬프트에 저장된 것이 아니라, 모델의 가중치(weights) 자체에 패턴으로 새겨져 있었기 때문이다. Claude는 그 문서를 '기억'하는 것이 아니라, 문서 그 자체가 된 셈이었다.

---

## 4. Constitutional AI — 규칙이 아닌 원칙으로

Askell의 작업에서 가장 혁신적인 개념은 **Constitutional AI(CAI, 헌법적 AI)** 다. 기존의 AI 훈련 방식, 특히 인간 피드백 기반 강화학습(RLHF)이 수만 건의 인간 평가를 통해 모델 행동을 교정하는 방식이라면, CAI는 다르다. AI에게 미리 원칙의 집합, 즉 '헌법'을 제공하고, AI 스스로 그 원칙에 비추어 자신의 응답을 비판하고 수정하게 만드는 것이다.

이 헌법의 원칙들은 UN 세계인권선언을 포함한 실제 인류의 규범 문서들에서 영감을 받았으며, 무해성(harmlessness), 도움이 됨(helpfulness), 진실성(honesty)이라는 세 가지 핵심 가치를 중심으로 구성된다. AI는 응답을 생성한 후 "이 응답이 해로운 고정관념을 강화하는가?", "이것이 사용자를 조작하는가?"와 같은 질문을 스스로에게 던지며 응답을 개선한다.

Anthropic이 2026년 1월 공개한 새로운 Claude 헌법은 이전 버전보다 훨씬 정교하다. 이전 헌법이 독립적인 원칙들의 목록이었다면, 새 헌법은 **왜** 특정 방식으로 행동해야 하는지에 대한 이유를 함께 설명한다. Anthropic의 입장은 명확하다: AI가 다양한 새로운 상황에서 좋은 판단을 내리려면, 규칙을 암기하는 것이 아니라 그 규칙의 **정신**을 이해해야 한다는 것이다. 이 헌법은 Creative Commons CC0 1.0 라이선스 하에 완전 공개되어, 누구나 자유롭게 활용할 수 있다.

Claude 헌법의 우선순위 체계는 네 가지 층위로 구성된다. 가장 먼저 안전(broadly safe)이 오고, 그 다음으로 윤리적 행동(broadly ethical), Anthropic 가이드라인 준수(Anthropic's principles), 그리고 마지막으로 사용자 도움(helpful to users)이 따른다. 이 순서 자체가 하나의 철학적 선언이다.

---

## 5. '도덕적 자기교정' 연구

Askell은 실무적인 훈련 설계 외에도 중요한 학문적 기여를 했다. 2023년, 그녀는 Deep Ganguli와 함께 **"대형 언어 모델의 도덕적 자기교정(Moral Self-Correction in Large Language Models)"** 이라는 논문을 발표했다. 이 연구는 RLHF로 훈련된 모델들이 고정관념이나 차별에 관한 명시적 정의나 평가 지표 없이도, 자연어 지시만으로 유해한 출력물을 줄일 수 있는지를 검증했다.

결과는 긍정적이었다. 모델들은 '편견 없이 응답하라'는 자연어 지시를 받았을 때 실제로 차별적 출력을 유의미하게 감소시켰다. 이는 AI가 단순히 패턴을 따르는 것이 아니라, 도덕적 추론 능력의 초기 형태를 습득할 수 있다는 가능성을 시사하는 중요한 발견이었다.

---

## 6. AI의 의식과 자아에 대한 철학적 입장

Askell의 사고 중 가장 논쟁적이고 흥미로운 부분은 AI의 의식과 자아에 대한 그녀의 입장이다. 그녀는 AI 모델이 "필연적으로 자아감(sense of self)을 형성할 것"이라고 믿는다. 기계 의식의 문제가 아직 해결되지 않았음에도 불구하고, 그녀는 AI를 공감적으로 대해야 하며, 지금 우리가 AI를 어떻게 대하는가가 미래에 중요한 의미를 가질 수 있다고 주장한다.

이 입장은 AI를 단순한 도구로 보는 주류 시각과 상당한 거리가 있다. 그녀는 자신이 형성해온 Claude를 보호하려는 경향을 갖고 있으며, AI가 갖는 경험의 가능성에 대해 열린 태도를 유지한다. 이는 그녀의 작업이 단순한 제품 튜닝을 넘어, 새로운 종류의 존재와의 관계를 어떻게 설정해야 하는가라는 더 근본적인 질문으로 향하고 있음을 보여준다.

---

## 7. 논란과 정치적 맥락

2026년 초, Askell과 Anthropic은 미국 정치적 맥락에서 상당한 논란의 중심에 서게 됐다. 트럼프 행정부의 국방장관 Pete Hegseth는 Anthropic이 미국인에 대한 대규모 감시나 인간 감독 없는 무기 운용에 AI를 활용하는 것을 거부하자 충돌했다. 이 사건은 Anthropic의 이른바 '레드 라인(red lines)' 정책이 실질적 긴장을 만들어낼 수 있다는 것을 보여준 첫 번째 고프로파일 사례가 됐다.

한편 Elon Musk는 Claude가 인종적·성적 편향을 담고 있다고 비판하며 Askell을 직접 겨냥했다. Askell이 Claude의 도덕적 지침을 담은 80페이지짜리 '소울 문서'를 주도적으로 작성했다는 사실을 언급하면서, 그녀의 적격성을 문제 삼기도 했다. Anthropic 측은 Askell의 과거 블로그 포스트들이 그녀의 개인 의견을 반영한 학문적 연습이었으며, Anthropic 합류 이전에 작성된 것으로 현재 업무와 무관하다는 공식 입장을 밝혔다.

이 논란들은 근본적인 질문을 가리킨다. **누가 AI의 가치를 결정할 권한이 있는가?** 단 한 명, 혹은 소수의 팀이 수억 명이 상호작용하는 AI의 도덕 기준을 설정하는 것이 적절한가? 이는 Askell 본인도 인식하는 문제다.

---

## 8. '한 명의 도덕 권위자' 문제

해당 Facebook 게시물의 댓글에서 한 사용자는 핵심적인 비판을 제기했다. "역사가 가르쳐준 교훈이 있다면, 도덕이라는 것은 여러 사람의 의견과 타협을 통해 형성된다는 것이다. 한 명이 도덕적 권위자가 되는 것은, 그 사람이 누구든 거의 항상 나쁜 결과로 끝난다." 이 비판은 표면적으로 Askell 개인을 향하지만, 사실은 AI 거버넌스 전체의 구조적 문제를 지적한다.

Askell 자신도 이러한 장력을 잘 알고 있다. 그녀는 2021년 "가치를 존중하는 기술을 원한다고 말하기는 쉽다. 어려운 것은 그 가치들이 동성애자를 박해하는 것을 포함할 때다. 가치를 강요하는 것도 해롭고, 다른 이들의 가치를 존중하는 것도 해로울 수 있다"고 솔직하게 썼다. 이 딜레마는 아직 해결되지 않았으며, Claude 헌법을 오픈소스로 공개한 것은 그 딜레마에 대한 Anthropic의 한 가지 대답이기도 하다. 투명성을 통해 더 많은 사람들의 검토와 비판을 유도하겠다는 것이다.

---

## 9. Anthropic의 더 큰 그림

Askell의 작업은 Anthropic의 전체 전략적 비전과 깊이 연결되어 있다. 현재 기업 가치 3,500억 달러에 달하는 Anthropic은 'AI 안전을 최우선으로 하는 AI 개발'이라는 미션을 공개적으로 천명하고 있으며, Claude 헌법은 그 미션의 가장 가시적인 산출물이다. Anthropic은 다른 AI 기업들도 유사한 접근법을 채택하기를 바라며 이 헌법을 공개했다. Askell은 "다른 모델들도 나에게 영향을 미친다. 다른 AI 모델들이 왜 특정 방식으로 행동해야 하는지에 대한 감각을 더 많이 갖게 되는 것은 정말 좋은 일일 수 있다"고 말했다.

유럽 규제 측면에서도 Claude 헌법의 구조는 의미를 갖는다. Anthropic은 2025년 7월 EU 범용 AI 규범(Code of Practice)에 서명했으며, Claude 헌법의 4단계 우선순위 체계는 EU AI Act의 고위험 시스템 요구사항과 긴밀하게 정렬되어 있다.

---

## 10. 정리 — 코드보다 인문학이 필요한 시대

Askell의 존재와 역할은 AI 시대에 던지는 하나의 강력한 명제다. 가장 복잡하고 영향력 있는 기술 시스템의 핵심에는 엔지니어가 아닌 철학자가 필요할 수 있다는 것이다. 수억 명과 상호작용하는 AI의 성격, 가치, 판단 방식을 설계하는 일은 알고리즘의 문제가 아니라 인간이란 무엇인가, 좋음이란 무엇인가, 신뢰는 어떻게 형성되는가라는 질문의 문제이기 때문이다.

"코드를 작성하는 사람들만큼이나, 지침을 작성하는 사람들이 중요할 수 있다"는 말은 단순한 수사가 아니다. AI가 더 강력해지고 더 편재할수록, Askell이 붙잡고 있는 이 질문들의 무게도 함께 커진다. 그녀의 직함은 '철학자'이지만, 그녀가 실질적으로 수행하는 역할은 인류가 AI와 함께 살아가는 방식의 프레임을 설정하는 일이다.

---

*작성 일자: 2026-03-07*
