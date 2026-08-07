# Sift 태스크 사이클 워크플로우

사용자가 **“다음 태스크를 탐색하고 진행하라”**고 요청하면 이 문서를 단일 실행 계약으로 사용한다.
작업은 루트 `siftnews/`에서 조율하고, Git·빌드·테스트 명령은 대상 하위 저장소에서 실행한다.

## 상태 계약

현재 사이클 상태는 `.omx/state/task-cycle.json`에 기록한다. 상태 파일은 원본 문서가 아닌
세션 복구용 런타임 상태다. 각 단계는 다음 중 하나다.

```text
discover → issue → branch → design → implement → verify → docs
         → commit → push → pr → review → review-fix (반복) → ready
```

상태에는 최소한 `issue`, `repo`, `branch`, `phase`, `pr`, `head_sha`, `review_head_sha`,
`verification`, `blocker`를 기록한다. 단계가 바뀔 때마다 상태를 갱신하고, 실패·사용자 결정 대기·외부 서비스 지연은 `blocker`에 남긴다.

## 실행 원칙

- 기존 활성 이슈와 워킹 트리 변경을 먼저 복구한다. 기존 변경을 삭제·stash·reset하지 않는다.
- 한 사이클은 하나의 이슈와 하나의 구현 브랜치만 소유한다.
- 되돌릴 수 있는 로컬 구현·테스트·문서 갱신은 자동 진행한다.
- 이슈 생성, 커밋, feature 브랜치 push, PR 생성은 사용자가 이 사이클에 명시적으로 위임한 범위로 자동 진행한다.
- PR merge, main 공개 상태 변경, 배포, 데이터 삭제·백필, 외부 운영 발송은 사용자 결정 게이트다.
- CodeRabbit 리뷰를 받으면 actionable 지적은 요구사항·코드·테스트와 대조해 반영하고, 반영하지 않는 지적은 근거를 기록한다. 리뷰 답글을 사용자 명의로 작성하지 않는다.

## 단계별 절차

### 1. 다음 작업 탐색 (`discover`)

1. `sift-docs/STATE.md`, `BACKLOG.md`, `TASKS.md`를 읽는다.
2. `sift-api`와 `sift-docs`의 `git status --short --branch`를 확인한다.
3. `gh issue list --repo siftnews/sift-api --state open`으로 원격 열린 이슈를 조회한다.
4. `STATE`의 현재 작업·`BACKLOG`의 진행 이슈·현재 브랜치가 있으면 그것을 우선 재개한다.
5. 활성 작업이 없을 때만 `TASKS.md`의 첫 미완료 후보와 GitHub 이슈를 제목·범위·중복 여부로 대조해 다음 작업을 선택한다.
6. 선택 근거와 예상 영향 범위를 상태 파일에 기록한다.

### 2. 이슈 확인 또는 생성 (`issue`)

1. 기존 이슈가 있으면 `gh issue view`로 본문과 상태를 확인한다.
2. 없으면 [이슈 생성 워크플로우](create-issue.md)에 따라 제목·`Description`·`TODO`·assignee·label을 한 번에 만든다.
3. 생성·조회 직후 제목, 두 본문 섹션, `chltjsdl0119` assignee, 유형 label을 다시 검증한다.
4. 검증되지 않은 이슈에서는 브랜치 생성과 구현을 시작하지 않는다.

### 3. 브랜치 준비 (`branch`)

1. 기본 통합 브랜치는 `develop`, 배포 브랜치는 `main`이다.
2. 기능 브랜치는 `feature/<issue-number>-<kebab-slug>`, 버그는 `fix/<issue-number>-<kebab-slug>`, 정리는 `chore/<issue-number>-<kebab-slug>` 형식으로 만든다.
3. `sift-api`에서 `origin/develop`을 fetch한 뒤 브랜치를 생성·체크아웃한다.
4. 브랜치가 이미 있고 해당 이슈의 변경이면 재사용한다. 다른 이슈의 dirty 변경이 있으면 중단하고 사용자에게 보고한다.
5. `sift-docs` 변경은 별도 저장소의 현재 브랜치·기존 변경을 보존한다. 코드 레포에 문서 원본을 복사하지 않는다.

### 4. 설계·규약 확인 (`design`)

