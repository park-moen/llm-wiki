# Database Unique Constraint

> Sources: PostgreSQL Global Development Group, Unknown; Microsoft, 2026-07-20
> Raw: [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md); [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md)
> Updated: 2026-09-22

## Overview

`UNIQUE` constraint는 table의 한 column 또는 여러 column 조합에 같은 값이 중복 저장되지 않도록 database가 강제하는 data integrity 규칙이다. email처럼 primary key는 아니지만 중복되면 안 되는 business identifier에 사용한다. 중복 `INSERT`가 들어오면 database가 위반 오류를 반환하므로, application의 사전 조회와 별개로 저장소의 최종 규칙을 선언할 수 있다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md); [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md)

## 선언 방식과 복합 제약

한 column에는 column constraint로 선언할 수 있고, 여러 column을 묶을 때는 table constraint로 선언한다.

```sql
CREATE TABLE members (
    email text UNIQUE
);

CREATE TABLE memberships (
    organization_id bigint,
    user_id bigint,
    UNIQUE (organization_id, user_id)
);
```

복합 `UNIQUE`는 각 column이 각각 고유해야 한다는 뜻이 아니다. 지정한 모든 column 값의 **조합**이 table 전체에서 고유해야 한다는 뜻이다. 따라서 한 사용자는 여러 organization에 속할 수 있지만, 같은 organization에 같은 사용자를 두 번 연결하지 않도록 표현할 수 있다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md)

## PRIMARY KEY와 구분

둘 다 중복을 막지만 용도가 다르다.

| 제약 | 의미 | `NULL` | table당 개수 |
| --- | --- | --- | --- |
| `PRIMARY KEY` | row를 식별하는 대표 key | 허용하지 않음 | 하나 |
| `UNIQUE` | primary key가 아닌 column 또는 조합의 중복 방지 | DBMS별 동작을 확인해야 함 | 여러 개 가능 |

PostgreSQL은 `PRIMARY KEY`를 `UNIQUE`와 `NOT NULL`을 함께 적용한 것과 같은 data 규칙으로 설명한다. SQL Server도 primary key가 아닌 column 또는 조합의 고유성을 보장할 때 `UNIQUE`를 사용하라고 안내한다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md); [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md)

## `NULL`은 DBMS마다 다르다

`UNIQUE`가 있다고 해서 nullable column의 `NULL`까지 항상 한 번만 허용되는 것은 아니다. PostgreSQL의 기본값은 `NULL` 둘을 같지 않게 보므로 constrained column 중 하나라도 `NULL`이면 같은 값의 row를 여러 개 저장할 수 있다. 반면 SQL Server는 `UNIQUE` column마다 `NULL`을 하나만 허용한다고 설명한다. SQL standard도 기본 `NULL` 처리를 implementation-defined로 두므로, portable schema라면 대상 DBMS에서 반드시 확인해야 한다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md); [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md)

PostgreSQL에서 `NULL`도 같은 값으로 취급해야 하면 `UNIQUE NULLS NOT DISTINCT`를 사용한다. `NULL`을 허용하지 않아야 한다면 의도를 더 직접적으로 나타내는 `NOT NULL`을 함께 선언한다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md)

## Index와 도입 시 확인할 점

PostgreSQL과 SQL Server는 `UNIQUE` constraint를 강제하기 위해 unique index를 자동으로 만듭니다. 그러므로 constraint를 단지 application validation의 보조 수단으로 보지 말고, data rule을 선언한 schema 요소로 다룹니다. [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md); [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md)

기존 table에 constraint를 추가할 때는 먼저 중복 data가 있는지 확인합니다. SQL Server는 이미 중복된 값이 있으면 constraint 추가를 실패시킨다고 명시합니다. PostgreSQL에서 일부 row에만 고유성을 적용해야 한다면 일반 `UNIQUE` constraint 대신 partial unique index를 사용합니다. [SQL Server Unique Constraints](../../raw/database/2026-07-20-sql-server-unique-constraints.md); [PostgreSQL Unique Constraints](../../raw/database/postgresql-unique-constraints.md)

## See Also

- [Spring DB 접근 기술 비교](../spring/spring-database-access-technologies.md)
