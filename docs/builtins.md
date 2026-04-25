---
layout: default
title: Built-in Functions
nav_order: 7
---

# Built-in Functions
{: .no_toc }

These are compiled directly to bytecode instructions — no function call
overhead. They all follow the same S-expression call syntax.

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## List / Cons

### `cons`

```
(cons <head> <tail>) → cons-cell
```

Allocates a new cons cell with `<head>` as the car and `<tail>` as the cdr.
Both arguments can be any value.

```lisp
(cons 1 2)                     ; pair (1 . 2)
(cons 1 nil)                   ; one-element list [1]
(cons 1 (cons 2 (cons 3 nil))) ; list [1, 2, 3]
(cons "a" (cons "b" nil))      ; list of strings
```

A proper linked list is terminated by `nil`:

```lisp
; build a list from scratch
(let xs (cons 10 (cons 20 (cons 30 nil))))
(car xs)           ; => 10
(car (cdr xs))     ; => 20
(car (cdr (cdr xs))) ; => 30
```

---

### `car`

```
(car <cons>) → value
```

Returns the **head** (first element) of a cons cell.

```lisp
(car (cons 'a' 'b'))   ; => "a"
(car (cons 1 nil))     ; => 1
```

Calling `car` on a non-cons value is a runtime type error.

---

### `cdr`

```
(cdr <cons>) → value
```

Returns the **tail** (second element) of a cons cell.

```lisp
(cdr (cons 1 2))    ; => 2
(cdr (cons 1 nil))  ; => nil (0)
```

Calling `cdr` on a non-cons value is a runtime type error.

---

### Building and Traversing Lists

```lisp
; sum of a linked list
(let list-sum
  (fn list-sum (xs acc)
    (if (= xs nil)
      acc
      (list-sum (cdr xs) (+ acc (car xs))))))

(let nums (cons 1 (cons 2 (cons 3 (cons 4 (cons 5 nil))))))
(list-sum nums 0)   ; => 15
```

```lisp
; length of a list
(let length
  (fn length (xs n)
    (if (= xs nil)
      n
      (length (cdr xs) (+ n 1)))))

(length (cons 0 (cons 0 (cons 0 nil))) 0)   ; => 3
```

---

## I/O

### `print`

```
(print <expr>) → <expr>
```

Prints the value of `<expr>` to stdout **without** a trailing newline.
Returns the value that was printed.

```lisp
(print "Hello")        ; outputs: Hello
(print 42)             ; outputs: 42
(print (+ 1 2))        ; outputs: 3, returns 3
```

Because `print` returns its argument, it can be used inside larger expressions:

```lisp
(+ (print 10) 5)   ; prints 10, then evaluates to 15
```

---

### `println`

```
(println <expr>) → <expr>
```

Same as `print` but appends a newline `\n` after the value.

```lisp
(println "Hello, world!")   ; Hello, world!\n
(println (+ 2 3))           ; 5\n
```

---

### Printing Different Types

| Value type | Output |
|------------|--------|
| Integer | Decimal digits, e.g. `42`, `-7` |
| Float (tagged double) | Shortest decimal representation via `%g`, e.g. `3.14` |
| Bigint | Full decimal string |
| Bigfloat | Full decimal string |
| String | Raw bytes (no quotes added) |
| Cons cell | Printed as `(car . cdr)` pairs recursively |
| Closure | `<closure>` |
| Nil | *(empty — nothing printed)* |

```lisp
(println (cons 1 (cons 2 nil)))   ; (1 . (2 . 0))
(println nil)                     ; (empty line)
(println (fn (x) x))              ; <closure>
```

---

## Strings

### `strlen`

```
(strlen <s>) → integer
```

Returns the **byte length** of string `<s>` (not the number of Unicode code
points).

```lisp
(strlen "hello")   ; => 5
(strlen "café")    ; => 5  (é is 2 bytes in UTF-8)
(strlen "")        ; => 0
```

---

### `strcc`

```
(strcc <a> <b>) → string
```

