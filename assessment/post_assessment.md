# 사후평가 — Advanced RAG 엔지니어링 구축 전략 (6/18 과목)

> 본 평가는 전 과목 사후평가의 일부로, 본 과목 교안(모듈1·모듈2 슬라이드, lab0~lab6 + b-lab)과 함께 제출된다.
> 총 5문항. 모듈1(RAG 평가·운영) 2문항 + 모듈2(검색 고도화 / SOTA / Graph RAG) 3문항. 객관식 4지선다 3문항 + 시나리오·적용형 2문항(Q4·Q5).
> Q1~Q3은 기존 문항을 유지하고, Q4·Q5는 보강 패스 v2(SOTA·운영)로 신규 추가했다.

## 출제 범위 요약

| 문항 | 출제 모듈 | 토픽 | 유형 | 측정 학습목표 | 출제근거 |
|---|---|---|---|---|---|
| Q1 | 모듈1 — RAG 평가 | Faithfulness vs Answer Relevancy(생성 지표 해석) | 객관식·해석형 | 검색/생성 지표의 의미를 구분하고 약점을 진단 | module1-rag-evaluation.md / lab1 |
| Q2 | 모듈2 전반 — 검색 고도화 | Dense vs Sparse 실패 차이 → Hybrid(RRF) 처방 | 시나리오·적용형 | 검색 증상을 진단하고 알맞은 검색 기법 선택 | module2-advanced-graph-rag.md / lab3 |
| Q3 | 모듈2 후반 — Graph RAG | 벡터 RAG vs Graph RAG 적용 판단, Local vs Global search | 객관식·적용형 | 질문 유형에 따라 그래프 검색의 적합성·라우팅 판단 | module2-advanced-graph-rag.md / lab5·lab6 |
| Q4 | 모듈1 · 운영(비용 최적화) | prompt caching vs Batches API 구분·적용 | 시나리오·적용형 | 대량 judge 채점/반복 RAG 호출 비용을 어떤 기법으로 줄일지 판단 | module1-rag-evaluation.md / lab1 심화(11·12단계) |
| Q5 | 모듈2 · SOTA(검색기법 선택) | 증상별 처방: Contextual Retrieval / Hybrid / Graph RAG | 시나리오·적용형 | 검색 품질 증상을 진단하고 최신 검색기법을 올바르게 처방 | module2-advanced-graph-rag.md / lab2·lab3·lab5 |

---

# 섹션 A · 정답·해설 포함본 (강사용)

### Q1. (모듈1) 어떤 RAG 시스템을 평가했더니, 답변이 **검색된 근거 문서에 충실(faithful)** 하여 환각은 거의 없는데, 정작 **사용자의 질문과는 동떨어진 내용**을 답하는 경우가 잦았다. 이 증상을 가장 정확히 잡아내는 생성 평가지표는 무엇이며, 그 이유로 옳은 것은?

1) **Faithfulness** — 답이 근거에 기반하는지를 보므로 동문서답을 직접 탐지한다
2) **Answer Relevancy** — 답이 원래 질문에 실제로 답했는지를 보므로 동문서답을 탐지한다
3) **Context Recall** — 검색 context가 정답 근거를 충분히 담았는지를 보므로 동문서답을 탐지한다
4) **BLEU/ROUGE** — 정답 문장과의 표면 단어 일치를 보므로 동문서답을 탐지한다

- **정답: 2**
- **해설:** Faithfulness(충실성)는 "답이 검색 context에 근거하는가"(환각 여부)를 보고, Answer Relevancy는 "답이 원래 질문에 실제로 답했는가"(동문서답 여부)를 본다. 두 지표는 직교한다 — 근거에 충실하면서도 질문과 무관할 수 있다(근거는 인용했지만 딴소리). 문제의 증상은 "충실하나 무관"이므로 정답은 Answer Relevancy(2). (1)은 이미 충실하다고 했으므로 이 증상을 잡지 못한다. (3) Context Recall은 생성이 아니라 검색 context의 충분성을 보는 지표다. (4) BLEU/ROUGE는 표면 단어 일치만 봐 RAG 답변 품질 평가에 부적합하다.
- **출제근거:** 모듈1 슬라이드 `module1-rag-evaluation.md` — "생성 평가지표", "Faithfulness vs Answer Relevancy — 헷갈리지 말 것"(두 지표가 직교) / 실습 `lab1_rag_evaluation.ipynb`(judge로 faithfulness·answer_relevancy 채점)
- **평가 학습목표:** 생성 평가지표의 의미를 구분하고, 증상으로부터 어떤 지표가 그 실패를 잡는지 판단한다.

