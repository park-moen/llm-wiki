# Hibernate ORM — Embeddable values

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/#embeddables
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named section with subsections; version 7.1.36.Final.

### 3.3. Embeddable values

Historically Hibernate called these components.
Jakarta Persistence calls them embeddables.
Either way, the concept is the same: a composition of values.

For example, we might have a `Publisher` class that is a composition of `name` and `country`,
or a `Location` class that is a composition of `country` and `city`.

|  |  |
| --- | --- |
|  | Usage of the word *embeddable*  To avoid any confusion with the annotation that marks a given embeddable type, the annotation will be further referred to as `@Embeddable`.  Throughout this chapter and thereafter, for brevity sake, embeddable types may also be referred to as *embeddable*. |

Example 104. Embeddable type example

```
@Embeddable
public static class Publisher {

	private String name;

	private Location location;

	public Publisher(String name, Location location) {
		this.name = name;
		this.location = location;
	}

	private Publisher() {}

	//Getters and setters are omitted for brevity
}

@Embeddable
public static class Location {

	private String country;

	private String city;

	public Location(String country, String city) {
		this.country = country;
		this.city = city;
	}

	private Location() {}

	//Getters and setters are omitted for brevity
}
```

An embeddable type is another form of a value type, and its lifecycle is bound to a parent entity type, therefore inheriting the attribute access from its parent (for details on attribute access, see Access strategies).

Embeddable types can be made up of basic values as well as associations, with the caveat that, when used as collection elements, they cannot define collections themselves.

#### 3.3.1. Component / Embedded

Most often, embeddable types are used to group multiple basic type mappings and reuse them across several entities.

Example 105. Simple Embeddable

```
@Entity(name = "Book")
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	private Publisher publisher;

	//Getters and setters are omitted for brevity
}

@Embeddable
public static class Publisher {

	@Column(name = "publisher_name")
	private String name;

	@Column(name = "publisher_country")
	private String country;

	//Getters and setters, equals and hashCode methods omitted for brevity

}
```

```
create table Book (
    id bigint not null,
    author varchar(255),
    publisher_country varchar(255),
    publisher_name varchar(255),
    title varchar(255),
    primary key (id)
)
```

|  |  |
| --- | --- |
|  | Jakarta Persistence defines two terms for working with an embeddable type: `@Embeddable` and `@Embedded`.  `@Embeddable` is used to describe the mapping type itself (e.g. `Publisher`).  `@Embedded` is for referencing a given embeddable type (e.g. `book.publisher`). |

So, the embeddable type is represented by the `Publisher` class and
the parent entity makes use of it through the `book#publisher` object composition.

The composed values are mapped to the same table as the parent table.
Composition is part of good object-oriented data modeling (idiomatic Java).
In fact, that table could also be mapped by the following entity type instead.

Example 106. Alternative to embeddable type composition

```
@Entity(name = "Book")
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	@Column(name = "publisher_name")
	private String publisherName;

	@Column(name = "publisher_country")
	private String publisherCountry;

	//Getters and setters are omitted for brevity
}
```

The composition form is certainly more object-oriented, and that becomes more evident as we work with multiple embeddable types.

#### 3.3.2. Overriding Embeddable types

Although from an object-oriented perspective, it’s much more convenient to work with embeddable types, when we reuse the same
embeddable multiple times on the same class, the Jakarta Persistence specification requires to set the associated column names explicitly.

This requirement is due to how object properties are mapped to database columns.
By default, Jakarta Persistence expects a database column having the same name with its associated object property.
When including multiple embeddables, the implicit name-based mapping rule doesn’t work anymore because multiple object
properties could end-up being mapped to the same database column.

When an embeddable type is used multiple times, Jakarta Persistence defines the `@AttributeOverride`
and `@AssociationOverride` annotations to handle this scenario to override the default column names defined
by the Embeddable.

|  |  |
| --- | --- |
|  | See Embeddables and ImplicitNamingStrategy for an alternative to using `@AttributeOverride` and `@AssociationOverride` |

