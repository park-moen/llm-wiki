# Kotlin 상속, 인터페이스와 위임

> Sources: Kotlin Documentation, 2026-07-01; Kotlin Documentation, 2026-07-01
> Raw: [Classes and interfaces](../../raw/kotlin/2026-07-01-kotlin-intermediate-classes-interfaces.md); [Open and special classes](../../raw/kotlin/2026-07-01-kotlin-open-and-special-classes.md)
> Updated: 2026-08-12

## Overview

Kotlin은 class를 기본적으로 상속할 수 없게 두며, 구체 class를 상속 가능하게 만들 때 `open`을 명시한다. 공통 뼈대와 미완성 동작이 필요하면 abstract class를 사용하고, 서로 다른 class에 공통 행동 계약을 부여할 때는 interface를 사용한다. Interface 구현을 이미 가진 객체가 있다면 `by` delegation으로 그 처리를 맡겨 반복 코드를 줄일 수 있다.

## 먼저 잡을 한 문장 모델

| 개념 | 한 문장 모델 | 해결하려는 문제 |
|---|---|---|
| 상속 | 기존 class의 상태와 기능을 물려받는다 | 관련 class 사이의 코드 공유 |
| Open class | 구체 class가 상속과 선택적 override를 허용한다고 명시한다 | 기본적으로 닫힌 class의 확장 허용 |
| Abstract class | 같은 계열의 공통 뼈대와 미완성 부분을 정의한다 | 공통 구현은 재사용하고 일부 구현은 child에게 강제 |
| Interface | 이 class가 제공해야 할 행동을 계약으로 정한다 | 하나의 parent 제한을 넘어서 여러 역할 조합 |
| Delegation | interface의 실제 처리를 다른 객체에게 맡긴다 | 전달용 boilerplate 제거 |

흐름으로 보면 다음과 같다.

```text
class 사이에서 코드를 공유하고 싶다
└── 상속
    ├── 완성된 구체 class를 확장해야 한다
    │   └── open class
    └── 공통 구현과 자식별 구현을 나누고 싶다
        └── abstract class
            └── 하나의 parent 외에도 여러 역할이 필요하다
                └── interface
                    └── 기존 interface 구현을 거의 그대로 재사용하고 싶다
                        └── delegation
```

## 클래스 상속

Kotlin class는 기본적으로 다른 class가 상속할 수 없다. 이는 의도하지 않은 상속을 방지하고 class를 유지보수하기 쉽게 만드는 설계다. Class inheritance는 하나의 parent class만 허용하는 single inheritance 방식이다.

모든 class는 최종적으로 공통 parent인 `Any`를 상속한다. `Any`가 제공하는 `toString()` 같은 member function은 일반 class에서도 사용할 수 있다.

복잡한 class 사이에서 코드를 공유해야 할 때는 먼저 abstract class를 고려할 수 있다.

여기서 상속은 단순히 함수 몇 개를 복사하는 문법이 아니다. Parent와 child가 하나의 hierarchy를 이루며, child는 parent의 한 종류로 취급된다. 그래서 공통 기반이 분명할 때 사용해야 한다.

## Open class와 override

Interface나 abstract class가 맞지 않고 완성된 구체 class를 확장해야 한다면 class 선언 앞에 `open`을 붙인다.

```kotlin
open class Vehicle(val make: String, val model: String)

class Car(
    make: String,
    model: String,
    val numberOfDoors: Int
) : Vehicle(make, model)
```

Child class는 colon 뒤에서 parent constructor를 호출하며 parent header의 parameter를 초기화한다. Parent가 물려준 member의 동작까지 바꾸려면 parent member에도 `open`, child의 새 구현에는 `override`가 필요하다.

```kotlin
open class Vehicle(val make: String, val model: String) {
    open fun displayInfo() = "$make $model"
}

class Car(make: String, model: String) : Vehicle(make, model) {
    override fun displayInfo() = "Car: $make $model"
}
```

