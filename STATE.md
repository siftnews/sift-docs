# Sift — STATE (루프 나침반)

> 매 사이클 **시작에 읽고, 종료에 갱신**한다. 현재 상황의 단일 진실원천.
> 프로토콜: [HARNESS.md §0.6](./HARNESS.md) · 마지막 갱신: 2026-07-19

## 현재 Phase
**Phase 0 — 스캐폴딩 + 골든패스** (진행 중)

## 지금 (in progress)
- (착수 대기) **M1-6 `[FEAT] collectionJob 배치 + 측정 베이스라인`** — 다음 태스크. 이슈 발행(사용자 확인 후) → BACKLOG 분해(착수 전) → 구현 순서 (§0.7)

## 다음 액션 (next)
- 🤖 **M1-6 이슈 초안 작성 → 사용자 확인 → 발행** — `gh issue create --assignee @me --label feature` (D-028 컨벤션 첫 적용) → BACKLOG 분해 후 착수
- 👤 **main 브랜치 보호 규칙 정비** — 가드 훅 폐지(D-027)로 main 방어는 서버측 브랜치 보호만 남음. sift-docs main 포함, 사용자 진행 중 (2026-07-19)
- 👤 **`~/.claude/settings.local.json`에서 `Bash(gh api *)` 줄 제거** — D-024 백스톱 무력화 구멍, 전역 보호 훅 때문에 에이전트가 편집 불가 (D-026 비고)
- 👤 **PORTFOLIO.md 유실 처리 결정** — 복원(재작성) 또는 폐기 (아래 "정정" 참조)

## 정정 (사람 재검증·수정 흔적)
- **(2026-07-14) PORTFOLIO.md 실종** — 최근 완료(2026-07-05)에 작성 기록이 있으나 워크스페이스·sift-docs 어디에도 실물 없음. `docs/` → `sift-docs/` 이전(D-021) 과정 유실 추정 (루트 git 부재 시기라 이력 없음 — D-015 트레이드오프 실증 2번째 사례). 복원 여부는 사용자 결정 대기.
- **(2026-07-14) `.coderabbit.yaml` path 결함은 이미 해소** — 기존 비고의 "후속 수정 대기"는 죽은 기록: 98ec1d5(2026-07-06)에서 path 병합으로 수정 완료됨을 git 이력으로 확인. 스크래치패드의 `coderabbit-final.yaml` 사본도 유실 확인(세션 임시 디렉터리 소멸) — 이후 세션 간 임시물은 `.omc/notepad` 사용 (D-023).

### (2026-07-05 하네스 검토)
- **source 도메인/포트 골격은 실제로 존재하지 않음** — 브랜치·전체 히스토리·stash에 `Source`/`Article` 도메인·포트 코드 없음(`source/package-info.java`만 존재). 이전 STATE의 "source 골격 커밋" 기록은 오기(커밋 정리 과정에서 유실 추정). BACKLOG C 체크 해제, 다음 이슈 범위에 도메인/포트 생성 포함.
- 이슈 #1 커밋은 4개가 아니라 **3개** (common / 모듈경계+검증테스트 / 테스트인프라).

