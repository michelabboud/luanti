# ADR-0001: C++17 as engine language standard

Date: 2019-01-01
Status: Accepted

## Context

Luanti is a large C++ codebase (~200K LOC). As the C++ standard evolved, the engine
needed to commit to a minimum standard version that all supported compilers could handle.
Key considerations:

- C++14 was the previous baseline; C++17 introduced significant quality-of-life features
- The engine targets a wide range of platforms (Linux, Windows, macOS, Android, BSD)
- The oldest commonly-requested compiler support was GCC 7.5 (Ubuntu 18.04 LTS) and
  Clang 7.0.1 — both of which shipped with substantial C++17 support
- C++20 modules and concepts were not yet stable enough across all targets at the time

## Decision

Adopt C++17 as the minimum required language standard for the engine. Minimum compiler
versions are GCC 7.5+ or Clang 7.0.1+. CMake target `CMAKE_CXX_STANDARD 17` is set in
the root `CMakeLists.txt`.

## Consequences

**Positive:**
- `std::optional<T>` replaces error-prone sentinel value patterns
- Structured bindings (`auto [k, v] = ...`) improve readability
- `if constexpr` allows cleaner template specialization
- `std::string_view` enables zero-copy string passing
- Parallel STL algorithms (`std::execution::par`) available for performance-critical paths
- `[[nodiscard]]` / `[[maybe_unused]]` attributes improve compile-time checks

**Negative / trade-offs:**
- Drops support for GCC < 7.5 and Clang < 7.0.1 (affects very old distros like Ubuntu 16.04)
- C++17 standard library support on some older Android NDKs required careful feature-testing

**Neutral / notes:**
- clang-tidy enforces modernize-* rules to ensure C++17 idioms are used consistently;
  see `.clang-tidy` and `util/ci/clang-tidy.sh`
- C++20 adoption will be a separate decision when compiler coverage is sufficient
