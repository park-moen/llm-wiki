# 객체 지향 설계와 Spring의 SOLID 원칙

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편 슬라이드)
> Raw: [객체 지향 설계와 스프링 슬라이드 PDF companion](../../raw/spring/spring-core-object-oriented-design-slides.md); [기본편 회원·주문 예제 PDF companion](../../raw/spring/spring-core-member-order-example.md)
> Updated: 2026-10-11

## Overview

기본편의 첫 단원은 Spring을 좋은 객체 지향 애플리케이션 개발을 돕는 framework로 설명한다. 역할과 구현을 나누면 구현 교체가 쉬워지지만, 클라이언트가 구체 객체를 직접 생성하면 다형성만으로 OCP·DIP를 지킬 수 없다. 생성과 연결을 외부로 분리하는 DI가 필요한 이유를 이해한 뒤, 순수 Java 예제로 그 필요성을 확인하는 흐름이다. 인터페이스를 도입하는 비용도 함께 고려한다.

## Spring의 등장 배경과 핵심 목적

슬라이드는 복잡한 EJB(Enterprise Java Beans) 환경의 대안으로 Spring이 등장했으며, EJB 없이도 확장 가능한 애플리케이션을 만들 수 있음을 보여주는 이야기로 시작한다. Hibernate는 EJB 엔티티 Bean의 대안과 JPA 구현체의 맥락에서 소개한다. JPA라는 표준 인터페이스와 Hibernate·EclipseLink 같은 구현체의 관계도 역할과 구현을 구분하는 예다.

강의가 강조하는 Spring의 핵심은 웹·DB 기능 목록보다 객체 지향 설계를 실현하기 쉽게 해주는 데 있다. IoC·DI와 DI container가 객체의 역할은 유지하면서 실제 구현을 조립하고 교체하도록 지원한다.

## Spring Framework와 Spring Boot의 역할

| 용어 | 슬라이드에서 설명하는 역할 |
|---|---|
| Spring DI container | 객체 생성·관리와 의존관계 연결을 지원하는 핵심 기술 |
| Spring Framework | DI, AOP, 이벤트, 웹, 데이터 접근과 테스트 등 기반 기능 제공 |
| Spring Boot | 실행 가능한 애플리케이션 생성, 내장 서버, starter와 자동 구성 등으로 Framework 사용을 편리하게 함 |
| Spring 생태계 | Framework·Boot와 Data·Security 등 여러 프로젝트를 포괄 |

따라서 Spring이라는 말이 무엇을 가리키는지는 문맥으로 구분한다. Boot의 프로젝트 준비 편의 기능과 DI가 해결하는 객체 조립 문제는 연결되지만 같은 역할은 아니다.

자료의 릴리스 현황은 2020년 9월 시점의 설명이다. 그 버전 목록이나 당시 현재라는 표현을 오늘의 설치 안내로 사용하지 않는다. 이 문서는 역사적 배경과 설계 개념을 정리한다.

## 역할·구현 분리와 객체의 협력

객체 지향 프로그래밍은 객체들이 메시지를 주고받으며 협력하는 구조로 설명한다. 한 객체는 다른 객체에 요청하는 클라이언트이면서, 다른 요청에는 응답하는 서버 역할을 할 수 있다.

Java에서 역할은 인터페이스, 구현은 이를 구현한 클래스와 실제 객체로 표현한다. 클라이언트는 역할의 계약을 알고 호출하며 구현 내부를 알 필요가 없다. 인터페이스를 구현한 다른 객체를 연결하면 같은 호출 코드로 다른 동작을 사용할 수 있다. 이는 메서드 overriding과 다형성을 활용하는 관계다.

단, 역할의 계약 자체가 바뀌면 클라이언트와 구현 모두 영향을 받을 수 있다. 인터페이스를 만드는 것뿐 아니라 안정적인 계약을 설계하는 것이 중요하다.

## SOLID의 판단 기준

