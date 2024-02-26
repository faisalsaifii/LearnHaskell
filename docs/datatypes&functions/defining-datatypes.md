---
sidebar_position: 5
---

# Defining data types

## New datatypes with the data construct

```hs
data Weekday = Mo | Tu | We | Th | Fr | Sa | Su
```

This is an **enumeration** type. There are 7 **constructors**:

```hs
Mo, Tu, We, Th, Fr, Sa, Su :: Weekday
data Date = D Int Int Int -- year, month, day
```

This is a **record** type. It has a single constructor:

```hs
D :: Int -> Int -> Int -> Date
```

## (Data) Constructors

```hs
Mo :: Weekday
D :: Int -> Int -> Int -> Date
False :: Bool
(:) :: a -> [a] -> [a]
(,) :: a -> b -> (a, b)
Just :: a -> Maybe a
```

- Constructors are constants or functions that can be used to
construct terms on the right hand side of a declaration.
- They have types targetting the datatype they belong to.
- Constructors determine the shape of values – they are not
reduced, but evaluate to themselves.
- We can pattern-match on constructors (and not on ordinary constants or functions).

## Datatypes yield programming patterns

From the datatype definition, we can read off the standard design
principle for functions over the datatype:

- For each constructor, make a case.
- Use the arguments of the constructor on the right hand side.
- Whenever the datatype is recursive, consider making the function recursive.

## Booleans

```hs
data Bool = False | True
```

An enumeration type, like Weekday.

Two constructors, no recursion.

Example functions:

```hs
not :: Bool -> Bool
not False = True
not True = False
(&&) :: Bool -> Bool -> Bool
(&&) True True = True
(&&) __ = False
```

## Tuples

```hs
data (a, b) = (a, b)
data (a, b, c) = (a, b, c)
```

Parameterized. One constructor each. No recursion. Built-in syntax.

We could define our own, with less convenient syntax:

```hs
data Pair a b = MakePair a b
data Triple a b c = MakeTriple a b c
```

Example functions:

```hs
secondOfThree :: (a, b, c) -> b
secondOfThree (x, y, z) = y
secondOfThree' :: Triple a b c -> b
secondOfThree' (MakeTriple x y z) = y
```

## Maybe

```hs
data Maybe a = Nothing | Just a
```

Parameterized. Two constructors. No recursion.
Example function:

```hs
fromMaybe :: a -> Maybe a -> a
fromMaybe def Nothing = def
fromMaybe def (Just x) = x
```

## Lists

```hs
data [a] = [] | a : [a]
```

Parameterized. Two constructors. Recursive. Built-in syntax.
We could define our own, with less convenient syntax.

```hs
data List a = Nil | Cons a (List a)
```

We have seen lots of example functions following the standard design
principle.

## The `data` construct

The syntax of the **`data`** construct:

```hs
data Type arg1...argm = Con1 ty1 ... tyn
                      | Con2 ...
                      | ...
```

Introduces the new datatype Type and the data constructors Con1 ,
Con2 , ... .

Types of constructors are determined by the data declaration:

```hs
Con1 :: ty1 -> ... -> tyn -> Type arg1 ... argm
```

Type and constructor names must start with an uppercase letter;
symbolic infix constructors must start with a colon ( : ). Lists and
tuples support additional built-in syntax that cannot be used for other
datatypes.

## Applying the design principle

Also for new datatypes, always keep in mind that by looking at the
datatype, you obtain a design principle for functions over that type:

```hs
data Weekday = Mo | Tu | We | Th | Fr | Sa | Su

```hs
isWeekend :: Weekday -> Bool
isWeekend Mo = False
isWeekend Tu = False
isWeekend We = False
isWeekend Th = False
isWeekend Fr = False
isWeekend Sa = True
isWeekend Su = True
```

```hs
isWeekend :: Weekday -> Bool
isWeekend Sa = True
isWeekend Su = True
isWeekend _ = False
```

Collapsing cases – the order of cases then matters!

## Another example

```hs
data Date = D Int Int Int -- year, month, day
```

One constructor. No recursion.

Example function:

```hs
valid :: Date -> Bool
valid (D y m d) =
  m >= 1 && m <= 12 && d >= 1 && d <= 31
```

Of course, this is not an optimal definition.

## Type synonyms with `type`

Often, for datatypes with a single constructor, the constructor is
named the same as the datatype itself:

```hs
data Date = Date Int Int Int
```

It’s often better to give more meaningful names to types without
creating a completely new type:

```hs
type Year = Int
type Month = Int
type Day = Int
data Date = Date Year Month Day
```

Note that `type` introduces type synonyms. For example,
`2 :: Year and 2 :: Int`. No conversion function is required.

## Renamed types with `newtype`

We could also define:

```hs
data Year = Year Int
```

Now Year and Int are **different**:

```hs
Year :: Int -> Year -- the constructor
```

To extract the Int from a year, we can use **pattern matching**:

```hs
fromYear :: Year -> Int
fromYear (Year n) = n
```

For the case of a single-constructor, single-argument datatype (i.e., a
**renamed** type), there’s a more efficient construct:

```hs
newtype Year = Year Int
```

## Lessons

- Let the types guide you.
- Use pattern matching to get at components of values, and to
distinguish cases.
- Try to follow the recursive structure of types (lists, trees).
- Cover all cases.
- Use precise types, such as Maybe , rather than causing
uncontrolled errors.
