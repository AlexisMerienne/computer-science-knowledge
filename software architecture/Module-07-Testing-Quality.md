# Module 7 — Testing and Software Quality

Objective: Integrate testing and quality practices into the development process and build robust, verifiable C++ systems.

Navigation: [Prev: Module 6](Module-06-Performance-Optimization.md) • [Next: Module 8](Module-08-CaseStudies.md)

## 7.1 Unit testing with Google Test, Catch2 and test doubles

Unit tests verify small units of behavior. Google Test and Catch2 are popular frameworks; Google Test is widely used in larger C++ projects and integrates well with GoogleMock for test doubles.

Example: Google Test + Google Mock fixture

```cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>

class IStorage { public: virtual ~IStorage() = default; virtual bool write(const std::string&) = 0; };
class MockStorage : public IStorage { public: MOCK_METHOD(bool, write, (const std::string&), (override)); };

class Uploader { IStorage& s; public: Uploader(IStorage& st): s(st) {} bool upload(const std::string &payload){ return s.write(payload); } };

TEST(UploaderTest, UploadsPayload) {
  MockStorage mock;
  EXPECT_CALL(mock, write(::testing::HasSubstr("event"))).WillOnce(::testing::Return(true));
  Uploader u(mock);
  ASSERT_TRUE(u.upload("event data"));
}
```

Testing patterns:
- Use fixtures to share test scaffolding and avoid duplication.
- Use mocks sparingly — prefer lightweight fakes for integration where possible.
- Test-driven design (TDD) helps define clear responsibilities and small interfaces.

## 7.2 Integration and system testing

Integration tests verify that components collaborate correctly (network, DB, messaging), while system tests validate the entire application in environment-like conditions. For reliability, keep integration tests short and run them on PRs; schedule longer system tests nightly or in a dedicated environment to avoid flakiness in fast CI loops.

Best practices and patterns:
- Use containerized dependencies (Docker, Testcontainers) so the test environment matches production components.
- Consumer-driven contract tests ensure services evolve without breaking clients maintained by other teams.
- Golden/fixture-based tests or deterministic replay can validate complex output and avoid flaky timing issues.

Tooling:
- Test harnesses that provision and teardown dependencies programmatically (e.g., ephemeral databases or mock servers).
- Use environment-specific test configs to run more thorough system tests in isolated staging environments.

## 7.3 Code reviews, static analysis and runtime checks

Automated checks reduce the cognitive load during code reviews and let reviewers focus on design, correctness and architectural decisions.

Suggested toolchain and practices:
- `clang-tidy` for modernization and detection of common mistakes.
- `clang-format` for consistent style (enforce in pre-commit hooks or CI to keep diffs clean).
- `cppcheck` and other specialized static analyzers for defensive checks.
- Sanitizers (ASan/UBSan/MSan) for catching memory and undefined-behavior during tests.

Integration tips:
- Run quick checks locally with pre-commit hooks and run heavier checks in CI.
- Treat failures from sanitizers or critical static checks as blocking in CI to avoid shipping memory/UB issues.

## 7.4 Continuous Integration & release pipelines for C++

CI must prioritize reproducible builds and cross-platform coverage for C++ projects.

Recommended CI model:

- PR jobs (fast): build the project with minimal dependencies, run unit tests, run clang-tidy and format checks, run sanitizer-enabled unit tests.
- Merge gate (medium): include short integration tests, basic packaging validation and dependency checks.
- Nightly or scheduled jobs (heavy): full cross-platform builds, full integration test suites, fuzzing campaign runs and code-coverage analysis.

Practical knobs:

- Use container images or pinned toolchain manifests to ensure reproducible builds.
- Cache dependencies and CMake outputs (or use `ccache`) in CI to speed builds.
- Prefer splitting jobs in CI: fast pre-merge tests keep PR turnaround low; heavier, high-coverage validations run asynchronously.

Security and quality checks:

- Run sanitizers as part of CI (AddressSanitizer, UndefinedBehaviorSanitizer) to catch memory errors and undefined behavior early.
- Integrate fuzzing for parsers and input handlers; store minimized reproducers in artifacts for triage.
- Regularly run dependency scans and produce an SBOM for release artifacts.

Final note: automated CI + targeted nightly checks (fuzzing, integration tests, coverage) dramatically reduce regression risk in production-grade C++ systems.
