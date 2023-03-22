---
layout: post
title: Full List of PLIR Changes
---

# Intro to PLIR

**PLIR (Poligon Language Intermediate Representation)** is an intermediate step between the full Poligon code and the LLVM IR compilation.

Its purpose is to reduce responsibilities from the LLVM IR compilation (so instead of having several responsibilities in one step, we have them broken into two steps).

Additionally, PLIR plays an additional role as type checker and variable resolution for Poligon's compiler. PLIR's code generation checks types and verifies that identifiers reference *something* (a variable or function).

In the project, PLIR is represented as a syntax tree and passed onto the LLVM code generation step. There is a script form of PLIR, written in `.plir.gon` files, but the project does not parse them (**NOTE: PLIR deviates slightly from main Poligon**). It only generates the code for them.

PLIR maintains a lot of the same syntax as the main Poligon language, but it adds some new restraints:

# Blocks

Blocks have to be explicitly marked with a terminating statement (`return`, `break`, `continue`,  `exit`).
The first 3 are as you would expect:
- `return [expr]`: Exit out of the current function, output `[expr]` or `void` from the function call.
- `break`: Exit the loop the expression is currently in.
- `continue`: Exit the current iteration of the loop the expression is currently in and continue onto the next iteration.
The last is a simple exit:
- `exit [expr]`: Exit out of the current block, outputting `[expr]` or `void` from the block.
	- For a loop, this acts the same as `continue`.

There are a few cases to consider for Poligon code:
1. **Block has a terminating statement at the end.** Then, nothing is changed in PLIR.
2. **Block has a terminating statement in the middle.** Then, every statement after the terminating statement is removed and ignored.
3. **Block does not have a terminating statement.** Then, the last statement is upgraded into an `exit` statement (if it is not a function block), or a `return` statement (if it is a function block).

```rust
//// in Poligon
let hello = {
	2 + 2;
};

fun world() -> int {
	3 + 3;
}

//// in PLIR
let hello: int = <int>{
	exit <int>(<int>(2) + <int>(2));
};

fun world() -> int {
	return <int>(<int>(3) + <int>(3));
}
```

# Explicit typing

Types are implicitly resolved and made explicit in PLIR:

```rust
//// in Poligon
let a = 1;

let b = {
	2 + 2;
};

//// in PLIR
let a: int = <int>(1);
let b: int = <int>{
	exit <int>(<int>(2) + <int>(2));
};
```

## `for` IR
```rust
//// in Poligon
let b = for i in 1..10 {
	i;
};

//// in PLIR
let b: list<int> = <list<int>>(for <int>(i) in <range<int>>(1..10) <int>{
	exit <int>(i);
});
```

## `if` IR

```rust
//// in Poligon
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

//// in PLIR
let x: int = <int>(if <bool>cond <int>{
	exit <int>(2);
} else <int>{
	exit <int>(3);
});

// ERROR: cannot resolve type
let y: ? = <?>(if <bool>cond <int>{
	exit <int>(2);
} else <void>{
	exit;
});

let z: maybe<int> = <maybe<int>>(castfrom <int | void>(
	if <bool>(cond) <int>{
		exit <int>(2);
	}
));
```

# Casting

If types are implicitly cast in Poligon, they are explicitly cast in PLIR.
```rust
//// in Poligon
let str2str:   string = "hello";
let chr2str:   string = 'h';

let int2int:   int   = 3;
let int2float: float = 2;

//// in PLIR
let str2str:   string = <string>("hello");
let chr2str:   string = <string>(castfrom <char>('h'));

let int2int:   int   = <int>(3);
let int2float: float = <float>(castfrom <int>(2));
```

# Pattern matching

Pattern matching is destructured:
```rust
//// in Poligon
let [a, b, c] = [1, 2, 3];

//// (roughly) in PLIR
let _tmp: <list<int>> = [1, 2, 3];
let a: int = <int>((<list<int>>(_tmp))~[0]);
let b: int = <int>((<list<int>>(_tmp))~[1]);
let c: int = <int>((<list<int>>(_tmp))~[2]);
```

# Classes, Functions, Identifiers

Functions and classes are lifted out of any nested locations. PLIR's code generator already does the verification to confirm that the function and class is only accessed in scope, so this widening is allowed.

```rust
//// in Poligon
fun foo() {
	print("foo");
	
	fun bar() {
		print("bar");
	}

	bar();
}

// prints:
// foo
// bar

//// in PLIR
fun bar() -> void <void>{
	return <void>(
		(<(string) -> void>(print))(<string>("bar"))
	);
}
	
fun foo() -> void <void>{
	<void>(
		(<(string) -> void>(print))(<string>("foo"))
	);
	return <void>((<() -> void>(bar))());
}
```

Methods are decoupled from their classes, made into normal functions, the names of defined fields are stripped, and overloaded operators are converted into normal function calls.

```rust
//// in Poligon
class Vector3 {
    x: int, y: int, z: int

    fun self.add_Vector3(other: Vector3) -> Vector3 {
        Vector3 #{
            x: self.x + other.x,
            y: self.y + other.y,
            z: self.z + other.z
        };
    }

    fun ::new(x: int, y: int, z: int) -> Vector3 {
        Vector3 #{ x, y, z };
    }
}

fun main() {
    let v1 = Vector3::new(10, 10, 0);
    let v2 = Vector3::new(15, 10, 0);
    
    let vr = v1 + v2;
    print(vr.x);
    print(vr.y);
    print(vr.z);
}

//// (roughly) in PLIR
class Vector3 { int, int, int }

fun "Vector3::add_Vector3"(self: Vector3, other: Vector3) -> Vector3 {
	return Vector3 #{
		<int>(<int>(<Vector3>(self).0) + <int>(<Vector3>(other).0)),
		<int>(<int>(<Vector3>(self).1) + <int>(<Vector3>(other).1)),
		<int>(<int>(<Vector3>(self).2) + <int>(<Vector3>(other).2)),
	};
}

fun "Vector3::new"(x: int, y: int, z: int) -> Vector3 {
	return Vector3 #{ <int>(x), <int>(y), <int>(z) };
}

fun main() {
    let v1: Vector3 = <Vector3>(
	    <(int, int, int) -> Vector3>(Vector3::new)(<int>(10), <int>(10), <int>(0))
    );
    let v2: Vector3 = <Vector3>(
	    <(int, int, int) -> Vector3>(Vector3::new)(<int>(15), <int>(10), <int>(0))
    );
    
    let vr: Vector3 = <Vector3>(
	    <(Vector3, Vector3) -> Vector3>(Vector3::new)(<Vector3>(v1), <Vector3>(v2))
    );
    <void>(<(string) -> void>(print)(<string>(castfrom <int>(<Vector3>(vr).0))));
    <void>(<(string) -> void>(print)(<string>(castfrom <int>(<Vector3>(vr).1))));
    return <void>(<(string) -> void>(print)(<string>(castfrom <int>(<Vector3>(vr).2))));
}
```