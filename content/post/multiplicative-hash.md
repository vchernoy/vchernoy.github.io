+++
date = "2017-08-02T07:29:43Z"
math = true
highlight = true
tags = ["multiplicative hashing", "hash functions", "prime", "goldan ratio", "gcd", "coprimes", "prime"]
draft = true

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""

+++

## Introduction

Distribution of partitions amongst slices has a huge impact on the performance of XIV systems.
Different solutions for this problem were discussed and implemented in Gen2 and Gen3.
These approaches are not perfect; in fact, any solution for this problem has some drawbacks.
Our goal is to find a "good" solution having not too much impact on customers and having acceptable cost in development and QA and have acceptable performance (as less collisions as possible).
Due to great importance of this problem, we would like present a new approach not yet discussed.

The problem is defined as follows. Given a logical partition number $P$, compute the corresponding slice number $S = s(P)$.
Where $0 ≤ P < 2^{32}$, $0 ≤ S < M$, and $M$ denotes the table size, e.g. in Gen2, $M = 2^{14}$.
In our "binary" world, assumptions that input data (partition numbers) have uniform distribution are not always correct.
Therefore, the hash function $s: P \to S$ must be designed very carefully.
In addition to providing uniform distribution of hash values (slice numbers), it has to add some randomness.
Luckily, this field is well studied: well-known textbooks of Corman's and Knuth's have a good introduction to this field.
The last one has more detail explanation; therefore, without hesitation, we make use of Knuth's book (Section 6.4.):
usually explaining in two paragraphs what the book is saying just in one sentence.

## Basic Ideas

In Gen2 we used the following hash function:
$s(P) = P \bmod M$,

where $M = 2^{14}$ -- the table size or the number of slices. In Gen3 it was proposed to change the number of slices from a power of 2 to a prime number:
$s(P) = P \bmod M$,

where $M = 16411$ is a prime number $> 2^{14}$. Such form of hashing is called "Division Remainder Method". The main idea of this Notes is to demonstrate another hashing method called "Multiplicative Method":

$$f(P) = A \cdot P \bmod W$$

$$s(P) = f(P) \div (W \div M)$$

Where $W = 2^{32}$ -- the machine word, $M = 2^{14}$ -- the table size, $\div$ denotes an integer division, $W \div M = 2^{32 - 14} = 2^{18}$, $A = 2654435761$ -- some "magic" integer.
It will be explained later, for now we assume that due to this magic, the value of $f(P)$ is almost random.
It is easy to see that $s(P)$ is in valid interval of slices, between 0 and $M = 2^{14}$.

### Example of Hash Codes

| partition $P$ | dec $P$ | hex $P$ | binary $f(P)$ | binary $S$ | hex $S$ | dec $S$ |
|---------------|---------|---------|---------------|------------|---------|---------|
| $0$ | 0 | 00000000 | 00000000000000000000000000000000 | 00000000000000 | 0000 | 0 |
| $1$ | 1 | 00000001 | 10011110001101110111100110110001 | 10011110001101 | 278D | 10125 |
| $2$ | 2 | 00000002 | 00111100011011101111001101100010 | 00111100011011 | 0F1B | 3867 |
| $3$ | 3 | 00000003 | 11011010101001100110110100010011 | 11011010101001 | 36A9 | 13993 |
| $2^{14}-1$ | 16383 | 00003FFF | 01000000001101001100011001001111 | 01000000001101 | 100D | 4109 |
| $2^{14}$ | 16384 | 00004000 | 11011110011011000100000000000000 | 11011110011011 | 379B | 14235 |
| $2^{14}+1$ | 16385 | 00004001 | 01111100101000111011100110110001 | 01111100101000 | 1F28 | 7976 |
| $2^{14}+2$ | 16386 | 00004002 | 00011010110110110011001101100010 | 00011010110110 | 06B6 | 1718 |
| $2^{15}-1$ | 32767 | 00007FFF | 00011110101000010000011001001111 | 00011110101000 | 07A8 | 1960 |
| $2^{15}$ | 32768 | 00008000 | 10111100110110001000000000000000 | 10111100110110 | 2F36 | 12086 |
| $2^{15}+1$ | 32769 | 00008001 | 01011011000011111111100110110001 | 01011011000011 | 16C3 | 5827 |
| $2^{15}+2$ | 32770 | 00008002 | 11111001010001110111001101100010 | 11111001010001 | 3E51 | 15953 |
| $2^{30}-1$ | 1073741823 | 3FFFFFFF | 10100001110010001000011001001111 | 10100001110010 | 2872 | 10354 |
| $2^{30}$ | 1073741824 | 40000000 | 01000000000000000000000000000000 | 01000000000000 | 1000 | 4096 |
| $2^{30}+1$ | 1073741825 | 40000001 | 11011110001101110111100110110001 | 11011110001101 | 378D | 14221 |
| $2^{30}+2$ | 1073741826 | 40000002 | 01111100011011101111001101100010 | 01111100011011 | 1F1B | 7963 |
| $2^{31}-1$ | 2147483647 | 7FFFFFFF | 11100001110010001000011001001111 | 11100001110010 | 3872 | 14450 |
| $2^{31}$ | 2147483648 | 80000000 | 10000000000000000000000000000000 | 10000000000000 | 2000 | 8192 |
| $2^{31}+1$ | 2147483649 | 80000001 | 00011110001101110111100110110001 | 00011110001101 | 078D | 1933 |
| $2^{31}+2$ | 2147483650 | 80000002 | 10111100011011101111001101100010 | 10111100011011 | 2F1B | 12059 |
| $2^{32}-1$ | 4294967295 | FFFFFFFF | 01100001110010001000011001001111 | 01100001110010 | 1872 | 6258 |

