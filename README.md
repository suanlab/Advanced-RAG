# Advanced RAG 엔지니어링 구축 전략

> 신한카드 Agentic AI 엔지니어링 과정 · 8H 집중 과목 · **이수안 교수 (이수안컴퓨터연구소 / SuanLab)**
>
> RAG를 **"감으로 고치는 것"에서 "측정으로 고치는 것"** 으로 — 평가·개선 루프부터 Hybrid·Reranking·Graph RAG·Agentic RAG·운영까지, 모든 LLM 단계는 **Claude API** 기반으로 직접 구현합니다. 데이터는 전부 합성·내장(민감/저작권 데이터 없음).

## 🖥️ 강의 슬라이드 (온라인 열람)

- **모듈 1 — RAG 평가 및 개선하기** — [슬라이드 열기](https://suanlab.github.io/Advanced-RAG/slides/module1-rag-evaluation.html)
- **모듈 2 — Advanced RAG & Graph RAG** — [슬라이드 열기](https://suanlab.github.io/Advanced-RAG/slides/module2-advanced-graph-rag.html)

> 발표 뷰어: `←/→`·Space 이동 · `o` 목차 · `n` 발표자 노트 · `f` 풀스크린 · Print(PDF 저장).

## 🧪 Colab 실습 (8개 통합 노트북)

각 노트북은 **첫 셀의 '환경 셋업'만 1회 실행**하면 됩니다(Claude API 키 입력 1회). 모든 LLM 단계 = Claude(anthropic SDK), 임베딩·BM25·그래프 등 불가피한 부분만 타 라이브러리(사유 주석 포함).

| # | 실습 | 내용 | 열기 |
|---|---|---|---|
| 1 | lab1 RAG 평가 | RAG 평가 — 베이스라인·검색/생성 지표·LLM-as-a-judge·편향 실험 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab1%20RAG%20%ED%8F%89%EA%B0%80.ipynb) |
| 2 | lab2 메타데이터·테이블 & Contextual Retrieval | 메타필터·Self-Query·테이블 직렬화·Contextual Retrieval | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab2%20%EB%A9%94%ED%83%80%EB%8D%B0%EC%9D%B4%ED%84%B0%C2%B7%ED%85%8C%EC%9D%B4%EB%B8%94%20%26%20Contextual%20Retrieval.ipynb) |
| 3 | lab3 Hybrid 검색 & 쿼리 재작성·HyDE | BM25+Dense Hybrid(RRF)·쿼리 재작성·멀티쿼리·HyDE | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab3%20Hybrid%20%EA%B2%80%EC%83%89%20%26%20%EC%BF%BC%EB%A6%AC%20%EC%9E%AC%EC%9E%91%EC%84%B1%C2%B7HyDE.ipynb) |
| 4 | lab4 Reranking | Cross-Encoder 2단계 리랭킹·top-N/k 트레이드오프 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab4%20Reranking.ipynb) |
| 5 | lab5 Graph RAG | 지식그래프 구축·Local/Global search·중심성·금융+법률+하이브리드 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab5%20Graph%20RAG.ipynb) |
| 6 | lab6 Agentic RAG & 쿼리 라우팅 | 검색을 도구로 반복 호출하는 에이전트형 RAG·질문 유형 라우팅 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab6%20Agentic%20RAG%20%26%20%EC%BF%BC%EB%A6%AC%20%EB%9D%BC%EC%9A%B0%ED%8C%85.ipynb) |
| 7 | lab7 통합 Capstone | 베이스라인→Contextual→Hybrid→Rerank 누적 개선을 수치로 입증 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab7%20%ED%86%B5%ED%95%A9%20Capstone.ipynb) |
| 8 | lab8 RAG 운영 모니터링 | 골든셋 회귀·버전별 점수 drift 추적·비용 대시보드 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/suanlab/Advanced-RAG/blob/main/colab/lab8%20RAG%20%EC%9A%B4%EC%98%81%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81.ipynb) |

## 📋 사후평가 · 발표자 노트

- [사후평가 문항 (5문항·해설)](assessment/post_assessment.md)
- 발표자 노트: [모듈 1](notes/module1-presenter-notes.md) · [모듈 2](notes/module2-presenter-notes.md)

## 📂 구성

```
slides/      HTML 슬라이드 2덱 (자기완결 — 더블클릭으로도 열림)
colab/       Colab 실습 노트북 8개
assessment/  사후평가 5문항
notes/       슬라이드별 발표자 노트
```

## ℹ️ 안내

- 실습 기본 모델: `claude-sonnet-4-6` (정확도가 중요한 judge·추출 단계는 `claude-opus-4-8`로 상향 가능). API 키는 노트북에서 `getpass`로 입력하며 코드에 저장되지 않습니다.
- 모든 데이터는 교육용 합성 데이터입니다(실제 기업·판례·인물 아님).
- © 이수안컴퓨터연구소(SuanLab) · 교육용 자료.
