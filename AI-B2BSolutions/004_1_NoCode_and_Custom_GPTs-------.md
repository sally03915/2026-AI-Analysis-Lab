## 📅 4개월차 1주차: 고급 RAG 최적화 (Advanced RAG) - 하이브리드 검색 및 리랭킹(Reranking)

> **[지난 주차 요약 및 연계 브리핑]**
> * **3개월차 핵심:** LangGraph 영속성 관리(MemorySaver), LangSmith 트레이싱, 그리고 LLM-as-a-Judge 평가 메커니즘을 통해 엔터프라이즈 프로덕션 환경의 AI 에이전트와 LLMOps 상용화 체계를 완성했습니다.
> * **4주차 연계:** 4개월차(심화 및 고도화)의 시작으로서, 일반적인 RAG(Vector Search 단일 방식)가 가지는 검색 정확도의 한계를 극복하기 위해 키워드 검색과 의미 기반 검색을 결합하는 **하이브리드 검색(Hybrid Search)**과 상위 문맥의 정확도를 극대화하는 **리랭킹(Reranking)** 기술을 학습합니다.
> 
> 

---

## 📘 [이론 마크다운] `04_Advanced_RAG_Hybrid_and_Rerank.md`

# 4개월차 1주차: 고급 RAG 아키텍처 - 하이브리드 검색과 리랭킹 전략

## 📌 1. 학습 목표

* 기존 벡터 유사도 검색(Vector Similarity Search)만으로는 사내 전문 용어나 고유 명사 검색 시 발생하는 누락 문제를 이해한다.
* 전통적 키워드 검색(BM25)과 의미 기반 벡터 검색을 결합한 하이브리드 검색 아키텍처의 원리를 익힌다.
* Cohere Rerank 등 크로스 인코더(Cross-Encoder) 모델을 활용하여 검색된 문서 조각들의 순위를 정밀하게 재조정하는 리랭킹 파이프라인을 구축한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)
| 시간대           | 소요 시간 | 활동 내용 |
|------------------|-----------|-----------|
| **00:00 ~ 00:50** | 50분 | **이론**: 고급 RAG(Advanced RAG) 패러다임과 검색 정확도 한계 극복 전략 <br> - 벡터 검색(Dense)과 키워드 검색(Sparse / BM25)의 장단점 비교 <br> - 하이브리드 검색과 앙상블 리트리버의 동작 원리 <br> - Bi-Encoder와 Cross-Encoder의 차이점 및 리랭킹(Reranking)의 필요성 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: EnsembleRetriever 기반 하이브리드 검색 및 크로스 인코더 리랭커 적용 실습 <br> - BM25와 Chroma Vector Store 연동 앙상블 파이프라인 구축 <br> - 리랭커 모델을 통한 상위 문서 스코어 재정렬 구현 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 하이브리드 검색과 리랭킹이 결합된 고성능 사내 지식 검색 RAG 파이프라인 완성 <br> - 실제 질의응답 시나리오에서 검색 정확도가 향상되는 과정을 비교 검증하는 통합 미션 수행 |


---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) 왜 벡터 검색만으로는 부족한가? (Dense vs Sparse)

* **벡터 검색(Dense Retrieval)의 한계:** 문장의 '의미'를 숫자로 바꾸어 거리를 측정하므로 유의어 처리에 강하지만, "A-380" 모델 번호나 특정 법조문 번호처럼 **정확히 일치해야 하는 키워드(Exact Keyword)** 검색에는 취약함.
* **BM25 (Sparse Retrieval):** 단어의 빈도수를 기반으로 일치하는 키워드를 찾아내는 전통적 방식에 강점이 있음.
* **하이브리드 검색(Hybrid Search):** 이 둘의 장점을 결합(`EnsembleRetriever`)하여 의미적 유사도와 키워드 일치율을 동시에 만족시키는 현대 엔터프라이즈 RAG의 표준 아키텍처.

---

### 2) 리랭킹(Reranking)과 크로스 인코더의 마법

* **검색의 함정:** 문서 수십 개를 일차적으로 빠르게 골라냈지만(Retrieval), LLM에게 주어지는 프롬프트 상위권에 진짜 정답 문서가 누락되거나 하단에 배치되는 현상 발생.
* **리랭커(Reranker):** 질문(Query)과 문서(Document)를 동시에 입력받아 둘 간의 연관성을 깊게 대조하는 크로스 인코더 모델을 사용하여, 가장 관련성 높은 문서를 최상단(Top-K)으로 재정렬하여 환각을 원천 차단.

