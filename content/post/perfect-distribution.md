+++
date = "2017-08-01T07:29:43Z"
math = true
highlight = true
tags = ["math","python", "recursion", "recursive functions", "gcd", "uniform distribution"]
title = "Perfect Distribution Based on GCD"
draft = false

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""

+++

## Perfect Distribution Based on GCD

We discuss an algorithm that distributes $a$ ones among $n$ positions so that the gaps between consecutive ones differ by at most one—a **perfect distribution**. The examples below are software engineering use cases for this algorithm.

### Use Case 0: Text spacing

Given a string of characters $s$ and a number $n$ greater than the length of $s$,
extend the string $s$ to length $n$ by inserting spaces between the words.
The key requirement is that the distances between two consecutive words be uniform.

### Use Case 1: Stress testing

Assume we want to stress-test a server.
Our test contains positive and negative requests.
Suppose we want to send $1000$ requests, $300$ of which are negative.
Our test tool might send requests simultaneously or sequentially,
but we require that the negative requests arrive at the server with uniform distribution.
I.e., we want to avoid sending 300 negative followed by 700 positive requests.
Using the algorithm, we simply distribute the 300 negative requests by calling ${\cal PD}(300, 1000)$:
cell $i$ is $1$ if and only if request $i$ is negative.
The test tool then uses the array to send requests (possibly sequentially).

### Use Case 2: Rate limiting / profiling

Assume the test tool can run performance tests—creating stress and measuring throughput (requests per second).
We want to extend the test tool with a profiling feature: specify the number of requests sent per second.
E.g., a performance test shows the server handles $1000$ requests per second.
We want the test tool to send requests at a specified rate, say $500$, $200$, $20$, or even $0.2$ requests per second (one request every 5 seconds).
Such a profiling tool allows exploring the server state under various load conditions: CPU, memory, threads depending on the number of requests received.

The test tool may divide 1 second into $100$ parts, each $20$ milliseconds.
Suppose we specify $230$ requests per second: then during every part, $2.3$ requests must be sent on average.
In other words, the tool must send 2 requests in most parts and 3 in some parts.
We need to choose which $30$ parts get the extra request; ${\cal PD}(30, 100)$ yields that array (a $1$ in cell $i$ means part $i$ gets 3 requests).

We present the general solution ${\cal PD}$ to the problem—for any specified input in the above examples, ${\cal PD}$ produces the exact solution.
Any probabilistic or approximation solutions do not achieve the required exactness and are usually more complicated.
The algorithm ${\cal PD}$ is simple, has a clear correctness proof, and admits $O(n)$ time and space complexity.
Moreover, it has a direct relation to Euclid's algorithm for computing the Greatest Common Divisor, denoted here by ${\cal GCD}$.

### Notation

*POST* denotes a condition that the procedure guarantees will hold after the call.

### Euclid's Algorithm ${\cal GCD}$

* POST: $res = {\cal GCD}(a, n)$.

```python
def gcd(a, n):
    assert 0 <= a <= n

    res = n
    if a > 0:
        res = gcd(n % a, a)

    return res
```

### Deterministic algorithm of perfect distribution ${\cal PD}$

The recurrence mirrors Euclid's algorithm: ${\cal PD}(a, n)$ calls ${\cal PD}(n \bmod a, a)$, so the recursion depth is $O(\log \min(a, n))$ and total time is $O(n)$.

* POST: size of $res$ is $n$.
* POST: $\forall i,\, res[i] \in \{0, 1\}$.
* POST: $\sum_{i=0}^{n-1} res[i] = a$ (exactly $a$ ones).
* POST: $res$ is perfectly distributed (gaps between consecutive ones differ by at most 1).

```python
def pd(a, n):
    assert 0 <= a <= n

    res = [0] * n
    if a > 0:
        steps = pd(n % a, a)
        step = n // a
        ofs = 0
        for i in range(a):
            res[ofs] = 1
            ofs += step + steps[i]

    return res
```

### Example. ${\cal PD}(2, 10)$: $a = 2$, $n = 10$, so $n - a = 8$ zeros
We want to mix $2$ ones and $8$ zeros.
The recursive call ${\cal PD}(0, 2)$ returns $[0, 0] = 0^2$, which is assigned to $arr$.
Then the array $res$ is filled as follows: $[(1, 0, 0, 0, 0), (1, 0, 0, 0, 0)] = (1, 0^4)^2$, which is returned by ${\cal PD}(2, 10)$.

### Example. ${\cal PD}(8, 10)$: $a = 8$, $n = 10$, so $n - a = 2$ zeros
We want to mix $8$ ones and $2$ zeros.
The recursive call ${\cal PD}(2, 8)$ returns $[1, 0, 0, 0, 1, 0, 0, 0] = (1, 0^3)^2$, which is assigned to $arr$.
Then the array $res$ is filled as follows: $[(1, 0, 1, 1, 1), (1, 0, 1, 1, 1)] = (1, 0, 1^3)^2$, which is returned by ${\cal PD}(8, 10)$.

### Examples.

* ${\cal PD}(1, 10) : [1, 0, \ldots, 0] = 1, 0^9$
* ${\cal PD}(2, 10) : [1, 0, 0, 0, 0, 1, 0, 0, 0, 0] = 1, 0^4 , 1, 0^4$
* ${\cal PD}(3, 10) : [1, 0, 0, 1, 0, 0, 1, 0, 0, 0] = 1, 0^2 , 1, 0^2 , 1, 0^3$
* ${\cal PD}(4, 10) : [1, 0, 0, 1, 0, 1, 0, 0, 1, 0] = 1, 0^2 , 1, 0, 1, 0^2 , 1, 0$ $= (1, 0^2 , 1, 0)^2$
* ${\cal PD}(5, 10) : [1, 0, 1, 0, 1, 0, 1, 0, 1, 0] = (1, 0)^5$
* ${\cal PD}(6, 10) : [1, 0, 1, 1, 0, 1, 0, 1, 1, 0] = 1, 0, 1^2 , 0, 1, 0, 1^2 , 0$ $ = (1, 0, 1^2 , 0)^2$

* ${\cal PD}(0, 2) : [0, 0] = 0^2$
* ${\cal PD}(2, 6) : [1, 0, 0, 1, 0, 0] = 1, 0^2 , 1, 0^2 = (1, 0^2)^2$
* ${\cal PD}(6, 8) : [1, 0, 1, 1, 1, 0, 1, 1] = 1, 0, 1^3, 0, 1^2 = (1, 0, 1^2)^2$
* ${\cal PD}(8, 14) : [1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0] = (1, 0, 1, 1, 0, 1, 0)^2$ $ = ((1, 0, 1)^2 , 0)^2$

### Alternative formulation: subtraction-based (slow) versions

The same recurrence can be expressed using subtraction instead of modulo, which makes the symmetry with Euclid's algorithm more explicit. These versions have the same recursion structure but do more work per step.

**Slow GCD (subtraction-based):**

```python
def gcd(a, n):
    assert 0 <= a <= n

    res = n
    if a > 0:
        b = n - a
        if b > a:
            res = gcd(a, b)
        else:
            res = gcd(b, a)

    return res
```

**Slow PD (subtraction-based):**

```python
def pd(a, n):
    assert 0 <= a <= n

    res = [0] * n
    if a > 0:
        ofs = 0
        b = n - a
        if b > a:
            arr = pd(a, b)
            for i in range(b):
                res[ofs] = arr[i]
                ofs += 1 + arr[i]

        else:
            arr = pd(b, a)
            for i in range(a):
                res[ofs] = 1 - arr[i]
                ofs += 2 - arr[i]

    return res
```

