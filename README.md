# Ende

I just want to share what I've come up with about the design of a new programming language, and here it is.
This language has a strong type system, and its syntax is whitespace-insensitive because I prefer it.
It's very welcomed for anyone to write an implementation for it.

# Terms and Statements

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
   
   2. **Floats**:
      Literals of floats would be similar to ones of integers, e.g. `2.71828f32`.

2. Applications of binary operators:
   There will only be binary operators instead of unary or trinary ones, also because that's enough of illustrative purposes.
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
fn factorial(n : U32) -> U32 = {
    if n == 0 { 0 }
    else { n * factorial(n - 1) }
};
```

# User-Defined Data Types

Data types can be defined with the keyword `data`.
They can be seen as `enum`s in Rust.
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
};
```

Here, we directly match the arguments of the `fn`; now we don't need to pick a name for the argument of type `Season`; we can write `fn isSummer(Summer)` in lieu of `fn isSummer(_ : Summer)` in this case.
I'm going to provide another example for clarity:

```rust
fn and(Bool, Bool) -> Bool {
    (Bool::true, b) => b,
    (_, _) => false,
};
```

Variants may also take parameters.
The `OptionI32` below is a type that could possibly carry an `I32`.

```rust
data OptionI32 = some(I32), none;
```

Parameters of Variants can also be pattern matched:

```rust
fn unwrapOr42(OptionI32) -> I32 {
    (OptionI32::some(i)) => i,
    (_) => 42i32,
};
```

There's another way to define a data type, using `class`.
`class`es in Ende are like `struct`s in Rust.
`class`es can be seen as `data` with only one variant.
The name of the variant isn't namespaced, though.

```rust
class Point = point {
    x : I32,
    y : I32,
};
```

Members of an instance of a `class` can either be accessed by its name after a dot (`.`) or by pattern matching.

```rust
fn getX1(p : Point) -> I32 = p.x;
fn getX2(Point) -> I32 {
    (point {x => result, y => _}) => result,
};
```

To pattern match or to construct an instance of a `class`, we write a fat arrow (`=>`) after each name of the fields instead of a colon (`:`) used when a `class` is defined:

```rust
let p = point { x => 0i32, y => 0i32 };
```

# Generics

We've seen a `OptionI32` type above.
In practice, we want a type that is generic over all types, not just `I32`.
But let's start with a simplest generic function: `id`.
`id` simply returns its only argument.

```rust
fn id[T](t : T) -> T = t;
```

Here, I introduced another delimiter while defining the function: brackets (`[]`).
different delimiters on a function represent different *modes* in which the paramenters are passed.
**Modes** are a very important feature in Ende; different modes serve as different purposes and have different characteristics.
We say that the arguments inside the parentheses (`()`) (`t` in the above example) are arguments in **normal mode**.
In contrast, arguments inside the brackets (`[]`) (`T` in the above example) are arguments in **`const` mode**.
For now, you just have to know that arguments in `const` mode have to be supplied at compile time and can be inferred.
Therefore, you can write `id(0i32)` instead of the more verbose `id[I32](0i32)`.

I haven't mentioned function types, did I?
Function types are literally the types of functions and are written as `(A, B, C, ...) -> R`.
`A, B, C, ...` are the types of the arguments, and `R` is the return type.
As a side note, arguments in normal mode cannot be curried in Ende similar to the ones in C++/Scala/Rust, but arguments in `const` mode can:

```rust
// They are different:

foo(a, b);
foo(a)(b);

// But they aren't:

bar[A, B];
bar[A][B];
```

This is because of the way that current machine works.
At runtime, functions can have several arguments natively.
If arguments in normal mode were curryable, the compiler would have to retern lambdas often or generate several partially applied copies of the original function.
Arguments in `const` mode are curryable, though, because that performance at compile time isn't that important, and programmers are supposed to do heavy calculation at runtime.

Back to generics, here is a `compose` function, which `compose`s its 2 function arguments.

```rust
fn compose[A, B, C](f : (B) -> C, g: (A) -> B)(x : A) -> C = f(g(x));
```

And here is the definition of a generic `Option` type:

```rust
data Option[T] = some(T), none;
```

# More General `class`

Why are Rust `struct`s called `class`es in Ende?
Because they can not only act as Rust `struct`s but also Haskell `class`es.
Here, I'm going to write a `Monoid` interface.
Of course, it's just another `class`.

