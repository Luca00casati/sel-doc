---
layout: default
title: Operators
nav_order: 6
---

# Operators
{: .no_toc }

All operators are written in **prefix position** inside a list:

```lisp
(+ a b)
(<< x 3)
(not flag)
```

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Arithmetic

| Form | Returns | Notes |
|------|---------|-------|
| `(+ a b)` | `a + b` | Integer or float; auto-promotes to bigint/bigfloat on overflow |
| `(- a b)` | `a - b` | Same promotion rules |
| `(* a b)` | `a * b` | Same promotion rules |
| `(/ a b)` | `a / b` | Integer division for integers; float division for floats |
| `(% a b)` | `a mod b` | Integer only; error on bigint operands |

### Integer vs Float

If **both** operands are integers, the result is an integer (exact). If
**either** operand is a float (or bigfloat), the result is a float.

```lisp
(+ 1 2)        ; => 3     (integer)
(+ 1 2.0)      ; => 3.0   (float)
(/ 7 2)        ; => 3     (integer division, truncates)
(/ 7 2.0)      ; => 3.5   (float division)
(% 10 3)       ; => 1
```

### Overflow → Bigint

Integers that overflow 64 bits are silently promoted to arbitrary-precision
bigint:

```lisp
(* 9999999999999999999 9999999999999999999)
; => 99999999999999999980000000000000000001
```

### Compile-time Constant Folding

All arithmetic on **literal** operands is evaluated at compile time — no
instructions are emitted for the computation:

```lisp
(* 6 7)       ; emits a single LOAD_CONST 42
(+ 1 (+ 2 3)) ; emits LOAD_CONST 6
```

---

## Comparison

| Form | Returns | Notes |
|------|---------|-------|
| `(= a b)` | `1` if equal, `0` otherwise | Works for integers, floats, strings |
| `(< a b)` | `1` if `a < b`, else `0` | |
| `(> a b)` | `1` if `a > b`, else `0` | |

Comparisons return `1` (true) or `0` (false) as integers.

```lisp
(= 3 3)      ; => 1
(= 3 4)      ; => 0
(< 1 2)      ; => 1
(> 10 5)     ; => 1
(< 2.5 3.0)  ; => 1  (float comparison)
```

**String equality** with `=` compares byte-by-byte:

```lisp
(= "abc" "abc")   ; => 1
(= "abc" "xyz")   ; => 0
```

---

## Bitwise

All bitwise operators require **integer** operands. They do not work on floats
or bigfloats.

| Form | Operation | Notes |
|------|-----------|-------|
| `(& a b)` | Bitwise AND | |
| `(\| a b)` | Bitwise OR | |
| `(^ a b)` | Bitwise XOR | |
| `(~ a)` | Bitwise NOT (unary) | One's complement |
| `(<< a n)` | Left shift `a` by `n` bits | `n` must be 0–63 |
| `(>> a n)` | Right shift `a` by `n` bits (arithmetic) | `n` must be 0–63 |

```lisp
(& 0b1100 0b1010)   ; = (& 12 10) => 8   (0b1000)
(| 12 10)           ; => 14  (0b1110)
(^ 12 10)           ; => 6   (0b0110)
(~ 0)               ; => -1
(<< 1 10)           ; => 1024
(>> 1024 3)         ; => 128
```

**Constant folding** applies to bitwise operators too:

```lisp
(<< 1 8)   ; compiles to LOAD_CONST 256
(& 255 15) ; compiles to LOAD_CONST 15
```

---

## Operator Precedence

sel has no operator precedence — all operators are explicit via parentheses.
There is no infix syntax.

```lisp
; "1 + 2 * 3" must be written explicitly:
(+ 1 (* 2 3))   ; => 7  (multiplication first)
(* (+ 1 2) 3)   ; => 9  (addition first)
```
