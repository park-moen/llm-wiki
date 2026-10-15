# Hibernate ORM — Fetching

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/#fetching
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named section with subsections; version 7.1.36.Final.

## 12. Fetching

Fetching, essentially, is the process of grabbing data from the database and making it available to the application.
Tuning how an application does fetching is one of the biggest factors in determining how an application will perform.
Fetching too much data, in terms of width (values/columns) and/or depth (results/rows),
adds unnecessary overhead in terms of both JDBC communication and ResultSet processing.
Fetching too little data might cause additional fetching to be needed.
Tuning how an application fetches data presents a great opportunity to influence the overall application performance.

### 12.1. The basics

The concept of fetching breaks down into two different questions.

- When should the data be fetched? Now? Later?
- How should the data be fetched?

|  |  |
| --- | --- |
|  | "Now" is generally termed eager or immediate while "later" is generally termed lazy or delayed. |

There are a number of scopes for defining fetching:

*static*
:   Static definition of fetching strategies is done in the mappings.
    The statically-defined fetch strategies are used in the absence of any dynamically defined strategies.

    SELECT
    :   Performs a separate SQL select to load the data. This can either be EAGER (the second select is issued immediately) or LAZY (the second select is delayed until the data is needed).
        This is the strategy generally termed N+1.

    JOIN
    :   Inherently an EAGER style of fetching. The data to be fetched is obtained through the use of an SQL outer join.

    BATCH
    :   Performs a separate SQL select to load a number of related data items using an IN-restriction as part of the SQL WHERE-clause based on a batch size.
        Again, this can either be EAGER (the second select is issued immediately) or LAZY (the second select is delayed until the data is needed).

    SUBSELECT
    :   Performs a separate SQL select to load associated data based on the SQL restriction used to load the owner.
        Again, this can either be EAGER (the second select is issued immediately) or LAZY (the second select is delayed until the data is needed).

*dynamic* (sometimes referred to as runtime)
:   The dynamic definition is really use-case centric. There are multiple ways to define dynamic fetching:

    fetch profiles
    :   defined in mappings, but can be enabled/disabled on the `Session`.

    HQL / JPQL
    :   both Hibernate and Jakarta Persistence Criteria queries have the ability to specify fetching, specific to said query.

    entity graphs
    :   using Jakarta Persistence EntityGraphs

|  |  |
| --- | --- |
|  | `fetch profile` and `entity graph` are mutually exclusive. When both are present, `entity graph` would take effect and `fetch profile` would be ignored. |

### 12.2. Direct fetching vs. entity queries

To see the difference between direct fetching and entity queries in regard to eagerly fetched associations, consider the following entities:

Example 461. Domain model

```
@Entity(name = "Department")
public static class Department {

	@Id
	private Long id;

	//Getters and setters omitted for brevity
}

@Entity(name = "Employee")
public static class Employee {

	@Id
	private Long id;

	@NaturalId
	private String username;

	@ManyToOne(fetch = FetchType.EAGER)
	private Department department;

	//Getters and setters omitted for brevity
}
```

The `Employee` entity has a `@ManyToOne` association to a `Department` which is fetched eagerly.

When issuing a direct entity fetch, Hibernate executed the following SQL query:

Example 462. Direct fetching example

```
Employee employee = entityManager.find(Employee.class, 1L);
```

```
select
    e.id as id1_1_0_,
    e.department_id as departme3_1_0_,
    e.username as username2_1_0_,
    d.id as id1_0_1_ 
from
    Employee e 
left outer join
    Department d 
        on e.department_id=d.id 
where
    e.id = 1
```

The `LEFT OUTER JOIN` clause is added to the generated SQL query because this association is required to be fetched eagerly.

On the other hand, if you are using an entity query that does not contain a `JOIN FETCH` directive to the `Department` association:

Example 463. Entity query fetching example

```
Employee employee = entityManager.createQuery(
		"select e " +
		"from Employee e " +
		"where e.id = :id", Employee.class)
.setParameter("id", 1L)
.getSingleResult();
```

```
select
    e.id as id1_1_,
    e.department_id as departme3_1_,
    e.username as username2_1_ 
from
    Employee e 
where
    e.id = 1

select
    d.id as id1_0_0_
from
    Department d
where
    d.id = 1
```

Hibernate uses a secondary select instead. This is because the entity query fetch policy cannot be overridden,
so Hibernate requires a secondary select to ensure that the EAGER association is fetched prior to returning the result to the user.

