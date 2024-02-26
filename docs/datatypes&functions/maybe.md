---
sidebar_position: 3
---

# Maybe

## How to fix lookup

We need a disciplined way to express failure without crashing.

:::tip[Idea]
Let’s use a different result type for `lookup`, containing one additional value called `Nothing` to express failure.
:::

## The Maybe datatype

Given a type a , the type Maybe a contains all the values of type a
plus one additional value:

- the term Nothing is a value of type Maybe a ,
- if x is of type a , then Just x is of type Maybe a .

There are two shapes / constructors of the Maybe datatype:
Nothing and Just .

## Defining lookup – “good” version

```hs
lookup :: Eq key => key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = lookup x ys
```

This is the version we had before.

```hs
lookup :: Eq key => key -> [(key, val)] -> Maybe val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = lookup x ys
```

We are now adapting the result type.
This requires changes in the rest of the function.

```hs
lookup :: Eq key => key -> [(key, val)] -> Maybe val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = lookup x ys
```

The first of the right hand sides can be improved now. The other is no
longer type correct.

```hs
lookup :: Eq key => key -> [(key, val)] -> Maybe val
lookup x [] = Nothing
lookup x ((k, v) : ys)
  | x == k = v -- still wrong
  | otherwise = lookup x ys
```

Instead of using error , we can now return Nothing.

```hs
lookup :: Eq key => key -> [(key, val)] -> Maybe val
lookup x [] = Nothing
lookup x ((k, v) : ys)
  | x == k = Just v
  | otherwise = lookup x ys
```

To inject v into the Maybe type, we use Just .

```hs
lookup :: Eq key => key -> [(key, val)] -> Maybe val
lookup x [] = Nothing
lookup x ((k, v) : ys)
| x == k = Just v
| otherwise = lookup x ys
```

Done. This version of lookup is total.

It does not crash, but the type tells the user that Nothing may be
returned, and forces the caller to deal with it.

## Handling exceptions with Maybe

Given a default value, we can always recover a value from a Maybe:

```hs
fromMaybe :: a -> Maybe a -> a
fromMaybe def x = ...
```

We pattern match on the Maybe .
Two constructors, Nothing and Just .

Given a default value, we can always recover a value from a Maybe:

```hs
fromMaybe :: a -> Maybe a -> a
fromMaybe def Nothing = ...
fromMaybe def (Just x) = ...
```

We use the default value in the Nothing case, and the wrapped value in the other.

Given a default value, we can always recover a value from a Maybe :

```hs
fromMaybe :: a -> Maybe a -> a
fromMaybe def Nothing = def
fromMaybe def (Just x) = x
```

Done.

## Combining Maybe computations

We can provide a “backup” computation for a possibly failing computation.

```hs
(<|>) :: Maybe a -> Maybe a -> Maybe a
x <|> y = ...
```

We pattern match on the first input.

```hs
(<|>) :: Maybe a -> Maybe a -> Maybe a
Nothing <|> y = ...
Just x <|> y = ...
```

We take the second computation if the first fails, otherwise ignore it.

```hs
(<|>) :: Maybe a -> Maybe a -> Maybe a
Nothing <|> y = y
Just x <|> y = Just x
```

Done.

Note the similarity with (||) on Booleans (from the Quick Tour).

## A variant of filter for Maybe

```hs
mapMaybe :: (a -> Maybe b) -> [a] -> [b]
mapMaybe p [] = []
mapMaybe p (x : xs) = ... mapMaybe p xs ...
```

We get to this point, but now we have to inspect the result of p x.
We could define a helper function, but we can also use a case
expression ...

```hs
mapMaybe :: (a -> Maybe b) -> [a] -> [b]
mapMaybe p [] = []
mapMaybe p (x : xs) =
  case p x of
    Nothing -> ... mapMaybe p xs ...
    Just y -> ... mapMaybe p xs ...
```

With case , we can pattern match on the result of an expression.
Note that the left hand sides are separated from the right hand sides
with an arrow `( -> )` and not an equality sign `( = )`.

```hs
mapMaybe :: (a -> Maybe b) -> [a] -> [b]
mapMaybe p [] = []
mapMaybe p (x : xs) =
  case p x of
    Nothing -> mapMaybe p xs
    Just y -> y : mapMaybe p xs
```

We can complete the definition similarly to filter .
Everything looks fine now.

## Why Maybe?

- By using Maybe in a result, we can express explicitly that the function can fail.
- The caller has to address the potential failure.
- By using Maybe in an argument, we can express that an argument is optional.
- The function writer has to say what to do if Nothing is passed.
- Only Maybe types have Nothing . This is different from null in other languages. There are no “null pointer exceptions” in Haskell.
