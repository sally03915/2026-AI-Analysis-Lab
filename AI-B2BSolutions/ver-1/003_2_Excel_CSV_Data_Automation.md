## 📅 3개월차 2주차: Excel/CSV 데이터 자동화 및 보고서용 차트 자동 생성

> **[지난 주차 요약 및 연계 브리핑]**
> * **1주차 핵심:** 비정형 VOC 데이터를 전처리하고, 다중 라벨 태깅 및 우선순위 스코어링을 거쳐 자동 사과문 및 대응 시나리오를 JSON으로 출력하는 솔루션을 구축했습니다.
> * **2주차 연계:** 텍스트 데이터를 넘어, 실무에서 가장 빈번하게 다루는 **Excel 및 CSV 형태의 정형 데이터**를 LLM이 직접 읽고 분석하며, 파이썬 시각화 라이브러리와 연동해 보고서용 차트까지 자동 생성하는 데이터 에이전트 기술을 학습합니다.
> 
> 

---

## 📘 [이론 마크다운] `03_Excel_CSV_Data_Automation.md`

# 3개월차 2주차: 파이썬 Pandas와 LLM 데이터 에이전트 연동 및 자동 시각화

## 📌 1. 학습 목표

* 대용량 행/열 구조의 정형 데이터를 LLM이 이해하고 자연어 명령으로 제어할 수 있도록 돕는 프롬프팅 기법을 이해한다.
* Pandas DataFrame과 LLM을 결합하여 복잡한 데이터 필터링, 집계, 통계 추출 코드를 동적으로 생성하고 실행하는 파이프라인을 구축한다.
* `matplotlib` 및 `seaborn`을 활용해 비즈니스 보고서에 즉시 삽입 가능한 고품질 차트 이미지를 자동 생성하고 Streamlit에 표출한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)
| 시간대           | 소요 시간 | 활동 내용 |
|------------------|-----------|-----------|
| **00:00 ~ 00:50** | 50분 | **이론**: Pandas와 LLM 연동 아키텍처 및 데이터 에이전트 딥다이브 <br> - 정형 데이터 프롬프팅의 핵심 (스키마 정보 전달 및 코드 생성 패턴) <br> - 자연어 기반 데이터 필터링 및 통계 추출 원리 <br> - 시각화 자동화 파이프라인 설계 전략 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: Pandas DataFrame 연동 및 자연어 기반 통계 추출 실습 <br> - CSV 파일 로드 및 LLM을 통한 동적 데이터 쿼리 생성 <br> - 안전한 코드 실행 환경(Sandbox)에서의 데이터 가공 실습 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 데이터 시각화 자동화 파이프라인 및 Streamlit 대시보드 표출 완성 <br> - 매출 데이터 분석 후 차트 이미지를 자동 렌더링하는 실무형 모듈 구축 |


---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) 왜 Pandas와 LLM을 결합해야 하는가?

* **정형 데이터의 한계:** LLM 자체는 토큰 기반 언어 모델이므로 수십만 행의 숫자를 직접 계산하는 데 한계가 있고 환각이 발생할 수 있음.
* **Code Interpreter 패러다임:** LLM에게 직접 계산을 시키는 대신, 데이터의 구조(Schema)와 샘플 데이터를 보여주고 **Pandas 파이썬 코드를 작성**하게 만든 뒤, 이를 안전하게 실행하여 정확한 결과값을 도출하는 방식이 엔터프라이즈 데이터 분석의 표준임.

---

### 2) 자연어 쿼리에서 시각화까지의 원스톱 파이프라인

* 사용자가 "지난달 제품군별 판매 추이를 시각화해줘"라고 입력하면, LLM이 데이터프레임을 그룹화(`groupby`)하는 코드를 짜고 이어 `matplotlib`을 이용해 PNG 차트 파일로 저장하는 스크립트를 동적으로 완성.
* 생성된 이미지를 Streamlit 대시보드 UI에 즉시 바인딩하여 비개발자 임직원도 누쉽게 시각화 보고서를 얻을 수 있는 구조 설계.

