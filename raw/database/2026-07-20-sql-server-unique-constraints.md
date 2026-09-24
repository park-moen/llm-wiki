# Unique constraints and check constraints

> Source: https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints?view=sql-server-ver17
> Collected: 2026-09-22
> Published: 2026-07-20

## UNIQUE constraints

Constraints are rules that the SQL Server Database Engine enforces for you. For example, you can use `UNIQUE` constraints to make sure that no duplicate values are entered in specific columns that don't participate in a primary key. Although both a `UNIQUE` constraint and a `PRIMARY KEY` constraint enforce uniqueness, use a `UNIQUE` constraint instead of a `PRIMARY KEY` constraint when you want to enforce the uniqueness of a column (or combination of columns) that isn't the primary key.

Unlike `PRIMARY KEY` constraints, `UNIQUE` constraints allow for the value `NULL`. However, as with any value participating in a `UNIQUE` constraint, only one null value is allowed per column. A `UNIQUE` constraint can be referenced by a `FOREIGN KEY` constraint.

When a `UNIQUE` constraint is added to an existing column or columns in the table, by default, the Database Engine examines the existing data in the columns to make sure all values are unique. If a `UNIQUE` constraint is added to a column that has duplicate values, the Database Engine returns an error and doesn't add the constraint.

The Database Engine automatically creates a `UNIQUE` index to enforce the uniqueness requirement of the `UNIQUE` constraint. Therefore, if an attempt to insert a duplicate row is made, the Database Engine returns an error message that states the `UNIQUE` constraint was violated, and doesn't add the row to the table. Unless a clustered index is explicitly specified, a unique, nonclustered index is created by default to enforce the `UNIQUE` constraint.
