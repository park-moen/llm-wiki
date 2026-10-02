# Hibernate ORM 7.1 User Guide: Filtering entities and associations

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/
> Collected: 2026-10-02
> Published: Unknown

## 6.9. Filtering entities and associations

Hibernate offers two options if you want to filter entities or entity associations:

static (e.g. `@SQLRestriction` and `@SQLJoinTableRestriction`) which are defined at mapping time and cannot change at runtime.

dynamic (e.g. `@Filter` and `@FilterJoinTable`) which are applied and configured at runtime.

## 6.9.1. `@SQLRestriction`

Sometimes, you want to filter out entities or collections using custom SQL criteria. This can be achieved using the `@SQLRestriction` annotation, which can be applied to entities and collections.

```java
@SQLRestriction("account_type = 'DEBIT'")
@OneToMany(mappedBy = "client")
private List<Account> debitAccounts = new ArrayList<>();

@Entity(name = "Account")
@SQLRestriction("active = true")
public static class Account {
    // ...
}
```

When executing an `Account` entity query, Hibernate is going to filter out all records that are not active.

```sql
SELECT
    a.id as id1_0_,
    a.active as active2_0_,
    a.amount as amount3_0_,
    a.client_id as client_i6_0_,
    a.rate as rate4_0_,
    a.account_type as account_5_0_
FROM
    Account a
WHERE ( a.active = true )
```

When fetching the `debitAccounts` or the `creditAccounts` collections, Hibernate is going to apply the `@SQLRestriction` clause filtering criteria to the associated child entities.

```sql
SELECT
    d.client_id as client_i6_0_0_,
    d.id as id1_0_0_,
    d.active as active2_0_1_,
    d.amount as amount3_0_1_,
    d.account_type as account_5_0_1_
FROM
    Account d
WHERE ( d.active = true and d.account_type = 'DEBIT' ) AND d.client_id = 1
```

## 6.9.3. `@Filter`

The `@Filter` annotation is another way to filter out entities or collections using custom SQL criteria. Unlike the `@SQLRestriction` annotation, `@Filter` allows you to parameterize the filter clause at runtime.
