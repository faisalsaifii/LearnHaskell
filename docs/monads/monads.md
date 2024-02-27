---
sidebar_position: 4
---

# Monads

## Monad class

```hs
class Applicative m => Monad m where
return :: a -> m a
(>>=) :: m a -> (a -> m b) -> m b
```

- The name “monad” is borrowed from category theory.
- A monad is an algebraic structure similar to a **monoid**.
- Monads have been popularized in functional programming via the work of Moggi and Wadler.

## Instances

```hs
instance Monad Maybe where
...
instance Monad (Either e) where
...
instance Monad [] where
...
newtype State s a = State {runState :: s -> (a, s)}
instance Monad (State s) where
...
```

The newtype for State is required because Haskell does not allow
us to directly make a type s -> (a, s) an instance of Monad .
(Question: why not?)

## There are more monads

The types we have seen: Maybe , Either , [] , State , IO are
among the most frequently used monads – but there are many more
you will encounter sooner or later.

In fact, we have already seen one more! Which one?

The generators Gen from QuickCheck form a monad. You can see it
as an abstract state monad, allowing access to the state of a random
number generator.

## Monad laws

```hs
return is the unit of (>>=)
return a >>= f = f a
m >>= return = m
Associativity of (>>=)
(m >>= f) >>= g = m >>= (\ x -> f x >>= g)
```

## Monad laws for `Maybe`

```hs
return a >>= f
= { Definition of (>>=) }
case return a of
Nothing -> Nothing
Just x -> f x
= { Definition of return }
case Just a of
Nothing -> Nothing
Just x -> f x
= { case }
f a
```

## Monad laws for Maybe (contd.)

```hs
m >>= return
= { Definition of (>>=) }
case m of
Nothing -> Nothing
Just x -> return x
= { Definition of return }
case m of
Nothing -> Nothing
Just x -> Just x
= { case }
m
```

:::note[Lemma]

```hs
forall (f :: a -> Maybe b) . Nothing >>= f = Nothing
```

:::

:::note[Proof]

```hs
Nothing >>= f
= { Definition of (>>=) }
case Nothing of
Nothing -> Nothing
Just x -> f x
= { case }
Nothing
```

:::

```hs
(m >>= f) >>= g = m >>= (\ x -> f x >>= g)
```

Induction on m . Case m is Nothing :

```hs
(Nothing >>= f) >>= g
= { Lemma }
Nothing >>= g
= { Lemma }
Nothing
= { Lemma }
Nothing >>= (\ x -> f x >>= g)
```

```hs
(Just y >>= f) >>= g
= { Definition of (>>=) }
(case Just y of
Nothing -> Nothing
Just x -> f x) >>= g
= { case }
f y >>= g
= { beta-expansion }
(\ x -> f x >>= g) y
= { case }
case Just y of
Nothing -> Nothing
Just x -> (\ x -> f x >>= g) x
= { definition of (>>=) }
Just y >>= (\ x -> f x >>= g)
```

## Additional monad operations

Class Monad contains two additional methods, but with default
methods:

```hs
class Monad m where
...
(>>) :: m a -> m b -> m b
m >> n = m >>= \ _ -> n
fail :: String -> m a
fail s = error s
```

While the presence of (>>) can be justified for efficiency reasons,
the presence of fail is often considered to be a design mistake.

## do notation

The `do` notation we have introduced when discussing IO is available for all monads:

```hs
generateLevel >>= \ lvl ->
generateStairs lvl >>= \ lvl' ->
placeMonsters lvl' >>= \ ms ->
return (combine lvl' ms)
```

```hs
do
lvl <- generateLevel
lvl' <- generateStairs lvl
ms <- placeMonsters lvl'
return (combine lvl' ms)
```

## do notation – contd

```hs
up l1 >>= \ l2 ->
right l2 >>= \ l3 ->
down l3 >>= \ l4 ->
return (update (+ 1) l4)
```

```hs
do
l2 <- up l1
l3 <- right l2
l4 <- down l3
return (update (+ 1) l4)
```

## Tree labelling, revisited once more

Using Control.Monad.State and do notation:

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
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

How to get at the final tree?

## Running a stateful computation

```hs
evalState :: State s a -> s -> a
labelTreeFrom0 :: Tree a -> Tree (a, Int)
labelTreeFrom0 t = evalState (labelTree t) 0
```

There’s also

```hs
runState :: State s a -> s -> (a, s)
```

(which is just unpacking State ’s newtype wrapper).

## List comprehensions

```hs
map length
  (concat (map words (concat (map lines txts))))

do
t <- txts
l <- lines t
w <- words l
return (length w)
```

Also list comprehensions:

```hs
[length w | t <- txts, l <- lines t, w <- words l]
```

## More on `do` notation (and list comprehensions)

- Use it, the special syntax is usually more concise.
- Never forget that it is just syntactic sugar. Use (>>=) and (>>) directly when it is more convenient.

And some things I’ve already said about IO :

- Remember that return is just a normal function:
  - Not every do -block ends with a return .
  - return can be used in the middle of a do -block, and it doesn’t “jump” anywhere.
- Not every monad computation has to be in a do -block. In
particular do e is the same as e .
- On the other hand, you may have to “repeat” the do in some
places, for instance in the branches of an if .
