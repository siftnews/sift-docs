# 오케스트레이터·서브 에이전트 운영 규칙

> **현재 비활성 (D-039, 2026-08-06)** — `sift-harness`와 역할 런처가 삭제됐다. 원격 하네스 재구축 시 재검토할 과거 운영 모델이다.

## 목적

Sift는 하나의 조율 세션이 작업을 지휘하고, 탐색·설계·검증을 읽기 전용 서브 에이전트로 병렬화한다. 실제 코드 변경은 별도 구현 세션(`sift-impl`)이 담당한다.

```text
sift-orch
  ├─ explorer
  ├─ architect
  ├─ test-analyst
  ├─ security-reviewer
  ├─ performance-analyst
  └─ document-reviewer

sift-impl
  └─ 코드 수정·테스트·커밋·push·PR
```

## 기본 원칙

1. 오케스트레이터는 분석 결과를 종합하고 구현 범위·DoD를 확정한다.
2. 분석 서브 에이전트는 읽기 전용이다. 같은 작업 트리의 파일을 수정하지 않는다.
3. 구현은 이슈 하나당 구현 세션 하나가 담당한다.
4. 하나의 feature를 여러 서브 에이전트가 같은 worktree에서 동시에 구현하지 않는다.
5. 구현 세션은 오케스트레이터가 만든 이슈의 `## TODO`와 분석 결과를 원본으로 삼는다.
6. 권한 경계가 필요한 구현 서브 에이전트는 부모 세션의 권한을 상속하는 일반 서브 에이전트로 대체하지 않는다. 별도 구현 세션 또는 분리된 worktree를 사용한다.

## 표준 분석 역할

| 역할 | 책임 | 기본 산출물 |
|---|---|---|
| `explorer` | 관련 코드·문서·의존성 탐색 | 영향 파일·모듈 목록 |
| `architect` | 모듈 경계·의존 방향·설계 이탈 분석 | 설계 위험·변경 경계 |
| `test-analyst` | 테스트 전략·누락·회귀 위험 분석 | 필요한 테스트 목록 |
| `security-reviewer` | 인증·인가·입력 검증·민감정보 분석 | 보안 위험·검증 항목 |
| `performance-analyst` | 쿼리·배치·동시성·JVM 병목 분석 | 측정 계획·병목 가설 |
| `document-reviewer` | 코드와 `sift-docs` 정합성 분석 | 갱신 대상·drift 위험 |

서브 에이전트는 문제를 발견해도 직접 수정하지 않고, 파일·위치·근거·제안 형식으로 보고한다.

## 작업 유형별 병렬화

| 작업 유형 | 기본 분석 역할 |
|---|---|
| 일반 기능 | `explorer`, `architect`, `test-analyst` |
| API 변경 | `explorer`, `architect`, `security-reviewer`, `test-analyst` |
| DB·마이그레이션 변경 | `explorer`, `architect`, `test-analyst` |
| 보안 변경 | `explorer`, `security-reviewer`, `test-analyst` |
| 성능·배치 변경 | `explorer`, `performance-analyst`, `test-analyst` |
| PR 검수 | `architect`, `test-analyst`, `document-reviewer` |

모든 역할을 항상 실행하지 않는다. 작업의 영향 범위에 해당하는 역할만 선택한다.

## 구현 위임 계약

오케스트레이터는 이슈 또는 `.handoff/notes/`에 다음 내용을 남긴다.

```markdown
## Context
- 작업 배경
- 현재 문제

## Impact analysis
- 영향받는 모듈
- 영향받는 문서
- 예상 위험

## Implementation plan
1. ...
2. ...

## Validation
- 필수 테스트
- 빌드 명령
- 조건부 검증

## Decisions
- 확정된 결정
- 사용자 판단이 필요한 항목
```

## Graphify 사용 기준

다음 작업에서는 Graphify 그래프의 존재와 최신성을 먼저 확인한다.

- 영향 범위가 넓은 작업
- 모듈 경계를 넘는 작업
- 처음 분석하는 코드 영역
- 대규모 리팩터링
- 아키텍처 검토

단일 클래스 수정이나 작은 버그 수정에는 Graphify를 의무적으로 실행하지 않는다. 그래프가 stale이면 갱신한 뒤 `GRAPH_REPORT.md`를 먼저 읽고, 필요한 원본 파일만 서브 에이전트에 전달한다.
