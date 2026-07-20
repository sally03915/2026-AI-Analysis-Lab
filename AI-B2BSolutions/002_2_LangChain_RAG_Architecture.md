## 📅 2개월차 2주차: RAG 기반 문서 검색 및 요약 자동화

### 1주차. 학습 내용 요약

* **Ch 1. RAG(Retrieval-Augmented Generation) 아키텍처 기본**
* Document Loaders(PDF, TXT, CSV)와 Text Splitters(청킹 전략)를 통한 대용량 문서 분할 기법
* 임베딩(Embedding) 모델의 원리와 Vector 스토어(Chroma DB) 연동 구조 이해


* **Ch 2. 기업 비용 절감을 위한 Prompt Caching 전략**
* 중복 컨텍스트 토큰 비용을 최적화하기 위한 캐싱 메커니즘 이론


* **Ch 3. [실습] 대용량 보고서 요약 및 Advanced RAG 튜닝**
* 방대한 사내 문서를 청킹·임베딩하여 핵심 요약본 추출 파이프라인 구현
* Re-ranking 알고리즘을 활용한 유사도 검색 쿼리 고도화



---

## 📘 [이론 마크다운] `02_LangChain_RAG_Architecture.md`

 
# 2개월차 2주차: RAG 아키텍처 및 Vector Store 엔지니어링 마스터

## 📌 1. 학습 목표
- 대규모 사내 문서(PDF, TXT)를 LLM이 이해할 수 있는 작은 조각으로 나누는 청킹(Chunking) 전략을 이해한다.
- 임베딩 모델을 통해 텍스트를 다차원 벡터 공간에 매핑하고, Chroma DB를 활용해 밀리초 단위의 유사도 검색(Retrieval) 시스템을 구축한다.
- 토큰 비용 절감을 위한 프롬프트 캐싱(Prompt Caching) 최적화 전략과 검색 정확도를 높이는 Advanced RAG 튜닝 기법을 습득한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: RAG 아키텍처 원리와 Vector Store 딥다이브 <br> - Document Loader & RecursiveCharacterTextSplitter 메커니즘 <br> - 임베딩 공간과 코사인 유사도 기반 검색 원리 <br> - 엔터프라이즈 환경에서의 Prompt Caching 비용 절감 전략 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: 사내 문서 로딩, 청킹 및 Chroma DB 벡터 스토어 연동 <br> - PDF/텍스트 파일 파싱 및 벡터화 파이프라인 구축 <br> - Retriever 객체를 통한 시맨틱 검색 테스트 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 대용량 보고서 요약 챗봇 및 Advanced RAG 튜닝 <br> - 검색된 문서 조각(Context)을 프롬프트에 주입하여 답변 정확도 극대화 <br> - 수강생 개별 실습 및 Re-ranking 적용 피드백 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) RAG(Retrieval-Augmented Generation)의 필요성
*   **LLM의 한계 극복:** LLM은 학습된 시점의 정보만 알 수 있고, 기업 내부의 기밀 문서나 최신 규정은 알지 못함. 또한 컨텍스트 윈도우 제한으로 인해 방대한 책 한 권을 통째로 넣을 수 없음.
*   **RAG 파이프라인 흐름:** `문서 수집(Load)` -> `텍스트 쪼개기(Split)` -> `벡터화 및 저장(Embed & VectorStore)` -> `사용자 질문과 유사한 조각 검색(Retrieve)` -> `LLM에 주입하여 답변 생성(Generate)`.

---

### 2) 문서 청킹(Chunking)과 오버랩(Overlap) 전략
*   **RecursiveCharacterTextSplitter:** 문맥이 끊기지 않도록 단락, 문장, 단어 단위 순으로 문서를 재귀적으로 분할.
*   **Chunk Overlap:** 조각과 조각 사이에 겹치는 구간을 두어, 핵심 키워드가 잘리거나 문맥이 유실되는 현상을 방지.

---

### 3) 프롬프트 캐싱 (Prompt Caching)과 비용 최적화
*   대규모 RAG 시스템에서는 매번 유사한 사내 매뉴얼 컨텍스트가 프롬프트에 반복 주입되므로 토큰 비용이 급증함.
*   API 레벨에서 고정된 시스템 프롬프트나 대용량 컨텍스트 영역을 캐싱하여, LLM 제공사(OpenAI 등)의 토큰 처리 비용을 대폭 절감하는 엔터프라이즈 필수 아키텍처 전략.

 

---

## 💻 [코랩 실습 노트북] `02_LangChain_RAG_Pipeline.py`

