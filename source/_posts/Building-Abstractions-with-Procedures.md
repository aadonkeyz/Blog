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
- **primitive expressions**: represent the simplest entities the language is concerned with
- **means of combination**: compound elements are built from simpler ones
- **means of abstraction**: compound elements can be named and manipulated as units
{% endnote %}

In programming, we deal with two kinds of elements: procedures and data. Later we will discover that they are really not so distinct. Informally, data is "stuff" that we want to manipulate, and procedures are descriptions of the rules for manipulating the data. Thus, any powerful programming languages should have methods for combining and abstracting procedures and data.

## Expressions

One easy way to get started at programming is to examine some typical interactions with an interpreter for the Scheme dialect of Lisp. Imagine that you are sitting at a computer terminal. You type an **expression**, and the interpreter responds by displaying the result of its **evaluating** that expression.

One kind of primitive expression you might type is a number. If you present Lisp with a number

```lisp
486
```

the interpreter will respond by printing

```lisp
486
```

Expressions representing numbers may be combined with an expression representing a primitive procedure (such as `+` or `*`) to form a compound expression that represents the application of the procedure to those numbers. For example:

```lisp
(+ 137 349)
486

(- 1000 334)
666
```

Expressions such as these, formed by delimiting a list of expressions within parentheses in order to denote procedure application, are called **combinations**. The leftmost element in the list is called the **operator**, and the other elements are called **operands**. The value of a combination is obtained by applying the procedure specified by the operator to the **arguments** that are the values of the operands.

The convention of placing the operator to the left of the operands is known as **prefix notation**, and it may be somewhat confusing at first because it departs significantly from the customary mathematical convention. Prefix notation has several advantages, however. One of them is that it can accommodate procedures that may take an arbitrary number of arguments, as in the following examples:

```lisp
(+ 21 35 12 7)
75
```

No ambiguity can rise, because the operator is always the leftmost element and the entire combination is delimited by the parentheses.

A second advantage of prefix notation is that it extends in a straightforward way to allow combinations to be **nested**, that is, to have combinations whose elements are themselves combinations:

```lisp
(+ (* 3 5) (- 10 6))
19
```

There is no limit (in principle) to the depth of such nesting and to the overall complexity of the expressions that the Lisp interpreter can evaluate. It is we humans who get confused by still relatively simple expressions such as

```lisp
(+ (* 3 (+ (* 2 4) (+ 3 5))) (+ (- 10 7) 6))
```

which the interpreter would readily evalute to be $57$. We can help ourselves by writing such an expression in the form

```lisp
(+ (* 3
      (+ (* 2 4)
         (+ 3 5)))
   (+ (- 10 7)
      6))
```

following a formatting convention known as **pretty-printing**, in which each long combination is written so that the operands are aligned vertically. The resulting indentations display clearly the structure of the expression.

Even with complex expressions, the interpreter always operates in the same basic cycle: It reads an expression from the terminal, evaluates the expression, and prints the result. This mode of operation is often expressed by saying that the interpreter runs in a **read-eval-print** loop. Observe in particular that it is not necessary to explicitly instruct the interpreter to print the value of the expression.

## Naming and the Environment

A critical aspect of a programming language is the means it provides for using names to refer to computational objects. We say that the name identifies a **variable** whose **value** is the object.

In the Scheme dialect of Lisp, we name things with `define`. Typing

```lisp
(define size 2)
```

causes the interpreter to associate the value $2$ with the name `size`. Once the name `size` has been associated with the number $2$, we can refer to the value $2$ by name:

```lisp
size
2

(* 5 size)
10
```

Here are further examples of the use of `define`:

```lisp
(define pi 3.14159)
(define radius 10)
(* pi (* radius radius))
314.159
(define circumference (* 2 pi radius))
circumference
62.8318
```

`define` is our language's simplest means of abstraction, for it allows us to use simple names to refer to the results of compound operations, such as the `circumference` computed above. In general, computational objects may have very complex structures, and it would be extremely inconvenient to have to remember and repeat their details each time we want to use them. Indeed, complex programs are constructed by building, step by step, computational objects of increasing complexity. The interpreter makes this step-by-step program construction particularly convenient because name-object associations can be created incrementally in successive interactions. This feature encourages the incremental development and testing of programs and is largely responsible for the fact that a Lisp program usually consist of a large number of relatively simple procedures.

