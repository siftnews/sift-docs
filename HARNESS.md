# Sift — 하네스 엔지니어링 진행 방식

> Sift는 일회성 코드 생성이 아니라 **에이전트 하네스 위에서** 개발한다.
> 즉 반복 가능한 골격(규칙·스킬·서브에이전트·평가 루프·메모리)을 갖추고 그 위에서 기능을 찍어낸다.
> 최종 수정: 2026-07-16

---

## 0. 원칙

1. **골든패스 먼저, 추상화는 나중.** 박제할 패턴이 없는 상태에서 스킬/규칙부터 만들면 헛돈다.
   → 수직 슬라이스 1개를 손으로(=에이전트와 함께) 완성 → 그 패턴을 박제 → 이후 반복.
2. **2번 했으면 박제한다.** 같은 형태의 작업을 두 번째 할 때, 스킬/CLAUDE.md 규칙/서브에이전트로 굳힌다.
3. **컨텍스트는 외부화한다.** 결정·설계는 `sift-docs/`(공통)·각 레포 `docs/`(설계)와 메모리에 남겨 다음 세션이 cold start 하지 않게 한다.
4. **하네스 산출물도 리뷰 대상이다.** 스킬·규칙·에이전트도 코드처럼 검증한다.
5. **측정을 먼저 깐다.** 개선할 대상엔 베이스라인부터. evals·메트릭을 기능과 *함께*(나중이 아니라) 깐다. 측정 없이 개선 없음.
6. **워크플로우로 충분하면 에이전트를 쓰지 않는다.** 단순함이 우선 (§0.5).

---

## 0.5 워크플로우 vs 에이전트 (작업 분류)

> Anthropic 에이전트 설계의 핵심 구분. **예측 가능한 작업은 워크플로우로, 자율 판단이 필요할 때만 에이전트로.**

| 분류 | 성격 | Sift 작업 | 하네스 산출물 |
|---|---|---|---|
| **워크플로우** | 경로가 사전에 정해짐, 결정론적 → *재현성*이 가치 | 유스케이스 풀구현, 배치 Job 추가, 모듈 추가, 토픽·소스 등록, 포맷·테스트 게이트 | 스킬 · 훅 · 고정 절차 |
| **에이전트** | 경로가 사전에 안 정해짐, 탐색·판단 필요 → *적응성*이 가치 | 성능 병목 분석, 선별 가중치 튜닝, 장애 원인 추적, 코드 리뷰 | 서브에이전트 |

- 구현·반복 작업은 입력이 같으면 결과도 같아야 하므로 **워크플로우(스킬/훅)**.
- 분석·튜닝은 입력(로그·메트릭)이 매번 다르고 경로가 열려 있으므로 **에이전트**.
- 애매하면 워크플로우부터. 에이전트는 워크플로우로 표현이 안 될 때만 승격한다.

---

## 0.6 루프 프로토콜 (한 사이클의 절차)

> Sift를 `/loop` 또는 자기페이스 반복으로 개발할 때, 매 사이클은 이 절차를 따른다.
> 루프의 의지처: **STATE.md**(나침반) · **BACKLOG.md**(작업) · **DECISIONS.md**(일관성) · 메모리(영속).

**한 사이클**
1. **상태 읽기** — `sift-docs/STATE.md` + 메모리로 현재 Phase·진행·다음 액션 파악
2. **태스크 선택** — `sift-docs/BACKLOG.md` 우선순위 최상단의 미완 작업 1개. DoD·의존성 확인
3. **실행** — 분류(§0.5)에 따라 워크플로우(스킬/고정 절차) 또는 에이전트(분석/탐색)
4. **검증** — 컴파일·`build`·`test`는 에이전트가 자가검증. 되돌리기 어려운 명령·배포는 게이트(사람)
5. **커밋** — 작은 작업 1개 완료·검증 통과 시 🤖 커밋 실행 → 다음 작업 (D-026 — 형식·트레일러는 가드 훅이 검사)
6. **기록·갱신** — STATE 갱신, BACKLOG 체크, 결정 났으면 DECISIONS 추가, 필요시 메모리

