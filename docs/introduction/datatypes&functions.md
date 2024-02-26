---
sidebar_position: 4
---

# Datatypes & Functions

## Bindings

A binding is a declaration that gives a name to an expression so that it can be reused.

```hs
five = 2 + 3
ten = five + five
aList = [1,2,3,4,five]
```

Such bindings are typically put into a Haskell file in an editor, with the extension `.hs`.

(One can make bindings directly in GHCi, but certain restrictions are then in place.)

## Using new bindings in GHCi

One can then load the source file into GHCi and use the bindings:

```hs
GHCi> five
5
GHCi> ten
10
GHCi> map (\ x -> x * ten) aList
[10,20,30,40,50]
```

## Function bindings

```hs
double = \ x -> x + x
```

A different way to define the same function:

```hs
double x = x + x
GHCi> double 3
6
GHCi> double (-10)
-20
```

Careful with negative numbers!

```hs
GHCi> double - 10
Non type-variable argument in the constraint: Num (a -> a)
```

(A slightly strange way to complain that there is no Num instance for
function types.)

## Type signatures

Type signatures are optional, but strongly encouraged:

```hs
distance :: Num a => a -> a -> a
distance x1 x2 = abs (x1 - x2)
```

- Type signatures are checked and enforced.
- Types provide guidance for defining functions.
- Typically type errors are better with explicit type signatures.

## The `Prelude`

Very little is built into Haskell. E.g., all of

```hs
(+)
min
not
```

are **library** functions.

- Code is organized into **modules**.
- One special module `Prelude` is implicitly available in any other
Haskell module.

## Defining a datatype

```hs
data Choice = Rock | Paper | Scissors
deriving Show
Defines a new datatype Choice with just three values: Rock ,
Paper , and Scissors .
GHCi> :t Rock
Choice
GHCi> :t Paper
Choice
GHCi> Rock
Rock
```

The last command works only because **deriving** `Show` instructs GHC to create an “obvious” `Show` instance for the new datatype.

## Pattern matching

```hs
improve :: Choice -> Choice
improve Rock = Paper
improve Paper = Scissors
improve Scissors = Rock
```

The terms Rock , Paper and Scissors are called the (data) constructors of the Choice type.

Data constructors can be used for pattern matching.

If a binding has multiple equations, then the patterns on the left hand sides determine which equation applies for a given argument.

```hs
GHCi> improve Paper
Scissors
```

## Logical or

The actual definitions of Bool and (|) :

```hs
data Bool = False | True
deriving Show -- and other classes
(||) :: Bool -> Bool -> Bool
False || y = y
True || y = True
GHCi> True || False
True
GHCi> False || False
False
```

## Our own list type

```hs
data List a = Nil | Cons a (List a)
deriving Show
GHCi> :t Nil
List a
GHCi> :t Cons
a -> List a -> List a
GHCi> :t Cons 1 (Cons 2 (Cons 3 Nil))
Cons 1 (Cons 2 (Cons 3 Nil)) :: Num a => List a
GHCi> Cons 1 (Cons 2 (Cons 3 Nil))
Cons 1 (Cons 2 (Cons 3 Nil))
```

## Built-in lists

Special syntax:

```hs
data [a] = [] | a : [a]
deriving Show -- and other classes
infixr 5 : -- the “cons” operator is right-associative
GHCi> :t []
[a]
GHCi> :t (:)
a -> [a] -> [a]
GHCi> :t 1 : 2 : 3 : []
1 : 2 : 3 : [] :: Num a => [a]
GHCi> 1 : 2 : 3 : []
[1,2,3]
```

The notation `[1,2,3]` is actually syntactic sugar for `1 : 2 : 3 : []`.

## Pattern matching on lists

```hs
elem x [] = ...
```

What if we are looking for an element in the empty list?

```hs
elem x [] = False
```

If a list is not `[]` , it must be of shape `y : ys` for a suitable “head” y and “tail” ys ...

```hs
elem x [] = False
elem x (y : ys) = ...
```

One option is that x is equal to y ...

```hs
elem x [] = False
elem x (y : ys) = x == y
```

Here, `(==)` tests two expressions for equality.

This definition works, but it is not correct. We also need to consider ys ...

```hs
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

Recursion is the answer!

The list datatype definition is recursive. Functions on lists are typically recursive as well.

What about a type signature?

```hs
elem :: Eq a => a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

We make no assumptions about the type of list elements except that we can perform the equality test, which comes from the `Eq` class.

## Haskell’s evaluation model

- Expressions are “reduced” to values.
- For function calls, find matching equations.
- Replace left hand sides by right hand sides.
- Stop once no more reduction is possible (a value is reached).
- This process is called **equational reasoning**.

## Equational reasoning example

```hs
elem 9 [6,9,42]
⇝ elem 9 (6 : (9 : (42 : [])))
⇝ 9 == 6 || elem 9 (9 : 42 : [])
⇝ False || elem 9 (9 : 42 : [])
⇝ elem 9 (9 : 42 : [])
⇝ 9 == 9 || elem 9 (42 : [])
⇝ True || elem 9 (42 : [])
⇝ True
```

Remember:

```hs
elem x [] = False
elem x (y : ys) = x == y || elem x ys
False || y = y
True || y = True
```

## Lazy evaluation

Let’s look at the definition of “or” again:

```hs
False || y = y
True || y = True
```

- We can make a decision without looking at the second argument (and indeed we did, while reducing elem ).
- This definition of (||) has “shortcut behaviour”.
- Unlike in many languages, this does not require a special hack, but follows from the definition and Haskell’s evaluation strategy that essentially says “only evaluate things once they are needed”.

## Recap

The most fundamental concept to define functions on datatypes is
pattern matching:

- We distinguish multiple cases, usually one per data constructor of
the datatype of the function argument we analyze.
- We often use recursion when the underlying datatype is recursive
(such as lists are).

It is very easy to define new datatypes. New datatypes start out with
no operations (except pattern matching), but for some classes, we can
use deriving to obtain instances automatically.

## Outlook

In the next part of the course, we take a much more detailed look at how to define functions systematically using pattern matching, and discuss various datatypes.

We also discuss the expression language in more detail.

In the other parts we will discuss:

- parametric polymorphism and overloading,
- higher-order functions and abstraction,
- explicit effects (the IO ) type,
- more advanced abstraction patterns such as applicative functors and monads.
