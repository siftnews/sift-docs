# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-08-01

## 현재 Phase
**Phase 1 (선별) — M2 선별 코드 경로 완주 + 무인 가동.** #17·#19·#21·#23·#25·#27·#29·#31·#33 전부 develop 병합(`783529a`) — 수집·선별 양쪽에 트리거가 붙어 파이프라인이 무인으로 돈다. 남은 M2는 수집 품질·인프라 계열(URL 정규화 · Liquibase · TopicSeeder 이동 · Atom 검증 · 배치 테스트 Clock 고정).

## 지금 (in progress) — #39 → PR #40 (TopicSeeder 인바운드 어댑터 이동)
- **이슈 #39 → PR #40 (2026-08-01)** — `TopicSeeder`가 `content/adapter/out/persistence`에 놓인 채 JPA 리포지토리를 직접 부르던 헥사고날 위반을 정리했다. 브랜치 `feature/39-topic-seeder-inbound`, base=develop. **182 tests 통과**. CodeRabbit 감시 Monitor 가동
- **어긋나 있던 것 3가지** — ① 위치가 반대(인바운드인데 out 패키지, 카탈로그 `TopicSeedData`도 함께) ② 유스케이스·포트 우회(`TopicJpaRepository` 직접 호출 + 엔티티 매핑을 시더가 보유, `TopicMapper`가 있는데 미사용) ③ **저장이 conflict-ignore가 아니었다** — "있는지 조회 → 없으면 save"라 두 인스턴스 동시 기동 시 둘 다 없다고 판정한다. slug UNIQUE가 있어 중복은 안 들어가지만 **한쪽 기동이 예외로 죽는다**
- **변경** — `SeedTopicsUseCase`(in) + `SaveTopicPort`(out) + `SeedTopicsService` 신설(Source와 같은 계약), `TopicPersistenceAdapter`가 `ON CONFLICT (slug) DO NOTHING`으로 구현, 시더 2파일을 `adapter/in/bootstrap`으로 `git mv`(이력 보존)
- ⚠️ **JSON 컬럼 한 겹이 더 든다** — `Topic`은 `include_keywords`·`exclude_keywords`·`keyword_weights`·`source_categories` 넷이 `@JdbcTypeCode(SqlTypes.JSON)`인데 **네이티브 insert는 이 매핑을 타지 않는다**. 어댑터가 직렬화한 문자열을 넘기고 리포지토리가 `jsonb`로 캐스팅한다. 직렬화를 손으로 하는 경로라 조용히 깨지면 선별이 빈 키워드로 돌아 후보를 못 고른다 → `seedRoundTripsJsonColumns`로 실 DB 왕복 검증
- **자체 리뷰로 죽은 코드 발견** — 시더가 포트를 타게 되면서 `existsBySlug`의 사용처가 0이 됐다. 리팩터링 PR에 죽은 코드를 남길 이유가 없어 제거(`9301b08`). ⚠️ **PR #40 본문의 "남긴 것" 문단이 이 커밋으로 부정확해졌다** — 본문 edit는 사람 영역(D-028)이라 교체 문구를 사용자에게 전달
- 시더는 `SourceSeeder`와 마찬가지로 **Liquibase 도입 시 마이그레이션으로 흡수될 코드**다
- 🚨 **rate limit 4번째 재발 → 해소** — PR #22·#28·#38에 이어 네 번째로, `Review limit reached`로 리뷰가 **시작조차 안 됐는데 체크는 `CodeRabbit: SUCCESS`**였다(gh로 확인). **base 최신화 머지 커밋을 push하자 리뷰가 자동 재트리거돼 정상 완료**됐다 — 사람이 `@coderabbitai review` 코멘트를 달지 않고도 공백을 메우는 경로가 확인된 셈이다(에이전트가 할 수 있는 유일한 재시도 수단). 다만 **체크가 통과로 뜨는 구조 자체는 그대로**다
- **CodeRabbit 리뷰 3건 처리 (2026-08-01)** — ① 🔵 Trivial Micrometer 계측: **미반영**. PR #38에서 👤가 이미 패스로 결정한 것과 같은 사안이고, **CodeRabbit 자신의 스크립트가 `SourceSeeder`에 Micrometer 참조가 없음을 확인**했다 — 선례에 없는 계측을 Topic 시더에만 붙이면 두 시더가 비대칭이 되어 이 PR의 목적("Source와 같은 구조로 맞추기")과 어긋난다. 미반영이라 스레드 resolve 안 함(D-033) ② ⚠️ Docstring 36.36%: 반박 유지(3회째 동일) ③ ❓ Linked Issues **Inconclusive** — `docs/MVP-DESIGN.md`가 `!**/*.md`에 걸려 문서 갱신 확인 불가. **PR #30·#38에 이어 세 번째 재발** ✅ Out of Scope 체크는 통과(`existsBySlug` 제거를 미리 반영한 것이 유효)

