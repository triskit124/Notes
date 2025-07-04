
# Smart Pointers

- In Modern C++ `raw pointers` should only allowed if there is no ownership involved
- If ownership is involved
  - Use `std::unique_ptr` by default
  - Use `std::shared_ptr` if shared ownership is required

## `std::unique_ptr`

Cannot be copied. Requires `std::move` in order to transfer ownership.

To create
```cpp
std::unique_ptr<foo> bar = std::make_unique<foo>();
```

To get underlying raw pointer
```cpp
bar* = foo.get();
```

## `std::shared_ptr`

Can be casted to other shared_ptr types using `const_pointer_cast`, `dynamic_pointer_cast`, etc. Unique pointers cannot be cast.

```cpp
std::shared_ptr<foo> bar = std::make_shared<foo>();
```

## Passing smart pointers to functions
- A function should accept a smart pointer as a parameter **only if** ownership sharing/transferral is required
  - Pass `shared_ptrs` by value for shared ownership
  - Pass `unique_ptrs` by value for transferred ownership (requires move semantics)
- Otherwise use raw `pointers` 

## Returning from functions

- Unique and shared pointers can directly be returned by value from functions.

# Classes and Inheritance

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

## Virtual destructors

Destructors should almost always be `virtual`, except if the class is marked as `final`.  If you don't need to do any work in the destructor, you should default it, at least for your base classes:
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

A method from a `Base` class that has been overriden by a `Derived` class can still be called:
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

This does not happen with pointers/references
```cpp
// casting from derived to base is called upcasting
Base* myDerivedClass { &myDerived }; // not sliced
Base& myDerivedClass { myDerived }; // not sliced
```

Casting from base to derived is called `downcasting` and is generally not good practice as there is not guarantee that the object is actually of the derived type
```cpp
// downcasting
Derived* myDerived { dynamic_cast<Derived*>(myBase) };
if (!myDerived) {
  throw std::runtime_error();
}
```

## Polymorphism

Polymorphism allows objects with a common parent class to be used interchangably and to use objects in place of their parents

## Abstract base classes

A class with a `pure virtual` method is said to be an `abstract` class. Pure virtual methods are virtual methods that have no implementation.
```cpp
public:
  virtual void foo() = 0; // pure virtual method
```

Abstract classes cannot be instantiated

## Obscure inheritance issues

### Changing the overriden method's return type

An overriding method can change the return type if the original return type is a pointer or reference and the new return type is a pointer/reference to a descendant class (ie `covariant return types`)



