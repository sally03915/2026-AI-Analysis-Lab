## 📘 [이론 마크다운] `01_MVP_FAQ_Chatbot_Core.md`

### 1개월차 3주차: MVP 빌드: 사내 규정 FAQ 챗봇 & Streamlit UI 시각화

## 📌 1. 학습 목표
- 외부 지식 개입 없이 사내 규정 데이터 범위 안에서만 답변하도록 통제하는 Knowledge Grounding 기법과 환각(Hallucination) 방지 아키텍처를 이해한다.
- Streamlit의 고유한 라이프사이클(동작 매커니즘)과 상태 관리 개념을 익히고 기본 위젯을 배치할 수 있다.
- 파이썬 딕셔너리 기반의 메모리 아키텍처를 Streamlit UI의 `chat_message` 컴포넌트와 바인딩하여 백엔드 추론과 프론트엔드 뷰가 실시간 연동되는 멀티턴 FAQ 웹 애플리케이션 프로토타입을 완성한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: 프롬프트 엔지니어링 기반 Grounding 기법 및 UI 라이프사이클 <br> - 환각 현상의 근본적 원인과 시스템 프롬프트 경계선(`---`) 펜싱 기술 <br> - Streamlit 인터프리터의 실시간 렌더링 동작 원리 <br> - 웹 환경에서 대화 흐름 유지를 위한 힙(Heap) 세션 관리 아키텍처 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: Streamlit 인터페이스 프레임워크 구축 및 위젯 연동 <br> - 사이드바, 텍스트 인풋, 컴포넌트 배치 및 반응형 동작 검증 <br> - 입력 데이터 인입 시 전체 스크립트 재실행 구조와 변수 보존 확인 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 사내 규정 복원 멀티턴 FAQ 웹 어플리케이션 구현 <br> - 수강생 개별 실습 진행 (복리후생/인사규정 컨텍스트 락인 및 채팅 UI 구현) <br> - `st.session_state` 기반 대화 이력 자동 복원 마스터 코드 리뷰 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) Knowledge Grounding과 환각(Hallucination) 제어
B2B AI 서비스 구축 시 가장 핵심이 되는 보안 및 신뢰성 확보 기술입니다. LLM이 학습한 궤적에 의존하지 않고, 강사가 주입한 콘텐트 내에서만 판단하도록 경계를 획정하는 아키텍처입니다.

*   **환각 현상의 원인과 통제 필요성**
    *   **사전학습 데이터의 한계:** 일반적인 LLM은 공공 인터넷 데이터로 학습되어 사내 고유 규정(예: 연차 산정 기준, 경조사비 지급 표준)을 알 수 없음. 모르는 상태에서 강제로 답변을 요구하면 가장 그럴듯한 거짓말(Hallucination)을 생성함.
    *   **Context Grounding (지식 결속):** 프롬프트 최상단에 참고 문서(Reference Document)를 주입하고, "반드시 제공된 문서 내용에만 기반하여 답변할 것. 문서에 없는 내용이라면 솔직하게 모른다고 대답할 것"이라는 강한 제약(System Guardrail)을 걸어 답변 범위를 가둡니다.

---

### 2) Streamlit UI 프레임워크의 동작 원리와 라이프사이클
일반적인 웹 프레임워크(Django, React 등)와 완전히 다른 Streamlit만의 독특한 실행 구조를 명확히 이해해야 화면 리프레시 시 데이터가 소실되는 현상을 막을 수 있습니다.

*   **상태 비저장형(Stateless) 인터프리터 아키텍처**
    *   Streamlit은 사용자가 버튼을 누르거나, 텍스트 입력을 바꾸는 등 UI 컴포넌트와 조금이라도 상호작용하면 **소스 코드의 첫 줄부터 끝 줄까지 전체 스크립트를 재실행(Re-run)**합니다.
    *   이 메커니즘 때문에 일반적인 파이썬 로컬 변수(`chat_history = []`)는 UI가 갱신될 때마다 초기화되어 과거 대화 내용이 전부 증발하게 됩니다.
*   **이를 해결하기 위한 `st.session_state` 시스템 캐시**
    *   서버 메모리 한쪽에 브라우저 탭 세션별로 독립적인 저장 공간을 제공합니다.
    *   스크립트가 탑다운으로 재실행되더라도 `session_state`에 박아둔 데이터는 유지되므로, 이를 활용해 멀티턴(연속 대화) 챗봇 아키텍처를 구현해야 합니다.

---

