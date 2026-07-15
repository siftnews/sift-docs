# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-14

## 현재 Phase
**Phase 0 — 스캐폴딩 + 골든패스** (진행 중)

## 지금 (in progress)
- **M1-4 `[feature] Source 영속 어댑터` 구현 완료, PR 대기** — 이슈 #10 → `feature/10-source-persistence-adapter` 브랜치. TDD로 구현, `./gradlew build` 전체 통과(Testcontainers 통합 테스트 포함). 👤 커밋·push·PR 발행 남음.

## 다음 액션 (next)
- 👤 **M1-4 커밋·push·PR 발행** — 변경 파일: `UpdateSourcePort`(신설)·`CrawlSourcesService`/테스트(연동)·`source/adapter/out/persistence/*`(Source·Article 엔티티·매퍼·repository·adapter 8개 파일)·`AbstractIntegrationTest`(싱글톤 컨테이너 수정)·문서(MVP-DESIGN·TASKS·DECISIONS)
- 🤖 **M1-5 `[feature] RSS 수집 어댑터 (RssFeedAdapter)` 착수** — M1-4 병합 후
- 👤 **develop 처리 결정**: PR #5·#7·#9가 develop으로 병합되어 develop이 main보다 13커밋 앞섬 (2026-07-11 기준) — D-012(GitHub Flow, develop 삭제)와 상충. ①develop→main 동기화 후 삭제(D-012 유지) 또는 ②develop 유지로 전략 변경(DECISIONS 기록 필요) 중 결정
- 👤 main 브랜치 보호 규칙 설정 (미완 게이트)
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)

## 정정 (사람 재검증·수정 흔적)
- **(2026-07-14) PORTFOLIO.md 실종** — 최근 완료(2026-07-05)에 작성 기록이 있으나 워크스페이스·sift-docs 어디에도 실물 없음. `docs/` → `sift-docs/` 이전(D-021) 과정 유실 추정 (루트 git 부재 시기라 이력 없음 — D-015 트레이드오프 실증 2번째 사례). 복원 여부는 사용자 결정 대기.
- **(2026-07-14) `.coderabbit.yaml` path 결함은 이미 해소** — 기존 비고의 "후속 수정 대기"는 죽은 기록: 98ec1d5(2026-07-06)에서 path 병합으로 수정 완료됨을 git 이력으로 확인. 스크래치패드의 `coderabbit-final.yaml` 사본도 유실 확인(세션 임시 디렉터리 소멸) — 이후 세션 간 임시물은 `.omc/notepad` 사용 (D-023).

### (2026-07-05 하네스 검토)
- **source 도메인/포트 골격은 실제로 존재하지 않음** — 브랜치·전체 히스토리·stash에 `Source`/`Article` 도메인·포트 코드 없음(`source/package-info.java`만 존재). 이전 STATE의 "source 골격 커밋" 기록은 오기(커밋 정리 과정에서 유실 추정). BACKLOG C 체크 해제, 다음 이슈 범위에 도메인/포트 생성 포함.
- 이슈 #1 커밋은 4개가 아니라 **3개** (common / 모듈경계+검증테스트 / 테스트인프라).

