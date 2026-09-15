# Collections

> Source: https://kotlinlang.org/docs/kotlin-tour-collections.html
> Collected: 2026-08-10
> Published: Unknown

When programming, it is useful to be able to group data into structures for later processing. Kotlin provides collections for exactly this purpose.

Kotlin has the following collections for grouping items:

| Collection type | Description |
|---|---|
| Lists | Ordered collections of items |
| Sets | Unique unordered collections of items |
| Maps | Sets of key-value pairs where keys are unique and map to only one value |

Each collection type can be mutable or read only.

## List

Lists store items in the order that they are added, and allow for duplicate items.

To create a read-only list ( `List` ), use the `listOf()` function.

To create a mutable list ( `MutableList` ), use the `mutableListOf()` function.

When creating lists, Kotlin can infer the type of items stored. To declare the type explicitly, add the type within angled brackets `<>` after the list declaration:

```kotlin
fun main() { 
//sampleStart
    // Read only list
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println(readOnlyShapes)
    // [triangle, square, circle]
    
    // Mutable list with explicit type declaration
    val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
    println(shapes)
    // [triangle, square, circle]
//sampleEnd
}
```

To prevent unwanted modifications, you can create a read-only view of a mutable list by assigning it to a `List`:

```kotlin
    val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
    val shapesLocked: List<String> = shapes
```

This is also called casting .

Lists are ordered so to access an item in a list, use the indexed access operator `[]`:

```kotlin
fun main() { 
//sampleStart
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println("The first item in the list is: ${readOnlyShapes[0]}")
    // The first item in the list is: triangle
//sampleEnd
}
```

To get the first or last item in a list, use `.first()` and `.last()` functions respectively:

```kotlin
fun main() { 
//sampleStart
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println("The first item in the list is: ${readOnlyShapes.first()}")
    // The first item in the list is: triangle
//sampleEnd
}
```

`.first()` and `.last()` functions are examples of extension functions. To call an extension function on an object, write the function name after the object appended with a period `.`

Extension functions are covered in detail in the intermediate tour . For now, you only need to know how to call them.

To get the number of items in a list, use the `.count()` function:

```kotlin
fun main() { 
//sampleStart
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println("This list has ${readOnlyShapes.count()} items")
    // This list has 3 items
//sampleEnd
}
```

To check that an item is in a list, use the `in` operator :

```kotlin
fun main() {
//sampleStart
    val readOnlyShapes = listOf("triangle", "square", "circle")
    println("circle" in readOnlyShapes)
    // true
//sampleEnd
}
```

To add or remove items from a mutable list, use `.add()` and `.remove()` functions respectively:

```kotlin
fun main() { 
//sampleStart
    val shapes: MutableList<String> = mutableListOf("triangle", "square", "circle")
    // Add "pentagon" to the list
    shapes.add("pentagon") 
    println(shapes)  
    // [triangle, square, circle, pentagon]

    // Remove the first "pentagon" from the list
    shapes.remove("pentagon") 
    println(shapes)  
    // [triangle, square, circle]
//sampleEnd
}
```

## Set

Whereas lists are ordered and allow duplicate items, sets are unordered and only store unique items.

To create a read-only set ( `Set` ), use the `setOf()` function.

To create a mutable set ( `MutableSet` ), use the `mutableSetOf()` function.

When creating sets, Kotlin can infer the type of items stored. To declare the type explicitly, add the type within angled brackets `<>` after the set declaration:

```kotlin
fun main() {
//sampleStart
    // Read-only set
    val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
    // Mutable set with explicit type declaration
    val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")
    
    println(readOnlyFruit)
    // [apple, banana, cherry]
//sampleEnd
}
```

You can see in the previous example that because sets only contain unique elements, the duplicate `"cherry"` item is dropped.

To prevent unwanted modifications, you can create a read-only view of a mutable set by assigning it to a `Set`:

```kotlin
    val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")
    val fruitLocked: Set<String> = fruit
```

As sets are unordered , you can't access an item at a particular index.

To get the number of items in a set, use the `.count()` function:

```kotlin
fun main() { 
//sampleStart
    val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
    println("This set has ${readOnlyFruit.count()} items")
    // This set has 3 items
//sampleEnd
}
```

To check that an item is in a set, use the `in` operator :

```kotlin
fun main() {
//sampleStart
    val readOnlyFruit = setOf("apple", "banana", "cherry", "cherry")
    println("banana" in readOnlyFruit)
    // true
//sampleEnd
}
```

To add or remove items from a mutable set, use `.add()` and `.remove()` functions respectively:

```kotlin
fun main() { 
//sampleStart
    val fruit: MutableSet<String> = mutableSetOf("apple", "banana", "cherry", "cherry")
    fruit.add("dragonfruit")    // Add "dragonfruit" to the set
    println(fruit)              // [apple, banana, cherry, dragonfruit]
    
    fruit.remove("dragonfruit") // Remove "dragonfruit" from the set
    println(fruit)              // [apple, banana, cherry]
//sampleEnd
}
```

## Map

Maps store items as key-value pairs. You access the value by referencing the key. You can imagine a map like a food menu. You can find the price (value), by finding the food (key) you want to eat. Maps are useful if you want to look up a value without using a numbered index, like in a list.

-
Every key in a map must be unique so that Kotlin can understand which value you want to get.

-
You can have duplicate values in a map.

To create a read-only map ( `Map` ), use the `mapOf()` function.

To create a mutable map ( `MutableMap` ), use the `mutableMapOf()` function.

