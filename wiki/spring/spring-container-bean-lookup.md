# Spring container와 Bean 조회

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [4. 스프링 컨테이너와 스프링 빈 PDF companion](../../raw/spring/spring-core-container-and-beans.md)
> Updated: 2026-10-11

## Overview

Spring container는 구성 정보를 읽어 Bean을 등록하고 의존관계를 연결한다. ApplicationContext는 BeanFactory의 관리·조회 기능과 애플리케이션용 부가 기능을 제공한다. Bean 조회는 이름, 타입, 상속 관계에 따라 후보가 달라지므로 대상이 없거나 여러 개인 상황을 구분해야 한다.

## container 생성과 등록

Java 구성은 `new AnnotationConfigApplicationContext(AppConfig.class)`로 읽는다. ApplicationContext는 인터페이스이고 AnnotationConfigApplicationContext는 그 구현체다. 전달한 구성 정보를 바탕으로 Bean을 등록하고 의존관계를 설정한다.

`@Bean` 메서드 이름이 기본 Bean 이름이며 `@Bean(name="memberService2")`처럼 지정할 수도 있다. 같은 이름을 중복 사용하면 설정에 따라 덮어쓰기나 오류가 발생할 수 있으므로 이름을 구분한다.

등록과 의존관계 설정은 개념적으로 구분하지만 생성자 주입은 생성과 연결이 함께 일어날 수 있다. 별개 단계라는 설명을 모든 주입 방식의 고정된 실행 순서로 이해하지 않는다.

## 이름·타입·상속 관계로 조회하기

| 조회 | 의미와 주의점 |
|---|---|
| `getBean(name, type)` | 이름과 타입으로 특정 Bean 조회 |
| `getBean(type)` | 해당 타입의 Bean 조회; 후보가 여럿이면 단일 대상을 선택할 수 없음 |
| `getBeansOfType(type)` | 해당 타입의 모든 Bean 조회 |
| 부모 타입 조회 | 그 타입을 구현·상속한 Bean도 조회 후보에 포함 |
| `Object` 타입 조회 | 모든 Bean 조회 |

조회 대상이 없으면 `NoSuchBeanDefinitionException`이 발생한다. 같은 타입이 여럿이면 이름을 함께 지정하거나 전체 목록을 조회한다. 구현 타입을 직접 지정하면 구현 교체 시 유연성이 떨어질 수 있다. 자동 주입에서 후보를 고르는 방법은 [Bean 선택](spring-autowiring-and-bean-selection.md)에서 다룬다.

## 애플리케이션 Bean과 내부 Bean 구분

`getBeanDefinitionNames()`로 등록된 이름을 얻고 `getBean()`으로 객체를 조회한다. BeanDefinition의 `getRole()`로 역할을 구분할 수 있다.

- `ROLE_APPLICATION`: 사용자가 정의한 일반적인 Bean
- `ROLE_INFRASTRUCTURE`: Spring이 내부에서 사용하는 Bean

전체 등록 목록과 직접 작성한 Bean 목록을 구분해 확인할 때 유용하다.

## BeanFactory와 ApplicationContext

BeanFactory는 Bean을 관리하고 `getBean()`으로 조회하는 최상위 인터페이스다. ApplicationContext는 이를 상속하고 메시지 국제화, 환경 정보, 이벤트 발행·구독, 파일·classpath 등의 리소스 조회 기능을 제공한다. PDF는 부가 기능이 필요한 일반 애플리케이션에서 ApplicationContext를 사용한다고 설명한다.

## 구성 형식과 BeanDefinition

Java 설정은 AnnotationConfigApplicationContext, XML 설정은 GenericXmlApplicationContext로 읽을 수 있다. 입력 형식이 달라도 reader가 BeanDefinition이라는 공통 메타 정보를 만들고 container가 이를 바탕으로 Bean을 생성한다.

BeanDefinition에는 생성할 클래스 또는 factory Bean·메서드, scope, 지연 생성 여부, 초기화·종료 메서드, 생성자 인수와 속성 정보 등이 들어간다. `@Bean` factory 방식과 직접 클래스 등록 방식의 정보 구성은 다를 수 있다.

핵심은 XML·Java 형식 자체와 Bean 생성 정보를 분리하는 추상화다. PDF는 BeanDefinition을 직접 정의하는 상세 구현까지 외우기보다 이 구조를 이해하는 것으로 충분하다고 설명한다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [AppConfig와 IoC·DI](spring-appconfig-ioc-di.md)
- [자동 주입과 Bean 선택](spring-autowiring-and-bean-selection.md)
