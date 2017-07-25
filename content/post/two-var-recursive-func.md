+++
date = "2017-07-04T07:29:43Z"
highlight = true
math = true
tags = ["math"]
title = "Cracking Multivariate Recursive Equations Using Generating Functions"

[header]
  caption = ""
  image = ""

+++


In this post, we consider a combinatorial problem.
It has a nice recursive solution, which could be implemented efficiently using Dynamic Programming technique.
Then using generating function, we find the closed formula, expressed with the binomial coefficients.
Finaly, we show that the closed formula gives us much faster way to compute the result than the the DP solution.

Generating functions are usually applied to the case of single variable recursive function.
But actually, the technique may be extended to multivariate recursive functions, or even for a system of recursive functions.

Sometimes the generating functions may give a fantastic result, and here we discuss one of such cases.

## The Problem

We consider a problem that has a nice DP (Dynamic Programming) solution.
Similar questions might be asked on coding interviews or may be suggested as trivial problems on coding contests.

**Problem:** Compute the number of ways to choose m elements from n elements in a way that resulted elements are not adjacent.

For example, $F\_{4,2}=3$ since from the 4-elements set: $\lbrace 1,2,3,4 \rbrace$,
there are three feasible 2-elements combinations: $\lbrace 1,4 \rbrace$, $\lbrace 2,4 \rbrace$, $\lbrace 1,3 \rbrace$.
And $F\_{5,3}=1$, since there is only one feasible 3-elements combination: $\lbrace 1,3,5 \rbrace$.

Let's make recursive function for $ F\_{n, m} $ by looking at $n$-th element. We can either:

* Skip the $n$-th element, then $ F\_{n, m} = F\_{n-1, m} $.
* Pick the $n$-th element, then $ F\_{n, m} = F\_{n-2, m-1} $.

The following recursive function: $ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} $,
with the corner cases: $F\_{0, 0} = F\_{1, 1} = 1$.

The function $F$ has a straighforward recursive implementation in any programming language.
But the naive recursive solution will have exponential in $n$ time complexity and will be very slow. 
Let's look at the following `Python 3` code snippet that uses memoization technique:

```python
import functools
import sys

sys.setrecursionlimit(100000)

@functools.lru_cache(maxsize=None)
def f_mem(n, m):
    if n < 0 or m < 0:
        return 0

    if n + 1 < 2 * m:
        return 0

    if n == m == 0:
        return 1

    if n == m == 1:
        return 1

    return f_mem(n-1, m) + f_mem(n-2, m-1)
```

Nice `@functools.lru_cache(maxsize=None)` notation created a wrapper on the `f_mem` function and internally caches results of all calls.
Such caching (or memoization) improves significantly the speed of the recursion, and basically reduces the number of calls to something like $n \cdot m$.
The recursion still may fail on the stack overflow even on relatively small values of $n$.
That is why we increase the stack size, which is easy to do in Python by calling `sys.setrecursionlimit()`-method.

The next implementation is iterative DP (Dynamic Programming).
Basically, it fills out the $n \times m$ table starting from the low values of $n$ and $m$.

```python
def f_dp(n, m):
    assert n >= 0 and m >= 0

    if n+1 < 2*m:
        return 0

    table = [[0] * (m+1) for _ in range(n+1)]

    table[0][0] = 1
    if n >= 1:
        table[1][0] = 1
        if m >= 1:
            table[1][1] = 1

    for i in range(2, n+1):
        table[i][0] = 1
        for j in range(1, min(m, (i + 1) // 2) + 1):
            table[i][j] = table[i-1][j] + table[i-2][j-1]

    return table[n][m]
```

The time and space complexities are $O(n\cdot m) $.
More accurate upper bound for the running time would be $O(n\cdot \min(n,m))$.
It is not hard to notice that the space consumption could be improved to $O(m)$.

## The Generating Function

The notion of generation function and its application for solving recursive equations are very well known.
Let's consider how this works on the example of Fibonacci numbers.
Readers who are familiar with one-variable case may jump to the next section where we discuss how to apply the generation functions on $F\_{n,m}$.

For the Fibonacci numbers, defined by the recursion $f\_n = f\_{n-1} + f\_{n-2} + [n=0]$.
We assume that for any $n < 0$, $f\_n = 0$. The indicator $[n=0]$ equals to 1 only if $n=1$.
Together with the main case, starting from $n=0$, it produces the following sequence: 1, 1, 2, 3, 5, 8, 13, $\dots$.

The generating function for $f\_n$ is defined as a infinit sum of one variable $x$: $\Phi(x) = \sum\_{n\geq 0} f\_n\cdot x^n$.
Usually, it is hard to build an intuition why it is defined that way and why it could be useful.
So let's just focuse on what we can do with this and we start from substituting the defintion of Fibonacci recursion into the formular of generation function:

$$\Phi(x) = \sum\_n f\_n\cdot x^n = \sum\_n (f\_{n-1} + f\_{n-2} + [n=0]) \cdot x^n $$

