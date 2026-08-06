# D-018 · Article 애그리거트 = Source 컨텍스트 소유 (2026-07-07)

> [ADR 인덱스](README.md) · 결정 ID: D-018

- **결정**: `Article`(과 article 테이블)은 **Source 컨텍스트가 소유**한다. Content는 후보 기사를 Source가 노출하는 **named interface(port.in) 경유로 조회**만 한다 (Modulith 규칙 준수). PLAN §3의 "기사 저장 = Content 책임" 기술은 오기로 보고 정정.
- **이유**: 기사는 수집·정규화의 산물 — 생산자인 Source가 저장·스키마(URL 정규화, UNIQUE(normalized_url) 멱등)를 책임지는 것이 응집도에 맞다. 두 컨텍스트가 한 테이블을 공유하는 모호함 제거 (바운디드 컨텍스트 매핑 중 발견·결정).
- **비고**: ~~⚠️ 열린 꼬리~~ **해소 (2026-07-22, D-030)**: dedup의 `article.dedup_cluster_id` 갱신은 **Source named interface에 갱신 오퍼레이션 추가**(옵션 ①)로 결정 — Content가 named interface로 호출, Source가 자기 컬럼 갱신을 통제. 별도 테이블 분리(옵션 ②)는 기각.
