# Kotlin language specification — Type inference

> Source: https://kotlinlang.org/spec/type-inference.html
> Collected: 2026-10-15
> Published: Unknown
> Scope: Complete named chapter; global table of contents omitted.
> Collection note: The source header identifies this specification as Version 1.9-rfc+0.1; later compiler behavior must be checked against current documentation and release notes.

## Type inference

Kotlin has a concept of *type inference* for compile-time type information, meaning some type information in the code may be omitted, to be inferred by the compiler. There are two kinds of type inference supported by Kotlin.

- [Local type inference](https://kotlinlang.org/spec/type-inference.html#local-type-inference), for inferring types of expressions locally, in statement/expression scope;
- [Function signature type inference](https://kotlinlang.org/spec/type-inference.html#function-signature-type-inference), for inferring types of function return values and/or parameters.

Type inference is a [type constraint](https://kotlinlang.org/spec/kotlin-type-constraints.html#kotlin-type-constraints) problem, and is usually solved by a type constraint solver. For this reason, type inference is applicable in situations when the type context contains enough information for the type constraint solver to create an [optimal constraint system solution](https://kotlinlang.org/spec/kotlin-type-constraints.html#finding-optimal-constraint-system-solution) w.r.t. type inference problem.

> Note: for the purposes of type inference, an optimal solution is the one which does not contain any free type variables with no explicit constraints on them.

Kotlin also supports flow-sensitive types in the form of [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts), which have direct effect on type inference. Therefore, we will discuss them first, before talking about type inference itself.

### Smart casts

Kotlin introduces a limited form of flow-sensitive typing called *smart casts*. Flow-sensitive typing means some expressions in the program may introduce changes to the compile-time types of variables. This allows one to avoid unneeded explicit casting of values in cases when their runtime types are guaranteed to conform to the expected compile-time types.

Flow-sensitive typing may be considered a specific instance of traditional data-flow analysis. Therefore, before we discuss it further, we need to establish the data-flow framework, which we will use for smart casts.

#### Data-flow framework

##### Smart cast lattices

We assume our data-flow analysis is run on a classic control-flow graph (CFG) structure, where most non-trivial expressions and statements are simplified and/or desugared.

Our data-flow domain is a map lattice SmartCastData⁡=Expression⁡→SmartCastType⁡\operatorname{\text{SmartCastData}}= \operatorname{\text{Expression}}\rightarrow \operatorname{\text{SmartCastType}}SmartCastData=Expression→SmartCastType
, where Expression⁡\operatorname{\text{Expression}}Expression
is any Kotlin expression and SmartCastType⁡=Type⁡×Type⁡\operatorname{\text{SmartCastType}}= \operatorname{\text{Type}}\times \operatorname{\text{Type}}SmartCastType=Type×Type
sublattice is a product lattice of smart cast data-flow facts of the following kind.

- First component describes the type, which an expression definitely **has**
- Second component describes the type, which an expression definitely **does not have**

The sublattice order, join and meet are defined as follows.

P1×N1⊑P2×N2⇔P1<:P2∧N1:>N2
P_1 \times N_1 \sqsubseteq
P_2 \times N_2
\Leftrightarrow P_1 <: P_2 \land N_1 :> N_2
P1​×N1​⊑P2​×N2​⇔P1​<:P2​∧N1​:>N2​

P1×N1⊔P2×N2=LUB⁡(P1,P2)×GLB⁡(N1,N2)P1×N1⊓P2×N2=GLB⁡(P1,P2)×LUB⁡(N1,N2)
\begin{aligned}
P_1 \times N_1 \sqcup
P_2 \times N_2
&= \operatorname{\text{LUB}}(P_1, P_2) \times \operatorname{\text{GLB}}(N_1, N_2) \\
P_1 \times N_1 \sqcap
P_2 \times N_2
&= \operatorname{\text{GLB}}(P_1, P_2) \times \operatorname{\text{LUB}}(N_1, N_2)
\end{aligned}
P1​×N1​⊔P2​×N2​P1​×N1​⊓P2​×N2​​=LUB(P1​,P2​)×GLB(N1​,N2​)=GLB(P1​,P2​)×LUB(N1​,N2​)​

> Note: a well-informed reader may notice the second component is behaving very similarly to a *negation* type. (P1&⁡¬N1)∣⁡(P2&⁡¬N2)⊑(P1∣⁡P2)&⁡(¬N1∣⁡¬N2)=(P1∣⁡P2)&⁡¬(N1&⁡N2)(P1&⁡¬N1)&⁡(P2&⁡¬N2)=(P1&⁡P2)&⁡(¬N1&⁡¬N2)=(P1&⁡P2)&⁡¬(N1∣⁡N2)
> \begin{aligned}
> (P_1 \mathbin{\operatorname{\&}}\neg N_1) \mathbin{\operatorname{|}}(P_2 \mathbin{\operatorname{\&}}\neg N_2)
> &\sqsubseteq (P_1 \mathbin{\operatorname{|}}P_2) \mathbin{\operatorname{\&}}(\neg N_1 \mathbin{\operatorname{|}}\neg N_2) \\
> &= (P_1 \mathbin{\operatorname{|}}P_2) \mathbin{\operatorname{\&}}\neg (N_1 \mathbin{\operatorname{\&}}N_2) \\
> (P_1 \mathbin{\operatorname{\&}}\neg N_1) \mathbin{\operatorname{\&}}(P_2 \mathbin{\operatorname{\&}}\neg N_2)
> &= (P_1 \mathbin{\operatorname{\&}}P_2) \mathbin{\operatorname{\&}}(\neg N_1 \mathbin{\operatorname{\&}}\neg N_2) \\
> &= (P_1 \mathbin{\operatorname{\&}}P_2) \mathbin{\operatorname{\&}}\neg (N_1 \mathbin{\operatorname{|}}N_2)
> \end{aligned}
> (P1​&¬N1​)∣(P2​&¬N2​)(P1​&¬N1​)&(P2​&¬N2​)​⊑(P1​∣P2​)&(¬N1​∣¬N2​)=(P1​∣P2​)&¬(N1​&N2​)=(P1​&P2​)&(¬N1​&¬N2​)=(P1​&P2​)&¬(N1​∣N2​)​
>
> This is as intended, as “type which an expression definitely does not have” is exactly a negation type. In smart casts, as Kotlin [type system](https://kotlinlang.org/spec/type-system.html#type-system) does not have negation types, we overapproximate them when needed.

##### Smart cast transfer functions

The data-flow information uses the following transfer functions.

[ ⁣[assume⁡(xis⁡T)] ⁣](s)=s[x→s(x)⊓(T×⊤)][ ⁣[assume⁡(x!is⁡T)] ⁣](s)=s[x→s(x)⊓(⊤×T)][ ⁣[xas⁡T] ⁣](s)=s[x→s(x)⊓(T×⊤)][ ⁣[x!as⁡T)] ⁣](s)=s[x→s(x)⊓(⊤×T)][ ⁣[assume⁡(x==⁡null)] ⁣](s)=s[x→s(x)⊓(kotlin.Nothing?⁡×⊤)][ ⁣[assume⁡(x! ⁣ ⁣=⁡null)] ⁣](s)=s[x→s(x)⊓(⊤×kotlin.Nothing?⁡)][ ⁣[assume⁡(x===⁡null)] ⁣](s)=s[x→s(x)⊓(kotlin.Nothing?⁡×⊤)][ ⁣[assume⁡(x! ⁣ ⁣==⁡null)] ⁣](s)=s[x→s(x)⊓(⊤×kotlin.Nothing?⁡)][ ⁣[assume⁡(x==⁡y)] ⁣](s)=s[x→s(x)⊓s(y),y→s(x)⊓s(y)][ ⁣[assume⁡(x! ⁣ ⁣=⁡y)] ⁣](s)=s[x→s(x)⊓swap⁡(isNullable⁡(s(y))),y→s(y)⊓swap⁡(isNullable⁡(s(x)))][ ⁣[assume⁡(x===⁡y)] ⁣](s)=s[x→s(x)⊓s(y),y→s(x)⊓s(y)][ ⁣[assume⁡(x! ⁣ ⁣==⁡y)] ⁣](s)=s[x→s(x)⊓swap⁡(isNullable⁡(s(y))),y→s(y)⊓swap⁡(isNullable⁡(s(x)))][ ⁣[x=y] ⁣](s)=s[x→s(y)][ ⁣[killDataFlow⁡(x)] ⁣](s)=s[x→(⊤×⊤)][ ⁣[l] ⁣](s)=⨆p∈predecessor(l)[ ⁣[p] ⁣](s)
\begin{alignedat}{3}
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{is}}T) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(T \times \top)]
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{!is}}T) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\top \times T)]
\\
\\
&\left[\!\left[x \operatorname{\texttt{as}}T \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(T \times \top)]
\\
&\left[\!\left[x \operatorname{\texttt{!as}}T) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\top \times T)]
\\
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{==}}null) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\operatorname{\texttt{kotlin.Nothing?}}\times \top)]
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{!\!\!=}}null) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\top \times \operatorname{\texttt{kotlin.Nothing?}})]
\\
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{===}}null) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\operatorname{\texttt{kotlin.Nothing?}}\times \top)]
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{!\!\!==}}null) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap(\top \times \operatorname{\texttt{kotlin.Nothing?}})]
\\
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{==}}y) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap s(y), \\
& && &&y \rightarrow s(x) \sqcap s(y)]
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{!\!\!=}}y) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap\operatorname{\mathit{swap}}(\operatorname{\mathit{isNullable}}(s(y))), \\
& && &&y \rightarrow s(y) \sqcap\operatorname{\mathit{swap}}(\operatorname{\mathit{isNullable}}(s(x)))]
\\
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{===}}y) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap s(y), \\
& && &&y \rightarrow s(x) \sqcap s(y)]
\\
&\left[\!\left[\operatorname{\mathit{assume}}(x \operatorname{\texttt{!\!\!==}}y) \right]\!\right](s)
&&= s[&&x \rightarrow s(x) \sqcap\operatorname{\mathit{swap}}(\operatorname{\mathit{isNullable}}(s(y))), \\
& && &&y \rightarrow s(y) \sqcap\operatorname{\mathit{swap}}(\operatorname{\mathit{isNullable}}(s(x)))]
\\
\\
&\left[\!\left[x = y \right]\!\right](s)
&&= s[&&x \rightarrow s(y)]
\\
\\
&\left[\!\left[\operatorname{\mathit{killDataFlow}}(x) \right]\!\right](s)
&&= s[&&x \rightarrow (\top \times \top)]
\\
\\
&\left[\!\left[l \right]\!\right](s)
&&= &&\bigsqcup_{p \in predecessor(l)} \left[\!\left[p \right]\!\right](s)
\end{alignedat}
​[[assume(xisT)]](s)[[assume(x!isT)]](s)[[xasT]](s)[[x!asT)]](s)[[assume(x==null)]](s)[[assume(x!=null)]](s)[[assume(x===null)]](s)[[assume(x!==null)]](s)[[assume(x==y)]](s)[[assume(x!=y)]](s)[[assume(x===y)]](s)[[assume(x!==y)]](s)[[x=y]](s)[[killDataFlow(x)]](s)[[l]](s)​​=s[=s[=s[=s[=s[=s[=s[=s[=s[=s[=s[=s[=s[=s[=​​x→s(x)⊓(T×⊤)]x→s(x)⊓(⊤×T)]x→s(x)⊓(T×⊤)]x→s(x)⊓(⊤×T)]x→s(x)⊓(kotlin.Nothing?×⊤)]x→s(x)⊓(⊤×kotlin.Nothing?)]x→s(x)⊓(kotlin.Nothing?×⊤)]x→s(x)⊓(⊤×kotlin.Nothing?)]x→s(x)⊓s(y),y→s(x)⊓s(y)]x→s(x)⊓swap(isNullable(s(y))),y→s(y)⊓swap(isNullable(s(x)))]x→s(x)⊓s(y),y→s(x)⊓s(y)]x→s(x)⊓swap(isNullable(s(y))),y→s(y)⊓swap(isNullable(s(x)))]x→s(y)]x→(⊤×⊤)]p∈predecessor(l)⨆​[[p]](s)​

