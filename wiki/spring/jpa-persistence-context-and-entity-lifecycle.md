# JPA 영속성 컨텍스트와 엔티티 생명주기

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, Unknown
> Raw: [Hibernate 엔티티 생명주기](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md); [JPA 생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md); [JPA 영속성 컨텍스트 정의](../../raw/spring/jpa-persistence-context-definitions.md)
> Updated: 2026-10-15

## Overview

영속성 컨텍스트는 **엔티티 객체를 관리하는 작업 공간**이다. 같은 영속성 컨텍스트 안에서는 같은 영속 식별자를 가진 엔티티가 하나의 관리 인스턴스에 대응한다. `EntityManager`는 이 작업 공간을 통해 저장·조회·삭제를 요청하는 API다. 엔티티를 단순한 Kotlin 객체와 구분하려면 먼저 **지금 JPA가 이 객체를 관리하고 있는가**를 확인해야 한다. [JPA 영속성 컨텍스트 정의](../../raw/spring/jpa-persistence-context-definitions.md)

## 객체와 DB 사이에 관리 상태가 있다

| 상태 | 의미 | 대표적인 진입 경로 |
| --- | --- | --- |
| 새 객체(new) | 아직 영속성 컨텍스트와 연결되지 않은 객체 | 생성자 호출 |
| 영속(managed) | 영속성 컨텍스트가 관리하는 객체 | `persist()` 또는 조회 |
| 준영속(detached) | 영속 식별자는 있지만 현재 컨텍스트가 관리하지 않는 객체 | `detach()`, `clear()`, 컨텍스트 종료 |
| 삭제(removed) | 삭제를 위해 관리되는 객체 | 관리 객체에 `remove()` 호출 |

`id` 값이 있다는 사실만으로 영속 상태라고 판단할 수 없다. 직접 ID를 지정한 새 객체도 있고, DB에 대응하는 행이 있어도 현재 컨텍스트에서는 준영속일 수 있다. `contains()`로 현재 객체의 관리 여부를 확인한다. [Hibernate 엔티티 생명주기](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md); [JPA 생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## Kotlin 실습에 사용할 공통 엔티티

이 학습 경로의 `Member` 예제는 아래 엔티티를 사용한다. 공식 문서의 API 동작을 보여 주기 위해 만든 **학습용 코드**이며, 실제 프로젝트의 패키지와 의존성은 생략했다. 뒤의 문서에서 등장하는 `active`, `email`, `name`은 이 프로퍼티를 가리킨다.

```kotlin
import jakarta.persistence.*

@Entity
@Table(name = "members")
open class Member protected constructor() {
    @field:Id
    @field:GeneratedValue(strategy = GenerationType.IDENTITY)
    open var id: Long? = null
        protected set

    @field:Column(nullable = false)
    open var name: String = ""

    @field:Column(nullable = false, unique = true)
    open var email: String = ""

    open var active: Boolean = true

    constructor(name: String, email: String) : this() {
        this.name = name
        this.email = email
    }
}
```

이 코드는 인자 없는 생성자와 `open`을 직접 적었다. plugin으로 생성하는 방법은 기존 [Kotlin JPA 설정 설명](spring-database-access-technologies.md)에 있다. 키와 column 매핑의 기초도 기존 [엔티티와 DB 구조](jpa-entity-association-collection-mapping.md), [키 매핑](../database/primary-foreign-and-composite-keys.md)을 먼저 읽는다.

## 같은 컨텍스트에서 조회하기

아래 함수 본문은 **쓰기 가능한 트랜잭션 안**에서 실행한다고 가정한다.

```kotlin
val member = Member("민수", "minsu@example.com")
entityManager.persist(member)

val id = requireNotNull(member.id)
val found = entityManager.find(Member::class.java, id)

check(entityManager.contains(member))
check(member === found)

entityManager.detach(member)
check(!entityManager.contains(member))
```

`persist()`는 객체를 관리 상태로 만든다. `find()`는 같은 컨텍스트에 해당 식별자의 객체가 있으면 그 객체를 반환한다. `detach()` 이후 객체를 수정해도 그 수정은 자동으로 DB에 반영되지 않는다. `clear()`는 특정 객체 대신 컨텍스트의 모든 관리 객체를 준영속 상태로 만든다. [JPA 생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## `merge()`는 전달한 객체를 그대로 관리하지 않는다

```kotlin
member.name = "수정한 이름"       // member는 위에서 detach한 객체
val managed = entityManager.merge(member)

check(entityManager.contains(managed))
check(!entityManager.contains(member))
```

`merge()`는 전달받은 객체의 상태를 관리 객체로 복사하고 그 관리 객체를 반환한다. 준영속 객체를 전달한 경우 원래 객체 자체가 다시 관리되는 것으로 이해하면 안 된다. 또한 수정할 필드만 골라 적용하는 API가 아니므로, DTO에서 빈 값으로 만든 엔티티 전체를 무심코 merge하지 않는다. 마지막 문장은 상태 복사 동작에서 도출한 사용 지침이다. [Hibernate 엔티티 생명주기](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md)

## 직접 확인하는 연습

같은 회원을 `find()`로 조회한 뒤 `contains()`와 참조 비교 `===`를 확인한다. 그다음 `flush()`로 변경을 반영한 후 `clear()`하고 다시 조회해 객체 참조가 어떻게 달라지는지 살펴본다. `clear()` 전에 flush하지 않은 변경은 유실될 수 있다. [JPA 생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## See Also

- [JPA 학습 순서](jpa-learning-roadmap.md)
- 다음: [변경 감지와 flush](jpa-dirty-checking-and-flush.md)
