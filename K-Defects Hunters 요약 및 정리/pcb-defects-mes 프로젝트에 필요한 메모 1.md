
# 개발 환경 실행 가이드

> 설치 명령은 **최초 1회만**, 실행 명령은 **개발할 때마다** 사용합니다.

# 백엔드 (FastAPI)

## 최초 1회만

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### 명령어 설명

| 명령어 | 설명 |
| ------ | ---- |
| `cd backend` | 백엔드 프로젝트 폴더로 이동 |
| `python -m venv .venv` | 프로젝트 전용 Python 가상환경 생성 |
| `.venv\Scripts\activate` | 가상환경 활성화 |
| `pip install -r requirements.txt` | 필요한 Python 패키지 설치 |

> **주의**
>
> `python -m venv .venv`는 최초 한 번만 실행합니다.  
> `.venv` 폴더가 이미 있다면 다시 생성할 필요가 없습니다.

---

## 이후 실행할 때마다

새 PowerShell(또는 터미널)을 열었다면 다음 명령을 실행합니다.

```bash
cd backend
.venv\Scripts\activate
uvicorn app.main:app --reload
```

### 권장 실행 방법

가상환경을 따로 활성화하지 않고 한 줄로 실행할 수도 있습니다.

```bash
cd backend
.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

> 이 방법은 가상환경을 따로 활성화할 필요가 없어 가장 추천하는 실행 방식입니다.

---

## requirements.txt가 변경된 경우

의존성이 변경되었을 때만 다시 설치합니다.

```bash
pip install -r requirements.txt
```

---

# 프론트엔드 (React)

## 최초 1회만

```bash
cd frontend
npm install
```

### 명령어 설명

| 명령어 | 설명 |
| ------ | ---- |
| `cd frontend` | 프론트엔드 프로젝트 폴더로 이동 |
| `npm install` | `package.json`을 읽어 필요한 패키지를 `node_modules`에 설치 |


## 백엔드 터미널
```bash
cd backend
.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

## 프론트엔드 터미널
```bash
cd frontend
npm run dev
```

---



# **api**

| 배치 기능                     | 생산계획 | PCB 검사 |
| ------------------------- | ---- | ------ |
| 전체 배치 조회 `listBatches`    | 사용   | 사용     |
| 배치 페이지 조회 `listBatchPage` | 사용   | 미사용    |
| 배치 생성 `createBatch`       | 사용   | 미사용    |
| 배치 취소 `cancelBatch`       | 사용   | 미사용    |
| 배치 PCB 조회 `listBatchPcbs` | 사용   | 미사용    |
| 배치 검사 결과 조회               | 미사용  | 사용     |
| 배치 검사 실행 `runInspection`  | 미사용  | 사용     |

---

권장 배포 구조
# AWS Lightsail 권장 배포 구조

> **목표**
>
> 현재 PCB Defects MES 프로젝트 규모에서 가장 단순하면서도 운영하기 쉬운 배포 구조

---

# 전체 아키텍처

```text
사용자 브라우저
        │
        │ HTTPS / WebSocket
        ▼
┌────────────────────────────────────────────┐
│          AWS Lightsail Instance            │
│                                            │
│  ┌──────────────┐                          │
│  │    Nginx     │                          │
│  └──────────────┘                          │
│      │                                     │
│      ├── /            → React 정적 파일     │
│      ├── /api         → FastAPI API        │
│      ├── /ws          → WebSocket          │
│      └── /api/pcb/... → 이미지 응답         │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │ FastAPI + Uvicorn                    │  │
│  │ React Build                          │  │
│  │ YOLO Inference                       │  │
│  │ LangChain / OpenAI API               │  │
│  └──────────────────────────────────────┘  │
│                                            │
│              이미지 저장소                   │
│        ├─ Instance Disk                    │
│        ├─ Block Storage                    │
│        └─ Object Storage                   │
└────────────────────────────────────────────┘
                    │
                    ▼
              MariaDB / MySQL
```

---

# 요청 흐름

