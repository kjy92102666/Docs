# CI/CD · AWS 배포 작업일지 (요약)

> **저장소** KDTbigdata8th/pcb-defects-mes · **발표** 2026-08-22 (토) · **최종 갱신** 2026-08-19
> 상세 기록은 `CICD_AWS_배포_작업일지_상세.docx` 참조

---

## 진행 현황

```
사전테스트  ████████████████████  8/9~10  완료
본작업     ███████████████████░ 8/11~22 9/10일차
```


| 날짜           | 목표                                                  | 상태                                                                                                                                                                                                                                                           |
| ------------ | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 8/09(일)      | 브랜치 병합 정리, CI 구축                                    | ✅                                                                                                                                                                                                                                                            |
| 8/10 (월)     | 배포 파이프라인 연습                                         | ✅                                                                                                                                                                                                                                                            |
| 8/11 (화)     | CI dev 반영, Dockerfile + compose                     | ✅ 초과 달성                                                                                                                                                                                                                                                      |
| 8/12 (수)     | 로컬 검증 보완 / 서버 준비 ⚠️판단시점                             | ✅ 초과 달성 (AWS 첫 배포까지)                                                                                                                                                                                                                                         |
| 8/13 (목)     | Lightsail 4GB 생성, 첫 배포                              | ✅                                                                                                                                                                                                                                                            |
| 8/14 (금)     | 배포 문제 해결 (버퍼)                                       | ✅                                                                                                                                                                                                                                                            |
| 8/15 (토)     | health check, CD 워크플로우 🔒스키마동결                      | ✅                                                                                                                                                                                                                                                            |
| **8/16 (일)** | **스키마 반영 재배포, YOLO fixture**                        | **✅ 데이터 시딩·공정상태 버그 수정 완료 (YOLO 모델 업로드는 미착수, 8/17로 이월)**                                                                                                                                                                                                      |
| **8/17 (월)** | **YOLO 배치 실측, 사양 조정**                               | **✅ 4GB 서버로 충분 확정 (8GB 증설 불필요), Precision 0.957/Recall 0.915/mAP50 0.947**                                                                                                                                                                                   |
| **8/18 (화)** | **성능 튜닝, DB 백업 cron, CI MariaDB, Migration 안전배포체계** | **🔄 CI MariaDB·이미지 표시 버그 수정 dev/main/AWS 배포 완료. DB Migration Phase 1~3(schema_migrations+runner+bootstrap+predeploy backup+deploy.yml 연결) 구현·로컬 검증 완료, AWS 실배포는 8/19로 이월. 대시보드 로고 반영 착수**                                                                   |
| **8/19 (수)** | **Migration 안전배포 AWS 실배포, 전체 리허설 착수**               | **🔄 Migration+로고 배포 완료(PR #53/#54, AWS 검증까지). 공정상태 AGV 즉시점유·조기점유 버그 2건 발견·수정(타임존+상태판정), 검사화면 이미지 로딩 과다요청 완화 — 둘 다 dev/main 병합·2차 AWS 배포 완료(43357e2), SSH/브라우저 최종검증은 8/20으로 이월. 팀원 YOLO 최종모델(22MB) AWS 반영 확인(부하 재테스트 이월). 발표전 테스트 계획 TC-01 착수 중 버그 대응으로 중단** |
| 8/20 (목)     | 발표 자료 🔒코드동결                                        | ⬜                                                                                                                                                                                                                                                            |
| 8/21 (금)     | 예비일                                                 | ⬜                                                                                                                                                                                                                                                            |
| 8/22 (토)     | 🎯 발표                                               | ⬜                                                                                                                                                                                                                                                            |

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
| YOLO 모델 품질 (8/17 확인) | `best.pt`, YOLOv8n, 160epoch(best epoch 121), **Precision 0.957 / Recall 0.915 / mAP@50 0.947 / mAP@50-95 0.508** |
| 서버 처리 성능 (8/17 실측) | PCB 1개당 0.22~0.28초로 일정(배치 크기·동시 실행 개수 무관), 단일 500개 배치·8라인 동시(160개) 모두 메모리 13% 내외(450~522MiB), CPU는 세마포어(`YOLO_MAX_CONCURRENCY=1`)로 안정 관리 |
| Backend 테스트 (8/18 갱신) | **420** passed, 2 skipped (migration runner 18건 + bootstrap 3건 신규 추가) |
| Frontend 테스트 (8/18 갱신) | **527** passed (이미지 URL production 회귀 테스트 신규 추가) |
| Migration/Bootstrap 통합 테스트 (8/18 신규) | **21** passed (plan 100% read-only 증명, APPLIED/RECONCILED/checksum mismatch/drift/DDL성공+history누락 복구 등 실제 MariaDB 기반 검증) |
| YOLO 최종 모델 (8/19 팀원 반영) | `best.pt` 22MB (기존 6.29MB 대비 3배 이상 증가, 팀원이 별도 정밀 튜닝) — AWS 배포·헬스체크 확인, 부하 재테스트는 8/17 기준(구모델)이라 미실시(8/20 이월) |
| Backend 테스트 (8/19 갱신, 브랜치별) | **436** passed, 8 skipped (AGV/타임존 수정 브랜치, YOLO 최종모델 병합 후 재검증) |
| Frontend 테스트 (8/19 갱신, 브랜치별) | **538** passed(이미지로딩 브랜치) / **537** passed(AGV 브랜치) — 두 브랜치 병합 후 통합 재검증은 8/20 이월 |
| 검사화면 이미지 로딩 윈도우 (8/19 개선) | 328개 중 동시 원본 이미지 요청을 최대 **9개**로 제한(뒤2+현재+앞6), 328개 순차 진행 모션은 그대로 유지 |

> skip 2건은 YOLO 스모크 테스트(fixture 미비, 8/17 확인 결과 기능 자체엔 문제 없음으로 확정). `best.pt` 모델은 8/17 확인 결과 이미 Git 추적 중이며 별도 업로드 불필요했음(아래 8/17 일일 기록 참조). 8/19 팀원이 최종 튜닝 모델로 교체 반영.

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

### 8/17 (월) — YOLO 배치 실측 ✅

**범위**: YOLO 배치 실측 — 4GB 서버 한계 확인, 필요 시 8GB 증설 판단 (8/15→8/16 두 번 밀려 8/17로 이월된 항목)

**한 일**

1. **YOLO 모델 파일 소재 확인** — 결과: 이미 서버에 존재, 별도 업로드 작업 불필요
   - `backend/models/best.pt`가 2026-08-05 커밋(`b2037f9`)으로 이미 Git에 추적되어 있었음
   - `docker-compose.yml` 볼륨 마운트(`./backend/models:/app/backend/models:ro`) 덕분에 서버가 `git pull`할 때마다 자동 최신화 — 8/12 첫 배포 시점에 이미 반영된 것으로 추정
   - 서버 확인: `best.pt` 6,289,066 bytes, 로컬 저장소 파일과 크기 일치
   - 모델 품질: 클래스 6종(missing_hole, mouse_bite, open_circuit, short, spur, spurious_copper)이 프로젝트 결함 유형과 정확히 일치(COCO 기본 가중치 아님), 160epoch 학습(best epoch 121, 학습완료 2026-07-29 UTC), **Precision 0.957 / Recall 0.915 / mAP@50 0.947 / mAP@50-95 0.508**
   - 부수 확인: `INSPECTION_MODE` 환경변수는 코드 어디서도 읽지 않는 죽은 설정. 시뮬레이션(`/api/batches/{id}/inspect`)과 실제 YOLO(`/api/inspection/batches/{id}/start`)는 환경변수 분기가 아니라 애초에 분리된 별개 엔드포인트

2. **순차 부하 테스트** (신규 `production_plan` 등록 → 배치생성→라인이송→실제 YOLO 검사)

   | 배치 | 개수 | 소요시간 | 개당 시간 | 메모리 | CPU |
   |---|---|---|---|---|---|
   | batch 6 | 50 | 14초 | 0.28초 | 450MiB | 0.13~0.15%(관찰 타이밍 놓침 추정) |
   | batch 7 | 100 | 26초 | 0.26초 | 483MiB | 98~99% |
   | batch 8 | 200 | 53초 | 0.265초 | 450~514MiB | 97~99% |
   | batch 9 | 360 | 79초 | 0.219초 | 476MiB(거의 고정) | 98~99%, 순간 108% |
   | **batch 10** | **500** | **123초** | **0.246초** | **511MiB** | **97~108%** |

   - 메모리 전 구간 450~514MiB(전체 한도 3.744GiB 대비 13% 내외), 배치 커져도 누적 없음(메모리 누수 없음)
   - `YOLO_MAX_CONCURRENCY=1`이 프로세스 전역 세마포어(`asyncio.Semaphore`)로 CPU 폭주를 막는 핵심 장치임을 코드로 확인. 서버는 2 vCPU

3. **동시성 부하 테스트** (여러 라인 동시 검사, `Start-Job`으로 병렬 발사, 배치당 20개로 통일)

   | 동시 배치 수 | 총 이미지 | 소요시간 | 개당 시간 | CPU 최고치 | 메모리 범위 |
   |---|---|---|---|---|---|
   | 2개 | 40 | 10초 | 0.25초 | 98~99% | 465~508MiB |
   | 4개 | 80 | 19~20초 | 0.24초 | 100.3% | 499~519MiB |
   | 6개 | 120 | 29~30초 | 0.25초 | 100.35% | 498~522MiB |
   | **8개(전체 라인)** | **160** | **40~41초** | **0.25초** | **167.94%** | **501~510MiB** |

   - 개당 처리 속도 0.24~0.25초로 완전히 일정(순차 테스트와도 같은 범위), 메모리도 순차 테스트와 사실상 동일 범위
   - CPU는 8개 전체 동시 실행에서 처음 167.94%(2코어 대부분 사용) 관측되나 순간 피크 후 즉시 안정 — 위험 신호 아닌 "2코어 최대 활용" 수준
   - 전 구간 에러·실패·중복 처리 없이 정상 완료

4. **종합 결론**: **4GB 서버로 충분, 8GB 증설 불필요.** 처리 속도가 배치 크기·동시 실행 개수와 무관하게 PCB 1개당 약 0.22~0.28초로 일정 → 8라인이 각 500개씩 동시에 돌아도(4,000장) 대략 17분 내 전량 처리 가능하다는 예측 근거 확보(단, 이 최대 규모 조합 자체는 미실측)

**아직 실측하지 않은 것 (참고, 필요시 추가 검증)**
- "8개 라인 각각 대용량(500개)으로 동시 실행"(순차·동시성 두 축을 합친 최악의 조합)은 미시도, 다만 각 축 특성상 문제없을 것으로 예측
- "당일 계획 배치만 검사 화면에 노출"은 팀의 의도된 UI 설계로 확인됨(버그 아님) — 발표 후 고도화 항목으로 이월
- `INSPECTION_MODE` 죽은 환경변수 — `.env.example` 정리 여부는 발표 후 결정

**막힌 지점**: 없음(계획대로 순조롭게 진행, 어제까지 이월됐던 항목을 당일 완전히 해소)

> **어제 작업일지의 Go/No-Go 기준(8/17: 서버 YOLO 추론 실패 시 8GB 증설) → 통과로 최종 확정.**

---

### 8/18 (화) — CI MariaDB, DB Migration 안전배포체계(Phase 1~3), 이미지 표시 버그 수정 🔄

**범위**: 성능 튜닝(급하지 않아 보류), CI MariaDB 정식 반영, DB 볼륨 재사용 시 migration 자동 미실행 문제 정식 해결(8/16 발견 이월 항목), 그 외 실사용 중 발견한 이미지 표시 버그·대시보드 로고 반영

**한 일**

1. **Dashboard 카드 CSS 마무리 확인**
   - 대시보드 우측 PROCESS&AGV/INVENTORY 패널이 좌측 카드와 동일한 컨테이너 스타일(`padding:12px; border:1px solid rgba(48,54,61,0.95); border-radius:8px; background:var(--dashboard-panel-soft)`) 사용하는지 실제 브라우저로 재확인. 이 과정에서 실습실 로컬 DB의 migration 누락(`normal_PCB_count`, `smd_handoff_at`, `batch_material_consumption`)도 함께 발견·적용
   - 커밋 `8091b6c` 이미 dev/main 포함 확인 — 별도 PR 불필요

2. **Git 정리 + AWS 최신 main 배포**
   - AWS 서버에 SCP로 직접 올렸던 `scripts/bulk_resolve_reviews.py`가 `git pull --ff-only origin main`을 막던 문제 → `~/pcb-mes-manual-backup/`으로 이동 후 `Re-run failed jobs`로 배포 성공
   - 최종 확인: AWS HEAD `459f37f`, `git status --short` clean, backend/mariadb/nginx 전부 `Up`, `/health/ready` 200 OK — GitHub 최신 main과 AWS 실제 배포본 일치 확인

3. **CI MariaDB 서비스 컨테이너 통합** (브랜치 `fix/ci-mariadb-integration`, 커밋 `76fd431`+`d6cbab6`)
   - CI가 `mariadb:lts` → `schema.sql` 적용 → 실제 DB 기반 backend pytest 실행하는 구조로 전환
   - 기존 `@pytest.mark.skipif(os.getenv("CI")=="true")`로 건너뛰던 Analysis Export DB 테스트가 실제 MariaDB에서 통과(변경 전 398 passed 3 skipped → 변경 후 399 passed 2 skipped, 남은 skip 2건은 YOLO smoke fixture 부재로 무관)
   - PR #50 → dev 병합(`ea3cf61`) → 병합 완료 후 브랜치 삭제

4. **AWS 운영 DB 읽기 전용 실태조사 (Migration 전수조사 후속)**
   - 대상: 17개 테이블, `production_batch` lifecycle 컬럼, `post_inspection_buffer_slot` 슬롯 1~16, `agv.loaded_count` comment 등을 `SELECT`/`SHOW`/`information_schema`만으로 조사(ALTER/INSERT/UPDATE/DELETE/DROP 금지)
   - 결과: 기존 legacy migration 11개 중 유효 forward 9개는 전부 AWS에 이미 반영(`APPLIED_EQUIVALENT`), destructive/rollback 2개는 `DO_NOT_APPLY`로 분류, **AWS에 지금 당장 적용해야 할 미적용 forward migration은 0개** — baseline(2026-08-18 AWS 상태 = Legacy Migration Baseline)으로 확정
   - `post_inspection_buffer_slot` 슬롯 1~16은 AWS에 이미 존재하나, 최신 `schema.sql`만으로 신규 DB를 만들면 이 초기 데이터가 빠진다는 점도 함께 확인(→ 이후 bootstrap 설계로 해결)

5. **DB Migration 안전 배포 체계 — Phase 1: schema_migrations + Python runner** (브랜치 `fix/db-migration-runner`, 커밋 `8728256`)
   - `backend/app/db_migrations/`: `plan`/`apply`/`verify` CLI, migration 파일 탐색·정렬, SQL+verifier 합산 SHA-256 checksum, `GET_LOCK`/`RELEASE_LOCK`, `APPLIED`/`RECONCILED` 상태 처리
   - MariaDB DDL의 implicit commit 특성상 "DDL 성공 후 history 기록 전 장애" 상황을 verify 기반 `RECONCILED`로 복구하는 상태 기계 설계·구현(실제 MariaDB trigger로 history INSERT 강제 실패시켜 재현·검증)
   - 기존 legacy `migrations/*.sql` 11개는 절대 자동 실행하지 않고 별도 보존, `deploy_migrations/`를 신규 forward-only 전용 디렉터리로 분리
   - `plan`이 100% read-only임을 CLI 레벨(`before/after schema_migrations: 0`)과 통합 테스트 레벨 이중으로 증명
   - 실제 MariaDB 기준 6개 핵심 시나리오(APPLIED, RECONCILED, checksum mismatch, history있음+verify실패 drift, SQL실패시 history 미기록, DDL성공+history누락→RECONCILED 복구) 전부 검증, 임시 DB에 `schema.sql` 전체 적용해 clean schema도 검증
   - 통합 테스트 18 passed, 전체 backend 417 passed 2 skipped

6. **DB Migration 안전 배포 체계 — Phase 2: Buffer Bootstrap** (동일 브랜치, 커밋 `b6a082c`, 개인 노트북 환경으로 전환 후 진행)
   - `backend/db/bootstrap/post_inspection_buffer_slots.sql`+`.verify.sql` 신규: 슬롯 1~16 누락분만 생성, 기존 운영 상태(`status`/`batch_id`/`transfer_id`/`reserved_at`/`occupied_at`)는 절대 덮어쓰지 않음(`ON DUPLICATE KEY UPDATE slot_no = VALUES(slot_no)`만 수행)
   - 범위 밖(`slot_no<1` 또는 `>16`) 데이터는 자동 삭제·수정하지 않고 `NOT NULL` 제약으로 전체 문장을 실패시켜 보호
   - CI 순서를 `schema.sql → bootstrap/verify → migration runner integration tests → backend tests`로 확장
   - 노트북 로컬 `pcb_mes`(실행 전 0행) 대상 실제 bootstrap 검증: 최초 실행 후 16행(고유 슬롯 1~16), 재실행해도 16행 유지·운영 필드 변화 없음
   - 통합 테스트 21 passed, 전체 backend 420 passed 2 skipped

7. **PCB 이미지 표시 버그 발견·조사·수정** (브랜치 `fix/inspection-image-display`, 커밋 `2139056`)
   - 증상: 검사 화면 Batch 관리에서 카드 리스트는 정상이나 PCB 썸네일은 로컬·AWS 클라우드 양쪽 모두 "PCB 이미지 없음" placeholder만 표시
   - devtools Network 탭 확인 결과 실제 이미지 요청 자체가 발생하지 않음(JS 모듈 로딩 요청만 관찰) — 서버/이미지 파일 자체는 정상(`/api/pcb/{id}/image` 단건 조회는 200)
   - **원인 확정**: `frontend/src/domains/inspection/imageUrl.js`의 `resolveApiImageUrl()`이 production `API_BASE`(커밋 `a026064`에서 same-origin 최적화로 빈 문자열로 변경됨)일 때, `new URL(상대경로, apiBase)` 호출이 "Invalid base URL" 예외를 던지고 catch에서 무조건 `null` 반환 → `<img>` 자체가 렌더되지 않아 요청조차 안 나감
   - 수정: `API_BASE`가 빈 문자열이면 상대경로를 그대로 반환하도록 변경 + production same-origin 회귀 테스트 추가
   - 동일 함수를 쓰는 다른 5개 호출 지점(BatchManagementPanel, RealtimeInspectionWorkspace, InspectionReviewWorkspace, InspectionImageZoomModal, 분석 heatmap mapper)도 중앙 수정으로 함께 해결
   - 검증: 관련 이미지 테스트 31 passed, 전체 프론트 527 passed, Production Vite build 성공, 전체 backend 399 passed 2 skipped
   - PR #51 → dev 병합(`e4d152a`) → PR #52(dev→main) 병합(`d8c64c2`) → AWS `Run workflow` 배포 → health check 통과 → **AWS 실제 화면에서 이미지 정상 표시 최종 확인**(Network 탭 `image` 요청 전부 200 jpeg)
   - 부가 확인: 개인 노트북 로컬 환경에선 이미지 서빙 경로(Google Drive bind mount)가 비어 있어 별도 404가 발생했으나, 이는 로컬 개발 환경 데이터 부재 문제로 코드와 무관함을 확인(진짜 703장 데이터셋은 노트북 내 다른 경로에 있으나 현재 compose 설정이 그쪽을 참조하지 않음 — 개인 로컬 이슈로 조치는 보류)

8. **DB Migration 안전 배포 체계 — Phase 3: Predeploy Backup + deploy.yml 연결** (동일 브랜치, 커밋 `13d8692`)
   - **치명적 버그 발견①**: `backend/app/db_migrations/runner.py`가 절대 임포트(`backend.app...`)를 사용 — CI는 `PYTHONPATH=repo root`라 통과하지만, 실제 프로덕션 Docker 이미지는 `backend/app`만 COPY하므로 컨테이너 안엔 `backend` 패키지 자체가 없어 `docker compose run` 실행 시 `ModuleNotFoundError`로 100% 실패하는 상태였음 → 상대 임포트로 수정
   - **치명적 버그 발견②(더 위험)**: `Dockerfile`이 `backend/db/deploy_migrations` 디렉터리를 이미지에 COPY하지 않아, 컨테이너 안에서 forward migration이 항상 "0개 발견"으로 **에러 없이 조용히** 처리되는 상태였음 — 실패하는 버그보다 위험한 무증상 사고 → `Dockerfile`에 COPY 라인 추가
   - `backend/scripts/predeploy_backup.sh` 신규: `docker compose exec -T mariadb`로 컨테이너에 이미 주입된 환경변수 재사용, `.tmp`→exit code/0바이트 확인→rename, `~/backups/predeploy/`(기존 daily backup과 경로·이름 미충돌), 최근 5개만 보존. 강제 실패 테스트에서 `.tmp` 미잔존·rotation 미실행·기존 백업 보존 확인
   - `deploy.yml` 변경 설계: `git pull → frontend build → docker compose build backend(컨테이너 유지) → migration plan(read-only) → predeploy backup → migration apply → 성공시만 docker compose up -d → health check`, `set -e`로 각 단계 실패 시 이후 단계 중단
   - 실패 경로를 실제 CLI로 3가지 시나리오(잘못된 DB 비밀번호, checksum 불일치, apply만 실패) 재현 — 전부 exit 1, 전체 스크립트 시뮬레이션에서 `set -e`가 실제로 `docker compose up -d`로 안 넘어감을 MARKER 로그 대조군 비교로 확인
   - **AWS 실배포(`Run workflow`)와 PR/merge는 미실행 — 8/19로 이월** (첫 실전 적용은 컨디션 좋은 시간대에 진행하기로 결정)

9. **최신 dev(이미지 hotfix 포함) 기준으로 Migration 브랜치 병합**
   - `fix/db-migration-runner`가 `dev`보다 2커밋 뒤처진 상태 확인(이미지 hotfix는 별도 독립 hotfix로 먼저 배포했으므로) → `git merge origin/dev`로 병합(충돌 없음, merge commit `7dc332a`, 초기 커밋 메시지 오타는 amend로 정정)
   - 병합 후 재검증: migration/bootstrap 통합 테스트 21 passed, 전체 backend 420 passed 2 skipped — 이미지 hotfix와 migration 작업이 실제로도 완전히 독립적이었음을 재확인

10. **브랜치 정리**: 병합 완료된 `fix/inspection-image-display`, `fix/ci-mariadb-integration` 로컬/원격 삭제

11. **(착수, 진행 중) 대시보드 좌측 상단 로고 이미지 반영** — 신규 로고 이미지를 대시보드에 반영하는 작업 착수. Migration 작업과 무관한 독립 프론트 변경이므로 최신 dev 기준 별도 브랜치로 분리해 진행 예정

**막힌 지점**

| 문제 | 원인 | 해결 |
|---|---|---|
| CI는 통과하지만 실제 배포 컨테이너에서 `ModuleNotFoundError` | `runner.py`의 절대 임포트가 CI의 `PYTHONPATH` 환경에서만 성립, 실제 Docker 이미지엔 `backend` 패키지 자체가 없음 | 상대 임포트로 변경, 실제 컨테이너에서 재검증 |
| Forward migration이 항상 "0개 발견"으로 조용히 처리(에러 없음) | `Dockerfile`이 `backend/db/deploy_migrations`를 이미지에 COPY하지 않음 | `Dockerfile`에 COPY 라인 추가, 실제 checksum 변조까지 컨테이너에서 재현·검증 |
| production에서 PCB 이미지 요청 자체가 발생 안 함 | `API_BASE` 빈 문자열일 때 `new URL(상대경로, base)`가 예외를 던지고 catch에서 무조건 `null` 반환 | `API_BASE`가 빈 문자열이면 상대경로를 그대로 반환하도록 수정 |
| 노트북 로컬에서 이미지 여전히 404 | Google Drive bind mount 경로가 비어있음(로컬 개발 데이터 부재, 코드 문제 아님) | AWS에서 최종 검증으로 대체, 로컬 데이터 정비는 보류 |
| 노트북 Docker Desktop 데몬 미기동 | 로그인 직후 Docker Desktop 앱 자체가 실행 전 상태 | 수동 실행 후 정상 확인 |
| git merge 중 커밋 메시지에 오타(`devt`) 유입 | vim 편집 중 실수 | `git commit --amend`로 정정 |
| 작업 도구를 Codex(주간 사용량 21%까지 소진)에서 Claude로 전환 | Phase 3까지 진행하며 하루 동안 사용량 상당량 소진 | 브랜치 상태·커밋 SHA·완료된 Phase를 명시적으로 요약해 인계, 무거운 작업(Phase 3 AWS 실배포)은 다음 세션으로 분리 |

**다음 우선순위 (8/19)**

1. DB Migration 안전 배포 체계 — AWS 실제 배포(`Run workflow`)와 최종 검증 (predeploy backup + migration gate 첫 실전 적용, 컨디션 좋은 시간대에 신중히 진행)
2. 대시보드 로고 이미지 반영 완료
3. Default Branch를 `main`으로 원복 (8/13부터 이월, `datamon78` 계정 필요)
4. 여러 브라우저 동시 접속 시연 리허설 (AGV 애니메이션·공정상태·계획등록 동기화 확인 — 8/16 AGV 동기화 버그 재발 여부 포함)
5. (여유 시간) `FastAPI on_event` deprecation 등 사소한 정리

---

### 8/19 (수) — Migration AWS 실배포, 공정상태 AGV 버그 2건 수정, 검사화면 이미지 로딩 개선 🔄

**범위**: 8/18에서 이월된 DB Migration 안전배포 체계 AWS 실배포, 대시보드 로고 반영 완료, 발표전 최종 테스트 계획 착수 — 테스트 도중 실사용 중 발견한 공정상태 AGV 버그 2건과 검사화면 이미지 로딩 문제를 우선 처리

**한 일**

1. **Migration Phase 1~3 + 로고 반영 — dev/main 병합·AWS 1차 배포** (PR #53 `fix/db-migration-runner`→dev, PR #54 dev→main)
   - 어제 로고 커밋(`9322bde`, `5564e10`)이 별도 브랜치가 아니라 `fix/db-migration-runner`에 함께 있던 것을 확인 — cherry-pick으로 분리하는 대신, migration 작업과 로고가 서로 다른 파일이라 리스크가 낮다고 판단해 그대로 함께 진행하기로 결정
   - 최종 통합 검증 중 실제 실패 1건 발견: `appStructure.test.js`가 로고 작업 이전 "M" 텍스트 placeholder를 여전히 기대하고 있어 실패 — 실제 이미지 마크업(`<img src={kdhLogo}> alt="K-Defects-Hunters"`) 검증으로 테스트 갱신(`37496fa`)
   - PR #53 → dev 병합(`de731a3`) → PR #54(dev→main) 병합(`9b1f0f5`) → AWS `Run workflow` 배포(1차) → health check 통과(502 1회 재시도 후 정상, 설계된 재시도 로직 그대로 작동)
   - SSH 검증: HEAD `9b1f0f5` 일치, `git status` clean, `schema_migrations` 테이블 정상 생성(0건, 정상), `~/backups/predeploy/predeploy_20260819_014821_9b1f0f5.sql`(445KB) 정상 생성·기존 daily backup과 미충돌
   - 브라우저 검증: 로고 정상 표시, Batch 관리 화면 PCB 이미지 정상 표시(회귀 없음) — **Migration 안전배포체계 첫 실전 적용 성공**

2. **이미지 렌더링 지연 발견·원인 분석** (조치는 보류, 기록만)
   - 배치 목록에서 이미지가 뜨긴 하나 로딩이 느림을 발견 — devtools Timing 확인 결과 실제 서버 응답은 0.48ms인데 `Queueing` 단계에서만 9.80초 소요
   - 원인: 브라우저의 origin당 동시 연결 6개 제한(HTTP/1.1) + 이미지가 원본 크기 그대로 서빙되는 것이 겹쳐 대기열이 김
   - 검사담당자의 YOLO 최종모델 CD가 아직 진행 중이었으므로, **YOLO 최종모델 반영 시점에 함께 재검증하기로 결정** (검출 결과 변화로 NG 이미지 렌더링 패턴이 달라질 수 있어 지금 손대는 것보다 합리적 판단)
   - 로고 사이즈도 68px → 34px로 재조정하기로 결정, 이 역시 YOLO 반영 시점에 함께 배포 예정 (이 시점 기준 **로컬 변경만 존재, 아직 커밋·배포 안 됨**)

3. **발표전 최종 테스트 계획 착수** (`PCB_MES_발표전_최종테스트계획.xlsx` 기준)
   - TC-01(P0, 계획 정상 등록), TC-02(P1, 입력 검증·취소 제약) 순서로 진행 가이드
   - 테스트 중 신규 배치가 "대기" 상태에서 "검사 시작" 버튼이 비활성화되는 현상 발견 → 조사 결과, 공정상태 화면에서 **라인 이송(AGV가 실제로 배치를 라인까지 옮기는 과정)을 완료해야 검사 시작이 활성화되는 의도된 설계**로 확인(TC-05 `inspection_ready` 전제조건과 일치)
   - 라인 이송 확인 과정에서 아래 4번 버그 2건을 발견해 우선 처리 — **정식 TC 통과/실패 기록은 중단, 8/20 재개 예정**

4. **공정상태(AGV/LINE) 버그 2건 발견·근본원인 규명·수정** (브랜치 `fix/process-status-agv-occupancy-sync`, `git worktree`로 물리적 격리해 병행 작업)
   - **증상 1**: 빈 라인에 배치를 처음 등록하면 AGV 이동 애니메이션 없이 곧바로 라인이 "가동 중"으로 점유 표시됨
   - **증상 2**: 같은 라인에 배치를 재등록하면 AGV는 실제로 이동하지만, WAITING ZONE 점유 표시가 AGV가 물리적으로 도착하기 전에 먼저 켜짐
   - **증상 1 근본원인(1차 시도의 결함까지 재귀적으로 발견)**: `process_status/service.py`가 `transfer_started_at`(naive datetime)을 무조건 UTC로 라벨링했는데, MariaDB 세션 타임존이 SYSTEM(호스트 OS 타임존)을 따르는 구조라 로컬(Windows/KST) 환경에서는 `NOW()`가 실제로 KST 값을 반환하고 있었음 — 재접속 회귀 테스트(2.2초 뒤 재접속 시 예상 2.8초 vs 실제 5.66초, 거의 전체 구간 재이동)로 이 불일치를 실측으로 잡아냄. 잘못 라벨링된 타임스탬프를 `Date.parse()`가 9시간 뒤(미래)로 해석 → `elapsedSeconds` 음수 → `Math.max(0, ...)`가 0으로 clamp → **8/16에 고친 "경과시간 기반 위치 재계산"이 재접속 케이스에서 조용히 무력화**되고 있었음(최초 생성 시엔 경과시간이 원래 0에 가까워 티가 안 남)
   - **근본 수정**: `database.py`의 모든 DB 연결에 `SET time_zone = '+00:00'` 고정 — 호스트 OS 타임존과 무관하게 `NOW()`가 항상 진짜 UTC를 반환하도록 함
   - **증상 2 근본원인**: `FactoryMap.jsx`의 `WaitingSlot`이 점유 여부를 `slot.batchId` 존재 여부만으로 판정 — 실제로는 큐 엔트리에 `staging`(이동 중)/`waiting`(도착 완료) 구분이 이미 존재하는데 뷰가 이를 무시. `assignWaitingJobs()`가 AGV 배정 즉시(출발도 전에) `batchId`를 세팅하는 게 원인
   - **수정**: 점유 판정을 `entry.state === 'waiting'` 기준으로 변경
   - **검증**: Playwright로 실제 uvicorn+vite 서버 띄워 브라우저 재현 — 빈 라인 등록 시 AGV 이동시간(~5초) 동안 회색 유지 후 점유 전환 확인, WAITING ZONE도 도착 후에만 점유 전환 확인. 재접속 시나리오(3초 지점 재접속 → 총 10.23초로 정상 이어받음)와 대조군 비교로 8/16 fix 회귀 없음 확인(오히려 이 검증 과정에서 8/16 fix가 무력화되고 있던 걸 발견해 잡아냄)
   - 커밋 `7eda7cc` → 이후 최신 dev(YOLO 최종모델 포함) 병합(`b3cb15f`, 커밋 메시지 오타 `Mergt` — 기능 영향 없음) → 재검증(436 passed/8 skipped, 537 passed) → push
   - 검증 중 로컬(Windows 네이티브 MySQL, 포트 3306)에 생성된 테스트 데이터(`plan_id 18~23`, `batch_id 15~21`)는 AWS와 물리적으로 분리된 별도 프로세스라 AWS 운영 DB와 무관함을 확인

5. **오래된 미병합 브랜치 검토** (`fix/websocket-invalid-url-empty-api-base`, 2026-08-13 생성)
   - `git fetch` 중 발견 — 브랜치명이 어제 고친 이미지 URL 버그와 같은 계열(WebSocket URL 생성 시 `API_BASE` 빈 문자열일 때 `Invalid URL` 예외)로 보여 즉시 확인
   - `git diff origin/dev` 결과 대량 삭제처럼 보였으나(migration 체계, analysis 기능, CI MariaDB 등), 이는 **이 브랜치가 그 기능들이 생기기 전인 8/13 시점에서 분기됐기 때문**(그대로 merge 시 최근 작업 대량 삭제 위험 — merge 보류 판단)
   - 실제 유효 수정(`7bf6e8f`, `inspectionProgressWebSocket.js`+`processStatusWebSocket.js` 각 1줄)만 확인한 결과, **이미 오래전 PR #35로 dev/main에 정식 반영되어 있었음**을 확인 — cherry-pick 시도 결과 빈 diff로 검증됨, 추가 조치 불필요. 임시 확인용 로컬 브랜치 삭제

6. **검사화면 실시간 이미지 로딩 과다요청 문제 발견·완화** (브랜치 `fix/inspection-image-loading-investigation`, 별도 `git worktree`로 병행)
   - 진행 중인 검사(328개 배치)에서 대부분 PCB 썸네일이 "로딩 중"에 멈추고, 검사 완료 후 검사·결과 화면으로 이동해도 NG 목록·Review Status가 비어있는 증상 발견 — devtools에서 592개 요청·400MB+ 전송, 일부 요청이 30분 넘게 `Stalled` 확인(NG 목록 관련은 추후 해당 배치가 실제 NG 0건이었음이 확인되어 정상 동작으로 판명, 로딩 지연만 실제 문제)
   - 조사 결과: 캐러셀이 애니메이션을 위해 화면 표시 개수보다 10개 더 마운트(데스크톱 기준 5개 표시 → 15개 마운트·이미지 요청), 검사 진행에 따라 버퍼 가장자리마다 새 원본 이미지 요청 발생, 명시적 요청 취소(`AbortController`) 없이 검사 완료 시점까지 결과적으로 328개 전부 요청됨. "로딩 중" placeholder는 순수 시각효과일 뿐 실제로는 숨겨진 `<img>`가 이미 요청을 쏘고 있었음(`loading="lazy"` 없음). 이미지 API는 리사이즈 옵션 없이 원본을 그대로 반환
   - **개선 방향 결정**: 캐러셀 마운트·애니메이션 로직(11~15개)은 그대로 유지하되, 실제 원본 이미지 요청 범위만 현재 위치 기준 뒤 2개+현재+앞 4~6개(최대 9개)로 제한 — 328개 순차 진행 모션은 유지하면서 동시 네트워크 부하만 감소. `AbortController` 명시적 취소, 썸네일 API는 이번 단계에서 보류(2단계 과제로 이월)
   - 구현(`abb7baf`): 328개 시뮬레이션으로 요청 범위 검증(첫 위치 7 / 중간 최대 9 / 마지막 3, 전체 328개 순차 커버 확인), `npm run build` 성공, 관련 프론트 테스트 538 passed
   - **팀원의 YOLO 최종모델 반영분(캐러셀 지오메트리 변경 포함, `carouselSlideGeometry.js` 신규)과 겹치는지 확인** — 공통 조상(`009abe9`) 이후 `RealtimeInspectionWorkspace.jsx`는 dev에서 추가 변경 없음을 확인해 충돌 없음 확정, 최신 dev 병합(`ec47c6e`) 후 재검증(프론트 538 passed, 백엔드 1건 error는 이 worktree에 로컬 DB 인증정보(`.env`)가 없어 발생한 환경 문제로 코드와 무관함을 확인)
   - **실제 브라우저 검증**: 로컬 backend(`8001`)·frontend preview(`4174`) 별도 구성, health/live·ready 200, WebSocket 연결 성공까지 확인했으나 이 노트북에 원본 이미지 파일 자체가 없어(8/18에 확인된 문제) 요청이 404로 빠르게 실패해 원래 재현하려던 "느린 로딩" 자체는 로컬에서 재현 불가 — 코드 검증(테스트·diff 안전성)까지만 로컬에서 완료하고, **실제 육안 검증은 AWS 배포 후로 이월** 판단(8/18 이미지 URL 버그 때와 동일한 패턴)

7. **AGV/타임존 브랜치 + 이미지로딩 브랜치 — dev/main 병합 및 AWS 2차 배포** (PR #59, PR #60 → dev, PR #61 dev→main)
   - PR #59(`fix/process-status-agv-occupancy-sync`) → dev 병합(`8eeb5c9`)
   - PR #60(`fix/inspection-image-loading-investigation`) → dev 병합(`217d191`)
   - PR #61(dev→main) 병합(`43357e2`) → AWS `Run workflow` 배포(2차) → health check 통과(502 1회 재시도 후 정상)
   - SSH 1차 확인: HEAD `43357e2` 일치, `git status` clean, backend만 재생성(mariadb/nginx 무중단 유지)
   - **노트북이 PR #61 CI 대기 중 예기치 않게 종료**되어 대화 세션 일부 컨텍스트 유실 — 실제 작업(merge)은 GitHub에 이미 반영되어 있어 데이터 손실은 없었음, `git log`/`gh` 재확인으로 실제 상태 검증 후 이어감
   - **`schema_migrations`/`predeploy backup` 파일 확인과 실제 브라우저 회귀 검증(로고·이미지·AGV·이미지로딩)은 8/20으로 이월** (컨디션 고려한 명시적 결정)

8. **(팀원 작업, 병행 확인만) YOLO 최종 모델 AWS 반영**
   - `feature/pcb-inspection` → dev(PR #55, `009abe9`) → main(PR #56, `545aa89`) 경로로 검사화면 담당자가 최종 모델 배포
   - `backend/models/best.pt` 22MB로 교체 확인(기존 6.29MB 대비 3배 이상), `docker compose ps`/`health/ready` 정상 확인
   - **8/17 부하 실측(0.22~0.28초/PCB, 4GB 충분)은 이전 모델 기준 — 최종 모델 부하 재테스트는 8/20 이월**

9. **발표자료(PPT) 작업 병행** (Claude Design)
   - "기술 스택" 슬라이드: 실제 사용 스택(React/Vite, FastAPI, YOLOv8n, MariaDB, Docker/nginx/Lightsail/GitHub Actions, OpenAI) 기반 1차 제작 후, 로고 콜라주 스타일로 재구성
   - "인프라 구축"/"시스템 아키텍처" 슬라이드: AI가 처음 생성한 내용(EC2/RDS/S3/ALB, MySQL, RAG/LLM)이 실제 구성과 전혀 다름을 발견 — 실제 구성(Lightsail 단일 인스턴스 + Docker Compose 3컨테이너, MariaDB, OpenAI 챗봇, migration 안전배포 게이트 포함 CI/CD)으로 정정하는 프롬프트 작성
   - "공정상태 페이지 — 동작원리/시나리오/실제 화면 1·2" 4장 초안 완료 — 디버깅 과정(타임존 처리 등)은 텍스트로 나열하지 않고 결과만 짧은 태그로 강조, 상세 설명은 라이브 데모로 대체하는 방향으로 정리

**막힌 지점**

| 문제 | 원인 | 해결 |
|---|---|---|
| 새 배치 등록 시 AGV 애니메이션 없이 즉시 라인 점유 표시 | `NOW()`가 SYSTEM 타임존(호스트 OS)을 따라 로컬(KST)에서 UTC 아닌 값 반환, naive datetime을 UTC로 잘못 라벨링 | DB 연결에 `SET time_zone = '+00:00'` 고정 |
| WAITING ZONE 점유 표시가 AGV 도착 전에 먼저 켜짐 | `slot.batchId` 존재 여부로만 점유 판정, `staging`/`waiting` 상태 구분을 무시 | `entry.state === 'waiting'` 기준으로 변경 |
| 검사 시작 버튼 비활성화(신규 배치) | 라인 이송 미완료 상태(의도된 설계) | 문제 아님 — 공정상태 화면에서 라인 이송 완료 후 정상 활성화 확인 |
| 검사 진행 중 이미지 대부분 "로딩 중"에서 멈춤, 결과 화면도 빈 상태 | 캐러셀 버퍼(11~15개 마운트) 전체가 실제 원본 이미지를 요청, 취소 로직 없이 검사 완료까지 328개 누적 | 이미지 요청 범위를 현재 위치 기준 최대 9개로 제한(모션은 유지) |
| `fix/websocket-invalid-url-empty-api-base` 브랜치 diff가 대량 삭제처럼 보임 | 8/13 시점(최근 기능들 반영 이전)에서 분기된 오래된 브랜치라 dev와 비교 시 그 사이 모든 변경이 "차이"로 잡힘 | 실제 유효 커밋만 별도 확인 → 이미 PR #35로 반영되어 있어 조치 불필요 |
| 이미지로딩 worktree에서 backend 테스트 1건 error | 해당 worktree에 로컬 DB 인증정보(`.env`)가 자동 복사되지 않음(체크아웃별 개별 파일) | 코드 회귀 아님 확인, 급하지 않아 보류 |
| 로컬(노트북) 검사화면 이미지 로딩 재현 불가 | 원본 이미지 파일 자체가 로컬에 없어(8/18부터 확인된 문제) 요청이 404로 빠르게 실패, 재현하려던 "느린 로딩" 증상 자체가 안 나타남 | 코드 검증까지만 로컬에서 완료, 육안 검증은 AWS 배포 후로 이월 |
| 노트북이 PR #61 CI 대기 중 예기치 않게 종료 | 배터리/전원 문제로 추정 | 실제 작업은 GitHub에 이미 반영되어 있어 데이터 손실 없음, `git log` 재확인으로 상태 복구 |

**다음 우선순위 (8/20)**

1. 8/19 2차 배포(AGV/타임존 + 이미지로딩, `43357e2`) SSH 최종 검증(`schema_migrations`, `predeploy backup` 파일) 및 실제 브라우저 회귀 확인(로고, PCB 이미지, AGV 애니메이션 도착기준 점유, 이미지 로딩 모션·요청개수)
2. YOLO 최종 모델(22MB) 반영 후 순차/동시성 부하 재테스트 (8/17 실측은 이전 모델 기준)
3. 로고 34×34 축소 반영 — 로컬 변경만 존재, YOLO 재배포와 함께 묶어서 진행
4. 발표전 최종 테스트 계획(TC-01~) 재개 — 오늘 버그 대응으로 중단된 지점부터
5. Default Branch를 `main`으로 원복 (8/13부터 계속 이월)
6. 발표자료(PPT) 계속 작업 — 공정상태 슬라이드 4장 확정, 나머지 기능 슬라이드 진행

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

### 성능·부하 테스트 (신규, 8/17)
- **처리 속도는 배치 크기·동시 실행 개수와 거의 무관하게 일정**할 수 있음(이번엔 PCB 1개당 0.22~0.28초로 20개든 500개든, 1개 라인이든 8개 동시든 동일) — 이런 경우 총 처리시간은 "PCB 총량 × 개당 시간"으로 예측 가능
- **세마포어(`asyncio.Semaphore`) 같은 전역 동시성 제한 장치가 있으면 요청이 몰려도 CPU가 폭주하지 않고 순차 처리로 안전하게 흡수됨** — 부하 테스트 시 이런 장치가 실제로 설계대로 작동하는지 CPU/메모리 실측으로 반드시 확인할 것
- 순차 부하(단일 배치 대용량)와 동시성 부하(다수 배치 소량)는 **서로 다른 축**이라 각각 따로 검증해야 함 — 한쪽만 통과했다고 다른 쪽도 안전하다고 가정하지 말 것(단, 이번 사례처럼 두 축의 특성이 뚜렷하면 최악의 조합까지 실측하지 않고도 합리적 예측은 가능)

### CI / 실제 배포 환경 불일치 (신규, 8/18)
- **CI 통과가 실제 배포 컨테이너 동작을 보장하지 않음** — CI는 `PYTHONPATH=repo root`로 절대 임포트(`backend.app...`)가 통하지만, 실제 Docker 이미지는 `backend/app`만 COPY하므로 컨테이너 안엔 `backend` 패키지 자체가 없어 같은 코드가 컨테이너에서는 100% 실패할 수 있음 → 새 실행 파일(runner, CLI 등)을 만들 때는 임포트 방식을 CI 환경이 아니라 실제 컨테이너 기준으로 검증할 것
- **가장 위험한 유형은 "실패하지 않고 조용히 아무것도 안 하는" 버그** — `Dockerfile`이 필요한 디렉터리를 이미지에 COPY하지 않으면, 그걸 참조하는 로직이 에러 없이 "대상 0개"로 조용히 처리되어 겉으로는 정상 배포처럼 보임. 에러가 나는 버그보다 훨씬 늦게 발견됨 → 새 배포 파이프라인 구성 요소는 반드시 실제 `docker compose run`으로 실패 경로까지 CLI로 재현해 검증할 것
- **pre-deploy backup의 의미를 지키려면 `plan`(조사)과 `apply`(실행) 단계를 코드 레벨에서 확실히 분리해야 함** — 설계 문서상 순서가 맞아 보여도 실제 구현에서 `plan`이 조금이라도 DB를 변경하면, backup 이전에 이미 상태가 바뀌어버려 backup의 존재 이유가 무너짐

### Migration 안전 배포 설계 원칙 (신규, 8/18)
- 기존 legacy `migrations/*.sql`(destructive/rollback/duplicate 혼재)은 자동 실행 대상에서 완전히 물리적으로 분리하고, 현재 운영 DB 상태를 baseline으로 문서에만 선언(실행된 것처럼 history에 소급 등록하지 않음)
- 신규 forward migration은 `deploy_migrations/`에 SQL+verify SQL 쌍으로만 추가하고, `schema.sql`도 같은 PR에서 동시 갱신을 의무화해야 "새 DB 생성 결과 = 운영 DB 최신 상태" 약속이 실제로 유지됨
- MariaDB DDL은 implicit commit이 발생해 DDL 실행과 history INSERT를 하나의 트랜잭션으로 묶을 수 없음 — "DDL은 성공했는데 history 기록 전에 장애가 남"이라는 상황을 verify 기반 `RECONCILED` 판정으로 복구하는 상태 기계가 필요(단순 `BEGIN-DDL-INSERT-COMMIT`으로는 원자성이 보장되지 않음)
- `verify.sql`은 read-only 계약(단일 SELECT/SHOW, 정확히 1행 반환)을 강제해야 함 — `RECONCILED` 판정 전체가 verify 정확성에 의존하므로, 계약이 느슨하면 "실제로 잘못된 DB 상태"를 "정상"으로 오판할 위험이 있음
- bootstrap(초기 참조 데이터)과 migration(구조 변경)은 책임을 분리 — bootstrap이 `schema_migrations`에 위장 등록되지 않도록 하고, 운영 중인 상태값(예: 버퍼 슬롯의 `status`/`batch_id`)은 재실행 시 절대 덮어쓰지 않는 `ON DUPLICATE KEY UPDATE <PK> = VALUES(<PK>)` 같은 no-op 패턴 사용

### 여러 AI 에이전트 병행 작업 (신규, 8/18)
- Codex와 Claude 등 서로 다른 에이전트를 같은 저장소에서 쓸 경우, **같은 working tree에서 브랜치만 전환하며 병행**하면 미커밋 변경이 충돌·유실될 위험이 있음 — 물리적으로 분리하려면 `git worktree`가 필요하나, 관리 부담을 줄이려면 "한 작업을 완전히 끝내고(push까지) 다음 작업 시작" 순차 진행이 더 안전함
- 작업 도구를 전환할 때(예: 한 에이전트 세션 종료 후 다른 에이전트로 이어감)는 **브랜치 상태·최신 커밋 SHA·완료된 단계(Phase)를 명시적으로 요약해 인계**해야 함 — 요약 없이 이어가면 이전에 합의한 설계 원칙(legacy 격리, verify 계약 등)을 새 세션이 놓치고 반복 위반할 위험이 있음
- 에이전트 세션의 **주간 사용량·컨텍스트 윈도우 잔량**을 무거운 작업(운영 배포 등) 시작 전에 확인하는 습관 필요 — 부족한 상태로 시작해 중간에 끊기면, 배포 파이프라인처럼 되돌리기 번거로운 작업일수록 피해가 큼

### Frontend / Production 환경 특이사항 (신규, 8/18)
- **`new URL(상대경로, base)`는 `base`가 완전한 절대 URL이 아니면 예외를 던짐** — production 최적화로 `API_BASE`를 same-origin 빈 문자열로 바꾸면, 상대경로 처리 코드가 이 경우를 명시적으로 분기하지 않는 한 조용히 깨짐(catch에서 `null` 반환 → `<img>` 자체가 안 그려져 Network 탭에 요청조차 안 보이므로 원인 추적이 까다로움)
- 로컬 개발 환경(특히 새로 세팅한 컴퓨터)에 원본 데이터셋(이미지 파일 등)이 없는 것과 코드 버그를 구분할 것 — 동일한 404/표시 안 됨 증상이라도 원인이 다를 수 있으므로, 로컬 재현이 안 되면 실제 서버(AWS)에서 최종 검증해 코드 문제와 환경 문제를 갈라볼 것

### AGV 타임존 버그 디버깅 (신규, 8/19)
- MariaDB 세션 타임존 기본값은 SYSTEM(호스트 OS 타임존)을 따름 — 도커 컨테이너(UTC)와 로컬 개발 머신(KST) 환경이 다르면 `NOW()`가 서로 다른 값을 반환할 수 있음. "`NOW()`는 항상 UTC"라는 가정은 위험 — 실제로 DB 연결에 `SET time_zone = '+00:00'`을 명시적으로 고정해야 환경 무관하게 일관된 동작 보장
- naive datetime(오프셋 없는 타임스탬프)을 잘못된 타임존으로 라벨링하면, 프론트의 `Date.parse()`가 실제와 반대 방향(과거를 미래로)으로 해석할 수 있고, 이게 `Math.max(0, elapsedSeconds)` 같은 방어 코드에 의해 조용히 0으로 clamp되어 버그가 겉으로 티가 안 날 수 있음 — 특히 "경과시간이 원래 0에 가까운 케이스"(최초 생성)에서는 안 드러나고 "경과시간을 이어받아야 하는 케이스"(재접속)에서만 드러나므로, 회귀 테스트는 반드시 두 케이스를 다 검증해야 함
- 재현 테스트를 만들 때는 "몇 초 뒤 재접속하면 남은 시간만 이어서 진행되는지"처럼 실제 수치(예: 2.2초 후 재접속 → 예상 2.8초 vs 실제 5.66초)를 재서 비교하면 눈에 안 띄는 버그도 정량적으로 잡아낼 수 있음

### UI 상태 판정 설계 원칙 (신규, 8/19)
- "배정됨(assigned)"과 "도착 완료(arrived)"처럼 유사해 보이지만 다른 두 상태를 하나의 플래그(예: `slot.batchId` 존재 여부)로 뭉뚱그려 판정하면, 데이터 모델에 이미 정확한 구분(`state: staging/waiting`)이 있어도 뷰 레이어가 그걸 무시하는 버그가 생길 수 있음 — 점유/진행 상태를 표시할 때는 반드시 어떤 상태값을 기준으로 판정하는지 명시적으로 확인할 것

### 오래된 브랜치 검토 (신규, 8/19)
- 오래 방치된 브랜치를 최신 dev와 `git diff --stat`으로 비교하면 대량 삭제처럼 보일 수 있음(그 브랜치 분기점 이후 dev에 추가된 모든 것이 "차이"로 잡히기 때문) — 이는 실제 삭제 의도가 아니라 단순히 오래된 기준점 때문일 수 있으므로, 그대로 merge하지 말고 반드시 실제 유효 커밋(`git show <sha>`)만 따로 추출해서 검토할 것. 이번 경우 실제로 필요했던 수정은 이미 오래전에 별도 PR로 정상 반영되어 있었음(중복 작업 방지)

### 실시간 UI 성능·이미지 로딩 (신규, 8/19)
- 캐러셀처럼 "여러 카드를 미리 마운트해 애니메이션을 부드럽게 하는" 구조에서는, **마운트 개수(애니메이션용)와 실제 리소스 요청 개수(이미지 등)를 분리**해야 함 — 마운트는 넉넉하게 하되(모션 유지), 실제 무거운 리소스 요청은 훨씬 좁은 범위로 제한하는 "윈도우" 패턴이 효과적
- "로딩 중" placeholder가 있다고 해서 실제 네트워크 요청이 지연되고 있다는 보장은 없음 — 화면에 숨겨진 `<img>` 태그가 이미 요청을 쏘고 있을 수 있으니, 시각적 로딩 상태와 실제 요청 발생 여부는 devtools로 별도 확인 필요
- 로컬 개발 환경에 실제 데이터(이미지 파일 등)가 없으면 오히려 문제가 "빠른 실패(404)"로 위장되어 원래 재현하려던 "느린 로딩" 증상 자체가 재현이 안 될 수 있음 — 이런 경우 무리해서 로컬 재현에 시간 쓰지 말고, 코드 검증(테스트, diff 안전성)까지만 로컬에서 하고 실제 육안 확인은 실 데이터가 있는 환경(AWS)에서 하는 것이 효율적

### 세션·도구 안정성 (신규, 8/19)
- 노트북이 예기치 않게 종료되어 AI 세션의 대화 컨텍스트가 일부 유실되어도, 실제 작업 결과(커밋, PR, merge)는 GitHub에 이미 반영되어 있으므로 데이터 손실은 아님 — 다만 이런 경우 "기억이 안 난다"는 걸 추측으로 메우지 말고, `git log`/`gh pr list` 등으로 실제 원격 상태를 다시 확인한 뒤 이어가는 것이 안전

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
- [x] ~~저장소 Default branch를 dev로 변경 요청~~ (8/13 완료) → **8/15: 다시 main으로 되돌릴 것 (8/19까지도 미완료, `datamon78` Admin 계정 필요 — 8/20으로 재이월)**
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
- [x] **YOLO 모델(`best.pt`) 소재 확인 + 서버 업로드** (8/17 완료 — 이미 Git 추적 중, 별도 업로드 불필요, 품질 검증(Precision 0.957/Recall 0.915/mAP50 0.947)까지 완료)
- [x] **CI에 MariaDB 서비스 컨테이너 추가** — 실 DB 검증 테스트 CI 정식 지원 (8/18 완료, PR #50, 399→420 passed 흐름으로 반영)
- [ ] SSH 키 파일 로컬 정리 (발표용 단기 프로젝트라 우선순위 낮음, 보류 결정)
- [x] **DB 볼륨 재사용 시 migration 자동 미실행 문제 — 정식 해결 체계 구현·로컬 검증 완료** (8/18, schema_migrations+Python runner+buffer bootstrap+predeploy backup+deploy.yml 연결 설계까지)
- [x] **DB Migration 안전 배포 체계 — AWS 실제 배포(`Run workflow`) 및 최종 검증** (8/19 완료 — PR #53/#54, `9b1f0f5` 배포, SSH·브라우저 검증까지 완료)
- [x] **PCB 이미지 표시 오류(production API_BASE 빈 문자열 이슈) 발견·수정·dev/main/AWS 배포·실제 화면 검증** (8/18, 신규)
- [x] **대시보드 좌측 상단 로고 이미지 반영** (8/19 완료 — PR #53/#54로 AWS 배포·브라우저 확인까지)
- [ ] **로고 34×34 축소 재조정** (8/19 로컬 변경만 존재, 아직 미커밋·미배포 — YOLO 재배포 시점에 함께 반영 예정)
- [x] **공정상태(AGV/LINE) 즉시점유·조기점유 버그 2건 발견·근본원인 규명·수정** (8/19, `fix/process-status-agv-occupancy-sync` — 타임존 처리(`SET time_zone='+00:00'`)+상태(`staging`/`waiting`) 기반 판정으로 해결, PR #59)
- [x] **검사화면 실시간 이미지 로딩 과다요청 문제 발견·완화** (8/19, `fix/inspection-image-loading-investigation` — 이미지 요청 윈도우 최대 9개로 제한, PR #60)
- [ ] **8/19 2차 배포(`43357e2`, AGV+이미지로딩) SSH 최종 검증 및 실제 브라우저 회귀 확인** (8/20 이월)
- [ ] **YOLO 최종 모델(22MB) 반영 후 부하 재테스트** (8/17 실측은 이전 모델 기준, 8/20 이월)
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
- [x] **YOLO 순차/동시성 부하 테스트 완료, 4GB 서버로 충분 확정 (8/17)**
- [x] **CI MariaDB 서비스 컨테이너 통합, 실 DB 기반 테스트 전환 (8/18, PR #50 → dev)**
- [x] **AWS 운영 DB 읽기 전용 실태조사 — 미적용 forward migration 0개 확인, baseline 확정 (8/18)**
- [x] **DB Migration 안전 배포 체계 Phase 1~3 구현·로컬 검증 완료 (8/18)** — schema_migrations, Python runner(plan/apply/verify), buffer bootstrap, predeploy backup, deploy.yml 연결 설계. 배포 전 치명적 버그 2건(절대 임포트, Dockerfile COPY 누락) 사전 발견·수정. AWS 실배포는 8/19로 이월
- [x] **PCB 이미지 미표시 버그(production API_BASE 빈 문자열) 발견·수정·dev/main/AWS 배포·실제 화면 검증 완료 (8/18)**
- [x] **DB Migration 안전 배포 체계 AWS 실제 배포 완료 + 로고 반영 배포 완료 (8/19)** — PR #53→dev→PR #54→main, AWS 배포·헬스체크·실제화면(로고+PCB이미지) 확인까지
- [x] **공정상태 AGV 즉시점유·WAITING ZONE 조기점유 버그 2건 발견·근본원인 규명·수정 (8/19)** — 타임존 처리(DB 세션 UTC 고정)로 해결한 즉시점유 문제, `staging`/`waiting` 상태 기반 판정으로 해결한 조기점유 문제. Playwright 실브라우저 재현·8/16 fix 회귀 없음까지 검증
- [x] **검사화면 실시간 이미지 로딩 과다요청 문제 원인 규명·완화 조치 (8/19)** — 캐러셀 애니메이션(11~15개 마운트)은 유지하며 실제 이미지 요청만 최대 9개로 제한
- [x] **오래된 미병합 브랜치(`fix/websocket-invalid-url-empty-api-base`) 검토 (8/19)** — 실제 유효 수정이 이미 PR #35로 반영되어 있음을 확인해 안전하게 정리
- [x] **AWS main 2차 배포 완료(`43357e2`, YOLO 최종모델+AGV수정+이미지로딩개선) (8/19)** — 배포·헬스체크까지 확인, SSH/브라우저 최종검증은 8/20 이월

---

## 중단 기준 (Go / No-Go)

| 기한   | 조건               | 대응                |                                                                   |
| ---- | ---------------- | ----------------- | ----------------------------------------------------------------- |
| 8/12 | 로컬 compose 구동 실패 | AWS 포기, 로컬 데모     | ✅ 통과                                                              |
| 8/13 | OpenAI 전환 실패     | 챗봇 데모 제외          | ✅ 통과 (8/15 서버 반영까지 완료)                                            |
| 8/15 | 재고관리 미완          | 미완 상태로 동결         | ✅ 통과 (`batch_material_consumption` 등 재고 관련 기능 8/8 이전 이미 구현·연동 확인) |
| 8/16 | CD 미완            | 수동 배포 유지          | ✅ 통과 (수동 `workflow_dispatch` 방식, 오늘도 3회 정상 배포)                    |
| 8/17 | 서버 YOLO 추론 실패    | 8GB 증설 → 사전 계산 결과 | ✅ 통과 (4GB로 충분, 8GB 증설 불필요 확정. Precision 0.957 / Recall 0.915 / mAP@50 0.947, 단일 500개·8라인 동시 160개 모두 메모리 13% 내외) |

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
 └─ 볼륨 : PCB 이미지(703장, 8/15 업로드 완료), best.pt (8/19 팀원이 최종 튜닝 모델로 교체, 22MB, 부하 재테스트 8/20 이월)
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