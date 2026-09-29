# Hibernate ORM 7.1 — GeneratedColumn Javadoc

> Source: https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/GeneratedColumn.html
> Collected: 2026-09-29
> Published: Unknown

Package org.hibernate.annotations
Annotation Interface GeneratedColumn
@Target({FIELD,METHOD})
@Retention(RUNTIME)
public @interface GeneratedColumn
Specifies that a column is defined using a DDL generated always as
clause or equivalent, and that Hibernate should fetch the generated value
from the database after each SQL INSERT or UPDATE.
Since:
6.0
See Also:
ColumnDefault
DialectOverride.GeneratedColumn
Required Element Summary
Required Elements
Modifier and Type
Required Element
Description
String
value
The expression to include in the generated DDL.
Element Details
value
String value
The expression to include in the generated DDL.
Returns:
the SQL expression that is evaluated to generate the column value.
