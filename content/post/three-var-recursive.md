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
$$
\begin{aligned}
a(m,n,k) &= 2a(m-1,n-1,k-1) + a(m-1,n-1,k) \\ 
&+ a(m-1,n,k-1) + a(m,n-1,k-1)
\end{aligned}
$$
where $m$, $n$, $k$ are nonnegative integers, with boundary conditions:

- $a(0,0,0) = a(1,0,0) = a(0,1,0) = a(0,0,1) = 1$
- $a(m,0,0) = a(0,m,0) = a(0,0,m) = 0$ for any $m > 1$
- $a(m,n,k)$ is symmetric in $m$,$n$,$k$

A subtlety: $a(0,1,1)$ is not defined by the recurrence alone, since it would require values like $a(-1,0,0)$. We take $a(m,n,k) = 0$ whenever any argument is negative.

## The Generating Function

Define

$$\Phi(x,y,z) = \sum_{m,n,k \geq 0} a(m,n,k) \cdot x^m y^n z^k$$

Using the initial values above, we can define $a(m,n,k)$ as follows:

$$
\begin{aligned}
a(m, n, k) &= 2a(m-1, n-1, k-1) + a(m-1, n-1, k) \\
&+ a(m-1, n, k-1) + a(m, n-1, k-1) \\
&+ [m=n=k=0] + [m=n=0 \wedge k=1] + [m=k=0 \wedge n=1] + [n=k=0 \wedge m=1]
\end{aligned}
$$

 believe there are still some initial conditions missing, since for example $a(0,1,1)$ is not well defined. Computing its value will result in negative arguments: $a(0,1,1) = 2a(-1, 0, 0) + a(-1, 0, 1) + a(-1, 1, 0) + a(0, 0, 0) = 2a(-1, 0, 0) + 2a(-1, 0, 1) + 1$. 

Adding the extra condition that $a(m,n,k)=0$ for any negative argument(s) solves the issue.

Let's move forward and substitute the definition of $a(m,n,k)$ into the generating function:

$$
\begin{aligned}
\Phi(x,y,z)
&=\sum_{m,n,k}a(m,n,k) \cdot x^m y^n z^k \\
&= 2\sum_{m,n,k}a(m,n,k) \cdot x^{m+1} y^{n+1} z^{k+1} + \sum_{m,n,k}a(m,n,k) \cdot x^{m+1} y^{n+1} z^k + \sum_{m,n,k}a(m,n,k) \cdot x^{m+1} y^n z^{k+1}+\sum_{m,n,k}a(m,n,k) \cdot x^m y^{n+1} z^{k+1} + 1 + x + y + z \\
&= 2\Phi(x,y,z) \cdot x y z + \Phi(x,y,z)\cdot x y + \Phi(x,y,z) \cdot x z + \Phi(x,y,z) \cdot y z + 1 + x + y + z \\
&= \Phi(x,y,z)\left(2 x y z + x y + x z + y z\right) + 1 + x + y + z
\end{aligned}
$$


Substituting the recurrence and collecting terms, we get

$$
\begin{aligned}
\Phi(x,y,z)
&= \sum_{m,n,k} a(m,n,k) \cdot x^m y^n z^k \\
&= 2\Phi \cdot xyz + \Phi \cdot xy + \Phi \cdot xz + \Phi \cdot yz + 1 + x + y + z
\end{aligned}
$$

where the boundary terms $1 + x + y + z$ come from $a(0,0,0)$, $a(1,0,0)$, $a(0,1,0)$, $a(0,0,1)$. Solving for $\Phi$:

$$\Phi(x,y,z) = \frac{1 + x + y + z}{1 - 2xyz - xy - xz - yz}$$

## From Generating Function to Closed Form

Using $\frac{1}{1-\rho} = \sum_{i \geq 0} \rho^i$ and the multinomial expansion

$$(x_1+x_2+x_3+x_4)^N = \sum_{k_1+k_2+k_3+k_4=N} \binom{N}{k_1,k_2,k_3,k_4} x_1^{k_1} x_2^{k_2} x_3^{k_3} x_4^{k_4}$$

with $\binom{N}{k_1,k_2,k_3,k_4} = \frac{N!}{k_1!\cdot k_2!\cdot k_3!\cdot k_4!}$, we expand the denominator. Let $\rho = 2xyz + xy + xz + yz$. Then

$$
\Phi = (1+x+y+z) \sum_{N \geq 0} \rho^N
$$


$$
\begin{aligned}
&= \frac{1 + x + y + z}{1-2 x y z - x y - x z - y z} \\
&= (1 + x + y + z) \sum_{N}(2 x y z + x y + x z + y z)^N \\
&= (1 + x + y + z) \sum_{k_1+k_2+k_3+k_4=N} \binom{N} {k_1,k_2,k_3,k_4} (2 x y z)^{k_1} \cdot (x y)^{k_2} \cdot (x z)^{k_3} \cdot (y z)^{k_4}  \\
&= (1 + x + y + z) \sum_{k_1+k_2+k_3+k_4=N} \binom{N} {k_1,k_2,k_3,k_4} 2^{k_1} x^{k_1+k_2+k_3} y^{k_1+k_2+k_4} z^{k_1+k_3+k_4} \\
&= (1 + x + y + z) \sum_{k_1+k_2+k_3\leq N} \binom{N} {k_1,k_2,k_3,N-k_1-k_2-k_3} 2^{k_1} x^{k_1+k_2+k_3} y^{N-k_3} z^{N-k_2}
\end{aligned}
$$


