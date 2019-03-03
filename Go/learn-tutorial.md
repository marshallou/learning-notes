https://tour.golang.org

### variables

1. initializer

```
var i, j int = 1, 2
var c, python = true, false

k := 3 (only used inside function)
```

variable block

```
var (
  i int = 1
  j boolean = true
  z complex128 = cmplx.Sqrt(-5 + 12i)
  )
```

2. type conversions 

```
var i int = 42
var f float = float(i)
```

3. const

```
const word = "世界"
const turth = true

const(
  Big = 1 << 100
  Small = Big >> 99
  )
```

### Flow control

1. for: variable defined is only available in for's scope

```
for i := 0; i < 10; i++ {
   fmt.Println(sum)
}
```

init and post statements are optional

```
for ; sum < 1000; {
 sum += sum
}
```

drop smeicolons

```
for sum < 1000 {

}
```

2. if

```
if x < 0 {
  
}
```

define a variable to use in if block (also available in else block). 
the predicate is "v < lim"

```

if v := math.Pow(x, n); v < lim {
  return v
}
```

3. switch: break is provided automatically

```
switch os := runtime.GOOS; os {
  case "darwin":
    fmt.Println("OS X.")
  case "linux":
    fmt.Println("Linux.")
  default:
    // freebsd, openbsd,
    // plan9, windows...
    fmt.Printf("%s.", os)
  }
```

switch true is equivalent to if-else chain

```
 switch {
  case t.Hour() > 12:
    fmt.Println("good morning")
  case t.Hour() < 17:
    fmt.Println("good afternoon")
  default:
    fmt.Println("good evening")
 }
```

4. Defer
A defer statement defers the execution of a function until the surrounding function returns.

The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.

eferred function calls are pushed onto a stack. When a function returns, its deferred calls are executed in last-in-first-out order.


### More types: structs, slices and maps

1. pointers

```
var p *int // define pointer
i := 42
p = & i    // & generate a pointer
fmt.Println(*p)  // * denotes the pointer's value -> dereferencing/indirecting
```

2. struct

```
type Vertex struct {
  X int
  Y int
}

fmt.Println(Vertex{1, 2})

v := Vertex{1, 2}
v.X = 4
fmt.Println(v.X)
```

struct pointer

```
v := Vertex{1, 2}
var p *Vertex = &V
fmt.Println(p.X)      //(*p).X is equivalent 
```

struct constructor?

```
v1 := Vertex{1, 2}
v2 := Vertex{x: 1}
```

3. arrays

array can not be resized

```
var a [10]int
a[0] = 1
a[1] = 2

primes := [6]int{2, 3, 5, 7}
```

4. Slice

slice is defined by []T

```
primes := [6]int{2, 3, 5, 7, 11, 13}
var s []int = primes[1:4]

//slice literal
var s := []int{2, 3, 5}
```

slice default is similar to python

```
a[0:10]
a[:10]
a[0:]
a[:]
```

slice length and capacity

```
len(s)
cap(s)

// short the length of s
s = s[:0]， s = s[2:]

// extend the length of s
s = s[:4]

// previous operation is called re-slice
```

nil: slice with both length and capacity of 0. Note slice[0:0] is not nil, since the 
capcacity of r may not be 0

make: create dynamic sized slice

```
a := make([]int, 5)     //length is 5, capacity is also 5
b := make([]int, 0, 5)  //length is 0, capacity is 5

```

slice of slice
```
board := [][]string{
  []string{"_", "_", "_"},
  []string{"_", "_", "_"}
}
```

append: append will allocate a bigger array if the original array is too small. 

```
apend(s []T, vs ...T) []T
```

example show new array allocated.

```
var s[]int
s = append(s, 0, 1, 2)
q := s[:]

s = append(s, 3) // now s = {0, 1, 2, 3,} but q is still {0, 1, 2}
```

5. range

range used in for loop

```
var pow = []int{1, 2, 4, 8}

func main() {
  for i, v := range pow {

  }
}

//drop the value
for i := range pow {

}

//drop the index
for _, v := pow {

}
```

