# D-019 · 발송 모델 = 매일 고정 + 구독자 선호 시각, dispatch는 매시 pull 스캔 (2026-07-07)

> [ADR 인덱스](README.md) · 결정 ID: D-019

- **결정**: ① 발송 주기는 **전 토픽 DAILY 고정** (topic.cadence·send_at_hour 제거, econ WEEKLY 시드도 DAILY로). ② 발송 시각은 **구독자가 선택** — `subscriber.preferred_send_hour`(시 단위). ③ 선별은 매일 발행 기준 시각(예: 06:00, 구현 이슈에서 확정)에 토픽당 1회 실행해 당일 이슈 생성 (D-003 유지). ④ 발송 기동은 push(IssueScheduled 이벤트)가 아니라 **dispatchJob 매시 정각 pull 스캔**: 당일 SCHEDULED 이슈 × `preferred_send_hour = 현재 시각`인 ACTIVE 구독자 → 스냅샷·발송. **IssueScheduled 이벤트는 MVP 제외.**
- **이유**: 제품 의도 확정 — "구독자가 이메일 + 토픽 + 수신 시각을 정하고, 매일 그 시각에 받는다." 발송 시점의 주인이 구독자이므로 이슈 확정 시점 ≠ 발송 시점 → 시간대 스캔(pull)이 자연스럽고, 확정 즉시 발송하는 push는 성립하지 않음.
- **비고**: 부하 서사는 시드로 유지 — 선호 시각을 현실 분포(아침 피크 집중) 또는 최악(전원 단일 시각)으로 시드해 버스트 측정. 신선도 트레이드오프(늦은 시각 구독자는 오래된 이슈 수신)는 MVP 수용, 개선 후보로 기록. preferred_send_hour는 subscriber 레벨(구독별 아님 — MVP 단순화, 개인 다이제스트 단계에서 재검토). 멱등 키를 `hash(delivery_job_id, subscriber_id)` → `hash(issue_id, subscriber_id)`로 바꿀지 구현 이슈에서 검토(발송 후 시각 변경 시 이중 발송 방지).
