1. Type erase: the generic type will be erased at run time which means we are not able to get parameterized class 
information at Runtime.
For example, 

```
List<String> strs = new ArrayList<>();
strs.getClass() will lose information of "String"
```

2. Why TokenType: 
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

3.other usage of TokenTpe:
