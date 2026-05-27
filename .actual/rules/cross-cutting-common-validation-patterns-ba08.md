# Enforce Strict Input Validation with Type-Safe Parsing: Common Validation Patterns

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, netrc files, HTML content, and user-provided data.

### Rules

- **R-VALIDATION-001** SHOULD: Common validation patterns SHOULD be extracted into reusable validation modules (e.g., uv-normalize for name validation).
- **R-VALIDATION-002** MUST: All external input parsing MUST return Result types with explicit error handling (no unwrap() in production code paths).
- **R-VALIDATION-003** MUST: Security-sensitive modules (uv-client, uv-netrc, uv-keyring) MUST have no unvalidated external input usage.
- **R-VALIDATION-004** SHOULD: Validation SHOULD be implemented as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-VALIDATION-005** SHOULD: Descriptive error types (not just String) SHOULD be returned that can be programmatically handled and provide context for debugging.
- **R-VALIDATION-006** SHOULD: For complex parsing (HTML, netrc), established libraries (html5ever, netrc parsers) SHOULD be used rather than custom implementations.

### Verify

```bash
# Check for unwrap() usage in external input parsing
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
- Validation logic includes tests for boundary cases, malformed input, and security-relevant attack patterns (injection attempts, path traversal)

<enforcement>
Claude Code MUST NOT skip or defer verification of input validation rules. All external input processing MUST be validated at trust boundaries. Security team review is required for any exceptions in security-critical paths.
</enforcement>