Writing $\rho^N$ with terms $(2xyz)^{k_1}(xy)^{k_2}(xz)^{k_3}(yz)^{k_4}$ where $k_1+k_2+k_3+k_4 = N$, and extracting the coefficient of $x^m y^n z^k$, yields the closed form:

$$
\begin{aligned}
a(m,n,k)
&= \sum_{\max(m,n,k) \leq N \leq \frac{m+n+k}{2}}
   \binom{N}{m+n+k-2N, N-m, N-n, N-k} 2^{m+n+k-2N} \\
&\quad + \text{three similar sums from the } 1, x, y, z \text{ terms}
\end{aligned}
$$

The full expression has four sums corresponding to the four terms in the numerator $1+x+y+z$. The exact formulas are:

$$
\begin{aligned}
a(m,n,k)
&= \sum_{N=\max(m,n,k)}^{\lfloor (m+n+k)/2 \rfloor}
   \binom{N}{m+n+k-2N, N-m, N-n, N-k} 2^{m+n+k-2N} \\
&\quad + \sum_{N=\max(m-1,n,k)}^{\lfloor (m+n+k-1)/2 \rfloor}
   \binom{N}{m+n+k-2N-1, N-m+1, N-n, N-k} 2^{m+n+k-2N-1} \\
&\quad + \sum_{N=\max(m,n-1,k)}^{\lfloor (m+n+k-1)/2 \rfloor}
   \binom{N}{m+n+k-2N-1, N-m, N-n+1, N-k} 2^{m+n+k-2N-1} \\
&\quad + \sum_{N=\max(m,n,k-1)}^{\lfloor (m+n+k-1)/2 \rfloor}
   \binom{N}{m+n+k-2N-1, N-m, N-n, N-k+1} 2^{m+n+k-2N-1}
\end{aligned}
$$

There may be room to simplify this further; the symmetry in $m,n,k$ could help.

## Complexity

- **Recursion with memoization (DP):** $\Theta(m \cdot n \cdot k)$ time and space.
- **Closed form:** Precompute factorials, then loop over $N$; time and space $\Theta(m+n+k)$, ignoring the cost of arithmetic on large integers.

## Implementation

Both the recursive and closed-form versions in Python:

```python
import functools
import math

@functools.lru_cache(maxsize=None)
def a_rec(m: int, n: int, k: int) -> int:
    if min(m, n, k) < 0:
        return 0
    if m + n + k == 0 or m + n + k == 1:
        return 1
    if m + n == 0 or m + k == 0 or n + k == 0:
        return 0
    return (
        2 * a_rec(m - 1, n - 1, k - 1)
        + a_rec(m - 1, n - 1, k)
        + a_rec(m - 1, n, k - 1)
        + a_rec(m, n - 1, k - 1)
    )

@functools.lru_cache(maxsize=None)
def _binom4(N: int, a: int, b: int, c: int) -> int:
    r = N - a - b - c
    vals = sorted([a, b, c, r])
    assert vals[0] >= 0
    return math.factorial(N) // (
        math.factorial(vals[0]) * math.factorial(vals[1])
        * math.factorial(vals[2]) * math.factorial(vals[3])
    )

def a_closed(m: int, n: int, k: int) -> int:
    if min(m, n, k) < 0:
        return 0
    s = 0
    for N in range(max(m, n, k), (m + n + k) // 2 + 1):
        s += _binom4(N, N - m, N - n, N - k) * 2 ** (m + n + k - 2 * N)
    for N in range(max(m - 1, n, k), (m + n + k - 1) // 2 + 1):
        s += _binom4(N, N - m + 1, N - n, N - k) * 2 ** (m + n + k - 2 * N - 1)
    for N in range(max(m, n - 1, k), (m + n + k - 1) // 2 + 1):
        s += _binom4(N, N - m, N - n + 1, N - k) * 2 ** (m + n + k - 2 * N - 1)
    for N in range(max(m, n, k - 1), (m + n + k - 1) // 2 + 1):
        s += _binom4(N, N - m, N - n, N - k + 1) * 2 ** (m + n + k - 2 * N - 1)
    return s

# Sanity check
r, r1 = a_rec(100, 200, 210), a_closed(100, 200, 210)
print(f"Recursive: {r}, Closed: {r1}, Match: {r == r1}")
```

We did not exploit the symmetry $a(m,n,k) = a(\sigma(m,n,k))$ for permutations $\sigma$; it could speed up computation but does not obviously simplify the closed expression.

---

*Originally answered on [Math Stack Exchange][math-se].*

[two-var-recursive-func]: /post/two-var-recursive-func/
[math-se]: https://math.stackexchange.com/questions/1093271/how-to-solve-this-multivariable-recursion/2730331#2730331
