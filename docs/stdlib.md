---
layout: default
title: Standard Library
nav_order: 8
---

# Standard Library
{: .no_toc }

The standard library is defined in `core.sel` and ships as a plain source
file alongside the `sel` binary.  Load it explicitly when you need its
functions:

```lisp
(load "core.sel")
```

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Boolean

| Macro | Expansion |
|-------|-----------|
| `(not x)` | `(if x 0 1)` |
| `(and a b)` | `(if a b 0)` |
| `(or a b)` | `(if a a b)` |

---

## Control

| Macro | Expansion |
|-------|-----------|
| `(when cond body)` | `(if cond body 0)` |
| `(unless cond body)` | `(if cond 0 body)` |

---

## Pair accessors

All `cXXr` combinations up to depth 4:

`caar` `cadr` `cdar` `cddr`  
`caaar` `caadr` `cadar` `caddr` `cdaar` `cdadr` `cddar` `cdddr`  
`cadddr`

Readable aliases:

| Name | Equivalent |
|------|-----------|
| `(first lst)` | `(car lst)` |
| `(second lst)` | `(cadr lst)` |
| `(third lst)` | `(caddr lst)` |
| `(fourth lst)` | `(cadddr lst)` |
| `(rest lst)` | `(cdr lst)` |

Fixed-arity list constructors:

```lisp
(list1 a)         ; (cons a 0)
(list2 a b)       ; (cons a (cons b 0))
(list3 a b c)     ; (cons a (cons b (cons c 0)))
(list4 a b c d)
```

Variadic constructor — takes any number of arguments:

```lisp
(list 1 2 3 4 5)   ; => (1 2 3 4 5)
(list)             ; => 0  (nil)
```

---

## Math

```
(abs x)           ; absolute value
(min a b)         ; smaller of a, b
(max a b)         ; larger of a, b
(clamp x lo hi)   ; x clamped to [lo, hi]
(square x)        ; x * x
(even? n)         ; 1 if n is even
(odd? n)          ; 1 if n is odd
(gcd a b)         ; greatest common divisor
(lcm a b)         ; least common multiple
(pow base exp)    ; base^exp  (non-negative integer exp)
```

---

## Lists

`0` is the empty list (nil).

```
(null? lst)           ; 1 if lst is 0
(length lst)          ; number of elements
(nth lst n)           ; element at index n  (0-based)
(last lst)            ; last element
(append a b)          ; concatenate two lists
(reverse lst)         ; reverse a list
(contains? lst x)     ; 1 if x is in lst
(take n lst)          ; first n elements
(drop n lst)          ; lst without first n elements
```

### Higher-order

```
(foldl f acc lst)     ; left fold:  f(f(f(acc, x0), x1), x2)…
(foldr f init lst)    ; right fold: f(x0, f(x1, f(x2, init)))
(map f lst)           ; apply f to each element; return new list
(filter pred lst)     ; keep elements where pred returns truthy
(foreach f lst)      ; call f on each element for side effects
(zipwith f a b)      ; (f a0 b0), (f a1 b1), … while both lists last
```

```lisp
(map (fn (x) (* x x)) (list3 1 2 3))
; => (. 1 (. 4 (. 9 0)))

(filter odd? (list4 1 2 3 4))
; => (. 1 (. 3 0))

(foldl (fn (acc x) (+ acc x)) 0 (list3 1 2 3))
; => 6
```

---

## Strings

### Predicates and byte access

```
(strempty? s)          ; 1 if s has byte length 0
```

### UTF-8 helpers

```
(strcpbytes b)        ; byte width of a UTF-8 sequence given its leading byte (1–4)
(strdecodeat s i)     ; Unicode code point starting at byte index i
(strcharlen s)        ; number of code points (not bytes) in s
(strcharref s n)      ; code point at character index n  (O(n))
(strcodepoints s)     ; list of all code points in s
```

```lisp
(strcharlen "café")        ; => 4
(strcharref "café" 3)      ; => 233  (é)
(strcodepoints "hi")       ; => (104 . (105 . 0))
```

### Joining and splitting

```
(strjoin sep lst)      ; concatenate list of strings with separator
(strsplitnl s)         ; split s on newlines; returns list of non-empty lines
```

```lisp
(strjoin ", " (list "a" "b" "c"))   ; => "a, b, c"
(strsplitnl "foo\nbar\nbaz")        ; => ("foo" "bar" "baz")
```

### Building strings

```
(makestr <part> ...)   ; variadic concat; numbers are auto-stringified
```

`makestr` is a thin wrapper over `strcc` + `numtostr`.  Strings flow
through `numtostr` unchanged, so mixed string/number arguments compose
without explicit conversions.

```lisp
(makestr "fps: " 60 " (" 16.7 "ms)")   ; => "fps: 60 (16.7ms)"
(makestr "x=" x ", y=" y)
(makestr)                               ; => ""
```

---

## I/O

I/O functions are implemented in `core.sel` using the FFI layer.

```
(open path mode)   ; fopen(path, mode) → file pointer (integer)
(close port)       ; fclose(port) → 0
(write port val)   ; fputs(val, port) + newline → val
```

```lisp
(let fp (open "/tmp/out.txt" "w"))
(write fp "hello")
(close fp)
```

```
(readline)        ; read all of stdin; return list of lines (strings)
(read fp)          ; read entire file fp; return list of lines
```

```lisp
(let lines (readline))           ; block until EOF on stdin
(foreach println lines)

(let fp (open "/etc/hostname" "r"))
(let lines (read fp))
(close fp)
(println (car lines))
```
