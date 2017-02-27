# Lang Items

In Rust and also in Ende, there's a concept of *lang items*.
Lang items are items that are treated specially, having something to do with the compiler.
I won't introduce a keyword for lang items but instead use *annotations* to mark them.
Annotations are written before the annotated item, start with an at sign (`@`) and is followed by a colon (`:`).

```
@lang("whatever"):
fn whatever() -> Unit = unit
```
