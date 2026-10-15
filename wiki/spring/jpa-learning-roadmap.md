# JPA와 Spring Data JPA 학습 순서

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, Unknown; Spring Data JPA, Unknown
> Raw: [Hibernate 영속성 컨텍스트](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md); [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md); [JPA Query Language](../../raw/spring/jakarta-persistence-3-2-query-language.md); [Spring Data 저장](../../raw/spring/spring-data-jpa-entity-persistence-reference.md); [Spring Data 조회](../../raw/spring/spring-data-jpa-query-methods-reference.md); [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md)
> Updated: 2026-10-15

## Overview

이 경로는 **기존 JPA 기초 문서 → 객체의 관리와 저장 원리 → 관계의 생명주기와 조회 → Spring Data JPA 활용** 순서로 읽도록 구성했다. 공식 문서의 목차를 그대로 번역한 것이 아니라, 원문에 있는 개념을 선수 지식에 맞춰 배치한 학습 안내다. 이미 Wiki에 있는 주제는 새로 만들지 않고 아래 경로에 연결했다.

## 먼저 기존 Wiki에서 읽을 내용

| 순서 | 기존 문서 | 읽을 부분 |
| --- | --- | --- |
| 1 | [Spring DB 접근 기술 비교](spring-database-access-technologies.md) | JPA·Hibernate·Spring Data JPA의 관계, Kotlin JPA 설정, Repository CRUD 입문 |
| 2 | [엔티티·필드·연관 컬렉션과 DB 구조](jpa-entity-association-collection-mapping.md) | 객체와 테이블·column·외래 키 대응, 연관관계의 주인 |
| 3 | [Primary Key, Foreign Key와 복합 키](../database/primary-foreign-and-composite-keys.md) | 키와 식별자, JPA의 ID 매핑 |

Kotlin 문법이 낯설면 [클래스](../kotlin/kotlin-classes-and-data-classes.md), [Null Safety](../kotlin/kotlin-null-safety.md), [상속과 인터페이스](../kotlin/kotlin-inheritance-interfaces-and-delegation.md)를 먼저 복습한다. “주 생성자”와 “인자 없는 생성자”는 다른 개념이다. JPA 예제에 등장하는 생성자는 어떤 역할인지 함께 확인한다.

## 새로 보완한 문서의 읽기 순서

아래 순서는 학습을 위한 편집 구성이다. 각 문서는 원문을 Raw에 연결하고, 공식 동작을 설명하기 위해 만든 Kotlin 코드를 학습용 예제로 구분했다. 코드의 패키지·import와 Spring Boot 프로젝트 전체 설정은 필요한 부분 외에는 생략했다. 연관 엔티티가 새로 등장하는 일부 예제는 매핑 또는 함수의 일부이며, 모든 코드 블록을 하나의 파일에 붙이는 방식은 아니다.

