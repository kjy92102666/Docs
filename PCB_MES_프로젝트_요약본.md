# AI Vision 기반 PCB 품질 판별 및 AGV 연계 스마트 MES — 요약본

> 전체 상세 내용은 `PCB_MES_프로젝트_전체정리본_원본.md` 참조

## 한 줄 소개

PCB 이미지를 YOLO로 검사 → 작업자가 최종 판정 → AGV 이송·재고·품질 분석까지 연결한 스마트 MES

## 해결하려는 문제

육안 검사 편차 / 검사·물류·재고 시스템 단절 / 불량 위치·유형 분석 어려움 / 실시간 현황 파악 어려움 / 검사 결과가 후속 공정에 자동 연결 안 됨

---

## 시스템 구조

```
사용자 → React+Vite → REST/WebSocket → FastAPI
                                          ├ 업무 서비스
                                          ├ YOLO 추론 서비스
                                          └ 챗봇 서비스
                                          → MariaDB / 이미지·모델 데이터셋
```

**기술 스택**: React 18·Vite / FastAPI·Python / WebSocket / MariaDB·PyMySQL / Ultralytics YOLO / PyTorch·ProcessPoolExecutor / OpenPyXL·ReportLab / Ollama+OpenAI Provider / Docker Compose·Nginx / pytest·Node Test Runner

**계층 구조**: Router(요청처리) → Service(업무규칙) → Repository(DB) → Schema(Pydantic 검증)

---

## 전체 흐름 (8단계)

1. **검사계획 등록** — 날짜/모델/수량/라인 선택, PLANNED 상태. 취소는 소프트 삭제(CANCELLED)
2. **입고 배치 생성** — production_batch + batched_pcb 생성. 라인당 작업 1개+대기 1개 제한, 모델 혼입 차단
3. **AGV 라인 이송** — WAITING_AREA → TO_LINE → LINE. 8라인/12AGV, 시뮬레이션 기반
4. **YOLO 불량검사** — 3×3 타일 분할 추론 → 좌표 복원 → 병합 저장. 모델은 워커 프로세스에 상주(재로딩 없음)
5. **실시간 화면 반영** — 전용 WebSocket으로 진행률/판정 즉시 갱신
6. **작업자 최종 검수** — PENDING → CONFIRMED_DEFECT/FALSE_ALARM. 확정 후 배치 판정 잠금(DEFECT/NORMAL)
7. **검사 후 AGV 이송** — LINE → 버퍼(16슬롯, EMPTY/RESERVED/OCCUPIED) → 최종 적재(COMPLETED)
8. **재고·후속 공정 인계** — 정상PCB수×BOM수량 계산 → 재고 충분시만 SMD 인계, 트랜잭션+행잠금으로 동시성 보호

---

## 화면 8개 요약

| 화면 | 핵심 기능 |
|---|---|
| 대시보드 | 검사량·불량률·AI수용률·배치 파이프라인·라인/AGV 현황·재고경고 통합 |
| 검사등록 | 캘린더 기반 계획 등록 + 계획→배치 생성 |
| 불량검사(실시간) | 배치 선택→검사시작, WebSocket 실시간 진행률/판정, 캐러셀+Bounding Box |
| 불량검사(결과/검수) | AI판정 승인/수정/오탐처리, 일괄승인, 이미지 확대·BBox 토글 |
| 공정상태 | 8라인·12AGV·버퍼·적재구역 디지털트윈, 상세 패널 |
| 재고관리 | BOM 소요량 계산, ENOUGH/ORDER_NEED/PENDING 판정, SMD 인계 |
| 분석정보 | AI판정 매트릭스·오탐률·재학습검토상태 / Pareto·히트맵 / Excel·PDF 내보내기 |
| MES 챗봇 | Dashboard·Batch 데이터 자연어 질의, Ollama/OpenAI 교체 구조 |

---

## 데이터베이스 핵심 관계

