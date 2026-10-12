# Kotlin Null Safety

> Sources: Kotlin Documentation, Unknown; Kotlin Documentation, 2026-09-02
> Raw: [Null safety](../../raw/kotlin/null-safety-3.md); [Null safety — intermediate tour](../../raw/kotlin/2026-09-02-kotlin-intermediate-null-safety.md)
> Updated: 2026-10-12

## Overview

Kotlin의 null safety는 `null`로 인해 생길 수 있는 문제를 런타임보다 컴파일 시점에 발견하도록 돕는다. 타입을 nullable로 명시하고, 조건 검사·안전 호출(safe call)·Elvis 연산자를 조합해 값이 없을 때의 동작을 표현한다. 타입 검사와 안전한 cast, 컬렉션의 null 처리, 조기 반환을 결합해 유효한 값만 정상 처리 경로로 보낼 수 있다.

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

## 타입 검사와 smart cast

`is`는 객체가 해당 타입인지, `!is`는 해당 타입이 아닌지 검사해 Boolean 값을 반환한다. `if`나 `when`에서 검사한 뒤 컴파일러가 타입을 좁혀 다루는 것을 smart cast라고 한다. 원문의 `item is String` 분기에서는 `Any` 타입으로 받은 `item`의 `length`에 접근할 수 있다.

```kotlin
fun calculateTotalStringLength(items: List<Any>): Int {
    var totalLength = 0

    for (item in items) {
        totalLength += if (item is String) {
            item.length
        } else {
            0  // Add 0 for non-String items
        }
    }

    return totalLength
}
```

## `as`와 `as?`: 명시적인 cast

Cast는 값을 특정 타입으로 다루도록 지정하는 것이다. `as`는 타입이 맞지 않으면 런타임 오류가 발생한다. 예를 들어 `null`인 `String?`을 `as String`으로 다루려 하면 실패한다. `as?`는 변환에 실패하면 오류 대신 `null`을 반환하므로 이후 null 처리가 필요하다.

```kotlin
fun calculateTotalStringLength(items: List<Any>): Int {
    return items.sumOf { (it as? String)?.length ?: 0 }
}
```

위 코드는 `as?`로 문자열인지 확인하고, `?.`로 문자열 길이를 읽으며, 문자열이 아니면 `?:`로 `0`을 사용한다. `as?`는 타입을 안전하게 다루는 연산자이고, `?.`는 nullable 값의 멤버에 안전하게 접근하는 연산자다.

## 컬렉션에서 null 처리

| 함수 | 동작 | 결과가 없을 때 |
|---|---|---|
| `filterNotNull()` | 기존 컬렉션에서 null 원소 제거 | 빈 리스트 |
| `listOfNotNull()` | 전달받은 값 중 null을 제외해 리스트 생성 | 빈 리스트 |
| `mapNotNull()` | 각 원소를 변환하고 null 결과 제외 | 빈 리스트 |
| `maxOrNull()`·`minOrNull()` | 최댓값·최솟값 조회 | 빈 컬렉션이면 null |
| `singleOrNull { ... }` | 조건을 만족하는 원소가 정확히 하나일 때 반환 | 일치하는 원소가 없거나 여럿이면 null |
| `firstNotNullOfOrNull { ... }` | 원소를 변환하며 처음 나온 non-null 결과 반환 | non-null 결과가 없으면 null |
| `reduceOrNull { ... }` | 원소를 순서대로 처리해 누적 결과 생성 | 빈 컬렉션이면 null |

`singleOrNull()`은 처음 일치하는 원소를 고르는 함수가 아니다. 정확히 하나인지 확인한다. 원문의 활성 구독 연습에서는 활성 구독이 여럿이면 `null`을 반환한다.

```kotlin
data class Subscription(val name: String, val isActive: Boolean)

fun getActiveSubscription(subscriptions: List<Subscription>): Subscription? =
    subscriptions.singleOrNull { it.isActive }
```

`firstNotNullOfOrNull()`은 입력 원소 자체보다 변환 결과의 null 여부를 확인한다. `reduceOrNull()`은 빈 컬렉션을 처리할 수 있지만, nullable 원소를 자동으로 제거하는 함수는 아니다. 원문은 최댓값·최솟값·단일 원소·누적 계산 예제를 null 원소가 없는 컬렉션을 대상으로 설명한다.

## Elvis 연산자로 조기 반환

`?:` 오른쪽에는 기본값뿐 아니라 `return`도 쓸 수 있다. 필요한 값이 없으면 바로 반환하고, 이후에는 확보한 값으로 정상 처리를 이어간다. 원문의 재고 검사 예제는 null과 잘못된 수량을 먼저 제외한다.

```kotlin
fun validateStock(requested: Int?, available: Int?): Int {
    val validRequested = requested ?: return -1
    val validAvailable = available ?: return -1

    if (validRequested < 0) return -1
    if (validRequested > validAvailable) return -1

    return validRequested
}
```

`requested`나 `available`이 null이면 `-1`을 반환한다. 요청 수량이 음수이거나 재고보다 많아도 같은 값을 반환하며, 정상적인 경우에는 요청 수량을 반환한다. 이 검사는 중첩된 조건문 없이 실패 조건을 먼저 처리한다.

## 연습 예제: 타입 검사·기본값·조건부 목록 조합

```kotlin
data class User(val name: String?)

fun getNotificationPreferences(user: Any, emailEnabled: Boolean, smsEnabled: Boolean): List<String> {
    val validUser = user as? User ?: return emptyList()
    val userName = validUser.name ?: "Guest"

    return listOfNotNull(
        "Email Notifications enabled for $userName".takeIf { emailEnabled },
        "SMS Notifications enabled for $userName".takeIf { smsEnabled }
    )
}
```

`user`가 `User`가 아니면 빈 리스트를 반환하고, 이름이 null이면 `Guest`를 사용한다. `takeIf { 조건 }`은 조건이 참이면 원래 값을, 거짓이면 null을 반환한다. `listOfNotNull()`이 그 null을 제외하므로 활성화된 알림 문구만 목록에 들어간다. `mapNotNull()`과 `takeIf`도 같은 원리로 활성 사용자 이름만 골라낼 수 있다.

## See Also

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
