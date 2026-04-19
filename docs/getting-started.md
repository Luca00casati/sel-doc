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
`make`. If it is missing at runtime the interpreter exits with an error; run
`./sel --compile core.sel core.selc` to rebuild it.

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
| `--no-jit` | Disable the JIT compiler; run purely in the interpreter |
| `--compile <input.sel> [output.selc]` | Compile a `.sel` file to bytecode and exit; output defaults to `<input>.selc` |
| `--dump-ast <file>` | Print the AST of a file and exit (no execution) |
| `--dump-inst <file>` | Compile a file and print its bytecode disassembly; useful for inspecting compiler output |

```sh
./sel --no-core program.sel        # bare interpreter, no stdlib
./sel --no-jit program.sel         # interpreter only, no JIT
./sel --compile core.sel           # → core.selc
./sel --compile mylib.sel          # → mylib.selc
./sel --compile mylib.sel out.selc # explicit output name
./sel --dump-ast program.sel       # inspect parse tree
./sel --dump-inst program.sel      # inspect bytecode
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
