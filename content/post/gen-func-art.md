+++
date = "2017-07-05T07:29:43Z"
highlight = true
math = true
tags = ["math", "generating function", "recursion", "dynamic programming", "binomial", "combinatorics", "golden ratio", "fibonacci numbers"]
title = "The Art of Generating Functions"

[header]
  caption = ""
  image = ""

+++

The notion of generating functions and its application to solving recursive equations are very well-known.
For reader who did not have a chance to get familiar with the topics,
I recommend to take a look at very good book:
[Concrete Mathematics: A Foundation for Computer Science, by Ronald L. Graham, Donald E. Knuth, Oren Patashnik][wiki-concrete-math].

Generating functions are usually applied to single variable recursive equations.
But actually, the technique may be extended to multivariate recursive equations, or even to a system of recursive equations.
Readers who are familiar with one-variable case, may jump directly to the next post:
[Cracking Multivariate Recursive Equations Using Generating Functions][two-var-recursive-func].

## The Generating Function for Fibonacci Sequence

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

## Two Closed Forms for Fibonacci Relation

Nice expression, but what can we do with this "magic" formula?
We can hack it in two different ways, and depending on our next step, we can get different closed forms for $f\_n$.

### The 1st closed form

One of the standard ways is to split the fraction into two simple ones of the form: $\frac{A}{x+B}$.
Note that the roots of the quadratic equation: $1 - x - x^2 = 0$ are $x\_0=-\phi$ and $x\_1=\phi^{-1}$,
where $\phi$ is the _Golden Ratio_:

$ \phi = \frac{1 + \sqrt{5}}{2} = 1.618\dots $.

Let's do a quick test:

$ -(x-x\_0) (x-x\_1) $
$ = -(x+\phi) \left(x-\phi^{-1}\right) $
$ =-x^2 - x\cdot\left(\phi - \phi^{-1}\right) + 1 $
$ = -x^2 - x + 1 $.

Now we can transform the generating function $\Phi(x)$ as follows:

$\Phi(x)$
$ = \frac{1}{1-x-x^2}$
$ = -\frac{1}{(x+\phi) \left(x-\phi^{-1}\right)}$
$ = \frac{A}{x+\phi} - \frac{A}{x-\phi^{-1}}$
$ = A\cdot\phi^{-1} \frac{1}{1+x\cdot\phi^{-1}} + A\cdot\phi \frac{1}{1 - x\cdot\phi}$,

where $A=\frac{1}{\phi+\phi^{-1}}$.

The next trick is to apply the infinite series $\frac{1}{1-z} = \sum\_n z^n$ to each of two expressions and to simplify the sums:

$\Phi(x) $
$ = A\cdot\phi^{-1} \sum\_n \left(-x\cdot \phi^{-1}\right)^n + A\cdot\phi \sum\_n (x\cdot\phi)^n $
$ = A\cdot\phi^{-1} \sum\_n (-\phi)^{-n} x^n + A\cdot\phi \sum\_n \phi^n x^n $
$ = \sum\_n A\cdot\left(\phi^{-1} (-\phi)^{-n} + \phi\cdot \phi^n \right)  x^n $
$ = \sum\_n A\cdot\left(\phi^{n+1} - (-\phi)^{-n-1}\right) x^n $
$ = \sum\_n \frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}} x^n $.

The term before $x^n$ is nothing but $f\_n$, which gives the first closed form for the Fibonacci relation:

$$f\_n=\frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}}$$

### The 2nd closed form

Another way to crack the generating function is to apply the infinite series $\frac{1}{1-z} $ $= \sum\_n z^n$ directly to the generating function:

$ \Phi(x) $
$ = \frac{1}{1 - x - x^2} $
$ = \frac{1}{1 - (x+x^2)}$
$ = \sum\_n (x+x^2)^n $
$ = \sum\_n (1+x)^n x^n $

Now we can use the Binomial formula: $(a + b)^n = \sum\_{0\leq k\leq n} {n \choose k}  a^{n-k} b^k$ in order to expand $(1+x)^n$:

$ = \sum\_{0\leq k \leq n} {n \choose k} x^{n+k} $.

By introducing the new variable $t=n+k$, we obtain

$ = \sum\_{t/2 \leq n \leq t} {n \choose t-n} x^t $
$ = \sum\_t \sum\_{n=t/2}^t {n \choose t-n} x^t $

Which gives us another closed form for the Fibonacci numbers:

$$f\_t = \sum\_{n=t/2}^t {n \choose t-n}$$

## Conclusion

Believe it or not, but the two forms define the same Fibonacci sequence.

$$ f\_n = \frac{\phi^{n+1} - (-\phi)^{-n-1}}{\phi+\phi^{-1}} $$

$$ f\_n = \sum\_{i=n/2}^n {i \choose n-i} $$

Just notice how powerful the generating functions are!
An interested reader can try to prove by induction the correctness of the relations above.
If you liked the topic, you are wellcome to take a look at the next post where we extend the tools to mutivariate recursions.



[two-var-recursive-func]: /post/two-var-recursive-func/
[wiki-concrete-math]: https://en.wikipedia.org/wiki/Concrete_Mathematics
