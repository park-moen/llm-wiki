# Jakarta Persistence 3.2 Table Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/table
> Collected: 2026-10-02
> Published: Unknown

Specifies the primary table mapped by the annotated entity type.

Additional tables may be specified using `SecondaryTable` or `SecondaryTables` annotation.

If no `Table` annotation is specified for an entity class, the default values apply.

Example:

```java
@Entity
@Table(name = "CUST", schema = "RECORDS")
public class Customer { ... }
```

`name`: (Optional) The name of the table.

`schema`: (Optional) The schema of the table.
