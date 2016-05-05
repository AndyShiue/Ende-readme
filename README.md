# Ende

I just want to share what I've come up with about the design of a new programming language, and here it is.
This language would have a strong type system, and its syntax would be whitespace-insensitive because I prefer it.

1. At the very beginning, let me introduce the terms of the language.
   Terms can be seen as trees that build up value.
   Terms include:

   1. Literals
      There are several built-in types in Ende.
      They are numbers.
      Boolean values still won't be introduced yet but will be defined as a data type.
      Types like `Char` and `String` isn't presented because numbers suffice for demonstration purposes but should be added if there is an actual implementation of the language.

      1. Integers
         There would be several types of integers of different size, being either signed or not.
         A literal of type `I32` would be written as `42i32`.
         `42` is it's actual value, and `i32` means it's a 32-bit unsigned integer.
         Similarly, `666u64` would be a 64-bit signed integer.
         An integer without any prefix would have type `Nat`.
         Instead of being built-in, `Nat` is just a normal recursively defined data type.
         I will also discuss about that type later.
      
      2. Floats
         Literals of floats would be similar to ones for integers, e.g. `2.71828f32`.

   2. Applications of binary operators
      
