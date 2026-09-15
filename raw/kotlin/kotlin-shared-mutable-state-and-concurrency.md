# Shared mutable state and concurrency

> Source: https://kotlinlang.org/docs/shared-mutable-state-and-concurrency.html
> Collected: 2026-08-11
> Published: Unknown

## Relevant official excerpt

Coroutines can be executed parallelly using a multi-threaded dispatcher like the `Dispatchers.Default`.
It presents all the usual parallelism problems.
The main problem being synchronization of access to shared mutable state.

Consider a shared mutable counter:

```kotlin
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

It is highly unlikely to always print the expected final value because the counter is incremented concurrently from multiple threads without synchronization.

The general solution that works both for threads and for coroutines is to use a thread-safe data structure that provides all the necessary synchronization for the operations that need to be performed on a shared state.

