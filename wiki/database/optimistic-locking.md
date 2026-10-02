# 낙관적 잠금 (Optimistic Locking)

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, 2024-04-10
> Raw: [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md); [Jakarta Persistence 명세: 낙관적 잠금과 엔티티 버전](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)
> Updated: 2026-10-02

## Overview

낙관적 잠금은 여러 작업이 같은 데이터를 읽더라도 먼저 잠그지 않고, 변경을 반영할 때 다른 작업이 그사이 데이터를 바꿨는지 확인하는 동시성 제어 방식이다. 충돌이 확인되면 뒤늦게 변경을 반영하려던 작업을 실패시켜 앞선 변경이 덮어써지는 일을 막는다. 데이터는 자주 읽고 드물게 수정하는 상황에 잘 맞는다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

## DDL, SQL과의 관계

낙관적 잠금은 `CREATE TABLE`이나 `ALTER TABLE` 같은 DDL 명령 또는 제약 조건의 이름이 아니다. 버전 기반 방식을 쓰려면 테이블에 버전 값을 담는 열을 마련할 수 있지만, 열을 만드는 것만으로 충돌 검사가 작동하지는 않는다. 데이터를 수정하거나 삭제할 때 읽어 둔 버전과 현재 버전을 비교하는 동작이 함께 필요하다. Hibernate에서는 entity의 `@Version` 속성을 버전 열에 매핑하고, 이 값으로 충돌하는 갱신을 감지한다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

## Hibernate/JPA에서 사용하는 방법

일반적인 방식은 entity의 버전 속성에 `@Version`을 붙이는 것이다. 버전은 순차적으로 증가하는 숫자나 timestamp가 될 수 있다. Hibernate는 버전 속성을 직접 수정하지 말라고 안내하며, 충돌한 갱신은 성공한 것으로 처리하지 않는다. 숫자 버전은 timestamp보다 신뢰성이 높다고 설명한다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

버전 검사에 실패하면 JPA 구현체는 `OptimisticLockException`을 던지고 현재 트랜잭션을 롤백 대상으로 표시한다. 검사 시점은 구현체에 따라 `merge`, flush 또는 커밋일 수 있다. 같은 값을 읽은 두 작업 중 뒤늦은 저장이 앞선 변경을 덮어쓰는 과정은 [갱신 손실](lost-update-optimistic-locking.md)에 예시로 정리했다. [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

전용 버전 열을 추가하기 어려운 기존 schema라면 Hibernate의 `@OptimisticLocking`을 사용해 갱신 또는 삭제 SQL의 `WHERE` 조건에 entity의 모든 필드(`ALL`)나 변경된 필드(`DIRTY`)를 비교 대상으로 넣을 수도 있다. 이는 Hibernate가 제공하는 별도 방식이므로, 낙관적 잠금에 항상 버전 열이 필요한 것은 아니다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

## See Also

- [낙관적 락과 비관적 락: 충돌 처리와 선택 기준](optimistic-vs-pessimistic-locking.md)
- [갱신 손실 (Lost Update): 낙관적 잠금이 막는 동시 수정](lost-update-optimistic-locking.md)
- [Spring DB 접근 기술 비교](../spring/spring-database-access-technologies.md)