## 최근 완료 (최근 4~5건만 유지 — 이전 이력은 git 히스토리, D-023)
- **M1-5 `[FEAT] RSS 수집 어댑터` 병합 ✅ (이슈 #12 → PR #13 → develop, 2026-07-19)** — rometools 의존성 + `RssFeedAdapter`(`FetchFeedPort` 구현, RSS 파싱 → `RawArticle` 변환) + 픽스처 4케이스 단위 테스트. 로컬 develop 동기화·feature 브랜치 정리 완료
- **gh issue/pr edit 허용 + assignee·라벨 컨벤션 박제 (2026-07-19, D-028)** — edit를 deny→allow(assignee·라벨 관리 용도 한정, 제목·본문은 사람 — deny 완화는 분류기 차단으로 사용자가 직접 적용). 이슈 #12 `feature` 라벨·PR #13 assignee/라벨 소급 지정 완료. create 시 `--assignee @me --label {타입 라벨}` 컨벤션 HARNESS §0.7 박제
- **가드 훅 폐지 (2026-07-19, D-027 — D-026 개정)** — 훅 검토에서 우회 경로(`git push origin HEAD`·`+` refspec·`--amend` 형식 미검사)·fail-open 한계 확인 → 사용자 결정으로 `git-gh-guard.sh` 삭제·settings 훅 등록 해제(두 사본 정합 유지). 형식 준수는 지침·리뷰로, main 방어는 GitHub 브랜치 보호로 일원화(👤 규칙 정비 진행 중). 부수: 조직 `.github` 템플릿 레포를 워크스페이스 루트에 클론, 이슈·PR 본문 템플릿 구조를 HARNESS §0.7에 박제(2026-07-19)
- **하네스 개편 — git/GitHub 쓰기 위임 + 가드 훅 (2026-07-17, D-025·D-026)** — ① 브랜치 전략 확정: develop 통합·main 배포(D-025, develop 게이트 해소) ② 쓰기 위임: 이슈·PR 생성·커밋·push를 에이전트 실행으로 승격, 병합·edit·close·comment·release·인프라는 deny 유지 ③ `.claude/hooks/git-gh-guard.sh` 신설(main push 차단·force 금지·push 대상 명시 강제·커밋 메시지 형식·트레일러 금지·이슈/PR 제목·close #N 검사 — 19케이스 검증 통과) ④ settings 두 사본 재편·정합(`git branch` 표기 불일치 해소 포함) ⑤ HARNESS §0.6·§0.7·§1 개정. 계기: gh 이슈·PR 스타일 정독(2026-07-17) 후 사용자 결정
- **M1-4 `[FEAT] Source 영속 어댑터` 병합 ✅ (이슈 #10 → PR #11 → develop, 2026-07-16)**
- (이전 완료 이력은 이 레포 git 히스토리로 조회 — D-023: STATE는 최근 4~5건만 유지)

## 대기 / 블록 (게이트)
- ~~develop 처리 결정~~ — **해소 (2026-07-17, D-025)**: develop = 통합 브랜치, main = 배포 브랜치로 공식화. develop이 앞서 있는 것은 정상(배포 전 통합분), 승격은 마일스톤 단위로 사람이 결정
- ~~main 브랜치 보호 규칙 설정~~ — **완료 확인 (2026-07-15, D-024 비고)**: sift-api main에 PR 필수·strict checks 설정됨 (gh api로 검증)
- ~~#6 PR 발행·병합~~ — 완료 (PR #7 병합, 2026-07-10)
- ~~M1-3 이슈 발행~~ — 완료 (이슈 #8 → PR #9 병합, 2026-07-11)
- ~~이슈 #1 push·PR~~ — 완료 (PR #2 병합)
- ~~루트 `siftnews/` git 저장소화~~ — **취소 (D-015)**: 루트는 로컬 전용, git은 하위 sift-* 레포에만

## 비고
- **역할 분담 (D-026)**: 이슈 발행·브랜치·구현·자가검증·**커밋·push·PR 생성** = 에이전트 (모든 기록은 사용자 명의·기존 스타일 — HARNESS §0.7 컨벤션을 스스로 준수, 검사 장치 없음 D-027). **병합·리뷰 승인·issue/pr close·comment·release·repo/인프라 쓰기** = 사람. issue/pr `edit`는 에이전트 허용이되 assignee·라벨 관리 용도만 (D-028).
- **구현 리듬 (D-026)**: 작은 작업 1개 → build/test 자가검증 → 에이전트 커밋(`{type}: {한국어 요약}` 한 줄, 트레일러 금지) → 다음 작업.
- 게이트(settings.json deny): gh pr merge/review/ready, issue·pr close/comment/delete, release, repo, workflow run, secret, variable (edit는 D-028로 allow — assignee·라벨 용도만). 파괴 명령: git reset --hard / clean, docker compose down -v. main push 방어 = GitHub 브랜치 보호(D-027). `gh api`는 allow/deny 양쪽 제외(백스톱) — ⚠️ settings.local.json의 `gh api *` allow 제거는 사람 대기.
