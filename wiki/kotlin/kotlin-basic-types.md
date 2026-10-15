# Kotlin 기본 타입과 타입 추론

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown
> Raw: [Basic types](../../raw/kotlin/basic-types-3.md); [Kotlin language specification — Type system](../../raw/kotlin/kotlin-spec-type-system.md); [Kotlin language specification — Type inference](../../raw/kotlin/kotlin-spec-type-inference.md); [Numbers](../../raw/kotlin/kotlin-numbers.md)
> Updated: 2026-10-15

## Overview

Kotlin의 모든 변수와 자료구조에는 타입이 있으며, 타입은 사용할 수 있는 함수와 프로퍼티를 컴파일러에 알려준다. 컴파일러는 초기값에서 타입을 추론할 수 있고, 초기화를 나중으로 미룰 때는 타입을 명시해야 한다.

## 타입 추론과 명시적 타입

정수 값을 대입한 변수는 문맥을 통해 `Int` 같은 수치 타입으로 추론된다. 타입을 직접 지정하려면 변수명 뒤에 `:`과 타입을 쓴다.

```kotlin
val inferred = 10
val explicit: String = "hello"
```

변수는 선언과 초기화를 분리할 수 있지만 처음 읽기 전에는 반드시 초기화되어야 한다.

```kotlin
val count: Int
count = 3
println(count)
```

초기화 전에 읽으면 컴파일 오류가 발생한다.

## 기본 타입 범주

Kotlin Tour가 소개하는 기본 타입은 다음과 같다.

| 범주 | 타입 |
|---|---|
| 정수 | `Byte`, `Short`, `Int`, `Long` |
| 부호 없는 정수 | `UByte`, `UShort`, `UInt`, `ULong` |
| 부동소수점 수 | `Float`, `Double` |
| 불리언 | `Boolean` |
| 문자 | `Char` |
| 문자열 | `String` |

`+=`, `-=`, `*=`, `/=`, `%=`는 복합 대입 연산자(augmented assignment operator)다.

## 타입은 실행 중 값과 어떻게 다른가

타입 추론은 compiler가 표현식과 사용 문맥에서 타입 제약을 풀어 선언의 타입을 정하는 과정이다. `val value = ...`라고 타입 표기를 생략한 것을 JavaScript의 동적 타입 변수처럼 이해하면 안 된다. 정해진 타입의 범위를 벗어나는 값을 임의로 다시 대입할 수 없다. [Kotlin language specification — Type inference](../../raw/kotlin/kotlin-spec-type-inference.md)

`Any`는 non-null 타입 계층의 최상위 타입이며 nullable 값까지 다루려면 `Any?`를 구분한다. `Unit`은 정상적으로 끝나되 유용한 결과를 반환하지 않는 경우, `Nothing`은 정상 결과값을 만들지 않는 경로와 연결된다. null·빈 결과·정상 종료·예외 종료를 같은 개념으로 묶지 않는다. [Kotlin language specification — Type system](../../raw/kotlin/kotlin-spec-type-system.md)

Kotlin의 숫자 타입과 JVM의 primitive·wrapper 표현도 별개다. `Int?`, 제네릭 원소, `IntArray`의 차이는 [숫자의 JVM 표현과 boxing](kotlin-numeric-representation-and-boxing.md)에서 이어서 읽는다. [Numbers](../../raw/kotlin/kotlin-numbers.md)

## 직접 작성하는 연습: 상품 수량과 금액 계산

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

수량은 변경 가능한 `Int`, 단가는 읽기 전용 `Double`로 선언하세요. 수량을 늘린 뒤 두 값을 곱해 총액을 출력하세요.

### 시작 코드

```kotlin
fun main() {
    // TODO: quantity를 Int 타입의 var로 선언하고 2로 초기화
    // TODO: unitPrice를 Double 타입의 val로 선언하고 3.5로 초기화
    // TODO: quantity에 1 더하기
    val total: Double = TODO("총액 계산")
    println(quantity)
    println(total)
}
```

### 예상 출력

```text
3
10.5
```

> [!tip]- 힌트
> 변수 선언은 val 또는 var, 이름, 타입, 초기값 순서입니다.

> [!success]- 정답 확인
> ```kotlin
> fun main() {
>     var quantity: Int = 2
>     val unitPrice: Double = 3.5
>     quantity += 1
>     val total: Double = quantity * unitPrice
>     println(quantity)
>     println(total)
> }
> ```

### 한 번 더 바꿔 보기

quantity를 val로 바꾸면 어느 줄에서 컴파일 오류가 생기는지 확인하세요.

## See Also

- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin Null Safety](kotlin-null-safety.md)

- 학습 순서 이전: [Kotlin 스코프와 이름 해석](kotlin-scopes-and-name-resolution.md)
- 학습 순서 다음: [Kotlin 숫자의 JVM 표현과 boxing](kotlin-numeric-representation-and-boxing.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
