# 작업 유형별 워크플로 라우팅

> **현재 비활성 (D-039, 2026-08-06)** — 아래 라우팅은 과거 역할 런처 기반 모델이다. 현재는 일반 세션에서 작업 유형에 맞는 검토·검증 절차만 참고한다.

## 공통 사전 단계

모든 작업은 다음 순서로 시작한다.

1. `STATE.md`와 현재 이슈·PR 상태를 확인한다.
2. 작업 유형과 영향 범위를 판정한다.
3. 해당 영향 범위의 필수 문서를 읽는다.
4. 넓은 작업이면 Graphify 최신성을 확인한다.
5. 필요한 분석 서브 에이전트를 병렬 실행한다.
6. 결과를 종합한 뒤 구현 범위와 DoD를 확정한다.

## Feature

```text
사전 단계 → 설계 확인 → 이슈 발행 → sift-impl 구현 → 검증 → PR → sift-review
```

필수 확인:

- `references/HARNESS.md`
- `references/coding-conventions.md`
- 관련 설계 문서
- 관련 역할 문서

## Bug

```text
재현 → 실패 테스트 → 수정 → 전체 검증 → PR → sift-review
```

버그의 원인과 회귀 테스트가 불명확하면 구현을 시작하지 않고 분석 역할을 추가한다.

## Refactor

```text
영향 분석 → 아키텍처 검토 → 작은 단위 리팩터링 → 검증 → PR
```

동작 변경이 포함되면 Feature 또는 Bug workflow로 승격한다.

## Performance

```text
베이스라인 → 측정·프로파일링 → 병목 가설 → 변경 → 벤치마크 → 검증 → PR
```

측정 결과가 없는 성능 개선은 완료로 간주하지 않는다.

## Review

```text
PR diff → 테스트 적절성 → 설계 이탈 → 문서 정합성 → 외부 리뷰 공백 → 리포트
```

검수 세션은 수정하지 않고 `.handoff/reviews/`에 결과를 남긴다.

## 검증 게이트

### 필수

- 빌드 성공
- 전체 테스트 통과
- 컴파일 오류 없음
- 포맷·정적 분석 통과

### 조건부

- DB migration
- 캐시 무효화
- API versioning
- breaking change 검토
- rollback 계획

### 선택

- 성능 benchmark
- load test
- architecture review
- dependency upgrade
