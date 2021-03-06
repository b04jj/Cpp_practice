# C++ Note

> You don’t have to know every detail of C++ to write good programs.
> Focus on programming techniques, not on language features.
```cpp
// to iterate over an iterable
// lets say v is an array
for(auto x : v){
    // some tasks 
}

/* if want to create the iterating variable without copying
the orginal data. we can pass x by reference */
for(auto& x : v){
    // some tasks
}
```

### Pass-by-reference
it is like passing the alias of the data at the arguments

```cpp
// avoid copying the passing data

void sort(vector<double>& v);

/* don't want to modify the arguments, also don't want to 
copy it */
double sum(const vector<double>& v);
```
## User-Defined Types

- **structure `struct`**

```cpp
// basic structure
struct Vector{
    int sz;
    double* elem;
}

// however, elem points to nothing, so we have to initialize it
void vector_init(Vector& v, int s){
    v.elem = new double[s];
    v.sz = s;
}

// The following demonstrate how to access `struct` member
void f(Vector r, Vector& rv, Vector* pv){
    int i1 = r.sz;
    int i2 = rv.sz;  // access through reference
    int i3 = pv->sz; // access through pointer
}
// reference is like a nickname, though different name, but the same person
```
- **Class**

Notice that regardless of the number of elements of each object, 
all the objects are of the same size (i.e. taking up the same size of space) 
```cpp
class Vector{
    public:
        Vector(int s): elem{new double[s]}, sz{s}{} // constructor
        // the part before {} is called memeber initializer list
        // TODO
    private:
        double* elem;
        int sz;
        // TODO
}
```
_A struct is simply a class with its members public by default_\
Notice that you can also define constructor and member function for `struct` objects.

- **Union**

The purpose of union is to save memory by using the same memory region for storing different objects at different times.\
That is the same class might have some subtypes, which can effect the type of clas member it uses.\

Use "naked"`union` can easily produce errors due to the confusion of types. use _tagged union_\

```cpp
enum Type{ str, num};

struct Entry{
    Type t;
    char* s; // if type is str
    int i; // if type is num
}; 

void f(Entry* p ){
    if(p->t == str){
        // do something on p->s
    }
    if(p->t == num){
        // do something on p->i
    }
}
// the above can be improved by:
union Value{ // all member share the same memory space and does't exist at the same time
    char* s;
    int i;
};

struct Entry{
    Type t;
    Value v;
};

void f(Entry* p){
    if(p->t == str){
        // do on p->v.s
    }
    if(p->t == num){
        // do on p->v.i
    }
}
```
- **Enumeration**

    1. `enum class`

    2. plain `enum`

In `enum class`, the variables are in a local scope of enum class. However, for plain `enum`,  those variables are in the same scope of `enum` name.

```cpp
// Error, cuz redefinition of variables
enum TrainLight{
    red,
    blue,
    pink
};
enum TrafficLight{
    red,
    blue,
    pink
};

// better use enum class
// No error
enum class TrainLight{
    red,
    blue,
    pink
};
enum class TrafficLight{
    red,
    blue,
    pink
};
```
Define operation for enumerations is recommended. Also, `enumeration class` is preferred over plain `enum`.

## Modularity

- **Separate compliation**
![](./figs/header.png)

- **Error handling**

```cpp
class Error{};

void f(){
    // some tasks...
    // if error happens, throw something
    // it's like a error version of return. you can throw anything
    throw Error
}

int main(){
    try{
        f(); // call f()
    }
    catch(Error){
        // if 'Error' is thrown, do the following
        // some tasks...
    } // so that the program will not directly terminated when Error happens
}
```
You can do error handling at any level. However, when working in a large project, you might not
know if the provided function does error handling inside, so that you don't have do it.

`noexcept` is a way to tell others that you have done error handling inside, so others don't have to catch anything (i.e. nothing will be thrown)

```cpp 
void f() noexcept{
    // TODO
}
// if an error is still thrown though declared noexcept, program will terminate with error
```

- **Invariant**

    The invariant of a function or class is its predefinition. For example, when we write a customized `Vector`, we would assume that the index should be an non-negative integer. Then this assumption is the invariant of Vector class.

```cpp
Vector::Vector(int s){
    if(s<0) throw length_error{};
    // TODO
}
```
However, sometimes it is preferable to check these assumptions while in the compile time. This is when the static assertion comes in.

- **Static Assertion**

    To check something in the compile time you can simply call `static_assert(A, S)`. This means that if `A` is not satisfied, it will print compile error message `S`.

   ```cpp
    static_assert(4<sizeof(int), "interger space is too small");
   ```
    _Be aware that you can only check the const expression with static assertions!!_

## Classes

- Concrete tpye





### constant member function

