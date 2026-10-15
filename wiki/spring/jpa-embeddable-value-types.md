# JPA 값 타입과 @Embeddable·@Embedded

> Sources: Hibernate ORM, Unknown; Jakarta Persistence, Unknown
> Raw: [Hibernate Embeddable values](../../raw/spring/hibernate-7-1-embeddable-values.md); [JPA 값 타입 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)
> Updated: 2026-10-15

## Overview

주소처럼 여러 필드를 한 값으로 묶되 별도 엔티티 식별자가 필요하지 않다면 embeddable을 사용할 수 있다. **`@Embeddable`은 값 타입의 정의에, `@Embedded`는 그 값을 사용하는 필드에 붙인다.** 저장되는 값은 소유 엔티티의 상태에 속한다. [Hibernate Embeddable values](../../raw/spring/hibernate-7-1-embeddable-values.md)

## 별도 엔티티와 구분하기

엔티티는 영속 식별자로 구분한다. embeddable은 독립된 영속 식별자가 없으며 소유 엔티티 상태의 일부로 존재한다. 주소를 값으로 다룰지 별도 엔티티로 다룰지는 자체 식별자와 독립적인 관리가 필요한지에 따라 정한다. 뒤 문장은 명세의 식별자·소유 규칙에서 도출한 설계 지침이다. [JPA 값 타입 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## Kotlin으로 주소를 묶는 예제

공식 매핑을 Kotlin으로 옮긴 학습용 코드다. 기존 엔티티를 바꾸라는 지시가 아니라 별도의 매핑 예제다.

```kotlin
@Embeddable
open class Address protected constructor() {
    @field:Column(name = "city")
    open var city: String = ""
        protected set

    @field:Column(name = "street")
    open var street: String = ""
        protected set

    constructor(city: String, street: String) : this() {
        this.city = city
        this.street = street
    }
}

// 엔티티 본문 안에서 사용
@field:Embedded
var address: Address = Address("서울", "테헤란로")
```

기본적인 embedded 매핑에서는 주소의 각 프로퍼티가 소유 엔티티 테이블의 column으로 저장된다. 주소 객체 자체를 담는 column이 생기거나 `@Embedded`만으로 별도 주소 테이블이 생기는 것이 아니다. Hibernate의 별도 aggregate 매핑 기능과도 구분한다. [Hibernate Embeddable values](../../raw/spring/hibernate-7-1-embeddable-values.md)

여기서 “기본 생성자”는 `Address()`처럼 **인자가 없는 생성자**를 뜻한다. Kotlin의 주 생성자나 `init`과 다른 개념이다. 일반 클래스인 embeddable에는 JPA의 생성자 요구사항을 적용하며, 명세가 별도로 허용하는 Java record는 Kotlin의 `data class`와 같은 문법이 아니다. Kotlin plugin 설정은 기존 [JPA 설정](spring-database-access-technologies.md)을 참고한다. [JPA 값 타입 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## 같은 타입을 여러 필드에 사용할 때

청구 주소와 배송 주소에 같은 `Address` 타입을 쓰면 기본 column 이름이 겹칠 수 있다. 이때 표준 `@AttributeOverride`로 사용하는 위치마다 column 이름을 지정한다.

```kotlin
@field:Embedded
@field:AttributeOverrides(
    AttributeOverride(name = "city", column = Column(name = "shipping_city")),
    AttributeOverride(name = "street", column = Column(name = "shipping_street"))
)
var shippingAddress: Address = Address("서울", "배송지")
```

`name`은 Kotlin 프로퍼티 이름이고 `Column(name = …)`은 DB column 이름이다. 청구 주소에도 같은 방식으로 별도 이름을 지정한다. [Hibernate Embeddable values](../../raw/spring/hibernate-7-1-embeddable-values.md)

## 객체 공유에 주의하기

embedded 객체는 소유 엔티티에 속하며 여러 영속 엔티티 사이에 같은 인스턴스를 공유했을 때의 의미는 명세상 정의되지 않는다. 두 회원의 주소 값이 같더라도 각각 새 `Address`를 만들어 넣는 방식이 안전하다. 위 예제에서 외부 setter를 막은 이유도 값 일부를 여기저기서 바꾸기보다 새 값으로 교체하도록 유도하기 위해서다. 마지막 두 문장은 소유 규칙에서 도출한 학습 예제의 설계 선택이다. [JPA 값 타입 규칙](../../raw/spring/jakarta-persistence-3-2-value-types-relationships-lifecycle.md)

## 직접 작성하는 연습

한 엔티티에 청구 주소와 배송 주소를 선언하고 column 이름이 겹치지 않도록 매핑한다. 저장 후 `clear()`하고 다시 조회해 두 주소가 각각 복원되는지 확인한다. 값 컬렉션은 새로 중복 정리하지 않고 기존 [ElementCollection 문서](jpa-element-collection.md)에서 이어서 읽는다.

## See Also

- 이전: [변경 감지와 flush](jpa-dirty-checking-and-flush.md)
- 다음: [cascade와 orphanRemoval](jpa-cascade-and-orphan-removal.md)
- [JPA ElementCollection](jpa-element-collection.md)
