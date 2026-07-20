## 📘 [이론 마크다운] `01_B2B_Delivery_Troubleshooting.md`
 
### 1개월차 4주차: 강의 시뮬레이션 및 실전 Q&A 방어 전략

## 📌 1. 학습 목표
- B2B 기업 출강에 특화된 Hook-Body-Close 데모 구조를 이해하고, 비개발자도 직관적으로 이해할 수 있는 라이브 코딩 비유법을 습득한다.
- 기업 사내 방화벽, 프록시(Proxy) 서버 및 망분리 환경에서 발생하는 API 통신 거부 현상을 이해하고, 네트워크 우회 프로토콜 설정을 코드로 구현할 수 있다.
- 1~3주차 통합 코드 자산을 유기적으로 결합하여, 실무에서 즉시 시연 가능한 15분 분량의 마스터 강의 데모 파이프라인과 트러블슈팅 시나리오를 완성한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: B2B 교수법 아키텍처 및 기업 사내 보안망 대응 기술 <br> - 청중의 몰입도를 극대화하는 3단계(Hook-Body-Close) 강의 설계 <br> - 프록시 서버(HTTP/HTTPS Proxy) 및 SSL 인증서 검증 우회 매커니즘 <br> - 실전 출강 시 강사를 당황하게 만드는 인프라 에러 유형 분석 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: 프록시 네트워크 통제 및 모킹(Mocking) 테스트 <br> - `os.environ` 및 `httpx` 클라이언트 기반 프록시 터널링 가상 환경 구축 <br> - 네트워크 단절 상황을 가정한 백엔드 타임아웃(Timeout) 방어벽 설계 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 1~3주차 통합 15분 마스터 강의 데모 빌드 <br> - 수강생 개별 라이브 코딩 및 에러 핸들링 종합 스크립트 작성 <br> - 강사 피드백 및 인프라 트러블슈팅 최종 체크리스트 확정 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) B2B 출강 특화 교수법 및 엔터프라이즈 강의 설계
개발 스택을 전달하는 것을 넘어, 기업 임직원과 HRD 담당자에게 '실무 적용 가능성'을 시각적으로 증명하는 전달 아키텍처입니다.

*   **Hook-Body-Close 데모 구조**
    *   **Hook (5분):** 시작과 동시에 작동 가능한 완성형 Streamlit UI 데모를 먼저 보여주어 "오늘 수업이 끝나면 여러분도 이걸 만듭니다"라는 명확한 보상을 제시, 몰입도 유도.
    *   **Body (70분):** 전체 코드를 한 번에 주지 않고 뼈대 코드(Boilerplate)에서 시작하여 살을 붙여가는 라이브 코딩 진행. 코드 한 줄마다 '액셀 함수'나 '인사팀 직원' 등의 현업 비유법 매핑.
    *   **Close (15분):** 완성된 코드가 기업 내부 인프라와 결합할 때의 확장성(보안, 비용 절감)을 짚어주며 마무리.

---

### 2) 인프라 트러블슈팅 및 보안망 대응 프로토콜
대기업 및 금융권 출강 시 가장 많이 발생하며, 준비되지 않은 강사들을 침묵하게 만드는 보안 인프라 트랙션입니다.

*   **사내 방화벽과 프록시(Proxy) 서버의 개념**
    *   대기업 내부 PC는 인터넷 퍼블릭 망으로 직접 나가지 못하고, 모든 아웃바운드 트래픽을 검사하는 사내 프록시 서버를 거칩니다. 
    *   이 환경에서 단순히 `client = OpenAI()`를 호출하면 프록시 서버가 요청을 차단하여 `APIconnectionError`가 발생합니다.
*   **네트워크 우회 가이드 아키텍처**
    *   OpenAI SDK의 기반이 되는 `httpx` 라이브러리에 기업 프록시 주소(`http://proxy.company.com:8080`)를 명시적으로 주입하여 터널을 뚫어주는 프록시 마운트 기술이 필수적입니다.
    *   간혹 사내 보안 장비가 외부 SSL 인증서를 강제로 교체하는 과정에서 인증서 불일치 에러(`SSLCertVerificationError`)가 발생할 수 있는데, 테스트 환경에 한해 Verification을 비활성화하는 방어 메커니즘을 숙지해야 합니다.

---

