# Kotlin CoroutineContext와 dispatcher

> Sources: Kotlin Documentation, Unknown
> Raw: [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md); [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)
> Updated: 2026-10-15

## Overview

`CoroutineContext`는 coroutine의 실행 정보를 담는 원소들의 집합이다. `Job`은 생명주기, dispatcher는 실행할 thread·thread pool 등과 관련한 역할을 한다. **coroutine 하나와 thread 하나를 고정된 일대일 관계로 이해하지 않는다.** [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

## 문맥은 실행 정보의 조합이다

부모 scope에서 시작한 coroutine은 기본적으로 그 문맥을 이어받고 지정한 원소를 조합할 수 있다. `withContext`는 지정한 문맥에서 블록을 실행하고 결과를 돌려주는 suspend 함수다. 문맥을 바꿨다는 사실이 매번 다른 OS thread로 이동했다는 뜻은 아니다. [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

| 항목 | 확인할 질문 |
| --- | --- |
| `Job` | 누가 기다리고 취소하는가? |
| dispatcher | 어떤 실행 자원에서 실행하는가? |
| `CoroutineName` | 디버깅 시 어떤 작업인지 구분하는가? |
| scope | 어떤 생명주기 안에서 자식을 시작하는가? |

문맥의 key가 같은 원소를 조합할 때는 단순 목록처럼 중복으로 계속 쌓인다고 보지 않는다. 자세한 문맥 구성 규칙은 원문 예제를 따른다. [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

## 직접 확인하는 연습

thread 이름은 환경에 따라 달라질 수 있으므로, 문맥에 지정한 이름만 출력하는 학습용 코드다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    withContext(Dispatchers.Default + CoroutineName("worker")) {
        println(coroutineContext[CoroutineName]?.name)
    }
    println("back")
}
```

예상 출력:

```text
worker
back
```

이름을 읽는 것과 실제 thread를 고정하는 것은 다르다. 예제의 `withContext`가 끝나야 `back`을 출력하므로 호출자의 순차 흐름도 유지된다. [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

## dispatcher 선택과 blocking

CPU 계산과 blocking I/O처럼 작업 성격을 먼저 구분한다. 라이브러리가 suspend API를 제공하는지, 호출 자체가 thread를 block하는지도 확인한다. `Dispatchers.Default`에 blocking 작업을 많이 올리는 것과 실제 non-blocking 대기를 사용하는 것은 다른 선택이다. 문맥 전환이 호출한 라이브러리의 I/O 구현까지 바꾸지는 않는다. [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md); [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

`Unconfined`는 중단 이후 재개 thread를 일반적인 고정 dispatcher처럼 추정할 수 없어 특별한 용도에 사용한다. 학습 초기에 이름만 보고 모든 비용을 없애는 기본값으로 선택하지 않는다. [Coroutine context and dispatchers](../../raw/kotlin/kotlin-context-dispatchers.md)

## See Also


- 학습 순서 이전: [Kotlin coroutine 기초와 suspend의 내부 동작](kotlin-coroutines-and-suspension.md)
- 학습 순서 다음: [Kotlin 구조화된 동시성·취소·예외 전파](kotlin-structured-concurrency-and-cancellation.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
