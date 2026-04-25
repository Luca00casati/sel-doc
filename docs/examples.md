---
layout: default
title: Examples
nav_order: 11
---

# Examples
{: .no_toc }

A collection of sel programs demonstrating common patterns.

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Fibonacci

### Tail-recursive (fast)

```lisp
(let fib
  (fn fib (n a b)
    (if (= n 0)
      a
      (fib (- n 1) b (+ a b)))))

(println (fib 50 0 1))    ; 12586269025
(println (fib 100 0 1))   ; 354224848179261915075
```

### Naive recursive (for illustration)

```lisp
(let fibnaive
  (fn fibnaive (n)
    (if (< n 2)
      n
      (+ (fibnaive (- n 1))
         (fibnaive (- n 2))))))

(fibnaive 20)   ; 6765  (slow for large n)
```

---

## Factorial

```lisp
(let fac
  (fn fac (n acc)
    (if (= n 0)
      acc
      (fac (- n 1) (* acc n)))))

(println (fac 20 1))   ; 2432902008176640000
(println (fac 30 1))   ; bigint result, no overflow
```

---

## List Operations

```lisp
; --- building lists ---
(let iota
  (fn iota (n acc)
    (if (= n 0)
      acc
      (iota (- n 1) (cons n acc)))))

(let nums (iota 5 nil))   ; (1 2 3 4 5)

; --- map ---
(let map
  (fn map (f xs)
    (if (= xs nil)
      nil
      (cons (f (car xs)) (map f (cdr xs))))))

; --- filter ---
(let filter
  (fn filter (pred xs)
    (if (= xs nil)
      nil
      (if (pred (car xs))
        (cons (car xs) (filter pred (cdr xs)))
        (filter pred (cdr xs))))))

; --- fold ---
(let fold
  (fn fold (f acc xs)
    (if (= xs nil)
      acc
      (fold f (f acc (car xs)) (cdr xs)))))

; sum of squares of even numbers 1..10
(let evens
  (filter (fn (x) (= (% x 2) 0))
          (iota 10 nil)))

(let sumsq
  (fold (fn (acc x) (+ acc (* x x))) 0 evens))

(println sumsq)   ; 220  (4+16+36+64+100)
```

---

## Higher-Order Functions / Currying

```lisp
(let compose
  (fn (f g)
    (fn (x) (f (g x)))))

(let double  (fn (x) (* x 2)))
(let inc     (fn (x) (+ x 1)))

(let doublethen (compose inc double))
(doublethen 5)   ; => 11

(let incthen (compose double inc))
(incthen 5)   ; => 12
```

---

## FizzBuzz

```lisp
(let fizzbuzz
  (fn fizzbuzz (n)
    (if (= (% n 15) 0) (println "FizzBuzz")
    (if (= (% n  3) 0) (println "Fizz")
    (if (= (% n  5) 0) (println "Buzz")
                       (println n))))))

(let run
  (fn run (i limit)
    (if (> i limit)
      nil
      (let _ (fizzbuzz i)
        (run (+ i 1) limit)))))

(run 1 20)
```

---

## Bitwise Operations

```lisp
; count set bits (popcount)
(let popcount
  (fn popcount (n acc)
    (if (= n 0)
      acc
      (popcount (>> n 1) (+ acc (& n 1))))))

(popcount 255 0)   ; => 8
(popcount 1023 0)  ; => 10
```

---

## Arbitrary-Precision Arithmetic

sel automatically uses bigint / bigfloat when values exceed native size:

```lisp
; 2^100 — far beyond 64-bit integers
(let pow2
  (fn pow2 (n acc)
    (if (= n 0)
      acc
      (pow2 (- n 1) (* acc 2)))))

(println (pow2 100 1))
; 1267650600228229401496703205376

; exact pi to 50 decimal places (bigfloat literal)
(let pi 3.14159265358979323846264338327950288419716939937510)
(println pi)
```

---

## String Output

```lisp
(let greet
  (fn (name)
    (print "Hello, ")
    (println name)))

(greet "sel")    ; Hello, sel
(greet "world")  ; Hello, world
```