## 최근 완료 (최근 4~5건만 유지 — 이전 이력은 git 히스토리, D-023)
- **문서 아키텍처 재설계 Quick Win 적용 (2026-07-14, D-023)** — 리뷰에서 확인한 실드리프트 정리: `.coderabbit.yaml` D-018 정합(content "기사 저장" → source 소유로 정정), HARNESS §2 상태 체크박스 제거(상태 원본 = STATE·TASKS) + §3에 D-020 각주, BACKLOG M1-4 상태 정정, DECISIONS 개정 태그(D-007·D-011·D-012·D-015·D-017) + D-023 신설, CLAUDE.md 지침 우선순위 절(게이트 > HARNESS·DECISIONS > OMC), `implement-feature`·`unit-test` 스킬 아카이브 격리(D-012 완결), 컨텍스트 맵 `.drawio.svg` 변환·버전 관리 편입(D-017 형식 준수), `.gitignore`에 `.omc/`·`graphify-out/` 추가. 부수 발견: PORTFOLIO.md 실종 (→ 정정)
- **M1-4 `[feature] Source 영속 어댑터` 코드 완료 (2026-07-13, TDD)** — `UpdateSourcePort`(신설, D-022) + `CrawlSourcesService` 연동(fake 포트 단위 테스트 갱신). `source.adapter.out.persistence`: `SourceJpaEntity`/`SourceMapper`/`SourceJpaRepository`/`SourcePersistenceAdapter`, `ArticleJpaEntity`/`ArticleMapper`/`ArticleJpaRepository`/`ArticlePersistenceAdapter`(UNIQUE normalized_url 중복 무시). Testcontainers 통합 테스트 2건 통과. 부수: `AbstractIntegrationTest` 컨테이너를 `@Container` → static 싱글톤으로 수정(다중 통합 테스트 클래스 간 포트 불일치로 인한 연결 실패 해결). `source.trust_score`는 M2 스코어링 구현까지 보류
- **M1-3 `[feature] CrawlSources 유스케이스` 병합 ✅ (이슈 #8 → PR #9 → develop, 2026-07-11)** — `CrawlSourcesUseCase`(crawlAll/crawl) + outbound port 3종 + `CrawlSourcesService`(소스 단위 실패 격리, Clock 주입). fake 포트 단위 테스트 통과. 리뷰 반영: `crawl(sourceId)` 단일 조회를 `LoadActiveSourcesPort.findActiveById`로 변경
- **문서 정합성 작업 (2026-07-11)** — MVP-DESIGN §4에 `findActiveById` 반영 + §6에 `markCrawled` 영속 반영 열린 질문 추가(M1-4 태깅), TASKS·BACKLOG·STATE 병합 상태 동기화
- **#6 `[docs] 하네스 문서·설계 문서 이동` 병합 ✅ (PR #7 → develop, 2026-07-10)** — MVP-DESIGN·SELECTION·EVENTS `sift-api/docs/` 원본 이동(D-021) + 권한 게이트(`.claude/settings.json`) + README
- (2026-07-10 이전 완료 이력은 이 레포 git 히스토리로 조회 — D-023: STATE는 최근 4~5건만 유지)

## 대기 / 블록 (게이트)
- **develop 처리 결정** (사람) — PR #5·#7·#9 병합으로 develop이 main보다 13커밋 앞섬 (위 "다음 액션" 참조)
- **main 브랜치 보호 규칙** 설정 (사람)
- ~~#6 PR 발행·병합~~ — 완료 (PR #7 병합, 2026-07-10)
- ~~M1-3 이슈 발행~~ — 완료 (이슈 #8 → PR #9 병합, 2026-07-11)
- ~~이슈 #1 push·PR~~ — 완료 (PR #2 병합)
- ~~루트 `siftnews/` git 저장소화~~ — **취소 (D-015)**: 루트는 로컬 전용, git은 하위 sift-* 레포에만

## 비고
- **역할 분담 (D-013·D-014)**: git/GitHub 쓰기(**커밋**·이슈 발행·push·PR·병합) = 사람. 태스크 트리(TASKS.md)·이슈/PR/**커밋 메시지** 초안·브랜치·구현·자가검증 = 에이전트. gh CLI는 에이전트 필수 아님.
- **구현 리듬 (D-014)**: 작은 작업 1개 → build/test 자가검증 → 에이전트가 커밋 메시지 제안 → 사람 커밋 → 다음 작업. 메시지 형식 `{type}: {한국어 요약}`.
- 게이트(settings.json deny): git commit / push, gh issue·pr create / pr merge, git reset --hard / clean, docker compose down -v.