**게이트 (사람 개입 지점)**
- 되돌리기 어려운 명령 — `git reset --hard`, `git clean`, `docker compose down -v` (settings.json deny)
- git/GitHub 쓰기 중 **병합·공개 상태 변경·인프라** — `gh pr merge`·`review`·`ready`, issue/pr `close`·`comment`·`delete`, `release`, repo·workflow·secret·variable (D-026, settings deny). issue/pr `edit`는 🤖 허용이되 **assignee·라벨 관리 용도로만** — 제목·본문 수정은 사람 (D-028). **커밋·push·이슈 발행·PR 생성은 🤖 에이전트 실행** — 모든 기록은 사용자 명의·기존 스타일(§0.7 컨벤션 준수 — 검사 장치 없음, D-027). main push 방어 = GitHub 브랜치 보호
- 외부 영향 — 스키마 파괴, 외부 발송, 배포
- 설계 분기·모호성 — 추정 말고 질문
- (build·test·컴파일은 게이트 아님 — 에이전트 자가검증)

**중단 조건 (멈추고 사람에게)**
- 백로그 최상단이 게이트 대기 / 같은 작업 연속 실패(2회) / 요구 모호 / 백로그 비었음

**자기검증** — 사이클 종료 전, 선택 작업의 DoD 충족 여부를 STATE에 명시한다.

---

## 0.7 협업 흐름 (GitHub) — 작업 단위 = 이슈 → PR → 리뷰 → 병합

> **이슈 1개 = 루프 1회분 = PR 1개.**
> **역할 분담 (D-026)**: 이슈 발행·브랜치·구현·자가검증·**커밋·push·PR 생성**·gh 읽기는 🤖 *에이전트* — 단 모든 기록은 사용자 명의·기존 스타일(아래 컨벤션을 스스로 준수 — 검사 장치 없음, D-027). **병합·리뷰 승인·issue/pr close·comment·release·repo/인프라 쓰기**는 👤 *사람* (settings deny). issue/pr `edit`는 🤖 assignee·라벨 관리 용도로만 허용 — 제목·본문 수정은 👤 (D-028).
> **main 직접 커밋·push 금지** (GitHub 브랜치 보호로 강제 — D-027).

**태스크 계층 (재귀 분해의 GitHub 매핑)**

| 계층 | GitHub | 원본 문서 |
|---|---|---|
| Phase/기능 묶음 | 마일스톤 | `TASKS.md` M1~M4 |
| 태스크 (= PR 1개 분량) | 이슈 | `TASKS.md` 이슈 후보 |
| 태스크 내 하위 단계 | 이슈 본문 체크리스트 | `BACKLOG.md` (루프가 집는 단위) |

