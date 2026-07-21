## 📘 [이론 마크다운] `01_HF_Infrastructure_Core.md`

 
### 1개월차 2주차: 오픈소스 LLM 인프라 대응 및 환경 오류 제로 가이드

## 📌 1. 학습 목표
- 오픈소스 모델의 필요성을 이해하고, 글로벌 표준 AI 저장소인 Hugging Face Hub 생태계를 자유롭게 다룰 수 있다.
- 토크나이저(Tokenizer)와 모델(Model) 객체의 파이프라인 연동 매커니즘을 파이썬 코드로 제어할 수 있다.
- 구글 코랩(Colab) 환경의 하드웨어 한계를 극복하기 위해 `gc` 및 `torch.cuda` 메모리 관리 아키텍처를 도입하고, OOM(Out Of Memory) 발생 시 시스템 다운 없이 우회하는 고성능 B2B 로컬 스크리닝 엔진을 구현한다.

## ⏱️ 2. 시간 배정 (총 3시간 / 180분 기준)

| 시간대 | 소요 시간 | 활동 내용 |
| :--- | :---: | :--- |
| **00:00 ~ 00:50** | 50분 | **이론**: 오픈소스 생태계 아키텍처와 GPU 컴퓨팅 자원 이해 <br> - 사내 보안망/망분리 환경의 대안으로서의 로컬 오픈소스 모델 당위성 <br> - 토크나이저 Vocab 매핑 및 모델(Weight) 레이어 로드 메커니즘 <br> - GPU VRAM 적재 공식 및 `pipeline` 내부 최적화 아키텍처 해석 |
| **00:50 ~ 01:00** | 10분 | **쉬는 시간** |
| **01:00 ~ 01:50** | 50분 | **실습 1 & 2**: Hugging Face Pipeline 제어 및 메모리 누수 방어 <br> - `transformers` 라이브러리를 활용한 감정 분석 및 텍스트 생성 모델 로드 실습 <br> - 반복 호출 시 파이썬 가비지 컬렉터(`gc`)와 CUDA 캐시 비우기 검증 |
| **01:50 ~ 02:00** | 10분 | **쉬는 시간** |
| **02:00 ~ 03:00** | 50분 | **실습 3**: 사내 보안 우회용 로컬 가속 스크리닝 엔진 미션 <br> - 수강생 개별 실습 진행 (고객 메일 기밀 필터링 및 VRAM 방어벽 구축) <br> - CUDA OutOfMemoryError 예외 처리 파이프라인 마스터 코드 리뷰 |

---

## 📖 3. 핵심 이론 집중 강의안 (50분 분량)

### 1) Hugging Face 생태계와 클라우드 인프라 아키텍처

* **슬라이드 1제목:** B2B 환경에서의 오픈소스 도입 당위성
* **Data Sovereignty (데이터 주권):** 고객 개인정보와 금융 거래 내역이 외부 퍼블릭 API로 나가는 순간 규제 위반. 사내 망 내에 격리된 프라이빗 클라우드 필수.
* **TCO (총소유비용) 최적화:** 대규모 단순 분류·스크리닝 업무에는 토큰 과금형 API보다 미세조정된 경량 오픈소스 모델이 경제적.


* **슬라이드 2제목:** 토크나이저와 모델 레이어의 파이프라인 매커니즘
* 텍스트 $\rightarrow$ 서브워드 분할 $\rightarrow$ Token ID 인코딩 $\rightarrow$ 트랜스포머 레이어 거쳐 Logits(확률 분포) 출력 과정의 직관적 이해.



> **🎤 [강사 멘트]**
> *"여러분, 지난주에 배운 OpenAI API는 정말 편리하지만, 삼성전자, 현대차, 혹은 시중 은행 같은 곳에 가면 '고객 비밀번호나 미공개 재무 제표를 외부 서버에 전송하면 안 된다'는 철통같은 망분리 방어막에 부딪힙니다. 이때 구원투수로 등장하는 것이 바로 오픈소스 모델입니다. 사내 서버에 직접 올리되, GPU 자원을 어떻게 쥐고 흔들어야 서버가 안 죽는지 오늘 마스터하게 될 겁니다."*


엔터프라이즈 환경에서 외부 API(OpenAI 등)를 쓸 수 없는 극단적인 망분리 또는 기밀 데이터 취급 부서의 요구를 해결하기 위한 B2B 필수 기술 영역입니다.