Considering you have the following `Publisher` embeddable type
which defines a `@ManyToOne` association with the `Country` entity:

Example 107. Embeddable type with a `@ManyToOne` association

```
@Embeddable
public static class Publisher {

	private String name;

	@ManyToOne(fetch = FetchType.LAZY)
	private Country country;

	//Getters and setters, equals and hashCode methods omitted for brevity

}

@Entity(name = "Country")
public static class Country {

	@Id
	@GeneratedValue
	private Long id;

	@NaturalId
	private String name;

	//Getters and setters are omitted for brevity
}
```

```
create table Country (
    id bigint not null,
    name varchar(255),
    primary key (id)
)

alter table Country
    add constraint UK_p1n05aafu73sbm3ggsxqeditd
    unique (name)
```

Now, if you have a `Book` entity which declares two `Publisher` embeddable types for the ebook and paperback versions,
you cannot use the default `Publisher` embeddable mapping since there will be a conflict between the two embeddable column mappings.

Therefore, the `Book` entity needs to override the embeddable type mappings for each `Publisher` attribute:

Example 108. Overriding embeddable type attributes

```
@Entity(name = "Book")
@AttributeOverrides({
		@AttributeOverride(
				name = "ebookPublisher.name",
				column = @Column(name = "ebook_pub_name")
		),
		@AttributeOverride(
				name = "paperBackPublisher.name",
				column = @Column(name = "paper_back_pub_name")
		)
})
@AssociationOverrides({
		@AssociationOverride(
				name = "ebookPublisher.country",
				joinColumns = @JoinColumn(name = "ebook_pub_country_id")
		),
		@AssociationOverride(
				name = "paperBackPublisher.country",
				joinColumns = @JoinColumn(name = "paper_back_pub_country_id")
		)
})
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	private Publisher ebookPublisher;

	private Publisher paperBackPublisher;

	//Getters and setters are omitted for brevity
}
```

```
create table Book (
    id bigint not null,
    author varchar(255),
    ebook_pub_name varchar(255),
    paper_back_pub_name varchar(255),
    title varchar(255),
    ebook_pub_country_id bigint,
    paper_back_pub_country_id bigint,
    primary key (id)
)

alter table Book
    add constraint FKm39ibh5jstybnslaoojkbac2g
    foreign key (ebook_pub_country_id)
    references Country

alter table Book
    add constraint FK7kqy9da323p7jw7wvqgs6aek7
    foreign key (paper_back_pub_country_id)
    references Country
```

#### 3.3.3. @EmbeddedColumnNaming

The most common use case for `@AttributeOverride` in relation to an embeddable is to rename the associated columns.

Consider a typical embeddable mapping -

Example 109. Typical embeddable mapping

```
@Entity
class Person {
	// ...
    @Embedded
    Address homeAddress;
    @Embedded
    Address workAddress;
}

@Embeddable
class Address {
	String street;
	String city;
	// ...
}
```

In strict Jakarta Persistence sense, this will lead to a bootstrapping error because
Jakarta Persistence requires that the implicit names for the columns for both of the
embedded `Address` mappings to be based on the attribute names from the embeddable -
`street`, `city`, etc. However, that will lead to duplicate column names here.

The strict compliance way to accomplish this would be a tedious use of the `@AttributeOverride` annotation
as discussed previously.

Since this is such a common pattern, Hibernate offers a much simpler solution through its `@EmbeddedColumnNaming` annotation
which allows to "patternize" the column naming -

Example 110. Simple `@EmbeddedColumnNaming` example

```
@Entity
class Person {
	// ...
    @Embedded
    @EmbeddedColumnNaming("home_%s")
    Address homeAddress;
    @Embedded
    @EmbeddedColumnNaming("work_%s")
    Address workAddress;
}
```

This mapping produces implicit column names `home_street`, `home_city`, `work_street`, `work_city`, etc.

`@EmbeddedColumnNaming` also works in nested usages and plays nicely with explicit column names.

Example 111. Explicit @Column example

