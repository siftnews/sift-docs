# D-028 · gh issue/pr edit 에이전트 허용 — assignee·라벨 관리 용도 (2026-07-19 · D-026 deny 목록 개정)

> [ADR 인덱스](README.md) · 결정 ID: D-028

- **결정**: `gh issue edit`·`gh pr edit`를 deny에서 allow로 이동(두 settings 사본). **용도는 assignee·라벨 관리로 한정**(`--add-assignee`·`--add-label` 등) — 제목·본문·마일스톤 등 그 외 수정은 여전히 사람 영역. 권한 규칙으로는 플래그 단위 구분이 불가하므로 용도 제한은 지침(HARNESS §0.7·메모리)으로 유지. close·comment·delete·merge 등 나머지 deny 게이트 불변.
- **이유**: 사용자 결정(2026-07-19) — create 시점에 assignee·라벨을 누락하면(이슈 #12·PR #13 사례) 소급 지정이 deny에 막혀 사람 몫이 되는 불편. 담당자·라벨은 가역적 저위험 메타데이터라 게이트 실익이 없음.
- **비고**: deny 완화는 에이전트가 스스로 편집 불가(권한 분류기 차단 — 자기 게이트 해제 금지 경계) → 사용자가 jq 원라이너로 직접 적용(2026-07-19). 적용 직후 이슈 #12 `feature` 라벨, PR #13 assignee(`chltjsdl0119`)·`feature` 라벨 소급 지정 완료.
