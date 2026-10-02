# Kotlin Null Safety

> Sources: Kotlin Documentation, Unknown
> Raw: [Null safety](../../raw/kotlin/null-safety-3.md)
> Updated: 2026-08-10

## Overview

Kotlin의 null safety는 `null`로 인해 생길 수 있는 문제를 런타임보다 컴파일 시점에 발견하도록 돕는다. 타입을 nullable로 명시하고, 조건 검사·안전 호출(safe call)·Elvis 연산자를 조합해 값이 없을 때의 동작을 표현한다.

## Nullable 타입

Kotlin 타입은 기본적으로 `null`을 허용하지 않는다. 타입 뒤에 `?`를 붙이면 `null`을 담을 수 있는 nullable 타입이 된다.

```kotlin
var neverNull: String = "value"
var nullable: String? = "value"
nullable = null
```

non-null 타입의 변수에 `null`을 대입하거나 nullable 값을 non-null 매개변수에 전달하면 컴파일 오류가 발생한다.

## 명시적 null 검사

조건 표현식에서 `null` 여부를 검사한 뒤 값의 프로퍼티를 사용할 수 있다.

```kotlin
fun describeString(maybeString: String?): String {
    return if (maybeString != null && maybeString.length > 0) {
        "String of length ${maybeString.length}"
    } else {
        "Empty or null string"
    }
}
```

## 안전 호출 연산자

`?.`는 nullable 객체의 프로퍼티나 함수를 안전하게 호출한다. 객체 또는 접근 경로의 값이 `null`이면 호출을 진행하지 않고 `null`을 반환한다.

```kotlin
fun lengthString(value: String?): Int? = value?.length
val country = person.company?.address?.country
val upper = nullable?.uppercase()
```

안전 호출은 연쇄할 수 있으며, 중간의 어느 값이라도 `null`이면 전체 결과가 `null`이 된다.

## Elvis 연산자

`?:`는 왼쪽 표현식의 결과가 `null`일 때 오른쪽의 기본값을 반환한다.

```kotlin
val length = nullable?.length ?: 0
```

안전 호출과 Elvis 연산자를 함께 사용하면 nullable 객체에서 값을 얻되 값이 없을 때의 대안을 간결하게 지정할 수 있다.

## See Also

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
