# D-044 · Prometheus scrape endpoint와 인프라 레포 경계 (2026-08-08)

> [ADR 인덱스](README.md) · 관련 이슈 [#66](https://github.com/siftnews/sift-api/issues/66)

## 결정

1. `sift-api`는 `micrometer-registry-prometheus`를 runtime registry로 사용하고
   Spring Boot Actuator의 `/actuator/prometheus`를 애플리케이션 메트릭의 scrape endpoint로 제공한다.
2. 기존 `health`·`info`·`modulith` endpoint 노출과 `[measure]` 배치 로그는 유지한다.
3. 발송·재시도 Timer는 Prometheus histogram을 활성화해 V1에서 p95를 집계할 수 있게 한다.
4. Prometheus 서버, Grafana, scrape 설정, dashboard provisioning 파일은 애플리케이션 레포가 아닌
   향후 `sift-infra` 레포에 둔다. 현재 해당 레포가 없으므로 #66은 애플리케이션 측 계약과 지표 정의까지를 범위로 한다.

## 이유

- Spring Boot Actuator가 제공하는 표준 scrape endpoint를 사용하면 배치 어댑터가 이미 기록하는
  Micrometer 지표를 별도 HTTP 어댑터 없이 외부 시계열 저장소로 전달할 수 있다.
- Prometheus/Grafana 실행 수명주기는 애플리케이션 배포와 다르므로 `sift-api`의 Docker Compose에
  인프라 서비스를 섞지 않는다.
- Timer를 histogram으로 내보내야 Prometheus가 여러 상태·인스턴스의 p95를 집계할 수 있다.

## 영향

- Prometheus는 `sift-api:8080/actuator/prometheus`를 scrape target으로 사용한다.
- Grafana 패널은 [관측 문서의 V1 계약](../references/observability.md#5-m4-모니터링-경계와-대시보드-계약)을 기준으로 만든다.
- `/actuator/prometheus`의 인증·네트워크 접근 제한은 운영 보안 구성에서 별도로 결정한다.
