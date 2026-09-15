# Kotlin 제어 흐름

> Sources: Kotlin Documentation, Unknown
> Raw: [Control flow](../../raw/kotlin/control-flow-3.md)
> Updated: 2026-08-10

## Overview

Kotlin은 조건 분기에 `if`와 `when`, 반복에 `for`, `while`, `do-while`을 사용한다. `if`와 `when`은 실행 흐름을 나누는 문장으로도, 값을 생성하는 표현식으로도 사용할 수 있다. 범위(range)는 `for` 반복에서 순회할 값을 표현한다.

## if 표현식

Kotlin에는 `condition ? then : else` 형태의 삼항 연산자가 없다. 대신 `if` 자체가 값을 반환하는 표현식이 될 수 있다.

```kotlin
val max = if (a > b) a else b
```

각 분기의 실행문이 하나라면 중괄호를 생략할 수 있다.

## when 표현식

`when`은 여러 분기를 다룰 때 사용한다. 각 분기는 조건과 동작을 `->`로 구분하며, 위에서부터 차례로 검사하여 처음 만족하는 분기만 실행한다.

```kotlin
val result = when (obj) {
    "1" -> "One"
    "Hello" -> "Greeting"
    else -> "Unknown"
}
```

`when`은 주제(subject)를 두거나 생략할 수 있다. 주제를 생략하면 각 분기에 Boolean 표현식을 작성하며 `else` 분기가 필요하다. 주제가 있는 형태는 가능한 경우가 모두 처리됐는지 Kotlin이 검사하는 데 도움이 된다.

## 범위

- `1..4`: 끝값을 포함하는 범위
- `1..<4`: 끝값을 포함하지 않는 범위
- `4 downTo 1`: 역순 범위
- `1..5 step 2`: 지정한 간격으로 증가하는 범위

같은 문법은 `Char` 범위에도 사용할 수 있다.

## for 반복

`for`는 범위나 컬렉션의 값을 순회한다. 반복자와 대상은 괄호 안에서 `in`으로 연결한다.

```kotlin
for (number in 1..5) {
    print(number)
}

for (cake in cakes) {
    println(cake)
}
```

## while과 do-while

`while`은 조건이 참인 동안 코드 블록을 반복한다. `do-while`은 블록을 먼저 실행한 뒤 조건을 검사하므로 본문이 조건 검사보다 앞서 실행된다.

```kotlin
while (cakesEaten < 3) {
    cakesEaten++
}

do {
    cakesBaked++
} while (cakesBaked < cakesEaten)
```

## See Also

- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin Null Safety](kotlin-null-safety.md)