```
production_plan → production_batch → batched_pcb → original_pcb
                                    → inspection_result → defect_review
                                    → post_inspection_buffer_slot
                                    → batch_material_consumption
product_model → bom_item → part
line → agv / sensor_data
```

**설계 핵심**: 계획·배치·PCB·Detection 분리(추적성) / AI판정·작업자판정 별도보존 / 트랜잭션+행잠금 / 삭제 대신 상태관리

---

## 주요 상태 전이

- 계획: `PLANNED → BATCH_CREATED | CANCELLED`
- 배치: `CREATED → LINE_ASSIGNED → INSPECTING → INSPECTED → COMPLETED`
- 물류: `WAITING_AREA → TO_LINE → LINE → TO_POST_INSPECTION_BUFFER → POST_INSPECTION_BUFFER → TO_FINAL_DOCK → FINAL_DOCK`
- 판정: `PENDING → CONFIRMED_DEFECT | FALSE_ALARM`

## 실시간 통신 설계

`/ws`(계획·배치·판정·인계) / `/ws/inspections`(검사진행) / `/ws/process-status`(AGV·라인·버퍼) — 3개 채널 목적별 분리.
**원칙**: REST=정확한 현재상태 조회, WebSocket=변경 알림, 재연결시 REST로 재동기화 → 메시지 유실에도 복원 가능

## 주요 안정성 방어 로직

중복검사/중복이송 차단, 라인 중복점유·모델혼입 차단, 버퍼풀 대기, 이송완료 멱등처리, 판정잠금 후 변경차단, 재고차감 전 행잠금 재검증, 이미지 경로 접근차단, `/health/live`·`/health/ready`

---

## 차별점 5가지

1. **전체 업무 흐름**: 계획→배치→AI검사→사람검수→AGV→재고→분석까지 연결 (단순 검출 데모 아님)
2. **Human-in-the-loop**: AI 결과를 최종 정답 취급 안 함, 작업자 승인/수정/오탐 별도 저장 → 재학습 데이터 축적
3. **타일 추론**: 3×3 분할로 작은 결함 특징 손실 방지
4. **상태 기반 AGV 연계**: 애니메이션이 아니라 배치상태·물류위치·버퍼·라인점유가 실제로 연결됨
5. **품질 데이터 재활용**: 검사 결과를 대시보드/검수/재고/분석/챗봇이 공통으로 사용

---

## 시연 순서 (7~10분)

대시보드 → 검사등록(계획+배치) → 공정상태(AGV 라인이송) → 불량검사 실시간(검사시작+BBox) → 검사결과(승인/수정/오탐) → 공정상태(버퍼+적재) → 재고관리(BOM+SMD인계) → 분석정보(매트릭스+Pareto+히트맵) → 리포트 내보내기 → 챗봇 질의

## 마무리 멘트 (핵심 문장만)

> "AI 판정 정확도만 보여주는 시스템이 아니라, 검사계획부터 배치·AI검사·작업자검수·AGV이송·재고관리·품질분석까지 제조 현장 전체 데이터 흐름을 하나의 MES로 연결했습니다. AI 판정과 작업자 판정을 분리해 신뢰성을 확보하고, 그 결과를 분석·재학습 판단에 재활용합니다."

---

## 한계점 (발표 시 반드시 구분)

- AGV = 하드웨어 제어 아닌 상태전이 시뮬레이션
- 일부 센서 수치는 시뮬레이션
- 인증·권한 관리 미구현
- 검사 작업상태 일부 서버 메모리 의존(다중서버 시 큐 필요)
- 판정 잠금 후 관리자 재개방 기능 없음
- 챗봇 실시간 조회범위는 Dashboard·Batch 한정

## 향후 발전 방향 (핵심만)

실제 AGV/PLC 연동, 카메라 실시간 스트림, 센서 실데이터, Redis/Celery 분산처리, 인증·권한, YOLO 재학습 파이프라인, RAG 지식검색, 예지보전, 오브젝트 스토리지
