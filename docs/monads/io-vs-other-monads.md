---
sidebar_position: 5
---

# IO vs. other monads

## The IO monad is special

- IO is a primitive type, and (>>=) and return for IO are
primitive functions,
- there is no (politically correct) function runIO :: IO a -> a ,
whereas for most other monads there is a corresponding
function, or at least some way to get an a out of the monad;
- values of IO a denote side-effecting programs that can be
executed by the run-time system.

## Effectful programming

- IO being special has little to do with it being a monad;
- you can use IO an functions on IO very much ignoring the
presence of the Monad class;
- IO is about allowing real side effects to occur; the other types
we have seen are entirely pure as far as Haskell is concerned,
even though they capture a form of effects.

## IO, internally

If you ask GHCi about IO by saying :i IO , you get

```hs
newtype IO a
= GHC.Types.IO (GHC.Prim.State# GHC.Prim.RealWorld
-> (# GHC.Prim.State# GHC.Prim.RealWorld, a #))
-- Defined in ‘GHC.Types ’
```

So internally, GHC models IO as a kind of state monad having the
“real world” as state!