*   **왜 오픈소스 모델인가? (B2B 관점)**
    *   **데이터 주권(Data Sovereignty):** 고객 개인정보, 금융 거래 기록, 제조 공정 기밀 등은 약관상 외부 클라우드로 전송되는 순간 규제 위반이 될 수 있음. 사내 인프라(On-Premise) 또는 격리된 프라이빗 클라우드에 직접 올릴 수 있는 모델이 필요함.
    *   **비용 및 인프라 통제:** 초당 수만 건의 단순 분류/스크리닝 연산이 일어날 경우, 토큰당 과금되는 상용 API보다 특정 연산에 맞게 미세조정(Fine-Tuning)된 경량 오픈소스 모델을 자체 서버에서 띄우는 것이 TCO(총소유비용) 측면에서 압도적으로 유리.

*   **토크나이저(Tokenizer)와 모델(Model)의 데이터 엔지니어링 파이프라인**
    *   **Tokenizer:** 자연어 문자열을 서브워드(Subword) 단위로 쪼개고 이를 고유한 정수 인덱스(Token ID) 배열로 인코딩. 이 과정에서 Attention Mask가 생성되어 패딩 토큰 위치를 무시하도록 설계됨.
    *   **Model:** 고차원 임베딩 공간으로 토큰 ID를 변환한 후, 트랜스포머 레이어를 거쳐 다음 단어의 확률 분포(Logits)를 출력.

---

### 2) 구글 코랩(Colab) 환경 최적화 및 VRAM 가비지 수집 메커니즘


* **슬라이드 1제목:** 딥러닝 메모리의 2단계 해제 메커니즘
* `del model` (파이썬 힙 영역 참조 끊기) $\neq$ VRAM 반환
* `gc.collect()`: 파이썬 순환 참조 찌꺼기 수거
* `torch.cuda.empty_cache()`: PyTorch 메모리 할당기가 쥐고 있던 물리적 GPU 캐시를 OS로 전송


* **슬라이드 2제목:** OOM(Out of Memory) 크래시의 공포
* 일반 `Exception`으로는 GPU 드라이버 레벨의 메모리 부족 에러를 잡지 못해 백엔드 데몬 전체가 셧다운되는 치명적 한계 극복.



> **🎤 [강사 멘트]**
> *"파이썬에서 변수를 지울 때 그냥 `del`만 치면 메모리가 깨끗해질 것 같죠? 절대 아닙니다. PyTorch는 연산 속도를 높이기 위해 GPU 메모리를 자기 주머니에 꽉 쥐고 놓지 않습니다. 그래서 새로운 모델을 로드하는 순간 빨간색 경고와 함께 코랩 세션이 펑 하고 터지는 겁니다. 오늘 배운 **`gc.collect()`와 `torch.cuda.empty_cache()`의 콤보**를 쓰지 않으면, 사내 프로덕션 서버에서 밤마다 개발팀 전화벨이 울리게 될 겁니다."*


무료 혹은 제한된 클라우드 GPU 자원(T4)을 사용하는 교육 현장과 실제 가상 인프라 배포 단계에서 가장 빈번하게 발생하는 크래시 원인을 다룹니다.


```

[ Active VRAM Context ] ──(변수 해제: del model)──> [ Zombie Tensors in Memory ]
│
(파이썬 힙 영역 해제: gc.collect())
│
▼
[ Free Hardware Memory ] <──(GPU 메모리 물리 반환: torch.cuda.empty_cache())

```

*   **파이썬 가비지 컬렉션(`gc.collect()`)의 역할**
    *   파이썬 변수를 `del` 키워드로 지우더라도 순환 참조 등의 이유로 힙(Heap) 메모리에 찌꺼기가 남아있을 수 있음. 강제로 가비지 컬렉터를 호출하여 참조 카운트가 0이 된 객체를 수집해야 함.
*   **CUDA 캐시 비우기 (`torch.cuda.empty_cache()`)의 물리적 의미**
    *   파이썬 단에서 텐서 변수가 삭제되어도, PyTorch의 메모리 할당기(Memory Allocator)는 속도 향상을 위해 GPU 하드웨어 메모리를 즉시 OS에 반환하지 않고 캐시 공간으로 들고 있음. 
    *   `empty_cache()`를 실행해야 비로소 좀비 텐서들이 차지하던 VRAM 캐시 영역이 초기화되어 다음 대형 모델이 들어올 자리가 확보됨. (강사 필수 강조 항목)

