# Spring Data JPA Reference: Locking

> Source: https://docs.spring.io/spring-data/jpa/reference/jpa/locking.html
> Collected: 2026-10-02
> Published: Unknown

To specify the lock mode to be used, you can use the `@Lock` annotation on query methods, as shown in the following example:

Example 1. Defining lock metadata on query methods

```java
interface UserRepository extends Repository<User, Long> {

  // Plain query method
  @Lock(LockModeType.READ)
  List<User> findByLastname(String lastname);
}
```

This method declaration causes the query being triggered to be equipped with a `LockModeType` of `READ`. You can also define locking for CRUD methods by redeclaring them in your repository interface and adding the `@Lock` annotation, as shown in the following example:

Example 2. Defining lock metadata on CRUD methods

```java
interface UserRepository extends Repository<User, Long> {

  // Redeclaration of a CRUD method
  @Lock(LockModeType.READ)
  List<User> findAll();
}
```
