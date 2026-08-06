# 작업 유형별 워크플로 라우팅

작업의 키워드가 아니라 실제 영향 범위로 필요한 문서·검증을 선택한다. 공통 시작점은 [development](development.md)와 [HARNESS](harness.md)다.

| 작업 | 설계·구현 참조 | 검증·릴리스 참조 |
|---|---|---|
| 기능/API | architecture, api-design, security | feature-checklist, security-checklist, create-pr |
| 서비스·도메인 | architecture, coding-conventions | feature-checklist, create-pr |
| 영속성·트랜잭션 | database-design, architecture | feature-checklist, rollback |
| 보안 | security, dependencies | security-checklist, integration test |
| 배치·스케줄러 | observability, architecture | performance-test, feature-checklist |
| 성능 | observability, dependencies | performance-test |
| 리팩터링 | coding-conventions, architecture | refactoring prompt, 전체 회귀 검증 |
| 장애·버그 | debugging prompt, 관련 설계 | 재현 테스트, 전체 회귀 검증 |
| 배포·릴리스 | deployment, rollback | deployment-checklist, release-note |

영향 범위가 넓거나 처음 보는 영역이면 읽기 전용 탐색·설계·테스트·보안 검토를 병렬화할 수 있다. 같은 워킹 트리에서 의존적인 기능 구현을 병렬화하지 않는다.