---

### 3) 로컬 스크리닝 엔진의 무결한 예외 처리 설계



* **슬라이드 1제목:** `torch.cuda.OutOfMemoryError` 타겟팅 방어벽
* 일반 에러 처리 구문으로 잡히지 않는 하드웨어 레벨의 비정상 크래시를 가로채어 시스템 중단을 원천 차단.


* **슬라이드 2제목:** Warm Bootstrapping (자동 자원 복구 전략)
* OOM 감지 시 폭발한 엔진 컨텍스트를 즉시 폐기($null$)하고, 메모리를 비운 뒤 안전하게 파이프라인을 재조립하는 B2B 고가용성(High Availability) 아키텍처 구현.



> **🎤 [강사 멘트]**
> *"현업에서 여러분의 코드가 욕을 먹는 순간은 AI가 답변을 틀렸을 때가 아니라, **서버가 다운되어 서비스가 마비되었을 때**입니다. 악의적인 사용자가 기가 바이트 단위의 텍스트를 던져서 의도적으로 GPU를 터뜨려도, 우리의 스크리닝 엔진은 **`torch.cuda.OutOfMemoryError`**를 캐치해서 스스로 메모리를 청소하고 백업 모드로 살아나야 합니다. 이것이 주니어와 시니어 아키텍트를 나누는 결정적 차이입니다."*



기업용 내부 차단 시스템(Filtering System)에 오픈소스 모델을 붙일 때 가장 위협적인 요소는 **하드웨어 에러로 인한 전체 파이프라인 다운**입니다.

*   **왜 일반 Exception 문으로는 예외 처리가 안 되는가?**
    *   오픈소스 및 PyTorch 환경에서는 CPU 하드웨어 단의 에러나 GPU 드라이버 레벨의 메모리 에러(`torch.cuda.OutOfMemoryError`)가 발생할 수 있습니다. 
    *   이를 단순 `ValueError`나 포괄적인 일반 예외로 묶어두면, 딥러닝 코어가 터졌을 때 예외 처리가 가동되지 못하고 전체 백엔드 데몬(Daemon)이 셧다운됩니다.
*   **하드웨어 자원 임계치 예외 처리의 고도화**
    *   사용자의 입력 텍스트가 너무 길어 트랜스포머의 컨텍스트 윈도우 한계를 넘거나 VRAM을 초과하는 순간을 캐치하여, 시스템을 재부팅하는 대신 메모리를 강제 청소하고 안전한 에러 응답 코드를 리턴하는 아키텍처를 구현해야 현업 솔루션으로 인정받습니다.

---

## 💡 4. 강사 라이브 코딩 멘트 & 트러블슈팅 팁
> **"여러분, OpenAI 같은 API는 서버가 펑펑 터져도 우리 하드웨어는 안전합니다. 하지만 오픈소스 모델을 사내 서버나 코랩에 직접 올리는 순간부터는 GPU 자원 관리가 온전히 '나의 책임'이 됩니다. 코드 몇 줄 잘못 쓰면 GPU 메모리가 꽉 차는 OOM(Out of Memory) 현상으로 서버가 바로 뻗어버리죠. 오늘 배울 `gc.collect()`와 하드웨어 전용 예외 처리 기법이 오픈소스 인프라를 다루는 핵심 엔지니어링 역량입니다."**
> 
> * **자주 터지는 에러 및 방어 팁:** 실습 중 다른 모델을 여러 번 로드하면 코랩 오른쪽 상단의 'RAM/VRAM' 인디케이터가 빨간색으로 변하며 세션이 죽습니다. 수강생들에게 새로운 모델을 테스트하기 전 반드시 마스터 코드에 선언된 메모리 릴리즈 함수를 실행하도록 통제하세요.
 

---

## 💻 [코랩 실습 노트북] `02_HF_Infrastructure_Core.ipynb`

