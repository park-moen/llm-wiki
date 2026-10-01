# Spring Data JPA: JPA Query Methods

> Source: https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html
> Collected: 2026-10-01
> Published: Unknown

## Query Lookup Strategies

The JPA module supports defining a query manually as a String or having it being derived from the method name.

## Query Creation

Generally, the query creation mechanism for JPA works as described in Query Methods. The following example shows what a JPA query method translates into:

```java
public interface UserRepository extends Repository<User, Long> {
    List<User> findByEmailAddressAndLastname(String emailAddress, String lastname);
}
```

We create a query using JPQL translating into the following query: `select u from User u where u.emailAddress = ?1 and u.lastname = ?2`. Spring Data JPA does a property check and traverses nested properties, as described in Property Expressions.
