# Spring Data JPA Projection과 Kotlin DTO

> Sources: Spring Data JPA, Unknown; Jakarta Persistence, Unknown
> Raw: [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md); [JPA 생성자 표현식](../../raw/spring/jakarta-persistence-3-2-query-language.md)
> Updated: 2026-10-15

## Overview

목록 화면에서 이름과 이메일만 필요하다면 엔티티 전체 대신 projection으로 필요한 모양을 반환할 수 있다. **조회 결과용 DTO는 엔티티와 역할이 다르다.** Kotlin의 `data class`는 이처럼 결과값을 담는 용도로 사용할 수 있다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md)

## interface projection

공식 Java getter 예제를 Kotlin interface로 옮긴 학습용 코드다.

```kotlin
interface MemberSummary {
    val name: String
    val email: String
}

interface MemberRepository : JpaRepository<Member, Long> {
    fun findByActive(active: Boolean): List<MemberSummary>
}
```

접근자가 엔티티 프로퍼티와 대응하면 closed projection으로 볼 수 있고, Spring Data가 필요한 속성을 파악해 조회를 최적화할 수 있다. 반환 객체는 원래 엔티티를 직접 수정하기 위한 인터페이스로 취급하지 않는다. SpEL로 계산하는 open projection은 필요한 속성을 미리 확정할 수 없어 같은 최적화를 기대하기 어렵다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md)

## 명시적인 Kotlin DTO 조회

```kotlin
package example

data class MemberSummaryDto(val name: String, val email: String)
```

```kotlin
@Query("select new example.MemberSummaryDto(m.name, m.email) from Member m")
fun findSummaries(): List<MemberSummaryDto>
```

DTO 생성자 인자의 타입과 순서가 조회값과 맞아야 한다. JPQL `new`에는 패키지를 포함한 클래스 이름을 쓴다. 이 DTO는 `@Entity`가 아니므로 단순 결과 DTO에 JPA용 인자 없는 생성자를 만들 필요가 없다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md); [JPA 생성자 표현식](../../raw/spring/jakarta-persistence-3-2-query-language.md)

Spring Data는 일부 쿼리를 DTO 생성자 표현식으로 재작성할 수 있다. 이미 `new` 표현식을 썼다면 그 재작성을 하지 않는다. 처음 학습할 때는 위처럼 생성자 표현식을 명시하여 어떤 값이 DTO에 들어가는지 보이게 하는 편을 권한다. 마지막 문장은 학습을 위한 선택이다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md)

## 중첩 projection과 native query 주의점

projection을 쓴다고 항상 적은 column만 읽는 것은 아니다. 중첩 프로퍼티가 join으로 이어지면 전체 중첩 프로퍼티가 materialize될 수 있으므로 생성 SQL을 확인한다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md)

native query의 DTO는 결과 column 순서·타입과 생성자가 직접 맞는 경우와 별도 변환이 필요한 경우를 구분한다. 필요한 경우 `@SqlResultSetMapping` 같은 명시적 매핑을 사용한다. JPQL 생성자 표현식에서 select 항목에 붙인 alias도 DTO 매핑에 그대로 사용할 수 있는 문법은 아니다. [Spring Data Projections](../../raw/spring/spring-data-jpa-projections-reference.md)

## 직접 작성하는 연습

회원 이름·이메일 목록을 엔티티 반환, interface projection, DTO projection으로 각각 조회한다. 반환 타입과 SQL select 목록을 비교한다. 화면에서 쓰지 않는 엔티티 필드가 DTO에 들어가지 않도록 작성하고, 같은 DTO를 `data class`로 썼다고 엔티티에도 그대로 적용할 수 있는지 구분해서 설명한다.

## See Also

- 이전: [직접 작성하는 쿼리](spring-data-jpa-declared-and-modifying-queries.md)
- 다음: [Service 트랜잭션 경계](spring-data-jpa-service-transactions.md)
- [Kotlin data class](../kotlin/kotlin-classes-and-data-classes.md)
