# Jakarta Persistence — Embeddable classes, relationships and lifecycle

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Selected complete sections: 2.7. Embeddable Classes, 2.11. Entity Relationships, 3.3. Entity Instance’s Life Cycle. Copyright (c) 2024 Eclipse Foundation.

### 2.7. Embeddable Classes

An entity may use other fine-grained classes
to represent entity state. Instances of these classes, unlike entity
instances, do not have persistent identity of their own. Instead, they
exist only as part of the state of the entity to which they belong. An
entity may have collections of embeddables as well as single-valued
embeddable attributes. Embeddables may also be used as map keys and map
values. Embedded objects belong strictly to their owning entity, and are
not sharable across persistent entities. Attempting to share an embedded
object across entities has undefined semantics.

Embeddable classes must be annotated as
`Embeddable` or denoted in the XML descriptor as such. The access type
for an embedded object is determined as described in Section 2.3.

An embeddable class may be a regular Java class which adheres to the
requirements specified in Section 2.1 for entities, with the exception that
an embeddable class is not annotated as `Entity`, and an embeddable
class may not be abstract.

Alternatively, an embeddable class may be any Java record type.

An embeddable class may be used to represent
the state of another embeddable class.

An embeddable class (including an
embeddable class within another embeddable class) may contain a
collection of a basic type or other embeddable
class.[22]

An embeddable class may contain a
relationship to an entity or collection of entities. Since instances of
embeddable classes themselves have no persistent identity, the
relationship *from* the referenced entity is to the *entity* that
contains the embeddable instance(s) and not to the embeddable
itself.[23] An embeddable class that is used as an
embedded id or as a map key must not contain such a relationship.

Additional requirements and restrictions on
embeddable classes are described in Section 2.8.

### 2.11. Entity Relationships

Relationships among entities may be
one-to-one, one-to-many, many-to-one, or many-to-many. Relationships are
polymorphic.

If there is an association between two
entities, one of the following relationship modeling annotations must be
applied to the corresponding persistent property or field of the
referencing entity: `OneToOne`, `OneToMany`, `ManyToOne`,
`ManyToMany`. For associations that do not specify the target type
(e.g., where Java generic types are not used for collections), it is
necessary to specify the entity that is the target of the
relationship.[25] Equivalent XML elements may be used
as an alternative to these mapping annotations.

*These annotations mirror common practice in
relational database schema modeling. The use of the relationship
modeling annotations allows the object/relationship mapping of
associations to the relational database schema to be fully defaulted, to
provide an ease-of-development facility. This is described in Section 2.12.*

Relationships may be bidirectional or
unidirectional. A bidirectional relationship has both an owning side and
an inverse (non-owning) side. A unidirectional relationship has only an
owning side. The owning side of a relationship determines the updates to
the relationship in the database, as described in Section 3.3.4.

The following rules apply to bidirectional
relationships:

The inverse side of a bidirectional
relationship must refer to its owning side by use of the `mappedBy`
element of the `OneToOne`, `OneToMany`, or `ManyToMany` annotation.
The `mappedBy` element designates the property or field in the entity
that is the owner of the relationship.

- The many side of one-to-many / many-to-one
  bidirectional relationships must be the owning side, hence the
  `mappedBy` element cannot be specified on the `ManyToOne` annotation.
- For one-to-one bidirectional relationships,
  the owning side corresponds to the side that contains the corresponding
  foreign key.
- For many-to-many bidirectional relationships
  either side may be the owning side.

The relationship modeling annotation
constrains the use of the `cascade=REMOVE` specification. The
`cascade=REMOVE` specification should only be applied to associations
that are specified as `OneToOne` or `OneToMany`. Applications that
apply `cascade=REMOVE` to other associations are not portable.

Associations that are specified as `OneToOne`
or `OneToMany` support use of the `orphanRemoval` option. The following
behaviors apply when `orphanRemoval` is in effect:

