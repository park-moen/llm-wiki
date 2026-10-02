# Kotlin factory 함수와 객체 생성 규칙

> Sources: Kotlin Documentation, 2026-08-12; Kotlin Documentation, 2025-11-13; Kotlin Documentation, 2025-02-23; Stripe Documentation, Unknown; Jakarta Persistence Specification, 2024-04-10
> Raw: [Coding conventions: factory functions](../../raw/kotlin/2026-08-12-kotlin-coding-conventions-factory-functions.md); [Visibility modifiers: constructors](../../raw/kotlin/2025-11-13-kotlin-visibility-modifiers-constructors.md); [Classes: constructors and initializer blocks](../../raw/kotlin/2026-08-12-kotlin-classes-initializer-validation.md); [Object declarations and expressions](../../raw/kotlin/2025-02-23-kotlin-object-declarations-and-expressions.md); [Stripe balance transaction](../../raw/database/stripe-balance-transaction-signed-amounts.md); [Jakarta Persistence entity rules](../../raw/spring/jakarta-persistence-3-2-entity-and-composite-id-rules.md)
> Updated: 2026-10-02

## Overview

**Factory 함수는 객체를 만드는 목적이 드러나는 일반 함수**다. `factory`는 Kotlin keyword가 아니다. Kotlin 공식 문서는 특별한 생성 의미를 드러내도록 `Point.fromPolar(...)` 같은 이름을 권한다. `companion object` 안에 두면 `PaymentLedger.cancel(...)`처럼 class 이름으로 호출할 수 있다. 하지만 생성자가 외부에 열려 있으면 호출자는 factory를 거치지 않고 객체를 만들 수 있다.

## 생성자와 factory는 무엇이 다른가

```kotlin
val entry = PaymentLedger(LedgerType.CANCEL, amount)
val entry = PaymentLedger.cancel(amount)
```

첫 줄은 **생성자** 호출이다. 호출자가 `type`과 금액의 부호를 직접 조합한다. 둘째 줄은 **factory 함수** 호출이다. 호출자는 “취소 기록을 만든다”는 의도와 금액의 크기만 전달하고, 함수가 부호를 결정할 수 있다. TypeScript의 `new PaymentLedger(...)`와 `PaymentLedger.cancel(...)`을 비교하면 같은 차이다.

Kotlin 문법에서 `class`는 class 선언, `constructor`는 생성자를 명시하는 keyword, `private`은 접근 범위 지정자다. `companion object`는 class에 연결된 object를 선언하고, `fun`은 그 안의 함수를 선언한다. `init`은 생성 과정에서 실행되는 초기화 블록이다. `require(...)`는 조건을 검사하는 **함수**이며 keyword가 아니다. 생성자 접근 범위를 적지 않으면 기본값은 `public`이다.

## 질문의 결제 원장 규칙을 코드로 읽기

다음 부호 규칙은 **질문에 나온 프로젝트의 설계 전제**다. Stripe 문서는 잔액 기록에 양수·음수를 사용한다는 유사 사례를 보여 주지만, 모든 결제 시스템에서 승인·취소·환불에 같은 부호를 써야 한다는 근거는 아니다. 어떤 잔액의 관점에서 기록하는지 먼저 정해야 한다.

| 기록 종류 | 질문의 규칙 | 주문별 합계에 미치는 영향 |
|---|---|---|
| 승인 | 양수 | 잔액 증가 |
| 취소 | 음수 | 승인 금액 차감 |
| 환불 | 음수 | 승인 금액 차감 |

예를 들어 승인 `+amount` 뒤에 같은 금액을 취소해 `-amount`를 기록하면 합계는 `0`이다. 그런데 `CANCEL`에 `+amount`를 허용하면 취소가 차감 대신 증가로 계산된다. 질문 속 코드 리뷰의 핵심은 **호출 경로에 따라 같은 `CANCEL`이 서로 다른 부호로 만들어질 수 있다**는 점이다. 취소용 factory가 없다면 호출자가 취소의 올바른 부호를 직접 기억해야 한다.

