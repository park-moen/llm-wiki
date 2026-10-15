# Kotlin 컬렉션

> Sources: Kotlin Documentation, Unknown; Kotlin Documentation, 2026-09-02
> Raw: [Collections](../../raw/kotlin/collections-3.md); [Null safety — intermediate tour](../../raw/kotlin/2026-09-02-kotlin-intermediate-null-safety.md); [Collections overview](../../raw/kotlin/kotlin-collections-overview.md); [Equality](../../raw/kotlin/kotlin-equality.md)
> Updated: 2026-10-15

## Overview

Kotlin은 데이터를 묶어 처리하기 위해 `List`, `Set`, `Map` 컬렉션을 제공한다. 각 컬렉션에는 읽기 전용(read-only) 인터페이스와 변경 가능한(mutable) 인터페이스가 있으며, 생성 시 원소 타입을 추론하거나 제네릭 타입으로 명시할 수 있다.

## 컬렉션 선택

| 컬렉션 | 핵심 특성 | 읽기 전용 생성 | 변경 가능 생성 |
|---|---|---|---|
| `List` | 삽입 순서를 유지하고 중복을 허용 | `listOf()` | `mutableListOf()` |
| `Set` | 고유한 원소를 보관하며 반복 순서는 구현에 따라 다름 | `setOf()` | `mutableSetOf()` |
| `Map` | 고유한 키를 값에 대응하며 값 중복은 허용 | `mapOf()` | `mutableMapOf()` |

변경 가능한 컬렉션을 읽기 전용 타입에 대입하면 읽기 전용 뷰를 만들 수 있다.

```kotlin
val mutableShapes: MutableList<String> = mutableListOf("triangle", "square")
val shapes: List<String> = mutableShapes
```

## List

리스트는 `[]` 인덱스 연산자로 원소에 접근하며 `.first()`, `.last()`로 양 끝의 원소를 얻을 수 있다. `.count()`는 원소 수를 반환하고 `in`은 원소 포함 여부를 검사한다. `MutableList`에서는 `.add()`와 `.remove()`로 원소를 추가하거나 제거한다.

```kotlin
val shapes = listOf("triangle", "square", "circle")
val first = shapes[0]
val containsCircle = "circle" in shapes
```

## Set

집합은 중복 원소를 제거하고 특정 인덱스로 접근할 수 없다. `List`와 마찬가지로 `.count()`와 `in`을 사용할 수 있으며, `MutableSet`에서는 `.add()`와 `.remove()`로 내용을 변경한다.

```kotlin
val fruit = setOf("apple", "banana", "cherry", "cherry")
```

## Map

맵은 `to`로 키-값 쌍을 만들 수 있으며, 키는 고유해야 하지만 값은 중복될 수 있다. 타입을 명시할 때는 키 타입과 값 타입을 함께 쓴다.

```kotlin
val menu: MutableMap<String, Int> = mutableMapOf(
    "apple" to 100,
    "kiwi" to 190
)
```

`map[key]`로 값을 조회하거나 `MutableMap`에 값을 추가한다. 존재하지 않는 키를 조회하면 `null`이 반환된다. `.remove()`는 항목을 제거하며, `.containsKey()`는 특정 키의 존재 여부를 검사한다. `keys`와 `values` 프로퍼티로 키 또는 값의 컬렉션을 얻을 수 있고, `in`으로 키나 값의 포함 여부를 확인할 수 있다.

```kotlin
menu["coconut"] = 150
val missing = menu["pineapple"] // null
val hasApple = "apple" in menu
```

## null을 제외하거나 결과 부재 처리하기

`filterNotNull()`은 기존 리스트의 null 원소를 제외하고, `listOfNotNull()`은 null을 제외해 리스트를 생성한다. 변환과 null 결과 제거를 함께 하려면 `mapNotNull()`을 사용한다. 빈 컬렉션의 최댓값·최솟값은 `maxOrNull()`·`minOrNull()`로 조회하면 null로 처리할 수 있다.

조건을 만족하는 원소가 정확히 하나인지 확인할 때는 `singleOrNull { ... }`을 쓴다. 일치하는 원소가 없거나 여럿이면 null을 반환한다. 변환 결과에서 처음 non-null 값을 찾는 `firstNotNullOfOrNull()`과 빈 컬렉션에 null을 반환하는 `reduceOrNull()`의 비교·예제는 [Kotlin Null Safety](kotlin-null-safety.md)에 정리했다.

## read-only와 불변은 다르다

`List<T>`는 그 인터페이스로 변경 연산을 제공하지 않는다는 뜻이다. 같은 객체를 가리키는 `MutableList<T>` 참조가 남아 있으면 다른 경로로 내용이 바뀔 수 있다. `val` 또한 컬렉션 참조의 재할당을 막을 뿐 내부 원소를 모두 동결하지 않는다. [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)

```kotlin
fun main() {
    val mutable = mutableListOf("apple")
    val readOnly: List<String> = mutable
    mutable.add("banana")
    println(readOnly.joinToString())
}
```

출력은 `apple, banana`다. API에서 읽기 전용 타입을 반환하는 것과 객체를 독립적으로 복사해 전달하는 것은 별도의 설계 선택이다. [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)

## 구현과 동등성의 영향을 함께 본다

Set은 원소의 고유성을 표현하며 일반 계약이 특정 인덱스 접근을 제공하는 것은 아니다. 그렇다고 모든 구현의 반복 순서가 무작위인 것은 아니다. `LinkedHashSet` 등 순서 특성을 가진 구현을 구분한다. Map도 사용하는 구현의 순서·해시 특성을 함께 확인한다. [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)

컬렉션에 넣는 객체의 `equals()`·`hashCode()` 계약은 중복 판단과 해시 기반 조회에 영향을 준다. [객체 참조와 동등성](kotlin-object-references-and-equality.md)을 먼저 읽고, 변환 체인의 실행 순서는 [Sequence](kotlin-sequences-and-lazy-evaluation.md)에서 이어서 비교한다. [Equality](../../raw/kotlin/kotlin-equality.md)

## 직접 작성하는 연습: 상품 목록과 재고 Map 수정

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

변경 가능한 상품 목록에 book을 추가하고 pen을 제거하세요. 재고 Map에는 book 수량을 2로 저장하세요.

### 시작 코드

```kotlin
fun main() {
    val products = mutableListOf("pen", "cup")
    // TODO: products에 book 추가
    // TODO: products에서 pen 제거
    val stock = mutableMapOf("cup" to 1)
    // TODO: stock에 book 수량 2 저장
    println(products)
    println(stock["book"])
    println(stock["pen"])
}
```

### 예상 출력

```text
[cup, book]
2
null
```

> [!tip]- 힌트
> MutableList의 add·remove와 Map의 []를 사용하세요.

> [!success]- 정답 확인
> ```kotlin
> fun main() {
>     val products = mutableListOf("pen", "cup")
>     products.add("book")
>     products.remove("pen")
>     val stock = mutableMapOf("cup" to 1)
>     stock["book"] = 2
>     println(products)
>     println(stock["book"])
>     println(stock["pen"])
> }
> ```

### 한 번 더 바꿔 보기

products 선언을 listOf로 바꾸면 어떤 수정이 불가능해지는지 확인하세요.

## See Also

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- [Kotlin Null Safety](kotlin-null-safety.md)

- 학습 순서 이전: [Kotlin 특수 클래스](kotlin-special-classes.md)
- 학습 순서 다음: [Kotlin Sequence와 지연 평가](kotlin-sequences-and-lazy-evaluation.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
