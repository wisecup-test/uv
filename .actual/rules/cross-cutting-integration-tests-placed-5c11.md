# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Integration Tests Placed

These rules are ALWAYS ACTIVE for all Rust source files in library crates where unit tests are written.

### Rules

- **R-TEST-001** MUST: Place unit tests in inline #[cfg(test)] modules within the same source file as the implementation code they test.
- **R-TEST-002** MUST: Use `mod tests` as the standard module name for test modules unless a more specific name adds clarity.
- **R-TEST-003** MUST: Import necessary items at the top of the test module using `use super::*;` to access parent module members.
- **R-TEST-004** MAY: Place integration tests in separate files under the `tests/` directory when testing public API contracts across module boundaries.
- **R-TEST-005** SHOULD: Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability.
- **R-TEST-006** SHOULD: Extract test code to integration tests or test utilities when a test module exceeds 500 lines.

### Verify

```bash
# Count inline test modules
grep -r '#\[cfg(test)\]' crates/*/src --include='*.rs' | wc -l

# Run all unit tests
cargo test --workspace --lib --no-fail-fast

# Verify release builds contain no test artifacts
cargo build --release && ! grep -r 'cfg(test)' target/release/ 2>/dev/null
```

**Accept when:**
- All library crates contain at least one #[cfg(test)] module with unit tests
- All unit tests pass successfully with `cargo test --lib` command
- Release builds contain no test code artifacts (verified by absence of test symbols in release binaries)
- Code coverage for unit tests meets project threshold (typically >80% for core logic)

<enforcement>
Claude Code MUST NOT skip or defer verification. All new test code must be marked with #[cfg(test)] and placed inline with implementation. Integration tests require exception approval per the ADR exception process.
</enforcement>