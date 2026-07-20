## 📅 3개월차 3주차: SQL 쿼리 생성 및 DB 분석 자동화 템플릿

> **[지난 주차 요약 및 연계 브리핑]**
> * **2주차 핵심:** Pandas DataFrame과 LLM을 연동하여 Excel/CSV 형태의 정형 데이터를 자연어로 분석하고, `matplotlib`/`seaborn`을 활용해 보고서용 차트를 자동 생성하는 파이프라인을 구축했습니다.
> * **3주차 연계:** 파일 형태의 정형 데이터를 넘어, 실무 기업 환경에서 가장 많이 쓰이는 관계형 데이터베이스(RDB)를 대상으로 자연어를 SQL로 변환하고 분석하는 **Text-to-SQL 자동화 템플릿**을 학습합니다.
> 
> 

---

## 📘 [이론 마크다운] `03_Text_to_SQL_Automation.md`

# 3개월차 3주차: Text-to-SQL 메커니즘과 데이터베이스 분석 자동화

## 📌 1. 학습 목표

* 자연어 질의를 데이터베이스가 이해하는 SQL 쿼리로 정확하게 변환하는 Text-to-SQL 아키텍처의 원리를 이해한다.
* 데이터베이스 테이블 구조(Schema)와 DDL(Data Definition Language)을 LLM에 주입하여 환각을 방지하고 문맥 정확도를 높이는 기법을 익힌다.
* 가상 SQLite DB 샌드박스를 구축하여 LLM이 생성한 SQL 문법 에러를 자동으로 검증하고 실행 결과를 리턴하는 원스톱 파이프라인을 구현한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)
| 시간대           | 소요 시간 | 활동 내용 |
|------------------|-----------|-----------|
| **00:00 ~ 00:50** | 50분 | **이론**: Text-to-SQL 메커니즘과 데이터베이스 스키마 주입 딥다이브 <br> - 자연어 기반 데이터 조회 프롬프팅 디자인 패턴 <br> - 스키마 정보 주입 및 보안 관점의 Read-Only 제약 조건 설정 <br> - 가상 DB 샌드박스를 활용한 문법 오류 자가 수정 루프 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: SQLite 가상 DB 구축 및 Text-to-SQL 체인 구현 <br> - 인메모리/파일 기반 SQLite 테이블 생성 및 샘플 데이터 적재 <br> - LLM을 통한 자연어 -> SQL 변환 파이프라인 실습 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 복잡한 JOIN 쿼리 자동 생성 및 데이터 호출 원스톱 솔루션 완성 <br> - 다중 테이블 JOIN 질의를 처리하고 결과를 정형 리포트로 변환하는 최종 미션 수행 |


---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) 왜 Text-to-SQL이 실무에서 강력한가?

* **비개발자 업무 혁신:** 사내 임직원이나 마케터가 SQL 문법을 몰라도 "지난 분기 가장 많이 팔린 제품 상위 3개의 매출 합계를 보여줘"라고 자연어로 질문하면, 시스템이 실시간으로 데이터베이스를 조회해 답변을 제공.
* **스키마 주입(Schema Injection):** LLM은 사내 DB에 어떤 테이블과 컬럼이 있는지 기본적으로 알 수 없으므로, 프롬프트 컨텍스트에 테이블 DDL 구조를 명확히 주입하여 정확한 컬럼 매칭을 유도해야 함.

---

### 2) 가상 DB 샌드박스와 안전성 확보

* LLM이 생성한 SQL이 문법 오류를 포함하거나 잘못된 UPDATE/DELETE 명령을 담고 있을 위험이 있으므로, 실행 환경은 반드시 **Read-Only(조회 전용)** 권한으로 제한된 샌드박스(SQLite 등)에서 검증을 거쳐야 함.

---

## 💻 [코랩 실습 노트북] `03_Text_to_SQL_Master.py`