- If an entity that is the target of the
  relationship is removed from the relationship (by setting the
  relationship to null or removing the entity from the relationship
  collection), the remove operation will be applied to the entity being
  orphaned. The remove operation is applied at the time of the flush
  operation. The `orphanRemoval` functionality is intended for entities
  that are privately “owned” by their parent entity. Portable applications
  must otherwise not depend upon a specific order of removal, and must not
  reassign an entity that has been orphaned to another relationship or
  otherwise attempt to persist it. If the entity being orphaned is a
  detached, new, or removed entity, the semantics of `orphanRemoval` do
  not apply.
- If the remove operation is applied to a
  managed source entity, the remove operation will be cascaded to the
  relationship target in accordance with the rules of Section 3.3.3,
  (and hence it is not necessary to specify `cascade=REMOVE` for the
  relationship)[26].

Section 2.12, defines relationship mapping defaults
for entity relationships. Additional mapping annotations (e.g., column
and table mapping annotations) may be specified to override or further
refine the default mappings and mapping strategies described in Section 2.12.

In addition, this specification also requires
support for the following alternative mapping strategies:

- The mapping of unidirectional one-to-many
  relationships by means of foreign key mappings. The `JoinColumn`
  annotation or corresponding XML element must be used to specify such
  non-default mappings. See Section 11.1.26.
- The mapping of unidirectional and
  bidirectional one-to-one relationships, bidirectional
  many-to-one/one-to-many relationships, and unidirectional many-to-one
  relationships by means of join table mappings. The `JoinTable`
  annotation or corresponding XML element must be used to specify such
  non-default mappings. See Section 11.1.28.

Such mapping annotations must be specified on
the owning side of the relationship. Any overriding of mapping defaults
must be consistent with the relationship modeling annotation that is
specified. For example, if a many-to-one relationship mapping is
specified, it is not permitted to specify a unique key constraint on the
foreign key for the relationship.

The persistence provider handles the
object/relational mapping of the relationships, including their loading
and storing to the database as specified in the metadata of the entity
class, and the referential integrity of the relationships as specified
in the database (e.g., by foreign key constraints).

|  |  |
| --- | --- |
|  | Note that it is the application that bears responsibility for maintaining the consistency of runtime relationships—for example, for insuring that the “one” and the “many” sides of a bidirectional relationship are consistent with one another when the application updates the relationship at runtime. |

If there are no associated entities for a
multi-valued relationship of an entity fetched from the database, the
persistence provider is responsible for returning an empty collection as
the value of the relationship.

### 3.3. Entity Instance’s Life Cycle

This section describes the `EntityManager`
operations for managing an entity instance’s lifecycle. An entity
instance can be characterized as being new, managed, detached, or
removed.

- A new entity instance has no persistent
  identity, and is not yet associated with a persistence context.
- A managed entity instance is an instance with
  a persistent identity that is currently associated with a persistence
  context.
- A detached entity instance is an instance
  with a persistent identity that is not (or no longer) associated with a
  persistence context.
- A removed entity instance is an instance with
  a persistent identity, associated with a persistence context, that will
  be removed from the database upon transaction commit.

The following subsections describe the effect
of lifecycle operations upon entities. Use of the `cascade` annotation
element may be used to propagate the effect of an operation to
associated entities. The cascade functionality is most typically used in
parent-child relationships.

#### 3.3.1. Entity Instance Creation

Entity instances are created by means of the
`new` operation. An entity instance, when first created by `new` is not
yet persistent. An instance becomes persistent by means of the
`EntityManager` API.

#### 3.3.2. Persisting an Entity Instance

A new entity instance becomes both managed
and persistent by invoking the `persist` method on it or by cascading
the persist operation.

The semantics of the persist operation,
applied to an entity `X` are as follows:

- If X is a new entity, it becomes managed. The
  entity X will be entered into the database at or before transaction
  commit or as a result of the flush operation.
- If X is a preexisting managed entity, it is
  ignored by the persist operation. However, the persist operation is
  cascaded to entities referenced by X, if the relationships from X to
  these other entities are annotated with the `cascade=PERSIST` or
  `cascade=ALL` annotation element value or specified with the equivalent
  XML descriptor element.
- If X is a removed entity, it becomes managed.
- If X is a detached object, the
  `EntityExistsException` may be thrown when the persist operation is
  invoked, or the `EntityExistsException` or another
  `PersistenceException` `may` be thrown at flush or commit time.
