---
sidebar_position: 5
---

# Functional programming with IO

## Asking a question

```hs
ask :: String -> IO String
ask question = do
    putStrLn question
    getLine

GHCi> ask "What is your name?"
What is your name?
Andres
"Andres"
```

## Asking many questions

```hs
askMany :: [String] -> IO [String]
askMany [] = return []
askMany (q : qs) = do
    answer <- ask q
    answers <- askMany qs
    return (answer : answers)
```

The **standard design pattern** on lists is back!

## Feels like a map

A `map` has the wrong result type:

```hs
askMany' :: [String] -> [IO String]
askMany' = map ask
```

But we can sequence a list of plans:

```hs
sequence :: [IO a] -> IO [a]
sequence [] = return []
sequence (x : xs) = do
    a <- x
    as <- sequence xs
    return (a : as)
```

## Mapping an IO action

```hs
mapM :: (a -> IO b) -> [a] -> IO [b]
mapM f xs = sequence (map f xs)

askMany :: [String] -> IO [String]
askMany questions = mapM ask questions
```
