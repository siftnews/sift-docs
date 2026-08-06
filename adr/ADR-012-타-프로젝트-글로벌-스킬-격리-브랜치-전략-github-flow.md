# D-012 · 타 프로젝트 글로벌 스킬 격리 + 브랜치 전략 = GitHub Flow (2026-07-04 · 스킬 유지 항목 개정 → D-023 · 브랜치 전략 개정 → D-025: develop 통합·main 배포)

> [ADR 인덱스](README.md) · 결정 ID: D-012

- **결정**: ① `~/.claude/skills/`의 타 프로젝트(MSA 4계층) 스킬 — `code-review`, `full-feature-cycle`, `create-branch`, `feign-client-*`, `kafka-event-handling`, `http-test` — 을 `~/.claude/skills-archive/`로 격리. Sift용은 골든패스 완료 후 `siftnews/.claude/skills/`에 자체 박제(Phase 1). ② 브랜치 전략 명칭을 **GitHub Flow**로 정정 — `main`에서 `feature/{이슈번호}-{kebab-case}` 분기 → PR → squash 병합. develop/release 레이어 없음.
- **이유**: 타 프로젝트 스킬이 자동 트리거되어 Sift의 Modulith·헥사고날 규칙과 배치되는 패턴(4계층·soft delete·FeignClient)을 주입할 위험. HARNESS §0.7의 실제 흐름은 Git Flow가 아니라 GitHub Flow였음.
- **비고**: `implement-feature`(Phase 1 재활용 예정)·`unit-test`(범용)는 유지. 아카이브 스킬은 원 프로젝트 `.claude/skills/`로 이전 권장.
