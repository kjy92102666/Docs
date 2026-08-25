# 🤖 MES 챗봇 시스템 분석 및 발표 가이드

## 📌 개요 및 현재 브랜치 상태
* **확인된 브랜치**: `feature/chatbot-openai` (`origin/feature/chatbot-openai`와 동일)
* **작업 트리**: 변경사항 없음 (Clean)
* **주요 특징**: Ollama(로컬 LLM)와 OpenAI(외부 LLM)를 환경변수로 선택하여 사용할 수 있는 챗봇 기능 구현 완료.

---

## 1. 챗봇 작동 원리

### 🔄 전체 흐름
1. **사용자**: React ChatbotWidget에서 질문 입력
2. **프론트엔드**: `POST /api/chatbot/messages` 호출
3. **FastAPI (router.py)**: `CHATBOT_PROVIDER` 환경변수 확인
   * `ollama` → `OllamaProvider`
   * `openai` → `OpenAIProvider`
4. **질문 유형 분석 (`capabilities.py`)**:
   * Dashboard 데이터 조회
   * Inspection / Batch 데이터 조회
   * 정적 프로젝트 지식 (`knowledge.py`) 사용
   * 지원하지 않는 질문 차단
5. **Context 조합 (`service.py`)**: System Prompt + Context + 사용자 질문
6. **AI Provider**: LLM 응답 생성
7. **화면 표시**: React 챗봇 UI에 결과 출력

### 📁 주요 파일 구조
* **백엔드 (`backend/app/features/chatbot/`)**
  * `router.py`: API 엔드포인트
  * `service.py`: 질문 처리 및 Context 조합
  * `dependencies.py`: Ollama/OpenAI Provider 선택
  * `capabilities.py`: 질문 유형 및 지원 범위 판단
  * `knowledge.py`: 정적 프로젝트 지식
  * `prompts.py`: AI 시스템 프롬프트
  * `schemas.py`: 요청/응답 스키마
  * `providers/`:
    * `base.py`: 공통 Provider 인터페이스 (Protocol)
    * `ollama_provider.py`: 로컬 Ollama 호출
    * `openai_provider.py`: OpenAI API 호출
* **프론트엔드 (`frontend/src/`)**
  * `features/chatbot/ChatbotWidget.jsx`
  * `features/chatbot/ChatbotWidget.css`
  * `features/chatbot/model/chatbotState.js`
  * `domains/chatbot/chatbotApi.js`

---

## 2. API 요청 및 응답

* **엔드포인트**: `POST /api/chatbot/messages`

### 📤 요청 예시
```json
{
  "message": "오늘 검사 현황 알려줘",
  "context_hints": []
}

`context_hints`는 `docs`, `dashboard`, `inspection` 중 데이터 영역을 직접 지정하는 기능입니다. 현재 프론트엔드는 기본적으로 빈 배열을 보내며, 백엔드가 질문 키워드로 자동 판단합니다.
```


📥 응답 예시
```
{
  "reply": "오늘 검사 PCB는 120개이며 불량률은 4.2%입니다.",
  "contexts_used": ["dashboard"],
  "provider": "openai"
}
```


## 3. 현재 지원하는 질문 범위

### 📊 Dashboard 관련 (실시간 데이터 조회)

- **질문 예시**: "오늘 검사 현황 알려줘", "현재 불량률은?", "오늘 검사 수량은?", "주요 불량 유형은?", "현재 검사 중인 Batch가 몇 개야?"
    
- **활용 데이터**: 오늘 검사 PCB 수, 불량률, AI 판정 수용률, AI 검수 완료 객체 수, 검사 대기/검사 중/검수 대기 Batch, 주요 불량 유형
    

### 📦 PCB 검사 및 Batch 관련 (최대 5개 조회)

- **질문 예시**: "최근 Batch 상태 알려줘", "검사 중인 PCB가 있어?", "Batch 검사 준비 상태 알려줘", "최근 검사 결과를 설명해줘"
    

### 📘 정적 시스템 설명 (`knowledge.py`)

