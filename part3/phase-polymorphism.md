# Phase Polymorphism

## The Problem

`const` still isn't flexible enough in some situations.  
See the following 2 examples for instance:

1. **The **`curry`** function**:  
   Having variadic arguments, we should be able to define a function that curries the arguments of another function.  
   That is, if the function `func` has type `(I32; U32; F32) -> Bool`, `curry(func)` should have type `(I32)(U32)(F32) -> Bool`.

   let me show you how to define such function.

       \\ Don't ask what `???` is for now.
       \\ Just pretend it's magic; I'll debunk it later.
       \\ The type system still isn't necessarily strong enough to actually write it down.

       \\ We can also pattern match the tuple types instead of the values of the tuple types.)
       const fn CurriedFuncType(Type; .._ : Ordered[''Type]) -> ??? where match
           (Ret) => Ret
           (Ret; Head, ..Tail) => (Head) -> CurriedFuncType(Ret; ..Tail)

       #pub:
       const fn curry[Args : Ordered[''Type]; Ret](func : (..Args) -> Ret)
           -> CurriedFuncType(Ret; ..Args) where match
           (fn() -> Ret = ret) =>
               ret
           [varargs (Head; ..Tail)](fn(head : Head; ..tail : Tail) -> Ret = ret) =>
               fn(head : Head) -> CurriedFuncType(Ret; Tail) = curry(fn(..tail : Tail) -> Ret = ret)

   What's problematic about it?  
   The problem is:

   > If a `const fn` has multiple argument lists in normal mode, supplying one or more but not all lists of arguments outputs another `const fn`.

   If the `func` is a constant but **is not** a `const fn`, because the above `curry` function is a `const fn`, `curry(func)` **is** also a `const fn`.  
   Now there's a contradiction, so the compiler should not accept the definition of `curry`.  
   But we want it to be a `const fn`!  
   Moreover, if the input `func` is a `const fn`, we want the output function to be also a `const fn`.

2. **Lazy evaluation**:  
   If mixfix operators are implemented, `if_then_else` can be implemented as a function, but we now need a way to say that some of the parameters are passed in lazily:

   ```
   #pub:
   const fn if_then_else[T](_0_ : Bool, _1_ : Lazy[T], lazy _2_ : Lazy[T]) -> T =
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

\(TBD\)

