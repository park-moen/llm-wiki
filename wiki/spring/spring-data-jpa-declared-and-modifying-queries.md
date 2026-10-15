# Spring Data JPA @Query·native query·@Modifying

> Sources: Spring Data JPA, Unknown; Jakarta Persistence, Unknown
> Raw: [Spring Data JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md); [JPA bulk 변경 규칙](../../raw/spring/jakarta-persistence-3-2-query-language.md); [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md); [Modifying API](../../raw/spring/spring-data-jpa-modifying-javadoc.md)
> Updated: 2026-10-15

## Overview

메서드 이름이 지나치게 길어지거나 원하는 조회를 표현하기 어렵다면 `@Query`로 쿼리를 직접 작성할 수 있다. JPQL은 엔티티를 대상으로 하고 native query는 DB의 테이블·column을 대상으로 한다. **bulk 수정·삭제는 조회와 별도로 `@Modifying`과 트랜잭션을 확인해야 한다.** [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md)

## 이름 있는 매개변수로 바인딩하기

다음은 [공통 Member](jpa-persistence-context-and-entity-lifecycle.md)를 사용하는 학습 예제다. `@Param`을 명시하여 compiler의 매개변수 이름 보존 여부에 기대지 않는다.

```kotlin
@Query("select m from Member m where m.name = :name and m.active = true")
fun findActiveByName(@Param("name") name: String): List<Member>
```

이름 있는 매개변수는 함수 매개변수의 순서를 바꾸어도 쿼리의 이름과 연결할 수 있다. 조회값을 문자열에 직접 붙이지 않고 바인딩한다. JPQL에서 사용하는 이름은 `Member`, `name`, `active`이며 DB 테이블 이름이 아니다. [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md)

## native query와 countQuery

```kotlin
@Query(
    value = "select * from members where active = true",
    countQuery = "select count(*) from members where active = true",
    nativeQuery = true
)
fun findActiveNative(pageable: Pageable): Page<Member>
```

예제는 실제 테이블 이름을 `members`로 정한 경우다. boolean 표현과 DB dialect는 적용 환경에 맞춰 확인한다. 복잡한 native query에서는 자동 count 변환에 기대기보다 의도에 맞는 `countQuery`를 명시해야 할 수 있다. native SQL의 페이징·정렬 지원 범위도 사용하는 Spring Data 버전에서 확인한다. [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md)

## 일괄 수정은 관리 객체를 자동 갱신하지 않는다

```kotlin
@Modifying(flushAutomatically = true, clearAutomatically = true)
@Query("update Member m set m.active = false where m.name = :name")
fun deactivateByName(@Param("name") name: String): Int
```

이 메서드는 쓰기 가능한 Service 트랜잭션 안에서 호출한다. 반환값은 영향을 받은 행 수다. bulk 변경은 DB에 직접 적용되므로 이미 조회한 관리 객체에는 예전 값이 남을 수 있다. `clearAutomatically`는 실행 후 컨텍스트를 비우므로, 다른 객체 참조도 준영속 상태가 된다는 점을 함께 고려한다. [JPA bulk 변경 규칙](../../raw/spring/jakarta-persistence-3-2-query-language.md); [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md)

위 예제의 `flushAutomatically`는 clear 전에 대기 중인 변경을 잃지 않도록 먼저 flush하기 위한 선택이다. 이 설정을 모든 수정 쿼리에 기계적으로 붙이지 않는다. 대기 중인 다른 변경까지 먼저 반영되는 점과 clear 이후 작업을 함께 설계한다. `@Modifying` 자체가 트랜잭션을 시작하는 주석은 아니다. [Modifying API](../../raw/spring/spring-data-jpa-modifying-javadoc.md); [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md); [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md)

## 파생 삭제와 bulk 삭제의 차이

`deleteBy…`는 대상을 조회한 뒤 개별 삭제하여 생명주기 callback을 호출할 수 있다. `@Modifying @Query("delete …")`는 bulk 삭제이므로 로딩된 엔티티의 `@PreRemove` 같은 callback을 같은 방식으로 호출하지 않는다. 둘을 이름만 다른 동일 최적화로 취급하지 않는다. [JPA Query Methods](../../raw/spring/spring-data-jpa-query-methods-reference.md)

## 직접 확인하는 연습

같은 회원을 먼저 조회한 뒤 bulk 수정한다. clear를 하지 않은 경우 객체의 `active` 값과 DB 값을 비교하고, clear 후 재조회했을 때의 차이를 확인한다. callback이 필요한 삭제인지도 별도로 판단한다.

## See Also

- 이전: [조회 반환 타입과 페이징](spring-data-jpa-query-results-and-pagination.md)
- 다음: [Projection과 DTO](spring-data-jpa-projections-and-dtos.md)
- [영속성 컨텍스트](jpa-persistence-context-and-entity-lifecycle.md)
