+++
date = "2018-04-10T03:14:00Z"
math = true
highlight = true
tags = ["math", "python", "generating function", "recursion", "combinatorics", "dynamic programming", "multivariate function"]
title = "Solving a Three-Variable Recursion via Generating Functions"
draft = false

[header]
image = ""
caption = ""
+++

This post extends the generating-function technique from the [two-variable recursion][two-var-recursive-func] to a three-variable case. I originally wrote this as an answer to a [Math Stack Exchange question][math-se]; here it is adapted for the blog with clearer exposition and code.

## The Problem

We want to solve the recurrence

$a_{n,m,k} = 2a_{n-1,m-1,k-1} + a_{n-1,m-1,k}$
$ + a_{n,m-1,k-1} + a_{n-1,m,k-1}$

where $m$, $n$, $k$ are nonnegative integers, with boundary conditions:

- $a_{0,0,0} = a_{0,1,0} = a_{1,0,0} = a_{0,0,1} = 1$
- $a_{n,0,0} = a_{0,n,0} = a_{0,0,n} = 0$ for any $n > 1$
- $a_{n,m,k}$ is symmetric in $n$, $m$, $k$

A subtlety: $a_{1,0,1}$ is not defined by the recurrence alone, since it would require values like $a_{0,-1,0}$. We take $a_{n,m,k} = 0$ whenever any subscript is negative.

## The Generating Function

Define

$$\Phi(x,y,z) = \sum_{n,m,k \geq 0} a_{n,m,k} \cdot x^n y^m z^k$$

Using the initial values above, we can write the recurrence including boundary terms as follows:

$a_{n,m,k} = 2a_{n-1,m-1,k-1} + a_{n-1,m-1,k}$
$ + a_{n,m-1,k-1} + a_{n-1,m,k-1}$
$ + [n=m=k=0] + [n=m=0 \wedge k=1]$
$ + [n=k=0 \wedge m=1] + [m=k=0 \wedge n=1]$

Substituting the recurrence into the generating function and collecting terms:

$\Phi(x,y,z) = \sum_{n,m,k} a_{n,m,k} \cdot x^n y^m z^k $
$ = 2\sum_{n,m,k} a_{n,m,k} \cdot x^{n+1} y^{m+1} z^{k+1} $
$   + \sum_{n,m,k} a_{n,m,k} \cdot x^{n+1} y^{m+1} z^k $
$   + \sum_{n,m,k} a_{n,m,k} \cdot x^{n+1} y^m z^{k+1} $
$   + \sum_{n,m,k} a_{n,m,k} \cdot x^n y^{m+1} z^{k+1} $
$   + 1 + x + y + z $
$ = 2 \Phi \cdot x y z + \Phi \cdot x y + \Phi \cdot x z + \Phi \cdot y z + 1 + x + y + z$
$ = \Phi \cdot ( 2 x y z + x y + x z + y z ) + 1 + x + y + z$

where the boundary terms $1 + x + y + z$ come from $a_{0,0,0}$, $a_{1,0,0}$, $a_{0,1,0}$, $a_{0,0,1}$. 

Solving for $\Phi$:

$$\Phi(x,y,z) = \frac{1 + x + y + z}{1 - 2xyz - xy - xz - yz}$$

## From Generating Function to Closed Form

Using 

$$\frac{1}{1-\rho} = \sum_{i \geq 0} \rho^i$$ 

and the multinomial expansion

$$(x_1+x_2+x_3+x_4)^N = \sum_{r_1+r_2+r_3+r_4=N} \binom{N}{r_1,r_2,r_3,r_4} x_1^{r_1} x_2^{r_2} x_3^{r_3} x_4^{r_4}$$

with

$$\binom{N}{r_1,r_2,r_3,r_4} = \frac{N!}{r_1!\cdot r_2!\cdot r_3!\cdot r_4!}$$

we expand the denominator of $\Phi$. Let $\rho = 2xyz + xy + xz + yz$. Then

$$\Phi = \frac{1+x+y+z}{1-\rho} = (1+x+y+z) \sum_{N \geq 0} \rho^N$$

Expanding $\rho^N$ with the multinomial theorem (and writing $r_4 = N - r_1 - r_2 - r_3$):

$\sum_{N \geq 0} \rho^N = \sum_{N}(2 x y z + x y + x z + y z)^N $
$ = \sum_{r_1+r_2+r_3+r_4=N} \binom{N} {r_1,r_2,r_3,r_4} (2 x y z)^{r_1} \cdot (x y)^{r_2} \cdot (x z)^{r_3} \cdot (y z)^{r_4}$
$ = \sum_{r_1+r_2+r_3+r_4=N} \binom{N} {r_1,r_2,r_3,r_4} 2^{r_1} x^{r_1+r_2+r_3} y^{r_1+r_2+r_4} z^{r_1+r_3+r_4}$
$ = \sum_{r_1+r_2+r_3 \leq N} \binom{N} {r_1,r_2,r_3, N-r_1-r_2-r_3} 2^{r_1} x^{r_1+r_2+r_3} y^{N-r_3} z^{N-r_2}$

