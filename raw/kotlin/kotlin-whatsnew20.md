# What's new in Kotlin 2.0.0 — compiler and JVM excerpts

> Source: https://kotlinlang.org/docs/whatsnew20.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Selected complete sections: kotlin-k2-compiler, kotlin-jvm. Other platform and tooling sections omitted.

## Kotlin K2 compiler

The road to the K2 compiler has been a long one, but now the JetBrains team is finally ready to announce its stabilization. In Kotlin 2.0.0, the new Kotlin K2 compiler is used by default and it is [Stable](https://kotlinlang.org/docs/components-stability.html) for all target platforms: JVM, Native, Wasm, and JS. The new compiler brings major performance improvements, speeds up new language feature development, unifies all platforms that Kotlin supports, and provides a better architecture for multiplatform projects.

The JetBrains team has ensured the quality of the new compiler by successfully compiling 10 million lines of code from selected user and internal projects. 18,000 developers were involved in the stabilization process, testing the new K2 compiler across a total of 80,000 projects and reporting any problems they found.

To help make the migration process to the new compiler as smooth as possible, we've created a [K2 compiler migration guide](https://kotlinlang.org/docs/k2-compiler-migration-guide.html). This guide explains the many benefits of the compiler, highlights any changes you might encounter, and describes how to roll back to the previous version if necessary.

In a [blog post](https://blog.jetbrains.com/kotlin/2024/04/k2-compiler-performance-benchmarks-and-how-to-measure-them-on-your-projects/), we explored the performance of the K2 compiler in different projects. Check it out if you'd like to see real data on how the K2 compiler performs and find instructions on how to collect performance benchmarks from your own projects.

You can also watch this talk from KotlinConf 2024, where Michail Zarečenskij, the lead language designer, discusses the feature evolution in Kotlin and the K2 compiler:

### Current K2 compiler limitations

Enabling K2 in your Gradle project comes with certain limitations that can affect projects using Gradle versions below 8.3 in the following cases:

- Compilation of source code from `buildSrc`.
- Compilation of Gradle plugins in included builds.
- Compilation of other Gradle plugins if they are used in projects with Gradle versions below 8.3.
- Building Gradle plugin dependencies.

If you encounter any of the problems mentioned above, you can take the following steps to address them:

- Set the language version for `buildSrc`, any Gradle plugins, and their dependencies:

  kotlin {
  compilerOptions {
  languageVersion.set(org.jetbrains.kotlin.gradle.dsl.KotlinVersion.KOTLIN_1_9)
  apiVersion.set(org.jetbrains.kotlin.gradle.dsl.KotlinVersion.KOTLIN_1_9)
  }
  }

  If you configure language and API versions for specific tasks, these values will override the values set by the `compilerOptions` extension. In this case, language and API versions should not be higher than 1.9.
- Update the Gradle version in your project to 8.3 or later.

### Smart cast improvements

The Kotlin compiler can automatically cast an object to a type in specific cases, saving you the trouble of having to explicitly cast it yourself. This is called [smart casting](https://kotlinlang.org/docs/typecasts.html#smart-casts). The Kotlin K2 compiler now performs smart casts in even more scenarios than before.

In Kotlin 2.0.0, we've made improvements related to smart casts in the following areas:

- [Local variables and further scopes](https://kotlinlang.org/docs/whatsnew20.html#local-variables-and-further-scopes)
- [Type checks with logical `or` operator](https://kotlinlang.org/docs/whatsnew20.html#type-checks-with-logical-or-operator)
- [Inline functions](https://kotlinlang.org/docs/whatsnew20.html#inline-functions)
- [Properties with function types](https://kotlinlang.org/docs/whatsnew20.html#properties-with-function-types)
- [Exception handling](https://kotlinlang.org/docs/whatsnew20.html#exception-handling)
- [Increment and decrement operators](https://kotlinlang.org/docs/whatsnew20.html#increment-and-decrement-operators)

#### Local variables and further scopes

Previously, if a variable was evaluated as not `null` within an `if` condition, the variable would be smart-cast. Information about this variable would then be shared further within the scope of the `if` block.

However, if you declared the variable outside the `if` condition, no information about the variable would be available within the `if` condition, so it couldn't be smart-cast. This behavior was also seen with `when` expressions and `while` loops.

From Kotlin 2.0.0, if you declare a variable before using it in your `if`, `when`, or `while` condition, then any information collected by the compiler about the variable will be accessible in the corresponding block for smart-casting.

This can be useful when you want to do things like extract boolean conditions into variables. Then, you can give the variable a meaningful name, which will improve your code readability and make it possible to reuse the variable later in your code. For example:

class Cat {
fun purr() {
println("Purr purr")
}
}
fun petAnimal(animal: Any) {
val isCat = animal is Cat
if (isCat) {
// In Kotlin 2.0.0, the compiler can access
// information about isCat, so it knows that
// animal was smart-cast to the type Cat.
// Therefore, the purr() function can be called.
// In Kotlin 1.9.20, the compiler doesn't know
// about the smart cast, so calling the purr()
// function triggers an error.
animal.purr()
}
}
fun main() {
val kitty = Cat()
petAnimal(kitty)
// Purr purr
}

#### Type checks with logical or operator

In Kotlin 2.0.0, if you combine type checks for objects with an `or` operator (`||`), a smart cast is made to their closest common supertype. Before this change, a smart cast was always made to the `Any` type.

In this case, you still had to manually check the object type afterward before you could access any of its properties or call its functions. For example:

interface Status {
fun signal() {}
}
interface Ok : Status
interface Postponed : Status
interface Declined : Status
fun signalCheck(signalStatus: Any) {
if (signalStatus is Postponed || signalStatus is Declined) {
// signalStatus is smart-cast to a common supertype Status
signalStatus.signal()
// Prior to Kotlin 2.0.0, signalStatus is smart cast
// to type Any, so calling the signal() function triggered an
// Unresolved reference error. The signal() function can only
// be called successfully after another type check:
// check(signalStatus is Status)
// signalStatus.signal()
}
}

The common supertype is an approximation of a union type. [Union types](https://en.wikipedia.org/wiki/Union_type) are not supported in Kotlin.

#### Inline functions

In Kotlin 2.0.0, the K2 compiler treats inline functions differently, allowing it to determine in combination with other compiler analyses whether it's safe to smart-cast.

Specifically, inline functions are now treated as having an implicit [`callsInPlace`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.contracts/-contract-builder/calls-in-place.html) contract. This means that any lambda functions passed to an inline function are called in place. Since lambda functions are called in place, the compiler knows that a lambda function can't leak references to any variables contained within its function body.

The compiler uses this knowledge along with other compiler analyses to decide whether it's safe to smart-cast any of the captured variables. For example:

interface Processor {
fun process()
}
inline fun inlineAction(f: () -> Unit) = f()
fun nextProcessor(): Processor? = null
fun runProcessor(): Processor? {
var processor: Processor? = null
inlineAction {
// In Kotlin 2.0.0, the compiler knows that processor
// is a local variable, and inlineAction() is an inline function, so
// references to processor can't be leaked. Therefore, it's safe
// to smart-cast processor.
// If processor isn't null, processor is smart-cast
if (processor != null) {
// The compiler knows that processor isn't null, so no safe call
// is needed
processor.process()
// In Kotlin 1.9.20, you have to perform a safe call:
// processor?.process()
}
processor = nextProcessor()
}
return processor
}

#### Properties with function types

In previous versions of Kotlin, there was a bug that meant that class properties with a function type weren't smart-cast. We fixed this behavior in Kotlin 2.0.0 and the K2 compiler. For example:

class Holder(val provider: (() -> Unit)?) {
fun process() {
// In Kotlin 2.0.0, if provider isn't null, then
// provider is smart-cast
if (provider != null) {
// The compiler knows that provider isn't null
provider()
// In 1.9.20, the compiler doesn't know that provider isn't
// null, so it triggers an error:
// Reference has a nullable type '(() -> Unit)?', use explicit '?.invoke()' to make a function-like call instead
}
}
}

This change also applies if you overload your `invoke` operator. For example:

interface Provider {
operator fun invoke()
}
interface Processor : () -> String
class Holder(val provider: Provider?, val processor: Processor?) {
fun process() {
if (provider != null) {
provider()
// In 1.9.20, the compiler triggers an error:
// Reference has a nullable type 'Provider?' use explicit '?.invoke()' to make a function-like call instead
}
}
}

#### Exception handling

In Kotlin 2.0.0, we've made improvements to exception handling so that smart cast information can be passed on to `catch` and `finally` blocks. This change makes your code safer as the compiler keeps track of whether your object has a nullable type. For example:

//sampleStart
fun testString() {
var stringInput: String? = null
// stringInput is smart-cast to String type
stringInput = ""
try {
// The compiler knows that stringInput isn't null
println(stringInput.length)
// 0
// The compiler rejects previous smart cast information for
// stringInput. Now stringInput has the String? type.
stringInput = null
// Trigger an exception
if (2 > 1) throw Exception()
stringInput = ""
} catch (exception: Exception) {
// In Kotlin 2.0.0, the compiler knows stringInput
// can be null, so stringInput stays nullable.
println(stringInput?.length)
// null
// In Kotlin 1.9.20, the compiler says that a safe call isn't
// needed, but this is incorrect.
}
}
//sampleEnd
fun main() {
testString()
}

#### Increment and decrement operators

Prior to Kotlin 2.0.0, the compiler didn't understand that the type of an object can change after using an increment or decrement operator. As the compiler couldn't accurately track the object type, your code could lead to unresolved reference errors. In Kotlin 2.0.0, this has been fixed:

interface Rho {
operator fun inc(): Sigma = TODO()
}
interface Sigma : Rho {
fun sigma() = Unit
}
interface Tau {
fun tau() = Unit
}
fun main(input: Rho) {
var unknownObject: Rho = input
// Check if unknownObject inherits from the Tau interface
// Note, it's possible that unknownObject inherits from both
// Rho and Tau interfaces.
if (unknownObject is Tau) {
// Use the overloaded inc() operator from interface Rho.
// In Kotlin 2.0.0, the type of unknownObject is smart-cast to
// Sigma.
++unknownObject
// In Kotlin 2.0.0, the compiler knows unknownObject has type
// Sigma, so the sigma() function can be called successfully.
unknownObject.sigma()
// In Kotlin 1.9.20, the compiler doesn't perform a smart cast
// when inc() is called so the compiler still thinks that
// unknownObject has type Tau. Calling the sigma() function
// throws a compile-time error.
// In Kotlin 2.0.0, the compiler knows unknownObject has type
// Sigma, so calling the tau() function throws a compile-time
// error.
unknownObject.tau()
// Unresolved reference 'tau'
// In Kotlin 1.9.20, since the compiler mistakenly thinks that
// unknownObject has type Tau, the tau() function can be called,
// but it throws a ClassCastException.
}
}

### Kotlin Multiplatform improvements

In Kotlin 2.0.0, we've made improvements in the K2 compiler related to Kotlin Multiplatform in the following areas:

- [Separation of common and platform sources during compilation](https://kotlinlang.org/docs/whatsnew20.html#separation-of-common-and-platform-sources-during-compilation)
- [Different visibility levels of expected and actual declarations](https://kotlinlang.org/docs/whatsnew20.html#different-visibility-levels-of-expected-and-actual-declarations)

#### Separation of common and platform sources during compilation

Previously, the design of the Kotlin compiler prevented it from keeping common and platform source sets separate at compile time. As a consequence, common code could access platform code, which resulted in different behavior between platforms. In addition, some compiler settings and dependencies from common code used to leak into platform code.

In Kotlin 2.0.0, our implementation of the new Kotlin K2 compiler included a redesign of the compilation scheme to ensure strict separation between common and platform source sets. This change is most noticeable when you use [expected and actual functions](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html#expected-and-actual-functions). Previously, it was possible for a function call in your common code to resolve to a function in platform code. For example:

| Common code | Platform code |
| --- | --- |
| fun foo(x: Any) = println("common foo") fun exampleFunction() { foo(42) } | // JVM fun foo(x: Int) = println("platform foo") // JavaScript // There is no foo() function overload // on the JavaScript platform |

In this example, the common code has different behavior depending on which platform it is run on:

- On the JVM platform, calling the `foo()` function in the common code results in the `foo()` function from the platform code being called as `platform foo`.
- On the JavaScript platform, calling the `foo()` function in the common code results in the `foo()` function from the common code being called as `common foo`, as there is no such function available in the platform code.

In Kotlin 2.0.0, common code doesn't have access to platform code, so both platforms successfully resolve the `foo()` function to the `foo()` function in the common code: `common foo`.

In addition to the improved consistency of behavior across platforms, we also worked hard to fix cases where there was conflicting behavior between IntelliJ IDEA or Android Studio and the compiler. For instance, when you used [expected and actual classes](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html#expected-and-actual-classes), the following would happen:

| Common code | Platform code |
| --- | --- |
| expect class Identity { fun confirmIdentity(): String } fun common() { // Before 2.0.0, // it triggers an IDE-only error Identity().confirmIdentity() // RESOLUTION_TO_CLASSIFIER : Expected class // Identity has no default constructor. } | actual class Identity { actual fun confirmIdentity() = "expect class fun: jvm" } |

In this example, the expected class `Identity` has no default constructor, so it can't be called successfully in common code. Previously, an error was only reported by the IDE, but the code still compiled successfully on the JVM. However, now the compiler correctly reports an error:

Expected class 'expect class Identity : Any' does not have default constructor

##### When resolution behavior doesn't change

We're still in the process of migrating to the new compilation scheme, so the resolution behavior is still the same when you call functions that aren't within the same source set. You'll notice this difference mainly when you use overloads from a multiplatform library in your common code.

Suppose you have a library, which has two `whichFun()` functions with different signatures:

// Example library
// MODULE: common
fun whichFun(x: Any) = println("common function")
// MODULE: JVM
fun whichFun(x: Int) = println("platform function")

If you call the `whichFun()` function in your common code, the function that has the most relevant argument type in the library is resolved:

// A project that uses the example library for the JVM target
// MODULE: common
fun main() {
whichFun(2)
// platform function
}

In comparison, if you declare the overloads for `whichFun()` within the same source set, the function from the common code will be resolved because your code doesn't have access to the platform-specific version:

// Example library isn't used
// MODULE: common
fun whichFun(x: Any) = println("common function")
fun main() {
whichFun(2)
// common function
}
// MODULE: JVM
fun whichFun(x: Int) = println("platform function")

Similar to multiplatform libraries, since the `commonTest` module is in a separate source set, it also still has access to platform-specific code. Therefore, the resolution of calls to functions in the `commonTest` module exhibits the same behavior as in the old compilation scheme.

In the future, these remaining cases will be more consistent with the new compilation scheme.

#### Different visibility levels of expected and actual declarations

Before Kotlin 2.0.0, if you used [expected and actual declarations](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html) in your Kotlin Multiplatform project, they had to have the same [visibility level](https://kotlinlang.org/docs/visibility-modifiers.html). Kotlin 2.0.0 now also supports different visibility levels but only if the actual declaration is more permissive than the expected declaration. For example:

expect internal class Attribute // Visibility is internal
actual class Attribute // Visibility is public by default,
// which is more permissive

Similarly, if you are using a [type alias](https://kotlinlang.org/docs/type-aliases.html) in your actual declaration, the visibility of the underlying type should be the same or more permissive than the expected declaration. For example:

expect internal class Attribute // Visibility is internal
internal actual typealias Attribute = Expanded
class Expanded // Visibility is public by default,
// which is more permissive

### Compiler plugins support

Currently, the Kotlin K2 compiler supports the following Kotlin compiler plugins:

- [`all-open`](https://kotlinlang.org/docs/all-open-plugin.html)
- [AtomicFU](https://github.com/Kotlin/kotlinx-atomicfu)
- [`jvm-abi-gen`](https://github.com/JetBrains/kotlin/tree/master/plugins/jvm-abi-gen)
- [`js-plain-objects`](https://github.com/JetBrains/kotlin/tree/master/plugins/js-plain-objects)
- [kapt](https://kotlinlang.org/docs/whatsnew1920.html#preview-kapt-compiler-plugin-with-k2)
- [Lombok](https://kotlinlang.org/docs/lombok.html)
- [`no-arg`](https://kotlinlang.org/docs/no-arg-plugin.html)
- [Parcelize](https://plugins.gradle.org/plugin/org.jetbrains.kotlin.plugin.parcelize)
- [SAM with receiver](https://kotlinlang.org/docs/sam-with-receiver-plugin.html)
- [serialization](https://kotlinlang.org/docs/serialization.html)
- [Power-assert](https://kotlinlang.org/docs/power-assert.html)

In addition, the Kotlin K2 compiler supports:

- The [Jetpack Compose](https://developer.android.com/jetpack/compose) compiler plugin 2.0.0, which was [moved into the Kotlin repository](https://android-developers.googleblog.com/2024/04/jetpack-compose-compiler-moving-to-kotlin-repository.html).
- The [Kotlin Symbol Processing (KSP) plugin](https://kotlinlang.org/docs/ksp-overview.html) since [KSP2](https://android-developers.googleblog.com/2023/12/ksp2-preview-kotlin-k2-standalone.html).

If you use any additional compiler plugins, check their documentation to see if they are compatible with K2.

### Experimental Kotlin Power-assert compiler plugin

The Kotlin Power-assert plugin is [Experimental](https://kotlinlang.org/docs/components-stability.html#stability-levels-explained). It may be changed at any time.

Kotlin 2.0.0 introduces an experimental Power-assert compiler plugin. This plugin improves the experience of writing tests by including contextual information in failure messages, making debugging easier and more efficient.

Developers often need to use complex assertion libraries to write effective tests. The Power-assert plugin simplifies this process by automatically generating failure messages that include intermediate values of the assertion expression. This helps developers quickly understand why a test failed.

When an assertion fails in a test, the improved error message shows the values of all variables and sub-expressions within the assertion, making it clear which part of the condition caused the failure. This is particularly useful for complex assertions where multiple conditions are checked.

To enable the plugin in your project, configure it in your `build.gradle(.kts)` file:

plugins {
kotlin("multiplatform") version "2.0.0"
kotlin("plugin.power-assert") version "2.0.0"
}
powerAssert {
functions = listOf("kotlin.assert", "kotlin.test.assertTrue")
}

plugins {
id 'org.jetbrains.kotlin.multiplatform' version '2.0.0'
id 'org.jetbrains.kotlin.plugin.power-assert' version '2.0.0'
}
powerAssert {
functions = ["kotlin.assert", "kotlin.test.assertTrue"]
}

Learn more about the [Kotlin Power-assert plugin in the documentation](https://kotlinlang.org/docs/power-assert.html).

### How to enable the Kotlin K2 compiler

Starting with Kotlin 2.0.0, the Kotlin K2 compiler is enabled by default. No additional actions are required.

### Try the Kotlin K2 compiler in Kotlin Playground

Kotlin Playground supports the 2.0.0 release. [Check it out!](https://pl.kotl.in/czuoQprce)

### Support in IDEs

By default, IntelliJ IDEA and Android Studio still use the previous compiler for code analysis, code completion, highlighting, and other IDE-related features. To get the full Kotlin 2.0 experience in your IDE, enable K2 mode.

In your IDE, go to Settings | Languages & Frameworks | Kotlin and select the Enable K2 mode option. The IDE will analyze your code using its K2 mode.

![Enable K2 mode](https://kotlinlang.org/docs/images/k2-mode.png "Enable K2 mode")

After enabling K2 mode, you may notice differences in IDE analysis due to changes in compiler behavior. Learn how the new K2 compiler differs from the previous one in our [migration guide](https://kotlinlang.org/docs/k2-compiler-migration-guide.html).

- Learn more about K2 mode in [our blog](https://blog.jetbrains.com/idea/2024/11/k2-mode-becomes-stable/).
- We are actively collecting feedback about K2 mode, so please share your thoughts in our [public Slack channel](https://kotlinlang.slack.com/archives/C0B8H786P).

### Leave your feedback on the new K2 compiler

We would appreciate any feedback you may have!

- Report any problems you face with the new K2 compiler in [our issue tracker](https://kotl.in/issue).
- [Enable the "Send usage statistics" option](https://www.jetbrains.com/help/idea/settings-usage-statistics.html) to allow JetBrains to collect anonymous data about K2 usage.

## Kotlin/JVM

Starting with version 2.0.0, the compiler can generate classes containing Java 22 bytecode. This version also brings the following changes:

- [Generation of lambda functions using invokedynamic](https://kotlinlang.org/docs/whatsnew20.html#generation-of-lambda-functions-using-invokedynamic)
- [The kotlinx-metadata-jvm library is now Stable](https://kotlinlang.org/docs/whatsnew20.html#the-kotlinx-metadata-jvm-library-is-stable)

### Generation of lambda functions using invokedynamic

Kotlin 2.0.0 introduces a new default method for generating lambda functions using `invokedynamic`. This change reduces the binary sizes of applications compared to the traditional anonymous class generation.

Since the first version, Kotlin has generated lambdas as anonymous classes. However, starting from [Kotlin 1.5.0](https://kotlinlang.org/docs/whatsnew15.html#lambdas-via-invokedynamic), the option for `invokedynamic` generation has been available by using the `-Xlambdas=indy` compiler option. In Kotlin 2.0.0, `invokedynamic` has become the default method for lambda generation. This method produces lighter binaries and aligns Kotlin with JVM optimizations, ensuring applications benefit from ongoing and future improvements in JVM performance.

Currently, it has three limitations compared to ordinary lambda compilation:

- A lambda compiled into `invokedynamic` is not serializable.
- Experimental [`reflect()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect.jvm/reflect.html) API does not support lambdas generated by `invokedynamic`.
- Calling `.toString()` on such a lambda produces a less readable string representation:

fun main() {
println({})
// With Kotlin 1.9.24 and reflection, returns
// () -> kotlin.Unit
// With Kotlin 2.0.0, returns
// FileKt$$Lambda$13/0x00007f88a0004608@506e1b77
}

To retain the legacy behavior of generating lambda functions, you can either:

- Annotate specific lambdas with `@JvmSerializableLambda`.
- Use the compiler option `-Xlambdas=class` to generate all lambdas in a module using the legacy method.

### The kotlinx-metadata-jvm library is Stable

In Kotlin 2.0.0, the `kotlinx-metadata-jvm` library became [Stable](https://kotlinlang.org/docs/components-stability.html#stability-levels-explained). Now that the library has changed to the `kotlin` package and coordinates, you can find it as `kotlin-metadata-jvm` (without the "x").

Previously, the `kotlinx-metadata-jvm` library had its own publishing scheme and version. Now, we will build and publish the `kotlin-metadata-jvm` updates as part of the Kotlin release cycle, with the same backward compatibility guarantees as the Kotlin standard library.

The `kotlin-metadata-jvm` library provides an API to read and modify metadata of binary files generated by the Kotlin/JVM compiler.
