# Kotlin Visibility modifiers: Constructors

> Source: https://kotlinlang.org/docs/visibility-modifiers.html
> Collected: 2026-10-02
> Published: 2025-11-13

## Class members

For members declared inside a class:

* `private` means that the member is visible inside this class only (including all its members).
* `protected` means that the member has the same visibility as one marked as `private`, but that it is also visible in subclasses.
* `internal` means that any client inside this module who sees the declaring class sees its `internal` members.
* `public` means that any client who sees the declaring class sees its `public` members.

## Constructors

Use the following syntax to specify the visibility of the primary constructor of a class:

You need to add an explicit `constructor` keyword.

```kotlin
class C private constructor(a: Int) { ... }
```

Here the constructor is `private`. By default, all constructors are `public`, which effectively amounts to them being visible everywhere the class is visible (this means that a constructor of an `internal` class is only visible within the same module).
