---
sidebar_position: 6
---

# Numbers

## Numeric types and classes

There are several numeric types and classes in Haskell:

```hs
type instance of
Int Num Integral
Integer Num Integral
Float Num Fractional Floating RealFrac
Double Num Fractional Floating RealFrac
Rational Num Fractional
```

The class `Num` is a **superclass** of `Integral`.
The class `Fractional` is a superclass of `Floating`.

- Whereas `Int` is bounded, `Integer` is unbounded (bounded
by memory only).
- A `Double` is usually of higher precision than a `Float`.
- The datatype `Rational` is for fractions.

## Operations on numbers

Most operations on numbers and even numeric literals are **overloaded**:

```hs
(+) :: (Num a) => a -> a -> a
(-) :: (Num a) => a -> a -> a
(_) :: (Num a) => a -> a -> a

1 :: (Num a) => a -- overloaded literals
1.2 :: (Fractional a) => a -- overloaded literals

(/) :: (Fractional a) => a -> a -> a
mod :: (Integral a) => a -> a -> a
div :: (Integral a) => a -> a -> a
sin :: (Floating a) => a -> a
log :: (Floating a) => a -> a
```

## No automatic coercion

We can use overloaded functions at different types:

```hs
3 * 4
3.2 * 4.5
```

But there is no implicit coercion:

```hs
3.2 * (5 `div` 2) -- type error
3.2 * fromIntegral (5 `div` 2)
```

:::note[Question]
Why is `3.2 * 2` ok, but not 3.2 * (5 \`div\` 2)?

:::

Because `2 :: (Num a) => a`, but (5\`div\`2) :: (Integral a) => a.

## Converting between numeric types

From an integral type to another:

```hs
fromIntegral :: (Integral a, Num b) => a -> b
```

From a fractional type to an integral:

```hs
round :: (RealFrac a, Integral b) => a -> b
floor :: (RealFrac a, Integral b) => a -> b
ceiling :: (RealFrac a, Integral b) => a -> b
```

Here, `round` rounds to the nearest even number.
