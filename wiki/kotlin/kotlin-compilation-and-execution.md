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

디컴파일한 Java 코드는 bytecode를 이해하기 위한 재구성이다. 원래 Kotlin 소스를 완벽히 되돌린 결과나 JVM이 실제로 실행하는 Java 소스로 취급하지 않는다. 줄 번호·지역 변수 정보와 실제 소스가 어긋나면 디버깅 화면의 해석도 달라질 수 있다. [Sources, Bytecode, Debugging](../../raw/kotlin/2025-05-26-jetbrains-sources-bytecode-debugging.md)

## 나중에 읽기: compiler는 무엇을 하는가

frontend는 소스의 구조와 의미를 분석하고 이름·타입을 해석한다. 이후 중간 표현(IR)을 만들고 변환하는 과정을 거쳐 backend가 대상 플랫폼의 코드를 생성한다. JetBrains의 K2 설명은 분석, IR 변환, IR lowering, 코드 생성 등의 측정 단계를 구분한다. Kotlin compiler가 하는 최적화와 JVM이 실행 중에 수행하는 JIT 최적화는 다른 단계다. [K2 Compiler Performance Benchmarks and How to Measure Them on Your Projects](../../raw/kotlin/2024-04-25-jetbrains-k2-compiler-performance-2.md)

Kotlin `2.0.0`에서는 K2가 기본 compiler가 되었다. 과거 compiler 글을 읽을 때는 설명한 버전과 현재 프로젝트의 설정을 구분한다. compiler의 새 구조를 도입했다는 사실을 언어 문법이 전부 달라졌다는 뜻으로 해석하지 않는다. [What's new in Kotlin 2.0.0 — compiler and JVM excerpts](../../raw/kotlin/kotlin-whatsnew20.md)

## 나중에 읽기: 수정한 파일만 다시 빌드할 수 있는 이유

증분 컴파일은 변경이 영향을 주는 부분을 추적해 재컴파일 범위를 줄이는 방식이다. ABI(Application Binary Interface)는 다른 컴파일 단위가 의존하는 바이너리 형태의 계약이다. Kotlin에서는 공개 inline 함수의 본문처럼 호출자에 반영되는 정보도 영향을 줄 수 있다. [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md)

수집한 증분 컴파일 글은 당시의 classpath snapshot 설계를 설명하는 자료다. 글의 실험 기능 설정이나 benchmark 수치를 현재 프로젝트의 기본값·보장된 성능으로 옮기지 않는다. 변경 추적의 원리를 배운 뒤 실제 Gradle·Kotlin plugin 설정에 맞춰 확인한다. [A New Approach to Incremental Compilation in Kotlin](../../raw/kotlin/2022-07-15-jetbrains-incremental-compilation-2.md)

## See Also


- 학습 순서 다음: [Kotlin 스코프와 이름 해석](kotlin-scopes-and-name-resolution.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
