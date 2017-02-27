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
