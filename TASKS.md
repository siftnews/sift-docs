# Sift — 작업 후보

> 이슈 후보와 미완료 로드맵의 원본이다. 완료 이력은 GitHub 이슈·PR과 Git 히스토리에서 조회한다.
> 이슈 1개 = 작업 1개 = PR 1개. 발행된 항목에는 `#번호`를 붙이고, 현재 진행 중인 작업은 [BACKLOG.md](./BACKLOG.md)에 둔다.

## 현재 진행

- [ ] [#55 `[FEAT] Subscriber/Subscription 도메인 및 구독 API`](https://github.com/siftnews/sift-api/issues/55) — 구현 진행 (`feature/55-subscriber-subscription`)
- [x] [#47 `[CHORE] 배치 통합 테스트의 시각 의존 제거 (Clock 고정)`](https://github.com/siftnews/sift-api/issues/47) — PR #48 및 후속 PR #49 병합
- [x] [#52 `[FEAT] Atom 피드 파싱 검증 및 네이버 D2 소스 추가`](https://github.com/siftnews/sift-api/issues/52) — PR #54 병합

## M2 잔여

- [ ] **`[CHORE] Sift용 스킬 박제`** — M2 유스케이스에서 공통 패턴을 추출해 헥사고날 유스케이스 구현·코드 검토·브랜치 생성 절차를 정리한다.
- [x] **`[#50] [CHORE] Liquibase 마이그레이션 도입`** — 제약·인덱스·데이터 백필을 버전드 마이그레이션으로 관리하고, 기존 시더의 이전 범위를 결정한다.
- [x] **`[#52] [FEAT] Atom 피드 파싱 검증 및 네이버 D2 소스 추가`** — Atom 피드 픽스처 테스트와 실제 D2 소스 시드를 추가했다(PR #54).

## M3 — 구독·발송

- [ ] **`[FEAT] Subscriber/Subscription 도메인 + 구독 API`** — 구독/해지 UseCase, REST 어댑터, `preferred_send_hour`를 포함한다.
- [ ] **`[FEAT] 발송 스냅샷 (delivery_job/delivery_task)`** — DispatchIssueUseCase와 멱등 키를 정의하고 매시 pull 스캔 트리거를 구현한다.
- [ ] **`[FEAT] sendStep — 템플릿 렌더링 + SendEmailPort`** — 메일 템플릿 엔진 선택과 단일 스레드 발송을 구현한다.
- [ ] **`[FEAT] retryJob — 오류 분류 + 지수 백오프`** — 재시도·격리·다음 재시도 시각을 정의한다.

## M4 — 측정·부하

- [ ] **`[CHORE] 10만 구독자 시드 생성기`** — 재실행 가능한 시드와 수신 시각 분포를 제공한다.
- [ ] **`[CHORE] 모니터링 스택`** — Prometheus·Grafana와 배치 대시보드의 위치를 결정한다.
- [ ] **`[FEAT] V1 부하 측정 — 베이스라인 확보`** — 10만 건 발송의 소요 시간·TPS를 측정한다.