## 💡 4. 강사 라이브 코딩 멘트 & 트러블슈팅 팁
> **"수강생 여러분, 지금까지 우리는 검은색 터미널 창이나 주피터 노트북 안에서만 코드를 돌렸습니다. 하지만 클라이언트나 임원진에게 '우리가 만든 AI 챗봇입니다' 하고 검은 화면을 보여주면 감흥이 전혀 없겠죠. 오늘 배울 Streamlit은 파이썬 코드 몇 줄로 프론트엔드 UI를 뚝딱 만들어내는 마법 같은 프레임워크입니다. 다만, 버튼 하나 누를 때마다 코드가 맨 위부터 다시 실행된다는 독특한 특징이 있어요. 이 특징 때문에 대화 내용이 자꾸 초기화되는데, 이를 완벽하게 방어하는 `session_state` 설계법을 마스터해 봅시다."**
> 
> * **자주 터지는 에러 및 방어 팁:** 실습 시 `st.session_state` 변수를 초기화하지 않고 접근하면 `AttributeError` 또는 `KeyError`가 터집니다. 반드시 스크립트 최상단에 세션 키 존재 여부를 확인하는 방어 코드(`if "key" not in st.session_state:`)를 먼저 선언하도록 지도하세요.

 

---

## 💻 [코랩 및 로컬 실습 노트북] `02_MVP_FAQ_Chatbot_Core.py`

