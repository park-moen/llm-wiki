# Kotlin — Data classes

> Source: https://kotlinlang.org/docs/data-classes.html
> Collected: 2026-09-29
> Published: Unknown
> Scope: Selected passages on generated equality and parameterless constructors.

Data classes in Kotlin are primarily used to hold data. For each data class, the compiler automatically generates additional member functions that allow you to print an instance to readable output, compare instances, copy instances, and more.

The compiler automatically derives the following members from all properties declared in the primary constructor:

- `equals()`/`hashCode()` pair.
- `toString()` of the form `"User(name=John, age=42)"`.
- `componentN()` functions corresponding to their order of declaration.
- `copy()` function.

Data classes can't be abstract, open, sealed, or inner.

On the JVM, if the generated class needs to have a parameterless constructor, default values for the properties have to be specified (see Constructors):

```kotlin
data class User(val name: String = "", val age: Int = 0)
```
