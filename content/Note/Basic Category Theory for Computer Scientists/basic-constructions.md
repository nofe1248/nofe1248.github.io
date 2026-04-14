---
title: "Basic Constructions"
tags:
    - note
    - category theory
    - Basic Category Theory for Computer Scientists
---

## Categories
### Definition of Category

> [!info] Definition: Category
> A *category* $upright("C")$ comprises:
> 1. a collection of *objects*.
> 2. a collection of *arrows* (*morphisms*).
> 3. operations assigning each arrow $f$ an object $italic("dom") f$, its *domain*, and an object $italic("cod") f$, its *codomain*. We write $f:A -> B$ to show that $italic("dom") f = A$ and $italic("cod") f = B$. The collection of all arrows with domain $A$ and codomain $B$ is denoted by $upright("C")(A, B)$.
> 4. a composition operatior assigning to each pair of arrows $f$ and $g$, with $italic("dom") g = italic("cod") f$, a *composite* arrow $g compose f:italic("cod") f -> italic("cod") g$, satisfying the following *associative law*:
>   for any arrows $f:A -> B,g:B -> C,h:C -> D$ (with $A,B,C,D$ not necessarily distinct), we have $h compose (g compose f) = (h compose g) compose f$.
> 5. for each object $A$, an *identity* arrow $"id"_A:A -> A$ satisfying the following *identity law*:
>   for any arrow $f:A -> B$, we have $"id"_B compose f = f$ and $f compose "id"_A = f$.

> [!note] Remark
> Categories are defined in terms of ordinary set theory:
> - "Collections" are sets, or proper classes.
> - "Operations" are set-theoretic functions.
> - "Equality" is set-theoretic identity.

### Common Examples of Categories

An important intuition is that the objects are sets and arrows are functions. By doing this, we are *presenting* a well-known mathematical structure domain as a category:

> [!example] Example: Category $upright("Set")$
> 1. An object in $upright("Set")$ is a set.
> 2. An arrow $f:A -> B$ in $upright("Set")$ is a total function from the set $A$ into the set $B$.
> 3. For each total function $f:A -> B$, we have $italic("dom") f = A$ and $italic("cod") f = B$.
> 4. The composition of total functions $f:A -> B$ and $g:B -> C$ is the total function from $A$ to $C$ mapping each $a in A$ to $g(f(a)) in C$. Composition of total functions is associative.
> 5. For each set $A$, the identity function $"id"_A:A -> A$ is the total function with domain and codomain $A$. For any total function $f:A -> B$, we have $"id"_B compose f = f$ and $f compose "id"_A = f$.

By this we see that $upright("Set")$ is indeed a category. But observe one small subtlety: the function $f(x)=x^2$ can corresponds to many arrows in the category. To be rigorous, we can define the arrows in $upright("Set")$ as a tuple $(f,B)$, where $f$ is a total function with domain $A$ and $B$ is a set containing $f$'s image.

