# GADTs

Normal `data` types are called ADTs in Haskell.  
GADTs are the **G**eneralized version of ADTs.  
GADTs let you define inductive families, that is, to be specific on the return types of the variants.  
We can define a GADT by writing `where` instead of an equal sign \(`=`\) after the parameters of the `data`.

```
@allPub:
data Array[_ : Nat; T] where
    nil : Array[0nat; T]
    cons : [n : Nat](T; Array[n; T]) -> Array[Nat::succ(n); T]
```



