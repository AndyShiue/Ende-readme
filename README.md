# Ende

I just want to share what I've come up with about the design of a new programming language, and here it is.
This language has a strong type system, its functions are call-by-value, and its syntax is whitespace-insensitive because I prefer it.
It's very welcomed for anyone to write an implementation for it.

# Overview

This article was initially designed to be implemented from the beginning to the end, but it has already been too long.
As some people suggested, I provide a brief overview here.

The core feature of Ende is **modes**.
Modes are ways to pass arguments to a function.
In current programming languages, the boundary between phases, which mean compile time and runtime, are either not blurry enough or too blurry.
The former includes more conservative languages like Java or Go.
The latter are basically those dependently-typed languages.
The lack of compile-time constructs make higher abstraction at compile time impossible, but dependent types make the phase of which the term is evaluated unpredictable.
The idea of modes is basically genericity over phases.
In complement to modes, you can specify stuff that only works at runtime or works at both runtime and compile time using the `const` system of Ende.

The presence of modes makes almost everything in Ende first-class, which means they can be passed as arguments and returned.
They include:

1. all functions
2. all structs
3. all class constructors (in the sense of Java jargon)
4. all interfaces
5. all implementations of interfaces
6. all extensions between interfaces
7. the vast majority of types

As you will see, Ende achieve the unification of different concepts with carefully designed syntax and semantics.

