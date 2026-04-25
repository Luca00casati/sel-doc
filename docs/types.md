---
layout: default
title: Types
nav_order: 4
---

# Types
{: .no_toc }

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Overview

sel is dynamically typed. Every value is a `Val` — a tagged union that can
hold one of the following kinds:

| Kind | Description |
|------|-------------|
| **Integer** | 64-bit `long`, stored inline (no allocation) |
| **Float** | 64-bit `double`, stored inline (no allocation) |
| **Bigint** | Arbitrary-precision integer, stored as a decimal string |
| **Bigfloat** | Arbitrary-precision float, stored as a decimal string |
| **String** | Byte array (ASCII or UTF-8) |
| **Cons** | A pair `(car . cdr)`, each element is any value |
| **Closure** | A function + captured environment |
| **Hash map** | Mutable key→value store (string or integer keys) |
| **FFI function** | Callable wrapper around a C library symbol |
| **Struct** | Heap buffer holding a C struct's bytes (for FFI by-value) |
| **Nil** | The zero/empty value |

---

## Integers

Small integers (fitting in a signed 64-bit `long`) are stored as **tagged
integers** — no heap allocation, no string representation.

```lisp
42
-100
0
```

When an arithmetic result **overflows** a `long`, sel automatically falls back
to **bigint** arithmetic using decimal strings. You never need to do anything
special — it just works.

```lisp
(* 999999999999999999 999999999999999999)
; => 999999999999999998000000000000000001  (bigint, no overflow)
```

---

## Floats

Floating-point numbers with up to 17 significant digits are stored as **tagged
doubles** — also inline, no allocation.

```lisp
3.14
-2.5e10
0.0
```

If a float literal has **more than 17 significant digits**, or if an arithmetic
result would require more precision, sel upgrades to **bigfloat** — a
decimal-string representation with unlimited significant digits.

```lisp
3.14159265358979323846264338327950288   ; bigfloat literal
```

---

## Mixed Arithmetic

Arithmetic is **automatically widening**:

- `integer op integer` → integer (or bigint on overflow)
- `integer op float` → float (or bigfloat)
- `float op float` → float (or bigfloat)
- `bigint op anything` → bigfloat (via promotion)

---

## Strings

sel has two string literal syntaxes:

| Syntax | Encoding | Backslash |
|--------|----------|-----------|
| `'…'` | ASCII | escape sequences interpreted (`\n`, `\t`, `\r`, `\\`, `\'`, `\"`, `\0`, `\xHH`) |
| `"…"` | UTF-8 | **raw** — backslash has no special meaning |

```lisp
'hello\nworld'     ; hello, newline, world  (escape interpreted)
"hello\nworld"     ; hello\nworld literally  (raw)
"café ☕"          ; any UTF-8 content
""                 ; empty string
```

Because `"…"` is raw, there is no way to embed a literal `"` inside one.
Use `'...'` if you need to embed a double-quote via `\"`.

Strings support several built-in operations: byte length, byte indexing, slicing,
concatenation, and conversion to/from numbers. See [Built-in Functions](builtins)
for the full list.

---

## Cons Cells

A **cons cell** is a pair of two values. Lists are built by chaining cons cells,
with `nil` as the terminator (the same as in classic Lisp).

```lisp
(cons 1 nil)            ; (1)
(cons 1 (cons 2 nil))   ; (1 2)
(cons 'a' (cons 'b' (cons 'c' nil)))
```

Access the parts with `car` (head) and `cdr` (tail):

```lisp
(car (cons 1 2))   ; 1
(cdr (cons 1 2))   ; 2
```

See [Built-in Functions](builtins) for details.

---

## Closures

A closure is a function value together with a snapshot of the lexical
environment at the point the `fn` expression was evaluated.

```lisp
(let makeadder
  (fn (n)
    (fn (x) (+ x n))))    ; inner fn captures n

(let add5 (makeadder 5))
(add5 3)   ; => 8
```

Closures are truthy values. You can store them in variables, pass them as
arguments, and return them from functions.

---

## Nil

`nil` is the canonical falsy zero. It is numerically equal to `0`.

```lisp
nil         ; 0
(= nil 0)   ; 1 (true)
(not nil)   ; 1 (true)
```

`nil` is the standard list terminator:

```lisp
(cons 1 (cons 2 nil))   ; linked list [1, 2]
```

---

## Hash Maps

A hash map is a mutable key→value store. Keys must be strings or integers;
values can be any `Val`. Hash maps are **always truthy**.

```lisp
(let h (hashmake))
(hashset h "x" 10)
(hashset h "y" 20)
(hashget h "x")          ; => 10
(hashhas? h "z")          ; => 0
(hashkeys h)             ; => ("x" . ("y" . 0))  (order unspecified)
(hashdel h "x")
(hashhas? h "x")          ; => 0
```

Hash maps support integer keys too:

```lisp
(let counts (hashmake))
(hashset counts 42 1)
(hashget counts 42)   ; => 1
```

See [Built-in Functions](builtins#hash-maps) for the full API.

---

## Structs

A struct is a heap buffer that holds the bytes of a C struct in the
host's native ABI layout.  It exists so that sel can pass and return C
structs **by value** through libffi.

```lisp
(defstruct Color uchar uchar uchar uchar)
(let c (makestruct "Color" 255 128 64 255))
(structget c 0)                ; => 255
(structset c 3 200)            ; mutates in place; returns c
```

Layouts are registered at compile time with `defstruct` and looked up
by name.  See [Built-in Functions → Structs](builtins#structs) for the
full API.  Struct values are **truthy**.

---

## FFI Functions

An FFI function is a callable wrapper around a symbol resolved from the
dynamic linker. It is created with the `ffi` built-in and called like any
other function.

```lisp
(let strlen (ffi "strlen" "long" "string"))
(strlen "hello")   ; => 5
```

FFI functions are **truthy** values and are first-class: they can be stored in
variables, passed as arguments, and stored in hash maps.

```lisp
(let h (hashmake))
(hashset h "puts" (ffi "puts" "int" "string"))
((hashget h "puts") "hi")
```

See [Built-in Functions](builtins#ffi) for supported type strings and usage.

---

## Truthiness

Falsy values:

- `0` / `nil`
- The empty string `''` / `""`

Everything else — any nonzero integer, any nonzero float, any non-empty string,
any cons cell, any closure, any hash map, any FFI function — is **truthy**.

```lisp
(if 0 'yes' 'no')     ; no
(if 1 'yes' 'no')     ; yes
(if "" 'yes' 'no')    ; no
(if "x" 'yes' 'no')   ; yes
(if (cons 0 0) 'yes' 'no')  ; yes  (cons cells are always truthy)
```
