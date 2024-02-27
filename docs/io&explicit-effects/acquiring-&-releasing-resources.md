---
sidebar_position: 7
---
# Acquiring & releasing resources

## Whole-file IO

```hs
readFile :: FilePath -> IO String
writeFile :: FilePath -> String -> IO ()
```

## Handle-based file IO

All in `System.IO`:

```hs
hGetLine :: Handle -> IO String
hPutStrLn :: Handle -> String -> IO ()
hIsEOF :: Handle -> IO Bool

withFile ::
FilePath -> IOMode
-> (Handle -> IO r) -- continuation (aka callback)
-> IO r
data IOMode =
ReadMode | WriteMode
| AppendMode | ReadWriteMode
```

## Reading a file line by line

```hs
readFileLineByLine :: FilePath -> IO [String]
readFileLineByLine file =
withFile file ReadMode readFileHandle
readFileHandle :: Handle -> IO [String]
readFileHandle h = do
eof <- hIsEOF h
if eof
then return []
else do
line <- hGetLine h
lines <- readFileHandle h
return (line : lines)
```

Handle is automatically released at end of continuation.

:::warning
Both `readFile` and `readFileLineByLine` are actually
problematic for different reasons.
We will (probably) learn about better ways to process (in particular
large) files later this week.
:::
