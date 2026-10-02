# MariaDB: Database vs. Schema

> Source: https://mariadb.com/docs/server/server-management/install-and-upgrade-mariadb/migrating-to-mariadb/differences-between-mariadb-and-other-dbmss
> Collected: 2026-10-02
> Published: Unknown

In MariaDB, the terms schema and database are synonymous and used interchangeable. The `CREATE SCHEMA` statement is a synonym for `CREATE DATABASE`, meaning they create the same object—a container for database objects like tables and views.

MariaDB does not support a distinction between schema and database as seen in other systems like SQL Server or PostgreSQL, where a schema is a logical container within a database. Instead, a database in MariaDB serves as both a namespace and a logical container to separate objects, and it has a default character set and collation that are inherited by its tables.
