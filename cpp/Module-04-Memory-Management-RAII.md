 # Module 04 — Memory Management and RAII

 Objective: Deepen understanding of memory management and RAII (Resource Acquisition Is Initialization).

 Contents

 - [4.1 Manual memory management pitfalls and best practices](#41-manual-memory-management-pitfalls-and-best-practices)
 - [4.2 RAII: principles, examples, and custom RAII wrappers](#42-raii-principles-examples-and-custom-rai-wrappers)
 - [4.3 Smart pointers in depth: ownership semantics and cycles](#43-smart-pointers-in-depth-ownership-semantics-and-cycles)
 - [4.4 Custom allocators and memory pools](#44-custom-allocators-and-memory-pools)

 ---

 ## 4.1 Manual memory management pitfalls and best practices

 - Manual new/delete is error-prone: leaks, double-free, use-after-free.
 - Avoid manual memory management in modern C++ unless implementing low-level resources.

 Bad example: use-after-free

 ```cpp
 #include <iostream>

 int* leak() {
     int* p = new int(42);
     delete p;
     return p; // use-after-free
 }

 int main(){
     int* p = leak();
     std::cout << *p << '\n'; // undefined behavior
 }
 ```

 ---
 ## 4.2 RAII: principles, examples, and custom RAII wrappers

 - RAII binds resource lifetime to object lifetime. Acquire in ctor, release in dtor.
 - Use existing RAII types (std::unique_ptr, std::lock_guard) when possible.

 Example — RAII for a file handle:

 ```cpp
 #include <cstdio>
 #include <stdexcept>

 struct FileRAII {
     FILE* f;
     FileRAII(const char* path) : f(std::fopen(path, "r")) {
         if (!f) throw std::runtime_error("can't open file");
     }
     ~FileRAII() { if (f) std::fclose(f); }
     FILE* get() const { return f; }
 };

 int main(){
     try {
         FileRAII fr("example.txt");
         // use fr.get()
     } catch(...) {
         // handle
     }
 }
 ```

 ---
 ## 4.3 Smart pointers in depth: ownership semantics and cycles

 - unique_ptr: single owner; shared_ptr: reference-counted owner; weak_ptr: non-owning observer.
 - Cyclic references with shared_ptr cause leaks; break cycles with weak_ptr.

 Example — cycle and breaking it with weak_ptr:

 ```cpp
 #include <memory>
 #include <iostream>

 struct A; struct B;

 struct A { std::shared_ptr<B> b; ~A(){ std::cout<<"A dtor\n"; } };
 struct B { std::shared_ptr<A> a; ~B(){ std::cout<<"B dtor\n"; } };

 int main(){
     {
         auto a = std::make_shared<A>();
         auto b = std::make_shared<B>();
         a->b = b;
         b->a = a; // cycle: neither destructor runs when leaving scope
     }
     std::cout << "end scope\n";

     // break cycle example
     {
         struct A2{ std::weak_ptr<B> b; ~A2(){ std::cout<<"A2 dtor\n"; } };
         struct B2{ std::shared_ptr<A2> a; ~B2(){ std::cout<<"B2 dtor\n";} };
         auto a = std::make_shared<A2>();
         auto b = std::make_shared<B2>();
         a->b = b; // no strong cycle
         b->a = a;
     }
     std::cout<<"end2\n";
 }
 ```

 ---
 ## 4.4 Custom allocators and memory pools

 - Custom allocators are advanced but useful in high-performance or embedded contexts.
 - Consider object pools for frequently allocated/deallocated small objects.

 Example sketch — simple object pool (illustrative)

 ```cpp
 #include <vector>
 #include <cstdint>

 template<typename T>
 class SimplePool {
     std::vector<T*> free_list;
 public:
     ~SimplePool(){ for (auto p : free_list) delete p; }
     T* acquire(){ if (free_list.empty()) return new T(); T* p = free_list.back(); free_list.pop_back(); return p; }
     void release(T* p){ free_list.push_back(p); }
 };

 // usage: SimplePool<MyObject> pool; auto o = pool.acquire(); pool.release(o);
 ```

 ---

 Continue to Module 05 — Design Patterns in C++ for practical architectural patterns.

 [Back to Overview](00-Overview.md)
 [Prev: Module 03 — Modern C++ Features](Module-03-Modern-CPP-Features.md)
 [Next: Module 05 — Design Patterns in C++](Module-05-Design-Patterns.md)
