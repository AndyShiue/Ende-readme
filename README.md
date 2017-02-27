# Named Mode

Compare this record in Ende

```
data Example = new {
    "example" -: Unit
}
```

and this `struct` in Rust:

```
struct Example {
    example: ()
}
```

Why do we need to write `new` after the equal sign (`=`)?
It's not a keyword!
The intention is to disambiguate between the type `Example` and the constructor of it.
In Rust, which doesn't have dependent types, `Example` can mean both, and a usage of `Example` can always be resolved, but not in a language in which types are first-class.

Variants in `data` have types, what is the type of `new` then?
In order to assign a type to it, we need new modes in which parameters are named.
Let's call them **named mode**s.
The named normal mode is similar to the normal mode, except that the parameters are named and unordered; the named `const` mode corresponds to the `const` mode.
The above `new` is now of the type `{"example" -: Unit} -> Example`.
To be more general, now we can also have struct variants and arbitrary functions accepting named parameters accepting arguments in named modes.

# More General records

Records in Ende can not only act as Rust `struct`s but also Rust `trait`s or Java `interface`s or Haskell `class`es.
I'll call them traits in the rest of the article if they are used like a Rust `trait`.
Here, I'm going to write a `Monoid` trait.
Of course, it's just another record.

```
pub(in) data Monoid[T] = new {
    "unit" -: T
    "append" -: (self : T, T) -> T
}
```

To implement a trait, I introduce another keyword `impl`.
All `impl`s are items and can be defined locally in a function body.
Unlike the `impl`s in Rust or `instance`s in Haskell, `impl`s in Ende are always named.

## Simple `impl`s

The `i32Monoid` below is a simple `impl`:

```
pub impl i32Monoid : Monoid[I32] = Monoid::new {
    "unit" -: 0i32
    "append" -: fn(self : I32, another : I32) -> I32 = self + another
}
```

If `i32Monoid` is in scope, we can call the `append` method on `I32`:

```
\\ They are equivalent because the first argument of the field of `"append"` is `self`:

let sum1 = i32Monoid."append"(1i32, 2i32)
let sum2 = 1i32#append(2i32)
```

In order to write a function that is generic over types implementing a trait, the third mode is introduced.
It's called the **instance mode**, and is delimited by `[()]`.
Arguments in the instance mode can also be inferred, however not by looking at other arguments, but by searching for appropriate `impl`s.
Here is a function generic over types implementing `Monoid`; it sums up all the values in a `List` using `Monoid`'s `append` method:

```
fn concat[T][(Monoid[T])](List[T]) -> T where match
    (List::nil) => unit
    (List::cons(head, tail)) => head#append(concat(tail))
```

You can see that when you put an argument inside the instance mode, the fields of it are automatically brought into scope without the quotation marks (except those using weird characters that would be invalid identifiers in the syntax); it's just a syntax sugar.

When you write `concat(list)`, the compiler automatically chooses the right `impl` of `Monoid`; if there are 0 or more than 1 choices, the compiler generates an error.
Nonetheless, you can explicitly provide a specific `impl`, delimiting which in `[()]`:

```
let sum = concat[(i32Monoid)](i32Vec)
```

## `impl` Functions

`impl` functions are literally, `impl`s that are functions.
`impl`s need to be functions mainly because of 2 reasons:

1. It's generic over arguments in the `const` mode.
2. It's generic over arguments in the instance mode.

I'm going to provide an `impl` function generic over arguments in both modes.
First, define a `Group` trait.

```
pub(in) data Group[T] = new {
    "unit" -: T
    "append" -: (self : T, T) -> T
    "inverse" -: (self : T) -> T
}
```

We know all `Group`s are `Monoid`s, so all instances of `Group[T]` should be able to be automatically converted to instances of `Monoid[T]`.
How do we automatically convert stuff?
The answer is obviously, again, using `impl`s.

```
pub impl groupToMonoid[T][(Group[T])] -> Monoid[T] = Monoid::new {
    "unit" -: unit
    "append" -: append
}
```

