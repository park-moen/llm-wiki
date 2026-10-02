# Hibernate ORM User Guide: Long Conversations and Concurrency

> Source: https://docs.hibernate.org/orm/7.1/userguide/html_single/
> Collected: 2026-10-02
> Published: 2026-09-17

## 9.5.3. Conversations

The first screen of a dialog opens. The data seen by the user is loaded in a particular `Session` and database transaction. The user is free to modify the objects.

The user uses a UI element to save their work after five minutes of editing. The modifications are made persistent. The user also expects to have exclusive access to the data during the edit session.

A first naive implementation might keep the `Session` and database transaction open while the user is editing, using database-level locks to prevent other users from modifying the same data and to guarantee isolation and atomicity. This is an anti-pattern because lock contention is a bottleneck which will prevent scalability in the future.

Several database transactions are used to implement the conversation. In this case, maintaining isolation of business processes becomes the partial responsibility of the application tier. A single conversation usually spans several database transactions. These multiple database accesses can only be atomic as a whole if only one of these database transactions (typically the last one) stores the updated data. All others only read data.

A common way to receive this data is through a wizard-style dialog spanning several request/response cycles. Hibernate includes some features which make this easy to implement.

Automatic Versioning  | Hibernate can perform automatic optimistic concurrency control for you. It can automatically detect (at the end of the conversation) if a concurrent modification occurred during user think time.

Detached Objects  | If you decide to use the session-per-request pattern, all loaded instances will be in the detached state during user think time. Hibernate allows you to reattach the objects and persist the modifications. The pattern is called session-per-request-with-detached-objects. Automatic versioning is used to isolate concurrent modifications.
