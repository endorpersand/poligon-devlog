---
layout: post
title: Research: C and the Wacky World of Strings
---

I need to document the messiness of strings in C, because much of LLVM's API involves strings (woohoo).

But first,

# Rust `char`, `str`, `String`

Main sources:
* https://doc.rust-lang.org/std/primitive.char.html
* https://doc.rust-lang.org/std/string/struct.String.html

Rust has a primitive `char` type (obviously to hold singular characters). They have a few notable qualities:
- They are 4 bytes (32-bit, size `u32`).
- They are represented in code as their Unicode code point (UTF-32).
- They are restricted to: `0x000000-0x10FFFF`, and cannot be `0x00D800-0x00DFFF`.
	- This aligns with the chars UTF-8 can represent.

Rust also has a heap-allocated `String` type (to hold strings of characters). They have a few notable qualities:
- They are literally defined as...
```rs
pub struct String {
	vec: Vec<u8>
}
```
*(They are simply a heap-allocated list of bytes.)*

- They are represented as a string of UTF-8 codepoints.

For a `char` to be appended to a `String`, it is encoded into UTF-8 and converted to a string via `char::encode_utf8`.

`encode_utf8`'s encoding process involves:
- Computing the number of bytes of the `char` in UTF-8
- Do bit magic to perform the computation

# C `char`

https://en.cppreference.com/w/c/language/arithmetic_types
https://en.cppreference.com/w/c/io/putchar
https://en.cppreference.com/w/c/io/puts
https://en.cppreference.com/w/c/io/fprintf

Unlike Rust, `char` is a byte long. 

Strings are represented as character arrays OR a char pointer (`char*`).
- The char pointer points to an address in memory. 

If a string is represented as `char*`, then the string ends before the first null-terminator (`\0`).

`char` has a few utilities to print to stdout:
- `putchar`: print a singular char to stdout
- `puts`: print a string to stdout
- `printf("%.7s", str)`: print 7 characters of a string to stdout (or less if `\0` appears before)
- `printf("%.*s", len, str)`: print `len` characters of a string to `stdout` (or less if `\0` appears before)

When printed, `char` is printed AS the given byte. As such, it is up to the OS to determine how the bytes should be represented.

# C `wchar_t`

https://en.cppreference.com/w/c/language/arithmetic_types
https://en.cppreference.com/w/c/io/putwchar
https://en.cppreference.com/w/c/io/fputws
https://en.cppreference.com/w/c/io/fprintf

`wchar_t` is: 
- a 4-byte type (similar to Rust `char`) representing UTF-32 (on POSIX)
- a 2-byte type representing UTF-16 units (on Windows)

It is also called a "wide character." Wide strings also exist, of the form `wchar_t*`, which act similarly to narrow strings.

`wchar_t` also has a few corresponding utilities to print to stdout:
- `putwchar`: put a singular wide character to stdout, using the locale to encode the char into bytes
- `fputws`: put a wide char string to a file, using the locale to encode the char into bytes
- `printf("%ls", wide_str)`: fputws
- `printf("%.7ls", wide_str)`
- etc.

## Locale

https://en.cppreference.com/w/c/locale/setlocale

The locale defines... how conversions occur and how wide chars are encoded. The relevant ones (for my purposes):
- `""`: OS's locale
- `"C"` (default): error on non-ASCII
- `"en_US.UTF-8"`: encode conversions in US, encode `wchar_t` in UTF-8

The locale **has to be set** before the function is called. This can be done with `setlocale(int category, const char* locale)`.

For my purposes, the category `LC_ALL` (everything) is 0.

# The Approach

I aim to take a similar approach to that in Rust. Namely,
- `char` is defined as a 4-byte type holding a character's UTF-32 value
- `string` is equivalent to a list of bytes representing UTF-8

Thankfully, there are some shortcuts I can take without having to reimplement everything.

- `char` can simply be represented as an LLVM `i32`, and I can cast Rust `char` to `u32` to LLVM `i32` via inkwell. This is pretty much identical to C `wchar_t`.

## `char` to `string`

https://en.cppreference.com/w/c/experimental/dynamic/asprintf
https://en.cppreference.com/w/c/string/multibyte/wctomb
https://en.cppreference.com/w/c/io/fprintf

1. `asprintf` is a function that formats a string via `printf` and allocates a string to the output. 
Its signature:
```c
int asprintf( char **strp, const char *fmt, ... );
```

`asprintf` asks for
* a pointer (which can hold a string pointer)
* printf arguments
and returns 
- the length written (ignoring null-terminator)
- a malloc'd str pointer in the provided pointer parameter

I could perform calls like
```c
asprintf(str_ptr, "%lc", wide_char)
```
giving me a `char*`, which I can use to extend a dynamic array.

This does malloc the `char*` though, so if I don't `free` it, it'll stay present. Doing a heap allocation every time also seems very expensive.

2. `wctomb` (wide character to multibyte) is a function that writes the multibyte representation of a `wchar_t` to a string buffer.
```c
int wctomb( char *s, wchar_t ws );
```

`wctomb` asks for
- an allocated narrow string buffer to write to
- the `wchar_t` to write
and returns
- the number of characters written
- an allocated 

For UTF-8, C asks the allocated buffer to be at least 6 bytes to ensure the entire `wchar_t` can be encoded. As of 2003, only 4 are needed.

This is likely more efficient because `wctomb` can be used with a stack string pointer which can be dropped.

3. `sprintf` is a function that formats a string via `printf` and writes the output to a given buffer.

This is similar to `asprintf` but does not allocate a new string. This could be used, but I think `wctomb` is identical purpose-wise.

## `string` to `char`

https://en.cppreference.com/w/c/string/multibyte

Some useful functions for this:

Seeking `wchar_t`:
`mblen(str, 4)`: Compute length of next `wchar_t` in narrow string
`mbtowc`: Reads a character in narrow string into wide string, limiting to `n` bytes

All at once:
`mbstowcs`: Write narrow string into a wide string, limiting to `n` wide characters