> [!example] Example: Category $upright("Poset")$ 
> 1. An object in $upright("Poset")$ is a partially ordered set, i.e., a set $P$ equipped with a reflexive, transitive, antisymmetric relation $scripts(<=)_P$.
> 2. An arrow $f:(P,scripts(<=)_P) -> (Q,scripts(<=)_Q)$ in $upright("Poset")$ is a order-preserving total function from $P$ to $Q$.
> 3. For each total order-preserving function $f$ with domain $P$ and codomain $Q$, we have $italic("dom") f = (P,scripts(<=)_P)$, $italic("cod") f = (Q,scripts(<=)_Q)$, and $f in upright("Poset")((P,scripts(<=)_P),(Q,scripts(<=)_Q))$.
> 4. The composition of two total order-preserving functions $f:P -> Q$ and $g:Q -> R$ is a total function $g compose f$ from $P$ to $R$. Furthermore, if $p scripts(<=)_P p'$ then, since $f$ is order-preserving, $f(p) scripts(<=)_Q f(p')$; and since $g$ is also order-preserving, $g(f(p)) scripts(<=)_R g(f(p'))$. Thus, $g compose f$ is also order-preserving. Composition of total order-preserving functions is associative.
> 5. For each partial order $(P,scripts(<=)_P)$, the identity function $"id"_P$ preserves the ordering on $P$ and satisfies the equation of the identity law.

> [!example] Example: Category $upright("Mon")$
> A *monoid* $(M,dot,e)$ is an underlying set $M$ equipped with a binary operation $dot$ from pairs of elements of $M$ into $M$ s.t. $(x dot y) dot z=x dot (y dot z)$ for all $x,y,z in M$ and a distinguished element $e$ s.t. $e dot x=x=x dot e$ for all $x in M$. A *monoid homomorphism* from $(M,dot,e)$ to $(M',dot',e')$ is a function $f:M -> M'$ s.t. $f(e)=e'$ and $f(x dot y)=f(x) dot' f(y)$.
> 
> The category $upright("Mon")$ has monoids as objects and monoid homomorphisms as arrows. It is easy to verify that $upright("Mon")$ is indeed a category.

> [!example] Example: Category $upright("Omega-Alg")$
> Let $Omega$ be a set of operator symbols, equipped with a mapping $"ar":Omega -> bb(N)$ indicating the *arity* of $omega in Omega$. An $Omega$*-algebra* is a set $|A|$ (the *carrier* of $A$) and, for each operator $omega$ a arity $"ar"(omega)$, a function $a_omega:|A|^("ar"(omega)) -> |A|$ (the *interpretation* of $omega$). An $Omega$*-homomorphism* from an $Omega$-algebra $A$ to an $Omega$-algebra $B$ is a function $h:|A| -> |B|$ s.t. for each $omega in Omega$ and tuple $(x_1,x_2,...,x_("ar"(omega)))$ of elements of $|A|$, we have
> $$
> h(a_omega (x_1,x_2,...,x_("ar"(omega)))) = b_omega (h(x_1),h(x_2),...,h(x_("ar"(omega)))),
> $$
> The category $upright("Omega-Alg")$ has $Omega$-algebras as objects and $Omega$-homomorphisms as arrows. It is easy to verify that $upright("Omega-Alg")$ is indeed a category.

In all above examples, the objects can be viewed as "sets with structure" and the arrows as "structure preserving maps". Such categories are called *concrete categories*. Some examples:

| Category             | Objects                         | Arrows                 |
|----------------------|---------------------------------|------------------------|
| $upright("Set")$        | sets                            | total functions        |
| $upright("Pfn")$        | sets                            | partial functions      |
| $upright("FinSet")$     | finite sets                     | finite total functions |
| $upright("Mon")$        | monoids                         | monoid homomorphisms   |
| $upright("Poset")$      | posets                          | monotone functions     |
| $upright("Grp")$        | groups                          | group homomorphisms    |
| $upright("Omega-Alg")$ | algebras with signature $Omega$ | $Omega$-homomorphisms  |
| $upright("CPO")$        | complete partial orders         | continuous functions   |
| $upright("Vect")$       | vector spaces                   | linear transforms      |
| $upright("Met")$        | metric spaces                   | contraction maps       |
| $upright("Top")$        | topological spaces              | continuous functions   |

A few useful *finite categories*:

> [!info] Definition: Category $0$
> The category $0$ has no objects and no arrows. The identity and associativity laws are thus vacuously satisfied.

> [!info] Definition: Category $1$
> The category $1$ has one object and one arrow. The identity and associativity laws are thus trivially satisfied.

> [!info] Definition: Category $2$
> The category $2$ has two objects $A,B$, two identity arrows, and one arrow $f$ from $A$ to $B$.

> [!info] Definition: Category $3$
> The category $3$ has three objects $A,B,C$, three identity arrows, and three arrows $f:A -> B$ and $g:B -> C$, and $h:A -> C$.

We can present the categories $2, 3$ as the followng graphs:

Category $2$:
$$
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#diagram(spacing: 2cm, {
 let (A, B) = ((0,0), (1,0))
 node(A, $A$)
 node(B, $B$)
 edge(A, B, $f$, "->")
 edge(A, A, $"id"_A$, "->", bend: 130deg, loop-angle: 90deg)
 edge(B, B, $"id"_B$, "->", bend: 130deg, loop-angle: 90deg)
})
$$

Category $3$:

$$
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#diagram(spacing: 2cm, {
 let (A, C, B) = ((0,0), (2,0), (1,-1))
 node(A, $A$)
 node(B, $B$)
 node(C, $C$)
 edge(A, B, $f$, "->")
 edge(B, C, $g$, "->")
 edge(A, C, $h$, "->")
 edge(A, A, $"id"_A$, "->", bend: -130deg, loop-angle: 90deg)
 edge(B, B, $"id"_B$, "->", bend: 130deg, loop-angle: 90deg)
 edge(C, C, $"id"_C$, "->", bend: -130deg, loop-angle: 90deg)
})
$$

We can obtain a different sort of category by considering an *individual* algebraic structure as a category. For example, a poset $(P,<=)$ gives rise to a category whose objects are the elements of $P$. Between each pair of objects $p$ and $p'$ with $p<=p'$, there is a single arrow representing this fact. If we regard posets as categories, then $upright("Poset")$ is a category of categories.

### Category Theory in Computer Science

Some applications of category theory in computer science:

1. We can call the objects in a arbitrary category *formulas* and the arrows *proofs*. An arrow $f:A -> B$ is viewed as a proof of the logical implication $A => B$. In particular, the identity arrow $"id"_A:A->A$ is an instance of the reflexivity axiom, and the composition of arrows:
$$
#import "@preview/curryst:0.6.0": rule, prooftree, rule-set

#prooftree(rule(
    [$f:A->B$],
    [$g:B->C$],
    [$g compose f:A->C$],
))
$$
  is a rule of inference asserting the transitivity of implication.
  
