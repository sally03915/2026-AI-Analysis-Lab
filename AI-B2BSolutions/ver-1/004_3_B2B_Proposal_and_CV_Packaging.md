## 📅 4개월차 3주차: 강사 프로필 및 과정 제안서 최고급 패키징

> **[지난 주차 연계 브리핑]**
> * **4개월차 2주차 핵심:** Azure OpenAI 온프레미스 하이브리드 아키텍처, Prompt Caching 비용 최적화, 그리고 Advanced RAG 옵저버빌리티를 다루는 IT/SI 대기업 개발자 타겟 심화 트랙을 완성했습니다.
> 
> 
> * **4주차 연계:** 이번 주차는 1~3개월차 동안 갈고닦은 강력한 기술 솔루션과 4개월차의 비개발자/개발자 투트랙 교육 상품을 **B2B 시장에서 즉시 결재를 받을 수 있는 최고급 비즈니스 자산(강사 CV, 포트폴리오, 제안서)**으로 완벽하게 패키징합니다.
> 
> 

---

## 📘 [이론 마크다운] `04_B2B_Proposal_and_CV_Packaging.md`

# 4개월차 3주차: HRD 담당자를 매료시키는 강사 프로필 및 B2B 제안서 패키징

## 📌 1. 학습 목표

* IT 엔지니어 및 개발자 출신의 이력을 HRD(인적자원개발) 담당자와 기업 교육 구매 담당자가 선호하는 B2B 교육용 키워드로 리브랜딩하는 법을 익힌다.
* 1~3개월차에 클라우드(Streamlit Cloud 등)에 배포한 실무 데모 웹 애플리케이션 링크들을 원스톱 디지털 포트폴리오로 구조화한다.
* 예산 편성과 결재 라인 통과를 극대화하는 표준 B2B 교육 제안서 및 견적서 패키징 공식을 마스터한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| --- | --- | --- |
| **00:00 ~ 00:50** | 50분 | **이론**: HRD 담당자의 마음을 훔치는 B2B 강사 프로필(CV) 리브랜딩 전략 <br>

<br> - 기술 용어 중심의 이력을 교육 성과 및 비즈니스 임팩트 중심으로 전환 <br>

<br> - 1~3개월차 클라우드 데모 링크를 활용한 신뢰성 있는 디지털 포트폴리오 구성법 <br>

<br> - 기업 교육 예산 구조 이해 및 제안서 스토리텔링 공식 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: B2B 강사 CV 구조화 및 디지털 포트폴리오 매핑 실습 <br>

<br> - 핵심 역량 키워드 추출 및 강의 경력 기술서 작성 <br>

<br> - Streamlit 데모 웹 링크 연동 원스톱 포트폴리오 템플릿 완성 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 즉시 결재가 가능한 B2B 교육 제안서 및 견적서 패키징 <br>

<br> - 교육 목표, 커리큘럼, 기대 효과, 강사료 단가 산정 기준이 포함된 마스터 제안서 완성 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) HRD 담당자를 매료시키는 강사 프로필(CV)의 조건

* **비즈니스 언어 번역:** "LangChain과 Chroma DB를 다룬다"는 표현을 HRD 담당자가 이해하기 쉬운 "사내 문서 기반 RAG 챗봇 구축 및 업무 자동화 실무 역량 내재화"로 치환하여 어필해야 함.
* **증명 가능한 포트폴리오:** 말로만 하는 강사가 아니라, 수강생들이 직접 눈으로 확인하고 테스트할 수 있는 **Streamlit 라이브 데모 웹 링크**를 포트폴리오 상단에 배치하여 강의 신뢰도를 극대화함.

---

### 2) 결재를 부르는 B2B 제안서의 3대 요소

* **명확한 Pain Point 제기:** 기업이 겪고 있는 반복 업무 비효율, 보안 우려, AI 도입 지연 문제를 정확히 짚어냄.
* **ROI(투자 대비 효과) 중심의 기대 효과:** 교육 이수 후 임직원들의 업무 단축 시간과 생산성 향상 지표를 가시화.
* **투명한 견적 및 운영 프로세스:** 모듈별 시간, 교재 제공 여부, 사후 피드백 체계가 담긴 깔끔한 견적서 동봉.

---

## 💻 [코랩 실습 노트북] `04_B2B_Proposal_Packaging.py`

