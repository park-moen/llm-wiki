# Knowledge Base Index

## intellij

IntelliJ IDEA의 탐색·설정·action 실행을 빠르게 찾는 방법과 shortcut을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [IntelliJ macOS 핵심 단축키](intellij/search-everywhere.md) | macOS 기본 Keymap의 Search Everywhere, file 탐색, import 추가·정리와 JPA entity DDL, format, refactoring과 Git 작업 shortcut | 2026-09-22 |

## database

관계형 database의 정규화, schema 제약, data integrity 규칙과 동시성 제어를 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Database Unique Constraint](database/unique-constraints.md) | `UNIQUE`의 선언·복합 key·primary key와의 차이·DBMS별 `NULL` 처리와 조건부 고유성 문서 연결 | 2026-10-01 |
| [부분 유니크 인덱스와 MariaDB의 조건부 고유성 구현](database/partial-unique-indexes-and-mariadb-alternatives.md) | PostgreSQL 부분 유니크 인덱스의 사용법·선택 시점·한계와 MariaDB 생성 열 대안 | 2026-10-01 |
| [낙관적 잠금 (Optimistic Locking)](database/optimistic-locking.md) | 버전 검사를 통한 충돌 감지와 실패 처리, DDL과의 관계, Hibernate `@Version` 및 버전 열 없는 방식 | 2026-10-02 |
| [낙관적 락과 비관적 락: 충돌 처리와 선택 기준](database/optimistic-vs-pessimistic-locking.md) | JPA 락 방식의 시점·동작·실패 처리 비교와 Spring Data JPA 적용 및 선택 기준 | 2026-10-02 |
| [갱신 손실 (Lost Update): 낙관적 잠금이 막는 동시 수정](database/lost-update-optimistic-locking.md) | 동시 편집·오래 열린 화면·카운터·서로 다른 필드의 덮어쓰기 사례와 `@Version` 충돌 검사 | 2026-10-02 |
| [관계형 데이터베이스 정규화 원칙](database/database-normalization-principles.md) | 함수 종속과 중복 데이터, 정규형, 의도적인 비정규화의 판단 기준 | 2026-09-28 |
| [거래 시점 스냅샷과 비정규화](database/transaction-snapshot-and-denormalization.md) | 현재 값의 중복 저장과 주문 당시 확정된 거래 사실을 구분하는 방법 | 2026-09-28 |
| [DB 설계에서 다형 참조](database/polymorphic-references.md) | 종류·ID 쌍의 편의와 외래 키 무결성 한계, 종류별 테이블과 고정 외래 키 대안 | 2026-09-28 |
| [외래 키의 `ON DELETE CASCADE`와 `ON DELETE SET NULL`](database/foreign-key-on-delete-actions.md) | 참조 행 삭제·외래 키 NULL 처리의 차이와 관계별 선택 기준 | 2026-09-29 |
| [Primary Key, Foreign Key와 복합 키](database/primary-foreign-and-composite-keys.md) | PK·FK·복합 키의 DB 규칙과 Kotlin/JPA `@IdClass`·`@EmbeddedId` 매핑 비교 | 2026-09-29 |
| [MySQL EXPLAIN의 인덱스와 Using filesort 읽기](database/mysql-explain-index-and-filesort.md) | 실행 계획의 `key`·`rows`·`Extra`, 복합 인덱스로 `WHERE`와 `ORDER BY`를 함께 처리하는 조건과 추가 정렬 예시 | 2026-09-29 |
| [DB 콘텐츠 다국어화 저장 구조 선택](database/multilingual-content-schema-design.md) | 언어별 컬럼·번역 테이블·JSON 저장의 적용 조건과 조회·폴백·재검토 설계 기준 | 2026-09-30 |

## seo

검색 노출과 크롤링·색인 생성에 필요한 URL·콘텐츠 신호를 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Google Search에서 canonical URL을 일관되게 지정하는 방법](seo/canonical-url-consolidation.md) | 중복 URL의 canonical 신호를 통일하고 Search Console의 canonical 관련 색인 제외를 점검하는 절차 | 2026-09-08 |
| [Next.js App Router에서 canonical과 페이지별 metadata 구현](seo/nextjs-app-router-canonical-and-page-metadata.md) | middleware로 pathname을 전달해 canonical을 만들고 페이지별 metadata·사이트맵·production 응답을 검증하는 실무 절차 | 2026-09-08 |

## spring

Spring 기반 웹 애플리케이션 개발의 학습 순서, 핵심 원리와 실무 기술을 정리한다.

강의 기반 문서는 입문·기본편 순서로, 웹에서 수집한 자료를 중심으로 정리한 문서는 보완 학습·실무 주제로 나눴습니다. 각 문서의 Sources·Raw를 기준으로 `근거`를 표시했으며, 강의에 공식 문서 설명을 보탠 경우에는 `강의 + 공식 문서`로 표시합니다. `공식 문서`에는 공식 API 문서·설정 예제·구현 소스도 포함합니다. 원본 링크는 각 문서의 Raw에서 확인할 수 있습니다.

### 스프링 입문 강의

강의 흐름에 따라 프로젝트 생성 → 웹 개발 기초 → 회원 관리 → Bean 설정 → 웹 MVC → DB 접근 → AOP 순서로 읽습니다. 학습 로드맵과 Bean 설정 문서에는 기본편으로 이어지는 내용도 포함되어 있습니다.

