# Sift — TASKS (MVP 태스크 트리 = 이슈 발행 원본)

> **마일스톤(M) = GitHub 마일스톤, 태스크 1개 = 이슈 1개 = PR 1개** (HARNESS §0.7).
> 🤖 에이전트가 초안·구현, 👤 사람이 발행·커밋·push·PR·병합 (D-013·D-014).
> 발행되면 제목 앞에 `#번호`를 태깅한다. 표기: `[ ]` 대기 · `[~]` 진행 · `[x]` 병합 완료
> 근거 문서: [PLAN.md](./PLAN.md) §5~7 · [MVP-DESIGN.md](https://github.com/siftnews/sift-api/blob/main/docs/MVP-DESIGN.md) · [SELECTION.md](https://github.com/siftnews/sift-api/blob/main/docs/SELECTION.md)

---

## M1 — 골든패스: 수집 (Phase 0)

- [x] **#1 `[chore] 프로젝트 골격 & 모듈 경계`** — PR #2로 병합 완료 (2026-07-06 · develop으로 병합됨 → main 동기화는 STATE 게이트)
> **M1 재편 (D-020, 2026-07-07)**: 작업 순서 = **DDD inside-out** (도메인 → 유스케이스 → 어댑터 → 통합).
> 각 이슈의 DoD = 컴파일이 아니라 **해당 레벨의 테스트 통과** (실행 가능한 증거).

- [x] **#4 `[FEAT] Source 도메인 모델`** (M1-2) — PR #5로 develop 병합 완료 (2026-07-08)
  - 범위: `Source`·`Article` 애그리거트 + URL 정규화 불변식(UTM 제거·호스트 소문자화), `RawArticle`, 도메인 예외(BusinessException 기반). **JPA·Spring 무의존** (D-009·D-018). 선행: 클래스 다이어그램 (D-017)
  - DoD: **도메인 단위 테스트 통과** (정규화 규칙 케이스 포함), 도메인 패키지에 기술 의존 없음
- [x] **#6 `[docs] 하네스 문서·설계 문서 이동`** — PR #7로 develop 병합 완료 (2026-07-10). 레포 종속 설계 문서 `sift-api/docs/` 이동(D-021) + 권한 게이트 + README
- [x] **#8 `[FEAT] CrawlSources 유스케이스`** (M1-3) — PR #9로 develop 병합 완료 (2026-07-11)
  - 범위: `CrawlSourcesUseCase`(port.in) + `LoadActiveSourcesPort`·`FetchFeedPort`·`SaveArticlePort`(port.out) 정의, `CrawlSourcesService` 구현. 리뷰 반영: `LoadActiveSourcesPort.findActiveById` 추가
  - DoD: **fake 포트 기반 서비스 단위 테스트 통과** ✅
- [~] **#10 `[FEAT] Source 영속 어댑터`** (M1-4) — PR #11 발행 (2026-07-15), 👤 리뷰·병합 대기
  - 범위: Source/Article JPA 엔티티·매핑·repository·adapter (BaseEntity 상속, UNIQUE normalized_url) · `Source.markCrawled()` 영속 반영 = 전용 포트 `UpdateSourcePort`로 결정(D-022)
  - DoD: **Testcontainers 통합 테스트 통과** ✅ (중복 저장 무시 검증 포함), `ApplicationModules.verify()` 유지 ✅
  - 부수 수정: `AbstractIntegrationTest`의 Postgres 컨테이너를 `@Container`(클래스별 stop/restart) → static 블록 싱글톤 패턴으로 변경 — 통합 테스트가 2개 이상일 때 컨테이너 재시작으로 포트가 바뀌며 Spring 컨텍스트 캐시가 옛 포트를 참조해 연결 실패하던 문제 수정
- [ ] **`[FEAT] RSS 수집 어댑터 (RssFeedAdapter)`** (M1-5)
  - 범위: `rome` 의존성, RSS 파싱 → `RawArticle` 변환, lang 추출
  - DoD: **실제 RSS 픽스처(xml) 파싱 테스트 통과**
- [ ] **`[FEAT] collectionJob 배치 + 측정 베이스라인`** (M1-6)
  - 범위: Spring Batch Job(인바운드 어댑터, chunk=50), Actuator 노출, 처리량·소요시간 계측 (원칙 5)
  - DoD: **`bootRun`으로 실제 RSS → `article` 적재 확인 (👤 실환경 게이트)**, 측정 로그 출력
- [ ] **`[chore] 골든패스 패턴 박제`** (M1-7) — 하네스 산출물 (Phase 0→1 전환점)
  - 범위: 골든패스에서 확립된 패턴 → `sift-api/CLAUDE.md` 규칙화 + "유스케이스 풀구현" 스킬(`implement-feature` 확장), D-009 잠정 결정 확정
  - DoD: 스킬로 두 번째 유스케이스 생산 가능 상태

## M2 — 선별 (Phase 1)

- [ ] **`[FEAT] Topic 도메인 + 시드 3종`** — topic 테이블, dev/ai/econ 시드 (MVP-DESIGN §1), 소스 RSS URL 확정 포함
- [ ] **`[FEAT] 선별 1/3: Normalize + Dedup`** — 정규화 검증·컷, 클러스터링(`dedup_cluster_id`) · ⚠️ 열린 질문: SimHash vs Jaccard → 이슈 안에서 결정·DECISIONS 기록
- [ ] **`[FEAT] 선별 2/3: Filter + Score`** — 토픽 필터, 가중합 스코어링, `article_score` + **breakdown JSON** (튜닝·eval 토대)
- [ ] **`[FEAT] 선별 3/3: Rank & Select`** — threshold·랭킹·다양성(MMR 여부 결정), `issue` + `issue_item` 생성
- [ ] **`[FEAT] selectionJob 배치 + selectionTrigger`** — Step 조립, 매일 발행 기준 시각 트리거(@Scheduled, 시각은 이 이슈에서 확정 — D-019)
  - M2 공통 DoD: build/test 통과 + 시드 토픽으로 이슈 1건 생성 확인

## M3 — 구독·발송 (Phase 1)

- [ ] **`[FEAT] Subscriber/Subscription 도메인 + 구독 API`** — 구독/해지 UseCase, REST 어댑터, **`preferred_send_hour` 포함** (이메일+토픽+수신 시각이 가입의 3요소 — D-019)
- [ ] **`[FEAT] 발송 스냅샷 (delivery_job/delivery_task)`** — DispatchIssueUseCase, `idempotency_key` 멱등(키 기준 issue vs job은 여기서 결정 — D-019 비고), **매시 pull 스캔 트리거** (당일 이슈 × preferred_send_hour)
- [ ] **`[FEAT] sendStep — 템플릿 렌더링 + SendEmailPort`** — LocalSmtpAdapter(mailpit), V1 단일 스레드 chunk=500 · ⚠️ 열린 질문: Thymeleaf vs Mustache → 이슈 안에서 결정
- [ ] **`[FEAT] retryJob — 에러 분류 + 지수 백오프`** — TRANSIENT/PERMANENT 분류, `next_retry_at`, DEAD 격리
  - M3 공통 DoD: E2E — 수집→선별→발송→**mailpit 수신 확인** (👤 게이트)

## M4 — 측정·부하: V1 베이스라인 (Phase 2 토대)

- [ ] **`[chore] 10만 구독자 시드 생성기`** — 부하용 데이터, 재실행 가능, **선호 시각 분포 파라미터화** (현실 분포=아침 피크 / 최악=전원 단일 시각 — D-019 부하 서사)
- [ ] **`[chore] 모니터링 스택`** — Prometheus + Grafana (compose), 배치 대시보드 · 위치(sift-infra 분리 여부) 이슈에서 결정
- [ ] **`[FEAT] V1 부하 측정 — 베이스라인 확보`** — 10만 건 발송 소요시간·TPS 측정 → `sift-docs/`에 기록 (포트폴리오 1챕터)

> **V2~V4는 여기서 미리 확정하지 않는다** — V1 측정 결과(병목 분석)가 다음 이슈 분해의 입력 (§0.5 에이전트 작업). 개선 사이클마다 `[FEAT] V{n} ...` 이슈를 그때 구성한다.
