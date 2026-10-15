# JPA 변경 감지와 flush·commit의 차이

> Sources: Hibernate ORM, Unknown; Spring Data JPA, Unknown
> Raw: [Hibernate 관리 상태 변경](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md); [Hibernate Flushing](../../raw/spring/hibernate-7-1-flushing.md); [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md)
> Updated: 2026-10-15

## Overview

관리 상태의 엔티티를 변경하면 Hibernate가 변경을 감지하고 flush 시점에 SQL로 반영한다. **flush는 객체의 변경을 DB에 동기화하는 것이고, commit은 트랜잭션을 완료하는 것**이다. SQL이 실행됐다는 사실과 트랜잭션이 확정됐다는 사실은 구분해야 한다. [Hibernate Flushing](../../raw/spring/hibernate-7-1-flushing.md)

## `save()` 없이 값이 바뀌는 이유

다음은 [공통 Member 엔티티](jpa-persistence-context-and-entity-lifecycle.md)를 트랜잭션 안에서 수정하는 학습용 예제다.

```kotlin
@Service
class MemberService(private val entityManager: EntityManager) {
    @Transactional
    fun rename(id: Long, name: String) {
        val member = entityManager.find(Member::class.java, id)
            ?: error("회원을 찾을 수 없습니다")
        member.name = name
    }
}
```

쓰기 가능한 트랜잭션에서 조회한 객체는 관리 상태다. 프로퍼티를 바꾸면 flush 시 그 변경이 감지되어 DB로 전달된다. JPA 관점에서는 관리 엔티티 수정에 별도의 `save()`가 필수는 아니다. 반면 준영속 객체나 DTO를 수정하는 것에는 이 규칙이 적용되지 않는다. [Hibernate 관리 상태 변경](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md); [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md)

## flush가 일어나는 시점

Hibernate의 기본 `AUTO` 모드에서는 commit 전, 아직 반영하지 않은 변경과 관련된 JPQL/HQL 조회 전 등에 flush가 발생할 수 있다. 따라서 SQL이 항상 함수 마지막에만 실행된다고 외우면 안 된다. native SQL은 사용하는 API와 동기화 설정에 따라서도 동작을 확인해야 한다. [Hibernate Flushing](../../raw/spring/hibernate-7-1-flushing.md)

`IDENTITY` 키 생성에서는 생성된 ID를 얻기 위해 `persist()` 직후 INSERT가 필요할 수 있다. 이 학습 경로의 `Member`가 그 전략을 사용하므로, 저장 SQL이 commit 전까지 무조건 미뤄진다고 기대하지 않는다. 실제 실행 시점은 키 생성 전략과 트랜잭션 조건을 함께 본다. [Hibernate Flushing](../../raw/spring/hibernate-7-1-flushing.md)

## flush·clear·commit 비교

| 동작 | 역할 | 흔한 오해 |
| --- | --- | --- |
| `flush()` | 대기 중인 엔티티 변경을 SQL로 동기화 | 호출 즉시 commit된다는 해석 |
| `clear()` | 모든 관리 객체를 컨텍스트에서 분리 | 저장을 실행하는 기능이라는 해석 |
| commit | 트랜잭션 완료 | flush와 같은 API라는 해석 |

flush 후에도 트랜잭션을 rollback할 수 있다. 반면 flush 전에 `clear()`하면 아직 DB에 동기화하지 않은 변경을 잃을 수 있다. SQL 실행 순서는 Kotlin 코드에서 프로퍼티를 바꾼 순서와 반드시 같지 않으며 Hibernate의 작업 큐에 따라 결정된다. [Hibernate Flushing](../../raw/spring/hibernate-7-1-flushing.md); [Hibernate 관리 상태 변경](../../raw/spring/hibernate-7-1-persistence-context-lifecycle.md)

## 직접 확인하는 연습

쓰기 트랜잭션에서 회원을 조회하고 이름을 바꾼다. `flush()` 후 SQL 로그의 UPDATE를 확인하고, 예외를 발생시켜 rollback한 뒤 **새 트랜잭션**에서 다시 조회한다. 변경이 취소되는지 확인하면 SQL 실행과 commit을 구분할 수 있다. `readOnly` 트랜잭션은 변경 감지 실습에 사용하지 않는다. [Spring Data 트랜잭션](../../raw/spring/spring-data-jpa-transactions-reference.md)

## See Also

- 이전: [영속성 컨텍스트와 생명주기](jpa-persistence-context-and-entity-lifecycle.md)
- 다음: [값 타입과 Embedded](jpa-embeddable-value-types.md)
- [JPA SQL 로그 설정](spring-database-access-technologies.md#jpa)
