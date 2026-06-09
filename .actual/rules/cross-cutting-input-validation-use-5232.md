# Enforce Strict Input Validation with Type-Safe Parsing: Input Validation Use

These rules are ALWAYS ACTIVE for all code that processes external input, including CLI arguments, HTTP responses, configuration files, and user-provided data.

### Rules

- **R-INPUT-001** MUST: Input validation MUST use type-safe parsing with explicit error handling rather than assuming input correctness.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All external input parsing must be validated at trust boundaries using type-safe parsing with explicit error handling. Security team review is required for any modifications to validation logic in security-critical modules.
</enforcement>