---
title: "Untyped Lambda Calculus"
tags:
    - note
    - lambda calculus
    - type system
    - programming language
    - Types and Programming Languages
---

## Basics

The *lambda-calculus* embodies function definition and application it their purest form, and in the lambda-calculus, everything is a function. The syntax of the lambda-calculus consists of just three kinds of terms:

$$
#import "@preview/simplebnf:0.1.1": *

#bnf(
  Prod(
    $t$,
    annot: $sans("Terms")$,
    {
      Or[$x$][_variable_]
      Or[$λ x. t$][_abstraction_]
      Or[$t$ $t$][_application_]
    },
  ),
)
$$

Function application is left associative, and function abstraction extends as far to the right as possible. For example:

- $s space t space u$ stands for $(s space t) space u$.
- $lambda x. lambda y. x space y space x$ stands for $lambda x. (lambda y. ((x space y) space x))$.

An occurrence of the variable $x$ is said to be *bound* when it occurs in the body $t$ of an abstraction $lambda x. t$. Equivalently, we can say that $lambda x$ is a *binder* whose scope is $t$. An occurrence of $x$ is *free* if it appears in a position where it is not bound by an enclosing abstraction on $x$.

A term with no free variables is said to be *closed* or a *combinator*.

## Operational Semantics

The word "compute" means the application of functions to arguments in the lambda-calculus. The only computational step is *beta-reduction*, which is the process of substituting an argument for a parameter in the body of a function. Graphically, we write

$$
(lambda x.t_(12)) t_2 --> [x |-> t_2]t_(12)
$$

where $[x |-> t_2]t_(12)$ means the term obtained by replacing all free occurrences of $x$ in $t_(12)$ with $t_2$. A term of the form $(lambda x.t_(12))t_2$ is called a *redex* (reducible expression), and the term $[x |-> t_2]t_(12)$ is called the *contractum* of the redex.

There exists several different strategies for choosing which redex to reduce first:

- *Full beta-reduction*: Any redex can be reduced at any time.
- *Normal order*: The leftmost, outermost redex is always reduced first.
- *Call-by-name*: The leftmost, outermost redex is always reduced first, but no reduction is performed inside the body of a lambda-abstraction.
- *Call-by-value*: The leftmost, innermost redex is always reduced first and where a redex is reduced only when its right-hand side is reduced to a value.

The CBV strategy is *strict* in the sense that it always evaluates the argument of a function before applying the function, whereas the CBN strategy is *non-strict* (or *lazy*) because it does not necessarily evaluate the argument of a function before applying the function.

## Church Encoding

### Church Booleans

Define the terms `tru` and `fls` as follows:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("tru") = #lmd.display("\\t f.t")\
mono("fls") = #lmd.display("\\t f.f")
$$

The terms `tru` and `fls` can be viewed as *representing* the boolean values `true` and `false`, in the sense that we can use these terms to perform the operation of testing the truth of a boolean value:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("test") = #box[#lmd.display("\\l m n.l m n")]
$$

For example:

> [!example]
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\l m n.l m n) (\\t f.t) v w").map(lmd.display).join([\ = ])
>]]
>$$

We can also define boolean operators such as `and`, `or`, and `not` as follows:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("and") &= #box[#lmd.display("\\b c.b c (\\t f.f)")]\
mono("or") &= #box[#lmd.display("\\b c.b (\\t f.t) c")]\
mono("not") &= #box[#lmd.display("\\b.b (\\t f.f) (\\t f.t)")]
$$

> [!example]
>`and tru fls`:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\b c.b c (\\t f.f)) (\\t f.t) (\\t f.f)").map(lmd.display).join([\ = ])
>]]
>$$

> [!example]
>`or tru fls`:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\b c.b (\\t f.t) c) (\\t f.t) (\\t f.f)").map(lmd.display).join([\ = ])
>]]
>$$

> [!example]
>`not tru`:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\b.b (\\t f.f) (\\t f.t)) (\\t f.t)").map(lmd.display).join([\ = ])
>]]
>$$

### Pairs