6. Maps

definition and initialization

```
//declaration
var m map[string]Vertex

//initialization
m = make(map[string]Vertex)

//initialization2: literal
m = map[string]Vertex{
  "Bell Labs": Vertex {
    40.68, -74.39,
  },
  "Google": Vertex {
    37.42, -122.08
  }
}

//initialization3: since "Vertex" has appeared in type definition of map
//the "Vertex" in literal can be obmit

m = map[string]Vertex {
  "Bell Labs": {40.68, -74},
  "Google": {37, -122}
}

```

Mutating map

```
//insert
map[key] elem

//retrieve
elem = m[key]

//delete
delete(m, key)

//test
elem, ok = m[key]
elem, ok := m[key] //if elem and ok is not defined

```

exercise 

```
func WordCount(s string) map[string]int {
    words := strings.Fields(s)
    wordCountMap :=make(map[string]int)
  for _, word := range words {
     count, ok := wordCountMap[word]
     if ok {
         wordCountMap[word] = count + 1
     } else {
         wordCountMap[word] = 1
     }
  }
  return wordCountMap
}
```

7. function values and closure

function is also a value which can be passed into another function as argument

```
func compute(fn func(float64, float64) float64) float64 {
    return fn(3, 4)
}
```

closure

```
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}
```

### Methods

1. Go does not have class. But methods has receiver. method's name is using capter letter

```
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X * v.X +  v.Y *v.Y)
}

func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())   
    //v.Abs() looks like "Abs" becomes the method of "v", but it is defined at top level
    //so go is very functional oriented?
}

```

2. Receiver of the method and the method have to be defined within the same package.

3. Pointer receivers

Method with non-poiter receiver operates on a copy of the original value. While pointer receiver method operates 
on orignal value directly. 

```
func (v Vertex) Scale1(f float64) {
  v.X = v.X * f
  v.Y = v.Y * f
}

func (v *Vertex) Scale2(f float64) {
  v.X = v.X * f
  v.Y = v.Y * f
}

func main() {
  v := vertex{3, 4}
  v.Scale1(10)      //v defined in main is not scaled
  v.Scale2(10)      //v defined in main is scaled
  fmt.Println(v.Abs())
}

```

> question: as we can always rewrite a Method to be a normal function. What is the difference? When to use function and when to use Method? 
The answer is Interface

### Interfaces

1. Method has receiver argument. So we can see the receiver argument implements a Method.
If the Method has the same signature as interface. The value of type interface can hold the receiver argument

```
type Abser interface {
  Abs() float64
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
   ....
}

func main() {
  var a Abser
  f := myfloat(-math.Sqrt2)
  a = f
}

```

2. nil receiver

```
type I interface {
    M()
}

//Type *T implements I interface 
type T struct {
    S string
}

func (t *T) M() {
    if (t == nil) {
        fmt.Prinln("nil")
        return
    }
    
    fmt.Println(t.S)
}

func main() {
    var i I
    var t *T   //define a varaible "t" which is nil
    i = t      //assign "t" to "i". But i is NOT nil. We can treat i as tuple of (value, a concreate type), i.e
               // (nil, main.*T)

    i.M()      //this will print "nil". If we remove "nil" handling in Method "M", 
               //it will throw error of "invalid memory address"
}
```

3. The empty interface

an empty interface may hold values of any type

4. type assertions

```
// this is an assertion that interface i's concrete type is T
t := i.(T)

// if not sure, we can use "test"
t, ok := i.(T)

// type switch
switch v := i.(type) {
  case int:
    fmt.Printf("Twice %v is %v\n", v, v*2)
  case string:
    fmt.Printf("%q is %v bytes long\n", v, len(v))
  default:
    fmt.Printf("I don't know about type %T!\n", v)
  }
```

5. most ubiquitous interface is Stringer

```
type Stringer interface {
    String() string
}
```

6. error interface

```
type error interface {
    Error() string
}

//check error, when calling functions

i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

7. Readers interface
https://tour.golang.org/methods/21


