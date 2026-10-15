# Spring Data — Class PageRequest

> Source: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: API documentation main content; navigation omitted.

# Class PageRequest

[java.lang.Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html "class or interface in java.lang")

[org.springframework.data.domain.AbstractPageRequest](AbstractPageRequest.html "class in org.springframework.data.domain")

org.springframework.data.domain.PageRequest

All Implemented Interfaces:
:   `Serializable, Pageable`

---

public class PageRequest
extends [AbstractPageRequest](AbstractPageRequest.html "class in org.springframework.data.domain")

Basic Java Bean implementation of [`Pageable`](Pageable.html "interface in org.springframework.data.domain").

Author:
:   Oliver Gierke, Thomas Darimont, Anastasiia Smirnova, Mark Paluch, Thach Le

See Also:
:   - [Serialized Form](../../../../serialized-form.html#org.springframework.data.domain.PageRequest)

- ## Constructor Summary

  Constructors

  Modifier

  Constructor

  Description

  `protected`

  `PageRequest(int pageNumber,
  int pageSize,
  Sort sort)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort parameters applied.
- ## Method Summary

  All MethodsStatic MethodsInstance MethodsConcrete Methods

  Modifier and Type

  Method

  Description

  `boolean`

  `equals(Object o)`

  `PageRequest`

  `first()`

  Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the first page.

  `Sort`

  `getSort()`

  Returns the sorting parameters.

  `int`

  `hashCode()`

  `PageRequest`

  `next()`

  Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the next [`Page`](Page.html "interface in org.springframework.data.domain").

  `static PageRequest`

  `of(int pageNumber,
  int pageSize)`

  Creates a new unsorted [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

  `static PageRequest`

  `of(int pageNumber,
  int pageSize,
  Sort sort)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort parameters applied.

  `static PageRequest`

  `of(int pageNumber,
  int pageSize,
  Sort.Direction direction,
  String... properties)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort direction and properties applied.

  `static PageRequest`

  `ofSize(int pageSize)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") for the first page (page number `0`) given `pageSize` .

  `PageRequest`

  `previous()`

  Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the previous [`Page`](Page.html "interface in org.springframework.data.domain").

  `String`

  `toString()`

  `PageRequest`

  `withPage(int pageNumber)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with `pageNumber` applied.

  `PageRequest`

  `withSort(Sort sort)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with [`Sort`](Sort.html "class in org.springframework.data.domain") applied.

  `PageRequest`

  `withSort(Sort.Direction direction,
  String... properties)`

  Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with [`Sort.Direction`](Sort.Direction.html "enum class in org.springframework.data.domain") and `properties` applied.

  ### Methods inherited from class [AbstractPageRequest](AbstractPageRequest.html#method-summary "class in org.springframework.data.domain")

  `getOffset, getPageNumber, getPageSize, hasPrevious, previousOrFirst`

  ### Methods inherited from class [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#method-summary "class or interface in java.lang")

  `clone, finalize, getClass, notify, notifyAll, wait, wait, wait`

  ### Methods inherited from interface [Pageable](Pageable.html#method-summary "interface in org.springframework.data.domain")

  `getSortOr, isPaged, isUnpaged, toLimit, toOptional, toScrollPosition`

- ## Constructor Details

  - ### PageRequest

    protected PageRequest(int pageNumber,
    int pageSize,
    [Sort](Sort.html "class in org.springframework.data.domain") sort)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort parameters applied.

    Parameters:
    :   `pageNumber` - zero-based page number, must not be negative.
    :   `pageSize` - the size of the page to be returned, must be greater than 0.
    :   `sort` - must not be null, use [`Sort.unsorted()`](Sort.html#unsorted()) instead.
- ## Method Details

  - ### of

    public static [PageRequest](PageRequest.html "class in org.springframework.data.domain") of(int pageNumber,
    int pageSize)

    Creates a new unsorted [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

    Parameters:
    :   `pageNumber` - zero-based page number, must not be negative.
    :   `pageSize` - the size of the page to be returned, must be greater than 0.

    Since:
    :   2.0
  - ### of

    public static [PageRequest](PageRequest.html "class in org.springframework.data.domain") of(int pageNumber,
    int pageSize,
    [Sort](Sort.html "class in org.springframework.data.domain") sort)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort parameters applied.

    Parameters:
    :   `pageNumber` - zero-based page number, must not be negative.
    :   `pageSize` - the size of the page to be returned, must be greater than 0.
    :   `sort` - must not be null, use [`Sort.unsorted()`](Sort.html#unsorted()) instead.

    Since:
    :   2.0
  - ### of

    public static [PageRequest](PageRequest.html "class in org.springframework.data.domain") of(int pageNumber,
    int pageSize,
    [Sort.Direction](Sort.Direction.html "enum class in org.springframework.data.domain") direction,
    [String](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html "class or interface in java.lang")... properties)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with sort direction and properties applied.

    Parameters:
    :   `pageNumber` - zero-based page number, must not be negative.
    :   `pageSize` - the size of the page to be returned, must be greater than 0.
    :   `direction` - must not be null.
    :   `properties` - must not be null.

    Since:
    :   2.0
  - ### ofSize

    public static [PageRequest](PageRequest.html "class in org.springframework.data.domain") ofSize(int pageSize)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") for the first page (page number `0`) given `pageSize` .

    Parameters:
    :   `pageSize` - the size of the page to be returned, must be greater than 0.

    Returns:
    :   a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

    Since:
    :   2.5
  - ### getSort

    public [Sort](Sort.html "class in org.springframework.data.domain") getSort()

    Description copied from interface: `Pageable`

    Returns the sorting parameters.

    Returns:
    :   the sorting order.
  - ### next

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")(" -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") next()

    Description copied from interface: `Pageable`

    Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the next [`Page`](Page.html "interface in org.springframework.data.domain").

    Specified by:
    :   `next` in interface `Pageable`

    Specified by:
    :   `next` in class `AbstractPageRequest`

    Returns:
    :   the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the next [`Page`](Page.html "interface in org.springframework.data.domain").
  - ### previous

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")(" -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") previous()

    Description copied from class: `AbstractPageRequest`

    Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the previous [`Page`](Page.html "interface in org.springframework.data.domain").

    Specified by:
    :   `previous` in class `AbstractPageRequest`

    Returns:
    :   the previous [`Pageable`](Pageable.html "interface in org.springframework.data.domain")
  - ### first

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")(" -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") first()

    Description copied from interface: `Pageable`

    Returns the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the first page.

    Specified by:
    :   `first` in interface `Pageable`

    Specified by:
    :   `first` in class `AbstractPageRequest`

    Returns:
    :   the [`Pageable`](Pageable.html "interface in org.springframework.data.domain") requesting the first page.
  - ### withPage

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")("\_ -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") withPage(int pageNumber)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with `pageNumber` applied.

    Parameters:
    :   `pageNumber` - the page number to apply.

    Returns:
    :   a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

    Since:
    :   2.5
  - ### withSort

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")("\_, \_ -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") withSort([Sort.Direction](Sort.Direction.html "enum class in org.springframework.data.domain") direction,
    [String](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html "class or interface in java.lang")... properties)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with [`Sort.Direction`](Sort.Direction.html "enum class in org.springframework.data.domain") and `properties` applied.

    Parameters:
    :   `direction` - must not be null.
    :   `properties` - must not be null.

    Returns:
    :   a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

    Since:
    :   2.5
  - ### withSort

    [@Contract](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/Contract.html "class or interface in org.springframework.lang")("\_ -> new")
    [@CheckReturnValue](https://docs.spring.io/spring-framework/docs/7.0.8/javadoc-api/org/springframework/lang/CheckReturnValue.html "class or interface in org.springframework.lang")
    public [PageRequest](PageRequest.html "class in org.springframework.data.domain") withSort([Sort](Sort.html "class in org.springframework.data.domain") sort)

    Creates a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain") with [`Sort`](Sort.html "class in org.springframework.data.domain") applied.

    Parameters:
    :   `sort` - must not be null.

    Returns:
    :   a new [`PageRequest`](PageRequest.html "class in org.springframework.data.domain").

    Since:
    :   2.5
  - ### equals

    public boolean equals([Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html "class or interface in java.lang") o)

    Overrides:
    :   `equals` in class `AbstractPageRequest`
  - ### hashCode

    public int hashCode()

    Overrides:
    :   `hashCode` in class `AbstractPageRequest`
  - ### toString

    public [String](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html "class or interface in java.lang") toString()

    Overrides:
    :   `toString` in class `Object`

---

Copyright © 2011–2026 [Pivotal Software, Inc.](https://www.spring.io). All rights reserved.
