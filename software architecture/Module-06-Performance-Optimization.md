# Module 6 — Performance and Optimization

Objective: Optimize C++ application performance while preserving maintainability and architectural integrity.

Navigation: [Prev: Module 5](Module-05-Dependency-Modularity.md) • [Next: Module 7](Module-07-Testing-Quality.md)

## 6.1 Profiling and identifying bottlenecks

Good performance work starts with profiling. Use both sampling profilers (low overhead, good for hotspots) and instrumentation (detailed traces) depending on the problem.

Tools & techniques:
- Linux: perf, `perf record` + `perf report`, flamegraphs (FlameGraph)
- Windows: Visual Studio Profiler, Windows Performance Recorder / Analyzer
- macOS: Instruments
- Cross-platform: Intel VTune, async-profiler, heap profilers for allocation hotspots

Approach:
- Measure first — capture representative production-like workloads
- Identify hotspots (CPU), allocation pressure (heap profilers), and I/O waits
- Avoid premature optimization: fix algorithmic issues before micro-optimizations

## 6.2 Code optimization techniques

Optimization techniques must be applied with care and measured. A clean architecture helps by isolating bottlenecks so you can apply targeted optimizations without rewriting the whole codebase.

- Inlining: reduce call overhead for trivial methods; rely on compiler heuristics and profile-guided hints for hot paths.
- Caching / memoization: remove repeated computation but keep invalidation logic simple.
- Parallelization: use thread pools, work-stealing task systems (TBB, Folly, custom executors), or C++17 parallel algorithms where appropriate. Consider contention and cache effects.

Real-world case — web server critical path:

- Hot path: request parsing, routing, and serialization. Keep common paths minimal and move expensive checks off the hot path.
- Use per-thread caches and pooled buffers to minimize contention.

## 6.3 Efficient use of the STL

Make conscious container choices — `std::vector` is usually best because of cache locality. Avoid `std::list` in performance-critical areas because it’s cache-unfriendly.

Guidelines:
- Use `reserve()` for vectors where size is predictable.
- Use `emplace_back()` to construct in-place and avoid temporary copies.
- Prefer algorithms from `<algorithm>` to express intent and enable compiler optimizations.

Practical example — cache-friendly layout for entity systems:

- Prefer Structure of Arrays (SoA) when iterating over specific fields for better cache performance.

## 6.4 Memory management and optimization

- Reduce fragmentation and allocation churn with object pools, slab allocators, or arena allocators for short-lived, high-volume objects.
- Use `std::pmr` (polymorphic memory resources) to switch allocation strategies without changing APIs.
- Move semantics and `emplace` help avoid unnecessary copies in performance-critical paths.

Advanced topics — low-latency systems:

- Lock-free structures can avoid blocking latency but are complex and expensive to reason about; prefer well-tested libraries.
- Consider CPU affinity and cache topology (NUMA) in multi-socket systems to reduce cross-socket traffic.

Example: move semantics for efficiency

```cpp
struct Big { std::vector<int> data; };

Big makeBig(){ Big b; b.data.resize(10'000); return b; }

int main(){
    Big a = makeBig(); // move elision / move constructor
}
```

---

Always validate improvements using representative benchmarks and production-like workloads. Where possible, use automated benchmarks in CI to prevent regressions in hot code paths.
