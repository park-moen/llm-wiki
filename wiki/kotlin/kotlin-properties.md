# Kotlin 프로퍼티와 접근·위임

> Sources: Kotlin Documentation, 2026-09-02; Kotlin Documentation, 2026-09-02
> Raw: [Properties](../../raw/kotlin/2026-09-02-kotlin-intermediate-properties.md); [Libraries and APIs](../../raw/kotlin/2026-09-02-kotlin-intermediate-libraries-and-apis.md)
> Updated: 2026-10-12

## Overview

Kotlin의 프로퍼티는 값을 읽고 쓰는 접근 방식을 포함한다. Getter는 값을 읽고 setter는 값을 변경하며, 실제 값을 저장하는 공간은 backing field다. 접근 과정에 로직을 넣을 때는 `get()`·`set()`을 직접 정의하고, 기존 클래스에서 계산한 값을 제공할 때는 확장 프로퍼티를 쓴다. 접근 처리를 다른 객체에 맡기는 위임 프로퍼티와 표준 위임 객체인 `lazy`·`observable`도 사용할 수 있다.

## Getter·setter와 backing field

프로퍼티에 접근하면 getter·setter가 값을 읽거나 변경한다. 직접 선언하지 않은 기본 접근자는 컴파일러가 제공한다. `val`은 읽기 접근을 제공하고, `var`는 읽기와 쓰기 접근을 제공한다.

```kotlin
class Contact(val id: Int, var email: String) {
    var category: String = ""
}
```

원문은 `category`의 기본 동작을 다음 의사 코드로 설명한다.

```kotlin
class Contact(val id: Int, var email: String) {
    var category: String = ""
        get() = field
        set(value) {
            field = value
        }
}
```

`get()`은 저장된 값을 반환하고, `set(value)`는 새로 받은 값을 저장한다. 여기서 `field`는 프로퍼티의 실제 값을 저장하는 backing field를 가리킨다. 기본 getter 또는 setter를 사용하거나, 직접 정의한 접근자에서 `field`를 참조하면 backing field가 필요하다. 프로퍼티 표기와 값을 저장하는 공간을 같은 것으로 생각하지 않는 것이 중요하다.

### Setter 안에서 자기 프로퍼티를 다시 대입하면 안 되는 이유

원문의 `Person.name` 예제는 이름의 첫 글자를 대문자로 바꾼다. Setter 안에서 `name = ...`을 쓰면 같은 setter가 다시 호출되므로 재귀 호출이 반복되고 `StackOverflowError`가 발생한다. 실제 저장 공간에 쓰려면 `field`를 사용한다.

```kotlin
class Person {
    var name: String = ""
        set(value) {
            field = value.replaceFirstChar { firstChar -> firstChar.uppercase() }
        }
}

fun main() {
    val person = Person()
    person.name = "kodee"
    println(person.name)
    // Kodee
}
```

외부의 `person.name = ...`은 setter를 호출하고, setter 내부의 `field = ...`은 값을 저장한다. 이런 구조로 값 변환, 로그, 변경 알림처럼 접근에 필요한 로직을 추가할 수 있다.

## 확장 프로퍼티: 기존 값에서 계산한 결과 제공

확장 프로퍼티는 원본 클래스를 수정하거나 상속하지 않고 프로퍼티처럼 접근할 수 있는 계산을 추가한다. 선언은 `타입.프로퍼티명` 형태이며, 해당 객체가 receiver(접근 대상 객체)가 된다.

```kotlin
data class Person(val firstName: String, val lastName: String)

// Extension property to get the full name
val Person.fullName: String
    get() = "$firstName $lastName"
```

`person.fullName`으로 접근하면 getter가 `firstName`과 `lastName`을 조합한다. 확장 프로퍼티에는 backing field가 없으므로 확장한 객체에 새 상태를 저장하지 못한다. `val`에는 getter를, 쓰기를 허용하는 `var`에는 setter도 직접 정의해야 한다. 기존 클래스의 프로퍼티를 override하는 용도로는 사용할 수 없다.

원문의 거리 변환 연습도 같은 방식이다.

```kotlin
val Double.asMiles: Double
    get() = this * 0.621371
```

`asMiles`가 변환 결과를 보관하는 것이 아니라, 접근 대상인 `Double` 값에서 결과를 계산한다. 확장 함수가 `receiver.function()` 형태라면 확장 프로퍼티는 `receiver.property` 형태로 사용한다.

### 표준 라이브러리의 확장 프로퍼티 가져오기

`Duration`의 `minutes`·`hours`처럼 companion object에 선언된 확장 프로퍼티도 있다. 원문은 `import kotlin.time.Duration.Companion.minutes`와 `import kotlin.time.Duration.Companion.hours`를 각각 선언한 뒤 `30.minutes`·`0.5.hours`를 사용한다. Companion object에는 wildcard import를 사용할 수 없다. Import와 별도 라이브러리 의존성 추가의 차이는 [Kotlin 라이브러리와 API 사용](kotlin-libraries-and-apis.md)에 정리했다.

