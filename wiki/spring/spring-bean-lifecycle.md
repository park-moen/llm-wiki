# Spring Bean 초기화와 종료 콜백

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [8. 빈 생명주기 콜백 PDF companion](../../raw/spring/spring-core-bean-lifecycle.md); [2. 스프링 핵심 원리 이해1 - 예제 만들기 PDF companion](../../raw/spring/spring-core-member-order-example.md); [0. 스프링 핵심 원리 - 기본편 v2023-12-04 PDF companion](../../raw/spring/spring-core-course-overview.md); [9. 빈 스코프 PDF companion](../../raw/spring/spring-core-bean-scopes.md)
> Updated: 2026-10-11

## Overview

외부 연결처럼 의존성 설정이 필요한 초기화는 객체 생성 이후, 필요한 의존관계가 모두 연결된 시점에 수행한다. Spring은 초기화·종료 콜백을 제공하며 직접 작성한 객체에는 annotation을, 수정할 수 없는 외부 라이브러리에는 Bean 등록 설정을 사용할 수 있다. 실제 종료 관리 여부는 Bean scope에 따라 달라진다.

## 생성과 초기화의 차이

기본편의 NetworkClient는 생성자에서 연결을 시도하지만 URL은 객체 생성 이후 setter로 들어온다. 따라서 생성자에서 URL을 사용하면 값이 없는 상태다. 예제는 실제 네트워크 연결 대신 메시지 출력으로 순서를 보여준다.

생성자는 필수 정보를 받아 객체를 만들고, 초기화는 준비된 정보를 이용해 연결 등 작업을 수행한다. 무거운 초기화는 분리하는 편이 유지보수하기 쉽지만 간단한 내부 값 설정은 생성자에서 처리할 수 있다.

## 기본 lifecycle 흐름

container 생성 → Bean 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸 전 콜백 → container 종료 순으로 설명한다. 생성자 주입은 객체 생성과 연결이 함께 일어날 수 있으므로 핵심 기준은 모든 필요한 설정이 끝난 뒤 초기화한다는 것이다.

기본 singleton Bean은 container 종료 시 종료 콜백을 받는다. prototype은 이 흐름 전체를 container가 관리하지 않으며, [Bean scope](spring-bean-scopes-and-provider.md)에서 종료 책임을 구분한다.

## 콜백 방식 비교

| 방식 | 초기화·종료 | 장단점 |
|---|---|---|
| Spring 인터페이스 | InitializingBean.afterPropertiesSet(), DisposableBean.destroy() | Spring에 의존하고 메서드 이름이 고정됨 |
| Bean 등록 설정 | `@Bean(initMethod = "init", destroyMethod = "close")` | 이름을 지정할 수 있고 수정 불가능한 외부 객체에도 적용 가능 |
| annotation | `@PostConstruct`, `@PreDestroy` | 직접 작성한 클래스에 간단히 적용하고 component scan과 함께 사용하기 좋음 |

자료는 일반적으로 PostConstruct·PreDestroy를 사용하고, 수정할 수 없는 외부 라이브러리는 initMethod·destroyMethod로 설정하는 기준을 제시한다.

## 종료 메서드 추론

`@Bean`의 destroyMethod 기본값은 `(inferred)`다. 자료는 `close`, `shutdown`이라는 종료 메서드를 추론해 호출한다고 설명한다. 추론을 끄려면 `destroyMethod=""`로 지정한다. 이는 container가 종료를 관리하는 Bean의 설정이며 prototype 자동 종료를 보장하지 않는다.

## 자료의 annotation 패키지 전환

생명주기 PDF의 예시는 `javax.annotation.PostConstruct`를 사용한다. 같은 자료 묶음의 프로젝트 생성 안내는 Spring Boot 3.0 이상에서 `jakarta.annotation.PostConstruct`로 변경하라고 명시한다.

> **Status: Outdated** (2023-12-04)
> javax.annotation 표기는 이전 환경의 예시다. 기본편의 전체 버전 안내와 프로젝트 생성 PDF를 함께 읽어 Jakarta annotation 전환을 적용한다. 콜백의 역할과 import 경로의 버전 차이는 구분한다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring Bean scope와 Provider](spring-bean-scopes-and-provider.md)
- [기본편 회원·주문 도메인 설계](spring-core-member-order-design.md)
