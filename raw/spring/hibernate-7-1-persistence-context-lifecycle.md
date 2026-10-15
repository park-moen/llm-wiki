# Hibernate ORM — Persistence Context and entity lifecycle

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/#pc
> Collected: 2026-10-15
> Published: Unknown
> Scope: Selected introductory paragraphs and lifecycle sections; version 7.1.36.Final. Filtering and cascade sections are omitted.



### 6.3. Making entities persistent

Once you’ve created a new entity instance (using the standard `new` operator) it is in `new` state.
You can make it persistent by associating it to either an `org.hibernate.Session` or a `jakarta.persistence.EntityManager`.

Example 362. Making an entity persistent with Jakarta Persistence

```
Person person = new Person();
person.setId(1L);
person.setName("John Doe");

entityManager.persist(person);
```

Example 363. Making an entity persistent with Hibernate API

```
Person person = new Person();
person.setId(1L);
person.setName("John Doe");

session.persist(person);
```

`org.hibernate.Session` also has a method named persist which follows the exact semantics defined in the Jakarta Persistence specification for the persist method.
It is this `org.hibernate.Session` method to which the Hibernate `jakarta.persistence.EntityManager` implementation delegates.

Instances of entity types using generated identifiers will be automatically associated with
an identifier value when the save or persist operation is called.
If an entity type does not rely on a generated id, then an identifier value
(usually natural) must be manually assigned to the entity instance before the save or persist operations can be called.

### 6.4. Deleting (removing) entities

Entities can also be deleted.

Example 364. Deleting an entity with Jakarta Persistence

```
entityManager.remove(person);
```

Example 365. Deleting an entity with the Hibernate API

```
session.remove(person);
```

|  |  |
| --- | --- |
|  | Hibernate itself can handle deleting entities in detached state. Jakarta Persistence, however, disallows this behavior.  The implication here is that the entity instance passed to the `org.hibernate.Session` delete method can be either in managed or detached state, while the entity instance passed to remove on `jakarta.persistence.EntityManager` must be in the managed state. |

### 6.5. Obtain an entity reference without initializing its data

Sometimes referred to as lazy loading, the ability to obtain a reference to an entity without having to load its data is hugely important.
The most common case being the need to create an association between an entity and another existing entity.

Example 366. Obtaining an entity reference without initializing its data with Jakarta Persistence

```
Book book = new Book();
book.setAuthor(entityManager.getReference(Person.class, personId));
```

Example 367. Obtaining an entity reference without initializing its data with Hibernate API

```
Book book = new Book();
book.setId(1L);
book.setIsbn("123-456-7890");
entityManager.persist(book);
book.setAuthor(session.getReference(Person.class, personId));
```

The above works on the assumption that the entity is defined to allow lazy loading, generally through use of runtime proxies.
In both cases an exception will be thrown later if the given entity does not refer to actual database state when the application attempts to use the returned proxy in any way that requires access to its data.

|  |  |
| --- | --- |
|  | Unless the entity class is declared `final`, the proxy extends the entity class. If the entity class is `final`, the proxy will implement an interface instead. See the `@Proxy` mapping section for more info. |

### 6.6. Obtain an entity with its data initialized

It is also quite common to want to obtain an entity along with its data (e.g. like when we need to display it in the UI).

Example 368. Obtaining an entity reference with its data initialized with Jakarta Persistence

```
Person person = entityManager.find(Person.class, personId);
```

Example 369. Obtaining an entity reference with its data initialized with Hibernate API

```
Person person = session.get(Person.class, personId);
```

Example 370. Obtaining an entity reference with its data initialized using the `byId()` Hibernate API

```
Person person = session.byId(Person.class).load(personId);
```

In both cases null is returned if no matching database row was found.

It’s possible to return a Java 8 `Optional` as well:

Example 371. Obtaining an Optional entity reference with its data initialized using the `byId()` Hibernate API

```
Optional<Person> optionalPerson = session.byId(Person.class).loadOptional(personId);
```

### 6.10. Modifying managed/persistent state

Entities in managed/persistent state may be manipulated by the application, and any changes will be automatically detected and persisted when the persistence context is flushed.
There is no need to call a particular method to make your modifications persistent.

