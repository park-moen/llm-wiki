# MySQL 8.4 Reference Manual: Descending Indexes

> Source: https://dev.mysql.com/doc/refman/8.4/en/descending-indexes.html
> Collected: 2026-09-29
> Published: Unknown

## Descending Indexes

MySQL supports descending indexes: `DESC` in an index definition is no longer ignored but causes storage of key values in descending order. Previously, indexes could be scanned in reverse order but at a performance penalty. A descending index can be scanned in forward order, which is more efficient. Descending indexes also make it possible for the optimizer to use multiple-column indexes when the most efficient scan order mixes ascending order for some columns and descending order for others.

The optimizer can perform a forward index scan for each of the `ORDER BY` clauses and need not use a `filesort` operation:

```sql
ORDER BY c1 ASC, c2 ASC    -- optimizer can use idx1
ORDER BY c1 DESC, c2 DESC  -- optimizer can use idx4
ORDER BY c1 ASC, c2 DESC   -- optimizer can use idx2
ORDER BY c1 DESC, c2 ASC   -- optimizer can use idx3
```

Descending indexes are supported only for the `InnoDB` storage engine. Descending indexes are supported for `BTREE` but not `HASH` indexes.
