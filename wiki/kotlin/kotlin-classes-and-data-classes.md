# Kotlin 클래스와 `data class`

> Sources: Kotlin Documentation, Unknown; Kotlin Documentation, 2026-03-14; Kotlin Documentation, 2026-09-02; Kotlin Language Specification, Unknown
> Raw: [Classes](../../raw/kotlin/classes-3.md); [Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md); [Properties](../../raw/kotlin/2026-09-02-kotlin-intermediate-properties.md); [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [Equality](../../raw/kotlin/kotlin-equality.md)
> Updated: 2026-10-15

## Overview

Kotlin 클래스는 객체의 데이터인 프로퍼티(property)와 동작인 멤버 함수(member function)를 묶는다. 클래스 헤더의 매개변수로 주 생성자를 선언할 수 있으며, `data class`는 출력·비교·복사에 필요한 멤버 함수를 자동으로 제공한다.

## 클래스와 프로퍼티

클래스는 `class` 키워드로 선언한다. 프로퍼티는 클래스 이름 뒤의 괄호나 클래스 본문에 둘 수 있다.

```kotlin
class Contact(val id: Int, var email: String) {
    val category: String = "work"
}
```

인스턴스 생성 후 변경할 필요가 없는 프로퍼티는 `val`로 선언하는 것이 권장된다. 클래스 헤더의 매개변수에 `val`이나 `var`를 붙이지 않으면 인스턴스 생성 후 프로퍼티로 접근할 수 없다. 프로퍼티에는 함수 매개변수처럼 기본값을 줄 수 있다.

프로퍼티 표기로 값을 읽고 쓰는 동작은 getter·setter를 거친다. 기본 접근자는 컴파일러가 제공하며, 필요하면 접근 과정에 로직을 추가할 수 있다. 실제 저장 공간인 backing field와 `field` 사용법은 [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)에 정리했다.

## 인스턴스와 생성자

클래스 이름 뒤에 선언한 매개변수는 주 생성자를 정의한다. 클래스 이름을 함수처럼 호출해 인스턴스를 생성한다.

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

## `data class`

`data class`는 **데이터를 담는 일이 중심인 클래스**다. 일반 `class`처럼 프로퍼티와 함수를 가질 수 있지만, 주 생성자의 프로퍼티를 바탕으로 비교·출력·복사 등에 쓰는 함수를 컴파일러가 만들어 준다. 예를 들어 이름과 ID를 한 묶음으로 전달하거나, 일부 값만 바꾼 새 값을 만드는 데 사용할 수 있다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)

```kotlin
data class User(val name: String, val id: Int)
```

주요 자동 제공 기능은 다음과 같다. **자동 생성의 대상은 주 생성자에 `val` 또는 `var`로 선언한 프로퍼티**다. 클래스 본문에 선언한 프로퍼티는 여기에 포함되지 않는다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)

| 기능 | 용도 |
|---|---|
| `toString()` | 인스턴스와 주 생성자 프로퍼티를 읽기 쉬운 문자열로 표현 |
| `equals()`·`hashCode()` | 주 생성자 프로퍼티의 값으로 동등성을 비교하고 해시값 계산 |
| `copy()` | 새 인스턴스를 만들고 지정한 프로퍼티만 다른 값으로 교체 |
| `componentN()` | 선언 순서에 따라 프로퍼티를 꺼내 구조 분해에 사용 |

```kotlin
val user = User("Alex", 1)
val sameUser = user.copy()
val renamed = user.copy(name = "Max")
val (name, id) = renamed
```

`user`와 `sameUser`는 다른 인스턴스지만 주 생성자 값이 같아 `==` 비교가 성립한다. `renamed`는 `name`만 바뀐 새 인스턴스다. 구조 분해한 `name`, `id`는 주 생성자에 적은 순서대로 나온다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)

### 주의할 점

- **`copy()`는 얕은 복사다.** `data class` 안에 변경 가능한 리스트 같은 객체가 있으면 원본과 복사본이 그 객체를 함께 참조한다. 복사본의 리스트를 수정하면 원본에서도 변경이 보인다. 따라서 `copy()`만으로 내부 객체까지 독립된다고 생각하면 안 된다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)
- **클래스 본문의 프로퍼티는 값 비교와 복사에 빠진다.** `data class Person(val name: String) { var age: Int = 0 }`에서 이름이 같고 `age`만 다른 두 인스턴스는 `equals()` 결과가 같다. `copy()`도 `age`를 복사 대상으로 삼지 않는다. 동등성을 결정해야 하는 값이라면 주 생성자에 둘지 검토해야 한다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)
- **선언에는 제약이 있다.** 주 생성자에 프로퍼티가 하나 이상 있어야 하고, 모든 주 생성자 매개변수에 `val` 또는 `var`를 붙여야 한다. `data class` 자체를 `open`, `abstract`, `sealed`, `inner`로 선언할 수 없다. [Kotlin Data classes](../../raw/kotlin/2026-03-14-kotlin-data-classes.md)

