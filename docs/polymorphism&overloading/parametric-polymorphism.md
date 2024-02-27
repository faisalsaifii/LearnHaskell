---
sidebar_position: 2
---

# Parametric polymorphism

## One function, several types

Some Haskell expressions and functions can have more than one type.

Example:

```hs
fst (x, y) = x
Possible type signatures (all would work):
fst :: (a, a) -> a
fst :: (Int, a) -> Int
fst :: (Int, Int) -> Int
fst :: (a, b) -> a
fst :: (Int, Char) -> Int
```

Is one of these clearly the “best” choice?

## Most general type

Haskell’s type system is designed such that (ignoring some language
extensions) each term has a most general type:

- the most general type allows the most flexible use;
- all other types the term has can be obtained by instantiating the most general type, i.e., by substituting type variables with type expressions.

## Instantiating types

The type signature

```hs
fst :: (a, b) -> a
declares the most general type for fst . Types like
fst :: (a, a) -> a
fst :: (Int, Char) -> Int
fst :: (a -> Int -> b, c) -> a -> Int -> b
```

are instantiations of the most general type.

Type inference will always infer the most general type!

(So sometimes it’s worth asking GHC about the inferred type of a
function, even if you started by providing a type signature, and you
might be surprised that the inferred type is more general than what
you had specified.)
No run-time type information
Haskell terms carry no type information at run-time.
Remember
You can only ever use a term in the ways its type dictates.

## No run-time type information

Haskell terms carry no type information at run-time.
:::note[Remember]
You can only ever use a term in the ways its type dictates.
:::

Example:

```hs
fst :: (a, b) -> a
fst (x, y) = x
restrictedFst :: (Int, Int) -> Int
restrictedFst = fst -- ok
newFst :: (a, b) -> a
newFst = restrictedFst -- type error!
```

## What is Parametric polymorphism?

- A type with type variables (but no class constraints) is called
(parametrically) polymorphic.
- Type variables can be instantiated to any type expression, but
several occurrences of the same variable have to be the same
type.
- If a function argument has polymorphic type, then you know
nothing about it. No pattern matching is possible. You can only
pass it on.
- If a function result has polymorphic type, then (except for
undefined and error ) you can only try to build one from the
function arguments.

Let us look at examples.

## Example

How many functions can you think of that have this type:

```hs
(Int, Int) -> (Int, Int)
```

And of this one?

```hs
(a, a) -> (a, a)
```

And of this one?

```hs
(a, b) -> (b, a)
```

(Thanks to Doaitse Swierstra for the example.)

## Parametricity

- In general, parametric polymorphism severely restricts how a
function can be implemented.
- So if the functionality you’re trying to implement is quite general,
this is a good thing, because it really prevents you from making
errors.
- Conversely, if you see a function with parametrically polymorphic
type, you know that it cannot look at the polymorphic values.
- By looking at polymorphic types alone, one can obtain non-trivial
properties of the functions. (This is sometimes called
“parametricity”.)
- For example, map :: (a -> b) -> [a] -> [b] must produce
a list in which all elements are obtained by applying the given
function to elements of the original list – but we don’t know how
long the resulting list is, or in which order the elements occur.

## A common pitfall: who gets to choose

Sometimes, it may be tempting to write a program like the following:

```hs
parse :: String -> a
parse "False" = False
parse "0" = 0
...
```

What is wrong here?
For polymorphic types, it is always the caller who gets to choose at
which type the function should be used.
A function with polymorphic result type (but no polymorphic
arguments) is impossible to write without either looping or causing an
exception: we’d have to produce a value that belongs to every type
imaginable!

## What if we need to return values of different types?

**Option 1:** use Either :

```hs
data Either a b = Left a | Right b
parse :: String -> Either Bool Int
parse "False" = Left False
parse "0" = Right 0
```

**Option 2:** define your own datatype.

```hs
data Value = VBool Bool | VInt Int
parse :: String -> Value
parse "False" = VBool False
parse "0" = VInt 0
```

The second option is quite common in libraries that interface with
dynamically typed languages (SQL, JSON, ...).