Using the Church booleans, we can encode pairs of values as terms:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("pair") &= #box[#lmd.display("\\f s b.b f s")]\
mono("fst") &= #box[#lmd.display("\\p.p (\\t f.t)")]\
mono("snd") &= #box[#lmd.display("\\p.p (\\t f.f)")]\
$$

`pair v w` is a function that, when applied to a boolean value `b`, applies `b` to `v` and `w`. If `b` is `tru`, then the result is `v`, and if `b` is `fls`, then the result is `w`. Therefore, we can define the first and second projection functions as shown above.

>[!example]
>`fst (pair v w)`:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\p.p (\\t f.t)) ((\\f s b.b f s) v w)").map(lmd.display).join([\ = ])
>]]
>$$

### Church Numerals

We define the *Church numerals* $c_0,c_1,c_2,...$ as follows:

$$
#import "@preview/lambdabus:0.1.0" as lmd

c_0 &= #box[#lmd.display("\\s z.z")]\
c_1 &= #box[#lmd.display("\\s z.s z")]\
c_2 &= #box[#lmd.display("\\s z.s (s z)")]\
c_3 &= #box[#lmd.display("\\s z.s (s (s z))")]\
... &= ...
$$

Each number $n$ is represented by a combinator $c_n$ that takes two arguments: $s$ (successor) and $z$ (zero). The combinator $c_n$ applies the successor function $s$ to the zero value $z$ exactly $n$ times. A interesting observation is that $c_0$ is identical to the Church boolean `fls`. We can define the successor function as follows:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("scc") = #box[#lmd.display("\\n s z.s (n s z)")]
$$

The successor function takes a Church numeral $n$ and returns a new Church numeral that applies the successor function one more time than $n$ does. 

> [!example]
>$mono("scc") space c_3$:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\n s z.s (n s z)) (\\s z.s (s (s z)))").map(lmd.display).join([\ = ])
>]]
>$$

The addition of Church numerals can be performed by a term `plus` that takes two Church numerals, $m$ and $n$, and yields a Church numeral that applies the successor function $m$ times and then applies the successor function $n$ times:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("plus") = #box[#lmd.display("\\m n s z.m s (n s z)")]
$$

> [!example]
> $mono("plus") space c_2 space c_3$:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\m n s z.m s (n s z)) (\\s z.s (s z)) (\\s z.s (s (s z)))").map(lmd.display).join([\ = ])
>]]
>$$

To implement the multiplication, we see that multiplying $m$ and $n$ is equivalent to adding $m$ to zero $n$ times, i.e. $lambda m. lambda n. m space ("plus" space n) space c_0$:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("times") = #box[#lmd.display("\\m n.m (\\a b s z.a s (b s z) n) (\\s z.z)")]
$$

To implement the zero test, we need to find some appropriate pair of arguments that will give us back the information. Specifically, we must apply our numeral to a pair of terms $"zz"$ and $"ss"$ s.t. applying $"ss"$ to $"zz"$ one or more times yields $mono("fls")$, while not applying it at all yields $mono("tru")$. We can achieve this by defining $"zz"$ to be $mono("tru")$ and $"ss"$ to be $lambda x. mono("fls")$:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("iszro") = #box[#lmd.display("\\m.m (\\x.(\\t f.f)) (\\t f.t)")]
$$

> [!example]
> $mono("iszro") space c_0$:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\m.m (\\x.(\\t f.f)) (\\t f.t)) (\\s z.z)").map(lmd.display).join([\ = ])
>]]
>$$
>
> $mono("iszro") space c_1$:
>
>$$
>#import "@preview/lambdabus:0.1.0" as lmd
>
>#box[#align(left)[
>#lmd.normalization-steps("(\\m.m (\\x.(\\t f.f)) (\\t f.t)) (\\s z.s z)").map(lmd.display).join([\ = ])
>]]
>$$

The subtraction using Church numerals is a bit more complicated. It can be done using the following predecessor function, which, given $c_0$, returns $c_0$, and given $c_{n+1}$, returns $c_n$:

$$
mono("zz") &= mono("pair") space c_0 space c_0\
mono("ss") &= lambda p. mono("pair") space (mono("snd") space p) space (mono("scc") space (mono("snd") space p))\
mono("prd") &= lambda m. mono("fst") space (m space mono("ss") space mono("zz"))
$$

