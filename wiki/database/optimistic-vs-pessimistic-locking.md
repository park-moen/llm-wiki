# 낙관적 락과 비관적 락: 충돌 처리와 선택 기준

> Sources: Jakarta Persistence, 2024-04-10; Hibernate ORM, Unknown; Hibernate ORM, 2026-09-17; Spring Data JPA, Unknown
> Raw: [Jakarta Persistence 명세: 낙관적 락과 엔티티 버전](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md); [Jakarta Persistence 명세: 비관적 락과 락 모드](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md); [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md); [Hibernate ORM User Guide: Long Conversations and Concurrency](../../raw/database/2026-09-17-hibernate-7-1-long-conversation-concurrency.md); [Spring Data JPA: Locking](../../raw/spring/spring-data-jpa-locking-reference.md)
> Updated: 2026-10-02

## Overview

두 방식은 **같은 데이터를 수정하려는 작업이 충돌할 때 언제 대응하는가**가 다르다. 낙관적 락은 먼저 작업을 진행하고 저장할 때 읽어 둔 버전이 여전히 유효한지 확인한다. 비관적 락은 DB 락을 먼저 얻어, 락을 보유한 트랜잭션이 끝날 때까지 다른 트랜잭션의 같은 엔티티 갱신을 막는다. [Jakarta Persistence 명세: 낙관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md) · [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

| 비교 | 낙관적 락 | 비관적 락 |
| --- | --- | --- |
| 충돌 대응 | 수정 시 버전 불일치를 검사하고 실패시킴 | 선택한 엔티티에 DB 락을 먼저 얻고 트랜잭션이 끝날 때까지 유지함 |
| JPA의 대표 수단 | `@Version` | `LockModeType.PESSIMISTIC_WRITE` |
| 상대 작업 | 먼저 읽고 편집할 수 있지만 나중에 저장할 때 충돌할 수 있음 | 같은 행을 수정하거나 삭제하려는 트랜잭션은 락이 해제될 때까지 성공할 수 없음 |
| 비용 | 충돌하면 진행한 작업을 롤백하고 처리해야 함 | 락 대기·획득 실패가 생길 수 있고 락 범위가 넓어질 수도 있음 |

낙관적 락의 버전 검사와 롤백은 Jakarta Persistence 명세가 규정한다. 비관적 락의 DB 락 유지 기간·범위·획득 실패도 같은 명세가 규정한다. **비관적 락이 모든 일반 조회를 막는다고 해석하면 안 된다.** `PESSIMISTIC_READ`는 다른 읽기를 막지 않는 용도이며, DB별 락 구현도 다르다. [Jakarta Persistence 명세: 낙관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md) · [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

## 같은 게시글을 고친다면

아래는 A와 B가 같은 게시글을 수정하는 **개념 예시**다.

- **낙관적 락:** A와 B가 기존 내용을 읽고 편집한다. A가 먼저 저장해 버전이 바뀌면, 예전 버전을 들고 있는 B의 저장은 `OptimisticLockException`으로 실패하고 해당 트랜잭션은 롤백 대상으로 표시된다. [Jakarta Persistence 명세: 낙관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)
- **비관적 락:** A가 트랜잭션 안에서 게시글에 `PESSIMISTIC_WRITE` 락을 얻는다. B가 같은 행을 수정하려는 트랜잭션은 A의 트랜잭션이 끝날 때까지 성공할 수 없다. 락 획득에 실패하면 DB의 롤백 범위에 따라 `PessimisticLockException` 또는 `LockTimeoutException`이 발생할 수 있다. [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

## JPA와 Spring Data JPA에서의 형태

낙관적 락은 엔티티에 `@Version` 속성을 두는 것이 이식성 있는 기본 방식이다. 버전이 있는 엔티티는 JPA 구현체가 변경 시 자동으로 검사한다. 비관적 락은 필요한 조회에 `PESSIMISTIC_WRITE` 같은 락 모드를 지정한다. [Jakarta Persistence 명세: 낙관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md) · [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

```kotlin
// 개념 예시: 한 트랜잭션 안에서 락을 얻고 수정한다.
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select p from Post p where p.id = :id")
fun findForUpdate(id: Long): Post?
```

Spring Data JPA는 저장소의 조회 메서드나 다시 선언한 CRUD 메서드에 `@Lock`을 붙여 JPA 락 모드를 전달한다. 위 코드는 그 방식을 `PESSIMISTIC_WRITE`에 적용한 **개념 예시**이며, 락을 건 조회와 수정은 같은 트랜잭션 안에서 처리해야 한다. Hibernate는 비관적 락에 DB의 락 기능을 사용하며, `SELECT ... FOR UPDATE`는 가능한 SQL 형태의 예시일 뿐 모든 DB에서 같은 SQL이 생성된다는 뜻은 아니다. [Spring Data JPA: Locking](../../raw/spring/spring-data-jpa-locking-reference.md) · [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md) · [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

## 언제 선택하는가

- **읽기가 많고 충돌이 드물거나 사람이 화면에서 오래 편집한다면:** `@Version` 기반 낙관적 락을 먼저 검토한다. Hibernate는 사용자 편집 시간에 DB 트랜잭션과 락을 계속 유지하는 방식을 락 경합 때문에 피해야 한다고 설명한다. [Hibernate ORM: Long Conversations and Concurrency](../../raw/database/2026-09-17-hibernate-7-1-long-conversation-concurrency.md) · [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)
- **같은 행을 여러 트랜잭션이 자주 수정하고, 뒤늦게 충돌해 작업을 버리는 비용이 크다면:** 짧은 트랜잭션에서 비관적 락을 검토한다. Jakarta Persistence 명세는 낙관적 락의 늦은 실패가 문제가 되는 경우와 높은 충돌 가능성을 비관적 락의 사용 근거로 든다. [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)
- **락 대기와 범위도 함께 판단한다:** 비관적 락은 다른 갱신을 기다리게 하고, JPA 명세는 구현체나 DB가 요청보다 더 많은 행에 락을 걸 수 있다고 한다. 따라서 락이 필요한 구간을 짧게 유지한다. 마지막 문장은 락 유지 기간과 범위 규칙에서 도출한 설계 지침이다. [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

**주의:** 두 방식을 같은 엔티티에 함께 사용할 수도 있다. 버전이 있는 엔티티에 비관적 락을 요청하면 JPA가 버전 검사도 수행할 수 있으므로, 두 방식이 언제나 서로 배타적인 설정은 아니다. [Jakarta Persistence 명세: 비관적 락](../../raw/database/2024-04-10-jakarta-persistence-3-2-pessimistic-locking.md)

## See Also

- [낙관적 락 (Optimistic Locking)](optimistic-locking.md)
- [갱신 손실 (Lost Update): 낙관적 락이 막는 동시 수정](lost-update-optimistic-locking.md)
