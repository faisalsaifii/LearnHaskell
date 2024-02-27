---
sidebar_position: 4
---

# Important classes

## `Eq`

- For equality and inequality.
- Note that equality in Haskell is structural equality. There is no
“object identity”, and no pointer equality.
- Supported by most datatypes, such as numbers, characters, tuples, lists, `Maybe` , `Either` , ...
- Not supported for function types.

## `Ord`

For comparisons between values of the same type.

```hs
class Eq a => Ord a where
compare :: a -> a -> Ordering
(<) :: a -> a -> Bool
(<=) :: a -> a -> Bool
(>) :: a -> a -> Bool
(>=) :: a -> a -> Bool
max :: a -> a -> a
min :: a -> a -> a
```

Several default definitions – you’d typically define just `compare` or `(<=)`.

```hs
data Ordering = LT | EQ | GT
```

## Superclasses

```hs
class Eq a => Ord a where
...
```

The condition indicates that Eq is a **superclass** of Ord :

- You cannot give an instance for Ord without first providing an instance to Eq.
- Conversely, a constraint `Ord a => ...` on a function implies `Eq a` . In other words, `(Ord a, Eq a) => ...` is equivalent to
`Ord a => ...`.

## Overloading vs. parameterization

Consider:

```hs
sort :: Ord a => [a] -> [a]
sortBy :: (a -> a -> Ordering) -> [a] -> [a]
```

Both functions are rather similar:

- the first takes the comparison function to use from the **instance** declaration for the element type of the list;
- the second is passed an explicit comparison function.

Using an overloaded function is a bit more convenient, but using `sortBy` is a bit more flexible.

Interestingly, GHC implements overloaded functions by passing type
class “dictionaries” as additional arguments.

## Excursion: performance impact of overloading

- You pay no price whatsoever for parametric polymorphism.
- Overloaded functions get extra arguments at runtime. There is a
slight performance penalty for that.
- Only overloaded functions get extra arguments – remember that
there is no general run-time type information!
- It is possible to instruct GHC to generate specialized versions for
overloaded functions at particular types, thereby eliminating the
run-time overhead.
- GHC also has a relatively aggressive inliner. Inlining overloaded
functions can also remove the overhead, much like specialization.

## `Show`

```hs
class Show a where
show :: a -> String
showsPrec :: Int -> a -> ShowS
showList :: [a] -> ShowS
```

The most important method is `show`:

- used to produce a human-readable `String` -representation of a
value;
- it is sufficient to define `show` in new instances, as the others
have default definitions;
- the other two functions can be used to more efficiently and
beautifully implement `show` internally (for example, remove
unnecessary parentheses);
- also used by GHCi to print result values of evaluated terms;
- once again, function types are not an instance.

## `Read`

```hs
class Read a where
readsPrec :: Int -> ReadS a
readList :: ReadS [a]
```

Most often, the derived function read is used:

```hs
read :: Read a => String -> a
```

Tries to interpret a given `String` (such as produced by `show` ) as a value of a type.

How the value is interpreted is statically determined by the context:

```hs
read "1" + 2 -- used as a number, parsed as a number
not (read "False") -- used as a Bool , parsed as a Bool
```

## Unresolved overloading

The following function produces an error (not in GHCi, but if placed in a
file):

```hs
strange x = show (read x)
```

The error will say something about an “ambiguous type variable” and
mention constraints for Read and Show .
Can you imagine what the problem is?
The `x` is a `String` which is then parsed into something by `read` .
But what type should it be parsed at? The context does not tell,
because the result is passed to `Show` , which is also overloaded.

## Manually resolving overloading

This works:

```hs
strange :: String -> String
strange x = show (read x :: Bool)
```

Or this:

```hs
strange :: String -> String
strange x = show (read x :: Int)
```

But note that the choice of intermediate type does make a difference!

In general, if several overloaded functions are combined such that the
resulting type does not mention any overloaded variables anymore,
you have to specify the intermediate types manually to help the type
checker resolve the overloading.

## `deriving`

For a limited number of type classes (but in particular Eq , Ord ,
Show , Read ), the Haskell compiler has a built-in algorithm to derive
an instance for nearly any datatype.

deriving
For a limited number of type classes (but in particular `Eq` , `Ord` ,
`Show` , `Read` ), the Haskell compiler has a built-in algorithm to derive
an instance for nearly any datatype.

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
deriving (Eq, Ord, Show, Read)
```

Defines the `Tree` datatype of binary trees together with suitable
instances:

- equality is always deep and structural;
- ordering depends on the order of constructors;
- Show and Read assume the natural human-readable Haskell
string representation.