## 병합 대기 — #37 → PR #38 (쿼리로 기사를 구분하는 소스의 URL 정규화)
- **이슈 #37 → PR #38 (2026-08-01)** — `UriNormalizer`가 쿼리를 통째로 버려 기사가 뭉개지던 문제를 **추적 파라미터만 제거**하는 규칙으로 고쳤다. 브랜치 `feature/37-url-normalize-query`, base=develop. **187 tests 통과**. CodeRabbit 감시 Monitor 가동
- **실측으로 피해 범위를 확정했다** (시드 9종 실 피드 조회) — 피해 소스는 **AI타임스 하나**(50건 → 고유 1건, 손실 49). BBC는 쿼리가 `at_medium`·`at_campaign` 추적 파라미터 고정이라 손실 0이고(원본 피드에 같은 URL이 2번 실려 생긴 3건은 쿼리와 무관), 매일경제·한국경제 포함 나머지 7종은 전부 path로 기사를 구분한다. **"한국 언론사 다수"로 적어 뒀던 종전 추정보다 범위가 훨씬 좁다**
- ⚠️ **적재만의 문제가 아니었다** — `DedupClusterer`의 병합 기준 ①이 `normalizedUrl` 완전 일치라, 정규화가 뭉개지면 **제목이 전혀 다른 기사들이 한 클러스터로 병합**된다. 선별은 클러스터 대표 1건만 스코어링하므로 그대로 발행 누락이 된다 — 적재가 `UNIQUE(normalized_url)`에서 먼저 막혀 가려져 있던 두 번째 피해다
- **뿌리는 설계 의도와 구현의 괴리** — #4의 범위는 "URL 정규화 불변식(**UTM 제거**·호스트 소문자화)"였는데 구현과 테스트(`normalizeRemovesQueryStringAndLowercasesHost`)가 **쿼리 전량 제거**로 확대돼 있었다. 원래 의도로 되돌린 것
- **결정** — 추적 파라미터 블랙리스트(`utm_*`·`at_*` 접두어 + `fbclid`·`gclid`·`igshid`·`mc_cid`·`mc_eid`·`ref`)만 제거하고 나머지 쿼리는 키 정렬해 보존. 후보였던 «쿼리 전량 유지»는 추적 파라미터가 달라진 같은 기사가 중복 적재되고, «소스별 화이트리스트»는 피해 소스 1종인 지금 과설계
- **백필 불필요** — BBC는 추적 파라미터를 걷어내면 쿼리가 비어 기존 정규화 결과와 값이 같다. 실제로 값이 바뀌는 건 AI타임스뿐이고, 그 소스는 기존 DB에 옛 키로 1건만 적재돼 있어 신규와 충돌하지 않는다(고아 1건으로 남는다)
- ✅ **회귀 방어를 실증했다** — 옛 동작(쿼리 전량 제거)을 일시 주입해 돌리니 **새 테스트 8건만 실패하고 기존 19건은 통과**. `DedupClustererTest#articlesDistinguishedOnlyByQueryStayInSeparateClusters` 실패가 위 dedup 오병합의 실재를 증명한다
- 부수 확인: 실측 중 매일경제가 403을 냈으나 조회 UA 문제였고, `RssFeedAdapter`가 실제로 쓰는 경로(Java 기본 UA)로는 200 — 결함 아니다
- **CodeRabbit 리뷰 반영 완료 (`67d4348`)** — 인라인 1건(🟠 Major) 수용: **정규화 규칙이 바뀌면 기존 `normalized_url` 키가 무효해져, 같은 기사를 재수집할 때 `UNIQUE`가 옛 키의 행을 못 찾아 중복 행이 된다**는 지적. 원리가 타당해 `RenormalizeArticleUrlsUseCase` + `ArticleUrlPort` + 기동 러너 `ArticleUrlRenormalizer`로 백필을 구현했다. **198 tests**. 스레드 resolve(D-033)
  - 규모 재확인: 새 규칙으로 **키가 바뀌는 기사는 AI타임스 50/50, 나머지 8종은 0**(2026-08-01 재실측). 그런데 AI타임스는 결함 때문에 DB에 1행만 적재돼 있으므로 **실제 백필 대상은 1건**이다. 세션은 «영향 1건 + Liquibase 미도입»을 근거로 Liquibase 백필 대상 등록을 권했으나, **👤 이 PR에서 처리로 결정**
  - 설계: id 커서 페이징(500) + 건별 트랜잭션 — 한 건의 충돌이 앞서 옮긴 행을 되돌리지 않는다. 키 점유는 `UNIQUE` 위반 예외가 아니라 **사전 조회로 판정**한다(예외로 잡으면 트랜잭션이 rollback-only가 돼 false 반환으로 수습이 안 된다). 본문(text)을 싣지 않으려 프로젝션 조회. 멱등이라 매 기동 반복 무해
  - **기동 러너 순서가 중요하다** — `SourceSeeder` → `ArticleUrlRenormalizer` → Batch `JobLauncherApplicationRunner`(order 0). 재정규화가 수집 뒤로 밀리면 배치가 먼저 새 키로 적재해 같은 기사가 두 행으로 남는다
  - ⚠️ **이 러너는 Liquibase 도입 시 마이그레이션으로 흡수하고 지울 대상**이다 — 매 기동마다 전 행을 훑을 이유가 없다. 지금 러너인 것은 마이그레이션 인프라가 없어서다
  - **재리뷰 3건 처리 (2026-08-01)** — ① 🔵 Trivial Micrometer 계측: **미반영**(👤 결정, 스레드도 👤 resolve). 반박 근거는 `.coderabbit.yaml:81`이 대상을 "**새 배치 Job/Step**"으로 한정하는데 이 코드는 `ApplicationRunner`이고, Liquibase 도입 시 지울 일회성 코드라 메트릭을 붙이면 전환 때 대시보드까지 정리해야 한다는 것 ② ⚠️ Docstring 35%: 반박 유지(신규 main 8파일 **전부** javadoc 보유, 35%는 record accessor까지 세는 도구 계산이고 임계 80%는 도구 기본값) ③ ⚠️ Out of Scope: **타당 — 세션 누락**
  - ⚠️ **PR #38 본문이 코드와 어긋나 있다** — 백필 커밋 후 본문의 "백필이 필요 없는 이유" 절을 갱신하지 않았다. **본문 edit는 사람 영역(D-028)**이라 교체 문구만 사용자에게 전달했다. 후속 PR에서 반복하지 않으려면 **리뷰 반영 커밋 때 PR 본문 갱신 필요 여부를 함께 점검**할 것
  - 부수 발견: `SourceSeeder` javadoc의 "Liquibase 마이그레이션 전환은 후속(**MVP-DESIGN §2**)" 참조가 **댕글링**이다 — 현재 §2는 ERD이고 Liquibase 서술이 없다(문서 개정 중 소실 추정). 이번 PR에서는 내 문서만 참조를 정리했고 기존 주석은 손대지 않았다

