---
sidebar_position: 3
---

# Overloading

## Reusing code, reusing names

:::note[Parametric polymorphism]
Allows you to use the same implementation in as many contexts as
possible.
:::

:::note[Overloading (ad-hoc polymorphism)]
Allows you to use the same function name in different contexts, but
with different implementations for different types.
:::

## Type classes

A type class defines an interface that can be implemented by
potentially many different types.
Example:

```hs
class Eq a where
(==) :: a -> a -> Bool
(/=) :: a -> a -> Bool
```

Using instance declarations, we can explain how a certain type (or
types of a certain shape) implement the interface.

## Instances

```hs
instance Eq Bool where
False == False = True
True == True = True
_==_ = False
x /= y = not (x == y)
instance Eq a => Eq [a] where
[] == [] = True
(x : xs) == (y : ys) = x == y && xs == ys
_==_ = False
xs /= ys = not (xs == ys)
```

We use equality on a while defining equality on `[a]`.

## Class constraints

All the instances of a given type class specify a subset of all the Haskell
types, namely the subset that implements the class interface.
In type signatures, class constraints specify that a type variable can
only be instantiated to types belonging to a certain class:

```hs
(==) :: Eq a => a -> a -> Bool
```

Read: “Given that a is an instance of Eq , the function has the type `a -> a -> Bool`.”

## Overloading and inference

Not only class methods, but also functions that directly or indirectly
use class methods can have types with constraints. Example:

```hs
allEqual :: Eq a => [a] -> Bool
allEqual [] = True
allEqual [x] = True
allEqual (x : y : ys) = x == y && allEqual (y : ys)
```

Also recall `elem` or `lookup` .

Class constraints will be automatically inferred by the compiler.

## Several class constraints

There can be multiple constraints on a function, and they can apply to
several variables:

```hs
example ::
(Eq a, Eq b, Show a) => (a, b) -> (a, b) -> String
example (x1, y1) (x2, y2)
    | x1 == x2 && y1 == y2 = show x1
    | otherwise = "different"
```

## Default definitions

```hs
class Eq a where
(==) :: a -> a -> Bool
(/=) :: a -> a -> Bool
x == y = not (x /= y)
x /= y = not (x == y)
```

Now:

```hs
instance Eq Bool where
False == False = True
True  == True  = True
_     == _     = False
```

And `(/=)` will work automatically.

Careful: if you provide neither (==) nor (/=) , you won’t get a complaint, but both functions will loop.

## Classes are not types

Note that

```hs
f :: Eq -> Eq -> Bool
f :: Eq a -> Eq a -> Bool
```

are both invalid. Classes appear in constraints!
Also note that the type

```hs
Eq a => a -> a -> Bool
```

forces both arguments to be of the same type. You cannot pass two
different types that are both an instance of Eq – that would require a
function of type

```hs
(Eq a, Eq b) => a -> b -> Bool
```
