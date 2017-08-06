+++
date = "2017-07-06T07:29:43Z"
highlight = true
math = true
tags = ["math","python", "generating functions", "recursion", "recursive functions", "dynamic programming", "memoization", "binomial", "coding interview", "combinarotics"]
title = "Cracking Multivariate Recursive Equations Using Generating Functions"

[header]
  caption = ""
  image = ""

+++


In this post, we return back to the combinatorial problem discussed in [Introduction to Dynamic Programming and Memoization][intro-to-dp] post.
We will show that generating functions may work great not only for single variable case (see [The Art of Generating Functions][gen-func-art]),
but else can be very useful for hacking two-variable relations (and of course, in general for multivariate case too).

## The Problem

For making the post self-contained, we repeat the problem definition here.

**Problem:** Compute the number of ways to choose $m$ elements from $n$ elements such that selected elements in one combination are not adjacent.

For example, for $m=4$ and $m=3$, the answer is $3$, since from the $4$-element set: $\lbrace 1,2,3,4 \rbrace$,
there are three feasible $2$-element combinations: $\lbrace 1,4 \rbrace$, $\lbrace 2,4 \rbrace$, $\lbrace 1,3 \rbrace$.

Another example, for $n=5$ and $m=3$, there is only one $3$-element combination: $\lbrace 1,3,5 \rbrace$.

As we discussed in the first post, there is a nice recursive relation for this problem:

$$ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=0] + [n=m=1] $$

We assume that for any $n < 0$ or $m < 0$, $F\_{n,m} = 0$.
The indicator $[P]$ gives $1$ if the predicate $P$ is true.

# The Generating Function for $F\_{n,m}$

We continue our analysis of the two variable recursion expression:

$$F\_{n,m} = F\_{n-1,m} + F\_{n-2,m-1} + [n=m=0] + [n=m=1]$$

Let's introduce the generating function $\Phi(x,y)$ of two (floating point) variables $x$ and $y$:

$$\Phi(x,y) = \sum\_{n,m} F\_{n,m} x^n y^m$$

Substituting the definition of $F\_{n,m}$ and simplifying the sums, we will get:
$ \Phi(x,y) $
$ = \sum\_{n,m} F\_{n,m} x^n y^m $
$ = \sum\_{n,m} \left(F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=1] + [n=m=0] \right) x^n y^m$
$ = \sum\_{n,m} F\_{n - 1, m} x^n y^m + \sum\_{n,m} F\_{n - 2, m - 1} x^n y^m + x \cdot y + 1 $
$ = x \sum\_{n,m} F\_{n - 1, m} x^{n-1} y^m + x^2 y \sum\_{n,m} F\_{n - 2, m - 1} x^{n-2} y^{m-1} + x \cdot y + 1 $
$ = x \Phi(x,y) + x^2 y \Phi(x,y) + x \cdot y + 1 $

We have just found the simple representation for the generating function:

$$\Phi(x, y) = \frac{1 + x \cdot y}{1 - x - x^2 y}$$

Using the infinite series: $ \frac{1}{1-z} = \sum\_{k\geq 0} z^k $, we can transform the expression as following:
$\Phi(x, y) $
$ = \frac{1 + x \cdot y}{1 - x - x^2 y}$
$ = (1 + x \cdot y) \sum\_{k\geq 0} (x + x^2 y)^k $
$ = (1 + x \cdot y) \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i $
$ = \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i + \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i+1} y^{i+1}$

Introducing the new variables: $n=k+i, m=i$, we cat transform the first expression as following:
$ \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i $
$ = \sum\_{m \geq 0, n \geq 2m} {n-m \choose m} x^{n} y^m $.

And introducing the new variables: $n=k+i+1, m=i+1$, we can transform the second expression similarly:
$ \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i+1} y^{i+1} $
$ = \sum\_{m \geq 1, n \geq 2m-1} {n-m \choose m-1} x^n y^m $.

The last two transformations give us the closed form: $F\_{n, m} $
$ = {n - m \choose m} + {n - m \choose m - 1}$.
Which actually equals to

