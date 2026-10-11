# AppConfig와 IoC·DI

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용 PDF companion](../../raw/spring/spring-core-object-oriented-principles-application.md); [객체 지향 설계와 스프링 슬라이드 PDF companion](../../raw/spring/spring-core-object-oriented-design-slides.md)
> Updated: 2026-10-11

## Overview

AppConfig는 구현 객체를 생성하고 생성자로 연결하는 책임을 서비스에서 분리한다. 서비스는 인터페이스를 호출해 기능을 수행하고, 어떤 구현을 사용할지는 구성 영역에서 결정한다. 이 분리로 SRP·DIP·OCP를 적용한 뒤, 같은 조립 정보를 Spring container에 넘기는 방식으로 전환한다.

## 인터페이스 필드만으로 DIP가 완성되지 않는 이유

초기 주문 서비스의 `DiscountPolicy` 필드는 인터페이스 타입이지만 `new FixDiscountPolicy()`로 구현체를 직접 생성한다. `RateDiscountPolicy`를 적용하려면 `OrderServiceImpl`도 수정해야 한다. 구현 클래스에 의존하는 것은 DIP 위반이며, 정책 확장이 사용 영역의 수정으로 이어지는 것은 이 예제의 OCP 위반이다.

직접 생성을 지우기만 하고 인터페이스 필드를 남기면 실제 객체가 연결되지 않아 실행 시 null 참조 문제가 생긴다. 해결하려면 외부에서 구현체를 생성해 전달해야 한다.

## 생성과 실행의 책임 분리

`MemberServiceImpl`은 `MemberRepository`를, `OrderServiceImpl`은 `MemberRepository`와 `DiscountPolicy`를 생성자 매개변수로 받는다. 서비스는 전달받은 객체를 사용하고, AppConfig가 실제 구현을 고른다.

역할별 factory 메서드로 구성을 정리하면 저장소 생성 코드의 중복을 줄이고 구현 교체 지점을 찾기 쉽다. 아래는 PDF의 정률 할인 적용 후 구성을 필요한 부분만 옮긴 예다.

```java
public class AppConfig {
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

이 단계는 순수 Java 구성이며 메서드를 호출할 때마다 생성 코드가 실행된다. 객체 공유를 보장하는 [Spring singleton 처리](spring-singleton-and-configuration.md)와 구분해야 한다.

## 적용되는 설계 원칙

| 원칙 | 이 예제에서의 효과 |
|---|---|
| SRP: 단일 책임 원칙 | 서비스는 실행, AppConfig는 생성과 연결 담당 |
| DIP: 의존관계 역전 원칙 | 서비스가 구체 저장소·할인 구현 대신 인터페이스에 의존 |
| OCP: 개방-폐쇄 원칙 | 할인 구현 교체 시 구성 영역을 변경하고 기존 주문 서비스 유지 |

OCP를 적용해도 구성 영역은 바뀐다. 이 예제의 목표는 새 정책을 추가할 때 사용 영역의 변경을 막는 것이다.

## 책임의 범위와 인터페이스 도입 판단

첫 단원 슬라이드는 SRP의 책임 범위가 문맥에 따라 달라지며, 변경 시 파급 효과가 적은지를 기준으로 판단한다고 설명한다. AppConfig의 분리는 클래스 수를 늘리는 데 목적이 있는 것이 아니라 생성·연결과 기능 실행의 변경을 나누는 것이다.

또한 인터페이스에는 추상화 비용이 있으므로 확장 가능성이 없으면 구체 클래스를 사용하다 필요할 때 분리할 수 있다고 설명한다. 모든 객체에 동일한 형식을 강제하지 않고, 이 예제처럼 교체 가능한 저장소와 할인 정책의 경계를 먼저 확인한다. 전체 원칙은 [객체 지향 설계와 SOLID](spring-object-oriented-design-and-solid.md)에 정리했다.

## IoC와 DI의 차이

IoC(Inversion of Control, 제어의 역전)는 객체가 직접 생성·연결을 결정하던 흐름을 외부 구성자가 관리하는 것이다. PDF는 내가 만든 코드를 외부에서 제어해 실행하는 framework와, 내 코드가 흐름을 제어하면서 호출하는 library를 구분한다.

DI(Dependency Injection, 의존관계 주입)는 외부에서 실제 구현 객체를 생성하고 그 참조를 전달해 연결하는 것이다. 정적인 클래스 의존관계는 코드에 나타나는 인터페이스·타입 관계이며, 동적인 객체 의존관계는 실행 시 연결된 실제 인스턴스 관계다. DI로 정적 의존관계를 유지하면서 실제 구현을 바꿀 수 있다.

객체를 생성·관리하고 의존관계를 연결하는 AppConfig 같은 역할을 IoC container 또는 DI container라고 부른다.

## Spring container로 전환

AppConfig에 `@Configuration`을 붙이고 factory 메서드에 `@Bean`을 붙인다. 이후 직접 `appConfig.memberService()`를 호출하던 코드를 container 조회로 바꾼다.

```java
ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
MemberService memberService =
        applicationContext.getBean("memberService", MemberService.class);
```

container가 구성 정보를 읽고 객체를 Bean으로 등록하며 의존관계를 연결한다. 기본 Bean 이름은 `@Bean` 메서드 이름이다. 비즈니스 실행 코드 대신 조립과 조회 방식이 바뀌는 과정이다.

## 자료 버전에 따른 로그 안내

PDF는 Spring Boot 3.1 이상에서 `MemberApp`, `OrderApp`처럼 ApplicationContext를 직접 만드는 예제의 DEBUG 로그가 보이지 않을 수 있다고 설명한다. `src/main/resources/logback.xml`에서 DEBUG를 설정하는 예를 제시하되, Spring Boot로 실행하는 `CoreApplication`에서는 파일을 제거하거나 INFO로 바꾸도록 구분한다. 이 안내는 자료의 실습 환경을 설명하며 현재 제품의 모든 로그 설정으로 일반화하지 않는다.

## See Also

- [기본편 회원·주문 도메인 설계](spring-core-member-order-design.md)
- [할인 정책 확장과 단위 테스트](discount-policy-extension-and-testing.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring container와 Bean 조회](spring-container-bean-lookup.md)
- [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](../software-career/frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md)
- [객체 지향 설계와 Spring의 SOLID 원칙](spring-object-oriented-design-and-solid.md)