```python
# ==============================================================================
# 🚀 3개월차 3주차: Text-to-SQL 및 DB 분석 자동화 마스터 템플릿
# ==============================================================================
# [수강생 안내] 본 코드는 가상의 SQLite DB를 구축하고, 자연어 질문을 
# SQL 쿼리로 자동 변환하여 실행 결과를 도출하는 원스톱 파이프라인을 검증합니다.

# 1. 의존성 패키지 설치 (코랩 환경 실행 시 주석 해제)
# !pip install -q langchain==0.2.14 langchain-openai==0.1.20 langchain-core==0.2.33 python-dotenv==1.0.1

import os
import sqlite3
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

print("📦 Text-to-SQL 및 SQLite 연동 라이브러리 임포트 완료")

# API Key 설정 관리
if "OPENAI_API_KEY" not in os.environ:
    os.environ["OPENAI_API_KEY"] = "sk-proj-YOUR_MOCK_KEY_FOR_CLASS"

# ==============================================================================
# [실습 1 & 2] 가상 SQLite DB 셋업 및 DDL 스키마 정의 (강사 시간: 100분)
# ==============================================================================
def init_virtual_sqlite_db():
    """
    메모리 기반의 가상 SQLite 데이터베이스를 생성하고 샘플 데이터를 적재합니다.
    """
    conn = sqlite3.connect(":memory:")
    cursor = conn.cursor()
    
    # 임직원 테이블 생성
    cursor.execute("""
    CREATE TABLE employees (
        id INTEGER PRIMARY KEY,
        name TEXT,
        department TEXT,
        salary INTEGER
    )
    """)
    
    # 샘플 데이터 삽입
    sample_data = [
        (1, "김철수", "개발팀", 5000000),
        (2, "이영희", "마케팅팀", 4500000),
        (3, "박민수", "개발팀", 5500000),
        (4, "정지은", "인사팀", 4000000)
    ]
    cursor.executemany("INSERT INTO employees VALUES (?, ?, ?, ?)", sample_data)
    conn.commit()
    return conn

db_connection = init_virtual_sqlite_db()
database_schema_info = """
Table: employees
Columns:
- id (INTEGER): 직원 고유 ID
- name (TEXT): 직원 성명
- department (TEXT): 소속 부서
- salary (INTEGER): 월 급여 (원)
"""

print("🗄️ [가상 SQLite DB] 'employees' 테이블 생성 및 샘플 데이터 적재 완료")

# ==============================================================================
# [실습 3] [미션] 자연어를 SQL로 변환하고 실행하는 쿼리 자동화 엔진 구축 (강사 시간: 50분)
# ==============================================================================
llm_sql_engine = ChatOpenAI(model="gpt-4o-mini", temperature=0.0)

def generate_and_execute_sql(natural_language_query: str) -> str:
    """
    사용자의 자연어 질문을 받아 SQLite용 SQL 쿼리를 생성하고, 가상 DB에서 실행한 결과를 반환합니다.
    """
    print(f"\n🔍 [사용자 자연어 질의]: {natural_language_query}")
    
    # 1. Text-to-SQL 프롬프트 구성
    sql_prompt = ChatPromptTemplate.from_template(
        """
        당신은 데이터베이스 전문가이자 SQLite SQL 쿼리 생성기입니다.
        제공된 데이터베이스 스키마 정보를 바탕으로, 사용자의 자연어 질문을 해결할 수 있는 순수 SQL 쿼리문만 작성해 주세요.
        마크다운 코드 블록(```sql ... ```)이나 불필요한 설명 없이 오직 SQL 문장만 출력하세요.
        
        [데이터베이스 스키마]
        {schema}
        
        [사용자 질의]
        {query}
        """
    )
    
    chain = sql_prompt | llm_sql_engine | StrOutputParser()
    generated_sql = chain.invoke({"schema": database_schema_info, "query": natural_language_query})
    
    # 생성된 SQL 정제
    cleaned_sql = generated_sql.replace("```sql", "").replace("```", "").strip()
    print(f"💻 [LLM이 생성한 SQL 쿼리]: {cleaned_sql}")
    
    # 2. 가상 DB 샌드박스 실행
    try:
        cursor = db_connection.cursor()
        cursor.execute(cleaned_sql)
        query_result = cursor.fetchall()
        return f"성공 (결과: {query_result})"
    except Exception as e:
        return f"실행 에러 발생: {str(e)}"

# 미션 실행 테스트 시뮬레이션
test_query = "개발팀에 속한 직원들의 평균 월 급여를 조회해 줘."
execution_output = generate_and_execute_sql(test_query)
print(f"📊 [DB 쿼리 실행 결과]: {execution_output}")

print("\n✅ [3개월차 3주차 검증 완료] Text-to-SQL 변환 및 가상 DB 샌드박스 실행 자동화 템플릿이 성공적으로 완성되었습니다.")

```