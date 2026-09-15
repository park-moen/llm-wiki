# Spring AOP와 공통 관심사 분리

> Sources: 김영한, 2026-01-30
> Raw: [AOP PDF companion](../../raw/spring/spring-aop.md); [AOP가 필요한 상황 강의 원문](../../raw/spring/spring-aop-need-lecture.md); [Spring AOP 적용 강의 원문](../../raw/spring/spring-aop-application-lecture.md)
> Updated: 2026-08-22

## Overview

AOP(Aspect-Oriented Programming)는 시간 측정처럼 여러 계층에 반복되는 공통 관심사(cross-cutting concern)를 회원 업무 같은 핵심 관심사(core concern)에서 분리한다. Spring AOP는 대상 객체 앞에 proxy를 두고, pointcut으로 선택한 method 호출 전후에 공통 logic을 실행한다.

## 공통 관심사를 직접 넣을 때의 문제

회원 가입과 조회 시간을 측정하려고 각 Controller, Service, Repository method에 시작·종료 시각 계산을 넣으면 business logic과 측정 logic이 섞인다. 같은 code가 여러 곳에 반복되고 측정 방식을 바꿀 때 모든 method를 수정해야 하며, 적용 대상을 일괄 변경하기도 어렵다.

강의는 먼저 AOP 용어를 외우기보다 구체적인 실패 사례에서 필요성을 이해하도록 설명한다. `MemberService.join()`의 시간을 직접 재면 다음과 같은 code가 business logic을 감싸게 된다.

```java
long start = System.currentTimeMillis();
try {
    validateDuplicateMember(member);
    memberRepository.save(member);
    return member.getId();
} finally {
    long finish = System.currentTimeMillis();
    long timeMs = finish - start;
    System.out.println("join timeMs = " + timeMs);
}
```

`finally`에 종료 측정을 두는 이유는 business logic이 정상 반환할 때뿐 아니라 exception이 발생할 때도 종료 시각을 남기기 위해서다. 문제는 같은 구조를 `findMembers()`와 다른 모든 method에도 넣어야 한다는 점이다.

| 구분 | 예시 | 변경 이유 |
|---|---|---|
| 핵심 관심 사항(core concern) | 회원 가입, 중복 회원 검사, 회원 조회 | Business 요구사항이 바뀔 때 변경 |
| 공통 관심 사항(cross-cutting concern) | 실행 시간 측정 | 측정 방식이나 적용 범위가 바뀔 때 변경 |

시간 측정은 실제 method 실행 전과 후를 함께 감싸야 한다. 단순 계산 부분을 helper method로 추출하더라도 각 대상 method에 시작·종료 호출과 `try-finally` 구조가 남으므로 관심사가 제대로 분리되지 않는다. 측정 단위나 출력 형식을 변경할 때도 흩어진 code를 찾아야 한다.

### 측정값을 읽을 때의 warm-up 주의

강의 예제처럼 application을 띄운 직후 첫 호출은 이후 호출보다 오래 걸릴 수 있다. 첫 실행에는 class와 metadata loading 같은 초기화 작업이 포함될 수 있기 때문이다. 따라서 첫 번째 단일 측정값만으로 정상 운영 성능을 판단하지 않는다. 강의는 높은 성능이 필요한 server에서 시작 후 기능을 미리 호출하는 warm-up을 수행하기도 한다고 설명한다.

이 예제의 목적은 정밀한 benchmark 방법을 가르치는 것이 아니라, 동일한 측정 concern을 여러 business method에 직접 넣었을 때 생기는 구조적 문제를 보여주는 데 있다.

앞선 AOP 필요성 강의 원문에는 IntelliJ IDEA keyboard shortcut이 등장하지 않는다.

## Aspect와 Advice

강의의 `TimeTraceAop`는 `@Aspect`로 aspect를 선언하고 `@Around`로 적용 범위를 지정한다.

```java
@Component
@Aspect
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return joinPoint.proceed();
        } finally {
            long timeMs = System.currentTimeMillis() - start;
            System.out.println("END: " + joinPoint + " " + timeMs + "ms");
        }
    }
}
```

`@Aspect`는 이 class가 AOP 역할을 한다는 뜻이고 `@Around`는 선택한 method 호출의 전후를 모두 감싸는 advice다. `joinPoint.proceed()`가 실제 대상 method를 실행한다. 이를 호출하지 않으면 다음 대상 method로 진행하지 않을 수도 있다. `ProceedingJoinPoint`에서는 호출 method와 target, argument 같은 실행 context도 확인할 수 있다.