## 💡 4. 강사 라이브 코딩 멘트 & 트러블슈팅 팁
> **"여러분, 밤새도록 완벽한 챗봇 코드를 짜왔어도 대기업 강의장 와서 와이파이 연결하고 실행했을 때 'Connection Error' 하나 뜨는 순간 등에서 식은땀이 흐를 겁니다. 그 기업의 사내 방화벽과 프록시 서버가 API 요청을 해킹 시도로 보고 차단했기 때문이죠. 오늘 배울 환경 변수 기반 프록시 우회 설정과 타임아웃 방어벽 코드는 여러분이 어떤 철벽 보안의 대기업에 출강하더라도 15분 만에 라이브 코딩 환경을 복구해 내는 무기가 될 것입니다."**
> 
> * **자주 터지는 에러 및 방어 팁:** 실전에서 네트워크 상태가 불안정하면 API가 무한 대기에 빠져 웹 UI가 얼어버릴 수 있습니다. 반드시 호출 옵션에 `timeout=10.0`을 명시하여 10초 내에 응답이 없으면 안전하게 에러를 뱉고 다음 코드로 넘어가는 예외 분기를 설계해야 합니다.
 

---

## 💻 [코랩 실습 노트북] `04_B2B_Delivery_Troubleshooting.ipynb`

