---
layout: post
title: Poligon's Grammar (pt. 1)
---

So, I wanted to talk about the language structure (or rather, just formalize my notes from 09/08-09/11), and also just discuss the name of the language.

# Language

## Basic types

### Primitive types

- **int**: An integer value. Like Python, this `int` is arbitrary precision.
- **float**: IEEE-754 float
- **char**: A Unicode codepoint
- **bool**: true/false

They would be written as so:

```java
/* int:   */ 1, 2, 14, 944, 32042
/* float: */ 1.1, 232.2, 1., 494.
/* char:  */ 'z', '@', 'α', '🙃'
/* bool:  */ true, false
```

### Collection types

There will *at least* be:

- **list\<E>**: a list/array of items of type E
  - **string** is an alias for `list<char>`
- **dict\<K, V>**: a mapping that takes items of type K to type V
- **set\<E>**: a set of items of type E

which could be initialized as:

```kotlin
// list<int>
[1, 2, 3, 4]

// string == list<char>
"hello!"

// dict<string, int>
dict {
    "a": 1,
    "b": 2,
}

// set<int>
set: set { 1, 2, 3, 4 }
```

The `dict` and `set` are placed to avoid possible ambiguity later on with other structures using `{}`.

### Other miscellaneous types

- **unk**: Top/universal type (similar to TypeScript `unknown`)
  - Any value is `unk`
- **never**: Bottom type (similar to TypeScript `never`)
  - No values are `never`. Nothing is type `never`.
- **void**: Unit type
  - The return type of functions that don't return anything.

- **maybe\<T>**:
  - An value of type `maybe<T>` *may* have a value or it might not.
    - This is essentially a *nullable* type.
  - It can be initialized as either `some(obj)` or `none`.
- **result\<T, E>**:
  - An value of type `result<T, E>` *may* have a value or the computation of the result may have caused an error.
  - It can be initialized as either `ok(obj)` or `err(err_obj)`.

### Value operations

Numeric operations:

- Unary plus: `+a`
- Addition: `a + b`
- Unary minus: `-a`
- Subtraction: `a - b`
- Multiplication: `a * b`
- Float division: `a / b`
  - `int / int == float`
- Modulo: `a % b`
  - This follows Python's `%`, i.e. the result must always satisfy: `0 <= (a % b) < b`
- Floor division: `a.floor_div(b)`
  - `int / int == int`
  - `float / float == float`
- Exponentiation: `a.pow(b)`

Bitwise operations:

- Bitwise or: `a | b`
- Bitwise and: `a & b`
- Bitwise not: `~a`
- Bitwise xor: `a ^ b`

Logical operations:

- Logical or: `a || b`
- Logical and: `a && b`
- Logical not: `!a`

Collection operations:

- Concatenation: `a | b`
- Indexing: `a[k]`
  - Not valid for sets
- Set operations:
  - Intersection: `a & b`
  - Symmetric difference: `a ^ b`

Comparison operations:

`a < b`, `a > b`, `a <= b`, `a >= b`, `a != b`, `a == b`

- Operations can be chained like in Python:
  - `(a < b < c) == (a < b) && (b < c)`

## Declarations

Declarations in their simplest form:

```rust
let variable = 15;
```

You can declare a few properties of a variable: the variable's type, the variable's assignability, and the value's mutability.

- **Type**: What type of value does this variable store?
- **Reassignability**: Can we change what value this variable is assigned to?
- **Mutability**: Can the value of the variable change?

```rust
// Difference in type:
let nineteen: int = 19;
let twenty: float = 20.0;
let truth = true; // implicit type 'bool'

// Difference in reassignability:
let a = 14;      // a can be reassigned
const pi = 3.14; // pi CANNOT be reassigned

// Difference in value mutability:
let abc = [1, 2, 3, 4];      // abc[0] = 1 is an error
let mut abcd = [1, 2, 3, 4]; // abc[0] = 1 is ok
```

Reassignability and mutability can be mixed and matched:

```rust
let red = [0, 0, 0, 0];          // reassignable, but immutable
let mut blue = [1, 1, 1, 1];     // reassignable and mutable
const green = [2, 2, 2, 2];      // non-reassignable and immutable
const mut yellow = [3, 3, 3, 3]; // non-reassignable, but mutable
```

## Usability Things

Some of these are straightforward with JS/Python experience:

```rust
let x = 14; // <-- semicolons required
let y = 19;

// assigning a value
x = 15;
x = 16;

// destructuring
[x, y] = [20, 20];
let [a, b, ..c] = [1, 2, 3, 4, 5]; // a = 1, b = 2, c = [3, 4, 5]
// there may be more pattern matching here later?

// comments :)

// single-line comments
// unlike JS, '-->' is not a valid single-line comment

/*
    multi-line comments, for multiple lines!
*/

/* 
  /* Nested multi-lines are okay too. */ 
*/
```

