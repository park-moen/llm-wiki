# Hibernate ORM 7.1 User Guide: Bidirectional OneToMany

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/
> Collected: 2026-10-02
> Published: Unknown

The bidirectional `@OneToMany` association also requires a `@ManyToOne` association on the child side. Although the Domain Model exposes two sides to navigate this association, behind the scenes, the relational database has only one foreign key for this relationship.

Every bidirectional association must have one owning side only (the child side), the other one being referred to as the inverse (or the `mappedBy`) side.

Example 205. `@OneToMany` association mappedBy the `@ManyToOne` side

```java
@Entity(name = "Person")
public static class Person {
    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "person", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Phone> phones = new ArrayList<>();
}
```
