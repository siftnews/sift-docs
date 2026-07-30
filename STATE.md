# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-31

## 현재 Phase
**Phase 1 (선별) — M2 선별 코드 경로 완주.** #17·#19·#21·#23·#25·#27·#29·#31 전부 develop 병합(`bee0b18`). 남은 M2는 수집 품질·인프라 계열(collectionTrigger #33 진행 중 · URL 정규화 · Liquibase · TopicSeeder 이동 · Atom 검증) + 아래 🚨 시간대 결함.

## 지금 (in progress) — #33 → PR #34 (collectionTrigger)
- **이슈 #33 → PR #34 (2026-07-31)** — `collectionJob` 주기 기동(`@Scheduled`) + `launchedAt` 식별 파라미터 + 스케줄러 풀 확보. **전체 178 tests 통과**(`TZ=UTC` 기준 — 아래 🚨). 브랜치 `feature/33-collection-trigger`, base=develop. CodeRabbit 감시 Monitor 가동
- **확정한 운영값** — 수집 주기 **매시 10분**(`sift.collection.cron`). 정각은 수집기가 몰리는 시각이라 비켜 뒀다. MVP-DESIGN §3①이 "매시간 등"으로만 남겨 둔 항목
- ⚠️ **`collectionJob`은 JobParameters가 없어 두 번째 주기부터 거부된다** — 파라미터가 비면 매 기동이 같은 JobInstance가 되고 `JobInstanceAlreadyCompleteException`이 난다. 트리거가 없어 드러나지 않던 결함이고, 주기 기동을 붙이는 순간 첫 재실행에서 터진다. `launchedAt`으로 해소(수집은 중복 무시 저장이라 재실행 멱등). 회귀 방어는 실 JobRepository 통합 테스트
- ⚠️ **스케줄러 스레드 풀이 기본 1개였다** — `jobLauncher.run`은 동기라 Job이 끝날 때까지 스레드를 점유한다. 수집(매시간)·선별(06:00)이 한 스레드를 두고 줄을 서, 06:00에 수집이 물리면 그날 발행이 그만큼 밀린다 → `spring.task.scheduling.pool.size: 2`
- **범위에서 뺀 것** — 수동 기동 REST 엔드포인트(인증 없는 Job 기동 API가 되어 보안 표면이 생기고, 주기를 설정으로 짧게 바꾸면 확인 목적은 충족된다). 운영 단계에서 인증과 함께 재검토

