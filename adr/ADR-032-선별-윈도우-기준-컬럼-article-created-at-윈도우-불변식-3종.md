# D-032 · 선별 윈도우 기준 컬럼 = `article.created_at` + 윈도우 불변식 3종 (2026-07-25 · D-031 구현 중 파생, 이슈 #21 → PR #22)

> [ADR 인덱스](README.md) · 결정 ID: D-032

- **결정**: ① 후보 로드 윈도우 `[from, to)`의 기준 컬럼은 **`article.created_at`(수집 시각)**. `published_at`은 쓰지 않는다. ② M2-5 배선이 반드시 지켜야 할 **윈도우 불변식 3종**을 `LoadCandidateArticlesPort` javadoc과 [MVP-DESIGN §4 ③](../references/mvp-design.md)에 명문화한다 — (1) `dedup_cluster_id`는 그 기사를 마지막으로 포함한 실행의 윈도우 기준 결과이므로 윈도우가 다른 값끼리는 비교·집계 불가, (2) scoreStep·selectStep의 대상 집합은 직전 `normalizeDedupStep` 윈도우의 **부분집합**이어야 한다, (3) 한 runDate의 모든 selectionJob 실행은 동일한 `[from, to)`를 공유해야 한다.
- **이유**: `published_at`은 nullable이라(발행시각 없는 피드 — 의도된 설계) 윈도우 조회에서 null인 기사가 통째로 누락되고, 피드 백필·소스 장애 복구로 뒤늦게 수집된 과거 기사가 영영 클러스터링되지 않는다. `created_at`은 수집 시점이 곧 처리 대상 편입 시점이라 누락이 없다. 불변식을 문서화하는 이유는 `normalizeDedupStep`이 **토픽 독립 전역 단계인데 selectionJob은 토픽마다 기동**되기 때문 — 불변식이 깨지면 예외 없이 조용히 틀린 결과가 난다(윈도우 밖으로 밀려난 기사가 옛 clusterId를 유지 → trendScore 과소 계산 + MMR 다양성 페널티 미적용으로 같은 사건 기사가 한 이슈에 중복 게재).
- **비고**: 대표 기사 선정은 별개로 `published_at` 기준을 유지한다(최신 보도가 대표). 윈도우 **크기**의 운영값은 트리거 주기와 함께 M2-5에서 확정. 불변식 (2)·(3)은 코드로 강제되지 않는 문서 계약이라, M2-5에서 Step 조립 시 검증 지점을 둘지 함께 판단할 것.
