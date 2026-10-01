# Jakarta Persistence 3.2: OneToOne

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/onetoone
> Collected: 2026-10-01
> Published: Unknown

## Annotation Interface OneToOne

Specifies a single-valued association to another entity class that has one-to-one multiplicity.

## fetch

(Optional) Whether the association should be lazily loaded or must be eagerly fetched.

The `EAGER` strategy is a requirement on the persistence provider runtime that the associated entity must be eagerly fetched.

The `LAZY` strategy is a hint to the persistence provider runtime.

If not specified, defaults to `EAGER`.

## mappedBy

(Optional) The field that owns the relationship. This element is only specified on the inverse (non-owning) side of the association.