where

swap⁡(P×N)=N×PisNullable⁡(s)={(kotlin.Nothing?⁡×⊤)if s⊑(kotlin.Nothing?⁡×⊤)(⊤×⊤)otherwise
\begin{alignedat}{1}
\operatorname{\mathit{swap}}(P \times N) &= N \times P
\\
\operatorname{\mathit{isNullable}}(s) &=
\left.
\begin{cases}
(\operatorname{\texttt{kotlin.Nothing?}}\times \top) & \text{if } s \sqsubseteq (\operatorname{\texttt{kotlin.Nothing?}}\times \top) \\
(\top \times \top) & \text{otherwise}
\end{cases}
\right.
\end{alignedat}
swap(P×N)isNullable(s)​=N×P={(kotlin.Nothing?×⊤)(⊤×⊤)​if s⊑(kotlin.Nothing?×⊤)otherwise​​

> Important: transfer functions for `==` and `!=` are used only if the corresponding [`equals` implementation](https://kotlinlang.org/spec/expressions.html#value-equality-expressions) is known to be equivalent to [reference equality check](https://kotlinlang.org/spec/expressions.html#reference-equality-expressions). For example, generated `equals` implementation for [data classes](https://kotlinlang.org/spec/declarations.html#data-class-declaration) is considered to be equivalent to reference equality check.

> Note: in some cases, after the CFG simplification a program location lll
> may be duplicated and associated with several locations l1,…,lNl_1, \ldots, l_Nl1​,…,lN​
> in the resulting CFG. If so, the data-flow information for lll
> is calculated as
>
> [ ⁣[l] ⁣]=⨆i=1N[ ⁣[li] ⁣]\left[\!\left[l \right]\!\right]= \bigsqcup_{i=1}^N \left[\!\left[l_i \right]\!\right][[l]]=i=1⨆N​[[li​]]

> Note: a killDataFlow⁡\operatorname{\mathit{killDataFlow}}killDataFlow
> instruction is used to reset the data-flow information in cases, when a compiler deems necessary to stop its propagation. For example, it may be used in loops to speed up data-flow analysis convergence. This is the current behaviour of the Kotlin compiler.

After the data-flow analysis is done, for a program location lll
we have its data-flow information [ ⁣[l] ⁣]\left[\!\left[l \right]\!\right][[l]]
, which contains data-flow facts [ ⁣[l] ⁣][e]=(P×N)\left[\!\left[l \right]\!\right][e] = (P \times N)[[l]][e]=(P×N)
for an expression eee
.

#### Smart cast types

The data-flow information is used to produce the smart cast type as follows.

First, smart casts may influence the compile-time type of an expression eee
(called *smart cast sink*) only if the sink is [*stable*](https://kotlinlang.org/spec/type-inference.html#smart-cast-sink-stability).

Second, for a stable smart cast sink eee
we calculate the overapproximation of its possible type.

[ ⁣[l] ⁣][e]=(P×N)⇒smartCastTypeOf⁡(e)=typeOf⁡(e)&⁡P&⁡approxNegationType⁡(N)
\left[\!\left[l \right]\!\right][e] = (P \times N)
\Rightarrow
\operatorname{\mathit{smartCastTypeOf}}(e) = \operatorname{\mathit{typeOf}}(e) \mathbin{\operatorname{\&}}P \mathbin{\operatorname{\&}}\operatorname{\mathit{approxNegationType}}(N)
[[l]][e]=(P×N)⇒smartCastTypeOf(e)=typeOf(e)&P&approxNegationType(N)

approxNegationType⁡(N)={kotlin.Any⁡if kotlin.Nothing?⁡<:Nkotlin.Any?⁡otherwise
\operatorname{\mathit{approxNegationType}}(N) =
\left.
\begin{cases}
\operatorname{\texttt{kotlin.Any}}& \text{if } \operatorname{\texttt{kotlin.Nothing?}}<: N \\
\operatorname{\texttt{kotlin.Any?}}& \text{otherwise}
\end{cases}
\right.
approxNegationType(N)={kotlin.Anykotlin.Any?​if kotlin.Nothing?<:Notherwise​

As a result, smartCastTypeOf⁡(e)\operatorname{\mathit{smartCastTypeOf}}(e)smartCastTypeOf(e)
is used as a compile-time type of eee
for most purposes (including, but not limited to, function overloading and type inference of other values).

> Note: the most important exception to when smart casts are used in type inference is direct property declaration.
>
> ```
> fun noSmartCastInInference() {
>     var a: Any? = null
>
>     if (a == null) return
>
>     var c = a // Direct property declaration
>
>     c // Declared type of `c` is Any?
>       // However, here it's smart casted to Any
> }
>
> fun <T> id(a: T): T = a
>
> fun smartCastInInference() {
>     var a: Any? = null
>
>     if (a == null) return
>
>     var c = id(a)
>
>     c // Declared type of `c` is Any
> }
> ```

Smart casts are introduced by the following Kotlin constructions.

- [Conditional expressions](https://kotlinlang.org/spec/control--and-data-flow-analysis.html#conditional-expressions-1) (`if`)
- [When expressions](https://kotlinlang.org/spec/expressions.html#when-expressions) (`when`);
- [Elvis operator](https://kotlinlang.org/spec/expressions.html#elvis-operator-expressions) (operator `?:`);
- [Safe navigation operator](https://kotlinlang.org/spec/expressions.html#navigation-operators) (operator `?.`);
- [Logical conjunction expressions](https://kotlinlang.org/spec/expressions.html#logical-conjunction-expressions) (operator `&&`);
- [Logical disjunction expressions](https://kotlinlang.org/spec/expressions.html#logical-disjunction-expressions) (operator `||`);
- [Not-null assertion expressions](https://kotlinlang.org/spec/expressions.html#not-null-assertion-expressions) (operator `!!`);
- [Cast expressions](https://kotlinlang.org/spec/expressions.html#cast-expressions) (operator `as`);
- [Type-checking expressions](https://kotlinlang.org/spec/expressions.html#type-checking-expressions) (operator `is`);
- [Simple assignments](https://kotlinlang.org/spec/statements.html#simple-assignments);
- Platform-specific cases: different platforms may add other kinds of expressions which introduce additional smart cast sources.

> Note: property declarations are not listed here, as their types are derived from initializers.

> Note: for the purposes of smart casts, most of these constructions are simplified and/or desugared, when we are building the program CFG for the data-flow analysis. We informally call such constructions *smart cast sources*, as they are responsible for creating smart cast specific instructions.

#### Smart cast sink stability

A smart cast sink is *stable* for smart casting if its value cannot be changed via means external to the CFG; this guarantees the smart cast conditions calculated by the data-flow analysis still hold at the sink. This is one of the necessary conditions for smart cast to be applicable to an expression.

Smart cast sink stability breaks in the presence of the following aspects.

- concurrent writes;
- mutable value capturing;
- separate module compilation;
- custom getters;
- delegation.

The following smart cast sinks are considered stable.

1. Immutable local or classifier-scope properties without delegation or custom getters;
2. Mutable local properties without delegation or custom getters, if the compiler can prove that they are [effectively immutable](https://kotlinlang.org/spec/type-inference.html#effectively-immutable-smart-cast-sinks), i.e., cannot be changed by external means;
3. Immutable properties of immutable stable properties without delegation or custom getters, if they are declared in the current [module](https://kotlinlang.org/spec/packages-and-imports.html#modules).

##### Effectively immutable smart cast sinks

We will call redefinition of eee
***direct*** redefinition, if it happens in the same declaration scope as the definition of eee
. If eee
is redefined in a nested declaration scope (w.r.t. its definition), this is a ***nested*** redefinition.

> Note: informally, a nested redefinition means the property has been captured in another scope and may be changed from that scope in a concurrent fashion.

We define ***direct*** and ***nested*** smart cast sinks in a similar way.

> Example:
>
> ```
> fun example() {
>     // definition
>     var x: Int? = null
>
>     if (x != null) {
>         run {
>             // nested smart cast sink
>             x.inc()
>
>             // nested redefinition
>             x = ...
>         }
>         // direct smart cast sink
>         x.inc()
>     }
>
>     // direct redefinition
>     x = ...
> }
> ```

A mutable local property PPP
defined at DDD
is considered effectively immutable at a direct sink SSS
, if there are no nested redefinitions on any CFG path between DDD
and SSS
.

A mutable local property PPP
defined at DDD
is considered effectively immutable at a nested sink SSS
, if there are no nested redefinitions of PPP
and all direct redefinitions of PPP
precede SSS
in the CFG.

> Example:
>
> ```
> fun directSinkOk() {
>     var x: Int? = 42 // definition
>     if (x != null)   // smart cast source
>         x.inc()      // direct sink
>     run {
>         x = null     // nested redefinition
>     }
> }
>
> fun directSinkBad() {
>     var x: Int? = 42 // definition
>     run {
>         x = null     // nested redefinition
>                      //   between a definition
>                      //   and a sink
>     }
>     if (x != null)   // smart cast source
>         x.inc()      // direct sink
> }
>
> fun nestedSinkOk() {
>     var x: Int? = 42     // definition
>     x = getNullableInt() // direct redefinition
>     run {
>         if (x != null)   // smart cast source
>             x.inc()      // nested sink
>     }
> }
>
> fun nestedSinkBad01() {
>     var x: Int? = 42     // definition
>     run {
>         if (x != null)   // smart cast source
>             x.inc()      // nested sink
>     }
>     x = getNullableInt() // direct redefinition
>                          //   after the nested sink
> }
>
> fun nestedSinkBad02() {
>     var x: Int? = 42     // definition
>     run {
>         x = null         // nested redefinition
>     }
>     run {
>         if (x != null)   // smart cast source
>             x.inc()      // nested sink
>     }
> }
> ```

#### Loop handling

As mentioned before, a compiler may use killDataFlow⁡\operatorname{\mathit{killDataFlow}}killDataFlow
instructions in loops to avoid slow data-flow analysis convergence. In the general case, a loop body may be evaluated zero or more times, which, combined with killDataFlow⁡\operatorname{\mathit{killDataFlow}}killDataFlow
instructions, causes the smart cast sources from the loop body to *not* propagate to the containing scope. However, some loops, for which we can have static guarantees about how their body is evaluated, may be handled differently. For the following loop configurations, we consider their bodies to be definitely evaluated *one or more* times.

- `while (true) { ... }`
- `do { ... } while (condition)`

> Note: in the current implementation, only the exact `while (true)` form is handled as described; e.g., `while (true == true)` does not work.

> Note: one may extend the number of loop configurations, which are handled by smart casts, if the compiler implementation deems it necessary.

> Example:
>
> ```
> fun breakFromInfiniteLoop() {
>     var a: Any? = null
>
>     while (true) {
>         if (a == null) return
>
>         if (randomBoolean()) break
>     }
>
>     a // Smart cast to Any
> }
>
> fun doWhileAndSmartCasts() {
>     var a: Any? = null
>
>     do {
>         if (a == null) return
>     } while (randomBoolean())
>     
>     a // Smart cast to Any
> }
>
> fun doWhileAndSmartCasts2() {
>     var a: Any? = null
>
>     do {
>         println(a)
>     } while (a == null)
>
>     a // Smart cast to Any
> }
> ```

#### Bound smart casts

In some cases, it is possible to introduce smart casting *between properties* if it is known at compile-time that these properties are *bound* to each other. For instance, if a variable `a` is initialized as a copy of variable `b` and both are [stable](https://kotlinlang.org/spec/type-inference.html#smart-cast-sink-stability), they are guaranteed to reference the same runtime value and any assumption about `a` may be also applied to `b` and vice versa.

> Example:
>
> ```
> val a: Any? = ...
> val b = a
>
> if (b is Int) {
>     // as a and b point to the same value,
>     // a also is Int
>     a.inc()
> }
> ```

In more complex cases, however, it may not be trivial to deduce that two (or more) properties point to the same runtime object. This relation is known as *must-alias* relation between program references and it is implementation-defined in which cases a particular Kotlin compiler may safely assume this relation holds between two particular properties at a particular program point. However, it must guarantee that if two properties are considered bound, it is impossible for these properties to reference two different values at runtime.

One way of implementing bound smart casts would be to divide the space of stable program properties into disjoint *alias sets* of properties, and the [analysis described above](https://kotlinlang.org/spec/type-inference.html#data-flow-framework) links the smart cast data flow information to sets of properties instead of single properties.

Such view could be further refined by considering special *alias sets* separately; e.g., an alias set of definitely non-null properties, which would allow the compiler to infer that `a?.b !== null` implies `a !== null` (for non-nullable `b`).

### Local type inference

Local type inference in Kotlin is the process of deducing the compile-time types of expressions, lambda expression parameters and properties. As previously mentioned, type inference is a [type constraint](https://kotlinlang.org/spec/kotlin-type-constraints.html#kotlin-type-constraints) problem, and is usually solved by a type constraint solver.

In addition to the types of intermediate expressions, local type inference also performs deduction and substitution for generic type parameters of functions and types involved in every expression. You can use the [Expressions](https://kotlinlang.org/spec/control--and-data-flow-analysis.html#expressions-cdfa) part of this specification as a reference point on how the types for different expressions are constructed.

> Important: additional effects of [smart casts](https://kotlinlang.org/spec/type-inference.html#smart-casts) are considered in local type inference, if applicable.

Type inference in Kotlin is bidirectional; meaning the types of expressions may be derived not only from their arguments, but from their usage as well. Note that, albeit bidirectional, this process is still local, meaning it processes one statement at a time, strictly in the order of their appearance in a scope; e.g., the type of property in statement S1S_1S1​
that goes before statement S2S_2S2​
cannot be inferred based on how S1S_1S1​
is used in S2S_2S2​
.

As solving a type constraint system is not a definite process (there may be more than one valid solution for a given [constraint system](https://kotlinlang.org/spec/kotlin-type-constraints.html#type-constraint-solving)), type inference may create several valid solutions. In particular, one may always derive a constraint A<:T<:BA <: T <: BA<:T<:B
for every free type variable TTT
, where types AAA
and BBB
are both valid solutions.

> Note: this is valid even if TTT
> is a free type variable without any explicit constraints, as every type in Kotlin has an implicit constraint kotlin.Nothing<:T<:kotlin.Any?\mathtt{kotlin.Nothing} <: T <: \mathtt{kotlin.Any?}kotlin.Nothing<:T<:kotlin.Any?
> .

In these cases an [optimal constraint system solution](https://kotlinlang.org/spec/kotlin-type-constraints.html#finding-optimal-constraint-system-solution) is picked w.r.t. local type inference.

> Note: for the purposes of local type inference, an optimal solution is the one which does not contain any free type variables with no explicit constraints on them.

### Function signature type inference

Function signature type inference is a variant of [local type inference](https://kotlinlang.org/spec/type-inference.html#local-type-inference), which is performed for [function declarations](https://kotlinlang.org/spec/declarations.html#function-declaration), [lambda literals](https://kotlinlang.org/spec/expressions.html#lambda-literals) and [anonymous function declarations](https://kotlinlang.org/spec/expressions.html#anonymous-function-declarations).

#### Named and anonymous function declarations

As described [here](https://kotlinlang.org/spec/declarations.html#function-declaration), a named function declaration body may come in two forms: an expression body (a single expression) or a [control structure body](https://kotlinlang.org/spec/statements.html#code-blocks). For the latter case, an expected return type must be provided or is assumed to be `kotlin.Unit` and no special kind of type inference is needed. For the former case, an expected return type may be provided or can be inferred using [local type inference](https://kotlinlang.org/spec/type-inference.html#local-type-inference) from the expression body. If the expected return type is provided, it is used as an expected constraint on the result type of the expression body.

> Example:
>
> ```
> fun <T> foo(): T { ... }
> fun bar(): Int = foo() // an expected constraint T' <: Int 
> // allows the result of `foo` to be inferred automatically.
> ```

#### Statements with lambda literals

Complex statements involving one or more lambda literals introduce an additional level of complexity to type inference and overload resolution mechanisms. As mentioned in the [overload resolution section](https://kotlinlang.org/spec/overload-resolution.html#overload-resolution), the overload resolution of callables involved in such statements is performed regardless of the contents of the lambda expressions and before any processing of their bodies is performed (including local type inference).

For a complex statement SSS
involving (potentially overloaded) callables C1,…,CNC_1, \ldots, C_NC1​,…,CN​
and lambda literals L1,…,LML_1, \ldots, L_ML1​,…,LM​
, excluding the bodies of these literals, they are processed as follows.

1. An empty [type constraint system](https://kotlinlang.org/spec/kotlin-type-constraints.html#kotlin-type-constraints) QQQ
   is created;
2. The overload resolution, if possible, picks candidates for C1,…,CNC_1, \ldots, C_NC1​,…,CN​
   according to the [overload resolution](https://kotlinlang.org/spec/overload-resolution.html#overload-resolution) rules;
3. For each lambda literal with unspecified number of parameters, we decide whether it has zero or one parameter based on the form of the callables and/or the expected type of the lambda literal. If there is no way to determine the number of parameters, it is assumed to be zero. If the number of parameters is determined to be one, the phantom parameter `it` is proceeded in further steps as if it was a named lambda parameter;

   > Important: the presence or absence of the phantom parameter `it` in the lambda body does not influence this process in any way.
4. For each lambda body L1,…,LNL_1, \ldots, L_NL1​,…,LN​
   , the expected constraints on the lambda arguments and/or lambda result type from the selected overload candidates (if any) are added to QQQ
   , and the overload resolution for all statements in these bodies is performed w.r.t. updated type constraint system QQQ
   . This may result in performing steps 1-3 in a recursive *top-down* fashion for nested lambda literals;

   > Important: in some cases overload resolution may fail to pick a candidate, e.g., because the expected constraints are incomplete, causing the constraint system to be unsound. If this happens, it is implementation-defined whether the compiler continues the top-down analysis or stops abruptly.
5. When the top-down analysis is done and the overload candidates are fixed, local type inference is performed on each lambda body and each statement *bottom-up*, from the most inner lambda literals to the outermost ones, processing one lambda literal at a time, with the following additions.

   - When inferring type of the return value (the last expression of a lambda body and/or the subjects for [return expressions](https://kotlinlang.org/spec/expressions.html#jump-expressions) referring to this lambda literal), the additional constraints introduced on the result type of this lambda literal are added to QQQ
     ;
   - If inference with these constraints fails, but the result type is a subtype of `kotlin.Unit`, the inference is repeated without the additional constraints on the return value;
   - The type of each lambda literal is considered to be the functional type FT⁡(P1,…,PS)→R\operatorname{\text{FT}}(P_1, \ldots, P_S) \rightarrow RFT(P1​,…,PS​)→R
     , where P1,…,PSP_1, \ldots, P_SP1​,…,PS​
     are the types of its parameters inferred from external constraints or specified in the lambda literal itself and RRR
     is the inferred type of its return value in the presence of external constraints.

The external constraints on lambda parameters, return value and body may come from the following sources:

- The (possibly overloaded) callable which uses the lambda literal as an argument;

  > Note: as overload resolution is performed before any lambda literal inference takes place, this candidate is always known before external constraints are needed;
- The expected type of the declaration which uses the lambda literal as its body or initializer.

> Examples:
>
> ```
> fun <T> foo(): T { ... }
> fun <R> run(body: () -> R): R { ... }
>
> fun bar() {
>     val x = run { 
>         run {
>             run {
>                 foo<Int>() // last expression inferred to be of type Int
>             } // this lambda is inferred to be of type () -> Int
>         } // this lambda is inferred to be of type () -> Int
>     } // this lambda is inferred to be of type () -> Int
>     // x is inferred to be of type Int
>
>     val y: Double = run { // this lambda has an external constraint R' <: Double
>         run { // this lambda has an external constraint R'' <: Double
>             foo() // this call has an external constraint T' <: Double
>                   // allowing to infer T to be Double in foo
>         }
>     }
> }
> ```

### Bare type argument inference

Bare type argument inference is a special kind of type inference where, given a type TTT
and a constructor TCTCTC
, the type arguments A0,A1…ANA_0, A_1 \ldots A_NA0​,A1​…AN​
are inferred such that TC[A0,A1…AN]<:TTC[A_0, A_1 \ldots A_N] <: TTC[A0​,A1​…AN​]<:T
. It is used together with *bare types* syntax sugar that can be employed in [type checking](https://kotlinlang.org/spec/expressions.html#type-checking-expressions) and [casting](https://kotlinlang.org/spec/expressions.html#cast-expressions) operators. The process is performed as follows.

First, let’s consider the simple case of TTT
being non-nullable, non-intersection type. Then, a simple [type constraint system](https://kotlinlang.org/spec/kotlin-type-constraints.html#kotlin-type-constraints) is constructed by introducing type variables for A0,A1…ANA_0, A_1 \ldots A_NA0​,A1​…AN​
and then solving the constraint TC[A0,A1…AN]<:TTC[A_0, A_1 \ldots A_N] <: TTC[A0​,A1​…AN​]<:T
.

If TTT
is an intersection type, the same process is performed for every member of the intersection type individually and then the resulting type argument values for each parameter AKA_KAK​
are merged using the following principle:

- If all values for a particular parameters are star-projections, the result is a star-projection;
- If some of the values are not star-projections and are strictly equal to each other, the result is one of their values;
- Else, the result is a star-projection.

If TTT
is a nullable type U?U?U?
, the steps given above are performed for its non-nullable counterpart type UUU
.

### Builder-style type inference

> Note: before Kotlin 1.7, builder-style type inference required using the [`@BuilderInference`](https://kotlinlang.org/spec/annotations.html#built-in-annotations) annotation on lambda parameters. Currently, for simple cases when there is a single lambda parameter which requires builder-style inference, this annotation may be omitted.

When working with DSLs that have generic builder functions, one may want to infer the generic builder type parameters using the information from the builder’s lambda body. Kotlin supports special kind of type inference called **builder-style type inference** to allow this in some cases.

In order to allow builder-style inference for a generic builder function and its type parameter `P`, it should satisfy the following requirements:

- It has a lambda parameter of [function type with receiver](https://kotlinlang.org/spec/type-system.html#function-types), with receiver type `T`
- The receiver type `T` uses type parameter `P` in its type arguments
- The receiver type `T` can be used as receiver for callables which can provide information about `P` via their use

> Note: using the type parameter `P` *directly* as the receiver type `T` (e.g., `fun <Q /* P */> myBuilder(builder: Q /* T */.() -> Unit)`) is not yet supported.

In essence, the builder-style inference allows the type of the lambda parameter receiver to be inferred from its usage in the lambda body. This is performed only if the standard type inference cannot infer the required types, meaning one could provide additional type information to help with the inference, e.g., via explicit type arguments, and avoid the need for builder-style inference.

If the builder-style inference is needed, for a call to an eligible function with a lambda parameter, the inference is performed [as described above](https://kotlinlang.org/spec/type-inference.html#statements-with-lambda-literals), but the type arguments of the lambda parameter receiver are viewed as *postponed* type variables till the body of the lambda expression is proceeded.

> Note: during the builder-style inference process, a postponed type variable is not required to be inferred to a concrete type.

After the inference of statements inside the lambda is complete, these postponed type variables are inferred using an additional type inference step, which takes the resulting type constraint system and tries to find the instantiation of the postponed type variables to concrete types.

If the system cannot be solved, it is a compile-time error.

Builder-style inference has the following important restrictions.

- Any attempt to use an expression with type which is a postponed type variable is a compile-time error.
- If a call needs builder-style inference for more than one lambda parameter, they all should be marked with [`@BuilderInference`](https://kotlinlang.org/spec/annotations.html#built-in-annotations) annotation. Otherwise, it is a compile-time error.

> Note: notable examples of builder-style inference-enabled functions are `kotlin.sequence` and `kotlin.iterator`. See standard library documentation for details.

> Example:
>
> ```
> fun <K, V> buildMap(action: MutableMap<K, V>.() -> Unit): Map<K, V> { ... }
>
> interface Map<K, out V> : Map<K, V> { ... }
> interface MutableMap<K, V> : Map<K, V> {
>     fun put(key: K, value: V): V?
>     fun putAll(from: Map<out K, V>): Unit
> }
>
> fun addEntryToMap(baseMap: Map<String, Number>,
>                   additionalEntry: Pair<String, Int>?) {
>    val myMap = buildMap/* <?, ?> */ { // cannot infer type arguments
>                                       // needs builder-style inference
>        putAll(baseMap)
>        // provides information about String <: K, Number <: V
>
>        if (additionalEntry != null) {
>            put(additionalEntry.first, additionalEntry.second)
>            // provides information about String <: K, Int <: V
>        }
>    }
>    // solves to String =:= K, Number =:= V
>    // ...
> }
> ```
