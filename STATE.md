# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-07

## 현재 Phase
**Phase 0 — 스캐폴딩 + 골든패스** (진행 중)

## 지금 (in progress)
- **M1-2 `[feature] Source 도메인 모델` 구현 시작 (2026-07-07)** — 이슈 발행·브랜치 체크아웃 완료(👤). 클래스 다이어그램 확정(class diagram 페이지).
- **⚠️ 멀티세션 역할 전환 (D-016)**: 루프 주인(루프 문서 쓰기) = **구현 세션**(별도 탭)으로 이관. 기존 세션은 설계·문서화·리뷰 역할로 전환 — 이후 루프 문서 읽기 전용. **구현 세션: 이슈 번호를 TASKS M1-2에 태깅하고 BACKLOG를 분해한 뒤 시작할 것.**

## 다음 액션 (next)
- 👤 **잔여 브랜치 정리**: 로컬·원격 `develop` 삭제 + main 브랜치 보호 규칙 설정 (동기화 자체는 완료, 정리만 남음)
- 👤 컨텍스트 맵 마무리: BounceDetected에 "(2차)" 표기만 남음 (IssueScheduled 제거·DeliveryJobCompleted 라벨 변경은 완료 확인 ✅ 2026-07-07)
- 👤 **클래스 다이어그램 작성** (Source 도메인 — M1-2 선행, D-017) → 🤖 검증
- 👤 **M1-2 이슈 발행** (초안 전달됨)
- 🤖 M1-2 발행되면 BACKLOG 분해 → 구현 착수
- 🤖 다음 이슈 브랜치에서 `.coderabbit.yaml` path 항목 수정 반영 (아래 비고)

## 정정 (2026-07-05 하네스 검토)
- **source 도메인/포트 골격은 실제로 존재하지 않음** — 브랜치·전체 히스토리·stash에 `Source`/`Article` 도메인·포트 코드 없음(`source/package-info.java`만 존재). 이전 STATE의 "source 골격 커밋" 기록은 오기(커밋 정리 과정에서 유실 추정). BACKLOG C 체크 해제, 다음 이슈 범위에 도메인/포트 생성 포함.
- 이슈 #1 커밋은 4개가 아니라 **3개** (common / 모듈경계+검증테스트 / 테스트인프라).

## 최근 완료
- **develop→main 동기화 확인 (2026-07-07)** — `sift-api` 실 저장소 점검 결과 PR #3(develop→main)이 이미 병합되어 main·develop 모두 `98ec1d5` 동일 커밋. `origin/HEAD`도 main. STATE의 기존 "동기화 대기" 기록은 정정.
- **M1 재편 확정 (D-020, 2026-07-07)** — DDD inside-out 순서(도메인→유스케이스→영속→RSS→배치→박제, M1-2~7) + 단계별 테스트 DoD. 수직 슬라이스 안 기각(사용자 방법론 결정). TASKS·BACKLOG 반영
- **EVENTS.md 명세 완성 (2026-07-07)** — `DeliveryJobCompleted`(MVP, job 단위 + 멱등 upsert) · `BounceDetected`(2차, HARD/SOFT 판정) 작성, `IssueScheduled` 검토 기록 복원. 공통 규약(at-least-once + 구독자 멱등) 헤더 명시. 열린 항목 2개는 본문에 태깅(DONE 정의 → M3, SOFT 임계 N → 2차)
- **D-019: 발송 모델 확정 (2026-07-07)** — 매일 DAILY 고정 + `subscriber.preferred_send_hour`(구독자 선택 시각), dispatch = 매시 pull 스캔, IssueScheduled 이벤트 MVP 제외. topic.cadence/send_at_hour 제거. MVP-DESIGN(§0·§1·§2·§3·§5)·PLAN·EVENTS 반영
- **D-018: Article = Source 소유 결정 (2026-07-07)** — 바운디드 컨텍스트 매핑 중 소유권 모호(PLAN vs MVP-DESIGN) 발견 → Source 소유 + Content는 named interface 조회로 확정. PLAN·MVP-DESIGN·SELECTION 정정. 열린 꼬리: dedup_cluster_id 갱신 주체 (M2에서)
- **설계 문서화 규칙 박제 (2026-07-07)** — HARNESS §0.9 + D-017: 구조 변경은 다이어그램 먼저(Mermaid 기본, 자유형은 draw.io `.drawio.svg`), 구조 이슈 DoD에 다이어그램 갱신 포함. 다음 작업 = 바운디드 컨텍스트 검증(첫 적용)
- **이슈 #1 병합 (PR #2 → develop, 2026-07-06)** — 리뷰 반영 리팩터 4커밋(Clock 주입 방식, package-info, 테스트 인프라 상수, 모듈 검증 테스트) + CodeRabbit 설정 3커밋 포함. main 동기화만 남음
- **CodeRabbit 리뷰 설정 작성 (2026-07-05~06)** — Sift 기준 전면 개정(모듈 경계·헥사고날·상태머신·멱등성·관측성·테스트 규칙), `sift-api/.coderabbit.yaml`
- **멀티세션 운영 규칙 박제 (2026-07-05)** — HARNESS §0.8 신설 + D-016: 역할별 탭 병렬 허용, 루트에서 실행, 루프 문서 쓰기는 구현 세션 단일 창구, 코드 동시 편집 금지(필요시 worktree)
- **PORTFOLIO.md 작성 (2026-07-05)** — 포트폴리오 강조 포인트 4개 카테고리를 Sift 실증 지점(V1~V4·마일스톤)에 매핑 + 운영 규칙(이슈 초안 시 확인·사이클 태깅). CLAUDE.md·HARNESS §0.6/§2에 참조 연결
- **하네스 정합성 검토·개선 (2026-07-04)**:
  - 이슈 #1 코드를 `feature/1-module-boundaries`에 3개 커밋으로 정리 (common / 모듈경계+검증테스트 / 테스트인프라), `./gradlew build` 통과 ✅
  - push 게이트 개정 → **D-011** (main 보호 기반, feature push는 보호 설정 후 허용)
  - 타 프로젝트 글로벌 스킬 7개 `~/.claude/skills-archive/`로 격리, 브랜치 전략 = GitHub Flow 명명 정정 → **D-012**
  - HARNESS §0.6·§0.7 정정 (ultrareview는 사람만 트리거 가능), "모노레포" → "멀티레포 워크스페이스" 용어 통일
