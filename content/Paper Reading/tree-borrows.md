---
title: "Paper Reading: Tree Borrows"
tags:
    - paper reading
    - rust
    - borrow checker
---

> This is a paper reading note for the paper [@villani2025tree].

# What is Tree Borrows and Why We Need It?

Rust already offers strong safety guarantees through its ownership-based type system and borrow checker. And the Rust compiler can also exploit these guarantees to optimize code. However, Rust also provides an `unsafe` escaping hatch that can easily invalidate such optimizations. The *Stacked Borrows* try to mitigate, but failed to support advanced features of the Rust borrow checker. The *Tree Borrows* model aims to provide a more accurate and flexible model in those `unsafe` contexts. As the name suggests, the Tree Borrows use a tree to track the relationship of references, which allows it:

- to support two-phase borrows;
- to allow reordering of reads;
- to track the range of references dynamically.

## Use Tree to Track Aliases

For example:

```rust
let mut root = 42;
let ref1 = &mut root;
let ref2 = &mut *ref1;
let ref3 = &mut root;
```

The corresponding tree is:

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`root`], 
        ([`ref1`], [`ref2`]), 
        [`ref3`]
    ))
})
$$

When a reference is derived from another reference, it becomes a child of the original reference in the tree. So a reference is defined by a pair of a memory location and an identifier that determines its corresponding node in the tree (the *tag* of the reference).

Each node in the tree tracks a state machine, and every time a memory access occurs through the reference in the tree, all references in the same tree will be updated according to the state machine. The transitions of the state machine are indexed by whether the access is a read or write, and by the relationship of the current reference and the reference being accessed. If the accessed reference is a child or is the current reference, the access is considered *local*; otherwise, it is considered *foreign*. The state machine is given as follows:

$$
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#let fw = text(red)[$arrow.t W$]
#let fr = text(red)[$arrow.t R$]
#let lw = text(purple)[$arrow.b W$]
#let lr = text(purple)[$arrow.b R$]

#diagram(
	spacing: 4em,
    node((-1,0), [```rust &mut T```]),
    edge((-1,0), (0,0), "->", dash:"dashed"),
	node((0,0), [Reserved]),
    edge((0,0), (1,0), lw, "->"),
    edge((0,0), (0,0), lr + fr, "->", bend: -120deg),
    edge((0,0), (3,-1), fw, "->", bend: 30deg),
    node((-1,1), [```rust &mut Cell<T>```]),
    edge((-1,1), (0,1), "->", dash:"dashed"),
	node((0,1), [ReservedIM]),
    edge((0,1), (0,1), lr + fr + fw, "->", bend: -120deg),
    edge((0,1), (1,0), lw, "->"),
    node((1,0), [Unique]),
    edge((1,0), (2,0), fr, "->"),
    edge((1,0), (1,0), lr + lw, "->", bend: -120deg),
    edge((1,0), (3,-1), fw, "->", bend: 20deg),
    node((2,0), [Frozen]),
    edge((2,0), (2,0), lr + fr, "->", bend: -120deg),
    edge((2,0), (4,0), lw, "->"),
    edge((2,0), (3,-1), fw, "->"),
    node((3,1), [```rust &T```]),
    edge((3,1), (2,0), "->", dash:"dashed"),
    node((3,-1), [Disabled]),
    edge((3,-1), (3,-1), fr + fw, "->", bend: 120deg),
    edge((3,-1), (4,0), lr + lw, "->", bend: 15deg),
    node((4,0), [UB]),
)
$$

## Important Use Cases

### Model Mutable References

```rust
let mut root = 42;
let x = &mut root;
*x += 1;
root = 0;
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`root`], [`x`]
    ))
})
$$

Initially, `x` is given the `Reserved` state, and after the first local write, it transitions to the `Unique` state, which allows arbitrary local R/W accesses. After the write to `root` (which is a foreign write to `x`), `x` transitions to the `Disabled` state, which forbids all accesses. This captures most of the desired semantics of mutable references.

### Two-phase Borrows with the `Reserved` State

```rust
v.push(v.len())

