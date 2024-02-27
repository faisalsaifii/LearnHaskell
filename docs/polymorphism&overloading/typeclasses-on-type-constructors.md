---
sidebar_position: 5
---

# Type classes on type constructors

## Generic concepts

Some of the concepts we have seen are not specific to lists; for
example:

- the function `foldr` replaces data constructors by suitable
functions and follows the structure of the datatype, just like the
standard design principle;
- the function elem traverses a data structure and checks
whether it contains a particular element;
- the function filter traverses a data structure and produces a
substructure containing just the elements with a certain property;
- the function map traverses a data structure and produces a new
structure of the same shape, but with modified elements.
For some of these concepts, Haskell therefore offers more type classes.

## `Foldable`

A class for data structures that can be viewed as a list, i.e., that have
elements in some natural order.

```hs
class Foldable t where
foldr :: (a -> b -> b) -> b -> t a -> b
foldl' :: (b -> b -> b) -> b -> t a -> b
toList :: t a -> [a]
null :: t a -> Bool
length :: t a -> Int
elem :: Eq a => a -> t a -> Bool
maximum :: Ord a => t a -> a
product :: Num a => t a -> a
...
```

Some of these are only available via `Data.Foldable` .

Note that Foldable abstracts over a **parameterized** type t .

## Other foldable types

The `Maybe` type is a container with 0 or 1 elements:

```hs
GHCi> null (Just 3)
False
GHCi> null Nothing
True
GHCi> product Nothing
1
```

## Possible pitfall: foldable tuples and `Either`

A pair is a container containing exactly 1 element (its second
component). (Tagged value.)

```hs
GHCi> toList (3, 4)
[4]
GHCi> toList ("foo", True)
[True]
GHCi> sum (3, 4)
4
```

An `Either` is like Maybe where `Nothing` is replaced by `Left` . So `Right` injects an element, `Left` does not.

```hs
GHCi> length (Right 3)
1
GHCi> length (Left 3)
0
```

## Mapping over other types

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
mapTree :: (a -> b) -> Tree a -> Tree b
mapTree f (Leaf x) = Leaf (f x)
mapTree f (Node l r) = Node (mapTree f l) (mapTree f r)

data Maybe a = Nothing | Just a
mapMaybe :: (a -> b) -> Maybe a -> Maybe b
mapMaybe f Nothing = Nothing
mapMaybe f (Just x) = Just (f x)
```

## The Functor class

```hs
class Functor f where
fmap :: (a -> b) -> f a -> f b
```

Like `Foldable` , the class `Functor` abstracts over a parameterized
type.

```hs
instance Functor [] where
fmap = map
instance Functor Tree where
fmap = mapTree
instance Functor Maybe where
fmap = mapMaybe
```

```hs
(<$>) :: Functor f => (a -> b) -> f a -> f b
f <$> x = fmap f x -- just a different name
```

## Deriving `Functor` and `Foldable`

Class instances for `Functor` and `Foldable` (and a few other classes) can be derived via language extensions:

```hs
{-# LANGUAGE DeriveFunctor, DeriveFoldable #-}
```

Language pragmas have to appear at the top of the module.

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
deriving (Show, Eq, Functor, Foldable)
GHCi> length (Node (Leaf 3) (Leaf 4))
2
GHCi> (+ 1) <$> Node (Leaf 3) (Leaf 4)
Node (Leaf 4) (Leaf 5)
```
