# Sift — TASKS (MVP 태스크 트리 = 이슈 발행 원본)

> **마일스톤(M) = GitHub 마일스톤, 태스크 1개 = 이슈 1개 = PR 1개** (HARNESS §0.7).
> 🤖 에이전트가 이슈 발행·구현·커밋·push·PR 생성(사용자 명의·기존 스타일, §0.7 컨벤션 자율 준수 — D-027), 👤 사람이 리뷰·병합·배포 승격 (D-025·D-026).
> 발행되면 제목 앞에 `#번호`를 태깅한다. 표기: `[ ]` 대기 · `[~]` 진행 · `[x]` 병합 완료
> 근거 문서: [PLAN.md](./PLAN.md) §5~7 · [MVP-DESIGN.md](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md) · [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)

---

## M1 — 골든패스: 수집 (Phase 0)

- [x] **#1 `[chore] 프로젝트 골격 & 모듈 경계`** — PR #2로 병합 완료 (2026-07-06 · develop으로 병합됨 → main 동기화는 STATE 게이트)
> **M1 재편 (D-020, 2026-07-07)**: 작업 순서 = **DDD inside-out** (도메인 → 유스케이스 → 어댑터 → 통합).
> 각 이슈의 DoD = 컴파일이 아니라 **해당 레벨의 테스트 통과** (실행 가능한 증거).

- [x] **#4 `[FEAT] Source 도메인 모델`** (M1-2) — PR #5로 develop 병합 완료 (2026-07-08)
  - 범위: `Source`·`Article` 애그리거트 + URL 정규화 불변식(UTM 제거·호스트 소문자화), `RawArticle`, 도메인 예외(BusinessException 기반). **JPA·Spring 무의존** (D-009·D-018). 선행: 클래스 다이어그램 (D-017)
  - DoD: **도메인 단위 테스트 통과** (정규화 규칙 케이스 포함), 도메인 패키지에 기술 의존 없음
- [x] **#6 `[docs] 하네스 문서·설계 문서 이동`** — PR #7로 develop 병합 완료 (2026-07-10). 레포 종속 설계 문서 `sift-api/docs/` 이동(D-021) + 권한 게이트 + README
- [x] **#8 `[FEAT] CrawlSources 유스케이스`** (M1-3) — PR #9로 develop 병합 완료 (2026-07-11)
  - 범위: `CrawlSourcesUseCase`(port.in) + `LoadActiveSourcesPort`·`FetchFeedPort`·`SaveArticlePort`(port.out) 정의, `CrawlSourcesService` 구현. 리뷰 반영: `LoadActiveSourcesPort.findActiveById` 추가
  - DoD: **fake 포트 기반 서비스 단위 테스트 통과** ✅
- [x] **#10 `[FEAT] Source 영속 어댑터`** (M1-4) — PR #11로 develop 병합 완료 (2026-07-16)
  - 범위: Source/Article JPA 엔티티·매핑·repository·adapter (BaseEntity 상속, UNIQUE normalized_url) · `Source.markCrawled()` 영속 반영 = 전용 포트 `UpdateSourcePort`로 결정(D-022)
  - DoD: **Testcontainers 통합 테스트 통과** ✅ (중복 저장 무시 검증 포함), `ApplicationModules.verify()` 유지 ✅
  - 부수 수정: `AbstractIntegrationTest`의 Postgres 컨테이너를 `@Container`(클래스별 stop/restart) → static 블록 싱글톤 패턴으로 변경 — 통합 테스트가 2개 이상일 때 컨테이너 재시작으로 포트가 바뀌며 Spring 컨텍스트 캐시가 옛 포트를 참조해 연결 실패하던 문제 수정
- [x] **#12 `[FEAT] RSS 수집 어댑터 (RssFeedAdapter)`** (M1-5) — PR #13으로 develop 병합 완료 (2026-07-19)
  - 범위: `rome` 의존성, RSS 파싱 → `RawArticle` 변환, lang 추출
  - DoD: **실제 RSS 픽스처(xml) 파싱 테스트 통과** ✅ (valid/empty/missing-pubdate/malformed 4케이스)
