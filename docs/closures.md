---
layout: default
title: Closures
nav_order: 8
---

# Closures
{: .no_toc }

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is a Closure?

A closure is a function value that **captures** a snapshot of the lexical
environment at the point it was created. When the closure is later called, free
variables (variables not in the parameter list) are looked up in the captured
environment.

```lisp
(let make-counter
  (fn (start)
    (fn () start)))

(let c (make-counter 10))
(c)   ; => 10
```

---

## Creating Closures

Every `fn` expression creates a closure — even top-level functions. The closure
is the first-class value that gets bound to a name via `let`.

```lisp
(let add
  (fn (a b) (+ a b)))    ; add is a closure

(add 3 4)   ; => 7
```

---

## Capturing Variables

Free variables inside a `fn` body are captured at the moment `fn` is evaluated.

```lisp
(let offset 100)

(let shift
  (fn (x) (+ x offset)))   ; captures offset = 100

(shift 5)   ; => 105
```

If `offset` were rebound later, the closure still holds the original value from
when it was captured:

```lisp
(let offset 100)
(let shift (fn (x) (+ x offset)))
(let offset 999)   ; rebind — does NOT affect shift's capture

(shift 5)   ; => 105  (uses the captured 100)
```

---

## Higher-Order Functions

Closures can be passed as arguments and returned as values.

### Function as Argument

```lisp
(let apply
  (fn (f x) (f x)))

(apply (fn (n) (* n n)) 7)   ; => 49
```

### Function as Return Value (Currying)

```lisp
(let add-n
  (fn (n)
    (fn (x) (+ x n))))

(let add10 (add-n 10))
(let add20 (add-n 20))

(add10 5)    ; => 15
(add20 5)    ; => 25
```

### Map over a List

```lisp
(let map
  (fn map (f xs)
    (if (= xs nil)
      nil
      (cons (f (car xs)) (map f (cdr xs))))))

(map (fn (x) (* x x))
     (cons 1 (cons 2 (cons 3 nil))))
; => (1 . (4 . (9 . 0)))  i.e. [1, 4, 9]
```

### Fold / Reduce

```lisp
(let fold
  (fn fold (f acc xs)
    (if (= xs nil)
      acc
      (fold f (f acc (car xs)) (cdr xs)))))

(fold (fn (a b) (+ a b)) 0
      (cons 1 (cons 2 (cons 3 nil))))
; => 6
```

---

## Recursive Closures

Use the named `fn` form so the function can call itself:

```lisp
(let fac
  (fn fac (n)
    (if (= n 0)
      1
      (* n (fac (- n 1))))))

(fac 5)   ; => 120
```

The name `fac` is bound inside the body but **not** automatically in the outer
scope — that is what the surrounding `let` is for.

---

## Mutual Recursion

Because `let` bindings persist across top-level expressions, mutually recursive
functions can be defined in sequence:

```lisp
(let even?
  (fn even? (n)
    (if (= n 0) 1 (odd? (- n 1)))))

(let odd?
  (fn odd? (n)
    (if (= n 0) 0 (even? (- n 1)))))

(even? 10)   ; => 1
(odd? 7)     ; => 1
```

`even?` references `odd?` — which doesn't exist yet when `even?` is compiled.
Because `odd?` is resolved at **runtime** (via environment lookup), this works
as long as `odd?` is defined before `even?` is actually **called**.

---

## Implementation Notes

At runtime, creating a closure (via `OP_MAKE_CLOSURE`) snapshots a subset of
the current environment into the closure struct. When a closure is called (via
`OP_CALL_VAL`), the callee environment is built by first loading all captured
bindings, then layering the argument bindings on top — so parameters shadow
captured variables with the same name.

**Captured-variable trimming.** The compiler scans the compiled function body
for `LOAD_VAR` instructions whose name is not a parameter. Only those names —
the function's true free variables — are captured when `OP_MAKE_CLOSURE` runs.
A closure defined inside a scope with many bindings captures only what it
actually reads, not the whole environment.

**Escape analysis.** If a function body contains no inner `fn` expressions, its
`let` bindings cannot be captured by any closure and `STORE_VAR` instructions
are skipped entirely, leaving values only in registers.
