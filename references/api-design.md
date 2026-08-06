# API 설계

현재 Sift MVP는 내부 배치 중심이며 공개 REST API 계약은 없다. REST API, Controller, Endpoint를 새로 도입하거나 변경할 때 이 문서를 계약의 원본으로 확장한다.

## 도입 시 필수 결정
- 리소스·HTTP 메서드·상태 코드·오류 형식
- 요청 검증과 인증·인가 정책
- 페이지네이션·정렬·필터 규칙
- 호환성, 버전 정책 및 폐기 일정
- 계약 테스트와 OpenAPI 등 계약 산출물

API 변경은 [security](security.md)와 [workflows/create-pr](../workflows/create-pr.md)의 조건부 검증을 함께 따른다.
