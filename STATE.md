# Sift — 현재 상태

> 현재 운영 상태와 다음 행동의 단일 진실원천이다. 과거 이력은 GitHub 이슈·PR, Git, [ADR](adr/README.md)에서 조회한다.
> 프로토콜: [HARNESS](references/harness.md) · 현재 운영 결정: [D-044](adr/ADR-044-prometheus-scrape-endpoint와-인프라-레포-경계.md) · 마지막 갱신: 2026-08-08

## 현재 운영 상태

- 일반 작업 세션은 비-Git 루트 `siftnews/`에서 시작하며, Git·빌드·테스트는 대상 하위 저장소에서 실행한다.
- `sift-harness`와 과거 역할 런처는 현재 사용하지 않는다. “다음 태스크를 탐색하고 진행하라” 사이클은 루트 `$task-cycle`와 `workflows/task-cycle.md`를 따르고, 협업·병렬화·사용자 결정 게이트는 루트 `AGENTS.md`와 HARNESS를 따른다.
- `sift-api` 코드 탐색용 Graphify 인덱스는 `sift-api/graphify-out/`만 사용한다.

## 현재 작업

- [#58 발송 스냅샷과 매시 pull 스캔 구현](https://github.com/siftnews/sift-api/issues/58): 구현·검증·병합 완료(PR #59).
- [#47 배치 통합 테스트의 시각 의존 제거](https://github.com/siftnews/sift-api/issues/47): 구현·검증·병합 완료(PR #48, 후속 보정 PR #49).
- [#50 Liquibase 마이그레이션 도입](https://github.com/siftnews/sift-api/issues/50): PR #51 병합 완료.
- [#52 Atom 피드 파싱 검증 및 네이버 D2 소스 추가](https://github.com/siftnews/sift-api/issues/52): PR #54 병합 완료.
- [#55 Subscriber/Subscription 도메인 및 구독 API](https://github.com/siftnews/sift-api/issues/55): 구현·검증·병합 완료(PR #56, 후속 보정 PR #57).
- [#60 sendStep 고정 HTML 렌더링 및 로컬 SMTP 발송](https://github.com/siftnews/sift-api/issues/60): PR #61 병합 완료. `SENDING` lease 만료 복구는 별도 설계 범위로 보류한다.
- [#62 retryJob 오류 분류 및 지수 백오프](https://github.com/siftnews/sift-api/issues/62): PR #63 병합 완료.
- [#64 10만 구독자 시드 생성기](https://github.com/siftnews/sift-api/issues/64): `feature/62-retry-job` 위에 `chore/64-100k-subscriber-seed`로 구현해 stacked PR [#65](https://github.com/siftnews/sift-api/pull/65)를 열었다. `loadtest` 프로파일 전용 결정적 시드, 아침 피크 `preferred_send_hour` 분포, 이메일 기준 bulk 멱등 저장을 포함한다.
- [#66 모니터링 스택](https://github.com/siftnews/sift-api/issues/66): `chore/66-monitoring-stack`에서 Prometheus registry·`/actuator/prometheus`·발송 Timer histogram과 회귀 테스트를 구현해 [PR #67](https://github.com/siftnews/sift-api/pull/67)을 열었다. Prometheus/Grafana 실행 환경은 D-044에 따라 향후 `sift-infra`에서 관리한다.

## 다음 우선순위

1. PR #65·#67 리뷰 및 병합 — 병합은 사용자 게이트
2. M4 V1 10만 건 발송 기준선 측정
3. `sift-infra` 생성 시 Prometheus/Grafana scrape·dashboard provisioning 구현