아래는 **JPA 엔티티가 아닌 일반 Kotlin class**에서 생성 경로를 통제하는 예시다.

```kotlin
enum class LedgerType { APPROVAL, CANCEL, REFUND }

class PaymentLedger private constructor(
    val type: LedgerType,
    val signedAmount: Long,
) {
    init {
        require(
            when (type) {
                LedgerType.APPROVAL -> signedAmount > 0
                LedgerType.CANCEL, LedgerType.REFUND -> signedAmount < 0
            }
        )
    }

    companion object {
        fun approve(amount: Long): PaymentLedger {
            require(amount > 0)
            return PaymentLedger(LedgerType.APPROVAL, amount)
        }

        fun cancel(amount: Long): PaymentLedger {
            require(amount > 0)
            return PaymentLedger(LedgerType.CANCEL, -amount)
        }

        fun refund(amount: Long): PaymentLedger {
            require(amount > 0)
            return PaymentLedger(LedgerType.REFUND, -amount)
        }
    }
}

fun recordExample(amount: Long) {
    val approval = PaymentLedger.approve(amount)
    val cancellation = PaymentLedger.cancel(amount)
    // PaymentLedger(LedgerType.CANCEL, amount)는 class 밖에서 호출할 수 없다.
}
```

호출자는 항상 **양수인 금액의 크기**를 넘긴다. Factory가 기록 종류에 맞춰 부호를 붙이고, `private constructor`가 직접 생성을 막는다. `init`의 `require`는 class 내부의 생성 코드가 바뀌더라도 부호 규칙을 다시 확인한다. 이 예시는 부호 규칙만 보여 준다. 실제 취소·환불 가능 금액이나 중복 기록 같은 별도 업무 규칙까지 검증하지는 않는다.

## 언제 쓰고 무엇을 조심할까

- 이름만 다른 동일한 생성에는 생성자만으로 충분할 수 있다. 입력 변환이나 부호 부여처럼 **특별한 생성 의미**가 있으면 이름 있는 factory가 읽기 쉽다. Kotlin 코딩 규약도 이런 경우 factory에 구별되는 이름을 권한다.
- Factory를 추가해도 생성자가 `public`이면 우회할 수 있다. 일반 Kotlin class에서 외부의 직접 생성을 막으려면 생성자 접근 범위를 제한한다.
- `init` 검증은 어떤 factory가 생성자를 호출하든 실행된다. Factory에는 입력값의 의미를, `init`에는 생성된 객체 자체가 지켜야 할 규칙을 둘 수 있다.
- 이 factory와 `init`는 **Kotlin 코드 경로**의 규칙이다. DB를 직접 수정하는 경로까지 막는다는 뜻은 아니므로, 실제 원장에서는 모든 쓰기 경로가 같은 규칙을 지키는지도 따로 확인해야 한다.

### JPA 엔티티라면

위 `private constructor` 예시를 JPA 엔티티에 그대로 붙이면 안 된다. Jakarta Persistence는 엔티티에 `public` 또는 `protected` 무인자 생성자를 요구하고, 엔티티 class가 `final`이어서는 안 된다고 규정한다. 따라서 실제 `PaymentLedger`가 JPA 엔티티라면 사용하는 JPA 설정에 맞는 생성자를 두고, 애플리케이션의 factory와 생성·저장 경로 검증을 함께 설계해야 한다. **질문의 리뷰만으로는 실제 코드의 생성자 가시성, 엔티티 여부, DB 제약을 확인할 수 없다.**

## See Also

- [Kotlin Objects](kotlin-objects.md): `companion object`가 무엇인지
- [Kotlin 클래스와 `data class`](kotlin-classes-and-data-classes.md): class와 생성자의 기본 문법