It should be clear that the possibility of associating values with symbols and later retrieving them means that the interpreter must maintain some sort of memory that keeps track of the name-object pairs. This memory is called the **environment**.

## Evaluating Combinations

As a case in point, let us consider that, in evaluating combinations, the interpreter is itself following a procedure.

To evaluate a combination, do the following:

{% note info %}
1. Evaluate the subexpressions of the combination
2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands)
{% endnote %}

Even this simple rule illustrates some important points about processes in general. First, observe that the first step dictates that in order to accomplish the evaluation process for a combination we must first perform the evaluation process on each element of the combination. Thus, the evaluation rule is **recursive** in nature; that is, it includes, as one of its steps, the need to invoke the rule itself.

Notice how succinctly the idea of recursion can be used to express what, in the case of a deeply nested combination, would otherwise be viewed as a rather complicated process. For example, evaluating

```lisp
(* (+ 2 (* 4 6))
   (+ 3 5 7))
```

requires that the evaluation rule be applied to four different combinations. We can obtain a picture of this process by representing the combination in the form of a tree.

![tree representation, showing the value of each subcombination](https://blog-images-1258719270.cos.ap-shanghai.myqcloud.com/%E3%80%8A%20Structure%20and%20Interpretation%20of%20Computer%20Programs%20%28Lisp%29%20%E3%80%8B/tree%20representation%2C%20showing%20the%20value%20of%20each%20subcombination.png)

Each combination is represented by a node with branches corresponding to the operator and the operands of the combination stemming from it. The terminal nodes (that is, nodes with no branches stemming from them) respent either operators or numbers. Viewing evaluation in terms of the tree, we can imagine that the values of the operands percolate upward, starting from the terminal nodes and then combining at higher and higher levels. In general, we shall see that recursion is a very powerful technique for dealing with hierarchical, treelike objects. In fact, the "percolate values upward" form of the evaluation rule is an example of a general kind of process knowns as **tree accumulation**.

Next, observe that the repeated application of the first step brings us to the point where we need to evaluate, not combinations, but primitive expressions such as numerals, built-in operators, or other names. We take care of the primitive cases by stipulating that

{% note info %}
- the values of numerals are the numbers that they name
- the values of built-in operators are the machine instruction sequences that carry out the corresponding operations
- the values of other names are the objects associated with those names in the environment
{% endnote %}

We may regard the second rule as a special case of the third one by stipulating that symbols such as `+` and `*` are also included in the global environment, and are associated with the sequences of machine instructions that are their "values". The key point to notice is the role of the environment in determining the meaning of the symbols in expressions. In an interactive language such as Lisp. it is meaningless to speak of the value of an expression such as `(+ x 1)` without specifying any information about the environment that would provide a meaning for the symbol `x` (or even for the symbol `+`). The general notion of the environment as providing a context in which evaluation takes place will play an important role in our understanding of program execution.

Notice that the evaluation rule given above does not handle definitions. For instance, evaluating `(define x 3)` does not apply `define` to two arguments, since the purpose of the `define` is precisely to associate `x` with a value. That is, `(define x 3)` is not a combination. Such exceptions to the general evaluation rule are called **special forms**. `define` is the only example of a special form that we have seen so far, but we will meet others shortly.

## Compound Procedures

We have identified in Lisp some of the elements that must appear in any powerful programming language:

{% note info %}
- Numbers and arithmetic operations are primitive data and procedures
- Nesting of combinations provides a means of combining operations
- Definitions that associate names with values provide a limited means of abstraction
{% endnote %}

Now we will learn about **procedure definitions**, a much more powerful abstraction technique by which a compound operation can be given a name and then referred to as a unit.

The general form of a procedure definition is

```lisp
(define (<name> <formal parameters>)
    <body>)
```

The `<name>` is a symbol to be associated with the procedure definition in the environment. The `<formal parameters>` are the names used within the body of the procedure to refer to the corresponding arguments of the procedure. The `<body>` is an expression that will yield the value of the procedure application when the formal parameters are replaced by the actual arguments to which the procedure is applied. The `<name>` and the `<formal parameters>` are grouped within parentheses, just as they would be in an actual call to the procedure being defined.

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

{% note warning %}
The purpose of the substitution is to help us think about procedure application, not to provide a description of how the interpreter really works. Typical interpreters do not evaluate procedure applications by manipulating the text of a procedure to substitute values for the formal parameters. In practice, the "substitution" is accomplished by using a local environment for the formal parameters.
{% endnote %}

### Applicative Order

The interpreter first evaluates the operator and operands and then applies the resulting procedure to the resulting arguments

```lisp
; step 1
(f 5)

; step 2
(sum-of-squares (+ 5 1) (* 5 2))

; step 3
(+ (square 6) (square 10))

; step 4
(+ 36 100)

; step 5
136
```

### Normal Order

The interpreter first substitute operand expressions for parameters until it obtained an expression involving only primitive operators, and would then perform the evaluation

```lisp
; step 1
(f 5)

; step 2
(sum-of-squares (+ 5 1) (* 5 2))

; step 3
(+ (square (+ 5 1)) (square (* 5 2)) )

; step 4
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2)))

; step 5
(+ (* 6 6) (* 10 10))

; step 6
(+ 36 100)

; step 7
136
```

## Conditional Expressions and Predicates

The general form of a conditional expression is

```lisp
(cond (<p_1> <e_1>)
      (<p_2> <e_2>)
      ...
      (<p_n> <e_n>))
```

The general form of an `if` expression is

```lisp
(if predicate consequent alternative)
```

blablabla...

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

; subprocedures
(define (improve guess x)
    (average guess (/ x guess)))

(define (average x y)
    (/ (+ x y) 2))

(define (good-enough? guess x)
    (< (abs (- (square guess) x)) 0.001))
```

Finally, we need a way to get started. For instance, we can always guess that the square root of any number is 1:

```lisp
(define (sqrt x)
    (sqrt-iter 1.0 x))
```

If we type these definitions to the interpreter, we can use `sqrt` just as we can use any procedure:

```lisp
(sqrt 9)
3.00009155413138
```

## Procedure as Block-Box Abstractions

Observe that the problem of computing square roots breaks up naturally into a number of subproblems. Each of these tasks is accomplished by a separate procedure. The entire `sqrt` program can be viewed as a cluster of procedures that mirrors the decomposition of the problem into subproblems.

The importance of this decomposition strategy is not simply that one is dividing the program into parts. After all, we could take any large program and divide it into parts -- the first ten lines, the next ten lines, the next ten lines, and so on. Rather, it is crucial that each procedure accomplishes an identifiable task that can be used as a module in defining other procedures. For example, when we define the `good-enough?` procedure in terms of `square`, we are able to regard the `square` procedure as a "black box". We are not at that moment concerned with how the procedure computes its result, only with the fact that it computes the square. The details of how the square is computed can be suppressed, to be considered at a later time. Indeed, as far as the `good-enough?` procedure is concerned, `square` is not quite a procedure but rather an abstraction of a procedure, a so-called **procedural abstraction**. At this level of abstraction, any procedure that computes the square is equally good.

### Local names

One detail of a procedures's implementation that should not matter to the user of the procedure is the implementer's choice of names for the procedure's formal parameters. Thus, the following procedures should not be distinguishable:

```lisp
(define (square x) (* x x))
(define (square y) (* y y))
```

This principle -- that the meaning of a procedure should be independent of the parameter names used by its author -- seems on the surface to be self-evident, but its consequences are profound. The simplest consequence is that the parameter names of a procedure must be local to the body of the procedure. For example, we used `square` in the definition of `good-enough?` in our `square-root` procedure.

```lisp
(define (good-enough? guess x)
    (< (abs (- (square guess) x))
       0.001))
```

The intention of the author of `good-enough?` is to determine if the square of the first argument is within a given tolerance of the second argument. We see that the author of `good-enough?` used the name `guess` to refer to the first argument and `x` to refer to the second argument. The argument of `square` is `guess`. If the author of `square` used `x` (as above) to refer to that argument, we see that the `x` in `good-enough?` must be a different `x` than the one in `square`. Running the procedure `square` must not affect the value of `x` that is used by `good-enough?`, because that value of `x` may be needed by `good-enough?` after `square` is done computing.

If the parameters were not local to the bodies of their respective procedures, then the parameter `x` in `square` could be confused with the parameter `x` in `good-enough?`, and the behavior of `good-enough?` would depend upon which version of `square` we used. Thus, `square` would not be the black box we desired.

A formal parameter of a procedure has a very special role in the procedure definition, in that it doesn't matter what name the formal parameter has. Such a name is called a **bound variable**, and we say that the procedure definition **binds** its formal parameters. The meaning of a procedure definition is unchanged if a bound variable is consistently renamed throughout the definition. If a variable is not bound, we say that it is **free**. The set of expressions for which a binding defines a name is called the **scope** of that name. In a procedure definition, the bound variables declared as the formal parameters of the procedure have the body of the procedures as their scope.

### Internal definitions and block structure

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

Such nesting of definitions, called **block structure**, is basically the right solution to the simplest name-packaging problem. But there is a better idea lurking here. In addition to internalizing the definitions of the auxiliary procedures, we can simplify them. Since `x` is bound in the definition of `sqrt`, the procedures `good-enough?`, `improve`, and `sqrt-iter` which are defined internally to `sqrt` are in the scope of `x`. Thus, it is not necessary to pass `x` explicitly to each of these procedures. Instead, we allow `x` to be a free variable in the internal definitions, as shown below. Then `x` gets its value from the argument with which the enclosing procedure `sqrt` is called. This discipline is called **lexical scoping**.

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

A procedure is a pattern for the **local evolution** of a computational process. It specifies how each stage of the process is built upon the previous stage. We would like to be able to make statements about the overall, or global, behavior of a process whose local evolution has been specified by a procedure.

## Linear Recursion and Iteration

We begin by considering the factorial function, defined by

{% note info %}
<center>$n! = n \cdot (n-1) \cdot (n-2) \cdots 3 \cdot 2 \cdot 1$</center>
{% endnote %}

There are many ways to compute factorials. One way is to make use of the observation that $n!$ is equal to $n$ times $(n-1)!$ for any positive integer $n$. Thus, we can compute $n!$ by computing $(n-1)!$ and multiplying the result by $n$. If we add the stipulation that $1!$ is equal to $1$, this observation translates directly into a procedure:

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

The contrast between the two processes can be seen in another way. In the iterative case, the program variables provides a complete description of the state of the process at any point. If we stopped the computation between steps, all we would need to do to resume the computation is to supply the interpreter with the values of the three program variables. Not so with the recursive process. In this case there is some additional "hidden" information, maintained by the interpreter and not contained in the program variables, which indicates "where the process is" in negotiating the chain of deferred operations. The longer the chain, the more information must be maintained.

In contransting iteration and recursion, we must be careful not to confuse the notion of a **recursive process** with the notion of a **recursive procedure**. When we describe a procedure as recursive, we are referring to the syntactic fact that the procedure definition refers (either directly or indirectle) to the procedure itself. But when we describe a process as following a pattern that is, say, linearly recursive, we are speaking about how the process evolves, not about the syntax of how a procedure is written. It may seem disturbing that we refer to a recursive procedure such as `fact-iter` as generating an iterative process. However, the process really is iterative: **Its state is captured completely by its three state variables, and an interpreter need keep track of only three variables in order to execute the process.**

## Tree Recursion

Another common pattern of computation is called **tree recursion**. As an example, consider computing the sequence of Fibonacci numbers

```lisp
(define (fib n)
    (cond ((= n 0) 0)
          ((= n 1) 1)
          (else (+ (fib (- n 1)
                   (fib (- n 2)))))))
```

This procedure is instructive as a prototypical tree recursion, but it is a terrible way to compute Fibonacci numbers because it does so much redundant computation.

We can also formulate an iterative process for computing the Fibonacci numbers. The idea is to use a pair of integers $a$ and $b$, initialized to $Fib(1) = 1$ and $Fib(0) = 0$, and to repeatedly apply the simultaneous transformations

{% note info %}
- $a$ ← $a$ + $b$
- $b$ ← $a$
{% endnote %}

It is not hard to show that, after applying this transformation $n$ times, $a$ and $b$ will be equal, respectively, to $Fib(n+1)$ and $Fib(n)$. Thus, we can compute Fibonacci numbers iteratively using the procedure

```lisp
(define (fib n)
    (fib-iter 1 0 n))
(define (fib-iter a b count)
    (if (= count 0)
        b
        (fib-iter (+ a b) a (- count 1))))
```

One shoule not conclude from this that tree-recursive process are useless. When we consider processes that operate on hierarchically structured data rather than numbers, we will find that tree recursion is a natural processes can be useful in helping us to understand and design programs.

Consider the following problem, how many different ways can we make change of $1.00, given half-dollars, quarters, dimes, nickels, and pennies? More generally, can we write a procedure to compute the number of ways to change any given amount of money?

This problem has a simple solution as a recursive procedure. Suppose we think of the types of coins available as arranged in some order. Then the following relation holds:

{% note info %}
The number of ways to change amount $a$ using $n$ kinds of coins equals the sum of the following two cases
- the number of ways to change amount $a$ using all but the first kind of coin, plus
- the number of ways to change amount $a-d$ using all $n$ kinds of coins, where $d$ is the denomination of the first kind of coin.
{% endnote %}

The latter number is equal to the number of ways to make change for the amount that remains after using a coin of the first kind. Thus, we can recursively reduce the problem of changing a given amount to the problem of changing smaller amounts using fewer kinds of coins. Consider this reduction rule carefully, and convince yourself that we can use it to describe an algorithm if we specify the following degenerate cases.

{% note info %}
- If $a$ is exactly $0$, we should count that as $1$ way to make change.
- If $a$ is less than $0$, we should count that as $0$ ways to make change.
- If $n$ is $0$, we should count that as $0$ ways to make change.
{% endnote %}

We can easily translate this description into a recursive procedure:

```lisp
(define (count-change amount) (cc amount 5))
(define (cc amount kinds-of-coins)
    (cond ((= amount 0) 1)
          ((or (< amount 0) (= kinds-of-coins 0)) 0
          (else (+ (cc amount
                       (- kinds-of-coins 1))
                   (cc (- amount
                          (first-denomination
                           kinds-of-coins))
                       kinds-of-coins))))))
(define (first-denomination kinds-of-coins)
    (cond ((= kinds-of-coins 1) 1)
          ((= kinds-of-coins 2) 5)
          ((= kinds-of-coins 3) 10)
          ((= kinds-of-coins 4) 25)
          ((= kinds-of-coins 5) 50)))
```

# Formulating Abstractions with Higher-Order Procedures

Even in numerical processing we will be severely limited in our ability to create abstractions if we are restricted to procedures whose parameters must be numbers. Often the same programming pattern will be used with a number of different procedures. To express such patterns as concepts, we will need to construct procedures that can accept procedures as arguments or return procedures as values. Procedures that manipulate procedures are called **higher-order procedures**.

## Procedures as Arguments

Consider the following three procedures. The first computes the sum of the integers from `a` through `b`:

```lisp
(define (sum-integers a b)
    (if (> a b)
        0
        (+ a (sum-integers (+ a 1) b))))
```

The second computes the sum of the cubes of the integers in the given range:

```lisp
(define (sum-cubes a b)
    (if (> a b)
        0
        (+ (cube a)
           (sum-cubes (+ a 1) b))))
```

The third computes the sum of a sequence of terms in the series

{% note info %}
<center>$\frac{1}{1 \cdot 3} + \frac{1}{5 \cdot 7} + \frac{1}{9 \cdot 11} + \cdots$</center>
{% endnote %}

```lisp
(define (pi-sum a b)
    (if (> a b)
        0
        (+ (/ 1.0 (* a (+ a 2)))
           (pi-sum (+ a 4) b))))
```

These three procedures clearly share a common underlying pattern. They are for the most part identical, differing only in the name of the procedure, the function of `a` used to computed the term to be added, and the function that provides the next value of `a`. We could generate each of the procedures by filling in slots in the same template:

```lisp
(define (<name> a b)
    (if (> a b)
        0
        (+ (<term> a)
           (<name> (<next> a) b))))
```

The presence of such a common pattern is strong evidence that there is a useful abstraction waiting to be brought to the surface. Indeed, mathematicians long ago identified the abstraction of summation of a series and invented "sigma notation", for example

{% note info %}
<center>$\sum_{n=a}^b f(n) = f(a) + \cdots + f(b)$</center>
{% endnote %}

to express this concept. The power of sigma notation is that it allows mathematicians to deal with the concept of summation itself rather than only with particular sums -- for example, to formulate general results about sums that are independent of the particular series being summed.

Similary, as program designers, we would like our language to be powerful enough so that we can write a procedure that expresses the concept of summation itself rather than only procedures that compute particular sums. We can do so readily in our procedural language by taking the common template shown above and transforming the "slots" into formal parameters:

```lisp
(define (sum term a next b)
    (if (> a b)
        0
        (+ (term a)
           (sum term (next a) next b))))
```

## Constructing Procedures Using lambda

In general, lambda is used to create procedures in the same way as `define`, except that no name is specified for the procedure:

```lisp
(define (<formal-parameters>) <body>)
```

The resulting procedure is just as much as a procedure as one that is created using `define`. The only difference is that it has not been associated with any name in the environment.

Like any expression that has a procedure as its value, a lambda expression can be used as the operator in a combination such as

```lisp
((lambda (x y z) (+ x y (square z)))
 1 2 3)
12
```

Another use of lambda is in creating local variables. We often need local variables in our procedures other than those that have been bound as formal parameters. For example, suppose we wish to compute the function

{% note info %}
<center>$f(x,y) = x(1+xy)^2 + y(1-y) + (1+xy)(1-y)$</center>
which we could also express as
<center>$a = 1 + xy$</center>
<center>$b = 1 - y$</center>
<center>$f(x,y) = xa^2 + yb + ab$</center>
{% endnote %}

In writing a procedure to compute `f`, we would like to include as local variables not only `x` and `y` but also the names of intermeditate quantities like `a` and `b`. One way to accomplish this is to use an auxiliary procedure to bind the local variables:

```lisp
(define (f x y)
    (define (f-helper a b)
        (+ (* x (square a))
           (* y b)
           (* a b)))
    (f-helper (+ 1 (* x y))
              (- 1 y)))
```

Of course, we could use a lambda expression to specify an anonymous procedure for binding our local variables. The body of `f` then becomes a single call to that procedure:

```lisp
(define (f x y)
    ((lambda (a b)
        (+ (* x (square a))
           (* y b)
           (* a b)))
     (+ 1 (* x y))
     (- 1 y)))
```

This construct is so useful that there is a special form called `let` to make its use more convenient. Using `let`, the `f` procedure could be written as

```lisp
(define (f x y)
    (let ((a (+ 1 ( * x y)))
          (b (- 1 y)))
        (+ (* x (square a))
           (* y b)
           (* a b))))
```

The general form of a `let` expression is

```lisp
(let ((<var_1> <exp_1>)
      (<var_2> <exp_2>)
      ...
      (<var_n> <exp_n>))
    <body>)
```

## Procedures as General Methods

The half-interval method is simple but powerful technique for finding roots of an equation $f(x) = 0$, where $f$ is a continuous function.

```lisp
(define (search f neg-point pos-point)
    (let ((midpoint (average neg-point pos-point)))
        (if (close-enough? neg-point pos-point)
            midpoint
            (let ((test-value (f midpoint)))
                (cond ((positive? test-value)
                       (search f neg-point midpoint))
                      ((negative? test-value)
                       (search f midpoint pos-point))
                      (else midpoint))))))
(define (close-enough? x y) (< (abs (- x y)) 0.001))
```

`search` is awkward to use directly, because we can accidentally give it points at which $f's$ value do not have the required sign, in which case we get a wrong answer. Instead we will use `search` via the following procedure, which checks to see which of the endpoints has a negative function value and which has a positive value, and calls the `search` procedure accordingly. If the function has the same sign on the two given points, the half-interval method cannot be used, in which case the procedure signals an error.

```lisp
(define (half-interval-method f a b)
    (let ((a-value (f a))
          (b-value (f b)))
        (cond ((and (negative? a-value) (positive? b-value))
               (search f a b))
              ((and (negative? b-value) (positive? a-value))
               (search f b a))
              (else
               (error "Values are not of opposite sign" a b)))))
```

The following example uses the half-interval method to approximate $\pi$ as the root between $2$ and $4$ of $\sin x = 0$

```lisp
(half-interval-method sin 2.0 4.0)
3.14111328125
```

## Procedures as Returned Values

The above examples demonstrate how the ability to pass procedures as arguments significantly enhances the expressive power of our programming language. We can achieve even more expressive power by creating procedures whose returned values are themselves procedures.

But because I am a little bit lazy, I won't summary the content of this section of the book.


