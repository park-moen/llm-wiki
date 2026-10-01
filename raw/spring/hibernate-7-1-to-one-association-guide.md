# A Short Guide to Hibernate 7: To-one associations

> Source: https://docs.hibernate.org/orm/7.1/introduction/html_single/
> Collected: 2026-10-01
> Published: Unknown

## Many-to-one

The `@ManyToOne` annotation marks the "to one" side of the association, so a unidirectional many-to-one association looks like this:

```java
class Book {
    @Id @GeneratedValue
    Long id;

    @ManyToOne(fetch=LAZY)
    Publisher publisher;
}
```

Here, the `Book` table has a foreign key column holding the identifier of the associated `Publisher`.

A very unfortunate misfeature of JPA is that `@ManyToOne` associations are fetched eagerly by default. This is almost never what we want. Almost all associations should be lazy. The only scenario in which `fetch=EAGER` makes sense is if we think there’s always a very high probability that the associated object will be found in the second-level cache. Whenever this isn’t the case, remember to explicitly specify `fetch=LAZY`.

## One-to-one (first way)

The simplest sort of one-to-one association is almost exactly like a `@ManyToOne` association, except that it maps to a foreign key column with a `UNIQUE` constraint.

A one-to-one association must be annotated `@OneToOne`:

```java
@Entity
class Author {
    @Id @GeneratedValue
    Long id;

    @OneToOne(optional=false, fetch=LAZY)
    Person person;
}
```

Here, the `Author` table has a foreign key column holding the identifier of the associated `Person`.

## Lazy fetching for one-to-one associations

Notice that we did not declare the unowned end of the association `fetch=LAZY`. That’s because:

1. not every `Person` has an associated `Author`, and
2. the foreign key is held in the table mapped by `Author`, not in the table mapped by `Person`.

Therefore, Hibernate can’t tell if the reference from `Person` to `Author` is `null` without fetching the associated `Author`.

On the other hand, if every `Person` was an `Author`, that is, if the association were non-`optional`, we would not have to consider the possibility of `null` references, and we would map it like this:

```java
@OneToOne(optional=false, mappedBy = Author_.PERSON, fetch=LAZY)
Author author;
```

## One-to-one (second way)

An arguably more elegant way to represent such a relationship is to share a primary key between the two tables.

```java
@OneToOne(optional=false, fetch=LAZY)
@MapsId
Person person;
```

This lets Hibernate know that the association to `Person` is the source of primary key values for `Author`.
