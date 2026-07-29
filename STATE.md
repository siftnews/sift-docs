# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-27

## 진행 중 2 — #25 → PR #26 (선별 2/3 Filter + Score)
- **이슈 #25 → PR #26 (2026-07-27)** — 토픽 필터 + 4항목 가중합 스코어링 + `article_score`(breakdown JSON) 구현 완료, **전체 132 tests 통과**. 브랜치 `feature/25-selection-filter-score`는 **#24 위 스택** — base가 `feature/23-source-rss-seed`라 **#24 병합 후 base를 develop으로 재지정해야 CodeRabbit이 리뷰한다 (👤)**. CodeRabbit 감시 Monitor 가동
- 범위는 #19·#21과 동일하게 content 도메인·애플리케이션까지 — Source named interface 실 어댑터 배선은 M2-5 유지(`LoadCandidateArticlesPort`는 fake로만 검증)

## 진행 중 3 — #27 → PR #28 (선별 3/3 Rank & Select)
- **이슈 #27 → PR #28 (2026-07-27)** — threshold 컷 → 점수 내림차순 → 소스 쏠림 감점 → 상위 `maxItems` → `issue`(DRAFT)+`issue_item` 생성. **전체 158 tests 통과**. `feature/27-selection-rank-select`는 **#26 위 스택**(base=`feature/25-...`) — 아래 PR들이 병합돼야 base를 develop으로 돌릴 수 있다 👤. **M2 공통 DoD "시드 토픽으로 이슈 1건 생성"의 코드 경로 완성** (실배선 확인은 M2-5)
- **전면 MMR을 넣지 않았다** — SELECTION §2.5의 다양성 요구 중 *같은 클러스터 쏠림*은 #25가 이미 클러스터당 대표 1건으로 줄여 이 단계에 도달하지 않는다. 남은 *소스 쏠림*만 곱셈 감점(×0.7)으로 처리. §6 열린 질문은 유지
- ⚠️ **ERD 변경 2건 (👤 확인 필요)** — ① `issue.run_date` + `UNIQUE(topic_id, run_date)` 추가: 기존 스키마엔 "몇 일자 호인가"가 없어 재실행 시 같은 날 호가 두 개 생길 수 있었다 ② `article_score.source_id` 비정규화: 소스 쏠림 완화에 필요한데 article은 Source 소유(D-018)라 조인 불가