Example 398. Modifying managed state with Jakarta Persistence

```
Person person = entityManager.find(Person.class, personId);
person.setName("John Doe");
entityManager.flush();
```

Example 399. Modifying managed state with Hibernate API

```
Person person = session.byId(Person.class).load(personId);
person.setName("John Doe");
session.flush();
```

By default, when you modify an entity, all columns but the identifier are being set during update.

Therefore, considering you have the following `Product` entity mapping:

Example 400. `Product` entity mapping

```
@Entity(name = "Product")
public static class Product {

	@Id
	private Long id;

	@Column
	private String name;

	@Column
	private String description;

	@Column(name = "price_cents")
	private Integer priceCents;

	@Column
	private Integer quantity;

	//Getters and setters are omitted for brevity

}
```

If you persist the following `Product` entity:

Example 401. Persisting a `Product` entity

```
Product book = new Product();
book.setId(1L);
book.setName("High-Performance Java Persistence");
book.setDescription("Get the most out of your persistence layer");
book.setPriceCents(29_99);
book.setQuantity(10_000);

entityManager.persist(book);
```

When you modify the `Product` entity, Hibernate generates the following SQL UPDATE statement:

Example 402. Modifying the `Product` entity

```
doInJPA(this::entityManagerFactory, entityManager -> {
	Product book = entityManager.find(Product.class, 1L);
	book.setPriceCents(24_99);
});
```

```
UPDATE
    Product 
SET
    description = ?,
    name = ?,
    price_cents = ?,
    quantity = ? 
WHERE
    id = ?
        
-- binding parameter [1] as [VARCHAR] - [Get the most out of your persistence layer]
-- binding parameter [2] as [VARCHAR] - [High-Performance Java Persistence]
-- binding parameter [3] as [INTEGER] - [2499]
-- binding parameter [4] as [INTEGER] - [10000]
-- binding parameter [5] as [BIGINT]  - [1]
```

The default UPDATE statement containing all columns has two advantages:

- it allows you to better benefit from JDBC Statement caching.
- it allows you to enable batch updates even if multiple entities modify different properties.

However, there is also one downside to including all columns in the SQL UPDATE statement.
If you have multiple indexes, the database might update those redundantly even if you don’t actually modify all column values.

To fix this issue, you can use dynamic updates.

#### 6.10.1. Dynamic updates

To enable dynamic updates, you need to annotate the entity with the `@DynamicUpdate` annotation:

Example 403. `Product` entity mapping

```
@Entity(name = "Product")
@DynamicUpdate
public static class Product {

	@Id
	private Long id;

	@Column
	private String name;

	@Column
	private String description;

	@Column(name = "price_cents")
	private Integer priceCents;

	@Column
	private Integer quantity;

	//Getters and setters are omitted for brevity

}
```

This time, when rerunning the previous test case, Hibernate generates the following SQL UPDATE statement:

Example 404. Modifying the `Product` entity with a dynamic update

```
UPDATE
    Product
SET
    price_cents = ?
WHERE
    id = ?

-- binding parameter [1] as [INTEGER] - [2499]
-- binding parameter [2] as [BIGINT]  - [1]
```

The dynamic update allows you to set just the columns that were modified in the associated entity.

### 6.11. Refresh entity state

You can reload an entity instance and its collections at any time.

Example 405. Refreshing entity state with Jakarta Persistence

```
Person person = entityManager.find(Person.class, personId);

entityManager.createQuery("update Person set name = UPPER(name)").executeUpdate();

entityManager.refresh(person);
assertEquals("JOHN DOE", person.getName());
```

Example 406. Refreshing entity state with Hibernate API

```
Person person = session.byId(Person.class).load(personId);

session.doWork(connection -> {
	try(Statement statement = connection.createStatement()) {
		statement.executeUpdate("UPDATE Person SET name = UPPER(name)");
	}
});

session.refresh(person);
assertEquals("JOHN DOE", person.getName());
```

One case where this is useful is when it is known that the database state has changed since the data was read.
Refreshing allows the current database state to be pulled into the entity instance and the persistence context.

Another case where this might be useful is when database triggers are used to initialize some of the properties of the entity.