### Implementation in C

The new hashing function looks at bit complicated, so let's consider how it (and others) could be implemented in C code. The type uint32_t denotes 32-bit unsigned integer. We start from examples of Division Remainder hashing. For $M$ being a power of 2:

```c
const uint32_t M = 2 << 14;

uint32_t slice(uint32_t P) {
    return P & (M - 1);
}
```

For prime $M$:

```c
const uint32_t M = 16411;

uint32_t slice(uint32_t P) {
    return P % M;
}
```

The next function implements Multiplicative hashing:

```c
const uint32_t M = 2 << 14;
const uint32_t A = 2654435761;

uint32_t slice(uint32_t P) {
    return (P * A) >> 18;
}
```

It really works well for any input data (partitions) and allows to use the same number of slices as in Gen2: $2^{14}$. As a read can see, the last function has only two operation, one is multiplication and other is logical shift. On some architectures this function may be faster then the second one finding a modulo!

Details for Math Fans

In this section we describe properties of every approach. On analysis of the simple functions, we demonstrate the informal tools that we later apply to more complex functions. We hope that even non great fan of mathematics will be able to catch the main point.

## The Power of 2 is Evil

The first function we will consider is $s(P) = P \bmod 2^{14}$.
Let's recall some properties of this function.

Flipping just one bit in the given partition not always leads to a change in the corresponding slice (in hash-value).
More formal: for some bit $i$, "occasionally" $s(P) = s(P + 2^i)$ holds; here we assume for simplicity that $P$ has $i$ bit cleared.
In our "binary" world, partitions differing in only one bit may have a strong correlation.
Therefore, we are expecting that a "good" hashing will produce different hash-code for them (i.e., it will place the partitions in different slices).
But for this function, the property fails on any partition $P ≥ 2^{14}$.
There are simple templates of partitions having the same hash-code.
For example, for any $i$ and $S$, partitions $P = S + 2^{14} \cdot i$ will be placed by the function to the same slice $S$.
From a "good" hashing, we are expecting not to have very common patterns, especially, of binary form.
But again, this function fails on this condition.

## The Art of Primes

The second function is $s(P) = P \bmod M$.
Where $M = 16411$.
For this function, flipping any bit in a partition $P$ changes the hash code, because $M$ is chosen to be a prime.
Actually, this condition on $M$ is too strong.
For satisfying this property, it is sufficient that $M ≠ 2^i$ will hold.
For example, $M = 15 · 12 · 97 = 17460$ (15 modules, 12 disks, 97 is a prime) is also "good".

