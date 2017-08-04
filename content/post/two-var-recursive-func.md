+++
date = "2017-07-04T07:29:43Z"
highlight = true
math = true
tags = ["math","python", "generating functions", "recursion", "recursive functions", "dynamic programming", "memoization", "binomial", "coding interview", "combinarotics"]
title = "Cracking Multivariate Recursive Equations Using Generating Functions"

[header]
  caption = ""
  image = ""

+++


In this post, we consider a combinatorial problem.
It has a nice recursive solution, which could be implemented efficiently using Dynamic Programming (DP) technique.
Then using generating function, we find the closed formula, expressed with the binomial coefficients.
Finaly, we show that the closed formula gives us much faster way to compute the result than the DP solution.

Generating functions are usually applied to the case of single variable recursive equations.
But actually, the technique may be extended to multivariate recursive equations, or even to a system of recursive equations.

Sometimes the generating functions may give a fantastic result, and here we discuss one of such cases.

## The Problem

Let's consider the following problem.

**Problem:** Compute the number of ways to choose $m$ elements from $n$ elements such that selected elements in one combination are not adjacent.

For example, for $m=4$ and $m=3$, the answer is $3$, since from the $4$-element set: $\lbrace 1,2,3,4 \rbrace$,
there are three feasible $2$-element combinations: $\lbrace 1,4 \rbrace$, $\lbrace 2,4 \rbrace$, $\lbrace 1,3 \rbrace$.

Another example, for $n=5$ and $m=3$, there is only one $3$-element combination: $\lbrace 1,3,5 \rbrace$.

This problem, of similar to this one, could be asked on coding interviews.
Typically, the interviewer may expect the candidate start from suggesting a _Brute Force_ solution that generates all the combination.
Then the candidate is expected to suggest and implement a simple recursive function computing the answer.
Then the interviewer is willing to hear that the recursive solution is very slow (has exponential time complexity) and
has linear space complexity (due to stack calls).
Finally, the candidate is expected to tell that the running time could be improved by using either DP or Memoization techniques.
This gives much faster solution.

Similar questions might be suggested as trivial problems on coding contests.
In that case, extra optimizaitons might be required.
Basically, after we discuss the typical DP and Memoization implementations, we will show that using generating function,
we can build very fast and simple solution.

Let's define $F\_{n, m}$ is the function that gives as the answer for $n$ and $m$.
Let's look at the $n, m$ case. We have two non overlapping sub cases:

* Skip the $n$-th element, then $ F\_{n, m} = F\_{n-1, m} $.
* Pick the $n$-th element, then $ F\_{n, m} = F\_{n-2, m-1} $.

From the above, we can define the solution in the form of simple recursion:
$ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} $,
let's not forget the corner cases:
$F\_{0, 0} = F\_{1, 1} = 1$.

Basically, we can combine the general and the corner cases into one expression:

$$ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=0] + [n=m=1] $$

We assume that for any $n < 0$ or $m < 0$, $F\_{n,m} = 0$.
The indicator $[P]$ gives $1$ if the predicate $P$ is true.

The function $F\_{n,m}$ has a straighforward recursive implementation in any programming language.
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

    if n + 1 < 2*m:
        return 0

    if n == m == 0:
        return 1

    if n == m == 1:
        return 1

    return f_mem(n - 1, m) + f_mem(n - 2, m - 1)
```

Nice [@functools.lru_cache](https://docs.python.org/3/library/functools.html#functools.lru_cache)
notation creates a wrapper on the `f_mem` function and internally caches the results of all calls.
Such caching (or memoization) significantly improves the speed of the recursion,
and basically reduces the number of calls to something like $O(n \cdot m)$.
The recursion still may fail on the stack overflow even on relatively small values of $n$.
That is why we increase the stack size for the Python interpreter by calling
[sys.setrecursionlimit()-method](https://docs.python.org/3/library/sys.html#sys.setrecursionlimit).

The next iterative implementation uses DP.
Basically, it fills out the $n \times m$ table starting from the low values of $n$ and $m$.

```python
def f_dp(n, m):
    assert n >= 0 and m >= 0

    if n + 1 < 2*m:
        return 0

    table = [[0] * (m + 1) for _ in range(n + 1)]

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
More accurately, time complexity might be bounded by $\Theta(n\cdot \min(n,m))$.
It is not hard to notice that the space consumption could be reduced to $O(\min(n, m))$.

