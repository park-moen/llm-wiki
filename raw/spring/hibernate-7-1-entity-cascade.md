# Hibernate ORM — Cascading entity state transitions

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/#pc-cascade
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named section with subsections; version 7.1.36.Final.

### 6.15. Cascading entity state transitions

Jakarta Persistence allows you to propagate the state transition from a parent entity to a child.
For this purpose, the Jakarta Persistence `jakarta.persistence.CascadeType` defines various cascade types:

`ALL`
:   cascades all entity state transitions.

`PERSIST`
:   cascades the entity persist operation.

`MERGE`
:   cascades the entity merge operation.

`REMOVE`
:   cascades the entity remove operation.

`REFRESH`
:   cascades the entity refresh operation.

`DETACH`
:   cascades the entity detach operation.

Additionally, the `CascadeType.ALL` will propagate any Hibernate-specific operation, which is defined by the `org.hibernate.annotations.CascadeType` enum:

`SAVE_UPDATE`
:   cascades the entity saveOrUpdate operation.

`REPLICATE`
:   cascades the entity replicate operation.

`LOCK`
:   cascades the entity lock operation.

The following examples will explain some of the aforementioned cascade operations using the following entities:

```
@Entity
public class Person {

	@Id
	private Long id;

	private String name;

	@OneToMany(mappedBy = "owner", cascade = CascadeType.ALL)
	private List<Phone> phones = new ArrayList<>();

	//Getters and setters are omitted for brevity

	public void addPhone(Phone phone) {
		this.phones.add(phone);
		phone.setOwner(this);
	}
}

@Entity
public class Phone {

	@Id
	private Long id;

	@Column(name = "`number`")
	private String number;

	@ManyToOne(fetch = FetchType.LAZY)
	private Person owner;

	//Getters and setters are omitted for brevity
}
```

#### 6.15.1. `CascadeType.PERSIST`

The `CascadeType.PERSIST` allows us to persist a child entity along with the parent one.

Example 420. `CascadeType.PERSIST` example

```
Person person = new Person();
person.setId(1L);
person.setName("John Doe");

Phone phone = new Phone();
phone.setId(1L);
phone.setNumber("123-456-7890");

person.addPhone(phone);

entityManager.persist(person);
```

```
INSERT INTO Person ( name, id )
VALUES ( 'John Doe', 1 )

INSERT INTO Phone ( `number`, person_id, id )
VALUE ( '123-456-7890', 1, 1 )
```

Even if just the `Person` parent entity was persisted, Hibernate has managed to cascade the persist operation to the associated `Phone` child entity as well.

#### 6.15.2. `CascadeType.MERGE`

The `CascadeType.MERGE` allows us to merge a child entity along with the parent one.

Example 421. `CascadeType.MERGE` example

```
Phone phone = entityManager.find(Phone.class, 1L);
Person person = phone.getOwner();

person.setName("John Doe Jr.");
phone.setNumber("987-654-3210");

entityManager.clear();

entityManager.merge(person);
```

```
SELECT
    p.id as id1_0_1_,
    p.name as name2_0_1_,
    ph.owner_id as owner_id3_1_3_,
    ph.id as id1_1_3_,
    ph.id as id1_1_0_,
    ph."number" as number2_1_0_,
    ph.owner_id as owner_id3_1_0_ 
FROM
    Person p 
LEFT OUTER JOIN
    Phone ph 
        on p.id=ph.owner_id 
WHERE
    p.id = 1
```

During merge, the current state of the entity is copied onto the entity version that was just fetched from the database.
That’s the reason why Hibernate executed the SELECT statement which fetched both the `Person` entity along with its children.

#### 6.15.3. `CascadeType.REMOVE`

The `CascadeType.REMOVE` allows us to remove a child entity along with the parent one.
Traditionally, Hibernate called this operation delete, that’s why the `org.hibernate.annotations.CascadeType` provides a `DELETE` cascade option.
However, `CascadeType.REMOVE` and `org.hibernate.annotations.CascadeType.DELETE` are identical.

Example 422. `CascadeType.REMOVE` example

```
Person person = entityManager.find(Person.class, 1L);

entityManager.remove(person);
```

```
DELETE FROM Phone WHERE id = 1

DELETE FROM Person WHERE id = 1
```

#### 6.15.4. `CascadeType.DETACH`

`CascadeType.DETACH` is used to propagate the detach operation from a parent entity to a child.

Example 423. `CascadeType.DETACH` example

```
Person person = entityManager.find(Person.class, 1L);
assertEquals(1, person.getPhones().size());
Phone phone = person.getPhones().get(0);

assertTrue(entityManager.contains(person));
assertTrue(entityManager.contains(phone));

entityManager.detach(person);

assertFalse(entityManager.contains(person));
assertFalse(entityManager.contains(phone));
```

#### 6.15.5. `CascadeType.LOCK`

Although unintuitively, `CascadeType.LOCK` does not propagate a lock request from a parent entity to its children.
Such a use case requires the use of the `PessimisticLockScope.EXTENDED` value of the `jakarta.persistence.lock.scope` property.

#### 6.15.6. `CascadeType.REFRESH`

The `CascadeType.REFRESH` is used to propagate the refresh operation from a parent entity to a child.
The refresh operation will discard the current entity state, and it will override it using the one loaded from the database.

Example 424. `CascadeType.REFRESH` example

