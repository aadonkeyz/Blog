---
title: Building Abstractions with Data
categories:
  - 《 Structure and Interpretation of Computer Programs (Lisp) 》
abbrlink: 84aac73f
date: 2020-01-28 13:05:41
mathjax: true
---

# Introduction to Data Abstraction

We noted that a procedure used as an element in creating a more complex procedure could be regarded not only as a collection of particular operations but also a procedural abstraction. That is, the details of how the procedure was implemented could be suppressed, and the particular procedure itself could be replaced by any other procedure with the same overall behavior. In other words, we could make an abstraction that would separate the way the procedure would be used from the details of how the procedure would be implemented in terms of more primitive procedures. The analogous notion for compound data is called **data abstraction**. Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed from more primitive data objects.

The basic idea of data abstraction is to structure the programs that are to use compound data objects so that they operate on "abstract data." That is, our programs should use data in such a way as to make no assumptions about the data that are not strictly necessary for performing the task at hand. At the same time, a "concrete" data representation is defined independent of the programs that use the data. The interface between these two parts of our system will be a set of procedures, called **selectors** and **constructors**, that implement the abstract data in terms of the concrete representation.

## Example: Arithmetic Operations for Rational Numbers

Suppose we want to do arithmetic with rational numbers. We want to be able to add, subtract, multipy, and divide them and to test whether two rational numbers are equal.

Let us begin by assumping that we already have a way of constructing a rational number from a numerator and a denominator. We also assume that, given a rational number, we have a way of extracting (or selecting) its numerator and its denominator. Let us further assume that the constructor and selectors are available as procedures:

{% note info %}
- `(make-rat <n> <d>)` returns the rational number whose numerator is the integer `<n>` and whose denominator is the integer `<d>`
- `(numer <x>)` returns the numerator of the rational number `<x>`
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

## Abstraction Barriers

In general, the underlying idea of data abstraction is to identify for each type of data object a basic set of operations in terms of which all manipulations of data objects of that type will be expressed, and then to use only those operations in manipulating the data.

![Data-abstraction barriers in the rational-number package](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/Data-abstraction%20barriers%20in%20the%20rational-number%20package.png)

We can envision the structure of the rational-number system as shown above. The horizontal lines represent **abstraction barriers** that isolate different "levels" of the system. At each level, the barrier separates the programs (above) that use the data abstraction from the programs (below) that implement the data abstraction. Programs that use rational numbers manipulate them solely in terms of the procedures supplied "for public use" by the rational-number package: `add-rat`, `sub-rat`, `mul-rat`, `div-rat`, and `equal-rat?`. These, in turn, are implemented solely in terms of the constructor and selectores `make-rat`, `numer`, and `denom`, which themselves are implemented in terms of pairs. The details of how pairs are implemented are irrelevant to the rest of the rational-number package so long as pairs can be manipulated by the use of `cons`, `car`, and `cdr`. In effect, procedures at each level are the interfaces that define the abstraction barriers and connect the different levels.

This simple idea has many advantages. One advantage is that it makes programs much easier to maintain and to modify. Any complex data structure can be represented in a variety of ways with the primitive data structures provided by a programming language. Of course, the choice of representation influences the programs that operate on it; thus, if the representation were to be changed at some later time, all such programs might have to be modified accordingly. This task could be time-consuming and expensive in the case of large programs unless the dependence on the representation were to be confined by design to a very few program modules.

For example, an alternate way to address the problem of reducing rational numbers to lowest terms is to perform the reduction whenever we access the parts of a rational number, rather than when we construct it. This leads to different constructor and selector procedures:

```lisp
(define (make-rat n d) (cons n d))
(define (numer x)
    (let ((g (gcd (car x) (cdr x))))
        (/ (car x) g)))
(define (denom x)
    (let ((g (gcd (car x) (cdr x))))
        (/ (cdr x) g)))
```

The difference between this implementation and the previous one lies in when we compute the `gcd`. If in our typical use of rational numbers we access the numerators and denominators of the same rational numbers many times, it would be preferable to compute the `gcd` when the rational numbers are constructed. If not, we may be better off waiting until access time to compute the `gcd`. In any case, when we change from one representation to the other, the procedures `add-rat`, `sub-rat`, and so on do not have to be modified at all.

Constraining the dependence on the representation to a few interface procedures helps us design programs as well as modify them, because it allows us to maintain the flexibility to consider alternate implementations. To continue with our simple example, suppose we are designing a rational-number package and we can't decide initially whether to perform the `gcd` at construction time or at selection time. The data-abstraction methodology gives us a way to defer that decision without losing the ability to make progress on the rest of the system.

## What Is Meant by Data?

We began the rational-number implementation by implementing the rational-number operations `add-rat`, `sub-rat`, and so on in terms of three unspecified procedures: `make-rat`, `numer`, and `denom`. At that point, we could think of the operations as being defined in terms of data objects -- numerators, denominators, and rational numbers -- whose behavior was specified by the latter three procedures.

But exactly what is meant by data? It is not enough to say "whatever is implemented by the given selectors and constructors." Clearly, not every arbitary set of three procedures can serve as an appropriate basis for the rational-number implementation. We need to guarantee that, if we construct a rational number `x` from a pair of integers `n` and `d`, then extracting the `numer` and the `denom` of `x` and dividing them should yield the same result as dividing `n` by `d`. In other words, `make-rat`, `numer`, and `denom` must satisfy the condition that, for any integer `n` and any non-zero integer `d`, if `x` is `(make-rat n d)`, then

{% note info %}
<center>(numer $x$)/(denom $x$) = $n$/$d$</center>
{% endnote %}

