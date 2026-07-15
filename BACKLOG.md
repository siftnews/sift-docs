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
- [x] 협업 역할 분담 확정 — GitHub 쓰기(이슈·push·PR·병합)는 사람, 초안·구현·커밋은 에이전트 (D-013)
- [x] MVP 태스크 트리 — [TASKS.md](./TASKS.md) 작성 (이슈 발행 원본)
- [G] main 브랜치 보호 규칙 설정 (사람)
- ~~루트 `siftnews/` git 저장소화~~ — D-015로 취소 (루트는 로컬 전용, git은 하위 sift-* 레포만)
- [G] develop 처리 결정 (사람) — PR #5·#7·#9 병합으로 develop이 main보다 13커밋 앞섬 (2026-07-11): ①main 동기화 후 develop 삭제(D-012 유지) 또는 ②develop 유지로 전략 변경(DECISIONS 기록) → STATE "다음 액션" 참조
- [G] `develop` 브랜치 삭제 (사람) — 위 결정이 ①일 때. GitHub 기본 브랜치 = main 확인 (D-012)
- [x] 이슈 #1 push·PR — PR #2 병합 완료 (2026-07-06)
- ~~권한 조정(git push/gh 허용)~~ · ~~gh CLI 설치~~ — D-013으로 불필요 (GitHub 쓰기는 사람)

### B. 프로젝트 골격 (코드 기초 틀) — 이슈 #1 `[chore] 프로젝트 골격 & 모듈 경계`
- [x] (W) `common` 모듈 — `BaseEntity`(+JPA Auditing), `Clock` 빈, `BusinessException` · **DoD**: 컴파일 ✅
- [x] (W) Modulith 모듈 경계 선언 — source/content/subscriber/delivery/tracking `package-info` + `allowedDependencies={"common"}` · **DoD**: 구조 생성 ✅
- [x] (W) 모듈 검증 테스트 `ApplicationModulesTest` — `ApplicationModules.verify()` · **DoD**: verify 통과 ✅
- [x] (W) 테스트 인프라 — testcontainers 의존성 + `AbstractIntegrationTest` 베이스 + `application-test.yaml` ✅
- [ ] (W) 측정 베이스라인 골격 — Actuator + 배치 처리량·소요시간 로깅 · **→ TASKS M1 `collectionJob 배치 + 측정 베이스라인` 이슈로 이동** (배치 있어야 측정 대상 존재)

### C. 골든패스 (collectionJob)
> **D-020 재편 (2026-07-07)**: 이 섹션의 구 분해(레이어 단위·DoD 컴파일)는 폐기 — TASKS M1-2~7(DDD inside-out, 단계별 테스트 DoD)로 대체.
> 이슈가 발행되면 해당 이슈의 구현 단계를 여기에 분해한다 (§0.7 계층).
- [x] M1-2 `[feature] Source 도메인 모델` — 이슈 #4 → PR #5 병합 완료 (2026-07-08)
- [x] M1-3 `[feature] CrawlSources 유스케이스` — 이슈 #8 → PR #9 병합 완료 (2026-07-11 · 완료된 분해 내역은 git 히스토리 — D-023)
- [~] M1-4 `[feature] Source 영속 어댑터` — 이슈 #10, 코드 완료(TDD·`./gradlew build` 통과), 👤 커밋·push·PR 대기
  - ⚠️ 프로토콜 이탈 기록: 이 이슈는 BACKLOG 분해 없이 구현됨 (§0.7 절차 4 미준수) — 재발 방지는 D-023 후속(/harness-check)
- (다음) M1-5 `[feature] RSS 수집 어댑터` — 이슈 발행(👤) 후 여기에 구현 단계 분해

## Phase 1 이후 — [TASKS.md](./TASKS.md) M1(골든패스 박제)~M4 참조
- 이슈가 발행되면 해당 태스크를 이 파일에 구현 단계로 분해해 루프를 돈다 (§0.7 절차 4)
- Sift용 `code-review` · `create-branch` 스킬 박제는 TASKS M1 "골든패스 패턴 박제" 이슈에 포함 (D-012)
