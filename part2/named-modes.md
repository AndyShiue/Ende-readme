# Named Modes

Compare this record in Ende

```
data Example = new {
    "example" -: Unit
}
```

and this `struct` in Rust:

```rust
struct Example {
    example: ()
}
```

Why do we need to write `new` after the equal sign \(`=`\)?  
It's not a keyword!  
The intention is to disambiguate between the type `Example` and the constructor of it.  
In Rust, which doesn't have dependent types, `Example` can mean both, and a usage of `Example` can always be resolved, but not in a language in which types are first-class.

Variants in `data` have types, what is the type of `new` then?  
In order to assign a type to it, we need new modes in which parameters are named.  
Let's call them **named mode**s.  
The named normal mode is similar to the normal mode, except that the parameters are named and unordered; the named `const` mode corresponds to the `const` mode.  
The above `new` is now of the type `{"example" -: Unit} -> Example`.  
To be more general, now we can also have struct variants and arbitrary functions accepting named parameters.

