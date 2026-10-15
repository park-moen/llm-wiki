# Kotlin language specification — Scopes and identifiers

> Source: https://kotlinlang.org/spec/scopes-and-identifiers.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named chapter; global table of contents omitted.
> Collection note: The source header identifies this specification as Version 1.9-rfc+0.1; later compiler behavior must be checked against current documentation and release notes.

## Scopes and identifiers

Kotlin program is logically divided into *scopes*.

A scope is a syntactically-delimited region of code which constitutes a context in which entities and their names can be introduced. Scopes can be nested, with entities introduced in outer scopes possibly available in the inner scopes if [linked](https://kotlinlang.org/spec/scopes-and-identifiers.html#linked-scopes).

The top level of a Kotlin file is also a scope, containing all the scopes within a file.

All the scopes are divided into two kinds: declaration scopes and statement scopes. These two kinds of scopes differ in how the identifiers may refer to the entities defined in the scopes.

Declaration scopes include:

- The project [modules](https://kotlinlang.org/spec/packages-and-imports.html#modules);
- The project [packages](https://kotlinlang.org/spec/packages-and-imports.html#packages-and-imports);
- The top level scopes of non-script Kotlin files;
- The bodies of [classifier declarations](https://kotlinlang.org/spec/declarations.html#classifier-declaration);
- The bodies of [object literals](https://kotlinlang.org/spec/expressions.html#object-literals);
- Function parameter scope containing the declared value parameters in a [function declaration](https://kotlinlang.org/spec/declarations.html#function-declaration) or a non-primary [constructor declaration](https://kotlinlang.org/spec/declarations.html#constructor-declaration);
- Primary constructor parameter scope containing the declared value parameters in a [primary constructor](https://kotlinlang.org/spec/declarations.html#constructor-declaration).

Statement scopes include:

- The top level scopes of script Kotlin files;
- Scopes produced by control structure bodies of different [expressions](https://kotlinlang.org/spec/control--and-data-flow-analysis.html#expressions-cdfa);
- The bodies of [function declarations](https://kotlinlang.org/spec/declarations.html#function-declaration);
- The bodies of [function literals](https://kotlinlang.org/spec/expressions.html#function-literals);
- The bodies of getters and setters of [properties](https://kotlinlang.org/spec/declarations.html#property-declaration);
- The bodies of [constructors](https://kotlinlang.org/spec/declarations.html#constructor-declaration);
- The bodies of instance initialization blocks in [classifier declarations](https://kotlinlang.org/spec/declarations.html#classifier-declaration);
- Special initialization scope for a body of a [classifier declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration).

All declarations in a particular scope introduce new *bindings* of identifiers in this scope to their respective entities in the program. These entities may be types or values, where values refer to objects, functions or properties (which may also be delegated). Top-level scopes additionally allow to introduce new bindings using [`import` directives](https://kotlinlang.org/spec/packages-and-imports.html#packages-and-imports) from other top-level scopes.

In most situations, it is not allowed to bind several values to the same identifier in the same scope, but it is allowed to bind a value to an identifier already available in the scope through linked scopes or imports.

An exception to this rule are [function declarations](https://kotlinlang.org/spec/declarations.html#function-declaration), which are matched by [signatures](https://kotlinlang.org/spec/declarations.html#function-signature) and allow defining several functions with the same name in the same scope. When [calling functions](https://kotlinlang.org/spec/expressions.html#call-and-property-access-expressions), a process called [overload resolution](https://kotlinlang.org/spec/overload-resolution.html#overload-resolution) allows for differentiating between such functions. Overload resolution also applies to properties if they are used as functions through `invoke`-convention, but it does not allow defining several properties with the same name and with the same receivers in the same scope.

> Note: platforms may introduce additional restrictions on which identifiers may be declared together in the same or linked scopes.

The main difference between declaration scopes and statement scopes is that names in the statement scope are bound in the order of appearance. It is not allowed to access a value through an identifier in code which (syntactically) precedes the binding itself. On the contrary, in declaration scopes it is fully allowed, although initialization cycles may occur leading to unspecified behaviour.

> Note: Kotlin compiler may attempt to detect and report such initialization cycles as compile-time warnings or errors.

It also means that statement scopes nested inside declaration scopes may access values declared afterwards in parent declaration scopes, but any values declared inside a statement scope can be accessed only after their declaration point.

Examples:

- In declaration scope:

  ```
  // x refers to the property defined below 
  //   even if there is another property
  //   called x in outer scope or imported
  fun foo() = x + 2
  val x = 3
  ```
- In statement scope:

  ```
  // x refers to another property 
  //   defined in outer scope or imported
  //   or is a compile-time error
  fun foo() = x + 2
  val x = 3
  ```

### Linked scopes

Scopes `A` and `B` in a Kotlin program may be *downwards-linked* (`A ~> B`), meaning identifiers from `A` can be used in `B` without the need for additional qualification. If scopes `A` and `B` are downwards-linked, scopes `B` and `A` are considered *upwards-linked* (`B <~ A`).

> Note: link relation is transitive, unless specified otherwise.

Scopes are downwards-linked (DLD) or upwards-linked (ULD) as follows:

- A statement scope is DLD to any directly nested scope;
- An [object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is DLD to any nested scopes;
- An [object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is non-transitively ULD to the companion object scopes of its superclasses;
- An [object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is non-transitively ULD to the companion object scopes of its parent classifier superclasses;
- An [object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is ULD to the companion object declaration scope of its parent classifier;
- A [companion object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is DLD to any nested scopes;
- A [companion object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is non-transitively ULD to the companion object scopes of its superclasses;
- A [companion object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is non-transitively ULD to the companion object scopes of its parent classifier superclasses;
- A [companion object declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is ULD to the companion object declaration scope of the *parent* of its parent classifier;
- A [classifier or nested class declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is DLD to any nested statement scopes;
- A [classifier or nested class declaration](https://kotlinlang.org/spec/declarations.html#classifier-declaration) scope is ULD to its companion object declaration scope;
- An [inner class declaration](https://kotlinlang.org/spec/declarations.html#nested-and-inner-classifiers) scope is DLD to any nested statement scopes;
- An [inner class declaration](https://kotlinlang.org/spec/declarations.html#nested-and-inner-classifiers) scope is ULD to the classifier declaration scope of its parent classifier;
- A function or non-primary constructor parameter scope is ULD to the scope containing the function declaration and DLD to the function body;
- A primary constructor parameter scope is ULD to the scope containing the classifier declaration (but not the classifier declaration scope itself) and DLD to the classifier initialization scope;
- The instance initialization blocks are ULD to the classifier initialization scope.

> Important: linked scopes **do not** cover cases when identifiers from supertypes are used in subtypes, as this is covered by the [inheritance](https://kotlinlang.org/spec/inheritance.html#inheritance) rules.

### Identifiers and paths

Kotlin program operates with different *entities*, such as classes, interfaces, values, etc. An entity can be referenced using its *path*: a sequence of identifiers which references this entity in a given [scope](https://kotlinlang.org/spec/scopes-and-identifiers.html#scopes-and-identifiers).

Kotlin supports two kinds of paths.

- Simple paths `P`, which consist of a single identifier
- Qualified paths `P.m`, which consist of a path `P` and a member identifier `m`

Besides identifiers which are introduced by the developer (e.g., via declaring classes or introducing variables), there are several predefined identifiers with special semantics.

- `this` – an identifier which references the default receiver available in the current scope, further details are available [here](https://kotlinlang.org/spec/expressions.html#this-expressions)
- `this@label` – an identifier which references the default receiver available in the selected scope, further details are available [here](https://kotlinlang.org/spec/expressions.html#this-expressions)
- `super<Klazz>` – an identifier which references the supertype `Klazz` available in the current scope, further details are available [here](https://kotlinlang.org/spec/expressions.html#super-forms)
- `super<Klazz>@label` – an identifier which references the supertype `Klazz` available in the selected scope, further details are available [here](https://kotlinlang.org/spec/expressions.html#super-forms)

### Labels

Labels are special syntactic marks which allow one to reference certain code fragments or elements. [Lambda expressions](https://kotlinlang.org/spec/expressions.html#lambda-literals) and [loop statements](https://kotlinlang.org/spec/statements.html#loop-statements) are allowed to be labeled, with label identifier associated with the corresponding entity.

> Note: in Kotlin version 1.3 and earlier, labels were allowed to be placed on any expression or statement.

Labels can be *redeclared*, meaning that the same label identifier may be reused with different parts of code (or even on the same expression/loop) several times. Labels are *scoped*, meaning that they are only available in the scope they were declared in.

Labels are used by certain expressions, such as [`break`](https://kotlinlang.org/spec/expressions.html#break-expressions), [`continue`](https://kotlinlang.org/spec/expressions.html#continue-expressions) and [`return`](https://kotlinlang.org/spec/expressions.html#return-expressions), to specify exactly what entity the expression corresponds to. Please refer to the corresponding sections for details.

When resolving labels (determining which label an expression refers to), the *closest* label with the matching identifier is chosen, i.e., a label in an innermost scope syntactically closest to the point of its use.
