# Sift — TASKS (MVP 태스크 트리 = 이슈 발행 원본)

> **마일스톤(M) = GitHub 마일스톤, 태스크 1개 = 이슈 1개 = PR 1개** (HARNESS §0.7).
> 🤖 에이전트가 이슈 발행·구현·커밋·push·PR 생성(사용자 명의·기존 스타일, §0.7 컨벤션 자율 준수 — D-027), 👤 사람이 리뷰·병합·배포 승격 (D-025·D-026).
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
- [x] **#10 `[FEAT] Source 영속 어댑터`** (M1-4) — PR #11로 develop 병합 완료 (2026-07-16)
  - 범위: Source/Article JPA 엔티티·매핑·repository·adapter (BaseEntity 상속, UNIQUE normalized_url) · `Source.markCrawled()` 영속 반영 = 전용 포트 `UpdateSourcePort`로 결정(D-022)
  - DoD: **Testcontainers 통합 테스트 통과** ✅ (중복 저장 무시 검증 포함), `ApplicationModules.verify()` 유지 ✅
  - 부수 수정: `AbstractIntegrationTest`의 Postgres 컨테이너를 `@Container`(클래스별 stop/restart) → static 블록 싱글톤 패턴으로 변경 — 통합 테스트가 2개 이상일 때 컨테이너 재시작으로 포트가 바뀌며 Spring 컨텍스트 캐시가 옛 포트를 참조해 연결 실패하던 문제 수정
- [x] **#12 `[FEAT] RSS 수집 어댑터 (RssFeedAdapter)`** (M1-5) — PR #13으로 develop 병합 완료 (2026-07-19)
  - 범위: `rome` 의존성, RSS 파싱 → `RawArticle` 변환, lang 추출
  - DoD: **실제 RSS 픽스처(xml) 파싱 테스트 통과** ✅ (valid/empty/missing-pubdate/malformed 4케이스)
- [x] **#14 `[FEAT] collectionJob 배치 + 측정 베이스라인`** (M1-6) — PR #15로 develop 병합 완료 (2026-07-21)
  - 범위: Spring Batch Job(인바운드 어댑터, chunk=50), Actuator 노출, 처리량·소요시간 계측 (원칙 5)
  - DoD: 측정 로그 출력 ✅ (`CollectionMetricsListener` — read/write count·소요시간) · **`bootRun`으로 실제 RSS → `article` 적재 확인 (👤 실환경 게이트) ⏳ 미확인** — 코드 경로는 병합됐으나 실환경 검증은 남음
  - 리뷰 반영: 소스별 오류 skip 격리(한 소스 실패가 Job 전체를 죽이지 않도록), step 실패 시 원인 예외 로깅. UseCase 우회 지적은 MVP-DESIGN §3① 이원화 근거로 반박 유지
  - 남은 결정: 배치 경로의 `markCrawled`(`UpdateSourcePort`)는 미반영 상태로 병합 — 기본 제외 확정 vs 후속 이슈 (M1-7에서 배치 ↔ `CrawlSourcesService` 역할 경계와 함께 정리)
## M2 — 선별 (Phase 1)

> **M1 잔여물 (M2 진행 중 정리 — D-029)**: ① D-009(도메인↔JPA 분리) → **분리 유지 확정 (2026-07-22)** ② markCrawled 배치 반영 결정 — 기본 제외 vs 후속 이슈 [👤 미결] ③ SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환 [sift-api 이슈→PR]
> **스킬 박제**(유스케이스 풀구현·`code-review`·`create-branch`)·`sift-api/CLAUDE.md` 규칙화는 M2 유스케이스에서 공통 패턴 추출 (D-029 — 1개 예시 조기 박제 회피).

- [x] **#17 `[FEAT] Topic 도메인 + 시드 3종`** — PR #18로 develop 병합 완료 (2026-07-23). Topic 도메인(POJO, D-009 분리) + topic 테이블·영속 + dev/ai/econ 시드 (MVP-DESIGN §1). **소스 RSS URL 확정·source 시드는 별도 이슈로 분리**
- [x] **#19 `[FEAT] 선별 1/3: Normalize + Dedup`** — PR #20으로 develop 병합 완료 (2026-07-23). 정규화 컷 + Jaccard 클러스터링(`dedup_cluster_id`, 임계 0.7), D-030 적용. content 로직·서비스(fake 포트)까지, 실 어댑터는 M2-5 배치
  - 후속 분리: **#21 재실행 멱등성** (CodeRabbit Major 수용)
- [x] **#21 `[FEAT] 선별 normalizeDedup 재실행 멱등성`** — PR #22로 develop 병합 완료 (2026-07-25). 윈도우 로드 + 실행 단위 상태 교체 + `clusterId = "c-" + min(memberIds)` + 벌크 갱신 (D-031). 범위는 fake 포트까지 — 실 어댑터 배선은 M2-5 유지
  - DoD: **fake 포트 서비스 단위 테스트 통과 ✅** (79건 전체 통과, CI pass) — 탈락 해제·신규 합류 시 clusterId 불변·동일 입력 동일 결과 3종 회귀
  - 파생 결정: 윈도우 기준 컬럼 = `article.created_at`, 윈도우 불변식 3종 명문화 (D-032)
