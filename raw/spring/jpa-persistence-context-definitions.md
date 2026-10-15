# Hibernate and Jakarta Persistence — Persistence context definitions

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/#pc; https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2.html#a11432
> Collected: 2026-10-15
> Published: Unknown
> Scope: Hibernate Persistence Context introduction and Jakarta Persistence section 7.1; original excerpts.

Both the `org.hibernate.Session` API and `jakarta.persistence.EntityManager` API represent a context for dealing with persistent data.
This concept is called a `persistence context`.
Persistent data has a state in relation to both a persistence context and the underlying database.

`transient`
:   the entity has just been instantiated and is not associated with a persistence context.
    It has no persistent representation in the database and typically no identifier value has been assigned (unless the *assigned* generator was used).

`managed` or `persistent`
:   the entity has an associated identifier and is associated with a persistence context.
    It may or may not physically exist in the database yet.

`detached`
:   the entity has an associated identifier but is no longer associated with a persistence context (usually because the persistence context was closed or the instance was evicted from the context)

`removed`
:   the entity has an associated identifier and is associated with a persistence context, however, it is scheduled for removal from the database.

Much of the `org.hibernate.Session` and `jakarta.persistence.EntityManager` methods deal with moving entities among these states.

### 7.1. Persistence Contexts

A persistence context is a set of managed
entity instances in which for any persistent entity identity there is a
unique entity instance. Within the persistence context, the entity
instances and their lifecycle are managed by the entity manager.

In Jakarta EE environments, a JTA transaction
typically involves calls across multiple components. Such components may
often need to access the same persistence context within a single
transaction. To facilitate such use of entity managers in Jakarta EE
environments, when an entity manager is injected into a component or
looked up directly in the JNDI naming context, its persistence context
will automatically be propagated with the current JTA transaction, and
the EntityManager references that are mapped to the same persistence
unit will provide access to this same persistence context within the JTA
transaction. This propagation of persistence contexts by the Jakarta EE
container avoids the need for the application to pass references to
EntityManager instances from one component to another. An entity manager
for which the container manages the persistence context in this manner
is termed a *container-managed entity manager*. A container-managed
entity manager’s lifecycle is managed by the Jakarta EE container.

In less common use cases within Jakarta EE
environments, applications may need to access a persistence context that
is “stand-alone”—i.e. not propagated along with the JTA transaction
across the EntityManager references for the given persistence unit.
Instead, each instance of creating an entity manager causes a new
isolated persistence context to be created that is not accessible
through other EntityManager references within the same transaction.
These use cases are supported through the `createEntityManager` methods
of the `EntityManagerFactory` interface. An entity manager that is used
by the application to create and destroy a persistence context in this
manner is termed an *application-managed entity manager*. An
application-managed entity manager’s lifecycle is managed by the
application.

Both container-managed entity managers and
application-managed entity managers and their persistence contexts are
required to be supported in Jakarta EE web containers and EJB containers.
Within an EJB environment, container-managed entity managers are
typically used.

In Java SE environments and in Jakarta EE
application client containers, only application-managed entity managers
are required to be [[87](#_footnotedef_87 "View footnote.")].