|  |  |
| --- | --- |
|  | Only the entity instance and its value type collections are refreshed unless you specify `REFRESH` as a cascade style of any associations. However, please note that Hibernate has the capability to handle this automatically through its notion of generated properties. See the discussion of non-identifier generated attributes. |

|  |  |
| --- | --- |
|  | Traditionally, Hibernate allowed detached entities to be refreshed. however, Jakarta Persistence prohibits this practice and specifies that an `IllegalArgumentException` should be thrown instead. This is the default behaviour from version 7.0 onwards. |

#### 6.11.1. Refresh gotchas

The `refresh` entity state transition is meant to overwrite the entity attributes according to the info currently contained in the associated database record.

However, you have to be very careful when cascading the refresh action to any transient entity.

For instance, consider the following example:

Example 407. Refreshing entity state gotcha

```
try {
	Person person = entityManager.find(Person.class, personId);

	Book book = new Book();
	book.setId(100L);
	book.setTitle("Hibernate User Guide");
	book.setAuthor(person);
	person.getBooks().add(book);

	entityManager.refresh(person);
}
catch (EntityNotFoundException expected) {
	log.info("Beware when cascading the refresh associations to transient entities!");
}
```

In the aforementioned example, an `EntityNotFoundException` is thrown because the `Book` entity is still in a transient state.
When the refresh action is cascaded from the `Person` entity, Hibernate will not be able to locate the `Book` entity in the database.

For this reason, you should be very careful when mixing the refresh action with transient child entity objects.

### 6.12. Working with detached data

Detachment is the process of working with data outside the scope of any persistence context.
Data becomes detached in a number of ways.
Once the persistence context is closed, all data that was associated with it becomes detached.
Clearing the persistence context has the same effect.
Evicting a particular entity from the persistence context makes it detached.
And finally, serialization will make the deserialized form be detached (the original instance is still managed).

Detached data can still be manipulated, however, the persistence context will no longer automatically know about these modifications, and the application will need to intervene to make the changes persistent again.

#### 6.12.1. Merging detached data

Merging is the process of taking an incoming entity instance that is in the detached state and copying its data over onto a new managed instance.

Although not exactly per se, the following example is a good visualization of the `merge` operation internals.

Example 408. Visualizing merge

```
public Person merge(Person detached) {
	Person newReference = session.byId(Person.class).load(detached.getId());
	newReference.setName(detached.getName());
	return newReference;
}
```

Example 409. Merging a detached entity with Jakarta Persistence

```
Person person = entityManager.find(Person.class, personId);
//Clear the EntityManager so the person entity becomes detached
entityManager.clear();
person.setName("Mr. John Doe");

person = entityManager.merge(person);
```

Example 410. Merging a detached entity with Hibernate API

```
Person person = session.byId(Person.class).load(personId);
//Clear the Session so the person entity becomes detached
session.clear();
person.setName("Mr. John Doe");

person = (Person) session.merge(person);
```

##### Merging gotchas

For example, Hibernate throws `IllegalStateException` when merging a parent entity which has references to 2 detached child entities `child1` and `child2` (obtained from different sessions), and `child1` and `child2` represent the same persistent entity, `Child`.

A new configuration property, `hibernate.event.merge.entity_copy_observer`, controls how Hibernate will respond when multiple representations of the same persistent entity ("entity copy") is detected while merging.

The possible values are:

disallow (the default)
:   throws `IllegalStateException` if an entity copy is detected

allow
:   performs the merge operation on each entity copy that is detected

log
:   (provided for testing only) performs the merge operation on each entity copy that is detected and logs information about the entity copies.
    This setting requires DEBUG logging be enabled for `org.hibernate.event.internal.EntityCopyAllowedLoggedObserver`

In addition, the application may customize the behavior by providing an implementation of `org.hibernate.event.spi.EntityCopyObserver` and setting `hibernate.event.merge.entity_copy_observer` to the class name.
When this property is set to `allow` or `log`, Hibernate will merge each entity copy detected while cascading the merge operation.
In the process of merging each entity copy, Hibernate will cascade the merge operation from each entity copy to its associations with `cascade=CascadeType.MERGE` or `CascadeType.ALL`.
The entity state resulting from merging an entity copy will be overwritten when another entity copy is merged.

