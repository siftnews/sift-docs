# D-025 · 브랜치 전략 확정 — develop 통합·main 배포 이원화 (2026-07-17 · D-012 브랜치 항목 개정)

> [ADR 인덱스](README.md) · 결정 ID: D-025

- **결정**: D-012의 "GitHub Flow(develop 없음)"를 개정 — **develop = 통합 브랜치, main = 배포 브랜치**로 공식화. feature는 develop에서 분기(`feature/{이슈번호}-{kebab}`), PR base = develop. develop→main 승격(배포)은 사람이 결정·실행. develop이 main보다 앞서 있는 것은 **정상 상태**(배포 전 통합분).
- **이유**: 사용자 결정(2026-07-17) — main을 배포용으로 쓰는 운영 감각이 실제 관행(PR #5·#7·#9·#11 모두 develop 병합)과 이미 일치. 문서(D-012)만 실무와 어긋나 있었으므로 문서를 실무에 맞춘다.
- **비고**: STATE의 "develop 처리 결정" 게이트 해소. develop 삭제 항목(BACKLOG A) 폐기. main 승격 주기는 마일스톤 단위로 추후 감 잡히면 기록.
