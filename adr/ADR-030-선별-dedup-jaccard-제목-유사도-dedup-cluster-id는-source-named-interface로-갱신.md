# D-030 · 선별 Dedup — Jaccard 제목 유사도 + dedup_cluster_id는 Source named interface로 갱신 (2026-07-22 · D-018 꼬리 해소)

> [ADR 인덱스](README.md) · 결정 ID: D-030

- **결정**: ① Dedup 2차(제목 유사도)는 **Jaccard 토큰 유사도**(임계값 기본 0.7, 튜닝 대상)로 구현 — SimHash는 대규모 전환이 필요할 때 성능 로드맵에서 재검토. ② `article.dedup_cluster_id` 갱신은 **Source가 port.in named interface에 노출하는 갱신 오퍼레이션**(예: `updateDedupCluster(articleId, clusterId)`)을 Content가 호출해 수행한다 — article 스키마의 주인은 Source(D-018)이므로 컬럼 갱신도 Source가 통제. 별도 클러스터 테이블 분리안은 기각(article.dedup_cluster_id 컬럼이 족적이 됨). 후보 기사 조회도 Source named interface 경유(D-018).
- **이유**: 사용자 결정(2026-07-22). Jaccard는 구현·튜닝이 단순해 MVP·측정 우선 원칙에 맞다. 클러스터 갱신을 Source가 통제하면 Modulith 경계(Content→Source는 named interface만)와 D-018 소유권이 함께 지켜진다.
- **비고**: M2-2(#선별 1/3)는 content 도메인·애플리케이션 로직(정규화 컷 + Jaccard 클러스터링 + 서비스, fake 포트 단위 테스트)까지. Source named interface 실제 어댑터·조회/갱신 배선은 selectionJob 배치(M2-5) 또는 후속에서 연결(YAGNI — 소비 배치가 생길 때). SELECTION §3의 D-018 꼬리 경고 해소.
