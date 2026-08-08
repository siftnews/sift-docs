# Sift — 진행 장부

> **현재 진행 중인 이슈 1개의 구현 단계**만 담는다.
> 작업 목록의 원본은 GitHub 이슈 본문의 `## TODO`다. 태스크 후보는 [TASKS.md](./TASKS.md), 현재 상태는 [STATE.md](./STATE.md)를 따른다.

## 진행 중

### [#66 모니터링 스택](https://github.com/siftnews/sift-api/issues/66)

- [x] 이슈 발행·범위 확정·`develop` 기반 독립 브랜치 생성
- [x] `micrometer-registry-prometheus` runtime registry와 Actuator scrape endpoint 구현
- [x] dispatch/send/retry Timer histogram 설정 및 기존 endpoint 노출 보존
- [x] Prometheus 응답·custom metric/histogram 노출 통합 회귀 테스트 추가
- [x] 대상 테스트·전체 테스트(249건)·`check`·모듈 경계·diff 정적 검사를 통과했다.
- [x] 관측 문서와 D-044로 Prometheus/Grafana 인프라 경계를 기록했다.
- [x] `chore/66-monitoring-stack` head `585f09a`를 PR #67로 생성했다.
- [ ] PR #67 리뷰 및 병합은 사용자 게이트다.


## 최근 완료

### [#64 10만 구독자 시드 생성기](https://github.com/siftnews/sift-api/issues/64)

- [x] `loadtest` 프로파일 전용 결정적 100,000건 subscriber 시드와 D-019 아침 피크 분포를 구현했다.
- [x] 이메일 UNIQUE 기준 bulk 멱등 저장과 회귀 검증을 완료했다.
- [x] stacked PR #65의 CI·CodeRabbit 검토가 통과했다.
- [ ] PR #65 병합은 사용자 게이트다.

### [#62 retryJob 오류 분류 및 지수 백오프](https://github.com/siftnews/sift-api/issues/62)

- [x] `feature/60-send-step` 위에 `feature/62-retry-job`으로 구현·검증했다.
- [x] 단위·reader·트리거·메트릭·Testcontainers 통합 회귀 검증과 전체 테스트(246건)를 완료했다.
- [x] 스택드 PR #63을 생성했다.
- [ ] PR #63의 최신 head `034c1aa`는 non-default base라 GitHub Actions/CodeRabbit 자동 검토가 skip된다.

### [#60 sendStep 고정 HTML 렌더링 및 로컬 SMTP 발송](https://github.com/siftnews/sift-api/issues/60)

- [x] 구현·전용 회귀 검증 및 설계 원본 갱신을 완료했다.
- [x] CodeRabbit actionable 지적 9건을 반영한 후속 커밋 `6cfd123`을 PR #61에 push했다.
- [x] 최신 CodeRabbit 지적 중 task별 실패 결과·부분 실패 ExitStatus와 `SENDING → FAILED` 회귀 검증을 반영하고 `c44e75c`까지 push했다.
- [ ] `SENDING` lease 만료 복구는 스키마·운영 정책을 추가하는 별도 범위라 PR #61에 미결정 게이트로 남아 있다. 최신 CodeRabbit 검토는 rate limit 상태다.
- [ ] PR #61 병합은 사용자 게이트다.

### [#58 발송 스냅샷과 매시 pull 스캔 구현](https://github.com/siftnews/sift-api/issues/58)

- [x] delivery_job·delivery_task Liquibase 스키마와 제약을 추가했다.
- [x] Delivery 도메인·포트·서비스 및 멱등 키를 구현했다.
- [x] Content·Subscriber named interface 조회를 연결했다.
- [x] 매시 pull 스캔 트리거와 snapshotStep을 구현했다.
- [x] 도메인·서비스·영속 회귀와 전체 테스트를 검증했다.
- [ ] dispatchJob 트리거·Job 실행의 통합 회귀는 후속 점검 항목이다.
- [x] delivery_job 상태 전이·완료 판정 설계 원본을 갱신했다.
- [x] PR #59를 병합했다.

### [#55 Subscriber/Subscription 도메인 및 구독 API](https://github.com/siftnews/sift-api/issues/55)

- [x] 이슈·브랜치 생성 및 API 계약을 확정했다.
- [x] subscriber·subscription Liquibase 스키마를 추가했다.
- [x] Subscriber/Subscription 도메인·서비스·영속 어댑터를 추가했다.
- [x] Content named interface를 통한 활성 토픽 조회를 연결했다.
- [x] REST 어댑터와 경계 오류 응답을 추가했다.
- [x] 웹·영속 통합 검증과 문서 검증을 완료했다.
- [x] PR #56 및 동시성·매퍼 복원 회귀를 보강한 후속 PR #57을 병합했다.

### [#52 Atom 피드 파싱 검증 및 네이버 D2 소스 추가](https://github.com/siftnews/sift-api/issues/52)

- [x] Atom 피드 픽스처와 `RssFeedAdapter` 회귀 테스트를 추가했다.
- [x] 네이버 D2 Atom URL의 실제 응답을 확인했다.
- [x] `SourceSeedData`에 네이버 D2를 추가하고 시더 멱등성 기대값을 10개로 갱신했다.
- [x] 전체 테스트·문서 검증 및 PR #54 병합을 완료했다.
