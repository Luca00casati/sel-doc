---
layout: home
title: Home
nav_order: 1
---

# sel

**sel** is a small, Lisp-like language compiled to register-based bytecode and
executed by an iterative stack VM. The entire implementation — tokenizer, AST,
compiler, and VM — is written in C with no external runtime dependencies.

```lisp
; classic recursive Fibonacci (tail-recursive accumulator form)
(let fib
  (fn fib (n acc)
    (if (= n 0)
      acc
      (fib (- n 1) (+ acc n)))))

(println (fib 100 0))
```

---

## Highlights

| Feature | Detail |
|---------|--------|
| **Syntax** | Uniform S-expression syntax |
| **Numbers** | Native `long`, `double`, arbitrary-precision bigint and bigfloat |
| **Strings** | ASCII single-quoted `'…'` and UTF-8 double-quoted `"…"` |
| **Lists** | Linked cons cells — `cons`, `car`, `cdr` |
| **Functions** | First-class, lexically-scoped closures |
| **Tail calls** | Optimised — deep recursion uses O(1) stack frames |
| **REPL** | Interactive read-eval-print loop, multi-line aware |

---

## Quick navigation

- [Getting Started](docs/getting-started) — build, REPL, run a file
- [Syntax](docs/syntax) — tokens, S-expressions, comments
- [Types](docs/types) — integers, floats, bignum, strings, lists, closures, nil
- [Special Forms](docs/special-forms) — `let`, `if`, `fn`, `and`, `or`, `not`
- [Operators](docs/operators) — arithmetic, comparison, bitwise, shift
- [Built-in Functions](docs/builtins) — `cons`, `car`, `cdr`, `print`, `println`
- [Closures](docs/closures) — higher-order functions, captures, currying
- [Tail Calls](docs/tail-calls) — TCO, tail-recursive patterns
- [Internals](docs/internals) — compiler passes, bytecode, VM architecture
