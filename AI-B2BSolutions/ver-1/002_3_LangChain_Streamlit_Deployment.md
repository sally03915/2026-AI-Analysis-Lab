## 📅 2개월차 3주차: 사내 문서 검색 챗봇 제작 및 Streamlit Cloud 배포

> **[지난 주차 요약 및 연계 브리핑]**
> * **2주차 핵심:** 대용량 사내 문서를 청킹(`RecursiveCharacterTextSplitter`)하고, Chroma DB 벡터 스토어에 임베딩하여 유사도 검색을 수행하는 RAG 백엔드 엔진을 완성했습니다.
> * **3주차 연계:** 이번 주차에는 앞서 구축한 RAG 엔진을 직관적인 웹 인터페이스에 이식하고, 브라우저 새로고침 시에도 대화 상태가 유지되는 Streamlit 고급 상태 관리 기법을 적용한 뒤, 최종적으로 [Streamlit Cloud](https://www.tistory.com/)를 통해 전사 임직원이 접속할 수 있는 프로덕션 환경으로 배포합니다.
 

---

## 📘 [이론 마크다운] `02_LangChain_Streamlit_Deployment.md`
 
# 2개월차 3주차: Streamlit 고급 상태 관리 및 프로덕션 Cloud 배포

## 📌 1. 학습 목표
- Streamlit의 렌더링 사이클과 `session_state`의 개념을 이해하고, 멀티턴 대화 이력과 RAG 컨텍스트가 새로고침에도 유실되지 않도록 UI 상태를 제어한다.
- 2주차에 빌드한 RAG 백엔드 엔진을 Streamlit 컴포넌트(사이드바, 채팅 입력 창)와 유기적으로 결합하여 완성도 높은 웹 애플리케이션을 구축한다.
- GitHub 퍼블릭 저장소 연동 및 Streamlit Cloud 대시보드를 통해 별도의 서버 구축 없이 무료로 웹 서비스를 배포하고, `.streamlit/secrets.toml`을 활용한 API Key 보안 격리 프로세스를 습득한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: Streamlit 렌더링 생명주기 및 Cloud 배포 아키텍처 <br> - 새로고침 시 state 초기화 현상 방어 메커니즘 <br> - 퍼블릭 저장소 보안 관리 및 `Secrets` 환경 변수 바인딩 구조 <br> - 엔터프라이즈 환경에서의 웹 배포 가이드라인 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: Streamlit 세션 상태 관리 및 RAG 엔진 UI 결합 <br> - `st.session_state` 기반 채팅 히스토리 누적 레이아웃 구현 <br> - `st.chat_message` 컴포넌트를 활용한 시각적 대화 인터페이스 튜닝 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] GitHub 소스 푸시 및 Streamlit Cloud 프로덕션 배포 <br> - 수강생 개별 웹 앱 배포 시연 및 방화벽/API Key 에러 최종 점검 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) Streamlit 렌더링 생명주기와 `session_state`
*   **스크립트 재실행 구조:** Streamlit은 사용자가 버튼을 누르거나 채팅을 입력할 때마다 파이썬 스크립트 전체를 처음부터 다시 실행하는 단방향 렌더링 구조를 가짐.
*   **상태 유실 방어:** 이로 인해 대화 기록이나 검색된 문서 인스턴스가 초기화되는 문제가 발생하므로, 브라우저 탭 세션 메모리에 데이터를 고정시키는 `st.session_state` 관리가 필수적임.

---

### 2) Production 환경 배포와 Secrets 관리
*   **보안 격리:** 소스코드를 GitHub에 업로드할 때 OpenAI API Key를 코드 내부에 하드코딩하면 계정이 탈취되거나 과도한 과금이 발생할 수 있음.
*   **Streamlit Secrets:** 클라우드 대시보드 보안 설정 영역(`secrets.toml`)에 키를 등록하고, 코드 상에서는 `st.secrets["OPENAI_API_KEY"]` 형태로 안전하게 호출하는 베스트 프랙티스 습득.

 

---

## 💻 [코랩/로컬 실습 스크립트] `app.py` (Streamlit 프로덕션 배포용)