2. Objects of a category can also be viewed as the *types* (also sees [[./Note/Types-and-Programming-Languages/simply-typed-lambda-calculus.md|Simply Typed Lambda Calculus]]) of a functional programming language. Consider a simple functional programming language with primitive types:
  - $mono("Int")$ for integers
  - $mono("Real")$ for real numbers
  - $mono("Bool")$ for booleans
  - $mono("Unit")$ for a one-element unit type
  
  and built-in operations:
  - $mono("iszero:Int" -> "Bool")$ for testing if an integer is zero
  - $mono("not:Bool" -> "Bool")$ for negating a boolean
  - $mono("succ"_"Int" ":Int" -> "Int")$ for incrementing an integer
  - $mono("succ"_"Real" ":Real" -> "Real")$ for incrementing a real number
  - $mono("toReal:Int" -> "Real")$ for converting an integer to a real number

  and constants:
  - $mono("zero:Int")$
  - $mono("true:Bool")$
  - $mono("false:Bool")$
  - $mono("unit:Unit")$

The corresponding category $upright("FPL")$ is built by
1. taking the types to be objects;
2. taking the operations to be arrows;
3. taking the constants to be arrows from the $mono("Unit")$ object to their corresponding type;
4. adding arrows for the identity functions;
5. for every composable pair of arrows, adding an arrow for their composition;
6. equating certain arrows, e.g. $mono("false")$ and $mono("not") compose mono("true")$, that represent the same functions according the the semantics.

The diagram is as follows (identities and compositions are omitted for clarity):

$$
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#diagram(spacing: 2cm, {
 let (B, I, U, R) = ((0,0), (1,-0.8),(1,0.8),(2,0))
 node(B, $mono("Bool")$)
 node(I, $mono("Int")$)
 node(U, $mono("Unit")$)
 node(R, $mono("Real")$)
 edge(B, B, $mono("not")$, "->", bend: 130deg, loop-angle: 150deg)
 edge(I, I, $mono("succ"_"int")$, "->", bend: 130deg, loop-angle: 90deg)
 edge(U, U, $mono("unit")$, "->", bend: -130deg, loop-angle: 90deg)
 edge(R, R, $mono("succ"_"real")$, "->", bend: -130deg, loop-angle: -150deg)
 edge(U, B, $mono("true")$, "->", shift: -3pt)
 edge(U, B, $mono("false")$, "->", shift: 3pt, label-side: left)
 edge(U, I, $mono("zero")$, "->")
 edge(I, B, $mono("iszero")$, "->")
 edge(I, R, $mono("toReal")$, "->")
})
$$

### Categories from Categories
> [!info] Definition: Dual Category
> For each category $upright("C")$, the objects of the *dual category* $upright("C")^"op"$ are the same as those of $upright("C")$; the arrows in $upright("C")^"op"$ are the opposites of the arrows in $upright("C")$. Composite and identity arrows are defined in the obvious way.

Most definitions comse in pairs, e.g., product/coproduct, equalizer/coequalizer, monomorphism/epimorphism, effect/coeffect, with a "co-x" in a category $upright("C")$ being the same thing as an $x$ in $upright("C")^"op"$

Moreover, any statement about categories can be transformed into a dual statement $S^"op"$ by exchaning "domain" and "codomain" and replacing each composite $g compose f$ by $f compose g$. If $S$ is true for $upright("C")$, then by definition $S^"op"$ is true of $upright("C")^"op"$. The is known as the *duality principle*.

> [!info] Definition: Product Category
> For any pair of categories $upright("C")$ and $upright("D")$, the *produce category* $upright("C") times upright("D")$ has as objects pairs $(A,B)$ of a $upright("C")$-object $A$ and a $upright("D")$-object $B$ and as arrows pairs $(f,g)$ of a $upright("C")$-arrow $f$ and a $upright("D")$-arrow $g$. Composition and identity arrows are defined pairewise.

> [!info] Definition: Category of Arrows
> $upright("C")^->$ is the *category of arrows* of $upright("C")$. The objects of $upright("C")^->$ are the arrows in $upright("C")$. An arrow in $upright("C")^->$ from $f:A->B$ to $f':A'->B'$ is defined to be a pair $(a,b)$ of $upright("C")$-arrows $a:A->A'$ and $b:B->B'$ s.t. $f' compose a = b compose f$. The composition of $upright("C")^->$-arrows
> $$
> (a,b):(f:A->B)->(f':A'->B')
> $$
> and
> $$
> (a',b'):(f':A'->B')->(f'':A''->B'')
> $$
> is defined to be $(a',b') compose (a,b)=(a' compose a, b' compose b)$.

> [!info] Definition: Subcategory
> A category $upright("B")$ is a *subcategory* of $upright("C")$ if
> 1. each object of $upright("B")$ is an object of $upright("C")$;
> 2. for all $upright("B")$-objects $B$ and $B'$, $upright("B")(B,B') subset.eq upright("C")(B,B')$;
> 3. composition and identity arrows in $upright("B")$ are those in $upright("C")$.