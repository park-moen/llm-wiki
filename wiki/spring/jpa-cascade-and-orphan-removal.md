# JPA cascade와 orphanRemoval

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, Unknown
> Raw: [Hibernate Cascade](../../raw/spring/hibernate-7-1-entity-cascade.md); [JPA 관계·생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)
> Updated: 2026-10-15

## Overview

cascade는 부모에 수행한 JPA 작업을 연관 엔티티에도 전파하는 설정이다. `orphanRemoval`은 관계에서 빠진 자식을 삭제하는 설정이다. **외래 키 관계를 선언하는 것, JPA 작업을 전파하는 것, DB의 삭제 동작을 설정하는 것은 각각 다르다.** 연관관계 매핑 자체는 기존 [엔티티와 연관 컬렉션](jpa-entity-association-collection-mapping.md)을 먼저 읽는다. [Hibernate Cascade](../../raw/spring/hibernate-7-1-entity-cascade.md)

## 무엇이 전파되는가

| 설정 | 부모에서 시작한 작업 | 자식으로 전파되는 작업 |
| --- | --- | --- |
| `PERSIST` | 새 부모를 영속화 | 연관 자식도 영속화 |
| `MERGE` | 부모 상태를 merge | 자식 상태도 merge |
| `REMOVE` | 부모를 삭제 | 연관 자식도 삭제 |
| `ALL` | 여러 생명주기 작업 | persist·merge·remove·refresh·detach 등 |

`ALL`은 단순히 “자식 저장을 편하게 한다”는 옵션이 아니다. 부모를 삭제할 때 자식도 삭제되는 설정을 포함하므로 공유 객체에 무심코 적용하지 않는다. 표준 명세는 `REMOVE`를 일대일·일대다 이외의 관계에 적용하는 동작의 이식성을 보장하지 않는다. [Hibernate Cascade](../../raw/spring/hibernate-7-1-entity-cascade.md); [JPA 관계·생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## 관계에서 자식을 빼는 경우

다음은 부모가 자식을 전용으로 소유하는 모델의 학습용 매핑 일부다. `OrderLine.order`가 외래 키를 가진 연관 필드라고 가정한다.

```kotlin
@field:OneToMany(
    mappedBy = "order",
    cascade = [CascadeType.PERSIST],
    orphanRemoval = true
)
var lines: MutableList<OrderLine> = mutableListOf()

fun addLine(line: OrderLine) {
    lines.add(line)
    line.order = this
}

fun removeLine(line: OrderLine) {
    lines.remove(line)
    line.order = null
}
```

관리 상태의 부모·자식 관계에서 `removeLine()`을 실행하면, orphanRemoval 규칙에 따라 관계에서 빠진 자식에 대한 remove가 flush 시 적용된다. `orphanRemoval`은 일대일·일대다에서 지원하며, 부모에 전용으로 속한 자식을 위한 기능이다. 새 객체·준영속 객체 등에는 같은 삭제 의미를 그대로 적용하지 않는다. [JPA 관계·생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

양방향 관계의 메모리 상태도 애플리케이션이 맞춰야 한다. `mappedBy`를 붙였다는 사실이나 cascade 설정만으로 양쪽 Kotlin 참조가 자동으로 일치하지 않는다. [JPA 관계·생명주기 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## DB의 `ON DELETE CASCADE`와 비교

JPA의 `REMOVE`는 엔티티 작업을 전파한다. DB 외래 키의 `ON DELETE CASCADE`는 DB가 부모 행 삭제에 따라 자식 행을 삭제하는 규칙이다. Hibernate의 `@OnDelete`는 DB 수준 FK 기능을 사용하는 매핑으로, JPA cascade와 같은 설정으로 해석하면 안 된다. [Hibernate Cascade](../../raw/spring/hibernate-7-1-entity-cascade.md)

## 직접 확인하는 연습

부모와 자식을 저장한 뒤 새 트랜잭션에서 조회한다. 부모 자체는 유지하면서 자식만 컬렉션에서 제거하고 flush한다. `orphanRemoval`을 끈 경우와 켠 경우의 SQL을 비교한다. 실습 데이터는 전용 자식 관계를 사용하고, 공유 회원·상품 같은 객체에는 삭제 전파를 적용하지 않는다.

## See Also

- 이전: [값 타입과 Embedded](jpa-embeddable-value-types.md)
- 다음: [JPQL 조인·집계·DTO 조회](jpa-jpql-joins-and-aggregation.md)
- [DB 외래 키 삭제 규칙](../database/foreign-key-on-delete-actions.md)
