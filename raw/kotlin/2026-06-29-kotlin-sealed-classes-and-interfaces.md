# Sealed classes and interfaces

> Source: https://kotlinlang.org/docs/sealed-classes.html
> Collected: 2026-08-11
> Published: 2026-06-29

## Relevant official excerpt

Sealed classes and interfaces provide controlled inheritance of your class hierarchies.
All direct subclasses of a sealed class are known at compile time.

When you combine sealed classes and interfaces with the `when` expression, you can cover the behavior of all possible subclasses and ensure that no new subclasses are created to affect your code adversely.

The key benefit of using sealed classes comes into play when you use them in a `when` expression.
The `when` expression, used with a sealed class, allows the Kotlin compiler to check exhaustively that all possible cases are covered.
In such cases, you don't need to add an `else` clause.

### State management in UI applications

You can use sealed classes to represent different UI states in an application.
This approach allows for structured and safe handling of UI changes.

```kotlin
sealed class UIState {
    data object Loading : UIState()
    data class Success(val data: String) : UIState()
    data class Error(val exception: Exception) : UIState()
}

fun updateUI(state: UIState) {
    when (state) {
        is UIState.Loading -> showLoadingIndicator()
        is UIState.Success -> showData(state.data)
        is UIState.Error -> showError(state.exception)
    }
}
```

