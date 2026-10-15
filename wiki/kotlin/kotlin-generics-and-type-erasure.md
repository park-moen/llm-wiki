# Kotlin 제네릭·variance·타입 소거

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown
> Raw: [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md); [Kotlin language specification — Type system](../../raw/kotlin/kotlin-spec-type-system.md); [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)
> Updated: 2026-10-15

## Overview

제네릭은 타입 사이의 관계를 표현하지만, 타입 인자가 실행 중 항상 그대로 남는 것은 아니다. **컴파일 시 타입 검사와 실행 시 타입 정보**를 나눠 이해해야 한다. `in`·`out`은 어떤 방향으로 값을 안전하게 전달할 수 있는지 제약한다. [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md)

## out과 in을 읽는 방법

`Producer<out T>`는 주로 `T`를 밖으로 제공하는 방향을, `Consumer<in T>`는 `T`를 받는 방향을 나타낸다. 선언 위치의 variance와 개별 사용 위치의 projection을 구분한다. 같은 원소 관계라도 mutable container를 아무 방향으로나 대입할 수 있는 것은 아니다. [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md); [Kotlin language specification — Type system](../../raw/kotlin/kotlin-spec-type-system.md)

## 타입 소거가 의미하는 것

일반 제네릭 인스턴스의 타입 인자는 실행 시 소거된다. `List<*>`인지 검사하는 것과 `List<String>`의 모든 원소가 문자열인지 검사하는 것은 다르다. `as List<String>` 같은 unchecked cast가 통과했다고 원소의 안전성까지 검증된 것으로 보지 않는다. [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md)

`inline fun <reified T>`는 호출 위치의 타입을 코드에 반영할 수 있다. 하지만 `T`가 `List<String>`처럼 또 다른 제네릭 구조를 포함하면 그 안쪽 인자까지 언제나 검사할 수 있다는 뜻은 아니다. [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md); [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## 직접 확인하는 연습

variance와 star projection을 비교하기 위한 학습용 코드다.

```kotlin
class Producer<out T>(private val value: T) {
    fun read(): T = value
}

fun main() {
    val text: Producer<String> = Producer("hello")
    val broader: Producer<Any> = text
    val values: List<*> = listOf("hello", 3)
    println(broader.read())
    println(values.map { it is String })
}
```

예상 출력:

```text
hello
[true, false]
```

`List<*>`에서는 원소를 구체적인 문자열로 단정하지 않는다. 각 원소를 `is String`으로 검사하는 것과 컬렉션 전체를 unchecked cast하는 것을 비교해 본다. [Generics: in, out, where](../../raw/kotlin/kotlin-generics.md)

## 다음에 확인할 것

함수의 타입 인자를 실행 중 사용해야 한다면 `Class`·`KClass` 같은 명시적 토큰을 받을지, 호출 위치에서 inline·reified로 반영할지 결정한다. 타입 소거 문제를 compiler 경고 무시만으로 처리하지 않는다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## See Also


- 학습 순서 이전: [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
- 학습 순서 다음: [Kotlin 확장 함수](kotlin-extension-functions.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