```text
사용자
    │
    ▼
HTTPS 요청
    │
    ▼
Nginx
    │
    ├── React 화면 제공
    ├── FastAPI API 전달
    ├── WebSocket 연결
    └── PCB 이미지 응답
          │
          ▼
FastAPI
    │
    ├── DB 조회
    ├── YOLO 추론
    ├── OpenAI API 호출
    └── 이미지 저장
```

---

# Nginx 역할

| 경로 | 역할 |
|------|------|
| `/` | React 빌드 파일 제공 |
| `/api` | FastAPI API 프록시 |
| `/ws` | WebSocket 연결 |
| `/api/pcb/...` | PCB 이미지 응답 |

---

# FastAPI 역할

FastAPI 서버는 다음 기능을 담당한다.

- REST API
- WebSocket
- YOLO 추론
- LangChain/OpenAI 연동
- DB 처리
- 이미지 관리

---

# React

React는

```text
npm run build
```

후 생성되는

```text
dist/
```

폴더를 Nginx가 서비스한다.

즉,

```
Browser
    ↓
Nginx
    ↓
React Build(dist)
```

형태이다.

---

# AI 구성

## YOLO

- PCB 이미지 추론
- 불량 탐지
- Bounding Box 생성

---

## LangChain / OpenAI

담당 기능

- MES 챗봇
- 자연어 질의
- DB 조회 보조
- 리포트 생성

---

# 이미지 저장소

세 가지 선택지가 있다.

| 저장 위치          | 특징                           | 추천도   |
| -------------- | ---------------------------- | ----- |
| Instance Disk  | 가장 단순하지만 인스턴스 삭제 시 데이터 손실 가능 | ⭐⭐    |
| Block Storage  | 별도 디스크처럼 사용 가능               | ⭐⭐⭐⭐  |
| Object Storage | 이미지 저장에 가장 적합(S3 유사)         | ⭐⭐⭐⭐⭐ |

---

# 데이터베이스

## 개발 단계

```text
Lightsail Instance
        │
        └── MariaDB
```

장점

- 설치 간단
- 비용 절감
- 관리 쉬움

---

## 운영 단계

```text
Lightsail Instance
        │
        ▼
Lightsail Managed MySQL
```

장점

- 백업 자동화
- 안정성 향상
- 운영 관리 편리

---

# 전체 구성 요소

| 구성 요소 | 역할 |
|-----------|------|
| Browser | 사용자 |
| Nginx | Reverse Proxy, 정적 파일 제공 |
| React | 사용자 화면(UI) |
| FastAPI | Backend API |
| Uvicorn | FastAPI 실행 서버 |
| YOLO | AI 불량 검출 |
| LangChain/OpenAI | AI 챗봇 및 LLM 기능 |
| MariaDB/MySQL | 데이터 저장 |
| Storage | PCB 이미지 저장 |

---

# 배포 흐름

```text
사용자
    │
    ▼
Nginx
    │
    ├── React
    └── FastAPI
            │
            ├── MariaDB
            ├── YOLO
            ├── OpenAI API
            └── Image Storage
```

---

# 현재 프로젝트 추천 구성

> ✅ **초기 개발**

- AWS Lightsail 인스턴스 1대
- Nginx
- FastAPI + Uvicorn
- React Build
- MariaDB
- YOLO
- LangChain/OpenAI
- Instance Disk 또는 Block Storage

---

> 🚀 **운영 단계**

- AWS Lightsail 인스턴스
- Nginx
- FastAPI + Uvicorn
- React Build
- Lightsail Managed MySQL
- Object Storage
- YOLO
- LangChain/OpenAI

---

# 핵심 요약

> **개발 환경**
>
> Lightsail 인스턴스 하나에 React, FastAPI, YOLO, MariaDB를 함께 구성하여 비용을 최소화한다.

> **운영 환경**
>
> DB는 Managed MySQL로 분리하고, 이미지는 Object Storage를 사용하여 안정성과 확장성을 확보한다.

---











