# D-021 · 문서 이원화 — 공통 문서는 `sift-docs` 레포 공개, 레포 종속 설계 문서는 해당 레포 `docs/` (2026-07-09 · ① 전면 개정 → D-038, 설계 문서도 sift-docs로 통합)

> [ADR 인덱스](README.md) · 결정 ID: D-021

- **결정**: ① 레포 종속 설계 문서(MVP-DESIGN·SELECTION·EVENTS)는 **해당 코드 레포**(`sift-api/docs/`)로 이동해 원본으로 관리 — 구조 변경 PR에서 코드와 같은 diff로 리뷰·정합 유지. ② 워크스페이스 공통 문서(PLAN·HARNESS·STATE·BACKLOG·TASKS·DECISIONS)는 루트 공통 문서 디렉터리(구 `docs/` → `sift-docs/`로 개명)를 git 저장소화해 **`siftnews/sift-docs`로 공개** — 갱신 커밋 히스토리 자체가 루프 운영의 증거. ③ 스냅샷 복사(구 §0.9 위치 규칙)는 폐기 — 원본 단일화로 drift 제거. 크로스 레포 참조는 절대 URL. **D-015 부분 개정**(루트 자체는 여전히 git 금지, `sift-docs/`만 하위 레포화), D-017 위치 규칙 개정.
- **이유**: 정석 정렬 — docs-as-code(코드 종속 문서는 코드와 함께 버전·리뷰) + 레포를 가로지르는 문서는 org 공통 레포. 스냅샷 이원화는 수동 동기화라 drift가 필연.
- **비고**: 레포 이름은 D-002 `sift-` 접두어 규칙 + PLAN §저장소 구조의 기존 예정명을 따라 `sift-docs`. 루프 문서가 public이 되므로 커밋 주체·주기(사이클마다 vs 모아서, D-013 완화 여부)는 추후 결정 필요.
