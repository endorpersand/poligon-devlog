---
layout: post
title: Reviewing the State of the Language & Creating an IR
---

So, after writing the interpreter for 2 months and the compiler, I do want to review some features of the language, and also create a "desugared" version of the language to make it easier to compile into LLVM IR.

This document is based on [the Poligon grammar post]({% post_url 2022-09-19-poligon-and-grammar %}).

# Language

## Basic types

### Primitive types

- **int**: An integer value. Like Python, this `int` is arbitrary precision. 
	- (REVISION: As implemented right now, it is `i64`).
- **float**: IEEE-754 float (`f64`).
- **char**: A Unicode codepoint
- **bool**: true/false

### Collection types

This is all unchanged. There will be:

- **list\<E>**: a list/array of items of type E
  - **string** is an alias for `list<char>`
- **dict\<K, V>**: a mapping that takes items of type K to type V
- **set\<E>**: a set of items of type E

#### String

For the string type, it terminates once it reaches the ending `"`.
It will pass through new lines:
```rust
"
hello!
this is all one string!
:D
"
```

There are also some escapes (for both `string` and `char`):
* `\0`: Null
* `\\`: Backslash (`\`)
* `\n`: New line
* `\t`: Tab
* `\r`: Return
* `\'`: Single-quote (`'`)
* `\"`: Double-quote (`"`)

And hex escapes: `\x70` and unicode-escapes `\u{1F97A}`.

### Other miscellaneous types

Unknown and never are the same as they are previously.

**void**:
- The value returned by a function that does not return.
- Inspiration:
	- Rust `()`
	- Python `None`
	- Kotlin `Unit`
- This should be a *passable* value. Variables (though useless) should be able to take this as a value.
- In Rust's LLVM, this is done by passing `{}` (LLVM IR, struct with 0 items) aka `()` (Rust). This could be done similarly.

**maybe\<T>**:
- An value of type `maybe<T>` *may* have a value or it might not.
	- This is essentially a *nullable* type.
- It can be initialized as either `some(obj)` or `none`.

Similarly to Rust, this would be implemented with an enumeration:
```rust
enum maybe<T> {
	some(T),
	none
}
```

ADD: However, unlike Rust, there are union types, which also have to be somewhat integrated into this type.
The type `T | ()` should be able to be implicitly cast to `maybe<T>`.

- **result\<T, E>**:
  - A value of type `result<T, E>` is an enumeration that designates:
	  - The result passed and has a value of `T`.
	  - The result failed, error produced has a value of `E`.
  - It can be initialized as either `ok(obj)` or `err(err_obj)`.

ADD: Similarly to Rust, the `?` operator propagates a `result<T, E>`.
```rust
fun foo<T, E>() -> result<T, E> {
	let a /*: T */ = fn_that_produces_err()?;
}
```

Unlike Rust, `?` propagates to the next block that has a type of `result<T, E>`:
```rust
fun foo() {
	let propa: result<T, E> = {
		let a = fn_that_produces_err()?;
		a + 1
	};
}
```

Notice that in these types of blocks, `T` automatically casts to `result<T, _>`.
Additionally, if no propagation block is found, the `?` operator errors.

### Value operations

Unchanged.

There's still a question of should concat be `|` or `+`.

## Declarations

Declarations are mostly unchanged, however there is also pattern matching:
```rust
let [a, b, c] = [1, 2, 3];
let [a, .., c] = [1, 2, 3, 4, 5, 6];
// etc.
```

These are recursive:
```rust
let [a, [b, c, [d, e]], f] = [1, [2, 3, [4, 5]], 6];
```

Assignment similarly accepts assignment but the assignment target is extended beyond just identifiers:
```rust
[a, b[0]] = [1, 2];
// a is 1
// b[0] is 2

[a, b.c.d] = [1, 2];
// a is 1
// b.c.d is 2
```

## Usability Things

Comments are the same.

## Control Flow

`if`/`else` stay the same.
In `for` and `while` loops, `break` and `continue` skip the iteration's value.

Ranges:

```rust
0..15 // from 0 (inclusive) to 15 (exclusive)
0..15 step 2 // from 0 (inclusive) to 15 (exclusive), skipping every other integer
..15 // from negative infinity to 15 (exclusive)
0..  // from 0 to positive infinity
```

ADD: These are likely to be implemented with a Range struct internally (or several, like Rust).
- These then need to be represented as an iterator.

### Functions

Functions are declared with
```rust
fun ident(a: int, b: string) -> string {
	// body
}
```

ADD: Varargs?

## Shapes & Enums & friends

These will be held off on analysis while the rest of this is implemented.

# PLIR

An IR is needed to make the conversion to LLVM IR easier. I will call it PLIR for Poligon Language IR.

It's nothing too complicated:

## Explicit typing

Types need to be explicitly resolved in the IR for every expression:
```rust
let a = 1;

let b = {
	2 + 2;
};

// deconstructs to
let a: int = <int>1;
let b: int = <int>{
	<int>2 + <int>2;
};
```

### `for` IR
```rust
let b = for i in 1..10 {
	i;
};

// deconstructs to
let b: list<int> = <list<int>>for <int>i in <range<int>>1..10 <int>{
	<int>i;
};
```

### `if` IR
- QUESTION: should an expression ever resolve into a union type?
```rust
let cond = 3;

let x = if cond {
	2;
} else {
	3;
};

// ERROR: cannot resolve type
let y = if cond {
	2;
} else {

};

let z: maybe<int> = if cond {
	2;
};

// deconstructs to
let x: int = <int>if <bool>cond <int>{
	<int>2;
} else <int>{
	<int>3;
};

// ERROR: cannot resolve type
let y: ? = <?>if <bool>cond <int>{
	<int>2;
} else <void>{

};

let z: maybe<int> = <int | void>if <bool>cond <int>{
	<int>2;
};
```

## Pattern matching

Pattern matching is destructured:
```rust
let [a, b, c] = [1, 2, 3];

// deconstructs to
let a: int = <int>1;
let b: int = <int>2;
let c: int = <int>3;
```

## Short-circuiting

Comparisons need to be unpacked into `if` statements:
```rust
// assume a, b, c, d are all () -> int
a() < b() < c() < d()

// destructures to
let _1: int = <int>a();
let _2: int = <int>b();

// note, type markers are a bit wonked up
let _cmp: bool = <bool>(_1 < _2);
<bool>if _cmp <bool>{
	let _3: int = <int>c();
	
	let _cmp_2: bool = <bool>(_2 < _3);
	<bool>if <bool>_cmp_2 <bool>{
		let _4: int = <int>d();

		<bool>(_3 < _4);
	} else <bool>{
		_cmp_2;
	}
} else <bool>{
	_cmp;
}
```

`&&` and `||` similarly are unpacked:
```rust
// assume pred1, pred2 are () -> bool
pred1() && pred2()
pred1() || pred2()

// destructures to
let _1: bool = <bool>pred1();
<bool>if <bool>_1 <bool>{
	<bool>pred2();
} else <bool>{
	<bool>_1;
}

let _2: bool = <bool>pred1();
<bool>if <bool>_2 <bool>{
	<bool>_2;
} else <bool>{
	<bool>pred2();
}
```