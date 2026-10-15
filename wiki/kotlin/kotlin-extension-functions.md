# Kotlin 확장 함수

> Sources: Kotlin Documentation, Unknown; Kotlin Documentation, 2026-09-02; Kotlin Language Specification, Unknown
> Raw: [Extension functions](../../raw/kotlin/kotlin-intermediate-extension-functions.md); [Properties](../../raw/kotlin/2026-09-02-kotlin-intermediate-properties.md); [Extensions](../../raw/kotlin/kotlin-extensions.md); [Kotlin language specification — Overload resolution](../../raw/kotlin/kotlin-spec-overload-resolution.md)
> Updated: 2026-10-15

## Overview

Kotlin의 확장 함수(extension function)는 원본 클래스의 소스 코드를 바꾸지 않고도 그 클래스에 유용한 함수를 추가한 것처럼 호출하게 해준다. 확장 대상 타입을 리시버 타입(receiver type)으로 선언하고, 함수 본문에서는 해당 인스턴스를 `this`로 참조한다. 핵심 기능과 선택적 편의 기능을 분리하는 extension-oriented design에도 활용할 수 있다.

## 리시버와 선언 문법

리시버(receiver)는 함수가 호출되는 대상이다. 예를 들어 `readOnlyShapes.first()`에서 `readOnlyShapes`가 리시버다. 확장 함수를 선언할 때는 확장할 타입 뒤에 `.`과 함수 이름을 쓴다.

```kotlin
fun String.bold(): String = "<b>$this</b>"

fun main() {
    println("hello".bold())
    // <b>hello</b>
}
```

이 선언에서 `String`은 리시버 타입이고 `bold`는 확장 함수 이름이다. 호출 시 `"hello"`가 리시버가 되며, 함수 본문에서는 이를 `this`로 사용할 수 있다. 호출 문법은 멤버 함수와 마찬가지로 점(`.`)을 사용한다.

## 기존 클래스를 바꾸지 않고 기능 추가하기

확장 함수는 서드파티 클래스처럼 원본 소스를 직접 변경하기 어렵거나, 핵심 클래스에 모든 편의 기능을 넣고 싶지 않을 때 유용하다. 예를 들어 `HttpClient`의 핵심 요청 함수가 `request()`라면 자주 사용하는 GET·POST 요청을 별도 확장 함수로 표현할 수 있다.

```kotlin
fun HttpClient.get(url: String): HttpResponse =
    request("GET", url, emptyMap())

fun HttpClient.post(url: String): HttpResponse =
    request("POST", url, emptyMap())
```

`HttpClient` 인스턴스가 리시버이므로 확장 함수 본문에서 해당 클래스의 `request()`를 직접 호출할 수 있다. 새로운 네트워크 로직을 핵심 클래스에 중복 구현하지 않고, 일반적인 사용 사례를 읽기 쉬운 API로 제공하는 방식이다.

## 확장 프로퍼티와의 관계

기존 클래스에서 계산한 값을 프로퍼티 표기로 제공하려면 확장 프로퍼티를 사용할 수 있다. 확장 함수와 마찬가지로 receiver를 지정하지만, `receiver.property` 형태로 접근한다. 확장 프로퍼티에는 backing field가 없어 새 상태를 보관할 수 없다. 선언과 getter 예제는 [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)에 정리했다.

## Extension-oriented design

확장 함수는 여러 위치에서 정의할 수 있으므로 핵심 기능과 유용하지만 필수적이지 않은 기능을 분리하는 설계를 지원한다. 이를 통해 중심 클래스는 기본 책임에 집중하고, 편의 API는 별도의 확장 함수 집합으로 구성할 수 있다. Kotlin standard library와 여러 Kotlin 라이브러리에서도 이 접근을 사용한다.

## 확장 함수는 정적으로 선택된다

확장 함수는 기존 클래스에 새 virtual 멤버를 실제로 넣지 않는다. 호출 지점에서 receiver의 선언 타입 등을 기준으로 후보를 선택한다. 다음은 공식 규칙을 작게 바꾼 학습용 예제다. [Extensions](../../raw/kotlin/kotlin-extensions.md)

```kotlin
open class Shape
class Circle : Shape()
fun Shape.label() = "shape"
fun Circle.label() = "circle"

fun main() {
    val value: Shape = Circle()
    println(value.label())
}
```

출력은 `shape`다. 실제 객체가 Circle이라는 사실만으로 `Circle.label()`이 동적으로 선택되지 않는다. 사용 가능한 멤버 함수와 같은 signature의 확장 함수가 경쟁하면 멤버가 우선하는 규칙도 구분한다. [Extensions](../../raw/kotlin/kotlin-extensions.md); [Kotlin language specification — Overload resolution](../../raw/kotlin/kotlin-spec-overload-resolution.md)

## 여러 receiver가 있는 경우

클래스 안에 확장 함수를 선언하면 그 클래스를 가리키는 dispatch receiver와 확장 대상을 가리키는 extension receiver가 함께 존재할 수 있다. `this`가 무엇인지 모호하면 qualified `this`로 드러낸다. Scope Functions의 `this`·`it` 선택도 변수 이름이 숨기는 호출 대상을 확인하는 관점에서 읽는다. [Extensions](../../raw/kotlin/kotlin-extensions.md)

## 직접 작성하는 연습: Int에 양수 검사 함수 추가

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

Int를 receiver로 받는 isPositive 확장 함수를 작성하세요. receiver가 0보다 큰지 반환하세요.

### 시작 코드

```kotlin
// TODO: Int.isPositive()를 선언하고 Boolean 반환

fun main() {
    println(3.isPositive())
    println(0.isPositive())
    println((-2).isPositive())
}
```

### 예상 출력

```text
true
false
false
```

> [!tip]- 힌트
> 확장 함수 본문에서 receiver 값은 this로 참조합니다.

> [!success]- 정답 확인
> ```kotlin
> fun Int.isPositive(): Boolean = this > 0
>
> fun main() {
>     println(3.isPositive())
>     println(0.isPositive())
>     println((-2).isPositive())
> }
> ```

### 한 번 더 바꿔 보기

String을 receiver로 받는 surround 함수를 만들어 앞뒤에 괄호를 붙여 보세요.

## See Also

- [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)

- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin Scope Functions](kotlin-scope-functions.md)

- 학습 순서 이전: [Kotlin 제네릭·variance·타입 소거](kotlin-generics-and-type-erasure.md)
- 학습 순서 다음: [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
