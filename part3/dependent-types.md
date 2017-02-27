# Dependent Types

In the above examples, we've seen types depending on values at compile time, but not values at runtime.
In order to be fully dependently-typed, another mode called the **pi mode** has to be introduced:

| ordered                   | normal    | `const`      | instance      | pi            |
|:-------------------------:|:---------:|:------------:|:-------------:|:-------------:|
| `T` means                 | `(_ : T)` | `[T : _]`    | `[(_ : T)]`   | `([T : _])`   |
| type inference            | no        | yes          | no            | yes           |
| phase (if not `const fn`) | runtime   | compile time | compile time  | runtime       |
| curryable                 | no        | yes          | yes           | yes           |
| argument inference        | no        | yes          | proof search  | no            |
| can be dependent on       | no        | yes          | yes           | yes           |
| **unordered**             | `{t : T}` | `{[t : T]}`  | `{[(t : T)]}` | `{([t : T])}` |
| type inference            | no        | no           | no            | no            |
| phase (if not `const fn`) | runtime   | compile time | compile time  | runtime       |
| curryable                 | no        | no           | no            | no            |
| argument inference        | no        | yes          | proof search  | no            |
| can be dependent on       | no        | yes          | yes           | yes           |


`(T)` and `[(T)]` mean `(_ : T)` and `[(_ : T)]`, respectively, but `[T]` and `([T])` mean `[T : _]` and `([T : _])`, respectively.
Types of arguments in the `const` modes and the pi ones can be inferred.
Usually arguments in the normal mode or pi mode are supplied at runtime, but not arguments in the `const` or the instance modes.
Modes except the normal mode are curryable because it would be more convenient then.
Arguments in the normal mode cannot be inferred and cannot be dependent on obviously.
Arguments in all argument lists can only be dependent on arguments in previous argument lists.

Existential types as an example of pi-types:

```
pub(in) data Sigma[A][B : ([A]) -> Type] = new([a : A])(B([a]))
```

(TBD: `data` from the `const` mode to the pi one)

## `with`

The value of one argument of a dependent function might depend on another argument in pi mode.
We need to be able to match against several arguments together.
Introduce **views**, through `with` clauses.
Top-level pattern matching can include arms with `with` clauses.
Here's an example:

(TBD)
