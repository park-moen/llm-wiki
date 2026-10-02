# Jakarta Persistence 3.2 Column Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/column
> Collected: 2026-10-02
> Published: Unknown

Specifies the column mapped by the annotated persistent property or field.

If no `Column` annotation is explicitly specified, the default values apply.

Example 1:

```java
@Column(name = "DESC", nullable = false, length = 512)
public String getDescription() { return description; }
```

`name`: (Optional) The name of the column. Defaults to the property or field name.
