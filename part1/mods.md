# `mod`s

`mod`s are containers of items, and `mod`s themselves are also items, so `mod`s can be nested.  
You can define `mod`s in function bodies.  
We use the keyword `mod` to declare a module.

```
#pub:
mod module where
    fn getUnit() -> Unit = Unit::unit
    @allPub:
    data Three =
        one
        two
        three
    #pub:
    mod inner where
        #pub:
        data Circle = new { "radius" -: U32 }
```

items in a `mod` could only mention previous items including itself, unless wrapped in `mutual` blocks, in which any one of them can mention any one of the others.

```
#pub:
mod another where
    mutual \\ This is required.
        data A =
            itself
            theOther(B) \\ Here it uses a type that is declared later.
        data B =
            itself
            theOther(A)
```

A `mod` can be declared without `where`.

```
mod somewhereElse
```

If the compiler sees such `mod`, the compiler will look for `./somewhereElse.ende` and `./somewhereElse/mod.ende` to read its content.  
If neither or both are presented, the compiler emits an error.

There are two ways to access an item in a `mod`.  
One is to write its fully qualified name, e.g. `module::inner::Circle`.  
The other way is to `use` the items.

```
use module::inner::Circle
use module::inner::Circle::new

fn getCircle() -> Circle = new { "radius" -: 1u32 }
```

An underscore \(`_`\) can be used as a wildcard to import everything in a module or in a `data`, so instead of writing 3 `use`s to import `one`, `two` and `three`, you can write

```
use module::Three::_
```

`mod`s are also first-class value of type `Mod`:

```
#lang("Mod"): #pub:
const data Mod
```

Here, `const` means the instances of `Mod` can only exist at compile time.  
I'll show how to manipulate `data` types at compile time later.