| Article | Summary | 근거 | Updated |
|---------|---------|------|---------|
| [스프링 입문 학습 로드맵](spring/spring-learning-roadmap.md) | 입문 실습에서 기본편의 객체 지향 설계·container·Bean 관리로 이어지는 학습 경로 | 강의 | 2026-10-11 |
| [Spring Boot 프로젝트 생성과 첫 실행](spring/spring-boot-project-setup.md) | Spring Initializr 프로젝트 생성, 기본 구조, 버전 지침과 내장 Tomcat 실행 확인 | 강의 | 2026-08-12 |
| [Spring Boot Starter와 의존성 구조](spring/spring-boot-starter-dependencies.md) | Starter의 전이 의존성, 주요 library와 IntelliJ에서 dependency tree를 확인하는 방법 | 강의 | 2026-08-12 |
| [Spring Boot Gradle 빌드와 JAR 실행](spring/spring-boot-gradle-build-and-run.md) | JAR 파일의 뜻부터 Gradle `jar`·`bootJar`의 차이, 실행 JAR 배포와 `bootRun`까지 설명 | 강의 + 공식 문서 | 2026-10-04 |
| [Spring MVC View 렌더링과 Thymeleaf](spring/spring-mvc-view-rendering.md) | MVC 책임 분리, `@RequestParam`, Thymeleaf 렌더링과 IntelliJ Parameter Info | 강의 | 2026-08-12 |
| [Spring MVC API 응답과 HttpMessageConverter](spring/spring-mvc-api-response.md) | `@ResponseBody`, JavaBean property, Jackson converter와 IntelliJ 구문 완성 단축키 | 강의 | 2026-08-12 |
| [Spring 회원 관리 백엔드와 테스트](spring/spring-member-backend-and-testing.md) | Repository·Service 구현, Given–When–Then과 예외 테스트, DI 및 IntelliJ 단축키 | 강의 | 2026-10-11 |
| [Spring Bean과 의존관계 설정](spring/spring-beans-and-dependency-injection.md) | Bean 등록·DI 입문과 AppConfig, container·scan·자동 주입·singleton 상세 연결 | 강의 | 2026-10-11 |
| [Spring 회원 관리 웹 MVC](spring/spring-member-web-mvc.md) | Form binding, Thymeleaf 목록·property 접근, memory 생명주기와 IntelliJ 단축키 | 강의 | 2026-08-13 |
| [Spring DB 접근 기술 비교](spring/spring-database-access-technologies.md) | H2부터 JdbcTemplate·JPA·Spring Data JPA까지의 전환, Kotlin 보조 예제와 DB 통합 테스트 | 강의 + 공식 문서 | 2026-08-22 |
| [Spring AOP와 공통 관심사 분리](spring/spring-aop-cross-cutting-concerns.md) | 직접 시간 측정의 문제와 Aspect 등록·pointcut·proxy·DI로 공통 관심사를 분리하는 원리 | 강의 | 2026-08-22 |

### 스프링 핵심 원리 기본편 강의

객체 지향 설계 → 회원·주문 예제 → 할인 정책 확장 → AppConfig → container와 Bean 관리 순서로 읽습니다.

| Article | Summary | 근거 | Updated |
|---------|---------|------|---------|
| [스프링 핵심 원리 기본편 학습 안내](spring/spring-core-learning-roadmap.md) | 기본편의 학습 순서, 입문편과의 차이, 자료 버전과 첫 단원 슬라이드 연결 | 강의 | 2026-10-11 |
| [객체 지향 설계와 Spring의 SOLID 원칙](spring/spring-object-oriented-design-and-solid.md) | Spring의 목적, 역할·구현 분리, SOLID와 인터페이스 도입 비용 | 강의 | 2026-10-11 |
| [기본편 회원·주문 도메인 설계](spring/spring-core-member-order-design.md) | 기본편의 회원·주문 API, 할인 위임과 초기 구현 의존성의 한계 | 강의 | 2026-10-11 |
| [할인 정책 확장과 단위 테스트](spring/discount-policy-extension-and-testing.md) | 정률 할인 테스트, 주문 서비스의 DIP·OCP 문제와 AppConfig로 정책 교체 | 강의 | 2026-10-11 |
| [AppConfig와 IoC·DI](spring/spring-appconfig-ioc-di.md) | AppConfig로 생성·연결 책임 분리, SRP·DIP·OCP와 Spring 전환 | 강의 | 2026-10-11 |
| [Spring container와 Bean 조회](spring/spring-container-bean-lookup.md) | ApplicationContext 생성, 타입·상속별 Bean 조회와 BeanDefinition | 강의 | 2026-10-11 |
| [Spring singleton과 Configuration](spring/spring-singleton-and-configuration.md) | 공유 상태의 위험, singleton registry와 @Configuration의 메서드 호출 보장 | 강의 | 2026-10-11 |
| [Spring component scan의 범위와 충돌](spring/spring-component-scan.md) | 탐색 범위, stereotype, 필터와 자동·수동 등록 충돌 | 강의 | 2026-10-11 |
| [Spring 자동 주입과 Bean 선택](spring/spring-autowiring-and-bean-selection.md) | 생성자 주입, 선택적 의존성, Qualifier·Primary와 List·Map 전략 선택 | 강의 | 2026-10-11 |
| [Spring Bean 초기화와 종료 콜백](spring/spring-bean-lifecycle.md) | 의존관계 주입 후 초기화, 종료 콜백과 외부 라이브러리 설정 | 강의 | 2026-10-11 |
| [Spring Bean scope와 Provider](spring/spring-bean-scopes-and-provider.md) | singleton·prototype·웹 scope, Provider와 scoped proxy의 지연 조회 | 강의 | 2026-10-11 |

### 공식 문서 기반 보완 학습

아래 문서는 웹에서 수집한 공식 문서를 바탕으로 구성한 보완 학습 자료입니다. 강의에서 직접 다룬 범위와 구분해서 읽습니다.

#### JPA와 Spring Data JPA

입문 강의 목록의 [Spring DB 접근 기술 비교](spring/spring-database-access-technologies.md)를 읽은 뒤, 아래 학습 안내와 기초 문서부터 시작해 저장 원리, 조회, Spring Data JPA 활용으로 이어집니다. 각 글의 직접 작성·확인 연습을 풀어 보세요. Auditing 다음은 필요할 때 읽는 심화 문서입니다.

키와 식별자가 낯설면 영속성 컨텍스트 문서에 들어가기 전에 [Primary Key, Foreign Key와 복합 키](database/primary-foreign-and-composite-keys.md)를 먼저 읽으세요. 동시 수정 처리는 이 경로를 마친 뒤 [낙관적 락과 비관적 락](database/optimistic-vs-pessimistic-locking.md)으로 이어집니다.

