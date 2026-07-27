# Sift — BACKLOG

> 잘게 쪼갠 작업 + 완료기준(DoD) + 분류. 루프는 **우선순위 최상단의 미완 작업**을 집어간다.
> 프로젝트 전체 태스크 트리(이슈 후보)는 [TASKS.md](./TASKS.md) — 여기는 **현재 진행 이슈의 구현 단계**를 담는다 (§0.7 계층).
> 표기: `[ ]` 대기 · `[~]` 진행 · `[x]` 완료 · `[G]` 게이트(사람 검증 필요)
> 분류: `(W)` 워크플로우 · `(A)` 에이전트 (§0.5)

## Phase 0 — 스캐폴딩 + 골든패스

### A. 운영 골격 (루프 기반)
- [x] 하네스 토대 — CLAUDE.md, settings.json
- [x] 루프 운영 골격 — STATE, BACKLOG, DECISIONS, 루프 프로토콜
- [x] 협업 흐름 규칙 — HARNESS §0.7 (이슈→PR→리뷰→병합)
- [x] 협업 역할 분담 확정 — GitHub 쓰기(이슈·push·PR·병합)는 사람, 초안·구현·커밋은 에이전트 (D-013 · **2026-07-17 D-026으로 개정**: 이슈·PR 생성·커밋·push도 에이전트)
- [x] MVP 태스크 트리 — [TASKS.md](./TASKS.md) 작성 (이슈 발행 원본)
- [x] main 브랜치 보호 규칙 설정 (사람) — sift-api 설정 확인 (2026-07-15, D-024 비고)
- ~~루트 `siftnews/` git 저장소화~~ — D-015로 취소 (루트는 로컬 전용, git은 하위 sift-* 레포만)
- [x] develop 처리 결정 — **develop 유지 확정 (2026-07-17, D-025)**: develop = 통합, main = 배포. 삭제 항목 폐기
- [x] git/GitHub 쓰기 위임 + 가드 훅 (2026-07-17, D-026) — settings 재편 + `git-gh-guard.sh` + HARNESS 개정 → **가드 훅은 폐지 (2026-07-19, D-027)**: main 방어는 브랜치 보호로, 형식은 컨벤션 준수
- [x] 이슈 #1 push·PR — PR #2 병합 완료 (2026-07-06)
- ~~권한 조정(git push/gh 허용)~~ · ~~gh CLI 설치~~ — D-013으로 불필요 → gh CLI 설치·읽기 전용 (D-024) → **쓰기 위임으로 최종 개정 (D-026, 2026-07-17)**

