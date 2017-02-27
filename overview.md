# Overview

This article was initially designed to be implemented from the beginning to the end, but it has already been too long.
As some people suggested, I provide a brief overview here.

The core feature of Ende is **modes**.
Modes are ways to pass arguments to functions.
In current programming languages, the boundary between phases, which mean compile time and runtime, are either not mixed well or too blurry.
The former includes more conservative languages like Java or Go.
The latter are basically those dependently-typed languages.
The lack of compile-time constructs makes higher abstraction at compile time impossible, but dependent types make the phase of which the term is evaluated unpredictable.
The idea of modes is basically genericity over phases.
In complement to modes, you can specify stuff that works only at runtime or either at runtime or compile time using the `const` system of Ende.

Being concrete, below are some examples that would make use of modes and the `const` system:

1. If you want iterate between some fixed numbers, and the operation you want to do is known at compile time (which is the majority of cases), the compiler would simply instantiate operations again and again iteratively between the fixed numbers you chose.
   Moreover, the same interface can also be used at runtime.
   Therefore, if you want to range over some indices only known at runtime, the change of the code would be minimal or even none.

2. Have you heard of writing a multiplication table with template meta-programming in C++?
   Because of the backwards compatibility wart with C, you have to use weird template-level syntax to write those functional code.
   In Ende, they can be written in a clean way and as I've said, the code could be also used at runtime.

3. Regular expressions, web template languages, SQL commands, etc. can be precompiled on demand in a type-safe manner, so you don't have to waste time at runtime.
   And as you would guess, you can also do them at runtime with the same API.

The presence of modes makes almost everything in Ende first-class, which means they can be passed as arguments or returned.
They include:

1. all functions
2. all structs
3. all data constructors
4. all traits
5. all implementations of traits
6. all modules
7. all types

As you will see, Ende achieves the unification of different concepts with carefully designed syntax and semantics.
Someone might find the similarities between your favorite programming language and Ende.
I actually took lots of ideas from Rust, Scala, Agda, Idris, etc.
Feel free to skip some of the initial paragraphs if you think they're obvious enough and reread them if later you find that you can't parse some of the syntax.
