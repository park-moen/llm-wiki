# Kotlin Objects

> Sources: Kotlin Documentation, 2026-07-01; Kotlin Documentation, 2025-02-23; Kotlin Documentation, 2026-06-29; Kotlin Documentation, Unknown
> Raw: [Objects](../../raw/kotlin/2026-07-01-kotlin-intermediate-objects.md); [Object declarations and expressions](../../raw/kotlin/2025-02-23-kotlin-object-declarations-and-expressions.md); [Sealed classes and interfaces](../../raw/kotlin/2026-06-29-kotlin-sealed-classes-and-interfaces.md); [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency.md)
> Updated: 2026-10-02

## Overview

Kotlin의 `object`를 처음 배울 때는 “class와 instance를 한 번에 만든다”라고 이해하면 된다. `class`는 여러 instance를 만들기 위한 설계도이고, `object` declaration은 이름이 붙은 유일한 instance까지 즉시 선언한다. `data object`는 값이 필요 없는 하나의 상태를 읽기 좋게 표현하며, `companion object`는 특정 class에 연결된 생성 function·상수·공유 기능을 둔다.

## 먼저 알아야 할 class와 instance

`class`는 객체를 만들기 위한 설계도다. 같은 class에서 서로 다른 값을 가진 instance를 여러 개 만들 수 있다.

```kotlin
class User(val name: String)

val minsu = User("민수")
val jiyoung = User("지영")
```

여기에는 `User`라는 설계도 하나와 `minsu`, `jiyoung`이라는 instance 두 개가 있다. Kotlin에서는 Java와 달리 instance를 만들 때 `new`를 쓰지 않고 `User(...)`처럼 constructor를 호출한다.

반면 프로그램에서 정말 하나만 있으면 되는 대상은 매번 `Something()`으로 만들 이유가 없다. 이때 `object` declaration을 사용할 수 있다.

```text
class User                 object AppLogger
├── User("민수")           └── AppLogger 하나뿐
└── User("지영")
```

## Object declaration

`object` keyword를 사용하면 이름이 있는 class와 그 class의 유일한 instance를 함께 선언한다. 이런 형태를 singleton이라고 한다.

```kotlin
object DoAuth {
    fun takeParams(username: String, password: String) {
        println("input Auth parameters = $username:$password")
    }
}

DoAuth.takeParams("user", "password")
```

`DoAuth()`로 instance를 만드는 과정이 없다. `DoAuth` 자체가 이미 사용할 instance이므로 `DoAuth.takeParams(...)`처럼 이름으로 바로 접근한다. Object declaration에는 constructor가 없다.

일반 class와 호출 모양을 비교하면 차이가 선명하다.

```kotlin
class AuthService {
    fun login() = println("로그인")
}

object AuthManager {
    fun login() = println("로그인")
}

val service = AuthService() // class: instance를 만든다
service.login()

AuthManager.login()         // object: 이미 존재하는 하나를 쓴다
```

Object는 처음 접근할 때 생성되는 lazy 방식이며, Kotlin이 thread-safe하게 생성한다. 또한 class나 interface를 상속할 수 있다.

```kotlin
interface Auth {
    fun takeParams(username: String, password: String)
}

object DoAuth : Auth {
    override fun takeParams(username: String, password: String) = Unit
}
```

### thread-safe 생성과 내부 상태는 별개다

“thread-safe하게 생성된다”는 말은 여러 thread가 동시에 처음 접근하더라도 singleton instance 생성 자체를 Kotlin이 안전하게 처리한다는 뜻이다. Object 안에 둔 모든 `var`와 연산까지 자동으로 안전해진다는 뜻은 아니다.

```kotlin
object Counter {
    var value = 0
    fun increase() {
        value++
    }
}
```

여러 thread나 coroutine이 동시에 `increase()`를 실행하면 `value++`는 별도 동기화가 필요할 수 있다. 초보 단계에서는 object 안의 변경 가능한 전역 상태를 최소화하고, 먼저 상수나 상태 없는 공통 function처럼 단순한 용도에 사용하는 편이 이해하기 쉽다.

## Data object

`data object`는 object declaration에 `data`를 붙인 형태다. 핵심 용도는 **추가 데이터가 필요 없는 하나의 상태나 선택지**를 이름 그대로 표현하는 것이다.

