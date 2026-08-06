# D-023 · 문서 아키텍처 = SSoT 원본 지도 + 참조 강등 (2026-07-14)

> [ADR 인덱스](README.md) · 결정 ID: D-023

- **결정**: 모든 사실에 원본 1곳을 지정하고, 다른 문서는 요약 사본이 아니라 **링크 참조로 강등**한다. **원본 지도** — 결정 = DECISIONS · 확정 설계(스키마·포트·배치) = 각 레포 `docs/` · 진행 상태 = STATE(현재·다음 중심, 완료 이력은 최근 4~5건 + git 히스토리) · 작업 목록 = TASKS(이슈 후보) + BACKLOG(현 이슈 단계) · 작동 방식 = HARNESS(상태 표기 금지) · 세션 간 임시물 = `.omc/notepad`(스크래치패드 사용 금지) · graphify = 파생 인덱스(원본 아님, gitignore). 지침 충돌 시 우선순위: settings.json 게이트 > CLAUDE.md·HARNESS·DECISIONS > OMC 기본 지침.
- **이유**: 문서 아키텍처 리뷰(2026-07-14)에서 실드리프트 다수 확인 — `.coderabbit.yaml`의 D-018 미반영(리뷰어가 폐기된 소유권 규칙으로 동작), HARNESS §2 상태 체크박스 미갱신, BACKLOG M1-4 상태 불일치, D-017 본문의 폐기 규칙 잔존, STATE 비고의 죽은 기록(coderabbit path 결함은 98ec1d5에서 이미 해소). 공통 원인 = 동일 사실의 다중 수동 사본. 정합을 기억(규칙)이 아니라 구조(원본 지도)와 장치로 강제한다.
- **후속 (Medium — M1-5~7 사이클에 편입)**: ① `/harness-check` 정합 검사 스킬 박제(settings 사본 diff·루프 문서 상태 일치·크로스 레포 링크·최신 D-번호의 coderabbit/CLAUDE.md 영향) ② 구조 변경 이슈 DoD에 `.coderabbit.yaml`·CLAUDE.md 요약 영향 확인 추가(D-017 확장) ③ PLAN §5·SELECTION §3 중복 스키마의 MVP-DESIGN 링크 치환 ④ 이슈 본문 체크리스트 폐지 — BACKLOG 단일 장부(이슈 본문은 배경+DoD만) ⑤ graphify 최초 빌드(M1-7 직후, 이후 병합 시 `--update`).
- **후속 (Long-term — 트리거 도달 시에만)**: TASKS→GitHub 이슈 승격(열린 이슈 20+) · DECISIONS 파일 분할(결정 30+) · MVP-DESIGN 모듈별 분할(M3 착수) · 문서 링크 CI(첫 GitHub Actions와 함께) · 루트 로컬 전용 git(harness-check 불일치 2회 보고 시, D-015 비고 재검토) · 커밋 게이트 완화 검토(Phase 2 진입, D-010 비고) · graphify wiki/MCP(2번째 코드 레포 활성화).
- **비고**: D-012 완결 — `implement-feature`·`unit-test`도 `~/.claude/skills-archive/`로 격리(타 프로젝트 관용구 잔존: `p_` 접두사·soft delete·BDDMockito 강제·존재하지 않는 SPEC.md 참조. Sift 골든패스의 fake 포트 테스트 패턴과 충돌). M1-7에서 Sift 기반으로 재박제. 컨텍스트 맵은 `.drawio.svg`로 변환해 sift-docs 버전 관리에 편입(D-017 형식 준수). 부수 발견: PORTFOLIO.md 유실(STATE 정정 참조).
