
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

