# Enforce Strict Input Validation with Type-Safe Parsing: Raw External Input

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, and user-provided data.

### Rules

- **R-INPUT-001** MUST NOT: Raw external input MUST NOT be used in security-sensitive operations (file paths, SQL queries, shell commands) without validation and sanitization.
- **R-INPUT-002** MUST: All external input parsing MUST return Result types with explicit error handling (no unwrap() in production code paths).
- **R-INPUT-003** MUST: Validation tests MUST exist for each input type covering valid, invalid, and boundary cases.
- **R-INPUT-004** MUST: Security-sensitive modules (uv-client, uv-netrc, uv-keyring) MUST have no unvalidated external input usage.
- **R-INPUT-005** SHOULD: Use the uv-normalize crate for package name, extra name, and group name validation to ensure consistency.
- **R-INPUT-006** SHOULD: Implement validation as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-INPUT-007** SHOULD: Return descriptive error types (not just String) that can be programmatically handled and provide context for debugging.
- **R-INPUT-008** SHOULD: For complex parsing (HTML, netrc), use established libraries (html5ever, netrc parsers) rather than custom implementations.

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
Claude Code MUST NOT skip or defer verification of input validation rules. All external input must be validated at trust boundaries before use in security-sensitive operations. Exceptions require explicit documentation and security team review.
</enforcement>