## Control Flow

`if`/`else`/`if else`, `while`, and `for` are similar to Rust's syntax, which work similarly to Python's:

```rust
if condition {
    // do something if condition is true
} else if condition2 {
    // do something if condition2 is true
} else {
    // do something if neither condition or condition2 are true
}

while condition {
    // repeat until condition is false
}

for i in iter {
    // i is an item of 'iter'. run this loop until you've gone through every item in iter.
}
```

As well, this inherits Rust's `if/else` by allowing it to be used as an expression:

```rust
let y = if a < b { 3 } else { 2 };

// in Rust, you can also do:
let y = {
    let mut z = 15;
    z = z + 14;
    z
};
// y = 29, z does not exist
```

It does extend Rust's functionality, though:

```rust
let y = if a < b { 3 }; 
// error: cannot determine type of 'y'
// cannot cast void to int

let y: maybe<int> = if a < b { 3 };

// while loops and for loops can be treated as lists:
let i = 1;

let z = while i % 7 == 0 {
    2 * i
};
let z = for i in 1..7 {
    2 * i
}
// z = [2, 4, 6, 8, 10, 12]
```

Ranges:

```rust
0..15 // from 0 (inclusive) to 15 (exclusive)
0..15 step 2 // from 0 (inclusive) to 15 (exclusive), skipping every other integer
..15 // from negative infinity to 15 (exclusive)
0..  // from 0 to positive infinity
```

### Functions

```kt
fun name(a: int, b: string) -> string;
```

## Shapes & Classes

Shapes act similarly to interfaces in TypeScript (or Java, but moreso, TypeScript).
They are the structural type system component of the language.

If an object has all the properties of the shape, then it *fits* that shape.

```_
shape Duck {
    age: float;
    fun quack();
    fun sit();
}
```

Anything that has a float `age`, can `quack`, and can `sit` is a `Duck`!

```_
let drake: Duck = #{ // <- declare arbitrary object with given properties
    age: 1.1,
    fun quack() {
        // ...
    }
    fun sit() {
        // ..
    }
}
```

This would be useful, for example, to generally describe a 2D vector:

```_
shape Vector2<N> {
    x: N,
    y: N
}
```

Nominal types also appear as well:

```_
class Complex {
    x: float,
    y: float,

    fun conj() -> Complex { /* ... */ }

    // ...
}
```

### Fitting

There are two types of fitting: You can assert that a class fits a shape in the class declaration OR you can implement the shape for the class itself.

```_
// assertion
class Complex fits Vector2<float> {
    // ...
}

// implement
class Drake {
    // ...
}

fit Drake to Duck {
    age = 99;

    fun quack() { /* ... */ }
    fun sit() { /* ... */ }
}
```

Shapes have a parallel for these two actions, as well:

```_
// assertion: any class/object that fits Duck must also fit Animal
shape Duck extends Animal {
    // ...
}

// implement: for every Duck, implement Animal. 
// This means that there already will be a definition provided 
// if a class fits Duck.
fit Duck to Animal {
    // ...
}
```

Note that classes don't support inheritance, but shapes do.

### Enums

Enums take the form that they do in Rust:

```_
enum IP {
    V4(int, int, int, int),
    V6(int, int, int, int, int, int)
}
```

This would be declarable as `IP::V4(0,0,0,0)` or `IP::V6(0,0,0,0,0,0)`.

### Units & Measures

(SYNTAX TENTATIVE)

Primitive types have two nominal variant: units and measures.

A unit is a value for a primitive that cannot be substituted for that primitive:

```_
unit Permissions of int {
    suffix: "p";
};

fun chmod(p: Permissions) {
    // ...
}

chmod(438p);
chmod(438); // error
```

A measure is a similar to a real world unit:

```_
measure m of float {}
measure cm of float {
    1 m = 100 cm;
}
```

## Union & Intersection Types

(This section was written: 09/20/2022)

Heavily inspired by TypeScript.

Let's say we have two types: `A`, `B`.
These types are either classes, shapes, `unk`, `never`, a primitive, a collection, whatever.

`A | B`: **Union type**

- The object provided is either type `A` OR type `B`.

`A & B`: **Intersect type**

- The object provided is both type `A` and type `B`.

Example:

```ts
let o: string | int = 14;
o = "14";

shape Vector2 {
    x: float,
    y: float
}
shape PolarVector2 {
    r: float,
    theta: float
}

let a: Vector2 & PolarVector2 = #{
    x: 3,
    y: 4,
    r: 5,
    theta: 0.927295218
};
```

Note that:

```ts
A | unk   = unk
A | never = A

A & unk   = A
A & never = never
```

# Naming

The name! Poligon!

It basically comes from how I want to base this language more on structural typing. The structural typing is called `shape`s, sooo polygon? poligon? yeah.
