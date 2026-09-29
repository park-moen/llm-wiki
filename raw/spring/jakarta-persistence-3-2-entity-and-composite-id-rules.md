# Jakarta Persistence 3.2 — Entity and composite primary key rules

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2
> Collected: 2026-09-29
> Published: 2024-04-10
> Scope: Selected passages from sections 2.1, 2.4 and 2.4.1.

## 2.1. The Entity Class

The entity class must have a public or protected constructor with no parameters, which is called by the persistence provider runtime to instantiate the entity. The entity class may have additional constructors for use by the application.

The entity class must be non-final. Every method and persistent instance variable of the entity class must be non-final.

## 2.4. Primary Keys and Entity Identity

A composite primary key must correspond to either a single persistent field or property, or to a set of fields or properties, as described below. A primary key class must be defined to represent the composite primary key.

If the composite primary key corresponds to a single field or property of the entity, the EmbeddedId annotation identifies the primary key, and the type of the annotated field or property is the primary key class.

Otherwise, when the composite primary key corresponds to multiple fields or properties, the Id annotation identifies the fields and properties which comprise the composite key, and the IdClass annotation must specify the primary key class.

## 2.4.1. Composite primary keys

The primary key class may be a non-abstract regular Java class with a public or protected constructor with no parameters. Alternatively, the primary key class may be any Java record type, in which case it need not have a constructor with no parameters.

The primary key class must define equals and hashCode methods. The semantics of value equality for these methods must be consistent with the database equality for the database types to which the key is mapped.

A composite primary key must either be represented and mapped as an embeddable class or it must be represented as an id class and mapped to multiple fields or properties of the entity class.

If the composite primary key class is represented as an id class, the names of primary key fields or properties of the primary key class and those of the entity class to which the id class is mapped must correspond and their types must be the same.
