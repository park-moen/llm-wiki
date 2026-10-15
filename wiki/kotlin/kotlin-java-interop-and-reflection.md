# Kotlin Java 연동·reflection·JVM 선언

> Sources: Kotlin Documentation, Unknown; Svetlana Isakova, 2020-07-16
> Raw: [Calling Java from Kotlin](../../raw/kotlin/kotlin-java-interop.md); [Calling Kotlin from Java](../../raw/kotlin/kotlin-java-to-kotlin-interop.md); [Reflection](../../raw/kotlin/kotlin-reflection.md); [What's new in Kotlin 2.2.0 — JVM excerpts](../../raw/kotlin/kotlin-whatsnew22.md); [Interfaces](../../raw/kotlin/kotlin-interfaces.md); [Kotlin 1.4-M3: Generating Default Methods in Interfaces](../../raw/kotlin/2020-07-16-jetbrains-jvm-default-methods-design-2.md)
> Updated: 2026-10-15

## Overview

같은 JVM에서 실행된다고 Kotlin 선언과 Java 선언이 항상 같은 문법·API 모양으로 보이는 것은 아니다. Kotlin의 프로퍼티·최상위 함수·companion object·기본 인자가 Java에서 어떤 형태로 노출되는지 확인해야 한다. **소스 수준의 편의 문법과 JVM의 공개 계약을 구분한다.** [Calling Kotlin from Java](../../raw/kotlin/kotlin-java-to-kotlin-interop.md)

## Java에서 넘어오는 platform type

nullability 정보가 없는 Java 타입은 Kotlin에서 platform type으로 다뤄질 수 있다. IDE에 `String!` 같은 형태로 보이지만 Kotlin 소스에 그대로 쓰는 타입 문법은 아니다. 이를 non-null로 받아도 실제 Java 값이 null이면 runtime assertion·접근 과정에서 실패할 수 있다. 명시적 null 처리나 Java 쪽 annotation으로 경계를 분명히 한다. [Calling Java from Kotlin](../../raw/kotlin/kotlin-java-interop.md)

## 선언이 보이는 형태

| Kotlin 선언 | Java 연동에서 볼 것 |
| --- | --- |
| 일반 프로퍼티 | 보통 getter·setter를 통한 접근; field 노출은 별도 조건 |
| 최상위 함수 | 파일에 대응하는 생성 클래스의 static 메서드 |
| companion object 멤버 | companion 접근과 `@JvmStatic`의 차이 |
| 기본 인자 | Java overload 생성 여부와 `@JvmOverloads` |
| JVM 이름 충돌 | `@JvmName`·value class 이름 변환 등 |

이 표는 모든 선언이 같은 형식으로 생성된다는 뜻이 아니다. backing field 유무, 접근 범위와 annotation을 함께 확인한다. [Calling Kotlin from Java](../../raw/kotlin/kotlin-java-to-kotlin-interop.md)

## 직접 확인하는 연습

reflection으로 Kotlin 클래스 정보와 프로퍼티 이름을 읽는 학습용 예제다. JVM reflection 기능을 사용하므로 실행 classpath에 `kotlin-reflect`를 포함한다.

```kotlin
import kotlin.reflect.full.memberProperties

data class User(val name: String)

fun main() {
    val klass = User::class
    println(klass.simpleName)
    println(klass.java.simpleName)
    println(klass.memberProperties.map { it.name }.sorted())
}
```

예상 출력:

```text
User
User
[name]
```

`KClass`는 Kotlin의 클래스 참조이며 JVM의 `Class`와 같은 타입이 아니다. reflection으로 선언을 조사하는 것과 compiler가 타입을 추론하는 것, annotation을 읽어 객체를 만드는 framework 동작은 서로 구분한다. [Reflection](../../raw/kotlin/kotlin-reflection.md)

## interface default 구현의 버전 차이

초기 설명에서는 `DefaultImpls` 같은 보조 클래스가 중심이었다. 수집한 JetBrains 글은 당시 JVM default method 생성 설계를 설명하는 역사 자료다. 현재 설정을 적용할 때는 수집한 Interfaces 문서와 Kotlin `2.2.0` 변경 사항을 함께 읽는다. [Kotlin 1.4-M3: Generating Default Methods in Interfaces](../../raw/kotlin/2020-07-16-jetbrains-jvm-default-methods-design-2.md); [What's new in Kotlin 2.2.0 — JVM excerpts](../../raw/kotlin/kotlin-whatsnew22.md)

현재 수집한 Interfaces 문서는 `-jvm-default`의 `enable`, `no-compatibility`, `disable`을 구분한다. 생성되는 default 구현과 호환 bridge·보조 클래스가 다를 수 있으므로 compiler 옵션을 고정하지 않은 디컴파일 예제를 절대적인 구조로 외우지 않는다. [Interfaces](../../raw/kotlin/kotlin-interfaces.md)

## See Also


- 학습 순서 이전: [Kotlin 예외 전파와 자원 정리](kotlin-exceptions-and-resource-management.md)
- 학습 순서 다음: [Kotlin 라이브러리와 API 사용](kotlin-libraries-and-apis.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
