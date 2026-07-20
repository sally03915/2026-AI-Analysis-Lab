## 📅 3개월차 4주차: 교육용 라이브러리 자산화 및 프롬프트 북 패키징

> **[지난 주차 요약 및 연계 브리핑]**
> * **3주차 핵심:** SQLite 가상 샌드박스를 활용하여 자연어 질의를 SQL로 변환하고, 데이터베이스 분석 자동화 쿼리를 실행하는 Text-to-SQL 파이프라인을 구축했습니다.
> * **4주차 연계:** 3개월차 비즈니스 솔루션 트랙의 마지막 주차로서, 실습 과정에서 다룬 민감한 사내 데이터를 보호하는 **비식별화(Anonymization)** 기술을 익히고, 지금까지 제작한 코드를 조립식 모듈로 리팩토링하여 독점 수강생 배포용 **'사내 업무 자동화 3대 프롬프트 북'** 마스터 패키지를 완성합니다.
> 
> 

---

## 📘 [이론 마크다운] `03_Prompt_Book_and_Anonymization.md`

# 3개월차 4주차: 기업 기밀 정보 비식별화와 프롬프트 북 패키징

## 📌 1. 학습 목표

* 클라우드 LLM API 호출 시 발생할 수 있는 기업 기밀 및 개인정보 유출을 방지하기 위한 비식별화(Anonymization) 가이드라인과 정규식 기반 마스킹 기법을 이해한다.
* 1주차부터 3주차까지 개발한 챗봇, 데이터 분석, Text-to-SQL 솔루션 코드를 조립식 모듈(Function) 형태로 리팩토링한다.
* 수강생과 실무 임직원들이 즉시 현업에 적용할 수 있는 '사내 업무 자동화 3대 프롬프트 북' 문서를 최종 패키징한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)
| 시간대           | 소요 시간 | 활동 내용 |
|------------------|-----------|-----------|
| **00:00 ~ 00:50** | 50분 | **이론**: 기업 보안 관점의 LLM 기밀 정보 비식별화 아키텍처 딥다이브 <br> - 정규식(Regex)을 이용한 주민번호, 연락처, 사내 자산명 자동 마스킹 원리 <br> - 모듈화된 파이썬 코드의 패키징 및 재사용성 극대화 전략 <br> - 실무 비즈니스 프롬프트 북 구조화 방법론 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: 개인정보 및 사내 자산 비식별화 필터링 함수 구현 <br> - 정규 표현식을 활용한 텍스트 가공 파이프라인 실습 <br> - 1~3주차 주요 솔루션 코드의 모듈화 리팩토링 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 독점 수강생 배포용 '사내 업무 자동화 3대 프롬프트 북' 완성 <br> - VOC 분석, 데이터 시각화, Text-to-SQL 템플릿이 모두 수록된 최종 통합 가이드북 마감 |


---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) 왜 사내 기밀 비식별화(Anonymization)가 필수인가?

* **보안 리스크 차단:** 기업 임직원들이 사내 소스코드나 고객 개인정보, 미공개 매출 데이터를 프롬프트에 그대로 입력해 외부 LLM API로 전송할 경우 심각한 보안 유출 사고로 이어짐.
* **전처리 마스킹 계층(Masking Layer):** API로 요청이 넘어가기 전, 정규식(Regex) 엔진을 거쳐 민감한 키워드(이름, 전화번호, 사내 프로젝트 코드 등)를 `[REDACTED_USER]`, `[PROJECT_CODE]` 형태로 자동 치환한 뒤 안전하게 전송하는 가드레일 구축 필수.

---

### 2) 프롬프트 북(Prompt Book) 자산화의 가치

* 일회성 실습 코드로 끝나지 않고, 비개발자 직군도 사내에서 복사-붙여넣기만으로 즉시 활용할 수 있도록 템플릿화된 **'3대 비즈니스 프롬프트 북'**(고객 응대 자동화, 정형 데이터 시각화, 데이터베이스 쿼리 생성) 문서 패키징 실무 노하우 전수.

---

