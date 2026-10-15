# What's new in Kotlin 2.2.0 — JVM excerpts

> Source: https://kotlinlang.org/docs/whatsnew22.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Selected complete sections: kotlin-jvm. Other platform and tooling sections omitted.

## Kotlin/JVM

Kotlin 2.2.0 brings many updates to the JVM. The compiler now supports Java 24 bytecode and introduces changes to default method generation for interface functions. The release also simplifies working with annotations in Kotlin metadata, improves Java interop with inline value classes, and includes better support for annotating JVM records.

### Changes to default method generation for interface functions

Starting from Kotlin 2.2.0, functions declared in interfaces are compiled to JVM default methods unless configured otherwise. This change affects how Kotlin's interface functions with implementations are compiled to bytecode.

This behavior is controlled by the new stable compiler option `-jvm-default`, replacing the deprecated `-Xjvm-default` option.

You can control the behavior of the `-jvm-default` option using the following values:

- `enable` (default): generates default implementations in interfaces and includes bridge functions in subclasses and `DefaultImpls` classes. Use this mode to maintain binary compatibility with older Kotlin versions.
- `no-compatibility`: generates only default implementations in interfaces. This mode skips compatibility bridges and `DefaultImpls` classes, making it suitable for new code.
- `disable`: disables default implementations in interfaces. Only bridge functions and `DefaultImpls` classes are generated, matching the behavior before Kotlin 2.2.0.

To configure the `-jvm-default` compiler option, set the `jvmDefault` property in your Gradle Kotlin DSL:

// build.gradle.kts
kotlin {
compilerOptions {
jvmDefault = JvmDefaultMode.NO_COMPATIBILITY
}
}

### Support for reading and writing annotations in Kotlin metadata

Previously, you had to read annotations from compiled JVM class files using reflection or bytecode analysis and manually match them to metadata entries based on signatures. This process was error-prone, especially for overloaded functions.

