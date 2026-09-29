# MySQL 8.4 Reference Manual: ORDER BY Optimization — Index Choice

> Source: https://dev.mysql.com/doc/refman/8.4/en/order-by-optimization.html
> Collected: 2026-09-29
> Published: Unknown

## Use of Indexes to Satisfy ORDER BY

In some cases, MySQL may use an index to satisfy an `ORDER BY` clause and avoid the extra sorting involved in performing a `filesort` operation.

The index may also be used even if the `ORDER BY` does not match the index exactly, as long as all unused portions of the index and all extra `ORDER BY` columns are constants in the `WHERE` clause. If the index does not contain all columns accessed by the query, the index is used only if index access is cheaper than other access methods.

Assuming that there is an index on `(key_part1, key_part2)`, the following queries may use the index to resolve the `ORDER BY` part. Whether the optimizer actually does so depends on whether reading the index is more efficient than a table scan if columns not in the index must also be read.

In this query, `key_part1` is constant, so all rows accessed through the index are in `key_part2` order, and an index on `(key_part1, key_part2)` avoids sorting if the `WHERE` clause is selective enough to make an index range scan cheaper than a table scan:

```sql
SELECT * FROM t1
  WHERE key_part1 = constant
  ORDER BY key_part2;
```

In some cases, MySQL cannot use indexes to resolve the `ORDER BY`, although it may still use indexes to find the rows that match the `WHERE` clause.