---

### Q2. (모듈2) 사내 검색 시스템에서 "삼성전자 GAAP 영업이익"처럼 **고유명사·약어가 정확히 들어간 질의**는 잘 찾지만, "실적이 좋아진 이유" 같은 **의미 기반 동의어 질의**에서는 누락이 잦다 — 혹은 그 반대 패턴이 나타난다. 한 쪽 방식만으로는 두 유형을 모두 만족시키기 어렵다. 가장 적절한 개선책과 그 동작 설명으로 옳은 것은?

1) cross-encoder Reranking만 단독 적용한다 — 1차 검색 없이 전체 코퍼스를 재정렬하면 모든 질의 유형이 해결된다
2) Dense(임베딩) 검색의 top-k만 늘린다 — k를 키우면 키워드 매칭 약점이 자동으로 보완된다
3) **Dense와 Sparse(BM25)를 함께 돌려 RRF로 융합하는 Hybrid search를 적용한다** — 두 방식이 서로 다른 질의에서 실패하므로 합집합으로 보완된다
4) 메타데이터 필터(Self-Query)로 연도·문서유형 조건만 강화한다 — 필터 조건을 늘리면 의미·키워드 검색 약점이 함께 사라진다

- **정답: 3**
- **해설:** Dense(밀집)는 동의어·문맥에 강하나 정확 매칭에 약하고, Sparse(BM25/SPLADE)는 정확 키워드·희귀어·고유명사에 강하나 의미 이해가 없다. 두 방식은 "서로 다른 질문에서" 실패하므로 합치면 약점이 보완된다 — 이것이 Hybrid이며, RRF(`score = Σ 1/(k+rank)`, k≈60)는 점수 스케일을 맞출 필요 없이 순위만으로 융합해 강건하다. (1) cross-encoder는 느려 전체 코퍼스에 직접 돌릴 수 없고, 1차 검색으로 후보를 좁힌 뒤 정밀 재정렬하는 2단계 기법이지 dense/sparse의 의미-키워드 보완 문제를 푸는 도구가 아니다. (2) dense의 k를 늘려도 정확 키워드 매칭 약점 자체는 그대로다. (4) 메타필터는 조건 필터링으로 노이즈를 줄일 뿐, dense/sparse의 의미·키워드 검색 약점을 보완하지 못한다.
- **출제근거:** 모듈2 슬라이드 `module2-advanced-graph-rag.md` — "Dense vs Sparse — 다른 실패를 한다", "Hybrid Search — RRF로 융합" / 실습 `lab3_hybrid_search.ipynb`(dense-only/BM25-only/Hybrid를 키워드형+의미형 쿼리셋으로 비교, RRF 직접 구현)
- **평가 학습목표:** 검색 실패 증상(키워드형 vs 의미형)을 진단하고, Dense/Sparse의 상보성과 Hybrid(RRF)의 동작 원리에 근거해 알맞은 기법을 선택한다.

---

### Q3. (모듈2 · Graph RAG 적용형) 금융 도메인 RAG에 다음 두 질문을 던졌다.
> (가) "A사가 투자한 회사의 CEO는 누구인가?"
> (나) "이번 분기 공시 전체에서 드러나는 핵심 리스크는 무엇인가?"
> 기존 벡터 RAG는 (가)에서 두 청크가 따로 검색돼 연결을 못 했고, (나)에서 단일 청크로는 전체를 답하지 못했다. Graph RAG로 전환할 때, 각 질문에 적합한 검색 방식 라우팅으로 가장 옳은 것은?

