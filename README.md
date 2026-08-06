# sift-docs

Sift 워크스페이스의 공통 문서 레포입니다. Sift는 외부 뉴스를 수집·선별해 구독자에게 발송하는 뉴스레터 배치 시스템이며, 코드는 `sift-api`에 있습니다.

## 백엔드 하네스

활성 개발 기준은 [references/HARNESS.md](references/harness.md)입니다. 작업은 **사전 확인 → 설계 → 구현 → 검증 → 릴리스** 순서로 진행하며, API·서비스·영속성·보안·배치·운영 작업은 영향 영역에 맞는 문서와 체크리스트를 함께 따릅니다.

| 영역 | 원본 |
|---|---|
| 개발 흐름·트리거·검증 정책 | [HARNESS](references/harness.md) · [workflow routing](references/workflow-routing.md) |
| 아키텍처·구현 규약 | [architecture](references/architecture.md) · [development](references/development.md) · [coding conventions](references/coding-conventions.md) |
| API·DB·보안·운영 | [api design](references/api-design.md) · [database design](references/database-design.md) · [security](references/security.md) · [observability](references/observability.md) |
| 실제 시스템 설계 | [MVP design](references/mvp-design.md) · [selection](references/selection.md) · [events](references/events.md) |
| 반복 절차 | [workflows/](workflows/) |
| 검증 게이트 | [checklists/](checklists/) |
| 검토 관점 | [prompts/](prompts/) |
| 결정 이력 | [ADR](adr/decisions.md) |

## 구조

```text
sift-docs/
├── references/   아키텍처·개발·API·DB·보안·관측·의존성 및 시스템 설계
├── workflows/    PR·성능 검증·배포·롤백·릴리스 노트 절차
├── checklists/   기능·PR·보안·배포·릴리스 검증 게이트
├── prompts/      코드 리뷰·리팩터링·성능·디버깅·아키텍처 검토 관점
├── adr/          결정 로그
└── STATE · TASKS · BACKLOG  현재 작업 상태와 계획
```

과거의 역할 런처와 로컬 하네스는 복원하지 않습니다. 협업·병렬화·사용자 결정 게이트는 [team-configuration](references/team-configuration.md)과 [HARNESS](references/harness.md)를 기준으로 합니다.
