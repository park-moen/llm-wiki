# Hibernate ORM 7.1 SQLRestriction Javadoc

> Source: https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/SQLRestriction.html
> Collected: 2026-10-02
> Published: Unknown

Specifies a restriction written in native SQL to add to the generated SQL for entities or collections.

For example, `@SQLRestriction` could be used to hide entity instances which have been soft-deleted, either for the entity class itself:

```java
@Entity
@SQLRestriction("status <> 'DELETED'")
class Document {
    @Enumerated(STRING)
    Status status;
}
```

or, at the level of an association to the entity:

```java
@OneToMany(mappedBy = "owner")
@SQLRestriction("status <> 'DELETED'")
List<Document> documents;
```

The `SQLJoinTableRestriction` annotation lets a restriction be applied to an association table.

Note that `@SQLRestriction`s are always applied and cannot be disabled. Nor may they be parameterized. They're therefore much less flexible than filters.

Since: 6.3

`value`: A predicate, written in native SQL.