```
@Entity
class Person {
	// ...
    @Embedded
	@EmbeddedColumnNaming("home_%s")
    Address homeAddress;
    @Embedded
    @EmbeddedColumnNaming("work_%s")
    Address workAddress;
}

@Embeddable
class Address {
	String street;
	String city;
    @Embedded
    private ZipPlus zip;
	// ...
}

@Embeddable
public static class ZipPlus {
    @Column(name="zip_code")
    private String code;
    @Column(name="zip_plus4")
    private String plus4;
}
```

This will produce implicit column names `home_street`, `home_city`, `home_zip_code`, `home_zip_plus4`, …​

When `@EmbeddedColumnNaming` is used withing nested embeddables, the affect will be cumulative.
Given the following model:

Example 112. Cumulative @EmbeddedColumnNaming example

```
@Entity
class Person {
	// ...
    @Embedded
	@EmbeddedColumnNaming("home_%s")
    Address homeAddress;
    @Embedded
    @EmbeddedColumnNaming("work_%s")
    Address workAddress;
}

@Embeddable
class Address {
	String street;
	String city;
    @Embedded
    @EmbeddedColumnNaming("zip_%s")
    private ZipPlus zip;
	// ...
}

@Embeddable
public static class ZipPlus {
    private String code;
    private String plus4;
}
```

Here we will end up with the columns `home_street`, `home_city`, `home_zip_code`, `home_zip_plus4`, …​

#### 3.3.4. Collections of embeddable types

Collections of embeddable types are specifically valued collections (as embeddable types are value types).
Value collections are covered in detail in Collections of value types.

#### 3.3.5. Embeddable type as a Map key

Embeddable types can also be used as `Map` keys.
This topic is converted in detail in Map - key.

#### 3.3.6. Embeddable type as identifier

Embeddable types can also be used as entity type identifiers.
This usage is covered in detail in Composite identifiers.

|  |  |
| --- | --- |
|  | Embeddable types that are used as collection entries, map keys or entity type identifiers cannot include their own collection mappings. |

#### 3.3.7. `@TargetEmbeddable` mapping

The [`@TargetEmbeddable`](https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/TargetEmbeddable.html) annotation is used to specify the implementation class of an embeddable-valued mapping when the declared type is a non-concrete type (interface, etc).

As an example, consider an `@Embedded` attribute (`City.coordinates`) whose declared type (`Coordinates`) is an interface. However, Hibernate needs to know about the concrete type to use, which is `GPS` in this case. The `@TargetEmbeddable` annotation is used to provide this information.

Example 113. Embedded mapping example

```
public interface Coordinates {
	double x();
	double y();
}

@Embeddable
public static class GPS implements Coordinates {
	// Omitted for brevity

}

@Entity(name = "City")
public static class City {
	// Omitted for brevity

	@Embedded
	@TargetEmbeddable(GPS.class)
	private Coordinates coordinates;
}
```

The mapping works just like any embeddable-valued mapping at this point, with Hibernate using `GPS` "under the covers":

Example 114. Usage example

```
City city = session.find(City.class, 1L);
assert city.getCoordinates() instanceof GPS;
```

For `@Embedded` and `@EmbeddedId` cases, `@TargetEmbeddable` may be specified on the member’s declared type instead of the member itself:

Example 115. Interface mapping example

```
@TargetEmbeddable(GPS.class)
public interface Coordinates {
	double x();
	double y();
}

@Embeddable
public static class GPS implements Coordinates {
	// Omitted for brevity

}

@Entity(name = "City")
public static class City {
	// Omitted for brevity

	@Embedded
	private Coordinates coordinates;
}
```

#### 3.3.8. `@Parent` mapping

The Hibernate-specific `@Parent` annotation allows you to reference the owner entity from within an embeddable.

Example 116. `@Parent` mapping usage

```
@Embeddable
public static class GPS {

	private double latitude;

	private double longitude;

	@Parent
	private City city;

	//Getters and setters omitted for brevity

}

@Entity(name = "City")
public static class City {

	@Id
	@GeneratedValue
	private Long id;

	private String name;

	@Embedded
	@TargetEmbeddable(GPS.class)
	private GPS coordinates;

	//Getters and setters omitted for brevity

}
```

