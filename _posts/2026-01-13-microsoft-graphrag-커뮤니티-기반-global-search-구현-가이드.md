---
title: "Microsoft GraphRAG 커뮤니티 기반 Global Search 구현 가이드"
date: 2026-01-13 20:30:00 +0900
categories: [AI,  RAG]
mermaid: [True]
tags: [AI,  RAG,  graph-rag,  Neo4j,  Microsoft-GraphRAG,  community-based-global-search,  Material,  Medium,  Claude.write]
---


## 문서 정보
- **작성일**: 2026-01-13
- **버전**: 1.0
- **대상**: 기존 GraphRAG-Neo4j 시스템 구축 완료 후 고도화 단계

---

## 목차
1. [개요 및 필요성](#1-개요-및-필요성)
2. [커뮤니티 탐지 이론과 실무](#2-커뮤니티-탐지-이론과-실무)
3. [커뮤니티 요약 생성](#3-커뮤니티-요약-생성)
4. [Global Search 구현](#4-global-search-구현)
5. [Local Search 고도화](#5-local-search-고도화)
6. [성능 최적화 및 비용 관리](#6-성능-최적화-및-비용-관리)
7. [실전 적용 가이드](#7-실전-적용-가이드)

---

[Implementing Microsoft’s GraphRAG Architecture with Neo4j](https://pub.towardsai.net/implementing-microsofts-graphrag-architecture-with-neo4j-95bb4300db1b)


## 1. 개요 및 필요성

### 1.1 현재 시스템의 한계

기존에 구축한 GraphRAG-Neo4j 시스템은 개별 엔티티와 관계를 효과적으로 저장하고 검색할 수 있습니다. Elasticsearch의 제로 조인 아키텍처를 통해 평균 0.8초 이내의 빠른 응답을 제공하며, DeepSeek을 활용한 비용 효율적인 엔티티 추출이 가능합니다. 그러나 이러한 시스템은 "구체적인 사실 검색"에 최적화되어 있어, 다음과 같은 질문 유형에는 충분한 답변을 제공하기 어렵습니다.

**기존 시스템이 어려워하는 질문들**:
- "우리 조직의 모든 프로젝트에서 나타나는 공통 기술 트렌드는 무엇인가?"
- "지난 2년간 보안 관련 이슈는 어떻게 진화해왔는가?"
- "서로 다른 부서의 프로젝트들 사이에 어떤 연결고리가 존재하는가?"
- "전사적으로 가장 영향력 있는 기술 스택은 무엇이며, 어떤 팀들이 선도하고 있는가?"

이러한 질문들의 공통점은 "개별 문서나 엔티티"가 아닌 "전체 지식 그래프에서 나타나는 패턴과 구조"를 묻고 있다는 것입니다. 벡터 검색은 의미적으로 유사한 개별 청크를 찾는 데는 탁월하지만, 그래프 전체를 조망하는 통찰을 제공하기에는 본질적으로 한계가 있습니다.

### 1.2 커뮤니티 기반 접근의 핵심 아이디어

Microsoft의 GraphRAG 연구는 이 문제를 해결하기 위해 "커뮤니티(Community)"라는 개념을 도입했습니다. 커뮤니티란 지식 그래프 내에서 서로 밀접하게 연결된 엔티티들의 클러스터를 의미합니다. 예를 들어, "프로젝트 A"에 참여한 사람들, 그 프로젝트에서 사용된 기술들, 관련 문서들이 하나의 커뮤니티를 형성합니다. 또 다른 예로, "보안 취약점 대응"이라는 주제로 연결된 여러 프로젝트, 담당자, 절차 문서들이 별도의 커뮤니티를 구성할 수 있습니다.

중요한 점은 이러한 커뮤니티들이 자동으로 탐지되고, 각 커뮤니티에 대해 LLM이 생성한 고수준의 요약이 만들어진다는 것입니다. 이 요약은 단순히 엔티티 목록이 아니라, 커뮤니티 내에서 일어나는 활동의 본질, 주요 패턴, 중요한 관계들을 압축한 "의미적 캡슐"입니다. 

이렇게 생성된 커뮤니티 요약들은 전체 지식 그래프의 "압축된 뷰"를 제공합니다. 수만 개의 개별 문서와 엔티티를 모두 읽는 대신, 수십 개의 커뮤니티 요약만 읽어도 조직 전체의 지식 지형을 파악할 수 있게 됩니다. 이것이 바로 "Global Search"의 핵심 원리입니다.

### 1.3 Local Search vs Global Search

기존 시스템은 본질적으로 "Local Search" 방식입니다. 쿼리에 대해 벡터적으로 가장 유사한 몇 개의 엔티티나 청크를 찾고, 그 주변의 그래프 구조를 탐색하여 답변을 구성합니다. 이는 "김철수가 참여한 프로젝트는?"이나 "React 사용 가이드는 어디에 있나?" 같은 구체적이고 국지적인 질문에 매우 효과적입니다.

반면 Global Search는 커뮤니티 요약들을 대상으로 작동합니다. 전체 커뮤니티 집합에서 질문과 관련된 것들을 선택하고, 각 커뮤니티에 대해 독립적으로 부분 답변을 생성한 후(Map 단계), 이들을 종합하여 최종 답변을 만듭니다(Reduce 단계). 이는 여러 부서, 여러 프로젝트에 걸친 넓은 맥락에서의 질문에 적합합니다.

**비유로 이해하기**:
- **Local Search**: 도서관에서 특정 책의 특정 페이지를 찾는 것
- **Global Search**: 도서관 전체의 주제별 섹션 요약을 읽고 전체 장서의 경향을 파악하는 것

### 1.4 언제 Global Search가 필요한가?

실무에서 Global Search가 진가를 발휘하는 경우는 다음과 같습니다:

**적합한 상황**:
1. **전략 기획 및 의사결정**: 경영진이 "우리 조직의 기술 부채는 어느 정도이며 어디에 집중되어 있는가?"와 같은 거시적 질문을 할 때
2. **신규 직원 온보딩**: "우리 회사의 주요 프로젝트들과 그 상호관계는?"처럼 전체 조직 구조를 이해하려 할 때
3. **연구 및 분석**: "지난 3년간 클라우드 마이그레이션 관련 모든 활동을 종합하면?"과 같은 역사적 패턴 분석
4. **리스크 관리**: "보안 취약점이 발견된 모든 프로젝트 영역은?"처럼 전사적 영향 범위 파악

**부적합한 상황**:
1. **일상적 업무 지원**: "오늘 회의 자료 어디 있지?" → Local Search가 더 빠름
2. **구체적 사실 확인**: "프로젝트 A의 배포 일정은?" → 직접 검색이 효율적
3. **실시간 문제 해결**: "이 에러 메시지 해결법은?" → 즉각적 답변 필요

따라서 Global Search는 기존 시스템을 대체하는 것이 아니라, 특정 유형의 질문에 대해 보완적으로 사용되는 기능입니다. 실제 운영에서는 질문의 특성을 분석하여 Local과 Global 중 적절한 방식을 자동으로 선택하는 하이브리드 접근이 이상적입니다.

---

## 2. 커뮤니티 탐지 이론과 실무

### 2.1 그래프 커뮤니티란 무엇인가

그래프 이론에서 커뮤니티는 "내부 연결은 강하고 외부 연결은 약한 노드들의 집합"으로 정의됩니다. 지식 그래프에서 이것은 구체적으로 무엇을 의미할까요?

예를 들어 "React 프론트엔드 개발" 관련 문서들을 생각해봅시다. 이 문서들에서 추출된 엔티티들 - "React", "Redux", "Webpack", "김철수(프론트엔드 개발자)", "프로젝트 A", "성능 최적화" 등 - 은 서로 밀접하게 연결되어 있습니다. 김철수는 React를 사용하여 프로젝트 A를 진행했고, 그 과정에서 Redux로 상태 관리를 하고 Webpack으로 빌드했으며, 성능 최적화 이슈를 다루었습니다. 이 엔티티들 사이에는 수많은 관계가 존재합니다.

반면, 같은 그래프에 "데이터베이스 백업 절차" 관련 엔티티들 - "PostgreSQL", "pg_dump", "이영희(DBA)", "백업 스크립트", "재해 복구" 등 - 도 존재합니다. 이들도 서로 강하게 연결되어 있지만, React 관련 엔티티들과는 상대적으로 약하게 연결되어 있습니다. 물론 "프로젝트 A가 PostgreSQL을 사용한다"는 연결은 있을 수 있지만, React-Redux 사이의 관계만큼 강하지는 않습니다.

이렇게 자연스럽게 형성된 클러스터들이 바로 커뮤니티입니다. 중요한 점은 이러한 구조가 미리 정의된 것이 아니라, 실제 데이터에서 자동으로 발견된다는 것입니다.

### 2.2 Louvain 알고리즘의 작동 원리

커뮤니티를 자동으로 탐지하는 방법은 여러 가지가 있지만, Microsoft GraphRAG와 Neo4j GDS(Graph Data Science)에서 권장하는 방법은 Louvain 알고리즘입니다. 이 알고리즘은 "모듈성(Modularity)"이라는 품질 지표를 최대화하는 방향으로 노드들을 그룹화합니다.

모듈성은 간단히 말해 "실제 커뮤니티 내부 연결 밀도"가 "무작위로 연결했을 때 기대되는 밀도"보다 얼마나 높은지를 측정합니다. 값이 높을수록 명확한 커뮤니티 구조가 존재한다는 의미입니다.

Louvain 알고리즘은 두 단계를 반복합니다:

**1단계: 지역 최적화**
각 노드를 순회하면서, 그 노드를 이웃한 커뮤니티로 이동시켰을 때 모듈성이 증가하는지 확인합니다. 증가한다면 이동시키고, 아니면 그대로 둡니다. 모든 노드에 대해 이 과정을 반복하여 더 이상 개선이 없을 때까지 계속합니다.

**2단계: 네트워크 응집**
탐지된 각 커뮤니티를 하나의 "슈퍼노드"로 축약하여 새로운 그래프를 만듭니다. 커뮤니티 간 연결은 보존됩니다.

이 두 단계를 반복하면서 점점 더 큰 단위의 커뮤니티를 형성해갑니다. 이를 통해 "계층적 커뮤니티 구조"를 얻을 수 있습니다. 예를 들어:
- Level 0: 개별 엔티티 (가장 세밀)
- Level 1: 작은 기능 단위 (예: "React 상태 관리")
- Level 2: 중간 프로젝트 단위 (예: "프론트엔드 개발")
- Level 3: 큰 부서 단위 (예: "개발팀 전체")

### 2.3 Neo4j GDS를 활용한 구현

Neo4j Graph Data Science 라이브러리는 Louvain을 포함한 다양한 그래프 알고리즘을 메모리 내에서 고속으로 실행할 수 있게 해줍니다. 기본적인 워크플로우는 다음과 같습니다.

먼저 Neo4j의 디스크 기반 저장소에서 분석하려는 부분 그래프를 메모리로 프로젝션합니다. 우리의 경우 `__Entity__` 노드들과 그들 사이의 `RELATIONSHIP` 또는 `SUMMARIZED_RELATIONSHIP` 엣지를 선택합니다.

```python
def project_entity_graph(driver, graph_id: str):
    """엔티티 그래프를 GDS 메모리로 프로젝션"""
    
    with driver.session() as session:
        # 기존 프로젝션 제거
        try:
            session.run("CALL gds.graph.drop('entity_graph')")
        except:
            pass
        
        # 새 프로젝션 생성
        result = session.run("""
            MATCH (source:__Entity__ {graph_id: $graph_id})-[r:SUMMARIZED_RELATIONSHIP]->(target:__Entity__ {graph_id: $graph_id})
            WITH gds.graph.project(
                'entity_graph',
                source,
                target,
                {
                    relationshipProperties: r.strength
                },
                {undirectedRelationshipTypes: ['*']}
            ) AS g
            RETURN g.graphName AS name, 
                   g.nodeCount AS nodes, 
                   g.relationshipCount AS rels
        """, graph_id=graph_id)
        
        stats = result.single()
        print(f"프로젝션 완료: {stats['nodes']}개 노드, {stats['rels']}개 관계")
        return stats
```

이제 메모리에 로드된 그래프에 대해 Louvain을 실행합니다. 중요한 것은 관계의 가중치를 고려하는 것입니다. DeepSeek이 추출할 때 부여한 `relationship_strength` 값을 활용하면 더 의미 있는 커뮤니티를 탐지할 수 있습니다. 강한 관계로 연결된 엔티티들이 같은 커뮤니티에 속할 확률이 높아집니다.

```python
def run_louvain_detection(driver):
    """Louvain 커뮤니티 탐지 실행"""
    
    with driver.session() as session:
        result = session.run("""
            CALL gds.louvain.write('entity_graph', {
                writeProperty: 'louvain',
                relationshipWeightProperty: 'strength',
                includeIntermediateCommunities: false
            })
            YIELD communityCount, 
                  modularity, 
                  ranIterations
        """)
        
        stats = result.single()
        
        print(f"커뮤니티 탐지 완료:")
        print(f"  - 탐지된 커뮤니티: {stats['communityCount']}개")
        print(f"  - 모듈성 점수: {stats['modularity']:.4f}")
        print(f"  - 반복 횟수: {stats['ranIterations']}")
        
        return stats
```

실행 후 각 `__Entity__` 노드에는 `louvain` 속성이 추가되며, 이 값이 커뮤니티 ID가 됩니다. 같은 ID를 가진 엔티티들이 하나의 커뮤니티를 구성합니다.

### 2.4 커뮤니티 크기와 품질 검증

모든 커뮤니티가 똑같이 유용한 것은 아닙니다. 너무 작은 커뮤니티(예: 엔티티 2-3개)는 의미 있는 패턴을 담기 어렵고, 너무 큰 커뮤니티는 지나치게 일반적이어서 구체성이 떨어집니다.

```python
def analyze_community_quality(driver, graph_id: str):
    """커뮤니티 크기 분포 및 품질 분석"""
    
    with driver.session() as session:
        result = session.run("""
            MATCH (e:__Entity__ {graph_id: $graph_id})
            WHERE e.louvain IS NOT NULL
            WITH e.louvain AS communityId, count(e) AS size
            RETURN communityId, size
            ORDER BY size DESC
        """, graph_id=graph_id)
        
        communities = result.data()
        
        # 통계 계산
        sizes = [c['size'] for c in communities]
        
        print(f"\n커뮤니티 분석:")
        print(f"  - 전체 커뮤니티 수: {len(communities)}")
        print(f"  - 평균 크기: {sum(sizes)/len(sizes):.1f}")
        print(f"  - 최소/최대 크기: {min(sizes)} / {max(sizes)}")
        print(f"  - 싱글톤 (크기 1): {sum(1 for s in sizes if s == 1)}")
        
        # 크기별 분포
        size_dist = {}
        for s in sizes:
            if s == 1:
                key = "1"
            elif s <= 5:
                key = "2-5"
            elif s <= 10:
                key = "6-10"
            elif s <= 20:
                key = "11-20"
            else:
                key = "20+"
            size_dist[key] = size_dist.get(key, 0) + 1
        
        print(f"\n  크기별 분포:")
        for key in ["1", "2-5", "6-10", "11-20", "20+"]:
            if key in size_dist:
                print(f"    {key}: {size_dist[key]}개")
        
        return communities
```

일반적으로 크기 4개 이상의 커뮤니티만 요약 생성 대상으로 삼는 것이 좋습니다. 싱글톤이나 작은 커뮤니티는 노이즈일 가능성이 높습니다.

### 2.5 계층적 커뮤니티 구조 활용

Louvain은 `includeIntermediateCommunities: true` 옵션을 주면 각 반복 단계에서의 중간 커뮤니티 구조도 함께 반환합니다. 이를 활용하면 다층적인 추상화를 얻을 수 있습니다.

예를 들어:
- **Level 1 커뮤니티**: "React Hook 사용 패턴" (엔티티 5개)
- **Level 2 커뮤니티**: "프론트엔드 개발 전반" (Level 1 커뮤니티 여러 개 포함)
- **Level 3 커뮤니티**: "웹 애플리케이션 개발" (Level 2 커뮤니티 여러 개 포함)

이런 계층 구조는 Global Search에서 질문의 추상화 수준에 따라 적절한 레벨의 커뮤니티를 선택하는 데 사용될 수 있습니다. "React Hook 구체적인 사용법"이라면 Level 1을, "우리 팀의 웹 개발 방향"이라면 Level 3을 참조하는 식입니다.

그러나 실무적으로는 단일 레벨로 시작하는 것을 권장합니다. 계층 구조는 시스템이 성숙한 후 추가하는 것이 관리가 용이합니다.

---

## 3. 커뮤니티 요약 생성

### 3.1 왜 LLM 요약이 필요한가

Louvain으로 커뮤니티를 탐지했다면, 각 커뮤니티는 엔티티 목록과 그들 사이의 관계로 표현됩니다. 예를 들어 커뮤니티 #42는 다음과 같을 수 있습니다:

**엔티티**:
- React (기술)
- Redux (기술)
- 김철수 (개발자)
- 프로젝트 A (프로젝트)
- 성능 최적화 (개념)

**관계**:
- 김철수 → 프로젝트 A (PARTICIPATED)
- 프로젝트 A → React (USES)
- React → Redux (DEPENDS_ON)
- 프로젝트 A → 성능 최적화 (RELATED_TO)

이 구조화된 데이터를 그대로 저장해도 되지만, 이것만으로는 "이 커뮤니티가 무엇에 관한 것인지"를 빠르게 파악하기 어렵습니다. 특히 커뮤니티가 수십 개일 때, 각각을 이해하려면 엔티티와 관계를 일일이 분석해야 합니다.

LLM 요약은 이 구조화된 데이터를 "사람이 읽기 쉬운 내러티브"로 변환합니다. 위 커뮤니티는 다음과 같이 요약될 수 있습니다:

> **커뮤니티 요약**: "프로젝트 A React 프론트엔드 개발"
> 
> 이 커뮤니티는 김철수가 주도한 프로젝트 A의 프론트엔드 개발 활동을 중심으로 형성되었습니다. React를 주요 프레임워크로 사용하며, 상태 관리를 위해 Redux를 채택했습니다. 프로젝트 진행 중 성능 최적화가 주요 과제로 부각되었으며, 이와 관련된 다양한 기술적 논의가 있었습니다.

이런 요약은 Global Search 시 매우 유용합니다. "우리 팀의 React 사용 현황"이라는 질문에 대해, 수십 개 커뮤니티 요약을 빠르게 스캔하면서 React 관련 커뮤니티들을 식별할 수 있습니다.

### 3.2 커뮤니티 데이터 수집 쿼리

요약을 생성하기 전에, 각 커뮤니티의 완전한 구조 데이터를 수집해야 합니다. 다음 Cypher 쿼리는 각 커뮤니티에 속한 노드들과 그들 사이의 모든 관계를 추출합니다.

```python
def collect_community_data(driver, graph_id: str):
    """각 커뮤니티의 엔티티 및 관계 수집"""
    
    with driver.session() as session:
        result = session.run("""
            MATCH (e:__Entity__ {graph_id: $graph_id})
            WHERE e.louvain IS NOT NULL
            WITH e.louvain AS communityId, collect(e) AS nodes
            WHERE size(nodes) > 3  // 최소 4개 이상
            
            // 커뮤니티 내부 서브그래프 추출
            CALL apoc.path.subgraphAll(nodes[0], {
                whitelistNodes: nodes
            })
            YIELD relationships
            
            // 노드 정보 수집
            UNWIND nodes AS n
            WITH communityId, collect({
                id: n.name,
                description: n.summary,
                type: [label IN labels(n) WHERE label <> '__Entity__'][0]
            }) AS nodeList, relationships
            
            // 관계 정보 수집
            UNWIND relationships AS r
            WITH communityId, nodeList, collect({
                start: startNode(r).name,
                type: type(r),
                end: endNode(r).name,
                description: CASE 
                    WHEN r.summary IS NOT NULL THEN r.summary
                    WHEN r.description IS NOT NULL THEN r.description
                    ELSE ''
                END
            }) AS relList
            
            RETURN communityId, nodeList AS nodes, relList AS rels
        """, graph_id=graph_id)
        
        communities = result.data()
        print(f"수집된 커뮤니티: {len(communities)}개")
        
        return communities
```

이 쿼리는 `apoc.path.subgraphAll` 함수를 사용하여 커뮤니티 내부의 모든 관계를 효율적으로 가져옵니다. APOC이 설치되어 있지 않다면 Neo4j 플러그인 디렉토리에 추가해야 합니다.

### 3.3 프롬프트 엔지니어링: 구조화된 요약 생성

커뮤니티 데이터를 LLM에 전달할 때, 명확하고 구조화된 프롬프트가 중요합니다. Microsoft의 GraphRAG 논문에서 제안한 프롬프트 구조를 한국어 환경에 맞게 조정합니다.

```python
COMMUNITY_SUMMARY_PROMPT = """당신은 지식 그래프 커뮤니티 분석 전문가입니다.

주어진 엔티티와 관계 정보를 바탕으로 이 커뮤니티에 대한 포괄적인 보고서를 작성하세요.

# 입력 데이터

**엔티티 목록**:
{entities}

**관계 목록**:
{relationships}

# 보고서 요구사항

다음 항목을 포함하는 JSON 형식으로 작성하세요:

1. **title**: 커뮤니티를 대표하는 간결한 제목 (예: "프로젝트 A React 개발팀")
   - 가능하면 구체적인 엔티티 이름 포함
   - 15단어 이내

2. **summary**: 커뮤니티의 전체 구조와 주요 활동에 대한 요약 (200-300자)
   - 엔티티들이 어떻게 연결되어 있는지
   - 이 커뮤니티의 주요 목적이나 활동
   - 특기할 만한 패턴이나 특징

3. **rating**: 0.0-10.0 사이의 중요도 점수
   - 이 커뮤니티가 조직에 미치는 영향도
   - 엔티티의 수, 관계의 강도, 활동의 활성도 등을 종합 고려
   - 일반적으로 5.0이 평균

4. **rating_explanation**: 점수 부여 근거 (1-2문장)

5. **findings**: 5-10개의 핵심 인사이트
   각 인사이트는 다음 형식:
{% raw %}
   {{
       "summary": "인사이트 요약 (1문장)",
       "explanation": "상세 설명 (2-3문장, 데이터 참조 포함)"
   }}
{% endraw %}
# 데이터 참조 규칙

설명 시 반드시 데이터 출처를 명시하세요:

형식: "김철수는 프로젝트 A의 리더입니다 [Data: Entities (김철수, 프로젝트 A); Relationships (1, 2)]"

- 최대 5개의 참조만 나열하고 더 있으면 "+more" 추가
- 근거 없는 추측이나 외부 지식은 포함 금지

# 출력 형식

반드시 다음 JSON 구조로만 응답하세요 (코드 블록 없이):
{% raw %}
{{
    "title": "...",
    "summary": "...",
    "rating": 7.5,
    "rating_explanation": "...",
    "findings": [
        {{
            "summary": "...",
            "explanation": "..."
        }}
    ]
}}
"""
{% endraw %}
def generate_community_summary(llm_client, entities, relationships):
    """단일 커뮤니티에 대한 LLM 요약 생성"""
    
    # 데이터 포맷팅
    entities_str = json.dumps(entities, ensure_ascii=False, indent=2)
    rels_str = json.dumps(relationships, ensure_ascii=False, indent=2)
    
    prompt = COMMUNITY_SUMMARY_PROMPT.format(
        entities=entities_str,
        relationships=rels_str
    )
    
    # LLM 호출
    response = llm_client.invoke(prompt)
    
    # JSON 파싱
    try:
        # 코드 블록 제거
        content = response.content
        if "```json" in content:
            content = content.split("```json")[1].split("```")[0]
        elif "```" in content:
            content = content.split("```")[1].split("```")[0]
        
        summary = json.loads(content.strip())
        return summary
        
    except json.JSONDecodeError as e:
        logger.error(f"JSON 파싱 실패: {e}")
        logger.debug(f"응답 내용: {response.content}")
        return None
```

### 3.4 배치 처리 및 비용 최적화

수십 개의 커뮤니티를 요약하려면 상당한 LLM 비용이 발생할 수 있습니다. DeepSeek을 활용하면 이 비용을 크게 줄일 수 있습니다.

```python
def generate_all_summaries(driver, llm_client, graph_id: str):
    """모든 커뮤니티 요약 생성 및 저장"""
    
    # 1. 커뮤니티 데이터 수집
    communities = collect_community_data(driver, graph_id)
    
    # 2. 요약 생성
    summaries = []
    total_cost = 0
    
    for community in tqdm(communities, desc="커뮤니티 요약 생성"):
        summary = generate_community_summary(
            llm_client,
            community['nodes'],
            community['rels']
        )
        
        if summary:
            summaries.append({
                'communityId': community['communityId'],
                **summary
            })
    
    print(f"\n생성된 요약: {len(summaries)}/{len(communities)}개")
    
    # 3. Neo4j에 저장
    save_community_summaries(driver, summaries, graph_id)
    
    return summaries

def save_community_summaries(driver, summaries, graph_id: str):
    """커뮤니티 요약을 Neo4j에 저장"""
    
    with driver.session() as session:
        session.run("""
            UNWIND $data AS row
            MERGE (c:__Community__ {
                communityId: row.communityId, 
                graph_id: $graph_id
            })
            SET c.title = row.title,
                c.summary = row.summary,
                c.rating = row.rating,
                c.rating_explanation = row.rating_explanation,
                c.findings = row.findings,
                c.updated_at = datetime()
            
            // 엔티티와 연결
            WITH c, row
            MATCH (e:__Entity__ {louvain: row.communityId, graph_id: $graph_id})
            MERGE (e)-[:IN_COMMUNITY]->(c)
        """, data=summaries, graph_id=graph_id)
    
    print(f"✓ {len(summaries)}개 커뮤니티 요약 저장 완료")
```

**비용 추정**:
- 커뮤니티당 평균 엔티티 10개, 관계 15개
- 프롬프트 토큰: ~1,500 tokens
- 응답 토큰: ~800 tokens
- 40개 커뮤니티 기준: 40 × 2,300 = 92,000 tokens
- DeepSeek 비용: $0.027/1M input + $0.11/1M output = **약 $0.01**

Claude나 GPT-4를 쓴다면 10-20배 더 비쌉니다. 커뮤니티 요약은 한 번만 생성하면 되므로 초기 비용이지만, DeepSeek을 쓰면 거의 무시할 수준입니다.

### 3.5 요약 품질 검증 및 개선

생성된 요약의 품질을 확인하고 필요시 재생성하는 것이 중요합니다.

```python
def validate_summaries(driver, graph_id: str):
    """요약 품질 검증"""
    
    with driver.session() as session:
        result = session.run("""
            MATCH (c:__Community__ {graph_id: $graph_id})
            RETURN c.communityId AS id,
                   c.title AS title,
                   c.rating AS rating,
                   size(c.findings) AS findings_count
        """, graph_id=graph_id)
        
        communities = result.data()
        
        issues = []
        
        for c in communities:
            # 제목 너무 짧음
            if len(c['title']) < 5:
                issues.append(f"Community {c['id']}: 제목 너무 짧음")
            
            # 인사이트 부족
            if c['findings_count'] < 3:
                issues.append(f"Community {c['id']}: 인사이트 {c['findings_count']}개만 생성됨")
            
            # 비정상적 rating
            if c['rating'] < 0 or c['rating'] > 10:
                issues.append(f"Community {c['id']}: 비정상적 rating {c['rating']}")
        
        if issues:
            print(f"\n⚠️ 발견된 문제 {len(issues)}개:")
            for issue in issues[:10]:
                print(f"  - {issue}")
        else:
            print("✓ 모든 커뮤니티 요약 품질 양호")
        
        return issues
```

문제가 발견되면 해당 커뮤니티만 재생성하거나, 프롬프트를 개선하여 전체를 다시 생성할 수 있습니다.

---

## 4. Global Search 구현

### 4.1 Map-Reduce 패러다임

Global Search의 핵심은 Map-Reduce 패턴입니다. 이는 대규모 데이터 처리에서 널리 쓰이는 방식으로, 여기서는 다음과 같이 적용됩니다:

**Map 단계**: 각 커뮤니티를 독립적으로 분석
- 입력: 사용자 질문 + 단일 커뮤니티 요약
- 처리: LLM이 해당 커뮤니티 관점에서 질문에 답변
- 출력: 커뮤니티별 부분 답변 (핵심 포인트 + 중요도 점수)

**Reduce 단계**: 모든 부분 답변을 통합
- 입력: 모든 커뮤니티의 부분 답변들
- 처리: LLM이 중복 제거, 우선순위 정렬, 일관성 있게 통합
- 출력: 최종 종합 답변

이 방식의 장점은 병렬화가 가능하다는 것입니다. 각 커뮤니티 분석은 독립적이므로 동시에 실행할 수 있습니다. 또한 품질 필터링도 쉽습니다. 중요도 점수가 낮은 부분 답변은 Reduce 단계에서 제외할 수 있습니다.

### 4.2 Map 단계 구현

```python
MAP_PROMPT_TEMPLATE = """당신은 데이터 분석 전문가입니다.

주어진 커뮤니티 보고서를 바탕으로 사용자의 질문에 답변하세요.

# 커뮤니티 보고서

**제목**: {title}

**요약**: {summary}

**주요 발견사항**:
{findings}

# 사용자 질문

{query}

# 답변 요구사항

이 커뮤니티 관점에서 질문과 관련된 핵심 포인트들을 추출하세요.

다음 JSON 형식으로 응답하세요:
{% raw %}
{{
    "points": [
        {{
            "description": "포인트 설명 [Data: Community ({community_id})]",
            "score": 0-100 사이의 중요도
        }}
    ]
}}
{% endraw %}
- 질문과 무관하면 빈 리스트 반환
- 각 포인트는 간결하되 구체적으로 (1-2문장)
- 데이터 참조 반드시 포함
- 추측 금지, 보고서 내용만 활용

JSON만 출력하세요:
"""

def map_community(llm_client, community, query: str):
    """단일 커뮤니티에 대한 Map 단계 실행"""
    
    # Findings 포맷팅
    findings_str = "\n".join([
        f"- {f['summary']}: {f['explanation']}"
        for f in community.get('findings', [])
    ])
    
    prompt = MAP_PROMPT_TEMPLATE.format(
        title=community['title'],
        summary=community['summary'],
        findings=findings_str,
        query=query,
        community_id=community['communityId']
    )
    
    response = llm_client.invoke(prompt)
    
    try:
        # JSON 파싱
        content = response.content
        if "```json" in content:
            content = content.split("```json")[1].split("```")[0]
        
        result = json.loads(content.strip())
        
        # 커뮤니티 정보 추가
        result['communityId'] = community['communityId']
        result['communityTitle'] = community['title']
        
        return result
        
    except json.JSONDecodeError:
        logger.warning(f"Community {community['communityId']} 파싱 실패")
        return {
            'points': [],
            'communityId': community['communityId'],
            'communityTitle': community['title']
        }
```

### 4.3 커뮤니티 필터링 전략

모든 커뮤니티를 Map 단계에 포함할 필요는 없습니다. 질문과 무관하거나 중요도가 낮은 커뮤니티는 제외하여 비용과 시간을 절약할 수 있습니다.

```python
def filter_communities(driver, graph_id: str, rating_threshold: float = 5.0):
    """고품질 커뮤니티만 선택"""
    
    with driver.session() as session:
        result = session.run("""
            MATCH (c:__Community__ {graph_id: $graph_id})
            WHERE c.rating >= $threshold
            RETURN c.communityId AS communityId,
                   c.title AS title,
                   c.summary AS summary,
                   c.rating AS rating,
                   c.findings AS findings
            ORDER BY c.rating DESC
        """, graph_id=graph_id, threshold=rating_threshold)
        
        communities = result.data()
        
        print(f"선택된 커뮤니티: {len(communities)}개 (rating >= {rating_threshold})")
        
        return communities
```

더 정교한 방법은 질문과의 의미적 유사도를 계산하는 것입니다. 커뮤니티 제목과 요약을 임베딩하고, 질문 임베딩과의 코사인 유사도가 높은 Top-K만 선택할 수 있습니다. 그러나 이는 추가 복잡도를 가져오므로, rating 기반 필터링으로 시작하는 것을 권장합니다.

### 4.4 Reduce 단계 구현

```python
REDUCE_PROMPT_TEMPLATE = """당신은 여러 분석가의 보고서를 통합하는 전문가입니다.

여러 커뮤니티에서 수집한 분석 결과를 종합하여 최종 답변을 작성하세요.

# 수집된 분석 결과

{intermediate_results}

# 사용자 질문

{query}

# 최종 답변 요구사항

1. **구조화된 답변**: 마크다운 형식으로 작성
   - 명확한 섹션 구분
   - 핵심 포인트를 논리적으로 조직

2. **중복 제거**: 여러 커뮤니티에서 반복되는 내용은 통합

3. **우선순위**: 중요도 점수가 높은 정보 우선 배치

4. **데이터 참조**: 모든 주장에 출처 명시
   - 형식: [Data: Community (1, 3, 5)]
   - 최대 5개까지만, 더 있으면 +more

5. **객관성**: 근거 없는 추측 금지

6. **완결성**: 질문에 직접적으로 답변

답변:
"""

def reduce_results(llm_client, intermediate_results, query: str):
    """Reduce 단계: 모든 중간 결과 통합"""
    
    # 중간 결과 포맷팅
    results_str = ""
    for i, result in enumerate(intermediate_results, 1):
        if not result['points']:
            continue
        
        results_str += f"\n## 커뮤니티 {result['communityId']}: {result['communityTitle']}\n"
        
        for point in result['points']:
            results_str += f"- [점수: {point['score']}] {point['description']}\n"
    
    prompt = REDUCE_PROMPT_TEMPLATE.format(
        intermediate_results=results_str,
        query=query
    )
    
    response = llm_client.invoke(prompt)
    
    return response.content
```

### 4.5 전체 Global Search 파이프라인

```python
class GlobalSearchEngine:
    """커뮤니티 기반 Global Search 엔진"""
    
    def __init__(self, neo4j_driver, llm_client):
        self.driver = neo4j_driver
        self.llm = llm_client
    
    def search(self, 
               query: str, 
               graph_id: str = "book_002",
               rating_threshold: float = 5.0,
               max_workers: int = 4):
        """Global Search 실행
        
        Args:
            query: 검색 질문
            graph_id: 그래프 ID
            rating_threshold: 커뮤니티 최소 rating
            max_workers: 병렬 처리 워커 수
        
        Returns:
            최종 답변 딕셔너리
        """
        
        logger.info(f"\n=== Global Search 시작 ===")
        logger.info(f"질문: {query}")
        
        # 1. 고품질 커뮤니티 선택
        communities = filter_communities(
            self.driver, 
            graph_id, 
            rating_threshold
        )
        
        if not communities:
            return {
                'query': query,
                'answer': "관련 커뮤니티를 찾을 수 없습니다. rating 임계값을 낮춰보세요.",
                'communities_analyzed': 0
            }
        
        # 2. Map 단계: 병렬 처리
        logger.info(f"\nMap 단계: {len(communities)}개 커뮤니티 분석 중...")
        
        intermediate_results = []
        
        if max_workers > 1:
            # 병렬 처리
            from concurrent.futures import ThreadPoolExecutor, as_completed
            
            with ThreadPoolExecutor(max_workers=max_workers) as executor:
                futures = {
                    executor.submit(map_community, self.llm, c, query): c
                    for c in communities
                }
                
                for future in tqdm(as_completed(futures), total=len(communities)):
                    result = future.result()
                    if result['points']:  # 빈 결과 제외
                        intermediate_results.append(result)
        else:
            # 순차 처리
            for community in tqdm(communities):
                result = map_community(self.llm, community, query)
                if result['points']:
                    intermediate_results.append(result)
        
        logger.info(f"✓ Map 완료: {len(intermediate_results)}개 커뮤니티에서 응답")
        
        if not intermediate_results:
            return {
                'query': query,
                'answer': "질문과 관련된 정보를 커뮤니티에서 찾지 못했습니다.",
                'communities_analyzed': len(communities)
            }
        
        # 3. Reduce 단계
        logger.info("\nReduce 단계: 최종 답변 생성 중...")
        
        final_answer = reduce_results(self.llm, intermediate_results, query)
        
        logger.info("✓ Global Search 완료\n")
        
        return {
            'query': query,
            'answer': final_answer,
            'communities_analyzed': len(communities),
            'communities_responded': len(intermediate_results)
        }

# 사용 예시
global_engine = GlobalSearchEngine(neo4j_driver, deepseek_client)

result = global_engine.search(
    query="우리 조직에서 React를 사용하는 모든 프로젝트의 공통점은?",
    rating_threshold=5.0,
    max_workers=4
)

print(result['answer'])
```

### 4.6 Global Search 성능 특성

**장점**:
- 수천 개 문서에 걸친 패턴 파악 가능
- 추상적이고 넓은 질문에 강함
- 커뮤니티 요약은 재사용 가능 (한 번 생성하면 계속 활용)

**단점**:
- 구체적 사실 검색에는 오버킬
- Map-Reduce 과정에서 LLM 비용 발생 (DeepSeek으로 완화 가능)
- 응답 시간 2-5초 (병렬화로 개선 가능)

**최적 사용 케이스**:
- 경영진 리포트 생성
- 조직 전체 분석
- 장기 트렌드 파악
- 신규 직원 온보딩

---

## 5. Local Search 고도화

### 5.1 기존 Local Search의 한계

현재 우리 시스템의 Local Search는 Elasticsearch에서 벡터 유사도 기반으로 Top-K 청크를 찾고, 해당 청크의 메타데이터를 활용하여 답변을 생성합니다. 이는 빠르고 효율적이지만, 다음과 같은 한계가 있습니다:

1. **그래프 구조 미활용**: Neo4j에 저장된 풍부한 관계 정보를 충분히 활용하지 못함
2. **단일 검색 신호**: 벡터 유사도에만 의존
3. **컨텍스트 부족**: 청크의 broader context (어떤 프로젝트, 어떤 커뮤니티)를 놓침

### 5.2 4계층 컨텍스트 수집

Microsoft GraphRAG의 Local Search는 "4계층 컨텍스트"를 수집합니다:

**Layer 1: Text Chunks** (원본 텍스트)
- 엔티티가 직접 언급된 문서 청크들
- 가장 구체적이고 사실적인 정보

**Layer 2: Community Reports** (커뮤니티 요약)
- 검색된 엔티티들이 속한 커뮤니티의 요약
- 광범위한 맥락 제공

**Layer 3: Relationships** (엔티티 관계)
- 엔티티 간의 명시적 관계 설명
- 연결성과 영향도 파악

**Layer 4: Entities** (엔티티 정보)
- 엔티티 자체의 요약 정보
- 정의와 특성

이 네 계층을 종합하면 질문에 대한 매우 풍부한 컨텍스트를 LLM에 제공할 수 있습니다.

### 5.3 엔티티 벡터 인덱스 구축

Local Search의 시작점은 엔티티 벡터 검색입니다. 각 엔티티의 `summary` 필드를 임베딩하여 Neo4j 벡터 인덱스를 만듭니다.

```python
def create_entity_embeddings(driver, embedder, graph_id: str):
    """엔티티 summary 임베딩 및 벡터 인덱스 생성"""
    
    # 1. 엔티티 summary 수집
    with driver.session() as session:
        result = session.run("""
            MATCH (e:__Entity__ {graph_id: $graph_id})
            WHERE e.summary IS NOT NULL
            RETURN e.name AS name, e.summary AS summary
            ORDER BY e.name
        """, graph_id=graph_id)
        
        entities = result.data()
    
    logger.info(f"임베딩 생성: {len(entities)}개 엔티티")
    
    # 2. 배치 임베딩
    batch_size = 32
    embeddings_data = []
    
    for i in range(0, len(entities), batch_size):
        batch = entities[i:i+batch_size]
        summaries = [e['summary'] for e in batch]
        
        # BGE-M3 임베딩 (1024 dim)
        embeddings = embedder.encode(summaries, normalize=True)
        
        for entity, embedding in zip(batch, embeddings):
            embeddings_data.append({
                'name': entity['name'],
                'embedding': embedding.tolist()
            })
        
        logger.debug(f"진행: {i+len(batch)}/{len(entities)}")
    
    # 3. Neo4j에 벡터 저장
    with driver.session() as session:
        session.run("""
            UNWIND $data AS row
            MATCH (e:__Entity__ {name: row.name, graph_id: $graph_id})
            CALL db.create.setNodeVectorProperty(e, 'embedding', row.embedding)
        """, data=embeddings_data, graph_id=graph_id)
    
    logger.info("✓ 벡터 속성 저장 완료")
    
    # 4. 벡터 인덱스 생성
    with driver.session() as session:
        session.run("""
            CREATE VECTOR INDEX entity_vector_idx IF NOT EXISTS
            FOR (e:__Entity__)
            ON (e.embedding)
            OPTIONS {
                indexConfig: {
                    `vector.dimensions`: 1024,
                    `vector.similarity_function`: 'cosine'
                }
            }
        """)
    
    logger.info("✓ 벡터 인덱스 생성 완료")
```

### 5.4 4계층 컨텍스트 수집 쿼리

이제 핵심 쿼리입니다. 질문 임베딩으로 Top-K 엔티티를 찾고, 그들을 중심으로 4계층 데이터를 수집합니다.

```python
FOUR_LAYER_QUERY = """
// 1. 벡터 검색으로 Top-K 엔티티 찾기
CALL db.index.vector.queryNodes('entity_vector_idx', $k, $embedding)
YIELD node, score
WITH collect(node) AS seedEntities, collect(score) AS scores

// Layer 1: Text Chunks (엔티티가 언급된 청크)
WITH collect {
    UNWIND seedEntities AS entity
    MATCH (entity)<-[:MENTIONS]-(chunk:__Chunk__)
    WITH chunk, count(DISTINCT entity) AS mentionFreq
    RETURN chunk.text AS text
    ORDER BY mentionFreq DESC
    LIMIT $topChunks
} AS chunks,

// Layer 2: Community Reports (소속 커뮤니티 요약)
collect {
    UNWIND seedEntities AS entity
    MATCH (entity)-[:IN_COMMUNITY]->(comm:__Community__)
    WITH comm, comm.rating AS rating
    RETURN comm.summary AS reportText
    ORDER BY rating DESC
    LIMIT $topCommunities
} AS reports,

// Layer 3: Relationships (엔티티 간 관계)
collect {
    UNWIND seedEntities AS entity
    MATCH (entity)-[r:SUMMARIZED_RELATIONSHIP]-(other)
    WHERE other IN seedEntities OR 
          EXISTS((other)-[:IN_COMMUNITY]->(:__Community__)<-[:IN_COMMUNITY]-(entity))
    WITH r, r.strength AS strength
    RETURN r.summary AS relationshipText
    ORDER BY strength DESC
    LIMIT $topRelationships
} AS relationships,

// Layer 4: Entities (엔티티 정보)
collect {
    UNWIND seedEntities AS entity
    RETURN entity.summary AS entityText
} AS entities,

// 엔티티 이름들 (참조용)
collect {
    UNWIND seedEntities AS entity
    RETURN entity.name
} AS entityNames

RETURN {
    chunks: chunks,
    reports: reports,
    relationships: relationships,
    entities: entities,
    entity_names: entityNames
} AS context
"""
```

이 쿼리는 단일 호출로 모든 계층의 데이터를 효율적으로 가져옵니다. 각 계층은 독립적으로 정렬되고 제한됩니다.

### 5.5 Advanced Local Search 엔진

```python
class AdvancedLocalSearchEngine:
    """4계층 컨텍스트 기반 Local Search"""
    
    def __init__(self, neo4j_driver, embedder, llm_client):
        self.driver = neo4j_driver
        self.embedder = embedder
        self.llm = llm_client
    
    def search(self,
               query: str,
               graph_id: str = "book_002",
               k_entities: int = 5,
               top_chunks: int = 5,
               top_communities: int = 3,
               top_relationships: int = 10):
        """Local Search 실행
        
        Args:
            query: 검색 질문
            graph_id: 그래프 ID
            k_entities: 검색할 엔티티 수
            top_chunks: 반환할 청크 수
            top_communities: 반환할 커뮤니티 수
            top_relationships: 반환할 관계 수
        """
        
        logger.info(f"\n=== Local Search 시작 ===")
        logger.info(f"질문: {query}")
        
        # 1. 질문 임베딩
        query_embedding = self.embedder.encode([query], normalize=True)[0]
        
        # 2. 4계층 컨텍스트 수집
        with self.driver.session() as session:
            result = session.run(
                FOUR_LAYER_QUERY,
                embedding=query_embedding.tolist(),
                k=k_entities,
                topChunks=top_chunks,
                topCommunities=top_communities,
                topRelationships=top_relationships
            ).single()
        
        context = result['context']
        
        logger.info(f"수집된 컨텍스트:")
        logger.info(f"  - Chunks: {len(context['chunks'])}개")
        logger.info(f"  - Reports: {len(context['reports'])}개")
        logger.info(f"  - Relationships: {len(context['relationships'])}개")
        logger.info(f"  - Entities: {len(context['entities'])}개")
        
        # 3. 컨텍스트 포맷팅
        context_str = self._format_context(context)
        
        # 4. LLM 답변 생성
        answer = self._generate_answer(query, context_str)
        
        logger.info("✓ Local Search 완료\n")
        
        return {
            'query': query,
            'answer': answer,
            'context_stats': {
                'chunks': len(context['chunks']),
                'reports': len(context['reports']),
                'relationships': len(context['relationships']),
                'entities': len(context['entities'])
            },
            'entities_found': context['entity_names']
        }
    
    def _format_context(self, context: dict) -> str:
        """4계층 컨텍스트를 문자열로 포맷팅"""
        
        formatted = "# 검색 컨텍스트\n\n"
        
        # Layer 1: Chunks
        if context['chunks']:
            formatted += "## 1. 관련 문서 청크\n\n"
            for i, chunk in enumerate(context['chunks'], 1):
                formatted += f"**[Chunk {i}]**\n{chunk}\n\n"
        
        # Layer 2: Community Reports
        if context['reports']:
            formatted += "## 2. 커뮤니티 요약\n\n"
            for i, report in enumerate(context['reports'], 1):
                formatted += f"**[Report {i}]**\n{report}\n\n"
        
        # Layer 3: Relationships
        if context['relationships']:
            formatted += "## 3. 엔티티 관계\n\n"
            for i, rel in enumerate(context['relationships'], 1):
                formatted += f"- **[Rel {i}]**: {rel}\n"
            formatted += "\n"
        
        # Layer 4: Entities
        if context['entities']:
            formatted += "## 4. 엔티티 정보\n\n"
            for i, entity in enumerate(context['entities'], 1):
                formatted += f"- **[Entity {i}]**: {entity}\n"
            formatted += "\n"
        
        return formatted
    
    def _generate_answer(self, query: str, context: str) -> str:
        """LLM을 사용한 최종 답변 생성"""
        
        prompt = f"""당신은 지식 그래프 기반 질의응답 전문가입니다.

제공된 다층 컨텍스트를 바탕으로 사용자 질문에 답변하세요.

{context}

# 사용자 질문

{query}

# 답변 요구사항

1. **정확성**: 제공된 컨텍스트에만 기반하여 답변
2. **데이터 참조**: 모든 주장에 출처 명시
   - 형식: [Data: Chunks (1, 3), Entities (2)]
   - 계층 이름 사용: Chunks, Reports, Relationships, Entities
3. **구조**: 마크다운으로 명확하게 구조화
4. **간결성**: 핵심만 전달, 불필요한 반복 제거
5. **솔직성**: 정보 부족 시 명시

답변:
"""
        
        response = self.llm.invoke(prompt)
        return response.content

# 사용 예시
advanced_local = AdvancedLocalSearchEngine(
    neo4j_driver, 
    bge_m3_embedder, 
    deepseek_client
)

result = advanced_local.search(
    query="김철수가 프로젝트 A에서 어떤 기술을 사용했나?",
    k_entities=5,
    top_chunks=5,
    top_communities=3,
    top_relationships=10
)

print(result['answer'])
```

### 5.6 Local vs Global 자동 선택

실무에서는 질문의 특성을 분석하여 자동으로 적절한 검색 방식을 선택하는 것이 좋습니다.

```python
class HybridSearchRouter:
    """Local/Global 자동 라우팅"""
    
    # 키워드 기반 분류
    GLOBAL_KEYWORDS = [
        '전체', '모든', '모두', '전반', '전사',
        '트렌드', '패턴', '경향', '추세',
        '개요', '요약', '종합', '총괄',
        '비교', '대조', '차이'
    ]
    
    LOCAL_KEYWORDS = [
        '누가', '언제', '어디서', '어떻게', '왜',
        '구체적', '상세', '자세히', '정확히',
        '특정', '개별', '하나의'
    ]
    
    def __init__(self, local_engine, global_engine):
        self.local = local_engine
        self.global_search = global_engine
    
    def search(self, query: str, mode: str = 'auto'):
        """통합 검색
        
        Args:
            query: 검색 질문
            mode: 'local', 'global', 'auto'
        """
        
        if mode == 'auto':
            mode = self._classify_query(query)
            logger.info(f"자동 선택: {mode.upper()} search")
        
        if mode == 'global':
            return self.global_search.search(query)
        else:
            return self.local.search(query)
    
    def _classify_query(self, query: str) -> str:
        """질문 특성 분석"""
        
        query_lower = query.lower()
        
        # Global 신호 점수
        global_score = sum(
            1 for kw in self.GLOBAL_KEYWORDS 
            if kw in query_lower
        )
        
        # Local 신호 점수
        local_score = sum(
            1 for kw in self.LOCAL_KEYWORDS
            if kw in query_lower
        )
        
        # 질문 길이 (짧으면 Local 선호)
        length_signal = -1 if len(query) < 30 else 1
        
        # 최종 결정
        if global_score > local_score + length_signal:
            return 'global'
        else:
            return 'local'  # 기본값

# 사용
router = HybridSearchRouter(advanced_local, global_engine)

# 자동 선택
result1 = router.search("우리 팀의 모든 프로젝트 기술 스택은?")  # → global
result2 = router.search("김철수 연락처는?")  # → local
```

---

## 6. 성능 최적화 및 비용 관리

### 6.1 비용 분석

Community-based Global Search를 추가할 때의 비용 증가를 정확히 파악해야 합니다.

**일회성 비용** (초기 구축):
```
커뮤니티 요약 생성:
- 40개 커뮤니티 × 2,300 tokens/커뮤니티 = 92,000 tokens
- DeepSeek 비용: $0.01
- Claude/GPT-4 비용: $0.20-0.30 (20-30배 비싼)

엔티티 임베딩:
- 5,000개 엔티티 × 1024 dim
- BGE-M3 로컬 실행: 무료 (CPU 시간만)
- 소요 시간: ~10분
```

**쿼리당 비용** (운영):
```
Global Search:
- Map: 40개 커뮤니티 × 1,000 tokens = 40,000 tokens input
- Map: 40개 커뮤니티 × 300 tokens = 12,000 tokens output
- Reduce: 5,000 tokens input + 1,000 tokens output
- DeepSeek 총 비용: $0.015/쿼리
- 100 queries/day → $1.50/day = $45/month

Local Search (기존과 동일):
- DeepSeek 비용: $0.003/쿼리
- 1000 queries/day → $3/day = $90/month
```

**절감 방법**:
1. **커뮤니티 캐싱**: 동일 커뮤니티 Map 결과 재사용
2. **배치 처리**: 동시 여러 질문 처리 시 토큰 공유
3. **Lazy Loading**: rating 낮은 커뮤니티는 필요시만 분석

### 6.2 메모리 영향

16GB RAM 환경에서 추가 메모리 사용:

```
엔티티 벡터 인덱스:
- 5,000개 × 1024 dim × 4 bytes = 20MB
- Neo4j 페이지캐시 증가: ~50MB

커뮤니티 노드:
- 40개 × 평균 5KB = 200KB (무시 가능)

총 증가: ~70MB (전체의 0.4%, 여유 있음)
```

### 6.3 응답 시간 최적화

**병렬화**:
```python
# Map 단계 병렬 처리로 2-3배 속도 향상
with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [
        executor.submit(map_community, llm, c, query)
        for c in communities
    ]
```

**사전 필터링**:
```python
# 질문과 커뮤니티 제목의 키워드 매칭으로 50% 커뮤니티 제거
def quick_filter(query, communities):
    query_words = set(query.lower().split())
    
    filtered = []
    for c in communities:
        title_words = set(c['title'].lower().split())
        if query_words & title_words:  # 교집합 존재
            filtered.append(c)
    
    return filtered
```

**결과 캐싱**:
```python
# 동일 질문 반복 시 즉시 응답
from functools import lru_cache

def cached_global_search(query_hash):
    # 실제 검색
    pass
```

### 6.4 품질 vs 비용 트레이드오프

| 전략 | 품질 | 비용 | 속도 |
|------|------|------|------|
| **All Communities** | ⭐⭐⭐⭐⭐ | 💰💰💰 | 🐌 |
| **High Rating Only** | ⭐⭐⭐⭐ | 💰💰 | 🐇 |
| **Keyword Filtered** | ⭐⭐⭐ | 💰 | 🚀 |

권장: High Rating (>= 5.0) + Keyword Pre-filter 조합

---

## 7. 실전 적용 가이드

### 7.1 단계별 구현 로드맵

**Week 1-2: 커뮤니티 탐지 및 요약**
```
Day 1-3: Neo4j GDS 설치 및 Louvain 실행
Day 4-5: 커뮤니티 데이터 수집 쿼리 개발
Day 6-10: LLM 요약 생성 및 저장
Day 11-14: 품질 검증 및 개선
```

**Week 3-4: Global Search 구현**
```
Day 1-5: Map 단계 구현 및 테스트
Day 6-10: Reduce 단계 구현 및 통합
Day 11-14: 병렬화 및 최적화
```

**Week 5-6: Local Search 고도화**
```
Day 1-7: 엔티티 임베딩 및 벡터 인덱스
Day 8-14: 4계층 쿼리 개발 및 통합
```

**Week 7-8: 통합 및 배포**
```
Day 1-7: Hybrid Router 구현
Day 8-14: API 통합 및 문서화
```

### 7.2 검증 체크리스트

**커뮤니티 품질**:
- [ ] 모듈성 점수 > 0.3
- [ ] 평균 커뮤니티 크기 5-15개
- [ ] 싱글톤 비율 < 20%
- [ ] 모든 커뮤니티에 요약 존재

**Global Search 품질**:
- [ ] 넓은 질문에 포괄적 답변
- [ ] 데이터 참조 정확성
- [ ] 응답 시간 < 5초
- [ ] 비용 < $0.02/쿼리

**Local Search 품질**:
- [ ] 구체적 질문에 정확한 답변
- [ ] 4계층 모두 활용
- [ ] 응답 시간 < 2초
- [ ] 엔티티 검색 정확도 > 80%

### 7.3 운영 가이드

**일일 모니터링**:
```python
def daily_health_check():
    """일일 시스템 체크"""
    
    # 1. 커뮤니티 상태
    with driver.session() as session:
        result = session.run("""
            MATCH (c:__Community__)
            RETURN count(c) AS total,
                   avg(c.rating) AS avg_rating
        """).single()
        
        print(f"커뮤니티: {result['total']}개 (평균 rating: {result['avg_rating']:.2f})")
    
    # 2. 엔티티 벡터 인덱스
    with driver.session() as session:
        result = session.run("""
            MATCH (e:__Entity__)
            WHERE e.embedding IS NOT NULL
            RETURN count(e) AS indexed
        """).single()
        
        print(f"인덱싱된 엔티티: {result['indexed']}개")
    
    # 3. 최근 검색 성능
    # (로그에서 수집)
```

**주간 재구축**:
```python
def weekly_rebuild():
    """주간 커뮤니티 재탐지 및 요약 업데이트"""
    
    # 새로운 문서가 많이 추가되었을 때만
    new_docs_count = check_new_documents()
    
    if new_docs_count > 100:
        logger.info("주간 재구축 시작...")
        
        # 1. Louvain 재실행
        detect_communities(driver)
        
        # 2. 변경된 커뮤니티만 요약 재생성
        regenerate_changed_summaries()
        
        # 3. 엔티티 임베딩 업데이트
        update_entity_embeddings()
```

### 7.4 트러블슈팅

**문제: "커뮤니티가 너무 많이 생성됨 (100개 이상)"**
- 원인: 그래프가 지나치게 파편화
- 해결: Louvain resolution 파라미터 조정
```python
session.run("""
    CALL gds.louvain.write('entity_graph', {
        writeProperty: 'louvain',
        resolution: 1.5  // 기본 1.0, 높일수록 큰 커뮤니티
    })
""")
```

**문제: "Global Search가 관련 없는 답변 생성"**
- 원인: rating 임계값이 너무 낮아 노이즈 커뮤니티 포함
- 해결: rating_threshold를 6.0-7.0으로 상향

**문제: "Local Search가 엔티티를 못 찾음"**
- 원인: 엔티티 summary가 빈 값이거나 임베딩 미생성
- 해결: 
```python
# 빈 summary 확인
session.run("""
    MATCH (e:__Entity__)
    WHERE e.summary IS NULL OR e.summary = ''
    RETURN count(e)
""")

# 재생성
regenerate_entity_summaries()
```

### 7.5 성공 지표

**정량 지표**:
- Global Search 사용률 > 20%
- Local Search 정확도 > 85%
- 평균 응답 시간 < 3초
- 월 LLM 비용 증가 < $50

**정성 지표**:
- 경영진 만족도 조사
- 신규 직원 온보딩 효율성
- "답을 찾지 못함" 비율 감소

---

## 결론

Community-based Global Search는 기존 GraphRAG-Neo4j 시스템의 강력한 확장입니다. 구체적 사실 검색에 강한 Local Search와 넓은 통찰 제공에 강한 Global Search를 결합하면, 모든 유형의 질문에 효과적으로 대응할 수 있습니다.

핵심은 **점진적 적용**입니다. 먼저 MVP 시스템으로 Local Search의 가치를 입증하고, 사용자 피드백을 바탕으로 Global Search의 필요성이 확인되면 추가하는 것이 현명합니다. 

DeepSeek과 BGE-M3의 조합은 이 고급 기능을 매우 경제적으로 구현할 수 있게 해줍니다. 초기 구축 비용은 $0.01 수준이고, 운영 비용도 쿼리당 $0.015로 실용적입니다.

16GB RAM 환경에서도 충분히 구현 가능하며, 메모리 증가는 70MB 미만으로 무시할 수준입니다. 병렬 처리와 캐싱을 활용하면 응답 시간도 5초 이내로 유지할 수 있습니다.

이 가이드가 여러분의 지식 검색 시스템을 한 단계 업그레이드하는 데 도움이 되기를 바랍니다.

**작성 일자: 2026-01-13**
