# D-022 · `Source.markCrawled()` 영속 반영 = 전용 포트 (2026-07-13)

> [ADR 인덱스](README.md) · 결정 ID: D-022

- **결정**: `UpdateSourcePort.markCrawled(sourceId, at)` outbound port를 신설해 `SourcePersistenceAdapter`가 구현한다. `CrawlSourcesService`는 도메인 객체의 `markCrawled()` 호출과 별개로 이 포트를 호출해 영속화한다(MVP-DESIGN §6 열린 질문 해결).
- **이유**: `CrawlSourcesService`는 트랜잭션 경계를 갖지 않는 순수 애플리케이션 서비스이고, `LoadActiveSourcesPort`가 반환하는 `Source`는 이미 영속 컨텍스트에서 분리된 도메인 POJO라 dirty checking으로 자동 반영되지 않는다. `SaveArticlePort`와 동일한 관용구(전용 outbound port)를 따르는 것이 헥사고날 경계를 지키면서 가장 단순하다.
- **비고**: 어댑터 내부는 `@Transactional` + JPA dirty checking으로 반영(명시적 `save()` 호출 불필요). ERD(§2)의 `source.trust_score` 컬럼은 이번 영속 어댑터 범위에서 제외 — 현재 도메인 모델(M1-2)에 필드가 없고 사용하는 유스케이스도 없어 컬럼을 추가하지 않음(YAGNI). M2 스코어링 구현 시 도메인·엔티티에 함께 추가 검토.