## 위임 프로퍼티: 접근 처리를 다른 객체에 맡기기

위임 프로퍼티는 선언 뒤에 `by`와 위임 객체를 적는다. 값을 읽을 때는 위임 객체의 `operator fun getValue()`가, 변경 가능한 프로퍼티에 값을 쓸 때는 `operator fun setValue()`가 처리를 맡는다.

클래스의 interface 구현을 맡기는 `by`와 문법은 비슷하지만 대상이 다르다. Interface 위임은 interface의 멤버 구현을 맡기고, 프로퍼티 위임은 프로퍼티의 읽기·쓰기 처리를 맡긴다. 저장이나 캐시 로직을 위임 객체에 모으면 프로퍼티마다 같은 접근 코드를 반복하지 않아도 된다.

### 직접 만든 위임 객체로 계산 결과 캐시하기

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null

    operator fun getValue(thisRef: User, property: Any?): String {
        if (cachedValue == null) {
            cachedValue = "${thisRef.firstName} ${thisRef.lastName}"
            println("Computed and cached: $cachedValue")
        } else {
            println("Accessed from cache: $cachedValue")
        }
        return cachedValue ?: "Unknown"
    }
}

class User(val firstName: String, val lastName: String) {
    val displayName: String by CachedStringDelegate()
}
```

`thisRef`는 위임 프로퍼티를 가진 객체이며, 예제에서는 `User`다. `property`는 접근 중인 프로퍼티의 정보를 받는 매개변수로, 원문 예제는 `Any?`로 받는다. 일반적인 선언 예시에는 `KProperty<*>`도 등장한다. 반환 타입은 대상 프로퍼티의 타입에 맞춰 정하며, 위 예제의 `String`이 모든 위임 객체에 고정된 타입은 아니다.

처음 `user.displayName`을 읽으면 이름을 계산해 `cachedValue`에 저장하고, 이후에는 저장된 값을 반환한다. 이 예제의 `displayName`은 `val`이므로 `setValue()`는 필요하지 않다.

## 표준 위임 객체

Kotlin 표준 라이브러리의 위임 객체를 사용하면 `getValue()`·`setValue()`를 직접 구현하지 않아도 된다.

### `by lazy`: 처음 읽을 때 초기화

`lazy { ... }`는 처음 읽을 때 람다를 실행하고, 그 결과를 저장해 이후 접근에 반환한다. 초기화 비용이 크거나 실제로 사용되지 않을 수도 있는 값에 적합하며, 기본 설정에서는 초기화가 thread-safe하다.

원문의 건강 상태 확인 연습에서 사용한 선언은 다음과 같다. `checkAppServer()`·`checkDatabase()`는 원문에 정의된 상태 확인 함수다.

```kotlin
val isAppServerHealthy by lazy { checkAppServer() }
val isDatabaseHealthy by lazy { checkDatabase() }
```

원문의 `when`은 애플리케이션 서버 상태를 먼저 확인한다. 해당 함수가 `true`를 반환해 첫 분기로 들어가면 데이터베이스 상태 프로퍼티는 읽지 않으므로 `checkDatabase()`도 실행되지 않는다. 선언한 시점에 모든 확인 작업을 실행하는 것과 차이가 있다.

### `observable`: 값 변경에 반응

`observable(초기값) { property, old, new -> ... }`는 프로퍼티에 값이 할당될 때 이전 값과 새 값을 받아 후속 처리를 한다. 로그나 변경 알림처럼 값 갱신에 반응할 때 사용할 수 있다.

```kotlin
import kotlin.properties.Delegates.observable

class Thermostat {
    var temperature: Double by observable(20.0) { _, old, new ->
        if (new > 25) {
            println("Warning: Temperature is too high! ($old°C -> $new°C)")
        } else {
            println("Temperature updated: $old°C -> $new°C")
        }
    }
}
```

`20.0`은 초기값이다. 람다의 첫 매개변수는 프로퍼티 정보이며, 예제에서는 사용하지 않으므로 `_`로 생략했다. `old`와 `new`는 이전 값과 새 값이다. 원문은 `22.5`를 할당했을 때 갱신 메시지를, `27.0`을 할당했을 때 경고를 출력한다.

## 선택 기준

| 필요한 동작 | 선택 |
|---|---|
| 저장된 값을 그대로 읽거나 변경 | 기본 getter·setter |
| 읽기·쓰기 과정에 값 변환이나 추가 로직 삽입 | 직접 정의한 getter·setter와 `field` |
| 기존 클래스의 값에서 계산한 결과 제공 | 확장 프로퍼티 |
| 반복되는 접근·저장·캐시 로직을 다른 객체에 모으기 | 위임 프로퍼티 |
| 첫 접근까지 초기화를 미루고 결과 재사용 | `by lazy` |
| 값 변경에 로그나 알림 등으로 반응 | `observable` |

## See Also

- [Kotlin 라이브러리와 API 사용](kotlin-libraries-and-apis.md)

- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- [Kotlin 확장 함수](kotlin-extension-functions.md)
- [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