$$ F\_{n, m} = {n - m + 1 \choose m} $$

Now we can reflect this idea in very trivial Python code:

```Python
import math

def f_binom(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return binom(n - m + 1, m)

def binom(n, m):
    assert 0 <= m <= n

    return math.factorial(n) // math.factorial(m) // math.factorial(n - m)
```

This implementation overperforms significantly the initial DP and memoization solutions.
A naive implementation of `math.factorial()` could be linear in $n$, which still might be faster than DP approach.
Actually, the actual implementation is much more advance, it is written in C and caches values for a small range of the argument.

## Faster Implementations Using `scipy` and `sympy`

More promissing ways for computing binomial coeffitients could be found in the third party libraries: `scipy` and `sympy`:
We can easily install both ones using `pip`-package manager:

```bash
pip install scipy sympy
```

Let's take a look at `scipy.special.comb()` and `sympy.binomial()` functions:

```Python
import scipy.special
import sympy

def f_sci(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return scipy.special.comb(n - m + 1, m, exact=True)

def f_sym(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return sympy.binomial(n - m + 1, m)
```

We can use the same `test()` helper function that we defined in [Introduction to Dynamic Programming and Memoization][intro-to-dp].
Let's run it on all the 5 implementatnions:

```python
funcs = [f_mem, f_dp, f_binom, f_sci, f_sym]
test(6000, 2000, funcs)
```

Which prints the following output:

```
f(6000,2000): 192496093
       f_mem:   6.7195 sec, x 4195.10
        f_dp:   5.3249 sec, x 3324.43
     f_binom:   0.0016 sec, x 1.00
       f_sci:   0.0021 sec, x 1.32
       f_sym:   0.0043 sec, x 2.69
```

This is not a surprise that the first two methods (memoization and DP) are much slower than the other three,
which are based on the binomial coefficients.
Memoization and DP techniques have $\Theta(n \cdot m)$ time complexity.
The `f_binom()`-method uses `math.factorial()` as a subroutine,
which is linear in $n$ and implemented in a naive way.
Note that we ignore here a lot of interesting details, for example,
Python has built-in long arithmetics for integers, which is used here and definetely not cheap.
Also `factorial`-function may use memoization for storing its value.
Other methods based on `sympy` and `scipy.special`, may (and actually do) use memoization as well.
The impact of C-impementation is also out of the scope of the current discassion,
as well as some other "hard-core" optimizations that one can apply.

Testing the implementations on different parameters, we may notice that there is no clear winner between the algorthms.
Probably, the most of the time is spent on the long arithmetic computation.

## Modular Arithmetics

In questions where it is required to count some objects, not rearly the answer might be very big even on very small input.
In such case, typically it is asked to print the answer modulo some big prime integer, let's say, $M=1000^3+7$.
Since Python has built-in long arithmetics, we can apply modulo on the final result,
but executing the entire agorithm with long arithmetics while knowing that only small part of it is really important is very costly,
and of course, not that efficient.

Let's look, briefly, at very simple change we can do for `f_binom` function that will speed up the computation significantly:

```python
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

As we can see, all the operations are computed modulo $M$.
The function `fact_mod` is recursive but uses Memoization.
The most tricky part is how to implemenent modular-division.
From [Fermat's little theorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem),
we know that if $M$ is prime and $0 < x < M$, then $x^{-1} \equiv x^{M-2} \pmod M$.
This allows to compute the multiplicative inverse of $x$ using the Python's built-in function [pow](https://docs.python.org/3/library/functions.html#pow).

Let's test the new approach against other implementations:

```python

fact_mod(10000) # for caching factorials

funcs = [f_binom_mod, f_binom, f_sci, f_sym]

test(10000, 1000, funcs)
test(10000, 2000, funcs)
test(10000, 3000, funcs)
```

It is not a surprise that taking the benefits of modular computations results in the huge speedup in running-time:

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

[intro-to-dp]: /post/intro-to-dp/
[gen-func-art]: /post/gen-func-art/

