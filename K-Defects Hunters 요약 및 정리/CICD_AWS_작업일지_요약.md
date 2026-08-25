# CI/CD · AWS 배포 작업일지 (요약)

> **저장소** KDTbigdata8th/pcb-defects-mes · **발표** 2026-08-22 (토) · **최종 갱신** 2026-08-11
> 상세 기록은 `CICD_AWS_배포_작업일지_상세.docx` 참조

---

## 진행 현황

```
사전테스트  ████████████████████  8/9~10  완료
본작업     ████░░░░░░░░░░░░░░░░ 8/11~22 2/10일차
```


| 날짜           | 목표                                  | 상태          |
| ------------ | ----------------------------------- | ----------- |
| 8/09 (일)     | 브랜치 병합 정리, CI 구축                    | ✅           |
| 8/10 (월)     | 배포 파이프라인 연습                         | ✅           |
| **8/11 (화)** | **CI dev 반영, Dockerfile + compose** | **✅ 초과 달성** |
| 8/12 (수)     | 로컬 검증 보완 / 서버 준비 ⚠️판단시점             | ⬜           |
| 8/13 (목)     | Lightsail 4GB 생성, 첫 배포              | ⬜           |
| 8/14 (금)     | 배포 문제 해결 (버퍼)                       | ⬜           |
| 8/15 (토)     | health check, CD 워크플로우 🔒스키마동결      | ⬜           |
| 8/16 (일)     | 스키마 반영 재배포, YOLO fixture            | ⬜           |
| 8/17 (월)     | YOLO 배치 실측, 사양 조정                   | ⬜           |
| 8/18 (화)     | 성능 튜닝, DB 백업 cron                   | ⬜           |
| 8/19 (수)     | 전체 리허설                              | ⬜           |
| 8/20 (목)     | 발표 자료 🔒코드동결                        | ⬜           |
| 8/21 (금)     | 예비일                                 | ⬜           |
| 8/22 (토)     | 🎯 발표                               | ⬜           |

---

## 기준선 수치

| 항목                | 값                                             |
| ----------------- | --------------------------------------------- |
| Backend 테스트       | **312** passed, 2 skipped (8/11 기준)           |
| Frontend 테스트      | **474** passed, 0 failed                      |
| CI 총 소요           | 1분 56초                                        |
| backend-test-fast | 17초                                           |
| backend-test-yolo | 2분 10초                                        |
| frontend-check    | 21초                                           |
| Python / Node     | 3.11 / 20                                     |
| 베이스 이미지           | `ultralytics/ultralytics:8.4.104-cpu` (646MB) |

> skip 2건은 YOLO 스모크 테스트(fixture 미비). 8/16 활성화 예정.

---

## 일일 기록

### 8/09 (일) — CI 구축 ✅

**한 일**
- `feature/process-status-agv-line-checkpoint` (49커밋/76파일) → dev 병합
- `requirements-dev.txt` 생성, `ci.yml` 작성 (2개 잡 병렬)

**결과** Backend 282 / Frontend 474 / CI 1분 56초

**핵심 발견** 프론트 테스트가 CI에서 **8개만 실행되고도 통과**하고 있었음
- 원인: `test/**/*.test.js` glob이 Linux `sh`에서 재귀 확장 안 됨
- 조치: `node --test`로 변경 → 474개 전체 실행
- 부수 발견: `src/` 아래 `factoryMapLayout.test.js` 25개가 한 번도 실행된 적 없었음

---

### 8/10 (월) — 배포 파이프라인 연습 ✅

**범위** 별도 연습 저장소(`KDT-ci-practice`) + 512MB 인스턴스 — 실제 프로젝트 무영향

**결과** `deploy #1 succeeded in 17s`, `{"status":"ok"}`
→ 버튼 클릭 → SSH → git pull → build → 컨테이너 교체 → 자체 헬스체크 전 과정 자동화 확인

**막힌 지점 6건**

| 문제 | 해결 |
|---|---|
| `docker build` 중 Killed (exit 137) | 512MB OOM → 스왑 1GB 추가 |
| `ERR_CONNECTION_TIMED_OUT` | Lightsail 방화벽에 TCP 8000 규칙 추가 |
| 방화벽 규칙 저장 안 됨 | 소스 IP `0.0.0.0/0` 입력 |
| Secrets 메뉴 못 찾음 | 계정 설정 ≠ **저장소** Settings |
| Actions에 워크플로우 미표시 | `.github/` push 누락 |
| `Dockerfile.txt` 생성 | notepad 자동 확장자 → `ren` |

