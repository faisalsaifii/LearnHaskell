---
sidebar_position: 3
---

# Types

## Expressions have types

- Every expression (& subexpression) has a type.
- Types are checked statically.
- Types can be inferred.

## Inferring types in GHCi

You can use **GHCi** (or your editor with haskell-language-server) to infer types.

```haskell
GHCi> :t 'x'
'x' :: Char

GHCi> :t False
False :: Bool

GHCi> :t [True,False]
[True,False] :: [Bool]

GHCi> :t not
not :: Bool -> Bool
```

- `:t` is a **GHCi command** – it’s not a part of Haskell.
- The `::` symbol reads “is of type”.
- Types are different from expressions. You cannot use a type as an expression.

## Parametric polymorphism

```haskell
GHCi> :t reverse
reverse :: [a] -> [a]

GHCi> reverse [True,False]
[False,True]

GHCi> reverse [1,2,3]
[3,2,1]

GHCi> reverse "Haskell"
"lleksaH"
```

- Lower-case identifiers in types are type variables.
- Types involving variables are called parametrically polymorphic.
- We can choose at which concrete type to use the expression.
- Strings are lists of characters.

## Currying

```haskell
GHCi> :t take
take :: Int -> [a] -> [a]
```

Two views:

- A function that takes an Int and a [a] and returns a [a] .
- A function that takes an Int and returns another function, which then expects a [a] and returns a [a] .

The function arrow associates to the right. These two types are the same:

```haskell
take :: Int -> [a] -> [a]
take :: Int -> ([a] -> [a])
```

## Partial application

```haskell
GHCi> :t take
take :: Int -> [a] -> [a]

GHCi> :t take 2
take 2 :: [a] -> [a]

GHCi> :t take 2 "currying"
take 2 "currying" :: [Char]

GHCi> take 2 "currying"
"cu"
```

## Partial application in use

```haskell
GHCi> :t map
map :: (a -> b) -> [a] -> [b]

GHCi> :t take 2
take 2 :: [a] -> [a]

GHCi> :t map (take 2)
map (take 2) :: [[a]] -> [[a]]

GHCi> map (take 2) [[1,2,3],[4,5,6],[7,8,9]]
[[1,2],[4,5],[7,8]]
```

Partial application can be seen as an abbreviation for a lambda term:

```hs
GHCi> map (\ x -> take 2 x) [[1,2,3],[4,5,6],[7,8,9]]
[[1,2],[4,5],[7,8]]
```

## Overloading

Some functions work for many, but not all types:

```hs
GHCi> :t enumFromTo
enumFromTo :: Enum a => a -> a -> [a]
```

The part of to the left of the `=>` is a constraint which restricts the
choice of the type variable a to types that are an instance of the
type class `Enum` .

Many types are an instance of `Enum`, but not all types are.

## Overloading in use

```hs
GHCi> enumFromTo 1 5
[1,2,3,4,5]
GHCi> enumFromTo 'a' 'c'
"abc"
GHCi> enumFromTo False True
[False,True]
GHCi> enumFromTo "abc" "def"
No instance for (Enum [Char]) arising from a use of ‘enumFromTo ’
```

Lists are not an instance of the Enum class.

## Overloading is used in lots of places

```hs
GHCi> :t max
max :: Ord a => a -> a -> a
GHCi> :t (+)
(+) :: Num a => a -> a -> a
GHCi> :t length
length :: Foldable f => f a -> Int
```

The “container” is restricted, but not the element type:

```hs
GHCi> length [(+),(-),\ x y -> x * x + y]
3
```

For now, if you see `Foldable` , think “list”.

## Type errors

```hs
GHCi> not 'x'
Couldn’t match expected type ‘Bool ’ with actual type ‘Char ’
Numeric literals are overloaded, and cause somewhat confusing errors:
GHCi> :t 1
1 :: Num p => p
GHCi> not 1
```

No instance for (Num Bool) arising from the literal ‘1 ’

## GHCi pitfall

This looks like a type error, but the error is not in your code:

```hs
GHCi> take
No instance for (Show (Int -> [a0] -> [a0]))
arising from a use of ‘print ’
The expression is in fact type-correct:
GHCi> :t take
take :: Int -> [a] -> [a]
```

GHCi implicitly tries to call print on the expressions you type, to print the result of evaluation on screen, and this fails ...

## Explicit effects

```hs
GHCi> :t print
print :: Show a => a -> IO ()
```

- The type () is a type containing just one value () .
- The type IO () denotes an IO action yielding no interesting result, but having the side effect of printing the argument to the screen.
- The argument is flexible, but constrained to be an instance of the Show class.
- Functions are not an instance of the Show class, hence the GHCi error when typing in anything of a functional type.
- In Haskell, all side-effecting operations are explicitly marked by being elements of the IO type.

We have learned about:

- Inferring types with GHCi,
- Currying and partial application,
- Parametric polymorphism (types containing unconstrained variables),
- Overloading (types containing constrained variables),
- Explicit effects (the IO type).

The primary goals for now are to be aware of :t in GHCi and to be
able to make some sense of the reported types.