The syntax is very similar to Rust.
If you are familiar with Rust, you might start reading from [this section](#more-general-class).
Just be aware of the top-level pattern matching syntax and that generic parameters are written inside square brackets (`[]`) instead of angle brackets (`<>`).
Users of Scala might also find the similarities between Scala and Ende.
I actually took lots of ideas from Rust, Scala, Agda, Idris, etc.

# Terms and Statements

At the very beginning, let me introduce the terms of the language.
Terms can be seen as trees that build up values.
Terms include:

1. **Literals**:
   There are several built-in types in Ende.
   They are numbers.
   Boolean values still won't be introduced yet but will be defined as a data type.
   Types like `Char` and `String` isn't presented because numbers suffice for illustrative purposes but should be added if there is an actual implementation of the language.

   1. **Integers**:
      There would be several types of integers of different size, being either signed or not.
      A literal of type `I32` would be written as `42i32`.
      `42` is its actual value, and `i32` means it's a 32-bit signed integer.
      Similarly, `666u64` would be a 64-bit unsigned integer.
      An integer without any suffix would have type `Nat`.
      Instead of being built-in, `Nat` is just a normal recursively defined data type.
      I'm going to defer the discussion of that type to later.
   
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
   As you can see, variables are written in camel case in Ende.
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
    if n == 0 { 0u32 }
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
// let season = spring; // Doesn't compile.
```

`data` variants could be matched through pattern matching:

```rust
fn isSummer(season : Season) -> Bool = {
    match season {
        case Season::summer => Bool::true,
        _ => Bool::false,
    }
};
```

`match`es are always irrefutable in Ende.
By the way, there is an alternative `fn` syntax to do top-level pattern matching.
We don't write the equal sign (`=`) after the return type in this case.

```rust
fn isSummer(Season) -> Bool {
    (Season::summer) => Bool::true,
    (_) => Bool::false,
};
```

Here, we directly match the arguments of the `fn`; now we don't need to pick a name for the argument of type `Season`; we can write `fn isSummer(Summer)` in lieu of `fn isSummer(_ : Summer)` in this case.
I'm going to provide another example for clarity:

```rust
fn and(Bool, Bool) -> Bool {
    (Bool::true, b) => b,
    (_, _) => Bool::false,
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

Someone mentioned that I sometimes write a trailing comma after the last matching arm or the last field of a `class`.
That's intentionally designed.
It's the same as the situation in Rust: all trailing commas are optional.
(Oh, except one case that I will mention.)
you can even write

```rust
data A =
    a,
    b,
    c,
    d,
;
```

# Generics

We've seen an `OptionI32` type above.
In practice, we want a type that is generic over all types, not just `I32`.
But let's start with a simplest generic function: `id`.
`id` simply returns its only argument.

```rust
fn id[T](t : T) -> T = t;
```

Here, I introduced another delimiter while defining the function: brackets (`[]`).
different delimiters on a function represent different *modes* in which the parameters are passed.
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

This is because of the way that current machines work.
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

# Named Mode

Compare this `class` in Ende

```rust
class Example = example {
    example : Unit,
};
```

and this `struct` in Rust:

```rust
struct Example {
    example: (),
}
```

Why do we need to write *example* twice, one starting in upper case and another starting in lower case?
The intention is to disambiguate between the type `Example` and the constructor `example`.
In a system that doesn't have dependent types, an usage of `Example` can always be resolved, but not in a language in which types are first-class.

Variants in `data` have types, what is the type of `example` then?
In order to assign a type to it, we new modes in which parameters are named.
Let's simply call them **named mode**.
The named normal mode is similar to the normal mode, except that the parameters are named and unordered; and the named `const` mode corresponds to the `const` mode.
The above `example` now has type `{example : Unit} -> Example`.
To be more general, now we can also have struct variants and arbitrary functions accepting named parameters.

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
An `impl` object must be an instance of a `class`.

```rust
impl i32Monoid : Monoid[I32] = monoid {
    unit => 0i32,
    fn append(self : I32, another : I32) -> I32 = self + another,
};
```

If the `impl` object `i32Monoid` is in scope, now we can call the `append` method on `I32`:

```rust
// They are equivalent because the first argument of `append` is `self`:

let sum1 = append(1i32, 2i32);
let sum2 = 1i32.append(2i32);
```

In order to write a function that is generic over types implementing a `class`, the third mode is introduced.
It's called **instance mode**, and is delimited by `[()]`.
Arguments passed in instance mode must be instances of `class`es.
`[(T)]` is always parsed as `[( T )]` but not `[ (T) ]` because putting a pair of parentheses (`()`) around a type variable doesn't make much sense.
Arguments in instance mode can also be inferred , however not by looking at other arguments, but by searching for appropriate `impl`s.
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
An `impl` function must return an instance of a `class`.
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

`groupToMonoid` is indeed an `impl` function.

## `special impl`

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
impl abelianExtendsGroup[T] -> Extends[Abelian[T], Group[T]] = extension; // The extension happens here.
```

It's really concise code!
But how do we get an instance of type `Group[T]` given the instances of types `Abelian[T]` and `Extends[Abelian[T], Group[T]]`?
It's actually a hack: we need more `impl`s.
I'll try to explain it.

### The Hack

First, I'll provide the hacky part of the source code:

```rust
class Extends[A, B] = extension;

// Ignore the `const` keyword before `fn` for now.
const fn implicitly[T][(inst : T)] -> T = inst;

special impl superClass[A, B][(A, Extends[A, B])] -> B = implicitly[B];
```

The basic idea is that no matter what `A` and `B` are, if `impl`s of `A` and `Extends[A, B]` are in scope, `impl` of `B` is made in scope.
In the revised version of example of `Abelian`,  because both `impl`s of `Abelian[T]` and `Extends[Abelian[T], Group[T]]` are in scope, `impl` of `Group[T]` is also made in scope.
Why is the keyword `special` before the `impl superClass` required then?
To know why it's needed, we need to go deeper to know how an `impl` is found.

### Searching for `impl`s

First, we search for `impl` objects.
We add an `impl` object in scope to the current **`impl` context** if the `class` it implements is also in scope.

Second, we add the `impl` functions to the `impl` context.
There is a necessary limitation of normal `impl` functions:
a normal `impl` function can only have a return type that is not a variable, so the example below does't work:

```rust
// impl abuseOfImplicitly[T] -> T = implicitly[T]; // Doesn't compile.
```

The reason why some limitation is needed is because we want to make searching `impl`s more predictable, so that we can filter out the `impl` functions that doesn't retern an `impl` of a type in scope.
Without the limitation, the `impl` searching process could stuck at some weird recursive `impl`.

And `special impl`s surpass that limitation.
It has to be used more carefully, but I don't think there's a lot of uses of it.
In fact the only one I can think of is `class` inheritance.
The `impl`s of the return types of the `special impl`s are recursively added to the `impl` context no matter whether the type it implements is in scope or not.

## Associated Types

Fields of an instance of a `class` can also be dependent on.
They are different from normal *input parameters* in that they don't determine the `impl` chosen but the `impl`s determine them.
They are *output parameters*.
For example, imagine if there is a way to overload operators.
There should be an interface for each operator.
To achieve maximal flexibility, I want to make types of the right-hand-side, the left-hand-side, and the returned term possibly different.
It means it has to be generic over these 3 types.
So maybe the interface could be like:

```rust
class Add[L, R, Output] = add {
    fn add(self : L, R) -> Output,
};
```

However, the interface is problematic because we can provide both instances of `Add[L, R, A]` and `Add[L, R, B]`; if I write `(l : L) + (r : R)`, the compiler wouldn't be able to know if the type of the result would be `A` or `B`.
This suggests that the `Output` type should not be an input parameter but rather an output one determined uniquely by `L` and `R`.
The correct interface should be:

```rust
class Add[L, R] = add[Output] {
    fn add(self : L, R) -> Output,
};
```

If you want to access the output type specified by an instance of a `class`, simply write `inst.Output`.

# `const`

You may found that I wrote *`const` mode* instead of *const mode* throughout the article.
This is intentional.
`const` is an important keyword in Ende, and is highly related to `const` mode.
The basic idea is that a *`const` something* is a *something* that can be done at compile time.
So a `const` value (a constant) is a value known at compile time, and a `const fn` is a function that could be run at compile time.

Now, let's go through all kinds of terms introduced and see if they are constants.

1. **Literals**: They are definitely constants.

2. **Operator applications**: An application of an operator outputs a constant if and only if all of its arguments are constants.

3. **Variables**:
   A variable is a constant if it's declared as `const varName`.
   For example, in `const meaningOfLife = 42i32;`, `meaningOfLife` is a constant.
   By contrast, in `let devil = 666i32`, devil **isn't** a constant.
   Of course, the right hand side of the `const` binding must be a constant as well.

4. **Control structures**:
   The value of a `while` loop is not a constant.
   The value of a `if` construct is a constant if and only if at least one of the conditions below are met.

   1. The term immediately after `if` is a constant and evaluates to `true`, and the first branch of the `if` represents a constant.
   
   2. The term immediately after `if` is a constant and evaluates to `false`, and the second branch of the `if` represents a constant.

5. **Functions**:
   Functions that aren't inside a `class` are always constants, but there's a difference between `const fn`s and normal functions.
   `const fn`s are functions that could be run at compile time (also at runtime).
   The return value of a `const fn` is a constant if and only if all of its arguments are constants in that invocation.
   The operations you can do in a `const fn` are more limited.
   You can only:

   1. declare a variable with `const`.
   
   2. declare a variable with `let`:
      The right-hand-side after the equal sign (`=`) must be a constant.
      Note that declaring a variable with `const` and `let` are also different in a `const fn`.
      They are both constants in a `const fn`.
      But a variable declared with `const` cannot depend on the arguments in normal mode, while a variable declared with `let` can.
      The gist of the design is to make changing a non-`const` function to a `const fn` (or conversely) the most seamless.
      Of course, `let mut` is forbidden in a `const fn`.

   3. normal statements:
      The term before the semicolon (`;`) must be a constant, so `while` cannot be used in a `const fn`, so if you want to do something again and again, use recursion.

   4. function calls:
      Only `const fn`s can be called.

   `const fn`s are also checked to be *positive*, meaning they don't recurse forever.
   A constant that evaluates to a `const fn` is also a `const fn`.
   If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.

6. **`impl`s**:
   Did I mention `impl`s are first-class citizens of Ende?
   They can be returned and passed as arguments.
   `impl` objects are always constants; `impl` functions and `special impl`s are always constants and `const fn`s.

7. **`data`**:
   In `data`, all variants are constants.
   In addition, all variants with parameters in normal `data` are `const fn`s.
   You can overwrite the default behavior, however.
   If you write `dyn` before `data`, all variants become non-`const`.
   You can make specific variants `const` again by writing `const` before the variants.
   Writing `dyn` before a variant in a non-`dyn` `data` makes it non-`const`.

   ```rust
   data Data =
       data1,
       dyn data2,
       data3(I32),
       dyn data4(I32);
   
   dyn data Dynamic =
       dynamic1,
       const dynamic2,
       dynamic3(I32),
       const dynamic4(I32);
   
   // Statements commented out can not be compiled.
   
   const _ = Data::data1;
   // const _ = Data::data2;
   const _ = Data::data3;
   const _ = Data::data3(0i32);
   const _ = Data::data4;
   // const _ = Data::data4(0i32);
   // const _ = Dynamic::dynamic1;
   const _ = Dynamic::dynamic2;
   const _ = Dynamic::dynamic3;
   // const _ = Dynamic::dynamic3(0i32);
   const _ = Dynamic::dynamic4;
   const _ = Dynamic::dynamic4(0i32);
   ```

8. **`class`es**:
   An instance of a `class` is a constant if all of its fields are constants by default.
   In comparison to `data`, the problem of constness is even more serious, though.
   When you write a `class`, I assume you want not only that the fields are constants, but also the function members are `const fn`s, and that's the default.
   Here is such an example, the `Monoid` class we've talked about.

   ```rust
   class Monoid[T] = monoid {
       unit : T,
       fn append(self : T, T) -> T,
   };
   
   impl i32Monoid : Monoid[I32] = monoid {
       unit => 0i32,
       fn append(self : I32, another : I32) -> I32 = self + another,
   };
   
   const _ = 0i32.append(0i32); // It works.
   ```

   But sometimes you want to use `class`es as Java `class`es, in that case, you want all non-function fields to be mutable, and all function members to be non-`const` functions.
   `dyn class` is used to define such `class`es.
   You can also write `dyn` in front of a function member in a non-`dyn` `class` to indicate it's not a `const` function.
   
   ```rust
   dyn class Counter = counter {
       inner : I32,
       fn increment(self : Counter) -> Unit;
   };
   
   fn newCounter() -> Counter = counter {
       inner => 0i32,
       fn increment(self : Counter) -> Unit = {
           self.inner += 1;
       },
   };
   ```
   
   Notice that `newCounter.increment` **is** a constant although it's not a `const fn`.
   
   If you want a single function field to be dynamic, write `dyn` **after** the keyword `fn`.
   
   ```rust
   dyn class Wierd = duh {
       fn dyn wierd : () -> Unit,
   };
   
   fn doNothing() -> Unit = {};
   
   let wierd = duh {
       wierd => main,
   };
   
   wierd.weird = doNothing; // It works.
   ```
   
   An instance of a non-`dyn` `class` is a constant if all of its fields are constants.
   An instance of a `dyn class` is never a constant.

## A `const` Version `factorial`

I'll define a `const fn factorial` in this subchapter.
The implementation of this `factorial` is different from the `U32` version above.
This is not to suggest Ende has overloading.
Just pretend the 2 functions are from different spaces.

The problem with `U32` is that it's is not defined recursively, so it would be harder for the computer to figure out if the functions terminate.
The solution is to use a recursively defined data type: `Nat`

```rust
data Nat = zero, succ(Nat);
```

And here's the `const fn factorial`:

```rust
const fn factorial(Nat) -> Nat {
    (0) => 1,
    (Nat::succ(n)) = Nat::succ(n) * factorial(n),
};
```

# More Powerful Generics

Arguments in `const` mode need not be types; they can be constants as well.
In fact, **any** constants which is typeable can be passed to a function in `const` mode.
So I can write something like:

```rust
const fn factorial[_ : Nat] -> Nat {
    [0] => 1,
    [Nat::succ(n)] = Nat::succ(n) * factorial(n),
};
```

But then we lose the ability to call `factorial` at runtime.

Actually, types are also first-class citizens of Ende.
They are constants.
In Ende, we can also be generic over type constructors, which are `const fn`s at the type level.
That is, to pass higher-kinded types around.
Normal types have kind `Type`.
However, `Option` is a type constructor and is of kind `[Type] -> Type`, which means it is a function from a type at compile time to another type.
Here's the `Functor` `class`.

```rust
class Functor[F : [Type] -> Type] = functor {
    fn map[From, To](self : F[From], (From) -> To) -> F[To],
};
```

Types of arguments in `const` mode are not always inferred to be `Type`, they can be inferred to be types or kinds of higher-kinded types as well.
For example, if you want to write a function generic over functors, you don't need to explicitly write down the kind of `F`:

```rust
fn doSomethingAboutFunctors[F, A][(Functor[F])](F[A]) -> Unit;
```

# GADTs

Normal `data` are called ADTs in Haskell.
GADTs are the **G**eneralized version of ADTs.
GADTs let you be specific on the return types of the variants.
We can define a type with GADT by dropping the equal sign (`=`) after the name of the `data`.

```rust
data Array[_ : Nat, T] {
    nil : Array[0, T],
    fn cons[n : Nat](T, Array[n, T]) -> Array[Nat::succ(n), T],
};
```

You can mix `dyn data` and GADTs; variants in GADTs can also be made `dyn`.

# Variadic Arguments

Usually, we don't want to make arguments in normal mode curryable.
But sometimes we do want to supply variable length of arguments.
There is a kind of mechanism in Ende to make genericity over arity possible.
Because I want to make variadic arguments as flexible as possible, it could be a little bit harder to understand.
I need to introduce a new kind of types called **tuple types**.
They are not really the same as tuples in Rust or Haskell.
Tuple types are type-level lists.
for example, `Unit, Bool` is a tuple type, `I32, F32, U64` is also a tuple type.
Although they are not real Ende terms because making them real terms would make the grammer ambiguous.
What is the kind of all tuple types then?
It's called the **ordered dynamic type**, and is written `..(Type)`

Now, I'm going to show you how to write a function accepting arbitrarily many arguments.
For clarity, let's consider a rather easy example first.
The function `sum` sums up all the `I32`s in the argument list no matter how many arguments there are.
First, I have to define a helper function for it:

```rust
const fn replicate[_ : Nat](Type) -> ..(Type) {
    [0](_) => dyn (),
    [Nat::succ(n)](T) => dyn (T, replicate[n](T))
}
```

Because of the ambiguity I mentioned before, a tuple type cannot be written down directly.
Instead, we write `dyn (something)` to write down a tuple type.
(The keyword `dyn` is overloaded again.)
`dyn ()` is an empty tuple type; `dyn (A, B, C)` is `A, B, C`; `dyn (A, B, C, D)` is `A, B, C, D`, etc.
A tuple type with only one element is written `dyn (A,)`.
Now focus on the `replicate` function above.

- `replicate[0](T)` is an empty tuple type.
- `replicate[1](T)` = `dyn (T, replicate[0](T))` = `T`.
  (It's not the type `T`, but the tuple type that has only one element.)
- `replicate[2](T)` = `dyn (T, replicate[1](T))` = `dyn (T, T)` = `T, T`.

So `replicate[n](T)` is `T` repeated for `n` times.

What are the types of the arguments of the `sum` function?
They are `I32` repeated for arbitrarily many times!
Now you can see how `replicate` could be useful.
In order to say that an argument in fact represents many arguments, we use the keyword `dyn` again.
a `dyn` argument can only appear at the end of an argument list.
It would be easier to understand by providing the example than describing it in words:

```rust
const fn sum[Args : replicate(I32)](dyn _ : Args) -> I32 {
    () => 0i32,
    (head, dyn tail) => head + sum(tail),
};
```

In contrast to the ordered dynamic type, there is `..{Type}`, which is the **unordered dynamic type**, the type of maps from identifiers to types.
This could be used for duck typing or row polymorphism, e.g.

(TBD)

# Dependent Types

In the above examples, we've seen types depending on values at compile time, but not values at runtime.
In order to be fully dependently-typed, another mode called **pi mode** has to be introduced:

| ordered                   | normal    | `const`      | instance      | pi            |
|:-------------------------:|:---------:|:------------:|:-------------:|:-------------:|
| `T` means                 | `(_ : T)` | `[T : _]`    | `[(_ : T)]`   | `([T : _])`   |
| type inference            | no        | yes          | no            | yes           |
| phase (if not `const fn`) | runtime   | compile time | compile time  | runtime       |
| curryable                 | no        | yes          | yes           | no            |
| argument inference        | no        | yes          | proof search  | no            |
| can be dependent on       | no        | yes          | yes           | yes           |
| unordered                 | `{t : T}` | `{[t : T]}`  | `{[(t : T)]}` | `{([t : T])}` |
| type inference            | no        | no           | no            | no            |
| phase (if not `const fn`) | runtime   | compile time | compile time  | runtime       |
| curryable                 | no        | yes          | yes           | no            |
| argument inference        | no        | yes          | proof search  | no            |
| can be dependent on       | no        | yes          | yes           | yes           |


`(T)` and `[(T)]` mean `(_ : T)` and `[(_ : T)]`, respectively, but `[T]` and `([T])` mean `[T : _]` and `([T : _])`, respectively.
Types of arguments in `const` and `pi` mode are inferred.
Usually arguments in normal mode or pi mode are supplied at runtime, but not arguments in `const` or instance modes.
Arguments in `const` or instance modes are curryable because they have nothing to do with the runtime.
Arguments in normal mode cannot be inferred and cannot be dependent on obviously.
Arguments in an argument list can only be dependent on arguments in previous argument lists.

The last argument in a list of arguments in pi mode can also be `dyn` so it can accept variadic arguments.

Existential types as an example of pi-types:

```rust
class Sigma[A][B : ([A]) -> Type] = sigma([a : A])(B([a]));
```

## `with`

Dependent functions might return terms of different types depending on the arguments in pi mode.
But all arms of a `match` must return terms of the same type.
To solve it, I introduce **dependent pattern matching** through `with` clauses.
If you write the function without the equal sign before the curly braces, you can do pattern matching at the top level.
Top-level pattern matching can include arms with `with` clauses.
Here's an example:

(TBD)

# Universes

What's the type of `Type`?
In Ende, `Type` is actually not a single type, but a series of types.
You can think that `Type` has a hidden natural number parameter.
The `Type` that all of us are familiar about is `Type[0]`, but the type of `Type[0]` is `Type[1]`, the type of which is `Type[2]`, and going on and on.
What about the types of function types?
The answer is:

```
A : Type[m]    B : Type[n]
--------------------------
 A -> B : Type[max(m, n)]
```

Imagine if we want to accept a potentially infinite list of arguments types of which are `Int, Type[0], Int, Type[0], Int, Type[0] ...`.
How do we write a helper function to generate the tuple type?
The dynamic type of the return type of the function cannot be `..(Type[0])` because the type of `Type[0]` isn't `Type[0]`.
The answer is to make universes cumulative:

```
      m < n
------------------
Type[m] <: Type[n]
```

and

```
T : Type[m]    Type[m] <: Type[n]
---------------------------------
           T : Type[n]
```

A term of a dynamic type is a list of types the types of all of which are the same:

```
T1 : U    T2 : U    T3 : U    ...
---------------------------------
     T1, T2, T3 ... : ..(U)

T1 : U    T2 : U    T3 : U    ...
---------------------------------
     T1, T2, T3 ... : ..{U}
```

Now that `Int : Type[1]`, so the type of `Int, Type[0], Int, Type[0], Int, Type[0] ...` is `..(Type[1])`.

## Hierarchies

It seams reasonable to assume that

```
    A <: B
--------------
..(A) <: ..(B)
```

and

```
    A <: B
--------------
..{A} <: ..{B}
```

Now we have at least 3 different hierarchies of universes, one is

```rust
Type[0] : Type[1] : Type[2] : Type[3] ...
```

, another two are

```rust
..(Type[0]) : ..(Type[1]) : ..(Type[2]) : ..(Type[3]) ...
..{Type[0]} : ..{Type[1]} : ..{Type[2]} : ..{Type[3]} ...
```

In reality there are infinite hierarchies because `..(..{Type})` and `..{..(Type)}` are also hierarchies.
What comes next in my mind is to make hierarchies user-definable.
Here is the syntax for defining a new hierarchy:

```rust
data AnotherWorld : AnotherWorld;
```

It seems that the type of `AnotherWorld` is itself, but that would make the type system inconsistent.
What it actually does is to create another hierarchy that is also cumulative:

```rust
AnotherWorld[0] : AnotherWorld[1] : AnotherWorld[2] : AnotherWorld[3] ...
```

Normally data types live in `Type`, but we can also define data types that live in another hierarchy:

```rust
data WhatTheHeck : AnotherWorld = whatever;
class YouAreCrazy : AnotherWorld = youAreCrazy {};
```

Now types of function types are:

```
World : Hierarchy    A : World[m]    B : World[n]
-------------------------------------------------
            A -> B : World[max(m, n)]
```

Function types from a hierarchy to another hierarchy such as the above `replicate` has no type; they are similar to `Setω` in Agda.

# Open Problems

1. Can function types from a hierarchy to another hierarchy have a type?
2. What is the use of hierarchies other than `Type`?
   I'm thinking that it should be able to postulate extra axioms which shouldn't be done in `Type` because they break the totality of `const fn`s at runtime.
2. Is it possible to be generic over hierarchies?
3. Is it possible to define new *hierarchy constructors* other than `..()` and `..{}`?
   If it's possible, I shouldn't use special syntax on dynamic types but something like `OrderedDynamic[Hierarchy]`.
   Is the ambiguity between an universe and a hierarchy okay?
