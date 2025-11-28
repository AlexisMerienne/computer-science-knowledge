 # Module 02 — Polymorphism in C++

 Objective: Deep dive into polymorphism — both runtime (dynamic) and static polymorphism — with practical examples, pitfalls, and best practices.

 Table of contents

 - [Overview and motivation](#overview-and-motivation)
 - [Runtime polymorphism: virtual functions and vtables](#runtime-polymorphism-virtual-functions-and-vtables)
 - [Abstract base classes and interfaces](#abstract-base-classes-and-interfaces)
 - [Object slicing and pointer/reference usage](#object-slicing-and-pointerreference-usage)
 - [Multiple and virtual inheritance (pitfalls and examples)](#multiple-and-virtual-inheritance-pitfalls-and-examples)
 - [Polymorphism with smart pointers and ownership](#polymorphism-with-smart-pointers-and-ownership)
 - [Runtime type information: typeid and dynamic_cast](#runtime-type-information-typeid-and-dynamic_cast)
 - [Static polymorphism alternatives (CRTP) and when to choose it](#static-polymorphism-alternatives-crtp-and-when-to-choose-it)
 - [Performance considerations and trade-offs](#performance-considerations-and-trade-offs)
 - [Small exercises & next steps](#small-exercises--next-steps)

 ---

 ## Overview and motivation

 Polymorphism lets you write code that operates on objects of different types through a common interface. In C++ this traditionally means dynamic dispatch (virtual functions), but templates and CRTP provide compile-time (static) polymorphism. Knowing when to use each style and how to avoid common pitfalls is critical for robust, correct, and performant C++.

 ---

 ## Runtime polymorphism: virtual functions and vtables

 A virtual function enables late binding; the compiler arranges for calls through pointers/references to go through an object's vtable at runtime.

 Example:

 ```cpp
 #include <iostream>
 #include <memory>

 struct Base {
     virtual void hello() const { std::cout << "Base\n"; }
     virtual ~Base() = default; // always add a virtual destructor for polymorphic base
 };

 struct Derived : Base {
     void hello() const override { std::cout << "Derived\n"; }
 };

 int main(){
     std::unique_ptr<Base> p = std::make_unique<Derived>();
     p->hello(); // calls Derived::hello via vtable
 }
 ```

 Notes:
 - Always declare base destructors virtual when deleting via base pointers.
 - virtual functions can be overridden and marked `override` to catch errors.

 ---

 ## Abstract base classes and interfaces

 Define an abstract interface with one or more pure virtual functions ("= 0"). This model provides a clear contract and allows multiple implementations.

 ```cpp
 struct IShape {
     virtual double area() const = 0;
     virtual ~IShape() = default;
 };

 struct Circle : IShape { double r; Circle(double r):r(r){} double area() const override { return 3.14159 * r * r; } };
 ```

 ---

 ## Object slicing and pointer/reference usage

 If you store a derived object in a base object by value, you slice off derived parts; use pointers or references for polymorphism.

 Bad (slicing):

 ```cpp
 struct Base { int a; virtual ~Base()=default; };
 struct Derived : Base { int b; };

 Base b = Derived(); // Derived::b is sliced away
 ```

 Good:
 - store std::unique_ptr<Base> or std::shared_ptr<Base> or references

 ---

 ## Multiple and virtual inheritance (pitfalls and examples)

 Multiple inheritance is powerful but complex. Virtual inheritance solves diamond problems but adds complexity (and often performance and memory overhead).

 ```cpp
 struct Top { int v; };
 struct Left : virtual Top {};
 struct Right : virtual Top {};
 struct Bottom : Left, Right {};

 // Bottom contains a single shared Top subobject instead of two separate ones.
 ```

 When to avoid multiple inheritance:
 - prefer composition over inheritance
 - use interfaces (pure virtual classes) when multiple behaviors are required

 ---

 ## Polymorphism with smart pointers and ownership

 Use smart pointers (unique_ptr/shared_ptr) for polymorphic ownership. Be careful with custom deleters when using incomplete types and forward declarations.

 Example:

 ```cpp
 std::unique_ptr<Base> make_derived(){ return std::make_unique<Derived>(); }
 ```

 std::shared_ptr enables shared ownership but watch for cycles; prefer weak_ptr to break ownership cycles when observers or back-references are involved.

 ---

 ## Runtime type information: typeid and dynamic_cast

 Sometimes you need to recover derived type information at runtime.

 - dynamic_cast<T*>(p) safely downcasts and returns nullptr on failure (or throws if using references).
 - typeid gives static or dynamic type info depending on whether the operand is polymorphic.

 ```cpp
 Base* b = new Derived();
 if (auto d = dynamic_cast<Derived*>(b)) {
     // success
 }
 ```

 Prefer virtual interfaces and design that avoids downcasting where possible — downcasts signal design smells.

 ---

 ## Static polymorphism alternatives (CRTP) and when to choose it

 CRTP lets you get polymorphic-like syntax without virtual calls (no vtable) — useful for high-performance or compile-time customization.

 ```cpp
 template<typename Derived>
 struct Printable { void print(){ static_cast<Derived*>(this)->do_print(); } };

 struct Foo : Printable<Foo>{ void do_print(){ std::cout<<"Foo\n"; } };
 ```

 Tradeoffs:
 - CRTP requires that implementations are known at compile time.
 - No runtime substitution; cannot hold different types behind a single base pointer.

 ---

 ## Performance considerations and trade-offs

 - virtual calls have small overhead due to vtable indirection. For hot paths, prefer static polymorphism or inlining.
 - Additional memory for vtable pointer per polymorphic object.
 - dynamic_cast and RTTI have overhead and should be minimized in high-performance code.

 Guidance:
 - Use dynamic polymorphism when you need runtime flexibility.
 - Use static polymorphism (CRTP/templates) for performance-critical code where types are known.
 - Avoid heavy use of downcasts and RTTI.

 ---

 ## Small exercises & next steps

 1. Implement a mini plugin system using abstract base classes and factory registration.
 2. Convert a runtime polymorphic container to a CRTP-based static polymorphic alternative and benchmark the speed difference.
 3. Identify an example of object slicing in an existing codebase and refactor it to use smart pointers.

 ---

 [Back to Module 01 — Core Refresher](Module-01-Core-Refresher.md)
 [Next: Module 03 — Modern C++ Features](Module-03-Modern-CPP-Features.md)
