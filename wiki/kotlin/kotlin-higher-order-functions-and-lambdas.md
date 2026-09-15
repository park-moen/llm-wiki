# Kotlin 고차 함수와 람다

> Sources: Kotlin Documentation, Unknown
> Raw: [Higher-order functions and lambdas](../../raw/kotlin/higher-order-functions-and-lambdas.md); [Functions](../../raw/kotlin/functions-3.md); [Lambda expressions with receiver](../../raw/kotlin/kotlin-intermediate-lambdas-with-receiver.md)
> Updated: 2026-08-10

## Overview

Kotlin에서 함수는 변수나 자료구조에 저장하고, 다른 함수에 인자로 전달하거나 반환값으로 돌려줄 수 있는 일급 값(first-class value)이다. 이를 함수 타입(function type), 고차 함수(higher-order function), 람다 표현식(lambda expression), 익명 함수(anonymous function), 호출 가능 참조(callable reference) 같은 언어 요소로 표현한다.

## 고차 함수

고차 함수는 함수를 매개변수로 받거나 함수를 반환하는 함수다. 컬렉션의 `fold`는 초기 누산값과 결합 함수를 받아 각 원소를 순서대로 누산값에 결합하는 대표적인 예다.

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

여기서 `combine`의 타입 `(R, T) -> R`은 `R`과 `T`를 받아 `R`을 반환하는 함수다. 람다뿐 아니라 `Int::times` 같은 함수 참조도 고차 함수의 인자로 전달할 수 있다.

## 함수 타입

함수 타입은 매개변수 타입을 괄호 안에 쓰고 화살표 뒤에 반환 타입을 둔다.

- `(A, B) -> C`: `A`, `B`를 받아 `C`를 반환한다.
- `() -> A`: 매개변수가 없고 `A`를 반환한다.
- `A.(B) -> C`: `A`를 리시버(receiver)로 삼고 `B`를 받아 `C`를 반환한다.
- `suspend () -> Unit`: 일시 중단 함수 타입(suspending function type)이다.
- `((Int, Int) -> Int)?`: 함수 타입 전체가 nullable임을 뜻한다.

함수 타입의 `Unit` 반환 타입은 생략할 수 없다. 화살표 표기는 오른쪽 결합(right-associative)이므로 `(Int) -> (Int) -> Unit`은 `(Int) -> ((Int) -> Unit)`과 같고, `((Int) -> (Int)) -> Unit`과는 다르다. 매개변수 이름을 `(x: Int, y: Int) -> Point`처럼 넣어 의미를 문서화하거나 `typealias`로 별칭을 붙일 수도 있다.

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

## 함수 타입 인스턴스 생성과 호출

함수 타입의 값은 다음 방식으로 만들 수 있다.

- 람다 표현식: `{ a, b -> a + b }`
- 익명 함수: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`
- 함수·프로퍼티·생성자의 호출 가능 참조: `::isOdd`, `String::toInt`, `List<Int>::size`, `::Regex`
- 함수 타입 인터페이스를 구현하고 `operator fun invoke`를 정의한 클래스의 인스턴스

함수 타입의 값은 `f.invoke(x)` 또는 축약형 `f(x)`로 호출한다. 리시버가 있는 함수 타입은 리시버를 첫 번째 인자로 넘기거나 확장 함수처럼 호출할 수 있다.

```kotlin
val intPlus: Int.(Int) -> Int = Int::plus