So we have

$$ \Phi = (1 + x + y + z) \sum_{r_1+r_2+r_3 \leq N} \binom{N} {r_1,r_2,r_3, N-r_1-r_2-r_3} 2^{r_1} x^{r_1+r_2+r_3} y^{N-r_3} z^{N-r_2}$$

Extracting the coefficient of $x^n y^m z^k$ gives the closed form. The full expression has four sums (from the numerator $1+x+y+z$):

$$a_{n,m,k} = \sum_{N=\max(n,m,k)}^{ (n+m+k)/2 } \binom{N}{n+m+k-2N, N-n, N-m, N-k} 2^{n+m+k-2N}$$
$$ + \sum_{N=\max(n,m-1,k)}^{ (n+m+k-1)/2 } \binom{N}{n+m+k-2N-1, N-n, N-m+1, N-k} 2^{n+m+k-2N-1}$$
$$ + \sum_{N=\max(n-1,m,k)}^{ (n+m+k-1)/2 } \binom{N}{n+m+k-2N-1, N-n+1, N-m, N-k} 2^{n+m+k-2N-1}$$
$$ + \sum_{N=\max(n,m,k-1)}^{ (n+m+k-1)/2 } \binom{N}{n+m+k-2N-1, N-n, N-m, N-k+1} 2^{n+m+k-2N-1}$$

There may be room to simplify this further; the symmetry in $n,m,k$ could help.

## Complexity

**Recursion with memoization (DP):** 
- time and space: $\Theta(n \cdot m \cdot k)$.

**Closed form:**
- Precompute factorials, then loop over $N$; 
- time and space: $\Theta(n+m+k)$, 
- we ignore the cost of arithmetic on large integers.

## Implementation

Both the recursive and closed-form versions in Python:

```python
import functools
import math
```

```python
@functools.lru_cache(maxsize=None)
def a_rec(n: int, m: int, k: int) -> int:
    if not (n <= m <= k):
        n,m,k = sorted([n,m,k])
        return a_rec(n,m,k)

    if n < 0:
        return 0
    if n + m + k <= 1:
        return 1
    if n + m == 0 or n + k == 0 or m + k == 0:
        return 0
    if n == 0:
        return int(m + 1 >= k)
    
    return (
        2 * a_rec(n - 1, m - 1, k - 1)
        + a_rec(n - 1, m - 1, k)
        + a_rec(n - 1, m, k - 1)
        + a_rec(n, m - 1, k - 1)
    )
```

```python
@functools.lru_cache(maxsize=None)
def binom4(N: int, r1: int, r2: int, r3: int) -> int:
    r4 = N - r1 - r2 - r3

    return math.factorial(N) // (
        math.factorial(r1) * math.factorial(r2)
        * math.factorial(r3) * math.factorial(r4)
    )


def a_closed(n: int, m: int, k: int) -> int:
    if min(n, m, k) < 0:
        return 0
    s = 0
    for N in range(max(n, m, k), (n + m + k) // 2 + 1):
        s += binom4(N, N - n, N - m, N - k) * 2 ** (n + m + k - 2 * N)
    for N in range(max(n, m - 1, k), (n + m + k - 1) // 2 + 1):
        s += binom4(N, N - n, N - m + 1, N - k) * 2 ** (n + m + k - 2 * N - 1)
    for N in range(max(n - 1, m, k), (n + m + k - 1) // 2 + 1):
        s += binom4(N, N - n + 1, N - m, N - k) * 2 ** (n + m + k - 2 * N - 1)
    for N in range(max(n, m, k - 1), (n + m + k - 1) // 2 + 1):
        s += binom4(N, N - n, N - m, N - k + 1) * 2 ** (n + m + k - 2 * N - 1)
    return s
```

```python
import timeit
setup_code = "from __main__ import a_rec, a_closed"

execution_time = timeit.timeit('a_rec(200, 300, 400)', setup=setup_code, number=1)
print(f"Execution time for recursive:   {execution_time} seconds")

execution_time = timeit.timeit('a_closed(200, 300, 400)', setup=setup_code, number=1)
print(f"Execution time for closed form: {execution_time} seconds")
```

```text
Execution time for recursive:   8.308788761030883 seconds
Execution time for closed form: 0.003819321980699897 seconds
```

```python
# Sanity check
res0 = a_rec(100, 200, 210)
res1 = a_closed(100, 200, 210)

print(f"Recursive: {res0}")
print(f"Closed:    {res1}")
print(f"Match: {res0 == res1}")
```

We did not exploit the symmetry $a_{n,m,k} = a_{\sigma(n,m,k)}$ for permutations $\sigma$; it could speed up computation but does not obviously simplify the closed expression.

---

*Originally answered on [Math Stack Exchange][math-se].*

[two-var-recursive-func]: /post/two-var-recursive-func/
[math-se]: https://math.stackexchange.com/questions/1093271/how-to-solve-this-multivariable-recursion/2730331#2730331