- [x] **#14 `[FEAT] collectionJob 배치 + 측정 베이스라인`** (M1-6) — PR #15로 develop 병합 완료 (2026-07-21)
  - 범위: Spring Batch Job(인바운드 어댑터, chunk=50), Actuator 노출, 처리량·소요시간 계측 (원칙 5)
  - DoD: 측정 로그 출력 ✅ (`CollectionMetricsListener` — read/write count·소요시간) · **`bootRun`으로 실제 RSS → `article` 적재 확인 (👤 실환경 게이트) ⏳ 미확인** — 코드 경로는 병합됐으나 실환경 검증은 남음
  - 리뷰 반영: 소스별 오류 skip 격리(한 소스 실패가 Job 전체를 죽이지 않도록), step 실패 시 원인 예외 로깅. UseCase 우회 지적은 MVP-DESIGN §3① 이원화 근거로 반박 유지
  - 남은 결정: 배치 경로의 `markCrawled`(`UpdateSourcePort`)는 미반영 상태로 병합 — 기본 제외 확정 vs 후속 이슈 (M1-7에서 배치 ↔ `CrawlSourcesService` 역할 경계와 함께 정리)
## M2 — 선별 (Phase 1)

> **선별 코드 경로는 완주했다** — #17~#39 전부 develop 병합. 수집·선별 양쪽에 트리거가 붙어 파이프라인이 무인으로 돈다. 남은 것은 아래 「M2 잔여」의 인프라 계열.
> **M1 잔여물 (D-029)**: ① D-009(도메인↔JPA 분리) → **분리 유지 확정 (2026-07-22)** ② markCrawled 배치 반영 결정 — 기본 제외 vs 후속 이슈 [👤 미결] ③ SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환 [sift-api 이슈→PR]
> **스킬 박제**(유스케이스 풀구현·`code-review`·`create-branch`)는 D-029로 "M2에서 공통 패턴 추출"까지 미뤄 뒀다 — **M2가 끝났으므로 이제 착수 가능한 상태**다.

- [x] **#17 `[FEAT] Topic 도메인 + 시드 3종`** — PR #18로 develop 병합 완료 (2026-07-23). Topic 도메인(POJO, D-009 분리) + topic 테이블·영속 + dev/ai/econ 시드 (MVP-DESIGN §1). **소스 RSS URL 확정·source 시드는 별도 이슈로 분리**
- [x] **#19 `[FEAT] 선별 1/3: Normalize + Dedup`** — PR #20으로 develop 병합 완료 (2026-07-23). 정규화 컷 + Jaccard 클러스터링(`dedup_cluster_id`, 임계 0.7), D-030 적용. content 로직·서비스(fake 포트)까지, 실 어댑터는 M2-5 배치
  - 후속 분리: **#21 재실행 멱등성** (CodeRabbit Major 수용)
- [x] **#21 `[FEAT] 선별 normalizeDedup 재실행 멱등성`** — PR #22로 develop 병합 완료 (2026-07-25). 윈도우 로드 + 실행 단위 상태 교체 + `clusterId = "c-" + min(memberIds)` + 벌크 갱신 (D-031). 범위는 fake 포트까지 — 실 어댑터 배선은 M2-5 유지
  - DoD: **fake 포트 서비스 단위 테스트 통과 ✅** (79건 전체 통과, CI pass) — 탈락 해제·신규 합류 시 clusterId 불변·동일 입력 동일 결과 3종 회귀
  - 파생 결정: 윈도우 기준 컬럼 = `article.created_at`, 윈도우 불변식 3종 명문화 (D-032)
- [x] **#23 `[FEAT] 소스 RSS URL 확정 + source 시드`** — PR #24로 develop 병합 완료 (2026-07-28). MVP-DESIGN §1 소스 시드 실제 RSS URL 확정(9종: dev/ai/econ 각 3, 한국어 4·영어 5 — 2026-07-26 실 응답 검증) + `SourceSeeder`. **M1-6 e2e 게이트(실 RSS → article 적재) 해소** — 258건·9개 소스 전부 적재 확인. e2e가 잡아낸 수집 결함 4건은 [BACKLOG 해당 절](./BACKLOG.md) 참조
- [x] **#25 `[FEAT] 선별 2/3: Filter + Score`** — PR #26으로 develop 병합 완료 (`ea5626b`, 2026-07-28). 토픽 필터 + 4항목 가중합 + `article_score`(breakdown JSON, `(article_id, topic_id)` upsert). **132 tests**. 리뷰 반영 `3ccfd22`(윈도우 검증을 도메인 예외로 통일, 점수 근거 불변식 강제)
  - 👤 열린 결정 2건: ① `sourceScore`를 중립 상수 1.0으로 둠(`source.trust_score` 재검토 시점) ② 키워드 매칭이 대소문자 무시 부분 문자열(영어 과매칭 수용)
