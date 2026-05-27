# Adopt Blocking Wrapper Pattern for Async-to-Sync Bridge Operations: Blocking Wrappers Not

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Activation

This ADR is ACTIVE for all Rust codebases implementing async/sync interoperability patterns. It governs how blocking wrappers should be structured when bridging asynchronous runtime operations to synchronous APIs.

## Context

- The codebase contains multiple modules that need to expose synchronous APIs while internally using async operations (e.g., uv-keyring blocking.rs, credential.rs, macos.rs)
- Rust's async ecosystem requires explicit runtime management when bridging between async and sync contexts, particularly for library code that must support both paradigms
- Evidence shows 38 files across the codebase implementing this pattern with 89.26% confidence, indicating a deliberate architectural choice for concurrency model abstraction
- The pattern appears in critical paths including keyring operations, installer uninstall logic, metadata handling, and command execution layers
- Testing infrastructure (threading.rs, basic.rs) and integration tests (auth.rs, build.rs) demonstrate the pattern's necessity for both production and test scenarios

## Problem Statement

When building Rust applications that use async/await internally but must expose synchronous APIs to callers (or vice versa), there is no standard approach for managing the async-to-sync boundary. Without a consistent pattern, teams may create ad-hoc solutions leading to runtime panics, deadlocks, or inconsistent error handling across the codebase. This is particularly problematic in library code where the caller's execution context (async vs sync) is unknown.

## Decision

1. MUST_NOT: Blocking wrappers MUST NOT be called from within an async context where .await would be appropriate, to avoid unnecessary runtime overhead

## Policy Block

- MUST_NOT Blocking wrappers MUST NOT be called from within an async context where .await would be appropriate, to avoid unnecessary runtime overhead

In scope:
- All Rust modules that expose synchronous APIs wrapping async operations
- Library code (crates) that must support both async and sync consumers
- Keyring operations, credential management, and system integration layers
- Command-line interface implementations that call async backend services
- Test infrastructure requiring synchronous test execution of async code

Out of scope:
- Pure async code paths where all callers are async-aware
- Performance-critical hot loops where blocking would introduce unacceptable latency
- WebAssembly targets where tokio runtime may not be available
- No-std environments lacking runtime support

Exceptions:
- EXC-001: The module is exclusively used in async contexts and will never be called synchronously
- EXC-002: Platform-specific constraints prevent runtime initialization (e.g., embedded systems)

## Rationale

- The pattern appears in 38 files with 89.26% confidence, demonstrating widespread adoption and validation through production use
- Separating blocking wrappers into dedicated modules (blocking.rs) provides clear API boundaries and prevents accidental misuse of async functions in sync contexts
- Evidence from uv-keyring threading tests shows this pattern successfully handles concurrent access scenarios, which are common in CLI tools and multi-threaded applications
- The pattern enables library code to serve both async and sync consumers without forcing API consumers to adopt async/await, maintaining backward compatibility and ergonomics

## Consequences

Positive:
- Clear separation of concerns between async and sync APIs improves code maintainability and reduces cognitive load
- Library consumers can choose the appropriate API (async or blocking) based on their execution context without code duplication
- Consistent error handling across async/sync boundaries reduces debugging complexity
- Testing becomes more straightforward as blocking wrappers enable standard synchronous test patterns

Negative:
- Additional runtime overhead from spawning/managing tokio runtime instances in blocking contexts
- Increased code surface area requiring maintenance of parallel async and sync implementations
- Potential for deadlocks if blocking wrappers are misused within async contexts (mitigated by R-17-006)
- Binary size increase from including both async runtime and blocking wrapper infrastructure

## Alternatives

- Expose only async APIs and require all consumers to adopt async/await (rejected)
  Rejected because: Forces async adoption on all consumers including simple CLI tools and test code, creating unnecessary complexity and migration burden
  When valid: Valid for new greenfield projects with no legacy sync consumers and where async is universally beneficial
- Use callback-based APIs instead of async/await to avoid runtime complexity (rejected)
  Rejected because: Callback-based patterns are less ergonomic in Rust, harder to compose, and don't integrate well with the async ecosystem
  When valid: Valid for FFI boundaries or C-compatible APIs where async/await cannot cross language boundaries
- Implement fully synchronous versions of all operations without async internals (rejected)
  Rejected because: Duplicates business logic and prevents leveraging async ecosystem libraries (tokio, hyper, etc.) that provide superior performance for I/O operations
  When valid: Valid for simple operations with no I/O or where async overhead exceeds benefits

## Risks

- Blocking wrappers called from async contexts can cause thread pool exhaustion and deadlocks
  Mitigation: Enforce R-17-006 through clippy lints and code review. Document blocking nature clearly in API documentation with #[must_use] attributes where appropriate
  Owner: Engineering team / Runtime architecture group
- Runtime initialization overhead in blocking wrappers may impact performance in tight loops
  Mitigation: Use lazy_static or thread_local runtime instances for hot paths. Profile and optimize based on actual usage patterns
  Owner: Performance engineering team
- Inconsistent error handling between async and blocking implementations could lead to subtle bugs
  Mitigation: Implement comprehensive integration tests (as seen in auth.rs, build.rs) that exercise both paths. Use shared error types and conversion traits
  Owner: Quality assurance / Testing team

## Implementation Notes

- Create a blocking.rs module alongside async implementations, re-exporting types with blocking method variants (e.g., fetch() becomes fetch_blocking())
- Use tokio::runtime::Builder::new_current_thread() for lightweight single-threaded runtimes in blocking wrappers to minimize overhead
- Implement Drop traits for runtime cleanup in blocking wrapper types to ensure proper resource management
- Add module-level documentation explaining when to use async vs blocking APIs, with code examples for both patterns
- Consider using #[cfg(feature = "blocking")] to make blocking wrappers optional for consumers who only need async APIs

## Continuation Context


Verify commands:
- grep -r "pub mod blocking" crates/ --include="*.rs" | wc -l
- rg "Runtime::block_on|Handle::block_on" crates/ --type rust -c
- cargo test --all-features -- threading --nocapture

Accept when:
- All modules exposing sync wrappers for async operations have dedicated blocking.rs files or clearly marked blocking functions
- Grep commands show consistent usage of Runtime::block_on or Handle::block_on in blocking wrapper implementations
- Threading tests pass demonstrating safe concurrent access to blocking wrappers without deadlocks or panics

## Enforcement

- Verified by: Automated CI checks using grep/ripgrep to detect blocking wrapper patterns in new modules
- Verified by: Code review checklist requiring blocking wrapper implementation for any new async-to-sync boundaries
- Verified by: Clippy custom lints detecting block_on calls within async functions (enforcing R-17-006)
- Violation handling: CI pipeline fails if new async modules lack corresponding blocking wrappers when exposing public APIs
- Violation handling: Code review blocks merge if blocking wrappers are called from async contexts without justification
- Violation handling: Runtime warnings or panics in debug builds when detecting nested runtime usage patterns
- Exception process: Submit exception request to architecture review board with justification and alternative approach
- Exception process: Document exception in module-level comments with ADR reference and rationale
- Exception process: Add suppression annotations (e.g., #[allow(clippy::...)]) with explanatory comments for lint exceptions