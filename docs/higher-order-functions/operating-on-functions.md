---
sidebar_position: 4
---

# Operating on functions

## Flipping a function

If you want to change the order of arguments of a two-argument
curried function, you can use

```hs
flip :: (a -> b -> c) -> b -> a -> c
flip f x y = f y x
```

Note once again that the function arrow associates to the right, so
flip can really be seen as a function with three arguments:

```hs
f :: a -> b -> c
x :: b
y :: a
```

Example use:

```hs
foreach = flip map
example = foreach [1, 2, 3] (\ x -> x * x)
```

## Currying and uncurrying

Sometimes, you end up with a pair and want to apply a function to it
that typically (in Haskell) is in curried form. Fortunately, we can convert
between curried and uncurried form easily:

```hs
uncurry :: (a -> b -> c) -> (a, b) -> c
uncurry f (x, y) = f x y
curry :: ((a, b) -> c) -> a -> b -> c
curry f x y = f (x, y)
```

Example:

```hs
map (uncurry (*)) (zip [1 . . 3] [4 . . 6])
```
