# 코드 규약 (sift-api)

> **이 문서는 지어낸 규칙이 아니라 `origin/develop`에서 추출한 관행이다.** 코드가 바뀌면 이 문서가 틀린 것이므로 함께 고친다.
> 원본이 다른 곳에 있는 것은 링크만 둔다 — 아키텍처 결정은 [DECISIONS](../adr/DECISIONS.md), ERD·배치 설계는 [MVP-DESIGN](./MVP-DESIGN.md), 개발 절차는 [HARNESS](./HARNESS.md).

구현 세션이 매번 코드를 읽어 스타일을 역추론하지 않도록 하는 것이 목적이다. **여기 없는 것은 규약이 아니다** — 판단이 필요하면 추정하지 말고 보고한다.

---

## 1. 패키지 배치

```
com.siftnews.{module}/
├─ domain/                      순수 자바. 프레임워크·JPA 의존 금지
├─ application/
│  ├─ port/in/    UseCase       인바운드 포트 (+ Summary 반환 레코드)
│  ├─ port/out/   Port          아웃바운드 포트
│  └─ service/    Service       유스케이스 구현
├─ adapter/
│  ├─ in/batch/                 Spring Batch Job·Step·Trigger·Listener
│  ├─ in/bootstrap/             기동 시 시드 (Seeder·SeedData)
│  ├─ out/persistence/          JPA
│  └─ out/{외부}/               rss, source(타 모듈) 등
├─ api/                         ★ 타 모듈에 공개하는 표면만 (named interface)
└─ package-info.java            @ApplicationModule
```

모듈은 `source` / `content` / `subscriber` / `delivery` / `tracking` (+ `common`). **모듈 = 바운디드 컨텍스트**이고 모듈 내부가 헥사고날이다.

`adapter/out/source/`처럼 **타 모듈을 부르는 것도 아웃바운드 어댑터**다 — `content`가 `source`의 카탈로그를 읽는 `ArticleCatalogAdapter`가 그 예다. 도메인·서비스는 상대 모듈을 모른다.

## 2. 네이밍

| 대상 | 규칙 | 실례 |
|---|---|---|
| 인바운드 포트 | `{동사}{대상}UseCase` | `CrawlSourcesUseCase` · `ScoreArticlesUseCase` |
| 유스케이스 결과 | `{동사}{대상}Summary` (record) | `ScoreArticlesSummary` · `RenormalizeSummary` |
| 아웃바운드 포트 | `{동사}{대상}Port` | `SaveArticlePort` · `LoadCandidateArticlesPort` · `UpdateArticleClusterPort` |
| 서비스 | `{동사}{대상}Service` | `NormalizeDedupService` |
| 영속 어댑터 | `{대상}PersistenceAdapter` | `ArticlePersistenceAdapter` |
| 조회 전용 어댑터 | `{대상}QueryAdapter` | `ArticleQueryAdapter` |
| JPA 엔티티·리포지토리 | `{대상}JpaEntity` · `{대상}JpaRepository` | `SourceJpaEntity` |
| 매퍼 | `{대상}Mapper` | `ArticleMapper` |
| 배치 | `{Job}JobConfig` · `{Job}JobParameters` · `{Job}MetricsListener` · `{Job}Trigger` | `CollectionJobConfig` |
| 예외 | `{Module}Exception` extends `BusinessException` | `ContentException` · `SourceException` |

**포트 이름의 동사는 그 포트가 하는 일 하나를 가리킨다.** `SaveArticlePort`·`ArticleQueryPort`처럼 저장과 조회를 나누는 것이 관행이다 — 리포지토리 한 덩어리로 묶지 않는다.

### 도메인 팩토리 메서드 — `create` / `restore` / `of`

세 이름의 의미가 다르므로 섞어 쓰지 않는다:

