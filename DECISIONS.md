# Sift — 결정 로그 (DECISIONS)

> 경량 ADR. 루프가 일관성을 유지하고 과거 결정을 되돌아보기 위한 기록. **새 결정은 맨 위에 추가.**
> 개정·폐기된 결정은 **본문을 다시 쓰지 않고 제목에 `(… 개정 → D-xxx)`를 표기**한다 (D-023).

## D-029 · M1-7 골든패스 박제 태스크 해체 — 스킬 박제는 M2로 연기 (2026-07-22 · D-012 스킬 박제 시점 개정 · D-020 M1-7 항목 개정)
- **결정**: TASKS M1-7 `[chore] 골든패스 패턴 박제`를 **해체**한다. 스킬 박제(유스케이스 풀구현·`code-review`·`create-branch`)와 `sift-api/CLAUDE.md` 규칙화는 **M2(2번째 유스케이스 selectionJob)를 만들며/만든 뒤 공통 패턴 추출**로 연기. M1-7에 묶여 있던 잔여물 3건은 M2 착수 전 정리 또는 M2 이슈에 편입: ① **D-009**(도메인↔JPA 분리) 잠정 결정 확정 — 골든패스 코드 검토 후(👤 설계 분기) ② **markCrawled 배치 반영** 결정 — 기본 제외 확정 vs 후속 이슈(👤) ③ **SELECTION.md §3 중복 스키마 → MVP-DESIGN 링크 치환**(sift-api 이슈→PR, D-023 후속 ③). **Phase 0→1 전환은 M2로 이뤄진다.**
- **이유**: 하네스 원칙 2("2번 했으면 박제")·원칙 1("박제할 패턴이 없는 상태에서 스킬부터 만들면 헛돈다")에 비춰, 유스케이스 1개(collectionJob)만으로 스킬을 굳히면 M2에서 재작업 가능성이 크다. 무엇이 공통이고 무엇이 매번 달라지는지는 2번째 유스케이스에서 드러난다. 사용자 결정(2026-07-22) — 골든패스 코드 경로는 M1-6에서 완주했으므로, 박제는 selectionJob이라는 2번째 반복이 생길 때로 미룬다.
- **비고**: D-012의 "골든패스 완료 후 Sift용 스킬 자체 박제(Phase 1)" 계획은 시점만 M2로 구체화(개정). 잔여물 중 D-009·markCrawled는 설계 분기라 결정 시 사용자 게이트, SELECTION 치환은 sift-api 레포라 이슈→PR. 삭제된 옛 MSA 스킬(2026-07-22 정정)과 이름 충돌은 이미 해소돼 M2 박제 시 네이밍 자유.