By utilizing the predecessor function, we can define subtraction as follows:

$$
mono("sub") = lambda m. lambda n. n space mono("prd") space m
$$

which literally means applying the predecessor function $n$ times to $m$.

With these function defined, we can also define other operations, e.g., the equality test, as follows:
$$
mono("equal") = lambda m. lambda n. mono("and") space (mono("iszro") space (mono("sub") space m space n)) space (mono("iszro") space (mono("sub") space n space m))
$$

### Lists

A list can be represented in the lambda-calculus by its $mono("fold")$ function. For example, the list $[x,y,z]$ becomes a function that takes two arguments $c$ and $n$ and returns $c space x space (c space y space (c space z space n))$. Then the representation of $mono("nil")$ would be:
$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("nil") = #box[#lmd.display("\\c n.n")]
$$

And the function $mono("cons")$ for constructing lists would be:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("cons") = #box[#lmd.display("\\h t c n.c h (t c n)")]
$$

We can construct the list $[x,y,z]$ as follows:

$$
[x,y,z] = mono("cons") space x space (mono("cons") space y space (mono("cons") space z space mono("nil")))
$$

We can intuitively think that when we apply a list to a function $c$ (combiner) and a value $n$ (base value), we are folding the list with the function $c$ and the base case $n$. The folding is a recursive process that applies the function $c$ to each element of the list and the result of folding the rest of the list. That is to say, instead of directly storing the elements, the Church encoding of lists *is* its own "eliminator", the only way to use a list is to tell it how to collapse itself. This kind of data encoding is called a *behavioral representation*.

Notice that the representation of $mono("nil")$ is identical to the Church boolean `fls`. We can define the $mono("isnil")$ function as follows:

$$
mono("isnil") = lambda l. l space (lambda h. lambda t. mono("fls")) space mono("tru")
$$

We can also define the $mono("head")$ function as follows:

$$
mono("head") = lambda l. l space (lambda h. lambda t. h) space mono("fls")
$$

The $mono("tail")$ function is a bit more complicated and requires a trick analogous to the one used to define the predecessor function:

$$
mono("tail") &= lambda l. mono("fst") space (l space (lambda x. lambda p. mono("pair") space (mono("snd") space p) space (mono("cons") space x space (mono("snd") space p))) space (mono("pair") space mono("nil") space mono("nil")))
$$

## Recursion

A term that cannot take a step under the evaluation relation is called a *normal form*. Interestingly, in the untyped lambda-calculus, some terms cannot be evaluated to a normal form. For example, the *divergent* combinator:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("omega") = #box[#lmd.display("(\\x.x x) (\\x.x x)")]
$$

Terms with no normal form are said to *diverge*.

The $mono("omega")$ combinator has a particularly interesting generalization called the *fixed-point combinator* (or *Y combinator*), which can be used to defined recursive functions:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("fix") = #box[#lmd.display("\\f.(\\x.f (\\y.x x y)) (\\x.f (\\y.x x y))")]
$$

> [!note]
> This definition of the fixed-point combinator is specifically tailored for the call-by-value evaluation strategy. For the call-by-name evaluation strategy, we can use the following simpler definition:
> $$
> #import "@preview/lambdabus:0.1.0" as lmd
>
> Y = #box[#lmd.display("\\f.(\\x.f (x x)) (\\x.f (x x))")]
> $$
> This definition is useless in a call-by-value setting, since the expression $Y space g$ diverges for any term $g$.

Suppose we want to write a recursive function $h=#box[body containing] space h$. The intention is that $h$ should be "unrolled" at the point where it occurs. This can be achieved by first defining $g=lambda f.#box[body containing]space f$ and then $h=mono("fix") g$. For example, to define the factorial function, we can first define $g$ as follows:

$$
g=lambda f.lambda n. (mono("iszro") space n) space c_1 space (mono("times") space n space (f space (mono("prd") space n)))
$$

then

$$
h = mono("fix") space g
$$

