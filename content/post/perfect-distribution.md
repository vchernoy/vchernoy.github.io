+++
date = "2017-08-01T07:29:43Z"
math = true
highlight = true
tags = ["math","python", "recursion", "recursive functions", "gcd", "uniform distribution"]
title = "Perfect Distribution Based On GCD"
draft = true

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""

+++

## Perfect Distribution Based On GCD

We discuss an algorithm that has property of uniform distribution.
The examples below are Software Engineering used cases of this algorithm.

### Software Engineering Used Case 0

Given a string of characters $s$, and the given number $n$ that is greater then the length of $s$.
Extend the string $s$ to the length $n$ by inserting spaces between the words of $s$.
They key requirement is that the distances between two consecutive words will be uniform.

### Software Engineering Used Case 1

Assume we want to test a server for stress.
Our test contains positive and negative requests.
Assume we want to sends $1000$ request that $300$ of them are negative.
Our test tool might send requests simultaneously or sequentially,
but what we required is that the negative requests will arrive the server with uniform distribution.
I.e., we want to prevent the following: sending 300 negative following by $700$ positive requests.
Using the algorithm, we simple distribute $300$ request by calling ${\cal PD}(300, 1000)$,
containing $1$ in cell $i$ corresponds to the request $i$ to be negative.
Then the test tool will use the array to sends request that now might be sent even sequentially.

### Software Engineering Used Case 2

Assume the test tool can also run performance tests, that is, creates stress on the server and measure performance -- the number of requests per second processed by the server.
We want to extend the test tool by proﬁling feature: to specify the number of requests being sent per second.
E.g. a performance test shows that the server performance is $1000$ requests per second.
We want test tool will send requests according to speciﬁed speed, say $500$, or $200$, or $20$, or even $0.2$ requests per second (i.e., one request per $5$ second).
Such proﬁling tool allows to explore the state of the server under various stress conditions: utility of CPU, Memory, Threads in depending on various number of requests received.
The test tool may divide $1$ seconds, say, on $100$ parts, i.e. every part is $20$ milliseconds.
Assume $230$ is speciﬁed, then during every part, $2.3$ request must be send.
In other words, the test tool must send 2 requests in most of parts and $3$ request in some parts.
Calling ${\cal PD}(30, 100)$ yields an array deﬁning the parts in which 3 requests will be sent.

We present the general solution ${\cal PD}$ to the problem -- for any speciﬁed input in above examples, ${\cal PD}$ produces the exact solution.
Any probabilistic or approximation solutions do not achieve the required exactness and usually are much more complicated.
The algorithm ${\cal PD}$ is very simple, clear to prove correctness and to find time complexity.
Moreover, it has a nice relation to well-known Euclid's algorithm of computing Greatest Common Divisor, which is denoted here by ${\cal GCD}$.

Euqlid’s Algorithm $\cal GCD$

* POST: $res = {\cal GCD}(a, n)$.

```python
def gcd(a, n):
    assert 0 <= a <= n

    res = n
    if a > 0:
        res = gcd(n % a, a)

    return res
```

Deterministic algorithm of perfect distribution $\cal PD$

* POST: size of $res$ is $n$.
* POST: $\forall i, res[i] ∈ \{0, 1\}$.
* POST: $\sum_{i=1}^n res[i] = a$
* POST: $res$ is perfectly distributed.

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

## Notation.

* POST means a condition that the procedure guarantees will hold after the call.

### Example. ${\cal PD}(2, 10): a = 2, b = 8$
We want to mix $2$ ones and $8$ zeros up.
The recursive call ${\cal PD}(0, 2)$ returns $[0, 0] = 0^2$, which is assigned to $arr$.
Then the array $res$ is filled as follows: $[(1, 0, 0, 0, 0), (1, 0, 0, 0, 0)] = (1, 0^4)^2$, which is returned by ${\cal PD}(2, 10)$.

### Example. ${\cal PD}(8, 10): a = 8, b = 2$
We want to mix $8$ ones and $2$ zeros up.
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
* ${\cal PD}(6, 8) : [1, 0, 1, 1, 1, 0, 1, 1] = 1, 0, 13 , 0, 12 = (1, 0, 12)^2$
* ${\cal PD}(8, 14) : [1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0] = (1, 0, 1, 1, 0, 1, 0)^2$ $ = ((1, 0, 1)^2 , 0)^2$

### Slow version of $\cal GCD$.

```python
def gcd(a, n):
    assert 0 <= a <= n

    res = n
    if a > 0:
        b = n − a
        if b > a:
            res = gcd(a, b)
        else:
            res = gcd(b, a)

    return res
```

### Slow versions of $\cal PD$

```python
def pd(a, n):
    assert 0 <= a <= n

    res = [0] * n
    if a > 0:
        ofs = 0
        b = n − a
        if b > a:
            arr = pd(a, b)
            for i in range(b):
                res[ofs] = arr[i]
                ofs += 1 + arr[i]

        else:
            arr = pd(b, a)
            for i in range(a):
                res[ofs] = 1 − arr[i]
                ofs += 2 − arr[i]

    return res
```

