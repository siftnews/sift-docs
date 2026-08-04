# Sift — BACKLOG

> **현재 진행 중인 이슈 1개의 구현 단계**를 담는다. 그 이상은 담지 않는다.
> 작업 목록의 **원본은 GitHub 이슈 본문의 `## TODO`**다 (D-035) — 조율 세션이 이슈를 발행할 때 함께 쓰고, 구현 세션은 그것을 따라간다.
> 태스크 트리(이슈 후보)는 [TASKS.md](./TASKS.md) · 진행 상태는 [STATE.md](./STATE.md) · 역할 분담은 [HARNESS §0.8](./HARNESS.md)
> 표기: `[ ]` 대기 · `[~]` 진행 · `[x]` 완료 · `[G]` 게이트(사람 검증 필요) · 분류: `(W)` 워크플로우 · `(A)` 에이전트

---

## 진행 중

_없음 — 다음 이슈가 발행되면 여기에 분해한다._

---

## 완료된 분해 (요약 — 상세는 이슈·PR·git 히스토리)

> D-023의 "참조 강등" 적용. 각 이슈 본문과 PR diff가 원본이고, 여기에는 **되짚을 가치가 있는 것만** 남긴다.

### Phase 0 — 운영 골격 · 프로젝트 골격 · 골든패스

| 이슈 | 내용 | 결과 |
|---|---|---|
| [#1](https://github.com/siftnews/sift-api/issues/1) | `common` 모듈 · Modulith 경계 선언 · `ApplicationModules.verify()` · Testcontainers 베이스 | PR #2 |
| [#4](https://github.com/siftnews/sift-api/issues/4) | Source·Article 도메인 + URL 정규화 불변식 | PR #5 |
| [#8](https://github.com/siftnews/sift-api/issues/8) | CrawlSources 유스케이스 (fake 포트 단위 테스트) | PR #9 |
| [#10](https://github.com/siftnews/sift-api/issues/10) | Source 영속 어댑터 (Testcontainers) | PR #11 |
| [#12](https://github.com/siftnews/sift-api/issues/12) | RssFeedAdapter (RSS 픽스처 4케이스) | PR #13 |
| [#14](https://github.com/siftnews/sift-api/issues/14) | collectionJob 배치 + `CollectionMetricsListener` | PR #15 |

- ⚠️ **프로토콜 이탈 기록**: #10·#12는 BACKLOG 분해 없이 착수했다(§0.7 미준수, 2회). #14부터는 착수 전 분해를 지켰다.
- 부수 수정: `AbstractIntegrationTest`의 Postgres 컨테이너를 `@Container`(클래스별 stop/restart) → **static 싱글톤**으로 변경 — 통합 테스트가 2개 이상일 때 컨테이너 재시작으로 포트가 바뀌며 Spring 컨텍스트 캐시가 옛 포트를 참조해 연결이 실패했다.

### Phase 1 — 선별 (M2)

| 이슈 | 내용 | 결과 |
|---|---|---|
| [#17](https://github.com/siftnews/sift-api/issues/17) | Topic 도메인 + dev/ai/econ 시드 | PR #18 |
| [#19](https://github.com/siftnews/sift-api/issues/19) | Normalize 컷 + Jaccard 클러스터링 | PR #20 |
| [#21](https://github.com/siftnews/sift-api/issues/21) | 재실행 멱등성 — 윈도우 후보 + 상태 교체 (D-031) | PR #22 |
| [#23](https://github.com/siftnews/sift-api/issues/23) | 소스 9종 확정 + `SourceSeeder` + **e2e 게이트 해소** | PR #24 |
| [#25](https://github.com/siftnews/sift-api/issues/25) | 토픽 필터 + 4항목 가중합 + `article_score` | PR #26 |
| [#27](https://github.com/siftnews/sift-api/issues/27) | Rank & Select — 소스 쏠림 감점 + `issue`/`issue_item` | PR #28 |
| [#29](https://github.com/siftnews/sift-api/issues/29) | Source named interface + 실 어댑터 배선 | PR #30 |
| [#31](https://github.com/siftnews/sift-api/issues/31) | selectionJob 조립 + 일일 트리거 | PR #32 |
| [#33](https://github.com/siftnews/sift-api/issues/33) | collectionTrigger — 상시 기동 | PR #34 |
| [#35](https://github.com/siftnews/sift-api/issues/35) | 발행 이슈가 비는 시간대 결함 | PR #36 |
| [#37](https://github.com/siftnews/sift-api/issues/37) | URL 정규화 — 추적 파라미터만 제거 | PR #38 |
| [#39](https://github.com/siftnews/sift-api/issues/39) | TopicSeeder 인바운드 어댑터 이동 | PR #40 |

---

## 되짚을 가치가 있는 것

### e2e 게이트가 잡아낸 수집 결함 5건 (#23)

첫 실행은 9개 소스 중 **4개만** 수집됐고, 원인이 전부 코드 결함이었다. **시드만 넣고 끝냈으면 모두 묻혔을 것들이다.**

1. **시더가 배치 Job보다 늦게 실행** — `JobLauncherApplicationRunner`(order 0) vs `@Order` 없는 러너(`LOWEST_PRECEDENCE`) → 첫 기동은 항상 소스 0건
2. **`article` 문자열 컬럼이 전부 `varchar(255)`** — 본문 255자 초과 기사 하나가 그 소스 전체를 실패시켰다. `body`→`text`, url 2048, title 1024 (*수정 후 실측 최대 본문 31,838자*)
3. **`saveNew`가 입력 리스트 내 중복을 못 걸렀다** — DB 기존분만 필터해 한 chunk 내 중복 시 UNIQUE 위반 → 소스 skip
4. **`entry.getDescription()` null 방어 없음** — description 없는 피드(한국경제)·`content:encoded`만 있는 피드(토스)에서 NPE
5. **skip이 조용했다** — `.faultTolerant().skip(Exception.class)`에 `SkipListener`가 없어 어떤 소스가 왜 빠졌는지 로그가 없었다(Job status는 `COMPLETED`). `CollectionSkipListener`를 넣고 나서야 4번이 드러났다

### 회귀 방어를 실증하는 방식 (#35·#37에서 확립)

새 테스트가 정말 그 결함을 잡는지 확인하려고 **옛 구현을 일시 주입해 돌린다.**
- #35: 새 테스트 2건만 실패, 나머지 178건 통과 → 기존 테스트로는 못 잡던 결함임이 증명됨
- #37: 새 테스트 8건만 실패, 기존 19건 통과 → dedup 오병합의 실재가 증명됨

### 미반영으로 남긴 것

- **`Fake*Port` 중복 추출** (PR #15 CodeRabbit Trivial) — 테스트 4파일의 fake를 `adapter/in/batch/support`로. 반영 여부는 👤
- **`NormalizeDedupSummary`에 해제 건수(cleared) 추가** (#21) — 재실행 상태 교체량이 안 보인다. 측정 우선 원칙 5 관점에서 재검토 대상
- **`source.url` 유니크 제약의 버전드 마이그레이션** (PR #24 CodeRabbit ⑥) — Liquibase 도입 태스크로 이관