Property도 같은 문법으로 override할 수 있지만, 공식 tour는 단순히 값만 달라지는 property라면 parent constructor parameter로 한 번 선언하고 child가 값을 전달하는 방식을 권한다. 이 방식은 불필요한 override를 줄이고 상태가 어디서 정의되는지 분명하게 한다.

```kotlin
open class Vehicle(
    val make: String,
    val model: String,
    val transmissionType: String = "Manual"
)

class Car(make: String, model: String) :
    Vehicle(make, model, "Automatic")
```

## Abstract class

Abstract class는 다른 class에 member를 제공하기 위한 공통 뼈대다. Constructor를 가질 수 있지만 abstract class 자체의 instance는 만들 수 없다. 구현이 있는 function·property와 구현이 없는 abstract function·property를 함께 선언할 수 있다.

```kotlin
abstract class Product(val name: String) {
    abstract val category: String

    fun productInfo(): String = "$name: $category"
}

class Electronic(name: String) : Product(name) {
    override val category = "Electronic"
}
```

이 예제의 역할을 나누면 다음과 같다.

- `name`: 모든 product가 함께 사용하는 공통 상태
- `productInfo()`: 모든 product가 그대로 재사용하는 공통 구현
- `category`: product마다 달라서 child가 채워야 하는 미완성 부분
- `Electronic`: 실제 instance를 만들 수 있는 구체 class

구현이 없는 member에는 `abstract`를 사용하고, child class에서 그 동작이나 값을 정의할 때는 `override`를 사용한다. Parent constructor가 필요하면 class 이름 뒤의 colon 다음에 parent constructor를 호출한다.

```kotlin
class Electronic(name: String) : Product(name)
```

- 왼쪽 `Electronic(name: String)`은 child constructor다.
- `:`는 상속 대상을 연결한다.
- 오른쪽 `Product(name)`은 parent class의 constructor 호출이다.

따라서 abstract class는 **공통 상태와 공통 구현을 보유하면서, 일부 구현을 child에게 맡길 때** 적합하다.

## Interface

Interface는 나중에 class가 구현할 function과 property의 집합, 즉 행동 계약을 정의한다. Interface는 instance를 만들 수 없고 constructor나 class header를 갖지 않는다. 구현이 없는 interface member는 별도로 `abstract`를 표시하지 않아도 된다.

```kotlin
interface PaymentMethod {
    fun initiatePayment(amount: Double): String
}

class CardPayment : PaymentMethod {
    override fun initiatePayment(amount: Double): String = "Payment: $amount"
}
```

Class가 interface를 구현할 때는 class header 뒤에 colon과 interface 이름을 쓴다. Interface에는 constructor가 없으므로 이름 뒤에 parentheses를 붙이지 않는다.

```kotlin
class CardPayment : PaymentMethod
```

`PaymentMethod` 뒤에 `()`가 없는 것이 parent class와의 눈에 띄는 문법 차이다. Interface는 생성 방법보다 구현체가 제공해야 할 행동을 나타낸다.

Interface를 사용하면 구현 세부사항보다 추상화에 집중하고, 구현을 mock으로 바꾸기 쉬워 테스트에도 유리하다.

## 하나의 parent class와 여러 interface

Class는 parent class 하나만 상속할 수 있지만 interface는 여러 개 구현할 수 있다. Parent class와 interface를 함께 사용할 때는 colon 뒤에 parent constructor와 interface들을 comma로 나열한다.

```kotlin
interface Refundable {
    fun refund(amount: Double)
}

abstract class PaymentMethod(val name: String) {
    abstract fun processPayment(amount: Double)
}

class CreditCard(name: String) : PaymentMethod(name), Refundable {
    override fun processPayment(amount: Double) = Unit
    override fun refund(amount: Double) = Unit
}
```

이 선언은 다음 두 의미를 동시에 가진다.

```text
CreditCard는 PaymentMethod 계열이다.
CreditCard는 Refundable 역할을 제공한다.
```

문법상으로도 차이가 보인다.

