# Buffer pool manager

## 1. c++ learning: template
* There are two kinds of templates: **template function** and **template class**. ExtendibleHash is template class.
* Both tempate function and template class needs to be **initialized**. 
* When the template function is called, the process of generating code starts and the process is called "initialization". For a class template we must supply additional information inside angle brackets following the template’s name to initialize class template.

### 1.1 template function example

```
template <typename T>
int compare(const T &v1, const T &v2)
{
    if (v1 < v2) return -1; if (v2 < v1) return 1; return 0;
}
```

* The above template requires the parameter passed in to support "<" operator. Depending on how the compiler manages instantiation, these errors may be reported at link time.

### 1.2 template class example

```
template <typename T> class Blob { 
public:
    typedef T value_type;
    typedef typename std::vector<T>::size_type size_type;
    // constructors
    Blob();
    Blob(std::initializer_list<T> il);
    // number of elements in the Blob
    size_type size() const { return data->size(); }
    bool empty() const { return data->empty(); }
    // add and remove elements
    void push_back(const T &t) {data->push_back(t);}
    // move version; see § 13.6.3 (p. 548)
    void push_back(T &&t) { data->push_back(std::move(t)); } 
    void pop_back();
    // element access
    T& back();
    T& operator[](size_type i); // defined in
private:
    std::shared_ptr<std::vector<T>> data;
    // throws msg if data[i] isn't valid
    void check(size_type i, const std::string &msg) const;
};
```

* initialization

```
Blob<int> ia;
```

* The "const" in "void check(size_type i, const std::string &msg) const;" means the function "check" will not change the member of Blob class.

###1.3 template class's member function declaration

* member function can be defined either inside template or outside template. Example of function defined outside template below. "template" keyword followed by typename. We also need to specify this function is for "Blob" by using Blob<T>::

```
template <typename T>void Blob<T>::check(size_type i, const std::string &msg) const{	if (i >= data->size())		throw std::out_of_range(msg);}
```

* constructor

```
template <typename T>Blob<T>::Blob(): data(std::make_shared<std::vector<T>>()) { }
```

* refer the type using **typename** keyword

```
template <typename T>typename T::value_type top(const T& c){	if (!c.empty())
		return c.back();	else		return typename T::value_type();}
```

* auto generated declaration. When implementing Extendible hash table, I tried to declare a private member function outside template which returns a nested defined type **Bucket**. Note since **Bucket's** definition is inside template, in order to properly infer it, I have to use "->" tail expressioin so that it is under closure of **```Extendiblehash<K, V>::```**

```
//inside template, definition:
std::shared_ptr<Bucket> breakBucket(std::shared_ptr<Bucket> oldBucketint, int offSet);

//outside template, declaration:
template <typename K, typename V>
auto ExtendibleHash<K, V>::breakBucket(std::shared_ptr<Bucket> oldBucket, int offSet) -> std::shared_ptr<Bucket> {

}
```

### 1.4 Nested member template

* We can also define a member template of a class template. In this case, both the class and the member have their own, independent, template parameters. example:

```
template <typename T> class Blob {	template <typename It> Blob(It b, It e);	// ... 
};
```

* initialization: We must supply arguments for the template parameters for both the class and the function templates

```
vector<long> vi = {0,1,2,3,4,5,6,7,8,9};
Blob<int> a2(vi.begin(), vi.end());
```

### 1.5 Friend and Template

* We want to define a template ```BlobPtr<T>``` as a friend of ```Blob<T>``` so that the pointer have access to the non-public field of Blob


```
template <typename> class BlobPtr;template <typename> class Blob; // needed for parameters in template <typename T>bool operator==(const Blob<T>&, const Blob<T>&); 

template <typename T> class Blob {	// each instantiation of Blob grants access to the version of	// BlobPtr and the equality operator instantiated with the same type	friend class BlobPtr<T>; 
	friend bool operator==<T>(const Blob<T>&, const Blob<T>&); };
```

* We start by **declaring** ```Blob```, ```BlobPtr``` templates (first two lines).
* These declarations are needed for the parameters used in the ```operator==``` function declaration and the friend declarations in ```Blob```.
* ```operator==<T>``` function inside ```Blob``` takes ```Blob<T>&``` as parameter. ```friend class BlobPtr<T>;``` also refers the same ```T```. Thus the friendship is **restricted** to those instantiations of BlobPtr and the equality operator that are instantiated with the same type. i.e ```BlobPtr<char>``` does not access the nonpublic parts of ```Blob<int>```. But it can access any field of ```Blob<char>```

