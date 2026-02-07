+++
date = "2017-07-08T07:29:43Z"
highlight = true
math = true
tags = ["math", "python", "binomial", "combinatorics", "programming", "modular arithmetic", "competitive programming"]
title = "Binomial Coefficients Modulo a Prime: Fermat's Theorem and the Non-Adjacent Selection Problem"

[header]
  caption = ""
  image = ""

+++

In the [previous post][efficient-impl], we implemented the closed form $F_{n,m} = \binom{n-m+1}{m}$ using Python's `math.factorial`, and with `scipy` and `sympy`. Here we cover the common competitive-programming case: computing the answer **modulo a large prime** $M$ (e.g. $M = 10^9+7$).

## Why modulo?

In counting problems, the result can be huge even for moderate input. Often the problem asks for the answer modulo a big prime so that it fits in a standard integer type. We could compute the full number and then take the remainder, but that forces expensive long-integer arithmetic. Computing **everything** modulo $M$ from the start is much faster.

## From binomials to modular inverses

We have $F_{n,m} = \binom{n-m+1}{m} = \frac{(n-m+1)!}{m!\,(n-2m+1)!}$. To compute this mod $M$, we need factorials mod $M$ and division mod $M$. Division mod $M$ is multiplication by the **modular inverse**: for prime $M$ and $0 < x < M$, the inverse of $x$ is $x^{M-2} \bmod M$ by [Fermat's little theorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem). In Python we can use `pow(x, M - 2, M)`.

## Implementation

```python
import functools

M = 10**9 + 7

def f_binom_mod(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return binom_mod(n - m + 1, m)

def binom_mod(n, m):
    assert 0 <= m <= n

    return ((fact_mod(n) * inv_mod(fact_mod(m))) % M * inv_mod(fact_mod(n - m))) % M

@functools.lru_cache(maxsize=None)
def fact_mod(m):
    if m <= 1:
        return 1

    return (m * fact_mod(m - 1)) % M

def inv_mod(x):
    return pow(x, M - 2, M)
```

All operations stay in the ring of integers mod $M$. The only non-obvious part is modular division: we replace division by $d$ with multiplication by `inv_mod(d)` using Fermat's little theorem.

## Benchmarks

Compared to computing the full binomial and then taking the remainder, the modular version avoids long arithmetic and is much faster:

```python
fact_mod(10000)  # for caching factorials

funcs = [f_binom_mod, f_binom, f_sci, f_sym]

test(10000, 1000, funcs)
test(10000, 2000, funcs)
test(10000, 3000, funcs)
```

Example output:

```
f(10000,1000): 450169549
  f_binom_mod:   0.0000 sec, x 1.00
      f_binom:   0.0073 sec, x 337.60
        f_sci:   0.0011 sec, x 49.33
        f_sym:   0.0076 sec, x 353.22

f(10000,2000): 75198348
  f_binom_mod:   0.0000 sec, x 1.00
      f_binom:   0.0063 sec, x 368.94
        f_sci:   0.0026 sec, x 153.33
        f_sym:   0.0053 sec, x 308.93

f(10000,3000): 679286557
  f_binom_mod:   0.0000 sec, x 1.00
      f_binom:   0.0060 sec, x 361.12
        f_sci:   0.0056 sec, x 338.13
        f_sym:   0.0053 sec, x 319.02
```

The same pattern—factorials mod $M$ plus Fermat-based inverses—works for any combinatorial formula that can be written in terms of factorials and binomials modulo a prime.

[efficient-impl]: /post/efficient-implementation-non-adjacent-selection/