## 병합 대기 — #35 → PR #36 (발행 이슈가 비는 시간대 결함)
- **이슈 #35 → PR #36 (2026-07-31)** — `BuildIssueService`의 점수 조회 하한을 **트리거가 계산한 윈도우 `from`으로 일치**시켰다. 브랜치 `feature/35-issue-lower-bound-zone`, base=develop. **180 tests 통과**. CodeRabbit 감시 Monitor 가동
- **결함** — 하한이 `runDate.atStartOfDay(ZoneOffset.UTC)`였는데 `runDate`는 시스템 존(KST) 날짜다. KST 날짜 D의 UTC 자정은 KST D 09:00이라 **KST 00:00~09:00에 계산된 점수는 전부 하한 미만**이 되고, 확정 발행 시각 **06:00 Asia/Seoul(#31)이 이 구간 한가운데**다 → 그대로 뒀으면 운영에서 **매일 빈 호**가 나갔다
- **수정** — `selectTasklet`이 `WINDOW_FROM`을 주입받아 `buildIssueForTopic(topicId, runDate, scoredFrom)`으로 넘긴다. **날짜에서 시각을 유도하는 것 자체를 없앤 것**이 핵심 — 트리거는 이미 윈도우를 전 토픽에 넘기고 있었고 `selectTasklet`만 그걸 안 받고 있었다. `runDate`는 호의 식별자·제목 용도로만 남는다
- ⚠️ **테스트 존 고정만으로는 못 잡는다 — 시각에도 좌우된다.** `build.gradle`에 `user.timezone=Asia/Seoul`을 박았지만, 이 결함은 **실행 시각**이 재현 구간(KST 00:00~09:00)에 들어와야 드러난다. 실제로 이번 작업 중 **KST 23:54에 돌린 전체 테스트는 결함이 있는 상태에서도 통과**했다. 그래서 `runDate`를 UTC 기준 내일로 주어 **시각 의존을 데이터로 제거한** 통합 테스트(`buildsIssueWhenRunDateIsAheadOfUtcDate`)를 함께 넣었다
- ✅ **회귀 방어를 실증했다** — 옛 구현을 일시 주입해 돌리니 **새 테스트 2건만 실패하고 나머지 178건은 통과.** 새 테스트가 결함을 잡는다는 확인이자, 기존 테스트로는 이 결함을 잡을 수 없었다는 증거다
- **남긴 것** — `SelectionJobIntegrationTest` 나머지 케이스는 여전히 `Instant.now()` 기반이다. 배치 전반을 벽시계와 무관하게 검증하려면 test 프로파일에서 `Clock`을 고정해야 한다 → TASKS에 별도 태스크로 등록
- **CodeRabbit 리뷰 반영 완료** — 인라인 1건(🟡 Minor) 수용: `LocalDate`에는 존 정보가 없는데 javadoc이 "존을 가진 날짜"라고 적어, **"어느 존의 자정이냐는 질문이 생긴다"는 바로 다음 절과 앞뒤가 어긋나 있었다**(질문이 생기는 이유가 존을 *안* 가졌기 때문이므로). `docs/SELECTION.md`의 같은 표현과 함께 `b7e403e`로 수정하고 스레드 resolve(D-033). 반박 2건: Docstring Coverage 33% 경고(도구 기본 임계 80%이지 이 레포 컨벤션이 아니다 — 포트 계약·회귀 테스트 의도에는 전부 달았고 빠진 건 테스트 헬퍼 오버로드), Linked Issues 「확인 불가」(아래 md 필터 결함)
- ⚠️ **rate limit 3번째 재발** — 반영 커밋(`b7e403e`) 이후 CodeRabbit 체크가 다시 `pass` 표시에 실제 내용은 `Review rate limited`. PR #22·#28에 이어 세 번째다. 이번엔 첫 라운드가 정상 리뷰돼 영향이 작지만(후속 커밋만 미리뷰), **체크가 통과로 뜨는 구조는 그대로**다

## 다음 액션 (next)
- 👤 **PR #38·#40 병합** — **#36은 병합 완료**(`83781dd`). 두 PR 모두 develop 최신을 머지해 두었다(#38 `cc6c249` **200 tests** · #40 **184 tests**, 둘 다 충돌 없음). 먼저 병합하는 쪽이 끝나면 나머지 하나는 다시 최신화가 필요하다 — **`MVP-DESIGN.md` §4의 인접 구역(Source 포트 / Content 포트)을 각각 고쳐** 그때 충돌이 날 수 있다
- 👤 **PR #40 본문 "남긴 것" 문단 교체** — `existsBySlug`를 제거해 서술이 어긋났다(아래 교체 문구는 세션이 전달)
- 🤖 **M2 잔여 태스크 착수** — Liquibase · Atom 검증 · **배치 테스트 Clock 고정(#36 병합으로 이제 착수 가능 — `SelectionJobIntegrationTest` 충돌 위험 해소)**
- 👤 **#25 열린 결정 2건** — ① `sourceScore`를 중립 상수 1.0으로 둠(`source.trust_score` 재검토 시점이 지금) ② 키워드 매칭이 대소문자 무시 부분 문자열(영어 과매칭 수용). 둘 다 되돌리기 쉬운 쪽으로 잡았고 방향 지시가 있으면 후속에서 변경
- 👤 **#27 ERD 변경 2건 확인** — ① `issue.run_date` + `UNIQUE(topic_id, run_date)`: 기존 스키마엔 "몇 일자 호인가"가 없어 재실행 시 같은 날 호가 두 개 생길 수 있었다 ② `article_score.source_id` 비정규화: 소스 쏠림 완화에 필요한데 article은 Source 소유(D-018)라 조인 불가
- 👤 **CodeRabbit 리뷰 공백 — 재발 방침 결정 (4회째)** — PR #22·#28에 이어 #38(후속 커밋)·#40(첫 라운드부터 0건)까지 왔다. **#40은 새 커밋 push로 해소됐지만**, rate limit이 걸려도 체크가 `SUCCESS`로 뜨는 구조는 그대로라 다음에도 조용히 빌 수 있다. 선택지: ① usage-based reviews 과금 활성화 ② 리뷰 볼륨 축소(라벨 opt-in·증분 자동리뷰 중단) ③ 병합 전 리뷰 실제 수행 여부 확인을 절차로 굳히기(**구현 세션이 gh로 확인해 보고 — 이번처럼**) ④ 현 상태 수용하고 구현 세션 자체 리뷰로 대체
- 👤 **`.coderabbit.yaml`의 `!**/*.md`가 설계 문서를 리뷰에서 가린다** — PR #30에서 `MVP-DESIGN.md`·`SELECTION.md`를 실제로 갱신했는데 CodeRabbit은 "문서 갱신 여부 확인 불가"로 판정했다. PR #22의 `!out/**` 수정과 같은 성격의 필터 결함 — "구조 변경 PR에서 설계 문서를 코드와 함께 갱신"(CLAUDE.md) 규칙이 앞으로도 검증 불가로 남는다. **PR #36에서 재발**(`docs/SELECTION.md is excluded by !**/*.md`로 명시 출력) — 이제 CodeRabbit이 제외 사실을 코멘트에 찍어 주므로 재발 여부는 눈으로 확인된다. **PR #38·#40에서 연속 재발** — #40은 아예 Linked Issues 체크가 `Inconclusive`로 떨어졌다("문서 갱신 여부를 확인할 수 없습니다"). 즉 이 필터는 이제 **리뷰 누락을 넘어 체크 판정까지 흐린다**
- 👤 **PR #22 확인 부탁 (잔여 1건)** — `updateClusters(Map)` 단일 인자 vs `updateClusters(assigned, cleared)` 2-인자 분리(null 값 계약이 부담이면 재검토). ②`[from, to)` 실경계 검증은 #29에서 해소됨
- 👤 **markCrawled 배치 반영 결정** — 기본 제외 확정 vs 후속 이슈 (M1부터 계류 — D-029 잔여물 ②)
- 🤖 **SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환** — sift-api 이슈→PR (D-029 잔여물 ③)
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)
- 🤖 **로컬 위생** — 병합 끝난 로컬 feature 브랜치 7개 잔존(`feature/4·8·10·19·21·23` + 원격에도 남은 `feature/6-harness-docs`). 정리 미수행

## 정정 (사람 재검증·수정 흔적)
- **(2026-07-26) 세션이 `sift-docs`를 fetch하지 않고 문서 상태를 오진단** — `sift-api`만 fetch한 채 로컬 `sift-docs` 사본을 읽고 "루프 문서가 4일 뒤처졌고 D-031이 DECISIONS에 없는 댕글링 참조"라고 사용자에게 보고했으나, **실제로는 원격이 최신이었고 로컬 클론이 낡은 것**이었다(원격에는 D-031·D-032 모두 존재). 그 오진단 위에서 D-031을 중복 작성해 커밋 2개(`d2d9cd7`·`8b929c9`)를 만들었고, push 단계에서 non-fast-forward로 발각돼 `git reset --keep origin/main`으로 폐기했다. 원인은 2대 머신(맥북↔Mac Studio) 공유 환경에서 **fetch 없이 로컬 파일을 진실로 취급**한 것 — 메모리에 재발 방지 기록(`fetch-before-reading-loop-docs`).
- **(2026-07-25) `.coderabbit.yaml` path_filter가 헥사고날 `out` 패키지를 리뷰에서 제외하고 있었음** — `!**/out/**`(IntelliJ 빌드 산출물 의도)가 `application/port/out`·`adapter/out`까지 매칭시켜, **아웃바운드 포트·영속/피드 어댑터가 CodeRabbit 리뷰 사각지대**였다. PR #22에서 `!out/**`로 수정(`27d8f8b`). 소급 영향: M1-4 Source 영속 어댑터·M1-5 RssFeedAdapter·M2-2 포트 2종 등 `out` 경로 코드는 리뷰를 안 받았을 가능성이 높다 — 필요하면 사후 리뷰 요청은 👤 (comment는 사람 영역, D-026)
- **(2026-07-25) PR #22는 CodeRabbit 리뷰 없이 병합됨** — 체크 결과가 `pass`로 표시되지만 실제 내용은 `Review rate limited`. **rate limit이 걸려도 체크는 통과로 뜨므로 리뷰 게이트가 조용히 비는 구조**다. #22는 구현 세션의 자체 리뷰 패스(회귀 테스트가 회귀를 못 잡던 결함·빈 맵 SQL 오류·from/to 무검증 발견)로 대체됐다. **2026-07-29 PR #28에서 재발** — 👤 대응 방침 결정 필요
- **(2026-07-25) 로컬 feature 브랜치 "정리 완료" 기록은 부정확** — 최근 완료 항목들의 "로컬 feature 브랜치 정리 완료" 표현과 달리, 실제 로컬에는 병합 끝난 브랜치들이 잔존한다. 로컬 위생 문제라 기능 영향은 없으나, 다음 세션이 브랜치 목록을 오해하지 않도록 기록 — 정리는 구현 세션에서 수행
- **(2026-07-22) 스킬 격리 풀림 발견 → 삭제로 종결** — D-012·D-023의 skills-archive 격리가 풀려 타 프로젝트 MSA 스킬 8종(code-review·create-branch·feign-client 2종·full-feature-cycle·http-test·implement-feature·kafka-event-handling·unit-test)이 `~/.claude/skills/`에 복원돼 있었음(하네스 이관 2026-07-22 과정 추정, skills-archive는 비어 있음). 사용자 결정으로 아카이브가 아닌 **삭제**로 종결(graphify.bak 백업 디렉터리 포함). 하네스 관리 심링크 3종(algo-pacemaker·graphify·symlink)만 유지 — M1-7에서 Sift용 code-review·create-branch 재박제 예정은 불변.
- **(2026-07-14) PORTFOLIO.md 실종** — 최근 완료(2026-07-05)에 작성 기록이 있으나 워크스페이스·sift-docs 어디에도 실물 없음. `docs/` → `sift-docs/` 이전(D-021) 과정 유실 추정 (루트 git 부재 시기라 이력 없음 — D-015 트레이드오프 실증 2번째 사례). 복원 여부는 사용자 결정 대기.
- **(2026-07-14) `.coderabbit.yaml` path 결함은 이미 해소** — 기존 비고의 "후속 수정 대기"는 죽은 기록: 98ec1d5(2026-07-06)에서 path 병합으로 수정 완료됨을 git 이력으로 확인. 스크래치패드의 `coderabbit-final.yaml` 사본도 유실 확인(세션 임시 디렉터리 소멸) — 이후 세션 간 임시물은 `.omc/notepad` 사용 (D-023).

### (2026-07-05 하네스 검토)
- **source 도메인/포트 골격은 실제로 존재하지 않음** — 브랜치·전체 히스토리·stash에 `Source`/`Article` 도메인·포트 코드 없음(`source/package-info.java`만 존재). 이전 STATE의 "source 골격 커밋" 기록은 오기(커밋 정리 과정에서 유실 추정). BACKLOG C 체크 해제, 다음 이슈 범위에 도메인/포트 생성 포함.
- 이슈 #1 커밋은 4개가 아니라 **3개** (common / 모듈경계+검증테스트 / 테스트인프라).

## 최근 완료 (최근 4~5건만 유지 — 이전 이력은 git 히스토리, D-023)
- **`[FEAT] collectionTrigger — 수집 배치 상시 기동` 병합 ✅ (이슈 #33 → PR #34 → develop `783529a`, 2026-07-31)** — `collectionJob` 주기 기동(`@Scheduled`) + `launchedAt` 식별 파라미터 + 스케줄러 풀 2. **178 tests**. CodeRabbit `No actionable comments`. 확정값: 수집 주기 **매시 10분**(정각은 수집기가 몰려 비켜 뒀다 — MVP-DESIGN §3①이 "매시간 등"으로만 남긴 항목). ⚠️ **`collectionJob`은 JobParameters가 없어 두 번째 주기부터 `JobInstanceAlreadyCompleteException`**이 날 상태였다 — 트리거가 없어 안 드러나던 결함이라, 주기 기동을 붙이는 순간 첫 재실행에서 터진다. ⚠️ **스케줄러 풀이 기본 1개**라 06:00에 수집이 물리면 그날 발행이 밀린다. 범위 제외: 수동 기동 REST 엔드포인트(인증 없는 Job 기동 API가 되어 보안 표면이 생긴다)
- **M2-5 `[FEAT] selectionJob 배치 + selectionTrigger` 병합 ✅ (이슈 #31 → PR #32 → develop `bee0b18`, 2026-07-30)** — Step 3종(tasklet) 조립 + `@Scheduled` 일일 트리거 + `SelectionMetricsListener`. **174 tests 통과**. 🎯 **M2 공통 DoD 충족** — fake 없이 실 DB로 이슈 1건 + 항목 2건 생성 확인(단, UTC 기준이었다 — 위 🚨). 확정값: 윈도우 **24h**, 트리거 **06:00 Asia/Seoul**(D-031·D-032가 "M2-5에서 확정"으로 남긴 항목). **D-032 불변식 (3)을 트리거가 구조로 보장** — 윈도우를 한 번만 계산해 전 토픽 잡에 주입하므로 토픽 간 어긋남이 불가능. ⚠️ `@EnableScheduling`이 없어 배치가 영영 안 돌 뻔했다. 정정: `scoreStep (chunk)` → **tasklet**(윈도우 전체 재계산이 멱등성 전제라 아이템 단위로 못 쪼갬)
- **M2-5 `[FEAT] Source named interface + 선별 후보 조회·클러스터 갱신 실 배선` 병합 ✅ (이슈 #29 → PR #30 → develop `8bc6c67`, 2026-07-29)** — `@NamedInterface("article-catalog")` 노출 + content `allowedDependencies` 확장 + 두 포트 실 어댑터. **170 tests**. ⚠️ `article.dedup_cluster_id` 컬럼이 ERD엔 처음부터 있었으나 **실제로 만들어진 적이 없었다**(포트가 전부 fake라 아무도 안 써서 안 드러남). PR #22 확인 항목 ② 해소(`[from, to)` 경계 Testcontainers 실검증). ❌ CodeRabbit Major "마이그레이션 함께 배포"는 **실측 반박** — `ddl-auto: update`는 **nullable 컬럼 추가를 자동 반영**한다(유니크 제약만 미소급) → Liquibase 태스크 범위가 제약·인덱스·백필로 좁혀짐. 스레드 미해결 유지(D-033)
- **M2-4 `[FEAT] 선별 3/3: Rank & Select` 병합 ✅ (이슈 #27 → PR #28 → develop `cd8de49`, 2026-07-29)** — threshold 컷 → 점수 내림차순 → 소스 쏠림 감점(×0.7) → 상위 `maxItems` → `issue`(DRAFT)+`issue_item`. **158 tests**. 전면 MMR은 넣지 않았다 — 같은 클러스터 쏠림은 #25가 대표 1건으로 이미 줄여 이 단계에 도달하지 않으므로 남은 소스 쏠림만 곱셈 감점으로 처리(SELECTION §6 열린 질문은 유지). ERD 2건 추가(`issue.run_date` UNIQUE · `article_score.source_id`). ⚠️ **CodeRabbit 리뷰 없이 병합**(`Review limit reached`, 인라인 0건)
- **M2-3 `[FEAT] 선별 2/3: Filter + Score` 병합 ✅ (이슈 #25 → PR #26 → develop `ea5626b`, 2026-07-28)** — 토픽 필터 + 4항목 가중합 + `article_score`(breakdown JSON, `(article_id, topic_id)` upsert). **132 tests**. 리뷰 반영 `3ccfd22`(윈도우 검증을 도메인 예외로 통일, 점수 근거 불변식 강제)
- (이전 완료 이력은 이 레포 git 히스토리로 조회 — D-023: STATE는 최근 4~5건만 유지. 하네스 개편 D-025·D-026, 가드 훅 폐지 D-027, edit 허용 D-028, M1-4·M1-5·M1-6·M2-1·M2-2·#23 병합은 여기서 밀려남)

## 대기 / 블록 (게이트)
- ~~develop 처리 결정~~ — **해소 (2026-07-17, D-025)**: develop = 통합 브랜치, main = 배포 브랜치로 공식화. develop이 앞서 있는 것은 정상(배포 전 통합분), 승격은 마일스톤 단위로 사람이 결정
- ~~main 브랜치 보호 규칙 설정~~ — **완료 확인 (2026-07-15, D-024 비고)**: sift-api main에 PR 필수·strict checks 설정됨 (gh api로 검증)
- ~~CI required 체크 등록 + main 보호 정비~~ — **완료 확인 (2026-07-22, gh api 검증)**: sift-api **main·develop 양쪽** `build & test` required + strict + PR 필수 설정됨(develop도 신규 보호). `enforce_admins=false`·승인 0명은 1인 운영 수용(D-024 불변). sift-docs main은 미보호 유지(사람 직접 커밋 레포 — D-024 수용), sift-harness는 private이라 보호 불가(Pro 필요, 수용)
- ~~M1-6 e2e 게이트 (실 RSS → article 적재)~~ — **해소 (2026-07-27, 이슈 #23)**: 258건·9개 소스 전부 적재 확인
- ~~#6 PR 발행·병합~~ — 완료 (PR #7 병합, 2026-07-10)
- ~~M1-3 이슈 발행~~ — 완료 (이슈 #8 → PR #9 병합, 2026-07-11)
- ~~이슈 #1 push·PR~~ — 완료 (PR #2 병합)
- ~~루트 `siftnews/` git 저장소화~~ — **취소 (D-015)**: 루트는 로컬 전용, git은 하위 sift-* 레포에만

## 비고
- **역할 분담 (D-026)**: 이슈 발행·브랜치·구현·자가검증·**커밋·push·PR 생성** = 에이전트 (모든 기록은 사용자 명의·기존 스타일 — HARNESS §0.7 컨벤션을 스스로 준수, 검사 장치 없음 D-027). **병합·리뷰 승인·issue/pr close·comment·release·repo/인프라 쓰기** = 사람. issue/pr `edit`는 에이전트 허용이되 assignee·라벨 관리 용도만 (D-028). **반영 완료한 리뷰 스레드의 resolve는 에이전트** — 답글은 여전히 사람 (D-033).
- **세션 역할 (2026-07-26 개정)**: 07-25의 "루프 문서 쓰기 = 문서 세션 전담"은 **구현 세션이 문서 쓰기까지 겸임**하는 것으로 사용자 확정. 단 쓰기 전 `sift-docs` **fetch 필수** (위 "정정" 07-26 사고 참조).
- **구현 리듬 (D-026)**: 작은 작업 1개 → build/test 자가검증 → 에이전트 커밋(`{type}: {한국어 요약}` 한 줄, 트레일러 금지) → 다음 작업.
- ~~로컬 테스트는 `TZ=UTC`로 돌려야 한다~~ — **해소 (2026-08-01, PR #36 병합)**: 결함 자체가 고쳐졌고 `build.gradle`이 `systemProperty 'user.timezone', 'Asia/Seoul'`로 존을 못 박아 어디서 돌리든 같은 결과가 나온다. 단 **존 고정만으로는 이 계열 결함을 못 잡는다** — 재현이 실행 *시각*에도 좌우되기 때문이다(#35 기록 참조). 남은 방어는 test 프로파일 `Clock` 고정 태스크.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — settings.local.json의 `gh api *` allow는 워크스페이스·하네스 시드 양쪽 제거 완료(👤 2026-07-22, 백스톱 복원).