```python
# ==============================================================================
# 🚀 4개월차 3주차: B2B 강사 프로필 및 교육 제안서 자동 패키징 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 보유한 기술 스택과 Streamlit 배포 데모 링크를 결합하여,
# 대형 교육 에이전시 및 기업 HRD 담당자에게 즉시 발송 가능한 B2B 제안서 데이터를 생성합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip idnstall -q langchain-openai==0.1.20 langchain-core==0.2.33 python-dotenv==1.0.1

import os
import json
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

print("📦 B2B 제안서 및 CV 패키징 실습 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 기술 이력을 B2B 교육 강사 CV 및 포트폴리오로 리브랜딩 (강사 시간: 100분)
# ==============================================================================
def generate_rebranded_cv_profile(tech_stack_list: list, demo_links: dict) -> str:
    """
    개발자/엔지니어의 기술 스택을 기업 HRD 교육용 프로필 키워드로 리브랜딩하고,
    클라우드 데모 포트폴리오 링크를 체계적으로 매핑합니다.
    """
    print("💼 [강사 프로필 리브랜딩] 기술 스택을 B2B 교육용 역량으로 변환 중...")
    
    cv_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.2)
    
    cv_prompt = ChatPromptTemplate.from_messages([
        ("system", 
         "당신은 IT 전문 교육 에이전시의 수석 커리어 컨설턴트입니다. "
         "제시된 기술 스택과 데모 링크를 바탕으로, 대기업 HRD 담당자의 시선을 사로잡을 수 있는 "
         "프로페셔널 강사 소개서(CV) 요약본과 포트폴리오 서문을 작성해주세요."
        ),
        ("user", "보유 기술 스택: {techs}\n실무 데모 링크 모음: {links}")
    ])
    
    chain = cv_prompt | cv_engine | StrOutputParser()
    result = chain.invoke({
        "techs": ", ".join(tech_stack_list),
        "links": json.dumps(demo_links, ensure_ascii=False, indent=2)
    })
    return result

# 입력 데이터 시뮬레이션
my_tech_stacks = ["OpenAI API", "LangChain", "Advanced RAG", "Streamlit", "Pandas & Text-to-SQL", "Azure OpenAI"]
my_portfolio_links = {
    "FAQ 챗봇": "https://company-faq-demo.streamlit.app",
    "RAG 사내 문서 검색기": "https://enterprise-rag.streamlit.app",
    "데이터 분석 자동화": "https://biz-analytics-agent.streamlit.app"
}

rebranded_cv = generate_rebranded_cv_profile(my_tech_stacks, my_portfolio_links)
print(f"\n📄 [리브랜딩된 강사 CV 및 포트폴리오 요약]:\n{rebranded_cv}")


# ==============================================================================
# [실습 3] [미션] 즉시 결재가 가능한 B2B 교육 제안서 및 견적서 패키징 (강사 시간: 50분)
# ==============================================================================
def generate_b2b_proposal_package(course_title: str, target_audience: str) -> str:
    """
    기업 HRD 결재 라인을 통과하기 위한 마스터 B2B 교육 제안서 및 견적 구조를 출력합니다.
    """
    print(f"\n📋 [B2B 제안서 패키징] '{course_title}' 과정 제안서 및 견적 구조 생성 중...")
    
    proposal_template = f"""
    ==================================================
    [B2B 공식 교육 제안서 및 견적서 패키지]
    ==================================================
    1. 과정명: {course_title}
    2. 교육 대상: {target_audience}
    3. 교육 기대 효과 (ROI): 
       - 반복 업무 소요 시간 70% 단축 및 임직원 AI 리터러시 즉시 확보
       - 사내 보안 가이드라인을 준수하는 자체 AI 자동화 솔루션 구축 완료
    4. 커리큘럼 구성 (총 4주 / 12시간 완성 패키지):
       - Week 1: 핵심 API 통제 및 비정형 데이터 정형화
       - Week 2: RAG 기반 사내 문서 검색 및 웹 배포
       - Week 3: 데이터 분석 자동화 및 DB 쿼리 연동
       - Week 4: 실무 에이전트 확장 및 현업 적용 워크숍
    5. 표준 견적 및 운영 조건:
       - 강사료 및 교재비 포함 (기업 맞춤형 커스텀 튜닝 별도 협의)
       - 실습용 코랩 노트북 및 독점 프롬프트 북 전원 제공
    ==================================================
    """
    return proposal_template.strip()

b2b_package = generate_b2b_proposal_package(
    course_title="생성형 AI 기반 실무 업무 자동화 & 엔터프라이즈 솔루션 빌드", 
    target_audience="대기업 및 공공기관 실무 임직원 / IT 기획자"
)
print(f"\n{b2b_package}")

print("\n✅ [4개월차 3주차 검증 완료] 강사 프로필 리브랜딩과 B2B 교육 제안서 패키징이 성공적으로 완료되었습니다!")

```