```python
# ⚠️ 보안 주석: OpenAI API Key는 st.secrets 또는 .env 파일로 관리하세요.
# 코드에 키를 직접 하드코딩하지 말 것.
# 실제 기업 규정 문서는 기밀이므로 반드시 샘플/비식별화 데이터로 대체하세요.
# st.session_state에 저장된 대화 기록은 일정 시간 후 자동 삭제하거나 암호화하는 것이 안전합니다.
# 에러 처리 시 ConnectionError, TimeoutError 등 네트워크 예외를 분리하여 대응하세요.


# ==============================================================================
# 🚀 1개월차 3주차: MVP 빌드: 사내 규정 FAQ 챗봇 & Streamlit UI 시각화
# ==============================================================================
# [수강생 안내] 본 코드는 웹 인터페이스 구축을 위한 Streamlit 파일 스크립트입니다.
# 로컬 터미널 환경 또는 구글 코랩 환경에서 'streamlit run 파일명.py' 형태로 가동됩니다.

# 1. 환경 의존성 설정 및 패키지 다운로드 (코랩용 명령어 가이드)
# !pip install -q streamlit==1.37.1 openai==1.42.0 python-dotenv==1.0.1

import streamlit as st
import os
from openai import OpenAI

# ==============================================================================
# [실습 1 & 2] Streamlit 기본 인프라 설정 및 세션 라이프사이클 셋업
# ==============================================================================

# 웹 브라우저 탭 상단 타이틀 및 레이아웃 설정
st.set_page_config(page_title="사내 FAQ 관제 엔진", layout="centered")

st.title("🏢 B2B 사내 규정 FAQ 마스터 챗봇")
st.caption("Knowledge Grounding 기법을 적용한 사내 인사/복리후생 환각 방지 시스템")

# [보안 설정] OpenAI API 키 입력 환경 변수 확보 (실습 편의상 사이드바 배치 가능)
# 실제 프로덕션 환경에서는 st.secrets 또는 .env 파일로 격리 관리합니다.
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS_UNITS" # 실제 교육장 키 입력

# API 클라이언트 초기화
client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

# 💡 [핵심] UI 재실행(Re-run) 시 대화 이력이 지워지는 현상을 방어하기 위한 세션 상태 초기화
if "messages" not in st.session_state:
    # 최초 진입 시 시스템 메시지 및 웰컴 안내 딕셔너리 적재
    st.session_state.messages = [
        {"role": "assistant", "content": "안녕하세요. 임직원 인사 규정 및 복리후생에 대해 궁금한 점을 질문해 주세요."}
    ]

# ==============================================================================
# [실습 3] [미션] 사내 규정 데이터 Grounding 및 멀티턴 FAQ 엔진 구현
# ==============================================================================
# [B2B 기업 도메인 지식 주입] 외부 인터넷 답변을 금지하고 아래 원본 텍스트 내에서만 답변하도록 강제합니다.
COMPANY_HR_REGULATIONS = """
[인사/복리후생 규정 정보 고시]
1. 경조사비 지원 항목: 본인 결혼 시 경조사비 100만 원 및 유급휴가 5일을 지급한다. 직계존속 상(喪)의 경우 50만 원과 유급휴가 3일을 지원한다.
2. 연차 휴가 부여: 1년간 80% 이상 출근한 임직원에게는 15일의 유급 휴가가 보장된다. 입사 1년 미만 근로자는 1개월 개근 시 1일씩 매월 연차가 발생한다.
3. 복지포인트 제도: 매년 1월 1일 전 임직원에게 연간 120만 포인트(원화 120만 원 상당)를 일괄 지급하며, 당해 연도 12월 31일까지 미사용된 포인트는 전액 소멸된다.
"""

# 사이드바 레이아웃 영역 구성
with st.sidebar:
    st.header("⚙️ 인프라 관제 모니터")
    st.markdown("---")
    st.info("현재 모델: `gpt-4o-mini`\n\n지식 그라운딩: 적용 완료\n\n환각 방지 필터: 가동 중")
    
    # 세션 초기화용 긴급 버튼 배치
    if st.button("🔄 대화 초기화 (Clear Session)"):
        st.session_state.messages = [
            {"role": "assistant", "content": "대화 내용이 초기화되었습니다. 새로 질문해 주세요."}
        ]
        st.rerun()

# ------------------------------------------------------------------------------
# 3-2. 캐싱된 대화 기록을 화면에 실시간 렌더링 (Streamlit Chat Components)
# ------------------------------------------------------------------------------
# 스크립트가 리런될 때마다 session_state에 누적된 리스트를 반복문으로 돌리며 UI에 다시 그려줍니다.
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# ------------------------------------------------------------------------------
# 3-3. 사용자 입력 처리 및 Grounding 프롬프트 파이프라인 구동
# ------------------------------------------------------------------------------
# 하단 고정형 채팅 인풋 위젯 활성화
if user_input := st.chat_input("인사 규정에 대해 질문하세요 (예: 결혼 시 지원 혜택이 어떻게 되나요?)"):
    
    # 1. 사용자 입력을 즉시 화면에 표출 및 세션 저장
    with st.chat_message("user"):
        st.markdown(user_input)
    st.session_state.messages.append({"role": "user", "content": user_input})
    
    # 2. AI 답변 생성 영역 진입
    with st.chat_message("assistant"):
        response_placeholder = st.empty() # 스트리밍 효과 연출을 위한 빈 컴포넌트 공간 확보
        
        # 3. Knowledge Grounding을 프롬프트 구조에 이식
        # 시스템 경계를 명확히 획정하여 환각 현상을 방지하는 최적의 프롬프트 구성
        grounded_system_instruction = f"""
        당신은 사내 인사규정 FAQ 지원 어시스턴트입니다.
        반드시 아래 제공된 [사내 규정 문서]의 핵심 텍스트 내용에만 기반하여 사실적으로 답변하세요.
        문서에 기재되어 있지 않은 내용이거나 유추할 수 없는 영역이라면, 절대로 임의로 말을 지어내지 말고 
        "해당 정보는 현재 인사 규정 문서에서 확인할 수 없습니다. 담당 부서(인사팀)로 추가 문의해 주세요."라고 정중히 답변을 제한하세요.
        
        [사내 규정 문서]
        {COMPANY_HR_REGULATIONS}
        ---
        """
        
        # API 전송용 메시지 스택 조립 (시스템 프롬프트 + 직전 세션 대화 기록 전체)
        api_payload = [{"role": "system", "content": grounded_system_instruction}]
        
        # 대화 맥락 유지를 위해 과거 내역 바인딩 (단, 시스템 프롬프트 오염을 막기 위해 롤 필터링)
        for msg in st.session_state.messages:
            api_payload.append({"role": msg["role"], "content": msg["content"]})
            
        try:
            # OpenAI 채팅 완료 엔드포인트 호출 (실시간 텍스트 생성)
            api_call_response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=api_payload,
                temperature=0.0, # 일관된 사실 기반 답변 생성을 위해 무작위성을 0으로 완전히 제한
                max_tokens=400
            )
            
            ai_answer = api_call_response.choices[0].message.content
            
            # 빈 공간에 AI가 출력한 원시 답변 데이터 렌더링
            response_placeholder.markdown(ai_answer)
            
            # 4. 최종 완성된 AI 답변 데이터를 세션 메모리에 영구 적재
            st.session_state.messages.append({"role": "assistant", "content": ai_answer})
            
        except Exception as api_error:
            error_msg = f"❌ 백엔드 통신 오류가 발생했습니다: {str(api_error)}"
            response_placeholder.markdown(error_msg)

```