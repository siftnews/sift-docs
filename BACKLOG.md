# Sift — 진행 장부

> **현재 진행 중인 이슈 1개의 구현 단계**만 담는다.
> 작업 목록의 원본은 GitHub 이슈 본문의 `## TODO`다. 태스크 후보는 [TASKS.md](./TASKS.md), 현재 상태는 [STATE.md](./STATE.md)를 따른다.

## 진행 중

### [#52 Atom 피드 파싱 검증 및 네이버 D2 소스 추가](https://github.com/siftnews/sift-api/issues/52)

- [x] Atom 피드 픽스처와 `RssFeedAdapter` 회귀 테스트를 추가했다.
- [x] 네이버 D2 Atom URL의 실제 응답을 확인했다.
- [x] `SourceSeedData`에 네이버 D2를 추가하고 시더 멱등성 기대값을 10개로 갱신했다.
- [ ] 전체 테스트·문서 검증 및 PR 생성.
