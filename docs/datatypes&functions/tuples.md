---
sidebar_position: 2
---

# Tuples

## Lists vs. tuples

It’s time to talk about a new (family of) datatypes: tuples.

- lists are a datatype that collects an arbitrary number of elements; all elements must be of the same type.
- tuples are a family of datatypes that collect a fixed number of elements; each element can have a different type.

Let’s look at examples.

## Example tuples

Pairing a Bool and a String :

```hs
example :: (Bool, String)
example = (True, "yes, it's true")
```

- The type states it is a pair, and also states the type of each
component.
- The corresponding expression has similar syntax, and constructs
a pair out of two components.

Here’s a triple:

```hs
triple :: ([a] -> [a], [b] -> Int, Char)
triple = (reverse, length, 'x')
```

## General structure of tuples

For each n >= 2 , there’s a type of n -tuples.

- Each of these is a different type.
- Unlike lists (or Bool ), each tuple type has a single constructor,
conveniently written using parentheses and commas, with the
arguments interspersed (it can also be used in prefix notation,
actually).
- We can use pattern matching to extract the components of a
tuple (but we do not need to distinguish several cases).
Remarks:
- There are no 1-tuples. Haskell treats `(Int)` and `Int` as the
same type.
- The unit type `()` can be seen as a 0-tuple.

## Example: selecting the first component of a pair

```hs
fst :: ...
```

What is the type here?

```hs
fst :: (Int, Char) -> Int
```

is certainly too specific ...

```hs
fst :: (a, b) -> a
```

We can accept arbitrary component types. The two components can
have different types. But the result type matches the type of the first component.

Now let’s apply pattern matching on the input.

```hs
fst :: (a, b) -> a
fst (x, y) = ...
```

Now x is the component of type a , and y the component of type b .
It’s nearly trivial to finish the definition.

```hs
fst :: (a, b) -> a
fst (x, y) = x
```

And we are done.

## Zipping two lists

Sometimes, we have two lists of equal length and want to combine
them element by element:

```hs
zip :: [a] -> [b] -> [(a, b)]
```

We start with the type.

It’s a function over (two) lists, so let’s apply the standard principle to the first list.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip [] ys = ...
zip (x : xs) ys = ... zip xs ...
```

We actually need to look at the second list, too. So let’s just match on that one as well.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip [] [] = ...
zip [] (y : ys) = ...
zip (x : xs) [] = ...
zip (x : xs) (y : ys) = ... zip xs ys ...
```

The first case is easy: if both lists are empty, we return the empty list.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip [] [] = []
zip [] (y : ys) = ...
zip (x : xs) [] = ...
zip (x : xs) (y : ys) = ... zip xs ys ...
```

In the final case, we can produce the first element of the resulting list and recurse.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip [] [] = []
zip [] (y : ys) = ...
zip (x : xs) [] = ...
zip (x : xs) (y : ys) = (x, y) : zip xs ys
```

In the other two cases, there’s a bit of flexibility:

- We could fail, yielding a partial function.
- But we can also just agree to return the shorter list.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip [] [] = []
zip [] (y : ys) = []
zip (x : xs) [] = []
zip (x : xs) (y : ys) = (x, y) : zip xs ys
```

This definition has the advantage that we can use an infinite list as one
argument:

```hs
zip [1..] listOfNames
```

```js
zip :: [a] -> [b] -> [(a, b)]
zip (x : xs) (y : ys) = (x, y) : zip xs ys
zip xs ys = []
```

We can actually collapse the first three cases into one, but now the
order of patterns matters.

Simple variables match everything.

```hs
zip :: [a] -> [b] -> [(a, b)]
zip (x : xs) (y : ys) = (x, y) : zip xs ys
zip _ _ = []
```

Pattern variables that are not used on the right hand side can be
replaced by underscores.

## Association lists

A list of pairs serves as a primitive way to associate keys with values.

```hs
numbers :: [(Int, String)]
numbers = [(1, "one"), (5, "five"), (42, "forty-two")]
```

Let’s try to write a lookup function that obtains the value associated
with a particular key ...

## Defining lookup – “bad” version

```hs
lookup :: key -> [(key, val)] -> val
```

A first approximation of the type.

Let’s analyze the input list.

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = ...
lookup x (y : ys) = ... lookup ... ys ...
```

What val can we return if we reach the empty list and haven’t found our key?

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x (y : ys) = ... lookup ... ys ...
```

A bad solution is to trigger a run-time exception. We’ll improve on that
shortly.

For the other case, we have to look at the first pair ...

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys) = ... lookup ... ys ...
```

Now we have to compare x and k . Let’s use guards.

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = ... lookup ... ys ...
  | otherwise = ... lookup ... ys ...
```

If we found the key, we can immediately return the value. (So what will
happen if the key occurs multiple times?)

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = ... lookup ... ys ...
```

In the remaining case, we simply recurse.

```hs
lookup :: key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = lookup x ys
```

Let’s take a final look. Oh, we need equality on the key type ...

```hs
lookup :: Eq key => key -> [(key, val)] -> val
lookup x [] = error "lookup: unknown key"
lookup x ((k, v) : ys)
  | x == k = v
  | otherwise = lookup x ys
```

Now we’re done, apart from the ugly call to error .

## Excursion: about error and undefined

A call to error (as well as a function call for which no pattern match succeeds) causes a run-time exception.

```hs
error :: String -> a
undefined :: a
```

Note that these are polymorphic in the result type. This means they can be used in any context, because they abort normal control flow.

## Excursion: total and partial functions

- A function that can trigger a run-time exception or that may loop is called a **partial** function.
- Writing and using partial functions is discouraged – always try to cover all cases and make your functions **total**.
- However, `undefined` and `error` can be useful tools while incrementally developing a program.
