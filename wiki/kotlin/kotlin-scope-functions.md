# Kotlin Scope Functions

> Sources: Kotlin Documentation, Unknown
> Raw: [Scope functions](../../raw/kotlin/kotlin-intermediate-scope-functions.md)
> Updated: 2026-08-10

## Overview

Kotlin의 scope function은 객체 주위에 임시 scope를 만들고 lambda를 실행한다. `let`, `apply`, `run`, `also`, `with`는 객체를 `this` 또는 `it`으로 참조하며, 객체 자체 또는 lambda 실행 결과를 반환한다. 선택할 때는 객체를 어떻게 참조하는지와 어떤 값을 반환해야 하는지를 함께 판단한다.

## 두 가지 선택 축

Scope function은 두 질문으로 구분할 수 있다.

1. 임시 scope 안에서 객체를 `this`로 사용할 것인가, `it`으로 사용할 것인가?
2. 호출한 객체 자체를 반환할 것인가, lambda의 결과를 반환할 것인가?

| Function | 객체 접근 | 반환값 | 대표 용도 |
|----------|-----------|--------|-----------|
| `let` | `it` | Lambda result | null 검사 후 결과 사용 |
| `apply` | `this` | 객체 `x` | 생성 시 객체 초기화 |
| `run` | `this` | Lambda result | 객체를 사용해 결과 계산 |
| `also` | `it` | 객체 `x` | logging·debugging 같은 부가 작업 |
| `with` | `this` | Lambda result | 한 객체의 여러 함수 호출 |

`with`는 나머지와 달리 extension function이 아니므로 리시버 객체를 인자로 전달한다.

## `let`: nullable 값 처리와 결과 반환

`let`은 객체를 `it`으로 전달하고 lambda 결과를 반환한다. Safe call과 결합하면 nullable 값이 존재할 때만 작업을 실행하고 그 결과를 보존할 수 있다.

```kotlin
val address: String? = getNextAddress()
val confirm = address?.let {
    sendNotification(it)
}
```

`address`가 `null`이 아니면 `sendNotification()`의 결과가 `confirm`에 들어가고, `null`이면 전체 표현식도 `null`이 된다.

## `apply`: 객체를 생성하면서 설정

`apply`는 scope 안에서 객체를 `this`로 사용하고, 실행 후 같은 객체를 반환한다. 따라서 생성 직후 프로퍼티 설정과 초기 작업을 한곳에 모으는 데 적합하다.

```kotlin
val client = Client().apply {
    token = "asdf"
    connect()
    authenticate()
}
```

## `run`: 객체 문맥에서 결과 계산

`run`도 객체를 `this`로 다루지만 객체가 아니라 lambda의 마지막 결과를 반환한다. 특정 시점에 객체의 여러 동작을 묶어 실행한 후 계산 결과를 얻을 때 사용할 수 있다.

```kotlin
val result: String = client.run {
    connect()
    authenticate()
    getData()
}
```

## `also`: 흐름을 유지하며 부가 작업 실행

`also`는 객체를 `it`으로 전달하고 같은 객체를 반환한다. 주 흐름의 값을 바꾸지 않은 채 logging, debugging 또는 side effect를 중간에 삽입하기 좋다.

```kotlin
val result = medals
    .map { it.uppercase() }
    .also { println(it) }
    .filter { it.length > 4 }
    .also { println(it) }
    .reversed()
```

## `with`: 한 객체의 여러 멤버 호출

`with`에는 리시버 객체와 lambda를 각각 인자로 전달한다. scope 안에서는 해당 객체의 멤버를 `this` 한정자 없이 호출할 수 있고, lambda 결과를 반환한다.

```kotlin
with(canvas) {
    text(10, 10, "Foo")
    rect(20, 30, 100, 50)
    circ(40, 60, 25)
}
```

## 선택 원칙

Scope function은 단순히 코드를 짧게 만들기 위한 문법이 아니다. 객체를 설정하는 것이 목적이면 `apply`, 결과 계산이면 `run`이나 `let`, 기존 chain을 유지하며 부가 동작을 넣는다면 `also`, 한 객체의 여러 동작을 묶는다면 `with`가 의도를 더 직접적으로 드러낸다.

## See Also

- [Kotlin 확장 함수](kotlin-extension-functions.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin Null Safety](kotlin-null-safety.md)
