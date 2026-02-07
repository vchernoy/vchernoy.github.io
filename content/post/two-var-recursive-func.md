+++
date = "2017-07-06T07:29:43Z"
highlight = true
math = true
tags = ["math", "python", "generating function", "recursion", "dynamic programming", "memoization", "binomial", "multivariate function", "programming", "combinatorics"]
title = "Cracking Multivariate Recursive Equations Using Generating Functions"

[header]
  caption = ""
  image = ""

+++


In this post, we return back to the combinatorial problem discussed in [Introduction to Dynamic Programming and Memoization][intro-to-dp] post.
We will show that generating functions may work great not only for single variable case (see [The Art of Generating Functions][gen-func-art]),
but also could be very useful for hacking two-variable relations (and of course, in general for multivariate case too).

For making the post self-contained, we repeat the problem definition here.

## The Problem

> Compute the number of ways to choose $m$ elements from $n$ elements such that selected elements in one combination are not adjacent.

For example, for $n=4$ and $m=2$, the answer is $3$, since from the $4$-element set: $\lbrace 1,2,3,4 \rbrace$,
there are three feasible $2$-element combinations: $\lbrace 1,4 \rbrace$, $\lbrace 2,4 \rbrace$, $\lbrace 1,3 \rbrace$.

Another example: for $n=5$ and $m=3$, there is only one $3$-element combination: $\lbrace 1,3,5 \rbrace$.

As we discussed in the first post, there is a nice recursive relation for this problem:

$$ F\_{n, m} = F\_{n - 1, m} + F\_{n - 2, m - 1} + [n=m=0] + [n=m=1] $$

We assume that for any $n < 0$ or $m < 0$, $F\_{n,m} = 0$.
The indicator $[P]$ gives $1$ if the predicate $P$ is true.

## The Generating Function for $F\_{n,m}$

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

## The Closed Form for $F\_{n,m}$

Using the infinite series: $ \frac{1}{1-z} = \sum\_{k\geq 0} z^k $,
we can make the following transforms:

$\Phi(x, y) $
$ = \frac{1 + x \cdot y}{1 - x - x^2 y}$
$ = (1 + x \cdot y) \sum\_{k\geq 0} (x + x^2 y)^k $
$ = (1 + x \cdot y) \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i $
$ = \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i + \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i+1} y^{i+1}$

1. Introducing the new variables: $n=k+i, m=i$, we can transform the first expression as follows:
$ \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i} y^i $
$ = \sum\_{m \geq 0, n \geq 2m} {n-m \choose m} x^{n} y^m $.

2. And introducing the new variables: $n=k+i+1, m=i+1$, we can transform the second expression similarly:
$ \sum\_{0 \leq i \leq k} {k \choose i} x^{k+i+1} y^{i+1} $
$ = \sum\_{m \geq 1, n \geq 2m-1} {n-m \choose m-1} x^n y^m $.

The last two transformations give us the closed form:

$F\_{n, m} $
$ = {n - m \choose m} + {n - m \choose m - 1}$.

Which actually equals to

$$ F\_{n, m} = {n - m + 1 \choose m} $$

In the [next post][efficient-impl], we discuss how to implement this closed form efficiently in Python—from a simple factorial-based solution to library implementations and modular arithmetic for competitive programming.

[intro-to-dp]: /post/intro-to-dp/
[gen-func-art]: /post/gen-func-art/
[efficient-impl]: /post/efficient-implementation-non-adjacent-selection/