| 원칙 | 강의의 설명 | 설계에서 확인할 점 |
|---|---|---|
| SRP: 단일 책임 원칙 | 클래스가 하나의 책임을 가지며 책임의 범위는 문맥에 따라 다름 | 변경 시 파급 효과가 작고 생성·사용 책임이 구분되는가 |
| OCP: 개방-폐쇄 원칙 | 확장을 허용하면서 기존 코드의 변경은 제한 | 새 구현을 추가할 때 클라이언트까지 수정해야 하는가 |
| LSP: 리스코프 치환 원칙 | 하위 타입으로 바꿔도 프로그램의 정확성을 깨뜨리지 않음 | 컴파일뿐 아니라 인터페이스의 행동 규약을 지키는가 |
| ISP: 인터페이스 분리 원칙 | 특정 클라이언트용 인터페이스를 여러 개 두는 편이 범용 인터페이스 하나보다 나음 | 각 클라이언트에 필요한 역할이 분리되는가 |
| DIP: 의존관계 역전 원칙 | 구체 구현보다 추상화에 의존 | 필드 타입뿐 아니라 생성 코드에서도 구체 클래스 의존이 남아 있는가 |

### LSP는 문법보다 행동의 계약

강의의 자동차 예에서는 가속 기능이 앞으로 가야 한다는 계약을 가진다. 구현을 교체했을 때 뒤로 가면 그 계약을 깨뜨린다. 인터페이스를 구현하고 컴파일에 성공했다는 사실만으로 올바른 대체 가능성이 보장되지는 않는다.

### ISP는 클라이언트가 사용하는 역할의 분리

자동차의 운전 인터페이스와 정비 인터페이스를 분리하면 정비 기능의 변경이 운전자에게 영향을 주지 않을 수 있다. 기준은 메서드 수를 줄이는 것 자체보다 서로 다른 클라이언트가 사용하는 계약을 나누는 데 있다.

## 다형성만으로 OCP·DIP를 지킬 수 없는 이유

슬라이드의 MemberService는 MemberRepository 타입을 사용하면서 `new MemoryMemberRepository()`로 구현을 직접 선택한다. 저장소를 JdbcMemberRepository로 바꾸려면 서비스의 생성 코드를 수정해야 한다.

```java
MemberRepository m = new MemoryMemberRepository();
// 구현을 교체하려면 클라이언트 코드도 수정한다.
MemberRepository m = new JdbcMemberRepository();
```

위 두 줄은 변경 전후를 비교한 예이며 함께 선언해 실행하는 코드는 아니다. 추상 타입을 사용해도 구체 생성 의존이 남아 있으므로 DIP 문제가 있고, 구현 확장이 서비스 수정으로 이어지므로 OCP 문제도 있다.

해결하려면 객체를 만들고 연결하는 별도 조립자가 필요하다. 서비스는 전달받은 역할을 사용하고 조립자가 구현을 선택한다. 기본편의 [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)는 이 원리를 회원·주문 코드에 적용하는 과정이다.

## 인터페이스 도입의 비용

슬라이드는 이상적으로 역할과 구현을 나누는 방향을 제시하면서도, 인터페이스 도입에 추상화 비용이 따른다고 설명한다. 확장할 가능성이 없다면 구체 클래스를 사용하고 실제 필요가 생겼을 때 인터페이스를 도입하는 것도 방법이다.

따라서 모든 클래스에 인터페이스를 의무적으로 추가하기보다 변경 가능성과 교체 필요성을 확인한다. 기본편에서는 저장소와 할인 정책의 변경 가능성이 요구사항에 있으므로 그 경계를 인터페이스로 분리한다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [기본편 회원·주문 도메인 설계](spring-core-member-order-design.md)
- [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring Boot Starter와 의존성 구조](spring-boot-starter-dependencies.md)
- [Frontend에서 OCP와 의존성 역전을 적용하는 실무 가이드](../software-career/frontend-ocp-and-dependency-inversion-practical-guide-2026-08-24.md)
- [헥사고날 아키텍처의 포트: 계약과 Adapter의 역할](../software-career/hexagonal-architecture-ports.md)
