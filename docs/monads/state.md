---
sidebar_position: 2
---

# State

Maintaining state explicitly

- We pass state to a function as an argument.
- The function modifies the state and produces it as a result.
- If the function does anything except modifying the state, we must
return a tuple (or a special-purpose datatype with multiple fields).
This motivates the following type definition:

```hs
type State s a = s -> (a, s)
```

## Using state

There are many situations where maintaining state is useful:

- using a random number generator
type Random a = State StdGen a
- using a counter to generate unique labels
type Counter a = State Int a
- maintaining the complete current configuration of an application
(an interpreter, a game, ...) using a user-defined datatype
data ProgramState = ...
type Program a = State ProgramState a

## Example: labelling the leaves of a tree

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
labelTree :: Tree a -> State Int (Tree (a, Int))
labelTree (Leaf x) c = (Leaf (x, c), c + 1)
labelTree (Node l r) c1 = let (ll, c2) = labelTree l c1
(lr, c3) = labelTree r c2
in (Node ll lr, c3)
```

## Encoding state passing

\ s1 -> let (lvl , s2) = generateLevel s1
(lvl', s3) = generateStairs lvl s2
(ms , s4) = placeMonsters lvl' s3
in (combine lvl' ms, s4)
Again, we need

- a way to sequence function calls and use their results
- a way to modify or produce successful results.

## Bind and return for state

```hs
\ s1 -> let (lvl , s2) = generateLevel s1
(lvl', s3) = generateStairs lvl s2
(ms , s4) = placeMonsters lvl' s3
in (combine lvl' ms, s4)
(>>=) :: State s a -> (a -> State s b) -> State s b
f >>= g = \ s -> let (x, s') = f s in g x s'
return :: a -> State s a
return x = \ s -> (x, s)
```

```hs
generateLevel >>= \ lvl ->
\ s2 -> let (lvl', s3) = generateStairs lvl s2
(ms , s4) = placeMonsters lvl' s3
in (combine lvl' ms, s4)
(>>=) :: State s a -> (a -> State s b) -> State s b
f >>= g = \ s -> let (x, s') = f s in g x s'
return :: a -> State s a
return x = \ s -> (x, s)
```

```hs
generateLevel >>= \ lvl ->
generateStairs lvl >>= \ lvl' ->
\ s3 -> let (ms , s4) = placeMonsters lvl' s3
in (combine lvl' ms, s4)
(>>=) :: State s a -> (a -> State s b) -> State s b
f >>= g = \ s -> let (x, s') = f s in g x s'
return :: a -> State s a
return x = \ s -> (x, s)
```

```hs
generateLevel >>= \ lvl ->
generateStairs lvl >>= \ lvl' ->
placeMonsters lvl' >>= \ ms ->
\ s4 -> (combine lvl' ms, s4)
(>>=) :: State s a -> (a -> State s b) -> State s b
f >>= g = \ s -> let (x, s') = f s in g x s'
return :: a -> State s a
return x = \ s -> (x, s)
```

```hs
generateLevel >>= \ lvl ->
generateStairs lvl >>= \ lvl' ->
placeMonsters lvl' >>= \ ms ->
return (combine lvl' ms)
(>>=) :: State s a -> (a -> State s b) -> State s b
f >>= g = \ s -> let (x, s') = f s in g x s'
return :: a -> State s a
return x = \ s -> (x, s)
```

## Observation

Again, the code looks a bit like imperative code. Compare:

```hs
generateLevel >>= \ lvl ->
generateStairs lvl >>= \ lvl' ->
placeMonsters lvl' >>= \ ms ->
return (combine lvl' ms)
```

```hs
lvl := generateLevel;
lvl' := generateStairs lvl;
ms := placeMonsters lvl';
return combine lvl' ms
```

- In the imperative language, the occurrence of memory updates
(random numbers) is a side effect.
- Haskell is more explicit because we use the State type and the
appropriate sequencing operation.

## “Primitive” operations for state handling

We can completely hide the implementation of State if we provide
the following two operations as an interface:

```hs
get :: State s s
get = \ s -> (s, s)
put :: s -> State s ()
put s = \ _ -> ((), s)
inc :: State Int ()
inc = get >>= \ s -> put (s + 1)
```

## Labelling a tree, revisited

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)
labelTree :: Tree a -> State Int (Tree (a, Int))
labelTree (Leaf x) c = (Leaf (x, c), c + 1)
labelTree (Node l r) c1 = let (ll, c2) = labelTree l c1
(lr, c3) = labelTree r c2
in (Node ll lr, c3)
```

The old version, with tedious explicit threading of the state.

```hs
data Tree a = Leaf a | Node (Tree a) (Tree a)

labelTree :: Tree a -> State Int (Tree (a, Int))
labelTree (Leaf x) = get >>= \ c ->
inc >> return (Leaf (x, c))
labelTree (Node l r) = labelTree l >>= \ ll ->
labelTree r >>= \ lr ->
return (Node ll lr)
```

New version, with implicit state passing, yet explicit sequencing.

```hs
(>>) :: State s a -> State s b -> State s b
x >> y = x >>= \ _ -> y
```

(The same definition as for IO ...)
