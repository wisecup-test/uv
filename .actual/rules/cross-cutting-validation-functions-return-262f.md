# Enforce Strict Input Validation with Type-Safe Parsing: Validation Functions Return

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, netrc files, HTML content, and user-provided data.

### Rules

- **R-VAL-001** SHOULD: Validation functions SHOULD return Result types with descriptive error messages indicating what validation failed and why.

### Verify

```bash
# Check for unwrap() usage on external input in security-critical modules
grep -r 'unwrap()' crates/uv-cli/src/ crates/uv-client/src/ crates/uv-normalize/src/ | grep -v test | wc -l

# Verify FromStr implementations have explicit error types
rg 'impl.*FromStr' crates/uv-normalize/src/ -A 5 | grep -c 'type Err'

# Run validation test suites
cargo test --package uv-normalize --package uv-cli -- validation

# Check for unchecked parsing or disabled validation
rg 'parse.*unchecked|validate.*false' crates/ --type rust
```

**Accept when:**
- All external input parsing returns Result types with explicit error handling (no unwrap() in production code paths)
- Validation tests exist for each input type covering valid, invalid, and boundary cases
- Security-sensitive modules (uv-client, uv-netrc, uv-keyring) have no unvalidated external input usage
- Common validation patterns are extracted into reusable modules with documented validation rules
- Validation functions use TryFrom or FromStr traits to integrate with Rust's type system
- Error types are descriptive (not just String) and provide context for debugging

<enforcement>
Clause Code MUST NOT skip or defer verification of validation function return types and error handling. All external input parsing MUST return Result types with explicit error messages. Security team review is required for any modifications to validation logic in security-critical modules.
</enforcement>