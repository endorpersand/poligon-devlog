---
layout: post
title: Devlog setup!
---

After some annoying confusingness with Jekyll, this devlog is *mostly* ready to be used! Hooray!

As an initial post, I will discuss what I want to be doing with this project.

## Language Analysis

I want to explore programming languages, see what I like and what I don't like about languages. I have quite a few thoughts about languages already:

- **Java.** :(
- **Haskell.** Quite alien compared to languages I have seen, but I do love its laziness and pureness.

    The idea that it can optimize functions because it is known *not* to have side effects seems interesting and I want to see if I can integrate that somehow.

    I like that expressions aren't evaluated until they *need* to be. It just seems like a good idea.

    Partial application is also really cool and I want to see if I can somehow make that look *natural* and also be the default option.

```hs
double a = (2 *)

double 2
double 2 -- no need to compute again because we already computed it once
```

- **TypeScript.** The type system is... terrifyingly powerful. Nothing says that more than [HypeScript](https://github.com/ronami/HypeScript). I love the expressiveness of it and I want to make something like it.

- **Rust.** Rust's language has a lot of utilities to ensure type safety, and those features are really neat ways to express types. Some big ones I am thinking of:
  - Pattern matching
  - Objects implementing interfaces (as opposed to inheritance)

## Compilers

My main goal is to learn more about languages and type theory. But, I also want to try to use LLVM.
It would be a venture into low-level programming and something I want to try.
