---
layout: default
title: Internals
nav_order: 10
---

# Internals
{: .no_toc }

This page documents the compiler pipeline, bytecode format, and VM architecture
for contributors and curious users.

<details open markdown="block">
  <summary>Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Pipeline

```
source text
    │
    ▼
tokenizer.c  ──  Token (flat array of String + kind + line)
    │
    ▼
tokenizer.c  ──  AST (Node tree: NODE_NUMBER / NODE_SYMBOL / NODE_LIST)
    │
    ▼
compiler.c   ──  Chunk (array of Instructions + register count)
    │
    ▼
vm.c         ──  String (result value)
```

---

## Source Files

| File | Responsibility |
|------|---------------|
| `tokenizer.c` / `tokenizer.h` | Lexing and AST construction |
| `compiler.c` / `compiler.h` | AST → bytecode; all compile-time optimisations |
| `vm.c` / `vm.h` | Bytecode interpreter, function registry, emit helpers |
| `main.c` | REPL and file runner |
| `sel_string.h` / `sel_string.c` | Tagged `String` value type |
| `bigint.c` / `bigint.h` | Arbitrary-precision integer arithmetic |
| `bigfloat.c` / `bigfloat.h` | Arbitrary-precision float arithmetic |
| `arena.c` / `arena.h` | Bump-pointer arena allocator |

---

## Value Type — `String`

Every sel runtime value is a `String` struct (`sel_string.h`):

```c
typedef struct {
    char*           data;     /* NULL → numeric / closure / cons */
    int             size;
    long            ival;     /* tagged integer when data==NULL, is_float==0 */
    struct Closure* closure;  /* non-NULL → closure value */
    struct Cons*    cons;     /* non-NULL → cons cell */
    double          fval;     /* tagged double when data==NULL, is_float!=0 */
    int             is_float; /* non-zero: tagged double or bigfloat string */
} String;
```

Dispatch rules:

| `data` | `closure` | `cons` | `is_float` | Kind |
|--------|-----------|--------|------------|------|
| NULL | NULL | NULL | 0 | Tagged integer (`ival`) |
| NULL | NULL | NULL | ≠0 | Tagged double (`fval`) |
| ≠NULL | NULL | NULL | 0 | String / bigint bytes |
| ≠NULL | NULL | NULL | ≠0 | Bigfloat decimal string |
| NULL | ≠NULL | NULL | — | Closure |
| NULL | NULL | ≠NULL | — | Cons cell |

---

## Bytecode — Opcodes

All opcodes are in the `OpCode` enum in `vm.h`. Each instruction has:
- `op` — the opcode
- `dst`, `a`, `b` — register indices (overloaded for jumps and calls — see below)
- `value` — a `String` literal (for LOAD_CONST, LOAD_VAR, STORE_VAR, IMM ops)
- `arg_regs` / `argc` — argument register array (for CALL variants)
- `line` — source line number

### Variable Access

| Opcode | Effect |
|--------|--------|
| `OP_LOAD_CONST` | `dst = value` |
| `OP_LOAD_VAR` | `dst = env[value]` |
| `OP_STORE_VAR` | `env[value] = dst` |
| `OP_MOVE` | `dst = a` |

### Arithmetic

| Opcode | Effect |
|--------|--------|
| `OP_ADD` | `dst = a + b` |
| `OP_SUB` | `dst = a - b` |
| `OP_MUL` | `dst = a * b` |
| `OP_DIV` | `dst = a / b` |
| `OP_MOD` | `dst = a % b` |
| `OP_ADD_IMM` | `dst = a + value` (immediate constant in `value` field) |
| `OP_SUB_IMM` | `dst = a - value` |
| `OP_MUL_IMM` | `dst = a * value` |

### Comparison

| Opcode | Effect |
|--------|--------|
| `OP_LT` | `dst = (a < b)` |
| `OP_EQ` | `dst = (a == b)` |
| `OP_LT_IMM` | `dst = (a < value)` |
| `OP_GT_IMM` | `dst = (a > value)` |
| `OP_EQ_IMM` | `dst = (a == value)` |

### Boolean

| Opcode | Effect |
|--------|--------|
| `OP_NOT` | `dst = !a` |

### Bitwise

| Opcode | Effect |
|--------|--------|
| `OP_BAND` | `dst = a & b` |
| `OP_BOR` | `dst = a \| b` |
| `OP_BXOR` | `dst = a ^ b` |
| `OP_BNOT` | `dst = ~a` |
| `OP_SHL` | `dst = a << b` |
| `OP_SHR` | `dst = a >> b` |

### Control Flow

| Opcode | Effect |
|--------|--------|
| `OP_JMP` | `ip = a` |
| `OP_JMP_IF_FALSE` | `if falsy(dst): ip = a` |
| `OP_HALT` | stop; return `regs[dst]` |

### Calls and Closures

