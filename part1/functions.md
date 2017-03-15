# Functions

Functions are the simplest concept that has the ability to encapsulate information in functional programming languages; function calls are terms.  
The declaration of a function is an _item_.  
Items are basically some construct that can appear at the top level of the source code.

Every application needs to have an entry, traditionally called `main`.  
In Ende, `main` is a function of type `() -> IO[Unit]`.  
That is, a function that accepts no inputs and returns something that has something to do with `IO`.  
Now, let's write the well-known _Hello, world!_ program in Ende.

```
fn main() -> IO[Unit] = putStrLn("Hello, world!")
```

Nothing special.  
The syntax is inspired by Rust; the keyword `fn` is used to declare a function.  
Unlike Rust, there is an equal sign \(`=`\) after the type of the function.

```
fn factorial(n : U32) -> U32 =
    if n == 0u32
    then 1u32
    else n * factorial(n - 1)
```

All operators in Ende are just normal functions with special names.  
For instance, the exponential operator `^^` could be defined as:

```
fn _^^_(_0_ : U32; _1_ : U32) -> U32 =
    if _1_ == 0u32
    then 1u32
    else _0_ * id(_0_ ^^ id(_1_ - 1u32))
```

In the above example, the underscores in the function name `_^^_` represent where the arguments go when it's applied as an operator.  
Each argument `_n_` is a binding of the argument of the nth underscore.  
The allowed symbols in a variable name include `!`, `$`, `%`, `&`, `*`, `+`, `.`, `/`, `<`, `=`, `>`, `?`, `^`, `|`, `-`, `~`, `:`, `,`.  
Of course you cannot redefine built-in syntax such as `=`, `->` or `:` because that would cause parsing ambiguity.

\(TBD: fixity\)

In all of the above examples, the definition of the function is just a single term.  
There are two ways to write more complicated function.  
The first is to write `where` instead of `=` after the return type of the function; we call the code inside it _inside the function body_.  
The valid constructs inside the function body are `let ... = ...` and local function definitions only visible inside the function body.

```
fn getBMI() -> F64 where
    let weight = 100f64
    let height = 200f64
    fn calcBMI(w : F64, h : F64) -> F64 = w / h / h
    return calcBMI(weight, height)
```

The last clause in a function body must be `return` something.

Another way to write a complicated function is to use do-notation; the valid clauses of which are those I called \_statement\_s.  
You need them to do several `IO` operations at once.

```
fn greeting() -> IO[Unit] = do
    putStr("Hello, ")
    let name <- mut("Andy")
    putStr(name)
    putStrLn(".")
    putStr("Hello, ")
    name := "Sandy"
    putStr(name)
    putStrLn(".")
```

You will see do-notation can be used much more generally than writing `IO` code.

## lambdas

Lambdas are unnamed function literals.  
Like normal functions, lambdas must specify their return types.  
Lambdas are written similar to a function but without the name.

```
let abs =
    fn(n : I32) -> I32 =
        if n >= 0i32
        then n
        else -n
```



