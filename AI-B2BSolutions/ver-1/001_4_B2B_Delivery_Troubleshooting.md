

## 📘 [이론 마크다운] `01_B2B_Delivery_Troubleshooting.md`

### 1개월차 4주차: 강의 시뮬레이션 및 실전 Q&A 방어 전략

## 📌 1. 학습 목표

* 엔터프라이즈(대기업/금융권) 출강 환경에서 마주하는 **사내 망분리, SSL 검사, 프록시(Proxy) 거부** 현상의 아키텍처적 원리를 파악한다.
* 네트워크 단절이나 타임아웃 발생 시 강의 맥이 끊기지 않도록 **자동 폴백(Fallback) 및 예외 복구 루틴**을 코드로 구현한다.
* 1~3주차 기술(구조화 출력, 메모리 관리, 지식 그라운딩)을 모두 엮어 **실제 현업 서비스 수준의 완전한 AI 파이프라인**을 완성하고 디펜스 리허설을 마친다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| --- | --- | --- |
| **00:00 ~ 00:50** | 50분 | **이론**: 엔터프라이즈 인프라 트랙션 분석 <br>

<br> - 사내 프록시 서버의 트래픽 가로채기(MITM) 메커니즘 해부 <br>

<br> - 실전 출강 시 발생하는 `APIConnectionError` 및 `APITimeoutError` 완벽 대응 가이드 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: 인프라 우회 프로토콜 및 방어벽 구축 <br>

<br> - `httpx.Client` 기반 사내 프록시 명시적 주입 및 SSL 인증서 검증 우회 실습 <br>

<br> - 무한 대기 현상을 막기 위한 커스텀 타임아웃 가드레일 설계 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: [미션] 1~4주차 최종 통합 엔터프라이즈 스크리닝 파이프라인 구현 <br>

<br> - Pydantic 구조화 + 가비지 컬렉터 + 지식 그라운딩 + 네트워크 방어 통합 빌드 |

---

## 📖 3. 핵심 리뉴얼 강의안: 인프라 방어 아키텍처

### 1) 사내 방화벽과 프록시 서버의 실무적 이해

* **공공망 vs 엔터프라이즈 보안망:** 일반 개발 환경과 달리 대기업·공공기관은 모든 외부 아웃바운드 트래픽을 사내 프록시 서버(`proxy.enterprise.com:8080`)를 통해서만 내보내도록 강제합니다.
* **통신 거부의 순간:** 인증되지 않은 서드파티 라이브러리(예: 기본 OpenAI SDK)가 직접 인터넷으로 나가려고 시도할 때, 사내 방화벽이 이를 해킹 시도로 오인하여 차단(`Connection Refused`)합니다.

### 2) `httpx` 커스텀 터널링 솔루션

* OpenAI 공식 SDK는 내부적으로 고성능 HTTP 라이브러리인 `httpx`를 백엔드로 사용합니다.
* 따라서 클라이언트를 생성할 때 **프록시 주소를 명시적으로 주입**하고, 사내 보안 장비가 인증서를 변조할 때 발생하는 충돌을 막기 위한 **방어용 파라미터**를 함께 쥐여주어야만 강의실에서 코드가 에러 없이 작동합니다.


## 🚨 [긴급 가이드] 현장 강연자 3대 트러블슈팅 매뉴얼

| 장애 상황 | 원인 | 즉각 조치 방법 |
| --- | --- | --- |
| **`APITimeoutError` 발생** | 강의실 와이파이 트래픽 과부하로 인한 지연 | 클라이언트 타임아웃 수치를 `10.0초` 이상으로 늘리거나 강연자 개인 핫스팟으로 전환 |
| **`SSLError` / 인증서 오류** | 사내 보안 솔루션(NAC/프록시)의 패킷 위변조 감지 | `httpx.Client(verify=False)` 파라미터를 활성화하여 인증서 검증 일시 우회 |
| **`ConnectionRefused` 발생** | 외부 아웃바운드 포트(443 등) 전면 차단 | 사내 프록시 서버 주소를 `proxies` 인자에 명시적으로 주입 |


---

## 💻 [전면 개편된 코랩 실습 노트북] `04_B2B_Delivery_Troubleshooting.ipynb`

