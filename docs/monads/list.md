---
sidebar_position: 3
---

# List

## Encoding multiple results and nondeterminism

Get the length of all words in a list of multi-line texts:

```hs
map length
(concat (map words
(concat (map lines txts))
))
```

Embedding and sequencing for computations with many results
nondeterministic computations:

- Embedding: a computation with exactly one result.
- Sequencing: performing the second computation on all possible
results of the first one.

## Defining bind and return for lists

```hs
(>>=) :: [a] -> (a -> [b]) -> [b]
xs >>= f = concat (map f xs)
return :: a -> [a]
return x = [x]
```

We have to use concat in (>>=) to flatten the list of lists.

## Using bind and return for lists

```hs
map length
(concat (map words
(concat (map lines txts))))
```

```hs
txts >>= \ t ->
lines t >>= \ l ->
words l >>= \ w ->
return (length w)
```

```hs
t := txts
l := lines t
w := words w
return length w
```

- Again, we have a similarity to imperative code.
- Imperative language: implicit nondeterminism.
- Haskell: explicit by using the list datatype and (>>=) .

## Intermediate Summary

At least four types with (>>=) and return :

- Maybe : (>>=) sequences operations that may fail and
shortcuts evaluation once failure occurs; return embeds a
function that never fails;
- State : (>>=) sequences operations that may modify some
state and threads the state through the operations; return
embeds a function that never modifies the state;
- [] : (>>=) sequences operations that may have multiple
results and executes subsequent operations for each of the
previous results; return embeds a function that only ever has
one result.
- IO : (>>=) sequences the side effects to the outside world,
and return embeds a function without any side effects.
There is a common interface here!
