 # Module 08 — Testing and Debugging

 Objective: Reinforce testing and debugging skills for C++ projects.

 Contents

 - [8.1 Unit testing frameworks (Google Test, Catch2)](#81-unit-testing-frameworks)
 - [8.2 Writing testable code: mocking and dependency injection](#82-writing-testable-code)
 - [8.3 Debugging techniques (GDB, LLDB, print debugging, static analysis)](#83-debugging-techniques)
 - [8.4 Continuous Integration (CI) for C++ projects](#84-continuous-integration-ci-for-c-projects)

 ---

 ## 8.1 Unit testing frameworks (Google Test, Catch2)

 - Google Test (gtest) and Catch2 are popular frameworks. Use parameterized and fixture-based tests for repeating patterns.

 Simple Google Test example (CMake integration):

 ```cpp
 // add a file mylib.cpp + mylib.h with functions to test
 // tests/test_mylib.cpp
 #include <gtest/gtest.h>
 #include "mylib.h"

 TEST(AddTests, Basic) {
     EXPECT_EQ(add(2,3), 5);
 }
 ```

 ---
 ## 8.2 Writing testable code: mocking and dependency injection

 - Use interfaces and DI to decouple and allow mocking.
 - Fake small components or use mocking frameworks (gmock) for interactions.

 ---
 ## 8.3 Debugging techniques (GDB, LLDB, print debugging, static analysis)

 - Learn how to use gdb/lldb for breakpoints, stack traces, and variable introspection.
 - Use sanitizers (AddressSanitizer, UndefinedBehaviorSanitizer) and static analysis (clang-tidy) to catch issues early.

 Run with AddressSanitizer for memory errors: compile with -fsanitize=address -g

 ---
 ## 8.4 Continuous Integration (CI) for C++ projects

 - Use CI (GitHub Actions, Azure Pipelines) for building, running tests, and static analysis on every commit.
 - Create reproducible builds with toolchains (e.g., Docker images) and matrix builds for multiple compilers/platforms.

 Example GitHub Actions step (conceptual):

 ```yaml
 # steps: checkout, install deps, build with cmake, run tests
 - name: Run tests
   run: cmake --build build --target test
 ```

 ---

 Continue to Module 09 — Practical Project where we'll design and implement a small real-world C++ application.

 [Back to Overview](00-Overview.md)
 [Prev: Module 07 — Performance Optimization](Module-07-Performance-Optimization.md)
 [Next: Module 09 — Practical Project](Module-09-Practical-Project.md)
