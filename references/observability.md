# 관측 (observability)

> **지금 있는 것과 없는 것을 구분해 적는다.** 이 프로젝트의 상품은 *"구독자 규모를 키우며 발송 배치의 병목을 측정·개선한 기록"*([PLAN](plan.md))이므로, 측정 수단은 기능이 아니라 **본문**이다.
> 성능 로드맵 V1~V4의 지표 정의는 [PLAN §6](plan.md)이 원본이다.

---

## 1. 현재 상태 — 로그 기반 측정

| 있는 것 | 없는 것 |
|---|---|
| Spring Boot Actuator (`health`·`info`·`modulith`) | **Prometheus 엔드포인트 미노출** — micrometer 레지스트리 추가 시 |
| `spring-modulith-actuator` · `spring-modulith-observability` (runtimeOnly) | Grafana 대시보드 (계획: `sift-infra`) |
| 배치 `MetricsListener` 3종 → `[measure]` 로그 | 부하 도구(k6) — M4 |
| Spring Batch가 글로벌 레지스트리에 발행하는 `spring.batch.*` 타이머 | 알림·SLO |

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, modulith   # prometheus는 micrometer 레지스트리 추가 시 노출
```

**"아직 없다"는 결함이 아니라 순서다** — 발송(delivery) 모듈이 성능 핵심인데 아직 M2(선별) 단계다. 측정할 병목이 생기는 시점에 도입한다(하네스 원칙 1).

## 2. 계측은 **배치 어댑터**에 둔다 — 서비스가 아니라

`CollectionMetricsListener`(source) · `SelectionMetricsListener`(content) ·
`DispatchMetricsListener`(delivery)는 모두 `adapter/in/batch/`에 있다.

**이유**: 애플리케이션 서비스는 배치를 몰라야 한다(헥사고날 — Spring Batch Job은 인바운드 어댑터다). 서비스에 `MeterRegistry`를 주입하면 도메인 쪽이 실행 수단을 알게 된다.

> 이건 실제로 판정된 사안이다 — **PR #26 리뷰에서 "서비스에 Micrometer를 붙이라"는 제안이 있었고, 위 근거로 반박하고 어댑터에 두기로 정리했다.** 같은 제안이 다시 오면 이 문단을 근거로 든다.

## 3. `[measure]` 로그 규약

측정용 로그는 **`[measure]` 접두어**로 시작한다. 사람이 읽는 동시에 `grep '\[measure\]'`로 뽑아낼 수 있게 하기 위함이다.

```
[measure] step={} read={} write={} filter={} skip={} elapsedMs={} readThroughput/s={}
[measure] job={} status={} elapsedMs={}
[measure] job={} topicId={} runDate={} status={} elapsedMs={}
```

- **Step 실패 시 원인 예외를 함께 남긴다** (`log.warn`). 카운트만으로는 실패 원인을 추적할 수 없다.
- 처리량은 초당 read 건수로 계산해 로그에 직접 넣는다 — 사후 계산을 강요하지 않는다.

### chunk와 tasklet의 차이를 알고 볼 것

| | read/write count | 건수는 어디서 |
|---|---|---|
| **collectStep** (chunk) | 나온다 | `CollectionMetricsListener` |
| **selectionJob 3 Step** (tasklet) | **전부 0** | 서비스가 자기 로그로 남긴다 (예: `스코어링 완료: loaded=… scored=…`) |

tasklet Step에서 `read=0 write=0`은 **결함이 아니다.** 리스너는 그 경우 **단계별 소요시간**을 책임진다. chunk로 전환하면 카운트도 리스너에서 나온다.

## 4. 새 배치 Job을 만들 때

1. `{Job}MetricsListener`를 `adapter/in/batch/`에 만들고 `StepExecutionListener` + `JobExecutionListener`를 구현한다.
2. `afterStep`에 소요시간 + (chunk면) 카운트, `afterJob`에 Job 결과 + 식별 파라미터(`topicId`·`runDate` 등)를 남긴다.
3. 실패 예외를 `log.warn`으로 함께 남긴다.
4. 시간 계산은 `start == null || end == null → Duration.ZERO` 가드를 둔다 — Job이 비정상 종료하면 null이 온다.

`snapshotStep`처럼 tasklet이 수신자 목록을 직접 생성하는 경우, Step의 read/write count는 0이다.
생성 건수는 `DispatchIssueService`가 `[measure] delivery snapshot createdTasks={}`로 남기고,
`DispatchMetricsListener`가 execution context에서 이를 읽어 `sift.delivery.tasks.created` Counter에 기록한다.
소요시간은 `sift.delivery.snapshot.duration`·`sift.delivery.send.duration`·
`sift.delivery.dispatch.duration` Timer로 기록한다. `sendStep`의 write count는
`sift.delivery.tasks.processed` Counter로, task별 실패 건수는
`sift.delivery.tasks.failed` Counter로 남긴다. 발송 task 실패가 있으면 Step ExitStatus는
`COMPLETED_WITH_ERRORS`로 구분한다. claim 경합은 실패로 세지 않으며, task별 INFO 로그를 추가하지
않고 상태·카운터·기존 `[measure]` 로그로 관측한다. 모든 태그는 유한한 `status`만 쓴다.

`retryJob`은 `RetryMetricsListener`가 `sift.delivery.retry.duration`·
`sift.delivery.retry.job.duration` Timer와 `sift.delivery.retry.tasks.processed` Counter로
재시도 Step/Job 소요시간과 처리 건수를 기록한다. 실패 task는
`sift.delivery.retry.tasks.failed` Counter와 `COMPLETED_WITH_ERRORS` ExitStatus로 구분한다.
`sendStep`과 동일하게 실패 원인 예외는 `[measure]` 경고 로그에 남기고, 메트릭 태그에는 유한한
상태만 사용한다.

[PR 체크리스트](../checklists/pr.md)의 CONDITIONAL에 "배치 Job/Step 추가 시 계측" 항목이 있다.

## 5. 앞으로 (도입 시점에 이 문서를 고친다)

| 시점 | 도입 | 목적 |
|---|---|---|
| 발송 배치 착수 (M3) | `micrometer-registry-prometheus` + `prometheus` 엔드포인트 노출 | V1 베이스라인 수치 확보 |
| 성능 사이클 시작 (V1→V2) | Grafana 대시보드, k6 부하 스크립트 (`sift-infra`) | 부하 측정 → 병목 분석 → 개선 → 재측정 |
| V2~ | DB wait·커넥션 풀·TPS 지표 | [PLAN §6](plan.md) 측정 지표 표 |

**로그 기반 측정을 버리지 않는다.** Prometheus가 들어와도 `[measure]` 로그는 단건 추적(어느 실행이 왜 느렸나)에 계속 쓰인다 — 시계열 지표와 역할이 다르다.
