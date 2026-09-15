# Kotlin 함수

> Sources: Kotlin Documentation, Unknown
> Raw: [Functions](../../raw/kotlin/functions-3.md)
> Updated: 2026-08-10

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

## 람다와의 관계

람다는 함수를 값으로 표현해 다른 함수에 전달하거나 함수에서 반환하고, 별도로 호출할 수 있게 한다. 함수 타입과 람다 문법, 후행 람다, 익명 함수에 관한 상세 규칙은 별도 문서에서 다룬다.

## See Also

- [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin 클래스와 데이터 클래스](kotlin-classes-and-data-classes.md)