```python
# ==============================================================================
# 🚀 2개월차 3주차: Streamlit UI 상태 관리 및 RAG 백엔드 통합 웹 앱 템플릿
# ==============================================================================
# [수강생 안내] 본 스크립트는 로컬 실행 및 Streamlit Cloud 배포를 위한 
# 통합 프로덕션 템플릿 코드입니다. (파일명: app.py로 저장하여 구동)

import os
import streamlit as st
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma

# 1. 페이지 레이아웃 메타 설정
st.set_page_config(
    page_title="엔터프라이즈 사내 규정 RAG 챗봇", 
    page_icon="🛡️", 
    layout="centered"
)

st.title("🛡️ 엔터프라이즈 사내 규정 관제 챗봇")
st.caption("LangChain RAG 파이프라인과 Streamlit Cloud 프로덕션 연동 시스템")

# 2. API Key 보안 로드 (Streamlit Cloud Secrets 우선 탐색, 없으면 환경변수 활용)
try:
    OPENAI_API_KEY = st.secrets["OPENAI_API_KEY"]
except Exception:
    OPENAI_API_KEY = os.environ.get("OPENAI_API_KEY", "sk-proj-YOUR_MOCK_KEY")

if not OPENAI_API_KEY.startswith("sk-"):
    st.warning("⚠️ OpenAI API Key가 설정되지 않았거나 유효하지 않습니다. secrets.toml 설정을 확인하세요.")

# ==============================================================================
# [실습 1 & 2] RAG 엔진 메모리 적재 및 벡터 스토어 초기화 (캐싱 처리)
# ==============================================================================
@st.cache_resource
def load_rag_engine_and_vectorstore():
    """
    웹 앱이 리프레시될 때마다 무거운 임베딩과 문서 인덱싱을 다시 수행하지 않도록
    Streamlit 캐시 데코레이터로 벡터 스토어를 안전하게 메모리에 고정합니다.
    """
    sample_manual_data = """
    [제 1장 정보 보안 지침]
    제1조(비밀유지) 임직원은 업무상 취득한 사내 기밀 문서를 외부 AI 서비스나 퍼블릭 클라우드에 무단으로 업로드할 수 없다.
    제2조(인증 체계) 모든 사내 내부망 서버 접근 시 2채널 인증(MFA)을 필수로 거쳐야 하며, 예외 허용은 없다.
    제3조(RAG 시스템 기준) 사내 문서 자동화 검색 챗봇 구축 시, 벡터 스토어는 Chroma DB 인메모리 엔진을 표준으로 채택한다.
    """
    
    splitter = RecursiveCharacterTextSplitter(chunk_size=120, chunk_overlap=20)
    chunks = splitter.create_documents([sample_manual_data])
    
    embeddings = OpenAIEmbeddings(model="text-embedding-3-small", openai_api_key=OPENAI_API_KEY)
    vstore = Chroma.from_documents(chunks, embeddings)
    return vstore.as_retriever(search_kwargs={"k": 1})

# 리트리버 로드
retriever = load_rag_engine_and_vectorstore()

# LLM 및 LCEL 파이프라인 구성
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0.0, openai_api_key=OPENAI_API_KEY)
prompt = ChatPromptTemplate.from_messages([
    ("system", "너는 사내 보안 규정 안내 어시스턴트야. 아래 참고 문서 내용에만 근거해서 명확하게 답변해줘."),
    ("human", "[참고 문서]\n{context}\n\n[사용자 질문]\n{question}")
])
rag_chain = prompt | llm | StrOutputParser()


# ==============================================================================
# [실습 3] Streamlit session_state 기반 대화 이력 유지 UI 구성
# ==============================================================================
if "messages" not in st.session_state:
    st.session_state.messages = []

# 기존 대화 기록 화면 재출력 (새로고침 방어)
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# 사용자 질의 입력 위젯
if user_input := st.chat_input("사내 보안 규정이나 지침에 대해 질문해 주세요 (RAG 연동 검색)"):
    
    # 1. 사용자 메시지 UI 표출 및 세션 적재
    with st.chat_message("user"):
        st.markdown(user_input)
    st.session_state.messages.append({"role": "user", "content": user_input})
    
    # 2. RAG 검색 및 AI 답변 생성 프로세스
    with st.chat_message("assistant"):
        with st.spinner("🔍 사내 규정 문서에서 관련 지침을 검색 중입니다..."):
            try:
                # Retriever를 통한 문서 조각 인출
                docs = retriever.invoke(user_input)
                ctx = docs[0].page_content if docs else "관련 규정을 찾을 수 없습니다."
                
                # LCEL 파이프 실행
                answer = rag_chain.invoke({"context": ctx, "question": user_input})
                
                st.markdown(answer)
                # 세션에 AI 답변 적재
                st.session_state.messages.append({"role": "assistant", "content": answer})
                
            except Exception as e:
                error_msg = f"❌ RAG 파이프라인 실행 중 오류가 발생했습니다: {str(e)}"
                st.error(error_msg)

```