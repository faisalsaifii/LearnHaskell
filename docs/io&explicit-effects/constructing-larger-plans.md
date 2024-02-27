---
sidebar_position: 3
---

# Constructing larger plans

## Basic sequencing

```hs
(>>) :: IO a -> IO b -> IO b
```

Function that takes two plans and constructs a plan that first executes
the first plan, discard its result, then executes the second plan, and
returns its result.

## Reading two lines

```hs
getTwoLines :: IO String
getTwoLines = getLine >> getLine
GHCi> getTwoLines
Line 1.
Line 2.
"Line 2."
```

## Modifying the result of a plan

```hs
liftM :: (a -> b) -> IO a -> IO b
```

Takes a function and a plan. Constructs a plan that executes the given plan, but before returning the result, applies the function.

```hs
duplicateLine :: IO String
duplicateLine = liftM (\ x -> x ++ x) getLine
GHCi> duplicateLine
Hello
"HelloHello"
```

## Shouting

```hs
GHCi> :t toUpper
toUpper :: Char -> Char
GHCi> toUpper 'x'
'X'
GHCi> liftM (map toUpper) getLine
Hello
"HELLO"
```

## Combining the output of two sequenced plans

```hs
liftM2 :: (a -> b -> c) -> IO a -> IO b -> IO c
```

Takes an operator and two plans. Constructs a plan that executes the
two plans in sequence, and uses the operator to combine the two
results.

```hs
joinTwoLines :: IO String
joinTwoLines = liftM2 (++) getLine getLine

GHCi> joinTwoLines
Hello
world
"Helloworld"
```

## Joining and flipping two lines

```hs
flipTwoLines :: IO String
flipTwoLines =
    liftM2 (\ x y -> y ++ x) getLine getLine

GHCi> flipTwoLines
Hello
world
"worldHello"
```

## Revisiting the problematic examples

Wrong:

```hs
program1 = getLine ++ getLine
program2 = (\ x -> x ++ x) getLine
program3 = (\ x y -> y ++ x) getLine getLine
```

Better:

```hs
joinTwoLines1 = liftM2 (++) getLine getLine
joinTwoLines2 = (\ x -> liftM2 (++) x x) getLine
joinTwoLines3 =
    (\ x y -> liftM2 (++) y x) getLine getLine
duplicateLine = liftM (\ x -> x ++ x) getLine
flipTwoLines =
    liftM2 (\ x y -> y ++ x) getLine getLine
```
