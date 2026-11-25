---
title: "Polynomials, and starting K. N. King's C"
published: 2026-11-26
description: "A day log: a Class 9 polynomials worksheet, the first two chapters of King's C Programming: A Modern Approach, and the small C harness I built to work through it."
image: ./cover.png
tags: [Journal, C, K. N. King, Maths, Polynomials]
category: Learning
draft: false
---

A shorter post than the nand2tetris ones: a log of what I covered today. Two tracks: a maths worksheet, and the first
two chapters of K. N. King's *C Programming: A Modern Approach* (2nd ed.).

## Where things stand

| | |
|---|---|
| Maths | Class 9 ch 2, *Polynomials*: 15 problems, **11 on my own, 4 with help** · worksheet 2 of 93 |
| C | King ch 1–2 done, all ch 2 exercises and projects on my own · **2 of 27 chapters** |
| Next | Class 9 ch 3 worksheet · King ch 3, *Formatted Input/Output* |

## Maths: polynomials

I'm working through school maths again, Class 9 to 12, one worksheet per chapter. This one covered degree and value,
zeroes, the remainder and factor theorems, factorising quadratics and cubics, and the algebraic identities.

The idea that carried most of the worksheet: **dividing by $x - a$ is the same as substituting $a$.** The division
algorithm gives $p(x) = (x - a)\,q(x) + r$, and putting $x = a$ kills the first term, so the remainder is $p(a)$.
Then $x - a$ is a factor exactly when $p(a) = 0$. Long division is only needed when the divisor isn't linear.

The one I needed help with and want to remember: $x^3 + ax^2 + bx - 12$ has both $x - 1$ and $x + 2$ as factors. Find
$a$, $b$ and the third factor. Each factor gives one equation:

$$
\begin{aligned}
p(1) = 0 &\;\Rightarrow\; a + b = 11 \\
p(-2) = 0 &\;\Rightarrow\; 4a - 2b = 20 \;\Rightarrow\; 2a - b = 10
\end{aligned}
$$

Adding them gives $3a = 21$, so $a = 7$, $b = 4$. For the third factor, the constant term has to come out as $-12$:
$(-1)(2)(c) = -12$, so $c = 6$, and $x^3 + 7x^2 + 4x - 12 = (x - 1)(x + 2)(x + 6)$.

The identities give arithmetic shortcuts too: $103 \times 107 = (100 + 3)(100 + 7) = 100^2 + 10 \cdot 100 + 21 = 11021$.

## C: chapters 1 and 2

### A clean compile proves nothing

Chapter 1 is history plus King's advice to let tools find bugs, since C trusts the programmer. I tried it on a
program that prints an `int` and a `float` before giving them values:

| Tool | Result |
|---|---|
| `clang -std=c99` | compiles, 0 warnings |
| add `-Wall` | 2 × `-Wuninitialized` |
| run with `-fsanitize=address,undefined` | no report, prints garbage (`2000097024 0.000000`) |
| run with `-fsanitize=memory` | `use-of-uninitialized-value` |

The surprise was the third row: AddressSanitizer and UBSan don't look for uninitialized reads. That's
MemorySanitizer's job. Each tool covers different bugs, so "no warnings" only means "no warnings from the tools I
switched on".

### Falling off the end of `main`

`int main(void) { printf("hi\n"); }` compiles with no warning under `-std=c99`, even with `-pedantic`. C99 has a rule
just for `main`: reaching its closing `}` returns 0. Under `-std=c89` the exit status is undefined, and gcc's
binary exited with **3**, which is `printf`'s return value (three characters written) left in the return register.
Any other function that ends without a `return` hands its caller garbage, and `-Wall` warns about that.

### Rounding up with integers

Chapter 2's `dweight.c` computes a box's dimensional weight, volume ÷ 166, and the shipping company wants it rounded
**up**. Integer division truncates, so King adds 165 first:

```c title="dweight.c"
weight = (volume + 165) / 166;   /* 960 → 6, not 5 */
```

It's the standard ceiling-division idiom $(a + b - 1)/b$. Write $a = bq + r$: an exact multiple ($r = 0$) gains less
than one full $b$ and stays at $q$, and any leftover ($r \ge 1$) pushes it to $q + 1$. The Linux kernel calls it
`DIV_ROUND_UP`, and glibc has the same formula as `howmany`.

Dividing by `166.0` instead gives `5.783`, which isn't rounded up either, and assigning it to an `int` truncates it
back to 5. Two limits on the integer version: it overflows when $a$ is near `INT_MAX`
(`a / b + (a % b != 0)` is the safe form), and it's off by one for negative $a$, because C99 already truncates
toward zero.

## The harness

I set up a small workspace for the book, [kingc](https://github.com/utsavo-afk/kingc), with the same layout I use for
[PPP3](https://github.com/utsavo-afk/ppp3):

- **clang 22 + a plain Makefile.** Everything compiles as `-std=c99 -Wpedantic`, so code from later standards gets
  flagged. On top of `-Wall -Wextra`, it adds `-Wconversion`, `-Wshadow`, `-Wvla` and `-Wstrict-prototypes`.
- **Debug builds run with AddressSanitizer + UBSan.** Release builds are plain `-O2`.
- **A Rust-style `dbg()`** in `include/dbg.h`, force-included into every program. `dbg(x)` prints the file, line,
  expression, value and type to stderr, and returns the value, so it can wrap any expression. It also prints arrays,
  pointers (NULL ones are flagged) and structs. Release builds compile it away.
- **VS Code + clangd** reading the Makefile's flags, a small clang-tidy set that finds real bugs, format on save, and
  F-keys to build, run and debug the current file.

```txt title="Terminal"
$ make new CH=02 NAME=drill        # new exercise file
$ make run FILE=src/ch02/dweight.c
```

One side effect I noticed: since `dbg.h` pulls in `<stdio.h>`, forgetting `#include <stdio.h>` isn't an error
inside this harness, though it is with plain clang in C99.

The chapter programs so far (`uninit.c`, `pun.c`, `dweight.c`, `dweight2.c`) are in the repo too, one folder per
chapter under `src/`.

## Next

Maths: Class 9 ch 3. C: chapter 3, `printf` conversion specifications and how `scanf` actually matches its input.
