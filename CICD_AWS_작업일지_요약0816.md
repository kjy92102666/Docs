# CI/CD · AWS 배포 작업일지 (요약)

> **저장소** KDTbigdata8th/pcb-defects-mes · **발표** 2026-08-22 (토) · **최종 갱신** 2026-08-16
> 상세 기록은 `CICD_AWS_배포_작업일지_상세.docx` 참조

---

## 진행 현황

```
사전테스트  ████████████████████  8/9~10  완료
본작업     ████████████████░░░░ 8/11~22 6/10일차
```


| 날짜           | 목표                                 | 상태                                 |
| ------------ | ---------------------------------- | ---------------------------------- |
| 8/09(일)      | 브랜치 병합 정리, CI 구축                   | ✅                                  |
| 8/10 (월)     | 배포 파이프라인 연습                        | ✅                                  |
| 8/11 (화)     | CI dev 반영, Dockerfile + compose    | ✅ 초과 달성                            |
| 8/12 (수)     | 로컬 검증 보완 / 서버 준비 ⚠️판단시점            | ✅ 초과 달성 (AWS 첫 배포까지)               |
| 8/13 (목)     | Lightsail 4GB 생성, 첫 배포             | ✅                                  |
| 8/14 (금)     | 배포 문제 해결 (버퍼)                      | ✅                                  |
| 8/15 (토)     | health check, CD 워크플로우 🔒스키마동결     | ✅                                  |
| **8/16 (일)** | **스키마 반영 재배포, YOLO fixture**       | **✅ 데이터 시딩·공정상태 버그 수정 완료 (YOLO 모델 업로드는 미착수, 8/17로 이월)** |
| 8/17 (월)     | YOLO 배치 실측, 사양 조정                  | ⬜                                  |
| 8/18 (화)     | 성능 튜닝, DB 백업 cron                  | ⬜                                  |
| 8/19 (수)     | 전체 리허설                             | ⬜                                  |
| 8/20 (목)     | 발표 자료 🔒코드동결                       | ⬜                                  |
| 8/21 (금)     | 예비일                                | ⬜                                  |
| 8/22 (토)     | 🎯 발표                              | ⬜                                  |

---

## 기준선 수치

| 항목                | 값                                             |
| ----------------- | --------------------------------------------- |
| Backend 테스트       | **322** passed, 2 skipped (8/15 기준, 챗봇 provider 테스트 1건 추가) |
| Frontend 테스트      | **474** passed, 0 failed                      |
| CI 총 소요           | 1분 56초                                        |
| backend-test-fast | 17초                                           |
| backend-test-yolo | 2분 10초                                        |
| frontend-check    | 21초                                           |
| Python / Node     | 3.11 / 20                                     |
| 베이스 이미지           | `ultralytics/ultralytics:8.4.104-cpu` (646MB) |
| Frontend 테스트 (8/16 갱신) | **522** passed (AGV 애니메이션 동기화 테스트 3건 추가) |
| DB 테이블 수 | **17개** (8/15 v10 기준, v9 대비 신규 2개: `post_inspection_buffer_slot`, `batch_material_consumption`) |
| original_pcb 데이터 | **703건** (8/15 이미지 업로드·시딩 완료) |
| 시딩된 배치 데이터 (8/16) | production_batch 4건, batched_pcb 576건, inspection_result 576건, defect_review 39건 |

> skip 2건은 YOLO 스모크 테스트(fixture 미비). `best.pt` 모델 가중치 파일 소재 미확인 — 8/17 확인 예정(8/16→8/17 이월).

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

### 8/13 (목) — SSH 키 재발급 + CI/CD 파이프라인 완성 ✅

**한 일**
- 서버 상태·DB 데이터 현황 확인 (line 8개·product_model 10개만 존재, 나머지 15개 테이블 0건 — 정상 판단)
- ufw 이중 방어 적용 완료 (default deny incoming, 22/80/443 allow)
- **SSH 배포 키 보안 사고 발견·조치**: GitHub Repository Variables(평문)에 `SSH_HOST`/`SSH_PRIVATE_KEY`/`SSH_USER`를 잘못 등록했던 것 발견 → Variables 삭제, 새 키(`v2`) 발급, 서버 `authorized_keys` 교체, Secrets(암호화)로 재등록
- `deploy.yml` 작성·push

**막힌 지점 — GitHub Actions가 Deploy 워크플로우를 인식 못함**
- 증상: `deploy.yml`이 저장소에 정상 존재·커밋됐는데도 Actions 목록에 전혀 안 뜸
- **원인 확정**: GitHub Actions는 **Default Branch에 실제로 존재하는 워크플로우 파일만 인덱싱**함. 당시 Default Branch가 `main`이었는데 `deploy.yml`은 `dev`에만 있었음
- 조치: **Default Branch를 `main → dev`로 변경**(Admin 권한 확보 후) → 즉시 인식됨

**결과**
- Deploy 워크플로우 정상 노출, `Run workflow` 수동 배포 테스트 성공 (git pull → docker build/재기동 → 헬스체크 통과)
- 로컬 SSH 키 파일 4개 `.gitignore` 처리, 과거 커밋 이력 오염 없음 확인