In fact, this is the only condition `make-rat`, `numer`, and `denom` must fulfill in order to form a suitable basis for a rational-number representation. In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation.

This point of view can serve to define not only "high-level" data objects, such as rational numbers, but lower-level objects as well. Consider the notion of a pair, which we used in order to define our rational numbers. We never actually said what a pair was, only that the language supplied procedures `cons`, `car` and `cdr` for operations on pairs. But the only thing we need to know about these three operations is that if we glue two objects using `cons` we can retrieve the objects using `car` and `cdr`. That is, the operations satisfy the condition that, for any objects `x` and `y`, if `z` is `(cons x y)` then `(car z)` is x and `(cdr z)` is `y`.

Indeed, we mentioned that these three procedures are included as primitives in our language. However, any triple of procedures that satisfies the above condition can be used as the basis for implementing pairs. This point is illustrated strikingly by the fact that we could implement `cons`, `car` and `cdr` without using any data structures at all but only using procedures. Here are the definitions:

```lisp
(define (cons x y)
    (define (dispatch m)
        (cond ((= m 0) x)
              ((= m 1) y)
              (else (error "Argument not 0 or 1: CONS" m))))
    dispatch)
(define (car z) (z 0))
(define (cdr z) (z 1))
```

This use of procedures corresponds to nothing like our intuitive notion of what data should be. Nevertheless, all we need to do to show that this is a valid way to represent pairs is to verify that these procedures satisfy the condition given above.

The point of exhibiting the procedural representation of pairs is not that our language works this way (Scheme, and Lisp systems in general, implement pairs directly, for efficiency reasons) but that it could work this way. The procedural representation, althougn obscure, is a perfectly adequate way to represent pairs, since it fulfills the only conditions that pairs need to fulfill. This example also demonstrates that the ability to manipulate procedures as objects automatically provides the ability to represent compound data.

# Hierarchical Data and the Closure Property

As we have seen, pairs provide a primitive "glue" that we can use to construct compound data objects. The picutre shows a standard way to visualize a pair -- in this case, the pair formed by `(cons 1 2)`. In this representation, which is called **box-and-pointer notation**, each object is shown as a pointer to a box. The box for a primitive object contains a representation of the object. For example, the box for a number contains a numeral. The box for a pair is actually a double box, the left part containing (a pointer to) the `car` of the pair and the right part containing the `cdr`.

![box-and-pointer](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/box-and-pointer.png)

The ability to create pairs whose elements are pairs is the essence of list structure's importance as a representation tool. We refer to this ability as the **closure property** of `cons`. In general, an operation for combining data objects satisfies the closure property if the results of combining things with that operation can themselves be combined using the same operation. Closure is the key to power in any means of combination because it permits us to create **hiearchical structures** -- structures made up of parts, which themselves are made up of parts, and so on.

{% note warning %}
The use of the word "closure" here comes from abstract algebra, where a set of elements is said to be closed under an operation if applying the operation to elements in the set produces an element that is again an element of the set. The Lisp community also (unfortunately) uses the word "closure" to describe a totally unrelated concept: A closure is an implementation technique for representing procedures with free variables. We do not use the world "closure" in this second sense in this book.
{% endnote %}

## Representing Sequences

One of the useful structures we can build with paires is a sequence -- an ordered collection of data objects. There are, of course, many ways to represent sequences in term of pairs. One particularly straightforward representation is illustrated in the picture, where the sequence `1, 2, 3, 4` is represented as a chain of pairs. 

![The sequence 1, 2, 3, 4 represented as a chain
of pairs.](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/The%20sequence%201%2C%202%2C%203%2C%204%20represented%20as%20a%20chain%20of%20pairs.png)

The `car` of each pair is the corresponding item in the chain, and the `cdr` of the pair is the next pair in the chain. The `cdr` of the final pair signals the end of the sequence by pointing to a distinguished value that is not a pair, represented in box-and-pointer diagrams as a diagonal line and in programs as the value of the variable `nil`. The entrie sequence is constructed by nested `cons` operations:

```lisp
(cons 1
      (cons 2
            (cons 3
                  (cons 4 nil))))
```

Such a sequence of pairs, formed by nested conses, is called a list, and Scheme provides a primitive called `list` to help in constructing lists. The above sequence could be produced by `(list 1 2 3 4)`. In general,

```lisp
(list <a1> <a2> ... <an>)
```

is equivalent to

```lisp
(cons <a1>
      (cons <a2>
            (cons ...
                  (cons <an>
                        nil)...)))
```

### List operations

The use of pairs to represent sequences of elements as lists is accompanied by conventional programming techniques for manipulating lists by successively "cdring down" the lists. For example, the procedure `list-ref` takes as arguments a list and a number `n` and returns the $n^{th}$ item of the list. It is customary to number the elements of the list beginning with 0. The method for computing `list-ref` is the following:

```lisp
(define (list-ref items n)
    (if (= n 0)
        (car items)
        (list-ref (cdr items) (- n 1))))

(define squares (list 1 4 9 16 25))
(list-ref squares 3)
16
```

Often we `cdr` down the whole list. To aid in this, Scheme includes a primitive predicate `null?`, which tests whether its argument is the empty list. The procedure `length`, which returns the number of items in a list, illustrate this typical pattern of use:

```lisp
(define (length items)
    (if (null? items)
        0
        (+ 1 (length (cdr items)))))

(define odds (list 1 3 5 7))
(length odds)
4
```

We could also compute `length` in an iterative style:

```lisp
(define (lenght items)
    (define (length-iter a count)
        (if (null? a)
            count
            (length-iter (cdr a) (+ 1 count))))
    (length-iter items 0))
```