## 🚨 발견 — 06:00 트리거는 항상 빈 이슈를 만든다 (develop 기존 결함, #33 범위 밖)
- `BuildIssueService`가 `article_score` 조회 하한을 `runDate.atStartOfDay(ZoneOffset.UTC)`로 잡는데 **`runDate`는 시스템 존(KST) 날짜**다. KST 날짜 D의 UTC 자정은 KST D 09:00이라, **KST 00:00~09:00에 계산된 점수는 항상 하한 미만**이 된다
- 확정된 운영 트리거 시각 **06:00 Asia/Seoul**(#31)이 이 구간에 정확히 들어간다 → 그대로 두면 **운영에서 매일 빈 이슈가 발행된다**
- 재현 조건: `TZ=Asia/Seoul`이면 `SelectionJobIntegrationTest` 2건 실패, `TZ=UTC`면 통과. **CI가 UTC라 계속 녹색이었다.** `feature/33`을 stash하고 develop(`bee0b18`)에서 그대로 재현해 이 PR과 무관함을 확인
- 증거: `스코어링 완료: loaded=3, filtered=2, scored=2`(저장 2건) → `이슈 생성 완료: 점수=0건 → 게재=0건`(조회 0건)
- 수정 방향: 하한을 `runDate`와 같은 존의 자정으로 잡거나, 트리거가 계산한 윈도우 `from`을 그대로 넘긴다 — **`runDate`의 존과 하한의 존을 일치**시키는 것이 핵심. TASKS M2에 등록함
- **M2 공통 DoD가 UTC에서만 성립했다** — #31이 "실 DB로 이슈 1건 + 항목 2건 생성"을 확인했지만, 그 통합 테스트도 CI/UTC 기준이었다. 시간대를 바꾸면 같은 경로가 빈 이슈를 낸다

## 다음 액션 (next)
- 👤 **PR #34 리뷰·병합** — CodeRabbit 리뷰가 달리면 Monitor가 세션을 깨운다
- 🚨 **시간대 결함 이슈 발행 여부 결정** — 위 발견. 운영 발행이 매일 비는 사안이라 남은 M2 태스크 중 우선순위가 가장 높다
- 👤 **#25 열린 결정 2건** — ① `sourceScore`를 중립 상수 1.0으로 둠(`source.trust_score` 재검토 시점이 지금) ② 키워드 매칭이 대소문자 무시 부분 문자열(영어 과매칭 수용). 둘 다 되돌리기 쉬운 쪽으로 잡았고 방향 지시가 있으면 후속에서 변경
- 👤 **#27 ERD 변경 2건 확인** — ① `issue.run_date` + `UNIQUE(topic_id, run_date)`: 기존 스키마엔 "몇 일자 호인가"가 없어 재실행 시 같은 날 호가 두 개 생길 수 있었다 ② `article_score.source_id` 비정규화: 소스 쏠림 완화에 필요한데 article은 Source 소유(D-018)라 조인 불가
- 👤 **CodeRabbit 리뷰 공백 — 재발 방침 결정** — PR #22에 이어 **PR #28도 `Review limit reached` 상태로 병합**됐다(인라인 0건, 체크는 pass 표시). rate limit이 걸려도 체크가 통과로 뜨므로 리뷰 게이트가 조용히 빈다
- 👤 **`.coderabbit.yaml`의 `!**/*.md`가 설계 문서를 리뷰에서 가린다** — PR #30에서 `MVP-DESIGN.md`·`SELECTION.md`를 실제로 갱신했는데 CodeRabbit은 "문서 갱신 여부 확인 불가"로 판정했다. PR #22의 `!out/**` 수정과 같은 성격의 필터 결함 — "구조 변경 PR에서 설계 문서를 코드와 함께 갱신"(CLAUDE.md) 규칙이 앞으로도 검증 불가로 남는다
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
- **M2-5 `[FEAT] selectionJob 배치 + selectionTrigger` 병합 ✅ (이슈 #31 → PR #32 → develop `bee0b18`, 2026-07-30)** — Step 3종(tasklet) 조립 + `@Scheduled` 일일 트리거 + `SelectionMetricsListener`. **174 tests 통과**. 🎯 **M2 공통 DoD 충족** — fake 없이 실 DB로 이슈 1건 + 항목 2건 생성 확인(단, UTC 기준이었다 — 위 🚨). 확정값: 윈도우 **24h**, 트리거 **06:00 Asia/Seoul**(D-031·D-032가 "M2-5에서 확정"으로 남긴 항목). **D-032 불변식 (3)을 트리거가 구조로 보장** — 윈도우를 한 번만 계산해 전 토픽 잡에 주입하므로 토픽 간 어긋남이 불가능. ⚠️ `@EnableScheduling`이 없어 배치가 영영 안 돌 뻔했다. 정정: `scoreStep (chunk)` → **tasklet**(윈도우 전체 재계산이 멱등성 전제라 아이템 단위로 못 쪼갬)
- **M2-5 `[FEAT] Source named interface + 선별 후보 조회·클러스터 갱신 실 배선` 병합 ✅ (이슈 #29 → PR #30 → develop `8bc6c67`, 2026-07-29)** — `@NamedInterface("article-catalog")` 노출 + content `allowedDependencies` 확장 + 두 포트 실 어댑터. **170 tests**. ⚠️ `article.dedup_cluster_id` 컬럼이 ERD엔 처음부터 있었으나 **실제로 만들어진 적이 없었다**(포트가 전부 fake라 아무도 안 써서 안 드러남). PR #22 확인 항목 ② 해소(`[from, to)` 경계 Testcontainers 실검증). ❌ CodeRabbit Major "마이그레이션 함께 배포"는 **실측 반박** — `ddl-auto: update`는 **nullable 컬럼 추가를 자동 반영**한다(유니크 제약만 미소급) → Liquibase 태스크 범위가 제약·인덱스·백필로 좁혀짐. 스레드 미해결 유지(D-033)
- **M2-4 `[FEAT] 선별 3/3: Rank & Select` 병합 ✅ (이슈 #27 → PR #28 → develop `cd8de49`, 2026-07-29)** — threshold 컷 → 점수 내림차순 → 소스 쏠림 감점(×0.7) → 상위 `maxItems` → `issue`(DRAFT)+`issue_item`. **158 tests**. 전면 MMR은 넣지 않았다 — 같은 클러스터 쏠림은 #25가 대표 1건으로 이미 줄여 이 단계에 도달하지 않으므로 남은 소스 쏠림만 곱셈 감점으로 처리(SELECTION §6 열린 질문은 유지). ERD 2건 추가(`issue.run_date` UNIQUE · `article_score.source_id`). ⚠️ **CodeRabbit 리뷰 없이 병합**(`Review limit reached`, 인라인 0건)
- **M2-3 `[FEAT] 선별 2/3: Filter + Score` 병합 ✅ (이슈 #25 → PR #26 → develop `ea5626b`, 2026-07-28)** — 토픽 필터 + 4항목 가중합 + `article_score`(breakdown JSON, `(article_id, topic_id)` upsert). **132 tests**. 리뷰 반영 `3ccfd22`(윈도우 검증을 도메인 예외로 통일, 점수 근거 불변식 강제)
- **`[FEAT] 소스 RSS URL 확정 + source 시드` 병합 ✅ (이슈 #23 → PR #24 → develop `4366f7b`, 2026-07-28)** — 소스 9종 시드 + `SourceSeeder`. **M1-6 e2e 게이트 해소 ✅** — 실 RSS → `article` **258건, 9개 소스 전부**(`read=9 write=9 skip=0`). CodeRabbit 4건 반영·resolve(시더를 `adapter.in.bootstrap`으로 옮겨 `SeedSourcesUseCase`→`SaveSourcePort` 경유 + `ON CONFLICT (url) DO NOTHING`). ⚠️ **e2e가 수집 결함 4건 + 관측 공백 1건을 잡아냈다**(시더 실행 순서·`varchar(255)`·입력 내 중복 미필터·description NPE·SkipListener 부재) — **게이트를 실제로 열어보지 않았으면 전부 묻혔을 결함들**
- (이전 완료 이력은 이 레포 git 히스토리로 조회 — D-023: STATE는 최근 4~5건만 유지. 하네스 개편 D-025·D-026, 가드 훅 폐지 D-027, edit 허용 D-028, M1-4·M1-5·M1-6·M2-1·M2-2 병합은 여기서 밀려남)

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
- **로컬 테스트는 `TZ=UTC`로 돌려야 CI와 같은 결과가 나온다** — 시간대 의존 결함(위 🚨)이 해소되기 전까지. 로컬 기본 KST로 돌리면 `SelectionJobIntegrationTest` 2건이 실패한다.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — settings.local.json의 `gh api *` allow는 워크스페이스·하네스 시드 양쪽 제거 완료(👤 2026-07-22, 백스톱 복원).