// The code above roughly desugars to the following code:
let v_for_push = &twophase v;
let v_for_len = &v;
let len = Vec::len(v_for_len);
Vec::push(v_for_push, len);
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`v`], [`v_for_push`], [`v_for_len`]))
})
$$

The `Reserved` state allows the state machine to capture the *reservation phase* of the two-phase borrows. During the reservation phase, arbitrary read accesses to the memory the reference points to are allowed, even via other references. After *activation* (which happens when the reference is first used for writing), the reference transitions to the `Unique` state, which allows arbitrary local R/W accesses but forbids foreign accesses.


### Enable Read-Reordering with the `Frozen` State

```rust
let mut root = 0;
let x = &mut root;
*x = 42;
```

If we try to validate this program without the `Frozen` state, we will find that reading from `x` then `root` is fine, but reading from `root` then `x` is not. This is because the read through `root` is foreign to `x`, causing it to transition to the `Disabled` state. However, with the `Frozen` state, the read through `root` is allowed, and it only causes `x` to transition to the `Frozen` state, which still allows reads through `x`. This allows more flexible read-reordering.

### Shared References

A shared reference:

- Allow local and foreign reads;
- Forbid local writes;
- Become `Disabled` after a foreign write.

which is exactly the behavior of the `Frozen` state.

### Raw Pointers

In Tree Borrows, raw pointers do not have their own distinct tags. Instead, they inherit the tag of the reference they are derived from, since raw pointer allows arbitrary aliasing:

```rust
let mut root = 42;
let ref1 = &mut root;
let ptr1 = ref1 as *mut i32;
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`root`], [`ref1,ptr1`]
    ))
})
$$

The Tree Borrows also supports dynamically extend a reference to a larger range:

```rust
let mut v = [0u8, 0];
let x = &mut v[0];
let y = (x as *mut u8).add(1);
unsafe {
    *y = 42;
}
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`v`], [`x,y`]
    ))
})
$$

The model supports the code above because `&mut v[0]` creates a permission and state machine for the new reference *everywhere* in the current allocation.

### Interior Mutability with the `ReservedIM` State

In terms of aliasing, shared references to interior mutable types are basically equivalent to raw pointers, so the model treats them as such. Mutable references to interior mutable types are indistinguishable from regular mutable references as soon as they become `Unique`. However, during their `Reserved` phase, since the possibly existing shared references permit writes, the mutable reference must also permit foreign writes (see the following example), which is captured by the `ReservedIM` state.