When creating maps, Kotlin can infer the type of items stored. To declare the type explicitly, add the types of the keys and values within angled brackets `<>` after the map declaration. For example: `MutableMap `. The keys have type `String` and the values have type `Int`.

The easiest way to create maps is to use `to` between each key and its related value:

```kotlin
fun main() {
//sampleStart
    // Read-only map
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println(readOnlyJuiceMenu)
    // {apple=100, kiwi=190, orange=100}

    // Mutable map with explicit type declaration
    val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println(juiceMenu)
    // {apple=100, kiwi=190, orange=100}
//sampleEnd
}
```

To prevent unwanted modifications, you can create a read-only view of a mutable map by assigning it to a `Map`:

```kotlin
    val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    val juiceMenuLocked: Map<String, Int> = juiceMenu
```

To access a value in a map, use the indexed access operator `[]` with its key:

```kotlin
fun main() {
//sampleStart
    // Read-only map
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println("The value of apple juice is: ${readOnlyJuiceMenu["apple"]}")
    // The value of apple juice is: 100
//sampleEnd
}
```

If you try to access a key-value pair with a key that doesn't exist in a map, you see a `null` value:

```kotlin
fun main() {
//sampleStart
    // Read-only map
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println("The value of pineapple juice is: ${readOnlyJuiceMenu["pineapple"]}")
    // The value of pineapple juice is: null
//sampleEnd
}
```

This tour explains null values later in the Null safety chapter.

You can also use the indexed access operator `[]` to add items to a mutable map:

```kotlin
fun main() {
//sampleStart
    val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    juiceMenu["coconut"] = 150 // Add key "coconut" with value 150 to the map
    println(juiceMenu)
    // {apple=100, kiwi=190, orange=100, coconut=150}
//sampleEnd
}
```

To remove items from a mutable map, use the `.remove()` function:

```kotlin
fun main() {
//sampleStart
    val juiceMenu: MutableMap<String, Int> = mutableMapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    juiceMenu.remove("orange")    // Remove key "orange" from the map
    println(juiceMenu)
    // {apple=100, kiwi=190}
//sampleEnd
}
```

To get the number of items in a map, use the `.count()` function:

```kotlin
fun main() {
//sampleStart
    // Read-only map
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println("This map has ${readOnlyJuiceMenu.count()} key-value pairs")
    // This map has 3 key-value pairs
//sampleEnd
}
```

To check if a specific key is already included in a map, use the `.containsKey()` function:

```kotlin
fun main() {
//sampleStart
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println(readOnlyJuiceMenu.containsKey("kiwi"))
    // true
//sampleEnd
}
```

To obtain a collection of the keys or values of a map, use the `keys` and `values` properties respectively:

```kotlin
fun main() {
//sampleStart
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println(readOnlyJuiceMenu.keys)
    // [apple, kiwi, orange]
    println(readOnlyJuiceMenu.values)
    // [100, 190, 100]
//sampleEnd
}
```

`keys` and `values` are examples of properties of an object. To access the property of an object, write the property name after the object appended with a period `.`

Properties are discussed in more detail in the Classes chapter. At this point in the tour, you only need to know how to access them.

To check that a key or value is in a map, use the `in` operator :

```kotlin
fun main() {
//sampleStart
    val readOnlyJuiceMenu = mapOf("apple" to 100, "kiwi" to 190, "orange" to 100)
    println("orange" in readOnlyJuiceMenu.keys)
    // true
    
    // Alternatively, you don't need to use the keys property
    println("orange" in readOnlyJuiceMenu)
    // true
    
    println(200 in readOnlyJuiceMenu.values)
    // false
//sampleEnd
}
```

For more information on what you can do with collections, see Collections .

Now that you know about basic types and how to manage collections, it's time to explore the control flow that you can use in your programs.

## Practice

### Exercise 1

You have a list of &ldquo;green&rdquo; numbers and a list of &ldquo;red&rdquo; numbers. Complete the code to print how many numbers there are in total.

```kotlin
fun main() {
    val greenNumbers = listOf(1, 4, 23)
    val redNumbers = listOf(17, 2)
    // Write your code here
}
```

fun main() {
val greenNumbers = listOf(1, 4, 23)
val redNumbers = listOf(17, 2)
val totalCount = greenNumbers.count() + redNumbers.count()
println(totalCount)
}

### Exercise 2

You have a set of protocols supported by your server. A user requests to use a particular protocol. Complete the program to check whether the requested protocol is supported or not (`isSupported` must be a Boolean value).

```kotlin
fun main() {
    val SUPPORTED = setOf("HTTP", "HTTPS", "FTP")
    val requested = "smtp"
    val isSupported = // Write your code here 
    println("Support for $requested: $isSupported")
}
```
Hint
Make sure that you check the requested protocol in upper case. You can use the `.uppercase()` function to help you with this.

fun main() {
val SUPPORTED = setOf("HTTP", "HTTPS", "FTP")
val requested = "smtp"
val isSupported = requested.uppercase() in SUPPORTED
println("Support for $requested: $isSupported")
}

### Exercise 3

Define a map that relates integer numbers from 1 to 3 to their corresponding spelling. Use this map to spell the given number.

```kotlin
fun main() {
    val number2word = // Write your code here
    val n = 2
    println("$n is spelled as '${<Write your code here >}'")
}
```

fun main() {
val number2word = mapOf(1 to "one", 2 to "two", 3 to "three")
val n = 2
println("$n is spelt as '${number2word[n]}'")
}

-
Previous step

-
Next step

Last modified: 01 July 2026
