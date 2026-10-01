# Jakarta Persistence 3.2: ManyToOne

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/manytoone
> Collected: 2026-10-01
> Published: Unknown

## Annotation Interface ManyToOne

Specifies a single-valued association to another entity class that has many-to-one multiplicity.

A `ManyToOne` association usually maps a foreign key column or columns. This mapping may be specified using the `JoinColumn` annotation.

## fetch

(Optional) Whether the association should be lazily loaded or must be eagerly fetched.

The `EAGER` strategy is a requirement on the persistence provider runtime that the associated entity must be eagerly fetched.

The `LAZY` strategy is a hint to the persistence provider runtime.

If not specified, defaults to `EAGER`.
