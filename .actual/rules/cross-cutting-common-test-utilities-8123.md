# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Common Test Utilities

These rules are ALWAYS ACTIVE for all Rust unit tests written in library crates and module files using inline #[cfg(test)] modules.

### Rules

- **R-TEST-001** SHOULD: Common test utilities and fixtures SHOULD be extracted to shared test modules (e.g., tests/common/mod.rs) when used across multiple test files.
- **R-TEST-002** MUST: Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability.
- **R-TEST-003** SHOULD: Use 'mod tests' as the standard module name for test modules unless a more specific name adds clarity (e.g., 'mod parsing_tests').
- **R-TEST-004** MUST: Import necessary items at the top of the test module using 'use super::*;' to access parent module members, and add specific external imports as needed.
- **R-TEST-005** SHOULD: For common test utilities used across multiple files within a crate, create a tests/common/mod.rs file and import with 'use crate::common::*;' in integration tests.

### Verify

```bash
# Count #[cfg(test)] modules across crates
grep -r '#\[cfg(test)\]' crates/*/src --include='*.rs' | wc -l

# Run all unit tests
cargo test --workspace --lib --no-fail-fast

# Verify release builds contain no test artifacts
cargo build --release && ! grep -r 'cfg(test)' target/release/ 2>/dev/null
```

**Accept when:**
- All library crates contain at least one #[cfg(test)] module with unit tests
- All unit tests pass successfully with 'cargo test --lib' command
- Release builds contain no test code artifacts (verified by absence of test symbols in release binaries)
- Code coverage for unit tests meets project threshold (typically >80% for core logic)

<enforcement>
Claude Code MUST NOT skip or defer verification. All inline test modules MUST use #[cfg(test)] conditional compilation and MUST be verified to be excluded from production builds.
</enforcement>