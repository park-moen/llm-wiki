# Kotlin coroutine 기초와 suspend의 내부 동작

> Sources: Kotlin Documentation, Unknown; Kotlin Language Specification, Unknown; Kotlin design/compiler repository, Unknown
> Raw: [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md); [Kotlin language specification — Asynchronous programming with coroutines](../../raw/kotlin/kotlin-spec-coroutines.md); [Kotlin Coroutines](../../raw/kotlin/kotlin-coroutines-design-design-note.md); [Coroutines Codegen](../../raw/kotlin/kotlin-coroutines-codegen-implementation-note.md)
> Updated: 2026-10-15

## Overview

coroutine은 실행을 중단했다가 재개할 수 있는 계산이다. Kotlin/JVM에서 coroutine도 결국 thread 위에서 실행되지만, 중단 중에 그 thread를 계속 점유해야 하는 것은 아니다. **`suspend`를 붙이는 것과 새 coroutine·thread를 시작하는 것은 다르다.** [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)

## 언어 기능과 라이브러리 역할

`suspend`와 중단 함수 타입은 언어의 일부다. `launch`, `async`, `delay`, `Job` 등 대부분의 사용 기능은 `kotlinx.coroutines` 라이브러리가 제공한다. 선언에 suspend가 있다는 사실만으로 함수의 모든 코드가 비동기 실행되거나 실제 중단 지점이 생기는 것은 아니다. [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)

`runBlocking`은 현재 thread를 block하며 coroutine 코드를 실행하는 연결 수단이다. 이 학습 과정에서는 독립적인 `main` 예제에 사용한다. 이미 suspend 환경인 서버 요청 안에서 무조건 감싸는 기본 패턴으로 삼지 않는다. 뒤 문장은 blocking과 suspension의 차이에서 도출한 사용 지침이다. [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)

## 직접 확인하는 연습

`kotlinx-coroutines-core`가 classpath에 있는 JVM 학습 환경을 가정한다. 서로 다른 thread의 실행 순서를 추측하는 대신 기본 `runBlocking` 문맥과 `join()`으로 결과를 확인한다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch {
        delay(1)
        println("child")
    }
    println("parent")
    job.join()
    println("done")
}
```

예상 출력:

```text
parent
child
done
```

이 예제의 지연은 중단 가능한 대기를 보여 주기 위한 값이며 실행 시간 측정이 아니다. `Thread.sleep()` 같은 blocking 호출은 suspend 함수 내부에 넣었다고 자동으로 non-blocking 호출로 바뀌지 않는다. [Coroutines basics](../../raw/kotlin/kotlin-coroutines-basics.md)

## Continuation과 상태 머신

compiler는 중단 이후에 이어 실행할 정보를 `Continuation`을 이용해 표현할 수 있다. CPS(Continuation Passing Style)는 다음에 할 일을 continuation 인자로 전달하는 형태다. 중단 가능한 코드는 실행 위치와 필요한 지역 상태를 보관하는 상태 머신으로 설명할 수 있다. [Kotlin language specification — Asynchronous programming with coroutines](../../raw/kotlin/kotlin-spec-coroutines.md)

개념적으로는 다음과 같다. 이것은 JVM에서 항상 같은 클래스·field·메서드 이름이 나온다는 보장이 아니다.

```text
중단 전 코드 실행
    → 이어갈 위치와 필요한 상태 보관
    → 중단 가능 호출
    → 재개되면 저장한 위치에서 계속 실행
```

## 설계 자료와 실제 compiler 결과 구분

KEEP 설계 문서와 compiler 저장소의 Codegen 노트는 변환이 필요한 이유와 구현 예시를 설명한다. 수집 시점 저장소의 자료이므로, 오래된 bytecode 명령·내부 클래스 이름까지 현재 모든 compiler의 보장으로 일반화하지 않는다. 일반적인 개념은 명세로 확인하고 실제 결과는 프로젝트 compiler로 관찰한다. [Kotlin Coroutines](../../raw/kotlin/kotlin-coroutines-design-design-note.md); [Coroutines Codegen](../../raw/kotlin/kotlin-coroutines-codegen-implementation-note.md)

## See Also


- 학습 순서 이전: [Kotlin 라이브러리와 API 사용](kotlin-libraries-and-apis.md)
- 학습 순서 다음: [Kotlin CoroutineContext와 dispatcher](kotlin-coroutine-context-and-dispatchers.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
