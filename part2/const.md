# `const`

You may found that I wrote `const`_ mode_ instead of _const mode_ throughout the article.  
This is intentional.  
`const` is an important keyword in Ende, and is highly related to the `const` mode.  
The basic idea is that a `const`_ something_ is a _something_ that can be done at compile time.  
So a `const` value \(a constant\) is a value known at compile time, and a `const fn` is a function that could be run at compile time.

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
   Specificly, The value of the entire `while` loop is not a constant \(actually because it's not a `const fn`\).  
   `if`is special that the value of an `if` construct is a constant if and only if either of the conditions below is met.

   1. The term immediately after `if` is a constant and evaluates to `Bool::true`, and the term after `then` is a constant.

   2. The term immediately after `if` is a constant and evaluates to `Bool::false`, and the term after `else` is a constant.

5. **Functions**:  
   The types of the parameters and the return type of any functions must be constants.  
   Functions are always constants, but there's a difference between `const fn`s and normal functions.  
   `const fn`s are functions that could be run at compile time \(also at runtime\).  
   The return value of a `const fn` is a constant if and only if all of its arguments are constants in that invocation.  
   The operations you can do in the body of a `const fn` are more limited.  
   You can only:

   1. Declare a variable with `const`.

   2. Declare a variable with `let`:  
      The right-hand-side after the equal sign \(`=`\) must be a constant.  
      Note that declaring a variable with `const` and `let` are also different in a `const fn`.  
      They are both constants in a `const fn`.  
      But a variable declared with `const` cannot depend on the arguments that could possibly not be known at compiletime, while a variable declared with `let` can.  
      The gist of the design is to make changing a non-`const` function to a `const fn` \(or conversely\) the most seamless.

   3. Normal terms:  
      Each subterm in them must be a constant.

   4. Function calls:  
      Only `const fn`s can be called.

   `const fn`s are also checked to be _total_, meaning they don't recurse forever.  
   A constant that evaluates to a `const fn` is also a `const fn`.  
   If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.  
   There are also `const fn` lambdas.  
   The syntax for it should be obvious.

6. `data`:  
   In normal `data`, all variants are constants.  
   In addition, all variants with parameters in `data` are `const fn`s.  
   If you get a part of constant `data` by pattern matching or field accessing, what you get would still be a constant.

7. `impl`**s**:  
   Did I mention `impl`s are first-class citizens of Ende?  
   They can be returned and passed as arguments too.  
   `impl`s are always constants; all `impl`s mentioned before are also `const fn`s that are guaranteed to be total.  
   Nevertheless, Auto `impl`s need not be `const fn`s.  
   Auto `impl`s that operate only at runtime are moreover annotated `#dyn:`.  
   They exist because sometimes we can never get the value of the parameter at compile time, e.g. dereferencing a pointer into its underlying type.

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
#lang("Nat"): @allPub:
data Nat = zero; succ(Nat)
```

`Nat` is special that it has a literal form.  
Literal of type `Nat` has a suffix `nat`.

And here's the `const fn factorial`:

```
const fn factorial(m : Nat) -> Nat where match
    (0nat) => 1nat
    (Nat::succ(n)) => m * factorial(n)
```



