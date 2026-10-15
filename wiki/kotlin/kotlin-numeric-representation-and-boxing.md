# Kotlin 숫자의 JVM 표현과 boxing

> Sources: Kotlin Documentation, Unknown
> Raw: [Numbers](../../raw/kotlin/kotlin-numbers.md); [Calling Java from Kotlin](../../raw/kotlin/kotlin-java-interop.md)
> Updated: 2026-10-15

## Overview

Kotlin 코드에서 `Int`를 객체처럼 다룬다고 JVM에서도 항상 같은 wrapper 객체가 만들어지는 것은 아니다. JVM에서는 숫자가 primitive로 표현되거나 nullable·제네릭 등의 문맥에서 boxing될 수 있다. **Kotlin의 타입, JVM의 표현, JIT의 실제 최적화 결과를 구분해야 한다.** [Numbers](../../raw/kotlin/kotlin-numbers.md); [Calling Java from Kotlin](../../raw/kotlin/kotlin-java-interop.md)

## 같은 숫자라도 문맥이 다르다

| Kotlin 선언 | 이해할 점 |
| --- | --- |
| `val amount: Int` | JVM에서 primitive로 표현할 수 있음 |
| `val amount: Int?` | null을 표현하기 위한 boxing이 필요할 수 있음 |
| `List<Int>` | 제네릭 원소는 boxed 표현을 사용함 |
| `IntArray` | primitive 정수 배열을 위한 별도 타입 |

Kotlin의 `Array<Int>`와 `IntArray`는 같은 타입이 아니다. nullable과 제네릭에서 생기는 표현상의 차이는 Java 연동에서도 중요하다. [Numbers](../../raw/kotlin/kotlin-numbers.md); [Calling Java from Kotlin](../../raw/kotlin/kotlin-java-interop.md)

## 숫자는 값으로 비교한다

boxing과 JVM의 wrapper cache 때문에 숫자의 참조 동등성은 값 비교를 대신할 수 없다. 숫자 자체가 같은지 확인하려면 `==`를 사용한다. wrapper cache의 특정 범위를 외우거나 참조 비교 결과에 업무 규칙을 의존시키지 않는다. [Numbers](../../raw/kotlin/kotlin-numbers.md)

정수끼리 나누면 결과도 정수다. 숫자 변환은 다른 숫자 타입으로의 자동 확대를 기대하기보다 `toLong()`·`toDouble()`처럼 명시한다. 연산 문맥과 상수 대입의 추론 규칙은 구분한다. [Numbers](../../raw/kotlin/kotlin-numbers.md)

## 직접 확인하는 연습

표현과 연산을 비교하기 위한 학습용 예제다.

```kotlin
fun main() {
    val count: Int = 3
    val boxed: Int? = count
    val generic: List<Int> = listOf(count)
    val primitiveArray = intArrayOf(count)
    println(boxed == generic.first())
    println(count / 2)
    println(count.toDouble() / 2)
    println(primitiveArray.first())
}
```

예상 출력:

```text
true
1
1.5
3
```

실행 결과만으로 boxing 객체의 실제 할당 횟수를 셀 수는 없다. 생성된 bytecode를 확인하는 것과 실행 중 최적화를 측정하는 것은 별도 작업이다. [JVM 메모리와 실행](kotlin-jvm-memory-and-execution.md)에서 이어서 읽는다.

## 정밀도와 표현 범위

정수의 범위를 넘는 연산과 부동소수점의 근삿값 표현은 별개의 문제다. `Float`·`Double`은 실수를 무한한 정밀도로 담지 않는다. 부동소수점 비교는 정적으로 알려진 타입과 `Any`·제네릭 문맥의 동등성 규칙도 구분해야 한다. [Numbers](../../raw/kotlin/kotlin-numbers.md)

## See Also


- 학습 순서 이전: [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- 학습 순서 다음: [Kotlin 제어 흐름](kotlin-control-flow.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
