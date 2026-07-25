# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-25

## 현재 Phase
**Phase 1 (선별) 진행 중** — 골든패스 코드 경로는 M1-6에서 완주, M1-7 박제 태스크는 해체(D-029). M2-1·M2-2(+#21 후속) 병합 완료, 선별 2/3이 다음

## 지금 (in progress) — 열린 이슈 없음
- M2-2 후속 #21이 PR #22로 병합되며 **열린 이슈가 0건**이다. 다음 태스크 이슈 발행이 루프 재개 지점
- ⚠️ **세션 역할 분담 (2026-07-25)**: 루프 문서(STATE·BACKLOG·TASKS·DECISIONS) 쓰기 창구 = **문서 세션**, 구현 세션은 `sift-api` 코드만 담당 (D-016 쓰기 주인 단일화)

## 다음 액션 (next)
- 🤖 **`[FEAT] 소스 RSS URL 확정 + source 시드` 발행 → 구현** — TASKS M2 순서상 다음. **M1-6 e2e 게이트(👤)를 여기서 해소**
- 👤 **CodeRabbit 리뷰 공백 확인** — PR #22는 `Review rate limited`로 **외부 리뷰 없이 병합**됐다(체크는 pass 표시). 병합 전 자체 리뷰 패스로 대체됐으나, rate limit이 재발하면 리뷰 게이트가 조용히 비는 구조 (아래 "정정" 참조)
- 👤 **PR #22 확인 부탁 2건** — ① `updateClusters(Map)` 단일 인자 vs `updateClusters(assigned, cleared)` 2-인자 분리(null 값 계약이 부담이면 재검토) ② `[from, to)` 경계 검증은 fake 계층에서 pass-through에 그침 — 실 경계 검증은 M2-5 Testcontainers 몫
- 👤 **M1-6 e2e 게이트 확인** — `bootRun`으로 실제 RSS → `article` 적재 + 측정 로그 확인. PR #15는 병합됐으나 이 실환경 게이트는 미확인 상태로 남아 있다 (BACKLOG C의 `[G]` 항목)
- 👤 **markCrawled 배치 반영 결정** — 기본 제외 확정 vs 후속 이슈 (M1부터 계류 — D-029 잔여물 ②)
- 🤖 **SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환** — sift-api 이슈→PR (D-029 잔여물 ③)
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)

## 정정 (사람 재검증·수정 흔적)
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
- **역할 분담 (D-026)**: 이슈 발행·브랜치·구현·자가검증·**커밋·push·PR 생성** = 에이전트 (모든 기록은 사용자 명의·기존 스타일 — HARNESS §0.7 컨벤션을 스스로 준수, 검사 장치 없음 D-027). **병합·리뷰 승인·issue/pr close·comment·release·repo/인프라 쓰기** = 사람. issue/pr `edit`는 에이전트 허용이되 assignee·라벨 관리 용도만 (D-028).
- **구현 리듬 (D-026)**: 작은 작업 1개 → build/test 자가검증 → 에이전트 커밋(`{type}: {한국어 요약}` 한 줄, 트레일러 금지) → 다음 작업.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — settings.local.json의 `gh api *` allow는 워크스페이스·하네스 시드 양쪽 제거 완료(👤 2026-07-22, 백스톱 복원).
