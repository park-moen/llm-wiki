# Kotlin 기본 타입과 타입 추론

> Sources: Kotlin Documentation, Unknown
> Raw: [Basic types](../../raw/kotlin/basic-types-3.md)
> Updated: 2026-08-10

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

## See Also

- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin Null Safety](kotlin-null-safety.md)
