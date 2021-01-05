---
date: 2020-09-29
code: ma5.101
author: Abhinav Menon
title: Discrete Structures Lecture 7
number: 7
---


## Elliptic Curves
These curves have the form $y^{2} = x^{3} + ax + b$.

For the purposes of cryptography, those elliptic curves are considered whose variables and coefficients are restricted to a finite field (either $\mathbb{Z}_{p}$ or $\mathbb{GF}(2^{n})$; we will consider only the former).


## Elliptic Curves over $\mathbb{Z}_{p}$
We consider the congruence $y^{2} \equiv x^{3} + ax + b$ (mod $p$), where $a$ and $b$ are constants in $\mathbb{Z}_ {p}$ such that $4a^{3} + 27b^{2} \not\equiv 0$ (mod $p$). This is equivalent to the set $E _{p}(a,b)$, consisting of solutions to the congruence together with the point at infinity $\mathcal{O}$, which functions as an additive identity.

The number of points in $E_{p}(a,b)$ is denoted by #$E$ and satisfies $p + 1 - 2\sqrt{p} \leq$ #$E \leq p + 1 + 2\sqrt{p}$.

A binary operation, called "addition" and denoted by $+$, is defined on $E_{p}(a,b)$. Under this operation, $E_{p}(a,b)$ forms an abelian group. Refer to slides (21 to 26) for details of addition, doubling, scalar multiplication and additive inverses of points.


## Elliptic Curve Cryptography (ECC)
ECC relies on the computational infeasibility of determining $k$ given $kP$ and $P$, even though it is relatively easy to determine $kP$ given $k$ and $P$. This is called the elliptic curve discrete logarithm problem (ECDLP).


## Hierarchical Access Control using ECC
Refer to slides (32 to 34) for a general overview of hierarchical access control. In the system described by Chung et al., the security classes having higher clearance use publicly available information to determine the secret keys of any lower-clearance classes. Here and elsewhere, the terms "higher-clearance class" and "lower-clearance class" are equivalent to "predecessor class" and "successor class" respectively.

### Relationship-building Phase
The central authority or CA determines the hierarchical structure among the security classes and their relative clearances.

### Key Generation Phase
First, a large prime $p$, an elliptic curve $E_{p}(a,b)$ over $\mathbb{Z}_ {p}$ and a function $m: E _{p}(a,b) \rightarrow \mathbb{Z}$ are selected.

For each security class $SC_{j}$, the CA now:

**Step 1.** Assigns a base point $G_{j}$, a secret key $sk_{j}$ and a sub-secret key $s_{j}$.

**Step 2.** Computes $s_{i}G_{j} = (a_{j,i},b_{j,i})$ for all predecessor classes $SC_{i}$ of $SC_{j}$. In other words, it computes the product of the base point with the sub-secret key of each of the higher-clearance classes. The resulting point is converted into a number $m(a_{j,i},b_{j,i})$ using the function $m(\cdot)$.

**Step 3.** Computes a polynomial $f_{j}(x)$ using the above numbers as:

$f_{j}(x) \equiv \prod_{SC_{i} \geq SC_{j}} (x - m(a_{j,i},b_{j,i})) + sk_{j}$ (mod $p$). [*]

**Step 4.** Sends $sk_{j}$ and $s_{j}$ through a secure channel and announces $p$, $m(\cdot)$, $G_{j}$ and $f_{j}(x)$ publicly.

[*] Note that the zeroes of $f_{j}(x) - sk_{j}$ are the numbers obtained from $s_{i}G_{j}$ using $m(\cdot)$.

### Key Derivation Phase
In this phase, a security class $SC_{i}$ finds the secret keys $sk_{j}$ of all successor classes $SC_{j} \geq SC_{i}$, i.e., all classes with equal or lower clearance.

First it computes $s_{i}G_{j} = (a_{j,i},b_{j,i})$, i.e., multiplies its sub-secret key with the lower class's base point. It converts this point to a number using $m(\cdot)$. This number is $m(a_{j,i},b_{j,i})$.

By the definiton of $f_{j}$, this number is a root of $f_{j}(x) - sk_{j}$, i.e., $f_{j}(m(a_{j,i},b_{j,i})) - sk_{j} \equiv 0$ (mod $p$). Therefore, $f_{j}(m(a_{j,i},b_{j,i})) \equiv sk_{j}$ (mod $p$), and $SC_{i}$ has found $sk_{j}$.

**Note.** The function $m(x,y)$ is generally defined using a hash function as $m(x,y) = h(x \Vert y)$, where $\Vert$ is a bit concatenation operator. The intermediate $m(\cdot)$ was introduced in this description for convenience.

### An Example
Let us assume that the CA has completed the relationship-building phase and the key generation phase. Further, suppose that $SC_{4}$ would like to find the secret key $sk_{5}$ of its successor class $SC_{5}$. Other predecessor classes of $SC_{5}$ are, say, $SC_{1}$ and $SC_{2}$.

Therefore, for $SC_{5}$, the CA has computed $s_{1}G_{5} = (a_{5,1},b_{5,1})$, $s_{2}G_{5} = (a_{5,2},b_{5,2})$ and $s_{4}G_{5} = (a_{5,4},b_{5,4})$. From these, the public polynomial $f_{5}$ is computed and announced: $f_{5} \equiv (x - m(a_{5,1},b_{5,1}))(x - m(a_{5,2},b_{5,2}))(x - m(a_{5,4},b_{5,4}))(x - m(a_{5,5},b_{5,5})) + sk_{5}$ (mod $p$).



Now, $SC_{4}$ needs to find $sk_{5}$. It does this using $s_{4}$ (which only it knows), $m(\cdot)$, $G_{5}$ and $f_{5}$ (which are public).

First, it computes $s_{4}G_{5} = (a_{5,4},b_{5,4})$. This is converted to a number using $m(\cdot)$ ( i.e., $m(a_{5,4},b_{5,4})$ is calculated) and substituted in $f_{5}(x)$.

By the definition of $f_{5}$, this number is a root of $f_{5}(x) - sk_{5}$. Therefore, $f_{5}(m(a_{5,4},b_{5,4})) \equiv sk_{5}$ (mod $p$), and $SC_{4}$ has found $sk_{5}$.

What if $SC_4$ wanted to find, say, $sk_2$? It would need to substitute $m(a_{2,2},b_{2,2})$ in $f_2$ (since, by construction, $m(a_{4,2},b_{4,2})$ is not a root of $f_2 - sk_2$). But $(a_{2,2},b_{2,2})$ is simply $s_2G_2$, so $SC_2$ would need to find $s_2$ first. The only way to do this is to factorise $f_5 - sk_5$ (which it knows), getting $m(a_{5,2},b_{5,2})$, and calculating $s_2$ from this..

However, $m$ is nearly impossible to invert, so to compute $(a_{5,2},b_{5,2})$ (which is $s_2G_5$) is infeasible. Furthermore, even if this were possible, calculating $s_2$ from $s_2G_5$ and $G_5$ is also computationally infeasible (ECDLP). Hence, $SC_4$ has no way to find out $s_2$, and therefore $sk_2$.

