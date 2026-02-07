+++
date = "2017-07-07T07:29:43Z"
highlight = true
math = true
tags = ["math", "python", "binomial", "combinatorics", "programming", "dynamic programming", "memoization", "coding interview"]
title = "Efficient Implementation of the Non-Adjacent Selection Formula"

[header]
  caption = ""
  image = ""

+++

In the [previous post][two-var-recursive], we derived the closed form for the non-adjacent selection problem:

$$ F_{n, m} = {n - m + 1 \choose m} $$

Now we discuss how to implement this efficiently in Python—from a simple factorial-based solution to library implementations. For the common case of computing the answer **modulo a large prime** (e.g. in competitive programming), see the [next post][binom-mod].

## Fast Solutions Based on Binomials

We can reflect the closed form in very trivial Python code:

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

This implementation overperforms significantly the initial DP and memoization solutions from [Introduction to Dynamic Programming and Memoization][intro-to-dp].
A naive implementation of `math.factorial()` might make $n$ multiplications.
This could still be faster than doing $\Theta(n)$ additions in DP approach.

The actual implementation of `math.factorial()` is written in C
and probably has precomputed results for some range of $n$
and might even cache the results for bigger $n$.

## Implementations Based on `scipy` and `sympy` Libraries

Several third party libraries provide a functionality to compute binomial coefficients.
Let's take a look at `scipy` and `sympy`.

We can install both of them using `pip`-package manager:

```bash
pip install scipy sympy
```

We can easily write two implementations of $F_{n,m}$ which will call `scipy.special.comb()` or `sympy.binomial()` functions:

```Python
import scipy.special

def f_sci(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return scipy.special.comb(n - m + 1, m, exact=True)
```

The second one is very similar to the first one:

```Python
import sympy

def f_sym(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    return sympy.binomial(n - m + 1, m)
```

We can use the same `test()` helper function that we defined in [Introduction to Dynamic Programming and Memoization][intro-to-dp].
Let's run it on all the 5 implementations:

```python
funcs = [f_mem, f_dp, f_binom, f_sci, f_sym]
test(6000, 2000, funcs)
```

It will print something similar to following output:

```
f(6000,2000): 192496093
       f_mem:   6.7195 sec, x 4195.10
        f_dp:   5.3249 sec, x 3324.43
     f_binom:   0.0016 sec, x 1.00
       f_sci:   0.0021 sec, x 1.32
       f_sym:   0.0043 sec, x 2.69
```

The first two methods, which are based on memoization and DP, are much slower than the last three,
which are based on the binomial coefficients.

## The Intuition for the Time Complexity Analysis

DP and memoization makes $O(n^2)$ of "addition" operations over long integers.
The long integers are bounded by $F_{n,m}$ value, which could be bounded by $2^n$.
One "addition" operation takes $O(N)$ time for $N$-digit integer input.
For our case $N$ could be bounded by $ O(\log 2^n)$ $ = O(n)$.
So the total time complexity is bounded by
$ O(n^2 \cdot N) $
$ = O(n^2 \cdot \log 2^n) $
$ = O(n^2 \cdot n) $
$ = O(n^3)$.

The binomial based solutions make $O(n)$ "multiplication" operations over long integers
The long integers could be bounded by $O(n!)$ $ = O(n^n)$.
The "multiplication" operation could be implemented in a naive way which runs $O(N^2)$ in time and is used for a small input.
But it also has a more advanced implementation, which takes $O(N^{\log_2 3})$ $ = O(N^{1.59})$ and is used on big integers.
Note that here $N$ denotes the number of digits in the input long integers for multiplication.
In this case, $N$ is bounded by
$ O(\log n!) $
$ = O(\log (n^n)) $
$ = O(n \log n) $.
The total time complexity of the binomial based implementations is bounded by
$ O\left(n \cdot N^{\log_2 3}\right) $
$ = O\left(n \cdot (\log n!)^{\log_2 3}\right)$
$ = O\left(n \cdot (n \log n)^{1.59}\right)$
$ = O\left(n^{2.59} \cdot (\log n)^{1.59}\right)$.

This is not really a formal proof, but it gives some intuition why the last approach overperforms the former one.
In practice, the results of factorial computation are cached,
therefore we observe even bigger gap in performance (yeahh again not formal claim, just an intuition).

You can play with running tests on different $n$ and $m$.
What I saw that actually there is no clear winner between the last 3 implementations.
Probably, the most of the time is spent on the long arithmetic computation.

When the problem asks for the answer **modulo a large prime** (e.g. $10^9+7$), computing everything mod $M$ from the start is much faster than using long integers. We cover that in a [separate post][binom-mod]: binomial coefficients modulo a prime using Fermat's little theorem.

[intro-to-dp]: /post/intro-to-dp/
[two-var-recursive]: /post/two-var-recursive-func/
[binom-mod]: /post/binomial-modulo-prime/
