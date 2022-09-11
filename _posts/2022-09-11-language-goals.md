---
layout: post
title: Language Goals and Philosophies
---

Alright, so after deliberating and deciding for the past few days, I have come up with a couple of design goals and design philosophies for the project. So, I shall list them now:

1. The language should be **high-level** and **general purpose**. For this, I am thinking more to the level of Python and Java (and not like extremely abstract to the level of Haskell).

2. The language should be **simple**. Features in the language should feel intuitive and not like strange exceptions.

3. The language should be **readable**. It should be easy to look at for me and other people reading code in it.<br>
In particular, I aim to make something with C-style syntax, and I want to avoid introducing symbols to syntax. So, avoid Haskell-like syntax and their `>>=`s and their `<$>`s and their `<*>`s and `++`s.

4. **Mutations should be explicit.** If something can mutate, it should be explicitly clear that it can. <br>
If a variable can be mutated, it needs to be specifically designated as a mutable variable. If a function parameter can be mutated, it needs to be specifically designated as a mutable parameter. Etc.

5. **Classifications are strict.** If some variable `s` holds a string, then `s` has all of the properties of a string. It cannot be `null`, and it cannot be an error.

## Features To Explore

I also have some design features I aim to explore (which I brushed on a bit in [the devlog setup post]({% post_url 2022-09-07-jekyll-setup %})).

### Structural Type System

In TypeScript, you can have:

```ts
interface Duck {
    age: number

    quack(): void
    sit(): void
}

let drake: Duck = {
    age: 17,
    quack() {
        console.log("quack!")
    },
    sit() {
        // ...
    }
}
```

The variable `drake` isn't initialized from any `Duck` class, but it *is* a `Duck`, because it has all the properties and methods of `Duck`.

I think there's an elegance to that and it is something I want to explore more. I will have more conventional ("conventional" meaning Java-esque) nominal types, because I think those are also necessary in some cases.

**Inspirations**: TypeScript, Rust

### Composition over Inheritance

In Rust, there are **traits**.

These traits are similar to interfaces in that they are a blueprint of what *methods* a **struct** or **enum** that implements that trait should have. Structs and enums on their own cannot be extended.

So the convention is to create traits that describe what behaviors are needed and then write an implementation of that trait for a given struct or enum.

I think this makes more sense than inheritance because when you need an object to do something, you need them to be able to do *some action*. You don't really care if they are classified in some specific way.

**Inspirations**: Rust, Haskell (typeclasses)

### Partial Application, Laziness, Purity

All of these features are commonplace in functional Haskell.

They are mainly things I want to explore lightly:

- **Partial Application**: the idea that you can create new functions by partially filling in the parameters of some function. I think it's neat. I want to put it in just because.
- **Laziness**: the idea that values should not be evaluated until they are needed. I'm not sure if I would be willing to implement this fully but I'm interested in applying this to function parameters. Sometimes, function parameters aren't used, so could there be an optimization there?
- **Pure functions**: the idea that functions should not be able to modify outside variables or their inputs. The existence of pure functions could be useful for optimizing performance and I want to see if I could do some sort of optimization there.

**Inspirations**: Haskell

### Type System of Horrors

So, there exists **dependent types**, types that depend on specific values.

This is brushed upon very lightly(?) in TypeScript with the `typeof` type, but perhaps it's possible to explore it?

Though, all the languages that have **dependent types** seem terrifying and far off, so maybe it is not the best idea. Oh well! Might as well try.