Returns a new string that is the concatenation of `<a>` and `<b>`.

```lisp
(strcc "foo" "bar")   ; => "foobar"
(strcc "" "x")        ; => "x"
```

---

### `strref`

```
(strref <s> <i>) → integer
```

Returns the **byte value** (0–255) at byte index `<i>` of string `<s>`. Indices
are zero-based.

```lisp
(strref "ABC" 0)   ; => 65  (ASCII 'A')
(strref "ABC" 2)   ; => 67  (ASCII 'C')
```

---

### `strslice`

```
(strslice <s> <start> <len>) → string
```

Returns a substring of `<s>` starting at byte index `<start>` with byte length
`<len>`.

```lisp
(strslice "hello" 1 3)   ; => "ell"
(strslice "hello" 0 5)   ; => "hello"
```

---

### `numtostr`

```
(numtostr <n>) → string
```

Converts a number to its decimal string representation.

```lisp
(numtostr 42)     ; => "42"
(numtostr 3.14)   ; => "3.14"
(numtostr -100)   ; => "-100"
```

---

### `strtonum`

```
(strtonum <s>) → integer | float | 0
```

Parses a decimal string as a number. Returns `0` (nil) if `<s>` is not a valid
number.

```lisp
(strtonum "42")     ; => 42
(strtonum "3.14")   ; => 3.14
(strtonum "abc")    ; => 0
```

---

### `chr`

```
(chr <codepoint>) → string
```

Returns a one-character UTF-8 string for the given Unicode code point.

```lisp
(chr 65)      ; => "A"
(chr 955)     ; => "λ"
(chr 128512)  ; => "😀"
```

---

## Hash Maps

### `hashmake`

```
(hashmake) → hash-map
```

Allocates a new empty hash map. Keys can be strings or integers; values can be
any `Val`.

```lisp
(let h (hashmake))
```

---

### `hashset`

```
(hashset <map> <key> <val>) → map
```

Inserts or updates the mapping `<key> → <val>` in `<map>`. Returns the same
map (mutation in place).

```lisp
(let h (hashmake))
(hashset h "name" "Alice")
(hashset h 1 100)
```

---

### `hashget`

```
(hashget <map> <key>) → value | 0
```

Returns the value for `<key>`, or `0` (nil) if the key is absent.

```lisp
(hashget h "name")   ; => "Alice"
(hashget h "age")    ; => 0
```

---

### `hashhas?`

```
(hashhas? <map> <key>) → 0 | 1
```

Returns `1` if `<key>` is present in `<map>`, `0` otherwise.

```lisp
(hashhas? h "name")   ; => 1
(hashhas? h "age")    ; => 0
```

---

### `hashdel`

```
(hashdel <map> <key>) → map
```

Removes `<key>` from `<map>`. Returns the same map. No-op if the key is absent.

```lisp
(hashdel h "name")
(hashhas? h "name")   ; => 0
```

---

### `hashkeys`

```
(hashkeys <map>) → list
```

Returns a linked list of all live keys in the map (order unspecified).

```lisp
(let h (hashmake))
(hashset h "a" 1)
(hashset h "b" 2)
(hashkeys h)   ; => ("a" . ("b" . 0))  or some permutation
```

---

## Timing

### `clock`

```
(clock) → integer
```

Returns the current monotonic time in **microseconds**. Useful for benchmarking.

```lisp
(let t0 (clock))
; ... work ...
(let elapsed (- (clock) t0))
(println (strcc "µs: " (numtostr elapsed)))
```

---

## Errors

### `error`

```
(error <msg>) → (does not return)
```

Prints `<msg>` to stderr and exits the interpreter with status `1`.

```lisp
(if (< x 0)
  (error "x must be non-negative")
  x)
```

---

## FFI

### `ffi`

```
(ffi <name> <ret-type> <arg-type>...) → callable
```

