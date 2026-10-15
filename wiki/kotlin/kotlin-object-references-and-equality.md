# Kotlin 객체 참조·동등성·불변성

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown; Oracle Java SE Documentation, Unknown
> Raw: [Equality](../../raw/kotlin/kotlin-equality.md); [Collections overview](../../raw/kotlin/kotlin-collections-overview.md); [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)
> Updated: 2026-10-15

## Overview

객체를 변수에 대입하거나 함수에 전달하면 서로 다른 이름이 같은 객체를 가리킬 수 있다. Kotlin/JVM에서는 참조도 전달되는 값이다. **매개변수에 다른 객체를 대입하는 것과, 전달된 참조로 객체 내부를 바꾸는 것은 다른 동작**이다. JVM 명세의 참조 타입·지역 변수·호출 frame 모델을 바탕으로 읽는다. [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md)

## val은 객체를 동결하지 않는다

`val`은 참조를 다른 값으로 재할당하지 못하게 할 뿐, 그 참조가 가리키는 객체의 모든 변경을 막지 않는다. 읽기 전용 `List`도 다른 mutable 참조가 같은 객체를 바꿀 가능성까지 없애는 것은 아니다. [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)

## 두 가지 동등성

| 표현 | 의미 |
| --- | --- |
| `a == b` | null을 고려한 구조적 동등성 검사; 일반 객체는 `equals()`와 연결 |
| `a === b` | 같은 객체를 가리키는지 참조 동등성 검사 |

일반 클래스와 `data class`의 `equals()` 동작은 다를 수 있다. `data class`는 주 생성자의 프로퍼티를 기준으로 생성된 값 동등성을 사용한다. 배열의 원소 비교는 `contentEquals()` 같은 배열 전용 연산도 구분한다. [Equality](../../raw/kotlin/kotlin-equality.md)

## 직접 확인하는 연습

동일 객체 공유와 얕은 복사를 보여 주기 위해 만든 학습용 예제다.

```kotlin
data class Basket(val items: MutableList<String>)

fun append(items: MutableList<String>) {
    items.add("banana")
}

fun main() {
    val basket = Basket(mutableListOf("apple"))
    val alias = basket
    val copied = basket.copy()
    append(copied.items)
    println(basket === alias)
    println(basket === copied)
    println(basket == copied)
    println(basket.items.joinToString())
}
```

예상 출력:

```text
true
false
true
apple, banana
```

`copy()`는 바깥 `Basket`을 새로 만들지만 `items`를 재귀적으로 복제하지 않는다. 기존 [클래스와 data class](kotlin-classes-and-data-classes.md)의 얕은 복사 설명과 연결된다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [Equality](../../raw/kotlin/kotlin-equality.md)

## equals와 hashCode를 함께 보기

동등한 객체는 같은 hash code를 가져야 한다. `equals()`를 재정의할 때 `hashCode()`도 계약에 맞게 다룬다. 해시 기반 컬렉션에 넣은 뒤 동등성·hash code에 영향을 주는 값을 바꾸는 설계는 신중히 검토한다. 마지막 문장은 동등성과 해시 기반 저장 구조의 관계에서 도출한 설계 지침이다. [Equality](../../raw/kotlin/kotlin-equality.md); [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)

## See Also


- 학습 순서 이전: [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)
- 학습 순서 다음: [Kotlin Objects](kotlin-objects.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
