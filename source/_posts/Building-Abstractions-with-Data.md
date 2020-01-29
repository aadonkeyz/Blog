---
title: Building Abstractions with Data
categories:
  - 《 Structure and Interpretation of Computer Programs (Lisp) 》
abbrlink: 84aac73f
date: 2020-01-28 13:05:41
---

# Introduction to Data Abstraction

We noted that a procedure used as an element in creating a more complex procedure could be regarded not only as a collection of particular operations but also a procedural abstraction. That is, the details of how the procedure was implemented could be suppressed, and the particular procedure itself could be replaced by any other procedure with the same overall behavior. In other words, we could make an abstraction that would separate the way the procedure would be used from the details of how the procedure would be implemented in terms of more primitive procedures. The analogous notion for compound data is called **data abstraction**. Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects.

The basic idea of data abstraction is to structure the programs that are to use compound data objects so that they operate on "abstract data." That is, our programs should use data in such a way as to make no assumptions about the data that are not strictly necessary for performing the task at hand. At the same time, a "concrete" data representation is defined independent of the programs that use the data. The interface between these two parts of our system will be a set of procedures, called **selectors** and **constructors**, that implement the abstract data in terms of the concrete representation.

## Example: Arithmetic Operations for Rational Numbers

suppose we want to do arithmetic with rational numbers. We want to be able to add, subtract, multipy, and divide them and to test whether two rational numbers are equal.

Let us begin by assumping that we already have a way of constructing a rational number from a numerator and a denominator. We also assume that, given a rational number, we have a way of extracting (or selecting) its numerator and its denominator. Let us further assume that the constructor and selectors are available as procedures:

{% note info %}
- `(make-rat <n> <d>)` returns the rational number whose numerator is the integer `<n>` and whose denominator is the integer `<d>`
- `(number <x>)` returns the numerator of the rational number `<x>`
- `(denom <x>)` returns the denominator of the rational number `<x>`
{% endnote %}

We are using here a powerful strategy of synthesis: **wishful thinking**. We haven't yet said how a rational number is represented, or how the procedures `numer`, `denom`, and `make-rat` should be implemented. Even so, if we did have these three procedures, we could then add, subtract, multiply, divide, and test equality.

```lisp
(define (add-rat x y)
    (make-rat (+ (* (numer x) (denom y))
                 (* (numer y) (denom x)))
              (* (denom x) (denom y))))


(define (sub-rat x y)
    (make-rat (- (* (numer x) (denom y))
                 (* (numer y) (denom x)))
              (* (denom x) (denom y))))

(define (mul-rat x y)
    (make-rat (* (numer x) (numer y))
              (* (denom x) (denom y))))

(define (div-rat x y)
    (make-rat (* (numer x) (denom y))
              (* (denom x) (numer y))))

(define (equal-rat? x y)
    (= (* (numer x) (denom y))
       (* (numer y) (denom x))))
```

Now we have the operations on rational numbers defined in terms of the selector and constructor procedures `numer`, `denom`, and `make-rat`. But we haven't yet defined these. What we need is some way to glue together a numerator and a denominator to form a rational number.

### Pairs

To enable us to implement the concrete level of our data abstraction, our language provides a compound structure called a **pair**, which can be constructed with the primitive procedure `cons`. This procedure takes two arguments and returns a compound data object that contains the two arguments as parts. Given a pair, we can extract the parts using the primitive procedure `car` and `cdr`.

```lisp
(define x (cons 1 2))
(car x)
1
(cdr x)
2
```

Notice that a pair is a data object that can be given a name and manipulated, just like a primitive data object. Moreover, `cons` can be used to form pairs whose elements are pairs, and so on:

```lisp
(define x (cons 1 2))
(define y (cons 3 4))
(define z (cons x y))
(car (car z))
1
(car (cdr z))
3
```

Later we will see how this ability to combine pairs means that pairs can be used as general-purpose building blocks to create all sorts of complex data structures. The single compound-data primitive pair, implemented by the procedures `cons`, `car` and `cdr`, is the only glue we need. Data objects constructed from pairs are called `list-structured` data.

### Representing rational numbers

Pairs offer a natural way to complete the rational-number system. Simply represent a rational number as a pair of two integers: a numerator and a denominator. Then `make-rat`, `numer`, and `denom` are readily implemented as follows:

```lisp
(define (make-rat n d) (cons n d))
(define (numer x) (car x))
(define (denom x) (cdr x))
```

Also, in order to display the results of our computations, we can print rational numbers by printing the numerator, a slash, and the denominator:

```lisp
(define (print-rat x)
    (newline)
    (display (numer x))
    (display "/")
    (display (denom x)))
```

Now we can try our rational-number procedures:

```lisp
(define one-half (make-rat 1 2))
(print-rat one-half)
1/2
(define on-third (make-rat 1 3))
(print-rat (add-rat one-half one-third))
5/6
(print-rat (mul-rat one-half one-third))
1/6
(print-rat (add-rat one-third one-third))
6/9
```

As the final example shows, our rational-number implementation does not reduce rational numbers to lowest terms. We can remedy this by changing `make-rat`. If we have a `gcd` procedure that produces the greatest common divisor of two integers, we can use `gcd` to reduce the numerator and the denominator to lowest terms before constructing the pair:

```lisp
(define (make-rat n d)
    (let ((g (gcd n d)))
        (cons (/ n g) (/ d g))))
```

Now we have

```lisp
(print-rat (add-rat one-third one-third))
2/3
```

as desired. This modification was accomplished by changing the constructor `make-rat` without changing any of the procedures (such as `add-rat` and `mul-rat`) that implement the actual operations.