- [x] **#27 `[FEAT] 선별 3/3: Rank & Select`** — PR #28로 develop 병합 완료 (`cd8de49`, 2026-07-29). threshold 컷 → 점수 내림차순 → 소스 쏠림 감점(×0.7) → 상위 `maxItems` → `issue`(DRAFT)+`issue_item`. **158 tests**. 전면 MMR은 넣지 않았다 — 같은 클러스터 쏠림은 #25가 대표 1건으로 이미 줄여 이 단계에 도달하지 않는다
  - ERD 2건 추가: `issue.run_date` + `UNIQUE(topic_id, run_date)` · `article_score.source_id` 비정규화(article은 Source 소유라 조인 불가 — D-018)
  - ⚠️ **CodeRabbit 리뷰 없이 병합**(`Review limit reached`, 인라인 0건)
- [x] **#29 `[FEAT] Source named interface + 선별 후보 조회·클러스터 갱신 실 배선`** — PR #30으로 develop 병합 완료 (`8bc6c67`, 2026-07-29). `@NamedInterface("article-catalog")` 노출 + content `allowedDependencies` 확장 + 두 포트 실 어댑터 + **`article.dedup_cluster_id` 컬럼 신설**(ERD엔 처음부터 있었으나 포트가 전부 fake라 실제로 만들어진 적이 없었다). **170 tests**. **`[from, to)` 경계를 Testcontainers로 실검증** — PR #22 확인 항목 ② 해소
  - ❌ CodeRabbit Major "마이그레이션 함께 배포"는 **실측 반박** — `ddl-auto: update`는 nullable 컬럼 추가를 자동 반영한다(유니크 제약만 미소급) → Liquibase 태스크 범위가 제약·인덱스·백필로 좁혀짐
