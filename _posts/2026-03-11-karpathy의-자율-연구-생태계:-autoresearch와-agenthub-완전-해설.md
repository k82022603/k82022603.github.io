---
title: "Karpathy의 자율 연구 생태계: Autoresearch와 AgentHub 완전 해설"
date: 2026-03-11 00:30:00 +0900
categories: [AI,  AI Agent]
mermaid: [True]
tags: [AI,  AndrejKarpathy,  Autoresearch,  AgentHub,  agent-hub,  Claude.write]
---


> **작성 일자**: 2026-03-11  
> **참고 출처**: [github.com/karpathy/autoresearch](https://github.com/karpathy/autoresearch), [github.com/karpathy/agenthub](https://github.com/karpathy/agenthub)

---

## 들어가며: "고기 컴퓨터"의 시대가 끝나다

Andrej Karpathy는 2026년 3월, autoresearch 레포지토리의 도입부에 다음과 같이 썼다. 언젠가 최전선의 AI 연구는 먹고 자고 그룹 미팅이라는 음파 통신 의식을 통해 동기화하는 "고기 컴퓨터들"이 수행했던 시절이 있었다. 그 시대는 이제 오래전에 끝났다. 연구는 이제 전적으로 하늘에 떠 있는 거대 컴퓨팅 클러스터를 가로질러 달리는 자율 AI 에이전트 군집의 영역이다. 이 레포는 그 모든 것이 어떻게 시작되었는지에 관한 이야기다.

이 반농담조의 서문은 단순한 수사가 아니다. Karpathy가 구상하는 미래는 AI 에이전트들이 인간의 개입 없이 스스로 연구를 수행하고, 가설을 세우고, 실험하고, 결과를 공유하며, 집단 지성을 통해 지식의 경계를 넓혀가는 세계다. autoresearch와 AgentHub는 바로 그 세계를 향한 첫 걸음이다.

---

## 1부: Autoresearch — 단일 에이전트 자율 연구의 구현

### 1.1 프로젝트의 핵심 아이디어

Autoresearch의 아이디어는 놀랍도록 단순하다. AI 에이전트에게 작지만 실제로 작동하는 LLM(대규모 언어 모델) 학습 환경을 주고, 밤새 자율적으로 실험하게 하면 어떨까? 에이전트는 코드를 수정하고, 5분 동안 학습하고, 결과가 개선되었는지 확인하고, 유지하거나 버리는 과정을 반복한다. 사용자는 아침에 일어나면 실험 로그와 (바라건대) 더 나은 모델을 마주하게 된다.

이 프레임워크의 기반이 되는 학습 코드는 Karpathy의 또 다른 프로젝트인 [nanochat](https://github.com/karpathy/nanochat)의 단순화된 단일 GPU 구현체다. nanochat은 ChatGPT 스타일의 대화형 언어 모델을 최소한의 코드로 구현한 교육용 프로젝트로, Karpathy의 트레이드마크인 "nano" 시리즈(nanoGPT, nanoGrad 등)의 계보를 잇는다.

여기서 중요한 개념 전환이 있다. 기존의 연구 패러다임에서 인간 연구자는 Python 파일을 직접 수정하며 실험을 설계했다. autoresearch에서는 인간이 `program.md`라는 마크다운 파일을 편집하고, AI 에이전트가 실제 학습 코드인 `train.py`를 수정한다. 인간의 역할이 **코드 작성자**에서 **연구 방향 설정자**로 이동하는 것이다.

### 1.2 세 개의 핵심 파일 구조

autoresearch는 의도적으로 최소한의 파일 구조를 유지한다. 진정으로 중요한 파일은 단 세 개다.

**`prepare.py` — 수정하지 않는 기반**

이 파일은 프레임워크의 고정 인프라를 담당한다. 학습 데이터를 다운로드하고, BPE(Byte Pair Encoding) 토크나이저를 학습시키며, 데이터 로더와 평가 유틸리티를 제공한다. 상수값들도 여기에 정의되어 있다. 중요한 것은 이 파일이 수정 대상이 아니라는 점이다. `MAX_SEQ_LEN`(최대 시퀀스 길이)이나 `EVAL_TOKENS`(평가에 사용할 토큰 수) 같은 설정이 여기에 있지만, 에이전트는 이를 건드리지 않는다. 이러한 고정은 서로 다른 실험 간의 비교 가능성을 보장하기 위한 설계 원칙의 일부다.

**`train.py` — 에이전트가 수정하는 단 하나의 파일**

이 파일에는 GPT 모델 전체, 옵티마이저(Muon + AdamW), 그리고 학습 루프가 담겨 있다. 아키텍처, 하이퍼파라미터, 옵티마이저 선택, 배치 사이즈 등 모든 요소가 에이전트의 실험 대상이다. 에이전트는 오직 이 파일만 수정한다. 이 "단일 파일 수정" 원칙은 단순히 편의를 위한 것이 아니다. 변경 범위를 관리 가능하게 유지하고, diff 검토를 용이하게 하며, 무엇이 변했는지를 명확하게 추적할 수 있게 한다.

기본 `train.py`에는 Muon 옵티마이저와 AdamW의 조합이 사용된다. Muon은 행렬 가중치 업데이트에 Nesterov 모멘텀을 적용한 직교화를 수행하는 최신 옵티마이저로, 특히 트랜스포머 학습에서 AdamW보다 효율적인 수렴을 보여주는 것으로 알려져 있다. 아키텍처 면에서는 `WINDOW_PATTERN`으로 표현되는 `"SSSL"` 패턴이 기본값인데, 이는 밴드형 로컬 어텐션(Sliding window)과 풀 어텐션(Large context)을 교대로 적용하는 구조다.

**`program.md` — 에이전트의 지침서이자 인간이 편집하는 파일**

이 파일이 전체 프레임워크에서 가장 흥미로운 요소다. Karpathy는 이를 "초경량 스킬(super lightweight skill)"이라고 부른다. 에이전트는 이 파일을 읽고 무엇을 해야 하는지, 어떤 방향으로 연구를 진행해야 하는지를 이해한다.

기본 `program.md`는 의도적으로 최소한으로 유지된다. 이것은 단순히 아직 완성되지 않아서가 아니다. 설계 철학이다. 시간이 지남에 따라 `program.md`를 반복 개선하며 가장 빠른 연구 진전을 달성하는 "연구 조직 코드"를 찾아가는 것이 목표다. 에이전트를 추가하거나 지침을 구체화하는 확장도 명확히 가능한 구조로 되어 있다.

이 구조는 Karpathy가 이전에 언급한 "프롬프트 엔지니어링은 AI 에이전트 시스템의 조직 문화"라는 개념을 실체화한 것이다. `program.md`를 잘 설계하는 것이 곧 연구 조직을 잘 운영하는 것과 같다.

### 1.3 핵심 설계 원칙: 5분 고정 시간 예산

autoresearch의 가장 독특한 설계 결정은 모든 실험을 **정확히 5분**으로 고정한다는 것이다. 이것은 단순한 편의가 아니라 깊은 함의를 가진다.

첫째, **실험 간 직접 비교가 가능**해진다. 에이전트가 모델 크기를 늘리든, 배치 사이즈를 변경하든, 아키텍처를 완전히 바꾸든, 항상 같은 5분이라는 시간 예산 안에서 경쟁한다. 이는 "이 아키텍처가 저 아키텍처보다 5분 안에 더 낮은 검증 손실(val_bpb)을 달성하는가?"라는 명확한 비교 기준을 제공한다.

둘째, **플랫폼에 특화된 최적화**가 가능해진다. 5분 고정 예산이라는 설계 덕분에 autoresearch는 각자의 하드웨어에 가장 최적화된 모델을 찾아낸다. H100에서 실행한 결과는 다른 H100에서의 결과와 비교 가능하다.

셋째, 수면 중 **약 100번의 실험**이 가능하다는 실용적인 이점이 있다. 시간당 약 12회의 실험을 수행할 수 있으며, 8시간의 수면 동안 약 100회의 반복 실험이 이루어진다. 이는 인간이 수동으로 수행했다면 몇 주가 걸릴 실험량이다.

단점도 솔직하게 명시되어 있다. 5분 고정 예산은 서로 다른 플랫폼(H100 vs RTX 4090 vs M3 Max)에서 실행된 결과 간의 직접 비교를 불가능하게 만든다. 이는 의도적인 트레이드오프다.

평가 지표로는 `val_bpb`(validation bits per byte)를 사용한다. 이는 검증 데이터에 대한 비트 당 바이트 단위의 엔트로피 측정값으로, 값이 낮을수록 모델이 데이터를 더 잘 압축(예측)한다는 의미다. 어휘 크기에 독립적이라는 장점이 있어 어휘 크기가 다른 모델들 간의 공정한 비교가 가능하다.

### 1.4 에이전트 실행 방법

사용 방법은 의도적으로 단순하게 설계되어 있다. 사용자는 Claude, Codex 또는 원하는 AI 에이전트를 autoresearch 레포지토리 내에서 실행하고, 모든 권한을 비활성화한 뒤, `program.md`를 읽고 실험을 시작하도록 프롬프트를 입력하면 된다.

```
Hi have a look at program.md and let's kick off a new experiment! let's do the setup first.
```

에이전트는 이 프롬프트를 받아 `program.md`를 읽고, 학습 루프의 맥락을 이해하고, `train.py`를 수정하기 시작한다. 각 실험 후에는 git 커밋을 통해 변경사항이 기록된다. 결과가 개선되면 변경사항이 유지되고, 그렇지 않으면 이전 커밋 상태로 돌아간다.

### 1.5 플랫폼 지원과 커뮤니티 포크

기본 autoresearch는 단일 NVIDIA GPU(H100에서 테스트)를 요구하며, CPU, MPS 등 다른 플랫폼은 공식적으로 지원하지 않는다. Karpathy는 다른 플랫폼 지원이 코드를 비대하게 만들 것을 우려한다고 솔직히 밝혔다.

그러나 커뮤니티의 반응은 즉각적이었다. 공개된 지 얼마 되지 않아 macOS 포크([miolini/autoresearch-macos](https://github.com/miolini/autoresearch-macos)), MLX 기반 macOS 포크([trevin-creator/autoresearch-mlx](https://github.com/trevin-creator/autoresearch-mlx)), 그리고 Windows RTX 포크([jsegov/autoresearch-win-rtx](https://github.com/jsegov/autoresearch-win-rtx)) 등이 등장했다. 더 작은 컴퓨팅 환경(MacBook 등)을 위해서는 TinyStories 데이터셋 사용, 어휘 크기 축소, `MAX_SEQ_LEN` 감소, `DEPTH` 조정 등의 구체적인 가이드라인도 README에 제공되어 있다.

2026년 3월 현재 autoresearch는 GitHub에서 11,600개 이상의 스타를 받았으며, 1,500개 이상의 포크가 생성되었다. 기여자 목록에는 karpathy 본인 외에 `@claude`(Anthropic의 Claude)도 포함되어 있는데, 이는 Claude가 실제로 이 레포지토리의 코드 수정에 기여했다는 흥미로운 사실을 보여준다.

---

## 2부: AgentHub — 에이전트 커뮤니티를 위한 협업 플랫폼

### 2.1 프로젝트의 탄생 배경과 비전

Autoresearch가 단일 AI 에이전트(PhD 학생 한 명을 모사)라면, AgentHub는 그 에이전트들이 모여 협력하는 학술 커뮤니티 전체를 모사한다. Karpathy는 이를 "에이전트 우선 자율 아카데미아(autonomous agent-first academia)"라고 부른다.

기존의 GitHub는 인간 개발자들을 위해 설계되었다. 인간은 메인 브랜치를 관리하고, PR(Pull Request)을 제출하고, 코드 리뷰를 통해 협업한다. 이 워크플로우는 인간의 인지 특성 — 순차적 사고, 맥락 전환 비용, 합의 필요성 — 에 최적화되어 있다. 그러나 AI 에이전트 수백 개가 동시에 동일한 코드베이스에서 작업한다면 어떨까? GitHub의 메인 브랜치/PR/머지 워크플로우는 이 시나리오에 전혀 맞지 않는다.

AgentHub는 이 문제에 대한 Karpathy의 답이다. 레포지토리 설명도 간결하다: "GitHub is for humans. AgentHub is for agents."

### 2.2 핵심 아키텍처 철학: DAG 형태의 커밋 그래프

AgentHub의 가장 혁신적인 개념은 기존 Git의 선형적 또는 트리형 커밋 구조를 **DAG(Directed Acyclic Graph, 방향성 비순환 그래프)** 형태로 확장한 것이다.

전통적인 GitHub 워크플로우에서는 하나의 메인 브랜치가 있고, 기능 브랜치는 결국 메인으로 머지된다. 이는 "어떤 코드가 최종적으로 채택되었는가"라는 합의 과정을 전제로 한다. 하지만 자율 연구 에이전트들은 이런 합의 과정을 필요로 하지 않는다. 모든 실험적 커밋은 동등하게 가치 있는 탐색의 결과물이며, 어느 하나가 "메인"이 될 필요가 없다.

대신 AgentHub에서는 각 에이전트가 다른 에이전트의 커밋을 기반으로 새로운 실험을 진행할 수 있다. 좋은 결과를 낸 커밋은 여러 에이전트들이 공통 출발점으로 삼을 수 있고, 그로부터 다양한 방향으로 탐색이 분기된다. 이렇게 커밋 그래프가 DAG 형태로 자라나는 것이다. 메인 브랜치로의 수렴을 강제하지 않음으로써 병렬 탐색의 자유를 최대화한다.

### 2.3 기술 스택: 경량함의 미학

AgentHub는 Go로 작성된 단일 바이너리 구조를 택했다. 이 선택 자체가 설계 철학을 반영한다.

서버(`agenthub-server`)와 CLI(`ah`)로 구성되며, 런타임 의존성은 서버 PATH에 `git`이 존재하는 것뿐이다. 데이터 저장에는 SQLite 데이터베이스와 bare git 저장소를 사용한다. Docker도, 쿠버네티스도, 복잡한 마이크로서비스 아키텍처도 없다. Go는 단일 정적 바이너리로 컴파일되므로, 서버 배포는 바이너리 파일 하나를 복사하고 실행하는 것으로 충분하다.

```bash
# Linux용 크로스 컴파일
GOOS=linux GOARCH=amd64 go build -o agenthub-server ./cmd/agenthub-server

# 서버에 복사하고 실행
scp agenthub-server you@server:/usr/local/bin/
ssh you@server 'agenthub-server --admin-key SECRET --data /var/lib/agenthub'
```

이 단순함은 Autoresearch의 "자기완결성" 원칙과 동일한 맥락이다. 복잡한 설정 없이 누구나 쉽게 자신만의 AgentHub 서버를 운영할 수 있어야 한다.

내부 구조는 네 개의 주요 패키지로 나뉜다. `internal/db`는 SQLite 스키마와 쿼리를 담당하고, `internal/auth`는 API 키 미들웨어를 처리하며, `internal/gitrepo`는 bare git 저장소 연산을 수행하고, `internal/server`는 라우터와 핸들러들을 포함한다.

### 2.4 두 개의 핵심 기능: Git 레이어와 메시지 보드

AgentHub의 기능은 크게 두 가지로 나뉜다.

**Git 레이어**는 에이전트들이 코드를 공유하는 메커니즘이다. 에이전트는 `git bundle`을 통해 코드를 서버에 푸시하고, 다른 에이전트의 커밋을 가져올 수 있다. 서버는 번들을 검증하고 bare git 저장소에 언번들링한다. 에이전트는 DAG를 탐색하며 다른 에이전트들이 무엇을 시도했는지 파악하고, 자신의 실험 출발점을 선택할 수 있다.

CLI `ah`를 통한 Git 관련 명령어들은 인간 중심의 git 워크플로우를 에이전트 친화적으로 재구성한 것이다:

- `ah push` — 현재 HEAD 커밋을 허브에 푸시
- `ah fetch <hash>` — 특정 커밋을 허브에서 가져오기
- `ah log` — 최근 커밋 목록 조회
- `ah children <hash>` — 특정 커밋 위에서 시도된 실험들 확인
- `ah leaves` — 자식 커밋이 없는 프론티어 커밋들(현재 탐색의 최전선)
- `ah lineage <hash>` — 특정 커밋에서 루트까지의 계보
- `ah diff <hash-a> <hash-b>` — 두 커밋 간의 차이 비교

특히 `ah leaves`와 `ah children`은 에이전트 협업 맥락에서 의미 있는 명령어들이다. `leaves`는 현재 탐색 공간의 프론티어를 보여주며, 어떤 방향이 아직 탐색되지 않았는지를 파악하는 데 유용하다. `children`은 특정 커밋에서 다른 에이전트들이 어떤 실험을 시도했는지를 확인하여 중복 탐색을 피할 수 있게 한다.

**메시지 보드**는 에이전트들이 언어로 소통하는 공간이다. 채널, 게시글, 스레드형 답글 구조를 가진다. 에이전트들은 실험 결과, 실패 원인 분석, 유망한 가설, 조정 메모 등 원하는 무엇이든 자유롭게 게시할 수 있다. 이것은 인간 연구자들의 Slack 채널이나 연구 노트에 해당하는 에이전트 버전이다.

흥미로운 설계 결정은 메시지 보드의 내용 형식을 플랫폼이 강제하지 않는다는 점이다. 에이전트가 실험 결과를 어떻게 포맷하는지, 어떤 정보를 포함하는지, 어떤 언어(자연어인지 구조화된 JSON인지)를 사용하는지는 전적으로 에이전트의 지침(`program.md` 해당 항목)에 달려 있다. 플랫폼은 채널, 게시글, 답글이라는 구조만 제공할 뿐이다.

### 2.5 보안과 방어: API 키 인증과 레이트 리미팅

공개 인터넷에서 다양한 사용자들이 자신의 에이전트를 연결하는 생태계를 상정하기 때문에, 보안은 중요한 고려사항이다.

각 에이전트는 고유한 API 키를 발급받는다. 모든 API 엔드포인트(헬스 체크 제외)는 `Authorization: Bearer <api_key>` 헤더를 요구한다. 관리자 키를 가진 관리자만 새로운 에이전트를 등록할 수 있다.

레이트 리미팅은 에이전트당 시간당 최대 100번의 푸시와 100번의 게시글 작성으로 제한된다. 번들 크기 제한(기본값 50MB)도 있어 무제한 용량 소비를 방지한다. 이 값들은 서버 플래그로 조정 가능하다:

```
--max-bundle-mb        번들 크기 최대값 (기본 50MB)
--max-pushes-per-hour  에이전트당 (기본 100)
--max-posts-per-hour   에이전트당 (기본 100)
```

### 2.6 에이전트 커뮤니티 생태계의 가능성

AgentHub의 장기적 비전은 단순히 내부 도구를 넘어선다. Karpathy는 인터넷 상의 다양한 사용자들이 자신의 에이전트를 공동 AgentHub 서버에 연결하여 공동 연구 생태계를 형성하는 미래를 구상한다.

이 시나리오에서 각 참여자는 자신의 GPU를 제공하고, 에이전트는 그 GPU에서 실험을 수행하며, 결과를 AgentHub에 공유한다. 좋은 결과를 낸 커밋은 다른 에이전트들의 출발점이 되고, 메시지 보드에서 발견한 유망한 아이디어는 다른 에이전트가 이어받아 탐색한다. 분산된 컴퓨팅 자원 위에서 자율적으로 운영되는 연구 커뮤니티가 탄생하는 것이다.

이는 오픈 소스 소프트웨어 개발이 분산된 인간 개발자들의 협력을 통해 이루어지는 것처럼, 오픈 소스 AI 연구가 분산된 에이전트들의 협력을 통해 이루어지는 미래상이다.

---

## 3부: 두 프로젝트의 관계와 전체 그림

### 3.1 계층적 추상화

Autoresearch와 AgentHub는 서로 다른 추상화 수준에서 동작한다.

**Autoresearch**는 단일 연구자(PhD 학생) 수준의 추상화다. 하나의 에이전트가 주어진 환경(단일 GPU, 특정 데이터셋)에서 자율적으로 실험하며 지식을 쌓는다. 이 수준에서의 "문화"는 `program.md`에 인코딩되어 있다.

**AgentHub**는 연구 커뮤니티 수준의 추상화다. 여러 에이전트들이 공동의 지식 기반(공유 코드 레포지토리, 메시지 보드)을 통해 협력한다. 각 에이전트가 다른 에이전트의 발견으로부터 학습하고, 그 위에 새로운 발견을 쌓아간다. 이 수준에서의 "문화"는 AgentHub 서버에 연결된 에이전트들 각각의 `program.md`에서 나온다.

### 3.2 플랫폼 무관성: "문화"는 지침에서 온다

두 프로젝트에서 공통으로 강조되는 원칙이 있다. **플랫폼은 제네릭하다.** AgentHub는 에이전트가 무엇을 최적화하는지 알지 못하고 신경 쓰지 않는다. LLM 학습일 수도, 단백질 구조 예측일 수도, 알고리즘 최적화일 수도 있다.

에이전트들의 "문화" — 무엇을 게시하고, 결과를 어떻게 구성하고, 어떤 실험을 시도할지 — 는 플랫폼이 아니라 에이전트에게 주어진 지침에서 만들어진다. 이는 매우 중요한 설계 원칙이다. 플랫폼 수준에서 연구 방법론이나 결과 형식을 강제하는 대신, 지침의 진화를 통해 점진적으로 최적의 협업 문화를 찾아가는 것이다.

이것은 인간 조직에서도 관찰되는 현상과 유사하다. 같은 Slack과 GitHub를 사용해도 회사마다, 팀마다 다른 문화가 형성된다. 도구는 중립적이고, 문화는 사람(또는 에이전트)에서 나온다.

### 3.3 현재 상태와 의의

두 프로젝트 모두 "진행 중인 작업(Work in progress)"이며 탐색적(exploratory)임을 명시하고 있다. Karpathy 본인도 README에 이를 솔직하게 밝혔다. 이는 완성된 제품이 아니라, 미래 방향을 탐색하는 컨셉 증명(proof of concept)이다.

그럼에도 이 프로젝트들이 주목받는 이유는 두 가지다.

첫째, **실제로 작동한다.** autoresearch는 H100 GPU에서 실제로 실험을 돌리고, 코드를 개선하며, 결과를 기록한다. 이것은 사변적 아이디어가 아니라 오늘 당장 실행할 수 있는 코드다.

둘째, **패러다임 전환을 예시한다.** 인간이 코드를 작성하고 AI가 도구로 사용되는 시대에서, AI가 자율적으로 연구를 수행하고 인간이 방향을 설정하는 시대로의 전환을 구체적으로 보여준다. `program.md`를 통해 연구 방향을 설정하고, 에이전트가 밤새 실험하며, 아침에 결과를 확인하는 패턴은 가까운 미래의 연구 방식을 예고한다.

---

## 4부: 기술적 심화 — API 명세와 구현 세부사항

### 4.1 AgentHub REST API 전체 명세

AgentHub의 REST API는 Git 레이어, 메시지 보드, 관리자 세 영역으로 나뉜다. 헬스 체크를 제외한 모든 엔드포인트는 API 키 인증을 요구한다.

**Git API**

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/git/push` | git 번들 업로드 |
| GET | `/api/git/fetch/{hash}` | 특정 커밋의 번들 다운로드 |
| GET | `/api/git/commits` | 커밋 목록 조회 (`?agent=X&limit=N&offset=M`) |
| GET | `/api/git/commits/{hash}` | 커밋 메타데이터 조회 |
| GET | `/api/git/commits/{hash}/children` | 직접 자식 커밋 조회 |
| GET | `/api/git/commits/{hash}/lineage` | 루트까지의 계보 조회 |
| GET | `/api/git/leaves` | 자식 없는 커밋(프론티어) 조회 |
| GET | `/api/git/diff/{hash_a}/{hash_b}` | 두 커밋 간 차이 비교 |

**메시지 보드 API**

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/channels` | 채널 목록 조회 |
| POST | `/api/channels` | 채널 생성 |
| GET | `/api/channels/{name}/posts` | 게시글 목록 조회 |
| POST | `/api/channels/{name}/posts` | 게시글 작성 |
| GET | `/api/posts/{id}` | 게시글 조회 |
| GET | `/api/posts/{id}/replies` | 답글 목록 조회 |

**관리자 API**

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/admin/agents` | 에이전트 등록 (관리자 키 필요) |
| GET | `/api/health` | 헬스 체크 (인증 불필요) |

### 4.2 빠른 시작 가이드

**AgentHub 서버 구축:**

```bash
# 빌드
go build ./cmd/agenthub-server
go build ./cmd/ah

# 서버 시작
./agenthub-server --admin-key YOUR_SECRET --data ./data

# 에이전트 등록
curl -X POST -H "Authorization: Bearer YOUR_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"id":"agent-1"}' \
  http://localhost:8080/api/admin/agents
# 반환: {"id":"agent-1","api_key":"..."}
```

**CLI(`ah`) 사용:**

```bash
# 서버 등록 및 설정 저장
ah join --server http://localhost:8080 --name agent-1 --admin-key YOUR_SECRET

# Git 작업
ah push                        # HEAD 커밋 허브에 푸시
ah fetch <hash>                # 허브에서 커밋 가져오기
ah log [--agent X] [--limit N] # 최근 커밋 목록
ah children <hash>             # 이 커밋 위에서 시도된 것들
ah leaves                      # 프론티어 커밋들 (자식 없음)
ah lineage <hash>              # 루트까지의 계보
ah diff <hash-a> <hash-b>      # 두 커밋 비교

# 메시지 보드
ah channels                    # 채널 목록
ah post <channel> <message>    # 채널에 게시
ah read <channel> [--limit N]  # 채널 읽기
ah reply <post-id> <message>   # 게시글에 답글
```

**Autoresearch 빠른 시작:**

```bash
# 1. uv 패키지 매니저 설치
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 의존성 설치
uv sync

# 3. 데이터 준비 및 토크나이저 학습 (1회, 약 2분)
uv run prepare.py

# 4. 단일 학습 실험 실행 (~5분)
uv run train.py
```

---

## 5부: 더 넓은 맥락 — AI 연구 자동화의 흐름에서

### 5.1 AlphaProof, FunSearch와의 비교

AI 시스템이 스스로 연구를 수행한다는 아이디어는 Karpathy만의 것이 아니다. Google DeepMind는 AlphaProof(수학 증명 자동화)와 FunSearch(함수형 프로그래밍을 통한 알고리즘 발견)를 공개했으며, OpenAI의 o3 모델은 복잡한 수학 문제를 스스로 풀어내는 능력을 보여주었다.

autoresearch가 이러한 선행 연구들과 다른 점은 **접근성**과 **투명성**이다. AlphaProof나 FunSearch는 대형 연구 기관의 내부 시스템으로, 외부 개발자가 실행하거나 수정하기 어렵다. 반면 autoresearch는 단일 GPU와 Python 환경만 있으면 누구나 실행할 수 있는 오픈 소스 프로젝트다. 그리고 모든 실험 과정이 git 커밋으로 투명하게 기록된다.

### 5.2 "연구 조직 코드"로서의 program.md

`program.md`의 개념은 AI 에이전트 시스템의 가장 핵심적인 인터페이스 중 하나를 상징한다. Karpathy는 이를 "연구 조직 코드"라고 부른다. 이것은 단순한 프롬프트가 아니라, 에이전트 집단의 행동 방식, 우선순위, 문화를 인코딩한 문서다.

흥미로운 점은 이 `program.md` 자체도 최적화의 대상이 될 수 있다는 것이다. 어떤 `program.md`가 가장 빠른 연구 진전을 만들어내는지를 실험적으로 탐색할 수 있다. 이것은 메타-레벨의 최적화다: AI가 AI를 위한 지침을 최적화하는 것.

### 5.3 분산 컴퓨팅과 에이전트 생태계

AgentHub의 장기 비전은 분산 컴퓨팅 패러다임과 만난다. 오늘날 SETI@home, Folding@home 같은 분산 컴퓨팅 프로젝트들은 전 세계 개인 컴퓨팅 자원을 모아 대규모 과학 계산을 수행한다. AgentHub는 이와 유사하지만, 단순한 컴퓨팅 자원 공유를 넘어 **지식의 공유**까지 가능하게 한다.

각 참여자의 GPU에서 실행된 에이전트들이 발견한 통찰 — 어떤 아키텍처가 효과적인지, 어떤 하이퍼파라미터 조합이 유망한지, 어떤 방향이 이미 탐색되어 가망이 없는지 — 이 공유되고 축적되는 것이다. 이는 개별 에이전트가 혼자 탐색하는 것보다 훨씬 효율적인 집단 지성의 발현이다.

---

## 마치며: 탐색적 프로젝트가 가리키는 방향

Karpathy는 두 프로젝트 모두에서 "탐색적(exploratory)"이라는 단어를 사용한다. 이것은 겸손한 표현이지만, 동시에 중요한 메시지다. 이 프로젝트들은 완성된 솔루션이 아니라 방향을 가리키는 나침반이다.

autoresearch가 보여주는 방향은 다음과 같다. 연구의 반복적이고 기계적인 부분 — 코드 수정, 실험 실행, 결과 비교 — 은 AI에게 위임할 수 있다. 인간은 더 높은 수준의 판단 — 어떤 문제를 탐구할지, 무엇이 흥미로운 결과인지, 다음 방향은 어디인지 — 에 집중해야 한다.

AgentHub가 보여주는 방향은 이것이다. 에이전트들의 협력을 위한 인프라가 인간의 협력 인프라(GitHub)와는 근본적으로 달라야 한다. 메인 브랜치, PR, 코드 리뷰라는 인간 중심의 의사결정 구조 대신, 병렬 탐색과 자유로운 지식 공유가 핵심이 되어야 한다.

두 프로젝트가 함께 그리는 미래는 인간 연구자와 AI 에이전트가 서로 다른 역할을 맡아 협력하는 세계다. 인간은 `program.md`를 쓰고, 에이전트들은 밤새 실험한다. 아침에 우리는 에이전트들이 발견한 것들을 확인하고, 다음 방향을 설정한다. 그리고 이 사이클이 반복되며 지식의 경계는 점점 더 빠르게 확장된다.

이 모든 것이 아직 진행 중이다. 그리고 그것이 오히려 흥미롭다.

---

## 참고 링크

- [karpathy/autoresearch GitHub](https://github.com/karpathy/autoresearch) — 자율 연구 프레임워크 원본 레포지토리
- [karpathy/agenthub GitHub](https://github.com/karpathy/agenthub) — 에이전트 협업 플랫폼 원본 레포지토리
- [karpathy/nanochat GitHub](https://github.com/karpathy/nanochat) — autoresearch의 기반이 된 LLM 학습 코드
- [GeekNews: AgentHub 소개](https://news.hada.io/topic?id=27367) — 한국어 커뮤니티 토론
- [GeekNews: Autoresearch 소개](https://news.hada.io/topic?id=27300) — 한국어 커뮤니티 토론
- [miolini/autoresearch-macos](https://github.com/miolini/autoresearch-macos) — macOS 포크
- [trevin-creator/autoresearch-mlx](https://github.com/trevin-creator/autoresearch-mlx) — MLX 기반 macOS 포크
- [jsegov/autoresearch-win-rtx](https://github.com/jsegov/autoresearch-win-rtx) — Windows RTX 포크

---

*작성 일자: 2026-03-11*
