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
  - [G] **e2e 게이트 (👤) 미확인** — `bootRun`으로 실제 RSS → `article` 적재 + 측정 로그 확인. 코드 경로는 병합됐으나 실환경 검증은 남음
  - 남은 결정: 배치 경로의 `markCrawled`(`UpdateSourcePort`)는 **미반영 상태로 병합** — 기본 제외 확정 vs 후속 이슈. 배치 ↔ 비배치 `CrawlSourcesService` 역할 경계와 함께 M1-7에서 정리
  - 미반영 리뷰(Trivial): 테스트 4파일의 `Fake*Port` 중복 → `adapter/in/batch/support` 패키지 추출 (선택 · PR 병합으로 코멘트가 묻혀 여기 기록)

## Phase 1 — M2-1 (#17) Topic 도메인 + 시드 3종 [진행]
> 브랜치 `feature/17-topic-domain-seed` (base develop). DDD inside-out, 단계별 테스트 DoD (D-020).
- [ ] (W) Topic 도메인 POJO + 불변식 (content/domain, JPA·Spring 무의존 — D-009 분리) · **DoD**: 도메인 단위 테스트 (slug 필수·정규화, keywordWeights 기본, exclude/include 규칙)
- [ ] (W) `topic` JpaEntity + 매핑 + JpaRepository (content/adapter/out/persistence, BaseEntity 상속, UNIQUE slug) · **DoD**: Testcontainers 통합 테스트 (저장·slug 유니크)
- [ ] (W) dev/ai/econ 시드 — 프로파일 가드 시더, 멱등 (MVP-DESIGN §1 키워드) · **DoD**: 기동 후 3개 적재 확인
- [ ] (W) `ApplicationModules.verify()` 통과 (content allowedDependencies 유지)
> 비고: outbound 포트(`LoadTopicPort`)·영속 어댑터는 소비자(선별 유스케이스)가 생기는 후속 M2 태스크로 미룸 — 이 이슈는 도메인·엔티티·시드까지 (YAGNI).

## Phase 1 이후 — [TASKS.md](./TASKS.md) M2~M4 참조
- 이슈가 발행되면 해당 태스크를 이 파일에 구현 단계로 분해해 루프를 돈다 (§0.7 절차 4)
- Sift용 `code-review` · `create-branch` 스킬 박제는 **M2로 이동** (D-029 — 2번째 유스케이스에서 공통 패턴 추출, D-012 시점 개정)
