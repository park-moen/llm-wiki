# Kotlin 제어 흐름

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown
> Raw: [Control flow](../../raw/kotlin/control-flow-3.md); [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md)
> Updated: 2026-10-15

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

## 분기 표현식에도 타입이 있다

`if`·`when`을 값으로 사용하면 선택한 분기의 결과가 전체 표현식의 값이 된다. compiler는 가능한 분기의 타입을 함께 고려한다. `throw`처럼 정상 결과를 만들지 않는 분기는 `Nothing`으로 설명되므로 정상 값을 반환하는 다른 분기와 함께 사용할 수 있다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md)

`if`의 조건이 실행되고 선택된 본문만 평가되는 것, `&&`·`||`가 단축 평가하는 것도 실행 순서의 일부다. 분기 안에 부수 효과가 있다면 선택되지 않은 쪽의 함수가 호출된다고 기대하지 않는다. `when`의 빠짐없는 분기 검사와 sealed 계층은 [특수 클래스](kotlin-special-classes.md)에서 연결한다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md)

## 직접 작성하는 연습: 수량별 처리 상태 분류

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

`classify()`에 when을 작성하세요. 음수면 invalid, 0이면 empty, 양수면 ready를 반환합니다. for로 모든 수량을 순회하세요.

### 시작 코드

```kotlin
fun classify(quantity: Int): String = TODO("when으로 수량 분류")

fun main() {
    val quantities = listOf(-1, 0, 2)
    // TODO: quantities를 for로 순회하며 classify 결과 출력
}
```

### 예상 출력

```text
invalid
empty
ready
```

> [!tip]- 힌트
> 조건을 직접 쓰는 when에는 주제를 넣지 않습니다.

> [!success]- 정답 확인
> ```kotlin
> fun classify(quantity: Int): String = when {
>     quantity < 0 -> "invalid"
>     quantity == 0 -> "empty"
>     else -> "ready"
> }
>
> fun main() {
>     val quantities = listOf(-1, 0, 2)
>     for (quantity in quantities) {
>         println(classify(quantity))
>     }
> }
> ```

### 한 번 더 바꿔 보기

양수 중 1만 low로 분류하도록 분기를 추가하세요.

## See Also

- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin Null Safety](kotlin-null-safety.md)

- 학습 순서 이전: [Kotlin 숫자의 JVM 표현과 boxing](kotlin-numeric-representation-and-boxing.md)
- 학습 순서 다음: [Kotlin 함수](kotlin-functions.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
