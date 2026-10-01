# MariaDB MDEV-15140: Implement Partial / Filtered Indexes

> Source: https://jira.mariadb.org/browse/MDEV-15140
> Collected: 2026-10-01
> Published: 2018-01-31

## Details

Type: New Feature

Status: Open

Resolution: Unresolved

Fix Version/s: None

## Description

It would be nice, if MariaDB has some way to index only subset of values / rows.

For example:
We have 1% of rows value set to "active" and 99% of rows set to "inactive". There is no need to index "inactive" rows. It would be similar to table scan. But indexing "active" rows would work well.

This feature is called Partial Index or Filtered Index in other databases.

There is no way to create this index in MySQL / Mariadb. Closest thing would be using another table and trigger. This complicate a lot of things and isn't much useful.