## 2. c++ learning: Copy control

* There is an good [stackoverflow](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom) explaination. **Should take a look, when I get some time.**

### 2.1 copy constructor

```
class Foo { 
public:	Foo();	Foo(const Foo&);};
```

### 2.2 direct initialization vs copy initialization

* direct initialization: use ordinary constuctor

```
string dots(10, '.');string s(dots);```

* copy initialization: use copy constructor

```
string s2 = dots;string null_book = "9-999-99999-9"; 
string nines = string(100, '9');
```

**Note:** § 13.6.2 (p. 534), if a class has a move constructor, then copy initialization sometimes uses the move constructor instead of the copy constructor.

### 2.3 copy assignment operator

* copy constructor takes reference as input and copy assignment operator returns a reference to their left-hand operand.

```
class Foo {
public:
    Foo& operator=(const Foo&);
}
```

### 2.4 copy constructor vs copy assignment operator
* copy constructor controls how that class is initialized.

```
Element<int, int> e = Element<int, int>(1, 1);
Element<int, int> e2 = e;  //this is e2's initialization which triggers copy constructor
```

* copy assignment operator controls how objects of its class are assigned.

```
Element<int, int> e = Element<int, int>(1, 1);
Element<int, int> e2 = Element<int, int>(2, 2);
e2 = e  //Since e2 has been declared at previous line, this is e2's assignment, not initialization. So it triggers copy assignment operator
```

* Example

```
class Element {
public:
    K key;
    V value;
    //constructor
    Element() {
    };

    //copy constructor
    Element(const Element &e): key(e.key), value(e.value) {
        cout << "copy constructor is called" << endl;
    };

    //copy assignment operator
    Element& operator=(const Element &e) {
        key = e.key;
        value = e.value;
        cout << "copy assignment operator is called" << endl;
        return *this;
    };

    Element(K k, V v): key(k), value(v) {
        cout << "constructor is called" << endl;
    };
};
```

### 2.5 const data member and copy assignment operator

* **If class has const member, it does not allow to have copy assignment operator. Intrinsically, copy assignment operator "=" is not replace the pointer to the object, but to modify the existing object with the value on the right hand-side.** It uses the information/the member of right hand side to modify the the member of left hand side.

* "When you make a data member const, you're telling the compiler and all the world that this data member never changes. Of course then you cannot assign to it and you certainly must not trick the compiler into accepting code that does so, no matter how clever the trick.
You can either have a const data member or an assignment operator assigning to all data members. You can't have both" from [stackoverflow](https://stackoverflow.com/questions/4136156/const-member-and-assignment-operator-how-to-avoid-the-undefined-behavior)

* c++ primer "The synthesized copy-assignment operator is defined as deleted if a member has a deleted or inaccessible copy-assignment operator, or if the class has a const or reference member."

### 2.6 rules

* **If you need destructor, normally you will need both copy constructor and copy assignment operator. Normally you need destructor because class's member is allocated on heap which means copying will make multiple instance share the same heap allocated member. That member's destructor can be called multiple times.**
* **If you need to copy constructor, you normally need copy assignment operator and vice versa.**

### 2.7 delete and default constructor

```
struct NoCopy {	NoCopy() = default; // use the synthesized default constructor 
	NoCopy(const NoCopy&) = delete; // no copy 
	NoCopy &operator=(const NoCopy&) = delete; // no assignment 
	~NoCopy() = default; // use the synthesized destructor	// other members};
```

