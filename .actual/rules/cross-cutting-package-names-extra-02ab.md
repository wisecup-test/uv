# Enforce Strict Input Validation with Type-Safe Parsing: Package Names Extra

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, netrc files, HTML content, and user-provided data.

### Rules

- **R-VAL-001** MUST: Package names, extra names, group names, and identifiers MUST be validated against defined naming rules and character constraints before use in any operation.
- **R-VAL-002** MUST: All external input parsing (CLI arguments, HTTP responses, configuration files, credentials, HTML content, file paths, URLs) MUST return Result types with explicit error handling; no unwrap() or expect() on external input in production code paths.
- **R-VAL-003** MUST: Validation MUST be implemented as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-VAL-004** MUST: Validation errors MUST return descriptive error types (not just String) that can be programmatically handled and provide context for debugging.
- **R-VAL-005** SHOULD: Use the uv-normalize crate for package name, extra name, and group name validation to ensure consistency across the codebase.
- **R-VAL-006** SHOULD: For complex parsing (HTML, netrc), use established libraries (html5ever, netrc parsers) rather than custom implementations.
- **R-VAL-007** SHOULD: Validation rules MUST be documented in module documentation so users understand what input is acceptable.

### Verify

```bash
# Check for unwrap() and expect() on external input in production code
grep -r 'unwrap()' crates/uv-cli/src/ crates/uv-client/src/ crates/uv-normalize/src/ | grep -v test | wc -l

# Verify FromStr implementations have explicit error types
rg 'impl.*FromStr' crates/uv-normalize/src/ -A 5 | grep -c 'type Err'

# Run validation test suites
cargo test --package uv-normalize --package uv-cli -- validation

# Check for unchecked or disabled validation patterns
rg 'parse.*unchecked|validate.*false' crates/ --type rust
```

**Accept when:**
- All external input parsing returns Result types with explicit error handling (no unwrap() in production code paths)
- Validation tests exist for each input type covering valid, invalid, and boundary cases
- Security-sensitive modules (uv-client, uv-netrc, uv-keyring) have no unvalidated external input usage
- Common validation patterns are extracted into reusable modules with documented validation rules
- Validation logic includes tests for security-relevant attack patterns (injection attempts, path traversal)

<enforcement>
Claude Code MUST NOT skip or defer verification. All external input must be validated at trust boundaries before use. Security team review is required for any changes to validation logic in security-critical modules.
</enforcement>