# Sift 백엔드 하네스

Sift의 활성 개발 기준이다. 문서 구조는 안정적으로 유지하며, 아직 도입하지 않은 기술 영역은 도입 이슈에서 해당 설계·검증 문서를 구체화한다.

## 작업 트리거

| 영역 | 트리거 예시 | 필수 참조 |
|---|---|---|
| API | REST API, Controller, Endpoint | [API 설계](api-design.md) · [보안](security.md) |
| 서비스 | Business Logic, Use Case | [아키텍처](architecture.md) · [코드 규약](coding-conventions.md) |
| 영속성 | JPA, QueryDSL, Persistence | [데이터베이스 설계](database-design.md) |
| 트랜잭션 | `@Transactional`, Isolation, Lock | [아키텍처](architecture.md) · [데이터베이스 설계](database-design.md) |
| 데이터베이스 | Schema, Migration, Index | [데이터베이스 설계](database-design.md) · [롤백](../workflows/rollback.md) |
| 보안 | Spring Security, JWT, OAuth2 | [보안](security.md) · [보안 체크리스트](../checklists/security-checklist.md) |
| 캐시·메시징 | Redis, Caffeine, Kafka, RabbitMQ, Event | [아키텍처](architecture.md) · [롤백](../workflows/rollback.md) |
| 배치·스케줄러 | Spring Batch, `@Scheduled` | [관측](observability.md) · [성능 테스트](../workflows/performance-test.md) |
| 운영 | Docker, Kubernetes, CI, Monitoring | [배포](../workflows/deployment.md) · [배포 체크리스트](../checklists/deployment-checklist.md) |
| 성능 | GC, JVM, Profiling, K6, JMeter | [성능 테스트](../workflows/performance-test.md) |
| 이슈 발행 | GitHub Issue, Assignees, Label | [이슈 생성](../workflows/create-issue.md) |
| PR·병합 | Pull Request, Merge | [PR 생성](../workflows/create-pr.md) · [PR 체크리스트](../checklists/pr.md) |

## 기본 흐름

1. **사전 확인**: 루트 `AGENTS.md`, [아키텍처](architecture.md), [개발 절차](development.md), [코드 규약](coding-conventions.md), [협업 구성](team-configuration.md)을 읽는다.
2. **설계**: 영향에 따라 API·DB·보안 설계를 확인하고 결정·완료 조건을 정한다.
3. **구현**: controller·service·repository·integration 변경을 작은 단위로 수행한다.
4. **검증**: 단위·통합·성능·보안 검증 중 필요한 항목을 실행한다.
5. **릴리스**: [PR](../workflows/create-pr.md), 배포, 롤백, 릴리스 노트를 준비한다.

## GitHub 이슈 발행 규약

이슈는 구현 착수 전에 **제목·본문·메타데이터를 한 단위로** 완료한다.

1. `.github/.github/ISSUE_TEMPLATE/`에서 `FEAT`·`FIX`·`REFACTOR`·`CHORE` 템플릿을 선택한다. 테스트 안정화 작업은 `CHORE`로 발행한다.
2. 제목은 `[TYPE] 한국어 요약` 형식으로 작성한다.
3. 본문은 `## Description`과 `## TODO`를 모두 채운다. Description에는 배경·범위·완료 조건을, TODO에는 검증 가능한 구현 단위를 적는다.
4. 생성 시 Assignees와 Label을 함께 지정한다. 담당자는 `chltjsdl0119`, 라벨은 `FEAT → feature`, `FIX → bug`, `REFACTOR/CHORE → chore`다.
5. 생성 직후 제목, 두 본문 섹션, Assignees, Label을 다시 확인한다. 하나라도 비어 있거나 불일치하면 구현을 시작하지 않는다.

권한 근거는 [D-026](../adr/ADR-026-git-github-쓰기-에이전트-위임-이슈-pr-생성-커밋-push-로컬-가드-훅.md)과 [D-028](../adr/ADR-028-gh-issue-pr-edit-에이전트-허용-assignee-라벨-관리-용도.md)에 둔다.

## 검증 정책

- **필수**: 빌드 성공, 전체 테스트 통과, 컴파일 오류 없음, 포맷 적용, 정적 분석 통과
- **선택**: 성능 벤치마크, 부하 테스트, 아키텍처 검토, 의존성 업그레이드 검토
- **조건부**: DB 마이그레이션, 캐시 무효화, API 버전 관리, 호환성 파괴 검토, 롤백 계획

## 사용자 결정 게이트

설계 분기, 파괴적 스키마 변경, 데이터 삭제·백필, 외부 발송·배포, 병합·공개 상태 변경은 근거와 선택지를 사용자에게 보고한 뒤 진행한다.
