# MySQL 8.4 Reference Manual: ORDER BY Optimization

> Source: https://dev.mysql.com/doc/refman/8.4/en/order-by-optimization.html
> Collected: 2026-09-29
> Published: Unknown

## Use of Indexes to Satisfy ORDER BY

In some cases, MySQL may use an index to satisfy an `ORDER BY` clause and avoid the extra sorting involved in performing a `filesort` operation.

In the next query, the `ORDER BY` does not name `key_part1`, but all rows selected have a constant `key_part1` value, so the index can still be used:

```sql
SELECT * FROM t1
  WHERE key_part1 = constant1 AND key_part2 > constant2
  ORDER BY key_part2;
```

In some cases, MySQL cannot use indexes to resolve the `ORDER BY`, although it may still use indexes to find the rows that match the `WHERE` clause.

## Use of filesort to Satisfy ORDER BY

If an index cannot be used to satisfy an `ORDER BY` clause, MySQL performs a `filesort` operation that reads table rows and sorts them. A `filesort` constitutes an extra sorting phase in query execution.

A `filesort` operation uses temporary disk files as necessary if the result set is too large to fit in memory. Some types of queries are particularly suited to completely in-memory `filesort` operations.
