# Hibernate ORM 7.1 SoftDelete Javadoc

> Source: https://docs.hibernate.org/orm/7.1/javadocs/org/hibernate/annotations/SoftDelete.html
> Collected: 2026-10-02
> Published: Unknown

Describes a soft-delete indicator mapping.

Soft deletes handle "deletions" from a database table by setting a column in the table to indicate deletion.

May be defined at various levels:

* PACKAGE, where it applies to all mappings defined in the package, unless defined more specifically.
* TYPE, where it applies to an entity hierarchy. The annotation must be defined on the root of the hierarchy and affects to the hierarchy as a whole.
* FIELD / METHOD, where it applies to the rows of an `ElementCollection` or `ManyToMany` table.

Since: 6.4
