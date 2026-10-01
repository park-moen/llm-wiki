# MariaDB: Getting Started with Indexes Guide — Unique Index

> Source: https://mariadb.com/docs/server/mariadb-quickstart-guides/mariadb-indexes-guide
> Collected: 2026-10-01
> Published: Unknown

## Unique Index

A unique index ensures that all values in the indexed column (or combination of columns) are unique. However, unlike a primary key, columns in a unique index can store `NULL` values.

**Multi-Column Unique Indexes:** An index can span multiple columns. MariaDB can use the leftmost part(s) of such an index if it cannot use the whole index (except for HASH indexes).

**`NULL` Values in Unique Indexes:** A `UNIQUE` constraint allows multiple `NULL` values because in SQL, `NULL` is never equal to another `NULL`.

```sql
CREATE TABLE t1 (a INT NOT NULL, b INT, UNIQUE (a,b));
INSERT INTO t1 VALUES (3,NULL), (3, NULL); -- Both rows are inserted
```

**Conditional Uniqueness with Virtual Columns:** You can enforce uniqueness over a subset of rows using unique indexes on virtual columns. This example ensures `user_name` is unique for 'Active' or 'On-Hold' users, but allows duplicate names for 'Deleted' users:

```sql
CREATE TABLE Table_1 (
  user_name VARCHAR(10),
  status ENUM('Active', 'On-Hold', 'Deleted'),
  del CHAR(0) AS (IF(status IN ('Active', 'On-Hold'), '', NULL)) PERSISTENT,
  UNIQUE(user_name, del)
);
```

### Choosing Indexes

**Avoid Over-Indexing:** Extra indexes consume storage and can slow down `INSERT`, `UPDATE`, and `DELETE` operations.