`finally`에서 시간을 기록하면 정상 반환과 exception 발생 모두에서 측정 종료 logic이 실행된다. 공통 logic은 한 class에서 변경하고, pointcut expression으로 적용 대상을 조정할 수 있다.

### Aspect를 Spring Bean으로 등록하기

Spring AOP가 aspect를 사용하려면 `TimeTraceAop`도 Spring Bean이어야 한다. 강의는 두 가지 방법을 설명한다.

- `@Component`를 붙여 component scan으로 자동 등록
- Configuration에서 `@Bean` method로 명시적으로 등록

정형화된 Service나 Repository에는 component scan이 편리하다. 반면 AOP처럼 application 전체에 특별한 영향을 주는 infrastructure는 configuration에 명시하면 application에 특정 aspect가 적용된다는 사실을 쉽게 발견할 수 있다는 장점이 있다. 강의 실습에서는 `@Component` 방식도 사용할 수 있음을 보여준다.

### Pointcut으로 적용 범위 선택하기

```java
@Around("execution(* hello.hellospring..*(..))")
```

이 `execution` expression은 `hello.hellospring` package와 그 하위 method를 적용 대상으로 선택한다. Package 범위를 service로 좁히면 Controller와 Repository는 제외하고 Service 계층만 측정할 수 있다. Class, method와 parameter 조건도 조합할 수 있지만 입문 단계에서는 자주 쓰는 package 범위를 이해하는 것이 핵심이다.

적용 범위를 넓게 잡으면 Controller → Service → Repository 호출마다 aspect가 실행된다. 시작 log는 호출이 안쪽으로 들어가는 순서로, 종료 log는 다시 바깥으로 돌아오는 역순으로 나타난다.

```text
START Controller
  START Service
    START Repository
    END Repository
  END Service
END Controller
```

이 구조로 어느 계층에서 시간이 오래 걸리는지 관찰할 수 있다. 원래 Service에 직접 넣었던 측정 code는 제거하고 핵심 business logic만 남긴다.

## Spring Proxy 동작

Spring container는 AOP 적용 대상 앞에 proxy 객체를 둔다. 예를 들어 Controller가 주입받아 호출하는 것은 실제 `MemberService`가 아니라 proxy이고, proxy가 advice를 실행한 뒤 실제 Service로 위임한다. 따라서 호출 흐름은 다음과 같다.

1. Controller가 Service proxy를 호출한다.
2. Proxy가 시간 측정을 시작한다.
3. `joinPoint.proceed()`가 실제 Service method를 호출한다.
4. 실제 method 완료 후 proxy가 시간을 기록한다.

Spring이 관리하지 않는 객체에 이 proxy 기반 AOP가 자동 적용되는 것은 아니다. Bean 등록과 의존관계 설정이 먼저 필요하다.

강의 환경에서는 Controller에 주입된 Service의 `getClass()`를 출력해 CGLIB 기반으로 생성된 proxy type을 확인한다. 생성 class 이름은 framework version과 proxy 방식에 따라 달라질 수 있으므로 특정 문자열 자체보다 실제 target 대신 Spring이 만든 proxy가 주입된다는 사실이 중요하다.

DI를 사용하면 Controller는 concrete object를 직접 `new`하지 않고 container가 제공하는 dependency를 받는다. Spring은 이 지점에 실제 Service 대신 proxy를 넣을 수 있고, proxy는 advice를 실행한 뒤 `joinPoint.proceed()`로 실제 Service를 호출한다. 직접 생성한 객체나 Spring Bean이 아닌 객체에는 이 container 기반 proxy 흐름을 기대할 수 없다.

강의는 AOP 구현 방식이 proxy만 있는 것은 아니며 compile 시점에 code를 결합하는 방식도 있다고 언급한다. 이 입문 과정의 초점은 Spring container가 Bean 앞에 proxy를 두는 Spring AOP 방식이다.

이번 AOP 적용 강의 원문에도 IntelliJ IDEA keyboard shortcut이 등장하지 않는다.

## See Also

- [Spring Bean과 의존관계 설정](spring-beans-and-dependency-injection.md)
- [Spring 회원 관리 백엔드와 테스트](spring-member-backend-and-testing.md)
- [스프링 입문 학습 로드맵](spring-learning-roadmap.md)
