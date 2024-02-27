---
sidebar_position: 8
---

# Exceptions

## What happens if the file does not exist?

```hs
GHCi> readFileLineByLine "doesnotexist"
*** Exception: doesnotexist: openFile: does not exit
(No such file or directory)
```

## Exceptions in effectful vs effect-free code

Exceptions in pure code (via `error` , missing patterns, ...) are bad:

- It is unclear when exactly, or if, they will be triggered,
- It is therefore also unclear where or when to best handle them,
- Explicitly handling failure via Maybe or similar is almost always the better solution.

Exceptions in effectful ( IO ) code are different:

- Execution order is explicit, and handling is easier.
- There are many things that go wrong.

## Catching IO errors

```hs
From System.IO.Error :
catchIOError :: IO a -> (IOError -> IO a) -> IO a
readFileLineByLine' ::
FilePath -> IO (Maybe [String])
readFileLineByLine' file =
catchIOError
(liftM Just (readFileLineByLine file))
(const (return Nothing))
```

## Testing it

```hs
GHCi> writeFile "test" "foo\nbar"
GHCi> readFileLineByLine' "test"
Just ["foo", "bar"]
GHCi> removeFile "test"
GHCi> readFileLineByLine' "test"
Nothing
```

From System.Directory :

```hs
removeFile :: FilePath -> IO ()
```

## Recap

- The role of the `IO` type.
- Composing `IO` functions.
- Higher-order `IO` functions ( `sequence` , `mapM` ).
- File IO.
- Resources.
- Exceptions.
