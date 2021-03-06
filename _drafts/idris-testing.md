---
layout: post
title: "Testing!"
date: 2019-05-08
isEditable: false
runOrEdit: true
---

# Testing!

Hello world!

This is a code block for python:

```python,norun
print("hello") # norun
```

```python
print("hello")
```

This is a code block for idris:

```idris,editable,path=proofs/naturals.idr,slice=2
module Naturals.PNat

%access public export
%default total
||| PNat is a positive natural number (one or greater). The definition is
||| the same as that of Nat.
data PNat : Type where
  ||| One
  O : PNat
  ||| Successor
  N : PNat -> PNat

%name PNat i, j, k, m, n

||| We always have x' != 1
axiom3 : (x : PNat) -> (N x) = O -> Void
axiom3 _ Refl impossible

axiom4 : Nat -> Nat
axiom4 x = ?easdfasdf
```

No path:

```idris,editable
module Naturals.PNat

%access public export
%default total
||| PNat is a positive natural number (one or greater). The definition is
||| the same as that of Nat.
data PNat : Type where
  ||| One
  O : PNat
  ||| Successor
  N : PNat -> PNat

%name PNat i, j, k, m, n

||| We always have x' != 1
axiom3 : (x : PNat) -> (N x) = O -> Void
axiom3 _ Refl impossible

axiom4 : Nat -> Nat
axiom4 x = if True then x else x
```

Non-editable:

```idris,path=proofs/naturals.idr,slice=2
module Naturals.PNat

%access public export
%default total
-- Not editable!
```

Non-runnable:

```idris,norun
module Naturals.PNat

%access public export
%default total
-- Not runnable!
```

And for rust:

```rust
fn main() {
    let number = 5;
    print!("{}", number);
}
```