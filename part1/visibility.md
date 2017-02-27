# Visibility

All items, `data` variants and fields of records are by default private.
Items and variants being private means we can't use them outside the module.
Fields being private means we can't access it through the dot (`.`) notation or pattern matching; we can't construct an instance of that record outside either so we must rely on the `pub fn` it provides to initiate the record.

You can write `pub` in front of the item to override that behavior, examples include:

```
pub fn zero() -> I32 = 0i32
pub data One = one
```

If you want to make all variants and fields of a `data` type `pub`, you can write `pub(in)` in front of the `data`.
You usually want this for `data` that aren't records.

```
pub(in) data Shape =
    circle
    triangle
    square
```
