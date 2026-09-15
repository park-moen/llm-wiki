# Spring TestContext transaction 관리 공식 문서 companion

> Source: https://docs.spring.io/spring-framework/reference/testing/testcontext-framework/tx.html
> Collected: 2026-08-22
> Published: Unknown

Spring Framework 공식 문서의 test-managed transaction 관련 근거를 보존한 companion이다.

- Spring TestContext Framework는 `TransactionalTestExecutionListener`를 통해 test transaction을 관리한다.
- Test class 또는 test method에 Spring의 `@Transactional`을 선언하면 해당 test method가 transaction 안에서 실행된다.
- Test-managed transaction은 test가 끝날 때 기본적으로 자동 rollback된다.
- `@Commit`과 `@Rollback`으로 test 완료 후의 commit·rollback 동작을 선언적으로 변경할 수 있다.
- Transaction 지원에는 test ApplicationContext에 `PlatformTransactionManager` Bean이 필요하다.
- JUnit Jupiter의 `@BeforeEach`와 `@AfterEach`는 transactional test의 test-managed transaction 안에서 실행된다.
- Preemptive timeout이 test method를 별도 thread에서 실행하면 그 작업은 thread-bound test transaction 밖에서 실행되어 rollback되지 않을 수 있다.

