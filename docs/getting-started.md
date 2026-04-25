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

Build-time requirements:

- `gcc` or `clang`
- standard C library + `libm`
- `make`
- For first-time `libffi` bootstrap: `autoconf`, `automake`, `libtool`,
  `libtool-bin`, `libltdl-dev`, `pkg-config`, `m4`, `texinfo`
  (Debian/Ubuntu names; equivalent on other distros).  These are not
  needed once `libffi/` has been built once.

Three submodules are vendored under the repo root: `linenoise/` (REPL
editing), `mir/` (JIT backend), and `libffi/` (FFI marshalling).  All
three are built statically and linked into the `sel` binary — no system
runtime dependencies beyond libc.

```sh
git clone --recurse-submodules https://github.com/Luca00casati/sel.git
cd sel
make
```

If you already cloned without `--recurse-submodules`:

```sh
git submodule update --init
```

`make` produces a single `sel` binary.  The standard library
(`core.sel`) is a plain source file — sel loads it on demand via
`(load "core.sel")`, so the working directory needs access to it.

### Optional extras

- `make raylib-binding` regenerates `gen_raylib.sel` from
  `/usr/include/raylib.h` using `genffi.py`.  Requires `python-clang`
  (libclang Python bindings) and raylib's dev headers.

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

Load the standard library explicitly when you need it:

```
sel> (load "core.sel")
```

After that, core macros and functions (`map`, `filter`, `list3`, etc.) are
available.

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
| `--no-jit` | Disable the JIT compiler; run purely in the interpreter |
| `--dump-ast <file>` | Print the AST of a file and exit (no execution) |
| `--dump-inst <file>` | Compile a file and print its bytecode disassembly; useful for inspecting compiler output |

```sh
./sel --no-jit program.sel         # interpreter only, no JIT
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
