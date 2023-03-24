---
layout: post
title: Research - CI Bytecode
---
This is a summary document of [Crafting Interpreters: 14 - Chunks of Bytecode](https://craftinginterpreters.com/chunks-of-bytecode.html)

# Tree interpreters are too slow & inefficient
- To compute `fib(40)` recursively,
	- 72s on `jlox` interpreter
	- 0.5s in C

	- \[an obnoxious amount of time\] on Poligon's interpreter
	- 1.30s in Rust
- Optimization is hard because it's a tree
- Memory-inefficient
	- lot of pointers and things (CI refers to Java here, but this is probably applicable to my Rust tree as well)

# Alternative solutions

**Native code**
- Significantly faster
- Extremely difficult to deal with
- Highly unportable
	
**Bytecode**
- Maintains portability, but has a performance boost when compared to tree interpreters
- Linear sequence of binary instructions
	- Like native code, but less 

# Bytecode

Bytecode has parallels to a tree-traversal interpreter:
|purpose||tree-traversal interpreter|bytecode compiler|
|-|-|-|-|
|string to code representation||parser| compiler |
| code representation ||syntax tree|byte code|
|runtime || interpreter | VM |

Byte code consists of set of instructions.
Examples from this chapter:
- `OP_RETURN`: Returning from the current function
- `OP_CONSTANT [CONST_IDX]`: "loads" constant at `CONST_IDX` for use
	- `CONST_IDX` is the operand

As these are a *linear* list of instructions, it is easier to reference an instruction and assign it a line number.
# Assembler/Disassembler
**Assembler**: Program that takes human-readable CPU instructions (e.g. `ADD`, `MULT`) and converts them into binary machine code

**Disassembler**: Program that does the reverse (binary machine code -> human-readable CPU instructions)
- Akin to printing out a syntax tree

# Constants & Value Representation
For small-sized constants (ints, floats), VMs often attach a **constant pool** (an array that holds constants).
- Constants can then be referenced by index.