---

## 💻 [코랩 실습 노트북] `03_Excel_Data_Automation_Master.py`

```python
# ==============================================================================
# 🚀 3개월차 2주차: Pandas & LLM 데이터 자동화 및 시각화 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 가상의 매출 CSV 데이터를 Pandas로 로드하고, 
# LLM을 통해 자연어 명령을 데이터 분석 코드로 변환 및 시각화하는 파이프라인을 검증합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 pandas matplotlib seaborn streamlit python-dotenv==1.0.1

import os
import io
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

print("📦 Pandas 및 데이터 시각화 자동화 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 가상 매출 데이터프레임 구축 및 자연어 분석 쿼리 연동 (강사 시간: 100분)
# ==============================================================================
# 샘플 사내 매출 데이터 생성
data = {
    "날짜": ["2026-06-01", "2026-06-02", "2026-06-03", "2026-06-04", "2026-06-05"],
    "제품군": ["A제품", "B제품", "A제품", "C제품", "B제품"],
    "매출액": [1500000, 2300000, 1200000, 4500000, 3100000],
    "수량": [10, 15, 8, 30, 20]
}
df_sales = pd.DataFrame(data)

print("📊 [가상 사내 매출 데이터프레임 스키마]:")
print(df_sales.info())

llm_data_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

def generate_pandas_query_code(df_info_str: str, user_natural_query: str) -> str:
    """
    사용자의 자연어 요청을 받아 이를 처리할 수 있는 Pandas 파이썬 코드를 생성합니다.
    """
    prompt = ChatPromptTemplate.from_template(
        """
        당신은 전문 데이터 분석가 파이썬 개발자입니다. 
        아래의 Pandas DataFrame 정보(`df`)와 사용자의 자연어 요청을 바탕으로, 
        요청을 수행하기 위한 순수 파이썬/Pandas 코드만 작성해 주세요. (마크다운 백틱 ```python 및 ``` 형식으로 출력)
        변수명은 반드시 `df`를 사용하고, 최종 결과물은 `result` 변수에 담아주세요.
        
        [데이터프레임 정보]
        {df_info}
        
        [사용자 자연어 요청]
        {query}
        """
    )
    chain = prompt | llm_data_engine | StrOutputParser()
    code_result = chain.invoke({"df_info": df_info_str, "query": user_natural_query})
    return code_result

# 테스트 쿼리 실행
user_query = "A제품의 총 매출액 합계를 구해줘"
print(f"\n🔍 [사용자 자연어 질의]: {user_query}")

generated_code = generate_pandas_query_code(str(df_sales.dtypes), user_query)
print(f"💻 [LLM이 생성한 Pandas 분석 코드]:\n{generated_code}")


# ==============================================================================
# [실습 3] [미션] 데이터 시각화 자동화 파이프라인 구현 (강사 시간: 50분)
# ==============================================================================
def create_automated_sales_chart(df: pd.DataFrame) -> str:
    """
    데이터를 기반으로 시각화 차트를 그리고 이미지 파일로 저장하는 자동화 함수
    """
    print("\n📈 [시각화 자동화 엔진] Seaborn/Matplotlib을 이용한 리포트 차트 생성 중...")
    
    plt.figure(figsize=(8, 5))
    sns.barplot(data=df, x="제품군", y="매출액", palette="Blues_d")
    plt.title("2026년 상반기 제품군별 매출 현황 리포트")
    plt.xlabel("제품 카테고리")
    plt.ylabel("매출액 (KRW)")
    
    chart_path = "sales_report_chart.png"
    plt.savefig(chart_path, bbox_inches="tight")
    plt.close()
    
    return chart_path

saved_chart_file = create_automated_sales_chart(df_sales)
print(f"✨ [시각화 완료] 보고서용 차트 이미지 저장 경로: {saved_chart_file}")

print("\n✅ [3개월차 2주차 검증 완료] Excel/CSV 정형 데이터 분석 및 차트 자동 생성 파이프라인이 성공적으로 구현되었습니다.")

```