# Jakarta Persistence 3.2: JoinColumn

> Source: https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/joincolumn
> Collected: 2026-10-01
> Published: Unknown

## name

(Optional) The name of the foreign key column.

If the join is for a `OneToOne` or `ManyToOne` mapping using a foreign key mapping strategy, the foreign key column is in the table of the source entity or embeddable.

## unique

(Optional) Whether the property is a unique key. This is a shortcut for the `UniqueConstraint` annotation at the table level and is useful for when the unique key constraint is only a single field.

Default: `false`

## nullable

(Optional) Whether the foreign key column is nullable.

Default: `true`
