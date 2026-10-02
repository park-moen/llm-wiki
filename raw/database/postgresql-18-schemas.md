# PostgreSQL 18: Schemas

> Source: https://www.postgresql.org/docs/18/ddl-schemas.html
> Collected: 2026-10-02
> Published: Unknown

To create or access objects in a schema, write a qualified name consisting of the schema name and table name separated by a dot:

`schema.table`

This works anywhere a table name is expected, including the table modification commands and the data access commands discussed in the following chapters.

The same object name can be used in different schemas without conflict; for example, both `schema1` and `myschema` can contain tables named `mytable`.