| 순서 | 문서 | 확인할 질문 |
| --- | --- | --- |
| 1 | [JPA 영속성 컨텍스트와 엔티티 생명주기](jpa-persistence-context-and-entity-lifecycle.md) | 객체를 만들기만 한 것과 JPA가 관리하는 것은 어떻게 다른가? |
| 2 | [JPA 변경 감지와 flush·commit의 차이](jpa-dirty-checking-and-flush.md) | 값을 바꾸기만 했는데 UPDATE가 나가는 이유는 무엇인가? |
| 3 | [JPA 값 타입과 @Embeddable·@Embedded](jpa-embeddable-value-types.md) | 주소는 왜 별도 엔티티 대신 값으로 묶을 수 있는가? |
| 4 | [JPA cascade와 orphanRemoval](jpa-cascade-and-orphan-removal.md) | 부모 저장·삭제와 관계에서 빠진 자식 삭제는 어떻게 다른가? |
| 5 | [JPA JPQL 조인·집계·DTO 조회](jpa-jpql-joins-and-aggregation.md) | 테이블 대신 엔티티를 조인하고 집계하려면 어떻게 쓰는가? |
| 6 | [JPA N+1과 fetch join·Entity Graph](jpa-fetch-plans-and-n-plus-one.md) | 목록 조회 뒤 추가 SQL이 반복되는 이유와 해결 수단은 무엇인가? |
| 7 | [Spring Data JPA save와 새 엔티티 판별](spring-data-jpa-save-and-is-new.md) | save는 언제 persist와 merge를 선택하는가? |
| 8 | [Spring Data JPA 조회 반환 타입과 페이징](spring-data-jpa-query-results-and-pagination.md) | 단건·목록·Page·Slice는 어떤 반환 계약인가? |
| 9 | [Spring Data JPA @Query·native query·@Modifying](spring-data-jpa-declared-and-modifying-queries.md) | 직접 쓴 조회와 bulk 변경은 관리 객체에 어떤 영향을 주는가? |
| 10 | [Spring Data JPA Projection과 Kotlin DTO](spring-data-jpa-projections-and-dtos.md) | 화면에서 필요한 값만 어떻게 반환하는가? |
| 11 | [Spring Data JPA Service 트랜잭션 경계와 readOnly](spring-data-jpa-service-transactions.md) | 어떤 업무 범위를 함께 commit하거나 rollback해야 하는가? |
| 12 | [Spring Data JPA 동적 검색: Specification과 Query by Example](spring-data-jpa-dynamic-search.md) | 선택적 검색 조건은 어떻게 조합하는가? |
| 13 | [Spring Data JPA 사용자 정의 Repository와 fragment](spring-data-jpa-custom-repository-fragments.md) | 기본 Repository에 직접 구현한 기능을 어떻게 합치는가? |
| 14 | [Spring Data JPA Auditing으로 생성·수정 정보 기록](spring-data-jpa-auditing.md) | 생성·수정 정보는 어떤 설정과 동작 경로로 기록되는가? |

## 기존 문서로 이어지는 심화 주제

- 값 타입을 여러 개 저장하기: [ElementCollection](jpa-element-collection.md)
- 일대일과 이력형 관계의 차이: [1:1 매핑과 지연 로딩](jpa-to-one-lazy-loading-and-unique-foreign-key.md)
- 동시 수정 처리: [낙관적 락과 비관적 락](../database/optimistic-vs-pessimistic-locking.md)
- 실행 SQL 관찰: [DB 접근 기술 비교의 JPA SQL 로그 설정](spring-database-access-technologies.md#jpa)
- 고정 조회 조건: [SQLRestriction](hibernate-sqlrestriction.md)
- DB 계산 열: [GeneratedColumn](hibernate-generated-column.md)

## 원문을 읽는 기준

JPA 표준 규칙은 Jakarta Persistence, Hibernate에서의 구체적 동작은 Hibernate User Guide, Repository 편의 기능은 Spring Data JPA 문서를 근거로 한다. Hibernate의 확장 기능을 JPA 공통 규칙으로 확대하지 않는다. 각 문서의 Raw 링크에서 실제 근거를 확인한다.

수집한 Hibernate 자료는 원문에 표시된 버전 기준이며, Spring Data 문서는 버전이 바뀔 수 있는 reference URL이다. 실제 프로젝트에 적용할 때는 Spring Boot가 관리하는 Hibernate·Spring Data 버전에 맞춰 읽는다. 예를 들어 동적 검색 문서의 최신 API와 오래전부터 있던 Specification을 구분한다. 이 안내의 학습 순서는 특정 최신 버전으로 업그레이드하라는 뜻이 아니다.

## 연습 방법

각 문서의 직접 확인 연습은 전용 테스트 DB에서 진행한다. 엔티티 객체의 값과 생성 SQL을 함께 확인하고, 필요한 경우 flush·clear 후 다시 조회한다. 문서의 Kotlin 예제는 학습용으로 작성했으며, 이 Wiki 저장소에는 Spring Boot 실행 프로젝트가 없어 실제 DB 통합 실행을 검증한 코드는 아니다.

## See Also

- [스프링 입문 학습 로드맵](spring-learning-roadmap.md)
- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
- [첫 문서: 영속성 컨텍스트와 생명주기](jpa-persistence-context-and-entity-lifecycle.md)
