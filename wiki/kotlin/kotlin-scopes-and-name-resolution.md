# Kotlin 스코프와 이름 해석

> Sources: Kotlin Language Specification, Unknown
> Raw: [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)
> Updated: 2026-10-15

## Overview

스코프는 이름을 사용할 수 있는 범위이며, 이름 해석은 같은 이름의 후보 중 어떤 선언을 가리킬지 정하는 과정이다. Kotlin은 **선언 스코프와 문장 스코프**를 구분한다. 지역 변수와 클래스·최상위 선언을 하나의 호이스팅 규칙으로 설명하면 안 된다. [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md)

## 선언 위치와 이름을 사용할 수 있는 시점

문장 스코프에서는 이름이 등장 순서대로 바인딩된다. 따라서 지역 변수를 선언하기 전에 그 값에 접근할 수 없다. 선언 스코프에서는 뒤에 선언한 이름을 참조할 수 있지만, 이것이 해당 값의 초기화까지 먼저 끝났다는 뜻은 아니다. [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md)

함수가 나중에 선언되어도 호출 가능한 경우와 지역 변수를 선언하기 전에 읽을 수 없는 경우를 구분한다. JavaScript의 `var`·`let`·TDZ 설명을 그대로 대응시키기보다 Kotlin의 이름 해석과 초기화 규칙을 각각 읽는다. [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md)

## 같은 이름이 여러 개 있을 때

안쪽 스코프의 선언이 바깥 이름을 가릴 수 있다. 멤버와 지역 변수의 이름을 구분하려면 `this.name`처럼 receiver를 명시한다. 여러 receiver가 섞인 경우는 [확장 함수](kotlin-extension-functions.md)와 [Scope Functions](kotlin-scope-functions.md)에서 이어서 읽는다. [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 직접 확인하는 연습

이름 가림을 보여 주기 위해 만든 독립적인 학습 예제다.

```kotlin
class Message(val text: String) {
    fun printWith(text: String) {
        println(text)
        println(this.text)
    }
}

fun main() {
    val text = "local"
    Message("member").printWith(text)
}
```

예상 출력:

```text
local
member
```

`printWith` 본문의 `text`는 매개변수이고 `this.text`는 인스턴스의 프로퍼티다. 이름 가림과 별개로 객체가 초기화되기 전에 프로퍼티를 읽는 문제는 [객체 초기화 순서](kotlin-object-initialization-order.md)에서 다룬다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 주의할 점

Kotlin 명세의 선언 스코프에서 허용되는 전방 참조는 초기화 순환을 안전하게 만들어 주지 않는다. 초기화 이전 프로퍼티 접근의 결과를 특정 JVM에서 본 기본값으로 일반화하지 않는다. 수집한 명세는 자체 버전을 표시하므로, 이후 언어 변경은 프로젝트 버전과 별도로 대조한다. [Kotlin language specification — Scopes and identifiers](../../raw/kotlin/kotlin-spec-scopes-and-identifiers.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## See Also


- 학습 순서 이전: [Kotlin 컴파일과 실행: 소스에서 JVM까지](kotlin-compilation-and-execution.md)
- 학습 순서 다음: [Kotlin 기본 타입과 타입 추론](kotlin-basic-types.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