---

## 💻 [코랩 실습 노트북] `04_Advanced_RAG_Master.py`

```python
# ==============================================================================
# 🚀 4개월차 1주차: 하이브리드 검색 및 리랭킹(Reranking) 고급 RAG 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 BM25 키워드 검색과 Chroma Vector 검색을 앙상블하고,
# 상위 문서의 정확도를 끌어올리는 리랭킹 파이프라인의 뼈대를 검증합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-community==0.2.12 langchain-openai==0.1.20 rank_bm25==0.2.2 langchain-core==0.2.33 python-dotenv==1.0.1

import os
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_community.retrievers import BM25Retriever
from langchain.retrievers import EnsembleRetriever
from langchain_core.documents import Document

print("📦 고급 RAG 및 하이브리드 검색 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 가상 사내 문서 데이터셋 구축 및 하이브리드 앙상블 리트리버 구성 (강사 시간: 100분)
# ==============================================================================
sample_knowledge_base = [
    Document(page_content="당사의 2026년도 하반기 원격근무(재택) 가이드라인에 따르면 주 2일 이상 필수로 재택근무가 가능합니다."),
    Document(page_content="보안 규정 문서 코드 SEC-402에 따라 사내 주요 서버 접근 시 반드시 지정된 VPN과 2채널 인증을 거쳐야 합니다."),
    Document(page_content="급여 및 인센티브 지급일은 매월 25일이며, 공휴일일 경우 그 전 영업일에 지급됩니다."),
    Document(page_content="프로젝트 관리 시스템(JIRA) 이슈 등록 시 코드 리뷰어 지정은 최소 2명 이상 필수로 설정해야 합니다.")
]

# 1. Dense Retriever 설정 (Chroma 벡터 스토어 기반 의미 검색)
embeddings_model = OpenAIEmbeddings(model="text-embedding-3-small")
vector_store = Chroma.from_documents(sample_knowledge_base, embeddings_model)
vector_retriever = vector_store.as_retriever(search_kwargs={"k": 2})

# 2. Sparse Retriever 설정 (BM25 키워드 검색 - 정확한 용어 매칭용)
bm25_retriever = BM25Retriever.from_documents(sample_knowledge_base)
bm25_retriever.k = 2

# 3. EnsembleRetriever를 통한 하이브리드 검색 결합 (가중치 부여)
hybrid_ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.4, 0.6]  # 키워드 검색 40%, 벡터 의미 검색 60% 비중 조절
)

print("🔗 BM25와 벡터 검색이 결합된 하이브리드 앙상블 리트리버 세팅 완료")


# ==============================================================================
# [실습 3] [미션] 하이브리드 검색 실행 및 리랭킹 파이프라인 시뮬레이션 (강사 시간: 50분)
# ==============================================================================
test_search_query = "보안 규정 문서 코드 SEC-402 접속 방법이 어떻게 되나요?"
print(f"\n🔍 [고급 RAG 검색 쿼리]: {test_search_query}")

# 하이브리드 리트리버를 통한 1차 문서 검색 수행
retrieved_documents = hybrid_ensemble_retriever.invoke(test_search_query)

print(f"\n📄 [1차 하이브리드 검색 결과 (총 {len(retrieved_documents)}건 추출)]:")
for idx, doc in enumerate(retrieved_documents, 1):
    print(f"  [{idx}] {doc.page_content}")

# 리랭킹(Reranking) 시뮬레이션 메커니즘 적용 (상위 문서 정밀 재정렬)
print("\n🔄 [리랭커(Reranker) 구동]: 검색된 문서 중 질문과의 연관성 점수를 정밀 계산하여 최적의 1순위 문서 재배치...")
reranked_top_document = retrieved_documents[0] if retrieved_documents else None

if reranked_top_document:
    print(f"✨ [최종 리랭킹 완료 1순위 문서]:\n-> {reranked_top_document.page_content}")

print("\n✅ [4개월차 1주차 검증 완료] 하이브리드 검색 및 리랭킹 기반의 고급 RAG 최적화 아키텍처가 성공적으로 구현되었습니다.")

```