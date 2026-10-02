# Jakarta Persistence 3.2: Persistent fields and collection-valued attributes

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2
> Collected: 2026-10-02
> Published: Unknown

The type of a persistent field or property of an entity class may be:

* any basic type listed below in Section 2.6, including any Java `enum` type,
* an entity type or a collection of some entity type, as specified in Section 2.11,
* an embeddable class, as defined in Section 2.7, or
* a collection of a basic type or embeddable type, as specified in Section 2.8.

Collection-valued persistent fields and properties must be defined in terms of one of the following collection-valued interfaces, regardless of whether the entity class otherwise adheres to the JavaBeans method conventions noted below, and of whether field or property access is used: `java.util.Collection`, `java.util.Set`, `java.util.List`, `java.util.Map`.

The terms “collection” and “collection-valued” are used in this specification to denote any of the above types, unless further qualified.