Another conventional programming technique is to "cons up" an answer list while cdring down a list, as in the procedure `append`, which takes two lists as arguments and combinds their elements to make a new list:

```lisp
(define (append list1 list2)
    (if (null? list1)
        list2)
        ((cons (car list1) (append (cdr list1) list2))))

(append squares odds)
(1 4 9 16 25 1 3 5 7)
(appen odds squares)
(1 3 5 7 1 4 9 16 25)
```

### Mapping over lists

One extremely useful operation is to apply some transfomation to each element in a list and generate the list of results. For instance, the following procedure scales each number in a list by a given factor:

```lisp
(define (scale-list items factor)
    (if (null? items)
        nil
        (cons (* (car items) factor)
              (scale-list (cdr items)
                          factor))))

(scale-list (list 1 2 3 4 5) 10)
(10 20 30 40 50)
```

We can abstract this general and capture it as a common pattern expressed as a higher-order procedure. The higher-order procedure here is called `map`. `map` takes as arguments a procedure of one argument and a list, and returns a list of the results produced by applying the procedure to each element in the list:

```lisp
(define (map proc items)
    (if (null? items)
        nil
        (cons (proc (car items))
              (map proc (cdr items)))))

(map abs (list -10 2.5 -11.6 17))
(10 2.5 11.6 17)
```

Now we can give a new definition of `scale-list` in terms of map:

```lisp
(define (scale-list items factor)
    (map (lambda (x) (* x factor))
         items))
```

