# Kotlin 라이브러리와 API 사용

> Sources: Kotlin Documentation, 2026-09-02
> Raw: [Libraries and APIs](../../raw/kotlin/2026-09-02-kotlin-intermediate-libraries-and-apis.md)
> Updated: 2026-10-12

## Overview

라이브러리는 반복해서 사용할 수 있는 코드를 배포하며, API(Application Programming Interface)는 그 라이브러리를 사용하는 함수·클래스·프로퍼티 등의 접점이다. Kotlin 표준 라이브러리에서 필요한 기능을 먼저 찾고, 표준 라이브러리가 다루지 않는 기능은 별도 라이브러리로 보완할 수 있다. 사용할 때는 라이브러리 의존성 추가, 코드에서의 import, API의 opt-in 요구를 구분한다.

## 라이브러리·package·API의 관계

라이브러리 내부에는 관련 클래스·함수·유틸리티를 묶은 package와 객체가 있다. 라이브러리가 공개한 API를 통해 그 기능을 사용한다. 원문의 구분을 정리하면 다음과 같다.

| 개념 | 역할 |
|---|---|
| 라이브러리 | 재사용할 코드를 배포 |
| Package와 객체 | 관련 클래스·함수·기능을 묶음 |
| API | 사용하는 쪽에 공개된 함수·클래스·프로퍼티 등의 접점 |
| `import` | 파일에서 사용할 package나 멤버를 지정 |
| `@OptIn` | opt-in이 필요한 API를 사용하겠다고 명시 |

## 표준 라이브러리와 import

Kotlin 표준 라이브러리는 기본 타입, 함수, 컬렉션과 유틸리티를 제공한다. `kotlin` package의 기능은 명시적인 import 없이 사용할 수 있다. 원문은 문자열의 `reversed()`와 `print()`를 예로 든다.

표준 라이브러리라고 모든 기능을 import 없이 쓰는 것은 아니다. 원문의 시간 관련 예제는 `kotlin.time`을 import한다.

```kotlin
import kotlin.time.*
```

`*`는 package의 기능을 가져오는 wildcard import다. Companion object에는 wildcard import를 사용할 수 없으므로 필요한 멤버를 개별적으로 적는다.

```kotlin
import kotlin.time.Duration
import kotlin.time.Duration.Companion.hours
import kotlin.time.Duration.Companion.minutes

fun main() {
    val thirtyMinutes: Duration = 30.minutes
    val halfHour: Duration = 0.5.hours
    println(thirtyMinutes == halfHour)
    // true
}
```

`30.minutes`와 `0.5.hours`는 같은 시간을 나타내는 `Duration`을 만든다. `minutes`·`hours`는 companion object에 선언된 확장 프로퍼티다. `Duration` 클래스를 import하는 것과 이 확장 프로퍼티를 import하는 것을 구분해야 한다.

### 직접 구현하기 전에 찾아볼 곳

원문은 컬렉션, sequence, 문자열 처리, 시간 관리 기능을 먼저 표준 라이브러리에서 찾아보도록 안내한다. [표준 라이브러리 API reference](https://kotlinlang.org/api/core/kotlin-stdlib/)에서 공개된 타입·함수·프로퍼티를 확인할 수 있다.

연습문제에서도 기존 함수를 활용한다. `kotlin.math`의 `pow()`는 거듭제곱 계산에, `kotlin.time.measureTime`은 코드 실행에 걸린 시간 측정에 사용한다.

```kotlin
import kotlin.math.*

fun calculateCompoundInterest(P: Double, r: Double, n: Int, t: Int): Double {
    return P * (1 + r / n).pow(n * t)
}
```

위 함수는 원문의 복리 계산 연습 정답이다. `P`는 원금, `r`은 연 이율, `n`은 연간 복리 적용 횟수, `t`는 투자 기간의 연수다. 시간 측정 연습에서 제시한 `16 ms`는 출력 예시이며 실제 실행 시간을 보장하는 값은 아니다.

## 별도 라이브러리: 의존성 추가 후 import

표준 라이브러리가 다루지 않는 기능은 Kotlin 팀이나 커뮤니티가 만든 라이브러리로 보완할 수 있다. 원문은 여러 플랫폼의 시간 처리를 돕는 `kotlinx-datetime`을 예로 든다.

별도 라이브러리는 먼저 해당 라이브러리의 안내에 따라 의존성이나 plugin을 추가해야 한다. 그다음 코드에서 package를 import한다. `import`만 적는 것과 프로젝트에 라이브러리를 추가하는 것은 서로 다른 단계다.

원문의 예제는 `kotlinx.datetime`을 import하고 `Clock.System.now()`로 현재 시점을 얻은 다음, `TimeZone.of("America/New_York")`와 `toLocalDateTime()`으로 해당 시간대의 날짜·시간을 계산한다. 이 문서는 원문의 사용 흐름을 정리하며, 의존성 버전이나 빌드 설정은 원문에 제시되어 있지 않다.

라이브러리는 원문이 연결한 [klibs.io](https://klibs.io/)에서 찾을 수 있고, 각 라이브러리 저장소의 설치 안내와 API reference를 확인한다.

## Opt-in이 필요한 API

라이브러리 작성자는 아직 개발 중이거나 앞으로 변경될 수 있는 API에 opt-in을 요구할 수 있다. Opt-in이 없으면 원문과 같은 경고나 오류가 표시된다.

```text
This declaration needs opt-in. Its usage should be marked with '@...' or '@OptIn(...)'
```

필요한 marker 클래스에 `::class`를 붙여 `@OptIn(...)`에 전달한다. 원문은 `uintArrayOf()` 사용 예제로 다음 선언을 제시한다.

```kotlin
@OptIn(ExperimentalUnsignedTypes::class)
fun main() {
    // Create an unsigned integer array
    val unsignedArray: UIntArray = uintArrayOf(1u, 2u, 3u, 4u, 5u)

    // Modify an element
    unsignedArray[2] = 42u
    println("Updated array: ${unsignedArray.joinToString()}")
    // Updated array: 1, 2, 42, 4, 5
}
```

`import`로 이름을 사용할 수 있게 하더라도 opt-in 요구는 별도로 충족해야 한다. 원문의 마지막 연습은 `ExperimentalStdlibApi`로 표시된 기능을 사용할 때의 선언을 묻는다.

```kotlin
@OptIn(ExperimentalStdlibApi::class)
```

Opt-in 방식의 상세 규칙은 원문에 연결된 [Opt-in requirements](https://kotlinlang.org/docs/opt-in-requirements.html)에서 확인할 수 있다.

## See Also

- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin 확장 함수](kotlin-extension-functions.md)
- [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)
- [Kotlin Objects](kotlin-objects.md)