- [x] **#31 `[FEAT] selectionJob 배치 + selectionTrigger`** — PR #32로 develop 병합 완료 (`bee0b18`, 2026-07-30). Step 3종(tasklet) 조립 + `@Scheduled` 일일 트리거 + `SelectionMetricsListener`. **174 tests**. **윈도우 24h·트리거 06:00(Asia/Seoul) 확정** — D-031·D-032가 "M2-5에서 정하라"고 남긴 항목. 🎯 **M2 공통 DoD 충족** — fake 없이 실 DB로 이슈 1건 + 항목 2건 생성 확인
  - ※ 원래 M2-5 한 덩어리였으나 **Modulith 경계 개방(#29)과 배치 조립은 성격이 달라** 리뷰 단위를 둘로 나눔 (2026-07-29)
  - 파생 정정: `scoreStep (chunk)` → **tasklet** (윈도우 전체 재계산이 멱등성 전제라 아이템 단위로 못 쪼갬, D-031) · `@EnableScheduling` 부재 발견(없으면 배치가 영영 안 돎)
  - **D-032 불변식 (3)을 트리거가 구조로 보장** — 윈도우를 한 번만 계산해 전 토픽 잡에 주입하므로 토픽 간 어긋남이 불가능
- [x] **#33 `[FEAT] collectionTrigger — 수집 배치 상시 기동`** — PR #34로 develop 병합 완료 (`783529a`, 2026-07-31). CodeRabbit `No actionable comments`. `@Scheduled` 주기 기동 + `launchedAt` 식별 파라미터 + 스케줄러 풀 확보. **수집 주기 매시 10분 확정**(정각은 수집기가 몰린다). 브랜치 `feature/33-collection-trigger`, base=develop
  - 착수 중 발견 2건: ① `collectionJob`은 JobParameters가 없어 **두 번째 주기부터 `JobInstanceAlreadyCompleteException`** ② 스케줄러 풀이 기본 1개라 06:00에 수집이 물리면 그날 발행이 밀린다
  - 범위 제외: 수동 기동 REST 엔드포인트 — 인증 없는 Job 기동 API가 되어 보안 표면이 생긴다. 주기를 설정으로 짧게 바꾸면 확인 목적은 충족
- [x] **🚨 #35 `[FIX] 발행 이슈가 비는 시간대 결함 — 점수 조회 하한을 윈도우 from으로 일치`** — PR #36으로 develop 병합 완료 (`83781dd`, 2026-08-01). `BuildIssueService`가 조회 하한을 `runDate.atStartOfDay(ZoneOffset.UTC)`로 잡는데 **`runDate`는 시스템 존(KST) 날짜**여서, KST 00:00~09:00에 계산된 점수가 전부 하한 미만이 됐다 — 확정 트리거 **06:00 Asia/Seoul(#31)이 이 구간 한가운데**라 운영에서 매일 빈 호가 나갈 상태였다. 브랜치 `feature/35-issue-lower-bound-zone`, base=develop. **180 tests 통과**
  - 수정: `selectTasklet`이 `WINDOW_FROM`을 주입받아 `buildIssueForTopic(topicId, runDate, scoredFrom)`으로 전달 — **날짜에서 시각을 유도하는 것 자체를 없앴다.** 트리거는 이미 윈도우를 전 토픽에 넘기고 있었고 `selectTasklet`만 안 받고 있었다. `runDate`는 호의 식별자·제목 용도로만 남음
  - ⚠️ **테스트 존 고정만으로는 부족했다** — 이 결함은 존뿐 아니라 **실행 시각**에도 좌우된다. 실제로 KST 23:54에 돌린 전체 테스트는 결함이 있는 상태에서도 통과했다(재현 구간 KST 00:00~09:00). 그래서 ① `build.gradle`에 `user.timezone=Asia/Seoul` 고정 ② `runDate`를 UTC 기준 내일로 주어 **시각 의존을 데이터로 제거한** 통합 테스트를 함께 넣었다
  - ✅ **회귀 방어를 실증했다** — 옛 구현을 일시 주입해 돌리니 **새 테스트 2건만 실패하고 나머지 178건은 통과**. 새 테스트가 결함을 잡는다는 확인이자, 기존 테스트로는 못 잡았다는 증거
  - 남김: `SelectionJobIntegrationTest` 나머지 케이스는 여전히 `Instant.now()` 기반 — 배치 전반을 시각 무관하게 검증하려면 테스트에서 `Clock` 고정이 필요(범위가 커 분리)
- [x] **#37 `[FIX] 쿼리로 기사를 구분하는 소스의 URL 정규화 — 추적 파라미터만 제거`** — PR #38로 develop 병합 완료 (`a9febc2`, 2026-08-04). `UriNormalizer`가 쿼리를 통째로 버려 `?idxno=213427`처럼 쿼리로 기사를 구분하는 소스의 기사가 **전부 같은 url로 정규화**되던 문제. **198 tests**
  - **실측으로 피해 범위를 확정했다** (시드 9종 실 피드 조회) — 피해 소스는 **AI타임스 하나**(50건 → 고유 1건, 손실 49). BBC는 쿼리가 `at_medium`·`at_campaign` 추적 파라미터 고정이라 손실 0, 나머지 7종은 전부 path로 기사를 구분한다
  - ⚠️ **적재만의 문제가 아니었다** — `DedupClusterer`의 병합 기준 ①이 `normalizedUrl` 완전 일치라, 정규화가 뭉개지면 **제목이 전혀 다른 기사들이 한 클러스터로 병합**된다. 선별은 대표 1건만 스코어링하므로 그대로 발행 누락이 된다 — 적재가 `UNIQUE`에서 먼저 막혀 가려져 있던 두 번째 피해
  - **뿌리는 설계 의도와 구현의 괴리** — #4의 범위는 "UTM 제거"였는데 구현·테스트가 "쿼리 전량 제거"로 확대돼 있었다. 원래 의도로 되돌린 것
  - 결정: **추적 파라미터 블랙리스트만 제거 + 나머지 쿼리 키 정렬 보존**(`utm_*`·`at_*` 접두어 + `fbclid`·`gclid`·`igshid`·`mc_cid`·`mc_eid`·`ref`)
  - ✅ **회귀 방어 실증** — 옛 동작을 일시 주입하니 **새 테스트 8건만 실패, 기존 19건 통과**. `DedupClustererTest#articlesDistinguishedOnlyByQueryStayInSeparateClusters` 실패가 dedup 오병합의 실재를 증명
  - **CodeRabbit 🟠 Major 수용 → 기존 키 백필 포함** (`67d4348`) — 규칙이 바뀌면 옛 `normalized_url` 키가 무효해져 재수집 시 중복 행이 된다. `RenormalizeArticleUrlsUseCase` + `ArticleUrlPort` + 기동 러너 `ArticleUrlRenormalizer`. 실제 백필 대상은 1건 — 👤 이 PR에서 처리로 결정
  - 설계 근거: id 커서 페이징 + 건별 트랜잭션 · 키 점유는 예외가 아닌 **사전 조회로 판정**(예외로 잡으면 트랜잭션이 rollback-only가 된다) · 러너 순서는 `SourceSeeder` → 재정규화 → Batch(order 0)
  - ⚠️ **Liquibase 도입 시 이 러너를 마이그레이션으로 흡수하고 지울 것** — 아래 Liquibase 태스크의 백필 범위에 포함
- [x] **#39 `[REFACTOR] TopicSeeder를 인바운드 어댑터로 이동`** — PR #40으로 develop 병합 완료 (`9f66a6d`, 2026-08-04). #23에서 `SourceSeeder`만 정리하고(PR #24) 모듈이 달라 남아 있던 쪽. **184 tests**
  - 어긋나 있던 것 3가지: ① 위치가 반대(인바운드인데 out 패키지) ② 유스케이스·포트 우회(`TopicJpaRepository` 직접 호출 + 엔티티 매핑 보유, `TopicMapper` 미사용) ③ **저장이 conflict-ignore가 아니었다** — 동시 기동 시 slug UNIQUE에 걸려 한쪽이 죽는 창
  - 변경: `SeedTopicsUseCase` + `SaveTopicPort` + `SeedTopicsService` 신설, `ON CONFLICT (slug) DO NOTHING`, 시더 2파일 `git mv`
  - ⚠️ **JSON 컬럼 4종은 네이티브 insert가 `@JdbcTypeCode(SqlTypes.JSON)` 매핑을 타지 않는다** — 어댑터가 직렬화해 넘기고 리포지토리가 `jsonb` 캐스팅. 조용히 깨지면 선별이 빈 키워드로 도므로 `seedRoundTripsJsonColumns`로 실 DB 왕복 검증
  - 자체 리뷰: 시더가 포트를 타면서 `existsBySlug` 사용처가 0이 돼 제거(`9301b08`)

### M2 잔여 (미착수)

- [ ] **#41 `[CHORE] 공개 README 정정 + CodeRabbit 설계 문서 리뷰 허용`** — 2026-08-04 전체 검토에서 발견. ① README가 **존재하지 않는 `.claude/skills/`를 광고**(깨진 링크) ② 역할 분담 서술이 **D-026 이전** ③ `.coderabbit.yaml`의 `!**/*.md`가 설계 문서를 리뷰에서 가려 "문서 갱신 확인 불가"가 PR #30·#36·#38·#40에서 연속 재발
- [ ] **`[CHORE] Sift용 스킬 박제`** — D-029가 "M2 유스케이스에서 공통 패턴 추출"로 미뤄 둔 것. **M2가 끝나 착수 가능**해졌다. 후보: 헥사고날 유스케이스 풀구현(도메인→포트→서비스→어댑터→테스트) · `code-review` · `create-branch`. #41이 README에서 이 항목을 지우는 이유이기도 하다

- [ ] **`[CHORE] Liquibase 마이그레이션 도입`** — 스키마가 `ddl-auto: update`에만 의존한다(운영 부적합, MVP-DESIGN §2에 "추후 전환 권장"으로만 적혀 있고 태스크로는 미등록이었음). 계기는 #23의 `ON CONFLICT (url)` — **유니크 제약이 없으면 예외인데 `update`는 기존 테이블에 제약을 소급 생성하지 않는다**(PR #24 CodeRabbit 지적 ⑥, e2e DB에서 실측 확인). 도입 시 두 시더(`SourceSeeder`·`TopicSeeder`)의 데이터 시드도 마이그레이션으로 흡수 검토
  - **2026-07-29 실측으로 범위가 좁혀짐 (PR #30 리뷰 계기)**: 기존 DB에 `ddl-auto: update`로 기동해 보니 **nullable 컬럼 추가는 자동 반영된다**(`dedup_cluster_id varchar(64)` 생성 확인). 반면 **유니크 제약은 소급 생성되지 않아** 같은 기동이 `ON CONFLICT (url)`에서 실패했다. 즉 마이그레이션이 실제로 필요한 대상은 **제약·인덱스·데이터 백필**이지 단순 컬럼 추가가 아니다
- [ ] **`[TEST] 배치 통합 테스트의 시각 의존 제거 (Clock 고정)`** — `SelectionJobIntegrationTest`·`CollectionJobIntegrationTest`가 `Instant.now()`로 윈도우를 잡아 **언제 돌리느냐에 따라 검증 강도가 달라진다**. #35가 그 대가를 실증했다 — 시간대 결함이 하루 15시간은 통과했고 CI(UTC)에서는 영영 안 걸렸다. `Clock` 빈을 test 프로파일에서 고정해 배치 전반을 벽시계와 무관하게 검증한다 (#35에서 분리)
- [ ] **`[FEAT] Atom 피드 파싱 검증`** — `rome`은 Atom(`<feed>`/`<entry>`)도 파싱하지만 `RssFeedAdapter` 테스트가 RSS 픽스처만 다뤄 미검증. 네이버 D2 등 Atom 소스 추가와 함께 픽스처 테스트 (#23에서 분리)

## M3 — 구독·발송 (Phase 1)

- [ ] **`[FEAT] Subscriber/Subscription 도메인 + 구독 API`** — 구독/해지 UseCase, REST 어댑터, **`preferred_send_hour` 포함** (이메일+토픽+수신 시각이 가입의 3요소 — D-019)
- [ ] **`[FEAT] 발송 스냅샷 (delivery_job/delivery_task)`** — DispatchIssueUseCase, `idempotency_key` 멱등(키 기준 issue vs job은 여기서 결정 — D-019 비고), **매시 pull 스캔 트리거** (당일 이슈 × preferred_send_hour)
- [ ] **`[FEAT] sendStep — 템플릿 렌더링 + SendEmailPort`** — LocalSmtpAdapter(mailpit), V1 단일 스레드 chunk=500 · ⚠️ 열린 질문: Thymeleaf vs Mustache → 이슈 안에서 결정
- [ ] **`[FEAT] retryJob — 에러 분류 + 지수 백오프`** — TRANSIENT/PERMANENT 분류, `next_retry_at`, DEAD 격리
  - M3 공통 DoD: E2E — 수집→선별→발송→**mailpit 수신 확인** (👤 게이트)

## M4 — 측정·부하: V1 베이스라인 (Phase 2 토대)

- [ ] **`[chore] 10만 구독자 시드 생성기`** — 부하용 데이터, 재실행 가능, **선호 시각 분포 파라미터화** (현실 분포=아침 피크 / 최악=전원 단일 시각 — D-019 부하 서사)
- [ ] **`[chore] 모니터링 스택`** — Prometheus + Grafana (compose), 배치 대시보드 · 위치(sift-infra 분리 여부) 이슈에서 결정
- [ ] **`[FEAT] V1 부하 측정 — 베이스라인 확보`** — 10만 건 발송 소요시간·TPS 측정 → `sift-docs/`에 기록 (포트폴리오 1챕터)

> **V2~V4는 여기서 미리 확정하지 않는다** — V1 측정 결과(병목 분석)가 다음 이슈 분해의 입력 (§0.5 에이전트 작업). 개선 사이클마다 `[FEAT] V{n} ...` 이슈를 그때 구성한다.