Assuming we have persisted the following `City` entity:

Example 117. `@Parent` persist example

```
doInJPA(this::entityManagerFactory, entityManager -> {

	City cluj = new City();
	cluj.setName("Cluj");
	cluj.setCoordinates(new GPS(46.77120, 23.62360));

	entityManager.persist(cluj);
});
```

When fetching the `City` entity, the `city` property of the embeddable type acts as a back reference to the owning parent entity:

Example 118. `@Parent` fetching example

```
doInJPA(this::entityManagerFactory, entityManager -> {

	City cluj = entityManager.find(City.class, 1L);

	assertSame(cluj, cluj.getCoordinates().getCity());
});
```

Therefore, the `@Parent` annotation is used to define the association between an embeddable type and the owning entity.

#### 3.3.9. Custom instantiation

Jakarta Persistence requires embeddable classes to follow Java Bean conventions. Part of this is the
definition of a non-arg constructor. However, not all value compositions applications might map as embeddable
values follow Java Bean conventions - e.g. a struct or Java 15 record.

Hibernate allows the use of a custom instantiator for creating the embeddable instances through the
`org.hibernate.metamodel.spi.EmbeddableInstantiator` contract. For example, consider the following
embeddable:

Example 119. `EmbeddableInstantiator` - Embeddable

```
@Embeddable
public class Name {
	@Column(name = "first_name")
	private final String first;
	@Column(name = "last_name")
	private final String last;

	private Name() {
		throw new UnsupportedOperationException();
	}

	public Name(String first, String last) {
		this.first = first;
		this.last = last;
	}

	public String getFirstName() {
		return first;
	}

	public String getLastName() {
		return last;
	}
}
```

Here, `Name` only allows use of the constructor accepting its state. Because this class does not follow Java Bean
conventions, in terms of constructor, a custom strategy for instantiation is needed.

Example 120. `EmbeddableInstantiator` - Implementation

```
public class NameInstantiator implements EmbeddableInstantiator {
	@Override
	public Object instantiate(ValueAccess valueAccess) {
		// alphabetical
		final String first = valueAccess.getValue( 0, String.class );
		final String last = valueAccess.getValue( 1, String.class );
		return new Name( first, last );
	}

	// ...

}
```

There are a few ways to specify the custom instantiator. The `@org.hibernate.annotations.EmbeddableInstantiator`
annotation can be used on the embedded attribute:

Example 121. `@EmbeddableInstantiator` on attribute

```
@Entity
public class Person {
	@Id
	public Integer id;
	@Embedded
	@EmbeddableInstantiator( NameInstantiator.class )
	public Name name;
	@ElementCollection
	@Embedded
	@EmbeddableInstantiator( NameInstantiator.class )
	public Set<Name> aliases;

}
```

`@EmbeddableInstantiator` may also be specified on the embeddable class:

Example 122. `@EmbeddableInstantiator` on class

```
@Embeddable
@EmbeddableInstantiator( NameInstantiator.class )
public class Name {
	@Column(name = "first_name")
	private final String first;
	@Column(name = "last_name")
	private final String last;

	private Name() {
		throw new UnsupportedOperationException();
	}

	public Name(String first, String last) {
		this.first = first;
		this.last = last;
	}

	public String getFirstName() {
		return first;
	}

	public String getLastName() {
		return last;
	}
}

@Entity
public class Person {
	@Id
	public Integer id;
	@Embedded
	public Name name;
	@ElementCollection
	@Embedded
	public Set<Name> aliases;
}
```

Lastly, `@org.hibernate.annotations.EmbeddableInstantiatorRegistration` may be used, which is useful
when the application developer does not control the embeddable to be able to apply the instantiator
on the embeddable.

Example 123. `@EmbeddableInstantiatorRegistration`

