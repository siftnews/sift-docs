# 성능 테스트 워크플로우

성능 작업은 측정 전후 비교가 있어야 완료다.

1. 목표 지표와 현실적인 부하 시나리오를 정한다.
2. 기준선(처리량, 지연, 오류율, CPU/메모리/GC, DB 지표)을 수집한다.
3. 병목 가설을 하나씩 검증하고 변경한다.
4. 동일 조건에서 재측정해 회귀와 개선을 비교한다.
5. 결과, 한계, 재현 방법을 PR 또는 설계 문서에 남긴다.

M4 subscriber seed는 `local,loadtest` 프로파일로만 실행한다. 기본값은 100,000건이며
`sift.load-test.subscribers.count`와 `sift.load-test.subscribers.email-domain`으로 재현 조건을
바꿀 수 있다. 생성기는 subscription fan-out이나 외부 SMTP 발송을 수행하지 않으므로, 실제 V1
측정에서는 별도 workload 준비 단계와 TestSendEmailAdapter 또는 명시한 sink를 사용한다.
