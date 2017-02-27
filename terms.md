# Terms and Statements

At the very beginning, let me introduce the terms of the language.
Terms can be seen as trees that build up values.
Terms include:

1. **Literal**s:
   There are several built-in types in Ende.
   They include numbers and strings.
   Boolean values still won't be introduced yet but will be defined as a data type.

   1. **Integers**:
      There would be several types of integers of different size, being either signed or not.
      A literal of type `I32` would be written as `42i32`.
      `42` is its actual value, and `i32` means it's a 32-bit signed integer.
      Similarly, `666u64` would be a 64-bit unsigned integer.

   2. **Floats**:
      Literals of floats would be similar to ones of integers, e.g. `2.71828f32`.

   3. **Strings**:
      String literals are written in a pair of double quotation marks (`""`), in which you can escape some special characters as you do in other normal languages.

2. Applications of operators:
   When one talks about operators, we usually think of infix operators.
   But in actual implementations, operators of any fixity could be introduced.

   Examples would be `1u32 + 1u32`, `42u32 * 666u32`, `1u32 == 2u32`, etc.
   Fixity of operators should follow the common sense.
   If you need parentheses to group expressions, write `id()`, e.g. `id(2u32 + 2u32) * 2u32`.
   As you will see, this is actually not special syntax, but just a call to the identity function.

3. Variables, which are going to be discussed immediately.

Throughout the article, more and more other kinds of terms will be introduced.

Ende intends to be usable as an imperative language, so there are statements including control structures.
Statements include:

1. **`let` binding**:
   A `let` binding binds its right-hand-side value to a variable.
   There are 2 flavors of `let` bindings, which can be seen as a mutable one and an immutable one respectively currently.
   `let` is by default immutable.
   For example, `let meaningOfLife = 42i32` binds `42i32` to a variable called `meaningOfLife`.
   As you can see, variables are written in camel case in Ende.
   Mutable variables can be declared in a different syntax.
   So, `let count <- mut(0i32)` binds `0i32` to a variable named `count`.

2. **mutation**:
   The value of a mutable variable can be mutated.
   `count := 1i32` changes the value of `count` to `1i32`.
   Mutating an immutable variable generates a compile error.

3. **`while` loop**:
   Normal `while` loop similar to other imperative languages.

   ```
   while count == 0i32 then do
       forever()
   ```

   The value a `while` loop returns carries no data.

Now, introduce `if`.
Unlike `while`, `if` can return something that isn't trivial.
So we can write code like:

```
let number = if count == 0i32 then 42u32 else 666u32
```

Of course, `if` can also be used in an old-style, C-like fashion.

```
let number <- mut(0u32)
if count == 0i32
then number := 42u32
else number := 666u32
```