| 이름 | 뜻 | 실례 |
|---|---|---|
| `create` | **신규 생성.** 식별자 없음, 불변식 검증 수행 | `Article.create(raw, sourceId)` · `Source.create(...)` · `Topic.create(...)` |
| `restore` | **영속 상태 복원.** 식별자 포함, 검증 생략 (이미 저장된 것) | `Source.restore(sourceId, ...)` · `Topic.restore(...)` · `Issue.restore(...)` |
| `of` | **값 계산.** 입력에서 파생되는 값 객체 | `ClusterWindow.of(windowCandidates)` |

## 3. 도메인과 영속의 분리 (D-009)

- 도메인은 **순수 POJO**다. JPA 애너테이션이 붙지 않는다.
- 영속 계층은 `{대상}JpaEntity`가 따로 있고, 공통 상위 타입은 [`common/BaseEntity`](https://github.com/siftnews/sift-api/blob/main/src/main/java/com/siftnews/common/BaseEntity.java) — id(IDENTITY) + `createdAt`·`updatedAt`(JPA Auditing).
- 변환은 **package-private final 클래스 + static 메서드**로 손으로 짠다. MapStruct 같은 라이브러리를 쓰지 않는다.

```java
final class ArticleMapper {
    private ArticleMapper() {}
    static ArticleJpaEntity toEntity(Article article) { ... }
}
```

## 4. 모듈 경계

- 각 모듈 루트 `package-info.java`에 `@ApplicationModule(displayName, allowedDependencies)`. **`allowedDependencies`에 없으면 컴파일이 아니라 테스트가 깨진다** — `ApplicationModulesTest.verifiesModuleBoundaries()`가 강제한다(컨텍스트·DB 불필요, 빠름).
- 타 모듈에 무언가를 열어야 하면 **`{module}/api/` 패키지에만** 두고 `@NamedInterface`를 붙인다. 그 패키지 밖의 타입은 상대 모듈에서 보이지 않는다.
- 모듈 간 통신은 **도메인 이벤트 우선**. 동기 조회가 꼭 필요할 때만 named interface.

## 5. 시간 — `Clock` 주입 (`Instant.now()` 금지)

`ClockConfig`가 `Clock.systemUTC()` 빈을 제공한다. 도메인·서비스는 이를 주입받는다.

```java
private static final Clock CLOCK = Clock.fixed(NOW, ZoneOffset.UTC);   // 테스트
```

**이 규칙은 실제 결함에서 나왔다** — 시각 의존 결함이 하루 15시간 동안 테스트를 통과하던 사고(#35·PR #36). 시각이 개입하는 로직은 고정 시계로 검증할 수 있어야 한다.

## 6. 테스트

### 위치와 이름
`src/test/java`에 **main과 같은 패키지 구조를 미러링**하고 `{대상}Test`로 짓는다. 공용 지원 클래스는 `com.siftnews.support`.

### 세 층 — 대상에 따라 방식이 갈린다

| 층 | 대상 | 방식 |
|---|---|---|
| **순수 단위** | `domain/**` | 스프링 없음. 객체를 직접 만들어 검증 |
| **서비스** | `application/service/**` | 스프링 없음. **Fake 포트를 손으로 짜서 주입** |
| **통합** | `adapter/**`, 배치 Job | `AbstractIntegrationTest` 상속 + `@Transactional` |

### Fake 포트를 손으로 짠다 — **Mockito를 쓰지 않는다**

`src/test` 전체에 Mockito import가 **0건**이다. 대신 테스트 클래스 안에 `private record` 또는 `private static final class`로 포트 구현을 만든다:

```java
private record FakeLoadTopicPort(Topic topic) implements LoadTopicPort {
    @Override public Optional<Topic> load(Long topicId) { return Optional.ofNullable(topic); }
    @Override public List<Topic> loadActive() { return topic == null ? List.of() : List.of(topic); }
}

private static final class FakeSaveArticleScorePort implements SaveArticleScorePort {
    private final List<ArticleScore> saved = new ArrayList<>();
    @Override public void saveAll(List<ArticleScore> scores) { saved.addAll(scores); }
}
```

상태를 확인해야 하면 `final class` + 수집 필드, 값만 돌려주면 `record`. **저장된 내용을 직접 검증할 수 있어 `verify()`보다 읽기 쉽다**는 것이 이 선택의 이유다.

> 같은 Fake가 세 번째 테스트에서 필요해지면 `support/`로 올린다 (하네스 원칙 2). 현재는 전부 테스트 클래스 내부에 있다.

### 통합 테스트 — Testcontainers
[`support/AbstractIntegrationTest`](https://github.com/siftnews/sift-api/blob/main/src/test/java/com/siftnews/support/AbstractIntegrationTest.java)를 상속한다. `@SpringBootTest` + `@ActiveProfiles("test")` + `@ServiceConnection` PostgreSQL이 붙어 있다.

- **테스트에 `local` 프로파일 금지.** `application-test.yaml`은 `ddl-auto: create-drop`, 배치 Job 자동 실행 off.
- 컨테이너는 **static 블록 싱글톤**이다. `@Container` 라이프사이클을 쓰면 테스트 클래스마다 재기동돼 포트가 바뀌고, Spring 컨텍스트 캐시가 옛 포트를 참조해 실패한다.
- 이미지 태그는 `TestContainerImages` 상수로만 참조한다 (`postgres:16-alpine`).
- **Docker가 떠 있어야 한다.** 안 떠 있으면 통합 테스트가 무더기로 실패한다 — 빌드 실패를 코드 결함으로 오진하기 쉬우니 먼저 확인할 것.

### 단언
**AssertJ**를 쓴다 (35개 파일). `assertThat` · `assertThatThrownBy` 정적 임포트. JUnit 5.

### 고정값은 상수로
`Instant.parse("2026-07-26T12:00:00Z")` 같은 값은 테스트 상단에 `private static final`로 올린다. 리터럴을 여러 곳에 흩뿌리지 않는다.

## 7. Lombok — 쓰는 것만 쓴다

실측 사용: `@RequiredArgsConstructor`(19) · `@Slf4j`(13) · `@Getter`(11) · `@NoArgsConstructor`(6, JPA 요구).

**`@Data`·`@Builder`·`@Setter`는 쓰지 않는다.** 도메인 불변식을 팩토리 메서드로 지키는데 setter·builder가 그것을 우회하기 때문이다. (`@Value`가 코드에 보이면 Lombok이 아니라 **Spring의 프로퍼티 주입**이다.)

## 8. 설정

- 애플리케이션 고유 설정은 **`sift.*` 네임스페이스** 아래에 둔다 (`sift.collection.cron`, `sift.selection.window-hours`).
- 기본 프로파일은 `local`, 운영·CI는 `SPRING_PROFILES_ACTIVE`로 override.
- **설정값 옆에 이유를 주석으로 남긴다.** 지금 코드의 주석들이 그 예다 — 스케줄러 풀 크기가 2인 이유(트리거 2개가 한 스레드를 두고 줄 서면 발행이 밀린다), cron이 매시 10분인 이유(수집기가 몰리는 정각을 피함).

## 9. 하지 않는 것 (과설계 회피)

- **에러 코드 체계 없음.** `BusinessException`은 메시지만 받는다 — 필요해지는 시점에 확장한다.
- **DTO 매핑 라이브러리 없음.** 손으로 짠 static 매퍼.
- **목 라이브러리 없음.** Fake 직접 구현.
- **`@Id` 전략은 IDENTITY.** 대량 발송 배치 insert 성능 단계에서 SEQUENCE + allocationSize 전환을 검토한다(성능 로드맵 V2~).

이것들은 "안 해서 부족한 것"이 아니라 **필요가 실측되기 전에 만들지 않는다**는 하네스 원칙 1의 적용이다. 필요해지면 그때 DECISIONS에 결정으로 남기고 도입한다.
