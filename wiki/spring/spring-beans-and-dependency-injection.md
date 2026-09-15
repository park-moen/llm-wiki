# Spring Bean과 의존관계 설정

> Sources: 김영한, 2026-01-30
> Raw: [스프링 빈과 의존관계 PDF companion](../../raw/spring/spring-beans-and-dependency-injection.md), [컴포넌트 스캔과 자동 의존관계 설정 강의 원문](../../raw/spring/spring-component-scan-auto-dependency-injection-lecture.md), [Java 코드로 직접 Spring Bean 등록 강의 원문](../../raw/spring/spring-java-configuration-dependency-injection-lecture.md)
> Updated: 2026-08-13

## Overview

Spring container는 application 객체를 Spring Bean으로 등록하고 필요한 의존관계를 연결한다. Component scan은 stereotype annotation을 찾아 자동 등록하고, Java configuration은 `@Bean` method로 객체 생성과 조립을 명시한다. 의존성이 실행 중 바뀔 이유가 거의 없는 일반적인 application code에서는 constructor injection이 가장 적합하다.

## Spring Bean으로 등록해야 하는 이유

`MemberController`가 `MemberService`를 사용하려면 두 객체가 Spring container의 관리 대상이어야 한다. 등록되지 않은 type을 주입하려 하면 다음과 같은 오류가 발생한다.

```text
No qualifying bean of type 'hello.hellospring.service.MemberService' available
```

즉, class가 존재하는 것과 Spring이 그 객체를 생성·관리하는 것은 다르다. `@Autowired`도 Spring이 관리하는 객체 사이에서만 동작하며 직접 `new`로 만든 객체에는 적용되지 않는다.

## Component scan과 stereotype

`@Component`가 붙은 class는 component scan 대상이 된다. 다음 stereotype annotation도 내부적으로 `@Component`를 포함한다.

- `@Controller`: Web 요청 처리 객체
- `@Service`: Business service 객체
- `@Repository`: Data access 객체

생성자에 `@Autowired`를 붙이면 Spring이 parameter type에 맞는 Bean을 찾아 연결한다. 생성자가 하나뿐이면 annotation을 생략할 수 있다. 별도 scope를 지정하지 않은 기본 Bean은 singleton이므로 container 안에서 같은 instance를 공유한다.

### 기본 scan 범위

Spring Boot의 기본 component scan은 `@SpringBootApplication`이 있는 application class의 package부터 시작해 그 package와 하위 package를 탐색한다. 이 범위 밖의 class에 `@Component`나 stereotype annotation을 붙여도 기본 설정만으로는 Bean으로 등록되지 않는다. 따라서 application class는 보통 project의 최상위 package에 둔다.

### 자동 등록과 의존관계 연결 흐름

회원 관리 예제의 의존관계는 다음 순서로 조립된다.

1. Component scan이 `@Controller`, `@Service`, `@Repository`가 붙은 class를 찾아 각각 Bean으로 등록한다.
2. Spring이 `MemberController`를 생성하면서 생성자에 필요한 `MemberService` Bean을 주입한다.
3. Spring이 `MemberService`를 생성하면서 생성자 parameter인 `MemberRepository` type에 맞는 `MemoryMemberRepository` Bean을 주입한다.

`@Component` 계열 annotation은 객체를 container에 **등록**하고, `@Autowired`는 등록된 객체 사이의 의존관계를 **연결**한다. 여러 Controller나 Service가 같은 singleton Bean을 요청하면 기본적으로 동일한 instance가 주입된다.

## Java configuration

`@Configuration` class 안에서 `@Bean` method로 객체를 직접 등록할 수도 있다. `MemberService(memberRepository())`처럼 생성자 호출 관계를 작성하면 객체 생성과 조립이 configuration에 드러난다.

```java
@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

이 구성에서는 `MemberService`와 `MemoryMemberRepository`의 stereotype annotation을 제거하고 `SpringConfig`에서 직접 등록한다. 반면 Web 요청을 받는 `MemberController`는 `@Controller`로 자동 등록하며, 생성자 `@Autowired`를 통해 configuration에 등록된 `MemberService`를 주입받는다. 자동 등록과 직접 등록은 한 application 안에서 함께 사용할 수 있다.

강의에서는 정형화된 Controller·Service·Repository에는 component scan을 쓰되, 교체 가능성이 있는 Repository 구현은 Java configuration으로 선택하는 방식을 보여준다. Memory, JDBC, JPA 구현을 바꿀 때 Service code 대신 조립부만 변경할 수 있다.

예를 들어 저장 방식을 바꿀 때 `memberRepository()`가 반환하는 구현체만 교체하면 `MemberService`나 Controller의 business code는 수정하지 않아도 된다. 과거에는 XML configuration도 사용했지만 강의는 Java code 설정을 중심으로 다룬다.

## DI 방식 선택

- Field injection: 짧지만 의존성이 감춰지고 test에서 교체하기 어렵다.
- Setter injection: 선택적 변경에는 쓸 수 있지만 public setter로 실행 중 의존관계를 바꿀 수 있다.
- Constructor injection: 필수 의존성을 생성 시점에 확정하고 불변 관계로 표현한다.

Application을 조립한 뒤 의존관계가 바뀌는 경우는 드물기 때문에 강의는 constructor injection을 권장한다.

## IntelliJ IDEA 단축키

| 환경 | 단축키 | 기능 | 강의에서의 사용 맥락 |
|---|---|---|---|
| macOS | `Command + P` | 호출할 method 또는 constructor의 parameter 정보 표시 | `new MemberService(...)`에 필요한 `MemberRepository` parameter 확인 |

이번 강의 원문에는 Windows/Linux 대응 단축키가 제시되지 않았다.

## See Also

- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [Spring DB 접근 기술 비교](spring-database-access-technologies.md)
- [Spring AOP와 공통 관심사 분리](spring-aop-cross-cutting-concerns.md)
