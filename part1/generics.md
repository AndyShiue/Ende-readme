# Generics

We've seen an `OptionI32` type above.  
In practice, we want an `Option` type that is generic over all types, not just `I32`.  
But let's start with a simplest generic function: `id`.  
`id` simply returns its only argument.

```
#pub:
fn id[T](t : T) -> T = t
```

Here, I introduced another sort of delimiter while defining the function: brackets \(`[]`\).  
Different sorts of delimiters after the function name represent different _modes_ in which the parameters are passed.  
**Modes** are a very important feature in Ende; different modes serve as different purposes and have different characteristics.  
We say that the arguments inside the parentheses \(`()`\) \(`t` in the above example\) are arguments in the **normal mode**.  
In contrast, arguments inside the brackets \(`[]`\) \(`T` in the above example\) are arguments in the `const`** mode**.  
For now, you just have to know that arguments in `const` mode have to be supplied at compile time and can be inferred, so you can write `id(0i32)` instead of the more verbose `id[I32](0i32)`.

I haven't mentioned function types, have I?  
Function types are literally the types of functions and are written as `(A; B; C; ...) -> R`.  
`A`,`B`,`C`, ... are the types of the arguments, and `R` is the return type.  
As a side note, similar to the ones in C++/Scala/Rust, arguments in normal mode cannot be curried in Ende.  
However, arguments in the `const` mode can:

```
\\ They are different:

foo(a; b)
foo(a)(b)

\\ But they aren't:

bar[A; B]
bar[A][B]
```

This is because of the way that current machines work.  
At runtime, functions can have several arguments natively.  
If arguments in the normal mode were curryable, the compiler would have to return lambdas often or generate several partially applied copies of the original function.  
Arguments in the `const` mode are curryable, though, because that performance at compile time isn't that important, and programmers are supposed to do heavy calculation at runtime.

Back to generics, here is a `compose` function, which `compose`s its 2 function arguments.

```
#pub:
fn compose[A; B; C](f : (B) -> C; g: (A) -> B)(x : A) -> C = f(g(x))
```

And here is the definition of a generic `Option` type:

```
@allPub:
data Option[T] = some(T); none
```