```
@Entity
@EmbeddableInstantiatorRegistration( embeddableClass = Name.class, instantiator = NameInstantiator.class )
public class Person {
	@Id
	public Integer id;
	@Embedded
	public Name name;
	@ElementCollection
	@Embedded
	public Set<Name> aliases;

}
```

#### 3.3.10. Custom type mapping

Another approach is to supply the implementation of the `org.hibernate.usertype.CompositeUserType` contract using `@CompositeType`,
which is an extension to the `org.hibernate.metamodel.spi.EmbeddableInstantiator` contract.

There are also corresponding, specialized forms of `@CompositeType` for specific model parts:

- When mapping a Map, `@CompositeType` describes the Map value while `@MapKeyCompositeType` describes the Map key
- For collection mappings, `@CompositeType` describes the elements

For example, consider the following custom type:

Example 124. `CompositeUserType` - Domain type

```
public class Name {
	private final String first;
	private final String last;

	public Name(String first, String last) {
		this.first = first;
		this.last = last;
	}

	public String firstName() {
		return first;
	}

	public String lastName() {
		return last;
	}
}
```

Here, `Name` only allows use of the constructor accepting its state. Because this class does not follow Java Bean
conventions, a custom user type for instantiation and state access is needed.

Example 125. `CompositeUserType` - Implementation

```
public class NameCompositeUserType implements CompositeUserType<Name> {

	public static class NameMapper {
		String firstName;
		String lastName;
	}

	@Override
	public Class<?> embeddable() {
		return NameMapper.class;
	}

	@Override
	public Class<Name> returnedClass() {
		return Name.class;
	}

	@Override
	public Name instantiate(ValueAccess valueAccess) {
		// alphabetical
		final String first = valueAccess.getValue( 0, String.class );
		final String last = valueAccess.getValue( 1, String.class );
		return new Name( first, last );
	}

	@Override
	public Object getPropertyValue(Name component, int property) throws HibernateException {
		// alphabetical
		switch ( property ) {
			case 0:
				return component.firstName();
			case 1:
				return component.lastName();
		}
		return null;
	}

	@Override
	public boolean equals(Name x, Name y) {
		return x == y || x != null && Objects.equals( x.firstName(), y.firstName() )
				&& Objects.equals( x.lastName(), y.lastName() );
	}

	@Override
	public int hashCode(Name x) {
		return Objects.hash( x.firstName(), x.lastName() );
	}

	@Override
	public Name deepCopy(Name value) {
		return value; // immutable
	}

	@Override
	public boolean isMutable() {
		return false;
	}

	@Override
	public Serializable disassemble(Name value) {
		return new String[] { value.firstName(), value.lastName() };
	}

	@Override
	public Name assemble(Serializable cached, Object owner) {
		final String[] parts = (String[]) cached;
		return new Name( parts[0], parts[1] );
	}

	@Override
	public Name replace(Name detached, Name managed, Object owner) {
		return detached;
	}

}
```

A composite user type needs an embeddable mapper class, which represents the embeddable mapping structure of the type
i.e. the way the type would look like if you had the option to write a custom `@Embeddable` class.

In addition to the instantiation logic, a composite user type also has to provide a way to decompose the returned type
into the individual components/properties of the embeddable mapper class through `getPropertyValue`.
The property index, just like in the `instantiate` method, is based on the alphabetical order of the attribute names
of the embeddable mapper class.

The composite user type also needs to provide methods to handle the mutability, equals, hashCode and the cache
serialization and deserialization of the returned type.

There are a few ways to specify the composite user type. The `@org.hibernate.annotations.CompositeType`
annotation can be used on the embedded and element collection attributes:

Example 126. `@CompositeType` on attribute

```
@Entity
public class Person {
	@Id
	public Integer id;

	@Embedded
	@AttributeOverride(name = "firstName", column = @Column(name = "first_name"))
	@AttributeOverride(name = "lastName", column = @Column(name = "last_name"))
	@CompositeType( NameCompositeUserType.class )
	public Name name;

	@ElementCollection
	@AttributeOverride(name = "firstName", column = @Column(name = "first_name"))
	@AttributeOverride(name = "lastName", column = @Column(name = "last_name"))
	@CompositeType( NameCompositeUserType.class )
	public Set<Name> aliases;

}
```