| Article | Summary | 근거 | Updated |
|---------|---------|------|---------|
| [JPA와 Spring Data JPA 학습 순서](spring/jpa-learning-roadmap.md) | 기존 기초 문서와 신규 심화 문서를 연결한 읽기 순서, 원문 구분과 Kotlin·SQL 실습 안내 | 공식 문서 | 2026-10-15 |
| [JPA 엔티티·필드·연관 컬렉션과 DB 구조의 대응](spring/jpa-entity-association-collection-mapping.md) | 엔티티·객체·필드·연관 컬렉션을 DB 테이블·행·열·외래 키와 연결해 설명 | 공식 문서 | 2026-10-02 |
| [JPA 영속성 컨텍스트와 엔티티 생명주기](spring/jpa-persistence-context-and-entity-lifecycle.md) | EntityManager, 관리·준영속 상태와 persist·find·detach·merge의 차이; Kotlin 실습 포함 | 공식 문서 | 2026-10-15 |
| [JPA 변경 감지와 flush·commit의 차이](spring/jpa-dirty-checking-and-flush.md) | 관리 객체 변경의 SQL 반영, AUTO flush와 IDENTITY 예외, flush·clear·commit 구분 | 공식 문서 | 2026-10-15 |
| [JPA 값 타입과 @Embeddable·@Embedded](spring/jpa-embeddable-value-types.md) | 값 타입 정의·사용 위치, 소유 엔티티와의 관계, AttributeOverride와 Kotlin 예제 | 공식 문서 | 2026-10-15 |
| [JPA cascade와 orphanRemoval](spring/jpa-cascade-and-orphan-removal.md) | 엔티티 작업 전파와 관계에서 빠진 자식 삭제의 차이, DB ON DELETE CASCADE와 구분 | 공식 문서 | 2026-10-15 |
| [JPA JPQL 조인·집계·DTO 조회](spring/jpa-jpql-joins-and-aggregation.md) | 엔티티 기준 경로 탐색과 조인, GROUP BY·HAVING, 생성자 표현식과 bulk 변경 주의점 | 공식 문서 | 2026-10-15 |
| [JPA N+1과 fetch join·Entity Graph](spring/jpa-fetch-plans-and-n-plus-one.md) | 목록 조회의 추가 SQL 원인, fetch join·EntityGraph·batch 선택과 컬렉션 페이징 주의점 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA save와 새 엔티티 판별](spring/spring-data-jpa-save-and-is-new.md) | save의 persist·merge 분기, nullable Version·ID 검사와 직접 할당 ID의 Persistable 적용 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA 조회 반환 타입과 페이징](spring/spring-data-jpa-query-results-and-pagination.md) | 단건·목록·nullable 반환 계약, Page·Slice·List와 count 비용, Kotlin 조회 예제 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA @Query·native query·@Modifying](spring/spring-data-jpa-declared-and-modifying-queries.md) | 명시적 JPQL과 매개변수 바인딩, native countQuery, bulk 변경 후 flush·clear와 callback 차이 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA Projection과 Kotlin DTO](spring/spring-data-jpa-projections-and-dtos.md) | interface·DTO projection의 차이, JPQL 생성자 표현식과 native mapping·중첩 조회 주의점 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA Service 트랜잭션 경계와 readOnly](spring/spring-data-jpa-service-transactions.md) | 작업 단위와 Repository 기본 설정, readOnly의 한계, proxy 내부 호출과 rollback 규칙 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA 동적 검색: Specification과 Query by Example](spring/spring-data-jpa-dynamic-search.md) | 조건 조합을 위한 Specification과 probe 기반 QBE의 용도·제약, Kotlin 예제 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA 사용자 정의 Repository와 fragment](spring/spring-data-jpa-custom-repository-fragments.md) | 사용자 정의 인터페이스·Impl 구현·Repository 합성, 이름 규칙과 트랜잭션 경계 | 공식 문서 | 2026-10-15 |
| [Spring Data JPA Auditing으로 생성·수정 정보 기록](spring/spring-data-jpa-auditing.md) | CreatedDate·LastModifiedDate, EntityListener·EnableJpaAuditing 설정과 작업자 기록·bulk 변경 한계 | 공식 문서 | 2026-10-15 |
| [JPA `@ElementCollection`](spring/jpa-element-collection.md) | 기본 타입·embeddable 컬렉션의 사용 상황과 예시, `targetClass`·`fetch` 설정 | 공식 문서 | 2026-09-29 |
| [JPA 1:1 매핑 선택과 지연 로딩](spring/jpa-to-one-lazy-loading-and-unique-foreign-key.md) | 전체 고유 1:1과 이력형 다대일을 구분하고, 역방향 지연 로딩과 조건부 고유 인덱스 선택 기준을 설명 | 공식 문서 | 2026-10-01 |

### 실무 주제와 도구

프로젝트 메모와 웹 자료를 함께 정리한 문서, SQL 로그 도구와 Hibernate 기능별 참고 문서입니다. 시드 데이터 문서는 사용자 메모·공식 문서·기술 글을 함께 사용하므로 `사용자 메모 + 웹 자료`로 표시합니다.

| Article | Summary | 근거 | Updated |
|---------|---------|------|---------|
| [시드 데이터 초기화와 병렬 개발](spring/seed-data-initialization-and-parallel-development.md) | 시드 데이터·초기화 코드·fixture를 구분하고 Spring profile로 병렬 개발용 데이터를 격리하는 방법 | 사용자 메모 + 웹 자료 | 2026-09-17 |
| [P6Spy SQL 로그 읽기와 설정](spring/p6spy-sql-logging.md) | SQL·바인딩 값·JDBC 호출 시간·category 읽기, 출력 설정·필터와 N+1 조사 시 주의점 | 공식 문서 | 2026-10-13 |
| [Hibernate `@SQLRestriction`: 고정 조회 조건의 동작과 한계](spring/hibernate-sqlrestriction.md) | 엔티티·연관 컬렉션의 차이, 고정 SQL 조건의 조합 방식과 `@Filter`·`@SoftDelete`와의 차이 | 공식 문서 | 2026-10-02 |
| [Hibernate @GeneratedColumn과 DB 생성 열](spring/hibernate-generated-column.md) | 생성 열을 쓰는 상황과 계산식 예시, 값 재조회 및 관련 주석의 차이 | 공식 문서 | 2026-09-29 |

## statistics

데이터 분포를 읽는 데 필요한 기초 통계 개념과 해석 기준을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Percentile과 Percentage의 차이](statistics/percentiles-and-percentages.md) | percentage의 비율과 percentile의 정렬된 데이터 경계값을 구분하고 p95 해석과 계산 방식 차이를 설명 | 2026-09-21 |

## docker

Docker image·container·storage·network와 Docker Compose 기반 다중 container 애플리케이션 운영을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Docker 핵심 개념](docker/docker-fundamentals.md) | Docker client-daemon 구조와 image·container의 기본 실행 모델 | 2026-08-10 |
| [Docker Compose](docker/docker-compose.md) | Compose application model, service lifecycle와 production 운영 기준 | 2026-08-10 |
| [Docker 보안과 운영 주의점](docker/docker-security-and-operations.md) | Docker daemon 권한, port·secret과 production 운영 시 주의사항 | 2026-08-10 |

## kotlin

Kotlin/JVM의 문법에서 시작해 타입·객체·함수의 내부 원리와 JVM 실행, 비동기·동시성으로 이어지는 학습 경로입니다.

아래 여섯 단계의 순서대로 읽고 각 문서의 연습을 실행해 보세요. 기존 문법 문서는 유지하고 관련 내부 동작을 보강했으며, 독립적인 주제는 새 문서로 연결했습니다. 예제는 별도 파일에서 실행하는 학습용 코드이며 제품 코드의 권장 설정을 뜻하지 않습니다.

언어가 보장하는 규칙, compiler가 생성한 표현, JVM의 실행 중 최적화를 구분합니다. 수집한 명세·설계 노트와 과거 JetBrains 글은 버전 범위가 다르므로 각 문서의 Raw와 버전 안내를 함께 확인하세요.

### 1. 실행과 기본 문법

