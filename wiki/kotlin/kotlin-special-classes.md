# Kotlin 특수 클래스

> Sources: Kotlin Documentation, 2026-07-01; Kotlin Documentation, 2026-06-29
> Raw: [Open and special classes](../../raw/kotlin/2026-07-01-kotlin-open-and-special-classes.md); [Sealed classes and interfaces](../../raw/kotlin/2026-06-29-kotlin-sealed-classes-and-interfaces.md)
> Updated: 2026-08-12

## Overview

Kotlin의 `sealed class`, `enum class`, inline value class는 모두 class 문법을 특수한 목적에 맞게 제한한다. Sealed class는 상속 가능한 계층을 통제하고, enum class는 서로 구별되는 유한한 값 집합을 표현한다. Inline value class는 하나의 값을 별도 type으로 감싸면서 작은 객체가 만드는 성능 부담을 줄이는 데 초점을 둔다.

## 먼저 보는 선택 기준

| 필요한 모델 | 선택 | 핵심 특징 |
|---|---|---|
| 서로 다른 data와 동작을 가진 제한된 상태 계층 | `sealed class` | compiler가 직접 child를 파악할 수 있는 통제된 상속 |
| 이름으로 구분되는 고정된 값 목록 | `enum class` | 각 enum constant가 enum class의 instance |
| 하나의 값을 별도 type으로 감싸기 | inline value class | class header의 단일 property와 경량 표현 |

`open class`도 특수한 class modifier를 사용하지만 목적은 반대 방향이다. Sealed class가 상속 범위를 통제한다면 open class는 기본적으로 닫힌 구체 class의 상속을 명시적으로 허용한다. 자세한 상속 문법은 [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)에 정리되어 있다.

## Sealed class: 가능한 계층을 통제한다

Sealed class는 제한된 class hierarchy를 표현하는 abstract class의 특수한 형태다. 직접 subclass가 compile time에 알려지므로, `when` expression에서 가능한 경우를 빠짐없이 다루는지 compiler가 검사할 수 있다.

```kotlin
sealed class UIState {
    data object Loading : UIState()
    data class Success(val data: String) : UIState()
    data class Error(val exception: Exception) : UIState()
}

fun render(state: UIState) = when (state) {
    UIState.Loading -> "Loading"
    is UIState.Success -> state.data
    is UIState.Error -> state.exception.message
}
```

각 상태의 data 모양이 다르다는 점이 중요하다. `Success`에는 결과가, `Error`에는 예외가 들어가고, `Loading`처럼 추가 값이 없는 상태는 `data object`로 나타낼 수 있다. 이런 경우 enum보다 sealed hierarchy가 상태별 data와 동작을 표현하기 쉽다.

## Enum class: 유한한 값 목록을 표현한다

Enum class는 서로 구별되는 유한한 값을 class로 표현한다. Enum constant는 단순 문자열이 아니라 해당 enum class의 instance이며, class 이름과 constant 이름을 함께 사용해 접근한다.

```kotlin
enum class State {
    IDLE,
    RUNNING,
    FINISHED
}

val state = State.RUNNING
```

값 집합이 고정되어 있으므로 `when`과 자연스럽게 결합된다.

```kotlin
val message = when (state) {
    State.IDLE -> "It's idle"
    State.RUNNING -> "It's running"
    State.FINISHED -> "It's finished"
}
```

Enum class도 일반 class처럼 property와 member function을 가질 수 있다. Constant마다 constructor property를 초기화하고, constant 목록 뒤에 member를 선언할 때는 semicolon으로 구분한다.

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF);

    fun containsRed() = (rgb and 0xFF0000) != 0
}
```

## Inline value class: 값을 별도 type으로 감싼다

Inline value class는 짧게 사용되는 작은 값 객체의 성능 부담을 줄이기 위한 class다. 공식 tour의 JVM 예제에서는 `value` keyword와 `@JvmInline` annotation을 함께 쓰며, class header에서 하나의 property를 초기화해야 한다.

```kotlin
@JvmInline
value class Email(val address: String)

fun sendEmail(email: Email) {
    println("Sending email to ${email.address}")
}
```

`String`을 그대로 받는 대신 `Email` type으로 감싸면 function이 기대하는 값의 의미가 type에 드러난다. 동시에 compiler가 representation을 최적화할 수 있어 단순 wrapper 객체를 계속 만드는 부담을 줄이는 목적에 맞는다.

## Sealed class와 enum class 비교

| 질문 | `sealed class` | `enum class` |
|---|---|---|
| 각 경우가 서로 다른 data 구조를 가질 수 있는가? | 가능 | 모든 constant가 같은 enum class 구조를 공유 |
| 각 경우가 별도 class인가? | child class 또는 object | enum constant |
| 대표 용도 | 결과·화면·작업 상태의 계층 | 방향·모드·고정 상태 코드 |
| `when`과 결합하는 이유 | 알려진 child 계층을 빠짐없이 처리 | 고정된 constant를 빠짐없이 처리 |

다음처럼 판단하면 된다.

```text
값의 이름만 다르면 enum class
상태마다 담는 data나 구현이 다르면 sealed class
하나의 원시적 값을 의미 있는 별도 type으로 감싸면 inline value class
```

## See Also

- [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
- [Kotlin Objects](kotlin-objects.md)
- [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 클래스와 데이터 클래스](kotlin-classes-and-data-classes.md)
