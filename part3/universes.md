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
varargs (T1; T2; T3 ...) : Ordered[''U]
```

Now that `Int : Type<1>`, so the type of `varargs (Int; Type<0>; Int; Type<0>; Int; Type<0> ...)` can be `Ordered[''Type<1>]`.

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
It sounds reasonable to say that all the universes I mentioned are so called _small_ universes the type of which is `Universe<0>`.  
`Universe` is special in that elements of it can inherit another one.  
`Ordered` would become a constructor from `Universe` to `Universe` then.

```
#lang("Ordered"): @allPub:
universe Ordered[''U : Universe] =
    nil
    cons(U; Ordered[''U])
```

Now types of function types could be:

```
A : U1<m>    ''U1<m> : Universe    B : U2<n>    ''U2<n> : Universe
------------------------------------------------------------------
                      A -> B : U2<max(m, n)>
```

If either the argument type or the return type is `Universe<0>`, perhaps we need `Universe<1>`, and we can go up until infinity.  
I don't know if what's beyond would be useful.

## `Unordered`

Row types are defined in terms of the unordered universe constructor, the definition of which is the same as `Ordered` but the compiler handles spreading of them differently:

```
#lang("Unordered"): @allPub:
universe Unordered[''U : Universe] =
    nil
    cons(U; Unordered[''U])

@allPub:
universe KeyValue[Label; ''U : Universe] =
    _-:_(_0_ : Label; _1_ : U)

#pub:
fn Row[Code : Universe] -> Unordered[''KeyValue[Str; Code]] =
    Unordered[''KeyValue[Str; Code]]
```

Terms the type of which are `Type<0>` are types.  
What is the rule to determine if a term belongs to a custom universe?  
Below, I clarify the rule for it.  
To be more general, I introduce _generalized types_, the type of which are any universes.  
Inductively, a term is a generalized type iff

1. It's a type, or
2. It's a variant of a universe, and each of its fields are either a term the type of the type of which is \`Type\` or a generalized type.



