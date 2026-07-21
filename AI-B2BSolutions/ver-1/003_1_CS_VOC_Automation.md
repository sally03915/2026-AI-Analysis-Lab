## 📅 3개월차 1주차: CS/고객센터 자동 응답 및 감정 분석 챗봇

> **[지난 주차 요약 및 연계 브리핑]**
> * **2개월차 핵심:** LangChain LCEL 파이프라인, Chroma DB 기반 RAG 구축, Streamlit Cloud 웹 앱 프로덕션 배포까지 엔터프라이즈 RAG 아키텍처를 완성했습니다.
> * **3주차 연계:** 3개월차 비즈니스 솔루션 트랙에서는 문서 검색을 넘어, 기업 현업에서 가장 빈번하게 발생하는 **고객센터 VOC(Voice of Customer) 대응, 정형/비정형 데이터 분석, 데이터베이스 자동화** 등 실무 직무 효율을 극대화하는 자동화 솔루션 구축에 집중합니다. 첫 주차인 이번 시간에는 인입되는 불만 민원을 자동으로 분류하고, 맞춤형 사과문 초안을 JSON 형태로 출력하는 시스템을 구축합니다.
> 
> 

---

## 📘 [이론 마크다운] `03_CS_VOC_Automation.md`

# 3개월차 1주차: 비정형 VOC 전처리 및 JSON 모드 기반 감정 분석 자동화

## 📌 1. 학습 목표

* 텍스트 형태의 고객 불만 및 문의(VOC) 데이터를 전처리하고, LLM의 프롬프트 구조화를 통해 핵심 키워드를 추출하는 기법을 익힌다.
* 다중 라벨(Multi-label) 태깅과 우선순위 스코어링(Urgency Scoring)을 적용하여 악성 민원과 일반 문의를 자동으로 식별한다.
* OpenAI의 최신 `response_format: {"type": "json_object"}` 기능을 활용하여, 후속 시스템(CRM, 메일 발송 API 등)과 완벽하게 연동되는 정형화된 JSON 형태의 사과문 및 대응 시나리오를 출력한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| --- | --- | --- |
| **00:00 ~ 00:50** | 50분 | **이론**: 비정형 VOC 데이터 처리 메커니즘과 JSON 모드 딥다이브 <br>

<br> - 사내 고객 CS 데이터의 구조적 특징과 전처리 필요성 <br>

<br> - 감정 분석(Sentiment Analysis) 및 우선순위 스코어링 프롬프트 디자인 패턴 <br>

<br> - `json_object` 모드를 활용한 파싱 에러 제로화 원리 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: VOC 분류 프롬프트 엔지니어링 및 다중 라벨 태깅 구현 <br>

<br> - 인입 메일 데이터 클리닝 및 감정 점수 산정 실습 <br>

<br> - 카테고리(배송, 품질, 환불) 동시 분류 로직 구현 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 자동 사과문 및 대응 시나리오 회신 엔진 완성 <br>

<br> - 분석된 VOC 결과를 바탕으로 현업 실무자가 즉시 발송 가능한 JSON 포맷의 대응 초안 생성 시스템 구축 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) 왜 사내 VOC 자동화가 비즈니스 혁신인가?

* **고객 응대 속도 극대화:** 하루 수백 건씩 인입되는 고객 불만 메일이나 리뷰를 CS 팀원이 일일이 읽고 분류하는 데 막대한 리소스 소모.
* **AI 기반 선제적 분류:** 인입 즉시 AI가 감정 상태(긍정/중립/악성)와 긴급도를 평가하고, 카테고리를 자동 태깅하여 담당 부서에 실시간 라우팅 및 1차 초안까지 작성해 주는 파이프라인 구축.

---

### 2) JSON 모드를 통한 시스템 연동 안정성

* LLM이 일반 텍스트로 답변을 출력하면 파이썬 코드에서 이를 파싱(Parsing)할 때 서식 오류가 잦음.
* `response_format={"type": "json_object"}` 설정을 강제하면, LLM이 반드시 유효한 JSON 스키마 구조로만 답변을 반환하므로, 후속 자동화 시스템(예: 사내 웹훅, 이메일 자동 발송 모듈)과 에러 없이 안전하게 연동 가능.