1. `sift-docs/references/coding-conventions.md`, `development.md`, `workflow-routing.md`, `harness.md`를 읽는다.
2. 영향 범위에 따라 architecture, database, observability, API, security, rollback 문서를 추가로 읽는다.
3. `sift-api` 코드 관계 질문은 기존 `sift-api/graphify-out/`에 대해 query/path/explain을 먼저 수행한다.
4. 구조·계약·상태 전이 변경이면 구현 전에 설계 원본과 Mermaid/다이어그램을 갱신한다.
5. 기존 ADR과 충돌하는 선택은 새 ADR로 결정하고, 사용자 결정 게이트가 필요한 분기는 구현하지 않는다.

### 5. 구현 (`implement`)

1. 헥사고날 경계와 Spring Modulith named interface를 지킨다.
2. 모듈 간 직접 internal 참조 대신 도메인 이벤트 또는 공개 named interface를 사용한다.
3. 테스트를 먼저 추가해 실패를 확인하고 작은 단위로 구현한다.
4. 기존 사용자 변경을 보존하며, 같은 하위 저장소의 순차 의존 변경은 한 소유자가 끝까지 수행한다.

### 6. 검증 (`verify`)

순서대로 실행하고 출력에서 성공을 확인한다.

1. 변경 대상 테스트
2. `sift-api`에서 `./gradlew test`
3. `ApplicationModulesTest`와 조건부 통합·보안·성능·마이그레이션 검증
4. `git diff --check`, 컴파일·빌드·정적 분석·포맷 검사
5. 실패 시 `verify`를 통과할 때까지 구현으로 돌아간다.

### 7. SSOT 갱신 (`docs`)

1. `STATE.md`의 현재 작업·다음 우선순위를 갱신한다.
2. `BACKLOG.md`의 진행 이슈 단계와 완료 조건을 갱신한다.
3. `TASKS.md`의 후보 상태를 갱신한다.
4. 설계·ADR·관측 문서와 체크리스트를 코드 변경과 정합하게 갱신한다.
5. 두 저장소의 변경을 각각 확인하고 원본 문서를 코드 레포에 복사하지 않는다.

### 8. 커밋 (`commit`)

1. 두 저장소의 diff를 각각 검토하고 의도하지 않은 파일을 제외한다.
2. 커밋 메시지는 `{type}: {한국어 요약}` 한 줄이다. 허용 type은 `feat`, `fix`, `refactor`, `test`, `chore`, `build`, `docs`다.
3. 트레일러와 `Co-Authored-By`를 넣지 않는다.
4. 코드 레포와 문서 레포는 각각 의미 있는 단위로 커밋한다.
5. 커밋 후 `git show --stat --check`로 커밋을 검증한다.

### 9. Push 및 PR (`push`, `pr`)

1. `sift-api`는 feature/fix/chore 브랜치만 `origin`으로 push하고 `main` 직접 push는 하지 않는다. `sift-docs`는 D-024에 따라 문서 원본을 현재 `main`에 직접 커밋·push한다.
2. [PR 생성 워크플로우](create-pr.md)에 따라 base `develop`, 연결 이슈 `close #N`, assignee·label, Ready for review를 확인한다.
3. PR 제목은 연결 이슈 제목과 문자 단위까지 일치시킨다.
4. PR 생성 직후 `gh pr view`, CI 상태, CodeRabbit 활성 상태, head SHA를 기록한다.

### 10. CodeRabbit 리뷰 루프 (`review`, `review-fix`)

1. 현재 PR head SHA에 대한 CodeRabbit 리뷰 제출·인라인 스레드·CI를 조회한다. 이전 head의 지적과 안내성 댓글은 제외한다.
2. 리뷰가 지연되면 짧게 재조회하되, 무기한 대기하지 말고 상태를 기록한다.
3. actionable 지적마다 `accept`, `reject`, `needs-decision`으로 분류한다.
4. `accept`는 코드를 수정하고 관련 테스트를 실행한다. `reject`는 근거를 기록한다. `needs-decision`은 사용자 게이트로 종료한다.
5. 수정이 있으면 새 커밋·push 후 새 head SHA의 리뷰만 다시 조회한다.
6. 반영한 스레드만 resolve하고, 반박·미검증·결정 대기 스레드는 열어 둔다. 스레드 답글은 사용자 명의로 작성하지 않는다.
7. actionable 미해결 스레드와 CI 실패가 없을 때만 `ready`로 전환한다.

### 11. 종료 (`ready`)

최종 보고에는 이슈·브랜치·커밋·PR·최신 head SHA·테스트 결과·CodeRabbit 처리 결과·남은 사용자 게이트를 포함한다. `ready`는 PR 생성 및 리뷰 루프 종료 상태이며 merge·배포 완료를 뜻하지 않는다.
