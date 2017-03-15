# Spreading

Usually, we don't want to make arguments in normal mode curryable.  
But sometimes we do want to supply variable length of arguments.  
There is a kind of mechanism in Ende to make genericity over arity doable.  
Because I want to make variadic arguments as flexible as possible, it might be a little bit harder to understand.  
I need to introduce a new kind of types called **tuple types**.  
They are not really the same as tuples in Rust or Haskell.  
Tuple types are type-level lists.  
For example, `varargs (Unit; Bool)` is a tuple type, and `varargs (I32; F32; U64)` is another tuple type.  
What is the kind of all tuple types then?  
It's called the **ordered variadic type**, and is written `Ordered[''Type]`.

Now, I'm going to show you how to write a function accepting arbitrarily many arguments.  
For clarity, let's consider a rather easy example first.  
The function `sum` sums up all the `I32`s in the argument list no matter how many arguments there are.  
First, I have to define a helper function for it:

```
#pub:
const fn FreshTuple[_ : Nat](Type) -> Ordered[''Type] where match
    [0nat](_) => varargs ()
    [Nat::succ(n)](T) => varargs (T; ..FreshTuple[n](T))
```

A special operator `..` was used; it's called the **spread operator**, and its purpose is to literally spread the arguments in a tuple type.  
A spreaded tuple therefore becomes _naked_ without the `varargs()` outside.  
For instance, now focus on the `FreshTuple` function above.

* `FreshTuple[0nat](T)` is an empty tuple type.
* `FreshTuple[1nat](T)` = `varargs (T; ..FreshTuple[0nat](T))` = `varargs (T)`.
* `FreshTuple[2nat](T)` = `varargs (T; ..FreshTuple[1nat](T))` = `varargs (T; ..varargs (T))` = `varargs (T; T)`.

So `FreshTuple[n](T)` is `T` repeated for `n` times.

What are the types of the arguments of the `sum` function?  
They are `I32` repeated for arbitrarily many times!  
Now you can see how `FreshTuple` could be useful.  
We can accept a `FreshTuple(I32)` and spread it, leaving how many times it's repeated inferred.  
It would be easier to understand it by providing the concrete case than describing it in words:

```
const fn sum(.._ : FreshTuple(I32)) -> I32 where match
    () => 0i32
    (head; ..tail) => head + sum(tail)
```

In contrast to the ordered variadic type, there is `Row[''Type]`, which is a special kind of the **unordered variadic type**, which need not be ordered when deconstructing it.  
First, we need another helper function.

    \\ `Array[n; T]` is the type of arrays length of which is `n` and elements of which are of type `T`.
    #pub:
    const fn FreshRow[n : Nat][_ : Array[n; Str]](Type) -> Row[''Type] where match
        [0nat; Array::nil](_) => varargs {}
        [Nat::succ(n); Array::cons(head; tail)](T) => varargs { head -: T; ..FreshRow[n, tail](T) }

After that we can write functions generic over named arguments.  
However, if you want to use the result of calling `FreshRow` more than once, you can't simply call them several times because the inferred arguments need not be the same.  
If you want them to be the same without writing down all the arguments in the `const` mode concretely, here's the trick:

```
@allPub:
data Replicate[T; n : Nat] = new {
    "Args" -: Tuple[''Type]
}

#pub:
impl replicate[T; n : Nat] -> Replicate[T; n] = Replicate::new {
    "Args" -: match n
        0nat => varargs {}
        Nat::succ(n) => varargs (T; ..replicate[T, n]."Args")
}
```

You define not the helper function but a trait recording the arguments.  
Here's how you could use it:

```
const fn sum[(Replicate[I32])](.._ : Args) -> I32 where match
    () => 0i32
    (head; ..tail) => head + sum(tail)
```

This trick is especially important with name modes.  
The corresponding `Replicate` trait of name modes would be:

```
@allPub:
data Replicate[T; n : Nat][_ : Array[n, Str]] = new {
    "Args" -: Row[''Type]
}

#pub:
impl replicate[T; n : Nat; arr : Array[n, Str]] -> Replicate[T; n; arr] = Replicate::new {
    "Args" -: match n
        0nat => varargs {}
        Nat::succ(n) => match arr
            Array::cons(head; tail) =>
                varargs { head -: T; ..replicate[T; n; tail]."Args" }
}
```

Below is a structural `data` type.

```
data Structral[R : Row[''Type]] = structural { ..R }
```

You can add a field to `Structural`:

```
fn addField[T][(Replicate[T])](Structural { ..Args })
    -> Structural { “foo” => Int; ..Args } = ...
```

Or remove a field of it:

```
fn removeField[T][(Replicate[T])](Structural { “bar” => Int; ..Args })
    -> Structural { ..Args } = ...
```

The ability to add and remove fields is called _row polymorphism_.  
You can also pattern match the fields in name modes to achieve limited reflection.

