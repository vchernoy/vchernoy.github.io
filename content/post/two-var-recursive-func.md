+++
date = "2017-07-04T07:29:43Z"
highlight = true
math = true
tags = ["math"]
title = "Solving a Two Variable Recursive Equation Using Generating Functions"

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

Let's make recursive function for $ F_{n, m} $ by looking at $n$-th element. We can either:

* Skip the $n$-th element, then $ F\_{n, m} = F\_{n-1, m} $.
* Pick the $n$-th element, then $ F\_{n, m} = F\_{n-2, m-1} $.

The following recursive function: $ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} $,
with the corner cases: $F\_{0, 0} = F\_{1, 1} = 1$.

Here is very simple code in Python that computes the function, using the DP technique:

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
More accurate upper bound for the running time is $O(n\cdot \min(n,m))$.
One can also improve space consumption to $O(m)$.

## The Generating Function

Let's introduce the generating function: $\Phi(x,y) = \sum_{n,m} F\_{n,m}\cdot x^n y^m$.
Substituting the definition of $F\_{n,m}$ and simplifying the sums, we will get:

$$ \Phi(x,y) = \sum_{n,m} F\_{n,m}\cdot x^n y^m $$

$$ = \sum_{n,m} \left(F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=1] + [n=m=0] \right)\cdot x^n y^m$$

$$ = \sum\_{n,m} F\_{n - 1, m} \cdot x^n y^m  + \sum\_{n,m} F_{n - 2, m - 1} \cdot x^n y^m + x \cdot y + 1 $$

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

But more promissing optimization is to find a library implementing binomial coefficient, let's look at `scipy.special.comb()`:

```Python
import scipy.special

def f_sci(n, m):
    assert n >= 0 and m >= 0

    return scipy.special.comb(n-m+1, m, exact=True) if n+1 >= 2*m else 0
```

and `sympy.binomial()`:

```Python
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

def test(n, m, funcs, number=100, module='__main__'):
    results = []
    func_times = []
    for func in funcs:
        results.append(func(n, m))

        stmt='{}({},{})'.format(func.__name__, n, m)
        setup='from {} import {}'.format(module, func.__name__)
        t = timeit.timeit(stmt=stmt, setup=setup, number=number)
        func_times.append(t)

    assert len(set(results)) <= 1

    if results:
        print('f({},{}): {}'.format(n, m, results[0]))

    best_time = min(func_times)
    for i, func in enumerate(funcs):
        func_time = func_times[i]
        print('{:>8}: {:6.3f} sec, x {:.1f}'.format(func.__name__, func_time, func_time/best_time))
```

We can run it as following:

```python
funcs = [f_dp, f_binom, f_sci, f_sym]
test(1000, 200, funcs)
```

It runs the four discussed implementations on the same input: $n=1000$ and $m=200$.
The function validates that all the solutions are consistent, namely, produce the same result.
It prints the result and the time it takes to execute each implementation `number=100` times.
For each run, it also prints the relative factor computed based on the fastest case.

Let's test the functions on different $m$:

```python
test(1000, 100, funcs)
test(1000, 200, funcs)
test(1000, 300, funcs)
```

The output shows that the implementations using the closed form work much faster than the DP approach. The code benefits from libraries' functionality, which is very well optimized and written in C.

```
f(1000,100): 1055554268626824420007558704051435171794370568248714546059151891116400068098669888824348337982251284084060970452636576818160835738923780
    f_dp:  3.047 sec, x 28776.0
 f_binom:  0.009 sec, x 85.0
   f_sci:  0.002 sec, x 16.1
   f_sym:  0.000 sec, x 1.0

f(1000,200): 102959559398876709101987749103496956192748291997545179868396950180980024279315520736658787450735140705066645604315981112602395264722448393108564244943632128016608965665225093459140490108467690800
    f_dp:  5.715 sec, x 54069.2
 f_binom:  0.007 sec, x 68.6
   f_sci:  0.004 sec, x 37.5
   f_sym:  0.000 sec, x 1.0

f(1000,300): 216041785281835022932717773340894622391167159765932233268357621588611118890447413630526105759449201694312865174825389570985567896663730782942657696701009691262132150170383821261834224318912512938247596625130
    f_dp:  7.786 sec, x 71123.4
 f_binom:  0.005 sec, x 47.8
   f_sci:  0.007 sec, x 66.5
   f_sym:  0.000 sec, x 1.0
```

More refined comparizons for fast implementations:

```python
test(1000, 100, funcs[1:], number=10**5)
test(1000, 200, funcs[1:], number=10**5)
test(1000, 300, funcs[1:], number=10**5)
```

It looks like absolute winner is `sympy` library:

```
f(1000,100): 1055554268626824420007558704051435171794370568248714546059151891116400068098669888824348337982251284084060970452636576818160835738923780
 f_binom:  8.538 sec, x 72.9
   f_sci:  1.867 sec, x 15.9
   f_sym:  0.117 sec, x 1.0

f(1000,200): 102959559398876709101987749103496956192748291997545179868396950180980024279315520736658787450735140705066645604315981112602395264722448393108564244943632128016608965665225093459140490108467690800
 f_binom:  7.024 sec, x 62.7
   f_sci:  4.467 sec, x 39.9
   f_sym:  0.112 sec, x 1.0

f(1000,300): 216041785281835022932717773340894622391167159765932233268357621588611118890447413630526105759449201694312865174825389570985567896663730782942657696701009691262132150170383821261834224318912512938247596625130
 f_binom:  5.527 sec, x 48.7
   f_sci:  7.760 sec, x 68.3
   f_sym:  0.114 sec, x 1.0
```
