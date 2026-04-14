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

sel has no dependencies beyond a C99-compatible compiler. Clone the repository
and run `make` (or compile the sources directly):

```sh
git clone https://github.com/Luca00casati/sel.git
cd sel
make
```

This produces a single `sel` binary.

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
<closure>
sel> (add 3 4)
7
```

State persists across expressions in the same session — bindings created with
`let` and functions defined with `fn` are available for the rest of the session.

Exit the REPL with `Ctrl+D` (EOF).

---

## Running a File

Pass a `.sel` file as the only argument:

```sh
./sel program.sel
```

All top-level expressions in the file are evaluated in order. The value of the
**last** expression is printed as `Result: <value>`.

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
