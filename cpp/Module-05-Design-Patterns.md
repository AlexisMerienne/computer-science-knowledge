 # Module 05 — Design Patterns in C++

 Objective: Review and apply classic design patterns in C++ using modern idioms.

 Contents

 - [5.1 Creational patterns: Singleton, Factory, Builder](#51-creational-patterns-singleton-factory-builder)
 - [5.2 Structural patterns: Adapter, Decorator, Facade](#52-structural-patterns-adapter-decorator-facade)
 - [5.3 Behavioral patterns: Observer, Strategy, Command](#53-behavioral-patterns-observer-strategy-command)
 - [5.4 Modern C++ idioms: CRTP, Type Erasure, Policy-Based Design](#54-modern-c-idioms-crtp-type-erasure-policy-based-design)

 ---

 ## 5.1 Creational patterns: Singleton, Factory, Builder

 Singleton: prefer dependency injection over singletons; if needed, use Meyers' singleton for thread-safe, lazy init.

 ```cpp
 // Meyers' singleton
 struct Logger {
     static Logger& instance(){ static Logger i; return i; }
     void log(const char* m){}
 };
 ```

 Factory: hide creation details behind an interface.

 ```cpp
 #include <memory>

 struct Product { virtual void op() = 0; virtual ~Product()=default; };
 struct ConcreteA : Product { void op() override {} };
 struct ConcreteB : Product { void op() override {} };

 std::unique_ptr<Product> makeProduct(int type){
     if (type==1) return std::make_unique<ConcreteA>();
     return std::make_unique<ConcreteB>();
 }
 ```

 Builder: useful when constructing complex objects step-by-step.

 ---
 ## 5.2 Structural patterns: Adapter, Decorator, Facade

 Adapter: adapt incompatible interfaces through a thin wrapper.
 Decorator: wrap to add behavior (e.g., logging).
 Facade: simplified API exposing only necessary operations.

 Example — Decorator with stream-like interface:

 ```cpp
 #include <iostream>
 #include <memory>

 struct Writer{ virtual void write(const std::string& s)=0; virtual ~Writer()=default; };
 struct ConsoleWriter : Writer { void write(const std::string& s) override { std::cout << s << '\n'; } };

 struct LoggingWriter : Writer {
     std::unique_ptr<Writer> inner;
     LoggingWriter(std::unique_ptr<Writer> w): inner(std::move(w)){}
     void write(const std::string& s) override {
         std::cout << "LOG:";
         inner->write(s);
     }
 };
 ```

 ---
 ## 5.3 Behavioral patterns: Observer, Strategy, Command

 Observer: publish-subscribe semantics with careful lifetime management (weak_ptr to avoid dangling observers).
 Strategy: swap algorithms by encapsulating behavior in strategy objects.
 Command: encapsulate operations as objects to queue, log, or undo.

 Example — Observer with weak_ptr:

 ```cpp
 #include <vector>
 #include <memory>
 #include <algorithm>

 struct IObserver{ virtual void notify(int) = 0; virtual ~IObserver()=default; };

 struct Subject{
     std::vector<std::weak_ptr<IObserver>> obs;
     void subscribe(std::shared_ptr<IObserver> o){ obs.push_back(o); }
     void notifyAll(int v){
         for (auto it = obs.begin(); it!=obs.end(); ){
             if (auto s = it->lock()) { s->notify(v); ++it; }
             else it = obs.erase(it);
         }
     }
 };
 ```

 ---
 ## 5.4 Modern C++ idioms: CRTP, Type Erasure, Policy-Based Design

 CRTP (Curiously Recurring Template Pattern) enables static polymorphism.
 Type erasure (e.g., std::function, small-object optimizations) removes compile-time type dependence.
 Policy-based design allows behavior injection via templates.

 Short example — CRTP for static polymorphism:

 ```cpp
 template<typename Derived>
 struct Base {
     void interface(){ static_cast<Derived*>(this)->implementation(); }
 };

 struct Impl : Base<Impl> { void implementation(){ /* ... */ } };
 ```

 ---

 Continue to Module 06 — Software Architecture in C++ to apply patterns to larger projects.

 [Back to Overview](00-Overview.md)
 [Prev: Module 04 — Memory Management & RAII](Module-04-Memory-Management-RAII.md)
 [Next: Module 06 — Software Architecture in C++](Module-06-Software-Architecture.md)