- 태스크가 PR 1개 분량을 넘으면 **이슈를 분할**하고 TASKS.md에 반영한다.
- 이슈 제목 포맷: `[FEAT|CHORE|FIX|REFACTOR|docs] 요약` — 실제 발행 관행 박제 (예: 이슈 #10 `[FEAT] Source 영속 어댑터`, 이슈 #1 `[chore] 프로젝트 골격 & 모듈 경계`). 본문 구조는 아래 템플릿.

**이슈·PR 본문 템플릿 (원본 = 조직 [`.github`](https://github.com/siftnews/.github) 레포 — 로컬 클론 `siftnews/.github/`)**
- ⚠️ 비대화식 `gh issue/pr create --body`는 GitHub 템플릿을 자동 적용하지 않는다 — 아래 구조로 본문을 직접 작성한다. 헤딩 원문은 로컬 클론 `.github/.github/`의 [ISSUE_TEMPLATE](https://github.com/siftnews/.github/tree/main/.github/ISSUE_TEMPLATE)·[pull_request_template.md](https://github.com/siftnews/.github/blob/main/.github/pull_request_template.md) 참조.
- **이슈 본문**: `## Description`(한 줄 "- …를 구현합니다.") + `## TODO`(체크박스 = 하위 단계, DoD 포함). 라벨은 타입과 일치(`feature`·`chore`·`documentation` 등).
- **PR 본문**: `## Issue number`(`- close #N`) + `## Check list`(테스트 통과 확인 · 모든 commit push 확인 · merge branch 확인) + `## (Optional) Additional description`(대체로 비움). 제목은 이슈 제목 그대로.
- **Assignees·Labels**: 이슈·PR 모두 create 시점에 지정 — `--assignee @me`(사용자 본인) + 타입 일치 라벨 `[FEAT]`→`feature` · `[CHORE]`→`chore` · `[docs]`→`documentation` · `[FIX]`→`bug`. 누락 시 `gh issue/pr edit --add-assignee @me --add-label {라벨}`로 소급 가능 (D-028 — `edit`는 assignee·라벨 용도로만, 제목·본문 수정은 사람).

**절차**
1. 🤖 **이슈 초안 → 발행** — TASKS.md에서 다음 태스크 선택 → 제목(`[FEAT] ...`)·배경·DoD 작성 → 사용자 확인 후 `gh issue create --assignee @me --label {타입 라벨}` 실행 (D-026) → TASKS/BACKLOG에 `#번호` 태깅
2. 🤖 **브랜치** — `develop`에서 `feature/{이슈번호}-{영어-kebab-case}` 분기 (예: `feature/12-rss-feed-adapter`). **develop = 통합, main = 배포** (D-025)
3. 🤖 **구현 루프** — BACKLOG로 분해 → 작은 작업마다: 구현 + build/test 자가검증 → 커밋 (`{type}: {한국어 요약}`) → 다음 작은 작업
4. 🤖 **push + PR 생성** — `git push -u origin feature/...`(대상 명시 관행) → 자가 리뷰(`/code-review`) → `gh pr create --assignee @me --label {타입 라벨}` (base = develop, 본문에 `close #N` + 체크리스트)
5. 👤 **리뷰** — CodeRabbit 리뷰 확인, 필요시 `ultrareview`(사람만 트리거 가능 — 과금·사용자 전용). 리뷰 지적 반영은 🤖 후속 커밋
6. 👤 **병합** — 리뷰 승인 후 병합 (merge commit — 현 관행) → 🤖 develop 갱신 확인, STATE·TASKS 체크
7. 👤 **배포 승격** — 마일스톤 단위로 develop → main PR (D-025)

**커밋 컨벤션 (실행 🤖 — D-026)**
- 형식: `{type}: {한국어 요약}` — type: `feat` | `fix` | `refactor` | `test` | `chore` | `build` | `docs`. 한 줄, **트레일러 없음(Co-Authored-By 등 에이전트 서명 금지 — 기록은 사용자 명의)**.
- 단위: **작은 작업(BACKLOG 단계) 1개 = 커밋 1개.** 여러 단위가 워킹트리에 쌓였으면 파일 그룹별로 나눠 순서대로 커밋한다.

**규칙**
- **이슈 입도 = PR 1개로 리뷰할 분량.** 미세 작업은 이슈로 만들지 말고 BACKLOG로, 거대 작업은 이슈를 분할.
- main 직접 push/커밋 금지 (브랜치 보호로 강제)
- 리뷰 승인 없이 병합 금지
- PR은 한 작업 단위로 작게, 이슈 없는 PR 지양 (추적성)

---

## 0.8 멀티세션 운영 (터미널 탭 병렬 — D-016)

> 여러 터미널 탭에 역할별 세션을 병렬로 띄워 쓸 수 있다 (예: 📝 문서화 / 🔧 구현 / 👀 리뷰).
> 각 세션은 독립 컨텍스트, 하네스(CLAUDE.md·settings.json·메모리)는 공유. 리뷰를 별도 세션에 두는 이유는 **컨텍스트 격리** — 구현한 세션이 자기 코드를 리뷰하면 편향된다.

**규칙**

1. **모든 세션은 `siftnews/` 루트에서 실행한다.** 프로젝트 메모리는 실행 위치(cwd) 기준으로 잡히므로, 실행 위치가 다르면 메모리가 갈라지고 게이트(settings.json deny) 적용도 어긋난다.
2. **STATE·BACKLOG·TASKS·DECISIONS의 쓰기 주인은 구현(루프) 세션 1개.** 이 파일들은 잠금 없는 단일 진실원천이라 동시 편집 시 나중 쓰기가 먼저 쓰기를 덮는다. 다른 세션은 **읽기 전용** — 작업 결과는 사용자에게 보고하고, 기록이 필요하면 사용자가 루프 세션에 전달해 반영한다 (모든 갱신은 한 세션을 통과).
3. **같은 코드 파일을 두 세션이 동시에 편집하지 않는다.** 리뷰 세션은 읽기 전용으로 운영. 구현 세션이 2개 이상 필요해지면 탭 분리가 아니라 **git worktree**로 작업 트리를 분리한다.
4. **리뷰 결과를 즉시 수정에 반영하는 흐름**이 필요하면 별도 탭 대신 구현 세션 안의 서브에이전트/`/code-review`가 더 매끄럽다. 탭 분리는 격리가 이득일 때만.

---

## 0.9 설계 문서화 규칙 (다이어그램 포함 — D-017)

> 구현 전에 설계를 **제대로 그리고, 그린 것을 남긴다.** 설계 산출물도 코드처럼 하네스의 1급 산출물이다.

**산출물 종류와 도구**

| 산출물 | 도구 | 이유 |
|---|---|---|
| 컨텍스트 맵, 아키텍처 오버뷰 (자유 배치·표현력 필요) | **draw.io** — `.drawio.svg` 형식 | GitHub에서 이미지로 렌더되면서 draw.io로 재편집 가능. 사람이 그린다 |
| ERD, 상태 머신, 시퀀스, 배치 플로우 (구조가 정형) | **Mermaid** (문서 내 코드블록) | 텍스트라 diff 가능, GitHub 자동 렌더, **에이전트가 직접 작성·갱신 가능** |

- 기본값은 Mermaid. draw.io는 Mermaid로 표현이 옹색한 자유형 다이어그램에만.
- 위치 (D-021): **레포 종속 설계 문서·다이어그램**(ERD·이벤트·선별 등)은 해당 `sift-*` 레포의 `docs/`가 원본 — 구조 변경 PR에서 코드와 같은 diff로 리뷰한다.
  **워크스페이스 공통 문서**(PLAN·HARNESS·루프 문서)는 `sift-docs` 레포(= 루트 `sift-docs/`)가 원본으로 공개된다. 스냅샷 사본은 만들지 않는다(drift 방지) — 크로스 레포 참조는 절대 URL로 링크한다.

**규칙**

1. **설계 먼저, 구현은 그 다음.** 구조에 영향 주는 이슈(새 모듈·새 유스케이스·스키마 변경)는 착수 전에 해당 설계 산출물(다이어그램 포함)을 먼저 작성·갱신한다.
2. **다이어그램도 정합 대상.** 구조를 바꾸는 이슈의 DoD에 관련 다이어그램 갱신을 포함한다 — 코드와 어긋난 다이어그램은 없느니만 못하다.
3. **결정은 DECISIONS, 구조는 다이어그램, 이유는 문서 본문** — 셋을 서로 링크한다.

---

## 1. 하네스 구성요소 (Sift 매핑)

| 요소 | 분류 | Sift에서의 모습 | 상태 |
|---|---|---|---|
| 규칙 주입 | — | `siftnews/CLAUDE.md` (멀티레포 워크스페이스 루트) — Modulith·헥사고날 규칙·개발 명령 | ✅ 완료 |
| 권한·훅 | — | `.claude/settings.json` (allow/deny — D-026 재편, 훅 없음 — 가드 훅은 D-027로 폐지). 루트·sift-api 사본 정합 유지 (D-023) | ✅ 완료 |
| 반복작업 스킬 | 워크플로우 | "유스케이스 풀구현", "소스 어댑터 추가", "배치 Job 추가", "토픽 등록" | Phase 1 |
| 측정 골격 | — | Actuator 메트릭 + 배치 처리량·소요시간 로깅 (evals 토대) | **Phase 0~1 (앞당김)** |
| 역할 서브에이전트 | 에이전트 | 선별-튜닝 / 성능-측정·분석 / 리뷰 | Phase 2 |
| 평가 루프 | 에이전트 | 선별 품질 eval, 발송 성능 벤치 (측정→분석→개선) | Phase 2 (토대는 일찍) |
| 컨텍스트 영속화 | — | `sift-docs`(PLAN·HARNESS·루프 문서) + 각 레포 `docs/`(설계 문서) + 메모리 | 진행 중 |

---

## 2. 로드맵

> **진행 상태는 이 문서에서 추적하지 않는다** — 상태의 원본은 [STATE.md](./STATE.md)·[TASKS.md](./TASKS.md) (D-023).
> 이 절은 각 Phase의 *구성*만 정의한다.

### Phase 0 — 스캐폴딩 + 골든패스
- `sift-api` Spring Boot 3.5 / Java 21 / Gradle + **Spring Modulith** 채택
- 의존성(batch·jpa·postgres·validation·actuator·modulith·testcontainers), `docker-compose.yaml`(PostgreSQL + **mailpit**), 프로파일 분리(`local`), 기동 확인
- 하네스 토대: `siftnews/CLAUDE.md` + `.claude/settings.json`
- **측정 베이스라인 골격** — Actuator 메트릭 + 배치 처리량·소요시간 로깅 (원칙 5: evals 앞당김)
- **골든패스 = `collectionJob` 슬라이스** (범위 정의는 §3, 구현 순서는 D-020: DDD inside-out)
- 모듈 검증 테스트 `ApplicationModules.verify()`

### Phase 1 — 기능 구현 하네스 박제
- 골든패스에서 확립된 패턴을 `sift-api/CLAUDE.md`로 규칙화
- "헥사고날 유스케이스 풀구현" 스킬화 (보유 스킬 implement-feature 재활용·확장)
- 두 번째/세 번째 유스케이스(selectionJob, dispatchJob)는 하네스로 생산하며 검증

### Phase 2 — 성능·선별 평가 하네스 (본격 개선 사이클)
> 측정 인프라(메트릭·로깅·시드)는 Phase 0~1에 미리 깐다(원칙 5). Phase 2는 그 토대 위에서 사이클을 돈다.
- **발송 성능**: 10만 구독자 시드 → 부하 측정 → 병목 분석 → V1→V2→V3 개선 (PLAN §6)
- **선별 품질**: breakdown 로그 기반 eval, 가중치 A/B
- 측정→분석→개선 사이클을 **서브에이전트(자율)** + `/loop`·스케줄로 반자동화
- 각 사이클 결과를 `sift-docs/`에 기록 → 포트폴리오 본문

---

## 3. 첫 골든패스: `collectionJob` 수직 슬라이스 (선정 이유)

> 구현 *순서*는 D-020으로 재편됨 — 슬라이스 일괄 구현이 아니라 **DDD inside-out**(도메인→유스케이스→영속→RSS→배치, TASKS M1-2~7).
> 이 절은 골든패스의 *범위*(어떤 레이어 조각으로 구성되는가) 정의로 유지된다.

`Source` 컨텍스트의 수집 슬라이스를 첫 골든패스로 삼는다:

```
domain      Source, Article
port.in     CrawlSourcesUseCase
port.out    LoadActiveSourcesPort, FetchFeedPort(RSS), SaveArticlePort
service     CrawlSourcesService (UseCase 구현)
adapter.in  collectionJob (Spring Batch — 인바운드 어댑터)
adapter.out RssFeedAdapter, ArticleJpaAdapter
```

- **배치 어댑터 패턴**을 담는다 → Sift의 핵심 반복 단위(이후 selection/dispatch도 동일 패턴)
- **outbound port 2종**(외부연동 + 영속) 패턴을 한 번에 확립
- 도메인→포트→서비스→어댑터 **전 레이어 관통**
- 의존성 최소 — 소스만 있으면 독립 실행 가능(발송보다 선행 데이터 부담 없음)

→ 이 슬라이스가 Phase 1에서 "유스케이스 구현 하네스"의 원형(템플릿)이 된다.
