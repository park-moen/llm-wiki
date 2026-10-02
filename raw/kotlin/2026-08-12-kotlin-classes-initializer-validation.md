# Kotlin Classes: Constructors and initializer blocks

> Source: https://kotlinlang.org/docs/classes.html
> Collected: 2026-10-02
> Published: 2026-08-12

## Constructors and initializer blocks

When you create a class instance, you call one of its constructors. The primary constructor is the main way to initialize a class. A secondary constructor provides additional initialization logic. Both primary and secondary constructors are optional, but a class must have at least one constructor.

## Initializer blocks

The primary constructor initializes the class and sets its properties. In most cases, you can handle this with simple code.

If you need to perform more complex operations during instance creation, place that logic in initializer blocks inside the class body. These blocks run when the primary constructor executes.

Declare initializer blocks with the `init` keyword followed by curly braces `{}`. Write within the curly braces any code you want to run during initialization:

```kotlin
class Person(val name: String, var age: Int) {
    init {
        println("Person created: $name, age $age.")
    }
}
```

You can use primary constructor parameters in initializer blocks. A common use case for `init` blocks is data validation. For example, by calling the `require` function:

```kotlin
class Person(val age: Int) {
    init {
        require(age > 0) { "age must be positive" }
    }
}
```

## Companion objects

In Kotlin, each class can have a companion object. Companion objects are a type of object declaration that allows you to access its members using the class name without creating a class instance.

Suppose you need to write a function that can be called without creating an instance of a class, but it is still logically connected to the class (such as a factory function). In that case, you can declare it inside a companion object declaration within the class:

```kotlin
class Person(val name: String) {
    companion object {
        fun createAnonymous() = Person("Anonymous")
    }
}

val anonymous = Person.createAnonymous()
```
