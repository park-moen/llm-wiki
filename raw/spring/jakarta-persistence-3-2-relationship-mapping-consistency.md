# Jakarta Persistence 3.2: Entity Relationships and Mapping Defaults

> Source: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2
> Collected: 2026-10-01
> Published: Unknown

## 2.11. Entity Relationships

Such mapping annotations must be specified on the owning side of the relationship. Any overriding of mapping defaults must be consistent with the relationship modeling annotation that is specified. For example, if a many-to-one relationship mapping is specified, it is not permitted to specify a unique key constraint on the foreign key for the relationship.

## 2.12.3. Unidirectional Single-Valued Relationships

The unidirectional single-valued relationship modeling case can be specified as either a unidirectional `OneToOne` or as a unidirectional `ManyToOne` relationship.

### 2.12.3.1. Unidirectional OneToOne Relationships

Table `A` contains a foreign key to table `B`. The foreign key column name is formed as the concatenation of the name of the relationship property or field of entity A; " `_` "; the name of the primary key column in table `B`. The foreign key column has the same type as the primary key of `B` and there is a unique key constraint on it.
