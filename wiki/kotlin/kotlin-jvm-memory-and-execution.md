# Kotlin/JVM 메모리·호출 frame·JIT

> Sources: Oracle Java SE Documentation, Unknown; Igor Kulakov, 2025-05-26
> Raw: [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md); [Chapter 5. Loading, Linking, and Initializing](../../raw/kotlin/kotlin-jvm-jvm-loading-linking-initialization.md); [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md); [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)
> Updated: 2026-10-15

## Overview

Kotlin/JVM의 실행 원리를 이해하려면 Kotlin의 타입·객체 규칙과 JVM의 실행 모델을 나눠 봐야 한다. JVM 명세는 thread별 stack과 호출 frame, 공유 heap 등의 논리적 구조를 설명한다. **명세의 실행 모델을 실제 프로세스의 모든 메모리 배치와 일대일로 대응시키지 않는다.** [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md)

## 실행 공간을 구분하기

| 공간 | 역할 |
| --- | --- |
| JVM stack | thread별 호출 frame 보관 |
| frame | 호출의 지역 변수, operand stack, 실행 관련 정보 |
| heap | 객체·배열을 위한 공유 실행 영역 |
| method area | 클래스 구조, 메서드 등과 관련한 공유 영역 |

지역 변수에는 primitive 값뿐 아니라 객체를 가리키는 참조도 들어갈 수 있다. 따라서 지역 변수에 객체를 담았다는 이유만으로 객체 전체가 stack에 저장됐다고 결론 내리지 않는다. JVM의 자동 메모리 관리는 명세에 포함되지만 특정 GC 알고리즘과 구체적인 수집 시점까지 이 표가 정하는 것은 아니다. [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md)

## 로딩·연결·초기화

class 파일을 읽어 클래스를 만드는 로딩, 검증·준비·해석 등의 연결, 클래스 초기화는 구분되는 과정이다. 인스턴스 생성자의 실행과 클래스 초기화도 같은 작업이 아니다. 상수의 준비와 초기화 메서드 실행을 포함한 상세 단계는 사용한 JVM 명세에서 확인한다. [Chapter 5. Loading, Linking, and Initializing](../../raw/kotlin/kotlin-jvm-jvm-loading-linking-initialization.md)

## 직접 확인하는 연습

호출과 지역 변수의 관계를 보기 위한 학습용 코드다. 이 출력은 메모리 주소나 실제 할당 위치를 측정한 결과가 아니다.

```kotlin
class Box(var value: Int)

fun change(box: Box) {
    box.value += 1
}

fun main() {
    val box = Box(1)
    change(box)
    println(box.value)
}
```

예상 출력:

```text
2
```

`main`과 `change`의 참조가 같은 객체로 이어지므로 내부 변경이 보인다. 새 호출은 별도 frame에서 실행되지만 객체를 깊게 복제해 전달하는 것은 아니다. 컴파일 후 `javap -c`로 생성 메서드와 호출 명령을 관찰할 수 있다. [Chapter 2. The Structure of the Java Virtual Machine](../../raw/kotlin/kotlin-jvm-jvm-runtime-areas.md); [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

## JIT와 escape analysis

HotSpot은 실행 프로파일을 활용하는 컴파일과 escape analysis 등으로 코드를 최적화할 수 있다. 소스에 객체 생성이 보인다고 실행 중 그 할당이 항상 남는 것은 아니다. 반대로 escape하지 않는 객체를 항상 stack으로 옮긴다고 외우면 안 된다. 수집한 HotSpot 문서는 scalar replacement를 통한 할당 제거와 stack 할당을 구분한다. [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md)

bytecode 관찰은 Kotlin compiler가 생성한 결과를 보여 준다. JIT 이후의 실제 기계어·메모리 비용과 같지는 않다. 디컴파일 결과만으로 정확한 실행 성능을 보장하지 않는다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md); [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md)

## 적용 범위

이 문서는 Java SE 명세와 HotSpot 구현 자료에 근거한다. Kotlin/Native의 메모리 관리나 JavaScript의 실행 환경에 같은 구조를 그대로 적용하지 않는다. JVM 구현이나 실행 옵션이 달라지면 구체적인 최적화 결과도 확인해야 한다. [5 Java HotSpot Virtual Machine Performance Enhancements](../../raw/kotlin/kotlin-jvm-hotspot-performance.md)

## See Also


- 학습 순서 이전: [Kotlin Sequence와 지연 평가](kotlin-sequences-and-lazy-evaluation.md)
- 학습 순서 다음: [Kotlin 예외 전파와 자원 정리](kotlin-exceptions-and-resource-management.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
