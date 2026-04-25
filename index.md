---
layout: home
title: Home
nav_order: 1
---

# sel

**sel** is a small, Lisp-like language compiled to register-based bytecode
and executed by an iterative stack VM, with optional JIT compilation via
[MIR](https://github.com/vnmakarov/mir).  The tokenizer, AST, compiler,
VM, and runtime are written in C; MIR (JIT), libffi (FFI), and
[linenoise](https://github.com/antirez/linenoise) (REPL editing) are
vendored as git submodules and built statically — no system runtime
dependencies beyond libc.

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
| **Strings** | `'…'` ASCII with escape sequences; `"…"` raw UTF-8 literal |
| **Lists** | Linked cons cells — `cons`, `car`, `cdr` |
| **Functions** | First-class, lexically-scoped closures |
| **Macros** | Compile-time `defmacro` with textual substitution |
| **Tail calls** | Self-tail-call → JMP; general TCO via `OP_TAIL_CALL` |
| **JIT** | Lazy per-function native compile via MIR; disable with `--no-jit` |
| **FFI** | `(ffi "name" ret-type arg-types…)` via libffi; struct by-value via `defstruct`; dynamic loading via `loadlibrary` |
| **Standard library** | `core.sel` — load explicitly with `(load "core.sel")` |
| **REPL** | Interactive read-eval-print loop (linenoise), multi-line aware |

---

## Quick navigation

- [Getting Started](docs/getting-started) — build, REPL, run a file, flags
- [Syntax](docs/syntax) — tokens, S-expressions, comments
- [Types](docs/types) — integers, floats, bignum, strings, lists, closures, nil
- [Special Forms](docs/special-forms) — `if`, `fn`, `let`, `begin`, `defmacro`
- [Operators](docs/operators) — arithmetic, comparison, bitwise, shift
- [Built-in Functions](docs/builtins) — `cons`, `car`, `cdr`, `print`, `println`
- [Standard Library](docs/stdlib) — `core.sel`: booleans, math, list functions
- [Closures](docs/closures) — higher-order functions, captures, currying
- [Tail Calls](docs/tail-calls) — TCO, tail-recursive patterns
- [Internals](docs/internals) — macro system, compiler passes, VM, JIT, FFI trampoline
