# Sift — 현재 상태

> 현재 운영 상태와 다음 행동의 단일 진실원천이다. 과거 이력은 GitHub 이슈·PR, Git, [ADR](adr/README.md)에서 조회한다.
> 프로토콜: [HARNESS](references/harness.md) · 현재 운영 결정: [D-041](adr/ADR-041-루트-omx-오케스트레이터-sift-api-전용-graphify.md) · 마지막 갱신: 2026-08-06

## 현재 운영 상태

- 일반 작업 세션은 비-Git 루트 `siftnews/`에서 시작하며, Git·빌드·테스트는 대상 하위 저장소에서 실행한다.
- `sift-harness`와 역할 런처는 현재 사용하지 않는다. 협업·병렬화·사용자 결정 게이트는 루트 `AGENTS.md`와 HARNESS를 따른다.
- `sift-api` 코드 탐색용 Graphify 인덱스는 `sift-api/graphify-out/`만 사용한다.

## 현재 작업

- [#47 배치 통합 테스트의 시각 의존 제거](https://github.com/siftnews/sift-api/issues/47): 구현·검증·병합 완료(PR #48, 후속 보정 PR #49).
- [#50 Liquibase 마이그레이션 도입](https://github.com/siftnews/sift-api/issues/50): PR #51 병합 완료.
- [#52 Atom 피드 파싱 검증 및 네이버 D2 소스 추가](https://github.com/siftnews/sift-api/issues/52): PR #54 병합 완료.
- [#55 Subscriber/Subscription 도메인 및 구독 API](https://github.com/siftnews/sift-api/issues/55): PR #56 생성, CI·리뷰 대기.

## 다음 우선순위

1. #55 Subscriber/Subscription 도메인 및 구독 API 구현·검증
2. M3 발송 스냅샷·메일 발송·재시도 구현
