---
layout: post
title: Poligon's Grammar (pt. 2)
---

# Grammar

Okay, time to formalize the grammar! Or at least, everything that doesn't involve classes/shapes.

There's no formal grammar system I need to follow. I could use:

- [EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)
- [Python's grammar specification](https://docs.python.org/3/reference/grammar.html)
- Something entirely different

So, I'll just label how I want to do grammar now:

|usage|definition|
|-|-|
|definition|`=`|
|concatenation|*tokens spaced apart*|
|termination|`;`|
|alternation|<code>&#124;</code>|
|optional|`( ... )?`|
|repetition (0+)|`( ... )*`|
|repetition (1+)|`( ... )+`|
|repetition (with sep `,`)<br>(trailing optional)|`( ... ),+`|
|grouping|`( ... )`|
|terminal string|`" ... "`|
|terminal string|`' ... '`|
|comment|`/* ... */`|
|regex char class |`[ ... ]`|

## Operators

| operator | desc | precedence | associativity |
|-|-|-|-|
|-|-|first|-|
|literals, `call()`, `a.b`, `a::b` ||...|left|
|`!`, `~`, `-`, `+`|unary operators|...|left|
|`*`, `/`, `%`|binary mul, div|...|left|
|`+`, `-`|binary add, sub|...|left|
|`<<`, `>>`|binary shl, shr|...|left|
|`&`|bitwise AND|...|left|
|`^`|bitwise XOR|...|left|
|<code>&#124;</code>|bitwise OR|...|left|
|`a..b`|range|...|left|
|`..a`|spread|...|left|
|`<`, `<=`, `>`, `>=`, `==`, `!=`|comparison|...|left|
|`&&`|logical AND|...|left|
|<code>&#124;&#124;</code>|logical OR|...|left|
|-|-|last|-|

## Tree

I'm following [Crafting Interpreters section 6.1](https://craftinginterpreters.com/parsing-expressions.html#ambiguity-and-the-parsing-game) + [Python's grammar specification](https://docs.python.org/3/reference/grammar.html) for this tree.

```ebnf
program = ( statement ";" )* ; /* must have trailing ; */
block = "{" program "}" ;

statement = 
    | declaration 
    | "return" expression
    | "break"
    | "continue"
    | fun_declaration
    | expression ;

declaration = ("let" | "const") ("mut")? ident (":" ty)? "=" expression ;
fun_declaration = "fun" ident "(" fun_params ")" ("->" ty)? block ;
fun_params = ( ("mut")? ident (":" ty)? ),* ;

ty = ident ("<" (ty),+ ">")? ;

literal = char | string | numeric | bool ;
numeric = 
    | [0-9]+
    | [0-9]+.[0-9]* ;
bool = "true" | "false" ;

/* expressions */
unit =
    | ident
    | literal
    | list
    | set
    | dict
    | block
    | if
    | while
    | for
    | "(" expression ")" ;
path = /* a chain of calls: a.b.c.d::e.f.g */
    | unit ( ("."|"::") ident )* ;
call = path ( "(" ( expression ),* ")" )? ;
unary =
    | ("!"|"~"|"-") unary
    | call ;
muldiv = unary ( ("*"|"/"|"%") unary )* ;
addsub = muldiv ( ("+"|"-") muldiv )* ;
shift  = addsub ( ("<<"|">>") addsub )* ;
band   = shift ( "&" shift )* ;
bxor   = band ( "^" band )* ;
bor    = bxor ( "|" bxor )* ;
range  = bor ( ".." bor )? ;
spread = ("..")? range ;
comparison = spread ( cmp_op spread ( cmp_op spread )? )? ;
land   =  comparison ( "&&" comparison )* ;
lor    =  land ( "||" land )* ;
assignment = ( ident ("="|asg_op) )? assignment 
    | lor;
expression = assignment

cmp_op = "<"|"<="|"=="|"!="|">="|">"
asg_op = "*="|"/="|"%="|"+="|"-="|"<<="|">>="|"&="|"|="|"^="

list  = "[" ( expression ),* "]" ;
set   = "set" "{" ( expression ),* "}" ;
dict  = "dict" "{" ( entry ),* "}" ;
entry = expression ":" expression ;

if    = "if" expression block ( "else" (if|block) )? ;
while = "while" expression block ;
for   = "for" ident "in" expression block ;
```