1) (가)는 Global search, (나)는 Local search — (가)가 여러 엔티티를 거치므로 전역 요약이 필요하다
2) **(가)는 Local search, (나)는 Global search — (가)는 특정 엔티티 중심의 관계·이웃 탐색(multi-hop)이고, (나)는 커뮤니티 요약을 map-reduce로 종합하는 전역 질문이다**
3) (가)·(나) 모두 Local search — 그래프 질의는 항상 엔티티 이웃 탐색으로 수렴한다
4) (가)·(나) 모두 벡터 RAG의 top-k를 키우면 해결되므로 그래프 전환이 불필요하다

- **정답: 2**
- **해설:** 벡터 RAG는 청크를 독립적으로 검색해 청크 간 "관계"에 약하다. (가)는 "A사 → 투자한 회사 → 그 회사의 CEO"로 이어지는 multi-hop 관계 질문으로, 특정 엔티티에서 출발해 이웃·관계를 따라가는 **Local search**(점, 엔티티 중심)에 적합하다. (나)는 "전체에서 드러나는 핵심 리스크" 같은 전역·주제형 질문으로, 커뮤니티 요약을 map-reduce로 종합하는 **Global search**(면)에 적합하다. (1)은 Local/Global을 반대로 라우팅했다. (3) 그래프 질의는 질문 유형에 따라 Local/Global로 라우팅하며 항상 Local로 수렴하지 않는다. (4) top-k를 키워도 청크 간 관계를 연결하거나 전역을 종합하는 능력은 생기지 않는다 — 벡터와 그래프는 보완재다.
- **출제근거:** 모듈2 슬라이드 `module2-advanced-graph-rag.md` — "왜 그래프인가 — 벡터 RAG의 한계"(multi-hop·전역 질문 실패 예시 그대로), "Local vs Global search"(질문 유형별 라우팅) / 실습 `lab5_graphrag_finance.ipynb`(Local·Global search 직접 구현, 벡터 vs 그래프 비교), `lab6_graphrag_legal.ipynb`(인용망 Local/Global 확장)
- **평가 학습목표:** 질문 유형(관계·multi-hop vs 전역·주제형)을 기준으로 벡터 RAG와 Graph RAG의 적합성을 판단하고, Local/Global search로 올바르게 라우팅한다.

---

### Q4. (모듈1 · 운영 · 비용 최적화 적용형) 운영 중인 RAG의 비용을 줄이려 한다. 두 가지 부담이 있다.
> (상황 ①) 매일 밤, 고정된 채점 rubric으로 골든셋 수백 건을 LLM judge로 채점한다 — **실시간 응답은 필요 없고**, 같은 rubric/지침이 매 호출마다 반복된다.
> (상황 ②) 사용자에게 **실시간으로 응답**하는 질의에서, 매번 같은 시스템 프롬프트와 (자주 재사용되는) 검색 context가 프롬프트 앞부분에 반복 투입된다.
> 각 상황에 가장 적합한 비용 최적화 기법 조합으로 옳은 것은?

1) ① prompt caching, ② Batches API — 대량 작업은 캐시로, 실시간 작업은 Batches로 처리한다
2) **① Batches API, ② prompt caching — ①은 실시간이 불필요한 대량·비동기 채점이라 묶음 50% 할인(Batches)이 맞고, ②는 반복되는 고정 prefix(rubric·context)를 재사용하는 캐시(읽기 ~0.1x)가 맞다**
3) ①·② 모두 Batches API — 호출이 많으면 무조건 Batches로 묶어 50% 할인받는 것이 항상 최선이다
4) ①·② 모두 모델을 Haiku 4.5로만 교체한다 — 모델만 저가로 바꾸면 두 비용 문제가 모두 사라진다

