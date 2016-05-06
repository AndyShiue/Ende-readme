# Ende

I just want to share what I've come up with about the design of a new programming language, and here it is.
This language would have a strong type system, and its syntax would be whitespace-insensitive because I prefer it.

# Terms and statements

At the very beginning, let me introduce the terms of the language.
Terms can be seen as trees that build up value.
Terms include:

1. **Literals**:
   There are several built-in types in Ende.
   They are numbers.
   Boolean values still won't be introduced yet but will be defined as a data type.
   Types like `Char` and `String` isn't presented because numbers suffice for illustrative purposes but should be added if there is an actual implementation of the language.

   1. **Integers**:
      There would be several types of integers of different size, being either signed or not.
      A literal of type `I32` would be written as `42i32`.
      `42` is it's actual value, and `i32` means it's a 32-bit unsigned integer.
      Similarly, `666u64` would be a 64-bit signed integer.
      An integer without any suffix would have type `Nat`.
      Instead of being built-in, `Nat` is just a normal recursively defined data type.
      I will also discuss that type later.
   
   2. Floats
      Literals of floats would be similar to ones of integers, e.g. `2.71828f32`.

2. Applications of binary operators:
   There will only be binary operators instead of unary of trinary ones, also because that's enough of illustrative purposes.
   In actual implementations, operators of any fixity could be introduced.
   There could even be user-defined mixfix operators, but that's way beyond the scope of this article.
   
   Examples would be `1 + 1`, `42 * 666`, `1 == 2`, etc.
   Fixity of operators should follow the common sense.
   Parentheses (`()`) are used to disambiguate if the fixity isn't clear.

3. Variables, which are going to be discussed immediately.

Throughout the article, more and more other kinds of terms will be introduced.

Ende is generally an imperative language, so there are statements including control structures.
Statements include:

1. **`let` binding**:
   A `let` binding bind its right-hand-side value to a variable.
   There are 2 flavors of `let` bindings, a mutable one and an immutable one.
   `let` is by default immutable.
   For example, `let meaningOfLife = 42i32;` binds `42i32` to a variable called `meaningOfLife`.
   As you can see, variables are written in snake case in Ende.
   variables can be made mutable by adding a keyword `mut` after `let`.
   So, `let mut count = 0i32;` binds `0i32` to a variable `count`.

2. **mutation**:
   The value of a mutable variable can be mutated.
   `count = 1i32;` changes the value of `count` to `1i32`.
   Mutating an immutable variable generates a compile error.

3. **`while` loop**:
   Normal `while` loop similar to other imperative languages.
   The syntax is especially similar to Rust's, because I'm a fan of Rust.

   ```rust
   while count == 0i32 {
      forever();
   };
   ```
   
   Notice unlike in Rust, the semicolon after the whole loop is required.
   In fact, `while` loop is just another kind of term returning value of type `Unit`.
   `Unit` is a type that carries no data.
   When a term is follow by a semicolon, it discards the value of the whole term.
   But a `while` loop already has nothing to be discarded, so the semicolon just turns it into a statement.

Now, introduce `if`,
unlike `while`, `if` can return something that isn't trivial.
So we can write code like:

```rust
let number = if count == 0i32 { 42 } else { 666 };
```

Of course, `if` can also be used in an old-style, C-like fashion.

```rust
let mut number;
if count == 0i32 {
    number = 42;
} else {
    number = 666;
};
```

# Functions

Functions are the simplest concept that has the ability to encapsulate information in functional programming languages; functions are terms.
Every application needs to have an entry, traditionally called `main`.
In Ende, `main` is a function of type `() -> Unit`.
That is, a function that accepts no inputs and returns nothing.
Now, let's write the well-known *Hello, world!* program in Ende.

```rust
fn main() -> Unit = {
    putStrLn("Hello, world!");
};
```

Nothing special.
The syntax is heavily inspired by Rust; the keyword `fn` is used to declare a function.
Unlike Rust, there is a equal sign (`=`) after the type of the function so the braces (`{}`) is optional if you just want to return a single term.
The trailing semicolon (`;`) is also required similar to the one of `while`.

```rust
fn factorial(n : U32) -> U32 = n * factorial(n - 1);
```

# User-Defined Data Types

Data types can be defined with the keyword `data`.
They can be seen as `enum`s in Rust, but being more powerful.
I'll first consider its easiest usage, though.
Let's show how to define a C/Java-like `enum` in Ende:

```rust
data Unit = unit;
data Bool = true, false;
data Season = spring, summer, autumn, winter;
```

In Ende, `data` can have **variants**.
Variants are the possible values of the defined type.
Variants are namespaced; in order to access the varient `spring`, you need to write

```rust
let season = Season::spring;
```

instead of

```rust
let season = spring; // Doesn't compile.
```

`data` variants could be matched through pattern matching:

```rust
fn isSummer(season : Season) -> Bool = {
    match season {
        case Season::summer => true,
        _ => false,
    }
};
```

`match`es are always irrefutable in Ende.
By the way, there is an alternative `fn` syntax to do top-level pattern matching.
We don't write the equal sign (`=`) after the return type in this case.

```rust
fn isSummer(Season) -> Bool {
    (Season::summer) => true,
    (_) => false,
}
```

Here, we directly match the arguments of the `fn`; now we don't need to pick a name for the argument of type `Season`; we can write `fn isSummer(Summer)` in lieu of `fn isSummer(_ : Summer)` in this case.
I'm going to provide another example for clarity:

```rust
fn and(Bool, Bool) -> Bool {
    (true, b) => b,
    (false, _) => false,
}
```

Variants may also take parameters.
The `OptionI32` below is a type that could possibly carry an `I32`.

```rust
data OptionI32 = some(I32), none;
```
