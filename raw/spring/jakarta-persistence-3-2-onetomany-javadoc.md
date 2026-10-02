# Jakarta Persistence 3.2 OneToMany Javadoc

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/onetomany
> Collected: 2026-10-02
> Published: Unknown

Specifies a many-valued association with one-to-many multiplicity.

If the collection is defined using generics to specify the element type, the associated target entity type need not be specified; otherwise the target entity class must be specified. If the relationship is bidirectional, the `mappedBy()` element must be used to specify the relationship field or property of the entity that is the owner of the relationship.

A `OneToMany` association usually maps a foreign key column or columns in the table of the associated entity. This mapping may be specified using the `JoinColumn` annotation. Alternatively, a unidirectional `OneToMany` association is sometimes mapped to a join table using the `JoinTable` annotation.

Example 1: One-to-Many association using generics

```java
// In Customer class:
@OneToMany(cascade = ALL, mappedBy = "customer")
public Set<Order> getOrders() { return orders; }

// In Order class:
@ManyToOne
@JoinColumn(name = "CUST_ID", nullable = false)
public Customer getCustomer() { return customer; }
```
