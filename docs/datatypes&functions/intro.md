---
sidebar_position: 1
---

# Introduction

## Goals

- Learn how to define functions
- How types help you while programming
- Syntax of Haskell
- How to define and use datatypes
- Overview of base types and datatypes

## Structure of a Haskell program

- Haskell programs comprise one or more **modules**. One module per file. Main module is always called `Main`.
- Modules consist of **declarations**. Declarations introduce datatypes, functions and constants, type classes and instances.
- We will focus on functions and constants first, then datatypes. Later type classes and instances.

## Declaring new functions and constants

```hs
length :: [a] -> Int
length [] = 0
length (x : xs) = 1 + length xs
```

- The name being introduced.
- Type signature (optional, but recommended).
- One or more equations defining the function.
- The = symbol separates the **left hand sides** from the **right hand sides**.
- Cases are distinguished by **patterns**.
- On the right hand side, we have **expressions**.

## Declarations, patterns, expressions

Informally:

- A (function or constant) **declaration** binds a (new) identifier to an expression.
- A **pattern** occurs as an argument to an identifier on the left hand side of a declaration. It introduces names that are available on the right hand side. Patterns can be matched against actual
function arguments. Matches can fail or succeed.
- **Expressions** occur on the right hand side of a function definition. Expressions can be evaluated.

## Types

Every expression must have a **type** in Haskell – otherwise it will be rejected by the compiler:

- Haskell types can be inferred. There’s usually no need for type annotations.
- Use `:t` in GHCi to obtain the inferred type of an expression.
- Type annotations (using `::` ) are optional. But if they’re given, their correctness is checked.

## How to define a function?

There are two main design principles for defining functions:

- by (systematic) pattern matching and recursion,
- by applying a higher-order function (such as composition, `map` , `foldr` , ...) and thereby reducing the problem to smaller subproblems.

In both cases, thinking about the types first helps you! We will focus on the pattern matching approach first.

## Functions on lists

Most functions operate on **structured data**.
Lists are a simple data structure, so they’re ideal for learning.

Recall from the Quick Tour

Lists are defined **inductively**:

- The empty list [] is a list.
- Given a single element y and a list ys , we can construct a new list y : ys (pronounced y cons ys ).

We call `[]` and `(:)` the **constructors** of the list datatype.

## Another look at `elem`

Let’s try to implement `elem` once more, systematically.

```hs
elem :: Int -> [Int] -> Bool
```

We **start** with the type.

Do we want to restrict ourselves to `Int` lists? No!

```hs
elem :: a -> [a] -> Bool
```

Let’s make as few assumptions as possible.

In order to split up the programming problem, let’s take a look at the
input list ...

```hs
elem :: a -> [a] -> Bool
elem x [] = ...
elem x (y : ys) = ...
```

There are two cases, one per constructor of the list datatype.
Let’s see if we can solve the simple case for `[]` .

```hs
elem :: a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = ...
```

Now to the cons-case.
The ys is a shorter list – the most natural way to define functions on
recursive datatypes is to use recursive functions.

Let’s try to implement elem once more, systematically.

```hs
elem :: a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = ... elem ys ...
```

Let’s try to complete the second case making use of the recursive call.

```hs
elem :: a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

Done?

```hs
elem :: a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

Oh, we actually need equality on the elements. That seems to be a
sensible requirement for `elem` , so let’s refine the type ...

```hs
elem :: Eq a => a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

Now we’re really done

```hs
elem :: Eq a => a -> [a] -> Bool
elem x [] = False
elem x (y : ys) = x == y || elem x ys
```

The systematic development we’ve just seen generalizes to most functions on lists and most functions on other structured datatypes.

## Mapping over a list

```hs
map :: (a -> b) -> [a] -> [b]
```

Start with the type. A function is like any other argument.

```hs
map :: (a -> b) -> [a] -> [b]
map f [] = ...
map f (x : xs) = ...
```

Introduce cases based on the list constructors.

```hs
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x : xs) = ...
```

Solve the simple case first.

```hs
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x : xs) = ... map f xs ...
```

Keep recursion in mind.

```hs
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x : xs) = f x : map f xs
```

As f is a function, we can apply it.
Take a final look.
Everything looks fine.

## `drop` elements from a list

The call drop n xs should remove the first n elements from xs .

```hs
drop :: Int -> [a] -> [a]
drop n [] = ...
drop n (x : xs) = ...
```

What do we actually want to do if we want to drop 3 elements of an empty list?

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop n (x : xs) = ...
```