- `PaymentMethod(name)`: class이므로 constructor를 호출한다.
- `Refundable`: interface이므로 parentheses를 붙이지 않는다.

Abstract class가 공통 상태와 기본 구현을 제공하고, interface가 추가 역할을 붙이는 조합이다.

## Interface delegation

Interface의 member가 많으면 wrapper class가 기존 객체로 호출을 전달하는 boilerplate code를 반복하게 된다. Kotlin은 `by` keyword로 interface 구현을 다른 instance에 위임할 수 있다.

위임하지 않으면 wrapper가 다음과 같은 전달 코드를 interface member마다 직접 작성해야 한다.

```kotlin
class SmartMessenger(
    val basicMessenger: Messenger
) : Messenger {
    override fun sendMessage(message: String) {
        basicMessenger.sendMessage(message)
    }

    override fun receiveMessage(): String {
        return basicMessenger.receiveMessage()
    }
}
```

두 function 모두 새로운 동작 없이 `basicMessenger`를 호출할 뿐이다. `by`는 이 전달 구현을 compiler가 대신 만들게 한다.

```kotlin
interface Messenger {
    fun sendMessage(message: String)
    fun receiveMessage(): String
}

class SmartMessenger(
    val basicMessenger: Messenger
) : Messenger by basicMessenger {
    override fun sendMessage(message: String) {
        basicMessenger.sendMessage("[smart] $message")
    }
}
```

`Messenger by basicMessenger`를 사용하면 compiler가 위임 코드를 제공하므로, wrapper는 변경할 behavior만 `override`하면 된다. 위 예제에서는 `receiveMessage()`는 자동으로 `basicMessenger`에 전달되고, `sendMessage()`만 `SmartMessenger`가 새로 정의한다.

Delegation은 inheritance hierarchy를 추가하는 대신 기존 instance의 동작을 재사용한다. 따라서 대부분의 동작은 그대로 전달하면서 일부 동작만 바꾸는 wrapper를 만들 때 유용하다.

### 위임에서 주의할 점

Wrapper가 property를 override해도 delegate 내부의 member function이 그 값을 자동으로 사용하는 것은 아니다. 공식 `DrawingTool` 예제에서 `CanvasSession.color`는 `blue`로 override되지만, 위임된 `PenTool.draw()`는 `PenTool` 자신의 `black`을 사용한다.

```text
session.color
└── CanvasSession의 blue

session.draw("circle")
└── PenTool.draw()에 위임
    └── PenTool 자신의 black 사용
```

위임은 delegate의 내부 구현을 wrapper에 복사하는 것이 아니라, 호출을 해당 객체로 전달하는 기능이기 때문이다. Delegate 내부에서도 wrapper의 override를 사용해야 한다면 그 member function을 wrapper에서 직접 override해 연결 방식을 명시해야 한다.

## 선택 기준

| 질문 | 선택 |
|---|---|
| 완성된 구체 class를 의도적으로 상속 가능하게 해야 하는가? | `open class` |
| 관련 class가 공통 상태와 기본 구현을 공유해야 하는가? | Abstract class |
| 서로 다른 class가 같은 행동을 제공하도록 만들고 싶은가? | Interface |
| 하나의 parent 외에 여러 역할을 조합해야 하는가? | 여러 interface 구현 |
| 기존 interface 구현을 대부분 재사용하고 일부만 변경하는가? | `by` delegation |
| 원본 class를 수정하지 않고 편의 기능만 추가하려는가? | Extension function도 비교 |

마지막으로 다음처럼 암기할 수 있다.

```text
abstract class = 공통 상태와 뼈대
open class     = 구체 class의 상속을 명시적으로 허용
interface      = 제공해야 할 행동의 약속
by             = 그 행동을 다른 객체에게 맡김
override       = 물려받거나 위임받은 동작을 내가 다시 정의
```

## See Also

- [Kotlin Objects](kotlin-objects.md)
- [Kotlin 특수 클래스](kotlin-special-classes.md)
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md)
- [Kotlin 확장 함수](kotlin-extension-functions.md)
