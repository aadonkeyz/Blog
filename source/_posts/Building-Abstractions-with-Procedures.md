---
title: Building Abstractions with Procedures
categories:
  - 《 Structure and Interpretation of Computer Programs (Lisp) 》
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

{% note info %}
<center>$\sqrt x$ = the $y$ such that $y \geq 0$ and $y^2 = x$</center>
{% endnote %}

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

Actually, we allow a procedure to have internal definitions that are local to that procedure. For example, in the square-root problem we can write

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

A procedure is a pattern for the local evolution of a computational process. It specifies how each stage of the process is built upon the previous stage. We would like to be able to make statements about the overall, or global, behavior of a process whose local evolution has been specified by a procedure.

## Linear Recursion and Iteration

We begin by considering the factorial function, defined by

{% note info %}
<center>$n! = n \cdot (n-1) \cdot (n-2) \cdots 3 \cdot 2 \cdot 1$</center>
{% endnote %}

There are many ways to compute factorials. One way is to make use of the observation that $n!$ is equal to $n$ times $(n-1)!$ for any positive integer $n$. Thus, we can compute $n!$ by computing $(n-1)!$ and multiplying the result by $n$. If we add the stipulation that $1!$ is equal to $1$, this observation translates directly into a procedure

```lisp
(define (factorial n)
    (if (= n 1)
        1)
        (* n (factorial(- n 1))))

; (factorial 6)
; (* 6 (factorial 5))
; (* 6 (* 5 (factorial 4)))
; (* 6 (* 5 (* 4 (factorial 3))))
; (* 6 (* 5 (* 4 (* 3 (factorial 2)))))
; (* 6 (* 5 (* 4 (* 3 (* 2 (factorial 1))))))
; (* 6 (* 5 (* 4 (* 3 (* 2 1)))))
; (* 6 (* 5 (* 4 (* 3 2))))
; (* 6 (* 5 (* 4 6)))
; (* 6 (* 5 24))
; (* 6 120)
; 720
```

Now let’s take a different perspective on computing factorials. We could describe a rule for computing $n!$ by specifying that we first multiply $1$ by $2$, then multiply the result by $3$, then by $4$, and so on until we reach $n$. More formally, we maintain a running product, together with a counter that counts from $1$ up to $n$. We can describe the computation by saying that the counter and the product simultaneously change from one step to the next according to the rule

{% note info %}
- product ← counter * product
- counter ← counter + 1
{% endnote %}

and stipulating that $n!$ is the value of the product when the counter exceeds $n$.

```lisp
(define (factorial n)
    (fact-iter 1 1 n))

(define (fact-iter product counter max-count)
    (if (> counter max-count)
        product)
        (fact-iter (* counter product)
                   (+ counter 1)
                   max-count))

; (factorial 6)
; (factorial 1 1 6)
; (factorial 1 2 6)
; (factorial 2 3 6)
; (factorial 6 4 6)
; (factorial 24 5 6)
; (factorial 120 6 6)
; (factorial 720 7 6)
```

Consider the first process. The substitution model reveals a shape of expansion followed by contraction, the expansion occurs as the process builds up a chain of deferred operations (in this case, a chain of multiplications). The contraction occurs as the operations are actually performed. This type of process, characterized by a chain of deferred operations, is called a **recursive process**. Carrying out this process requires that the interpreter keep track of the operations to be performed later on. In the computation of $n!$, the length of the chain of deferred multiplications, and hence the amount of information needed to keep track of it, grows linearly with $n$ (is proportional to $n$), just like the number of steps. Such a process is called a **linear recursive process**.

By contrast, the second process does not grow and shrink. At each step, all we need to keep track of, for any $n$, are the current values of the variables `product`, `counter`, and `max-count`. We call this an **iterative process**. In general, an iterative process is one whose state can be summarized by a fixed number of state variables, together with a fixed rule that describes how the state variables should be updated as the process moves from state to state and an (optional) end test that specifies conditions under which the process should terminate. In computing $n!$, the number of steps required grows linearly with $n$. Such a process is called
a **linear iterative process**.

The contrast between the two processes can be seen in another way. In the iterative case, the program variables provides a complete description of the state of the process at any point. If we stopped the computation between steps, all we would need to do to resume the computation is to supply the interpreter with the values of the three program variables. Not so with the recursive process. In this case there is some additional "hidden" information, maintained by the interpreter and not  contained in the program variables, which indicates "where the process is" in negotiating the chain of deferred operations. The longer the chain, the more information must be maintained.

In contranstin iteration and recursion, we must be careful not to confuse the notion of a **recursive process** with the notion of a **recursive procedure**. When we describe a procedure as recursive, we are referring to the syntactic fact that the procedure definition refers (either directly or indirectle) to the procedure itself. But when we describe a process as following a pattern that is, say, linearly recursive, we are speaking about how the process evolves, not about the syntax of how a procedure is written. It may seem disturbing that we refer to a recursive procedure such as `fact-iter` as generating an iterative process. However, the process really is iterative: **Its state is captured completely by its three state variables, and an interpreter need keep track of only three variables in order to execute the process.**

## Tree Recursion