```kotlin
sealed interface UiState

data object Loading : UiState
data class Success(val message: String) : UiState
data class Error(val reason: String) : UiState
```

이 코드는 다음처럼 읽는다.

- `Loading`: 로딩 중이라는 사실만 필요하다. 안에 담을 추가 값이 없으므로 instance가 하나면 충분하다.
- `Success("완료")`: 성공할 때는 message가 달라질 수 있으므로 `data class` instance를 만든다.
- `Error("네트워크 오류")`: 오류마다 reason이 달라질 수 있으므로 `data class` instance를 만든다.

```kotlin
fun render(state: UiState) {
    when (state) {
        Loading -> println("로딩 중")
        is Success -> println(state.message)
        is Error -> println(state.reason)
    }
}
```

`Loading`은 이미 하나뿐인 값이므로 `Loading()`이 아니라 `Loading`이라고 쓴다. `Success`와 `Error`는 호출할 때마다 서로 다른 값을 담은 instance를 만들 수 있으므로 괄호를 사용한다.

Data object에는 `toString()`, `equals()`, `hashCode()`가 자동 제공된다. 출력하면 기본 hash 형태 대신 이름이 나오며, 비교할 때는 구조적 동등성 연산자 `==`를 사용한다.

```kotlin
println(Loading)            // Loading
println(Loading == Loading) // true
```

Data class와 달리 `copy()`와 `componentN()`은 제공되지 않는다. Instance가 하나뿐인 singleton을 복사하거나, 생성자에 담긴 데이터가 없는 object를 구조 분해하는 것은 의미가 없기 때문이다.

### object와 data object 중 무엇을 고를까

- 하나뿐인 관리자나 상태 없는 공통 기능에는 일반 `object`가 자연스럽다.
- `sealed class/interface` 안에서 `Loading`, `EndOfFile`, `NotFound`처럼 값 없는 상태를 나타내고 출력·비교할 일이 있으면 `data object`가 자연스럽다.
- 이름은 `data object`이지만, body의 모든 property를 data class처럼 비교해 준다는 뜻은 아니다.


## Companion object

Companion object는 **특정 class에 딸린 하나의 object**다. 일반 instance마다 새로 생기는 부분이 아니라 class마다 하나만 존재하며, 그 안의 property와 function은 모든 instance에서 공유된다.

다음처럼 class와 companion object를 분리해서 생각하면 이해하기 쉽다.

```text
Temperature class
├── Temperature instance
├── Temperature instance
└── companion object 하나
    └── fromFahrenheit()
```

일반 instance는 `Temperature(...)`를 호출할 때마다 만들어질 수 있다. 반면 companion object는 `Temperature` class에 연결된 하나의 공유 object이며 class가 처음 참조될 때 생성된다.

### 왜 사용하는가

Instance member는 이미 만들어진 instance의 상태를 다룬다. 그러나 instance를 만들기 전부터 class와 관련된 function이 필요할 수도 있다. 공식 문서의 예제는 Fahrenheit 값을 받아 `Temperature` instance를 생성하는 function을 companion object에 둔다.

```kotlin
data class Temperature(val celsius: Double) {
    val fahrenheit: Double = celsius * 9 / 5 + 32

    companion object {
        fun fromFahrenheit(fahrenheit: Double): Temperature =
            Temperature((fahrenheit - 32) * 5 / 9)
    }
}

val temperature = Temperature.fromFahrenheit(90.0)
```

이 호출은 다음 순서로 읽을 수 있다.

1. `Temperature`는 class 이름이다.
2. `fromFahrenheit()`는 companion object 안의 function이다.
3. 이 function이 Fahrenheit 값을 Celsius로 변환한다.
4. 변환된 값으로 일반 `Temperature` instance를 만들어 반환한다.

즉 `fromFahrenheit()`는 특별한 constructor 문법이 아니라, companion object에 들어 있는 function이 `Temperature(...)` constructor를 호출해 instance를 반환하는 구조다.

이처럼 객체를 만드는 function을 factory 함수라고 부른다. Factory의 이름과 생성자 접근 범위를 어떻게 정할지는 [Kotlin factory 함수와 객체 생성 규칙](kotlin-factory-functions-and-construction-rules.md)에서 다룬다.

### Instance member와 비교

```kotlin
val temp = Temperature.fromFahrenheit(90.0)

// class 이름을 통해 companion object의 function 호출
Temperature.fromFahrenheit(90.0)

// 생성된 instance를 통해 instance property 접근
println(temp.celsius)
println(temp.fahrenheit)
```