Since $M$ is prime, it seems that the following pattern is not common: $P = S + M · i$. But actually, primes that are close to a power of 2 are also not good. Knuth recommends to choose such $M$ that the following condition will not hold for any small integers $a$ and $j$: $r^j ≡ ± a \pmod M$. Where $r$ denotes the base of computation. From explanation of Knuth, the meaning of $r$ for our case is not too clear: whether $r=2$, $r=16$, or $r=256$? It seems the answer very depends on the type of input data. By Knuth, if $r=2$, the chosen $M$ is not so good, since $M = 16411 = 2^{14} + 27$, and hence $2^{14} ≡ -27 \pmod M$. For $r=16$, we get that $16^ ≡ -108 \pmod M$. For $r=256$, $16^2 ≡ -108 \pmod M$. Knuth explains that such $M$ may produce a hash code that is a simple composition of key digits (in $r$ base system). Instead of trying to understand this explanation, we will give some intuition. Working with numbers, a programmer usually chooses powers of 2 for sizes of structures and buffers (e.g., $2^{10}$ bytes). Then he defines the format of such data and introduces headers (e.g. the header size = 20 bytes). Hence, the size of data without the header becomes very close to the power of 2 (in our example, $2^{10} - 20 = 1004$). On the other hand, embedding this structure to an outer packet (assume the size of this outer header is 30 bytes) leads to the total size being also close to the power of 2 ($2^{10} + 30 = 1054$). As result, most of numbers in our "binary" world are either powers of 2 or close to them. Therefore, such choice of $M$ increases collisions. In other words, not only powers of 2 are *evil*, but primes closing to them are *evil* too.

As an example of a "good" prime, let's consider $M = 24571$. It is a bit smaller then the middle of $2^{14}$ and $2^{15}$.

Introducing a little change in this hashing function, we get better hashing even for $M = 2^{14}$: $s(P) = P \bmod N \bmod M$. In this case $N$ must be some "big" and "far" from a power of 2 prime number. The discussion of this method is out of scope of this Notes.

## The Magic of Golden Ratio

We now present in details the multiplicative hashing. First, we discuss the properties of the function $f$: $f(P) = A · P \bmod W$.

The magic number $A$ is chosen to have the following nice properties: $W / A ≈ 1.6$ -- the golden ratio and $GCD(A, W) = 1$ ($A$ and $W$ are relatively prime or coprime). Therefore, the function $f$ is 1-1 on 32-bit integers. In other word, for any $P_1~ ≠ P_2$, $f(P_1) ≠ f(P_2)$ holds. This function has a role of perturbation or randomization of the input data (partitions).

Assume given a partition $P$. Let's set the 0 bit in $P$. Then we get $f(P + 1) = (f(P) + A) \bmod W$. So the bit 0 dirties those bits of $f(P)$ that are set in $A$. This implies that $A$ must be chosen from the following range: $2^{31} < A < 2^{32}$, and in addition it must have a half of bits being set. By Knuth, the best choice is to define it to be the golden value of $W$: $W / A = 1.6 \ldots$ . In this case setting $i$\-bit in $P$ dirties in $f(P)$ all the bits from $i$ to 31.

Using the properties of $f()$, we show the intuition why the function $s(P) = f(P) \div 2^{18}$ is a "good" hashing function.

The function $s$ remains 14 most significant bits of $f(P)$. Therefore, from the properties of $f()$, flipping any bit in $P$ will yield flipping at least in one bit of $s(P)$.
The function $s$ puts into one slice $S$ all the partitions of the form: $P = (S · 2^{18} + i) · B \bmod 2^{32}$, where $B = 244002641$ (see for details the section Invertibility) and $0 ≤ i < 2^{18}$. Assume that $P_0 = S · B · 2^{18} \bmod 2^{32}$, then $P_i = (P_0 + i · B) \bmod 2^{32}$. The function $g(i) = i · B$ produces almost random 32-bit integers from 18-bit integers. Therefore, it is almost impossible to derive a "nice" sub-sequence from the values of set $\{g(i) | 0 ≤ i < 2^{18}\}$.

## Invertibility

Given a 14-bit slice $S$ and some 18-bit identifier $Id$, we are interested to compute the partition $P$ that is placed by the function $s$ at slice $S$. Note that given a partition $P$, the computation of $Id$ is also depends on the chosen method of hashing. We present for each hashing how the identifier and the inverse of $s()$ can be computed.

For the simplest function $s(P) = P \bmod 2^{14}$, the identifier is computed as follows: $Id = P \div 2^{14}$.
Then its inverse is $p(S, Id) = S + Id · 2^{14}$.
For prime $M$, the identifier is $Id = P \div M$ and the inverse function is $p(S, Id) = S + Id · M$.
 For multiplicative hashing, we have $Id = A · P \bmod 2^{14}$. Then $p(S, Id) = (S · 2^{18} + Id) · B \bmod 2^{32}$.
This implies from the fact that $A · B ≡ 1 \pmod 2^{32}$ and the inverse of $f()$ is $f^{-1}(x) = x · B \bmod 2^{32}$.

