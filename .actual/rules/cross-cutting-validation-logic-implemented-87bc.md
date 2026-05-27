# Enforce Strict Input Validation with Type-Safe Parsing: Validation Logic Implemented

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, and user-provided data.

### Rules

- **R-VAL-001** MUST: Validation logic MUST be implemented at system boundaries (CLI parsers, HTTP clients, file readers) before data enters the application domain.
- **R-VAL-002** MUST: All external input parsing MUST return Result types with explicit error handling (no unwrap() in production code paths).
- **R-VAL-003** MUST: Security-sensitive modules (uv-client, uv-netrc, uv-keyring) MUST have no unvalidated external input usage.
- **R-VAL-004** SHOULD: Use the uv-normalize crate for package name, extra name, and group name validation to ensure consistency.
- **R-VAL-005** SHOULD: Implement validation as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-VAL-006** SHOULD: Return descriptive error types (not just String) that can be programmatically handled and provide context for debugging.
- **R-VAL-007** SHOULD: For complex parsing (HTML, netrc), use established libraries (html5ever, netrc parsers) rather than custom implementations.

### Verify

```bash
# Check for unwrap() usage on external input in security-critical modules
grep -r 'unwrap()' crates/uv-cli/src/ crates/uv-client/src/ crates/uv-normalize/src/ | grep -v test | wc -l

# Verify FromStr implementations have explicit error types
rg 'impl.*FromStr' crates/uv-normalize/src/ -A 5 | grep -c 'type Err'

# Run validation test suites
cargo test --package uv-normalize --package uv-cli -- validation

# Check for unchecked or unvalidated parsing patterns
rg 'parse.*unchecked|validate.*false' crates/ --type rust
```

**Accept when:**
- All external input parsing returns Result types with explicit error handling (no unwrap() in production code paths)
- Validation tests exist for each input type covering valid, invalid, and boundary cases
- Security-sensitive modules (uv-client, uv-netrc, uv-keyring) have no unvalidated external input usage
- Common validation patterns are extracted into reusable modules with documented validation rules
- Clippy warnings on unsafe patterns are addressed or explicitly documented with allow() and justification

<enforcement>
Claude Code MUST NOT skip or defer verification. All external input handling MUST be validated at system boundaries before use. Security team review is required for any exception in security-critical paths.
</enforcement>