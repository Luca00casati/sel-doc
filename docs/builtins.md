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

### `str-len`

```
(str-len <s>) → integer
```

Returns the **byte length** of string `<s>` (not the number of Unicode code
points).

```lisp
(str-len "hello")   ; => 5
(str-len "café")    ; => 5  (é is 2 bytes in UTF-8)
(str-len "")        ; => 0
```

---

### `str-concat`

```
(str-concat <a> <b>) → string
```

Returns a new string that is the concatenation of `<a>` and `<b>`.

```lisp
(str-concat "foo" "bar")   ; => "foobar"
(str-concat "" "x")        ; => "x"
```

---

### `str-ref`

```
(str-ref <s> <i>) → integer
```

Returns the **byte value** (0–255) at byte index `<i>` of string `<s>`. Indices
are zero-based.

```lisp
(str-ref "ABC" 0)   ; => 65  (ASCII 'A')
(str-ref "ABC" 2)   ; => 67  (ASCII 'C')
```

---

### `str-slice`

```
(str-slice <s> <start> <len>) → string
```

Returns a substring of `<s>` starting at byte index `<start>` with byte length
`<len>`.

```lisp
(str-slice "hello" 1 3)   ; => "ell"
(str-slice "hello" 0 5)   ; => "hello"
```

---

### `num->str`

```
(num->str <n>) → string
```

Converts a number to its decimal string representation.

```lisp
(num->str 42)     ; => "42"
(num->str 3.14)   ; => "3.14"
(num->str -100)   ; => "-100"
```

---

### `str->num`

```
(str->num <s>) → integer | float | 0
```

Parses a decimal string as a number. Returns `0` (nil) if `<s>` is not a valid
number.

```lisp
(str->num "42")     ; => 42
(str->num "3.14")   ; => 3.14
(str->num "abc")    ; => 0
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

### `hash-make`

```
(hash-make) → hash-map
```

Allocates a new empty hash map. Keys can be strings or integers; values can be
any `Val`.

```lisp
(let h (hash-make))
```

---

### `hash-set`

```
(hash-set <map> <key> <val>) → map
```

Inserts or updates the mapping `<key> → <val>` in `<map>`. Returns the same
map (mutation in place).

```lisp
(let h (hash-make))
(hash-set h "name" "Alice")
(hash-set h 1 100)
```

---

### `hash-get`

```
(hash-get <map> <key>) → value | 0
```

Returns the value for `<key>`, or `0` (nil) if the key is absent.

```lisp
(hash-get h "name")   ; => "Alice"
(hash-get h "age")    ; => 0
```

---

### `hash-has`

```
(hash-has <map> <key>) → 0 | 1
```

Returns `1` if `<key>` is present in `<map>`, `0` otherwise.

```lisp
(hash-has h "name")   ; => 1
(hash-has h "age")    ; => 0
```

---

### `hash-del`

```
(hash-del <map> <key>) → map
```

Removes `<key>` from `<map>`. Returns the same map. No-op if the key is absent.

```lisp
(hash-del h "name")
(hash-has h "name")   ; => 0
```

---

### `hash-keys`

```
(hash-keys <map>) → list
```

Returns a linked list of all live keys in the map (order unspecified).

```lisp
(let h (hash-make))
(hash-set h "a" 1)
(hash-set h "b" 2)
(hash-keys h)   ; => ("a" . ("b" . 0))  or some permutation
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
(println (str-concat "µs: " (num->str elapsed)))
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

Looks up the C symbol `<name>` in the dynamic linker and returns an FFI
function value that can be called like a normal closure.

Type strings:

| String | C type |
|--------|--------|
| `"void"` | `void` (return only) |
| `"int"` | `int` |
| `"long"` | `long` |
| `"double"` | `double` |
| `"string"` | `char*` |
| `"ptr"` | `void*` (stored as integer) |

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