- For all entities Y referenced by a
  relationship from X, if the relationship to Y has been annotated with
  the `cascade` element value `cascade=PERSIST` or `cascade=ALL`, the
  persist operation is applied to Y.

#### 3.3.3. Removal

A managed entity instance becomes removed by
invoking the `remove` method on it or by cascading the remove operation.

The semantics of the remove operation,
applied to an entity X are as follows:

- If X is a new entity, it is ignored by the
  remove operation. However, the remove operation is cascaded to entities
  referenced by X, if the relationship from X to these other entities is
  annotated with the `cascade=REMOVE` or `cascade=ALL` annotation element
  value.
- If X is a managed entity, the remove
  operation causes it to become removed. The remove operation is cascaded
  to entities referenced by X, if the relationships from X to these other
  entities is annotated with the `cascade=REMOVE` or `cascade=ALL`
  annotation element value.
- If X is a detached entity, an
  `IllegalArgumentException` will be thrown by the remove operation (or
  the transaction commit will fail).
- If X is a removed entity, it is ignored by the remove operation.
- A removed entity X will be removed from the
  database at or before transaction commit or as a result of the flush
  operation.

After an entity has been removed, its state
(except for generated state) will be that of the entity at the point at
which the remove operation was called.

#### 3.3.4. Synchronization to the Database

In general, a persistence context will be
synchronized to the database as described below. However, a persistence
context of type `SynchronizationType.UNSYNCHRONIZED` or an
application-managed persistence context that has been created outside
the scope of the current transaction will only be synchronized to the
database if it has been joined to the current transaction by the
application’s use of the `EntityManager` `joinTransaction` method.

The state of persistent entities is
synchronized to the database at transaction commit. This synchronization
involves writing to the database any updates to persistent entities and
their relationships as specified above.

An update to the state of an entity includes
both the assignment of a new value to a persistent property or field of
the entity as well as the modification of a mutable value of a
persistent property or field[32].

Synchronization to the database does not
involve a refresh of any managed entities unless the `refresh` operation
is explicitly invoked on those entities or cascaded to them as a result
of the specification of the `cascade=REFRESH` or `cascade=ALL`
annotation element value.

Bidirectional relationships between
managed entities will be persisted based on references held by the
owning side of the relationship. It is the developer’s responsibility to
keep the in-memory references held on the owning side and those held on
the inverse side consistent with each other when they change. In the
case of unidirectional one-to-one and one-to-many relationships, it is
the developer’s responsibility to insure that the semantics of the
relationships are adhered to.[33]

|  |  |
| --- | --- |
|  | It is particularly important to ensure that changes to the inverse side of a relationship result in appropriate updates on the owning side, so as to ensure the changes are not lost when they are synchronized to the database. |

The persistence provider runtime is permitted
to perform synchronization to the database at other times as well when a
transaction is active and the persistence context is joined to the
transaction. The `flush` method can be used by the application to force
synchronization. It applies to entities associated with the persistence
context. The `setFlushMode` methods of the `EntityManager`, `Query`,
`TypedQuery`, and `StoredProcedureQuery` interfaces can be used to
control synchronization semantics. The effect of `FlushModeType.AUTO` is
defined in Section 3.11.2. If `FlushModeType.COMMIT` is specified, flushing will occur at
transaction commit; the persistence provider is permitted, but not
required, to perform to flush at other times. If there is no transaction
active or if the persistence context has not been joined to the current
transaction, the persistence provider must not flush to the database.

The semantics of the flush operation, applied
to an entity `X` are as follows:

- If X is a managed entity, it is synchronized
  to the database.

  - For all entities Y referenced by a
    relationship from X, if the relationship to Y has been annotated with
    the `cascade` element value `cascade=PERSIST` or `cascade=ALL`, the
    persist operation is applied to Y.
  - For any entity Y referenced by a relationship
    from X, where the relationship to Y has not been annotated with the
    `cascade` element value `cascade=PERSIST` or `cascade=ALL`:

    - If Y is new or removed, an
      `IllegalStateException` will be thrown by the flush operation (and the
      transaction marked for rollback) or the transaction commit will fail.
    - If Y is detached, the semantics depend upon
      the ownership of the relationship. If X owns the relationship, any
      changes to the relationship are synchronized with the database;
      otherwise, if Y owns the relationships, the behavior is undefined.
