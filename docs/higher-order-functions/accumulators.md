---
sidebar_position: 1
---

# Accumulators

## Efficiency of reverse

Let’s look at the standard solutions for `(++)` and `reverse` earlier:

```hs
(++) :: [a] -> [a] -> [a]
[] ++ ys = ys
(x : xs) ++ ys = x : (xs ++ ys)
reverse :: [a] -> [a]
reverse [] = []
reverse (x : xs) = reverse xs ++ [x]
```

What is the efficiency of `reverse`?

Answer: it has quadratic complexity, as `(++)` is linear in its left argument and `reverse` reduces to a linear chain of `(++)` invocations.

## A better reverse

Let’s build the reversed list as we go in an additional, accumulating argument:

```hs
reverseAcc :: [a] -> [a] -> [a]
reverseAcc acc [] = acc
reverseAcc acc (x : xs) = reverseAcc (x : acc) xs
reverse :: [a] -> [a]
reverse = reverseAcc []
```

:::note

- We need the extra creative step to move from reverse to
reverseAcc ,
- But the function reverseAcc uses the standard design principle
for lists again;
- We now traverse the list only once.

:::

## Accumulators in general

We have seen that `reverse` benefits significantly from introducing
an accumulator.

Can the same idea be applied elsewhere?

## The `sum` function

Applying the standard design pattern:

```hs
sum :: [Int] -> Int
sum [] = 0
sum (x : xs) = x + sum xs
```

Trying in GHCi with artificially limited stack size (ghci +RTS -K1M):

```hs
GHCi> sum [1 . . 100000]
*** Exception: stack overflow
```

Why is the function consuming so much stack space?

## Equational reasoning

```hs
sum [1, 2, 3]
= sum (1 : 2 : 3 : []) -- removing syntactic sugar
= 1 + sum (2 : 3 : []) -- by definition of sum
= 1 + (2 + sum (3 : [])) -- by definition of sum
= 1 + (2 + (3 + sum [])) -- by definition of sum
= 1 + (2 + (3 + 0)) -- by definition of sum
= 1 + (2 + 3) -- by definition of (+)
= 1 + 5 -- by definition of (+)
= 6 -- by definition of (+)
```

We build an entire right-nested chain of additions until we reach the
end of the list. Only then can we start reducing them.

## An accumulator for the sum?

Can an accumulator help? Let’s try:

```hs
sumAcc :: Int -> [Int] -> Int
sumAcc acc [] = acc
sumAcc acc (x : xs) = sumAcc (x + acc) xs
sum :: [Int] -> Int
sum = sumAcc 0
```

Trying in stack-limited GHCi again:

```hs
GHCi> sum [1 . . 100000]
*** Exception: stack overflow
```

## More equational reasoning

```hs
sum [1, 2, 3]
= sum (1 : 2 : 3 : []) -- removing syntactic sugar
= sumAcc 0 (1 : 2 : 3 : []) -- by definition of sum
= sumAcc (0 + 1) (2 : 3 : []) -- by definition of sumAcc
= sumAcc ((0 + 1) + 2) (3 : []) -- by definition of sumAcc
= sumAcc (((0 + 1) + 2) + 3) [] -- by definition of sumAcc
= ((0 + 1) + 2) + 3 -- by definition of sumAcc
= (1 + 2) + 3 -- by definition of (+)
= 3 + 3 -- by definition of (+)
= 6 -- by definition of (+)
= 1 + (2 + (3 + sum [])) -- by definition of sum
= 1 + (2 + (3 + 0)) -- by definition of sum
= 1 + (2 + 3) -- by definition of (+)
= 1 + 5 -- by definition of (+)
= 6 -- by definition of (+)
```

## The problem

We build the expression

```hs
sumAcc (0 + 1) (2 : 3 : [])
```

Perhaps surprisingly, 0 + 1 is not reduced, because it is not needed
until much later.

We therefore build a (left-nested) chain of additions again until we
reach the end of the list ...

Accumulators are problematic in a lazy evaluation scenario: we
update them often, but do not really need them evaluated for a long
time.

Perhaps we can provide a hint to the compiler that we want this
evaluated sooner?

## Making the accumulator strict with bang patterns

```hs
sumAcc :: Int -> [Int] -> Int
sumAcc !acc [] = acc
sumAcc !acc (x : xs) = sumAcc (x + acc) xs
sum :: [Int] -> Int
sum = sumAcc 0
```

Note that we write `!acc` now. A bang pattern means that GHC will
evaluate the passed value to its outermost constructor (but only to
that!) even though the pattern is just a variable and would normally
match without evaluation.

For an `Int`, evaluating to the outermost constructor means
evaluating it completely.

## Bang patterns language extension

Bang patterns require the BangPatterns language extension, which
can be enabled via a compiler pragma:

```hs
{-# LANGUAGE BangPatterns #-}
```

at the top of the file.

## Problem fixed

In our stack-restricted GHCi:

```hs
GHCi> sum [1 . . 100000]
5000050000
```

Or even:

```hs
GHCi> sum [1 . . 10000000]
50000005000000
```

This version of `sum` actually runs in constant space.
The equational reasoning is similar to the previous accumulating
version, only that the accumulator is now evaluated immediately on
every update.

## Make accumulators strict

Accumulators should nearly always be strict.
Make them strict by default.
There are few situations (such as `reverse`) where strictness of the
accumulator is not required, but even there it is not harmful.

## Always accumulators?

**No!**

The accumulating parameter pattern always has to traverse the entire
list before producing a result! It cannot stop early or produce results
incrementally. It can also not work for infinite lists:

```hs
GHCi> and (False : repeat True)
False
GHCi> take 10 (map (+ 1) [1 . . 10])
[2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
```

Both of these functions should be defined using the standard design
pattern for lists – without an accumulator – to enable the behaviour
observed above.
