# 스프링 빈과 의존관계

> Source: [강의 참고 PDF](spring-beans-and-dependency-injection.pdf)
> Collected: 2026-08-12
> Published: 2026-01-30

이 문서는 같은 디렉터리의 원본 PDF를 검색하고 근거 검사에 사용할 수 있도록 옮긴 companion text다.

## Component scan과 자동 의존관계 설정

- `MemberController`는 `MemberService`를 사용한다.
- Spring container에 등록되지 않은 `MemberService`를 주입하려 하면 `No qualifying bean of type 'hello.hellospring.service.MemberService' available` 오류가 발생한다.
- `@Controller`, `@Service`, `@Repository`는 `@Component`를 포함하므로 component scan 대상이 된다.
- 생성자에 `@Autowired`를 사용하면 Spring container가 matching bean을 찾아 연결한다. 생성자가 하나뿐이면 `@Autowired`를 생략할 수 있다.
- 기본 bean scope는 singleton이므로 같은 Spring bean instance를 공유한다.

## Java configuration

`@Configuration` class에서 `@Bean` method로 `MemberService`와 `MemberRepository`를 직접 등록할 수 있다. 강의 예제는 `MemberService(memberRepository())`로 의존관계를 구성한다.

- 정형화된 Controller, Service, Repository에는 component scan을 주로 사용한다.
- 구현 class를 바꿀 가능성이 있는 Repository 같은 영역은 Java configuration으로 조립하면 교체가 명확하다.
- XML 방식도 있으나 최근에는 잘 사용하지 않는다고 설명한다.

## DI 방식

- Field injection
- Setter injection: public setter를 통해 실행 중 의존관계가 바뀔 수 있다는 단점이 있다.
- Constructor injection: application 조립 시 의존관계가 설정되고 이후 바뀌지 않는 일반적인 경우에 권장된다.

`@Autowired`는 Spring이 관리하는 객체 사이에서만 동작하며, 직접 `new`로 만든 객체에는 적용되지 않는다.