## D-028 · gh issue/pr edit 에이전트 허용 — assignee·라벨 관리 용도 (2026-07-19 · D-026 deny 목록 개정)
- **결정**: `gh issue edit`·`gh pr edit`를 deny에서 allow로 이동(두 settings 사본). **용도는 assignee·라벨 관리로 한정**(`--add-assignee`·`--add-label` 등) — 제목·본문·마일스톤 등 그 외 수정은 여전히 사람 영역. 권한 규칙으로는 플래그 단위 구분이 불가하므로 용도 제한은 지침(HARNESS §0.7·메모리)으로 유지. close·comment·delete·merge 등 나머지 deny 게이트 불변.
- **이유**: 사용자 결정(2026-07-19) — create 시점에 assignee·라벨을 누락하면(이슈 #12·PR #13 사례) 소급 지정이 deny에 막혀 사람 몫이 되는 불편. 담당자·라벨은 가역적 저위험 메타데이터라 게이트 실익이 없음.
- **비고**: deny 완화는 에이전트가 스스로 편집 불가(권한 분류기 차단 — 자기 게이트 해제 금지 경계) → 사용자가 jq 원라이너로 직접 적용(2026-07-19). 적용 직후 이슈 #12 `feature` 라벨, PR #13 assignee(`chltjsdl0119`)·`feature` 라벨 소급 지정 완료.

## D-027 · 가드 훅 폐지 — main 방어는 브랜치 보호로 일원화, 형식은 컨벤션 준수 (2026-07-19 · D-026 가드 훅 항목 개정)
- **결정**: D-026의 로컬 가드 훅(`.claude/hooks/git-gh-guard.sh`)을 삭제하고 두 settings(루트·sift-api)의 PreToolUse 등록을 해제한다. **쓰기 위임과 deny 게이트는 D-026 그대로 유지.** 형식(커밋 `{type}: {한국어 요약}` 한 줄·트레일러 금지·이슈/PR 템플릿·push 대상 명시)은 CLAUDE.md·HARNESS §0.7·메모리 지침으로 준수하고, main push 방어는 **GitHub 브랜치 보호 규칙으로 일원화** — 규칙 정비는 사용자가 직접 (sift-docs main 포함, 진행 중).
- **이유**: 사용자 결정(2026-07-19) — 훅 검토에서 우회 경로(`git push origin HEAD`·`+` refspec force·`--amend` 형식 미검사)와 fail-open 한계(jq 부재·래핑 명령 미매치) 확인. 클라이언트측 부분 장치를 유지·패치하기보다, 되돌리기 어려운 지점(main)은 서버측 브랜치 보호로 옮기고 형식은 지침·리뷰가 담당.
- **비고**: D-023 "기억이 아니라 장치로 강제"의 부분 후퇴 — 형식 이탈은 amend·후속 커밋으로 교정 가능한 가역 영역이라 수용, 이탈 사례가 반복되면 재도입 재검토. settings 사본 정합 대상에서 훅 스크립트 제외.

## D-026 · git/GitHub 쓰기 에이전트 위임 — 이슈·PR 생성·커밋·push + 로컬 가드 훅 (2026-07-17 · 가드 훅 항목 개정 → D-027)
- **결정**: D-013·D-014의 "쓰기 = 사람 전담"을 개정 — 에이전트가 **이슈 생성(`gh issue create`)·PR 생성(`gh pr create`)·`git commit`·`git push`**까지 실행한다(settings allow 승격 + deny 해제, `git add`·`git switch` 포함). **유지되는 게이트(deny)**: `gh pr merge`·`review`·`ready`, issue/pr `edit`·`close`·`comment`·`delete`, `release`, `repo *`, `workflow run`, `secret`, `variable set`, 파괴 명령(`git reset --hard`·`clean`, `compose down -v`). `gh api`는 D-024대로 allow/deny 양쪽 제외(프롬프트 백스톱). **스타일 강제**: 모든 기록은 사용자 명의로, 기존 관행 그대로 — 이슈 제목 `[FEAT|CHORE|FIX|REFACTOR|docs]` 접두어 + Description/TODO 템플릿, PR 본문 `close #N` + 체크리스트, 커밋 `{type}: {한국어 요약}` 한 줄·**트레일러 금지(Co-Authored-By 등 에이전트 서명 금지)**. **로컬 가드 훅 도입**(`.claude/hooks/git-gh-guard.sh`, PreToolUse/Bash): ① main 직접 push 차단 + push는 `origin {브랜치}` 명시 강제 ② 커밋 메시지 형식·트레일러 검사 ③ 이슈/PR 제목 접두어·PR 본문 `close #N` 검사.
- **이유**: 사용자 결정(2026-07-17) — 루프 자율성을 높이되 기록의 명의·스타일은 본인 것으로 유지. 병합·공개 상태 변경·인프라 쓰기만 사람 게이트로 남기면 되돌리기 어려운 지점은 보호된다(main은 브랜치 보호 + 훅 이중 방어). 형식은 규칙(기억)이 아니라 훅(장치)으로 강제 — D-023 철학.
- **비고**: 테스트 게이트 훅(push 전 `./gradlew test` 강제)은 **채택 안 함** — Testcontainers 포함 수 분짜리 실행이라 훅 타임아웃·루프 리듬과 충돌. 자가검증(§0.6)과 CI·브랜치 보호가 담당. settings 사본 정합 대상에 훅 스크립트 추가(루트·sift-api 동일 유지). ⚠️ 잔여 구멍: `~/.claude/settings.local.json`의 `Bash(gh api *)` allow가 백스톱을 무력화 — 전역 보호 훅이 이 파일 편집을 차단하므로 **사용자가 직접 해당 줄 제거 필요**.

## D-025 · 브랜치 전략 확정 — develop 통합·main 배포 이원화 (2026-07-17 · D-012 브랜치 항목 개정)
- **결정**: D-012의 "GitHub Flow(develop 없음)"를 개정 — **develop = 통합 브랜치, main = 배포 브랜치**로 공식화. feature는 develop에서 분기(`feature/{이슈번호}-{kebab}`), PR base = develop. develop→main 승격(배포)은 사람이 결정·실행. develop이 main보다 앞서 있는 것은 **정상 상태**(배포 전 통합분).
- **이유**: 사용자 결정(2026-07-17) — main을 배포용으로 쓰는 운영 감각이 실제 관행(PR #5·#7·#9·#11 모두 develop 병합)과 이미 일치. 문서(D-012)만 실무와 어긋나 있었으므로 문서를 실무에 맞춘다.
- **비고**: STATE의 "develop 처리 결정" 게이트 해소. develop 삭제 항목(BACKLOG A) 폐기. main 승격 주기는 마일스톤 단위로 추후 감 잡히면 기록.

## D-024 · gh CLI 도입 — 에이전트는 읽기 전용, 쓰기 게이트 실효화 (2026-07-15 · 쓰기 위임으로 개정 → D-026)
- **결정**: gh CLI 설치·인증 완료(사람, 2026-07-15). 에이전트에 **gh 읽기 명령**(issue/pr `list`·`view`, pr `diff`·`checks`, `repo view`, run `list`·`view`, `search`) allow. 쓰기는 D-013·D-014 그대로 사람 전담 — 기존 deny(issue/pr create, pr merge)에 issue/pr `comment`·`edit`·`close`, issue `delete`, pr `review`·`ready`, `release`, repo `create`/`edit`/`delete`, `workflow run`, `secret`, `variable set`을 추가해 게이트를 실효화. `gh api`는 읽기·쓰기 겸용이라 allow/deny 모두 제외(프롬프트 백스톱).
- **이유**: gh 부재 시 deny 게이트는 장식이었으나 설치로 실제 게이트가 됨. 에이전트가 이슈·PR·리뷰 코멘트를 직접 조회하면 초안 작성·리뷰 반영의 컨텍스트 수집이 빨라짐. 쓰기 주도권(D-013)은 불변.
- **비고**: D-011(feature push 허용)은 여전히 보류(D-013). settings 변경은 루트·sift-api 두 사본 동일 적용(D-023 사본 정합 대상). 부수 확인(gh api 읽기로 검증): **sift-api main 보호 규칙은 이미 설정됨** — PR 필수·strict status checks·force push/삭제 금지. 단 `enforce_admins=false`(관리자 본인은 직접 push 가능)·필수 승인 리뷰어 0명은 1인 운영으로 수용. sift-docs main은 미보호 — 사람이 직접 커밋·push하는 레포라 수용(원하면 추후 설정).

## D-023 · 문서 아키텍처 = SSoT 원본 지도 + 참조 강등 (2026-07-14)
- **결정**: 모든 사실에 원본 1곳을 지정하고, 다른 문서는 요약 사본이 아니라 **링크 참조로 강등**한다. **원본 지도** — 결정 = DECISIONS · 확정 설계(스키마·포트·배치) = 각 레포 `docs/` · 진행 상태 = STATE(현재·다음 중심, 완료 이력은 최근 4~5건 + git 히스토리) · 작업 목록 = TASKS(이슈 후보) + BACKLOG(현 이슈 단계) · 작동 방식 = HARNESS(상태 표기 금지) · 세션 간 임시물 = `.omc/notepad`(스크래치패드 사용 금지) · graphify = 파생 인덱스(원본 아님, gitignore). 지침 충돌 시 우선순위: settings.json 게이트 > CLAUDE.md·HARNESS·DECISIONS > OMC 기본 지침.
- **이유**: 문서 아키텍처 리뷰(2026-07-14)에서 실드리프트 다수 확인 — `.coderabbit.yaml`의 D-018 미반영(리뷰어가 폐기된 소유권 규칙으로 동작), HARNESS §2 상태 체크박스 미갱신, BACKLOG M1-4 상태 불일치, D-017 본문의 폐기 규칙 잔존, STATE 비고의 죽은 기록(coderabbit path 결함은 98ec1d5에서 이미 해소). 공통 원인 = 동일 사실의 다중 수동 사본. 정합을 기억(규칙)이 아니라 구조(원본 지도)와 장치로 강제한다.
- **후속 (Medium — M1-5~7 사이클에 편입)**: ① `/harness-check` 정합 검사 스킬 박제(settings 사본 diff·루프 문서 상태 일치·크로스 레포 링크·최신 D-번호의 coderabbit/CLAUDE.md 영향) ② 구조 변경 이슈 DoD에 `.coderabbit.yaml`·CLAUDE.md 요약 영향 확인 추가(D-017 확장) ③ PLAN §5·SELECTION §3 중복 스키마의 MVP-DESIGN 링크 치환 ④ 이슈 본문 체크리스트 폐지 — BACKLOG 단일 장부(이슈 본문은 배경+DoD만) ⑤ graphify 최초 빌드(M1-7 직후, 이후 병합 시 `--update`).
- **후속 (Long-term — 트리거 도달 시에만)**: TASKS→GitHub 이슈 승격(열린 이슈 20+) · DECISIONS 파일 분할(결정 30+) · MVP-DESIGN 모듈별 분할(M3 착수) · 문서 링크 CI(첫 GitHub Actions와 함께) · 루트 로컬 전용 git(harness-check 불일치 2회 보고 시, D-015 비고 재검토) · 커밋 게이트 완화 검토(Phase 2 진입, D-010 비고) · graphify wiki/MCP(2번째 코드 레포 활성화).
- **비고**: D-012 완결 — `implement-feature`·`unit-test`도 `~/.claude/skills-archive/`로 격리(타 프로젝트 관용구 잔존: `p_` 접두사·soft delete·BDDMockito 강제·존재하지 않는 SPEC.md 참조. Sift 골든패스의 fake 포트 테스트 패턴과 충돌). M1-7에서 Sift 기반으로 재박제. 컨텍스트 맵은 `.drawio.svg`로 변환해 sift-docs 버전 관리에 편입(D-017 형식 준수). 부수 발견: PORTFOLIO.md 유실(STATE 정정 참조).

## D-022 · `Source.markCrawled()` 영속 반영 = 전용 포트 (2026-07-13)
- **결정**: `UpdateSourcePort.markCrawled(sourceId, at)` outbound port를 신설해 `SourcePersistenceAdapter`가 구현한다. `CrawlSourcesService`는 도메인 객체의 `markCrawled()` 호출과 별개로 이 포트를 호출해 영속화한다(MVP-DESIGN §6 열린 질문 해결).
- **이유**: `CrawlSourcesService`는 트랜잭션 경계를 갖지 않는 순수 애플리케이션 서비스이고, `LoadActiveSourcesPort`가 반환하는 `Source`는 이미 영속 컨텍스트에서 분리된 도메인 POJO라 dirty checking으로 자동 반영되지 않는다. `SaveArticlePort`와 동일한 관용구(전용 outbound port)를 따르는 것이 헥사고날 경계를 지키면서 가장 단순하다.
- **비고**: 어댑터 내부는 `@Transactional` + JPA dirty checking으로 반영(명시적 `save()` 호출 불필요). ERD(§2)의 `source.trust_score` 컬럼은 이번 영속 어댑터 범위에서 제외 — 현재 도메인 모델(M1-2)에 필드가 없고 사용하는 유스케이스도 없어 컬럼을 추가하지 않음(YAGNI). M2 스코어링 구현 시 도메인·엔티티에 함께 추가 검토.

## D-021 · 문서 이원화 — 공통 문서는 `sift-docs` 레포 공개, 레포 종속 설계 문서는 해당 레포 `docs/` (2026-07-09)
- **결정**: ① 레포 종속 설계 문서(MVP-DESIGN·SELECTION·EVENTS)는 **해당 코드 레포**(`sift-api/docs/`)로 이동해 원본으로 관리 — 구조 변경 PR에서 코드와 같은 diff로 리뷰·정합 유지. ② 워크스페이스 공통 문서(PLAN·HARNESS·STATE·BACKLOG·TASKS·DECISIONS)는 루트 공통 문서 디렉터리(구 `docs/` → `sift-docs/`로 개명)를 git 저장소화해 **`siftnews/sift-docs`로 공개** — 갱신 커밋 히스토리 자체가 루프 운영의 증거. ③ 스냅샷 복사(구 §0.9 위치 규칙)는 폐기 — 원본 단일화로 drift 제거. 크로스 레포 참조는 절대 URL. **D-015 부분 개정**(루트 자체는 여전히 git 금지, `sift-docs/`만 하위 레포화), D-017 위치 규칙 개정.
- **이유**: 정석 정렬 — docs-as-code(코드 종속 문서는 코드와 함께 버전·리뷰) + 레포를 가로지르는 문서는 org 공통 레포. 스냅샷 이원화는 수동 동기화라 drift가 필연.
- **비고**: 레포 이름은 D-002 `sift-` 접두어 규칙 + PLAN §저장소 구조의 기존 예정명을 따라 `sift-docs`. 루프 문서가 public이 되므로 커밋 주체·주기(사이클마다 vs 모아서, D-013 완화 여부)는 추후 결정 필요.

## D-020 · M1 작업 단위 = DDD inside-out + 단계별 테스트 DoD (2026-07-07)
- **결정**: M1 골든패스 이슈를 **DDD inside-out 순서**로 재편 — ① 도메인 모델 → ② 유스케이스(포트·서비스) → ③ 영속 어댑터 → ④ RSS 어댑터 → ⑤ 배치+측정 → ⑥ 패턴 박제 (TASKS M1-2~7). 각 이슈의 DoD는 컴파일이 아니라 **해당 레벨의 실행 가능한 테스트 통과**: 도메인 단위 → fake 포트 서비스 단위 → Testcontainers 통합 → RSS 픽스처 → bootRun E2E(👤 게이트).
- **이유**: 사용자 방법론 결정 — 도메인 주도 개발 원칙 준수 우선. 수직 슬라이스(워킹 스켈레톤 A/B/C/D) 안은 기각. 레이어 분할의 원 문제(중간 이슈에 검증 가능한 산출물 없음)는 단계별 테스트 DoD로 해소.
- **비고**: 트레이드오프 — 시스템 첫 E2E 기동은 M1-6. 설계 선행(컨텍스트 맵·D-018·D-019·EVENTS 확정)이 통합 리스크의 완충재. 구현 전 클래스 다이어그램 작성 (D-017 규칙 ①).

## D-019 · 발송 모델 = 매일 고정 + 구독자 선호 시각, dispatch는 매시 pull 스캔 (2026-07-07)
- **결정**: ① 발송 주기는 **전 토픽 DAILY 고정** (topic.cadence·send_at_hour 제거, econ WEEKLY 시드도 DAILY로). ② 발송 시각은 **구독자가 선택** — `subscriber.preferred_send_hour`(시 단위). ③ 선별은 매일 발행 기준 시각(예: 06:00, 구현 이슈에서 확정)에 토픽당 1회 실행해 당일 이슈 생성 (D-003 유지). ④ 발송 기동은 push(IssueScheduled 이벤트)가 아니라 **dispatchJob 매시 정각 pull 스캔**: 당일 SCHEDULED 이슈 × `preferred_send_hour = 현재 시각`인 ACTIVE 구독자 → 스냅샷·발송. **IssueScheduled 이벤트는 MVP 제외.**
- **이유**: 제품 의도 확정 — "구독자가 이메일 + 토픽 + 수신 시각을 정하고, 매일 그 시각에 받는다." 발송 시점의 주인이 구독자이므로 이슈 확정 시점 ≠ 발송 시점 → 시간대 스캔(pull)이 자연스럽고, 확정 즉시 발송하는 push는 성립하지 않음.
- **비고**: 부하 서사는 시드로 유지 — 선호 시각을 현실 분포(아침 피크 집중) 또는 최악(전원 단일 시각)으로 시드해 버스트 측정. 신선도 트레이드오프(늦은 시각 구독자는 오래된 이슈 수신)는 MVP 수용, 개선 후보로 기록. preferred_send_hour는 subscriber 레벨(구독별 아님 — MVP 단순화, 개인 다이제스트 단계에서 재검토). 멱등 키를 `hash(delivery_job_id, subscriber_id)` → `hash(issue_id, subscriber_id)`로 바꿀지 구현 이슈에서 검토(발송 후 시각 변경 시 이중 발송 방지).

## D-018 · Article 애그리거트 = Source 컨텍스트 소유 (2026-07-07)
- **결정**: `Article`(과 article 테이블)은 **Source 컨텍스트가 소유**한다. Content는 후보 기사를 Source가 노출하는 **named interface(port.in) 경유로 조회**만 한다 (Modulith 규칙 준수). PLAN §3의 "기사 저장 = Content 책임" 기술은 오기로 보고 정정.
- **이유**: 기사는 수집·정규화의 산물 — 생산자인 Source가 저장·스키마(URL 정규화, UNIQUE(normalized_url) 멱등)를 책임지는 것이 응집도에 맞다. 두 컨텍스트가 한 테이블을 공유하는 모호함 제거 (바운디드 컨텍스트 매핑 중 발견·결정).
- **비고**: ⚠️ 열린 꼬리 하나 — SELECTION의 dedup은 `article.dedup_cluster_id`를 **갱신**하는데, 이는 Content가 Source 소유 데이터를 쓰는 행위. M2 Dedup 이슈에서 해소 방안 결정 필요: ① Source named interface에 갱신 오퍼레이션 추가 ② 클러스터 정보를 Content 측 테이블로 분리. 결정 시 이 항목 갱신.

## D-017 · 설계 문서화 규칙 — 다이어그램 1급 산출물화 (2026-07-07 · 위치 규칙 개정 → D-021)
- **결정**: 구조에 영향 주는 작업은 **설계 산출물(다이어그램) 먼저 작성·갱신 후 구현**한다. 도구는 정형 다이어그램(ERD·상태머신·시퀀스·플로우) = Mermaid 기본, 자유형(컨텍스트 맵·아키텍처 오버뷰) = draw.io(`.drawio.svg`). 작업용은 루트 `docs/`(로컬 전용, D-015), 포트폴리오 노출 최종본은 해당 sift-* 레포에 복사·커밋. 규칙 상세는 [HARNESS §0.9](./HARNESS.md).
- **이유**: 사용자 요구 — 설계 측면의 문서를 제대로 만들고 넘어가는 습관을 하네스에 강제. Mermaid는 에이전트가 직접 작성·갱신 가능(자동화 적합), draw.io는 표현력이 필요한 그림에 사람이 사용.
- **비고**: 구조 변경 이슈 DoD에 다이어그램 갱신 포함. 첫 적용 대상 = 바운디드 컨텍스트 검증(컨텍스트 맵 + 이벤트 흐름).

## D-016 · 멀티세션 운영 규칙 — 역할별 탭 병렬 + STATE 쓰기 주인 단일화 (2026-07-05)
- **결정**: 터미널 탭별 역할 세션(문서화/구현/리뷰) 병렬 운영을 허용한다. 단 ① 모든 세션은 `siftnews/` 루트에서 실행, ② 루프 문서(STATE·BACKLOG·TASKS·DECISIONS)의 쓰기는 **구현(루프) 세션 1개만** 담당하고 나머지는 읽기 전용, ③ 같은 코드 파일 동시 편집 금지(구현 병렬화가 필요하면 git worktree). 규칙 상세는 [HARNESS §0.8](./HARNESS.md).
- **이유**: 리뷰의 컨텍스트 격리(자기 코드 자기 리뷰 편향 방지)는 이득이지만, 루프 문서는 잠금 없는 단일 진실원천이라 다중 쓰기 시 갱신 유실이 발생. 쓰기 창구를 하나로 좁혀 조율.
- **비고**: 루트는 git이 없으므로(D-015) 문서 충돌을 머지로 복구할 수 없음 — 규칙 준수가 유일한 방어선.

## D-015 · 루트 `siftnews/`는 로컬 전용 — git 저장소화하지 않음 (2026-07-05 · 부분 개정 → D-021: sift-docs만 레포화)
- **결정**: 워크스페이스 루트(`siftnews/` — `docs/`·`.claude/`·루트 `CLAUDE.md` 포함)는 **git 저장소로 만들지 않고 로컬에만 둔다**. git은 하위 실제 프로젝트 레포(`sift-api`, 향후 `sift-web` 등)에만 존재한다. 기존 게이트 항목 "루트 저장소화"는 취소.
- **이유**: 사용자 결정 — GitHub에는 실제 산출물 레포만 노출하고, 하네스·설계 문서는 로컬 작업 환경으로 유지.
- **비고**: 트레이드오프 — `docs/`·`.claude/`는 버전 이력·백업이 없으므로 문서-코드 불일치(예: 2026-07-05 source 골격 유실 정정)를 git으로 추적할 수 없음. STATE '정정' 섹션 같은 문서 내 기록으로 보완. 필요해지면 로컬 전용 git(원격 없음)으로 재검토 가능.

## D-014 · git 커밋도 사람 전담 — 에이전트는 커밋 포인트 제안 (2026-07-05 · 커밋 위임으로 개정 → D-026)
- **결정**: D-013 확장 — `git commit`도 **사람이 실행**한다. 에이전트는 이슈 내 작은 작업(BACKLOG 단계) 1개를 마치고 자가검증을 통과할 때마다 **변경 파일 목록 + 제안 커밋 메시지**를 응답으로 제시하고, 사람의 커밋 확인 후 다음 작업으로 진행한다. settings.json deny에 `git commit` 추가로 강제.
- **이유**: 사용자 선호 — 커밋 이력도 본인 손으로. 작은 작업 단위의 커밋 리듬을 대화로 유지.
- **비고**: 메시지 형식 `{type}: {한국어 요약}` (feat|fix|refactor|test|chore|build|docs, 기존 이력 스타일). 한 줄, 트레일러 없음(사람 커밋). 커밋 단위 = 작은 작업 1개.

## D-013 · GitHub 쓰기 작업 = 사람 전담 (2026-07-04 · 쓰기 위임으로 개정 → D-026)
- **결정**: 이슈 발행·`git push`·PR 생성·병합은 **사람이 실행**한다. 에이전트는 태스크 트리 구성·분해(`docs/TASKS.md`), 이슈/PR **초안** 작성, 브랜치·구현·커밋·자가검증까지 담당. D-011의 "보호 설정 후 feature push 에이전트 허용"은 **보류** — settings.json의 push deny 유지.
- **이유**: 사용자 선호 — GitHub 활동 기록(이슈·PR)을 본인 주도로 남기고, 외부로 나가는 쓰기는 직접 통제.
- **비고**: 역할 분담·태스크 계층 매핑은 HARNESS §0.7. gh CLI는 에이전트 필수 아님(사람 편의). 원격 `develop` 브랜치 삭제는 사람 몫(로컬은 삭제됨).

## D-012 · 타 프로젝트 글로벌 스킬 격리 + 브랜치 전략 = GitHub Flow (2026-07-04 · 스킬 유지 항목 개정 → D-023 · 브랜치 전략 개정 → D-025: develop 통합·main 배포)
- **결정**: ① `~/.claude/skills/`의 타 프로젝트(MSA 4계층) 스킬 — `code-review`, `full-feature-cycle`, `create-branch`, `feign-client-*`, `kafka-event-handling`, `http-test` — 을 `~/.claude/skills-archive/`로 격리. Sift용은 골든패스 완료 후 `siftnews/.claude/skills/`에 자체 박제(Phase 1). ② 브랜치 전략 명칭을 **GitHub Flow**로 정정 — `main`에서 `feature/{이슈번호}-{kebab-case}` 분기 → PR → squash 병합. develop/release 레이어 없음.
- **이유**: 타 프로젝트 스킬이 자동 트리거되어 Sift의 Modulith·헥사고날 규칙과 배치되는 패턴(4계층·soft delete·FeignClient)을 주입할 위험. HARNESS §0.7의 실제 흐름은 Git Flow가 아니라 GitHub Flow였음.
- **비고**: `implement-feature`(Phase 1 재활용 예정)·`unit-test`(범용)는 유지. 아카이브 스킬은 원 프로젝트 `.claude/skills/`로 이전 권장.

## D-011 · push 게이트 개정 — main 보호 기반 (2026-07-04 · 보류 → D-013: push deny 유지)
- **결정**: 게이트 대상은 "push 자체"가 아니라 **main 병합/변경**. main 브랜치 보호 규칙(직접 push 금지·PR 필수) 설정 후에는 feature 브랜치 push를 에이전트에 허용(settings.json deny 해제 + `gh issue`/`gh pr` allow). 보호 규칙 설정 전까지는 현행 deny 유지.
- **이유**: §0.7 협업 흐름은 에이전트의 push+PR 생성을 전제하는데 D-007이 push 전체를 게이트로 규정해 충돌. feature push는 브랜치 보호가 있으면 되돌리기 어려운 행위가 아님. 로컬 커밋은 원래 게이트 아님(되돌리기 가능).
- **비고**: D-007 부분 개정. 선결: gh CLI 설치·인증 + main 보호 설정 (BACKLOG A).

## D-010 · GitHub 협업 흐름 채택 (2026-06-30)
- **결정**: 작업 단위마다 이슈 → 브랜치 → PR → 리뷰 → 병합. **이슈 1개 = 루프 1회분 = PR 1개**(입도), 이슈→BACKLOG 분해는 에이전트. main 직접 커밋 금지(브랜치 보호). 규칙은 [HARNESS §0.7](./HARNESS.md).
- **이유**: 사용자 워크플로우 요구 + 포트폴리오 활동 기록. 루프 *성립* 필수는 아니나, 의도적 통합 레이어로 추가.
- **비고**: gh CLI 설치·인증 + main 보호 설정이 선결. 병합 실행은 초기 사람(추후 에이전트 승격 가능).

## D-009 · 도메인 ↔ JPA 엔티티 분리 (2026-06-30 · 2026-07-22 확정)
- **결정**: `domain` POJO와 `persistence` JPA 엔티티를 분리 (헥사고날).
- **이유**: 도메인이 영속 기술에 오염되지 않게. 골든패스가 이후 모든 컨텍스트의 템플릿이 됨.
- **상태**: **확정 (2026-07-22)** — 골든패스(Source/Article) 코드 검토 결과 분리 패턴이 단위·통합 테스트와 `ApplicationModules.verify()` 통과로 검증됨. 도메인 순수성·헥사고날 정석 유지 우선(정석 선호). 겸용 전환 안 함. content 모듈 Topic(M2-1)부터 동일 패턴 적용. 사용자 결정.

## D-008 · 루프/하네스 운영 골격 도입 (2026-06-30)
- **결정**: `STATE.md` + `BACKLOG.md` + `DECISIONS.md` + 루프 프로토콜(HARNESS §0.6)로 반복·자율 개발을 운영.
- **이유**: 루프 성공 조건 = 상태 외부화 + 작업 분해 + 명확한 완료/게이트 기준.

## D-007 · 게이트 = 되돌리기 어려운 것만 (2026-06-30 · push 게이트 항목 개정 → D-011·D-013)
- **결정**: 사람 게이트(deny)는 *되돌리기 어려운 명령*만 — `git push`, `git reset --hard`, `git clean`, `docker compose down -v`. **build·test·컴파일은 에이전트 허용** → 루프가 빌드·테스트까지 자가검증.
- **이유**: 부작용 없는 검증까지 막으면 루프 자율성이 떨어진다. 차단은 손실·외부영향이 큰 것에 집중.

## D-006 · 하네스 위치 = `siftnews/` 상위 (2026-06-30)
- **결정**: `.claude/`, `CLAUDE.md`를 모노레포 루트에. sift-api·sift-web·docs를 모두 아우름.

## D-005 · 하네스 철학 정렬 (2026-06-30)
- **결정**: 워크플로우/에이전트 경계 명문화(§0.5) + evals 조기화(원칙 5).
- **이유**: Anthropic 권장(단순성 우선·측정 우선)에 정렬.

## D-004 · Spring Modulith 채택 (2026-06-27)
- **결정**: Modulith 모듈 = 바운디드 컨텍스트. 모듈 간 통신은 도메인 이벤트 우선.
- **이유**: 5개 컨텍스트 경계를 빌드에서 강제 + Kafka(V4)로 가는 길.

## D-003 · 선별 = 토픽 구독 + 스코어링·랭킹 (2026-06-25)
- **결정**: 토픽 단위 구독, 이슈는 토픽당 1회분, 5단계 파이프라인. (→ SELECTION.md)
- **이유**: 선별을 토픽당 1회 → 대량 발송 성능 서사와 충돌 없음.

## D-002 · 조직 핸들 `siftnews`, 브랜드 `Sift` (2026-06-26)
- **결정**: GitHub 조직 `siftnews`(sift 선점됨), 레포 `sift-` 접두어, 멀티레포.

## D-001 · 프로젝트 = Sift (2026-06-25)
- **결정**: 뉴스 수집·선별·대량발송 뉴스레터 배치. 목적은 포트폴리오(성능 개선 서사).
- **이유**: "원하는 뉴스만 걸러주기" 동기 → 장기적으로 개인화로 확장.
