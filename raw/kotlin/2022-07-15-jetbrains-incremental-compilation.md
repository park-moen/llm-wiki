# A New Approach to Incremental Compilation in Kotlin

> Source: https://blog.jetbrains.com/kotlin/2022/07/a-new-approach-to-incremental-compilation-in-kotlin/
> Collected: 2026-10-15
> Published: 2022-07-15
> Author: Alejandro Serrano Mena
> Scope: Article content; site navigation and unrelated recommendations omitted.

[News](https://blog.jetbrains.com/kotlin/category/news/)

# A New Approach to Incremental Compilation in Kotlin

![Andrey Uskov](https://secure.gravatar.com/avatar/a3cc1711cb89da3126a950e27a82196b?s=200&r=g)

[Andrey Uskov](https://blog.jetbrains.com/author/andrey-uskov-jetbrains-com)

Read this post in other languages:

- [한국어](https://blog.jetbrains.com/ko/kotlin/2022/08/a-new-approach-to-incremental-compilation-in-kotlin/)
,- [简体中文](https://blog.jetbrains.com/zh-hans/kotlin/2022/07/a-new-approach-to-incremental-compilation-in-kotlin/)

In Kotlin 1.7.0, we’ve reworked incremental compilation for project changes in cross-module dependencies. The new approach lifts previous limitations on incremental compilation. It’s now supported when changes are made inside dependent non-Kotlin modules, and it is compatible with the [Gradle build cache](https://docs.gradle.org/current/userguide/build_cache.html). Support for compilation avoidance has also been improved. All of these advancements decrease the number of necessary full-module and file recompilations, making the overall compilation time faster.

The new scheme for incremental compilation is currently [Experimental](https://kotlinlang.org/docs/components-stability.html) and supports only the JVM backend in the Gradle build system.

[TRY THE NEW APPROACH](https://blog.jetbrains.com/kotlin/2022/07/a-new-approach-to-incremental-compilation-in-kotlin/#how-to "TRY THE NEW APPROACH")

## Benchmarks

We expect you’ll see the most significant improvements from the new approach if you use the Gradle build cache or frequently make changes in non-Kotlin Gradle modules. Here are some benchmark results measured for the Kotlin project on the kotlin-gradle-plugin module:

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_3.png)

## How to enable it

To use this new approach to incremental compilation, set the following option in your `gradle.properties` file:

`kotlin.incremental.useClasspathSnapshot=true`

We think it’s critical for incremental compilation to be stable and reliable. That’s why we’d appreciate your reports about any issues or strange behavior you encounter when using this compilation scheme.

Sometimes problems with incremental compilation become visible several rounds after the failure occurs, so you may want to use [build reports](https://blog.jetbrains.com/kotlin/2022/06/introducing-kotlin-build-reports/) to track the history of changes and compilations. Doing so may also help you provide reproducible bug reports.

## Under the hood

### Compilation avoidance and incremental compilation

You can skip this section if you are familiar with the [secrets of fast compilation for Kotlin](https://blog.jetbrains.com/kotlin/2020/09/the-dark-secrets-of-fast-compilation-for-kotlin/), [incremental Java compilation in Gradle](https://docs.gradle.org/current/userguide/java_plugin.html#sec:incremental_compile), and [compilation avoidance in Gradle](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_compile_avoidance).

One of the central aspects of fast compilation is the Application Binary Interface (ABI). Two classes have the same ABI if they are interchangeable when used as a compilation classpath.

Take a look at these sample Java classes:

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_4.png)

These classes have identical ABIs. The `private1()` and `private2()` methods are both invisible from other classes during compilation. Method bodies also don’t affect the compilation of other classes. Thus, these versions of JavaClass are interchangeable during compilation.

Gradle can track changes in the Java ABI. That’s why the state of pure Java compilation tasks will remain up-to-date if changes in their dependencies don’t affect the ABI. This feature is known as compilation avoidance – it was [introduced in Gradle 3.4](https://blog.gradle.org/incremental-compiler-avoidance) and delivered a dramatic performance improvement.

Since the Kotlin ABI contains more information (for example, bodies of inline functions), we can’t rely on the ABI comparison currently implemented in Gradle. This makes it necessary to start the Kotlin compiler following every change in dependencies.

Another way to make compilation faster is to recompile only affected files. This concept is known as *incremental compilation*. Let’s say that the compilation classpath has some ABI changes. What is the best way to handle this? Usually, ABI changes in the classpath affect only a subset of files in the module. The Kotlin compiler saves dependencies between the classes being compiled. Thus, during the subsequent compilation, it’s possible to find and recompile only the classes impacted by changes in the ABI.

If the ABI of recompiled classes has also changed, it’s possible to find classes affected by new changes in the ABI and repeat the compilation. This operation is a bit more complicated. Some files or classes should always be compiled together (for example, multi-file classes or [sealed interfaces](https://youtrack.jetbrains.com/issue/KT-42433) and their inheritors). We should also track constants calculated at compile time, but this topic is beyond our current focus.

### Tracking changes in cross-module dependencies

To explain how we track ABI changes in cross-module dependencies, consider the following sample. Here, Module B depends on Module A. The first full build is invoked in Revision 1. After applying Revision 2, the compilation of Module B is invoked. This operation is repeated in Revision 3.

![](https://blog.jetbrains.com/wp-content/uploads/2025/09/BsDyB6Se5JKwE90wOiOqLc1MNYDNYbEtwr6JMNbHsYIZshNtRcM-QuASgsbf0QtWyHEPf0aWc2qptin4srDRJoTARfsV-zKMeqynwz0woRyT0IUHZF5Plo6jvwnVdTvO1hUwSINTuKhYSDfagb4.png)

### History files

First, let’s consider our current, default approach. The Kotlin compiler can save changes in the ABI and produce class files. These are the `build-history.bin` files that you may have noticed in your build directory.

In Revision 1, the following operations are performed:

- Module A is fully built, as there is no previous state.
- Information is saved into the history file for Module A.
- Module B is fully built, as there is no previous state.
- All dependencies between compiled classes are saved in Module B.

Of course, there are some other phases. For example, history files are saved for Module B, but we’re leaving these phases out for the sake of clarity.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_6.png)

Operations in Revision 2:

- Module A is incrementally built.
- The information that the ABI has no changes in Module A is saved into the history file of Module A. Note that method bodies do not affect the ABI.
- Changes in dependencies for Module B are collected. It’s possible that the ABI hasn’t changed, too
- The compilation task for Module B is finished, as there are no files to recompile.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_7.png)

Operations in Revision 3:

- Module A is incrementally built.
- Changes in `A.doA` are added to the corresponding history file.
- Changes in dependencies for module B are analyzed, which finds a changed method: `A.doA`.
- Class B is marked for recompilation, as stated in the internally stored dependency map.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_8.png)

**Benefits**

- It’s quite effective – there is no need to save the compilation classpath or to compare classpaths.

**Drawbacks**

- In Revision 2, Gradle didn’t handle the up-to-date state of inputs. We spent some time starting the Kotlin compiler.
- Making revisions relocatable is quite expensive. That’s why incremental compilation is not compatible with the Gradle build cache.
- This approach is not applicable if Module A doesn’t produce history files (in the case of an external library, for example).

If you use build reports, you may have already encountered these reasons for rebuilds: `DEP_CHANGE_HISTORY_IS_NOT_FOUND` and `DEP_CHANGE_HISTORY_NO_KNOWN_BUILDS`. They are related to these drawbacks.

### Tracking the compilation classpath

Now, our alternative approach requires storing the ABI of the compilation classpath on every call of the Kotlin compiler. This approach also requires comparing crosspaths on every compilation. These operations are quite heavy, and we need aggressive optimizations to achieve acceptable performance:

**Possible optimizations**

(a) Preserve only those parts of the classpath ABI that were actually used during compilation.

(b) Extract only the parts of the ABI that can be used by the compiler from the classpath.

(c) Produce the ABI on the producer side along with the class files.

(d) Cache extracted ABIs.

Some of these options are incompatible with each other. For example, options **(b)** and **(c)** could not be implemented at the same time. In some cases, the optimal approach depends on the features provided by the build system. It also strongly depends on the use case:

- If you add a dependency on a large library and use only one class in one module, it’s more efficient to use option **(b)** (left picture below).
- If you add a similar dependency in many modules and use almost all classes from the dependency in many of them, it’s more efficient to use option **(d)** and cache the calculated ABI of the common dependency.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_6-copy.png)

Our estimates on several open-source projects proved that the most efficient approach is to perform a single ABI extraction for every dependency and cache the execution result.

In our new approach, we use [Gradle artifact transformation](https://docs.gradle.org/current/userguide/artifact_transforms.html) for ABI extraction. This makes the result cacheable and fully relocatable. If you use a remote build cache, the heavy work of extracting ABI from library dependencies most likely won’t be performed on your machine. This artifact will be just downloaded.

Now look at how the three revisions from the previous sample are compiled in the new approach.

In Revision 1, a full build of both modules will be performed because there is no previous state. Nothing changes here. Unlike in the previous scheme, we also store a snapshot of the compilation classpath for Module B.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_7-copy.png)

In Revision 2, the Kotlin compiler produces a different bytecode for Module A, but artifact transformation yields the same result. Gradle marks all inputs of Module B as ‘UP-TO-DATE’. No additional operations are required. The build chain is interrupted, and we get the result faster.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_8-copy.png)

In Revision 3, Module A produces a different output, the artifact transformation produces a different ABI, and thus the compilation of Module B is triggered. During the compilation of Module B, the following steps are required:

- The previously-stored classpath snapshot is compared with the new one and a list of changed ABIs is produced. The only difference is the `A.doA` method.
- `class B` is marked for recompilation, as stated in the internally stored dependency map.
- A snapshot of the compilation classpath for Module B is stored.
- `class B` in Module B is recompiled, as its types depend on the changed `A.doA`.

![](https://blog.jetbrains.com/wp-content/uploads/2022/07/A-New-Approach-to-Incremental-Compilation-in-Kotlin-EN-2_6-copy-2.png)

## Next steps

We are going to stabilize this approach, and we plan to implement support for other backends (JS, for instance) and build systems.

## Leave your feedback

You are welcome to try new incremental compilation in your projects. If you have any feedback or encounter any issues, please report them in our issue [tracker](https://youtrack.jetbrains.com/issues/KT). Thank you!

## Words of appreciation

We are very grateful to our external contributors for their tremendous help: [Ivan Gavrilovic](https://github.com/gavra0), [Hung Nguyen](https://github.com/hungvietnguyen), [Cédric Champeau](https://github.com/melix), and others.

- Share
- [*Facebook*](https://www.facebook.com/sharer.php?u=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2022%2F07%2Fa-new-approach-to-incremental-compilation-in-kotlin%2F)
- [Twitter](https://twitter.com/intent/tweet?source=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2022%2F07%2Fa-new-approach-to-incremental-compilation-in-kotlin%2F&text=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2022%2F07%2Fa-new-approach-to-incremental-compilation-in-kotlin%2F&via=kotlin)
- [*Linkedin*](http://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2022%2F07%2Fa-new-approach-to-incremental-compilation-in-kotlin%2F)

[*Prev post* Kotlin News: KotlinConf, Build Reports, DataFrame Preview, and More](https://blog.jetbrains.com/kotlin/2022/07/kotlin-news-june/)[KotlinConf returns to Amsterdam in April 2023! *Next post*](https://blog.jetbrains.com/kotlin/2022/07/kotlinconf-2023-2/)

#### Subscribe to Kotlin Blog updates

Subscribe form

By submitting this form, I agree to the JetBrains [Privacy Policy *Notification icon*](https://www.jetbrains.com/company/privacy.html) 

By submitting this form, I agree that JetBrains s.r.o. ("JetBrains") may use my name, email address, and location data to send me newsletters, including commercial communications, and to process my personal data for this purpose. I agree that JetBrains may process said data using [third-party](https://www.jetbrains.com/legal/privacy/third-parties.html) services for this purpose in accordance with the [JetBrains Privacy Policy](https://www.jetbrains.com/company/privacy.html). I understand that I can revoke this consent at any time in [my profile](https://account.jetbrains.com/profile-details/privacy). In addition, an unsubscribe link is included in each email.

Submit

Thanks, we've got you!