> **8/15 재검토 결과**: 이 Default Branch 변경은 "워크플로우 인식 문제"라는 당시 상황을 해결하기 위한 **임시 조치**였음. 원래 팀 설계(main=운영, dev=개발)와는 반대 방향 조치였고, 8/15에 `deploy.yml`을 main에도 반영하면서 근본 원인이 해소됨 → Default Branch를 다시 `main`으로 되돌리는 게 맞음 (8/16 이후, Admin 계정 확보 시 처리 예정, 미완료)

---

### 8/14 (금) — 챗봇 로컬 개발 + 팀 CI 이슈 해결 ✅

**한 일**
- OpenAI 챗봇 로컬 검증(1단계) 완료: `openai_provider.py` 구현, `CHATBOT_PROVIDER` 환경변수 분기, 예외 4종을 Ollama와 1:1 대응 구조로 작성
- **버그 발견**: `ChatResponse.provider` 필드가 실제 호출된 AI와 무관하게 항상 `"ollama"`로 하드코딩 → `ChatProvider`에 `name` 속성 추가, `service.py`에서 `provider.name` 반환하도록 수정. 부수적으로 발견한 테스트 격리 문제(로컬 `.env`의 `CHATBOT_PROVIDER` 값이 무관한 라우터 테스트에 영향)도 함께 수정
- 로컬 커밋(`43f3956`), 팀 합의 전까지 push 보류
- **다른 팀원 PR(#38, 분석 리포트) 1차 CI 실패 해결**: `openpyxl` 모듈 누락 → CI가 `requirements-base.txt`만 설치하는데 `openpyxl`은 `requirements.txt`에만 있었음 → `requirements-base.txt`에 추가
- **분석 리포트 기능 main 배포**: 원래 설계(main=운영/AWS) 재확인, `deploy.yml`을 `dev` 기준에서 `main` 기준으로 전환. nginx가 `frontend/dist`를 볼륨 마운트로 직접 서빙하는 구조 확인, 배포 스크립트에 `npm ci && npm run build` 추가 (기존엔 프론트 빌드 누락 상태였음)
- PR #41(`deploy.yml` 수정) → dev → main 병합 완료

**막힌 지점**
- dev push 거부 반복 (다른 팀원들이 그 사이 dev에 병합) → `git fetch` 후 `git pull --rebase origin dev`로 해결, 이후에도 계속 반복되는 패턴으로 확인됨
- 배포 성공 로그인데도 Actions엔 실패(X)로 기록되는 현상 최초 발견 (원인 미파악 상태로 8/15로 이월)

---

### 8/15 (토) — 챗봇 서버 반영 + 스키마 동결 + 데이터 업로드 ✅

**한 일**

1. **챗봇 서버 최종 배포**
   - `feature/chatbot-openai` → dev → main 병합
   - **서버 인프라 배선 누락 발견**: `docker-compose.yml`의 backend `environment:`에 `CHATBOT_PROVIDER`/`OPENAI_API_KEY`/`OPENAI_MODEL`이 전혀 없어서, `.env`에 값을 넣어도 컨테이너에 전달 안 됨 → 3줄 추가로 해결
   - 서버 `.env`에 실제 API 키 반영, main으로 재배포
   - 최종 검증: `curl POST /api/chatbot/messages` → `"provider":"openai"` 정상 확인

2. **8/14에 발견한 "성공했는데 실패로 기록되는" 문제 원인 규명·해결**
   - 원인: `deploy.yml`이 `docker compose up -d --build` 직후 `sleep 3` 뒤 헬스체크를 **단 1회만** 시도 → backend가 늦게 뜨면 502로 스크립트 실패(`set -e`) → 실제 배포는 성공인데 Actions엔 실패로 기록되는 가짜 실패였음
   - 조치: 3초 간격 최대 20회(약 60초) 재시도로 변경, 실제 배포에서 정상 통과 검증 완료

3. **DB 백업 방법 확보**: `mariadb-dump` 수동 백업 명령 검증 완료 (23KB 정상 생성). 자동화(cron)는 8/18 예정대로 보류
   ```bash
   docker compose exec mariadb mariadb-dump -h 127.0.0.1 -u root -p pcb_mes > backup_$(date +%Y%m%d).sql
   ```

4. **스키마 동결 — ERD/테이블 정의서 v9 → v10**
   - v9(7/31) 이후 미문서화된 스키마 변경 전수 조사 (git log 2026-08-01~08-10 기준)
   - 신규 테이블 2개(`post_inspection_buffer_slot` — 후검사 버퍼 슬롯, 8/6 / `batch_material_consumption` — SMD 자재 소비 이력, 8/8) 확인, 둘 다 백엔드+프론트 완전 연동되어 실사용 중
   - `production_batch` 물류 라이프사이클 컬럼 8개, `inspection_result` 다중 검출/BBox 5개, `defect_review.corrected_defect_type` 문서 반영
   - `PCB_테이블_정의서_v10.docx` 작성 완료 (17개 테이블 전체). ERD 다이어그램(.dbml)은 별도 대화창에서 진행
   - v9 "검토 필요 사항" 8개 항목 재확인 → 발표 범위 내 막힌 핵심 미결 사항 없음 확정
   - **알려진 이슈로 재확인(미해결)**: `inspection_ready` 계산 로직에 `line_id` 검증 누락

5. **PCB 원본 이미지 700장 서버 업로드**
   - 접근 방식 확정: Google Drive API 연동 아님, **로컬 파일시스템 방식**(`PCB_DATASET_PATH`) — Drive for Desktop 마운트는 가능하나 API 미사용
   - `images`(693장) + `PCB_USED`(10장) = 703장(약 998MB) scp 전송
   - `backend/db/pcb_seed.sql` 로드 → `original_pcb` 703건 확인

6. **다른 팀원 PR(#38) 2차 CI 실패 해결**: 일부 테스트가 "mock 없이 실 DB 검증"용으로 설계됐는데 CI엔 DB 자체가 없어 `Connection refused` → `@pytest.mark.skipif(os.getenv("CI")=="true")`로 CI에서만 스킵 (임시 조치, 정식 해결은 8/18 이후 CI에 MariaDB 서비스 컨테이너 추가 예정)

7. **데이터 시딩 API 스펙 조사 완료** (실행은 8/16로 이월)
   - `POST /api/plans/{plan_id}/batch` (배치 생성)
   - `POST /api/batches/{batch_id}/line-transfer/complete` (라인 이송 완료)
   - `POST /api/batches/{batch_id}/inspect` (시뮬레이션 검사, YOLO 모델 없이도 작동 확인)

**막힌 지점 / 재발 패턴**

| 문제 | 원인 | 해결 |
|---|---|---|
| dev push 거부 (3회 반복) | 다른 팀원이 그 사이 dev에 병합, 작업 컴퓨터도 실습실→노트북 전환되며 로컬 dev가 크게 뒤처짐(18커밋) | `git fetch` → 대상 파일 충돌 여부(`git log origin/dev -- <파일>`) 확인 → `git pull --rebase origin dev` |
| 팀원 PR CI 실패 (openpyxl) | `ci.yml`의 `backend-test-fast`가 `requirements-base.txt`만 설치 | `openpyxl`을 `requirements-base.txt`에도 추가 |
| 팀원 PR CI 실패 (DB 연결) | "mock 없이 실 DB 검증"용 테스트인데 CI엔 DB 없음 | `@pytest.mark.skipif(os.getenv("CI")=="true")`로 CI에서만 스킵 |
| 배포는 성공, Actions는 실패(X) | `deploy.yml` 헬스체크가 `sleep 3` 후 단 1회만 curl, backend 늦게 뜨면 502로 `set -e`에 걸려 스크립트 전체 실패 | 3초 간격 최대 20회(약 60초) 재시도로 변경 |
| 노트북에서 SSH 접속 안 됨 | 8/13 만든 키(`pcb-mes-prod-deploy-v2`)가 실습실 PC에만 있음. 저장해둔 개인키 텍스트를 파일로 복원 시도했으나 `invalid format` 지속 | 원인 규명 대신 노트북용 새 키 발급 후 서버 `authorized_keys`에 추가하는 방식으로 우회 |
| 이미지 폴더(Google Drive)가 노트북에서 완전히 비어있음 | 착오 — 실제로는 로컬 디스크 방식이 맞았고, Drive 폴더 자체가 이 계정/컴퓨터에 동기화된 적 없었음 | 로컬(`Desktop\PCB_DATASET`)에서 실제 703장 데이터 재발견, 거기서 scp 전송 |
| `docker-compose.yml`에 챗봇 환경변수 배선 누락 | `.env`에 `OPENAI_API_KEY=` 자리는 미리 있었으나 `environment:` 목록엔 연결 안 됨 | `CHATBOT_PROVIDER`/`OPENAI_API_KEY`/`OPENAI_MODEL` 3줄을 `environment:`에 추가 |

---

### 8/16 (일) — 데이터 시딩 완료 + 공정상태 다중 사용자 동기화 버그 수정 ✅

**범위**: 데이터 시딩(어제 API 조사 완료분 실행), 공정상태 화면 실사용 검증 중 발견한 버그 조사·수정. YOLO 모델 업로드는 착수 못하고 8/17로 이월.

**한 일**

1. **배치/검사 데이터 시딩 실행** (기존 `production_plan` 4건 전량)
   - plan 1(8개)·2(8개)·3(460개)·4(100개) 순서로 배치 생성 → 라인 이송 완료 → 검사(시뮬레이션 API `/inspect`) 실행
   - 결과: `production_batch` 4건, `batched_pcb`/`inspection_result` 576건, `defect_review` 39건 정상 생성
   - `curl.exe` 실행 자체가 이 컴퓨터에서 원인 불명으로 차단되는 문제 발생 → **PowerShell `Invoke-RestMethod`로 전환**하여 계속 진행 (curl 문제 자체는 원인 미규명, 우회만 함)

2. **`post_inspection_buffer_slot` 0건 문제 발견·해결**
   - plan 4 배치 생성 시 `LINE_MODEL_CONFLICT`(라인 점유 중) → plan 3을 후검사 버퍼로 이송하려 하자 `POST_INSPECTION_BUFFER_FULL` 발생
   - 원인: 8/6 migration(`20260806_add_process_status_lifecycle.sql`)의 "16개 슬롯 초기 INSERT"가 `docker-entrypoint-initdb.d`에 마운트 안 되어 있어, **기존 mariadb_data 볼륨을 계속 재사용해온 이 서버에는 한 번도 실행된 적 없었음** (schema.sql만 자동 실행되는 구조)
   - 조치: 16개 슬롯 수동 INSERT (`slot_no` 1~16, `status='EMPTY'`)로 즉시 해결
   - **부가 발견**: `POST_INSPECTION_BUFFER_FULL` 에러가 "테이블 행 0건"과 "행은 있으나 EMPTY 0건"을 구분하지 않고 동일하게 반환 (`backend/app/features/process_status/repository.py`) — 지금 당장 문제는 아니지만 향후 디버깅 시 혼동 소지 있음, 기록만 해둠

3. **공정상태 화면 로직 3건 검증** (사용자 우려 제기 → 순차 조사)
   - **트랜잭션 범위**: 계획/배치 생성, `/inspect`(시뮬레이션)는 전체 원자적 트랜잭션. YOLO `/start`는 PCB 단위 개별 트랜잭션(장시간 비동기 검사 고려한 설계). review 확인·공정 이송(버퍼/최종)은 API 호출별 트랜잭션. 전체 업무 흐름을 하나로 묶은 트랜잭션은 없으나, 이는 의도된 표준 패턴으로 판단(락 장시간 점유 방지) — **문제 없음**
   - **batch 3/4가 `/inspect` 후 다르게 동작(버퍼 이송 여부)한 원인**: 백엔드 `/inspect`는 검사만 수행, 버퍼 자동 이송 로직 없음. **원인은 프론트엔드**(`ProcessStatus.jsx`)가 WebSocket으로 상태 변화 감지 시 `INSPECTED + LINE` 조건이면 자동으로 `post-inspection-transfer/start`를 호출하는 구조 — batch 4는 화면이 열려있어 자동 진행, batch 3은 당시 버퍼 슬롯 0건이라 자동 시도가 실패했던 것으로 추정. **의도된 기능, 버그 아님**
   - **⚠️ 새로고침 시 AGV 애니메이션이 경로 시작점부터 재생되는 문제 — 실제 버그로 확정**
     - DB 상태 자체는 `idempotent` 처리로 항상 안전 (재검증 완료, 테스트 존재)
     - 그러나 `resumeServerTransfer()`(`frontend/src/domains/process-status/processStatusModel.js`)가 `batch.transfer_started_at`을 전혀 사용하지 않고 항상 경로 0%에서 애니메이션을 시작
     - **AWS 배포 목적(여러 사용자 동시 시연)과 정면 충돌**: 서로 다른 시점에 접속한 두 브라우저가 같은 배치를 보면서도 서로 다른 AGV 위치를 표시하는 것을 코드 흐름으로 확인 → 우선순위를 YOLO보다 올려 즉시 수정 착수

4. **AGV 애니메이션 동기화 버그 수정** (PR #46 → dev → main, `fix/agv-animation-sync` 브랜치)
   - `positionAlongRoute()` 신규 함수: route 배열을 따라 주어진 거리만큼 이동한 지점의 정확한 좌표/routeIndex 계산
   - `resumeServerTransfer()`가 `transfer_started_at` 기준 경과 시간 × `MOVE_SPEED(18)`로 이동 거리를 계산해 실제 위치에서 애니메이션 시작하도록 수정
   - 이동 시간 초과 시(이미 도착했어야 하는 경우) 경로 끝 지점에 배치, 기존 도착 처리와 자연스럽게 연결
   - `transfer_started_at` 없을 시 기존 동작(0%) 유지하는 fallback 보존
   - 같은 세션 내 기존 AGV 이어받기 로직은 미변경
   - 신규 테스트 3건 추가(`processStatusLifecycle.test.js`), 전체 프론트 테스트 522 passed, `npm run build` 성공
   - 파일 최상단에 시뮬레이션 모듈 전체 동작 원리 요약 주석 추가 (한글, 기존 파일은 영어 주석 — 톤 불일치 있으나 우선 보류)
   - `.gitignore`의 SSH 키 패턴을 `pcb-mes-prod-deploy*` → `pcb-mes-*-deploy*`로 확장 (컴퓨터별 다른 이름의 키 실수 커밋 방지 목적, 작업 중 개인키 파일이 untracked 상태로 노출될 뻔한 것 발견 후 조치)
   - dev(PR #46) → main(PR #47 상당) → `Run workflow`(main, Deploy #7) 배포 완료, `Success` 확인

**막힌 지점**

| 문제 | 원인 | 해결 |
|---|---|---|
| `curl.exe` 실행 자체가 차단됨(`Access is denied.`, "현재 PC에서는 이 앱을 실행할 수 없습니다") | 원인 미규명. Windows 보안 보호 기록에도 관련 차단 이력 없음, curl.exe/git 등 0바이트 빈 파일이 프로젝트 폴더에 생성되는 등 이례적 현상 동반 | PowerShell `Invoke-RestMethod`로 전환해 우회, 근본 원인은 미해결로 남김 |
| `plans/{id}/batch` 요청이 curl 차단 중 실제로는 서버에 성공 처리됨(`PLAN_ALREADY_BATCHED`로 뒤늦게 발견) | curl이 막혀 응답을 못 받았을 뿐 요청 자체는 서버에 도달·처리됨 | DB 직접 조회로 실제 상태(batch_id, transfer_id) 확인 후 이어서 진행 |
| `post_inspection_buffer_slot` 0건 | migration이 `docker-entrypoint-initdb.d`에 마운트 안 되어 기존 볼륨 재사용 서버에 미반영 | 수동 INSERT 16개 슬롯 |
| SSH DB 접속 시 `no configuration file provided` | `docker compose` 명령을 `~/pcb-defects-mes` 밖에서 실행 | `cd ~/pcb-defects-mes` 후 재실행 |
| DB 비밀번호 변수(`$DB_ROOT_PW`)가 매번 빈 값 | 새 SSH 세션마다 초기화됨(전날 세션에만 설정됐던 것) | 매 세션마다 `DB_ROOT_PW=$(grep DB_ROOT_PASSWORD .env | cut -d '=' -f2)` 재실행 필요 — 반복 재발 |
| `feature/process-status-agv-line-checkpoint` 브랜치 재사용 검토 | 8/9에 이미 dev 병합 완료된 오래된 브랜치로 확인됨, 그대로 체크아웃 시 rebase 충돌 위험 | dev에서 새 브랜치(`fix/agv-animation-sync`) 생성으로 대체 |
| 프로젝트 폴더에 `curl.exe`, `git`, `how efcb842 --stat` 등 낯선 0바이트 파일 다수 발견 | curl 차단 사건과 연관된 것으로 추정(정확한 발생 경위 불명) | 삭제 조치, `.gitignore` 패턴도 함께 보강해 커밋 전 재확인하는 습관 필요 |

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
- ~~Default branch를 dev로 변경 요청 중~~ → **8/13 완료(당시 워크플로우 인식 문제 해결 목적), 8/15 재검토 결과 다시 main으로 되돌릴 예정(미완료, 원래 설계는 main=운영)**
- `credential.helper`는 서버=`store` / Windows=`manager`, 혼용 시 충돌
- **dev는 여러 팀원이 동시에 병합하는 공용 브랜치** — push 전 항상 `git fetch` 후 `git log origin/dev --oneline -5`로 새 커밋 확인, 거부되면 `git pull --rebase origin dev`
- **PR 규모는 "작을수록 안전"** — 8/15에 커밋 1~2개짜리 소규모 PR(deploy.yml, docker-compose.yml)을 여러 번 나눠 진행, rebase 충돌 위험 최소화 확인
- **작업 컴퓨터를 바꾸면 로컬 브랜치가 크게 뒤처질 수 있음** — 새 컴퓨터에서는 `git fetch` 후 반드시 `git log --oneline -5`로 로컬/원격 커밋 일치 여부부터 확인

### CI / 테스트
- 테스트는 **저장소 루트**에서 `pytest backend/tests` 실행해야 통과
- 새 패키지 추가 시 `requirements.txt`와 `requirements-base.txt` **양쪽** 반영
- `schema.sql` 문자열 검사 테스트 3건은 스키마 변경 시 실패 — 예상된 현상
- required check 전환은 **8/15 스키마 동결 이후**
- **"mock 없이 실 DB 검증" 테스트는 CI 환경(DB 없음)에서 반드시 실패** — 작성 시 `@pytest.mark.skipif(os.getenv("CI")=="true")` 사전 적용 권장, 정식 해결은 CI에 MariaDB 서비스 컨테이너 추가(8/18 예정)

### CD / 배포 워크플로우 (신규, 8/15)
- **GitHub Actions는 Default Branch에 있는 워크플로우 파일만 인식** — `dev`에만 있고 Default Branch가 `main`이면 Actions 탭에 아예 안 뜸. `deploy.yml`을 두 브랜치 모두에 반영해두면 이 문제가 근본적으로 재발하지 않음
- **배포 스크립트 헬스체크는 재시도 로직 필수** — 단발성 curl은 backend 기동 지연 시 "실제 성공·기록상 실패"라는 혼란스러운 가짜 실패를 만듦. 3초 간격 다회 재시도 패턴 채택
- **`docker-compose.yml`의 `environment:`에 명시되지 않은 변수는 `.env`에 값이 있어도 컨테이너에 전달 안 됨** — 새 환경변수 추가 시 `.env`와 `docker-compose.yml` **양쪽** 다 확인
- **`Run workflow` 실행 시 브랜치 선택을 반드시 확인** — 목록에 여러 브랜치가 뜨므로 실수로 옛 버전(dev)을 선택하지 않도록 주의

### DB 볼륨·migration (신규, 8/16)
- **`docker-entrypoint-initdb.d`에 마운트된 스크립트는 볼륨이 처음 생성될 때 딱 한 번만 실행됨** — 우리 서버는 볼륨을 계속 재사용 중이라, `schema.sql`(테이블 구조)만 최초 반영되고 `migrations/` 폴더의 초기 데이터 INSERT는 이후 추가돼도 서버에 자동 반영 안 됨
- 실제로 8/6 migration(버퍼 슬롯 16개)이 이 문제로 누락되어 있었음 — **앞으로 새 migration이 생길 때마다 서버에 수동 반영 필요**, 자동화 방법은 8/18 이후 검토
- SSH 세션에서 설정한 쉘 변수(`DB_ROOT_PW` 등)는 **세션 종료 시 사라짐** — 새 SSH 접속마다 재설정 필요, 매번 빈 값으로 조용히 실패할 수 있으니 `echo "[$VAR]"`로 먼저 확인하는 습관 필요

### 공정상태 화면 / 프론트 상태 동기화 (신규, 8/16)
- **공정상태 화면(`ProcessStatus.jsx`)이 검사 완료 배치를 자동으로 다음 단계까지 진행시킴** — 백엔드가 아니라 프론트가 WebSocket 상태 변화를 감지해 자동으로 이송 API를 호출하는 구조(의도된 기능). 데모 중 "안 눌렀는데 저절로 이동"해도 정상이니 팀 전체 공유 필요
- **AGV 애니메이션은 브라우저별 로컬 상태로 실행되며, 서버 시각(`transfer_started_at`) 기반 위치 보정이 없으면 새 세션마다 경로 0%부터 재생됨** — 데이터 상태는 항상 안전(`idempotent` 처리)하지만, 화면 표시는 접속 시점에 따라 달라질 수 있음. 여러 사용자가 동시 접속하는 시연 환경에서는 반드시 서버 경과 시간 기반 위치 계산을 구현해야 함 (8/16 수정 완료)
- **로컬 개발 환경에서 `curl.exe`가 원인 불명으로 실행 차단되는 경우가 있었음** — Windows 보안 보호 기록에 특별한 차단 이력 없이 발생. 재발 시 PowerShell `Invoke-RestMethod`로 즉시 전환해 우회하는 것을 권장 (curl 문제 자체 해결에 시간 쓰지 않기)
- **API 요청이 클라이언트 쪽에서 실패한 것처럼 보여도 서버에는 이미 처리됐을 수 있음** — 응답을 못 받은 것과 요청이 실패한 것은 다름. 애매하면 DB를 직접 조회해서 실제 상태부터 확인 후 재시도 여부 판단할 것

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
- [x] ~~저장소 Default branch를 dev로 변경 요청~~ (8/13 완료) → **8/15: 다시 main으로 되돌릴 것 (미완료, Admin 계정 필요)**
- [x] **`pcb-mes-prod`에 배포 전용 SSH 키 등록** (8/13 완료, v2 키)
- [x] **`OPENAI_API_KEY` 발급 → `.env` 반영 → 챗봇 Provider 전환** (8/15 완료, 서버 반영·검증까지)
- [ ] **`ufw` 이중 방어가 `pcb-mes-prod`에 적용됐는지 재확인**
- [ ] `get-docker.sh` 정리 (서버, 우선순위 낮음)
- [x] **`deploy.yml`을 main 기준으로 전환 + 프론트 빌드(`npm run build`) 단계 추가** (8/14~15)
- [x] **`deploy.yml` 헬스체크 재시도 로직 추가** (8/15, 가짜 실패 문제 해결)
- [x] **`docker-compose.yml` 챗봇 환경변수 배선** (8/15)
- [x] **DB 백업 방법(수동) 확보** (8/15)
- [x] **ERD/테이블 정의서 v9 → v10 갱신** (8/15, 17개 테이블)
- [x] **PCB 원본 이미지 700장(703건) 서버 업로드 + 시딩** (8/15)
- [x] **배치/검사 데이터 시딩 실행** (8/16 완료 — plan 1~4 전량, production_batch 4/batched_pcb 576/inspection_result 576/defect_review 39건)
- [x] **`post_inspection_buffer_slot` 0건 문제 발견·해결** (8/16, migration 미반영이 원인, 수동 INSERT로 조치)
- [x] **AGV 애니메이션 다중 세션 동기화 버그 수정** (8/16, PR #46, main 배포·검증 완료)
- [ ] **YOLO 모델(`best.pt`) 소재 확인 + 서버 업로드** (8/15→8/16 순연, 8/17로 재이월 — 아직 미착수)
- [ ] CI에 MariaDB 서비스 컨테이너 추가 — 실 DB 검증 테스트 CI 정식 지원 (8/18 예정)
- [ ] SSH 키 파일 로컬 정리 (발표용 단기 프로젝트라 우선순위 낮음, 보류 결정)
- [ ] DB 볼륨 재사용 시 migration 자동 미실행 문제 — `docker-entrypoint-initdb.d` 마운트 방식 개선 또는 배포 스크립트에 migration 실행 단계 추가 검토 (8/16 실제로 재현·확인됨, 8/18 이후 정식 해결 예정)
- [x] 공정상태 프론트 자동 이송 로직 문서화 — 원인 규명 완료, 본 일지에 기록 (팀 공유는 별도 진행 필요)
- [ ] `POST_INSPECTION_BUFFER_FULL` 에러가 "테이블 0건"과 "EMPTY 0건"을 구분 안 하는 문제 — 우선순위 낮음, 참고용 기록만
- [ ] `curl.exe`가 로컬(노트북) 환경에서 실행 차단되는 원인 규명 — 급하지 않음, PowerShell로 우회 중
- [ ] AGV 애니메이션 최상단 요약 주석이 한글로 작성되어 기존 파일(전체 영어 주석)과 톤 불일치 — 발표 후 정리 검토


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
- [x] **SSH 배포 키 보안 사고 조치 (평문 노출 발견 → 재발급) (8/13)**
- [x] **Deploy 워크플로우 GitHub Actions 인식 문제 해결 (8/13)**
- [x] **챗봇 OpenAI Provider 구현 + 로컬 검증 (8/14)**
- [x] **`ChatResponse.provider` 하드코딩 버그 수정 (8/14)**
- [x] **분석 리포트(히트맵/Excel/PDF export) 기능 main 배포 (8/14)**
- [x] **챗봇 기능 서버 반영 완료, `provider:"openai"` 실증 확인 (8/15)**
- [x] **`deploy.yml` 가짜 실패(헬스체크 타이밍) 문제 원인 규명·해결 (8/15)**
- [x] **DB 수동 백업 방법 확보 (8/15)**
- [x] **ERD/테이블 정의서 v10 갱신, 17개 테이블 전수 검증 (8/15)**
- [x] **PCB 원본 이미지 703건 서버 업로드·시딩 완료 (8/15)**
- [x] **배치/검사 데이터 시딩 실행 완료 (8/16, 576건 검사·39건 검토 대상 생성)**
- [x] **`post_inspection_buffer_slot` migration 미반영 문제 발견·수동 조치 (8/16)**
- [x] **공정상태 트랜잭션 범위·자동 이송 로직 전수 검증 (8/16, 문제 없음 확정)**
- [x] **AGV 애니메이션 다중 사용자 동기화 버그 발견·수정·배포 (8/16, PR #46 → main Deploy #7)**

---

## 중단 기준 (Go / No-Go)

| 기한   | 조건               | 대응                |                                                                   |
| ---- | ---------------- | ----------------- | ----------------------------------------------------------------- |
| 8/12 | 로컬 compose 구동 실패 | AWS 포기, 로컬 데모     | ✅ 통과                                                              |
| 8/13 | OpenAI 전환 실패     | 챗봇 데모 제외          | ✅ 통과 (8/15 서버 반영까지 완료)                                            |
| 8/15 | 재고관리 미완          | 미완 상태로 동결         | ✅ 통과 (`batch_material_consumption` 등 재고 관련 기능 8/8 이전 이미 구현·연동 확인) |
| 8/16 | CD 미완            | 수동 배포 유지          | ✅ 통과 (수동 `workflow_dispatch` 방식, 오늘도 3회 정상 배포)                    |
| 8/17 | 서버 YOLO 추론 실패    | 8GB 증설 → 사전 계산 결과 | ⏳ 판단 대기 (`best.pt` 파일 미확보, 8/16에도 미착수 — 8/17 최우선 처리 필요)          |

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
 └─ 볼륨 : PCB 이미지(703장, 8/15 업로드 완료), best.pt (이미지에 굽지 않음, 8/16 미착수 → 8/17 최우선 처리)
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


































## 이게 뭔가요

PCB 검사가 끝난 배치를 SMD(부품 실장) 공정으로 넘기기 직전에, **지금 자재 재고로 이 배치를 진행할 수 있는지** 보여주고 실제로 재고를 차감하는 "재고 관리" 화면입니다. 좌측 메뉴의 빈 placeholder였던 "재고 관리" 탭이 이제 실제로 동작합니다.

- 검사완료된 배치들이 큐(대기열) 형태로 카드로 뜨고, 각 카드에 "진행 가능 / 진행 불가"가 바로 보입니다.
- 카드를 누르면 그 배치가 어떤 부품을 얼마나 필요로 하는지, 지금 재고로 충분한지(ENOUGH / ORDER_NEED / PENDING) 상세 창에서 볼 수 있습니다.
- "SMD로 인계" 버튼을 누르면 그 배치가 쓴 만큼 실제로 재고(`part.stock_qty`)가 줄어들고, 큐에서 빠집니다.

설계 배경이 궁금하면 [`docs/superpowers/specs/2026-08-08-inventory-management-design.md`](https://github.com/KDTbigdata8th/pcb-defects-mes/pull/docs/superpowers/specs/2026-08-08-inventory-management-design.md), 구현 순서가 궁금하면 [`docs/superpowers/plans/2026-08-08-inventory-management.md`](https://github.com/KDTbigdata8th/pcb-defects-mes/pull/docs/superpowers/plans/2026-08-08-inventory-management.md)를 보시면 됩니다 — 이 PR의 모든 커밋이 그 계획을 그대로 따라갑니다.

## 왜 이렇게 만들었나 (설계 포인트 3가지)

**1. 소요량은 저장하지 않고 매번 계산합니다.**  
`부품별 소요량 = 배치의 정상 PCB 수(normal_PCB_count) × BOM의 부품 수량(bom_item.quantity)`. 이 값을 캐싱하는 테이블을 따로 만들지 않고 조회할 때마다 계산합니다. BOM이 나중에 수정돼도 항상 최신값을 보여주기 위해서고, 이미 대시보드 KPI들([`dashboard/repository.py`](https://github.com/KDTbigdata8th/pcb-defects-mes/pull/backend/app/features/dashboard/repository.py))이 쓰는 것과 같은 패턴입니다.

**2. 재고 차감은 동시성 잠금(`FOR UPDATE`)으로 보호합니다.**  
같은 부품을 쓰는 배치 두 개를 두 사람이 거의 동시에 "인계"를 누르면, 잠금 없이는 둘 다 재고가 충분하다고 통과해서 재고가 음수로 내려갈 수 있습니다. `backend/app/features/inventory/service.py`의 `handoff_batch`는 배치와 관련 부품 행을 `SELECT ... FOR UPDATE`로 잠근 뒤 다시 검증하고, 통과했을 때만 같은 트랜잭션 안에서 차감·이력 기록·배치 상태 갱신을 한 번에 처리합니다. 이 패턴은 `backend/app/features/batch/`에서 이미 쓰던 방식을 그대로 따랐습니다.

**3. 인계 여부는 기존 `production_batch.status`와 별개 컬럼(`smd_handoff_at`)으로 관리합니다.**  
검사 흐름(`CREATED → ... → INSPECTED`)과 "SMD로 넘겼는지"는 서로 다른 개념이라, 기존 enum에 끼워 넣지 않고 nullable timestamp 컬럼을 새로 뒀습니다(`batched_pcb.line_entered_at`과 같은 기존 컨벤션).

## 화면/API 요약

|화면 요소|API|
|---|---|
|상단 4개 요약 타일(검사완료 배치 수, 진행가능/불가, 부족 자재 종류 수, 종합 판정)|`GET /api/inventory/queue`|
|좌측 큐 카드 목록|`GET /api/inventory/queue`|
|우측 부품 테이블 (카드 미선택 시 전체 재고, 선택 시 그 배치 소요량)|`GET /api/inventory/parts`, `GET /api/inventory/batches/{id}/requirements`|
|"SMD로 인계" 버튼|`POST /api/inventory/batches/{id}/handoff`|

## ⚠️ DB 스키마 변경 3건 — 리뷰 시 꼭 확인해 주세요

- `production_batch.smd_handoff_at` (신규 컬럼)
- `batch_material_consumption` (신규 테이블, 인계 시점마다 부품별 소비 이력 기록)
- `production_batch.normal_PCB_count` (신규 컬럼) — **이건 원래 검사 완료 로직을 담당하는 팀원이 추가하기로 했던 컬럼입니다.** 로컬에서 이 기능을 테스트하려고 팀원이 공유해준 DDL을 그대로 마이그레이션 파일로 만들어 적용했습니다(`backend/db/migrations/20260808_add_normal_pcb_count.sql`). `ADD COLUMN IF NOT EXISTS`라 팀원이 별도로 같은 컬럼을 추가해도 충돌은 안 나지만, 마이그레이션 파일이 두 벌 남는 상태가 될 수 있으니 **DB 담당자 확인 부탁드립니다.**

세 마이그레이션 다 `backend/db/migrations/`에 파일로 있고 `backend/db/schema.sql`에도 반영해 뒀습니다.

## 테스트

- 백엔드: `backend/tests/inventory/` 신규 5개 파일, unittest + mock 패턴 (기존 DB 연결 없이 repository 계층을 모킹). `python -m unittest discover -s backend/tests -t .` → 245개 전부 통과.
- 프론트: `frontend/test/inventoryStructure.test.js` 등 신규 6개 테스트. `npm test` → 404/407 통과 (나머지 3개는 이 PR과 무관한 기존 실패 — dashboard CSS 셀렉터 누락 등, `dev`에도 이미 있던 문제입니다).
- `npm run build` 정상.

## 알아두면 좋은 것 (병합을 막을 정도는 아니지만)

- **프론트 테스트가 실제 렌더링을 안 합니다.** `inventoryStructure.test.js`의 테스트들은 컴포넌트를 렌더링하는 게 아니라 소스 코드 문자열에 특정 패턴이 있는지 정규식으로 확인하는 방식입니다(이 리포의 기존 프론트 테스트 관행과 동일). 그래서 실제로 백엔드 응답 모양이 프론트 코드의 가정과 다른 버그(`PartsResponse`가 `{items: [...]}` 형태인데 배열로 착각한 버그)가 자동 테스트로는 안 잡히고 실제 브라우저에서만 발견됐습니다 — 이번 PR엔 고쳐서 반영했지만, 이런 종류의 버그를 자동으로 잡으려면 나중에 실제 렌더링 테스트가 필요합니다.
- 재고 관리 페이지 자체는 조회/인계만 하고, 소비 이력을 보여주는 화면은 없습니다(이력 테이블만 쌓아둠 — "리포트 생성" 메뉴가 나중에 만들어질 때 쓸 수 있게).
- 재고 부족(PENDING)인 배치는 인계 버튼이 비활성화됩니다. 강제로 인계하는 기능은 없습니다.