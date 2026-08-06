# D-024 · gh CLI 도입 — 에이전트는 읽기 전용, 쓰기 게이트 실효화 (2026-07-15 · 쓰기 위임으로 개정 → D-026)

> [ADR 인덱스](README.md) · 결정 ID: D-024

- **결정**: gh CLI 설치·인증 완료(사람, 2026-07-15). 에이전트에 **gh 읽기 명령**(issue/pr `list`·`view`, pr `diff`·`checks`, `repo view`, run `list`·`view`, `search`) allow. 쓰기는 D-013·D-014 그대로 사람 전담 — 기존 deny(issue/pr create, pr merge)에 issue/pr `comment`·`edit`·`close`, issue `delete`, pr `review`·`ready`, `release`, repo `create`/`edit`/`delete`, `workflow run`, `secret`, `variable set`을 추가해 게이트를 실효화. `gh api`는 읽기·쓰기 겸용이라 allow/deny 모두 제외(프롬프트 백스톱).
- **이유**: gh 부재 시 deny 게이트는 장식이었으나 설치로 실제 게이트가 됨. 에이전트가 이슈·PR·리뷰 코멘트를 직접 조회하면 초안 작성·리뷰 반영의 컨텍스트 수집이 빨라짐. 쓰기 주도권(D-013)은 불변.
- **비고**: D-011(feature push 허용)은 여전히 보류(D-013). settings 변경은 루트·sift-api 두 사본 동일 적용(D-023 사본 정합 대상). 부수 확인(gh api 읽기로 검증): **sift-api main 보호 규칙은 이미 설정됨** — PR 필수·strict status checks·force push/삭제 금지. 단 `enforce_admins=false`(관리자 본인은 직접 push 가능)·필수 승인 리뷰어 0명은 1인 운영으로 수용. sift-docs main은 미보호 — 사람이 직접 커밋·push하는 레포라 수용(원하면 추후 설정).