- `Temperature.fromFahrenheit(...)`: 아직 특정 instance가 없어도 class 이름으로 호출한다.
- `temp.celsius`: 이미 만들어진 `temp` instance의 값을 읽는다.

판단 기준은 특정 instance의 상태가 필요한지 여부다. 필요하면 instance member에 두고, instance를 만들기 전부터 class와 연결해 호출해야 한다면 companion object를 고려한다.

### 이름과 접근 방식

Companion object의 member는 companion object의 이름을 직접 쓰지 않고 class 이름으로 접근할 수 있다.

Companion object는 이름을 생략할 수 있으며, 생략하면 기본 이름은 `Companion`이다. 이름을 지정할 때는 다음처럼 `companion object` 뒤에 쓴다.

```kotlin
class BigBen {
    companion object Bonger {
        fun ring() = Unit
    }
}

BigBen.ring()
```

위 코드에서 companion object의 이름은 `Bonger`지만 호출은 `BigBen.ring()`처럼 class 이름으로 한다.

### 기억할 문장

**Companion object는 instance 안에 들어가는 복사본이 아니라, class에 연결된 하나의 공유 object다.**

다른 언어의 `static` member처럼 보이지만 완전히 같은 개념은 아니다. Companion object의 member는 실제 companion object instance에 속하며, companion object 자체가 interface를 구현할 수도 있다. Kotlin 초보 단계에서는 “class 이름으로 호출할 수 있는, class 전용 공유 공간”으로 기억하면 충분하다.

## Object expression: 한 번만 쓸 익명 객체

추가 공식 문서에는 object declaration과 이름이 비슷한 **object expression**도 나온다. 이름이 붙은 singleton을 만드는 object declaration과 달리, object expression은 그 자리에서 한 번 사용할 익명 객체(anonymous object)를 만든다.

```kotlin
val greeting = object {
    val hello = "안녕"
    val target = "Kotlin"

    override fun toString() = "$hello, $target"
}

println(greeting) // 안녕, Kotlin
```

두 문법은 생성 시점도 다르다.

- Object declaration: 처음 접근할 때 lazy하게 초기화된다.
- Object expression: 코드가 실행되어 해당 표현식에 도달하면 즉시 생성된다.

처음에는 object declaration, data object, companion object를 먼저 익히고, object expression은 “이름 없는 일회용 구현”이 필요할 때 다시 살펴봐도 된다.

## 전체 형태 비교

| 형태 | instance 수 | 사용하는 모습 | 적합한 상황 |
|---|---:|---|---|
| `class` | 여러 개 가능 | `User("민수")` | instance마다 다른 값을 가질 때 |
| `object` | 하나 | `AppLogger.log()` | 하나뿐인 관리자나 공통 기능 |
| `data object` | 하나 | `Loading` | 값 없는 고정 상태·선택지 |
| `companion object` | class마다 하나 | `User.create()` | class와 밀접한 factory·상수·공유 기능 |
| object expression | 표현식 실행마다 생성 가능 | `val x = object { ... }` | 이름 없는 일회용 객체 |

## 선택 기준

- 애플리케이션에서 유일한 coordinator나 공통 reference가 필요하면 object declaration을 고려한다.
- 값 없는 singleton 상태를 읽기 좋게 출력하거나 비교하려면 data object를 고려한다.
- 특정 class와 밀접한 생성 function이나 공유 동작이 필요하면 companion object를 고려한다.
- 호출할 때마다 서로 다른 상태를 가진 instance가 필요하면 object가 아니라 일반 class를 사용한다.
- 이름 없는 구현을 한 지점에서만 잠깐 사용하려면 object expression을 고려한다.

## 초보자 학습 순서

1. `class User`와 `User(...)`를 구분한다.
2. `object Logger`에는 왜 `Logger()`가 없는지 설명해 본다.
3. `Loading`은 `data object`, `Success(data)`는 `data class`인 이유를 비교한다.
4. `Temperature.fromFahrenheit(...)`가 companion object function 호출임을 확인한다.
5. 마지막으로 object expression이 이름 있는 object declaration과 어떻게 다른지 살펴본다.

## See Also

- [Kotlin factory 함수와 객체 생성 규칙](kotlin-factory-functions-and-construction-rules.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