## The Generating Functions

The notion of generating functions and its application to solving recursive equations are very well-known.
For reader who did not have a chance to learn this,
I recommend to take a look at very good book [Concrete Mathematics: A Foundation for Computer Science](https://en.wikipedia.org/wiki/Concrete_Mathematics).
It has great explanation of generics functions.

For those who are not patient who will briefly introduce the generating functions and consider their application to Fibonacci numbers.
Readers who are familiar with one-variable case,
may jump directly to the next section where we discuss how to apply the generating functions to two variable recursions, like $F\_{n,m}$.

Let's consider the generating function's application to Fibonacci numbers.
The Fibonacci numbers could defined by the recursive expression:

$$f\_n = f\_{n-1} + f\_{n-2} + [n=0]$$

We assume that for any $n < 0$, $f\_n = 0$.
The indicator $[n=0]$ equals to $1$ only if $n=0$.
This definition produces the following sequence of the Fibonacci numbers: $1$, $1$, $2$, $3$, $5$, $8$, $13$, $\dots$.
Note that sometime, the Fibonacci sequence is defined to start from 0, but it is really not important for the perpose of our discussion.

The generating function $\Phi(x)$ on a floating point variable $x$ is defined as the infinite sum:

$$\Phi(x) = \sum\_{n\geq 0} f\_n x^n$$

Usually, it is hard to develop an intuition why it is defined that way and why it could be useful.
So let's just focuse on what we can do with this,
and let's start from substituting the defintion of Fibonacci recursion into the formular of generating function:
$ \Phi(x) $
$ = \sum\_{n\geq 0} f\_n x^n $
$ = \sum\_n f\_n x^n $
$ = \sum\_n (f\_{n-1} + f\_{n-2} + [n=0]) x^n $
$ = \sum\_n f\_{n-1} x^n + \sum\_n f\_{n-2} x^n + \sum\_n [n=0] x^n $
$ = x \sum\_n f\_{n-1} x^{n-1} + x^2 \sum\_n f\_{n-2} x^{n-2} + 1 x^0 $
$ = x \sum\_n f\_n x^n + x^2 \sum\_n f\_n x^n + 1 $
$ = x \Phi(x) + x^2 \Phi(x) + 1 $

And we can obtain the generating function for $f\_n$:

$$\Phi(x) = \frac{1}{1 - x - x^2}$$

Nice expression, but what can we do with this "magic" formula?
We can hack it in different ways, and depending on our next step, we can get different closed forms for $f\_n$.

One of the standard ways is to split the fraction into two simple ones of the form: $\frac{A}{x+B}$.
Note that the roots of the quadratic equation: $1 - x - x^2 = 0$ are $x\_0=-\phi$ and $x\_1=\phi^{-1}$,
where $\phi$ is the _Goldan Ratio_:
$ \phi = \frac{1 + \sqrt{5}}{2} = 1.618\dots $.

A quick test:
$ -(x-x\_0) (x-x\_1) $
$ = -(x+\phi) \left(x-\phi^{-1}\right) $
$ =-x^2 - x\cdot\left(\phi - \phi^{-1}\right) + 1 $
$ = -x^2 - x + 1 $.

This results in representing $\Phi(x)$ in the following way:
$\Phi(x)$
$ = \frac{1}{1-x-x^2}$
$ = -\frac{1}{(x+\phi) \left(x-\phi^{-1}\right)}$
$ = \frac{A}{x+\phi} - \frac{A}{x-\phi^{-1}}$
$ = A\cdot\phi^{-1} \frac{1}{1+x\cdot\phi^{-1}} + A\cdot\phi \frac{1}{1 - x\cdot\phi}$,
where $A=\frac{1}{\phi+\phi^{-1}}$.

The next trick is to apply the infinite series $\frac{1}{1-z} = \sum\_n z^n$ to each expression and to simplify the sums:
$\Phi(x) $
$ = A\cdot\phi^{-1} \sum\_n \left(-x\cdot \phi^{-1}\right)^n + A\cdot\phi \sum\_n (x\cdot\phi)^n $
$ = A\cdot\phi^{-1} \sum\_n (-\phi)^{-n} x^n + A\cdot\phi \sum\_n \phi^n x^n $
$ = \sum\_n A\cdot\left(\phi^{-1} (-\phi)^{-n} + \phi\cdot \phi^n \right)  x^n $
$ = \sum\_n A\cdot\left(\phi^{n+1} - (-\phi)^{-n-1}\right) x^n $
$ = \sum\_n \frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}} x^n $.

