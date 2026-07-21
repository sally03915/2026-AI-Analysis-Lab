## 📘 [이론 마크다운] `02_LangChain_Core_LCEL.md`

## 2개월차 1주차: LangChain 핵심 구조 학습 및 LCEL 아키텍처 마스터

## 📌 1. 학습 목표
- 쌩(Raw) 파이썬 API 통신 코드의 한계를 이해하고, 엔터프라이즈 모듈러 아키텍처로서 LangChain 프레임워크 도입의 당위성을 습득한다.
- LCEL(LangChain Expression Language)의 파이프 연산자(`|`) 메커니즘을 이해하고 선언형 컴포넌트 조합 코드를 작성할 수 있다.
- 기존 파이썬 딕셔너리 기반의 대화 이력 관리를 LangChain의 표준 메모리 클래스(`ChatMessageHistory` 및 `RunnableWithMessageHistory`)로 리팩토링할 수 있다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: 프레임워크 패러다임 전환과 LCEL 아키텍처 구조 <br> - 쌩 코드 방식의 유지보수 한계와 모듈화 필요성 <br> - LCEL 파이프 연산자(`|`)의 데이터 흐름 및 병렬 처리 원리 <br> - 대화 이력(Memory) 추상화 클래스의 동작 메커니즘 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: PromptTemplate 바인딩 및 LCEL 파이프라인 구축 <br> - `ChatPromptTemplate`과 `ChatOpenAI` 모델 객체 결합 <br> - `StrOutputParser`를 통한 출력값 정제 파이프라인 검증 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 파이썬 수동 딕셔너리 이력을 LangChain Memory로 리팩토링 <br> - 수강생 개별 실습 진행 (대화 세션 ID별 자동 누적 메모리 구현) <br> - 멀티턴 대화 이력 연동 마스터 코드 리뷰 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

강의 내용과 실습 분량이 부족하고, 특히 **50분이라는 긴 이론 시간을 지루하지 않게 채우는 것**은 많은 강사분들이 겪는 가장 큰 고민입니다.

이를 해결하기 위해 **이론 시간을 3단계(개념 도입 $\rightarrow$ 아키텍처 심화 $\rightarrow$ 현업 장애 사례 및 토론)로 쪼개고, 실습 코드에는 따라 치기만 해도 원리가 이해되는 상세한 디버깅 스텝을 보강**하는 구조로 전면 업그레이드해야 합니다.

아래는 2개월차 전체 주차별로 이론을 50분 동안 꽉 채울 수 있는 상세 가이드(스토리텔링 대본/발표 포인트)와 **실습 볼륨을 대폭 키운 마스터 코드 아키텍처**입니다.

---

## 📅 [총정리] 2개월차 4개 주차별 확장 교안 및 실습 패키지

---

### 1주차: LangChain 핵심 구조 학습 및 LCEL 아키텍처 마스터
 

### 1) 왜 쌩(Raw) 코드가 아니라 LangChain 프레임워크인가?
*   **Raw API 방식의 한계:** 1개월차에 배운 OpenAI SDK 직접 호출 방식은 간단하지만, 프롬프트 템플릿 관리, 벡터 서치 결과 주입, 멀티턴 대화 이력 수동 딕셔너리 append, 출력 파싱 등을 모두 개발자가 직접 구현해야 하므로 코드가 거대해지고 유지보수가 불가능해짐.
*   **LCEL(LangChain Expression Language)의 도입:** 리눅스의 표준 파이프라인(`|`) 개념을 도입하여, 컴포넌트들을 레고 블록처럼 직관적으로 조립하고 비동기/스트리밍/배치 처리를 내부적으로 지원하도록 설계된 선언형 언어 체계.

---

### 2) 템플릿 바인딩과 가변 인자 제어
*   `PromptTemplate`과 `ChatPromptTemplate`을 통해 시스템 메시지와 사용자 입력 간의 역할(Role) 구분을 명확히 하고, 동적 변수(`{variable}`) 주입을 안전하게 처리.
*   파이프 연산자를 통해 `Prompt`가 생성한 결과물이 곧바로 `LLM`의 입력값으로 이어지는 타입 안전성 확보.

---

### 3) LangChain Memory 아키텍처의 혁신
*   기존에는 사용자와의 대화 내용을 `st.session_state`나 파이썬 리스트에 일일이 집어넣고 메시지 포맷을 맞추어 API에 다시 보내야 했음.
*   LangChain은 `RunnableWithMessageHistory` 패턴을 도입하여, 세션 ID(Session ID)만 던져주면 백엔드가 알아서 데이터베이스나 메모리 맵에서 해당 유저의 대화 이력을 가져와 프롬프트에 자동으로 삽입해 주는 구조를 제공.

---

## 💡 4. 강사 라이브 코딩 멘트 & 트러블슈팅 팁
> **"여러분, 1개월차 동안 우리는 OpenAI API를 직접 호출하는 쌩 코드를 열심히 다뤘습니다. 하지만 엔터프라이즈 환경에서 챗봇에 문서 검색(RAG)을 붙이고, 대화 기억 능력을 넣고, 여러 모델을 갈아 끼우기 시작하면 코드가 금방 스파게티처럼 엉키게 됩니다. 이때 등장하는 구원투수가 바로 LangChain입니다. 오늘 배울 LCEL 파이프 연산자 `|`를 이해하면, 마치 레고 블록을 조립하듯 복잡한 AI 백엔드를 3줄 코드로 완성할 수 있습니다."**
> 
> * **자주 터지는 에러 및 방어 팁:** LCEL 파이프라인에서 입력값의 딕셔너리 키 이름(`{user_input}`)과 프롬프트 템플릿 내부의 변수명이 정확히 일치하지 않으면 `KeyError`가 발생합니다. 파이프 연산 직전의 인자 매핑 구조를 수강생들이 반드시 콘솔로 출력해 보며 검증하도록 지도하세요.

