# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Test Code Not

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Rust crate development and applies to all test code written in the project.

## Context

- The project uses Rust's built-in testing framework with inline test modules, as evidenced by 77 files across multiple crates including uv-keyring, uv-client, uv-cli, uv-normalize, uv-python, uv-logging, and uv-netrc
- Test code is co-located with implementation code using #[cfg(test)] conditional compilation, allowing tests to access private module internals while being excluded from production builds
- The pattern shows consistent usage across both library crates (src/lib.rs) and integration test suites (tests/it/), indicating a standardized testing approach throughout the codebase
- This testing strategy supports rapid iteration and refactoring by keeping tests close to the code they verify, reducing context switching for developers
- The high support count (77 files) and confidence (89.42%) indicate this is an established, well-adopted pattern rather than an experimental approach

## Problem Statement

Development teams need a consistent, maintainable approach to unit testing that balances test isolation with developer ergonomics, enables testing of private implementation details, and integrates seamlessly with the build system without adding complexity or increasing binary size in production builds.

## Decision

1. MUST_NOT: Test code MUST NOT be included in production builds or affect the size or performance of release binaries

## Policy Block

- MUST_NOT Test code MUST NOT be included in production builds or affect the size or performance of release binaries

In scope:
- All unit tests for library crates (lib.rs and module files)
- Tests that need access to private functions, structs, or module internals
- Tests that verify implementation details and internal logic
- Test helper functions and utilities specific to a single module

Out of scope:
- Integration tests that verify cross-crate or cross-module public API contracts
- End-to-end tests that require external resources or system-level setup
- Benchmark tests marked with #[bench]
- Documentation tests embedded in doc comments (/// or //!)

Exceptions:
- EXC-001: Integration tests require testing public API contracts across multiple modules or crates
- EXC-002: Test utilities are shared across multiple crates and warrant a separate test-support crate

## Rationale

- Pattern detected across 77 files with 89.42% confidence indicates this is a proven, stable approach that has been successfully adopted throughout the codebase
- Rust's #[cfg(test)] conditional compilation provides zero-cost abstraction - tests are completely removed from production builds, ensuring no runtime or binary size overhead
- Co-locating tests with implementation code reduces cognitive load and makes it easier to maintain test coverage as code evolves, supporting the project's quality goals
- Access to private module members enables thorough testing of implementation details without compromising encapsulation in the public API, supporting both quality and design principles

## Consequences

Positive:
- Tests remain synchronized with implementation changes, reducing the risk of stale or outdated tests
- Developers can test private implementation details without exposing internal APIs, maintaining proper encapsulation
- Zero runtime overhead - test code is completely excluded from production builds through conditional compilation
- Reduced context switching for developers who can view and modify tests in the same file as the implementation
- Consistent testing pattern across 77 files creates predictable codebase structure and reduces onboarding time

Negative:
- Source files may become longer and harder to navigate when they contain extensive test suites
- Test code increases compilation time during development even though it's excluded from release builds
- Inline tests cannot easily test interactions between modules or integration scenarios without additional test infrastructure
- Refactoring that splits modules may require moving and reorganizing test code along with implementation code

## Alternatives

- Separate test files in parallel directory structure (e.g., src/foo.rs and tests/unit/foo.rs) (rejected)
  Rejected because: Increases maintenance burden by requiring developers to navigate between files, and the pattern evidence shows strong preference for inline tests (77 files with 89.42% confidence)
  When valid: May be appropriate for very large test suites that would make source files unwieldy (>1000 lines of test code)
- External test-only crates that depend on the main crate (rejected)
  Rejected because: Cannot test private implementation details, only public APIs, which limits test coverage and contradicts the detected pattern of testing internal logic
  When valid: Appropriate for integration tests and cross-crate API contract testing (already used in tests/it/ directory)
- Mix of inline unit tests and separate integration tests (accepted)
  When valid: This is the actual implemented pattern - inline #[cfg(test)] modules for unit tests, with tests/ directory for integration tests as seen in uv-client/tests/it/

## Risks

- Source files may become excessively large if test code is not managed, making navigation and maintenance difficult
  Mitigation: Establish guidelines for maximum test module size (e.g., 500 lines) and extract to integration tests or test utilities when exceeded. Use IDE folding features to collapse test modules during development.
  Owner: Engineering team
- Developers may inadvertently create dependencies between test code and production code that affect compilation or architecture
  Mitigation: Enforce strict #[cfg(test)] usage through CI checks and code review. Use cargo build --release to verify production builds contain no test artifacts.
  Owner: CI/CD pipeline and code reviewers
- Inconsistent test organization across crates may emerge as the project grows, reducing the benefits of standardization
  Mitigation: Document testing patterns in CONTRIBUTING.md, provide test templates, and include testing structure in code review checklist. Monitor pattern adherence through automated detection.
  Owner: Technical leadership and documentation team

## Implementation Notes

- Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability
- Use 'mod tests' as the standard module name for test modules unless a more specific name adds clarity (e.g., 'mod parsing_tests')
- Import necessary items at the top of the test module using 'use super::*;' to access parent module members, and add specific external imports as needed
- For common test utilities used across multiple files within a crate, create a tests/common/mod.rs file and import with 'use crate::common::*;' in integration tests
- Run 'cargo test' to execute all tests, 'cargo test --lib' for unit tests only, and 'cargo test --test integration_test_name' for specific integration tests

## Continuation Context


Verify commands:
- grep -r '#\[cfg(test)\]' crates/*/src --include='*.rs' | wc -l
- cargo test --workspace --lib --no-fail-fast
- cargo build --release && ! grep -r 'cfg(test)' target/release/ 2>/dev/null

Accept when:
- All library crates contain at least one #[cfg(test)] module with unit tests
- All unit tests pass successfully with 'cargo test --lib' command
- Release builds contain no test code artifacts (verified by absence of test symbols in release binaries)
- Code coverage for unit tests meets project threshold (typically >80% for core logic)

## Enforcement

- Verified by: Automated CI pipeline runs 'cargo test --workspace' on every pull request
- Verified by: Code review checklist includes verification of #[cfg(test)] usage for new test code
- Verified by: Static analysis tools check for test code patterns outside of #[cfg(test)] modules
- Verified by: Release build verification ensures no test artifacts in production binaries
- Violation handling: CI pipeline fails if tests are not properly marked with #[cfg(test)] and can be detected in release builds
- Violation handling: Pull requests without tests for new functionality are flagged for review and require justification
- Violation handling: Code review feedback requests addition of #[cfg(test)] modules when missing from new source files
- Violation handling: Quarterly audits identify files lacking test coverage and create remediation tasks
- Exception process: Request exception through pull request description with clear justification for alternative testing approach
- Exception process: Technical lead or architect reviews exception request and approves/denies based on technical merit
- Exception process: Approved exceptions are documented in tests/README.md or relevant crate documentation
- Exception process: Exception decisions are reviewed quarterly to determine if they should become permanent patterns or be refactored