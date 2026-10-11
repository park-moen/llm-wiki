# Spring singleton과 Configuration

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [5. 싱글톤 컨테이너 PDF companion](../../raw/spring/spring-core-singleton-container.md)
> Updated: 2026-10-11

## Overview

Spring container는 기본 singleton Bean을 공유해 요청마다 같은 서비스 객체를 다시 만들지 않도록 한다. 클래스에 직접 singleton 패턴을 구현하는 대신 container가 인스턴스를 관리한다. 공유 객체의 필드에 사용자별 변경 값을 저장하면 요청 사이에 값이 섞일 수 있으며, Java 구성의 Bean 메서드 호출에도 별도의 보장이 필요하다.

## singleton 패턴과 container의 관리 구분

순수 Java AppConfig는 메서드를 호출할 때마다 새 객체를 만들 수 있다. 직접 singleton 패턴을 구현하면 static 인스턴스, private 생성자와 `getInstance()`로 공유 객체를 얻지만 구체 클래스 의존, 테스트·확장 제약 등이 생긴다.

Spring은 singleton registry로 객체를 관리하므로 서비스마다 이 패턴을 직접 구현할 필요가 없다. 기본 Bean을 같은 container에서 조회하면 같은 인스턴스를 사용한다. Spring은 singleton 외에 다른 scope도 지원한다.

## 공유 Bean의 변경 가능한 상태

여러 클라이언트가 동일한 Bean을 사용하므로 특정 사용자나 요청에 종속된 변경 가능한 필드를 두지 않는다. 주문 금액을 `price` 필드에 저장하면 다른 사용자의 주문이 그 값을 덮어쓰고 첫 사용자가 잘못된 금액을 읽을 수 있다.

PDF의 사례에서는 사용자A의 주문금액이 10000원이어야 하는데 사용자B의 값인 20000원이 나왔다. 실제 thread를 사용하지 않고 호출 순서만으로 공유 필드의 위험을 보여준다.

요청별 값은 메서드 매개변수와 지역변수로 처리하고 계산 결과를 반환한다. 공유 객체라는 사실이 상태 변경의 안전성까지 보장하는 것은 아니다.

## Configuration이 Bean 메서드 호출을 처리하는 이유

`memberService()`와 `orderService()`가 각각 `memberRepository()`를 호출해도, PDF의 `@Configuration` 예제에서는 같은 저장소 인스턴스가 연결된다. Spring이 CGLIB로 구성 클래스의 하위 클래스를 만들어 Bean 메서드 호출을 처리하기 때문이다.

등록된 Bean이 있으면 기존 객체를 반환하고 없으면 생성·등록하는 방식으로 설명한다. PDF의 예상 코드는 원리를 보여주는 의사 코드이며 Spring 내부 구현을 그대로 옮긴 코드는 아니다.

## Bean만 붙였을 때의 직접 호출

같은 예제에서 `@Configuration`을 제거하고 `@Bean`만 남기면 Bean 자체는 등록된다. 하지만 구성 클래스 안에서 `memberRepository()`를 직접 호출하는 부분은 일반 Java 호출이 되어 새 객체를 만들 수 있다. 그 객체는 등록된 저장소 Bean과 서로 다를 수 있다.

이때 보장되지 않는 것은 **구성 코드의 직접 메서드 호출에서 같은 Bean을 재사용하는 동작**이다. `@Bean`으로 등록된 객체의 기본 scope가 전부 singleton이 아니게 된다는 뜻으로 확대하지 않는다. PDF도 마지막에 직접 호출의 경우라고 범위를 설명한다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Bean scope와 Provider](spring-bean-scopes-and-provider.md)
- [Kotlin 객체](../kotlin/kotlin-objects.md)
