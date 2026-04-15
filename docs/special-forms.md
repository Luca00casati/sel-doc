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

## `if` — conditional

```
(if <condition> <then> <else>)
```

Evaluates `<condition>`. If truthy, evaluates and returns `<then>`; otherwise
evaluates and returns `<else>`. Only the chosen branch is evaluated.

```lisp
(if (= x 0) "zero" "nonzero")
(if (< n 0) (- 0 n) n)    ; absolute value
```

**Compile-time optimisation:** if the condition is a constant, the dead branch
is never emitted to bytecode.

```lisp
(if 1 "always" "never")   ; compiles to just LOAD_CONST "always"
```

---

## `fn` — anonymous and named functions

### Anonymous

```
(fn (<param> …) <body>)
```

Creates a closure with the given parameters and body.

```lisp
(let square (fn (x) (* x x)))
(square 9)   ; => 81
```

### Named (for recursion)

```
(fn <name> (<param> …) <body>)
```

Gives the closure a name visible **inside the body**, enabling direct recursion.

```lisp
(let fac
  (fn fac (n)
    (if (= n 0)
      1
      (* n (fac (- n 1))))))

(fac 10)   ; => 3628800
```

---

## `let` — bind a name

```
(let <name> <expr>)
```

Evaluates `<expr>` and binds the result to `<name>`. Returns the value.

```lisp
(let x 42)
(let y (+ x 1))
```

`let` is the only way to introduce a named binding. There is no `define` or `set!`.

---

## `begin` — sequence

```
(begin <expr> …)
```

Evaluates expressions left to right and returns the value of the last one.
Useful for sequencing side effects.

```lisp
(begin
  (println "step 1")
  (println "step 2")
  42)          ; => 42
```

---

## `defmacro` — compile-time macro

```
(defmacro <name> (<param> …) <body>)
```

Defines a compile-time macro. At every call site, the arguments are substituted
into `<body>` textually before compilation — no evaluation happens yet.

```lisp
(defmacro swap (a b)
  (let tmp a
    (let a b
      (let b tmp))))
```

Macros expand to `if`-expressions, `let` chains, and other forms. There is no
hygiene — parameter names may shadow outer bindings.

`not`, `and`, `or`, `when`, and `unless` are all defined as macros in
`core.sel`:

```lisp
(defmacro not    (x)      (if x 0 1))
(defmacro and    (a b)    (if a b 0))
(defmacro or     (a b)    (if a a b))
(defmacro when   (c body) (if c body 0))
(defmacro unless (c body) (if c 0 body))
```

n-ary `and` / `or` compose by nesting: `(and a (and b c))`.
