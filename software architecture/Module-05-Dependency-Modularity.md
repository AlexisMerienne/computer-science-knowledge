# Module 5 — Dependency Management and Modularity

Objective: Learn to structure C++ projects for maximum modularity and minimal dependencies, and practical C++/CMake techniques for maintaining large codebases.

Navigation: [Prev: Module 4](Module-04-Common-Architectures.md) • [Next: Module 6](Module-06-Performance-Optimization.md)


## 5.1 Coupling and cohesion

- Coupling — degree to which modules depend on each other. Lower coupling reduces the blast radius of changes and enables independent builds.
- Cohesion — how closely related responsibilities inside a module are. High cohesion improves understandability and testability.

Strategies to improve modularity:

- Define small stable interfaces and keep implementation details hidden behind the interface (pimpl or private headers).
- Use dependency injection and inversion of control at logical module boundaries so replacements are trivial.
- Minimize header inclusion in public headers; prefer forward declarations in headers and include implementation headers in .cpp files to reduce compile-time coupling.

Real-world examples:

- Large game engine (Unreal/Unity internals): split rendering, physics, audio and scripting systems in separate modules so graphics engineers can iterate without touching physics.
- High-performance database: isolate storage engine from query planner and client-facing API to allow independent tuning and scaling.


## 5.2 Using namespaces and libraries in C++

Namespaces prevent name collisions and group related functionality. Libraries (static/shared/header-only) are primitive modular units in C++ ecosystems and should be designed with stable public headers and clear versioning.

Considerations & patterns:

- API vs implementation headers: keep public headers minimal and place implementation details in internal headers; use header-only libraries for small utilities but prefer compiled libraries for large subsystems to avoid long compile times.
- Symbol visibility and ABI: when shipping shared libraries, manage symbol visibility (hidden by default) and maintain ABI compatibility when evolving public headers.

Real-world example — shared library boundaries in OSS projects:

- Folly / Boost adopt careful header organization and semantic versioning so downstream consumers can depend on stable interfaces.


## 5.3 Managing dependencies with CMake

CMake modern best practices use targets to declare public/private include directories and link dependencies so transitive requirements are explicit and maintainable.

Best practices:

- Use `target_include_directories(target PUBLIC|PRIVATE|INTERFACE)` and `target_link_libraries` to control transitive propagation.
- Prefer `target_compile_features` and `target_compile_options` to impose compile-time requirements on consumers.
- Avoid global variables like `include_directories()` or `add_definitions()` which leak settings to unrelated targets.

Example CMake layout for modular project:

```cmake
add_library(core STATIC src/core.cpp)
target_include_directories(core PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)

add_library(util STATIC src/util.cpp)
target_include_directories(util PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
target_link_libraries(util PUBLIC core)

add_executable(app src/main.cpp)
target_link_libraries(app PRIVATE util)
```

Advanced considerations:

- Use `FetchContent` or package managers (conan, vcpkg) to manage third-party dependencies with reproducible versions.
- Consider using interface libraries (header-only targets) for utilities that must remain zero-overhead.

Real-world pattern — staged API evolution:

- When evolving public headers, create an `include/myproj/v2/` namespace and migrate consumers gradually. Use deprecation macros to signal removal timelines.

If you want a tiny, practical scaffold showing a library, an executable, and tests wired via CMake, I can create a targeted example on request.


## 5.4 Introduction to generic programming and templates

Generic programming enables reusable, high-performance abstractions in C++. Use templates to express algorithms that are zero-cost at runtime but be mindful of compilation complexity and type safety.

Practical notes:

- Use concepts (C++20) to document type requirements and provide better diagnostics.
- Keep large template instantiations out of public headers where possible, or provide explicit instantiations to reduce compile costs.

Example: type-safe policy-based container (simplified)

```cpp
template<typename T, typename Alloc = std::allocator<T>>
class SmallBufferVector {
    // uses Alloc to control allocation behaviour for embedded-optimized containers
};
```

Real-world case — template tradeoffs in libraries:

- The C++ STL and Boost use templates heavily to provide generic containers and algorithms. However, large internal template code can significantly increase compile times for consumers. Libraries like Abseil balance header-only ergonomics with carefully designed precompiled components where necessary.

---

This module focused on practical ways to structure large C++ codebases, keep dependencies manageable, and use CMake to enforce separation and a clear dependency graph.

Summary and further reading:

- Consider applying these guidelines to an existing codebase and note the places where public interface reduction and header minimization would speed up builds and reduce coupling.
