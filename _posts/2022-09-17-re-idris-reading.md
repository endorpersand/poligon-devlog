---
layout: post
title: Research: Idris Reading
---

So, on the 11th (around a week ago, oops), I read this journal article: [Idris, a general-purpose dependently typed programming language: Design and implementation](https://doi.org/10.1017/S095679681300018X), which studied Idris, a language that's meant to be like Haskell but with *dependent types*.

I only read through the examination of the design, because the implementation part was incomprehensible to me. So, yeah...

## What I learned

- **Dependent types** are types that are based on values.
  - So, for example, the example they use is `Vect`, a linked list that keeps track of the length in the typing.

```idris
data Nat: Z | S Nat

data Vect : Nat -> Type -> Type where
    Nil : Vect Z a
    (::) : a -> Vect b a -> Vect (S k) a
```

Its syntax is based on Haskell, so this is a bit weird to understand but...

**Nat** is an algebraic data type, which can either be `Z` (which is supposed to represent 0) or `S Nat` (which is supposed to represent the successor).

- So, the representation of 4 could be written as `S (S (S (S Z)))`.

**Vect** is another algebraic data type. Something constructed by Vect would have the type `Vect n a`, where `n` is a Nat and `a` is some arbitrary type.

Vect can either be `Nil` (a zero-length Vect), or it can be some sort of construction using `::`.

`::` is an operator that concatenates together an element and a Vect to create another Vect.

So,

```idris
v = 1 :: 2 :: 3 :: Nil
```

would be similar to `[1, 2, 3]`. This would have the type `Vect 3 Int` (a Vect consisting of 3 elements of type `Int`).

It reminds me of literal types in TS and const types in Rust, but those seem much more limited in scope.

Idris's main question is seemingly to answer the question `What if Haskell had full dependent types?` (types can rely on values without any limitations)

## What I like

1. It is expressive. The idea of `Vect` makes a lot of sense.
2. From what I read, the introduction of dependent types doesn't interfere with anything else in the language. Many of the features from its main insipration, Haskell, are still there: recursively defined functions, extensive pattern matching, dataclasses. This addition of dependent types just makes things more possible.

## What I don't like

1. It has Haskell syntax, which is fine, but not my goal for this project. Haskell syntax is... a bit too out there.
2. I have no clue how to implement this. (Or if I want to?)
3. I have performance concerns. `Vect` is based on Haskell `List`, which is based on linked lists. Linked lists are obviously not the most performant, nor is the representation of natural numbers as `S (S (S (S Z)))`. I checked [Idris's docs](https://docs.idris-lang.org/en/latest/tutorial/index.html) for more information and they note that the basic operations of natural numbers (addition, multiplication, etc.) are optimized by the compiler. So it's abstraction meets reality here.
