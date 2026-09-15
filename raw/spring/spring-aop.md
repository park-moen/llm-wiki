# AOP

> Source: [강의 참고 PDF](spring-aop.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다. 시간 측정 예제와 proxy diagram의 관계를 보존했다.

## 문제

모든 회원 기능의 실행 시간을 측정하려 하면 Controller, Service, Repository의 각 method에 시간 측정 코드를 넣게 된다. 시간 측정은 공통 관심 사항(cross-cutting concern), 회원 업무는 핵심 관심 사항(core concern)이다. 두 관심사가 섞이면 핵심 logic을 유지하기 어렵고 변경 지점도 늘어난다.

## AOP 적용

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
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint + " " + timeMs + "ms");
        }
    }
}
```

- `@Aspect`로 AOP class임을 표시한다.
- `@Around` pointcut expression으로 적용 대상을 선택한다.
- `ProceedingJoinPoint.proceed()`가 실제 대상 method를 호출한다.
- 공통 logic을 한곳에서 관리하고 핵심 business logic을 깨끗하게 유지할 수 있다.

## Proxy 동작

Spring AOP를 적용하면 Controller가 실제 `MemberService`가 아니라 proxy `MemberService`를 호출한다. Proxy가 시간 측정 logic을 실행한 뒤 `joinPoint.proceed()`로 실제 객체를 호출한다. 적용 대상마다 Spring container에는 proxy와 실제 객체의 관계가 형성된다.
