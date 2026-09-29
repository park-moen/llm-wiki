# Jakarta Persistence 3.2 — IdClass Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/idclass
> Collected: 2026-09-29
> Published: Unknown

Module jakarta.persistence
Package jakarta.persistence

# Annotation Interface IdClass

@Target(TYPE) @Retention(RUNTIME) public @interface IdClass

Specifies a composite primary key type whose fields or properties map to the identifier fields or properties of the annotated entity class.

The specified primary key type must:

- be a non-abstract regular Java class, or a Java record type,
- have a public or protected constructor with no parameters, unless it is a record type, and
- implement equals(java.lang.Object) and hashCode(), defining value equality consistently with equality of the mapped primary key of the database table.

The primary key fields of the entity must be annotated Id, and the specified primary key type must have fields or properties with matching names and types. The mapping of fields or properties of the entity to fields or properties of the primary key class is implicit. The primary key type does not itself need to be annotated.

Example:

```java
@IdClass(EmployeePK.class)
@Entity
public class Employee {
    @Id
    String empName;
    @Id
    Date birthDay;
    ...
}

public record EmployeePK(String empName, Date birthDay) {}
```
