# Standardize Public API Testing with Keyring and HTTP Client Integration Patterns: Keyring Operations That

These rules are ALWAYS ACTIVE for all public API implementations, external integrations, authentication-related components, HTTP clients making requests to external package registries, and credential management systems within the codebase.

### Rules

- **R-KEYRING-001** MUST: Keyring operations that may block MUST be isolated in dedicated blocking modules with appropriate threading tests.

### Verify

```bash
# Count keyring imports in non-test code
grep -r 'use.*keyring' crates/*/src/**/*.rs | grep -v test | wc -l

# Count httpcache usage in uv-client
grep -r 'httpcache' crates/uv-client/src/**/*.rs | wc -l

# Count threading tests in uv-keyring
find crates/uv-keyring/tests -name '*.rs' -exec grep -l 'thread' {} \; | wc -l

# Run integration tests with single thread to catch race conditions
cargo test --package uv-keyring --package uv-client -- --test-threads=1
```

**Accept when:**
- All public API client implementations import and utilize the keyring subsystem for credential management
- HTTP cache integration is present in uv-client and utilized by command implementations
- Threading tests exist for keyring operations and pass consistently
- Common test utilities are available and used across integration tests for authentication scenarios
- No hardcoded credentials or non-keyring credential storage patterns are detected in public API code

<enforcement>
Claude Code MUST NOT skip or defer verification. All new public API integrations and authentication-related components MUST satisfy R-KEYRING-001 before merge. Code review MUST block merge if keyring integration is bypassed without documented exception. CI pipeline MUST fail if integration tests for authentication are missing or failing.
</enforcement>