## 진행 중 4 — ~~#29 → PR #30~~ **병합 완료 ✅ (2026-07-29, develop `8bc6c67`)**
- **병합됨** — `com.siftnews.source.api`에 `@NamedInterface("article-catalog")` 노출(`ArticleCatalog`+`ArticleCandidate`), content `allowedDependencies`에 `source :: article-catalog` 추가, `LoadCandidateArticlesPort`·`UpdateArticleClusterPort` 실 어댑터. **전체 170 tests 통과**. CodeRabbit 지적 1건은 **반박**(아래) — 스레드는 미해결로 남김(D-033)
- ⚠️ **`article.dedup_cluster_id` 컬럼이 애초에 없었다** — MVP-DESIGN §2 ERD엔 처음부터 있었지만 엔티티에 만들어진 적이 없다. 포트가 전부 fake라 아무도 쓰지 않아 드러나지 않았던 것. #19부터 미뤄 온 fake를 걷어내며 발견
- ✅ **PR #22 확인 항목 ② 해소** — `[from, to)` 경계를 Testcontainers로 실검증(from 포함·to 미포함). `created_at`은 JPA Auditing이 채우므로 저장 후 네이티브 UPDATE로 경계 시각을 만들어 확인
- ❌ **반박한 지적 (PR #30, Major)**: "`dedup_cluster_id` 마이그레이션을 함께 배포하라". **실측으로 반박** — 기존 `sift` DB(258건, 컬럼 없음)에 `ddl-auto: update`로 기동하니 `dedup_cluster_id varchar(64)`가 **자동 생성**됐다. 다만 같은 기동이 `ON CONFLICT (url)`에서 실패해(유니크 제약 미소급) **마이그레이션이 실제로 필요한 대상은 제약·인덱스·백필이지 단순 컬럼 추가가 아님**이 함께 확인됐다 → Liquibase 태스크 범위에 반영
- **설계 판단**: 경계를 건너는 타입은 애그리거트가 아니라 `ArticleCandidate` record(계약) · `dedup_cluster_id`는 **도메인 `Article`에 넣지 않음**(애그리거트가 이 값으로 아무 결정도 안 해 빈 필드가 됨, 이슈 TODO에서 변경) · named interface는 **서비스가 구현**(어댑터가 port.in을 구현하면 의존 방향이 뒤집힘 — PR #24 시더와 같은 이유)

## 진행 중 5 — #31 → PR #32 (selectionJob 배치 + 트리거)
- **이슈 #31 → PR #32 (2026-07-29)** — Step 3종(tasklet) 조립 + `@Scheduled` 일일 트리거 + 계측. **전체 174 tests 통과**. `feature/31-selection-job`은 **#30 위 스택**
- 🎯 **M2 공통 DoD 충족** — fake 없이 실 DB에 토픽·기사를 심고 세 Step을 돌려 **이슈 1건 + 항목 2건 생성** 확인. #29의 실 배선 덕에 가능해진 검증
- **확정한 운영값** — 윈도우 **24h**(발행 주기·`recency_half_life_hours` 기본값과 정렬), 트리거 **06:00 Asia/Seoul**. D-031·D-032가 "M2-5에서 확정"으로 남긴 항목이었다. 둘 다 `sift.selection.*` 설정값
- **D-032 불변식 (3)을 트리거가 구조로 보장** — 윈도우를 트리거가 한 번만 계산해 전 토픽 잡에 주입한다. Step은 윈도우를 만들지 않고 받기만 하므로 토픽 간 어긋남이 불가능
- ⚠️ **`@EnableScheduling`이 없었다** — `spring.batch.job.enabled=false`인데 스케줄 활성화도 안 돼 있어, 이대로면 배치가 영영 돌지 않는다. 수집 쪽 `collectionTrigger` 후속 이슈도 이 설정을 전제로 한다
- **정정**: MVP-DESIGN §3③의 `scoreStep (chunk)` → **tasklet**. 선별은 윈도우 전체 재계산이 멱등성의 전제라(D-031) 아이템 단위로 쪼갤 수 없다

## 현재 Phase
**Phase 1 (선별) 진행 중** — 골든패스 코드 경로는 M1-6에서 완주, M1-7 박제 태스크는 해체(D-029). M2-1·M2-2(+#21 후속) 병합 완료, 소스 시드(#23) 진행 중

## 지금 (in progress) — #23 → PR #24 리뷰 대기
- **이슈 #23 → PR #24 (2026-07-26~27)** — 소스 9종 시드 + `SourceSeeder` 구현·검증 완료(전체 **94** tests 통과). **M1-6 e2e 게이트 해소 ✅** — 실 RSS → `article` **258건, 9개 소스 전부**(`read=9 write=9 skip=0`). CodeRabbit 감시 Monitor 가동
- **CodeRabbit 리뷰 4건 전부 처리·resolve 완료 (2026-07-27, `e366fff`)** — ①skip 로그 식별자·④예외 cause는 `2c365a3`, **②시더 위치(헥사고날 위반)·③원자적 저장은 `e366fff`**로 반영. ②③은 뿌리가 같아 함께 고침: 시더를 `adapter.in.bootstrap`으로 옮겨 `SeedSourcesUseCase`→`SaveSourcePort` 경유로 바꾸고, 저장을 `INSERT … ON CONFLICT (url) DO NOTHING`(네이티브)으로 교체해 동시·재기동 멱등 확보. 신규 DB 2회 기동으로 실측 검증(**신규 9건 → 신규 0건, 예외 없음**). 스레드 4개 전부 resolved (D-033)
- ⚠️ **e2e가 수집 결함 4건 + 관측 공백 1건을 잡아냈다** — 첫 실행은 9개 중 4개 소스만 수집. ① 시더가 배치보다 늦게 실행 ② `article` 컬럼이 전부 `varchar(255)`(본문 포함) ③ `saveNew`가 입력 내 중복 미필터 ④ description 없는 피드에서 NPE ⑤ `SkipListener` 부재로 skip 사유가 로그에 없었음(Job은 `COMPLETED`). **게이트를 실제로 열어보지 않았으면 전부 묻혔을 결함들** — e2e 게이트의 가치가 입증된 사례
- ⚠️ **세션 역할 분담 개정 (2026-07-26)**: 07-25의 "루프 문서 쓰기 = 문서 세션 전담"은 **이 세션이 문서 쓰기까지 겸임**하는 것으로 사용자 확정. 단 쓰기 전 `sift-docs` **fetch 필수** — 07-26에 fetch 없이 로컬 사본만 보고 "문서 4일 뒤처짐·D-031 미등재"로 오진단해 중복 커밋 2개를 폐기한 사고 발생 (아래 "정정")

## 다음 액션 (next)
- 👤 **PR #24 병합** — CodeRabbit 지적 4건 반영·resolve 완료. **증분 리뷰로 5번째 지적(Minor) 도착** — `source.url` 유니크 제약을 버전드 마이그레이션에 넣으라는 것으로, 아래 ⚠️와 같은 사안이다. Liquibase 도입은 별도 태스크로 등록하고 **스레드는 미해결로 남겼다**(보류는 resolve하지 않음 — D-033). 병합 판단은 사용자. 확인 부탁 2건: ① `article` 컬럼 길이 관행값(url 2048 / title 1024) ② description fallback으로 `content:encoded`를 쓰면 HTML이 통째로 들어옴(실측 31,838자) — 본문 정제는 선별 Normalize 몫으로 남김
- ⚠️ **기존 로컬 DB는 `source.url` 유니크 제약이 없어 새 시더가 기동 실패한다** — `ddl-auto: update`는 이미 있는 테이블에 유니크 제약을 소급 생성하지 않는다(e2e 컨테이너 `sift` DB에서 확인). `ON CONFLICT (url)`은 제약이 없으면 예외다. 신규 DB·CI·Testcontainers는 테이블 생성 시점에 제약이 붙어 무관. 기존 로컬 DB를 계속 쓰려면 `ALTER TABLE source ADD CONSTRAINT … UNIQUE (url)` 1회 필요 — Liquibase 전환 시 정식 해소
- 🤖 **#23 병합 후 후속 이슈 4건 발행** — ⓐ `[FIX] 쿼리로 기사를 구분하는 소스의 URL 정규화`(AI타임스 50→1건, **기사 동일성 규칙 변경이라 dedup·선별 전제에 영향 — 설계 결정 필요 👤**) ⓑ `[FEAT] collectionTrigger` ⓒ `[FEAT] Atom 피드 파싱 검증` ⓓ `[REFACTOR] TopicSeeder를 인바운드 어댑터로 이동`(#24에서 source만 정리, content는 범위 밖). 넷 다 TASKS M2에 등록함
- 🤖 **M2 잔여 선별 태스크** — ~~선별 2/3~~(#25 → PR #26 리뷰 대기) → ~~3/3(Rank & Select)~~(#27 → PR #28) → ~~named interface 실 배선~~(#29 → PR #30) → ~~selectionJob 배치 + selectionTrigger~~(#31 → PR #32) — **M2 선별 코드 경로 완주**
- 👤 **#25 열린 결정 2건** — ① `sourceScore`를 중립 상수 1.0으로 둠(`source.trust_score` 재검토 시점이 지금) ② 키워드 매칭이 대소문자 무시 부분 문자열(영어 과매칭 수용). 둘 다 되돌리기 쉬운 쪽으로 잡았고 방향 지시가 있으면 후속에서 변경
- 👤 **CodeRabbit 리뷰 공백 확인** — PR #22는 `Review rate limited`로 **외부 리뷰 없이 병합**됐다(체크는 pass 표시). 병합 전 자체 리뷰 패스로 대체됐으나, rate limit이 재발하면 리뷰 게이트가 조용히 비는 구조 (아래 "정정" 참조)
- 👤 **PR #22 확인 부탁 2건** — ① `updateClusters(Map)` 단일 인자 vs `updateClusters(assigned, cleared)` 2-인자 분리(null 값 계약이 부담이면 재검토) ② `[from, to)` 경계 검증은 fake 계층에서 pass-through에 그침 — 실 경계 검증은 M2-5 Testcontainers 몫
- 👤 **M1-6 e2e 게이트 확인** — `bootRun`으로 실제 RSS → `article` 적재 + 측정 로그 확인. PR #15는 병합됐으나 이 실환경 게이트는 미확인 상태로 남아 있다 (BACKLOG C의 `[G]` 항목)
- 👤 **markCrawled 배치 반영 결정** — 기본 제외 확정 vs 후속 이슈 (M1부터 계류 — D-029 잔여물 ②)
- 🤖 **SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환** — sift-api 이슈→PR (D-029 잔여물 ③)
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)

## 정정 (사람 재검증·수정 흔적)
- **(2026-07-26) 세션이 `sift-docs`를 fetch하지 않고 문서 상태를 오진단** — `sift-api`만 fetch한 채 로컬 `sift-docs` 사본을 읽고 "루프 문서가 4일 뒤처졌고 D-031이 DECISIONS에 없는 댕글링 참조"라고 사용자에게 보고했으나, **실제로는 원격이 최신이었고 로컬 클론이 낡은 것**이었다(원격에는 D-031·D-032 모두 존재). 그 오진단 위에서 D-031을 중복 작성해 커밋 2개(`d2d9cd7`·`8b929c9`)를 만들었고, push 단계에서 non-fast-forward로 발각돼 `git reset --keep origin/main`으로 폐기했다. 원인은 2대 머신(맥북↔Mac Studio) 공유 환경에서 **fetch 없이 로컬 파일을 진실로 취급**한 것 — 메모리에 재발 방지 기록(`fetch-before-reading-loop-docs`).
- **(2026-07-25) `.coderabbit.yaml` path_filter가 헥사고날 `out` 패키지를 리뷰에서 제외하고 있었음** — `!**/out/**`(IntelliJ 빌드 산출물 의도)가 `application/port/out`·`adapter/out`까지 매칭시켜, **아웃바운드 포트·영속/피드 어댑터가 CodeRabbit 리뷰 사각지대**였다. PR #22에서 `!out/**`로 수정(`27d8f8b`). 소급 영향: M1-4 Source 영속 어댑터·M1-5 RssFeedAdapter·M2-2 포트 2종 등 `out` 경로 코드는 리뷰를 안 받았을 가능성이 높다 — 필요하면 사후 리뷰 요청은 👤 (comment는 사람 영역, D-026)
- **(2026-07-25) PR #22는 CodeRabbit 리뷰 없이 병합됨** — 체크 결과가 `pass`로 표시되지만 실제 내용은 `Review rate limited`. **rate limit이 걸려도 체크는 통과로 뜨므로 리뷰 게이트가 조용히 비는 구조**다. #22는 구현 세션의 자체 리뷰 패스(회귀 테스트가 회귀를 못 잡던 결함·빈 맵 SQL 오류·from/to 무검증 발견)로 대체됐다. 👤 재발 시 대응 방침 결정 필요
- **(2026-07-25) 로컬 feature 브랜치 "정리 완료" 기록은 부정확** — 아래 최근 완료 항목들의 "로컬 feature 브랜치 정리 완료" 표현과 달리, 실제 로컬에는 병합 끝난 `feature/4-source-domain`·`feature/8-crawl-sources-usecase`·`feature/10-source-persistence-adapter`·`feature/19-selection-normalize-dedup` 4개가 잔존(원격은 `feature/6-harness-docs`만 잔존). 로컬 위생 문제라 기능 영향은 없으나, 다음 세션이 브랜치 목록을 오해하지 않도록 기록 — 정리는 구현 세션에서 수행
- **(2026-07-22) 스킬 격리 풀림 발견 → 삭제로 종결** — D-012·D-023의 skills-archive 격리가 풀려 타 프로젝트 MSA 스킬 8종(code-review·create-branch·feign-client 2종·full-feature-cycle·http-test·implement-feature·kafka-event-handling·unit-test)이 `~/.claude/skills/`에 복원돼 있었음(하네스 이관 2026-07-22 과정 추정, skills-archive는 비어 있음). 사용자 결정으로 아카이브가 아닌 **삭제**로 종결(graphify.bak 백업 디렉터리 포함). 하네스 관리 심링크 3종(algo-pacemaker·graphify·symlink)만 유지 — M1-7에서 Sift용 code-review·create-branch 재박제 예정은 불변.
- **(2026-07-14) PORTFOLIO.md 실종** — 최근 완료(2026-07-05)에 작성 기록이 있으나 워크스페이스·sift-docs 어디에도 실물 없음. `docs/` → `sift-docs/` 이전(D-021) 과정 유실 추정 (루트 git 부재 시기라 이력 없음 — D-015 트레이드오프 실증 2번째 사례). 복원 여부는 사용자 결정 대기.
- **(2026-07-14) `.coderabbit.yaml` path 결함은 이미 해소** — 기존 비고의 "후속 수정 대기"는 죽은 기록: 98ec1d5(2026-07-06)에서 path 병합으로 수정 완료됨을 git 이력으로 확인. 스크래치패드의 `coderabbit-final.yaml` 사본도 유실 확인(세션 임시 디렉터리 소멸) — 이후 세션 간 임시물은 `.omc/notepad` 사용 (D-023).

### (2026-07-05 하네스 검토)
- **source 도메인/포트 골격은 실제로 존재하지 않음** — 브랜치·전체 히스토리·stash에 `Source`/`Article` 도메인·포트 코드 없음(`source/package-info.java`만 존재). 이전 STATE의 "source 골격 커밋" 기록은 오기(커밋 정리 과정에서 유실 추정). BACKLOG C 체크 해제, 다음 이슈 범위에 도메인/포트 생성 포함.
- 이슈 #1 커밋은 4개가 아니라 **3개** (common / 모듈경계+검증테스트 / 테스트인프라).

## 최근 완료 (최근 4~5건만 유지 — 이전 이력은 git 히스토리, D-023)
- **M2-2 후속 `[FEAT] normalizeDedup 재실행 멱등성` 병합 ✅ (이슈 #21 → PR #22 → develop `2c6f1df`, 2026-07-25)** — `clusterId = "c-" + min(memberIds)`(대표 선정과 id 발급 기준 분리) + 윈도우 로드 `loadCandidates(from, to)` `[from, to)` + 실행 단위 상태 교체(컷 탈락 기사 `null` 해제) + `updateClusters(Map)` 벌크. **테스트 79건 통과·CI `build & test` pass**. 윈도우 기준 컬럼은 `article.created_at`으로 확정, 윈도우 불변식 3종을 포트 javadoc·MVP-DESIGN §4에 명문화 (D-032). 자체 리뷰에서 4건 교정(회귀 테스트가 회귀를 못 잡던 결함·빈 맵 SQL 오류·from/to 무검증·TreeMap 순서 고정). 부수: `.coderabbit.yaml` path_filter 결함 수정. **미반영**: BACKLOG 선택 항목이던 `Summary.cleared`(해제 건수 계측)는 넣지 않음
- **M2-2 `[FEAT] 선별 1/3: Normalize + Dedup` 병합 ✅ (이슈 #19 → PR #20 → develop `c55c24c`, 2026-07-23)** — content 도메인(`CandidateArticle`·`ArticleNormalizer`·`TitleSimilarity`·`DedupClusterer`·`ArticleCluster`) + `NormalizeDedupUseCase`/`Service` + 포트 2종(`LoadCandidateArticlesPort`·`UpdateArticleClusterPort`, 실 어댑터는 M2-5). 도메인 단위 3 + fake 포트 서비스 1. D-030 적용. 스택 브랜치 base 재지정 후 병합. **CodeRabbit Major(재실행 멱등성)는 수용 → 이슈 #21로 분리**
- **M2-1 `[FEAT] Topic 도메인 + 시드 3종` 병합 ✅ (이슈 #17 → PR #18 → develop `4464753`, 2026-07-23)** — Topic 도메인 POJO(D-009 분리 유지) + `topic` JpaEntity·Repository(JSON 컬럼) + dev/ai/econ 시더(프로파일 가드·멱등). 도메인 8 + 통합 2 + 시더 1. 리뷰 반영 `0fb7499`(scoreThreshold 유한값·slug 로케일·컬렉션 null 검증). outbound 포트는 소비자 생길 때로 미룸(YAGNI)
- **M1-6 `[FEAT] collectionJob 배치 + 측정 베이스라인` 병합 ✅ (이슈 #14 → PR #15 → develop `f3e33d5`, 2026-07-21)** — collectStep 3요소(reader·processor·writer) + Job 조립(chunk=50) + `CollectionMetricsListener`(처리량·소요시간 로깅), 단위 3 + `@SpringBatchTest` 통합 1. CodeRabbit Major 2건은 후속 커밋 반영(소스별 오류 skip 격리 `73e9486`, step 실패 예외 로깅 `6f16920`), UseCase 우회 지적은 MVP-DESIGN §3① 이원화 근거로 반박 유지. **골든패스 코드 경로 완주** — 남은 것은 e2e 게이트(👤)와 M1-7 박제. 로컬 develop 동기화·feature 브랜치 정리 완료
- **CI 워크플로 도입 (2026-07-21)** — `.github/workflows/ci.yml`(JDK 21 + `./gradlew build`, develop·main PR/push 대상)을 develop에 직접 커밋(`03397b4`). PR 체크가 CodeRabbit뿐이던 공백을 메움 — PR #15 병합 때 테스트 자동 검증이 없어 로컬 수동 확인으로 대체한 것이 계기. ⚠️ 이슈·PR 흐름(§0.7) 밖의 직접 커밋이고, 브랜치 보호 required 등록은 미완(👤)
- (이전 완료 이력은 이 레포 git 히스토리로 조회 — D-023: STATE는 최근 4~5건만 유지. 하네스 개편 D-025·D-026, 가드 훅 폐지 D-027, edit 허용 D-028, M1-4·M1-5 병합은 여기서 밀려남)

## 대기 / 블록 (게이트)
- ~~develop 처리 결정~~ — **해소 (2026-07-17, D-025)**: develop = 통합 브랜치, main = 배포 브랜치로 공식화. develop이 앞서 있는 것은 정상(배포 전 통합분), 승격은 마일스톤 단위로 사람이 결정
- ~~main 브랜치 보호 규칙 설정~~ — **완료 확인 (2026-07-15, D-024 비고)**: sift-api main에 PR 필수·strict checks 설정됨 (gh api로 검증)
- ~~CI required 체크 등록 + main 보호 정비~~ — **완료 확인 (2026-07-22, gh api 검증)**: sift-api **main·develop 양쪽** `build & test` required + strict + PR 필수 설정됨(develop도 신규 보호). `enforce_admins=false`·승인 0명은 1인 운영 수용(D-024 불변). sift-docs main은 미보호 유지(사람 직접 커밋 레포 — D-024 수용), sift-harness는 private이라 보호 불가(Pro 필요, 수용)
- ~~#6 PR 발행·병합~~ — 완료 (PR #7 병합, 2026-07-10)
- ~~M1-3 이슈 발행~~ — 완료 (이슈 #8 → PR #9 병합, 2026-07-11)
- ~~이슈 #1 push·PR~~ — 완료 (PR #2 병합)
- ~~루트 `siftnews/` git 저장소화~~ — **취소 (D-015)**: 루트는 로컬 전용, git은 하위 sift-* 레포에만

## 비고
- **역할 분담 (D-026)**: 이슈 발행·브랜치·구현·자가검증·**커밋·push·PR 생성** = 에이전트 (모든 기록은 사용자 명의·기존 스타일 — HARNESS §0.7 컨벤션을 스스로 준수, 검사 장치 없음 D-027). **병합·리뷰 승인·issue/pr close·comment·release·repo/인프라 쓰기** = 사람. issue/pr `edit`는 에이전트 허용이되 assignee·라벨 관리 용도만 (D-028). **반영 완료한 리뷰 스레드의 resolve는 에이전트** — 답글은 여전히 사람 (D-033).
- **구현 리듬 (D-026)**: 작은 작업 1개 → build/test 자가검증 → 에이전트 커밋(`{type}: {한국어 요약}` 한 줄, 트레일러 금지) → 다음 작업.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — settings.local.json의 `gh api *` allow는 워크스페이스·하네스 시드 양쪽 제거 완료(👤 2026-07-22, 백스톱 복원).
