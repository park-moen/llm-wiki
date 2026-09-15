# Spring JdbcTemplate 강의 원문

> Source: Inflearn 김영한, 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
> Collected: 2026-08-22
> Published: Unknown
> Transcript note: 사용자 제공 자동 전사에서 문장부호와 반복 발화만 정리한 보존본

네 이제 이번 시간에는 Spring JDBC Template에 대해서 알아보겠습니다.

설정은 기존의 Spring Boot Starter JDBC 설정과 똑같이 하면 됩니다. Spring JDBC Template은 MyBatis와 비슷한 성격의 라이브러리인데, JDBC API의 반복 코드를 제거합니다. 이전 순수 JDBC 코드에는 ResultSet과 Connection 처리처럼 반복되는 부분이 많았습니다. JdbcTemplate은 이런 반복 코드를 많이 제거하지만 SQL은 개발자가 직접 작성해야 합니다. 반복 코드만 제거해도 훨씬 편리하고, JdbcTemplate은 실무에서도 많이 사용합니다.

`JdbcTemplateMemberRepository`를 만들고 `MemberRepository`를 구현합니다. 먼저 `JdbcTemplate`이 필요합니다. 강의에서는 `JdbcTemplate` 자체를 주입받는 대신 이전처럼 `DataSource`를 생성자로 주입받고 `new JdbcTemplate(dataSource)`로 생성합니다. Spring에서도 이런 방식을 권장합니다. 생성자가 하나뿐이면 `@Autowired`를 생략해도 Spring이 `DataSource`를 자동으로 주입합니다. 생성자가 둘 이상이면 그대로 생략할 수 없습니다.

먼저 조회 query를 구현합니다. `jdbcTemplate.query("select * from member where id = ?", ...)`를 사용하고, 조회 결과는 `RowMapper`로 mapping합니다. `RowMapper`를 구현해 전달하면 ResultSet이 들어오고, `rs.getLong("id")`, `rs.getString("name")`으로 값을 읽어 Member를 만들어 반환합니다. 익명 구현은 lambda로 바꿀 수 있습니다. IntelliJ에서 `Option + Enter`를 눌러 `Replace with lambda`를 선택하면 Java 8 lambda 형태로 줄일 수 있습니다.

`query` 결과는 List로 반환되므로 강의의 단건 조회는 결과를 stream으로 바꾸고 `findAny()`를 사용해 Optional로 반환합니다. 순수 JDBC의 `findById`는 매우 길지만 JdbcTemplate에서는 SQL 실행과 RowMapper 호출을 포함해 짧은 코드로 작성할 수 있습니다. JDBC 코드를 template method pattern과 callback 등을 사용해 줄여 놓은 라이브러리라고 이해할 수 있습니다.

저장은 `SimpleJdbcInsert`를 사용합니다. JdbcTemplate으로 만들고 `withTableName`으로 table을, `usingGeneratedKeyColumns("id")`로 DB가 생성할 key column을 알려 줍니다. Table name, generated key와 입력할 name을 알면 INSERT를 만들 수 있으므로 직접 INSERT SQL을 작성하지 않아도 됩니다. `executeAndReturnKey`로 생성된 key를 받고 Member의 id에 설정합니다. 자세한 사용법은 JdbcTemplate documentation을 참고하면 됩니다.

`findByName`은 WHERE 조건을 name으로 바꾸고 RowMapper로 mapping합니다. `findAll`은 `select * from member`와 RowMapper를 전달하면 List를 바로 반환하므로 더 간단합니다. ResultSet의 각 row를 callback인 RowMapper가 Member object로 변환합니다.

Repository 구현이 끝나면 configuration의 조립부에서 `new JdbcTemplateMemberRepository(dataSource)`를 반환하도록 교체합니다. 앞에서 Spring integration test를 만들어 두었기 때문에 web application을 띄워 회원 가입 화면을 수동으로 조작하지 않고도 DB 연동을 검증할 수 있습니다. 단, 실습용 DB server는 실행 중이어야 합니다.

처음 test를 실행했을 때 parameter 하나가 설정되지 않았다는 오류가 발생했습니다. 조회 query의 마지막에 id parameter 전달이 빠져 있었기 때문입니다. 빠진 id를 추가하고 IntelliJ에서 `Control + R`을 누르면 마지막으로 실행했던 integration test를 다시 실행할 수 있습니다. 녹색 표시가 뜨면 JdbcTemplate version에서도 DB까지 연동한 test가 성공한 것입니다.

이 사례처럼 잘 작성했다고 생각한 코드도 test를 실행하면 단순한 누락이나 복사·붙여넣기 오류를 발견할 수 있습니다. Application 규모가 커질수록 test code를 꼼꼼히 작성하는 일이 중요합니다.

다음에는 직접 작성하던 SQL까지 줄일 수 있는 JPA를 알아봅니다.