```rust
class Monoid[T] = monoid {
    unit : T,
    fn append(self : T, T) -> T,
};
```

To implement a class, I introduce another keyword `impl`, unlike the `impl`s in Rust or `instance`s in Haskell, `impl`s in Ende are always named.
There are 3 kinds of `impl`s in Ende.

## `impl` objects

`impl` objects are the first kind of `impl`.
The `i32Monoid` below is an `impl` object.

```rust
impl i32Monoid : Monoid[I32] = monoid {
    unit => 0i32,
    fn append(self : I32, another : I32) = self + another,
};
```

If the `impl` object `i32Monoid` is in scope, now we can call the `append` method on `I32`:

```rust
// They are equivalent:

let sum1 = append(1i32, 2i32);
let sum2 = 1i32.append(2i32);
```

In order to write a function that is generic over types implementing a `class`, the third mode is introduced.
It's called **instance mode**, and is delimited by `[()]`.
`[(T)]` is always parsed as `[( T )]` but not `[ (T) ]` because putting a pair of parentheses (`()`) around a type variable doesn't make much sense.
Arguments in instance mode can also be inferred , however not by looking at other arguments, but by searching for appropriate `impl`s
Here is a function generic over types implementing `Monoid`; it sums up all the values in a `List` using `Monoid`'s `append` method:

```rust
fn concat[T][(Monoid[T])](List[T]) -> T {
    (List::nil) => unit,
    (List::cons(head, tail)) => head.append(concat(tail)),
};
```

When you write `concat(vec)`, the compiler automatically chooses the right `impl` of `Monoid`; if there are 0 or over 2 choices, the compiler generates an error.
Nonetheless, you can explicitly provide a specific `impl`, delimiting which in `[()]`:

```rust
let sum = concat[(i32Monoid)](i32Vec);
```

## `impl` functions

`impl` functions are literally, `impl`s that are functions.
`impl`s need to be functions mainly because of 2 reasons:
1. It's generic over an argument in the `const` mode.
2. It's generic over an argument in the instance mode.
I'm going to provide a `impl` function generic over both modes.
First, define a `Group` `class`.

```rust
class Group[T] = group {
    unit : T,
    fn append(self : T, T) -> T,
    fn inverse(self : T) -> T,
};
```

We know all `Group`s are `Monoid`s, so all instances of `Group[T]` should be able to be automatically converted to instances of `Monoid[T]`.
How do we automatically convert stuff?
The answer is obviously, again, using `impl`s.

```rust
impl groupToMonoid[T][(Group[T])] -> Monoid[T] = {
    monoid {
       unit => unit,
       append => append,
    }
};
```

`group` is indeed an `impl` function.

## `dynamic impl`

What we don't have yet is the ability to define one `class` to be a super`class` of another `class`. In other words, asserting if you implement a `class`, another `class` must be implemented.
You may ask, isn't the above `impl` functions enough?
Not really.
Imagine you want to define an `Abelian` `class`, the code you need to add would be:

```rust
class Abelian[T] = abelian {
    unit : T,
    fn append(self : T, T) -> T,
    fn inverse(self : T) -> T,
};

impl abelianToGroup[T][(Abelian[T])] -> Group[T] = {
    group {
       unit => unit,
       append => append,
       inverse => inverse,
    }
};
```

That's a lot of boilerplate!
The code is very similar to implementing `Monoid`s for `Group`s; we have to write this kind of code again and again while creating an inheritance tree of `class`es.
That is not tolerable.
Imagine if we can write code like this:

```rust
// The type `Abelian` carries no additional data, but it extends the `class` `Group`.
data Abelian[T] = abelian;
impl abelianExtendsGroup[T] -> Extends[Abelian[T], Group[T]] = Extends::extension; // The extension happens here.
```

It's really concise code!
But how do we get an instance of type `Group[T]` given the instances of types `Abelian[T]` and `Extends[Abelian[T], Group[T]]`?
It's actually a hack: we need more `impl`s.
I'll try to explain it.

### The Hack

First, I'll provide the hacky part of the source code:

```rust
data Extends[A, B] = extension;

fn implicitly[T][(inst : T)] -> T = inst;

dynamic impl superClass[A, B][(A, Extends[A, B])] -> B = implicitly[B];
```