처음에는 컴파일 문서의 앞부분만 읽고, 스코프·타입·분기·함수·null 처리로 진행합니다. 컴파일러 내부 구조는 뒤에서 다시 읽어도 됩니다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin 컴파일과 실행: 소스에서 JVM까지](kotlin/kotlin-compilation-and-execution.md) | 소스·bytecode·JVM의 관계, K2의 분석·IR·코드 생성, 디버깅과 증분 컴파일의 기초 | 2026-10-15 |
| [Kotlin 스코프와 이름 해석](kotlin/kotlin-scopes-and-name-resolution.md) | 선언 스코프·문장 스코프, 이름 가림과 초기화 순서, JavaScript 호이스팅과의 구분 | 2026-10-15 |
| [Kotlin 기본 타입과 타입 추론](kotlin/kotlin-basic-types.md) | 기본 타입과 정적 타입 추론, Any·Unit·Nothing의 구분과 수치 표현 연결; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 숫자의 JVM 표현과 boxing](kotlin/kotlin-numeric-representation-and-boxing.md) | Int·Int?·제네릭 숫자의 표현, 값 비교와 참조 비교, 정수 연산과 부동소수점 주의점 | 2026-10-15 |
| [Kotlin 제어 흐름](kotlin/kotlin-control-flow.md) | 분기·반복 문법과 표현식 타입·단축 평가·실패 경로; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 함수](kotlin/kotlin-functions.md) | 함수 선언·기본값·Unit과 인자 평가 순서·overload·Java 호출 구분; 기존 연습 포함 | 2026-10-15 |
| [Kotlin Null Safety](kotlin/kotlin-null-safety.md) | nullable·안전 호출·smart cast의 안정성과 platform type 경계; 기존 연습 포함 | 2026-10-15 |

### 2. 객체와 상태

클래스와 생성자를 익힌 뒤 초기화·프로퍼티·참조를 연결합니다. 클래스 문서의 data class·copy 절은 동등성 문서를 읽은 후 다시 확인하세요.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin 클래스와 `data class`](kotlin/kotlin-classes-and-data-classes.md) | 클래스·주 생성자·init 구분, data class 생성 함수와 얕은 복사; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 생성자와 객체 초기화 순서](kotlin/kotlin-object-initialization-order.md) | 부모·주 생성자·프로퍼티·init·보조 생성자의 순서, 초기화 중 open 호출과 object 초기화 | 2026-10-15 |
| [Kotlin 프로퍼티와 접근·위임](kotlin/kotlin-properties.md) | getter·setter·field와 위임 변환, lazy·lateinit 초기화 전략; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 객체 참조·동등성·불변성](kotlin/kotlin-object-references-and-equality.md) | 객체 공유와 참조 값 전달, val과 불변성의 차이, equals·hashCode와 얕은 복사 | 2026-10-15 |
| [Kotlin Objects](kotlin/kotlin-objects.md) | object·companion object·object expression과 초기화·공유 상태 구분; 기존 연습 포함 | 2026-10-15 |
| [Kotlin factory 함수와 객체 생성 규칙](kotlin/kotlin-factory-functions-and-construction-rules.md) | factory와 생성자의 차이, 결제 원장 부호 예시, 직접 생성 우회와 JPA 제약; 직접 작성하는 연습 포함 | 2026-10-12 |

### 3. 타입과 함수의 원리

상속과 제네릭을 바탕으로 호출 대상 선택, receiver, closure와 inline을 배웁니다. 제네릭의 List 예제는 원소 타입 관계를 보기 위한 것으로, 컬렉션 연산은 다음 단계에서 다룹니다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin 상속, 인터페이스와 위임](kotlin/kotlin-inheritance-interfaces-and-delegation.md) | open·override·interface·by와 초기화 중 호출, JVM default 구현의 버전 차이; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 제네릭·variance·타입 소거](kotlin/kotlin-generics-and-type-erasure.md) | in·out의 읽기·쓰기 제약, star projection과 실행 시 타입 정보, reified의 한계 | 2026-10-15 |
| [Kotlin 확장 함수](kotlin/kotlin-extension-functions.md) | receiver와 확장 문법, 정적 선택·멤버 우선·여러 receiver 구분; 기존 연습 포함 | 2026-10-15 |
| [Kotlin 고차 함수와 람다](kotlin/kotlin-higher-order-functions-and-lambdas.md) | 함수 타입·람다·closure와 mutable 캡처, JVM 람다 생성 방식; 기존 연습 포함 | 2026-10-15 |
| [Kotlin Scope Functions](kotlin/kotlin-scope-functions.md) | let·apply·run·also·with의 객체 접근 방식, 반환값과 선택 기준; 직접 작성하는 연습 포함 | 2026-10-12 |
| [Kotlin inline·noinline·crossinline·reified](kotlin/kotlin-inline-and-reified.md) | 호출 지점 확장과 코드 크기, 비지역 반환, 저장할 람다와 reified 타입 검사 | 2026-10-15 |
| [Kotlin 특수 클래스](kotlin/kotlin-special-classes.md) | sealed·enum·value class의 선택 기준과 boxing·타입 별칭·생성 코드 구분; 기존 연습 포함 | 2026-10-15 |

### 4. 데이터 처리

참조 공유와 read-only의 차이를 바탕으로 컬렉션을 다루고, Sequence에서 실제 평가 시점·순서·비용을 비교합니다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin 컬렉션](kotlin/kotlin-collections.md) | List·Set·Map, read-only·불변·참조 공유와 구현별 순서·동등성; 기존 연습 포함 | 2026-10-15 |
| [Kotlin Sequence와 지연 평가](kotlin/kotlin-sequences-and-lazy-evaluation.md) | Iterable과 Sequence의 연산 순서, terminal 연산·중간 컬렉션·작은 작업의 추가 비용 | 2026-10-15 |

### 5. JVM과 실무 연동

메모리·예외·Java 연동을 읽은 뒤 라이브러리의 실행 의존성과 바이너리 계약을 확인합니다. 첫 컴파일 문서의 IR·증분 컴파일 절도 이 단계에서 다시 읽습니다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin/JVM 메모리·호출 frame·JIT](kotlin/kotlin-jvm-memory-and-execution.md) | JVM의 stack·heap·method area, 클래스 로딩·초기화와 HotSpot 최적화의 구분 | 2026-10-15 |
| [Kotlin 예외 전파와 자원 정리](kotlin/kotlin-exceptions-and-resource-management.md) | try 표현식·catch 선택·finally·use, Unit·Nothing과 일반 예외·취소의 구분 | 2026-10-15 |
| [Kotlin Java 연동·reflection·JVM 선언](kotlin/kotlin-java-interop-and-reflection.md) | platform type, top-level·프로퍼티·기본 인자의 Java 표현, KClass·Class·reflection과 버전 의존성 | 2026-10-15 |
| [Kotlin 라이브러리와 API 사용](kotlin/kotlin-libraries-and-apis.md) | import·의존성·API 사용과 reflection 실행 의존성·바이너리 계약; 기존 연습 포함 | 2026-10-15 |

