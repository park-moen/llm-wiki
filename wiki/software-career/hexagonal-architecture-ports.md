# 헥사고날 아키텍처의 포트: 계약과 Adapter의 역할

> Sources: Alistair Cockburn, Unknown (Hexagonal architecture the original 2005 article); AWS Prescriptive Guidance, Unknown (Hexagonal architecture pattern); dory.m, 카카오페이 기술 블로그, 2025-01-21
> Raw: [Cockburn의 Ports and Adapters 원문](../../raw/software-career/hexagonal-architecture-the-original-2005-article.md); [AWS의 Hexagonal architecture pattern](../../raw/software-career/hexagonal-architecture-pattern.md); [카카오페이의 포트 경계와 제거 경험](../../raw/software-career/2025-01-21-home-hexagonal-architecture.md)
> Updated: 2026-10-23

## Overview

Ports and Adapters에서 포트는 애플리케이션과 외부가 특정 목적의 대화를 나누기 위한 계약이다. 외부에 제공하는 기능이나 외부에 요청하는 기능을 기술과 분리해 표현한다. Adapter는 그 계약을 UI·데이터베이스·외부 서비스 같은 구체 기술과 연결한다. 핵심 목표는 비즈니스 로직을 외부 기술에서 분리해 독립적으로 실행하고 테스트하는 것이다.

## 포트는 무엇을 정하는가

Cockburn은 포트를 목적이 있는 대화로 설명한다. 예를 들어 할인 계산을 요청하는 대화와 계산에 필요한 할인율을 조회하는 대화는 목적이 다르다. 이를 각각 경계의 API로 표현할 수 있다.

- 할인 계산 계약: 금액을 받으면 할인액을 반환한다.
- 할인율 조회 계약: 금액에 적용할 할인율을 반환한다.

여기서 API는 HTTP API만 뜻하지 않는다. 원문은 함수 호출과 메시지도 다룬다. 따라서 포트를 네트워크의 포트 번호나 특정 URL과 같은 것으로 이해하지 않는다. 계약을 호출할 때 HTTP를 사용할지는 adapter가 맡는 기술 선택이다.

계약을 읽을 때는 어떤 입력을 받아 어떤 결과를 제공하는지, 어떤 기능을 의미하는지 확인한다. DB 테이블명이나 웹 화면의 버튼보다 애플리케이션의 목적을 먼저 표현하는 경계다.

## 포트와 Adapter의 차이

| 구분 | 책임 | 할인 계산 예 |
|---|---|---|
| 포트 | 기술과 분리된 대화의 계약 | 할인 계산 호출, 할인율 조회 |
| Adapter | 특정 기술의 입력·출력을 계약에 맞게 변환하고 연결 | GUI 입력 처리, HTTP 요청 처리, SQL 조회 구현 |
| 애플리케이션 | 계약을 통해 비즈니스 기능 실행 | 금액과 할인율로 할인액 계산 |

같은 할인 계산 포트에 GUI와 자동 테스트가 연결될 수 있다. 할인율 조회 포트에는 실제 DB를 사용하는 adapter와 메모리에서 답하는 테스트용 adapter를 연결할 수 있다. 포트 하나와 adapter 하나가 항상 짝을 이루는 구조는 아니다.

## 입력 포트와 출력 포트: 누가 호출을 시작하는가

이 문서는 애플리케이션을 호출하는 쪽을 입력 포트, 애플리케이션이 외부 기능을 호출하는 쪽을 출력 포트로 부른다. Cockburn 원문의 primary·secondary 구분과 driving·driven 구분에 대응한다.

| 구분 | 호출을 시작하는 쪽 | 계약의 목적 | Adapter의 예 |
|---|---|---|---|
| 입력 포트: primary / driving 측 | 외부 actor | 애플리케이션이 제공하는 기능 실행 | UI, 배치 실행기, 자동 테스트 |
| 출력 포트: secondary / driven 측 | 애플리케이션 | 외부에서 필요한 답을 얻거나 외부에 알림 | DB 접근, 외부 서비스 호출, 테스트 대역 |

출력 포트로 DB를 조회해도 데이터는 애플리케이션 안으로 돌아온다. 입력 포트를 호출해도 결과가 외부로 반환된다. 따라서 데이터가 들어오는지 나가는지보다 대화를 시작하고 주도하는 쪽을 기준으로 읽는다.

```text
UI / 배치 / 테스트
        │ 호출
        ▼
입력 포트 → 애플리케이션의 비즈니스 로직
                          │ 호출
                          ▼
                      출력 포트
                          │ 구현에 연결
                          ▼
                 DB Adapter / 테스트 대역
```

위 그림은 실행 시의 호출 흐름을 단순화한 것이다. 코드의 의존관계와는 구분해서 읽는다. 애플리케이션은 출력 포트의 계약을 사용하고, 구체 DB adapter의 내부 구현을 알지 않도록 구성한다.

## 포트는 반드시 interface 파일이어야 하는가

포트는 설계 개념이고 `interface`는 이를 표현하는 방법이다. Cockburn의 예제에서는 할인 계산을 애플리케이션의 메서드로 노출하고, 할인율 조회는 `RateRepository` 인터페이스로 분리한다. 입력 포트마다 별도의 인터페이스 파일이 필수라는 규칙은 이 예제로부터 나오지 않는다.

반대로 모든 인터페이스가 헥사고날 아키텍처의 포트가 되는 것도 아니다. 그 인터페이스가 애플리케이션과 외부 사이의 어떤 대화를 정의하는지 확인해야 한다. 파일명이나 폴더명만으로 포트 여부를 판단하지 않는다.