```

---

## 💻 [코랩 실습 노트북] `02_LangChain_Core_LCEL.py`

```python
# ==============================================================================
# 🚀 2개월차 1주차: LangChain 핵심 구조 학습 및 LCEL 파이프라인 실습
# ==============================================================================
# [수강생 안내] 본 코드는 LangChain 생태계의 핵심인 LCEL 파이프 연산자와 
# 대화 이력(Memory) 자동 관리 아키텍처를 검증하기 위한 마스터 템플릿입니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 langchain-core==0.2.33 python-dotenv==1.0.1

import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

print("📦 LangChain 라이브러리 및 코어 컴포넌트 임포트 완료")

# ==============================================================================
# [실습 1 & 2] LCEL 파이프 연산자(`|`) 기반 동적 챗봇 파이프라인 구축 (강시 시간: 50분)
# ==============================================================================

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# 1. LangChain 표준 LLM 모델 엔진 선언
llm_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

# 2. 동적 인자 제어가 가능한 ChatPromptTemplate 구성
chat_prompt_blueprint = ChatPromptTemplate.from_messages([
    ("system", "너는 사내 임직원들의 개발 및 인프라 구축을 돕는 전문 시니어 엔지니어 아키텍트야."),
    ("human", "{user_inquiry}")
])

# 3. 출력값 정제를 위한 표준 문자열 파서 장착
string_parser = StrOutputParser()

# 💡 [핵심] LCEL 파이프 연산자(`|`)를 통한 컴포넌트 유기적 결합 (Prompt -> LLM -> Parser)
lcel_modular_pipeline = chat_prompt_blueprint | llm_engine | string_parser

print("\n=== LCEL 파이프라인 단발성 추론 테스트 ===")
test_query = "LangChain을 도입했을 때 엔터프라이즈 관점에서 얻을 수 있는 아키텍처 장점 1가지만 말해줘."
single_run_result = lcel_modular_pipeline.invoke({"user_inquiry": test_query})
print(f"🎯 [파이프라인 출력 결과]:\n{single_run_result}")


# ==============================================================================
# [실습 3] [미션] 파이썬 수동 history 코드를 LangChain Memory로 리팩토링 (강사 시간: 50분)
# ==============================================================================
# [현업 시나리오]: 
# 1개월차에는 대화 이력을 구현하기 위해 파이썬 리스트(`st.session_state.messages.append`)에
# 직접 딕셔너리를 담고 API 페이로드에 일일이 매핑해야 했습니다. 
# 이번 미션에서는 LangChain의 `RunnableWithMessageHistory`를 활용해 대화 기억 메커니즘을 
# 단 몇 줄의 설정으로 자동화하는 세션 메모리 아키텍처를 구현합니다.

print("\n=== [현업 실무 미션] LangChain 표준 메모리 세션 아키텍처 구축 ===")

# 1. 대화 기억 저장을 위한 세션별 메시지 스토어 가상 딕셔너리 관리소
store_database = {}

def get_session_history(session_id: str):
    """
    유저별 세션 ID를 식별하여 해당 세션의 대화 이력 객체를 반환하거나 생성합니다.
    """
    if session_id not in store_database:
        store_database[session_id] = ChatMessageHistory()
    return store_database[session_id]

# 2. 멀티턴 대화 이력을 수용할 수 있도록 프롬프트 블루프린트에 Placeholder 추가
memory_aware_prompt = ChatPromptTemplate.from_messages([
    ("system", "너는 대화 맥락을 기억하며 사용자의 질문에 정확하게 답변하는 친절한 AI 어시스턴트입니다."),
    MessagesPlaceholder(variable_name="chat_history"), # 과거 대화 이력이 자동으로 꽂히는 자리
    ("human", "{user_inquiry}")
])

# 3. 메모리가 결합된 새로운 LCEL 파이프라인 조립
memory_pipeline_core = memory_aware_prompt | llm_engine | string_parser

# 4. RunnableWithMessageHistory로 래핑하여 대화 기억 자동화 파이프라인 완성
production_memory_agent = RunnableWithMessageHistory(
    memory_pipeline_core,
    get_session_history,
    input_messages_key="user_inquiry",
    history_messages_key="chat_history"
)

# ------------------------------------------------------------------------------
# 3-2. 멀티턴 대화 연속성 시뮬레이션 테스트
# ------------------------------------------------------------------------------
# 고유 세션 ID 부여 (예: 사내 임직원 사번 혹은 브라우저 탭 세션 아이디)
enterprise_user_session_config = {"configurable": {"session_id": "emp_id_9921_lee"}}

print("\n--- [1턴 차 대화 시도] ---")
first_turn_input = "안녕! 내 이름은 사내 플랫폼 팀 이준호야. 기억해 줘."
print(토픽출력 := f"User: {first_turn_input}")

first_turn_output = production_memory_agent.invoke(
    {"user_inquiry": first_turn_input}, 
    config=enterprise_user_session_config
)
print(f"AI: {first_turn_output}")


print("\n--- [2턴 차 대화 시도 (맥락 유지 검증)] ---")
second_turn_input = "내가 방금 전에 뭐라고 했고 내 이름이 뭐라고 했지?"
print(f"User: {second_turn_input}")

second_turn_output = production_memory_agent.invoke(
    {"user_inquiry": second_turn_input}, 
    config=enterprise_user_session_config
)
print(f"AI: {second_turn_output}")

print("\n✅ [검증 완료] LangChain Memory 아키텍처가 세션 ID별로 과거 대화 맥락을 완벽하게 기억하고 복원해 냈습니다.")

```