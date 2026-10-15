# Kotlin 컴파일과 실행: 소스에서 JVM까지

> Sources: Sarah Haggarty, 2024-04-25; Igor Kulakov, 2025-05-26; Andrey Uskov, 2022-07-15; Kotlin Documentation, Unknown
> Raw: [K2 Compiler Performance Benchmarks and How to Measure Them on Your Projects](../../raw/kotlin/2024-04-25-jetbrains-k2-compiler-performance-2.md); [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md); [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md); [What's new in Kotlin 2.0.0 — compiler and JVM excerpts](../../raw/kotlin/kotlin-whatsnew20.md)
> Updated: 2026-10-15

## Overview

Kotlin/JVM 코드는 compiler가 JVM bytecode로 바꾸고 JVM이 실행한다. 처음에는 **작성한 소스, 컴파일 결과, 실행 환경이 서로 다르다**는 큰 흐름을 이해하면 된다. 소스 문법을 모두 알기 전에 compiler의 내부 구조 전체를 외울 필요는 없다. 이 문서의 뒤쪽 IR·증분 컴파일 설명은 함수와 클래스를 배운 후 다시 읽는다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

## 먼저 읽기: 실행까지의 큰 흐름

```text
Kotlin 소스(.kt)
    → Kotlin compiler
    → JVM class 파일
    → JVM의 로딩·실행
```

이 흐름은 Kotlin/JVM 대상이다. Kotlin/JS·Kotlin/Native의 실행 형태까지 동일하다는 뜻은 아니다. `.class`에는 명령어뿐 아니라 타입·메서드 정보와 디버깅에 필요한 정보 등이 들어갈 수 있다. JVM이 소스 파일의 각 줄을 그대로 해석해서 실행하는 것으로 생각하지 않는다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

`main()`은 학습 예제의 실행 진입점이다. 아래 예제는 표준 라이브러리만 사용하며, 이 학습 경로의 독립적인 예제는 각각 별도 파일에서 실행한다. compiler·JDK·JVM target 버전이 달라지면 관찰한 bytecode도 달라질 수 있다. [What's new in Kotlin 2.0.0 — compiler and JVM excerpts](../../raw/kotlin/kotlin-whatsnew20.md)

## 직접 확인하는 연습

공식 컴파일 흐름을 관찰하기 위해 작성한 학습용 코드다. 먼저 실행 결과를 확인하고, 나중에 IntelliJ IDEA의 bytecode 보기 또는 JDK의 `javap`로 생성 메서드를 살펴본다. IDE 메뉴 위치는 사용하는 버전에 따라 확인한다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

```kotlin
fun main() {
    val message = "Hello, Kotlin"
    println(message)
}
```

예상 출력:

```text
Hello, Kotlin
```

Kotlin 소스를 컴파일하면 JVM이 실행할 bytecode가 `.class` 파일에 담긴다. 이 파일을 사람이 읽기 쉬운 소스 코드 형태로 재구성하는 작업을 **디컴파일**이라고 한다.

디컴파일 도구는 결과를 Java 문법으로 보여줄 수 있다. 따라서 Java 코드가 보이더라도 Kotlin이 Java 소스로 컴파일됐다는 뜻은 아니다. JVM이 실행하는 것은 `.class` 파일의 bytecode이며, 디컴파일한 Java 코드는 그 내용을 살펴보기 위한 표현이다.

디컴파일은 원래 Kotlin 코드를 완벽하게 복원하는 작업은 아니다. 또한 디버깅할 때 실행 중인 bytecode와 화면에 표시된 소스가 맞지 않으면, 멈춘 줄이나 변수 표시가 예상과 다를 수 있다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

## 나중에 읽기: compiler는 무엇을 하는가

Kotlin compiler는 작성한 코드를 읽고 뜻을 파악한 뒤, 대상 플랫폼에서 실행할 수 있는 코드로 변환한다. Kotlin/JVM에서는 최종적으로 JVM bytecode를 생성한다. 이 과정을 이해하려면 먼저 **소스 분석 → 내부 표현으로 변환 → 코드 생성**이라는 흐름을 알아두면 된다.

```text
Kotlin 소스
    → 소스의 구조·의미·타입 분석
    → IR 생성
    → IR을 더 단순한 형태로 변환
    → JVM bytecode 생성
```

### 1. frontend: 작성한 코드의 뜻을 파악한다

여기서 **frontend는 컴파일러 내부에서 소스 분석을 담당하는 부분**이다. 웹 개발에서 화면을 만드는 frontend와는 다른 의미다.

컴파일러는 소스의 글자를 그대로 실행 명령어로 바꿀 수는 없다. 먼저 어떤 것이 변수·함수·연산인지 구분하고, 이름이 무엇을 가리키는지와 타입이 올바른지 확인해야 한다.

예를 들어 다음 코드를 분석한다고 생각해 보자.

```kotlin
val count: Int = 1
val next = count + 1
```

컴파일러는 `count`가 앞에서 선언한 변수이고 타입은 `Int`라는 것을 확인한다. 그 정보를 바탕으로 `count + 1`이 어떤 연산인지 판단하고, `next`의 타입도 추론한다.

### 2. IR: 코드의 의미를 내부 자료 구조로 표현한다

**IR(Intermediate Representation, 중간 표현)**은 소스의 의미를 컴파일러가 처리하기 좋은 형태로 정리한 내부 자료 구조다. 개발자가 작성한 소스와 최종 실행 코드 사이에 있기 때문에 ‘중간 표현’이라고 부른다.

앞의 예제를 이해하기 위한 개념적 표현은 다음과 같다. 실제 Kotlin IR 문법을 보여주는 예시는 아니다.

```text
count: Int 변수에 1 저장
next: Int 변수에 (count를 읽은 값 + 1) 저장
```

컴파일러는 분석한 내용을 이런 내부 표현으로 옮겨 이후 변환에 사용한다. IR 자체가 Java 소스이거나 JVM이 곧바로 실행하는 bytecode인 것은 아니다.

**IR lowering**은 이 내부 표현에 담긴 복잡한 언어 기능을 더 단순한 형태로 풀어내는 과정이다. 최종 코드를 생성하기에 적합하도록 표현을 단계적으로 바꾼다.

### 3. backend: 대상 플랫폼의 코드를 생성한다

**backend는 컴파일러 내부에서 대상 플랫폼의 코드 생성을 담당하는 부분**이다. Kotlin/JVM에서는 변환된 내부 표현을 바탕으로 JVM bytecode를 생성하며, 결과는 `.class` 파일에 담긴다.

위 예제와 설명은 JetBrains가 소개한 분석·IR 변환·IR lowering·코드 생성 단계를 학습용으로 풀어쓴 것이다. [K2 Compiler Performance Benchmarks and How to Measure Them on Your Projects](../../raw/kotlin/2024-04-25-jetbrains-k2-compiler-performance-2.md)

### K2는 무엇인가

**K2는 Kotlin compiler의 새 구현을 가리키는 이름**이다. Kotlin `2.0.0`에서 기본 compiler가 되었다. 소스를 분석하고 코드를 생성하는 내부 구조가 바뀌었다는 뜻이며, Kotlin 문법 전체가 바뀌었다는 뜻은 아니다. 과거 자료를 읽을 때는 자료의 버전과 프로젝트의 Kotlin 버전을 함께 확인한다. [What's new in Kotlin 2.0.0](../../raw/kotlin/kotlin-whatsnew20.md)

### JVM 실행 중 최적화는 별도 단계다

여기까지는 Kotlin 소스를 JVM bytecode로 만드는 과정이다. 이후 JVM이 프로그램을 실행하면서 수행하는 **JIT(Just-In-Time compilation, 실행 중 컴파일)**는 별도 단계다. HotSpot JVM은 실행 정보를 활용해 코드를 기계어로 컴파일하고 최적화할 수 있다. 자세한 내용은 [Kotlin/JVM 메모리·호출 frame·JIT](kotlin-jvm-memory-and-execution.md#jit와-escape-analysis)에서 이어서 읽는다.

## 나중에 읽기: 수정한 파일만 다시 빌드할 수 있는 이유

증분 컴파일은 변경이 영향을 주는 부분을 추적해 재컴파일 범위를 줄이는 방식이다. ABI(Application Binary Interface)는 다른 컴파일 단위가 의존하는 바이너리 형태의 계약이다. Kotlin에서는 공개 inline 함수의 본문처럼 호출자에 반영되는 정보도 영향을 줄 수 있다. [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md)

수집한 증분 컴파일 글은 당시의 classpath snapshot 설계를 설명하는 자료다. 글의 실험 기능 설정이나 benchmark 수치를 현재 프로젝트의 기본값·보장된 성능으로 옮기지 않는다. 변경 추적의 원리를 배운 뒤 실제 Gradle·Kotlin plugin 설정에 맞춰 확인한다. [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md)

## See Also


- 학습 순서 다음: [Kotlin 스코프와 이름 해석](kotlin-scopes-and-name-resolution.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
