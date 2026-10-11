# Spring Bean scope와 Provider

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [9. 빈 스코프 PDF companion](../../raw/spring/spring-core-bean-scopes.md)
> Updated: 2026-10-11

## Overview

Scope는 Bean을 생성·공유하고 관리하는 범위다. singleton은 같은 객체를 재사용하고, prototype은 container 조회마다 새 객체를 만든다. 수명이 다른 Bean을 함께 사용할 때는 주입 시점과 실제 사용 시점을 구분해야 한다. Provider와 scoped proxy는 실제 대상 조회를 필요한 시점까지 늦추는 방법이다.

## scope별 관리 범위

| scope | 생성·공유 범위 | 종료 관리 |
|---|---|---|
| singleton | 같은 container에서 Bean 공유; 기본 scope | container 종료 시 |
| prototype | container에 요청할 때마다 새 객체 | 반환 이후에는 클라이언트가 관리 |
| request | HTTP 요청별 객체 | 해당 요청의 종료 시 |
| session | HTTP session 범위 | 해당 scope 종료 시 |
| application | ServletContext 범위 | 해당 scope 종료 시 |
| websocket | WebSocket 범위 | 해당 scope 종료 시 |

`@Scope("prototype")`처럼 클래스 또는 Bean 등록 메서드에 지정할 수 있다. 웹 scope는 웹 환경에서 동작하며, prototype과 달리 Spring이 해당 scope의 종료까지 관리한다.

## prototype의 초기화와 종료

container는 prototype을 생성하고 의존관계를 주입하며 초기화 콜백까지 처리한다. 반환된 객체의 이후 사용과 종료는 관리하지 않으므로 container를 닫아도 prototype의 PreDestroy가 자동 실행되지 않는다. 필요한 종료 작업은 받은 쪽이 처리한다.

## singleton에 prototype을 주입해도 매번 새로 생기지 않는 이유

singleton 생성 시 prototype을 주입하면 그 시점에 새로운 객체가 만들어진다. 하지만 singleton은 그 참조를 필드에 저장한다. 이후 메서드를 다시 호출해도 이미 연결된 같은 prototype을 사용한다.

기본편의 count 예제에서는 서로 다른 호출자가 같은 singleton을 통해 prototype의 count를 증가시켜 값이 누적된다. 다른 singleton마다 주입받는 prototype은 다를 수 있지만, 한 singleton이 사용할 때마다 새로 조회하는 것은 아니다.

## Provider로 사용 시점에 조회하기

container를 직접 주입받아 `getBean()`을 호출하면 새 prototype을 얻을 수 있지만 container 전체에 의존하게 된다. 필요한 타입의 조회 기능만 받는 ObjectProvider를 사용하면 의존성을 좁힐 수 있다.

| 도구 | 조회 | 특징 |
|---|---|---|
| ObjectFactory | `getObject()` | Spring의 단순 조회 기능 |
| ObjectProvider | `getObject()` | ObjectFactory를 상속하고 선택적 조회·stream 등 편의 기능 제공 |
| Provider | `get()` | 표준 API; 별도 의존성 필요 |

직접 필요한 객체를 찾는 것을 DL(Dependency Lookup, 의존관계 조회)이라고 한다. prototype의 Provider에서 매번 조회하면 매번 새 객체를 받는다. Provider 자체가 모든 scope의 객체를 새로 만드는 것은 아니며, 실제 결과는 대상 scope에 따른다.

PDF는 Spring Boot 3.0 미만의 `javax.inject.Provider`와 3.0 이상의 `jakarta.inject.Provider`를 구분한다. 별도 라이브러리 안내도 자료 버전에 따른 내용으로 읽는다.

## request scope와 시작 시점 문제

기본편의 MyLogger는 request scope에 UUID와 requestURL을 보관해 요청별 로그를 구분한다. 같은 요청에서 Controller와 Service가 사용하면 같은 로거 객체를 얻고, 다른 요청은 분리된다.

애플리케이션 시작 시 singleton에 request Bean을 직접 주입하려 하면 아직 HTTP 요청이 없어 실제 대상을 만들 수 없다. ObjectProvider를 주입하고 요청 처리 중 `getObject()`를 호출하면 대상 조회를 지연할 수 있다. 같은 요청에서 조회하면 같은 request Bean을 얻는다.

자료는 로거의 requestURL 설정을 단순화를 위해 Controller에 두되, 웹 처리에 익숙해지면 공통 interceptor나 filter로 옮길 수 있다고 설명한다. 웹 요청 정보를 Service의 매개변수로 계속 전달하는 것과 scope가 관리하는 로거를 사용하는 방식의 차이를 보여준다.

## scoped proxy

클래스 기반 대상에는 다음과 같이 proxyMode를 지정한다.

```java
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
```

인터페이스 기반 대상에는 `INTERFACES`를 선택한다. 실제 request 객체 대신 proxy를 미리 주입하고, 메서드를 호출할 때 proxy가 현재 scope의 실제 객체로 위임한다. 클래스 기반 예제에서는 CGLIB로 대상 클래스의 하위 proxy를 만든다.

Provider와 proxy의 공통 원리는 실제 객체 조회를 필요한 시점까지 지연하는 것이다. proxy가 singleton처럼 주입된다는 사실이 대상 request 객체도 모든 요청에서 공유된다는 뜻은 아니다. 특별한 scope는 필요한 곳에 제한해서 사용한다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [Spring singleton과 Configuration](spring-singleton-and-configuration.md)
- [Spring Bean 초기화와 종료 콜백](spring-bean-lifecycle.md)
- [Spring 자동 주입과 Bean 선택](spring-autowiring-and-bean-selection.md)