- If X is a removed entity, it is removed from
  the database. No cascade options are relevant.

#### 3.3.5. Refreshing an Entity Instance

The state of a managed entity instance is
refreshed from the database by invoking the `refresh` method on it or by
cascading the refresh operation.

The semantics of the refresh operation,
applied to an entity X are as follows:

- If X is a managed entity, the state of X is
  refreshed from the database, overwriting changes made to the entity, if
  any. The refresh operation is cascaded to entities referenced by X if
  the relationship from X to these other entities is annotated with the
  `cascade=REFRESH` or `cascade=ALL` annotation element value.
- If X is a new, detached, or removed entity,
  the `IllegalArgumentException` is thrown.

#### 3.3.6. Evicting an Entity Instance from the Persistence Context

An entity instance is removed from the
persistence context by invoking the `detach` method on it or cascading
the detach operation. Changes made to the entity, if any (including
removal of the entity), will not be synchronized to the database after
such eviction has taken place.

Applications must use the `flush` method
prior to the `detach` method to ensure portable semantics if changes
have been made to the entity (including removal of the entity). Because
the persistence provider may write to the database at times other than
the explicit invocation of the `flush` method, portable applications
must not assume that changes have not been written to the database if
the `flush` method has not been called prior to detach.

The semantics of the detach operation,
applied to an entity X are as follows:

- If X is a managed entity, the detach
  operation causes it to become detached. The detach operation is cascaded
  to entities referenced by X if the relationships from X to these other
  entities is annotated with the `cascade=DETACH` or `cascade=ALL`
  annotation element value. Entities which previously referenced X will
  continue to reference X.
- If X is a new or detached entity, it is
  ignored by the detach operation.
- If X is a removed entity, the detach
  operation causes it to become detached. The detach operation is cascaded
  to entities referenced by X if the relationships from X to these other
  entities is annotated with the `cascade=DETACH` or `cascade=ALL`
  annotation element value. Entities which previously referenced X will
  continue to reference X. Portable applications should not pass removed
  entities that have been detached from the persistence context to further
  EntityManager operations.

#### 3.3.7. Detached Entities

A detached entity results from transaction
commit if a transaction-scoped persistence context is used (see Section 3.4);
from transaction rollback (see Section 3.4.3); from detaching
the entity from the persistence context; from clearing the persistence
context; from closing an entity manager; or from serializing an entity
or otherwise passing an entity by value—e.g., to a separate application
tier, through a remote interface, etc.

Detached entity instances continue to live
outside of the persistence context in which they were persisted or
retrieved. Their state is no longer guaranteed to be synchronized with
the database state.

The application may access the available
state of available detached entity instances after the persistence
context ends. The available state includes:

- Any persistent field or property not marked `fetch=LAZY`
- Any persistent field or property that was
  accessed by the application or fetched by means of an entity graph

If the persistent field or property is an
association, the available state of an associated instance may only be
safely accessed if the associated instance is available. The available
instances include:

- Any entity instance retrieved using `find()`.
- Any entity instances retrieved using a query or explicitly requested in a fetch join.
- Any entity instance for which an instance
  variable holding non-primary-key persistent state was accessed by the
  application.
- Any entity instance that can be reached from
  another available instance by navigating associations marked `fetch=EAGER`.

##### 3.3.7.1. Merging Detached Entity State

The merge operation allows for the
propagation of state from detached entities onto persistent entities
managed by the entity manager.

The semantics of the merge operation applied
to an entity X are as follows:

- If X is a detached entity, the state of X is
  copied onto a pre-existing managed entity instance X' of the same
  identity or a new managed copy X' of X is created.
- If X is a new entity instance, a new managed
  entity instance X' is created and the state of X is `copied` into the
  new managed entity instance X'.
- If X is a removed entity instance, an
  `IllegalArgumentException` will be thrown by the merge operation (or the
  transaction commit will fail).
- If X is a managed entity, it is ignored by
  the merge operation, however, the merge operation is cascaded to
  entities referenced by relationships from X if these relationships have
  been annotated with the `cascade` element value `cascade=MERGE` or
  `cascade=ALL` annotation.