| Opcode | Effect |
|--------|--------|
| `OP_CALL` | Push new frame; `a` = function id |
| `OP_CALL_VAL` | Push new frame; `a` = register holding closure |
| `OP_TAIL_CALL` | Reuse current frame; `a` = function id |
| `OP_TAIL_CALL_VAL` | Reuse current frame; `a` = register holding closure |
| `OP_RET` | Pop frame; write `regs[dst]` into caller's `ret_dst` |
| `OP_MAKE_CLOSURE` | `dst = new Closure(fn_id=a, captured_env=copy of cur_env)` |

### List

| Opcode | Effect |
|--------|--------|
| `OP_CONS` | `dst = cons(a, b)` |
| `OP_CAR` | `dst = car(a)` |
| `OP_CDR` | `dst = cdr(a)` |

### I/O

| Opcode | Effect |
|--------|--------|
| `OP_PRINT` | print `a`; `dst = a` |
| `OP_PRINTLN` | print `a` + newline; `dst = a` |

---

## VM Architecture

The VM is **iterative** (no C recursion for sel function calls). The key data
structures are:

### VMFrame

```c
typedef struct {
    Chunk* chunk;
    int    ip;
    int    regs_base;  /* offset into the global register pool */
    int    ret_dst;    /* destination register in caller's frame */
} VMFrame;
```

### Pools

| Pool | Max size | Allocation |
|------|----------|------------|
| Frame stack (`vm_frame_stack`) | 200,000 frames | `realloc`-grown |
| Register pool (`vm_reg_pool`) | 2,000,000 slots | `realloc`-grown |
| Per-frame env (`vm_envs`) | One per frame | Arena-allocated per call |

Register pools are **flat arrays**. Each frame gets a contiguous slice starting
at `regs_base`. No frame sees another frame's registers.

### Call Dispatch

`OP_CALL`:
1. Advance caller `ip` past the `CALL` instruction.
2. Grow pools if needed.
3. Push a new `VMFrame` at `frame_top + 1`.
4. Arena-allocate a callee `Env`; bind argument registers by name.
5. Switch `cur`, `cur_env`, `regs` to the callee frame.

`OP_TAIL_CALL`:
1. Reuse the **current** frame (update `chunk`, reset `ip = 0`).
2. Reset `cur_env`; rebind arguments.
3. No frame growth — O(1) stack.

`OP_RET`:
1. Write return value into caller's `regs[ret_dst]`.
2. Shrink `regs_top` back to caller's `regs_base + caller->chunk->reg_count`.
3. Decrement `frame_top`.
4. Switch `cur`, `cur_env`, `regs` back to the caller.

---

## Compiler Optimisations

Nine optimisation passes are applied at compile time, in order:

### 1. Constant Folding

Arithmetic, comparison, boolean, and bitwise expressions on **literal**
operands are evaluated at compile time and replaced by a single `LOAD_CONST`.

```lisp
(+ 2 (* 3 4))   ; → LOAD_CONST 14
```

Overflow, bigfloat, and bigint cases are deferred to the runtime to keep the
compiler simple.

### 2. Dead Branch Elimination

`if` forms with a constant condition compile only the live branch.

```lisp
(if 1 "yes" "no")   ; → LOAD_CONST "yes"  (else branch is never emitted)
```

### 3. Escape Analysis

Inside a function body that contains **no inner `fn` expressions**, no closure
can capture the function's `let` bindings. Therefore `STORE_VAR` instructions
for those bindings are suppressed — values live only in registers.

### 4. SSA Register Tracking

`let` bindings are tracked by name → register in a compile-time `VarScope`.
Subsequent references to the name resolve directly to the register, avoiding
`LOAD_VAR` lookups for the common case.

### 5. Dead Parameter Elimination

Function parameters that are never referenced in the body are never loaded.
No `LOAD_VAR` is emitted for them.

### 6. Destination-Hint Propagation

The compile function accepts a `dst` hint register. Sub-expressions write
directly into the requested register, avoiding a `MOVE` to copy the result.

### 7. Immediate-Operand Folding

When one operand of `+`, `-`, `*`, `=`, `<`, or `>` is a compile-time
constant, the compiler emits an `*_IMM` instruction (e.g. `OP_ADD_IMM`) that
carries the constant inline. This saves a `LOAD_CONST` + register.

### 8. Tail Call Optimisation

Calls in tail position emit `OP_TAIL_CALL` / `OP_TAIL_CALL_VAL`, which reuse
the current frame. See [Tail Calls](tail-calls).

### 9. Function Inlining

Small, pure functions (body ≤ 8 real instructions, no nested calls or
closures, no free-variable loads) are inlined at call sites. Inlining is
attempted **before** the tail-call check, so tail calls to eligible functions
are also inlined.

Eligibility guard: `f->compiled` must be `1`. This prevents inlining a
function whose body is still being compiled (which would cause infinite
recursion in the compiler).

---

## Arena Allocator

All heap memory used by the compiler and VM goes through a bump-pointer arena
(`arena.c`). At program exit (or REPL exit), `arena_release()` frees everything
in a single call. There are no individual `free` calls in the hot path.
