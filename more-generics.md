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
