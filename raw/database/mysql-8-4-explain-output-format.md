# MySQL 8.4 Reference Manual: EXPLAIN Output Format

> Source: https://dev.mysql.com/doc/refman/8.4/en/explain-output.html
> Collected: 2026-09-29
> Published: Unknown

## EXPLAIN Output Columns

The `possible_keys` column indicates the indexes from which MySQL can choose to find the rows in this table.

The `key` column indicates the key (index) that MySQL actually decided to use. If MySQL decides to use one of the `possible_keys` indexes to look up rows, that index is listed as the key value.

If `key` is `NULL`, MySQL found no index to use for executing the query more efficiently.

The `key_len` column indicates the length of the key that MySQL decided to use. The value of `key_len` enables you to determine how many parts of a multiple-part key MySQL actually uses.

The `rows` column indicates the number of rows MySQL believes it must examine to execute the query. For `InnoDB` tables, this number is an estimate, and may not always be exact.

## EXPLAIN Extra Information

`Using filesort`: MySQL must do an extra pass to find out how to retrieve the rows in sorted order.

`Using index`: The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row.