## Kotlin으로 보는 계약과 구현

아래는 원문의 할인 계산·할인율 조회 예를 Kotlin으로 재구성한 학습용 예제다. 원문 코드를 그대로 옮긴 것이 아니며, 입력 포트도 명시적인 인터페이스로 표현했다. 특정 Spring 프로젝트의 필수 구조를 뜻하지 않는다.

```kotlin
// 입력 포트: 애플리케이션이 제공하는 기능
interface CalculateDiscount {
    fun discount(amount: Double): Double
}

// 출력 포트: 애플리케이션이 외부에 요구하는 기능
interface RateRepository {
    fun getRate(amount: Double): Double
}

// 입력 포트를 구현하고 출력 포트를 사용하는 애플리케이션
class DiscountService(
    private val rates: RateRepository,
) : CalculateDiscount {
    override fun discount(amount: Double): Double =
        amount * rates.getRate(amount)
}

// 출력 포트를 구현하는 테스트용 Adapter
class FixedRateAdapter(private val rate: Double) : RateRepository {
    override fun getRate(amount: Double): Double = rate
}

// 외부 입력을 받아 입력 포트를 호출하는 Adapter
class TextDiscountAdapter(private val calculator: CalculateDiscount) {
    fun calculate(amountText: String): String =
        calculator.discount(amountText.toDouble()).toString()
}
```

`TextDiscountAdapter`는 문자열 변환을 맡고 `DiscountService`는 계산을 맡는다. 서비스는 할인율을 구할 때 `RateRepository`를 호출하며, 실제 DB나 테스트 대역 중 무엇이 연결됐는지는 알 필요가 없다. 조립하는 쪽에서 구현을 선택해 생성자로 전달한다.

입력 측 adapter는 포트를 **호출**하고, 출력 측 adapter는 애플리케이션이 요구하는 포트를 **구현**한다. 이 예에서 입력 포트의 구현은 `DiscountService`다. 모든 adapter가 입력·출력 포트 인터페이스를 똑같은 방식으로 구현한다고 외우지 않는다.

학습을 위해 문자열 변환 오류 처리와 실제 DB 접근은 생략했다. 금액 타입이나 반올림 정책을 정하는 금융 계산 예제로 사용하지 않는다.

## 포트의 개수와 경계 선택

Cockburn은 모든 use case를 별도 포트로 나누는 극단과 모든 입력·출력을 각각 하나로 합치는 극단 모두 최적이라고 보지 않는다. 대화의 목적에 맞게 경계를 고른다. 육각형을 그린다고 포트가 여섯 개여야 하는 것도 아니다.

외부 기술을 바꾸는 목적, 여러 실행 경로에서 비즈니스 로직을 사용하는 목적, 독립 테스트의 목적 중 어떤 문제를 해결하려는지 먼저 확인한다. 계약과 구현의 구분이 실제 변경과 테스트에 도움이 되는지 판단하며 읽는다.

## 도입의 이점과 유지보수 비용

AWS 가이드는 여러 종류의 클라이언트가 같은 비즈니스 로직을 사용하거나, UI·DB 기술이 바뀌거나, 여러 입력·출력 대상 때문에 복잡성이 커질 때 이 구조를 고려하도록 설명한다. 같은 포트에 REST adapter와 GraphQL adapter를 연결하는 예도 제시한다.

하지만 adapter를 추가하면 유지할 코드와 경계도 늘어난다. AWS는 여러 입력·출력 대상이나 시간에 따른 기술 변경 필요가 없다면 추가 계층이 유지보수 부담이 될 수 있다고 명시한다. 분리를 신중하게 다루지 않으면 복잡성이 커질 수 있고, 추가 계층에 따른 지연 가능성도 언급한다. 성능 영향의 크기나 모든 프로젝트에서의 우수성을 입증한 자료로 읽지는 않는다.

따라서 모든 서비스·클래스에 포트를 기계적으로 추가하기보다, 어떤 외부 기술 의존성을 격리할지와 무엇을 독립적으로 테스트할지를 먼저 정한다. 이는 원문의 목적 중심 경계와 AWS의 비용 설명을 함께 읽어 도출한 실무 판단 기준이다.

## 외부 응답을 그대로 포트로 옮기면 생기는 문제

카카오페이 홈 서버 사례에서는 독립적으로 보호할 도메인 경계가 모호한 상태에서 외부 클라이언트 응답을 기준으로 출력 포트를 작성했다. 그 결과 외부 변경이 내부 계약까지 전파됐고, 신규 기능마다 포트·DTO·adapter와 매핑을 함께 바꾸는 비용이 커졌다고 보고한다.

이는 포트 인터페이스를 추가하는 것과 안정적인 업무상 계약을 설계하는 것이 다르다는 사례다. 포트의 모양을 외부 응답과 똑같이 만들기 전에 애플리케이션이 실제로 요구하는 목적을 확인한다. 이 확인 기준은 원문의 목적 중심 포트와 해당 사례를 함께 읽어 도출한 설명이다.

전체 도입·제거 맥락과 모듈 구성은 [헥사고날 아키텍처 전체 설명](hexagonal-architecture.md)에서 다룬다.

## See Also

- [헥사고날 아키텍처: 구조·의존관계와 적용 기준](hexagonal-architecture.md)

- [객체 지향 설계와 Spring의 SOLID 원칙](../spring/spring-object-oriented-design-and-solid.md)
- [AppConfig와 IoC·DI](../spring/spring-appconfig-ioc-di.md)
- [Spring DB 접근 기술 비교](../spring/spring-database-access-technologies.md)
