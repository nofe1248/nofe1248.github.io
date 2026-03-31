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

## References