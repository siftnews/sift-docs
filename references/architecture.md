# 아키텍처 기준

Sift 백엔드는 Spring Modulith 위의 헥사고날 구조를 따른다. 모듈은 바운디드 컨텍스트이고, 모듈 밖에서는 공개된 named interface 또는 도메인 이벤트만 사용한다.

## 변경 전 확인
- 모듈 소유권·의존 방향: [MVP-DESIGN](mvp-design.md)
- 이벤트 계약: [EVENTS](events.md)
- 실제 코드 규약: [coding-conventions](coding-conventions.md)

구조를 바꾸면 설계 문서와 다이어그램을 먼저 또는 같은 변경 단위에서 갱신한다. `ApplicationModules.verify()`는 모듈 경계의 최소 검증이다.