Now, in Kotlin 2.2.0, the [Kotlin Metadata JVM library](https://kotlinlang.org/docs/metadata-jvm.html) introduces support for reading annotations stored in Kotlin metadata.

To make annotations available in the metadata for your compiled files, add the following compiler option:

-Xannotations-in-metadata

Alternatively, add it to the `compilerOptions {}` block of your Gradle build file:

// build.gradle.kts
kotlin {
compilerOptions {
freeCompilerArgs.add("-Xannotations-in-metadata")
}
}

With this option enabled, the Kotlin compiler writes annotations into metadata alongside the JVM bytecode, making them accessible to the `kotlin-metadata-jvm` library.

The library provides the following APIs for accessing annotations:

- `KmClass.annotations`
- `KmFunction.annotations`
- `KmProperty.annotations`
- `KmConstructor.annotations`
- `KmPropertyAccessorAttributes.annotations`
- `KmValueParameter.annotations`
- `KmFunction.extensionReceiverAnnotations`
- `KmProperty.extensionReceiverAnnotations`
- `KmProperty.backingFieldAnnotations`
- `KmProperty.delegateFieldAnnotations`
- `KmEnumEntry.annotations`

These APIs are [Experimental](https://kotlinlang.org/docs/components-stability.html#stability-levels-explained). To opt in, use the `@OptIn(ExperimentalAnnotationsInMetadata::class)` annotation.

Here's an example of reading annotations from Kotlin metadata:

@file:OptIn(ExperimentalAnnotationsInMetadata::class)
import kotlin.metadata.ExperimentalAnnotationsInMetadata
import kotlin.metadata.jvm.KotlinClassMetadata
annotation class Label(val value: String)
@Label("Message class")
class Message
fun main() {
val metadata = Message::class.java.getAnnotation(Metadata::class.java)
val kmClass = (KotlinClassMetadata.readStrict(metadata) as KotlinClassMetadata.Class).kmClass
println(kmClass.annotations)
// [@Label(value = StringValue("Message class"))]
}

If you use the `kotlin-metadata-jvm` library in your projects, we recommend testing and updating your code to support annotations. Otherwise, when annotations in metadata become [enabled by default](https://youtrack.jetbrains.com/issue/KT-75736) in a future Kotlin version, your projects may produce invalid or incomplete metadata.

If you experience any problems, please report them in our [issue tracker](https://youtrack.jetbrains.com/issue/KT-31857).

### Improved Java interop with inline value classes

Kotlin 2.2.0 introduces a new experimental annotation: [`@JvmExposeBoxed`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.jvm/-jvm-expose-boxed/). This annotation makes it easier to consume [inline value classes](https://kotlinlang.org/docs/inline-classes.html) from Java.

You can find an overview of this feature in this video:

By default, Kotlin compiles inline value classes to use unboxed representations, which are more performant but often hard or even impossible to use from Java. For example:

@JvmInline value class PositiveInt(val number: Int) {
init { require(number >= 0) }
}

In this case, because the class is unboxed, there is no constructor available for Java to call. There's also no way for Java to trigger the `init` block to ensure that `number` is positive.

When you annotate the class with `@JvmExposeBoxed`, Kotlin generates a public constructor that Java can call directly, ensuring that the `init` block also runs.

You can apply the `@JvmExposeBoxed` annotation at the class, constructor, or function level to gain fine-grained control over what's exposed to Java.

For example, in the following code, the extension function `.timesTwoBoxed()` is not accessible from Java:

@JvmInline
value class MyInt(val value: Int)
fun MyInt.timesTwoBoxed(): MyInt = MyInt(this.value \* 2)

To make it possible to create an instance of the `MyInt` class and call the `.timesTwoBoxed()` function from Java code, add the `@JvmExposeBoxed` annotation to both the class and the function:

@JvmExposeBoxed
@JvmInline
value class MyInt(val value: Int)
@JvmExposeBoxed
fun MyInt.timesTwoBoxed(): MyInt = MyInt(this.value \* 2)

With these annotations, the Kotlin compiler generates a Java-accessible constructor for the `MyInt` class. It also generates an overload for the extension function that uses the boxed form of the value class. As a result, the following Java code runs successfully:

MyInt input = new MyInt(5);
MyInt output = ExampleKt.timesTwoBoxed(input);

If you don't want to annotate every part of the inline value classes that you want to expose, you can effectively apply the annotation to a whole module. To apply this behavior to a module, compile it with the `-Xjvm-expose-boxed` option. Compiling with this option has the same effect as if every declaration in the module had the `@JvmExposeBoxed` annotation.

This new annotation does not change how Kotlin compiles or uses value classes internally, and all existing compiled code remains valid. It simply adds new capabilities to improve Java interoperability. The performance of Kotlin code using value classes is not impacted.

The `@JvmExposeBoxed` annotation is useful for library authors who want to expose boxed variants of member functions and receive boxed return types. It eliminates the need to choose between an inline value class (efficient but Kotlin-only) and a data class (Java-compatible but always boxed).

For a more detailed explanation of how the `@JvmExposedBoxed` annotation works and the problems it solves, see this [KEEP](https://github.com/Kotlin/KEEP/blob/jvm-expose-boxed/proposals/jvm-expose-boxed.md) proposal.

### Improved support for annotating JVM records

Kotlin has supported [JVM records](https://kotlinlang.org/docs/jvm-records.html) since Kotlin 1.5.0. Now, Kotlin 2.2.0 improves how Kotlin handles annotations on record components, particularly in relation to Java's [`RECORD_COMPONENT`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/annotation/ElementType.html#RECORD_COMPONENT) target.

Firstly, if you want to use a `RECORD_COMPONENT` as an annotation target, you need to manually add annotations for Kotlin (`@Target`) and Java. This is because Kotlin's `@Target` annotation doesn't support `RECORD_COMPONENT`. For example:

@Target(AnnotationTarget.CLASS, AnnotationTarget.PROPERTY)
@java.lang.annotation.Target(ElementType.CLASS, ElementType.RECORD_COMPONENT)
annotation class exampleClass

Maintaining both lists manually can be error-prone, so Kotlin 2.2.0 introduces a compiler warning if the Kotlin and Java targets don't match. For instance, if you omit `ElementType.CLASS` in the Java target list, the compiler reports:

Incompatible annotation targets: Java target 'CLASS' missing, corresponding to Kotlin targets 'CLASS'.

Secondly, Kotlin's behavior differs from Java when it comes to propagating annotations in records. In Java, annotations on a record component automatically apply to the backing field, getter, and constructor parameter. Kotlin doesn't do this by default, but you can now replicate the behavior using the [`@all:` use-site target](https://kotlinlang.org/docs/whatsnew22.html#all-meta-target-for-properties).

For example:

@JvmRecord
data class Person(val name: String, @all:Positive val age: Int)

When you use `@JvmRecord` with `@all:`, Kotlin now:

- Propagates the annotation to the property, backing field, constructor parameter, and getter.
- Applies the annotation to the record component, as well, if the annotation supports Java's `RECORD_COMPONENT`.
