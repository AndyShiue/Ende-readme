# Ende

I just want to share what I've come up with about the design of a new programming language, and here it is.
This language has a strong type system, its functions are call-by-value, and its syntax is whitespace-insensitive because I prefer it.
It's very welcomed for anyone to write an implementation for it.

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Ende](#ende)
- [Overview](#overview)
- [Comments](#comments)
- [Terms and Statements](#terms-and-statements)
- [Functions](#functions)
	- [lambdas](#lambdas)
- [Lang Items](#lang-items)
- [User-Defined Data Types](#user-defined-data-types)
- [Visibility](#visibility)
- [`mod`s](#mods)
- [Generics](#generics)
- [Named Mode](#named-mode)
- [More General `record`](#more-general-record)
	- [`impl` objects](#impl-objects)
	- [`impl` functions](#impl-functions)
	- [`fundamental impl`](#fundamental-impl)
		- [The Hack](#the-hack)
		- [Searching for `impl`s](#searching-for-impls)
	- [Associated Types](#associated-types)
	- [`auto impl`](#auto-impl)
	- [Visibility of `impl`s](#visibility-of-impls)
- [Memory Management](#memory-management)
	- [Heap Allocation](#heap-allocation)
	- [Moving](#moving)
	- [Reference Types](#reference-types)
- [`const`](#const)
	- [`const(fundamental)`](#constfundamental)
	- [A `const` Version `factorial`](#a-const-version-factorial)
- [More Powerful Generics](#more-powerful-generics)
- [GADTs](#gadts)
- [Variadic Arguments](#variadic-arguments)
- [Phase Polymorphism](#phase-polymorphism)
	- [The Problem](#the-problem)
	- [The Solution](#the-solution)
- [Dependent Types](#dependent-types)
	- [`with`](#with)
- [Universes](#universes)
	- [Hierarchies](#hierarchies)
- [Open Problems](#open-problems)
- [TODOs](#todos)

<!-- /TOC -->

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
In complement to modes, you can specify stuff that works at either runtime or compile time using the `const` system of Ende.

The presence of modes makes almost everything in Ende first-class, which means they can be passed as arguments and returned.
They include:

1. all functions
2. all structs
3. all class constructors
4. all traits
5. all implementations of traits
6. all extensions between traits
7. all modules
8. the vast majority of types

As you will see, Ende achieve the unification of different concepts with carefully designed syntax and semantics.

The syntax is very similar to Rust's.
If you are familiar with Rust, you might start reading from [this section](#more-general-record).
Just be aware of the top-level pattern matching syntax and that generic parameters are written inside square brackets (`[]`) instead of angle brackets (`<>`).
Users of Scala might also find the similarities between Scala and Ende.
I actually took lots of ideas from Rust, Scala, Agda, Idris, etc.

# Comments

There are 2 kinds of comments in Ende.
The first is line comments: they start with `--` and extend toward the end of the line.
The other one is block comments.
They can span over multiple lines.
They start with `{-` and end with `-}`.

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

   2. **Floats**:
      Literals of floats would be similar to ones of integers, e.g. `2.71828f32`.

2. Applications of operators:
   When one talks about operators, we usually think of infix operators.
   But in actual implementations, operators of any fixity could be introduced.
   There could possibly even be user-defined mixfix operators.

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
let number = if count == 0i32 then 42u32 else 666u32;
```

Of course, `if` can also be used in an old-style, C-like fashion.

```rust
let mut number;
if count == 0i32 then {
    number = 42u32;
} else {
    number = 666u32;
};
```

# Functions

Functions are the simplest concept that has the ability to encapsulate information in functional programming languages; function calls are terms.
The declaration of a function is an *item*.
Items are basically some construct that can appear at the top level of the source code.

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
    if n == 0
    then 0u32
    else n * factorial(n - 1)
};
```

## lambdas

Lambdas are unnamed function literals.
Like normal functions, lambdas must specify their return types.
Lambdas are written similar to a function but without the name.

```rust
let factorial = fn(n : U32) -> U32 = {
    if n == 0u32
    then 0u32
    else n * factorial(n - 1)
};
```

# Lang Items

In Rust and also in Ende, there's a concept of *lang items*.
Lang items are items that are treated specially, having something to do with the compiler.
I won't introduce a keyword for lang items but instead use *annotations* to mark them.
Annotations are written before the annotated item, start with an at sign (`@`) and is followed by a colon (`:`).

```rust
@lang("Whatever"):
data Whatever;
```

# User-Defined Data Types

Data types are items.
They can be defined with the keyword `data`.
They can be seen as `enum`s in Rust and are more powerful than that of C/Java.
I'll first consider its easiest usage, though.
Let's show how to define a C/Java-like `enum` in Ende:

```rust
@lang("Unit"):
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
-- let season = spring; -- Doesn't compile.
```

`data` variants could be matched through pattern matching:

```rust
fn isSummer(season : Season) -> Bool = {
    match season {
        Season::summer => Bool::true,
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

There's another way to define a data type, using `record`.
`record`s in Ende are like `struct`s in Rust.
`record`s can be seen as `data` with only one variant.
The name of the variant isn't namespaced, though.

```rust
record Point = point {
    x : I32,
    y : I32,
};
```

Members of an instance of a `record` can either be accessed by its name after a dot (`.`) or by pattern matching.

```rust
fn getX1(p : Point) -> I32 = p.x;
fn getX2(Point) -> I32 {
    (point {x => result, y => _}) => result,
};
```

To pattern match or to construct an instance of a `record`, we write a fat arrow (`=>`) after each name of the fields instead of a colon (`:`) used when a `record` is defined:

```rust
let p = point { x => 0i32, y => 0i32 };
```

Someone mentioned that I sometimes write a trailing comma after the last matching arm or the last field of a `record`.
That's intentionally designed.
It's the same as the situation in Rust: all trailing commas are optional.
you can even write

```rust
data A =
    a,
    b,
    c,
    d,
;
```
# Visibility

(TBD)

# `mod`s

`mod`s are containers of items, and `mod`s themselves are also items, so `mod`s can be nested.
We use the keyword `mod` to declare a module.

```rust
pub mod module {
    fn unit() -> Unit = Unit::unit;
    data Three = one, two, three;
    pub mod inner {
        pub record Circle = circle {
            radius : U32,
        };
    };
};
```

A `mod` can be declared without curly braces (`{}`).

```rust
mod somewhereElse;
```

If the compiler sees such `mod`, the compiler will look for `./somewhereElse.ende` and `./somewhereElse/mod.ende` to read its content.
If neither are presented, the compiler emits an error.

There are two ways to access an item in a `mod`.
one is to write its fully qualified name, e.g. `module::inner::Circle`.
the other way is to use `use` statements to import the items.

```rust
use module::inner::Circle;
use module::inner::circle;

fn getCircle() -> Circle = circle {
    radius : 1u32,
};
```

An underscore (`_`) can be used as a wildcard to import everything in a module or in a `data`, so instead of writing 2 `use` statements, you can write

```rust
use module::inner::_;
```

`mod`s are also first-class value of type `Mod`:

```rust
@lang("Mod"):
pub const(fundamental) data Mod;
```

Here, `const(fundamental)` means the instance of `Mod` can only exist at compile time.
I'll show how to manipulate `data` at compile time later.

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

I haven't mentioned function types, have I?
Function types are literally the types of functions and are written as `(A, B, C, ...) -> R`.
`A, B, C, ...` are the types of the arguments, and `R` is the return type.
As a side note, arguments in normal mode cannot be curried in Ende similar to the ones in C++/Scala/Rust, but arguments in `const` mode can:

```rust
-- They are different:

foo(a, b);
foo(a)(b);

-- But they aren't:

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

Compare this `record` in Ende

```rust
record Example = example {
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
In order to assign a type to it, we need new modes in which parameters are named.
Let's simply call them **named mode**.
The named normal mode is similar to the normal mode, except that the parameters are named and unordered; the named `const` mode corresponds to the `const` mode.
The above `example` now has type `{example : Unit} -> Example`.
To be more general, now we can also have struct variants and arbitrary functions accepting named parameters.

# More General `record`

`record`s in Ende can not only act as Rust `struct`s but also Rust `trait`s/ Java `interface`s/Haskell `class`es.
I'll call them traits in the rest of the article if they are used like a Rust `trait`.
Here, I'm going to write a `Monoid` trait.
Of course, it's just another `record`.

```rust
record Monoid[T] = monoid {
    unit : T,
    fn append(self : T, T) -> T,
};
```

To implement a trait, I introduce another keyword `impl`, unlike the `impl`s in Rust or `instance`s in Haskell, `impl`s in Ende are always named.
There are 3 kinds of `impl`s in Ende.

## `impl` objects

`impl` objects are the first kind of `impl`.
The `i32Monoid` below is an `impl` object.
An `impl` object must be an instance of a `record`.

```rust
impl i32Monoid : Monoid[I32] = monoid {
    unit => 0i32,
    fn append(self : I32, another : I32) -> I32 = self + another,
};
```

If the `impl` object `i32Monoid` is in scope, now we can call the `append` method on `I32`:

```rust
-- They are equivalent because the first argument of `append` is `self`:

let sum1 = append(1i32, 2i32);
let sum2 = 1i32.append(2i32);
```

In order to write a function that is generic over types implementing a `record`, the third mode is introduced.
It's called **instance mode**, and is delimited by `[()]`.
Arguments passed in instance mode must be instances of `record`s.
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
An `impl` function must return an instance of a `record`.
`impl`s need to be functions mainly because of 2 reasons:

1. It's generic over an argument in the `const` mode.
2. It's generic over an argument in the instance mode.

I'm going to provide a `impl` function generic over both modes.
First, define a `Group` `record`.

```rust
record Group[T] = group {
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

## `fundamental impl`

What we don't have yet is the ability to define one `record` to be a supertrait of another `record`. In other words, asserting if you implement a trait, another trait must be implemented.
You may ask, isn't the above `impl` functions enough?
Not really.
Imagine you want to define an `Abelian` trait, the code you need to add would be:

```rust
record Abelian[T] = abelian {
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
The code is very similar to implementing `Monoid`s for `Group`s; we have to write this kind of code again and again while creating an inheritance tree of `record`s.
That is not tolerable.
Imagine if we can write code like this:

```rust
-- The type `Abelian` carries no additional data, but it extends the trait `Group`.
data Abelian[T] = abelian;
impl abelianExtendsGroup[T] -> Extends[Abelian[T], Group[T]] = extension; -- The extension happens here.
```

It's really concise code!
But how do we get an instance of type `Group[T]` given the instances of types `Abelian[T]` and `Extends[Abelian[T], Group[T]]`?
It's actually a hack: we need more `impl`s.
I'll try to explain it.

### The Hack

First, I'll provide the hacky part of the source code:

```rust
record Extends[A, B] = extension;

-- Ignore the `const` keyword before `fn` for now.
const fn implicitly[T][(inst : T)] -> T = inst;

fundamental impl superTrait[A, B][(A, Extends[A, B])] -> B = implicitly[B];
```

The basic idea is that no matter what `A` and `B` are, if `impl`s of `A` and `Extends[A, B]` are in scope, `impl` of `B` is made in scope.
In the revised version of example of `Abelian`,  because both `impl`s of `Abelian[T]` and `Extends[Abelian[T], Group[T]]` are in scope, `impl` of `Group[T]` is also made in scope.
Why is the keyword `fundamental` before the `impl superTrait` required then?
To know why it's needed, we need to go deeper to know how an `impl` is found.

### Searching for `impl`s

First, we search for `impl` objects.
We add an `impl` object in scope to the current **`impl` context** if the trait it implements is also in scope.

Second, we add the `impl` functions to the `impl` context.
There is a necessary limitation of normal `impl` functions:
a normal `impl` function can only have a return type that is not a variable, so the example below does't work:

```rust
-- impl abuseOfImplicitly[T] -> T = implicitly[T]; -- Doesn't compile.
```

The reason why some limitation is needed is because we want to make searching `impl`s more predictable, so that we can filter out the `impl` functions that doesn't retern an `impl` of a type in scope.
Without the limitation, the `impl` searching process could stuck at some weird recursive `impl`.

And `fundamental impl`s surpass that limitation.
It has to be used more carefully, but I don't think there's a lot of uses of it.
In fact the only one I can think of is trait inheritance.
The `impl`s of the return types of the `fundamental impl`s are recursively added to the `impl` context no matter whether the type it implements is in scope or not.

## Associated Types

Fields of an instance of a `record` can also be dependent.
They are different from normal *input parameters* in that they don't determine the `impl` chosen but the `impl`s determine them.
They are *output parameters*.
For example, imagine if there is a way to overload operators.
There should be an trait for each operator.
To achieve maximal flexibility, I want to make types of the right-hand-side, the left-hand-side, and the returned terms possibly different.
It means it has to be generic over these 3 types.
So maybe the trait could be like:

```rust
@lang("Add"):
pub record Add[L, R, Output] = add {
    fn add(self : L, R) -> Output,
};
```

However, the trait is problematic because we can provide both instances of `Add[L, R, A]` and `Add[L, R, B]`; if I write `(l : L) + (r : R)`, the compiler wouldn't be able to know if the type of the result would be `A` or `B`.
This suggests that the `Output` type should not be an input parameter but rather an output one determined uniquely by `L` and `R`.
The correct trait should be:

```rust
@lang("Add"):
pub record Add[L, R] = add[Output] {
    fn add(self : L, R) -> Output,
};
```

If you want to access the output type specified by an instance of a trait, simply write `inst.Output`.

## `auto impl`

`auto impl`s are `impl`s with a `self` parameter in normal mode.
The keyword `auto` indicates they are `impl`s that will be automatically inserted.
For example, if one wants to overload string literals, they could provide an `auto impl` from `String` to whatever type they want.
For now, let's assume that type is called `StrLike`.
The Ende source code would be something like:

```rust
auto impl strLike(self : String) -> StrLike = ...;
```

`auto impl`s could be inserted at any node in the term AST if the expected type doesn't match the actual type, so if a term `str` occurs in the source code, it could possibly be transformed to `str.strLike()` everywhere.
`auto impl`s wouldn't be inserted more than once at a particular node, however, which means the following code doesn't type check.

```rust
auto impl firstToSecond(self : First) -> Second = ...;
auto impl secondToThird(self : Second) -> Third = ...;

fn manipulateThird(third : Third) -> Third = third;

let second : Second = ...;
manipulateThird(first); -- It works.

let first : First = ...;
-- manipulateThird(first); -- `first` cannot be transformed to value of type `Third`.
```

## Visibility of `impl`s

(TBD)

# Memory Management

Until now, the language is still fairly compatible with system programming.
Functions are ... well, functions; non-capturing lambdas are function pointers; `record`s in Ende can be implemented as structs in C: I didn't say recursive types work.
Surely some form of recursive type must be implemented in order to make Ende really useful, but I think it should be done with explicit pointer; I will discuss on it later.
`enum`s are enums in C each with a tag indicating its variant to make them type safe.
`mod`s are `mod`s.
Generics are monomorphized at compile time.
`impl`s have nothing to do with runtime.

## Heap Allocation

Perhaps the most important topic in memory management is heap allocation.
In order to make Ende a system programming language, users of the language must be able to manipulate raw pointers.
But in addition to it, there are supposed to be a higher-level interface for normal programmers since manipulating raw pointers are highly unsafe thus error-prone.
I've consider 3 different approaches in the past:

1. **The Rust Approach**:
   closest to bare metal and theoretically the most efficient but requires lots and lots of lang items.

2. **The Swift Approach**:
   Doesn't require a garbage collector (GC), but instead implicitly using reference counting everywhere.
   Cycles between reference counted (RC) pointers could cause memory leak and users have to be very careful about it.
   Dereferencing an unowned pointer could fail.

3. **The Java Approach**:
   GC everywhere.
   The safest solution the easiest to use, but one sometimes needs to *stop the world*.
   Bad for application needing low latency.

I prefered the approach of using RC *explicitly* at the beginning, but found out that doesn't work quite well.
One would end up need to write out RC almost everywhere.
Although that would arguably make Ende *Rust++*, I think explicit is better than implicit and by far Rust's solution is the best one also because implementing ownership doesn't prevent Ende from implementing explicit RC/GC.
I'll try to descibe the compiler work needed in the rest of this section.

## Moving

By default, values in Ende can only be used once.
Using a value *consumes* or **move**s it.

```rust
data Movable = movable;

let original = Movable::movable;
let moved = id(original);
-- let moveAgain = id(original); -- Cannot move a value more than once.
```

But sometimes we really want to use some *plain old data* more than once.
It's time for the `Copy` trait to come to rescue.

```Rust
@lang("Copy"):
record Copy[T] = copy;
```

`Copy` is a built-in trait that can be implemented by types that can be copied bitwise.
It is implemented for all the primitive types introduced above, but I'll mention some very important types that aren't copyable very soon.
Users can also implement `Copy` for the types they define.
A `data` can implement `Copy` if and only if all parameters of all of its variants implemented `Copy` (with some exception,  which will be discussed below).
Similar for a `record`.

```Rust
record I32Wrapper = wrap {
    inner : I32,
};

impl copyableI32Wrapper : Copy[I32Wrapper] = copy;
```

## Reference Types

(TBD)

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

   1. The term immediately after `if` is a constant and evaluates to `true`, and the term after `then` represents a constant.

   2. The term immediately after `if` is a constant and evaluates to `false`, and the term after `else` represents a constant.

5. **Functions**:
   Functions that aren't inside a `record` are always constants, but there's a difference between `const fn`s and normal functions.
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
   There are also `const fn` lambdas.

6. **`impl`s**:
   Did I mention `impl`s are first-class citizens of Ende?
   They can be returned and passed as arguments.
   `impl`s are always constants; all `impl`s mentioned before are also `const fn`s.
   Nevertheless, there exist `impl`s that aren't `const fn`s.
   They are `dyn auto impl`.
   `dyn auto impl`s are `auto impl`s that operate at runtime.
   `dyn auto impl`s exist because sometimes we can never get the value of the self parameter at compile time, e.g. dereferencing a smart pointer into its underlying type.
   `dyn auto impl`s aren't guarantee to be total.

7. **`data`**:
   In normal `data`, all variants are constants.
   In addition, all variants with parameters in `data` are `const fn`s.

8. **`record`s**:
   An instance of a `record` is a constant if all of its fields are constants by default.
   In comparison to `data`, the problem of constness is even more serious, though.
   When you write a `record`, I assume you want not only that the fields are constants, but also the function members are `const fn`s, and that's the default.
   Here is such an example, the `Monoid` trait we've talked about.

   ```rust
   record Monoid[T] = monoid {
       unit : T,
       fn append(self : T, T) -> T,
   };

   impl i32Monoid : Monoid[I32] = monoid {
       unit => 0i32,
       fn append(self : I32, another : I32) -> I32 = self + another,
   };

   const _ = 0i32.append(0i32); -- It works.
   ```

   But sometimes you want to use `record`s as Java `class`es, in that case, you want all non-function fields to be mutable, and all function members to be non-`const` functions.
   Classy `record`s are used to define such classes.
   To define classy `record`s, you write `dyn` after the equal sign (`=`) after the name of the `record` and before the record constructor.
   Classy `record`s can only be binded using `let mut` but not `let`.
   You can also write `dyn` in front of a function member in a non-`dyn` `record` to indicate it's not a `const` function.

   ```rust
   record Counter = dyn counter {
       inner : I32,
       fn increment(self : Counter) -> Unit,
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
   record Wierd = dyn duh {
       fn dyn wierd : () -> Unit,
   };

   fn doNothing() -> Unit = Unit::unit;

   let mut wierd = duh {
       wierd => main,
   };

   wierd.weird = doNothing; -- It works.
   ```

   An instance of a non-classy `record` is a constant if all of its fields are constants.
   An instance of a classy `record` is never a constant.
   A classy `record` cannot have field in any mode other than named/unnamed normal mode.

## `const(fundamental)`

(TBD)

## A `const` Version `factorial`

I'll define a `const fn factorial` in this subchapter.
The implementation of this `factorial` is different from the `U32` version above.
This is not to suggest Ende has overloading.
Just pretend the 2 functions are from different spaces.

The problem with `U32` is that it's is not defined recursively, so it would be harder for the computer to figure out if the functions terminate.
The solution is to use a recursively defined data type: `Nat`

```rust
@lang("Nat"):
data Nat = zero, succ(Nat);
```

`Nat` is special that it has a literal form.
Literal of type `Nat` has a suffix `nat`.

And here's the `const fn factorial`:

```rust
const fn factorial(m : Nat) -> Nat {
    (0nat) => 1nat,
    (Nat::succ(n)) => m * factorial(n),
};
```

# More Powerful Generics

Arguments in `const` mode need not be types; they can be constants as well.
In fact, **any** constants which is typeable can be passed to a function in `const` mode.
So I can write something like:

```rust
const fn factorial[m : Nat] -> Nat {
    [0nat] => 1nat,
    [Nat::succ(n)] => m * factorial(n),
};
```

But then we lose the ability to call `factorial` at runtime.

Actually, types are also first-class citizens of Ende.
They are constants.
In Ende, we can also be generic over type constructors, which are `const fn`s at the type level.
That is, to pass higher-kinded types around.
Normal types have kind `Type`.
However, `Option` is a type constructor and is of kind `[Type] -> Type`, which means it is a function from a type at compile time to another type.
Here's the `Functor` trait.

```rust
record Functor[F : [Type] -> Type] = functor {
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
    nil : Array[0nat, T],
    fn cons[n : Nat](T, Array[n, T]) -> Array[Nat::succ(n), T],
};
```

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
It's called the **ordered variadic type**, and is written `..(Type)`

Now, I'm going to show you how to write a function accepting arbitrarily many arguments.
For clarity, let's consider a rather easy example first.
The function `sum` sums up all the `I32`s in the argument list no matter how many arguments there are.
First, I have to define a helper function for it:

```rust
const fn replicate[_ : Nat](Type) -> ..(Type) {
    [0nat](_) => tuple (),
    [Nat::succ(n)](T) => tuple (T, replicate[n](T))
}
```

Because of the ambiguity I mentioned before, a tuple type cannot be written down directly.
Instead, we write `tuple (something)` to write down a tuple type.
`tuple ()` is an empty tuple type; `tuple (A, B, C)` is `A, B, C`; `tuple (A, B, C, D)` is `A, B, C, D`, etc.
Now focus on the `replicate` function above.

- `replicate[0nat](T)` is an empty tuple type.
- `replicate[1nat](T)` = `tuple (T, replicate[0nat](T))` = `T`.
  (It's not the type `T`, but the tuple type that has only one element.)
- `replicate[2nat](T)` = `tuple (T, replicate[1nat](T))` = `tuple (T, T)` = `T, T`.

So `replicate[n](T)` is `T` repeated for `n` times.

What are the types of the arguments of the `sum` function?
They are `I32` repeated for arbitrarily many times!
Now you can see how `replicate` could be useful.
In order to say that an argument in fact represents many arguments, we overload the keyword `dyn`.
a `dyn` argument can only appear at the end of an argument list.
It would be easier to understand by providing the example than describing it in words:

```rust
const fn sum[Args : replicate(I32)](dyn _ : Args) -> I32 {
    () => 0i32,
    (head, dyn tail) => head + sum(tail),
};
```

In contrast to the ordered variadic type, there is `..{Type}`, which is the **unordered variadic type**, the type of maps from identifiers to types.
This could be used for duck typing or row polymorphism, e.g.

(TBD)

# Phase Polymorphism

## The Problem

`const` still isn't flexible enough in some situation.
See the following 2 examples for instance:

1. **The `curry` function**:
   Having variadic arguments, we should be able to define a function that curries the arguments of another function.
   That is, if the function `func` has type `(I32, U32, F32) -> Bool`, `curry(func)` should have type `(I32)(U32)(F32) -> Bool`.

   let me show you how to define such function.

   ```rust
   -- Don't ask what `???` is for now.
   -- Just pretend it's magic; I'll debunk it later.
   -- The type system still isn't strong enough to actually write it down.

   -- We can also pattern match the tuple types instead of the values of the tuple types.)
   const fn curriedFuncType(Type, dyn _ : ..(Type)) -> ??? {
       (Ret) => Ret,
       (Ret, Head, dyn Tail) => (Head) -> curriedFuncType(Ret, Tail),
   }

   const fn curry[Args : ..(Type), Ret](func : (dyn Args) -> Ret)
       -> curriedFuncType(Ret, dyn Args) {
       (fn() -> Ret = ret) =>
           ret,
       [tuple (Head, dyn Tail)](fn(head : Head, dyn tail : Tail) -> Ret = ret) =>
           fn(head : Head) -> curriedFuncType(Ret, Tail) = curry(fn(dyn tail : Tail) -> Ret = ret),
   }
   ```

   What's problematic about it?
   The problem is:

   > If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.

   If the `func` is a constant but **is not** a `const fn`, because the above `curry` function is a `const fn`, `curry(func)` **is** also a `const fn`.
   Now there's a contradiction, so the compiler should not accept the definition of `curry`.
   But we want it to be a `const fn`!
   Moreover, if the input `func` is a `const fn`, we want the output function to be also a `const fn`.

2. **Lazy evaluation**:
   If mixfix operators are implemented, maybe `if_then_else` can be implemented as a function, but we now need a way to say that some of the parameters are passed in lazily:

   ```rust
   const fn if_then_else[T](_0_ : Bool, lazy _1_ : T, lazy _2_ : T) -> T {
       match _0_ {
           Bool::true => _1_,
           Bool::false => _2_,
       }
   }
   ```

   However, this is not quite right, either.
   Because if `_0_` evaluates to `true`, the constness of `_2_` doesn't matter to the constness of the whole function.

## The Solution

The solution is to give the user finer control of the `const` system.
Instead of writing `const var` or `dyn var`, we can write `phase ph var` to specify its phase further.

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

The last argument in a list of arguments in pi mode can also be `dyn`.

Existential types as an example of pi-types:

```rust
record Sigma[A][B : ([A]) -> Type] = sigma([a : A])(B([a]));
```

## `with`

The value of one argument of a dependent function might depend on another argument in pi mode.
We need to be able to match several arguments together.
Introduce **dependent pattern matching**, through `with` clauses.
If you write the function without the equal sign before the curly braces, you can do pattern matching at the top level.
Top-level pattern matching can include arms with `with` clauses.
Here's an example:

(TBD)

# Universes

What's the type of `Type` and type constructors?
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

i.e. variadic types are covariant.
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
record YouAreCrazy : AnotherWorld = youAreCrazy;
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
3. Is it possible to be generic over hierarchies?
4. Is it possible to define new *hierarchy constructors* other than `..()` and `..{}`?
   If it's possible, I shouldn't use special syntax on variadic types but something like `OrderedVariadic[Hierarchy]`.
   Is the ambiguity between an universe and a hierarchy okay?

# TODOs

1. Foreign function interface.
2. Equality types. I don't know how to do it.
3. Provisional definitions.
