---
sidebar_position: 3
---

# Higher Order functions

## Functions, functions, functions

A function parameterized by another function or returning a function
is called a **higher-order function**.

:::note[Currying]
Strictly speaking, every curried function in Haskell is a function
returning another function:

```hs
elem :: Eq a => a -> ([a] -> Bool)
elem 3 :: (Eq a, Num a) => [a] -> Bool
```

:::

## Filtering and mapping

Two of the most useful list functions are higher-order, as they each
take a function as an argument:

```hs
filter :: (a -> Bool) -> ([a] -> [a])
map :: (a -> b) -> ([a] -> [b])
```

The use of a function `a -> Bool` to express a predicate is generally common. And mapping a function over a data structure is an operation that isn’t limited to lists.

## Folds

The functions `foldr` and `foldl'` are among the most general
higher-order functions on lists. Nearly any other list function can be
expressed in terms of them.

Some rules of thumb:

- If a more specific function applies (such as `filter` or `map` ),
use that.
- If a function becomes more readable by using a fold, then use it.
It has the advantage that it also signals to the programmer that a
common pattern is being used and nothing special is going on.
- If a function becomes harder to read by using a fold, then do not
force it into that pattern (except perhaps for training purposes).

## Function composition

One of the most ubiquitous higher-order functions is function
composition:

```hs
(.) :: ...
(f . g) x = f (g x)
```

For once – rather than starting from a type – let’s infer the type from
the code.

```hs
(.) :: ... -> ... -> ... -> ...
(f . g) x = f (g x)
```

It’s apparently a curried function taking three arguments `f` , `g` and `x` .

```hs
(.) :: (... -> ...) -> (... -> ...) -> ... -> ...
(f . g) x = f (g x)
```

Both `f` and `g` are applied to something, so they must be functions.

```hs
(.) :: (... -> ...) -> (... -> ...) -> a -> ...
(f . g) x = f (g x)
```

No requirements seem to be made about the type of x , except that
its passed to g , so let’s assume a type variable here ...

```hs
(.) :: (... -> ...) -> (a -> ...) -> a -> ...
(f . g) x = f (g x)
```

... which then should be the source type of g as well.

```hs
(.) :: (b -> ...) -> (a -> b) -> a -> ...
(f . g) x = f (g x)
```

The target type of g should match the source type of f .

```hs
(.) :: (b -> c) -> (a -> b) -> a -> c
(f . g) x = f (g x)
```

The target type of f is also the type of the overall result.

```hs
(.) :: (b -> c) -> (a -> b) -> (a -> c)
(f . g) x = f (g x)
```

Putting extra parentheses in the type may make it more obvious that
we are indeed composing two matching functions.

## Composing functions

We can often build functions from existing functions simply by
composing them.

Example: Computing the first 100 odd square numbers.

```hs
example :: [Int]
example = [1 . .]
```

We start by generating all numbers (lazy evaluation in action).

```hs
example :: [Int]
example = map (\ x -> x * x) [1 . .]
```

We use map to compute the square numbers. Note that map and
filter are often used with anonymous functions.

```hs
example :: [Int]
example = ( filter odd . map (\ x -> x * x)) [1 . .]
```

We use function composition (and partial application) to subsequently filter the odd square numbers.

```hs
example :: [Int]
example = (take 100 . filter odd . map (\ x -> x * x)) [1 . .]
```

Finally, we use composition again to take the first 100 elements of this
list.

## Composition as a design pattern

- Function composition gives you a way to split one programming problem into several, possibly smaller, programming problems.
- In general, higher-order functions are part of your toolbox for attacking programming problems. Recognizing something as a `map` or `filter` is also useful.
- Of course, you should never forget the standard design principle of following the datatype structure as a good way of defining most functions, if applying a higher-order function fails.

## Lessons

- Function composition is a bit like the functional semicolon. It
allows us to decompose larger tasks into smaller ones.
- Lazy evaluation allows us to separate the generation of possible
results from selecting interesting results. This allows more
modular programs in many situations.
- Partial application and anonymous functions help to keep such
composition chains concise.