- [~] **#23 `[FEAT] 소스 RSS URL 확정 + source 시드`** — MVP-DESIGN §1 소스 시드 실제 RSS URL 확정(9종: dev/ai/econ 각 3, 한국어 4·영어 5 — 2026-07-26 실 응답 검증) + `SourceSeeder`. **M1-6 e2e 게이트(실 RSS → article 적재)를 여기서 해소** (D-029 잔여물 분리). 브랜치 `feature/23-source-rss-seed`
- [ ] **`[FIX] 쿼리로 기사를 구분하는 소스의 URL 정규화`** — `UriNormalizer`가 쿼리스트링을 버려, `?idxno=213188`처럼 쿼리로 기사를 구분하는 사이트(AI타임스 등 한국 언론사 다수)의 기사가 **전부 같은 url로 정규화**된다 → `UNIQUE(normalized_url)`에 걸려 50개 중 1건만 적재(#23 e2e 실측). **기사 동일성 판정 규칙 변경**이라 dedup(D-030·D-031)·선별 전제에 영향 — 결정 필요
- [ ] **`[FEAT] collectionTrigger — 수집 배치 상시 기동`** — `spring.batch.job.enabled: false`인데 collectionJob 트리거가 없어 `bootRun`으로 Job이 기동되지 않는다(#23 착수 중 발견 — M1-6 e2e 게이트가 미해결로 남아 있던 실질적 원인). `@Scheduled` 주기 기동(MVP-DESIGN §3① "매시간" — 실제 주기는 이 이슈에서 확정) + 수동 기동 수단 검토
- [ ] **`[CHORE] Liquibase 마이그레이션 도입`** — 스키마가 `ddl-auto: update`에만 의존한다(운영 부적합, MVP-DESIGN §2에 "추후 전환 권장"으로만 적혀 있고 태스크로는 미등록이었음). 계기는 #23의 `ON CONFLICT (url)` — **유니크 제약이 없으면 예외인데 `update`는 기존 테이블에 제약을 소급 생성하지 않는다**(PR #24 CodeRabbit 지적 ⑥, e2e DB에서 실측 확인). 도입 시 두 시더(`SourceSeeder`·`TopicSeeder`)의 데이터 시드도 마이그레이션으로 흡수 검토
- [ ] **`[REFACTOR] TopicSeeder를 인바운드 어댑터로 이동`** — #23에서 `SourceSeeder`만 `adapter.in.bootstrap`으로 옮기고 `SeedSourcesUseCase`→`SaveSourcePort` 경유로 정리했다(PR #24 리뷰 반영). `content`의 `TopicSeeder`는 여전히 `adapter.out.persistence`에서 JPA 리포지토리를 직접 호출한다 — 같은 헥사고날 위반이고 저장도 conflict-ignore가 아니다. 모듈이 달라 #24 범위 밖으로 분리
- [ ] **`[FEAT] Atom 피드 파싱 검증`** — `rome`은 Atom(`<feed>`/`<entry>`)도 파싱하지만 `RssFeedAdapter` 테스트가 RSS 픽스처만 다뤄 미검증. 네이버 D2 등 Atom 소스 추가와 함께 픽스처 테스트 (#23에서 분리)
- [~] **#25 `[FEAT] 선별 2/3: Filter + Score`** — PR #26 리뷰 대기 (2026-07-27). 토픽 필터 + 4항목 가중합 + `article_score`(breakdown JSON, `(article_id, topic_id)` upsert). 브랜치 `feature/25-selection-filter-score` — **#24 위 스택이라 병합 후 base를 develop으로 재지정 필요 👤**
- [~] **#27 `[FEAT] 선별 3/3: Rank & Select`** — PR #28 리뷰 대기 (2026-07-27). threshold 컷·랭킹·**소스 쏠림 감점(전면 MMR 대신)**·`issue`+`issue_item` 생성. 브랜치 `feature/27-selection-rank-select` — **#26 위 스택이라 base 재지정 필요 👤**
- [~] **#29 `[FEAT] Source named interface + 선별 후보 조회·클러스터 갱신 실 배선`** — PR #30 리뷰 대기 (2026-07-29). `@NamedInterface("article-catalog")` 노출 + content `allowedDependencies` 확장 + 두 포트 실 어댑터 + **`article.dedup_cluster_id` 컬럼 신설**(ERD엔 있었으나 실제로 만들어진 적 없었음). **`[from, to)` 경계를 Testcontainers로 실검증** — PR #22 확인 항목 ② 해소. 브랜치 `feature/29-source-named-interface` (#28 위 스택)
- [~] **#31 `[FEAT] selectionJob 배치 + selectionTrigger`** — PR #32 리뷰 대기 (2026-07-29). Step 3종(tasklet) 조립 + 일일 트리거 + `SelectionMetricsListener`. **윈도우 24h·트리거 06:00(Asia/Seoul) 확정** — D-031·D-032가 "M2-5에서 정하라"고 남긴 항목. **M2 공통 DoD "시드 토픽으로 이슈 1건 생성" 실 DB로 충족 ✅**. 브랜치 `feature/31-selection-job` (#30 위 스택)
  - ※ 원래 M2-5 한 덩어리였으나 **Modulith 경계 개방(#29)과 배치 조립은 성격이 달라** 리뷰 단위를 둘로 나눔 (2026-07-29)
  - 파생 정정: `scoreStep (chunk)` → **tasklet** (윈도우 전체 재계산이 멱등성 전제라 아이템 단위로 못 쪼갬, D-031) · `@EnableScheduling` 부재 발견(없으면 배치가 영영 안 돎)

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
