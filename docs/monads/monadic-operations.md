---
sidebar_position: 6
---


# Monadic operations

The advantages of an abstract interface
Several advantages to identifying the “monad” interface:

- Have to learn fewer names. Same return and (>>=) (and
do notation) in many different situations.
- Useful derived functions that only use return and (>>=) . All
these library functions become automatically available for every
monad.

## The advantages of an abstract interface

Several advantages to identifying the “monad” interface:

- Have to learn fewer names. Same return and (>>=) (and do notation) in many different situations.
- Useful derived functions that only use return and (>>=) . All these library functions become automatically available for every monad.
- There are many more monads than the ones we’ve discussed so far. Monads can be combined to form new monads.
- Application-specific code often uses just the monadic interface plus a few extra functions. As such, it is easy to switch the underlying monad of a large part of a program in order to accommodate a new aspect (error handling, logging, backtracking, ...).

## Useful monad operations

```hs
liftM :: (a -> b) -> IO a -> IO b
mapM :: (a -> IO b) -> [a] -> IO [b]
mapM_:: (a -> IO b) -> [a] -> IO ()
forM :: [a] -> (a -> IO b) -> IO [b]
forM_ :: [a] -> (a -> IO b) -> IO ()
sequence :: [IO a] -> IO [a]
sequence_:: [IO a] -> IO ()
forever :: IO a -> IO b
filterM :: (a -> IO Bool) -> [a] -> IO [a]
replicateM :: Int -> IO a -> IO [a]
replicateM_ :: Int -> IO a -> IO ()
when :: Bool -> IO () -> IO ()
unless :: Bool -> IO () -> IO ()
```

```hs
liftM :: Monad m => (a -> b) -> m a -> m b
mapM :: Monad m => (a -> m b) -> [a] -> m [b]
mapM_:: Monad m => (a -> m b) -> [a] -> m ()
forM :: Monad m => [a] -> (a -> m b) -> m [b]
forM_ :: Monad m => [a] -> (a -> m b) -> m ()
sequence :: Monad m => [m a] -> m [a]
sequence_:: Monad m => [m a] -> m ()
forever :: Monad m => a -> m b
filterM :: Monad m => (a -> m Bool) -> [a] -> m [a]
replicateM :: Monad m => Int -> m a -> m [a]
replicateM_ :: Monad m => Int -> m a -> m ()
when :: Monad m => Bool -> m () -> m ()
unless :: Monad m => Bool -> m () -> m ()
```

## Example: labelling a rose tree

```hs
data Rose a = Fork a [Rose a]
```

Each node has a (possibly empty) list of subtrees.

## Example: labelling a rose tree

```hs
data Rose a = Fork a [Rose a]
```

Each node has a (possibly empty) list of subtrees.

```hs
labelRose :: Rose a -> State Int (Rose (a, Int))
labelRose (Fork x cs) = do
  c <- get
  put (c + 1)
  lcs <- mapM labelRose cs
  return (Fork (x, c) lcs)
```

## Questions

What do you think these will evaluate to:

```hs
replicateM 2 [1 . . 3]
mapM return [1 . . 3]
sequence [[1, 2], [3, 4], [5, 6]]
mapM (flip lookup [(1, 'x'), (2, 'y'), (3, 'z')]) [1 . . 3]
mapM (flip lookup [(1, 'x'), (2, 'y'), (3, 'z')]) [1, 4, 3]
evalState (replicateM_ 5 (modify (+ 2)) >> get) 0
```

## About `liftM` and `fmap`

```hs
liftM :: (Monad m) => (a -> b) -> m a -> m b
fmap :: (Functor f) => (a -> b) -> f a -> f b
```

- Nearly same type as `fmap` , but a different class constraint.
- But every monad can be made an instance of Functor , by
defining `fmap` to be `liftM` .
- In practice, nearly all Haskell monads provide a Functor
instance. So you usually have `liftM` , `fmap` and (`<$>`)
available, all doing the same.

## A common pattern

Let’s once again look at tree labelling:

```hs
labelTree :: Tree a -> State Int (Tree (a, Int))
labelTree (Leaf x) = do
  c <- get
  put (c + 1) -- or modify (+ 1)
  return (Leaf (x, c))
labelTree (Node l r) = do
  ll <- labelTree l
  lr <- labelTree r
  return (Node ll lr)
```

We are returning an application of (constructor) function Node to the
results of monadic computations.

## A common pattern (contd.)

```hs
do
r1 <- comp1
r2 <- comp2
...
rn <- compn
return (f r1 r2...rn)
```

This isn’t type correct:

```hs
f comp1 comp2...compn
```

But we can get close:

```hs
f <$> comp1 <*> comp2... <*> compn
```

## Monadic application

We need a function that’s like function application, but works on monadic values:

```hs
ap :: Monad m => m (a -> b) -> m a -> m b
ap mf mx = do
f <- mf
x <- mx
return (f x)
```

Types supporting return and ap have their own name:

```hs
class Functor f => Applicative f where
pure :: a -> f a -- like return
(<*>) :: f (a -> b) -> f a -> f b -- like ap
```

## Functor and Applicative in terms of Monad

```hs
instance Monad T where ...
```

Requires superclass instances for Functor and Applicative :

```hs
instance Functor T where
  fmap = liftM
```

```hs
instance Applicative T where
  pure = return
  (<*>) = ap
```

### Example

```hs
labelTree :: Tree a -> State Int (Tree (a, Int))
labelTree (Leaf x) = do
c <- get
put (c + 1) -- or modify (+ 1)
return (Leaf (x, c))
labelTree (Node l r) =
Node <$> labelTree l <*> labelTree r
```

Exercise: Convince yourself that this is type correct.

Lessons

- The abstraction of monads is useful for a multitude of different types.
- Monads can be seen as tagging computations with effects.
- While IO is impure and cannot be defined in Haskell, the other effects we have seen can be modelled in a pure way:
  - exceptions via Maybe or Either ;
  - state via State ;
  - nondeterminism via [] .
- The monad interface offers a large number of useful abstractions that can all be applied to these different scenarios.
- All monads are also applicative functors and in particular functors. The (`<$>`) and (`<*>`) operations are also useful for structuring effectful code in Haskell.
