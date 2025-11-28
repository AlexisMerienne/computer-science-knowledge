# Module 3 — Design Patterns in C++

Objective: Master classic design patterns, understand when to apply them in real systems, and learn idiomatic C++ implementations and their trade-offs.

Navigation: [Prev: Module 2](Module-02-SOLID-Cpp-BestPractices.md) • [Next: Module 4](Module-04-Common-Architectures.md)

## 3.1 Creational patterns

Creational patterns control how objects are created so the rest of the system can remain decoupled from concrete types. In C++ the same pattern can be implemented using polymorphism (run-time flexibility) or templates (compile-time flexibility and zero-cost abstractions).

Singleton — a global single instance. Use sparingly: causes hidden coupling and testability issues. In C++ prefer controlled singletons with explicit lifecycle and thread-safe initialization (e.g., Meyers' Singleton using function scope statics).

Real-world use: a process-wide configuration manager used by subsystems that must share read-only configuration at runtime.

Factory Method / Abstract Factory — decouple creation from usage. Useful when product families vary across platforms (different database backends, OS-specific implementations).

Builder — helpful for constructing complex objects (e.g., constructing an HTTP request, or a parsed AST). In C++ builders often return move-only types to avoid copies.

Practical C++ example (Factory + Plugin registry for a data parser):

```cpp
// Product: parser interface
struct Parser { virtual ~Parser() = default; virtual void parse(const std::string &input) = 0; };

// Registry & factory
using ParserFactory = std::function<std::unique_ptr<Parser>()>;
class ParserRegistry {
    static std::map<std::string, ParserFactory>& regs() {
        static std::map<std::string, ParserFactory> r; return r;
    }
public:
    static void registerFactory(const std::string &name, ParserFactory f) { regs()[name] = std::move(f); }
    static std::unique_ptr<Parser> make(const std::string &name) {
        auto it = regs().find(name);
        return it != regs().end() ? it->second() : nullptr;
    }
};

// Plugins can call ParserRegistry::registerFactory at load time.
```

Notes for C++: use unique_ptr to enforce ownership, avoid raw pointers, and consider ABI/loader boundaries if plugins are implemented as shared libraries.

## 3.2 Structural patterns

Structural patterns manage relationships between objects to build larger structures while preserving encapsulation and flexibility.

Adapter — useful when adapting legacy libraries or third-party APIs with an incompatible interface.

Real-life use: Adapting a blocking I/O storage SDK behind an asynchronous interface in a new event loop.

Decorator — wrap objects to extend behavior without changing the original class. In C++ use composition and careful ownership (unique_ptr) to chain decorators.

Facade — provide a simplified entry-point to complex subsystems (e.g., a single `DatabaseClient` façade for query, transaction and metrics orchestration).

Composite — represents hierarchical structures (GUI widgets, scene graphs, filesystem trees).

Adapter example with modern C++ memory-safety considerations:

```cpp
class LegacyApi { public: int oldCall(int x); };
struct INew { virtual int calc(int) = 0; virtual ~INew() = default; };
class LegacyAdapter : public INew {
    std::shared_ptr<LegacyApi> legacy; // shared ownership across adapters
public:
    LegacyAdapter(std::shared_ptr<LegacyApi> l): legacy(std::move(l)){}
    int calc(int x) override { return legacy->oldCall(x); }
};
```

Decorator (logging wrapper for an interface):

```cpp
struct IService { virtual ~IService() = default; virtual void op() = 0; };
class LoggingService : public IService {
    std::unique_ptr<IService> inner;
public:
    explicit LoggingService(std::unique_ptr<IService> i): inner(std::move(i)){}
    void op() override { /* log */ inner->op(); /* log */ }
};
```

## 3.3 Behavioral patterns

Behavioral patterns deal with communication and responsibility between objects and are often key to building flexible, testable systems.

Observer — a publish/subscribe mechanism. Real-world use: UI event propagation, telemetry/event buses, or plugin events. In C++ it's important to manage listener lifetimes; prefer weak references or connection tokens to avoid dangling callbacks.

Strategy — select algorithm at runtime (e.g., sorting, compression). Use polymorphism for runtime flexibility or templates for compile-time performance.

Command — represent actions as objects (useful for undo/redo, queuing, macro-recording). Commands can carry serialization support for persistence or distributed execution.

State — model state-dependent behavior (e.g., protocol parsers, UI components). Use the State pattern to avoid large switch statements and localize state transitions.

Observer example with lifetime safety (weak subscription):

```cpp
#include <functional>
#include <vector>
#include <memory>

class Broadcaster;
class Listener : public std::enable_shared_from_this<Listener> {
public:
    void onEvent(int v) { /* handle event */ }
    void subscribe(Broadcaster &b);
};

class Broadcaster {
    std::vector<std::weak_ptr<Listener>> listeners;
public:
    void subscribe(std::shared_ptr<Listener> l) { listeners.push_back(l); }
    void publish(int x){
        // prune dead listeners while notifying
        std::vector<std::weak_ptr<Listener>> alive;
        for(auto &w: listeners){ if(auto s = w.lock()){ s->onEvent(x); alive.push_back(w); } }
        listeners.swap(alive);
    }
};

// Usage: auto l = std::make_shared<Listener>(); b.subscribe(l); b.publish(7);
```

Command pattern example (undo/redo):

```cpp
struct ICommand { virtual ~ICommand() = default; virtual void execute() = 0; virtual void undo() = 0; };
class InsertText : public ICommand {
    Document &doc; size_t pos; std::string text;
public:
    InsertText(Document &d, size_t p, std::string t): doc(d), pos(p), text(std::move(t)) {}
    void execute() override { doc.insert(pos, text); }
    void undo() override { doc.erase(pos, text.size()); }
};
```

## 3.4 Case studies: implementing patterns in real-world projects

When applying patterns, choose the one that addresses the underlying problem rather than applying a pattern for its own sake. Patterns should reduce complexity and improve clarity.

Real-world case study — extensible media pipeline (audio/video transcoder):

- Requirements: support multiple input formats, modular chain of transforms (decode → filters → encode), and runtime-loaded codec plugins.
- Architecture: use a Factory/Registry to load decoders and encoders; apply a Chain of Responsibility / Strategy for transform steps; use Decorators to add metrics or caching layers around decode/encode stages.

Real-world case study — GUI framework widgets:

- Widgets are naturally represented with Composite (container components) + Observer (event propagation). Adapters wrap OS-specific windowing backends, and Facades can expose a simple rendering API that hides shaders, buffers, and platform differences.

Trade-offs and C++ specifics:

- Memory & ownership: prefer smart pointers and clear ownership semantics across boundaries (e.g., plugin loader vs runtime).
- Performance: avoid excessive virtual dispatch in hot loops; prefer templates or value-based strategies where relevant.
- Error handling: patterns interacting with I/O or OS boundaries should consider fallible constructors, explicit initialization, and strong RAII to avoid resource leaks.

---

Further reading: "Design Patterns" (Gamma et al.), "Modern C++ Design" (Andrei Alexandrescu), and code examples from production-grade systems (Qt, Chromium) for how patterns are adapted at scale.