The term before $x^n$ is nothing but $f\_n$, which means that

$$f\_n=\frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}}$$

Another way to crack the generating function is to apply the infinite series $\frac{1}{1-z} $ $= \sum\_n z^n$ directly to the generating function:
$ \Phi(x) $
$ = \frac{1}{1 - x - x^2} $
$ = \frac{1}{1 - (x+x^2)}$
$ = \sum\_n (x+x^2)^n $
$ = \sum\_n (1+x)^n x^n $
$ = \sum\_{0\leq k \leq n} {n \choose k} x^{n+k} $.

Introducing the new variable $t=n+k$, we obtain
$ = \sum\_{t/2 \leq n \leq t} {n \choose t-n} x^t $
$ = \sum\_t \sum\_{n=t/2}^t {n \choose t-n} x^t $

So we receive another closed form for the Fibonacci numbers:

$$f\_t = \sum\_{n=t/2}^t {n \choose t-n}$$

Believe or not, but the forms define the same Fibonacci sequence.

$$ f\_n=\frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}} $$

$$ f\_n = \sum\_{i=n/2}^t {i \choose n-i} $$

An interested reader can try to prove by induction that correctness of the expressions above.
We just notice how powerful the generating functions might be and we will use them for two variable recursive expressions in the next section.

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

In order to measure the time and to check the correctness, let's write the following helper function:

```python
import timeit

M = 1000**3 + 7

def test(n, m, funcs, number=1, module=__name__):
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
        print('{:>13}: {:8.4f} sec, x {:.2f}'.format(func.__name__, func_time, func_time/best_time))
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

The function `test()` computes $F\_{6000, 2000}$ using five different ways.
It validates that all the solutions are consistent and produce the same result.
It also measures the time it takes to execute each method.
For each run, it also prints the relative factor computed based on the fastest solution.
The function's result is printed modulo $M=1000^3+7$.

This is not a surprise that the first two methods (based on memoization and DP) are much slower than the other three, which are based on the binomial coefficients.
Memoization and DP techniques have $\Theta(n \cdot m)$ time complexity.
The `f_binom`-method uses $factorial$ as a subroutine, which is linear in $n$ and implemented in a naive way.
Note that we ignore here a lot of interesting details, for example, Python has built-in long arithmetics for integers, which is used here and definetely not cheap.
Also `factorial`-function may use memoization for storing its value.
Other methods based on `sympy` and `scipy.special`, may (and actually do) use memoization as well.
The impact of C-impementation is also out of the scope of the current discassion, as well as some other "hard-core" optimizations that one can apply.

Testing the implementations on different parameters, we may notice that there is no clear winner between the algorthms.
Probably, the most of the time is spent on the long arithmetic computation.

## Modular Arithmetics

In questions where it is required to count some objects, not rearly the answer might be very big even on very small input.
In such case, typically it is asked to print the answer modulo some big prime integer, let's say, $M=1000^3+7$.
Since Python has built-in long arithmetics, we can apply modulo on the final result, but executing the entire agorithm with long arithmetics while knowing that only small part of it is really important is very costly, and of course, not that efficient.

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

