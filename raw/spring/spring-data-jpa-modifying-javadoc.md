# Spring Data — Annotation Interface Modifying

> Source: https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Modifying.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: API documentation main content; navigation omitted.

# Annotation Interface Modifying

---

[@Retention](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/Retention.html "class or interface in java.lang.annotation")([RUNTIME](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/RetentionPolicy.html#RUNTIME "class or interface in java.lang.annotation"))
[@Target](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/Target.html "class or interface in java.lang.annotation")({[METHOD](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/ElementType.html#METHOD "class or interface in java.lang.annotation"),[ANNOTATION\_TYPE](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/ElementType.html#ANNOTATION_TYPE "class or interface in java.lang.annotation")})
[@Documented](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/Documented.html "class or interface in java.lang.annotation")
public @interface Modifying

Indicates a query method should be considered as modifying query as that changes the way it needs to be executed.
This annotation is only considered if used on query methods defined through a [`Query`](Query.html "annotation interface in org.springframework.data.jpa.repository") annotation. It's not
applied on custom implementation methods or queries derived from the method name as they already have control over
the underlying data access APIs or specify if they are modifying by their name.

Queries that require a `@Modifying` annotation include `INSERT`, `UPDATE`, `DELETE`, and DDL
statements.

Author:
:   Oliver Gierke, Christoph Strobl, Nicolas Cirigliano, Jens Schauder

See Also:
:   - [`Query`](Query.html "annotation interface in org.springframework.data.jpa.repository")

- ## Optional Element Summary

  Optional Elements

  Modifier and Type

  Optional Element

  Description

  `boolean`

  `clearAutomatically`

  Defines whether we should clear the underlying persistence context after executing the modifying query.

  `boolean`

  `flushAutomatically`

  Defines whether we should flush the underlying persistence context before executing the modifying query.

- ## Element Details

  - ### flushAutomatically

    boolean flushAutomatically

    Defines whether we should flush the underlying persistence context before executing the modifying query.

    Returns:

    Default:
    :   `false`
  - ### clearAutomatically

    boolean clearAutomatically

    Defines whether we should clear the underlying persistence context after executing the modifying query.

    Returns:

    Default:
    :   `false`

---

Copyright © 2011–2026 [Pivotal Software, Inc.](https://www.spring.io). All rights reserved.