We show the implementation of $p()$ in C code for the multiplicative hashing only:

```c
const uint32_t M = 2 << 14;
const uint32_t B = 244002641;

uint32$t partition(uint32_t S, uint32_t Id) {
    return (S << 18 + Id) * B;
}
```

## Big Numbers

In this section we discuss the behavior of multiplicative hashing on big partition numbers. Recall that flipping just one $i$-bit in $P$ has impact on $(32 - i)$ bits (from $i$ to 31) in $f(P)$. By the definition, $s(P)$ derives 14 most significant bits of $f(P)$. As result, for $i < 18$, the flipping $i$-bit dirties most of 14 bits of $s(P)$, while for $i ≥ 18$, it change less then 14 bits of $s(P)$: $32 - i ≤ 14$. For example, flipping the most significant 31-bit of the partition $P$ flips the only one 14-bit of the corresponding slice $S = s(P)$. This is not a real problem for our case, the partitions are still putted into different slides.

Another problem with flipping of most significant bits is following. For a set of partitions differing in most significant bits only, the number of collisions is greater then for the case that partitions are different in less significant bits. But due to the nice choice of $A$ (being golden ratio of $W$), this is still sufficiently good distribution. Intuitively, if the partitions differ in x most significant bits, then slices for them will also differ in x bits. So most of these partitions will be placed to different slices.

Note that the number of impacted bits in $s(P)$ decreases only for partitions with more then 18 bits. Even then, as described above, it does not mean that we will have a huge degradation of the hashing performance. But if we want to prevent any potentially dangerous case, we may extend the machine word to 64-bits, especially if the used architecture is also 64-bit. In this case, we just compute a new pair of integers $A$ and $B$ ($A$ is the golden ration of and is coprime with $W=2^{64}$, $B$ is coprime with $A$) and correct logical shifts. This is an example of implementation in C:

```c
const uint64_t M = 2 << 14;
const uint64_t A = 11400714819323198485;

uint64_t slice(uint64_t P) {
    return (P * A) >> 50;
}
```

### Example of Big Partitions

The table contains computation of slices for partitions that differ only in three most significant bits. As we can see, corresponding slices also differ only in three bits. This property may be considered as drawback. But note that this case has no collisions.

| binary $P$ | hex $P$ | binary $f(P)$ | binary $S$ | hex $S$ | dec $S$ |
|------------|---------|---------------|------------|---------|---------|
| **000**10101010111010100100101011001 | 155D4959 | **100**01101010010011100011110001001 | **100**01101010010 | $2352$ | $9042$ |
| **001**10101010111010100100101011001 | 355D4959 | **101**01101010010011100011110001001 | **101**01101010010 | $2B52$ | $11090$ |
| **010**10101010111010100100101011001 | 555D4959 | **110**01101010010011100011110001001 | **110**01101010010 | $3352$ | $13138$ |
| **011**10101010111010100100101011001 | 755D4959 | **111**01101010010011100011110001001 | **111**01101010010 | $3B52$ | $15186$ |
| **100**10101010111010100100101011001 | 955D4959 | **000**01101010010011100011110001001 | **000**01101010010 | $0352$ | $850$ |
| **101**10101010111010100100101011001 | B55D4959 | **001**01101010010011100011110001001 | **001**01101010010 | $0B52$ | $2898$ |
| **110**10101010111010100100101011001 | D55D4959 | **010**01101010010011100011110001001 | **010**01101010010 | $1352$ | $4946$ |
| **111**10101010111010100100101011001 | F55D4959 | **011**01101010010011100011110001001 | **011**01101010010 | $1B52$ | $6994$ |


## Conclusion

TODO

### Links and References

* T Corman, Ch Leiserson, R Rivest, "Introduction to Algorithms", 1999
* D Knuth, "Sorting and Searching", volume 3 of "The Art of Computer Programming", 1973. Section 12.3
* Integer Hash Function, Thomas Wang
* Selecting a Hash Function, Scott Gasch
* Hash functions, Paul Hsieh
* Wikipedia: Hash Table -- sites Thomas Wang that the multiplicative hash has "particularly poor clustering behavior".
* Prime Double Hash Table, Thomas Wang -- the wrong proof.
* http://web.archive.org/web/19990903133921/http://www.concentric.net/~Ttwang/tech/primehash.htm
* Wikipedia: Talking Hash Table -- note that the proof is wrong, see "Info about multiplicative hashing is wrong" section.