intPlus.invoke(1, 1)
intPlus(1, 2)
2.intPlus(3)
```

리터럴이 아닌 함수 값에서는 `(A, B) -> C`와 `A.(B) -> C`가 상호 교환 가능하여, 리시버가 첫 번째 매개변수 역할을 할 수 있다. 다만 확장 함수 참조로 변수를 초기화하더라도 기본 추론 결과는 리시버 없는 함수 타입이며, 리시버 타입을 원하면 변수 타입을 명시해야 한다.

## 람다 표현식 문법

람다는 중괄호로 감싸며, 매개변수 선언은 중괄호 안에서 `->` 앞에 둔다. 타입을 추론할 수 있으면 매개변수 타입을 생략할 수 있다. 추론된 반환 타입이 `Unit`이 아니면 본문의 마지막 표현식이 반환값이다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

함수의 마지막 매개변수가 함수라면 대응하는 람다를 괄호 밖으로 옮기는 후행 람다(trailing lambda) 문법을 사용할 수 있다.

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

람다가 유일한 인자라면 호출 괄호도 생략할 수 있다.

```kotlin
run { println("...") }
```

매개변수가 하나이고 시그니처를 추론할 수 있으면 매개변수 선언과 `->`를 생략하고 암시적 이름 `it`을 쓸 수 있다. 사용하지 않는 람다 매개변수는 이름 대신 `_`로 표시한다.

```kotlin
ints.filter { it > 0 }
map.forEach { (_, value) -> println("$value!") }
```

## 람다 활용 패턴

람다는 컬렉션 처리 함수의 동작을 인자로 전달하는 데 사용할 수 있다. `.filter()`는 각 원소에 predicate를 적용해 `true`를 반환한 원소만 유지하고, `.map()`은 transform 함수를 적용해 원소를 변환한다.

```kotlin
val positives = numbers.filter { x -> x > 0 }
val doubled = numbers.map { x -> x * 2 }
```

함수에서 람다를 반환하려면 반환되는 람다의 함수 타입을 선언한다.

```kotlin
fun toSeconds(time: String): (Int) -> Int = when (time) {
    "hour" -> { value -> value * 60 * 60 }
    "minute" -> { value -> value * 60 }
    else -> { value -> value }
}
```

람다 리터럴 뒤에 호출 괄호와 인자를 붙여 즉시 호출할 수도 있다.

```kotlin
val upper = { text: String -> text.uppercase() }("hello")
```

## 반환과 익명 함수

람다는 마지막 표현식을 암시적으로 반환하며, 명시적으로 반환하려면 `return@filter` 같은 한정 반환(qualified return)을 사용할 수 있다.

```kotlin
ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter
}
```

익명 함수는 이름이 빠진 일반 함수 형태이며 반환 타입을 명시할 수 있다. 표현식 본문의 반환 타입은 추론할 수 있지만 블록 본문에서는 명시하지 않으면 `Unit`으로 간주된다. 익명 함수는 함수 호출의 괄호 안에 전달해야 하며, 후행 람다 축약은 람다 표현식에만 적용된다.

레이블 없는 `return`의 동작도 다르다. 람다 내부의 `return`은 둘러싼 함수에서 반환하는 반면, 익명 함수 내부의 `return`은 그 익명 함수 자체에서 반환한다.

## 클로저

람다와 익명 함수는 바깥 스코프의 변수를 포함하는 클로저(closure)에 접근할 수 있으며, 캡처한 변수를 변경할 수도 있다.

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

## 리시버가 있는 함수 리터럴

`A.(B) -> C` 형태의 함수 타입은 리시버가 있는 함수 리터럴(function literal with receiver)로 인스턴스화할 수 있다. 호출 시 제공된 리시버는 본문 안에서 암시적 `this`가 되므로, 한정자 없이 리시버의 멤버에 접근할 수 있다.

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }
val anonymousSum = fun Int.(other: Int): Int = this + other
```

리시버 타입을 문맥에서 추론할 수 있으면 람다도 리시버가 있는 함수 리터럴로 사용할 수 있다. 이 형태는 type-safe builder에 쓰인다.

```kotlin
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```

리시버가 있는 람다를 고차 함수의 매개변수로 받으면 lambda 본문에서 리시버의 프로퍼티와 멤버 함수를 한정자 없이 호출할 수 있다. 다음 `render()`에서는 `Canvas`가 `block`의 리시버다.

```kotlin
fun render(block: Canvas.() -> Unit): Canvas {
    val canvas = Canvas()
    canvas.block()
    return canvas
}

render {
    drawCircle()
    drawSquare()
}
```

이 문법은 API, UI framework, configuration builder 같은 DSL(domain-specific language)을 구성할 때 유용하다. 예를 들어 `Menu.() -> Unit`을 받는 builder에서는 호출부가 `Menu`의 내부 문맥처럼 `item()`을 직접 호출할 수 있다.

```kotlin
fun menu(name: String, init: Menu.() -> Unit): Menu {
    val menu = Menu(name)
    menu.init()
    return menu
}

val mainMenu = menu("Main Menu") {
    item("Home")
    item("Settings")
    item("Exit")
}
```

Kotlin standard library의 `buildList()`와 `buildString()`도 이 설계 패턴의 예다. 리시버가 있는 람다는 type-safe builder와 결합해 타입 문제를 런타임이 아니라 컴파일 시점에 찾는 DSL을 만드는 데 사용할 수 있다.

## See Also

- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 확장 함수](kotlin-extension-functions.md)
- [Kotlin Scope Functions](kotlin-scope-functions.md)
