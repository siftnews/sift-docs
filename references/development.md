# 개발 절차

모든 백엔드 작업은 다음 순서를 따른다.

1. 작업 유형과 영향 범위를 판정한다.
2. [architecture](architecture.md), 이 문서, [coding-conventions](coding-conventions.md), [team-configuration](team-configuration.md)을 읽는다.
3. API·스키마·보안 등 영향 영역의 설계를 확인하고 구현 범위와 완료 조건을 정한다.
4. 작은 단위로 구현하고 대상 테스트를 먼저 실행한다.
5. 빌드·전체 테스트·포맷·정적 분석을 확인한다.
6. 조건부 검증과 문서 갱신을 확인한 뒤 PR 또는 릴리스 절차로 넘긴다.

설계 분기, 데이터 손실 가능성, 외부 발송·배포는 구현 전에 사용자 결정이 필요하다.
