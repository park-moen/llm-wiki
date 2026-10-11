# 할인 정책 확장과 단위 테스트

> Sources: Inflearn Spring 강의 사용자 제공 스크립트, Unknown; 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [새로운 할인 정책 개발 강의 원문](../../raw/spring/spring-rate-discount-policy-development-lecture.md); [3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용 PDF companion](../../raw/spring/spring-core-object-oriented-principles-application.md)
> Updated: 2026-10-11

## Overview

고정 할인에서 주문 금액에 비례하는 정률 할인으로 요구사항이 바뀌면, 기존 할인 정책 인터페이스에 새 구현을 추가할 수 있다. 기본편 PDF는 이어서 실제 정책 적용의 문제와 AppConfig를 통한 해결도 설명한다. 이번 강의는 `RateDiscountPolicy`를 개발하고 VIP와 비VIP의 할인 금액을 단위 테스트로 검증한다. 할인 책임을 별도로 분리하면 주문 전체를 실행하지 않고도 할인 규칙을 직접 테스트하기 쉽다. 다만 구현 추가와 실제 주문 코드에 정책을 적용하는 과정은 구분해야 한다.

## 고정 할인에서 정률 할인으로 변경

기존 `FixDiscountPolicy`는 VIP에게 주문 금액과 관계없이 천원을 할인한다. 새 요구사항은 VIP에게 주문 금액의 10%를 할인하는 것이다.

| 주문 금액 | 기존 고정 할인 | 새 정률 할인 |
|---|---|---|
| 만원 | 천원 | 천원 |
| 2만원 | 천원 | 2천원 |
| 3만원 | 천원 | 3천원 |

기존 `DiscountPolicy` 인터페이스를 유지하면서 이를 구현하는 `RateDiscountPolicy`를 추가한다. 이 단계에서는 새 할인 규칙 자체를 개발하고 검증한다.

## 할인 계산의 계약

`discount(member, price)`는 최종 결제 금액이 아니라 **할인 금액**을 반환한다.

- 회원 등급이 VIP이면 주문 금액의 10%를 반환한다.
- 회원 등급이 BASIC이면 할인이 적용되지 않으므로 0원을 반환한다.

강의에서 설명한 VIP 계산식은 다음과 같다. `discountPercent`는 강의에서 할인율을 설명한 이름을 Java 식으로 정리한 표기다.

```java
price * discountPercent / 100
```

만원 주문을 넣었을 때 VIP의 할인 금액은 1000원이어야 한다. BASIC은 같은 주문 금액을 넣어도 0원이어야 한다.

## JUnit과 AssertJ로 할인 규칙 검증

`RateDiscountPolicyTest`를 만들고 JUnit 5를 선택한다. `@DisplayName`을 사용하면 테스트 결과에 한글로 검증 의도를 표시할 수 있다.

Given–When–Then 구조로 준비, 실행, 검증을 구분한다.

| 단계 | VIP 테스트 | 비VIP 테스트 |
|---|---|---|
| Given | VIP 등급 회원 준비 | BASIC 등급 회원 준비 |
| When | 만원을 전달해 할인 계산 | 만원을 전달해 할인 계산 |
| Then | 할인 금액이 1000원인지 검증 | 할인 금액이 0원인지 검증 |

AssertJ의 `assertThat(discount).isEqualTo(...)`로 결과를 비교한다. `org.assertj`의 `Assertions`를 사용하며, static import를 적용하면 assertion을 클래스 이름 없이 작성할 수 있다.

### 할인 제외 조건도 정상 규칙으로 검증

강의에서 말하는 실패 테스트는 여기서는 **VIP가 아닌 경우에 할인이 적용되지 않는지 확인하는 테스트**다. 예외 발생을 검증하는 테스트는 아니다. BASIC에게 0원을 반환하면 테스트는 통과해야 한다.

강의는 BASIC 테스트의 기대값을 일부러 천원으로 두고 실행해, 기대값은 천원인데 실제 값은 0원이라는 assertion 실패를 확인한다. 기대값을 0원으로 바꾼 뒤에는 테스트가 통과한다. 이는 할인 제외 조건뿐 아니라 assertion이 결과 차이를 잡아내는지도 확인하는 과정이다.

메서드 단위로 개별 테스트를 실행할 수 있고, 클래스 단위로 실행하면 VIP와 BASIC 테스트를 함께 검증할 수 있다.

## 책임 분리와 테스트 범위

할인 정책이 할인 계산만 담당하므로 회원 등급과 주문 금액을 준비해 규칙을 직접 검증할 수 있다. 강의는 단순해 보이는 금액 계산도 의심하고 테스트할 것을 강조하며, 실무에서는 경계값 테스트 등 더 많은 검증이 필요하다고 설명한다.

이번 원문에서는 VIP와 BASIC 사례를 검증했으며, 경계값이나 금액 처리의 세부 규칙은 구현하지 않았다.

## 인터페이스 분리 이후에도 확인할 설계 문제

강의 도입부는 역할과 구현을 나눴더라도 새 정책을 실제로 적용하면 DIP·OCP를 지키지 못하는 문제가 드러날 수 있다고 예고한다. 따라서 새 구현을 추가하고 단위 테스트를 통과했다는 사실만으로 전체 주문 설계가 변경에 유연하다고 결론 내릴 수 없다.

처음 제공된 강의 스크립트는 정책 개발과 테스트까지 다루고 실제 정책 적용과 Spring 전환을 예고한다. 이후 보관한 「객체 지향 원리 적용」 PDF에는 실제 적용에서 발생하는 위반 코드와 AppConfig로 해결하는 과정이 포함되어 있다.

## PDF에서 이어지는 정책 적용과 해결

초기 `OrderServiceImpl`은 `DiscountPolicy` 타입 필드를 선언하면서도 `new FixDiscountPolicy()`를 직접 호출한다. 이를 `new RateDiscountPolicy()`로 바꾸면 주문 서비스 코드도 수정해야 하므로 DIP·OCP 문제가 드러난다.

정책을 직접 생성하는 코드를 제거하고 생성자로 주입받게 바꾼다. AppConfig가 실제 정책 객체를 생성하고 주문 서비스에 연결하면 `discountPolicy()`의 반환 구현만 교체해 주문 서비스는 유지할 수 있다. 이후 `@Configuration`, `@Bean`을 붙여 Spring container로 전환한다. 전체 과정은 [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)에 정리했다.

## IntelliJ IDEA 단축키

원문에서 설명한 단축키는 다음과 같다.

| 작업 | macOS | Windows |
|---|---|---|
| 테스트 생성 메뉴 열기 | `Command + Shift + T` | `Ctrl + Shift + T` |
| static import 제안 메뉴 열기 | `Option + Enter` | `Alt + Enter` |

테스트 생성 메뉴에서 `Create New Test`를 선택하고 JUnit 5와 대상 클래스 이름 뒤에 `Test`를 붙인 이름을 사용한다. 원문의 음성 인식 표기는 보관하고, 이 문서에서는 문맥에 따라 `DiscountPolicy`, `FixDiscountPolicy`, `RateDiscountPolicy`, `assertThat` 등 코드 이름을 정리했다.

## See Also

- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](../software-career/frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md)
- [기본편 회원·주문 도메인 설계](spring-core-member-order-design.md)
- [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)
