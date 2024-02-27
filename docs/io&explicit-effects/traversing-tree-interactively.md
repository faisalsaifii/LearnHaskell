---
sidebar_position: 6
---

# Traversing a tree interactively

## A tree of yes-no questions

```hs
data Interaction =
      Question String Interaction Interaction
    | Result String
```

Constructors:

```hs
Question ::
String
-> Interaction -> Interaction -> Interaction
Result :: String -> Interaction
```

## Pick a language

```hs
pick :: Interaction
pick =
Question "Do you like FP?"
(Question "Do you like static types?"
(Result "Try OCaml.")
(Result "Try Clojure.")
)
(Question "Do you like dynamic types?"
(Result "Try Python.")
(Result "Try Rust.")
)
```

## Pick a car

```hs
ford :: Interaction
ford =
Question "Would you like a car?"
(Question "Do you like it in black?"
(Result "Good for you.")
ford
)
(Result "Never mind then.")
```

## Asking a Boolean question

```hs
askBool :: String -> IO Bool
askBool question = do
putStrLn (question ++ " [yN]")
x <- getChar
putStrLn ""
return (x `elem` "yY")
```

## Traversing the tree interactively

```hs
interaction :: Interaction -> IO ()
interaction (Question q y n) = do
b <- askBool q
if b then interaction y else interaction n
interaction (Result r) = putStrLn r
```

## Traversing the tree non-interactively

```hs
simulate :: Interaction -> [Bool] -> Maybe String
simulate (Question _y_) (True : bs) =
simulate y bs
simulate (Question **n) (False : bs) =
simulate n bs
simulate (Result r) [] = Just r
simulate** = Nothing
```