### 6. 비동기와 동시성

coroutine의 사용 흐름과 suspend 변환을 연결한 뒤 실행 문맥, 취소·실패, Flow와 공유 상태를 배웁니다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Kotlin coroutine 기초와 suspend의 내부 동작](kotlin/kotlin-coroutines-and-suspension.md) | 중단과 thread blocking, Continuation·CPS·상태 머신, 언어 기능과 kotlinx.coroutines의 역할 | 2026-10-15 |
| [Kotlin CoroutineContext와 dispatcher](kotlin/kotlin-coroutine-context-and-dispatchers.md) | Job·CoroutineName·dispatcher의 역할, 문맥 상속·withContext와 thread 전환·blocking 경계 | 2026-10-15 |
| [Kotlin 구조화된 동시성·취소·예외 전파](kotlin/kotlin-structured-concurrency-and-cancellation.md) | scope·Job 부모 자식 관계, 협력적 취소와 finally, launch·async·supervisor의 예외 처리 | 2026-10-15 |
| [Kotlin Flow·StateFlow의 실행과 상태 전달](kotlin/kotlin-flow-and-stateflow.md) | cold flow의 수집과 실행, flowOn·buffer, StateFlow의 최신 상태·동등성 기반 병합과 원자적 update | 2026-10-15 |
| [Kotlin/JVM 공유 상태·가시성·원자성](kotlin/kotlin-shared-state-and-memory-visibility.md) | JVM 메모리 모델과 happens-before, Volatile의 한계, atomic·Mutex·상태 격리 선택 | 2026-10-15 |

## git

Git 저장소의 작업 공간, 태그와 GitLab 배포 흐름을 효율적으로 관리하는 방법을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Git Worktree](git/git-worktree.md) | 하나의 저장소를 공유하는 다중 작업 디렉터리의 개념, 제약과 주요 명령어 | 2026-08-10 |
| [Git 태그와 GitLab 태그 기반 배포](git/git-tags-and-gitlab-tag-deployment.md) | Git tag 종류, tag pipeline, protected tag와 버전 규칙을 연결한 배포 학습 가이드 | 2026-08-10 |

## ux

개발자가 사용자 경험, 인터페이스 사용성과 직군 간 협업을 학습하는 근거와 도서를 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [개발자를 위한 한국어 UX 도서 근거 가이드](ux/developer-ux-book-evidence-guide.md) | 실제 개발자·디자이너·기획자 후기로 비교한 UX 도서별 강점, 번역과 한계 | 2026-08-21 |
| [개발자를 위한 한국어 UX 도서 추천](ux/developer-ux-book-recommendations-2026-08-21.md) | [Archived] 화면 설계 원칙, 사용성, 국내 프로젝트와 협업 목적에 따른 독서 순서 | 2026-08-21 |
| [디자이너 없는 팀을 위한 AI 디자인 레퍼런스 가이드](ux/non-designer-ai-design-reference-guide-2026-09-09.md) | [Archived] 실제 제품 흐름·디자인 시스템·접근성 자료를 Claude 디자인과 연결하는 개발자용 참고 체계 | 2026-09-09 |
| [비디자이너를 위한 디자인 시안 가이드 작성법](ux/ieve-design-reference-selection-guide-2026-09-09.md) | [Archived] 사용자 문제와 레퍼런스를 Claude가 실행할 수 있는 시안 지침과 검증 기준으로 바꾸는 방법 | 2026-09-09 |
| [비디자이너를 위한 디자인 레퍼런스 도구함](ux/non-designer-design-reference-toolkit-2026-09-09.md) | [Archived] 실제 사이트·국내 사례·디자인 시스템·학습 자료와 asset을 목적별로 고르는 실전 목록 | 2026-09-09 |
| [일본·중국 디자인 레퍼런스 플랫폼 활용법](ux/japan-china-design-reference-platforms-2026-09-09.md) | [Archived] 일본의 실제 사이트 gallery와 중국의 design community를 CJK 조판·interaction·moodboard 목적에 맞게 활용하는 방법 | 2026-09-09 |

## ai-agents

