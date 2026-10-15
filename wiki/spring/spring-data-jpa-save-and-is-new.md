# Spring Data JPA save와 새 엔티티 판별

> Sources: Spring Data JPA, Unknown; Hibernate ORM, Unknown
> Raw: [Spring Data Persisting Entities](../../raw/spring/spring-data-jpa-entity-persistence-reference.md); [Hibernate Merge](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md)
> Updated: 2026-10-15

## Overview

`save()`는 메서드 이름만 보고 INSERT 전용이라고 해석하면 안 된다. Spring Data JPA는 엔티티가 새 객체인지 판별한 뒤 새 객체이면 `persist()`, 아니면 `merge()`를 사용한다. **이 판별은 DB에 매번 존재 여부를 조회하는 것과 다르다.** 기본 CRUD 선언은 기존 [Spring Data JPA 입문](spring-database-access-technologies.md)에 있다. [Persisting Entities](../../raw/spring/spring-data-jpa-entity-persistence-reference.md)

## 기본 판별 순서

| 조건 | 새 객체 판별 |
| --- | --- |
| primitive가 아닌 `@Version` 프로퍼티 존재 | version이 null이면 새 객체 |
| 위 조건에 해당하는 version 없음 | ID가 null이면 새 객체 |
| `Persistable` 구현 | `isNew()` 결과에 위임 |

따라서 ID의 유무만으로 persist·merge 분기를 판단하면 nullable version을 놓친다. primitive version은 초기 값과 저장 후 값을 구별하는 기준으로 사용할 수 없으므로 같은 null 판별에 사용되지 않는다. [Persisting Entities](../../raw/spring/spring-data-jpa-entity-persistence-reference.md)

## 반환값을 사용하는 이유

```kotlin
val saved = memberRepository.save(member)
```

새 엔티티의 persist와 준영속 엔티티의 merge는 같은 동작이 아니다. 특히 merge는 전달한 상태를 관리 인스턴스에 복사하므로, 계속 작업할 때는 반환값을 사용한다. `save()`를 호출했다는 사실만으로 SQL이 그 줄에서 모두 실행되거나 commit됐다고 판단하지 않는다. [Persisting Entities](../../raw/spring/spring-data-jpa-entity-persistence-reference.md); [Hibernate Merge](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md)

## 직접 ID를 지정하는 엔티티

ID를 애플리케이션에서 미리 할당하고 nullable version도 없다면 기본 ID 검사만으로 새 객체를 구분하기 어렵다. 이 경우 공식 문서는 `Persistable`과 생명주기 callback으로 새 객체 여부를 관리하는 패턴을 제시한다. 아래는 Kotlin으로 옮긴 학습 예제다.

```kotlin
@Entity
open class ImportedMember protected constructor() : Persistable<String> {
    @field:Id
    private var externalId: String = ""

    @field:Transient
    private var newEntity: Boolean = true

    constructor(externalId: String) : this() {
        this.externalId = externalId
    }

    override fun getId(): String = externalId
    override fun isNew(): Boolean = newEntity

    @PostPersist
    @PostLoad
    protected fun markNotNew() {
        newEntity = false
    }
}
```

`Persistable`은 `org.springframework.data.domain.Persistable`, `Transient`는 `jakarta.persistence.Transient`를 사용한다. 새 객체의 flag가 저장 후 또는 DB에서 로딩한 뒤 false로 바뀐다. 이것은 판별을 명시하는 패턴이지, 이미 DB에 같은 ID가 있을 때 충돌을 없애는 기능은 아니다. [Persisting Entities](../../raw/spring/spring-data-jpa-entity-persistence-reference.md)

## 직접 확인하는 연습

자동 생성 ID의 `Member`와 직접 할당 ID의 엔티티를 비교한다. `save()` 전후 ID·`isNew()`·반환 객체를 살펴보고, 실제 SQL은 트랜잭션 종료까지 관찰한다. 생성자 호출 여부만으로 저장 경로를 판단하지 않는다.

## See Also

- 이전: [N+1과 fetch 계획](jpa-fetch-plans-and-n-plus-one.md)
- 다음: [조회 반환 타입과 페이징](spring-data-jpa-query-results-and-pagination.md)
- [merge와 관리 상태](jpa-persistence-context-and-entity-lifecycle.md)
