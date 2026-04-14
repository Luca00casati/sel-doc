---
layout: default
title: Tail Calls
nav_order: 9
---

# Tail Calls
{: .no_toc }

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is a Tail Call?

A call is in **tail position** if its return value is the return value of the
enclosing function — i.e., there is nothing left to do after the call returns.

```lisp
; tail call — last thing in the function body
(fn fib (n acc)
  (if (= n 0)
    acc
    (fib (- n 1) (+ acc n))))   ; <-- tail call
```

```lisp
; NOT a tail call — the result is used in a multiplication
(fn fac (n)
  (* n (fac (- n 1))))   ; <-- NOT a tail call
```

---

## Tail Call Optimisation (TCO)

sel implements full tail call optimisation. A tail call reuses the **current
stack frame** instead of pushing a new one. This means:

- Deep recursive loops that use tail calls use **O(1)** stack space.
- You can write "loop-as-recursion" patterns with no stack overflow risk.

The VM supports up to 200,000 active frames for non-tail calls; tail-recursive
loops have no effective depth limit.

---

## Writing Tail-Recursive Loops

The standard pattern is an **accumulator parameter**:

```lisp
; sum of 1..n — tail recursive
(let sum
  (fn sum (n acc)
    (if (= n 0)
      acc
      (sum (- n 1) (+ acc n)))))

(sum 1000000 0)   ; no stack overflow
```

```lisp
; factorial — tail recursive
(let fac
  (fn fac (n acc)
    (if (= n 0)
      acc
      (fac (- n 1) (* acc n)))))

(fac 20 1)   ; => 2432902008176640000
```

```lisp
; Fibonacci — tail recursive
(let fib
  (fn fib (n a b)
    (if (= n 0)
      a
      (fib (- n 1) b (+ a b)))))

(fib 50 0 1)   ; => 12586269025
```

---

## Tail Calls via Closures

TCO also applies when calling a **closure** in tail position (e.g. a
higher-order loop):

```lisp
(let loop
  (fn loop (f n)
    (if (= n 0)
      0
      (loop f (- n 1)))))   ; tail call to loop (named fn)

; calling an argument closure in tail position
(let repeat
  (fn repeat (f n)
    (if (= n 0)
      nil
      (let _ (f n)
        (repeat f (- n 1))))))
```

---

## How TCO Works Internally

The compiler detects tail position during AST compilation. A call in tail
position emits `OP_TAIL_CALL` (for known named functions) or
`OP_TAIL_CALL_VAL` (for closure values) instead of `OP_CALL`.

The VM handles `OP_TAIL_CALL` by reusing the current `VMFrame`: it resets the
frame's chunk, instruction pointer, and environment in-place without growing the
frame stack.

---

## Function Inlining and Tail Calls

The compiler tries function inlining **before** checking for tail position.
A tail call to a small, pure function is therefore inlined directly — zero
call overhead, even better than `OP_TAIL_CALL`.

A function is eligible for inlining if:
- It has been fully compiled (its body is complete).
- Its body is at most 8 "real" instructions.
- It contains no nested calls, closures, or free-variable loads.