Intuitively speaking, a fixed point of a function $f$ is a value $x$ such that $f space x = x$. The fixed-point combinator is a term that, when applied to a function $f$, returns a fixed point of $f$. For recursion, $f$ is usually a "function-body generator" that takes the "recursive function so far" and returns a further unrolling of the recursive function. By applying the fixed-point combinator to $f$, we get a term that is equal to $f$ applied to itself, which is exactly what we need for recursion.

## Formalities

### Syntax

> [!info] Definition: Terms
> Let $cal(V)$ be a countable set of variable names. The set of terms is the smallest set $cal(T)$ s.t.
> 1. $x in cal(T)$ for every $x in cal(V)$;
> 2. if $t_1 in cal(T)$ and $x in cal(T)$, then $lambda x.t_1 in cal(T)$;
> 3. if $t_1 in cal(T)$ and $t_2 in cal(T)$, then $t_1 space t_2 in cal(T)$.

> [!info] Definition: Size of Terms
> The *size* of a term $t$, denoted by $"size"(t)$, is defined as follows:
> 1. $"size"(x) = 1$ for every variable $x$;
> 2. $"size"(lambda x.t_1) = 1 + $"size"(t_1);
> 3. $"size"(t_1 space t_2) = 1 + $"size"(t_1) + $"size"(t_2).

> [!info] Definition: Free Variables
> The set of *free variables* of a term $t$, denoted by $"FV"(t)$, is defined as follows:
>$$
>"FV"(x) &= {x}\
>"FV"(lambda x.t_1) &= "FV"(t_1) backslash {x}\
>"FV"(t_1 space t_2) &= "FV"(t_1) union "FV"(t_2)
>$$

### Substitution

A common pitfall when defining substitution is the so-called *variable capture* problem. For example, consider the term $lambda x. y$ and the substitution $[y |-> x]$. If we naively replace $y$ with $x$, we get $lambda x. x$, which is not the intended result. To avoid it, we need to make sure that the bound variable names of the term are kept distinct from the free variable names of the term being substituted. A substitution operation that does this correctly is called *capture-avoiding substitution*. When combined with *alpha-conversion* (renaming bound variables), we can always avoid variable capture. But to keep things simple, we will just assume that terms that differ only in the names of bound variables are interchangeable in all contexts. This yields the following definition of substitution:

> [!info] Definition: Substitution
>$$
>&[x |-> s]x &&= s\
>&[x |-> s]y &&= y &"if" space x != y\
>&[x |-> s](lambda y.t_1) &&= lambda y. [x |-> s]t_1 &"if" space x != y space "and" space y in.not "FV"(s)\
>&[x |-> s](t_1 space t_2) &&= ([x |-> s]t_1) space ([x |-> s]t_2)
>$$

### Operational Semantics

$$
#import "@preview/simplebnf:0.1.1": *
#import "@preview/curryst:0.5.1": rule, prooftree
#box[#align(left)[
_Syntax_:

#bnf(
  Prod(
    $t$,
    annot: $sans("Terms")$,
    {
      Or[$x$][_variable_]
      Or[$λ x. t$][_abstraction_]
      Or[$t$ $t$][_application_]
    },
  ),
)

#bnf(
  Prod(
    $v$,
    annot: $sans("Values")$,
    {
      Or[$λ x. t$][_abstraction value_]
    },
  ),
)

_Evaluation_:

#smallcaps[E-App1]:
#let eapp1 = rule(
	[$t_1$ $t_2->t_1^'$ $t_2$],
	[$t_1 -> t_1^'$],
)
#prooftree(stroke: 1pt + white, eapp1)

#smallcaps[E-App2]:
#let eapp2 = rule(
	[$v_1$ $t_2->v_1$ $t_2^'$],
	[$t_2 -> t_2^'$],
)
#prooftree(stroke: 1pt + white, eapp2)

#smallcaps[E-AppAbs]: $(lambda x. t_"12")v_2 -> [x |-> v_2]t_"12"$
]]
$$

There are two kinds of rules: the *computation rule* $#smallcaps[E-AppAbs]$ and the *congruence rules* $#smallcaps[E-App1]$ and $#smallcaps[E-App2]$. Notice how the choice of metavariables in the rules helps control the order of evaluation.