$$ = \sum\_n f\_{n-1}\cdot x^n  + \sum\_n f\_{n-2}\cdot x^n  + \sum\_n [n=0]\cdot x^n  $$

$$ = x \cdot \sum\_n f\_{n-1}\cdot x^{n-1}  + x^2\cdot \sum\_n f\_{n-2}\cdot x^{n-2}  + 1\cdot x^0 $$

$$ = x \cdot \sum\_n f\_n\cdot x^n  + x^2\cdot \sum\_n f\_n\cdot x^n  + 1 =  x \cdot \Phi(x) + x^2\cdot \Phi(x)  + 1 $$

And we can obtain the generating function for $f\_n$: $\Phi(x) = \frac{1}{1 - x - x^2}$.
Nice, but what can we do with this "magic" formula?
We can hack it in different ways, and depending on our next step, we can get different expression for $f\_n$.

One of the standard ways is to split the fraction into two simple ones of the form: $\frac{A}{B-x}$.
Note that the roots of the quadratic equation: $1 - x - x^2 = 0$ are $x\_0=-\phi$ and $x\_1=\phi^{-1}$, where $\phi$ is the _Goldan Ratio_: 
$\phi=\frac{1+\sqrt{5}}{2}=1.618\dots$.

A quick test: 
$ -(x-x\_0)\cdot(x-x\_1) = -(x+\phi)\cdot \left(x-\phi^{-1}\right) $
$ =-x^2 - x\cdot\left(\phi - \phi^{-1}\right) + 1= -x^2 - x + 1 $.

This results in $\Phi(x)=\frac{1}{1-x-x^2} = -\frac{1}{(x+\phi)\cdot\left(x-\phi^{-1}\right)} = \frac{A}{x+\phi} - \frac{A}{x-\phi^{-1}}$
$ = A\cdot\phi^{-1}\cdot(1+x/\phi) + A\cdot\phi\cdot(1 - x\cdot\phi)$
, where $A=\frac{1}{\phi+\phi^{-1}}$.

We can apply the infinit series $1-z = \sum\_i z^i$ for each expression and we will get:
$\Phi(x) = A\cdot\phi^{-1}\cdot \sum\_i (-x/\phi)^i + A\cdot\phi\cdot \sum\_i (x\cdot\phi)^i $
$ = A\cdot\phi^{-1}\cdot \sum\_i (-\phi)^{-i}\cdot x^i + A\cdot\phi\cdot \sum\_i \phi^i\cdot x^i $
$ = \sum\_i \left(A\cdot\phi^{-1}\cdot(-\phi)^{-i} + A\cdot\phi\cdot \phi^i \right) \cdot x^i $.

The term before $x^i$ is nothing but $f\_i$, which means that $f\_n=A\cdot\phi^{-1}\cdot(-\phi)^{-n} + A\cdot\phi\cdot \phi^n$.



# The Generating Function for $F\_{n,m}$
 
Let's introduce the generating function: $\Phi(x,y) = \sum\_{n,m} F\_{n,m}\cdot x^n y^m$.
Substituting the definition of $F\_{n,m}$ and simplifying the sums, we will get:

$$ \Phi(x,y) = \sum\_{n,m} F\_{n,m}\cdot x^n y^m $$

$$ = \sum\_{n,m} \left(F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=1] + [n=m=0] \right)\cdot x^n y^m$$

$$ = \sum\_{n,m} F\_{n - 1, m} \cdot x^n y^m  + \sum\_{n,m} F\_{n - 2, m - 1} \cdot x^n y^m + x \cdot y + 1 $$

$$ = x\cdot\sum\_{n,m} F\_{n - 1, m} \cdot x^{n-1} y^m + x^2y\cdot\sum\_{n,m} F\_{n - 2, m - 1} \cdot x^{n-2} y^{m-1} + x \cdot y + 1 $$

$$ = x\cdot \Phi(x,y)  + x^2y\cdot \Phi(x,y) + x \cdot y + 1 $$

Now we find the simple representation: $\Phi(x, y) = \frac{1 + x \cdot y}{1 - x - x^2 \cdot y}$.
We can convert it to an infinite series, using: $ \frac{1}{1-z} = \sum\_{k\geq 0} z^k $:

$$\Phi(x, y) = \frac{1 + x \cdot y}{1 - x - x^2 \cdot y}$$

$$ = (1 + x \cdot y) \cdot \sum\_{k\geq 0} (x+x^2\cdot y)^k  $$

$$ = (1 + x \cdot y) \cdot \sum\_{0 \leq i \leq k} {k \choose i} \cdot x^{k+i} \cdot y^i  $$

Introduce $n=k+i, m=i$, then

$$
\sum\_{0 \leq i \leq k} {k \choose i} \cdot x^{k+i} \cdot y^i
 = \sum\_{m \geq 0, n \geq 2m} {n-m \choose m} \cdot x^{n} \cdot y^m
