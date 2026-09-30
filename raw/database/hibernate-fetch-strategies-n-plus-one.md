# Hibernate ORM 7.1 User Guide: Fetching strategies

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/
> Collected: 2026-09-30
> Published: Unknown

## The basics

The concept of fetching breaks down into two different questions.

- When should the data be fetched? Now? Later?
- How should the data be fetched?

SELECT

Performs a separate SQL select to load the data. This can either be EAGER (the second select is issued immediately) or LAZY (the second select is delayed until the data is needed). This is the strategy generally termed N+1.

JOIN

Inherently an EAGER style of fetching. The data to be fetched is obtained through the use of an SQL outer join.

BATCH

Performs a separate SQL select to load a number of related data items using an IN-restriction as part of the SQL WHERE-clause based on a batch size. Again, this can either be EAGER (the second select is issued immediately) or LAZY (the second select is delayed until the data is needed).
