---
sidebar_position: 1
---

# Maybe

## The `Maybe` type

```hs
data Maybe a = Nothing
| Just a
```

The `Maybe` datatype is often used to encode failure or an exceptional value:

```hs
lookup :: (Eq a) => a -> [(a, b)] -> Maybe b
find :: (a -> Bool) -> [a] -> Maybe a
```

## Encoding exceptions using `Maybe`

Assume that we have a data structure with the following operations:

```hs
up, down, right :: Loc -> Maybe Loc
update :: (Int -> Int) -> Loc -> Loc
```

Given a location `l1`, we want to move up, right, down, and update the resulting position with using `update (+ 1)` ...

Each of the steps can fail.

## Encoding exceptions using Maybe (contd.)

```hs
case up l1 of
Nothing -> Nothing
Just l2 -> case right l2 of
Nothing -> Nothing
Just l3 -> case down l3 of
Nothing -> Nothing
Just l4 -> Just (update (+ 1) l4)
```

In essence, we need

- a way to sequence function calls and use their results if successful
- a way to modify or produce successful results.

Sequencing:

```hs
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
```

```hs
up l1 >>=
\ l2 -> case right l2 of
Nothing -> Nothing
Just l3 -> case down l3 of
Nothing -> Nothing
Just l4 -> Just (update (+ 1) l4)
```

Sequencing:

```hs
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
```

```hs
up l1 >>=
\ l2 -> right l2 >>=
\ l3 -> case down l3 of
Nothing -> Nothing
Just l4 -> Just (update (+ 1) l4)
```

Sequencing:

```hs
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
```

```hs
up l1 >>=
\ l2 -> right l2 >>=
\ l3 -> down l3 >>=
\ l4 -> Just (update (+ 1) l4)
```

Sequencing:

```hs
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
```

## Sequencing and embedding

```hs
up l1 >>=
\ l2 -> right l2 >>=
\ l3 -> down l3 >>=
\ l4 -> Just (update (+ 1) l4)
```

```hs
up l1 >>=
\ l2 -> right l2 >>=
\ l3 -> down l3 >>=
\ l4 -> return (update (+ 1) l4)

(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
return :: a -> Maybe a
return x = Just x
```

```hs
up l1 >>=
\ l2 -> right l2 >>=
\ l3 -> down l3 >>=
\ l4 -> return (update (+ 1) l4)
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
f >>= g = case f of
Nothing -> Nothing
Just x -> g x
return :: a -> Maybe a
return x = Just x
(up l1) >>= right >>= down >>= return . update (+ 1)
```

## Observation

Code looks a bit like imperative code. Compare:

```hs
up l1 >>= \ l2 ->
right l2 >>= \ l3 ->
down l3 >>= \ l4 ->
return (update (+ 1) l4)
```

```hs
l2 := up l1;
l3 := right l2;
l4 := down l3;
return update (+ 1) l4
```

- In the imperative language, the occurrence of possible exceptions
is a side effect.
- Haskell is more explicit because we use the Maybe type and the
appropriate sequencing operation.

## A variation: `Either`

Compare the datatypes

```hs
data Either a b = Left a  | Right b
data Maybe a    = Nothing | Just a
```

The datatype Maybe can encode exceptional function results (i.e.,
failure), but no information can be associated with Nothing . We
cannot distinguish different kinds of errors.

Using Either , we can use Left to encode errors, and Right to
encode successful results.

## Sequencing and returning for Either

We can define variants of the operations for Maybe :

```hs
(>>=) :: Either Error a -> (a -> Either Error b)
-> Either Error b
f >>= g = case f of
Left e -> Left e
Right x -> g x
return :: a -> Either Error a
return x = Right x
```

## Simulating exceptions

We can abstract completely from the definition of the underlying
Either type if we define functions to throw and catch errors.

```hs
throwError :: e -> Either e a
throwError e = Left e
catchError :: Either e a -> -- computation
(e -> Either e a) -> -- handler
Either e a
catchError f handler = case f of
Left e -> handler e
Right x -> Right x
```
