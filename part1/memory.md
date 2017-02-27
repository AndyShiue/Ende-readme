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