Resolves the C symbol `<name>` against the compile-time table in
`ffi_syms.c` (common libc symbols: `fopen`, `fclose`, `sin`, `strlen`,
`getchar`, `calloc`, …) **and** any shared object loaded with
`loadlibrary`.  The returned value behaves like a normal closure.

Under the hood, sel uses **libffi** (vendored as a git submodule) to
build a `ffi_cif` from the declared types and dispatch the call —
arguments are unboxed per their type codes, marshalled through `ffi_call`,
then the result is re-boxed.

Type strings:

| String | C type | Notes |
|--------|--------|-------|
| `"void"` | `void` | return only |
| `"int"` | `int` | signed 32-bit |
| `"uint"` | `unsigned int` | |
| `"long"` | `long` | 64-bit on most ABIs |
| `"uchar"` | `unsigned char` | 1 byte; useful in struct fields |
| `"ushort"` | `unsigned short` | 2 bytes |
| `"float"` | `float` | sel boxes into a double |
| `"double"` | `double` | |
| `"string"` | `char*` | reads from a sel string; returned strings are copied |
| `"ptr"` | `void*` | stored as integer |
| `"<StructName>"` | by-value struct | name of a struct registered via `defstruct` |

```lisp
; call C puts()
(let puts (ffi "puts" "int" "string"))
(puts "hello from C")

; open a file
(let fopen  (ffi "fopen"  "ptr" "string" "string"))
(let fclose (ffi "fclose" "int" "ptr"))
(let fp (fopen "/tmp/test.txt" "w"))
(fclose fp)
```

FFI functions are first-class values: they can be stored in variables, passed
as arguments, or stored in hash maps.

---

### `loadlibrary`

```
(loadlibrary <path>) → nil
```

Compile-time `dlopen(path, RTLD_NOW | RTLD_LOCAL)` plus registration of
the handle on a global list.  Symbols looked up via `(ffi …)` after this
form fall back to the registered handles when the static table misses.
Loaded libraries live for the process's lifetime; there is no
`unloadlibrary`.

```lisp
(loadlibrary "libraylib.so")
(let CloseWindow (ffi "CloseWindow" "void"))
```

The path argument must be a literal string — `loadlibrary` is resolved
at compile time so symbols are visible to subsequent `(ffi …)` forms in
the same source file.

---

## Structs

sel can pass and return C structs **by value** through libffi.  A struct
layout is declared once with `defstruct`, then referenced by name in
`(ffi …)` argument/return positions.

### `defstruct`

```
(defstruct <name> <member-type> ...) → nil
```

Registers a struct layout.  Member types are the same set as `(ffi …)`
(primitives plus previously-registered struct names for nested
by-value members).  libffi computes the size and member offsets via
`ffi_get_struct_offsets`, matching the host C ABI.

```lisp
(defstruct Color uchar uchar uchar uchar)         ; raylib's Color
(defstruct Vector2 float float)
(defstruct Camera2D Vector2 Vector2 float float)  ; nested structs OK
```

Inline fixed-size arrays (`T arr[N]` in C) are not directly expressible —
the binding generator (`genffi.py`) expands them into N consecutive
members of `T`, which has the same byte layout.

---

### `makestruct`

```
(makestruct "<name>" <v0> <v1> ...) → struct
```

Allocates a new struct of the named layout and initialises each field
from the supplied values.  The first argument must be a literal string.

```lisp
(let RAYWHITE (makestruct "Color" 245 245 245 255))
(let pos      (makestruct "Vector2" 100.0 200.0))
```

---

### `structget` / `structset`

```
(structget <s> <i>) → value
(structset <s> <i> <v>) → s
```

Read or mutate field `<i>` (a literal integer index).  Field types
follow the layout declared in `defstruct`; numeric fields are returned
as int/double, string fields are copied to a sel string.

```lisp
(structget RAYWHITE 0)        ; => 245   (the red component)
(structset pos 0 320.0)       ; pos.x = 320.0; returns pos
```

`structset` mutates in place and returns the same struct, so calls
chain.  Field indices are zero-based and match the order given to
`defstruct`.
