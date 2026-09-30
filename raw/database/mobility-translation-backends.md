# Mobility: Backends

> Source: https://github.com/shioyama/mobility
> Collected: 2026-09-30
> Published: Unknown

## Backends

Mobility supports different storage strategies, called "backends". The default backend is the `KeyValue` backend, which stores translations in two tables, by default named `mobility_text_translations` and `mobility_string_translations`.

### Table Backend (like Globalize)

The `Table` backend stores translations as columns on a model-specific table. If your model uses the table `posts`, then by default this backend will store an attribute `title` on a table `post_translations`, and join the table to retrieve the translated value.

### Column Backend (like Traco)

The `Column` backend stores translations as columns with locale suffixes on the model table. For an attribute `title`, these would be of the form `title_en`, `title_fr`, etc.

### PostgreSQL-specific Backends

Mobility also supports JSON and Hstore storage options, if you are using PostgreSQL as your database. To use this option, create column(s) on the model table for each translated attribute, and set your backend to `:json`, `:jsonb` or `:hstore`.

Another option is to store all your translations on a single jsonb column (one per model). This is called the "container" backend.
