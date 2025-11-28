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

- Stack allocation: automatic lifetime, fast.
- Heap allocation: dynamic, requires explicit management unless using RAII/smart pointers.
- Raw pointers vs references: prefer references for non-nullable aliasing and smart pointers for ownership.

Example with raw pointers and ownership pitfalls:

```cpp
#include <iostream>

int* make() {
    return new int(100); // caller must delete
}

int main() {
    int* p = make();
    std::cout << *p << '\n';
    delete p; // missing delete -> memory leak
}
```

Better: use std::unique_ptr or std::shared_ptr (covered in Module 03).

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