AI agent의 병렬 작업, harness 선택, 기계적 gate와 brownfield 개인 운영 지침을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Claude Code Background Bash·Monitor와 Codex의 대응 기능](ai-agents/claude-code-background-bash-monitor-and-codex.md) | 장기 명령 실행과 이벤트 감시의 차이, Claude의 수명·제한과 Codex background terminal·관리 명령 비교 | 2026-10-17 |
| [Stagehand: Playwright와의 차이와 시작 방법](ai-agents/stagehand-vs-playwright.md) | Playwright 대비 API·속도 비교 조건, SDK 사용법과 AI agent의 개발 화면 검증 시뮬레이션 | 2026-09-25 |
| [Plan mode와 지속적인 이해 형성](ai-agents/plan-mode-as-iterative-understanding.md) | 긴 계획서 중심 흐름의 한계와 작업 중 이해·시도·확인·조정을 반복하는 접근 | 2026-09-28 |
| [AI Agent Teams와 Git Worktree](ai-agents/agent-teams-and-git-worktrees.md) | 병렬 agent의 작업 조정과 파일 격리를 구분하고 Worktree 도입 기준을 설명 | 2026-08-16 |
| [Agent Harness의 구조와 Deterministic Control Loop](ai-agents/agent-harness-anatomy-and-deterministic-control-loop.md) | Model 주변의 tool·context·guardrail·trace·verify·retry를 연결해 거짓 완료를 차단하는 구조 | 2026-08-16 |
| [Superpowers의 Agent 관리와 Spec-Driven 개발 Workflow](ai-agents/superpowers-agent-management-and-spec-driven-development.md) | 사람의 intent를 spec·작은 TDD task·역할 분리 review·behavior proof로 연결하는 agent 관리 방법론 | 2026-08-16 |
| [AI Agent 산문 게이트와 결정적 게이트](ai-agents/ai-agent-prose-vs-deterministic-gates.md) | [Personal Guide] Agent 규칙의 산문 준수와 기계적 차단을 구분하는 판단 기준 | 2026-08-16 |
| [AI Coding Autonomy Experiment와 Human-in-the-loop](ai-agents/ai-coding-autonomy-experiment.md) | Spring Boot coding agent 실험의 자율성 전략, 실패 패턴과 인간 검증의 역할 | 2026-08-16 |
| [Claude Code Hooks](ai-agents/claude-code-hooks.md) | Claude Code hook의 생명주기, Skill과의 차이, 입출력·차단·보안·loop guard | 2026-08-10 |
| [Claude Code ELI5 Skill로 쉬운 시각 설명 만들기](ai-agents/claude-code-eli5-skill.md) | 초보자 눈높이와 HTML 시각 형식을 고정하는 ELI5 Skill의 원리, 사용법과 검증 경계 | 2026-09-15 |
| [AI Harness 실측과 선택 가이드](ai-agents/ai-harness-audit-and-selection.md) | [Personal Guide] 작업 유형과 현재 설치 상태에 맞춰 AI harness를 평가·선택하는 기준 | 2026-08-16 |
| [Brownfield AI Agent Workflow](ai-agents/brownfield-ai-agent-workflow.md) | [Personal Guide] Code를 정본으로 삼아 범위를 제한하고 안전망부터 만드는 개인 workflow 제안 | 2026-08-11 |
| [AI Agent Teams에서 Git Worktree를 사용할지 판단하기](ai-agents/agent-teams-git-worktree-recommendation.md) | [Archived] 병렬 agent 작업에서 Worktree를 적용할 조건과 최소 운영 방식 | 2026-08-10 |
| [Superpowers 기반 Brownfield 연습 워크플로 초기 설계](ai-agents/superpowers-brownfield-practice-workflow-initial-design.md) | [Archived] Claude에서 Skill을 명시 호출하며 Issue·TDD·Storybook·design system 흐름을 검증하기 위한 초기 실험안 | 2026-08-11 |
| [Superpowers Brownfield 실전 가이드](ai-agents/superpowers-brownfield-field-guide.md) | 실제 Issue 실행과 원 설계의 intent를 비교한 Skill 흐름, local branch 통합과 Worktree 운영 기준 | 2026-08-16 |
| [Superpowers를 지속 사용하는 Harness 운영 전략](ai-agents/superpowers-continuous-use-harness-strategy.md) | [Archived] Superpowers workflow를 재사용하면서 프로젝트별 verification·gate·human checkpoint를 분리하는 운영 지침 | 2026-08-16 |
| [Superpowers 기반 Accelerator Harness Engineering 설계](ai-agents/superpowers-accelerator-harness-engineering-design-2026-09-16.md) | [Archived] Superpowers workflow에 Task Contract·Teach-back·Recovery Drill·deterministic completion gate를 결합하는 개인 harness 설계 | 2026-09-16 |
| [im-not-ai Humanize Korean Skill 사용 가이드](ai-agents/im-not-ai-humanize-korean-skill-guide.md) | Codex·Claude Code에서 한국어 AI 문체를 진단·윤문하는 사용법과 보존 규칙 | 2026-08-11 |
| [gstack으로 AI 개발 Workflow 이해하기](ai-agents/gstack-ai-engineering-workflow.md) | 역할별 Skill과 browser·검증 도구를 연결한 gstack의 구조, 주니어용 사용 순서와 한계 | 2026-09-08 |
| [Matt Pocock Skills의 Repository 설정 방식](ai-agents/matt-pocock-skills-repository-setup.md) | Issue tracker·triage label·domain 문서를 공통 설정으로 만들어 engineering Skill에 연결하는 방법 | 2026-09-16 |
| [find-skills로 Agent Skill 탐색과 설치하기](ai-agents/find-skills-discovery-and-installation.md) | skills.sh와 Skills CLI에서 후보를 찾고 source·품질·검색 한계를 검증한 뒤 project 또는 global 범위에 설치하는 방법 | 2026-09-16 |
| [Vercel Agent Skills의 구조와 활용 범위](ai-agents/vercel-agent-skills-structure-and-scope.md) | React·UI·문서·Vercel 운영 Skill의 구성, 설치·배포 구조와 권한·검증 경계 | 2026-09-16 |
| [Vercel Agent Skills·Superpowers·gstack·Harness Engineering 비교](ai-agents/vercel-agent-skills-superpowers-gstack-harness-comparison-2026-09-16.md) | [Archived] 기술별 전문 지식, 개발 workflow, 역할별 실행 도구와 deterministic control의 차이와 조합 방법 | 2026-09-16 |
| [개인 AI Engineering Harness 주말 구축 시뮬레이션](ai-agents/personal-ai-engineering-harness-weekend-simulation-2026-09-15.md) | [Archived] Blueprint를 정본으로 두고 Superpowers·gstack·Matt Pocock Skills와 개인 hook을 선택적으로 연결하는 최소 실험안 | 2026-09-15 |
| [교체 가능한 Personal Blueprint Harness 설계](ai-agents/replaceable-personal-blueprint-harness-architecture-2026-09-15.md) | [Archived] 사람이 읽기 쉬운 추적성, capability adapter, shadow mode와 단계별 hook으로 구성한 개인 기획 실험 환경 | 2026-09-15 |
| [Personal Blueprint 변경 세트와 정합성 종료 Gate 설계](ai-agents/personal-blueprint-change-set-consistency-gate-2026-09-15.md) | [Archived] 수정 중에는 변경을 누적하고 변경 종료 시 전체 문서 동기화·정합성 검사와 검증 상태를 확정하는 방법 | 2026-09-15 |
| [AI Agent 지침과 개인 지식 자산 운영 원칙](ai-agents/ai-agent-instructions-judgment-and-personal-knowledge-assets-2026-09-14.md) | [Archived] 공식 문서 기반 지침, 사용자 판단 기준, Codex memory와 개인 Git 지식 자산을 연결한 운영 가이드 | 2026-09-14 |
| [AI 개발에서 Markdown 편집과 Git Diff 검토 워크플로우](ai-agents/markdown-editing-and-git-diff-review-workflow-2026-09-21.md) | [Archived] AI가 수정한 Markdown을 읽기 중심 도구와 Git diff 검토 도구의 역할 분리로 확인하고 hunk 단위로 반영하는 운영 제안 | 2026-09-21 |

## software-career