Or `@org.hibernate.annotations.CompositeTypeRegistration` may be used, which is useful
when the application developer wants to apply the composite user type for all domain type uses.

Example 127. `@CompositeTypeRegistration`

```
@Entity
@CompositeTypeRegistration( embeddableClass = Name.class, userType = NameCompositeUserType.class )
public class Person {
	@Id
	public Integer id;

	@Embedded
	@AttributeOverride(name = "firstName", column = @Column(name = "first_name"))
	@AttributeOverride(name = "lastName", column = @Column(name = "last_name"))
	public Name name;

	@ElementCollection
	@AttributeOverride(name = "firstName", column = @Column(name = "first_name"))
	@AttributeOverride(name = "lastName", column = @Column(name = "last_name"))
	public Set<Name> aliases;

}
```

#### 3.3.11. Embeddables and ImplicitNamingStrategy

|  |  |
| --- | --- |
|  | The `ImplicitNamingStrategyComponentPathImpl` is a Hibernate-specific feature. Users concerned with Jakarta Persistence provider portability should instead prefer explicit column naming with `@AttributeOverride`. |

Hibernate naming strategies are covered in detail in Naming.
However, for the purposes of this discussion, Hibernate has the capability to interpret implicit column names in a way that is safe for use with multiple embeddable types.

Example 128. Implicit multiple embeddable type mapping

```
@Entity(name = "Book")
public static class Book {

	@Id
	@GeneratedValue
	private Long id;

	private String title;

	private String author;

	private Publisher ebookPublisher;

	private Publisher paperBackPublisher;

	//Getters and setters are omitted for brevity
}

@Embeddable
public static class Publisher {

	private String name;

	@ManyToOne(fetch = FetchType.LAZY)
	private Country country;

	//Getters and setters, equals and hashCode methods omitted for brevity
}

@Entity(name = "Country")
public static class Country {

	@Id
	@GeneratedValue
	private Long id;

	@NaturalId
	private String name;

	//Getters and setters are omitted for brevity
}
```

To make it work, you need to use the `ImplicitNamingStrategyComponentPathImpl` naming strategy.

Example 129. Enabling implicit embeddable type mapping using the component path naming strategy

```
metadataBuilder.applyImplicitNamingStrategy(
	ImplicitNamingStrategyComponentPathImpl.INSTANCE
);
```

Now the "path" to attributes are used in the implicit column naming:

```
create table Book (
    id bigint not null,
    author varchar(255),
    ebookPublisher_name varchar(255),
    paperBackPublisher_name varchar(255),
    title varchar(255),
    ebookPublisher_country_id bigint,
    paperBackPublisher_country_id bigint,
    primary key (id)
)
```

You could even develop your own naming strategy to do other types of implicit naming strategies.

#### 3.3.12. Aggregate embeddable mapping

An embeddable mapping is usually just a way to encapsulate columns of a table into a Java type,
but as of Hibernate 6.2, it is also possible to map embeddable types as SQL aggregate types.

Currently, there are three possible SQL aggregate types which can be specified by annotating one of the following
annotations on a persistent attribute:

- `@Struct` - maps to a named SQL object type
- `@JdbcTypeCode(SqlTypes.JSON)` - maps to the SQL type JSON
- `@JdbcTypeCode(SqlTypes.SQLXML)` - maps to the SQL type XML

Any read or assignment (in an update statement) expression for an attribute of such an embeddable
will resolve to the proper SQL expression to access/update the attribute of the SQL type.

Since object, JSON and XML types are not supported equally on all databases, beware that not every mapping will work on all databases.
The following table outlines the current support for the different aggregate types:

