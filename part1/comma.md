# Semicolon

Instead of resorting to indentation, we can use semicolons to seperate stuff.  
Therefore, The above `Three` can also be written as:

```
@allPub:
data Three = one; two; three
```

The rule is that whenever we see a semicolon either with an indentation after a new line after it or not, we append a clause to the innermost possible place in the syntax tree.  
If there isn't some indentation after a new line in comparison to the previous line, it's parsed as a sibling clause of the previous one.  
Below are some examples:

1. This definition of `Three` is the same as the above ones:

   ```
   @allPub:
   data Three = one;
       two;
       three;
   data SomeOtherData = someOtherData
   ```

2. This would generate a parsing error:

   ```
   @allPub:
   data Three =
   a;
   b;
   c
   ```

3. This would also generate a parsing error when the parser encounters `b`:

   ```
   @allPub:
   data Three = a;
   b;
   c
   ```

You can write 2 semicolons in a row to go out of one layer of the syntax, so `data First = first;; data Second = second` in the same line is valid syntax.  
More consecutive semicolons can be used to go out of several layers of the syntax in a similar fashion.

Also, there's a way to escape all indentation after the next line.  
You can write `\` at the end of the line to do it as in C.

