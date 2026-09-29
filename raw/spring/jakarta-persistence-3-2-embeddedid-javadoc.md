# Jakarta Persistence 3.2 — EmbeddedId Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/embeddedid
> Collected: 2026-09-29
> Published: Unknown

Module jakarta.persistence
Package jakarta.persistence

# Annotation Interface EmbeddedId

@Target({METHOD,FIELD}) @Retention(RUNTIME) public @interface EmbeddedId

Specifies that the annotated persistent field or property of an entity class or mapped superclass is the composite primary key of the entity. The type of the annotated field or property must be an embeddable type, and must be explicitly annotated Embeddable.

If a field or property of an entity class is annotated EmbeddedId, then no other field or property of the entity may be annotated Id or EmbeddedId, and the entity class must not declare an IdClass.

The embedded primary key type must implement equals(java.lang.Object) and hashCode(), defining value equality consistently with equality of the mapped primary key of the database table.

The AttributeOverride annotation may be used to override the column mappings declared within the embeddable class.

The MapsId annotation may be used in conjunction with the EmbeddedId annotation to declare a derived primary key.

Relationship mappings defined within an embedded primary key type are not supported.

Example:

```java
@Entity
public class Employee {
    @EmbeddedId
    protected EmployeePK empPK;
    ...
}

public record EmployeePK(String empName, Date birthDay) {}
```