AI 시대의 개발자 역할, 신입 성장, 경력과 직업 시장에 관한 판단과 근거를 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [해외 개발자 컨퍼런스와 녹화본 탐색 가이드](software-career/developer-conferences-and-recordings.md) | KotlinConf·IntelliJ IDEA Conf·Spring I/O를 포함한 해외 행사와 무료·등록·유료 녹화본 탐색 | 2026-10-21 |
| [AI 활용·개발 workflow 컨퍼런스 가이드](software-career/ai-development-conferences-and-recordings.md) | AI Native DevCon·AI Engineer와 Kotlin·Spring AI 세션의 선택, 공개 영상과 검증 관찰 기준 | 2026-10-21 |
| [Jira 작업 항목의 분할 크기와 관리 비용](software-career/work-item-decomposition-and-tracking-granularity.md) | 작은 구현 단계와 팀이 추적할 작업 단위를 구분하고 페이지 수준에서 분할을 시작하는 개인 실무 지침 | 2026-08-18 |
| [Software Engineering을 Time·Scale·Trade-off로 이해하기](software-career/software-engineering-time-scale-tradeoffs.md) | 장기 software의 변경 가능성, Hyrum’s Law, 조직 확장성, shift left와 evidence 기반 판단 원칙 | 2026-08-16 |
| [효과적인 Software Design Document 작성법](software-career/effective-software-design-document.md) | 위험과 변경 비용에 따라 설계 문서의 범위·구조·운영 항목·미결정 사항을 작성하는 방법 | 2026-09-15 |
| [AS-IS와 TO-BE GAP 분석](software-career/as-is-to-be-gap-analysis.md) | 현재 업무 evidence와 목표 조건을 process·role·data·rule 관점에서 비교하고 전환 전략으로 연결하는 방법 | 2026-09-21 |
| [AI Coding에서 Software Fundamentals가 더 중요해지는 이유](software-career/ai-coding-software-fundamentals.md) | AI 시대의 design concept, ubiquitous language, TDD, deep module과 사람의 전략적 설계 책임 | 2026-09-16 |
| [AI Coding에서 Code Reading과 Intent 보존](software-career/ai-coding-code-reading-and-intent-preservation.md) | Accelerator와 vibecoder의 유지보수 계약, code reading의 역할과 intent debt 보존 방법 | 2026-09-16 |
| [LLM과 함께 프로그래밍의 즐거움과 주도권 지키기](software-career/llm-assisted-programming-enjoyment-and-ownership.md) | 직접 코딩할 부분을 남기고 계획·조사·검토에 LLM을 활용하는 작업 방식과 다른 개발자들의 경험 | 2026-09-28 |
| [Vibecoder에서 Accelerator로 전환하는 실천 가이드](software-career/vibecoder-to-accelerator-transition-guide-2026-09-16.md) | [Archived] 이해하지 못한 AI 변경의 크기를 줄이고 조사·plan·작은 구현·diff review·직접 복구로 전환하는 방법 | 2026-09-16 |
| [AI를 활용한 개발자 성장과 Career 판단](software-career/ai-assisted-engineering-growth-and-career-judgment.md) | AI를 동료처럼 활용하는 학습, role-based context, fundamentals, 협업·product·migration 책임을 연결 | 2026-08-16 |
| [Claude Code 팀의 AI-Native 개발 Workflow와 개발자 역량](software-career/claude-code-team-ai-native-development-workflow.md) | 학습·생산 mode 전환, plan 기반 병렬 agent와 그 반론, 다층 검증과 AI 시대의 개발자 역량 | 2026-09-28 |
| [최신 현업 개발 자료와 AI 네이티브 소프트웨어 엔지니어링](software-career/current-engineering-sources-and-ai-native-development.md) | 한국 빅테크 최신 공식 자료와 production AI 개발의 구현·검증·운영 구조를 연결한 학습 지도 | 2026-08-16 |
| [한국어로 읽는 해외 기술 정보 채널 지도](software-career/korean-tech-translation-and-curation-channels.md) | Frontend·Backend·Cloud·Product·Design 직군별 번역·요약·큐레이션 채널과 조합 방법 | 2026-08-27 |
| [Martin Fowler의 AI 시대 소프트웨어 엔지니어링 학습 가이드](software-career/martin-fowler-ai-software-engineering-study-guide.md) | AI 검증 원칙부터 mentor, source 평가, Technology Radar와 pattern까지 주니어 관점에서 풀어낸 인터뷰 학습서 | 2026-08-16 |
| [해외 현업 중심 AI 네이티브 엔지니어링 학습 로드맵](software-career/overseas-ai-native-engineering-learning-path.md) | [Archived] Fowler·Thoughtworks·Stripe·Uber를 먼저 학습하고 검증 역량을 갖춘 뒤 한국 사례로 확장하는 순차 학습 과정 | 2026-08-16 |
| [현업 개발자의 AI 활용과 최신 기술 콘텐츠 학습 가이드](software-career/production-engineering-youtube-guide-2026-08-16.md) | [Archived] 최신 한국 기업 자료를 포함해 무검증 vibe coding과 AI-native software engineering을 구분한 추천 답변 | 2026-08-16 |
| [AI 시대 신입 개발자의 생존과 성장 판단](software-career/ai-era-junior-developer-survival-assessment-2026-08-13.md) | [Archived] AI 위임 중심 환경에서 신입 개발자가 겪는 성장 위험과 대응 원칙에 대한 시점 고정 판단 | 2026-08-13 |
| [AI 시대의 Software Engineering 학습 전략](software-career/ai-era-software-engineering-learning-strategy-2026-08-23.md) | [Archived] 문법·API 암기에서 실행 모델, 설계 판단, 검증과 복구 중심으로 이동하는 개발자 학습 전략 | 2026-08-23 |
| [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](software-career/ai-native-fullstack-learning-roadmap-2026-08-23.md) | [Archived] JS·React·Next.js와 Kotlin·Spring Boot 실무를 AI 기반 기능 소유, 검증과 복구 역량으로 연결하는 6개월 로드맵 | 2026-08-23 |
| [풀스택 개발자를 위한 SQL·DB·AWS 자격증 선택](software-career/fullstack-developer-certification-path-2026-09-29.md) | [Archived] 전문대·비전공 개발자의 SQLD·SQLP·Oracle·AWS·정보처리 자격증 선택과 유효 기간 정리 | 2026-09-29 |
| [취업 초기 주니어를 위한 AI-Native 개발 프로세스](software-career/junior-ai-native-development-process.md) | [Archived] 이해 가능한 작은 변경, TDD, advisory subagent와 단계적 Worktree 학습을 연결한 개인 업무 방법론 | 2026-08-16 |
| [취업 초기 주니어를 위한 AI-Native 개발 프로세스 2: Harness Engineering](software-career/junior-ai-native-development-harness-engineering.md) | [Archived] 1편의 원칙을 Superpowers Skill, human checkpoint, deterministic verification과 단계적 Worktree 운영으로 실행하는 방법 | 2026-08-16 |
| [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](software-career/frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md) | [Archived] Spring의 OCP·DIP를 TypeScript 경계, React 합성·주입과 과잉 추상화 방지 기준으로 번역한 실무 가이드 | 2026-08-24 |
| [Frontend OCP·DIP와 React 설계 패러다임 통합 가이드](software-career/frontend-ocp-dip-and-react-software-engineering-guide-2026-08-24.md) | [Archived] OOP 중심 관점과 React의 함수형·선언형·합성 모델을 연결하고 FE에서 OCP·DIP를 적용하는 통합 가이드 | 2026-08-24 |
| [요구사항 분석부터 설계 다이어그램까지: Spring·React 통합 학습 로드맵](software-career/requirements-modeling-and-design-diagrams-learning-roadmap-2026-08-24.md) | [Archived] 요구사항·Domain Modeling·UML·C4·ERD·API와 React의 User Flow·상태·Data Flow 설계를 연결한 학습 로드맵 | 2026-08-24 |

