# kotlinx.coroutines — StateFlow

> Source: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/
> Collected: 2026-10-15
> Published: Unknown
> Scope: API page main content; navigation omitted.

[kotlinx-coroutines-core](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/index.html)/[kotlinx.coroutines.flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/index.html)/StateFlow

# StateFlow

interface [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)<out [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)> : [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)> ([source](https://github.com/kotlin/kotlinx.coroutines/tree/master/kotlinx-coroutines-core/common/src/flow/StateFlow.kt#L135))

A [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) that represents a read-only state with a single updatable data [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) that emits updates to the value to its collectors. A state flow is a *hot* flow because its active instance exists independently of the presence of collectors. Its current value can be retrieved via the [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) property.

**State flow never completes**. A call to [Flow.collect](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/collect.html) on a state flow never completes normally, and neither does a coroutine started by the [Flow.launchIn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/launch-in.html) function. An active collector of a state flow is called a *subscriber*.

A [mutable state flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/index.html) is created using `MutableStateFlow(value)` constructor function with the initial value. The value of mutable state flow can be updated by setting its [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) property. Updates to the [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) are always [conflated](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/conflate.html). So a slow collector skips fast updates, but always collects the most recently emitted value.

[StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html) is useful as a data-model class to represent any kind of state. Derived values can be defined using various operators on the flows, with [combine](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) operator being especially useful to combine values from multiple state flows using arbitrary functions.

For example, the following class encapsulates an integer state and increments its value on each call to `inc`:

```
class CounterModel {  
    private val _counter = MutableStateFlow(0) // private mutable state flow  
    val counter = _counter.asStateFlow() // publicly exposed as read-only state flow  
  
    fun inc() {  
        _counter.update { count -> count + 1 } // atomic, safe for concurrent use  
    }  
}
```

Content copied to clipboard

Having two instances of the above `CounterModel` class one can define the sum of their counters like this:

```
val aModel = CounterModel()  
val bModel = CounterModel()  
val sumFlow: Flow<Int> = aModel.counter.combine(bModel.counter) { a, b -> a + b }
```

Content copied to clipboard

As an alternative to the above usage with the `MutableStateFlow(...)` constructor function, any *cold* [Flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) can be converted to a state flow using the [stateIn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html) operator.

### Strong equality-based conflation

Values in state flow are conflated using [Any.equals](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-any/equals.html) comparison in a similar way to [distinctUntilChanged](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html) operator. It is used to conflate incoming updates to [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/value.html) in [MutableStateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/index.html) and to suppress emission of the values to collectors when new value is equal to the previously emitted one. State flow behavior with classes that violate the contract for [Any.equals](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-any/equals.html) is unspecified.

### State flow is a shared flow

State flow is a special-purpose, high-performance, and efficient implementation of [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) for the narrow, but widely used case of sharing a state. See the [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) documentation for the basic rules, constraints, and operators that are applicable to all shared flows.

State flow always has an initial value, replays one most recent value to new subscribers, does not buffer any more values, but keeps the last emitted one, and does not support [resetReplayCache](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/reset-replay-cache.html). A state flow behaves identically to a shared flow when it is created with the following parameters and the [distinctUntilChanged](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html) operator is applied to it:

```
// MutableStateFlow(initialValue) is a shared flow with the following parameters:  
val shared = MutableSharedFlow(  
    replay = 1,  
    onBufferOverflow = BufferOverflow.DROP_OLDEST  
)  
shared.tryEmit(initialValue) // emit the initial value  
val state = shared.distinctUntilChanged() // get StateFlow-like behavior
```

Content copied to clipboard

Use [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html) when you need a [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html) with tweaks in its behavior such as extra buffering, replaying more values, or omitting the initial value.

### StateFlow vs ConflatedBroadcastChannel

Conceptually, state flow is similar to [ConflatedBroadcastChannel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-conflated-broadcast-channel/index.html) and is designed to completely replace it. It has the following important differences:

- `StateFlow` is simpler, because it does not have to implement all the [Channel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/index.html) APIs, which allows for faster, garbage-free implementation, unlike `ConflatedBroadcastChannel` implementation that allocates objects on each emitted value.
- `StateFlow` always has a value which can be safely read at any time via [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) property. Unlike `ConflatedBroadcastChannel`, there is no way to create a state flow without a value.
- `StateFlow` has a clear separation into a read-only `StateFlow` interface and a [MutableStateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/index.html).
- `StateFlow` conflation is based on equality like [distinctUntilChanged](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html) operator, unlike conflation in `ConflatedBroadcastChannel` that is based on reference identity.
- `StateFlow` cannot be closed like `ConflatedBroadcastChannel` and can never represent a failure. All errors and completion signals should be explicitly *materialized* if needed.

`StateFlow` is designed to better cover typical use-cases of keeping track of state changes in time, taking more pragmatic design choices for the sake of convenience.

To migrate [ConflatedBroadcastChannel](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-conflated-broadcast-channel/index.html) usage to [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html), start by replacing usages of the `ConflatedBroadcastChannel()` constructor with `MutableStateFlow(initialValue)`, using `null` as an initial value if you don't have one. Replace [send](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-conflated-broadcast-channel/send.html) and [trySend](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-conflated-broadcast-channel/try-send.html) calls with updates to the state flow's [MutableStateFlow.value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/value.html), and convert subscribers' code to flow operators. You can use the [filterNotNull](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/filter-not-null.html) operator to mimic behavior of a `ConflatedBroadcastChannel` without initial value.

### Concurrency

All methods of state flow are **thread-safe** and can be safely invoked from concurrent coroutines without external synchronization.

### Operator fusion

Application of [flowOn](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html), [conflate](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/conflate.html), [buffer](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/buffer.html) with [CONFLATED](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/-factory/-c-o-n-f-l-a-t-e-d.html) or [RENDEZVOUS](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/-factory/-r-e-n-d-e-z-v-o-u-s.html) capacity, [distinctUntilChanged](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html), or [cancellable](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/cancellable.html) operators to a state flow has no effect.

### Implementation notes

State flow implementation is optimized for memory consumption and allocation-freedom. It uses a lock to ensure thread-safety, but suspending collector coroutines are resumed outside of this lock to avoid dead-locks when using unconfined coroutines. Adding new subscribers has `O(1)` amortized cost, but updating a [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html) has `O(N)` cost, where `N` is the number of active subscribers.

### Not stable for inheritance

`The StateFlow` **interface is not stable for inheritance in 3rd party libraries**, as new methods might be added to this interface in the future, but is stable for use. Use the `MutableStateFlow(value)` constructor function to create an implementation.

#### Inheritors

[MutableStateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/index.html)

MembersMembers & Extensions

## Properties

[value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html)

Link copied to clipboard

abstract val [value](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/value.html): [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)

The current value of this state flow.

## Functions

[onSubscription](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)

Link copied to clipboard

fun <[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)> [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)>.[onSubscription](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)(action: suspend [FlowCollector](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)>.() -> [Unit](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-unit/index.html)): [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html)>

Returns a flow that invokes the given [action](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-subscription.html) **after** this state flow starts to be collected (after the subscription is registered).
