# More General records

Records in Ende can not only act as Rust `struct`s but also Rust `trait`s or Java `interface`s or Haskell `class`es.  
I'll call them traits in the rest of the article if they are used like a Rust `trait`.  
Here, I'm going to write a `Monoid` trait.  
Of course, it's just another record.

```
@allPub:
data Monoid[T] = new {
    "unit" -: T
    "append" -: (self : T; T) -> T
}
```

To implement a trait, I introduce another keyword `impl`.  
All `impl`s are items and can be defined locally in a function body.  
Unlike the `impl`s in Rust or `instance`s in Haskell, `impl`s in Ende are always named.

## Simple `impl`s

The `i32Monoid` below is a simple `impl`:

```
#pub:
impl i32Monoid : Monoid[I32] = Monoid::new {
    "unit" -: 0i32
    "append" -: fn(self : I32; another : I32) -> I32 = self + another
}
```

If `i32Monoid` is in scope, we can call the `append` method on `I32`:

    \\ They are equivalent because the first argument of the field of `"append"` is `self`:

    let sum1 = i32Monoid."append"(1i32, 2i32)
    let sum2 = 1i32#append(2i32)

In order to write a function that is generic over types implementing a trait, the third mode is introduced.  
It's called the **instance mode**, and is delimited by `[()]`.  
Arguments in the instance mode can also be inferred, however not by looking at other arguments, but by searching for appropriate `impl`s.  
Here is a function generic over types implementing `Monoid`; it sums up all the values in a `List` using `Monoid`'s `append` method:

```
fn concat[T][(Monoid[T])](List[T]) -> T where match
    (List::nil) => unit
    (List::cons(head; tail)) => head#append(concat(tail))
```

You can see that when you put an argument inside the instance mode, the fields of it are automatically brought into scope without the quotation marks \(except those using weird characters that would be invalid identifiers in the syntax\); it's just a syntax sugar.

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
@allPub:
data Group[T] = new {
    "unit" -: T
    "append" -: (self : T; T) -> T
    "inverse" -: (self : T) -> T
}
```

We know all `Group`s are `Monoid`s, so all instances of `Group[T]` should be able to be automatically converted to instances of `Monoid[T]`.  
How do we automatically convert stuff?  
The answer is obviously, again, using `impl`s.

```
#pub:
impl groupToMonoid[T][(Group[T])] -> Monoid[T] = Monoid::new {
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
    "append" -: (self : T; T) -> T
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

    @allPub:
    data Group[T] = new[(Monoid[T])] {
        "inverse" -: (self : T) -> T
    }
    @allPub:
    data Abelian[T] = new[(Group[T])]

    \\ The `impl`s also have to be changed ...

As you can see, it's simply the instance mode after the constructor.

Searching of supertrait arguments isn't guaranteed to terminate because one could recurse on them, though.

## Associated Values

Fields of a record can also be dependent on the others.  
They are different from normal _input parameters_ in that they don't determine the `impl` chosen but the `impl`s determine them.  
They are _output parameters_.  
For example, imagine if I want to define a trait `Add` for all types that implement the `_+_` operator.  
To achieve maximal flexibility, I want to make types of the the left-hand-side, right-hand-side, and the returned terms possibly different.  
It means it has to be generic over these 3 types.  
So maybe the trait could be like:

```
@allPub:
data Add[L; R; Output] = add {
    "_+_" -: (self : L; R) -> Output
}
```

However, the trait is problematic because we can provide both instances of `Add[L; R; A]` and `Add[L; R; B]`; if I write `id(l : L) + id(r : R)`, the compiler wouldn't be able to know if the type of the result would be `A` or `B`.  
This suggests that the `Output` type should not be an input parameter but rather an output one determined uniquely by `L` and `R`.  
The correct trait should be:

```
@allPub:
data Add[L, R] = add[Output] {
    "_+_" -: (self : L; R) -> Output
}
```

The `impl` ought to specify the `Output` type.  
If you want to access the output type specified by an instance of a trait, simply do a pattern matching.  
Or you can put them inside the `const` mode to make calling it with the dot syntax possible.

## Auto `impl`s

Automatic `impl`s are `impl`s with a `self` parameter in normal mode.  
The annotation `#auto:` indicates they are `impl`s that will be automatically inserted.  
For example, if one wants to overload string literals, they could provide an auto `impl` from `Str` to whatever type they want.  
For now, let's assume that type is called `StrLike`.  
The Ende source code would be something like:

```
#pub: #auto:
impl strLike(self : Str) -> StrLike = ...
```

Auto `impl`s could be inserted at any node in the AST of a term if the expected type doesn't match the actual type, so if a term `str` occurs in the source code, it could possibly be transformed to `str#strLike()` anywhere.  
Auto `impl`s wouldn't be inserted more than once at a particular node, however, which means the following code doesn't type check.

    #auto:
    firstToSecond(self : First) -> Second = ...
    #auto:
    secondToThird(self : Second) -> Third = ...

    fn manipulateThird(third : Third) -> Third = third

    let second : Second = ...
    manipulateThird(first) \\ It works.

    let first : First = ...
    \\ manipulateThird(first) \\ `first` cannot be transformed to value of type `Third`.

\(TBD: auto `impl` in an instance argument\)

## Visibility of `impl`s

The visibility of `impl`s are the same as that of `fn`s.  
Because people usually want to import all `impl`s in a `mod`, I purpose a little syntax to do that:

```
use some_module::impl
```

Syntax for importing every `fn`/`data`/`mod` in a `mod` could be implemented likewise.

