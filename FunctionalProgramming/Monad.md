### Functor
Functor is a type class which implements 'fmap'.
```
class Functor f where  
    fmap :: (a -> b) -> f a -> f b 
```

Looking at definition of Functor, we know this type class is different from other typeclass like Eq etc.
Because "f" is not a concreate class but a type contructor. It takes a type parameter and returns a concreate type.

Looking at definition of fmap, what it does is, given a transform function which transforms a to b and concreate functor a type, it produces functor b type.

The fmap will not change the intrinsic property of functor, after applying fmap, a functor is
still a fucntor. Just the content in "box" changed from "a" to "b"

### Maybe example
 - Maybe is type contrutor which takes a type parameter a and produces a concreate type Maybe a
 - Maybe implements fmap like this:
instance Maybe where
  fmap f (Just x) = Just (f x)
  fmap f Nothing = Nothing
 - When Maybe invokes fmap, type a has already been determined. You can invoke fmap on May Int,
Maybe String. But you can invoke fmap on just Maybe. 

### Two Functor laws:
 - id law: fmap id = id. fmap on identity function equals applying identity function directly.
 - function composition law: fmap (f.g) F = fmap f (fmap g F)

### Maybe, [] and other Functors share some same properties:
 - Those are all type constructors which take a type parameter to produce a concreate type. 
This implies the "box" or "context" nature of Functor where itself does not have anything. 
It becomes concreate by putting things into "box", 
 - it has to implement fmap function which satisfies functor law

### Based on num "3", we know function which takes a parameter and return one result is also
a Functor. Why?

```
instance Functor ((->) r) where
    fmap f g = (\x -> f (g x)) 
```

"-> r" is a type constructor which takes another type parameter will produce a function type.
e.g "-> r a" means a function takes type r as input and returns type a.

Thus, when applying fmap on function (r -> a), the type "a" has already been determined which means
transform functions' input type has already been determined.

Thus "Predicate" is function is also a functor. It takes a random type and returns a boolean value.
transform function can be type (Bool -> b). But chaining Prediate in Java has nothing to do with Functor
and transform.

### references
https://stackoverflow.com/questions/44965/what-is-a-monad
http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html