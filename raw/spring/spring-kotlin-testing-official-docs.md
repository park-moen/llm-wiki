# Kotlin과 Spring testing 공식 문서 companion

> Source: https://docs.spring.io/spring/reference/languages/kotlin/spring-projects-in.html
> Collected: 2026-08-22
> Published: Unknown

Spring Framework의 Kotlin testing 지침에서 이 Wiki에 필요한 내용을 보존한 companion이다.

- Kotlin과 Spring Framework 조합의 test framework로 JUnit을 안내한다.
- Kotlin test function 이름은 backtick으로 감싸 사람이 읽기 쉬운 문장으로 작성할 수 있다.
- JUnit Jupiter는 Spring Bean constructor injection을 지원한다.
- `@TestConstructor(autowireMode = AutowireMode.ALL)`을 사용하면 모든 test constructor parameter를 autowiring할 수 있다.
- Constructor injection을 사용하면 Kotlin test dependency를 `lateinit var` 대신 `val`로 선언할 수 있다.
- `junit-platform.properties`의 `spring.test.constructor.autowire.mode = all`로 같은 동작을 project 기본값으로 설정할 수도 있다.
