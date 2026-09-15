# Kotlin 확장 함수

> Sources: Kotlin Documentation, Unknown
> Raw: [Extension functions](../../raw/kotlin/kotlin-intermediate-extension-functions.md)
> Updated: 2026-08-10

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

## Extension-oriented design

확장 함수는 여러 위치에서 정의할 수 있으므로 핵심 기능과 유용하지만 필수적이지 않은 기능을 분리하는 설계를 지원한다. 이를 통해 중심 클래스는 기본 책임에 집중하고, 편의 API는 별도의 확장 함수 집합으로 구성할 수 있다. Kotlin standard library와 여러 Kotlin 라이브러리에서도 이 접근을 사용한다.

## See Also

- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 고차 함수와 람다](kotlin-higher-order-functions-and-lambdas.md)
- [Kotlin Scope Functions](kotlin-scope-functions.md)