```python
# ⚠️ 보안 주석: verify=False 옵션은 교육용 데모 환경에서만 사용하세요.
# 실무 환경에서는 반드시 SSL 인증서 검증을 유지해야 합니다.
# API Key는 .env 또는 Key Vault에서 관리하세요.
# 에러 처리: APITimeoutError, APIConnectionError 외에도 AuthenticationError, RateLimitError를 추가하세요.
# 로그 기록 시 민감 데이터(문서 내용, 프로젝트명)는 반드시 마스킹 처리하세요.


# ==============================================================================
# 🚀 1개월차 4주차: 강의 시뮬레이션 및 실전 Q&A 방어 전략 실습 템플릿
# ==============================================================================
# [수강생 안내] 본 노트북은 대기업 사내 방화벽 및 네트워크 불안정 상황을 가정한 
# 인프라 우회 및 타임아웃 트러블슈팅 종합 실습 환경입니다.

# 1. 환경 의존성 설정
!pip install -q openai==1.54.0 python-dotenv==1.0.1 httpx==0.27.0 pydantic==2.9.0

import os
import gc
import httpx
import torch
from openai import OpenAI, APIConnectionError, APITimeoutError
from pydantic import BaseModel, Field

# ==============================================================================
# [실습 1 & 2] 엔터프라이즈 사내 보안망 대응 및 프록시/타임아웃 제어 (강의 시간: 50분)
# ==============================================================================

print("=== [인프라 방어] 1. 사내 방화벽 프록시 및 타임아웃 제어 파이프라인 ===")

# [가상 시나리오] 대기업 사내 프록시 서버 주소가 존재한다고 가정 (모킹 환경 구성)
MOCK_CORPORATE_PROXY = "http://proxy.secure-enterprise.co.kr:8080"

# 강사 팁: 실제 출강 현장에서 OS 환경 변수에 프록시를 등록하여 
# 파이썬 전체 패키지가 보안 통로를 타도록 설정하는 표준 가이드라인입니다.
os.environ["HTTP_PROXY"] = MOCK_CORPORATE_PROXY
os.environ["HTTPS_PROXY"] = MOCK_CORPORATE_PROXY

def initialize_secure_client(use_proxy=False):
    """
    네트워크 보안 환경에 따라 프록시 마운트 및 타임아웃이 설정된 클라이언트를 안전하게 생성합니다.
    """
    try:
        if use_proxy:
            print(f"📡 사내 프록시 터널링을 가동합니다: {MOCK_CORPORATE_PROXY}")
            # httpx 클라이언트를 생성하여 SSL 검증 우회 및 프록시 주소 마운트
            # ⚠️ 주의: 실전 배포 시에는 verify=False를 지양해야 하나, 보안망 내 테스트용 강의 시 필수 안내
            custom_http_client = httpx.Client(
                proxies={"all://": MOCK_CORPORATE_PROXY},
                verify=False,
                timeout=5.0 # 네트워크 지연으로 인한 무한 대기 방지 (5초 타임아웃)
            )
            
            client = OpenAI(
                api_key=os.environ.get("OPENAI_API_KEY", "sk-mock-key"),
                http_client=custom_http_client
            )
        else:
            # 일반적인 표준 네트워크 환경용 클라이언트 (3초 타임아웃 제한 강제)
            client = OpenAI(
                api_key=os.environ.get("OPENAI_API_KEY", "sk-mock-key"),
                timeout=httpx.Timeout(3.0, connect=2.0)
            )
            
        print("✅ OpenAI 클라이언트 인프라 셋업이 완료되었습니다.")
        return client
        
    except Exception as e:
        print(f"❌ 클라이언트 생성 실패: {e}")
        return None

# 시스템 테스트 구동
secure_client = initialize_secure_client(use_proxy=False)

# ==============================================================================
# [실습 3] [미션] 1~3주차 통합 15분 마스터 강의 데모 빌드 (강의 시간: 50분)
# ==============================================================================
# [종합 실무 미션]: 
# 1주차의 Structured Outputs, 2주차의 VRAM/RAM 가비지 컬렉터 방어벽, 
# 3주차의 Knowledge Grounding 기법을 한데 모아, 네트워크 에러까지 철저히 방어하는 
# 사내 규정 자동 스크리닝 엔터프라이즈 백엔드 엔진을 완성하십시오.

print("\n=== [종합 미션] 2. 통합 엔터프라이즈 스크리닝 엔진 가동 ===")

# 1주차 패턴: Pydantic 기반 무결성 데이터 규격 선언
class EnterpriseSafetySchema(BaseModel):
    is_compliant: bool = Field(description="규정 준수 여부 (가이드라인을 위반하지 않았으면 True)")
    violation_details: str = Field(description="위반 사항이 있을 경우 원인을 기술, 없으면 'None'")
    risk_level: str = Field(description="위험도 등급 (LOW, MEDIUM, HIGH)")

# 3주차 패턴: 지식 그라운딩을 위한 사내 보안 제약사항 컨텍스트 정의
ENTERPRISE_SECURITY_POLICY = """
[사내 정보 보안 행동 강령]
1. 모든 임직원은 소스 코드 내부나 공용 문서에 비밀번호, API Key, 주민등록번호를 기재할 수 없다.
2. 외부 협력 업체와 공유하는 파일에는 당사 핵심 반도체 회로도 관련 키워드(프로젝트명: ZEUS)가 포함되어서는 안 된다.
"""

def master_delivery_pipeline(input_context):
    """
    1~3주차 모든 기술 스택과 4주차 트러블슈팅이 결합된 강사 마스터 데모 파이프라인
    """
    # 2주차 패턴: 새로운 작업 시작 전 VRAM 및 RAM 가비지 청소로 메모리 누수 방지
    gc.collect()
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        
    # 클라이언트 초기화
    client = initialize_secure_client(use_proxy=False)
    
    # 3주차 패턴: Knowledge Grounding 프롬프트 조립
    system_instruction = f"""
    당신은 기업 내부 기밀 유출을 차단하는 자산 관제 AI입니다.
    제공된 [사내 정보 보안 행동 강령] 가이드라인에 근거하여 입력된 컨텍스트를 정밀 검사하세요.
    
    [사내 정보 보안 행동 강령]
    {ENTERPRISE_SECURITY_POLICY}
    """
    
    try:
        # 1주차 패턴: 최신 Structured Outputs API 연동
        response = client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            response_format=EnterpriseSafetySchema,
            messages=[
                {"role": "system", "content": system_instruction},
                {"role": "user", "content": input_context}
            ],
            temperature=0.0
        )
        return {"status": "success", "data": response.choices[0].message.parsed}
        
    # 4주차 패턴: 출강 현장에서 발생하는 네트워크 하드웨어 에러 집중 방어
    except APITimeoutError:
        return {
            "status": "network_fault", 
            "message": "🚨 [트러블슈팅] API 응답 제한 시간(Timeout)을 초과했습니다. 네트워크 연결을 재점검하세요."
        }
    except APIConnectionError:
        return {
            "status": "network_fault", 
            "message": "🚨 [트러블슈팅] 사내 방화벽 혹은 프록시 장비에 의해 연결이 거부되었습니다. Proxy 마운트 코드를 가동하세요."
        }
    except Exception as general_error:
        return {"status": "system_fault", "message": f"기타 시스템 예외: {str(general_error)}"}

# ------------------------------------------------------------------------------
# 3-2. 실전 시뮬레이션 및 결과 관제 데모 실행
# ------------------------------------------------------------------------------
# 위반 시나리오 샘플 (기밀 프로젝트명 유출 상황)
mock_leak_document = "이번 외부 협력업체 미팅 시 공유할 PPT 초안 파일명은 'PROJECT_ZEUS_SPEC_V1.pdf' 입니다."

print(f"\n📥 관제 대상 사내 문서 인입:")
print(f"[{mock_leak_document}]")

# 마스터 파이프라인 작동
pipeline_report = master_delivery_pipeline(mock_leak_document)

print("\n📊 [라이브 데모 최종 모니터링 리포트]")
if pipeline_report["status"] == "success":
    report_data = pipeline_report["data"]
    print("🎯 검사 결과 완수 및 Pydantic 스키마 파싱 성공")
    print(f"🔒 규정 준수 여부 : {'🟢 안전' if report_data.is_compliant else '🔴 위반 감지'}")
    print(f"🔍 위반 세부 내용 : {report_data.violation_details}")
    print(f"🚨 위험 관제 등급 : {report_data.risk_level}")
    print("\n[DB 적재용 원시 JSON 스크립트 출력]")
    print(report_data.model_dump_json(indent=2))
else:
    print(pipeline_report["message"])

```