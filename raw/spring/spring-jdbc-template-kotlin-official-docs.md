# Kotlin JdbcTemplate 공식 문서 companion

> Source: https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html; https://docs.spring.io/spring-framework/reference/data-access/jdbc/simple.html
> Collected: 2026-08-22
> Published: Unknown

Spring Framework 공식 JDBC 문서는 Kotlin에서도 `JdbcTemplate(dataSource)`로 template을 만들고, lambda 형태의 `RowMapper`로 `ResultSet`의 각 row를 object에 mapping하는 예를 제공한다. `query`는 여러 row를 List로 반환하며 mapper는 재사용 가능한 property로 분리할 수 있다.

`SimpleJdbcInsert`는 JDBC driver가 제공하는 database metadata를 사용해 필요한 설정을 줄인다. `withTableName`으로 table을, `usingGeneratedKeyColumns`로 자동 생성 key column을 지정하고 `executeAndReturnKey`로 생성된 `Number`를 받을 수 있다. 전달하는 parameter map의 key는 database column 이름과 일치해야 한다.

Kotlin의 immutable data class를 사용한다면 생성된 key를 기존 object에 변경하는 대신 `copy(id = generatedKey.toLong())`로 새 object를 반환할 수 있다.

