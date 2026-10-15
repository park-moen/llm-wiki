# Kotlin language specification — Type system

> Source: https://kotlinlang.org/spec/type-system.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named chapter; global table of contents omitted.
> Collection note: The source header identifies this specification as Version 1.9-rfc+0.1; later compiler behavior must be checked against current documentation and release notes.

## Type system

### Glossary

TTT
:   Type (with unknown nullability)

T!!T!!T!!
:   [Non-nullable type](https://kotlinlang.org/spec/type-system.html#nullable-types)

T?T?T?
:   [Nullable type](https://kotlinlang.org/spec/type-system.html#nullable-types)

{T}\{T\}{T}
:   Universe of all possible types

{T!!}\{T!!\}{T!!}
:   Universe of non-nullable types

{T?}\{T?\}{T?}
:   Universe of nullable types

Well-formed type
:   A properly constructed type w.r.t. Kotlin type system

Γ\GammaΓ
:   Type context

A<:BA <: BA<:B
:   A is a subtype of B

A<:>⁡BA \mathrel{\operatorname{\cancel{<:>}}}BA<:>​B
:   A and B are not related w.r.t. subtyping

Type constructor
:   An abstract type with one or more type parameters, which must be instantiated before use

Parameterized type
:   A concrete type, which is the result of type constructor instantiation

Type parameter
:   Formal type parameter of a type constructor

Type argument
:   Actual type argument in a parameterized type

T[A1,…,An]T\lbrack A_1, \ldots, A_n\rbrackT[A1​,…,An​]
:   The result of type constructor TTT
    instantiation with type arguments AiA_iAi​

T[σ]T\lbrack\sigma\rbrackT[σ]
:   The result of type constructor T(F1,…,Fn)T(F_1, \ldots, F_n)T(F1​,…,Fn​)
    instantiation with the assumed substitution σ:F1=A1,…,Fn=An\sigma : F_1 = A_1, \ldots, F_n = A_nσ:F1​=A1​,…,Fn​=An​

σT\sigma TσT
:   The result of type substitution in type TTT
    w.r.t. substitution σ\sigmaσ

KT(F,A)K_T(F, A)KT​(F,A)
:   Captured type from the [type capturing](https://kotlinlang.org/spec/type-system.html#type-capturing) of type parameter FFF
    and type argument AAA
    in parameterized type TTT

T⟨K1,…,Kn⟩T\langle K_1, \ldots, K_n\rangleT⟨K1​,…,Kn​⟩
:   The result of type capturing for parameterized type TTT
    with *captured* types KiK_iKi​

T⟨τ⟩T\langle \tau \rangleT⟨τ⟩
:   The result of type capturing for parameterized type T(F1,…,Fn)T(F_1, \ldots, F_n)T(F1​,…,Fn​)
    with *captured* substitution τ:F1=K1,…,Fn=Kn\tau : F_1 = K_1, \ldots, F_n = K_nτ:F1​=K1​,…,Fn​=Kn​

A&⁡BA \mathbin{\operatorname{\&}}BA&B
:   Intersection type of AAA
    and BBB

A∣⁡BA \mathbin{\operatorname{|}}BA∣B
:   Union type of AAA
    and BBB

GLB⁡\operatorname{\text{GLB}}GLB
:   Greatest lower bound

LUB⁡\operatorname{\text{LUB}}LUB
:   Least upper bound

### Introduction

Similarly to most other programming languages, Kotlin operates on data in the form of *values* or *objects*, which have *types* — descriptions of what is the expected behaviour and possible values for their datum. An empty value is represented by a special `null` object; most operations with it result in runtime [errors or exceptions](https://kotlinlang.org/spec/exceptions.html#exceptions).

Kotlin has a type system with the following main properties.

- Hybrid static, gradual and flow type checking;
- Null safety;
- No unsafe implicit conversions;
- Unified top and bottom types;
- Nominal subtyping with bounded parametric polymorphism and mixed-site variance.

Type safety (consistency between compile and runtime types) is verified *statically*, at compile time, for the majority of Kotlin types. However, for better interoperability with platform-dependent code Kotlin also support a variant of *gradual types* in the form of [flexible types](https://kotlinlang.org/spec/type-system.html#flexible-types). Even more so, in some cases the compile-time type of a value may *change* depending on the control- and data-flow of the program; a feature usually known as *flow typing*, represented in Kotlin as [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts).

Null safety is enforced by having two type universes: *nullable* (with nullable types T?T?T?
) and *non-nullable* (with non-nullable types T!!T!!T!!
). A value of any non-nullable type cannot contain `null`, meaning all operations within the non-nullable type universe are safe w.r.t. empty values, i.e., should never result in a runtime error caused by `null`.

Implicit conversions between types in Kotlin are limited to safe upcasts w.r.t. subtyping, meaning all other (unsafe) conversions must be explicit, done via either a conversion function or an [explicit cast](https://kotlinlang.org/spec/expressions.html#cast-expressions). However, Kotlin also supports smart casts — a special kind of implicit conversions which are safe w.r.t. program control- and data-flow, which are covered in more detail [here](https://kotlinlang.org/spec/type-inference.html#smart-casts).

The unified supertype type for all types in Kotlin is kotlin.Any?⁡\operatorname{\texttt{kotlin.Any?}}kotlin.Any?
, a [nullable](https://kotlinlang.org/spec/type-system.html#nullable-types) version of [kotlin.Any⁡\operatorname{\texttt{kotlin.Any}}kotlin.Any](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.any-builtins). The unified subtype type for all types in Kotlin is [kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.nothing-builtins).

Kotlin uses nominal subtyping, meaning subtyping relation is defined when a type is declared, with bounded parametric polymorphism, implemented as generics via [parameterized types](https://kotlinlang.org/spec/type-system.html#parameterized-classifier-types). Subtyping between these parameterized types is defined through [mixed-site variance](https://kotlinlang.org/spec/type-system.html#mixed-site-variance).

### Type kinds

For the purposes of this section, we establish the following type kinds — different flavours of types which exist in the Kotlin type system.

- [Built-in types](https://kotlinlang.org/spec/type-system.html#built-in-types)
- [Classifier types](https://kotlinlang.org/spec/type-system.html#classifier-types)
- [Type parameters](https://kotlinlang.org/spec/type-system.html#type-parameters)
- [Function types](https://kotlinlang.org/spec/type-system.html#function-types)
- [Array types](https://kotlinlang.org/spec/type-system.html#array-types)
- [Flexible types](https://kotlinlang.org/spec/type-system.html#flexible-types)
- [Nullable types](https://kotlinlang.org/spec/type-system.html#nullable-types)
- [Intersection types](https://kotlinlang.org/spec/type-system.html#intersection-types)
- [Union types](https://kotlinlang.org/spec/type-system.html#union-types)

We distinguish between *concrete* and *abstract* types. Concrete types are types which are assignable to values. Abstract types need to be instantiated as concrete types before they can be used as types for values.

> Note: for brevity, we omit specifying that a type is concrete. All types not described as abstract are implicitly concrete.

We further distinguish *concrete* types between *class* and *interface* types; as Kotlin is a language with single inheritance, sometimes it is important to discriminate between these kinds of types. Any given concrete type may be either a class or an interface type, but never both.

We also distinguish between *denotable* and *non-denotable* types. The former are types which are expressible in Kotlin and can be written by the end-user. The latter are special types which are *not* expressible in Kotlin and are used by the compiler in [type inference](https://kotlinlang.org/spec/type-inference.html#type-inference), [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts), etc.

#### Built-in types

Kotlin type system uses the following built-in types, which have special semantics and representation (or lack thereof).

##### `kotlin.Any`

kotlin.Any⁡\operatorname{\texttt{kotlin.Any}}kotlin.Any
is the unified [supertype](https://kotlinlang.org/spec/type-system.html#subtyping) (⊤\top⊤
) for {T!!}\{T!!\}{T!!}
, i.e., all non-nullable types are subtypes of kotlin.Any⁡\operatorname{\texttt{kotlin.Any}}kotlin.Any
, either explicitly, implicitly, or by [subtyping relation](https://kotlinlang.org/spec/type-system.html#subtyping).

> Note: additional details about kotlin.Any⁡\operatorname{\texttt{kotlin.Any}}kotlin.Any
> are available [here](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.any-builtins).

##### `kotlin.Nothing`

kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing
is the unified [subtype](https://kotlinlang.org/spec/type-system.html#subtyping) (⊥\bot⊥
) for {T}\{T\}{T}
, i.e., kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing
is a subtype of all well-formed Kotlin types, including user-defined ones. This makes it an uninhabited type (as it is impossible for anything to be, for example, a function and an integer at the same time), meaning instances of this type can never exist at runtime; subsequently, there is no way to create an instance of kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing
in Kotlin.

> Note: additional details about kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing
> are available [here](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.nothing-builtins).

##### `kotlin.Function`

kotlin.Function⁡(R)\operatorname{\texttt{kotlin.Function}}(R)kotlin.Function(R)
is the unified supertype of all [function types](https://kotlinlang.org/spec/type-system.html#function-types). It is parameterized over function return type `R`.

##### Built-in integer types

Kotlin supports the following signed integer types.

- `kotlin.Int`
- `kotlin.Short`
- `kotlin.Byte`
- `kotlin.Long`

Besides their use as types, integer types are important w.r.t. [integer literal types](https://kotlinlang.org/spec/type-system.html#integer-literal-types).

> Note: additional details about built-in integer types are available [here](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#built-in-integer-types-builtins).

##### Array types

Kotlin arrays are represented as a [parameterized type](https://kotlinlang.org/spec/type-system.html#parameterized-classifier-types) kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
, where TTT
is the type of the stored elements, which supports `get`/`set` operations. The kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
type follows the rules of regular type constructors and parameterized types w.r.t. subtyping.

> Note: unlike Java, arrays in Kotlin are declared as invariant. To use them in a co- or contravariant way, one should use [use-site variance](https://kotlinlang.org/spec/type-system.html#use-site-variance).

In addition to the general kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
type, Kotlin also has the following specialized array types:

- `DoubleArray` (for kotlin.Array⁡(kotlin.Double⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Double}})kotlin.Array(kotlin.Double)
  )
- `FloatArray` (for kotlin.Array⁡(kotlin.Float⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Float}})kotlin.Array(kotlin.Float)
  )
- `LongArray` (for kotlin.Array⁡(kotlin.Long⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Long}})kotlin.Array(kotlin.Long)
  )
- `IntArray` (for kotlin.Array⁡(kotlin.Int⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Int}})kotlin.Array(kotlin.Int)
  )
- `ShortArray` (for kotlin.Array⁡(kotlin.Short⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Short}})kotlin.Array(kotlin.Short)
  )
- `ByteArray` (for kotlin.Array⁡(kotlin.Byte⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Byte}})kotlin.Array(kotlin.Byte)
  )
- `CharArray` (for kotlin.Array⁡(kotlin.Char⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Char}})kotlin.Array(kotlin.Char)
  )
- `BooleanArray` (for kotlin.Array⁡(kotlin.Boolean⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Boolean}})kotlin.Array(kotlin.Boolean)
  )

These array types structurally match the corresponding kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
type; i.e., `IntArray` has the same methods and properties as kotlin.Array⁡(kotlin.Int⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Int}})kotlin.Array(kotlin.Int)
. However, they are **not** related by subtyping; meaning one cannot pass a `BooleanArray` argument to a function expecting an kotlin.Array⁡(kotlin.Boolean⁡)\operatorname{\texttt{kotlin.Array}}(\operatorname{\texttt{kotlin.Boolean}})kotlin.Array(kotlin.Boolean)
.

> Note: the presence of such specialized types allows the compiler to perform additional array-related optimizations.

> Note: specialized and non-specialized array types match modulo their iterator types, which are also specialized; `Iterator<Int>` is specialized to `IntIterator`.

*Array type specialization* ATS⁡(A)\operatorname{\text{ATS}}(A)ATS(A)
is a transformation of a generic kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
type to a corresponding specialized version, which works as follows.

- if kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
  has a specialized version `TArray`, ATS⁡(kotlin.Array⁡(T))=TArray\operatorname{\text{ATS}}(\operatorname{\texttt{kotlin.Array}}(T)) = \texttt{TArray}ATS(kotlin.Array(T))=TArray
- if kotlin.Array⁡(T)\operatorname{\texttt{kotlin.Array}}(T)kotlin.Array(T)
  does not have a specialized version, ATS⁡(kotlin.Array⁡(T))=kotlin.Array⁡(T)\operatorname{\text{ATS}}(\operatorname{\texttt{kotlin.Array}}(T)) = \operatorname{\texttt{kotlin.Array}}(T)ATS(kotlin.Array(T))=kotlin.Array(T)

ATS⁡\operatorname{\text{ATS}}ATS
takes an important part in how [variable length parameters](https://kotlinlang.org/spec/declarations.html#variable-length-parameters) are handled.

> Note: additional details about built-in array types are available [here](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#built-in-array-types).

#### Classifier types

Classifier types represent regular types which are declared as [classes](https://kotlinlang.org/spec/declarations.html#class-declaration), [interfaces](https://kotlinlang.org/spec/declarations.html#interface-declaration) or [objects](https://kotlinlang.org/spec/declarations.html#object-declaration). As Kotlin supports parametric polymorphism, there are two variants of classifier types: simple and parameterized.

##### Simple classifier types

A simple classifier type

T:S1,…,SmT : S_1, \ldots, S_mT:S1​,…,Sm​

consists of

- type name TTT
- (optional) list of supertypes S1,…,SmS_1, \ldots, S_mS1​,…,Sm​

To represent a well-formed simple classifier type, T:S1,…,SmT : S_1, \ldots, S_mT:S1​,…,Sm​
should satisfy the following conditions.

- TTT
  is a valid type name
- ∀i∈[1,m]:Si\forall i \in [1,m]: S_i∀i∈[1,m]:Si​
  must be concrete, [non-nullable](https://kotlinlang.org/spec/type-system.html#nullable-types), well-formed type
- the transitive closure S∗(T)\mathbb{S}^\*(T)S∗(T)
  of the set of type supertypes S(T:S1,…,Sm)={S1,…,Sm}∪S(S1)∪…∪S(Sm)\mathbb{S}(T : S_1, \ldots, S_m) = \{S_1, \ldots, S_m\} \cup \mathbb{S}(S_1) \cup \ldots \cup \mathbb{S}(S_m)S(T:S1​,…,Sm​)={S1​,…,Sm​}∪S(S1​)∪…∪S(Sm​)
  is *consistent*, i.e., does not contain two [parameterized types](https://kotlinlang.org/spec/type-system.html#parameterized-classifier-types) with different type arguments.

> Example:
>
> ```
> // A well-formed type with no supertypes
> interface Base
>
> // A well-formed type with a single supertype Base
> interface Derived : Base
>
> // An ill-formed type,
> // as nullable type cannot be a supertype
> interface Invalid : Base?
> ```

> Note: for the purpose of different type system examples, we assume the presence of the following well-formed concrete types:
>
> - class `String`
> - interface `Number`
> - class `Int` <:<:<:
>   `Number`
> - class `Double` <:<:<:
>   `Number`

> Note: `Number` is actually a built-in abstract class; we use it as an interface for illustrative purposes.

##### Parameterized classifier types

A classifier type constructor

T(F1,…,Fn):S1,…,SmT(F_1, \ldots, F_n) : S_1, \ldots, S_mT(F1​,…,Fn​):S1​,…,Sm​

describes an abstract type and consists of

- type name TTT
- type parameters F1,…,FnF_1, \ldots, F_nF1​,…,Fn​
- (optional) list of supertypes S1,…,SmS_1, \ldots, S_mS1​,…,Sm​

To represent a well-formed type constructor, T(F1,…,Fn):S1,…,SmT(F_1, \ldots, F_n) : S_1, \ldots, S_mT(F1​,…,Fn​):S1​,…,Sm​
should satisfy the following conditions.

- TTT
  is a valid type name
- ∀i∈[1,n]:Fi\forall i \in [1,n]: F_i∀i∈[1,n]:Fi​
  must be well-formed [type parameter](https://kotlinlang.org/spec/type-system.html#type-parameters)
- ∀j∈[1,m]:Sj\forall j \in [1,m]: S_j∀j∈[1,m]:Sj​
  must be concrete, [non-nullable](https://kotlinlang.org/spec/type-system.html#nullable-types), well-formed type

To instantiate a type constructor, one provides it with type arguments, creating a concrete parameterized classifier type

T[A1,…,An]T[A_1, \ldots, A_n]T[A1​,…,An​]

which consists of

- type constructor TTT
- type arguments A1,…,AnA_1, \ldots, A_nA1​,…,An​

To represent a well-formed parameterized type, T[A1,…,An]T[A_1, \ldots, A_n]T[A1​,…,An​]
should satisfy the following conditions.

- TTT
  is a well-formed type constructor with nnn
  type parameters
- ∀i∈[1,n]:Ai\forall i \in [1,n]: A_i∀i∈[1,n]:Ai​
  must be well-formed concrete type
- ∀i∈[1,n]:\forall i \in [1,n]:∀i∈[1,n]:
  variance of AiA_iAi​
  does not [contradict](https://kotlinlang.org/spec/type-system.html#use-site-variance) variance of FiF_iFi​
- ∀i∈[1,n]:Ai<:τUi\forall i \in [1,n]: A_i <: \tau U_i∀i∈[1,n]:Ai​<:τUi​
  , where UiU_iUi​
  is the upper bound for FiF_iFi​
  and captured substitution τ:F1=K1,…,Fn=Kn\tau : F_1 = K_1, \ldots, F_n = K_nτ:F1​=K1​,…,Fn​=Kn​
  manipulates [captured types](https://kotlinlang.org/spec/type-system.html#type-capturing).
- the transitive closure S∗(T)\mathbb{S}^\*(T)S∗(T)
  of the set of type supertypes S(T⟨τ⟩:τS1,…,τSm)={τS1,…,τSm}∪S(τS1)∪…∪S(τSm)\mathbb{S}(T\langle \tau \rangle : \tau S_1, \ldots, \tau S_m) = \{\tau S_1, \ldots, \tau S_m\} \cup \mathbb{S}(\tau S_1) \cup \ldots \cup \mathbb{S}(\tau S_m)S(T⟨τ⟩:τS1​,…,τSm​)={τS1​,…,τSm​}∪S(τS1​)∪…∪S(τSm​)
  is *consistent*, i.e., does not contain two [parameterized types](https://kotlinlang.org/spec/type-system.html#parameterized-classifier-types) with different type arguments.

> Example:
>
> ```
> // A well-formed type constructor with no supertypes
> // A and B are unbounded type parameters
> interface Generic<A, B>
>
> // A well-formed type constructor
> //   with a single parameterized supertype
> // Int and String are well-formed concrete types
> interface ConcreteDerived<P, Q> : Generic<Int, String>
>
> // A well-formed type constructor
> //   with a single parameterized supertype
> // P and Q are type parameters of GenericDerived,
> //   used as type arguments of Generic
> interface GenericDerived<P, Q> : Generic<P, Q>
>
> // An ill-formed type constructor,
> //   as abstract type Generic
> //   cannot be used as a supertype
> interface Invalid<P> : Generic
>
>
> // A well-formed type constructor with no supertypes
> // out A is a projected type parameter
> interface Out<out A>
>
>
> // A well-formed type constructor with no supertypes
> // S : Number is a bounded type parameter
> // (S <: Number)
> interface NumberWrapper<S : Number>
>
> // A well-formed type constructor
> //   with a single parameterized supertype
> // NumberWrapper<Int> is well-formed,
> //   as Int <: Number
> interface IntWrapper : NumberWrapper<Int>
>
> // An ill-formed type constructor,
> //   as NumberWrapper<String> is an ill-formed parameterized type
> //   (String not(<:>) Number)
> interface InvalidWrapper : NumberWrapper<String>
> ```

#### Type parameters

Type parameters are a special kind of types, which are introduced by type constructors. They are considered well-formed concrete types only in the type context of their declaring type constructor.

When creating a parameterized type from a type constructor, its type parameters with their respective type arguments go through [capturing](https://kotlinlang.org/spec/type-system.html#type-capturing) and create *captured* types, which follow special rules described in more detail below.

Type parameters may be either unbounded or bounded. By default, a type parameter FFF
is unbounded, which is the same as saying it is a bounded type parameter of the form F<:kotlin.Any?⁡F <: \operatorname{\texttt{kotlin.Any?}}F<:kotlin.Any?
.

A bounded type parameter additionally specifies upper type bounds for the type parameter and is defined as F<:B1,…,BnF <: B_1, \ldots, B_nF<:B1​,…,Bn​
, where BiB_iBi​
is an i-th upper bound on type parameter FFF
.

To represent a well-formed bounded type parameter of type constructor TTT
, F<:B1,…,BnF <: B_1, \ldots, B_nF<:B1​,…,Bn​
should satisfy either of the following sets of conditions.

- Bounded type parameter with regular bounds:
  - FFF
    is a type parameter of type constructor TTT
  - ∀i∈[1,n]:Bi\forall i \in [1,n]: B_i∀i∈[1,n]:Bi​
    must be concrete, non-type-parameter, well-formed type
  - No more than one of BiB_iBi​
    may be a class type

> Note: the last condition is a nod to the single inheritance nature of Kotlin: any type may be a subtype of no more than one class type. For any two class types, either these types are in a subtyping relation (and you should use the more specific type in the bounded type parameter), or they are unrelated (and the bounded type parameter is empty).
>
> Actual support for multiple class type bounds would be needed only in very rare cases, such as the following example.
>
> ```
> interface Foo
> interface Bar
>
> open class A<T>
> class B<T> : A<T>
>
> class C<T> where T : A<out Foo>, T : B<out Bar>
> // A convoluted way of saying T <: B<out Foo & Bar>,
> // which contains a non-denotable intersection type
> ```

- Bounded type parameter with type parameter bound:
  - FFF
    is a type parameter of type constructor TTT
  - i=1i = 1i=1
    (i.e., there is a single upper bound)
  - B1B_1B1​
    must be well-formed [type parameter](https://kotlinlang.org/spec/type-system.html#type-parameters)

From the definition, it follows F<:B1,…,BnF <: B_1, \ldots, B_nF<:B1​,…,Bn​
can be represented as F<:UF <: UF<:U
where U=B1&⁡…&⁡BnU = B_1 \mathbin{\operatorname{\&}}\ldots \mathbin{\operatorname{\&}}B_nU=B1​&…&Bn​
(aka [intersection type](https://kotlinlang.org/spec/type-system.html#intersection-types)).

##### Function type parameters

Function type parameters are a flavor of type parameters, which are used in [function declarations](https://kotlinlang.org/spec/declarations.html#function-declaration) to create parameterized functions. They are considered well-formed concrete types only in the type context of their declaring function.

> Note: one may view such parameterized functions as a kind of function type constructors.

Function type parameters work similarly to regular type parameters, however, they do not support specifying [mixed-site variance](https://kotlinlang.org/spec/type-system.html#mixed-site-variance).

##### Mixed-site variance

To implement subtyping between parameterized types, Kotlin uses *mixed-site variance* — a combination of declaration- and use-site variance, which is easier to understand and reason about, compared to wildcards from Java. Mixed-site variance means you can specify, whether you want your parameterized type to be co-, contra- or invariant on some type parameter, both in type parameter (declaration-site) and type argument (use-site).

> Info: *variance* is a way of describing how [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping) works for *variant* parameterized types. With declaration-site variance, for two [non-equivalent](https://kotlinlang.org/spec/type-system.html#subtyping) types A<:BA <: BA<:B
> , subtyping between `T<A>` and `T<B>` depends on the variance of type parameter FFF
> for some type constructor TTT
> .
>
> - if FFF
>   is covariant (out ⁡F\operatorname{\texttt{out\,}}FoutF
>   ), `T<A>` <:<:<:
>   `T<B>`
> - if FFF
>   is contravariant(in ⁡F\operatorname{\texttt{in\,}}FinF
>   ), `T<A>` :>:>:>
>   `T<B>`
> - if FFF
>   is invariant (default), `T<A>` <:>⁡\mathrel{\operatorname{\cancel{<:>}}}<:>​
>   `T<B>`
>
> Use-site variance allows the user to change the type variance of an *invariant* type parameter by specifying it on the corresponding type argument. out ⁡A\operatorname{\texttt{out\,}}AoutA
> means covariant type argument, in ⁡A\operatorname{\texttt{in\,}}AinA
> means contravariant type argument; for two [non-equivalent](https://kotlinlang.org/spec/type-system.html#subtyping) types A<:BA <: BA<:B
> and an invariant type parameter FFF
> of some type constructor TTT
> , subtyping for use-site variance has the following rules.
>
> - `T<out A>` <:<:<:
>   `T<out B>`
> - `T<in A>` :>:>:>
>   `T<in B>`
> - `T<A>` <:<:<:
>   `T<out A>`
> - `T<A>` <:<:<:
>   `T<in A>`
>
> Important: by the transitivity of the subtyping operator these rules imply that the following also holds:
>
> - `T<A>` <:<:<:
>   `T<out B>`
> - `T<in A>` :>:>:>
>   `T<B>`

> Note: Kotlin does not support specifying both co- and contravariance at the same time, i.e., it is impossible to have `T<out A in B>` neither on declaration- nor on use-site.

> Note: informally, covariant type parameter out ⁡A\operatorname{\texttt{out\,}}AoutA
> of type constructor TTT
> means “TTT
> is a producer of AAA
> s and gets them out”; contravariant type parameter in ⁡A\operatorname{\texttt{in\,}}AinA
> of type constructor TTT
> means “TTT
> is a consumer of AAA
> s and takes them in”.

For further discussion about mixed-site variance and its practical applications, we readdress you to [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping).

##### Declaration-site variance

A type parameter FFF
may be invariant, covariant or contravariant.

By default, all type parameters are invariant.

To specify a covariant type parameter, it is marked as out ⁡F\operatorname{\texttt{out\,}}FoutF
. To specify a contravariant type parameter, it is marked as in ⁡F\operatorname{\texttt{in\,}}FinF
.

The variance information is used by [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping) and for checking allowed operations on values of co- and contravariant type parameters.

> Important: declaration-site variance can be used only when declaring types, e.g., [function type parameters](https://kotlinlang.org/spec/type-system.html#function-type-parameters) cannot be variant.

> Example:
>
> ```
> // A type constructor with an invariant type parameter
> interface Invariant<A>
> // A type constructor with a covariant type parameter
> interface Out<out A>
> // A type constructor with a contravariant type parameter
> interface In<in A>
>
> fun testInvariant() {
>     var invInt: Invariant<Int> = ...
>     var invNumber: Invariant<Number> = ...
>     
>     if (random) invInt = invNumber // ERROR
>     else invNumber = invInt // ERROR
>     
>     // Invariant type parameters do not create subtyping
> }
>
> fun testOut() {
>     var outInt: Out<Int> = ...
>     var outNumber: Out<Number> = ...
>     
>     if (random) outInt = outNumber // ERROR
>     else outNumber = outInt // OK
>     
>     // Covariant type parameters create "same-way" subtyping
>     //   Int <: Number => Out<Int> <: Out<Number>
>     // (more specific type Out<Int> can be assigned
>     //  to a less specific type Out<Number>)
> }
>
> fun testIn() {
>     var inInt: In<Int> = ...
>     var inNumber: In<Number> = ...
>     
>     if (random) inInt = inNumber // OK
>     else inNumber = inInt // ERROR
>     
>     // Contravariant type parameters create "opposite-way" subtyping
>     //   Int <: Number => In<Int> :> In<Number>
>     // (more specific type In<Number> can be assigned
>     //  to a less specific type In<Int>)
> }
> ```

##### Use-site variance

Kotlin also supports use-site variance, by specifying the variance for type arguments. Similarly to type parameters, one can have type arguments being co-, contra- or invariant.

> Important: use-site variance cannot be used when declaring a supertype top-level type argument.

By default, all type arguments are invariant.

To specify a covariant type argument, it is marked as out ⁡A\operatorname{\texttt{out\,}}AoutA
. To specify a contravariant type argument, it is marked as in ⁡A\operatorname{\texttt{in\,}}AinA
.

Kotlin prohibits contradictory combinations of declaration- and use-site variance as follows.

- It is a compile-time error to use a covariant type argument in a contravariant type parameter
- It is a compile-time error to use a contravariant type argument in a covariant type parameter

In case one cannot specify any well-formed type argument, but still needs to use a parameterized type in a type-safe way, they may use *bivariant* type argument ⋆\star⋆
, which is roughly equivalent to a combination of out ⁡kotlin.Any?⁡\operatorname{\texttt{out\,}}\operatorname{\texttt{kotlin.Any?}}outkotlin.Any?
and in ⁡kotlin.Nothing⁡\operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}inkotlin.Nothing
(for further details, see [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping)).

> Note: informally, T[⋆]T[\star]T[⋆]
> means “I can give out something very generic (kotlin.Any?⁡\operatorname{\texttt{kotlin.Any?}}kotlin.Any?
> ) and cannot take in anything”.

> Example:
>
> ```
> // A type constructor with an invariant type parameter
> interface Inv<A>
>
> fun test() {
>     var invInt: Inv<Int> = ...
>     var invNumber: Inv<Number> = ...
>     var outInt: Inv<out Int> = ...
>     var outNumber: Inv<out Number> = ...
>     var inInt: Inv<in Int> = ...
>     var inNumber: Inv<in Number> = ...
>     
>     when (random) {
>         1 -> {
>             inInt = invInt    // OK
>             // T<in Int> :> T<Int>
>             
>             inInt = invNumber // OK
>             // T<in Int> :> T<in Number> :> T<Number>
>         }
>         2 -> {
>             outNumber = invInt    // OK
>             // T<out Number> :> T<out Int> :> T<Int>
>             
>             outNumber = invNumber // OK
>             // T<out Number> :> T<Number>
>         }
>         3 -> {
>             invInt = inInt  // ERROR
>             invInt = outInt // ERROR
>             // It is invalid to assign less specific type
>             // to a more specific one
>             //   T<Int> <: T<in Int>
>             //   T<Int> <: T<out Int>
>         }
>         4 -> {
>             inInt = outInt    // ERROR
>             inInt = outNumber // ERROR
>             // types with co- and contravariant type parameters
>             // are not connected by subtyping
>             //   T<in Int> not(<:>) T<out Int>
>         }
>     }
> }
> ```

#### Type capturing

Type capturing (similarly to Java capture conversion) is used when instantiating type constructors; it creates *abstract captured* types based on the type information of both type parameters and arguments, which present a unified view on the resulting types and simplifies further reasoning.

The reasoning behind type capturing is closely related to variant parameterized types being a form of *bounded existential types*; e.g., `A<out T>` may be loosely considered as the following existential type: ∃X:X<:T.A⟨X⟩\exists X : X <: T . A\langle X\rangle∃X:X<:T.A⟨X⟩
. Informally, a bounded existential type describes a *set* of possible types, which satisfy its bound constraints. Before such a type can be used, it needs to be *opened* (or *unpacked*): existentially quantified type variables are lifted to fresh type variables with corresponding bounds. We call these type variables *captured* types.

For a given type constructor T(F1,…,Fn):S1,…,SmT(F_1, \ldots, F_n) : S_1, \ldots, S_mT(F1​,…,Fn​):S1​,…,Sm​
, its instance T[σ]=T⟨τ⟩T[\sigma] = T\langle \tau \rangleT[σ]=T⟨τ⟩
uses the following rules to create captured type KiK_iKi​
from the type parameter FiF_iFi​
and type argument AiA_iAi​
, at least one of which should have specified variance to create a captured type. In case both type parameter and type argument are invariant, their captured type is *equivalent* to AiA_iAi​
.

> Important: type capturing is **not** recursive.

> Note: **All** applicable rules are used to create the resulting constraint set.

- For a covariant type parameter out ⁡Fi\operatorname{\texttt{out\,}}F_ioutFi​
  , if AiA_iAi​
  is an ill-formed type or a contravariant type argument, KiK_iKi​
  is an ill-formed type. Otherwise, Ki<:AiK_i <: A_iKi​<:Ai​
  .
- For a contravariant type parameter in ⁡Fi\operatorname{\texttt{in\,}}F_iinFi​
  , if AiA_iAi​
  is an ill-formed type or a covariant type argument, KiK_iKi​
  is an ill-formed type. Otherwise, Ki:>AiK_i :> A_iKi​:>Ai​
  .
- For a bounded type parameter Fi<:Ui≡B1&⁡…&⁡BmF_i <: U_i \equiv B_1 \mathbin{\operatorname{\&}}\ldots \mathbin{\operatorname{\&}}B_mFi​<:Ui​≡B1​&…&Bm​
  , if ¬(Ai<:τUi)\neg (A_i <: \tau U_i)¬(Ai​<:τUi​)
  , KiK_iKi​
  is an ill-formed type. Otherwise, Ki<:τUiK_i <: \tau U_iKi​<:τUi​
  .

  > Note: captured substitution τ:F1=K1,…,Fn=Kn\tau : F_1 = K_1, \ldots, F_n = K_nτ:F1​=K1​,…,Fn​=Kn​
  > manipulates captured types.
- For a covariant type argument out ⁡Ai\operatorname{\texttt{out\,}}A_ioutAi​
  , if FiF_iFi​
  is a contravariant type parameter, KiK_iKi​
  is an ill-formed type. Otherwise, Ki<:AiK_i <: A_iKi​<:Ai​
  .
- For a contravariant type argument in ⁡Ai\operatorname{\texttt{in\,}}A_iinAi​
  , if FiF_iFi​
  is a covariant type parameter, KiK_iKi​
  is an ill-formed type. Otherwise, Ki:>AiK_i :> A_iKi​:>Ai​
  .
- For a bivariant type argument ⋆\star⋆
  , kotlin.Nothing⁡<:Ki<:kotlin.Any?⁡\operatorname{\texttt{kotlin.Nothing}}<: K_i <: \operatorname{\texttt{kotlin.Any?}}kotlin.Nothing<:Ki​<:kotlin.Any?
  .
- Otherwise, Ki≡AiK_i \equiv A_iKi​≡Ai​
  .

By construction, every captured type KKK
has the following form:

{L1<:K,…,Lp<:K,K<:U1,…,K<:Uq}\{L_1 <: K, \ldots, L_p <: K, K <: U_1, \ldots, K <: U_q\}{L1​<:K,…,Lp​<:K,K<:U1​,…,K<:Uq​}

which can be represented as

L<:K<:UL <: K <: UL<:K<:U

where L=L1∣⁡…∣⁡LpL = L_1 \mathbin{\operatorname{|}}\ldots \mathbin{\operatorname{|}}L_pL=L1​∣…∣Lp​
and U=U1&⁡…&⁡UqU = U_1 \mathbin{\operatorname{\&}}\ldots \mathbin{\operatorname{\&}}U_qU=U1​&…&Uq​
.

> Note: for implementation reasons the compiler may [approximate](https://kotlinlang.org/spec/type-system.html#type-approximation) LLL
> and/or UUU
> ; for example, in the current implementation LLL
> is always approximated to be a single type.

> Note: as every captured type corresponds to a fresh type variable, two different captured types KiK_iKi​
> and KjK_jKj​
> which describe the same set of possible types (i.e., their constraint sets are equal) are *not* considered equal. However, in some cases [type inference](https://kotlinlang.org/spec/type-inference.html#type-inference) may [approximate](https://kotlinlang.org/spec/type-system.html#type-approximation) a captured type KKK
> to a concrete type K≈K^{\approx}K≈
> ; in our case, it would be that Ki≈≡Kj≈K_i^{\approx} \equiv K_j^{\approx}Ki≈​≡Kj≈​
> .

> Examples: also show the use of [type containment](https://kotlinlang.org/spec/type-system.html#type-containment) to establish [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping).
>
> ```
> interface Inv<T>
> interface Out<out T>
> interface In<in T>
>
> interface Root<T>
>
> interface A
> interface B : A
> interface C : B
>
> fun <T> mk(): T = TODO()
>
> interface Bounded<T : A> : Root<T>
>
> fun test01() {
>
>     val bounded: Bounded<in B> = mk()
>
>     // Bounded<in B> <: Bounded<KB> where B <: KB <: A
>     //   (from type capturing)
>     // Bounded<KB> <: Root<KB>
>     //   (from supertype relation)
>
>     val test: Root<in C> = bounded
>
>     // ?- Bounded<in B> <: Root<in C>
>     //
>     // Root<KB> <: Root<in C> where B <: KB <: A
>     //   (from above facts)
>     // KB ⪯ in C
>     //   (from subtyping for parameterized types)
>     // KB ⪯ in KC where C <: KC <: C
>     //   (from type containment rules)
>     // KB :> KC
>     //   (from type containment rules)
>     // (A :> KB :> B) :> (C :> KC :> C)
>     //   (from subtyping for captured types)
>     // B :> C
>     //   (from supertype relation)
>     // True
>
> }
>
> interface Foo<T> : Root<Out<T>>
>
> fun test02() {
>
>     val foo: Foo<out B> = mk()
>
>     // Foo<out B> <: Foo<KB> where KB <: B
>     //   (from type capturing)
>     // Foo<KB> <: Root<Out<KB>>
>     //   (from supertype relation)
>
>     val test: Root<out Out<B>> = foo
>
>     // ?- Foo<out B> <: Root<out Out<B>>
>     //
>     // Root<Out<KB>> <: Root<out Out<B>> where KB <: B
>     //   (from above facts)
>     // Out<KB> ⪯ out Out<B>
>     //   (from subtyping for parameterized types)
>     // Out<KB> <: Out<B>
>     //   (from type containment rules)
>     // Out<out KB> <: Out<out B>
>     //   (from declaration-site variance)
>     // out KB ⪯ out B
>     //   (from subtyping for parameterized types)
>     // out KB ⪯ out KB' where B <: KB' <: B
>     //   (from type containment rules)
>     // KB <: KB'
>     //   (from type containment rules)
>     // (KB :< B) <: (B <: KB' <: B)
>     //   (from subtyping for captured types)
>     // B <: B
>     //   (from subtyping definition)
>     // True
>
> }
>
> interface Bar<T> : Root<Inv<T>>
>
> fun test03() {
>
>     val bar: Bar<out B> = mk()
>
>     // Bar<out B> <: Bar<KB> where KB <: B
>     //   (from type capturing)
>     // Bar<KB> <: Root<Inv<KB>>
>     //   (from supertype relation)
>
>     val test: Root<out Inv<B>> = bar
>
>     // ?- Bar<out B> <: Root<out Inv<B>>
>     //
>     // Root<Inv<KB>> <: Root<out Inv<B>> where KB <: B
>     //   (from above facts)
>     // Inv<KB> ⪯ out Inv<B>
>     //   (from subtyping for parameterized types)
>     // Inv<KB> <: Inv<B>
>     //   (from type containment rules)
>     // KB ⪯ B
>     //   (from subtyping for parameterized types)
>     // KB ⪯ KB' where B <: KB' <: B
>     //   (from type containment rules)
>     // KB ⊆ KB'
>     //   (from type containment rules)
>     // (Nothing <: KB :< B) ⊆ (B <: KB' <: B)
>     //
>     // False
>
> }
>
> interface Recursive<T : Recursive<T>>
>
> fun <T : Recursive<T>> probe(e: Recursive<T>): T = mk()
>
> fun test04() {
>     val rec: Recursive<*> = mk()
>
>     // Recursive<*> <: Recursive<KS> where KS <: Recursive<KS>
>     //   (from type capturing)
>     // Recursive<KS> <: Root<KS>
>     //   (from supertype relation)
>
>     val root: Root<*> = rec
>
>     // ?- Recursive<*> <: Root<*>
>     //
>     // Root<KS> <: Root<KT>
>     //     where Nothing <: KS <: Recursive<KS>
>     //           Nothing <: KT <: Any?
>     //   (from above facts and type capturing)
>     // KS ⪯ KT
>     //   (from subtyping for parameterized types)
>     // KS ⊆ KT
>     //   (from type containment rules)
>     // (Nothing <: KS <: Recursive<KS>) ⊆ (Nothing <: KT <: Any?)
>     //
>     // True
>
>     val rootRec: Root<Recursive<*>> = rec
>
>     // ?- Recursive<*> <: Root<Recursive<*>>
>     //
>     // Root<KS> <: Root<Recursive<*>>
>     //     where Nothing <: KS <: Recursive<KS>
>     //   (from above facts)
>     // KS ⪯ Recursive<*>
>     //   (from subtyping for parameterized types)
>     // KS ⪯ KT where Recursive<*> <: KT <: Recursive<*>
>     //   (from type containment rules)
>     // KS ⊆ KT
>     //   (from type containment rules)
>     // (Nothing <: KS <: Recursive<KS) ⊆ (Recursive<*> <: KT <: Recursive<*>)
>     //
>     // False
>     
> }
> ```

#### Type containment

Type containment operator ⪯\preceq⪯
is used to decide, whether a type AAA
is contained in another type BBB
denoted A⪯BA \preceq BA⪯B
, for the purposes of establishing type argument [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping).

Let A,BA, BA,B
be concrete, well-defined non-type-parameter types, KA,KBK_A, K_BKA​,KB​
be captured types.

> Important: type parameters Fi<:UiF_i <: U_iFi​<:Ui​
> are handled as if they have been converted to well-formed captured types Ki:kotlin.Nothing⁡<:Ki<:UiK_i : \operatorname{\texttt{kotlin.Nothing}}<: K_i <: U_iKi​:kotlin.Nothing<:Ki​<:Ui​
> .

⪯\preceq⪯
is defined as follows.

- A⪯BA \preceq BA⪯B
  if A≡BA \equiv BA≡B
- A⪯out ⁡BA \preceq \operatorname{\texttt{out\,}}BA⪯outB
  if A<:BA <: BA<:B
- A⪯in ⁡BA \preceq \operatorname{\texttt{in\,}}BA⪯inB
  if A:>BA :> BA:>B
- out ⁡A⪯out ⁡B\operatorname{\texttt{out\,}}A \preceq \operatorname{\texttt{out\,}}BoutA⪯outB
  if A<:BA <: BA<:B
- in ⁡A⪯in ⁡B\operatorname{\texttt{in\,}}A \preceq \operatorname{\texttt{in\,}}BinA⪯inB
  if A:>BA :> BA:>B

Rules for captured types follow the same structure.

- KA⪯KBK_A \preceq K_BKA​⪯KB​
  if KA⊆KBK_A \subseteq K_BKA​⊆KB​
- KA⪯out ⁡KBK_A \preceq \operatorname{\texttt{out\,}}K_BKA​⪯outKB​
  if KA<:KBK_A <: K_BKA​<:KB​
- KA⪯in ⁡KBK_A \preceq \operatorname{\texttt{in\,}}K_BKA​⪯inKB​
  if KA:>KBK_A :> K_BKA​:>KB​
- out ⁡KA⪯out ⁡KB\operatorname{\texttt{out\,}}K_A \preceq \operatorname{\texttt{out\,}}K_BoutKA​⪯outKB​
  if KA<:KBK_A <: K_BKA​<:KB​
- in ⁡KA⪯in ⁡KB\operatorname{\texttt{in\,}}K_A \preceq \operatorname{\texttt{in\,}}K_BinKA​⪯inKB​
  if KA:>KBK_A :> K_BKA​:>KB​

In case we need to establish type containment between regular type AAA
and captured type KBK_BKB​
, AAA
is considered as if it is a captured type KA:A<:KA<:AK_A : A <: K_A <: AKA​:A<:KA​<:A
.

#### Function types

Kotlin has first-order functions; e.g., it supports function types, which describe the argument and return types of its corresponding function.

A function type FT⁡\operatorname{\text{FT}}FT

FT⁡(A1,…,An)→R\operatorname{\text{FT}}(A_1, \ldots, A_n) \rightarrow RFT(A1​,…,An​)→R

consists of

- argument types AiA_iAi​
- return type RRR

and may be considered the following instantiation of a special type constructor FunctionN⁡(in ⁡P1,…,in ⁡Pn,out ⁡R)\operatorname{\text{FunctionN}}(\operatorname{\texttt{in\,}}P_1, \ldots, \operatorname{\texttt{in\,}}P_n, \operatorname{\texttt{out\,}}R)FunctionN(inP1​,…,inPn​,outR)
(please note the variance of type parameters)

FT⁡(A1,…,An)→R≡FunctionN⁡[A1,…,An,R]\operatorname{\text{FT}}(A_1, \ldots, A_n) \rightarrow R \equiv \operatorname{\text{FunctionN}}[A_1, \ldots, A_n, R]FT(A1​,…,An​)→R≡FunctionN[A1​,…,An​,R]

These FunctionN⁡\operatorname{\text{FunctionN}}FunctionN
types follow the rules of regular type constructors and parameterized types w.r.t. subtyping.

A function type with receiver FTR⁡\operatorname{\text{FTR}}FTR

FTR⁡(RT⁡,A1,…,An)→R\operatorname{\text{FTR}}(\operatorname{\text{RT}}, A_1, \ldots, A_n) \rightarrow RFTR(RT,A1​,…,An​)→R

consists of

- receiver type RT⁡\operatorname{\text{RT}}RT
- argument types AiA_iAi​
- return type RRR

From the type system’s point of view, it is equivalent to the following function type

FTR⁡(RT⁡,A1,…,An)→R≡FT⁡(RT⁡,A1,…,An)→R\operatorname{\text{FTR}}(\operatorname{\text{RT}}, A_1, \ldots, A_n) \rightarrow R \equiv \operatorname{\text{FT}}(\operatorname{\text{RT}}, A_1, \ldots, A_n) \rightarrow RFTR(RT,A1​,…,An​)→R≡FT(RT,A1​,…,An​)→R

i.e., receiver is considered as yet another argument of its function type.

> Note: this means that, for example, these two types are equivalent w.r.t. type system
>
> - `Int.(Int) -> String`
> - `(Int, Int) -> String`
>
> However, these two types are **not** equivalent w.r.t. [overload resolution](https://kotlinlang.org/spec/overload-resolution.html#overload-resolution), as it distinguishes between functions with and without receiver.

Furthermore, all function types FunctionN⁡\operatorname{\text{FunctionN}}FunctionN
are subtypes of a general argument-agnostic type [kotlin.Function⁡\operatorname{\texttt{kotlin.Function}}kotlin.Function](https://kotlinlang.org/spec/type-system.html#kotlin.function-typesystem) for the purpose of unification; this subtyping relation is also used in [overload resolution](https://kotlinlang.org/spec/overload-resolution.html#determining-function-applicability-for-a-specific-call).

> Note: a compiler implementation may consider a function type FunctionN⁡\operatorname{\text{FunctionN}}FunctionN
> to have additional supertypes, if it is necessary.

> Example:
>
> ```
> // A function of type Function1<Number, Number>
> //   or (Number) -> Number
> fun foo(i: Number): Number = ...
>
> // A valid assignment w.r.t. function type variance
> // Function1<in Int, out Any> :> Function1<in Number, out Number>
> val fooRef: (Int) -> Any = ::foo
>
> // A function with receiver of type Function1<Number, Number>
> //   or Number.() -> Number
> fun Number.bar(): Number = ...
>
> // A valid assignment w.r.t. function type variance
> // Receiver is just yet another function argument
> // Function1<in Int, out Any> :> Function1<in Number, out Number>
> val barRef: (Int) -> Any = Number::bar
> ```

##### Suspending function types

Kotlin supports structured concurrency in the form of [coroutines](https://kotlinlang.org/spec/asynchronous-programming-with-coroutines.html#coroutines) via [suspending functions](https://kotlinlang.org/spec/asynchronous-programming-with-coroutines.html#suspending-functions).

For the purposes of type system, a suspending function has a *suspending* function type suspend⁡FT⁡(A1,…,An)→R\operatorname{\texttt{suspend}}\operatorname{\text{FT}}(A_1, \ldots, A_n) \rightarrow RsuspendFT(A1​,…,An​)→R
, which is **unrelated by subtyping** to any non-suspending function type. This is important for [overload resolution](https://kotlinlang.org/spec/overload-resolution.html#overload-resolution) and [type inference](https://kotlinlang.org/spec/type-inference.html#type-inference), as it directly influences the types of function values and the applicability of different functions w.r.t. overloading.

Most function values have either non-suspending or suspending function type based on their declarations. However, as [lambda literals](https://kotlinlang.org/spec/expressions.html#lambda-literals) do not have any explicitly declared function type, they are considered as possibly being both non-suspending and suspending function type, with the final selection done during [type inference](https://kotlinlang.org/spec/type-inference.html#type-inference).

> Example:
>
> ```
> fun foo(i: Int): String = TODO()
>
> fun bar() {
>     val fooRef: (Int) -> String = ::foo
>     val fooLambda: (Int) -> String = { it.toString() }
>     val suspendFooLambda: suspend (Int) -> String = { it.toString() }
>
>     // Error: as suspending and non-suspending
>     //   function types are unrelated
>     // val error: suspend (Int) -> String = ::foo
>     // val error: suspend (Int) -> String = fooLambda
>     // val error: (Int) -> String = suspendFooLambda
> }
> ```

#### Flexible types

Kotlin, being a multi-platform language, needs to support transparent interoperability with platform-dependent code. However, this presents a problem in that some platforms may not support null safety the way Kotlin does. To deal with this, Kotlin supports *gradual typing* in the form of flexible types.

A flexible type represents a range of possible types between type LLL
(lower bound) and type UUU
(upper bound), written as (L..U)(L..U)(L..U)
. One should note flexible types are *non-denotable*, i.e., one cannot explicitly declare a variable with flexible type, these types are created by the type system when needed.

To represent a well-formed flexible type, (L..U)(L..U)(L..U)
should satisfy the following conditions.

- LLL
  and UUU
  are well-formed concrete types
- L<:UL <: UL<:U
- LLL
  and UUU
  are **not** flexible types (but may contain other flexible types as some of their type arguments)

As the name suggests, flexible types are flexible — a value of type (L..U)(L..U)(L..U)
can be used in any context, where one of the possible types between LLL
and UUU
is needed (for more details, see [subtyping rules for flexible types](https://kotlinlang.org/spec/type-system.html#subtyping-for-flexible-types)). However, the actual runtime type TTT
will be a specific type satisfying ∃S:T<:S∧L<:S<:U\exists S : T <: S \land L <: S <: U∃S:T<:S∧L<:S<:U
, thus making the substitution possibly unsafe, which is why Kotlin generates dynamic assertions, when it is impossible to prove statically the safety of flexible type use.

##### Dynamic type

Kotlin includes a special `dynamic` type, which in many contexts can be viewed as a flexible type (kotlin.Nothing⁡..kotlin.Any?⁡)(\operatorname{\texttt{kotlin.Nothing}}..\operatorname{\texttt{kotlin.Any?}})(kotlin.Nothing..kotlin.Any?)
. By definition, this type represents *any* possible Kotlin type, and may be used to support interoperability with dynamically typed libraries, platforms or languages.

However, as a platform may assign special meaning to the values of `dynamic` type, it may be handled differently from the regular flexible type. These differences are to be explained in the corresponding platform-dependent sections of this specification.

##### Platform types

The main use cases for flexible types are *platform types* — types which the Kotlin compiler uses, when interoperating with code written for another platform (e.g., Java). In this case all types on the interoperability boundary are subject to *flexibilization* — the process of converting a platform-specific type to a Kotlin-compatible flexible type.

For further details on how *flexibilization* is done, see the corresponding JVM section.

> Important: platform types should not be confused with *multi-platform projects* — another Kotlin feature targeted at supporting platform interop.

#### Nullable types

Kotlin supports null safety by having two type universes — nullable and non-nullable. All classifier type declarations, built-in or user-defined, create non-nullable types, i.e., types which cannot hold `null` value at runtime.

To specify a nullable version of type TTT
, one needs to use T?T?T?
as a type. Redundant nullability specifiers are ignored: T??≡T?T?? \equiv T?T??≡T?
.

> Note: informally, question mark means “T?T?T?
> may hold values of type TTT
> or value `null`”

To represent a well-formed nullable type, T?T?T?
should satisfy the following conditions.

- TTT
  is a well-formed concrete type

> Note: if an operation is safe regardless of absence or presence of `null`, e.g., assignment of one nullable value to another, it can be used as-is for nullable types. For operations on T?T?T?
> which may violate null safety, e.g., access to a property, one has the following null-safe options:
>
> 1. Use safe operations
>    - [safe call](https://kotlinlang.org/spec/expressions.html#navigation-operators)
> 2. Downcast from T?T?T?
>    to T!!T!!T!!
>    - [unsafe cast](https://kotlinlang.org/spec/expressions.html#cast-expressions)
>    - [type check](https://kotlinlang.org/spec/expressions.html#type-checking-expressions) combined with [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts)
>    - null check combined with [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts)
>    - [not-null assertion operator](https://kotlinlang.org/spec/expressions.html#not-null-assertion-expressions)
> 3. Supply a default value to use if `null` is present
>    - [elvis operator](https://kotlinlang.org/spec/expressions.html#elvis-operator-expressions)

##### Nullability lozenge

> ![](data:image/svg+xml;utf8,<?xml version="1.0"?>
> <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:jfreesvg="http://www.jfree.org/jfreesvg/svg" width="289" height="194" text-rendering="auto" shape-rendering="auto">
> <defs></defs>
> <line x1="7.5" y1="7.5" x2="127.5" y2="7.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="7.5" x2="397.5" y2="7.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="135" y1="52.5" x2="277.5" y2="52.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 127.5 52.5 L 142.5 47.5 L 142.5 57.5 Z "/></g><line x1="7.5" y1="97.5" x2="127.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="97.5" x2="397.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="202.5" y1="127.5" x2="292.5" y2="127.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="112.5" y1="142.5" x2="202.5" y2="142.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="7.5" y1="172.5" x2="127.5" y2="172.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="172.5" x2="397.5" y2="172.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="135" y1="217.5" x2="277.5" y2="217.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 127.5 217.5 L 142.5 212.5 L 142.5 222.5 Z "/></g><line x1="7.5" y1="262.5" x2="127.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="262.5" x2="397.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="7.5" y1="7.5" x2="7.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="127.5" y1="7.5" x2="127.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="7.5" x2="277.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="397.5" y1="7.5" x2="397.5" y2="97.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="292.5" y1="97.5" x2="292.5" y2="127.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="202.5" y1="127.5" x2="202.5" y2="142.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="112.5" y1="142.5" x2="112.5" y2="165" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 112.5 172.5 L 107.5 157.5 L 117.5 157.5 Z "/></g><line x1="7.5" y1="172.5" x2="7.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="127.5" y1="172.5" x2="127.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="277.5" y1="172.5" x2="277.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="397.5" y1="172.5" x2="397.5" y2="262.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="54.5" y="60.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >A?</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="324.5" y="60.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >B?</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="47.5" y="225.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >A!!</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="317.5" y="225.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >B!!</text></g></svg>
> )

Nullability lozenge represents valid possible [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping) relations between two nullable or non-nullable types in different combinations of their *versions*. For type TTT
, we call T!!T!!T!!
its non-nullable version, T?T?T?
its nullable version.

> Note: trivial subtyping relation A!!<:A?A!! <: A?A!!<:A?
> is not represented in the nullability lozenge.

Nullability lozenge may also help in establishing subtyping between two types by following its structure.

Regular (non-type-variable) types are mapped to nullability lozenge *vertices*, as for them AAA
corresponds to A!!A!!A!!
, and A?A?A?
corresponds to A?A?A?
. Following the lozenge structure, for regular types AAA
and BBB
, as soon as we have established any valid subtyping between two versions of AAA
and BBB
, it implies subtyping between all other valid w.r.t. nullability lozenge combinations of versions of types AAA
and BBB
.

Type variable types (e.g., captured types or type parameters) are mapped to either nullability lozenge *edges* or *vertices*, as for them TTT
corresponds to either T!!T!!T!!
or T?T?T?
, and T?T?T?
corresponds to T?T?T?
. Following the lozenge structure, for type variable type TTT
(i.e., either non-nullable or nullable version) we need to consider valid subtyping for both versions T!!T!!T!!
and T?T?T?
w.r.t. nullability lozenge.

> Example: if we have kotlin.Int⁡?<:T?\operatorname{\texttt{kotlin.Int}}? <: T?kotlin.Int?<:T?
> , we also have kotlin.Int⁡!!<:T?\operatorname{\texttt{kotlin.Int}}!! <: T?kotlin.Int!!<:T?
> and kotlin.Int⁡!!<:T!!\operatorname{\texttt{kotlin.Int}}!! <: T!!kotlin.Int!!<:T!!
> , meaning we can establish kotlin.Int⁡!!<:T≡kotlin.Int⁡<:T\operatorname{\texttt{kotlin.Int}}!! <: T \equiv \operatorname{\texttt{kotlin.Int}}<: Tkotlin.Int!!<:T≡kotlin.Int<:T
> .

> Example: if we have T?<:kotlin.Int⁡?T? <: \operatorname{\texttt{kotlin.Int}}?T?<:kotlin.Int?
> , we also have T!!<:kotlin.Int⁡?T!! <: \operatorname{\texttt{kotlin.Int}}?T!!<:kotlin.Int?
> and T!!<:kotlin.Int⁡!!T!! <: \operatorname{\texttt{kotlin.Int}}!!T!!<:kotlin.Int!!
> , however, we can establish only T<:kotlin.Int⁡?T <: \operatorname{\texttt{kotlin.Int}}?T<:kotlin.Int?
> , as T<:kotlin.Int⁡T <: \operatorname{\texttt{kotlin.Int}}T<:kotlin.Int
> would need T?<:kotlin.Int⁡!!T? <: \operatorname{\texttt{kotlin.Int}}!!T?<:kotlin.Int!!
> which is forbidden by the nullability lozenge.

##### Definitely non-nullable types

As discussed [here](https://kotlinlang.org/spec/type-system.html#nullability-lozenge), type variable types have unknown nullability, e.g., a type parameter TTT
may correspond to either nullable version T?T?T?
, or non-nullable version T!!T!!T!!
. In some cases, one might need to specifically denote a nullable/non-nullable version of TTT
.

> Note: for example, it is needed when overriding a Java method with a `@NotNull` annotated generic parameter.
>
> Example:
>
> ```
> public interface JBox {
>     <T> void put(@NotNull T t);
> }
> ```
>
> ```
> class KBox : JBox {
>     override fun <T> put(t: T/* !! */) = TODO()
> }
> ```

To denote a nullable version of TTT
, one can use the [nullable type](https://kotlinlang.org/spec/type-system.html#nullable-types) syntax T?T?T?
.

To denote a non-nullable version of TTT
, one can use the definitely non-nullable type syntax T&⁡AnyT \mathbin{\operatorname{\&}}AnyT&Any
.

To represent a well-formed definitely non-nullable type, T&⁡AnyT \mathbin{\operatorname{\&}}AnyT&Any
should satisfy the following conditions.

- TTT
  is a well-formed [type parameter](https://kotlinlang.org/spec/type-system.html#type-parameters) with a nullable upper bound
- AnyAnyAny
  is resolved to kotlin.Any⁡\operatorname{\texttt{kotlin.Any}}kotlin.Any

> Example:
>
> ```
> typealias MyAny = kotlin.Any
>
> fun <T /* : Any? */, Q : Any> bar(t: T?, q: Q?, i: Int?) {
>     // OK
>     val a: T & Any = t!!
>     // OK: MyAny is resolved to kotlin.Any
>     val b: T & MyAny = t!!
>     // ERROR: Int is not kotlin.Any
>     val c: T & Int = t!!
>
>     // ERROR: Q does not have a nullable upper bound
>     val d: Q & Any = q!!
>
>     // ERROR: Int? is not a type parameter
>     val e: Int? & Any = i!!
> }
> ```

One may notice the syntax looks like an intersection type T&⁡AnyT \mathbin{\operatorname{\&}}AnyT&Any
, and that is not a coincidence, as an intersection type with AnyAnyAny
describes exactly a type which cannot hold `null` values. For the purposes of the type system, a definitely non-nullable type T&⁡AnyT \mathbin{\operatorname{\&}}AnyT&Any
is consider to be the same as an [intersection type](https://kotlinlang.org/spec/type-system.html#intersection-types) T&⁡AnyT \mathbin{\operatorname{\&}}AnyT&Any
.

#### Intersection types

Intersection types are special *non-denotable* types used to express the fact that a value belongs to *all* of *several* types at the same time.

Intersection type of two types AAA
and BBB
is denoted A&⁡BA \mathbin{\operatorname{\&}}BA&B
and is equivalent to the [greatest lower bound](https://kotlinlang.org/spec/type-system.html#greatest-lower-bound) of its components GLB⁡(A,B)\operatorname{\text{GLB}}(A, B)GLB(A,B)
. Thus, the normalization procedure for GLB⁡\operatorname{\text{GLB}}GLB
may be used to *normalize* an intersection type.

> Note: this means intersection types are commutative and associative (following the GLB properties); e.g., A&⁡BA \mathbin{\operatorname{\&}}BA&B
> is the same type as B&⁡AB \mathbin{\operatorname{\&}}AB&A
> , and A&⁡(B&⁡C)A \mathbin{\operatorname{\&}}(B \mathbin{\operatorname{\&}}C)A&(B&C)
> is the same type as A&⁡B&⁡CA \mathbin{\operatorname{\&}}B \mathbin{\operatorname{\&}}CA&B&C
> .

> Note: for presentation purposes, we will henceforth order intersection type operands lexicographically based on their notation.

When needed, the compiler may *approximate* an intersection type to a *denotable concrete* type using [type approximation](https://kotlinlang.org/spec/type-system.html#type-approximation).

One of the main uses of intersection types are [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts). Another restricted version of intersection types are [definitely non-nullable types](https://kotlinlang.org/spec/type-system.html#definitely-non-nullable-types).

#### Integer literal types

An integer literal type containing types T1,…,TNT_1, \ldots, T_NT1​,…,TN​
, denoted ILT⁡(T1,…,TN)\operatorname{\text{ILT}}(T_1, \ldots, T_N)ILT(T1​,…,TN​)
is a special *non-denotable* type designed for integer literals. Each type T1,…,TNT_1, \ldots, T_NT1​,…,TN​
must be one of the [built-in integer types](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#built-in-integer-types-builtins).

Integer literal types are the types of [integer literals](https://kotlinlang.org/spec/expressions.html#integer-literals) and have special handling w.r.t. [subtyping](https://kotlinlang.org/spec/type-system.html#subtyping-for-integer-literal-types).

#### Union types

> Important: Kotlin does **not** have union types in its type system. However, they make reasoning about several type system features easier. Therefore, we decided to include a brief intro to the union types here.

Union types are special *non-denotable* types used to express the fact that a value belongs to *one* of *several* possible types.

Union type of two types AAA
and BBB
is denoted A∣⁡BA \mathbin{\operatorname{|}}BA∣B
and is equivalent to the [least upper bound](https://kotlinlang.org/spec/type-system.html#least-upper-bound) of its components LUB⁡(A,B)\operatorname{\text{LUB}}(A, B)LUB(A,B)
. Thus, the normalization procedure for LUB⁡\operatorname{\text{LUB}}LUB
may be used to *normalize* a union type.

Moreover, as union types are *not* used in Kotlin, the compiler always *decays* a union type to a *non-union* type using [type decaying](https://kotlinlang.org/spec/type-system.html#type-decaying).

### Type contexts and scopes

The way types and [scopes](https://kotlinlang.org/spec/scopes-and-identifiers.html#scopes-and-identifiers) interoperate is very similar to how values and scopes work; this includes [visibility](https://kotlinlang.org/spec/declarations.html#declaration-visibility), accessing types via qualified names or [imports](https://kotlinlang.org/spec/packages-and-imports.html#importing). This means, in many cases, type contexts are equivalent to the corresponding scopes. However, there are several important differences, which we outline below.

#### Inner and nested type contexts

[Type parameters](https://kotlinlang.org/spec/type-system.html#type-parameters) are well-formed types in the type context (scope) of their declaring type constructor, including inner type declarations. However, type context for a [*nested* type declaration](https://kotlinlang.org/spec/declarations.html#nested-and-inner-classifiers) ND⁡\operatorname{\text{ND}}ND
of a parent type declaration PD⁡\operatorname{\text{PD}}PD
does **not** include the type parameters of PD⁡\operatorname{\text{PD}}PD
.

> Note: nested type declarations cannot capture parent type parameters, as they simply create a regular type available under a nested path.

> Example:
>
> ```
> class Parent<T> {
>     class Nested(val i: Int)
>
>     // Can use type parameter T as a type
>     // in an inner class
>     inner class Inner(val t: T)
>
>     // Cannot use type parameter T as a type
>     // in a nested class
>     class Error(val t: T)
> }
>
> fun main() {
>     val nested = Parent.Nested(42)
>
>     val inner = Parent<String>().Inner("42")
> }
> ```

### Subtyping

Kotlin uses the classic notion of *subtyping* as *substitutability* — if SSS
is a subtype of TTT
(denoted as S<:TS <: TS<:T
), values of type SSS
can be safely used where values of type TTT
are expected. The subtyping relation <:<:<:
is:

- reflexive (A<:AA <: AA<:A
  )
- *rigidly* transitive (A<:B∧B<:C⇒A<:CA <: B \land B <: C \Rightarrow A <: CA<:B∧B<:C⇒A<:C
  for non-flexible types AAA
  , BBB
  and CCC
  )

Two types AAA
and BBB
are *equivalent* (A≡BA \equiv BA≡B
), iff A<:B∧B<:AA <: B \land B <: AA<:B∧B<:A
. Due to the presence of flexible types, this relation is also only *rigidly* transitive, e.g., holds only for non-flexible types (see [here](https://kotlinlang.org/spec/type-system.html#subtyping-for-flexible-types) for more details).

#### Subtyping rules

Subtyping for non-nullable, concrete types uses the following rules.

- ∀T:kotlin.Nothing⁡<:T<:kotlin.Any⁡\forall T : \operatorname{\texttt{kotlin.Nothing}}<: T <: \operatorname{\texttt{kotlin.Any}}∀T:kotlin.Nothing<:T<:kotlin.Any
- For any simple classifier type T:S1,…,SmT : S_1, \ldots, S_mT:S1​,…,Sm​
  it is true that ∀i∈[1,m]:T<:Si\forall i \in [1,m]: T <: S_i∀i∈[1,m]:T<:Si​
- For any parameterized type T^=T⟨τ⟩:S1,…,Sm\widehat{T} = T\langle \tau \rangle: S_1, \ldots, S_mT=T⟨τ⟩:S1​,…,Sm​
  it is true that ∀i∈[1,m]:T^<:τSi\forall i \in [1,m]: \widehat{T} <: \tau S_i∀i∈[1,m]:T<:τSi​
- For any two parameterized types T^=T⟨τ⟩\widehat{T} = T\langle \tau \rangleT=T⟨τ⟩
  and T′^=T⟨τ′⟩\widehat{T^\prime} = T\langle \tau^\prime \rangleT′=T⟨τ′⟩
  with captured type arguments KiK_iKi​
  and Ki′K_i^\primeKi′​
  it is true that T^<:T′^\widehat{T} <: \widehat{T^\prime}T<:T′
  if ∀i∈[1,n]:Ki⪯Ki′\forall i \in [1,n]: K_i \preceq K_i^\prime∀i∈[1,n]:Ki​⪯Ki′​

Subtyping for captured types uses the following rules.

- ∀K:kotlin.Nothing⁡<:K<:kotlin.Any?⁡\forall K : \operatorname{\texttt{kotlin.Nothing}}<: K <: \operatorname{\texttt{kotlin.Any?}}∀K:kotlin.Nothing<:K<:kotlin.Any?
- For any two captured types L<:K<:UL <: K <: UL<:K<:U
  and L′<:K′<:U′L^\prime <: K^\prime <: U^\primeL′<:K′<:U′
  , it is true that K<:K′K <: K^\primeK<:K′
  if U<:L′U <: L^\primeU<:L′

Subtyping for nullable types is checked separately and uses a special set of rules which are described [here](https://kotlinlang.org/spec/type-system.html#subtyping-for-nullable-types).

#### Subtyping for flexible types

Flexible types (being flexible) follow a simple subtyping relation with other rigid (i.e., non-flexible) types. Let T,A,B,L,UT, A, B, L, UT,A,B,L,U
be rigid types.

- L<:T⇒(L..U)<:TL <: T \Rightarrow (L..U) <: TL<:T⇒(L..U)<:T
- T<:U⇒T<:(L..U)T <: U \Rightarrow T <: (L..U)T<:U⇒T<:(L..U)

This captures the notion of flexible type (L..U)(L..U)(L..U)
as something which may be used in place of any type in between LLL
and UUU
. If we are to extend this idea to subtyping between *two* flexible types, we get the following definition.

- L<:B⇒(L..U)<:(A..B)L <: B \Rightarrow (L..U) <: (A..B)L<:B⇒(L..U)<:(A..B)

This is the most extensive definition possible, which, unfortunately, makes the type equivalence relation non-transitive. Let A,BA, BA,B
be two *different* types, for which A<:BA <: BA<:B
. The following relations hold:

- A<:(A..B)∧(A..B)<:A⇒A≡(A..B)A <: (A..B) \land (A..B) <: A \Rightarrow A \equiv (A..B)A<:(A..B)∧(A..B)<:A⇒A≡(A..B)
- B<:(A..B)∧(A..B)<:B⇒B≡(A..B)B <: (A..B) \land (A..B) <: B \Rightarrow B \equiv (A..B)B<:(A..B)∧(A..B)<:B⇒B≡(A..B)

However, A≢BA \not \equiv BA​≡B
.

#### Subtyping for intersection types

Intersection types introduce several new rules for subtyping. Let A,B,C,DA, B, C, DA,B,C,D
be non-nullable types.

- A&⁡B<:AA \mathbin{\operatorname{\&}}B <: AA&B<:A
- A&⁡B<:BA \mathbin{\operatorname{\&}}B <: BA&B<:B
- A<:C∧B<:D⇒A&⁡B<:C&⁡DA <: C \land B <: D \Rightarrow A \mathbin{\operatorname{\&}}B <: C \mathbin{\operatorname{\&}}DA<:C∧B<:D⇒A&B<:C&D

Moreover, any type TTT
with supertypes S1,…,SNS_1, \ldots, S_NS1​,…,SN​
is also a subtype of S1&⁡…&⁡SNS_1 \mathbin{\operatorname{\&}}\ldots \mathbin{\operatorname{\&}}S_NS1​&…&SN​
.

#### Subtyping for integer literal types

All integer literal type are equivalent w.r.t. subtyping, meaning that for any sets T1,…,TKT_1, \ldots, T_KT1​,…,TK​
and U1,…,UNU_1, \ldots, U_NU1​,…,UN​
of built-in integer types:

- ILT⁡(T1,…,TK)<:ILT⁡(U1,…,UN)\operatorname{\text{ILT}}(T_1, \ldots, T_K) <: \operatorname{\text{ILT}}(U_1, \ldots, U_N)ILT(T1​,…,TK​)<:ILT(U1​,…,UN​)
- ILT⁡(U1,…,UN)<:ILT⁡(T1,…,TK)\operatorname{\text{ILT}}(U_1, \ldots, U_N) <: \operatorname{\text{ILT}}(T_1, \ldots, T_K)ILT(U1​,…,UN​)<:ILT(T1​,…,TK​)
- ∀Ti∈{T1,…,TK}:ILT⁡(T1,…,TK)<:Ti\forall T_i \in \{T_1, \ldots, T_K\} : \operatorname{\text{ILT}}(T_1, \ldots, T_K) <: T_i∀Ti​∈{T1​,…,TK​}:ILT(T1​,…,TK​)<:Ti​
- ∀Ti∈{T1,…,TK}:Ti<:ILT⁡(T1,…,TK)\forall T_i \in \{T_1, \ldots, T_K\} : T_i <: \operatorname{\text{ILT}}(T_1, \ldots, T_K)∀Ti​∈{T1​,…,TK​}:Ti​<:ILT(T1​,…,TK​)

> Note: the last two rules mean ILT⁡(T1,…,TK)\operatorname{\text{ILT}}(T_1, \ldots, T_K)ILT(T1​,…,TK​)
> can be considered as an intersection type T1&⁡…&⁡TKT_1 \mathbin{\operatorname{\&}}\ldots \mathbin{\operatorname{\&}}T_KT1​&…&TK​
> or as a union type T1∣⁡…∣⁡TKT_1 \mathbin{\operatorname{|}}\ldots \mathbin{\operatorname{|}}T_KT1​∣…∣TK​
> , depending on the context. Viewing ILT⁡\operatorname{\text{ILT}}ILT
> as intersection type allows us to use integer literals where built-in integer types are expected. Making ILT⁡\operatorname{\text{ILT}}ILT
> behave as union type is needed to support cases when they appear in contravariant position.

> Example:
>
> ```
> interface In<in T>
>
> fun <T> T.asIn(): In<T> = ...
>
> fun <S> select(a: S, b: In<S>): S = ...
>
> fun iltAsIntersection() {
>     val a: Int = 42 // ILT(Byte, Short, Int, Long) <: Int
>     
>     fun foo(a: Short) {}
>     
>     foo(1377) // ILT(Short, Int, Long) <: Short
> }
>
> fun iltAsUnion() {
>     val a: Short = 42
>     
>     select(a, 1337.asIn())
>         // For argument a:
>         //   Short <: S
>         // For argument b:
>         //   In<ILT(Short, Int, Long)> <: In<S> =>
>         //     S <: ILT(Short, Int, Long)
>         // Solution: S =:= Short
> }
> ```

#### Subtyping for nullable types

Subtyping for two possibly nullable types AAA
and BBB
is defined via *two* relations, both of which must hold.

1. Regular subtyping <:<:<:
   for types AAA
   and BBB
   using the [nullability lozenge](https://kotlinlang.org/spec/type-system.html#nullability-lozenge)
2. Subtyping by nullability <:null⁡\mathrel{\operatorname{\stackrel{null}{<:}}}<:null​

Subtyping by nullability <:null⁡\mathrel{\operatorname{\stackrel{null}{<:}}}<:null​
for two possibly nullable types AAA
and BBB
uses the following rules.

1. A!!<:null⁡BA!! \mathrel{\operatorname{\stackrel{null}{<:}}}BA!!<:null​B
2. A<:null⁡BA \mathrel{\operatorname{\stackrel{null}{<:}}}BA<:null​B
   if ∃T!!:A<:T!!\exists T!! : A <: T!!∃T!!:A<:T!!
3. A<:null⁡B?A \mathrel{\operatorname{\stackrel{null}{<:}}}B?A<:null​B?
4. A<:null⁡BA \mathrel{\operatorname{\stackrel{null}{<:}}}BA<:null​B
   if ∄T!!:B<:T!!\nexists T!! : B <: T!!∄T!!:B<:T!!
5. A?̸<:null⁡BA? \not \mathrel{\operatorname{\stackrel{null}{<:}}}BA?​<:null​B

> Informally: these rules represent the following idea derived from the nullability lozenge.
>
> > A̸<:null⁡BA \not \mathrel{\operatorname{\stackrel{null}{<:}}}BA​<:null​B
> > if BBB
> > is definitely non-nullable and AAA
> > may be nullable or BBB
> > may be non-nullable and AAA
> > is definitely nullable.

> Note: these rules follow the structure of the nullability lozenge and check the absence of nullability violation A?<:null⁡B!!A? \mathrel{\operatorname{\stackrel{null}{<:}}}B!!A?<:null​B!!
> via underapproximating it using the *supertype* relation (as we cannot enumerate the *subtype* relation for BBB
> ).

> Example:
>
> ```
> class Foo<A, B : A?> {
>     val b: B = mk()
>     val bQ: B? = mk()
>     
>     // For this assignment to be well-formed,
>     //   B must be a subtype of A
>     // Subtyping by nullability holds per rule 4
>     // Regular subtyping does not hold,
>     //   as B <: A? is not enough to show B <: A
>     //   (we are missing B!! <: A!!)
>     val ab: A = b // ERROR
>
>     // For this assignment to be well-formed,
>     //   B? must be a subtype of A
>     // Subtyping by nullability does not hold per rule 5
>     val abQ: A = bQ // ERROR
>     
>     // For this assignment to be well-formed,
>     //   B must be a subtype of A?
>     // Subtyping by nullability holds per rule 3
>     // Regular subtyping does hold,
>     //   as B <: A? is enough to show B <: A?
>     val aQb: A? = b // OK
>     
>     // For this assignment to be well-formed,
>     //   B? must be a subtype of A?
>     // Subtyping by nullability holds per rule 3
>     // Regular subtyping does hold,
>     //   as B <: A? is enough to show B? <: A?
>     //   (taking the upper edge of the nullability lozenge)
>     val aQbQ: A? = bQ // OK
> }
>
> class Bar<A, B : A> {
>     val b: B = mk()
>     val bQ: B? = mk()
>     
>     // For this assignment to be well-formed,
>     //   B must be a subtype of A
>     // Subtyping by nullability holds per rule 4
>     // Regular subtyping does hold,
>     //   as B <: A is enough to show B <: A
>     val ab: A = b // OK
>
>     // For this assignment to be well-formed,
>     //   B? must be a subtype of A
>     // Subtyping by nullability does not hold per rule 5
>     val abQ: A = bQ // ERROR
>     
>     // For this assignment to be well-formed,
>     //   B must be a subtype of A?
>     // Subtyping by nullability holds per rule 3
>     // Regular subtyping does hold,
>     //   as B <: A is enough to show B <: A?
>     //   (taking the upper triangle of the nullability lozenge)
>     val aQb: A? = b // OK
>     
>     // For this assignment to be well-formed,
>     //   B? must be a subtype of A?
>     // Subtyping by nullability holds per rule 3
>     // Regular subtyping does hold,
>     //   as B <: A is enough to show B? <: A?
>     //   (taking the upper edge of the nullability lozenge)
>     val aQbQ: A? = bQ // OK
> }
> ```

> Example:
>
> ![](data:image/svg+xml;utf8,<?xml version="1.0"?>
> <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:jfreesvg="http://www.jfree.org/jfreesvg/svg" width="299" height="79" text-rendering="auto" shape-rendering="auto">
> <defs></defs>
> <line x1="142.5" y1="52.5" x2="232.5" y2="52.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="307.5" y1="52.5" x2="330" y2="52.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 337.5 52.5 L 322.5 57.5 L 322.5 47.5 Z "/></g><line x1="67.5" y1="37.5" x2="67.5" y2="75" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 67.5 82.5 L 62.5 67.5 L 72.5 67.5 Z "/></g><line x1="142.5" y1="37.5" x2="142.5" y2="52.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="232.5" y1="37.5" x2="232.5" y2="52.5" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><line x1="412.5" y1="37.5" x2="412.5" y2="75" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 412.5 82.5 L 407.5 67.5 L 417.5 67.5 Z "/></g><line x1="187.5" y1="52.5" x2="187.5" y2="75" style="stroke-width: 1.5;stroke: rgb(0,0,0);stroke-opacity: 1.0;stroke-linecap: square;" transform="matrix(0.7,0,0,0.7,0,0)" /><g style="fill: rgb(0,0,0); fill-opacity: 1.0; stroke: none" transform="matrix(0.7,0,0,0.7,0,0)" ><path d="M 187.5 82.5 L 182.5 67.5 L 192.5 67.5 Z "/></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="60.5" y="30.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >A</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="129.5" y="30.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >B?</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="212.5" y="30.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >C!!</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="405.5" y="30.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >A</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="60.5" y="105.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >B</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="180.5" y="105.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >T</text></g><g transform="matrix(0.7,0,0,0.7,0,0)"><text x="405.5" y="105.5" style="fill: rgb(0,0,0); fill-opacity: 1.0; font-family: monospace; font-size: 23px;" >T</text></g></svg>
> )
>
> This example shows a situation, when the subtyping by nullability relation from T<:C!!T <: C!!T<:C!!
> is used to prove T<:AT <: AT<:A
> .

### Upper and lower bounds

A type UUU
is an *upper bound* of types AAA
and BBB
if A<:UA <: UA<:U
and B<:UB <: UB<:U
. A type LLL
is a *lower bound* of types AAA
and BBB
if L<:AL <: AL<:A
and L<:BL <: BL<:B
.

> Note: as the type system of Kotlin is bounded by definition (the upper bound of all types is kotlin.Any?⁡\operatorname{\texttt{kotlin.Any?}}kotlin.Any?
> , and the lower bound of all types is kotlin.Nothing⁡\operatorname{\texttt{kotlin.Nothing}}kotlin.Nothing
> ), any two types have at least one lower bound and at least one upper bound.

#### Least upper bound

The *least upper bound* LUB⁡(A,B)\operatorname{\text{LUB}}(A, B)LUB(A,B)
of types AAA
and BBB
is an upper bound UUU
of AAA
and BBB
such that there is no other upper bound of these types which is less by subtyping relation than UUU
.

> Note: LUB⁡\operatorname{\text{LUB}}LUB
> is commutative, i.e., LUB⁡(A,B)=LUB⁡(B,A)\operatorname{\text{LUB}}(A, B) = \operatorname{\text{LUB}}(B, A)LUB(A,B)=LUB(B,A)
> . This property is used in the subsequent description, e.g., other properties of LUB⁡\operatorname{\text{LUB}}LUB
> are defined only for a specific order of the arguments. Definitions following from commutativity of LUB⁡\operatorname{\text{LUB}}LUB
> are implied.

LUB⁡(A,B)\operatorname{\text{LUB}}(A, B)LUB(A,B)
has the following properties, which may be used to *normalize* it. This normalization procedure, if finite, creates a *canonical* representation of LUB.

> Important: AAA
> and BBB
> are considered to be non-flexible, unless specified otherwise.

- LUB⁡(A,A)=A\operatorname{\text{LUB}}(A, A) = ALUB(A,A)=A
- if A<:BA <: BA<:B
  , LUB⁡(A,B)=B\operatorname{\text{LUB}}(A, B) = BLUB(A,B)=B
- if AAA
  is nullable, LUB⁡(A,B)=LUB⁡(A!!,B!!)?\operatorname{\text{LUB}}(A, B) = \operatorname{\text{LUB}}(A!!, B!!)?LUB(A,B)=LUB(A!!,B!!)?
- if A=T⟨KA,1,…,KA,n⟩A = T\langle K_{A,1}, \ldots, K_{A,n}\rangleA=T⟨KA,1​,…,KA,n​⟩
  and B=T⟨KB,1,…,KB,n⟩B = T\langle K_{B,1}, \ldots, K_{B,n}\rangleB=T⟨KB,1​,…,KB,n​⟩
  , LUB⁡(A,B)=T⟨ϕ(η(KA,1),η(KB,1)),…,ϕ(η(KA,n),η(KB,n))⟩\operatorname{\text{LUB}}(A, B) = T\langle \phi(\eta(K_{A,1}), \eta(K_{B,1})), \ldots, \phi(\eta(K_{A,n}), \eta(K_{B,n}))\rangleLUB(A,B)=T⟨ϕ(η(KA,1​),η(KB,1​)),…,ϕ(η(KA,n​),η(KB,n​))⟩
  , where η(T)\eta(T)η(T)
  and ϕ(X,Y)\phi(X, Y)ϕ(X,Y)
  are defined as follows:

  η(K:L<:K<:U)={out ⁡U,in ⁡L}
  \eta(K : L <: K <: U) = \{\operatorname{\texttt{out\,}}U, \operatorname{\texttt{in\,}}L\}
  η(K:L<:K<:U)={outU,inL}

  > Informally: in many cases, one may view η(T)\eta(T)η(T)
  > as follows. η(inv ⁡X)={out ⁡X,in ⁡X}η(out ⁡X)={out ⁡X,in ⁡kotlin.Nothing⁡}η(in ⁡X)={out ⁡kotlin.Any?⁡,in ⁡X}η(⋆)={out ⁡kotlin.Any?⁡,in ⁡kotlin.Nothing⁡}
  > \begin{aligned}
  > \eta(\operatorname{\texttt{inv\,}}X) &= \{\operatorname{\texttt{out\,}}X, \operatorname{\texttt{in\,}}X\} \\
  > \eta(\operatorname{\texttt{out\,}}X) &= \{\operatorname{\texttt{out\,}}X, \operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}\} \\
  > \eta(\operatorname{\texttt{in\,}}X) &= \{\operatorname{\texttt{out\,}}\operatorname{\texttt{kotlin.Any?}}, \operatorname{\texttt{in\,}}X\} \\
  > \eta(\star) &= \{\operatorname{\texttt{out\,}}\operatorname{\texttt{kotlin.Any?}}, \operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}\}
  > \end{aligned}
  > η(invX)η(outX)η(inX)η(⋆)​={outX,inX}={outX,inkotlin.Nothing}={outkotlin.Any?,inX}={outkotlin.Any?,inkotlin.Nothing}​

  ϕ({out ⁡Xout,in ⁡Xin},{out ⁡Yout,in ⁡Yin})=η−1({out ⁡LUB⁡(Xout,Yout),in ⁡GLB⁡(Xin,Yin)})
  \begin{aligned}
  &\phi(\{\operatorname{\texttt{out\,}}X_{out}, \operatorname{\texttt{in\,}}X_{in}\}, \{\operatorname{\texttt{out\,}}Y_{out}, \operatorname{\texttt{in\,}}Y_{in}\}) = \\
  & \qquad \eta^{-1} (\{\operatorname{\texttt{out\,}}\operatorname{\text{LUB}}(X_{out}, Y_{out}), \operatorname{\texttt{in\,}}\operatorname{\text{GLB}}(X_{in}, Y_{in})\})
  \end{aligned}
  ​ϕ({outXout​,inXin​},{outYout​,inYin​})=η−1({outLUB(Xout​,Yout​),inGLB(Xin​,Yin​)})​
- if A=(LA..UA)A = (L_A..U_A)A=(LA​..UA​)
  and B=(LB..UB)B = (L_B..U_B)B=(LB​..UB​)
  , LUB⁡(A,B)=(LUB⁡(LA,LB)..LUB⁡(UA,UB))\operatorname{\text{LUB}}(A, B) = (\operatorname{\text{LUB}}(L_A, L_B)..\operatorname{\text{LUB}}(U_A, U_B))LUB(A,B)=(LUB(LA​,LB​)..LUB(UA​,UB​))
- if A=(LA..UA)A = (L_A..U_A)A=(LA​..UA​)
  and BBB
  is not flexible, LUB⁡(A,B)=(LUB⁡(LA,B)..LUB⁡(UA,B))\operatorname{\text{LUB}}(A, B) = (\operatorname{\text{LUB}}(L_A, B)..\operatorname{\text{LUB}}(U_A, B))LUB(A,B)=(LUB(LA​,B)..LUB(UA​,B))

> Important: in some cases, the least upper bound is handled as described [here](https://kotlinlang.org/spec/kotlin-type-constraints.html#the-relations-on-types-as-constraints), from the point of view of type constraint system.

In the presence of recursively defined parameterized types, the algorithm given above is not guaranteed to terminate as there may not exist a finite representation of LUB⁡\operatorname{\text{LUB}}LUB
for particular two types. The detection and handling of such situations (compile-time error or leaving the type in some kind of denormalized state) is implementation-defined.

In some situations, it is needed to construct the least upper bound for more than two types, in which case the least upper bound operator LUB⁡(T1,T2,…,TN)\operatorname{\text{LUB}}(T_1, T_2, \ldots, T_N)LUB(T1​,T2​,…,TN​)
is defined as LUB⁡(T1,LUB⁡(T2,…,TN))\operatorname{\text{LUB}}(T_1, \operatorname{\text{LUB}}(T_2, \ldots, T_N))LUB(T1​,LUB(T2​,…,TN​))
.

#### Greatest lower bound

The *greatest lower bound* GLB⁡(A,B)\operatorname{\text{GLB}}(A, B)GLB(A,B)
of types AAA
and BBB
is a lower bound LLL
of AAA
and BBB
such that there is no other lower bound of these types which is greater by subtyping relation than LLL
.

> Note: GLB⁡\operatorname{\text{GLB}}GLB
> is commutative, i.e., GLB⁡(A,B)=GLB⁡(B,A)\operatorname{\text{GLB}}(A, B) = \operatorname{\text{GLB}}(B, A)GLB(A,B)=GLB(B,A)
> . This property is used in the subsequent description, e.g., other properties of GLB⁡\operatorname{\text{GLB}}GLB
> are defined only for a specific order of the arguments. Definitions following from commutativity of GLB⁡\operatorname{\text{GLB}}GLB
> are implied.

GLB⁡(A,B)\operatorname{\text{GLB}}(A, B)GLB(A,B)
has the following properties, which may be used to *normalize* it. This normalization procedure, if finite, creates a *canonical* representation of GLB.

> Important: AAA
> and BBB
> are considered to be non-flexible, unless specified otherwise.

- GLB⁡(A,A)=A\operatorname{\text{GLB}}(A, A) = AGLB(A,A)=A
- if A<:BA <: BA<:B
  , GLB⁡(A,B)=A\operatorname{\text{GLB}}(A, B) = AGLB(A,B)=A
- if AAA
  is non-nullable, GLB⁡(A,B)=GLB⁡(A!!,B!!)\operatorname{\text{GLB}}(A, B) = \operatorname{\text{GLB}}(A!!, B!!)GLB(A,B)=GLB(A!!,B!!)
- if A=T⟨KA,1,…,KA,n⟩A = T\langle K_{A,1}, \ldots, K_{A,n}\rangleA=T⟨KA,1​,…,KA,n​⟩
  and B=T⟨KB,1,…,KB,n⟩B = T\langle K_{B,1}, \ldots, K_{B,n}\rangleB=T⟨KB,1​,…,KB,n​⟩
  , GLB⁡(A,B)=T⟨ϕ(η(KA,1),η(KB,1)),…,ϕ(η(KA,n),η(KB,n))⟩\operatorname{\text{GLB}}(A, B) = T\langle \phi(\eta(K_{A,1}), \eta(K_{B,1})), \ldots, \phi(\eta(K_{A,n}), \eta(K_{B,n}))\rangleGLB(A,B)=T⟨ϕ(η(KA,1​),η(KB,1​)),…,ϕ(η(KA,n​),η(KB,n​))⟩
  , where η(T)\eta(T)η(T)
  and ϕ(X,Y)\phi(X, Y)ϕ(X,Y)
  are defined as follows:

  η(K:L<:K<:U)={out ⁡U,in ⁡L}
  \eta(K : L <: K <: U) = \{\operatorname{\texttt{out\,}}U, \operatorname{\texttt{in\,}}L\}
  η(K:L<:K<:U)={outU,inL}

  > Informally: in many cases, one may view η(T)\eta(T)η(T)
  > as follows. η(inv ⁡X)={out ⁡X,in ⁡X}η(out ⁡X)={out ⁡X,in ⁡kotlin.Nothing⁡}η(in ⁡X)={out ⁡kotlin.Any?⁡,in ⁡X}η(⋆)={out ⁡kotlin.Any?⁡,in ⁡kotlin.Nothing⁡}
  > \begin{aligned}
  > \eta(\operatorname{\texttt{inv\,}}X) &= \{\operatorname{\texttt{out\,}}X, \operatorname{\texttt{in\,}}X\} \\
  > \eta(\operatorname{\texttt{out\,}}X) &= \{\operatorname{\texttt{out\,}}X, \operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}\} \\
  > \eta(\operatorname{\texttt{in\,}}X) &= \{\operatorname{\texttt{out\,}}\operatorname{\texttt{kotlin.Any?}}, \operatorname{\texttt{in\,}}X\} \\
  > \eta(\star) &= \{\operatorname{\texttt{out\,}}\operatorname{\texttt{kotlin.Any?}}, \operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}\}
  > \end{aligned}
  > η(invX)η(outX)η(inX)η(⋆)​={outX,inX}={outX,inkotlin.Nothing}={outkotlin.Any?,inX}={outkotlin.Any?,inkotlin.Nothing}​

  ϕ({out ⁡Xout,in ⁡Xin},{out ⁡Yout,in ⁡Yin})=(η−1∘Ω)({out ⁡GLB⁡(Xout,Yout),in ⁡LUB⁡(Xin,Yin)})Ω({out ⁡A,in ⁡B})={{out ⁡A,in ⁡B}if A:>B{out ⁡A,in ⁡kotlin.Nothing⁡}if A<:B∧A≢B
  \begin{aligned}
  & {\phi(\{\operatorname{\texttt{out\,}}X_{out}, \operatorname{\texttt{in\,}}X_{in}\}, \{\operatorname{\texttt{out\,}}Y_{out}, \operatorname{\texttt{in\,}}Y_{in}\}) = } \\
  & \qquad {(\eta^{-1} \circ \Omega)} (\{\operatorname{\texttt{out\,}}\operatorname{\text{GLB}}(X_{out}, Y_{out}), \operatorname{\texttt{in\,}}\operatorname{\text{LUB}}(X_{in}, Y_{in})\}) \\
  & {\Omega(\{\operatorname{\texttt{out\,}}A, \operatorname{\texttt{in\,}}B\}) = } \\
  & \qquad
  \begin{cases}
  \{\operatorname{\texttt{out\,}}A, \operatorname{\texttt{in\,}}B\} & \text{if } A :> B \\
  \{\operatorname{\texttt{out\,}}A, \operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}\} & \text{if } A <: B \land A \not \equiv B
  \end{cases}
  \end{aligned}
  ​ϕ({outXout​,inXin​},{outYout​,inYin​})=(η−1∘Ω)({outGLB(Xout​,Yout​),inLUB(Xin​,Yin​)})Ω({outA,inB})={{outA,inB}{outA,inkotlin.Nothing}​if A:>Bif A<:B∧A​≡B​​

> Note: the Ω\OmegaΩ
> function preserves type system consistency; ∀A,B:A<:B∧A≢B\forall A, B : A <: B \land A \not\equiv B∀A,B:A<:B∧A​≡B
> , type T⟨{out ⁡A,in ⁡B}⟩T\langle \{\operatorname{\texttt{out\,}}A, \operatorname{\texttt{in\,}}B\}\rangleT⟨{outA,inB}⟩
> is the evidence of type T⟨X⟩:X<:A<:B<:XT\langle X\rangle : X <: A <: B <: XT⟨X⟩:X<:A<:B<:X
> , which makes the type system inconsistent. To avoid this situation, we overapproximate in ⁡B\operatorname{\texttt{in\,}}BinB
> with in ⁡kotlin.Nothing⁡\operatorname{\texttt{in\,}}\operatorname{\texttt{kotlin.Nothing}}inkotlin.Nothing
> when needed. Further details are available in the [“Mixed-site variance” paper](https://kotlinlang.org/spec/control--and-data-flow-analysis.html#references-1).

- if A=(LA..UA)A = (L_A..U_A)A=(LA​..UA​)
  and B=(LB..UB)B = (L_B..U_B)B=(LB​..UB​)
  , GLB⁡(A,B)=(GLB⁡(LA,LB)..GLB⁡(UA,UB))\operatorname{\text{GLB}}(A, B) = (\operatorname{\text{GLB}}(L_A, L_B)..\operatorname{\text{GLB}}(U_A, U_B))GLB(A,B)=(GLB(LA​,LB​)..GLB(UA​,UB​))
- if A=(LA..UA)A = (L_A..U_A)A=(LA​..UA​)
  and BBB
  is not flexible, GLB⁡(A,B)=(GLB⁡(LA,B)..GLB⁡(UA,B))\operatorname{\text{GLB}}(A, B) = (\operatorname{\text{GLB}}(L_A, B)..\operatorname{\text{GLB}}(U_A, B))GLB(A,B)=(GLB(LA​,B)..GLB(UA​,B))

> Important: in some cases, the greatest lower bound is handled as described [here](https://kotlinlang.org/spec/kotlin-type-constraints.html#the-relations-on-types-as-constraints), from the point of view of type constraint system.

In the presence of recursively defined parameterized types, the algorithm given above is not guaranteed to terminate as there may not exist a finite representation of GLB⁡\operatorname{\text{GLB}}GLB
for particular two types. The detection and handling of such situations (compile-time error or leaving the type in some kind of denormalized state) is implementation-defined.

In some situations, it is needed to construct the greatest lower bound for more than two types, in which case the greatest lower bound operator GLB⁡(T1,T2,…,TN)\operatorname{\text{GLB}}(T_1, T_2, \ldots, T_N)GLB(T1​,T2​,…,TN​)
is defined as GLB⁡(T1,GLB⁡(T2,…,TN))\operatorname{\text{GLB}}(T_1, \operatorname{\text{GLB}}(T_2, \ldots, T_N))GLB(T1​,GLB(T2​,…,TN​))
.

### Type approximation

As we mentioned [before](https://kotlinlang.org/spec/type-system.html#type-kinds), Kotlin type system has denotable and non-denotable types. In many cases, we need to *approximate* a non-denotable type, which appeared, for example, during type inference, into a denotable type, so that it can be used in the program. This is achieved via *type approximation*, which we describe below.

> Important: at the moment, type approximation is applied only to [intersection](https://kotlinlang.org/spec/type-system.html#intersection-types) and [union](https://kotlinlang.org/spec/type-system.html#union-types) types.

Type approximation function α\alphaα
is defined as follows.

- α(A⟨τA⟩&⁡B⟨τB⟩)=(α↓∘GLB⁡)(S⟨τA→S⟩,S⟨τB→S⟩)\alpha(A\langle \tau_A \rangle \mathbin{\operatorname{\&}}B\langle \tau_B \rangle) = (\alpha {\downarrow} \circ \operatorname{\text{GLB}})(S\langle \tau_{A \rightarrow S} \rangle, S\langle \tau_{B \rightarrow S} \rangle)α(A⟨τA​⟩&B⟨τB​⟩)=(α↓∘GLB)(S⟨τA→S​⟩,S⟨τB→S​⟩)
  , where type SSS
  is the least single common supertype of AAA
  and BBB
  , substitution τP→Q\tau_{P \rightarrow Q}τP→Q​
  is the result of chain applying substitutions from type PPP
  to type Q:>PQ :> PQ:>P
  , α↓\alpha {\downarrow}α↓
  is a function which applies type approximation function to the type arguments if needed;
- α(A⟨τA⟩∣⁡B⟨τB⟩)=α(δ(A⟨τA⟩∣⁡B⟨τB⟩))\alpha(A\langle \tau_A \rangle \mathbin{\operatorname{|}}B\langle \tau_B \rangle) = \alpha(\delta(A\langle \tau_A \rangle \mathbin{\operatorname{|}}B\langle \tau_B \rangle))α(A⟨τA​⟩∣B⟨τB​⟩)=α(δ(A⟨τA​⟩∣B⟨τB​⟩))
  , where δ\deltaδ
  is the [type decaying](https://kotlinlang.org/spec/type-system.html#type-decaying) function.

> Note: when we talk about the least **single** common supertype of AAA
> and BBB
> , we mean exactly that: if they have several unrelated common supertypes (e.g., several common superinterfaces), we continue going up the supertypes, until we find a single common supertype or reach [`kotlin.Any?`](https://kotlinlang.org/spec/built-in-types-and-their-semantics.html#kotlin.any-builtins).

### Type decaying

All [union types](https://kotlinlang.org/spec/type-system.html#union-types) are subject to *type decaying*, when they are converted to a specific [intersection type](https://kotlinlang.org/spec/type-system.html#intersection-types), representable within Kotlin type system.

> Important: at the moment, type decaying is applied only to [union](https://kotlinlang.org/spec/type-system.html#union-types) types. Note: type decaying is comparable to how *least upper bound* computation works in Java.

Type decaying function δ\deltaδ
is defined as follows.

- δ(A⟨τA⟩∣⁡B⟨τB⟩)=&⁡S∈S(A,B)(δ↓∘LUB⁡)(S⟨τA→S⟩,S⟨τB→S⟩)\delta(A\langle \tau_A \rangle \mathbin{\operatorname{|}}B\langle \tau_B \rangle) = \mathbin{\operatorname{\&}}_{S \in \mathbb{S}(A, B)} (\delta {\downarrow} \circ \operatorname{\text{LUB}})(S\langle \tau_{A \rightarrow S} \rangle, S\langle \tau_{B \rightarrow S} \rangle)δ(A⟨τA​⟩∣B⟨τB​⟩)=&S∈S(A,B)​(δ↓∘LUB)(S⟨τA→S​⟩,S⟨τB→S​⟩)
  , where substitution τP→Q\tau_{P \rightarrow Q}τP→Q​
  is the result of chain applying substitutions from type PPP
  to type Q:>PQ :> PQ:>P
  , δ↓\delta {\downarrow}δ↓
  is a function which applies type decaying function to the type arguments if needed, S(A,B)\mathbb{S}(A, B)S(A,B)
  is a set of most specific common supertypes of AAA
  and BBB
  .

> Note: a set of most specific common supertypes S(A,B)\mathbb{S}(A, B)S(A,B)
> is a reduction of a set of all common supertypes U(A,B)\mathbb{U}(A, B)U(A,B)
> , which excludes all types T∈UT \in \mathbb{U}T∈U
> such that ∃V∈U:V≠T∧V<:T\exists V \in \mathbb{U} : V \neq T \land V <: T∃V∈U:V​=T∧V<:T
> .

### References

1. Ross Tate. “Mixed-site variance.” FOOL, 2013.
2. Ross Tate, Alan Leung, and Sorin Lerner. “Taming wildcards in Java’s type system.” PLDI, 2011.
