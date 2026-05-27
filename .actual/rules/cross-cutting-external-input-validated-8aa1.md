# Enforce Strict Input Validation with Type-Safe Parsing: External Input Validated

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, and user-provided data.

### Rules

- **R-INPUT-001** MUST: All external input MUST be validated before use, including CLI arguments, HTTP responses, file contents, and user-provided strings.
- **R-INPUT-002** MUST: All external input parsing MUST return Result types with explicit error handling (no unwrap() in production code paths).
- **R-INPUT-003** MUST: Validation logic MUST be implemented as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-INPUT-004** MUST: Return descriptive error types (not just String) that can be programmatically handled and provide context for debugging.
- **R-INPUT-005** SHOULD: Use the uv-normalize crate for package name, extra name, and group name validation to ensure consistency.
- **R-INPUT-006** SHOULD: For complex parsing (HTML, netrc), use established libraries (html5ever, netrc parsers) rather than custom implementations.
- **R-INPUT-007** SHOULD: Document validation rules in module documentation so users understand what input is acceptable.

### Verify

```bash
# Check for unwrap() usage on external input in production code
grep -r 'unwrap()' crates/uv-cli/src/ crates/uv-client/src/ crates/uv-normalize/src/ | grep -v test | wc -l

# Verify FromStr implementations have error types
rg 'impl.*FromStr' crates/uv-normalize/src/ -A 5 | grep -c 'type Err'

# Run validation test suites
cargo test --package uv-normalize --package uv-cli -- validation

# Check for unchecked parsing patterns
rg 'parse.*unchecked|validate.*false' crates/ --type rust
```

**Accept when:**
- All external input parsing returns Result types with explicit error handling (no unwrap() in production code paths)
- Validation tests exist for each input type covering valid, invalid, and boundary cases
- Security-sensitive modules (uv-client, uv-netrc, uv-keyring) have no unvalidated external input usage
- Common validation patterns are extracted into reusable modules with documented validation rules
- Descriptive error types are used throughout input validation logic
- Established parsing libraries are used for complex formats rather than custom implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All external input validation MUST be verified before accepting code changes. Security team review is required for any modifications to validation logic in security-critical modules.
</enforcement>