`groupToMonoid` is indeed an `impl` function.

## Supertrait Arguments

What we don't have yet is the ability to define one trait to be a supertrait of another one.
In other words, asserting if you implement a trait, another trait must be implemented.
You may ask, isn't the above `impl` functions enough?
Not really.
Imagine you want to define an `Abelian` trait, the code you need to add would be:

```
data Abelian[T] = new {
    "unit" -: T
    "append" -: (self : T, T) -> T
    "inverse" -: (self : T) -> T
}

impl abelianToGroup[T][(Abelian[T])] -> Group[T] = Group::new {
    "unit" -: unit
    "append" -: append
    "inverse" -: inverse
}
```

That's a lot of boilerplate!
The code is very similar to implementing `Monoid`s for `Group`s; we have to write this kind of code again and again while creating an inheritance tree of traits.
That is not tolerable.
Can't we just store a `Group[T]` inside an `Abelian[T]`?
Yes, we can!
Moreover, we want to call the methods or more generally use the fields from the supertraits without accessing the fields explicitly.
**supertrait argument**s let us do all of that.

In order to use this feature, the trait `Group` and `Abelian` can be rewritten as below:

```
pub(in) data Group[T] = new[(Monoid[T])] {
    "inverse" -: (self : T) -> T
}
data Abelian[T] = new[(Group[T])]

\\ The `impl`s also have to be changed ...
```

As you can see, it's simply the instance mode after the constructor.

Searching of supertrait arguments isn't guaranteed to terminate because one could recurse on them, though.

## Associated Values

Fields of a record can also be dependent on the others.
They are different from normal *input parameters* in that they don't determine the `impl` chosen but the `impl`s determine them.
They are *output parameters*.
For example, imagine if I want to define a trait `Add` for all types that implement the `_+_` operator.
To achieve maximal flexibility, I want to make types of the the left-hand-side, right-hand-side, and the returned terms possibly different.
It means it has to be generic over these 3 types.
So maybe the trait could be like:

```
pub(in) data Add[L, R, Output] = add {
    "_+_" -: (self : L, R) -> Output
}
```

However, the trait is problematic because we can provide both instances of `Add[L, R, A]` and `Add[L, R, B]`; if I write `id(l : L) + id(r : R)`, the compiler wouldn't be able to know if the type of the result would be `A` or `B`.
This suggests that the `Output` type should not be an input parameter but rather an output one determined uniquely by `L` and `R`.
The correct trait should be:

```
pub(in) data Add[L, R] = add[Output] {
    "_+_" -: (self : L, R) -> Output
}
```

The `impl` ought to specify the `Output` type.
If you want to access the output type specified by an instance of a trait, simply do a pattern matching.
Or you can put them inside the `const` mode to make calling it with the dot syntax possible.

## `impl(auto)`

Automatic `impl`s are `impl`s with a `self` parameter in normal mode.
The keyword `auto` indicates they are `impl`s that will be automatically inserted.
For example, if one wants to overload string literals, they could provide an auto `impl` from `Str` to whatever type they want.
For now, let's assume that type is called `StrLike`.
The Ende source code would be something like:

```
pub impl(auto) strLike(self : Str) -> StrLike = ...
```

Auto `impl`s could be inserted at any node in the AST of a term if the expected type doesn't match the actual type, so if a term `str` occurs in the source code, it could possibly be transformed to `str#strLike()` anywhere.
Auto `impl`s wouldn't be inserted more than once at a particular node, however, which means the following code doesn't type check.

```
impl(auto) firstToSecond(self : First) -> Second = ...
impl(auto) secondToThird(self : Second) -> Third = ...

fn manipulateThird(third : Third) -> Third = third

let second : Second = ...
manipulateThird(first) \\ It works.

let first : First = ...
\\ manipulateThird(first) \\ `first` cannot be transformed to value of type `Third`.
```

(TBD: impl(auto) in an instance argument)

## Visibility of `impl`s

The visibility of `impl`s are the same as that of `fn`s.
Because people usually want to import all `impl`s in a `mod`, I purpose a little syntax to do that:

```
use some_module::impl
```

Syntax for importing all `fn`/`data`/`mod` in a `mod` could be implemented likewise.

# `const`

You may found that I wrote *`const` mode* instead of *const mode* throughout the article.
This is intentional.
`const` is an important keyword in Ende, and is highly related to the `const` mode.
The basic idea is that a *`const` something* is a *something* that can be done at compile time.
So a `const` value (a constant) is a value known at compile time, and a `const fn` is a function that could be run at compile time.

Now, let's go through all kinds of terms I introduced and see if they are constants.

1. **Literals**: They are definitely constants.

2. **Operator applications**: Operators are just functions with special names, and functions are discussed below.

3. **Variables**:
   A variable is a constant if it's declared as `const varName`.
   For example, in `const meaningOfLife = 42i32`, `meaningOfLife` is a constant.
   By contrast, in `let devil = 666i32`, `devil` **isn't** a constant.
   Of course, the right hand side of the `const` binding must be a constant as well.

4. **Control structures**:
   See the section about do-notation below.
   Specificly, The value of the entire `while` loop is not a constant (actually because it's not a `const fn`).
   `if`is special that the value of an `if` construct is a constant if and only if either of the conditions below is met.

   1. The term immediately after `if` is a constant and evaluates to `Bool::true`, and the term after `then` is a constant.

   2. The term immediately after `if` is a constant and evaluates to `Bool::false`, and the term after `else` is a constant.

5. **Functions**:
   The types of the parameters and the return type of any functions must be constants.
   Functions are always constants, but there's a difference between `const fn`s and normal functions.
   `const fn`s are functions that could be run at compile time (also at runtime).
   The return value of a `const fn` is a constant if and only if all of its arguments are constants in that invocation.
   The operations you can do in the body of a `const fn` are more limited.
   You can only:

   1. Declare a variable with `const`.

   2. Declare a variable with `let`:
      The right-hand-side after the equal sign (`=`) must be a constant.
      Note that declaring a variable with `const` and `let` are also different in a `const fn`.
      They are both constants in a `const fn`.
      But a variable declared with `const` cannot depend on the arguments that could possibly not be known at compiletime, while a variable declared with `let` can.
      The gist of the design is to make changing a non-`const` function to a `const fn` (or conversely) the most seamless.

   3. Normal terms:
      Each subterm in them must be a constant.

   4. Function calls:
      Only `const fn`s can be called.

   `const fn`s are also checked to be *total*, meaning they don't recurse forever.
   A constant that evaluates to a `const fn` is also a `const fn`.
   If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.
   There are also `const fn` lambdas.
   The syntax for it should be obvious.

6. **`data`**:
   In normal `data`, all variants are constants.
   In addition, all variants with parameters in `data` are `const fn`s.
   If you get a part of constant `data` by pattern matching or field accessing, what you get would still be a constant.

7. **`impl`s**:
   Did I mention `impl`s are first-class citizens of Ende?
   They can be returned and passed as arguments too.
   `impl`s are always constants; all `impl`s mentioned before are also `const fn`s that are guaranteed to be total.
   Nevertheless, Auto `impl`s need not be `const fn`s.
   Auto `impl`s that operate only at runtime are denoted `impl(auto, dyn)`.
   They exist because sometimes we can never get the value of the `self` parameter at compile time, e.g. dereferencing a pointer into its underlying type.

## `const data`

A data type marked `const` cannot have any instances at runtime.
How is it useful then?
It must have something to do with the compiler!
And `Mod` is an example.
It's special that it could be `use`d.

What you can do in a `use` statement is what you can do in a `const fn`, so you can write something like:

```
use if isWindows then os::win else os::nix
```

You can write `const fn` accepting or returning `Mod`s, but they are only usable at compile time.
And `mod`s can have parameters in the `const` or the instance mode:

```
mod Magic[magicNumber : I32] where
    \\ Use the magic number.
```

Such `mod`s are `const fn`s the return type of which is `Mod`.

## A `const` Version of `factorial`

I'll define a `const fn factorial` in this subsection.
The implementation of this `factorial` is different from the `U32` version above.
This is not to suggest Ende has overloading; they're just declared in different namespaces.

The problem with `U32` is that it's not defined recursively, so it would be harder for the computer to figure out if the functions using it terminate.
The solution is to use a recursively defined data type: `Nat`

```
@lang("Nat"):
pub(in) data Nat = zero, succ(Nat)
```

`Nat` is special that it has a literal form.
Literal of type `Nat` has a suffix `nat`.

And here's the `const fn factorial`:

```
const fn factorial(m : Nat) -> Nat where match
    (0nat) => 1nat
    (Nat::succ(n)) => m * factorial(n)
```

# More Powerful Generics

Arguments in `const` mode need not be types; they can be constants as well.
In fact, **any** constants can be passed to a function in the `const` mode.
So I can write something like:

```
const fn factorial[m : Nat] -> Nat where match
    [0nat] => 1nat
    [Nat::succ(n)] => m * factorial(n)
```

But then we lose the ability to call `factorial` at runtime.

Actually, types are also first-class citizens of Ende.
That's why there could be associated types.
`data` types *per se* are constants.
In Ende, we can also be generic over type constructors, which are `const fn`s at the type level.
That is, to pass higher-kinded types around.
Normal types have kind `Type`.
However, `Option` is a type constructor and is of kind `[Type] -> Type`, which means it is a function from a type at compile time to another type.
Here's the `Functor` trait.

```
pub(in) data Functor[F : [Type] -> Type] = new {
    "map" -: [From, To](self : F[From], (From) -> To) -> F[To]
}
```

Types of arguments in `const` mode are not always inferred to be `Type`, they can be inferred to be types or kinds of higher-kinded types as well.
For example, if you want to write a function generic over functors, you don't need to explicitly write down the kind of `F`:

```
fn doSomethingAboutFunctors[F, A][(Functor[F])](F[A]) -> Whatever
```

# Do Notation

We've seen `Functor`s, and traditional Haskell tutorial would probably go directly to applicatives and monads, and I'm also going to do that.

The definition of applicative would be:

```
pub(in) data Applicative[F : [Type] -> Type] = new[(Functor[F])] {
    "pure" -: [A](self : A) -> F[A]
    "_<*>_" -: [From, To](self : F[(From) -> To], F[From]) -> F[To]
}
```

And monad would be:

```
pub(in) data Monad[F : [Type] -> Type] = new[(Applicative[F])] {
    "bind" -: [From, To](self : F[From], (From) -> F[To]) -> F[To]
}
```

If you know haskell you must know this `Monad` trait is very very very important.
They can be used to model effects such as logging, error handling, mutable state, non-determinism, etc.
`Monad`s are so important that there's also some special syntax, namely *do-notation*, to make writing monadic code easier.
In Ende, I also want the do-notation, but a more general one than Haskell's, in that it doesn't necessarily have to desugar to the `bind` operation of `Monad`, but **any** method with `@lang("bind"):`.
This is what Idris did because this kind of flexibility is needed when we need something stronger or something that resembles a `Monad` but isn't actually one.

Below are the valid commands in the do-notation in Ende:

1.  normal binding:
    ```
    let plain = something
    ```

2.  pseudo-unwrapping:
    ```
    let unwrapped <- something
    ```

3.  no binding:
    ```
    any term
    ```

The one-to-one correspondence between Ende's syntax and Haskell's (and the desugaring) should be straightforward.

# Spreading

Usually, we don't want to make arguments in normal mode curryable.
But sometimes we do want to supply variable length of arguments.
There is a kind of mechanism in Ende to make genericity over arity doable.
Because I want to make variadic arguments as flexible as possible, it might be a little bit harder to understand.
I need to introduce a new kind of types called **tuple types**.
They are not really the same as tuples in Rust or Haskell.
Tuple types are type-level lists.
For example, `varargs (Unit, Bool)` is a tuple type, and `varargs (I32, F32, U64)` is another tuple type.
What is the kind of all tuple types then?
It's called the **ordered variadic type**, and is written `Ordered[''Type]`.

Now, I'm going to show you how to write a function accepting arbitrarily many arguments.
For clarity, let's consider a rather easy example first.
The function `sum` sums up all the `I32`s in the argument list no matter how many arguments there are.
First, I have to define a helper function for it:

```
pub const fn FreshTuple[_ : Nat](Type) -> Ordered[''Type] where match
    [0nat](_) => varargs ()
    [Nat::succ(n)](T) => varargs (T, ..FreshTuple[n](T))
```

A special operator `..` was used; it's called the **spread operator**, and its purpose is to literally spread the arguments in a tuple type.
A spreaded tuple therefore becomes *naked* without the `varargs()` outside.
For instance, now focus on the `FreshTuple` function above.

- `FreshTuple[0nat](T)` is an empty tuple type.
- `FreshTuple[1nat](T)` = `varargs (T, ..FreshTuple[0nat](T))` = `varargs (T)`.
- `FreshTuple[2nat](T)` = `varargs (T, ..FreshTuple[1nat](T))` = `varargs (T, ..varargs (T))` = `varargs (T, T)`.

So `FreshTuple[n](T)` is `T` repeated for `n` times.

What are the types of the arguments of the `sum` function?
They are `I32` repeated for arbitrarily many times!
Now you can see how `FreshTuple` could be useful.
We can accept a `FreshTuple(I32)` and spread it, leaving how many times it's repeated inferred.
It would be easier to understand it by providing the concrete case than describing it in words:

```
const fn sum(.._ : FreshTuple(I32)) -> I32 where match
    () => 0i32
    (head, ..tail) => head + sum(tail)
```

In contrast to the ordered variadic type, there is `Row[''Type]`, which is a special kind of the **unordered variadic type**, which need not be ordered when deconstructing it.
First, we need another helper function.

```
\\ `Array[n, T]` is the type of arrays length of which is `n` and elements of which are of type `T`.
pub const fn FreshRow[n : Nat][_ : Array[n, Str]](Type) -> Row[''Type] where match
    [0nat, Array::nil](_) => varargs {}
    [Nat::succ(n), Array::cons(head, tail)](T) => varargs { head -: T, ..FreshRow[n, tail](T) }
```

After that we can write functions generic over named arguments.
However, if you want to use the result of calling `FreshRow` more than once, you can't simply call them several times because the inferred arguments need not be the same.
If you want them to be the same without writing down all the arguments in the `const` mode concretely, here's the trick:

```
pub(in) data Replicate[T, n : Nat] = new {
    "Args" -: Tuple[''Type]
}

pub impl replicate[T, n : Nat] -> Replicate[T, n] = Replicate::new {
    "Args" -: match n
        0nat => varargs {}
        Nat::succ(n) => varargs (T, ..replicate[T, n]."Args")
}
```

You define not the helper function but a trait recording the arguments.
Here's how you could use it:

```
const fn sum[n : Nat][(Replicate[I32, n])](.._ : Args) -> I32 where match
    () => 0i32
    (head, ..tail) => head + sum(tail)
```

This trick is especially important with name modes.
The corresponding `Replicate` trait of name modes would be:

```
pub(in) data Replicate[T, n : Nat][_ : Array[n, Str]] = new {
    "Args" -: Row[''Type]
}

pub impl replicate[T, n : Nat, arr : Array[n, Str]] -> Replicate[T, n, arr] = Replicate::new {
    "Args" -: match n
        0nat => varargs {}
        Nat::succ(n) => match arr
            Array::cons(head, tail) =>
                varargs { head -: T, ..replicate[T, n, tail]."Args" }
}
```

Below is a structural `data` type.

```
data Structral[R : Row[''Type]] = structural { ..R }
```

You can add fields to an instance of `Structural`

```
fn addField[T, n : Nat][arr : Array[n, Str]][(Replicate[T, n, arr])](Structural { ..Args })
    -> Structural { “foo” => Int, ..Args } = ...
```

, or remove some of its fields:
```
fn removeField[T, n : Nat][arr : Array[n, Str]][(Replicate[T, n, arr])]
              (Structural { “bar” => Int, ..Args })
    -> Structural { ..Args } = ...
```

The ability to add and remove fields is called *row polymorphism*.
You can also pattern match the fields in name modes to achieve limited reflection.

# Phase Polymorphism

## The Problem

`const` still isn't flexible enough in some situations.
See the following 2 examples for instance:

1.  **The `curry` function**:
    Having variadic arguments, we should be able to define a function that curries the arguments of another function.
    That is, if the function `func` has type `(I32, U32, F32) -> Bool`, `curry(func)` should have type `(I32)(U32)(F32) -> Bool`.

    let me show you how to define such function.

    ```
    \\ Don't ask what `???` is for now.
    \\ Just pretend it's magic; I'll debunk it later.
    \\ The type system still isn't necessarily strong enough to actually write it down.

    \\ We can also pattern match the tuple types instead of the values of the tuple types.)
    const fn CurriedFuncType(Type, .._ : Ordered[''Type]) -> ??? where match
        (Ret) => Ret
        (Ret, Head, ..Tail) => (Head) -> CurriedFuncType(Ret, ..Tail)

    pub const fn curry[Args : Ordered[''Type], Ret](func : (..Args) -> Ret)
        -> CurriedFuncType(Ret, ..Args) where match
        (fn() -> Ret = ret) =>
            ret
        [varargs (Head, ..Tail)](fn(head : Head, ..tail : Tail) -> Ret = ret) =>
            fn(head : Head) -> CurriedFuncType(Ret, Tail) = curry(fn(..tail : Tail) -> Ret = ret)
    ```

    What's problematic about it?
    The problem is:

    > If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.

    If the `func` is a constant but **is not** a `const fn`, because the above `curry` function is a `const fn`, `curry(func)` **is** also a `const fn`.
    Now there's a contradiction, so the compiler should not accept the definition of `curry`.
    But we want it to be a `const fn`!
    Moreover, if the input `func` is a `const fn`, we want the output function to be also a `const fn`.

2.  **Lazy evaluation**:
    If mixfix operators are implemented, `if_then_else` can be implemented as a function, but we now need a way to say that some of the parameters are passed in lazily:

    ```
    pub const fn if_then_else[T](_0_ : Bool, _1_ : Lazy[T], lazy _2_ : Lazy[T]) -> T =
        match _0_ {
            Bool::true => _1_#force()
            Bool::false => _2_#force()
        }
    ```

    However, this is not quite right, either.
    Because if `_0_` evaluates to `true`, the constness of `_2_` doesn't matter to the constness of the whole function.

## The Solution

The solution is to give the users finer control of the `const` system.
Instead of writing `const var` or plain `var`, we can write `phase(...) var` to specify its phase further.

(TBD)

# Memory Management

Until now, the language is still fairly compatible with system programming.
Functions are ... well, functions; non-capturing lambdas are function pointers.
`data` are enums in C each with a tag indicating its variant to make them type safe but records specifically can be implemented as structs in C: I didn't say recursive types work.
Surely some form of recursive type must be implemented in order to make Ende really useful, but I think it should be done with explicit pointer; I will discuss on it later.
`mod`s are modules.
Generics are monomorphized at compile time.
`impl`s have nothing to do with runtime.

## Heap Allocation

Perhaps the most important topic in memory management is heap allocation.
In order to make Ende a system programming language, users of the language must be able to manipulate raw pointers.
But in addition to it, there are supposed to be a higher-level interface for normal programmers since manipulating raw pointers are highly unsafe thus error-prone.
I've considered 3 different approaches in the past:

1. **The Rust Approach**:
   the closest to the bare metal and theoretically the most efficient but requires lots and lots of lang items.

2. **The Swift Approach**:
   Doesn't require a garbage collector (GC), but instead implicitly using reference counting everywhere.
   Cycles between reference counted (RC) pointers could cause memory leaks and users have to be very careful about it.
   Dereferencing an unowned pointer could fail.

3. **The Java Approach**:
   GC everywhere.
   The safest solution the easiest to use, but one sometimes needs to *stop the world*.
   Bad for application needing low latency.

I prefered the approach of using RC *explicitly* at the beginning, but found out that doesn't work quite well.
One would end up need to write out RC almost everywhere.
Although that would arguably make Ende *Rust++*, I think explicit is better than implicit and by far Rust's solution is the best one also because implementing ownership doesn't prevent Ende from implementing explicit RC/GC.
I'll try to descibe the compiler work needed in the rest of this section.

(TBD)

# GADTs

Normal `data` types are called ADTs in Haskell.
GADTs are the **G**eneralized version of ADTs.
GADTs let you define inductive families, that is, to be specific on the return types of the variants.
We can define a GADT by writing `where` instead of an equal sign (`=`) after the parameters of the `data`.

```
pub(in) data Array[_ : Nat, T] where
    nil : Array[0nat, T]
    cons : [n : Nat](T, Array[n, T]) -> Array[Nat::succ(n), T]
```

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

# Universes

What's the type of `Type` and type constructors?
In Ende, `Type` is actually not a single type, but a series of types.
You can think that `Type` has a hidden parameter that is a natural number.
The `Type` that all of us are familiar about is `Type<0>`, but the type of `Type<0>` is `Type<1>`, the type of which is `Type<2>`, and going on and on.
What about the types of function types?
The answer is:

```
A : Type<m>    B : Type<n>
--------------------------
 A -> B : Type<max(m, n)>
```

Imagine if we want to accept a potentially infinite list of arguments types of which are `Int, Type<0>, Int, Type<0>, Int, Type<0> ...`.
How do we write a helper function to generate the tuple type?
The variadic type of the return type of the function cannot be `Ordered[''Type<0>]` because the type of `Type<0>` cannot be `Type<0>`.
The answer is to make universes cumulative:

```
      m < n
------------------
Type<m> <: Type<n>
```

and

```
T : Type<m>    Type<m> <: Type<n>
---------------------------------
           T : Type<n>
```

A term of a variadic type is a list of types the types of all of which are the same:

```
   T1 : U    T2 : U    T3 : U    ...
---------------------------------------
varargs (T1, T2, T3) ... : Ordered[''U]
```

Now that `Int : Type<1>`, so the type of `varargs (Int, Type<0>, Int, Type<0>, Int, Type<0> ...)` is `Ordered[''Type<1>]`.

## Hierarchies

We can see that

```
           A <: B
----------------------------
Ordered[''A] <: Ordered[''B]
```

i.e. variadic types are covariant.
Now we have at least 2 different hierarchies of universes, one is

```
Type<0> : Type<1> : Type<2> : Type<3> ...
```

, another one is

```
Ordered[''Type<0>] : Ordered[''Type<1>] : Ordered[''Type<2>] : Ordered[''Type<3>] ...
```

which could also be written

```
Ordered[''Type]<0> : Ordered[''Type]<1> : Ordered[''Type]<2> : Ordered[''Type]<3> ...
```

In reality there are infinite hierarchies because `Ordered[''Ordered[''Type]]` and so on are also hierarchies.
It sounds reasonable to say that all the universes I mentioned are so called *small* universes the type of which is `Type<ω>`.
`Type<ω>` is special in that elements of it can inherit another one.
`Ordered` would become a data type from `Type<ω>` to `Type<ω>` then.

```
data Ordered[_ : Type<ω>] : Type<ω> = ...
```

Now types of function types could be:

```
A : U1<m>    ''U1<m> : Type<ω>    B : U2<n>    ''U2<n> : Type<ω>
----------------------------------------------------------------
                     A -> B : U2<max(m, n)>
```

If either the argument type or the return type is `Type<ω>`, perhaps we need `Type<ω+1>`, and we can go up until `Type<ω+ω>`.
I don't know if what's beyond would be useful.
