---
title: Building Abstractions with Procedures
categories:
  - Structure and Interpretation of Computer Programs (Lisp)
abbrlink: 38ee15cf
date: 2020-01-18 01:30:57
mathjax: true
---

# The Elements of Programming

When we describe a language, we should pay particular attention to the means that the language provides for combining simple ideas to form more complex ideas. Every powerful language has three mechanisms for accomplishing this:

{% note info %}
- **primitive expressions**
- **means of combination**
- **means of abstraction**
{% endnote %}

## Compound Procedures

We begin by examining how to express the idea of "squaring". We might say, "To square something, multiply it by itself". This is expressed in Lisp as

```lisp
(define (square x) (* x x))
```

We can also use square as a building block in defining other procedures. For example, $x^2 + y^2$ can be expressed as

```lisp
(+ (square x) (square y))
```

We can easily define a procedure `sum-of-squares` that, given any two numbers as arguments, produces the sum of their squares

```lisp
(define (sum-of-squares x y)
    (+ (square x) (square y)))
```

Now we can use `sum-of-squares` as a building block in constructing further procedures

```lisp
(define (f a)
    (sum-of-squares (+ a 1) (* a 2)))
```

## The Substitution Model for Procedure Application

### Applicative Order

The interpreter first evaluates the operator and operands and then applies the resulting procedure to the resulting arguments

```lisp
<!-- step 1 -->
(f 5)

<!-- step 2 -->
(sum-of-squares (+ 5 1) (* 5 2))

<!-- step 3 -->
(+ (square 6) (square 10))

<!-- step 4 -->
(+ 36 100)

<!-- step 5 -->
136
```

### Normal Order

The interpreter first substitute operand expressions for parameters until it obtained an expression involving only primitive operators, and would then perform the evaluation

```lisp
<!-- step 1 -->
(f 5)

<!-- step 2 -->
(sum-of-squares (+ 5 1) (* 5 2))

<!-- step 3 -->
(+ (square (+ 5 1)) (square (* 5 2)) )

<!-- step 4 -->
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2)))

<!-- step 5 -->
(+ (* 6 6) (* 10 10))

<!-- step 6 -->
(+ 36 100)

<!-- step 7 -->
136
```

## Example: Square Roots by Newton’s Method

Procedures, as introduced above, are much like ordinary mathematical functions. They specify a value that is determined by one or more parameters. But there is an important difference between mathematical functions and computer procedures. Procedures must be effective.

As a case in point, consider the problem of computing square roots. We can define the square-root function as

$\sqrt x$ = the $y$ such that $y \geq 0$ and $y^2 = x$

This describes a perfectly legitimate mathematical function. We could use it to recognize whether one number is the square root of another, or to derive facts about square roots in general. On the other hand, the definition does not describe a procedure. Indeed, it tells us almost nothing about how to actually find the square root of a given number.

The contrast between function and procedure is a reflection of the general distinction between describing properties of things and describing how to do things, or, as it is sometimes referred to, the distinction between declarative knowledge and imperative knowledge. In mathematics we are usually concerned with declarative (what is) descriptions, whereas in computer science we are usually concerned with imperative (how to) descriptions.

How does one compute square roots? The most common way is to use Newton’s method of successive approximations, which says that whenever we have a guess $y$ for the value of the square root of a number $x$, we can perform a simple manipulation to get a better guess (one closer to the actual square root) by averaging $y$ with $x/y$. For example, we can compute the square root of 2 as follows.

Suppose our initial guess is 1:

| Guess | Quotient | Average |
| :--: | :--: | :--: |
| 1 | (2/1) = 2 | ((2 + 1)/2) = 1.5 |
| 1.5 | (2/1.5) = 1.3333 | ((1.3333 + 1.5)/2) = 1.4167 |
| 1.4167 | (2/1.4167) = 1.4118 | ((1.4167 + 1.4118)/2) = 1.4142 |
| 1.4142 | ... | ... |

Continuing this process, we obtain better and better approximations to the square root.

Now let’s formalize the process in terms of procedures. We start with a value for the radicand (the number whose square root we are trying to compute) and a value for the guess. If the guess is good enough for our purposes, we are done; if not, we must repeat the process with an improved guess. We write this basic strategy as a procedure:

```lisp
(define (sqrt-iter guess x)
    (if (good-enough? guess x)
        guess
        (sqrt-iter (improve guess x) x)))

<!-- there subprocedures -->
(define (improve guess x)
    (average guess (/ x guess)))

(define (average x y)
    (/ (+ x y) 2))

(define (good-enough? guess x)
    (< (abs (- (square guess) x)) 0.001))
```

Finally, we need a way to get started. For instance, we can always guess that the square root of any number is 1

```lisp
(define (sqrt x)
    (sqrt-iter 1.0 x))
```

Actually, we allow a procedure to have internal definitions that are local to that procedure. For example, in the square-root problem we can write

```lisp
(define (sqrt x)
    (define (good-enough? guess x)
        (< (abs (- (square guess) x)) 0.001))
    (define (improve guess x) (average guess (/ x guess)))
    (define (sqrt-iter guess x)
        (if (good-enough? guess x)
            guess
            (sqrt-iter (improve guess x) x)))
    (sqrt-iter 1.0 x))
```

Such nesting of definitions, called **block structure**, is basically the right solution to the simplest name-packaging problem. But there is a better idea lurking here. In addition to internalizing the definitions of the auxiliary procedures, we can simplify them. Since `x` is bound in the definition of `sqrt`, the procedures `good-enough?`, `improve`, and `sqrt-iter` which are defined internally to `sqrt` are in the scope of `x`. Thus, it is not necessary to pass `x` explicitly to each of these procedures. Instead,
we allow `x` to be a free variable in the internal definitions, as shown below. Then `x` gets its value from the argument with which the enclosing procedure `sqrt` is called. This discipline is called **lexical scoping**.

```lisp
(define (sqrt x)
    (define (good-enough? guess)
        (< (abs (- (square guess) x)) 0.001))
    (define (improve guess) (average guess (/ x guess)))
    (define (sqrt-iter guess)
        (if (good-enough? guess)
            guess
            (sqrt-iter (improve guess))))
    (sqrt-iter 1.0))
```

# Procedures and the Processes They Generate

