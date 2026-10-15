# Kotlin 생성자와 객체 초기화 순서

> Sources: Kotlin Language Specification, Unknown; Oracle Java SE Documentation, Unknown
> Raw: [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [Chapter 5. Loading, Linking, and Initializing](../../raw/kotlin/kotlin-jvm-jvm-loading-linking-initialization.md)
> Updated: 2026-10-15

## Overview

주 생성자는 객체를 만들 때 받을 값과 생성 경로를 선언한다. `init`은 그 과정에 실행할 초기화 코드를 둔다. **인자 없는 생성자, 주 생성자, 초기화 블록은 서로 다른 개념**이다. 기존 [클래스 문서](kotlin-classes-and-data-classes.md) 다음에 읽는다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 일반적인 초기화 순서

부모 생성자를 처리하고, 해당하는 인터페이스 위임 식을 평가한 뒤, 주 생성자에 선언한 프로퍼티를 초기화한다. 그다음 클래스 본문의 프로퍼티 초기화 식과 `init` 블록을 작성 순서대로 실행한다. 보조 생성자를 통해 진입했다면 그 경로에서 주 생성자 처리가 끝난 뒤 보조 생성자 본문을 실행한다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

본문 프로퍼티를 모두 초기화한 다음 모든 `init`을 실행하는 것이 아니다. `init`이 프로퍼티 사이에 있으면 그 위치에서 실행된다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 직접 확인하는 연습

초기화 규칙을 관찰하기 위해 작성한 독립적인 학습 예제다.

```kotlin
fun mark(text: String): String {
    println(text)
    return text
}

open class Parent {
    init { mark("parent") }
}

class Child(val name: String) : Parent() {
    val first = mark("property-first")
    init { mark("init") }
    val last = mark("property-last")

    constructor() : this("guest") {
        mark("secondary")
    }
}

fun main() {
    Child()
}
```

예상 출력:

```text
parent
property-first
init
property-last
secondary
```

이 예제의 `Child()`는 인자가 없는 보조 생성자이며 `Child(name: String)` 주 생성자로 위임한다. `init`은 호출부에서 선택하는 또 다른 생성자 함수가 아니다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 초기화 중 open 멤버를 호출할 때

부모 초기화 중 override된 함수를 호출하면 자식의 초기화가 끝나기 전에 자식 구현을 실행할 수 있다. 아직 초기화하지 않은 프로퍼티를 읽는 결과에 의존하지 않는다. 수집한 명세는 초기화 이전 접근이나 순환의 결과를 unspecified로 설명하므로, 특정 JVM에서 본 null·숫자 기본값을 언어 보장으로 바꾸지 않는다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 객체 초기화와 클래스 초기화는 다르다

인스턴스를 생성하는 과정과 JVM이 클래스를 로딩·연결·초기화하는 과정은 다른 층위다. JVM의 클래스 초기화에는 별도의 동기화 규칙이 있다. `object`·companion object의 Kotlin 규칙은 [Objects](kotlin-objects.md)에서 읽고, JVM 표현을 관찰할 때는 이 구분을 유지한다. [Chapter 5. Loading, Linking, and Initializing](../../raw/kotlin/kotlin-jvm-jvm-loading-linking-initialization.md)

## See Also


- 학습 순서 이전: [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- 학습 순서 다음: [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
