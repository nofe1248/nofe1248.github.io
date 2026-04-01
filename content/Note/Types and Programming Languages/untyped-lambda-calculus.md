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

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\l m n.l m n) (\\t f.t) v w").map(lmd.display).join([\ = ])
]]
$$

We can also define boolean operators such as `and`, `or`, and `not` as follows:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("and") &= #box[#lmd.display("\\b c.b c (\\t f.f)")]\
mono("or") &= #box[#lmd.display("\\b c.b (\\t f.t) c")]\
mono("not") &= #box[#lmd.display("\\b.b (\\t f.f) (\\t f.t)")]
$$

Examples:

1. `and tru fls`:

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\b c.b c (\\t f.f)) (\\t f.t) (\\t f.f)").map(lmd.display).join([\ = ])
]]
$$

2. `or tru fls`:

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\b c.b (\\t f.t) c) (\\t f.t) (\\t f.f)").map(lmd.display).join([\ = ])
]]
$$

3. `not tru`:

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\b.b (\\t f.f) (\\t f.t)) (\\t f.t)").map(lmd.display).join([\ = ])
]]
$$

### Pairs

Using the Church booleans, we can encode pairs of values as terms:

$$
#import "@preview/lambdabus:0.1.0" as lmd

mono("pair") &= #box[#lmd.display("\\f s b.b f s")]\
mono("fst") &= #box[#lmd.display("\\p.p (\\t f.t)")]\
mono("snd") &= #box[#lmd.display("\\p.p (\\t f.f)")]\
$$

`pair v w` is a function that, when applied to a boolean value `b`, applies `b` to `v` and `w`. If `b` is `tru`, then the result is `v`, and if `b` is `fls`, then the result is `w`. Therefore, we can define the first and second projection functions as shown above.

Example `fst (pair v w)`:

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\p.p (\\t f.t)) ((\\f s b.b f s) v w)").map(lmd.display).join([\ = ])
]]
$$

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

The successor function takes a Church numeral $n$ and returns a new Church numeral that applies the successor function one more time than $n$ does. Example $mono("scc") space c_3$:

$$
#import "@preview/lambdabus:0.1.0" as lmd

#box[#align(left)[
#lmd.normalization-steps("(\\n s z.s (n s z)) (\\s z.s (s (s z)))").map(lmd.display).join([\ = ])
]]
$$
