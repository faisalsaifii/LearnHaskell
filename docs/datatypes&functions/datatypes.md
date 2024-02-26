---
sidebar_position: 4
---

# Datatypes

## Parameterized types, type constructors

- In Haskell, many types are parameterized by others.
- From existing types, we can make new types.
- Parameterized types are often called type constructors.

Examples:

“List” is a type constructor.

Given any type a , there is also a type `[a]`.

“Pair” is a type constructor.

Given any types a and b , there is also a type `(a, b)`.

“Function” is a type constructor.

Given any types `a` and `b` , there is also a type `a -> b`.

Building types with type constructors
There are no limits to composing type constructors:

```hs
([Int -> Bool -> Int], [[Double]] -> (Bool, Int))
```

There are arbitrarily many type constructors, because we can define
our own!
