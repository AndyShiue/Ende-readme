# Do Notation

We've seen `Functor`s, and traditional Haskell tutorial would probably go directly to applicatives and monads, and I'm also going to do that.

The definition of applicative would be:

```
@allPub:
data Applicative[F : [Type] -> Type] = new[(Functor[F])] {
    "pure" -: [A](self : A) -> F[A]
    "_<*>_" -: [From; To](_0_ self : F[(From) -> To]; _1_ : F[From]) -> F[To]
}
```

And monad would be:

```
@allPub:
data Monad[F : [Type] -> Type] = new[(Applicative[F])] {
    "bind" -: [From; To](self : F[From]; (From) -> F[To]) -> F[To]
}
```

If you know haskell you must know this `Monad` trait is very very very important.  
They can be used to model effects such as logging, error handling, mutable state, non-determinism, etc.  
`Monad`s are so important that there's also some special syntax, namely _do-notation_, to make writing monadic code easier.  
In Ende, I also want the do-notation, but a more general one than Haskell's, in that it doesn't necessarily have to desugar to the `bind` operation of `Monad`, but **any** method with `#lang("bind"):`.  
This is what Idris did because this kind of flexibility is needed when we need something stronger or something that resembles a `Monad` but isn't actually one.

Below are the valid commands in the do-notation in Ende:

1. normal binding:

   ```
   let plain = something
   ```

2. pseudo-unwrapping:

   ```
   let unwrapped <- something
   ```

3. no binding:

   ```
   any term
   ```

The one-to-one correspondence between Ende's syntax and Haskell's \(and the desugaring\) should be straightforward.