|  |  |
| --- | --- |
|  | Because cascade order is undefined, the order in which the entity copies are merged is undefined. As a result, if property values in the entity copies are not consistent, the resulting entity state will be indeterminate, and data will be lost from all entity copies except for the last one merged. Therefore, the **last writer wins**.  If an entity copy cascades the merge operation to an association that is (or contains) a new entity, that new entity will be merged (i.e., persisted and the merge operation will be cascaded to its associations according to its mapping), even if that same association is ultimately overwritten when Hibernate merges a different representation having a different value for its association.  If the association is mapped with `orphanRemoval = true`, the new entity will not be deleted because the semantics of orphanRemoval do not apply if the entity being orphaned is a new entity.  There are known issues when representations of the same persistent entity have different values for a collection. See [HHH-9239](https://hibernate.atlassian.net/browse/HHH-9239) and [HHH-9240](https://hibernate.atlassian.net/browse/HHH-9240) for more details. These issues can cause data loss or corruption.  By setting `hibernate.event.merge.entity_copy_observer` configuration property to `allow` or `log`, Hibernate will allow entity copies of any type of entity to be merged.  The only way to exclude particular entity classes or associations that contain critical data is to provide a custom implementation of `org.hibernate.event.spi.EntityCopyObserver` with the desired behavior, and setting `hibernate.event.merge.entity_copy_observer` to the class name. |

|  |  |
| --- | --- |
|  | Hibernate provides limited DEBUG logging capabilities that can help determine the entity classes for which entity copies were found. By setting `hibernate.event.merge.entity_copy_observer` to `log` and enabling DEBUG logging for `org.hibernate.event.internal.EntityCopyAllowedLoggedObserver`, the following will be logged each time an application calls `EntityManager.merge( entity )` or  `Session.merge( entity )`:  - number of times multiple representations of the same persistent entity was detected summarized by entity name; - details by entity name and ID, including output from calling toString() on each representation being merged as well as the merge result.  The log should be reviewed to determine if multiple representations of entities containing critical data are detected. If so, the application should be modified so there is only one representation, and a custom implementation of `org.hibernate.event.spi.EntityCopyObserver` should be provided to disallow entity copies for entities with critical data.  Using optimistic locking is recommended to detect if different representations are from different versions of the same persistent entity. If they are not from the same version, Hibernate will throw either the Jakarta Persistence `OptimisticEntityLockException` or the native `StaleObjectStateException` depending on your bootstrapping strategy. |

### 6.13. Checking persistent state

An application can verify the state of entities and collections in relation to the persistence context.

Example 411. Verifying managed state with Jakarta Persistence

```
boolean contained = entityManager.contains(person);
```

Example 412. Verifying managed state with Hibernate API

```
boolean contained = session.contains(person);
```

Example 413. Verifying laziness with Jakarta Persistence

```
PersistenceUnitUtil persistenceUnitUtil = entityManager.getEntityManagerFactory().getPersistenceUnitUtil();

boolean personInitialized = persistenceUnitUtil.isLoaded(person);

boolean personBooksInitialized = persistenceUnitUtil.isLoaded(person.getBooks());

boolean personNameInitialized = persistenceUnitUtil.isLoaded(person, "name");
```

Example 414. Verifying laziness with Hibernate API

```
boolean personInitialized = Hibernate.isInitialized(person);

boolean personBooksInitialized = Hibernate.isInitialized(person.getBooks());

boolean personNameInitialized = Hibernate.isPropertyInitialized(person, "name");
```

In Jakarta Persistence there is an alternative means to check laziness using the following `jakarta.persistence.PersistenceUtil` pattern (which is recommended wherever possible).

Example 415. Alternative Jakarta Persistence means to verify laziness

```
PersistenceUtil persistenceUnitUtil = Persistence.getPersistenceUtil();

boolean personInitialized = persistenceUnitUtil.isLoaded(person);

boolean personBooksInitialized = persistenceUnitUtil.isLoaded(person.getBooks());

boolean personNameInitialized = persistenceUnitUtil.isLoaded(person, "name");
```