- **정답: 2**
- **해설:** 두 기법은 해결하는 문제가 다르다. **Batches API**는 모든 토큰을 50% 할인하지만 **비동기(최대 24h)** 라 실시간 응답에는 부적합하고, 골든셋 대량 채점·야간 회귀평가처럼 "급하지 않은 대량 작업"에 이상적이다 → 상황 ①. **prompt caching**은 프롬프트의 **안 바뀌는 앞부분(prefix)**, 즉 고정 rubric·시스템 프롬프트·반복 재사용되는 context에 `cache_control`을 걸어 두 번째 호출부터 그 부분을 캐시 읽기(~0.1x, 최대 90%↓)로 처리한다 → 상황 ②. 따라서 ① Batches, ② caching인 (2)가 정답이다. (1)은 두 기법을 정반대로 배치했다 — 캐시는 비동기 대량 작업의 핵심 해법이 아니고(상황 ①은 반복 prefix가 아니라 "대량을 한 번에"가 관건), Batches는 비동기라 ②의 실시간 응답에 쓸 수 없다. (3) Batches는 비동기라 실시간 질의(②)에는 못 쓰므로 "무조건 Batches"는 틀리다. (4) 모델 교체(Haiku)는 정확도 트레이드오프가 있는 부분적 수단일 뿐, caching/Batches가 푸는 "반복 prefix·비동기 대량" 구조 자체를 해결하지 못한다(caching·Batches와 함께 쓸 수는 있으나 그것만으로 "끝"이 아니다).
- **출제근거:** 모듈1 슬라이드 `module1-rag-evaluation.md` — "Production RAG — 배포 후가 진짜 시작"(골든셋 회귀 모니터링·비용 최적화 예고), "💰 평가 비용을 줄이는 두 손잡이 — caching · Batches"(caching=반복 prefix 재사용 90%↓ / Batches=비동기 대량 50%↓, "둘은 함께 쓸 수 있다") / 실습 `lab1_rag_evaluation.ipynb` 심화 11단계(prompt caching judge — 고정 rubric·context에 `cache_control: ephemeral`)·12단계(Batches API로 골든셋 대량 비동기 채점 50% 절감)
- **평가 학습목표:** prompt caching(반복 고정 prefix 재사용·실시간 가능)과 Batches API(비동기 대량·50% 할인·실시간 불가)의 적용 조건을 구분하고, 비용 증상에 맞는 기법을 처방한다.

---

### Q5. (모듈2 · SOTA 검색기법 처방 적용형) 한 RAG 시스템에서 세 가지 서로 다른 검색 품질 문제가 관측됐다.
> (증상 A) 긴 보고서를 작은 청크로 쪼갰더니, 청크만 보면 **"이게 어느 회사·어느 연도 얘기인지" 문맥이 사라져** 검색이 자주 빗나간다.
> (증상 B) **정확한 고유명사·약어 질의**와 **의미·동의어 질의**가 섞여 들어오는데, 한 검색 방식만으로는 양쪽을 다 못 맞춘다.
> (증상 C) "A사가 인수한 회사의 모회사는?"처럼 **여러 사실을 관계로 이어가야(multi-hop)** 풀리는 질문에서 청크들이 따로 검색돼 연결이 안 된다.
> 각 증상에 가장 적합한 처방의 조합으로 옳은 것은?

1) A → Contextual Retrieval, B → Hybrid search(RRF), C → Graph RAG
2) A → Graph RAG, B → Contextual Retrieval, C → Hybrid search(RRF)
3) A·B·C 모두 → 벡터 검색의 top-k를 크게 늘리면 한 번에 해결된다
4) A → Hybrid search, B → Graph RAG, C → Contextual Retrieval

