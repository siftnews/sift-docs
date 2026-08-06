# API 설계

현재 Sift MVP는 내부 배치 중심이며, 구독 관리부터 최소 REST API를 도입한다. 이후 REST API, Controller, Endpoint를 추가·변경할 때 이 문서를 계약의 원본으로 확장한다.

## Subscriber API (M3 · #55)
- `POST /api/v1/subscribers`: `{email, preferredSendHour}` → `201 Created`; `Location`과 생성 요약을 반환한다.
- `POST /api/v1/subscribers/{subscriberId}/subscriptions`: `{topicId}` → `201 Created`; 활성 토픽만 허용한다.
- `DELETE /api/v1/subscribers/{subscriberId}/subscriptions/{topicId}` → `204 No Content`; 해지는 `PAUSED`로 기록한다.
- 경계 검증 실패 `400`, 없는 구독자·토픽·구독 `404`, 중복 email·구독 `409`.
- MVP에는 인증·인가를 도입하지 않는다. 입력 외 개인정보를 응답·로그에 추가하지 않는다.

## 도입 시 필수 결정
- 리소스·HTTP 메서드·상태 코드·오류 형식
- 요청 검증과 인증·인가 정책
- 페이지네이션·정렬·필터 규칙
- 호환성, 버전 정책 및 폐기 일정
- 계약 테스트와 OpenAPI 등 계약 산출물

API 변경은 [security](security.md)와 [workflows/create-pr](../workflows/create-pr.md)의 조건부 검증을 함께 따른다.
