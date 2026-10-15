# Kotlin Sequence와 지연 평가

> Sources: Kotlin Documentation, Unknown
> Raw: [Sequences](../../raw/kotlin/kotlin-sequences.md); [Collections overview](../../raw/kotlin/kotlin-collections-overview.md)
> Updated: 2026-10-15

## Overview

`Iterable`의 일반적인 변환 체인은 각 단계의 결과를 만들며 진행하고, `Sequence`는 원소를 요청할 때 연산을 이어 실행한다. **지연 평가는 실행 시점과 순서를 바꾸는 전략이지 항상 더 빠르다는 보장이 아니다.** 컬렉션을 배운 다음 읽는다. [Sequences](../../raw/kotlin/kotlin-sequences.md)

## 중간 연산과 terminal 연산

`map`·`filter`는 Sequence를 구성하는 중간 연산으로 연결할 수 있다. `toList()`처럼 결과를 요구하는 terminal 연산이 실행되면 실제 원소 처리가 시작된다. Sequence는 재료인 컬렉션을 불변 복사해 두는 기능이 아니며, 시작 시점과 원소 소비를 구분해야 한다. [Sequences](../../raw/kotlin/kotlin-sequences.md)

## 직접 확인하는 연습

실행 순서를 드러내기 위해 출력 부수 효과를 넣은 학습용 예제다.

```kotlin
fun main() {
    val values = sequenceOf(1, 2, 3)
        .map {
            println("map:$it")
            it * 2
        }
        .filter {
            println("filter:$it")
            it > 2
        }
        .take(1)
    println("ready")
    println(values.toList())
}
```

예상 출력:

```text
ready
map:1
filter:2
map:2
filter:4
[4]
```

결과를 얻는 데 충분한 원소를 처리하면 `take(1)` 이후에는 나머지를 읽지 않는다. 같은 체인을 `listOf(...).map(...).filter(...)`로 바꾸면 단계별 평가 순서가 어떻게 다른지 비교한다. [Sequences](../../raw/kotlin/kotlin-sequences.md)

## 지연 평가의 비용

중간 컬렉션을 피하는 이점이 있지만 Sequence 자체의 추가 비용도 있다. 작은 컬렉션이나 단순 계산에서는 그 비용이 중요할 수 있다. 컬렉션 크기, 실제로 소비한 원소 수, 변환의 개수를 함께 보고 선택한다. [Sequences](../../raw/kotlin/kotlin-sequences.md)

## Sequence와 coroutine 연결

`sequence { yield(...) }`는 원소를 요청하는 쪽과 생산하는 쪽 사이를 이어주는 builder다. 이 문법의 일시 중단 구조가 비동기 thread 작업을 자동 시작한다는 뜻은 아니다. Sequence builder 안의 허용된 suspension과 일반 `kotlinx.coroutines`의 dispatcher·취소는 별도로 읽는다. [Sequences](../../raw/kotlin/kotlin-sequences.md)

## See Also


- 학습 순서 이전: [Kotlin 컬렉션](kotlin-collections.md)
- 학습 순서 다음: [Kotlin/JVM 메모리·호출 frame·JIT](kotlin-jvm-memory-and-execution.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