A constant member function can be accessed by both constant or non-constant object. But a non-constant member function can only be accessed by non-constant function. 

 Think about the following scenario: when you want your object to be a constant, you wouldn't want the data inside that object to be change by any of its member function. Thus, to protect the data inside, such as read(), you can only call the member function that is declared `const`. 

```cpp 
class Test{
    private:
        // todo
    public:
        // todo
        int getValue() const; // const Test can call
        void writeValue(int x); // const Test cannot call 
};

int main(){
    const Test t;
    int a = t.getValue(); // OK
    t.writeValue(10); // Error: writeValue is not a const memebr function
}
```

- **Container**

    Container is an object that stores a collection of objects inside, for example `Vector`.

- **Destructor**

    If allocating memory with `new`, when the data is no longer needed, it should be deallocated. Otherwise, there will be space wasted

    ```cpp 
    class Vector{
        private:
            double* elem;
            int sz;
        public:
            Vector(int s): elem{new double[s]}, sz{s}{ // maybe some error handling here
                for(int i = 0; i!=s; i++){
                    elem[i] = 0;
                }
            }
        
            ~Vector(){delete[] elem;}     
            double& operator[](int i);
            int size(); 
    }; 
    ```
    The way destructor is called is as follow:

    ```cpp
    void f(int n){
        Vector v1(n);
        
        // use v1
        
        {
           Vector v2(2*n);
            // use v2
        } // v2 is destroyed
    
         // use v1
    }// v1 is destroyed
    ```
- **Member Initializer List**

    The following is the useage of member initialier list:
    
    ```cpp
    class A{
        public:
            A(): x(0){}
            A(int i): x(i){}

        private:
            const int x;
    };
    ```
    However, why should we use it? Let's see the following example:
    ```cpp
    class A{
        public:
            A(){ x = 0 }
            A(int i){ x =i }
        
        private:
            int x;
    }; 
    
    class B{
        public:
            B(){
                a.x = 3;
            }

        private:
            A a;
    };
    ```

    If you want to initialize object B, you would have to call default constructor of A, and then assign `3` to `a.x`. But if you use member initializer list, you can directly call the constructor `A(int i)`.

    ```cpp
    class B{
        public:
            B(): a(3){}

        private:
            A a;
    };
    ```
    
    However, when you have some const member or a memeber that does not have 'default constructor', then you must use memeber initializer list.

    ```cpp
    class B{
        public:
            B(): a(3){}
        private:
            const A a;
    }
    ```
    
    In conclusion, the timing of using memeber initializer list is when you have to initialize a member with some arguments or you want to reduce the time to initialize a member.
    
    The moment a const type is initialized, you have to provide it with an argument (when an object is initialized, its constructor is called first, and then the constructor of its member object)
    
    **Reference**: [Why should I prefer to use member initialization lists?](https://stackoverflow.com/questions/926752/why-should-i-prefer-to-use-member-initialization-lists)

- **Special pointer**

    * Null pointer

        If initializing a pointer without assignment, it will be randomly assigned with a address of a memory that might be free or in use. Therefore, if we write something in them without notice, there might be some trouble.

        When you have not decided which adress you want to point to, just assign it with NULL
        like this : `int *ptr = NULL;`. 

    * Void pointer (generic pointer)

        A void pointer can point to variables of any type. However, it cannot be dereferenced. It should be casted before dereferenced. 
        
        ```cpp
        int var = 10;
        void *ptr = &var;
        
        cout << *ptr << endl; // compile error
        cout << *(int *) ptr << endl; // OK
        ```
        
        what `malloc()` returns is also a void pointer.

    * Dangling pointer

        When you `free()` to deallocate that memory, it would become unreserved. Yet, it still points to the same address as before. The value might or might not be changed (Surely, `free()` will not change that value)
    
    * Pointer arithmetic
    
    ```cpp
    int num = 5; // let's say the address is 100
    int *ptr = &num; // ptr : 100
    int *ptr2 = ptr + 3 // ptr2 : 100 + 3 * sizeof(int) = 112   

    // IMPORTANT : + 3 means plus the 3 units of that object size
    
    /* Notice : different types of pointer cannot do +/- operation */
    
    // arr and &arr is different in type
    /* &arr will evaluate this as the address of the whole array object. which is set to the base element of the array */
    
    int arr[5] = {1, 2, 3, 4, 5};
    cout << arr << endl;           // 100
    cout << &arr << endl;          // 100 
    cout << &arr[0] << endl;       // 100
    // all of the above address are the same

    cout << arr + 1 << endl;       // 104 = 100 + 1 unit of int size = 100 + 1*4 
    cout << &arr + 1 << endl;      // 120 = 100 + 1 unit of arraysize = 100 + 5*4 
    cout << &arr[0] + 1 << endl;   // 104 = 100 + 1 unit of int size = 100 + 1*4
    ```
    **Notice that** : the type of `&arr` is `int*[5]`.

<p align="center">
  <img src="./figs/arr.gif">
</p