##3 c++ learning: move
* There is a very good explaination on [stackoverflow](https://stackoverflow.com/questions/3106110/what-is-move-semantics)

### 3.1 why move?
* performance boost
* there are things that can not be copied, like I/O or unique_ptr.

### 3.2 move constuctor
* it takes rvalue reference as input

```
// move won't throw any exceptions.
// member initializers take over the resources in "s"
StrVec::StrVec(StrVec &&s) noexcept: elements(s.elements), first_free(s.first_free), cap(s.cap){// leave s in a state in which it is safe to run the destructors.elements = s.first_free = s.cap = nullptr; 
}
```

#### 3.2.1 the differene between lvalue and rvalue
* copied some explaination from previous stackoverflow as below

```
string a(x);                                    // Line 1
string b(x + y);                                // Line 2
string c(some_function_returning_a_string());   // Line 3
```

* Did you notice how I just said x three times (four times if you include this sentence) and meant the exact same object every time? We call expressions such as x "lvalues".
* The arguments in lines 2 and 3 are not lvalues, but rvalues, because the underlying string objects have no names, so the client has no way to inspect them again at a later point in time. rvalues denote temporary objects which are destroyed at the next semicolon (to be more precise: at the end of the full-expression that lexically contains the rvalue).
* **An rvalue of class type is an expression whose evaluation creates a temporary object. Under normal circumstances, no other expression inside the same scope denotes the same temporary object.**
* This is important because during the initialization of b and c, we could do whatever we wanted with the source string, and the client couldn't tell a difference!
* variable is lavlue

#### 3.2.2 rvalue reference
* As we know, moving lvalue is dangerous, since lvalue can be referenced again later after the move. As move will "steal" the resource from lvalue, try use those resources in lvalue after move will cause undefine behavior.
* rvalue reference implemented in c++ 11 is used to solve the problem.
* it is reference that binds to a rvalue. 
* We use ```&&r``` to get rvalue reference, while ```&r``` is lvalue reference.

```
int &r1 = i;   //ok, lvalue
int &&r2 = i;  //error, rvalue can not bind to variable which is lvalue.

int &r1 = i * 42 //error, i * 42 is rvalue
```

* The move constructor takes rvalue reference as parameter. This prevents the lvalue to be moved. For example, ```unique_ptr```'s move constructor takes rvalue reference.

```
unique_ptr(unique_ptr&& source)   // note the rvalue reference
    {
        ptr = source.ptr;
        source.ptr = nullptr;
    }
```

```
unique_ptr<Shape> a(new Triangle);
unique_ptr<Shape> b(a);                 // error
unique_ptr<Shape> c(make_triangle());   // okay
```

#### 3.2.3 about noexcept
* To avoid this potential problem, vector must use a copy constructor instead of a move constructor during reallocation unless it knows that the element type’s move constructor cannot throw an exception. 

### 3.3 ```std:move```
* Sometimes, we want to move from lvalues. That is, sometimes we want the compiler to treat an lvalue as if it were an rvalue, so it can invoke the move constructor, even though it could be potentially unsafe.
* When ```std:move``` is called. We’re using the ```move``` constructor. Since we are using move constructor, the memory managed by those objects will not be copied. 
* It is essential to realize that the call to move promises that we do not intend to use lvalue again except to assign to it or to destroy it.
* ```std::move(some_lvalue)``` casts an lvalue to an rvalue, thus enabling a subsequent move. But it does not move anything by itself.

```int &&rr3 = std::move(rr1);
```

### 3.4 xvalue

* Note that even though ```std::move(a)``` is an rvalue, its evaluation does not create a temporary object. This conundrum forced the committee to introduce a third value category. Something that can be bound to an rvalue reference, even though it is not an rvalue in the traditional sense, is called an xvalue (eXpiring value). The traditional rvalues were renamed to prvalues (Pure rvalues).

### 3.4 move assignment operator
* like copy, for move, we also have move assignment operator
* These members are similar to the corresponding copy operations, but they “steal” resources from their given object rather than copy them.
* In particular, once its resources are moved, the original object must no longer point to those moved resources. Setting moved resources in rvalue to nullptr is very important. The reason is because the rvalue may be destroyed and during the time, the resources in the rvalue may also be destroyed by deconstructor.
* add noexcept if possible
* Like a copy-assignment operator, a move-assignment operator must guard against self-assignment
* note &rhs is the address of rvalue. Checking ```this != $rhs``` is basically checking if they are the same address.

```
StrVec &StrVec::operator=(StrVec &&rhs) noexcept{	// direct test for self-assignment 
	if (this != &rhs) {		free(); // free existing elements 
		elements = rhs.elements; // take over resources from rhs		first_free = rhs.first_free; 
		cap = rhs.cap;		// leave rhs in a destructible state
		rhs.elements = rhs.first_free = rhs.cap = nullptr;	} 
		return *this; 
}
```

* From stackoverflow link above we know, We can also pass original type to move assignment operator. Take string as an example.

```
string& operator=(string that)
    {
        std::swap(data, that.data);
        return *this;
    }
```

* Note that we pass the parameter that by value, so that has to be initialized just like any other string object. Exactly how is that going to be initialized? In the olden days of C++98, the answer would have been "by the copy constructor". In C++0x, the compiler chooses between the copy constructor and the move constructor based on whether the argument to the assignment operator is an lvalue or an rvalue.

### 3.5 synthesized move operator
* if a class defines its own copy constructor, copy-assignment operator, or destructor, the move constructor and move- assignment operator are not synthesized. 
* The compiler will synthesize a move constructor or a move-assignment operator only if the class doesn’t define any of its own copy-control members and if every nonstatic data member of the class can be moved. The compiler can move members of built-in type. It can also move members of a class type if the member’s class has the corresponding move operation.
* Classes that define a move constructor or move-assignment operator must also define their own copy operations. Otherwise, those members are deleted by default.

### 3.6 when move is used V.S when copy is used
* Rvalues Are Moved, Lvalues Are Copied ...
```
StrVec v1, v2;v1 = v2;                     // v2 is an lvalue; copy assignmentStrVec getVec(istream &);    // getVec returns an rvaluev2 = getVec(cin);            // getVec(cin) is an rvalue; move assignment
```


## 4. c++ learning others
### 4.1 RAII-style
* In RAII, holding a resource is a class invariant, and is tied to object lifetime: resource allocation (or acquisition) is done during object creation (specifically initialization), by the constructor, while resource deallocation (release) is done during object destruction (specifically finalization), by the destructor. In other words, resource acquisition must succeed for initialization to succeed. Thus the resource is guaranteed to be held between when initialization finishes and finalization starts (holding the resources is a class invariant), and to be held only when the object is alive. Thus if there are no object leaks, there are no resource leaks.
* [wiki](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)

### 4.2 mutex
* the std::mutex is not usually directly accessed. Normally accessed by std::unique_lock, std::lock_guard

```
std::lock_guard<std::mutex> guard(mtx);
```

* The concept is called RAII-style. The lock is released after gurad object is out of scope.
* I got a problem of compiling code. It turns out that lock_guard will internally trigger mutex.lock which will change the state of a class. Thus, if a method is marked as const, there is no way to call lock_guard. [detail at stackoverflow](https://stackoverflow.com/questions/48133164/how-to-use-a-stdlock-guard-without-violating-const-correctness)
* https://en.cppreference.com/w/cpp/thread/mutex
* https://en.cppreference.com/w/cpp/thread/lock_guard

### 4.3 remove element from list

* remove will invalidate the list. So do not use ```for``` loop. Use ```while```.
* Note, erase returns Iterator **following** the last removed element.

```
auto it = list.begin()

while (it != list.end() {
	if (*it & 1) {  //if element is odd number
	  it = list.erase(it); 
	} else {
	  ++it;
	}
}
```

* The way I prefer most is to use lambda and remove_if

```
list.remove_if(list.begin(), list.end(),
    [](int const &i) {
    	return i & 1;
    });
```
* [stackoverflow](https://stackoverflow.com/questions/596162/can-you-remove-elements-from-a-stdlist-while-iterating-through-it)

### 4.4 store object as key in std::map

* we can not store reference as key in std::map. As reference is immutable, map requires modification of key, I guess. Also I heard this is syntax problem where underline the reference implementation is using pointers.
* We normally have three choices
 * make class implement Comparator
 * make class implement < (less than)
 * store pointer of class as key, like following code. [stackoverflow](https://stackoverflow.com/questions/25122932/pointers-as-keys-in-map-c-stl) . The default implementation will compare the addresses stored by the pointers, so different objects will be considered as different keys.

* The code below stores pointer as key. ```p1``` and ```p2``` are different objects. But they point to the same ```i```. So the addresses stored by them are the same. As the result, the following code will print "find p2".

```
	std::unordered_map<int *, int> map;
    int i = 1;
    int *p1 = &i;
    int *p2 = p1;

    map[p1] = 2;

    if (map.find(p2) != map.end()) {
-> 		//this branch will be executed
        cout << "find p2" << endl;
    } else {
        cout << "not find p2" << endl;
    }
```

 * Note the same thing applies to shared_ptr

```
    std::unordered_map<shared_ptr<int>, int> map;
    shared_ptr<int> p1 = make_shared<int>(1);
    
    //copy constructor, p2 is different from p1. 
    //But they point to the same int object
    shared_ptr<int> p2 = p1;     
    map[p1] = 2;

    if (map.find(p2) != map.end()) {
-> 		//this branch will be executed
        cout << "find p2" << endl; 
    } else {
        cout << "not find p2" << endl;
    }
```

### 4.5 retrieve the last element of list

```
//error
value = *(queue.end()--);

//correct
value = *(--queue.end());

//or
queue.back();
```


## 5.Extendible Hash Table