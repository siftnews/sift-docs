# sift-docs

**Sift** 워크스페이스의 공통 문서 레포 — 기획, 에이전트 하네스 설계, 루프 운영 기록.

> Sift는 외부 뉴스를 자동 수집·선별해 구독자에게 대량 발송하는 뉴스레터 배치 시스템이다.
> 코드는 [sift-api](https://github.com/siftnews/sift-api)(백엔드), sift-web(예정)에 있고,
> 이 레포에는 **레포를 가로지르는 문서**가 산다 — 이 레포의 커밋 히스토리 자체가 에이전트 루프를 실제로 운영한 기록이다.

## 문서 맵

| 문서 | 내용 |
|---|---|
| [HARNESS.md](HARNESS.md) | **먼저 읽을 것** — 에이전트 하네스 개발 방식: 원칙, 워크플로우 vs 에이전트 분류, 루프 프로토콜, 협업 흐름(이슈→PR→리뷰→병합), 멀티세션 운영, 문서화 규칙 |
| [PLAN.md](PLAN.md) | 기획 전체 — 도메인(바운디드 컨텍스트), 발송 설계, 성능 로드맵 V1~V4 |
| [STATE.md](STATE.md) | 루프의 나침반 — 현재 Phase·진행 상황·다음 액션. "정정" 섹션은 에이전트 기록을 사람이 재검증·수정한 흔적 |
| [BACKLOG.md](BACKLOG.md) | 루프가 집는 작업 큐 (이슈 내 하위 단계) |
| [TASKS.md](TASKS.md) | 태스크 트리 — 마일스톤 M1~M4 → 이슈 후보 매핑 |
| [DECISIONS.md](DECISIONS.md) | 경량 ADR — D-001부터의 결정 로그 (결정·이유·비고) |

## 레포 종속 설계 문서는 각 코드 레포에

ERD·배치 설계([MVP-DESIGN](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md)), 선별 파이프라인([SELECTION](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)), 도메인 이벤트([EVENTS](https://github.com/siftnews/sift-api/blob/main/docs/EVENTS.md))는 `sift-api/docs/`가 원본이다 — 구조를 바꾸는 PR에서 코드와 같은 diff로 리뷰하기 위함 (D-021).
