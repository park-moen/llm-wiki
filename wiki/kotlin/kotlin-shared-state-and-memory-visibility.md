# Kotlin/JVM 공유 상태·가시성·원자성

> Sources: Oracle Java SE Documentation, Unknown; Kotlin Documentation, Unknown
> Raw: [Chapter 17. Threads and Locks](../../raw/kotlin/kotlin-jvm-java-memory-model.md); [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency-full-guide.md); [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)
> Updated: 2026-10-15

## Overview

여러 thread에서 같은 상태를 읽고 쓸 때는 **변경을 다른 실행 흐름이 볼 수 있는가**와 **여러 단계를 중간 개입 없이 수행하는가**를 따로 판단한다. 앞은 가시성, 뒤는 원자성과 관련한다. coroutine을 사용해도 여러 thread에서 실행하면 공유 상태 문제가 남는다. [Chapter 17. Threads and Locks](../../raw/kotlin/kotlin-jvm-java-memory-model.md); [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency-full-guide.md)

## Volatile만으로 증가가 안전해지지 않는 이유

`counter++`는 값을 읽고 계산하고 다시 쓰는 복합 작업이다. `@Volatile`이 개별 접근의 가시성과 원자성을 제공해도 이 복합 연산 전체가 자동으로 원자적이 되지는 않는다. 원문은 volatile counter가 기대한 결과를 보장하지 않는 예제와 atomic counter 대안을 비교한다. [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency-full-guide.md)

## happens-before는 무엇을 설명하는가

Java 메모리 모델의 happens-before 관계는 실행 간 순서와 관찰 가능성의 보장을 설명한다. 프로그램 순서, monitor 동기화, volatile 접근 등에 관련한 규칙이 있다. 단순히 벽시계상 먼저 실행했거나 잠깐 sleep했다는 이유로 같은 보장이 생기는 것은 아니다. [Chapter 17. Threads and Locks](../../raw/kotlin/kotlin-jvm-java-memory-model.md)

## 직접 확인하는 연습

원문 Mutex 예제를 작게 바꾼 학습용 코드다. 서로 다른 thread가 공유 상태를 갱신하더라도 보호 구간 안에서 증가하도록 한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.sync.Mutex
import kotlinx.coroutines.sync.withLock

fun main() = runBlocking {
    var counter = 0
    val mutex = Mutex()
    coroutineScope {
        repeat(3) {
            launch(Dispatchers.Default) {
                repeat(3) {
                    mutex.withLock { counter++ }
                }
            }
        }
    }
    println(counter)
}
```

예상 출력:

```text
9
```

기다릴 자식을 scope에 묶고 증가를 보호하기 때문에 최종 값을 확인할 수 있다. 잠금을 제거한 예제가 우연히 같은 값을 출력하더라도 경쟁 상태가 없다는 증거로 삼지 않는다. [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency-full-guide.md)

## 어떤 수단을 선택하는가

| 수단 | 적용할 때 확인할 것 |
| --- | --- |
| atomic 연산 | 단일 값의 증가·교체처럼 지원하는 원자적 연산인가? |
| Mutex | 여러 값을 함께 다루는 보호 구간이 필요한가? |
| thread confinement | 상태 접근을 정한 실행 자원으로 제한할 수 있는가? |
| 공유 상태 줄이기 | 복사·불변 값·소유권 분리로 공유 자체를 줄일 수 있는가? |

원문은 thread-safe 자료구조, 세밀한·큰 단위의 thread confinement, Mutex를 비교한다. `val`·read-only 인터페이스나 `object`의 안전한 초기화만으로 내부 상태 갱신까지 안전해지는 것은 아니다. [Shared mutable state and concurrency](../../raw/kotlin/kotlin-shared-mutable-state-and-concurrency-full-guide.md)

## StateFlow와 연결하기

StateFlow는 thread-safe하게 상태를 관리하는 API이지만, 복합 갱신에는 `update` 같은 수단을 사용한다. 상태 객체를 외부에서 직접 변경하는 설계까지 자동으로 동기화해 주는 것으로 보지 않는다. [Flow와 StateFlow](kotlin-flow-and-stateflow.md)에서 상태 전달 계약과 함께 읽는다. [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

## See Also


- 학습 순서 이전: [Kotlin Flow·StateFlow의 실행과 상태 전달](kotlin-flow-and-stateflow.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
