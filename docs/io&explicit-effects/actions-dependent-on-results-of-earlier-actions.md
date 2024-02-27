---
sidebar_position: 4
---

# Actions dependent on results of earlier actions

## Bind: letting an action use an earlier result

```hs
(>>=) :: IO a -> (a -> IO b) -> IO b
```

## Shouting back

Transforms the result (but does not print it back):

```hs
shout :: IO String
shout = liftM (map toUpper) getLine
shoutBack :: IO ()
shoutBack = shout >>= putStrLn
(>>=) :: IO a -> (a -> IO b) -> IO b
shout :: IO String
putStrLn :: String -> IO ()
shout >>= putStrLn :: IO ()
```

## Shouting back twice

```hs
shoutBackTwice :: IO ()
shoutBackTwice =
shout >>= \ x -> putStrLn x >> putStrLn x
```

### In GHCi

```hs
GHCi> shoutBack
Hello
Hello
GHCi> shoutBackTwice
can you hear me?
CAN YOU HEAR ME?
CAN YOU HEAR ME?
```

## Optioning out of doing IO

```hs
return :: a -> IO a
```

An plan that when executed, perform no effects and returns the given
result.

- Intuitively, `IO` a says that we **may** use effects to obtain an `a`. We are not required to.
- On the other hand, `a` says that we **must not** use effects to obtain an `a`.

## No escape from IO

There is no* function

```hs
runIO :: IO a -> a
```

If a value requires effects to obtain, we should not ever pretend that it
does not.

- : There actually is one, called unsafePerformIO , but its use is generally not justified.

## Escaping temporarily

```hs
(>>=) :: IO a -> (a -> IO b) -> IO b
```

- Gives us access to the a that results from the first action.
- But wraps it all up in another IO action.

## Bind is the most general sequencing function

```hs
(>>) :: IO a -> IO b -> IO b
a1 >> a2 = a1 >>= \_ -> a2
```

Or:

```hs
(>>) :: IO a -> IO b -> IO b
ioa >> iob = ioa >>= const iob
const :: a -> b -> a
const a b = a
```

## Bind and return can implement lifting

```hs
liftM :: (a -> b) -> IO a -> IO b
liftM f ioa = ioa >>= \ a -> return (f a)
liftM2 :: (a -> b -> c) -> IO a -> IO b -> IO c
liftM2 f ioa iob =
ioa >>= \ a -> iob >>= \ b -> return (f a b)
```

## do notation

```hs
liftM2 :: (a -> b -> c) -> IO a -> IO b -> IO c
```

```hs
liftM2 f ioa iob =
  ioa >>= \ a ->
  iob >>= \ b ->
  return (f a b)
```

```hs
liftM2 f ioa iob = do
  a <- ioa
  b <- iob
  return (f a b)
```

## A larger example

```hs
greeting :: IO ()
greeting =
  putStrLn "What is your name?" >>
  getLine >>= \ name ->
  putStrLn "Where do you live?" >>
  getLine >>= \ loc ->
  let
    answer
    | loc == "Regensburg" = "Fantastic!"
    | otherwise = "Sorry, don't know that."
  in
    putStrLn answer
```

```hs
greeting :: IO ()
greeting = do
putStrLn "What is your name?"
name <- getLine
putStrLn "Where do you live?"
loc <- getLine
let
  answer
    | loc == "Regensburg" = "Fantastic!"
    | otherwise           = "Sorry, don't know that."
putStrLn answer
```

## More about `do`

A corner case is a single IO action:

```hs
helloWorld = do
  putStrLn "Hello world"
```

is the same as writing

```hs
helloWorld =
  putStrLn "Hello world"
```

Remember that a do is never required. It is just “syntactic sugar” for a chain of `(>>=)`and `(>>)` applications.

## Nested do

```hs
loop :: IO ()
loop = do
  putStrLn "Type 'q' to quit."
  c <- getChar -- reads a single character
  if c == 'q'
    then putStrLn "Goodbye"
  else do
    putStrLn "Here we go again ..."
    loop
```

If we want to embed a sequence commands into a subexpression and use do -notation for that, we need another do .
Note furthermore that there are lots of do blocks without `return`.

## About `return`

- The purpose of `return` in Haskell is to embed computations into the IO type.
- As such, `return` can be used in many different places.
- It is fine to use `return` in the middle of a sequence of commands. It does not jump anywhere.

This is fine:

```hs
do
  n <- return 2
  print n
```
