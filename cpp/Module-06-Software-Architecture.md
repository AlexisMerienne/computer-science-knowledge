 # Module 06 — Software Architecture in C++

 Objective: Review architectural principles and apply them to C++ projects.

 Contents

 - [6.1 SOLID principles in C++](#61-solid-principles-in-c)
 - [6.2 Layered architecture and dependency injection](#62-layered-architecture-and-dependency-injection)
 - [6.3 Component-based design and modularity](#63-component-based-design-and-modularity)
 - [6.4 Event-driven and reactive architectures in C++](#64-event-driven-and-reactive-architectures-in-c)

 ---

 ## 6.1 SOLID principles in C++

 - Single Responsibility: each class/module has one reason to change.
 - Open/Closed: prefer extension over modification (polymorphism, templates).
 - Liskov Substitution: derived classes should behave like their base.
 - Interface Segregation: small, specific interfaces.
 - Dependency Inversion: depend on abstractions, not concretions.

 Example — Dependency inversion using constructor injection

 ```cpp
 #include <memory>
 #include <iostream>

 struct IDataSource { virtual int read() = 0; virtual ~IDataSource() = default; };
 struct FileSource : IDataSource { int read() override { return 42; } };

 struct Processor {
     std::shared_ptr<IDataSource> ds;
     Processor(std::shared_ptr<IDataSource> ds_) : ds(std::move(ds_)) {}
     int compute(){ return ds->read() * 2; }
 };

 int main(){
     auto p = Processor(std::make_shared<FileSource>());
     std::cout << p.compute() << '\n';
 }
 ```

 ---
 ## 6.2 Layered architecture and dependency injection

 - Layered design: separate presentation, domain, persistence layers.
 - Use DI (dependency injection) to provide implementations at composition-time, aiding testing and decoupling.
 - Consider service-locators or DI containers sparingly; prefer explicit constructor injection for clarity.

 ---
 ## 6.3 Component-based design and modularity

 - Break system into small, testable components with well-defined interfaces.
 - Use C++ modules (C++20) or clean header/source boundaries to reduce compile times and create clearer contracts.
 - Prefer proper encapsulation and minimize global state.

 ---
 ## 6.4 Event-driven and reactive architectures in C++

 - Event-driven systems are useful for decoupling producers and consumers.
 - Libraries: Boost.Asio for I/O, RxCpp for reactive streams, or custom event-dispatchers.

 Example — simple dispatcher

 ```cpp
 #include <functional>
 #include <vector>

 struct EventBus{
     std::vector<std::function<void(int)>> subs;
     void subscribe(std::function<void(int)> f){ subs.push_back(std::move(f)); }
     void publish(int x){ for(auto &f : subs) f(x); }
 };
 ```

 ---

 Continue to Module 07 — Performance Optimization for profiling and concurrency best practices.

 [Back to Overview](00-Overview.md)
 [Prev: Module 05 — Design Patterns in C++](Module-05-Design-Patterns.md)
 [Next: Module 07 — Performance Optimization](Module-07-Performance-Optimization.md)
