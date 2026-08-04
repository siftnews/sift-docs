# sift-docs

**Sift** 워크스페이스의 공통 문서 레포 — 기획, 에이전트 하네스 설계, 루프 운영 기록.

> Sift는 외부 뉴스를 자동 수집·선별해 구독자에게 대량 발송하는 뉴스레터 배치 시스템이다.
> 코드는 [sift-api](https://github.com/siftnews/sift-api)(백엔드), sift-web(예정)에 있고,
> 이 레포에는 **레포를 가로지르는 문서**가 산다 — 이 레포의 커밋 히스토리 자체가 에이전트 루프를 실제로 운영한 기록이다.

## 문서 맵

| 문서 | 내용 |
|---|---|
| [references/HARNESS.md](references/HARNESS.md) | **먼저 읽을 것** — 에이전트 하네스 개발 방식: 원칙, 워크플로우 vs 에이전트 분류, 루프 프로토콜, 협업 흐름(이슈→PR→리뷰→병합), 역할 체계, 문서화 규칙 |
| [references/PLAN.md](references/PLAN.md) | 기획 전체 — 도메인(바운디드 컨텍스트), 발송 설계, 성능 로드맵 V1~V4 |
| [references/coding-conventions.md](references/coding-conventions.md) | 코드 규약 — 패키지 배치·네이밍·`create`/`restore`/`of`·`Clock` 주입·테스트 3층. **`origin/develop`에서 추출한 것만** 담는다 (Mockito 0건, Fake 직접 구현 등) |
| [references/observability.md](references/observability.md) | 관측 — `[measure]` 로그 규약, 계측을 배치 어댑터에 두는 이유, **지금 있는 것과 없는 것** (Prometheus 미도입) |
| [roles/](roles/) | 세션 역할 3종 — [🧭 조율](roles/ORCHESTRATOR.md) · [🔧 구현](roles/IMPLEMENTER.md) · [👀 검수](roles/REVIEWER.md). 경계는 권한 프로파일로 강제된다 (D-034) |
| [checklists/](checklists/) | [PR](checklists/pr.md) — REQUIRED / OPTIONAL / **CONDITIONAL**(조건에 걸리면 반드시) · [릴리스](checklists/release.md) develop → main 승격. 조건부 항목은 **실제로 데인 사례**에서 뽑았다 |
| [adr/DECISIONS.md](adr/DECISIONS.md) | 경량 ADR — D-001부터의 결정 로그 (결정·이유·비고) |
| [STATE.md](STATE.md) | 루프의 나침반 — 현재 Phase·진행 상황·다음 액션. "정정" 섹션은 에이전트 기록을 사람이 재검증·수정한 흔적 |
| [TASKS.md](TASKS.md) | 태스크 트리 — 마일스톤 M1~M4 → 이슈 후보 매핑 |
| [BACKLOG.md](BACKLOG.md) | 진행 중인 이슈 1개의 구현 단계 (작업 목록 원본은 이슈 본문 `## TODO` — D-035) |

```
sift-docs/
├─ references/   HARNESS · PLAN          어떻게 개발하는가 · 무엇을 만드는가
│                coding-conventions      어떤 규약으로 짜는가
│                observability           무엇을 어떻게 측정하는가
├─ roles/        조율 · 구현 · 검수       누가 무엇을 하는가 (D-034)
├─ checklists/   pr · release            언제 무엇을 확인하는가
├─ adr/          DECISIONS               왜 그렇게 정했는가
└─ STATE · TASKS · BACKLOG               지금 어디까지 왔는가 (루프 문서)
```

## 레포 종속 설계 문서는 각 코드 레포에

ERD·배치 설계([MVP-DESIGN](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md)), 선별 파이프라인([SELECTION](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)), 도메인 이벤트([EVENTS](https://github.com/siftnews/sift-api/blob/main/docs/EVENTS.md))는 `sift-api/docs/`가 원본이다 — 구조를 바꾸는 PR에서 코드와 같은 diff로 리뷰하기 위함 (D-021).
