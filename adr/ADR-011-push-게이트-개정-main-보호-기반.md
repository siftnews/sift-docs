# D-011 · push 게이트 개정 — main 보호 기반 (2026-07-04 · 보류 → D-013: push deny 유지)

> [ADR 인덱스](README.md) · 결정 ID: D-011

- **결정**: 게이트 대상은 "push 자체"가 아니라 **main 병합/변경**. main 브랜치 보호 규칙(직접 push 금지·PR 필수) 설정 후에는 feature 브랜치 push를 에이전트에 허용(settings.json deny 해제 + `gh issue`/`gh pr` allow). 보호 규칙 설정 전까지는 현행 deny 유지.
- **이유**: §0.7 협업 흐름은 에이전트의 push+PR 생성을 전제하는데 D-007이 push 전체를 게이트로 규정해 충돌. feature push는 브랜치 보호가 있으면 되돌리기 어려운 행위가 아님. 로컬 커밋은 원래 게이트 아님(되돌리기 가능).
- **비고**: D-007 부분 개정. 선결: gh CLI 설치·인증 + main 보호 설정 (BACKLOG A).
