# Sift — 진행 장부

> **현재 진행 중인 이슈 1개의 구현 단계**만 담는다.
> 작업 목록의 원본은 GitHub 이슈 본문의 `## TODO`다. 태스크 후보는 [TASKS.md](./TASKS.md), 현재 상태는 [STATE.md](./STATE.md)를 따른다.

## 진행 중

### [#58 발송 스냅샷과 매시 pull 스캔 구현](https://github.com/siftnews/sift-api/issues/58)

- [x] 이슈 발행 및 구현 범위 확정
- [x] delivery_job·delivery_task Liquibase 스키마와 제약 추가
- [x] Delivery 도메인·포트·서비스 및 멱등 키 구현
- [x] Content·Subscriber 모듈 경계 조회 연결
- [x] 매시 pull 스캔 트리거와 snapshotStep 구현
- [ ] 영속·배치 통합 및 동시성 회귀 검증
- [x] delivery_job 상태 전이·완료 판정 설계 원본 갱신



## 최근 완료

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
