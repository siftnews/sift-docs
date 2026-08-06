# Sift — 진행 장부

> **현재 진행 중인 이슈 1개의 구현 단계**만 담는다.
> 작업 목록의 원본은 GitHub 이슈 본문의 `## TODO`다. 태스크 후보는 [TASKS.md](./TASKS.md), 현재 상태는 [STATE.md](./STATE.md)를 따른다.

## 진행 중

### [#55 Subscriber/Subscription 도메인 및 구독 API](https://github.com/siftnews/sift-api/issues/55)

- [x] 이슈·브랜치 생성 및 API 계약 확정
- [x] subscriber·subscription Liquibase 스키마 초안 추가
- [x] Subscriber/Subscription 도메인·서비스·영속 어댑터 초안 추가
- [x] Content named interface를 통한 활성 토픽 조회 연결
- [x] REST 어댑터와 경계 오류 응답 추가
- [x] 웹·영속 통합 검증·문서 검증
- [ ] PR #56 리뷰·CI 확인 및 병합



## 최근 완료

### [#52 Atom 피드 파싱 검증 및 네이버 D2 소스 추가](https://github.com/siftnews/sift-api/issues/52)

- [x] Atom 피드 픽스처와 `RssFeedAdapter` 회귀 테스트를 추가했다.
- [x] 네이버 D2 Atom URL의 실제 응답을 확인했다.
- [x] `SourceSeedData`에 네이버 D2를 추가하고 시더 멱등성 기대값을 10개로 갱신했다.
- [x] 전체 테스트·문서 검증 및 PR #54 병합을 완료했다.
