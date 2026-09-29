# Kotlin — No-arg compiler plugin

> Source: https://kotlinlang.org/docs/no-arg-plugin.html
> Collected: 2026-09-29
> Published: 2026-08-12
> Scope: Selected passages from No-arg compiler plugin and JPA support.

The no-arg compiler plugin generates an additional zero-argument constructor for classes with a specific annotation.

The generated constructor is synthetic, so it can't be directly called from Java or Kotlin, but it can be called using reflection.

This allows the Java Persistence API (JPA) to instantiate a class, although it doesn't have the zero-parameter constructor from Kotlin or Java point of view (see the description of `kotlin-jpa` plugin below).

## JPA support

As with the `kotlin-spring` plugin wrapped on top of `all-open`, `kotlin-jpa` is wrapped on top of `no-arg`. The plugin specifies `@Entity`, `@Embeddable`, and `@MappedSuperclass` no-arg annotations automatically.

Add the plugin using the Gradle plugins DSL:

```kotlin
plugins { kotlin("plugin.jpa") version "2.4.20" }
```
