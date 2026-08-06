# GitHub 이슈 생성 워크플로우

이슈는 구현 작업의 원본이다. 작업 시작 전에 다음을 완료한다.

1. 작업 유형에 맞는 `.github/.github/ISSUE_TEMPLATE/` 템플릿을 선택한다.
2. 제목을 `[FEAT|FIX|REFACTOR|CHORE] 한국어 요약`으로 작성한다. 테스트 인프라·테스트 안정화는 `[CHORE]`를 사용한다.
3. `## Description`에 문제·범위·완료 조건을, `## TODO`에 검증 가능한 작업 단위를 채운다.
4. 생성 시 `--assignee chltjsdl0119`와 유형별 `--label`을 함께 지정한다.
   - FEAT → `feature`
   - FIX → `bug`
   - REFACTOR → `chore`
   - CHORE → `chore`
5. 생성 직후 이슈를 다시 조회해 제목, `Description`, `TODO`, Assignees, Label을 확인한다. 모두 충족하기 전에는 브랜치 생성·구현을 시작하지 않는다.

예시:

```bash
gh issue create \
  --repo siftnews/sift-api \
  --title "[FEAT] 한국어 요약" \
  --body-file /tmp/issue-body.md \
  --assignee chltjsdl0119 \
  --label feature
```

Assignees·Label을 누락했다면 `gh issue edit`로 즉시 보정한다. 제목·본문 수정은 사람 영역이며, 이 워크플로우는 기존 내용 변경을 허용하지 않는다.
