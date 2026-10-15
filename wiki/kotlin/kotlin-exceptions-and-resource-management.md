# Kotlin 예외 전파와 자원 정리

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown; Oracle Java SE Documentation, Unknown
> Raw: [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md); [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md)
> Updated: 2026-10-15

## Overview

예외가 발생하면 현재 흐름의 정상 결과 대신 예외를 처리할 경로로 이동한다. 일치하는 `catch`가 없으면 호출자 쪽으로 전파될 수 있다. **오류를 잡는 것, 정상 결과를 반환하는 것, 자원을 정리하는 것은 서로 다른 책임**이다. [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md); [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md)

## try는 표현식이다

Kotlin의 `try`는 값을 만들 수 있다. 정상 경로의 마지막 표현식 또는 실행된 `catch`의 결과가 사용된다. `finally`는 정리 코드를 실행하는 부분이며, 그 블록의 마지막 표현식이 일반적인 try 결과값을 대체하는 것은 아니다. `finally` 안에서 return·throw로 흐름을 덮는 코드는 피하는 편이 해석하기 쉽다. 마지막 문장은 제어 흐름 규칙에서 도출한 지침이다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md)

Kotlin은 Java의 checked exception 처리를 compiler가 강제하는 방식과 다르다. 그렇다고 예외 타입과 호출자의 복구 정책이 없어지는 것은 아니다. Java API와 연동할 때는 선언·문서화의 역할을 함께 확인한다. [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md)

## 직접 확인하는 연습

메모리상의 작은 자원을 사용해 성공·실패와 정리 경로를 관찰하는 학습용 예제다.

```kotlin
class Resource : java.io.Closeable {
    override fun close() {
        println("closed")
    }
}

fun main() {
    try {
        Resource().use {
            println("using")
            error("failed")
        }
    } catch (exception: IllegalStateException) {
        println("caught")
    }
}
```

예상 출력:

```text
using
closed
caught
```

`use`는 블록을 벗어날 때 자원을 닫도록 구성된 표준 함수다. 위 예제에서는 예외가 블록 밖으로 전파되기 전에 close가 실행된다. 모든 객체가 자동으로 자원 정리 대상이 되는 것이 아니라 해당 close 계약을 가진 자원을 사용한다. [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md)

## Nothing과 실패 경로

`throw`처럼 정상 값으로 돌아오지 않는 표현식은 `Nothing`과 연결된다. 유용한 값을 반환하지 않되 정상 종료할 수 있는 `Unit`과는 의미가 다르다. 그래서 Elvis 연산자의 오른쪽에서 조기 반환·예외를 사용하여 이후 경로의 타입을 좁힐 수 있다. [Kotlin language specification — Expressions](../../raw/kotlin/kotlin-spec-expressions.md); [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md)

## catch의 범위를 결정하기

기술 예외를 모두 잡아 기본값으로 바꾸면 실패의 의미가 사라질 수 있다. 호출자가 복구할 수 있는 경계에서 필요한 타입을 처리하고 자원 정리는 `use`·`finally`로 분리한다. 이는 원문의 예외 처리와 정리 구조에 근거한 설계 지침이다. coroutine의 `CancellationException`은 일반 오류 복구와 다른 의미가 있으므로 [구조화된 동시성](kotlin-structured-concurrency-and-cancellation.md)에서 이어서 읽는다. [Exception and error handling](../../raw/kotlin/kotlin-exceptions.md)

## See Also


- 학습 순서 이전: [Kotlin/JVM 메모리·호출 frame·JIT](kotlin-jvm-memory-and-execution.md)
- 학습 순서 다음: [Kotlin Java 연동·reflection·JVM 선언](kotlin-java-interop-and-reflection.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
