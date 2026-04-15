---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started
{: .no_toc }

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Building

sel has no dependencies beyond a C99-compatible compiler and GNU binutils.
Clone the repository and run `make`:

```sh
git clone https://github.com/Luca00casati/sel.git
cd sel
make
```

This produces two files that must be kept together:

| File | Purpose |
|------|---------|
| `sel` | the interpreter binary |
| `core.selc` | precompiled standard library bytecode |

`core.selc` is compiled from `core.sel` by the binary itself as part of
`make`. If it is missing at runtime the interpreter falls back to compiling
`core.sel` from source automatically.

---

## Running the REPL

Launch the interactive REPL by invoking `sel` with no arguments:

```
$ ./sel
sel repl  (ctrl+d to exit)
sel> (+ 1 2)
3
sel> (let x 10)
10
sel> (* x x)
100
sel>
```

The REPL is **multi-line aware**: if you open a parenthesis and press Enter
without closing it, the prompt switches to `...>` and keeps accumulating input
until the expression is balanced.

```
sel> (let add
...>   (fn (a b) (+ a b)))
#<closure>
sel> (add 3 4)
7
```

The standard library is loaded automatically, so all core macros and functions
(`map`, `filter`, `list3`, etc.) are available immediately.

State persists across expressions — bindings and functions defined with `let`
and `fn` remain available for the rest of the session.

Exit the REPL with `Ctrl+D` (EOF).

---

## Running a File

Pass one or more `.sel` files as arguments:

```sh
./sel program.sel
./sel lib.sel program.sel   # lib.sel is loaded first
```

All top-level expressions are evaluated in order across all files. The value of
the **last** expression is printed as `Result: <value>`.

```lisp
; hello.sel
(println "Hello, world!")
(+ 1 1)
```

```
$ ./sel hello.sel
Hello, world!
Result: 2
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--no-core` | Skip loading the standard library |
| `--compile-core core.selc` | Compile `core.sel` → `core.selc` and exit |

```sh
./sel --no-core program.sel       # bare interpreter, no stdlib
./sel --compile-core core.selc    # regenerate bytecode after editing core.sel
```

---

## First Program

```lisp
; sum of 1..100
(let sum
  (fn sum (n acc)
    (if (= n 0)
      acc
      (sum (- n 1) (+ acc n)))))

(println (sum 100 0))
```

Save as `sum.sel` and run:

```
$ ./sel sum.sel
5050
Result: 5050
```
