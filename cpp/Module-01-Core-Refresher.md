# Module 01 — Core C++ Refresher

Objective: Revise foundational C++ concepts and syntax.

Table of contents

- [1.1 Variables, data types, and type deduction](#11-variables-data-types-and-type-deduction)
- [1.2 Pointers, references, and memory management](#12-pointers-references-and-memory-management)
- [1.3 Functions, lambdas, and std::function](#13-functions-overloading-default-args-lambdas-and-stdfunction)
- [1.4 Polymorphism (moved to dedicated module)](Module-02-Polymorphism.md)
- [1.5 Operator overloading and special member functions](#15-operator-overloading-and-special-member-functions)

---

## 1.1 Variables, data types, and type deduction (auto, decltype)

- Basic integer, floating point, bool and char types.
- Standard library types: std::string, containers, smart pointers.
- auto allows the compiler to deduce types; decltype inspects an expression’s type.

Example:

```cpp
#include <iostream>
#include <vector>
#include <type_traits>

int main() {
    auto x = 42;                // int
    auto y = 3.14;              // double
    const auto name = "Alex"; // const char*

    std::vector<int> v{1,2,3};
    for (auto &el : v)          // range-based loop with auto reference
        el *= 2;

    decltype(x) z = x + 1;      // z has same type as x (int)
    std::cout << x << ' ' << y << ' ' << z << '\n';
}
```

Tip: use auto for complex iterators, lambda return types, and when type verbosity is high. Prefer explicit types for API clarity.

---
## 1.2 Pointers, references, and memory management (stack vs. heap)

- Stack allocation: automatic lifetime, fast (objects are destroyed when the scope ends).
- Heap allocation: dynamic, lifetime controlled by the programmer (requires deletion or RAII/smart pointers).
- Raw pointers vs references: prefer references for non-nullable aliasing, prefer smart pointers (std::unique_ptr/std::shared_ptr) for ownership, and use raw pointers for non-owning observation or low-level code.

This section dives deeper into how pointers and references behave, common pitfalls, and idiomatic C++ usage.

Pointers — fundamentals

- A pointer stores the memory address of another object. Use `&` to get the address and `*` to dereference.
- Pointers can be null (use `nullptr`), can be reassigned to point to different objects, and allow pointer arithmetic for low-level array access.

Example: basics and a leak example (intentional leak)

```cpp
#include <iostream>

int* make_leak() {
    // returns pointer to heap memory — caller MUST delete to avoid a leak
    return new int(100);
}

int main() {
    int* p = make_leak();
    std::cout << *p << '\n';
    // OOPS: no delete -> memory leak
}
```

Fixed example using raw delete (not recommended long-term):

```cpp
#include <iostream>

int* make_owned() {
    return new int(42);
}

int main() {
    int* p = make_owned();
    std::cout << *p << '\n';
    delete p; // correct cleanup — but manual ownership is fragile
}
```

Better (modern C++): use RAII via smart pointers (safer and preferred):

```cpp
#include <iostream>
#include <memory>

std::unique_ptr<int> make_safe() {
    return std::make_unique<int>(42);
}

int main() {
    auto p = make_safe(); // automatic cleanup when p leaves scope
    std::cout << *p << '\n';
}
```

Pointer arithmetic & arrays (low-level)

- When dealing with low-level arrays, pointers support arithmetic but you must avoid out-of-bounds access.

```cpp
int arr[3] = {1,2,3};
int* ptr = arr; // points to arr[0]
std::cout << ptr[1] << '\n'; // 2 (pointer + index)
std::cout << *(ptr + 2) << '\n'; // 3
```

References — fundamentals

- A reference is an alias for an existing object. It must be initialized at creation and cannot be reseated.
- References are safer and usually preferred for function parameters when the callee should not take ownership and the value must exist.

Example — reference usage and const references:

```cpp
#include <iostream>

void increment(int& x) { ++x; }
void show(const int& x) { std::cout << x << '\n'; }

int main(){
    int a = 10;
    increment(a);      // modifies a
    show(a);           // prints 11

    const int& r = a;  // r is a const reference, can't modify through r
    // int& bad;        // error: references must be initialized
}
```

Reference vs pointer: quick comparison

- References: must be initialized, cannot be null (in normal usage), no reseating, often used for function parameters and operator overloads.
- Pointers: can be null, can be reassigned, support arithmetic, often used for optional/non-owning relationships or when low-level control is required.

Const correctness

- `const T* p` — pointer to const T; you cannot change *p via p, though pointer can be reassigned.
- `T* const p` — const pointer to T; pointer cannot be changed, but *p can be.
- `const T* const p` — const pointer to const T; neither pointer nor pointed value can change.

Double pointers and function pointers (brief):

- `T**` is a pointer-to-pointer — useful in some APIs or when managing arrays of pointers.
- Function pointers (and std::function/lambdas) are used for callbacks; prefer std::function where type erasure and flexibility are needed.

Ownership guidance & best practices

- Prefer automatic storage (stack) for small objects with clear scope.
- Prefer RAII (smart pointers, value types like std::string/std::vector) for heap-managed resources.
- Use references for mandatory, non-owning access; use raw pointers only for non-owning or low-level code where nullability and reseating are needed.
- For ownership, prefer `std::unique_ptr` (exclusive ownership) and `std::shared_ptr` (shared ownership). Use `std::weak_ptr` to break cycles.

When to use which:
- Use references for function parameters when you expect an object and don’t need to reassign or store ownership.
- Use `std::unique_ptr` for private ownership and resource lifetime management.
- Use raw pointers for optional, non-owning references when you need to allow null, or for low-level APIs.

See Module 03 — Modern C++ Features for smart pointer patterns and detailed ownership examples.

---
## 1.3 Functions: overloading, default arguments, lambdas, and std::function

- Overload functions by signature, be mindful of ambiguity.
- Default arguments reduce boilerplate.
- Lambdas are lightweight function objects; std::function can hold any callable (type-erased).

Example combining all:

```cpp
#include <iostream>
#include <functional>

int add(int a, int b = 0) { return a + b; }

int main() {
    auto twice = [](int x){ return x * 2; };

    std::function<int(int)> f = twice;
    std::cout << add(2, 3) << ' ' << add(5) << ' ' << f(4) << '\n';
}
```

---
## 1.4 Polymorphism (moved)

Polymorphism details were moved to a dedicated module so we can explore runtime vs static polymorphism, object slicing, multiple inheritance, RTTI, and practical examples in much more depth.

See: [Module 02 — Polymorphism in C++](Module-02-Polymorphism.md)

---
## 1.5 Operator overloading and special member functions (Rule of Three/Five/Zero)

- Overload operators that have natural semantics for your class.
- Rule of Three: if you define destructor, copy constructor, or copy assignment, define all three.
- Rule of Five extends Rule of Three for move constructor and move assignment in C++11+.
- Rule of Zero: prefer using RAII types so you don't need custom special members.

Example of operator overloading + special members:

```cpp
#include <iostream>
#include <cstring>

class Buffer {
    char* data_{nullptr};
    std::size_t size_ = 0;
public:
    Buffer(const char* s) { // ctor allocates
        size_ = std::strlen(s);
        data_ = new char[size_+1];
        std::strcpy(data_, s);
    }

    ~Buffer() { delete[] data_; }

    // copy
    Buffer(const Buffer& other): size_(other.size_) {
        data_ = new char[size_+1];
        std::strcpy(data_, other.data_);
    }

    Buffer& operator=(const Buffer& other){
        if (this == &other) return *this;
        delete[] data_;
        size_ = other.size_;
        data_ = new char[size_+1];
        std::strcpy(data_, other.data_);
        return *this;
    }

    // move
    Buffer(Buffer&& other) noexcept : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr; other.size_ = 0;
    }

    Buffer& operator=(Buffer&& other) noexcept {
        if (this == &other) return *this;
        delete[] data_;
        data_ = other.data_; size_ = other.size_;
        other.data_ = nullptr; other.size_ = 0;
        return *this;
    }

    const char* data() const { return data_; }
};

int main(){
    Buffer b("Hello");
    Buffer b2 = b;               // copy
    Buffer b3 = std::move(b2);   // move
    std::cout << b.data() << ' ' << b3.data() << '\n';
}
```

When possible, prefer the Rule of Zero: use std::string, std::vector, smart pointers and avoid manual new/delete.

---

Continue to Module 02 — Polymorphism for a focused, deeper look at runtime and static polymorphism.

[Back to Overview](00-Overview.md)
[Next: Module 02 — Polymorphism in C++](Module-02-Polymorphism.md)
