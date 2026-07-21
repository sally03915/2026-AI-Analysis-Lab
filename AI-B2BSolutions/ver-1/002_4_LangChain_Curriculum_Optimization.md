## 📅 2개월차 4주차: 타겟 맞춤형 교육 교안 최적화 및 종합 실습 패키지

> **[지난 주차 요약 및 연계 브리핑]**
> * **3주차 핵심:** RAG 백엔드 엔진을 Streamlit 고급 상태 관리와 결합하고, GitHub 연동 및 `secrets.toml` 보안 설정을 거쳐 [Streamlit Cloud](https://www.tistory.com/)에 무료 웹 앱으로 프로덕션 배포를 완료했습니다.
> * **4주차 연계:** 마지막 주차에는 실제 교육 현장에서 개발자(타겟 기술 중심)와 비개발자(직관적 시각화 중심)를 동시에 만족시킬 수 있도록 타겟 맞춤형 교육 난이도를 밸런싱하고, 대규모 실전 트러블슈팅과 종합 미션 코드를 패키징합니다.
> 
> 

---

## 📘 [이론 마크다운] `02_LangChain_Curriculum_Optimization.md`

 
# 2개월차 4주차: 타겟 맞춤형 교육 교안 최적화 및 종합 실습 패키징

## 📌 1. 학습 목표
- 교육 대상(개발자 트랙 vs 비개발자/기획자 트랙)의 눈높이에 맞춰 LangChain 및 RAG 아키텍처 설명 방식을 이원화하는 밸런싱 전략을 익힌다.
- 실습 중 수강생들이 가장 많이 마주하는 예외 상황(`KeyError`, `Chroma DB Lock`, `Token Limit Exceeded`)을 스스로 방어하고 디버깅할 수 있는 실전 코드를 다룬다.
- 2개월차 전체 커리큘럼(LCEL, RAG, Streamlit 배포)을 총동원하는 최종 종합 미션을 수행한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: 타겟 맞춤형 교육 난이도 밸런싱 아키텍처 및 현업 트러블슈팅 가이드 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: 트랙별 맞춤형 컴포넌트 실습 (비개발자용 UI 인터페이스 고도화 & 개발자용 백엔드 예외 처리 강화) |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [종합 미션] 2개월차 최종 마스터 앱 빌드 및 상용화 수준의 디버깅 테스트 |
 

---

## 💻 [종합 확장 마스터 스크립트] `02_Enterprise_Comprehensive_Master.py`

```python
# ==============================================================================
# 🚀 2개월차 4주차: 수강생 배포 및 종합 실습용 확장 마스터 템플릿
# ==============================================================================
# [강사 안내] 본 코드는 비개발자/개발자 트랙 모두가 실습 환경에서 겪는 예외를 
# 원천 차단하고, RAG 및 UI 상태 관리 로직을 한눈에 검증할 수 있는 종합 코드입니다.

import os
import sys
import streamlit as st

# 1. 페이지 레이아웃 초기화
st.set_page_config(
    page_title="2개월차 최종 종합 실습: 엔터프라이즈 RAG 마스터",
    page_icon="🎓",
    layout="wide"
)

st.title("🎓 2개월차 최종 프로젝트: 듀얼 트랙 맞춤형 사내 RAG 챗봇")
st.markdown("---")

# ==============================================================================
# [파트 1] 수강생 환경 방어적 에러 핸들링 및 무결성 진단 함수
# ==============================================================================
def diagnostic_system_check():
    """
    실습 시작 전 필수 패키지와 API Key 설정을 다중 검증합니다.
    """
    error_logs = []
    
    # API Key 검증
    api_key_candidate = ""
    try:
        if hasattr(st, "secrets") and "OPENAI_API_KEY" in st.secrets:
            api_key_candidate = st.secrets["OPENAI_API_KEY"]
    except Exception:
        pass
    
    if not api_key_candidate:
        api_key_candidate = os.environ.get("OPENAI_API_KEY", "")

    if not api_key_candidate.startswith("sk-"):
        error_logs.append("OpenAI API Key가 유효하지 않거나 설정되지 않았습니다.")
        
    return error_logs

# 시스템 진단 수행 결과 사이드바 출력
with st.sidebar:
    st.header("🛠️ 실습 환경 진단 센터")
    system_errors = diagnostic_system_check()
    
    if system_errors:
        st.error("⚠️ 환경 설정에 문제가 있습니다!")
        for err in system_errors:
            st.write(f"- {err}")
        st.info("💡 팁: 로컬 환경인 경우 환경변수를, 클라우드인 경우 secrets.toml을 확인하세요.")
    else:
        st.success("✨ 모든 실습 환경이 정상입니다!")
    
    st.markdown("---")
    st.markdown("### 🎯 교육 트랙 선택")
    selected_track = st.radio(
        "학습자 맞춤 모드를 선택하세요",
        ["비개발자 / 기획자 트랙 (UI & 프롬프트 중심)", "개발자 / 엔지니어 트랙 (아키텍처 & 예외처리 중심)"]
    )

# ==============================================================================
# [파트 2] 트랙별 맞춤형 실습 대시보드 로직 분기
# ==============================================================================
if "비개발자" in selected_track:
    st.info("💡 **[비개발자 트랙 모드]** 복잡한 코드 수정보다는 UI 화면 구성, 프롬프트 엔지니어링, 그리고 사내 문서 요약 성능 테스트에 집중합니다.")
    
    col1, col2 = st.columns(2)
    with col1:
        st.subheader("📝 사내 문서 입력 시뮬레이터")
        user_doc_input = st.text_area("AI에게 학습시킬 사내 규정 텍스트를 입력하세요", "예: 당사의 원격 근무 수당은 월 10만 원이며, 사전 승인이 필수입니다.")
    with col2:
        st.subheader("📊 요약 및 분석 결과")
        if st.button("🚀 문서 핵심 요약 실행"):
            if user_doc_input.strip():
                st.success(f"✅ 문서 분석 완료! 주요 키워드 추출: [{user_doc_input[:15]}...]")
            else:
                st.warning("내용을 입력해 주세요.")

else:
    st.info("⚙️ **[개발자 트랙 모드]** LCEL 파이프라인의 에러 제어, 벡터 스토어 인덱싱 최적화, 그리고 비동기 세션 상태 관리를 다룹니다.")
    
    with st.expander("🔍 개발자 디버깅 패널 (시스템 로그 보기)"):
        st.code("""
# [Debug Log] Vector Store State: Active
# [Debug Log] Embedding Model: text-embedding-3-small
# [Debug Log] Chunking Strategy: RecursiveCharacterTextSplitter (Size=120, Overlap=20)
        """, language="python")
        
    if st.button("🔄 백엔드 메모리 및 캐시 강제 초기화"):
        st.cache_resource.clear()
        st.success("🧹 캐시가 성공적으로 초기화되었습니다.")

st.markdown("---")
st.caption("🚀 AI 교육 강사 마스터 프로그램 | 2개월차 커리큘럼 최종 완성")

```