- **정답: 1**
- **해설:** 세 증상은 각각 다른 기법이 정확히 겨냥한다. **(증상 A · 청크 문맥 소실)** → **Contextual Retrieval**: 청크를 임베딩하기 전에 Claude로 "이 조각이 어느 문서의 무엇인지" 문맥 한두 줄을 생성해 앞에 prepend → 회사·연도가 청크 안에 명시돼 검색 실패가 준다(반복 색인 비용은 prompt caching으로 절감). **(증상 B · 키워드+의미 혼재)** → **Hybrid search(RRF)**: Dense(의미)와 Sparse/BM25(정확 키워드)를 RRF로 융합해 두 질의 유형의 약점을 합집합으로 보완(Q2와 동일 원리). **(증상 C · multi-hop 관계)** → **Graph RAG**: 엔티티·관계를 그래프로 만들어 "A사 → 인수한 회사 → 모회사"를 이어가며 추적(Local search). 따라서 정답은 (1). (2)·(4)는 처방을 뒤섞었다 — 특히 **Contextual Retrieval을 multi-hop 관계 질문(C)에 쓰면** 청크에 문맥을 붙일 뿐 청크 간 관계를 연결하지 못해 한계가 있다(C는 Graph RAG가 적합). (3) top-k를 키워도 문맥 소실(A)·키워드 약점(B)·관계 연결(C) 중 어느 것도 구조적으로 해결되지 않는다 — 후보를 더 많이 가져올 뿐 검색 실패 원인 자체는 그대로다.
- **출제근거:** 모듈2 슬라이드 `module2-advanced-graph-rag.md` — "Contextual Retrieval — 청크에 '문맥 한 줄'을 입히기"·"Contextual Retrieval 파이프라인 + contextual BM25"(증상 A), "Dense vs Sparse — 다른 실패를 한다"·"Hybrid Search — RRF로 융합"(증상 B), "왜 그래프인가 — 벡터 RAG의 한계(multi-hop)"·"Local vs Global search"(증상 C) / 실습 `lab2_metadata_table.ipynb` 8단계(Contextual Retrieval — 문맥 한 줄 prepend, 검색 실패율 전/후 비교), `lab3_hybrid_search.ipynb`(RRF Hybrid), `lab5_graphrag_finance.ipynb`(multi-hop Local search)
- **평가 학습목표:** 검색 품질 증상(청크 문맥 소실 / 키워드·의미 혼재 / multi-hop 관계)을 진단하고, Contextual Retrieval·Hybrid(RRF)·Graph RAG를 증상에 맞게 처방하며, 각 기법이 풀 수 없는 한계(예: Contextual Retrieval은 관계 연결 불가)를 구분한다.

---

# 섹션 B · 응시자용 (정답 제외)

> 각 문항당 정답은 1개입니다. 가장 적절한 것을 하나만 고르세요. (총 5문항)

### Q1.
어떤 RAG 시스템을 평가했더니, 답변이 **검색된 근거 문서에 충실(faithful)** 하여 환각은 거의 없는데, 정작 **사용자의 질문과는 동떨어진 내용**을 답하는 경우가 잦았다. 이 증상을 가장 정확히 잡아내는 생성 평가지표는 무엇이며, 그 이유로 옳은 것은?

1) Faithfulness — 답이 근거에 기반하는지를 보므로 동문서답을 직접 탐지한다
2) Answer Relevancy — 답이 원래 질문에 실제로 답했는지를 보므로 동문서답을 탐지한다
3) Context Recall — 검색 context가 정답 근거를 충분히 담았는지를 보므로 동문서답을 탐지한다
4) BLEU/ROUGE — 정답 문장과의 표면 단어 일치를 보므로 동문서답을 탐지한다

---

### Q2.
사내 검색 시스템에서 "삼성전자 GAAP 영업이익"처럼 **고유명사·약어가 정확히 들어간 질의**는 잘 찾지만, "실적이 좋아진 이유" 같은 **의미 기반 동의어 질의**에서는 누락이 잦다 — 혹은 그 반대 패턴이 나타난다. 한 쪽 방식만으로는 두 유형을 모두 만족시키기 어렵다. 가장 적절한 개선책과 그 동작 설명으로 옳은 것은?

1) cross-encoder Reranking만 단독 적용한다 — 1차 검색 없이 전체 코퍼스를 재정렬하면 모든 질의 유형이 해결된다
2) Dense(임베딩) 검색의 top-k만 늘린다 — k를 키우면 키워드 매칭 약점이 자동으로 보완된다
3) Dense와 Sparse(BM25)를 함께 돌려 RRF로 융합하는 Hybrid search를 적용한다 — 두 방식이 서로 다른 질의에서 실패하므로 합집합으로 보완된다
4) 메타데이터 필터(Self-Query)로 연도·문서유형 조건만 강화한다 — 필터 조건을 늘리면 의미·키워드 검색 약점이 함께 사라진다

