---
layout: default
title: Built-in Functions
nav_order: 7
---

# Built-in Functions
{: .no_toc }

These are compiled directly to bytecode instructions rather than being
user-defined functions. They all follow the same S-expression call syntax.

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