- **질문 예시**: "이 시스템은 뭐야?", "PCB 검사 기능을 설명해줘", "YOLO는 어떤 역할을 해?", "Dashboard에서는 무엇을 볼 수 있어?", "분석 화면은 어떤 기능이야?", "챗봇은 어떤 정보를 제공해?"
    

### 🧮 불량 PCB 수량 질문 (백엔드 직접 계산)

- **질문 예시**: "오늘 불량 PCB 몇 개야?", "불량품은 몇 개 발생했어?", "오늘 NG 수량이 얼마야?"
    
- **계산 공식**: $\text{불량 PCB 수} = \frac{\text{검사 PCB 수} \times \text{불량률}}{100}$
    
- **특징**: AI가 숫자를 임의로 생성(환각)하는 것을 방지하기 위해 백엔드에서 직접 계산 후 반환.
    

### 🚫 지원하지 않는 질문 (명시적 차단)

- **대상 데이터**: AGV 현재 위치/운행 상태, 재고 수량, 원자재 현황, 설비 온도, 작업자 정보
    
- **안내 메시지**: _"현재 챗봇에 연결된 시스템 데이터에서는 해당 정보를 확인할 수 없습니다. 현재는 Dashboard 및 PCB 검사/Batch 현황을 조회할 수 있습니다."_
    

## 4. OpenAI와 Ollama 전환 방식

환경변수 설정으로 Provider를 전환합니다.

### 🟢 OpenAI 설정
```
CHATBOT_PROVIDER=openai
OPENAI_API_KEY=your_actual_api_key
OPENAI_MODEL=gpt-4o-mini
```
⚠️ **주의사항**: 현재 `feature/chatbot-openai` 브랜치의 `docker-compose.yml`에는 `CHATBOT_PROVIDER`, `OPENAI_API_KEY`, `OPENAI_MODEL` 환경변수가 누락되어 있습니다. 서버 Docker 배포 전 해당 설정을 선반영해야 합니다.

## 5. 현재 구현의 강점 (발표 포인트)

1. **Provider 추상화 (의존성 역전)**
    
    - `ChatProvider` Protocol 인터페이스를 공유하므로, 서비스 로직 수정 없이 환경변수만으로 로컬 LLM과 상용 API 전환 가능.
        
2. **실운영 데이터 vs 정적 지식 분리**
    
    - 실시간 데이터(Dashboard/Batch)와 정적 지식(문서)을 구분하고, 불가능한 질문은 명시적으로 차단하여 AI의 오답/환각 방지.
        
3. **수치 계산의 정확성 보장**
    
    - 불량 수량 등 정확성이 핵심인 수치는 LLM 추론에 맡기지 않고 백엔드에서 결정론적으로 계산.
        
4. **구조화된 오류 응답**
    
    - 단순 500 에러가 아닌, 사용자 친화적인 안내 메시지(연결 실패, 타임아웃, API 키 오류 등)로 변환하여 반환.


## 6. 개선 권장사항 (우선순위 순)

|**순위**|**항목**|**설명 및 개선 방향**|
|---|---|---|
|**우선 1**|**OpenAI 오류 처리 Router 연결**|`router.py`에 OpenAI 전용 예외(`OpenAITimeoutError`, `OpenAIConfigurationError` 등) 매핑 추가 또는 공통 `ProviderError`로 통합|
|**우선 2**|**provider 하드코딩 제거**|직접 계산 응답이나 예외 응답 시 `provider` 값이 "ollama"로 고정되는 현상 수정 (`provider="deterministic"` 또는 `provider.name` 활용)|
|**우선 3**|**OpenAI 전용 단위 테스트 추가**|API 키 미설정, 401 Unauthorized, Timeout, 정상 추출 흐름 테스트 추가|
|**우선 4**|**대화 이력 (History) 지원**|프론트엔드에서 최근 5~10개 메시지 Context를 백엔드로 전달하도록 확장|
|**우선 5**|**질문 분류 (Intent) 보완**|단순 키워드 매칭의 한계 보완을 위해 정규식 패턴 추가 또는 LLM 기반 Intent Classifier 도입|
|**우선 6**|**비용 및 보안 제어**|요청 Rate Limit, 최대 토큰 제한, API 키 로깅 방지, 응답 시간 캘리브레이션|

