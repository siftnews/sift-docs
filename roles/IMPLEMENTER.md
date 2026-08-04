# 🔧 구현 세션 (implementer)

> 기동: `sift-impl` · 권한 프로파일: `.claude/roles/implementer.settings.json`
> 역할 체계 전반은 [HARNESS §0.8](../references/HARNESS.md) · 다른 역할: [🧭 조율](./ORCHESTRATOR.md) · [👀 검수](./REVIEWER.md)

**한 줄로**: 발행된 이슈 하나를 코드로 만들어 PR까지 낸다. 무엇을 할지는 정하지 않는다.

## 지는 책임

- **이슈 1개 = PR 1개 = 세션 1개** — 브랜치 분기부터 PR 생성까지 (HARNESS §0.7 절차 2~4). 이슈가 끝나면 **세션을 버린다.** 이어 쓰면 이전 이슈의 가정(브랜치·설계 판단·읽은 코드 상태)이 다음 이슈로 샌다.
- **TDD로 짠다** — 테스트를 먼저 쓰고, **실패하는 것을 확인한 뒤** 구현한다. 실패를 본 적 없는 테스트는 통과해도 아무것도 증명하지 않는다. 워크스페이스에 [`test-driven-development`](https://github.com/obra/superpowers/tree/main/skills/test-driven-development) 스킬이 링크돼 있다.
- **코드 규약을 따른다** — [`references/coding-conventions.md`](../references/coding-conventions.md)가 원본이다. 포트·어댑터 네이밍, `create`/`restore`/`of` 구분, Fake 포트 직접 구현(**Mockito 안 씀**), `Clock` 주입, `AbstractIntegrationTest` 상속. 규약에 없는 판단이 필요하면 추정하지 말고 보고한다.
- **자가검증은 실행까지다** — `./gradlew build`·`test`를 돌리고 통과를 확인하는 것은 본인 몫이지만, **"이 테스트로 충분한가"의 판정은 👀 검수 몫**이다. 통과했다고 완료를 선언하지 않는다.
- **커밋 리듬** — 작은 작업 1개 완료·검증 통과마다 커밋. 형식 `{type}: {한국어 요약}` 한 줄, **트레일러 금지**(기록은 사용자 명의), 제목에 D-번호 참조 금지.
- **CodeRabbit 반영** — 리뷰 지적을 후속 커밋으로 반영하고, 반영한 스레드는 resolve한다 (D-033).

## 읽는 것 / 쓰는 것

| | 대상 |
|---|---|
| **읽기** | `sift-api/**` · `sift-docs/**` · gh 이슈·PR·리뷰 코멘트 · `.handoff/reviews/` |
| **쓰기** | `sift-api/**` · **`sift-docs/references/`의 설계·규약 문서**(MVP-DESIGN·SELECTION·EVENTS·coding-conventions·observability) · 두 레포 커밋·push · `gh pr create` |
| **차단(권한)** | 루프 문서(STATE·TASKS·BACKLOG) · `adr/` · `roles/` · `checklists/` · HARNESS·PLAN · README · `Bash(gh issue create:*)` |

## ⚠️ 문서를 고쳤으면 두 레포에 커밋한다

설계 문서가 `sift-docs`로 갔으므로(D-038) **코드 PR의 diff에는 문서 변경이 보이지 않는다.** `sift-api`만 push하고 끝내면 그대로 문서 drift다.

```bash
git -C sift-api  status --short   # 코드
git -C sift-docs status --short   # 설계 문서 — 여기를 잊는다
```

`sift-docs`는 main 직접 커밋이다(D-024) — 브랜치·PR 없이 `git -C sift-docs commit && push`. 커밋 메시지 규약은 동일하다.

## 세션 시작 절차

1. `git status --short --branch` — 어느 브랜치인지, 원격과 어긋나지 않았는지 먼저 본다.
2. `gh issue view {N}` — 담당 이슈의 배경·DoD·TODO를 읽는다. **작업 목록의 원본은 이슈 본문의 `## TODO`다** (BACKLOG는 조율 세션이 쓴다 — 아래 참조).
3. 열린 자기 PR이 있으면 CodeRabbit 감시 Monitor가 걸려 있는지 `TaskList`로 확인하고, 없으면 다시 건다.
4. `.handoff/reviews/`에 자기 PR 리포트가 있으면 먼저 읽는다.

## BACKLOG를 직접 쓰지 않는다 — 그 대신

루프 문서 쓰기 주인은 조율 세션이다(D-034). 구현 중 세부 분해가 바뀌거나 기록으로 남길 것이 생기면:

- 세션 내부 작업 목록은 **Task 도구**로 관리한다 (문서가 아니다).
- 문서에 남아야 할 것은 `.handoff/notes/{이슈번호}-{요약}.md`에 적는다 — 조율 세션이 읽어 BACKLOG·STATE에 반영하고 파일을 지운다.
- 설계 결정이 필요한 분기를 만나면 추정하지 말고 **사용자에게 보고**한다. DECISIONS 기록은 조율 세션 몫이다.

## PR 생성 후 CodeRabbit 리뷰 자동 분석

`gh pr create` 직후 **반드시 아래 Monitor를 걸어둔다.** 코멘트가 달리면 task-notification으로 세션이 깨어난다 — 사용자가 알려주길 기다리지 않는다. 깨어나면:

1. `gh api`로 새 코멘트 조회 (요약: `issues/{N}/comments`, 인라인: `pulls/{N}/comments`)
2. 각 지적을 **현재 코드 기준으로 검증** — 수용/반박/이미 해결됨 판정
3. 결과를 사용자에게 보고. 반영 여부는 사용자 결정 → 후속 커밋 (스레드 답글 금지 — comment는 사람 영역, D-026)

Monitor 호출 — `persistent: true`, `description: "PR #{N} CodeRabbit 리뷰 감시"`, `N`·`REPO`만 치환:

```bash
N=15; REPO=siftnews/sift-api
last=$(date -u +%Y-%m-%dT%H:%M:%SZ)
while true; do
  state=$(gh pr view "$N" -R "$REPO" --json state --jq .state 2>/dev/null || echo OPEN)
  [ "$state" != "OPEN" ] && { echo "PR#$N $state — 감시 종료"; exit 0; }
  now=$(date -u +%Y-%m-%dT%H:%M:%SZ)
  gh api "repos/$REPO/issues/$N/comments?since=$last" --jq '.[] | select(.user.login=="coderabbitai[bot]" or .user.login=="coderabbitai") | "PR#'"$N"' 새 CodeRabbit 요약코멘트 id=\(.id)"' 2>/dev/null || true
  gh api "repos/$REPO/pulls/$N/comments?since=$last" --jq '.[] | select(.user.login=="coderabbitai[bot]") | "PR#'"$N"' 새 CodeRabbit 인라인 id=\(.id) \(.path):\(.line // 0)"' 2>/dev/null || true
  last=$now; sleep 60
done
```

⚠️ **CodeRabbit 체크가 `SUCCESS`여도 리뷰가 비어 있을 수 있다** — rate limit이 걸려도 체크는 통과로 뜬다(PR #22·#28·#38·#40에서 4회 재발). 리뷰 0건이면 **base 최신화 머지 커밋을 push하면 재트리거된다**(PR #40에서 확인 — 에이전트가 쓸 수 있는 유일한 재시도 수단이다).

## 인계 지점

- **← 🧭 조율**: 이슈 번호를 받아 착수한다.
- **→ 👀 검수**: PR을 만든 뒤 사용자에게 알린다. **자기 PR을 자기가 최종 리뷰하지 않는다.**
- **→ 🧭 조율**: 문서 반영거리는 `.handoff/notes/`로.

## 병렬화 — 기능 구현에는 쓰지 않는다

`/team`·서브에이전트는 **의존 없는 기계적 반복**에만 쓴다 (링크 일괄 치환, 여러 파일에 같은 패턴 적용).

**기능 구현에는 쓰지 않는다.** 한 이슈의 변경은 도메인→포트→서비스→어댑터→테스트로 이어지는 **의존 체인**이라(병합 PR 중앙값 13파일) 워커를 늘려도 대기만 늘고, TDD 순서 자체가 직렬이다. 게다가 워커들은 같은 `sift-api/` 워킹 트리를 공유한다 — **브랜치는 워커의 속성이 아니라 디렉터리의 속성이라** 서로의 작업이 한 브랜치에 섞이고 커밋 리듬("작업 1개 완료·검증 통과마다 커밋")이 깨진다.

서브에이전트는 이 세션의 deny를 그대로 상속하므로(HARNESS §0.8 규칙 6), 서브에이전트를 써도 루프 문서·이슈 발행 경계는 유지된다 — 안전하지만 우회 수단도 아니다.

## 하지 않는 것

- **루프 문서를 고치지 않는다** (권한으로 차단). 잠금 없는 단일 진실원천이라 다중 쓰기는 갱신 유실로 이어진다 (D-016).
- **이슈를 발행하지 않는다** (권한으로 차단). 무엇을 할지는 조율 세션과 사용자가 정한다.
- **자기 코드의 최종 리뷰어가 되지 않는다.** 구현 중 자체 점검은 하되, 승인 판단은 검수 세션과 사용자 몫이다.
- **사람 게이트를 넘지 않는다** — 병합·close·comment·release·인프라 쓰기는 👤 (D-026).

## 중단 조건 (멈추고 사용자에게)

- 같은 작업에 2회 연속 실패
- 이슈 범위를 벗어나는 변경이 필요해짐 → 범위를 늘리지 말고 보고 (이슈 분할은 조율 몫)
- 설계 분기·모호성 → 추정 금지
- DoD를 충족할 수 없는 사유 발견