> **512MB는 경량 앱조차 스왑 없이 빌드 실패** → 4GB 결정이 타당함을 실증 확인

---

### 8/11 (화) — CI dev 반영 + 컨테이너화 ✅

**한 일**
- 로컬 dev 최신화 (재고관리 병합분 `e3a28f5`)
- 진행 중 STALE 배치 작업 6파일 → `git stash` 보관 (`stash@{0}`)
- `requirements-base.txt` 분리, CI 잡 **fast/yolo 분리**
- PR #31 → 팀 리뷰 → dev 병합 (`5415d66`)
- `feature/dockerize` 분기 → Dockerfile + docker-compose + nginx.conf
- Docker Desktop 활성화(WSL 설치) → **`docker compose up` 성공**

**결과**
- CI dev 트리거 실전 검증: 3개 잡 전부 성공
- torch 제외 43초 vs 포함 3분 33초 (**약 5배**)
- mariadb Healthy / nginx worker 16개 / backend Uvicorn 정상
- `localhost` 대시보드 렌더링 + WebSocket "실시간 연결됨" 확인

**막힌 지점 6건**

| 문제 | 해결 |
|---|---|
| `DB_PASSWORD is missing` | compose 파일과 같은 위치(루트)에 `.env` 생성 |
| `CREATE USER failed for root@%` | `.env`의 `DB_USER=root` → `pcb_user`, `down -v` 후 재시도 |
| `input/output error` | 디스크 잔여 17.4MB → 16.5GB 확보 |
| `Virtualization support not detected` | Windows 기능에서 Hyper-V / WSL / 가상머신플랫폼 활성화 |
| `auth.docker.io EOF` | 일시적 네트워크 오류, 재시도 |
| `uvicorn backend.app.main:app` 실패 우려 | `__init__.py` 추가 대신 WORKDIR `/app/backend` + `app.main:app` |

> **계획(8/12) 대비 하루 앞당겨 로컬 전체 구동 검증 통과**

---

### 8/12 (수) — AWS 서버 첫 배포 완전 성공 ✅

**범위** 실제 프로젝트 저장소(dev), Lightsail `pcb-mes-prod` (4GB, 실서버)

