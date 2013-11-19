---
layout: post
title: "Designing the GCD function in an invariant rich language"
description: ""
category: "blog"
tags: [invariant-lang]
---
{% include JB/setup %}

First, what is an invariant?

> In computer science, an invariant is a condition that can be relied upon to be true during execution of a program, or during some portion of it. It is a logical assertion that is held to always be true during a certain phase of execution. For example, a loop invariant is a condition that is true at the beginning and end of every execution of a loop.

You may be wondering why invariants are important. That's simple. A compiler can use invariants to check your code at compile time so you get less run-time errors and exceptions. With enough invariants in place, the compiler could check for edge cases in every function, reducing the amount of errors you ship to production.

Let's design a simple function that calculates the greatest common divisor of two integers. We will use the division-based [Euclidean algorithm](http://en.wikipedia.org/wiki/Euclidean_algorithm):

	def gcd(a, b)
	    while b != 0
	       t := b
	       b := a mod b
	       a := t
	    return a

What invariants can we impose on this function? Well first, we know that both a and b should be positive integers. The output should also be a positive integer. Excellent, but how do we implement it? Notice first that we have a `mod` function that we haven't defined.