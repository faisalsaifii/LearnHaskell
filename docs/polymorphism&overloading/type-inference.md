---
sidebar_position: 1
---

# Type Inference

- The compiler will infer types for expressions, and for constant and
function declarations automatically. Type annotations are rarely
required.
- Type annotations can always be provided and will be checked for
correctness by the compiler.
- Type signatures for top-level declarations are considered good
style. They serve as invaluable machine-checked interface
documentation.
- You can use GHC(i) to obtain inferred types. Use :t often, but
also try to train your own type inference capabilities over time – it
will help you to understand errors with less effort.
