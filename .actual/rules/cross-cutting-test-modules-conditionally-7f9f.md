# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Test Modules Conditionally

These rules are ALWAYS ACTIVE for all Rust unit tests and test modules in library crates using inline #[cfg(test)] conditional compilation.

### Rules

- **R-TEST-001** MUST: Test modules MUST be conditionally compiled using #[cfg(test)] to ensure they are excluded from production builds.
- **R-TEST-002** MUST: Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability.
- **R-TEST-003** SHOULD: Use 'mod tests' as the standard module name for test modules unless a more specific name adds clarity (e.g., 'mod parsing_tests').
- **R-TEST-004** SHOULD: Import necessary items at the top of the test module using 'use super::*;' to access parent module members, and add specific external imports as needed.
- **R-TEST-005** MAY: For common test utilities used across multiple files within a crate, create a tests/common/mod.rs file and import with 'use crate::common::*;' in integration tests.

### Verify

```bash
# Count #[cfg(test)] modules in source files
grep -r '#\[cfg(test)\]' crates/*/src --include='*.rs' | wc -l

# Run all unit tests
cargo test --workspace --lib --no-fail-fast

# Verify release builds contain no test artifacts
cargo build --release && ! grep -r 'cfg(test)' target/release/ 2>/dev/null

# Run specific test commands
cargo test                          # Execute all tests
cargo test --lib                    # Unit tests only
cargo test --test integration_test_name  # Specific integration tests
```

**Accept when:**
- All library crates contain at least one #[cfg(test)] module with unit tests
- All unit tests pass successfully with 'cargo test --lib' command
- Release builds contain no test code artifacts (verified by absence of test symbols in release binaries)
- Code coverage for unit tests meets project threshold (typically >80% for core logic)
- All new source files with test code use #[cfg(test)] conditional compilation

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests must pass the verify commands before acceptance. Code review MUST check for #[cfg(test)] usage on all new test code. CI pipeline MUST fail if tests are not properly marked with #[cfg(test)] and can be detected in release builds.
</enforcement>