### B. 프로젝트 골격 (코드 기초 틀) — 이슈 #1 `[chore] 프로젝트 골격 & 모듈 경계`
- [x] (W) `common` 모듈 — `BaseEntity`(+JPA Auditing), `Clock` 빈, `BusinessException` · **DoD**: 컴파일 ✅
- [x] (W) Modulith 모듈 경계 선언 — source/content/subscriber/delivery/tracking `package-info` + `allowedDependencies={"common"}` · **DoD**: 구조 생성 ✅
- [x] (W) 모듈 검증 테스트 `ApplicationModulesTest` — `ApplicationModules.verify()` · **DoD**: verify 통과 ✅
- [x] (W) 테스트 인프라 — testcontainers 의존성 + `AbstractIntegrationTest` 베이스 + `application-test.yaml` ✅
- [x] (W) 측정 베이스라인 골격 — Actuator + 배치 처리량·소요시간 로깅 · **M1-6(이슈 #14 → PR #15)에서 `CollectionMetricsListener`로 구현 완료** (2026-07-21)

### C. 골든패스 (collectionJob)
> **D-020 재편 (2026-07-07)**: 이 섹션의 구 분해(레이어 단위·DoD 컴파일)는 폐기 — TASKS M1-2~7(DDD inside-out, 단계별 테스트 DoD)로 대체.
> 이슈가 발행되면 해당 이슈의 구현 단계를 여기에 분해한다 (§0.7 계층).
- [x] M1-2 `[FEAT] Source 도메인 모델` — 이슈 #4 → PR #5 병합 완료 (2026-07-08)
- [x] M1-3 `[FEAT] CrawlSources 유스케이스` — 이슈 #8 → PR #9 병합 완료 (2026-07-11 · 완료된 분해 내역은 git 히스토리 — D-023)
- [x] M1-4 `[FEAT] Source 영속 어댑터` — 이슈 #10 → PR #11 병합 완료 (2026-07-16)
  - ⚠️ 프로토콜 이탈 기록: 이 이슈는 BACKLOG 분해 없이 구현됨 (§0.7 미준수) — 재발 방지는 D-023 후속(/harness-check)
- [x] M1-5 `[FEAT] RSS 수집 어댑터` — 이슈 #12 → PR #13 병합 완료 (2026-07-19 · 완료된 분해 내역은 git 히스토리 — D-023)
  - ⚠️ 프로토콜 이탈 기록: BACKLOG 분해 없이 착수, 소급 기록 (§0.7 미준수 — M1-4에 이어 2회째)
- [x] M1-6 `[FEAT] collectionJob 배치 + 측정 베이스라인` — 이슈 #14 → PR #15 병합 완료 (2026-07-21 · 착수 전 분해 ✅ 이탈 3회 방지 준수 · 완료된 분해 내역 (1)~(3)은 git 히스토리 — D-023)
  - [x] ~~[G] e2e 게이트 (👤) 미확인~~ — **해소 (2026-07-26, 이슈 #23)**: 실 RSS → `article` 258건 적재, 9개 소스 전부. 이 게이트를 여는 과정에서 수집 결함 4건이 드러났다(#23 섹션 참조)
  - 남은 결정: 배치 경로의 `markCrawled`(`UpdateSourcePort`)는 **미반영 상태로 병합** — 기본 제외 확정 vs 후속 이슈. 배치 ↔ 비배치 `CrawlSourcesService` 역할 경계와 함께 M1-7에서 정리
  - 미반영 리뷰(Trivial): 테스트 4파일의 `Fake*Port` 중복 → `adapter/in/batch/support` 패키지 추출 (선택 · PR 병합으로 코멘트가 묻혀 여기 기록)

## Phase 1 — M2-1 (#17 → PR #18) Topic 도메인 + 시드 3종 [병합 완료 2026-07-23]
> 브랜치 `feature/17-topic-domain-seed` (base develop). DDD inside-out, 단계별 테스트 DoD (D-020). 전체 테스트 통과 ✅
- [x] (W) Topic 도메인 POJO + 불변식 (content/domain, JPA·Spring 무의존 — D-009 분리) · **DoD**: 도메인 단위 테스트 8건 ✅ (slug 정규화·필수값, 컬렉션 기본, 검증 규칙)
- [x] (W) `topic` JpaEntity + 매핑 + JpaRepository (BaseEntity 상속, UNIQUE slug, 키워드/가중치 JSON 컬럼) · **DoD**: Testcontainers 통합 2건 ✅ (JSON 라운드트립·slug 유니크)
- [x] (W) dev/ai/econ 시드 — 프로파일 가드(`@Profile("!test")`) 시더, slug 멱등 · **DoD**: 시더 통합 테스트 1건 ✅ (3종 적재·재실행 멱등)
- [x] (W) `ApplicationModules.verify()` 통과 (content allowedDependencies 유지) ✅
> 비고: outbound 포트(`LoadTopicPort`)·영속 어댑터는 소비자(선별 유스케이스)가 생기는 후속 M2 태스크로 미룸 (YAGNI). 리뷰 반영 `0fb7499`(scoreThreshold 유한값·slug 로케일·컬렉션 null 검증).

## Phase 1 — M2-2 (#19 → PR #20) 선별 1/3 Normalize + Dedup [병합 완료 2026-07-23]
> 스택 브랜치 `feature/19-...` (base `feature/17`/PR #18 → develop 재지정 후 병합). D-030 적용. 전체 테스트 통과 ✅
- [x] (W) content 도메인 `CandidateArticle` + Normalize 컷(`ArticleNormalizer`: 언어·본문 최소 길이) · **DoD**: 도메인 단위 ✅
- [x] (W) Jaccard 유사도(`TitleSimilarity`) + 클러스터링(`DedupClusterer`: URL 1차 + Jaccard≥0.7 union-find, 최신 대표) · **DoD**: 도메인 단위 ✅
- [x] (W) `NormalizeDedupUseCase`/`Summary` + `LoadCandidateArticlesPort`·`UpdateArticleClusterPort` + `NormalizeDedupService` · **DoD**: fake 포트 서비스 단위 ✅
- [x] (W) `ApplicationModules.verify()` 유지 ✅
> 비고: Source named interface 실 어댑터·후보 조회/갱신 배선은 selectionJob 배치(M2-5)에서 연결(YAGNI). **CodeRabbit Major(재실행 멱등성) 수용 → 이슈 #21로 분리**, 나머지 지적은 해소.

## Phase 1 — M2-2 후속 (#21 → PR #22) normalizeDedup 재실행 멱등성 [병합 완료 2026-07-25]
> 브랜치 `feature/21-normalize-dedup-idempotency` (base develop). **범위·설계 D-031 확정** — fake 포트까지, 실 어댑터 배선은 M2-5 유지. 테스트 79건 통과·CI pass ✅
> 전제: 후보 로드가 **윈도우**여야 멱등성이 성립한다 — "신규만" 로드면 이전 실행에 묶인 기사가 후보에 없어 되돌릴 대상 자체가 없다.
- [x] (W) 후보 로드 범위 = 윈도우로 확정 — `loadCandidates(Instant from, Instant to)`, `[from, to)` 반열림 · **DoD**: 시그니처 변경 + fake 반영 ✅
- [x] (W) `clusterId = "c-" + min(memberIds)`로 변경 (`DedupClusterer`) — `representativeId`는 최신 `publishedAt` 기준 현행 유지 · **DoD**: `DedupClustererTest` 안정성 케이스 ✅
- [x] (W) 실행 단위 클러스터 상태 교체 (`NormalizeDedupService`) — 후보 전체를 `null`로 채운 뒤 클러스터 소속만 덮어써 컷 탈락 기사를 해제 · **DoD**: 서비스 단위 테스트 ✅
- [x] (W) `UpdateArticleClusterPort` 벌크 갱신 — `updateClusters(Map<Long, String>)`, `null` 값 = 해제. `TreeMap`으로 articleId 오름차순 고정(재현성·lock order) · **DoD**: fake 포트 반영 ✅
- [x] (W) 순차 재실행 회귀 테스트 (`NormalizeDedupServiceTest`) · **DoD**: ① 탈락 시 clusterId 비워짐 ② 신규 합류 후 기존 멤버 clusterId 불변 ③ 동일 입력 동일 결과 ✅
- [x] (W) `ApplicationModules.verify()` 유지 ✅
- [ ] (선택) `NormalizeDedupSummary`에 해제 건수(cleared) 추가 — **미반영**. 측정 우선 원칙 5 관점에서 재실행 상태 교체량이 안 보이므로 M2-5 배선 때 재검토
> 자체 리뷰 교정 4건: 회귀 테스트가 실제로는 회귀를 못 잡던 결함(픽스처에서 최소 id와 대표가 우연히 동일) · 후보 0건 시 빈 맵 → `IN ()` SQL 오류 가능 → 조기 반환 + 계약 명시 · `from`/`to` 무검증이 파라미터 실수를 "후보 없음" 성공 로그로 위장 · `CandidateArticle.articleId` null 방어.
> 미포함(M2-5로): Source named interface 실 어댑터, Testcontainers 통합 검증(`[from, to)` 실 경계 포함), 윈도우 크기 운영값 확정.
> ⚠️ 이 PR은 **CodeRabbit 리뷰 없이 병합**됨(`Review rate limited` — 체크는 pass 표시). STATE 정정 참조.

## Phase 1 — #23 소스 RSS URL 확정 + source 시드 [PR #24 리뷰 대기]
> 브랜치 `feature/23-source-rss-seed` (base develop). 착수 전 분해 ✅ (§0.7 절차 3 — M1-4·M1-5 이탈 재발 방지)
> 소스 9종은 2026-07-26 실 응답으로 검증(HTTP 200 + RSS 루트 태그 + 최근 갱신) — 목록·제외 사유는 [이슈 #23](https://github.com/siftnews/sift-api/issues/23) 본문이 원본.
- [x] (W) `Source.create` 팩토리 + 불변식 — 피드 url은 **정규화 없이 원본 보존**(쿼리로 피드를 구분하는 사이트가 있어 `UriNormalizer` 재사용 불가) · **DoD**: 도메인 단위 11건 ✅
- [x] (W) `SourceSeedData` — 소스 9종 정의 (dev 3 / ai 3 / econ 3, 한국어 4 · 영어 5)
- [x] (W) `SourceSeeder` — `@Profile("!test")`, `url` 기준 멱등 · **DoD**: Testcontainers 통합 3건 ✅
- [x] (W) `ApplicationModules.verify()` 유지 ✅
- [x] (W) `sift-api/docs/MVP-DESIGN.md` §1 소스 시드 표를 확정 URL로 갱신 ✅
- [x] [G] **e2e 게이트 해소 ✅** — 실 RSS 수집 → `article` **258건 적재, 9개 소스 전부**(`read=9 write=9 skip=0`). **M1-6 잔여 게이트(BACKLOG C `[G]`) 닫힘**

**e2e가 잡아낸 수집 결함 4건** — 첫 실행은 9개 중 4개 소스만 수집됐고, 원인이 전부 코드 결함이었다. 시드만 넣고 끝냈으면 모두 묻혔을 것들이다.
- [x] ① **시더가 배치 Job보다 늦게 실행** — `JobLauncherApplicationRunner`(order 0) vs `@Order` 없는 러너(`LOWEST_PRECEDENCE`) → 첫 기동은 항상 소스 0건. 두 시더에 `@Order(HIGHEST_PRECEDENCE)`
- [x] ② **`article` 문자열 컬럼이 전부 `varchar(255)`** — 본문 255자 초과 기사 하나가 그 소스 전체를 실패시킴. `body`→`text`, url 2048, title 1024. *수정 후 실측 최대 본문 31,838자*
- [x] ③ **`saveNew`가 입력 리스트 내 중복을 못 거름** — DB 기존분만 필터해 한 chunk 내 중복 시 UNIQUE 위반 → 소스 skip
- [x] ④ **`entry.getDescription()` null 방어 없음** — description 없는 피드(한국경제)·`content:encoded`만 있는 피드(토스)에서 NPE → 소스 전체 실패
- [x] ⑤ **skip이 조용했다** — `.faultTolerant().skip(Exception.class)`에 `SkipListener`가 없어 어떤 소스가 왜 빠졌는지 로그가 없었다(Job status는 `COMPLETED`). `CollectionSkipListener` 추가 후에야 ④가 드러남
- [x] 부수: `docker-compose.yaml`이 `postgres:16`을 가리켜 Testcontainers(`16-alpine`)와 불일치 → 이미지 중복 다운로드 + 테스트/로컬 DB 상이. `16-alpine`으로 통일

**리뷰 반영 (CodeRabbit 4건 — 2026-07-26~27)**
- [x] ① skip 로그에 `sourceId` 추가 · ④ url 파싱 예외 cause 보존 (`2c365a3`)
- [x] ② **시더가 out 어댑터에서 JPA를 직접 호출**(헥사고날 위반) → `adapter.in.bootstrap`으로 이동, `SeedSourcesUseCase`→`SaveSourcePort` 경유 (`e366fff`)
- [x] ③ **`existsByUrl`→`save`가 원자적이지 않음** → `INSERT … ON CONFLICT (url) DO NOTHING`. 신규 DB 2회 기동 실측(신규 9건 → 0건, 예외 없음) (`e366fff`)
- [ ] ⑥ **`source.url` 유니크 제약이 버전드 마이그레이션에 없음** [보류 — 후속] : `ON CONFLICT (url)`은 제약이 없으면 예외인데 현재는 `ddl-auto: update` 의존이고, **update는 기존 테이블에 제약을 소급 생성하지 않는다**(e2e 컨테이너 `sift` DB에서 확인). 신규 DB·CI·Testcontainers는 생성 시점에 붙어 무관하고 배포 환경도 아직 없어, Liquibase 도입(TASKS)으로 정식 해소한다 — 스레드는 미해결로 남김(D-033)

> **후속 이슈로 분리한 것** — ⓐ **AI타임스 50개 중 1건만 적재**: `?idxno=213188`처럼 쿼리로 기사를 구분하는데 `UriNormalizer`가 쿼리를 버려 전부 같은 url로 정규화된다. 한국 언론사 다수가 쓰는 패턴이라 영향이 크지만 **기사 동일성 판정 규칙 변경**이라 dedup(D-030·D-031)·선별까지 번짐 ⓑ **collectionJob 상시 트리거 부재** — `spring.batch.job.enabled: false`인데 트리거가 없어 `bootRun`으로 Job이 안 뜬다(이번엔 `--spring.batch.job.enabled=true` 일회 기동). **M1-6 e2e 게이트가 미해결로 남아 있던 실질적 원인** ⓒ Atom 피드 파싱 미검증(네이버 D2 등)

## Phase 1 — #25 선별 2/3 Filter + Score [PR #26 리뷰 대기]
> 브랜치 `feature/25-selection-filter-score` — **base가 `feature/23-source-rss-seed`인 스택 PR**. #24 병합 후 base를 `develop`으로 재지정해야 CodeRabbit이 리뷰한다 (👤).
- [x] (W) `CandidateArticle`에 `sourceId`·`category`·`dedupClusterId` 추가 — 필터·소스점수·화제성이 셋 다 필요로 하는데 뷰에 없었다 · **DoD**: 기존 normalizeDedup 회귀 통과 ✅
- [x] (W) `TopicFilter` — 언어 → 카테고리 → 제외어 → 포함어 순(싼 것부터) · **DoD**: 도메인 단위 11건 ✅
- [x] (W) `ArticleScorer` + `ScoreBreakdown`·`ScoreWeights` — 4항목 가중합 · **DoD**: 도메인 단위 15건, 근거만으로 점수 재현 ✅
- [x] (W) `article_score` 영속 — `(article_id, topic_id)` UNIQUE + `ON CONFLICT DO UPDATE` · **DoD**: Testcontainers 3건(JSON 왕복·재실행 시 행 안 늘어남) ✅
- [x] (W) `ScoreArticlesUseCase`/`Service` · **DoD**: fake 포트 단위 7건 ✅
- [x] (W) `ApplicationModules.verify()` 유지 ✅ · `docs/SELECTION.md` §2.4·`MVP-DESIGN.md` §4 갱신 ✅
- [x] 전체 테스트 **132건 통과** ✅

**설계에서 확정한 것**
- **단계 순서**: 화제성 집계(필터 전, 토픽 무관) → 토픽 필터 → 클러스터당 대표 1건 → 스코어. 순서를 바꾸면 ① 화제성이 토픽마다 달라지거나 ② 대표가 필터에 걸릴 때 통과 가능했던 같은 클러스터 기사까지 잃는다
- **D-032 불변식 (2)를 구조로 강제** — 클러스터 크기를 별도 집계 쿼리가 아니라 `loadCandidates`가 준 윈도우 안에서 센다(`ClusterWindow`). 대상이 윈도우를 벗어날 방법이 없어진다
- **대표 선정 규칙 단일화** — `DedupClusterer`에 있던 규칙을 `RepresentativeRule`로 빼 스코어링과 공유. 갈라져도 예외가 안 나고 다른 기사가 실릴 뿐이라 발견이 늦다
- **재실행 멱등성을 처음부터** — #21에서 뒤늦게 후속 이슈로 뺐던 전례를 반영

**👤 확인 대기**
- `sourceScore`를 **중립 상수 1.0**으로 뒀다 — `source.trust_score`는 M1-4에서 범위 제외되며 "M2 스코어링에서 재검토"로 남아 있던 항목(MVP-DESIGN §2·§6). 항목만 배선하고 값은 상수로 두는 편이 되돌리기 쉽다고 판단
- 키워드 매칭이 **대소문자 무시 부분 문자열** — 한국어 조사 때문인데 영어 `Java`→`JavaScript` 과매칭을 수용. breakdown 로그로 오탐 빈도 본 뒤 재판단

## Phase 1 이후 — [TASKS.md](./TASKS.md) M2~M4 참조
- 이슈가 발행되면 해당 태스크를 이 파일에 구현 단계로 분해해 루프를 돈다 (§0.7 절차 4)
- Sift용 `code-review` · `create-branch` 스킬 박제는 **M2로 이동** (D-029 — 2번째 유스케이스에서 공통 패턴 추출, D-012 시점 개정)