## 💻 [코랩 실습 노트북] `03_Prompt_Book_Packaging_Master.py`

```python
# ==============================================================================
# 🚀 3개월차 4주차: 비식별화 필터링 및 프롬프트 북 패키징 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 사내 기밀 및 개인정보를 외부 LLM 전송 전에 
# 자동으로 마스킹하는 비식별화 필터링 로직과 최종 프롬프트 북 구조를 검증합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 langchain-core==0.2.33 python-dotenv==1.0.1

import os
import re
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

print("📦 비식별화 및 프롬프트 북 패키징 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 정규식 기반 사내 기밀 및 개인정보 비식별화(Anonymization) 함수 구현 (강사 시간: 100분)
# ==============================================================================
def sanitize_sensitive_data(raw_text: str) -> str:
    """
    텍스트 내에 포함된 전화번호, 이메일, 사내 기밀 코드(예: SEC-XXX)를 마스킹하여 비식별화합니다.
    """
    print("🔒 [보안 가드레일] 사내 기밀 및 개인정보 비식별화(Anonymization) 필터 가동 중...")
    
    # 1. 이메일 주소 마스킹
    masked_text = re.sub(r'[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+', '[EMAIL_REDACTED]', raw_text)
    
    # 2. 전화번호 마스킹 (010-0000-0000 형태)
    masked_text = re.sub(r'\d{2,3}-\d{3,4}-\d{4}', '[PHONE_REDACTED]', masked_text)
    
    # 3. 사내 기밀 프로젝트/보안 코드 마스킹 (예: SEC-402)
    masked_text = re.sub(r'SEC-\d+', '[CONFIDENTIAL_CODE]', masked_text)
    
    return masked_text

# 비식별화 테스트 실행 시뮬레이션
sample_raw_memo = "김철수 책임 (kim.chulsoo@company.com, 010-9999-8888)이 보안 규정 SEC-402 위반 건에 대해 보고함."
sanitized_memo = sanitize_sensitive_data(sample_raw_memo)

print(f"\n📝 [원문 데이터]: {sample_raw_memo}")
print(f"🛡️ [비식별화 마스킹 완료 데이터]: {sanitized_memo}")


# ==============================================================================
# [실습 3] [미션] 3대 업무 자동화 프롬프트 북 최종 패키징 검증 (강사 시간: 50분)
# ==============================================================================
llm_prompt_book_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

def run_prompt_book_template(template_type: str, safe_input_data: str) -> str:
    """
    3대 프롬프트 북(VOC 자동응답, 데이터 요약, Text-to-SQL) 중 선택된 템플릿을 실행합니다.
    """
    print(f"\n📖 [프롬프트 북 가동] '{template_type}' 솔루션 모듈 실행 중...")
    
    templates = {
        "VOC": "당신은 전문 CS 응대 매니저입니다. 다음 고객 민원을 바탕으로 공감형 정중한 사과문 초안을 작성해주세요:\n{data}",
        "DATA": "당신은 수석 데이터 분석가입니다. 다음 정형 데이터 내용을 바탕으로 핵심 비즈니스 인사이트 3가지를 도출해주세요:\n{data}",
        "SQL": "당신은 데이터베이스 전문가입니다. 다음 요구사항을 바탕으로 실행 가능한 SQLite 쿼리문을 작성해주세요:\n{data}"
    }
    
    selected_template = templates.get(template_type, "다음 내용을 요약해주세요:\n{data}")
    chain = ChatPromptTemplate.from_template(selected_template) | llm_prompt_book_engine | StrOutputParser()
    
    return chain.invoke({"data": safe_input_data})

# 프롬프트 북 패키징 검증 테스트
prompt_book_result = run_prompt_book_template("VOC", sanitized_memo)
print(f"✨ [프롬프트 북 실행 결과 리포트]:\n{prompt_book_result}")

print("\n✅ [3개월차 4주차 최종 검증 완료] 기밀 정보 비식별화 가드레일 및 '사내 업무 자동화 3대 프롬프트 북' 패키징이 성공적으로 완료되었습니다!")

```