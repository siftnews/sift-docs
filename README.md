# sift-docs

> **현재 상태 (D-039, 2026-08-06)**: `sift-harness`는 삭제됐다. 아래 하네스·역할 문서는 과거 운영 기록과 향후 원격 재구축 참고용이다.

**Sift** 워크스페이스의 공통 문서 레포 — 기획, 하네스 설계 기록, 루프 운영 기록.

> Sift는 외부 뉴스를 자동 수집·선별해 구독자에게 대량 발송하는 뉴스레터 배치 시스템이다.
> 코드는 [sift-api](https://github.com/siftnews/sift-api)(백엔드), sift-web(예정)에 있고,
> 이 레포에는 **레포를 가로지르는 문서**가 산다 — 이 레포의 커밋 히스토리 자체가 에이전트 루프를 실제로 운영한 기록이다.

## 문서 맵

| 문서 | 내용 |
|---|---|
| [references/HARNESS.md](references/HARNESS.md) | 하네스 과거 운영 모델과 향후 재구축 참고: 원칙, 루프 프로토콜, 협업 흐름, 역할 체계 |
| [references/PLAN.md](references/PLAN.md) | 기획 전체 — 도메인(바운디드 컨텍스트), 발송 설계, 성능 로드맵 V1~V4 |
| [references/coding-conventions.md](references/coding-conventions.md) | 코드 규약 — 패키지 배치·네이밍·`create`/`restore`/`of`·`Clock` 주입·테스트 3층. **`origin/develop`에서 추출한 것만** 담는다 (Mockito 0건, Fake 직접 구현 등) |
| [references/team-configuration.md](references/team-configuration.md) | 오케스트레이터·읽기 전용 분석 서브 에이전트·구현 세션의 병렬화 경계 |
| [references/workflow-routing.md](references/workflow-routing.md) | Feature·Bug·Refactor·Performance·Review 작업별 라우팅과 검증 게이트 |
| [references/observability.md](references/observability.md) | 관측 — `[measure]` 로그 규약, 계측을 배치 어댑터에 두는 이유, **지금 있는 것과 없는 것** (Prometheus 미도입) |
| [references/MVP-DESIGN.md](references/MVP-DESIGN.md) | sift-api 상세 설계 — ERD·테이블 정의, 배치 Job/Step 명세, 포트 시그니처 |
| [references/SELECTION.md](references/SELECTION.md) | sift-api 선별 파이프라인 — Normalize·Dedup·Filter·Score·Rank & Select |
| [references/EVENTS.md](references/EVENTS.md) | sift-api 도메인 이벤트 — 모듈 간 통신 계약 |
| [roles/](roles/) | 과거 세션 역할 3종 — 현재 비활성, 원격 하네스 재구축 참고용 (D-039) |
| [checklists/](checklists/) | [오케스트레이터](checklists/orchestrator.md) · [PR](checklists/pr.md) — REQUIRED / OPTIONAL / **CONDITIONAL**(조건에 걸리면 반드시) · [릴리스](checklists/release.md) develop → main 승격. 조건부 항목은 **실제로 데인 사례**에서 뽑았다 |
| [adr/DECISIONS.md](adr/DECISIONS.md) | 경량 ADR — D-001부터의 결정 로그 (결정·이유·비고) |
| [STATE.md](STATE.md) | 루프의 나침반 — 현재 Phase·진행 상황·다음 액션. "정정" 섹션은 에이전트 기록을 사람이 재검증·수정한 흔적 |
| [TASKS.md](TASKS.md) | 태스크 트리 — 마일스톤 M1~M4 → 이슈 후보 매핑 |
| [BACKLOG.md](BACKLOG.md) | 진행 중인 이슈 1개의 구현 단계 (작업 목록 원본은 이슈 본문 `## TODO` — D-035) |

```
sift-docs/
├─ references/   HARNESS · PLAN          어떻게 개발하는가 · 무엇을 만드는가
│                coding-conventions      어떤 규약으로 짜는가
│                team-configuration      어떻게 분석을 병렬화하는가
│                workflow-routing        작업 유형별로 어떻게 흐르는가
│                observability           무엇을 어떻게 측정하는가
│                MVP-DESIGN · SELECTION · EVENTS   sift-api를 어떻게 설계했는가
├─ roles/        조율 · 구현 · 검수       누가 무엇을 하는가 (D-034)
├─ checklists/   orchestrator · pr · release 언제 무엇을 확인하는가
├─ adr/          DECISIONS               왜 그렇게 정했는가
└─ STATE · TASKS · BACKLOG               지금 어디까지 왔는가 (루프 문서)
```

## 문서는 전부 여기에, 코드는 각 레포에

설계 문서까지 이 레포로 모았다 (D-038 — D-021 개정). 심사자든 새 세션이든 **한 곳에서 시작**하면 되고, 코드 레포에는 코드만 남는다.

대가는 있다 — 설계 문서가 코드와 **같은 diff에서 리뷰되지 않는다.** 그래서 구조를 바꾸는 작업은 **두 레포에 커밋해야 하고**, 한쪽 push 누락이 곧 문서 drift다. [PR 체크리스트](checklists/pr.md)의 CONDITIONAL과 [구현 세션 절차](roles/IMPLEMENTER.md)가 그 지점을 막는다.

코드는 [sift-api](https://github.com/siftnews/sift-api)(백엔드) · sift-web(예정)에 있다.