$$

Introduce $n=k+i+1, m=i+1$, then

$$
x\cdot y \cdot \sum\_{0 \leq i \leq k} {k \choose i} \cdot x^{k+i} \cdot y^i
 = \sum\_{0 \leq i \leq k} {k \choose i} \cdot x^{k+i+1} \cdot y^{i+1}
$$

$$ = \sum\_{m \geq 1, n \geq 2m-1} {n-m \choose m-1} \cdot x^n \cdot y^m$$

The last two transformations give us the closed form: $F\_{n, m} = {n - m \choose m} + {n - m \choose m - 1}$.
Which actually equals to $F\_{n, m} = {n - m + 1 \choose m}$.

Now we can implement the solution in very trivial Python code:

```Python
import math

def f_binom(n, m):
    assert n >= 0 and m >= 0

    return binom(n-m+1, m) if n+1 >= 2*m else 0

def binom(n, m):
    assert 0 <= m <= n

    return math.factorial(n) // math.factorial(m) // math.factorial(n-m)
```

This implementation runs much faster than our initial DP solution. The reason for this is that `math.factorial()` is implemented in C. If we know the limits on $n$ am $m$, we can do memoization for factorial with just $O(\max(n,m))$ extra memory, which may give even more boost in time.

## Faster Implementations Using `scipy` and `sympy`

But more promissing optimization is to find a library implementing binomial coefficient, let's look at `scipy.special.comb()` and `sympy.binomial()`:
:
 
```Python
import scipy.special

def f_sci(n, m):
    assert n >= 0 and m >= 0

    return scipy.special.comb(n-m+1, m, exact=True) if n+1 >= 2*m else 0

import sympy

def f_sym(n, m):
    assert n >= 0 and m >= 0

    return sympy.binomial(n-m+1, m) if n+1 >= 2*m else 0
```

You can easily install both by running:

```bash
pip install scipy sympy
```

In order to measure the time and to check the correctness, let's use the following helper function:

```python
import timeit

M=1000000007

def test(n, m, funcs, number=1, module=__name__, M=M):
    f_mem.cache_clear()
    results = []
    func_times = []
    for func in funcs:
        stmt='{}({},{})'.format(func.__name__, n, m)
        setup='from {} import {}'.format(module, func.__name__)
        t = timeit.timeit(stmt=stmt, setup=setup, number=number)
        func_times.append(t)
        results.append(func(n, m) % M)

    assert len(set(results)) <= 1

    if results:
        print('f({},{}): {}'.format(n, m, results[0]))

    best_time = min(func_times)
    for i, func in enumerate(funcs):
        func_time = func_times[i]
        print('{:>8}: {:8.4f} sec, x {:.2f}'.format(func.__name__, func_time, func_time/best_time))
```

We can run it as following:

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

The function $test()$ computes $F\_{6000, 2000}$ using five different ways.
It validates that all the solutions are consistent and produce the same result.
It also measures the time it takes to execute each method.
For each run, it also prints the relative factor computed based on the fastest case.
The function's result is printed modulo $M$.

This is not a surprise that the first two methods (memoization and DP) are much slower than the other three, which are based on the binomial coefficients.
Memoization and DP techniques have $Theta(n \cdot m) time complexity.
The `f_binom`-method uses $factorial$ as a subroutine, which is linear in $n$ implemented in a naive way.
Note that we ignore here a lot of interesting details, for example, Python has builtin integer long arithmetics, which is used here and definetely not cheap.
Also `factorial`-function may use memoization for storing its value.
Other methods based on `sympy` and `scipy.special`, may (and actually do) use memoization as well.  
The impact of C-impementation is also out of scope of the current discassion as well as some other hard-core optimizations that one can apply.

Let's run the fastest algorithms on the different values of $m$:

```python
test(6000,  500, funcs[2:])
test(6000, 1000, funcs[2:])
test(6000, 1500, funcs[2:])
test(6000, 2000, funcs[2:])
```

We may notice that there is no clear winner between the algorthms. It seems that `f_sym` runs slower but for $F\_{n, m} the range is pretty small, probably most of the time is spent on the long arithmetic computation:

```
f(6000,500): 940876663
 f_binom:   0.0038 sec, x 12.15
   f_sci:   0.0003 sec, x 1.00
   f_sym:   0.0046 sec, x 14.79

f(6000,1000): 491957471
 f_binom:   0.0025 sec, x 3.70
   f_sci:   0.0007 sec, x 1.00
   f_sym:   0.0037 sec, x 5.48

f(6000,1500): 325152517
 f_binom:   0.0019 sec, x 1.48
   f_sci:   0.0013 sec, x 1.00
   f_sym:   0.0035 sec, x 2.67

f(6000,2000): 192496093
 f_binom:   0.0016 sec, x 1.00
   f_sci:   0.0024 sec, x 1.49
   f_sym:   0.0033 sec, x 2.06
```