```rust
fn foo(c: &must Cell<i32>, n: i32) {
    *c.get_mut() = n;
}

let mut c = Cell::new(1);
let c_mut = &twophase c;
let c_shr = &c;
let val = Cell::replace(c_shr, 42);
foo(c_mut, val);
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`c`], [`c_mut`], [`c_shr`]))
})
$$

## Protectors in Tree Borrows

The Rust compiler assumes that a reference passed as function argument is unique and immutable for the duration of that function call. This allows compiler to optimize the code like the following:

```rust
fn write_and_call(x: &mut i32, opaque: impl Fn()) {
    // Since x is immutable and unique, the compiler can eliminate this write.
    *x = 13;
    opaque();
    *x = 0;
}
```

Although at the first glance Tree Borrows should be able to provide this guarantee. The actuall problem is that `opaque()` can never return:

```rust
let mut root = 42;
let ptr1 = &mut root as *mut i32;
let opaque = move || {
    let ptr2 = ptr1;
    println!("{}", unsafe { *ptr2 });
    std::process::exit(0);
};
write_and_call(unsafe { &mut *ptr1 }, opaque);
```

$$
#import "@preview/cetz:0.4.2": canvas, draw, tree

#canvas({
  import draw: *
  set-style(content: (padding: 0.5em))
  tree.tree(
    ([`root`], ([`ptr1,ptr2`], [`x`])))
})
$$

Here, `opaque` performs an access with a tag that is foreign to `x`, and then never returns. Applying the optimization will make the code print `42` instead of `13`. Also, this context does not exhibit UB: `x` transitions from `Unique` to `Frozen` due to the foreign read, and is never accessed again. To fix this, the paper proposes the introduction of *protectors*. The protectors will ensure the references do not get invalidated prematurely. This is done by making the program UB if a protected reference becomes `Disabled`. Also, when entering a function call, the references are implicitly reborrowed. The modified state machine is as follows:

$$
#import "@preview/fletcher:0.5.8" as fletcher: diagram, node, edge

#let fw = text(red)[$arrow.t W$]
#let fr = text(red)[$arrow.t R$]
#let lw = text(purple)[$arrow.b W$]
#let lr = text(purple)[$arrow.b R$]

#diagram(
	spacing: 4em,
    node((-1,0), [```rust &mut T```/```rust &mut Cell<T>```]),
    edge((-1,0), (0,0), "->", dash:"dashed"),
	node((0,0), [Reserved]),
    edge((0,0), (1,0), lw, "->"),
    edge((0,0), (0,0), lr, "->", bend: 120deg),
    edge((0,0), (0,1), fr, "->", bend: -30deg),
    edge((0,0), (2,2), fw, "->", bend: -10deg),
	node((0,1), [Reserved (conflicted)]),
    edge((0,1), (0,1), lr + fr, "->", bend: -120deg),
    edge((0,1), (2,2), lw + fw, "->", bend: -30deg),
    node((1,0), [Unique]),
    edge((1,0), (2,2), fr + fw, "->"),
    edge((1,0), (1,0), lr + lw, "->", bend: 120deg),
    node((2,0), [Frozen]),
    edge((2,0), (2,0), lr + fr, "->", bend: 120deg),
    edge((2,0), (2,2), lw + fw, "->", bend: 30deg),
    node((1,-1), [```rust &T```]),
    edge((1,-1), (2,0), "->", dash:"dashed"),
    node((2,2), [UB]),
)
$$

Also, consider the optimization that Rust compiler can perform:

```rust
fn read_write(x: *const i32, y: &mut i32) -> i32 {
    let val = unsafe { *x };
    *y = 20;    // Optimization: move write up
    val
}
```

Without `Reserved (conflicted)` state, this will introduce UB when `x` and `y` alias. Since the write to `y` will change its state to `Unique`, and therefore the subsequent foreign read through `x` will not be permitted. Thus we must ensure that if a mutable reference is written to at any point of the function call, then there was no foreign read at any point during the call. This is captured by the `Reserved (conflicted)` state. A conflicted `Reserved` reference cannoy become `Unique`, and a local write to such a reference will trigger UB.

### Implicit Accesses

By establishing the invariant that tag of references outlive the function call, optimizations that introduce *spurious reads* (speculatively reading from a location) can be safely performed. For example:

```rust
fn repeat(x: &i32, n: usize, opaque: imp Fn(i32)) {
    let val = *x;    // Optimization: speculatively read from x
    for _ in 0..n {
        opaque(*x);  // Optimization: use val instead of reading from x again
    }
}
```

However, if `n` is `0`, then the inserted read might trigger UB. To ensure that such optimizations cannot trigger UB, the paper proposes to *implicitly* read access the reference when they are reborrowed at the beginning of the function call. Also, we need implicit accesses at the expiry of the protectors, consider the following example:

```rust
fn write_opt(x: &mut i32, f: impl FnOnce(), g: impl FnOnce()) {
    // x is protected
    f();
    *x = 10;    // Optimization: write 0
    g();
    *x = 0;     // Optimization: remove write
}
```

When `g` runs, the Tree Borrows guarantee that `g` cannot observe the value of `x` since `g` cannot access to a reference/raw pointer derived from `x`. To prove that deleting the write is sound, the paper proposes the addition of *protector end semantics*: when a tag's protector expires:

- All used `Reserved` or `Frozen` locations are subject to an implicit read.
- All used `Unique` locations are subject to an implicit write.

In both cases, the access is only applied to foreign tags.

## References