## 7.추천 시연 순서

```
[1. UI 진입] ──> [2. 정적 지식] ──> [3. 실시간 현황] ──> [4. 정확한 계산] ──> [5. Batch 조회] ──> [6. 범위 제한 (환각 방지)]
 MES 메인 화면      "이 시스템은 뭐야?"   "오늘 검사 현황"   "오늘 불량 PCB 몇 개?"  "최근 Batch 상태"   "AGV 현재 위치 알려줘"
```

1. **시연 1 (UI 진입)**: MES 우측 하단 챗봇 버튼 클릭 및 추천 질문 확인
    
2. **시연 2 (정적 지식)**: `"이 시스템은 어떤 시스템인가요?"` (Tech Stack 및 YOLO 검출 기능 답변)
    
3. **시연 3 (실시간 데이터)**: `"오늘 검사 현황 알려줘"` (Dashboard 수치와 연동된 답변)
    
4. **시연 4 (정확한 수치 calculation)**: `"오늘 불량 PCB는 몇 개야?"` ⭐ **[주요 포인트]** (백엔드 직접 계산)
    
5. **시연 5 (Batch 현황)**: `"최근 Batch 검사 상태 알려줘"` (Batch 코드 및 NG 수 연동)
    
6. **시연 6 (범위 제한)**: `"현재 AGV 위치 알려줘"` ⭐ **[주요 포인트]** (환각 방지 및 지원 범위 제한 메시지)


## ## 8. 발표 스토리라인

```
1장. 문제 정의 (기존 MES의 파편화된 화면 이동 및 높은 데이터 접근 허들)
 ↓
2장. 해결 방법 (자연어 기반 MES 운용 지원 챗봇 도입)
 ↓
3장. 시스템 구조 (React - FastAPI - Capability Selector - Providers)
 ↓
4장. 핵심 기능 (실시간 데이터 질의, 계산 검증, 정적 지식 제공, Provider 전환)
 ↓
5장. 신뢰성 확보 (신뢰 기반 백엔드 연산 및 명시적 범위 제한으로 Hallucination 제거)
 ↓
6장. 서비스 시연 (추천 시연 시나리오 수행)
 ↓
7장. 확장 계획 (AGV/재고 데이터 연동, 대화 이력 지원, LLM Intent 분류 고도화)
```

## 9. 발표 전 최종 점검 리스트

- [ ] **환경변수 설정**: `CHATBOT_PROVIDER=openai`, `OPENAI_API_KEY`, `OPENAI_MODEL` 등록 확인
    
- [ ] **Docker Compose**: `docker-compose.yml` 내 챗봇 관련 환경변수 매핑 반영
    
- [ ] **시연용 데이터 세팅**:
    
    - 검사 완료 PCB 및 불량률 > 0 데이터
        
    - 최근 Batch (NG 건수 포함) 2~3개 확보
        
- [ ] **장애 시나리오 검증**:
    
    - OpenAI API Key 정상 작동 여부 및 인터넷 통신 확인
        
    - 브라우저 개발자 도구 Console 에러 유무 확인
        

> 💡 **발표용 핵심 한 문장**
> 
> _"우리 MES 챗봇은 단순한 대화형 AI가 아니라, **백엔드 정밀 연산과 실시간 MES 데이터를 결합하여 환각을 최소화한 실무 중심의 운영 지원 엔진**입니다."_




ubuntu@ip-172-26-4-140:~$ tail -3 ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHKZgF7zZvNiMeCsrUZ9NK/b4irqvN3PeusaqDq3s1UO owner@com180-132
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/bWeplItK8C0ggYcQlMf1tblLy7eFs7ITi2J83Q2m2 kjy92@KJY