- For all entities Y referenced by
  relationships from X having the `cascade` element value `cascade=MERGE`
  or `cascade=ALL`, Y is merged recursively as Y'. For all such Y
  referenced by X, X' is set to reference Y'. (Note that if X is managed
  then X is the same object as X'.)
- If X is an entity merged to X', with a
  reference to another entity Y, where `cascade=MERGE` or `cascade=ALL` is
  not specified, then navigation of the same association from X' yields a
  reference to a managed object Y' with the same persistent identity as Y.

The persistence provider must not merge
fields marked LAZY that have not been fetched: it must ignore such
fields when merging.

Any `Version` columns used by the entity must
be checked by the persistence runtime implementation during the merge
operation and/or at flush or commit time. In the absence of `Version`
columns there is no additional version checking done by the persistence
provider runtime during the merge operation.

##### 3.3.7.2. Detached Entities and Lazy Loading

Serializing entities and merging those
entities back into a persistence context may not be interoperable across
vendors when lazy properties or fields and/or relationships are used.

A vendor is required to support the
serialization and subsequent deserialization and merging of detached
entity instances (which may contain lazy properties or fields and/or
relationships that have not been fetched) back into a separate JVM
instance of that vendor’s runtime, where both runtime instances have
access to the entity classes and any required vendor persistence
implementation classes.

When interoperability across vendors is
required, the application must not use lazy loading.

#### 3.3.8. Managed Instances

It is the responsibility of the application
to insure that an instance is managed in only a single persistence
context. The behavior is undefined if the same Java instance is made
managed in more than one persistence context.

The `contains()` method can be used to
determine whether an entity instance is managed in the current
persistence context.

The `contains` method returns true:

- If the entity has been retrieved from the
  database or has been returned by `getReference`, and has not been
  removed or detached.
- If the entity instance is new, and the
  `persist` method has been called on the entity or the persist operation
  has been cascaded to it.

The `contains` method returns false:

- If the instance is detached.
- If the `remove` method has been called on the
  entity, or the remove operation has been cascaded to it.
- If the instance is new, and the `persist`
  method has not been called on the entity or the persist operation has
  not been cascaded to it.

Note that the effect of the cascading of
persist, merge, remove, or detach is immediately visible to the
`contains` method, whereas the actual insertion, modification, or
deletion of the database representation for the entity may be deferred
until the end of the transaction.

#### 3.3.9. Load State

An entity is considered to be loaded if all
attributes with `FetchType.EAGER` —whether explictly specified or by
default—(including relationship and other collection-valued attributes)
have been loaded from the database or assigned by the application.
Attributes with `FetchType.LAZY` may or may not have been loaded. The
available state of the entity instance and associated instances is as
described in Section 3.3.7.

An attribute that is an embeddable is
considered to be loaded if the embeddable attribute was loaded from the
database or assigned by the application, and, if the attribute
references an embeddable instance (i.e., is not null), the embeddable
instance state is known to be loaded (i.e., all attributes of the
embeddable with `FetchType.EAGER` have been loaded from the database or
assigned by the application).

A collection-valued attribute is considered
to be loaded if the collection was loaded from the database or the value
of the attribute was assigned by the application, and, if the attribute
references a collection instance (i.e., is not null), each element of
the collection (e.g. entity or embeddable) is considered to be loaded.

A single-valued relationship attribute is
considered to be loaded if the relationship attribute was loaded from
the database or assigned by the application, and, if the attribute
references an entity instance (i.e., is not null), the entity instance
state is known to be loaded.

A basic attribute is considered to be loaded
if its state has been loaded from the database or assigned by the
application.

The `PersistenceUtil.isLoaded` methods can be
used to determine the load state of an entity and its attributes
regardless of the persistence unit with which the entity is associated.
The `PersistenceUtil.isLoaded` methods return true if the above
conditions hold, and false otherwise. If the persistence unit is known,
the `PersistenceUnitUtil.isLoaded` methods can be used instead. See Section 7.11.

Persistence provider contracts for
determining the load state of an entity or entity attribute are
described in Section 9.9.1.