We take a simple approach.

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop n (x : xs) = ... drop ... xs ...
```

Wait, but what we want to do depends on n ?
We have multiple options here ...

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop 0 (x : xs) = ... drop ... xs ...
drop n (x : xs) = ... drop ... xs ...
```

We can pattern match on an Int too ...
If cases overlap, the first matching case applies.

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop 0 (x : xs) = x : xs
drop n (x : xs) = ... drop ... xs ...
```

Sometimes, we don’t need to recurse – even though we could.

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop 0 (x : xs) = []
drop n (x : xs) = drop (n - 1) xs
```

Done.
But what happens with negative numbers as arguments?

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop n (x : xs) =
    if n <= 0
        then x : xs
        else drop (n - 1) xs
```

We can use `if - then - else` .
We can include negative numbers now.
If an equation spans multiple lines, the subsequent lines must be
indented with respect to the first.

```hs
drop :: Int -> [a] -> [a]
drop n [] = []
drop n (x : xs)
  | n <= 0 = x : xs
  | otherwise = drop (n - 1) xs
```

Yet another option: use so-called guards.
Can only appear directly after the pattern match. Boolean conditions
are tried in order, `otherwise` is just a constant that is defined to be `True` .

## Exercise – define the following functions

Append two lists: `(++) :: [a] -> [a] -> [a]`

Hint: Only pattern match on the first list (i.e., don’t distinguish more
cases than needed).

Reverse a list: `reverse :: [a] -> [a]`

Hint: Follow the standard pattern, and make use of `(++)` that you
have just defined.

## Excursion: infix operators

Haskell allows you to create your own operators from a given set of
symbols:

- names are either completely symbolic or completely
alphanumeric;
- symbolic names are by default used infix, but can be used in
prefix notation by surrounding them in parentheses (Example:
(+) 2 3 evaluates to 5 );
- alphanumeric names are by default used prefix, but can be used
in infix notation by surrounding them in backquotes (Example:
8 `mod` 3 evaluates to 2 );
- you can define the associativity and priority of infix operators by
using infix , infixl , and infixr declarations;
- by using `:i` or `:info` in GHCi, you can obtain information
about the priority of infix operators.

## Filtering a list

We want to traverse a list and keep all elements that have a certain
property.
Question
How to best express a property of an element?
As a function from the element to a Bool .
Recall from the Quick Tour: a Bool is a another datatype with two
constructors, called True and False .

## Defining `filter`

```hs
filter :: (a -> Bool) -> [a] -> [a]
```

We can now write down the type

```hs
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = ...
filter p (x : xs) = ... filter ... xs ...
```

```hs
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (x : xs) = ... filter ... xs ...
```

It depends on the outcome of p x what we want to do!
We have several options here.

```hs
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (x : xs) = if p x
                        then x : filter p xs
                        else filter p xs
```

We can use the built-in if - then - else construct.
Note that Bool is a type like any other. No need to write
`p x == True` . Plain `p x` is simpler and equivalent.

```hs
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (x : xs)
  | p x = x : filter p xs
  | otherwise = filter p xs
```

We can also use guards – each guard is tried in order.
Note that

```hs
otherwise :: Bool
otherwise = True
```

### Using `filter`

There are some useful predicates:

```hs
even, odd :: Integral a => (a -> Bool)
isUpper, isDigit :: Char -> Bool
```

We can also define our own:

```hs
positiveInt :: Int -> Bool
positiveInt n = n > 0

palindrome :: [Char] -> Bool
palindrome xs = reverse xs == xs
```

Note that String is a (type) synonym for `[Char]`

Try using `filter` with these predicates.

## Excursion: anonymous functions

In practice, functions such as filter are often used with lambda
expressions or anonymous functions:

```hs
filter (\ n -> n > 10 && even n) [1 . . 100]
```

A lambda expression is a way to define a function without giving it a
name:

```hs
myPredicate n = n > 10 && even n
```

is just different syntax for

```hs
myPredicate = \ n -> n > 10 && even n
```

## Excursion: operator sections

Partially applied infix operators have yet again special syntax:

```hs
\ n -> n > 10
```

can be abbreviated to

```hs
(> 10)
```

Similarly, we can write `(1 +)` or `("Hello " ++)` or (\`div\`5).

So it’s possible to say

```hs
filter (> 10) [1 . . 100]
```
