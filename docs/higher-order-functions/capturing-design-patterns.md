---
sidebar_position: 2
---

# Capturing design patterns

## Abstraction

One of the strengths of Haskell’s flexibility with functions is that they
really allow to abstract from reoccuring patterns and thereby save
code.

The standard design principle for lists we’ve been using all the time
works as follows:

```hs
fun :: [someType] -> someResult
fun [] = ... -- code
fun (x : xs) = ... -- code that can use x and fun xs
```

## From an informal pattern to a function

```hs
fun :: [someType] -> someResult
fun [] = ... -- code
fun (x : xs) = ... -- code that can use x and fun xs
```

We have two interesting positions where we have to fill in
situation-specific code. Let’s abstract!

```hs
fun :: [someType] -> someResult
fun [] = nil
fun (x : xs) = cons x (fun xs)
```

- We give names to the cases that correspond to the constructors.
- The case cons can `use x` and `fun xs` , so we turn it into a
function.
- At the moment, this is not a valid function, because nil and
cons come out of nowhere – but we can turn them into
parameters of fun !

```hs
fun :: ... -> ... -> [someType] -> someResult
fun cons nil [] = nil
fun cons nil (x : xs) = cons x (fun cons nil xs)
```

We now have to look at the types of cons and nil:

- `nil` is used as a result, so `nil :: someResult` ;
- cons takes a list element and a result to a result, so
`cons :: someType -> someResult -> someResult`.

```hs
fun :: (someType -> someResult -> someResult)
  -> someResult
  -> [someType] -> someResult
fun cons nil [] = nil
fun cons nil (x : xs) = cons x (fun cons nil xs)
```

We can give shorter names to `someType` and `someResult` ...

```hs
fun :: (a -> r -> r) -> r -> [a] -> r
fun cons nil [] = nil
fun cons nil (x : xs) = cons x (fun cons nil xs)
```

This function is called foldr ...

```hs
foldr :: (a -> r -> r) -> r -> [a] -> r
foldr cons nil [] = nil
foldr cons nil (x : xs) = cons x (foldr cons nil xs)
```

We could equivalently define it using where ...

```hs
foldr :: (a -> r -> r) -> r -> [a] -> r
foldr cons nil = go
  where
    go [] = nil
    go (x : xs) = cons x (go xs)
```

The arguments `cons` and `nil` never change while traversing the
list, so we can just refer to them in the local definition go , without
explicitly passing them around.

## Using `foldr`

```hs
length :: [a] -> Int
length [] = 0
length (x : xs) = 1 + length xs
```

```hs
foldr :: (a -> r -> r) -> r -> [a] -> r
foldr cons nil = go
  where
    go [] = nil
    go (x : xs) = cons x (go xs)
``

```hs
length :: [a] -> Int
length [] = 0
length (x : xs) = 1 + length xs
```

```hs
foldr :: (a -> r -> r) -> r -> [a] -> r
foldr cons nil = go
  where
    go [] = nil
    go (x : xs) = cons x (go xs)
```

```hs
length = foldr (\ x r -> 1 + r) 0
```

or (using `const` and an operator section)

```hs
length = foldr (const (1 +)) 0
```

## Examples of using `foldr`

```hs
(++) :: [a] -> [a] -> [a]
(++) xs ys = foldr (:) ys xs
filter :: (a -> Bool) -> [a] -> [a]
filter p = foldr (\ x r -> if p x then x : r else r) []
map :: (a -> b) -> [a] -> [b]
map f = foldr (\ x r -> f x : r) []
```

```hs
(++) :: [a] -> [a] -> [a]
(++) xs ys = foldr (:) ys xs
filter :: (a -> Bool) -> [a] -> [a]
filter p = foldr (\ x r -> if p x then x : r else r) []
map :: (a -> b) -> [a] -> [b]
map f = foldr (\ x r -> f x : r) []
```

```hs
and :: [Bool] -> Bool
and = foldr (&&) True
any :: (a -> Bool) -> [a] -> Bool
any p = foldr (\ x r -> p x || r) False
```

And many more.

## The role of foldr

- When a list function is easy to express using `foldr` , then you
should.
- Makes it immediately recognizable for the reader that it follows
the standard design principle.
- Some functions can be expressed using `foldr` , but that does
not necessarily make them any clearer. In such cases, aim for
clarity.

## Accumulating parameter pattern

```hs
reverse :: [a] -> [a]
reverse = go []
  where
    go !acc [] = acc
    go !acc (x : xs) = go (x : acc) xs
sum :: Num a => [a] -> a
sum = go 0
  where
    go !acc [] = acc
    go !acc (x : xs) = go (x + acc) xs
```

See something to abstract here?

## Abstracting

```hs
fun :: [a] -> r
fun = go ...
  where
    go !acc [] = acc
    go !acc (x : xs) = go (... acc ... x ...) xs
```

We apply the same tactics as before: let’s abstract from the interesting
positions and introduce names.

```hs
fun :: [a] -> r
fun = go e
  where
    go !acc [] = acc
    go !acc (x : xs) = go (op acc x) xs
```

Now we need to introduce e and op as parameters.

```hs
fun :: ... -> ... -> [a] -> r
fun op e = go e
  where
    go !acc [] = acc
    go !acc (x : xs) = go (op acc x) xs
```

And we have to figure out the types (or let the compiler infer them).

```hs
fun :: (r -> a -> r) -> r -> [a] -> r
fun op e = go e
  where
    go !acc [] = acc
    go !acc (x : xs) = go (op acc x) xs
```

This function is called `foldl'` (in the module `Data.List` ).

```hs
foldl' :: (r -> a -> r) -> r -> [a] -> r
foldl' op e = go e
  where
    go !acc [] = acc
    go !acc (x : xs) = go (op acc x) xs
```

This function is called `foldl'` (in the module `Data.List` ).

## foldr and foldl'

```hs
foldr (⊕) e [x, y, z] = x ⊕ (y ⊕ (z ⊕ e))
foldl' (⊕) e [x, y, z] = ((e ⊕ x) ⊕ y) ⊕ z
```

:::note[Performance advice]
There’s a function `foldl` with the same type as `foldl'`, which
does not have the strict accumulator and should therefore almost
never be used!
:::
