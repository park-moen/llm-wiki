# Spring component scan의 범위와 충돌

> Sources: 김영한, Unknown (스프링 핵심 원리 - 기본편)
> Raw: [6. 컴포넌트 스캔 PDF companion](../../raw/spring/spring-core-component-scan.md)
> Updated: 2026-10-11

## Overview

Component scan은 대상 클래스를 찾아 Bean으로 등록하고, 자동 주입은 등록된 Bean 사이의 의존관계를 연결한다. 기본편은 Java 구성에 Bean을 일일이 나열하던 방식에서 scan으로 전환하면서 탐색 범위, 이름, 필터와 충돌을 설명한다. 등록과 연결의 역할을 구분하면 Bean이 없거나 예상치 않은 설정이 함께 등록되는 문제를 찾기 쉽다.

## 등록과 연결의 분리

`@ComponentScan`이 `@Component` 클래스를 등록한다. 기본 이름은 클래스명의 첫 글자를 소문자로 만든 이름이며 `@Component("memberService2")`처럼 지정할 수도 있다. 생성자의 `@Autowired`는 매개변수 타입에 맞는 Bean을 찾아 연결한다.

기본편의 AutoAppConfig는 과거 AppConfig 예제와 테스트를 유지하기 위해 `@Configuration`을 scan 대상에서 제외한다. 이는 예제끼리 설정이 섞이지 않게 하는 선택이며 모든 프로젝트에서 구성 클래스를 제외하라는 규칙은 아니다.

## 탐색 위치와 기본 대상

| 설정 | 의미 |
|---|---|
| `basePackages` | 지정 패키지와 하위 패키지를 탐색하며 여러 시작점을 지정할 수 있음 |
| `basePackageClasses` | 지정한 클래스의 패키지를 탐색 시작점으로 사용 |
| 탐색 위치 생략 | ComponentScan이 붙은 구성 클래스의 패키지에서 시작 |

PDF는 구성 클래스 또는 `@SpringBootApplication` 클래스를 프로젝트의 최상위 패키지에 두는 방식을 권장한다.

`@Controller`, `@Service`, `@Repository`, `@Configuration`은 `@Component`를 포함해 기본 대상이 된다. 이런 결합을 해석하는 것은 Java의 annotation 상속이 아니라 Spring의 기능이다. 대상 등록 외에 Controller 인식, Repository의 데이터 접근 예외 변환, Configuration의 구성 처리 역할도 구분한다. Service는 비즈니스 계층이라는 의도를 드러낸다.

## 필터

`includeFilters`는 추가 대상을, `excludeFilters`는 제외 대상을 지정한다. `useDefaultFilters`를 끄면 기본 탐색 대상이 제외된다.

| FilterType | 판단 기준 |
|---|---|
| `ANNOTATION` | annotation |
| `ASSIGNABLE_TYPE` | 지정 타입과 자식 타입 |
| `ASPECTJ` | AspectJ 패턴 |
| `REGEX` | 정규 표현식 |
| `CUSTOM` | TypeFilter 구현 |

자료는 특별한 이유가 없다면 기본 설정을 우선하고 필터를 복잡하게 바꾸지 않는 방향을 설명한다.

## 같은 Bean 이름이 충돌할 때

자동 등록끼리 이름이 같으면 `ConflictingBeanDefinitionException`이 발생할 수 있다. PDF의 Spring 예제에서는 수동 등록이 자동 등록을 덮어쓰는 경우도 보여준다.

다만 같은 자료는 Spring Boot 실행에서는 등록 충돌을 오류로 처리하는 기본 설정을 구분한다. 따라서 수동 Bean이 항상 조용히 우선한다는 규칙으로 읽으면 안 된다. 자료의 오류 안내에는 `spring.main.allow-bean-definition-overriding=true`가 나오지만, 설명의 초점은 의도하지 않은 중복 설정을 발견하는 데 있다.

## See Also

- [스프링 핵심 원리 기본편 학습 안내](spring-core-learning-roadmap.md)
- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [자동 주입과 Bean 선택](spring-autowiring-and-bean-selection.md)
- [Spring container와 Bean 조회](spring-container-bean-lookup.md)
