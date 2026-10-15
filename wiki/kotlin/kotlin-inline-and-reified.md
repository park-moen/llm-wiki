# Kotlin inline·noinline·crossinline·reified

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown; Andrey Uskov, 2022-07-15; Oracle Java SE Documentation, Unknown
> Raw: [Inline functions](../../raw/kotlin/kotlin-inline-functions.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md); [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md)
> Updated: 2026-10-15

## Overview

`inline`은 함수와 전달된 람다를 호출 지점에 확장할 수 있게 한다. 함수 객체·호출 비용을 줄일 수 있지만 코드 크기와 공개 API의 제약도 함께 생긴다. **inline을 모든 함수에 붙이는 성능 옵션으로 이해하지 않는다.** [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## 무엇이 달라지는가

일반 함수 값은 객체로 표현되고 외부 변수를 캡처할 수 있다. inline 가능한 람다는 호출 지점에 들어가므로 그 경로의 객체·호출 비용을 없앨 여지가 있다. Kotlin compiler의 inline과 JVM이 실행 중 수행하는 JIT inlining은 다른 단계다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md); [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md)

| 표기 | 핵심 의미 |
| --- | --- |
| `inline` | 함수와 inline 가능한 람다를 호출 지점에 확장 |
| `noinline` | 해당 람다는 inline하지 않아 저장·전달 가능 |
| `crossinline` | 다른 실행 문맥에서 호출할 수 있도록 비지역 반환 제한 |
| `reified T` | inline 호출 지점의 타입을 함수 본문에서 활용 |

`crossinline`을 썼다고 별도 thread에서 실행되는 것은 아니다. 호출 형태에 대한 제약과 실행 thread를 구분한다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## 직접 확인하는 연습

정상 반환과 비지역 반환을 비교하는 학습용 코드다.

```kotlin
inline fun invokeNow(block: () -> Unit) {
    block()
}

fun containsStop(values: List<String>): Boolean {
    values.forEach {
        if (it == "stop") return true
    }
    return false
}

inline fun <reified T> matches(value: Any): Boolean = value is T

fun main() {
    invokeNow { println("inline") }
    println(containsStop(listOf("go", "stop")))
    println(matches<String>("hello"))
}
```

예상 출력:

```text
inline
true
true
```

`return true`는 람다만 끝내는 것이 아니라 `containsStop()`을 끝낸다. `return@forEach` 같은 라벨 반환은 다른 의미다. 이 동작이 가능한 호출 경로인지 inline 제약을 함께 확인한다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## 공개 inline 함수와 빌드

공개 inline 함수 본문은 외부 호출자의 컴파일 결과에 들어갈 수 있다. 그래서 본문에서 private·일반 internal 선언을 자유롭게 참조할 수 없고, `@PublishedApi` 같은 명시적 계약을 검토해야 한다. 함수 본문 변경이 호출자 재컴파일에 영향을 줄 수 있는 이유도 여기서 연결된다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md); [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md)

## 성능을 판단할 때

작은 함수가 inline됐다는 사실과 실제 애플리케이션이 빨라졌다는 결론은 다르다. 코드 크기가 커질 수도 있으므로, compiler 결과 관찰과 실제 실행 측정을 구분한다. [Inline functions](../../raw/kotlin/kotlin-inline-functions.md)

## See Also


- 학습 순서 이전: [Kotlin Scope Functions](kotlin-scope-functions.md)
- 학습 순서 다음: [Kotlin 특수 클래스](kotlin-special-classes.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