**한 일**
- process-status 이송 경로 **데드락 위험 발견·수정** (PR #33 → dev 병합)
  - 잠금 순서 통일: `line → batch → buffer slot → waiting batch`
  - `complete_line_transfer`, `complete_post_inspection_transfer` 재정렬
  - `start_post_inspection_transfer` idempotency 신규 구현
  - STALE 배치 처리(고아 INSPECTING → STALE) 동반 반영
- **`feature/dockerize` PR이 `dev`가 아니라 `main`으로 병합됐던 사고 발견** → `main`을 `dev`로 재병합해 정상화
- Lightsail 4GB 인스턴스 생성, 고정 IP 할당, 방화벽(22/80/443만, 3306 제외)
- `/health/live`, `/health/ready` 엔드포인트 신규 구현
- 서버에서 `docker compose up` → 문제 9건 해결 후 전체 스택 정상 기동

**결과**
- `http://3.38.222.241/health/ready` → `{"status":"ok"}`
- `http://3.38.222.241/` → 대시보드 정상, "실시간 연결됨", 실데이터(line 8개) 표출
- Backend 321 passed / Frontend 484 passed

**막힌 지점 9건**

| 문제 | 원인 | 해결 |
|---|---|---|
| `unknown shorthand flag 'd'` | Compose 플러그인 미설치 | `get.docker.com` 스크립트로 재설치 |
| Docker 데몬 시작 실패 | 소켓 활성화 순서 문제 | `docker.socket` 먼저 시작 |
| `undefined volume` | `.env` 경로에 `./` 누락 | `./PCB_defects_DATA/PCB_DATASET` |
| `CREATE USER failed for root@%` | `.env`의 `DB_USER=root` | `pcb_user`로 변경, `down -v` 후 재시도 |
| **nginx 500 무한 리다이렉트 (2회 재발)** | `/health` 라우팅 없음 → **서버 로컬 수정 커밋 안 하고 git pull로 소실** | location 추가 → **즉시 커밋** → 컨테이너 강제 재생성 |
| 프론트 403 Forbidden | Docker가 `frontend/dist`를 root 소유로 생성 | `chown` 후 재빌드 |
| CORS 차단 + WS 실패 | 빌드 JS에 `localhost:8000` 하드코딩 | WebSocket을 `window.location` 기반으로 수정 |
| **환경변수 수정 미반영** | **Vite가 루트 `.env`가 아니라 `frontend/.env`를 읽음** | `frontend/.env` 별도 생성 |
| PAT 반복 입력 | `credential.helper`의 `manager`/`store` 충돌 | 완전 초기화 후 `store`만 재설정 |

> **핵심 교훈**
> 1. 서버에서 파일을 고칠 때는 **그 자리에서 즉시 커밋** — `git pull`이 조용히 되돌림 (2회 실제 발생)
> 2. Vite는 **`frontend/` 기준**으로 `.env`를 찾음 — 저장소 루트 `.env`와 별개 파일

> **계획(8/13) 대비 하루 앞당겨 AWS 서버 첫 배포 완료**

---
### ~8/15


---



## 누적 주의사항

### Docker / 배포
- 서버 빌드 시 OOM(exit 137) 주의 → torch 포함 베이스 이미지 사용, 필요 시 스왑
- 디스크 여유 **최소 3~4GB** 필요
- `docker compose`는 **compose 파일이 있는 디렉토리**의 `.env`를 읽음
- 재시도 시 `docker compose down -v` (볼륨까지 삭제)
- `usermod -aG docker $USER` 후 **재로그인 필수**
- Windows CRLF → Linux 스크립트 실행 실패 가능, `.gitattributes` 고려

### DB
- `DB_USER=root` 금지 (MariaDB 초기화 실패) → `pcb_user`
- 컨테이너 초기화는 **`schema.sql`만**, `migrations/`는 넣지 않음 (컬럼 중복)
- `migrations`의 `ADD COLUMN IF NOT EXISTS`가 MySQL 8.0.36에서 오류 → 운영 DB 마이그레이션 시 사전 검증
- AWS 신규 DB에 **seed 데이터(line 등)** 필요 여부 조사 필요

### Docker / 배포 (추가)
- **Vite는 `frontend/` 기준 `.env`를 읽음** — 저장소 루트 `.env`와 별개, 반드시 `frontend/.env`에도 반영
- Compose 플러그인이 `apt install docker.io`에 없을 수 있음 → `get.docker.com` 스크립트 사용
- `docker.service`는 `docker.socket` 없이 직접 시작 시 실패 (최초 1회만 해당)
- **서버에서 직접 고친 설정 파일은 즉시 커밋** — `git pull`이 조용히 되돌림
- 볼륨 마운트 대상 폴더가 없으면 Docker가 root 소유로 자동 생성 → 빌드 권한 문제, `chown` 필요

### DB (추가)
- `mariadb` 컨테이너 클라이언트 명령은 `mysql`이 아니라 `mariadb`
- 컨테이너 내부 소켓 접속 시 `localhost`로 인식되어 `%` 계정 거부 → `-h 127.0.0.1` 필요

### Git / 브랜치 (신규)
- **PR base가 기본값(main)으로 자동 설정됨** — PR #29~32가 반복 잘못 병합, 매번 dev 확인 필요
- Default branch를 dev로 변경 요청 중 (Admin 권한 필요)
- `credential.helper`는 서버=`store` / Windows=`manager`, 혼용 시 충돌
### CI / 테스트
- 테스트는 **저장소 루트**에서 `pytest backend/tests` 실행해야 통과
- 새 패키지 추가 시 `requirements.txt`와 `requirements-base.txt` **양쪽** 반영
- `schema.sql` 문자열 검사 테스트 3건은 스키마 변경 시 실패 — 예상된 현상
- required check 전환은 **8/15 스키마 동결 이후**

### 개발 에이전트 프롬프트 가드레일
```
[범위] 만들 파일 / 수정 대상 명시
[금지] 그 외 모든 파일 수정 금지, 저장소 내 가상환경 생성 금지
[불확실] 판단이 애매하면 실행하지 말고 먼저 물어봐
```
> 이 원칙으로 "병합이 checkpoint가 아닌 로컬 dev에서 이뤄진" 사실을 사전 발견

---

## 보안 · 비용 가드레일

- **mariadb에 `ports:` 항목 쓰지 않음** — 호스트에 3306 미노출, 내부망만
- Lightsail 방화벽: **22 / 80 / 443만**, 3306 등록 금지
- 서버 내부 `ufw default deny incoming` 이중 방어
- AWS Budgets 알림 $30 / $60 / $80 유지
- DB 비밀번호는 `.env`로만 (`.gitignore` 확인)

**AWS 계정** 무료 플랜 · 크레딧 $100 (만료 2027-07-10) · 플랜 종료 2027-01-10 예상
→ 발표일이 훨씬 앞서므로 **만료 리스크 없음**

---

## 확인 · 대기 항목

### 인프라
- [x] `docs/2026-08-07-db-schema-reference.md` diff 확인 후 dev 커밋
- [x] `feature/dockerize` 커밋 및 PR (단, main으로 잘못 병합 → dev 재병합 완료)
- [ ] `.env.example`의 `DB_USER` 기본값 `root` → `pcb_user` (팀 공유 후)
- [ ] AWS 추가 크레딧 $100 획득 (기한 2027-01-10)
- [ ] 연습용 8000 포트 방화벽 규칙 정리 (512MB 연습 인스턴스)
- [ ] 3306 규칙(`58.227.90.78/32`) 용도 확인 (512MB 연습 인스턴스)
- [x] AWS 신규 DB용 seed 데이터 조사 — line 8개 존재 확인, 추가 조치 불필요
- [ ] **저장소 Default branch를 dev로 변경 요청** (Admin 권한 필요)
- [ ] **`pcb-mes-prod`에 배포 전용 SSH 키 등록** (CD 워크플로우 연결용)
- [ ] **`OPENAI_API_KEY` 발급 → `.env` 반영 → 챗봇 Provider 전환**
- [ ] **`ufw` 이중 방어가 `pcb-mes-prod`에 적용됐는지 재확인**
- [ ] `get-docker.sh` 정리 (서버, 우선순위 낮음)

### 기능
- [x] **공정상태 규칙 정리** — 완료
- [x] **`BATCH-20260811-0014` 관련 데드락 수정** (PR #33)
      - 미해결(범위 밖): `create_batch`/`cancel_batch`의 `plan↔batch` 잠금 순환
      - 미검증: 실제 MariaDB 동시 요청 통합 테스트 (Mock만 수행)
- [x] `stash@{0}` STALE 배치 작업 재개 — 커밋 완료 (`836d94c`)
- [ ] `main.py:32` `on_event("startup")` deprecation (우선순위 낮음)
- [ ] npm audit 취약점 4건 (우선순위 낮음)

### 완료 (추가)
- [x] **AWS Lightsail 서버 첫 배포 완료 (계획보다 하루 조기)**
- [x] **health check 엔드포인트 구현 및 검증**
- [x] **process-status 데드락 위험 수정 (PR #33)**
- [x] CI 구축 및 dev 반영 (PR #31)
- [x] 프론트 테스트 복구 (8 → 474)
- [x] 배포 파이프라인 연습 완주
- [x] Dockerfile / docker-compose / nginx.conf 작성
- [x] 로컬 전체 스택 구동 검증
- [x] torch 없이 pytest 수집 확인 (가능, 코드 수정 불필요)

---

## 중단 기준 (Go / No-Go)

| 기한   | 조건               | 대응                |      |
| ---- | ---------------- | ----------------- | ---- |
| 8/12 | 로컬 compose 구동 실패 | AWS 포기, 로컬 데모     | ✅ 통과 |
| 8/13 | OpenAI 전환 실패     | 챗봇 데모 제외          |      |
| 8/15 | 재고관리 미완          | 미완 상태로 동결         |      |
| 8/16 | CD 미완            | 수동 배포 유지          |      |
| 8/17 | 서버 YOLO 추론 실패    | 8GB 증설 → 사전 계산 결과 |      |

**우선순위: 배포 > 챗봇 > CI/CD**

---

## 아키텍처 (채택안)

```
브라우저
   │
   ▼
Lightsail 4GB "pcb-mes-prod" (3.38.222.241) ← 실서버 배포 완료
 └─ Docker Compose
     ├─ nginx    : React dist 서빙 + /api·/ws 프록시 (CORS 문제 소멸)
     ├─ backend  : FastAPI + YOLO
     └─ mariadb  : DB (포트 미노출, 내부망만)
 └─ 볼륨 : PCB 이미지, best.pt (이미지에 굽지 않음)
```

**예상 비용 월 $20~25** (크레딧 $100으로 4개월 이상)

Codex 제안(ECS+ALB+RDS+SQS+NAT)은 월 $91~120으로 크레딧이 한 달 내 소진 → 발표 범위 제외, "목표 아키텍처"로 발표 자료 활용

---

## 발표 이후 로드맵

1. 이미지·모델 S3 이전
2. CD 자동 트리거 전환
3. CI required check 승격
4. DB 관리형 분리 (PITR)
5. HTTPS + 도메인
6. DR 체계 (RTO 10분)
7. 상태 외부화 후 HA 전환

> 팀 프로젝트 범위에서는 **무중단(HA)보다 빠른 복구(DR)**에 투자하는 것이 합리적.
> 현재 구성은 앱이 stateless이므로 로드밸런서 추가만으로 HA 확장 가능.