## frontend

프런트엔드 애플리케이션의 구조, UI 경계와 아키텍처 커뮤니케이션 방법을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [프런트엔드 아키텍처를 위한 C4 모델](frontend/c4-model-for-frontend-architecture.md) | C4의 네 가지 확대 수준을 프런트엔드의 모듈과 UI 분해에 맞춰 적용하는 방법 | 2026-08-24 |
| [ChatGPT 웹의 성능 중심 아키텍처](frontend/chatgpt-web-performance-architecture.md) | 익명 사용자의 첫 입력과 응답을 빠르게 만드는 SSR·점진적 로딩·기능 플래그·보호 계층 설계 | 2026-09-08 |
| [Orval로 OpenAPI 클라이언트 생성하기](frontend/orval-openapi-client-generation.md) | TypeScript API 클라이언트·React Query hook·MSW mock 생성의 목적과 설정, 명세 검증 및 출력 제약 | 2026-10-13 |
| [openapi.yaml로 HTTP API 명세 읽고 작성하기](frontend/openapi-yaml.md) | YAML API 계약의 주요 필드·요청과 응답 예제·Swagger UI와 Orval·명세와 구현 검증의 구분 | 2026-10-13 |
| [next-intl로 Next.js 다국어 화면 만들기](frontend/next-intl.md) | locale·messages, App Router 설정, Server·Client Components, 언어별 URL과 번역 누락 처리 | 2026-10-13 |
| [HTTP 307 redirect와 rewrite의 네트워크 흐름](frontend/redirect-vs-rewrite.md) | 브라우저 재요청·서버 내부 매핑·주소창·메서드 보존과 미지원 언어 URL 처리의 선택 기준 | 2026-10-13 |

## llm-wiki

질문, 원본 수집, 지식 통합과 답변 보관을 연결하는 LLM Wiki 운영 방법을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Karpathy LLM Wiki 운영 워크플로우](llm-wiki/karpathy-llm-wiki-workflow.md) | raw·일반 wiki·Archive의 역할과 질문부터 지식 축적까지의 운영 흐름 | 2026-08-10 |
| [LLM Wiki를 질문·Ingest·Archive 허브로 사용하는 방법](llm-wiki/llm-wiki-question-ingest-archive-guide.md) | [Archived] 대화, 자료 ingest와 유용한 답변 보관을 연결하는 실용 가이드 | 2026-08-10 |

## orca

Worktree-native AI IDE인 Orca의 핵심 모델, 에이전트·리뷰·원격 실행·자동화와 운영 방법을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [이전 4레인 작업을 Orca Orchestration으로 실행하는 방법](orca/orca-orchestration-practical-guide.md) | [Archived] TNSP-1233 병렬 작업을 Run·Task·Dispatch·worker_done 기반 supervised orchestration으로 전환하는 실전 절차 | 2026-08-20 |
| [Orca CLI Handoff, Orchestration과 Claude Agent Teams](orca/orca-cli-handoff-orchestration-and-agent-teams.md) | 실제 4레인 병렬 개발을 바탕으로 완료 계약 없는 Worktree handoff와 worker_done 기반 orchestration, Agent Teams를 구분 | 2026-08-20 |
| [Orca 핵심 모델과 첫 멀티에이전트 세션](orca/orca-foundations-and-worktree-model.md) | Worktree 격리, 병렬 agent 비교, pane·상태·session restore를 연결한 Orca 기본 운용 모델 | 2026-08-12 |
| [Orca 에이전트와 세션 운영](orca/orca-agent-runtime-and-session-management.md) | Agent launch·권한·account·resume·hibernation·usage와 attention 관리 | 2026-08-12 |
| [Orca 리뷰·편집·브라우저 작업 흐름](orca/orca-review-editing-and-browser-workflow.md) | AI diff annotation부터 browser 검증, commit·push·hosted review까지의 폐쇄 루프 | 2026-08-12 |
| [Orca 원격 실행과 모바일 운영](orca/orca-remote-and-mobile-runtime.md) | Local·SSH·Remote Orca Server·per-workspace environment의 선택과 host ownership | 2026-08-12 |
| [Orca CLI·자동화·오케스트레이션](orca/orca-cli-automation-and-orchestration.md) | Selector·JSON 기반 CLI, scheduled automation과 추적 가능한 multi-agent coordination | 2026-08-12 |
| [Orca 설정·개인정보·문제 해결](orca/orca-operations-settings-privacy-troubleshooting.md) | Permission·public exposure·telemetry 설정과 host·Git·provider 중심 진단법 | 2026-08-12 |

## web-security

웹 애플리케이션의 브라우저 보안 정책, 공격 완화와 운영 검증 방법을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [Content Security Policy (CSP)](web-security/content-security-policy-csp.md) | 리소스 출처 제한, 정책 지시문, Report-Only 기반 점진적 배포와 위반 보고 해석 | 2026-08-24 |

## projects

익명화한 프로젝트의 목표와 구현 검토 내용을 보관한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [프로젝트의 AI 챗봇 MVP 출시 검토](projects/project-ai-mvp-discussion-2026-10-17.md) | [Archived] 대화 기반 기록: 확정된 AI 기능 출시 KPI와 검토 중인 사주 챗봇·RAG·비용 구분 | 2026-10-17 |

## ai-engineering

AI 모델을 서비스에 연동하는 구조와 RAG의 자료 준비·검색·비용·평가 원칙을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [RAG의 원리와 기존 모델 API 활용](ai-engineering/rag-fundamentals.md) | 검색·자료 전달·답변 생성의 역할과 API·embedding·추가 학습의 차이 | 2026-10-17 |
| [RAG 구축·비용·답변 범위와 평가](ai-engineering/rag-design-cost-and-evaluation.md) | 자료 분할·표준 RAG 구조, 비용 절감 조건과 추가 비용, 응답 제한 및 평가 방법 | 2026-10-17 |

## if(kakao)-26

if(kakao) 발표에서 다룬 개념을 이해하기 위한 외부 원문과 보충 설명을 정리한다.

| Article | Summary | Updated |
|---------|---------|---------|
| [실행 경험을 레슨 스킬로 축적하는 Agent 운영](if(kakao)-26/learning-from-experience-with-agent-skills.md) | Anthropic·Letta·ACE로 이해하는 실행·회고·지식 축적과 사람의 검토·역할별 재사용의 구분 | 2026-10-20 |