```
Person person = entityManager.find(Person.class, 1L);
Phone phone = person.getPhones().get(0);

person.setName("John Doe Jr.");
phone.setNumber("987-654-3210");

entityManager.refresh(person);

assertEquals("John Doe", person.getName());
assertEquals("123-456-7890", phone.getNumber());
```

```
SELECT
    p.id as id1_0_1_,
    p.name as name2_0_1_,
    ph.owner_id as owner_id3_1_3_,
    ph.id as id1_1_3_,
    ph.id as id1_1_0_,
    ph."number" as number2_1_0_,
    ph.owner_id as owner_id3_1_0_ 
FROM
    Person p 
LEFT OUTER JOIN
    Phone ph 
        ON p.id=ph.owner_id 
WHERE
    p.id = 1
```

In the aforementioned example, you can see that both the `Person` and `Phone` entities are refreshed even if we only called this operation on the parent entity only.

#### 6.15.7. `CascadeType.REPLICATE`

The `CascadeType.REPLICATE` is to replicate both the parent and the child entities.
The replicate operation allows you to synchronize entities coming from different sources of data.

Example 425. `CascadeType.REPLICATE` example

```
Person person = new Person();
person.setId(1L);
person.setName("John Doe Sr.");

Phone phone = new Phone();
phone.setId(1L);
phone.setNumber("(01) 123-456-7890");
person.addPhone(phone);

entityManager.unwrap(Session.class).replicate(person, ReplicationMode.OVERWRITE);
```

```
SELECT
    id 
FROM
    Person 
WHERE
    id = 1
    
SELECT
    id 
FROM
    Phone 
WHERE
    id = 1

UPDATE
    Person 
SET
    name = 'John Doe Sr.'
WHERE
    id = 1

UPDATE
    Phone 
SET
    "number" = '(01) 123-456-7890',
    owner_id = 1 
WHERE
    id = 1
```

As illustrated by the SQL statements being generated, both the `Person` and `Phone` entities are replicated to the underlying database rows.

#### 6.15.8. `@OnDelete` cascade

While the previous cascade types propagate entity state transitions, the `@OnDelete` cascade is a DDL-level FK feature which allows you
to remove a child record whenever the parent row is deleted.

So, when annotating the `@ManyToOne` association with `@OnDelete( action = OnDeleteAction.CASCADE )`,
the automatic schema generator will apply the ON DELETE CASCADE SQL directive to the Foreign Key declaration,
as illustrated by the following example.

Example 426. `@OnDelete` `@ManyToOne` mapping

```
@Entity(name = "Person")
public static class Person {

	@Id
	private Long id;

	private String name;

	//Getters and setters are omitted for brevity

}
```

```
@Entity(name = "Phone")
public static class Phone {

	@Id
	private Long id;

	@Column(name = "`number`")
	private String number;

	@ManyToOne(fetch = LAZY)
	@OnDelete(action = CASCADE)
	private Person owner;

	//Getters and setters are omitted for brevity

}
```

```
create table Person (
    id bigint not null,
    name varchar(255),
    primary key (id)
)

create table Phone (
    id bigint not null,
    "number" varchar(255),
    owner_id bigint,
    primary key (id)
)

alter table Phone
    add constraint FK82m836qc1ss2niru7eogfndhl
    foreign key (owner_id)
    references Person
    on delete cascade
```

Now, you can just remove the `Person` entity, and the associated `Phone` entities are going to be deleted automatically via the Foreign Key cascade.

Example 427. `@OnDelete` `@ManyToOne` delete example

```
Person person = entityManager.find(Person.class, 1L);
entityManager.remove(person);
```

```
delete from Person where id = ?

-- binding parameter [1] as [BIGINT] - [1]
```

The `@OnDelete` annotation can also be placed on a collection, as
illustrated in the following example.

Example 428. `@OnDelete` `@OneToMany` mapping

```
@Entity(name = "Person")
public static class Person {

	@Id
	private Long id;

	private String name;

	@OneToMany(mappedBy = "owner", cascade = CascadeType.ALL)
	@OnDelete(action = OnDeleteAction.CASCADE)
	private List<Phone> phones = new ArrayList<>();

	//Getters and setters are omitted for brevity

}
```

```
@Entity(name = "Phone")
public static class Phone {

	@Id
	private Long id;

	@Column(name = "`number`")
	private String number;

	@ManyToOne(fetch = FetchType.LAZY)
	private Person owner;

	//Getters and setters are omitted for brevity

}
```

Now, when removing the `Person` entity, all the associated `Phone` child entities are deleted via the Foreign Key cascade even if the `@OneToMany` collection was using the `CascadeType.ALL` attribute.

Example 429. `@OnDelete` `@ManyToOne` delete example

```
Person person = entityManager.find(Person.class, 1L);
entityManager.remove(person);
```

```
delete from Person where id = ?

-- binding parameter [1] as [BIGINT] - [1]
```

|  |  |
| --- | --- |
|  | Without the `@OnDelete` annotation, the `@OneToMany` association relies on the `cascade` attribute to propagate the `remove` entity state transition from the parent entity to its children. However, when the `@OnDelete` annotation is in place, Hibernate prevents the child entity `DELETE` statement from being executed while flushing the Persistence Context.  This way, only the parent entity gets deleted, and all the associated child records are removed by the database engine, instead of being deleted explicitly via `DELETE` statements. |
