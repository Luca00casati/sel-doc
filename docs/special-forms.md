---
layout: default
title: Special Forms
nav_order: 5
---

# Special Forms
{: .no_toc }

Special forms look like function calls but are handled directly by the compiler.
They do not evaluate all their arguments eagerly.

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## `let` — bind a name

```
(let <name> <expr>)
```

Evaluates `<expr>` and binds the result to `<name>` in the current environment.
Returns the value.

```lisp
(let x 42)          ; x = 42
(let y (+ x 1))     ; y = 43
(let msg "hello")
```

`let` is the only way to introduce a named binding. There is no `define` or
`set!`.

**Note:** Multiple `let` forms in the same top-level or function body are
independent. Later `let` forms can reference earlier ones because bindings
persist across expressions in the same session / file.

---

## `if` — conditional

```
(if <condition> <then> <else>)
```

Evaluates `<condition>`. If truthy, evaluates and returns `<then>`; otherwise
evaluates and returns `<else>`. Only the chosen branch is evaluated — the other
is not.

```lisp
(if (= x 0) "zero" "nonzero")
(if (< n 0) (- 0 n) n)    ; absolute value
```

**Compile-time optimisation:** If the condition is a constant, the compiler
discards the dead branch entirely — it is never emitted to bytecode.

```lisp
(if 1 "always" "never")   ; compiles to just "always"
```

---

## `fn` — anonymous and named functions

### Anonymous

```
(fn (<param> …) <body>)
```

Creates a closure with the given parameters and body. The body is a single
expression (wrap multiple expressions in a `let` chain or nest them).

```lisp
(let square (fn (x) (* x x)))
(square 9)   ; => 81
```

### Named (for recursion)

```
(fn <name> (<param> …) <body>)
```

Gives the closure a name that is visible **inside the body**, enabling direct
recursion without needing `let` first.

```lisp
(let fac
  (fn fac (n)
    (if (= n 0)
      1
      (* n (fac (- n 1))))))

(fac 10)   ; => 3628800
```

The name is bound only within the body — it is not automatically placed in the
outer environment. Use `let` around the `fn` to also bind it in the outer scope.

### Multiple Parameters

Parameters are listed in the `(…)` after the optional name:

```lisp
(fn (a b c) (+ a (+ b c)))
(fn add3 (a b c) (+ a (+ b c)))
```

### Zero Parameters

```lisp
(let greet (fn () (println "Hello!")))
(greet)
```

---

## `and` — short-circuit conjunction

```
(and <expr> …)
```

Evaluates expressions left to right. Returns the **first falsy value**, or the
**last value** if all are truthy. Returns `1` for the empty `(and)`.

```lisp
(and 1 2 3)       ; => 3
(and 1 0 3)       ; => 0  (short-circuits, 3 never evaluated)
(and)             ; => 1
```

---

## `or` — short-circuit disjunction

```
(or <expr> …)
```

Evaluates expressions left to right. Returns the **first truthy value**, or the
**last value** if all are falsy. Returns `0` for the empty `(or)`.

```lisp
(or 0 0 7)        ; => 7
(or 5 (/ 1 0))    ; => 5  (short-circuits, division never evaluated)
(or)              ; => 0
```

---

## `not` — logical negation

```
(not <expr>)
```

Returns `1` if `<expr>` is falsy, `0` otherwise.

```lisp
(not 0)          ; => 1
(not 1)          ; => 0
(not nil)        ; => 1
(not "hello")    ; => 0
(not (cons 1 2)) ; => 0
```