|  |  |
| --- | --- |
|  | If you forget to JOIN FETCH all EAGER associations, Hibernate is going to issue a secondary select for each and every one of those which, in turn, can lead to N + 1 query issue.  For this reason, you should prefer LAZY associations. |

### 12.3. Applying fetch strategies

Let’s consider these topics as it relates to a sample domain model and a few use cases.

Example 464. Sample domain model

```
@Entity(name = "Department")
public static class Department {

	@Id
	private Long id;

	@OneToMany(mappedBy = "department")
	private List<Employee> employees = new ArrayList<>();

	//Getters and setters omitted for brevity
}

@Entity(name = "Employee")
public static class Employee {

	@Id
	private Long id;

	@NaturalId
	private String username;

	@Column(name = "pswd", columnDefinition = "varbinary")
	@ColumnTransformer(
		read = "trim(trailing u&'\\0000' from cast(decrypt('AES', '00', pswd ) as character varying))",
		write = "encrypt('AES', '00', ?)"
	)
	private String password;

	private int accessLevel;

	@ManyToOne(fetch = FetchType.LAZY)
	private Department department;

	@ManyToMany(mappedBy = "employees")
	private List<Project> projects = new ArrayList<>();

	//Getters and setters omitted for brevity
}

@Entity(name = "Project")
public class Project {

	@Id
	private Long id;

	@ManyToMany
	private List<Employee> employees = new ArrayList<>();

	//Getters and setters omitted for brevity
}
```

|  |  |
| --- | --- |
|  | The Hibernate recommendation is to statically mark all associations lazy and to use dynamic fetching strategies for eagerness.  This is unfortunately at odds with the Jakarta Persistence specification which defines that all one-to-one and many-to-one associations should be eagerly fetched by default.  Hibernate, as a Jakarta Persistence provider, honors that default. |

### 12.4. No fetching

For the first use case, consider the application login process for an `Employee`.
Let’s assume that login only requires access to the `Employee` information, not `Project` nor `Department` information.

Example 465. No fetching example

```
Employee employee = entityManager.createQuery(
	"select e " +
	"from Employee e " +
	"where " +
	"	e.username = :username and " +
	"	e.password = :password",
	Employee.class)
.setParameter("username", username)
.setParameter("password", password)
.getSingleResult();
```

In this example, the application gets the `Employee` data.
However, because all associations from `Employee` are declared as LAZY (Jakarta Persistence defines the default for collections as LAZY) no other data is fetched.

If the login process does not need access to the `Employee` information specifically, another fetching optimization here would be to limit the width of the query results.

Example 466. No fetching (scalar) example

```
Integer accessLevel = entityManager.createQuery(
	"select e.accessLevel " +
	"from Employee e " +
	"where " +
	"	e.username = :username and " +
	"	e.password = :password",
	Integer.class)
.setParameter("username", username)
.setParameter("password", password)
.getSingleResult();
```

### 12.5. Dynamic fetching via queries

For the second use case, consider a screen displaying the `Projects` for an `Employee`.
Certainly access to the `Employee` is needed, as is the collection of `Projects` for that Employee. Information about `Departments`, other `Employees` or other `Projects` is not needed.

Example 467. Dynamic JPQL fetching example

```
Employee employee = entityManager.createQuery(
	"select e " +
	"from Employee e " +
	"left join fetch e.projects " +
	"where " +
	"	e.username = :username and " +
	"	e.password = :password",
	Employee.class)
.setParameter("username", username)
.setParameter("password", password)
.getSingleResult();
```

Example 468. Dynamic query fetching example

```
CriteriaBuilder builder = entityManager.getCriteriaBuilder();
CriteriaQuery<Employee> query = builder.createQuery(Employee.class);
Root<Employee> root = query.from(Employee.class);
root.fetch("projects", JoinType.LEFT);
query.select(root).where(
	builder.and(
		builder.equal(root.get("username"), username),
		builder.equal(root.get("password"), password)
	)
);
Employee employee = entityManager.createQuery(query).getSingleResult();
```

In this example we have an `Employee` and their `Projects` loaded in a single query shown both as an HQL query and a Jakarta Persistence Criteria query.
In both cases, this resolves to exactly one database query to get all that information.

### 12.6. Dynamic fetching via Jakarta Persistence entity graph

Jakarta Persistence also supports a feature called EntityGraphs to provide the application developer has more control over fetch plans. It has two modes to choose from:

fetch graph
:   In this case, all attributes specified in the entity graph will be treated as FetchType.EAGER, and all attributes not specified will **ALWAYS** be treated as FetchType.LAZY.

