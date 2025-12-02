
# Table of contents

- [Table of contents](#table-of-contents)
- [Memory management (ch 7)](#memory-management-ch-7)
  - [Arrays](#arrays)
  - [Pointers](#pointers)
  - [Common memory pitfalls](#common-memory-pitfalls)
  - [Smart Pointers](#smart-pointers)
    - [`std::unique_ptr`](#stdunique_ptr)
    - [`std::shared_ptr`](#stdshared_ptr)
    - [`std::weak_ptr`](#stdweak_ptr)
    - [Passing smart pointers to functions](#passing-smart-pointers-to-functions)
    - [Returning from functions](#returning-from-functions)
- [Classes (ch 8-9)](#classes-ch-8-9)
  - [In-class member initializers](#in-class-member-initializers)
  - [Default constructors](#default-constructors)
  - [Compiler-generated default constructor](#compiler-generated-default-constructor)
  - [Most vexxing parse](#most-vexxing-parse)
  - [Explicitly defaulted constructor](#explicitly-defaulted-constructor)
  - [Explicitly deleted constructor](#explicitly-deleted-constructor)
  - [Constructor initializers (member initializer lists)](#constructor-initializers-member-initializer-lists)
  - [Copy constructors](#copy-constructors)
  - [Delegating constructors](#delegating-constructors)
  - [Converting constructors](#converting-constructors)
  - [Summary of compiler-generated constructors](#summary-of-compiler-generated-constructors)
  - [Destructors](#destructors)
  - [Assignment](#assignment)
  - [Dynamic memory allocation in objects](#dynamic-memory-allocation-in-objects)
    - [Freeing memory in destructors](#freeing-memory-in-destructors)
    - [Handling copying and assignment](#handling-copying-and-assignment)
      - [Copy-swap idiom](#copy-swap-idiom)
    - [R-value references](#r-value-references)
    - [Move semantics](#move-semantics)
    - [The rule of five](#the-rule-of-five)
    - [The rule of zero](#the-rule-of-zero)
  - [More about methods](#more-about-methods)
    - [Mutable data members](#mutable-data-members)
    - [Inline methods](#inline-methods)
    - [Const static data members](#const-static-data-members)
  - [Comparison operators](#comparison-operators)
    - [Equality](#equality)
    - [Spaceship](#spaceship)
- [Inheritance (ch 10)](#inheritance-ch-10)
  - [Client's view of inertance](#clients-view-of-inertance)
  - [A derived class' view of inheritance](#a-derived-class-view-of-inheritance)
  - [Preventing inheritance](#preventing-inheritance)
  - [Overriding methods](#overriding-methods)
  - [A client's view of overriden methods](#a-clients-view-of-overriden-methods)
  - [Virtual destructors](#virtual-destructors)
  - [Virtual constructors](#virtual-constructors)
  - [Order of construction](#order-of-construction)
  - [Order of destruction](#order-of-destruction)
  - [Referring to parent names](#referring-to-parent-names)
  - [Slicing](#slicing)
  - [Polymorphism](#polymorphism)
  - [Abstract base classes](#abstract-base-classes)
  - [Multiple inheritance](#multiple-inheritance)
  - [Obscure inheritance issues](#obscure-inheritance-issues)
    - [Changing the overriden method's return type](#changing-the-overriden-methods-return-type)
    - [Inherited constructors](#inherited-constructors)
    - [Hiding of inherited constructors](#hiding-of-inherited-constructors)
    - [Initialization of data members](#initialization-of-data-members)
  - [Special cases in overriding methods](#special-cases-in-overriding-methods)
  - [Copy constructors and assignment operators in derived classes](#copy-constructors-and-assignment-operators-in-derived-classes)
  - [Runtime type facilities](#runtime-type-facilities)
  - [Virtual base classes](#virtual-base-classes)
  - [Casts](#casts)


# Memory management (ch 7)

In modern C++, you should avoid manual memory management as much as possible. For example, use `std::vector` instead of dynamically allocated c-style arrays, and use `smart pointers` instead of using `new` to create raw pointers.

- Stack memory
  - "Automatic" variables delared normally are allocated automatically on the stack and are deleted when they go out of scope:
    ```cpp
    int i { 7 };
    ```
- Free store
  - Variables declared with `new` are allocated on the free store and aren't deleted until manually done
    ```cpp
    int* ptr { new int };
    ```
  - Note that the above variable `ptr` is still on the stack even though it points to memory on the free store
  - You should always immediately initialize a pointer or intialize it to `nullptr`.
  - Every line that allocates memory using `new` should correspond to another line that releases it using `delete`
  - It is recommended but not required to set a pointer to `nullptr` after releasing the memory
    ```cpp
    int* ptr { new int };
    delete ptr;
    ptr = nullptr;
    ```
  - Don't use `malloc`, `realloc`, or `free` in C++

## Arrays

- Arrays are allocated in contiguous pieces of memory
- Variable-sized arrays are not standard C++. Some compilers may support them, but it is not recommended.
- Arrays can be initialized using
```cpp
/* on the stack */
int myArray[5]; // uninitialized values, bad!
int myArray[5] { 1, 2, 3, 4, 5 };
int myArray[5] { 0 };  // inits all to 0
int myArray[5] { }; // also inits all to 0

/* on the free store */
int* myArray { new int[] { 1, 2, 3, 4, 5 } };
delete[] myArray; // use the array version of delete
myArray = nullptr;
```

- The advantage of putting an array on the free store (ie dynamically allocating the memnory) is that you can define the size at runtime:
```cpp
size_t numDocs = getNumDocs();
Decument* newArray { new Document[numDocs] };
```

- Creating an array of objects will call the `default constructor` for each of the objects.
- Arrays are pointers
  - The address of a pointer is the address of the first element in memory.
  - Arrays are treated as pointers when passed into functions. Must also provide the size of the array since arrays do not store that information

## Pointers

- Pointers are just addresses to memory. 
- When you dereference a pointer using the `*` operator, you are looking up the contents of the memory at that address.
- Because pointers are addresses, they are somewhat weakly typed
- You can cast from one ptr type to another using c-style casts:
  ```cpp
  Document* myDoc { getDocument() };
  char* myCharPtr { (char*)myDoc }; // c-style cast to unrelated type. compiles, but what is the result?
  char* myCharPtr { static_cast<char*>(myDoc) }; // static cast to unrelated type, won't compile
  ```

- Pointer arithmetic
  - If you have an `int` pointer, incrementing the pointer value by 1 moves forward in memory by the size of an int.

## Common memory pitfalls

- Underallocating data buffers and out of bounds access
  - Solution: avoid c-style strings and arrays, use c++ strings and std::vectors instead
- Memory leaks
- Double-deletion, use-after-free, invalid pointers 

## Smart Pointers

- In Modern C++ `raw pointers` should only allowed if there is no ownership involved
- If ownership is involved
  - Use `std::unique_ptr` by default
  - Use `std::shared_ptr` if shared ownership is required

### `std::unique_ptr`

Cannot be copied and represents sole ownership of a resource. Requires `std::move` in order to transfer ownership.
You can specify a custom deleter function if needed.

To create:
```cpp
// always use std::make_unique to construct a unique pointer
// this is the most memory-safe way to construct
std::unique_ptr<foo> bar = std::make_unique<foo>(); // uses value initializtion (calls default constructors and sets numerical builtin types to 0)
```

To dereference, use `*` as normal:
```cpp
(*mySmartPointer).go();
```

To get underlying raw pointer:
```cpp
bar* = foo.get();
```

To free the underlying pointer and optionally change it to a different pointer:
```cpp
mySmartPointer.reset(); // frees resource and sets to nullptr
mySmartPointer.reset(new Simple {}); // frees and assigns new pointer
Simple* simple { mySmartPointer.release() }; // release ownership and get raw pointer
```


### `std::shared_ptr`

Can be casted to other shared_ptr types using `const_pointer_cast`, `dynamic_pointer_cast`, etc. Unique pointers cannot be cast.

```cpp
std::shared_ptr<foo> bar = std::make_shared<foo>();
```

Can call `reset()` but the underlying memory is **only freed when all references have been deleted/reset**.

### `std::weak_ptr`

Contains a reference to a resource managed by a `shared_ptr`. A `weak_ptr` does not own the resource, so it does not prevent freeing by the shared_ptr.
Can be used, for example, to see if a resource managed by a shared_ptr has been freed, use the `lock()` method:
```cpp
void useResource(std::weak_ptr<Simple>& weakSimple) {
  auto resource { weakSimple.lock() };
  if (resource) {
    std::cout << "Resource is still alive!" << std::endl;
  } else {
    std::cout << "Resource has been freed!" << std::endl;
  }
}
```

### Passing smart pointers to functions
- A function should accept a smart pointer as a parameter **only if** ownership sharing/transferral is required
  - Pass `shared_ptrs` by value for shared ownership
  - Pass `unique_ptrs` by value for transferred ownership (requires move semantics)
- Otherwise use a reference, const-reference, or raw pointer (if `nullptr` is a valid value for the parameter) 

### Returning from functions

- Unique and shared pointers can directly be returned by value from functions.

# Classes (ch 8-9)

## In-class member initializers

It is recommended to always initialize data members in a class. One way is to use in-class member initializers:
```cpp
private:
  m_value { 0 };
```

## Default constructors

A constructor that can be called without any parameters is called the `default constructor`.

## Compiler-generated default constructor
When no constructors are defined, the compiler generates the `compiler-generated default constructor` that calls the default constructor on all object members but does not initialize language primatives such as `ints` or `doubles`.

## Most vexxing parse

When instantiating an object of a class using the *default constructor*, you need to omit the function-call style parens:
```cpp
FooClass bar(); // error, compiler treats as function definition
FooClass bar; // correct
FooClass bar { }; // also correct
```

## Explicitly defaulted constructor

Once any constructor is implemented, **the compiler stops autogenerating the default constructor**. One solution is to get in the habit of `explicitly defaulting` the constructor:
```cpp
public:
    MyClass() = default;
```

## Explicitly deleted constructor

For classes with only `static` methods. Can disable constructor via
```cpp
public:
    MyClass() = delete;
```

## Constructor initializers (member initializer lists)

```cpp
SpreadsheetCell::SpreadsheetCell(double initialValue) : m_value { initialValue } { }
```

Initializing data members with `ctor initializers` behaves differently than initializng inside the constructor body. When a class is created, data members are created before the constructor is called. By the time you assign a value to an object inside the ctor body, you are actually modifying its value. A ctor-initializer allows you to provide initial values for data members as they are created, which is more efficient.

If your class has a data member that has a default constructor, you do not have to explicitly initialize it in the ctor-initializer.

Several types of data members must be initialized in a ctor-initializer or with an in-class initializer:
- const data members
- reference data members
- object data members w/o default constructors
- base classes w/o default constructors

If a class has both in-class initializers and constructor initializers, the value provided in the constructor initializer is used.

## Copy constructors

In most cases, there is no need to specify your own copy constructor. C++ autogenerates one for you. The exception is usually when you have dynamically allocated memory or other resources in your class. Similar to default constructors, you can explicitly default/delete copy constructors.

## Delegating constructors

Constructors that call a different constructor from the same class. Must be used in the ctor initializer.

## Converting constructors

When you have a constructor that takes a single parameter that is a different type from the class, it can be used as a `converting constructor` to convert from that type to the class. 

It is reccommended to mark a converting constructor as `explicit` to prevent this happening implicitly.

## Summary of compiler-generated constructors

If you define:

- no constructors
  - compiler generates a default constructor and copy constructor
- a default constructor only 
  - compiler generates a copy constructor
- a copy constructor only
  - compiler generates no constructors
- a single or multi-argument non-copy constructor only
  - compiler generates a copy constructor
- a default constructor and a single or multi-argument non-copy constructor
  - compiler generates a copy constructor

```cpp
class SpreadsheetCell
{
  public:
    SpreadsheetCell(double initVal); // constructor converts a double to a SpreadsheetCell
    SpreadsheetCell(const std::string& initVal); // constructor converts a string to a SpreadsheetCell
};

SpreadsheetCell myCell { 4 };
myCell = "4";
```

## Destructors

If you don't define a destructor, the compiler generates one that recursively calls destructos on members.

## Assignment

Copy constructor is only called if the object doesn't exit. If it already exists, the `assignment operator` is called instead

```cpp
SpreadsheetCell& operator=(const SpreadsheetCell& rhs); // always return a reference for copy assignment operators
```

## Dynamic memory allocation in objects

### Freeing memory in destructors

If you dynamically allocate memory in an object, the place to free that memory is the destructor.

### Handling copying and assignment

When you copy an object, primitives (ints, doubles) and **pointers** are `shallow-copied`, meaning that they just copy/assign data members from the source directly to the destination object. This is an issue for dynamically allocated memory, as the new object will contain a pointer to the same memory as the original.

Thus, whenever you have dynamically allocated memory in a class, you should write your own copy constructor and assignment operator to provide a `deep-copy` of the memory!

#### Copy-swap idiom

When implementing assignment operator, you want an all or nothing mechanism. You don't want an exception to happen in the middle of assignment, which would leave the object in a bad state:

- Implement a swap method for the class, as well as a non-member swap function for Standard Library algorithms:
```cpp
class Spreadsheet
{
    public:
        Spreadsheet& operator=(const Spreadsheet& rhs)
        { 
            Spreadsheet tmp { rhs }; // use copy constructor
            swap(temp); // swap current instance with new instance
            return *this; // old memory will be cleaned up in descructor for temp
        };

        void swap(Spreadsheet& other) noexcept
        {
            std::swap(m_width, other.m_width);
            std::swap(m_height, other.m_height);
            std::swap(m_cells, other.m_cells); // use swap utility for dynamically allocated memory
        };
};

// non-member swap function
void swap(Spreadsheet& first, Spreadsheet& second) { first.swap(second); };
```

### R-value references

- An `lvalue` is something of which you can take an address, for example a named variable
- An `rvalue` is anything that is not an `lvalue`, for example a literal or temperorary object/value
- An `rvalue reference` is a reference to an `rvalue`. Specifically it refers to temporary objects or an object that has been explicitly moved using `std::move()`. Rvalue references can be utlizied to copy pointers instead of large objects
  - If a temporary value is assigned to an rvalue reference, the lifetime is extended while that reference stays in scope

```cpp
void foo(string& msg); // function accepting an lvalue reference
void foo(string&& msg); // function accepting an rvalue reference

string a { "hello" };
string b { "world" };

foo(a); // calls the lvalue reference function overload
foo(a + b); // calls the rvalue reference function overload
foo("hello"); // calls the rvalue reference function overload
foo(std::move(a)); // calls the rvalue reference function overload
```

### Move semantics

Moving moves ownership of memory and other resources from one object to another. It basically does a shallow copy of data members and switches ownership of allocated memory to prevent dangling pointers and memory leaks.

To implement `move semantics` in a class, you must implement a `move constructor` and `move assignment operator`. These move ownership of the memory from a source object to a new object. Thus, move semantics are only useful if the source object is no longer needed.

The compiler automatically generates default move constructor if and only if the class has no user-declared copy constructor, copy assignment operator, move assignment operator, or destructor.

Similarly, the compiler autogenerates a default move assignment operator iff the class has no user-declared copy constructor, move constructor, copy assignment, or destructor.

```cpp
class Spreadsheet
{
    public:
        // move constructor
        Spreadsheet(Spreadsheet&& src) noexcept
        {
            swap(*this, src); // use the swap method introduced previously in copy-swap section
        }

        // move assignment operator
        Spreadsheet& operator=(Spreadsheet&& rhs) noexcept
        {
            swap(*this, rhs);
            return *this;
        }
};
```

### The rule of five

When you delcare one or more of the five `special member functions`, you should declare all of them - either through explicitly defining, defaulting, or deleting.

**Special member functions:**
- Copy constructor
- Copy assignment operator
- Move constructor
- Move assignment operator
- Destructor

### The rule of zero

In modern c++, avoid writing any of the five SMFs by avoiding dynamically allocating resources in classes. Instead use modern constructs such as Standard Library containers and smart pointers. The rule of five should be limited to custom RAII classes.

## More about methods

### Mutable data members

Tagging a data member as `mutable` means that it is ok for `const` methods to modify it. Neat!

### Inline methods

Tagging a method as `inline` hints to the compiler that the call to the method can be replaced by the body of the method itself. This can potetntially be an optimization if the method is very small and is called a lot.

### Const static data members

Use `const static` data members for classes instead of global constants if the constants apply only to the class.

## Comparison operators

### Equality

In C++ 20, it is recommended you implement `operator==` as a method. It will automatically implement `operator!=` for you. it is recommended you make the result `[[nodiscard]] as well:
```cpp
[[nodiscard]] bool SpreadsheetCell::operator==(const SpreadsheetCell& rhs) const;
```

### Spaceship

In C++ 20, if you've implemented `operator==`, the only other comparison operator you need to implement is `operator<=>` (spaceship). Once you've implemented these two operators, C++ will automatically provide support for all 6 comparison operators (<, <=, >, >=, ==, !=).

```cpp
std::partial_ordering SpreadsheetCell::operator<=>(const SpreadsheetCell& rhs) const {
    return getValue() <=> rhs.getValue;
}
```

Note: you can explicitly default the spaceship operator (which just calls comparison operators on all data members), which will use compiler-generator comparisons for all comparison operators.

# Inheritance (ch 10)

## Client's view of inertance

To a client, or any other part of code, an object of type `Derived` is also an object of type `BaseClass`. This is not true in the reverse direction

Pointers/references to `Base*/Base&` can also point to a class of type `Derived`. However, pointers to `Base*` cannot use methods defined in `Derived`.

## A derived class' view of inheritance

A class of type `Derived` can access `public` and `protected` members of class `Base`.

It is recommended to set data members to `private` by default to start out with the highest level of encapsulation. Getters and setters can be added as needed.

## Preventing inheritance

Can mark a class as `final` to prevent classes inhertiting from it
```cpp
class Foo final {};
```

Methods can similarly be marked as `final`.

## Overriding methods

Only methods marked as `virtual` in the base class can be overriden by a derived class.
```cpp
virtual void someMethod();
```

Methods can be overriden using the `override` keyword. adding `virtual` is redundant. Note that `override` is technically optional but strongly recommended.
```cpp
void someMethod() override;
```

## A client's view of overriden methods

- Objects of type `Base` will call the base version of the method
- Objects of type `Derived` will call the overriden version of the method
- Pointers/references of type `Base*` / `Base&` will call the appropriate version of the method depending on if the actual object is of type `Base` or `Derived` (so long as the method was correctly marked as `virtual`)
  - If `virtual` is mistakenly not added to method delcaration, this is known as `hiding`

## Virtual destructors

Destructors **should almost always** be `virtual`, except if the class is marked as `final`.  If you don't need to do any work in the destructor, you should default it, at least for your base classes:
```cpp
virtual ~Base() = default;
```

## Virtual constructors

Constructors _cannot_ be `virtual` because you always specify the exact class being constructed when creating an object.

## Order of construction

Classes are initialized starting with grandparent, then parent, then child, etc.

1. If class has a `Base` class, the `default` constructor of the base class is executed, unless a base class constructor is called in the `ctor-initializer`, in which case that constructor is called
2. Non-static data members of the class are constructed in the order in which they are delared
3. The body of the class' constructor is executed

Note that if a derived class overrides a base class' `virtual` method, calling the method in the `Base` constructor/destructor will call the base class' method, and not the derived.

## Order of destruction

1. body of the `Base` class' destructor is called
2. data members are destroyed in reverse order
3. parent class, if any, is destroyed

## Referring to parent names

A method from a `Base` class that has been overriden by a `Derived` class can still be called using `scope resolution`:
```cpp
void Derived::foo () {
  Base:foo()
  // do other things
}
```

## Slicing

Slicing occurs when a derived class is assigned by value to a base type. The assigned object becomes type `Base` and loses its derived methods:
```cpp
Base myDerivedClass { myDerived }; // sliced
```

This does not happen with pointers/references. This is called `upcasting`:
```cpp
// casting from derived to base is called upcasting
Base* myDerivedClass { &myDerived }; // not sliced
Base& myDerivedClass { myDerived }; // not sliced
```

Casting from base to derived is called `downcasting` and is generally not good practice as there is not guarantee that the object is actually of the derived type. The cast should be checked before using:
```cpp
// downcasting
Derived* myDerived { dynamic_cast<Derived*>(myBase) };
if (!myDerived) {
  throw std::runtime_error();
}
```

Note that while one can cast up and down the inheritance hierarchy, one cannot cast to a *sibling* class.

## Polymorphism

Polymorphism allows objects with a common parent class to be used interchangably and to use objects in place of their parents

## Abstract base classes

A class with a `pure virtual` method is said to be an `abstract` class. Pure virtual methods are virtual methods that have no implementation. Abstract classes cannot be instantiated

```cpp
// an abstract class
public:
  virtual void foo() = 0; // pure virtual method
```
## Multiple inheritance

Classes can inherit from more than one base class:
```cpp
class Baz : public Foo, public Bar { };
```

- Constructors are called in order of how they are listed in class definition
- If both Base classes implement the same method, must use `scope resolution` to specify which to call, or explicitly cast to the desired type
- Ambiguity can also arise if the Base classes have a data member with the same name

## Obscure inheritance issues

### Changing the overriden method's return type

An overriding method can change the return type if the original return type is a pointer or reference and the new return type is a pointer/reference to a descendant class (ie `covariant return types`)

### Inherited constructors

You cannot use constructors from a Base class unless specified with the `using` keyword:
```cpp
// derived class
public:
  using Base::Base; // inherits ALL constructors from Base class
  Derived(int i);
```

It is not possible to inherit constructors from one Base class if another Base class has a constructor with the same parameter list. The Derived class would need to explicitly define the constructors.

### Hiding of inherited constructors

Derived classes can hide constructors from Base class by defining a constructor with the same parameters.

### Initialization of data members

When using inherited constructors, make sure all data members are properly initialized. If a Base class constructor is used to construct a Derived class, certain data members of the Derived class may not be initialized if they don't exist in the Base class.

## Special cases in overriding methods

- You cannot override a `static` method. Further, a method cannot be both `static` and `virtual`.
- Overriding a method that has overloads in the Base class hides all of the overloads in the Derived class. They can still be called if you have a reference or pointer to the Derived object of type `Base&`/`Base*`.
  - To avoid obscure bugs, you should override all versions of an overloaded method
- You can override `private` and `protected` methods of a Base class
- When overiding a method that has a default argument, you should provide a default argument as well, preferably the same value. This is because if you have a reference/pointer to an object, the default arguments will be assigned according to the type of the reference/pointer, *not* the actual underlying type. This is because default parameters are assigned at compile-time, not run-time. 

## Copy constructors and assignment operators in derived classes

- When defining derived classes, you need to be careful about copy constructors and `operator=`. 
- If your Derived class has no special data members (pointers), it does not require a copy constructor or `operator=`. The default will be used for Derived data members, and the custom ones will be used for Base data members.
- If you do define them in the Derived class, you need to explicitly chain to the Parent copy constructor:
  ```cpp
  Derived(const Derived& src) : Base { src } { };
  ```

## Runtime type facilities

Run-Time Type Information (RTTI) facilities provide information about an Object's type at runtime:
- `dynamic_cast` is one example
- `typeid` lets you query an object's type at runtime. Use of `typeid` in conditional code should be used sparingly and is often a sign of poor design and could be replaced by using `virtual` methods instead. It is, however, useful for debugging or logging

## Virtual base classes

Virtual base classes prevent ambiguity in multiple inheritance when two or more Parent classes share the same Parent class
```cpp
class Dog : public virtual Animal
```

## Casts

- `const_cast()`
  - Removes const-ness
- `static_cast()`
  - Performs explicit conversions between builtin types (eg `int`, `double`, etc)
  - Can also be used to explicitly convert between classes if the target class has a Constructor that takes in the current Class type. This is done automatically by the compiler if possible.
- `reinterpret_cast()`
  - Can be used to perform casts that technically aren't allowed by static casts, but make sense to the progrmmer. Eg you can reinterpret_cast a `void*` to a known class. 
  - You can also use reinterpret_cast to cast from pointers to integral types (eg cast Foo* to int64. The int type must be large enough to hold the pointer. 
  - Be careful with these casts as they don't provide any type checking
- `dynamic_cast()`
  - Provides runtime checks on casts within an object inheritance heierarchy.
  - Used to cast pointers or references
  - Returns `nullptr` for bad pointer conversions and `std::bad_cast` for references
  - To use, your classes must have at least one `virtual` method (ie be polymorphic types)
- `std::bit_cast()` (C++20)
  - Creates a new object of given type and copies bits from source object to new object. 
  - Requires that size of source and target are same.
  - Requires that source and target are `trivially copyable`, meaning the underlying bytes of an object can be copied to and from an array
- C-style casts technically work but span all of the C++ casts so it is strongly recommended not to use them in C++ code






