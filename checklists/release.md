# 릴리스 체크리스트 (develop → main 승격)

> **main = 배포 브랜치, develop = 통합 브랜치** (D-025). 승격은 **마일스톤 단위로 사람이 결정**한다.
> develop이 main보다 앞서 있는 것은 **정상 상태**다 — 배포 전 통합분이다.
> 선례: [PR #16](https://github.com/siftnews/sift-api/pull/16) `[RELEASE] 수집 파이프라인 MVP(M1) develop → main`

## 승격 전

- [ ] 마일스톤의 이슈가 **전부 병합**됐다 (`gh issue list --milestone {M} --state open` 이 비어야 한다)
- [ ] 그 마일스톤의 **공통 DoD**를 충족했다 — 코드 경로가 아니라 **실제로 돌아간 증거**로 (예: M2는 "fake 없이 실 DB로 이슈 1건 + 항목 2건 생성")
- [ ] `origin/develop`에서 전체 테스트 통과 · CI green
- [ ] 열린 PR이 없다 (있으면 이번 승격에 포함할지 먼저 판단)
- [ ] 설계 문서(`sift-api/docs/`)가 코드와 정합하다 — 구조 변경이 있었다면 특히
- [ ] STATE의 「최근 완료」·TASKS 체크가 실제 병합 상태와 일치한다

## 승격 PR

- [ ] 제목 `[RELEASE] {요약} develop → main` (PR #16 선례)
- [ ] base = `main`, head = `develop`
- [ ] 본문에 **이 릴리스에 담긴 이슈 목록**과 마일스톤 DoD 충족 근거
- [ ] 👤 병합 (merge commit — 현 관행)

## 승격 후

- [ ] STATE의 현재 Phase 갱신
- [ ] 다음 마일스톤 태스크를 TASKS에서 확인 → 🧭 조율 세션이 다음 이슈 발행

## ⚠️ 지금은 배포 인프라가 없다

main 승격은 **"이 시점이 배포 가능한 상태"라는 표식**이지 실제 배포가 아니다. 되돌리기는 revert PR이 유일한 수단이다.
배포 대상이 생기면 이 문서에 배포·롤백 절차를 추가한다 — **그때 추가한다**(원칙 1: 골든패스 먼저, 추상화 나중).
