# MariaDB: Generated Columns

> Source: https://mariadb.com/docs/server/reference/sql-statements/data-definition/create/generated-columns
> Collected: 2026-10-01
> Published: Unknown

## Syntax

```text
<type> [GENERATED ALWAYS] AS (<expression>)
[VIRTUAL | PERSISTENT | STORED] [UNIQUE [KEY]] [COMMENT <text>]
```

## Description

A generated column is a column in a table that cannot explicitly be set to a specific value in a DML query. Instead, its value is automatically generated based on an expression.

* `PERSISTENT` (a.k.a. `STORED`): This type's value is actually stored in the table.
* `VIRTUAL`: This type's value is not stored at all. Instead, the value is generated dynamically when the table is queried. This type is the default.

## Index Support

Using a generated column as a table's primary key is not supported.

Defining indexes on both `VIRTUAL` and `PERSISTENT` generated columns is supported.

If an index is defined on a generated column, then the optimizer considers using it in the same way as indexes based on "real" columns.

From MariaDB 11.8: The optimizer can recognize use of indexed virtual column expressions in the `WHERE` clause and use them to construct range and `ref(const)` accesses.

Before MariaDB 11.8: The optimizer **cannot** recognize use of indexed virtual column expressions in the `WHERE` clause and use them to construct range and `ref(const)` accesses.

The `ALTER TABLE` statement has limited support for generated columns. It does not support altering a table if `ALGORITHM` is not set to `COPY` if the table has a `VIRTUAL` generated column that is indexed.

## Expression Support

Using anything that depends on data outside the row is not supported in expressions for generated columns.

Non-deterministic built-in functions are not supported in expressions for `PERSISTENT` or indexed `VIRTUAL` generated columns.

You can also use virtual columns to implement a "poor man's partial index". See example at the end of Unique Index.
