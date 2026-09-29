# MySQL 8.4 Reference Manual: EXPLAIN Statement

> Source: https://dev.mysql.com/doc/refman/8.4/en/explain.html
> Collected: 2026-09-29
> Published: Unknown

## Obtaining Execution Plan Information

The `EXPLAIN` statement provides information about how MySQL executes statements:

- `EXPLAIN` works with `SELECT`, `DELETE`, `INSERT`, `REPLACE`, `UPDATE`, and `TABLE` statements.
- When `EXPLAIN` is used with an explainable statement, MySQL displays information from the optimizer about the statement execution plan. That is, MySQL explains how it would process the statement, including information about how tables are joined and in which order.

## Obtaining Information with EXPLAIN ANALYZE

`EXPLAIN ANALYZE` runs a statement and produces `EXPLAIN` output along with timing and additional, iterator-based, information about how the optimizer's expectations matched the actual execution.

The query execution information is displayed using the `TREE` output format, in which nodes represent iterators. `EXPLAIN ANALYZE` always uses the `TREE` output format.
