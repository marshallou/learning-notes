### Type erase: 
the generic type will be erased at run time which means we are not able to get parameterized class information at Runtime.
For example, 

```
List<String> strs = new ArrayList<>();
strs.getClass() will lose information of "String"
```

### Why TokenType: 
According to [goole guava doc](https://github.com/google/guava/wiki/ReflectionExplained), 
TokenType is mainly used to create, manipulate, and query Type information.

For example, Gson library `fromJson` method will take a Json `String` and `Class<T>` to generate
a new instance of given class. 

If the given Json is a list of String, we need to pass in corresponding `Class<T>` to be `List<String>` which
is not able to get from the way mentioned above.

For places where we need type which is `List<String>` like this, we can use TokenType.

```
TypeToken<List<String>> typeToken = new TypeToken<List<String>>() {};
typeToken.getType();
```

The magic happens in `{}`. `{}` creates an anonymous sub class of `TypeToken<List<String>>`.

TypeToken utilizes following feature to get parameterized type `String`:

```
Class#getGenericSuperclass() does the following
Returns the Type representing the direct superclass of the entity (class, interface, primitive type or void) represented by this Class.

If the superclass is a parameterized type, the Type object returned must accurately reflect the actual type parameters used in the source code.
```

Some of library's, e.g Gson, will use this Type info to convert

### Parameterize Class Type of TypeToken
For previous example, we hardcoded the TypeToken's List type to be `String`. By doing
```
new TypeToken<List<String>>(){};
```

But let's say, I would like to write a funciton which returns a `TypeToken<List<T>>` so that the type is dynamic at Runtime.
How to do that?
According to [stackoverflow question](https://stackoverflow.com/questions/14139437/java-type-generic-as-argument-for-gson),
we can achieve this by

```
TypeToken.getParameterized(List.class, myType).getType();
```

Not only `List` can be parameterized, we can also do similar thing for `Map<K, V>`. 

```
static <K, V> TypeToken<Map<K, V>> mapToken(TypeToken<K> keyToken, TypeToken<V> valueToken) {
  return new TypeToken<Map<K, V>>() {}
    .where(new TypeParameter<K>() {}, keyToken)
    .where(new TypeParameter<V>() {}, valueToken);
}
...
TypeToken<Map<String, BigInteger>> mapToken = mapToken(
   TypeToken.of(String.class),
   TypeToken.of(BigInteger.class));
TypeToken<Map<Integer, Queue<String>>> complexToken = mapToken(
   TypeToken.of(Integer.class),
   new TypeToken<Queue<String>>() {});
```

### use subclass to capture type T
```
abstract class IKnowMyType<T> {
  TypeToken<T> type = new TypeToken<T>(getClass()) {};
}
...
new IKnowMyType<String>() {}.type;
```

The Type info is preserved because `IKnowMyType<String>() {}` creates an anonymous sub class.
Then `type` field is another anonymous sub class which is initiated by calling `getClass()` on previous an anonymous sub class.

According to source code of TypeToken's constructor here:
```
  protected TypeToken(Class<?> declaringClass) {
    Type captured = super.capture();
    if (captured instanceof Class) {
      this.runtimeType = captured;
    } else {
      this.runtimeType = of(declaringClass).resolveType(captured).runtimeType;
    }
  }
```
`super.caputured()` will be class `String`, so `type` field will be initiatized by `TypeToken<String>`
