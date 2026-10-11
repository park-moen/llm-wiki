# Spring 자동 주입과 Bean 선택

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [7. 의존관계 자동 주입 PDF companion](../../raw/spring/spring-core-autowiring.md)
> Updated: 2026-10-11

## Overview

자동 주입은 타입을 기준으로 의존성을 찾고, 여러 후보가 있으면 추가 규칙으로 사용할 Bean을 고른다. 기본편은 생성자 주입을 기본으로 사용하며 선택적 의존성, Qualifier·Primary, 모든 구현을 모으는 List·Map 주입과 자동·수동 등록의 선택 기준까지 설명한다.

## 주입 방법 선택

| 방식 | 용도와 제약 |
|---|---|
| 생성자 | 필수 의존성을 생성 시 확정하고 final 필드로 표현 |
| setter | 선택적이거나 변경 가능한 의존성; public 변경 경로가 열림 |
| 필드 | 간결하지만 순수 Java 테스트에서 외부 의존성을 넣기 어려움 |
| 일반 메서드 | 여러 의존성을 함께 넣을 수 있으나 예제에서는 자주 쓰지 않음 |

Spring Bean의 생성자가 하나면 `@Autowired`를 생략할 수 있다. 필요한 생성자 인수를 누락하면 컴파일 시 확인할 수 있고, final 필드를 초기화하지 않아도 컴파일 오류가 난다. 생성자 주입은 container 없이 테스트할 때도 필요한 객체를 전달하기 쉽다.

Lombok의 `@RequiredArgsConstructor`는 final 필드를 받는 생성자를 만들어 반복 코드를 줄인다. 이 편의 기능과 주입 원리는 구분한다. 순수 Java에서 직접 만든 객체에는 `@Autowired`가 자동으로 작동하지 않는다.

## 선택적 의존성

| 표기 | 대상 Bean이 없을 때 |
|---|---|
| `@Autowired(required=false)` | 예제의 setter 메서드 자체를 호출하지 않음 |
| `@Nullable` | 매개변수에 null 전달 |
| `Optional<T>` | Optional.empty 전달 |

자료에서 `@Autowired`의 required 기본값은 true다. 선택적 규칙은 실제로 없어도 동작할 수 있는 의존성에 사용한다. Nullable과 Optional은 생성자 매개변수에도 사용할 수 있다.

## 같은 타입의 후보가 여러 개일 때

FixDiscountPolicy와 RateDiscountPolicy를 모두 Bean으로 등록하고 DiscountPolicy를 단일 의존성으로 주입하면 `NoUniqueBeanDefinitionException`이 발생할 수 있다. 구현 타입으로 좁히면 교체 유연성이 떨어지고 같은 구현 타입이 여럿인 상황도 해결하지 못한다.

PDF는 타입 후보 중 필드 이름·매개변수 이름과 Bean 이름을 추가로 맞추는 방식도 설명한다. 이는 이 자료의 매칭 설명이며 모든 버전의 세부 조건을 보장하는 전체 알고리즘은 아니다.

### Qualifier와 Primary

`@Qualifier`는 Bean 이름을 바꾸는 기능이 아니라 추가 구분자를 부여하는 기능이다. 등록한 Bean과 주입 지점에 같은 qualifier를 사용한다. PDF는 qualifier로 찾지 못하면 Bean 이름으로도 찾을 수 있지만 qualifier끼리 맞추는 의도를 명확히 하라고 설명한다.

`@Primary`는 여러 후보 중 기본으로 사용할 Bean을 정한다. 대부분 사용하는 구현은 Primary로, 특정 구현이 필요한 지점은 Qualifier로 선택할 수 있다. 명시적인 Qualifier 선택이 Primary보다 우선한다.

qualifier 문자열의 오타를 줄이기 위해 `@MainDiscountPolicy` 같은 사용자 정의 annotation에 Qualifier를 결합하는 예도 있다. 이는 Java annotation의 상속과 구분한다.

## 모든 구현이 필요할 때

`List<DiscountPolicy>`는 해당 타입의 Bean들을 모으고, `Map<String, DiscountPolicy>`는 Bean 이름을 key로 각 구현을 모은다. 기본편의 DiscountService는 전달받은 할인 코드로 Map에서 정책을 골라 실행한다.

하나의 기본 구현을 고르는 주입과, 여러 전략을 모두 받아 실행 시 선택하는 주입은 목적이 다르다. 자료는 해당 타입의 Bean이 없을 때 빈 컬렉션을 전달하는 경우도 설명한다.

## 자동·수동 등록의 운영 기준

정형화된 Controller·Service·Repository 같은 업무 로직에는 자동 등록을 기본으로 사용한다. 직접 작성한 DB 연결·공통 로그 등 광범위한 기술 지원 객체는 수동 구성으로 드러내면 파악하기 쉽다.

여러 할인 전략을 Map으로 주입하는 등 다형성을 적극 사용하는 업무 로직은 수동 구성에 모으거나 같은 패키지에 묶어 후보를 알아보기 쉽게 한다. Spring Boot 자체의 자동 설정은 이 기준과 구분하고 의도된 설정을 활용한다. 수동 등록만 OCP·DIP를 지킬 수 있는 것은 아니다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring component scan의 범위와 충돌](spring-component-scan.md)
- [Spring container와 Bean 조회](spring-container-bean-lookup.md)
- [할인 정책 확장과 단위 테스트](discount-policy-extension-and-testing.md)
