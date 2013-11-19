---
layout: post
title: "Designing the GCD function in an invariant rich language"
description: ""
category: "blog"
tags: [invariant-lang]
---
{% include JB/setup %}

In this post, we will design a GCD function in an invariant language, but first, what is an invariant?

> In computer science, an invariant is a condition that can be relied upon to be true during execution of a program, or during some portion of it. It is a logical assertion that is held to always be true during a certain phase of execution.

You may be wondering why invariants are important. That's simple. A compiler can use invariants to check your code at compile time so you get less run-time errors and exceptions. With enough invariants in place, the compiler could check for edge cases in every function, reducing the amount of errors you ship to production.

Let's design a simple function that calculates the greatest common divisor of two integers. We will use the division-based [Euclidean algorithm](http://en.wikipedia.org/wiki/Euclidean_algorithm):

	def gcd(a, b)
	    while b != 0
	       t := b
	       b := a mod b
	       a := t
	    return a

What invariants can we impose on this function? Well first, we know that both a and b should be positive integers. The output should also be a positive integer. Excellent, but how do we implement it? Notice first that we have a `mod` function that we haven't defined. Let's define it:

	def mod (
				a : (int, positive, invariant(a>=b))
				b : (int, positive)
			)
		div = a / b
		sum = div * b
		res = a - sub
		return res

Simple enough. But how will the compiler assign invariants to the return value? Let's think about what invariants `res` should have. Remember that the modulus is the remainder when dividing `a` and `b`. So it makes sense that `res` should be an `int` and should satisfy the inequality `0 <= res < b`.

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

