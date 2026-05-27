# Adopt Inline Test Modules with #[cfg(test)] for Unit Testing: Unit Tests Placed

These rules are ALWAYS ACTIVE for all Rust unit test code written in library crates and module files across the project.

### Rules

- **R-UNIT-001** MUST: Unit tests MUST be placed in inline test modules marked with #[cfg(test)] within the same file as the code under test.
- **R-UNIT-002** MUST: Test modules MUST use 'mod tests' as the standard module name unless a more specific name adds clarity (e.g., 'mod parsing_tests').
- **R-UNIT-003** MUST: Test modules MUST import necessary items using 'use super::*;' to access parent module members, with specific external imports added as needed.
- **R-UNIT-004** SHOULD: Place the #[cfg(test)] module at the end of each source file to keep implementation code at the top and maintain readability.
- **R-UNIT-005** SHOULD: Extract test code to integration tests or test utilities when a test module exceeds 500 lines to prevent source files from becoming unwieldy.

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
Claude Code MUST NOT skip or defer verification. All three verify commands MUST execute successfully before accepting changes to unit test structure.
</enforcement>