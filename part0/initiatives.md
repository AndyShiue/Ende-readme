# Initiatives

The first (somehow) functional programming language I learned was Scala.
In Scala, functions are classes, and functions with different arities are represented with different classes.
This kind of ad-hoc solution not only violates the DRY (Don't Repeat Yourself) principle, but also is [limited](http://stackoverflow.com/questions/4152223/why-are-scala-functions-limited-to-22-parameters) up to some [fixed arity](http://www.scala-lang.org/api/2.12.0/scala/Function22.html).
Something came into my mind slowly: programming is all about **abstraction**, can't we abstract over arities, just like we do with types?
The language, Rust, I learned more lately, also lacks the ability to be generic over types.
However, the developers now is wiser than before, and they hide the deficit as an implementation detail, [expecting it to be implemented in the future](https://github.com/rust-lang/rfcs/issues/376).
Interestingly, the macros of Rust support abstraction over arities in a really clever way, but I doubt it could be applied to normal functions.
Some users of Rust have purposed another feature called [named arguments](https://internals.rust-lang.org/t/pre-rfc-named-arguments/3831) that Scala natively supports.
Although being a feature more-or-less syntax-wise, it's also one that could improve readability.

At the same time of learning those reportedly more "practical" languages, I'm also being interested in those said to be more "academic" ones, e.g. Haskell, Idris, Agda.
All of those languages is leaning toward an interesting and super powerful fancy idea: dependent types.
As I've said, dependent types blur the distinction between compile time and runtime, but perhaps *too much*.
Why not forcing some computation to happen at compile time for the sake of efficiency?
I've heard that one of the Rust's main competitor, D, has this kind of feature called [CTFE (Compile Time Function Evaluation)](https://tour.dlang.org/tour/en/gems/compile-time-function-evaluation-ctfe).
Despite Haskell's plan to implement dependent types at the end, it would also seperate between parameters sent in at compile time and runtime.
Even Idris, already having dependent types, also have some [erasure](http://docs.idris-lang.org/en/latest/reference/erasure.html) mechanism to erase terms at compile time.
All of that implies we still want the distinction between phases, and a powerful type system needs not sacrifice it.

I'm especially fascinated by the language Agda.
To me, it seems like the grand unified theory that describe everything in the same way, that is, to make them *first-class*.
Entities are all abiding by the same laws, nontheless, there's still some way to distinguish between them, in the way of *modes*.
I'm not sure if the author of Agda realized it could be extended to support all the features I mentioned above.
I've even never heard of a word to describe that feature either, so I coined it.
I believe modes are the idea that solves all of these problems above in a coherent way.
The interaction between modes also gives rise to another novel feature that a language named PureScript supports, namely [row polymorphism](https://leanpub.com/purescript/read#leanpub-auto-record-patterns-and-row-polymorphism).

To sum up, I found some similarities between the languages I love and abstracted over some concepts in them with modes and extended it to the maximum I could imagine.
Much of the following content will be center on it.
