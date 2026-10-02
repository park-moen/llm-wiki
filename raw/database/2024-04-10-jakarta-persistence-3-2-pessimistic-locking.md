# Jakarta Persistence: Pessimistic Locking and Lock Modes

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2
> Collected: 2026-10-02
> Published: 2024-04-10

## 3.5.3. Pessimistic Locking

While optimistic locking is typically appropriate in dealing with moderate contention among concurrent transactions, in some applications it may be useful to immediately obtain long-term database locks for selected entities because of the often late failure of optimistic transactions. Such immediately obtained long-term database locks are referred to here as “pessimistic” locks.

Pessimistic locking guarantees that once a transaction has obtained a pessimistic lock on an entity instance:

* no other transaction (whether a transaction of an application using the Jakarta Persistence API or any other transaction using the underlying resource) may successfully modify or delete that instance until the transaction holding the lock has ended.
* if the pessimistic lock is an exclusive lock, that same transaction may modify or delete that entity instance.

This specification does not define the mechanisms a persistence provider uses to obtain database locks, and a portable application should not rely on how pessimistic locking is achieved on the database. In particular, a persistence provider or the underlying database management system may lock more rows than the ones selected by the application.

Pessimistic locking may be applied to entities that do not contain version attributes. However, in this case correct interaction with applications using optimistic locking cannot be ensured.

## 3.5.4.2. PESSIMISTIC_READ, PESSIMISTIC_WRITE, PESSIMISTIC_FORCE_INCREMENT

The lock modes `PESSIMISTIC_READ`, `PESSIMISTIC_WRITE`, and `PESSIMISTIC_FORCE_INCREMENT` are used to immediately obtain long-term database locks.

Any such lock must be obtained immediately and retained until transaction T1 completes (commits or rolls back).

A lock with `LockModeType.PESSIMISTIC_WRITE` can be obtained on an entity instance to force serialization among transactions attempting to update the entity data. A lock with `LockModeType.PESSIMISTIC_READ` can be used to query data using repeatable-read semantics without the need to reread the data at the end of the transaction to obtain a lock, and without blocking other transactions reading the data.

A lock with `LockModeType.PESSIMISTIC_WRITE` can be used when querying data and there is a high likelihood of deadlock or update failure among concurrent updating transactions.

The persistence implementation must support calling `lock(entity, LockModeType.PESSIMISTIC_READ)` and `lock(entity, LockModeType.PESSIMISTIC_WRITE)` on a non-versioned entity as well as on a versioned entity.

When the lock cannot be obtained, and the database locking failure results in transaction-level rollback, the provider must throw the `PessimisticLockException` and ensure that the JTA transaction or EntityTransaction has been marked for rollback.

When the lock cannot be obtained, and the database locking failure results in only statement-level rollback, the provider must throw the `LockTimeoutException` (and must not mark the transaction for rollback).

When `lock(entity, LockModeType.PESSIMISTIC_READ)`, `lock(entity, LockModeType.PESSIMISTIC_WRITE)`, or `lock(entity, LockModeType.PESSIMISTIC_FORCE_INCREMENT)` is invoked on a versioned entity that is already in the persistence context, the provider must also perform optimistic version checks when obtaining the lock. An `OptimisticLockException` must be thrown if the version checks fail.