```python
# ⚠️ 보안 주석: Colab 환경은 공용 클라우드이므로 기업 기밀 데이터 업로드 금지.
# 반드시 샘플 데이터나 익명화된 데이터만 사용하세요.
# 실무 환경에서는 로컬 서버 또는 프라이빗 클라우드에서 모델을 운영해야 합니다.
# 로그 기록 시 입력 텍스트는 비식별화 처리하여 개인정보가 남지 않도록 하세요.

# ==============================================================================
# 🚀 1개월차 2주차: 오픈소스 LLM 인프라 대응 및 환경 오류 제로 실습 템플릿
# ==============================================================================
# [수강생 안내] 본 노트북은 구글 코랩(Colab) 환경의 T4 GPU 런타임에 최적화되어 있습니다.
# 실행 전 상단 메뉴 [런타임] -> [런타임 유형 변경]에서 'T4 GPU'가 선택되어 있는지 반드시 확인하세요.

# ==============================================================================
# [실습 1] 오프라인 가속을 위한 Hugging Face 인프라 셋업 (강의 시간: 15분)
# ==============================================================================
# 1. 의존성 패키지 및 가속 라이브러리 설치
# 오픈소스 최적화 파이프라인 구동을 위해 관련 생태계 라이브러리 버전을 고정하여 설치합니다.
!pip install -q transformers==4.44.0 torch==2.4.0 accelerate==0.33.0

import os
import gc
import sys
import torch
import logging
# Hugging Face 핵심 추상화 클래스 및 데이터 타입 임포트
from transformers import pipeline, AutoTokenizer, AutoModelForSequenceClassification

print(f"📦 PyTorch 버전: {torch.__version__} | CUDA 사용 가능 여부: {torch.cuda.is_available()}")

# ==============================================================================
# [실습 2] 구글 코랩 환경 최적화 및 VRAM 방어 아키텍처 (강의 시간: 35분)
# ==============================================================================

# ------------------------------------------------------------------------------
# 2-1. pipeline 기반 초고속 감정 분석 모델 로드 및 추론
# ------------------------------------------------------------------------------
print("\n=== 2-1. 경량 실시간 감정 분석(Sentiment) 파이프라인 가동 ===")

# 한국어 및 다국어 처리가 가능한 경량 스크리닝용 모델 타겟팅
model_identifier = "distilbert-base-uncased-finetuned-sst-2-english"

# 하드웨어 디바이스 스캔 (GPU가 있으면 0번 CUDA 디바이스 사용, 없으면 CPU 강제)
target_device = 0 if torch.cuda.is_available() else -1

try:
    # 추상화 툴킷 pipeline을 사용하여 토크나이저와 모델을 메모리에 원스톱 로드
    screening_pipeline = pipeline(
        task="sentiment-analysis",
        model=model_identifier,
        device=target_device
    )
    
    # 샘플 B2B 고객 컨텍스트 입력 테스트
    sample_email = "We are dynamic partners but your delivery delay significantly ruined our project schedule."
    inference_result = screening_pipeline(sample_email)
    print(f"🎯 [추론 완수] 결과 데이터: {inference_result}")

except Exception as init_err:
    print(f"❌ 모델 로드 중 예외 발생: {init_err}")

# ------------------------------------------------------------------------------
# 2-2. GPU 힙 공간 청소 및 메모리 파괴 시뮬레이션 (강사 데모)
# ------------------------------------------------------------------------------
print("\n=== 2-2. VRAM 방어벽 작동 및 가비지 자원 클리닝 ===")

def release_gpu_resources(pipeline_object):
    """
    메모리에 상주하는 모델 객체를 완전히 격리하고 하드웨어 VRAM 캐시를 강제 초기화합니다.
    """
    global screening_pipeline
    
    # 1. 파이썬 네임스페이스에서 객체 참조 삭제
    if pipeline_object in globals() or 'screening_pipeline' in globals():
        del pipeline_object
        if 'screening_pipeline' in globals():
            del screening_pipeline
            
    # 2. 파이썬 가비지 컬렉터를 가동하여 참조 카운트가 깨진 좀비 객체 수집
    collected = gc.collect()
    
    # 3. PyTorch 내부의 CUDA 캐시 할당기를 물리적으로 초기화하여 VRAM을 OS에 반환
    if torch.cuda.is_available():
        torch.cuda.empty_cache()
        torch.cuda.reset_max_memory_allocated()
        
    print(f"🧹 [메모리 자산 최적화] 수집된 가비지 오브젝트: {collected}개 | VRAM 캐시 초기화 완수.")

# VRAM 릴리즈 함수 실행
release_gpu_resources(screening_pipeline)

# ==============================================================================
# [실습 3] [미션] 사내 보안 우회용 로컬 스크리닝 엔진 구축 (강의 시간: 50분)
# ==============================================================================
# [현업 시나리오]: 외부 클라우드 API로 기업 기밀 정보가 유출되는 것을 차단하기 위해,
# 사내 망분리 서버 내에서 독립적으로 작동하는 로컬 고객 메일 분류/위험도 스크리닝 엔진을 구축해야 합니다.
# 대량의 가변 컨텍스트 인입 시 하드웨어(GPU VRAM) 크래시를 방어하는 예외 처리 코드를 내장하세요.

print("\n=== 3. [현업 실무] 로컬 스크리닝 엔진 가동 및 하드웨어 예외 처리 ===")

class LocalScreeningEngine:
    def __init__(self, model_name):
        self.model_name = model_name
        self.engine = None
        self.bootstrap_engine()
        
    def bootstrap_engine(self):
        device_idx = 0 if torch.cuda.is_available() else -1
        # 내부 구조 스키마 보장을 위해 고속 분류 파이프라인 자동 조립
        self.engine = pipeline(
            task="text-classification",
            model=self.model_name,
            device=device_idx
        )
        print(f"⚙️ [엔진 초기화] 로컬 오픈소스 모델 [{self.model_name}]이 하드웨어에 적재되었습니다.")

    def analyze_security_risk(self, target_text):
        try:
            # 입력값 데이터 무결성 체크
            if not target_text or len(target_text.strip()) == 0:
                return {"status": "skipped", "reason": "빈 데이터 인입"}
                
            # 💡 [보안 안전장치] 모델의 최대 컨텍스트 윈도우 초과를 방지하기 위해 
            # truncation=True와 max_length=512 옵션을 적용하여 토큰 초과 에러를 미연에 방어합니다.
            output = self.engine(target_text, truncation=True, max_length=512)
            return {"status": "success", "analytics": output[0]}
            
        # 💡 [변경 포인트] 하드웨어 단의 물리적 메모리 초과 에러(OOM) 정밀 타겟팅
        # 일반 Exception문으로는 잡히지 않는 PyTorch core의 VRAM 초과 에러를 방어합니다.
        except torch.cuda.OutOfMemoryError as oom_error:
            print("🚨 [하드웨어 비상 제동] GPU VRAM 용량 한도 초과 감지! 긴급 자원 청소를 가동합니다.")
            
            # 엔진 컨텍스트를 즉시 폐기하여 크래시난 연산 스택 제거
            self.engine = None
            gc.collect()
            torch.cuda.empty_cache()
            
            # 자원 재복구(Warm Bootstrapping) 프로세스 가동
            self.bootstrap_engine()
            
            return {
                "status": "hardware_fault_bypass", 
                "reason": "VRAM 메모리 한도 초과로 인한 자동 바이패스 처리",
                "analytics": {"label": "CRITICAL_OVERLOAD", "score": 0.0}
            }
            
        except Exception as general_err:
            return {"status": "system_error", "reason": f"일반 런타임 예외: {str(general_err)}"}

# ------------------------------------------------------------------------------
# 3-2. 로컬 스크리닝 파이프라인 연동 및 스트레스 테스트
# ------------------------------------------------------------------------------
# 엔진 인스턴스 생성
screening_center = LocalScreeningEngine(model_name="distilbert-base-uncased-finetuned-sst-2-english")

# 정상 시나리오 테스트
normal_email = "The system backend integration works fine. Excellent job team."
print(f"\n📥 일반 인입 데이터 분석 중...")
result_normal = screening_center.analyze_security_risk(normal_email)
print(f"📦 분석 결과 리턴: {result_normal}")

# 비정상 하드웨어 과부하 시나리오 시뮬레이션 (의도적 대량 컨텍스트 주입 공격)
# 강사 팁: 엄청나게 긴 컨텍스트를 복사하여 GPU 메모리 에러를 유도하는 샌드박스 영역입니다.
overloaded_email = "Deep learning payload crash test. " * 3000 

print(f"\n📥 과부하 유도 데이터(OOM 공격) 인입 데이터 분석 중...")
result_overload = screening_center.analyze_security_risk(overloaded_email)

print("\n🎯 [최종 방어 관제 시스템 가동 현황]")
if result_overload["status"] == "success":
    print(f"✅ 일반 분석 완료: {result_overload['analytics']}")
elif result_overload["status"] == "hardware_fault_bypass":
    print(f"⚠️ 시스템 다운 방어 성공: {result_overload['reason']}")
    print(f"📊 기본 대체 플래그 적재 현황: {result_overload['analytics']}")
else:
    print(f"❌ 관제 엔진 최종 실패: {result_overload['reason']}")
```