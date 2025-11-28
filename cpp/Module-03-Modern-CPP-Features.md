 # Module 03 — Modern C++ Features

 Objective: Master modern C++ features and idioms (C++11/14/17/20/23).

 Contents

 - [3.1 Smart pointers](#31-smart-pointers)
 - [3.2 Move semantics](#32-move-semantics)
 - [3.3 STL containers and algorithms](#33-stl-containers-and-algorithms)
 - [3.4 constexpr, noexcept, and compile-time programming](#34-constexpr-noexcept-and-compile-time-programming)
 - [3.5 Variadic templates and type traits](#35-variadic-templates-and-type-traits)

 ---

 ## 3.1 Smart pointers (std::unique_ptr, std::shared_ptr, std::weak_ptr)

 - std::unique_ptr: exclusive ownership, cannot be copied but movable.
 - std::shared_ptr: shared ownership with reference counting.
 - std::weak_ptr: non-owning observer to a shared_ptr-managed object — helps break cycles.

 Example — using unique_ptr and shared_ptr:

 ```cpp
 #include <memory>
 #include <vector>
 #include <iostream>

 struct Node {
     int value;
     std::unique_ptr<Node> child; // unique ownership
     explicit Node(int v):value(v){}
 };

 int main(){
     auto root = std::make_unique<Node>(1);
     root->child = std::make_unique<Node>(2);
     std::cout << root->child->value << '\n';

     // shared ownership
     auto sp1 = std::make_shared<int>(42);
     auto sp2 = sp1; // two owners
     std::cout << "use_count: " << sp1.use_count() << '\n';
 }
 ```

 ---
 ## 3.2 Move semantics (rvalue references, std::move, move constructors/assignment)

 - Moves transfer ownership/contents cheaply avoiding copies.
 - Define move ctor & move assignment to make containers/handles efficiently transferable.

 Example — move-aware string wrapper:

 ```cpp
 #include <string>
 #include <utility>

 struct Holder {
     std::string data;
     Holder(std::string s) : data(std::move(s)) {}
     Holder(Holder&& other) noexcept = default;            // defaulted move
     Holder& operator=(Holder&& other) noexcept = default; // defaulted move assign
 };
 ```

 ---
 ## 3.3 STL containers and algorithms (focus on performance and usage)

 - Choose the right container (vector, deque, list, set, unordered_set, map, unordered_map).
 - Use algorithms from <algorithm> when possible: std::sort, std::find_if, std::transform.
 - Prefer std::vector for cache locality unless insertion/deletion in middle matters.

 Example: algorithm + reserving capacity

 ```cpp
 #include <vector>
 #include <algorithm>
 #include <iostream>

 int main(){
     std::vector<int> v;
     v.reserve(1000); // avoid reallocation

     for (int i=0;i<1000;++i) v.push_back(i);

     std::sort(v.begin(), v.end(), std::greater<>());
     auto it = std::find_if(v.begin(), v.end(), [](int x){ return x % 17 == 0; });
     if (it != v.end()) std::cout << *it << '\n';
 }
 ```

 ---
 ## 3.4 constexpr, noexcept, and compile-time programming

 - constexpr enables compile-time evaluation; use it for small utilities and computing constants.
 - noexcept is a promise that a function does not throw — helps optimizations and strong exception-safety guarantees.

 Example — constexpr function and static_assert checks

 ```cpp
 #include <type_traits>

 constexpr int factorial(int n){
     return (n <= 1) ? 1 : (n * factorial(n-1));
 }

 static_assert(factorial(5) == 120);

 int main(){
     constexpr auto f5 = factorial(5);
     (void)f5;
 }
 ```

 ---
 ## 3.5 Variadic templates and type traits

 - Variadic templates are flexible ways to write type- and argument-generic code (e.g., tuple, print functions).
 - Use type traits (<type_traits>) to write conditional code at compile time.

 Example — simple variadic print utility:

 ```cpp
 #include <iostream>

 void print() { std::cout << "\n"; }

 template<typename T, typename... Ts>
 void print(T&& value, Ts&&... rest){
     std::cout << value << ' ';
     print(std::forward<Ts>(rest)...);
 }

 int main(){
     print("Numbers:", 1, 2.5, 'c');
 }
 ```

 ---

 Continue to Module 04 — Memory Management & RAII for ownership rules and custom allocators.

 [Back to Overview](00-Overview.md)
 [Prev: Module 02 — Polymorphism in C++](Module-02-Polymorphism.md)
 [Next: Module 04 — Memory Management and RAII](Module-04-Memory-Management-RAII.md)