`map` is an important construct, not only because it captures a common pattern, but because it establishes a higher level of abstraction in dealing with lists. In the original definition of `scale-list`, the recursive structure of the program draws attention to the element-by-element processing of the list. Defining `scale-list` in terms of `map` supresses that level of detail and emphasizes that scaling transforms a list of elements to a list of results. The difference between the two definitions is not that the computer is performing a different process (it isn't) but that we think about the process differently. In effect, `map` helps establish an abstraction barrier that isolates the implementation of procedures that transform lists from the details of how the elements of the list are extracted and combined. Like the barriers shown previously, this abstraction gives us the flexibility to change the low-level details of how sequences are implemented, while preserving the conceptual framework of operations that transform sequenecs to sequences.

## Hierarchical Structures

The representation of sequences in terms of lists generalizes naturally to represent sequences whose elements may themselves be sequences. For example, we can regard the object `((1, 2), 3, 4)` constructed by `(cons (list 1 2) (list 3 4))` as a list of three items, the first of which is itself a list. Indeed, this is suggested by the form in which the result is printed by the interpreter. The picture shows the representation of this structure in terms of pairs.

![Structure formed by (cons (list 1 2) (list 3 4))](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/Structure%20formed%20by%20%28cons%20%28list%201%202%29%20%28list%203%204%29%29.png)

Another way to think of sequences whose elements are sequences is as tree. The elements of the sequences are the branches of the tree, and elements that are themselves sequences are subtrees.

![The list structure viewed as a tree](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/The%20list%20structure%20viewed%20as%20a%20tree.png)

Recursion is a natural tool for dealing with tree structures, since we can often reduce operations on trees to operations on their branches, which reduce in turn to operations on the branches of the branches, and so on, until we reach the leaves of the tree. As an example, compare the `length` procedure with the `count-leaves` procedure, which returns the total number of leaves of a tree:

```lisp
(define (count-leaves x)
    (cond ((null? x) 0)
          ((not (pair? x)) 1)
          (else (+ (count-leaves (car x))
                   (count-leaves (cdr x))))))

(define x (cons (list 1 2) (list 3 4)))
(length x)
3
(count-leaves x)
4
```

### Mapping over trees

Just as `map` is a powerful abstraction for dealing with sequences, `map` together with recursion is a powerful abstraction for dealing with trees. For instance, the `scale-tree` procedure, analogous to `scale-list`, takes as arguments a numeric factor and a tree whose leaves are numbers. It returns a tree of the same shape, where each number is multiplied by the factor. The recursive plan for `scale-tree` is similar to the one for `count-leaves`:

```lisp
(define (scale-tree tree factor)
    (cond ((null? tree) nil)
          ((not (pair? tree)) (* tree factor))
          (else (cons (scale-tree (car tree) factor)
                      (scale-tree (cdr tree) factor)))))

(scale-tree (list 1 (list 2 (list 3 4) 5) (list 6 7)) 10)
(10 (20 (30 40) 50) (60 70))
```

Another way to implement `scale-tree` is to regard the tree as a sequence of sub-trees and use `map`. We map over the sequence, scaling each sub-tree in turn, and return the list of the results. In the base case, where the tree is a leaf, we simply multiply by the factor:

```lisp
(define (scale-tree tree factor)
    (map (lambda (sub-tree)
            （if (pair? sub-tree)
                 (scale-tree sub-tree factor)
                 (* sub-tree factor)))
         tree)
```

## Sequences as Conventional Interfaces

In working with compoind data, we've stressed how data abstraction permits us to design programs without becoming enmeshed in the details of data representations, and how abstraction preserves for us the flexibility to experiment with alternative representations. In this section, we introduce another powerful design principle for working with data structures -- the use of **conventional interfaces**.

We havs seen how program abstractions, implemented as higher-order procedures, can capture common patterns in programs that deal with numerical data. Our ability to formulate analogous operations for working with compound data depends crucially on the style in which we manipulate our data structures. Consider, for example, the following procedure, analogous to the `count-leaves` procedure:

```lisp
(define (sum-odd-squares tree)
    (cond ((null? tree) 0)
          ((not (pair? tree)) (if (odd? tree) (square tree) 0))
          (else (+ (sum-odd-squares (car tree))
                   (sum-odd-squares (cdr tree))))))
```

On the surface, this procedure is very different from the following one, which constructs a list of all the even Fibonacci numbers Fib($k$), where $k$ is less than or equal to a given integer $n$:

```lisp
(define (even-fibs n)
    (define (next k)
        (if (> k n)
            nil
            (let ((f (fib k)))
                (if (even? f)
                    (cons f (next (+ k 1)))
                    (next (+ k 1))))))
    (next 0))
```

Despite the fact that these two procedures are structurally very different, a more abstract description of the two computations reveals a great deal of similarity. 

{% note info %}
The first program:
1. enumerates the leaves of a tree;
2. filters them, selecting the odd ones;
3. squares each of the selected ones;
4. accumulates the results using `+`, starting with 0.

---
The second program:
1. enumeratres the integers from 0 to $n$;
2. computes the Fibonacci number for each integer;
3. filters them, selecting the even ones;
4. accumulates the results using `cons`, starting with the empty list.
{% endnote %}

A signal-processing engineer would find it natural to conceptualize these processes in terms of signals flowing through a cascade of stages, each of which implements part of the program plan, as shown in the following picture.

![signal-flow](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/signal-flow.png)

In `sum-odd-squares`, we begin with an `enumerator`, which generates a "signal" consisting of the leaves of a given tree. This signal is passed through a `filter`, which eliminates all but the odd elements. The resulting signal is in turn passed through a `map`, which is a "transducer" that applies the square procedure to each element. The output of the map is then fed to an `accumulator`, which combines the elements using `+`, starting from an initial 0. The plan for `even-fibs` is analogous.

Unfortunately, the two procedure definitions above fail to exhibit this signal-flow structure. For instance, if we examine the `sum-odd-squares` procedure, we find that the enumeration is implemented partly by the `null?` and `pair?` tests and partly by the tree-recursive structure of the procedure. Similarly, the accumulation is found partly in the tests and partly in the addition used in the recrusion. In general, there are no distinct parts of either procedure that correspond to the elements in the signal-flow description. Our two procedures decompose the computations in a different way, spreading the enumeration over the program and mingling it with the map, the filter, and the accumulation. If we could organize our programs to make the signal-flow structure manifest in the procedure we write, this would increase the conceptual clarity of the resulting code.

The key to organizing programs so as to more clearly reflect the signal-flow structure is to concentrate on the "signals" that flow from one stage in the process to the next. If we represent these signals as lists, then we can use list operations to implement the processing at each of the stages. For instance, we can implement the mapping stages of the signal-flow diagrams using the `map` procedure.

```lisp
(map square (list 1 2 3 4 5))
(1 4 9 16 25)
```

Filtering a sequence to select only those elements that satisfy a given predicate is accomplished by

```lisp
(define (filter predicate sequence)
    (cond ((null? sequence) nil)
          ((predicate (car sequence))
            (cons (car sequence) (filter predicate (cdr sequence))))
          (else (filter predicate (cdr sequence)))))

(filter odd? (list 1 2 3 4 5))
(1 3 5)
```

Accumulations can be implemented by

```lisp
(define (accumulate op initial sequence)
    (if (null? sequence)
        initial
        (op (car sequence)
            (accumulate op initial (cdr sequence)))))

(accumulate + 0 (list 1 2 3 4 5))
15
```

All that remains to implement signal-flow diagrams is to enumerate the sequence of elements to be processed. For `even-fibs`, we need to generate the sequence of integers in a given range, which we can do as follows:

```lisp
(define (enumerate-interval low high)
    (if (> low high)
        nil
        (cons low (enumerate-interval (+ low 1) high))))

(enumerate-interval 2 7)
(2 3 4 5 6 7)
```

To enumerate the leaves of a tree, we can use:

```lisp
(define (enumerate-tree tree)
    (cond ((null? tree) nil
          ((not (pair? tree)) (list tree))
          (else (append (enumerate-tree (car tree))
                        (enumerate-tree (cdr tree)))))))

(enumerate-tree (list 1 (list 2 (list 3 4)) 5))
(1 2 3 4 5)
```

Now we can reformulate `sum-odd-squares` and `even-fibs` as in the signal-flow diagrams.

```lisp
(define (sum-odd-squares tree)
    (accumulate
        +
        0
        (map square (filter odd? (enumerate-tree tree)))))

(define (even-fibs n)
    (accumulate
        cons
        nil
        (filter even? (map fib (enumerate-interval 0 n)))))
```

The value of expressing programs as sequence operations is that this helps us make program designs that are modular, that is, designs that are constructed by combining relatively independent pieces. We can encourage modular design by providing a library of standard components together with a conventional interface for connecting the components in flexible ways.

Modular construction is a powerful strategy for controlling complexity in engineering design. In real signal-processing applications, for example, designers regularly build systems by cascading elements selected from standardized families of filters and transducers. Similarly, sequence operations provide a library of standard program elements that we can mix and match. For instance, we can reuse pieces from the `sum-odd-squares` and `even-fibs` procedures in a program that constructs a list of the sequence of the first $n+1$ Fibonacci numbers:

```lisp
(define (list-fib-squares n)
    (accumulate
        cons
        nil
        (map square (map fib (enumerate-interval 0 n)))))

(list-fib-squares 10)
(0 1 1 4 9 25 64 169 441 1156 3025)
```

# Symbolic Data

## Quotation

In order to manipulate symbols we need a new element in our language: the ability to **quote** a data object. Suppose we want to construct  the list `(a, b)`. We can't accomplish this with `(list a b)`, because this expression constructs a list of the values of `a` and `b` rather than the symbols themselves. This issue is well known in the context of natural languages, where words and sentences may be regarded either as semantic entities or as character strings (syntactic entities). The common practice in natural languages is to use quotation marks to indicate that a word or a sentence is to be treated literally as a string of characters. For instance, if we tell somebody "say your name aloud," we expect to hear that person's name. However, if we tell somebody "say 'your name' aloud," we expect to hear the words "your name." Note that we are forced to nest quotation marks to describe what somebody else might say.

We can follow this same practice to identify lists and symbols that are to be treated as data objects rather than as expressions to be evaluated. However, our format for quoting differs from that of natural languages in that we place a quotation mark only at the beginning of the object to be quoted. We can get away with this in Scheme syntax because we rely on blanks and parentheses to be delimit objects. Thus, the meaning of the single quote character is to quote the next object.

Now we can distinguish between symbols and their values:

```lisp
(define a 1)
(define b 2)
(list a b)
(1 2)
(list 'a 'b)
(a b)
(list 'a b)
(a 2)
(car '(a b c))
a
(cdr '(a b c))
(b c)
```

One additional primitive used in manipulating symbols is `eq?`, which takes two symbols as arguments and tests whether they are the same. Here is a example:

```lisp
(define (memq item x)
    (cond ((null? x) false)
          ((eq? item (car x)) x
          (else (memq item (cdr x))))))
(memq 'apple '(pear banana prune))
false
(memq 'apple '(x (apple sauce) y apple pear)
(apple pear))
```

## Example: Symbolic Differentiation

As an illustration of symbol manipulation and a further illustration of data abstraction, consider the design of a procedure that performs symbolic differentiation of algebraic expressions. We would like the procedure to take as arguments an algebraic expression and a variable and to return the derivative of the expression with respect to the variable.

In developing the symbolic-differentiation program, we will first define a differentiation algorithm that operates on abstract objects such as "sum", "products", and "variables" without worrying about how these are to be represented. Only afterward will address the representation problem.

```lisp
(variable? e)           ; Is e a variable?
(same-variable? v1 v2)  ; Are v1 and v2 the same variable?
(sum? e)                ; Is e a sum?
(addend e)              ; Addend of the sum e.
(augend e)              ; Augend of the sum e.
(make-sum a1 a2)        ; Construct the sum of a1 and a2.
(product? e)            ; Is e a product?
(multiplier e)          ; Multiplier of the product e.
(multiplicand e)        ; Multiplicand of the product e.
(make-product m1 m2)    ; Construct the product of m1 and m2.

(define (deriv exp var)
    (cond ((number? exp) 0)
          ((variable? exp) (if (same-variable? exp var) 1 0))
          ((sum? exp) (make-sum (deriv (addend exp) var)
                                (deriv (augend exp) var)))
          ((product? exp)
            (make-sum
                (make-product (multiplier exp)
                              (deriv (multiplicand exp) var))
                (make-product (deriv (multiplier exp) var)
                              (multiplicand exp))))
          (else (error "unknown expression type: DERIV" exp))))
```

The `deriv` procedure incorporates the complete differentiation algorightm. Since it is expressed in terms of abstract data, it will work no matter how we choose to represent algebraic expressions, as long as we design a proper set of selectors and constructors.

## Example: Representing Sets

Informally, a set is simply a collection of distinct objects. To give a more precise definition we can employ the method of data abstraction. That is, we define "set" by specifying the operations that are to be used on sets. These are `union-set`, `intersection-set`, `element-of-set?`, and `adjoin-set`. `element-of-set?` is a predicate that determines whether a given element is a member of a set. `adjoin-set` takes an object and a set as arguments and returns a set that contains the elements of the original set and also the adjoined element. `union-set` computes the union of two sets, which is the set containing each element that appears in either argument. `intersection-set` computes the intersection of two sets, which is the set containing only elements that appear in both arguments. From the viewpoint of data abstraction, we are free to design any representation that implements these operations in a way consistent with the interpretations given above.

### Sets as unordered lists

One way to represent a set is as a list of its elements in which no element appears more than once. The empty set is represented by the empty list. In this representation, `element-of-set?` is similar to the procedure `memq`. It uses `equal?` instead of `eq?` so that the set elements need not be symbols:

```lisp
(define (element-of-set? x set)
    (cond ((null? set) false)
          ((equal? x (car set)) true)
          (else (element-of-set? x (cdr set)))))
```

Using this, we can write `adjoin-set`. If the object to be adjoined is already in the set, we just return the set. Otherwise, we use `cons` to add the object to the list that represents the set:

```lisp
(define (adjoin-set x set)
    (if (element-of-set? x set)
        set
        (cons x set)))
```

For `intersection-set` we can use a recursive strategy. If we know how to form the intersection of `set2` and the `cdr` of `set1`, we only need to decide whether to include the `car` of `set` in this. But this depends on whether `(car set1)` is also in `set2`. Here is the resulting procedure:

```lisp
(define (intersection-set set1 set2)
    (cond ((or (null? set1) (null? set2)) '())
          ((element-of-set? (car set1) set2)
            (cons (car set1) (intersection-set (cdr set1) set2)))
          (else (intersection-set (cdr set1) set2))))
```

In designing a representation, one of the issues we should be concerned with is efficiency. Consider the number of steps required by our set operations. Since they all use `element-of-set?`, the speed of this operation has a major impact on the efficiency of the set implementation as a whole. Now, in order to check whether an object is a member of a set, `element-of-set?` may have to scan the entire set. (In the worst case, the object turns out not to be in the set.) Hence, if the set has $n$ elements, `element-of-set?` might take up to $n$ steps. Thus, the number of steps required grows as $\Theta(n)$. The number of steps required by `adjoin-set`, which uses this operation, also grows as $\Theta(n)$. For `intersection-set`, which does an `element-of-set?` check for each element of `set1`, the number of steps required grows as the product of the sizes of the sets involved, or $\Theta(n^2)$ for two sets of size $n$. The same will be true of `union-set`.

### Sets as ordered lists

One way to speed up our set operations is to change the representation so that the set elements are listed in increasing order. To do this, we need some way to compare two objects so that we can say which is bigger. For example, we could compare symbols lexicographically, or we could agree on some method for assigning a unique number to an object and then compare the elements by comparing the corresponding numbers. To keep our discussion simple, we will consider only the case where the set elements are numbers, so that we can compare elements using $>$ and $<$. We will represent a set of numbers by listing its elements in increasing order.

One advantage of ordering shows up in `element-of-set?`: In checking for the presence of an item, we no longer have to scan the entire set. If we reach a set element that is larger than the item we are looking for, then we know that the item is not in the set.

```lisp
(define (element-of-set? x set)
    (cond ((null? set) false)
          ((= x (car set)) true)
          ((< x (car set)) false)
          (else (element-of-set? x (cdr set)))))
```

How many steps does this save? In the worst case, the item we are looking for may be the largest one in the set, so the number of steps is the same as for the unordered representation. On the other hand, if we search for items of many different sizes we can expect that somtimes we will be able to stop searching at a point near the beginning of the list and that other times we will still need to examine most of the list. On the average we should expect to have to examine about half of the items in the set. Thus, the average number of steps required will be about $n/2$. This is still $\Theta(n)$ growth, but it does save us, on the average, a factor of 2 in number of steps over the previous implementation.

We obtain a more impressive speedup with `intersection-seet`. In the unordered representation this operation required $\Theta(n^2)$ steps, because we performed a complete scan of `set2` for each element of `set1`. But with the ordered representation, we can use a more clever method. Begin by comparing the initial elements, `x1` and `x2`, of the two sets. If `x1` equals `x2`, then that gives an element of the intersection, and the rest of the intersection is the intersection of the `cdr` of the two sets. Suppose, however, that `x1` is less than `x2`. Since `x2` is the smallest element in `set2`, we can immediately conclude that `x1` cannot appear anywhere in `set2` and hence is not in the intersection. Hence, the intersection is equal to the intersection of `set2` with the `cdr` of `set1`. Similarly, if `x2` is less than `x1`, then the intersection is given by the intersection of `set1` with the `cdr` of `set2`.

```lisp
(define (intersection-set set1 set2)
    (if (or (null? set1) (null? set2))
        '()
        (let ((x1 (car set1)) (x2 (car set2)))
            (cond ((= x1 x2)
                    (cons x1 (intersection-set (cdr set1)
                                               (cdr set2))))
                  ((< x1 x2)
                    (intersection-set (cdr set1) set2))
                  ((< x2 x1)
                    (intersection-set set1 (cdr set2)))))))
```

To estimate the number of steps required by this process, observe that at each step we reduce the intersection problem to computing intersection of smaller sets-removing the first element from `set1` or `set2` or both. Thus, the number of steps required is at most the sum of the sizes of `set1` and `set2`, rather than the product of the sizes as with the unordered representation. This is $\Theta(n)$ growth rather than $\Theta(n^2)$ -- a considerable speedup, even for sets of moderate size.

### Sets as binary trees

We can do better than the ordered-list representation by arranging the set elements in the form of a tree. Each node of the tree holds one element of the set, called the "entry" at that node, and a link to each of two other (possibly empty) nodes. The "left" link points to elements smaller than the one at the node, and the "right" link to elements greater than the one at the node. The same set may be represented by a tree in a number of different ways. The only thing we require fro a valid representation is that all elements in the left subtree be smaller than the node entry and that all elements in the right subtree be larger.

The advantage of the tree representation is this: Suppose we want to check whether a number $x$ is contained in a set. We begin by comparing $x$ with the entry in the top node. If $x$ is greater, we need only search the right subtree. Now, if the tree is "balanced," each of these subtrees will be about half the size of the original. Thus, in one step we have reduced the problem of searching a tree of size $n$ to searching a tree of size $n/2$. Since the size of the tree is halved at each step, we should expect that the number of steps needed to search a tree of size $n$ grows as $\Theta(logn)$. For large sets, this will be a significant speedup over the previous representations.

We can represent trees by using lists. Each node will be a list of three items: the entry at the node, the left subtree, and the right subtree. A left or a right subtree of the empty list will indicate that there is no subtree connected there.

```lisp
(define (entry tree) (car tree))
(define (left-branch tree) (cadr tree))
(define (right-branch tree) (caddr tree))
(define (make-tree entry left right) 
    (list entry left right))
```

Now we can write the `element-of-set?` procedure using the strategy describe above：

```lisp
(define (element-of-set? x set)
    (cond ((null? set) false)
          ((= x (entry set)) true)
          ((< x (entry set))
            (element-of-set? x (left-branch set)))
          ((> x (entry set))
            (element-of-set? x (right-branch set)))))
```

Adjoining an item to a set is implemented similarly and also requires $\Theta(logn)$ steps. To adjoin an item $x$, we compare $x$ with the node entry to determine whether $x$ should be added to the right or to the left branch, and having adjoined $x$ to the appropriate branch we piece this newly constructed branch together with the original entry and the other branch. If $x$ is equal to the entry, we just return the node. If we are asked to adjoin $x$ to an empty tree, we generate a tree that has $x$ as the entry and empty right and left branches.

```lisp
(define (adjoin-set x set)
    (cond ((null? set) (make-tree x '() '()))
          ((= x (entry set)) set)
          ((< x (entry set))
            (make-tree (entry set)
                       (adjoin-set x (left-branch set))
                       (right-branch set)))
          ((> x (entry set))
            (make-tree (entry set) 
                       (left-branch set)
                       (adjoin-set x (right-branch set))))))
```

The above claim that searching the tree can be performed in a logarithmic number of steps rests on the assumption that the tree is "balanced", i.e., that the left and the right subtree of every tree have approximately the same number of elements, so that each subtree contains about half the elements of its parent. But how can we be certain that the trees we construct will be balanced? Even if we start with a balanced tree, adding elements with `adjoin-set` may produce an unbalanced result. One way to solve this problem is to define an operation that transforms an arbitrary tree into a balanced tree with the same elements. Then we can perform this transformation after every few `adjoin-set` operations to keep our set in balance. There are also other ways to solve this problem, most of which involve designing new data structures for which searching and insertion both can be done in $\Theta(logn)$ steps.

### Sets and information retrieval

Consider a data base containing a large number of individual records, such as the personnel files for a company or the transactions in an accounting system. A typical data-management system spends a large amount of time accessing or modifying the data in the records and therefore requires an efficient method for accessing records. This is done by identifying a part of each record to serve as an identifying **key**. A key can be anything that uniquely identifies the record. Whatever the key is, when we define the record as a data structure we should include a key selector procedure that retrieves the key associated with a given record.

Now we represent the data base as a set of records. To locate the record with a given key we use a procedure `lookup`, which takes as arguments a key and a data base and which returns the record that has that key, or false if there is no such record. `lookup` is implemented in almost the same way as `element-of-set?`. For example, if the set of records is implemented as an unordered list, we could use

```lisp
(define (lookup given-key set-of-records)
    (cond ((null? set-of-records) false)
          ((equal? given-key (key (car set-of-records)))
            (car set-of-records))
          (else (lookup given-key (cdr set-of-records)))))
```

Of course, there are better ways to represent large sets than as unordered lists. Information-retrieval systems in which records have to be "randomly accessed" are typically implemented by a tree-based method, such as the binary-tree representation discussed previously. In designing such a system the methodology of data abstraction can be a great help. The designer can create an initial implementation using a simple, straighforward representation such as unordered lists. This will be unsuitable for the eventual system, but it can be useful in providing a "quick and dirty" data base with which to test the rest of the system. Later on, the data representation can be modified to be more sophisticated. If the data base is accessed in terms of abstract selectors and constructors, this change in representation will not require any changes to the rest of the system.

## Example: Huffman Encoding Trees

This section provides practice in the use of list structure and data abstraction to manipulate sets and trees. The application is to methods for representing data as sequences of one and zeros (bits). For example, the ASCII standard code used to represent text in computers encodes each character as a sequence of seven bits. Using seven bits allows us to distinguish $2^7$, or 128, possible different characters. In general, if we want to distinguish $n$ different symbols, we will need to use $log_2n$ bits per symbol. If all our messages are made up of the eight symbols A, B, C, D, E, F, G, and H, we can choose a code with three bits per character, for example

A(000)、B(001)、C(010)、D(011)、E(100)、F(101)、G(110)、H(111)

With this code, the message `BACADAEAFABBAAAGAH` is encoded as the string of 54 bits

`001000010000011000100000101000001001000000000110000111`

Codes such as ASCII and the A-through-H code above are known as **fixed-length** codes, because they represent each symbol in the message with the same number of bits. It is sometimes advantageous to use **variable-length** codes, in which different symbols may be represented by different numbers of bits. For example, Morse code does not use the same number of dots and dashes for each letter of the alphabet. In particular, E, the most frequent letter, is represented by a single dot. In general, if our messages are such that some symbols appera very frequently and some very rarely, we can encode data more efficiently if we assign shorter codes to the frequent symbols. Consider the following alternative code for the letters A through H:

A(0)、B(100)、C(1010)、D(1011)、E(1100)、F(1101)、G(1110)、H(1111)

With this code, the same message as above is encoded as the string

`100010100101101100011010100100000111001111`

This string contains 42 bits, so it saves more than 20% in space in comparison with the fixed-length code shown above.

One of the difficulties of using a variable-length code is knowing when you have reached the end of a symbol in reading a sequence of zeros and ones. Morse code solves this problem by using a **special separator code** after the sequence of dots and dashes for each letter. Another solution is to design the code in such a way that no complete code for any symbol is the beginning of the code for another symbol. Such a code is called a **prefix code**. In the example above, `A` is encoded by `0` and `B` is encoded by `100`, so no other symbol can have a code that begins with `0` or with `100`.

In general, we can attain significant saving if we use variable-length prefix codes that take advantage of the relative frequencies of the symbols in the messages to be encoded. One particular scheme for doing this is called the Huffman encoding method, after its discoverer, David Huffman. A Huffman code can be represented as a binary tree whose leaves are the symbols that are encoded. At each non-leaf node of the tree there is a set containing all the symbols in the leaves that lie below the node. In addition, each symbol at a leaf is assigned a weight (which is its relative frequency), and each non-leaf node contains a weight that is the sum of all the weights of the leaves lying below it. The weights are not used in the encoding or the decoding process. We will see below how they are used to help construct the tree.

![A Huffman encoding tree](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/A%20Huffman%20encoding%20tree.png)

Given a Huffman tree, we can find the encoding of any symbol by starting at the root and moving down until we reach the leaf that holds the symbol. Each time we move down a left branch we add a $0$ to the code, and each time we move down a right branch we add a $1$. For example, staring from the root of the tree, we arrive at the leaf for D by following a right branch, then a left branch, then a right branch, then a right branch; hence, the code for D is `1011`.

To decode a bit sequence using a Huffman tree, we begin at the root and use the successive zeros and ones of the bit sequence to determine whether to move down the left or the right branch. Each time we come to a leaf, we have generated a new symbol in the message, at which point we start over from the root of the tree to find the next symbol. For example, suppose we are given the tree above and the sequence `10001010`, the entire message is BAC.

### Generating Huffman trees

Given an "alphabet" of symbols and their relative frequencies, how do we construct the "best" code? (In other words, which tree will encode messages with the fewest bits?) Huffman gave an algorithm for doing this and showed that the resulting code is indeed the best variable-length code for messages where the relative frequency of the symbols matches the frequencies with which the code was constructed. We will not prove this optimality of Huffman codes here, but we will show how Huffman trees are constructed.

The algorithm for generating a Huffman tree is very simple. The idea is to arrange the tree so that the symbols with the lowest frequency appear farthest away from the root. Begin with the set of leaf nodes, containing symbols and their frequencies, as determined by the initial data from which the code is to be constructed. Now find two leaves with the lowest weights and merge them to produce a node that has these two nodes as its left and right branches. The weight of the new node is sum of the two weights. Remove the two leaves from the original set and replace them by this new node. Now continue this process. At each step, merge two nodes with the smallest weights, removeing them from the set and replacing them with a node that has these two as its left and right branches. The process stops when there is only one node left, which is the root of the entrie tree. The algorithm does not always specify a unique tree, beacuse there may not be unique smallest-weight nodes at each step. Also, the choice of the order in which the two nodes are merged is arbitrary.

### Representing Huffman trees

In the exercise below we will work with a system that uses Huffman trees to encode and decode messages and generates Huffman trees according to the algorithm outlined above. We will begin by discussing how trees are represented.

Leaves of the tree are represented by a list consisting of the symbol leaf, the symbol at the leaf, and the weight:

```lisp
(define (make-leaf symbol weight) (list 'leaf symbol weight))
(define (leaf? object) (eq? (car object) 'leaf))
(define (symbol-leaf x) (cadr x))
(define (weight-leaf x) (caddr x))
```

A generate tree will be a list of a left branch, a right branch, a set of symbols, and a weight. The set of symbols will be simply a list of the symbols, rather than some more sophisticated set representation. When we make a tree by merging two nodes, we obtain the weight of the tree as the sum of the weights of the nodes, and the set of symbols as the union of the sets of symbols for the nodes. Since our symbol sets are represented as lists, we can form the union by using the `append` procedure.

```lisp
(define (make-code-tree left right)
    (list left
          right
          (append (symbols left) (symbols right))
          (+ (weight left) (weight right))))
```

If we make a tree in this way, we have the following selectors:

```lisp
(define (left-branch tree) (car tree))
(define (right-branch tree) (cadr tree))
(define (symbols tree)
    (if (leaf? tree)
        (list (symbol-leaf tree))
        (caddr tree)))
(define (weight tree)
    (if (leaf? tree)
        (weight-leaf tree)
        (cadddr tree)))
```

The procedures `symbols` and `weight` must do something slightly different depending on whether they are called with a leaf of a general tree. These are simple examples of **generic procedures** (procedures that can handle more than one kind of data)

### The decoding procedure

The following procedure implements the decoding algorithm. It takes as arguments a list of zeros and ones, together with a Huffman tree.

```lisp
(define (decode bits tree)
    (define (decode-1 bits current-branch)
        (if (null? bits)
            '()
            (let ((next-branch
                    (choose-branch (car bits) current-branch)))
                (if (leaf? next-branch)
                    (cons (symbol-leaf next-branch)
                          (decode-1 (cdr bits) tree))
                    (decode-1 (cdr bits) next-branch)))))
    (decode-1 bits tree))
(define (choose-branch bit branch)
    (cond ((= bit 0) (left-branch branch))
          ((= bit 1) (right-branch branch))
          (else (error "bad bit: CHOOSE-BRANCH" bit))))
```

The procedure `decode-1` takes two arguments: the list of remaining bits and the current position in the tree. It keeps moving "down" the tree, choosing a left or a right branch according to whether the next bit in the list is a zero or a one. When it reaches a leaf, it returns the symbol at that leaf as the next symbol in the message by consing it onto the result of the decoding the rest of the message, starting at the root of the tree. Note the error check in the final clause of `choose-branch`, which complains if the procedure finds something other than a zero or a one in the input data.

### Sets of weighted elements

In our representation of trees, each non-leaf node contains a set of symbols, which we have represented as a simple list. However, the tree-generating algorithm discussed above requires that we also work with sets of leaves and trees, successively merging the two smallest items. Since we will be required to repeatedly find the smallest item in a set, it is convenient to use an ordered representation for this kind of set.

We will represent a set of leaves and trees as a list of elements, arrange in increasing order of weight. 

```lisp
(define (adjoin-set x set)
    (cond ((null? set) (list x))
          ((< (weight x) (weight (car set))) (cons x set))
          (else (cons (car set)
                      (adjoin-set x (cdr set))))))
```

The following procedure takes a list of symbol-frequency pairs such as ((A 4) (B 2) (C 1) (D 1)) and constructs an initial ordered set of leaves, ready to be merged according to the Huffman algorithm:

```lisp
(define (make-leaf-set pairs)
    (if (null? pairs)
        '()
        (let ((pair (car pairs)))
            (adjoin-set (make-leaf (car pair)
                                   (cadr pair))
                        (make-leaf-set (cdr pairs))))))
```


