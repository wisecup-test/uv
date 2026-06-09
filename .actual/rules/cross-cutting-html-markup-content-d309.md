# Enforce Strict Input Validation with Type-Safe Parsing: Html Markup Content

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, HTML content, and user-provided data.

### Rules

- **R-HTML-001** MUST: HTML and markup content MUST be parsed with dedicated parsers that handle malformed input safely.
- **R-HTML-002** MUST: All external input parsing MUST return Result types with explicit error handling (no unwrap() in production code paths).
- **R-HTML-003** MUST: Security-sensitive modules (uv-client, uv-netrc, uv-keyring) MUST have no unvalidated external input usage.
- **R-HTML-004** SHOULD: Implement validation as TryFrom or FromStr traits to integrate with Rust's type system and enable ? operator usage.
- **R-HTML-005** SHOULD: Use established libraries (html5ever, netrc parsers) rather than custom implementations for complex parsing.
- **R-HTML-006** SHOULD: Return descriptive error types (not just String) that can be programmatically handled and provide context for debugging.

### Verify

```bash
# Check for unwrap() usage in security-critical modules
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
- HTML and markup content uses dedicated parsers (html5ever or equivalent) rather than string manipulation
- All validation logic includes comprehensive test coverage including boundary cases and security-relevant attack patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All external input parsing must be validated before use. Security-critical modules require explicit error handling and comprehensive test coverage.
</enforcement>