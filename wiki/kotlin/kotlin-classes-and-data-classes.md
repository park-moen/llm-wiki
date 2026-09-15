# Kotlin 클래스와 데이터 클래스

> Sources: Kotlin Documentation, Unknown
> Raw: [Classes](../../raw/kotlin/classes-3.md)
> Updated: 2026-08-10

## Overview

Kotlin 클래스는 객체의 데이터인 프로퍼티(property)와 동작인 멤버 함수(member function)를 묶는다. 클래스 헤더의 매개변수로 기본 생성자를 구성할 수 있으며, 데이터 클래스(data class)는 출력·비교·복사에 필요한 멤버 함수를 자동으로 제공한다.

## 클래스와 프로퍼티

클래스는 `class` 키워드로 선언한다. 프로퍼티는 클래스 이름 뒤의 괄호나 클래스 본문에 둘 수 있다.

```kotlin
class Contact(val id: Int, var email: String) {
    val category: String = "work"
}
```

인스턴스 생성 후 변경할 필요가 없는 프로퍼티는 `val`로 선언하는 것이 권장된다. 클래스 헤더의 매개변수에 `val`이나 `var`를 붙이지 않으면 인스턴스 생성 후 프로퍼티로 접근할 수 없다. 프로퍼티에는 함수 매개변수처럼 기본값을 줄 수 있다.

## 인스턴스와 생성자

Kotlin은 클래스 헤더에 선언한 매개변수를 사용하는 생성자를 기본으로 만든다. 클래스 이름을 함수처럼 호출해 인스턴스를 생성한다.

```kotlin
class Contact(val id: Int, var email: String)

val contact = Contact(1, "mary@gmail.com")
```

프로퍼티는 `instance.property` 형태로 읽고, `var` 프로퍼티는 같은 표기로 값을 변경할 수 있다.

```kotlin
println(contact.email)
contact.email = "jane@gmail.com"
```

## 멤버 함수

객체의 동작은 클래스 본문 안에 멤버 함수로 선언한다. 호출할 때는 인스턴스 뒤에 점과 함수 이름을 붙인다.

```kotlin
class Contact(val id: Int, var email: String) {
    fun printId() {
        println(id)
    }
}

contact.printId()
```

## 데이터 클래스

데이터 저장이 중심인 클래스는 `data class`로 선언할 수 있다.

```kotlin
data class User(val name: String, val id: Int)
```

컴파일러가 생성하는 멤버 함수에는 주 생성자(primary constructor)에 선언된 프로퍼티만 사용되며, 클래스 본문에 선언한 프로퍼티는 포함되지 않는다. 주요 자동 제공 기능은 다음과 같다.

| 기능 | 용도 |
|---|---|
| `toString()` | 인스턴스와 프로퍼티를 읽기 쉬운 문자열로 표현 |
| `equals()` 또는 `==` | 인스턴스 비교 |
| `copy()` | 기존 인스턴스를 복사하고 필요하면 일부 프로퍼티 교체 |

```kotlin
val user = User("Alex", 1)
val sameUser = user.copy()
val renamed = user.copy(name = "Max")
```

복사본을 변경하면 원본 인스턴스에 의존하는 코드에 영향을 주지 않고 별도의 값을 다룰 수 있다.

## See Also

- [Kotlin Objects](kotlin-objects.md)
- [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin Null Safety](kotlin-null-safety.md)