load graph
:   In this case, all attributes specified in the entity graph will be treated as FetchType.EAGER, but attributes not specified use their static mapping specification.

Below is a `fetch graph` dynamic fetching example:

Example 469. Fetch graph example

```
@Entity(name = "Employee")
@NamedEntityGraph(name = "employee.projects",
	attributeNodes = @NamedAttributeNode("projects")
)
```

```
Employee employee = entityManager.find(
	Employee.class,
	userId,
	Collections.singletonMap(
		"jakarta.persistence.fetchgraph",
		entityManager.getEntityGraph("employee.projects")
	)
);
```

|  |  |
| --- | --- |
|  | When executing a JPQL query, if an EAGER association is omitted, Hibernate will issue a secondary select for every association needed to be fetched eagerly, which can lead to N+1 query issues.  For this reason, it’s better to use LAZY associations, and only fetch them eagerly on a per-query basis. |

An EntityGraph is the root of a "load plan" and must correspond to an EntityType.

#### 12.6.1. Jakarta Persistence (key) subgraphs

A subgraph is used to control the fetching of sub-attributes of the AttributeNode it is applied to.
It is generally defined via the [`@NamedSubgraph`](https://javaee.github.io/javaee-spec/javadocs/javax/persistence/NamedSubgraph.html) annotation.

If we have a `Project` parent entity which has an `employees` child associations,
and we’d like to fetch the `department` for the `Employee` child association.

Example 470. Fetch graph with a subgraph mapping

```
@Entity(name = "Project")
@NamedEntityGraph(name = "project.employees",
	attributeNodes = @NamedAttributeNode(
		value = "employees",
		subgraph = "project.employees.department"
	),
	subgraphs = @NamedSubgraph(
		name = "project.employees.department",
		attributeNodes = @NamedAttributeNode("department")
	)
)
public static class Project {

	@Id
	private Long id;

	@ManyToMany
	private List<Employee> employees = new ArrayList<>();

	//Getters and setters omitted for brevity
}
```

When fetching this entity graph, Hibernate generates the following SQL query:

Example 471. Fetch graph with a subgraph mapping

```
Project project = doInJPA(this::entityManagerFactory, entityManager -> {
	return entityManager.find(
		Project.class,
		1L,
		Collections.singletonMap(
			"jakarta.persistence.fetchgraph",
			entityManager.getEntityGraph("project.employees")
		)
	);
});
```

```
select
    p.id as id1_2_0_, e.id as id1_1_1_, d.id as id1_0_2_,
    e.accessLevel as accessLe2_1_1_,
    e.department_id as departme5_1_1_,
    decrypt( 'AES', '00', e.pswd  ) as pswd3_1_1_,
    e.username as username4_1_1_,
    p_e.projects_id as projects1_3_0__,
    p_e.employees_id as employee2_3_0__
from
    Project p
inner join
    Project_Employee p_e
        on p.id=p_e.projects_id
inner join
    Employee e
        on p_e.employees_id=e.id
inner join
    Department d
        on e.department_id=d.id
where
    p.id = ?

-- binding parameter [1] as [BIGINT] - [1]
```

Specifying a subgraph is only valid for an attribute (or its "key") whose type is a ManagedType.
So while an EntityGraph must correspond to an EntityType, a Subgraph is legal for any ManagedType.

If the attribute is a Map and the Map’s key is a ManagedType, the "key subgraph" is applied to the Map’s key type.
In this case, the non-key subgraph applies to the Map value/element.

#### 12.6.2. Jakarta Persistence SubGraph sub-typing

Subgraphs can also be subtype specific. Given an attribute whose value is an inheritance hierarchy,
we can refer to attributes of a specific subtype using the forms of subgraph definition that accept
the subtype Class.

#### 12.6.3. Creating and applying Jakarta Persistence graphs from text representations

Hibernate allows the creation of Jakarta Persistence fetch/load graphs by parsing a textual representation
of the graph. Generally speaking, the textual representation of a graph is a comma-separated
list of attribute names, optionally including any subgraph specifications.
The starting point for such parsing operations is either `org.hibernate.graph.GraphParser`
or `SessionFactory#parseEntityGraph`

|  |  |
| --- | --- |
|  | Parsing a textual representation of a graph is not (yet) a part of the Jakarta Persistence specification. So the syntax described here is specific to Hibernate. We do hope to eventually make this syntax part of the Jakarta Persistence specification proper. |

Example 472. Parsing a simple graph

```
final EntityGraph<Project> graph = GraphParser.parse(
		Project.class,
		"employees(department)",
		entityManager
);
```

This example actually functions exactly as Fetch graph with a subgraph mapping,
just using a parsed graph rather than a named graph.

The syntax also supports defining "key subgraphs". To specify a key subgraph, `.key` is added
to the end of the attribute name.

Example 473. Parsing a map key graph

```
final EntityGraph<Ticket> graph = GraphParser.parse(
		Ticket.class,
		"showing(id(movie(cast)))",
		entityManager
);
```

Parsing can also handle subtype specific subgraphs. For example, given an entity hierarchy of
`LegalEntity` ← (`Corporation` | `Person` | `NonProfit`) and an attribute named `responsibleParty` whose
type is the `LegalEntity` base type we might have:

```
responsibleParty(Corporation: ceo)
```

We can even duplicate the attribute names to apply different subtype subgraphs:

```
responsibleParty(taxIdNumber), responsibleParty(Corporation: ceo), responsibleParty(NonProfit: sector)
```

The duplicated attribute names are handled according to the Jakarta Persistence specification which says that duplicate
specification of the attribute node results in the originally registered AttributeNode to be re-used
effectively merging the 2 AttributeNode specifications together. In other words, the above specification
creates a single AttributeNode with 3 distinct SubGraphs. It is functionally the same as calling:

```
Class<Invoice> invoiceClass = ...;
jakarta.persistence.EntityGraph<Invoice> invoiceGraph = entityManager.createEntityGraph( invoiceClass );
invoiceGraph.addAttributeNode( "responsibleParty" );
invoiceGraph.addSubgraph( "responsibleParty" ).addAttributeNode( "taxIdNumber" );
invoiceGraph.addSubgraph( "responsibleParty", Corporation.class ).addAttributeNode( "ceo" );
invoiceGraph.addSubgraph( "responsibleParty", NonProfit.class ).addAttributeNode( "sector" );
```

#### 12.6.4. Combining multiple Jakarta Persistence entity graphs into one

Multiple entity graphs can be combined into a single "super graph" that acts as a union. Graph from
the previous example can also be built by combining separate aspect graphs into one, such as:

Example 474. Combining multiple graphs into one

```
final EntityGraph<Project> a = GraphParser.parse(
		Project.class, "employees(username)", entityManager
);

final EntityGraph<Project> b = GraphParser.parse(
		Project.class, "employees(password, accessLevel)", entityManager
);

final EntityGraph<Project> c = GraphParser.parse(
		Project.class, "employees(department(employees(username)))", entityManager
);

final EntityGraph<Project> all = EntityGraphs.merge(entityManager, Project.class, a, b, c);
```

#### 12.6.5. @NamedEntityGraph with text representation

Hibernate also offers a `@org.hibernate.annotations.NamedEntityGraph` annotation, as a corollary
to the `@jakarta.persistence.NamedEntityGraph`, supporting the text representation
discussed previously. The annotation
may be placed on an entity or on a package.

Example 475. @NamedEntityGraph example

```
@Entity
@NamedEntityGraph( graph="title,isbn,author(name,phoneNumber)" )
class Book {
	// ...
}
```

### 12.7. Dynamic fetching via Hibernate profiles

Suppose we wanted to leverage loading by natural-id to obtain the `Employee` information in the "projects for and employee" use-case.
Loading by natural-id uses the statically defined fetching strategies, but does not expose a means to define load-specific fetching.
So we would leverage a fetch profile.

Example 476. Fetch profile example

```
@Entity(name = "Employee")
@FetchProfile(
	name = "employee.projects",
	fetchOverrides = {
		@FetchProfile.FetchOverride(
			entity = Employee.class,
			association = "projects",
			mode = FetchMode.JOIN
		)
	}
)
```

```
session.enableFetchProfile("employee.projects");
Employee employee = session.bySimpleNaturalId(Employee.class).load(username);
```

Here the `Employee` is obtained by natural-id lookup and the Employee’s `Project` data is fetched eagerly.
If the `Employee` data is resolved from cache, the `Project` data is resolved on its own.
However, if the `Employee` data is not resolved in cache, the `Employee` and `Project` data is resolved in one SQL query via join as we saw above.

### 12.8. Batch fetching

Hibernate offers the [`@BatchSize`](https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/BatchSize.html) annotation,
which can be used when fetching uninitialized entity proxies.

Considering the following entity mapping:

Example 477. `@BatchSize` mapping example

```
@Entity(name = "Department")
public static class Department {

	@Id
	private Long id;

	@OneToMany(mappedBy = "department")
	@BatchSize(size = 5)
	private List<Employee> employees = new ArrayList<>();

	//Getters and setters omitted for brevity

}

@Entity(name = "Employee")
public static class Employee {

	@Id
	private Long id;

	@NaturalId
	private String name;

	@ManyToOne(fetch = FetchType.LAZY)
	private Department department;

	//Getters and setters omitted for brevity
}
```

Considering that we have previously fetched several `Department` entities,
and now we need to initialize the `employees` entity collection for each particular `Department`,
the `@BatchSize` annotations allows us to load multiple `Employee` entities in a single database round trip.

Example 478. `@BatchSize` fetching example

```
List<Department> departments = entityManager.createQuery(
	"select d " +
	"from Department d " +
	"inner join d.employees e " +
	"where e.name like 'John%'", Department.class)
.getResultList();

for (Department department : departments) {
	log.infof(
		"Department %d has {} employees",
		department.getId(),
		department.getEmployees().size()
	);
}
```

```
SELECT
    d.id as id1_0_
FROM
    Department d
INNER JOIN
    Employee employees1_
    ON d.id=employees1_.department_id

SELECT
    e.department_id as departme3_1_1_,
    e.id as id1_1_1_,
    e.id as id1_1_0_,
    e.department_id as departme3_1_0_,
    e.name as name2_1_0_
FROM
    Employee e
WHERE
    e.department_id IN (
        0, 2, 3, 4, 5
    )

SELECT
    e.department_id as departme3_1_1_,
    e.id as id1_1_1_,
    e.id as id1_1_0_,
    e.department_id as departme3_1_0_,
    e.name as name2_1_0_
FROM
    Employee e
WHERE
    e.department_id IN (
        6, 7, 8, 9, 1
    )
```

As you can see in the example above, there are only two SQL statements used to fetch the `Employee` entities associated with multiple `Department` entities.

|  |  |
| --- | --- |
|  | Without `@BatchSize`, you’d run into a N + 1 query issue, so, instead of 2 SQL statements, there would be 10 queries needed for fetching the `Employee` child entities.  However, although `@BatchSize` is better than running into an N + 1 query issue, most of the time, a DTO projection or a `JOIN FETCH` is a much better alternative since it allows you to fetch all the required data with a single query. |

|  |  |
| --- | --- |
|  | When `LockModeType` is different from `NONE` Hibernate will not execute a batch fetching so uninitialized entity proxies will not be initialized.  This because the lock mode is different from the one of the proxies in the batch fetch queue. |

### 12.9. The `@Fetch` annotation mapping

Besides the `FetchType.LAZY` or `FetchType.EAGER` Jakarta Persistence annotations,
you can also use the Hibernate-specific `@Fetch` annotation that accepts one of the following `FetchMode`s:

SELECT
:   The association is going to be fetched using a secondary select for each individual entity,
    collection, or join load. This mode can be used for either `FetchType.EAGER` or `FetchType.LAZY`.

JOIN
:   Use an outer join to load the related entities, collections or joins when using direct fetching. This mode
    can only be used for `FetchType.EAGER`.

SUBSELECT
:   Available for collections only. When accessing a non-initialized collection,
    this fetch mode will trigger loading all elements of all collections of the same role for all owners associated
    with the persistence context using a single secondary select.

### 12.10. `FetchMode.SELECT`

To demonstrate how `FetchMode.SELECT` works, consider the following entity mapping:

Example 479. `FetchMode.SELECT` mapping example

```
@Entity(name = "Department")
public static class Department {

	@Id
	private Long id;

	@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
	@Fetch(FetchMode.SELECT)
	private List<Employee> employees = new ArrayList<>();

	//Getters and setters omitted for brevity

}

@Entity(name = "Employee")
public static class Employee {

	@Id
	@GeneratedValue
	private Long id;

	@NaturalId
	private String username;

	@ManyToOne(fetch = FetchType.LAZY)
	private Department department;

	//Getters and setters omitted for brevity

}
```

Considering there are multiple `Department` entities, each one having multiple `Employee` entities,
when executing the following test case, Hibernate fetches every uninitialized `Employee`
collection using a secondary `SELECT` statement upon accessing the child collection for the first time:

Example 480. `FetchMode.SELECT` mapping example

```
List<Department> departments = entityManager.createQuery(
	"select d from Department d", Department.class)
.getResultList();

log.infof("Fetched %d Departments", departments.size());

for (Department department : departments) {
	assertEquals(3, department.getEmployees().size());
}
```

```
SELECT
    d.id as id1_0_
FROM
    Department d

-- Fetched 2 Departments

SELECT
    e.department_id as departme3_1_0_,
    e.id as id1_1_0_,
    e.id as id1_1_1_,
    e.department_id as departme3_1_1_,
    e.username as username2_1_1_
FROM
    Employee e
WHERE
    e.department_id = 1

SELECT
    e.department_id as departme3_1_0_,
    e.id as id1_1_0_,
    e.id as id1_1_1_,
    e.department_id as departme3_1_1_,
    e.username as username2_1_1_
FROM
    Employee e
WHERE
    e.department_id = 2
```

The more `Department` entities are fetched by the first query, the more secondary `SELECT` statements are executed to initialize the `employees` collections.
Therefore, `FetchMode.SELECT` can lead to N + 1 query issue.

### 12.11. `FetchMode.SUBSELECT`

To demonstrate how `FetchMode.SUBSELECT` works, we are going to modify the `FetchMode.SELECT` mapping example to use
`FetchMode.SUBSELECT`:

Example 481. `FetchMode.SUBSELECT` mapping example

```
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<Employee> employees = new ArrayList<>();
```

Now, we are going to fetch all `Department` entities that match a given filtering predicate
and then navigate their `employees` collections.

Hibernate is going to avoid the N + 1 query issue by generating a single SQL statement to initialize all `employees` collections
for all `Department` entities that were previously fetched.
Instead of passing all entity identifiers, Hibernate simply reruns the previous query that fetched the `Department` entities.

Example 482. `FetchMode.SUBSELECT` mapping example

```
List<Department> departments = entityManager.createQuery(
	"select d " +
	"from Department d " +
	"where d.name like :token", Department.class)
.setParameter("token", "Department%")
.getResultList();

log.infof("Fetched %d Departments", departments.size());

for (Department department : departments) {
	assertEquals(3, department.getEmployees().size());
}
```

```
SELECT
    d.id as id1_0_
FROM
    Department d
where
    d.name like 'Department%'
    
-- Fetched 2 Departments

SELECT
    e.department_id as departme3_1_1_,
    e.id as id1_1_1_,
    e.id as id1_1_0_,
    e.department_id as departme3_1_0_,
    e.username as username2_1_0_
FROM
    Employee e
WHERE
    e.department_id in (
        SELECT
            fetchmodes0_.id
        FROM
            Department fetchmodes0_
        WHERE
            d.name like 'Department%'
    )
```

### 12.12. `FetchMode.JOIN`

To demonstrate how `FetchMode.JOIN` works, we are going to modify the `FetchMode.SELECT` mapping example to use
`FetchMode.JOIN` instead:

Example 483. `FetchMode.JOIN` mapping example

```
@OneToMany(mappedBy = "department")
@Fetch(FetchMode.JOIN)
private List<Employee> employees = new ArrayList<>();
```

Now, we are going to fetch one `Department` and navigate its `employees` collections.

|  |  |
| --- | --- |
|  | The reason why we are not using a JPQL query to fetch multiple `Department` entities is because the `FetchMode.JOIN` strategy would be overridden by the query fetching directive.  To fetch multiple relationships with a JPQL query, the `JOIN FETCH` directive must be used instead.  Therefore, `FetchMode.JOIN` is useful for when entities are fetched directly, via their identifier or natural-id.  Also, the `FetchMode.JOIN` acts as a `FetchType.EAGER` strategy. Even if we mark the association as `FetchType.LAZY`, the `FetchMode.JOIN` will load the association eagerly. |

Hibernate is going to avoid the secondary query by issuing an OUTER JOIN for the `employees` collection.

Example 484. `FetchMode.JOIN` mapping example

```
Department department = entityManager.find(Department.class, 1L);

log.infof("Fetched department: %s", department.getId());

assertEquals(3, department.getEmployees().size());
```

```
SELECT
    d.id as id1_0_0_,
    e.department_id as departme3_1_1_,
    e.id as id1_1_1_,
    e.id as id1_1_2_,
    e.department_id as departme3_1_2_,
    e.username as username2_1_2_
FROM
    Department d
LEFT OUTER JOIN
    Employee e
        on d.id = e.department_id
WHERE
    d.id = 1

-- Fetched department: 1
```

This time, there was no secondary query because the child collection was loaded along with the parent entity.
