# Module 2 — SOLID Principles and C++ Best Practices

Objective: Apply SOLID principles and C++ best practices to design and implement components that are robust, extensible, testable and maintainable in real projects.

Navigation: [Prev: Module 1](Module-01-Introduction.md) • [Next: Module 3](Module-03-Design-Patterns.md)

## 2.1 Single Responsibility Principle (SRP)

SRP says a class (or module) should have only one reason to change: its responsibilities should be narrowly focused. In large codebases this keeps the blast radius of changes small, reduces merge conflicts, and improves testability.

Real-life example: logging vs business logic

- Problem: a utility method that both performs a domain action and writes a log file mixes responsibilities. Changes to logging (format, destination) should not force edits to business rules.
- Solution: extract logging into a separate `Logger` interface and keep the core method focused on the domain behavior. This allows the logging policy to evolve independently.

Practical C++ example (refactor):

```cpp
// BAD: Database access + formatting inside OrderProcessor
class OrderProcessorBad {
    void process(Order &o){ /* validate, persist to DB, write report to disk */ }
};

// GOOD: single responsibility split
struct IOrderStore { virtual ~IOrderStore() = default; virtual void save(const Order&) = 0; };
class OrderValidator { public: bool validate(const Order&) noexcept; };
class OrderProcessor { IOrderStore &store; OrderValidator validator; public: void process(const Order &o) { if(!validator.validate(o)) throw; store.save(o); } };
```

## 2.2 Open/Closed Principle (OCP)

OCP means core modules should allow extension with new behaviors without requiring changes to tested, stable code. This is often realized via interfaces (virtual functions) or via templates (compile-time extension) in C++.

Real-life example: plugin-able data processors

- Problem: a data pipeline hardcodes parsing and transformation steps; adding a new transformation requires modifying core code.
- Solution: define an abstract `Transformer` interface and register implementations (plugins). New transformations are added by implementing the interface and registering them; the core pipeline remains unchanged.

Example (polymorphism) — a typical factory + registry approach:

```cpp
struct Transform { virtual ~Transform() = default; virtual void run(Data&) = 0; };
// new classes implement Transform; the pipeline obtains them via factory or registration
```

## 2.3 Liskov Substitution Principle (LSP)

LSP requires subtypes to be substitutable for their base types without breaking client expectations. Violations create brittle designs: clients that expect the base contract can be surprised by changed behavior in a subtype.

Real-life example — file abstractions and constraints

- Problem: A base type `Stream` allows `seek()` and `read()`, but a derived `NetworkStream` does not support seeking and throws. If client code relies on calling `seek()` when present on `Stream`, substituting a `NetworkStream` breaks behavior.
- Solution: make the contract explicit and avoid exposing unsupported operations on the base type (either split interfaces or document non-supported operations clearly and provide checks at call sites).

Classic pedagogical example (rectangles vs squares):

```cpp
// BAD: making Square inherit Rectangle can break expectations if Rectangle exposes setWidth/setHeight
```
Derived classes must be substitutable for their base classes. Avoid breaking contracts: pre/post conditions should not be violated.

Example: Don't make a Derived class throw new exceptions that break caller expectations.

## 2.4 Interface Segregation Principle (ISP)

ISP advises creating focused interfaces so clients depend only on the members they need. Large interfaces force implementers to provide unnecessary operations and make mocking and testing harder.

Real-life example — IO subsystem

- Problem: a single `IStorage` interface which includes `read`, `write`, `truncate`, `resize`, `stat`. A read-only cache implementation then must stub or throw for write operations.
- Solution: split into `IReadable` and `IWritable` and smaller concerns so implementations depend only on the operations they support.

Example in C++:

```cpp
struct IReadable { virtual std::string read(size_t offset, size_t length) = 0; virtual ~IReadable() = default; };
struct IWritable { virtual void write(size_t offset, std::string_view data) = 0; virtual ~IWritable() = default; };
```

## 2.5 Dependency Inversion Principle (DIP)

DIP promotes depending on abstractions rather than concrete types. High-level modules declare dependencies in terms of interfaces, and composition roots wire concrete implementations. This makes it easy to swap modules for testing or to introduce new concrete implementations.

Real-life example — payment gateway integration

- Problem: business logic directly instantiates and calls a vendor SDK. If the vendor changes API, many files must be edited.
- Solution: define an abstract `IPaymentGateway` interface and inject a concrete vendor adapter at startup — tests can substitute a stub. The production code never depends on vendor-specific types.

Example (C++): constructor injection

```cpp
class IPaymentGateway { public: virtual ~IPaymentGateway() = default; virtual bool charge(double amount) = 0; };
class StripeAdapter : public IPaymentGateway { /* wraps Stripe SDK calls */ };
class CheckoutService { IPaymentGateway &gateway; public: CheckoutService(IPaymentGateway &g): gateway(g) {} bool chargeCustomer(...) { return gateway.charge(...); } };
```

## 2.6 C++ best practices

Key C++ practices that align well with architecture goals:

- RAII (Resource Acquisition Is Initialization): ties lifetime of resources to object scope so you avoid leaks and cleanup code. Example: file handles, socket descriptors, locks.
- Smart pointers: prefer `std::unique_ptr` for exclusive ownership and `std::shared_ptr` when ownership must be shared. Use `std::weak_ptr` to avoid cycles.
- const-correctness: marking const on methods and parameters improves reasoning about side-effects and enables compiler checks.
- Prefer value semantics and moves for small-to-medium sized objects; avoid unnecessary virtual dispatch where performance is critical.
- Use exceptions or outcome types consistently for error propagation — prefer RAII + exceptions for C++ applications with clear rollback semantics.

Concrete industry examples:

- RAII in networking: a `Socket` class acquires a descriptor and closes it in the destructor; this prevents fd leaks in high-throughput services.
- Smart pointers in plugin systems: `std::shared_ptr` may be used to share plugin state between the loader and active components; use `weak_ptr` in observers to avoid preventing plugin unload.

Example of strong RAII pattern:

```cpp
#include <memory>
#include <cstdio>

struct FileHandle {
    std::unique_ptr<FILE, int(*)(FILE*)> f;
    FileHandle(const char* path) : f(std::fopen(path, "w"), &std::fclose) {}
};

// FileHandle fh("out.txt");  // closed automatically at end of scope
```

---

Concrete mapping: For each SOLID principle in a C++ system, identify a place in your codebase where violating it has caused problems (e.g., QA bugs, regressions, or hard-to-test code) and document how applying the principle would eliminate that problem.
