# Jakarta Persistence 3.2 — ElementCollection Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/elementcollection
> Collected: 2026-09-29
> Published: Unknown

Module jakarta.persistence
Package jakarta.persistence

# Annotation Interface ElementCollection

@Target({METHOD,FIELD}) @Retention(RUNTIME) public @interface ElementCollection

Declares a collection of instances of a basic type or embeddable class. Must be specified if the collection is to be mapped by means of a collection table.

The `CollectionTable` annotation specifies a mapping to a database table.

Example:

```java
@Entity
public class Person {
    @Id
    protected String ssn;
    protected String name;
    ...
    @ElementCollection
    protected Set<String> nickNames = new HashSet<>();
    ...
}
```

Since: 2.0

See Also: CollectionTable

## Optional Element Summary

| Modifier and Type | Optional Element | Description |
|---|---|---|
| FetchType | fetch | (Optional) Whether the collection should be lazily loaded or must be eagerly fetched. |
| Class<?> | targetClass | (Optional) The basic or embeddable class that is the element type of the collection. |

## Element Details

### targetClass

Class<?> targetClass

(Optional) The basic or embeddable class that is the element type of the collection. This element is optional only if the collection field or property is defined using Java generics, and must be specified otherwise. It defaults to the parameterized type of the collection when defined using generics.

Default: void.class

### fetch

FetchType fetch

(Optional) Whether the collection should be lazily loaded or must be eagerly fetched.

- The EAGER strategy is a requirement on the persistence provider runtime that the associated entity must be eagerly fetched.
- The LAZY strategy is a hint to the persistence provider runtime.

If not specified, defaults to LAZY.

Default: LAZY
