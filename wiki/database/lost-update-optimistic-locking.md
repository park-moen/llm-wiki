# 갱신 손실 (Lost Update): 낙관적 잠금이 막는 동시 수정

> Sources: Jakarta Persistence, 2024-04-10; Hibernate ORM, Unknown; Hibernate ORM, 2026-09-17; MySQL Reference Manual, Unknown
> Raw: [Jakarta Persistence 명세: 낙관적 잠금과 엔티티 버전](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md); [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md); [Hibernate ORM User Guide: Long Conversations and Concurrency](../../raw/database/2026-09-17-hibernate-7-1-long-conversation-concurrency.md); [MySQL Reference Manual: Locking Reads and Counter Example](../../raw/database/mysql-8-4-locking-reads-counter-example.md)
> Updated: 2026-10-02

## Overview

여기서 **갱신 손실**은 두 작업이 같은 엔티티의 이전 상태를 읽고 각각 수정할 때, 뒤늦게 저장된 변경이 먼저 저장된 변경을 덮어써서 앞선 작업의 결과가 사라지는 상황을 뜻한다. Hibernate는 이를 버전 검사 없이 마지막 커밋이 이기는 방식에서 생길 수 있는 손실로 설명한다. 이 문서는 **JPA 낙관적 잠금이 다루는 동시 수정**으로 범위를 한정한다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md) · [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

## 같은 게시글을 동시에 고치는 예시

아래는 두 관리자가 **같은 제목을 읽은 뒤** 각각 수정하는 가상 예시다. 처음 제목은 `공지`이고, A와 B는 모두 이 제목을 읽었다.

| 순서 | A | B | 저장된 제목 |
| --- | --- | --- | --- |
| 처음 읽기 | `공지`를 읽음 | `공지`를 읽음 | `공지` |
| 먼저 저장 | `행사 안내`로 저장 | 읽어 둔 `공지`를 바탕으로 편집 중 | `행사 안내` |
| 나중에 저장 | 이미 편집을 마침 | `점검 안내`로 저장 | `점검 안내` |

A의 `행사 안내`가 B에게 알려지지 않은 채 사라졌다. 핵심은 B가 **A의 변경 전 상태를 읽고 편집했다**는 점이다. A가 바꾼 최신 내용을 확인한 뒤 B가 의도적으로 다시 고친 상황과 구분해야 한다. 이는 읽은 버전과 수정 시점의 버전 사이에 다른 트랜잭션의 변경이 있었는지 검사한다는 명세를 풀어 쓴 예시다. [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

## 실제로 마주치기 쉬운 상황

### 오래 열어 둔 수정 화면에서 저장

관리자 A가 상품 수정 화면을 열고 가격을 편집하는 동안, B가 같은 상품을 먼저 수정한다. A가 나중에 **처음 화면에 표시됐던 값**을 바탕으로 저장하면 B의 변경을 덮을 수 있다. 두 사람이 같은 순간에 저장 버튼을 눌러야만 생기는 문제는 아니다. Hibernate는 사용자가 화면을 편집하는 시간 동안 다른 변경이 있었는지 자동 버전 검사로 감지하는 상황을 설명한다. 이 상품 예시는 그 설명을 적용한 것이다. [Hibernate ORM: Long Conversations and Concurrency](../../raw/database/2026-09-17-hibernate-7-1-long-conversation-concurrency.md) · [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

### 읽어 둔 값으로 카운터를 계산해 저장

예를 들어 두 요청이 같은 조회 수 `n`을 읽고 각각 애플리케이션에서 `n + 1`을 계산한 뒤, 계산한 값을 엔티티에 넣어 저장한다고 하자. 둘 다 **같은 이전 값**으로 계산하면 두 요청이 저장에 성공해도 증가분 하나가 사라질 수 있다. 이는 MySQL 문서가 카운터를 일반 조회로 읽을 때 두 사용자가 같은 값을 볼 수 있다고 경고하는 상황을, 값 덮어쓰기 관점에서 바꿔 본 예시다. [MySQL: Locking Reads](../../raw/database/mysql-8-4-locking-reads-counter-example.md) · [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

`@Version`으로 두 번째 저장의 충돌을 감지할 수 있다. 카운터 증가만 필요하다면 DB가 현재 값에 직접 더하는 `UPDATE ... SET count = count + 1` 형태도 검토할 수 있다. MySQL 문서도 카운터를 잠그고 증가시키는 방식과 단일 SQL 갱신을 제시한다. **애플리케이션에서 읽은 이전 값으로 새 값을 만들어 덮어쓰는 코드**와 **DB 안에서 현재 값에 더하는 SQL**은 동작이 다르다. [MySQL: Locking Reads](../../raw/database/mysql-8-4-locking-reads-counter-example.md)

### 서로 다른 필드를 고쳤는데도 앞선 변경이 사라짐

A가 전화번호를 고치고 B가 통화 횟수를 늘리는 경우처럼 수정한 필드가 달라도, 뒤늦은 갱신이 **읽어 둔 다른 필드의 옛값까지 함께 기록하면** 앞선 변경이 사라질 수 있다. Hibernate 공식 예시는 `@OptimisticLock(excluded = true)`로 통화 횟수를 버전 증가 대상에서 빼면, 전화번호 갱신이 충돌 없이 통화 횟수를 이전 값으로 되돌리는 과정을 보여준다. 이는 모든 부분 갱신에서 반드시 일어나는 일이 아니라, 버전 검사 대상과 실제 `UPDATE`에 포함된 열에 따라 달라지는 사례다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

## `@Version`을 붙이면 어떻게 달라지는가

엔티티에 `@Version` 필드를 두면 JPA 구현체는 읽을 때의 버전을 기억하고, 갱신할 때 DB의 현재 버전과 비교한다. A가 먼저 저장하면 버전이 바뀌므로, 이전 버전을 들고 있던 B의 갱신은 충돌로 처리된다. **B의 값으로 덮어쓰는 대신 B의 작업이 실패하고 A의 변경은 유지된다.** [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md) · [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

```kotlin
@Entity
class Post(
    @Id val id: Long,
    var title: String,
    @Version var version: Long? = null,
)
```

위 코드는 버전 필드의 위치를 보여 주는 **개념 예시**다. 실제 엔티티의 생성 방식과 Kotlin/JPA 매핑 설정은 프로젝트에 맞춰야 한다. Hibernate는 `@Version` 속성을 DB의 버전 열에 매핑해 충돌을 감지한다. [Hibernate ORM User Guide: Locking](../../raw/database/hibernate-orm-optimistic-locking.md)

버전이 달라지면 JPA 구현체는 `OptimisticLockException`을 던지고 현재 트랜잭션을 롤백 대상으로 표시해야 한다. 검사는 `save` 호출 순간으로 고정되지 않으며, 구현체에 따라 `merge`, flush 또는 커밋 시점에 발생할 수 있다. 따라서 충돌을 받은 요청은 최신 데이터를 다시 읽고 사용자가 변경을 비교하거나 재시도할지 결정하는 흐름이 필요하다. 마지막 문장은 명세의 실패·롤백 규칙에서 도출한 **애플리케이션 처리 지침**이다. [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

## 범위와 주의점

`@Version`은 버전이 매핑된 엔티티에 대한 낙관적 검사다. 한 작업에서 버전 없는 다른 엔티티까지 함께 갱신하면 객체 그래프 전체의 일관성까지 보장한다고 볼 수 없다. 또한 버전 없이 동시 수정을 허용한다면 애플리케이션이 데이터 일관성을 직접 책임져야 한다. [Jakarta Persistence 명세](../../raw/database/2024-04-10-jakarta-persistence-3-2-optimistic-locking.md)

## See Also

- [낙관적 락과 비관적 락: 충돌 처리와 선택 기준](optimistic-vs-pessimistic-locking.md)
- [낙관적 잠금 (Optimistic Locking)](optimistic-locking.md)
