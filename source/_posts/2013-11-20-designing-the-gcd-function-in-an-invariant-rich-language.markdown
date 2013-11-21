---
layout: post
title: "Designing the GCD function in an invariant rich language"
date: 2013-11-20 15:21:03 -0600
comments: true
categories: [invariant-lang]
---

What are invariants and what is an invariant rich language?

> In computer science, an invariant is a condition that can be relied upon to be true during execution of a program, or during some portion of it. It is a logical assertion that is held to always be true during a certain phase of execution.

Languages already have invariants. For instance, in most strongly-typed languages when you try to access a variable's value without first setting it you will get an error. But languages don't usually go much farther than that.

An ideal language has a compiler that knows the same amount of information as you do. It knows when you're multiplying by two possibly large ints and will warn you of an overflow. It will let you know when you're accessing an out-of-bound index. Basically, with invariants on every variable and function, the compiler could check every edge case, reducing the amount of errors you find in testing or possibly production.

To illustrate this fact, we design a simple language and iteratively build up a greatest common divisor out of smaller functions.

# Division and Addition
First, let's define division on integers.

	def / (
	    a : (int)
	    b : (int)
	    )
	    return low_level_int_divide(a, b) : (int)

Okay, dividing an `int` by an `int` gives an `int`. Uninteresting. Let's overload it:

	def / (
	    a : (int, positive)
	    b : (int, positive)
	    )
	    return low_level_int_divide(a, b) : (int, positive)

Finally, a useful result. This is something other programming languages don't keep track of. Let's make another quick definition:

	def + (
	    a : (int, positive)
	    b : (int, nonnegative)
	    )
	    return low_level_add(a, b) : (int, positive)

Here we see `nonnegative` for the first time. If a number is `positive` it is obviously `nonnegative`, so we are already seeing a sort of inheritence for invariants.

# Modulus
As I'm sure you remember, `mod` is the modulus function, which gives the remainder when dividing two integers (`5 mod 3 = 2`, `24 mod 6 = 0`).


	def mod (
	    a : (int, positive, invariant(>= b))
	    b : (int, positive)
	    )
	    quotient_floored = a / b
	    temp_product = quotient_floored * b
	    residue = a - temp_product
	    return residue

You may have noticed that `mod` only returns integers ranging from `1` to `b - 1`.

-----

# GCD

For all positive integers $a,b$ with $a\ge b$, let $\text{gcd}(a,b)$ be a positive integer at most $b$. Using the [Euclidean algorithm](http://en.wikipedia.org/wiki/Euclidean_algorithm) we can easily implement this function.

	def gcd (
		a : (int, positive)
		b : (int, positive)
		)
	    while b != 0
	       t := b
	       b := a mod b
	       a := t
	    return a


However, we haven't defined `mod`. Let's do that now. For all $a,b\in\mathbb{N}$, let

$$
\begin{align}
\text{mod}
\end{align}
$$

	def mod (
		a : (int, positive)
		b : (int, positive)
		)
	    quotient_floored = a / b
	    temp_product = quotient_floored * b
	    residue = a - temp_product
	    return residue

Excellent, now how do we implement it? Notice first that we have a `mod` function that we haven't defined, so let's do that. In our pseudocode, a variable's invariants are located to the right in parentheses.


Simple enough. But how will the compiler assign invariants to the return value? Let's think about what invariants `res` should have. Remember that `a modulus b` is the remainder when dividing `a` by `b`. It makes sense that `res` should be an `int` and should satisfy the inequality `0 <= res < b`.

However, as it stands, we don't know what invariants `a / b` will assign! So let's define the `/` operator:

	def /	(
				a : (int)
				b : (int, nonzero)
			)
		res = // some low-level implementation
		return res : (int)

Cool, dividing an `int` by an `int` gives an `int`. Every statically-typed language already gives us that. But really, when we only know that `a` and `b` are `int`s, we can't say much. Consider a stronger definition:

	def /	(
				a : (int, positive)
				b : (int, positive)
			)
		res = // some low-level implementation
		return res : (int)

