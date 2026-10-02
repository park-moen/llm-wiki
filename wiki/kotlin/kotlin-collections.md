# Kotlin 컬렉션

> Sources: Kotlin Documentation, Unknown
> Raw: [Collections](../../raw/kotlin/collections-3.md)
> Updated: 2026-08-10

## Overview

Kotlin은 데이터를 묶어 처리하기 위해 `List`, `Set`, `Map` 컬렉션을 제공한다. 각 컬렉션에는 읽기 전용(read-only) 인터페이스와 변경 가능한(mutable) 인터페이스가 있으며, 생성 시 원소 타입을 추론하거나 제네릭 타입으로 명시할 수 있다.

## 컬렉션 선택

| 컬렉션 | 핵심 특성 | 읽기 전용 생성 | 변경 가능 생성 |
|---|---|---|---|
| `List` | 삽입 순서를 유지하고 중복을 허용 | `listOf()` | `mutableListOf()` |
| `Set` | 순서가 없고 고유한 원소만 보관 | `setOf()` | `mutableSetOf()` |
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

## See Also

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- [Kotlin Null Safety](kotlin-null-safety.md)
