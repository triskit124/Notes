
# Table of contents

- [Table of contents](#table-of-contents)
- [Memory management](#memory-management)
  - [Smart Pointers](#smart-pointers)
    - [`std::unique_ptr`](#stdunique_ptr)
    - [`std::shared_ptr`](#stdshared_ptr)
    - [Passing smart pointers to functions](#passing-smart-pointers-to-functions)
    - [Returning from functions](#returning-from-functions)
- [Classes](#classes)
  - [Default constructors](#default-constructors)
  - [C plus plus' most vexxing parse](#c-plus-plus-most-vexxing-parse)
  - [Explicitly defaulted constructor \[c++ 11\]](#explicitly-defaulted-constructor-c-11)
  - [Explicitly deleted constructor \[c++ 11\]](#explicitly-deleted-constructor-c-11)
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


# Memory management 

## Smart Pointers

- In Modern C++ `raw pointers` should only allowed if there is no ownership involved
- If ownership is involved
  - Use `std::unique_ptr` by default
  - Use `std::shared_ptr` if shared ownership is required

### `std::unique_ptr`

Cannot be copied. Requires `std::move` in order to transfer ownership.

To create
```cpp
std::unique_ptr<foo> bar = std::make_unique<foo>();
```

To get underlying raw pointer
```cpp
bar* = foo.get();
```

### `std::shared_ptr`

Can be casted to other shared_ptr types using `const_pointer_cast`, `dynamic_pointer_cast`, etc. Unique pointers cannot be cast.

```cpp
std::shared_ptr<foo> bar = std::make_shared<foo>();
```

### Passing smart pointers to functions
- A function should accept a smart pointer as a parameter **only if** ownership sharing/transferral is required
  - Pass `shared_ptrs` by value for shared ownership
  - Pass `unique_ptrs` by value for transferred ownership (requires move semantics)
- Otherwise use raw `pointers` 

### Returning from functions

- Unique and shared pointers can directly be returned by value from functions.

# Classes

## Default constructors

Once any constructor is implemented, the compiler stops autogenerating the default constructor. One solution is to get in the habit of `explicitly defaulting` the constructor:
```cpp
public:
  MyFooClass() = default;
```

## C plus plus' most vexxing parse

When instantiating an object of a class using the *default constructor*, you need to omit the function-call style parens:
```cpp
FooClass bar(); // error, compiler treats as function definition
FooClass bar; // correct
FooClass bar { }; // also correct
```

## Explicitly defaulted constructor [c++ 11]

Saves needing to implement an empty constructor
```cpp
public:
    MyClass() = default;
```

## Explicitly deleted constructor [c++ 11]

For classes with only `static` methods. Can disable constructor via
```cpp
public:
    MyClass() = delete;
```

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