```python
# ==============================================================================
# 🚀 2개월차 2주차: RAG 문서 검색, 벡터 스토어(Chroma) 연동 및 요약 실습
# ==============================================================================
# [수강생 안내] 본 코드는 사내 문서를 청킹·임베딩하여 Vector Store에 적재하고, 
# 사용자의 질문과 가장 유사한 문맥을 찾아 답변을 생성하는 RAG 파이프라인 마스터 템플릿입니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 chromadb==0.5.5 python-dotenv==1.0.1

import os
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma

print("📦 RAG 및 벡터 스토어 관련 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1] 사내 문서 로딩 및 텍스트 청킹(Chunking) 설정 (강사 시간: 50분)
# ==============================================================================

# 가상의 방대한 사내 보안 및 인사 규정 문서 데이터셋
enterprise_raw_document = """
[제 1장 총칙]
제1조(목적) 이 지침은 주식회사 인프라코어의 사내 정보 보호 및 클라우드 자원 활용 기준을 명확히 함을 목적으로 한다.
제2조(적용 범위) 본 지침은 본사 및 전 계열사의 임직원, 협력사 상주 인력 모두에게 적용된다.

[제 2장 클라우드 보안 및 RAG 시스템 구축 지침]
제 5조(데이터 반출 금지) 
1. 클라우드 외부 반출이 승인되지 않은 사내 기밀 문서는 로컬 USB나 개인 메일, 외부 AI 서비스에 임의로 업로드할 수 없다.
2. 사내 문서 기반의 RAG(Retrieval-Augmented Generation) 파이프라인 구축 시, 벡터 스토어는 반드시 사내 보안 인증을 거친 Chroma DB 표준 엔진을 사용해야 한다.
3. 임베딩 모델은 OpenAI의 text-embedding-3-small 규격을 준용하여 벡터 공간의 일관성을 유지한다.

[제 3장 보안 사고 대응]
제 10조(이상 징후 신고) 임직원이 권한 없는 정보에 접근하거나 데이터 유출 정황을 발견한 즉시 정보보호실에 신고해야 한다.
"""

# RecursiveCharacterTextSplitter를 통한 정교한 청크 분할 설정
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=150,      # 한 청크당 최대 글자 수
    chunk_overlap=30,    # 문맥 유실 방지를 위한 겹침 구간 글자 수
    separators=["\n\n", "\n", " ", ""]
)

# 문서 청킹 수행
document_chunks = text_splitter.create_documents([enterprise_raw_document])
print(f"✂️ 총 분할된 문서 청크 개수: {len(document_chunks)}개")


# ==============================================================================
# [실습 2] Chroma DB Vector Store 및 Retriever 연동 (강사 시간: 50분)
# ==============================================================================

# 1. 임베딩 모델 초기화
embedding_model = OpenAIEmbeddings(model="text-embedding-3-small")

# 2. Chroma 인메모리 벡터 스토어에 문서 청크 적재 (Vectorization)
vector_database = Chroma.from_documents(
    documents=document_chunks,
    embedding=embedding_model
)

# 3. 유사도 검색을 수행할 Retriever 객체 추출 (Top-k = 1)
document_retriever = vector_database.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 1}
)

print("🔍 Chroma DB Vector Store 인덱싱 및 Retriever 장착 완료")


# ==============================================================================
# [실습 3] [미션] RAG 기반 사내 규정 질의응답 파이프라인 완성 (강사 시간: 50분)
# ==============================================================================

# 1. LLM 및 RAG 프롬프트 템플릿 구성
llm_rag_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

rag_prompt_blueprint = ChatPromptTemplate.from_messages([
    ("system", "너는 사내 규정 전문 어시스턴트야. 아래에 제공된 [참고 문서] 내용에만 근거해서 정확하게 답변해줘. 문서에 없는 내용은 모른다고 답해."),
    ("human", "[참고 문서]\n{context}\n\n[사용자 질문]\n{question}")
])

rag_chain = rag_prompt_blueprint | llm_rag_engine | StrOutputParser()

# 2. RAG 시뮬레이션 테스트 실행
user_query = "RAG 파이프라인 구축할 때 어떤 벡터 스토어를 써야 해?"
print(f"\n🗣️ 사용자 질문: {user_query}")

# 3. Retriever를 통한 관련 문서 조각 검색 (Retrieval)
retrieved_docs = document_retriever.invoke(user_query)
context_text = retrieved_docs[0].page_content if retrieved_docs else ""

print(f"📄 검색된 관련 문서 조각:\n{context_text}")

# 4. 검색된 문맥을 프롬프트에 주입하여 최종 답변 생성 (Generation)
final_rag_response = rag_chain.invoke({
    "context": context_text,
    "question": user_query
})

print(f"\n🤖 AI 최종 RAG 답변:\n{final_rag_response}")
print("\n✅ [2주차 검증 완료] RAG 문서 검색 및 요약 파이프라인이 정상적으로 동작합니다.")

```