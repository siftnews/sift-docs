# Sift 백엔드 하네스

Sift의 활성 개발 기준이다. 이 문서는 작업 흐름을 정의하고, 세부 기술 사실은 `references/`, 검증 항목은 `checklists/`, 반복 절차는 `workflows/`의 원본을 따른다.

## 1. 작업 트리거

| 영역 | 트리거 예시 | 필수 참조 |
|---|---|---|
| API | REST API, Controller, Endpoint | api-design, security |
| 서비스 | Business Logic, Use Case | architecture, coding-conventions |
| 영속성 | JPA, QueryDSL, Persistence | database-design |
| 트랜잭션 | `@Transactional`, Isolation, Lock | architecture, database-design |
| 데이터베이스 | Schema, Migration, Index | database-design, rollback |
| 보안 | Spring Security, JWT, OAuth2 | security, security-checklist |
| 캐시·메시징 | Redis, Caffeine, Kafka, RabbitMQ, Event | architecture, rollback |
| 배치·스케줄러 | Spring Batch, `@Scheduled` | observability, performance-test |
| 운영 | Docker, Kubernetes, CI, Monitoring | deployment, deployment-checklist |
| 성능 | GC, JVM, Profiling, K6, JMeter | performance-test |
| PR·병합 | Pull Request, Merge | create-pr, pr-checklist |

## 2. 기본 파이프라인

```text
사전 확인 → 설계 → 구현 → 검증 → 릴리스
```

1. **사전 확인**: architecture, development, coding-conventions, team-configuration을 읽는다.
2. **설계**: 영향에 따라 API·DB·보안 설계를 확인하고 결정·완료 조건을 정한다.
3. **구현**: controller·service·repository·integration 변경을 작은 단위로 수행한다.
4. **검증**: 단위·통합·성능·보안 검증 중 필요한 항목을 실행한다.
5. **릴리스**: PR, 배포, 롤백, 릴리스 노트를 준비한다.

## 3. 검증 정책

**필수**: 빌드 성공, 전체 테스트 통과, 컴파일 오류 없음, 포맷 적용, 정적 분석 통과.

**선택**: 성능 벤치마크, 부하 테스트, 아키텍처 검토, 의존성 업그레이드 검토.

**조건부**: DB 마이그레이션, 캐시 무효화, API 버전 관리, 호환성 파괴 검토, 롤백 계획.

## 4. 사용자 결정 게이트

설계 분기, 파괴적 스키마 변경, 데이터 삭제·백필, 외부 발송·배포, 병합·공개 상태 변경은 근거와 선택지를 즉시 사용자에게 보고하고 결정을 받은 뒤 진행한다.
