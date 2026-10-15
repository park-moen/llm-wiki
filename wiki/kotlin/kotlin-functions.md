# Kotlin 함수

> Sources: Kotlin Documentation, Unknown; Kotlin Documentation, 2026-09-02; Kotlin Language Specification, Unknown
> Raw: [Functions](../../raw/kotlin/functions-3.md); [Null safety — intermediate tour](../../raw/kotlin/2026-09-02-kotlin-intermediate-null-safety.md); [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [Kotlin language specification — Overload resolution](../../raw/kotlin/kotlin-spec-overload-resolution.md); [Calling Kotlin from Java](../../raw/kotlin/kotlin-java-to-kotlin-interop.md)
> Updated: 2026-10-15

## Overview

Kotlin 함수는 `fun` 키워드로 선언한다. 매개변수에는 타입을 지정하고, 반환 타입은 매개변수 목록 뒤에 `:`으로 표시한다. 이름 있는 인자, 기본값, `Unit`, 단일 표현식 함수와 조기 반환을 이용해 호출부와 구현을 간결하게 구성할 수 있다.

## 함수 선언

함수 본문은 중괄호 안에 작성하며 `return`으로 값을 반환하거나 실행을 끝낸다.

```kotlin
fun sum(x: Int, y: Int): Int {
    return x + y
}
```

매개변수는 괄호 안에서 쉼표로 구분하며 각 매개변수에 타입이 필요하다. Kotlin 코딩 규칙은 함수 이름을 소문자로 시작하고 밑줄 없는 camel case로 작성할 것을 권장한다.

## 이름 있는 인자와 기본값

호출할 때 `name = value` 형태로 매개변수 이름을 쓰면 인자의 의미가 분명해지고 선언 순서와 다른 순서로 전달할 수 있다.

```kotlin
printMessageWithPrefix(prefix = "Log", message = "Hello")
```

매개변수 타입 뒤에 `=`을 써서 기본값을 선언하면 호출 시 해당 인자를 생략할 수 있다. 기본값이 있는 매개변수를 하나 건너뛴 뒤에는 이후의 모든 인자에 이름을 붙여야 한다.

```kotlin
fun printMessageWithPrefix(message: String, prefix: String = "Info") {
    println("[$prefix] $message")
}
```

## Unit 반환

유용한 값을 반환하지 않는 함수의 반환 타입은 `Unit`이다. 이런 함수는 반환 타입과 `return`을 명시하지 않아도 된다.

```kotlin
fun printMessage(message: String) {
    println(message)
}
```

## 단일 표현식 함수

함수 본문이 하나의 표현식이면 중괄호와 `return` 대신 `=`을 사용할 수 있다. 이때 Kotlin이 반환 타입을 추론할 수 있지만, 읽기 쉬운 코드를 위해 반환 타입을 명시할 수도 있다.

```kotlin
fun sum(x: Int, y: Int): Int = x + y
```

반대로 중괄호 본문에서는 반환 타입이 `Unit`이 아닌 한 반환 타입을 선언해야 한다.

## 조기 반환

`return`은 함수의 나머지 처리를 중단하는 가드(guard)로 사용할 수 있다. 유효하지 않은 조건을 먼저 검사해 반환하면 정상 처리 경로를 뒤에 둘 수 있다.

```kotlin
fun registerUser(username: String): String {
    if (username in registeredUsernames) {
        return "Username already taken."
    }
    registeredUsernames.add(username)
    return "User registered successfully."
}
```

### nullable 값 확보와 조기 반환

`val value = nullableValue ?: return ...` 형태로 필요한 값이 없을 때 함수를 끝낼 수 있다. 정상 경로에서는 확보한 값을 사용하므로 null 검사 중첩을 줄일 수 있다. 안전한 cast와 조기 반환, 재고 수량 검사 예제는 [Kotlin Null Safety](kotlin-null-safety.md)에 정리했다.

## 람다와의 관계

람다는 함수를 값으로 표현해 다른 함수에 전달하거나 함수에서 반환하고, 별도로 호출할 수 있게 한다. 함수 타입과 람다 문법, 후행 람다, 익명 함수에 관한 상세 규칙은 별도 문서에서 다룬다.

## 호출할 함수와 인자 평가 순서는 별개다

overload resolution은 호출에 맞는 함수 후보를 고르는 과정이다. 그다음 receiver와 명시적으로 전달한 인자를 평가하는 순서를 구분한다. 명시한 인자는 호출부에 나타난 순서대로 평가되고, 생략한 기본 인자 식은 필요한 호출에서 평가된다. 이름 있는 인자로 선언 순서와 다르게 전달해도 부수 효과의 평가 순서를 매개변수 선언 순서라고 추측하면 안 된다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Kotlin language specification — Overload resolution](../../raw/kotlin/kotlin-spec-overload-resolution.md)

다음은 평가 순서를 드러내는 학습 예제다.

```kotlin
fun mark(label: String): String {
    println(label)
    return label
}

fun combine(first: String, second: String) = first + second

fun main() {
    combine(second = mark("second"), first = mark("first"))
}
```

출력은 `second` 다음 `first`다. 기본 인자는 값을 생략한 호출마다 식을 평가하는 것이며, 선언 시 한 번 만든 객체를 모든 호출이 반드시 공유하는 모델이 아니다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

Kotlin에서 인자를 생략할 수 있다고 Java에도 같은 호출 문법이 생기지는 않는다. `@JvmOverloads` 등 Java에서 보이는 overload 생성은 [Java 연동](kotlin-java-interop-and-reflection.md)에서 구분한다. [Calling Kotlin from Java](../../raw/kotlin/kotlin-java-to-kotlin-interop.md)

## 직접 작성하는 연습: 기본값이 있는 금액 함수

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

총액을 계산하는 함수를 작성하세요. 배송비 기본값은 0이고, 배송비가 필요한 호출에서는 이름 있는 인자를 사용하세요.

### 시작 코드

```kotlin
fun calculateTotal(price: Int, quantity: Int, shipping: Int /* TODO: 기본값 0 지정 */): Int {
    return TODO("총액 계산")
}

fun main() {
    println(calculateTotal(price = 10, quantity = 2))
    println(calculateTotal(price = 10, quantity = 2, shipping = 3))
}
```

### 예상 출력

```text
20
23
```

> [!tip]- 힌트
> 매개변수의 기본값은 타입 뒤에 = 값으로 지정합니다.

> [!success]- 정답 확인
> ```kotlin
> fun calculateTotal(price: Int, quantity: Int, shipping: Int = 0): Int {
>     return price * quantity + shipping
> }
>
> fun main() {
>     println(calculateTotal(price = 10, quantity = 2))
>     println(calculateTotal(price = 10, quantity = 2, shipping = 3))
> }
> ```

### 한 번 더 바꿔 보기

같은 함수를 = 표현식 하나로 다시 작성하세요.

## See Also

- [Kotlin Null Safety](kotlin-null-safety.md)

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)

- 학습 순서 이전: [Kotlin 제어 흐름](kotlin-control-flow.md)
- 학습 순서 다음: [Kotlin Null Safety](kotlin-null-safety.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
