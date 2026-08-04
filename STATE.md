# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 쓰기 주인: 🧭 조율 세션 (D-034) · 마지막 갱신: 2026-08-04

## 현재 Phase
**Phase 1 (선별) — M2 선별 코드 경로 완주. 지금은 하네스 재정립.** #17~#39 전부 develop 병합(`9f66a6d`) — 수집·선별 양쪽에 트리거가 붙어 파이프라인이 무인으로 돈다. 남은 M2는 인프라 계열(Liquibase · Atom 검증 · 배치 테스트 Clock 고정).

## 지금 (in progress) — 하네스 역할 3분할 (D-034, 2026-08-04)
- **세션을 🧭 조율 / 🔧 구현 / 👀 검수로 나누고 경계를 권한으로 강제했다.** 런처 `sift-orch`·`sift-impl`·`sift-review`(한 스크립트 `sift-harness/bin/sift-role`을 세 이름으로 링크) + 프로파일 `.claude/roles/*.settings.json` 3종 + 역할 문서 `sift-docs/roles/` 3종. 상세는 [D-034](./DECISIONS.md) · [HARNESS §0.8](./HARNESS.md)
- **루프 문서 쓰기 주인이 구현 → 조율로 이관됐다** (D-016 개정). 구현 세션은 `Edit(sift-docs/**)` deny로 **막힌다** — 지침이 아니라 장치다. 그 대가로 §0.7 절차 3의 "BACKLOG 분해"가 성립하지 않게 돼, **작업 목록 원본을 이슈 본문 `## TODO`로** 옮기고 구현→조율 인계는 `.handoff/notes/`로 처리한다
- ⚠️ **검증에서 문법 오류를 하나 잡았다** — `Write(경로)` 권한 규칙은 **무효**이고 `Edit(경로)`만 파일 권한 검사에 걸린다(Claude Code가 경고로 직접 알려줌 — Edit 규칙이 Write·Edit·NotebookEdit를 모두 커버). 초안의 `Write(...)` 줄을 전부 제거했다. 실동작 확인: 검수 프로파일로 `sift-api/README.md` 쓰기 → `BLOCKED`, `.handoff/reviews/` 쓰기 → `WROTE`
- 🚨 **선결 과제가 드러났다 — 루트 하네스 파일이 심링크가 아니라 복사본이었다.** `CLAUDE.md`·`AGENTS.md`·`.claude/CLAUDE.md`·`.claude/settings.json` 넷 다 실체 파일이었고, CLAUDE.md 본문은 "심링크로 배치된다"고 적혀 있었다 — **그동안의 하네스 편집이 Mac Studio로 전달되지 않고 있었다.** `bootstrap.sh` 재실행으로 복원
- ⚠️ **그 과정에서 두 머신의 갈라짐이 드러났다** — ① 메모리 `sift-api-open-followups.md`가 로컬은 07-25판, harness는 07-21판(로컬이 최신·상위집합) → 로컬 값으로 복원. 반대로 `sift-harness-remote-setup.md`는 harness가 최신이라 유지 ② 홈 `settings.json`의 `model`이 로컬 `opus` vs harness `claude-fable-5[1m]`, `effortLevel: high`는 로컬에만 → 👤 결정으로 로컬 값 병합. **심링크가 아니어서 생긴 drift다 — 복원으로 재발 경로가 닫혔다**
- `AGENTS.md`가 CLAUDE.md보다 낡아 있었다(존재하지 않는 `.Codex/` 경로, sift-harness 미언급) → 동기화. 단 **런처·권한 프로파일은 Claude Code 전용**이라 다른 CLI에서는 경계가 지침으로만 남는다는 단서를 달았다

## 전체 정합 검토 (2026-08-04)

역할 재정립 직후 문서·환경 전반을 점검했다. **문서가 실제 상태를 못 따라간 것이 공통 원인**이었다.

