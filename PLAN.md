# Sift — 뉴스레터 배치 서비스 기획안

> 프로젝트명: **Sift** (체로 거르다 = 원하는 뉴스만 선별) · 최종 수정: 2026-07-07 · 상태: 구현 진행 (진행 추적은 STATE.md)
>
> 구성: 백엔드 + 프론트엔드 · GitHub 조직 [`siftnews`](https://github.com/siftnews) 로 관리 (브랜드 표기는 `Sift`)

---

## 0. 배경 / 문제 정의 (왜 만드는가)

**"뉴스를 봐도, 내가 원하는 콘텐츠를 식별하기가 너무 어렵다."**

매일 쏟아지는 뉴스 속에서 정작 내게 의미 있는 정보를 골라내는 데 드는 인지 비용이 너무 크다.
이 프로젝트는 그 문제를 **"수집 → 자동 선별 → 정리된 형태로 받아보기"** 라는 흐름으로 풀어보려는 시도에서 출발한다.

- 1차 목표: 외부 뉴스 소스를 자동 수집하고, 정해진 기준으로 선별·정리해 **메일로 자동 발송**한다.
- 확장 방향(씨앗): 선별 기준을 점차 **개인 관심사 기반 필터링/개인화**로 발전시킨다.
  - 즉, "원하는 콘텐츠 식별"이라는 원래 문제의식이 곧 이 서비스의 장기 차별점이 된다.

### 포트폴리오로서의 정체성
기능 그 자체보다 **"구독자 규모를 키워가며 발송 배치의 병목을 측정하고 단계적으로 개선한 기록"** 이 이 프로젝트의 핵심 상품이다.
채용 관점에서 읽히는 것은 화려한 기능이 아니라 **개선 서사 + 수치**다. 그래서 처음부터 분산 구조로 만들지 않고, 의도적으로 단순하게 시작해 한계를 측정하고 깨나간다.

---

## 1. 한 줄 정의

> 외부 뉴스를 자동 수집·템플릿 편집하여 구독자에게 안정적으로 대량 자동 발송하는 뉴스레터 배치 시스템

- **콘텐츠 형태**: 외부 소스 수집형 (RSS/API)
- **발송 방식**: 완전 자동 발송 (운영자 개입 0) — 매일, **구독자가 선택한 시각**에 (D-019)
- **배치 핵심**: 대량 발송 스케줄링 + 재시도/실패 관리
- **자동 편집**: 템플릿 기반 (LLM 변수 제거 → 배치 성능에 집중)
- **선별 방식**: 토픽 구독 + 스코어링·랭킹 파이프라인 → 상세 [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)

---

## 2. 기술 스택

| 영역 | 선택 |
|---|---|
| 언어/프레임워크 | Spring Boot 3 / Java 21 |
| 아키텍처 | 헥사고날 (포트 & 어댑터) |
| 배치 | Spring Batch |
| 영속성 | JPA + PostgreSQL |
| 메일 발송 | `SendEmailPort` 추상화 + 적응형 (개발·부하테스트: 로컬 SMTP / 실증: AWS SES) |
| 부하 테스트 | k6 또는 Gatling |
| 모니터링 | Actuator + Prometheus + Grafana |

> **메일 발송 적응형 전략**: 발송 수단을 아웃바운드 포트 뒤에 두고 프로파일로 전환한다.
> - `LocalSmtpAdapter` (MailHog/Mailpit): 비용 0, 한도 없음 → 병목을 "내 서버 처리량"에 집중시켜 성능 실험에 최적
> - `SesAdapter` (AWS SES): 실제 발송·바운스·트래킹·Rate Limit 경험
> - 헥사고날의 포트/어댑터 가치가 그대로 드러나는 지점

### 저장소 구조 (멀티레포 · GitHub 조직 `siftnews`)

배포 단위·CI가 다르고, 포트폴리오에서 각 영역이 레포로 분리돼 명확히 보이도록 **멀티레포**로 간다.
조직 핸들은 `siftnews`, 레포는 `sift-` 접두어로 통일한다 (조직명과 레포명은 같을 필요 없음).

```
조직: siftnews   (브랜드: Sift)
├─ sift-api      Spring Boot 백엔드 — 핵심
├─ sift-web      프론트엔드
├─ sift-infra    docker-compose, k6 부하테스트, Prometheus/Grafana
└─ sift-docs     기획/설계 문서 (또는 각 레포 docs/ 분산)
```

> 현재 로컬 작업 디렉토리는 `siftnews/` 이며, 향후 위 레포들이 이 아래에 자리 잡는다.
> **D-015·D-021**: 루트 `siftnews/` 자체는 git 저장소화하지 않되(D-015), 공통 문서(`docs/`)는 **`sift-docs` 레포로 공개**하고, 레포 종속 설계 문서(MVP-DESIGN·SELECTION·EVENTS)는 **각 레포 `docs/`** 를 원본으로 둔다(D-021). `sift-infra` 분리 여부는 TASKS M4에서 결정.

---

## 3. 도메인 (Bounded Context)

| 컨텍스트 | 책임 | 비고 |
|---|---|---|
| **Source** (수집) | RSS/API 외부 소스 등록·크롤링·정규화, **기사(Article) 저장·소유 (D-018)** | 배치 ① |
| **Content** (콘텐츠) | **토픽별 선별**(스코어링·랭킹), 템플릿 이슈 구성 — 후보 기사는 Source named interface로 조회 | 배치 ② → [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md) |
| **Subscriber** (구독) | 구독/해지, **토픽 구독**, 수신 동의, 바운스·옵트아웃 | |
| **Delivery** (발송) | DeliveryTask 스냅샷, 대량 발송, 재시도 | 배치 ③④ — **성능 핵심** |
| **Tracking** (추적) | 오픈/클릭/바운스 이벤트 수집·집계 | 배치 ⑤ (2차) |

### 모듈/패키지 구조

```
newsletter
├─ source      (수집)  : RSS/API 크롤링, 정규화, 기사 저장·소유 (D-018)
├─ content     (콘텐츠): 자동 선별(스코어링·랭킹), 템플릿 이슈 구성
├─ subscriber  (구독)  : 구독/해지, 세그먼트, 바운스·옵트아웃
├─ delivery    (발송)  : DeliveryTask 스냅샷, 대량 발송, 재시도  ← 성능 핵심
└─ tracking    (추적)  : 오픈/클릭/바운스 집계

각 컨텍스트 내부:
  domain
  application
    ├─ port.in   (UseCase)
    ├─ port.out  (Repository / SendEmailPort 등)
    └─ service   (UseCase 구현)
  adapter
    ├─ in   : batch(Job/Step), web(REST)
    └─ out  : persistence(JPA), esp(메일 발송)
```

- **Spring Batch Job = 인바운드 어댑터.** 도메인은 배치를 알지 못한다.
- **`SendEmailPort` = 아웃바운드 포트.** 구현체를 프로파일로 전환.

---

## 4. 핵심 플로우

```
[Source 배치]        [Content 배치]            [Delivery 배치]              [Tracking]
RSS/API 수집  ──▶  중복제거·필터링  ──▶  스케줄 도래 시
                   토픽별 선별(스코어링·랭킹)    │
                   템플릿 이슈 자동 구성         ├▶ 발송 스냅샷 생성 (DeliveryTask)
                                               ├▶ 청크 단위 분산 발송 (Rate Limit 준수)
                                               ├▶ 실패 건 재시도 (지수 백오프)
                                               └▶ 결과 기록 ───────────▶ 오픈/클릭/바운스 집계
```

---

## 5. 발송 도메인 — 성능·재시도의 단일 기준점

발송은 **"발송 작업 생성"과 "실제 전송"을 분리**하는 것이 안정성의 핵심이다.

### 데이터 모델 (초안)

```
topic          (id, name, slug, ...)                                 -- 선별 기준 (→ SELECTION.md)
subscription   (id, subscriber_id, topic_id, status)                 -- 토픽 구독
issue          (id, topic_id, title, status, scheduled_at, published_at)  -- 토픽당 1회분
delivery_job   (id, issue_id, total_count, status, created_at)        -- 발송 한 회
delivery_task  (id, job_id, subscriber_id, email,                     -- 개별 1건 (핵심)
                status[PENDING|SENDING|SENT|FAILED|DEAD],
                attempt_count, next_retry_at, last_error, sent_at)
subscriber     (id, email, status[ACTIVE|UNSUB|BOUNCED], preferred_send_hour)  -- 수신 시각 구독자 선택 (D-019)
```

> `delivery_task`가 멱등성·재시도·집계의 단일 기준점.
> 토픽·선별 관련 상세 스키마(`article`, `article_score`, `issue_item` 등)는 [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md) 참고.

### 상태 머신

```
PENDING → SENDING → SENT
                 └→ FAILED → (재시도) → SENT
                                     └→ DEAD
```

### 재시도/실패 관리 설계 포인트

- **에러 분류**: 일시적(타임아웃·5xx·rate limit) → 재시도 / 영구적(invalid address·4xx) → 즉시 DEAD
- **멱등성**: `delivery_task` 단위 유니크 키 → 배치 중단·재실행 시 중복 발송 방지
- **백오프**: `next_retry_at` 으로 재시도 시각 관리, 도래한 건만 픽업
- **DLQ 개념**: `DEAD` 상태 격리 → 운영자(또는 별도 배치)가 분석·처리
- **부분 실패 복구**: Spring Batch restart/skip/retry 메타데이터 활용

### 헥사고날 매핑 (Delivery 예시)

- inbound port: `DispatchIssueUseCase`, `SendPendingDeliveriesUseCase`, `RetryFailedDeliveriesUseCase` (확정 시그니처는 [MVP-DESIGN §4](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md))
- outbound port: `LoadDeliveryTaskPort`, `SaveDeliveryTaskPort`, `SendEmailPort`
- adapter: 배치 Job/Step(in), JPA(out), ESP 클라이언트(`SendEmailPort` 구현, out)

---

## 6. 성능 개선 로드맵 (포트폴리오 본문)

각 단계 = **부하 측정 → 모니터링 → 병목 분석 → 개선 → 재측정** 1사이클. 이게 README의 챕터가 된다.

| 단계 | 구조 | 만들어지는 병목 | 개선 기법 | 측정 지표 |
|---|---|---|---|---|
| **V1** | 단일 스레드 Chunk Step | 순차 처리, 낮은 TPS | 베이스라인 측정 | 10만 건 발송 소요시간 |
| **V2** | Multi-threaded Step / AsyncItemProcessor | DB 락·커넥션 경합 | 커넥션 풀 튜닝, 벌크 update | TPS, DB wait |
| **V3** | Spring Batch Partitioning | 단일 JVM CPU/IO 한계 | 파티션 분할, 커서 페이징 | 처리량 선형성 |
| **V4** | Kafka 분산 / Remote Partitioning | 단일 노드 한계 | 워커 스케일아웃, 배압 제어 | 노드 추가 시 확장성 |

---

## 7. 로드맵 (MVP → 확장)

### MVP (V1) — 1차 목표: 베이스라인 수치 확보
소스 1~2개 등록 → 수집 배치 → 템플릿 이슈 자동 구성 → DeliveryTask 스냅샷 → 단일 스레드 청크 발송(로컬 SMTP) → 재시도 배치.

### 2차
세그먼트 발송, 오픈/클릭 트래킹, 바운스 자동 처리, 성능 V2~V3.

### 3차
개인 관심사 기반 필터링/개인화(원래 문제의식 해결), 성능 V4, 발송 대시보드.

---

## 8. 다음 단계

- [x] 선별 로직 구체화 — 토픽 구독 + 스코어링·랭킹 파이프라인 → [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)
- [x] MVP 상세 설계 — ERD, 배치 Job/Step 명세, 포트 인터페이스 시그니처 → [MVP-DESIGN.md](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md)
- [x] 프로젝트 스캐폴딩 — Spring 프로젝트 생성, 모듈/패키지 골격 (첫 배치 Job은 TASKS M1 collectionJob 이슈로)
- [ ] 부하 테스트 환경 + 모니터링 스택 구성 (→ TASKS M4)

> 이후 진행 추적은 [STATE.md](./STATE.md)·[TASKS.md](./TASKS.md)로 이관 (2026-07-05).
