# Kotlin 구조화된 동시성·취소·예외 전파

> Sources: Kotlin Documentation, Unknown
> Raw: [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md); [Cancellation and timeouts](../../raw/kotlin/kotlin-cancellation.md); [Coroutine exceptions handling](../../raw/kotlin/kotlin-coroutine-exceptions.md); [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md); [Flows](../../raw/kotlin/kotlin-flow.md)
> Updated: 2026-10-15

## Overview

구조화된 동시성은 자식 작업을 scope의 생명주기에 묶어 완료·취소·실패를 함께 다루는 원칙이다. 단순히 작업을 시작하는 것 외에 **누가 기다리고, 누가 취소하며, 실패를 어디에서 처리할지**를 정해야 한다. [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)

## 부모와 자식의 관계

일반적인 부모 Job의 취소는 자식으로 전파된다. 자식의 취소가 아닌 실패는 일반 scope의 부모와 다른 자식에도 영향을 줄 수 있다. `supervisorScope`·`SupervisorJob`은 이 실패 전파를 분리하는 모델이며, 모든 예외를 자동으로 소비하는 기능은 아니다. [Coroutine exceptions handling](../../raw/kotlin/kotlin-coroutine-exceptions.md); [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

## 취소는 협력적이다

`cancel()`은 임의의 Kotlin 코드 실행을 즉시 강제로 끊는 일반 도구가 아니다. 중단 가능한 취소 협력 함수나 `ensureActive()`·`isActive` 등의 확인 지점이 필요하다. CPU 계산 loop에서 아무 확인도 하지 않으면 취소 요청 이후에도 계속 실행할 수 있다. [Cancellation and timeouts](../../raw/kotlin/kotlin-cancellation.md)

`CancellationException`을 광범위한 catch로 잡았다면 취소를 정상 값으로 삼켜 버리지 않도록 한다. 필요한 정리를 한 뒤 다시 던지는 방식 등 취소 경로를 유지한다. [Cancellation and timeouts](../../raw/kotlin/kotlin-cancellation.md)

## 직접 확인하는 연습

작업이 정리 구간에 들어가는 것을 확인하기 위한 학습 예제다. `UNDISPATCHED`는 첫 중단 지점까지 시작하도록 하여 단순한 시작 순서 추측을 피한다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch(start = CoroutineStart.UNDISPATCHED) {
        try {
            awaitCancellation()
        } finally {
            println("cleanup")
        }
    }
    job.cancelAndJoin()
    println("done")
}
```

예상 출력:

```text
cleanup
done
```

정리 코드에서 다시 suspend해야 한다면 이미 취소된 문맥이라는 사실도 고려해야 한다. 원문은 필요한 경우 제한된 정리 구간에 `NonCancellable`을 사용하는 예제를 제시한다. 이를 작업 전체의 취소를 무력화하는 기본 패턴으로 확장하지 않는다. [Cancellation and timeouts](../../raw/kotlin/kotlin-cancellation.md)

## launch와 async의 예외

root coroutine의 예외를 다루는 방식과 자식 coroutine의 전파를 구분한다. `async`는 결과를 `Deferred`에 담고 `await()`에서 실패를 관찰할 수 있지만, 일반 자식의 실패가 await할 때까지 부모에 아무 영향도 주지 않는다고 단정하면 안 된다. `CoroutineExceptionHandler`는 이미 발생한 예외를 보고하는 역할이며 실패한 coroutine을 원래 지점에서 복구하지 않는다. [Coroutine exceptions handling](../../raw/kotlin/kotlin-coroutine-exceptions.md)

## 직접 바꿔 보기

정상 종료하는 자식과 실패하는 자식을 일반 `coroutineScope`와 `supervisorScope`에서 비교한다. 어느 작업을 기다렸고 어떤 예외를 누가 소비하는지 먼저 적은 뒤 실행 결과를 확인한다. 결과의 전체 출력 순서는 dispatcher와 동기화 여부에 따라 달라질 수 있다. [Coroutine exceptions handling](../../raw/kotlin/kotlin-coroutine-exceptions.md)

## See Also


- 학습 순서 이전: [Kotlin CoroutineContext와 dispatcher](kotlin-coroutine-context-and-dispatchers.md)
- 학습 순서 다음: [Kotlin Flow·StateFlow의 실행과 상태 전달](kotlin-flow-and-stateflow.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
