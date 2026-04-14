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

sel is dynamically typed. Every value is a `String` — a tagged union that can
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

A string is a byte array. Single-quoted strings are ASCII; double-quoted strings
are UTF-8.

```lisp
'hello'
"こんにちは"
""          ; empty string
```

Strings are not arrays — sel has no indexing or slicing operators yet. They are
mainly useful as data to `print` and `println`, and as keys if passed to cons
cells.

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
(let make-adder
  (fn (n)
    (fn (x) (+ x n))))    ; inner fn captures n

(let add5 (make-adder 5))
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

## Truthiness

Falsy values:

- `0` / `nil`
- The empty string `''` / `""`

Everything else — any nonzero integer, any nonzero float, any non-empty string,
any cons cell, any closure — is **truthy**.

```lisp
(if 0 'yes' 'no')     ; no
(if 1 'yes' 'no')     ; yes
(if "" 'yes' 'no')    ; no
(if "x" 'yes' 'no')   ; yes
(if (cons 0 0) 'yes' 'no')  ; yes  (cons cells are always truthy)
```
