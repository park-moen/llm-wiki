# Jakarta Persistence 3.2: Optimistic Locking and Entity Versions

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2
> Collected: 2026-10-02
> Published: 2024-04-10

## 3.5.1. Optimistic Locking

Optimistic locking is a system of concurrency control where each revision of an item of data is assigned a version number or timestamp. When the data is read and then updated within a given unit of work, the version or timestamp is:

1. read from the database when the data itself is read, and
2. verified and then updated in the database when the data is updated.

An optimistic lock failure occurs when verification fails, that is, if the version or timestamp held in the database changes between reading the data (step 1), and attempting to update or delete the data (step 2).

Thus, the unit of work is prevented from updating the data and creating a new revision, or from deleting the data, unless the revision it previously obtained is still the current revision. Optimistic lock verification ensures that an update of a given item is successful only when no intervening transaction has already updated the item, preventing the loss of updates made by such intervening transactions.

The persistence provider is required to perform optimistic locking automatically for every entity with a version, as defined in Section 2.5. A portable application which wishes to take advantage of automatic optimistic locking must specify a version field or property for each optimistically-locked entity using the `@Version` annotation defined in Section 11.1.57 or equivalent XML element.

When an optimistic lock failure is detected, the persistence provider must:

* throw an `OptimisticLockException` and
* mark the current transaction for rollback.

Applications are strongly encouraged to enable optimistic locking for every entity which may be concurrently accessed or which may be merged from a detached state. Failure to make use of optimistic locking often leads to inconsistent entity state, lost updates, and other anomalies. If an entity does not have a version, the application itself must bear the burden of maintaining data consistency during optimistic units of work.

## 3.5.2. Entity Versions and Optimistic Locking

The entity version must be updated by the persistence provider each time the state of an entity instance is written to the database. Furthermore, if the current persistence context contains a revision of the entity instance when the instance is written to the database, the persistence provider must verify that the revision held in the persistence context is identical to the revision held in the database by comparing the versions held in memory and in the database.

The persistence provider must examine the version field or property of a detached entity instance when it is merged, as defined in Section 3.3.7.1, and throw an `OptimisticLockException` if the instance being merged holds a stale revision of the state of the entity—that is, if the entity was updated since the entity instance became detached. The timing of this version check is provider-dependent:

* the version check might occur synchronously with the call to `merge()`, or
* a provider might choose to delay the version check until a flush operation occurs, as defined in Section 3.3.4, or until the transaction commits.