- **이슈 #1 `[chore] 프로젝트 골격 & 모듈 경계` — 코드 완료 ✅** (`ApplicationModules.verify()` 통과)
  - `common` 모듈: `BaseEntity`(JPA Auditing), `ClockConfig`, `JpaAuditingConfig`, `BusinessException`
  - 5개 모듈 경계 선언(`allowedDependencies={"common"}`) + `ApplicationModulesTest`
  - 테스트 인프라: testcontainers + `AbstractIntegrationTest` + `application-test.yaml`
- 스캐폴딩(Spring Boot 3.5 + Modulith), 의존성, `docker-compose.yaml`(postgres/mailpit), 프로파일 분리, **기동 확인 ✅**
- 하네스 토대: `siftnews/CLAUDE.md`, `.claude/settings.json` / 루프 운영 골격(STATE·BACKLOG·DECISIONS) / 협업 흐름 규칙(HARNESS §0.7, D-010)

## 대기 / 블록 (게이트)
- ~~develop→main 동기화~~ — 완료 확인 (2026-07-07, PR #3)
- **develop 삭제** (사람) — 로컬·원격 모두 (동기화는 끝났으나 브랜치 자체는 아직 존재)
- **main 브랜치 보호 규칙** 설정 (사람)
- **다음 이슈 발행** (사람) — Source 이슈 초안 전달됨
- ~~이슈 #1 push·PR~~ — 완료 (PR #2 병합)
- ~~루트 `siftnews/` git 저장소화~~ — **취소 (D-015)**: 루트는 로컬 전용, git은 하위 sift-* 레포에만

## 비고
- **역할 분담 (D-013·D-014)**: git/GitHub 쓰기(**커밋**·이슈 발행·push·PR·병합) = 사람. 태스크 트리(TASKS.md)·이슈/PR/**커밋 메시지** 초안·브랜치·구현·자가검증 = 에이전트. gh CLI는 에이전트 필수 아님.
- **구현 리듬 (D-014)**: 작은 작업 1개 → build/test 자가검증 → 에이전트가 커밋 메시지 제안 → 사람 커밋 → 다음 작업. 메시지 형식 `{type}: {한국어 요약}`.
- 게이트(settings.json deny): git commit / push, gh issue·pr create / pr merge, git reset --hard / clean, docker compose down -v.
- `.coderabbit.yaml` 후속 수정 대기: 병합본의 `src/main/resources` path 항목에 instructions 누락(무효 항목). 수정본은 스크래치패드 `coderabbit-final.yaml`에 보관 — 다음 이슈 브랜치에서 반영.
