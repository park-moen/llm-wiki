# JPA N+1과 fetch join·Entity Graph

> Sources: Hibernate ORM, Unknown; Spring Data JPA, Unknown; Jakarta Persistence, Unknown
> Raw: [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md); [Spring Data JPA 조회 기능](../../raw/spring/spring-data-jpa-query-methods-reference.md); [JPA fetch join 규칙](../../raw/spring/jakarta-persistence-3-2-query-language.md)
> Updated: 2026-10-15

## Overview

목록 조회 한 번 뒤에 각 결과의 연관 객체를 읽기 위한 추가 조회가 반복되면 N+1 문제가 생길 수 있다. **필요한 연관 데이터를 어떤 조회에서 함께 가져올지**를 설계해야 한다. SQL 로그 설정은 기존 [DB 접근 기술 비교의 JPA 절](spring-database-access-technologies.md#jpa)에 연결하고, 여기서는 fetch 계획을 다룬다. [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

## LAZY만 붙이면 해결되는가

LAZY는 연관 데이터가 필요할 때 가져오는 방향의 설정이다. 목록을 받은 뒤 모든 항목의 연관 프로퍼티를 읽으면 추가 조회가 반복될 수 있다. EAGER도 항상 한 번의 join SQL로 조회한다는 뜻이 아니므로 추가 select를 만들 수 있다. 정적 매핑만으로 모든 화면의 조회량을 해결하려 하지 않는다. [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

예를 들어 `Purchase` 목록을 조회한 다음 반복문에서 `purchase.member.name`을 읽으면, 초기화되지 않은 회원을 추가 조회할 수 있다. 정확한 SQL 수는 동일 회원 공유, 이미 로딩된 객체, cache와 batch 설정 등에 따라 달라진다. [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

## fetch join으로 함께 조회하기

다음은 `Purchase.member`가 다대일 관계인 학습용 Repository 메서드다.

```kotlin
@Query("select p from Purchase p join fetch p.member")
fun findAllWithMember(): List<Purchase>
```

조회 결과인 `Purchase`와 연관 회원을 함께 가져온다. 연관 회원이 없어도 구매 행을 유지해야 한다면 `left join fetch`가 필요한지 검토한다. 필요한 연관 경로를 명시하는 것이 핵심이며, 모든 관계를 한꺼번에 fetch하는 방식은 피한다. [JPA fetch join 규칙](../../raw/spring/jakarta-persistence-3-2-query-language.md); [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

## Entity Graph로 fetch 계획 전달하기

```kotlin
@EntityGraph(attributePaths = ["member"])
fun findByStatus(status: String): List<Purchase>
```

`Purchase.status`가 문자열 프로퍼티라는 가정이다. `@EntityGraph`는 메서드의 조회에 필요한 연관 경로를 지정한다. JPQL 문자열에 fetch join을 직접 적는 대신 fetch 계획을 전달할 수 있지만, 이것만으로 모든 DB·매핑에서 동일한 SQL 형태를 보장하는 것은 아니다. [Spring Data JPA 조회 기능](../../raw/spring/spring-data-jpa-query-methods-reference.md); [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

## 컬렉션과 페이징을 함께 쓸 때

부모와 컬렉션을 join하면 부모 행이 자식 수만큼 늘어날 수 있다. 컬렉션 fetch join에 페이지 제한을 적용한 결과는 JPA 명세상 정의되지 않는다. 따라서 이를 일반적인 페이징 방법으로 삼지 않는다. 부모 ID를 먼저 페이지로 조회한 뒤 필요한 연관 데이터를 추가 조회하는 방법은 이 제약에서 도출한 설계 대안이다. [JPA fetch join 규칙](../../raw/spring/jakarta-persistence-3-2-query-language.md)

Hibernate batch fetching은 여러 지연 로딩 대상을 묶어 가져와 왕복을 줄이는 방법이다. fetch join은 조회 자체에서 함께 가져오고, batch는 추가 조회를 묶는 방식이므로 목적과 결과를 구분한다. [Hibernate Fetching](../../raw/spring/hibernate-7-1-fetching-strategies.md)

## 직접 확인하는 연습

데이터를 준비한 뒤 `flush()`·`clear()`하여 미리 로딩된 객체가 결과를 가리지 않게 한다. 일반 조회 후 회원 이름을 읽는 경우와 fetch join을 사용하는 경우의 SQL을 비교한다. 같은 회원을 공유하는 구매와 서로 다른 회원의 구매를 각각 넣어 추가 조회 수가 달라지는지도 확인한다.

## See Also

- 이전: [JPQL 조인과 집계](jpa-jpql-joins-and-aggregation.md)
- 다음: [save와 새 엔티티 판별](spring-data-jpa-save-and-is-new.md)
- [1:1 지연 로딩의 별도 제약](jpa-to-one-lazy-loading-and-unique-foreign-key.md)