---

## 💻 [코랩 실습 노트북] `03_CS_VOC_Automation_Master.py`

```python
# ==============================================================================
# 🚀 3개월차 1주차: CS VOC 감정 분석 및 JSON 자동 응답 챗봇 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 인입된 고객 불만(VOC) 비정형 데이터를 전처리하고,
# 다중 라벨 태깅 및 우선순위 스코어링을 거쳐 JSON 형태의 사과문과 대응 시나리오를 출력합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 langchain-core==0.2.33 python-dotenv==1.0.1

import os
import json
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

print("📦 CS VOC 자동화 실습 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 비정형 VOC 데이터 클리닝 및 다중 라벨 태깅 프롬프트 셋업 (강사 시간: 100분)
# ==============================================================================
llm_voc_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

def analyze_customer_voc(raw_customer_email: str) -> dict:
    """
    고객의 인입 메일을 분석하여 감정 상태, 카테고리, 긴급도, 그리고 맞춤형 사과문 초안을 
    완벽한 JSON 포맷으로 반환합니다.
    """
    print(f"📥 [VOC 인입] 원문 분석 시작...")
    
    # 시스템 프롬프트에 JSON 출력 강제 규칙 명시
    voc_prompt = ChatPromptTemplate.from_messages([
        ("system", 
         "당신은 엔터프라이즈 기업의 최고 고객 책임자(CCO) 및 CS 자동화 에이전트입니다. "
         "전달된 고객의 불만 메일을 분석하여 다음 규칙에 맞는 JSON 객체 형식으로만 응답하세요.\n"
         "반드시 아래의 키(Key) 값을 포함해야 합니다:\n"
         "- sentiment (str): '긍정', '중립', '악성(분노)' 중 택 1\n"
         "- category (list): ['배송', '품질', '환불', '기타'] 중 해당하는 항목들\n"
         "- urgency_score (int): 1부터 5까지의 긴급도 점수 (5가 가장 긴급)\n"
         "- summary (str): 불만 내용 핵심 요약\n"
         "- response_draft (str): 고객에게 발송할 공감형 정중한 사과문 및 대응 안내 초안"
        ),
        ("user", "다음 고객 인입 메일을 분석해주세요:\n\n{email_content}")
    ])
    
    # response_format을 JSON 객체로 지정하여 안전성 확보
    # (LangChain 최신 버전 또는 ChatOpenAI 설정 연동)
    chain = voc_prompt | llm_voc_engine.bind(response_format={"type": "json_object"}) | StrOutputParser()
    
    raw_json_response = chain.invoke({"email_content": raw_customer_email})
    
    # 텍스트로 반환된 JSON 문자열을 파이썬 딕셔너리로 파싱
    parsed_result = json.loads(raw_json_response)
    return parsed_result


# ==============================================================================
# [실습 3] [미션] 실무 CS 메일 시뮬레이션 및 결과 확인 (강사 시간: 50분)
# ==============================================================================
sample_voc_email = """
주문번호 #2026-9812 상품 배송 관련해서 항의합니다. 
분명히 지난주에 결제하면서 이번 주 월요일까지는 무조건 도착해야 한다고 말씀드렸는데, 
오늘 화요일인데도 배송조회조차 안 되고 물건이 어디 있는지 알 수가 없네요! 
당장 행사 써야 하는데 다 망쳤습니다. 빠른 환불 처리와 보상안 내놓지 않으면 소비자원에 신고하겠습니다.
"""

print(f"\n📨 [테스트 대상 고객 메일 내용]:\n{sample_voc_email}")

# 자동 분석 엔진 실행
analysis_result = analyze_customer_voc(sample_voc_email)

print("\n📊 [AI VOC 분석 및 대응 솔루션 결과 (JSON Output)]:")
print(json.dumps(analysis_result, indent=4, ensure_ascii=False))

print("\n✅ [3개월차 1주차 검증 완료] CS VOC 비정형 데이터 분석 및 JSON 자동 응답 회신 엔진이 정상 작동합니다.")

```