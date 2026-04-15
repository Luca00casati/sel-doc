---
layout: default
title: Syntax
nav_order: 3
---

# Syntax
{: .no_toc }

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## S-Expressions

sel uses a uniform **S-expression** syntax. Every piece of code is either:

- an **atom** — a number, string literal, or symbol
- a **list** — zero or more expressions enclosed in `(` … `)`

The first element of a list is the operator (a special form or a function name);
the remaining elements are the arguments.

```lisp
(+ 1 2)          ; call operator +, two arguments
(println "hi")   ; call println with one string argument
(if (= x 0) "zero" "nonzero")
```

---

## Tokens

### Symbols

Any sequence of non-whitespace, non-parenthesis, non-quote characters that does
**not** parse as a number is a symbol. Symbols are case-sensitive.

```
x  foo  my-var  +  =  <<  some-thing-123
```

### Numbers

An optional leading `-`, at least one digit, optionally one `.` and more digits,
optionally an exponent `e`/`E` followed by an optional sign and digits.

```lisp
42        ; integer
-7        ; negative integer
3.14      ; float
1.5e10    ; scientific notation
-2.0e-3   ; negative float with negative exponent
```

**Integers** that fit in a 64-bit `long` are stored as tagged longs (no
allocation). Integers that overflow are automatically promoted to **bigint**
(arbitrary precision decimal strings).

**Floats** that fit in a `double` are stored as tagged doubles. Floats with
more than 17 significant digits are stored as **bigfloat** decimal strings and
handled with arbitrary-precision arithmetic.

### String Literals

sel has two string delimiters:

| Delimiter | Encoding | Backslash |
|-----------|----------|-----------|
| `'…'` | ASCII only | escape sequences interpreted; bytes > 127 rejected |
| `"…"` | UTF-8 | **raw** — backslash has no special meaning |

`'...'` escape sequences:

| Escape | Character |
|--------|-----------|
| `\n` | newline |
| `\t` | tab |
| `\r` | carriage return |
| `\\` | backslash |
| `\'` | single quote |
| `\"` | double quote |
| `\0` | null byte |
| `\xNN` | byte with hex value `NN` |

```lisp
'hello'           ; ASCII string
'line1\nline2'    ; newline in single-quoted string
'\x41\x42\x43'   ; "ABC"
"café ☕"         ; raw UTF-8 — any Unicode, no escaping
"C:\Users\name"  ; backslashes are literal
```

---

## Comments

A semicolon `;` begins a **line comment** that runs to the end of the line.

```lisp
; this is a comment
(+ 1 2)   ; inline comment
```

There are no block comments.

---

## Whitespace

Spaces, tabs, and newlines are all interchangeable whitespace. They separate
tokens and are otherwise insignificant. Expressions can be broken across lines
freely.

```lisp
(let result
  (if (> x 0)
    (* x 2)
    0))
```

---

## nil

The symbol `nil` represents the empty list and falsy zero value. It is
equivalent to `0` and compares equal to it.

```lisp
nil        ; same as 0
(= nil 0)  ; 1 (true)
```
