# Hibernate ORM 7.1 User Guide: Bidirectional @OneToOne lazy association

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/
> Collected: 2026-10-01
> Published: Unknown

## Bidirectional `@OneToOne` lazy association

Although you might annotate the parent-side association to be fetched lazily, Hibernate cannot honor this request since it cannot know whether the association is `null` or not.

The only way to figure out whether there is an associated record on the child side is to fetch the child association using a secondary query. Because this can lead to N+1 query issues, it’s much more efficient to use unidirectional `@OneToOne` associations with the `@MapsId` annotation in place.

However, if you really need to use a bidirectional association and want to make sure that this is always going to be fetched lazily, then you need to enable lazy state initialization bytecode enhancement.

## Associations

Therefore, the `@ManyToOne` and the `@OneToOne` child-side association are best to represent a `FOREIGN KEY` relationship.

The parent-side `@OneToOne` association requires bytecode enhancement so that the association can be loaded lazily. Otherwise, the parent-side association is always fetched even if the association is marked with `FetchType.LAZY`.

For this reason, it’s best to map `@OneToOne` association using `@MapsId` so that the `PRIMARY KEY` is shared between the child and the parent entities. When using `@MapsId`, the parent-side association becomes redundant since the child-entity can be easily fetched using the parent entity identifier.
