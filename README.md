# Sift 문서

Sift는 외부 뉴스를 수집·선별해 구독자에게 발송하는 뉴스레터 배치 시스템이다. 구현 코드는 `sift-api`에, 설계·운영 원본은 이 저장소에 둔다.

## 하네스

활성 개발 기준은 [하네스](references/harness.md)다. 문서 골격은 안정적으로 유지하고, 아직 도입하지 않은 영역은 해당 문서에서 도입 조건과 필요한 결정만 안내한다.

| 목적 | 원본 |
|---|---|
| 작업 흐름·트리거·검증 정책 | [하네스](references/harness.md) · [작업 라우팅](references/workflow-routing.md) |
| 아키텍처·개발 규약 | [아키텍처 기준](references/architecture.md) · [개발 절차](references/development.md) · [코드 규약](references/coding-conventions.md) · [협업 구성](references/team-configuration.md) |
| 설계 기준 | [API](references/api-design.md) · [데이터베이스](references/database-design.md) · [보안](references/security.md) · [관측](references/observability.md) · [의존성](references/dependencies.md) |
| 실제 시스템 설계 | [MVP 상세 설계](references/mvp-design.md) · [선별 파이프라인](references/selection.md) · [이벤트 명세](references/events.md) |
| 반복 절차 | [workflows/](workflows/) |
| 검증 게이트 | [checklists/](checklists/) |
| 검토 관점 | [prompts/](prompts/) |
| 현재 상태·작업 | [STATE](STATE.md) · [TASKS](TASKS.md) · [BACKLOG](BACKLOG.md) |
| 결정 이력 | [ADR 인덱스](adr/README.md) |

## 구조

```text
sift-docs/
├── references/   아키텍처·개발·API·DB·보안·관측·의존성 및 시스템 설계
├── workflows/    이슈·PR·배포·롤백·릴리스·성능 검증 절차
├── checklists/   기능·PR·배포·보안·릴리스 검증 게이트
├── prompts/      코드 리뷰·리팩터링·성능·디버깅·아키텍처 검토 관점
├── adr/          결정 1건당 ADR 파일과 탐색 인덱스
└── STATE · TASKS · BACKLOG  현재 상태·작업 후보·진행 이슈
```

일반 작업은 비-Git 루트 `siftnews/`에서 시작한다. 협업·병렬화·사용자 결정 게이트는 루트 `AGENTS.md`와 [하네스](references/harness.md)를 따른다.
