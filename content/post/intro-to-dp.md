+++
date = "2017-07-04T07:29:43Z"
highlight = true
math = true
tags = ["math", "python", "recursion", "dynamic programming", "memoization", "binomial", "coding interview", "programming", "combinatorics"]
title = "Introduction to Dynamic Programming and Memoization"

[header]
  caption = ""
  image = ""

+++

In the post, we discuss the basics of _Recursion_, _Dynamic Programming_ (DP), and _Memoization_.
As an example, we take a combinatorial problem, which has very short and clear description.
This allows us to focus on DP and memoization.
Note that the topics are very popular in coding interviews.
Hopefully, this article will help to somebody to prepare for such types of questions.

In the next posts,
we consider more advanced topics, like
[The Art of Generating Functions][gen-func-art] and
[Cracking Multivariate Recursive Equations Using Generating Functions][two-var-recursive-func].
The methods can be applied to the same combinatorial question.
Let's start from presenting the problem.

## The Problem

> Compute the number of ways to choose $m$ elements from $n$ elements such that selected elements in one combination are not adjacent.

For example, for $n=4$ and $m=2$, the answer is $3$, since from the $4$-element set: $\lbrace 1,2,3,4 \rbrace$,
there are three feasible $2$-element combinations: $\lbrace 1,4 \rbrace$, $\lbrace 2,4 \rbrace$, $\lbrace 1,3 \rbrace$.

Another example: for $n=5$ and $m=3$, there is only one $3$-element combination: $\lbrace 1,3,5 \rbrace$.

If you are asked such question during coding interview, interviewer is, probably, expecting to cover with you the following topics:

1. _Brute Force_ approach that generates and counts all the feasible combinations. It will work slow even for small input.
2. Recursive solution which counts the combination without generating them. It will work faster but still has exponential time ecomplexity.
3. Use DP or memoization techniques. In both cases, the time complexity becomes linear in $n$ and $m$.
4. Corner cases, recursion termination, call stack, testing.

Let's talk about most important topics: building a recursive solution and optimize it using DP or memoization.

## Recursive Relation

Let's define $F\_{n, m}$ to be the function that computes the answer for given $n$ and $m$.
Let's look at the $n, m$-task. We have two non-overlapping sub-tasks (or cases):

* Skip the $n$-th element, then $ F\_{n, m} = F\_{n-1, m} $.
* Pick the $n$-th element, then $ F\_{n, m} = F\_{n-2, m-1} $.

From the above, we can define the solution in the recursive form:

$ F\_{n, m} $ $ = F\_{n - 1, m} + F\_{n - 2, m - 1} $,

let's not forget to write down the corner cases:

$F\_{0, 0}$ $= F\_{1, 1}$ $= 1$.

Basically, we can combine the general and the corner cases into one expression:

$$ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=0] + [n=m=1] $$

It is very common to define $F\_{n,m} = 0$ for any negative $n$ or $m$.
The indicator $[P]$ gives $1$ if the predicate $P$ is true.
But what about special cases, like $n=1$ and $m=0$?
Actually, the relation covers this:

$F\_{1,0}$ $= F\_{0,0} + F\_{-1,-1} + [1=0=1] + [1=m=1]$ $=F\_{0,0} + 0 + 0 + 0$ $=[0=0=0]$ $=1$.

It is non-intuitive, but there is no need to add extra corner cases!

## Memoization

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

## Dynamic Programming (DP)

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

## Testing

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
test(n=6000, m=2000, funcs=[f_mem, f_dp])
```

Which prints output similar to this one:

```
f(6000,2000): 192496093
        f_mem:   6.5852 sec, x 1.28
         f_dp:   5.1507 sec, x 1.00
```


The function `test()` computes $F\_{6000, 2000}$ using two different ways.
It validates that all the solutions are consistent and produce the same result.
It also measures the time it takes to execute each method.
For each run, it also prints the relative factor computed based on the fastest solution.
The function's result is printed modulo $M=1000^3+7$.

Memoization and DP techniques have $\Theta(n \cdot m)$ time complexity.
Note that we ignore here a lot of interesting details, for example,
Python has built-in long arithmetics for integers, which is used here and definetely not cheap.

Testing the implementations on different parameters,
we may notice that there is no clear winner between the two algorithms.
Probably, most of the time is spent on the long arithmetic computation.


Can we do even better?
Definitely, we can!
The table (or cache) could be preprocessed.
Then computing the function $F\_{n,m}$ will require just one query from the table (or from the cache, for memoization solution).
As we mentioned before, such approach requires $\Theta(n\cdot m)$ space, which could be huge for $n=10000$.
Actually, there is a way, how we can remain in very low memory consuption and still get very fast solution.
We will discuss this in the next two posts.

 

[gen-func-art]: /post/gen-func-art/
[two-var-recursive-func]: /post/two-var-recursive-func/