- **TASKS: 8개 태스크(#23·#25·#27·#29·#31·#35·#37·#39)가 병합됐는데 `[~]` 리뷰 대기로 방치** + 항목 순서가 번호순도 발행순도 아니었다 → 번호순 재배열 + `gh` 실제 상태와 1:1 대조해 정정
- **BACKLOG: #27 이후 7개 태스크의 분해가 아예 없었다** — 이중 장부가 실제로는 안 쓰이고 있었다는 증거 → **D-035**로 작업 목록 원본을 이슈 본문 `## TODO`로 일원화, BACKLOG는 진행 중 이슈 1개로 축소(122줄 → 75줄). 되짚을 가치가 있는 기록(e2e가 잡은 수집 결함 5건·회귀 방어 실증 방식)은 남겼다
- **STATE 자체가 비대했다** — 이날 오전 "병합 대기"를 "최근 병합"으로 이름만 바꿔 40줄짜리 섹션 3개가 "최근 완료"와 중복돼 있었다(106줄) → 중복 제거 + 최근 5건 유지 규칙 복원(68줄)
- **sift-api README가 사실과 다르다** — 없는 `.claude/skills/`를 광고, 역할 분담이 D-026 이전 서술 → 위 「다음 액션」으로 등록
- ✅ **코드·설계 문서는 일치했다** — SELECTION·MVP-DESIGN·PLAN 모두 `origin/develop` 코드와 정합. 테스트 205개·CI 전부 통과·에이전트 서명 0건·문서 링크 전부 유효
- ⚠️ **검토 중 스스로 오진할 뻔했다** — 로컬 sift-api가 삭제된 브랜치에 체크아웃돼 있어 "SELECTION.md가 코드와 어긋난다"고 판단할 뻔했고, `git show origin/develop:`로 대조해서야 잡았다. **워킹트리를 진실로 취급하지 말 것** — 2026-07-26 fetch 누락 사고와 같은 계열이다

## 다음 액션 (next)
- 🔧 **sift-api README 정정 — 👤 결정됨 (2026-08-04)** — 공개 README가 **존재하지 않는 `.claude/skills/`를 "자작 스킬"로 광고**하고 있고(깨진 링크), 역할 분담 서술이 **D-026 이전 버전**("git/GitHub 쓰기는 사람이 직접")이다. 2026-08-04 전체 검토에서 발견 — 공개 문서가 사실과 다른 것이 가장 위험한 종류의 흠이라 우선 처리. 스킬 박제 자체는 별도 태스크(D-029가 "M2에서" 로 미뤄 뒀고 M2는 끝났다)
- 👤 **역할 런처로 갈아타기** — 다음 사이클부터 맨 `claude` 대신 `sift-orch`(계획·이슈) → `sift-impl`(구현·PR) → `sift-review`(검수)로 기동한다. 첫 실사용에서 경계가 실제로 맞는지(막혀야 할 게 막히고, 필요한 게 안 막히는지) 확인해 보고 — 과하게 조이면 프로파일 완화, 새는 곳이 있으면 추가
- 👤 **Mac Studio에 `bootstrap.sh` 재실행 필요** — 역할 런처·프로파일은 링크 배치라 그쪽에서도 한 번 돌려야 생긴다. 심링크 복원도 같은 이유로 그쪽에서 필요할 수 있다
- 🧭 **M2 잔여 태스크 착수** — Liquibase · Atom 검증 · **배치 테스트 Clock 고정**(#36 병합으로 착수 가능 — `SelectionJobIntegrationTest` 충돌 위험 해소)
- 👤 **#25 열린 결정 2건** — ① `sourceScore`를 중립 상수 1.0으로 둠(`source.trust_score` 재검토 시점이 지금) ② 키워드 매칭이 대소문자 무시 부분 문자열(영어 과매칭 수용). 둘 다 되돌리기 쉬운 쪽으로 잡았고 방향 지시가 있으면 후속에서 변경
- 👤 **#27 ERD 변경 2건 확인** — ① `issue.run_date` + `UNIQUE(topic_id, run_date)`: 기존 스키마엔 "몇 일자 호인가"가 없어 재실행 시 같은 날 호가 두 개 생길 수 있었다 ② `article_score.source_id` 비정규화: 소스 쏠림 완화에 필요한데 article은 Source 소유(D-018)라 조인 불가
- 👤 **CodeRabbit 리뷰 공백 — 재발 방침 결정 (4회째)** — PR #22·#28에 이어 #38(후속 커밋)·#40(첫 라운드부터 0건)까지 왔다. **#40은 새 커밋 push로 해소됐지만**, rate limit이 걸려도 체크가 `SUCCESS`로 뜨는 구조는 그대로라 다음에도 조용히 빌 수 있다. 선택지: ① usage-based reviews 과금 활성화 ② 리뷰 볼륨 축소(라벨 opt-in·증분 자동리뷰 중단) ③ 병합 전 리뷰 실제 수행 여부 확인을 절차로 굳히기(**구현 세션이 gh로 확인해 보고 — 이번처럼**) ④ 현 상태 수용하고 구현 세션 자체 리뷰로 대체
- 🔧 **`.coderabbit.yaml`에 `docs/**/*.md` 리뷰 허용 추가 — 👤 결정됨 (2026-08-04)** — `!**/*.md`가 설계 문서를 가려 "문서 갱신 여부 확인 불가"가 PR #30·#36·#38·#40에서 연속 재발했고, #40은 아예 Linked Issues 체크가 `Inconclusive`로 떨어졌다(리뷰 누락을 넘어 **체크 판정까지 흐린다**). PR #22의 `!out/**` 수정과 같은 성격의 필터 결함. 설계 문서만 리뷰 대상으로 되돌린다 — 아래 README 정정과 같은 PR로
- 👤 **PR #22 확인 부탁 (잔여 1건)** — `updateClusters(Map)` 단일 인자 vs `updateClusters(assigned, cleared)` 2-인자 분리(null 값 계약이 부담이면 재검토). ②`[from, to)` 실경계 검증은 #29에서 해소됨
- 👤 **markCrawled 배치 반영 결정** — 기본 제외 확정 vs 후속 이슈 (M1부터 계류 — D-029 잔여물 ②)
- 🔧 **SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환** — sift-api 이슈→PR (D-029 잔여물 ③)
- 🧭 **자작 스킬 박제 착수 가능** — D-029가 "M2 유스케이스에서 공통 패턴 추출"로 미뤄 뒀는데 **M2가 끝났다**. 유스케이스 풀구현·`code-review`·`create-branch` 후보. README가 이걸 이미 광고하고 있었던 것도 이 태스크가 밀린 흔적이다
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)
- 🔧 **로컬 위생** — ① **sift-api 체크아웃이 삭제된 브랜치(`feature/33-collection-trigger`)에 머물러 있다** — 2026-08-04 검토에서 하마터면 낡은 코드를 근거로 오진단할 뻔했다(`git show origin/develop:`로 대조해 잡음). `git switch develop` 필요 ② 병합 끝난 로컬 feature 브랜치 다수 잔존

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
- **`[REFACTOR] TopicSeeder를 인바운드 어댑터로 이동` 병합 ✅ (이슈 #39 → PR #40 → develop `9f66a6d`, 2026-08-04)** — 헥사고날 위반 3가지 정리: 인바운드인데 `adapter/out`에 있었고, `TopicJpaRepository`를 직접 불러 유스케이스·포트를 우회했으며, **저장이 conflict-ignore가 아니라 동시 기동 시 한쪽이 slug UNIQUE로 죽는 창**이 있었다. `SeedTopicsUseCase`+`SaveTopicPort` 신설, `ON CONFLICT (slug) DO NOTHING`, `git mv`로 이력 보존. **184 tests**. ⚠️ JSON 컬럼 4종은 네이티브 insert가 `@JdbcTypeCode` 매핑을 안 타므로 어댑터가 직렬화하고 리포지토리가 `jsonb`로 캐스팅한다 — 조용히 깨지면 선별이 빈 키워드로 돌아 `seedRoundTripsJsonColumns`로 실 DB 왕복 검증
- **`[FIX] 쿼리로 기사를 구분하는 소스의 URL 정규화` 병합 ✅ (이슈 #37 → PR #38 → develop `a9febc2`, 2026-08-04)** — `UriNormalizer`가 쿼리를 통째로 버려 AI타임스 50건이 고유 1건으로 뭉개지던 문제. **적재만의 피해가 아니었다** — `DedupClusterer`가 `normalizedUrl` 완전 일치로 병합하므로 제목이 전혀 다른 기사들이 한 클러스터가 되어 발행 누락으로 이어진다. 추적 파라미터 블랙리스트만 제거하는 규칙으로 교정 + CodeRabbit Major 수용해 기존 키 백필 러너 추가. **198 tests**
- **`[FIX] 발행 이슈가 비는 시간대 결함` 병합 ✅ (이슈 #35 → PR #36 → develop `83781dd`, 2026-08-01)** — 점수 조회 하한이 `runDate.atStartOfDay(UTC)`인데 `runDate`는 시스템 존(KST) 날짜여서 **KST 00:00~09:00 계산분이 전부 하한 미만**이 됐다. 확정 트리거 06:00 KST가 그 구간 한가운데라 **운영에서 매일 빈 호가 나갈 상태**였다. `selectTasklet`이 `WINDOW_FROM`을 주입받게 해 날짜에서 시각을 유도하는 것 자체를 없앴다. **180 tests**
- **`[FEAT] collectionTrigger — 수집 배치 상시 기동` 병합 ✅ (이슈 #33 → PR #34 → develop `783529a`, 2026-07-31)** — `collectionJob` 주기 기동(`@Scheduled`) + `launchedAt` 식별 파라미터 + 스케줄러 풀 2. **178 tests**. CodeRabbit `No actionable comments`. 확정값: 수집 주기 **매시 10분**(정각은 수집기가 몰려 비켜 뒀다 — MVP-DESIGN §3①이 "매시간 등"으로만 남긴 항목). ⚠️ **`collectionJob`은 JobParameters가 없어 두 번째 주기부터 `JobInstanceAlreadyCompleteException`**이 날 상태였다 — 트리거가 없어 안 드러나던 결함이라, 주기 기동을 붙이는 순간 첫 재실행에서 터진다. ⚠️ **스케줄러 풀이 기본 1개**라 06:00에 수집이 물리면 그날 발행이 밀린다. 범위 제외: 수동 기동 REST 엔드포인트(인증 없는 Job 기동 API가 되어 보안 표면이 생긴다)
- **M2-5 `[FEAT] selectionJob 배치 + selectionTrigger` 병합 ✅ (이슈 #31 → PR #32 → develop `bee0b18`, 2026-07-30)** — Step 3종(tasklet) 조립 + `@Scheduled` 일일 트리거 + `SelectionMetricsListener`. **174 tests 통과**. 🎯 **M2 공통 DoD 충족** — fake 없이 실 DB로 이슈 1건 + 항목 2건 생성 확인(단, UTC 기준이었다 — 위 🚨). 확정값: 윈도우 **24h**, 트리거 **06:00 Asia/Seoul**(D-031·D-032가 "M2-5에서 확정"으로 남긴 항목). **D-032 불변식 (3)을 트리거가 구조로 보장** — 윈도우를 한 번만 계산해 전 토픽 잡에 주입하므로 토픽 간 어긋남이 불가능. ⚠️ `@EnableScheduling`이 없어 배치가 영영 안 돌 뻔했다. 정정: `scoreStep (chunk)` → **tasklet**(윈도우 전체 재계산이 멱등성 전제라 아이템 단위로 못 쪼갬)
- (이전 완료 이력은 이 레포 git 히스토리와 [TASKS.md](./TASKS.md)로 조회 — D-023: STATE는 최근 4~5건만 유지. 하네스 개편 D-025·D-026, 가드 훅 폐지 D-027, edit 허용 D-028, M1 전체와 #23·#25·#27·#29 병합은 여기서 밀려남)

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
- **세션 역할 (2026-08-04 개정 — D-034)**: 세션은 🧭 조율 / 🔧 구현 / 👀 검수 3종이고 경계는 **권한 프로파일로 강제**된다. 루프 문서 쓰기 주인은 **조율 세션**(07-26의 "구현 세션 겸임"에서 이관). 쓰기 전 `sift-docs` **fetch 필수**는 불변 (위 "정정" 07-26 사고 참조). 각 역할 문서는 [`roles/`](./roles/).
- **구현 리듬 (D-026)**: 작은 작업 1개 → build/test 자가검증 → 에이전트 커밋(`{type}: {한국어 요약}` 한 줄, 트레일러 금지) → 다음 작업.
- ~~로컬 테스트는 `TZ=UTC`로 돌려야 한다~~ — **해소 (2026-08-01, PR #36 병합)**: 결함 자체가 고쳐졌고 `build.gradle`이 `systemProperty 'user.timezone', 'Asia/Seoul'`로 존을 못 박아 어디서 돌리든 같은 결과가 나온다. 단 **존 고정만으로는 이 계열 결함을 못 잡는다** — 재현이 실행 *시각*에도 좌우되기 때문이다(#35 기록 참조). 남은 방어는 test 프로파일 `Clock` 고정 태스크.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — settings.local.json의 `gh api *` allow는 워크스페이스·하네스 시드 양쪽 제거 완료(👤 2026-07-22, 백스톱 복원).