| Database | Struct | JSON | XML |
| --- | --- | --- | --- |
| PostgreSQL | Yes | Yes | Yes |
| Oracle | Yes | Yes | Yes |
| MySQL/MariaDB | No | Yes | No |
| DB2 | Yes | Yes | Yes |
| SQL Server | No [(not yet)](https://hibernate.atlassian.net/browse/HHH-16158) | Yes | Yes |
| Sybase ASE | No | No | Yes |
| H2 | No | Yes | No |
| CockroachDB | No | Yes | No |
| SAP HANA | No | Yes | Yes |

Also note that embeddable types that are used in array aggregate mappings do not support association mappings.

##### `@Struct` aggregate embeddable mapping

The `@Struct` annotation can be placed on either the persistent attribute, or the embeddable type,
and requires the specification of a name i.e. the name of the SQL object type that it maps to.

The following example mapping, maps the `EmbeddableAggregate` type to the SQL object type `structType`:

Example 130. Mapping embeddable as SQL object type on persistent attribute level

```
@Entity(name = "StructHolder")
public static class StructHolder {

	@Id
	private Long id;
	@Struct(name = "structType")
	private EmbeddableAggregate aggregate;

}
```

The schema generation will by default emit DDL for that object type, which looks something along the lines of

```
create type structType as (
    ...
)
create table StructHolder as (
    id bigint not null primary key,
    aggregate structType
)
```

The name and the nullability of the column can be refined through applying a `@Column` on the persistent attribute.

One very important thing to note is that the order of columns in the DDL definition of a type must match the order that Hibernate expects.
By default, the order of columns is based on the alphabetical ordering of the embeddable type attribute names.

Consider the following class:

```
@Embeddable
@Struct(name = "myStruct")
public class MyStruct {
	@Column(name = "b")
	String attr1;
	@Column(name = "a")
	String attr2;
}
```

The expected ordering of columns will be `(b,a)`, because the name `attr1` comes before `attr2` in alphabetical ordering.
This example aims at showing the importance of the persistent attribute name.

Defining the embeddable type as Java record instead of a class can force a particular ordering through the definition of canonical constructor.

```
@Embeddable
@Struct(name = "myStruct")
public record MyStruct (
	@Column(name = "a")
	String attr2,
	@Column(name = "b")
	String attr1
) {}
```

In this particular example, the expected ordering of columns will be `(a,b)`, because the canonical constructor of the record
defines a specific ordering of persistent attributes, which Hibernate makes use of for `@Struct` mappings.

It is not necessary to switch to Java records to configure the order though.
The `@Struct` annotation allows specifying the order through the `attributes` member,
an array of attribute names that the embeddable type declares, which defines the order in columns appear in the SQL object type.

The same ordering as with the Java record can be achieved this way:

```
@Embeddable
@Struct(name = "myStruct", attributes = {"attr2", "attr1"})
public class MyStruct {
	@Column(name = "b")
	String attr1;
	@Column(name = "a")
	String attr2;
}
```

##### JSON/XML aggregate embeddable mapping

The `@JdbcTypeCode` annotation for JSON and XML mappings can only be placed on the persistent attribute.

The following example mapping, maps the `EmbeddableAggregate` type to the JSON SQL type:

Example 131. Mapping embeddable as JSON

```
@Entity(name = "JsonHolder")
public static class JsonHolder {

	@Id
	private Long id;
	@JdbcTypeCode(SqlTypes.JSON)
	private EmbeddableAggregate aggregate;

}
```

The schema generation will by default emit DDL that ensures the constraints of the embeddable type are respected, which looks something along the lines of

```
create table JsonHolder as (
    id bigint not null primary key,
    aggregate json,
    check (json_value(aggregate, '$.attribute1') is not null)
)
```

Again, the name and the nullability of the `aggregate` column can be refined through applying a `@Column` on the persistent attribute.

#### 3.3.13. Embeddable mappings containing collections

Mapping collections inside an `@Embeddable` value is supported in most cases. There are a couple exceptions:

- If the values of an @ElementCollection is of embeddable type, that embeddable cannot contain nested collections;
- Explicitly selecting an embeddable that contains collections in a query is currently not supported (we wouldn’t be able to correctly initialize the collection since its owning entity instance would be missing from the Persistence Context).
