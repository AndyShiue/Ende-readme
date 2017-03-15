# User-Defined Data Types

Data types are items and can be defined in function bodies.  
They can be defined with the keyword `data`.  
They can be seen as `enum`s in Rust and are more powerful than that of C/Java.  
I'll first consider its easiest usage, though.  
Let's show how to define a C/Java-like `enum` in Ende:

```
data Unit = unit
data Bool =
    true
    false
data Season =
    spring
    summer
    autumn
    winter
```

In Ende, `data` can have **variants**.  
Variants are the possible values of the defined type.  
Variants are namespaced; in order to access the variant `spring`, you need to write

```
let season = Season::spring
```

instead of

```
\\ let season = spring \\ Doesn't compile.
```

`data` variants could be matched through pattern matching:

```
fn isSummer(season : Season) -> Bool =
    match season of
        Season::summer => Bool::true
        _ => Bool::false
```

`match`es are always irrefutable in Ende.  
By the way, there is an alternative `fn` syntax to do top-level pattern matching.  
We write `match` in the last clause in the function body in this case.

```
fn isSummer(Season) -> Bool where match
    (Season::summer) => Bool::true
    (_) => Bool::false
```

Here, we directly match the arguments of the `fn`; now we don't need to pick a name for the argument of type `Season`; we can write `fn isSummer(Season)` in lieu of `fn isSummer(_ : Season)` in this case.  
I'm going to provide another example for clarity:

```
fn and(Bool; Bool) -> Bool where match
    (Bool::true; b) => b
    (_; _) => Bool::false
```

Variants may also take parameters.  
The `OptionI32` below is a type that could possibly carry an `I32`.

```
data OptionI32 =
    some(I32)
    none
```

Parameters of variants can also be pattern matched:

```
fn unwrapOr42(OptionI32) -> I32 where match
    (OptionI32::some(i)) => i
    (_) => 42i32
```

Parameters of variants can be named, that is, to be indexed by a string:

```
data Point = new {
    "x" -: I32
    "y" -: I32
}
```

A `data` with a single variant is called a record.  
The compiler treats records different from normal `data` in subtle ways.  
Fields \(named arguments\) of an instance of a record can either be accessed by its name after a dot \(`.`\) or by pattern matching.

```
fn getX1(p : Point) -> I32 = p."x"
fn getX2(Point) -> I32 where match
    (Point::new { "x" -: result
                  "y" -: _ }) => result
```

The syntax of pattern matching a record or constructing an instance of it is the same as declaring one:

```
let center = Point::new {
    "x" -: 0i32
    "y" -: 0i32
}
```



