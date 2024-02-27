---
sidebar_position: 2
---

# Explicit effects

## The original motivation for explicit effects

- Given lazy evaluation as a strategy, the moment of evaluation is not easy to predict and hence not a good trigger for side-effecting actions.
- Even worse, it may be difficult to predict whether a term is evaluated at all.
- We would like to keep equational reasoning, and allow compiler optimisations such as
  - **strictness analysis** – evaluating things earlier than needed if they will definitely be needed, or
  - **speculative evaluation** – evaluating things even if they might not be needed at all.

## The classic approach

In most languages, **execution** of side effects is tied to **evaluation** of the side-effecting expression.

This is feasible for languages with eager evaluation, because the order in which expressions are written down corresponds closely to the resulting order of evaluation.

With lazy evaluation, this is not the case ...

## Problematic programs

Assume for the time being:

```hs
getLine :: String
```

Consider:

```hs
program1 =
  let
    x = getLine
    y = getLine
  in
    x ++ y
```

```hs
program2 =
  let
    x = getLine
  in
    x ++ x
```

```hs
program3 =
  let
    x = getLine
    y = getLine
  in
    y ++ x
```

If evaluation triggers the effect and evaluation is lazy, then when and how far we look at the resulting string will determine if and when lines are being read.

Using equational reasoning, all three programs should mean the same.

## The Haskell approach

We do not tie the execution of side effects to evaluation.

We introduce a new datatype IO and make evaluation and execution separate concepts!

## Evaluation vs. execution

```hs
data IO a -- abstract
```

The type of **plans** to perform effects that ultimately yield an a .

- **Evaluation** does **not** trigger the actual effects. It will at most
evaluate the plan.
- **Execution** triggers the actual effects. Executing a plan is not
possible from within a Haskell program.

## The main program

```hs
main :: IO ()
```

- The entry point into the program is a plan to perform effects (a possibly rather complex one).
- This is the one and only plan that actually gets executed.

## The unit type

```hs
data () = () -- special syntax
```

Constructor:

```hs
() :: ()
```

- A type with a single value (nullary tuple).
- Often used to parameterize other types.
- A plan for actions with no interesting result: `IO ()`.

## Execution of effects via GHCi

For convenience, GHCi also executes IO actions:

```hs
GHCi> getLine
Some text.
"Some text."
```

```hs
getLine :: IO String
```

A plan that when executed, reads a line interactively and returns that line as a `String`.

## Execution of effects with unit results in GHCi

GHCi does not print the final result of IO () -typed actions:

```hs
GHCi> writeFile "test.txt" "Hello"
GHCi> putStrLn "two\nlines"
two
lines

writeFile :: FilePath -> String -> IO ()
putStrLn :: String -> IO ()
```

## Explicit effects are a good idea

(Not just in Haskell, not just in a lazily evaluated language.)

- We can see via the type of a program whether it is guaranteed to have no side effects, or whether it is allowed to use effects.
- In principle, we can even make more fine-grained statements than just yes or no, by allowing just specific classes of effects.
- Encourages a programming style that keeps as much as possible effect-free.
- Makes it easier to test programs, or to run them in a different context.
