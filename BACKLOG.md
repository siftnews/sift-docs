# Sift — 진행 장부

> **현재 진행 중인 이슈 1개의 구현 단계**만 담는다.
> 작업 목록의 원본은 GitHub 이슈 본문의 `## TODO`다. 태스크 후보는 [TASKS.md](./TASKS.md), 현재 상태는 [STATE.md](./STATE.md)를 따른다.

## 진행 중

### [#50 Liquibase 마이그레이션 도입](https://github.com/siftnews/sift-api/issues/50)

- [x] Liquibase 의존성·master changelog·애플리케이션/Batch 스키마 changeset을 추가했다.
- [x] local·test 자동 DDL을 끄고 Hibernate 검증 모드로 전환했다.
- [x] URL 재정규화 백필을 changeset으로 옮기고 기동 러너·전용 포트·테스트를 제거했다.
- [x] Spring Modulith `event_publication` 스키마를 포함하고 전체 테스트를 통과했다.
- [ ] PR 생성 및 CI 검증.