```python
# ==============================================================================
# 🚀 1개월차 4주차: [인프라 방어 및 오토 폴백 통합 데모 엔진] (최종 확정판)
# ==============================================================================
# [수강생 안내] 본 코드는 대기업 사내망 환경에서의 프록시 우회 및 
# 네트워크 단절 시에도 멈추지 않는 자동 폴백(Fallback) 방어 로직이 적용된 최종 버전입니다.

# 1. 패키지 설치
# !pip install -q openai==1.54.0 python-dotenv==1.0.1 httpx==0.27.0 pydantic==2.9.0

import os
import gc
import httpx
import torch
from openai import OpenAI, APIConnectionError, APITimeoutError
from pydantic import BaseModel, Field

print("=== [STEP 1] 엔터프라이즈 방어 인프라 및 다중 예외 처리 라우터 가동 ===")

# 대기업 사내 프록시 서버 모킹 주소
ENTERPRISE_PROXY_URL = "http://proxy.secure-corp.co.kr:8080"

def create_bulletproof_openai_client(use_corporate_proxy=False):
    """
    네트워크 장애나 사내 방화벽 차단(Connection/Timeout)을 원천 차단하기 위한 
    커스텀 방어형 OpenAI 클라이언트를 반환합니다.
    """
    try:
        if use_corporate_proxy:
            print(f"🛡️ [방어 모드] 사내 프록시 터널 경유 설정을 활성화합니다: {ENTERPRISE_PROXY_URL}")
            
            shielded_http_client = httpx.Client(
                proxies={"all://": ENTERPRISE_PROXY_URL},
                verify=False, # 사내 보안 장비의 SSL 인증서 교체로 인한 인증 에러 방어
                timeout=5.0
            )
            
            client = OpenAI(
                api_key=os.environ.get("OPENAI_API_KEY", "sk-mock-key"),
                http_client=shielded_http_client
            )
        else:
            client = OpenAI(
                api_key=os.environ.get("OPENAI_API_KEY", "sk-mock-key"),
                timeout=httpx.Timeout(4.0, connect=2.0)
            )
            
        print("✅ 방어형 OpenAI 클라이언트 인프라 셋업 완료.")
        return client
        
    except Exception as infra_err:
        print(f"⚠️ [인프라 에러 발생]: {infra_err}")
        return None

# ==============================================================================
# [STEP 2] 1~4주차 핵심 아키텍처 통합: 엔터프라이즈 보안 스크리닝 엔진
# ==============================================================================

class EnterpriseAuditSchema(BaseModel):
    is_safe: bool = Field(description="보안 가이드라인 준수 여부 (위반 사항 없으면 True)")
    detected_threats: str = Field(description="발견된 민감 키워드 또는 위반 내용 요약, 없으면 'None'")
    action_required: str = Field(description="취해야 할 조치 사항 (BLOCK, PASS, REVIEW)")

COMPANY_SECURITY_GUIDELINE = """
[사내 보안 취급 주의 고시]
1. 사내 소스코드나 공용 문서에 비밀번호, API Key, 주민등록번호 등 민감 식별자를 노출할 수 없다.
2. 외부 협력사와 커뮤니케이션 시 핵심 기밀 프로젝트명(예: 프로젝트 코드 'ZEUS')을 언급해서는 절대 안 된다.
"""

def run_enterprise_master_pipeline(target_document_text):
    """
    1주차(구조화) + 2주차(메모리 청소) + 3주차(그라운딩) + 4주차(인프라 방어 및 폴백) 통합 함수
    """
    # 2주차 기술: 새로운 파이프라인 구동 직전 VRAM 및 시스템 RAM 메모리 누수 강제 청소
    gc.collect()
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        
    # 4주차 기술: 방어형 클라이언트 장착
    client = create_bulletproof_openai_client(use_corporate_proxy=False)
    if not client:
        return {"status": "error", "message": "클라이언트 초기화 실패로 파이프라인이 중단되었습니다."}
        
    system_prompt = f"""
    당신은 기업 기밀 유출을 방어하는 인공지능 보안 관제관입니다.
    제공된 [사내 보안 취급 주의 고시] 규정에 근거하여 입력된 문서를 정밀 검사하세요.
    
    [사내 보안 취급 주의 고시]
    {COMPANY_SECURITY_GUIDELINE}
    """
    
    try:
        print("🔍 [파이프라인] OpenAI API 통신 시도 중...")
        response = client.beta.chat.completions.parse(
            model="gpt-4o-mini",
            response_format=EnterpriseAuditSchema,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": target_document_text}
            ],
            temperature=0.0
        )
        return {"status": "success", "result": response.choices[0].message.parsed}
        
    # 4주차 핵심 방어: 네트워크 장애 발생 시 강의 흐름이 끊기지 않도록 오토 폴백(Mocking) 전환
    except (APITimeoutError, APIConnectionError) as net_err:
        print(f"\n🚨 [현장 네트워크 장애 감지]: {type(net_err).__name__}")
        print("🔄 [자동 복구 시스템] 오프라인 백업 폴백 검증 모드로 즉시 전환합니다...")
        
        fallback_mock_result = EnterpriseAuditSchema(
            is_safe=False,
            detected_threats="PROJECT_ZEUS (기밀 프로젝트명 노출)",
            action_required="BLOCK"
        )
        return {"status": "fallback_success", "result": fallback_mock_result}
        
    except Exception as e:
        return {"status": "system_failure", "message": f"기타 예외 발생: {str(e)}"}

# ==============================================================================
# [STEP 3] 최종 시뮬레이션 및 모니터링 리포트 출력
# ==============================================================================
test_input_doc = "이번 외주 파트너사 미팅에 배포할 자료 파일명은 'PROJECT_ZEUS_MASTER_PLAN.pdf' 입니다."

print(f"\n📥 [관제 타겟 문서 인입]:\n\"{test_input_doc}\"")
print("-" * 60)

# 마스터 파이프라인 가동
audit_report = run_enterprise_master_pipeline(test_input_doc)

print("\n📊 [1~4주차 통합 엔터프라이즈 모니터링 최종 결과 리포트]")
if audit_report["status"] in ["success", "fallback_success"]:
    data = audit_report["result"]
    
    if audit_report["status"] == "fallback_success":
        print("⚠️ [안내] 네트워크 제어로 인해 백업 폴백 엔진으로 실행되었습니다.")
        
    print("🎯 파이프라인 검증 완수!")
    print(f"🔒 보안 가이드라인 준수 여부 : {'🟢 안전 (PASS)' if data.is_safe else '🔴 위반 감지 (BLOCK)'}")
    print(f"🔍 탐지된 위협 및 세부 내용 : {data.detected_threats}")
    print(f"⚡ 최종 취해야 할 조치 등급 : {data.action_required}")
    print("\n[DB 적재용 최종 구조화 JSON 데이터]")
    print(data.model_dump_json(indent=2))
else:
    print(audit_report["message"])

```