---

### Q3.
금융 도메인 RAG에 다음 두 질문을 던졌다.
> (가) "A사가 투자한 회사의 CEO는 누구인가?"
> (나) "이번 분기 공시 전체에서 드러나는 핵심 리스크는 무엇인가?"

기존 벡터 RAG는 (가)에서 두 청크가 따로 검색돼 연결을 못 했고, (나)에서 단일 청크로는 전체를 답하지 못했다. Graph RAG로 전환할 때, 각 질문에 적합한 검색 방식 라우팅으로 가장 옳은 것은?

1) (가)는 Global search, (나)는 Local search — (가)가 여러 엔티티를 거치므로 전역 요약이 필요하다
2) (가)는 Local search, (나)는 Global search — (가)는 특정 엔티티 중심의 관계·이웃 탐색(multi-hop)이고, (나)는 커뮤니티 요약을 map-reduce로 종합하는 전역 질문이다
3) (가)·(나) 모두 Local search — 그래프 질의는 항상 엔티티 이웃 탐색으로 수렴한다
4) (가)·(나) 모두 벡터 RAG의 top-k를 키우면 해결되므로 그래프 전환이 불필요하다

---

### Q4.
운영 중인 RAG의 비용을 줄이려 한다. 두 가지 부담이 있다.
> (상황 ①) 매일 밤, 고정된 채점 rubric으로 골든셋 수백 건을 LLM judge로 채점한다 — **실시간 응답은 필요 없고**, 같은 rubric/지침이 매 호출마다 반복된다.
> (상황 ②) 사용자에게 **실시간으로 응답**하는 질의에서, 매번 같은 시스템 프롬프트와 (자주 재사용되는) 검색 context가 프롬프트 앞부분에 반복 투입된다.

각 상황에 가장 적합한 비용 최적화 기법 조합으로 옳은 것은?

1) ① prompt caching, ② Batches API — 대량 작업은 캐시로, 실시간 작업은 Batches로 처리한다
2) ① Batches API, ② prompt caching — ①은 실시간이 불필요한 대량·비동기 채점이라 묶음 50% 할인(Batches)이 맞고, ②는 반복되는 고정 prefix(rubric·context)를 재사용하는 캐시(읽기 ~0.1x)가 맞다
3) ①·② 모두 Batches API — 호출이 많으면 무조건 Batches로 묶어 50% 할인받는 것이 항상 최선이다
4) ①·② 모두 모델을 Haiku 4.5로만 교체한다 — 모델만 저가로 바꾸면 두 비용 문제가 모두 사라진다

---

### Q5.
한 RAG 시스템에서 세 가지 서로 다른 검색 품질 문제가 관측됐다.
> (증상 A) 긴 보고서를 작은 청크로 쪼갰더니, 청크만 보면 **"이게 어느 회사·어느 연도 얘기인지" 문맥이 사라져** 검색이 자주 빗나간다.
> (증상 B) **정확한 고유명사·약어 질의**와 **의미·동의어 질의**가 섞여 들어오는데, 한 검색 방식만으로는 양쪽을 다 못 맞춘다.
> (증상 C) "A사가 인수한 회사의 모회사는?"처럼 **여러 사실을 관계로 이어가야(multi-hop)** 풀리는 질문에서 청크들이 따로 검색돼 연결이 안 된다.

각 증상에 가장 적합한 처방의 조합으로 옳은 것은?

1) A → Contextual Retrieval, B → Hybrid search(RRF), C → Graph RAG
2) A → Graph RAG, B → Contextual Retrieval, C → Hybrid search(RRF)
3) A·B·C 모두 → 벡터 검색의 top-k를 크게 늘리면 한 번에 해결된다
4) A → Hybrid search, B → Graph RAG, C → Contextual Retrieval
