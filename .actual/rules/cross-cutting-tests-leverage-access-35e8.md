# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Tests Leverage Access

These rules are ALWAYS ACTIVE for all Rust unit test code written in library crates using inline test modules with #[cfg(test)] conditional compilation.

### Rules

- **R-TEST-001** SHOULD: Tests SHOULD leverage access to private module members to thoroughly test implementation details without exposing internal APIs.
- **R-TEST-002** SHOULD: Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability.
- **R-TEST-003** SHOULD: Use 'mod tests' as the standard module name for test modules unless a more specific name adds clarity.
- **R-TEST-004** SHOULD: Import necessary items at the top of the test module using 'use super::*;' to access parent module members.
- **R-TEST-005** SHOULD: For common test utilities used across multiple files within a crate, create a tests/common/mod.rs file and import with 'use crate::common::*;' in integration tests.

### Verify

```bash
# Count inline test modules across crates
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
Claude Code MUST NOT skip or defer verification. All unit tests must pass and release builds must be free of test artifacts before accepting changes.
</enforcement>