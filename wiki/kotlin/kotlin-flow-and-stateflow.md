# Kotlin Flow·StateFlow의 실행과 상태 전달

> Sources: Kotlin Documentation, Unknown
> Raw: [Flows](../../raw/kotlin/kotlin-flow.md); [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md); [Coroutine exceptions handling](../../raw/kotlin/kotlin-coroutine-exceptions.md)
> Updated: 2026-10-15

## Overview

`flow { ... }`로 만든 cold flow는 수집할 때 생산 코드를 실행하며, 같은 flow를 다시 수집하면 생산 코드도 다시 실행할 수 있다. `StateFlow`는 현재 상태를 가진 hot flow다. **순서대로 발생한 모든 이벤트 전달과 최신 상태 전달은 다른 계약**이다. [Flows](../../raw/kotlin/kotlin-flow.md); [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

## cold flow는 언제 실행되는가

flow를 선언하는 것과 `collect`·`toList` 같은 terminal 연산으로 수집하는 것은 구분한다. 일반적인 flow의 연산은 순차적이며, `buffer`·문맥 전환·병합 등으로 실행 구조를 바꾸면 동시성과 대기 관계도 달라진다. [Flows](../../raw/kotlin/kotlin-flow.md)

## 직접 확인하는 연습

cold flow를 다시 수집할 때 생산 코드가 다시 실행되는지 확인하는 학습용 예제다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking {
    val source = flow {
        println("produce")
        emit("value")
    }
    println("created")
    source.collect { println(it) }
    source.collect { println(it) }
}
```

예상 출력:

```text
created
produce
value
produce
value
```

`flowOn`은 upstream의 문맥에 영향을 준다. downstream collector까지 임의로 옮기는 일반 thread 전환 주석으로 이해하지 않는다. `catch` 역시 자신의 upstream 예외를 처리하는 경계와 downstream에서 발생한 예외를 구분한다. [Flows](../../raw/kotlin/kotlin-flow.md)

## StateFlow는 최신 상태를 표현한다

`MutableStateFlow`는 초기 상태가 있으며 `value`로 현재 값을 읽거나 바꿀 수 있다. 느린 수집자는 빠른 중간 상태들을 모두 보지 못할 수 있다. 새로운 값이 이전 값과 `equals()`로 같으면 전달을 억제하는 동등성 기반 병합을 사용한다. [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

일반 `collect`는 StateFlow의 정상 완료를 기다리며 끝나는 형태가 아니다. 테스트나 일회성 관찰에서는 수집을 끝내는 조건을 명시해야 한다. `stateIn`으로 cold flow를 상태 형태로 공유할 때도 scope와 시작 정책을 함께 설계한다. [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

## 상태 갱신의 원자성

`state.value = state.value + 1`은 읽기·계산·쓰기를 하나의 원자적 복합 연산으로 만드는 문장이 아니다. 상태 증가에는 원문의 `update { ... }`처럼 원자적 갱신 수단을 검토한다. StateFlow 메서드의 thread safety와 저장한 객체 내부의 모든 변경까지 안전하다는 주장은 구분한다. 뒤 문장은 thread-safe container와 객체 공유 규칙을 함께 해석한 설계 지침이다. [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

StateFlow를 모든 클릭·전송 요청 같은 이벤트의 무손실 이력 저장소로 사용하지 않는다. 필요한 것이 최신 상태인지, 매번 소비할 이벤트인지 먼저 정한다. 이 선택은 최신 값 재생과 중간 상태 생략 규칙에서 도출한 지침이다. [kotlinx.coroutines — StateFlow](../../raw/kotlin/kotlin-stateflow-api.md)

## See Also


- 학습 순서 이전: [Kotlin 구조화된 동시성·취소·예외 전파](kotlin-structured-concurrency-and-cancellation.md)
- 학습 순서 다음: [Kotlin/JVM 공유 상태·가시성·원자성](kotlin-shared-state-and-memory-visibility.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
