# Object declarations and expressions

> Source: https://kotlinlang.org/docs/object-declarations.html
> Collected: 2026-08-11
> Published: 2025-02-23

## Overview excerpt

In Kotlin, objects allow you to define a class and create an instance of it in a single step.
This is useful when you need either a reusable singleton instance or a one-time object.
To handle these scenarios, Kotlin provides two key approaches: object declarations for creating singletons and object expressions for creating anonymous, one-time objects.

> A singleton ensures that a class has only one instance and provides a global point of access to it.

## Object declarations excerpt

You can create single instances of objects in Kotlin using object declarations, which always have a name following the `object` keyword.
This allows you to define a class and create an instance of it in a single step, which is useful for implementing singletons.

> The initialization of an object declaration is thread-safe and done on first access.

To refer to the `object`, use its name directly:

```kotlin
DataProviderManager.registerDataProvider(exampleProvider)
```

Object declarations can also have supertypes.

Object declarations cannot be local, which means they cannot be nested directly inside a function.
However, they can be nested within other object declarations or non-inner classes.

## Data objects excerpt

When printing a plain object declaration in Kotlin, the string representation contains both its name and the hash of the `object`:

```kotlin
object MyObject

fun main() {
    println(MyObject)
    // MyObject@hashcode
}
```

By marking an object declaration with the `data` modifier, you can instruct the compiler to return the actual name of the object when calling `toString()`:

```kotlin
data object MyDataObject {
    val number: Int = 3
}

fun main() {
    println(MyDataObject)
    // MyDataObject
}
```

The compiler generates several functions for your `data object`:

* `toString()` returns the name of the data object.
* `equals()`/`hashCode()` enables equality checks and hash-based collections.

The `equals()` function for a `data object` ensures that all objects that have the type of your `data object` are considered equal.

> Make sure that you only compare `data objects` structurally (using the `==` operator) and never by reference (using the `===` operator).

Unlike a `data class`, a `data object` has no generated `copy()` or `componentN()` functions.

Data object declarations are particularly useful for sealed hierarchies like sealed classes or sealed interfaces.

```kotlin
sealed interface ReadResult
data class Number(val number: Int) : ReadResult
data class Text(val text: String) : ReadResult
data object EndOfFile : ReadResult
```

## Companion objects excerpt

Companion objects allow you to define class-level functions and properties.
This makes it easy to create factory methods, hold constants, and access shared utilities.

```kotlin
class User(val name: String) {
    companion object Factory {
        fun create(name: String): User = User(name)
    }
}

fun main() {
    val userInstance = User.create("John Doe")
    println(userInstance.name)
}
```

The name of the `companion object` can be omitted, in which case the name `Companion` is used.

Although members of companion objects in Kotlin look like static members from other languages, they are actually instance members of the companion object, meaning they belong to the object itself.

## Object expressions excerpt

Object expressions declare a class and create an instance of that class, but without naming either of them.
These classes are useful for one-time use.
Instances of these classes are also called anonymous objects because they are defined by an expression, not a name.

```kotlin
val helloWorld = object {
    val hello = "Hello"
    val world = "World"
    override fun toString() = "$hello $world"
}
```

Object expressions are executed and initialized immediately, where they are used.
Object declarations are initialized lazily, when accessed for the first time.

