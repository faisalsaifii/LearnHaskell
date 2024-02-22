---
sidebar_position: 2
---

# Expressions

- Programs structured into modules (files)
- Modules contain declarations (statements).
- The most important form of declarations are bindings for new
constants and functions.
- In such bindings, expressions play the central role.

Therefore we are going to look at expressions before everything else.

A Haskell expression is a (possibly nested) terms built up from
constants and function calls.

- An important property of expressions is that they can be
evaluated, yielding a value.
- Values are themselves expressions that cannot be evaluated any
further.
- You can type expressions into GHCi. GHCi will then try to evaluate
the expression and print its resulting value

## Examples of values

```haskell
GHCi> 2
2

GHCi> 'x'
'x'

GHCi> "Haskell"
"Haskell"

GHCi> True
True

GHCi> [1,2,4]
[1,2,4]
```

## Examples of function calls

```haskell
GHCi> not True
False

GHCi> min 7 2
2

GHCi> 2 + 3
5

GHCi> 3 : [10,99]
[3,10,99]

GHCi> take 2 [1,3,9,27,81]
[1,3]

GHCi> map odd [1,2,3,4,5]
[True,False,True,False,True]
```

## Operators are functions

Only syntactic differences between symbolic and alphanumeric
function names.

**Symbolic identifiers** (operators) are **infix** by default, and can be made prefix by enclosing them in parentheses.

```haskell
GHCi> 6 + 9
15

GHCi> (+) 6 9
15
```

**Alphanumeric identifiers** are **prefix** by default, and can be made
infix by enclosing them in backquotes.

```haskell
GHCi> max 12 20
20

GHCi> 12 `max` 20
20
```

**Space** is function application

```haskell
min 7 2 -- function applied to two arguments
```

**Parentheses** are used for grouping

```haskell
GHCi> min 7 (2 + 6)
7

GHCi> min 7 2 + 6
8
```

Function application binds stronger than operators.

```haskell
GHCi> reverse (reverse [1,2,3])
[1,2,3]

GHCi> sum (filter odd [1,2,3,4,5])
9

GHCi> take 1 "Haskell" ++ drop 4 "Haskell"
"Hell"
```

## Anonymous functions (or lambda terms)

Referring to functions without giving them a name:

```haskell
GHCi> (\ x -> x + 3) 4
7

GHCi> (\ list n -> take n (reverse list)) "hello" 3
"oll"
```

The \ is pronounced “lambda”.

Particularly useful as an argument to another function:

```haskell
GHCi> map (\ x -> 3 * x + 1) [1,2,3]
[4,7,10]
```

Functions taking other functions as arguments are called **higher-order
functions**.

We have learned about:

- Expressions and values,
- Functions and operators,
- Numbers, characters, Booleans, lists (and strings),
- Lambda terms.

In particular, try to get used to the function application syntax (using
space) and the use of parentheses only for grouping.