## 주 생성자·인자 없는 생성자·init 구분

클래스 이름 뒤의 매개변수 선언은 주 생성자(primary constructor)를 정의한다. 인자 없는 생성자는 호출할 때 인자를 받지 않는 생성자라는 뜻이며 주 생성자일 수도, 보조 생성자일 수도 있다. `init`은 생성 경로에서 실행할 초기화 블록이다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

```kotlin
class Member(val name: String) {
    init { require(name.isNotBlank()) }
    constructor() : this("guest")
}
```

`Member("민수")`는 주 생성자 경로이고 `Member()`는 위임하는 보조 생성자 경로다. 둘 다 주 생성자 경로의 초기화 코드를 거친다. [객체 초기화 순서](kotlin-object-initialization-order.md)에서 부모·프로퍼티·init의 정확한 순서를 확인한다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md)

## 자동 생성 함수와 객체 공유

일반 클래스의 기본 동등성과 `data class`가 생성한 값 동등성은 다르다. `copy()`가 새 바깥 객체를 만들더라도 내부의 변경 가능한 객체를 재귀적으로 복제하지는 않는다. 객체 참조·`==`·`===`를 함께 설명하는 [동등성과 불변성](kotlin-object-references-and-equality.md)을 읽은 뒤 기존 복사 연습을 다시 확인한다. [Kotlin language specification — Declarations](../../raw/kotlin/kotlin-spec-declarations.md); [Equality](../../raw/kotlin/kotlin-equality.md)

## 직접 작성하는 연습: 상품 복사와 원본 비교

본문 개념을 바탕으로 새로 작성한 학습용 과제입니다. 공식 원문의 문제를 그대로 옮긴 것은 아닙니다. 각 연습은 다른 연습과 별개로 새 Kotlin 파일에서 실행하세요. 시작 코드의 `TODO()`와 주석을 채운 뒤 실행하고, 정답은 마지막에 확인하세요.

### 할 일

이름과 가격을 가진 data class Product를 선언하세요. copy로 가격만 바꾼 객체와, 아무 값도 바꾸지 않은 객체를 만드세요.

### 시작 코드

```kotlin
// TODO: name: String과 price: Int를 가진 data class Product 선언

fun main() {
    val original = Product("book", 10)
    val discounted = TODO("가격을 7로 바꾼 복사본")
    val same = TODO("값을 유지한 복사본")
    println(original.price)
    println(discounted.price)
    println(original == same)
    println(original == discounted)
}
```

### 예상 출력

```text
10
7
true
false
```

> [!tip]- 힌트
> data class의 copy는 지정한 프로퍼티만 바꾼 새 객체를 반환합니다.

> [!success]- 정답 확인
> ```kotlin
> data class Product(val name: String, val price: Int)
>
> fun main() {
>     val original = Product("book", 10)
>     val discounted = original.copy(price = 7)
>     val same = original.copy()
>     println(original.price)
>     println(discounted.price)
>     println(original == same)
>     println(original == discounted)
> }
> ```

### 한 번 더 바꿔 보기

discounted의 이름도 바꿔 보고 original의 이름이 유지되는지 확인하세요.

## See Also

- [Kotlin 프로퍼티와 접근·위임](kotlin-properties.md)

- [Kotlin Objects](kotlin-objects.md)
- [Kotlin 상속, 인터페이스와 위임](kotlin-inheritance-interfaces-and-delegation.md)
- [Kotlin 함수](kotlin-functions.md)
- [Kotlin 컬렉션](kotlin-collections.md)
- [Kotlin Null Safety](kotlin-null-safety.md)

- 학습 순서 이전: [Kotlin Null Safety](kotlin-null-safety.md)
- 학습 순서 다음: [Kotlin 생성자와 객체 초기화 순서](kotlin-object-initialization-order.md)
- [Kotlin 전체 학습 순서](../index.md#kotlin)
