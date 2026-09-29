# Microsoft Learn — Primary and foreign key constraints

> Source: https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints?view=sql-server-ver17
> Collected: 2026-09-29
> Published: 2026-07-20
> Scope: Selected passages from Primary key constraints, Foreign key constraints, and Indexes on foreign key constraints.

## Primary key constraints

A table typically has a column or combination of columns that contain values that uniquely identify each row in the table. This column, or columns, is called the primary key (PK) of the table and enforces the entity integrity of the table.

When you specify a primary key constraint for a table, the Database Engine enforces data uniqueness by automatically creating a unique index for the primary key columns. This index also permits fast access to data when the primary key is used in queries. If a primary key constraint is defined on more than one column, values can be duplicated within one column, but each combination of values from all the columns in the primary key constraint definition must be unique.

As shown in the following illustration, the `ProductID` and `VendorID` columns in the `Purchasing.ProductVendor` table form a composite primary key constraint for this table. This makes sure that every row in the `ProductVendor` table has a unique combination of `ProductID` and `VendorID`. This prevents the insertion of duplicate rows.

* A table can contain only one primary key constraint.
* All columns defined within a primary key constraint must be defined as not null. If nullability isn't specified, all columns participating in a primary key constraint have their nullability set to not null.

## Foreign key constraints

A foreign key (FK) is a column or combination of columns that is used to establish and enforce a link between the data in two tables to control the data that can be stored in the foreign key table. In a foreign key reference, a link is created between two tables when the column or columns that hold the primary key value for one table are referenced by the column or columns in another table. This column becomes a foreign key in the second table.

By creating this foreign key relationship, a value for `SalesPersonID` can't be inserted into the `SalesOrderHeader` table if it doesn't already exist in the `SalesPerson` table.

## Indexes on foreign key constraints

Unlike primary key constraints, creating a foreign key constraint doesn't automatically create a corresponding index. However, manually creating an index on a foreign key is often useful for the following reasons:

* Foreign key columns are frequently used in join criteria when the data from related tables is combined in queries by matching the column or columns in the foreign key constraint of one table with the primary or unique key column or columns in the other table. An index enables the Database Engine to quickly find related data in the foreign key table. However, creating this index isn't required.
